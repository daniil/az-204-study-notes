### Using CLI

Logging in:

```
az login
```

### Deploying an app

```
az webapp up
```

Setting a `resourceGroup` variable:

```
az group list \
    --query "[].{id:name}" \
    -o tsv
```

Deploying an app with resourceGroup and appName variables provided:

```
az webapp up \
    -g $resourceGroup \
    -n $appName \
    --html
```

Creating a resource group with Creator tag:

```
az group create \
    --name $resourceGroup \
    --location eastus \
    --tags Creator="<name-or-id>"
```

Deleting group:

```
az group delete \
    --name $resourceGroup \
    --no-wait
```

### Application settings

Application settings can be found at app's management page under _Environment variables > Application settings_.

To add or edit app settings in bulk, select the Advanced edit button.

**For custom containers**:

```bash
az webapp config appsettings set \
    --resource-group <group-name> \
    --name <app-name> \
    --settings key1=value1 key2=value2
```

```PowerShell
Set-AzWebApp \
    -ResourceGroupName <group-name> \
    -Name <app-name> \
    -AppSettings @{"DB_HOST"="myownserver.mysql.database.azure.com"}
```

You can verify container environment variables with the URL https://<app-name>.scm.azurewebsites.net/Env.

**General settings**

In the _Configuration > General settings_ section you can configure some common settings for your app:

- Stack settings
- Platform settings
  - Platform bitness
  - FTP state
  - HTTP version
  - Web sockets
  - Always On
  - ARR affinity
  - HTTPS Only
  - Minimum TLS version
- Debugging
- Incoming client certificates

**Path mappings**

In the _Configuration > Path mappings_ section you can configure handler mappings, and virtual application and directory mappings.

Handler mappings let you add custom script processors to handle requests for specific file extensions.

You can add custom storage for your containerized app.

**Diagnostic logging**

Navigate to your app and select App Service logs.

To stream logs in the Azure portal, navigate to your app and select Log stream.

To stream logs live in Cloud Shell, use the following command:

```bash
az webapp log tail \
    --name appname \
    --resource-group myResourceGroup
```

### Scaling apps

Autoscaling performs scaling in and out, as opposed to scaling up and down. Autoscaling can be triggered according to a schedule, or by assessing whether the system is running short on resources. You can combine metric and schedule-based autoscaling in the same autoscale condition.

The metrics that can be monitored:

- CPU Percentage
- Memory Percentage
- Disk Queue Length
- Http Queue Length
- Data In
- Data Out

Autoscaling works in two steps (using values retrieved for a metric over a period of time): time aggregation over the time grain (ie: over 1 minute) and duration (aggregation of multiple time aggregations, ie: over 10 minutes).

The aggregation statistic options are Average, Minimum, Maximum, Sum, Last and Count. The most common is Average.

An autoscale action has a cool down period, in minutes.

All thresholds are calculated at an instance level.

An autoscale rules should be paired on how to scale out and in. Scaling out needs to meet one condition, where scaling in requires all conditions to be met. It's recommended to choose and adequate margin between the scale out and in treshholds (ie: +1 when CPU >= 80; -1 when CPU <= 60)

**Enable autoscale**

Under App Service plan, then _Settings > Scale out > Rules Based > Configure_.

The Azure portal enables you to track when autoscaling occurred through the _Run history_ chart. You can use the _Run history_ chart with the metrics shown on the _Overview_ page to correlate the autoscaling events with resource utilization.

### App deployment slots

You can swap deployment slots on your app's Deployment slots page and the Overview page.

To swap deployment slots:

- Go to your app's Deployment slots page and select Swap.
- Select the desired Source and Target slots. Usually, the target is the production slot. Also, select the Source Changes and Target Changes tabs and verify that the configuration changes are expected.
  - For preview follow the instruction in _Swap with preview_ (by selecting _Perform swap with preview_ checkbox)

**Configure auto swap**

- Go to your app's resource page and select the deployment slot you want to configure to auto swap. The setting is on the _Configuration > General settings_ page.

- Set _Auto swap enabled_ to On. Then select the desired target slot for _Auto swap deployment slot_, and select _Save_ on the command bar.

**Specify custom warm-up**

The `applicationInitialization` configuration element in web.config lets you specify custom initialization actions.

