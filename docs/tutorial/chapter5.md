---
title: Tutorial Chapter 5
---

A large part of LogicNG is [SAT Solving](../../documentation/solvers/sat-solving). In their most basic variant, SAT Solvers can compute whether a formula is satisfiable or not.  However, there are many algorithms which can compute more complex things by iteratively calling and modifying a SAT Solver.  Also, LogicNG provides many extensions to common SAT solvers which help solving real world problems.

In this chapter we present five use cases for a SAT solver:

1. Beatrice checks for the customer Sarah whether her desired configuration is satisfiable. Since it is satisfiable, she also _completes_ her order.
2. Beatrice checks for the customer Clemens whether his desired configuration is buildable. Since it is not buildable, she computes the reason for it.
3. Beatrice wonders what configuration options there are, if she decides on a certain wheel size.
4. Beatrice checks how many bike configurations she can sell overall, and how many configurations exist for a common combinations of features.
5. Beatrice checks what the bike configuration with the minimum and maximum number of variables is.

Along this chapter, we will cover the following related topics:

- The two techniques for incremental SAT solving (assumption solving and solving with save/load state)
- The difference between a satisfiable and a complete order
- Explanations for when the solver returns "UNSAT"
- Backbone computation
- Model counting and enumeration, projected model enumeration
- Performing functions on the solver


## Solving with Assumptions

The first customer, a young woman called Sarah, enters the shop.  After studying the feature table shortly, she claims that she knows exactly what she wants out of a bike.  Sarah wants the carbon frame `f1` in red `c2`, the drop bar `h2`, 26-inch front wheel `wf26` and the cute ladybug bike bell `b3`.

"Alright, let's see what we can do", Beatrice concedes.

Beatrice evaluates the set of rules that she has for configuring a bike with a SAT solver.

```java
SATSolver solver = MiniSat.miniSat(data.f);
propositions.forEach(solver::add);

List<Literal> wishedFeaturesSarah = new ArrayList<>(Arrays.asList(
        data.f1, data.c2, data.h2, data.f.variable("wf26"), data.b3));
Tristate sat = solver.sat(wishedFeaturesSarah);
```

The result is `TRUE`, meaning that this configuration is _satisfiable_. That is, a bike with the features that Sarah requested is buildable. (We use "satisfiable" and "buildable" equivalently - Satisfiable is the mathematical term, and buildable is closer to the use case of bike configuration.)

However, this is _different_ from saying that the requested features result in a valid bike.  Sarah's order is satisfiable, but not complete: If you tried to build a bike with the features Sarah requested, you would miss a back wheel and a saddle.

A complete order is always satisfiable, and a satisfiable order is complete only if all product rules evaluate to true under the given features.


## Satisfiable vs. Complete Configuration

Let's check whether Sarah's order is complete. In order to do this, we first build an assignment with Sarah's wished features. All other features will then automatically be set to `false`.

Then, we go over each formula and check whether it is violated under the assignment.  If it is violated, then we print the rule.

``` java
wishedFeaturesSarah.addAll(Arrays.asList(
        data.frame, data.handlebar, data.saddle, data.frontWheel,
        data.backWheel, data.luggageRack, data.color, data.bell)); // (1)!
Assignment assignment = new Assignment(wishedFeaturesSarah); // (2)!
List<Formula> formulas = propositions.stream()
        .map(Proposition::formula)
        .collect(Collectors.toList());

for (Formula formula : formulas) {
    boolean isViolated = !formula.evaluate(assignment); // (3)!
    if (isViolated) {
        System.out.println(formula);
    }
}
```

1. add the feature classes
2. create an assignment with the wished features
3. evaluate each formula with the given assignment

The formulas which are violated under the assignment are:

```
saddle <=> s1 | s2 | s3 | s4
s1 + s2 + s3 + s4 = 1
backWheel <=> wb26 | wb27 | wb29 | wb32
wb26 + wb27 + wb29 + wb32 = 1
wf26 <=> wb26
```

That makes sense. Sarah has to choose a saddle and a back wheel for the bike to result in a valid configuration! Also, back and front wheel must fit together.

