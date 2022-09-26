---
title: High-Level Configuration
---

## Rule Set Validation

The high level rule set describes every buildable product configuration and therefore often is the base of all subsequent design, product development, production, sales, and after-sales processes. Its correctness is of uttermost importance. The library is used to *validate rule sets* for given criteria, e.g., are there features which are not buildable at any point in time, are there rules which contradict other rules, etc. Our algorithms are used to search for time gaps in rules, e.g., that a new rule supersedes an old one but there is a time gap between the two. The library is also used in interactive rule generation and maintenance processes and checks if a new or modified rule is consistent with the current rule set, if it can be simplified, or if it contains parts which contradict other rules.


## Projections and Simplifications of Rules

Typically, the high level rule set consists of thousands of rules over many product lines, countries, and releases. Hence, it is often useful to project these rules to arbitrary filter conditions. LogicNG can be used to perform these *projections*, or *simplify* the resulting rules, showing which rules are affected by the projection, and also *highlight the differences* within milliseconds.


## Enumerate Buildable Configurations

Another very important supported use case, used in many software systems, is to generate the buildable variance over a certain set of features: a maintainer for breaking systems, e.g., knows exactly which features influence the configuration of the breaking system and wants to know which combinations between these features are buildable with respect to the high level rule set. Very efficient model enumeration algorithms allow to *enumerate these buildable configurations*, even for millions of combinations within seconds.


## Optimization of Required Product Configurations

Also a very popular use case is to *optimize the number of required product configurations*. Thinking of the production of test vehicles, there are hundreds or thousands of requirements for physical test vehicles: For a desert test drive, e.g., someone needs a vehicle with air conditioning, while for a photo shoot a red vehicle in the luxury line is needed, and so on. The question now is: How many physical vehicles do you need to produce *at least* in order to satisfy *all* these constraints. Since test vehicles are very expensive, saving one or two vehicles can be lucrative. Previously, these processes were often performed by hand, taking days or even weeks. With the library's optimization algorithms, the corresponding computations can be performed in minutes, yielding guaranteed minima.

