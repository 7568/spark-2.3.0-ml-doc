- [Overview](#overview)
  - [SQL](#sql)
  - [Datasets and DataFrames](#datasets-and-dataframes)
- [Getting Started](#getting-started)
  - [Starting Point: SparkSession](#starting-point-sparksession)
  - [Creating DataFrames](#creating-dataframes)
  - [Untyped Dataset Operations (aka DataFrame Operations)](#untyped-dataset-operations-aka-dataframe-operations)
  - [Running SQL Queries Programmatically](#running-sql-queries-programmatically)
  - [Global Temporary View](#global-temporary-view)
  - [Creating Datasets](#creating-datasets)
  - [Interoperating with RDDs](#interoperating-with-rdds)
    - [Inferring the Schema Using Reflection](#inferring-the-schema-using-reflection)
    - [Programmatically Specifying the Schema](#programmatically-specifying-the-schema)
  - [Aggregations](#aggregations)
    - [Untyped User-Defined Aggregate Functions](#untyped-user-defined-aggregate-functions)
    - [Type-Safe User-Defined Aggregate Functions](#type-safe-user-defined-aggregate-functions)
- [Data Sources](#data-sources)
  - Generic Load/Save Functions
    - [Manually Specifying Options](#manually-specifying-options)
    - [Run SQL on files directly](#run-sql-on-files-directly)
    - [Save Modes](#save-modes)
    - [Saving to Persistent Tables](#saving-to-persistent-tables)
    - [Bucketing, Sorting and Partitioning](#bucketing-sorting-and-partitioning)
  - [Parquet Files](#parquet-files)
    - [Loading Data Programmatically](#loading-data-programmatically)
    - [Partition Discovery](#partition-discovery)
    - [Schema Merging](#schema-merging)
    - Hive metastore Parquet table conversion
      - [Hive/Parquet Schema Reconciliation](#hiveparquet-schema-reconciliation)
      - [Metadata Refreshing](#metadata-refreshing)
    - [Configuration](#configuration)
  - [ORC Files](#orc-files)
  - [JSON Datasets](#json-datasets)
  - [Hive Tables](#hive-tables)
    - [Specifying storage format for Hive tables](#specifying-storage-format-for-hive-tables)
    - [Interacting with Different Versions of Hive Metastore](#interacting-with-different-versions-of-hive-metastore)
  - [JDBC To Other Databases](#jdbc-to-other-databases)
  - [Troubleshooting](#troubleshooting)
- [Performance Tuning](#performance-tuning)
  - [Caching Data In Memory](#caching-data-in-memory)
  - [Other Configuration Options](#other-configuration-options)
  - [Broadcast Hint for SQL Queries](#broadcast-hint-for-sql-queries)
- [Distributed SQL Engine](#distributed-sql-engine)
  - [Running the Thrift JDBC/ODBC server](#running-the-thrift-jdbcodbc-server)
  - [Running the Spark SQL CLI](#running-the-spark-sql-cli)
- PySpark Usage Guide for Pandas with Apache Arrow
  - [Apache Arrow in Spark](#apache-arrow-in -spark)
    - [Ensure PyArrow Installed](#ensure-pyarrow-installed)
  - [Enabling for Conversion to/from Pandas](#enabling-for-conversion-tofrom-pandas)
  - Pandas UDFs (a.k.a. Vectorized UDFs)
    - [Scalar](#scalar)
    - [Grouped Map](#grouped-map)
  - Usage Notes
    - [Supported SQL Types](#supported-sql-types)
    - [Setting Arrow Batch Size](#setting-arrow-batch-size)
    - [Timestamp with Time Zone Semantics](#timestamp-with-time-zone-semantics)
- Migration Guide
  - [Upgrading From Spark SQL 2.2 to 2.3](#upgrading-from-spark-sql-22-to-23)
  - [Upgrading From Spark SQL 2.1 to 2.2](#upgrading-from-spark-sql-21-to-22)
  - [Upgrading From Spark SQL 2.0 to 2.1](#upgrading-from-spark-sql-20-to-21)
  - [Upgrading From Spark SQL 1.6 to 2.0](#upgrading-from-spark-sql-16-to-20)
  - [Upgrading From Spark SQL 1.5 to 1.6](#upgrading-from-spark-sql-15-to-16)
  - [Upgrading From Spark SQL 1.4 to 1.5](#upgrading-from-spark-sql-14-to-15)
  - Upgrading from Spark SQL 1.3 to 1.4
    - [DataFrame data reader/writer interface](#dataframe-data-readerwriter-interface)
    - [DataFrame.groupBy retains grouping columns](#dataframegroupby-retains-grouping-columns)
    - [Behavior change on DataFrame.withColumn](#behavior-change-on-dataframewithcolumn)
  - Upgrading from Spark SQL 1.0-1.2 to 1.3
    - [Rename of SchemaRDD to DataFrame](#rename-of-schemardd-to-dataframe)
    - [Unification of the Java and Scala APIs](#unification-of-the-java-and-scala-apis)
    - [Isolation of Implicit Conversions and Removal of dsl Package (Scala-only)](#isolation-of-implicit-conversions-and-removal-of-dsl-package-scala-only)
    - [Removal of the type aliases in org.apache.spark.sql for DataType (Scala-only)](#removal-of-the-type-aliases-in-orgapachesparksql-for-datatype-scala-only)
    - [UDF Registration Moved to `sqlContext.udf` (Java & Scala)](#udf-registration-moved-to-sqlcontextudf-java--scala)
    - [Python DataTypes No Longer Singletons](#python-datatypes-no-longer-singletons)
  - Compatibility with Apache Hive
    - [Deploying in Existing Hive Warehouses](#deploying-in-existing-hive-warehouses)
    - [Supported Hive Features](#supported-hive-features)
    - [Unsupported Hive Functionality](#unsupported-hive-functionality)
    - [Incompatible Hive UDF](#incompatible-hive-udf)
- Reference
  - [Data Types](#data-types)
  - [NaN Semantics](#nan-semantics)

# Overview

Spark SQL is a Spark module for structured data processing. Unlike the basic Spark RDD API, the interfaces provided by Spark SQL provide Spark with more information about the structure of both the data and the computation being performed. Internally, Spark SQL uses this extra information to perform extra optimizations. There are several ways to interact with Spark SQL including SQL and the Dataset API. When computing a result the same execution engine is used, independent of which API/language you are using to express the computation. This unification means that developers can easily switch back and forth between different APIs based on which provides the most natural way to express a given transformation.

All of the examples on this page use sample data included in the Spark distribution and can be run in the `spark-shell`, `pyspark` shell, or `sparkR`shell.

Spark SQL是 Spark中的一个处理结构性数据的模块，不像基础的Spark RDD API那样, 这个借口通过 Spark SQL 提供给 Spark 更多的关于数据和执行上的结构化的信息。在内部，Spark SQL 通过这个外部的信息来优化它。这里有SQL和 Dataset API的方式来与Spark SQL 交互。Spark使用同一个执行引擎计算一个结果，这个引擎取决于你使用的是哪种计算表达式。这种同一的的方式意味着开发者能够轻易的进行不同API的前后切换，基于这个API提供了最原始的方式来表达一个给定的转换

在这个页面上的所有的例子都包含到了 Spark 分布式中，都能够通过 `spark-shell`, `pyspark` shell, or `sparkR`shell来执行。

## SQL

One use of Spark SQL is to execute SQL queries. Spark SQL can also be used to read data from an existing Hive installation. For more on how to configure this feature, please refer to the [Hive Tables](https://spark.apache.org/docs/latest/sql-programming-guide.html#hive-tables) section. When running SQL from within another programming language the results will be returned as a [Dataset/DataFrame](https://spark.apache.org/docs/latest/sql-programming-guide.html#datasets-and-dataframes). You can also interact with the SQL interface using the [command-line](https://spark.apache.org/docs/latest/sql-programming-guide.html#running-the-spark-sql-cli) or over [JDBC/ODBC](https://spark.apache.org/docs/latest/sql-programming-guide.html#running-the-thrift-jdbcodbc-server).

Spark SQL的一个功能是执行 SQL 查询语句。 Spark SQL 也能够用来读取hive中的数据。更多关于如何配置此功能请参考 [Hive Tables](https://spark.apache.org/docs/latest/sql-programming-guide.html#hive-tables) 章节. 当执行从其他程序中返回的 SQL时，它的结果的类型是 [Dataset/DataFrame](https://spark.apache.org/docs/latest/sql-programming-guide.html#datasets-and-dataframes)你也可以通过 [command-line](https://spark.apache.org/docs/latest/sql-programming-guide.html#running-the-spark-sql-cli) 或者是 [JDBC/ODBC](https://spark.apache.org/docs/latest/sql-programming-guide.html#running-the-thrift-jdbcodbc-server)来与SQL交互。

## Datasets and DataFrames

A Dataset is a distributed collection of data. Dataset is a new interface added in Spark 1.6 that provides the benefits of RDDs (strong typing, ability to use powerful lambda functions) with the benefits of Spark SQL’s optimized execution engine. A Dataset can be [constructed](https://spark.apache.org/docs/latest/sql-programming-guide.html#creating-datasets) from JVM objects and then manipulated using functional transformations (`map`, `flatMap`, `filter`, etc.). The Dataset API is available in [Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Dataset) and [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/sql/Dataset.html). Python does not have the support for the Dataset API. But due to Python’s dynamic nature, many of the benefits of the Dataset API are already available (i.e. you can access the field of a row by name naturally `row.columnName`). The case for R is similar.

A DataFrame is a *Dataset* organized into named columns. It is conceptually equivalent to a table in a relational database or a data frame in R/Python, but with richer optimizations under the hood. DataFrames can be constructed from a wide array of [sources](https://spark.apache.org/docs/latest/sql-programming-guide.html#data-sources) such as: structured data files, tables in Hive, external databases, or existing RDDs. The DataFrame API is available in Scala, Java, [Python](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrame), and [R](https://spark.apache.org/docs/latest/api/R/index.html). In Scala and Java, a DataFrame is represented by a Dataset of `Row`s. In [the Scala API](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Dataset), `DataFrame` is simply a type alias of `Dataset[Row]`. While, in [Java API](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/sql/Dataset.html), users need to use `Dataset<Row>` to represent a `DataFrame`.

Throughout this document, we will often refer to Scala/Java Datasets of `Row`s as DataFrames.

Dataset 指的是一个分布式的结果集数据。Dataset 是 Spark 1.6 之后新添加进来的，它提供了Spark SQL执行引擎的优化和RDD的好处。RDD是一种强类型，能够使用lambda强大功能的数据结构。一个Dataset 能够从JVM的对象中结构化出来，然后通过功能性数据转换来操作他 (`map`, `flatMap`, `filter`, etc.).。 Dataset 的API 有 [Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Dataset) 的和 [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/sql/Dataset.html).的。Python 不支持 Dataset API。但是因为 Python的动态特性, 很多关于 Dataset API 的好处早就已经存在了(i.e. you can access the field of a row by name naturally `row.columnName`).。这种情况在R语言中是一样的。

 DataFrame是已经组织化到命了名的列中的一个*Dataset* 。它的概念就像关系型数据库中表或者是 R/Python中的数据框架，但是它在hood下更加的优越 。 DataFrames 能够被从宽泛的数据源中结构化出来。比如结构化了的文件数据， Hive中的表，外部的数据库，或则是已经存在的 RDDs。 DataFrame API 有 Scala, Java, [Python](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrame), 和 [R](https://spark.apache.org/docs/latest/api/R/index.html). 在 Scala and Java, 一个 DataFrame 被 Dataset 中的行表示。 In [the Scala API](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Dataset)中`DataFrame` 是一个简单的Dataset[Row]的类型别名.然而，在 [Java API](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/sql/Dataset.html), 用户必须使用 `Dataset<Row>` 来代表 `DataFrame`.

在整个这份文档中，我们将会使用 Scala/Java Datasets 中的 `Row代表 DataFrames.

# Getting Started

## Starting Point: SparkSession

The entry point into all functionality in Spark is the [`SparkSession`](https://spark.apache.org/docs/latest/api/java/index.html#org.apache.spark.sql.SparkSession) class. To create a basic `SparkSession`, just use `SparkSession.builder()`:

在 Spark 中入口点是 [`SparkSession`](https://spark.apache.org/docs/latest/api/java/index.html#org.apache.spark.sql.SparkSession) 这个类，创建一个基本的SparkSession`， 只需要 `SparkSession.builder()`:

```java
import org.apache.spark.sql.SparkSession;

SparkSession spark = SparkSession
  .builder()
  .appName("Java Spark SQL basic example")
  .config("spark.some.config.option", "some-value")
  .getOrCreate();
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSparkSQLExample.java" in the Spark repo.

`SparkSession` in Spark 2.0 provides builtin support for Hive features including the ability to write queries using HiveQL, access to Hive UDFs, and the ability to read data from Hive tables. To use these features, you do not need to have an existing Hive setup.

`SparkSession` 在 Spark 2.0 中内置提供，为了支持 Hive 的功能，包括 使用 HiveQL来写查询语句，访问 Hive UDFs，和从 Hive 表中读取数据。使用这些功能，你不必进行Hive的设置。

## Creating DataFrames

With a `SparkSession`, applications can create DataFrames from an [existing `RDD`](https://spark.apache.org/docs/latest/sql-programming-guide.html#interoperating-with-rdds), from a Hive table, or from [Spark data sources](https://spark.apache.org/docs/latest/sql-programming-guide.html#data-sources).

As an example, the following creates a DataFrame based on the content of a JSON file:

通过 `SparkSession`，程序能够 从 [existing `RDD`](https://spark.apache.org/docs/latest/sql-programming-guide.html#interoperating-with-rdds)中创建一个 DataFrames,或者是一个Hive 表, 或者是通过 [Spark data sources](https://spark.apache.org/docs/latest/sql-programming-guide.html#data-sources).

下面这个例子中，通过基于JSON文件格式的文件来创建一个DataFrame ：

```java
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;

Dataset<Row> df = spark.read().json("examples/src/main/resources/people.json");

// Displays the content of the DataFrame to stdout
df.show();
// +----+-------+
// | age|   name|
// +----+-------+
// |null|Michael|
// |  30|   Andy|
// |  19| Justin|
// +----+-------+
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSparkSQLExample.java" in the Spark repo.

## Untyped Dataset Operations (aka DataFrame Operations)

不指定Dataset的操作

DataFrames provide a domain-specific language for structured data manipulation in [Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Dataset), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/sql/Dataset.html), [Python](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrame) and [R](https://spark.apache.org/docs/latest/api/R/SparkDataFrame.html).

As mentioned above, in Spark 2.0, DataFrames are just Dataset of `Row`s in Scala and Java API. These operations are also referred as “untyped transformations” in contrast to “typed transformations” come with strongly typed Scala/Java Datasets.

Here we include some basic examples of structured data processing using Datasets:

DataFrames 提供一个对结构化的数据在 [Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Dataset), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/sql/Dataset.html), [Python](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrame) and [R](https://spark.apache.org/docs/latest/api/R/SparkDataFrame.html)中操作的特殊域语言 

正如上面提到的在 Spark 2.0中 DataFrames 仅仅是 Scala and Java API中 Dataset 中的行数据，这些操作也涉及到与类型化数据转换相反的无类型数据转换从而来操作强类型的 Scala/Java 中的Datasets。

下面的例子中包含了一些我们常用的关于结构化数据的对Datasets的一些操作：

```java
// col("...") is preferable to df.col("...")
import static org.apache.spark.sql.functions.col;

// Print the schema in a tree format
df.printSchema();
// root
// |-- age: long (nullable = true)
// |-- name: string (nullable = true)

// Select only the "name" column
df.select("name").show();
// +-------+
// |   name|
// +-------+
// |Michael|
// |   Andy|
// | Justin|
// +-------+

// Select everybody, but increment the age by 1
df.select(col("name"), col("age").plus(1)).show();
// +-------+---------+
// |   name|(age + 1)|
// +-------+---------+
// |Michael|     null|
// |   Andy|       31|
// | Justin|       20|
// +-------+---------+

// Select people older than 21
df.filter(col("age").gt(21)).show();
// +---+----+
// |age|name|
// +---+----+
// | 30|Andy|
// +---+----+

// Count people by age
df.groupBy("age").count().show();
// +----+-----+
// | age|count|
// +----+-----+
// |  19|    1|
// |null|    1|
// |  30|    1|
// +----+-----+
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSparkSQLExample.java" in the Spark repo.

For a complete list of the types of operations that can be performed on a Dataset refer to the [API Documentation](https://spark.apache.org/docs/latest/api/java/org/apache/spark/sql/Dataset.html).

In addition to simple column references and expressions, Datasets also have a rich library of functions including string manipulation, date arithmetic, common math operations and more. The complete list is available in the [DataFrame Function Reference](https://spark.apache.org/docs/latest/api/java/org/apache/spark/sql/functions.html).

全部Dataset的操作详情请参考 [API Documentation](https://spark.apache.org/docs/latest/api/java/org/apache/spark/sql/Dataset.html).

此外，对简单的列的引用和表达式， Datasets 也拥有非常丰富的库，这些库中包含有对string, date 转换, 一般math 等等的操作，完整的信息请参考这里 [DataFrame Function Reference](https://spark.apache.org/docs/latest/api/java/org/apache/spark/sql/functions.html).

## Running SQL Queries Programmatically

The `sql` function on a `SparkSession` enables applications to run SQL queries programmatically and returns the result as a `Dataset<Row>`.

在 `SparkSession` 中的`sql`能执行SQL表达式然后返回 `Dataset<Row>`类型的结果

```java
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;

// Register the DataFrame as a SQL temporary view
df.createOrReplaceTempView("people");

Dataset<Row> sqlDF = spark.sql("SELECT * FROM people");
sqlDF.show();
// +----+-------+
// | age|   name|
// +----+-------+
// |null|Michael|
// |  30|   Andy|
// |  19| Justin|
// +----+-------+
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSparkSQLExample.java" in the Spark repo.

## Global Temporary View

Temporary views in Spark SQL are session-scoped and will disappear if the session that creates it terminates. If you want to have a temporary view that is shared among all sessions and keep alive until the Spark application terminates, you can create a global temporary view. Global temporary view is tied to a system preserved database `global_temp`, and we must use the qualified name to refer it, e.g. `SELECT * FROM global_temp.view1`.

在 Spark SQL 中的临时视图只存在于当前视图中，session结束，它也消失，如果你想让你的零时视图在Spark中的多个session中共享你可以创建一个全球化的视图。全球化的视图是被绑定到原有的数据库中`global_temp`，我们必须使用合格的名字才能使用它，例如 `SELECT * FROM global_temp.view1`。

```java

// Register the DataFrame as a global temporary view
df.createGlobalTempView("people");

// Global temporary view is tied to a system preserved database `global_temp`
spark.sql("SELECT * FROM global_temp.people").show();
// +----+-------+
// | age|   name|
// +----+-------+
// |null|Michael|
// |  30|   Andy|
// |  19| Justin|
// +----+-------+

// Global temporary view is cross-session
spark.newSession().sql("SELECT * FROM global_temp.people").show();
// +----+-------+
// | age|   name|
// +----+-------+
// |null|Michael|
// |  30|   Andy|
// |  19| Justin|
// +----+-------+
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSparkSQLExample.java" in the Spark repo.

## Creating Datasets

Datasets are similar to RDDs, however, instead of using Java serialization or Kryo they use a specialized [Encoder](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Encoder) to serialize the objects for processing or transmitting over the network. While both encoders and standard serialization are responsible for turning an object into bytes, encoders are code generated dynamically and use a format that allows Spark to perform many operations like filtering, sorting and hashing without deserializing the bytes back into an object.

Datasets 很像 RDDs, 但是, 不像java的序列化或者 Kryo ，他们使用特殊的编码( [Encoder](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Encoder) ）来实例化对象用来在网络中执行或者传输，然而，所有的编码和标准的序列化都是要将对象转换成byte，编码是动态的创建代码，使用一个格式来允许Spark执行很多操作，比如过滤，排序，或者是hash运算，都不需要将bytes反实例化成对象。

```java
import java.util.Arrays;
import java.util.Collections;
import java.io.Serializable;

import org.apache.spark.api.java.function.MapFunction;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.Encoder;
import org.apache.spark.sql.Encoders;

public static class Person implements Serializable {
  private String name;
  private int age;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public int getAge() {
    return age;
  }

  public void setAge(int age) {
    this.age = age;
  }
}

// Create an instance of a Bean class
Person person = new Person();
person.setName("Andy");
person.setAge(32);

// Encoders are created for Java beans
Encoder<Person> personEncoder = Encoders.bean(Person.class);
Dataset<Person> javaBeanDS = spark.createDataset(
  Collections.singletonList(person),
  personEncoder
);
javaBeanDS.show();
// +---+----+
// |age|name|
// +---+----+
// | 32|Andy|
// +---+----+

// Encoders for most common types are provided in class Encoders
Encoder<Integer> integerEncoder = Encoders.INT();
Dataset<Integer> primitiveDS = spark.createDataset(Arrays.asList(1, 2, 3), integerEncoder);
Dataset<Integer> transformedDS = primitiveDS.map(
    (MapFunction<Integer, Integer>) value -> value + 1,
    integerEncoder);
transformedDS.collect(); // Returns [2, 3, 4]

// DataFrames can be converted to a Dataset by providing a class. Mapping based on name
String path = "examples/src/main/resources/people.json";
Dataset<Person> peopleDS = spark.read().json(path).as(personEncoder);
peopleDS.show();
// +----+-------+
// | age|   name|
// +----+-------+
// |null|Michael|
// |  30|   Andy|
// |  19| Justin|
// +----+-------+
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSparkSQLExample.java" in the Spark repo.

## Interoperating with RDDs

Spark SQL supports two different methods for converting existing RDDs into Datasets. The first method uses reflection to infer the schema of an RDD that contains specific types of objects. This reflection based approach leads to more concise code and works well when you already know the schema while writing your Spark application.

The second method for creating Datasets is through a programmatic interface that allows you to construct a schema and then apply it to an existing RDD. While this method is more verbose, it allows you to construct Datasets when the columns and their types are not known until runtime.

Spark SQL 支持两种不同的方法来将 RDDs 转换成 Datasets。第一个方法是使用反射来将得到包含特殊类型对象的RDD的模式。当你在编写你的spark软件时，在你已经了解了模式之后，这种反射的方式会使代码更简洁，软件运行更好。

第二个创建Datasets的方法是通过程序接口，这种方法允许你组建一个模板，然后将它应用到一个已经存在的RDD中。然而这种方法更加的灵活，它能够让你在不知道他们运行前的类型前提下构造Dataset。

### Inferring the Schema Using Reflection

Spark SQL supports automatically converting an RDD of [JavaBeans](http://stackoverflow.com/questions/3295496/what-is-a-javabean-exactly) into a DataFrame. The `BeanInfo`, obtained using reflection, defines the schema of the table. Currently, Spark SQL does not support JavaBeans that contain `Map` field(s). Nested JavaBeans and `List` or `Array` fields are supported though. You can create a JavaBean by creating a class that implements Serializable and has getters and setters for all of its fields.

Spark SQL 支持动态的将 [JavaBeans](http://stackoverflow.com/questions/3295496/what-is-a-javabean-exactly) 类型的RDD转换成 DataFrame。 `BeanInfo`, 包含了使用反射来定义标的模板。 最近， Spark SQL 不再支持 JavaBeans 中包含 `Map` 类型的字段了。  嵌套的JavaBeans 和 `List` 或者 `Array` 类型的字段依然支持。你可以通过创建一个 能实例化且所有的属性字段都有get，set方法的类来创建一个JAVABean。

```java
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.MapFunction;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.Encoder;
import org.apache.spark.sql.Encoders;

// Create an RDD of Person objects from a text file
JavaRDD<Person> peopleRDD = spark.read()
  .textFile("examples/src/main/resources/people.txt")
  .javaRDD()
  .map(line -> {
    String[] parts = line.split(",");
    Person person = new Person();
    person.setName(parts[0]);
    person.setAge(Integer.parseInt(parts[1].trim()));
    return person;
  });

// Apply a schema to an RDD of JavaBeans to get a DataFrame
Dataset<Row> peopleDF = spark.createDataFrame(peopleRDD, Person.class);
// Register the DataFrame as a temporary view
peopleDF.createOrReplaceTempView("people");

// SQL statements can be run by using the sql methods provided by spark
Dataset<Row> teenagersDF = spark.sql("SELECT name FROM people WHERE age BETWEEN 13 AND 19");

// The columns of a row in the result can be accessed by field index
Encoder<String> stringEncoder = Encoders.STRING();
Dataset<String> teenagerNamesByIndexDF = teenagersDF.map(
    (MapFunction<Row, String>) row -> "Name: " + row.getString(0),
    stringEncoder);
teenagerNamesByIndexDF.show();
// +------------+
// |       value|
// +------------+
// |Name: Justin|
// +------------+

// or by field name
Dataset<String> teenagerNamesByFieldDF = teenagersDF.map(
    (MapFunction<Row, String>) row -> "Name: " + row.<String>getAs("name"),
    stringEncoder);
teenagerNamesByFieldDF.show();
// +------------+
// |       value|
// +------------+
// |Name: Justin|
// +------------+
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSparkSQLExample.java" in the Spark repo.

### Programmatically Specifying the Schema

When JavaBean classes cannot be defined ahead of time (for example, the structure of records is encoded in a string, or a text dataset will be parsed and fields will be projected differently for different users), a `Dataset<Row>` can be created programmatically with three steps.

1. Create an RDD of `Row`s from the original RDD;
2. Create the schema represented by a `StructType` matching the structure of `Row`s in the RDD created in Step 1.
3. Apply the schema to the RDD of `Row`s via `createDataFrame` method provided by `SparkSession`.

For example:

当 JavaBean 类之前没有被定义（例如，记录的结构被编码成一个字符串，或者被解析成text类型的dataset，而且它的字段被映射到不同的用户中），一个 Dataset<Row>能够通过以下三个方法被创建。

1. 根据原始的RDD，创建一个Row类型的 RDD;
2. 创建一个被第一步中的RDD的Row的结构匹配的`StructType`来代表的schema。
3. 通过使用 `SparkSession`中提供的`createDataFrame` 方法来将这个 schema 应用到 RDD 的 `Row`

例如:

```java
import java.util.ArrayList;
import java.util.List;

import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.function.Function;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;

import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

// Create an RDD
JavaRDD<String> peopleRDD = spark.sparkContext()
  .textFile("examples/src/main/resources/people.txt", 1)
  .toJavaRDD();

// The schema is encoded in a string
String schemaString = "name age";

// Generate the schema based on the string of schema
List<StructField> fields = new ArrayList<>();
for (String fieldName : schemaString.split(" ")) {
  StructField field = DataTypes.createStructField(fieldName, DataTypes.StringType, true);
  fields.add(field);
}
StructType schema = DataTypes.createStructType(fields);

// Convert records of the RDD (people) to Rows
JavaRDD<Row> rowRDD = peopleRDD.map((Function<String, Row>) record -> {
  String[] attributes = record.split(",");
  return RowFactory.create(attributes[0], attributes[1].trim());
});

// Apply the schema to the RDD
Dataset<Row> peopleDataFrame = spark.createDataFrame(rowRDD, schema);

// Creates a temporary view using the DataFrame
peopleDataFrame.createOrReplaceTempView("people");

// SQL can be run over a temporary view created using DataFrames
Dataset<Row> results = spark.sql("SELECT name FROM people");

// The results of SQL queries are DataFrames and support all the normal RDD operations
// The columns of a row in the result can be accessed by field index or by field name
Dataset<String> namesDS = results.map(
    (MapFunction<Row, String>) row -> "Name: " + row.getString(0),
    Encoders.STRING());
namesDS.show();
// +-------------+
// |        value|
// +-------------+
// |Name: Michael|
// |   Name: Andy|
// | Name: Justin|
// +-------------+
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSparkSQLExample.java" in the Spark repo.

## Aggregations

The [built-in DataFrames functions](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$) provide common aggregations such as `count()`, `countDistinct()`, `avg()`, `max()`, `min()`, etc. While those functions are designed for DataFrames, Spark SQL also has type-safe versions for some of them in [Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.expressions.scalalang.typed$) and [Java](https://spark.apache.org/docs/latest/api/java/org/apache/spark/sql/expressions/javalang/typed.html) to work with strongly typed Datasets. Moreover, users are not limited to the predefined aggregate functions and can create their own.

### Untyped User-Defined Aggregate Functions

Users have to extend the [UserDefinedAggregateFunction](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.expressions.UserDefinedAggregateFunction) abstract class to implement a custom untyped aggregate function. For example, a user-defined average can look like:

 [built-in DataFrames functions](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$) 提供常规的聚合方法例如 `count()`, `countDistinct()`, `avg()`, `max()`, `min()`, 等等. 这些方法都是为 DataFrames设计的, Spark SQL 也有在 [Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.expressions.scalalang.typed$) 和 [Java](https://spark.apache.org/docs/latest/api/java/org/apache/spark/sql/expressions/javalang/typed.html) 中提供类型安全的版本。他们工作的时候都是强类型的。而且，没有限制用户使用自己的聚合方法。

### Untyped User-Defined Aggregate Functions

用户必须继承于抽象类 [UserDefinedAggregateFunction](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.expressions.UserDefinedAggregateFunction) ，然后实现它的常规的无类型的聚合方法，例如一般用户自定义成这样：



```java
import java.util.ArrayList;
import java.util.List;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.expressions.MutableAggregationBuffer;
import org.apache.spark.sql.expressions.UserDefinedAggregateFunction;
import org.apache.spark.sql.types.DataType;
import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

public static class MyAverage extends UserDefinedAggregateFunction {

  private StructType inputSchema;
  private StructType bufferSchema;

  public MyAverage() {
    List<StructField> inputFields = new ArrayList<>();
    inputFields.add(DataTypes.createStructField("inputColumn", DataTypes.LongType, true));
    inputSchema = DataTypes.createStructType(inputFields);

    List<StructField> bufferFields = new ArrayList<>();
    bufferFields.add(DataTypes.createStructField("sum", DataTypes.LongType, true));
    bufferFields.add(DataTypes.createStructField("count", DataTypes.LongType, true));
    bufferSchema = DataTypes.createStructType(bufferFields);
  }
  // Data types of input arguments of this aggregate function
  public StructType inputSchema() {
    return inputSchema;
  }
  // Data types of values in the aggregation buffer
  public StructType bufferSchema() {
    return bufferSchema;
  }
  // The data type of the returned value
  public DataType dataType() {
    return DataTypes.DoubleType;
  }
  // Whether this function always returns the same output on the identical input
  public boolean deterministic() {
    return true;
  }
  // Initializes the given aggregation buffer. The buffer itself is a `Row` that in addition to
  // standard methods like retrieving a value at an index (e.g., get(), getBoolean()), provides
  // the opportunity to update its values. Note that arrays and maps inside the buffer are still
  // immutable.
  public void initialize(MutableAggregationBuffer buffer) {
    buffer.update(0, 0L);
    buffer.update(1, 0L);
  }
  // Updates the given aggregation buffer `buffer` with new input data from `input`
  public void update(MutableAggregationBuffer buffer, Row input) {
    if (!input.isNullAt(0)) {
      long updatedSum = buffer.getLong(0) + input.getLong(0);
      long updatedCount = buffer.getLong(1) + 1;
      buffer.update(0, updatedSum);
      buffer.update(1, updatedCount);
    }
  }
  // Merges two aggregation buffers and stores the updated buffer values back to `buffer1`
  public void merge(MutableAggregationBuffer buffer1, Row buffer2) {
    long mergedSum = buffer1.getLong(0) + buffer2.getLong(0);
    long mergedCount = buffer1.getLong(1) + buffer2.getLong(1);
    buffer1.update(0, mergedSum);
    buffer1.update(1, mergedCount);
  }
  // Calculates the final result
  public Double evaluate(Row buffer) {
    return ((double) buffer.getLong(0)) / buffer.getLong(1);
  }
}

// Register the function to access it
spark.udf().register("myAverage", new MyAverage());

Dataset<Row> df = spark.read().json("examples/src/main/resources/employees.json");
df.createOrReplaceTempView("employees");
df.show();
// +-------+------+
// |   name|salary|
// +-------+------+
// |Michael|  3000|
// |   Andy|  4500|
// | Justin|  3500|
// |  Berta|  4000|
// +-------+------+

Dataset<Row> result = spark.sql("SELECT myAverage(salary) as average_salary FROM employees");
result.show();
// +--------------+
// |average_salary|
// +--------------+
// |        3750.0|
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaUserDefinedUntypedAggregation.java" in the Spark repo.

### Type-Safe User-Defined Aggregate Functions

User-defined aggregations for strongly typed Datasets revolve around the [Aggregator](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.expressions.Aggregator) abstract class. For example, a type-safe user-defined average can look like:

用户自定义的强类型的Dataset是围绕责抽象类 [Aggregator](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.expressions.Aggregator) 来的。例如一个类型安全的用户自定义一般是这样的：



```java
import java.io.Serializable;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Encoder;
import org.apache.spark.sql.Encoders;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.TypedColumn;
import org.apache.spark.sql.expressions.Aggregator;

public static class Employee implements Serializable {
  private String name;
  private long salary;

  // Constructors, getters, setters...

}

public static class Average implements Serializable  {
  private long sum;
  private long count;

  // Constructors, getters, setters...

}

public static class MyAverage extends Aggregator<Employee, Average, Double> {
  // A zero value for this aggregation. Should satisfy the property that any b + zero = b
  public Average zero() {
    return new Average(0L, 0L);
  }
  // Combine two values to produce a new value. For performance, the function may modify `buffer`
  // and return it instead of constructing a new object
  public Average reduce(Average buffer, Employee employee) {
    long newSum = buffer.getSum() + employee.getSalary();
    long newCount = buffer.getCount() + 1;
    buffer.setSum(newSum);
    buffer.setCount(newCount);
    return buffer;
  }
  // Merge two intermediate values
  public Average merge(Average b1, Average b2) {
    long mergedSum = b1.getSum() + b2.getSum();
    long mergedCount = b1.getCount() + b2.getCount();
    b1.setSum(mergedSum);
    b1.setCount(mergedCount);
    return b1;
  }
  // Transform the output of the reduction
  public Double finish(Average reduction) {
    return ((double) reduction.getSum()) / reduction.getCount();
  }
  // Specifies the Encoder for the intermediate value type
  public Encoder<Average> bufferEncoder() {
    return Encoders.bean(Average.class);
  }
  // Specifies the Encoder for the final output value type
  public Encoder<Double> outputEncoder() {
    return Encoders.DOUBLE();
  }
}

Encoder<Employee> employeeEncoder = Encoders.bean(Employee.class);
String path = "examples/src/main/resources/employees.json";
Dataset<Employee> ds = spark.read().json(path).as(employeeEncoder);
ds.show();
// +-------+------+
// |   name|salary|
// +-------+------+
// |Michael|  3000|
// |   Andy|  4500|
// | Justin|  3500|
// |  Berta|  4000|
// +-------+------+

MyAverage myAverage = new MyAverage();
// Convert the function to a `TypedColumn` and give it a name
TypedColumn<Employee, Double> averageSalary = myAverage.toColumn().name("average_salary");
Dataset<Double> result = ds.select(averageSalary);
result.show();
// +--------------+
// |average_salary|
// +--------------+
// |        3750.0|
// +--------------+
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaUserDefinedTypedAggregation.java" in the Spark repo.

# Data Sources

Spark SQL supports operating on a variety of data sources through the DataFrame interface. A DataFrame can be operated on using relational transformations and can also be used to create a temporary view. Registering a DataFrame as a temporary view allows you to run SQL queries over its data. This section describes the general methods for loading and saving data using the Spark Data Sources and then goes into specific options that are available for the built-in data sources.

Spark SQL 支持通过DataFrame接口来操作不同类型的数据源的数据。一个 DataFrame 能够被相关的转换来操作，也能通过创建一个临时的视图来操作。将一个 DataFrame注册成一个临时的视图能够使你基于它的数据进行SQL查询。这部分描述了使用Spark Data Sources中一般方法来加载和保存数据，然后使用一些特殊的可选项来构架一个数据源。

## Generic Load/Save Functions

In the simplest form, the default data source (`parquet` unless otherwise configured by `spark.sql.sources.default`) will be used for all operations.

在最简单的表中默认的数据源能被所有的操作使用 (`parquet` 类型的数据除非被 `spark.sql.sources.default`另外配置过) 。

```java

Dataset<Row> usersDF = spark.read().load("examples/src/main/resources/users.parquet");
usersDF.select("name", "favorite_color").write().save("namesAndFavColors.parquet");
```

### Manually Specifying Options

You can also manually specify the data source that will be used along with any extra options that you would like to pass to the data source. Data sources are specified by their fully qualified name (i.e., `org.apache.spark.sql.parquet`), but for built-in sources you can also use their short names (`json`, `parquet`, `jdbc`, `orc`, `libsvm`, `csv`, `text`). DataFrames loaded from any data source type can be converted into other types using this syntax.

To load a JSON file you can use:

### Manually Specifying Options

你也可以手动的操作指定的数据源，这需要你提供额外的可选参数值。数据源使用他们的完整的名字来指定（比如org.apache.spark.sql.parquet），但是在构造数据源的时候你也可以使用名字的缩略形式（`json`, `parquet`, `jdbc`, `orc`, `libsvm`, `csv`, `text`）。从任何地方加载的数据都会被这种句法转换成指定的类型。

你可以这样加载一个json文件：

```java
Dataset<Row> peopleDF =
  spark.read().format("json").load("examples/src/main/resources/people.json");
peopleDF.select("name", "age").write().format("parquet").save("namesAndAges.parquet");
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSQLDataSourceExample.java" in the Spark repo.

To load a CSV file you can use:

你可以这样加载一个CSV文件：

```java
Dataset<Row> peopleDFCsv = spark.read().format("csv")
  .option("sep", ";")
  .option("inferSchema", "true")
  .option("header", "true")
  .load("examples/src/main/resources/people.csv");
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSQLDataSourceExample.java" in the Spark repo.

### Run SQL on files directly

Instead of using read API to load a file into DataFrame and query it, you can also query that file directly with SQL.

可以个直接通过SQL查询语句来代替使用API来加载一个文件到DataFrame中和查询它。

```java
Dataset<Row> sqlDF =
  spark.sql("SELECT * FROM parquet.`examples/src/main/resources/users.parquet`");
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSQLDataSourceExample.java" in the Spark repo.

### Save Modes

Save operations can optionally take a `SaveMode`, that specifies how to handle existing data if present. It is important to realize that these save modes do not utilize any locking and are not atomic. Additionally, when performing an `Overwrite`, the data will be deleted before writing out the new data.

保存操作能够使用可选的 `SaveMode`。它能够将如何处理一个已经存在的数据特殊化。你要知道保存modes不会使用任何锁，也没有原子化操作，这个是很重要的。另外，当你使用`Overwrite`的时候，原始数据会在写入新数据之前被删掉。



| Scala/Java                        | Any Language                          | Meaning                                                      |
| --------------------------------- | ------------------------------------- | ------------------------------------------------------------ |
| `SaveMode.ErrorIfExists`(default) | `"error" or "errorifexists"`(default) | When saving a DataFrame to a data source, if data already exists, an exception is expected to be thrown. |
| `SaveMode.Append`                 | `"append"`                            | When saving a DataFrame to a data source, if data/table already exists, contents of the DataFrame are expected to be appended to existing data. |
| `SaveMode.Overwrite`              | `"overwrite"`                         | Overwrite mode means that when saving a DataFrame to a data source, if data/table already exists, existing data is expected to be overwritten by the contents of the DataFrame. |
| `SaveMode.Ignore`                 | `"ignore"`                            | Ignore mode means that when saving a DataFrame to a data source, if data already exists, the save operation is expected to not save the contents of the DataFrame and to not change the existing data. This is similar to a `CREATE TABLE IF NOT EXISTS` in SQL. |

| Scala/Java                        | Any Language                          | Meaning                                                      |
| --------------------------------- | ------------------------------------- | ------------------------------------------------------------ |
| `SaveMode.ErrorIfExists`(default) | `"error" or "errorifexists"`(default) | 当保存一个DataFrame的时候，如果数据已经存在，那么就会抛出一个指定的异常。 |
| `SaveMode.Append`                 | `"append"`                            | 当保存一个 DataFrame的时候，如果数据已经存在，那么新的数据就会append到原有的数据后面。 |
| `SaveMode.Overwrite`              | `"overwrite"`                         | Overwrite mode 指的是当保存一个DataFrame 的时候，如果数据或者表已经存在，那么就覆盖原来的数据 |
| `SaveMode.Ignore`                 | `"ignore"`                            | Ignore mode 指的是当保存一个 DataFrame 的时候，如果数据已经存在，那么这次的保存操作就会被忽略掉，这个方式很像SQL中的 `CREATE TABLE IF NOT EXISTS` in SQL. |

### Saving to Persistent Tables

`DataFrames` can also be saved as persistent tables into Hive metastore using the `saveAsTable` command. Notice that an existing Hive deployment is not necessary to use this feature. Spark will create a default local Hive metastore (using Derby) for you. Unlike the `createOrReplaceTempView`command, `saveAsTable` will materialize the contents of the DataFrame and create a pointer to the data in the Hive metastore. Persistent tables will still exist even after your Spark program has restarted, as long as you maintain your connection to the same metastore. A DataFrame for a persistent table can be created by calling the `table` method on a `SparkSession` with the name of the table.

For file-based data source, e.g. text, parquet, json, etc. you can specify a custom table path via the `path` option, e.g. `df.write.option("path", "/some/path").saveAsTable("t")`. When the table is dropped, the custom table path will not be removed and the table data is still there. If no custom table path is specified, Spark will write data to a default table path under the warehouse directory. When the table is dropped, the default table path will be removed too.

Starting from Spark 2.1, persistent datasource tables have per-partition metadata stored in the Hive metastore. This brings several benefits:

- Since the metastore can return only necessary partitions for a query, discovering all the partitions on the first query to the table is no longer needed.
- Hive DDLs such as `ALTER TABLE PARTITION ... SET LOCATION` are now available for tables created with the Datasource API.

Note that partition information is not gathered by default when creating external datasource tables (those with a `path` option). To sync the partition information in the metastore, you can invoke `MSCK REPAIR TABLE`.

`DataFrames` 也能够通过使用Hive元数据存储中的`saveAsTable`命令来持久化到一个表中。注意，使用这个功能不一定要必须存在一个已经部署了的Hive项目。如果没有提供Hive，Spark 就会为你创建一个默认的本地化的Hive元数据存储。不像 `createOrReplaceTempView`命令, `saveAsTable`将会将DataFrame的内容元数据化 然后创建一个指针指向Hive元数据存储对应的数据。持久化了的表会一直存在，即使你的spark项目重启了，只要你对你的相同的元数据保持你的连接。一个 持久化到表中的DataFrame 能够通过使用在 `SparkSession` 中调用带有表名的 `table` 方法来创建。

对于一个基于文件的数据比如. text, parquet, json, etc. 你能够通过 `path` 选项参数来指定自定义表的路径，比如`df.write.option("path", "/some/path").saveAsTable("t")`当这个表被删除了之后，自定义的表的路径会被删掉，但是数据会还在那里。如果没有指定自定义表的路径。Spark会将数据写到一个默认的warehouse目录下的表中去。当表被删除，默认的表的路径也会被删除掉。

从Spark 2.1开始，持久化数据有了一个预分区的元数据属性，存在于Hive的元数据存储中。这种方式带有几个好处：

- 对一个查询，只需要返回必要的分区上的元数据化了的数据，第一次对表的查询就不再需要去查找所有的分区了。
- Hive DDLs 比如 `ALTER TABLE PARTITION ... SET LOCATION` 现在能使用Datasource API来创建表了。

注意，当创建一个已存在的数据表的时候分区信息不是默认就收集的 (他是`path` 属性中的). 在元数据化了的数据中同步分区信息你可以使用 `MSCK REPAIR TABLE`。

### Bucketing, Sorting and Partitioning

For file-based data source, it is also possible to bucket and sort or partition the output. Bucketing and sorting are applicable only to persistent tables:

基于文件的数据，他也能进行存储或者进行 sort（排序） or partition（分区） 操作；存储和排序只在持久化了的表里面有效

```java
peopleDF.write().bucketBy(42, "name").sortBy("age").saveAsTable("people_bucket");
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSQLDataSourceExample.java" in the Spark repo.

while partitioning can be used with both `save` and `saveAsTable` when using the Dataset APIs.

当使用 Dataset APIs分区能够使用 `save` and `saveAsTable` 方法来实现。



```java
usersDF
  .write()
  .partitionBy("favorite_color")
  .format("parquet")
  .save("namesPartByColor.parquet");
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSQLDataSourceExample.java" in the Spark repo.

It is possible to use both partitioning and bucketing for a single table:

也能对单个表进行分区和保存操作：



```java
eopleDF
  .write()
  .partitionBy("favorite_color")
  .bucketBy(42, "name")
  .saveAsTable("people_partitioned_bucketed");

```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSQLDataSourceExample.java" in the Spark repo.

`partitionBy` creates a directory structure as described in the [Partition Discovery](https://spark.apache.org/docs/latest/sql-programming-guide.html#partition-discovery) section. Thus, it has limited applicability to columns with high cardinality. In contrast `bucketBy` distributes data across a fixed number of buckets and can be used when a number of unique values is unbounded.

`partitionBy` 这个方法创建了一个直接的结构，在 [Partition Discovery](https://spark.apache.org/docs/latest/sql-programming-guide.html#partition-discovery) 章节中。 然而，它对高基数的列的使用很有限，相反 `bucketBy` 通过buckets 中的数据来灵活的分配数据，所以他能够用于当唯一数据的个数没有限制的时候。

## Parquet Files

[Parquet](http://parquet.io/) is a columnar format that is supported by many other data processing systems. Spark SQL provides support for both reading and writing Parquet files that automatically preserves the schema of the original data. When writing Parquet files, all columns are automatically converted to be nullable for compatibility reasons.

[Parquet](http://parquet.io/) 是一个列格式的而且是能被其他很多数据处理系统支持的一种数据结构。 Spark SQL 提供了对 Parquet 文件的读和写的支持，他能自动的维护原始数据中的schema。当写 Parquet 文件的时候，为了数据的完整性所有的列都会自动的被转化成nullable的。

### Loading Data Programmatically

Using the data from the above example:

```java
import org.apache.spark.api.java.function.MapFunction;
import org.apache.spark.sql.Encoders;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;

Dataset<Row> peopleDF = spark.read().json("examples/src/main/resources/people.json");

// DataFrames can be saved as Parquet files, maintaining the schema information
peopleDF.write().parquet("people.parquet");

// Read in the Parquet file created above.
// Parquet files are self-describing so the schema is preserved
// The result of loading a parquet file is also a DataFrame
Dataset<Row> parquetFileDF = spark.read().parquet("people.parquet");

// Parquet files can also be used to create a temporary view and then used in SQL statements
parquetFileDF.createOrReplaceTempView("parquetFile");
Dataset<Row> namesDF = spark.sql("SELECT name FROM parquetFile WHERE age BETWEEN 13 AND 19");
Dataset<String> namesDS = namesDF.map(
    (MapFunction<Row, String>) row -> "Name: " + row.getString(0),
    Encoders.STRING());
namesDS.show();
// +------------+
// |       value|
// +------------+
// |Name: Justin|
// +------------+
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSQLDataSourceExample.java" in the Spark repo.

### Partition Discovery

Table partitioning is a common optimization approach used in systems like Hive. In a partitioned table, data are usually stored in different directories, with partitioning column values encoded in the path of each partition directory. All built-in file sources (including Text/CSV/JSON/ORC/Parquet) are able to discover and infer partitioning information automatically. For example, we can store all our previously used population data into a partitioned table using the following directory structure, with two extra columns, `gender` and `country` as partitioning columns:

在系统中使用表分区是一个很通用的优雅的做法，比如Hive，在一个分区了的表中，数据一般存储在不同的目录中，分区的列的值被编码到一个分区目录的一个地址中。所有的被构建的文件数据 (包括 Text/CSV/JSON/ORC/Parquet) 都能够自动的引用和发现分区信息。例如，我们可以使用下面这种目录结构来保存我们之前用过了的人口数据到分区表里面，使用另外的两个字段 `gender` and `country` 作为分区字段：

```yaml
path
└── to
    └── table
        ├── gender=male
        │   ├── ...
        │   │
        │   ├── country=US
        │   │   └── data.parquet
        │   ├── country=CN
        │   │   └── data.parquet
        │   └── ...
        └── gender=female
            ├── ...
            │
            ├── country=US
            │   └── data.parquet
            ├── country=CN
            │   └── data.parquet
            └── ...
```

By passing `path/to/table` to either `SparkSession.read.parquet` or `SparkSession.read.load`, Spark SQL will automatically extract the partitioning information from the paths. Now the schema of the returned DataFrame becomes:

将`path/to/table` 传递到 `SparkSession.read.parquet` 或者 `SparkSession.read.load`中, Spark SQL 会自动的从路径中提取分区信息。现在spark的 schema 返回的 DataFrame 变成了这个样子

```yaml
root
|-- name: string (nullable = true)
|-- age: long (nullable = true)
|-- gender: string (nullable = true)
|-- country: string (nullable = true)
```

Notice that the data types of the partitioning columns are automatically inferred. Currently, numeric data types, date, timestamp and string type are supported. Sometimes users may not want to automatically infer the data types of the partitioning columns. For these use cases, the automatic type inference can be configured by `spark.sql.sources.partitionColumnTypeInference.enabled`, which is default to `true`. When type inference is disabled, string type will be used for the partitioning columns.

注意，分区中列中的数据类型会自动的根据原始的数据类型传递过来。现在, numeric data types, date, timestamp and string type 都已经支持。有时用户可能不想使分区中的自动产生的数据类型，对于这些用户的情况，可以通过设置`spark.sql.sources.partitionColumnTypeInference.enabled`来改变自动类型的推理，它默认是true的，当设置为false的时候，分区中的列的数据类型就全部是String了.

Starting from Spark 1.6.0, partition discovery only finds partitions under the given paths by default. For the above example, if users pass `path/to/table/gender=male` to either `SparkSession.read.parquet` or `SparkSession.read.load`, `gender` will not be considered as a partitioning column. If users need to specify the base path that partition discovery should start with, they can set `basePath` in the data source options. For example, when `path/to/table/gender=male` is the path of the data and users set `basePath` to `path/to/table/`, `gender` will be a partitioning column.

从 Spark 1.6.0开始， 默认情况下分区发现只能在给定路径的下找到分区信息。对于上面的例子，如果用户将 `path/to/table/gender=male` 传递到 `SparkSession.read.parquet` 或者 `SparkSession.read.load`中，那么 `gender` 就不会被认为是分区的列。如果用户想指定基本的分区发现的开始路径，他可以在数据源的可选参数中设置 `basePath` 的值。例如，当 `path/to/table/gender=male` 是数据的路径而用户设置了 `basePath` 为 `path/to/table/`，那么`gender` 就会被认为是分区列。

### Schema Merging

Like ProtocolBuffer, Avro, and Thrift, Parquet also supports schema evolution. Users can start with a simple schema, and gradually add more columns to the schema as needed. In this way, users may end up with multiple Parquet files with different but mutually compatible schemas. The Parquet data source is now able to automatically detect this case and merge schemas of all these files.

Since schema merging is a relatively expensive operation, and is not a necessity in most cases, we turned it off by default starting from 1.5.0. You may enable it by

1. setting data source option `mergeSchema` to `true` when reading Parquet files (as shown in the examples below), or
2. setting the global SQL option `spark.sql.parquet.mergeSchema` to `true`.

像 ProtocolBuffer, Avro, and Thrift一样， Parquet也支持schema演变，用户可以以一个简单的模式开始，然后逐步的添加需要的列到schema中，使用这种方法，用户最终可能会生成多个包含多个不同的但是兼容的schema的Parquet文件。这些 Parquet 数据源现在可以自动的检查这类情况，然后将这些文件中的schema合并。

然而 schema merging 是相对来说比较昂贵的操作。在大多数情况下不是必须的, 我们在1.5.0之后就取消了默认其用它，你可以使用个下面这个方法来启用它。

1. 当读取Parquet文件的时候，设置数据源的可选参数中的 `mergeSchema` 为true (就跟下面的例子中的那样), 或者
2. 设置全局的SQL的可选参数 `spark.sql.parquet.mergeSchema` 为 `true`.

```java
import java.io.Serializable;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;

public static class Square implements Serializable {
  private int value;
  private int square;

  // Getters and setters...

}

public static class Cube implements Serializable {
  private int value;
  private int cube;

  // Getters and setters...

}

List<Square> squares = new ArrayList<>();
for (int value = 1; value <= 5; value++) {
  Square square = new Square();
  square.setValue(value);
  square.setSquare(value * value);
  squares.add(square);
}

// Create a simple DataFrame, store into a partition directory
Dataset<Row> squaresDF = spark.createDataFrame(squares, Square.class);
squaresDF.write().parquet("data/test_table/key=1");

List<Cube> cubes = new ArrayList<>();
for (int value = 6; value <= 10; value++) {
  Cube cube = new Cube();
  cube.setValue(value);
  cube.setCube(value * value * value);
  cubes.add(cube);
}

// Create another DataFrame in a new partition directory,
// adding a new column and dropping an existing column
Dataset<Row> cubesDF = spark.createDataFrame(cubes, Cube.class);
cubesDF.write().parquet("data/test_table/key=2");

// Read the partitioned table
Dataset<Row> mergedDF = spark.read().option("mergeSchema", true).parquet("data/test_table");
mergedDF.printSchema();

// The final schema consists of all 3 columns in the Parquet files together
// with the partitioning column appeared in the partition directory paths
// root
//  |-- value: int (nullable = true)
//  |-- square: int (nullable = true)
//  |-- cube: int (nullable = true)
//  |-- key: int (nullable = true)
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/sql/JavaSQLDataSourceExample.java" in the Spark repo.

### Hive metastore Parquet table conversion

When reading from and writing to Hive metastore Parquet tables, Spark SQL will try to use its own Parquet support instead of Hive SerDe for better performance. This behavior is controlled by the `spark.sql.hive.convertMetastoreParquet` configuration, and is turned on by default.

当对Parquet表进行读和写Hive元数据化读写的时候， Spark SQL 将会试着使用自己支持的Parquet来代替Hive SerDe ，从而来达到更好的效果，这种行为通过配置 `spark.sql.hive.convertMetastoreParquet` 来控制，它默认是打开的状态。

#### Hive/Parquet Schema Reconciliation

There are two key differences between Hive and Parquet from the perspective of table schema processing.

1. Hive is case insensitive, while Parquet is not
2. Hive considers all columns nullable, while nullability in Parquet is significant

从表的schema的处理的角度来看，在Hive和Parquet中有连个主要的不同点：

1. Hive 大小写不敏感, 而 Parquet敏感
2. Hive 认为所有的列都是可以为null的，而在 Parquet 中可为空是有很重要意义的。

Due to this reason, we must reconcile Hive metastore schema with Parquet schema when converting a Hive metastore Parquet table to a Spark SQL Parquet table. The reconciliation rules are:

1. Fields that have the same name in both schema must have the same data type regardless of nullability. The reconciled field should have the data type of the Parquet side, so that nullability is respected.
2. The reconciled schema contains exactly those fields defined in Hive metastore schema.
   - Any fields that only appear in the Parquet schema are dropped in the reconciled schema.
   - Any fields that only appear in the Hive metastore schema are added as nullable field in the reconciled schema.

由于这个原因，我们在进行将Hive中的元数据存储的Parquet表转换成Spark SQL Parquet 表的时候，就必须协调好Hive中的schema和Parquet中的schema。 协调的规则是：

1. 所有的schema中有相同的名称的字段它的数据类型必须一样，不管是否能为null。协调之后的字段就会有Parquet侧的数据类型，所有字段是否能为null是要好好对待的。
2. 协调好之后的scheme包含如下的定义在hive中的额外的字段。
   - 任何只是出现在Parquet的schema中的字段都会在协调之后的schema中被丢掉。
   - 任何只出现在hive中的字段会在协调之后的schema中保留，并且可以是为null的。

#### Metadata Refreshing

Spark SQL caches Parquet metadata for better performance. When Hive metastore Parquet table conversion is enabled, metadata of those converted tables are also cached. If these tables are updated by Hive or other external tools, you need to refresh them manually to ensure consistent metadata.

Spark SQL 为了更好的表现会将 Parquet元数据化。当 将Hive 元数据化 Parquet 表的转换设置为true的的时候, 这些转换过后的元子化的数据也会被缓存下来。如果这些表被Hive跟新或者其他的扩展工具跟新，你需要去手动的刷新他们，保证数据的一致性。

```java
// spark is an existing SparkSession
spark.catalog().refreshTable("my_table");
```

### Configuration

Configuration of Parquet can be done using the `setConf` method on `SparkSession` or by running `SET key=value` commands using SQL.

可以通过配置SparkSession中的 `setConf` 方法来设置Parquet的参数或者在执行SQL的时候设置` key=value` 命令。

| Property Name                            | Default | Meaning                                                      |
| ---------------------------------------- | ------- | ------------------------------------------------------------ |
| `spark.sql.parquet.binaryAsString`       | false   | Some other Parquet-producing systems, in particular Impala, Hive, and older versions of Spark SQL, do not differentiate between binary data and strings when writing out the Parquet schema. This flag tells Spark SQL to interpret binary data as a string to provide compatibility with these systems. |
| `spark.sql.parquet.int96AsTimestamp`     | true    | Some Parquet-producing systems, in particular Impala and Hive, store Timestamp into INT96. This flag tells Spark SQL to interpret INT96 data as a timestamp to provide compatibility with these systems. |
| `spark.sql.parquet.cacheMetadata`        | true    | Turns on caching of Parquet schema metadata. Can speed up querying of static data. |
| `spark.sql.parquet.compression.codec`    | snappy  | Sets the compression codec use when writing Parquet files. Acceptable values include: uncompressed, snappy, gzip, lzo. |
| `spark.sql.parquet.filterPushdown`       | true    | Enables Parquet filter push-down optimization when set to true. |
| `spark.sql.hive.convertMetastoreParquet` | true    | When set to false, Spark SQL will use the Hive SerDe for parquet tables instead of the built in support. |
| `spark.sql.parquet.mergeSchema`          | false   | When true, the Parquet data source merges schemas collected from all data files, otherwise the schema is picked from the summary file or a random data file if no summary file is available. |
| `spark.sql.optimizer.metadataOnly`       | true    | When true, enable the metadata-only query optimization that use the table's metadata to produce the partition columns instead of table scans. It applies when all the columns scanned are partition columns and the query has an aggregate operator that satisfies distinct semantics. |

| Property Name                            | Default | Meaning                                                      |
| ---------------------------------------- | ------- | ------------------------------------------------------------ |
| `spark.sql.parquet.binaryAsString`       | false   | Some other Parquet-producing systems, in particular Impala, Hive, and older versions of Spark SQL, do not differentiate between binary data and strings when writing out the Parquet schema. This flag tells Spark SQL to interpret binary data as a string to provide compatibility with these systems. |
| `spark.sql.parquet.int96AsTimestamp`     | true    | Some Parquet-producing systems, in particular Impala and Hive, store Timestamp into INT96. This flag tells Spark SQL to interpret INT96 data as a timestamp to provide compatibility with these systems. |
| `spark.sql.parquet.cacheMetadata`        | true    | Turns on caching of Parquet schema metadata. Can speed up querying of static data. |
| `spark.sql.parquet.compression.codec`    | snappy  | Sets the compression codec use when writing Parquet files. Acceptable values include: uncompressed, snappy, gzip, lzo. |
| `spark.sql.parquet.filterPushdown`       | true    | Enables Parquet filter push-down optimization when set to true. |
| `spark.sql.hive.convertMetastoreParquet` | true    | When set to false, Spark SQL will use the Hive SerDe for parquet tables instead of the built in support. |
| `spark.sql.parquet.mergeSchema`          | false   | When true, the Parquet data source merges schemas collected from all data files, otherwise the schema is picked from the summary file or a random data file if no summary file is available. |
| `spark.sql.optimizer.metadataOnly`       | true    | When true, enable the metadata-only query optimization that use the table's metadata to produce the partition columns instead of table scans. It applies when all the columns scanned are partition columns and the query has an aggregate operator that satisfies distinct semantics. |











