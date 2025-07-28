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

To generage random variation for any variable `$RANDOM` can be used:

```
accountName=azureexercise$RANDOM
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

### Cosmos DB

Azure Cosmos DB is a fully managed NoSQL database designed to provide low latency, elastic scalability of throughput, well-defined semantics for data consistency, and high availability. You can configure your databases to be globally distributed and available in any of the Azure regions. Azure Cosmos DB offers 99.999% read and write availability for multi-region databases.

**Elements in an Azure Cosmos DB account**

- Azure Cosmos DB container is the fundamental unit of scalability
- Up to 50 Cosmos DB accounts > Databases > Containers > Items
- Depending on Cosmos API, a container is a collection, table or graph
- Depending on Cosmos API, an item is a document, row, node or edge

An Azure Cosmos DB container is where data is stored. Unlike most relational databases, which scale up with larger sizes of virtual machines, Azure Cosmos DB scales out to _partitions_. When you create a container, you need to supply a partition key.

The underlying storage mechanism for data in Azure Cosmos DB is called a `physical partition`. Physical partitions can have a throughput amount up to 10,000 Request Units per second, and they can store up to 50 GB of data. Azure Cosmos DB abstracts this partitioning concept with a logical partition, which can store up to 20 GB of data.

When you create a container, you configure throughput in one of the following modes:

- Dedicated throughput
  - The throughput on a container is exclusively reserved for that container.
  - There are two types of dedicated throughput: standard and autoscale.
- Shared throughput
  - Throughput is specified at the database level and then shared with up to 25 containers within the database.
  - Sharing of throughput excludes containers that are configured with their own dedicated throughput.

**Consistency levels**

Azure Cosmos DB offers five well-defined levels. From strongest to weakest, the levels are:

- Strong
  - Strong consistency offers a linearizability guarantee.
  - Linearizability refers to serving requests concurrently.
  - The reads are guaranteed to return the most recent committed version of an item.
  - A client never sees an uncommitted or partial write.
  - Users are always guaranteed to read the latest committed write.
- Bounded staleness
  - In bounded staleness consistency, the lag of data between any two regions is always less than a specified amount (_K_ versions/updates or _T_ time intervals)
  - Bounded Staleness is beneficial primarily to single-region write accounts with two or more regions.
- Session
  - In session consistency, within a single client session, reads are guaranteed to honor the read-your-writes, and write-follows-reads guarantees.
  - Like all consistency levels weaker than Strong, writes are replicated to a minimum of three replicas in the local region, with asynchronous replication to all other regions.
- Consistent prefix
  - In consistent prefix, updates made as single document writes see eventual consistency.
  - Updates made as a batch within a transaction, are returned consistent to the transaction in which they were committed.
  - Write operations within a transaction of multiple documents are always visible together.
- Eventual
  - In eventual consistency, there's no ordering guarantee for reads. In the absence of any further writes, the replicas eventually converge.
  - Eventual consistency is ideal where the application doesn't require any ordering guarantees (ie: Retweets, likes, or nonthreaded comments)

**Supported APIs**

Azure Cosmos DB offers multiple database APIs, which include NoSQL, MongoDB, PostgreSQL, Cassandra, Gremlin, and Table. API for NoSQL is native to Azure Cosmos DB.

**Request units**

With Azure Cosmos DB, you pay for the throughput you provision and the storage you consume on an hourly basis. Throughput must be provisioned to ensure that sufficient system resources are available for your Azure Cosmos database always.

The cost of all database operations is normalized in Azure Cosmos DB and expressed by _request units_ (or RUs, for short). The cost to do a point read, which is fetching a single item by its ID and partition key value, for a 1-KB item is 1RU.

The type of Azure Cosmos DB account you're using determines the way consumed RUs get charged. There are three modes in which you can create an account:

- Provisioned throughput mode
  - In this mode, you provision the number of RUs for your application on a per-second basis in increments of 100 RUs per second.
  - To scale the provisioned throughput for your application, you can increase or decrease the number of RUs at any time in increments or decrements of 100 RUs at container and database granularity level.
- Serverless mode
  - In this mode, you don't have to provision any throughput when creating resources in your Azure Cosmos DB account. At the end of your billing period, you get billed for the number of request units consumed by your database operations.
- Autoscale mode
  - In this mode, you can automatically and instantly scale the throughput (RU/s) of your database or container based on its usage.
  - This mode is well suited for mission-critical workloads that have variable or unpredictable traffic patterns, and require SLAs on high performance and scale.

**Create Azure Cosmos DB resources**

- Search for _Cosmos DB_ in Azure search bar
- Select _Azure Cosmos DB for NoSQL_ for native API support
- Capacity mode: _Serverless_
- To create containers and items via portal, go to _Data Explorer_ tab on Cosmos DB resource page

**Microsoft .NET SDK v3 for Azure Cosmos DB**

Because Azure Cosmos DB supports multiple API models, version 3 of the .NET SDK (_Microsoft.Azure.Cosmos_ NuGet package) uses the generic terms _container_ and _item_. A _container_ can be a collection, graph, or table. An _item_ can be a document, edge/vertex, or row, and is the content inside a container.

_CosmosClient_

Creates a new `CosmosClient` with a connection string. `CosmosClient` is thread-safe. The recommendation is to maintain a single instance of `CosmosClient` per lifetime of the application that enables efficient connection management and performance.

```C#
CosmosClient client = new CosmosClient(endpoint, key);
```

The `CosmosClient.CreateDatabaseAsync` method throws an exception if a database with the same name already exists.

```C#
// New instance of Database class referencing the server-side database
Database database1 = await client.CreateDatabaseAsync(
    id: "adventureworks-1"
);

// Checks if a database exists, and if it doesn't, creates it.
Database database2 = await client.CreateDatabaseIfNotExistsAsync(
    id: "adventureworks-2"
);

// Reads a database from the Azure Cosmos DB service as an asynchronous operation.
DatabaseResponse readResponse = await database.ReadAsync();

// Delete a Database as an asynchronous operation.
await database.DeleteAsync();
```

_Container examples_

The `Database.CreateContainerIfNotExistsAsync` method checks if a container exists, and if it doesn't, it creates it.

```C#
// Set throughput to the minimum value of 400 RU/s
ContainerResponse simpleContainer = await database.CreateContainerIfNotExistsAsync(
    id: containerId,
    partitionKeyPath: partitionKey,
    throughput: 400);

// Get a container by ID
Container container = database.GetContainer(containerId);
ContainerProperties containerProperties = await container.ReadContainerAsync();

// Delete a container as an asynchronous operation
await database.GetContainer(containerId).DeleteContainerAsync();
```

_Item examples_

```C#
// Use the Container.CreateItemAsync method to create an item.
/// The method requires a JSON serializable object that must contain an id property, and a partitionKey.
ItemResponse<SalesOrder> response = await container.CreateItemAsync(salesOrder, new PartitionKey(salesOrder.AccountNumber));

// Use the Container.ReadItemAsync method to read an item.
// The method requires type to serialize the item to along with an id property, and a partitionKey.
string id = "[id]";
string accountNumber = "[partition-key]";
ItemResponse<SalesOrder> response = await container.ReadItemAsync(id, new PartitionKey(accountNumber));
```

The `Container.GetItemQueryIterator` method creates a query for items under a container in an Azure Cosmos database using a SQL statement with parameterized values. It returns a FeedIterator.

```C#
QueryDefinition query = new QueryDefinition(
    "select * from sales s where s.AccountNumber = @AccountInput ")
    .WithParameter("@AccountInput", "Account1");

FeedIterator<SalesOrder> resultSet = container.GetItemQueryIterator<SalesOrder>(
    query,
    requestOptions: new QueryRequestOptions()
    {
        PartitionKey = new PartitionKey("Account1"),
        MaxItemCount = 1
    });
```

**Cosmos DB resource creation**

Create the Azure Cosmos DB account:

```
az cosmosdb create \
    --name $accountName \
    --resource-group $resourceGroup
```

Retrieve the `documentEndpoint` for the Azure Cosmos DB account:

```
az cosmosdb show \
    --name $accountName \
    --resource-group $resourceGroup \
    --query "documentEndpoint" \
    --output tsv
```

Retrieve the primary key for the account:

```
az cosmosdb keys list \
    --name $accountName \
    --resource-group $resourceGroup \
    --query "primaryMasterKey" \
    --output tsv
