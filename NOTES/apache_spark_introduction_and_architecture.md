# What is Apache Spark?

- In earlier days, we could work with small amounts of data, and one computer with medium resources was enough. But in modern days, data is very huge — often in millions — so a single computer cannot perform the computation efficiently.

- **Apache Spark** is a distributed computing framework where a group of computers work together to process large volumes of data in parallel. In Apache Spark, this group of computers is called **nodes in a cluster**.


# Why Apache Spark?

There are two primary approaches to handling big data:

## 1. Monolithic Approach

- Upgrade the existing system by adding more resources such as RAM and CPU cores.  
- Increase the computing power of a single machine.  
- This method is known as **vertical scaling**.

### Drawbacks of the Monolithic Approach

- A single machine has a physical limit on how much it can be scaled.  
- Very low availability due to dependency on one system.

## 2. Distributed Approach

- Add more machines or nodes to the network or cluster.  
- This method is known as **horizontal scaling**.  
- Provides **high availability** and better fault tolerance.


# Apache Spark vs. MapReduce (Before Apache Spark)

## How was big data processed before Apache Spark?

- Before Apache Spark, big data was primarily processed using **Hadoop MapReduce**.  
- Apache Spark was introduced around **2008–2009**.  
- Hadoop MapReduce also follows a **distributed computing model**. In this model:  
  - The **Map** phase is responsible for distributing the data across different nodes or machines.  
  - The **Reduce** phase gathers and processes the transformed data collected from all nodes.  
  - All intermediate results (such as filter, aggregation, group-by operations, etc.) are **written to disk**.

## Drawbacks of Hadoop MapReduce

- In Hadoop MapReduce, every stage involves writing data to disk (ROM) and reading it back again.  
- This continuous disk I/O makes the process **slow and time-consuming**.  
- In contrast, **Apache Spark performs computations in-memory (RAM)**, significantly reducing I/O overhead.  
- As a result, **Apache Spark is up to 100 times faster than Hadoop MapReduce** for many workloads.


# Apache Spark Features

1. **In-Memory Computation**  
   Enables faster processing by keeping data in RAM rather than frequently reading from and writing to disk.

2. **Lazy Evaluation**  
   Transformations are not executed immediately; Spark waits until an action is performed, which helps optimize the execution plan.

3. **Fault Tolerance**  
   Automatically recovers lost data and computations in case of node failures using lineage information.

4. **Partitioning**  
   Distributes data across multiple nodes to enable parallel processing and improve performance.

5. **Streaming Processing**  
   Supports real-time data processing through Spark Streaming or Structured Streaming.

6. **Batch Processing**  
   Efficiently handles large-scale batch data processing workloads.

# Pictorial Representation 

<img width="1245" height="701" alt="Apache Spark Features" src="https://github.com/user-attachments/assets/8db54da8-be8a-4e05-a6a5-7b65233e9ec5" />

# Apache Spark Architecture

Apache Spark follows a **Master–Slave architecture** to manage and process large-scale data efficiently.

# Key Components in Apache Spark

## 1. Resource Manager
- Acts as the **master component** of the cluster.  
- Responsible for **allocating resources** such as Driver and Worker nodes required to run an application.  
- Examples include YARN, Mesos, and Kubernetes.

## 2. Driver
- The Driver is responsible for **orchestrating** the execution of a Spark application.  
- It distributes tasks to the Worker nodes and manages the overall workflow.  
- The Driver sits between the **Resource Manager** and the **Worker nodes**.

## 3. Workers
- Worker nodes are responsible for **executing the tasks** assigned by the Driver.  
- They perform the actual computations on the data.

Together, the **Driver** and **Workers** form the **Spark cluster**, which processes data in a distributed manner.


# Apache Spark Flow

When a Spark application is submitted using **spark-submit**, it includes configuration details such as:

- **1 Driver** with 10 GB memory  
- **3 Executors**, each with 20 GB memory  

Below is the typical flow of how Spark processes this submission:

1. **Submitting the Application**  
   - The `spark-submit` command sends the application code along with its configuration to the **Resource Manager**.

2. **Driver Allocation**  
   - Upon receiving the request, the Resource Manager first allocates and launches the **Driver node**.

3. **Driver Initialization**  
   - The Driver reads the configuration and instructions provided by the Resource Manager.  
   - The Driver then requests the Resource Manager to allocate **3 Executor nodes**.

4. **Executor Allocation**  
   - The Resource Manager allocates the required Executors and hands them over to the Driver.  
   - Once all required resources are allocated, the Resource Manager steps back and the Driver takes full control.

5. **Driver–Worker Coordination**  
   - The Driver establishes a connection with the 3 Executor (Worker) nodes.  
   - The Driver contains the application logic and **distributes tasks** to the Executors.  
   - The actual computation is performed by the **Executor/Worker nodes**.

This flow ensures distributed, parallel processing of data within a Spark cluster.

# Pictorial Representation 

<img width="1865" height="868" alt="Screenshot 2025-11-13 103925" src="https://github.com/user-attachments/assets/a385b375-e132-47fb-9e7b-416aaf05178b" />

