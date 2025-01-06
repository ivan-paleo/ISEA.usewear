Summary statistics on the surface texture parameters for the ISEA
use-wear project
================
Ivan Calandra
2025-01-06 15:47:35 CET

- [Goal of the script](#goal-of-the-script)
- [Load packages](#load-packages)
- [Read in data](#read-in-data)
- [Summary statistics](#summary-statistics)
  - [Create function to compute the statistics at
    once](#create-function-to-compute-the-statistics-at-once)
  - [Compute summary statistics](#compute-summary-statistics)
  - [Save as XLSX](#save-as-xlsx)
- [sessionInfo()](#sessioninfo)
- [Cite R packages used](#cite-r-packages-used)
  - [References](#references)

------------------------------------------------------------------------

# Goal of the script

This script computes standard descriptive statistics for each group.  
The groups are based on:

- Chert type (A or B)  
- Bamboo species (Bambusa blumeana or Schizostachum lima)  
- Strokes (0 or 2000)

It computes the following statistics:

- sample size (n = `length`)  
- smallest value (`min`)  
- largest value (`max`)
- mean  
- median  
- standard deviation (`sd`)

``` r
dir_in <- "analysis/derived_data"
dir_stats <- "analysis/stats/"
```

Raw data must be located in “./analysis/derived_data”.  
Summary statistics will be saved in “./analysis/stats/”.

The knit directory for this script is the project directory.

------------------------------------------------------------------------

# Load packages

``` r
library(doBy)
library(grateful)
library(knitr)
library(R.utils)
library(rmarkdown)
library(tidyverse)
library(writexl)
```

------------------------------------------------------------------------

# Read in data

``` r
STA <- list.files(dir_in, pattern = "STA\\.Rbin$", full.names = TRUE) %>% 
       loadObject()
str(STA)
```

    'data.frame':   96 obs. of  44 variables:
     $ Sample                  : chr  "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" ...
     $ Chert_type              : chr  "Coarser" "Coarser" "Coarser" "Coarser" ...
     $ Chert_tool              : num  1 1 1 1 1 1 1 1 2 2 ...
     $ Bamboo_sp               : chr  "Bambusa blumeana" "Bambusa blumeana" "Bambusa blumeana" "Bambusa blumeana" ...
     $ Objective               : chr  "20x-0.70" "20x-0.70" "20x-0.70" "20x-0.70" ...
     $ Side                    : chr  "dorsal" "dorsal" "dorsal" "dorsal" ...
     $ Location                : num  1 1 2 2 1 1 2 2 1 1 ...
     $ Strokes                 : Factor w/ 2 levels "0","2000": 1 2 1 2 1 2 1 2 1 2 ...
     $ NMP                     : num  0.2 0.2 0.2 0.2 0.2 ...
     $ Sq                      : num  357 191 472 342 494 ...
     $ Ssk                     : num  0.214 -1.113 0.178 -0.483 0.16 ...
     $ Sku                     : num  3.57 5.75 3.62 3.99 3.54 ...
     $ Sp                      : num  1420 462 1680 972 1866 ...
     $ Sv                      : num  1545 915 1554 1339 1782 ...
     $ Sz                      : num  2965 1377 3234 2311 3648 ...
     $ Sa                      : num  281 138 363 257 384 ...
     $ Smr                     : num  11.76 97.77 8.12 57.82 4.77 ...
     $ Smc                     : num  458 204 611 398 600 ...
     $ Sxp                     : num  635 538 878 825 922 ...
     $ Sal                     : num  4.87 5.21 4.88 5.27 4.55 ...
     $ Str                     : num  0.758 0.569 0.703 0.685 0.381 ...
     $ Std                     : num  12.99 3.49 122.24 176 70.5 ...
     $ Ssw                     : num  0.425 0.425 0.425 0.425 0.425 ...
     $ Sdq                     : num  0.3 0.108 0.368 0.196 0.348 ...
     $ Sdr                     : num  4.214 0.567 6.193 1.829 5.46 ...
     $ Vm                      : num  0.02066 0.00826 0.02791 0.01744 0.03185 ...
     $ Vv                      : num  0.478 0.212 0.639 0.415 0.632 ...
     $ Vmp                     : num  0.02066 0.00826 0.02791 0.01744 0.03185 ...
     $ Vmc                     : num  0.309 0.141 0.388 0.273 0.427 ...
     $ Vvc                     : num  0.442 0.178 0.585 0.362 0.577 ...
     $ Vvv                     : num  0.0363 0.0336 0.0534 0.0529 0.0549 ...
     $ Maximum.depth.of.furrows: num  1943 1100 2275 1627 2427 ...
     $ Mean.depth.of.furrows   : num  618 288 849 578 761 ...
     $ Mean.density.of.furrows : num  4525 3019 4495 3192 4469 ...
     $ First.direction         : num  8.56e-03 1.80e+02 1.35e+02 9.00e+01 7.15e+01 ...
     $ Second.direction        : num  135.0227 44.9859 0.0106 135.0149 63.4786 ...
     $ Third.direction         : num  154 135 90 180 45 ...
     $ Texture.isotropy        : num  71.2 52.6 70.9 68.9 43.3 ...
     $ Asfc                    : num  9.485 0.868 13.86 2.864 11.287 ...
     $ Smfc                    : num  1.12 1.99 1.12 3.77 1.28 ...
     $ HAsfc9                  : num  0.115 0.364 0.081 0.456 0.151 ...
     $ HAsfc81                 : num  0.315 1.338 0.265 0.935 0.335 ...
     $ epLsar                  : num  0.001508 0.00233 0.000946 0.001366 0.003606 ...
     $ NewEplsar               : num  0.0183 0.0187 0.0179 0.0181 0.0174 ...
     - attr(*, "comment")= Named chr [1:36] "%" "nm" "<no unit>" "<no unit>" ...
      ..- attr(*, "names")= chr [1:36] "NMP" "Sq" "Ssk" "Sku" ...

``` r
head(STA)
```

        Sample Chert_type Chert_tool        Bamboo_sp Objective    Side Location
    1 ISEA-EX1    Coarser          1 Bambusa blumeana  20x-0.70  dorsal        1
    2 ISEA-EX1    Coarser          1 Bambusa blumeana  20x-0.70  dorsal        1
    3 ISEA-EX1    Coarser          1 Bambusa blumeana  20x-0.70  dorsal        2
    4 ISEA-EX1    Coarser          1 Bambusa blumeana  20x-0.70  dorsal        2
    5 ISEA-EX1    Coarser          1 Bambusa blumeana  20x-0.70 ventral        1
    6 ISEA-EX1    Coarser          1 Bambusa blumeana  20x-0.70 ventral        1
      Strokes       NMP       Sq        Ssk      Sku        Sp        Sv       Sz
    1       0 0.1999580 357.3100  0.2142039 3.572470 1419.5761 1545.3036 2964.880
    2    2000 0.1999580 191.0983 -1.1130610 5.748940  461.6364  914.8954 1376.532
    3       0 0.1999580 471.9081  0.1782349 3.617312 1680.0238 1554.4562 3234.480
    4    2000 0.1997480 341.9963 -0.4827666 3.993075  971.5410 1339.2310 2310.772
    5       0 0.1997480 494.3200  0.1601631 3.540429 1865.5175 1782.4093 3647.927
    6    2000 0.1995379 750.0697 -0.1556915 3.433134 2298.9824 2423.0967 4722.079
            Sa       Smr      Smc       Sxp      Sal       Str        Std       Ssw
    1 280.9127 11.763569 457.6620  635.0391 4.873127 0.7581494  12.989301 0.4248838
    2 137.9229 97.772232 203.5721  538.2645 5.211629 0.5687682   3.488485 0.4248838
    3 363.2579  8.120664 610.6844  877.8419 4.882320 0.7032365 122.238224 0.4248838
    4 257.0249 57.824111 397.9082  825.3227 5.274232 0.6848816 175.998406 0.4248838
    5 383.6674  4.769773 599.8546  921.5167 4.548254 0.3812570  70.500470 0.4248838
    6 581.3924  4.111890 867.8968 1656.4436 5.219970        NA  73.747646 0.4248838
            Sdq      Sdr          Vm        Vv         Vmp       Vmc       Vvc
    1 0.2997760 4.213511 0.020655078 0.4783037 0.020655078 0.3087765 0.4420467
    2 0.1077379 0.567455 0.008262461 0.2118332 0.008262461 0.1410964 0.1781942
    3 0.3679264 6.192882 0.027908316 0.6385740 0.027908316 0.3880665 0.5852113
    4 0.1962781 1.828874 0.017438165 0.4153293 0.017438165 0.2731001 0.3623892
    5 0.3484251 5.459936 0.031852956 0.6316956 0.031852956 0.4271848 0.5768188
    6 0.3764940 5.751873 0.043690817 0.9116036 0.043690817 0.6312824 0.8100546
             Vvv Maximum.depth.of.furrows Mean.depth.of.furrows
    1 0.03625709                 1942.646              618.3156
    2 0.03363900                 1100.149              288.0931
    3 0.05336270                 2275.050              849.3013
    4 0.05294018                 1627.004              577.8517
    5 0.05487685                 2427.417              760.6523
    6 0.10154900                 3050.083             1078.4730
      Mean.density.of.furrows First.direction Second.direction Third.direction
    1                4525.126    8.557293e-03     135.02266890       153.55462
    2                3018.786    1.799960e+02      44.98590072       134.99670
    3                4494.570    1.350326e+02       0.01062171        89.98775
    4                3191.931    8.999939e+01     135.01492680       179.98965
    5                4469.495    7.146393e+01      63.47858619        44.98322
    6                3565.619    7.152644e+01      63.52035976        89.99952
      Texture.isotropy       Asfc     Smfc     HAsfc9   HAsfc81       epLsar
    1        71.199627  9.4851127 1.124339 0.11542648 0.3151333 0.0015082547
    2        52.594359  0.8677973 1.993686 0.36432069 1.3382342 0.0023296235
    3        70.932914 13.8604793 1.124339 0.08100962 0.2650551 0.0009464416
    4        68.889231  2.8641345 3.767525 0.45631532 0.9349030 0.0013657628
    5        43.275061 11.2865846 1.276960 0.15097810 0.3349801 0.0036062641
    6         9.334622  8.1544882 1.993686 0.41098424 0.8871420 0.0058686109
       NewEplsar
    1 0.01833031
    2 0.01866927
    3 0.01791761
    4 0.01812746
    5 0.01742323
    6 0.01564744

------------------------------------------------------------------------

# Summary statistics

## Create function to compute the statistics at once

``` r
nminmaxmeanmedsd <- function(x){
    y <- x[!is.na(x)]     # Exclude NAs
    n_test <- length(y)   # Sample size (n)
    min_test <- min(y)    # Minimum
    max_test <- max(y)    # Maximum
    mean_test <- mean(y)  # Mean
    med_test <- median(y) # Median
    sd_test <- sd(y)      # Standard deviation
    out <- c(n_test, min_test, max_test, mean_test, med_test, sd_test) # Concatenate
    names(out) <- c("n", "min", "max", "mean", "median", "sd")         # Name values
    return(out)                                                        # Object to return
}
```

## Compute summary statistics

``` r
# Exclude Chert_tool and Location columns from data because they are numeric yet not relevant for stats
STA_sel <- select(STA, !c(Chert_tool, Location))

# Compute summary statistics based on Chert_type and Strokes
stats_chert <- summaryBy(. ~ Chert_type + Strokes, data = STA_sel, FUN = nminmaxmeanmedsd)
stats_chert[1:3]
```

      Chert_type Strokes NMP.n
    1    Coarser       0    24
    2    Coarser    2000    24
    3      Finer       0    24
    4      Finer    2000    24

``` r
# Compute summary statistics based on Bamboo_sp and Strokes
stats_bamboo <- summaryBy(. ~ Bamboo_sp + Strokes, data = STA_sel, FUN = nminmaxmeanmedsd)
stats_bamboo[1:3]
```

               Bamboo_sp Strokes NMP.n
    1   Bambusa blumeana       0    24
    2   Bambusa blumeana    2000    24
    3 Schizostachum lima       0    24
    4 Schizostachum lima    2000    24

``` r
# Compute summary statistics based on Chert_type, Bamboo_sp and Strokes
stats_chert_bamboo <- summaryBy(. ~ Chert_type + Bamboo_sp + Strokes, 
                                data = STA_sel, FUN = nminmaxmeanmedsd)
stats_chert_bamboo[1:4]
```

      Chert_type          Bamboo_sp Strokes NMP.n
    1    Coarser   Bambusa blumeana       0    12
    2    Coarser   Bambusa blumeana    2000    12
    3    Coarser Schizostachum lima       0    12
    4    Coarser Schizostachum lima    2000    12
    5      Finer   Bambusa blumeana       0    12
    6      Finer   Bambusa blumeana    2000    12
    7      Finer Schizostachum lima       0    12
    8      Finer Schizostachum lima    2000    12

## Save as XLSX

``` r
write_xlsx(list("Chert+Strokes" = stats_chert, "Bamboo+Strokes" = stats_bamboo, 
                "Chert+Bamboo+Strokes" = stats_chert_bamboo), 
           path = paste0(dir_stats, "/ISEA_use-wear_STA-stats.xlsx"))
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
    [1] stats     graphics  grDevices utils     datasets  methods   base     

    other attached packages:
     [1] writexl_1.5.1     lubridate_1.9.4   forcats_1.0.0     stringr_1.5.1    
     [5] dplyr_1.1.4       purrr_1.0.2       readr_2.1.5       tidyr_1.3.1      
     [9] tibble_3.2.1      ggplot2_3.5.1     tidyverse_2.0.0   rmarkdown_2.29   
    [13] R.utils_2.12.3    R.oo_1.27.0       R.methodsS3_1.8.2 knitr_1.49       
    [17] grateful_0.2.10   doBy_4.6.24      

    loaded via a namespace (and not attached):
     [1] sass_0.4.9           generics_0.1.3       stringi_1.8.4       
     [4] lattice_0.22-6       hms_1.1.3            digest_0.6.37       
     [7] magrittr_2.0.3       timechange_0.3.0     evaluate_1.0.1      
    [10] grid_4.4.2           fastmap_1.2.0        rprojroot_2.0.4     
    [13] jsonlite_1.8.9       Matrix_1.7-1         backports_1.5.0     
    [16] scales_1.3.0         modelr_0.1.11        microbenchmark_1.5.0
    [19] jquerylib_0.1.4      cli_3.6.3            rlang_1.1.4         
    [22] crayon_1.5.3         cowplot_1.1.3        munsell_0.5.1       
    [25] withr_3.0.2          cachem_1.1.0         yaml_2.3.10         
    [28] tools_4.4.2          tzdb_0.4.0           colorspace_2.1-1    
    [31] boot_1.3-31          Deriv_4.1.6          broom_1.0.7         
    [34] vctrs_0.6.5          R6_2.5.1             lifecycle_1.0.4     
    [37] MASS_7.3-63          pkgconfig_2.0.3      pillar_1.10.0       
    [40] bslib_0.8.0          gtable_0.3.6         glue_1.8.0          
    [43] xfun_0.49            tidyselect_1.2.1     rstudioapi_0.17.1   
    [46] htmltools_0.5.8.1    compiler_4.4.2      

------------------------------------------------------------------------

# Cite R packages used

| Package | Version | Citation |
|:---|:---|:---|
| base | 4.4.2 | R Core Team (2024) |
| doBy | 4.6.24 | Halekoh and Højsgaard (2024) |
| grateful | 0.2.10 | Rodriguez-Sanchez and Jackson (2024) |
| knitr | 1.49 | Xie (2014); Xie (2015); Xie (2024) |
| R.methodsS3 | 1.8.2 | Bengtsson (2003a) |
| R.oo | 1.27.0 | Bengtsson (2003b) |
| R.utils | 2.12.3 | Bengtsson (2023) |
| rmarkdown | 2.29 | Xie, Allaire, and Grolemund (2018); Xie, Dervieux, and Riederer (2020); Allaire et al. (2024) |
| tidyverse | 2.0.0 | Wickham et al. (2019) |
| writexl | 1.5.1 | Ooms (2024) |
| RStudio | 2024.9.1.394 | Posit team (2024) |

## References

<div id="refs" class="references csl-bib-body hanging-indent"
entry-spacing="0">

<div id="ref-rmarkdown2024" class="csl-entry">

Allaire, JJ, Yihui Xie, Christophe Dervieux, Jonathan McPherson, Javier
Luraschi, Kevin Ushey, Aron Atkins, et al. 2024.
*<span class="nocase">rmarkdown</span>: Dynamic Documents for r*.
<https://github.com/rstudio/rmarkdown>.

</div>

<div id="ref-RmethodsS3" class="csl-entry">

Bengtsson, Henrik. 2003a. “The <span class="nocase">R.oo</span>
Package - Object-Oriented Programming with References Using Standard R
Code.” In *Proceedings of the 3rd International Workshop on Distributed
Statistical Computing (DSC 2003)*, edited by Kurt Hornik, Friedrich
Leisch, and Achim Zeileis. Vienna, Austria:
https://www.r-project.org/conferences/DSC-2003/Proceedings/.
<https://www.r-project.org/conferences/DSC-2003/Proceedings/Bengtsson.pdf>.

</div>

<div id="ref-Roo" class="csl-entry">

———. 2003b. “The <span class="nocase">R.oo</span> Package -
Object-Oriented Programming with References Using Standard R Code.” In
*Proceedings of the 3rd International Workshop on Distributed
Statistical Computing (DSC 2003)*, edited by Kurt Hornik, Friedrich
Leisch, and Achim Zeileis. Vienna, Austria:
https://www.r-project.org/conferences/DSC-2003/Proceedings/.
<https://www.r-project.org/conferences/DSC-2003/Proceedings/Bengtsson.pdf>.

</div>

<div id="ref-Rutils" class="csl-entry">

———. 2023. *<span class="nocase">R.utils</span>: Various Programming
Utilities*. <https://CRAN.R-project.org/package=R.utils>.

</div>

<div id="ref-doBy" class="csl-entry">

Halekoh, Ulrich, and Søren Højsgaard. 2024.
*<span class="nocase">doBy</span>: Groupwise Statistics, LSmeans, Linear
Estimates, Utilities*. <https://CRAN.R-project.org/package=doBy>.

</div>

<div id="ref-writexl" class="csl-entry">

Ooms, Jeroen. 2024. *<span class="nocase">writexl</span>: Export Data
Frames to Excel “<span class="nocase">xlsx</span>” Format*.
<https://CRAN.R-project.org/package=writexl>.

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

<div id="ref-tidyverse" class="csl-entry">

Wickham, Hadley, Mara Averick, Jennifer Bryan, Winston Chang, Lucy
D’Agostino McGowan, Romain François, Garrett Grolemund, et al. 2019.
“Welcome to the <span class="nocase">tidyverse</span>.” *Journal of Open
Source Software* 4 (43): 1686. <https://doi.org/10.21105/joss.01686>.

</div>

<div id="ref-knitr2014" class="csl-entry">

Xie, Yihui. 2014. “<span class="nocase">knitr</span>: A Comprehensive
Tool for Reproducible Research in R.” In *Implementing Reproducible
Computational Research*, edited by Victoria Stodden, Friedrich Leisch,
and Roger D. Peng. Chapman; Hall/CRC.

</div>

<div id="ref-knitr2015" class="csl-entry">

———. 2015. *Dynamic Documents with R and Knitr*. 2nd ed. Boca Raton,
Florida: Chapman; Hall/CRC. <https://yihui.org/knitr/>.

</div>

<div id="ref-knitr2024" class="csl-entry">

———. 2024. *<span class="nocase">knitr</span>: A General-Purpose Package
for Dynamic Report Generation in r*. <https://yihui.org/knitr/>.

</div>

<div id="ref-rmarkdown2018" class="csl-entry">

Xie, Yihui, J. J. Allaire, and Garrett Grolemund. 2018. *R Markdown: The
Definitive Guide*. Boca Raton, Florida: Chapman; Hall/CRC.
<https://bookdown.org/yihui/rmarkdown>.

</div>

<div id="ref-rmarkdown2020" class="csl-entry">

Xie, Yihui, Christophe Dervieux, and Emily Riederer. 2020. *R Markdown
Cookbook*. Boca Raton, Florida: Chapman; Hall/CRC.
<https://bookdown.org/yihui/rmarkdown-cookbook>.

</div>

</div>