Sarah chooses the triathlon saddle `s3` and the back wheel with the same size as the front wheel, `wf26`.
Thus, Build 'n bike sells their first bike!

The approach with `#!java solver.sat(wishedFeatures)` is called *assumption solving*, meaning the `wishedFeatures` are not actually put permanently on the solver, but the `sat`-call checks whether the formulas on the solver are satisfiable with the given assumptions.


## Solving with Save and Load State

Just a little later a middle-aged man called Clemens enters the shop. He is an experienced biker and knows what he's looking for in a bike: He also wants the carbon frame `f1`, but in blue `c1`, front wheel 28-inch `wf28`. He would also like his bike to have the "comfort" saddle `s2`. Clemens doesn't mind the other components of the bike.

Beatrice calls the method from above with new parameters:

``` java
List<Literal> wishedFeaturesClemens = Arrays.asList(
        data.f1, data.c1, data.s2, data.f.variable("wf27"), data.b2);
sat = solver.sat(wishedFeaturesClemens);
```

However, this time, the result is `FALSE`, meaning the requested configuration is *not satisfiable*.


## Computing the Reason for a Conflict

But why is it not satisfiable? Beatrice wants to find the set of propositions which explain the conflict of the order.  This set of propositions is the [Unsat Core](../../documentation/explanations/unsat-cores) of the formula in conjunction with Clemens' order, i.e. a set of rules which formulate a contradiction together.

To compute the `UnsatCore`, she cannot use *assumption solving*.  Beatrice needs to put the `wishedFeatures` on the solver.  Further, the solver needs to have the flag `proofGeneration` in its configuration set to `true`.

The solver with its configuration and all propositions is created like this:

``` java
SATSolver solver = MiniSat.miniSat(data.f,
        MiniSatConfig.builder().proofGeneration(true).build());
propositions.forEach(solver::add);
```

Since after checking Clemens' order against the rules, Beatrice perhaps wants to test another order, she saves the solver state after adding all the propositions and before adding Clemens' order:

``` java
SolverState initialState = solver.saveState();
```

Now she can add formulas to the solver and when she is done she can return to this saved solver state.  This functionality - adding and removing formulas from the solver - is very important in solving real problems.  Beatrice can now add Clemens' order to the solver and finally compute its unsat core.

``` java
List<Literal> wishedFeaturesClemens = Arrays.asList(
        data.f1, data.c1, data.s2, data.f.variable("wf27"), data.b2);
wishedFeaturesClemens.forEach(feature ->
        solver.add(new StandardProposition("Order", feature)));
solver.sat(); // (1)!

UNSATCore<Proposition> unsatCore = solver.unsatCore();
```

1. the sat-call must be 'FALSE' for computing the unsat core

The result that we obtain is the following:

```
ExtendedProposition{formula=f1 + f2 + f3 = 1, backpack=MyBackpack{id=1, date=2020-01-01}}
ExtendedProposition{formula=~b2 | f2 | f3, backpack=MyBackpack{id=20, date=2022-01-01}}
StandardProposition{formula=f1, description=Order}
StandardProposition{formula=b2, description=Order}
```

Beatrice translates the findings:
The result of the `.unsatCore()` method is an unsatisfiable set of propositions:

- `f1` has been chosen by Clemens
- `b2` has been chosen by Clemens
- `f1 + f2 + f3 = 1` with backpack `{id=1, date=2020-01-01}`
- `~b2 | f2 | f3` with backpack `id=20, date=2022-01-01}`

That makes sense! Since Clemens wants the carbon frame `f1`, he may not choose any of `f2` or `f3`.
However, the last rule is: Either you do not take bike bell `b2`, or you take `f2` or `f3`.

Thus, if Clemens removed the bike bell `b2` from his configuration, it should work, right?  Beatrice checks this by resetting the solver with `.loadState()` and checking the new configuration:

``` java
solver.loadState(initialState);
List<Literal> wishedFeaturesFixed =
        Arrays.asList(data.f1, data.c1, data.s2, data.f.variable("wf28"));
solver.add(wishedFeaturesFixed);
Tristate sat = solver.sat();
```

