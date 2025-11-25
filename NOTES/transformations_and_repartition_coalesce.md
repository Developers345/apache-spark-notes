# Transformations

There are 2 types of transformations present:

## 1. Narrow

- Narrow Transformation means it does not take help from different partitions.
- Narrow Transformation is independent; it will not rely on any other partition and performs operations independently.
- Narrow Transformation is a **one-to-one** relationship.  
  **Examples:** `filter`, `select`

## 2. Wide

- Wide Transformation means it takes help from another partition to perform the task.
- Wide Transformation is dependent.
- Wide Transformation is a **one-to-many** relationship.
- By default, Spark will create **200 partitions** to perform Wide Transformation.  
  **Example:** `groupBy`

---
# Pictorial Representation for Narrow and Wide Transformation 

<img width="1817" height="1029" alt="Narrow And Wide Transformation" src="https://github.com/user-attachments/assets/715f5c61-67ca-4107-b833-1c993721f5b3" />


# Example Program for Narrow and Wide Transformation

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *
from pyspark.sql.functions import *

# Sample data
data = [
    ("Alice", 25, "New York"),
    ("Bob", 30, "San Francisco"),
    ("Charlie", 35, "Chicago")
]

# Define schema
schema = StructType([
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True),
    StructField("city", StringType(), True)
])

# Create DataFrame
df = spark.createDataFrame(data, schema=schema)

# Narrow Transformation
df = df.filter(col('city') == 'New York')
display(df)

# Wide Transformation
df = df.groupBy('city').agg(max(col('age')))
display(df)
df.explain('formatted')
````

---

# Output for Narrow Transformation

```
== Physical Plan ==
LocalTableScan (1)

(1) LocalTableScan
Output [3]: [name#11284, age#11285, city#11286]
Arguments: [name#11284, age#11285, city#11286]

== Photon Explanation ==
Photon does not fully support the query because:
        Unsupported node: LocalTableScan [name#11284, age#11285, city#11286].

Reference node:
        LocalTableScan [name#11284, age#11285, city#11286]
```

---

# Output for Wide Transformation

```
== Physical Plan ==
AdaptiveSparkPlan (6)
+- == Initial Plan ==
   ColumnarToRow (5)
   +- PhotonResultStage (4)
      +- PhotonGroupingAgg (3)
         +- PhotonRowToColumnar (2)
            +- LocalTableScan (1)

(1) LocalTableScan
Output [2]: [age#11252, city#11253]
Arguments: [age#11252, city#11253]

(2) PhotonRowToColumnar
Input [2]: [age#11252, city#11253]

(3) PhotonGroupingAgg
Input [2]: [age#11252, city#11253]
Arguments: [city#11253], [max(age#11252)], [max(age)#11269], [city#11253, max(age)#11269 AS max(age)#11270], true

(4) PhotonResultStage
Input [2]: [city#11253, max(age)#11270]

(5) ColumnarToRow
Input [2]: [city#11253, max(age)#11270]

(6) AdaptiveSparkPlan
Output [2]: [city#11253, max(age)#11270]
Arguments: isFinalPlan=false
```
# Query Plan and DAG Representation for Narrow Transformation

<img width="624" height="260" alt="narrow transformation query plan" src="https://github.com/user-attachments/assets/4e387737-7021-401a-b54d-0e69e3800094" />

<img width="504" height="787" alt="Narrow Transformation Dag Representation" src="https://github.com/user-attachments/assets/23963042-5b9a-4a93-b42f-e689f989107f" />

# Query Plan and DAG Representation for Wide Transformation

<img width="1122" height="615" alt="wide transformation query plan" src="https://github.com/user-attachments/assets/a7874d55-a92f-41b4-bee7-73a02715ce6b" />

<img width="625" height="923" alt="Wide Transformation DAG - 1" src="https://github.com/user-attachments/assets/18ff493b-dd18-422f-b564-e8a136a2426b" />

<img width="394" height="915" alt="Wide Transformation DAG - 2" src="https://github.com/user-attachments/assets/002afb57-02ce-44fe-a57a-aa02fbb0ec42" />


