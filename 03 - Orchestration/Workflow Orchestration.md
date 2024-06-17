# 3. Workflow Orchestration #

## Table of contents


**1. MLOps Workflow**

**2. Introduction of Prefect**
     - Install Prefect
     - Run Simple Example

**3. Run a Workflow**

**4. Deploying Your Workflow**

**5. Working with Deployments**



This section is mainly about [Work Pools, Workers & Agents](https://docs.prefect.io/2.10.17/concepts/work-pools/). At the beginning, it introduces what is **prefect** and why we need it. Then in 3.2 it introduces the basic use of prefect.

Start from 3.3, it is to transfer our previous work([02_xgboost.ipynb](..%2F02-experiment-tracking%2F02_xgboost.ipynb)) with prefect workflow.
The flow will be followed as:
![de_flow.png](images%2Fde_flow.png)

# 1. MLOps Workflow

```
Database -- Pandas -- scikit learn --> dmlc XGBoost
               |            |               |
               |            |_______________|
               |                    |
             Parquet              MLflow
                                    |
                                   Flask
```

# 2. Introduction of Prefect

[Prefect](https://github.com/PrefectHQ/prefect/tree/main) is a tool that helps **manage and organize tasks and workflows**, especially when dealing with a lot of data. It transforms any Python function into a unit of work that can be observed and orchestrated, making it easy to build workflows that can adapt to changes and recover from unexpected issues.

### Key Features of Prefect:

- **Automatic Retries:** Prefect can automatically try again if a task fails.

- **Distributed Execution:** Tasks can be spread across multiple machines.

- **Scheduling:** Tasks can be set to run at specific times.

- **Caching:** Results of tasks can be stored and reused.

- **Monitoring:** Every activity can be tracked and monitored using Prefect’s tools.

### Why Use Prefect?

Prefect simplifies the process of building and managing workflows. Instead of writing a lot of extra code to handle things like retries, scheduling, and monitoring, you can use Prefect’s features, making your code cleaner and easier to maintain.

### The Central Components of Prefect Workflows

#### Flows
A **flow** is the main structure in Prefect workflows. It’s a Python function marked with a `@flow` decorator. Key points about flows:
- All Prefect workflows are built around flows.
- Each workflow must have at least one flow function, which is the main entry point for running the workflow.
- Flows can include calls to tasks and subflows (child flows). This is similar to organizing a Python application where you break down the work into specific tasks and call them within the main function.

#### Tasks
A **task** is a smaller unit of work within a flow, marked with a `@task` decorator. Tasks represent individual pieces of work that the flow executes.

Using these components, you can build and manage complex workflows easily. Prefect takes care of executing the tasks, handling retries, and monitoring the workflow, so you can focus on the actual logic of your tasks and flows.


## Install Prefect

[Installing the latest version](https://docs.prefect.io/2.10.13/getting-started/installation/)


## Run simple example
*All the material is in the folder with the name of the section*
Create `03-orchestration` folder.
1. Open one terminal. Create a CONDA environment, and activate it.
    ```
    conda create -n prefect-ops 
    conda activate prefect-ops
    ```
2. Install the requirements for the project:
    
        - black==23.3.0
        
        - fastparquet==2023.4.0
        
        - hyperopt==0.2.7
        
        - mlflow==2.3.1
        
        - pandas==2.0.1
        
        - prefect==2.10.8
        
        - prefect-aws==0.3.1
        
        - scikit_learn==1.2.2
        
        - seaborn==0.12.2
        
        - xgboost==1.7.5
        
        - orjson==3.8.1
  
    `pip install -r requirements`
3. Open prefect server

    `prefect server start`
    
4. Check out the dashboard at http://127.0.0.1:4200

5. Open another terminal to run the sample

    `python hello_prefect.py`

    By adding the @flow decorator to a function, function calls will create a flow run — the Prefect orchestration engine manages flow and task state regardless of where your flow code runs.
    
6. After successfully run the sample, the dashboard will be:
   Red: Failed cases; Green: Successful cases

### Notes
*There is a `cat_facts.py`.

Since it does not work  (***Task run 'fetch_cat_fact-0' - [Errno 8] nodename nor servname provided, or not known***), just replace the address with `https://google.com`, and try it again.*


# 3. Run a work flow
```
├── duration_prediction_explore.ipynb
├── duration_prediction_original.ipynb
├── orchestrate.py
└── orchestrate_pre_prefect.py
```
- `duration_prediction_original.ipynb`: Original material
- `duration_prediction_explore.ipynb`: An exploration based on `duration_prediction_original.ipynb` for looking at the returned types.
- `orchestrate_pre_prefect.py`: the orchestration of the work. 
- `orchestrate.py`: Using `prefect` for the orchestration of the work with introducing `@task` and `@flow`(based on`orchestrate_pre_prefect.py`).

Differences between `orchestrate_pre_prefect.py` and `orchestrate.py`:

![diff_orch.png](images%2Fdiff_orch.png)
Then, run `prefect server start` in Terminal, and it can be:

![de_server.png](images%2Fde_server.png)

Then jump to the dashboard at ` http://127.0.0.1:4200`

![de_ui.png](images%2Fde_ui.png)

# 4. Deploying your Workflow

1. Jump to 3.4 folder. 
2. Copy `orchestrate.py` from 3.3. 
3. Open Terminal and check your GIT remote repo. (PUSH your updated local repo to remote right now!! )
4. Make sure you are under MLops (root) folder and create deployment files.

    `prefect project init` 

    The page should be as follows:

    ![yaml.png](images%2Fyaml.png)

5. Create a work pool with prefect UI. 

    `Name - 'zoompool', Type - 'process'`
6. Go back to terminal and deploy the task.

    `prefect deploy 03-orchestration/3.4/orchestrate.py:main_flow -n taxi1 -p zoompool`

    ![de_deploy.png](images%2Fde_deploy.png)
7.  Start a worker that pulls work from the `zoompool` work pool

   `prefect worker start -p zoompool`

8. Deploy flow from the UI: click on quick run for the flow you deployed on prefect. (This step can also be achieved by CLI)


# 5. Working with Deployments

1. Create a file named as `create_s3_bucket_block.py` in 3.5 folder
2. Open AWS IAM(Identity and Access Management (IAM))
- Step 1 Specify user details: 

  * Go to the right side bar, and click 'Users' to creat a new IAM user. 

  * Name the new IAM user as `mlops-zoom-user`, and then click on 'NEXT'.

- Step 2 Set permissions:
  * Create a new group and choose `AmazonS3FullAccess` as 'Permissions Policies', then 'NEXT'.
  ![s3group.png](images%2Fs3group.png)
- Step 3 Review and create:
  * Click on 'Creat User'
3. Go back to IAM, and click on 'Users'
- Stetp 1 Access key best practices & alternatives:
  - Jump to the new user we created. Go to `Security Cedentials` to `create access key`
- Step 2 (optional) Set description tag:
  - Choose `Other`, then `Next` to the end
- Step 3 Retrieve access keys:
  - Copy `Access key` and `Secret access key` and paste them to `create_s3_bucket_block.py`.
4. Go to Terminal, jump to root folder and run the file. Then open the prefect UI, you will see two blocks are generated in prefect.

   `python 03-orchestration/3.5/create_s3_bucket_block.py` 
   ![s3bucket.png](images%2Fs3bucket.png)

5. Create artifacts Upload dataset to AWS S3 
In [orchestrate_s3.py](3.5%2Forchestrate_s3.py)

    1). Load data from s3 bucket in main flow
   ```
   # your bucket name
   s3_bucket_block = S3Bucket.load("zoomcamp-mlops")
   s3_bucket_block.download_folder_to_path(from_folder = "data", to_folder = "data")
   ```
    2). Run `orchestrate_s3.py`

    3). Deployment of the flow.
    - Open `deployment.yaml`
      ```
      deployments:
      - name: taxi_local_data
        entrypoint: 03-orchestration/3.4/orchestrate.py:main_flow
        work_pool:
          name: zoompool
      - name: taxi_s3_data
        entrypoint: 03-orchestration/3.5/orchestrate_s3.py:main_flow_s3
        work_pool:
          name: zoompool
      ```
   - Deploy the flow after configuation
     `prefect deploy --all`
   - Deploy artifacts by adding `create_markdown_artifact` first, and then `prefect deployment run main-flow-s3/taxi_s3_data`

    ![s3artifacts.png](images%2Fs3artifacts.png)

    6). Schedule ([Official Docs](https://docs.prefect.io/2.10.17/concepts/schedules/))
   - You can add, modify, and view schedules by selecting Edit under the three dot menu next to a Deployment in the Deployments tab of the Prefect UI.
   - Creating schedules through a Python deployment file
