---
title: DuckDB
icon: simple/duckdb
full_width: true
---

DuckDB is an open-source column-oriented relational database management system (RDBMS). It is designed to provide high performance on complex queries against large databases. Unlike other embedded databases (for example, SQLite) DuckDB is not focusing on transactional (OLTP) applications and instead is specialized for online analytical processing (OLAP) workloads.

- it has bindings to the many languages (Python, Rust, Go...)
- query parser derived from PostgreSQL
- geometry type
- can read Delta table or iceberg
- remove files (cloud storages)
- ducklake: lakehouse format
    - ACID
