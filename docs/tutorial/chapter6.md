---
title: Tutorial Chapter 6
---

After adding a set of rules to a SAT solver one can determine whether the conjunction of these rules is satisfiable or not and when it is not satisfiable, one can compute a proof/explanation for the unsatisfiability.  But there is a second question for an unsatisfiable set of rules: what is the maximum number of rules which can be satisfied?  This is not a decision problem anymore, but an optimization problem and a special kind of solver is used for this use case: a MaxSAT solver.  Many use cases can be solved with a MaxSAT solver.

There are also two very interesting extensions to MaxSAT solvers which give them even more power:

1. One can differentiate between so-called "hard" and "soft" formulas. Hard formulas are required to be satisfied while soft clauses are not.  The MaxSAT solver then only optimizes over the soft formulas while always satisfying the hard formulas.
2. Soft formulas can also hava a *weight*.  Then the solver optimizes not the *number* of formulas, but their respective weight.

Formulas with both soft and hard formulas are called *partial* problems, and problems where the soft formulas carry weights are called *weighted* problems.  LogicNG has solvers which can solve partial weighted MaxSAT problems.  For a detailed introduction to MaxSAT-Solving in LogicNG, check out the [relevant chapter](../../documentation/solvers/maxsat-solving).

In this chapter we will demonstrate two use cases of MaxSAT-Solving:

1. Finding a configuration with the minimum number of features
2. Finding the cheapest bike


## Solving a Partial MaxSAT Problem

As mentioned in the previous chapter, Beatrice can also formulate the problem of finding the minimum number of features as a MaxSAT problem.  In this case all the original product rules (the restricted propositions) are hard clauses, meaning each configuration must still adhere to all rules.  If we want to find the minimum number of features in a configuration, we add *each feature* as a negative literal to the MaxSAT solver as a soft formula.  Since we do not differentiate between the single features, all soft clauses have weight 1.  So ideally we want *all* features to be negative, i.e. not in the configuration.  This is not possible, so now the solver searches for the maximum number of rules it can satisfy - or in other terms, the minimum number of rules it has to ignore.  In this case the solver has to ignore 6 rules at a minimum.  Since each soft rule set the feature to false, this means, at least 6 features have to be set to true.  This yields the same result as the optimization in the last chapter.

``` java
MaxSATSolver solver = MaxSATSolver.msu3();
propositions.forEach(x -> solver.addHardFormula(x.formula()));

SortedSet<Variable> vars = propositions.stream() // (1)
        .map(proposition -> proposition.formula().variables())
        .flatMap(Collection::stream)
        .filter(v -> !data.featureClasses.contains(v))
        .collect(Collectors.toCollection(TreeSet::new));

vars.forEach(v -> solver.addSoftFormula(v.negate(), 1));
solver.solve();
Assignment model = solver.model();
```

1. all relevant features (without feature classes)

The result is:

```
Assignment{
  pos=[f2, frame, h1, handlebar, s1, saddle, frontWheel, wf26, backWheel,
       wb26, luggageRack, c1, color],
  neg=[~f1, ~f3, ~h2, ~h3, ~h4, ~h5, ~s2, ~s3, ~s4, ~wf24, ~wf27, ~wf29, ~wf32,
       ~wb27, ~wb29, ~wb32, ~b1, ~b2, ~b3, ~bell, ~r1, ~r2, ~r3, ~c2, ~c4]
}
```

Note that the found configuration is different from the one in the last chapter, but the minimum number of features stays the same.  If you want to compute the maximum number of features in a configuration, you have to add the literals positively to the solver as soft clauses.  So maximizing the number of features, set to true.


## Solving a Weighted, Partial MaxSAT Problem

Beatrice has another customer: A young lady called Coco comes to the shop. Coco likes cycling, but she likes saving too.  Coco is interested in the cheapest bike.
Still, a bike has to fit and Beatrices trained eye tells her that Coco needs wheel size 27.5-inch.

This is Beatrice's price table:

| Feature | Price in Euros |
|---------|----------------|
| `f1`    | `1500`         |
| `f2`    | `900`          |
| `f3`    | `400`          |
| `h1`    | `380`          |
| `h2`    | `200`          |
| `h3`    | `120`          |
| `h4`    | `300`          |
| `s1`    | `200`          |
| `s2`    | `240`          |
| `s3`    | `160`          |
| `s4`    | `300`          |
| `r1`    | `130`          |
| `r2`    | `120`          |
| `r3`    | `20`           |
| `r4`    | `100`          |

For some features, however, the prices do not differentiate: The bike bells and the colors for example have all the same price.  Beatrice uses MaxSAT-Solving to find the cheapest bike. Like above, the product rules are added as hard clauses since each bike has to adhere them.  Beatrice then uses soft formulas with the features price as weight.  Since the solver searches the maximum weight but Beatrice wants the cheapest price (so the minimum) weight, she adds each feature negatively to the solver.  So now the solver tries to maximize the price of the features *not* in the configuration and thereby minimizing the price of the features actually *in* the configuration.  This technique (solving the dual problem) is very common in MaxSAT solving.

``` java
MaxSATSolver solver = MaxSATSolver.wbo();
propositions.forEach(proposition ->
        solver.addHardFormula(proposition.formula()));
solver.addHardFormula("wf27"); // (1)

solver.addSoftFormula(f1.negate(), 1500); // (2)
solver.addSoftFormula(f2.negate(), 900);
solver.addSoftFormula(f3.negate(), 400);

solver.addSoftFormula(h1.negate(), 380);
solver.addSoftFormula(h2.negate(), 200);
solver.addSoftFormula(h3.negate(), 120);
solver.addSoftFormula(h4.negate(), 300);

solver.addSoftFormula(s1.negate(), 200);
solver.addSoftFormula(s2.negate(), 240);
solver.addSoftFormula(s3.negate(), 160);
solver.addSoftFormula(s4.negate(), 300);

solver.addSoftFormula(r1.negate(), 130);
solver.addSoftFormula(r2.negate(), 120);
solver.addSoftFormula(r3.negate(), 20);
solver.addSoftFormula(r4.negate(), 100);

solver.solve(); // (3)
solver.model(); // (4)
```

1. add front wheel 27.5 inch
2. add prices for the different features
3. solve the problem
4. give out the model

The result is:

```
Assignment{
  pos=[f3, frame, h3, handlebar, s3, saddle, frontWheel, wf27, backWheel,
       wb27, luggageRack, c4, color],
  neg=[~f1, ~f2, ~h1, ~h2, ~h4, ~h5, ~s1, ~s2, ~s4, ~wf24, ~wf27, ~wf29,
       ~wf32, ~wb27, ~wb29, ~wb32, ~b1, ~b2, ~b3, ~bell, ~r1, ~r2, ~r3,
       ~c1, ~c2, ~r4]
}
```

This means that the cheapest bike consists of the following features: `f3`, `h3`, `s3`, `wf27`, `wb27`, `c4`.  The color `c4` is chosen randomly, since all colors cost the same.  The price of the configuration can be found with `#!java solver.result()`. It is `680`.
