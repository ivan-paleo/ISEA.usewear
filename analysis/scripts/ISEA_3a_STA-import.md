Import dataset from the surface texture analysis for the ISEA use-wear
project
================
Ivan Calandra
2024-06-24 14:02:08 CEST

- [Goal of the script](#goal-of-the-script)
- [Load packages](#load-packages)
- [Read and format data](#read-and-format-data)
  - [Read in CSV file](#read-in-csv-file)
  - [Select relevant columns and
    rows](#select-relevant-columns-and-rows)
  - [Identify results using frame
    numbers](#identify-results-using-frame-numbers)
  - [Add headers](#add-headers)
  - [Extract units](#extract-units)
  - [Split column ‘Name’](#split-column-name)
  - [Split column ‘Specimen’](#split-column-specimen)
  - [Add columns with sample ID and bamboo
    species](#add-columns-with-sample-id-and-bamboo-species)
  - [Keep only numerical parts in columns “Location” and
    “Strokes”](#keep-only-numerical-parts-in-columns-location-and-strokes)
  - [Convert all parameter variables to
    numeric](#convert-all-parameter-variables-to-numeric)
  - [NMP ratios](#nmp-ratios)
  - [Re-order columns and add units as
    comment](#re-order-columns-and-add-units-as-comment)
  - [Check the result](#check-the-result)
- [Save data](#save-data)
  - [As XLSX](#as-xlsx)
  - [As Rbin](#as-rbin)
- [sessionInfo()](#sessioninfo)
- [Cite R packages used](#cite-r-packages-used)
  - [References](#references)

------------------------------------------------------------------------

# Goal of the script

This script formats the output of the resulting files from applying
surface texture analysis to a sample of experimental lithics. The script
will:

1.  Read in the original files  
2.  Format the data  
3.  Write XLSX file and save R objects ready for further analysis in R

``` r
dir_in  <- "analysis/raw_data"
dir_out <- "analysis/derived_data"
```

Raw data must be located in “./analysis/raw_data”.  
Formatted data will be saved in “./analysis/derived_data”.

The knit directory for this script is the project directory.

------------------------------------------------------------------------

# Load packages

``` r
library(grateful)
library(knitr)
library(R.utils)
library(rmarkdown)
library(tidyverse)
library(writexl)
```

------------------------------------------------------------------------

# Read and format data

## Read in CSV file

``` r
# Loop through list of CSV files and read in
STA <- list.files(dir_in, pattern = "STA\\.csv$", full.names = TRUE) %>% 
       read.csv(header = FALSE, na.strings = "*****", fileEncoding = 'WINDOWS-1252')
```

## Select relevant columns and rows

``` r
STA_keep_col  <- c(3, 11:46)                               # Define columns to keep
STA_keep_rows <- which(STA[[1]] != "#")                    # Define rows to keep
STA_keep      <- STA[STA_keep_rows, STA_keep_col] # Subset rows and columns
```

## Identify results using frame numbers

``` r
frames <- as.numeric(unlist(STA[1, STA_keep_col]))
```

    Warning: NAs introduced by coercion

``` r
ID <- which(frames == 26)
ISO <- which(frames == 28)
furrow <- which(frames == 29)
diriso <- which(frames %in% 30:31)
SSFA <- which(frames %in% 32:33)
```

## Add headers

``` r
# Get headers from 2nd row
colnames(STA_keep) <- STA[2, STA_keep_col] %>% 
  
                      # Convert to valid names
                      make.names() %>% 
  
                      # Delete repeated periods
                      gsub("\\.+", "\\.", x = .) %>% 
  
                      # Delete periods at the end of the names
                      gsub("\\.$", "", x = .)
  
# Keep parameter name before the first period for ISO
colnames(STA_keep)[ISO] <- strsplit(names(STA_keep)[ISO], ".", fixed = TRUE) %>% 
                           sapply(`[[`, 1)

# Keep parameter name after the last period for SSFA
colnames(STA_keep)[SSFA] <- gsub("^([A-Za-z0-9]+\\.)+", "", colnames(STA_keep)[SSFA])

# Edit header non-measured point ratios
colnames(STA_keep)[ID] <- "NMP"

# Edit header for name of surfaces
colnames(STA_keep)[1] <- "Name"

# Check results
str(STA_keep)
```

    'data.frame':   96 obs. of  37 variables:
     $ Name                    : chr  "D:\\Data\\Riczar\\10_Mountains-analysis\\ConfoMap_shift-extract files\\ISEA-2_STA --- ISEA-Chert-A1_LSM_20x-0.7"| __truncated__ "D:\\Data\\Riczar\\10_Mountains-analysis\\ConfoMap_shift-extract files\\ISEA-2_STA --- ISEA-Chert-A1_LSM_20x-0.7"| __truncated__ "D:\\Data\\Riczar\\10_Mountains-analysis\\ConfoMap_shift-extract files\\ISEA-2_STA --- ISEA-Chert-A1_LSM_20x-0.7"| __truncated__ "D:\\Data\\Riczar\\10_Mountains-analysis\\ConfoMap_shift-extract files\\ISEA-2_STA --- ISEA-Chert-A1_LSM_20x-0.7"| __truncated__ ...
     $ NMP                     : chr  "0.199957992" "0.199957992" "0.199957992" "0.1997479521" ...
     $ Sq                      : chr  "357.3100043" "191.098347" "471.9081352" "341.9963087" ...
     $ Ssk                     : chr  "0.2142039482" "-1.113060972" "0.1782349483" "-0.4827665534" ...
     $ Sku                     : chr  "3.572470153" "5.748939883" "3.617311939" "3.993074559" ...
     $ Sp                      : chr  "1419.576102" "461.636368" "1680.023771" "971.5409887" ...
     $ Sv                      : chr  "1545.303564" "914.8954458" "1554.456194" "1339.231019" ...
     $ Sz                      : chr  "2964.879665" "1376.531814" "3234.479964" "2310.772008" ...
     $ Sa                      : chr  "280.9126963" "137.9228996" "363.2579132" "257.0249141" ...
     $ Smr                     : chr  "11.76356891" "97.77223198" "8.120663738" "57.82411091" ...
     $ Smc                     : chr  "457.6619638" "203.5721089" "610.6843806" "397.9081922" ...
     $ Sxp                     : chr  "635.0390839" "538.2645053" "877.8419055" "825.3226558" ...
     $ Sal                     : chr  "4.873127241" "5.211629034" "4.882320124" "5.274231682" ...
     $ Str                     : chr  "0.7581493861" "0.5687682374" "0.7032365061" "0.6848816212" ...
     $ Std                     : chr  "12.98930067" "3.488484842" "122.2382235" "175.9984065" ...
     $ Ssw                     : chr  "0.4248838084" "0.4248838084" "0.4248838084" "0.4248838084" ...
     $ Sdq                     : chr  "0.2997759573" "0.1077379432" "0.3679264124" "0.1962780616" ...
     $ Sdr                     : chr  "4.213510718" "0.5674549774" "6.192882483" "1.82887364" ...
     $ Vm                      : chr  "0.02065507767" "0.008262461435" "0.02790831635" "0.01743816527" ...
     $ Vv                      : chr  "0.4783037437" "0.2118331583" "0.638574011" "0.4153293317" ...
     $ Vmp                     : chr  "0.02065507767" "0.008262461435" "0.02790831635" "0.01743816527" ...
     $ Vmc                     : chr  "0.3087764946" "0.1410963832" "0.388066503" "0.2731001371" ...
     $ Vvc                     : chr  "0.442046657" "0.1781941588" "0.5852113083" "0.3623891563" ...
     $ Vvv                     : chr  "0.03625708677" "0.03363899956" "0.05336270273" "0.0529401754" ...
     $ Maximum.depth.of.furrows: chr  "1942.645932" "1100.149344" "2275.049793" "1627.003742" ...
     $ Mean.depth.of.furrows   : chr  "618.3156029" "288.0930895" "849.3012724" "577.8516544" ...
     $ Mean.density.of.furrows : chr  "4525.126446" "3018.786478" "4494.570357" "3191.931152" ...
     $ First.direction         : chr  "0.008557293329" "179.9959727" "135.032573" "89.99939109" ...
     $ Second.direction        : chr  "135.0226689" "44.98590072" "0.01062171041" "135.0149268" ...
     $ Third.direction         : chr  "153.5546248" "134.9966958" "89.9877501" "179.9896475" ...
     $ Texture.isotropy        : chr  "71.19962688" "52.59435919" "70.93291413" "68.88923084" ...
     $ Asfc                    : chr  "9.485112657" "0.8677972883" "13.86047929" "2.864134488" ...
     $ Smfc                    : chr  "1.124339164" "1.993685999" "1.124339164" "3.767524759" ...
     $ HAsfc9                  : chr  "0.1154264841" "0.3643206898" "0.08100961788" "0.4563153235" ...
     $ HAsfc81                 : chr  "0.3151333336" "1.338234209" "0.2650550944" "0.9349029537" ...
     $ epLsar                  : chr  "0.001508254735" "0.002329623452" "0.0009464415845" "0.001365762769" ...
     $ NewEplsar               : chr  "0.01833031405" "0.01866926505" "0.01791760554" "0.01812746064" ...

``` r
head(STA_keep)
```

                                                                                                                                                             Name
    4     D:\\Data\\Riczar\\10_Mountains-analysis\\ConfoMap_shift-extract files\\ISEA-2_STA --- ISEA-Chert-A1_LSM_20x-0.70_dorsal_loc1_shift-extract_0strokes.mnt
    5  D:\\Data\\Riczar\\10_Mountains-analysis\\ConfoMap_shift-extract files\\ISEA-2_STA --- ISEA-Chert-A1_LSM_20x-0.70_dorsal_loc1_shift-extract_2000strokes.mnt
    6     D:\\Data\\Riczar\\10_Mountains-analysis\\ConfoMap_shift-extract files\\ISEA-2_STA --- ISEA-Chert-A1_LSM_20x-0.70_dorsal_loc2_shift-extract_0strokes.mnt
    7  D:\\Data\\Riczar\\10_Mountains-analysis\\ConfoMap_shift-extract files\\ISEA-2_STA --- ISEA-Chert-A1_LSM_20x-0.70_dorsal_loc2_shift-extract_2000strokes.mnt
    8    D:\\Data\\Riczar\\10_Mountains-analysis\\ConfoMap_shift-extract files\\ISEA-2_STA --- ISEA-Chert-A1_LSM_20x-0.70_ventral_loc1_shift-extract_0strokes.mnt
    9 D:\\Data\\Riczar\\10_Mountains-analysis\\ConfoMap_shift-extract files\\ISEA-2_STA --- ISEA-Chert-A1_LSM_20x-0.70_ventral_loc1_shift-extract_2000strokes.mnt
               NMP          Sq           Ssk         Sku          Sp          Sv
    4  0.199957992 357.3100043  0.2142039482 3.572470153 1419.576102 1545.303564
    5  0.199957992  191.098347  -1.113060972 5.748939883  461.636368 914.8954458
    6  0.199957992 471.9081352  0.1782349483 3.617311939 1680.023771 1554.456194
    7 0.1997479521 341.9963087 -0.4827665534 3.993074559 971.5409887 1339.231019
    8 0.1997479521 494.3200158  0.1601630524 3.540428899  1865.51752  1782.40934
    9 0.1995379122 750.0697139 -0.1556914727 3.433134179  2298.98245 2423.096683
               Sz          Sa         Smr         Smc         Sxp         Sal
    4 2964.879665 280.9126963 11.76356891 457.6619638 635.0390839 4.873127241
    5 1376.531814 137.9228996 97.77223198 203.5721089 538.2645053 5.211629034
    6 3234.479964 363.2579132 8.120663738 610.6843806 877.8419055 4.882320124
    7 2310.772008 257.0249141 57.82411091 397.9081922 825.3226558 5.274231682
    8 3647.926859 383.6674209 4.769772956 599.8546361 921.5166666 4.548254255
    9 4722.079133 581.3923948 4.111890355 867.8968027 1656.443623 5.219969938
               Str         Std          Ssw          Sdq          Sdr
    4 0.7581493861 12.98930067 0.4248838084 0.2997759573  4.213510718
    5 0.5687682374 3.488484842 0.4248838084 0.1077379432 0.5674549774
    6 0.7032365061 122.2382235 0.4248838084 0.3679264124  6.192882483
    7 0.6848816212 175.9984065 0.4248838084 0.1962780616   1.82887364
    8 0.3812569971 70.50046989 0.4248838084 0.3484251452  5.459935798
    9         <NA> 73.74764643 0.4248838084 0.3764939562  5.751872939
                  Vm           Vv            Vmp          Vmc          Vvc
    4  0.02065507767 0.4783037437  0.02065507767 0.3087764946  0.442046657
    5 0.008262461435 0.2118331583 0.008262461435 0.1410963832 0.1781941588
    6  0.02790831635  0.638574011  0.02790831635  0.388066503 0.5852113083
    7  0.01743816527 0.4153293317  0.01743816527 0.2731001371 0.3623891563
    8  0.03185295598 0.6316956006  0.03185295598 0.4271847979 0.5768187556
    9  0.04369081706 0.9116035875  0.04369081706 0.6312823917 0.8100545875
                Vvv Maximum.depth.of.furrows Mean.depth.of.furrows
    4 0.03625708677              1942.645932           618.3156029
    5 0.03363899956              1100.149344           288.0930895
    6 0.05336270273              2275.049793           849.3012724
    7  0.0529401754              1627.003742           577.8516544
    8 0.05487684501              2427.416503            760.652286
    9      0.101549              3050.083105           1078.473001
      Mean.density.of.furrows First.direction Second.direction Third.direction
    4             4525.126446  0.008557293329      135.0226689     153.5546248
    5             3018.786478     179.9959727      44.98590072     134.9966958
    6             4494.570357      135.032573    0.01062171041      89.9877501
    7             3191.931152     89.99939109      135.0149268     179.9896475
    8             4469.494957     71.46393499      63.47858619     44.98321506
    9             3565.618621     71.52643718      63.52035976     89.99952136
      Texture.isotropy         Asfc        Smfc        HAsfc9      HAsfc81
    4      71.19962688  9.485112657 1.124339164  0.1154264841 0.3151333336
    5      52.59435919 0.8677972883 1.993685999  0.3643206898  1.338234209
    6      70.93291413  13.86047929 1.124339164 0.08100961788 0.2650550944
    7      68.88923084  2.864134488 3.767524759  0.4563153235 0.9349029537
    8      43.27506111  11.28658458 1.276959754  0.1509780971 0.3349801219
    9      9.334621652  8.154488245 1.993685999  0.4109842446 0.8871419954
               epLsar     NewEplsar
    4  0.001508254735 0.01833031405
    5  0.002329623452 0.01866926505
    6 0.0009464415845 0.01791760554
    7  0.001365762769 0.01812746064
    8  0.003606264068 0.01742323272
    9  0.005868610881 0.01564744046

## Extract units

``` r
# Filter out rows which contains units
n_units <- filter(STA, V4 == "<no unit>") %>% 
          
           # Keep only unique/distinct rows of units
           distinct() %>% 
  
           # Number of unique/distinct rows of units
           nrow()

if (n_units != 1) {
  stop("The different datasets have different units")
} else {
  # Extract unit line from 3rd row for considered columns
  STA_units <- unlist(STA[3, STA_keep_col[-1]])

  # Get names associated to the units
  names(STA_units) <- colnames(STA_keep)[-1]

  # Combine into a data.frame for export
  units_table <- data.frame(variable = names(STA_units), units = STA_units)
  row.names(units_table) <- NULL
}

# Check results
units_table
```

                       variable     units
    1                       NMP         %
    2                        Sq        nm
    3                       Ssk <no unit>
    4                       Sku <no unit>
    5                        Sp        nm
    6                        Sv        nm
    7                        Sz        nm
    8                        Sa        nm
    9                       Smr         %
    10                      Smc        nm
    11                      Sxp        nm
    12                      Sal        µm
    13                      Str <no unit>
    14                      Std         °
    15                      Ssw        µm
    16                      Sdq <no unit>
    17                      Sdr         %
    18                       Vm   µm³/µm²
    19                       Vv   µm³/µm²
    20                      Vmp   µm³/µm²
    21                      Vmc   µm³/µm²
    22                      Vvc   µm³/µm²
    23                      Vvv   µm³/µm²
    24 Maximum.depth.of.furrows        nm
    25    Mean.depth.of.furrows        nm
    26  Mean.density.of.furrows    cm/cm2
    27          First.direction         °
    28         Second.direction         °
    29          Third.direction         °
    30         Texture.isotropy         %
    31                     Asfc <no unit>
    32                     Smfc       µm²
    33                   HAsfc9 <no unit>
    34                  HAsfc81 <no unit>
    35                   epLsar <no unit>
    36                NewEplsar <no unit>

## Split column ‘Name’

``` r
STA_keep[c("Specimen", "Objective", "Side", "Location", "Strokes")] <- STA_keep$Name %>% 
  
                                                                       # Split by " --- " and keep 2nd element
                                                                       str_split_i(" --- ", i = 2) %>% 
                                                                       
                                                                       # Delete ".mnt"
                                                                       gsub(".mnt", "", .) %>% 
   
                                                                       # Split by "_" into a matrix with 7 cols
                                                                       str_split_fixed("_", n = 7) %>% 
  
                                                                       # Keep all cols but cols 2 and 6
                                                                       .[, -c(2, 6)]
```

## Split column ‘Specimen’

``` r
STA_keep[c("Chert_type", "Chert_tool")] <- STA_keep$Specimen %>% 
                                           
                                           # Split by "-" and keep 3rd element   
                                           str_split_i("-", i = 3) %>% 
  
                                           # Split after the first capital letter
                                           str_split("(?<=[A-Z]{1})", simplify = TRUE)
```

## Add columns with sample ID and bamboo species

``` r
# Load Weights dataset
weights <- loadObject(paste0(dir_out, "/ISEA_use-wear_Weights.Rbin"))

# Merge the data.frames by chert type and chert tool
STA_keep_ID_bamb <- merge(STA_keep, weights[1:4], by = c("Chert_type", "Chert_tool"))
```

## Keep only numerical parts in columns “Location” and “Strokes”

``` r
STA_keep_ID_bamb$Location <- gsub("loc", "", STA_keep_ID_bamb$Location)
STA_keep_ID_bamb$Strokes <- factor(gsub("strokes", "", STA_keep_ID_bamb$Strokes))
```

## Convert all parameter variables to numeric

``` r
STA_keep_ID_bamb <- type_convert(STA_keep_ID_bamb)
```

## NMP ratios

The summary statistics for NMP ratios are:

       Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
     0.1979  0.1997  0.2000  0.1998  0.2000  0.2002 

All seems good.

## Re-order columns and add units as comment

``` r
STA_final <- select(STA_keep_ID_bamb, Sample, Chert_type, Chert_tool, Bamboo_sp, 
                    Objective:Strokes, NMP:NewEplsar)
comment(STA_final) <- STA_units
```

Type `comment(STA_final)` to check the units of the parameters.

## Check the result

``` r
str(STA_final)
```

    'data.frame':   96 obs. of  44 variables:
     $ Sample                  : chr  "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" ...
     $ Chert_type              : chr  "A" "A" "A" "A" ...
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
head(STA_final)
```

        Sample Chert_type Chert_tool        Bamboo_sp Objective    Side Location
    1 ISEA-EX1          A          1 Bambusa blumeana  20x-0.70  dorsal        1
    2 ISEA-EX1          A          1 Bambusa blumeana  20x-0.70  dorsal        1
    3 ISEA-EX1          A          1 Bambusa blumeana  20x-0.70  dorsal        2
    4 ISEA-EX1          A          1 Bambusa blumeana  20x-0.70  dorsal        2
    5 ISEA-EX1          A          1 Bambusa blumeana  20x-0.70 ventral        1
    6 ISEA-EX1          A          1 Bambusa blumeana  20x-0.70 ventral        1
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

# Save data

## As XLSX

``` r
write_xlsx(list("data" = STA_final, "units" = units_table), path = paste0(dir_out, "/ISEA_use-wear_STA.xlsx"))
```

## As Rbin

``` r
saveObject(STA_final, file = paste0(dir_out, "/ISEA_use-wear_STA.Rbin"))
```

Rbin files (e.g. `ISEA_use-wear_STA.Rbin`) can be easily read into an R
object (e.g. `rbin_data`) using the following code:

``` r
library(R.utils)
rbin_data <- loadObject("ISEA_use-wear_STA.Rbin")
```

------------------------------------------------------------------------

# sessionInfo()

``` r
sessionInfo()
```

    R version 4.4.0 (2024-04-24 ucrt)
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
     [1] writexl_1.5.0     lubridate_1.9.3   forcats_1.0.0     stringr_1.5.1    
     [5] dplyr_1.1.4       purrr_1.0.2       readr_2.1.5       tidyr_1.3.1      
     [9] tibble_3.2.1      ggplot2_3.5.1     tidyverse_2.0.0   rmarkdown_2.27   
    [13] R.utils_2.12.3    R.oo_1.26.0       R.methodsS3_1.8.2 knitr_1.47       
    [17] grateful_0.2.7   

    loaded via a namespace (and not attached):
     [1] gtable_0.3.5      jsonlite_1.8.8    crayon_1.5.2      compiler_4.4.0   
     [5] tidyselect_1.2.1  jquerylib_0.1.4   scales_1.3.0      yaml_2.3.8       
     [9] fastmap_1.2.0     R6_2.5.1          generics_0.1.3    munsell_0.5.1    
    [13] rprojroot_2.0.4   tzdb_0.4.0        bslib_0.7.0       pillar_1.9.0     
    [17] rlang_1.1.4       utf8_1.2.4        stringi_1.8.4     cachem_1.1.0     
    [21] xfun_0.44         sass_0.4.9        timechange_0.3.0  cli_3.6.2        
    [25] withr_3.0.0       magrittr_2.0.3    digest_0.6.35     grid_4.4.0       
    [29] rstudioapi_0.16.0 hms_1.1.3         lifecycle_1.0.4   vctrs_0.6.5      
    [33] evaluate_0.23     glue_1.7.0        fansi_1.0.6       colorspace_2.1-0 
    [37] tools_4.4.0       pkgconfig_2.0.3   htmltools_0.5.8.1

------------------------------------------------------------------------

# Cite R packages used

| Package     | Version      | Citation                                                                                      |
|:------------|:-------------|:----------------------------------------------------------------------------------------------|
| base        | 4.4.0        | R Core Team (2024)                                                                            |
| grateful    | 0.2.7        | Rodriguez-Sanchez and Jackson (2023)                                                          |
| knitr       | 1.47         | Xie (2014); Xie (2015); Xie (2024)                                                            |
| R.methodsS3 | 1.8.2        | Bengtsson (2003a)                                                                             |
| R.oo        | 1.26.0       | Bengtsson (2003b)                                                                             |
| R.utils     | 2.12.3       | Bengtsson (2023)                                                                              |
| rmarkdown   | 2.27         | Xie, Allaire, and Grolemund (2018); Xie, Dervieux, and Riederer (2020); Allaire et al. (2024) |
| tidyverse   | 2.0.0        | Wickham et al. (2019)                                                                         |
| writexl     | 1.5.0        | Ooms (2024)                                                                                   |
| RStudio     | 2024.4.1.748 | Posit team (2024)                                                                             |

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

Rodriguez-Sanchez, Francisco, and Connor P. Jackson. 2023.
*<span class="nocase">grateful</span>: Facilitate Citation of r
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
