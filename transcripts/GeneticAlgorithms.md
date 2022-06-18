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
```

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

