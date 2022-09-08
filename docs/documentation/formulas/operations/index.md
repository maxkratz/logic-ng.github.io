---
title: Formula Operations
---

As mentioned in the [Formulas](..) chapter, there are three different kinds of operations on formulas which we will explain in detail in the following three chapter:

1. [Functions](formula-functions):  A function takes a formula as input and computes some value on that formula.  This value can be a simple integer e.g. the depth of a formula, or a more complex result type, like the list of sub-formulas.  Functions are applied on a formula with the `.apply()` method.
2. [Predicates](formula-predicates): A predicate takes a formula as input and computes a truth value on that formula, e.g. whether a formula is in a certain normal form like NNF, CNF, or DNF, or if it is satisfiable?  Predicates are evaluated on a formula with the `.holds()` method.
3. [Transformations](transformations): A transformation takes a formula as input and returns another formula, thus transforming the input formula.  Examples for transformations are normal form conversions like NNF, CNF, DNF, or formula simplification.  Transformations are executed on a formula with the `.transform()` method.

