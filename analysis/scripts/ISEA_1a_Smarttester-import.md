Import dataset from Smarttester’s sensors for the ISEA use-wear project
================
Ivan Calandra
2024-06-24 11:53:50 CEST

- [Goal of the script](#goal-of-the-script)
- [Load packages](#load-packages)
- [Read in TXT files](#read-in-txt-files)
- [Format data](#format-data)
  - [Pivot to wider format](#pivot-to-wider-format)
  - [Add stroke number](#add-stroke-number)
  - [Reorder columns and renumber
    rows](#reorder-columns-and-renumber-rows)
  - [Check result](#check-result)
- [Save data](#save-data)
  - [As XLSX](#as-xlsx)
  - [As Rbin](#as-rbin)
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
4.  Write an XLSX file and save an R object ready for further analysis
    in R

For each sample, each TXT file represents 10 back-and-fourth movements
(= 20 strokes) for a given sensor. After 20 strokes, the bamboo worked
piece was rotated.  
There were 5 positions (angles) for the bamboo: 0, -7.5, -15, +7.5 and
+15°.  
There were 5 sensors: depth sensor (`Depth`), position (`X_position`)
and velocity (`X_velocity`) along the X (movement) axis, and angle
(`Angle`) and torque (`Torque`) of the rotary drive for the bamboo.
Readings from each sensor are saved in a separate folder. It is possible
to save up to 6 sensors’ readings, so the folders are numbered from 1 to
6 (or multiples thereof, see below), even though we used only 5.

In order to smooth the saving and storing of the files, the sensors’
data were exported (written) after 20 strokes (hence each TXT file
represents 20 strokes). However, the readings for each angle were saved
in different folders, adding to the complexity of the folder structure.

There are 20 files per folder, where each file recorded 20 strokes,
meaning that a total of 400 strokes were performed at each angle. With 5
angles, a total 2000 strokes was recorded per sample.

With 20 files per folder and a folder for each combination of angle (5),
sensor (5) and sample (12), a total of 6000 TXT files were saved.
Nevertheless, there were issues during the experiment with ISEA-EX3 so
that the readings of the first 400 strokes with that sample were not
usable.

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
  the file “SmartDatafiles00000000.txt” gives the readings of
  `X_position` at angle = 0° for sample ISEA-EX1 for the strokes 1-20  
- in the folder
  “analysis/raw_data/Smarttester/ISEA-EX1/Messung7Date20231212131525”,
  the file “SmartDatafiles00000000.txt” gives the readings of `Depth` at
  angle = -7.5° for sample ISEA-EX1 for the strokes 21-40

``` r
dir_in <- "D:/Data/ISEA_use-wear/3_Experiments_Inotec/Data"
dir_out <- "analysis/derived_data/"
```

Due to the huge number of TXT files (see below), it was not possible to
upload them to GitHub. They were therefore stored and accessed locally
in “D:/Data/ISEA_use-wear/3_Experiments_Inotec/Data” for running the
script. They can be accessed on Zenodo: *add DOI*

Formatted data will be saved in “analysis/derived_data/”.

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

# Read in TXT files

``` r
# Create dataframe containing identification of folders
folders <- data.frame(bamboo = rep(c(0, -7.5, -15, 7.5, 15), each = 5), 
                      folder = seq_len(30)[-seq(from = 6, to = 30, by = 6)])

# Extract sample IDs from folder names
Samples_ID <- dir(dir_in)

# Create list, 1 element for each sample
Samples_data <- vector(mode = "list", length = length(Samples_ID)) 

# Create empty data.frame to contain a summary table of the number of detected TXT files per sample
num_txt <- data.frame(Sample = Samples_ID, NumberTXT = rep(NA, length(Samples_ID)))

# For each sample
for (i in seq_along(Samples_ID)) {
  
  # List all TXT files in dir_in/Samples_ID
  TXT_files <- list.files(paste(dir_in, Samples_ID[i], sep = "/"), 
                          pattern = "\\.txt$", recursive = TRUE, full.names = TRUE)
  
  # Display number of TXT files
  cat(length(TXT_files), "TXT files were found for sample", Samples_ID[i], "\n")
  
  # Save number of TXT files in num_txt
  num_txt[i, "NumberTXT"] <- length(TXT_files)
  
  # Import all TXT files 
  # Create list, 1 element for each TXT file
  TXT_data <- vector(mode = "list", length = length(TXT_files))

  # For each file
  for (j in seq_along(TXT_files)) {
    
  # Extract sensor's name and unit from the TXT file
               # Info on Line 3
  name_unit <- unlist(read.table(TXT_files[j], skip = 2, nrows = 1, sep = ";", 
                                 fileEncoding = 'WINDOWS-1252')) %>% 
                    
               # Format name and unit
               gsub("in ", "[", .)  %>% 
               paste0(., "]") %>%
               gsub("E2: ", "", .) %>% 
               gsub("Stroke", "X_position", .) %>% 
               gsub("Weg", "Depth", .) %>% 
               gsub("Velocity", "X_velocity", .)
  
  # Extract folder and file names from path
  messung <- as.numeric(gsub("Messung|Date[0-9]*", "", basename(dirname(TXT_files[j]))))
  smartfile <- as.numeric(gsub("SmartDatafiles000000|.txt", "", basename(TXT_files[j])))
  bamboo <- unique(folders[folders$folder == messung, "bamboo"])
  
  # Read in and format data
  TXT_data[[j]] <- read.table(TXT_files[j], skip = 4, sep = ";") %>%
                   mutate(Sample = Samples_ID[i], File = smartfile, Bamboo_position = bamboo, 
                          Sensor_name = name_unit) %>%
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
     $ Sensor_value   : num  -7.51 -7.53 -8.01 -8.16 -8.15 ...
     $ Step           : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample         : chr  "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" ...
     $ File           : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position: num  -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 ...
     $ Sensor_name    : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX10 
    'data.frame':   43210 obs. of  6 variables:
     $ Sensor_value   : num  -7.5 -7.51 -7.56 -7.63 -7.65 ...
     $ Step           : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample         : chr  "ISEA-EX10" "ISEA-EX10" "ISEA-EX10" "ISEA-EX10" ...
     $ File           : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position: num  -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 ...
     $ Sensor_name    : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX11 
    'data.frame':   41740 obs. of  6 variables:
     $ Sensor_value   : num  -7.5 -7.51 -7.54 -7.55 -7.56 ...
     $ Step           : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample         : chr  "ISEA-EX11" "ISEA-EX11" "ISEA-EX11" "ISEA-EX11" ...
     $ File           : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position: num  -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 ...
     $ Sensor_name    : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX12 
    'data.frame':   43175 obs. of  6 variables:
     $ Sensor_value   : num  -7.5 -7.5 -7.57 -7.59 -7.59 ...
     $ Step           : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample         : chr  "ISEA-EX12" "ISEA-EX12" "ISEA-EX12" "ISEA-EX12" ...
     $ File           : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position: num  -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 ...
     $ Sensor_name    : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX2 
    'data.frame':   55075 obs. of  6 variables:
     $ Sensor_value   : num  -7.47 -7.56 -8.65 -8.96 -9.17 ...
     $ Step           : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample         : chr  "ISEA-EX2" "ISEA-EX2" "ISEA-EX2" "ISEA-EX2" ...
     $ File           : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position: num  -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 ...
     $ Sensor_name    : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    400 TXT files were found for sample ISEA-EX3 
    'data.frame':   43605 obs. of  6 variables:
     $ Sensor_value   : num  -7.51 -7.57 -8.3 -8.44 -8.43 ...
     $ Step           : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample         : chr  "ISEA-EX3" "ISEA-EX3" "ISEA-EX3" "ISEA-EX3" ...
     $ File           : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position: num  -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 ...
     $ Sensor_name    : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX4 
    'data.frame':   54235 obs. of  6 variables:
     $ Sensor_value   : num  -7.49 -7.58 -8.41 -8.33 -8.27 ...
     $ Step           : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample         : chr  "ISEA-EX4" "ISEA-EX4" "ISEA-EX4" "ISEA-EX4" ...
     $ File           : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position: num  -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 ...
     $ Sensor_name    : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX5 
    'data.frame':   54580 obs. of  6 variables:
     $ Sensor_value   : num  -7.46 -7.56 -8.18 -8.2 -8.19 ...
     $ Step           : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample         : chr  "ISEA-EX5" "ISEA-EX5" "ISEA-EX5" "ISEA-EX5" ...
     $ File           : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position: num  -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 ...
     $ Sensor_name    : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX6 
    'data.frame':   54765 obs. of  6 variables:
     $ Sensor_value   : num  -7.48 -7.5 -8.28 -8.25 -8.28 ...
     $ Step           : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample         : chr  "ISEA-EX6" "ISEA-EX6" "ISEA-EX6" "ISEA-EX6" ...
     $ File           : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position: num  -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 ...
     $ Sensor_name    : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX7 
    'data.frame':   41205 obs. of  6 variables:
     $ Sensor_value   : num  -7.5 -7.5 -7.52 -7.52 -7.52 ...
     $ Step           : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample         : chr  "ISEA-EX7" "ISEA-EX7" "ISEA-EX7" "ISEA-EX7" ...
     $ File           : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position: num  -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 ...
     $ Sensor_name    : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX8 
    'data.frame':   42520 obs. of  6 variables:
     $ Sensor_value   : num  -7.5 -7.51 -7.66 -7.72 -7.73 ...
     $ Step           : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample         : chr  "ISEA-EX8" "ISEA-EX8" "ISEA-EX8" "ISEA-EX8" ...
     $ File           : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position: num  -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 ...
     $ Sensor_name    : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...
    500 TXT files were found for sample ISEA-EX9 
    'data.frame':   42425 obs. of  6 variables:
     $ Sensor_value   : num  -7.5 -7.51 -7.58 -7.65 -7.65 ...
     $ Step           : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample         : chr  "ISEA-EX9" "ISEA-EX9" "ISEA-EX9" "ISEA-EX9" ...
     $ File           : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position: num  -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 -7.5 ...
     $ Sensor_name    : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...

``` r
# rbind all files for all samples
all_data <- do.call(rbind, Samples_data) %>% 
            arrange(Sample, File, Bamboo_position, Sensor_name, Step)
str(all_data)
```

    'data.frame':   570975 obs. of  6 variables:
     $ Sensor_value   : num  -15 -15 -15.9 -15.9 -15.9 ...
     $ Step           : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample         : chr  "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" ...
     $ File           : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position: num  -15 -15 -15 -15 -15 -15 -15 -15 -15 -15 ...
     $ Sensor_name    : chr  "Angle [°]" "Angle [°]" "Angle [°]" "Angle [°]" ...

``` r
head(all_data)
```

      Sensor_value   Step   Sample File Bamboo_position Sensor_name
    1    -15.01365      0 ISEA-EX1    0             -15   Angle [°]
    2    -15.02978 100000 ISEA-EX1    0             -15   Angle [°]
    3    -15.89770 200000 ISEA-EX1    0             -15   Angle [°]
    4    -15.90114 300000 ISEA-EX1    0             -15   Angle [°]
    5    -15.87985 400000 ISEA-EX1    0             -15   Angle [°]
    6    -15.83350 500000 ISEA-EX1    0             -15   Angle [°]

The number of TXT files read in for each sample is as follow:

          Sample NumberTXT
    1   ISEA-EX1       500
    2   ISEA-EX2       500
    3   ISEA-EX3       400
    4   ISEA-EX4       500
    5   ISEA-EX5       500
    6   ISEA-EX6       500
    7   ISEA-EX7       500
    8   ISEA-EX8       500
    9   ISEA-EX9       500
    10 ISEA-EX10       500
    11 ISEA-EX11       500
    12 ISEA-EX12       500

In total, 5900 TXT files have been read in.

------------------------------------------------------------------------

# Format data

## Pivot to wider format

Create one column (variable) for each sensor and create an empty column
for the stroke number.

``` r
wide_data <- pivot_wider(all_data, names_from = Sensor_name, values_from = Sensor_value) %>% 
             mutate(Bamboo_position = factor(Bamboo_position, levels = unique(folders$bamboo)), 
                    StrokeNr = NA) %>% 
             arrange(Sample, File, Bamboo_position, Step) %>% 
             as.data.frame()
str(wide_data)
```

    'data.frame':   114195 obs. of  10 variables:
     $ Step             : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ Sample           : chr  "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" ...
     $ File             : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position  : Factor w/ 5 levels "0","-7.5","-15",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ Angle [°]        : num  0.000884 -0.016282 -0.589631 -0.811417 -0.699494 ...
     $ Depth [mm]       : num  10.15 10.15 9.92 9.36 9.23 ...
     $ Torque [Nm]      : num  1.4 1.6 4.9 5.7 6.3 7 8.2 -3.9 -2.7 -4.2 ...
     $ X_position [mm]  : num  275 275 279 284 285 ...
     $ X_velocity [mm/s]: num  0.00187 9.88465 59.21647 31.58108 0.39584 ...
     $ StrokeNr         : logi  NA NA NA NA NA NA ...

``` r
head(wide_data)
```

        Step   Sample File Bamboo_position     Angle [°] Depth [mm] Torque [Nm]
    1      0 ISEA-EX1    0               0  0.0008842885  10.151439         1.4
    2 100000 ISEA-EX1    0               0 -0.0162818470  10.146061         1.6
    3 200000 ISEA-EX1    0               0 -0.5896308000   9.922504         4.9
    4 300000 ISEA-EX1    0               0 -0.8114173000   9.362462         5.7
    5 400000 ISEA-EX1    0               0 -0.6994940600   9.229558         6.3
    6 500000 ISEA-EX1    0               0 -0.6383826000   9.255678         7.0
      X_position [mm] X_velocity [mm/s] StrokeNr
    1        275.0000       0.001873783       NA
    2        275.1974       9.884645000       NA
    3        279.2093      59.216473000       NA
    4        284.2325      31.581081000       NA
    5        284.9546       0.395838140       NA
    6        284.9808       0.040469542       NA

## Add stroke number

``` r
# Split by sample to work on each sample separately
split_sample <- split(wide_data, wide_data$Sample)

# Create empty data.frame to contain a summary table of the number of detected strokes per sample
num_stroke <- data.frame(Sample = names(split_sample), NumberStrokes = rep(NA, length(names(split_sample))))

# For each sample
for (i in seq_along(split_sample)) {
  
  # Start the first stroke at the first row
  # For ISEA-EX3, the first stroke in the dataset is stroke number 401
  if (grepl("ISEA-EX3", names(split_sample)[i])) {
    split_sample[[i]][1, "StrokeNr"] <- 401
  } else {
    split_sample[[i]][1, "StrokeNr"] <- 1
  }
  
  # Calculate the lagged difference between successive values of the X position ("Stroke [mm]")
  diff_stroke <- diff(split_sample[[i]][["X_position [mm]"]])
  
  # Duplicate the first value to have the same length as the original values
  diff_stroke <- c(diff_stroke[1], diff_stroke)
  
  # Identify differences of 0
  diff_stroke_0 <- which(diff_stroke == 0)
  
  # Replace the 0s with the previous value
  diff_stroke[diff_stroke_0] <- diff_stroke[diff_stroke_0 - 1]
  
  # Create empty vector to contain the result
  sign_diff_stroke <- vector(mode = "logical", length = length(diff_stroke))
  
  # For each difference (starting at the second value)
  for (j in 2:length(diff_stroke)) {
    
    # Identify inversions in the sign of the difference, i.e. when the drive starts moving in the other direction
    sign_diff_stroke[j] <- sign(diff_stroke[j]) != sign(diff_stroke[j-1])
  }
  
  # Identify inversions in the sign of the differences
  inv_stroke <- which(sign_diff_stroke)
  
  # Fill the data.frame num_stroke with the number of inversions + 1 (because it starts at stroke number 2)
  num_stroke[i, "NumberStrokes"] <- length(inv_stroke) + 1
  
  # For each inversion in the sign of the difference, 
  # using the index of the inversion - 1 in order to compensate for the lag,
  # increment the stroke number.
  # Stroke number starts at 2 because the 1st stroke was added manually at the 1st row
  # In case of ISEA-EX3, the stroke number starts at 402 because the first 400 strokes were not recorded properly
  if (grepl("ISEA-EX3", names(split_sample)[i])) {
    split_sample[[i]][inv_stroke - 1, "StrokeNr"] <- seq_along(inv_stroke) + 401
  } else {
    split_sample[[i]][inv_stroke - 1, "StrokeNr"] <- seq_along(inv_stroke) + 1
  }
  
  # Fill the stroke number down to fill in the rows of identical stroke numbers
  split_sample[[i]] <- fill(split_sample[[i]], StrokeNr)
}

# rbind all samples into a single data.frame
final_data <- do.call(rbind, split_sample)
```

The number of inversions (= strokes) detected in the sensor’s readings
for each sample is as follow:

          Sample NumberStrokes
    1   ISEA-EX1          2000
    2   ISEA-EX2          2000
    3   ISEA-EX3          1600
    4   ISEA-EX4          2000
    5   ISEA-EX5          2000
    6   ISEA-EX6          2000
    7   ISEA-EX7          2000
    8   ISEA-EX8          2000
    9   ISEA-EX9          2000
    10 ISEA-EX10          2000
    11 ISEA-EX11          2000
    12 ISEA-EX12          2000

## Reorder columns and renumber rows

``` r
final_data <- final_data[c(2:4, 1, 10, 5:9)]
row.names(final_data) <- NULL
```

## Check result

``` r
str(final_data)
```

    'data.frame':   114195 obs. of  10 variables:
     $ Sample           : chr  "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" "ISEA-EX1" ...
     $ File             : num  0 0 0 0 0 0 0 0 0 0 ...
     $ Bamboo_position  : Factor w/ 5 levels "0","-7.5","-15",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ Step             : int  0 100000 200000 300000 400000 500000 600000 700000 800000 900000 ...
     $ StrokeNr         : num  1 1 1 1 1 1 2 2 2 2 ...
     $ Angle [°]        : num  0.000884 -0.016282 -0.589631 -0.811417 -0.699494 ...
     $ Depth [mm]       : num  10.15 10.15 9.92 9.36 9.23 ...
     $ Torque [Nm]      : num  1.4 1.6 4.9 5.7 6.3 7 8.2 -3.9 -2.7 -4.2 ...
     $ X_position [mm]  : num  275 275 279 284 285 ...
     $ X_velocity [mm/s]: num  0.00187 9.88465 59.21647 31.58108 0.39584 ...

``` r
head(final_data)
```

        Sample File Bamboo_position   Step StrokeNr     Angle [°] Depth [mm]
    1 ISEA-EX1    0               0      0        1  0.0008842885  10.151439
    2 ISEA-EX1    0               0 100000        1 -0.0162818470  10.146061
    3 ISEA-EX1    0               0 200000        1 -0.5896308000   9.922504
    4 ISEA-EX1    0               0 300000        1 -0.8114173000   9.362462
    5 ISEA-EX1    0               0 400000        1 -0.6994940600   9.229558
    6 ISEA-EX1    0               0 500000        1 -0.6383826000   9.255678
      Torque [Nm] X_position [mm] X_velocity [mm/s]
    1         1.4        275.0000       0.001873783
    2         1.6        275.1974       9.884645000
    3         4.9        279.2093      59.216473000
    4         5.7        284.2325      31.581081000
    5         6.3        284.9546       0.395838140
    6         7.0        284.9808       0.040469542

------------------------------------------------------------------------

# Save data

## As XLSX

``` r
write_xlsx(final_data, path = paste0(dir_out, "/ISEA_use-wear_Smarttester.xlsx"))
```

Unfortunately, Git/GitHub/RStudio had issues pushing large ODS files
created with `readODS::write_ods()`.

## As Rbin

``` r
saveObject(final_data, file = paste0(dir_out, "/ISEA_use-wear_Smarttester.Rbin"))
```

Rbin files (e.g. `ISEA_use-wear_Smarttester.Rbin`) can be easily read
into an R object (e.g. `rbin_data`) using the following code:

``` r
library(R.utils)
rbin_data <- loadObject("ISEA_use-wear_Smarttester.Rbin")
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
     [1] gtable_0.3.5      jsonlite_1.8.8    compiler_4.4.0    tidyselect_1.2.1 
     [5] jquerylib_0.1.4   scales_1.3.0      yaml_2.3.8        fastmap_1.2.0    
     [9] R6_2.5.1          generics_0.1.3    munsell_0.5.1     rprojroot_2.0.4  
    [13] tzdb_0.4.0        bslib_0.7.0       pillar_1.9.0      rlang_1.1.4      
    [17] utf8_1.2.4        stringi_1.8.4     cachem_1.1.0      xfun_0.44        
    [21] sass_0.4.9        timechange_0.3.0  cli_3.6.2         withr_3.0.0      
    [25] magrittr_2.0.3    digest_0.6.35     grid_4.4.0        rstudioapi_0.16.0
    [29] hms_1.1.3         lifecycle_1.0.4   vctrs_0.6.5       evaluate_0.23    
    [33] glue_1.7.0        fansi_1.0.6       colorspace_2.1-0  tools_4.4.0      
    [37] pkgconfig_2.0.3   htmltools_0.5.8.1

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