The swap operation waits for this custom warm-up to finish before swapping with the target slot. Here's a sample web.config fragment:

```
<system.webServer>
    <applicationInitialization>
        <add initializationPage="/" hostName="[app hostname]" />
        <add initializationPage="/Home/About" hostName="[app hostname]" />
    </applicationInitialization>
</system.webServer>
```

You can also customize the warm-up behavior with one or both of the following app settings:

- WEBSITE_SWAP_WARMUP_PING_PATH

  - The path to ping to warm up your site.
  - An example is /statuscheck. The default value is /.

- WEBSITE_SWAP_WARMUP_PING_STATUSES

  - Valid HTTP response codes for the warm-up operation. - An example is 200,202. By default, all response codes are valid.

- WEBSITE_WARMUP_PATH
  - A relative path on the site that should be pinged whenever the site restarts (not only during slot swaps).
  - Example values include /statuscheck or the root path, /.

**Creating new deployment slot**

```
az webapp deployment slot create \
    -n $appName \
    -g $resourceGroup \
    --slot staging
```

To view deployment slots, select _Deployment > Deployment slots_

To zip up contents of current folder for deployment:

```
zip -r stagingcode.zip .
```

**Deploying to a deployment slot**

```
az webapp deploy \
    -g $resourceGroup \
    -n $appName \
    --src-path ./stagingcode.zip \
    --slot staging
```

You can perform a swap in the Azure portal with the _Swap_ option in the toolbar.

The Swap option will appear in the toolbar if you select _Overview_ or _Deployment > Deployment_ slots in the left menu.

**Routing traffic**

To route production traffic automatically:

- Go to your app's resource page and select _Deployment slots_.
- In the _Traffic %_ column of the slot you want to route to, specify a percentage (between 0 and 100) to represent the amount of total traffic you want to route.

On the client browser, you can see which slot your session is pinned to by looking at the `x-ms-routing-name` cookie in your HTTP headers. A request routed to the production slot has the cookie `x-ms-routing-name=self`.

To route production traffic manually, you use the `x-ms-routing-name` query parameter, ie:

```
<a href="<webappname>.azurewebsites.net/?x-ms-routing-name=self">Go back to production app</a>
```

### Azure functions

Azure Functions supports _triggers_, which are ways to start execution of your code, and _bindings_, which are ways to simplify coding for input and output data.

When you create a function app in Azure, you must choose a hosting plan for your app:

- Consumption plan
- Flex Consumption plan
- Premium plan
- Dedicated plan
- Container Apps

The hosting option you choose dictates the following behaviors:

- How your function app is scaled.
- The resources available to each function app instance.
- Support for advanced functionality, such as Azure Virtual Network connectivity.
- Support for Linux containers.

**Function apps**

A function app provides an execution context in Azure in which your functions run. A function app is composed of one or more individual functions that are managed, deployed, and scaled together.

A Functions project directory contains the following files in the project root folder, regardless of language:

- host.json
  - Contains configuration options that affect all functions in a function app instance
- local.settings.json
  - Settings used by local development tools. When deployed, need to be configured in application settings.
- Other files in the project depend on your language and specific functions.

**Triggers and bindings**

A trigger defines how a function is invoked and a function must have exactly one trigger. Triggers have associated data, which is often provided as the payload of the function.

Binding to a function is a way of declaratively connecting another resource to the function; bindings might be connected as input bindings, output bindings, or both. Data from bindings is provided to the function as parameters.

Testing functions locally:

- For HTTP triggers, you can call the HTTP endpoint on the local computer.
- Use connection strings that target live Azure services
  - By adding the appropriate connection string settings in the `Values` array in the local.settings.json file
- For storage-based triggers, use the local Azurite emulator (ie: (Queue Storage, Blob Storage, and Table Storage))
- Manually run non-HTTP trigger functions by using special administrator endpoints

For JS/TS triggers and bindings are confiured by updating _function.json_ schema.

The portal provides a UI for adding bindings in the _Integration_ tab. You can also edit the file directly in the portal in the _Code + test_ tab of your function.

For languages that are dynamically typed such as JavaScript, use the `dataType` property in the function.json file. For example, to read the content of an HTTP request in binary format, set `dataType` to `binary`:

```JSON
{
    "dataType": "binary",
    "type": "httpTrigger",
    "name": "req",
    "direction": "in"
}
```