# Repartition VS Coalesce

- Default partition (block) size is **128 MB**.
- When you want to **increase** the number of partitions, use **`repartition()`**.  
  Repartition performs a **full shuffle** of data.
- When you want to **decrease** the number of partitions, **`repartition()` is not recommended** — use **`coalesce()`**.
- `coalesce()` can **shuffle or avoid shuffle** depending on parameters.

---
# Pictorial Representation for RePartition

<img width="653" height="602" alt="Repartition" src="https://github.com/user-attachments/assets/9035b16d-26ca-4304-a215-65cb8be09f9c" />

# Pictorial Representation for Coalesce

<img width="748" height="544" alt="coalescue" src="https://github.com/user-attachments/assets/aa921392-d215-4b76-ba43-d83a2071259f" />

# Pictorial Representation for Coalesce without shuffle data

<img width="994" height="704" alt="Coalesce Wihout shuffle data" src="https://github.com/user-attachments/assets/c44011e0-83f9-4437-af15-85aff80e0e76" />

# Pictorial Representation for Coalesce with shuffle data

<img width="1283" height="703" alt="Coalesce with shuffle data" src="https://github.com/user-attachments/assets/d3411275-46e6-47c8-a0cc-064ecd8f3cad" />


# Example Program for Repartition VS Coalesce

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *
from pyspark.sql.functions import *

# Sample data
data = [
    ("Alice", 25, "New York"),
    ("Bob", 30, "San Francisco"),
    ("Charlie", 35, "Chicago")
]

# Define schema
schema = StructType([
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True),
    StructField("city", StringType(), True)
])

# Create DataFrame
df = spark.createDataFrame(data, schema=schema)

df = df.groupBy('city').agg(max(col('age')))  # created RDD1
display(df)

# Show number of partitions
df.select(spark_partition_id().alias("pid")).groupBy("pid").count().show()

df = df.repartition(5)  # created RDD2
df.explain('formatted')
````

---

# Output for Repartition

```
== Physical Plan ==
AdaptiveSparkPlan (14)
+- == Initial Plan ==
   ColumnarToRow (13)
   +- PhotonResultStage (12)
      +- PhotonShuffleExchangeSource (11)
         +- PhotonShuffleMapStage (10)
            +- PhotonShuffleExchangeSink (9)
               +- PhotonSort (8)
                  +- PhotonGroupingAgg (7)
                     +- PhotonShuffleExchangeSource (6)
                        +- PhotonShuffleMapStage (5)
                           +- PhotonShuffleExchangeSink (4)
                              +- PhotonGroupingAgg (3)
                                 +- PhotonRowToColumnar (2)
                                    +- LocalTableScan (1)

(1) LocalTableScan
Output [2]: [age#11604, city#11605]
Arguments: [age#11604, city#11605]

(2) PhotonRowToColumnar
Input [2]: [age#11604, city#11605]

(3) PhotonGroupingAgg
Input [2]: [age#11604, city#11605]
Arguments: [city#11605], [partial_max(age#11604) AS max#11662], [max#11661], [city#11605, max#11662], false

(4) PhotonShuffleExchangeSink
Input [2]: [city#11605, max#11662]
Arguments: hashpartitioning(city#11605, 1024)

(5) PhotonShuffleMapStage
Input [2]: [city#11605, max#11662]
Arguments: ENSURE_REQUIREMENTS, [id=#9704]

(6) PhotonShuffleExchangeSource
Input [2]: [city#11605, max#11662]

(7) PhotonGroupingAgg
Input [2]: [city#11605, max#11662]
Arguments: [city#11605], [finalmerge_max(merge max#11662) AS max(age)#11659], [max(age)#11659], [city#11605, max(age)#11659 AS max(age)#11660], true

(8) PhotonSort
Input [2]: [city#11605, max(age)#11660]
Arguments: [city#11605 ASC NULLS FIRST, max(age)#11660 ASC NULLS FIRST]

(9) PhotonShuffleExchangeSink
Input [2]: [city#11605, max(age)#11660]
Arguments: RoundRobinPartitioning(5)

(10) PhotonShuffleMapStage
Input [2]: [city#11605, max(age)#11660]
Arguments: REPARTITION_BY_NUM, [id=#9712]

(11) PhotonShuffleExchangeSource
Input [2]: [city#11605, max(age)#11660]

(12) PhotonResultStage
Input [2]: [city#11605, max(age)#11660]

(13) ColumnarToRow
Input [2]: [city#11605, max(age)#11660]

(14) AdaptiveSparkPlan
Output [2]: [city#11605, max(age)#11660]
Arguments: isFinalPlan=false
```

