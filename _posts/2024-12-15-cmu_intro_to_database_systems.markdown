# [Assignments](https://15445.courses.cs.cmu.edu/fall2024/assignments.html)

# [Lecture #1: Intro](https://www.youtube.com/watch?v=APqWIjtzNGE&list=PLSE8ODhjZXjYDBpQnSymaectKjxCy6BYq&index=3)
Skip this.

# [Lecture #2: Basic SQL Statements](https://www.youtube.com/watch?v=MzigBKf84aY&list=PLSE8ODhjZXjYDBpQnSymaectKjxCy6BYq&index=4)
Skip this if you've used SQL before.

# [Lecture #3: Database Storage, Files and Pages](https://www.youtube.com/watch?v=dSxV5Sob5V8&list=PLSE8ODhjZXjYDBpQnSymaectKjxCy6BYq)

* Storage hierarchy: CPU registers to network
    * Optane and CXL mentions (different assumptions than typical DRAM)
    * Latency rules of thumb: 1-4ns L1/L2 Cache, 100ns DRAM, 16k ns SSD, 2M ns HDD, 50M ns Network, 1B ns Tape Archive
* DBMS tries to maximize sequential access to disk (much faster)
* Disk has pages, usually managed by fs, read/written into memory atomically
* Most systems now use FS-managed files (some use lots of files, some use single files).
    * DBMS typically doesn't handle redudancy
* DB pages typically are self-describing
    * Can be bad if you have billions of small pages (redundant)
    * Good for disaster recovery
* Why would a DB decide to use larger pages than the underlying OS page size?
    * (row-based) Read-heavy loads prefer larger pages to reduce trips to disk (esp when scanning)
    * Writes now require more orchestration
* Page File storage can be 
    * Heap (most common); unordered tuples in pages
    * Tree; index organized structures
    * Sequential / Sorted
    * Hashing
* Heap needs additional metadata to track file location and free space
    * Page directory stores this table, file, page assoc
    * PG has `pg_freespace()` and `pg_relation_filepath()`

## page layout: how to store tuples in page?
* Page Header has metadata (size, checksum, version, transaction visibility, encoding metadata, schema info, data summary, ...)
* Page Layout (row-oriented, storing tuples)
    * Tuple-oriented storage (update them as they come)
    * Log-structured storage (redundant copies, append as they come along)
    * Index-organized storage (organized as tree, leaves are data: mysql, sqlite)
* Tuples: how do we store them?
    * Number in header, append them?
        * Update is PITA, might have to shift a bunch if sizes are different
        * Log structured DBs do this (no deletion!)
    * Slotted pages:
        * Slot array in the header, store offsets to the tuples.
        * Header grows down, tuples grow from end up.
        * Can compact on deletion (inexpensive if in memory), or just leave it.
            * PG leaves hole.
            * Moving pages around (compacting) requires system to hold lock longer.

## Neon: serverless postgres
* Branching, autoscaling.
* Architecture: Separate compute and storage
    * PG is compute, run in cloud by Neon.
    * Storage is Neon storage system.
        * Write-Ahead-Log moves through Safekeepers, then to Pageservers
        * Intercept WAL to send to safekeepers, they form consensus via paxos
        * Reuses a lot of PG synchronous replica storage.
    * Pagekeepers are bespoke
    * Stores log history, processes into log-structured merge tree for branching, time-travel, etc.
    * Intercept PG ask for 8KB page, Pageserver reconstructs page from log, sends back to PG.
    * Requests contain page number and LSM number (page version).
    * Since WAL is intercepted, PG page eviction just throws things away, doesn't use checkpoint logic.
* Single writer, multiple reader
    * No distributed transactions nor queries
        * Usually trading perf or consistency to accomplish these, hard to retrofit into existing DB (PG)
* Multi-tenant
* Cheap COW breanching and timetravel queries
* How to make PG serverless?
    * Separate compute / storage (quick spinup)
        * <1s startup, no WAL
    * Proxy to accept connections (intercept connections, startup if necessary)
    * PG in k8s VM
    * Control plane orchestration
