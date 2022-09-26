---
title: Homologation Requirements
---

With the introduction of the Worldwide harmonized Light vehicles Test Procedure (WLTP) in the automotive industry in 2019, the homologation process and the computation of vehicle-individual CO~2~ and consumption values got much more complex. LogicNG has been used from day one to help in both use cases.


## Computing Best and Worst Vehicle Configurations

*Computing the best and worst vehicles* with respect to their energy consumption is necessary for the homologation process. Since this is a linear programming problem, a linear optimizer like CPLEX or Gurobi is the best tool for the job. However, the state of the art tools could not yield the performance required to perform tens of thousands of computations a day. Our algorithms have been used to simplify all involved rules and pre-process many of the data in order to improve the performance of the linear optimizers. This helped to speed up the computation time from some minutes to a few seconds.


## Computing individual CO~2~ and Consumption Values

*Computing the individual CO~2~ and consumption values* for a vehicle requires the exact weight, aerodynamic values and rolling resistance of that particular vehicle. The library is used to calculate those physical data by efficiently computing the BOM explosion. Since these computations are integrated in the online configurators, speed and response time is extremely important. With our algorithms, the computation of a single vehicle's individual values requires less than one millisecond, and such operations are being performed millions of times a day in production systems.

