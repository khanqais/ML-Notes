**State four characteristics of Big Data and elaborate on them in the context of social media platforms**
The five characteristics of Big Data are commonly called the **5 Vs**: **Volume, Velocity, Variety, Veracity, and Value**. Social media platforms are a strong example because they continuously generate massive, fast-moving, mixed-format, and sometimes unreliable user data

## 1. Volume

**Volume** means the extremely large amount of data generated and stored. Social media platforms collect posts, likes, comments, shares, follows, messages, profile information, media uploads, and interaction logs from millions or billions of users

For example, on Instagram or YouTube, every video view, comment, like, watch-time record, and recommendation event adds to the platform’s data. This data is too large to process efficiently using a single traditional database, so platforms use distributed storage and processing systems such as Hadoop, Spark, cloud data lakes, and NoSQL databases.

## 2. Velocity

**Velocity** refers to the speed at which data is generated, received, and must be processed. Social media data arrives continuously and in real time, especially during live events, breaking news, sports matches, or viral trends.

For instance, during a football match, fans may post thousands of tweets, reels, comments, and live reactions every second. A platform must process this stream quickly to update trending topics, deliver notifications, detect spam, personalize the feed, and recommend related content before the moment becomes irrelevant.

## 3. Variety

**Variety** means that data exists in many formats: structured, semi-structured, and unstructured. Social media data is especially diverse because it includes text posts, images, videos, audio, emojis, hashtags, links, location tags, timestamps, and metadata.

A single Instagram post may contain an image, caption, hashtags, user ID, timestamp, location, comments, and engagement metrics. The image and video are unstructured, while user IDs, timestamps, and counts are structured or semi-structured. Platforms must use different technologies—such as object storage for media, search indexes for text, and databases for user and engagement data—to handle all of these formats.

## 4. Veracity

**Veracity** refers to the accuracy, reliability, and trustworthiness of data. Social media content is user-generated, so it may include fake accounts, bots, spam, misinformation, duplicate posts, biased opinions, and inconsistent metadata.

For example, a viral post claiming false health information may receive many likes and shares, but high engagement does not make it factually correct. Social media platforms therefore use verification systems, bot detection, spam filters, content moderation, and credibility signals to improve the quality of data used for recommendations, advertisements, and trend analysis.

## 5. Value

**Value** is the ability to convert raw data into useful insights or business decisions. Social media platforms analyze user behavior to improve feeds, target advertisements, recommend content, detect harmful activity, and understand audience interests

For example, if a user frequently watches cricket highlights and follows football pages, the platform can recommend related videos, accounts, and advertisements. The raw data itself has little value until analytics and machine-learning models turn it into personalized recommendations, better moderation, or revenue-generating ad targeting.

Q2)
**Describe the core components of Hadoop in detail**
Ans:
![[Pasted image 20261104204308.png]]
---
tags:
  - hadoop
  - big-data
  - sem7
  - exam
aliases:
  - Hadoop Concept
  - Hadoop Ecosystem
---

# Hadoop: Concept, Core Components and Ecosystem

> [!abstract] Topic frequency
> Asked 4 times. Covers 3 distinct questions (the two ecosystem questions are the same answer, so it is written once in section 3).

---

# 1. How Hadoop Addresses Big Data Challenges (5M)

## 1.1 What is Hadoop?
Apache Hadoop is an open-source framework that **stores and processes very large datasets across a cluster of commodity (cheap) computers**, using simple programming models. Instead of one super-computer, many ordinary machines share both **storage** and **computation**.

## 1.2 Big Data challenges (the 5 V's)

| Challenge | Meaning |
|---|---|
| **Volume** | Terabytes to petabytes; cannot fit on one disk or server |
| **Velocity** | Data arrives very fast (logs, sensors, clicks) |
| **Variety** | Structured (tables), semi-structured (JSON, XML), unstructured (text, images, video) |
| **Veracity** | Data may be noisy, incomplete or inconsistent |
| **Value** | Useful insight must be extracted at acceptable cost |

Traditional RDBMS struggle because they scale **vertically** (bigger server = very expensive), need a fixed schema, and move data to the program, which is slow for huge data.

## 1.3 How Hadoop solves each problem

| Challenge | Hadoop's solution |
|---|---|
| Huge volume | **HDFS** splits files into blocks (128 MB default) stored across many machines. Add nodes to grow (**horizontal scaling**). |
| Slow processing | **MapReduce / YARN** run the computation **in parallel** on many nodes. |
| Data movement cost | **Data locality**: code is sent to the node holding the data, not the other way round. |
| Hardware failure | **Replication** (default 3 copies) plus automatic task re-execution = **fault tolerance**. |
| Variety | **Schema-on-read**: any format can be stored; structure is applied when reading. |
| High cost | Runs on **commodity hardware**; open source, so no licence cost. |
| Velocity (streams) | Ecosystem tools such as **Flume, Kafka, Spark Streaming** ingest and process fast data. |

## 1.4 Example
A telecom company has 50 TB of call logs. One server would take days to scan them. On a 50-node Hadoop cluster, each node holds about 1 TB and processes its own part simultaneously, so the job takes minutes or hours. If one node dies, replicas keep the job running.

