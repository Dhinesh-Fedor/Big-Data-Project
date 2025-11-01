# Big Data Stock Price Forecasting 

This project implements and compares two deep learning models (LSTM and TCN) for time-series forecasting of ITC stock prices. A key component of this project is a performance comparison between an end-to-end big data pipeline (using PySpark and HDFS) and a standard Python/Pandas pipeline.

## Features

* **Time-Series Forecasting:** Predicts minute-by-minute closing prices of ITC stock.
* **Deep Learning Models:** Implements and evaluates both **Long Short-Term Memory (LSTM)** and **Temporal Convolutional Network (TCN)** models.
* **Big Data Pipeline:** Ingests and processes data from the Hadoop Distributed File System (HDFS) using **PySpark**.
* **Performance Analysis:** Provides a direct comparison of execution time (data loading, preprocessing, and training) between the **PySpark pipeline** and a **standard Pandas pipeline**.
* **Live Prediction:** Includes a script to fetch live daily data from the **Alpha Vantage API** and predict the next day's market trend.

## Dataset

1.  **Historical Data:**
    * **File:** `ITC_minute.csv`
    * **Size:** 972,555 entries
    * **Schema:** `date` (datetime), `open` (float), `high` (float), `low` (float), `close` (float), `volume` (int)
    * **Source:** Loaded from HDFS (`hdfs://localhost:9000/bigdata/itc/ITC_minute.csv`) for the Spark pipeline and locally (`Dataset/ITC_minute.csv`) for the Pandas pipeline.

2.  **Live Data:**
    * **Source:** Alpha Vantage API
    * **Symbol:** `ITC.BSE`
    * **Usage:** Used to fetch the latest 60 days of data for a "live" trend prediction.

## ⚙️ Methodology & Pipeline

The project follows these main steps:

1.  **Data Ingestion:**
    * **PySpark Pipeline:** Loads the `ITC_minute.csv` from HDFS into a Spark DataFrame.
    * **Pandas Pipeline:** Loads the same CSV from the local disk into a Pandas DataFrame.

2.  **Preprocessing:**
    * Converts the `date` column to a proper timestamp format.
    * Sorts the data chronologically.
    * The PySpark DataFrame is converted to Pandas using `.toPandas()` for compatibility with deep learning libraries. This step is a key performance bottleneck.

3.  **Feature Engineering:**
    * The `close` price is isolated and normalized using `MinMaxScaler` (scaled between 0 and 1).
    * The data is transformed into sequences: **60 consecutive minutes** (`X`) are used to predict the **61st minute's** price (`y`).

4.  **Data Split:**
    * The dataset is split into 80% for training and 20% for testing.

5.  **Model Training:**
    * Two models, an LSTM and a TCN, are trained on the data for 20 epochs.
    * The `Comparison.ipynb` notebook trains an identical LSTM model for 5 epochs on both pipelines to measure execution time.

## Models Used

Both models were built using Keras and compiled with the `adam` optimizer and `mean_squared_error` loss function.

### 1. LSTM (Long Short-Term Memory)
A stacked LSTM network designed to capture temporal dependencies.
* **Layer 1:** `LSTM(50, return_sequences=True)`
* **Layer 2:** `Dropout(0.2)`
* **Layer 3:** `LSTM(50)`
* **Layer 4:** `Dropout(0.2)`
* **Layer 5:** `Dense(25)`
* **Layer 6:** `Dense(1)` (Output)

### 2. TCN (Temporal Convolutional Network)
A convolutional architecture adapted for time-series data, often outperforming RNNs.
* **Layer 1:** `TCN(nb_filters=64, kernel_size=3, dilations=[1, 2, 4, 8], nb_stacks=3)`
* **Layer 2:** `Dense(25)`
* **Layer 3:** `Dense(1)` (Output)

## Key Findings & Results

### Model Predictive Performance
Both models were highly effective at learning the minute-level price trends and volatility from the test set.

**LSTM Prediction:**
<img width="588" height="455" alt="image" src="https://github.com/user-attachments/assets/a99056dd-012a-4d22-8cd8-9aac1c4af98c" />


**TCN Prediction:**
<img width="588" height="455" alt="image" src="https://github.com/user-attachments/assets/2b685109-c165-4792-8418-e9a2b132090c" />


### Big Data Pipeline Performance Comparison
The core "big data" question was to compare the end-to-end pipeline speed. The test included data loading, preprocessing, sequence creation, and 5 epochs of LSTM training.

| Pipeline | Total Execution Time |
| :--- | :--- |
| **Standard Pandas** | **413.94 seconds** |
| **PySpark + HDFS** | **431.48 seconds** |

**Insight:** For this dataset (~970k rows), the **standard Python/Pandas pipeline was approximately 4% faster**. This is attributed to the computational overhead of initializing a SparkSession and the significant bottleneck of the `.toPandas()` operation, which must collect all distributed data back to the driver node. The benefits of distributed processing were outweighed by this overhead at this scale.

### Live Trend Prediction
Using daily data from October 2025, both trained models provided a forward-looking forecast.

* **LSTM Prediction:** **Bullish** (Raw Output: 0.729)
* **TCN Prediction:** **Bullish** (Raw Output: 0.775)
Clone the repository and install the required packages:
```bash
git clone [https://github.com/your-username/your-repo-name.git](https://github.com/your-username/your-repo-name.git)
cd your-repo-name
pip install -r requirements.txt
