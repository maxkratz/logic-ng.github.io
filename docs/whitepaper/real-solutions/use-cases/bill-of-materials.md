---
title: Bill of Materials
---

!!! summary "Bill of Materials"

    The bill of material (BOM) maps high level configurations to physical parts of the product. E.g., for a vehicle, a high level feature like "Sport Steering Wheel" gets resolved into many parts like the steering wheel itself but also encompasses cables, mounts, or screws. Its validation is very important since a product-individual 100% BOM is computed for each configuration which is produced. This is called a "BOM explosion". Errors in the BOM can yield wrong parts at the moving assembly line and, in the worst case, causes line stoppage which potentially costs hundreds of thousands of Euro.


## BOM Validation

One of the first use cases of LogicNG was to *validate the BOM* for correctness criteria, e.g., that each buildable configuration gets exactly one steering wheel. These computations are always performed over the whole product variance, not only for certain vehicles. Therefore errors can be spotted *before* a vehicle is ordered or produced, because *every possible vehicle* is implicitly validated. It is also used, e.g., for computing if there are *parts which are not buildable* in a certain plant for a certain time period. This is key for advanced supply chain management.


## BOM Explosion

Today, our algorithms are also often employed for the traditional *BOM explosion* for a given product. Although this is a standard functionality of every BOM system, the achieved speed is often far superior to these systems. In the automotive context, our algorithms can resolve > 100,000 BOM positions per second on a standard laptop, single-threaded on one processor core.


## BOM Visualization

Thanks to the library's extensible BDD implementation, it is also used to *visualize BOM positions* and allows a graphical maintenance of parts at a position. It can first visualize the position as a graph, allowing the maintainer to add and remove parts or to draw and re-route edges, and then computes the new resulting constraints for the BOM system.

