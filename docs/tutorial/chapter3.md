---
title: Tutorial Chapter 3
---

Beatrice wants to analyze her rule set. In particular, she is interested in the variable distribution: How often do given variables each occur in the set of rules?

Luckily for her there is a LogicNG function to count the number of occurrences of each variable in a formula: `VariableProfileFunction` (c.f. the function `getVariableOccurrences` in the tutorial code).
Beatrice uses the `VariableProfileFunction` in the following way:

``` java
Map<Variable, Integer> variables2Occurrences =
        data.f.and(formulas).apply(new VariableProfileFunction());

SortedMap<Integer, List<Variable>> occurrences2Vars = new TreeMap<>();
variables2Occurrences.forEach((var, occ) ->
        occurrences2Vars.computeIfAbsent(occ, y -> new ArrayList<>()).add(var));
```

The `VariableProfileFunction` computes a map from each variable to its occurrence:

```
{r3=1, r2=1, color=1, saddle=1, h1=2, frontWheel=1, h3=3, f1=3, h2=2, h5=4, f3=5,
 h4=2, f2=3, b1=2, luggageRack=1, b3=2, b2=3, wf32=5, wb32=3, s2=2, s1=3, wf29=4,
 s4=2, s3=2, wf27=4, wb29=3, wf26=4, wb27=3, wf24=3, backWheel=1, bell=2, c2=2,
 c1=2, c4=2, handlebar=1, wb26=3, r1=1, frame=2}
```

In order to sort the variables by the number of their occurrences, she creates the map `occurrences2Vars`.  The result is:

```
{
 1=[r3, r2, color, saddle, frontWheel, luggageRack, backWheel, handlebar, r1],
 2=[h1, h2, h4, b1, b3, s2, s4, s3, bell, c2, c1, c4, frame],
 3=[h3, f1, f2, b2, wb32, s1, wb29, wb27, wf24, wb26],
 4=[h5, wf29, wf27, wf26], 5=[f3, wf32]
}
```

For more information about formula functions check out the [relevant chapter](../../documentation/formulas/operations/formula-functions#compute-the-number-of-occurrences-of-variables-and-literals-in-formulas) in the documentation.