```

**Stored procedures**

Azure Cosmos DB provides language-integrated, transactional execution of JavaScript that lets you write stored procedures, triggers, and user-defined functions (UDFs). To call a stored procedure, trigger, or user-defined function, you need to register it.

Stored procedures can create, update, read, query, and delete items inside an Azure Cosmos container. Stored procedures are registered per collection, and can operate on any document or an attachment present in that collection.

Here's a simple stored procedure that returns a "Hello World" response.

```JavaScript
var helloWorldStoredProc = {
    id: "helloWorld",
    serverScript: function () {
        var context = getContext();
        var response = context.getResponse();

        response.setBody("Hello, World");
    }
}
```

The context object provides access to all operations that can be performed in Azure Cosmos DB, and access to the request and response objects.

_Create an item using stored procedure_

```JavaScript
// This stored procedure takes as input documentToCreate, the body of a document to be created in the current collection.
var createDocumentStoredProc = {
    id: "createMyDocument",
    body: function createMyDocument(documentToCreate) {
        var context = getContext();
        var collection = context.getCollection();
        var accepted = collection.createDocument(collection.getSelfLink(),
              documentToCreate,
              function (err, documentCreated) {
                  if (err) throw new Error('Error' + err.message);
                  context.getResponse().setBody(documentCreated.id)
              });
        if (!accepted) return;
    }
}
```

_Arrays as input parameters for stored procedures_

When defining a stored procedure in the Azure portal, input parameters are always sent as a string to the stored procedure. To work around this, you can define a function within your stored procedure to parse the string as an array.

```JavaScript
function sample(arr) {
    if (typeof arr === "string") arr = JSON.parse(arr);

    arr.forEach(function(a) {
        // do something here
        console.log(a);
    });
}
```

**Create triggers and user-defined functions**

Azure Cosmos DB supports pretriggers and post-triggers. Pretriggers are executed before modifying a database item and post-triggers are executed after modifying a database item.

Triggers aren't automatically executed. They must be specified for each database operation where you want them to execute. After you define a trigger, you should register it by using the Azure Cosmos DB SDKs.

The following example shows how a pretrigger is used to validate the properties of an Azure Cosmos item that is being created:

```JavaScript
function validateToDoItemTimestamp() {
    var context = getContext();
    var request = context.getRequest();

    // item to be created in the current operation
    var itemToCreate = request.getBody();

    // validate properties (adds a timestamp property to a newly added item if it doesn't contain one)
    if (!("timestamp" in itemToCreate)) {
        var ts = new Date();
        itemToCreate["timestamp"] = ts.getTime();
    }

    // update the item that will be created
    request.setBody(itemToCreate);
}
```

Pretriggers can't have any input parameters.

The following example shows a post-trigger:

```JavaScript
function updateMetadata() {
  var context = getContext();
  var container = context.getCollection();
  var response = context.getResponse();

  // item that was created
  var createdItem = response.getBody();

  // query for metadata document
  var filterQuery = 'SELECT * FROM root r WHERE r.id = "_metadata"';
  var accept = container.queryDocuments(container.getSelfLink(), filterQuery,
      updateMetadataCallback);
  if(!accept) throw "Unable to update metadata, abort";

  function updateMetadataCallback(err, items, responseOptions) {
    if(err) throw new Error("Error" + err.message);
      if(items.length != 1) throw 'Unable to find metadata document';

      var metadataItem = items[0];

      // update metadata
      metadataItem.createdItems += 1;
      metadataItem.createdNames += " " + createdItem.id;
      var accept = container.replaceDocument(metadataItem._self,
          metadataItem, function(err, itemReplaced) {
                  if(err) throw "Unable to update metadata, abort";
          });
      if(!accept) throw "Unable to update metadata, abort";
      return;
    }
}
```

**Change feed in Azure Cosmos DB**

Change feed in Azure Cosmos DB is a persistent record of changes to a container in the order they occur.

Change feed support in Azure Cosmos DB works by listening to an Azure Cosmos DB container for any changes. It then outputs the sorted list of documents that were changed in the order in which they were modified.

You can work with the Azure Cosmos DB change feed using either a push model or a pull model.

- With a push model, the change feed processor pushes work to a client that has business logic for processing this work.
  - It's recommended to use the push model because you won't need to worry about polling the change feed for future changes, storing state for the last processed change, and other benefits.
  - There are two ways you can read from the change feed with a push model: Azure Functions Azure Cosmos DB triggers, and the change feed processor library.
- With a pull model, the client has to pull the work from the server. In this case, the client has business logic for processing work and also stores state for the last processed work.

_Change feed processor_

There are four main components of implementing the change feed processor:

- The monitored container
  - The monitored container has the data from which the change feed is generated.
  - Any inserts and updates to the monitored container are reflected in the change feed of the container.
- The lease container
  - The lease container acts as a state storage and coordinates processing the change feed across multiple workers.
  - The lease container can be stored in the same account as the monitored container or in a separate account.
- The compute instance
  - A compute instance hosts the change feed processor to listen for changes.
  - Depending on the platform, it might be represented by a VM, a kubernetes pod, an Azure App Service instance, an actual physical machine.
- The delegate
  - The delegate is the code that defines what you, the developer, want to do with each batch of changes that the change feed processor reads.

```C#
/// <summary>
/// Start the Change Feed Processor to listen for changes and process them with the HandleChangesAsync implementation.
/// </summary>
private static async Task<ChangeFeedProcessor> StartChangeFeedProcessorAsync(
    CosmosClient cosmosClient,
    IConfiguration configuration)
{
    string databaseName = configuration["SourceDatabaseName"];
    string sourceContainerName = configuration["SourceContainerName"];
    string leaseContainerName = configuration["LeasesContainerName"];

    Container leaseContainer = cosmosClient.GetContainer(databaseName, leaseContainerName);
    ChangeFeedProcessor changeFeedProcessor = cosmosClient.GetContainer(databaseName, sourceContainerName)
        .GetChangeFeedProcessorBuilder<ToDoItem>(processorName: "changeFeedSample", onChangesDelegate: HandleChangesAsync)
            .WithInstanceName("consoleHost")
            .WithLeaseContainer(leaseContainer)
            .Build();

    Console.WriteLine("Starting Change Feed Processor...");
    await changeFeedProcessor.StartAsync();
    Console.WriteLine("Change Feed Processor started.");
    return changeFeedProcessor;
}
```

Example of a delegate:

```C#
/// <summary>
/// The delegate receives batches of changes as they are generated in the change feed and can process them.
/// </summary>
static async Task HandleChangesAsync(
    ChangeFeedProcessorContext context,
    IReadOnlyCollection<ToDoItem> changes,
    CancellationToken cancellationToken)
{
    Console.WriteLine($"Started handling changes for lease {context.LeaseToken}...");
    Console.WriteLine($"Change Feed request consumed {context.Headers.RequestCharge} RU.");
    // SessionToken if needed to enforce Session consistency on another client instance
    Console.WriteLine($"SessionToken ${context.Headers.Session}");

    // We may want to track any operation's Diagnostics that took longer than some threshold
    if (context.Diagnostics.GetClientElapsedTime() > TimeSpan.FromSeconds(1))
    {
        Console.WriteLine($"Change Feed request took longer than expected. Diagnostics:" + context.Diagnostics.ToString());
    }

    foreach (ToDoItem item in changes)
    {
        Console.WriteLine($"Detected operation for item with id {item.id}, created at {item.creationTime}.");
        // Simulate some asynchronous operation
        await Task.Delay(10);
    }

    Console.WriteLine("Finished handling changes.");
}
```

### Azure Container Registry

Azure Container Registry (ACR) is a managed registry service based on the open-source Docker Registry 2.0.

Use cases:

- **Scalable orchestration systems** that manage containerized applications across clusters of hosts, including Kubernetes, DC/OS, and Docker Swarm.
- **Azure services** that support building and running applications at scale, including Azure Kubernetes Service (AKS), App Service, Batch, and Service Fabric.

Use **Azure Container Registry Tasks (ACR Tasks)** to streamline building, testing, pushing, and deploying images in Azure. Configure build tasks to automate your container OS and framework patching pipeline, and build images automatically when your team commits code to source control.

Azure Container Registry (ACR) tasks are a suite of features that:

- Provide cloud-based container image building for platforms like Linux, Windows, and Advanced RISC Machines (Arm).
- Extend the early parts of an application development cycle to the cloud with on-demand container image builds.
- Enable automated builds triggered by source code updates, updates to a container's base image, or timers.

Creating a container registry:

```
az acr create \
    --resource-group myResourceGroup \
    --name mycontainerregistry \
    --sku Basic
```

Build an image:

```
az acr build \
    --image sample/hello-world:v1 \
    --registry mycontainerregistry \
    --file Dockerfile .
```

List the repositories:

```
az acr repository list \
    --name myContainerRegistry \
    --output table
```

List the tags on the repository:

```
az acr repository show-tags \
    --name myContainerRegistry \
    --repository sample/hello-world \
    --output table
```

Run the image:

```
az acr run \
    --registry myContainerRegistry \
    --cmd '$Registry/sample/hello-world:v1' /dev/null
```

### Azure Container Instances

Azure Container Instances (ACI) is a great solution for any scenario that can operate in isolated containers, including simple applications, task automation, and build jobs.

The top-level resource in Azure Container Instances is the _container group_. A container group is a collection of containers that get scheduled on the same host machine. Containers in a container group share a lifecycle, resources, local network, and storage volumes. It's similar in concept to a _pod_ in Kubernetes.

There are two common ways to deploy a multi-container group: use a Resource Manager template or a YAML file.

**Create and deploy a container**

You create a container by providing a name, a Docker image, and an Azure resource group to the `az container create` command. You expose the container to the Internet by specifying a DNS name label.

```
DNS_NAME_LABEL=aci-example-$RANDOM

az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --os-type Linux \
    --cpu 1 \
    --memory 1.5 \
    --ports 80 \
    --dns-name-label $DNS_NAME_LABEL \
    --location myLocation
```

Verify the container is running:

```
az container show \
    --resource-group myResourceGroup \
    --name mycontainer \
    --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
    --out table
```

**Container restart policy**

When you create a container group in Azure Container Instances, you can specify one of three restart policy settings:

- `Always`
  - Containers in the container group are always restarted.
  - This is the default setting applied when no restart policy is specified at container creation.
- `Never`
  - Containers in the container group are never restarted.
  - The containers run at most once.
- `OnFailure`
  - Containers in the container group are restarted only when the process executed in the container fails (when it terminates with a nonzero exit code).
  - The containers are run at least once.

Specify the `--restart-policy` parameter when you call `az container create`:

```
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mycontainerimage \
    --restart-policy OnFailure
```

**Set environment variables in container instances**

If you need to pass secrets as environment variables, Azure Container Instances supports secure values for both Windows and Linux containers.

```
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --restart-policy OnFailure \
    --environment-variables 'NumWords'='5' 'MinLength'='8'\
```

Set a secure environment variable by specifying the `secureValue` property instead of the regular value for the variable's type. The two variables defined in the following YAML demonstrate the two variable types:

```YAML
apiVersion: 2018-10-01
location: eastus
name: securetest
properties:
  containers:
  - name: mycontainer
    properties:
      environmentVariables:
        - name: 'NOTSECRET'
          value: 'my-exposed-value'
        - name: 'SECRET'
          secureValue: 'my-secret-value'
      image: nginx
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  osType: Linux
  restartPolicy: Always
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

You would run the following command to deploy the container group with YAML:

```Bash
az container create \
    --resource-group myResourceGroup \
    --file secure-env.yaml
```

**Mount an Azure file share in Azure Container Instances**

