# Offline - Batch

## Table of contents

1. Obtain the Model and Build the Pipeline

2. Turn Script into Prefect Flow

3. Deploy Prefect Flow

4. Apply Model to Previous Months: Backfilling

5. Notes

## 1. Obtain the model and build the pipeline 
    
    **Step 1**: organize [score.ipynb](score.ipynb)

    **Step 2**: transfer [score.ipynb](score.ipynb) to score.py with `jupyter nbconvert --to script score.ipynb`
    
    **Step 3**: Run [score.py](score.py) with  `python score.py green 2021 4 YOUR_RUN_ID`

## 2. Turn script into Prefect flow

1. Build a virtual environment

2. Jump to `Batch` folder with `cd 04-deployment/Batch/`

3. Install virtual environment with `pipenv --python=3.10`

4. Install dependency with `pipenv install prefect==2.0b6 mlflow scikit-learn==1.0.2 pandas boto3 pyarrow s3fs`

5. Activate with `pipenv shell`
    
    5.1. Build prefect flow [score.py](score.py)
    
    5.2. Open another terminal and start prefect with `pipenv shell`, then `prefect orion start`
    
    5.3. Run `score.py` with `python score.py green 2021 11 7c8bb75fadea44aa9337ec3de35c430e `

![result.png](..%2Fimages%2Fresult.png)

## 3. Deploy Prefect flow

    **Step 1**: Create deploy file as [score_deploy.py](score_deploy.py)
    
    **Step 2**: Create prefect local storage with `prefect storage create`. Config:
```
storage types: 3) Local Storage
Select a storage type to create: /Users/amberm/PycharmProjects/MLops/04-deployment/Batch/tmp
Choose a name for this storage configuration: local_tmp
```
    **Step 3**: deploy prefect with `prefect deployment create score_deploy.py` and go to prefect UI to check if it works.
    
    **Step 4**: Create work queue on prefect UI with name as `ride_duration_queue` and obtained `Work Queue ID`.

## 4. Apply model to previous months: Backfilling
***Note: Fill and pull the date data from AWS***
    
    **Step 1**: Create `score_backfil.py` [score_backfil.py](score_backfil.py)
    
    **Step 2**:Run `python score_backfil.py ` to 
    
    **Step 3**: Start agent with `prefect agent start 55fc5cc0-d414-45ee-9d30-7bd5e70e8d0a' which is looking for work from the provided queue.



## 5. Notes

### 1. HTTPError: HTTP Error 403: Forbidden when call apply_model() in score.ipynb

**Solution**: instead of input_file = f'https://s3.amazonaws.com/nyc-tlc/trip+data/{taxi_type}_tripdata_{year:04d}-{month:02d}.parquet'  use input_file = f'https://d37ci6vzurychx.cloudfront.net/trip-data/{taxi_type}_tripdata_{year:04d}-{month:02d}.parquet'

### 2. When creating prefect storage, error:

```sqlalchemy.exc.OperationalError: (sqlite3.OperationalError) no such table: block_spec
[SQL: ALTER TABLE block_spec RENAME TO block_schema]
(Background on this error at: https://sqlalche.me/e/14/e3q8)```

**Solution**:
```
pipenv run pip install SQLAlchemy==1.4.49 
rm ~/.prefect/orion.db
pipenv shell        
prefect orion database reset -y 
prefect orion start  

```

