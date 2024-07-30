---
title: Backbones
---

The *backbone* of a formula is the set of variables which must *always* be assigned to a specific truth value (`true` or `false`) in order to make the formula satisfiable. The variables which have to be assigned to `true` can be thought of as *forced* variables, and those variables which have to be assigned to `false` can be thought of as *forbidden* variables.

One can also think of the backbone of a formula as the set of literals (positive and negative variables) which are present in their respective polarity in *every* satisfying assignment (model) of the formula.

The backbone of the formulas loaded on a [SAT Solver](../solvers/sat-solving) can be computed with the `SATSolver` method `backbone()`. The method takes two parameters:

1. The mandatory parameter `relevantVariables` indicates over which variables the backbone shall be computed.

2. The optional parameter [BackboneType](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/backbones/BackboneType.java) enables to control which parts of the backbone shall be computed: Only positive (`ONLY_POSITIVE`) variables, only negative (`ONLY_NEGATIVE`) variables, or both (`POSITIVE_AND_NEGATIVE`).

If no `BackboneType` is specified, then the complete backbone (`POSITIVE_AND_NEGATIVE`) is computed.
If you are only interested in one part (only positive or negative) you can specify the `BackboneType` to the relevant parameter in order to save computation time. If you don't mind performance or are interested in both the positive and negative part, there is no need to specify this parameter.

As an example, consider:

``` java
SATSolver solver = MiniSat.miniSat(this.f);
Formula formula1 = f.parse("A & (~B | C) & ~D");
Formula formula2 = f.parse("D | E");
solver.add(formula1);
solver.add(formula2);
Backbone backboneFull = solver.backbone(solver.knownVariables());
Backbone backboneRestricted = solver.backbone(formula1.variables());
Backbone backboneOnlyPos =
        solver.backbone(formula1.variables(), BackboneType.ONLY_POSITIVE);
```

Then `backboneFull` returns the backbone over all variables on the solver:

- Positive variables ("forced variables"): `A, E`
- Negative variables ("forbidden variables"): `D`
- Optional variables (neither "forced" nor "forbidden"): `B, C`

One can also compute the backbone restricted to a subset of variables: `backboneRestricted()` returns the backbone restricted to the variables of `formula1`, which are `A, B, C, D`.
Thus, only the backbone information about those variables is returned:
- `A` is forced
- `D` is forbidden
- `B, C` are optional

The backbone can be restricted to one part (positive or negative) by the `BackboneType`. The computed backbone from `backboneOnlyPos` is therefore simply `A`.

## Computing the backbone with a handler

The `backbone()` method on the solver internally calls the [BackboneFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/functions/BackboneFunction.java). You can use this function directly in order to specify a handler for the computation.

Consider the following example for when a handler aborts the computation:

``` java
SATSolver solver = MiniSat.miniSat(this.f);
Formula formula = f.parse("~A & B & C & D & (~E & F | G | H)");
solver.add(formula);
SATHandler handler = new BoundedSatHandler(3);
Backbone backbone = solver.execute(BackboneFunction.builder()
        .handler(handler)
        .variables(formula.variables())
        .build()
);
```

The [BoundedSatHandler](https://github.com/logic-ng/LogicNG/blob/master/src/test/java/org/logicng/handlers/BoundedSatHandler.java) aborts the computation in case the number of starts needed exceeds 3.
Since the handler aborts the computation, the computed backbone is *null*. For more information about handlers, see [the chapter on handlers](../handlers).


### Empty backbones

If the formulas for which you're trying to compute the backbone are unsatisfiable or do not contain variables, then an empty backbone is returned.

As an example, consider:

``` java
SATSolver solver = MiniSat.miniSat(this.f);
Formula formula = f.parse("A & (~A | B) & ~B");
solver.add(formula);
Backbone backbone = solver.execute(BackboneFunction.builder()
        .variables(formula.variables())
        .build()
);
```

Since the formula is unsatisfiable, an empty backbone is returned:

```
Backbone{positiveBackbone=[], negativeBackbone=[], optionalVariables=[]}
```

You can check whether the formulas for which you're computing the backbone are satisfiable using `backbone.isSat()`.

By the way: The simplifier which propagates the backbone through a formula is the [Backbone Simplifier](../formulas/operations/transformations/simplifier-transformations#backbone-simplifier).

For theoretical insights on the backbone computation, see the paper [Algorithms for Computing Backbones of Propositional Formulae](http://sat.inesc-id.pt/~mikolas/bb-aicom-preprint.pdf).
