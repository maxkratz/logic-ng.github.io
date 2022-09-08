---
title: Smallest MUS
---

We established in the last section on [MUS](../mus), that a set of formulas can have more than one minimal unsatisfiable set (MUS).  One often computes a MUS in order to find out why a set of formulas is unsatisfiable.  But when there are multiple MUS, which is best to consider?
Intuitively, one is interested in a particularly small MUS, as this means it describes the conflict very compactly.

A *smallest* minimal unsatisfiable set (SMUS) is a smallest MUS based on the number of formulas it contains.  So in contrast to a regular MUS it is not only locally minimal, but globally minimal.

The SMUS implementation in LogicNG is based on the paper [Smallest MUS Extraction with Minimal Hitting Set Dualization](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.702.5211&rep=rep1&type=pdf) by Ignatiev, Previti, Liffiton, and Marques-Silva from 2015.


## Computing the SMUS

The class [SmusComputation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/explanations/smus/SmusComputation.java) holds static methods for computing the SMUS: `computeSmus()` computes the SMUS over a set of propositions, `computeSmusForFormulas()` is a helper method and computes the SMUS over a set of formulas.

The second parameter in both methods holds an optional list of additional constraints. If this list is non-empty, the SMUS is computed under these additional conditions.  The third parameter is a [formula factory](../../formula-factory). The fourth parameter, which is optional (as both methods are overloaded), is an [OptimizationHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/OptimizationHandler.java) to control the computation execution and abort it if necessary.

Let's consider the following list of propositions:

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

We show three examples on how to compute the SMUS for these propositions.

### Compute Without Additional Constraints

When computing the SMUS without additional constraints:

``` java
List<Proposition> result = SmusComputation.computeSmus(props, null, f);
```

the result is

```
{~A, A | ~B, B}
```

Thus, the smallest existing MUS consists of only three propositions. In this case this is the only smallest MUS. In general, however, there could be another MUS with the same smallest number of propositions.

### Compute With Additional Constraints

Computing a SMUS using additional constraints means that a SMUS is computed from the given propositions under the assumption that the additional constraints hold. Note, the additional constraints are not part of the SMUS.

For example, the additional constraint `D` leads to a smaller MUS than the previously found one:

``` java
List<Proposition> result = SmusComputation.computeSmus(props,
        Collections.singletonList(f.parse("D")), f);
```

The result contains only two propositions:

```
{~D | E, ~E}
```

The result can be read as: Under the constraint that `D` holds, the smallest MUS consists of the propositions `~D | E` and `~E`.

!!! tip "Application Insight"

    A use case for additional constraints can be constraints which hold universally and do not need to occur in the explanation in a certain context. For example, a set of wheels `{r1, r2, r3}` which mutually exclude each other. If our propositions are `r1`, `r2` and `r1 + r2 + r3 <= 1`, then we get these three propositions back as SMUS. However, if instead the constraint `r1 + r2 + r3 <= 1` is given as an additional constraint, the resulting SMUS is `{r1, r2}`: Assuming the exclusion of different wheels is a known constraint, the SMUS `{r1, r2}` is sufficient to explain the conflict.


### Compute With a Handler

Suppose we perform the same operation as above, but we want to control the computation. The `TimeoutOptimizationHandler` below stops the computation after 100 ms.

``` java
List<Proposition> result = SmusComputation.computeSmus(
        props,
        Collections.singletonList(f.parse("D")),
        f,
        new TimeoutOptimizationHandler(100));
```

Using a handler for the SMUS computation is often advisable, since the involved computations are very hard and can take a long time for complex cases.  Therefore, executing the SMUS computation without handler may result in very long computation times, if possible at all.

