---
title: Formula Functions
---

A formula function takes a formula as input and computes some value on that formula. The result of the computation can be saved in the formulas cache.  There are some functions implemented in LogicNG or the user can implement her own functions, as shown in the basic chapter on [Formulas](../..).

Formula functions can be executed on formulas with the `.apply()` method.  In the following example the [LiteralProfileFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/functions/LiteralProfileFunction.java) is executed on the formula `f1` with activated caching:

``` java
Map<Literal, Integer> result = f1.apply(new LiteralProfileFunction());
```

or, if one wants to control the cache-strategy manually:

``` java
Map<Literal, Integer> result = f1.apply(new LiteralProfileFunction(), false);
```

The last parameter of the `apply` method indicates whether the result of the function should be cached in the formula. If you have an expensive computation which is likely to be executed more than once on formulas, it is a good idea to cache the result.

In LogicNG there are some useful helper functions.


## Compute the Variables and Literals of a Formula

The [VariablesFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/functions/VariablesFunction.java) and [LiteralsFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/functions/LiteralsFunction.java) compute all variables, respectively literals, occurring in a given formula and returns them in a sorted set.  Since these functions are used very frequently, there are two shortcuts implemented directly on the `Formula` class: `.variables()` and `.literals`.

The sorting order for variables is the default Java sorting order on the variable's name as `String`.  The sorting order on literals is by name first and positive literals before negative literals.

Here is a small example for the usage of the functions:

``` java
Formula f1 = p.parse("A & ~B => A | B | C");

// computes the set [A, B, C]
SortedSet<Variable> variables1 = f1.variables();
SortedSet<Variable> variables2 = f1.apply(VariablesFunction.get());

// computes the set [A, B, ~B, C]
SortedSet<Literal> literals1 = f1.literals();
SortedSet<Literal> literals2 = f1.apply(LiteralsFunction.get());
```


## Compute the Number of Occurrences of Variables and Literals in Formulas

The [VariableProfileFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/functions/VariableProfileFunction.java) and [LiteralProfileFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/functions/LiteralProfileFunction.java) count the number of occurrences for each variable (respectively, literal) and return it in a map.  The map contains the variable or literal as key and the number of occurrences as value.  E.g.

``` java
Formula f1 = p.parse("A & ~B => A | B | C & (A => (B | C))");

// computes the map {A=3, C=2, B=3}
Map<Variable, Integer> variableMap = f1.apply(new VariableProfileFunction());

// computes the map {A=3, C=2, ~B=1, B=2}
Map<Literal, Integer> literalMap = f1.apply(new LiteralProfileFunction());
```


## Compute the Number of Nodes and Atoms of a Formula

The [NumberOfAtomsFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/functions/NumberOfAtomsFunction.java) and [NumberOfNodesFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/functions/NumberOfNodesFunction.java) compute the number of atoms, respectively nodes of a formula.  An atom is a Boolean constant or variable.  Again, there are shortcuts in the `Formula` class `.numberOfAtoms()` and `.numberOfNodes()`, respectively. Also see [the number of atoms](../..#the-number-of-atoms) and [the number of nodes](../..#the-number-of-nodes-and-internal-nodes) in the chapter on formulas.

Let's consider

```java
Formula f1 = p.parse("A & B & (A | B) <=> C & (A | B)");
```

Using `#!java f1.numberOfAtoms()` we find that the number of atoms is 7, as the atoms are

- `A` (3x)
- `B` (3x)
- `C`

Using `#!java f1.numberOfNodes()` we find that the number of nodes is 12. A detailed example can be found in the [relevant section](../..#the-number-of-nodes-and-internal-nodes) in the chapter on formulas.


## Compute the Depth of a Formula's AST

The [FormulaDepthFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/functions/FormulaDepthFunction.java) returns the depth of a function's abstract syntax tree. The depth of a function indicates how many levels of *nested* sub-formulas a formula has. For example,

- `A` has depth zero,
- `A & B` has depth one,
- `(A & B) | C` has depth two, and
- `(A & B) | C & (E | F)` has depth three.

Intuitively speaking, if you think of the [tree](../..#the-number-of-nodes-and-internal-nodes) of `f1` in the preceding chapter, the formula depth is the maximal depth of the formula's abstract syntax tree.


## Compute all Sub-Formulas of a Formula

The [SubNodeFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/functions/SubNodeFunction.java) computes the set of all sub-formulas of a given formula.  For example, applied on the function `A & B | C` the sub-formulas are

- `A`
- `B`
- `C`
- `A & B`
- `A & B | C`

Since the result is a set of sub-formulas each sub-formula occurs only once in the result. The sub-formula function is implemented in such a way, that the order of the sub-formulas in the result is bottom-up, i.e. a sub-formula only appears in the result when all of its sub-formulas are already listed.  The formula itself is always the last element in the result.

```java
Formula f1 = p.parse("A & ~B => A | B | C");

// Computes the sub-formulas
// A
// ~B
// A & ~B
// B
// C
// A | B | C
// A & ~B => A | B | C]
LinkedHashSet<Formula> subFormulas = f1.apply(new SubNodeFunction());
```


## Compute the Minimum Prime Implicant of a Formula

The [MinimumPrimeImplicantFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/functions/MinimumPrimeImplicantFunction.java) computes a minimum-size prime implicant for a given formula.  In order to understand what a minimum-size prime implicant is, let's understand what an *implicant* of a formula is first.

Consider the formula `f1 = A & B | B & C | D`. An implicant `f2` of `f1` is any min-term – a conjunction of literals – such that `f2` logically implies `f1`.

Some example implicants of `f1` are:

- `A & B`
- `A & B & C`
- `B & C & D`
- `B & C`
- `D`

A *prime implicant* is an implicant which cannot be further reduced (i.e. literals being removed) such that the reduced term yields an implicant. In this example that is:

- The implicants `A & B`, `B & C` and `D` cannot be reduced without violating the implicant property: Thus, they are *prime implicants*
- However, implicant `A & B & C` can be reduced: Removing `C` yields `A & B`, which is still an implicant. The same holds for `B & C & D`. These are *not prime implicants*

!!! info "Prime Implicants"
    Note that a formula can have prime implicants of different sizes.  A prime implicant is not globally minimal in the number of literals.

Another way to think about prime implicants is that a prime implicant is an implicant for which none of its proper subsets is itself an implicant. For more information about (prime) implicants check out [Wikipedia](https://en.wikipedia.org/wiki/Implicant).

A *minimum-size prime implicant* is a prime implicant with minimum size, in terms of the number of literals, among all prime implicants of a formula.  The `MinimumPrimeImplicantFunction` computes a prime implicant with minimum size.  A minimum-size prime implicant in our example is `D`.  Beware, in general there are more than one minimum-size prime implicants.  In this case, the formula returns the one found first.

Prime implicants of a formula play a key role in the
[Quine-McCluskey algorithm](../transformations/simplifier-transformations#quinemccluskey-algorithm), which is implemented in two different ways in LogicNG.

An example for applying the function is:

```java
Formula f1 = p.parse("(A | B) & (A | C ) & (C | D) & (B | ~D)");

// Computes [B, C]
SortedSet<Literal> minimumPrimeImplicant =
        f1.apply(MinimumPrimeImplicantFunction.get());
```

