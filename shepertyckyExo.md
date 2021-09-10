ShepertyckyExo
================
Jereme Outerleys
2021-08-26

## Summary

3 overground walking trials were collected using Theia3D. Markerless
data were collected with 8 Sony RX0II cameras with a frame rate of 60 Hz
(actual 59.9401 Hz).

## Data Processing

An 8 Hz GCVSPL filter was applied to the markerless kinematics data.
Heel strike events were kinematically determined in Visual3D and
exported using the *Export\_Data\_To\_Ascii\_File* function. Data were
time normalized to full gait cycles (101 points).

### Attach R packages

``` r
#library(devtools)
#install_github("jouterleys/v3dR")
library(v3dR)
library(tidyverse)
library(stringr)
library(here)
```

### Build loopV3DR

``` r
# Build a function to do something on each dataframe
loopV3DR <- function(full_filepath){
  
  # Add some contextual information to the dataframe
  # from each loop on import.
  
  # side information is contained in the txt filename.
  side <- strsplit(basename(full_filepath), "_")[[1]][1]
  
  # condition information is contained in the folder path
  # In this case the folder name up two directories from the txt file is the subID
  condition <- basename(dirname(dirname(full_filepath)))
  
  # Read current txt file into a dataframe and add the columns condition and side.
  df <- v3dR(full_filepath) %>%  mutate(condition = condition, side = side)
  
  return(df)
  
}
```

### Define paths and build data frame

``` r
# Define path to the subject folders
resultsPath <- here('S00001')

# Create list of subjects based on subject folders
subList <- list.dirs(resultsPath, recursive=FALSE)

# Build dataframe containing all subject data
# Creates list of all txt files from within the results folder
# Then runs the loopV3DR function on each item in the list
df <- list.files(file.path(subList,'results'), pattern = "*.txt", full.names = TRUE) %>% 
  map_df(~loopV3DR(.)) 

df <- as.data.frame(df)

# The signal_names are named in visual3D as LANKLE_ANGLE, RANKLE_ANGLE, etc.
# Since we are using long data format and assign the side during import
# we will remove the first letter of the signal_name
df$signal_names <- as.factor(substring(df$signal_names, 2))
df$signal_names <- fct_relevel(df$signal_names, "ANKLE_ANGLE", "KNEE_ANGLE", "HIP_ANGLE")
```

## Kinematics

``` r
df %>%
  subset(select = -c(c3d_name, instance)) %>%
  filter(grepl("ANGLE", signal_names)) %>%
  ggplot(aes(x = item, y = value, group = side, color = side)) +
  geom_hline(yintercept = 0, color = "black", size = 0.25) +
  stat_summary(fun = mean, geom = "line") +
  stat_summary(fun.data = "mean_sdl", fun.args = list(mult = 1),
               mapping = aes(color = side, fill = side),
               geom = "ribbon", alpha = 0.25, colour = NA) +
  facet_wrap(signal_components ~ signal_names, scales = "free",
             labeller = labeller(signal_components = as_labeller(component_names))) +
 theme_minimal() +
 theme(axis.line = element_line(size = 1, colour = "black"), legend.position = "top") +
 scale_x_continuous(expand = c(0, 0)) +
  xlab('Percent of Gait Cycle (%)') +
  ylab('Joint Angle (Degrees)')
```

<div class="figure" style="text-align: center">

<img src="shepertyckyExo_files/figure-gfm/kinematics_right_fig-1.png" alt="Left and right lower limb joint kinematics. Note: Dorsi-flexion, knee flexion, and hip flexion are positive (+). Ankle inversion, knee adduction and hip adduction is positive (+). Toe-in, knee internal rotation, and hip internal rotation are positive (+)."  />
<p class="caption">
Left and right lower limb joint kinematics. Note: Dorsi-flexion, knee
flexion, and hip flexion are positive (+). Ankle inversion, knee
adduction and hip adduction is positive (+). Toe-in, knee internal
rotation, and hip internal rotation are positive (+).
</p>

</div>