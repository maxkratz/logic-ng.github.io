---
title: Pseudo-Boolean Constraints
---

A formula predicate takes a formula as input and computes a truth value on that formula, e.g. whether a formula is in a certain normal form like NNF, CNF, or DNF or if it is satisfiable.

It can be evaluated whether a formula predicate holds (evaulates to true) for a formula `f1` with the `.holds()` method:

``` java
boolean isCNF = f1.holds(CNFPredicate.get());
```

In this case the result of the predicate is cached in the formula.  If you do not want to cache the result, you can manually deactivate caching:


``` java
boolean isCNF = f1.holds(CNFPredicate.get(), false);
```

Most predicates implemented in LogicNG fall into one of two categories: (1) predicates which check if a given formula has a certain form, e.g. NNF, CNF, ..., and (2) predicates which check certain properties of a formula, e.g. if it is satisfiable.  One can also think of these to types of (1) checking syntactical properties, and (2) checking semantical properties of a formula.


## Syntactical Predicates

The **first type** is those predicates, which check whether the formula is of a certain *syntactical form*.  There are three predicates in LogicNG for common normal forms:

- negation normal form (NNF) - only conjunctions and disjunctions, and negations only occur before variables: [NNFPredicate](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/predicates/NNFPredicate.java)
- conjunctive normal form (CNF) - a conjunction of disjunctions of literals: [CNFPredicate](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/predicates/CNFPredicate.java)
- disjunctive normal form (DNF) - a disjunction of conjunctions of literals: [DNFPredicate](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/predicates/DNFPredicate.java)

Each CNF and DNF is also in NNF.  CNF play a very important role in LogicNG, since they are used as input format for SAT solvers.

Further, one can check whether the given formula is an "And-inverter graph" (AIG) with the [AIGPredicate](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/predicates/AIGPredicate.java).

