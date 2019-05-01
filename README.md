# Azure analysis Service processing fewer tables using ADF ,logic apps anf config file from Blob storage

In this Artcile i will explain how to automate to process selected tables in Azure analysis service using ADF v2, Logic apps and Config file stored in BLOB.

There is already sample avilable to [Full Refresh of Azure analysis service using Logic apps](https://docs.microsoft.com/en-us/azure/data-factory/concepts-data-flow-overview) .

#### But, some customer dont want to process full AAS and wants to process only few tables in Azure analysis service based on the config file .They can add remove tables to be processed from config file stored in Blob. this example will cover that part.

## Prerequisite :

1. [Create Azure Subscription](https://azure.microsoft.com/en-us/free/search/?&OCID=AID719825_SEM_cZGgGOIg&lnkd=Google_Azure_Brand&gclid=Cj0KCQjwh6XmBRDRARIsAKNInDGJOdHj9r9XuFXljGG4D5YgWzmnKTIugex27I8fQtSDyaBjIOO3zSoaAiJJEALw_wcB)
2. [Create Azure Analysis service](https://docs.microsoft.com/en-us/azure/analysis-services/analysis-services-create-server)
3. [Create Azure data Factory V2](https://docs.microsoft.com/en-us/azure/data-factory/quickstart-create-data-factory-portal)
4. Create Azure Logic Apps (https://docs.microsoft.com/en-us/azure/logic-apps/quickstart-create-first-logic-app-workflow)


## Abstract

Follow pre requisite guidelines to Create app Registration in this [link](https://jorgklein.com/2018/01/30/process-azure-analysis-services-objects-from-azure-data-factory-v2-using-a-logic-app/)

### Step 1. Create logic app

##### Craete new logic app from new resource in Azure portal. and then click edit

![Logic app edit](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/Createlogicapp_edit.PNG)

##### Select "When http request recieved" trigger from commonly used trigger list :

![http request trigger](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/CommonTrigger.PNG)

Expand Http request recieved trigger and add folling json :

{
    "properties": {
        "aastables": {
            "type": "string"
        }
    },
    "type": "object"
}

![http request recieved](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/httprequestrecieved.PNG)

##### Click on + New step and add HTTP action. Expand the http action and fill the detail as below.

###### Method : Get
###### URI : Refresh URI can be derived using your Azure analysis base URI. [Please follow this URI format](https://docs.microsoft.com/en-us/azure/analysis-services/analysis-services-async-refresh#base-url) .

###### Body : Copy pase below Json to Body content. For object part we are dynamically passing based on config file in Blob Storage.
###### [Please refer this link for refernce](https://docs.microsoft.com/en-us/azure/analysis-services/analysis-services-async-refresh#post-refreshes)

{
  "CommitMode": "transactional",
  "MaxParallelism": 2,
  "Objects": @{triggerBody()?['aastables']},
  "RetryCount": 2,
  "Type": "Full"
}

###### Authentication : Active Directory Aouth
###### Tenant : Use Tenat ID that derived from Pre requisite.
###### Audience :[https://*.asazure.windows.net](https://*.asazure.windows.net)
###### Client id : Use the app registration id that created as a part of pre requisite.
###### Credential Type : Secret
###### Secret : Use the App Registration Key that you saved before.

##### HTTP Action should look like this:
![http](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/HTTP.PNG)

##### Save the logic apps and open "When http request recieved" trigger and copy the post URL
![Copy post URI](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/CopyURI.PNG)

### Step 2. Create Config file and upload to Blob Storage .
##### In my case i added 4 tables which i want to process in my cube as in below format. "Schema" is header in the file. [Download sample file](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/AASSchema.txt).

schema
[{"table": "DimCurrency"},{"table": "DimDate"},{"table": "DimProduct"},{"table": "FactInternetSales"}]

##### Upload this file to Blob Storage.

### Step 3. Create Azure data factory pipeline.

##### In ADF v2 visual tool drag and drop Lookup and Web activity as below :

![Activity](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/ADFactivity.PNG)

##### Click on Look up activity ,under setting tab choose source data set as config file created in step 2.

![Lookup](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/LookupActivity.PNG)

##### Click on web activity and in setting configure belwo proerty :

###### URI :Paste URI copied from Logic apps in Step 1 
###### Method : Choose method as Post
###### Header : add new header ,a dn provide Name : Content-Type , Value : application/json
###### Body   : Add below Json to Body and make sure use right Lookup activity name (In may example "getmetadata" and header of the file in my example file hesder "schema")
{
      "aastables": @{activity('getmetadata').output.firstRow.schema}
}

##### Web activity should like below:

![Webactivity](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/Webactivity.PNG)


### Step 4. Save and trigger ADF pipeline . Test the Azure analysis service.



