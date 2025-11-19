# Check the spark plan details

---

## Example Code

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
df_new = spark.createDataFrame(data, schema=schema)

# Here Performed filter() transformation nothing happened store in PLAN. 
# You not perform ACTION on it.
df_new = df_new.filter(col('city') == 'New York')

# 2 transformations performed 
df_new = df_new.select('city')

# Action 
display(df_new)
````

---

## Query Plan

```python
df_new.explain("formatted")
```

### Output

```
== Physical Plan ==
LocalTableScan (1)

(1) LocalTableScan
Output [1]: [city#11145]
Arguments: [city#11145]

== Photon Explanation ==
Photon does not fully support the query because:
        Unsupported node: LocalTableScan [city#11145].

Reference node:
        LocalTableScan [city#11145]
```

### Pictorial Respresentation of spark query plan 

<img width="1354" height="266" alt="spark query plan" src="https://github.com/user-attachments/assets/470bd2d7-d5a2-434f-8241-e34025fc80b6" />

# DAG  
---

- **DAG** means **"Directed Acyclic Graph"**. Whenever you perform a transformation, Spark creates a DAG for you.  
- Every Spark job has its own DAG.  
- A DAG flows in a forward direction and never returns to the starting point (i.e., it does not form a cycle).  
- A DAG is a pictorial representation of your Spark job details.
  

### Pictorial Respresentation of DAG - 1

  <img width="1108" height="970" alt="spark query plan dag" src="https://github.com/user-attachments/assets/9a5305d7-2aab-41d4-9dfb-6df12c25212c" />

### Pictorial Respresentation of DAG - 2

  <img width="1683" height="897" alt="DAG" src="https://github.com/user-attachments/assets/850ce8f3-8b32-43ff-8b9c-232f62657943" />

  Partitions 
----------
-> If you have the one Data frame with 1 million records how you distribute to the nodes / machines for that Partitions comes into picture. Basically create the partitions in Data frame and distribute the partitions to multiple nodes in cluster.

-> We cannot create the partitions on data but you can create logical partitions.

### Pictorial Respresentation of Partition

 <img width="1630" height="1077" alt="Partitions" src="https://github.com/user-attachments/assets/a6596e37-3322-465d-8a67-2d3dc6e52a26" />


# Understand Logical Partitions

## RDD
- **RDD** means *Resilient Distributed Dataset*.
- RDD is the backbone of Apache Spark.
- RDD is a datatype in Apache Spark (similar to Java `List`, `Set`).
- RDD contains a list of **logical partitions**.
- RDD logical partitions are distributed across the executors in the cluster.
- RDD is **immutable**.
- RDD has **Fault Tolerance**.

---

## Example

```python
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
df_new = spark.createDataFrame(data, schema=schema)

# Here, filter() transformation — nothing stored in PLAN yet (no ACTION performed).
df_new = df_new.filter(col('city') == 'New York')

# Second transformation
df_new = df_new.select('city')
````

---

## Explanation

Here we performed **two transformations**:

1. `filter()`
2. `select()`

Each transformation does **not** modify the original data.
Apache Spark internally uses RDDs and creates **one logical partition** for each transformation:

```
df_new = df_new.filter(col('city') == 'New York') 
→ RDD1 (creates one logical partition for the first transformation)

df_new = df_new.select('city') 
→ RDD2 (creates one logical partition for the second transformation)
```

---

## Fault Tolerance

* Suppose **RDD2 fails** due to some issue.
* Spark reads the existing **DAG** and **recreates RDD2 automatically**.
* This behavior is called **Resilient** or **Fault Tolerant**.

---

## Note

* You don't need to write code using raw RDDs.
* RDD code is not optimized manually.
* DataFrame API is optimized by **Catalyst Optimizer** (Driver node).
* Spark generates optimized execution plans internally and then uses RDDs for execution.


### Pictorial Respresentation of RDD

   <img width="1855" height="928" alt="RDD" src="https://github.com/user-attachments/assets/60a94110-85d9-45f6-83b1-6cc2f296a306" />


