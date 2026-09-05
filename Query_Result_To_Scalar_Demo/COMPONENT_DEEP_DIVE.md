# Query Result to Scalar - Component Deep Dive

## Component Architecture

**Component ID**: `query-to-scalar`  
**Type**: Orchestration  
**Category**: Data Query & Variable Management

### Ports

**Input**: INPUT (many connections)  
**Outputs**: SUCCESS, FAILURE, UNCONDITIONAL (many connections each)

## How It Works

### Execution Flow

1. **Query Execution**: Runs SQL query against Snowflake
2. **Result Validation**: Ensures query returns exactly one row, one column
3. **Value Extraction**: Extracts the scalar value from result
4. **Variable Assignment**: Stores value in specified pipeline variable
5. **Transition**: Follows success/failure path

### Advanced Mode (Recommended)

```yaml
Query Result to Scalar:
  Mode: Advanced
  Query: |
    SELECT MAX(updated_at) as max_value
    FROM my_table
    WHERE processed = TRUE
  
  Scalar Variable Mapping:
    Variable Name: last_processed_date
    Input Column: MAX_VALUE
```

**Result**: Variable `last_processed_date` contains the max date value

### Basic Mode

- UI-driven query builder
- Limited to simple SELECT queries
- Best for single-table, single-column queries
- Less flexible than Advanced Mode

## High-Water Mark Pattern

### Pattern Overview

**Purpose**: Incrementally load only new/changed data

**Steps**:
1. Retrieve last processed value (HWM)
2. Load records greater than HWM
3. Calculate new HWM
4. Update HWM for next run

### Implementation Pattern

```yaml
Step 1: Get HWM (Query Result to Scalar)
  Query: SELECT hwm_value FROM control
  Store in: current_hwm

Step 2: Load New Data (SQL Executor)
  WHERE source_date > $current_hwm

Step 3: Get New HWM (Query Result to Scalar)
  Query: SELECT MAX(source_date) FROM loaded_data
  Store in: new_hwm

Step 4: Update Control (SQL Executor)
  UPDATE control SET hwm_value = $new_hwm
```

### HWM Variations

**Timestamp-Based**:
```sql
SELECT MAX(updated_at) FROM table
WHERE updated_at > '${last_hwm}'
```

**ID-Based**:
```sql
SELECT MAX(id) FROM table
WHERE id > ${last_id}
```

**Date-Based**:
```sql
SELECT MAX(order_date) FROM orders
WHERE order_date > '${last_date}'
```

## Scalar Variables

### Variable Types

**TEXT**:
- Strings, dates, timestamps
- Example: `"2024-01-01 10:30:00"`

**NUMBER**:
- Integers, decimals
- Example: `12345` or `99.99`

### Variable Scope

**SHARED** (Default):
- Same value across all pipeline branches
- Use for sequential processing

**COPIED**:
- Independent copy per branch
- Use for parallel processing

### Variable Visibility

**PRIVATE**:
- Visible only in current pipeline

**PUBLIC**:
- Accessible to child pipelines
- Can be overridden when calling child

## Real-World Patterns

### Pattern 1: Daily Incremental Load

```yaml
Schedule: Daily at 2 AM

Pipeline:
  1. Get HWM (last run date)
  2. Load yesterday's data
  3. Update HWM to yesterday
  
Result: Each day loads only new data
```

### Pattern 2: Multiple Table Incremental

```yaml
For each table:
  1. Get table-specific HWM
  2. Load incremental data
  3. Update table-specific HWM
  
Use HWM_CONTROL with TABLE_NAME as key
```

### Pattern 3: Data Quality Gating

```yaml
1. Load staging data
2. Query Result to Scalar: Count errors
3. If error_count > threshold:
     Stop pipeline
   Else:
     Continue to production
```

### Pattern 4: Dynamic Batch Sizing

```yaml
1. Query Result to Scalar: Count pending records
2. Calculator: Determine batch size
3. Process in batches
```

## Advanced Techniques

### Technique 1: COALESCE for NULL Handling

**Problem**: Query returns NULL when no new data

**Solution**:
```sql
SELECT COALESCE(MAX(date), '${current_hwm}') as max_date
FROM source
WHERE date > '${current_hwm}'
```

Returns current HWM if no new records.

