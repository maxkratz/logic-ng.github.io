---
title: Model Counting and Enumeration
---

When you determined the satisfiability of a formula with a [SAT Solver](../solvers/sat-solving) and the formula is satisfiable, the solver yields one satisfying assignment (also called model) of the formula.  But you might wonder *how many* such models there are or list them all explicitly.  The first question concerns *model counting*, and the second concerns *model enumeration*.

## Model enumeration

In LogicNG, you can either determine *all* models for a given formula (Model Enumeration, ME), or all models for a given subset of the formula's variables (Projected Model Enumeration, PME).

Consider the following example:

``` java
Formula f1 = p.parse("A | (~B & C)");
SATSolver solver = MiniSat.miniSat(f);
solver.add(f1);
```

Then `solver.enumerateAllModels()` returns all valid assignments of the solver:

- `~A, ~B, C`
- `A, B, C`
- `A, ~B, C`
- `A, ~B, ~C`
- `A, B, ~C`

In case you are not interested in the model for *every* variable, you can give over the variables you're interested in as a parameter to the method and perform projected model enumeration. For example, `#!java solver.enumerateAllModels(f.variable("A"), f.variable("B"))` returns the possible models for the variables `A` and `B`:

- `~A, ~B`
- `A, B`
- `A, ~B`

Similarly, `#!java solver.enumerateAllModels(f.variable("A"))` returns the models for `A`:

- `A`
- `~A`

The method `enumerateAllModels()` internally calls the solver function [ModelEnumerationFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/functions/ModelEnumerationFunction.java). That is, you get the same result (here for example for the projection on `A`) using

``` java
List<Assignment> models = solver.execute(
        ModelEnumerationFunction.builder().variables(f.variable("A")).build());
```

### Additional variables

When using the solver function directly, you can perform the projected model enumeration with so-called "additional variables". Those are variables which are not relevant for the actual enumeration, but we are interested in their assignment for any found model.

Consider the following example:

``` java
SATSolver solver = MiniSat.miniSat(this.f);
Formula formula1 = p.parse("A & (B | C)");
Formula formula2 = p.parse("B | D");
solver.add(formula1);
solver.add(formula2);

SortedSet<Variable> pmeVars =
        new TreeSet<>(Arrays.asList(f.variable("A"), f.variable("B")));

List<Assignment> models1 = solver.execute(ModelEnumerationFunction.builder()
        .variables(pmeVars)
        .build()
);
List<Assignment> models2 = solver.execute(ModelEnumerationFunction.builder()
        .variables(pmeVars)
        .additionalVariables(f.variable("C"))
        .build()
);
```
The projected model enumeration over `A` and `B` (`models1`) returns the models:

- `A, B`
- `A, ~B`

The projected model enumeration over `A` and `B` with additional variable `C` (`models2`) returns the models:

- `A, B, ~C`
- `A, ~B, C`

That is, the projected model enumeration with additional variables finds for those assignments computed by the projected model enumeration *one* possible assignment for the variable `C`.
This is really just *one* possible assignment: Note that not only `A, B, ~C`, but also  `A, B, C` is a valid model.

!!! tip "Application Insight"

    You may find `additionalVariables()` useful if you want to perform projected model enumeration over `variables()`, but are also interested in what truth value some "additional variables" can have in those assignments.

Note that it does not make sense to add `additionalVariables()` without `variables()` to the builder, and it also does not make sense that the variables in `additionalVariables()` and the variables in `variables()` are overlapping.


### Configuring the Assignment Type

[:octicons-tag-24: 2.3.0](https://github.com/logic-ng/LogicNG/releases/tag/v2.3.0)

The builder method `fastEvaluable()` configures whether the models of the model enumeration are generated as fast evaluable Assignments or not.  For details about fast evaluable assignment see the info box [here](../formulas#evaluating-formulas).


## Model counting

Model counting is #P-complete, which in practice is much harder than NP. It is the task to compute the number of satisfiable models of a formula.

If you already have an algorithm for model enumeration (as you have seen above), you can trivially count the number of models: `solver.enumerateAllModels().size()`.

However, this approach isn't feasible for large model counts, since every model explicitly has to be enumerated. Instead of this trivial approach, [knowledge compilers](../knowledge-compilation) can be used to compute the model count more effectively.  Both, [BDDs](../knowledge-compilation/bdd), and [DNNFs](../knowledge-compilation/dnnf) are suitable for this task. The [ModelCounter](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/modelcounting/ModelCounter.java) in LogicNG is based on the [d-DNNF](../knowledge-compilation/dnnf) of a formula.

An example is:

``` java
Formula f1 = p.parse("A & (B | C)");
Formula f2 = p.parse("B | D");
Formula f3 = p.parse("~A | B & E");
List<Formula> formulas = Arrays.asList(f1, f2, f3);

SortedSet<Variable> variables = new TreeSet<>(Arrays.asList(f.variable("A"),
        f.variable("B"), f.variable("C"), f.variable("D"), f.variable("E")));

BigInteger modelcount = ModelCounter.count(formulas, variables);
```

The result is 4, which can (in this simple case) be verified using model enumeration:

- `A, B, ~C, ~D, E`
- `A, B, ~C, D, E`
- `A, B, C, ~D, E`
- `A, B, C, D, E`

The second parameter of the `count` function is an important one: Because of the automatic simplifications of formulas in LogicNG, it can happen that "irrelevant" variables (which don't have any influence on whether the formula is satisfiable or not) are removed from the formula. However, these variables still do affect the model count, since they can be set to both `false` and `true` s.t. each of those variables increases the model count by a factor of 2. So usually it makes sense to collect the set of variables passed to the model counter not from the formula at hand, but from the source where the formula came from.

On the other hand, the set of variables must never be a strict subset of the variables of the formula.

Projected model counting is currently not implemented in LogicNG (but there are plans...).

!!! note
    It does not make sense to (1) compute a model, (2) perform (projected) model enumeration or (3) perform a model count when the conjunction of clauses which have been added to the solver is unsatisfiable. Depending on what you're trying to do, the result is `null` (1), `[]` (2) or `0` (3).

    If the solver is `UNSAT`, then there are two things you can do to understand why:

    1. Check the [MUS](../explanations/mus) of the SAT solver
    2. Check the [resolution proof](../explanations/unsat-cores) of the SAT solver using `unsatCore()`. Careful: this method only works if you have configured your SAT solver with `proofGeneration = true`

    More about this in the chapter on [explanations](../explanations).
