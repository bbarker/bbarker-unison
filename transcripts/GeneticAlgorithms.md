# Prerequisites

To experiment with this code, you can use the
UniOpt codebase from the UCM:


```ucm
.> pull https://github.com/bbarker/uniopt
```

The code is written as a
[Unison transcript](https://www.unison-lang.org/learn/tooling/transcripts/),
and is evaluated based on the most recent version of `UniOpt` stored on GitHub.

# Introduction and Overview

- I used to use optimization algorithms to find solutions
  - (to biological problems)
- In my case, I was limited to running these on a single system.
- After seeing Unison's distributed computing capabilities, I thought:
  - I should try a distributed algorithm in Unison
  - A relatively simple, flexible, and distributed-friendly algorithm I've used before
  - Also Unison is fun

---
- We'll talk about:Genetic Algorithms (GAs)
  - Where they fit
  - What they do
  - An example problem (the Knapsack Problem)
  - An implementation in Unison
  - How we make it distributed friendly
  - And show an example run
## Algorithms requiring Optimization

 - AI/ML
 - Scheduling
 - Routing
 - Cost Savings
 - Parameter tuning (this is where I used a GA previously)
## Genetic Algorithms

- There are many types of optimization algorithms
  - Beyond the scope of this presentation

- The most common form of evolutionary algorithm (EA).
- A sequence of "base pairs" (e.g. numbers) represent a
solution to the problem.
- The "DNA" is genetically manipulated.
- GAs are commonly used for optimization, compared to some other EAs.
- Attempts to find a global optima, but there is typically no guarantee.
[![2D depiction of local and global optima](/media/Extrema_example_original.svg)](https://en.wikipedia.org/wiki/Maxima_and_minima#/media/File:Extrema_example_original.svg)

- ***Requires many function evaluations, which may be expensive***.
  - Both in the # of evaluations, and the length of each evaluation.
  - **A distributed variant would be ideal in this case**


## An Example Problem: The Knapsack Problem

- You are going on a trip.
- You can only bring items that you can fit into a knapsack.
- The knapsack has a weight limit.
- Which items do you take?


[![Depiction of the Knapsack Problem](/media/Knapsack.svg)](https://commons.wikimedia.org/wiki/File:Knapsack.svg)

- There are many approaches to solve this.
- Some specialized approaches that guarantee a global optimum.
- Regardless, we can use the Knapsack problem as an example for a GA.

First, we model an `Item`, which has a weight and a value:

```ucm
.uniopt.evo.genetic.ex.knapsack> view Item
```

- We need a way to keep track of all items under consideration.
knapsack.
- We can use a `Map`, as well as an inverted `Map`.

```ucm
.uniopt.evo.genetic.ex.knapsack> view PossibleItems
.uniopt.evo.genetic.ex.knapsack> view PossibleItemsRev
```

The problem description includes:
  - The two maps
  - The weight limit

We also have a constructor to help us create `PossibleItemsRev`:

```ucm
.uniopt.evo.genetic.ex.knapsack> view KnapsackProblem
.uniopt.evo.genetic.ex.knapsack> view makeKnapsackProblem
```

Let's see if we have any existing test data sets by checking the `dependents`
of `makeKnapsackProblem`:

```ucm
.uniopt.evo.genetic.ex.knapsack> dependents makeKnapsackProblem
```

We do! Let's see how it is defined:

```ucm
.uniopt.evo.genetic.ex.knapsack> view testProblem10items
```


# Implementation of a simple Genetic Algorithm

This GA is simple enough that it was implemented from the ground up.


We start with some basic datatypes and functions.

## Base Pairs

```ucm
.uniopt.evo.genetic> view BasePair
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
.uniopt.evo.genetic> view makeFitnessEvaluator
```

## Going between the Problem Domain and the GA Model

First, we need a sensible fitness function:

```ucm
.uniopt.evo.genetic.ex.knapsack> view knapsackFitnessFn
```

- The fitness is the sum of item values
- ... Unless the sum of item weights exceeds the limit, then it is 0


After generating and evolving our chromosomes, we need a way
to translate them back into the problem space:

```ucm
.uniopt.evo.genetic.ex.knapsack> view chromosomeToItems
.uniopt.evo.genetic.ex.knapsack> view PossibleItemsRev
```

We have a few use cases:

```ucm
.uniopt.evo.genetic.ex.knapsack> dependents chromosomeToItems
```


## Changing the DNA (Exploring the Solution Space)

## Mutation

- During each generation, each "organism" (chromosome):
  - Will mutate zero or more of its basepairs
      - To keep things simple, we will just perform one mutation
      - Later we will specify the rate of mutation

```ucm
.uniopt.evo.genetic> view mutate
```

We also need to implement `mutateBasePair`, which handles mutation for
each component type of `BasePair`:


```ucm
.uniopt.evo.genetic> view mutateBasePair
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
```

## Setting Rates for Mutation and Crossover


```ucm
.uniopt.evo.genetic> view crossAndMutDefault
.uniopt.evo.genetic> view withRate

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
.uniopt.evo.genetic> view selectionRandomWeightDefault
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
```

Let's take a look at the implementation:


```ucm
.uniopt.evo.genetic> view iterateGenDefault
```

- Use the [distributed project](https://share.unison-lang.org/latest/namespaces/unison/distributed)'s
   `Seq` data structure in place of `List`:
  - We can do this internally, and still use `List` in the function signature.
- Note also that we use the `Parallel` mode
  - So if the fitness function is also parallel,
    it can fork multiple Unison processes
  - Semantics depend on the `Remote` ability handler being used.

We'll absorb the parameters in another function:

```ucm
.uniopt.evo.genetic.ex.knapsack> view runGeneration
```

## Generation Simulation for the KnapSack Problem

To make sure we don't have surprising performance issues, let's
make sure we're only using our fitness function in one place:

```ucm
.uniopt.evo.genetic.ex.knapsack> dependents knapsackFitnessFn
```

`iterateGenKnapsack`:
 - wraps the generic `iterateGen` function
 - supplies our custom fitness function
 - will construct a random population if we do not supply an existing population

```ucm
.uniopt.evo.genetic.ex.knapsack> view iterateGenKnapsack
```

## Running Multiple Generations

There are various ways to encode stopping conditions.

For example, we can just write a recursive function
to run N generations:


```ucm
.uniopt.evo.genetic.ex.knapsack> view runNgenerations
```

## Optimizing the Knapsack Problem


```unison:hide
seed = 1
popG100 = Remote.pure.run '(Random.splitmix seed '(runNgenerations runGeneration testProblem10items 100 (Left 30)))
```

```ucm:hide
.uniopt.evo.genetic.ex.knapsack> add popG100
```

```unison
top10Lists = prettyPrintPop 10 testProblem10items popG100
```


```ucm
.uniopt.evo.genetic.ex.knapsack> display top10Lists

```
**An aside**: no `IO` was used in this code 😀


# Acknowledgements
 - Unison Community (for answering questions)
  - Rebecca Mark
  - Rúnar Bjarnason
  - Paul Chiusano
  - and others!
 - Charad Sheerajin (for giving feedback on the talk and transcript)

# Appendix

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

