# Data README

### Data Processing Strategies to Determine Maximum Oxygen Uptake: A Systematic Scoping Review and Experimental Comparison

#### Review

The [search](https://github.com/smnnlt/vo2max-processing/tree/master/data/search) folder contains all data related to the initial literature search. It includes the results from [PubMed](https://github.com/smnnlt/vo2max-processing/blob/master/data/search/pubmed.csv), [WOS](https://github.com/smnnlt/vo2max-processing/blob/master/data/search/wos.xls), and additional abstract data as .txt files that could not be automatically retrieved.

The [screen](https://github.com/smnnlt/vo2max-processing/tree/master/data/screen) folder contains all data related to the article screening process. It contains screening results from both abstract and fulltext screening for the two screeners (i1 and i2) and a joined file with the final screening decision. Two .png files repeat the screening instructions as stated in the preregistration and contain the keys for exclusion reasons.

The [extract](https://github.com/smnnlt/vo2max-processing/tree/master/data/extract) folder contains the data extracted from the included articles. For an overview of the data variables, see the preregistration.

The file [fullsample.csv](https://github.com/smnnlt/vo2max-processing/blob/master/data/fullsample.csv) contains an overview of all articles included in the random sample and their exclusion status after screening.

The files [review.Rda](https://github.com/smnnlt/vo2max-processing/blob/master/data/review.Rda), [reporting.Rda](https://github.com/smnnlt/vo2max-processing/blob/master/data/reporting.Rda) and [screen/sample.Rda](https://github.com/smnnlt/vo2max-processing/blob/master/data/screen/sample.Rda) contain intermediate results of the reviewing process, that were saved to allow to skip computationally intensive processes when rerunning the analysis.

#### Experimental Comparison

The folder [ramptests](https://github.com/smnnlt/vo2max-processing/tree/master/data/ramptests) contains the raw breath-by-breath data for all N = 72 exercise tests.

The file [participants.csv](https://github.com/smnnlt/vo2max-processing/blob/master/data/participants.csv) contains information on the N = 72 athletes performing the ramp tests analysed in the study (e.g., body mass).

The file [results.csv](https://github.com/smnnlt/vo2max-processing/blob/master/data/results.csv) contains the results from calculating the V̇O2max using different data processing strategies. Each row corresponds to one strategy, given by the scheme `strategy_interval`, where `bm` corresponds to moving breath, `tm` to moving time, and `tb` to binned time averaging. This file is an intermediate results of the computations in the [comparison analysis script](https://github.com/smnnlt/vo2max-processing/blob/master/scripts/comparison.md).
