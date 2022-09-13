---
title: Tutorial Chapter 4
---

Beatrice wants to transform the rules in her rule set into so-called `Propositions`. These are objects which contain the formula, and some additional information for the formula.  This can be very useful to add additional information to formulas which are not relevant for logical computations but for interpreting results of algorithms and mapping formulas to original domain objects.

In LogicNG, the abstract class `Proposition` is implemented by the `StandardProposition` and the `ExtendedProposition`.  The `StandardProposition` stores the formula and optionally a description, and the `ExtendedProposition` stores the formula and a freely defined _backpack_, which is an implementation of the interface `PropositionBackpack`.

For a bike rule set, Beatrice finds that every formula should be equipped with an ID and the date on which it was created. Thus, she generates the *backpack* which she is going to add to the formula in the following way:

```java
class MyBackpack implements PropositionBackpack {
    private final long id;
    private final LocalDate date;

    MyBackpack(long id, LocalDate date) {
        this.id = id;
        this.date = date;
    }
}
```

Next, she creates propositions for every formula in her rule set. For this, she has the following logic:

- IDs are integers, starting with 0.
- For the creation date, luckily, she recalls when she created the different formulas which constitute the rule set:
    - She first created formulas of type "exactly one" and "at most one", and that was on the 01.01.2020.
    - Then she created the other formulas, that was on the 01.06.2020.
    - The pro saddle `s4` only came out in 2021, in fact on the 01.01.2021. Thus, every formula containing `s4` could have only been created then.
    - The bike bell `b2` only came out recently, in fact on the 01.01.2022. Thus, every formula containing `b2` could have only been created then.

For example, the proposition for `f_equiv` (`frame <=> f1 | f2 | f3`), would be the following:

``` java
Proposition proposition = new ExtendedProposition<>(
        new MyBackpack(1, LocalDate.of(2020, 1, 1)),
        data.f_equiv
      );
```

The following method transforms the current formulas to the propositions extended with the additional information.

``` java
public static List<Proposition> getPropositions(
        BicycleShopData data,
        List<Formula> formulas
) {
    long id = 0;
    List<Proposition> propositions = new ArrayList<>();
    for (Formula formula : formulas) {
        LocalDate date;
        if (formula.variables().contains(data.s4)) {
            date = LocalDate.of(2021, 1, 1); // (1)!
        } else if (formula.variables().contains(data.b2)) {
            date = LocalDate.of(2022, 1, 1); // (2)!
        } else if (formula.type() == FType.PBC) {
            date = LocalDate.of(2020, 1, 1); // (3)!
        } else {
            date = LocalDate.of(2020, 6, 1); // (4)!
        }
        Proposition proposition = new ExtendedProposition<>(
                new MyBackpack(id, date), formula); // (5)!
        propositions.add(proposition);
        id++;
    }
    return propositions;
}
```

1. formulas for the pro saddle
2. formulas for the metal strip bike bell
3. exo and amo constraints
4. implications and exclusions
5. create a new proposition

The result is the list of propositions:

```java
ExtendedProposition{formula=frame <=> f1 | f2 | f3, backpack=MyBackpack{id=0, date=2020-06-01}}
ExtendedProposition{formula=f1 + f2 + f3 = 1, backpack=MyBackpack{id=1, date=2020-01-01}}
ExtendedProposition{formula=handlebar <=> h1 | h2 | h3 | h4 | h5, backpack=MyBackpack{id=2, date=2020-06-01}}
ExtendedProposition{formula=h1 + h2 + h3 + h4 + h5 = 1, backpack=MyBackpack{id=3, date=2020-01-01}}
ExtendedProposition{formula=saddle <=> s1 | s2 | s3 | s4, backpack=MyBackpack{id=4, date=2021-01-01}}
ExtendedProposition{formula=s1 + s2 + s3 + s4 = 1, backpack=MyBackpack{id=5, date=2021-01-01}}
ExtendedProposition{formula=frontWheel <=> wf24 | wf26 | wf27 | wf29 | wf32, backpack=MyBackpack{id=6, date=2020-06-01}}
ExtendedProposition{formula=wf24 + wf26 + wf27 + wf29 + wf32 = 1, backpack=MyBackpack{id=7, date=2020-01-01}}
ExtendedProposition{formula=backWheel <=> wb26 | wb27 | wb29 | wb32, backpack=MyBackpack{id=8, date=2020-06-01}}
ExtendedProposition{formula=wb26 + wb27 + wb29 + wb32 = 1, backpack=MyBackpack{id=9, date=2020-01-01}}
ExtendedProposition{formula=bell <=> b1 | b2 | b3, backpack=MyBackpack{id=10, date=2022-01-01}}
ExtendedProposition{formula=b1 + b2 + b3 <= 1, backpack=MyBackpack{id=11, date=2022-01-01}}
ExtendedProposition{formula=luggageRack, backpack=MyBackpack{id=12, date=2020-06-01}}
ExtendedProposition{formula=r1 + r2 + r3 <= 0, backpack=MyBackpack{id=13, date=2020-01-01}}
ExtendedProposition{formula=color <=> c1 | c2 | c4, backpack=MyBackpack{id=14, date=2020-06-01}}
ExtendedProposition{formula=c1 + c2 + c4 = 1, backpack=MyBackpack{id=15, date=2020-01-01}}
ExtendedProposition{formula=f1 => ~s1 & ~h3, backpack=MyBackpack{id=16, date=2020-06-01}}
ExtendedProposition{formula=$true, backpack=MyBackpack{id=17, date=2020-06-01}}
ExtendedProposition{formula=f3 => ~h5, backpack=MyBackpack{id=18, date=2020-06-01}}
ExtendedProposition{formula=h5 => ~bell, backpack=MyBackpack{id=19, date=2020-06-01}}
ExtendedProposition{formula=~b2 | f2 | f3, backpack=MyBackpack{id=20, date=2022-01-01}}
ExtendedProposition{formula=~wf24, backpack=MyBackpack{id=21, date=2020-06-01}}
ExtendedProposition{formula=wf26 <=> wb26, backpack=MyBackpack{id=22, date=2020-06-01}}
ExtendedProposition{formula=wf27 <=> wb27, backpack=MyBackpack{id=23, date=2020-06-01}}
ExtendedProposition{formula=wf29 <=> wb29, backpack=MyBackpack{id=24, date=2020-06-01}}
ExtendedProposition{formula=wf32 <=> wb32, backpack=MyBackpack{id=25, date=2020-06-01}}
ExtendedProposition{formula=frame => wf26 | wf27 | wf29 | wf32, backpack=MyBackpack{id=26, date=2020-06-01}}
ExtendedProposition{formula=wf32 => f3, backpack=MyBackpack{id=27, date=2020-06-01}}
```

Beatrice is pleased.  It is more intuitive to work and it will be way easier to understand conflicts arising in her rule set with these propositions than with the original formulas.  For more information about Propositions, check out [the relevant chapter](../../documentation/propositions) in the documentation.

If you're interested in what Beatrice is doing with those propositions, check out the next chapter on [SAT solving](../../documentation/solvers/sat-solving).

