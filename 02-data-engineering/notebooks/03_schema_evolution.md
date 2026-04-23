# Schema Evolution

## Why Schema Evolution is Hard

Data schemas change over time: new fields are added, old ones renamed, types changed.
In a distributed system with producers and consumers deployed independently, you need
**compatibility contracts** so old consumers can read new data and vice versa.

---

## 1. Compatibility Types

| Type | Producer | Consumer | Key Rule |
|------|---------|---------|---------|
| **Backward** | New schema | Old schema | New schema can read old data |
| **Forward** | Old schema | New schema | Old schema can read new data |
| **Full** | Either | Either | Both backward and forward |
| **None** | Either | Either | No guarantees — risky |

### Safe vs Unsafe Changes

| Change | Backward | Forward | Full |
|--------|---------|---------|------|
| Add optional field with default | ✓ | ✓ | ✓ |
| Remove optional field | ✓ | ✓ | ✓ |
| Add required field (no default) | ✗ | ✓ | ✗ |
| Remove required field | ✓ | ✗ | ✗ |
| Rename a field | ✗ | ✗ | ✗ |
| Change field type (int→long) | ✓* | ✓* | ✓* |
| Change field type (string→int) | ✗ | ✗ | ✗ |

*Only if types are promotable (Avro: int→long, float→double).

---

## 2. Avro Schema Evolution

Avro schemas are stored as JSON and registered with a schema registry.

```json
// v1
{"type": "record", "name": "User",
 "fields": [
   {"name": "user_id", "type": "string"},
   {"name": "name",    "type": "string"}
 ]}

// v2 — backward compatible (added optional email with default)
{"type": "record", "name": "User",
 "fields": [
   {"name": "user_id", "type": "string"},
   {"name": "name",    "type": "string"},
   {"name": "email",   "type": ["null", "string"], "default": null}
 ]}
```

**Avro field resolution rules:**

*   If a field in the writer schema has no matching field in the reader schema → field ignored.
*   If a field in the reader schema has no matching field in the writer schema → reader uses default.
*   Names matched by `name` first, then `aliases`.

---

## 3. Protobuf Schema Evolution

Protobuf uses field numbers (not names) for binary encoding. Key rules:

*   **Never reuse field numbers** — old data will be misinterpreted.
*   **Never remove a required field** — breaks old decoders.
*   Adding optional fields is safe.
*   Adding fields at the end is conventional.

```protobuf
// v1
message User {
  string user_id = 1;
  string name    = 2;
}

// v2 — safe additions
message User {
  string user_id = 1;
  string name    = 2;
  string email   = 3;  // new — safe
  // NEVER reuse field number 1 or 2
}
```

---

## 4. Schema Registry Pattern

Centralised schema registry (Confluent Schema Registry, AWS Glue Schema Registry):

```
Producer → check/register schema → get schema_id
Producer → serialize(schema_id + payload) → Kafka
Consumer → read schema_id from message → fetch schema from registry → deserialize
```

Benefits:

*   Single source of truth for all schema versions.
*   Enforce compatibility mode (BACKWARD, FULL) at registration time.
*   Consumers always know how to deserialize any message.

---

## 5. Database Schema Evolution

### Additive-Only Changes (Safe)

*   Add nullable column with default value.
*   Add new table.
*   Add index.

### Risky Changes (Need Migration)

*   Rename column → use alias/view + dual-write migration.
*   Change column type → add new column, backfill, swap.
*   Remove column → soft-delete first (keep column, stop writing), then drop after all readers updated.

### Expand/Contract Pattern

```
Phase 1 (Expand): Add new column. Old code ignores it.
Phase 2 (Migrate): Backfill new column; update code to write both.
Phase 3 (Switch): Switch reads to new column.
Phase 4 (Contract): Remove old column once no readers remain.
```

---

## Key Interview Points

*   **Backward vs forward:** Interviewers often confuse these. Backward = new schema reads old data (consumer upgrades first). Forward = old schema reads new data (producer upgrades first).
*   **Protobuf field numbers:** Explain that numbers are the binary keys — renaming a field is free, but changing its number is catastrophic.
*   **Schema registry:** In Kafka architectures, a schema registry is mandatory for production. Every message carries a schema ID, not the full schema, keeping messages small.
*   **Expand/contract:** The safe database migration pattern for zero-downtime column changes.
