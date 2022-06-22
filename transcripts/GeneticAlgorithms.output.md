# Prerequisites

To experiment with this code, you can use the
UniOpt codebase from the UCM:


```ucm
.> pull https://github.com/bbarker/uniopt

  Nothing changed as a result of the merge.

  😶
  
  the current namespace was already up-to-date with
  https://github.com/bbarker/uniopt.

```
The code is written as a
[Unison transcript](https://www.unison-lang.org/learn/tooling/transcripts/),
and is evaluated based on the most recent version of `UniOpt` stored on GitHub.

# Introduction

## Optimization

## Algorithms requiring Optimization

## Types of Optimization algorithms

This is not a formal or complete classification!

### Genetic Algorithms

- The most common form of evolutionary algorithm (EA).
- A sequence of "base pairs" (e.g. numbers) represent a
solution to the problem.
- The "DNA" is genetically manipulated.
- GAs are commonly used for optimization, compared to some other EAs.
- ***Requires many function evaluations, which may be expensive***.
- **A distributed variant would be ideal in this case**


## An Example Problem: The Knapsack Problem

- You are going on a trip.
- You can only bring items that you can fit into a knapsack.
- The knapsack has a weight limit.
- Which items do you take?


[![Depiction of the Knapsack Problem](/media/Knapsack.svg)](https://commons.wikimedia.org/wiki/File:Knapsack.svg)

- There are many approaches to solve this.
- Some specialized approaches that guarantee a global optimum.
- GAs are much broader in scope.
- Regardless, we can use the Knapsack problem as an example.

First, we model an `Item`, which has a weight and a value:

```ucm
.uniopt.evo.genetic.ex.knapsack> view Item

  unique type Item = { weight : Float, value : Float }

```
- We need a way to keep track of all items under consideration.
knapsack.
- We can use a `Map`, as well as an inverted `Map`.

```ucm
.uniopt.evo.genetic.ex.knapsack> view PossibleItems

  unique type PossibleItems = PossibleItems (Map Item Nat)

.uniopt.evo.genetic.ex.knapsack> view PossibleItemsRev

  unique type PossibleItemsRev = PossibleItemsRev (Map Nat Item)

```
The problem description includes:
  - The two maps
  - The weight limit

We also have a constructor to help us create `PossibleItemsRev`:

```ucm
.uniopt.evo.genetic.ex.knapsack> view KnapsackProblem

  structural type KnapsackProblem
    = { items : PossibleItems,
        itemsRev : PossibleItemsRev,
        capacity : Float }

.uniopt.evo.genetic.ex.knapsack> view makeKnapsackProblem

  makeKnapsackProblem :
    PossibleItems -> Float -> KnapsackProblem
  makeKnapsackProblem items maxWeight =
    KnapsackProblem items (revItems items) maxWeight

```
Let's see if we have any existing test data sets by checking the `dependents`
of `makeKnapsackProblem`:

```ucm
.uniopt.evo.genetic.ex.knapsack> dependents makeKnapsackProblem

  Dependents of #gsr3tq15pk:
  
       Reference   Name
    1. #inubj94so9 testProblem10items

```
We do! Let's see how it is defined:

```ucm
.uniopt.evo.genetic.ex.knapsack> view testProblem10items

  testProblem10items : KnapsackProblem
  testProblem10items =
    makeKnapsackProblem
      (PossibleItems Map.empty
        |> addItem (Item 4.06 9.89)
        |> addItem (Item 81.68 5.0)
        |> addItem (Item 65.16 1.18)
        |> addItem (Item 96.47 8.49)
        |> addItem (Item 7.2 1.31)
        |> addItem (Item 58.53 9.21)
        |> addItem (Item 29.47 7.66)
        |> addItem (Item 40.3 0.86)
        |> addItem (Item 53.67 8.15)
        |> addItem (Item 82.96 0.31))
      200.0

```
# Implementation of a simple Genetic Algorithm

This GA is simple enough that it was implemented from the ground up.


We start with some basic datatypes and functions.

## Base Pairs

```ucm
.uniopt.evo.genetic> view BasePair

  structural type BasePair = Numeric Float | Binary Boolean

```
- In the future, `BasePair` may expand to include other types of data.
- We could potentially use a typeclass once Unison supports them.
- In the example below, we only need binary data.

Actual genetic sequences would be base 4 instead of base 2, but for our purposes,
binary (base 2) is more convenient.

- In the Knapsack example:
  -  a `0` to denote the absence
of an item
  - a `1` the presence of an item


## Fitness

- Pair evaluated fitnesses with their corresponding
solutions
  - we don't need to re-evaluate
the fitness function unnecessarily:

```ucm
.uniopt.evo.genetic> view EntityFitness

  unique type EntityFitness
    = { chromosome : [BasePair], fitness : Float }

.uniopt.evo.genetic> view makeFitnessEvaluator

  makeFitnessEvaluator :
    ([BasePair] ->{g} Float) -> [BasePair] ->{g} EntityFitness
  makeFitnessEvaluator fitnessFn chromosome =
    EntityFitness chromosome (fitnessFn chromosome)

```
## Going between the Problem Domain and the GA Model

First, we need a sensible fitness function:

```ucm
.uniopt.evo.genetic.ex.knapsack> view knapsackFitnessFn

  knapsackFitnessFn : KnapsackProblem -> [BasePair] -> Float
  knapsackFitnessFn problem chromosome =
    use Float >
    use List map
    use util sum
    itemsInChrom =
      chromosomeToItems (itemsRev problem) chromosome
    itemsWeight = map Item.weight itemsInChrom |> sum
    itemsValue = map Item.value itemsInChrom |> sum
    if itemsWeight > capacity problem then 0.0 else itemsValue

```
- The fitness is the sum of item values
- ... Unless the sum of item weights exceeds the limit, then it is 0


After generating and evolving our chromosomes, we need a way
to translate them back into the problem space:

```ucm
.uniopt.evo.genetic.ex.knapsack> view chromosomeToItems

  chromosomeToItems : PossibleItemsRev -> [BasePair] -> [Item]
  chromosomeToItems itemsRev chromosome =
    use List map
    bpsWithIx =
      use Nat +
      List.zip
        chromosome (List.range 1 (1 + List.size chromosome))
    bpIsOn : (BasePair, Nat) -> Optional Nat
    bpIsOn bpix =
      bp = at1 bpix
      ix = at2 bpix
      match bp with
        Binary b -> if b then Some ix else None
        _        -> None
    onIxs = map bpIsOn bpsWithIx |> somes
    let
      (PossibleItemsRev rItemMap) = itemsRev
      map (bp -> Map.get bp rItemMap) onIxs |> somes

.uniopt.evo.genetic.ex.knapsack> view PossibleItemsRev

  unique type PossibleItemsRev = PossibleItemsRev (Map Nat Item)

```
We have a few use cases:

```ucm
.uniopt.evo.genetic.ex.knapsack> dependents chromosomeToItems

  Dependents of #69ihqud12r:
  
       Reference   Name
    1. #g0ocp1iles knapsackFitnessFn
    2. #rvkno0tq8o prettyPrintPopBase
    3. #vj5isq0k81 testChromToItem

```
## Changing the DNA (Exploring the Solution Space)

## Mutation

- During each generation, each "organism" (chromosome):
  - Will mutate zero or more of its basepairs
      - To keep things simple, we will just perform one mutation
      - Later we will specify the rate of mutation

```ucm
.uniopt.evo.genetic> view mutate

  mutate : [BasePair] ->{Random} [BasePair]
  mutate chromosome =
    chromLength = List.size chromosome
    randomIndex = Random.natIn 0 chromLength
    chromOpt = modifyAt randomIndex mutateBasePair chromosome
    match chromOpt with
      Some chrom -> chrom
      None       -> chromosome

```
We also need to implement `mutateBasePair`, which handles mutation for
each component type of `BasePair`:


```ucm
.uniopt.evo.genetic> view mutateBasePair

  mutateBasePair : BasePair ->{Random} BasePair
  mutateBasePair = cases
    Numeric _ -> Numeric !Random.float
    Binary b  -> Binary (not b)

```
## Recombination and Crossover

- Recombination is the process of exchanging genetic material (DNA) between
different organisms.
- We can think about this in terms of a specific type of recombination: chromosomal crosssover.

Chromosomal Crossover            |  (depiction by Thomas Hunt Morgan)
:-------------------------:|:-------------------------:
[![Depiction of chromosomal crossover, by Thomas Hunt Morgan](/media/Morgan_crossover_1.jpg)](a "Image credit: Wikipedia")  |  [![Thomas Hunt Morgan](/media/Thomas_Hunt_Morgan.jpg)](a "Image credit: Wikipedia")


- Natural chromosomal crossover is:
  - bounded by physical limitations
  - results in genetic linkage (T.H. Morgan)
- Instead we can allow crossing over at every base pair.

```ucm
.uniopt.evo.genetic> view crossoverUniform

  crossoverUniform :
    [BasePair] -> [BasePair] ->{Random} [BasePair]
  crossoverUniform chrom1 chrom2 =
    List.map
      (bpbp -> (if !randomBool then at1 bpbp else at2 bpbp))
      (List.zip chrom1 chrom2)

```
## Setting Rates for Mutation and Crossover


```ucm
.uniopt.evo.genetic> view crossAndMutDefault

  crossAndMutDefault :
    Float
    -> Float
    -> ([BasePair], [BasePair])
    ->{Random} [BasePair]
  crossAndMutDefault crossRate mutRate ab =
    a = at1 ab
    b = at2 ab
    zygote = withRate crossRate (crossoverUniform b) a
    withRate mutRate mutate zygote

.uniopt.evo.genetic> view withRate

  withRate : Float -> (a ->{e} a) -> a ->{e, Random} a
  withRate rate f a =
    use Float <=
    if !Random.float <= rate then f a else a

```
`withRate` is a helper function
- allows us to apply a function at the specified rate to some input data
- we use it for mutation and crossover functions


## Selection


- Natural selection acts as a filter in our optimization search:
  - optimizes organisms' fitness for their environment
  - If the fitness is too low:
    - there is an increased risk the organism will not pass on genetic
material to future generations.

[![Photo of Charles Darwin, aged 51](/media/Charles_Darwin_aged_51.jpg)](https://en.wikipedia.org/wiki/Natural_selection#/media/File:Charles_Darwin_aged_51.jpg)

Here we emulate natural selection:


```ucm
.uniopt.evo.genetic> view selectionRandomWeight

  selectionRandomWeight :
    Nat
    -> Nat
    -> Float
    -> [EntityFitness]
    ->{Random} [EntityFitness]
  selectionRandomWeight
    keepNBest totalKept inversePerturbationFactor currentPop =
    use List map reverse take
    sortedPop = reverse <| sortBy (ef -> fitness ef) currentPop
    bestPop = take keepNBest sortedPop
    restPop = List.drop keepNBest sortedPop
    restKept =
      use Nat -
      Nat.max 0 (totalKept - keepNBest)
    weights =
      use Float /
      map
        (_ -> !Random.float / inversePerturbationFactor)
        (List.range 0 restKept)
    addWeightedFitness :
      EntityFitness -> Float -> (EntityFitness, Float)
    addWeightedFitness ef weight =
      use Float +
      (ef, fitness ef + weight)
    perturbedEFs =
      List.zipWith addWeightedFitness restPop weights
    sortedPerturbedEFs = reverse <| sortBy at2 perturbedEFs
    map at1 (take restKept sortedPerturbedEFs)

.uniopt.evo.genetic> view selectionRandomWeightDefault

  selectionRandomWeightDefault :
    [EntityFitness] ->{Random} [EntityFitness]
  selectionRandomWeightDefault population =
    use Nat / max
    popSize = List.size population
    keepNBest = max 1 (popSize / 20)
    totalKept = max 1 (popSize / 5)
    selectionRandomWeight keepNBest totalKept 10.0 population

```
## Simulating a Single Generation

- We need to simulate a population evolving over multiple generations
- First we create a function to step from one generation to the next.
- This is where fitness evaluations occur
  - We would like these evaluations to take advantage of a distributed system when available.

To do this:

- Add the `Remote` ability requirement

```ucm
.uniopt.evo.genetic> diff.namespace #bhro5l052t #mf909c2pu4

  Updates:
  
    1. iterateGenDefault : Float
       -> Float
       -> ([BasePair] ->{g} Float)
       -> ([#d8v7354nh6] ->{Random} [#d8v7354nh6])
       -> [#d8v7354nh6]
       ->{g, Random} [#d8v7354nh6]
       ↓
    2. iterateGenDefault : Float
       -> Float
       -> ([BasePair] -> Float)
       -> ([#d8v7354nh6] ->{Random} [#d8v7354nh6])
       -> [#d8v7354nh6]
       ->{Remote, Random} [#d8v7354nh6]

```
- Use the distribute package's `Seq` data structure in place of `List`:
  - We can do this internally, and still use `List` in the function signature.

```ucm
.uniopt.evo.genetic> view iterateGenDefault

  iterateGenDefault :
    Float
    -> Float
    -> ([BasePair] -> Float)
    -> ([EntityFitness] ->{Random} [EntityFitness])
    -> [EntityFitness]
    ->{Remote, Random} [EntityFitness]
  iterateGenDefault
    crossRate mutRate fitnessFn selectFn population =
    use List size
    selectedChromosomes =
      List.map chromosome (selectFn population)
    selectionSize = size selectedChromosomes
    randomChromIx = '(Random.natIn 0 selectionSize)
    randomChrom =
      '(Optional.getOrElse
          [] (List.at !randomChromIx selectedChromosomes))
    chromPairs =
      List.map
        (_ -> (!randomChrom, !randomChrom))
        (List.range 0 selectionSize)
    crossAndMut = crossAndMutDefault crossRate mutRate
    newPopChroms = List.map crossAndMut chromPairs
    fitnessEvaluator = makeFitnessEvaluator fitnessFn
    newEFs =
      Seq.map
        fitnessEvaluator (Seq.fromList Parallel newPopChroms)
        |> Seq.toList
    pastAndPresentPop =
      use List ++
      newEFs ++ population |> Set.fromList
    popSize = size population
    List.take
      popSize
      (List.reverse
        <| sortBy fitness (Set.toList pastAndPresentPop))

```
- Note also that we use the `Parallel` mode
  - So if the fitness function is also parallel,
    it can fork multiple Unison processes
  - Semantics depend on the `Remote` ability handler being used.

We'll absorb the parameters in another function:

```ucm
.uniopt.evo.genetic.ex.knapsack> view runGeneration

  runGeneration :
    ([BasePair] -> Float)
    -> ([EntityFitness] ->{Random} [EntityFitness])
    -> [EntityFitness]
    ->{Remote, Random} [EntityFitness]
  runGeneration = iterateGenDefault 0.8 0.5

```
## Generation Simulation for the KnapSack Problem

To make sure we don't have surprising performance issues, let's
make sure we're only using our fitness function in one place:

```ucm
.uniopt.evo.genetic.ex.knapsack> dependents knapsackFitnessFn

  Dependents of #g0ocp1iles:
  
       Reference   Name
    1. #e27onun2k9 iterateGenKnapsack

```
`iterateGenKnapsack`:
 - wraps the generic `iterateGen` function
 - supplies our custom fitness function
 - will construct a random population if we do not supply an existing population

```ucm
.uniopt.evo.genetic.ex.knapsack> view iterateGenKnapsack

  iterateGenKnapsack :
    (([BasePair] -> Float)
    ->{g} ([EntityFitness] ->{Random} [EntityFitness])
    ->{g1} [EntityFitness]
    ->{g2, Random} [EntityFitness])
    -> KnapsackProblem
    -> Either Nat [EntityFitness]
    ->{g, g1, g2, Random} [EntityFitness]
  iterateGenKnapsack iterateGen problem popOrSize =
    fitnessFn = knapsackFitnessFn problem
    fitnessEvaluator = makeFitnessEvaluator fitnessFn
    pop =
      match popOrSize with
        Right pop    -> pop
        Left popSize ->
          List.map
            fitnessEvaluator
            (randomPopulation (items problem) popSize)
    iterateGen fitnessFn selectionRandomWeightDefault pop

```
## Running Multiple Generations

There are various ways to encode stopping conditions.

For example, we can just write a recursive function
to run N generations:


```ucm
.uniopt.evo.genetic.ex.knapsack> view runNgenerations

  runNgenerations :
    (([BasePair] -> Float)
    -> ([EntityFitness] ->{Random} [EntityFitness])
    -> [EntityFitness]
    ->{g, Random} [EntityFitness])
    -> KnapsackProblem
    -> Nat
    -> Either Nat [EntityFitness]
    ->{g, Random} [EntityFitness]
  runNgenerations iterateGen problem nIter popOrSize =
    use Nat ==
    newPop = iterateGenKnapsack iterateGen problem popOrSize
    if nIter == 0 then newPop
    else
      use Nat -
      runNgenerations
        iterateGen problem (nIter - 1) (Right newPop)

```
## Optimizing the Knapsack Problem


```unison
seed = 1
popG100 = Remote.pure.run '(Random.splitmix seed '(runNgenerations runGeneration testProblem10items 100 (Left 30)))
```

```unison
top10Lists = prettyPrintPop 10 testProblem10items popG100
```

```ucm

  I found and typechecked these definitions in scratch.u. If you
  do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      top10Lists : [(Float, Float, [Item])]

```
```ucm
.uniopt.evo.genetic.ex.knapsack> display top10Lists

  [ (193.23000000000002,
    37.080000000000005,
    [ Item 4.06 9.89,
      Item 7.2 1.31,
      Item 58.53 9.21,
      Item 29.47 7.66,
      Item 40.3 0.86,
      Item 53.67 8.15 ]),
    (195.73,
    36.56,
    [ Item 4.06 9.89,
      Item 96.47 8.49,
      Item 7.2 1.31,
      Item 58.53 9.21,
      Item 29.47 7.66 ]),
    (152.93,
    36.220000000000006,
    [ Item 4.06 9.89,
      Item 7.2 1.31,
      Item 58.53 9.21,
      Item 29.47 7.66,
      Item 53.67 8.15 ]),
    (186.03000000000003,
    35.77,
    [ Item 4.06 9.89,
      Item 58.53 9.21,
      Item 29.47 7.66,
      Item 40.3 0.86,
      Item 53.67 8.15 ]),
    (188.53,
    35.25,
    [ Item 4.06 9.89,
      Item 96.47 8.49,
      Item 58.53 9.21,
      Item 29.47 7.66 ]),
    (145.73000000000002,
    34.910000000000004,
    [ Item 4.06 9.89,
      Item 58.53 9.21,
      Item 29.47 7.66,
      Item 53.67 8.15 ]),
    (180.94000000000003,
    33.07,
    [ Item 4.06 9.89,
      Item 81.68 5.0,
      Item 7.2 1.31,
      Item 58.53 9.21,
      Item 29.47 7.66 ]),
    (164.42,
    29.250000000000004,
    [ Item 4.06 9.89,
      Item 65.16 1.18,
      Item 7.2 1.31,
      Item 58.53 9.21,
      Item 29.47 7.66 ]),
    (139.56,
    28.930000000000003,
    [ Item 4.06 9.89,
      Item 7.2 1.31,
      Item 58.53 9.21,
      Item 29.47 7.66,
      Item 40.3 0.86 ]),
    (166.26,
    28.900000000000002,
    [ Item 4.06 9.89,
      Item 96.47 8.49,
      Item 7.2 1.31,
      Item 58.53 9.21 ]) ]

```
**An aside**: no `IO` was used in this code 😀

## Appendix

Developed using the following:

```plain

ucm transcript --save-codebase transcripts/GeneticAlgorithms.md

```

Get the location of the codebase from the output, e.g.:

```plain

/tmp/transcript-bb600d0d425b1cd9

```

Then use `transcript.fork` in subsequent runs:

```plain

ucm transcript.fork transcripts/GeneticAlgorithms.md -c /tmp/transcript-bb600d0d425b1cd9

```

