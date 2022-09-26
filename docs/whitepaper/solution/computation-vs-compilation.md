---
title: On-the-fly Computation vs. Compilation
hide:
  - toc
---

As shown in [the general approach](../general-approach), there are two fundamentally different approaches to answer queries on product rules: (1) using a solver engine, and (2) compiling the rules in a knowledge compilation format. LogicNG has support for both approaches. In our real-world projects in the automotive industry, however, we use solver engines > 95% of the time, and knowledge compilation only for special use cases. The big advantage of knowledge compilation is that compiled formats can be very succinct and allow for answering many questions very rapidly (often in constant or linear time). However, the computation time for the compilation itself can grow exponentially. A solver, on the other hand, can take exponential time to answer questions in the worst case, but it allows for very fast initial loading, addition and removal of rules. The following considerations indicate why the solver-approach is frequently superior in practice:

- Real industrial problems tend to be easily solvable for modern solver engines - often in a few milliseconds - rendering the performance advantage of knowledge compilations irrelevant.
- Large rule sets are very hard to compile into knowledge compilation formats. Typically taking minutes, often it is not even possible at all due to the exponential growth of compilation time.
- With some ground-breaking techniques been developed within the last twenty year, SAT solvers have advanced quickly in the recent past. Research on, e.g., BDDs did not progress comparably.

As a practical example, we take a small rule set from a German premium car manufacturer - for large rule sets a compilation into BDDs or DNNFs is infeasible. For this small rule set the compilation into a BDD can take minutes, whereas loading the formulas onto a solver takes only a few hundred milliseconds. Solving such formulas with a modern solver usually takes only few milliseconds - approximately the same time as for a compiled BDD. I.e., in the same time in which the BDD is generated, the solver could  have solved hundreds of thousands of queries already. Plus, even when looking at the compiled format, the BDD does not yield any performance advantages. Even worse: in an interactive environment where someone creates or changes rules in the rule set and wants to perform on-the-fly validations and computations, the BDD had to be recompiled after each change.

There are, of course, also useful scenarios for knowledge compilation: usually, when the rule set is small, when not the whole rule set has to be considered, or for visualizing formulas.

Therefore, we want to pursue and support both approaches, the solver-approach and knowledge compilation. To this respect, LogicNG on one hand hand provides three different kinds of solver engines:

- SAT solvers for solving Boolean and pseudo-Boolean formulas.
- Cardinality solvers specialized for formulas with many cardinality constraints.
- MaxSAT solvers including partial weighted MaxSAT solvers for optimizations on Boolean and pseudo-Boolean formulas.

On the other hand, the library also features two knowledge compilers:

- A BDD implementation with interactive reordering and different variable ordering strategies.
- A d-DNNF compiler based on DTrees.