Other options for dataType are `stream` and `string`.

All triggers and bindings have a direction property in the _function.json_ file:

- For triggers, the direction is always `in`
- Input and output bindings use `in` and `out`
- Some bindings support a special direction `inout`. If you use `inout`, only the _Advanced editor_ is available via the _Integrate_ tab in the portal.

**Example**: write a new row to Azure Table storage whenever a new message appears in Azure Queue storage.

Here's a _function.json_ file for this scenario.

```JSON
{
  "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        },
        {
          "tableName": "Person",
          "connection": "MyStorageConnectionAppSetting",
          "name": "tableBinding",
          "type": "table",
          "direction": "out"
        }
  ]
}
```

**Connect functions to Azure services**

As a security best practice, Azure Functions takes advantage of the application settings functionality of Azure App Service to help you more securely store strings, keys, and other tokens required to connect to other services.

The default configuration provider uses environment variables. These variables are defined in application settings when running in the Azure and in the local settings file when developing locally.

**Configure an identity-based connection**

Some connections in Azure Functions are configured to use an identity instead of a secret. Support depends on the extension using the connection.

In some cases, a connection string might still be required in Functions even though the service to which you're connecting supports identity-based connections.

Identities must have permissions to perform the intended actions. This is typically done by assigning a role in Azure role-based access control, or specifying the identity in an access policy depending on the service to which you're connecting.

**Creating Azure function**

In VS Code, press `Cmd+Shift+P` and search for _Azure Functions: Create New Project...._

To debug run the function locally, open Terminal and press `Fn+F5`.

To test the function, go to the _Azure: Functions_ area. Under _Functions_, expand _Local Project > Functions_. Right-click the _AzureFunctionName_ function and select _Execute Function Now..._

To create the function in Azure, press `Cmd+Shift+P` and search for _Azure Functions: Create Function App in Azure..._

To deploy the function to Azure, press `Cmd+Shift+P` and search for _Azure Functions: Deploy to Function App..._

### Blob storage

Azure Blob storage is a cloud-based solution for storing large amounts of unstructured data (object storage).

An Azure Storage account is the top-level container for all of your Azure Blob storage. The storage account provides a unique namespace for your Azure Storage data that is accessible over HTTP or HTTPS.

Azure Storage provides different options for accessing block blob data based on usage patterns:

- Hot
  - Optimized for frequent access of objects
  - Highest storage costs, but the lowers access costs
- Cool
  - Optimized for storing large amounts of data that is infrequently accessed and stored for a minimum of 30 days
  - Lower storage costs and higher access costs compared to the Hot tier
- Cold
  - Optimized for storing data that is infrequently accessed and stored for a minimum of 90 days
  - Lower storage costs and higher access costs compared to the Cool tier
- Archive
  - Optimized for data that can tolerate several hours of retrieval latency and remains in the Archive tier for a minimum 180 days
  - Most cost-effective option for storing data, but accessing that data is more expensive than accessing data in the Hot or Cool tiers

Azure Blob Storage lifecycle management offers a rule-based policy that you can use to transition blob data to the appropriate access tiers or to expire data at the end of the data lifecycle.

Blob storage offers three types of resources:

- The storage account
  - Provides a unique namespace in Azure for your data
  - An address that includes your unique account name
  - ie: http://mystorageaccount.blob.core.windows.net
- A container in the storage account
  - Organizes a set of blobs, similar to a directory in a file system
  - ie: https://myaccount.blob.core.windows.net/mycontainer
- A blob in a container
  - Block blobs store text and binary data. Block blobs can store up to about 190.7 TiB.
  - Append blobs are made up of blocks like block blobs, but are optimized for append operations. Append blobs are ideal for scenarios such as logging data from virtual machines.
  - Page blobs store random access files up to 8 TB in size. Page blobs store virtual hard drive (VHD) files and serve as disks for Azure virtual machines.
  - ie: https://myaccount.blob.core.windows.net/mycontainer/myblob

**Creating storage account via CLI**

```
az storage account create \
    --resource-group $resourceGroup \
    --name <myStorageAcct> \
    --location <myLocation> \
    --sku Standard_LRS
```

**Azure Storage security features**

Azure Storage uses service-side encryption (SSE) to automatically encrypt your data when it's persisted to the cloud.

