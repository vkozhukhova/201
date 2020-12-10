# Intro

## 3V

* volume
* velocity
* variety

### Volume

amount of data that is consumed or stored

### Velocity

a rate at which data comes in: real-time or not real-time

### Variety

we have structured, semi-structured and unstructured data

### Veracity \(Достоверность\)

* How accurate is that data in predicting business value
* Do the results of a big data analysis actually make sense
* Data must be able to be verified based on both accuracy and context. 
* it is necessary to identify the right amount and types of data that can be analyzed in real time to impact business outcomes.

Pluses:

* improve decision-making, collect more data
* cost reduction: cloud. open-source
* new data sources
* new analytics, beteer customer needs satisfaction

Data pyramid \(shows value of data\)

1. Raw
2. Information - data with basic context
3. Knowledge - data with business context or function - what is happening
4. Insight - understanding the question - business context, finction and related info - why this is happenning - here comes data science to retrieve future insights based on insights
5. Future insight - predict what may happen

**Data** - representation of **facts** as text, images, sound, graphics, video. Data is always right.

We use data to create information.

**Information** - data cleaned of errors and further processed in a way that makes it easier to measure, visualize. **Data processing** = aggregation \(combining data sources\), validation \(ensuring data is relevant and accurate\). Information can be wrong. Answers questions: who, what, when, where

**Knowledge** - answers question HOW. Here we understand how to apply information to our business goals and therefore turn it into knowledge.

**Insights** - deeper relations in info, that are not obvious, DS analysis.

## Types of data

### Structured data

* highly organised data that has a defined length and format for big data. 
* include numbers, dates, and groups of words and numbers called strings. 
* about 20 percent of the data that is out there. 
* usually is uploaded & stored in a **RDBMS**.
* easily detectable with search
* simple to store, query, analyze
* strict field and type
* SQL
* csv, tsv, data tables in RDBMS

### Semi-structured data

* contain tags or other markers to separate semantic elements and enforce hierarchies of records and fields within the data. 
* is also known as self-describing structure. 
* Log, JSON and XML are forms of semi-structured data.
* 5-10 % of data
* easier to analyse than unstructured data

### Unstructured Data

* information that either does not have a predefined data model or is not organised in a pre-defined manner. 
* typically text-heavy, but may contain data such as dates, numbers, and facts as well. 
* irregularities and ambiguities that make it difficult to understand using traditional programs 
* audio, video files or No-SQL databases, pictures, videos or PDF documents, social media, emails,...
* The ability to extract value from unstructured data is one of main drivers behind the quick growth of Big Data.
* MongoDB is optimised to store documents.
* Apache Giraph is optimised for storing relationships between nodes.

### Metadata

is data about data. It provides additional information about a specific set of data.

## Schema on write

the approach in which we define the columns, data format, relationships of columns, etc. before the actual data upload.

* collect data
* apply schema
* write data
* analyze

Pluses:

* fsat read: after loading data reads will be fast and determinant
* SQL
* fewer errors in data
* structured

Minuses:

* slower load
* not agile
* we can't upload data until the table is created 
* we can't create tables until we understand the schema of the data that will be in this table.
* changing the data leads to dropping table and loading it once again
* the data has been modified and structured for a specified limited purpose and cannot be reused for future uses that we do not know yet.

## Schema on read

we upload data as it arrives without any changes or transformations. we completely remove the original ETL process and soothe the nerves from understanding the original data patterns and their structure.

* collect data
* write data
* apply schema
* analyze

Pluses:

* structuread & unstructured data storage
* fast load: fast data ingestion because data shouldn't follow any internal schema — it's just copying/moving files.
* very agile : is more flexible in case of big data, unstructured data, or frequent schema changes.
* if we analyze the data and try to understand its structure and figure out another way to interpret it, we can simply change the code that is handling the data. We do not need to change the schemas and reload all the data in the data storage.
* NoSQL

Minuses:

* slower read
* more errors: there can be a lot of missing or invalid data, duplicates, and many other problems that can lead to inaccurate or incomplete query results.

## Speed of data

* Batch processing: historical data analysis, updated periodically
* micro-batching: reactive speed, can react to data chenges, use streaming data

## File formats

### Raw

access pattern:

* use all fields to validate, enhance, join data
* read through whole dataset

formats:

* text - instructured
* csv, tsv, json - semi-structured
* binary - images, videos
* avro - row oriented

### Processed

access pattern:

* use limited fields to aggregate data or run queries
* read filtered dataset

formats:

* parquet \(columnar\)
* orc

## OLTP vs OLAP

are data processing systems

### OLTP - Online Transactional Processing

* handle transactional data, 
* deal with read as well as write operations, 
* records are updated frequently
* use normalization to reduce data redundancy, 
* Speed and data integrity are critical
* are built to record everyday business transactions; 
* are small in size \(up to 10 GB\), 
* deal with simple queries; 
* ROW-ORIENTED DB are suitable

### OLAP - Online Analytical Processing

* handle aggregated historical data.
* fetches data for analysis from OLTP systems and other databases
* handle only read operations.
* runs complex queries on large datasets for business intelligence.
* use denormalized data for multidimensional data analysis.
* speed can vary depending on the data being analyzed.
* are built for gathering business intelligence.
* can be massive, running into several petabytes of data.
* deal with complex data analysis algorithms.
* focus in OLAP systems is on fast query performance
* multidimensional indexing and caching are used
* query performance is still slower compared to an OLTP system, since complex queries are run on large datasets for several columns.
* COLUMN-ORIENTED data formats are suitable

## Ideal big data solution

* scalable
* fault tolerant

