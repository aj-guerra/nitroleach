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

##### Visualizations

| data input | process | data output | script | status |
|------------|---------|-------------|--------|-------|
| sensor_data/ | full time series plots | in nb | ts-all | :heavy_check_mark: |
| sensor_data_clean/ | full clean time series plots | in nb | ts-clean | :heavy_check_mark: |
| sensor_data_clean_cal/ | full clean, calibrated time series plots | in nb | ts-clean-cal | :heavy_check_mark: |
| sensor_data/, sensor_data_clean/, sensor_data_clean_cal/ | comparision time series plots | in nb | ts-compare |

#### 1b. Raster Download/Creation/Preprocessing Steps /rasters/

| data input | process | data output | script | status |
|------------|---------|-------------|--------|-------|
| query | download data* | planet_tifs/ | planet-download | TBD when access to datacube resolved
| sensor_data_clean_cal/ | interpolate sm | sm_tifs/ | smkrige
| planet_tifs/ | calculate indices | index_tifs/ | indicies
| rasters/ | all rasters to same crs/ext | ../ | scale-rasters |

### 2. Model/Dataset Preparation  /prep/

#### 2a. Extraction for Point Data /prep/point

12ish sample dates, most have <10 patches sampled that day

|input|merged-with|cols|script|output|status|
|----|-----------|----|------|------|-|
|lora/true_data/ | | doy, n90, crop | sample_sensor_extract | sample_sensor_mr, sample_mr| :heavy_check_mark: |
|lora/sensor_sm_clean_cal/ | date, patch | sm30, sm60, sm90 | sample_sensor_extract | sensor_mr| :heavy_check_mark: |
|rasters/planet_tifs/. | date, patch | bands 1-8, indicies | planet_extract | planet_mr| :heavy_check_mark: |
|rasters/topo_tifs/.| patch | elevation, slope, aspect, TIR | topo_extract | topo_mr| :heavy_check_mark: |
|lora/data/shp/patch_coords.csv | patch | lon, lat, cluster | kmeanscluster | clusters | :heavy_check_mark: |

#### 2b. Source Extraction Points /prep/raster

|input|merged-with|cols|script|output|status|
|----|-----------|----|------|------|-|
|

### 3. Model Training and Predictions /train-predict/

#### 3a. Nitrogen Model Training

| data input | process | data output | script | status |
|------------|---------|-------------|--------|-------|
|prep/point/merge_ready/| merge points from sample dates | data/dataset_1 | merge_dataset_1 | :heavy_check_mark: |
|dataset_1 | feature inspection | in nb | features | :heavy_check_mark: |
|dataset_1 | optimize+fit model 1 | models/m1/ | train_nmodel_1 | :heavy_check_mark: |
|prep/point/merge_ready/, m1 | predict+create dataset 2 | data/dataset_2 | pred_ds_2 | |
|dataset_2| optimize+fit model 2 | models/m2/ | train_nmodel_2 | |
|dataset_3| optimize+fit model 3 | models/m3/ | train_nmodel_3 | |

#### 3b. Nitrogen Model Predictions

| data input | process | data output | script | status |
|------------|---------|-------------|--------|-------|
|raster/, model/|classify and predict| classifications/ | classify
|classifications/| summarize classifications | output/ | summarize
