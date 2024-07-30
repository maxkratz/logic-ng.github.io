---
title: The Formula Factory
---

The formula factory is the central concept of LogicNG and is always required when working with LogicNG.  A formula factory is an object consisting of two major components:

A *factory*, which creates formulas, and a *container*, which stores created formulas.

The *container function* is 'smart': A formula factory guarantees that syntactically equivalent formulas are created only once.  This mechanism also extends to variants of the formula in terms of associativity and commutativity. Therefore, if the user creates formulas for

- `A & B`
- `B & A`
- `(B & A)`

all of them are represented by only one formula in memory.  This approach is only possible, because formulas in LogicNG are immutable data structures.  So once created, a formula can never be altered again.

In order to use the fact that formula factories avoid unnecessary formula creations, it is generally recommended to use only *one* formula factory for a certain task.

During this documentation, we consider the formulas `f1`, `f2` and `f3` and use the following notion of a formula factory:

``` java
FormulaFactory f = new FormulaFactory();
```


## Creating Formulas with a Formula Factory

Formulas in LogicNG cannot be constructed directly but must be created by an instance of a formula factory. This factory ensures the following five invariants:

1. A constant (`true` or `false`) cannot be an operand to any other formula, i.e. constants are automatically removed.
2. The operand of a conjunction may not be another conjunction; the same applies to disjunctions. These cases are merged in one big conjunction/disjunction.
3. The operand of a negation may only be a binary operator, an n-ary operator or a pseudo-Boolean constraint. For other operands the respective simplifications are performed.
4. An n-ary operator has unique operands.  Duplicate operands in a disjunction or conjunction are filtered.
5. Every positive literal is guaranteed to be an instance of class `Variable`.

In the default configuration there is also a sixth invariant, which however can be deactivated by a configuration flag (see below):

6. Inverse operands of an n-ary operator are simplified, this means `f1 & ~f1` is parsed to `$false`, and `f1 | ~f1` is parsed to `$true`.

Furthermore, some further simplifications are performed when parsing or creating formulas, such as `A <=> A` is equivalent to `$true`, or `A <=> ~A` is equivalent to `$false`.

While being rather easy to realize, these restrictions simplify reasoning about the structure of a formula and thus significantly reduce the number of corner cases algorithms have to face.

Together with the smart container function presented in the last section, the example can be extended.  Not only are formulas `A & B`, `B & A`, or `(B & A)` represented by only one formula object in memory, but also variants like

- `A & A & B`
- `B & A & $true`
- `(B & A) & B & ($false | A) & ($true | C)`

There are two ways to create formulas using a formula factory:

Firstly, one can parse a formula from a string: E.g. `#!java Formula created = p.parse("A & B");`.

Secondly, one can create a formula of a certain type with the methods for formula creation in the formula factory. An overview about how to create those formulas is here:

| Object      | Factory Method                 | Syntax                              |
|-------------|--------------------------------|-------------------------------------|
| True        | `#!java f.verum()`             | `$true`                             |
| False       | `#!java f.falsum()`            | `$false`                            |
| Variable    | `#!java f.variable("A")`       | `A`                                 |
| Literal     | `#!java f.literal("A", false)` | `~A`                                |
| Negation    | `#!java f.not(f1)`             | `~f1`                               |
| Implication | `#!java f.implication(f1, f2)` | `f1 => f2`                          |
| Equivalence | `#!java f.equivalence(f1, f2)` | `f1 <=> f2`                         |
| Conjunction | `#!java f.and(f1, f2, f3)`     | `f1 & f2 & f3`                      |
| Disjunction | `#!java f.or(f1, f2, f3)`      | <code>f1 &vert; f2 &vert; f3</code> |

The order of operands in the resulting formula does not follow an overall ordering but depends on the formulas created first. If a formula `B & A` was created, the order of the operands will be always `B, A`. Thus, when creating another formula `A & B` it will result in `B & A`. If another formula `A & B & C` is created, the operands occur in the order `A, B, C`, since `A & B & C` was the first created formula.

It is also possible to write Pseudo-Boolean constraints, especially cardinality constraints like e.g. `A + B + C <= 1`:

``` java
f.cc(CType.LE, 1, Arrays.asList(f.variable("A"), f.variable("B"), f.variable("C")))
```

which means from variables `A`, `B`, `C` can be at most one variable assigned to true.  An example for a Pseudo-Boolean constraint is `A + 2* ~B - 3*C = 2`:

``` java
List<Literal> lits =
        Arrays.asList(f.variable("A"), f.literal("B", false), f.variable("C"));
List<Integer> coeffs = Arrays.asList(1, 2, -3);
f.pbc(CType.EQ, 2, lits, coeffs);
```

Beside the mentioned factory methods there are many convenience methods to create formulas. Examples are:

