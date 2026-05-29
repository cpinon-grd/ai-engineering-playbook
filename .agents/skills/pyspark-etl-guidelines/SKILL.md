---
name: pyspark-etl-guidelines
description: Use when writing, reviewing, refactoring, or debugging PySpark ETL, Spark SQL, Apache Iceberg, cumulative tables, snapshot tables, joins, window functions, map operations, partition-aware reads, or production data engineering code.
---

# PySpark ETL Guidelines

Apply these rules when generating or reviewing PySpark ETL code. Favor performant, idiomatic, testable code that is readable and safe for cumulative or snapshot tables.

## Project Structure

### Use an ETL class scaffold

Create a base class that manages the SparkSession lifecycle. Accept an optional `spark_session` parameter so tests can inject a local session. Use an abstract method for the job logic.

```python
import logging
from abc import ABC, abstractmethod
from pyspark.sql import SparkSession


class BaseETL(ABC):
    def __init__(self, config, app_name="ETL Job", spark_session=None):
        self.spark = spark_session or SparkSession.builder.appName(app_name).getOrCreate()
        self.config = config
        self.logger = logging.getLogger(self.__class__.__name__)

    @abstractmethod
    def run_job(self):
        ...

    def stop(self):
        self.spark.stop()
```

### Keep config construction testable

Keep dataclasses as pure data. Put CLI parsing in a standalone factory function so tests can construct configs without touching `sys.argv`.

```python
import argparse
from dataclasses import dataclass


@dataclass
class MyConfig:
    read_date: int = 20200101


def create_config() -> MyConfig:
    parser = argparse.ArgumentParser()
    parser.add_argument("--read_date", type=int, default=20200101)
    args = parser.parse_args()
    return MyConfig(read_date=args.read_date)
```

### Keep `run_job` as orchestration

Use named methods and `.transform()` for pipeline composition.

```python
events = (
    self.read_source()
    .transform(self.enrich)
    .transform(self.merge_with_existing)
)
```

### Use a shared partition-aware reader

Build a generic reader utility for partition mechanics such as date filters, hour ranges, and latest-partition lookups. Keep domain-specific business filters in the ETL where they are visible. Do not create one-off reader classes per table.

```python
import pyspark.sql.functions as F


class PartitionedReader:
    @staticmethod
    def read_latest(spark, table_name, partition_col):
        table_df = spark.read.table(table_name)
        row = table_df.agg(F.max(partition_col)).first()
        if row is None or row[0] is None:
            return spark.createDataFrame([], table_df.schema)
        return table_df.filter(F.col(partition_col) == row[0])

    @staticmethod
    def read_by_date(spark, table_name, partition_col, date_value):
        return spark.read.table(table_name).filter(F.col(partition_col) == date_value)


events = PartitionedReader.read_by_date(
    spark,
    "catalog.my_table",
    "event_date",
    20260319,
)
events = events.filter(F.col("event_type").isin("login", "purchase"))
```

### Reuse simple merge utilities

For simple outer-join-with-coalesce merges, use a reusable merge function that handles aliasing, join key coalescing, and per-column defaults. Use `map_zip_with` when per-key conflict resolution is needed, such as timestamp-aware merges.

## Code Style

### Always use `import pyspark.sql.functions as F`

Use `F.col()`, `F.when()`, `F.lit()`, and related functions throughout. Avoid DataFrame attribute column access because it binds the column to a specific DataFrame variable and breaks after joins or reassignment.

```python
# Bad
df.select(F.lower(df1.colA), F.upper(df2.colB))

# Good
df.select(F.lower(F.col("colA")), F.upper(F.col("colB")))
```

### Extract complex conditions

Keep logic inside `.filter()` or `F.when()` to about three expressions. Extract the rest into named variables.

```python
is_delivered = F.col("status") == "Delivered"
date_passed = F.datediff(F.col("date_a"), F.col("date_b")) < 0
has_registration = F.col("registration").rlike(".+")

status = F.when(is_delivered | (date_passed & has_registration), "Active")
```

### Prefer `select` over `withColumn` chains

Use `select` to specify the output schema in one pass. Avoid long `withColumn` chains that create repeated projections.

```python
df = df.select(
    F.col("a").cast("double").alias("a"),
    F.upper(F.col("b")).alias("b"),
    F.lit(1).alias("c"),
)
```

### Use `alias` instead of `withColumnRenamed`

```python
df = df.select("key", F.col("comments").alias("num_comments"))
```

### Keep chains short and separated by concern

Use no more than about five statements per chain. Separate projection/filtering, derived columns, joins, and drops.

```python
df = df.select("a", "b", "key").filter(F.col("a") == "x")
df = df.withColumn("ratio", F.col("a") / F.col("b"))
df = df.join(df2, "key", how="inner").drop("b")
```

## Joins

### Always specify `how=`

```python
df = df.join(other, "key", how="inner")
```

### Prefer left joins over right joins

Keep the primary DataFrame on the left and flip DataFrame order instead of using `how="right"`.

```python
flights = flights.join(aircraft, "aircraft_id", how="left")
```

### Use DataFrame aliases for disambiguation

Alias whole DataFrames instead of renaming every column before a join.

