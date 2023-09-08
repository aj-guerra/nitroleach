# Soil Moisture Data Cleaning via Anomaly Removal

## Code Link

[github](https://github.com/aj-guerra/nitroleach/blob/main/lora/sensor-cleaning-r.ipynb)

## Summary

The primary goal of the code is to clean sensor data by removing anomalies or outliers. This is essential for improving the accuracy and reliability of subsequent data analyses. The code uses advanced statistical techniques, specifically Time Decomposition and Anomalize methods, to identify and remove these anomalies.

## Key Steps for Anomaly Removal

### Time Decomposition

The first major step in removing anomalies is 'Time Decomposition.' In this step, the original time series data is broken down into three primary components:

- Seasonal Component: Represents periodic fluctuations in the data.
- Trend Component: Captures the overall direction in which the data is moving over time.
- Remainder (Noise) Component: Consists of random or irregular movements not explained by seasonality or trend.

```python
time_decompose(.data[[var]], method = "stl", frequency = 60, trend = 180)
```

The code uses the "STL" (Season-Trend decomposition using LOESS) method for this purpose. The 'frequency' and 'trend' parameters are set to capture hourly fluctuations and 3-hour trends, respectively. The rationale behind choosing these parameters is primarily that they work to remove the data spikes without removing accurate data.

### Anomalize Method

```python
anomalize(remainder, method = "iqr", alpha = 0.05)
```

Once the time decomposition is complete, the 'Anomalize' method is applied to the 'Remainder' component to identify anomalies. The code uses the 'IQR' (Interquartile Range) method to detect data points that deviate significantly from the norm. A critical parameter here is 'alpha,' set at 0.05, which was similarly chosen because it is default and it works.

### Filtering Outliers

```python
df_no_outliers <- df_outliers %>%
    filter(anomaly == 'No')
```

After identifying the anomalies, the code filters them out, retaining only the 'clean' data points. This cleaned data is transformed into a more standard long format, where columns include `dateTime, side, depth, wc, patch`, where patch is extracted from the file name, and then written to csv.
