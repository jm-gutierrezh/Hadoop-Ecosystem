# Hadoop-Ecosystem
Concepts behind the Hadoop ecosystem

---

# Hadoop

Hadoop is a distributed computing technology, which allows jobs to be processed in server farms, groups of connected computers.

There is a single co-ordinating software which must perform the following tasks:

* Partition data
* Co-ordinate computing tasks
* Handle fault tolerance and recovery
* Allocate capacity to different processes

Hadoop is composed of three base technologies:

* `HDFS` (Hadoop Distributed File System), a file system to manage **distributed storage**.
* `MapReduce`, a framework to manage **distributed computing**.
* `YARN` (Yet Another Resource Negotiator), a framework to run and manage the data processing tasks.

---

# HDFS

Hadoop Distributed File System

* Highly fault tolerant
* Suited to batch processing - data access has high throughput rather than low latency
* Supports very large data sets

How does HDFS manage file distribution?

One of the nodes in the cluster is selected as the master node, or **Name Node**, which contains all the metadata required to co-ordinate the other nodes. The Name node stores the directory structure and the metadata of the files, not the files themselves.

HDFS stores files in blocks of 128 MB, which are then stored in a distributed way in the cluster. Block size offers a tradeoff between parallelism and overhead. A larger block size decreases parallelism but increases overhead.

To read files in HDFS:

1. Use metadata in the Name node to lookup block locations.
2. Read the blocks from their respective locations.

To allow for fault tolerance (file corruption or node crashing):

1. Replicate blocks based on the replication factor (usually 3).
2. Store replicas in different locations.

Replication balances a tradeoff between redundancy and write bandwidth. More replication increases redundancy (and fault tolerance), but decreases write bandwidth.

---

# MapReduce

MapReduce is a programming paradigm for distributed computing. Consists of two operations: Map and Reduce.

Map is a parallel operation which takes in a record and returns a (key, value) pair.

Combines the results of the Map operations by their keys and returns a result.

Usually, MapReduce jobs are programmed in `Java`, by implementing three classes:

* Mapper class
* Reducer class
* Main class

---

# YARN

Yet Another Resource Negotiator, is a program in charge of co-ordinating tasks running on the cluster.

YARN has two distinct subcomponents: 

|Resource Manager|Node Manager|
|---|---|
|Runs on a single master node|Runs on all the other nodes|
|Schedules tasks across nodes|Manages tasks on the individual node|

The job submission process is as follows:

1. A MapReduce job is submitted to the ResourceManager
2. The Resource Manager finds a NodeManager with free capacity
3. The NodeManager runs the job
4. NodeManagers can request containers for Mappers and Reducers, or request CPU or memory requirements

---

# Hive

Hive provides an SQL interface to Hadoop which runs all of its internal processes as MapReduce jobs.

Hive exposes files in HDFS in the form of tables to the user through a component known as the Hive metastore, which is basically a bridge between HDFS and Hive.

The Hive metastore:

* Usually is a relational database itself
* Stores metadata for a tables in Hive
* Maps the files and directories in Hive to tables
* Holds table definitions and the schema for each table
* Any database with a JDBC driver can be used as a metastore

#### Hive vs RDBMS

|Factor|Hive|RDBMS|
|---|---|---|
|Data size|Large datasets (petabytes)|Small datasets (gigabytes)|
|Computation|Parallel computation (horizontal scaling)|Serial computation (vertical scaling)|
|Latency|High latency (no indexing)|Low latency (indexing)|
|Operations|Force schema-on-read|Force schema-on-write|
|ACID compliance|Yes (data can be dumped into tables from any source)|No (strict check for storing)|
|Query language|HiveQL|SQL|

#### Partitioning

Allows for data to be split into logical units, each unit will be stored in a different directory. Queries referencing partitions will run only on the required directories.

Note that partitioned units are not necessarily the same size.

#### Bucketing

Bucketing allows for partition of data into equally-sized units, each unit will be stored in a different file. Bucketing accomplished equal-size partitioning through hashing a column value.

#### Queries on Big Data

* Partitioning and bucketing of tables
* Join optimizations through reduction of required memory
  * In joins, one table is held in memory while the other is read from disk
  * For best performance, hold the smaller table in memory
  * Another optimization is to write joins as map-only operations
