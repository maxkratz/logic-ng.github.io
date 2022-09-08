---
title: Graphs
---

LogicNG has a very simple implementation of graph and hypergraph data structures with some useful algorithms implemented on them.  These can in no way compete with dedicated graph libraries, but provide enough performance and capabilities to be useful for certain scenarios.  There are two main use cases:

1. Generating constraint graphs for formulas
2. Generating hypergraphs for formulas


## Constraint Graphs and Connected Components

[Constraint graphs](https://en.wikipedia.org/wiki/Constraint_graph) can be used to represent relations among different formulas. The nodes of the graph are the variables in the formulas, and the edges indicate whether two variables occur in the same formula.

In LogicNG, you can generate a constraint graph with the [ConstraintGraphGenerator](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/graphs/generators/ConstraintGraphGenerator.java) in the following way:

``` java
List<Formula> formulas = Arrays.asList(f.parse("A | ~B | C"),
        f.parse("D | ~A"), f.parse("D + E = 1"), f.parse("G"));
Graph<Variable> constraintGraph =
        ConstraintGraphGenerator.generateFromFormulas(formulas);
```

In this example, the result is a graph looking like this:

``` mermaid
graph TD
  id0(("A"))
    style id0 stroke:#009432,color:#ffffff,fill:#009432
  id1(("B"))
    style id1 stroke:#009432,color:#ffffff,fill:#009432
  id2(("C"))
    style id2 stroke:#009432,color:#ffffff,fill:#009432
  id3(("D"))
    style id3 stroke:#009432,color:#ffffff,fill:#009432
  id4(("E"))
    style id4 stroke:#009432,color:#ffffff,fill:#009432
  id5(("G"))
    style id5 stroke:#009432,color:#ffffff,fill:#009432

  id0 --- id1
  id0 --- id2
  id0 --- id3
  id1 --- id2
  id3 --- id4
```

!!! tip "Application Insight"

    The constraint graph itself can be interesting for a visualization of formulas and their dependencies, but one can also use the constraint graph to improve computations.  For example one can compute strongly connected components of the graph.  A strongly connected component is a set of nodes where each node is reachable by all other nodes via some edges.  If a constraint graph has more than one connected component, some algorithms can be performed independently on the single components, therefore allowing parallelization and complexity reductions.  E.g. if you want to compute the model count for a large set of formulas and the constraint graph has three components `A`, `B`, and `C`, then you can compute the count of the three components independently and then multiply the three counts.  The LogicNG internal model counter e.g. makes use of this and computes strongly connected components before computing the count.

You can compute the strongly connected components in the following way:

```java
final Set<Set<Node<Variable>>> ccs =
        ConnectedComponentsComputation.compute(constraintGraph);
final List<List<Formula>> components =
        ConnectedComponentsComputation.splitFormulasByComponent(formulas, ccs);
```

Then the set of connected set of nodes `ccs` contains:

```
- Set1:
    - Node{content=A, neighbours:B,C,D}
    - Node{content=B, neighbours:A,C}
    - Node{content=C, neighbours:A,B}
    - Node{content=D, neighbours:A,E}
    - Node{content=E, neighbours:D}
- Set2:
    - Node{content=G, neighbours}
```

and the `components` are:

```
- [A | ~B | C, D | ~A, D + E = 1]
- [G]
```


## Hypergraphs

In a [hypergraph](https://en.wikipedia.org/wiki/Hypergraph) an edge can connect more than two nodes.  Hypergraph decomposition is an important method used in many algorithms to find a good variable ordering, e.g. for BDD or DNNF generation.  Since hypergraph decomposition is a very complex subject and there are dedicated libraries for that, LogicNG does not implement its own hypergraph decomposition.  However, the FORCE heuristic of [BDDs](../knowledge-compilation/bdd) uses an approximation of hypergraph decomposition on the hypergraph.

For this case, each edge in the hypergraph represents a single clause in the CNF (remember, the edge can connect more than two nodes).  Such a hypergraph of a CNF can be generated in the following way:

```java
List<Formula> formulas = Arrays.asList(f.parse("A | ~B | C"), f.parse("D | ~A"),
        f.parse("D | ~E"), f.parse("G"));
Hypergraph<Variable> hypergraph = HypergraphGenerator.fromCNF(formulas);
```

The result is then a hypergraph with four edges:

```
Hypergraph{
  ...
  edges=[
    HypergraphEdge{nodes=[
      HypergraphNode{content=A},
      HypergraphNode{content=B},
      HypergraphNode{content=C}
    ]},
    HypergraphEdge{nodes=[
      HypergraphNode{content=A},
      HypergraphNode{content=D}
    ]},
    HypergraphEdge{nodes=[
      HypergraphNode{content=D},
      HypergraphNode{content=E}
    ]},
    HypergraphEdge{nodes=[
      HypergraphNode{content=G}]}
    ]}
```
