---
title: Other Transformations
---

## Literal Substitution

The transformation [LiteralSubstitution](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/LiteralSubstitution.java) substitutes literals in a given formula. For example, given the formula `A & ~B | B & ~D`, and the substitutions

- `~B -> C`
- `B -> E`
- `D -> ~A`

then the substituted formula is `A & C | E & A`.

The difference to the `substitute` operation on formulas is, that such a substitution can only be applied from *variables* to arbitrary formulas, whereas the `LiteralSubstitution` operates on literals and therefore can replace positive and negative literals differently.

!!! info "Parallel Substitution"

    Note that both substitutions perform a *parallel* substitution:

    That is, the formula `(X | A) & (Y | B) & (Z | C)` with the substitutions
    - `A -> B`
    - `B -> C`

    results in `(X | B) & (Y | C) & (Z | C)`, and not `(X | C) & (Y | C) & (Z | C)`.

The mapping that should be applied can either be given to the transformation as a parameter by using the parameterised constructor, or, if one wants to add mappings in hindsight, by adding substitutions with `addSubstitution()`.


## Anonymizer

The [Anonymizer](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/Anonymizer.java) transforms the given formula into one where all names of the formulas' variables are replaced by new ones.  This is usually used when the original variable names contain domain-specific knowledge and one wants to anonymize the formula.  One can define how the new variables are named: A prefix with which the variable shall begin and a number where to start.  The default values are `v` and `0`, meaning the anonymized variables will be `v0, v1, v2, ...` .

!!! info "Variable Ordering"

    Note that the order of the new variables is based on the alphabetical order of the original variables (not in the order in which the variables occur in the formula). For example, when anonymizing the formula `A & C | B`, the transformed formula is `v0 & v2 | v1` (and not `v0 & v1 | v2`, as one might expect).

[:octicons-tag-24: 2.3.0](https://github.com/logic-ng/LogicNG/releases/tag/v2.3.0) The method `getSubstitution()` returns the current substitution map from the anonymizer.  This can be helpful if you want e.g. translate an anonymized formula back to its original one.


## QE Transformations

Using quantifier elimination transformations, one can eliminate quantified variables from a formula. There is the existential quantifier elimination [ExistentialQuantifierElimination](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/qe/ExistentialQuantifierElimination.java), which eliminates the existential quantifier `∃` in a given formula, and the universal quantifier elimination [UniversalQuantifierElimination](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/qe/UniversalQuantifierElimination.java), which respectively eliminates the universal quantifier `∀`.

Both transformations use the Shannon expansion for eliminating the quantifier, see ([Boolean quantification in a first-order context by Seidl and Sturm](https://www.researchgate.net/publication/245588395_Boolean_Quantification_in_a_First-Order_Context)).  In short: the formula is expanded once with the variable to eliminate substitued by false, once by true.  For an existential quantifier elimination the two formulas are then disjoined, for an universal quantifier elimination they have to be conjoined.

Firstly, consider the `ExistentialQuantifierElimination` with the formula `∃C: A & B & (C | D)`.

The transformation propagates the statement "there exists the variable `C`" through the formula, which yields `A & B | A & B & D` and can be simplified to `A & B`, meaning `A & B` is a necessary and sufficient condition in order that there exists an assignment for `C` in order to evaluate the whole formula to true.

Secondly, consider the `UniversalQuantifierElimination` with the formula `∀C: A & B & (C | D)`.
Let `f3 = A & B & (C | D)`. Then the statement is: "`f3` holds for all `C`", regardless whether `C` is assigned to `true` or `false`. Simplified by the `UniversalQuantifierElimination`, this yields`A & B & D`, meaning `A & B & D` is a necessary and sufficient condition in order that for all assignments of `C` the formula can evaluate to true.  The `UniversalQuantifierElimination` can also evaluate to `false`: For example, `∀C, D: A & B & (C | D)`, that is "`f3` holds for all `C` and all `D`", is unsatisfiable.

Both quantifier elimination transformations cannot be cached since they are dependent on the set of literals to eliminate.

Here is a code snippet for this example for the existential quantifier elimination (analogous for the universal quantifier elimination):

``` java
ExistentialQuantifierElimination elimination =
        new ExistentialQuantifierElimination(f.variable("C"));
Formula result = elimination.apply(f3, true);
```


## Formula Factory Importer

The [FormulaFactoryImporter](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/FormulaFactoryImporter.java) imports a formula from another formula factory into the current formula factory.  For example, the following code snippet imports the formula `f1` from the formula factory `f` to the new formula factory `g`:

```java
FormulaFactory g = new FormulaFactory();
FormulaFactoryImporter factoryImporter = new FormulaFactoryImporter(g);
Formula result = f1.transform(formulaImporter, true);
```

The formula `result` is equivalent to `f1` but belongs to the formula factory `g`.
