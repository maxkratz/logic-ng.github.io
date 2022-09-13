---
title: Tutorial Chapter 1
---

## Configuring a Bike

The brand-new shop *Build 'n Bike* pursues a new business idea: It is the first shop in *Boolea* which offers customers to build their bike completely from scratch.

The customers can configure their bike with the following components:

| Component    | Feature Class | Features                                                                                     |
|--------------|---------------|----------------------------------------------------------------------------------------------|
| Frame        | `frame`       | `f1` (carbon), `f2` (aluminium), `f3` (steel)                                                |
| Handlebar    | `handlebar`   | `h1` (cruiser bar),  `h2`  (drop bar), `h3` (touring  bar), `h4` (flat bar), `h5` (aero bar) |
| Saddle       | `saddle`      | `s1` (touring), `s2` (comfort), `s3` (triathlon), `s4` (pro)                                 |
| Front wheel  | `frontWheel`  | `wf1` (24 inch), `wf2` (26 inch), `wf3` (27.5 inch), `wf4` (29 inch), `wf5` (32 inch)        |
| Back wheel   | `backWheel`   | `wb1` (24 inch), `wb2` (26 inch), `wb3` (27.5 inch), `wb4` (29 inch), `wb5` (32 inch)        |
| Bike bell    | `bell`        | `b1` (classic), `b2` (metal strip), `b3` (ladybug)                                           |
| Luggage rack | `luggageRack` | `r1` (aluminium), `r2` (titanium), `r3` (steel), `r4` (minimalistic)                         |
| Color        | `color`       | `c1` (blue), `c2` (red), `c3` (white), `c4` (silver)                                         |

Beatrice, the founder of *Build 'n Bike*, did a course in propositional logic at university. She knows that for the configuration to result in a working bike, there are a couple of rules which have to be fulfilled.  Firstly, every bike needs:

- exactly one frame
- exactly one handlebar
- exactly one saddle
- exactly one front wheel
- exactly one back wheel
- exactly one color
- at most one bike bell
- at most one luggage rack

She writes the rules down concisely:

```
f <=> f1 | f2 | f3
f1 + f2 + f3 = 1

h <=> h1 | h2 | h3 | h4 | h5
h1 + h2 + h3 + h4 + h5= 1

s <=> s1 | s2 | s3 | s4
s1 + s2 + s3 + s4 = 1

wf <=> wf1 | wf2 | wf3 | wf4 | wf5
wf1 + wf2 + wf3 + wf4 + wf5 = 1

wb <=> wb1 | wb2 | wb3 | wb4 | wb5
wb1 + wb2 + wb3 + wb4 + wb5 = 1

b <=> b1 | b2 | b3
b1 + b2 + b3 <= 1

r <=> r1 | r2 | r3 | r4
r1 + r2 + r3 + r4 <= 1

c <=> c1 | c2 | c3 | c4
c1 + c2 + c3 + c4 = 1
```

The first type of formulas, e.g. `f <=> f1 | f2 | f3` is called an equivalence and can be read as an `if and only if` statement: A frame `f` is contained in the bike if and only if one of the frames `f1`, `f2` or `f3` are contained.

The second type of formulas is called *cardinality constraints*. For example, `f1 + f2 + f3 = 1` is read as: Exactly one of the features `f1`, `f2` and `f3` has to be contained in the bike.  For more information check out the [relevant section](../../documentation/formulas/cardinality-constraints) in the documentation.

Further, with an intuitive understanding for bikes, Beatrice knows the following rules:

- if you take the carbon frame, you cannot take the touring saddle and you cannot take the touring bar: `f1 => ~s1 & ~h3`
- if you take the carbon frame, you can either take no luggage rack or the minimalistic luggage rack: `f1 => ~r | r4`
- if you take the steel frame, you may not take the aero bar: `f3 => ~h5`
- if you take the aero bar, you may not take a bike bell: `h5 => ~b`
- if you take the minimalistic luggage rack, you may not take the bike bell with a metal strip, or you have to take the aluminium or the steel frame: `r4 => ~b2 | f2 | f3`
- front and back wheel must fit together:
  - you can take the front wheel of size 24-inch if and only if you take the back wheel of that size: `wf1 <=> wb1`
  - you can take front wheel of size 26-inch if and only if you take the back wheel of that size: `wf2 <=> wb2`
  - you can take front wheel of size 27.5-inch if and only if you take the back wheel of that size: `wf3 <=> wb3`
  - you can take front wheel of size 29-inch if and only if you take the back wheel of that size: `wf4 <=> wb4`
  - you can take front wheel of size 32-inch if and only if you take the back wheel of that size: `wf5 <=> wb5`
- all frames need wheels of size 26 - 32 inch: `f => wf2 | wf3 | wf4 | wf5`
- if you take the oversized wheels with 32-inch, you need a stable frame: You need to take the steel frame `f3`: `wf5 => f3`

Note that the last two rules can be written as concisely as they are because of the restrictions on "front and back wheel must fit together", so it suffices to make the rule for one of the wheels.  Written concisely, the rules are the following:

```
f1 => ~s1 & ~h3
f1 => ~r | r4
f3 => ~h5
h5 -> ~b
r4 => ~b2 | f2 | f3
wf1 <=> wb1
wf2 <=> wb2
wf3 <=> wb3
wf4 <=> wb4
wf5 <=> wb5
f => wf2 | wf3 | wf4 | wf5
wf5 => f3
```


## Translating the Problem in Java Code:

To use LogicNG, you always have to create a formula factory first.  The formula factory is used to create new variables, formulas, and so on, and is a very central concept in LogicNG, so after this tutorial, wie recommend reading its [documentation](../../documentation/formula-factory).

``` java
FormulaFactory f = new FormulaFactory();
```


### Specifying that each configuration contains the right components

The first step for translating the problem into Java code is to specify that each configuration contains exactly one frame, one handlebar, at most one bike bell, etc.

We show here how to specify rules that ensure that the configuration contains exactly one frame and at most one bike bell. The process is analogous for the other components.

We begin by defining the variables. A variable with name `name` in LogicNG can be created using `#!java f.variable("name")`.

``` java
// frames
Variable frame = f.variable("frame"); // (1)!
Variable f1 = f.variable("f1");
Variable f2 = f.variable("f2");
Variable f3 = f.variable("f3");

// bike bells
Variable bell = f.variable("bell"); // (2)!
Variable b1 = f.variable("b1");
Variable b2 = f.variable("b2");
Variable b3 = f.variable("b3");
```

1. variables for the feature class `frame` and frame 1, 2, and 3
2. variables for the feature class `bell` and bell 1, 2, and 3




Next, we define equivalence and cardinality constraints based on these variables, ensuring that *exactly one* frame and *at most one* bike bell is configured.  One way to do this is to parse the relevant formulas:

``` java
Formula f_equiv = f.parse("frame <=> f.or(f1, f2, f3)"); // (1)!
Formula f_exo = f.parse("f1 + f2 + f3 = 1"); // (2)!

Formula b_equiv = f.parse("bell <=> f.or(b1, b2, b3)"); // (3)!
Formula b_exo = f.parse("b1 + b2 + b3 <= 1"); // (4)!
```

1. a frame is configured if and only if one of f1, f2 and f3 is chosen
2. exactly one of f1, f2 and f3 has to be chosen
3. a bike bell is configured if and only if one of b1, b2 and b3 is chosen
4. at most one of b1, b2 and b3 has to be chosen

Another way to do this is to use the built-in functions from the formula factory: Equivalences can be created using `#!java f.equivalence()` and exactly-one constraints can be generated using `#!java f.exo()`. At-most-one constraints can be created using `#!java f.amo()`.

