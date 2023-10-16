# MLflow-Prefect

The repository discusses on how to build, track and orchestrate end-to-end machine learning pipelines using MLflow, Hyperopt, and Prefect.

- MLflow is a platform for managing the entire machine learning lifecycle, from experiment tracking to model deployment. 

- Hyperopt is a library for hyperparameter optimization, which can help you to find the best hyperparameters for your machine learning models. 

- Prefect is a workflow management tool that can help you to automate your machine learning pipelines.

Install the libraries using this command

`pip install mlflow prefect hyperopt`

## Step 1: Launch a mlflow server

To launch a mlflow server use this command `mlflow server --backend-store-uri sqlite:///backend.db --default-artifact-root ./mlruns`

- --backend-store-uri sqlite:///backend.db: specifies the backend store URI where the MLflow server should persist metadata related to experiments, runs, parameters, metrics, and artifacts. In this case, the backend store uses an SQLite database file named backend.db.
- --default-artifact-root ./mlruns: specifies the default artifact store location where the MLflow server should store artifacts generated by runs. In this case, the default artifact store location is the ./mlruns directory relative to the current working directory.

## Step 2: Launch a prefect server

To launch a prefect server use this command `prefect server start`

The Prefect Server is a central daemon that provides a variety of features for managing and executing Prefect flows.

Learn more about flows at: https://docs.prefect.io/2.13.7/concepts/flows/

To view the Prefect UI, open a web browser and navigate to http://127.0.0.1:4200/

## Quick explanation of the whole app.py

- The @task decorator tells Prefect that this function is a task.
- The mlflow_environment function connects to the server with the given experiment name and returns and experiment id.
- The next two functions i.e. two prefect tasks are load and split data
- The next Prefect task is called train_hyperparameter_tuning that performs hyperparameter tuning for an XGBoost classifier on a given dataset.
- The next task is all about getting the best model.
- And then a task to promote the best model to production is given
- As mlflow can also serve models we have the mlflow serve function used to serve the model embedded inside a prefect task.


The @flow decorator is used to define the main function as a Prefect flow.

The if __name__ == "__main__": block is used to define the deployment of the Prefect flow using Deployment.build_from_flow. T
his creates a new deployment of the main flow with the following configuration:

- Name: "model_training_and_tuning_weekly"
- Parameters: {'experiment_name':'digits_experiment', 'model_name':'xgboost'}
- Schedule: At 12:00 AM, only on Thursday, in the "Africa/Cairo" timezone
- Version: 1
- Work Queue Name: "ml"

The apply method is used to apply the deployment, which registers the flow and its configuration in the Prefect backend and schedules it to run according to the defined schedule.

Run the following command in cmd

`prefect agent start --pool default-agent-pool --work-queue ml`

The prefect agent start command starts a new Prefect agent process.

The --pool flag specifies the name of the pool that the agent should use for executing flows. In this case, the pool is named default-agent-pool.

The --work-queue flag specifies the name of the work queue that the agent should use for receiving work. In this case, the work queue is named ml.

app.py script contains a Prefect flow defined with the @flow decorator, the flow will be registered with the Prefect backend and scheduled to run according to its defined schedule.

Now finally run the pipeline with `python app.py`

In the Cron schedule, the time format is as seconds/minutes/hours

To start a run itself at the current movement use the run button in the deployments menu.

![Prefect run](https://github.com/ChiragChauhan4579/MLflow-Prefect/blob/main/images/deployment%20section.png)