By default, Azure Container Instances are stateless. If the container crashes or stops, all of its state is lost. To persist state beyond the lifetime of the container, you must mount a volume from an external store.

To mount an Azure file share as a volume in a container by using the Azure CLI, specify the share and volume mount point when you create the container with `az container create`:

```
az container create \
    --resource-group $ACI_PERS_RESOURCE_GROUP \
    --name hellofiles \
    --image mcr.microsoft.com/azuredocs/aci-hellofiles \
    --dns-name-label aci-demo \
    --ports 80 \
    --azure-file-volume-account-name $ACI_PERS_STORAGE_ACCOUNT_NAME \
    --azure-file-volume-account-key $STORAGE_KEY \
    --azure-file-volume-share-name $ACI_PERS_SHARE_NAME \
    --azure-file-volume-mount-path /aci/logs/
```

You can also deploy a container group and mount a volume in a container with the Azure CLI and a YAML template. Deploying by YAML template is the preferred method when deploying container groups consisting of multiple containers.

```YAML
apiVersion: '2019-12-01'
location: eastus
name: file-share-demo
properties:
  containers:
  - name: hellofiles
    properties:
      environmentVariables: []
      image: mcr.microsoft.com/azuredocs/aci-hellofiles
      ports:
      - port: 80
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      - mountPath: /aci/logs/
        name: filesharevolume
  osType: Linux
  restartPolicy: Always
  ipAddress:
    type: Public
    ports:
      - port: 80
    dnsNameLabel: aci-demo
  volumes:
  - name: filesharevolume
    azureFile:
      sharename: acishare
      storageAccountName: <Storage account name>
      storageAccountKey: <Storage account key>
tags: {}
type: Microsoft.ContainerInstance/containerGroups
```

To mount multiple volumes in a container instance, you must deploy using an Azure Resource Manager template or a YAML file. To use a template or YAML file, provide the share details and define the volumes by populating the `volumes` array in the `properties` section of the template.

```JSON
"volumes": [{
  "name": "myvolume1",
  "azureFile": {
    "shareName": "share1",
    "storageAccountName": "myStorageAccount",
    "storageAccountKey": "<storage-account-key>"
  }
},
{
  "name": "myvolume2",
  "azureFile": {
    "shareName": "share2",
    "storageAccountName": "myStorageAccount",
    "storageAccountKey": "<storage-account-key>"
  }
}]
```

Next, for each container in the container group in which you'd like to mount the volumes, populate the volumeMounts array in the properties section of the container definition.

```JSON
"volumeMounts": [{
  "name": "myvolume1",
  "mountPath": "/mnt/share1/"
},
{
  "name": "myvolume2",
  "mountPath": "/mnt/share2/"
}]
```

### Azure Container Apps

Azure Container Apps enables you to run microservices and containerized applications on a serverless platform that runs on top of Azure Kubernetes Service. Common uses of Azure Container Apps include:

- Deploying API endpoints
- Hosting background processing applications
- Handling event-driven processing
- Running microservices

Applications built on Azure Container Apps can dynamically scale based on: HTTP traffic, event-driven processing, CPU or memory load, and any KEDA-supported scaler.

Ensuring you have the latest version of the Azury Container Apps extension for the CLI:

```
az extension add --name containerapp --upgrade
```

There are two namespaces (used to group and manage resource providers) that need to be registered for Azure Container Apps, so ensure they are registered:

```
az provider register --namespace Microsoft.App
az provider register --namespace Microsoft.OperationalInsights
```

An environment in Azure Container Apps creates a secure boundary around a group of container apps. Container Apps deployed to the same environment are deployed in the same virtual network and write logs to the same Log Analytics workspace.

Create an Azure Container Apps environment:

```
az containerapp env create \
    --name my-container-env \
    --resource-group myResourceGroup \
    --location myLocation
```

Deploy an app container image:

```
az containerapp create \
    --name my-container-app \
    --resource-group myResourceGroup \
    --environment my-container-env \
    --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
    --target-port 80 \
    --ingress 'external' \
    --query properties.configuration.ingress.fqdn
```

By setting `--ingress` to `external`, you make the container app available to public requests.

**Containers in Azure Container Apps**

Here is an example of the containers array in the properties.template section of a container app resource template:

```JSON
"containers": [
  {
    "name": "main",
    "image": "[parameters('container_image')]",
    "env": [
      {
        "name": "HTTP_PORT",
        "value": "80"
      },
      {
        "name": "SECRET_VAL",
        "secretRef": "mysecret"
      }
    ],
    "resources": {
      "cpu": 0.5,
      "memory": "1Gi"
    },
    "volumeMounts": [
      {
        "mountPath": "/myfiles",
        "volumeName": "azure-files-volume"
      }
    ]
    "probes": [
        {
          "type": "liveness",
          "httpGet": {
            "path": "/health",
            "port": 8080,
            "httpHeaders": [
              {
                "name": "Custom-Header",
                "value": "liveness probe"
              }]
          },
          "initialDelaySeconds": 7,
          "periodSeconds": 3
// file is truncated for brevity
```

You can deploy images hosted on private registries by providing credentials in the Container Apps configuration.

To use a container registry, you define the required fields in registries array in the properties.configuration section of the container app resource template. The passwordSecretRef field identifies the name of the secret in the secrets array name where you defined the password.

```JSON
{
  ...
  "registries": [{
    "server": "docker.io",
    "username": "my-registry-user-name",
    "passwordSecretRef": "my-password-secret-name"
  }]
}
```

**Authentication and authorization in Azure Container Apps**

Azure Container Apps provides built-in authentication and authorization features to secure your external ingress-enabled container app with minimal or no code.

The authentication and authorization middleware component is a feature of the platform that runs as a sidecar container on each replica in your application. When enabled, every incoming HTTP request passes through the security layer before being handled by your application.

The platform middleware handles several things for your app:

- Authenticates users and clients with the specified identity providers
  - Microsoft Identity Platform (/.auth/login/aad)
  - Facebook (/.auth/login/facebook)
  - GitHub (/.auth/login/github)
  - Google (/.auth/login/google)
  - X (/.auth/login/twitter)
  - Any OpenID Connect provider (/.auth/login/<providerName>)
- Manages the authenticated session
- Injects identity information into HTTP request headers

**Manage revisions and secrets in Azure Container Apps**

Azure Container Apps implements container app versioning by creating revisions. A revision is an immutable snapshot of a container app version. You can use revisions to release a new version of your app, or quickly revert to an earlier version of your app.

With the az containerapp update command you can modify environment variables, compute resources, scale parameters, and deploy a different image. If your container app update includes revision-scope changes, a new revision is generated.

```
az containerapp update \
  --name <APPLICATION_NAME> \
  --resource-group <RESOURCE_GROUP_NAME> \
  --image <IMAGE_NAME>
```

You can list all revisions associated with your container app:

```
az containerapp revision list \
  --name <APPLICATION_NAME> \
  --resource-group <RESOURCE_GROUP_NAME> \
  -o table
```

Container Apps doesn't support Azure Key Vault integration. Instead, enable managed identity in the container app and use the Key Vault SDK in your app to access secrets.

When you create a container app, secrets are defined using the --secrets parameter.

```
az containerapp create \
  --resource-group "my-resource-group" \
  --name queuereader \
  --environment "my-environment-name" \
  --image demos/queuereader:v1 \
  --secrets "queue-connection-string=$CONNECTION_STRING"
```

To reference a secret in an environment variable in the Azure CLI, set its value to `secretref:`, followed by the name of the secret.

```
az containerapp create \
  --resource-group "my-resource-group" \
  --name myQueueApp \
  --environment "my-environment-name" \
  --image demos/myQueueApp:v1 \
  --secrets "queue-connection-string=$CONNECTIONSTRING" \
  --env-vars "QueueName=myqueue" "ConnectionString=secretref:queue-connection-string"
```

### Microsoft identity platform

The Microsoft identity platform helps you build applications your users and customers can sign in to using their Microsoft identities or social accounts, and provide authorized access to your own APIs or Microsoft APIs like Microsoft Graph.

There are several components that make up the Microsoft identity platform:

- OAuth 2.0 and OpenID Connect standard-compliant authentication service
- Open-source libraries
  - Microsoft Authentication Libraries (MSAL)
- Microsoft identity platform endpoint
- Application management portal
- Application configuration API and PowerShell

For developers, the Microsoft identity platform offers integration of modern innovations in the identity and security space like passwordless authentication, step-up authentication, and Conditional Access.

**Service principals**

To delegate Identity and Access Management functions to Microsoft Entra ID, an application must be registered with a Microsoft Entra tenant.

When you register an app in the Azure portal, you choose whether it is:

- Single tenant: only accessible in your tenant
- Multi-tenant: accessible in other tenants

**Application object**

