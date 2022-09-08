---
title: Solvers
---

One of the core features offered by LogicNG are (SAT) *Solvers*.

Right now, there are two types of solvers:

1. **SAT Solvers**, which, in their basic form, can compute whether a formula is satisfiable or not and therefore solving a decision problem.  However, there are many algorithms which can compute more complex things by iteratively calling and modifying a SAT Solver and LogicNG provides many extensions and functions on SAT solvers.  In [the next chapter](sat-solving) we describe the different SAT Solvers implemented in LogicNG as well as the algorithms implemented around them.
2. **MaxSAT Solvers** on the other hand can directly be used for optimization.  The question they answer is how many clauses in an unsatisfiable formula can be maximally satisfied.  In a partial MaxSAT problem one can also differentiate between "hard" and "soft" formulas.  Hard formulas are required to be satisfied while soft formulas are not, but the MaxSAT Solver tries to satisfy as many of them as possible.  Additionally soft formulas can also have a *weight*.  In this case the solver tries to satisfy the formulas with the maximum global weight.  MaxSAT solvers are presented in [the chapter on MaxSAT](maxsat-solving).

