# Design Patterns 17: Data Optimization

Optimizing how data is stored and transmitted.

## 1. Encoding
*   **Text (JSON/XML)**: Human readable, but large payload. Universal compatibility.
*   **Binary (Protobuf/MsgPack)**: Not human readable, very small. Fast to parse.

## 2. Compaction
Removing redundant data.
*   **Example**: Storing "Columnar Data" (Parquet users this) instead of Row-based.
*   **Run-Length Encoding**: `AAAAABBB` -> `5A3B`.
