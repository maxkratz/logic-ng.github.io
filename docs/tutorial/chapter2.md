---
title: Tutorial Chapter 2
---

In the example in the previous chapter we introduced a lot of variables and formulas. In this chapter, we will have a closer look at what one can do with formulas.

## Substituting Formulas:

Beatrice sometimes mixes up how big the different wheels are. Therefore, she decides to *substitute* the variables for the front and back wheel for variables whose name includes their size.

She replaces:

- `wf1` → `wf24`
- `wf2` → `wf26`
- `wf3` → `wf27` for the front wheel of size 27.5 inch
- `wf4` → `wf29`
- `wf5` → `wf32`

and

- `wb1` → `wb24`
- `wb2` → `wb26`
- `wb3` → `wb27` for the back wheel of size 27.5 inch
- `wb4` → `wb29`
- `wb5` → `wb32`

Beatrice substitutes the variables in the following way (c.f. the function `substituteLiterals` in the Tutorial):

``` java
final Substitution substitution = new Substitution();
substitution.addMapping(f.variable("wf1"), f.variable("wf24"));
substitution.addMapping(f.variable("wf2"), f.variable("wf26"));
substitution.addMapping(f.variable("wf3"), f.variable("wf27"));
substitution.addMapping(f.variable("wf4"), f.variable("wf29"));
substitution.addMapping(f.variable("wf5"), f.variable("wf32"));

substitution.addMapping(f.variable("wb1"), f.variable("wb24"));
substitution.addMapping(f.variable("wb2"), f.variable("wb26"));
substitution.addMapping(f.variable("wb3"), f.variable("wb27"));
substitution.addMapping(f.variable("wb4"), f.variable("wb29"));
substitution.addMapping(f.variable("wb5"), f.variable("wb32"));

final List<Formula> substituted = data.formulas.stream()
        .map(formula -> formula.substitute(substitution))
        .collect(Collectors.toList());
```

The result is the rule set in terms of the new variables:

```
frame <=> f1 | f2 | f3, f1 + f2 + f3 = 1
handlebar <=> h1 | h2 | h3 | h4 | h5
h1 + h2 + h3 + h4 + h5 = 1
saddle <=> s1 | s2 | s3 | s4
s1 + s2 + s3 + s4 = 1
frontWheel <=> wf24 | wf26 | wf27 | wf29 | wf32
wf24 + wf26 + wf27 + wf29 + wf32 = 1
backWheel <=> wb24 | wb26 | wb27 | wb29 | wb32
wb24 + wb26 + wb27 + wb29 + wb32 = 1
bell <=> b1 | b2 | b3
b1 + b2 + b3 <= 1
luggageRack <=> r1 | r2 | r3 | r4
r1 + r2 + r3 + r4 <= 1
color <=> c1 | c2 | c3 | c4
c1 + c2 + c3 + c4 = 1
f1 => ~s1 & ~h3
f1 => ~luggageRack | r4
f3 => ~h5
h5 => ~bell
r4 => ~b2 | f2 | f3
wf24 <=> wb24
wf26 <=> wb26
wf27 <=> wb27
wf29 <=> wb29
wf32 <=> wb32
frame => wf26 | wf27 | wf29 | wf32
wf32 => f3
```


## Restricting Formulas:

Due to a global pandemic, there are issues with the supply chain.  Beatrice's supplier of trust *Super Supplier* cannot deliver a couple of components:  Back wheel 24-inch `wb24` and color `c3` are out of stock. However, *Super Supplier* delivered a lot of the minimalistic luggage racks `r4`.

How can she still configure valid bikes with the given restrictions? She decides that all bikes from now on

- may not have the back wheel `wb24` or color `c3`
- must have the minimalistic luggage rack `r4`, since her little storage space will otherwise be cram-full within a couple of days.

Beatrice *restricts* the set of rules in the following way:

- `wb24` and `c3` are not allowed (set to `false`)
- `r4` must be contained (set to `true`).

