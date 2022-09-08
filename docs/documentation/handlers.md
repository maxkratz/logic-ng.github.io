---
title: Handlers
---

Handlers in general are user provided objects to algorithms which allow to take control over certain aspects of the algorithm and stopping the computation if given criteria are met.  Especially the computationally expensive algorithms like SAT and MaxSAT Solving, knowledge compilation, formula simplification, or MUS computation take handlers.  You can either use one of the handlers which are implemented in LogicNG or implement your own handler. The hierarchy of the classes and interfaces in the package `handlers` is here:

``` mermaid
classDiagram
  Handler <|-- DnnfCompilationHandler
  Handler <|-- SATHandler
    SATHandler <|-- TimeoutSATHandler
  Handler <|-- MaxSATHandler
    MaxSATHandler <|-- TimeoutMaxSATHandler
  Handler <|-- FactorizationHandler
  Handler <|-- OptimizationHandler
    OptimizationHandler <|-- TimeoutOptimizationHandler
  Handler <|-- BDDHandler
    BDDHandler <|-- TimeoutBDDHandler
  Handler <|-- ModelEnumerationHandler
    ModelEnumerationHandler <|-- TimeoutModelEnumerationHandler
  Handler <|-- ComputationHandler
    ComputationHandler <|-- TimeoutHandler
    ComputationHandler <|-- NumberOfNodesBDDHandler
    ComputationHandler <|-- NumberOfModelsHandler
  TimeoutHandler <|.. TimeoutSATHandler
  TimeoutHandler <|.. TimeoutMaxSATHandler
  TimeoutHandler <|.. TimeoutOptimizationHandler
  TimeoutHandler <|.. TimeoutBDDHandler
  TimeoutHandler <|.. TimeoutModelEnumerationHandler
```

The interface [Handler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/Handler.java) has two methods which can be overridden:

1. `aborted()` returns whether the computation was aborted or not
2. `started()` is called whenever the computation is started and can be used to initialize the handler

A number of interfaces implement the `Handler`. The handler you're using has to implement the relevant interface, depending on what sort of computation you are performing.

These are the top level handler interfaces:

- [FactorizationHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/FactorizationHandler.java) is an interface for handling factorization methods for normal forms (CNF, DNF). It has two methods: `performedDistristibution()` is called each time a distribution is performed, `createdClause()` is called each time a new clause is created.
- [SATHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/SATHandler.java) is an interface for handling the solving process of a [SAT solver](../solvers/sat-solving). `detectedConflict()` is called each time a conflict is found, and `finishedSolving()` is called when the SAT solver finished solving.
- [MaxSATHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/MaxSATHandler.java) is an interface for handling the solving process of a [Max-SAT solver](../solvers/maxsat-solving).  It has itself a `SATHandler` which is used for its underlying SAT solver and provides the two methods `foundUpperBound()` and `foundLowerBound()` which are called whenever a new upper or lower bound is found in the solving process as well as the two methods `lowerBoundApproximation()` and `upperBoundApproximation()` which return the current approximation of the lower/upper bound of the problem.  These methods can e.g. be used to abort the computation when a certain bound is found or to return the current bound when the computation is aborted.
- [BDDHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/BDDHandler.java) is an interface for [BDD](../knowledge-compilation/bdd) compilation handlers.  The method `newRefAdded()` is called every time a new node reference is added to the BDD kernel.
- [DnnfCompilationHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/DnnfCompilationHandler.java) is an interface for [DNNF](../knowledge-compilation/dnnf) compilation handlers. The method `shannonExpansion()` is called each time a Shannon expansion is performed.
- [ModelEnumerationHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/ModelEnumerationHandler.java) is an interface for (projected) model enumeration on the SAT solver with the `ModelEnumerationFunction`.  It has its own `SATHandler` for its underlying SAT solver as well as a method `foundModel` which is called every time a model is found.
- [OptimizationHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/OptimizationHandler.java) is an interface for optimizations on the SAT solver.  It has a `SATHandler` for its underlying SAT solver as well as the method `foundBetterBound` which is called every time a new better bound for the optimization problem is found.  It is used in the `OptimizationFunction` of the SAT solver and in internal optimization calls in the `AdvancedSimplifier` .


