
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
