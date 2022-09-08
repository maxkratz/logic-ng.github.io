---
title: DNNF
---

Another knowledge compilation format is **d-DNNF** - deterministic, decomposable negation normal form. The d-DNNF of a formula has proven to be more succinct than it's BDD.  Further, it helps to alleviate the ubiquitous memory explosion problem of BDDs with large formulas.  Much of the content in this chapter, such as the definition and the example below are taken from chapter 2.3.2 in [New Formal Methods for Automotive Configuration](https://publikationen.uni-tuebingen.de/xmlui/bitstream/handle/10900/57198/dissertation.pdf?sequence=1&isAllowed=y).

We first explain what a DNNF and a d-DNNF is.  In LogicNG, d-DNNFs are referred to simply as DNNFs. Note that DNNFs which are not deterministic are not implemented in LogicNG.

## DNNF

A formula `f1` in NNF is in *decomposable negation normal form* (DNNF) if the decompositional property holds, that is, the operands of a conjunction do not share variables.

As an example, consider the formula:

```
f1 = (A & B) | (A & ((~B | E) & F))
```

The formula is in DNNF, as can be seen by checking the conjunctions:

1. `(A & B)` with `{A} ∩ {B} = ∅`,
2. `(~B | E) & F)` with `{B, E} ∩ {F} = ∅`
3. `A & ((~B | E) & F)` with `{A} ∩ {B, E, F} = ∅`.

Each propositional formula can be transformed into a semantically equivalent DNNF. This is not obvious, but consider the [canonical DNF](../../formulas/operations/transformations/normal-form-transformations#canonical-dnf-enumeration) of a formula where all unsatisfiable minterms are deleted. Such a formula is obviously a DNNF: the decompositional property has to hold for each minterm. Since all minterms are satisfiable, they cannot contain conflicting literals, and therefore do not share any variables. Since each formula can be transformed into such a canonical DNF by enumerating all models and listing them as minterms, each formula can be transformed into a DNNF.


## d-DNNF

A DNNF `f1` is called *deterministic* (d-DNNF) if operands of a disjunction do not share models.

The DNNF of the example above is not deterministic, since e.g. the two disjunction operands `(A & B)` and `(A & ((~B | E) & F))` share the model `{A → true, B → true, E → true, F → true}`.

An example for a d-DNNF is `((~A & B) | (~B & A)) & ((C & D) | (~C & ~D))`.

Obviously decompositionality holds: no conjunction operands share variables. There are two disjunctions in the formula:

1. `(~A & B) | (~B & A)`, where both operands do not share a model, and
2. `(C & D) | (~C & ~D)`, where both operands do not share a model too.

Therefore, the formula is also deterministic.

Again, each formula can be transformed into a d-DNNF. Reconsider the example above, on how any formula can be transformed into DNNF: Since the resulting DNNF is canonical, no two minterms share a model, therefore it is also deterministic.

For example, consider the formula

``` java
Formula f2 = f.parse("(A | B) & (D | E)");
```

One can transform it to a d-DNNF using:

``` java
Dnnf dnnf = new DnnfFactory().compile(f2);
```

The result is `(A | ~A & B) & (D | ~D & E)`.


## Model counting with d-DNNFs

There are several queries which can be solved in polynomial time for d-DNNFs. One of these queries is implemented in LogicNG, namely counting the number of models of a formula.  Note that a non-deterministic DNNF does not support this operation in polynomial time, as has been shown by Pipatsrisawat & Darwiche in 2008[^1].

[^1]:
    Pipatsrisawat, Knot, & Darwiche, Adnan.2008. New compilation languages based on structured decomposability. Pages 517–522 of: Proceedings of the 23rd national conference on artificial intelligence, AAAI’08. AAAI Press

For the DNNF `dnnf` above, the model count can be computed via

``` java
BigInteger modelcount = dnnf.execute(DnnfModelCountFunction.get());
```

For this example, the model count is `9`.

For information about the model count algorithm of a formula in d-DNNF, check out chapter 2.4 in
[New Formal Methods for Automotive Configuration](https://publikationen.uni-tuebingen.de/xmlui/bitstream/handle/10900/57198/dissertation.pdf?sequence=1&isAllowed=y).

!!! tip "Application Insight"

    Instead of compiling the DNNF of a formula manually and counting the models, there is an advanced algorithm implemented in LogicNG which performs some further optimizations and can simply be called by `#!java ModelCounter.count()` for a list of formulas which is interpreted as conjunction.  This algorithm is used in production and counts models of large configuration spaces up to 10^60^ possible solutions.

