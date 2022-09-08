---
title: Pseudo-Boolean Constraints
---

A pseudo-Boolean constraint [PBConstraint](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/formulas/PBConstraint.java) is a generalization of a [Cardinality Constraint](../cardinality-constraints).  In a cardinality constraint, every variable has the same weight (i.e. 1); In a pseudo-Boolean constraint, each variable can have a different weight.  Further, pseudo-Boolean constraints admit negated variables (literals), whereas cardinality constraints only admit variables.

Pseudo-Boolean constraints have the form

$$
c_1 \cdot lit_1 + c_2 \cdot lit_2 + ... + c_n \cdot lit_n \quad \mathtt{?} \quad k
$$

where:

- $lit_i$ are Boolean literals, evaluating to `true` and `false`, for $1 \leq i \leq n$
- $c_i$ are integer coefficients, the "weights" of the literals, for $1 \leq i \leq n$
- $\mathtt{?}$ is a comparison operator `<=`, `<`, `>`, `>=` or `=`

A solution of a PB-constraint is an assignment of variables which satisfies the constraint (c.f. the next section).

Some examples for pseudo-Boolean constraints are:

- Clauses: `A | ~B | C` is equivalent to `A + ~B + C >= 1`,
- Cardinality constraints: `A + B + C >= 3`. Cardinality constraints are a special case of PB-constraints, where every coefficient is 1.
- General constraints: `A + 2*~B - 3*C = 2`


### Evaluating Pseudo-Boolean Constraints

If a literal in the pseudo-Boolean constraint evaluates to `true` for a given assignment, it is treated as `1`; if it evaluates to `false`, it is treated as `0`.
When evaluating the pseudo-Boolean constraint, the left-hand side is evaluated first with standard rules of linear arithmetic and then compared to the right-hand side.

As an example, consider the pseudo-Boolean constraint:

```
A + 2*~B + -3*C = 2
```

Consider the following assignments:

1. `{A = true, B = true, C = true}`. The left-hand side evaluates to `1 + 2*0 - 3*1 = -2`, thus this *is not* a model for the formula.
2. `{A = true, B = false, C = false}`. The left-hand side evaluates to `1 + 2*1 - 3*0 = 3`, thus this *is not* a model for the formula.
3. `{A = false, B = false, C = false}`. The left-hand side evaluates to `0 + 2*1 - 3*0 = 2`, thus this *is* a model for the formula.

!!! danger "Negation vs. Minus"
    Note that the negation of a literal is different to the minus in front of a coefficient, therefore e.g. `-2 * B` is not equal to `2 * ~B`: For the model `{B = true}`, `-2 * B = -2`, but `2 * ~B = 0`.


## Encoding Pseudo-Boolean Constraints

Some applications, such as [SAT solvers](../../solvers/sat-solving), cannot deal with pseudo-Boolean constraints in their "natural form" since a pseudo-Boolean constraint is not a purely Boolean construct.  Thus, the constraints have to be *encoded* to CNF before added to the solver. An encoded pseudo-Boolean constraint does not contain any signs like `<=`, `>=`, `<`, `>` and `=` or any coefficients anymore but is a proper Boolean formula in CNF.  There exist many algorithms for encoding pseudo-Boolean constraints, depending on their type.

LogicNG implements some of these algorithms as described below.  If you want to change the encoding of a pseudo-Boolean constraint using any of these algorithms, you can do this via the pseudo-Boolean constraint configuration [PBConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/pseudobooleans/PBConfig.java), see below.  If you don't specify an encoding, then the default encoder is used.  The default encoder for a certain pseudo-Boolean constraint was chosen based on theoretical and practical observations.

You can find a general overview over those algorithms [here](https://arxiv.org/pdf/2005.02073.pdf).

- [PBAdderNetworks](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/pseudobooleans/PBAdderNetworks.java):  The adder networks encoding for pseudo-Boolean constraints to CNF.
- [PBSWC](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/pseudobooleans/PBSWC.java): A sequential weight counter for the encoding of pseudo-Boolean constraints in CNF. You can find information about this [here](https://iccl.inf.tu-dresden.de/w/images/c/c3/Steinke:11:KI.pdf).
- [PBBinaryMerge](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/pseudobooleans/PBBinaryMerge.java): This algorithm encodes for pseudo-Boolean constraints to CNF due to [Manthey, Philipp, and Steinke](https://iccl.inf.tu-dresden.de/web/WVPub153/en). There are three parameters:
  - `binaryMergeUseGAC`: Sets whether general arc consistency should be used in the binary merge encoding. The default value is *true*.
  - `binaryMergeNoSupportForSingleBit`: Sets the support for single bits in the binary merge encoding. The default value is *false*
  - `binaryMergeUseWatchDog`: Sets whether the watchdog encoding should be used in the binary merge encoding. The default value is *true*.

The default encoding is the sequential weight counter `PBSWC`.


## Using the Encoder

There are two ways to encode a pseudo-Boolean constraint:

1. You can encode a constraint manually using the [PBEncoder](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/pseudobooleans/PBEncoder.java) and get a list of clauses of the CNF.
2. You can call the `cnf()` method on a pseudo-Boolean constraint and get the resulting CNF directly.

Consider the pseudo-Boolean constraint

```
A + 2*~B - 3*C = 2
```

which can be created in the following way:

``` java
List<Literal> lits =
        Arrays.asList(f.variable("A"), f.literal("B", false), f.variable("C"));
List<Integer> coeffs = Arrays.asList(1, 2, -3);
Formula f1 = f.pbc(CType.EQ, 2, lits, coeffs);
```

---

You can configure the `PBEncoder` with a [PBConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/pseudobooleans/PBConfig.java) object. In this case we explicitly choose the `PBBinaryMerge` encoding with the parameter `binaryMergeUseWatchDog` set to false, to generate the CNF for this pseudo-Boolean constraint. When using the `PBEncoder` the result is a list of formulas, each representing a single clause of the encoding's CNF.

``` java
PBConfig config = PBConfig.builder()
        .pbEncoding(PBConfig.PB_ENCODER.BINARY_MERGE)
        .binaryMergeUseGAC(false)
        .build();
List<Formula> encoding = new PBEncoder(f, config).encode((PBConstraint) f1);
```

The result is then the list of clauses (again with replaces auxiliary variables for readability):

``` java
[
 (x1), (~A | C | x5), (C | x4) (~A | x4), (B | C | x9), (C | x8), (B | x8),
 (~x8 | ~x5 | x11), (~x9 | ~x5 | x12), (~x5 | x10), (~x8 | x10), (~x9 | x11),
 ~x12, ~B, ~C
]
```

When calling the `cnf()` method on the constraint, the default encoding configured in the formula factory is used. So when you change this default for pseudo-Boolean constraints to `PBBinaryMerge` with parameter `binaryMergeUseWatchDog` set to false beforehand, the result of calling `cnf()` yield the same result:

``` java
PBConfig config = PBConfig.builder()
        .pbEncoding(PBConfig.PB_ENCODER.BINARY_MERGE)
        .binaryMergeUseGAC(false).build();
f.putConfiguration(config);
Formula encoding = f1.cnf();
```

The result is the CNF equivalent to the clauses list above:

```java
(x1) & (~A | C | x5) & (C | x4) & (~A | x4) & (B | C | x9) & (C | x8) &
(B | x8) & (~x8 | ~x5 | x11) & (~x9 | ~x5 | x12) & (~x5 | x10) &
(~x8 | x10) & (~x9 | x11) & ~x12 & ~B & ~C
```


