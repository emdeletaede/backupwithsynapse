# backupwithsynapse

---
page_type: sample
languages:
- spark , synapse 
products:
- azure-cosmosdb , synapse 
description: "This sample demonstrates a sample of how make your own backup and queryable backup and long term backup 

---
# build a backup system for comsosdb  without entreprise tools 

## About this sample

> This sample demostrate how to build in addition of the existing backup policies that you can find here  
https://docs.microsoft.com/en-us/azure/cosmos-db/online-backup-and-restore 
a more granular backup for the cosmosdb . 


### Overview





Cosmosdb have native integrate backup Systems in the recent update of Cosmosdb we offer the possibility to modify the number of back up and the frequency of the backup. For most of the clients this is enough, remember the backup is not here for HA but an anti-corruption backup or a human error of delete of the DB. 
Some clients need for legacy reason to keep and have long term backup and some other would like to have query able backup. this article will show you how with synapse link you can quickly make this using Azure Synapse Link 
Azure Synapse Link for Azure Cosmos DB is a cloud-native hybrid transactional and analytical processing (HTAP) capability that enables you to run near real-time analytics over operational data in Azure Cosmos DB. Azure Synapse Link creates a tight seamless integration between Azure Cosmos DB and Azure Synapse Analytics. 
The following image shows the Azure Synapse Link integration with Azure Cosmos DB and Azure Synapse Analytics:

![Cosmos DB account](media/.png)  

 
The Azure Synapse Link can be use with Cosmosdb SQL API and with Mongo API, so all I will present and explain can be done for both type of API.

Azure blob storage offer a solution to store business critical data, you can use immutable storage
Immutable storage for Azure Blob storage enables users to store business-critical data objects in a WORM (Write Once, Read Many) state. This state makes the data non-erasable and non-modifiable for a user-specified interval. For the duration of the retention interval, blobs can be created and read, but cannot be modified or deleted. 
The main idea of the solutions is using the synapse link for Cosmosdb and synapse link for storage to read the Cosmosdb data and write to immutable storage, now let me show you how to make. 
In synapse create a link to your Cosmosdb and to your blob storage, link in the sample 

![Cosmos DB account](media/.png)  
  
Now let’s open an notebook sparks in synapse and use the following instruction 
1.	Read the data from comsosdb using the olap storage and have no RU consume 
# Read from Cosmos DB analytical store into a Spark DataFrame and display 10 rows from the DataFrame
# To select a preferred list of regions in a multi-region Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

```python
tosave = spark.read\
    .format("cosmos.olap")\
    .option("spark.synapse.linkedService", "mongoAPI")\
    .load()
display(tosave.limit(10))

```
2.	Write your data to your immutable storage. Of course your need to create the folder when you want to write your data 
tosave.write.json('abfss://ede@edeadlsgen2.dfs.core.windows.net/ede/bakup/ede.json')
3.	In case of restore need, you have just to load your data in a dataset by make a read in synapse using the instruction below 

```python
%%pyspark
restore = spark.read.load('abfss://ede@edeadlsgen2.dfs.core.windows.net/ede/bakup/ede.json', format='json')
display(df2.limit(10))
```

4.	Now you want to restore your backup to cosmosdb just write the dataframe to cosmosdb , this operations will consume RU but will write your data into your cosmosdb 
```python
restore.write\
    .format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "mongoAPI")\
    .option("spark.cosmos.container", "ede")\
    .option("spark.cosmos.write.upsertEnabled", "true")\
    .mode('append')\
    .save()
```

5.	If you want to restore not all the data but a subset of data this is very easy, just query the data you want to restore in the data frame and write like in the sample below 
a.	Query using _ts columns for exemple 
```python

restorequery = restore[(restore._ts <= 1605517579) ]
display(df3)
```

b.	And not write the new dataframe 
```python
restorequery.write\
    .format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "mongoAPI")\
    .option("spark.cosmos.container", "ede")\
    .option("spark.cosmos.write.upsertEnabled", "true")\
    .mode('append')\
    .save()
```

Try for free , if you have questions contact me emdeleta@microsoft.com