> [!tip] Exam tip
> For 5 marks: define Hadoop, list the 5 V's, give the 6-row table (volume, speed, locality, failure, variety, cost), and add a one-line example.

---

# 2. Core Components of Hadoop in Detail (10M)

```text
                    HADOOP
                       │
        ┌──────────────┼──────────────┐
       HDFS           YARN        MapReduce
      Storage       Resources     Processing
        └──────────────┴──────────────┘
                Hadoop Common
```

## 2.1 Hadoop Common
Shared libraries and utilities used by all other modules: configuration, file-system abstraction, RPC/networking, serialization, security, and the Java files needed to start Hadoop. It is a **shared toolkit**, not a storage or processing engine.

## 2.2 HDFS: storage layer
Hadoop Distributed File System stores very large files across the cluster. It follows a **master/worker** architecture.
- **NameNode (master):** keeps metadata (file names, directory tree, permissions, which blocks make up a file, which DataNodes hold them). It does not store file data.
- **DataNode (worker):** stores the actual blocks, serves read/write requests, sends **heartbeats** and block reports to the NameNode.
- **Block:** the file is cut into 128 MB blocks (default), and each block is **replicated** (default 3).
- **Secondary NameNode:** performs checkpointing (merges `fsimage` and `edits`). It is not a backup.
- Design: write-once-read-many, streaming access, high throughput.

## 2.3 YARN: resource management layer
Yet Another Resource Negotiator (Hadoop 2.x onwards). It separates **resource management** from **processing**, so non-MapReduce engines such as Spark can also run on the cluster.
- **ResourceManager (RM):** cluster-wide scheduler and ApplicationsManager. It decides who gets CPU and memory.
- **NodeManager (NM):** runs on each worker, launches and monitors containers, and reports resource usage.
- **ApplicationMaster (AM):** one per job. It negotiates containers from the RM and coordinates its own tasks.
- **Container:** a bundle of resources (e.g. 2 GB RAM and 1 vcore) in which a task runs (not a Docker container).

## 2.4 MapReduce: processing layer
A programming model and engine for parallel batch processing in the stages Input → **Map** → Shuffle and Sort → **Reduce** → Output. Mappers read splits and emit intermediate `(key, value)` pairs, and reducers aggregate all values of each key. See [[MapReduce Pipeline, Tasks and Failure Handling]].

## 2.5 Summary table

| Component | Role | Key services |
|---|---|---|
| Hadoop Common | Shared utilities | Libraries, RPC, config |
| HDFS | Storage | NameNode, DataNode |
| YARN | Resource management | ResourceManager, NodeManager, ApplicationMaster |
| MapReduce | Processing | Mapper, Reducer |

> [!note] Hadoop 1 vs 2
> In Hadoop 1 the JobTracker and TaskTracker did both resource management and processing, which made the JobTracker a bottleneck and a single point of failure. YARN replaced them in Hadoop 2.

---

# 3. Hadoop Ecosystem: Diagram and Components (10M)
> One answer covers both "sketch the diagram, explain any two components" and "explain components with diagram".

## 3.1 Diagram

```text
 ┌──────────────────────────────────────────────────────────────────┐
 │                     HADOOP ECOSYSTEM                             │
 ├──────────────────────────────────────────────────────────────────┤
 │ Management & Monitoring :  Ambari      Oozie (workflow)          │
 │ Coordination            :  ZooKeeper                             │
 ├──────────────────────────────────────────────────────────────────┤
 │ Data Access / Analysis :                                         │
 │   Hive (SQL)   Pig (scripts)   Mahout/MLlib (ML)   Spark         │
 ├──────────────────────────────────────────────────────────────────┤
 │ NoSQL DB       :  HBase  (random read/write on HDFS)             │
 ├──────────────────────────────────────────────────────────────────┤
 │ Processing     :  MapReduce   (and Spark, Tez on YARN)           │
 ├──────────────────────────────────────────────────────────────────┤
 │ Resource Mgmt  :  YARN                                           │
 ├──────────────────────────────────────────────────────────────────┤
 │ Storage        :  HDFS                                           │
 ├──────────────────────────────────────────────────────────────────┤
 │ Data Ingestion :  Sqoop (RDBMS ⇄ Hadoop)   Flume / Kafka (logs, │
 │                   streams)                                       │
 └──────────────────────────────────────────────────────────────────┘
   Sources: RDBMS, log files, social media, sensors, web
```

## 3.2 Components explained

| Component | Layer | Purpose |
|---|---|---|
| **HDFS** | Storage | Distributed, replicated file storage |
| **YARN** | Resource mgmt | Allocates CPU and memory to applications |
| **MapReduce** | Processing | Batch parallel processing (Map, Shuffle, Reduce) |
| **Spark** | Processing | In-memory engine, 10-100x faster than MR for iterative work; supports batch, streaming, SQL and ML |
| **Hive** | Data access | SQL-like language (**HiveQL**) over HDFS data, compiled into MR/Tez/Spark jobs |
| **Pig** | Data access | Data-flow scripting language (**Pig Latin**) for ETL |
| **HBase** | NoSQL DB | Column-oriented, real-time random read/write on HDFS (modelled on Google BigTable) |
| **Sqoop** | Ingestion | Bulk transfer between RDBMS (MySQL, Oracle) and HDFS/Hive |
| **Flume** | Ingestion | Collects and moves streaming **log data** into HDFS |
| **Kafka** | Ingestion | Distributed publish-subscribe message queue for real-time streams |
| **Oozie** | Workflow | Schedules and chains Hadoop jobs (DAG of actions) |
| **ZooKeeper** | Coordination | Centralised service for configuration, naming, leader election, synchronisation |
| **Mahout / MLlib** | Machine learning | Scalable ML libraries (clustering, classification, recommendation) |
| **Ambari** | Management | Web UI to provision, monitor and manage the cluster |