A Microsoft Entra application is scoped to its one and only application object. The application object resides in the Microsoft Entra tenant where the application was registered (known as the application's "home" tenant). An application object is used as a template or blueprint to create one or more service principal objects. A service principal is created in every tenant where the application is used.

The application object describes three aspects of an application:

- How the service can issue tokens in order to access the application.
- Resources that the application might need to access.
- The actions that the application can take.

**Service principal object**

To access resources secured by a Microsoft Entra tenant, the entity that is requesting access must be represented by a security principal. This is true for both users (user principal) and applications (service principal).

The security principal defines the access policy and permissions for the user/application in the Microsoft Entra tenant. This enables core features such as authentication of the user/application during sign-in, and authorization during resource access.

There are three types of service principal:

- Application

  - This type of service principal is the local representation, or application instance, of a global application object in a single tenant or directory.
  - A service principal is created in each tenant where the application is used, and references the globally unique app object.
  - The service principal object defines what the app can actually do in the specific tenant, who can access the app, and what resources the app can access.

- Managed identity

  - This type of service principal is used to represent a managed identity.
  - Managed identities provide an identity for applications to use when connecting to resources that support Microsoft Entra authentication.
  - When a managed identity is enabled, a service principal representing that managed identity is created in your tenant.
  - Service principals representing managed identities can be granted access and permissions, but can't be updated or modified directly.

- Legacy
  - This type of service principal represents a legacy app, which is an app created before app registrations were introduced or an app created through legacy experiences.
  - A legacy service principal can have:
    - credentials
    - service principal names
    - reply URLs
    - and other properties that an authorized user can edit, but doesn't have an associated app registration.

An application object has:

- A one to one relationship with the software application, and
- A one to many relationships with its corresponding service principal objects.

A service principal must be created in each tenant where the application is used to establish an identity for sign-in and/or access to resources being secured by the tenant.

**Permissions and consent**

The Microsoft identity platform implements the OAuth 2.0 authorization protocol. Any web-hosted resource that integrates with the Microsoft identity platform has a resource identifier, or _application ID URI_.

In OAuth 2.0, these types of permission sets are called scopes. They're also often referred to as permissions. In the Microsoft identity platform, a permission is represented as a string value.

_Permission types:_

- **Delegated access** are used by apps that have a signed-in user present. For these apps, either the user or an administrator consents to the permissions that the app requests. The app is delegated with the permission to act as a signed-in user when it makes calls to the target resource.
- **App-only access permissions** are used by apps that run without a signed-in user present, for example, apps that run as background services or daemons. Only an administrator can consent to app-only access permissions.

_Consent types:_

- Static user consent
  - You must specify all the permissions it needs in the app's configuration in the Azure portal
- Incremental and dynamic user consent
  - You can ask for a minimum set of permissions upfront and request more over time as the customer uses more app features.
- Admin consent
  - Admin consent ensures that administrators have some other controls before authorizing apps or users to access highly privileged data from the organization.

In an OpenID Connect or OAuth 2.0 authorization request, an app can request the permissions it needs by using the scope query parameter:

```
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=00001111-aaaa-2222-bbbb-3333cccc4444
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=
https%3A%2F%2Fgraph.microsoft.com%2Fcalendars.read%20
https%3A%2F%2Fgraph.microsoft.com%2Fmail.send
&state=12345
```

**Conditional access**

Conditional Access enables developers and enterprise customers to protect services in a multitude of ways including:

- Multifactor authentication
- Allowing only Intune enrolled devices to access specific services
- Restricting user locations and IP ranges

### Microsoft Authentication Library

The Microsoft Authentication Library (MSAL) enables developers to acquire security tokens from the Microsoft identity platform to authenticate users and access secured web APIs.

The Microsoft Authentication Library (MSAL) defines two types of clients; public clients and confidential clients:

- Public client applications run on devices, such as desktop, browserless APIs, mobile or client-side browser apps.
  - They can't be trusted to safely keep application secrets, so they can only access web APIs on behalf of the user.
- Confidential client applications run on servers, such as web apps, web API apps, or service/daemon apps.
  - They're considered difficult to access by users or attackers, and therefore can adequately hold configuration-time secrets to assert proof of its identity.

**Client applications**

With MSAL.NET 3.x, the recommended way to instantiate an application is by using the application builders: PublicClientApplicationBuilder and ConfidentialClientApplicationBuilder.

Before initializing an application, you first need to register it so that your app can be integrated with the Microsoft identity platform. After registration, you might need the following information (which can be found in the Azure portal):

- **Application (client) ID** - This is a string representing a GUID.
- **Directory (tenant) ID** - Provides identity and access management (IAM) capabilities to applications and resources used by your organization. It can specify if you're writing a line of business application solely for your organization (also named single-tenant application).
- **The identity provider URL** (named the **instance**) and the sign-in audience for your application. These two parameters are collectively known as the authority.
- **Client credentials** - which can take the form of an application secret (client secret string) or certificate (of type `X509Certificate2`) if it's a confidential client app.
- For web apps, and sometimes for public client apps (in particular when your app needs to use a broker), you need to set the **Redirect URI** where the identity provider sends the security token back to your application.

**Initializing public and confidential client applications from code**

The following code instantiates a public client application, signing-in users in the Microsoft Azure public cloud.

```C#
IPublicClientApplication app = PublicClientApplicationBuilder.Create(clientId).Build();
```

In the same way, the following code instantiates a confidential application (a Web app located at https://myapp.azurewebsites.net) handling tokens from users in the Microsoft Azure public cloud.

```C#
string redirectUri = "https://myapp.azurewebsites.net";
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithClientSecret(clientSecret)
    .WithRedirectUri(redirectUri )
    .Build();
```

`.With` methods can be applied as modifiers:

- `.WithAuthority()`
  - Sets the application default authority to a Microsoft Entra authority, with the possibility of choosing the Azure Cloud, the audience, the tenant (tenant ID or domain name), or providing directly the authority URI.
- `.WithTenantId(string tenantId)`
  - Overrides the tenant ID, or the tenant description.
- `.WithClientId(string)`
  - Overrides the client ID.
- `.WithRedirectUri(string redirectUri)`
  - Overrides the default redirect URI. This is useful for scenarios requiring a broker.
- `.WithComponent(string)`
  - Sets the name of the library using MSAL.NET (for telemetry reasons).
- `.WithDebugLoggingCallback()`
  - If called, the application calls Debug.Write simply enabling debugging traces.
- `.WithLogging()`
  - If called, the application calls a callback with debugging traces.
- `.WithTelemetry(TelemetryCallback telemetryCallback)`
  - Sets the delegate used to send telemetry.

**Register a new application**

- In the portal, search for and select _App registrations_.
- Select _+ New registration_
- For _Redirect URI_ set _Public client/native..._ and value of `http://localhost`
- In the _Essentials_ section of the _Overview_ page record the _Application (client) ID_ and the _Directory (tenant) ID_.

### Shared access signatures

A shared access signature (SAS) is a signed URI that points to one or more storage resources and includes a token that contains a special set of query parameters. The token indicates how the resources might be accessed by the client.

Azure Storage supports three types of shared access signatures:

- User delegation SAS
  - A user delegation SAS is secured with Microsoft Entra credentials and also by the permissions specified for the SAS.
  - A user delegation SAS applies to Blob storage only.
  - Microsoft recommends that you use Microsoft Entra credentials when possible as a security best practice
- Service SAS
  - A service SAS is secured with the storage account key.
  - A service SAS delegates access to a resource in the following Azure Storage services: Blob storage, Queue storage, Table storage, or Azure Files.
- Account SAS
  - An account SAS is secured with the storage account key.
  - An account SAS delegates access to resources in one or more of the storage services. All of the operations available via a service or user delegation SAS are also available via an account SAS.

**Anatomy of SAS**

In a single URI, such as `https://medicalrecords.blob.core.windows.net/patient-images/patient-116139-nq8z7f.jpg?sp=r&st=2020-01-20T11:42:32Z&se=2020-01-20T19:42:32Z&spr=https&sv=2019-02-02&sr=b&sig=SrW1HZ5Nb6MbRzTbXCaPm%2BJiSEn15tC91Y4umMPwVZs%3D`, you can separate the URI from the SAS token as follows:

- **URI**: `https://medicalrecords.blob.core.windows.net/patient-images/patient-116139-nq8z7f.jpg?`
- **SAS token**: `sp=r&st=2020-01-20T11:42:32Z&se=2020-01-20T19:42:32Z&spr=https&sv=2019-02-02&sr=b&sig=SrW1HZ5Nb6MbRzTbXCaPm%2BJiSEn15tC91Y4umMPwVZs%3D`

The SAS token itself is made up of several components:

- `sp=r`
  - Controls the access rights. The values can be `a` for add, `c` for create, `d` for delete, `l` for list, `r` for read, or `w` for write.
  - This example is read only. The example `sp=acdlrw` grants all the available rights.
- `st=2020-01-20T11:42:32Z`
  - The date and time when access starts.
- `se=2020-01-20T19:42:32Z`
  - The date and time when access ends. This example grants eight hours of access.
- `sv=2019-02-02`
  - The version of the storage API to use.
- `sr=b`
  - The kind of storage being accessed. In this example, `b` is for blob.
- `sig=SrW1HZ5Nb6MbRzTbXCaPm%2BJiSEn15tC91Y4umMPwVZs%3D`
  - The cryptographic signature.

**Stored access policies**

A stored access policy provides an extra level of control over service-level shared access signatures (SAS) on the server side. Establishing a stored access policy groups SAS and provides more restrictions for signatures that bound by the policy.

The following storage resources support stored access policies:

- Blob containers
- File shares
- Queues
- Tables

The access policy for a SAS consists of the start time, expiry time, and permissions for the signature. The parameters can be specified on the combination of signature URI and on the stored access policy.

Examples of creating a stored access policy by using C# .NET and the Azure CLI:

```C#
BlobSignedIdentifier identifier = new BlobSignedIdentifier
{
    Id = "stored access policy identifier",
    AccessPolicy = new BlobAccessPolicy
    {
        ExpiresOn = DateTimeOffset.UtcNow.AddHours(1),
        Permissions = "rw"
    }
};

blobContainer.SetAccessPolicy(permissions: new BlobSignedIdentifier[] { identifier });
```

```
az storage container policy create \
    --name <stored access policy identifier> \
    --container-name <container name> \
    --start <start time UTC datetime> \
    --expiry <expiry time UTC datetime> \
    --permissions <(a)dd, (c)reate, (d)elete, (l)ist, (r)ead, or (w)rite> \
    --account-key <storage account key> \
    --account-name <storage account name> \
```

### Microsoft Graph

- _The Microsoft Graph API_ offers a single endpoint, https://graph.microsoft.com. You can use REST APIs or SDKs to access the endpoint. Microsoft Graph also includes services that manage user and device identity, access, compliance, and security.
- _Microsoft Graph connectors_ work in the incoming direction, delivering data external to the Microsoft cloud into Microsoft Graph services and applications, to enhance Microsoft 365 experiences such as Microsoft Search. Connectors exist for many commonly used data sources such as Box, Google Drive, Jira, and Salesforce.
- _Microsoft Graph Data Connect_ provides a set of tools to streamline secure and scalable delivery of Microsoft Graph data to popular Azure data stores. The cached data serves as data sources for Azure development tools that you can use to build intelligent applications.

**REST API method**

```
{HTTP method} https://graph.microsoft.com/{version}/{resource}?{query-parameters}
```

A resource can be an entity or complex type, commonly defined with properties. Entities differ from complex types by always including an `id` property.

**SDKs**

The Microsoft Graph SDKs are designed to simplify building high-quality, efficient, and resilient applications that access Microsoft Graph. The SDKs include two components: a service library and a core library.

Create a Microsoft Graph client:

```C#
var scopes = new[] { "User.Read" };

// Multi-tenant apps can use "common",
// single-tenant apps must use the tenant ID from the Azure portal
var tenantId = "common";

// Value from app registration
var clientId = "YOUR_CLIENT_ID";

// using Azure.Identity;
var options = new TokenCredentialOptions
{
    AuthorityHost = AzureAuthorityHosts.AzurePublicCloud
};

// Callback function that receives the user prompt
// Prompt contains the generated device code that you must
// enter during the auth process in the browser
Func<DeviceCodeInfo, CancellationToken, Task> callback = (code, cancellation) => {
    Console.WriteLine(code.Message);
    return Task.FromResult(0);
};

// /dotnet/api/azure.identity.devicecodecredential
var deviceCodeCredential = new DeviceCodeCredential(
    callback, tenantId, clientId, options);

var graphClient = new GraphServiceClient(deviceCodeCredential, scopes);
```

Read information from Microsoft Graph:

```C#
// GET https://graph.microsoft.com/v1.0/me
var user = await graphClient.Me
    .GetAsync();
```

Retrieve a list of entities:

- The `$filter` query parameter can be used to reduce the result set to only those rows that match the provided condition.
- The `$orderBy` query parameter requests that the server provides the list of entities sorted by the specified properties.

```C#
// GET https://graph.microsoft.com/v1.0/me/messages?
// $select=subject,sender&$filter=subject eq 'Hello world'
var messages = await graphClient.Me.Messages
    .GetAsync(requestConfig =>
    {
        requestConfig.QueryParameters.Select =
            ["subject", "sender"];
        requestConfig.QueryParameters.Filter =
            "subject eq 'Hello world'";
    });
```

Delete an entity:

```C#
// DELETE https://graph.microsoft.com/v1.0/me/messages/{message-id}
// messageId is a string containing the id property of the message
await graphClient.Me.Messages[messageId]
    .DeleteAsync();
```

Create a new entity:

```C#
// POST https://graph.microsoft.com/v1.0/me/calendars
var calendar = new Calendar
{
    Name = "Volunteer",
};

var newCalendar = await graphClient.Me.Calendars
    .PostAsync(calendar);
```

### Azure Key Vault

Azure Key Vault is a tool for securely storing and accessing secrets. A secret is anything that you want to tightly control access to, such as API keys, passwords, or certificates. A vault is logical group of secrets.

The Azure Key Vault service supports two types of containers: vaults and managed hardware security module(HSM) pools. Vaults support storing software and HSM-backed keys, secrets, and certificates. Managed HSM pools only support HSM-backed keys.

Azure Key Vault helps solve the following problems:

- **Secrets Management**: Azure Key Vault can be used to Securely store and tightly control access to tokens, passwords, certificates, API keys, and other secrets
- **Key Management**: Azure Key Vault can also be used as a Key Management solution. Azure Key Vault makes it easy to create and control the encryption keys used to encrypt your data.
- **Certificate Management**: Azure Key Vault is also a service that lets you easily provision, manage, and deploy public and private Secure Sockets Layer/Transport Layer Security (SSL/TLS) certificates for use with Azure and your internal connected resources.

**Authentication**

There are three ways to authenticate to Key Vault:

- **Managed identities for Azure resources**
  - When you deploy an app on a virtual machine in Azure, you can assign an identity to your virtual machine that has access to Key Vault. You can also assign identities to other Azure resources.
  - The benefit of this approach is that the app or service isn't managing the rotation of the first secret. Azure automatically rotates the service principal client secret associated with the identity.
  - _We recommend this approach as a best practice._
- **Service principal and certificate**
  - You can use a service principal and an associated certificate that has access to Key Vault.
  - We don't recommend this approach because the application owner or developer must rotate the certificate.
- **Service principal and secret**
  - Although you can use a service principal and a secret to authenticate to Key Vault, we don't recommend it. It's hard to automatically rotate the bootstrap secret that's used to authenticate to Key Vault.

Authentication with Key Vault works with Microsoft Entra ID, which is responsible for authenticating the identity of any given security principal. A security principal is anything that can request access to Azure resources. This includes:

- Users – Real people with accounts in Microsoft Entra ID.
- Groups – Collections of users. Permissions given to the group apply to all its members.
- Service Principals – Represent apps or services (not people). Think of it like a user account for an app.

**Create an Azure Key Vault resource**

```
az keyvault create \
    --name $keyVaultName \
    --resource-group $resourceGroup \
    --location $location
```

To create and retrieve a secret, assign your Microsoft Entra user to the Key Vault Secrets Officer role. This gives your user account permission to set, delete, and list secrets. In a typical scenario you may want to separate the create/read actions by assigning the Key Vault Secrets Officer to one group, and Key Vault Secrets User (can get and list secrets) to another.

**Retrieve `userPrincipalName`**

```
userPrincipal=$(az rest \
    --method GET \
    --url https://graph.microsoft.com/v1.0/me \
    --headers 'Content-Type=application/json' \
    --query userPrincipalName --output tsv)
```

**Retrieve `resourceID`**

```
resourceID=$(az keyvault show \
    --resource-group $resourceGroup \
    --name $keyVaultName \
    --query id \
    --output tsv)
```

**Create and assign the Key Vault Secrets Officer role**

```
az role assignment create \
    --assignee $userPrincipal \
    --role "Key Vault Secrets Officer" \
    --scope $resourceID
```

**Create a secret**

```
az keyvault secret set \
    --vault-name $keyVaultName \
    --name "MySecret" \
    --value "My secret value"
```

**Retrieve a secret**

```
az keyvault secret show \
    --name "MySecret" \
    --vault-name $keyVaultName
```

### Managed Identities

A common challenge for developers is the management of secrets, credentials, certificates, and keys used to secure communication between services. Managed identities eliminate the need for developers to manage these credentials. They provide an automatically managed identity in Microsoft Entra ID for applications to use when connecting to resources that support Microsoft Entra authentication.

There are two types of managed identities:

- A **system-assigned managed identity** is enabled directly on an Azure service instance. When the identity is enabled, Azure creates an identity for the instance in the Microsoft Entra tenant trusted by the subscription of the instance. After the identity is created, the credentials are provisioned onto the instance.
- A user-assigned managed identity is created as a standalone Azure resource. Through a create process, Azure creates an identity in the Microsoft Entra tenant that's trusted by the subscription in use. After the identity is created, the identity can be assigned to one or more Azure service instances.

Following are common use cases for managed identities:

- System-assigned managed identity
  - Workloads contained within a single Azure resource.
  - Workloads needing independent identities.
  - For example, an application that runs on a single virtual machine.
    -User-assigned managed identity
  - Workloads that run on multiple resources and can share a single identity.
  - Workloads needing preauthorization to a secure resource, as part of a provisioning flow.
  - Workloads where resources are recycled frequently, but permissions should stay consistent.
  - For example, a workload where multiple virtual machines need to access the same resource.

**System-assigned managed identity**

To create, or enable, an Azure virtual machine with the system-assigned managed identity your account needs the **Virtual Machine Contributor** role assignment. No other Microsoft Entra directory role assignments are required.

```
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image win2016datacenter \
    --generate-ssh-keys \
    --assign-identity \
    --role contributor \
    --scope mySubscription \
    --admin-username azureuser \
    --admin-password myPassword12
```

Use the `az vm identity assign` command to assign the system-assigned identity to an existing virtual machine:

```
az vm identity assign -g myResourceGroup -n myVm
```

**User-assigned managed identity**

To assign a user-assigned identity to a virtual machine during its creation, your account needs the **Virtual Machine Contributor** and **Managed Identity Operator** role assignments. No other Microsoft Entra directory role assignments are required.

Create a user-assigned identity

```
az identity create -g myResourceGroup -n myUserAssignedIdentity
```

Assign a user-assigned managed identity during the creation of an Azure resource

```
az vm create \
    --resource-group <RESOURCE GROUP> \
    --name <VM NAME> \
    --image Ubuntu2204 \
    --admin-username <USER NAME> \
    --admin-password <PASSWORD> \
    --assign-identity <USER ASSIGNED IDENTITY NAME> \
    --role <ROLE> \
    --scope <SUBSCRIPTION>
```

Assign the user-assigned identity to your virtual machine using `az vm identity` assign.

```
az vm identity assign \
    -g <RESOURCE GROUP> \
    -n <VM NAME> \
    --identities <USER ASSIGNED IDENTITY>
```

### Azure App Configuration

Azure App Configuration provides a service to centrally manage application settings and feature flags.

Azure App Configuration stores configuration data as key-value pairs. It's a common practice to organize keys into a hierarchical namespace by using a character delimiter, such as `/` or `:`. Keys stored in App Configuration are case-sensitive, unicode-based strings.

Key-values in App Configuration can optionally have a label attribute. By default, use `\0`.

```
Key = AppName:DbEndpoint & Label = Test
Key = AppName:DbEndpoint & Label = Staging
Key = AppName:DbEndpoint & Label = Production
```

App Configuration doesn't version key values automatically as they're modified. Use labels as a way to create multiple versions of a key value.

**App configuration feature flags**

Azure App Configuration is designed to be a centralized repository for feature flags.

Each feature flag has two parts: a name and a list of one or more filters that are used to evaluate if a feature's state is on (that is, when its value is `True`).

A filter defines a use case for when a feature should be turned on. When a feature flag has multiple filters, the filter list is traversed in order until one of the filters determines the feature should be enabled.

The feature manager supports `appsettings.json` as a configuration source for feature flags. The following example shows how to set up feature flags in a JSON file:

```JSON
"FeatureManagement": {
    "FeatureA": true, // Feature flag set to on
    "FeatureB": false, // Feature flag set to off
    "FeatureC": {
        "EnabledFor": [
            {
                "Name": "Percentage",
                "Parameters": {
                    "Value": 50
                }
            }
        ]
    }
}
```

**Enable customer-managed key capability**

The following components are required to successfully enable the customer-managed key capability for Azure App Configuration:

- Standard tier Azure App Configuration instance
- Azure Key Vault with soft-delete and purge-protection features enabled
- An RSA or RSA-HSM key within the Key Vault: The key must not be expired, it must be enabled, and it must have both wrap and unwrap capabilities enabled

Once these resources are configured, two steps remain to allow Azure App Configuration to use the Key Vault key:

1. Assign a managed identity to the Azure App Configuration instance
2. Grant the identity GET, WRAP, and UNWRAP permissions in the target Key Vault's access policy.

You can use private endpoints for Azure App Configuration to allow clients on a virtual network to securely access data over a private link.

**Managed identities**

A managed identity from Microsoft Entra ID allows Azure App Configuration to easily access other Microsoft Entra ID-protected resources, such as Azure Key Vault.

Add a system-assigned identity:

```
az appconfig identity assign \
    --name myTestAppConfigStore \
    --resource-group myResourceGroup
```

Add a user-assigned identity:

```
az identity create \
    --resource-group myResourceGroup \
    --name myUserAssignedIdentity

az appconfig identity assign \
    --name myTestAppConfigStore \
    --resource-group myResourceGroup \
    --identities /subscriptions/[subscription id]/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity
```

Create an Azure App Configuration resource:

```
az appconfig create \
    --location $location \
    --name $appConfigName \
    --resource-group $resourceGroup \
    --disable-local-auth true
```

To retrieve configuration information, you need to assign your Microsoft Entra user to the App Configuration Data Reader role.

```
userPrincipal=$(az rest \
    --method GET \
    --url https://graph.microsoft.com/v1.0/me \
    --headers 'Content-Type=application/json' \
    --query userPrincipalName --output tsv)

resourceID=$(az appconfig show \
    --resource-group $resourceGroup \
    --name $appConfigName \
    --query id \
    --output tsv)

az role assignment create \
    --assignee $userPrincipal \
    --role "App Configuration Data Reader" \
    --scope $resourceID
```

Store a string:

```
az appconfig kv set \
    --name $appConfigName \
    --key Dev:conStr \
    --value connectionString
```

### API Management

Azure API Management is made up of an _API gateway_, a _management plane_, and a _developer portal_.

**Products**

Products are how APIs are surfaced to developers. Products in API Management have one or more APIs, and are configured with a title, description, and terms of use. Products can be **Open** or **Protected**.

**API gateways**

An API gateway sits between clients and services. It acts as a reverse proxy, routing requests from clients to services. It might also perform various cross-cutting tasks such as authentication, SSL termination, and rate limiting.

API gateways can be **Managed** or **Self-hosted**.

**API Management policies**

In Azure API Management, policies allow the publisher to change the behavior of the API through configuration. A policy can apply changes to both the inbound request and outbound response.

The policy definition is a simple XML document that describes a sequence of inbound and outbound statements. A policy can be configured at the global level and a particular API level.

The configuration is divided into `inbound`, `backend`, `outbound`, and `on-error`.

```XML
<policies>
  <inbound>
    <!-- statements to be applied to the request go here -->
  </inbound>
  <backend>
    <!-- statements to be applied before the request is forwarded to
         the backend service go here -->
  </backend>
  <outbound>
    <!-- statements to be applied to the response go here -->
  </outbound>
  <on-error>
    <!-- statements to be applied if there is an error condition go here -->
    <!-- can review the error by using the context.LastError property -->
  </on-error>
</policies>
```

A policy expression is either:

- a single C# statement enclosed in `@(expression)`, or
- a multi-statement C# code block, enclosed in `@{expression}`, that returns a value

The following example uses policy expressions and the set-header policy to add user data to the incoming request.

```XML
<policies>
    <inbound>
        <base />
        <set-header name="x-request-context-data" exists-action="override">
            <value>@(context.User.Id)</value>
            <value>@(context.Deployment.Region)</value>
      </set-header>
    </inbound>
</policies>
```

**Advanced policies**

- Control flow
  - Conditionally applies policy statements based on the results of the evaluation of Boolean expressions.

```XML
<choose>
    <when condition="Boolean expression | Boolean constant">
        <!— one or more policy statements to be applied if the above condition is true  -->
    </when>
    <when condition="Boolean expression | Boolean constant">
        <!— one or more policy statements to be applied if the above condition is true  -->
    </when>
    <otherwise>
        <!— one or more policy statements to be applied if none of the above conditions are true  -->
    </otherwise>
</choose>
```

- Forward request
  - Forwards the request to the backend service.
  - Removing this policy results in the request not being forwarded to the backend service.

```XML
<forward-request timeout="time in seconds" follow-redirects="true | false"/>
```

- Limit concurrency
  - Prevents enclosed policies from executing by more than the specified number of requests at a time.

```XML
<limit-concurrency key="expression" max-count="number">
    <!— nested policy statements -->
</limit-concurrency>
```

- Log to Event Hubs
  - Sends messages in the specified format to an event hub defined by a Logger entity.

```XML
<log-to-eventhub logger-id="id of the logger entity" partition-id="index of the partition where messages are sent" partition-key="value used for partition assignment">
  Expression returning a string to be logged
</log-to-eventhub>
```

- Mock response
  - Aborts pipeline execution and returns a mocked response directly to the caller.

```XML
<mock-response status-code="code" content-type="media type"/>
```

- Retry
  - Retries execution of the enclosed policy statements, if and until the condition is met. Execution repeats at the specified time intervals and up to the specified retry count.

```XML
<retry
    condition="boolean expression or literal"
    count="number of retry attempts"
    interval="retry interval in seconds"
    max-interval="maximum retry interval in seconds"
    delta="retry interval delta in seconds"
    first-fast-retry="boolean expression or literal">
        <!-- One or more child policies. No restrictions -->
</retry>
```

- Return response
  - The `return-response` policy aborts pipeline execution and returns either a default or custom response to the caller.

```XML
<return-response response-variable-name="existing context variable">
  <set-header/>
  <set-body/>
  <set-status/>
</return-response>
```

**API subscription keys**

Keys can be passed in the request header, or as a query string in the URL.

The default header name is `Ocp-Apim-Subscription-Key`, and the default query string is `subscription-key`.

**Using certificates**

Certificates can be used to provide Transport Layer Security (TLS) mutual authentication between the client and the API gateway.

With TLS client authentication, the API Management gateway can inspect the certificate contained within the client request and check for properties like:

- Certificate Authority (CA)
  - Only allow certificates signed by a particular CA
- Thumbprint
  - Allow certificates containing a specified thumbprint
- Subject
  - Only allow certificates with a specified subject
- Expiration Date
  - Don't allow expired certificates

Check the thumbprint of a client certificate

```XML
<choose>
    <when condition="@(context.Request.Certificate == null || context.Request.Certificate.Thumbprint != "desired-thumbprint")" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

Check the thumbprint against certificates uploaded to API Management

```XML
<choose>
    <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify()  || !context.Deployment.Certificates.Any(c => c.Value.Thumbprint == context.Request.Certificate.Thumbprint))" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

Check the issuer and subject of a client certificate

```XML
<choose>
    <when condition="@(context.Request.Certificate == null || context.Request.Certificate.Issuer != "trusted-issuer" || context.Request.Certificate.SubjectName.Name != "expected-subject-name")" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

**Create an APIM instance**

```
az apim create \
    -n $myApiName \
    --location $myLocation \
    --publisher-email $myEmail  \
    --resource-group myResourceGroup \
    --publisher-name Import-API-Exercise \
    --sku-name Consumption
```

### Azure Event Grid

Azure Event Grid is a highly scalable, fully managed Pub Sub message distribution service that offers flexible message consumption patterns using the Hypertext Transfer Protocol (HTTP) and Message Queuing Telemetry Transport (MQTT) protocols.

Event Grid can be configured to send events to subscribers (push delivery) or subscribers can connect to Event Grid to read events (pull delivery).

Event Grid conforms to Cloud Native Computing Foundation’s open standard CloudEvents 1.0 specification using the HTTP protocol binding with JSON format. It means that your solutions publish and consume event messages using a format like the following example:

```JSON
{
    "specversion" : "1.0",
    "type" : "com.yourcompany.order.created",
    "source" : "https://yourcompany.com/orders/",
    "subject" : "O-28964",
    "id" : "A234-1234-1234",
    "time" : "2018-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "comexampleothervalue" : 5,
    "datacontenttype" : "application/json",
    "data" : {
       "orderId" : "O-28964",
       "URL" : "https://com.yourcompany/orders/O-28964"
    }
}
```

**Topics**

To respond to certain types of events, subscribers (an Azure service or other applications) decide which topics to subscribe to. There are several kinds of topics: custom topics, system topics, and partner topics.

- **System topics** are built-in topics provided by Azure services.
- **Custom topics** are application and third-party topics. When you create or are assigned access to a custom topic, you see that custom topic in your subscription.
- **Partner topics** are a kind of topic used to subscribe to events published by a partner. The feature that enables this type of integration is called Partner Events.

**Event schemas**

Azure Event Grid supports two types of event schemas: Event Grid event schema and Cloud event schema. Events consist of a set of four required string properties.

When posting events to an Event Grid topic, the array can have a total size of up to 1 MB. Each event in the array is limited to 1 MB. If an event or the array is greater than the size limits, you receive the response `413 Payload Too Large`. Operations are charged in 64 KB increments.

The following example shows the properties that are used by all event publishers:

```JSON
[
  {
    "topic": string,
    "subject": string,
    "id": string,
    "eventType": string,
    "eventTime": string,
    "data":{
      object-unique-to-each-publisher
    },
    "dataVersion": string,
    "metadataVersion": string
  }
]
```

In addition to its default event schema, Azure Event Grid natively supports events in the JSON implementation of CloudEvents v1.0 and HTTP protocol binding. CloudEvents is an open specification for describing event data.

Here's an example of an Azure Blob Storage event in CloudEvents format:

```JSON
{
    "specversion": "1.0",
    "type": "Microsoft.Storage.BlobCreated",
    "source": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-account}",
    "id": "9aeb0fdf-c01e-0131-0922-9eb54906e209",
    "time": "2019-11-18T15:13:39.4589254Z",
    "subject": "blobServices/default/containers/{storage-container}/blobs/{new-file}",
    "dataschema": "#",
    "data": {
        "api": "PutBlockList",
        "clientRequestId": "4c5dd7fb-2c48-4a27-bb30-5361b5de920a",
        "requestId": "9aeb0fdf-c01e-0131-0922-9eb549000000",
        "eTag": "0x8D76C39E4407333",
        "contentType": "image/png",
        "contentLength": 30699,
        "blobType": "BlockBlob",
        "url": "https://gridtesting.blob.core.windows.net/testcontainer/{new-file}",
        "sequencer": "000000000000000000000000000099240000000000c41c18",
        "storageDiagnostics": {
            "batchId": "681fe319-3006-00a8-0022-9e7cde000000"
        }
    }
}
```

The headers values for events delivered in the CloudEvents schema and the Event Grid schema are the same except for content-type. For CloudEvents schema, that header value is `"content-type":"application/cloudevents+json; charset=utf-8"`. For Event Grid schema, that header value is `"content-type":"application/json; charset=utf-8"`.

**Event delivery durability**

Event Grid provides durable delivery. It tries to deliver each event at least once for each matching subscription immediately. If a subscriber's endpoint doesn't acknowledge receipt of an event or if there's a failure, Event Grid retries delivery based on a fixed retry schedule and retry policy. By default, Event Grid delivers one event at a time to the subscriber, and the payload is an array with a single event. Event Grid doesn't guarantee order for event delivery, so subscribers might receive them out of order.

**Retry policy**

You can customize the retry policy when creating an event subscription by using the following two configurations. An event is dropped if either of the limits of the retry policy is reached.

- **Maximum number of attempts**
  - The value must be an integer between 1 and 30.
  - The default value is 30.
- **Event time-to-live (TTL)**
  - The value must be an integer between 1 and 1440.
  - The default value is 1440 minutes

```
az eventgrid event-subscription create \
  -g gridResourceGroup \
  --topic-name <topic_name> \
  --name <event_subscription_name> \
  --endpoint <endpoint_URL> \
  --max-delivery-attempts 18
