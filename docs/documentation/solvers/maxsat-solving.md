---
title: MaxSAT Solving
---

Given an unsatisfiable formula in CNF, the MaxSAT problem is the problem of finding an assignment which satisfies the maximum number of clauses and therefore solves an optimization problem rather than a decision problem.  There are two extensions to MaxSAT Solving: 1) the distinction of hard/soft clauses, and 2) additional weighted clauses, yielding four different flavours of MaxSAT solving:

1. Pure MaxSAT
2. Partial MaxSAT
3. Weighted MaxSAT
4. Weighted Partial MaxSAT

In a **Partial MaxSAT problem** you can distinguish between hard and soft clauses.  A hard clause _must_ be satisfied whereas a soft clause should be satisfied but can be left unsatisfied.  This means the solver only optimizes over the soft clauses.  If the hard clauses themselves are unsatisfiable, no solution can be found.

In a **Weighted MaxSAT problem** clauses can have a positive weight.  The solver then does not optimize the *number of satisfied clauses* but the *sum of the weights of the satisfied clauses*.

The **Weighted Partial MaxSAT problem** is the combination of Partial MaxSAT and weighted MaxSAT.  I.e. you can add hard clauses and weighted soft clauses to the MaxSAT solver.

Note two important points:

1. MaxSAT can be defined as weighted MaxSAT restricted to formulas whose clauses have weight 1, and as Partial MaxSAT in the case that all the clauses are declared to be soft.
2. The above definitions talk about *clauses* on the solver, not arbitrary formulas.  In real-world use cases you often want to weight whole formulas and not just clauses.  LogicNG's MaxSAT solver API gives you this possibility and internally translates the formulas and their weights accordingly.  This is a huge quality-of-life improvement when working with the solvers.


## MaxSAT Algorithms in LogicNG

LogicNG's MaxSAT implementation is based on [OpenWBO](http://sat.inesc-id.pt/open-wbo/) and its underlying encodings with [PBLib](https://iccl.inf.tu-dresden.de/web/WVPub90/en).  Therefore, you can use different algorithms, depending on your MaxSAT flavour and your specific use case.  There are two orthogonal strategies to solve MaxSAT problems:

1. Based on linear search
2. Based on producing and iteratively relaxing unsatisfiable cores

