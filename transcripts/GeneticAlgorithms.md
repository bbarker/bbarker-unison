## TODO

- Mention that this code is all checked and should be executable exactly as shown.
- How genetic algorithsm relate to evolutionary algorithms.
- Double check image credits

# Introduction

## Optimization

## Algorithms requiring Optimization

## Types of Optimization algorithms

This is not a formal or complete classification!

### Genetic Algorithms





```ucm
.> pull https://github.com/bbarker/uniopt
```

## Implementation of a simple Genetic Algorithm

This GA is simple enough that it was implemented from the ground up.


We start with some basic datatypes and functions.

### Base Pairs

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

### Mutation

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

### Recombination and Crossover

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

### Setting rates for Mutation and Crossover


```ucm
.uniopt.evo.genetic> view crossAndMutDefault
.uniopt.evo.genetic> view withRate

```

`withRate` is a helper function that allows us to apply a function
(such as our mutation and crossover functions) at the specified rate
to some input data.


In the following, assume we're operating the KnapSack problem example namespace.


```unison
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

