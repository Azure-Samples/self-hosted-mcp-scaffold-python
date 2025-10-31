# Azure Functions scaffold for MCP SDK-based remote servers (Python)

Azure Functions allows you remotely host servers built with the official MCP SDKs with no code changes. If you've already built a server with the Python MCP SDK, use this scaffold to quickly host the server remotely. The scaffold contains the required Azure Function artifacts, as well as Bicep files for automatic infrastructure provisioning and server deployment. [Bicep](https://learn.microsoft.com/azure/azure-resource-manager/bicep/overview?tabs=bicep) is a domain-specific language that uses declarative syntax to deploy Azure resources.

Follow instructions in the [mcp-sdk-functions-hosting-python repo](https://github.com/Azure-Samples/mcp-sdk-functions-hosting-python/tree/main?tab=readme-ov-file#prerequisites) to leverage this scaffold for hosting remote MCP servers. 

>[!NOTE]
>The hosting capability is currently in **early preview**. 

## Walk-through of components 
The repo has the following main components: 
1. `host.json`
1. `local.settings.json`
1. `infra` directory containing Bicep files for resource provisioning and server deployment

### host.json
This file is required by Azure Functions for deployment. It's used by the Azure Functions host to configure the right settings (through the configuration profile) for the MCP server to run. It also tells the host how to run the server and which port to listen to. 

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
This file is required only if you want to run the server locally with Azure Functions Core Tools, which provides a local version of the Azure Functions host. It allows you to run your server locally as if it's run in the Azure Functions environment in the cloud. 

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
