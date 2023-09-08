# nleachmap

## ZALF Nitrogen Leaching Project

This repo was built by Aaron Guerra for his summer 2023 internship with ZALF. The goal of this project is to use high temporal soil moisture data in combination with satellite imagery to predict nitrate leaching zones in a spatiotemporally diversified field of 'patches' which are seasonally rotated. Progress is ongoing.

### TODO

- create field interpolations for sample data (SM, nmin) for 5 good dates
- classify and predict (daily? weekly?)
- summarize classifications into final product

### SIDE TODO

- add ts plots comparing raw/clean/cleancal per patch, per depth

## Install / Setup

1. Download [conda](https://conda.io/projects/conda/en/latest/user-guide/install/download.html) and [git](https://git-scm.com/downloads)
2. Open Command Prompt, navigate where you want the repo to be, then copy github with ```git clone https://github.com/aj-guerra/nleachmap.git```
3. Open Anaconda Prompt and navigate to ```nleachmap/reproduction```

For gis-env kernel

1. ```conda env create -n gis-env --file gis-env.yml ```
2. ```conda activate gis-env```
3. ```python -m ipykernel install --user --name gis-env --display-name "Python (gis-env)"```

For lora-download kernel

1. ```conda env create -n lora-download --file lora-download-env.yml ```
2. ```conda activate lora-download```
3. ```python -m ipykernel install --user --name lora-download --display-name "Python (lora-download)"```

For R kernel

1. Open R, RStudio,
2. ```install.packages('IRkernel')```
3. ```IRkernel::installspec()```

Kernel is generally self-explanatory, either gis-env or R. lora-download is only used for Download-UI.

## Workflow

In general, just run notebooks all the way through in the order they are presented here and you should be able to get through it all. If data output is ```in nb``` then the script is probably not necessary for others.

### 1. Data Download/Creation/Preprocessing

#### 1a. LoRa Download/Creation/Preprocessing Steps /lora/

| data input | process | data output | script | status |
|------------|---------|-------------|--------|-------|
| none | download data* | data/sensor_data/ | DownloadUI | :heavy_check_mark: |
| sensor_data/ | outlier removal* | data/sensor_data_clean/ | sensor-cleaning-r | :heavy_check_mark: possibly redo in python using skl instead |
| sensor_data/ | resample daily sm* | data/dailydata/ | daily_resampling |  :heavy_check_mark: |
| dailydata/, sensor_data_clean/ | sensor calibration* | data/sensor_data_clean_cal/ | sensor-calibration-r | :heavy_check_mark: |


#### 1b. Raster Download/Creation/Preprocessing Steps /rasters/

| data input | process | data output | script | status |
|------------|---------|-------------|--------|-------|
| query | download data* | planet_tifs/ | planet-download | TBD when access to datacube resolved
| true_data/nmin.csv | interpolate n | nmin_tifs/ | nitrokrige
| sensor_data_clean_cal/ | interpolate sm | sm_tifs/ | smkrige
| planet_tifs | calculate indices | index_tifs/ | indicies

### 2. Model/Dataset Preparation  /prep/

#### 2a. Source Extraction Single Point /prep/single_point/

12ish dates, most have <10 patches sampled that day

|input|merged-with|cols|script|output|status|
|----|-----------|----|------|------|-|
|lora/true_data/ | | doy, n90, crop | sample_sensor_extract | sample_sensor_mr, sample_mr| :heavy_check_mark: |
|lora/sensor_sm_clean_cal/ | date, patch | sm30, sm60, sm90 | sample_sensor_extract | sensor_mr| :heavy_check_mark: |
|rasters/planet_tifs/. | date, patch | bands 1-8, indicies | planet_train_extract | planet_train_mr| :heavy_check_mark: |
|rasters/planet_tifs/. | date, patch | bands 1-8, indicies | planet_classify_extract | planet_classify_mr| |
|rasters/topo_tifs/.| patch | elevation, slope, aspect, TIR | topo_extract | topo_mr| :heavy_check_mark: |
|lora/data/shp/patch_coords.csv | patch | lon, lat, cluster | kmeanscluster | clusters | :heavy_check_mark: |

#### 2b. Source Extraction Interpolated /prep/interpolated_point/

5ish dates, all have almost all patches sampled that day

|input|merged-with|cols|script|output|status|
|----|-----------|----|------|------|-|
| lora/data/shp | ||maskpixels | pc-mask.tif |
|rasters/nmin_tifs/ | | doy, n90, crop | sample_extract| sample_mr|
|rasters/sm_tifs/ | date, x, y | sm30, sm60, sm90 | sm_extract | sensor_mr|
|rasters/planet_tifs/. | date, x, y | bands 1-8 | planet_extract | planet_mr|
|rasters/topo_tifs/.| x, y | elevation, slope, aspect, TIR | topo_extract | topo_mr|
|rasters/index_tifs/. | date, x, y |  indicies | index_extract | index_mr|

- topo_extract done with both 1m single pixel and 3x3m neighborhood calculation, extraction based on mean of shapefiles also possible but would reflect patch dynamics and not sensor location dynamics

#### 2c. Dataset Creation and Visualization

##### Merges

| data input | process | data output | script | status |
|------------|---------|-------------|--------|-------|
|sp/ sample_sensor_mr, planet_train_mr, topo_mr, clusters| merge single training points | train-predict/sp_training_data| sp_train_merge_r | :heavy_check_mark: |
|sp/ sensor_mr, sp_planet_train_mr, topo_mr, clusters| merge single classify points | train-predict/sp_classify_data| sp_classify_merge_r |  |
|ip/ sample_sensor_mr, planet_train_mr, topo_mr, clusters| merge interpolated training points | train-predict/ip_training_data| ip_train_merge_r | |
|ip/ sensor_mr, sp_planet_train_mr, topo_mr, clusters| merge interpolated classify points | train-predict/ip_classify_data| ip_classify_merge_r |  |
|merged, other| data inspection| in nb| features | :heavy_check_mark: |

##### Visualizations

| data input | process | data output | script | status |
|------------|---------|-------------|--------|-------|
| sensor_data/ | full time series plots | in nb | ts-all | :heavy_check_mark: |
| sensor_data_clean/ | full clean time series plots | in nb | ts-clean | :heavy_check_mark: |
| sensor_data_clean_cal/ | full clean, calibrated time series plots | in nb | ts-clean-cal | :heavy_check_mark: |
| sensor_data/, sensor_data_clean/, sensor_data_clean_cal/ | comparision time series plots | in nb | ts-compare |

### 3. Model Training and Predictions /train-predict/

#### 3a. Single-Point N Supervised Model (sp_s)

| data input | process | data output | script | status |
|------------|---------|-------------|--------|-------|
|data/sp_training_data| optimize+fit model| models/sp_s/ | fit_sp_s | :heavy_check_mark: |
|raster/, model/|classify and predict| classifications/ | classify
|classifications/| summarize classifications | output/ | summarize

#### 3b. Interpolated N Supervised Model (ip_s)

...

#### 3c. Single-Point N Unsupervised Model (sp_u)

...

#### 3c. Interpolated N Unsupervised Model (ip_u)

...
