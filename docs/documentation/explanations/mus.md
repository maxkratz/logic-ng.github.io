---
title: MUS
---

A *MUS* is a minimal unsatisfiable subset for a given set of formulas.  It contains only those formulas which lead to the given set of formulas being unsatisfiable.  In other words: If you remove at least one of the formulas in the MUS from the given set of formulas, your set of formulas is satisfiable.  This means a MUS is locally minimal.  Thus, given a set of formulas which is unsatisfiable, you can compute its MUS and have one locally minimal conflict description *why* it is unsatisfiable.

## MUS Algorithms

In LogicNG, two algorithms are implemented to compute the MUS: A *deletion*-based algorithm [DeletionBasedMUS](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/explanations/mus/DeletionBasedMUS.java) and an *insertion*-based algorithm [PlainInsertionBasedMUS](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/explanations/mus/PlainInsertionBasedMUS.java).

The main idea of the deletion-based algorithm is to start with all given formulas and iteratively test each formula for relevance. A formula is relevant for the conflict, if its removal yields in a satisfiable set of formulas. Only the relevant formulas are kept. The main idea of the insertion-based algorithm is to start with an empty set and incrementally add propositions to the MUS which have been identified to be relevant.

The default MUS generation [MUSGeneration](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/explanations/mus/MUSGeneration.java) uses the deletion-based algorithm.


## Computing the MUS

Consider the following propositions:

``` java
List<Proposition> props = new ArrayList<>();
props.add(new StandardProposition(f.parse("A & B")));
props.add(new StandardProposition(f.parse("B => ~C")));
props.add(new StandardProposition(f.parse("A | C & D")));
props.add(new StandardProposition(f.parse("D & (E | F)")));
props.add(new StandardProposition(f.parse("C")));
```

You can compute the MUS on a set of propositions in the following way:

``` java
UNSATCore<Proposition> core = new MUSGeneration().computeMUS(props, f);
```

The result is:

```
UNSATCore{
  isMUS=true,
  propositions=[
    StandardProposition{formula=B => ~C, description=},
    StandardProposition{formula=C, description=},
    StandardProposition{formula=A & B, description=}
  ]
}
```

indicating that the conflict arises from the following subset of formulas:

```
{A & B, B => ~C, C}
```

As you can see, no proposition in this set can be removed without turning the set satisfiable, so in fact it is locally minimal.

Note that it only makes sense to compute a MUS for an unsatisfiable formula set. If you call the MUS for a satisfiable formula set, an exception will be thrown.


## MUS Configuration

You can configure the MUS generation using the [MUSConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/explanations/mus/MUSConfig.java) in order to

1. specify the algorithm for the computation, and
2. control the computation using a [SATHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/SATHandler.java) which is passed to the SAT solver computing the MUS under the hood.

For example, the following code snippet defines a MUS configuration with the plain insertion algorithm (see above) and a [TimeoutSATHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/TimeoutSATHandler.java) which stops the computation after 100 ms.

``` java
MUSConfig config = MUSConfig.builder()
        .algorithm(MUSConfig.Algorithm.PLAIN_INSERTION)
        .handler(new TimeoutSATHandler(100))
        .build();
UNSATCore<Proposition> core = musGeneration.computeMUS(props, f, config);
```

In this case, the result is the same for both algorithms. In general, this is not always the case.


!!! attention "MUSes Are Not Unique"

    A set of formulas can have *multiple* MUSes. Consider the following example:

    ``` java
    List<Proposition> props = new ArrayList<>();
    props.add(new StandardProposition(f.parse("~A")));
    props.add(new StandardProposition(f.parse("A | ~B")));
    props.add(new StandardProposition(f.parse("B")));
    props.add(new StandardProposition(f.parse("~B | C")));
    props.add(new StandardProposition(f.parse("~C | D")));
    props.add(new StandardProposition(f.parse("~D")));
    props.add(new StandardProposition(f.parse("~C | E")));
    props.add(new StandardProposition(f.parse("~E")));
    ```

    The two different algorithms return different MUSes. The default deletion-based algorithm returns the MUS:

    ```
    {B, A | ~B, ~A}
    ```

    and the plain insertion algorithm returns the MUS:

    ```
    {B, ~D, ~B | C, ~C | D}
    ```
