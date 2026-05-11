# Distributed Data Processing
### Hiraya Solutions

---
## Video Presentation

[![Watch the Video Presentation](https://img.shields.io/badge/Google%20Drive-Watch%20Presentation-blue?logo=google-drive)](https://drive.google.com/drive/folders/1ZY7wLXmQYZ4kBkH3M8z9oa6SoJ8EIBGl?usp=sharing)

[Click here to watch the Final Project Video Presentation](https://drive.google.com/drive/folders/1ZY7wLXmQYZ4kBkH3M8z9oa6SoJ8EIBGl?usp=sharing)

---

## Project Overview

This project extends the Midterm outcome by migrating data processing to a distributed computing framework. Working from the same 400,000-row sales dataset (`synthetic_sales_data_400k.csv`), the system implements three core analytics operations -- filtering, aggregation, and sorting -- across three execution strategies:

- **Sequential** -- single-threaded, row-by-row processing using Python csv module
- **Parallel (Midterm)** -- multi-threaded processing using ThreadPoolExecutor
- **Distributed (Final)** -- distributed processing using Apache Spark (PySpark)

---

## Tools and Technologies Used

| Tool | Purpose |
| --- | --- |
| Python 3.12 | Primary programming language |
| csv module | Row-by-row data loading |
| ThreadPoolExecutor | Parallel execution across threads |
| multiprocessing | CPU core count for worker sizing |
| Apache Spark 4.1.1 (PySpark) | Distributed data processing framework |
| pyspark.sql.functions | Spark SQL operations |
| time | Execution time measurement |
| Jupyter Notebook | Interactive development environment |

---

## Instructions for Running the Project

1. Clone or download the repository and open it in VS Code
2. Install dependencies:
    ```
    pip install notebook pyspark
    ```
3. Open `FinalsIteration.ipynb` in VS Code or Jupyter
4. Run **Kernel -> Restart & Run All**
5. Observe the printed execution times for each operation

> The dataset file must be inside a directory named `assets` in the same directory as the notebook.

---

## Team Members

| Member Name | Role |
| --- | --- |
| Allan John Funelas | Programmer |
| Carl Angelo Hernandez | Researcher |
| Lucky Guevarra | Researcher |
| Zyrus Alvez | Documentation |


## Table of Contents
1. [Project Objective](#project-objective)
2. [System Architecture](#system-architecture)
3. [Implementation](#implementation)
4. [Performance Evaluation](#performance-evaluation)
5. [Scalability & Fault Tolerance](#scalability--fault-tolerance)
6. [Ethical & Professional Considerations](#ethical--professional-considerations)

---

## Project Objective

This project extends the Midterm outcome by migrating data processing to a distributed computing framework. Working from the same 400,000-row sales dataset (`synthetic_sales_data_400k.csv`), the system implements three core analytics operations — **filtering**, **aggregation**, and **sorting** — across three execution strategies:

- **Sequential** — single-threaded, row-by-row processing using Python's `csv` module
- **Parallel (Midterm)** — multi-threaded processing using `concurrent.futures.ThreadPoolExecutor`
- **Distributed (Final)** — distributed processing using Apache Spark (PySpark)

The goal is to compare execution times, analyze performance improvements, and reflect on the scalability and ethical implications of each approach.

---

## System Architecture

### Sequential
```
[ Python Process ]
       |
  [ csv.DictReader ] --> [ rows list (400k dicts) ]
       |
  [ Single Loop ] --> filter / aggregate / sort
```

### Parallel (ThreadPoolExecutor)
```
[ Python Process ]
       |
  [ rows list split into N chunks ]
       |
  ┌────┴────┐
  │ThreadPoolExecutor (N = cpu_count)│
  ├── Thread 1 --> chunk 1
  ├── Thread 2 --> chunk 2
  ├── Thread 3 --> chunk 3
  └── Thread N --> chunk N
       |
  [ Merge results in main thread ]
```

### Distributed (PySpark)
```
[ Driver Program ]
       |
  [ SparkContext ]
       |
  [ DAG Scheduler ] --> [ Task Scheduler ]
       |                        |
  [ Stage 1 ]          [ Executor 1 ] -- Partition A
  [ Stage 2 ]          [ Executor 2 ] -- Partition B
  [ Stage 3 ]          [ Executor N ] -- Partition N
       |
  [ Collect results to Driver ]
```

Spark transformations (`filter`, `groupBy`, `orderBy`) are **lazy** — they only build an execution plan. The plan is triggered and executed when an action (`count`, `show`) is called.

---

## Implementation

### Tools & Technologies

| Tool | Purpose |
|---|---|
| Python 3.12 | Primary programming language |
| `csv` module | Row-by-row data loading |
| `concurrent.futures.ThreadPoolExecutor` | Parallel execution across threads |
| `multiprocessing` | CPU core count for worker sizing |
| Apache Spark 4.1.1 (PySpark) | Distributed data processing framework |
| `pyspark.sql.functions` | Spark SQL operations (filter, groupBy, orderBy) |
| `time` | Execution time measurement |
| Jupyter Notebook | Interactive development environment |

### Dataset

- **File:** `synthetic_sales_data_400k.csv`
- **Rows:** 400,000 sales records
- **Key columns used:** `Sales_Amount`, `Product_Category`, `Sale_Date`

### Operations Implemented

**1. Filtering** — Select rows where `Sales_Amount > 1000`
- Sequential: list comprehension with `float()` conversion per row
- Parallel: same logic split across thread chunks, results flattened
- Spark: `df.filter(col('Sales_Amount') > 1000)`

**2. Aggregation** — Sum `Sales_Amount` grouped by `Product_Category`
- Sequential: dict accumulation loop
- Parallel: partial dicts per chunk, merged in main thread
- Spark: `df.groupBy('Product_Category').agg(spark_sum('Sales_Amount'))`

**3. Sorting** — Order all records by `Sale_Date` ascending
- Sequential: Python's built-in `sorted()` with a lambda key
- Parallel: each chunk sorted independently (chunk-level sort)
- Spark: `df.orderBy('Sale_Date')`

---

## Performance Evaluation

### Results (Last Recorded Run)

| Operation | Sequential (s) | Parallel/Thread (s) | Distributed/Spark (s) | Thread Speedup | Spark Speedup | Thread Efficiency | Spark Efficiency |
|---|---|---|---|---|---|---|---|
| Filtering | 0.4937 | 0.4623 | 3.4080 | 1.07x | 0.14x | 0.2670 | 0.0362 |
| Aggregation | 0.9506 | 0.6590 | 1.9439 | 1.44x | 0.49x | 0.3606 | 0.1223 |
| Sorting | 0.7782 | 0.8174 | 0.6258 | 0.95x | 1.24x | 0.2380 | 0.3109 |

> **Note:** Runtimes vary between runs depending on system load. Speedup = Sequential / Parallel. Efficiency = Speedup / Number of Workers (4 cores).

### Analysis

**Filtering**
Threading achieved a modest **1.07x speedup**. Filtering is a lightweight per-row operation — each row only requires a `float()` conversion and a comparison. The work per chunk is so small that thread coordination overhead consumes most of the potential gain. Spark was **7x slower** than sequential due to JVM startup, DAG compilation, and Py4J serialization overhead on a single local machine.

**Aggregation**
Threading achieved the best speedup of **1.44x**. Aggregation involves heavier per-row work (type conversion + dict lookup + addition), giving threads more meaningful computation to overlap. Spark was still slower locally (0.49x) but the gap narrowed compared to filtering, as Spark's `groupBy().agg()` benefits from partition-level pre-aggregation before merging.

**Sorting**
Threading was **slightly slower than sequential (0.95x)**. This is because each thread only sorts its own chunk — the result is multiple locally-sorted lists, not a globally sorted dataset. The thread overhead adds time without producing a correct global sort. Spark was the **fastest at 1.24x speedup** — its distributed sort algorithm sorts partitions in parallel using JVM-level optimizations, outperforming Python's single-threaded Timsort for this dataset size.

### Why Spark is Slower Locally

Running PySpark on a single local machine means all distributed overhead exists with none of the distributed benefits:

1. **JVM startup** — Spark runs on the JVM via Py4J, adding fixed startup cost
2. **Lazy evaluation** — transformations don't execute until an action is called, triggering full DAG compilation
3. **Py4J serialization** — data must be serialized between Python and the JVM for every operation
4. **Partition scheduling** — Spark schedules tasks across partitions even locally, adding overhead that doesn't exist in plain Python

On a **multi-node cluster with billions of rows**, these costs become negligible compared to the gains from true distributed parallel execution.

---

## Scalability & Fault Tolerance

### Scalability

| Method | Scalability | Limitation |
|---|---|---|
| Sequential | Low — O(n), time grows linearly with data | Single core, single machine |
| Parallel (Threading) | Medium — scales up to CPU core count | Limited to one machine's cores and RAM |
| Distributed (Spark) | High — scales out by adding nodes | Network overhead; overkill for small datasets |

- **Sequential** doubles in time when the dataset doubles.
- **Parallel** is bounded by the number of physical cores on one machine. Beyond that, adding more threads causes contention.
- **Spark** scales horizontally — adding worker nodes increases throughput proportionally. Each node handles only a partition of the data, so total time stays roughly constant as both data and nodes grow together.

### Fault Tolerance

| Method | Fault Tolerance |
|---|---|
| Sequential | None — a crash requires a full restart |
| Parallel (Threading) | None — a crashed thread loses its chunk result |
| Distributed (Spark) | Built-in — uses RDD lineage (DAG) to re-compute failed partitions |

Spark's fault tolerance works through **RDD lineage**: every transformation is recorded in the DAG. If a partition fails, Spark re-computes only that partition from its last known checkpoint — not the entire dataset. In cluster mode, the driver can also automatically restart failed executors.

---

## Ethical & Professional Considerations

### Data Privacy

The dataset contains sales records with fields such as `Sales_Rep`, `Region`, `Customer_Type`, and `Payment_Method`. While no direct personal identifiers (e.g., names, IDs) are present, combining multiple attributes can potentially re-identify individuals — a risk known as the **mosaic effect**.

- In a production system, sensitive fields should be **role-restricted** and **access-logged**
- Distributed systems increase the attack surface since data is replicated across partitions and nodes — **encryption at rest and in transit (TLS)** is essential
- Any cloud deployment (e.g., Google Colab, AWS EMR) must comply with relevant data protection regulations (GDPR, local laws)

### Responsible Data Usage

- Analytical results such as sales aggregations by region or representative must not be used to **unfairly evaluate or discriminate** against individuals without proper context
- Dashboards or models built on this data should be regularly **audited for bias** — for example, checking whether performance metrics are consistent across regions or sales channels
- Data should only be retained for as long as necessary and deleted securely afterwards (**data minimisation principle**)

### Professional Responsibility

- Engineers deploying distributed pipelines must **document data flows**, access controls, and retention policies
- Results should be communicated **transparently**, including the limitations of the analysis and the conditions under which benchmarks were measured (e.g., single local machine vs. cluster)
- Automated decisions based on this data should always include **human oversight**, especially for high-stakes outcomes


---


# Technical Report: Parallel CSV Data Processing

## Problem Description

The goal of this project is to process a large dataset of 400,000 sales records (`synthetic_sales_data_400k.csv`) efficiently using parallel computing techniques. The dataset contains columns such as `Sales_Amount`, `Product_Category`, and `Sale_Date`. Three core operations are performed on the data:

- **Filtering** - selecting rows where `Sales_Amount > 1000`
- **Grouping/Aggregation** - summing `Sales_Amount` per `Product_Category`
- **Sorting** - ordering all records by `Sale_Date`

The challenge is that processing 400,000 rows sequentially in pure Python is slow due to row-by-row iteration, and the goal is to demonstrate a measurable speedup using parallelization.

## Parallelization Approach

The `csv` module was chosen over `pandas` as the data processing library. While `pandas` uses highly optimized C-level vectorized operations that are already fast, the `csv` module processes data row-by-row in pure Python, making it CPU-bound and a better candidate for demonstrating parallel speedup.

The parallelization strategy used is **data parallelism**, the dataset is split into equal-sized chunks, and each chunk is processed independently by a worker thread. The implementation uses Python's `concurrent.futures.ThreadPoolExecutor`:

```python
def parallel_operation(data, func):
    num_workers = multiprocessing.cpu_count()
    size = len(data) // num_workers
    chunks = [data[i*size:(i+1)*size] for i in range(num_workers)]
    with ThreadPoolExecutor(max_workers=num_workers) as executor:
        results = list(executor.map(func, chunks))
    return results
```

Each chunk function (`filter_chunk`, `agg_chunk`, `sort_chunk`) operates on its assigned slice of rows and returns a partial result. The results are then merged in the main thread.

## Performance Analysis

| Operation | Sequential (s) | Parallel (s) | Speedup | Efficiency |
|---|---|---|---|---|
| Filtering | 0.5210 | 0.2461 | 2.12x | 0.5293 |
| Aggregation | 0.2800 | 0.2320 | 1.21x | 0.3018 |
| Sorting | 0.5110 | 0.7100 | 0.72x (slower) | 0.1799 |

> **Note:** Runtimes vary between runs depending on system load. The table above reflects the last recorded run.

Filtering showed the most notable speedup (2.12x), as each chunk independently evaluates a condition per row with no inter-chunk dependency. Aggregation saw a modest gain (1.21x) since merging partial dicts after threading adds some overhead. Sorting was **slower in parallel** (0.72x), this is expected because sorting requires a globally ordered result, meaning chunk-level sorting is only a partial sort. The overhead of sorting each chunk separately plus thread coordination outweighed any gains, making sequential sorting faster for this dataset size.

## Challenges Encountered

**1. pandas vs csv module**
Initially, `pandas` was used for all operations. Upon testing, the parallel execution showed no measurable speedup over sequential, in some cases it was even slower. After researching, this was found to be caused by pandas' built-in methods being already highly optimized at the C level, meaning the operations completed so fast that the overhead of splitting and recombining data exceeded any potential gains from parallelism. Switching to the `csv` module, which processes data row-by-row in pure Python, introduced enough per-row work to make the parallel speedup noticeable.

**2. Finding the right parallelism approach**
Even after switching to the `csv` module, achieving a shorter parallel runtime than sequential required testing multiple parallelism libraries. `multiprocessing.Pool` and `ProcessPoolExecutor` were tried first but produced worse runtimes than sequential due to process spawn overhead and the cost of serializing 400k rows across process boundaries on Windows. `ipyparallel` was also explored but required a running `ipcluster` engine cluster as a separate process, adding setup complexity with no runtime benefit. `ThreadPoolExecutor` ultimately proved to be the best fit, as threads share memory with no serialization cost and overlap well with the I/O-like overhead of the `csv` module's row-by-row processing.


---

# Results Explanation: FinalsIteration.ipynb

## Performance Summary (Actual Run)

| Operation   | Sequential (s) | Parallel/Thread (s) | Distributed/Spark (s) | Thread Speedup | Spark Speedup | Thread Efficiency | Spark Efficiency |
|---|---|---|---|---|---|---|---|
| Filtering   | 0.4937 | 0.4623 | 3.4080 | 1.0679 | 0.1449 | 0.2670 | 0.0362 |
| Aggregation | 0.9506 | 0.6590 | 1.9439 | 1.4425 | 0.4890 | 0.3606 | 0.1223 |
| Sorting     | 0.7782 | 0.8174 | 0.6258 | 0.9520 | 1.2435 | 0.2380 | 0.3109 |

---

## Filtering

**Sequential: 0.4937s | Parallel: 0.4623s | Spark: 3.4080s**

- The sequential filter iterates all 400,000 rows one by one, calling `float()` on each `Sales_Amount` value. This takes ~0.49s of pure Python work.
- The parallel version splits the rows across threads and runs the same loop concurrently, giving a small but real speedup of **1.07x**. The gain is modest because filtering is a lightweight per-row operation — the thread coordination overhead eats into the savings.
- Spark took **3.41s**, which is ~7x slower than sequential. This is because Spark's `filter()` is a **lazy transformation** — it doesn't execute until `count()` is called. At that point, Spark has to: build a DAG execution plan, serialize the query, schedule tasks across partitions, and collect results back to the driver. On a single local machine with no cluster, this overhead completely dominates the actual computation time.

---

## Aggregation

**Sequential: 0.9506s | Parallel: 0.6590s | Spark: 1.9439s**

- Sequential aggregation loops through all 400,000 rows, doing a `float()` conversion and a dict lookup/update per row. This is the heaviest sequential operation at ~0.95s because it involves more work per row than filtering.
- The parallel version achieves the best thread speedup of **1.44x** — each thread aggregates its own chunk into a partial dict, then the main thread merges them. Because the per-row work is heavier (float conversion + dict update), threads have more meaningful work to overlap, making parallelism more effective here than in filtering.
- Spark took **1.94s** — still slower than both sequential and parallel, but much closer than filtering. Spark's `groupBy().agg()` is well-optimized internally and benefits from partition-level aggregation before shuffling results. The overhead is still significant on a single machine, but the gap narrows because aggregation is inherently more compute-intensive.

---

## Sorting

**Sequential: 0.7782s | Parallel: 0.8174s | Spark: 0.6258s**

- Sequential sorting uses Python's built-in `sorted()` with a lambda key on `Sale_Date`. Python's Timsort is highly optimized and sorts all 400,000 rows in ~0.78s.
- The parallel version is actually **slightly slower (0.95x)** than sequential. This is expected — each thread sorts only its own chunk, producing multiple locally-sorted lists that are **not globally sorted**. The result is not a fully sorted dataset, just sorted-within-chunks. The thread overhead adds time without producing a correct global sort, making this the worst case for threading.
- Spark is the **fastest at 0.63s** with a speedup of **1.24x** over sequential. Spark's `orderBy()` uses a distributed sort algorithm that partitions data, sorts each partition in parallel, and merges results efficiently. Even on a single machine, Spark's internal sort implementation outperforms Python's single-threaded sort for this dataset size because it leverages JVM-level optimizations and parallel partition sorting.

---

## Why Spark Appears Slower Overall (Except Sorting)

Running PySpark locally on a single machine means all the distributed computing overhead exists with none of the distributed computing benefits:

1. **JVM startup** — Spark runs on the JVM via Py4J. Starting the JVM and the SparkContext adds fixed overhead to every run.
2. **Lazy evaluation** — Spark transformations (`filter`, `groupBy`, `orderBy`) don't execute until an action (`count`, `show`) is called. The first action triggers the full DAG compilation and optimization.
3. **Serialization** — Data must be serialized between Python and the JVM via Py4J for every operation.
4. **Partition overhead** — Spark splits data into partitions and schedules tasks even when running locally, adding scheduling cost that doesn't exist in plain Python.

On a **multi-node cluster** with millions of rows, these costs become negligible compared to the gains from true distributed parallel execution across many machines.

---

## Efficiency Values Explained

Efficiency = Speedup / Number of Workers

With 4 CPU cores (`num_workers = 4`):

- **Thread Filtering efficiency: 0.2670** — means each thread is only 26.7% as productive as it would be in a perfectly parallel system. The low per-row work makes thread overhead significant.
- **Thread Aggregation efficiency: 0.3606** — better at 36%, because heavier per-row work means threads spend more time doing useful computation relative to coordination overhead.
- **Thread Sorting efficiency: 0.2380** — low because parallel chunk sorting doesn't produce a globally correct result, so the work done is partially wasted.
- **Spark efficiencies are all low (0.03–0.31)** on a local machine because the overhead of the distributed framework outweighs the computation for a 400k-row dataset. These numbers would improve significantly on a real cluster with larger data.
