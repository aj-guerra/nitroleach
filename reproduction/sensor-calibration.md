# Soil Moisture Data Calibration and Visualization

## Code Link

[github](https://github.com/aj-guerra/nitroleach/blob/main/lora/sensor-calibration-r.ipynb)

## Overview

The R code aims to calibrate and visualize soil moisture (SM) data. It works by importing cleaned sensor data and raw sample data and then calibrating the sensor readings using linear models based on the relationship between different soil moisture measurements. The calibrated models are then applied to the cleaned sensor data, and the results are visualized.

## Importing and Preprocessing Data

### Importing Sample Data

The code imports a CSV file containing the true soil moisture data and preprocesses it by:

- Converting dates
- Recoding depth values
- Aggregating by `patchID`, `date`, and `depth`

```R
smdata_raw <- read.csv('data/true_data/nmin_raw.csv', ...)
smdata <- smdata_raw %>% ...
```

### Importing Sensor Data

The sensor data is  imported from a CSV created in daily_resampling and transformed to match the format of the true data.

```R
sensor_smdata <- read.csv("data/daily_data/all_data_daily.csv", ...)
```

## Calibration Model

### Grouping Data

The true data (`smdata_f`) is grouped by `patch` and `depth` to fit separate linear models for each group.

### Linear Model Fitting

For each combination of `patch` and `depth`, a linear model is fitted using the formula $ ( \text{sample vwc} = \alpha + \beta_0 \times \text{sensor data} )$.

```R
models <- by(smdata_f, list(as.factor(smdata_f$depth), as.factor(smdata_f$patch)), function(subset) lm(formula_obj, data = subset))
```

The coefficients (slope and intercept), number of input pairs, and $(R^2)$ values are stored for each model.

## Applying Calibration to Cleaned Data

The calibrated models are applied to the cleaned sensor data. Each file, representing one patch, undergoes the following process:

- the 'calibrator' dataframe of slopes and intercepts for each `patch`, `date`, and `depth` combination is joined.
- The relevant calibration model is applied.
- The calibrated data is saved in a new CSV file.

```R
calibrate <- function(file_path) {
    ...
}
```

Note that this step in the R script has an odd crash where once the files are written the cell runs indefinitely. Interrupt and restart the R kernel, and run cells NOT including the calibraiton cell in order to plot images.

## Visualization

The final step is to visualize the calibrated and uncalibrated data. The plots show how well the sensor data approximates the true soil moisture data before and after calibration.

```R
smplots_depth<- ggplot(smdata_f, aes(y=vwc, color=factor(patch))) + ...
smplots_cal<- ggplot(smdata_cal, aes(y=vwc, color=factor(patch))) + ...
```