## 3.3 Two components in detail (as the "explain any two" part)

### (a) Apache Hive
- Developed by Facebook; lets analysts query big data **without writing Java**.
- Tables are **schema-on-read** definitions over files in HDFS; the schema is stored in the **Metastore**.
- Flow: `HiveQL query → Parser/Compiler → Optimizer → Execution plan (MR/Tez/Spark job) → run on YARN → result`.
- Suitable for **batch analytics and data warehousing**, not low-latency OLTP.

```sql
SELECT dept, SUM(salary) FROM employee GROUP BY dept;
```
This is internally a MapReduce job (Map emits `(dept, salary)`, Reduce sums).

### (b) Apache HBase
- Distributed, **column-family NoSQL** database that runs on top of HDFS.
- Provides **real-time random read/write** (HDFS alone only supports sequential access).
- Architecture: **HMaster** (assigns regions, schema changes), **RegionServers** (serve data), **ZooKeeper** (coordination). Data is stored in **Regions**, with a MemStore plus HFiles on HDFS.
- Use cases: messaging, time-series, user profiles.

## 3.4 One-line flow of using the ecosystem
Sqoop/Flume ingest data → stored in HDFS → YARN schedules → MapReduce/Spark process → Hive/Pig give easy query → HBase serves real-time access → Oozie schedules → Ambari monitors.



---
tags:
  - hdfs
  - hadoop
  - sem7
  - exam
aliases:
  - HDFS Questions
---

# HDFS: Architecture and Limitations

> [!abstract] Topic frequency
> Asked 3 times, as 3 distinct questions answered below.

---

# 1. Why HDFS Suits Large Datasets, Not Many Small Files (5M)

## 1.1 Design goals of HDFS
HDFS is built for **very large files (GB to TB)**, **write-once-read-many** access, and **high throughput streaming reads** rather than low latency.

## 1.2 Reasons it favours large files

1. **Large block size (128 MB default).**
   A 1 KB file still occupies a block *entry* in the metadata (it does not waste 128 MB on disk, but it still costs one block entry). A 1 GB file needs only 8 blocks.

2. **NameNode memory is the bottleneck.**
   The NameNode keeps all metadata **in RAM**. Each file, directory and block is an object of roughly **150 bytes**.
   - 1 file with 1 block = about 2 objects, about 300 bytes.
   - **10 million small files** = about 3 GB of NameNode heap just for metadata, and it grows with every new file.
   - The same data as a few big files needs only a tiny fraction.

3. **Seek time vs transfer time.**
   HDFS is tuned so that seek time is about 1% of transfer time. Many small files mean many seeks and many DataNode hops, which lowers throughput.

4. **MapReduce overhead.**
   One map task is created per input split, so one small file means one mapper. Thousands of tiny mappers spend more time **starting JVMs/containers** than doing work.

5. **Network and replication overhead.** Each small file is replicated 3 times and needs separate NameNode round trips.

## 1.3 Example

| Case | Files | Size | NameNode objects | Map tasks |
|---|---|---|---|---|
| A | 1 file | 1 TB | about 8,192 blocks | about 8,192 |
| B | 10 million files | 1 TB (100 KB each) | about 20 million objects | 10 million |

Case B overloads the NameNode and is very slow, although the data volume is the same.

## 1.4 Workarounds
Hadoop Archives (HAR), SequenceFiles (merge many small files as key-value pairs), `CombineFileInputFormat`, or HBase.

---

# 2. NameNode vs DataNode (5M)

| Basis | NameNode | DataNode |
|---|---|---|
| Role | **Master** | **Worker (slave)** |
| Stores | **Metadata**: namespace, file-to-block map, block locations, permissions | **Actual data blocks** on local disk |
| Number | One active (plus standby in HA) | Many (hundreds or thousands) |
| Memory/disk need | High **RAM** (metadata is held in memory) | High **disk capacity** |
| Files | `fsimage` and `edits` log | Block files and checksum (`.meta`) files |
| Communication | Receives heartbeats and block reports | Sends heartbeat (every 3 s) and block report |
| Client interaction | Gives block locations to the client | Client reads/writes the data **directly** from/to it |
| Failure impact | Cluster unavailable unless HA is configured | Blocks are re-replicated from other replicas |
| Instructions | Commands DataNodes to create, delete and replicate blocks | Executes those commands |

```text
Client ──"where is block A?"──> NameNode
Client <──"Worker1, Worker2"──
Client ──────read block A─────> DataNode (Worker1)
```

---

