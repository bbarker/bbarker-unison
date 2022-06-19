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

  Nothing changed as a result of the merge.

  😶
  
  the current namespace was already up-to-date with
  https://github.com/bbarker/uniopt.

```
## Implementation of a simple Genetic Algorithm

This GA is simple enough that it was implemented from the ground up.


We start with some basic datatypes and functions.

### Base Pairs

```ucm
.uniopt.evo.genetic> view BasePair

  structural type BasePair = Numeric Float | Binary Boolean

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

  crossoverUniform :
    [BasePair] -> [BasePair] ->{Random} [BasePair]
  crossoverUniform chrom1 chrom2 =
    List.map
      (bpbp -> (if !randomBool then at1 bpbp else at2 bpbp))
      (List.zip chrom1 chrom2)

```
### Setting rates for Mutation and Crossover


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
`withRate` is a helper function that allows us to apply a function
(such as our mutation and crossover functions) at the specified rate
to some input data.


In the following, assume we're operating the KnapSack problem example namespace.


```unison
seed = 1
popG100 = Remote.pure.run '(Random.splitmix seed '(runNgenerations runGeneration testProblem10items 100 (Left 30)))
```

```ucm

  I found and typechecked these definitions in scratch.u. If you
  do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      popG100 : Either Failure [EntityFitness]
        (also named ex.knapsack.popG100)
      seed    : Nat
        (also named ex.knapsack.seed)

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
`--save-codebase option to the above command.

