# What is Databricks?

- Databricks is the most widely used cloud platform to work with big data.
- Databricks helps to work with Apache Spark. No need to install Apache Spark cluster locally — it takes a lot of time and cluster management becomes your responsibility such as start and stop operations.
- Databricks eliminates the Apache Spark cluster setup overhead. You only learn Apache Spark using Databricks Free Edition.

---

# How to Work with Apache Spark in Databricks?

## **STEP 1**
We create the **Workspace (Apache Spark Guide)** under the Workspace section.

- Go to the left side navigation panel → click **Workspace**
- You will find two options: **Home** and **Workspace**
- Select **Workspace** → create your new workspace *(Apache Spark Guide)*

---

## **STEP 2**
- Create a **Notebook**. Notebook provides a better UI for developers.  
  It divides code into multiple cells so you can execute code cell-by-cell.
- Create the notebook and **attach it to the Apache Spark cluster**.

**Steps:**

1. Open the Workspace you created *(Apache Spark Guide)*  
2. Create the Notebook inside it

---

## **STEP 3**
- In the **Free Edition**, a **Serverless Cluster** is available. No need to create a new Apache Spark cluster.
- Serverless cluster provides minimum configuration where Databricks acts as the resource manager.  
  It includes **one driver node** and **one worker node**.
- In Databricks, you **cannot manually create a SparkSession**.  
  Databricks automatically creates it for you.

---

### How to Create SparkSession (General Spark, Not Databricks)

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("tutorial")
````

* Here, `spark` is a Python variable.
* In Databricks, no need to create the SparkSession manually — just **print** it:

```python
print(spark)
```

---

# Code for Spark Session

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("tutorial").getOrCreate()

dir(spark)
print("Spark version:", spark.profile)
```

# Lazy Evaluation & Action

- **Lazy Evaluation & Action** is one of the key features in **Apache Spark**.
- The main purpose of this feature is that whenever Spark code is submitted, Spark will create all the **transformations** (like `filter`, `aggregation`) as a **plan**, but will **not execute** immediately because Spark first **optimizes** the code.
- Once the optimization is completed and we perform an **Action**, then a **Spark Job** is created and Spark starts executing the code.


### Common Spark Actions
- `.show()`
- `.count()`
- `display()`
- `.collect()`

---

## Pictorial Respresentation for Lazy Evaluation & Action

<img width="1909" height="1054" alt="Lazy Evaluation and Action Feature" src="https://github.com/user-attachments/assets/7ad33db3-4aec-45d1-a905-cf566fa78a75" />


## Example Code for Lazy Evaluation & Action

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *
from pyspark.sql.functions import *

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

# Transformation: filter()
# This will not be executed immediately and will be stored in the execution plan
df = df.filter(col('city') == 'New York')

# ACTION
display(df)

# ACTION
total = df.count()
print("Total Rows =", total)
````