# 3. Secondary NameNode: Backup of NameNode? (True/False, 5M)

## Answer: **FALSE**

## Justification
The Secondary NameNode is **not** a hot or failover backup. Its job is **checkpointing**:

1. The NameNode records every change in an **`edits` log**; the **`fsimage`** is the last metadata snapshot.
2. If `edits` grows without limit, a NameNode restart takes very long.
3. The Secondary NameNode periodically (default 1 hour, or 1 million transactions) **downloads `fsimage` + `edits`, merges them** into a new `fsimage`, and sends it back.

```text
fsimage (old) + edits ──merge──> fsimage (new checkpoint)
```

Reasons it cannot act as a backup:
- It does **not receive live block reports or heartbeats**, so it does not know current block locations.
- It lags behind the NameNode by the checkpoint interval.
- It **cannot take over** if the NameNode crashes; clients cannot connect to it.

## What actually provides failover
**HDFS High Availability:** Active NameNode plus **Standby NameNode** sharing edits through **JournalNodes (QJM)**, with automatic failover via ZooKeeper (ZKFC). The Standby does the checkpointing too, so Secondary NameNode is not needed in HA clusters.

---
tags:
  - mapreduce
  - hadoop
  - sem7
  - exam
aliases:
  - MapReduce Pipeline
---

# MapReduce: Pipeline, Map Tasks, Shuffle and Sort, Reducer, Node Failure

> [!abstract] Topic frequency
> Asked 5 times. The pipeline question was asked twice verbatim, so it is answered once. Detailed theory is in [[MapReduce Execution Pipeline]].

---

# 1. MapReduce Execution Pipeline with Example (10M)

## 1.1 Overview
MapReduce processes data as `(key, value)` pairs in these stages:

```text
Input in HDFS
   ↓ InputFormat → logical InputSplits
   ↓ RecordReader → (key, value) records
   ↓ MAP → intermediate (key, value)
   ↓ Partition + Sort (map side), spill to local disk
   ↓ Combiner (optional local aggregation)
   ↓ SHUFFLE (reducers fetch their partitions)
   ↓ Merge-Sort + Grouping by key
   ↓ REDUCE
   ↓ OutputFormat / RecordWriter
Output files (part-r-xxxxx) in HDFS
```

## 1.2 Stages explained

| Stage | What happens |
|---|---|
| **Input / InputFormat** | Data in HDFS is divided into logical **input splits** (about the block size). One split = one map task. |
| **RecordReader** | Converts split bytes into records. With `TextInputFormat`: key = byte offset, value = line. |
| **Map** | User `map()` runs on every record and emits zero or more intermediate pairs. |
| **Partitioner** | Decides the target reducer: `hash(key) mod numReducers`. Same key always goes to the same reducer. |
| **Sort + Spill** | Map output is buffered in memory, sorted by key within each partition, and spilled to **local disk** (not HDFS). |
| **Combiner (optional)** | A "mini-reducer" that aggregates locally to cut network traffic. It may run 0 or more times. |
| **Shuffle** | Each reducer **copies** its partition from every mapper over the network (HTTP). |
| **Merge-sort / Group** | Fetched segments are merged and values are grouped per key: `key → [v1, v2, ...]`. |
| **Reduce** | User `reduce()` runs once per key and emits the result. |
| **Output** | `RecordWriter` writes to HDFS: one file per reducer (`part-r-00000`, `part-r-00001`...). |

## 1.3 Example: counting words

Input (2 splits):
```text
Split 1: cat dog cat / dog bird      Split 2: cat bird dog
```

| Step | Result |
|---|---|
| Map (M1) | (cat,1)(dog,1)(cat,1)(dog,1)(bird,1) |
| Map (M2) | (cat,1)(bird,1)(dog,1) |
| Combiner (M1) | (cat,2)(dog,2)(bird,1) |
| Combiner (M2) | (cat,1)(bird,1)(dog,1) |
| Partition (2 reducers, illustrative) | R1 gets bird, cat. R2 gets dog |
| Shuffle + Sort + Group | R1: `bird→[1,1]`, `cat→[2,1]`. R2: `dog→[2,1]` |
| Reduce (sum) | (bird,2) (cat,3) (dog,3) |
| Output | `part-r-00000`: bird 2, cat 3. `part-r-00001`: dog 3 |

## 1.4 YARN coordination
Client submits the job → ResourceManager starts the **ApplicationMaster** → AM asks for containers → NodeManagers launch map and reduce tasks → AM tracks progress → output written to HDFS.

> [!warning] Points examiners like
> Split is not the same as block; combiner is optional; intermediate data is on local disk; shuffle can overlap with the map phase; multiple reducers give multiple output files.

---

# 2. Role of Map Tasks with Example (5M)

## 2.1 Role
A **map task** is the first processing step. It:
1. Reads **one input split** (one map task per split).
2. Receives records `(key, value)` from the RecordReader.
3. Applies user-defined `map()` to each record to **filter, transform or extract** data.
4. Emits **intermediate `(key, value)` pairs** (zero, one or many per record).
5. Writes output to **local disk**, partitioned and sorted, for the reducers.

