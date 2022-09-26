---
title: Decision Problems vs. Optimization Problems
hide:
  - toc
---
Another interesting question is whether the query at hand is a *decision problem* or an *optimization problem*. A decision problem can be answered with a simple yes/no answer, e.g., if a certain feature is buildable for a given configuration. An optimization problem on the other hand involves attaching weight factors to certain features, e.g., prices or mass in kg, and then searching for an optimal solution by minimizing or maximizing the cumulative weight factors of the configuration. The result would be for example the cheapest product configuration or the heaviest. Whereas decision problems can be solved with both solver engines and knowledge compilation formats, optimization problems usually are solved with optimizing solvers like a MaxSAT solver.

LogicNG supports solving with hard and soft rules: a part of the rules can be defined as hard which *must* be fulfilled by every configuration, whereas soft rules *should* be fulfilled. The solver then searches for a solution which satisfies a maximum number of the soft rules. Weighted problems are also possible, where each formula can have a weight and then a solution with a maximum weight is searched for by the solver. These are very powerful tools to model and solve many kinds of real-world optimization problems.