```

**Output batching**

You can configure Event Grid to batch events for delivery for improved HTTP performance in high-throughput scenarios.

Batched delivery has two settings:

- Max events per batch
- Preferred batch size in kilobytes

**Dead-letter events**

When Event Grid can't deliver an event within a certain time period or after trying to deliver the event a specific number of times, it can send the undelivered event to a storage account.

Event Grid dead-letters an event when one of the following conditions is met.

- Event isn't delivered within the **time-to-live** period.
- The **number of tries** to deliver the event exceeds the limit.

**Filter events**

When creating an event subscription, you have three options for filtering:

- Event types
- Subject begins with or ends with
- Advanced fields and operators

The JSON syntax for filtering by event type is:

```JSON
"filter": {
  "includedEventTypes": [
    "Microsoft.Resources.ResourceWriteFailure",
    "Microsoft.Resources.ResourceWriteSuccess"
  ]
}
```

The JSON syntax for filtering by subject is:

```JSON
"filter": {
  "subjectBeginsWith": "/blobServices/default/containers/mycontainer/log",
  "subjectEndsWith": ".jpg"
}
```

In advanced filtering, you specify the:

- operator type
  - The type of comparison.
- key
  - The field in the event data that you're using for filtering. It can be a number, boolean, or string.
- value or values
  - The value or values to compare to the key.

The JSON syntax for using advanced filters is:

```JSON
"filter": {
  "advancedFilters": [
    {
      "operatorType": "NumberGreaterThanOrEquals",
      "key": "Data.Key1",
      "value": 5
    },
    {
      "operatorType": "StringContains",
      "key": "Subject",
      "values": ["container1", "container2"]
    }
  ]
}
```

Register the Event Grid resource provider:

```
az provider register --namespace Microsoft.EventGrid
```

You can check the status of registration with the following command:

```
az provider show --namespace Microsoft.EventGrid --query "registrationState"
```

Create a topic:

```
az eventgrid topic create \
    --name $topicName \
    --location $location \
    --resource-group $resourceGroup