### Technique 2: Multiple Scalar Queries

**Pattern**: Get multiple metrics

```yaml
Query 1: SELECT MAX(date) -> max_date
Query 2: SELECT MIN(date) -> min_date  
Query 3: SELECT COUNT(*) -> row_count

Use all three in downstream logic
```

### Technique 3: Conditional HWM Update

**Only update HWM if data was loaded**:

```yaml
1. Get row count loaded
2. If count > 0:
     Update HWM
   Else:
     Skip HWM update
```

### Technique 4: HWM Rollback

**If load fails, keep old HWM**:

```yaml
Get HWM -> Load (success) -> Get New Max -> Update HWM
                   |
                (failure)
                   |
                Don't update HWM (rollback)
```

## Performance Optimization

### 1. Index HWM Columns

```sql
-- Dramatically speeds up queries
CREATE INDEX idx_updated_at ON table(updated_at)
```

### 2. Query Optimization

**Good**:
```sql
SELECT MAX(indexed_column) FROM table
```

**Bad**:
```sql
SELECT MAX(CAST(string_column AS DATE)) FROM table
-- Forces full table scan
```

### 3. Partition Pruning

```sql
-- If table partitioned by date
SELECT MAX(date) FROM table
WHERE date > '2024-01-01'  -- Prunes old partitions
```

## Common Pitfalls

### Pitfall 1: Using >= Instead of >

**Wrong**:
```sql
WHERE date >= $hwm  -- Loads HWM row again!
```

**Correct**:
```sql
WHERE date > $hwm   -- Loads only new rows
```

### Pitfall 2: Updatable HWM Columns

**Problem**: Using `updated_at` that can decrease

**Solution**: Use `created_at` or `insert_timestamp` (immutable)

### Pitfall 3: No NULL Handling

**Problem**: Query returns NULL, breaks pipeline

**Solution**: Use COALESCE or default values

### Pitfall 4: Updating HWM Before Load

**Wrong Order**:
```
Get New HWM -> Update Control -> Load Data
```

**Correct Order**:
```
Get HWM -> Load Data -> Get New HWM -> Update Control
```

## Testing Strategies

### Test 1: No New Data

**Setup**: Run incremental load twice

**Expected**: Second run loads 0 rows

### Test 2: New Data

**Setup**: Add data, run incremental

**Expected**: New rows loaded, HWM updated

### Test 3: Data Gaps

**Setup**: Add data with date gaps

**Expected**: All data loaded, HWM = latest date

### Test 4: Concurrent Loads

**Setup**: Two loads running simultaneously

**Expected**: No duplicate loads, HWM correct

## Monitoring & Alerting

### Key Metrics

1. **Rows Loaded**: Track incremental volume
2. **HWM Progression**: Ensure HWM advancing
3. **Load Duration**: Monitor performance
4. **Failure Rate**: Track errors

### Alert Conditions

```yaml
Alert if:
  - Rows loaded = 0 for 3+ consecutive runs
  - HWM not advancing for 24+ hours
  - Load duration > 2x average
  - Failure rate > 5%
```

## Comparison: Scalar vs Grid

| Aspect | Query Result to Scalar | Query Result to Grid |
|--------|------------------------|----------------------|
| **Returns** | Single value | Multiple rows |
| **Use Case** | Max, min, count | Lists, batch data |
| **Variable Type** | TEXT, NUMBER | GRID |
| **Example** | MAX(date) | SELECT * FROM table |

## Best Practices Summary

✅ **DO**:
- Use indexed columns for HWM
- Handle NULL results
- Log HWM changes
- Update HWM only after successful load
- Use immutable columns (created_at)
- Test with edge cases

❌ **DON'T**:
- Use >= in WHERE clause
- Update HWM before loading
- Use updatable columns as HWM
- Ignore NULL results
- Skip error handling

## Summary

The **Query Result to Scalar** component is essential for:

✅ **High-Water Mark Patterns**: Incremental loading  
✅ **Metrics Retrieval**: Get counts, maxes, mins  
✅ **Configuration**: Query dynamic values  
✅ **Data Quality**: Check thresholds  
✅ **Conditional Logic**: Make data-driven decisions  

Mastering this component is critical for building efficient, production-grade data pipelines.

---

**For practical examples, see [README.md](./README.md)**