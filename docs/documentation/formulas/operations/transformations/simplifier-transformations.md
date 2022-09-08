---
title: Simplifier Transformations
---

First we discuss simplifiers which can be applied to *all* formulas, second we discuss simplifiers for special formula types (CNF, DNF, and pseudo-Boolean constraints) and we close with those transformations which perform proper algorithms for simplification.


## Simplicity of Formulas and the Rating Function

The idea of the simplifiers in this chapter is to *simplify* a given formula.  But what is "simple" in terms of a formula?  Since "simple" is no mathematically defined term and can alter from application to application, some simplifiers let the user provide their own definiton of "simple".  This is done via a rating function.

A [rating function](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/simplification/RatingFunction.java) is an interface which can be implemented by the user and computes a simplicity rating for a given formula.  This could be for example the length of its string representation or the number of atoms.  This rating function is then used to compare two formulas during the simplification process and thus deciding which of the formulas is the "simpler" one.  There is a [default rating function](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/simplification/DefaultRatingFunction.java) which is a rating function which compares formulas based on the length of their string representation (using the [default string representation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/formulas/printer/DefaultStringRepresentation.java)).

The use of rating functions in the formula simplifiers works the following way: After a formula has been simplified with the relevant simplification (e.g. for the `FactorOutSimplifier` it is "after the formula has been factored out"), it is checked with the rating function whether the transformed formula is actually a simplification with respect to the specified criterion or not. *Only if the transformed formula is simpler in terms of the rating function, the transformed formula is being returned, otherwise the initial formula is being returned.*

Therefore, formula transformations which use a rating function are quite flexible: One can implement application-specific rating functions and use these for the simplifier. Both the `FactorOutSimplifier` and the `AdvancedSimplifier` (see below) use rating functions.


## Simplifiers for Arbitrary Formulas

### Factor Out Simplifier

The [FactorOutSimplifier](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/simplification/FactorOutSimplifier.java) transforms the given formula to one where variables are factored out. That is, if multiple terms in the formula share a common factor, this factor is being taken out of the terms and the terms are summarized. The simplifier works by applying the Distributive Law heuristically for a smaller formula.

For example, given the formula `A & B & C | A & D`, both conjunction terms have the common factor `A`. Thus, the simplification that the `FactorOutSimplifier` yields is `A & (B & C | D)`.

The `FactorOutSimplifier` can be used with and without a rating function. If the rating function is not specified, the `DefaultRatingFunction` is chosen. Thus, the following constructor creates a `FactorOutSimplifier` with the `DefaultRatingFunction`:

``` java
FactorOutSimplifier simplifier = new FactorOutSimplifier();
```


#### Basic Version of the Factor Out Simplifier: The Distributive Simplifier

The [DistributiveSimplifier](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/simplification/DistributiveSimplifier.java) is an old and simpler version of the `FactorOutSimplifier`.

The differences are the following:

- `FactorOutSimplifier` factors out repetitively, `DistributiveSimplifier` does not.  For example, consider

  ```
  (A | B) & (A | C & E) | B & C & D
  ```

  `A` can be factored out, leading to `A | B & C & E | B & C & D`, which is the result of the `DistributiveSimplifier`.
  The `FactorOutSimplifier` however, goes a step further: It also factors `B & C` out from the simplified formula, leading to `A | C & B & (E | D)`.

- The `DistributiveSimplifier` does not factor out if that would lead to losing an operand. Consider the two examples:
  - `A | A & B`, `DistributiveSimplifier`: `A | A & B`, `FactorOutSimplifier`: `A`
  - `A | A & B | A & C`, `DistributiveSimplifier`: `A | A & (B | C)`, `FactorOutSimplifier`: `A`

- The `FactorOutSimplifier` has a `RatingFunction` which enables the user to specify after which criteria the formula should be simplified

So, the `DistributiveSimpilifier` is inferior to the `FactorOutSimplifier`, is only kept for legacy reasons, and will likely be removed in upcoming versions of LogicNG.  Therefore, for all intents and purposes, we recommend to use the `FactorOutSimplifier`.


