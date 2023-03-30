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
Identity
  - There must be identity morphisms, these have left and right identities.
    - $id_a \circ f = f \iff g \circ id_a = g$
Associativity
  - if $a \xrightarrow{\text{f}} b \xrightarrow{\text{g}} c \xrightarrow{\text{h}} d$ then $a \xrightarrow{h \circ ( g \circ f )} c$ and $a \xrightarrow{( h \circ g ) \circ f} c$ are identical, called $h \circ g \circ f$

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

Easy way to understand directionality is to ask "is this function invertible?" Usually it's not (inverse unclear).

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
  - but $g_1 \circ f = g_2 \circ f$ due to the composition domain limitation
  - Epimorphism/Surjective definition: $\forall c \forall g_1, g_2 :: b \xrightarrow c \mid g_1 \circ f = g_2 \circ f \implies g_1 = g_2$
  - Can sort of "cancel" $f$ when it's epic: $g_1 \circ \cancel{f} = g_2 \circ \cancel{f}$



# [Part 2.2 -- Monomorphisms, simple types](https://www.youtube.com/watch?v=NcT7CGPICzo&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_&index=4)


# [Part 3.1 -- Examples of categories, orders, monoids](https://www.youtube.com/watch?v=aZjhqkD6k6w&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_&index=5)

# [Part 3.2 -- Kleisli category](https://www.youtube.com/watch?v=i9CU4CuHADQ&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_&index=6)