* Autoscaling
    * VMs can be resized, challenging to know when to scale
    * CPU is obvious, worst case slows down
    * Local disk is hard (queries might fail)
    * Memory is hard (when do you hotplug more?)
        * When you're out?
        * When queries would benefit from more cache?
            * Overshooting case: in-memory DB
        * When you would get different query plans?
* No changes to PG planner or executor, suppors all extensions & tools, index types, and PG handles MVCC
    * MVCC isn't really necessary given Neon's time travel, but still supported


# [Database Storage: Log-Structured Merge Trees & Tuples](https://www.youtube.com/watch?v=IHtVWGhG0Xg)
* Last time talked about tuple-oriented architecture using a slot array.
    * Indirection gives you API: page and slot number provide access.
    * Call these "record IDs": file, page, slot.
        * Most DBMS don't actually use tuples.
        * PG has `table.ctid` => `(page, slot)`

* Insert / Update TODO

* Problems
    * Page fragmentation (empty slots)
    * Useless IO (Fetch page to update single tuple)
    * Random IO (adjacent tuples might be on separate pages)
* Some DBMS only create new pages, no overwrites (e.g. HDFS, Colossus)

## Log Structured Storage Merge Trees
* LS Merge Trees Append only model via log storing changes => Faster writes, slower reads
* In-memory structure (MemTable) written sequentially to disk (SSTable)

Index-Organized Storage
* Index like B+ tree instead of Heap

* MemTable has trees, SSTable sorted and stored on disk, only most recent value stored (log)
    * Separate WAL to recover MemTable on crash
* SSTables organized into different levels, written out periodically
    * Puts or delete operations, keys inserted low-high
        * Compaction to puts is more expensive if partial updates are common
    * Writes really fast (in-memory), reads need memtable, then SSTable bin search
    * But have 3 levels (0, 1, 2) of SSTable => store SummaryTable with e.g. min/max key per SSTable, key (bloom) filter per level.

* Compaction: sort-merge of SSTables to only keep latest value, may keep at same level or move down a level.
* Common Log-structured DBs:
    * RocksDB (via LevelDB @ FB / BigTable Google)
        * Facebook rewrote G's BigTable to remove MMap and manage their own pages with bufferpool (instead of OS-managed)
        * Cockroach forked and distributed this
    * HBase, yugabyteDB, fauna, TiDB, ClickHouse, CockroachDB, cassandra, WIREDTIGER, neon
        * neon ripped out tuples from PG (heap files &c), replaced with distributed log-structured system, materializes PG pages from log records
* Downsides: write amplification, expensive compaction


Indexes
* Both covered storage approaches rely on indexes to find tuples, because tables are unsorted.
    * What if DBMS could keep tuples sorted automatically?
* Index-organized storage.
    * DBMS stores tuples as value of index data struct
    * Uses slotted page layout
    * Tuples stored in page based on key
        * Can do binsearch on keys in header array
    * B+ Tree pays maintenance upfront, LSMs defer it.
    * More common in ~1990s: SQLite, MySQL, Oracle, SQLServer

Tuple Storage
* Tuple is a sequence of bytes with a metadata header
    * DBMS interprets bytes, its catalogs contain schema info about tables to determine tuple layout
    * Might pad or reorder for CPU word alignment
* Variable precison numbers specified by IEEE-754, faster but not exact
    * HW support for these
* Fixed precision numbers have arbitrary precision and scale (NUMERIC, DECIMAL)
    * Differing implementations, potentially lots of metadata/overhead
* Null data types
    * Null column bitmap header (each tuple), preserves space for eventual value
        * common in row stores
    * Sentinel values
        * common in column stores
    * Per-attribute null flag (need more than 1 bit due to word alignment)
        * Don't do this.
* Large values
    * Can't store massive tuple (e.g. 1B columns) as they won't fit in a single page
    * Can store values larger than a page via _overflow_ pages
    * Can use pointers to more locations
        * German strings can store prefix, size, and location, which allows for better prefix comparisons
            * Common in newer tech.
* External Value Storage
    * Like BLOB type, DBMS cannot manipulate contents


So Log-structured storage is preferable to tuple-oriented storage for write-heavy workloads.
Storage manager is not independent from DBMS.

