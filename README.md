[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=23394499&assignment_repo_type=AssignmentRepo)

# Parallel CSV Data Processing

## Project Overview

This project demonstrates the effect of parallel computing on large-scale data processing using a dataset of 400,000 sales records (`synthetic_sales_data_400k.csv`). Three core operations are performed on the data. These are filtering, aggregation, and sorting. First performed sequentially, then in parallel, with execution times compared to measure the speedup gained from parallelization.

The project intentionally uses Python's `csv` module instead of `pandas` to process data row-by-row, making the workload CPU-bound enough for parallel computing to show a measurable improvement.

## Tools and Technologies Used

- **Python 3.12** — primary programming language
- **Jupyter Notebook** — interactive development environment
- **csv** — standard library module used for row-by-row data processing
- **concurrent.futures.ThreadPoolExecutor** — used for parallel execution across threads
- **multiprocessing** — used to determine the number of CPU cores for worker count
- **time** — used for measuring sequential and parallel execution times

## Instructions for Running the Project

1. Clone or download the repository and open it in VS Code

2. Ensure Python 3 is installed, then install Jupyter if needed:
    ```
    pip install notebook
    ```

3. Open `MidtermIteration1.ipynb` in VS Code or Jupyter

4. Run **Kernel → Restart & Run All** to execute all cells from top to bottom

5. Observe the printed execution times for each sequential and parallel operation

> The dataset file `synthetic_sales_data_400k.csv` must be in the a directory named `assets`, and this folder is on the same directory as `midterm.ipynb` for the notebook to run correctly.

For the full technical report including problem description, parallelization approach, performance analysis, and challenges encountered, see [`report.md`](report.md).

### Members
| Member Name | Role |
| --- | --- |
| Allan John Funelas | Programmer |
| Carl Angelo Hernandez | Researcher |
| Lucky Guevarra | Researcher |
| Zyrus Alvez | Documentation |