Data in Azure Storage is encrypted and decrypted transparently using 256-bit Advanced Encryption Standard (AES) encryption (Federal Information Processing Standards (FIPS) 140-2 compliant).

Encryption key management:

- Microsoft-managed keys by default
- Customer-managed key
  - Must be stored in Azure Key Vault or Azure Key Vault Managed Hardware Security Model (HSM).
- Customer-provided key
  - A client can include an encryption key on a read/write request for granular control over how blob data is encrypted and decrypted

Client-side encryption

The Azure Blob Storage client libraries for .NET, Java, and Python support encrypting data within client applications before uploading to Azure Storage, and decrypting data while downloading to the client.

The Queue Storage client libraries for .NET and Python also support client-side encryption.

The Blob Storage and Queue Storage client libraries uses AES in order to encrypt user data. There are two versions of client-side encryption available in the client libraries:

- Version 2 uses Galois/Counter Mode (GCM) mode with AES. The Blob Storage and Queue Storage SDKs support client-side encryption with v2.
- Version 1 uses Cipher Block Chaining (CBC) mode with AES. The Blob Storage, Queue Storage, and Table Storage SDKs support client-side encryption with v1.

**Storage lifecycle**

General Purpose v2 storage account types support lifecycle policies.

With the lifecycle management policy, you can:

- Transition blobs from cool to hot immediately when accessed, to optimize for performance.
- Transition current versions of a blob, previous versions of a blob, or blob snapshots to a cooler storage tier if these objects aren't accessed or modified for a period of time, to optimize for cost.
- Delete current versions of a blob, previous versions of a blob, or blob snapshots at the end of their lifecycles.
- Apply rules to an entire storage account, to select containers, or to a subset of blobs using name prefixes or blob index tags as filters.

A lifecycle management policy is a collection of rules in a JSON document. Each rule definition within a policy includes a filter set and an action set, ie:

```JSON
{
  "rules": [
    {
      "enabled": true,
      "name": "sample-rule",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "version": {
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          },
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90,
              "daysAfterLastTierChangeGreaterThan": 7
            },
            "delete": {
              "daysAfterModificationGreaterThan": 2555
            }
          }
        },
        "filters": {
          "blobTypes": [
            "blockBlob"
          ],
          "prefixMatch": [
            "sample-container/blob1"
          ]
        }
      }
    }
  ]
}
```

Rule filters:

- `blobTypes`
- `prefixMatch`
- `blobIndexMatch`

Rule actions:

- `tierToCool`
- `tierToCold`
- `enableAutoTierToHotFromCool`
- `tierToArchive`
- `delete`

Rule run conditions:

- `daysAfterModificationGreaterThan`
- `daysAfterCreationGreaterThan`
- `daysAfterLastAccessTimeGreaterThan`
- `daysAfterLastTierChangeGreaterThan`

**Implement policies**

Azure Portal

- In the Azure portal, navigate to your storage account.
- Under Data management, select Lifecycle Management to view or change lifecycle management policies.
- Select the Code View tab. On this tab, you can define a lifecycle management policy in JSON.

CLI

Write the policy to a JSON file, then call:

```
az storage account management-policy create \
    --account-name <storage-account> \
    --policy @policy.json \
    --resource-group <resource-group>
```

**Rehydrate blob data from the archive tier**

There are two options for rehydrating a blob that is stored in the archive tier:

- Copy an archived blob to an online tier (recommended)
- Change a blob's access tier to an online tier

You can set the priority for the rehydration operation via the optional `x-ms-rehydrate-priority` header on a Set Blob Tier or Copy Blob/Copy Blob From URL operation:

- Standard
  - Processed in the order it was received, might take up to 15 hours
- High
  - Prioritized over standard and might complete in under one hour

**Working with Azure Blob storage**

An authorized `BlobServiceClient` object allows your app to interact with resources at the storage account level.

`BlobServiceClient` provides methods to retrieve and configure account properties, as well as list, create, and delete containers within the storage account.

This client object is the starting point for interacting with resources in the storage account.

```C#
using Azure.Identity;
using Azure.Storage.Blobs;

public BlobServiceClient GetBlobServiceClient(string accountName)
{
    BlobServiceClient client = new(
        new Uri($"https://{accountName}.blob.core.windows.net"),
        new DefaultAzureCredential());

    return client;
}
```

