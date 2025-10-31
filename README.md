# Azure Functions scaffold for MCP SDK-based remote servers (Python)

Azure Functions allows you remotely host servers built with the official MCP SDKs with no code changes. If you've already built a server with the Python MCP SDK, use this scaffold to quickly host the server remotely. The scaffold contains the required Azure Function artifacts, as well as Bicep files for automatic infrastructure provisioning and server deployment. [Bicep](https://learn.microsoft.com/azure/azure-resource-manager/bicep/overview?tabs=bicep) is a domain-specific language that uses declarative syntax to deploy Azure resources.

>[!NOTE]
>The hosting capability is currently in **early preview**. 
>
>If you don't have an already-built server, refer to [the main sample](https://github.com/Azure-Samples/mcp-sdk-functions-hosting-python) to start from scratch. 

## Prerequisites 
* [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd)

## Server requirements 
Currently, your server must satisfy the requirements below for remote hosting on Azure Functions: 
1. Server is stateless
1. Server uses the streamable-http transport

## How to use this scaffold 
1. Inside the MCP server project, run `azd init --template self-hosted-mcp-scaffold-python`. In theory, there shouldn't be any duplicates between files in the scaffold and your original project. Fix duplicates if any by following the prompts. 
1. In `host.json`:
  - Put the main Python script path as the value of `arguments`, e.g. `weather.py`
  - Ensure the `port` value is the same as the one used by the MCP server
1. Follow instructions for local server testing and deployment in the [main sample repo](https://github.com/Azure-Samples/mcp-sdk-functions-hosting-python/tree/main?tab=readme-ov-file#prerequisites). Ensure you satisfy the other specified prerequisites before testing and deploying. 

## Walk-through of components in scaffold 
The scaffold has the following main components: 
1. `host.json`
1. `local.settings.json`
1. `infra` directory containing Bicep files for resource provisioning and server deployment

### host.json
This file is required for deployment. It's used by the Azure Functions host to configure the right settings (through the configuration profile) for the MCP server to run. It also tells the host how to run the server and which port to listen to. 

The default authorization level is `function`, which would require clients to pass an access key when connecting to the server. However, because we'll use a feature called Easy Auth, which provides an authorization flow with identity providers like Microsoft Entra ID, there's no need to require a key as well for server connection. 

```json
{
   "version": "2.0",
    "configurationProfile": "mcp-custom-handler",
    "customHandler": {
        "description": {
            "defaultExecutablePath": "python",
            "arguments": ["<path to main script file, e.g., weather.py>"] 
        },
        "http": {
            "DefaultAuthorizationLevel": "anonymous"
        },
        "port": "8000"
    }
}
```

### local.settings.json
This file is required if you want to run the server locally with Azure Functions Core Tools, which provides a local version of the Azure Functions host. It allows you to run your server locally as if it's run in the Azure Functions environment in the cloud. 

```json
{
    "IsEncrypted": false,
    "Values": {
        "FUNCTIONS_WORKER_RUNTIME": "python",
        "AzureWebJobsFeatureFlags": "EnableMcpCustomHandlerPreview"
    }
}
```

### Bicep files 
All Bicep files are in the `infra` directory. The `main.bicep` file specifies the provisioning of various Azure resources and their settings and/or configuration. Other than the Azure Functions app and the hosting plan (Flex Consumption) where the server is deployed to, other resources such as Azure Storage account, Application Insights, Virtual Network, Entra app, etc. are created as well. 
