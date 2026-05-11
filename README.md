# Log-DB-Bench: Dynamic Log Acquisition & Analysis Framework

Benchmark and comparative analysis of SQL (PostgreSQL, MySQL), NoSQL (MongoDB), and OLAP (DuckDB) databases for handling dynamic, semi-structured log data.

## Overview
Modern systems generate logs with unpredictable, evolving structures. This project evaluates how different database engines handle "schema-on-read" vs "schema-on-write" approaches in the context of data acquisition and AI readiness (feature engineering).

### Tested Systems:
- **PostgreSQL**: Using `JSONB` for binary-structured data and GIN indexing.
- **MongoDB**: Native BSON document storage for maximum write throughput.
- **DuckDB**: In-process OLAP engine for fast analytical queries on JSON-based log data.
- **MySQL/MariaDB**: Using the native `JSON` data type.

## System Architecture
The system is built as a pipeline consisting of three main modules:
1. **Synthetic Log Generator**: Produces logs with variable schemas (from 10 to 35 fields per record).
2. **Acquisition Layer**: A Python-based intermediary that validates and dispatches data sequentially to multiple databases.
3. **Analysis Engine**: Executes aggregation benchmarks (temporal, statistical, and categorical) across all platforms.

## Implementation Details
### The core of the project is a Python generator that simulates real-world system logs. It produces records with a mandatory base and a variable number of extra telemetry fields
import random
from datetime import datetime

def generate_dynamic_log():
    # Base mandatory fields
    log = {
        "timestamp": datetime.utcnow().isoformat(),
        "eventType": random.choice(["info", "warning", "error"]),
        "userId": random.randint(1, 1000),
        "latency": random.randint(10, 500),
        "device": random.choice(["Android", "iOS", "Web"])
    }
    
    # Inject 5 to 30 dynamic extra fields
    for i in range(random.randint(5, 30)):
        log[f"extraField_{i}"] = random.random()
        
    return log

### Multi-Engine Acquisition Layer
The system dispatches the same JSON payload to all databases in parallel to ensure benchmark consistency.
- PostgreSQL (JSONB): Uses the JSONB type for efficient binary storage and indexing.
- MongoDB (NoSQL): Stores data as native BSON documents without a predefined schema.
- DuckDB (OLAP): Ingests raw JSON data for high-speed analytical processing.

### Cross-Database Analytical Queries
The following examples show how the system aggregates dynamic data across different paradigms:

#### PostgreSQL (Hybrid SQL):
-- Aggregating by a key nested inside a JSONB column
SELECT payload->>'eventType' as type, COUNT(*) 
FROM logs 
GROUP BY payload->>'eventType';

#### MongoDB (Aggregation Pipeline):
// Native document aggregation
db.logs.aggregate([
  { "$group": { "_id": "$eventType", "count": { "$sum": 1 } } }
]);

#### DuckDB (OLAP Analytics):
-- High-speed aggregation directly on JSON structures
SELECT eventType, count(*) 
FROM read_json_auto('logs.json') 
GROUP BY eventType;

## Benchmark Methodology
All databases were tested using identical input data to ensure fair comparison.
Each scenario was executed using:
- identical dataset (10,000 logs)
- same aggregation query: COUNT(eventType = 'error')
- measurement of:
  - write time
  - read time
Results were validated to ensure consistency across all databases.

## Benchmark Scenarios
Tested with **10,000+ records** across three structural complexities:
- **Small**: Base fields (timestamp, eventType, userId, latency, device).
- **Medium**: Added context fields (userId, sessionId, deviceType).
- **Large**: High-cardinality dynamic fields (extra telemetry data, nested objects).

## Key Insights
- **Write Performance**: MongoDB outperformed relational databases in high-frequency ingestion scenarios due to its schemaless nature.
- **Analytical Flexibility**: PostgreSQL (JSONB) provided the most robust SQL interface for complex aggregations without sacrificing the ability to store dynamic fields.
- **Local Analytics**: DuckDB proved exceptionally efficient for "one-off" analysis of large JSON log files without the overhead of a database server.

## Tech Stack
- **Language**: Python (pandas, psycopg2, pymongo, duckdb)
- **Infrastructure**: Docker (for database orchestration)
- **Concepts**: ETL/ELT, Semi-structured data, OLAP vs OLTP, Performance Benchmarking