```

Subscribe to a topic:

```
endpoint="${siteURL}/api/updates"
topicId=$(az eventgrid topic show --resource-group $resourceGroup \
    --name $topicName --query "id" --output tsv)

az eventgrid event-subscription create \
    --source-resource-id $topicId \
    --name TopicSubscription \
    --endpoint $endpoint
```

Retrieve the URL (TOPIC_ENDPOINT) and access key (TOPIC_ACCESS_KEY) for the topic you created:

```
az eventgrid topic show --name $topicName -g $resourceGroup --query "endpoint" --output tsv
az eventgrid topic key list --name $topicName -g $resourceGroup --query "key1" --output tsv
```

### Azure Event Hubs

Azure Event Hubs is a native data-streaming service in the cloud that can stream millions of events per second, with low latency, from any source to any destination. Event Hubs is compatible with Apache Kafka (without any code changes).

Event Hubs is a multi-protocol event streaming engine that natively supports Advanced Message Queuing Protocol (AMQP), Apache Kafka, and HTTPS protocols.

**Event Hubs Capture**

Azure Event Hubs enables you to automatically capture the streaming data in Event Hubs in an Azure Blob storage or Azure Data Lake Storage account.

Event Hubs is a time-retention durable buffer for telemetry ingress, similar to a distributed log. The key to scaling in Event Hubs is the partitioned consumer model. Each partition is an independent segment of data and is consumed independently.

Captured data is written in _Apache Avro_ format.

The storage naming convention is as follows:

```
{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}
```

**Event processor**

The Azure Event Hubs SDKs provide distributed environment functionality (Scale, Load balance, Seamless resume on failures, Consume events). In .NET or Java SDKs, you use an event processor client (`EventProcessorClient`), and in Python and JavaScript SDKs, you use `EventHubConsumerClient`.

**Control access**

Azure Event Hubs supports both Microsoft Entra ID and shared access signatures (SAS) to handle both authentication and authorization. Azure provides the following Azure built-in roles for authorizing access to Event Hubs data using Microsoft Entra ID and OAuth:

- **Azure Event Hubs Data Owner**: Use this role to give complete access to Event Hubs resources.
- **Azure Event Hubs Data Sender**: Use this role to give send access to Event Hubs resources.
- **Azure Event Hubs Data Receiver**: Use this role to give receiving access to Event Hubs resources.

**Common operations with the Event Hubs client library**

Inspect event Hubs

```C#
var connectionString = "<< CONNECTION STRING FOR THE EVENT HUBS NAMESPACE >>";
var eventHubName = "<< NAME OF THE EVENT HUB >>";