# [5 - Row vs Column Storage + Compression](https://www.youtube.com/watch?v=nhlpwmOBEiE&list=PLSE8ODhjZXjYDBpQnSymaectKjxCy6BYq&index=6)
* Last class discussed approaches write-heavy workloads, but some are ready-heavy.
* Common Workload types
    * OLTP: read/update a small amount of data each time
    * OLAP: complex queries that read a lot of data
    * Hybrid Transaction + Analytical: OLTP & OLAP on the same instance


## Storage Models
Observation: relational model doesn't mean we have to store entire tuples in a page.
* OLTP: usually single tuple lookup, maybe some small joins.
* OLAP: read large portions of DB, usually executed on data collected from OLTP apps.
* Design will give different performance based on usage, has broad DBMS design implications.
    * N-ary Storage Model (NSM)
    * Decomposition Storage Model (DSM)
    * Hybrid Storage Model (PAX)

### N-ary Storage Model ("Row Stores")
Dump all attributes in tuple in single page, "row store"
* Ideal for OLTP, accessing individual entries with write-heavy loads
* Usually slotted page storage with record ID and header
* OLAP example: user accounts matching a given hostname, aggregte by last login
    * have to scan all pages to compare hostnames
    * have to scan all logins (concurrently with host=name)
    * fetches a bunch of unnecessary data (columns we don't care about)
* Good for:
    * Fast instert, update, delete
    * OLTP when you need the entire tuple
    * Index-oriented phys storage for clustering
* Bad for:
    * Scanning large portions of table or attribute subsets
    * Bad memory locality
    * Compression: multiple value domains in single page

### Decomposition Storage Model (DSM / "Column Stores")
Store single attribute for all tuples contiguously.
* Pages typically arrays of fixed-length values with header (usually null bitmap)
* Tuple Identification
    a. fixed-length offsets
    b. embedded tuple IDs on every value (bad idea), additional data structure that provides indexing for each column type
        * "contorting row store to look like a column store"
* Variable length data: dictionary compression to convert repetitive data into fixed length values (ref/pointer to string val)
* Good for:
    * Large scans / OLAP
    * Queries on a subset of attributes
    * Increased locality and cached data reuse
    * Data compression
* Bad for:
    * Queries needing the entire tuple
    * Point queries, inserts, updates, deletes due to splitting

Hybrid models exist: in-memory row store inserts, compaction in background to columns

### Partition Attributes Access Storage Model (PAX)
* OLAP almost never access a single column by itself, DSM must stitch back together
* Want a column store that keeps the data close for each tuple
* Goal: faster processing on columns with spatial locality of rows
    * "smaller blocks" stored as columns
    * Most modern column stores do this
        * Parquet, ORC, Arrow formats
* Physical Organization
    * Horizontally partition data into row groups, then partition attributes into column chunks.
    * Global metadata has row group offsets (footer if file is immutable)
    * Each row group has its own metadata header
    * e.g. 128MB row group, 1MB page sizes

### Compression
* IO from disk is the main bottleneck
    * Compressing pages can help, trading speed for compression ratio
* Goals
    1. Produce fixed-length values
    2. Postpone decompression during query execution ("late materialization")
    3. Lossless
* Granularity
    1. Block-level (tuples)
        * Typically olders systems or LSMs
    2. Tuple-level (entire tuple, NSM)
    3. Attribute-level (single attribute within a tuple, multiple attributes for same tuple)
    4. Column-level (DSM-only)
        * DSM / PAX / Parquet do this

* Naive Block Compression
    * use general purpose compression algo on each block (LZO, LZ4, Snappy, Zstd)
    * Computational overhead?
    * (de-)compression speed
    * InnoDB compression
        * Compressed page is [1,2,4,8] KB (padded)
        * mod log space at top of page
        * On write, can just put entry into mod log and not decompress entry
            * On update, have to decompress and modify
        * On read, read from log if available, otherwise decompress into standard 16kb page
        * Not a bad idea: more tuples for every IO op, but e.g. read followed by write can result in compression churn
    * DBMS must decompress data before it can read or modify it, doesn't consider _meaning_ of data

Ideally want DBMS to operate on compressed data w/o decompressing.

* Columnar Compression
    * Run-length encoding
        * Sorting column can help here
    * Bit-packing encoding
        * Reduce encoding size to fit range
        * Can use patching / mostly encoding to indicate values out of range
    * Bitmap encoding
        * Good for low cardinality attributes.
        * Compressed bitmaps good for sparse data, e.g. Roaring Bitmaps
    * Delta/Frame-of-reference encoding
        * Combine with RLE for even better compression
        * Can use global min value, start, or previous val as anchors
    * Incremental encoding
    * Dictionary encoding
        * Replace frequent values with integer codes
        * Most common
        * Can sort to preserve ordering
        * If we know it's ordered, can rewrite prefix queries in integer space

## Apache Pinot / StarTree
* OLAP increasing usage and adoption
    * Real time and offline servers
    * Segment store (NFS, HDFS, S3, ...)
    * Rest controllers for admin & broker
    * RT ingestion, offline ingestion from row stores
* Table split into segments of cols + indexes
    * Star-tree index built on multiple columns with pre-aggregated results
        * Tree structure with at most T records per leaf node
        * Star nodes contain pre-aggregated records after removing the dimension on which this level was split
        * [pinot star-tree docs](https://docs.pinot.apache.org/basics/indexing/star-tree-index)
    * Also supports inverted, text, and vector indexes
* Multi-stage Query Engine: in-memory system for decoupling data exchange and query engine layers

# [06 - Memory & Disk I/O Management](https://www.youtube.com/watch?v=aoewwZwVmv4&list=PLSE8ODhjZXjYDBpQnSymaectKjxCy6BYq&index=8)
Previously: How DBMS represents DB on disk
Now: How the DBMS manages its memory and moves data to/from disk

## Overview
* Spatial control: where to write on disk, keep things used together physically close
* Temporal control: when to read into memory, minimize number of stalls when reading

Memory houses buffer pool, with a bunch of frames ~ same size as disk page, execution engine queries memory mgmt system for necessary pages.
* DBMS needs more than just tuples and indexes, other pools may/may not be backed by disk
    * Sorting + Join Buffers
    * Query Caches
    * Maintenance Buffers
    * Log Buffers
    * Dictionary Caches

## Buffer Pool Manager
* Manages an array of fixed-size memory pages (frames)
    * Essentially a cache
* DBMS requests page => exact copy placed into frame from disk (if necessary)
* Dirty pages are buffered and not written immediately (write-back cache)
* Page table: map from page id to frame
    * Dirty flag, ref counter, access/audit tracking
    * Latches on frames
        * Lock
            * Protect DB logical contents btwn transactions
            * Held during transaction
            * Needs rollback
        * Latch / Mutex
            * Protect crit section of DBMS data structure (btwn threads)
            * Held during operation
            * No rollback
        * Lock nomenclature already used, higher level
        * Roll our own because don't want to use OS primitives
* Page directory maps page id to page location in file (on disk)
    * On disk for read on start
* Page table is ephemeral, page id to copy of a page in a frame

## Why MMAP will Murder your DBMS
Isn't this just virtual memory? Why not use OS?
* mmap stores contents of file in addr space of file, moves pages in/out so program doesn't worry about it
* what if DBMS allows multiple threads to access mmap files to hide page fault stalls?
    * Transaction safety: OS can flush dirty pages whenever, need atomic write across all affected pages
    * I/O stalls: DBMS doesn't know which pages are in memory, OS will stall while swaping
        * problematic for other threads (thrashing)
    * Error handing: difficult to validate pages, have to handle SIGBUS everywhere
    * Performance issues: data structure contention, TLB shootouts
* solutions to some of these:
    * madvise: tell OS how you'll read
    * mlock: don't page out memory ranges
    * msync: flush memory ranges to disk
    * Roughly as much work as building your own memory management.
* Might be OK in some cases, particularly RO like elasticsearch
    * LMDB has strong opinions to use mmap
* DBMS will almost always do a better job because it has more context than OS


## Buffer Replacment Policies
* Goals: correctness, accuracy, speed, metadata overhead
* Enterprise solutions have spent millions optimizing this, but they don't always disclose the details.
* Most common: LRU, timestamp of last access, LRU list in sorted order
    * Approximation with "clock", each page has a reference bit, set on access.
    * Clock "hand" sweeps circular buffer, sets to 0 or evicts.
    * Sequential flooding messes up LRU
        * Sequential scan reads every page one or more times => throws out other hot pages.
        * In OLAP, most recently used page is often best to evict.
        * Tracking how often a page is accesed would help
* LRU-K, track last K references to page, look at the longest interval between accesses
    * Balances frequency vs recency, maintain ephemeral cache for recently evicted pages.
* Localization: evict per-query
    * PG assigns limited number of pages to a query and uses as a ring buffer
* Priority hints: DB knows context of each page, can "light pin" it.
    * E.g. if inserting into right side of B+ tree, don't page out root and R child
* Dirty pages: drop if clean, otherwise flush to disk.
    * Tradeoff here, might prefer dropping clean pages.
    * Can mitigate with BG writing (periodically write dirty pages to disk)
        * NB: don't write page before its log record is written!

## Disk I/O Scheduling
OS & HW try to maximize disk bandwidth by rescheduling & batching IO
* But they don't know importance => many DBMSes recommend using deadline/noop/FIFO scheduler
DBMS can maintain its own prioritzation of IO requests
* Sequential vs random
* Critical path vs BG task
* Table vs index vs log vs ephemeral data
* Transaction metadata
* User SLAs
Some modern systems use `io_uring` for async IO (coroutines)

* OS page cache: most DBMSes use direct IO to bypass cache
    * Otherwise get redundant copies, differing eviction, loss of control over file IO
    * PG uses the OS Page Cache?!
        * Recommendation: only use 25% of available memory because OS page cache will need it.
        * Experimental feature disabling OS page cache; lots of focus on IO improvements recently.

Fsync problems
* What if DBMS calls fwrite? Copied to kernel, buffered and written to disk
* Okay, use fsync.
    * What if you get EIO?
    * Linux marks pages as clean, DBMS thinks write was successful.
        * (claim that this is due to disappearing disks, e.g. USB drive)
        * Was a bug for 20yrs in a bunch of DBs: fsync caused data loss.
* Many DBMS would call fsync in a loop, but can't guarantee it's successful

## Other Memory Pools
Buffer pool optimizations
* Multiple pools
    * per-database, per-page type
    * helps reduce latch contention and improve locality
    * How?
        A) Object ID: embed obj ID in record IDs, map to buffer pool instance
            * Each can have different eviction algorithms, different page sizes, etc
        B) Hash page id -> buffer pool
            * Note(me) consistent hashing might be interesting: dynamically size pools?
