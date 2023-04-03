---
layout: post
title:  "category theory for programmers"
date:   2023-03-29 21:00:00 -0700
categories: jekyll default
---

# TODO: fix LaTeX below, maybe via https://stackoverflow.com/questions/26275645/how-to-support-latex-in-github-pages

# [Part 1.1 -- Motivation and philosophy](https://www.youtube.com/watch?v=I8LbkfSSR58&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_)
  - Philosophy. Interesting, but not really relevant.
  - OOP abstracts the wrong things: hides state and mutation, so you get data races for free!
  - Category theory has informed tons of libraries (C++ to Haskell)
  - Idea that humans can only perceive and reason about decomposable systems, posit that the universe is not like this.

# [Part 1.2 -- What is a category?](https://www.youtube.com/watch?v=p54Hd7AmVFU&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_&index=2)
### Abstraction, Composability, Identity
  - Category is a bunch of objects, can think of it as a more general set or a (potentially infinite) graph
    - Objects and Morphisms are primitives
    - Morphisms (analogous to functions) go from one object to another
      - Morphisms must be composable: if $a \xrightarrow{\text{f}} b \xrightarrow{\text{g}} c$ then there must exist $a \xrightarrow{g \circ f} c$ called "g after f"
    - A category is a "multiplication table" defining, for every two morphisms ("arrows"), what their composition is.

### Laws
Composibility 

- (see above)

Identity
  
  - There must be identity morphisms, these have left and right identities.
    - $id_a \circ f = f \iff g \circ id_a = g$

Associativity
  
  - if $a \xrightarrow{\text{f}} b \xrightarrow{\text{g}} c \xrightarrow{\text{h}} d$ then $a \xrightarrow{h \circ ( g \circ f )} c$ and $a \xrightarrow{( h \circ g ) \circ f} c$ are identical, called $h \circ g \circ f$
    - `a -f-> b -g-> c -h-> d` then `h o (g o f) :: a -> c` and `(h o g) o f :: a -> c` are identical

### Questions/Asides
  - Categories that form sets are called "small" categories, otherwise "large" categories.
  - Morphisms between any two objects form a set.
    - Unless they're higher-order categories, in which morphisms form objects in a category.
  - There are some similarities to Groups, but mostly to Monoids
    - A Group is a Monoid that has an inverse.
    - If you define an inverse for each arrow in a Category, you have a Groupoid (category where every morphism has an inverse), "more than a group"
    - A Group is a Category with only one object.
    - In a Group you can compose anything with anything, not so for a Category
      - Morphisms can be composed only if the start of one morphism is the end of the other.

### Analogy to Programming
  - Objects are types, morphisms are functions!
  - Simplest model for types is they're sets of values
    - Model programming as a "category of sets"
  - "Functions" are mathematical functions, which take a value from one set and returns a value from another set.
    - One function corresponds to one morphism


#### The Set Category
Schizophrenic categories: lots of categories come from other models
  - One we'll reference a lot comes from Set Theory: the objects in a category are sets
    - Typical set definition--they have structure and functions map elements to elements
    - Building a category on top of this requires that we abstract/forget about the structure within objects
    - Every set has a trivial id function mapping itself to itself
      - There are many was to map a set to itself (e.g. exchanging elements), only one of these is the id morphism
    - Number of functions between sets defines the number of morphisms
      - Composition follows as you'd expect
    - "Multiplication"/Composition table from the above then fulfills the axioms of category theory
    - Can derive a lot of information from category theory that we "forgot" in our abstraction step.
      - Analogous to focusing on the interfaces between objects / types (the morphisms!)
      - Call this "data hiding" or "abstraction" if you like, category theory is the most abstract language we can think of.



