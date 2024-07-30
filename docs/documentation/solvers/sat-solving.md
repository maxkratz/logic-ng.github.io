---
title: SAT Solving
---

The SAT problem is the problem of deciding whether a formula in Boolean logic is satisfiable or not. In other words, does there exist a variable assignment under which the formula evaluates to `true`?

A SAT solver is a tool that, given a formula `f1`, returns `SAT` if `f1` is satisfiable, and `UNSAT` otherwise.

In order to understand the theoretical background for this chapter more in-depth, we recommend reading section 2.2. in [New Formal Methods for Automotive Configuration](https://publikationen.uni-tuebingen.de/xmlui/bitstream/handle/10900/57198/dissertation.pdf?sequence=1&isAllowed=y). The concepts of *unit clauses* and *empty clauses* are also explained in this chapter.


!!! summary "A Short History on SAT Solving"
    The satisfiability problem has been around for more than 60 years.  It was the first problem proved to be NP-complete by Stephen Cook in 1971.  The first algorithms to tackle it even date back to the 1960s.  Most modern SAT solvers work only on the CNF of a formula, that's why efficient CNF transformations are a very important topic in LogicNG.

    In 1962, the researchers Martin Davis, George Logemann and Donald W. Loveland developed the first algorithm that solved the SAT problem. It builds up on results of a previous collaboration from Davis and Hilary Putnam, and is thus called the DPLL algorithm.  The basic idea of DPLL is to have two phases: a decision phase where the algorithm takes a yet unassigned variable of the formula and assigns it to true or false and then a deduction phase in which the algorithm tries to infer new variable assignments from unit clauses (an unsatisfied clause where only one literal is unassigned and its assignment therefore is forced). These inferred assignments are also called "(unit) propagations". If, during this deduction phase, an empty clause (an unsatisfied clause where all literals are assigned) is detected, the last decision must have been wrong, so it is turned into a unit propagation with the opposite assignment. At the end, the algorithm either finds a satisfiable assignment of all variables or an empty clause at the "beginning" (where no decision can be undone). DPLL was improved over the years by finding better heuristics how to choose the next variable and what value to assign it to.

    About 15 years after the DPLL algorithm was developed, two research groups independently presented a new algorithm, the *Conflict-Driven Clause Learning* (CDCL) algorithm which was the largest break-through in SAT Solving to date. The idea is that whenever a conflict is reached in the search tree of DPLL, a new clause is *learned* from this conflict. This clause is added to the SAT solver's clause set, so the solver does not run in the same conflict again. Furthermore, the algorithm does not just jump back to the last decision, but rather detects the last decision which was actually relevant for the conflict. Together with clause learning, other important improvements on the technical side were discovered, like the 2-watched literals scheme, new activity-based search heuristics, search restarts and clause deletion. This all cumulated in the implementation of [MiniSat](https://github.com/niklasso/minisat) in 2004. Its implementation counts less than 1,000 lines of code and was very clear and well documented. Many researchers based their work on MiniSat; also the solvers in LogicNG are all based on MiniSat.


## The SAT Solvers in LogicNG

There are three SAT solvers implemented in LogicNG: _MiniSat_, _Glucose_ and _MiniCARD_.  MiniSat has as its base the original MiniSat implementation translated and adjusted to Java.  Glucose is a more recent extension of MiniSat.  The cardinality solver MiniCARD deals more efficiently with cardinality constraints than normal SAT solver.

In LogicNG, [SATSolver](https://github.com/logic-ng/LogicNG/tree/master/src/main/java/org/logicng/solvers) is the super class, which is extended by [MiniSat](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/MiniSat.java). It is possible to implement other types of SAT solvers (e.g. `CleaneLing`, which was present in early LogicNG versions), which would then also extend `SATSolver`.

This chapter aims to provide detailed information about how to use the SAT solvers implemented in LogicNG. The structure of this chapter is the following:

1. [Solver Overview](#solver-overview)
2. [Generating SAT Solvers](#generating-sat-solvers-in-logicng)
3. [Incremental/Decremental Interface](#incrementaldecremental-interface)
4. [Methods on SAT Solvers](#methods-on-sat-solvers)
5. [Solver Configuration](#solver-configuration)
6. [Functions on Solvers](#solver-functions)


## Solver Overview

Since all three solvers are based on MiniSat, there is a base class [MiniSatStyleSolver](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/sat/MiniSatStyleSolver.java) where those methods common to all solvers are implemented.  Each solver then implements its own sub-class from this class.

### MiniSat solver

The developer of MiniSAT say about their solver:

!!! quote "MiniSat"
    MiniSat is a minimalistic, open-source SAT solver, developed to help researchers and developers alike to get started on SAT.

    -- *Niklas Eén, Niklas Sörensson*

Check out MiniSat's [website](http://minisat.se/).  The core implementation of [MiniSat](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/sat/MiniSat2Solver.java) in LogicNG follows the original source code quite closely.  This is done on purpose and simplifies the implementation of tools based on MiniSat which there are many.  However, of course there had to be adjustments made for Java and there are many features already implemented in LogicNG's MiniSat version which are not present in the original code, among them:

- An incremental and *decremental* interface for saving and loading the solver state in order to add and remove formulas (more on that later)
- Optimized backbone computation directly on the solver
- Extraction of the literals propagated on level 0 and therefore direct consequences of the formula
- Integrated efficient incremental cardinality constraints
- Proof Tracing in the solver
- A generic function interface for executing self written functions directly on the solver


### Glucose solver

The developers of the Glucose solver say about their solver:

!!! quote "Glucose"
    Glucose is based on a new scoring scheme (well, not so new now, it was introduced in 2009) for the
    clause learning mechanism of so-called "Modern" SAT solvers (it is based on our IJCAI'09 paper). ...
    The name of the Solver name is a contraction of the concept of "glue clauses", a particular kind of clauses
    that glucose detects and preserves during search.

    -- *Gilles Audemard and Laurent Simon*

For more information about Glucose, check out their [GitHub](https://github.com/mi-ki/glucose-syrup).  As with MiniSat, LogicNG's implementation of [Glucose](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/sat/GlucoseSyrup.java) is heavily oriented on the original source code.  LogicNG's implementation has proof tracing like the original, but currently no incremental/decremental interface for Glucose.


### MiniCARD solver

MiniCARD is a *cardinality solver* based on MiniSat. A cardinality solver is a SAT solver which deals more efficiently with cardinality constraints than normal SAT solvers.

The inventor of MiniCARD says about his solver:

!!! quote "MiniCard"

    MiniCARD handles cardinality constraints natively, using the same efficient data
    structures and techniques MiniSat uses for clauses, giving it much better
    performance on cardinality constraints than CNF encodings of those constraints
    passed to a typical SAT solver.

    -- *Mark Liffiton*

For more information, check out [GitHub](https://github.com/liffiton/minicard).  LogicNG's implementation of [MiniCARD](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/sat/MiniCARD.java) does implement the incremental/decremental interface, but it does currently not support proof tracing.


## Generating SAT Solvers in LogicNG

The SAT solvers can be generated with factory methods on the `MiniSat` class, i.e.

```java
SATSolver miniSat = MiniSat.miniSat(f);   // Generate a new MiniSat solver
SATSolver glucose = MiniSat.glucose(f);   // Generate a new Glucose solver
SATSolver miniCard = MiniSat.miniCard(f); // Generate a new MiniCARD solver
```

Further, you may also create a SAT solver with your own *configuration*. In order to do so, specify the parameters in a [MiniSatConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/sat/MiniSatConfig.java) or a [GlucoseConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/sat/GlucoseConfig.java) and give it as a parameter to the solver. See below for information on those parameters.

Example of creating a MiniSat solver with activated proof tracing:

```java
MiniSatConfig config = MiniSatConfig.builder().proofGeneration(true).build();
SATSolver solver = MiniSat.miniSat(f, config);
```

For the rest of this chapter, we consider the solver `SATSolver solver = MiniSat.miniSat(f)`.


## Incremental/Decremental Interface

There are applications where SAT solvers get large formulas for which the satisfiability is computed over hours.

!!! tip "Application Insight"
    However, there is a completely different use case, for which the incremental/decremental interface has been developed:  You have a large satisfiable formula and want to check for _hundreds of thousands_ of small formulas, whether their conjunction with the large formula is still satisfiable.

For this, it is useful to be able to not only add formulas to the solver (incremental interface), but also *delete* formulas from the solver (decremental interface) without resetting the whole solver state or using auxiliary variables for clause activation/deactivation.

The incremental and *decremental* interface is one of the perks of LogicNG. You may use this interface by saving the state of your current solver at different times, and returning to it from later time points.  The state contains only the lengths of all internal data structures e.g. all stored variables, clauses, learnt clauses and so on. This makes the saving and loading state very efficient, since saving is essentially just storing the length of arrays and loading is shrinking those arrays to their stored length.

Let's look at an example:

``` java
Formula f1 = p.parse("A & B & C");
solver.add(f1);
solver.sat();   // TRUE

SolverState initialState = miniSat.saveState(); // (1)!
solver.add(p.parse("~A")); // (2)!
solver.sat();   // FALSE

solver.loadState(initialState); // (3)!
solver.add(p.parse("D")); // (4)!
solver.sat();   // TRUE
```

1. we save this state of the solver
2. we add a new formula to the solver, rendering it unsatisfiable
3. now we load the last solver state (with only `A & B & C` on it)
4. we add another formula to the solver

So with the first `sat()` call we checked the formula `A & B & C` for satisfiability, in the second call we checked `A & B & C & ~A` and in the last call we checked `A & B & C & D`.  But the original formula `A & B & C` was added only once.  This has a few critical advantages: we do not need to reset and refill the solver, and especially: everything learned on the original formula before the `saveState()` is kept between different SAT calls.

Since the save/load state operation only works on the length of data structures and not on their actual content (this would be far to inefficient), _it is not possible to go to a future state once you loaded an earlier state_.  This means that you can only load states older than the current one.  So you can think of the states of a SAT solver as a stack.  You can pop elements of the stack and go to older states, but you can never access an already popped element.

Consider the following example:

``` java
Formula f1 = p.parse("A & B & C");
miniSat.add(f1);
SolverState initialState = miniSat.saveState();

miniSat.add(p.parse("~A"));
SolverState nextState = miniSat.saveState();

miniSat.loadState(initialState); // (1)!
miniSat.loadState(nextState); // (2)!
```

1. Loading the initial state is possible
2. This is not possible, since you reverted "nextState" and can not go "back forward" to it

On the other hand, you can go back to the same state repeatedly as long as you don't go back before it.

``` java
Formula f1 = p.parse("A & B & C");
miniSat.add(f1);
SolverState initialState = miniSat.saveState();

miniSat.add(p.parse("~A"));
miniSat.sat();

solver.loadState(initialState); // check another formula
solver.add(p.parse("~B"));
solver.sat();

solver.loadState(initialState); // check another formula
solver.add(p.parse("~C"));
solver.sat();
```

### Methods on SAT Solvers

There is not only the possibility to solve the current formula and get SAT or UNSAT as result - that is one of the core functions of a SAT solver, but LogicNG's solvers implement far more methods that help users tackle different problems.


#### Adding Formulas and Propositions

Once the solver is created, you have to add formulas or propositions to it. The SAT solver then checks whether the conjunction of all added formulas is satisfiable.

The standard way to add formulas or propositions to the solver is the method `add()`. The formula is then, depending on the parameter `cnfMethod` in the SAT solver configuration, transformed into an at least equisatisfiable CNF.  Note that [equisatisfiability instead of equivalence](../../formulas/operations/transformations) is enough at this point.  If you have not specified this parameter, this is done using the [Plaisted-Greenbaum transformation](../../formulas/operations/transformations/normal-form-transformations#plaisted-greenbaum-transformation).  However, if the introduction of auxiliary variables is not wanted, you can use the [CNF factorization](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/cnf/CNFFactorization.java) or [BDD CNF transformation](../../formulas/operations/transformations/normal-form-transformations#bdd-cnf-transformation), which both yield a semantically equivalent CNF without auxiliary variables.

So in general you don't have to convert the formula to CNF before you add it to the solver - the solver does this on its own.  In fact: Unless you need the CNF at other points in you application, it is wise *not* to generate it manually, because then it is created on the formula factory.  If the solver creates the CNF, it generates it directly on the solver which is more performant and especially requires less heap and can be garbage collected when the solver is destroyed.  Therefore, in our use cases we almost never transform formulas to CNF before adding them to the solver unless there is a very specific reason for it.

Example which adds the formula `f1` to the solver:

```java
Formula f1 = p.parse("A & (~B | C)");
solver.add(f1);
```

Using `add()`, one can also add a list of formulas or a single proposition to the solver. Using `addPropositions()`, one can add a list of propositions to the solver (due to Java's type erasure these have to be two different methods).

You can also add cardinality constraints directly to the solver.  As with CNFs, it is a good idea _not_ to convert cardinality constraints to CNF before adding them to the solver.  When using MiniSat or Glucose, they are also only converted on the solver directly, so they are not polluting the formula factory.  When using MiniCARD, they are added to the solver as special clauses and can thus be processed more efficiently.  If you have many cardinality constraints in your problem, it is worth considering using MiniCARD. Depending on your specific use case, MiniCARD can yield performance advantages but does not provide all functionality which MiniSat does provide


#### Solving

The method `sat()` without further arguments solves the conjunction of all formulas which currently lie on the solver.  There are two *optional* parameters for the `sat()` call:

- a set of literals(s): the formula on the solver will be solved, and these literals act as so-called "assumptions" (c.f. Solving with Assumptions)
- a handler: allows you to control the solving process by providing own handlers

Every combination with/without literals and with/without handler leads to a valid `sat()` call.  We call solving without parameters a "standard solve".  One can also solve with a fixed variable ordering `satWithSelectionOrder()` (see below).   The result of the `sat()` and `satWithSelectionOrder()` call is an instance of the class `Tristate`: `TRUE`, `FALSE` or `UNDEF`.  When the formula on the solver is satisfiable, the `sat()` call yields `TRUE`, when it is unsatisfiable, it yields `FALSE`.  The result is `UNDEF` if the solving process did not finish, and this happens only if a handler aborted the computation.

If no arguments are provided, the formula on the solver is solved without assumptions and with no handler and always yields a `TRUE` or `FALSE` result.

Example for the "standard solve":

``` java
Formula f1 = p.parse("A & (~B | C)");
solver.add(f1);
Tristate sat = solver.sat();
```


##### Solving with Assumptions

When the `sat()` method is called with a list of literals, the result of this call indicates whether the formula on the solver in conjunction with these literals is satisfiable or not.  This is called _assumption solving_ or _solving with assumptions_.  Of course one would reach the same result with the incremental/decremental interface: Save the state, add the literals to the solver, and loading the state again.  But assumption solving is more efficient since the assumption literals can be treated specially during solving and there is no explicit save/load state involved.  _Assumptions can only be literals and not arbitrary formulas_ - for arbitrary formulas, use save/load state.

There are many use cases for assumption solving, e.g. a large formula lies on the solver and one wants to check many assignments against this formula.  However, assumption solving has also a disadvantage: one can not generate a proof trace after solving with assumptions.  There you still need to use the save/load state mechanism.

As an example, consider

``` java
Formula f1 = p.parse("A & (~B | C)");
solver.add(f1);
```

Then `#!java solver.sat(f.literal("A", true))` checks whether `f1` is satisfiable when `A` is assigned to `true`. The result is `TRUE`, but the result of `#!java solver.sat(f.literal("A", false))` is `FALSE`.

Similarly, one can add multiple literals to the SAT solver in the following way:

``` java
List<Literal> literals =
        Arrays.asList(f.literal("B", true), f.literal("C", false));
Tristate sat = solver.sat(literals);
```

The result of the last call is `FALSE` (because of `f1`).


##### Solving with Handlers

Further, one can control the solving process by using a SAT handler. For example, the `TimeoutSATHandler` cancels the solving process after a given timeout.  But there are also more advanced handlers which can count the number of conflicts or the number of restarts of the solver instance.  For more information about the handlers check out [this chapter](../../handlers).

As an example, the following code calls the solver and aborts the computation after 100 ms:

``` java
Tristate sat = solver.sat(new TimeoutSATHandler(100));
```

!!! info "Handler Abortion"
    Note that in this case when the solver can not compute the result within these 100 ms, the result will be `UNDEF`.


##### Solve with a Variable Selection Order

Let's reconsider the solving process of the SAT solvers: After each unit propagation step (which did not yield a conflict), the SAT solver makes a "decision", i.e. it chooses some variable and assigns it to a certain value (`true` or `false`) before starting unit propagation again. The order in which the variables are chosen and assigned to is usually an efficient activity heuristic.

However, it is possible to change this order. This can be useful in certain circumstances when you e.g. have certain domain knowledge about your problem and know you can provide a better selection order.  *Usually using a fixed order is slower than relying on the internal heuristics.*

Using `satWithSelectionOrder()` one can hand over

1. A selection order
2. A SAT handler (optional, see above)
3. Assumptions (optional, see above)

The selection order does not need to contain all variables known by the solver. After the solver has used all variables from the selection order, it will continue to choose from the remaining variables using its internal heuristics.

Alternatively, if you want to use the same selection order over multiple `sat()` calls, you can set a selection order via `setSelectionOrder()` and later potentially reset it again with `resetSelectionOrder()` s.t. the solver will use its own heuristics again.


#### Getting Models from the Solver

If the solver is `SAT`, you might be interested in the assignment (or "model") for the formula which the solver found.  The call to `model()` returns a model for the formulas which lie on the solver.  For example, the following code snippet returns a model for the formula `f1`:

``` java
MiniSat miniSat = MiniSat.miniSat(f);
Formula f1 = p.parse("A | (~B & C)");
miniSat.add(f1);
minisat.model() // returns the model `~A, ~B, C`
```

When not changing any parameters, all of LogicNG's solvers are deterministic, i.e. when filled with the same formula of the same factory they will always produce the same model.  This is critical for industrial applications.

You can get a list of all valid assignments using `#!java solver.enumerateAllModels()`. Optionally, you can restrict the solver to specific variables by handing over those variables as parameters to this method call. For more information about this, see the [chapter on model counting](../../model-counting-enumeration).  However, use this function very careful: even for few variables, the number of solutions can grow exponentially and therefore get fast out of hands.

!!! warning "Unsatisfiable Formulas"
    Note that it does not make sense to call a model when the conjunction of clauses which have been added to the solver is unsatisfiable. The method returns `null` in this case. If the solver is `UNSAT`, then you can compute an unsatisfiable core of the formulas on the solver by calling the `unsatCore()` method.  This method can only be called on MiniSat and Glucose when they were configured with `proofTracing = true` (this function is turned off by default).  More about this in the chapter [explanations](../../explanations).


#### Further Useful Methods

- Compute the [backbone](../../backbones) of a formula using `backbone()`
- Reset the solver using `reset()`, if you want to hold on to the configurations but get rid of all formulas
- Return the formula factory of this solver using `factory()`


## Solver Configuration

All SAT solvers implemented in LogicNG can be configured with many parameters.  MiniSat and MiniCARD can both be configured using `MiniSatConfig`, and `Glucose` can be configured using both the `MiniSatConfig` and the dedicated `GlucoseConfig`.  The parameters for the `GlucoseConfig` are those of the original implementation.  We recommend changing those only if you have an in-depth knowledge about the Glucose SAT solver.  The `MiniSatConfig` holds some LogicNG-specific parameters, which we recommend using. The other parameters are those of the original MiniSat implementation, which we again recommend to touch only if you know what you're doing. :)

The following LogicNG specific fields in the `MiniSatConfig` can be changed:


### Incrementality

The parameter `incremental` functions differently on MiniSat/MiniCARD vs. Glucose.  On MiniSat/MiniCARD it activates the incremental/decremental API.  So when this parameter is set to `true`, you can use the above-mentioned `saveState()` and `loadState()` methods.  Internally, this deactivates clause deletion since the save/load state functionality would not function with clause deletion.  Therefore, if you do not want to use the decremental/incremental interface, setting this parameter to `false` may yield some performance improvements.

Glucose in LogicNG does not implement the save/load state functionality, but provides its own version of an incremental/decremental interface.  The parameter `incremental` does activate this mode on Glucose.  It is based on the idea of adding clauses with selector variables and activating and deactivating them via assumptions.  Details on this approach can be found in chapter 6 of this [Glucose overview](https://hal-univ-artois.archives-ouvertes.fr/hal-03299473/document) which is also a very nice overview of current techniques in modern SAT solving.


### Initial Phase

As mentioned above, a SAT solver has two phases: the decision phase and the deduction phase.  In the decision phase, a yet unassigned variable is chosen by some heuristics or a fixed order and then assigned to a truth value.  To which value the variable is initially set is decided by the parameter `initialPhase`.  If the parameter is set to `true`, every new variable is assigned to `true` first. Otherwise, every new variable is assigned to `false` first. In the actual (resulting) models this initial assignment can of course be changed through backtracking.  This parameter is available for all solvers.


### Proof Generation

The parameter `proofGeneration` indicates whether information for generating a proof is collected during the solving process.  In order to generate a proof, some internal information has to be stored during solving, therefore slowing the solving process a bit.  Therefore, the initial value for this parameter is `false`.

!!! info "Proof Generation"
    Note that the parameter only steers whether this information is recorded or not.  The actual proof will only be generated when the `unsatCore()` method on the solver is called.  More information on proof tracing can be found in the chapter on [explanations](../../explanations).  Proof tracing in LogicNG is only available for the MiniSat and Glucose solvers.


### CNF Transformation

The first step when adding a formula to a SAT solver is to transform the given formula to CNF.  The parameter `cnfMethod` decides how this is done.  There are three options:

1. `FACTORY_CNF`: Calls the `cnf()`-method on the formula. Thus, the CNF encoding of the underlying formula factory is used. This can be configured, see the [relevant chapter](../../formula-factory).  This approach generates all CNFs on the formula factory.  If you do not need the CNF outside the SAT solver, this approach should be avoided since there is no advantage of having all the formulas on the factory.
2. `PG_ON_SOLVER`: Performs the Plaisted-Greenbaum transformation directly on the solver, therefore never generating the CNF on the formula factory.  However, it still generates the NNF of the formula on the factory.
3. `FULL_PG_ON_SOLVER`: Performs Plaisted-Greenbaum transformation directly on the solver and does not transform the formula to NNF first.

Which CNF method to use heavily depends on the application.  Options 2. and 3. do not pollute the formula factory with the generated CNF (nor with the NNF in option 3.), but therefore do not cache the generated normal forms. Thus, if you add the formula to e.g. different solvers, the CNF/NNF will be re-generated every time.  Choosing option 1. caches both CNF and NNF in the formula factory.  Therefore, after the first computation, the normal forms are never computed again.

This parameter is available for all solvers and is set by default to `PG_ON_SOLVER` - in our experience a good trade-off.


### Auxiliary Variables in Models

The parameter `auxiliaryVariablesInModels` controls whether auxiliary variables which occur in the encoding of CNFs, cardinality constraints, or pseudo-Boolean constraints (e.g. `@RESERVED_CNF_`, `@RESERVED_CC_`, or `@RESERVED_PB_`) will also appear in the model which is created by the solver.  Usually one is not interested in these variables since they are generated automatically.  Therefore, this parameter is set to `false` by default.  This option is available for all solvers.


### Clause Minimization

The parameter `clauseMin` controls how to minimize the learnt clauses.  This is a parameter present in the original MiniSat implementation. We mention this parameter, since it is a relatively easy way to control the solving process and can lead to performance improvements. The options are local minimization (`BASIC`), recursive minimization (`DEEP`) and no minimization (`NONE`). The parameter is available for all solvers and the default is `DEEP`.


## Solver Functions

Solver functions can be executed directly on the SAT solver and access its internal state.  You can implement your own functions, and some functions are implemented in LogicNG:

- [FormulaOnSolverFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/functions/FormulaOnSolverFunction.java) returns the current formula on the solver.

!!! info "Formula on Solver"
    Note that this formula is usually syntactically different from the formulas which were actually added to the solver, since the formulas are added as CNF and may be simplified or even removed, depending on the state of the solver.  Furthermore, the solver might add learnt clauses or propagate literals.  This can be useful to debug or analyze a solver at a given time.
- [UpZeroLiteralsFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/functions/UpZeroLiteralsFunction.java) returns all unit propagated literals on level 0 of the current formula on the solver.  These are direct implications from the original formula and a subset of the [backbone](../../backbones).
- [UnsatCoreFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/functions/UnsatCoreFunction.java) computes the [Unsat Core](../../explanations/unsat-cores) if the formula is unsatisfiable. The function is used for the method `unsatCore()`.
- [BackboneFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/functions/BackboneFunction.java): This function computes a [backbone](../../backbones) for the formula on the solver. The function is used for the `backbone()` call. However, here it is not possible to add a handler to the solver. This is why it can be useful to use the solver function directly.
- [ModelEnumerationFunction](https://github.com/logic-ng/LogicNG/tree/master/src/main/java/org/logicng/solvers/functions): The solver function for model enumeration. The function is used for the `enumerateAllModels()` call. For more information see [the chapter on model counting and enumeration](../../model-counting-enumeration).
- [OptimizationFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/functions/OptimizationFunction.java): This function can be used to compute a minimal or maximal model in the number of positive assigned literals.  I.e. when minimizing over a set of variables you can compute a model with a globally minimal number of positive assigned literals.  Of course this could be performed with [MaxSAT solver](../maxsat-solving), but sometimes you already have a SAT solver with the right formulas at hand, and it is more efficient to use it instead of generating a new MaxSAT solver.

An example for the `OptimizationFunction` is the following: We want to return the assignment with a minimal number of positive literals.

``` java
Formula formula = p.parse("(A | B) & (~C | D)");
solver.add(formula);
Assignment assignment = solver.execute(OptimizationFunction.builder()
        .literals(f.variable("A"), f.variable("B"), f.variable("C"))
        .minimize()
        .build());
```

In this example, either `A` or `B` can be set to `false` but not both.  The second clause can be satisfied by setting `C` to `false`.  Thus, two of the three desired literals can maximally be set to `true`.  The result in this case is `~A, B, ~C`. However, this result is not unique: `A, ~B, ~C` would also be an optimal solution.
