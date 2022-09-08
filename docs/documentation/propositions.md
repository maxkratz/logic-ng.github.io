---
title: Propositions
---

A [Proposition](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/propositions/Proposition.java) is a formula with additional information like a textual description or a user-provided object.

!!! tip "Application Insight"

    Storing this extra information with the formulas can be useful in various application contexts. For example, we use propositions when we know that we will not only be interested in the result of an algorithm, but also the explanation of its result. For more information see the chapter on [explanations](../explanations).  The big advantage here is that the original formula context is maintained.  When adding a formula to the SAT solver, internally the formula is transformed to CNF and single clauses are added to the solver.  When you now extract an explanation for an unsatisfiable formula from the solver, the result will contain these single clauses which often are hard to map to the original input formulas.  By using propositions, the result will be in terms of the original prospotision which makes understanding the explanation much easier in practise.

The abstract class `Proposition` has the single abstract method `formula()` which returns the formula of the proposition. LogicNG provides two implementations of a `Proposition`:

## Standard Proposition

The [StandardProposition](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/propositions/StandardProposition.java) holds a formula and a textual description.
You can configure a proposition with and without a description:

1. With a description:

    ``` java
    Proposition props1 =
            new StandardProposition("my formula", f.parse("A | ~B & C"));
    ```

    generates the proposition
    `StandardProposition{formula=A | ~B & C, description=my formula}`


2. Without a description:

    ```java
    Proposition props2 = new StandardProposition(f.parse("A | ~B & C"));
    ```

    generates the proposition
    `StandardProposition{formula=A | ~B & C, description=}`


## Extended Proposition

The idea from *extended propositions* is to store additional domain-specific information with a formula.
This information is not used for any algorithms in LogicNG - however, it can be useful to "drag it along" during your application.

An [ExtendedProposition](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/propositions/ExtendedProposition.java) is a formula with additional information provided in a user-defined object which implements the empty (marker-) interface [PropositionBackpack](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/propositions/PropositionBackpack.java).

In your implementation of the `PropositionBackpack` you can store all sorts of information which you want to keep to your formula. Some examples are: An ID from the respective rule system the formula is from, the person who is responsible for the formula, the origin or the type of the formula.

You can think of this information as literally the "backpack" of the formula. No algorithm in LogicNG looks "inside" this backpack, but the backpack is always kept. For example, if LogicNG performs algorithms on the formula, the result still holds the backpack, and maybe this helps you to understand the result better.

Let us consider an example of using the extended proposition with an own backpack `MyBackpack`.
Our backpack stores an ID, a person responsible for this formula and the rule type:

``` java
class MyBackpack implements PropositionBackpack {
    private final long id;
    private final String responsiblePerson;
    private final RuleType ruleType;

    MyBackpack(final long id, final String responsiblePerson,
            final RuleType ruleType) {
        this.id = id;
        this.responsiblePerson = responsiblePerson;
        this.ruleType = ruleType;
    }

    @Override
    public boolean equals(final Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        final MyBackpack that = (MyBackpack) o;
        return id == that.id &&
                Objects.equals(responsiblePerson, that.responsiblePerson) &&
                ruleType == that.ruleType;
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, responsiblePerson, ruleType);
    }

    @Override
    public String toString() {
        return "MyBackpack{" +
                "id=" + id +
                ", responsiblePerson='" + responsiblePerson + '\'' +
                ", ruleType=" + ruleType +
                '}';
    }
}

enum RuleType {
    IMPL,
    EQUIV,
    PBC,
    CC
}
```

Let's generate some propositions:

``` java
new ExtendedProposition<>(new MyBackpack(1, "Rouven", RuleType.EQUIV),
        f.equivalence(f.parse("A & B"), f.parse("C")));
new ExtendedProposition<>(new MyBackpack(2, "Verena", RuleType.IMPL),
        f.implication(f.parse("A"), f.parse("C | D")));
new ExtendedProposition<>(new MyBackpack(3, "Martin", RuleType.CC),
        f.amo(f.variable("A"), f.variable("C")));
```

The resulting propositions are:

```java
ExtendedProposition{formula=A & B <=> C, backpack=MyBackpack{id=1, responsiblePerson='Rouven', ruleType=EQUIV}}
ExtendedProposition{formula=A => C | D, backpack=MyBackpack{id=2, responsiblePerson='Verena', ruleType=IMPL}}
ExtendedProposition{formula=A + C <= 1, backpack=MyBackpack{id=3, responsiblePerson='Martin', ruleType=CC}}
```
