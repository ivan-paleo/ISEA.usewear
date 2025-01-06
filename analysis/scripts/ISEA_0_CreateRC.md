Create Research Compendium - ISEA use-wear
================
Ivan Calandra
2025-01-06 15:52:56 CET

- [Goal of the script](#goal-of-the-script)
- [Prerequisites](#prerequisites)
- [Preparations](#preparations)
- [Create the research compendium](#create-the-research-compendium)
  - [Load packages](#load-packages)
  - [Create compendium](#create-compendium)
  - [Create README.qmd file](#create-readmeqmd-file)
  - [Create a folders](#create-a-folders)
  - [Delete file ‘NAMESPACE’](#delete-file-namespace)
- [Before running the analyses](#before-running-the-analyses)
- [After running the analyses](#after-running-the-analyses)
  - [DESCRIPTION](#description)
  - [renv](#renv)
- [sessionInfo()](#sessioninfo)
- [Cite R packages used](#cite-r-packages-used)
  - [References](#references)

------------------------------------------------------------------------

# Goal of the script

Create and set up a research compendium for the paper on ISEA use-wear
using the R package `rrtools`.  
For details on rrtools, see Ben Marwick’s [GitHub
repository](https://github.com/benmarwick/rrtools).

Note that this script is there only to show the steps taken to create
the research compendium and is not part of the analysis per se. For this
reason, most of the code is not evaluated
(`knitr::opts_chunk$set(eval=FALSE)`).

The knit directory for this script is the project directory.

------------------------------------------------------------------------

# Prerequisites

This script requires that you have a GitHub account and that you have
connected RStudio, Git and GitHub. For details on how to do it, check
[Happy Git](https://happygitwithr.com/).

------------------------------------------------------------------------

# Preparations

Before running this script, the first step is to [create a repository on
GitHub and to download it to
RStudio](https://happygitwithr.com/new-github-first.html). In this case,
the repository is called “ISEA.usewear”.  
Finally, open the RStudio project created.

------------------------------------------------------------------------

# Create the research compendium

## Load packages

``` r
library(grateful)
library(renv)
library(rrtools)
library(usethis)
```

## Create compendium

``` r
rrtools::use_compendium(getwd())
```

A new project has opened in a new session.  
Edit the fields “Title”, “Author” and “Description” in the `DESCRIPTION`
file.

## Create README.qmd file

``` r
rrtools::use_readme_qmd()
```

Edit the `README.qmd` file as needed.  
Make sure you render it to create the `README.md` file.

## Create a folders

Create a folder ‘analysis’ and subfolders to contain raw data, derived
data, plots, statistics and scripts. Also create a folder for the Python
analysis:

``` r
dir.create("analysis", showWarnings = FALSE)
dir.create("analysis/raw_data", showWarnings = FALSE)
dir.create("analysis/derived_data", showWarnings = FALSE)
dir.create("analysis/plots", showWarnings = FALSE)
dir.create("analysis/scripts", showWarnings = FALSE)
dir.create("analysis/stats", showWarnings = FALSE)
```

Note that the folders cannot be pushed to GitHub as long as they are
empty.

## Delete file ‘NAMESPACE’

``` r
file.remove("NAMESPACE")
```

------------------------------------------------------------------------

# Before running the analyses

After the creation of this research compendium, I have moved the raw,
input data files to `"~/analysis/raw_data"` (as read-only files) and the
R scripts to `"~/analysis/scripts"`.

------------------------------------------------------------------------

# After running the analyses

## DESCRIPTION

Run this command to add the dependencies to the DESCRIPTION file.

``` r
rrtools::add_dependencies_to_description()
```

## renv

Save the state of the project library using the `renv` package.

``` r
renv::init()
```

------------------------------------------------------------------------

# sessionInfo()

``` r
sessionInfo()
```

    R version 4.4.2 (2024-10-31 ucrt)
    Platform: x86_64-w64-mingw32/x64
    Running under: Windows 10 x64 (build 19045)

    Matrix products: default


    locale:
    [1] LC_COLLATE=English_United States.utf8 
    [2] LC_CTYPE=English_United States.utf8   
    [3] LC_MONETARY=English_United States.utf8
    [4] LC_NUMERIC=C                          
    [5] LC_TIME=English_United States.utf8    

    time zone: Europe/Berlin
    tzcode source: internal

    attached base packages:
    [1] stats     graphics  grDevices datasets  utils     methods   base     

    other attached packages:
    [1] usethis_3.1.0   rrtools_0.1.6   renv_1.0.11     grateful_0.2.10

    loaded via a namespace (and not attached):
     [1] miniUI_0.1.1.1    jsonlite_1.8.9    compiler_4.4.2    crayon_1.5.3     
     [5] promises_1.3.2    Rcpp_1.0.13-1     git2r_0.35.0      later_1.4.1      
     [9] jquerylib_0.1.4   yaml_2.3.10       fastmap_1.2.0     here_1.0.1       
    [13] mime_0.12         R6_2.5.1          knitr_1.49        htmlwidgets_1.6.4
    [17] profvis_0.4.0     rprojroot_2.0.4   shiny_1.10.0      bslib_0.8.0      
    [21] rlang_1.1.4       cachem_1.1.0      httpuv_1.6.15     xfun_0.49        
    [25] fs_1.6.5          sass_0.4.9        pkgload_1.4.0     memoise_2.0.1    
    [29] cli_3.6.3         magrittr_2.0.3    digest_0.6.37     rstudioapi_0.17.1
    [33] xtable_1.8-4      clisymbols_1.2.0  remotes_2.5.0     devtools_2.4.5   
    [37] lifecycle_1.0.4   vctrs_0.6.5       glue_1.8.0        evaluate_1.0.1   
    [41] urlchecker_1.0.1  sessioninfo_1.2.2 pkgbuild_1.4.5    purrr_1.0.2      
    [45] rmarkdown_2.29    tools_4.4.2       ellipsis_0.3.2    htmltools_0.5.8.1

------------------------------------------------------------------------

# Cite R packages used

| Package  | Version      | Citation                             |
|:---------|:-------------|:-------------------------------------|
| base     | 4.4.2        | R Core Team (2024)                   |
| grateful | 0.2.10       | Rodriguez-Sanchez and Jackson (2024) |
| renv     | 1.0.11       | Ushey and Wickham (2024)             |
| rrtools  | 0.1.6        | Marwick (2019)                       |
| usethis  | 3.1.0        | Wickham et al. (2024)                |
| RStudio  | 2024.9.1.394 | Posit team (2024)                    |

## References

<div id="refs" class="references csl-bib-body hanging-indent"
entry-spacing="0">

<div id="ref-rrtools" class="csl-entry">

Marwick, Ben. 2019. *<span class="nocase">rrtools</span>: Creates a
Reproducible Research Compendium*.
<https://github.com/benmarwick/rrtools>.

</div>

<div id="ref-rstudio" class="csl-entry">

Posit team. 2024. *RStudio: Integrated Development Environment for r*.
Boston, MA: Posit Software, PBC. <http://www.posit.co/>.

</div>

<div id="ref-base" class="csl-entry">

R Core Team. 2024. *R: A Language and Environment for Statistical
Computing*. Vienna, Austria: R Foundation for Statistical Computing.
<https://www.R-project.org/>.

</div>

<div id="ref-grateful" class="csl-entry">

Rodriguez-Sanchez, Francisco, and Connor P. Jackson. 2024.
*<span class="nocase">grateful</span>: Facilitate Citation of R
Packages*. <https://pakillo.github.io/grateful/>.

</div>

<div id="ref-renv" class="csl-entry">

Ushey, Kevin, and Hadley Wickham. 2024.
*<span class="nocase">renv</span>: Project Environments*.
<https://CRAN.R-project.org/package=renv>.

</div>

<div id="ref-usethis" class="csl-entry">

Wickham, Hadley, Jennifer Bryan, Malcolm Barrett, and Andy Teucher.
2024. *<span class="nocase">usethis</span>: Automate Package and Project
Setup*. <https://CRAN.R-project.org/package=usethis>.

</div>

</div>