await using (var producer = new EventHubProducerClient(connectionString, eventHubName))
{
    string[] partitionIds = await producer.GetPartitionIdsAsync();
}
```

Publish events to Event Hubs

```C#
var connectionString = "<< CONNECTION STRING FOR THE EVENT HUBS NAMESPACE >>";
var eventHubName = "<< NAME OF THE EVENT HUB >>";

await using (var producer = new EventHubProducerClient(connectionString, eventHubName))
{
    using EventDataBatch eventBatch = await producer.CreateBatchAsync();
    eventBatch.TryAdd(new EventData(new BinaryData("First")));
    eventBatch.TryAdd(new EventData(new BinaryData("Second")));

    await producer.SendAsync(eventBatch);
}
```

Read events from an Event Hubs

```C#
var connectionString = "<< CONNECTION STRING FOR THE EVENT HUBS NAMESPACE >>";
var eventHubName = "<< NAME OF THE EVENT HUB >>";

string consumerGroup = EventHubConsumerClient.DefaultConsumerGroupName;

await using (var consumer = new EventHubConsumerClient(consumerGroup, connectionString, eventHubName))
{
    using var cancellationSource = new CancellationTokenSource();
    cancellationSource.CancelAfter(TimeSpan.FromSeconds(45));

    await foreach (PartitionEvent receivedEvent in consumer.ReadEventsAsync(cancellationSource.Token))
    {
        // At this point, the loop will wait for events to be available in the Event Hub. When an event
        // is available, the loop will iterate with the event that was received. Because we did not
        // specify a maximum wait time, the loop will wait forever unless cancellation is requested using
        // the cancellation token.
    }
}
```

Read events from an Event Hubs partition

```C#
var connectionString = "<< CONNECTION STRING FOR THE EVENT HUBS NAMESPACE >>";
var eventHubName = "<< NAME OF THE EVENT HUB >>";

string consumerGroup = EventHubConsumerClient.DefaultConsumerGroupName;

await using (var consumer = new EventHubConsumerClient(consumerGroup, connectionString, eventHubName))
{
    EventPosition startingPosition = EventPosition.Earliest;
    string partitionId = (await consumer.GetPartitionIdsAsync()).First();

    using var cancellationSource = new CancellationTokenSource();
    cancellationSource.CancelAfter(TimeSpan.FromSeconds(45));

    await foreach (PartitionEvent receivedEvent in consumer.ReadEventsFromPartitionAsync(partitionId, startingPosition, cancellationSource.Token))
    {
        // At this point, the loop will wait for events to be available in the partition. When an event
        // is available, the loop will iterate with the event that was received. Because we did not
        // specify a maximum wait time, the loop will wait forever unless cancellation is requested using
        // the cancellation token.
    }
}
```

Process events using an Event Processor client

For most production scenarios, the recommendation is to use `EventProcessorClient` for reading and processing events. Since the `EventProcessorClient` has a dependency on Azure Storage blobs for persistence of its state, you need to provide a `BlobContainerClient` for the processor, which has been configured for the storage account and container that should be used.

```C#
var cancellationSource = new CancellationTokenSource();
cancellationSource.CancelAfter(TimeSpan.FromSeconds(45));