### Unit Propagation Simplification

The transformation [UnitPropagation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/UnitPropagation.java) performs unit propagation on a given formula. Unit propagation works the following way: If a formula is such that a literal is forced for the formula to be satisfied, then this literal is propagated through the formula and thus simplifies the formula.
Unit propagation is a key concept in [SAT Solving](../../../../solvers/sat-solving) and the implementation actually uses a modified variant of the MiniSat Solver.

For example, consider the formula

`(A | C) & ~C & (B | C) & (A | ~C)`

Then the literal `~C` is forced in the formula. Thus, the simplified formula (created by unit propagation) yields `~C & A & B`.


### Backbone Simplifier

The [BackboneSimplifier](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/simplification/BackboneSimplifier.java) computes the [Backbone](../../../../backbones) of a formula and propagates it through the formula, similarly to the Unit Propagation Transformation.

The backbone of a formula is the set of variables which have to be assigned to `true` or to `false` for the formula to evaluate to `true`.

For example, for a formula `A & B & (A | C) & (D | E) & ~F` the positive backbone is `A, B` - in any valid assignment, these variables have to be assigned to `true`.
The negative backbone is `F`, meaning that in any valid assignment, `F` has to be assigned to `false`.
With the backbone assignment `{A, B, ~F}`, the formula reduces to `(D | E)` - therefore the simplified formula yields `A & B & ~F & (D | E)`.


#### Unit Propagation vs. Backbone Simplifier

At first sight the backbone simplifier and unit propagation simplifier seem to be doing the same thing: They propagate variables which can only be assigned to either `true` or `false` through the formula.  However, the backbone simplifier guarantees to compute the *complete* backbone, whereas the unit propagation only propagates *single* literals through the formula.

Thus, transforming a formula using unit propagation is always faster, but the result may not be as much simplified as using the backbone simplifier (nerdy details: the Unit Propagation only collects literals which are propagated by a SAT Solver on level 0).

Consider the formula `f1`: `A & (~A | B | C) & (~A | B | ~C)`

``` java
Formula unitprop = f1.transform(new UnitPropagation());
Formula bb = f1.transform(new BackboneSimplifier());
```

The `UnitPropagation` propagates `A` through the formula and comes up with `(B | C) & (B | ~C) & A`.

The `BackboneSimplifier` goes a step further: It computes the complete backbone of the formula which also contains `B` and can thus simplify the formula further to `A & B`.


### Negation Simplifier

The [NegationSimplifier](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/simplification/NegationSimplifier.java) minimizes the number of negations of a formula by applying De Morgan's Law heuristically for a smaller formula. The resulting formula is minimized for the length of its string representation (using the string representation which is defined in the formula's Formula Factory).

For example, the formula `~A & ~B & ~C` stays this way (since `~(A | B | C)` is of same length as the initial formula), but the formula `~A & ~B & ~C & ~D` is being transformed to `~(A | B | C | D)` since its length is 16 vs. 17 in the un-simplified version.


## Simplifiers for Special Formula Types

### Subsumption

Consider a CNF with some clauses (disjunctions of literals), among them `c1` and `c2`.  Imagine that the set of literals of clause `c1` contains all literals of clause `c2`, but not the other way round.  In other words, the set of literals of `c2` is a *proper subset* of the set of literals of `c1`.  We then say that `c2` *subsumes* `c1`.  In this case, `c1` can be removed from the CNF without changing the semantics (meaning) of the formula, since every satisfying assignment for `c2` will also satisfy `c1`.

Similarly, given a DNF with some minterms (conjunctions), among them `m1` and `m2`, we say that `m1` *subsumes* `m2` if the set of literals of `m2` is a proper subset of the set of literals of `m1`.  So `m1` can be removed from the DNF without changing the semantics of the formula.

The subsumption simplifiers for CNF and DNF remove clauses/terms which are subsumed others clauses/terms, in order to yield a simpler formula.