# [Part 2.1 -- Functions, epimorphisms](https://www.youtube.com/watch?v=O2lZkr-aAqk&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_&index=3)
Programming Semantics: Operational (how expressions can be transformed into others, Denotational (map to another concept, e.g. mathematics)

- In this course, "type" and "set" are usually semantically equivalent

Morphisms / functions here are "pure total functions"

- Total: Defined for all inputs of their types
- Pure: same args, same result (can be memoized given infite resources)
  - Can eventually describe operations with side effects (like IO) by building upon pure functions

Review: Set Category is sets and functions between sets
Functions in math are special kinds of relations.
In set theory, a Relation is a subset of pairs of elements between sets.
A set of pairs is called a Cartesian product, the cartesian product of S1 and S2 is the set of all pairs.


- Any subset of the Cartesian product is a relation, no inherent directionality.

Functions have directionality, they are limited in that one argument cannot be mapped to multiple different values.

Nomenclature: function f has a domain ("starting set") and a codomain ("ending set"), the image is the subset of the codomain obtained by mapping all of the domain.

Easy way to understand directionality is to ask "is this function invertible?" Usually it's not (inverse is unclear).

$f :: a \xrightarrow b$ is a function from type $a$ to type $b$, which is invertible if there exists $g :: b \xrightarrow a$, such that $g \circ f = id_a$ and $f \circ g = id_b$.
An invertible function is called an Isomorphism.

Isomorphisms identify sets and say "there's a 1-to-1 mapping between these sets". For finite sets this is an identification of elements (1:1 correspondence). 

More complicated for infinite sets, e.g. say $y = 2x$ where domain is even numbers and codomain is integers.
Isomorphisms kinda say "these sets are, for some purposes, identical".

Two reasons a function might not be an isomorphism:

- Collapses elements of a set (many in domain map to single in codomain)
  - Note: A "fiber" is the set of points in the domain mapped to the same point in the codomain
    - Construct a set of fibers and the function is invertible on that set
    - Corresponds to the notion of abstraction: throwing away information.
      - E.g. throw away exact number input, only care if it was even or odd.
- The image doesn't fill the codomain (points in codomain which don't correspond to anything in the domain)
  - Corresponds to the notion of modeling: recognizing pattern defined by first set in the second set. 
    - "injecting" or "embedding" the domain into the codomain.

Mathematicians define these differently:

- If a function _does not collapse things_, it's "injective"
  - "monic" or "monomorphism" in category theory
- If a function covers the entire codomain, it's "surjective" ("sur" means "on" in French or Latin, idk)
  -  "epic" or "epimorphism" in category theory
- Surjective & injective => isomorphism (invertible/bijective)


How do we look at surjections and injections in category theory?

- Can only use morphisms, have to look at all other objects in category to define the property.
- Epimorphims and monomorphisms are much more general than surjective and injective functions
- Consider $f :: a \xrightarrow b$ and $g :: b \xrightarrow c$
  - then $g \circ f$ does not act on the entirety of object b, only on the subset defined by set theory.
  - define functions/morphisms $g_1$ and $g_2$  which only differ in their mappings for points outside the image of f in b
  - Compositions are equal $g_1 \circ f = g_2 \circ f$ ONLY due to the composition domain limitation
  - Epimorphism/Surjective definition: $\forall c \forall g_1, g_2 :: b \xrightarrow c \mid g_1 \circ f = g_2 \circ f \implies g_1 = g_2$
  - Can sort of "cancel" $f$ when it's an epimorphism: $g_1 \circ \cancel{f} = g_2 \circ \cancel{f}$



# [Part 2.2 -- Monomorphisms, simple types](https://www.youtube.com/watch?v=NcT7CGPICzo&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_&index=4)

## Monomorphisms
What isn't a monomorphism? A funciton that maps two $x_1, x_2 \in a$ to the same $y \in b$
Defining this in category theory is similar to epimorphisms, but _pre_-composition instead of _post_-composition.