You can use a `BlobServiceClient` object to create a new `BlobContainerClient` object. A `BlobContainerClient` object allows you to interact with a specific container resource.

`BlobContainerClient` provides methods to create, delete, or configure a container, and includes methods to list, upload, and delete the blobs within it.

```C#
public BlobContainerClient GetBlobContainerClient(
    BlobServiceClient blobServiceClient,
    string containerName)
{
    // Create the container client using the service client object
    BlobContainerClient client = blobServiceClient.GetBlobContainerClient(containerName);
    return client;
}
```

You might choose to create a `BlobContainerClient` object directly without using `BlobServiceClient`.

```C#
public BlobContainerClient GetBlobContainerClient(
    string accountName,
    string containerName,
    BlobClientOptions clientOptions)
{
    // Append the container name to the end of the URI
    BlobContainerClient client = new(
        new Uri($"https://{accountName}.blob.core.windows.net/{containerName}"),
        new DefaultAzureCredential(),
        clientOptions);

    return client;
}
```

To interact with a specific blob resource, create a `BlobClient` object from a service client or container client. A `BlobClient` object allows you to interact with a specific blob resource.

```C#
public BlobClient GetBlobClient(
    BlobServiceClient blobServiceClient,
    string containerName,
    string blobName)
{
    BlobClient client =
        blobServiceClient.GetBlobContainerClient(containerName).GetBlobClient(blobName);
    return client;
}
```

To find the connection string go to _Security + networking > Access Keys_.

**Container properties and metadata**

Blob containers support system properties and user-defined metadata. Metadata name/value pairs are valid HTTP headers, and so should adhere to all restrictions governing HTTP headers.

The metadata consists of name/value pairs. The total size of all metadata pairs can be up to 8 KB in size.

Metadata values can only be read or written in full; partial updates aren't supported. Setting metadata on a resource overwrites any existing metadata values for that resource.

To retrieve container properties, call one of the following methods of the BlobContainerClient class:

- `GetProperties`
- `GetPropertiesAsync`

```C#
private static async Task ReadContainerPropertiesAsync(BlobContainerClient container)
{
    try
    {
        // Fetch some container properties and write out their values.
        var properties = await container.GetPropertiesAsync();
        Console.WriteLine($"Properties for container {container.Uri}");
        Console.WriteLine($"Public access level: {properties.Value.PublicAccess}");
        Console.WriteLine($"Last modified time in UTC: {properties.Value.LastModified}");
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine($"HTTP error code {e.Status}: {e.ErrorCode}");
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

You can specify metadata as one or more name-value pairs on a blob or container resource:

- `SetMetadata`
- `SetMetadataAsync`

```C#
public static async Task AddContainerMetadataAsync(BlobContainerClient container)
{
    try
    {
        IDictionary<string, string> metadata =
           new Dictionary<string, string>();

        // Add some metadata to the container.
        metadata.Add("docType", "textDocuments");
        metadata.Add("category", "guidance");

        // Set the container's metadata.
        await container.SetMetadataAsync(metadata);
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine($"HTTP error code {e.Status}: {e.ErrorCode}");
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

Metadata headers can be set on a request that creates a new container or blob resource, or on a request that explicitly creates a property on an existing resource.

The format for the header is:

```
x-ms-meta-name:string-value
```

The GET and HEAD operations both retrieve metadata headers for the specified container or blob:

```
GET/HEAD https://myaccount.blob.core.windows.net/mycontainer?restype=container
GET/HEAD https://myaccount.blob.core.windows.net/mycontainer/myblob?comp=metadata
```

The PUT operation sets metadata headers on the specified container or blob, overwriting any existing metadata on the resource:

```
PUT https://myaccount.blob.core.windows.net/mycontainer?comp=metadata&restype=container
PUT https://myaccount.blob.core.windows.net/mycontainer/myblob?comp=metadata
```

Containers and blobs also support certain standard HTTP properties.

The standard HTTP headers supported on containers include:

- `ETag`
- `Last-Modified`

The standard HTTP headers supported on blobs include:

- `ETag`
- `Last-Modified`
- `Content-Length`
- `Content-Type`
- `Content-MD5`
- `Content-Encoding`
- `Content-Language`
- `Cache-Control`
- `Origin`
- `Range`
