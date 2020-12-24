
# Cloud Based Machine Learning Production Model and Pipeline on Azure

In this project, we are using the Bank Marketing dataset to create a cloud-based machine learning production model and a pipeline on Azure Machine Learning. We will create a model with Auto Machine Learning, deploy it, and consume it. We also will  create, publish, and consume a pipeline. We will describe our work in this README file and in a screencast video.

 

![img](media/step-diagram.png)

**Main Steps**:

1. Authentication (required to interact with Azure ML Workspace)
2. Automated ML Experiment
3. Deploy the best model
4. Enable logging
5. Swagger Documentation
6. Consume model endpoints
7. Create and publish a pipeline
8. Documentation

## Key Steps

### Step 1: Authentication

In general, there three authentication workflows that you can use when connecting to the workspace:

- **Interactive**: You use your account in Azure Active Directory to either directly authenticate, or to get a token that is used for authentication. Interactive authentication is used during *experimentation and iterative development*. Interactive authentication enables you to control access to resources (such as a web service) on a per-user basis.
- **Service principal**: You create a service principal account in Azure Active Directory, and use it to authenticate or get a token. A service principal is used when you need an *automated process to authenticate* to the service without requiring user interaction. For example, a continuous integration and deployment script that trains and tests a model every time the training code changes.
- **Managed identity**: When using the Azure Machine Learning SDK *on an Azure Virtual Machine*, you can use a managed identity for Azure. This workflow allows the VM to connect to the workspace using the managed identity, without storing credentials in Python code or prompting the user to authenticate. Azure Machine Learning compute clusters can also be configured to use a managed identity to access the workspace when *training models*.

see [Set up authentication - Azure Machine Learning | Microsoft Docs](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-setup-authentication) for more information.

#### How To Get Service Principal Client ID

Prerequisites:

- AZ CLI Must be Installed (see [Install the Azure CLI for Windows | Microsoft Docs](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-cli))
- The `azure-cli-ml` extension is added or already installed.

##### Get Service Principal Client ID

1. Login to Azure with an account with "subscription Administrator" privileges`AZ Login`

   ![image-20201224094106756](media/az_login.png)

2. Create Service Principal `az ad sp create-for-rbac --sdk-auth --name <USERNAME>`

   ![image-20201224094345813](media/image-20201224094345813.png)

3. Take note of th the `"objectId"` in the `clientId`field. 



### Step 2: Automated ML Experiment

In this step, you we created an experiment using **Automated ML**, configured a compute cluster, and used that cluster to run the experiment.

<u>Bank Market Dataset Registered</u> 

![image-20201216162015134](media/registerd_dataset.png)

After the experiment run completes, a summary of all the models and their metrics are shown, including explanations. The *Best Model* is shown in the *Details* tab.

![image-20201216164732050](media/experiment_completed.png)

The "best model" can be easily selected and analized and deployed.

![image-20201216165148357](media/best_model.png)

### Step 3: Deploy the Best Model

Deploying the Best Model will allow to interact with the HTTP API service and interact with the model by sending data over POST requests.

![image-20201216165148357](media/best_model.png)

### Step 4: Enable Application Insights



Now that the *Best Model* is deployed, enable Application Insights and retrieve logs. Although this is configurable at deploy time with a check-box, it is useful to be able to run code that will enable it for you. In this step we demonstrate how to enable *Application Insights* and how to retrieve logs running [logs.py](logs.py) python script.

```python
service = Webservice(name=name, workspace=ws)
logs = service.get_logs()
service.update(enable_app_insights=True)

for line in logs.split('\n'):
    print(line)
```

Logs are shown immediately and the detail panel shows "Application Insights Enabled" = true

![image-20201216172259901](media/log_py_output.png)

![image-20201216172353552](media/app_insight_enabled.png)

1. ### Consume model endpoints

2. Create and publish a pipeline

3. Documentation

Consuming endpoint log

![image-20201216172723130](media/endpoint_log.png)

### Step 5: Swagger Documentation

In this step, we consumed the deployed model using Swagger.

Azure provides a [Swagger JSON file](https://swagger.io/) for deployed models. Head to the *Endpoints* section, and find your deployed model there, it should be the first one on the list.

The easyest way to run swagger locally is from the official docker image. The `swagger.sh` script will download the latest Swagger container, and it will run it on port 9000. 

```
docker pull swaggerapi/swagger-ui
docker run -p 9000:8080 swaggerapi/swagger-ui
```

![image-20201216174431317](media/swagger_running.png)

This script [serve.py](serve.py) creates an HTTP server to expose the current working directory. It is meant to be an easy way to expose a local swagger.json file so that a swagger-ui service can pick it up from localhost:8000

```python
class CORSRequestHandler(SimpleHTTPRequestHandler):
    """
    Allows a simple HTTP server to have CORS enabled by default
    """

    def end_headers(self):
        self.send_header("Access-Control-Allow-Origin", "*")
        SimpleHTTPRequestHandler.end_headers(self)


if __name__ == "__main__":
    if len(sys.argv) > 1:
        # Allows the port to be passed in as an argument
        port = sys.argv[-1]
    else:
        port = 8000

    test(CORSRequestHandler, HTTPServer, port=port)
```

Now Swagger can be used to explore and consume the swagger.json file from Azure endpoint

![image-20201216174555268](media/endpoint_local_swagger.png)

### Step 6: Consume Model Endpoints

Once the model is deployed,  The [endpoint.py](endpoint.py) script is used to demonstrate an interaction with the trained model. In the script the `scoring_uri` and the `key`must match the key for your service and the URI that was generated after deployment and that are visible in the "Details Tab"

![image-20201216175000883](media/consume_endpoint.png)

### Step 7: Create, Publish and Consume a Pipeline

For this part of the project, the [aml-pipelines-with-automated-machine-learning-step.ipynb](aml-pipelines-with-automated-machine-learning-step.ipynb) Jupyter Notebook has been used to create, publish and consume a pipeline.

![image-20201217123316161](media/pipeline_created.png)

![image-20201217123235696](media/pipeline_details.png)

Run Details

![image-20201217123752558](media/run_details.png)

Experiment Run

![image-20201217124028455](media/experiment_run.png)

![image-20201217124627570](media/pipeline_endpoint.png)

## Screen Recording



*TODO* Provide a link to a screen recording of the project in action. Remember that the screencast should demonstrate:

## Standout Suggestions
*TODO (Optional):* This is where you can provide information about any standout suggestions that you have attempted.