!!! info "Excursion: And-Inverter Graphs"
    An And-inverter graph is an efficient representation of a formula. AIGs have recently gained increasing interest because of its coupling opportunities with efficient SAT solvers.  The conversion from Boolean formulas to an AIG is fast and scalable. It only requires that every gate can be expressed in terms of ANDs and NOTs. This makes them, in comparison with BDD and DNF, an efficient representation.  More information about And-inverter graphs (AIG) can be found [here](https://en.wikipedia.org/wiki/And-inverter_graph).

An example for checking these properties on a formula can be seen in the following code snippet:

``` java
Formula f1 = f.parse("(A | B) & (A | C ) & (C | D) & (B | ~D)");

boolean isAIG = f1.holds(AIGPredicate.get()); // false
boolean isNNF = f1.holds(NNFPredicate.get()); // true
boolean isCNF = f1.holds(CNFPredicate.get()); // true
boolean isDNF = f1.holds(DNFPredicate.get()); // false
```

[:octicons-tag-24: 2.3.0](https://github.com/logic-ng/LogicNG/releases/tag/v2.3.0) For NNF/CNF/DNF there are three helper methods directly on the formula which execute the predicates as in the example above: `isNnf()`, `isCnf()`, `isDnf()`.

## Semantic Predicates

The **second type** of predicates checks whether formulas satisfy certain semantical properties related to their satisfiability:

``` mermaid
graph TD
  A[Satisfiable?] -->|Yes| B([SATPredicate holds]);
  A -->|No| C([ContradictionPredicate holds]);
  B --> D[Tautology?];
  D -->|Yes| E([TautologyPredicate holds]);
  D -->|No| F([ContingencyPredicate holds]);
```

Let's have a closer look to each of these predicates:


### SAT Predicate

The [SATPredicate](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/predicates/satisfiability/SATPredicate.java) tests whether a formula is satisfiable. A formula is satisfiable if there exists at least one assignment such that the formula evaluates to `true` with this assignment. Such an assignment is called *satisfying assignment* or *model*. For example `A & B | C` is satisfiable for the assignment `{A, B, ~C}`. In order to check for satisfiability, the `SATPredicate` internally calls a [SAT Solver](../../../solvers/sat-solving).

``` java
Formula f1 = f.parse("A & B | C");

boolean isSatisfiable = f1.holds(new SATPredicate(f)); // true
```


### Contradiction Predicate

In contrast to the `SATPredicate` the [ContradictionPredicate](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/predicates/satisfiability/ContradictionPredicate.java) can be used to test if a formula is *unsatisfiable*, i.e. the formula is a *contradiction*. That is, this predicate holds if and only if the `SATPredicate` does not hold and vice versa. An example for a contradiction is `A & (A => B) & (B => ~A)`.

``` java
Formula f1 = f.parse("A & (A => B) & (B => ~A)");

boolean isSatisfiable = f1.holds(new SATPredicate(f)); // false
boolean isContradiction = f1.holds(new ContradictionPredicate(f)); // true
```


### Tautology Predicate

The [TautologyPredicate](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/predicates/satisfiability/TautologyPredicate.java) indicates whether a given formula is a tautology, that is, always holds, regardless of the assignment.  An example for an tautology is `(A & B) | (~A & B) | (A & ~B) | (~A & ~B)`.

``` java
Formula f1 = f.parse("(A & B) | (~A & B) | (A & ~B) | (~A & ~B)");

boolean isSatisfiable = f1.holds(new SATPredicate(f)); // true
boolean isContradiction = f1.holds(new ContradictionPredicate(f)); // false
boolean isTautology = f1.holds(new TautologyPredicate(f)); // true
```

A very useful usage of the tautology predicate is to check whether two formulas are semantically equivalent.  To do this, create an equivalence consisting of the two formulas to check.  Then check whether this equivalence is a tautology:

``` java
Formula f1 = f.parse("(A | B) & (A | C ) & (C | D) & (B | ~D)");
Formula f2 = f.parse("D & A & B | ~D & C & A | C & B");

Formula equivalence = f.equivalence(f1, f2);
boolean formulasAreEquivalent =
    equivalence.holds(new TautologyPredicate(f)); // true
```

Also, testing if one formula is a logical implication of another formula can be tested the same way by creating an implication `#!java f.implication(f1, f2)` instead.

### Contingency Predicate

If a formula is satisfiable, but not a tautology, then the [ContingencyPredicate](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/predicates/satisfiability/ContingencyPredicate.java) holds. In other words, the formula is *satisfiable* and *falsifiable*. For example, for the formula `A & B | C` the contingency predicate holds: The formula is satisfiable (e.g. a model is `{A, B, C}`) and the formula is falsifiable (e.g. a falsifying assignment is `{~A, ~B, ~C}`).

``` java
Formula f1 = f.parse("A & B | C");

boolean isSatisfiable = f1.holds(new SATPredicate(f)); // true
boolean isContradiction = f1.holds(new ContradictionPredicate(f)); // false
boolean isTautology = f1.holds(new TautologyPredicate(f)); // false
boolean isContingency = f1.holds(new ContingencyPredicate(f)); // true
```


## Fast Evaluation of Partial Assignments

A special predicate is the [EvaluatesToConstantPredicate](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/predicates/EvaluatesToConstantPredicate.java).  It was created for a very special use case in an application developed by us.  Usually the standard [formula evaluation](../..#evaluating-formulas) is very fast - but the evaluation requires either a complete assignment of all the formula's variables or it automatically assigns all variables not in the assignment to false.

So if one wants to know, if a formula evaluates to true or false under a partial assignment, one has to use the [formula restriction](../..#restricting-formulas) and check whether the resulting formula is the constant true or false.  But if all one is interested in is whether the formula restricts to true or false (and does not care for the restricted formula otherwise) this approach is not as fast as it could be because it generates restricted formulas on its way.

!!! tip "Application Insight"
    In one of our applications we had to check hundreds of thousands of formulas against the same assignments over and over again.  Therefore we wrote this special predicate, which only checks if a formula evaluates to true or false under a potentially partial assignment.  Usually you will not need such a specialized function, but in case you do - there it is :smiley:

```java
Formula f1 = f.parse("A & B | C & ~D");
Formula f2 = f.parse("A & ~B | C & ~D");

Map<Variable, Boolean> assignment = new HashMap<>();
assignment.put(f.variable("A"), true);
assignment.put(f.variable("B"), false);

EvaluatesToConstantPredicate evaluatesToConstantPredicate =
        new EvaluatesToConstantPredicate(true, assignment);

boolean holdsForF1 = f1.holds(evaluatesToConstantPredicate); // false
boolean holdsForF2 = f2.holds(evaluatesToConstantPredicate); // true
```

You can see that this predicate is generated for one specific assignment given as a mapping from variable to truth value.  The first parameter in the predicate indicates for which evaluation result you want to check.  In this we want to now if a formula evaluates to true under the given assignment.
