---
title: Towards a Software-Defined World
---

The above use cases have a strong focus on the development, production and sales processes of the hardware of the product. Since the rise of the *Software Defined Vehicle*, software plays an equally important part in all phases of the life cycle of a vehicle.

## Management of Software Packages

One of the most important aspects is the *management of software packages*: which versions of a software or library are compatible with which other versions, which packages have dependencies to or restrictions with other packages. This is an area where SAT solvers are traditionally used to resolve package update problems. Linux package managers or the Eclipse plugin manager, e.g., use SAT solvers to resolve their packages. A very small (< 300 lines of codes) proof of concept package solver with different optimization strategies based on LogicNG was released on [GitHub](https://github.com/booleworks/package-solving-poc):


## Release and Update Validation

Another use case is to *validate a whole software release* including ECUs, libraries, and software packages. But this is no static process: over-the-air (OTA) updates make it necessary to constantly verify all software packages and updates against regulations like, e.g., the UNECE R 156. A change in a single rule in any of the rule sets (be it high level, BOM, ECU, or software) can for example have an effect on the homologation of a vehicle. A software update to a motor ECU, e.g., could change its emission values and invalidate a homologation. Within the SofDCar project, BooleWorks is working with its partners from industry and research to develop holistic data models for all relevant data and rule sets, which can then be validated both for all planned but also all already produced vehicles, perpetually and automatically.


## Computation of Selectable Software and Services

A use case which is already present today, but which will play an even more important role in the future is the whole aspect of *after-sales software and service add-ons*. While the customer today can buy some new services online, in the future, whole vehicle features will be available on-demand. Thus, the *computation of selectable services and software features* will play an important role, as well as *re-configuration algorithms*, enabling customers to buy a feature which is currently not possible in their vehicle.