See the function `restrictRuleset` in the tutorial code, how to do this:

```java
final Assignment assignment = new Assignment();

assignment.addLiteral(data.f.variable("wb24").negate());
assignment.addLiteral(data.c3.negate()); // (1)
assignment.addLiteral(data.r4);

final List<Formula> restricted = formulas.stream()
        .map(formula -> formula.restrict(assignment))
        .collect(Collectors.toList());
```

1. cheap color sold out

The result is the following rule set:

```
frame <=> f1 | f2 | f3
f1 + f2 + f3 = 1
handlebar <=> h1 | h2 | h3 | h4 | h5
h1 + h2 + h3 + h4 + h5 = 1
saddle <=> s1 | s2 | s3 | s4
s1 + s2 + s3 + s4 = 1
frontWheel <=> wf24 | wf26 | wf27 | wf29 | wf32
wf24 + wf26 + wf27 + wf29 + wf32 = 1
backWheel <=> wb26 | wb27 | wb29 | wb32
wb26 + wb27 + wb29 + wb32 = 1
bell <=> b1 | b2 | b3
b1 + b2 + b3 <= 1
luggageRack
r1 + r2 + r3 <= 0
color <=> c1 | c2 | c4
c1 + c2 + c4 = 1
f1 => ~s1 & ~h3
$true
f3 => ~h5
h5 => ~bell
~b2 | f2 | f3
~wf24
wf26 <=> wb26
wf27 <=> wb27
wf29 <=> wb29
wf32 <=> wb32
frame => wf26 | wf27 | wf29 | wf32
wf32 => f3
```

The following rules have changed:

| Rule                                                                | Restricted Rule                                     | Explanation                                                           |
|---------------------------------------------------------------------|-----------------------------------------------------|-----------------------------------------------------------------------|
| <code>luggageRack &lt;=&gt; r1 &#124; r2 &#124; r3 &#124; r4</code> | `luggageRack`                                       | since  `r4` is chosen, a luggage rack is taken                        |
| `r1 + r2 + r3 + r4 <= 1`                                            | `r1 + r2 + r3 <= 0`                                 | since `r4` is chosen, none of `r1`, `r2` and `r3` may be chosen too   |
| <code>color &lt;=&gt; c1 &#124; c2 &#124; c3 &#124; c4</code>       | <code>color &lt;=&gt; c1 &#124; c2 &#124; c4</code> | `~c3` is a restriction                                                |
| `c1 + c2 + c3 + c4 = 1`                                             | `c1 + c2 + c4 = 1`                                  | `~c3` is a restriction                                                |
| <code>f1 =&gt; ~luggageRack &#124; r4</code>                        | `$true`                                             | the right hand side is always `true`, since `r4` is chosen            |
| <code>r4 =&gt; ~b2 &#124; f2 &#124; f3</code>                       | <code>~b2 &#124; f2 &#124; f3</code>                | the left hand side is satisfied, thus the right hand part has to hold |
| `wf24 <=> wb24`                                                     | `~wf24`                                             | `~wb24` is a restriction                                              |

Two remarks:
1. When a formula restricts to `$true` this means it is always fullfilled and therefore can be ommitted.  If a formula would restrict to `$false` the whole rule set would be contradictionary.
2. The restricted second rule `r1 + r2 + r3 <= 0` is equivalent to `~r1 & ~r2 & ~r3`.

Beatrice translates the changes of the rule set for the next customer.  The updated rules are the following:

- you must choose the luggage rack `r4` and may not choose any of `r1`, `r2`, `r3`
- you must take exactly one of the colors `c1`, `c2`, `c4`
- either you do not take the bike bell `b2` or you take frame 2 `f2` or you take `f3`
- you may not take front wheel with 24-inch `wf24`

More information about substituting and restricting formulas can be found in the [relevant section](../../documentation/formulas#restricting-formulas) in the documentation.