Key properties:
- Runs **in parallel** across nodes (no communication between mappers).
- Uses **data locality**: the scheduler tries to run it on the node that holds the split's block.
- Does **not** compute the final answer; it only prepares data for grouping.
- Number of map tasks = number of input splits (e.g. 1 GB / 128 MB = about 8 maps).

## 2.2 Example: yearly maximum temperature (weather data)
Input lines (`year,temp`):
```text
1990,30
1991,25
1990,35
```
Map function: parse the line, emit `(year, temp)`.
```text
Map output: (1990,30) (1991,25) (1990,35)
```
Mapper extracts only the needed fields and discards bad records. The reducer later takes `max()` per year: (1990,35), (1991,25). The mapper itself never sees all data for a year, so it cannot give the final maximum.

---

# 3. Shuffle & Sort Phase and the Reducer Phase (5M)

## 3.1 Shuffle and Sort (the bridge between map and reduce)

**Map side**
1. Map output goes into a circular **in-memory buffer** (default 100 MB, `mapreduce.task.io.sort.mb`).
2. At about 80% full, a background thread **partitions** by reducer, **sorts by key**, runs the **combiner** if set, and **spills** to local disk.
3. Spill files are **merged** into one partitioned, sorted output file per map task.

**Reduce side**
4. **Copy phase:** each reducer fetches its partition from every finished mapper (parallel HTTP fetches). This can start before all maps finish.
5. **Merge/Sort phase:** fetched segments are merged (in memory or disk) into one key-sorted stream.
6. **Grouping:** adjacent equal keys are grouped, giving `key → Iterable<values>`.

> The sort is by key only. Values within a key have no guaranteed order (unless secondary sort is used).

## 3.2 Reducer Phase
- Runs the user-defined `reduce(key, values)` **once per unique key**, in sorted key order.
- Aggregates, joins or summarises the values (sum, count, max, join, etc.).
- Emits `(key, result)` through the `OutputFormat` into HDFS; **one output file per reducer**.
- Number of reducers is set by the user (`job.setNumReduceTasks(n)`); each reducer handles only its assigned keys.
- The reduce function only starts after shuffle and sort are complete for its input.

```text
Mappers ──partitions──> [Shuffle: copy] → [Merge-Sort] → [Group] → reduce() → HDFS
```

Example: reducer gets `cat → [2,1]`, sums to `(cat, 3)`.

---

# 4. How Node Failures Are Handled in Hadoop (5M)

Hadoop is designed assuming commodity hardware **will** fail. Failures are detected by **heartbeats**.

| Failed element | Detection | Recovery |
|---|---|---|
| **DataNode** | No heartbeat for about 10.5 minutes (heartbeat every 3 s) | NameNode marks it dead, finds blocks that are under-replicated, and **re-replicates** them from surviving copies to other DataNodes. Clients read from other replicas meanwhile. |
| **Map/Reduce task** (task crash, JVM crash) | AM notices container exit or timeout | Task is **re-run** on another node. Default **4 attempts**; if all fail, the job fails. |
| **NodeManager / worker node** | RM misses heartbeats (about 10 min) | Running containers are lost; AM reschedules those tasks elsewhere. **Completed map tasks** on that node are also **re-executed**, because their output is on that node's local disk. Completed reduce tasks are not (output is in HDFS). |
| **ApplicationMaster** | RM detects it | RM restarts the AM (default 2 attempts) and it recovers the job state. |
| **ResourceManager** | Single master | **RM High Availability** (active/standby with ZooKeeper). |
| **NameNode** | Single master | **HDFS HA**: Standby NameNode via JournalNodes and automatic failover. |
| **Slow node (straggler)** | Task progress much slower than others | **Speculative execution**: a duplicate task is started on another node; whichever finishes first is kept, the other is killed. |

Summary: **storage failures are handled by replication, processing failures by task re-execution**.


---
tags:
  - mapreduce
  - wordcount
  - sem7
  - exam
aliases:
  - Word Count MapReduce
---

# Word Count Problem using MapReduce

> [!abstract] Topic frequency
> Asked 4 times. The pseudocode is the same every time, so it is written once. Traces are given for all three documents that appeared, and any other document is handled the same way.

---

# 1. Problem
Count how many times each distinct word occurs in a document.

Pre-processing rule used here: **convert to lower case and remove punctuation** (so "Apple" and "apple" are the same word). If a question wants case-sensitive counting, skip the lower-casing.

# 2. Pseudocode

```text
// MAPPER
map(key, value):
    // key   = byte offset of the line (ignored)
    // value = text of one line
    for each word w in tokenize(value):          // split on spaces, strip punctuation, lower-case
        Emit(w, 1)

// COMBINER (optional, same logic as reducer)
combine(word, counts):
    Emit(word, sum(counts))

// REDUCER
reduce(key, values):
    // key    = a word
    // values = list of counts for that word, e.g. [1,1,1]
    total = 0
    for each v in values:
        total = total + v
    Emit(key, total)
```

Java-style mapper core, if asked:
```java
public void map(LongWritable key, Text value, Context ctx) {
    StringTokenizer it = new StringTokenizer(value.toString().toLowerCase().replaceAll("[^a-z0-9 ]", ""));
    while (it.hasMoreTokens()) { word.set(it.nextToken()); ctx.write(word, ONE); }
}
public void reduce(Text key, Iterable<IntWritable> vals, Context ctx) {
    int sum = 0; for (IntWritable v : vals) sum += v.get();
    ctx.write(key, new IntWritable(sum));
}
```

