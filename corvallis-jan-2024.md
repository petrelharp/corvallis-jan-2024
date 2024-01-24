---
title: "Simulation-based inference for ecology and evolution"
author: "Peter Ralph // University of Oregon"
date: "Ecology, Evolution, and Conservation Biology <br/> Oregon State University // 24 Jan 2024"
---


# Outline

<!-- Abstract:
Modern datasets have made it possible to use genomic data to try to answer
increasingly detailed and practically relevant questions about evolution and ecology.
Although in many applications analytical inference methods are infeasible or not available,
simulation-based inference is possible for any system that can be simulated.
In this talk I will describe recent progress in simulating large, evolving populations,
and storing and analyzing the entire genomic history of these simulations (using SLiM and
the "tree sequence" data structure). I will then give two examples of applications: understanding
how the balance of different types of natural selection, mutation rate variation, and other processes
produced the landscapes of genetic diversity in the great apes; and describing conditions under which
coevolution can produce spatial correlations in phenotypes such as seen in the classic
Taricha newt-Thamnophis garter snake system.
-->

## Outline of the talk

1. Tree sequences, and simulations

2. Spatial landscapes of coevolving traits

3. Landscapes of genetic diversity


*slides:* [github.com/petrelharp/corvallis-jan-2024](https://petrelharp.github.io/corvallis-jan-2024/corvallis-jan-2024.slides.html)


# Inference, with genomes


##

![](figs/modeling_empty.png)

##

![](figs/modeling_reality.png)

##

![](figs/modeling_model_parameters.png)

##

![](figs/modeling_model_parameters_inverse.png)

##

![](figs/modeling_parameters_inverse_computer.png)

## Simulation-based inference

::: {.centered}
![](figs/modeling_parameters_inverse_computer.png){width=60%}
:::


- bespoke confirmatory simulations
- optimization of one or two parameters
    <!-- *(if between-simulation noise is small)* -->
- Approximate Bayesian Computation (ABC)
- deep learning


# What do we need

##

1. Fast simulation of genomes

2. Fast computation of summary statistics

## Wish list:

::: {.smallish}

::: {.columns}
::::::: {.column}

Whole genomes,
thousands of samples, \
from millions of individuals.

**Demography:**

- life history 
- separate sexes
- selfing
- polyploidy
- species interactions

**Geography:**

- discrete populations
- continuous landscapes
- barriers

**History:**

- ancient samples
- range shifts

:::
::::::: {.column}

**Natural selection:**

- selective sweeps
- introgressing alleles
- background selection
- quantitative traits
- incompatibilities
- local adaptation

**Genomes:**

- recombination rate variation
- gene conversion
- infinite-sites mutation
- nucleotide models
- context-dependence
- mobile elements
- inversions
- copy number variation

:::
:::::::

:::




## Enter SLiM


::: {.columns}
::::::: {.column width=50%}


by Ben Haller and Philipp Messer

an individual-based, scriptable forwards simulator

:::: {.caption}
![Ben Haller](figs/ben-haller.jpg){width=45%}
![Philipp Messer](figs/philipp.jpg){width=45%}
::::

:::
::::::: {.column width=50%}

![SLiM GUI](figs/slim-gui.png)

[messerlab.org/SLiM](https://messerlab.org/SLiM/)

:::
:::::::



##

::: {.smallish}

::: {.columns}
::::::: {.column}

- <s>Whole genomes,</s>*
- <s>thousands of samples, </s>
- <s>from millions of individuals.</s>*

**Demography:**

- <s>life history</s>
- <s>separate sexes</s>*
- <s>selfing</s>
- polyploidy*
- <s>species interactions</s>

**Geography:**

- <s>discrete populations</s>
- <s>continuous landscapes</s>
- <s>barriers</s>*

**History:**

- <s>ancient samples</s>
- <s>range shifts</s>

:::
::::::: {.column}

**Natural selection:**

- <s>selective sweeps</s>
- <s>introgressing alleles</s>
- <s>background selection</s>
- <s>quantitative traits</s>*
- <s>incompatibilities</s>*
- <s>local adaptation</s>*

**Genomes:**

- <s>recombination rate variation</s>
- <s>gene conversion</s>
- <s>infinite-sites mutation</s>
- <s>nucleotide models</s>
- <s>context-dependence</s>*
- mobile elements*
- inversions*
- copy number variation

:::
:::::::

:::



<!-- Tree sequences -->

# The tree sequence


## History is a sequence of trees

For a set of sampled chromosomes,
at each position along the genome there is a genealogical tree
that says how they are related.

. . .

![Trees along a chromosome](figs/example.svg)


----------------------

The **succinct tree sequence**

::: {.floatright}
is a way to succinctly describe this, er, sequence of trees

*and* the resulting genome sequences.

:::: {.caption}
[Kelleher, Etheridge, & McVean](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1004842) 
::::
:::

. . .

::: {.columns}
:::::: {.column width=50%}

![tskit logo](figs/tskit_logo.png){width=80%}

:::
:::::: {.column width=50%}

::: {.floatright}
![jerome kelleher](figs/jerome.jpeg){width=50%}

:::: {.caption}
jerome kelleher
::::

:::

:::
::::::



## How's it work? File sizes:

::: {.centered}
![file sizes](figs/tsinfer_sizes.png){width=90%}
:::

::: {.caption}
100Mb chromosomes;
from [Kelleher et al 2018, *Inferring whole-genome histories in large population datasets*](https://www.nature.com/articles/s41588-019-0483-y), Nature Genetics
:::

<!-- Estimated sizes of files required to store the genetic variation data for a
simulated human-like chromosome (100 megabases) for up to 10 billion haploid
(5 billion diploid) samples. Simulations were run for 10 1 up to 10 7 haplotypes
using msprime [Kelleher et al., 2016], and the sizes of the resulting files plotted
(points). -->


---------------

![genotypes](figs/ts_ex/tree_sequence_genotypes.png)

---------------

![genotypes and a tree](figs/ts_ex/tree_sequence_genotype_and_tree.png)

---------------

![genotypes and the next tree](figs/ts_ex/tree_sequence_next_genotype_and_tree.png)



## For $N$ samples genotyped at $M$ sites

::: {.columns}
::::::: {.column width=50%}


*Genotype matrix*:

$N \times M$ things.


:::
::::::: {.column width=50%}

*Tree sequence:*

- $2N-2$ edges for the first tree
- $\sim 4$ edges per each of $T$ trees
- $M$ mutations

$O(N + T + M)$ things

:::
:::::::

![genotypes and a tree](figs/ts_ex/tree_sequence_genotype_and_tree.png){width=60%}




## How's it work, 2? Computation time:

::: {.centered}
![efficiency of treestat computation](figs/treestats/benchmarks_without_copy_longer_genome.png){width=70%}
:::

::: {.caption}
from [R., Thornton and Kelleher 2019, *Efficiently summarizing relationships in large samples*](https://academic.oup.com/genetics/article/215/3/779/5930459), Genetics
:::


# Application to genomic simulations

## The main idea

Genomes are very big.

. . .

But, if we *record the tree sequence*
that relates everyone to everyone else,

after the simulation is over we can put neutral mutations down on the trees.

. . .

Since neutral mutations don't affect demography,

this is *equivalent* to having kept track of them throughout.


------------

This means recording the entire genetic history of **everyone** in the population, **ever**.

.  . .

It is *not* clear this is a good idea.

. . .

But, with a few tricks...

. . .

:::: {.columns}
:::::::: {.column width=50%}

:::: {.caption}
From 
Kelleher, Thornton, Ashander, and R. 2018,
[Efficient pedigree recording for fast population genetics simulation](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1006581).

and Haller, Galloway, Kelleher, Messer, and R. 2018,
[*Tree‐sequence recording in SLiM opens new horizons for forward‐time simulation of whole genomes*](https://onlinelibrary.wiley.com/doi/abs/10.1111/1755-0998.12968)
:::

::::
:::::::: {.column width=50%}

::: {.floatright}
![jared galloway](figs/jared.jpeg){width=35%}
![jaime ashander](figs/jaime.jpg){width=30%}
![ben haller](figs/ben-haller.jpg){width=30%}
:::

::::
::::::::




## A 100x speedup!

::: {.columns}
:::::: {.column width=40%}

::: {.centered}
![SLiM logo](figs/slim_logo.png){width=100%}
:::

:::
:::::: {.column width=60%}

::: {.floatright}
![](figs/slim_timing_no_msprime.png){width=100%}
:::

:::
::::::


<!-- Snewts -->

# Spatial coevolution: snakes and newts

:::: {.columns}
:::::::: {.column width=70%}

![Battling newts and snakes](figs/snewts/biowarfare_cartoon.gif)

::::
:::::::: {.column width=30%}
::: {.centered}

![Victoria Caudill](figs/victoria.jpg){width=80%}

Victoria Caudill

:::
::::
::::::::

::::::::::: {.caption}
Image from [evolution.berkeley.edu](https://evolution.berkeley.edu/biological-warfare-and-the-coevolutionary-arms-race/coevolution/)
:::::::::::

## {data-background-image="figs/snewts/snewt-lit.png" data-background-position="top" data-background-width=60%}

::: {.caption .bottomright .smallish}
newt: [CC by Steve Jurvetson](https://en.m.wikipedia.org/wiki/File:California_Newt_%28Taricha_torosa%29_%284415749606%29.jpg)

snake: [CC by Jaden Clark](https://commons.wikimedia.org/wiki/File:California_Red-Sided_Garter_Snake_\(Thamnophis_sirtalis_infernalis\).jpg)
:::


---------------

![Fig 1 from Reimche et al 2020](figs/snewts/reimche2020-fig1.jpg){width=70%}

::::::::::: {.caption}
[Fig 1, Reimche et al 2020](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/1365-2656.13212)
:::::::::::

---------------

![Fig 2 from Hanifin et al 2008](figs/snewts/hanifin2008-fig2.png){width=70%}

::::::::::: {.caption}
[Fig 2, Hanifin et al 2008](https://journals.plos.org/plosbiology/article?id=10.1371/journal.pbio.0060060)
:::::::::::

## Why, and how?

![](figs/snewts/brodie2002abstract.png){width=100%}


## A spatial co-evolutionary simulation

:::: {.columns}
:::::::: {.column width=50%}

- continuous space
- local density-dependent mortality
- additive, costly traits ("toxicity" and "resistance")
- various genetic architectures
- snakes may encounter nearby newts,
    outcome depends on difference in traits:

    * snake eats newt, gets fitness benefit, or
    * snake dies, newt escapes

::::
:::::::: {.column width=50%}

![Screenshot of a SLiM simulation](figs/snewts/slim-screenshot.png)

::::
::::::::

## No spatial correlation on a flat map

:::: {.columns}
:::::::: {.column width=50%}

![](figs/snewts/nocor_map.png)

::::
:::::::: {.column width=50%}

![](figs/snewts/nocor_boxplots.png)

::::
::::::::

## Add some heterogeneity

![](figs/snewts/gradient-maps.png)

------------

:::: {.columns}
:::::::: {.column width=50%}

![](figs/snewts/cor_map.png)

::::
:::::::: {.column width=50%}

![](figs/snewts/cor_boxplots.png)

::::
::::::::

## Conclusions

:::: {.columns}
:::::::: {.column width=50%}

![Fig 2 from Hanifin et al 2008](figs/snewts/hanifin2008-fig2.png)

::::
:::::::: {.column width=50%}

- such large correlated differences across the landscape
    unlikely to be due to nonadaptive forces

- spatial heterogeneity in ecological factors
    much more plausible

- trait genetic architecture has little effect,
    given sufficient variation


::: {.caption .floatright}
![Victoria Caudill](figs/victoria.jpg){width=20% .floatright}

Victoria Caudill
:::

::::
::::::::




<!-- Great apes -->

# Landscapes of genetic diversity

[![https://www.biorxiv.org/content/10.1101/2023.02.07.527547v2](figs/murillo/bioRxiv.png){width=70% .floatleft}](https://www.biorxiv.org/content/10.1101/2023.02.07.527547v2)

::: {.caption .floatright}
![](figs/murillo.jpeg){width=80%}

[Murillo Rodrigues](https://www.m-rodrigues.me/)
:::

## Landscapes of genetic diversity within groups of species

![](figs/murillo/correlated-pi-examples.png){width=80%}

::: {.caption .floatright}
[Stankowski, Chase et al. (2019)](https://journals.plos.org/plosbiology/article?id=10.1371/journal.pbio.3000391); [Van Doren et al. (2017)](https://onlinelibrary.wiley.com/doi/10.1111/mec.14083); [Buri (2017)](https://academic.oup.com/evlett/article/1/3/118/6697077)
:::

## The study system: "us"

:::: {.columns}
:::::::: {.column width=50%}

- High quality genomic data for 5 species
- Deep divergence, spanning ~60N generations
- Conservation of genes, recombination rate and other genomic features

::::
:::::::: {.column width=50%}

![Phylogeny of the great apes from Javier-Prado et al. (2013) Nature](figs/murillo/greatapes.png)

:::{.caption .flushright}
[Javier-Prado et al. (2013)](https://www.nature.com/articles/nature12228)
:::

::::
::::::::

## Genetic diversity in the great apes: chromosome 12

![](figs/murillo/greatapes_pi.png)

## Genetic divergence between the great apes: chromosome 12

![](figs/murillo/greatapes_dxy.png)

## Goals {data-background-image="figs/murillo/landscapes_cover.png" data-background-position="bottom" data-background-size=60%}

- How correlated are landscapes for closely related species?
- How much is due to shared footprints of
    * history?
    * selection? what kind?
    * mutational processes?

![](figs/murillo/cor-pidxy-dT_selected-positive.png){.invisible height=80%}

----------------

::: {.centered}
![](figs/murillo/far_divergence_tree.png){width=80%}
:::

## A quick primer {data-background-image="figs/murillo/far_divergence_tree.png" data-background-position="top 200px right 50px" data-background-size=30%}

::: {.columns}
:::::: {.column width=70%}

> - Genetic diversity ($\pi$) and divergence ($d_{XY}$) are
>     *time since common ancestor*
>     multiplied by *effective mutation rate*:
>     $$ \pi, d_{XY} = T_\text{MRCA} \times \mu_\text{effective} . $$
> - Higher mutation rate $\Rightarrow$ higher $\pi$
>     and faster increase of $d_{XY}$.
> - Positive selection *increases* fixation rate of new mutations
>     $\Rightarrow$ $d_{XY}$ goes up faster,
> - ... but, it *decreases* $\pi$ nearby.

:::
::::::

--------------

**linked selection:** The indirect effects of selection on genomic locations
  that are *linked* to the sites under selection by a lack of recombination.

Selection tends to *decrease* diversity,
  over a distance determined by recombination rate.

::: {.centered}

![](figs/sweep_haps.png){.fragment width="40%"}
![](figs/background_haps.png){.fragment width="40%"}

:::

<!--
----------------------
![Langley et al 2012](figs/from_the_literature/langley-et-al-2012-chr3-pi-and-rho.png)
-->


## Diversity correlates with recombination rate

:::: {.columns}
:::::::: {.column width=80%}


![Corbett-Detig et al](figs/from_the_literature/corbett-detig-divergence-recomb-all-species.png){width=85%}

::::
:::::::: {.column width=20%}


*Hudson 1994; Cutter & Payseur 2013; Corbett-Detig et al 2015*

::::
::::::::


<!-- Divergence landscapes changing over time (from Murillo) -->

## How diversity landscapes change

:::: {.columns}
:::::::: {.column width=50%}

![](figs/murillo/close_divergence_tree.png)

::::
:::::::: {.column width=50%}

![](figs/murillo/two_spp_neutral_div_landscapes_0N.jpg)

::::
::::::::

## How diversity landscapes change

:::: {.columns}
:::::::: {.column width=50%}


::::
:::::::: {.column width=50%}

![](figs/murillo/two_spp_neutral_div_landscapes.gif)

::::
::::::::

## How diversity landscapes change

:::: {.columns}
:::::::: {.column width=50%}

![](figs/murillo/far_divergence_tree.png)

::::
:::::::: {.column width=50%}

![](figs/murillo/two_spp_neutral_div_landscapes_4N.jpg)

::::
::::::::

## How can landscapes stay correlated?

:::: {.columns}
:::::::: {.column width=50%}

- shared processes:
    * positive, negative selection
    * mutation rate variation
    * GC-biased gene conversion

::::
:::::::: {.column width=50%}

![](figs/murillo/two_spp_sel_div_landscapes_0N.jpg)

::::
::::::::

## How can landscapes stay correlated?

:::: {.columns}
:::::::: {.column width=50%}

- shared processes:
    * positive, negative selection
    * mutation rate variation
    * GC-biased gene conversion

::::
:::::::: {.column width=50%}

![](figs/murillo/two_spp_sel_div_landscapes.gif)

::::
::::::::

## How can landscapes stay correlated?

:::: {.columns}
:::::::: {.column width=50%}

- shared processes:
    * positive, negative selection
    * mutation rate variation
    * GC-biased gene conversion

::::
:::::::: {.column width=50%}

![](figs/murillo/two_spp_sel_div_landscapes_4N.jpg)

::::
::::::::

<!-- End divergence landscapes changing over time -->

## SLiMulations

:::: {.columns}
:::::::: {.column width=50%}

- Chromosome-scale simulation (chr12)
- selection on annotated exons (from Ensembl via [stdpopsim](https://popsim-consortium.github.io/stdpopsim-docs/main/catalog.html#sec_catalog_HomSap_annotations))
- deCode genetic map
- branches simulated independently with SLiM and merged [tskit](https://tskit.dev/pyslim/docs/latest/vignette_parallel_phylo.html)
- runtime: days to weeks

::::
:::::::: {.column width=50%}

![](figs/murillo/demog_tubes_greatapes.png)

::::
::::::::


## Back to the data

::: {.centered}
![](figs/murillo/greatapes_pi.png){width=70%}
![](figs/murillo/greatapes_dxy.png){width=70%}
:::

##

![](figs/murillo/scatter-examples-pidxy_win-size_1000000_merged-mask_True_state_all_curr_all_prop-acc_0.4.png)

![](figs/murillo/trees-with-dt.png)

## The data: empirical correlations for chromosome 12 in the great apes

![](figs/murillo/cor-pidxy-dT_data.png)



## No correlation under neutrality {data-background-image="figs/murillo/cor-pidxy-dT_data.png" data-background-position="top 100px right" data-background-size=30%}

:::: {.columns}
:::::::: {.column width=30%}

- correlations decay to zero quickly with split time

::::
:::::::: {.column width=70%}

![](figs/murillo/corr-pidxy-land-panel_neutral.png)

::::
::::::::



## Mutation rate variation can contribute {data-background-image="figs/murillo/cor-pidxy-dT_data.png" data-background-position="top 100px right" data-background-size=30%}

:::: {.columns}
:::::::: {.column width=30%}

- but correlations do not decay with time

::::
:::::::: {.column width=70%}

![](figs/murillo/cor-pidxy-dT_selected-mutvar-sims_1.png){width=80% .centered}

::::
::::::::



## Negative selection produces weak correlations {data-background-image="figs/murillo/cor-pidxy-dT_data.png" data-background-position="top 100px right" data-background-size=30%}


![](figs/murillo/cor-pidxy-dT_selected-negative.png)


## Positive selection produces strong correlations {data-background-image="figs/murillo/cor-pidxy-dT_data.png" data-background-position="top 100px right" data-background-size=30%}

- sometimes too strong

:::: {.columns}
:::::::: {.column width=30%}

::::
:::::::: {.column width=70%}

$$ \vphantom{ d_{xy} = \pi_\text{anc} \nearrow + \mu T_\text{MRCA} \searrow } $$

::::
::::::::

![](figs/murillo/cor-pidxy-dT_selected-positive.png){.centered}


<!--

## Positive selection produces strong correlations {data-background-image="figs/murillo/cor-pidxy-dT_data.png" data-background-position="top 100px right" data-background-size=30%}

:::: {.columns}
:::::::: {.column width=30%}

::::
:::::::: {.column width=70%}

$$ { d_{xy} = \pi_\text{anc} \nearrow + \mu T_\text{MRCA} \searrow } $$

::::
::::::::

![](figs/murillo/cor-pidxy-dT_selected-positive.png){.centered}

-->


## The best fit: both! {data-background-image="figs/murillo/cor-pidxy-dT_data.png" data-background-position="top 100px right" data-background-size=30%}

- chosen from 37 distinct simulations
- deleterious mutations: $1.2 \times 10^{-8}$
- beneficial mutations: $10^{-12}$

![](figs/murillo/cor-pidxy-dT_selected-both.png){width=60% .centered}

## Conclusions {data-background-image="figs/murillo/landscapes_cover.png" data-background-position="bottom" data-background-size=60%}

- positive selection most necessary for a good fit
- best guess: $\approx 10\%$ of fixations on human lineage
    due to positive selection (1/250 years);
- $\approx$ 70% of mutations in exons deleterious
- fixation rate in exons reduced by half
- GC-biased gene conversion causes "smile" at ends of chromosomes

![](figs/murillo/cor-pidxy-dT_selected-positive.png){.invisible height=80%}


# Wrap-up

## Other applications

- history of *Nebria* beetles (Gilia Patterson, Sean Schoville, Yi-Ming Weng)
- inference of mean dispersal distance (Chris Smith; disperseNN)
- and of maps of effective density and dispersal distance (Chris Smith; mapNN)
- inference of recombination rate maps (Jeff Adrion; ReLERNN)

. . .

- inference of maps of population density (Gilia Patterson)
- and individual movement
- identification of regions under selection

## Software development goals

::: {.columns}
:::::: {.column width=50%}

- open
- welcoming and supportive
- reproducible and well-tested
- backwards compatible
- well-documented
- capacity building

::: {.centered}
![popsim logo](figs/popsim.png){width=35%}
:::

[PopSim Consortium](https://popsim-consortium.github.io/stdpopsim-docs/stable/index.html)

:::
:::::: {.column width=50%}


::: {.centered}
![tskit logo](figs/tskit_logo.png){width=60%}

[tskit.dev](https://tskit.dev)

![SLiM logo](figs/slim_logo.png){width=80%}
:::

:::
::::::


## Thanks!

:::: {.columns}
:::::::: {.column width=50%}


- Andy Kern
- Victoria Caudill
- Murillo Rodrigues 
- Gilia Patterson
- Chris Smith
- Nate Pope
- Jiseon Min
- Clara Rehmann
- Bruce Edelman
- Anastasia Teterina
- Matt Lukac
<!--
- Saurabh Belsare
- Gabby Coffing
- Jeff Adrion
- CJ Battey
- Jared Galloway
-->
- the rest of [the Co-Lab](https://kr-colab.github.io/people)

Funding:

- NIH NIGMS
- NSF DBI

::::
:::::::: {.column width=50%}

<div style="font-size: 85%; margin-top: -40px;">


- Jerome Kelleher
- Ben Haller
- Ben Jeffery
- Yan Wong
- Elsie Chevy
- Madeline Chase
- Sean Stankowski
- Matt Streisfeld
<!--
- Georgia Tsambos
- Jaime Ashander
- Jared Galloway
- Gideon Bradburd
- Bill Cresko
- Alison Etheridge
- Evan McCartney-Melstad
- Brad Shaffer
-->

</div>

::: {.floatright}
![](figs/KernRalph_5x5.png){width=20%}
![tskit logo](figs/tskit_logo.png){width=30%}
![SLiM logo](figs/slim_logo.png){width=30%}
![](figs/colab.png){width=70%}
:::

::::
::::::::



## {data-background-image="figs/guillemots_thanks.png" data-background-position=center data-background-size=50%}

<!-- Nebria -->

# Beetles in the mountains

## *Nebria ingens/riversi*

::: {.columns}
:::::: {.column width=40%}

- ground beetles
- live on flowing snowmelt in the Sierra Nevada of California
- cannot fly
- live 1--2 years

:::
:::::: {.column width=60%}

::: {.floatright}
![](figs/nebria/nebria_overview.png){width=100%}
:::

:::
::::::


------------------


::: {.columns}
:::::: {.column width=40%}

::: {.centered}
![Nebria habitat in the LGM](figs/nebria/LGM.png){width=100%}
:::

:::
:::::: {.column width=60%}

Good habitat followed the glaciers uphill
(SDM by Yi-Ming Weng).

::: {.floatright}
![SDM](figs/nebria/sdm.png){width=100%}
:::

:::
::::::


## Goals

- What was the spatial demographic history of *Nebria* since the LGM?
- Where did the ancestors of today's beetles live in the past?
- Were the ancestors of *N. ingens* and *N. riversi* distinct at the LGM?

::: {.centered}
![SDM](figs/nebria/sdm.png){width=100%}
:::

## Data

::: {.columns}
:::::: {.column width=40%}

Collected by Yi-Ming Weng and Sean Schoville:

- 384 beetles at 27 sites
- low coverage WGS
- some presence/absence data
- expert knowledge about good locations
- a species distribution model

:::
:::::: {.column width=60%}

::: {.floatright}
![](figs/nebria/nebria_sampling.png){width=100%}
:::

:::
::::::

![](figs/nebria/big_sdm.png){width=100%}


## Methods overview

::: {.columns}
:::::: {.column width=40%}

1. Develop an individual-based SLiMulation.

:::
:::::: {.column width=60%}

::: {.floatright}
SLiM screenshot
:::

:::
::::::

## Methods overview

::: {.columns}
:::::: {.column width=40%}

1. Develop an individual-based SLiMulation.
2. Use presence-absence estimates to narrow the range of parameter values
    based on 200 years of simulation.

:::
:::::: {.column width=60%}

::: {.floatright}
SLiM screenshot
:::

:::
::::::



## Methods overview

::: {.columns}
:::::: {.column width=40%}

1. Develop an spatial, individual-based SLiMulation.
2. Do ABC on presence-absence estimates to narrow the range of parameter values
    based on 200 years of simulation.
3. Run simulations since LGM, and do ABC on mean heterozygosity and divergence.

:::
:::::: {.column width=60%}

::: {.floatright}
SLiM screenshot
:::

:::
::::::

## Simulations {data-background-image="figs/nebria/sims1.png" data-background-position="bottom 100px right" data-background-size=100%}

## Simulations {data-background-image="figs/nebria/sims2.png" data-background-position="bottom 100px right" data-background-size=100%}

## Simulations {data-background-image="figs/nebria/sims3.png" data-background-position="bottom 100px right" data-background-size=100%}
