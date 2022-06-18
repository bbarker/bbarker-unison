## TODO

- Mention that this code is all checked and should be executable exactly as shown.
- How genetic algorithsm relate to evolutionary algorithms.

# Introduction

## Optimization

## Algorithms requiring Optimization

## Types of Optimization algorithms

This is not a formal or complete classification!

### Genetic Algorithms


Chromosomal Crossover            |  (depiction by Thomas Hunt Morgan)
:-------------------------:|:-------------------------:
![Depiction of chromosomal crossover, by Thomas Hunt Morgan](/media/Morgan_crossover_1.jpg)  |  ![Thomas Hunt Morgan](/media/Thomas_Hunt_Morgan.jpg)



```ucm
.> pull https://github.com/bbarker/uniopt

  Nothing changed as a result of the merge.

  😶
  
  the current namespace was already up-to-date with
  https://github.com/bbarker/uniopt.

```
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
        (also named uniopt.evo.genetic.ex.knapsack.popG100)
      seed    : Nat
        (also named uniopt.evo.genetic.ex.knapsack.seed and
        datetime.std.systemTimeTicksPerSecond)

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