# 3. Steps (general)
1. **Split:** the document is divided into splits (assume each sentence is one split, so 2 mappers).
2. **Map:** every word becomes `(word, 1)`.
3. **(Combine):** local sum per mapper.
4. **Shuffle and sort:** pairs are sent to reducers, sorted by key and grouped: `word → [1,1,...]`.
5. **Reduce:** values are summed.
6. **Output:** `(word, total)` written to HDFS.

---

# 4. Trace 1: "This is an apple. Apple is red in color"

| Stage | Result |
|---|---|
| Split 1 / Split 2 | `This is an apple` / `Apple is red in color` |
| **Map 1** | (this,1) (is,1) (an,1) (apple,1) |
| **Map 2** | (apple,1) (is,1) (red,1) (in,1) (color,1) |
| Combiner | Mapper 1 and 2 have no repeated words, so nothing changes |
| **Shuffle + Sort + Group** | an→[1], apple→[1,1], color→[1], in→[1], is→[1,1], red→[1], this→[1] |
| **Reduce** | sum each list |

**Final output**
```text
an 1
apple 2
color 1
in 1
is 2
red 1
this 1
```
(If case-sensitive: `Apple 1` and `apple 1` would be separate.)

---

# 5. Trace 2: "This is NoSQL. NoSQL handles complex data."

| Stage | Result |
|---|---|
| Split 1 / Split 2 | `This is NoSQL` / `NoSQL handles complex data` |
| **Map 1** | (this,1) (is,1) (nosql,1) |
| **Map 2** | (nosql,1) (handles,1) (complex,1) (data,1) |
| **Shuffle + Sort + Group** | complex→[1], data→[1], handles→[1], is→[1], nosql→[1,1], this→[1] |

**Final output**
```text
complex 1
data 1
handles 1
is 1
nosql 2
this 1
```

---

# 6. Trace 3: "Big data is powerful. Big data drives decisions."

| Stage | Result |
|---|---|
| Split 1 / Split 2 | `Big data is powerful` / `Big data drives decisions` |
| **Map 1** | (big,1) (data,1) (is,1) (powerful,1) |
| **Map 2** | (big,1) (data,1) (drives,1) (decisions,1) |
| **Shuffle + Sort + Group** | big→[1,1], data→[1,1], decisions→[1], drives→[1], is→[1], powerful→[1] |

**Final output**
```text
big 2
data 2
decisions 1
drives 1
is 1
powerful 1
```

---

# 7. Diagram to draw in the exam

```text
 Document
    ↓ split
 [Sentence 1]        [Sentence 2]
     ↓                    ↓
  Mapper 1             Mapper 2
 (w,1) pairs          (w,1) pairs
      └──── Shuffle & Sort ────┘
          (group by word)
                ↓
             Reducer
        (sum list of 1's)
                ↓
          (word, count)
```


---
tags:
  - mapreduce
  - matrix
  - sem7
  - exam
aliases:
  - Matrix Multiplication MapReduce
---

# Matrix Multiplication using MapReduce

> [!abstract] Topic frequency
> Asked 2 times (once with 2x2 matrices, once with 3x3 × 3x2). One pseudocode covers both. Both traces are given.

---

# 1. Idea
For `P = M × N`, where M is `m × n` and N is `n × p`:

```text
P[i][k] = Σ over j  ( M[i][j] × N[j][k] ),   j = 1..n
```

Each output cell `(i,k)` needs **row i of M** and **column k of N**. So the **key is the output cell `(i,k)`**.

# 2. Single-pass algorithm (pseudocode)

```text
// MAPPER
// Input records are in the form: (matrixName, row, col, value)

map(record):
    if record is from M:                 // element M[i][j] = mij
        for k = 1 to p:                  // p = number of columns of N
            Emit( (i, k), ("M", j, mij) )

    if record is from N:                 // element N[j][k] = njk
        for i = 1 to m:                  // m = number of rows of M
            Emit( (i, k), ("N", j, njk) )

// REDUCER
reduce( key=(i,k), values ):
    // values = list of ("M", j, mij) and ("N", j, njk)
    create arrays Mlist[j] and Nlist[j]
    for each v in values:
        if v.matrix == "M": Mlist[v.j] = v.value
        else:               Nlist[v.j] = v.value
    sum = 0
    for j = 1 to n:
        sum = sum + Mlist[j] * Nlist[j]
    Emit( (i,k), sum )
```

**Why it works:**
- Each M element is needed for every column of N, so it is **replicated p times**.
- Each N element is needed for every row of M, so it is **replicated m times**.
- The tag `"M"`/`"N"` and the index `j` let the reducer pair the right elements.

---

# 3. Example A: 2×2

$$M=\begin{bmatrix}1&2\\3&4\end{bmatrix},\quad N=\begin{bmatrix}5&6\\7&8\end{bmatrix}$$

m = 2, n = 2, p = 2.

## Step 1: Map

