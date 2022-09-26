---
title: Mass Customization
hide:
  - toc
---

When considering interesting use cases for automated reasoning in product configuration, first we have to look at two aspects: product variance, i.e., how many different product configurations are buildable, and product volume, i.e., how many of these configurations are produced every day.  The following figure illustrates this situation.

![Product variance vs. product volume](../../assets/graphics/whitepaper/mass-customization-dark.png#only-dark){ width=400 }
![Product variance vs. product volume](../../assets/graphics/whitepaper/mass-customization-light.png#only-light){ width=400 }

In today's world there are highly customizable products like airplanes or cruise ships. These products possess high product variance, yet they are not mass-produced (upper left quadrant). The configuration process involves many months of planning and coordination.  On the other hand there are products, which are produced in great numbers, yet having only a very limited variance of only dozens or hundreds of different models (lower right quadrant), with a typical example being, e.g., microprocessors. In these cases, all possible configurations can be verified beforehand and there is almost no configuration process during production.

In the upper right quadrant we find products having both a high variance *and* are produced in large numbers. One such class of products are premium cars: A Mercedes E-Series in 2012, e.g., had over 10^102^ different buildable configurations and yet thousands of vehicles are produced every day. This is called *Mass Customization*, a term coined by Stan Davis in 1987 and defined by Mitchell M. Tseng and Jianxin Jiao in 1996 as

!!! quote "Mass Customization"
    producing goods and services to meet individual customer's needs with near mass production efficiency

In mass customization, very interesting but hard configuration problems arise: It is not possible to verify each single buildable configuration by hand or individually, even when this process is automated. Additionally, the mass production aspect allows no room for misconfigurations since a car leaves the moving assembly line every one to two minutes. Line stoppage hence can cost hundreds of thousands of Euros.

Configuration problems do exist in all four quadrants, and LogicNG can support the whole process from product development over production to sales here.  But the upper right quadrant is where it really shines.

