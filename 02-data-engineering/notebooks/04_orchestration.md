# Data Pipeline Orchestration

## What is Orchestration?

Orchestration is the scheduling, dependency management, monitoring, and error-handling
layer around data pipelines. Without it, you have cron jobs that silently fail and
interdependencies that break each other.

---

## 1. DAG Fundamentals

### What is a DAG?

A **Directed Acyclic Graph** where:

*   **Nodes** = tasks (extract, transform, load, validate, notify)
*   **Edges** = dependencies (task B runs only after task A succeeds)
*   **Acyclic** = no circular dependencies (no infinite loops)

### DAG Design Principles

| Principle | Description |
|----------|-------------|
| **Idempotency** | Running the same DAG twice produces the same result |
| **Atomicity** | A task either fully succeeds or leaves no partial state |
| **Observability** | Each task emits metrics, logs, and status |
| **Parametric** | DAG accepts `execution_date` and other runtime params |
| **Backfillable** | Can re-run for a historical date range |

---

## 2. Apache Airflow Patterns

### Core Concepts

*   **DAG:** Python file that defines tasks and dependencies.
*   **Operator:** Template for a task type (PythonOperator, BashOperator, SqlOperator).
*   **Task Instance:** A specific run of a task for a given `execution_date`.
*   **XCom:** Cross-communication between tasks (push/pull small values).
*   **Sensor:** Polls for a condition (file exists, API available) before proceeding.

### Scheduling

```python
# Run daily at midnight UTC
dag = DAG(
    "my_pipeline",
    schedule_interval="0 0 * * *",
    start_date=datetime(2025, 1, 1),
    catchup=False,  # don't backfill all missed runs
)
```

### BranchOperator Pattern

```
[check_data_type]
    ├─ "parquet" → [process_parquet]
    └─ "csv"     → [process_csv]
        └──────────┘
               ↓
        [load_to_warehouse]
```

### Dynamic Task Mapping (Airflow 2.3+)

```python
# Generate tasks dynamically based on config
process = PythonOperator.partial(task_id="process").expand(
    op_kwargs=[{"partition": p} for p in get_partitions()]
)
```

---

## 3. Retry Strategies

### Exponential Back-off

Wait `base * 2^attempt` seconds between retries:

```
Attempt 1: wait 2s
Attempt 2: wait 4s
Attempt 3: wait 8s
Attempt 4: wait 16s
```

Add jitter (`± 20%`) to avoid thundering herd when many tasks fail simultaneously.

### Retry Configuration

| Parameter | Typical Value |
|----------|-------------|
| `retries` | 3 |
| `retry_delay` | 5 minutes |
| `retry_exponential_backoff` | True |
| `max_retry_delay` | 30 minutes |

---

## 4. SLA Monitoring

*   Define expected completion time for each task.
*   If task hasn't completed by SLA → trigger `sla_miss_callback`.
*   Common action: send Slack alert, PagerDuty alert, create JIRA ticket.

```python
# Airflow SLA: task must complete within 2 hours of scheduled start
task = PythonOperator(
    task_id="transform_data",
    sla=timedelta(hours=2),
    ...
)
```

---

## 5. Pipeline Observability

### Key Metrics to Track

*   **Task success rate:** 7-day rolling average per task.
*   **Task duration p50/p95/p99:** Alert on regression > 50%.
*   **Data freshness:** `now() - max(event_time)` in output table.
*   **Row count delta:** Compare to 7-day average; alert on ±30% deviation.

### Metadata Store

Every pipeline run should write a metadata record:

```json
{
  "dag_id": "user_feature_pipeline",
  "execution_date": "2025-01-15T00:00:00Z",
  "start_time": "...", "end_time": "...",
  "status": "success",
  "rows_processed": 1_234_567,
  "output_partition": "s3://bucket/date=2025-01-15/"
}
```

---

## 6. Common Orchestration Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| God DAG | One DAG does everything | Split by logical boundary |
| Non-idempotent tasks | Re-run corrupts data | Use `INSERT OVERWRITE` / upsert |
| Silent failures | Task "succeeds" but produces wrong output | Add post-task validation |
| Tight coupling | DAG A waits on DAG B's internal task | Use ExternalTaskSensor on final output |
| Long polling sensors | Sensor runs every 30s for hours | Use deferrable operators |

---

## Key Interview Points

*   **Idempotency:** Explain using partition overwrite — always write `s3://bucket/date=YYYY-MM-DD/`, so re-running the same date replaces the partition cleanly.
*   **XCom limitations:** XCom is only for small values (< 100KB). For large data, write to S3/GCS and pass the path via XCom.
*   **Sensor vs trigger:** Sensors poll actively (CPU cost). Deferrable sensors (Airflow 2.2+) are async — much cheaper for long waits.
*   **SLA miss:** Distinguish from task failure — a slow task can still succeed after the SLA window.