```python
flights = flights.alias("f")
parking = parking.alias("p")

result = flights.join(parking, "code", how="left").select(
    F.col("f.start_time").alias("flight_start"),
    F.col("p.total_time").alias("parking_total"),
)
```

### Broadcast small dimension tables

Use `F.broadcast()` when joining a large fact DataFrame to a lookup or dimension table small enough to fit in executor memory. This is especially useful when Spark cannot infer the size after filters or transformations.

```python
df = df.join(F.broadcast(category_dim), "category_id", how="left")
```

Use development-only checks such as Spark UI scan size, a quick row count, or `df.explain()` to confirm broadcast suitability. Do not add exploratory counts to production code.

### Do not use `.dropDuplicates()` as a crutch

If duplicate rows appear, find the root cause. `.dropDuplicates()` can mask join bugs and adds shuffle overhead.

## Window Functions

Use `from pyspark.sql import Window as W` alongside `import pyspark.sql.functions as F`.

### Always specify an explicit frame

Spark chooses different default frames depending on whether `orderBy` is present. Make the intended frame explicit.

```python
from pyspark.sql import Window as W

w = (
    W.partitionBy("key")
    .orderBy("num")
    .rowsBetween(W.unboundedPreceding, W.unboundedFollowing)
)
```

### Choose `row_number` or `first` intentionally

Use `row_number` plus filter to drop rows and keep the best row. Use `first` over a window to overwrite a column value while keeping all rows.

### Use `ignorenulls=True`

Use `ignorenulls=True` with `F.first()` and `F.last()` so a null first or last row does not null out the partition.

```python
F.first("version", ignorenulls=True).over(window)
```

### Avoid empty `partitionBy()`

An empty window partition can force all data into one partition. Use `.agg()` for global aggregations.

```python
df = df.agg(F.sum("num").alias("total"))
```

## Map and Array Higher-Order Functions

### Use `map_zip_with` for conflict-aware map merges

Use `map_concat` only when there is no key overlap or precedence is intentionally simple. Use `map_zip_with` for custom per-key conflict resolution.

```python
merged_map = F.map_zip_with(
    F.col("new_map"),
    F.col("existing_map"),
    lambda key, v1, v2: (
        F.when(v1.isNull(), v2)
        .when(v2.isNull(), v1)
        .otherwise(F.when(v1.event_ts >= v2.event_ts, v1).otherwise(v2))
    ),
)
```

### Use native higher-order functions for nested structs

```python
max_event_ts = F.array_max(
    F.transform(F.map_values(F.col("my_map")), lambda x: x.event_ts)
)
```

### Avoid UDFs by default

Use built-in Spark SQL functions or higher-order functions before writing a UDF. UDFs block Catalyst optimization and add serialization overhead.

## Cumulative and Snapshot Tables

### Make merges idempotent

Re-running with the same input should produce the same result and should not create duplicates.

### Make merges order-independent

Backfilling old data should not overwrite newer data. Use explicit ordering criteria such as event timestamp, version number, or partition date. Do not rely on positional precedence such as `coalesce` argument order when freshness matters.

### Validate primary keys after writes

Add audit checks for primary key uniqueness and nulls in key columns after writes.

## Data Quality and Performance

### Use `F.lit(None)` for empty columns

Use nulls for missing values. Do not use empty strings or placeholder strings such as `"NA"` unless the domain explicitly requires them.

```python
df = df.withColumn("foo", F.lit(None))
```

### Avoid `.otherwise()` as a catch-all

Leave unmapped values null when that better surfaces data quality gaps.

```python
platform = (
    F.when(F.col("platform_type") == "android", "Mobile")
    .when(F.col("platform_type") == "ios", "Mobile")
)
```

### Do not leave debug actions in production

Do not use `.show()`, `.collect()`, or `.printSchema()` in deployed ETL code. Use them only for local debugging. Use `.count()` only intentionally, such as for monitoring row counts or materializing before a DAG fork.

### Use `persist()` intentionally

Persist only when a DataFrame is referenced by multiple subsequent actions. Choose the storage level deliberately:

- `MEMORY_AND_DISK`: safe default that spills to disk if memory is tight.
- `MEMORY_ONLY`: faster but risks recomputation if evicted.
- `DISK_ONLY`: useful for very large DataFrames that do not fit in memory.

## Iceberg Write Patterns

### Use `.byName()` for schema evolution safety

Match columns by name rather than by position.

```python
df.write.byName().mode("overwrite").insertInto("catalog.my_table")
```

### Use Iceberg `__partitions` for latest partition reads

Use the Iceberg metadata table to find the latest snapshot partition instead of scanning the full data table.

```python
partition_df = spark.read.table("catalog.my_table__partitions").select(
    "partition.partition_date",
    "partition.partition_hour",
)

max_partition = partition_df.orderBy(
    F.col("partition_date").desc(),
    F.col("partition_hour").desc(),
).first()

if max_partition is None:
    raise ValueError("No partitions found in catalog.my_table")

latest_date = max_partition["partition_date"]
```

### Understand `write.distribution-mode`

- `"none"`: no reshuffle before writing. Fastest, but output file sizes depend on upstream partitioning.
- `"hash"`: redistributes data by partition key. Produces evenly sized files but adds a shuffle.
- `"range"`: sorts data by partition key before writing. Useful for ordered scan performance but most expensive.