- CNF subsumption [CNFSubsumption](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/cnf/CNFSubsumption.java): Consider the formula `(A | B) & (D | E) & (A | B | C)`: The clause `(A | B)` subsumes the clause `(A | B | C)`. The transformed formula is thus `(A | B) & (D | E)`.
- DNF subsumption [DNFSubsumption](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/dnf/DNFSubsumption.java): Similarly, consider `(A & B) | (D & E) | (A & B & C)`. The term `(A & B & C)` subsumes the term `(A & B)`. The transformed formula is `(A & B & C) | (D & E)`.


### Pure Expansion Transformation

The pure expansion transformation [PureExpansionTransformation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/PureExpansionTransformation.java) expands "at most one" or "exactly one" cardinality constraints by pure encodings, i.e. encodings which do not introduce new variables.  This is a simplification which does surely not minimize the length of the formula, but simplifies it in terms of the used operators.  This simplification can be important when e.g. performing a model count on a formula with cardinality constraints.  Not each cardinality constraint encoding preserves the model count.  Therefore, the model counter in LogicNG uses this transformation on its input formulas before computing the count.

For example, given the at-most-one constraint `A + B + C <= 1`, the transformed formula is `(~A | ~B) & (~A | ~C) & (~B | ~C)`.

Given the exactly-one constraint `A + B + C = 1`, the transformed formula is `(~A | ~B) & (~A | ~C) & (~B | ~C) & (A | B | C)`.

!!! warning "Pseudo-Boolean Constraints"

    Note that the transformation throws an exception if the formula contains any pseudo-boolean constraint which is not a cardinality constraint of type AMO or EXO since there are currently no pure encodings for these types of cardinality constraints.


## Putting it all Together: The Advanced Simplifier

The [AdvancedSimplifier](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/simplification/AdvancedSimplifier.java) is the most advanced simplifier of LogicNG which performs a very performant variant of Quine-McCluskey and many of the above simplifications.  In industrial applications it outperformed all existing simplification algorithms by far.  It performs the following steps:

1. Computation of all [prime implicants](../../formula-functions#compute-the-minimum-prime-implicant-of-a-formula)
2. Finding the minimal coverage over the found prime implicants (by finding one [smallest MUS](../../../../explanations/smus))
3. Building a DNF from the minimal prime implicant coverage
4. Factoring out common factors of the DNF using the `FactorOutSimplifier`
5. Minimizing negations of the factored-out DNF using the `NegationSimplifier`

In contrast to the `FactorOutSimplifier`, the `AdvancedSimplifier` currently always needs a rating function and can not be used without one.  In LogicNG 2.3., the `AdvancedSimplifier` will be able to be called without a rating function - it will then use the `DefaultRatingFunction`.

As an optional parameter the function takes an [OptimizationHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/OptimizationHandler.java).  This can be useful since both the computation of the prime implicants and the SMUS computation can take a long time.  With the handler these computations can be aborted if necessary.

As an example, this creates an advanced simplifier with the `DefaultRatingFunction`, where each computation process is canceled after 100 ms (every example in this section can be computed in `< 100 ms`):

``` java
AdvancedSimplifier simplifier = new AdvancedSimplifier(
        new DefaultRatingFunction(), new TimeoutOptimizationHandler(100));
```

!!! note
    The algorithm performs very well on small to middle-size formulas but of course, there are limits.  Since step 1 and 2 are quite computation-intensive the advanced simplifier should be used with care.


### Quine–McCluskey Algorithm

LogicNG currently contains an implementation of the [Quine-McCluskey algorithm](https://en.wikipedia.org/wiki/Quine%E2%80%93McCluskey_algorithm), a very common algorithm for minimizing formulas.  But this implementation will be removed in upcoming versions of LogicNG since the first two steps of the advanced simplifier above are equivalent to the Quine-McCluskey algorithm, but they perform far better than the old implementation.

With LogicNG 2.3 the advanced simplifier will be extended with the possibility to configure the single simplification steps and therefore can be configured to behave exactly like Quine-McCluskey (by deactivating steps 3-5).

