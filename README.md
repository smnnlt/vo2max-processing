# Data Processing Strategies to Determine Maximum Oxygen Uptake: A Systematic Scoping Review and Experimental Comparison

A sport science project.

Read the preregistration on [OSF](https://osf.io/3am4s).

### Background & Aim

Raw breath-by-breath data is noisy and requires post-processing. Different processing strategies can influence outcome variables, such as the maximum oxygen uptake (V̇O2max). This project provides a systematic mapping of current processing practices and their reporting in the scientific literature. It additionally compares the most common practices on the basis of N = 72 standardized exercise tests.

### Repository Structure

The [scripts](https://github.com/smnnlt/vo2max-processing/tree/master/scripts) folder contains the full analysis scripts for both the scoping [review](https://github.com/smnnlt/vo2max-processing/blob/master/scripts/review.md) and the experimental [comparison](https://github.com/smnnlt/vo2max-processing/blob/master/scripts/comparison.md) of processing strategies.

The [data](https://github.com/smnnlt/vo2max-processing/tree/master/data) folder contains all data from the review (e.g., search results, screening results, extracted data) and from the experiments (raw breath-by-breath data). See the [data-README](https://github.com/smnnlt/vo2max-processing/blob/master/data/README.md) for more details.

The [plot](https://github.com/smnnlt/vo2max-processing/tree/master/plots) folder contains all images generated during the analysis.

The [preregistration](https://github.com/smnnlt/vo2max-processing/tree/master/preregistration) folder lists all documents of the preregistration (also published on the [OSF](https://osf.io/3am4s)).

The [thesis](https://github.com/smnnlt/vo2max-processing/tree/master/thesis) folder contains the bachelor thesis ([PDF](https://github.com/smnnlt/vo2max-processing/blob/master/thesis/thesis.pdf)) related to this project and its source code.

You can rerun the analyses of this project using R and Quarto. Simply download the repository, open it in R, and run the command `renv::restore()` to install all necessary packages.

### License

This project uses a [CC-BY 4.0](http://creativecommons.org/licenses/by/4.0/) license, which also applies to all data published in this repository. All code is additionally licensed under an [MIT license](https://github.com/smnnlt/vo2max-processing/blob/master/LICENSE.md). The thesis document is released under a [CC-BY-NC-ND](https://creativecommons.org/licenses/by-nc-nd/4.0/) license.
