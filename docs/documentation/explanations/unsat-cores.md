---
title: Unsat Cores
---

For an unsatisfiable set of formulas, the [UNSATCore](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/explanations/UNSATCore.java) is a set of propositions which the SAT Solver found to be unsatisfiable during its search.  As described in the [introduction chapter](..), this set does not need to be minimal.


## Computing the Unsat Core

Internally, the unsat core is being computed by the [UNSATCoreFunction](https://github.com/logic-ng/LogicNG/blob/994294fbb283b7b96230da3937369bf8fc062b5f/src/main/java/org/logicng/solvers/functions/UnsatCoreFunction.java) directly on the solver.  This only works for MiniSat and Glucose solvers and only if the parameter `proofTracing` is explicitly set to `true`.  Additionally, when using a Glucose solver, the `incremental` mode has to be deactivated.  As default, proof tracing is disabled since it requires slightly more time and memory.

As an example, consider the following list of propositions:

``` java
List<Proposition> props = new ArrayList<>();
props.add(new StandardProposition(f.parse("~A")));
props.add(new StandardProposition(f.parse("A | ~B")));
props.add(new StandardProposition(f.parse("B")));
props.add(new StandardProposition(f.parse("~B | C")));
props.add(new StandardProposition(f.parse("~C | D")));
props.add(new StandardProposition(f.parse("~D | E")));
props.add(new StandardProposition(f.parse("~C | E")));
props.add(new StandardProposition(f.parse("~E")));
```

When adding these formulas to a solver and solving them, the result is `FALSE`.  Now we can extract the unsatisfiable core with

``` java
Tristate sat = solver.sat(); // (1)!
UNSATCore<Proposition> unsatCore = solver.unsatCore();
```

1. `sat()` call is required and the result has to be `FALSE` before calling `unsatCore()`

The result is:

```
UNSATCore{
  isMUS=false,
  propositions=[
    StandardProposition{formula=~A, description=},
    StandardProposition{formula=A | ~B, description=},
    StandardProposition{formula=B, description=}
  ]
}
```

The first flag `isMUS` indicates whether the computed unsat core is known to be a [MUS](../mus). To make sense of this, recall that a `MUS` is always an unsat core. Thus, also the `MUS` algorithms return an UNSATCore. In the case of computing a `MUS` with a `MUS` algorithm, the flag `isMUS` is `true`. When an unsat core has been extracted from the solver, at this point there is no guarantee that it is a `MUS`. Hence, in this use case the flag is always `false`.

The set of propositions in this core is:

```
~A
A | B
~B
```

In fact, we can see that this set of formulas contains a contradiction.  And it only has 3 formulas compared to the original 8 formulas.  In this case, it is even a MUS and a SMUS but this is a coincidence.

!!! info "Practical Observation"

    Normally, the unsatisfiable core of a SAT solver does not have many redundant terms and usually reduces the original formulas/propositions significantly.
