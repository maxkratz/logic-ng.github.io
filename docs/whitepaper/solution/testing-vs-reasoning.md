---
title: Manual Testing vs. Automated Reasoning
hide:
  - toc
---

Traditionally, certain properties of rule sets were verified by manual tests. But with up to 10^100^ buildable configurations, each manual testing approach will yield a test coverage of de facto 0% - even when testing billions of different configurations manually. The problem is that if your manual tests do not find an invalid configuration, you never know whether there *is no* invalid configuration or you just *did not find it*. The left graphic in the next figure illustrates the situation.  The green space is the set of all buildable variants. The large red area are invalid configurations and the white dots are your tested configurations.  In this case your tests do not hit an invalid configuration.

![Testing vs. Reasoning](../../assets/graphics/whitepaper/testing-vs-reasoning.png){ width=700 }

However, a product formula as introduced in the last section describes *every* buildable variant, implicitly.  When using a solver or knowledge compilation engine we either find an invalid configuration or we get a *mathematical proof* that there can be none.  There is no uncertainty like in manual testing.  The right graphic in the figure illustrates this: all buildable vehicles are implicitly tested and therefore we quickly find an invalid configuration.

The advantage of always considering the complete solution space translates also to other algorithms.  When searching an *optimal product configuration*, e.g., the heaviest configuration, the solution is a mathematical global optimum, not some approximation.  When computing *all* possible combinations between a set of features, the result indeed is each mathematically possible configuration, not just a sample of it.