## Some Useful Handlers Implemented in LogicNG

All handlers implemented in LogicNG inherit from the abstract class [ComputationHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/ComputationHandler.java). There are three classes extending the `ComputationHandler`:

1. [TimeoutHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/TimeoutHandler.java) for aborting computations based on their computation time.
2. [NumberOfNodesBDDHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/NumberOfNodesBDDHandler.java) for aborting BDD generations based on the number of generated nodes in the kernel.
3. [NumberOfModelsHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/NumberOfModelsHandler.java) for aborting model enumerations based on the number of enumerated model.


### The TimeoutHandler

The `TimeoutHandler` stops the computation after a specific time. The constructor of a `TimeoutHandler` takes a timeout in milliseconds and a `TimerType`, which specifies the type of the timeout. The timer types are:

- `SINGLE_TIMEOUT`: The timeout is started when `started()` on the handler is called.
Subsequent calls to `started()` have no effect on the timeout. Thus, the timeout can only be started once.
- `RESTARTING_TIMEOUT`: The timeout is restarted when `started()` on the handler is called.
- `FIXED_END`: Timeout which is interpreted as fixed point in time (in milliseconds)
at which the computation should be aborted. The method `started()` must still be called, but does not have an effect on the timeout.

!!! info "Handler Responsiveness"

    Note that it might take a few milliseconds more until the computation is actually canceled, since the cancellation depends on the next call to the handler and for performance reasons calls to the handler are only performed on certain points during the computation.  For the overloaded constructor, that takes only a timeout in milliseconds, the timer type is `SINGLE_TIMEOUT`.

A number of classes extend the `TimeoutHandler`. For those classes, the interpretation of the timer type is identical.
However, they implement different interfaces and can thus be used for different use cases:

- [TimeoutSATHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/TimeoutSATHandler.java) handles the solving process of a [SAT solver](../solvers/sat-solving).  It aborts the solving process when the time limit is exceeded.
- [TimeoutMaxSATHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/TimeoutMaxSATHandler.java) handles the solving process of a [MaxSAT solver](../solvers/maxsat-solving).  It aborts the solving process when the time limit is exceeded.
- [TimeoutOptimizationHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/TimeoutOptimizationHandler.java) handles optimization problems, for example the `OptimizationFunction` or the `AdvancedSimplifier` (cf. [here](../formulas/operations/transformations/simplifier-transformations)).
- [TimeoutBDDHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/TimeoutBDDHandler.java) handles the construction process of [BDDs](../knowledge-compilation/bdd).
- [TimeoutModelEnumerationHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/TimeoutModelEnumerationHandler.java) handles the process of enumerating models. For more information see the chapter on [Model counting and enumeration](../model-counting-enumeration).


### The NumberOfNodesBDDHandler

The `NumberOfNodesBDDHandler` cancels the build process of a [BDD](../knowledge-compilation/bdd) after a given number of added nodes is reached.


### The NumberOfModelsHandler

The `NumberOfModelsHandler` terminates the model enumeration process after a given number of models is reached. For more information see the chapter on [Model counting and enumeration|](../model-counting-enumeration).


!!! example "Creating your own Handler"

    Consider the following example for creating an own handler. We implement a `SATHandler` which stops the computation after a certain number of conflicts is reached. To make our implementation easier, we extend the `ComputationHandler` which already provides the `aborted` attribute.

    ``` java
    public class MaxNumberOfConflictsSATHandler extends ComputationHandler implements SATHandler {
        private final int maxNumberOfConflicts;
        private int numConflicts;

        /**
         * Constructs a new instance with the given maximal number of conflicts.
         * @param maxNumberOfConflicts the maximal number of conflicts limit,
         *                             if -1 then no limit is set
         */
        public MaxNumberOfConflictsSATHandler(final int maxNumberOfConflicts) {
            this.maxNumberOfConflicts = maxNumberOfConflicts;
            this.numConflicts = 0;
        }

        @Override
        public void started() {
            this.started();
            this.numConflicts = 0;
        }

        @Override
        public boolean detectedConflict() {
            this.aborted = maxNumberOfConflicts != -1 && ++numConflicts >= maxNumberOfConflicts;
            return !aborted;
        }
    }
    ```

