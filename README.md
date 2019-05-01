# Azure analysis Service processing few tables using ADF ,logic apps and config file from Blob storage

In this Artcile i will explain how to automate the processing of selected tables in Azure analysis service using ADF v2, Logic apps and Config file stored in BLOB.

There is already sample avilable for [Azure analysis service Full Refresh using Logic apps](https://docs.microsoft.com/en-us/azure/data-factory/concepts-data-flow-overview) . Follow this example if you are doing full refresh.

#### But, some customer dont want to process full AAS cube and want to process only few tables in Azure analysis service based on the config file .They can add remove tables to be processed from config file stored in Blob. this example will cover that part.

## Prerequisite :

1. [Create Azure Subscription](https://azure.microsoft.com/en-us/free/search/?&OCID=AID719825_SEM_cZGgGOIg&lnkd=Google_Azure_Brand&gclid=Cj0KCQjwh6XmBRDRARIsAKNInDGJOdHj9r9XuFXljGG4D5YgWzmnKTIugex27I8fQtSDyaBjIOO3zSoaAiJJEALw_wcB)
2. [Create Azure Analysis service](https://docs.microsoft.com/en-us/azure/analysis-services/analysis-services-create-server)
3. [Create Azure data Factory V2](https://docs.microsoft.com/en-us/azure/data-factory/quickstart-create-data-factory-portal)
4. Create Azure Logic Apps (https://docs.microsoft.com/en-us/azure/logic-apps/quickstart-create-first-logic-app-workflow)


## Abstract
### Step 1 : Create App Registration in Azure active directory:

##### Navigate to the Azure Active Directory page. On the Overview page, click “App registrations” followed by “+ New application registration”.
![App reg](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/Appreg.png)

##### Enter a name for your App registration, select Web App / API for application type and enter a dummy Sign-on URL.
![App reg Create](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/Appregcreate.PNG)

##### In the Settings section for your App registration, click Required permissions set up the following permission as below.

![permisiion 1](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/appregrequiredpermission1.PNG)

![permisiion 2](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/appregrequiredpermission2.PNG)

![permisiion 3](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/appregrequiredpermission3.PNG)

##### In the Settings section for your App registration,  click Keys.Choose the key description , duration and save. make a note of key 
and you need this in step 2
![Key](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/AppregKey.PNG)

##### Take a note of applicatioon id :
![appid](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/AppregApplicationId.PNG)

##### In step 2 you need Tenan id (Directory id), Navigate to the Azure portal, go to the Azure Active Directory page and click Properties. Now take note of the “Directory ID”.:

![Tenatid](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/AppregTenantid.PNG)

##### Grant App Registration permissions to process your Azure Analysis Services model.
Connect to Azure analysis service using SSMS.To process models using the API, the App Registration needs Server administrator permissions.Open the Analysis Services Server Properties, click Security and click Add. You can add the App Registration as a manual entry using the Application ID (app guid) and the Azure Active Directory ID (tenant guid) that you saved before. Use the following syntax:  app:<app guid>@<tenant guid>
 
![AAS permission](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/AAS.PNG)

### Step 2. Create logic app

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

### Step 3: Create Config file and upload to Blob Storage .
##### In my case i added 4 tables which i want to process in my cube as in below format. "Schema" is header in the file. [Download sample file](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/AASSchema.txt).

schema
[{"table": "DimCurrency"},{"table": "DimDate"},{"table": "DimProduct"},{"table": "FactInternetSales"}]

##### Upload this file to Blob Storage.

### Step 4. Create Azure data factory pipeline.

##### In ADF v2 visual tool drag and drop Lookup and Web activity as below :

![Activity](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/ADFactivity.PNG)

##### Click on Look up activity ,under setting tab choose source data set as config file created in step 3.

![Lookup](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/LookupActivity.PNG)

##### Click on web activity and in setting configure belwo proerty :

###### URI :Paste URI copied from Logic apps in Step 2 
###### Method : Choose method as Post
###### Header : add new header ,a dn provide Name : Content-Type , Value : application/json
###### Body   : Add below Json to Body and make sure use right Lookup activity name (In may example "getmetadata" and header of the file in my example file hesder "schema")
{
      "aastables": @{activity('getmetadata').output.firstRow.schema}
}

##### Web activity should like below:

![Webactivity](https://github.com/nikris87/AzureanalysisService_ADF_Processing/blob/master/Webactivity.PNG)


### Step 4. Save and trigger ADF pipeline . Test the Azure analysis service.



