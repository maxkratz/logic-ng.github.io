---
title: BDD
---

## Introduction

A Binary Decision Diagram (BDD) is a directed acyclic graph of a given formula. It has a single root; Each inner node is labeled with a propositional variable and has one outgoing edge for a positive assignment, and one edge for a negative assignment of the respective variable. The leaves are labeled with `1` and `0` representing`true` and `false`. An assignment is represented by a path from the root node to a leaf and its evaluation is the respective value of the leaf. Therefore, all paths to a 1-leaf are valid (possibly partial) models for the formula.

LogicNG's BDD implementation is a Java implementation of the BDD package [BuDDy](http://buddy.sourceforge.net/manual/main.html).  All classes in the package `jbuddy` are part of the implementation.  The data structures in this package are very low-level and only used for working with BDDs internally.  If you want a "nicer" data structure for working with BDDs you can use the LogicNG BDD data structures in the package `datastructures` which lean more on a graphical representation of BDDs.  One can create the LogicNG internal data structure of a BDD using the `LNGBDDFunction`, see below.

Some content in this chapter, such as the description of BDDs above and the example below are taken from chapter 2.3.1 in [New Formal Methods for Automotive Configuration](https://publikationen.uni-tuebingen.de/xmlui/bitstream/handle/10900/57198/dissertation.pdf?sequence=1&isAllowed=y).


### Example

As an example for a [BDD](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/BDD.java), consider the formula

```
f1 = (x1 <=> x2) | x3
```

The BDD of `f1` is:

``` mermaid
graph TD
  id0["false"]
    style id0 stroke:#ea2027,color:#ffffff,fill:#ea2027
  id1["true"]
    style id1 stroke:#009432,color:#ffffff,fill:#009432
  id6(["x3"])
  id11(["x2"])
  id12(["x2"])
  id13(["x1"])

  id6 --> id0
    linkStyle 0 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id6 --> id1
    linkStyle 1 stroke:#009432,stroke-width:2px
  id11 --> id1
    linkStyle 2 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id11 --> id6
    linkStyle 3 stroke:#009432,stroke-width:2px
  id12 --> id6
    linkStyle 4 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id12 --> id1
    linkStyle 5 stroke:#009432,stroke-width:2px
  id13 --> id11
    linkStyle 6 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id13 --> id12
    linkStyle 7 stroke:#009432,stroke-width:2px
```

The green lines represent positive assignments of the respective variable, the red dotted lines represent negative assignments of the respective variable. That is, for some given assignment:

If, in this assignment, `x1 = 1` (meaning `x1` is assigned to `true`), then we follow the green path, else we follow the red path. Then, at `x2`, check if `x2 = 1` in the given assignment. If it is, follow the green path, else follow the red path, etc. The applied variable ordering is `x1 > x2 > x3`, meaning that on any path the variables appear in this order. As we will find out below, the variable ordering plays a key role for the size of the BDD.  In order to see that the BDD graph indeed represents `f1`, one can try out some data points. For example,

- `f1(1,0,1) = 1`
- `f1(1,0,0) = 0`


### Creating a BDD graph

You can create your own BDD graph in the [dot format](https://graphviz.org/doc/info/lang.html) using the [BDDDotFileWriter](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/io/BDDDotFileWriter.java). The syntax is, for a given formula `f1`:

```java
BDDDotFileWriter.write("name your BDD", f1.bdd());
```

## Variable Ordering

The structure and the size (number of nodes) of a BDD strongly depends on the specified variable ordering. There are examples where a bad variable ordering produces an exponential size BDD and a good variable ordering can lead to a linear size BDD.

Consider the formula

```
x1 & x2 | x3 & x4 | ... | x(2n-1) & x2n
```

1. Given the variable ordering
    ```
    x1 < x3 < ... < x(2n-1) < x2 < x4 < ... < x2n,
    ```
   the resulting BDD has more than `2^n` nodes.

2. Given the variable ordering
    ```
    x1 < x2 < ... < x2n,
    ```
   the BDD consists of `2n + 2` nodes.

For more information see [wikipedia](https://en.wikipedia.org/wiki/Binary_decision_diagram).

However, finding an optimal variable ordering is NP-complete.

In LogicNG, some common heuristics for BDD variable orderings are implemented:


### Sorting Based on Variables' Occurrences

The variable orderings [MinToMaxOrdering](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/orderings/MinToMaxOrdering.java) and [MaxToMinOrdering](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/orderings/MaxToMinOrdering.java) sort the variables from minimal to maximal (respectively, maximal to minimal) occurrence in the input formula. If two variables have the same number of occurrences, their ordering according to their DFS ordering (see below) will be considered.


### Sorting Based on Breath-First-Search and Depth-First-Search

The [DFSOrdering](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/orderings/DFSOrdering.java) traverses the formula in a DFS manner and gathers all variables in the occurrence. Analogously, the [BFSOrdering](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/orderings/BFSOrdering.java) traverses the formula in a BFS manner and gathers all variables in the occurrence. Check out how to traverse a formula in DFS manner (and, respectively, BFS manner) [here](https://www.geeksforgeeks.org/dfs-traversal-of-a-tree-using-recursion/).


### Force Ordering

The [ForceOrdering](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/orderings/ForceOrdering.java) is a simple implementation of the "FORCE" variable ordering heuristic based on hyper-graphs presented by [Aloul, Markov, and Sakallah](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.75.3561&rep=rep1&type=pdf). This ordering only works for CNF formulas, thus the given formula has to be converted to CNF before this ordering is called.  One can transform a formula to CNF using one of the [CNF transformations](../../formulas/operations/transformations/normal-form-transformations#cnf-transformations).

The code snippet to create a variable ordering (here with the example of a `DFSOrdering`) for a formula `f1` is:

```java
List<Variable> ordering = new DFSOrdering().getOrder(f1);
```

A practical example for the significance of the right variable ordering is the following:

Consider the following formula `f1` which we want to transform into a BDD and compute the number of clauses of the CNF representation of the BDD.

```java
final Formula f1 = f.parse("(~(v372 | v2095 | v2096 | v683 | v1629 | v1655 | v1487 | v141 | v1509 | v743 | v2137 | v1622 | v2276 | v811 | v2277 | v782 | v39 | " +
   "v1900 | v1375 | v1376 | v2113 | v2114 | v17 | v18) & v1604 | (v17 | v18) & ~(v372 | v2095 | v2096 | v683 | v1455 | v1655 | v1487 | " +
   "v1509 | v743 | v2137 | v1356 | v1622 | v2276 | v811 | v2277 | v782 | v39 | v1900 | v1375 | v1376 | v2113 | v2114) & v1604) & (v1421 " +
   "& v1455 & v1457 & v675 & v676 & v690 & v504 & v708 & v405 & v1669 & v1467 & v1466 & v1570 & v1472 & v1493 & v1454 & v507 & v695 & " +
   "v1469 & v1481 & ~(v17 | v18) & v1604 | (v17 | v18) & v1457 & v675 & v676 & v690 & (v707 | v708) & v2254 & v405 & v1669 & v1361 & " +
   "v1563 & v1467 & v1466 & v1570 & v1493 & v695 & v508 & v1469 & v1474 & v1483 & v1604) & (v1539 & v356 & v772 & v769 & v987 & v1607 & " +
   "(v1645 | v506) & v790 & v974 & v1486 & v512 & (v865 | v866 | v876 | v886 | v891) & v854 & v696 & (v801 | v836 | v832) & (v316 | " +
   "v344) & v1604 | v1539 & v357 & v682 & v1350 & v684 & v511 & v1449 & (v886 | v891 | v864 | v865 | v866) & v1354 & v1607 & v2172 & " +
   "v2117 & v862 & (v836 | v827 | v832) & v1604 | v1539 & v358 & v1640 & v769 & v1607 & v682 & v790 & v684 & v1486 & v511 & v1449 & " +
   "v1354 & v1356 & (v865 | v866 | v876 | v886 | v891) & v854 & v506 & v862 & (v836 | v827 | v832) & (v316 | v344) & v1604 | v17 & v356 " +
   "& v770 & v769 & (v500 | v503) & v512 & v1486 & (v865 | v886 | v891 | v898) & v854 & v1424 & (v836 | v1209 | v837 | v828) & (v316 | " +
   "v344) & v1604 | v17 & v771 & v770 & v769 & v1374 & v1604 | v17 & v505 & v769 & v682 & v684 & v1486 & v511 & v1449 & v1354 & (v865 | " +
   "v886 | v891 | v898) & v854 & v503 & v1424 & v862 & v1494 & v361 & (v836 | v837 | v827 | v828) & (v316 | v344) & v1604 | v17 & v359 &" +
   " v682 & v1625 & v1350 & v684 & v511 & v1449 & (v865 | v886 | v891 | v898) & v1354 & v1607 & v2172 & v2117 & v862 & v1494 & v361 & " +
   "(v836 | v837 | v827 | v828) & v1604) & (v1539 & v357 & v1604 | ~(v1401 | v781 | v684 | v1449 | v1354 | v2117 | v1494) & v1604 | v17 " +
   "& v505) & v391 & v1604");

final BDD bdd = formula.bdd(VariableOrdering.BFS); // (1)!
final BigInteger numberOfClauses = bdd.numberOfClausesCNF();
```

1. replace respectively by the other orderings

Depending on the variable ordering, we find that the resulting BDD has a very different number of CNF clauses:

| Variable Ordering | Number of Clauses |
| :---------------- | ----------------: |
| `MAX2MIN`         | 5.646.676.904     |
| `FORCE`           | 22.429.167        |
| `DFS`             | 8.963.730         |
| `MIN2MAX`         | 3.685.413         |
| `BFS`             | 1.182.347         |

So the number of clauses in the CNF varies from 1.182 million to 5.646 billion, depending on the variable ordering. This is a factor of ~5.000!

!!! warning
    This example does not mean, that in general the BFS heuristic performs better than any other heuristic.  One could easily find another example formula where the numbers are exactly opposite.


### BDD Reorderings

When you created a BDD with a given ordering and want to improve the ordering, you can *reorder* the given BDD. Reordering a BDD is usually more efficient than creating a new one with a different ordering. For example, consider the following formula and its respective BDD:

```java
Formula phi = f.parse("(x1 <=> x2) | x3 | x4");
BDD bdd = phi.bdd();
```

We can perform a random reordering with a [BDDReordering](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/jbuddy/BDDReordering.java) in the following way:

```java
BDDReordering bddReordering = new BDDReordering(bdd.underlyingKernel()); // (1)!
bddReordering.addVariableBlockAll(); // (2)!
bddReordering.reorder(BDDReorderingMethod.BDD_REORDER_RANDOM); // (3)!
```

1. create a BDDReordering
2. adds a single variable block for all variables known by the kernel
3. reorders randomly

The result is:


<table>
<tr>
  <th>Original BDD</th>
  <th>BDD after Reordering</th>
</tr>
<tr>
  <td>
``` mermaid
graph TD
  id0["false"]
    style id0 stroke:#ea2027,color:#ffffff,fill:#ea2027
  id1["true"]
    style id1 stroke:#009432,color:#ffffff,fill:#009432
  id8(["x4"])
  id16(["x3"])
  id17(["x2"])
  id18(["x2"])
  id19(["x1"])

  id8 --> id0
    linkStyle 0 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id8 --> id1
    linkStyle 1 stroke:#009432,stroke-width:2px
  id16 --> id8
    linkStyle 2 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id16 --> id1
    linkStyle 3 stroke:#009432,stroke-width:2px
  id17 --> id1
    linkStyle 4 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id17 --> id16
    linkStyle 5 stroke:#009432,stroke-width:2px
  id18 --> id16
    linkStyle 6 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id18 --> id1
    linkStyle 7 stroke:#009432,stroke-width:2px
  id19 --> id17
    linkStyle 8 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id19 --> id18
    linkStyle 9 stroke:#009432,stroke-width:2px
```
  </td>
  <td>
``` mermaid
graph TD
  id0["false"]
    style id0 stroke:#ea2027,color:#ffffff,fill:#ea2027
  id1["true"]
    style id1 stroke:#009432,color:#ffffff,fill:#009432
  id4(["x2"])
  id5(["x2"])
  id16(["x1"])
  id17(["x3"])
  id18(["x3"])
  id19(["x4"])

  id4 --> id0
    linkStyle 0 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id4 --> id1
    linkStyle 1 stroke:#009432,stroke-width:2px
  id5 --> id1
    linkStyle 2 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id5 --> id0
    linkStyle 3 stroke:#009432,stroke-width:2px
  id16 --> id17
    linkStyle 4 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id16 --> id18
    linkStyle 5 stroke:#009432,stroke-width:2px
  id17 --> id5
    linkStyle 6 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id17 --> id1
    linkStyle 7 stroke:#009432,stroke-width:2px
  id18 --> id4
    linkStyle 8 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id18 --> id1
    linkStyle 9 stroke:#009432,stroke-width:2px
  id19 --> id16
    linkStyle 10 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id19 --> id1
    linkStyle 11 stroke:#009432,stroke-width:2px
```
  </td>
</tr>
</table>

Note that the command `bddReordering.addVariableBlockAll()` is necessary: It means that the reordering should happen between all variables. Alternatively, you can define blocks of variables in which you want to reorder using the method `addVariableBlock()`.


## Creating BDDs

There are essentially two ways to create a BDD: The first one uses the [BDDFactory](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/BDDFactory.java) directly, the second is a "shortcut" and uses the `bdd()`-function on the respective formula.

In this chapter, consider the formula `f1` and the variable ordering `varOrder`.


### Creating BDDs Using a BDD Factory

Firstly, we consider the option of using a `BDDFactory`. BDDs are created with a kernel and, optionally, with a handler. The kernel holds all internal data structures and the internal state for the BDD creation, the handler can control the creation process.


#### BDD Kernel

Let `n` be the number of variables of `f1`. The constructor parameters of the [BDDKernel](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/jbuddy/BDDKernel.java) are the following. One of `numVars` and `ordering` must be present.

| Parameter   | Forced | Default Value           | Description                                       |
| ---------   | ------ | -------------           | -----------                                       |
| `f`         | yes    | Formula factory of `f1` | The formula factory to be used for the creation   |
| `numVars`   | no     | `n`                     | The number of variables                           |
| `ordering`  | no     | (none)                  | The variable ordering                             |
| `nodeSize`  | yes    | 30 * `n`                | The initial number of nodes in the node table     |
| `cacheSize` | yes    | 20 * `n`                | The fixed size of the internal caches.            |


!!! info "Initial `nodeSize`"

    The BDD kernel internally holds a table with all nodes in the BDD. This table can be extended dynamically, but this is an expensive operation. On the other hand, one wants to avoid reserving too much space for nodes, since this costs unnecessary memory. 30 * `x` proved to be efficient in practice for medium sized formulas.

A kernel can be constructed without a variable order, or with an ordering:

**Without a variable ordering**

For example, with `numVars`, `nodeSize` and `cacheSize` all being `10`, then the constructor is:

```java
BDDKernel kernel = new BDDKernel(f1.factory(), 10, 10, 10);
```

The variable ordering is then the ordering in which the variables occur in the formula. This is obviously not a very sensible ordering, therefore it is recommended to create a kernel with a variable ordering.

**With a variable ordering**

For example, with the variable ordering `MaxToMinOrdering` and `nodeSize` and `cacheSize` all being `10`, the constructor is:

```java
List<Variable> ordering = new MaxToMinOrdering().getOrder(f1);
BDDKernel kernel2 = new BDDKernel(f1.factory(), ordering, 10, 10);
```

When you don't define the BDD kernel for your applications, the *standard kernel* configuration with the default values from the table above will be used.


#### BDD Handler

Using a [BDDHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/BDDHandler.java), one can control the BDD compilation.  There are two BDD handler implemented in LogicNG:

1. The [TimeoutBDDHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/TimeoutBDDHandler.java) cancels the compilation of the BDD after the given timeout. There are different timeout types which are described in the class itself.
2. The [NumberOfNodesBDDHandler](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/handlers/NumberOfNodesBDDHandler.java) cancels the compilation of the BDD after a given number of added nodes.

Of course you can always implement your own handler for specific purposes.  To initialize the BDD factory with a handler, see the following code snippet:

```java
BDDKernel myKernel = new BDDKernel(f1.factory(), f1.variables().size(), 10, 15);
BDDHandler myHandler = new TimeoutBDDHandler(100);
BDD bdd = BDDFactory.build(f1, myKernel, myHandler);
```


### Create a BDD without a BDD Factory

The class `Formula` has a method to create BDDs. With this method one cannot set a handler or settings via a kernel. Since without a BDD handler it is impossible to control the building process, these methods should only be called for small formulas. The kernel is the *standard kernel* (see above).

1. Without a variable ordering: `f1.bdd()` returns the BDD of `f1` with the variable ordering like the variables occurr in the formula.
2. With a variable ordering: `f1.bdd(varOrder)` returns the BDD of `f1` with the variable ordering `varOrder`.


## Functions on BDDs

When given a formula in the BDD format, one can execute different functions on them. For this section, consider the BDD created above:

```
BDD bdd = f1.bdd();
```
that is:

``` mermaid
graph TD
  id0["false"]
    style id0 stroke:#ea2027,color:#ffffff,fill:#ea2027
  id1["true"]
    style id1 stroke:#009432,color:#ffffff,fill:#009432
  id6(["x3"])
  id11(["x2"])
  id12(["x2"])
  id13(["x1"])

  id6 --> id0
    linkStyle 0 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id6 --> id1
    linkStyle 1 stroke:#009432,stroke-width:2px
  id11 --> id1
    linkStyle 2 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id11 --> id6
    linkStyle 3 stroke:#009432,stroke-width:2px
  id12 --> id6
    linkStyle 4 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id12 --> id1
    linkStyle 5 stroke:#009432,stroke-width:2px
  id13 --> id11
    linkStyle 6 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id13 --> id12
    linkStyle 7 stroke:#009432,stroke-width:2px
```

There are a couple of functions to query the internal properties of the BDD:

- `underlyingKernel()` returns the kernel of the BDD
- `bddNodeCount()` counts the number of distinct nodes for the BDD. For `bdd`, it is `4`
- `getVariableOrder()` returns the variable order of the BDD. For `bdd`, it is `[x1, x2, x3]`
- `pathCountZero()` and `pathCountOne()` return the number of paths leading to the terminal `0` (false), respectively, `1` (true) node


### Tautology and Contradiction Check

To check whether a BDD is a tautology or a contradiction, you can call the two methods `isTautology()` and `isContradiction()`.  This is one of the questions which can be answered very easily and in fact in *constant time* on a BDD: a tautology BDD has only one node `1` and a contradiction has only one node `0`.


### Variable Profile
The method `variableProfile()` computes the how often each variable occurs in the BDD, i.e. how many nodes are there for each variable. The syntax is:

```java
SortedMap<Variable, Integer> variableIntegerSortedMap = bdd.variableProfile();
```

The result is `{x1=1, x2=2, x3=1}`, meaning that `x1` occurs `once` in the BDD, `x2` occurs twice in the BDD and `x3` occurs also just once in the BDD.

!!! info "Different Variable Profiles"

    Note that the result of the `variableProfile` of the BDD is usually different from the [variable profile on the original formula](../../formulas/operations/formula-functions#compute-the-variables-and-literals-of-a-formula):

```java
Formula f1 = f.parse("(x1 <=> x2) | x3"); // the initial formula of bdd
Map<Variable, Integer> variableProfile = f1.apply(new VariableProfileFunction());
```

The result is `{x1=1, x3=1, x2=1}`, as `x2` occurs in the *formula* only once.


### Variable Support

The *support* of a BDD is the set of variables which the BDD depends on, meaning the opposite of *don't-care* variables. The support is not necessarily equivalent to the set of variables of the formula with which the BDD was created. In order to see this, consider the example of a tautology:

```java
Formula f2 = f.parse("A | B | ~A & ~B");
```

Then the support of the formula can be computed via

```java
SortedSet<Variable> support = f2.bdd().support();
```
Thus, the support is empty, but the variables of `f2` are `A, B`. If we added a variable which is relevant to the BDD to the formula, say

```java
Formula f2 = f.parse("(A | B | ~A & ~B) & C");
```

then the support is `C`, as it is the only variable which the BDD depends on.


### Swapping Variables in the Orderung

You can change the initial variable ordering manually (in contrast to automatic variable reordering) using `swapVariables()`. For example, swapping the variables `x1` and `x2` can be achieved with:

```java
bdd.swapVariables(f.variable("x1"), f.variable("x2"));
```

The result is:

<table>
<tr>
  <th>Original BDD</th>
  <th>BDD after Swapping x1 and x2</th>
</tr>
<tr>
  <td>
``` mermaid
graph TD
  id0["false"]
    style id0 stroke:#ea2027,color:#ffffff,fill:#ea2027
  id1["true"]
    style id1 stroke:#009432,color:#ffffff,fill:#009432
  id6(["x3"])
  id11(["x2"])
  id12(["x2"])
  id13(["x1"])

  id6 --> id0
    linkStyle 0 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id6 --> id1
    linkStyle 1 stroke:#009432,stroke-width:2px
  id11 --> id1
    linkStyle 2 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id11 --> id6
    linkStyle 3 stroke:#009432,stroke-width:2px
  id12 --> id6
    linkStyle 4 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id12 --> id1
    linkStyle 5 stroke:#009432,stroke-width:2px
  id13 --> id11
    linkStyle 6 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id13 --> id12
    linkStyle 7 stroke:#009432,stroke-width:2px
```
  </td>
  <td>
``` mermaid
graph TD
  id0["false"]
    style id0 stroke:#ea2027,color:#ffffff,fill:#ea2027
  id1["true"]
    style id1 stroke:#009432,color:#ffffff,fill:#009432
  id6(["x3"])
  id8(["x1"])
  id9(["x1"])
  id13(["x2"])

  id6 --> id0
    linkStyle 0 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id6 --> id1
    linkStyle 1 stroke:#009432,stroke-width:2px
  id8 --> id1
    linkStyle 2 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id8 --> id6
    linkStyle 3 stroke:#009432,stroke-width:2px
  id9 --> id6
    linkStyle 4 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id9 --> id1
    linkStyle 5 stroke:#009432,stroke-width:2px
  id13 --> id8
    linkStyle 6 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id13 --> id9
    linkStyle 7 stroke:#009432,stroke-width:2px
```
  </td>
</tr>
</table>


### CNF and DNF generation

CNF and DNF creation on a BDD is very easy: For a DNF just enumerate all paths to the 1-node, for a CNF enumerate all paths to the 0-node and negate them. The function `numberOfClausesCNF()` counts the number of clauses of the CNF without building the CNF.

This is very helpful if you want to know if the resulting CNF is feasible to work with or not.  A seen in the examples above: harmless looking formulas can generate CNFs with more than 5.000 clauses.  But sometimes the CNF generated with the BDD can be a viable alternative for a factorization-based CNF without auxilliary variables.  Therefore there is a `FormulaTransformation` you can use directly on formulas to generate a BDD-based CNF: `BDDCNFTransformation`.

An example for these functions is:

```java
Formula cnf = bdd.cnf();
BigInteger numberOfClauses = bdd.numberOfClausesCNF();
```

The number of clauses is `2` and the result is
`(x1 | ~x2 | x3) & (~x1 | x2 | x3)`.

[:octicons-tag-24: 2.3.0](https://github.com/logic-ng/LogicNG/releases/tag/v2.3.0)  The method `dnf()` on a `BDD` returns the DNF of the BDD in the same way as the above desribed CNF.  There is also a formula transformation `BDDDNFTransformation` to use a BDD-based DNF generation directly on a formula.


### Model Counting and Enumeration

Counting and listing all possible models of a BDD is simple and fast since this simply requires following all paths which lead to "true".  This is another algorithmic approach for [model counting and enumeration](../../model-counting-enumeration).  For the example of `f1`, the syntax is

```java
BigInteger modelCount = bdd.modelCount();
List<Assignment> models = bdd.enumerateAllModels();
```

The model count is `6` and the resulting models are:

```
- x1, x2, x3
- ~x1, x2, x3
- x1, ~x2, x3
- x1, x2, ~x3
- ~x1, ~x2, x3
- ~x1, ~x2, ~x3
```

`enumerateAllModels()` has an optional parameter if you want to specify the set of variables over which you enumerate (just like the `enumerateAllModels()` method on the solver) for projected model enumeration.  For more information, see the chapter on [model enumeration](../../model-counting-enumeration).


### Finding One Model

If you are only interested in *one* model, there are three options to find one:

Firstly, you can use `model()`, which returns the variables of one path which leads to "true". This assignment is *not* complete; it contains only those variables which were on the path to `true`, which can be a subset of all variables occurring in the BDD.  All variables not occuring are *don't-care* variables.

For example:

``` java
Assignment model = bdd.model();

```
returns `Assignment{pos=[], neg=[~x2, ~x1]}`.

This is not a complete model, since `x3` is not assigned it is a don't-care varialbe, meaning that adding either `~x3` or `x3` to the partial assignment results in a valid model.

Secondly, if you want to specify which value the don't-care variables in the resulting model should have, you can use the method `model()` with extra parameters. The first parameter specifies to which value the don't-care variables should be set in the model (true/false), and the second parameter specifies which variables should be contained in the model.

For example,

``` java
Assignment model1 = bdd.model(true, f1.variables()); // (1)!
Assignment model2 = bdd.model(false, f1.variables()); // (2)!
```

1. don't-care variables are set to `true`
2. don't-care variables are set to `false`

returns the following results:

- `model1`: `Assignment{pos=[x3], neg=[~x2, ~x1]}`
- `model2`: `Assignment{pos=[], neg=[~x3, ~x2, ~x1]}`

Thirdly, if you don't-care what value the don't-care variables have, you can use the method `fullModel()`. This method returns a complete model with an arbitrary assignment of the don't care variables.

The syntax is:

```java
Assignment assignment = bdd.fullModel();
```

and the result is `Assignment{pos=[], neg=[~x3, ~x2, ~x1]}`.


### Restriction

Using the function `restrict()` one can restrict the BDD with literals, i.e. setting the variables to a fixed value and simplifying the BDD - like restriction on formulas.  As an example, we restrict the given BDD by `x2`.

```java
BDD bddRestricted = bdd.restrict(f.literal("x2", true));
```

The result is:

<table>
<tr>
  <th>Original BDD</th>
  <th>BDD after Restricting x2</th>
</tr>
<tr>
  <td>
``` mermaid
graph TD
  id0["false"]
    style id0 stroke:#ea2027,color:#ffffff,fill:#ea2027
  id1["true"]
    style id1 stroke:#009432,color:#ffffff,fill:#009432
  id6(["x3"])
  id11(["x2"])
  id12(["x2"])
  id13(["x1"])

  id6 --> id0
    linkStyle 0 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id6 --> id1
    linkStyle 1 stroke:#009432,stroke-width:2px
  id11 --> id1
    linkStyle 2 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id11 --> id6
    linkStyle 3 stroke:#009432,stroke-width:2px
  id12 --> id6
    linkStyle 4 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id12 --> id1
    linkStyle 5 stroke:#009432,stroke-width:2px
  id13 --> id11
    linkStyle 6 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id13 --> id12
    linkStyle 7 stroke:#009432,stroke-width:2px
```
  </td>
  <td>
``` mermaid
graph TD
  id0["false"]
    style id0 stroke:#ea2027,color:#ffffff,fill:#ea2027
  id1["true"]
    style id1 stroke:#009432,color:#ffffff,fill:#009432
  id6(["x3"])
  id14(["x1"])

  id6 --> id0
    linkStyle 0 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id6 --> id1
    linkStyle 1 stroke:#009432,stroke-width:2px
  id14 --> id6
    linkStyle 2 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id14 --> id1
    linkStyle 3 stroke:#009432,stroke-width:2px
```
  </td>
</tr>
</table>


### Quantifier Elimination

You can perform existential and universal quantifier elimination for a given set of variables using the `exists()` and `forall()` methods.  An introduction to Boolean quantifier elimination can be found e.g. in [Parametric Quantified SAT Solving by Sturm and Zengler](https://dl.acm.org/doi/abs/10.1145/1837934.1837954).  For example, if we want to eliminate the universally quantified variable `x3` from the BDD, we could perform the following code:

``` java
BDD qe = bdd.forall(f.variable("x3"));
```

The result is

<table>
<tr>
  <th>Original BDD</th>
  <th>BDD after Universally Eliminating x3</th>
</tr>
<tr>
  <td>
``` mermaid
graph TD
  id0["false"]
    style id0 stroke:#ea2027,color:#ffffff,fill:#ea2027
  id1["true"]
    style id1 stroke:#009432,color:#ffffff,fill:#009432
  id6(["x3"])
  id11(["x2"])
  id12(["x2"])
  id13(["x1"])

  id6 --> id0
    linkStyle 0 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id6 --> id1
    linkStyle 1 stroke:#009432,stroke-width:2px
  id11 --> id1
    linkStyle 2 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id11 --> id6
    linkStyle 3 stroke:#009432,stroke-width:2px
  id12 --> id6
    linkStyle 4 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id12 --> id1
    linkStyle 5 stroke:#009432,stroke-width:2px
  id13 --> id11
    linkStyle 6 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id13 --> id12
    linkStyle 7 stroke:#009432,stroke-width:2px
```
  </td>
  <td>
``` mermaid
graph TD
  id0["false"]
    style id0 stroke:#ea2027,color:#ffffff,fill:#ea2027
  id1["true"]
    style id1 stroke:#009432,color:#ffffff,fill:#009432
  id4(["x2"])
  id5(["x2"])
  id10(["x1"])

  id4 --> id0
    linkStyle 0 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id4 --> id1
    linkStyle 1 stroke:#009432,stroke-width:2px
  id5 --> id1
    linkStyle 2 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id5 --> id0
    linkStyle 3 stroke:#009432,stroke-width:2px
  id10 --> id5
    linkStyle 4 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id10 --> id4
    linkStyle 5 stroke:#009432,stroke-width:2px
```
  </td>
</tr>
</table>


### Creating the LogicNG-internal BDD Data Structure

Sometimes it can be useful to create a data structure which leans on the graphical representation of BDDs. Consider the representation of `f1` from above:

``` mermaid
graph TD
  id0["false"]
    style id0 stroke:#ea2027,color:#ffffff,fill:#ea2027
  id1["true"]
    style id1 stroke:#009432,color:#ffffff,fill:#009432
  id6(["x3"])
  id11(["x2"])
  id12(["x2"])
  id13(["x1"])

  id6 --> id0
    linkStyle 0 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id6 --> id1
    linkStyle 1 stroke:#009432,stroke-width:2px
  id11 --> id1
    linkStyle 2 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id11 --> id6
    linkStyle 3 stroke:#009432,stroke-width:2px
  id12 --> id6
    linkStyle 4 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id12 --> id1
    linkStyle 5 stroke:#009432,stroke-width:2px
  id13 --> id11
    linkStyle 6 stroke:#ea2027,stroke-width:2px,stroke-dasharray:3
  id13 --> id12
    linkStyle 7 stroke:#009432,stroke-width:2px
```

The LogicNG internal BDD data structure has a super class for all nodes: [BDDNode](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/datastructures/BDDNode.java), a class for inner nodes representing variables [BDDInnerNode](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/datastructures/BDDInnerNode.java), and a class for terminals of the BDD [BDDConstant](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/datastructures/BDDConstant.java).  Any inner node holds the variable it represents, it's high-edge, meaning the variable is assigned to `true` (green line in graph), and it's low-edge (red dotted line in graph), meaning the variable is assigned to `false`.

This LogicNG internal data structure of a given BDD can be created using the [LNGBDDFunction](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/knowledgecompilation/bdds/functions/LngBDDFunction.java) or with the auxiliary method `toLngBdd()`.

For example, given

```java
BDDNode node = bdd.toLngBdd();
```

then the result of the function is the representation of `f1` with the LogicNG internal representation:

```
<x1 |
 low=< x2 |
    low=<$true>
    high=<x3 |
      low=<$false>
      high=<$true>>
 >
 high=< x2 |
    low=<x3 |
      low=<$false>
      high=<$true>
    >
    high=<$true>
 >
>
```

