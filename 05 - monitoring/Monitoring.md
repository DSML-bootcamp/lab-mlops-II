# Monitoring

## Table of contents

1. Introduction
   
   1.1. Why We Need to Monitor Models After Deployment?
   
   1.2. What is Data Drift?
   
   1.3. How is Machine Learning Monitoring Different?
   
   1.4. Monitoring Types
   
   1.5. What is a Monitoring Scheme?

2. Build Monitoring Scheme
   
   2.1. Environment Setup
   
   2.2. Prepare Reference and Model
   
   2.3. Evidently Metrics Calculation
   
   2.4. Dummy Monitoring
   
   2.5. Data Quality Monitoring
   
   2.6. Save Grafana Dashboard
   
   2.7. Debugging with Test Suites and Reports

# 1. Introduction

## 1.1. Why we need to monitor models after deployment?

| Production | Key Questions|
|--------|-----|
| Data distribution changes | Why are there sudden changes in the values of my features?|
|Model ownership in production | Who owns the model in production? The DevOps team? Engineers? Data Scientists?|
|Training-serving skew|Why is the model giving poor results in production despite our rigorous testing and validation attempts during development?|
|Model/concept drift|Why was my model performing well in production and suddenly the performance dipped over time?|
|Black box models|How can I interpret and explain my model’s predictions in line with the business objective and to relevant stakeholders?|
|Concerted adversaries|How can I ensure the security of my model? Is my model being attacked?|
|Model readiness|How will I compare results from a newer version(s) of my model against the in-production version(s)?|
|Pipeline health issues|Why does my training pipeline fail when executed? Why does a retraining job take so long to run?|
|Underperforming system|Why is the latency of my predictive service very high? Why am I getting vastly varying latencies for my different models?|
|Cases of extreme events (outliers)|How will I be able to track the effect and performance of my model in extreme and unplanned situations?|
|Data quality issues|How can I ensure the production data is being processed in the same way as the training data was?|

**We need to**:
- Monitor model performance in production: Check how accurate the predictions are and see if performance decays over time, indicating a need for re-training.
- Monitor model input/output distribution: Ensure the distribution of input data and features has not changed, which could indicate data or concept drift.
- Monitor model training and re-training: Observe learning curves, prediction distributions, or confusion matrices during training.
- Monitor model evaluation and testing: Log metrics, charts, predictions, and other metadata for automated evaluation or testing pipelines.
- Monitor hardware metrics: Track CPU/GPU or memory usage during training and inference.
- Monitor CI/CD pipelines for ML: Evaluate CI/CD pipeline jobs and compare them visually. Metrics alone may not suffice, and visual inspection is often necessary.


