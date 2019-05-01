# Azure analysis Service processing fewer tables using ADF ,logic apps anf config file from Blob storage

In this Artcile i will explain how to automate to process selected tables in Azure analysis service using ADF v2, Logic apps and Config file stored in BLOB.

There is already sample avilable to [Full Refresh of Azure analysis service using Logic apps](https://docs.microsoft.com/en-us/azure/data-factory/concepts-data-flow-overview) .

#### But, some customer dont want to process full AAS and wants to process only few tables in Azure analysis service based on the config file . this example will cover that part.

## Prerequisite :