* Pre-Fetching
    * Sequential vs index scans
        * query plan helps these, index scans require DBMS knowledge (OS pre-fetching doesn't help)
* Scan Sharing (aka Synchronous Scans)
    * Attach multiple queries to a single cursor
    * May affect limit results if unordered: attach to running cursor at _some_ pos
* Buffer Pool Bypass
    * Sometimes won't store pages in buffer pool, use memory local to running query
    * "Light Scans" in Informix

## Project 1
Implement LRU-K, Disk Scheduling, Buffer Pool Manager; APIs are provided.
* LRUKReplacer should check "pinned" page status; return lowest page ID if none have been touched since last sweep.
* Scheduler: single queue, use std::promise, make it thread-safe. Can reorder/batch or whatever requests.
* Manager: Implement page guards, whatever data structure you want. Don't mess up operation order when pinning!

# [Lecture #7 Hash Tables, RelationalAI Talk](https://www.youtube.com/watch?v=VkP_yLM5KBU&list=PLSE8ODhjZXjYDBpQnSymaectKjxCy6BYq&index=9)
Starting to talk about supporting DBMS's execution engine, particularly RW from pages.
* Hash Tables (unordered)
* Trees (ordered)

## Background
Need to store internal metadata (e.g. pageId mappings), core data, temporary data, and indexes.
* Index should let us go from key -> record tuple
* Sometimes paying more compute (less efficient) makes sense to maximize disk IO
* Concurrency considerations: "physical" (clobbering threads to e.g. NPE) or logical (threads try to update same key)

Hash tables are useful to evenly (randomly) distribute data through a key space.
* Typically O(1) but DBMS care about constants (e.g. hash compute time)

Idea: Static hash table
* Giant array, one slot per element; `hash(key) % N -> value`
* Logically this works
* Bad idea because
    * Don't know the number of elemets ahead of time, it's dynamic
    * Keys aren't always unique (e.g. one:many relationships)
    * There is no perfect hash function across arbitrary key values.
## Hash Functions
Want to map large key space into smaller domain, speed vs collision tradeoffs
* Don't care about cryptographic properties (SHA-2, etc, generally too slow)
* Prior art
    * CRC-64 (network error detction), MurmurHash (fast, GP), CityHash, XXHash (SoTA, from zstd creator), FarmHash
    * smhasher on github, microbenchmarking of hash functions
## Static Hashing Schemes
* I.e. how we handle collisions. Tradeoff size vs additional instructions to get/put.
1. Linear Probe Hashing (Open Adressing)
* Not to be confused with Linear Hashing (dynamic scheme)
* Giant table of fixed-length slots
* Collisions: Linear search for next free slot
* Load factor determines when to resize
2. Cuckoo Hashing (Open Adressing)

Advanced topics:
* Robin Hood Hashing
* Hopscotch Hashing
* Swiss Tables

Linear Probe Hashing
* Hash, on conflict keep searching down until you find a slot
* Deletes are challenging: What if you delete the first entry in the conflict "linked list" of slots?
    * Can re-hash and shift all the following keys, but this is expensive.
    * Tombstones: can reuse the slot for new keys, might need GC
* Non-unique keys
    1. Separate linked list, value is pointer to another value list. Can overflow to multiple pages.
    2. Redundant keys, store duplicate key entries together: insert does scan as normal.

Optimizations:
* Specialized hash table implementations based on key types (e.g. different string sizes)
* Metadata in separate array (e.g. packed bitmap tracks empty/tombstone)
* Table + slot versioning metadata to quickly invalidate entries


Cuckoo Hashing
* Use multiple hash functions to insert
    * If none avaliable, evict another element and rehash it
    * Lookup/delete is O(1) because one location per table is checked
    * Inserts can be pathological.
        * Have to detect infinite loops, e.g tracking start slot.

## Dynamic Hashing Schemes
Chained Hashing (lots of impls)
* LL of buckets for each slot, insert collisions in list.
* Can optimize reads with key bloom filter per-bucket

Extendible Hashing (GDBM and Asterix, couchbase)
* Chained hash buckets can grow infinitely, problematic with skewed data
* Reshuffle bucket entries on split, increase bits to examine
* Global and local "bit counters", look at these to determine bucket chain.
* To rebalance, increase global counter and expand bucket LUT, add new bucket entry.
    * Multiple global LUT entries can point to the same chain, depending on skew

Linear Hashing (PG uses this as "dynahash" or similar)
* Hash Talble holds pointer tracking next bucket to split
    * Split when _any_ bucket overflows
    * Kinda like random splitting and amortizing cost.
* Use multiple hash functions.
    * When splitting update hash functions and rebalance split bucket.
    * Split pointer increments downwards, so know that new hash function only applies to previously split buckets.
* Deletes: if bucket is empty and below split pointer, can throw away and move pointer up


## Conclusion
Hash tables great, used everywhere
* Not typically for indexes as they require complete equality (no partial lookups)
* Usually use B+ Trees for these instead.


## Next time
"Order-Preserving Indexes featuring B+ Trees"
* Greatest data structure of all time! Tries are good, but can put inside a B+ Tree!

## RelationalAI Talk

# [08 - Tree Indexes: B+ Trees](https://www.youtube.com/watch?v=scUtG_6M_lU&list=PLSE8ODhjZXjYDBpQnSymaectKjxCy6BYq&index=10)
## Index vs Filters
* Index is a subset of attributes organized or sorted to location of specific tuples (e.g. B+ tree)
* Filter answers set membership queries, e.g. BLoom filter
    * Usually used internally, not by end user
* Keeping these synchronized and thread safe are future topics

## B-Tree Fam
There is a specific B-Tree structure, but most people mean B+Tree.
* B-Tree was 1970, B+ 1973, 1979 "Ubiquitous B-Tree", variants: Blink, Be, Bw Trees

## B+Tree
* Self-balancing, ordered m-way tree for searches, sequential access, insertions, and deletions in O(log_m(n)) time.
  * Every node other than root is at least half-full
  * Inner node with k keys has k+1 non-null children
  * Optimized for RW large data blocks
* Nodes have m key values, each child owns range from key i to i+1
  * B-link tree added sibling pointers, allows concurrent access for merges
  * k/v pairs, usually in key order, usually pointers to external data.
* Generally traversing top-bottom, so deadlocks are easier to avoid
    * Sibling pointers complicate this, some systems only use singly linked lists for siblings
### Leaf Nodes
* Can store key/value pairs contiguously
* Or store sorted keys and values separately, easier to do binary search

### Leaf node values
1. Record IDs (pointer to tuple, most common)
    * PG, SQL Server, DB2, Oracle
2. Tuple Data (index-organized storage)
    * Primary key: leaves store contents of tuple
    * Secodndary index: leaves store tuples primary key as value
        * Leaves move around, so store primary key here for secondary lookup
    * SQLite, SQLServer, MySQL, Oracle

### B-Tree vs B+Tree
* B-tree stores keys and values in all nodes in the tree, each key only occurrs once
* B+Tree stores values only in leaves, inner nodes are only for search.
  * Scan across bottom to find all values: don't have to go back up for sequential access.

## B+Tree operations
### Insert
* Find appropriate leaf node, insert if space.
    * Otherwise split (half) and propagate to inner nodes up the tree.
### Delete
* Find appropriate leaf, remove, if less than half full, steal from sibling.
    * Merge with sibling if both are less than half full.
        * Implementations differ in steal/merge direction preferences (left/right)
    * Propagate up the tree.
* Merges necessary to ensure `log(N)` lookup


## Composite Index
e.g. index on `(a, b, c)`
* DBMS can use B+Tree index if we have a prefix

## Duplicate keys
* Can append the record id or extend to overflow bucket (just append the id)

## Clustered indexes
* Store in sort order by primary key is fine, only retrieve once
* Index scan page sorting (non-clustered index)
    * potentially get redundant reads based on buffer pool size
    * optimization: fetch desired tuples (no pages), sort them based on pageID
        * PG calls this "bitmap heap scan"
        * can further refine to do intersections of pages if WHERE clauses are present


## B+Tree Design Choices
Node size
* Slower storage wants larger node sizes (1MB HDD => 512B RAM)
* Depends on workload (leaf node scans vs root-to-leaf traversals)

Merge Threshold
* Avg occupancy rate for B+ tree nodes is 69%
* Delaying merge operation may reduce reorganization work
    * PG calls it "non-balanced" B+Tree (nbtree)

Variable Length Keys
1. Pointers
    * Store keys as pointers to tuple attribute ("T-Trees")
    * Fine for e.g. 1GB keys, adds another pointer deref
2. Variable-Length Nodes
    * Size of each node index can vary, needs careful mem mgmt
    * Typically only academic systems
3. Padding
    * Pad to max length
4. Key Map / Indirection
    * Embed array of pointers that map to key+value within a node
    * Most common approach.

Intra-Node Search
1. Linear: Scan node keys, SIMD to vectorize comparisons
2. Binary
3. Interpolation: approximate based on known distribution
    * Academic uses only

Prefix Compression
* Keys in leaf node are likely to share prefix

Deduplication
* Shared Key, "posting list" of tuple values

Suffix Truncation
* Inner nodes only used to direct traffic => don't need the entire key (only prefix)
* Works because B+ tree doesn't store kv in inner nodes
* May require updating based on usage

Pointer Swizzling
* If page pinned in buffer pool, can store raw pointers instead of page IDs
    * Works for hot pages, sidesteps buffer manager

Bulk Inserts
* Sort keys, build index bottom-up

Write-optimized B+ Tree / "Fractal Trees" / B(epsilon) trees
* Modifying tree is expensive with splitting/merging nodes, can we batch updates?
* Mod log in front of every node, just append to log.
    * e.g. delete 10 goes in root log, finds will read this first
    * Push down when running out of room

Conclusion
* B+ Trees are almost always the right choice
    * Skip lists and tries have some uses


