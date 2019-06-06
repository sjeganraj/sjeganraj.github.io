
** Hive Doc**

_Hive Topics_
        --> Hive architecture
        --> Running Hive
        --> Data types and file formats
        --> Hive DDL (Temp table,external table,managed table,partition,compression,bucketing)
        --> Hive DML
        --> HiveQL (Queries) (SELECT,JOIN,SUBQUERY,GROUPBY,ANALYTICS )
        --> INDEXES
        --> Views
        --> Hive schema design
        --> Tuning
        --> File formats and compression
        --> Functions
        --> streaming
        --> Customize Hive files and record formats
        --> Hive Thrift service
        --> storage handlers and NoSQL
        --> Security and locking
        --> Hive integration with oozie
        --> Hive and AWS
        --> hCatalog

_Hive Architecture_
------------------
UI –	The user interface for users to submit queries and other operations to the system. As of 2011 the system had a command line interface and a web based GUI was being developed.

Driver –	The component which receives the queries. This component implements the notion of session handles and provides execute and fetch APIs modeled on JDBC/ODBC interfaces.

Compiler –	The component that parses the query, does semantic analysis on the different query blocks and query expressions and eventually generates an execution plan with the help of
     the table and partition metadata looked up from the metastore.

Metastore –	The component that stores all the structure information of the various tables and partitions in the warehouse including column and column type information, the serializers
   and deserializers necessary to read and write data and the corresponding HDFS files where the data is stored.

Execution Engine –	The component which executes the execution plan created by the compiler. The plan is a DAG of stages. The execution engine manages the dependencies between
      these different stages of the plan and executes these stages on the appropriate system components.


_Hive Data Model_
---------------

Tables – These are analogous to Tables in Relational Databases. Tables can be filtered, projected, joined and unioned. Additionally all the data of a table is stored in a directory in HDFS. Hive also supports the notion of external tables wherein a table can be created on prexisting files or directories in HDFS by providing the appropriate location to the table creation DDL. The rows in a table are organized into typed columns similar to Relational Databases.

Partitions – Each Table can have one or more partition keys which determine how the data is stored, for example a table T with a date partition column ds had files with data for a particular date stored in the <table location>/ds=<date> directory in HDFS. Partitions allow the system to prune data to be inspected based on query predicates, for example a query that is interested in rows from T that satisfy the predicate T.ds = '2008-09-01' would only have to look at files in <table location>/ds=2008-09-01/ directory in HDFS.

Buckets – Data in each partition may in turn be divided into Buckets based on the hash of a column in the table. Each bucket is stored as a file in the partition directory. Bucketing allows the system to efficiently evaluate queries that depend on a sample of data (these are queries that use the SAMPLE clause on the table).

Apart from primitive column types (integers, floating point numbers, generic strings, dates and booleans), Hive also supports arrays and maps. Additionally, users can compose their own types programmatically from any of the primitives, collections or other user-defined types. The typing system is closely tied to the SerDe (Serailization/Deserialization) and object inspector interfaces. User can create their own types by implementing their own  object inspectors, and using these object inspectors they can create their own SerDes to serialize and deserialize their data into HDFS files). These two interfaces provide the necessary hooks to extend the capabilities of
Hive when it comes to understanding other data formats and richer types. Builtin object inspectors like ListObjectInspector, StructObjectInspector and MapObjectInspector provide the necessary primitives to compose richer types in an extensible manner. For maps (associative arrays) and arrays useful builtin functions like size and index operators are provided.