The result now is `TRUE`, meaning this configuration is satisfiable. Note that the order may still not be complete, as explained above in the paragraph "Satisfiable vs. complete configuration".

Note:  The set of rules returned by `.unsatCore()` may not be minimal.  For a minimal unsatisfiable set (MUS) there are special algorithms which are described in [its own chapter](../../documentation/explanations/mus).


## Computing the Backbone

Beatrice wonders which features are still free to choose when she takes a certain back wheel. She starts with the biggest back wheel of size 32-inch.

This set of forced and forbidden variables of a formula is called the [backbone](../../documentation/backbones) of a formula and can be computed with LogicNG (c.f. the method `computeBackbone()`).  The features which are still free to choose are the variables which are not contained in the positive or negative backbone.

Beatrice computes the backbone in the following way:

``` java
MiniSat solver = MiniSat.miniSat(data.f);
propositions.forEach(solver::add);
solver.add(data.f.variable("wb32")); // (1)!
List<Variable> variablesInFormula = propositions.stream()
        .map(proposition -> proposition.formula().variables())
        .flatMap(Collection::stream)
        .collect(Collectors.toList());

Backbone backbone = solver.backbone(variablesInFormula);
```

1. restricting the backbone to wb32

The result is:

```
Positive: [backWheel, color, f3, frame, frontWheel, handlebar, luggageRack,
           saddle, wb32, wf32]
Negative: [f1, f2, h5, r1, r2, r3, wb26, wb27, wb29, wf24, wf26, wf27, wf29]
```

Beatrice translates the output. In a bike with a back wheel of size 32-inch,
- the following features have to be contained:
  - The feature classes `backWheel`, `color`, `frame`, `frontWheel`, `handlebar`, `luggageRack` and `saddle` have to be present in any bike.
  - Further, the features `f3` and `wf32` have to be present with the given restriction. She recalls that the rules `wf32 <=> wb32` and `wf32 => f3` are responsible for this.
- the following features can never be contained in any configuration: Frame `f1` or `f2`, the aero bar `h5`, luggage rack `r1`, `r2` or `r3`, the other front and back wheels.
- all other features are optional, meaning there are configurations with and without them


## Model Counting and Enumeration

Beatrice would like to know how many different bikes she can sell, and what they are.  *Model counting* answers how many different bike configurations there are, and *model enumeration* answers what these models are.  Check out the [relevant chapter](../../documentation/model-counting-enumeration) in the documentation.

As a simple example, consider

``` java
SATSolver solver = MiniSat.miniSat(data.f);
propositions.forEach(solver::add);

solver.sat(); // (1)!

Assignment aModel = solver.model(); // (2)!

List<Assignment> allModels = solver.enumerateAllModels(); // (3)!
```

1. the sat problem has to be solved before enumerating models
2. give out one model
3. return all models

With the call to `#!java solver.model()` one can find one assignment for the formulas. For example:

```
Assignment{
  pos=[b1, backWheel, bell, c4, color, f3, frame, frontWheel, h1, handlebar,
       luggageRack, s1, saddle, wb26, wf26],
  neg=[~b2, ~b3, ~c1, ~c2, ~f1, ~f2, ~h2, ~h3, ~h4, ~h5, ~r1, ~r2, ~r3, ~s2,
       ~s3, ~s4, ~wb27, ~wb29, ~wb32, ~wf24, ~wf27, ~wf29, ~wf32]
}
```

With the call to `#!java solver.enumerateAllModels()` one finds *all* possible models, that is, in this example, all complete bike configurations.  The models distinguish in all sorts of features, for example whether they have a bike bell or not, which color they have, which saddle and which size of front and back wheel they have. Then `#!java allModels.size()` returns `1650`. That is, there are 1650 possible assignments for the formulas, or in our case, configurations for a bike. Beatrice can sell 1650 different bike configurations!


### Enumerating After Deciding on Some Features

Next, Beatrice knows that a certain combination of features is quite popular: color blue, wheel size 27.5-inch, carbon frame and ladybug bike bell.  The fitting handlebar and saddle, however, differ with every buy.  Beatrice would like to know how many different models there are with those features.  For this, she first adds the relevant features to the solver.

