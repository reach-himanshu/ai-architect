# Design Patterns 12: ETL Pipelines

ETL (Extract, Transform, Load) is the backbone of Data Engineering.

## 1. Extract
Reading data from a source (Database, API, File).
*   **Challenges**: Network failures, Schema changes, Rate limits.

## 2. Transform
Processing and Cleaning the data.
*   **Operations**: Filtering, Joining, Aggregating, validating types, removing PII (Personally Identifiable Information).

## 3. Load
Writing the data to a destination (Data Warehouse, Data Lake).
*   **Modes**: Full overwrite vs Incremental append.

## 4. Pipeline Design
Instead of one giant function, we chain small, functional steps. This makes it easier to test and debug (similar to the "Pipe and Filter" integration pattern).
