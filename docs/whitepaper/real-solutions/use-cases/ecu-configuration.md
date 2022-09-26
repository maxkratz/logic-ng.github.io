---
title: ECU Configuration
---

!!! summary "Electronic Control Units"

    Electronic control units (ECU) play an ever growing part in today's vehicles. A modern vehicle has over one hundred ECUs. The validation of both the correct usage of the ECUs as well as their configuration is as important as the validation of the BOM. Since the ECUs of vehicles often are flashed on the moving assembly line with only a short time to flash the right configuration, concise and simplified formulas are required. Last but not least, not only the ECUs are highly configurable, but also the cable harnesses connecting them throughout the vehicle.


## Verification of the ECU Parameters

Similar to the validation of BOMs, LogicNG is used for the *verification of the ECU parameter configuration* for all possible buildable vehicles. In contrast to the BOM, which is usually documented for a vehicle series, ECUs are often documented for all series and product lines. This leads to very large and complex formulas in their configuration. Also some ECUs have thousands of parameters with tens of thousands of different possible parameter values. Once again the speed of our algorithms and solver engines is very important, enabling interactive verification of parameters over all product lines.


## Simplification of Flash Conditions

The formula simplification algorithms provided in the library are used to *simplify the flash conditions* for ECU parameters, allowing for a fast flash process on the assembly line. Formulas are not only simplified mathematically, but also with additional domain knowledge, providing even shorter representations.


## Optimization of Module Building in Cable Harnesses

The library is also used to *optimize module building in cable harnesses*. Today's cable harnesses consist of many modules (if they are not customer-specific). But the questions are: how many modules should you design and produce?  Are there modules which are always required? Modules which force other modules?  Or modules which are not required anymore? By taking take-rates into account one can also optimize the module building process by producing specific modules for popular feature combinations, enabling cost reductions.
