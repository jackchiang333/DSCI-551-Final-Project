# Quantitative Trade Audit System
**DSCI 551 - Course Project**
**Author:** Jack Chiang

## Project Overview
This project is a Quantitative Trade Audit System designed to demonstrate the internal architecture of SQLite, specifically focusing on Storage, Indexing, and the Virtual Database Engine (VDBE). The application simulates a compliance auditing environment where analysts need to quickly locate specific historical trades from a large dataset. 

By running this application, users can directly observe the performance mapping between a database's internal execution plan (Full Table Scan vs. B-Tree Traversal) and the application's external latency ($O(N)$ vs $O(\log N)$ complexity).

---

## 1. Set up the environment
This project is designed to be highly portable and relies entirely on the Python Standard Library.
* Ensure you have **Python 3.7 or higher** installed on your system.
* Clone or download this repository to your local machine.
* Ensure the primary script, `app.py`, is located in your current working directory.

## 2. Install dependencies
There are no external dependencies required for this project. 

* The application natively imports `sqlite3`, `time`, `os`, and `random`, all of which are included in the standard Python installation.

## 3. Configure the project
**No manual configuration, secret keys, or API tokens are required.**

**Data and Dataset Configuration:** To ensure seamless reproducibility and avoid large file uploads, this project utilizes an automated synthetic data pipeline. 
* You do **not** need to download or import any external CSV or dataset files.
* Upon execution, the `setup_database()` function automatically creates a local SQLite file named `audit_tool.db`.
* It dynamically generates **100,000 synthetic trade records** and inserts them into the database before the interactive prompt begins. 
* Note: If `audit_tool.db` already exists from a previous run, the script safely deletes and rebuilds it to ensure a clean testing environment and prevent "Index Already Exists" errors.

## 4. Run the application
Open your terminal or command prompt, navigate to the folder containing the repository, and execute the following command:
`python app.py`

## 5. Reproduce your results
The application runs as a fully interactive command-line interface. Follow the on-screen prompts to reproduce the performance results demonstrating the mapping between database internals and application behavior:

**Phase 1: Unindexed (Full Table Scans)**
1. The application will prompt you for inputs. Enter a ticker (e.g., `AAPL`), a single date (e.g., `2026-03-15`), a start date (e.g., `2026-03-01`), and an end date (e.g., `2026-03-07`).
2. The system will execute three operations (Point Lookup, Range Scan, and Top-K Sort) without an index.
3. **Observe the Internals:** Note that the `EXPLAIN QUERY PLAN` outputs a `SCAN` operation, and the VDBE Bytecode relies on the `Rewind` opcode (forcing a full table scan) or `SorterOpen` (forcing a memory sort).
4. **Observe the Latency:** Note the execution time for the 100-query stress tests (typically ~0.5 - 1.5 seconds).

**Phase 2: Building the Index**
1. Press `Enter` when prompted to construct the composite B-Tree index on `(ticker, execution_date)`. 

**Phase 3: Indexed (B-Tree Traversal)**
1. The system will automatically re-run the exact same three operations using the newly built index.
2. **Observe the Internals:** Note the internal plan shifts to `SEARCH ... USING INDEX`. The bytecode shifts to `IdxGE` (Index Greater or Equal), proving the engine is now traversing the B-tree branches instead of scanning rows.
3. **Observe the Latency:** Note that the 100-query stress tests now execute in a fraction of a millisecond, demonstrating the logarithmic scaling advantage of the B-tree architecture.