Details on this distinction can be found in the paper [Algorithms for Weighted Boolean Optimization](https://arxiv.org/pdf/0903.0843.pdf) by Manquinho, Marques-Silva and Planes.

Here is an overview of the algorithms in LogicNG and their capabilities:

| Algorithm  | Description                   | Strategy      | Unweighted | Weighted |
|------------|-------------------------------|---------------|------------|----------|
| `LinearSU` | Linear Sat-Unsat              | Linear Search | yes        | yes      |
| `LinearUS` | Linear Unsat-Sat              | Linear Search | yes        | no       |
| `MSU3`     | Seminal-core guided algorithm | Unsat-Core    | yes        | no       |
| `WMSU3`    | Weighted MSU3                 | Unsat-Core    | no         | yes      |
| `WBO`      | Weighted Boolean Optimization | Unsat-Core    | yes        | yes      |
| `IncWBO`   | Incremental WBO               | Unsat-Core    | yes        | yes      |

All algorithms in LogicNG support partial MaxSAT.  `LinearUS` and `MSU3` do not support weighted clauses/formulas, `WMSU3` does not support unweighted clauses/formulas.

The `LinearSU` stands for Linear SAT-UNSAT and it means that all sat-calls on the SAT-Solver are SAT except for the last call. That means: We start with a version without soft formulas on the SAT solver (which is SAT as long as the conjunction of the hard formulas evaluate to `true`).  Then we add clauses as long as an UNSAT comes for the first time.  `LinearUS` stands for Linear UNSAT-SAT and works the other way around: All SAT-Solver calls are UNSAT and the last one is SAT.  So first we add all soft clauses, then remove clauses one by one until the SAT-call is SAT.

The two `MSU3` and `WMSU3` algorithms are based on unsatisfiable cores.  The two `WBO` and `IncWBO` algorithms are based on a CNF encoding of the pseudo-Boolean Optimization problem.

Which algorithm to use depends strongly on the specific use case and must be evaluated.


## Using MaxSAT Solvers in LogicNG

A MaxSAT solver in LogicNG is created by using its corresponding factory method in the class [MaxSATSolver](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/MaxSATSolver.java).  E.g. the method `incWBO(f)` generates a MaxSAT solver with the `IncWBO` algorithm.  Each factory method can have an optional parameter for the solver's configuration.

Hard formulas are added with the method `addHardFormula()`, soft clauses are added with `addSoftClause(formula, weight)` where the weight has to be positive.  For an unweighted clause, use the weight 1.  The method `solve()` solves the problem for the formulas on the solver and returns a `MaxSATResult` which has one of three states:

1. `UNSATISFIABLE`: The hard clauses on the solver are already unsatisfiable, optimization could not be performed
2. `OPTIMUM`: An optimal solution was found
3. `UNDEF`: The search was aborted by a handler and yielded no result

As with SAT solvers, a MaxSAT solve process can be customized with a handler which can be implemented by the user and aborts the solve process in certain conditions.

The method `model()` returns the model found by the solver when an optimal solution was found.  The method `result()`  returns the minimum weight (or number of clauses if unweighted) of clauses/formulas which have to be unsatisfied.  Therefore, if the minimum number of weights is 0, the formula is satisfiable.

We look at some examples on how to use the MaxSAT solver with the following simple set of formulas which are unsatisfiable together:

```
A & B & (C | D)
A => ~B
~C
~D
```


### Pure MaxSAT Solving

We generate a `MSU3` solver and add all formulas as soft clauses with weight 1.  Therefore, we have a pure MaxSAT problem without distinction between hard/soft clauses and without weights.

```java
MaxSATSolver solver = MaxSATSolver.msu3(f);
solver.addSoftFormula(f.parse("A & B & (C | D)"), 1);
solver.addSoftFormula(f.parse("A => ~B"), 1);
solver.addSoftFormula(f.parse("~C"), 1);
solver.addSoftFormula(f.parse("~D"), 1);

solver.solve();  // OPTIMUM
solver.model();  // Assignment{pos=[], neg=[~A, ~B, ~C, ~D]}
solver.result(); // 1
```

In this case we can find an optimal solution: at least one formula remains unsatisfied.  In this case the model assigns all variables to `false`, thus the first formula is unsatisfied and all other formulas are satisfied.

As we will see in the next section, this is the only way of unsatisfying _only one_ formula.  If the first formula holds, we have to unsatisfy at least two other formulas.


### Partial Weighted MaxSAT Solving

We generate a `WMSU3` solver and add once again the first formula as hard clause and all other formulas as soft clauses with different weights.  Therefore, we have a partial weighted MaxSAT problem.

```java
MaxSATSolver solver = MaxSATSolver.wmsu3(f);
solver.addHardFormula(f.parse("A & B & (C | D)"));
solver.addSoftFormula(f.parse("A => ~B"), 2);
solver.addSoftFormula(f.parse("~C"), 4);
solver.addSoftFormula(f.parse("~D"), 8);

solver.solve();  // OPTIMUM
solver.model();  // Assignment{pos=[A, B, C], neg=[~D]}
solver.result(); // 6
```

In this case we can find an optimal solution once again.  Since the fourth formula now has a large weight, the incentive to satisfy it is larger than the incentive to satisfy the third formula.  So in contrast to the last section the model now includes `C` and *not* `D`, thus unsatisfying the third formula.

The result in this case is 6 - the sum of weights of all unsatisfied formulas: the second formula with weight 2 and the third formula with weight 4.

!!! info "Multiple Optimal Solutions"
    Note that this is the only optimal solution for this problem. In general, however, there can be multiple optimal solutions.


### Special Cases

Last we consider two special cases.  First we try to solve a formula where the hard clauses are already unsatisfiable:

```java
MaxSATSolver solver = MaxSATSolver.msu3(f);
solver.addHardFormula(f.parse("A & B & C"));
solver.addHardFormula(f.parse("A => ~B"));
solver.addHardFormula(f.parse("~C"));
solver.addSoftFormula(f.parse("X | Y"), 1);

solver.solve();  // UNSATISFIABLE
solver.model();  // null
solver.result(); // -1
```

In this case the `solve()` method returns the state `UNSATISFIABLE`.  There is no model and the result is -1.

Now we try a formula which is satisfiable:

```java
MaxSATSolver solver = MaxSATSolver.msu3(f);
MaxSATSolver solver = MaxSATSolver.msu3(f);
solver.addHardFormula(f.parse("A & B & C"));
solver.addSoftFormula(f.parse("~C | D"), 1);
solver.addSoftFormula(f.parse("X | Y"), 1);

solver.solve();  // OPTIMUM
solver.model();  // Assignment{pos=[A, B, C, D, X], neg=[~Y]}
solver.result(); // 0
```

The solver found an optimal solution in this case.  Further, the result is 0 - indicating that *all* formulas were satisfied and no formula had to be unsatisfied.


## Minimum vs. Maximum: Solving the Dual Problem

As seen above, the solver tries to *maximize* the number of satisfied clauses by *minimizing* the number of unsatisfied clauses. The returned result is the sum of weights of the *unsatisfied* clauses.

But what when you want to know the minimal number of satisfied clauses?  Then you have to adjust your problem modelling accordingly.  Let us consider a small example:

```java
MaxSATSolver solver = MaxSATSolver.wmsu3(f);
solver.addHardFormula(f.parse("(A & B & ~X & ~D) | (B & C & ~A) | (C & ~D & X)"));
solver.addSoftFormula(f.parse("A"), 2);
solver.addSoftFormula(f.parse("B"), 4);
solver.addSoftFormula(f.parse("C"), 8);
solver.addSoftFormula(f.parse("D"), 5);
solver.addSoftFormula(f.parse("X"), 7);
```

We have one hard formula and each variable in it has a given weight.  Solving the problem yields the assignment `Assignment{pos=[B, C, D, X], neg=[~A]}` with result 2. This means when setting all variables except `A` to `true`, only one formula (in this case variable) is unsatisfied and has weight 2.

If we give the variable weights a meaning, e.g. a price, then we have now computed the *most expensive* variable assignment.  `B`, `C`, `D`, `X` together have a price of 24.  In order to compute the *cheapest* variable assignment, we should not try to maximize the positive literals, but instead the negative literals: We want to set a maximal number of expensive variables to `false`.  We can do this by negating the variables in the soft formulas:

```java
MaxSATSolver solver = MaxSATSolver.wmsu3(f);
solver.addHardFormula(f.parse("(A & B & ~X & ~D) | (B & C & ~A) | (C & ~D & X)"));
solver.addSoftFormula(f.parse("~A"), 2);
solver.addSoftFormula(f.parse("~B"), 4);
solver.addSoftFormula(f.parse("~C"), 8);
solver.addSoftFormula(f.parse("~D"), 5);
solver.addSoftFormula(f.parse("~X"), 7);
```

Now the model is `Assignment{pos=[A, B], neg=[~C, ~D, ~X]}` with a result of 6.  This means the cheapest variable assignment consists of only `A` and `B` with a price of 6.

Once you wrap your head around this concept of duality you have a very mighty tool to solve all kinds of optimization problems with MaxSAT solving.

