---
title: Copy data from or to MongoDB Atlas
titleSuffix: Azure Data Factory & Azure Synapse
description: Learn how to copy data from MongoDB Atlas to supported sink data stores, or from supported source data stores to MongoDB Atlas, by using a copy activity in an Azure Data Factory pipeline.
ms.author: chez
author: chez-charlie
ms.service: data-factory
ms.subservice: data-movement
ms.topic: conceptual
ms.custom: synapse
ms.date: 06/01/2021
---

# Copy data from or to MongoDB Atlas using Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

This article outlines how to use the Copy Activity in Azure Data Factory to copy data from and to a MongoDB Atlas database. It builds on the [copy activity overview](copy-activity-overview.md) article that presents a general overview of copy activity.

## Supported capabilities

You can copy data from MongoDB Atlas database to any supported sink data store, or copy data from any supported source data store to MongoDB Atlas database. For a list of data stores that are supported as sources/sinks by the copy activity, see the [Supported data stores](copy-activity-overview.md#supported-data-stores-and-formats) table.

Specifically, this MongoDB Atlas connector supports **versions up to 4.2**.

## Prerequisites

If you use Azure Integration Runtime for copy, make sure you add the effective region's [Azure Integration Runtime IPs](azure-integration-runtime-ip-addresses.md) to the MongoDB Atlas IP Access List.

## Getting started

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

The following sections provide details about properties that are used to define Data Factory entities specific to MongoDB Atlas connector.

## Linked service properties

The following properties are supported for MongoDB Atlas linked service:

| Property | Description | Required |
|:--- |:--- |:--- |
| type |The type property must be set to: **MongoDbAtlas** |Yes |
| connectionString |Specify the MongoDB Atlas connection string e.g. `mongodb+srv://<username>:<password>@<clustername>.<randomString>.<hostName>/<dbname>?<otherProperties>`. <br/><br /> You can also put a connection string in Azure Key Vault. Refer to [Store credentials in Azure Key Vault](store-credentials-in-key-vault.md) with more details. |Yes |
| database | Name of the database that you want to access. | Yes |
| connectVia | The [Integration Runtime](concepts-integration-runtime.md) to be used to connect to the data store. Learn more from [Prerequisites](#prerequisites) section. If not specified, it uses the default Azure Integration Runtime. |No |

**Example:**

```json
{
    "name": "MongoDbAtlasLinkedService",
    "properties": {
        "type": "MongoDbAtlas",
        "typeProperties": {
            "connectionString": "mongodb+srv://<username>:<password>@<clustername>.<randomString>.<hostName>/<dbname>?<otherProperties>",
            "database": "myDatabase"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## Dataset properties

For a full list of sections and properties that are available for defining datasets, see [Datasets and linked services](concepts-datasets-linked-services.md). The following properties are supported for MongoDB Atlas dataset:

| Property | Description | Required |
|:--- |:--- |:--- |
| type | The type property of the dataset must be set to: **MongoDbAtlasCollection** | Yes |
| collectionName |Name of the collection in MongoDB Atlas database. |Yes |

**Example:**

```json
{
    "name": "MongoDbAtlasDataset",
    "properties": {
        "type": "MongoDbAtlasCollection",
        "typeProperties": {
            "collectionName": "<Collection name>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<MongoDB Atlas linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## Copy activity properties

For a full list of sections and properties available for defining activities, see the [Pipelines](concepts-pipelines-activities.md) article. This section provides a list of properties supported by MongoDB Atlas source and sink.

### MongoDB Atlas as source

The following properties are supported in the copy activity **source** section:

| Property | Description | Required |
|:--- |:--- |:--- |
| type | The type property of the copy activity source must be set to: **MongoDbAtlasSource** | Yes |
| filter | Specifies selection filter using query operators. To return all documents in a collection, omit this parameter or pass an empty document ({}). | No |
| cursorMethods.project | Specifies the fields to return in the documents for projection. To return all fields in the matching documents, omit this parameter. | No |
| cursorMethods.sort | Specifies the order in which the query returns matching documents. Refer to [cursor.sort()](https://docs.mongodb.com/manual/reference/method/cursor.sort/#cursor.sort). | No |
| cursorMethods.limit |	Specifies the maximum number of documents the server returns. Refer to [cursor.limit()](https://docs.mongodb.com/manual/reference/method/cursor.limit/#cursor.limit).  | No |
| cursorMethods.skip | Specifies the number of documents to skip and from where MongoDB Atlas begins to return results. Refer to [cursor.skip()](https://docs.mongodb.com/manual/reference/method/cursor.skip/#cursor.skip). | No |
| batchSize | Specifies the number of documents to return in each batch of the response from MongoDB Atlas instance. In most cases, modifying the batch size will not affect the user or the application. Cosmos DB limits each batch cannot exceed 40MB in size, which is the sum of the batchSize number of documents' size, so decrease this value if your document size being large. | No<br/>(the default is **100**) |

>[!TIP]
>ADF support consuming BSON document in **Strict mode**. Make sure your filter query is in Strict mode instead of Shell mode. More description can be found at [MongoDB manual](https://docs.mongodb.com/manual/reference/mongodb-extended-json/index.html).

**Example:**

```json
"activities":[
    {
        "name": "CopyFromMongoDbAtlas",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<MongoDB Atlas input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "MongoDbAtlasSource",
                "filter": "{datetimeData: {$gte: ISODate(\"2018-12-11T00:00:00.000Z\"),$lt: ISODate(\"2018-12-12T00:00:00.000Z\")}, _id: ObjectId(\"5acd7c3d0000000000000000\") }",
                "cursorMethods": {
                    "project": "{ _id : 1, name : 1, age: 1, datetimeData: 1 }",
                    "sort": "{ age : 1 }",
                    "skip": 3,
                    "limit": 3
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### MongoDB Atlas as sink

The following properties are supported in the Copy Activity **sink** section:

| Property | Description | Required |
|:--- |:--- |:--- |
| type | The **type** property of the Copy Activity sink must be set to **MongoDbAtlasSink**. |Yes |
| writeBehavior |Describes how to write data to MongoDB Atlas. Allowed values: **insert** and **upsert**.<br/><br/>The behavior of **upsert** is to replace the document if a document with the same `_id` already exists; otherwise, insert the document.<br /><br />**Note**: Data Factory automatically generates an `_id` for a document if an `_id` isn't specified either in the original document or by column mapping. This means that you must ensure that, for **upsert** to work as expected, your document has an ID. |No<br />(the default is **insert**) |
| writeBatchSize | The **writeBatchSize** property controls the size of documents to write in each batch. You can try increasing the value for **writeBatchSize** to improve performance and decreasing the value if your document size being large. |No<br />(the default is **10,000**) |
| writeBatchTimeout | The wait time for the batch insert operation to finish before it times out. The allowed value is timespan. | No<br/>(the default is **00:30:00** - 30 minutes) |

>[!TIP]
>To import JSON documents as-is, refer to [Import or export JSON documents](#import-and-export-json-documents) section; to copy from tabular-shaped data, refer to [Schema mapping](#schema-mapping).

**Example**

```json
"activities":[
    {
        "name": "CopyToMongoDBAtlas",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Document DB output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "MongoDbAtlasSink",
                "writeBehavior": "upsert"
            }
        }
    }
]
```

## Import and Export JSON documents

You can use this MongoDB Atlas connector to easily:

* Copy documents between two MongoDB Atlas collections as-is.
* Import JSON documents from various sources to MongoDB Atlas, including from Azure Cosmos DB, Azure Blob storage, Azure Data Lake Store, and other file-based stores that Azure Data Factory supports.
* Export JSON documents from a MongoDB Atlas collection to various file-based stores.

To achieve such schema-agnostic copy, skip the "structure" (also called *schema*) section in dataset and schema mapping in copy activity.


## Schema mapping

To copy data from MongoDB Atlas to tabular sink or reversed, refer to [schema mapping](copy-activity-schema-and-type-mapping.md#schema-mapping).

## Next steps
For a list of data stores supported as sources and sinks by the copy activity in Azure Data Factory, see [supported data stores](copy-activity-overview.md#supported-data-stores-and-formats).
