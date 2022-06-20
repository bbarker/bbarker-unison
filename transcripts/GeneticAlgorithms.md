# Prerequisites

To experiment with this code, you can use the
uniopt codebase:


```ucm
.> pull https://github.com/bbarker/uniopt
```
# Introduction

## Optimization

## Algorithms requiring Optimization

## Types of Optimization algorithms

This is not a formal or complete classification!

### Genetic Algorithms


## An example problem: Knapsack Problem

You are going on a trip and can only bring items that you can
fit into a knapsack, which has a specified weight limit.
Assuming volume constraints aren't an issue here - which items do you take?

There are many approaches to solve this, including approaches that guarantee
a global optimum; these methods tend to be specialized for the Knapsack problem,
whereas GAs are much broader in scope.

Regardless, we can use the Knapsack problem as an example.

First, we model an `Item`, which has a weight and a value:

```ucm
.uniopt.evo.genetic.ex.knapsack> view Item
```

We also need a way to keep track of all the items we are considering putting in our
knapsack. We choose a `Map`, as well as a reversed variant of the `Map` for convenience

```ucm
.uniopt.evo.genetic.ex.knapsack> view PossibleItems
.uniopt.evo.genetic.ex.knapsack> view PossibleItemsRev
```

We create an entire problem description as a value of a structural type
which includes possible items and the weight limit, and
create a constructor to help us generate `PossibleItemsRev`:

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

In the future, `BasePair` may expand to include other types of data.
In the example below, we only use binary data.

Actual genetic sequences would be base 4 instead of base 2, but for our purposes,
binary (base 2) is more convenient.


For example, in the Knapsack example, we'll be using a `0` to denote the absence
of an item, and `1` the presence of an item.


## Changing the DNA

## Mutation

With mutation, during each generation, each "organism" will mutate zero or more
of its basepairs. To keep things simple, we will cap this at one mutation:

```ucm
.uniopt.evo.genetic> view mutate
```

We also need to implement `mutateBasePair`, which handles mutation for
each component type of `BasePair`:


```ucm
.uniopt.evo.genetic> view mutateBasePair
```

## Recombination and Crossover

Recombination is the process of exchanging genetic material (DNA) between
different organisms.

We can think about this in terms of a specific type of recombination: chromosomal crosssover.



Chromosomal Crossover            |  (depiction by Thomas Hunt Morgan)
:-------------------------:|:-------------------------:
![Depiction of chromosomal crossover, by Thomas Hunt Morgan](/media/Morgan_crossover_1.jpg)  |  ![Thomas Hunt Morgan](/media/Thomas_Hunt_Morgan.jpg)


Unlike in natural chromosomal crossover, which is bound by physical limitations, we can allow
crossing over at every base pair.

```ucm
.uniopt.evo.genetic> view crossoverUniform
```

## Setting rates for Mutation and Crossover


```ucm
.uniopt.evo.genetic> view crossAndMutDefault
.uniopt.evo.genetic> view withRate

```

`withRate` is a helper function that allows us to apply a function
(such as our mutation and crossover functions) at the specified rate
to some input data.


## Selection

Natural selection is a form of optimization, typically optimizing an organisms
fitness for their environment.

If the fitness is too low, there is a risk the organism will not pass on genetic
material to future generations. Here we replicate that process:


```ucm
.uniopt.evo.genetic> view selectionRandomWeight
.uniopt.evo.genetic> view selectionRandomWeightDefault
```



## Simulating a Single Generation

We need to simulate a population evolving over multiple generations,
so first we create a function to step from one generation to the next.


This is where fitness evaluations occur, so we would like these evaluations
to take advantage of a distributed system when available.

To do this:

- Add the `Remote` ability requirement

```ucm
.uniopt.evo.genetic> diff.namespace #bhro5l052t #mf909c2pu4
```

- Use the distribute package's `Seq` data structure in place of `List`:

```ucm
.uniopt.evo.genetic> view iterateGenDefault
```

## Generation simulation for the KnapSack Problem

To make sure we don't have surprising performance issues, let's
make sure we're only using our fitness function in one place:

```ucm
.uniopt.evo.genetic.ex.knapsack> dependents knapsackFitnessFn
```

`iterateGenKnapsack` wraps the generic `iterateGen` function and supplies
our custom fitness function, and will construct a random population if
we do not supply an existing population.

```ucm
.uniopt.evo.genetic.ex.knapsack> view iterateGenKnapsack
```


## Optimizing the Knapsack Problem


```unison:hide
seed = 1
popG100 = Remote.pure.run '(Random.splitmix seed '(runNgenerations runGeneration testProblem10items 100 (Left 30)))
```

```unison
top10Lists = prettyPrintPop 10 testProblem10items popG100
```


```ucm
.uniopt.evo.genetic.ex.knapsack> display top10Lists
```

Before using the `Remote` ability, we could run our GA like this:

```
--popG10 = Random.splitmix seed '(runNgenerations runGeneration testProblem10items 3 (Left 30))
```


## Appendix

Developed using the following:

```plain
ucm transcript --save-codebase transcripts/GeneticAlgorithms.md
```


Get the location of the codebase from the output, e.g.:

```plain
/tmp/transcript-23e6a46192092a18
```

Then use `transcript.fork` in subsequent runs:

```plain
ucm transcript.fork transcripts/GeneticAlgorithms.md -c /tmp/transcript-23e6a46192092a18
```

Eventually, if you want to save a new state, you can add the
`--save-codebase` option to the above command.


## TODO

- Mention that this code is all checked and should be executable exactly as shown.
- Section on conversion between Problem (domain) and GA
- How genetic algorithms relate to evolutionary algorithms.
- Double check image credits
