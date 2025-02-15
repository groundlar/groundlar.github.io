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
# (06 - Memory & Disk I/O Management)[https://www.youtube.com/watch?v=aoewwZwVmv4&list=PLSE8ODhjZXjYDBpQnSymaectKjxCy6BYq&index=8]