**From M** (each emitted for k = 1, 2):
```text
M11=1: ((1,1),(M,1,1)) ((1,2),(M,1,1))
M12=2: ((1,1),(M,2,2)) ((1,2),(M,2,2))
M21=3: ((2,1),(M,1,3)) ((2,2),(M,1,3))
M22=4: ((2,1),(M,2,4)) ((2,2),(M,2,4))
```
**From N** (each emitted for i = 1, 2):
```text
N11=5: ((1,1),(N,1,5)) ((2,1),(N,1,5))
N12=6: ((1,2),(N,1,6)) ((2,2),(N,1,6))
N21=7: ((1,1),(N,2,7)) ((2,1),(N,2,7))
N22=8: ((1,2),(N,2,8)) ((2,2),(N,2,8))
```

## Step 2: Shuffle and group by key

| Key (i,k) | Values |
|---|---|
| (1,1) | (M,1,1) (M,2,2) (N,1,5) (N,2,7) |
| (1,2) | (M,1,1) (M,2,2) (N,1,6) (N,2,8) |
| (2,1) | (M,1,3) (M,2,4) (N,1,5) (N,2,7) |
| (2,2) | (M,1,3) (M,2,4) (N,1,6) (N,2,8) |

## Step 3: Reduce (multiply matching j, then add)

| Key | Calculation | Result |
|---|---|---|
| (1,1) | 1×5 + 2×7 | **19** |
| (1,2) | 1×6 + 2×8 | **22** |
| (2,1) | 3×5 + 4×7 | **43** |
| (2,2) | 3×6 + 4×8 | **50** |

$$M\times N=\begin{bmatrix}19&22\\43&50\end{bmatrix}$$

---

# 4. Example B: 3×3 × 3×2

$$M=\begin{bmatrix}1&2&3\\4&5&6\\7&8&9\end{bmatrix},\quad N=\begin{bmatrix}1&2\\1&2\\2&2\end{bmatrix}$$

m = 3, n = 3, p = 2. Result is **3×2**, so there are **6 reduce keys**: (1,1) (1,2) (2,1) (2,2) (3,1) (3,2).

## Step 1: Map
- Each of the 9 M elements is emitted **p = 2 times** (18 pairs). Example: `M12=2 → ((1,1),(M,2,2)) ((1,2),(M,2,2))`.
- Each of the 6 N elements is emitted **m = 3 times** (18 pairs). Example: `N21=1 → ((1,1),(N,2,1)) ((2,1),(N,2,1)) ((3,1),(N,2,1))`.

## Step 2: Shuffle, grouping (j-wise values for each key)

| Key | M values (j: value) | N values (j: value) |
|---|---|---|
| (1,1) | 1:1, 2:2, 3:3 | 1:1, 2:1, 3:2 |
| (1,2) | 1:1, 2:2, 3:3 | 1:2, 2:2, 3:2 |
| (2,1) | 1:4, 2:5, 3:6 | 1:1, 2:1, 3:2 |
| (2,2) | 1:4, 2:5, 3:6 | 1:2, 2:2, 3:2 |
| (3,1) | 1:7, 2:8, 3:9 | 1:1, 2:1, 3:2 |
| (3,2) | 1:7, 2:8, 3:9 | 1:2, 2:2, 3:2 |

## Step 3: Reduce

| Key | Calculation | Result |
|---|---|---|
| (1,1) | 1×1 + 2×1 + 3×2 = 1+2+6 | **9** |
| (1,2) | 1×2 + 2×2 + 3×2 = 2+4+6 | **12** |
| (2,1) | 4×1 + 5×1 + 6×2 = 4+5+12 | **21** |
| (2,2) | 4×2 + 5×2 + 6×2 = 8+10+12 | **30** |
| (3,1) | 7×1 + 8×1 + 9×2 = 7+8+18 | **33** |
| (3,2) | 7×2 + 8×2 + 9×2 = 14+16+18 | **48** |

$$M\times N=\begin{bmatrix}9&12\\21&30\\33&48\end{bmatrix}$$

---

# 5. Notes
- Output size is 3×2 (rows of M × columns of N).
- Cost: map emits `m·n·p·2` pairs here, so communication is heavy; there is also a **two-pass** version (pass 1 joins on `j` and multiplies, pass 2 groups by `(i,k)` and sums) that reduces the reducer memory need.
- Mapper must know m and p (passed through job configuration).

```text
Matrix elements → MAP (replicate with key (i,k)) → SHUFFLE (group by cell)
→ REDUCE (Σ M[i][j]·N[j][k]) → result cell
```


---
tags:
  - mapreduce
  - relational-algebra
  - sem7
  - exam
aliases:
  - Relational Algebra MapReduce
---

# Relational-Algebra Operations using MapReduce

> [!abstract] Topic frequency
> Asked 3 times, as two distinct questions answered below: (A) Selection and Projection, (B) Natural Join and Grouping and Aggregation. The sample relation tables are shared across answers.

A relation is a table; a tuple is a row. Each tuple is processed as a `(key, value)` record.

**Sample relations used**
```text
Employee(EmpID, Name, Dept, Salary)
  1, Asha, HR, 30000
  2, Ravi, IT, 50000
  3, Meena, IT, 60000
  4, John, HR, 40000
  5, Sara, IT, 50000
```

