---
title: Formula Transformations
---

A formula transformation takes a formula as input and returns another formula. A formula transformation is a special case of a [Formula Function](../formula-functions) but always returns a formula. We distinguish the following kinds of formula transformations:

- [Normal Form Transformations](normal-form-transformations), e.g. conjunctive normal form (CNF) or disjunction normal form (DNF)
- [Simplifier Transformations](simplifier-transformations), which try to minimize or simplify the formula
- [Other Transformations](other-transformations) for other useful transformations such as anonymizing a formula

Analogously to the [formula functions](../formula-functions) and the [formula predicates](../formula-predicates), formula transformations can be executed on formulas with the `.transform()` method. In the following example the `DistributiveSimplifier` is executed on the formula `f1` with activated caching:

``` java
Formula result = f1.transform(new DistributiveSimplifier());
```

Caching can also be deactivated by calling the overloaded method with flag `false`:

``` java
Formula result = f1.transform(new DistributiveSimplifier(), false);
```

The last parameter of the `.transfom()` method indicates whether the result of the transformation should be cached in the formula. If you have an expensive computation which is likely to be executed more than once on formulas, it is a good idea to cache the result.

The result of any transformation in this chapter is another formula. Depending on the transformation the resulting formula will be *equisatisfiable*, *equivalent* or neither of both compared to the input formula.

!!! info "Equivalence vs Equisatisfiability"

    Two formulas `f1` and `f2` are (logically) *equivalent* if both formulas evaluate to the same truth value *for all* assignments. With `eval(f1, a)` we denote the truth value of the formula `f1` when evaluated under assignment `a`. Formulas `f1` and `f2` are equivalent, if for *every* assignment `a` holds:

    ```
    eval(f1, a) = eval(f2, a)
    ```

    In contrast, two formulas `f1` and `f2` are *equisatisfiable* if either both are *satisfiable* or both are *unsatisfiable*, i.e. if the following holds:

    ```
    (f1 is satisfiable) <=> (f2 is satisfiable)
    ```

    Note that *equisatisfiability* is implied by *equivalence* but not the other way round.

    ```
    (f1 and f2 are equivalent) => (f1 and f2 are equisatisfiable)
    ```

    Consider the example:

    ``` java
    Formula f1 = f.parse("A | B");
    Formula f2 = f.parse("A | C) & (B | ~C");
    ```

    The formulas are both satisfiable, i.e. they are equisatisfiable. But they are not equivalent. For example, the assignment `a = {~A, B, ~C}` yields `eval(f1, a) = true`, but `eval(f2, a) = false`.

    For more information check out [Equisatisfiability in Wikipedia](https://en.wikipedia.org/wiki/Equisatisfiability).