Ref: 
* [A Comprehensive Guide on How to Monitor Your Models in Production](https://neptune.ai/blog/how-to-monitor-your-models-in-production-guide)
* [Best Tools to Do ML Model Monitoring](https://neptune.ai/blog/ml-model-monitoring-best-tools)
## 1.2. What is data drift

Data drift, also known as covariate shift, happens when the data that your machine learning model sees in production (real-world use) is different from the data it was trained on. This change in data distribution can cause the model's performance to decline because the model is no longer seeing data that is similar to what it was trained on.

#### Why Does Data Drift Matter?

1. **Accuracy Decline:** When the input data changes significantly, the model may become less accurate. For example, if a model was trained to predict customer churn based on past behavior patterns, but the behavior patterns change due to a new marketing campaign, the model's predictions might become less reliable.

2. **Performance Issues:** The model might start making more errors or less confident predictions because it is encountering data that it has not seen before.

3. **Unexpected Behavior:** Changes in data can lead to unexpected behavior in the model, such as increased error rates or biased predictions.

#### How to Detect Data Drift?

- **Monitoring:** Continuously monitor the input data and compare its distribution to the training data. Tools like histograms or statistical tests can help identify significant changes.
- **Metrics:** Track performance metrics over time. A sudden drop in performance can be an indicator of data drift.
- **Alerts:** Set up alerts to notify when there is a significant change in data distribution or model performance.

#### Example:

Imagine you have a model that predicts whether a customer will buy a product based on their online behavior. If your training data consists of user behavior from the past year, but your model starts getting input data from a new marketing campaign that changes user behavior drastically, your model might not perform well anymore. This difference in the type of data it sees in production versus what it was trained on is data drift.


Ref: 
[Data Drift vs. Concept Drift: What Is the Difference?](https://www.dataversity.net/data-drift-vs-concept-drift-what-is-the-difference/#:~:text=Data%20drift%20refers%20to%20the,of%20a%20machine%20learning%20model.)

## 1.3. How is machine learning monitoring different?

Monitoring machine learning models is different from traditional software monitoring because it involves tracking the performance and behavior of models, data quality, and the integrity of data pipelines. Here are the key aspects:

#### Main Aspects:

1. **Service Health:**
   - Monitored by a few essential metrics to ensure that the service is operational and running smoothly.

2. **Model Health:**
   - Evaluates how well the model is performing its task. The evaluation metrics depend on the type of task:
     - For a search or recommendation system, ranking metrics are used.
     - For regression tasks, Mean Absolute Error (MAE) or Mean Absolute Percentage Error (MAP) are common.
     - For classification tasks, metrics like Log Loss, Precision, or Recall are used.

3. **Data Quality and Integrity:**
   - Ensures that the data quality is maintained both when receiving input and generating output. Metrics to monitor include:
     - Share or count of missing values.
     - Value ranges for different features.

4. **Data and Concept Drift:**
   - Monitors the distribution of input and output data to detect changes caused by real-world environments. This helps in identifying if the model's input data distribution has shifted from what it was trained on.

#### Other Considerations:

1. **Performance by Segments:**
   - Evaluates the quality metrics by different segments or categories. This helps in understanding how the model performs across various subgroups of data.

2. **Bias and Fairness:**
   - Ensures that the model is fair and unbiased across different demographic groups.

3. **Outliers:**
   - Identifies and monitors outliers that might affect the model's performance or indicate unusual events.

4. **Explainability:**
   - Provides insights into how the model makes predictions. This is crucial for understanding and trusting the model’s decisions, especially for stakeholders and regulatory purposes.


## 1.4. Monitoring Types

#### Batch Monitoring:
Batch monitoring involves evaluating the performance of the machine learning model at regular intervals using batches of data. Instead of continuous real-time monitoring, the model's predictions and performance metrics are assessed periodically, such as daily, weekly, or monthly. This approach is useful for models that don't require immediate feedback and can benefit from aggregated insights over time.

#### Online (Non-batch) Monitoring:
Online monitoring, also known as real-time monitoring, involves continuously tracking the model's performance as it processes incoming data. This type of monitoring provides immediate feedback on the model's predictions and behavior, allowing for quick detection and response to any issues such as data drift, performance degradation, or anomalies. It is essential for applications that require real-time predictions and quick adaptability to changing data patterns.

## 1.5. What is monitoring scheme

A monitoring scheme outlines the steps and processes involved in tracking the performance and health of a machine learning model in production. Here are the key steps:

1. **Use Service:**
   - Implement the machine learning model as a REST API or a batch pipeline to make it accessible for predictions and analysis.

2. **Obtain Logs:**
   - Generate logs that record the model's activity and performance. These logs are typically saved as local log files whenever the service is used.

3. **Calculate Metrics Based on Logs:**
   - Analyze the logs to calculate important performance metrics. These metrics help in understanding how well the model is performing over time.

4. **Load Metrics into a Database:**
   - Save the calculated metrics into a PostgreSQL database. This centralized storage makes it easier to manage and query performance data.

5. **Build a Dashboard:**
   - Use Grafana to build a dashboard that visualizes the data from the PostgreSQL database. This dashboard provides an easy-to-understand interface to monitor the model's performance and health.


# 2. Build Monitoring Scheme

## 2.1. Environment set up
**Step 1**: Build virtual environment
```
mkdir taxi_monitoring
cd taxi_monitoring
conda create -n py11 python=3.11
conda activate py11
```

**Step 2**: Install a requirement.txt
`pip install -r requirement.txt`

**Step 3**: Build Docker compose
1. install Docker compose with `conda install docker-compose`
2. Write a [docker-compose.yml](taxi_monitoring%2Fdocker-compose.yml) and [grafana_datasources.yaml](taxi_monitoring%2Fconfig%2Fgrafana_datasources.yaml) (Under ./config) install it with 
`docker-compose up --build`
![docker-compose.png](img%2Fdocker-compose.png)
3. Go to Grafana by `localhost:3000`. with 
```
username: `admin` 
pwd: `admin`, 
```
then change to your own pwd.
4. Go to adminer by `localhost:8080`, with 
```
System: PostgreSQL
Server: db
Username: postgres
Password: example	
Database: test
```


## 2.2. Prepare reference and model
Make the dirs for the project and prepare
```
conda activate py11
docker-compose down
mkdir data
mkdir models
```
Then write the code as [baseline_model_nyc_taxi_data.ipynb](taxi_monitoring%2Fbaseline_model_nyc_taxi_data.ipynb)


## 2.3. Evidently metrics calculation
See details in [baseline_model_nyc_taxi_data.ipynb](taxi_monitoring%2Fbaseline_model_nyc_taxi_data.ipynb) about data drift between reference data (training data) and current data (val data) and other metrics.


## 2.4. Dummy monitoring

**Step 1**: Code for dummy metrics calculation as [dummy_metrics_calculation.py](dummy_metrics_calculation.py)

**Step 2**: Build services by docker-compose with `docker-compose up --build` 

**Step 3**: Check SQL on browser with `localhost:8080`.

**Step 4**: Build services on System: `PostgreSQL`, login info is in [grafana_datasources.yaml](taxi_monitoring%2Fconfig%2Fgrafana_datasources.yaml).
Here is what can be seen from sql.

![sql.png](img%2Fsql.png)

**Step 5**: Create a new dashboard and see the results as follows

![grafana.png](img%2Fgrafana.png)


## 2.5. Data quality monitoring
**Step 1**: Compared with [dummy_metrics_calculation.py](taxi_monitoring%2Fdummy_metrics_calculation.py), we loaded the reference data, models and the needed compared data, also report for metrics calculation; we also changed `calculate_metrics_postgresql` for insert necessary metrics into postgresql; all changes can be seen in [evidently_report_example.py](taxi_monitoring%2Fevidently_report_example.py).
Run `python evidently_metrics_calculation.py` and login `localhost:8080` to see if it works.
![data_quality_sql.png](img%2Fdata_quality_sql.png)

**Step 2**: Build Prefect pipeline (Add @task/@flow)

**Step 3**: Start Prefect server and run the script with `prefect server start` and `python evidently_metrics_calculation.py` in different terminals. The results should be:

![prefect.png](img%2Fprefect.png)

**Step 4**: Access Grafana with `localhost:3000` and create the new dashboard.

![dashboard.png](img%2Fdashboard.png)


## 2.6. Save Grafana Dashboard

**Step 1**: Create a [grafana_dashboards.yaml](taxi_monitoring%2Fconfig%2Fgrafana_dashboards.yaml) file under `./config`

**Step 2**: Add the following content to docker-compose file for updating dashboards info.
```
      - ./config/grafana_dashboards.yaml:/etc/grafana/provisioning/dashboards/dashboards.yaml:ro
      - ./dashboards:/opt/grafana/dashboards
```
**Step 3**: Create new `dashboards` dict and create a `data_drift.json` for Grafana. Copy the json data from the dashboards generated in the last step.

**Step 4**: Restart docker-compose with ```docker-compose down docker-compose up```, and go to Grafana to check the dashboards we saved.

![new_dashboards.png](img%2Fnew_dashboards.png)

![dashboard.png](img%2Fdashboard.png)

## 2.7. Debugging with test suites and reports

[debugging_nyc_taxi_data.ipynb](taxi_monitoring%2Fdebugging_nyc_taxi_data.ipynb)


### Note: 
1. `grafana_1 | Error: ✗ Datasource provisioning error: read /etc/grafana/provisioning/datasources/datasource.yaml: is a directory`

**Solutions**: `docker-compose rm` then restart it with `docker-compose up --build`

2. `OSError: [Errno 30] Read-only file system: './data'` since can not find the relative path 

**Solutions**:
- Change path './data' to absolute address
- Or change the config file of jupyter notebook 
    ```
    jupyter notebook --generate-config 
    cd /Users/amberm (YOUR_USER_NAME)/.jupyter
    vim jupyter_notebook_config.py
    ```
   and then change the default path to the one under your path
   ```
   ## The directory to use for notebooks and kernels
   #  Default: ''
   c.NotebookApp.notebook_dir = 'YOUR_PRO_PATH'
  ``` 
  
3. Install evidently package

**Solutions**:
`conda install -c conda-forge evidently`
