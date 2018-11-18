# Getting started with Impala


## Table of contents:
1. [Chapter 1: Why Impala?](#Chapter1)
2. [Chapter 2: Getting Up and Running with Impala](#Chapter2)
3. [Chapter 3: Impala for the Database Developer](#Chapter3)
4. [Chapter 4: Common Developer Tasks for Impala](#Chapter4)

## Chapter 1: Why Impala?<a name="Chapter1"></a>
Impala is an SQL tool build on top of hadoop that allows:

    * Flexibility for Your Big Data Workflow: Impala integrates with existing Hadoop components,
     security, metadata, storage management, and file formats
    * High-Performance Analytics: The Impala architecture provides a speed boost to SQL queries
    on Hadoop data
    * Exploratory Business Intelligence: Impala can query the raw data files, so you can skip
    the time-consuming stages of loading and reorganizing data previously needed.
    This ise suitable for exploratory BI, which typically involves ad hoc queries which often
    involve aggregation functions like AVG, SUM...

For data you intend to intensively analyze, expect to graduate from text or other unoptimized
file formats, and convert the data to a compressed columnar file format like Parquet.


## Chapter 2: Getting Up and Running with Impala<a name="Chapter2"></a>
### Installation
There are several ways to install Impala:

    * Cloudera Live Demo: Impala Query Editor through the Hue web interface
    * Cloudera QuickStart VM: Single-node VM configuration available in VMWare, KVM, or VirtualBox
    * Cloudera Manager and CDH 5: Installation available in standalone packages or by using the
    Cloudera Manager parcel feature. You need to install the Impala server on each
    DataNode and designate one node (typically the same as the Hadoop NameNode) to also run the
    Impala StateStore daemon
    * Manual installation: This installation procedure must be applied to every DataNode in the
    cluster
    * Building from source: You can also get the Impala source code from GitHub and build it
    yourself

### Connecting to Impala
Impala needs to have some daemons running to server queries: _impalad_ on every datanode, and
_catalogd_ and _statestored_ in the namenode. The Cloudera Manager installation path bundles
these daemons together as an Impala service that you can start and stop as a single unit.
Impala’s query engine is fully decentralized; you can connect to any DataNode.
_impala-shell_ defaults to connecting to _localhost_ (use -i to connect to remote servers).


## Chapter 3: Impala for the Database Developer<a name="Chapter3"></a>
### The SQL Language
Impala supports the following types: *STRING*, *INT*, *TINYINT*, *BIGINT*, *FLOAT*, *DOUBLE*,
*TIMESTAMP*, *BOOLEAN*, *DECIMAL*, *VARCHAR* and *CHAR*. The SQL syntax and DDL is similar
to standard SQL (SQL-92). The *EXPLAIN* statement provides a logical overview of statement
 execution: illustrates how parts of the query are distributed among the nodes in a cluster,
 and how intermediate results are combined at the end to produce the final result set.

#### Limited DML
Impald doesn't have operations such as *UPDATE* or *DELETE*, neither it have indexes,
constraints, or foreign keys. If you have new raw data files, you use *LOAD DATA* to move them
into an Impala table directory. Use an *INSERT SELECT* statement  to copy into another table,
optionally filtering, transforming, and converting in the process. Replace entire tables
or partitions with an *INSERT OVERWRITE* statement.

#### No Transactions
Impala only appends or replaces; it never actually updates existing data. In write operations,
Impala deals with millions of rows through statements such as *LOAD DATA* and *INSERT-SELECT*.
Deletion is done by droping groups of related rows with *DROP TABLE*, *ALTER TABLE* or *DROP
PARTITION*. All Impala nodes in the cluster are notified about newdata from a *LOAD DATA* or
*INSERT* statement, or DDL operations such as *CREATE TABLE* and *DROP TABLE*. When new files
are deposited in an Impala table directory by some non-Impala command, the Impala *REFRESH*
table_name statement has to be used so Impala is notified about the new content.

#### Numbers
Impala included binary-style numeric types: 8-bit, 16-bit, 32-bit, and 64-bit integer types, and
32-bit and 64-bit IEEE-754-style floating-point types. Impala 1.4 added support for the*DECIMAL*
 data type.

#### Recent Additions
Impala 2.0 includes the eagerly awaited analytic functions, such as *ROW_NUMBER()* and *RANK()*
functions, and *OVER* and *PARTITION* clauses, which allows sequencing and aggregation across
moving windows. It also included subquery capabilities. Impala 2.1 introduces a specialized
 variant of *COMPUTE STATS* for fast-growing partitioned tables. The *COMPUTE INCREMENTAL STATS*
statement only processes the data files in newly added partitions. Impala 2.3 added complex type
  support (data types *STRUCT*, *ARRAY*, and *MAP*).

### Big Data Considerations
#### Billions and Billions of Rows
Impala's performance and scalability shine when the data is large enough that you can’t produce,
  manipulate, and analyze it in reasonable time on a single server.

#### HDFS Block Size
Any table below 128MB (HDFS block size) can be represented in a single data block to be
 processed by a single core on a single server with no parallel execution.

#### Parquet Files: The Biggest Blocks of All
Impala writes parquet data files with a default block size of 256 MB, but you can decrease the
 block size before inserting data into a Parquet table.

### How Impala Is Like a Data Warehouse
Impala is optimized for fast bulk read and data load operations like many data warehouse-style
 queries. Impala does this by dividing the work of reading large data files across the nodes
of a cluster. The data is loaded in block in multiple servers and thus ready to be queried.

### Physical and Logical Data Layouts
Views let you impose a different logical arrangement without changing the actual tables and
 columns. When you run queries, how much data is read from disk, how
much memory is required, and how fast do the responses come back depends on physical aspects
 such as file format and partitioning.

#### The HDFS Storage Model
Data stored in Impala is stored in HDFS although it can work also with EMC DSSD storage
 appliance, Amazon S3, and Apache Kudu (incubating).

### Distributed Queries
When an Impala query runs on a Hadoop cluster, Impala breaks down the work into multiple stages
 and automatically sends the appropriate requests to the nodes in the cluster. With Parquet
format, only the columns related with the query are read. This operation is known as projection.

### Normalized and Denormalized Data
Impala can work just fine in either normalized or denormalized data. With the Parquet file
 format, you can use normalized or denormalized data. Parquet uses a column-oriented layout, which
avoids the performance overhead normally associated with wide tables (those with many columns).
 The compression and encoding of Parquet data minimizes storage overhead for repeated values.

### File Formats
#### Text File Format
Text format is the default file format for *CREATE TABLE* commands and also the least efficient
 for serious Big Data applications. The column-oriented layout and compact storage format,
with compression added on top and support for complex types, make Parquet the obvious choice
 when you are dealing with Big-Data-scale volume.

#### Parquet File Format
The Parquet file format, which originated from collaboration between Twitter and Cloudera, is
 optimized for data-warehouse-style queries. Parquet is a binary file format. Numeric values
are represented with consistent sizes, packed into a small number of bytes (either 4 or 8)
 depending on the range of the type. *TIMESTAMP* values are also represented in relatively few
bytes. *BOOLEAN* values are packed into a single bit, rather than the strings true and false as
 in a text table. If the same value is repeated over and over, Parquet uses run-length
encoding to condense that sequence down to two values: the value that’s repeated, and how many
 times it’s repeated. If a column has a modest number of different values, up to 16K,
Parquet uses dictionary encoding for that column: it makes up numeric IDs for the values and
 stores those IDs in the data file along with one copy of the values.

#### Getting File Format Information
The *SHOW TABLE STATS* statement provides the basic information about the file format of the
 table, and each individual partition. The *DESCRIBE FORMATTED* statement dumps a lot of
information about each table, including any delimiter and escape characters specified for text
 tables.

#### Switching File Formats
Impala preserves the flexibility to change a table’s file format at any time: simply replace the
  data with anew set of data files and run an *ALTER TABLE … SET FILEFORMAT* statement.
This can also be done on a per partition basis like in:
`ALTER TABLE t3 PARTITION (state = 'CA',  city = 'Berkeley') SET FILEFORMAT = PARQUET;`

### Aggregation
In the Impala context, aggregation comes up in several different contexts: such as MAX(), SUM(),
or AVG(), combined with GROUP BY clauses; you deal with cluster resources such as memory and
 disk capacity by considering the total (aggregate) capacity, not the capacity of a single
 machine; and for performance reasons, sometimes you aggregate small files into larger ones.


## Chapter 4: Common Developer Tasks for Impala<a name="Chapter4"></a>
