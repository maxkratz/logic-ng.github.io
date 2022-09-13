---
title: Normal Form Transformations
---

The two most important normal forms are the *Conjunctive Normal Form* (CNF) and the *Disjunctive Normal Form* (DNF).  In particular, the CNF is of special importance, since it is the input form required for SAT Solving and many other operations or algorithms.  There are many ways to obtain a CNF or DNF from a formula.  We will describe the methods LogicNG offers in this chapter.

Another important normal form is the *Negation Normal Form* (NNF) where only the operators `~`, `&`, and `|` are allowed and negations must only appear before variables.  In LogicNG this means that the formula consists only of literals Ands, and Ors.  The NNF is often used as pre-processing step before transforming the formula into another normal form.  Also, some algorithms require a formula to be in NNF to work.

For theoretical insights on this section, particularly the Tseitin and Plaisted-Greenbaum transformation, check out chapter 2.1. in [this dissertation](https://publikationen.uni-tuebingen.de/xmlui/bitstream/handle/10900/57198/dissertation.pdf?sequence=1&isAllowed=y).


## NNF Transformation

The NNF of a formula can be obtained by executing the [NNFTransformation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/NNFTransformation.java) on it.  There is also a shortcut method directly on the `Formula` class: `.nnf()`.

``` java
Formula f1 = f.parse("A | ~(B & C)");
Formula nnf = f1.nnf(); // yields A | ~B | ~C
```


## CNF Transformations

As mentioned in the introduction, the CNF is of special interest since SAT solvers require this normal form as input (c.f. [SAT Solving](../../../../solvers/sat-solving)).  A CNF is a conjunction of disjunctions of literals.  The disjunction of literals is called a "clause".  So the formula

```
(a | b | ~c) & (~a | c) & (d | f) & (e | ~c)
```

is a CNF with four clauses, the first contains three literals, the other two literals.

The naive approach for transforming a formula into CNF is to use equivalence transformations such as DeMorgan's Law and the distributive law. This approach is also called "Factorization" of the CNF.  However, the application of the distributivity law can lead to an exponential increase of the formula's size.  For example, consider the DNF `X_11 & X_12 | .... | X_n1 & X_n2` which contains `2*n` variables.  Transforming this with the distributive law to a CNF results into `2^n` clauses with `n` variables each.  Fortunately, there are more efficient ways to transform formulas to CNF.

CNF transformations can be categorized into equivalent transformations and equisatisfiable transformations (c.f. [here](..)).  Equivalent transformation *do not* introduce auxiliary variables.  LogicNG offers the following equivalent CNF transformations:

- **CNF Factorization** using the DeMorgan's Laws and the distributive laws: [CNFFactorization](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/cnf/CNFFactorization.java)
- **CNF Transformation using a BDD** of the formula and extracting the paths leading to `false`: [BDDCNFTransformation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/cnf/BDDCNFTransformation.java)

In contrast to the equivalent transformations, equisatisfiable CNF transformations introduce auxiliary variables.  The core idea is to introduce a new auxiliary variable for each sub-formula of the formula's parse tree.  By doing so, the exponential blow up of the size of the resulting formula can be avoided.  LogicNG offers the two most prominent equisatisfiable CNF transformations:

- **Tseitin Transformation** [TseitinTranformation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/cnf/TseitinTransformation.java)
- **Plaisted-Greenbaum Transformation** [PlaistedGreenbaumTransformation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/cnf/TseitinTransformation.java)

For the CNF transformations in this section, consider the formula `f1`:

``` java
Formula f1 = f.parse("A | ~(B & C)");
```


### CNF Factorization

The CNF factorization [CNFFactorization](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/cnf/CNFFactorization.java) is the naive approach to obtain a CNF from a given formula.  Since the CNF factorization easily blows up, it should only be used for *small* formulas.

In order to have control over the computation, one can use the CNF factorization with a [FactorizationHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/FactorizationHandler.java).  With such a handler one can specify when the computation shall be aborted. During the CNF factorization, the handler receives the following information:

- That the computation was `started()` (here you can for example do some initialization work)
- That a distribution was performed (`performedDistribution()`). You can ignore this information or, as in the example below, increase a counter and you have to return a boolean indicating whether the computation should succeed or not (thus be aborted).
- That a new clause was created in the result.  Similarly to `performedDistribution()` you can store some information and you have to return a boolean to succeed or abort the computation.

If `performedDistribution()` or `createdClause()` return `false`, the computation is aborted and will return `null`.  There is also the method `aborted()` with which you can ask the handler whether the computation was aborted or not. In the following example this method is also used internally to define the abortion criterion.

As an example, one can implement a factorization handler which aborts the computation as soon as either 4 distributions were performed or 3 clauses were created:

``` java
FactorizationHandler handler = new FactorizationHandler() {
   private int dists = 0;
   private int clauses = 0;

   @Override
   public void started() {
       this.dists = 0;
       this.clauses = 0;
   }

   @Override
   public boolean performedDistribution() {
       this.dists++;
       return !aborted();
   }

   @Override
   public boolean createdClause(Formula clause) {
       this.clauses++; // (1)!
       return !aborted();
   }

   @Override
   public boolean aborted() {
       return this.dists >= 4 || this.clauses >= 3; // (2)!
   }
};
CNFFactorization factorization = new CNFFactorization(handler);
Formula result = factorization.apply(f1, true);
```

1. You can also store more complex information based on the clause, if you wish
2. Define any abortion constraint here which you think is suitable for you application

The result is `(~B | A) & (~C | A)`.

!!! warning "Impact of Handlers"
    Note that with this handler, the variation `f2 = A | ~(B | C | D)` could not be factorized. The factorization we expect from this formula is `(~B | A) & (~C | A) & (~D | A)`, but this has 3 clauses and performs 4 distributions and thus exceeds both distribution and created clause boundary. A handler with distribution boundary 4 and created clause boundary 3 would solve it.


### BDD CNF Transformation

The [BDDCNFTransformation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/cnf/BDDCNFTransformation.java) transforms a formula to a CNF by converting it to a Binary Decision Diagram (BDD) first.  The conversion of a BDD to a CNF is fast.  However, the generated BDD may have an exponential size compared to the input formula.  We recommend using this transformation if you're interested in a CNF without auxiliary variables and your formula is too big for the `CNFFactorization`.  For more information on BDDs check out the chapter on [knowledge compilation](../../../../knowledge-compilation) and the chapter on [BDDs](../../../../knowledge-compilation/bdd).

The transformation has three constructors:

1. The simplest constructor has no parameters:

   ``` java
   BDDCNFTransformation transformation = new BDDCNFTransformation();
   ```

2. There is a constructor with a BDD kernel (for details check out the BDD chapter mentioned above):
   For example, for some formula `f1`, that is:

   ``` java
   BDDKernel bddKernel = new BDDKernel(f1.factory(), f1.variables().size(), 10, 15);
   BDDCNFTransformation transformation = new BDDCNFTransformation(bddKernel);
   ```

3. There is a constructor with a formula factory and a number of variables:

   ``` java
   BDDCNFTransformation transformation = new BDDCNFTransformation(f, 15);
   ```

Consider the formula

``` java
Formula f1 = f.parse("(x1 <=> x2) | x3");
```

When `transformation` is being created in one of the ways above, and we call

``` java
Formula result = f1.transform(transformation);
```

then we find that the result of the transformation is `(x1 | ~x2 | x3) & (~x1 | x2 | x3)`.

When creating the transformation as in case 2. and 3., keep the following in mind: You can create arbitrarily many transformations with the object. However, the number of different variables in all applied formulas *must not exceed* the number of variables in the kernel (in the example that is `15`), respectively the number of variables (in the example that is `#!java f1.variables().size()`).


### Tseitin Transformation

The Tseitin Transformation [TseitinTransformation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/cnf/TseitinTransformation.java) works on the NNF of a given formula. Each sub-formula of the formula's parse tree is replaced by a new auxiliary variable and an equivalence between the new variable and the sub-formula is added. Equal sub-formulas are replaced by the same auxiliary variable for efficiency.

Let's start with the NNF of `f1`:

```
f1.nnf() = A | ~B & ~C
```

For this NNF the equivalences are:

- `x1 <=> ~B & ~C`
- `x2 <=> A | x1`

Then the conjunction of the variable at the root node and those equivalences yields the Tseitin CNF:

```
tseitin = x2 & (x2 <=> A | x1) & (x1 <=> ~B & ~C)
```

Next, the equivalences are each transformed into CNF.  So for this example, this yields

```
tseitinCNF = x2 & (~x1 | ~B) & (~x1 | ~C) & (B | C | x1) & (~x2 | A | x1) &
             (~A | x2) & (~x1 | x2).
```

!!! info "Tseitin Transformation and NNF"
    Note that it is not necessary to transform the formula to NNF before calling the Tseitin Transformation.  The transformation will perform the NNF transformation on its own if the formula is not already in NNF.


### Plaisted-Greenbaum Transformation

The Plaisted-Greenbaum (PG) Transformation [PlaistedGreenbaumTransformation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/cnf/PlaistedGreenbaumTransformation.java) tries to improve the size of the resulting CNF compared to the Tseitin transformation by only introducing implications instead of equivalences.

The PG transformation transforms the given formula into NNF before the sub-formulas are replaced.  This simplifies the algorithm since only a few cases have to be considered for the structure of the formula.  However, there is a special version of PG in LogicNG which also works for formulas in general by introducing an implication according to the *polarity* of the sub-formula.  The polarity of a node is the number of negations on the path from the root to this node.  If the number of negations is even, then the node has *positive* polarity, else it has *negative* polarity.  This [version](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/cnf/PlaistedGreenbaumTransformationSolver.java) is a special implementation used for adding formulas to the SAT solver and there should be no need to ever use it manually.

If a subtree `s` has a positive polarity, then the implication `x => s`, `x` being a newly introduced variable, is added to the conjunction of clauses which yields the PG transformation. Otherwise, if the polarity is negative, the implication `s => x` is added.

Let's consider our example `A | ~(B & C)`:

- `B & C ` has negative polarity (the relevant number of negations is `1`), thus add `B & C => x1` to the conjunction
- `A | ~x1` has positive polarity (the relevant number of negations is `0`), thus add `x2 => A | ~x1` to the conjunction

Note that nodes whose operator is `NOT` (as in `~(B & C)`) are condensed with the parent node.

Analogously to the Tseitin transformation, the conjunction of the variable at the root node and those equivalences yields the PG CNF:

```
pg = x2 & (x2 => A | ~x1) & (B & C => x1)
```

The resulting CNF is

```
pgCNF = x2 & (~x2 | A | ~x1) & (~B | x1) & (~C | x1)
```

Clearly, the CNF resulting from applying the Plaisted-Greenbaum transformation is smaller than the formula resulting from the Tseitin transformation. However, in contrast to the Tseitin transformation, the PG transformation *does not preserve the number of satisfying assignments* of the original formula (c.f. [Model Counting](../../../../model-counting-enumeration)).  That is: When applying the Tseitin transformation, the model count of the original formula is preserved, but when applying PG transformation, it is not.  Therefore, when one wants to count models with methods which require a formula in CNF as the input, one should not use PG to transform the formula.


## DNF transformations

A DNF is a disjunction of conjunctions of literals.  The conjunction of literals is called a "min-term" or just "term".  So the formula

```
(a & b & ~c) | (~a & c) | (d & f) | (e & ~c)
```

is a DNF with four terms, the first contains three literals, the other two literals.


For the following examples consider

```java
Formula f2 = f.parse("(A | B) & C");
```

### DNF Factorization

Similar to the `CNFFactorization`, the [DNFFactorization](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/dnf/DNFFactorization.java) is the naive implementation of a DNF conversion using mainly the distributive law.  For example, the DNF of `f2` is `A & C | B & C`.

!!! warning "Exponential Blow-Up"
    Note that analogously to the CNF factorization this can lead to an exponential blow-up and is thus not recommended for big formulas.  DNF factorizations can, just as CNF factorizations, be controlled with a `FactorizationHandler`.


### Canonical CNF and DNF Enumeration

The canonical DNF enumeration [CanonicalDNFEnumeration](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/dnf/CanonicalDNFEnumeration.java) creates a DNF, in which every term contains every variable of the initial formula.  Therefore, every possible assignment of the DNF occurs as a term in the DNF. Thus, this DNF canonically describes a Boolean formula.

For example, for `f2 = (A | B) & C`, the canonical DNF is

```
~A & B & C | A & B & C | A & ~B & C
```

It is also possible to create a DNF via BDDs, see the [BDD](../../../../knowledge-compilation/bdd) chapter.

Note that all DNF transformations can lead to an exponential increase in the size of the formula.

[:octicons-tag-24: 2.3.0](https://github.com/logic-ng/LogicNG/releases/tag/v2.3.0)  There is also a canonical CNF tranformation [CanonicalCNFEnumeration](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/cnf/CanonicalCNFEnumeration.java) which conversely generates a canonical CNF where each clause contains all literals of the given formula.  The canonical CNF of `f2` e.g. is

```
(A | B | C) & (A | ~B | C) & (~A | ~B | C) & (~A | B | C) & (A | B | ~C)
```


## AIG Transformation

Using the AIG transformation [AIGTransformation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/AIGTransformation.java) one can transform any formula to an And-inverter graph.  An AIG consists only of conjunctions and negations.  For information about AIG graphs check out the excursion on [And-Inverter Graphs](../../formula-predicates#syntactical-predicates). For an example look at the following transformation:

``` java
Formula f3 = f.parse("A => ~(B | ~C)");
Formula aig = f3.transform(new AIGTransformation());
```

The result of this transformation is the formula

```
~(A & ~(~B & C))
```

so a formula with only conjunctions and negations.