var storageConnectionString = "<< CONNECTION STRING FOR THE STORAGE ACCOUNT >>";
var blobContainerName = "<< NAME OF THE BLOB CONTAINER >>";

var eventHubsConnectionString = "<< CONNECTION STRING FOR THE EVENT HUBS NAMESPACE >>";
var eventHubName = "<< NAME OF THE EVENT HUB >>";
var consumerGroup = "<< NAME OF THE EVENT HUB CONSUMER GROUP >>";

Task processEventHandler(ProcessEventArgs eventArgs) => Task.CompletedTask;
Task processErrorHandler(ProcessErrorEventArgs eventArgs) => Task.CompletedTask;

var storageClient = new BlobContainerClient(storageConnectionString, blobContainerName);
var processor = new EventProcessorClient(storageClient, consumerGroup, eventHubsConnectionString, eventHubName);

processor.ProcessEventAsync += processEventHandler;
processor.ProcessErrorAsync += processErrorHandler;

await processor.StartProcessingAsync();

try
{
    // The processor performs its work in the background; block until cancellation
    // to allow processing to take place.

    await Task.Delay(Timeout.Infinite, cancellationSource.Token);
}
catch (TaskCanceledException)
{
    // This is expected when the delay is canceled.
}

try
{
    await processor.StopProcessingAsync();
}
finally
{
    // To prevent leaks, the handlers should be removed when processing is complete.

    processor.ProcessEventAsync -= processEventHandler;
    processor.ProcessErrorAsync -= processErrorHandler;
}
```

Create an Event Hubs namespace:

> An Azure Event Hubs namespace is a logical container for event hub resources within Azure. It provides a unique scoping container where you can create one or more event hubs, which are used to ingest, process, and store large volumes of event data.

```
az eventhubs namespace create --name $namespaceName --resource-group $resourceGroup -l $location
```

Create an event hub:

```
az eventhubs eventhub create --name myEventHub --resource-group $resourceGroup \
  --namespace-name $namespaceName
```

Assign your Microsoft Entra user to the **Azure Service Bus Data Owner** role at the Service Bus namespace level.

```
# Retrieve user principal
userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
    --headers 'Content-Type=application/json' \
    --query userPrincipalName --output tsv)

# Retrieve the resource ID of the Service Bus namespace
resourceID=$(az eventhubs namespace show --resource-group $resourceGroup \
    --name $namespaceName --query id --output tsv)

# Create and assign the Azure Event Hubs Data Owner role
az role assignment create --assignee $userPrincipal \
    --role "Azure Event Hubs Data Owner" \
    --scope $resourceID
```

### Message Based Solutions

**Service Bus queues**

- Your solution needs to receive messages without having to poll the queue. With Service Bus, you can achieve it by using a long-polling receive operation using the TCP-based protocols that Service Bus supports.
- Your solution requires the queue to provide a guaranteed first-in-first-out (FIFO) ordered delivery.
- Your solution needs to support automatic duplicate detection.
- You want your application to process messages as parallel long-running streams (messages are associated with a stream using the session ID property on the message). In this model, each node in the consuming application competes for streams, as opposed to messages. When a stream is given to a consuming node, the node can examine the state of the application stream state using transactions.
- Your solution requires transactional behavior and atomicity when sending or receiving multiple messages from a queue.
- Your application handles messages that can exceed 64 KB but won't likely approach the 256 KB or 1-MB limit, depending on the chosen service tier (although Service Bus queues can handle messages up to 100 MB).
- You deal with a requirement to provide a role-based access model to the queues, and different rights/permissions for senders and receivers.

\*\* Storage queues

- Your application must store over 80 gigabytes of messages in a queue.
- Your application wants to track progress for processing a message in the queue. It's useful if the worker processing a message crashes. Another worker can then use that information to continue from where the prior worker left off.
- You require server side logs of all of the transactions executed against your queues.

### Azure Service Bus

Azure Service Bus is a fully managed enterprise message broker with message queues and publish-subscribe topics.

The primary wire protocol for Service Bus is Advanced Messaging Queueing Protocol (AMQP) 1.0, an open ISO/IEC standard.

Some common messaging scenarios are:

- _Messaging_. Transfer business data, such as sales or purchase orders, journals, or inventory movements.
- _Decouple applications_. Improve reliability and scalability of applications and services. Client and service don't have to be online at the same time.
- _Topics and subscriptions_. Enable 1:n relationships between publishers and subscribers.
- _Message sessions_. Implement workflows that require message ordering or message deferral.

The messaging entities that form the core of the messaging capabilities in Service Bus are **queues**, **topics and subscriptions**, and **rules/actions**.

**Queues**

You can specify two different modes in which Service Bus receives messages:

- **Receive and delete**
  - In this mode, when Service Bus receives the request from the consumer, it marks the message as consumed and returns it to the consumer application.
  - This mode is the simplest model. It works best for scenarios in which the application can tolerate not processing a message if a failure occurs.
- **Peek lock**.
  - In this mode, the receive operation becomes two-stage, which makes it possible to support applications that can't tolerate missing messages.
  - Finds the next message to be consumed, locks it to prevent other consumers from receiving it, and then, return the message to the application.
  - After the application finishes processing the message, it requests the Service Bus service to complete the second stage of the receive process. Then, the service marks the message as consumed.

**Topics and subscriptions**

A queue allows processing of a message by a single consumer. In contrast to queues, topics and subscriptions provide a one-to-many form of communication in a **publish and subscribe** pattern.

**Message payloads and serialization**

Messages carry a payload and metadata. The metadata is in the form of key-value pair properties, and describes the payload, and gives handling instructions to Service Bus and applications.

A Service Bus message consists of a binary payload section that Service Bus never handles in any form on the service-side, and two sets of properties.

- The broker properties are system defined.
  - A subset of the broker properties, specifically `To`, `ReplyTo`, `ReplyToSessionId`, `MessageId`, `CorrelationId`, and `SessionId`, help applications route messages to particular destinations.
- The user properties are a collection of key-value pairs defined and set by the application.

Create an Azure Service Bus namespace and queue

```
az servicebus namespace create \
    --resource-group $resourceGroup \
    --name $namespaceName \
    --location $location

az servicebus queue create \
    --resource-group $resourceGroup \
    --namespace-name $namespaceName \
    --name myqueue
```

To allow your app to send and receive messages, assign your Microsoft Entra user to the **Azure Service Bus Data Owner** role at the Service Bus namespace level.

```
az role assignment create \
    --assignee $userPrincipal \
    --role "Azure Service Bus Data Owner" \
    --scope $resourceID
```

### Azure Queue Storage

Azure Queue Storage is a service for storing large numbers of messages. You access messages from anywhere in the world via authenticated calls using HTTP or HTTPS. A queue message can be up to 64 KB in size.

Create the Queue service client

```C#
// Get the connection string from app settings
string connectionString = ConfigurationManager.AppSettings["StorageConnectionString"];

// Instantiate a QueueClient which will be used to create and manipulate the queue
QueueClient queueClient = new QueueClient(connectionString, queueName);
```

Create a queue

```C#
// Create the queue
queueClient.CreateIfNotExists();
```

Insert a message into a queue

A message can be either a string (in UTF-8 format) or a byte array.

```C#
if (queueClient.Exists())
{
    // Send a message to the queue
    queueClient.SendMessage(message);
}
```

Peek at the next message

If you don't pass a value for the `maxMessages` parameter, the default is to peek at one message.

```C#
if (queueClient.Exists())
{
    // Peek at the next message
    PeekedMessage[] peekedMessage = queueClient.PeekMessages();
}
```

Change the contents of a queued message

```C#
if (queueClient.Exists())
{
    // Get the message from the queue
    QueueMessage[] message = queueClient.ReceiveMessages();

    // Update the message contents
    queueClient.UpdateMessage(
        message[0].MessageId,
        message[0].PopReceipt,
        "Updated contents",
        TimeSpan.FromSeconds(60.0)  // Make it invisible for another 60 seconds
    );
}
```

Dequeue the next message

When you call `ReceiveMessages`, you get the next message in a queue. A message returned from `ReceiveMessages` becomes invisible to any other code reading messages from this queue. By default, this message stays invisible for 30 seconds. To finish removing the message from the queue, you must also call `DeleteMessage`.

```C#
if (queueClient.Exists())
{
    // Get the next message
    QueueMessage[] retrievedMessage = queueClient.ReceiveMessages();

    // Process (i.e. print) the message in less than 30 seconds
    Console.WriteLine($"Dequeued message: '{retrievedMessage[0].Body}'");

    // Delete the message
    queueClient.DeleteMessage(retrievedMessage[0].MessageId, retrievedMessage[0].PopReceipt);
}
```

Get the queue length

The `ApproximateMessagesCount` property contains the approximate number of messages in the queue. This number isn't lower than the actual number of messages in the queue, but could be higher.

```C#
if (queueClient.Exists())
{
    QueueProperties properties = queueClient.GetProperties();

    // Retrieve the cached approximate message count.
    int cachedMessagesCount = properties.ApproximateMessagesCount;

    // Display number of messages.
    Console.WriteLine($"Number of messages in queue: {cachedMessagesCount}");
}
```

Delete a queue

```C#
if (queueClient.Exists())
{
    // Delete the queue
    queueClient.Delete();
}
```

Create a storage account:

```
az storage account create \
    --resource-group $resourceGroup \
    --name $storAcctName \
    --location $location \
    --sku Standard_LRS
```

Create and assign the **Storage Queue Data Contributor** role.

```
az role assignment create \
    --assignee $userPrincipal \
    --role "Storage Queue Data Contributor" \
    --scope $resourceID
```