---

# A. Selection and Projection (10M)

## A.1 Selection σ_C(R)
Selects the tuples that satisfy condition C.

```text
map(tuple t):
    if t satisfies C:
        Emit(t, t)

reduce(key t, values):
    Emit(t, t)            // identity: just pass the tuple through
```
- It is essentially **map-only** (reducer is identity, so `numReduceTasks = 0` is possible).
- No grouping or sorting is needed because each tuple is tested independently.

**Example:** σ_{Dept = 'IT'}(Employee)
```text
Map keeps: (2,Ravi,IT,50000) (3,Meena,IT,60000) (5,Sara,IT,50000)
Map drops: Asha and John (HR)
Output:    the three IT tuples
```

## A.2 Projection π_S(R)
Keeps only the attributes in set S **and removes duplicates** (relational algebra uses set semantics).

```text
map(tuple t):
    t' = t with only attributes in S
    Emit(t', t')

reduce(key t', values [t', t', ...]):
    Emit(t', t')          // emit ONCE, which eliminates duplicates
```
- The **reducer eliminates duplicates**: all identical projected tuples reach the same reducer via the same key.
- Without duplicate elimination (bag semantics) it can be map-only.

**Example:** π_{Dept, Salary}(Employee)
```text
Map:      (HR,30000) (IT,50000) (IT,60000) (HR,40000) (IT,50000)
Shuffle:  (HR,30000)→[..]  (HR,40000)→[..]  (IT,50000)→[.., ..]  (IT,60000)→[..]
Reduce:   one copy of each key
Output:   (HR,30000) (HR,40000) (IT,50000) (IT,60000)     // (IT,50000) appears once
```

| Operation | Map | Reduce | Duplicates? |
|---|---|---|---|
| Selection | Test condition, emit tuple | Identity | None created |
| Projection | Cut attributes, emit | Emit one per key | Removed in reducer |

---

# B. Natural Join and Grouping and Aggregation (10M)

## B.1 Natural Join R(A,B) ⋈ S(B,C)
Join tuples of R and S on the **common attribute B**. The **join attribute B is the key**, so tuples with the same B meet at the same reducer.

```text
map(tuple from R = (a, b)):   Emit(b, ("R", a))
map(tuple from S = (b, c)):   Emit(b, ("S", c))

reduce(key b, values):
    Rlist = [a for ("R", a) in values]
    Slist = [c for ("S", c) in values]
    for each a in Rlist:
        for each c in Slist:
            Emit(a, b, c)          // output joined tuple (a,b,c)
```
The tag `"R"`/`"S"` records which relation each value came from.

**Example**
```text
R(Student, CourseID)      S(CourseID, CourseName)
  Amit, C1                  C1, DBMS
  Riya, C1                  C2, Networks
  Sam,  C2
```
Map:
```text
R: (C1,(R,Amit)) (C1,(R,Riya)) (C2,(R,Sam))
S: (C1,(S,DBMS)) (C2,(S,Networks))
```
Shuffle (group by CourseID):
```text
C1 → [(R,Amit) (R,Riya) (S,DBMS)]
C2 → [(R,Sam) (S,Networks)]
```
Reduce (cross product of the R-list and S-list per key):
```text
C1: Amit-DBMS, Riya-DBMS  → (Amit,C1,DBMS) (Riya,C1,DBMS)
C2: Sam-Networks          → (Sam,C2,Networks)
```
This is a **reduce-side join**. A key present in only one relation yields no output.

## B.2 Grouping and Aggregation γ_{A, θ(B)}(R)
Group tuples by attribute(s) A and apply an aggregate θ (SUM, COUNT, AVG, MAX, MIN) on attribute B.

```text
map(tuple t = (a, b, ...)):
    Emit(a, b)                    // key = grouping attribute, value = attribute to aggregate

reduce(key a, values [b1, b2, ...]):
    result = θ(values)            // e.g. sum, count, avg, max
    Emit(a, result)
```

**Example:** γ_{Dept, SUM(Salary)}(Employee) (and AVG)
```text
Map:      (HR,30000) (IT,50000) (IT,60000) (HR,40000) (IT,50000)
Shuffle:  HR → [30000, 40000]
          IT → [50000, 60000, 50000]
Reduce SUM:  (HR, 70000)   (IT, 160000)
Reduce AVG:  (HR, 35000)   (IT, 53333.33)
```
- Grouping is done **automatically by shuffle and sort**; aggregation is done by the reducer.
- A **combiner** can pre-sum for SUM, COUNT, MAX and MIN. For AVG, emit `(sum, count)` pairs from the combiner, because the average of partial averages is wrong.

## B.3 Comparison

| Aspect       | Natural Join                                    | Grouping and Aggregation             |
| ------------ | ----------------------------------------------- | ------------------------------------ |
| Key          | Join attribute (B)                              | Grouping attribute (A)               |
| Map value    | Tagged tuple (relation name + other attributes) | Attribute to aggregate               |
| Reducer work | Pair every R-value with every S-value           | Apply aggregate function to the list |
| Inputs       | Two relations                                   | One relation                         |