``` java
SATSolver solver = MiniSat.miniSat(data.f);
propositions.forEach(solver::add);
solver.add(data.f.variable("wf27"));
solver.add(data.f1);
solver.add(data.b3);
solver.add(data.c1);
solver.sat();
List<Assignment> models = solver.enumerateAllModels();
```

Then `#!java models.size()` returns `9`, meaning that there are 9 configurations possible with these restrictions. This is a dealable size, so we can print the models:

```
Assignment{pos=[b3, backWheel, bell, c1, color, f1, frame, frontWheel, h1, handlebar, luggageRack, s2, saddle, wb27, wf27], neg=[~b1, ~b2, ~c2, ~c4, ~f2, ~f3, ~h2, ~h3, ~h4, ~h5, ~r1, ~r2, ~r3, ~s1, ~s3, ~s4, ~wb26, ~wb29, ~wb32, ~wf24, ~wf26, ~wf29, ~wf32]}
Assignment{pos=[b3, backWheel, bell, c1, color, f1, frame, frontWheel, h1, handlebar, luggageRack, s3, saddle, wb27, wf27], neg=[~b1, ~b2, ~c2, ~c4, ~f2, ~f3, ~h2, ~h3, ~h4, ~h5, ~r1, ~r2, ~r3, ~s1, ~s2, ~s4, ~wb26, ~wb29, ~wb32, ~wf24, ~wf26, ~wf29, ~wf32]}
Assignment{pos=[b3, backWheel, bell, c1, color, f1, frame, frontWheel, h4, handlebar, luggageRack, s3, saddle, wb27, wf27], neg=[~b1, ~b2, ~c2, ~c4, ~f2, ~f3, ~h1, ~h2, ~h3, ~h5, ~r1, ~r2, ~r3, ~s1, ~s2, ~s4, ~wb26, ~wb29, ~wb32, ~wf24, ~wf26, ~wf29, ~wf32]}
Assignment{pos=[b3, backWheel, bell, c1, color, f1, frame, frontWheel, h2, handlebar, luggageRack, s3, saddle, wb27, wf27], neg=[~b1, ~b2, ~c2, ~c4, ~f2, ~f3, ~h1, ~h3, ~h4, ~h5, ~r1, ~r2, ~r3, ~s1, ~s2, ~s4, ~wb26, ~wb29, ~wb32, ~wf24, ~wf26, ~wf29, ~wf32]}
Assignment{pos=[b3, backWheel, bell, c1, color, f1, frame, frontWheel, h2, handlebar, luggageRack, s4, saddle, wb27, wf27], neg=[~b1, ~b2, ~c2, ~c4, ~f2, ~f3, ~h1, ~h3, ~h4, ~h5, ~r1, ~r2, ~r3, ~s1, ~s2, ~s3, ~wb26, ~wb29, ~wb32, ~wf24, ~wf26, ~wf29, ~wf32]}
Assignment{pos=[b3, backWheel, bell, c1, color, f1, frame, frontWheel, h2, handlebar, luggageRack, s2, saddle, wb27, wf27], neg=[~b1, ~b2, ~c2, ~c4, ~f2, ~f3, ~h1, ~h3, ~h4, ~h5, ~r1, ~r2, ~r3, ~s1, ~s3, ~s4, ~wb26, ~wb29, ~wb32, ~wf24, ~wf26, ~wf29, ~wf32]}
Assignment{pos=[b3, backWheel, bell, c1, color, f1, frame, frontWheel, h4, handlebar, luggageRack, s2, saddle, wb27, wf27], neg=[~b1, ~b2, ~c2, ~c4, ~f2, ~f3, ~h1, ~h2, ~h3, ~h5, ~r1, ~r2, ~r3, ~s1, ~s3, ~s4, ~wb26, ~wb29, ~wb32, ~wf24, ~wf26, ~wf29, ~wf32]}
Assignment{pos=[b3, backWheel, bell, c1, color, f1, frame, frontWheel, h1, handlebar, luggageRack, s4, saddle, wb27, wf27], neg=[~b1, ~b2, ~c2, ~c4, ~f2, ~f3, ~h2, ~h3, ~h4, ~h5, ~r1, ~r2, ~r3, ~s1, ~s2, ~s3, ~wb26, ~wb29, ~wb32, ~wf24, ~wf26, ~wf29, ~wf32]}
Assignment{pos=[b3, backWheel, bell, c1, color, f1, frame, frontWheel, h4, handlebar, luggageRack, s4, saddle, wb27, wf27], neg=[~b1, ~b2, ~c2, ~c4, ~f2, ~f3, ~h1, ~h2, ~h3, ~h5, ~r1, ~r2, ~r3, ~s1, ~s2, ~s3, ~wb26, ~wb29, ~wb32, ~wf24, ~wf26, ~wf29, ~wf32]}
```


