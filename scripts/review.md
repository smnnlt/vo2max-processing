
# Analysis Script for the Systematic Review

## Project: ‘Data Processing Strategies to Determine Maximum Oxygen Uptake: A Systematic Scoping Review and Experimental Comparison’

This script closely follows the preregistration, which is uploaded on
the [OSF](https://osf.io/3am4s).

``` r
knitr::opts_chunk$set(out.width = "70%", fig.align = "center")

# Packages used for the data workflow
library(readxl)
library(rentrez)
library(XML)
library(purrr)
library(dplyr)
library(ggplot2)
library(MetBrewer)
library(treemapify)
library(colorspace)
```

#### Get search data

The search was conducted on the 16th March 2022 at around 10 pm CET.

The Pubmed results are saved in the file ‘pubmed.csv’; the Web of
Science results are saved in the file ‘wos.xls’.

``` r
# file name for PubMed results
pm_file <- "../data/search/pubmed.csv"

# file name for merged WoS results
wos_file <- "../data/search/wos.xls"

# read PubMed data
pm_data_pre <- read.csv(pm_file)
pm_data <- data.frame(
  pmid = pm_data_pre[, 1],
  title = pm_data_pre$Title,
  authors = pm_data_pre$Authors,
  journal = pm_data_pre$Journal.Book,
  year = pm_data_pre$Publication.Year,
  doi = pm_data_pre$DOI,
  abstract = NA
)
# read Web of Science data
wos_data_pre <- readxl::read_xls(wos_file)
wos_data <- data.frame(
  pmid = wos_data_pre$`Pubmed Id`,
  title = wos_data_pre$`Article Title`,
  authors = wos_data_pre$`Author Full Names`,
  journal = wos_data_pre$`Source Title`,
  year = wos_data_pre$`Publication Year`,
  doi = wos_data_pre$DOI,
  abstract = wos_data_pre$Abstract
)

# count PubMed results
pm_count <- nrow(pm_data)
# count Web of Science results
wos_count <- nrow(wos_data)
overall_count <- pm_count + wos_count
```

The search yielded 7929 results (Pubmed: 3969; WOS: 3960)

#### Removal of duplicates

``` r
# Merge data frame
merge_data <- rbind(wos_data, pm_data)

# Filter entries without doi
no_doi <- is.na(merge_data$doi) | (merge_data$doi == "")
merge_data <- merge_data[!no_doi, ]

# Count removal from missing doi
no_doi_count <- sum(no_doi)

# Find duplicated DOIs
dupl <- duplicated(merge_data$doi)
# Filter duplicates
merge_data <- merge_data[!dupl, ]

# Count removal of duplicates
dupl_count <- sum(dupl)
```

Removed because of missing DOI: 352

Duplicates removed: 2935

Remaining: 4642

#### Automated title filtering

``` r
# Filter titles by terms
terms <- paste0("review|comment|correction|retraction|meta-analysis",
                "|editorial|erratum|reply")
excl <- grepl(terms, merge_data$title, ignore.case = TRUE)
filtered_data <- merge_data[!excl, ]

# Count removal by title filtering
title_excl_count <- sum(excl)
# count remaining articles
forsample_count <- nrow(filtered_data)
```

Removed by automated title filtering: 278

Remaining articles for random sampling: 4364

#### Random Sampling

``` r
# Assign ids to articles left
filtered_data$id <- seq_len(nrow(filtered_data))

# randomly sample 500 articles
set.seed(4711)
smpl <- sample(filtered_data$id, size = 500)

sampled_data <- filtered_data[smpl, ]
# assign sample id
sampled_data$sid <- seq_len(nrow(sampled_data))

# get abstract from PubMed where missing

no_abstract <- is.na(sampled_data$abstract)

no_abstract_pmid <- sampled_data$pmid[no_abstract]

# function to scrape abstract from the PubMed API
get_abstract <- function(pmid) {
  if (is.na(pmid))
    return(NA)
  xml_data <- rentrez::entrez_fetch(
    db = "pubmed", 
    id = pmid, 
    rettype = "xml", 
    parse = TRUE
  )
  text <- XML::xmlValue(XML::getNodeSet(xml_data, "//AbstractText"))
  out <- paste(text, collapse = "///")
  if (out == "") out <- NA
  out
}

# retrieve missing abstracts
sampled_data$abstract[no_abstract] <- purrr::map_chr(
  .x = no_abstract_pmid, 
  .f = get_abstract
)

# manually add abstract for articles, where automated abstract retrieval did not
# work. In these cases no blinding applies to the principal investigator
which(is.na(sampled_data$abstract))

sampled_data$abstract[50] <- readtext::readtext("../data/search/a50.txt")$text
sampled_data$abstract[238] <- readtext::readtext("../data/search/a238.txt")$text
sampled_data$abstract[288] <- readtext::readtext("../data/search/a288.txt")$text
sampled_data$abstract[416] <- readtext::readtext("../data/search/a416.txt")$text
sampled_data$abstract[488] <- readtext::readtext("../data/search/a488.txt")$text
sampled_data$abstract[490] <- readtext::readtext("../data/search/a490.txt")$text
sampled_data$abstract[500] <- readtext::readtext("../data/search/a500.txt")$text

# later plotting throws an error for some articles due to unrecognized html-tags
# in the abstracts. These abstract are replaced by their text equivalents
sampled_data$abstract[344] <- readtext::readtext("../data/search/a344.txt")$text
sampled_data$abstract[356] <- readtext::readtext("../data/search/a356.txt")$text

# save(sampled_data, file = "../data/screen/sample.Rda")
```

#### Prepare Title-Abstract plots for screening

``` r
load("../data/screen/sample.Rda")

# Function for creating plots with blinded abstract data
create_abstract_plot <- function(row, data) {
  # get random colour
  # different colours in plots are used to stay concentrated when scanning
  # the abstracts
  clrs <- c(MetBrewer::met.brewer("Moreau")[-4])
  clr <- sample(clrs, 1)
  
  g <- ggplot(data = NULL, aes(x = 0, y = 1)) +
  ggtext::geom_textbox(
    width = 1, 
    box.colour = "transparent", 
    label =  gsub(">", "", data$abstract[row]),
    colour = clr
  ) +
  labs(
    title = paste0(strwrap(data$title[row], 80), collapse = "\n"), 
    subtitle = paste0("Sampling ID: ", sprintf("%03i", data$sid[row])),
    caption = paste0("ID: ", data$id[row], "; PMID = ", data$pmid[row])
    ) +
  theme_void()
  
  ggsave(
    filename = paste0("../plots/abstracts/", sprintf("%03i", data$sid[row]), ".png"), 
    plot = g, 
    dpi = 300, width = 7, height = 5, bg = "white"
  )
}


# Create blinded abstract plots
# purrr::walk(
#   .x = seq_len(nrow(sampled_data)),
#   .f = create_abstract_plot, 
#   data = sampled_data
# )
```

#### Compare abstract screening results

``` r
load("../data/screen/sample.Rda")
# read screening results
s_res1 <- read.csv("../data/screen/abstract_screening_i1.csv")[, 2]
s_res2 <- read.csv("../data/screen/abstract_screening_i2.csv")[, 2]

# compare results
comp_res <- function(x1, x2) {
  if (is.na(x1)) x1c <- 0 else x1c <- x1
  if (is.na(x2)) x2c <- 0 else x2c <- x2
  if (x1c == x2c) out <- x1 else out <- "XXX"
  out
}

s_res <- purrr::map2_chr(s_res1, s_res2, comp_res)
s_res_df <- data.frame(
  id = 1:500,
  i1 = s_res1,
  i2 = s_res2,
  result = s_res,
  decision = rep.int("", 500),
  doi = sampled_data$doi
)
# write.csv(s_res_df, "../data/screen/abstract_screening.csv")

# Calculate relative agreement
1 - (sum(s_res == "XXX", na.rm = TRUE) / length(s_res))
```

    ## [1] 0.902

``` r
# Get final decisions
s_res_final <- read.csv("../data/screen/abstract_screening.csv")
# number excluded 
abstr_excl_count <- sum(!is.na(s_res_final$decision))
# reasons for exclusion
table(s_res_final$decision)
```

    ## 
    ##  h  l  m  r 
    ## 17  2 64 36

``` r
# extract remaining data
ft_screen <- sampled_data[is.na(s_res_final$decision), ]

# write file for full text screening
ft_screen_df <- data.frame(
  sid = ft_screen$sid,
  fulltext_exclusion = rep.int("", nrow(ft_screen)),
  exclusion_note = rep.int("", nrow(ft_screen)),
  doi = ft_screen$doi
)

# write.csv(ft_screen_df, file = "../data/screen/fulltext_screening_i1.csv", row.names = FALSE)
```

Removed by abstract screening: 119

#### Compare full text screening results

``` r
# read screening results
f_res_i2 <- read.csv("../data/screen/fulltext_screening_i2.csv")
f_res_i1 <- read.csv("../data/screen/fulltext_screening_i1.csv", sep = ";")

f_res1 <- f_res_i1$fulltext_exclusion
f_res2 <- f_res_i2$fulltext_exclusion

f_res <- purrr::map2_chr(f_res1, f_res2, comp_res)
f_res_df <- data.frame(
  sid = f_res_i2$sid,
  i1 = f_res1,
  i2 = f_res2,
  result = f_res,
  decision = rep.int("", length(f_res1)),
  i1_note = f_res_i1$exclusion_note,
  i2_note = f_res_i2$exclusion_note,
  doi = f_res_i1$doi
)

# write.csv(f_res_df, "../data/screen/fulltext_screening.csv", row.names = FALSE)

# Calculate relative agreement
1 - (sum(f_res == "XXX", na.rm = TRUE) / length(f_res))
```

    ## [1] 0.7585302

``` r
1 - ((sum(is.na(f_res1) | is.na(f_res2)) - sum(is.na(f_res))) / length(f_res))
```

    ## [1] 0.816273

``` r
# read final decision
ft_res <- read.csv("../data/screen/fulltext_screening.csv")
ft_res$decision <- factor(
  ft_res$decision, 
  levels = c("t", "l", "r", "h", "m", "c", "e", "d", "o"),
  labels = c("no fulltext", "no english full-text", "no original research", 
             "no humans", "VO2 not measured", "VO2 not continously measured", 
             "no exhaustion", "long protocol", "other")
)

# count exclusions
ft_excl_count <- sum(!is.na(ft_res$decision))

# count remaining articles for data extraction
final_count <- sum(is.na(ft_res$decision))

table(ft_res$decision, useNA = "ifany")
```

    ## 
    ##                  no fulltext         no english full-text 
    ##                           15                            7 
    ##         no original research                    no humans 
    ##                           11                            0 
    ##             VO2 not measured VO2 not continously measured 
    ##                           41                            3 
    ##                no exhaustion                long protocol 
    ##                           12                           18 
    ##                        other                         <NA> 
    ##                           32                          242

Excluded after full text screening: 139

Remaining articles: 242

#### Prepare data extraction

``` r
emp <- rep.int("", final_count)

extr <- data.frame(
  sid = ft_res$sid[is.na(ft_res$decision)],
  cart = emp,
  type = emp,
  outcome = emp,
  pre = emp,
  software = emp,
  interpolation = emp,
  ptype = emp,
  averaging = emp,
  parameters = emp,
  source = emp,
  doi = ft_res$doi[is.na(ft_res$decision)]
)

# write.csv(extr, "../data/extract/extraction.csv", row.names = FALSE)
```

#### Read and clean extracted data

``` r
ext <- read.csv("../data/extract/extraction.csv")

# Create column for processing type + averaging type combination
ext$proc_combination <- paste(ext$ptype, ext$averaging, sep = "-")
ext$proc_combination[ext$proc_combination == "NA-NA"] <- NA

# Separate parameter column for multiple parameters (i.e. multiple binned averages)
# and calculate total duration

split_parameters <- function(n) {
  value <- ext$parameters[n]
  if (grepl("x", value)) {
    value_split <- strsplit(value, "x")[[1]]
    out <- data.frame(
      param_times = as.numeric(value_split[1]), 
      param_value = as.numeric(value_split[2])
    )
    out$param_duration <- out$param_times * out$param_value
  } else {
    out <- data.frame(
      param_times = NA, 
      param_value = as.numeric(value), 
      param_duration = as.numeric(value)
    )
  }
  out
}

ext <- cbind(ext, purrr::map_dfr(seq_len(nrow(ext)), split_parameters))

# save data
# save(ext, file = "../data/review.Rda")

# Show metabolic carts used
table(ext$type, useNA = "ifany")
```

    ## 
    ##    bbb mixing   <NA> 
    ##    160     45     37

#### Analyze extracted data

``` r
load("../data/review.Rda")

n_total <- nrow(ext)
ext_bbb <- ext[ext$type == "bbb" & !is.na(ext$type),]
n_bbb <- nrow(ext_bbb)

# Type of metabolic cart
p_cart <- 1 - (sum(grepl("NA", ext$cart) | is.na(ext$cart)) / n_total)

# Preprocessing
p_pre <- sum(!is.na(ext_bbb$pre)) / n_bbb

# Software
p_software <- sum(!is.na(ext_bbb$software)) / n_bbb

# Processing strategy
p_proc <- sum(!is.na(ext$ptype) & !is.na(ext$averaging) & !is.na(ext$parameters)) / n_total

# Source
p_source <- sum(!is.na(ext$source)) / n_total

reporting <- data.frame(
  "cart" = p_cart,
  "pre" = p_pre,
  "software" = p_software,
  "processing" = p_proc,
  "source" = p_source
)

# save(reporting, file = "../data/reporting.Rda")

knitr::kable(purrr::map_dfr(reporting, scales::percent, accuracy = 0.1))
```

| cart  | pre  | software | processing | source |
|:------|:-----|:---------|:-----------|:-------|
| 88.0% | 5.6% | 15.0%    | 55.8%      | 5.8%   |

#### Plot: Calculation intervals

``` r
# Show data processing strategies overall
table(ext$ptype, useNA = "ifany")
```

    ## 
    ## breath   time   <NA> 
    ##      7    128    107

``` r
# Show data processing type for breath-by-breath measurements only
table(ext$proc_combination[ext$type == "bbb"], useNA = "ifany")
```

    ## 
    ## breath-binned breath-moving   time-binned  time-mbinned   time-moving 
    ##             2             5            70             5             6 
    ##          <NA> 
    ##           109

``` r
# Plot for total calculation interval lengths
p_int <- ext[ext$ptype == "time" & !is.na(ext$ptype), ] |>
  ggplot(aes(y = param_duration)) +
    geom_bar() +
  geom_text(aes(label = ..count..), stat = "count", hjust = 1.2, colour = "white") +
    scale_y_reverse(
      breaks = c(5, 10, 15, 20, 30, 45, 60),
      minor_breaks = NULL,
      name = "Duration (s)"
    ) +
    scale_x_continuous(
      minor_breaks = NULL,
      name = "Count",
      breaks = c(0, 10, 20, 30, 40),
      labels = NULL,
      expand = c(0, 0),
      position = "top"
    ) +
    theme_minimal(13) +
   theme(
     panel.grid.major.y = element_blank(),
     axis.ticks.y = element_line(colour = "grey", size = 0.5)
    )

# ggsave("../plots/duration_count.png", plot = p_int, width = 5, height = 3, dpi = 600, bg = "white")

knitr::include_graphics("../plots/duration_count.png")
```

<img src="../plots/duration_count.png" width="70%" style="display: block; margin: auto;" />

#### Plot: Processing strategies

``` r
ext_count <- ext |> 
  filter(type == "bbb") |>
  select(c(proc_combination, ptype)) |>
  filter(!is.na(proc_combination)) |>
  count(proc_combination, ptype)

ext_count$proc_combination <- factor(
  ext_count$proc_combination, 
  levels = c("breath-binned", "time-moving", "breath-moving", "time-binned", "time-mbinned"),
  labels = c("Binned Breath","Moving Time", "Moving Breath", "Binned Time", "Multiple Binned Time"))

percs <- scales::percent(ext_count$n / sum(ext_count$n))
ext_count$label <- paste0(ext_count$proc_combination, " (", percs, ")")
ext_count$subgroup <- c(1, 1, 2, 3, 4)

p_strat <- ggplot(ext_count, aes(area = n, fill = proc_combination, label = label, subgroup = subgroup)) +
  geom_treemap(show.legend = FALSE, start = "topleft") +
  geom_treemap_text(
    color = "black",
    reflow = TRUE, 
    place = "centre", 
    start = "topleft",
    padding.y = grid::unit(1.5, "mm")) +
  scale_fill_manual(
    values = colorspace::lighten(c("#663171", "#ea7428", "#653151", "#0c7156", "#0c7182"), amount = 0.2)
  )

# ggsave("../plots/strategies.png", plot = p_strat, width = 5, height = 3, dpi = 600, bg = "white")

knitr::include_graphics("../plots/strategies.png")
```

<img src="../plots/strategies.png" width="70%" style="display: block; margin: auto;" />
