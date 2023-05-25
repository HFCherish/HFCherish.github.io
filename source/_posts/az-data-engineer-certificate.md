---
title: az data engineer certificate
toc: true
tags:
  - certificate
  - cloud
  - big data
date: 2020-10-10 17:03:32
---

[learning paths](https://docs.microsoft.com/zh-cn/learn/certifications/azure-data-engineer?tab=tab-learning-paths)

## On-premises Env vs Cloud

[link](https://docs.microsoft.com/en-us/learn/modules/evolving-world-of-data/3-systems-on-premise-vs-cloud)

The term *total cost of ownership* (TCO) describes the final cost of owning a given technology. In **on-premises systems**, TCO includes the following costs:

- Hardware
- Software licensing
- Labor (installation, upgrades, maintenance)
- Datacenter overhead (power, telecommunications, building, heating and cooling)

**Cloud systems** like Azure track costs by subscriptions. A subscription can be based on usage that's measured in compute units, hours, or transactions. The cost includes hardware, software, disk storage, and labor. Because of economies of scale, an on-premises system can rarely compete with the cloud in terms of the measurement of the service usage.

The cost of operating an on-premises server system rarely aligns with the actual usage of the system. In cloud systems, the cost usually aligns more closely with the actual usage.

#### Comment

> However, many companies use the cloud still in a wasting way. The cost in cloud systems in fact rarely aligns with the actual usage, too.
> 
> The main advantage of Cloud is that it can **be charged on usage**. Thus, this advantage only works when using the cloud by need. Otherwise, the cloud advantage evaporates, especially from the aspect of cost.
> 
> The advantages of cloud:
> 
> 1. charge on usage
> 2. enjoy the high quality and compresensive services of big company.

## Data types

[link](https://docs.microsoft.com/en-us/learn/modules/survey-the-azure-data-platform/2-structured-vs-non-structured)

For nonstructured Data, the data **structure is defined only when the data is read**. The difference in the definition point gives you flexibility to use the same source data for different outputs.

> JSON is in fact semistructured data.

Examples of nonstructured data include binary, audio, and image files. 

NoSQL is in fact semistructured data. The open-source world offers four types of NoSQL databases:

1. **Key-value store**: Stores key-value pairs of data in a table structure.
2. **Document database**: Stores documents that are **tagged with metadata** to aid document searches.
3. **Graph database**: Finds relationships between data points by using a structure that's composed of vertices and edges.
4. **Column database**: Stores data based on columns rather than rows. Columns can be defined at the query's runtime, allowing flexibility in the data that's returned performantly.

### Azure Storage

[Azure Storage account](https://docs.microsoft.com/en-us/learn/modules/survey-the-azure-data-platform/3-store-data-using-azure-store-account) is the base storage type. It's mainly used to store data but with poor or no query ability.

Azure Storage offers four configuration options:

- **Azure Blob**: A scalable object store for text and binary data. The cheapst choice to store bot not query data.
- **Azure Files**: Managed file shares for cloud or on-premises deployments
- **Azure Queue**: A messaging store for reliable messaging between application components
- **Azure Table**: A NoSQL store for no-schema storage of structured data

## Tasks of an Azure data engineer:

 [link](https://docs.microsoft.com/en-us/learn/modules/data-engineering-processes/3-data-engineering-practices)

[example](https://docs.microsoft.com/zh-cn/learn/modules/data-engineering-processes/4-architecturing-project)

[Data Engineer vs Data Scientist vs AI engineer](https://docs.microsoft.com/en-us/learn/modules/data-engineering-processes/2-roles-and-responsibilities): Data engineer decide how to organized the data and pre-process the data. Data scientist use the result of Data engineer to create analysis model, and extract value. AI engineer use the existed model & tools to process the data. AI engineer may need the help of Data engineer to store the result, and the help of Data analysis to generate new model.

Here are some of the tasks of an Azure data engineer:

- Design and develop data storage and data processing solutions for the enterprise.
- Set up and deploy cloud-based data services such as blob services, databases, and analytics.
- Secure the platform and the stored data. Make sure only the necessary users can access the data.
- Ensure business continuity in uncommon conditions by using techniques for high availability and disaster recovery.
- Monitor to ensure that the systems run properly and are cost-effective.

## Plan the data storage solution

[Determine operational needs](https://docs.microsoft.com/en-us/learn/modules/choose-storage-approach-in-azure/3-operations-and-latency)

What are the main operations you'll be completing on each data type, and what are the performance requirements?

Ask yourself these questions:

- Will you be doing simple lookups using an ID?
- Do you need to query the database for one or more fields?
- How many create, update, and delete operations do you expect?
- Do you need to run complex analytical queries?
- How quickly do these operations need to complete?

[a storage solution example on the e-commerce system](https://docs.microsoft.com/en-us/learn/modules/choose-storage-approach-in-azure/5-choose-the-right-azure-service-for-your-data): 

* Product catalog data: cosmosDB
  
  * **Data classification:** Semi-structured because of the need to extend or modify the schema for new products
    
    **Operations:**
    
    - Customers require a high number of read operations, with the ability to query on many fields within the database.
    - The business requires a high number of write operations to track the constantly changing inventory.
    
    **Latency & throughput:** High throughput and low latency
    
    **Transactional support:** Required

* Photos & videos: azure blob (with azure CDN)
  
  * **Data classification:** Unstructured
    
    **Operations:**
    
    - Only need to be retrieved by ID.
    - Customers require a high number of read operations with low latency.
    - Creates and updates will be somewhat infrequent and can have higher latency than read operations.
    
    **Latency & throughput:** Retrievals by ID need to support low latency and high throughput. Creates and updates can have higher latency than read operations.
    
    **Transactional support:** Not required

* Buisiness Data: azure sql (with azure analysis services)
  
  * **Data classification:** Structured
    
    **Operations:** Read-only, complex analytical queries across multiple databases
    
    **Latency & throughput:** Some latency in the results is expected based on the complex nature of the queries.
    
    **Transactional support:** Required

# Questions

## Private cloud vs Public Cloud vs Specific Cloud

- [x] Are there still private cloud and public cloud??    ===> yes

- [ ] What's the difference? The private cloud will have higher quality cloud? The so-called SLA of public cloud is in fact not assured??

- [x] Maybe there're also specific cloud, like financial cloud (maybe more secure and fast) ===> yes

#### Points:

> 1. There is specific cloud. Mayi has financial cloud.
> 2. There are both public cloud & private cloud, but they're closely related. That is, they use the same cloud technology (images), but with different clusters & differnt tariff.

## Azure storage account

1. how to configure to use different type?
2. Why they're all in storage account instead of as independant service??
3. azure blob vs azure file storage

## Column database

1. How the data is organized in the hardware?
2. How should we save the data into column database?
3. Wide Column Stores
4. BigTable

Cosmosdb(Cassendra) vs HBase: infrastructure, load-balance???

Cassendra: Wide Column Stores

## ELT vs ELTL

- [ ] In fact, often ELTL???
- [ ] How is the data stored in T(transform)???

## Data Types

- [ ] what's markup language???
- [ ] why yaml is not markup language but json is???

## Azure SQL

- [ ] how azure sql support queries across multiple databases??
- [ ] Why does azure sql data warehouse not support???
