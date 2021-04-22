# Overview
Welcome to the private preview of the delete by partition key feature in Azure Cosmos DB! This repo contains documentation and a link to the latest private SDK version with the feature enabled. 

## Feature overview
 
The delete by partition key feature is an asynchronous, background operation that allows you to delete all documents with the same logical partition key value, using the Comsos SDK.

Because the number of documents to be deleted may be large, the operation runs in the background. Though the physical deletion operation runs in the background, the effects will be available immediately, as the documents to be deleted will not appear in the results of queries or read operations. 

To help govern the resources used by this background task, the delete by partition key operation consumes up to a pre-set fraction of the total available RU/s on the container. By default, the fraction is set to 10%. 

> [!IMPORTANT]
> This feature is currently in private preview. You can find the private preview drop of the .NET V3 SDK that supports this feature in this repo. Support for other SDKs is planned and not yet available. 

## Sample code
Use the private preview of version 3.x of the Azure Cosmos DB .NET SDK to delete items by partition key. 

```csharp
// Suppose our container is partitioned by tenantId, and we want to delete all the data for a particular tenant Contoso

// Get reference to the container
var container = cosmosClient.GetContainer("DatabaseName", "ContainerName");

// Delete by logical partition key
ResponseMessage deleteResponse = await container.DeleteAllItemsByPartitionKeyStreamAsync(new PartitionKey("Contoso"));

 if (deleteResponse.IsSuccessStatusCode) {
    Console.WriteLine($"Delete all documents with partition key operation has successfully started");
}
```
#### [Java](#tab/java-example)
```java
// Java support is not yet available
```

#### [Python](#tab/python-example)
```python
// Python support is not yet available
```

#### [JavaScript](#tab/javascript-example)
```javascript
// JavaScript support is not yet available
```
--- 

### Frequently asked questions (FAQ)
#### Are the results of the delete by partition key operation reflected immediately?
Yes, once the delete by partition key operation starts, the documents to be deleted will not appear in the results of queries or read operations. This also means that you can write new a new document with the same id and partition key as a document to be deleted without resulting in a conflict.

See [Known issues](#known-issues) for exceptions. 

#### What happens if I issue a delete by partition key operation, and then immediately write a new document with the same partition key?
When the delete by partition key operation is issued, only the documents that exist in the container at that point in time with the partition key value will be deleted. Any new documents that come in will not be in scope for the deletion. 

#### How is the delete by partition key operation prioritized among other operations against the container?
By default, the delete by partition key value operation can consume up to a reserved fraction - 0.1, or 10% - of the overall RU/s on the resource. Any Request Units (RUs) in this bucket that are unused will be available for other non-background operations, such as reads, writes, and queries. 

For example, suppose you have provisioned 1000 RU/s on a container. There is an ongoing delete by partition key operation that consumes 100 RUs each second for 5 seconds. During each of these 5 seconds, there are 900 RUs available for non-background database operations. Once the delete operation is complete, all 1000 RU/s are now available again. 

#### How do I set a custom user-defined fraction on the allowed throughput?
Currently, the 0.1 fraction is set by the Cosmos DB service. We are looking into ways to expose this control to the user and would welcome your feedback. Let us know by filing an issue with your feedback in this repo.

### Known issues
For certain scenarios, the effects of a delete by partition key operation is not guaranteed to be immediately reflected. The effect may be partially seen as the operation progresses. 

- Aggregate queries that use the index - for example, COUNT queries - that are issued during an ongoing delete by partition key operation may contain the results of the documents to be deleted. This may occur until the delete operation is fully complete.

- Queries issued against the analytical store during an ongoing delete by partition key operation may contain the results of the documents to be deleted. This may occur until the delete operation is fully complete.

## How to give feedback or report an issue/bug.
* Create an issue in this repo with your feedback/issue/bug.