### Projected Model Enumeration

The valid configurations distinguish only in their handlebars and saddles, because Beatrice has already decided on all other features (frame, color, wheel size, color).  One can also only enumerate over the features of interest.  For example Beatrice is now only interested which different saddles she can sell with the above configuration.  This is called _projected model enumeration_.  For this, Beatrice gives the method `enumerateAllModels()` a collection of variables over which the enumeration should be performed - in this case only the saddles.  The solver is the one defined above, with some features already added.

```java
List<Variable> variables = Arrays.asList(data.s1, data.s2, data.s3, data.s4); // (1)!
List<Assignment> modelsProjected = solver.enumerateAllModels(variables);
```

1. all possible saddles

The result is:

```
Assignment{pos=[s4], neg=[~s1, ~s2, ~s3]}
Assignment{pos=[s2], neg=[~s1, ~s3, ~s4]}
Assignment{pos=[s3], neg=[~s1, ~s2, ~s4]}
```

Thus, there are 3 configuration options for the saddle for bikes in blue, with wheel size 27.5-inch, carbon frame and ladybug bike bell. They are:
- `s2`
- `s3`
- `s4`
And saddle `s1` is never buildable with this configuration.


## Performing Functions on the Solver

Beatrice is curious about what the minimum and maximum number of features for a bike is.
In order to find out, she executes an `OptimizationFunction` on the solver.  This function can be called with the `.minimize()` or `.maximize()` option.  For the minimum number of features in any configuration, she chooses the minimize function, and for the maximum number of features she chooses the maximize function.

``` java
SATSolver solver = MiniSat.miniSat(data.f);
propositions.forEach(solver::add);

SortedSet<Variable> vars = propositions.stream() // (1)!
        .map(p -> p.formula().variables())
        .flatMap(Collection::stream)
        .filter(v -> !data.featureClasses.contains(v))
        .collect(Collectors.toCollection(TreeSet::new));

Assignment bikeWithMinNumberOfFeatures = solver.execute(
        OptimizationFunction.builder().literals(vars).minimize().build());
```

1. all relevant features (without feature classes)

The result is

```
Assignment{
  pos=[c4, f3, h1, s4, wb32, wf32],
  neg=[~b1, ~b2, ~b3, ~c1, ~c2, ~f1, ~f2, ~h2, ~h3, ~h4, ~h5, ~r1, ~r2, ~r3,
       ~s1, ~s2, ~s3, ~wb26, ~wb27, ~wb29, ~wf24, ~wf26, ~wf27, ~wf29]
}
```

meaning that the following features have to be taken: `c4`, `f3`, `h1`, `s4`, `wb32`, `wf32` and therefore 6 features are the minimum.  There may be other configurations with the minimum number of features (6), but not a configuration with less than 6 features.  To compute the maximum number of features, Beatrice can just use the `.maximize()` function

``` java
Assignment bikeWithMaxNumberOfFeatures = solver.execute(
        OptimizationFunction.builder().literals(vars).maximize().build());
```

and gets a configuration with 7 positive variables.

Another option to finding the bike with the minimum number of components is using a [MaxSATSolver](../../documentation/solvers/maxsat-solving), as you will find out in the next chapter.

