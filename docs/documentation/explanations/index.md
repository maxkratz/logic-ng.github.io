---
title: Explanations
---

Sometimes you might want to understand why a [SAT problem](../solvers/sat-solving) is unsatisfiable and therefore does not have a solution.  Given a set of formulas for which the `sat()` call returns `UNSAT`, there is at least one contradiction in this set of formulas.

Finding this contradiction is the goal of the explanation algorithms in LogicNG.  The approach of all of them is to find a subset of formulas which are involved in this conflict.  In LogicNG, three types of those subsets are implemented:

1. Unsatisfiable core ([Unsat Core](unsat-cores))
2. Minimal unsatisfiable subset ([MUS](mus))
3. Smallest minimal unsatisfiable subset ([SMUS](smus))

To explain the three different types of explanation, we consider the following example of a set of unsatisfiable formulas:

```
A & B & C
A => D
D => E
E => ~C
A => ~B
```

Analyzing this conflict by hand we can see that there are two problems in this formula set:

1. `A & B & C` must be true, but `A => ~B` contradicts this
2. `A & B & C` must be true, but the chain `A => D`, `D => E`, `E => ~C` contradicts this

An **unsatisfiable core** is _a_ subset of the original formulas which contains a conflict.  In this case this could be e.g.

```
A & B & C
A => D
A => ~B
```

We see this formula set still contains a conflict (the first from above) but also another formula which is not relevant for the conflict.  The unsat core may be (and usually is) smaller than the original set of formulas, as long as it still contains a conflict.  The advantage of an unsatisfiable core is that it can be efficiently computed on a SAT solver.

A **minimal unsatisfiable subset (MUS)** is an unsat core with a certain condition: No formula can be removed from it without dissolving the conflict.  So in this example, the formula set

```
A & B & C
A => D
D => E
E => ~C
```

would be a MUS.  No formula in this set can be removed without resolving the conflict.  A MUS is locally minimal, meaning you cannot remove formulas from the MUS but there may be smaller MUSes in the original formula set.

That is where the **smallest minimal unsatisfiable subset (SMUS)** comes into play.  This a globally minimal MUS, meaning there may be other MUSes but none with lesser formulas.  In this case the subset

```
A & B & C
A => ~B
```

is the only SMUS.

Depending on your application, you may need one of these variants.  Note that an unsatisfiable core can be produced by the SAT solver directly (if proof tracing is activated) and often it is already a MUS, but that is not guaranteed.  If you need a MUS and already work with a SAT solver, it is a good idea to first extract an unsatisfiable core and then compute the MUS only on the core which usually has far fewer formulas than the original formula set.

A SMUS is very hard to compute (since there can be exponentially many MUSes in a conflict).  Therefore, this really should only be used when required.

!!! tip "Application Insight"

    When working with explanations, we usually propose working with [propositions](../propositions) instead of [formulas](../formulas), as they can store additional information and the explanations are returned in terms of propositions: When working with the SAT solver and using formulas, your explanation will contain clauses of the generated CNFs which often bare not much resemblance with the original input formulas.
