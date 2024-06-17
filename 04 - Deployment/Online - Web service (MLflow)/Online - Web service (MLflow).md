# Online - Web service (MLflow)

## Table of contents

1. Get the Models from Model Registry (MLflow)
   
   - Install AWS CLI
   
   - Confirm S3 Bucket Access
   
   - Start MLflow Tracking Server
   
   - Train and Log Model Locally
   
   - Download and Use Model Artifacts

2. Make Pipeline for DictVectorizer and RandomForestRegressor

3. New Instance Using S3 Without Connecting to Tracking Server

## 1. Get the models from model the model registry(MLflow)

Here, using locally hosted sqlite database for tracking server and s3 bucket for artifact storage. 
For that as a prerequisite create an S3 bucket and EC2 instance being used has access to the S3 bucket. 

### Install AWS CLI

Install `aws` CLI:  

``` 
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

### Confirm S3 Bucket Access

***To confirm try the following command and see you are able to get the list of buckets.***

    `aws s3 ls <target>`
    
#### Notes:

**a. ERROR: ' Unable to locate credentials. you can configure credentials by running `aws configure`. '**
    ![awsconfig.png](imgs%2Fawsconfig.png)

     1). Check configure list if the credential is already set, `aws configure list`
     2). (optional) If it does not, and shows as follows, set the `config` file.
      - AWS Access Key ID [None]: The access key id from IAM 
      - AWS Secret Access Key [None]: The Secret key id from IAM
      - default region name should be the similar format as: `ca-central-1`

**b. `aws s3 ls <target> --recursive` shows the history of all uploaded files**

### Start MLflow Tracking Server

**Run the following to start the mlflow tracking server:**

    `mlflow server  --backend-store-uri sqlite:///mlflow.db --default-artifact-root=s3://zoomcamp-mlops/`

    ***zoomcamp-mlops*** is the name of the S3 bucket I created in `[03-orchestration](..%2F..%2F03-orchestration)` which is for artifact storage.


*If an error reported as "MlflowException: API request to endpoint /api/2.0/mlflow/experiments/get-by-name failed with error code 403 != 200. Response body: ''", it is because mlflow did not start.*


###  Train and Log Model Locally

Train the model in local environment. 
And check the experiment details and logged model artifact in mlflow UI ( http://127.0.0.1:5000/)

![awsartifact.png](imgs%2Fawsartifact.png)

![mlflowartifact.png](imgs%2Fmlflowartifact.png)


#### Note: 
"OSError: [Errno 30] Read-only file system: 'dict_vectorizer.bin'"
ANS: Run `jupyter notebook`, and run it online.

### Download and Use Model Artifacts

After we uploaded the 'dict_vectorizer.bin', copy the `Pipfile`, `Pipfile.lock`, `predict.py` and `test.py`.
![mlflowbin.png](imgs%2Fmlflowbin.png)

    -  Install `MLflow` into `pipenv` setting

    -  Edit the `predict.py`.
    ```
       import mlflow
        from mlflow.tracking import MlflowClient
    
    MLFLOW_TRACKING_URI = 'http://127.0.0.1:5000'
    RUN_ID = 'YOUR_RUN_ID'
    
    mlflow.set_tracking_uri(MLFLOW_TRACKING_URI)
    client = MlflowClient(tracking_uri=MLFLOW_TRACKING_URI)
    
    path = client.download_artifacts(run_id=RUN_ID, path='dict_vectorizer.bin')
    
    print(f'downloading the dict vectorizer to {path}')
    with open(path, 'rb') as f_out:
        dv = pickle.load(f_out)
    
    # Load model as a PyFuncModel.
    logged_model = f'runs:/{RUN_ID}/model'
    
    model = mlflow.pyfunc.load_model(logged_model)


    ```
    -  Run `python predict.py`
    -  Run `python test.py`

## 2. Make pipeline for DictVectorizer and RandomForestRegressor

1. Make DictVectorizer and RandomForestRegressor to a pipline
   - In [random-forest_pipeline.ipynb](random-forest_pipeline.ipynb), replace 
    ```
    dv = DictVectorizer()
    model = RandomForestRegressor(**params, n_jobs=-1)

    X_train = dv.fit_transform(dict_train)
    model.fit(X_train, y_train)

    X_val = dv.transform(dict_val)
    y_pred = model.predict(X_val)

    # pipeline = make_pipeline(
    #     DictVectorizer(),
    #     RandomForestRegressor(**params, n_jobs=-1)
    # )

    # pipeline.fit(dict_train, y_train)
    # y_pred = pipeline.predict(dict_val)

    # rmse = mean_squared_error(y_pred, y_val, squared=False)
    # print(params, rmse)
    # mlflow.log_metric('rmse', rmse)
    #
    # mlflow.sklearn.log_model(pipeline, artifact_path="model")
    #
    rmse = mean_squared_error(y_pred, y_val, squared=False)
    print(params, rmse)
    mlflow.log_metric('rmse', rmse)

    mlflow.sklearn.log_model(model, artifact_path="model")

    with open ('dict_vectorizer.bin', 'wb') as f_out:
        pickle.dump(dv, f_out)
    mlflow.log_artifact('dict_vectorizer.bin')
   ```
   with 
    ```
    pipeline = make_pipeline(
        DictVectorizer(),
        RandomForestRegressor(**params, n_jobs=-1)
    )

    pipeline.fit(dict_train, y_train)
    y_pred = pipeline.predict(dict_val)

    rmse = mean_squared_error(y_pred, y_val, squared=False)
    print(params, rmse)
    mlflow.log_metric('rmse', rmse)

    mlflow.sklearn.log_model(pipeline, artifact_path="model")
    ```
   - replace [predict.py](predict.py) with [predict-pipline.py](predict-pipline.py)

2. Run `python predict-pipeline.py` and `python test.py`

## 3. New instance using s3 withouht connecting to tracking server 

1. Replace [predict-pipline-s3.py](predict-pipline-s3.py) with [predict-pipline.py](predict-pipline.py) in 
   ```
   RUN_ID = os.getenv('RUN_ID')
      
   logged_model = f's3://zoomcamp-mlops/1/{RUN_ID}/artifacts/model'

   model = mlflow.pyfunc.load_model(logged_model)
   ```
2. Go to Terminal and enter `export RUN_ID="YOUR_RUN_ID" `for setting the environment variable (can deploy for K8S).
3. Run `python predict-pipline-s3.py`, and jump to another window for running `python test.py`

   ![pipeline-s3.png](imgs%2Fpipeline-s3.png)