Import dataset from Smarttester’s sensor for the ISEA use-wear project
================
Ivan Calandra
2024-05-17 15:12:16 CEST

- [Goal of the script](#goal-of-the-script)
- [Load packages](#load-packages)
- [List all files and get names of the
  files](#list-all-files-and-get-names-of-the-files)
- [sessionInfo()](#sessioninfo)
- [Cite R packages used](#cite-r-packages-used)
  - [References](#references)

------------------------------------------------------------------------

# Goal of the script

This script imports and merges all single TXT-files (strokes + sensors)
produced with the Inotec Smarttester during the experiments. The
experiment involved 12 samples (2 types of flint and 2 types of bamboo,
with 3 samples in each group) used for 2000 strokes.  
The script will:

1.  Read in the original TXT-files  
2.  Format and merge the data for each sample
3.  Combines the data from the 12 samples into one file  
4.  Write an ODS file and save an R object ready for further analysis in
    R

For each sample, each TXT file represents 10 back-and-fourth movements
(= 20 strokes) for a given sensor. After 20 strokes, the bamboo worked
piece was rotated.  
There were 5 positions (angles) for the bamboo: 0, -7.5, -15, +7.5 and
+15°.  
There were 5 sensors: depth sensor (`Depth`), position (`Position`) and
velocity (`Velocity`) along the X (movement) axis, and angle (`Angle`)
and torque (`Torque`) of the rotary drive for the bamboo. Readings from
each sensor are saved in a separate folder. It is possible to save up to
6 sensors’ readings, so the folders are numbered from 1 to 6 (or
multiples thereof, see below), even though we used only 5.

In order to smooth the saving and storing of the files, the sensors’
data were exported (written) after 20 strokes (hence each TXT file
represents 20 strokes). However, the readings for each angle were saved
in different folders, adding to the complexity of the folder structure.

There are 20 files per folder, where each file recorded 20 strokes,
meaning that a total of 400 strokes were performed at each angle. With 5
angles, a total 2000 strokes was recorded per sample.

With 20 files per folder and a folder for each combination of angle (5),
sensor (5) and sample (12), a total of 6000 TXT files were saved.

Therefore, for each sample:

- The folders ‘Messung1…’ to ‘Messung5…’ represent the readings for each
  of the 5 sensors, respectively, for Angle = 0°
- The folders ‘Messung7…’ to ‘Messung11…’ represent the readings for
  each of the 5 sensors, respectively, for Angle = -7.5°
- The folders ‘Messung13…’ to ‘Messung17…’ represent the readings for
  each of the 5 sensors, respectively, for Angle = -15°
- The folders ‘Messung19…’ to ‘Messung23…’ represent the readings for
  each of the 5 sensors, respectively, for Angle = +7.5°
- The folders ‘Messung25…’ to ‘Messung29…’ represent the readings for
  each of the 5 sensors, respectively, for Angle = +15°

In each folder, the files “SmartDatafiles00000000.txt” to
“SmartDatafiles00000019.txt” did not record contiguous strokes but
rather the strokes for a given sample and sensor at a given angle.  
For example:

- in the folder
  “analysis/raw_data/Smarttester/ISEA-EX1/Messung1Date20231212131525”,
  the file “SmartDatafiles00000000.txt” gives the readings of `Depth` at
  angle = 0° for sample ISEA-EX1 for the strokes 1-20  
- in the folder
  “analysis/raw_data/Smarttester/ISEA-EX1/Messung1Date20231212131525”,
  the file “SmartDatafiles00000001.txt” gives the readings of `Depth` at
  angle = 0° for sample ISEA-EX1 for the strokes 101-120  
- in the folder
  “analysis/raw_data/Smarttester/ISEA-EX1/Messung1Date20231212131525”,
  the file “SmartDatafiles00000019.txt” gives the readings of `Depth` at
  angle = 0° for sample ISEA-EX1 for the strokes 1901-1920  
- in the folder
  “analysis/raw_data/Smarttester/ISEA-EX1/Messung2Date20231212131525”,
  the file “SmartDatafiles00000000.txt” gives the readings of `Position`
  at angle = 0° for sample ISEA-EX1 for the strokes 1-20  
- in the folder
  “analysis/raw_data/Smarttester/ISEA-EX1/Messung7Date20231212131525”,
  the file “SmartDatafiles00000000.txt” gives the readings of `Depth` at
  angle = -7.5° for sample ISEA-EX1 for the strokes 21-40

``` r
dir_in <- "D:/Data/ISEA_use-wear/3_Experiments_Inotec/Data"
dir_out <- "analysis/derived_data/"
```

Due to the huge number of TXT files (6000), it was not possible to
upload them to GitHub. They were therefore stored and accessed locally
in “D:/Data/ISEA_use-wear/3_Experiments_Inotec/Data” for running the
script. They can be accessed on Zenodo:

Formatted data will be saved in “analysis/derived_data/”.

The knit directory for this script is the project directory.

------------------------------------------------------------------------

# Load packages

``` r
library(grateful)
library(knitr)
library(R.utils)
library(readODS)
library(rmarkdown)
library(tidyverse)
#library(openxlsx)
#library(tools)
```

------------------------------------------------------------------------

# List all files and get names of the files

``` r
# Create dataframe containing identification of folders
#folders <- data.frame(bamboo = c(0, -7.5, -15, 7.5, 15), Weg = seq(1, 30, 6), Stroke = seq(2, 30, 6), Velocity = seq(3, 30, 6), 
#                      Angle = seq(4, 30, 6), Torque = seq(5, 30, 6))
folders <- data.frame(bamboo = rep(c(0, -7.5, -15, 7.5, 15), each = 5), 
                      folder = seq_len(30)[-seq(from = 6, to = 30, by = 6)])


# Extract sample IDs from folder names
Samples_ID <- dir(dir_in)

# Exclude EX3 for now = TO ADJUST LATER
Samples_ID <- Samples_ID[Samples_ID != "ISEA-EX3"]

# Create list, 1 element for each sample
Samples_data <- vector(mode = "list", length = length(Samples_ID)) 

# For each sample
for (i in seq_along(Samples_ID)) {
  
  # List all TXT files in dir_in/Samples_ID
  TXT_files <- list.files(paste(dir_in, Samples_ID[i], sep = "/"), 
                          pattern = "\\.txt$", recursive = TRUE, full.names = TRUE)
  # Display number of TXT files
  cat(length(TXT_files), "TXT files were found for sample", Samples_ID[i], "\n")
  
  # Import all TXT files 
  # Create list, 1 element for each TXT file
  TXT_data <- vector(mode = "list", length = length(TXT_files))

  # For each file
  for (j in seq_along(TXT_files)) {
    
  # Extract sensor's name and unit from the TXT file
                      # Info on Line 3
  name_unit <- unlist(read.table(TXT_files[j], skip = 2, nrows = 1, sep = ";", fileEncoding = 'WINDOWS-1252')) %>% 
                    
               # Format name and unit
               gsub("in ", "[", .)  %>% 
               paste0(., "]") %>%
               gsub("E2: ", "", .)
  
  # Extract folder and file names from path
  messung <- as.numeric(gsub("Messung|Date[0-9]*", "", basename(dirname(TXT_files[j]))))
  smartfile <- as.numeric(gsub("SmartDatafiles000000|.txt", "", basename(TXT_files[j])))
  #bamboo <- unique(folders[folders$folder == messung, "bamboo"])
  
  # Read in and format data
  TXT_data[[j]] <- read.table(TXT_files[j], skip = 4, sep = ";") %>%
                   #mutate(Sample = Samples_ID[i], Folder = messung, File = smartfile, Bamboo_position = bamboo) %>%
                   mutate(Sample = Samples_ID[i], Folder = messung, File = smartfile, Sensor_name = name_unit) %>%
                   rename(Sensor_value = "V1", Step = "V2")
  }
  
  # rbind all files per sample
  Samples_data[[i]] <- do.call(rbind, TXT_data)
  str(Samples_data[[i]])
  head(Samples_data[[i]])
}
```

    500 TXT files were found for sample ISEA-EX1 
    'data.frame':   54440 obs. of  6 variables:
     $ Sensor_value: num  -7.51 -7.53 -8.01 -8.16 -8.15 ...
     $ Step        : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample      : chr  "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" ...
     $ Folder      : num  10 10 10 10 10 10 10 10 10 10 ...
     $ File        : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Sensor_name : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX10 
    'data.frame':   43210 obs. of  6 variables:
     $ Sensor_value: num  -7.5 -7.51 -7.56 -7.63 -7.65 ...
     $ Step        : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample      : chr  "ISEA-EX10" "ISEA-EX10" "ISEA-EX10" "ISEA-EX10" ...
     $ Folder      : num  10 10 10 10 10 10 10 10 10 10 ...
     $ File        : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Sensor_name : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX11 
    'data.frame':   41740 obs. of  6 variables:
     $ Sensor_value: num  -7.5 -7.51 -7.54 -7.55 -7.56 ...
     $ Step        : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample      : chr  "ISEA-EX11" "ISEA-EX11" "ISEA-EX11" "ISEA-EX11" ...
     $ Folder      : num  10 10 10 10 10 10 10 10 10 10 ...
     $ File        : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Sensor_name : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX12 
    'data.frame':   43175 obs. of  6 variables:
     $ Sensor_value: num  -7.5 -7.5 -7.57 -7.59 -7.59 ...
     $ Step        : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample      : chr  "ISEA-EX12" "ISEA-EX12" "ISEA-EX12" "ISEA-EX12" ...
     $ Folder      : num  10 10 10 10 10 10 10 10 10 10 ...
     $ File        : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Sensor_name : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX2 
    'data.frame':   55075 obs. of  6 variables:
     $ Sensor_value: num  -7.47 -7.56 -8.65 -8.96 -9.17 ...
     $ Step        : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample      : chr  "ISEA-EX2" "ISEA-EX2" "ISEA-EX2" "ISEA-EX2" ...
     $ Folder      : num  10 10 10 10 10 10 10 10 10 10 ...
     $ File        : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Sensor_name : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX4 
    'data.frame':   54235 obs. of  6 variables:
     $ Sensor_value: num  -7.49 -7.58 -8.41 -8.33 -8.27 ...
     $ Step        : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample      : chr  "ISEA-EX4" "ISEA-EX4" "ISEA-EX4" "ISEA-EX4" ...
     $ Folder      : num  10 10 10 10 10 10 10 10 10 10 ...
     $ File        : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Sensor_name : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX5 
    'data.frame':   54580 obs. of  6 variables:
     $ Sensor_value: num  -7.46 -7.56 -8.18 -8.2 -8.19 ...
     $ Step        : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample      : chr  "ISEA-EX5" "ISEA-EX5" "ISEA-EX5" "ISEA-EX5" ...
     $ Folder      : num  10 10 10 10 10 10 10 10 10 10 ...
     $ File        : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Sensor_name : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX6 
    'data.frame':   54765 obs. of  6 variables:
     $ Sensor_value: num  -7.48 -7.5 -8.28 -8.25 -8.28 ...
     $ Step        : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample      : chr  "ISEA-EX6" "ISEA-EX6" "ISEA-EX6" "ISEA-EX6" ...
     $ Folder      : num  10 10 10 10 10 10 10 10 10 10 ...
     $ File        : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Sensor_name : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX7 
    'data.frame':   41205 obs. of  6 variables:
     $ Sensor_value: num  -7.5 -7.5 -7.52 -7.52 -7.52 ...
     $ Step        : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample      : chr  "ISEA-EX7" "ISEA-EX7" "ISEA-EX7" "ISEA-EX7" ...
     $ Folder      : num  10 10 10 10 10 10 10 10 10 10 ...
     $ File        : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Sensor_name : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX8 
    'data.frame':   42520 obs. of  6 variables:
     $ Sensor_value: num  -7.5 -7.51 -7.66 -7.72 -7.73 ...
     $ Step        : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample      : chr  "ISEA-EX8" "ISEA-EX8" "ISEA-EX8" "ISEA-EX8" ...
     $ Folder      : num  10 10 10 10 10 10 10 10 10 10 ...
     $ File        : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Sensor_name : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX9 
    'data.frame':   42425 obs. of  6 variables:
     $ Sensor_value: num  -7.5 -7.51 -7.58 -7.65 -7.65 ...
     $ Step        : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample      : chr  "ISEA-EX9" "ISEA-EX9" "ISEA-EX9" "ISEA-EX9" ...
     $ Folder      : num  10 10 10 10 10 10 10 10 10 10 ...
     $ File        : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Sensor_name : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...

``` r
# rbind all files for all samples
all_data <- do.call(rbind, Samples_data)
str(all_data)
```

    'data.frame':   527370 obs. of  6 variables:
     $ Sensor_value: num  -7.51 -7.53 -8.01 -8.16 -8.15 ...
     $ Step        : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample      : chr  "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" ...
     $ Folder      : num  10 10 10 10 10 10 10 10 10 10 ...
     $ File        : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Sensor_name : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...

``` r
head(all_data)
```

      Sensor_value   Step   Sample Folder File Sensor_name
    1    -7.513763      0 ISEA-EX1     10    0   Angle [°]
    2    -7.527840 100000 ISEA-EX1     10    0   Angle [°]
    3    -8.007118 200000 ISEA-EX1     10    0   Angle [°]
    4    -8.157837 300000 ISEA-EX1     10    0   Angle [°]
    5    -8.150284 400000 ISEA-EX1     10    0   Angle [°]
    6    -8.104279 500000 ISEA-EX1     10    0   Angle [°]

------------------------------------------------------------------------

# sessionInfo()

``` r
sessionInfo()
```

    R version 4.3.3 (2024-02-29 ucrt)
    Platform: x86_64-w64-mingw32/x64 (64-bit)
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
     [1] lubridate_1.9.3   forcats_1.0.0     stringr_1.5.1     dplyr_1.1.4      
     [5] purrr_1.0.2       readr_2.1.5       tidyr_1.3.1       tibble_3.2.1     
     [9] ggplot2_3.5.0     tidyverse_2.0.0   rmarkdown_2.26    readODS_2.2.0    
    [13] R.utils_2.12.3    R.oo_1.26.0       R.methodsS3_1.8.2 knitr_1.45       
    [17] grateful_0.2.4   

    loaded via a namespace (and not attached):
     [1] gtable_0.3.4      jsonlite_1.8.8    compiler_4.3.3    tidyselect_1.2.1 
     [5] jquerylib_0.1.4   scales_1.3.0      yaml_2.3.8        fastmap_1.1.1    
     [9] R6_2.5.1          generics_0.1.3    munsell_0.5.1     rprojroot_2.0.4  
    [13] tzdb_0.4.0        bslib_0.7.0       pillar_1.9.0      rlang_1.1.3      
    [17] utf8_1.2.4        stringi_1.8.3     cachem_1.0.8      xfun_0.43        
    [21] sass_0.4.9        timechange_0.3.0  cli_3.6.2         withr_3.0.0      
    [25] magrittr_2.0.3    digest_0.6.35     grid_4.3.3        rstudioapi_0.16.0
    [29] hms_1.1.3         lifecycle_1.0.4   vctrs_0.6.5       evaluate_0.23    
    [33] glue_1.7.0        fansi_1.0.6       colorspace_2.1-0  tools_4.3.3      
    [37] pkgconfig_2.0.3   htmltools_0.5.8  

------------------------------------------------------------------------

# Cite R packages used

| Package     | Version | Citation                                                                                      |
|:------------|:--------|:----------------------------------------------------------------------------------------------|
| base        | 4.3.3   | R Core Team (2024)                                                                            |
| grateful    | 0.2.4   | Francisco Rodriguez-Sanchez and Connor P. Jackson (2023)                                      |
| knitr       | 1.45    | Xie (2014); Xie (2015); Xie (2023)                                                            |
| R.methodsS3 | 1.8.2   | Bengtsson (2003a)                                                                             |
| R.oo        | 1.26.0  | Bengtsson (2003b)                                                                             |
| R.utils     | 2.12.3  | Bengtsson (2023)                                                                              |
| readODS     | 2.2.0   | Schutten et al. (2024)                                                                        |
| rmarkdown   | 2.26    | Xie, Allaire, and Grolemund (2018); Xie, Dervieux, and Riederer (2020); Allaire et al. (2024) |
| tidyverse   | 2.0.0   | Wickham et al. (2019)                                                                         |

## References

<div id="refs" class="references csl-bib-body hanging-indent">

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

<div id="ref-grateful" class="csl-entry">

Francisco Rodriguez-Sanchez, and Connor P. Jackson. 2023.
*<span class="nocase">grateful</span>: Facilitate Citation of r
Packages*. <https://pakillo.github.io/grateful/>.

</div>

<div id="ref-base" class="csl-entry">

R Core Team. 2024. *R: A Language and Environment for Statistical
Computing*. Vienna, Austria: R Foundation for Statistical Computing.
<https://www.R-project.org/>.

</div>

<div id="ref-readODS" class="csl-entry">

Schutten, Gerrit-Jan, Chung-hong Chan, Peter Brohan, Detlef Steuer, and
Thomas J. Leeper. 2024. *<span class="nocase">readODS</span>: Read and
Write ODS Files*. <https://CRAN.R-project.org/package=readODS>.

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

<div id="ref-knitr2023" class="csl-entry">

———. 2023. *<span class="nocase">knitr</span>: A General-Purpose Package
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
