---
title: Ordering Process
---

## Interactive Configuration

LogicNG is used in configurators for both in-house product development, and customer touchpoints. Besides the standard functions like supporting an *interactive configuration*, where in each step the remaining selectable features are highlighted, it supports some advanced use cases.


## Explanation of Conflicts

In the case of a conflict, one can easily *explain why a certain configuration is not buildable* by employing one of many available explanation algorithms like, e.g., finding the shortest explanation for the conflict. During the product development process, this can often save many hours of analysing the rule sets manually.


## Re-Configuration

However, if the user explicitly wants to select a feature which contradicts the current configuration, you can *compute possible re-configurations* for user-provided criteria. As such, it is possible to compute the, e.g., cheapest re-configuration or the one with fewest changes to the current order. This is also used in automated processes like validating all currently active orderings every night, computing which vehicles are not buildable any longer because of changes in the rule sets, and proposing possible re-configurations to the user.