* Window functions make complex operations simple without needing many intermediate calculations, through the definition of a **window** and an **operation**

---

# HBase

A database management system on top of Hadoop. Integrates with applications just like a traditional database.

---

# Pig

Pig is a procedural, data flow language to transform unstructured or inconsistent data into a structured format. Is used to get data into the data warehouse.

Is focused on sequential transformations applied to the data.

---

# Spark

A distributed computing engine used to quickly process datasets. Has a bunch of built-in libraries for machine learning, stream processing and graph processing.

It is a general purpose engine for exploring, cleaning and preparing data, applying machine learning and building data applications.

Data in Spark is abstracted through RDDs or **Resilient Distributed Datasets**, which are in-memory collections of objects.

The basic functionality of Spark is the **Spark Core**, which is the computing engine. The Spark Core requires two additional components: the **Storage System** (HDFS), which stores the data to be processed, and the **Cluster Manager** (YARN), which helps Spark run tasks across a cluster of machines. In this way, Spark replaces MapReduce.

The **SparkContext** represents a connection to the Spark Cluster (which is the Hadoop cluster, as now we're using the Storage System and Cluster Manager).

#### RDDs

* The core abstraction of Spark
* RDDs don't execute, they just store metadata
* Partitioning
  * Data is divided into partitions
  * Distributed to multiple machines
* Immutability
  * RDDs are read-only
  * Only two types of operations: Transformations and Actions
  * Transformations transform the RDD into another RDD
  * Actions request results and materialize the RDDs
* Lineage
  * Every RDD remember its previous transformations
  
#### Lazy Evaluation

1. Spark keeps a record of the series of transformations requested, but doesn't evaluate them
2. Spark groups the transformations in an efficient way when an Action is requested

#### Installing Spark

1. Download Spark binaries
2. Update environment variables
   * SPARK_HOME
   * PATH
3. Configure iPython notebook for Spark
   * PYSPARK_DRIVER_PYTHON
   * PYSPARK_DRIVER_PYTHON_OPTS
   
---

# Oozie 

A tool to schedule workflows on all Hadoop ecosystem technologies.

---

# Kafka

A tool for stream processing for unbounded datasets.

---

# Flink

A tool for stream processing for unbounded datasets, similar to Kafka and SparkStreaming.

Starts with a distinction:

* Bounded datasets are processed in batches.
* Unbounded datasets are processed in streams.

*Unbounded datasets are those sets in which data is continously added until infinity.*

#### Batch vs Stream Processing

|Batch|Stream|
|---|---|
|Bounded, finite datasets|Unbounded, infinite datasets|
|Slow pipeline from data ingestion to analysis|Processing immediate, as data is received|
|Periodic updates as jobs complete|Continuous updates as jobs run constantly|
|Order of data received unimportant|Order important, out of order arrival tracked|
|Single global state at any point in time|No global state, only history of events received|

#### Stream Processing

1. Data is received as a stream (example: log messages, tweets, sensors)
2. Process the data one entity at a time (example: filter error messages, find references to latest movies, track weather patterns)
3. Must have a mechanism to store, display and act on filtered messages (example: trigger an alert, show trends, send warning)

#### Stream-first Architecture

* Used when data sources include streams
* Has two components: **Message Transport** and **Stream Processing**

#### Message Transport

* Buffer for event data
* High-performance and persistance
* Decoupling multiple sources from processing
* Kafka and MapR are Message Transport technologies

#### Stream Processing

* High throughput, low latency
* Fault tolerance with low overhead
* Manage out of order events (events that come at the wrong time)
* Easy to use, maintenable
* Ability to replay streams
* SparkStreaming, Storm and Flink are Stream Processing technologies

#### Microbatches

* Is an approximation to stream processing
* Grouping data in small batches. In the limit, as n_batches tends to infinity, Stream Processing emerges
* Allows for exactly-once semantics, replay microbatches
* Latency-throughput tradeoff based on batch sizes
* SparkStreaming and Storm Trident use microbatches

#### Microbatch Windows

* Tumbling window
  * Fixed window size
  * Non-overlapping time
  * Number of entities differ between windows
* Sliding window
  * Fixed window size
  * Overlapping time - sliding interval
  * Number of entities differ between windows
* Session window
  * Changing window size based on session data
  * Non-overlapping time
  * Number of entities differ between windows
  
---