Say $\exists g_1, g_2:: c \xrightarrow a$ which map a point $z \in c$ to $x_1 and x_2$ from above.
Then the compositions $f \circ g_1$ and $f \circ g_2$ does not hold for all $c$.
Thus, monomorphism if this holds for all c.
$\forall c \forall g_1, g_2 :: c \xrightarrow a$
$f \circ g_1 = f \circ g_2 \implies g_1 = g_2$

Note that we had to define epimorphisms and monomorphisms using the entire universe.
They're defined for every category, though they may not exist.

In set theory, injective + surjetive = isomorphism (invertible)
Not in category theory: epi + mono != isomorphism

**TODO Questions: the diagrams for epi- & monomorphisms all use sets as objects.
- What does this mean for categories, where we can't look inside the objects?
- Or does this trivially hold for some types of categories (e.g. where the objects aren't sets)?**

## Sets
Let's talk about sets, our model for types.
Empty set: is this a type in programming?
  
  - Not in imperative languages, in Haskell this is `void`.
  - Caveat: Haskell types are funny due to "bottom element," which helps with the halting problem.
  - No way to construct void objects.
  - Are there functions that take void? e.g. `f:: void -> int`
    - Mathematically yes. But it's a bluff, you can't produce an element of type void to invoke it.
    - So we can assume $id_\text{void} :: \text{void} \xrightarrow \text{void}$
    - Haskell calls the void -> int function `absurd`
    - This makes sense when equating logic and programming: `void` equates to falsehood, a function is a proof of a proposition.
      - From false premises you can define anything $\implies$ `absurd :: Void -> a`, where `a` is anything
Singleton set: `Unit () :: ()` in Haskell
  - Notation: expressions, terms, and types have different namespaces in Haskell so `()` is reused.
  - Has only one element, itself.
  - Corresponds to `true`, can always produce true (tautologically)
  - Does `unit :: a -> ()` exist? Sure, disregard the argument, create `Unit`
  - What about `f :: () -> int`? Sure, returns a constant (e.g. `one :: () -> int`
    - Holds for any type, e.g. two for `boolean`, ~4B for 32-bit integer
    - This gives us a sneaky way of defining sets: a family of morphisms from unit defines the elements of a set.
      - Each morphism "picks" an element
  - Later we'll generalize singleton set/type, and call these generalized elements.
  - Unit type doesn't exist in every category, but in the category of sets it's the singleton.
Two element type: `Bool`
  - Bool is not an atomic construction in our category of sets, it's the sum of two units.
  - Functions that return `Bool` are called predicates, e.g. `isDigit`



# [Part 3.1 -- Examples of categories, orders, monoids](https://www.youtube.com/watch?v=aZjhqkD6k6w&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_&index=5)
Last time talked about sets as sets of elements, tried to reformulate properties of sets in terms of morphisms/functions between sets.
This makes it easier to generalize to arbitrary categories.

What other categories might be interesting?

## Simple categories
What's the simplest possible category? Zero objects?

-  If there are no objects, there are no arrows (morphisms).
- Are the axioms satisfied?
  - Sure.
  - Associativity "for any pair", there are no pairs.
  - Identity "for every object", there are no objects.
- Category "Zero" $\null$. Useful in the context of categories of categories.

Category with one object

- Has to have at least one arrow: identity

Two objects? At least two identity arrows, arbitrary arrows between the two objects.

## Graphs and categories
We can keep adding arrows to graphs to make it a category.
Firstly, add identity on each node.
Composition?

- For every pair $f, g$ of composable arrows, add another $g \circ f$.
- Quickly grows as the compositions must be composed as well.
Associativity?
- Provides a "slowdown" of growth because certain compositions must give the same result.
This is called "free" construction in category theory.
- Only constraints are those required by definition of a category.


## Orders
Here arrows are not functions, they're relations.
An interesting relation is `<=` or $\leq$.

- Draw arrow between `a` and `b` if `a <= b`, `a` comes before `b` in some sense.
Other conditions must be fulfilled to be an order.
- Different types, E.g. preorder, partial order, total order
- Total ordering is sorting.
"Preorder" is something simpler.
- A relation that satisfies a minimum of conditions: composibility.
- Composibility: e.g. `a <= b <= c` implies `a <= c`
- Associativity: there is at most one arrow between objects, so trivially associative.
- Identity: `a <= a` yes, called "reflexivity`
- In a total ordering, there is an arrow between any two objects.
- This is called a *thin category*, there is a 1:1 correspondence between thin categories and preorders
	- Have to "add conditions on top" of a preorder to get other orders

Set of arrows between any two objects has a name: "hom-set". 
The hom-set from a to b is written `C(a,b)`

- A thin category is a category where every hom-set is empty or a singleton set.

Next is a "partial order"

- No loops between elements
- **TODO wtf, didn't we just said Preorers have at most one arrow, so they're thin sets? What is a preorder then?**
	- Maybe the above is a poor example, with strictly `<=` we'd get total ordering? 
- Correspond to DAGs.
- Some may not be comparable.

Total Orders have arrows between all objects (everything is comparable)

## Invertibility in thin categories
Epimorphism + monomorphism `!=` isomorphism

- Recall epimorphism is "post-composition" restriction (`f:: a -> b` and `g_1, g_2 :: b -> c`)
- Recall monomorphism is "pre-composition" restriction(`h_1, h_2 :: d -> a` and `f:: a -> b`)
In a thin category, both are trivially satisfied due to limitation on hom-set.
In a partial order, there cannot be reverse arrows, so these aren't invertible.

### Thick categories
Thick categories can have "loops", hom-sets can have multiple elements, somehow more general than orders.
Can think of the most general category as a "preorder which is thick"

- Orders are relations, true or false. Arrows signify the relation. Things are binary.
- Thick categories might say "no arrows mean no relation" but "some number of arrows in `C(a,b)` signifies a relation"
- Each "arrow" represents a different proof of the relation.
- A thin category defines relations, we may not know what the relations are.
	- Note about homotopic type theory using this: not enough to show a relation, but there are different, non-equivalent ways of showing relations.

## Return of the One-Object Category
Call the object `m`. As long as we have an identity relation, we can have as many other morphisms as we like in `C(m, m)`. Automatically composable.

This is called a `monoid`, monoid meaning single.
These crop up in group theory as well, as sort of "pre-groups".

### In Set Theory
Set notion of monoid predates category theory definition.

`monoid` is a binary operator, e.g. multiplication.
The monoid identity operator is `unit`, e.g. 1 in multiplication.

Associativity: `a * b * c` no matter the order.

Associativity and Identity:

- $\forall e \exists a, e * a = a * e = a$
- if it has a unit and it's associative, it's a monoid.
	- e.g. numbers with multiplication (unit 1), numbers with addition (unit 0), string or list concatenation (unit `""`)
		- Concatenation is interesting in that it's not symmetric (unlike multiplication and addition)

### Ok, back to categories
Does this correspond to set defn? (we sure hope so)

hom-set `M(m,m)` is the set of arrows starting and ending at `m`, it's reflexive. This hom-set _is a set_ of morphisms ("arrows"). If we take any two elements of this set, there is a third element corresponding to composition of the arrows. So can define multiplication as composition of two arrows (third arrow is the product of the two elements). The `id` element is of course the unit (associative composition with any element).

So we can freely translate between the category and set theory definitions (1:1 corresopndence).

#### Now programming
The set category (category of types) corresponds to a strongly typed system.
You can't compose any two functions, only if their types match.

Monoid is special in that any two functions are composable. This corresponds to weak typing (you just might get some fun errors). In this sense, there's no difference between the type systems, just the category you've chosen to operate in.



# [Part 3.2 -- Kleisli category](https://www.youtube.com/watch?v=i9CU4CuHADQ&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_&index=6)
