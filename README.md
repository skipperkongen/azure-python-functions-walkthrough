# Azure Python Functions Walkthrough

Here is a quick walkthrough for how to create a Python based Azure Functions app.
After completing the steps you will have a Python function hosted in the Azure cloud
that you can call via HTTP, e.g. with curl.

> Disclaimer: This walkthrough works and produces an endpoint you can call.
However, at the time of writing it seems that Azure API Management, which adds
nice features, such as caching and routing, does not support Functions written
in Linux/Python. It is my expectation that this problem will resolve itself swiftly.

## Create function

Initialise project:

```
mkdir azure-functions-hello
cd azure-functions-hello
pyenv local 3.6.8
pipenv install
func init HelloApp
```

Create a function:

```
cd HelloApp
# Create new function
# Choose option 5, i.e. HTTP trigger, and leave the default name (HttpTrigger)
func new
```

This creates the structure:

```
.
├── .python-version
├── HelloApp
│   ├── .gitignore
│   ├── .vscode
│   │   └── extensions.json
│   ├── HttpTrigger
│   │   ├── __init__.py
│   │   └── function.json
│   ├── host.json
│   ├── local.settings.json
│   └── requirements.txt
├── Pipfile
└── Pipfile.lock
```

Test function locally (in `HelloApp` directory):

```
func host start
# Test with GET
curl http://localhost:7071/api/HttpTrigger?name=Bob
# Test with POST
curl -H 'content-type' -d '{"name": "Bob"}' http://localhost:7071/api/HttpTrigger?name=Bob
```

(Optional) change the code of the function (in `HelloApp` directory):

```
vi HttpTrigger/__init__.py  # use editor of choice
```

## Deploy function to Azure

Create resources and deploy function (in `HelloApp` directory):

```
az login

# The ID you choose will be used to name resource groups etc.
UNIQUE_ID_FOR_APP=<some unique name, e.g. helloapp777>

az group create --name $UNIQUE_ID_FOR_APP --location westeurope

az storage account create --name $UNIQUE_ID_FOR_APP --location westeurope \
--resource-group $UNIQUE_ID_FOR_APP --sku Standard_LRS

az functionapp create --resource-group $UNIQUE_ID_FOR_APP --os-type Linux \
--consumption-plan-location westeurope  --runtime python \
--name $UNIQUE_ID_FOR_APP --storage-account $UNIQUE_ID_FOR_APP

func azure functionapp publish
```

Test your freshly deployed function:

```
curl https://$UNIQUE_ID_FOR_APP.azurewebsites.net/api/httptrigger?code=<autogenerated code>\&name=Bob
```

## Clean up

Destroy the Azure resources you created:

```
az group delete --name $UNIQUE_ID_FOR_APP
```
