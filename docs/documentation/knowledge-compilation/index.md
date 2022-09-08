---
title: Knowledge Compilation
---

In order to compute queries on a formula, such as "is the formula satisfiable?", "how many models does the formula have?", "does the formula imply another formula" etc., one can take essentially one of two approaches:

1. Use tools like a SAT solver, which compute the answers to such queries on-line on the original input formula (or normal forms which can be efficiently computed).  Usually this on-line computation step is the computation-intensive part, but the off-line part of filling the SAT solver with the formula is easy.

2. Off-line compile the original format into a more distinct format, e.g. a DNF, a BDD, or a DNNF and then answer the queries on this compiled format.  In this case, the off-line compile step is the computation-intensive part, but answering the queries on-line is efficient.

The second approach is called **knowledge compilation**.

As a very simple example consider that we want to know two properties of a formula: 1) is it satisfiable, and 2) if so, how many models does it have.

In order to answer these two questions with approach 1 you could fill a SAT solver with the formula, compute its satisfiability and then enumerate all models. These computations are hard (in this case in NP and #P).  For approach 2 you could compile the formula in a very simple knowledge compilation format, e.g. a canonical DNF.  Computing this canonical DNF is the hard part, but if you could compute it, the two questions would be trivial to answer: The formula is satisfiable, if the canonical DNF has at least one model, and the number of models are exactly the number of min-terms in the DNF.

But of course it is often not feasible to compute the canonical DNF of a formula, therefore more distinct knowledge compilation formats have been researched over the years.  There are many knowledge compilation forms. Check out [A Knowledge Compilation Map by Darwiche and Marquis](https://arxiv.org/pdf/1106.1819.pdf) to get an overview of the different knowledge compilation forms and how they perform in the distinct properties.

Generally, there are three aspects when considering different knowledge compilation forms:

- the succinctness of the compiled format
- the class of queries that can be answered in polynomial time on the compiled form
- the class of transformations that can be applied in polynomial time on the compiled form

In LogicNG, two advanced knowledge compilation forms are implemented:

1. Binary Decision Diagrams (BDD)
2. Decomposable Negation Normal Forms (DNNF)

BDDs are well-studied and have been presented by Randal E. Bryant in 1986. DNNFs, however, among other knowledge compilation formats, have been developed more recently by Darwiche. The next [chapter](bdd) in this documentary is on BDDs, the [succeeding one](dnnf) on DNNFs.

For more information about knowledge compilation forms, and BDDs and DNNFs, check out chapter 2.3. in
[New Formal Methods for Automotive Configuration by Zengler](https://publikationen.uni-tuebingen.de/xmlui/bitstream/handle/10900/57198/dissertation.pdf?sequence=1&isAllowed=y).