``` java
Formula f_equiv = f.equivalence(frame, f.or(f1, f2, f3)); // (1)!
Formula f_exo = f.exo(f1, f2, f3); // (2)!

Formula b_equiv = f.equivalence(bell, f.or(b1, b2, b3));
Formula b_amo = f.amo(b1, b2, b3); // (3)!
```

1. using the `equivalence` built-in function
2. using the `exo` built-in function
3. using the `amo` built-in function

### Specifying inclusions, exclusions and equivalences

The second step for translating the problem into Java code is to encode the additional formulas. Again, we show how to encode formulas based on two examples, as the others are modelled analogously.

Consider the first and the 6th formula (first examples for implication and equivalence):
- If you take the carbon frame, you cannot take the touring saddle and you cannot take the touring bar: `f1 => ~s1 & ~h3`
- You can take the front wheel of size 24-inch if and only if you take the back wheel of that size: `wf1 <=> wb1`

Again, one can create these formulas either by parsing the string or by using the built-in functions from the formula factory.

Parsing the formulas can be done in the following way:

```java
Formula formula1 = f.parse("f1 => ~s1 & ~h3"); // (1)!
Formula formula6 = f.parse("wf1 <=> wb1");
```

1. generate formulas by parsing a string

The relevant built-in functions are the following:

A literal is a variable and its polarity (positive/negative) and can be created directly using `#!java f.literal("v", true)` for `v` or `#!java f.literal("v", false)` for `~v`.  A variable `v` (or generally any formula) can be negated with `#!java v.negate()`.  Conjunctions (an *And*) can be created using `#!java f.and(s1.negate(), h3.negate())`, meaning the conjunction of not saddle 1 and not handlebar 3. Similarly, disjunctions (an *Or*) can be created using `#!java f.or(s1, s2)`, meaning the disjunction of saddle 1 and saddle 2.  Implications can be created using `#!java f.implication()`. For an exclusion the right-hand side has to be negated.

Therefore, the formulas can also be generated in the following way:

```java
Formula formula1 = f.implication(f1, f.and(s1.negate(), h3.negate())); // (1)!
Formula formula6 = f.equivalence(wf1, wb1);
```

1. generate using the built-in functions

For more information on generating formulas check out the chapter on [formula factories](../../documentation/formula-factory) and for information about properties and useful methods the chapter on [Formulas](../../documentation/formulas).

The total rules are the following, using built-in functions:

``` java
Formula formula1 = f.implication(f1, f.and(s1.negate(), h3.negate()));
Formula formula2 = f.implication(f1, f.or(luggageRack.negate(), r4));
Formula formula3 = f.implication(f3, h5.negate());
Formula formula4 = f.implication(h5, bell.negate());
Formula formula5 = f.implication(r4, f.or(b2.negate(), f2, f3));

Formula formula6 = f.equivalence(wf1, wb1);
Formula formula7 = f.equivalence(wf2, wb2);
Formula formula8 = f.equivalence(wf3, wb3);
Formula formula9 = f.equivalence(wf4, wb4);
Formula formula10 = f.equivalence(wf5, wb5);

Formula formula11 = f.implication(frame, f.or(wf2, wf3, wf4, wf5));
Formula formula12 = f.implication(wf5, f3);
```

The set of formulas which form the base for the rest of the tutorial is the conjunction of _all_ rules; rules for each component and additional rules:

``` java
List<Formula> formulas = Arrays.asList(f_equiv, f_exo, h_equiv, h_exo, s_equiv,
        s_exo, wf_equiv, wf_exo, wb_equiv, wb_exo, b_equiv, b_amo, r_equiv,
        r_amo, c_equiv, c_exo, formula1, formula2, formula3, formula4, formula5,
        formula6, formula7, formula8, formula9, formula10, formula11, formula12);
```

All variables and formulas and the overall set of rules are defined as fields in the class `BicycleShopData`.

For the reminder of the tutorial, we introduce an object of this class:

```java
BicycleShopData data = new BicycleShopData();
```

and will access fields of this class for example with `#!java data.formulas`.

