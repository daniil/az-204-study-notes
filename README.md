### Deploying an app

```
az webapp up
```

Setting a `resourceGroup` variable:

```
az group list --query "[].{id:name}" -o tsv
```

Deploying an app with resourceGroup and appName variables provided:

```
az webapp up -g $resourceGroup -n $appName --html
```

Creating a resource group Creator:

```
az group create --name $resourceGroup --location eastus --tags Creator="<name-or-id>"
```

### Application settings

Application settings can be found at app's management page under _Environment variables > Application settings_.

To add or edit app settings in bulk, select the Advanced edit button.

**For custom containers**:

```bash
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings key1=value1 key2=value2
```

```PowerShell
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"DB_HOST"="myownserver.mysql.database.azure.com"}
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
az webapp log tail --name appname --resource-group myResourceGroup
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
az webapp deployment slot create -n $appName -g $resourceGroup --slot staging
```

To view deployment slots, select _Deployment > Deployment slots_

To zip up contents of current folder for deployment:

```
zip -r stagingcode.zip .
```

**Deploying to a deployment slot**

```
az webapp deploy -g $resourceGroup -n $appName --src-path ./stagingcode.zip --slot staging
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