---

# Coalesce Example

```python
df = df.coalesce(1)  # created RDD3
df.explain('formatted')
```

---

# Output for Coalesce

```
== Physical Plan ==
AdaptiveSparkPlan (15)
+- == Initial Plan ==
   Coalesce (14)
   +- ColumnarToRow (13)
      +- PhotonResultStage (12)
         +- PhotonShuffleExchangeSource (11)
            +- PhotonShuffleMapStage (10)
               +- PhotonShuffleExchangeSink (9)
                  +- PhotonSort (8)
                     +- PhotonGroupingAgg (7)
                        +- PhotonShuffleExchangeSource (6)
                           +- PhotonShuffleMapStage (5)
                              +- PhotonShuffleExchangeSink (4)
                                 +- PhotonGroupingAgg (3)
                                    +- PhotonRowToColumnar (2)
                                       +- LocalTableScan (1)

(1) LocalTableScan
Output [2]: [age#11604, city#11605]
Arguments: [age#11604, city#11605]

(2) PhotonRowToColumnar
Input [2]: [age#11604, city#11605]

(3) PhotonGroupingAgg
Input [2]: [age#11604, city#11605]
Arguments: [city#11605], [partial_max(age#11604) AS max#11670], [max#11669], [city#11605, max#11670], false

(4) PhotonShuffleExchangeSink
Input [2]: [city#11605, max#11670]
Arguments: hashpartitioning(city#11605, 1024)

(5) PhotonShuffleMapStage
Input [2]: [city#11605, max#11670]
Arguments: ENSURE_REQUIREMENTS, [id=#9784]

(6) PhotonShuffleExchangeSource
Input [2]: [city#11605, max#11670]

(7) PhotonGroupingAgg
Input [2]: [city#11605, max#11670]
Arguments: [city#11605], [finalmerge_max(merge max#11670) AS max(age)#11667], [max(age)#11667], [city#11605, max(age)#11667 AS max(age)#11668], true

(8) PhotonSort
Input [2]: [city#11605, max(age)#11668]
Arguments: [city#11605 ASC NULLS FIRST, max(age)#11668 ASC NULLS FIRST]

(9) PhotonShuffleExchangeSink
Input [2]: [city#11605, max(age)#11668]
Arguments: RoundRobinPartitioning(5)

(10) PhotonShuffleMapStage
Input [2]: [city#11605, max(age)#11668]
Arguments: REPARTITION_BY_NUM, [id=#9792]

(11) PhotonShuffleExchangeSource
Input [2]: [city#11605, max(age)#11668]

(12) PhotonResultStage
Input [2]: [city#11605, max(age)#11668]

(13) ColumnarToRow
Input [2]: [city#11605, max(age)#11668]

(14) Coalesce
Input [2]: [city#11605, max(age)#11668]
Arguments: 1

(15) AdaptiveSparkPlan
Output [2]: [city#11605, max(age)#11668]
Arguments: isFinalPlan=false
```

---

### Note

> If any RDD fails to execute, Spark can use the previous RDD lineage to recompute and create a new RDD.

# Query Plan Representation for Repartition

<img width="1110" height="509" alt="Repartition query plan" src="https://github.com/user-attachments/assets/4f6a2a7b-0c27-4dd1-b84b-d6cd7da8b657" />

# Query Plan Representation for Coalesce

<img width="1153" height="620" alt="Coalease Query Plan" src="https://github.com/user-attachments/assets/5a27217c-80e4-4aa0-9a28-5740418eba41" />

