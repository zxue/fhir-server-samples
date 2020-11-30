# FHIR Server Samples

This respository contains example applications and scenarios that show use of the [FHIR Server for Azure](https://github.com/Microsoft/fhir-server) and the [Azure API for FHIR](https://docs.microsoft.com/azure/healthcare-apis).

The scenario is meant to illustrate how to connect a web application to the FHIR API. The scenario also illustrates features such as the SMART on FHIR Active Directory Proxy. It can be deployed using the Azure API for FHIR PaaS server:

<center><img src="images//fhir-server-samples-paas.png" width="512"></center>

Or the open source FHIR Server for Azure:

<center><img src="images//fhir-server-samples-oss.png" width="512"></center>

In both cases a storage account will be deploy and in this storage account there is a BLOB container called `fhirimport`, patient bundles generated with [Synthea](https://github.com/synthetichealth/synthea) can dumped in this storage container and they will be ingested into the FHIR server. The bulk ingestion is performed by an Azure Function.

The environments can also optionally be configured to support [`$export`](https://hl7.org/Fhir/uv/bulkdata/export/index.html). To enable `$export`, add the `-EnableExport $true` parameter to the script below. The `$export` operation will produce a new line delimited json (ndjson) for each resource type. These ndjson files are easily consumed with something like [Databricks](https://azure.microsoft.com/en-us/services/databricks/) (Apache-Spark). Please see the [analytics](analytics/) folder for some details and example queries. Note that the Databricks environment is not deployed automatically with the sandbox and must be set up separately.

> Note: To enable `$export` you must have subscription rights that allow you to set data plane access roles for storage accounts, e.g. you must be a subscription owner.

# Prerequisites

Before deploying the samples scenario make sure that you have `Az` and `AzureAd` powershell modules installed:

```PowerShell
Install-Module Az
Install-Module AzureAd
```

# Deployment

To deploy the sample scenario, take the following steps:

- Clone the repo and log into your Azure subscription
- Create a new or use an existing FHIR environment
- Deploy the data importer app (Azure Function)
- Deploy the sample apps including one dashboard and two SMART on FHIR apps

# Clone the repo and log into your Azure subscription

First clone this git repo and find the deployment scripts folder:

```PowerShell
git clone https://github.com/Microsoft/fhir-server-samples
cd fhir-server-samples/deploy/scripts
```

Log into your Azure subscription:

```PowerShell
Connect-AzAccount
```

Connect to Azure AD with:

```PowerShell
Connect-AzureAd -TenantDomain <AAD TenantDomain>
```
If you have multiple Azure subscriptions, you can choose a specific subscription.

```PowerShell
Get-AzSubscription
Set-AzContext -SubscriptionId "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
Get-AzContext -SubscriptionName "Your Subscription Name"
Get-AzContext
```

Note that the PowerShell scripts and Azure ARM templates are tested with the following versions.
- PowerShell: 5.1.19041.610
- Azure PowerShell Az Module: 4.7.0

You can verify PowerShell versions by running the following command lines.

```
#check PowerShell Az versions
Get-InstalledModule -Name Az -AllVersions

#check PowerShell versions
$PSVersionTable.PSVersion 
Get-Host | Select-Object Version

```

# Create a New or Use an Existing FHIR Environment

You can use an existing FHIR environment or create a new one to run the sample apps. To create a new FHIR environment, refer to the documentation for more detail. 

[Quickstart: Deploy Azure API for FHIR](https://docs.microsoft.com/en-us/azure/healthcare-apis/fhir-paas-portal-quickstart)

[Quickstart: Deploy Microsoft Open Source FHIR Server](https://github.com/microsoft/fhir-server/blob/master/docs/DefaultDeployment.md)

To use an existing FHIR environment, Azure API for FHIR or OSS FHIR server, make sure that you have registered a client application. For OSS FHIR server that has security enabled, make sure that you have registered a server application, as described in the documents.

Note: The data importer app and sample apps are desgined to work with a FHIR environment secured by Azure AD. If your OSS FHIR server has security disabled, follow the steps in the document to enable the security in order to run the sample apps.

# Deploy the Data Importer App (Azure Function)

Run the PowerShell scripts to deploy a data ingestion app called Importer, which is a Storage trigger Azure Function. You can modify the setting for scripts and replace the client application id and secret.

```PowerShell
#Update the parameters with your settings
$aadServiceClientId="Your client app id here"
$aadServiceClientSecret="your client app secret here"
$fhirsubid="your subscription id here"
$fhirrgname="resource group where the FHIR service is"
$fhirserviceurl="full url of your FHIR service, FHIR PaaS or FHIR OSS server"
$location="your new resource group location, e.g. westus2"

#Deploy the data importer app
$rgname=$fhirrgname+"new" 
$appNameImporter = $fhirservicemame+"importer"
$importerTemplate="https://github.com/microsoft/fhir-server-samples/blob/master/deploy/templates/azuredeploy-importer-1.json" 
Set-AzContext -SubscriptionId $fhirsubid
$rg = New-AzResourceGroup -Name $rgname  -Location $location -Force 
New-AzResourceGroupDeployment -TemplateUri $importerTemplate -ResourceGroupName $rgname -appNameImporter $appNameImporter  -fhirServiceUrl $fhirserviceurl -aadServiceClientId $aadServiceClientId -aadServiceClientSecret $aadServiceClientSecret
```

Once the importer app is deployed, you can upload FHIR bundles (json files) to the storage container named "fhirimport" in the storage account.

# Deploy the Sample apps including one Dashboard and Two SMART on FHIR Apps

Coming soon.

# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