- `constant(boolean value)` which creates a `$true` or `$false` constant depending on the given Boolean value
- `cnf(Formula... clauses)` creating a conjunctive normal form (CNF) for the given clauses
- `clause(Literal... literals)` creating a clause for the given literals
- `variables(Collection<String> names)` and `variables(String... names)` creating a set of variables from the given names (since [:octicons-tag-24: 2.4.0](https://github.com/logic-ng/LogicNG/releases/tag/v2.4.0))

For information about properties of formulas check out the next chapter [Formulas](../formulas).

When creating a formula factory, the factory does not contain any formulas except from the constants `true` and `false`, which are always generated when creating a formula factory.

Another option for using a formula within a formula factory without creating it, is to import a formula from a different formula factory (see further below).


## Configuring the Formula Factory

The user can configure some settings of the formula factory.  Those are given over via the `FormulaFactoryConfiguration` (see [FormulaFactoryConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/formulas/FormulaFactoryConfig.java)), with which the formula factory can be generated. If the formula factory is generated with no parameter, then the default formula factory configuration is chosen. The parameters that can be configured are:

- **The formula merge strategy**:
  The formula merge strategy is concerned with combining formulas of different formula factories. A formula from a different factory can only be used in the current factory after importing it.  In order to avoid bugs by using formulas from different factories without importing it, the default behaviour of the formula factory is to panic and throw an exception.  This however can be changed to the import strategy - then a formula from a different factory is automatically imported.  The respective configuration flags are `PANIC`, and `IMPORT`.
- **A flag, whether complementary operands shall be simplified**. This is the flag whether complementary operands within a conjunction or disjunction should be simplified. For example, shall a formula like `A & ~A`, `(A | ~A) & C`, or `(A & B) | ~(A & B)` be simplified or not.  If the flag is set to `true`, these formulas will be simplified during creation to `$false`, `C`, and `$true` respectively.  If the flag is set to `false` such trivial tautologies and contradictions are not simplified but kept.
- The **string representation**:  This is the default string representation of formulas when using Java's standard `toString()` method.  In LogicNG there are three default implementations:
  1. [UTF8StringRepresentation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/formulas/printer/UTF8StringRepresentation.java) for a UTF-8 representation with nice symbols for the operands and sub-script for indexed variables
  2. [LatexStringRepresentation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/formulas/printer/LatexStringRepresentation.java) for a LaTeX notation of the formulas
  3. [DefaultStringRepresentation](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/formulas/printer/DefaultStringRepresentation.java), which is the default string representation and outputs the formulas in the LogicNG syntax which can be parsed again by the LogicNG parsers.

Consider the formula `A & B | C`.  The UTF-8 representation gives the formula out as `A ∧ B ∨ C` and the Latex style representation as `A \land B \lor C`. The default representation is `A & B | C`.

- the **name**: An optional name for the formula factory (which can be useful when using more than one formula factory at the time).  The default is the empty string `""`. Note that the name will appear in all auxiliary variables (constructed variables for CNF, PB and CC encodings) and will *not* be changed when imported to other factories.

The parameters of the default configuration is summarized here:

!!! note "Default Configuration"

    When constructing a formula factory without configuration, the default configuration `FormulaFactoryConfig` is constructed. The settings are:

    - name: `""`
    - formula merge strategy: `PANIC`
    - string representation: `DefaultStringRepresentation`
    - simplify complementary operands: `true`
    - ccPrefix: `@RESERVED_CC_`, pbPrefix: `@RESERVED_PB_`, cnfPrefix: `@RESERVED_CNF_`

In order to create a formula factory with different parameters, one can use the `FormulaFactoryConfig` builder as follows:

``` java
FormulaFactory f = new FormulaFactory(
  FormulaFactoryConfig.builder()
    .formulaMergeStrategy(FormulaFactoryConfig.FormulaMergeStrategy.IMPORT)
    .stringRepresentation(UTF8StringRepresentation::new)
    .name("New factory")
  .build());
```


## Configuring Further Aspects of LogicNG in the Formula Factory

A formula factory can also hold configurations for different algorithms in LogicNG.  These algorithms can be configured when using or by putting an initial configuration in the formula factory.

E.g. currently the following configurations can be put on the formula factory:

- **CNF configuration** (see [CNFConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/transformations/cnf/CNFConfig.java)), defining the encoding procedure used to generate a CNF of a formula, e.g. when `#!java f1.cnf()` is called.
- **Cardinality constraint configuration** (see [CCConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/cardinalityconstraints/CCConfig.java)), defining the encoding procedure used when a cardinality constraints is encoded, e.g. when `#!java f1.cc()` is called.
- **Pseudo-Boolean configuration** (see [PBConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/pseudobooleans/PBConfig.java)),  defining the encoding procedure used when a Pseudo-Boolean constraint is encoded, e.g. when `#!java f1.pb()` is called.
- **Formula randomizer** (see [FormulaRandomizerConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/util/FormulaRandomizerConfig.java)), a configuration for random formula generation, which can be a useful tool for testing.
- **MUS** (see [MUSConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/explanations/mus/MUSConfig.java)), the configuration object for the MUS (minimal unsatisfiable set) generation.

Further, one can specify parameters for the different solvers:

- **MiniSAT solver** (see [MiniSatConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/sat/MiniSatConfig.java))
- **MaxSAT solver** (see [MaxSATConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/maxsat/algorithms/MaxSATConfig.java))
- **Glucose SAT solver** (see [GlucoseConfig](https://github.com/logic-ng/LogicNG/blob/master/src/main/java/org/logicng/solvers/sat/GlucoseConfig.java))

More information about these can be found in the [chapter on solvers](../solvers).

Using `#!java f.putConfiguration(config)`, for some configuration `config`, one can put a new configuration into the configuration database. Existing configurations for this type will be overwritten. `#!java f.configurationFor(configType)` (for some configuration type `configType`) returns the configuration of a specific configuration type.


!!! example "Example: Changing a configuration of the formula factory"

    Consider the cardinality constraint

    ```
    A + B + C <= 1
    ```

    Using the default cardinality constraint configuration, the cardinality constraint is encoded to:

    ```
    (~A | ~B) & (~A | ~C) & (~B | ~C)
    ```

    However, if you'd like to encode the cardinality constraint differently, you may choose to set a different algorithm for the encoding of cardinality constraints in the formula factory.  For example, changing the formula factory's default algorithm for encoding at-most-one constraints with the LADDER encoding, one could configure the formula factory in the following way:

    ``` java
    CCConfig config = CCConfig.builder().amoEncoding(CCConfig.AMO_ENCODER.LADDER).build();
    f.putConfiguration(config);
    ```

    The resulting cardinality constraint encoding is then, for the auxiliary variables `x1` and `x2`:
    ```
    (~A | x1) & (~B | x2) & (~x1 | x2) & (~B | ~x1) & (~C | ~x1)
    ```


## More Methods on Formula Factories

Some more useful methods on formula factories are:

- Remove all formulas from the factories cache with `#!java f.clear()`: Use this method if you want to hold on to the "settings" of the formula factory, but get rid of all formulas created so far
- Import a formula from another formula factory into this factory and return it: `#!java f.importFormula(f1)`. If the current factory of the formula is already this formula factory (`f`), the same instance will be returned
- Check whether a variable is a generated variable with `#!java f.isGeneratedVariable(v)` for some variable `v`. A variable is considered to be a generated variable (or auxiliary variable) if it was generated during a CNF, PG or cardinality constraints encoding
- Get some statistics about the formula factory: `#!java f.statistics()` returns a `FormulaFactoryStatistics` which includes some relevant information about the factory, for example:
    - The number of formulas the formula factory holds in total
    - The number of formulas the formula factory holds for each formula type


## Caching Formulas in Practice and the Extended Formula Factory

While caching formulas in the formula factory in most cases greatly improves heap space usage *and* performance, there are some cases where this behaviour could be undesirable. An example is an application which holds a large base rule set and there are many requests for other large formulas, which are checked against this base rule set. If the application runs in a server environment and uses one formula factory, this formula factory would potentially grow with every request. Usually this does not happen too often in practice, since in our experience, also the base rule set is something which is not static and is created with each request. However, for the described scenario, the ever-caching formula factory would be a suboptimal approach.

Therefore, there is an `ExtendedFormulaFactory` which extends the normal formula factory by the ability of saving and loading the state of the factory, therefore temporarily adding formulas to the factory and then removing them again. Thus, in the described scenario, one could add the base rule set and save the formula factory state. Then we add some large other formulas, perform our computations, and when we are done, we load the saved state.  By these operations, all internal data structures of the formula factory are shrunk to their length as stored in the saved state and therefore the formulas added *after* the state save are removed.  Also, for internal consistency reasons, the caches of *all* formulas are cleared.

The following code example shows the usage of this feature

```java
ExtendedFormulaFactory f = new ExtendedFormulaFactory();
PropositionalParser p = new PropositionalParser(f);

// read and work with a large base formula

FormulaFactoryState state = f.save(); // (1)!
p.parse("..."); // (2)!
f.load(state); // (3)!
```

1. saves the current state
2. work with large additional formulas
3. reset the factory to the state after adding the base formula


!!! danger "Formula Factories and Thread Safety"

    A formula factory has many internal data structures which are not thread safe.  Therefore, one must *never* use a single formula factory object in a multi-threaded fashion.  Different threads accessing the same formula factory at the same time can lead to concurrency exceptions, undefined behaviour, and incorrect computations.  In a multi-threaded application, each thread should have its own formula factory on which it operates.

