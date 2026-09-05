# Query Result to Scalar - High-Water Mark Demo

## 🎯 Overview

This project demonstrates the **[Query Result to Scalar](https://docs.matillion.com/data-productivity-cloud/designer/docs/query-to-scalar)** component using the classic **High-Water Mark (HWM)** incremental loading pattern - one of the most common use cases in data engineering.

### What is Query Result to Scalar?

The **Query Result to Scalar** component executes a SQL query and stores a **single value** (scalar) in a pipeline variable. Perfect for:
- Retrieving max/min values for incremental loads
- Getting row counts
- Fetching configuration values
- Retrieving last update timestamps
- Querying single aggregation results

### What is High-Water Mark?

**High-Water Mark (HWM)** is an incremental loading pattern that:
1. Tracks the last processed value (timestamp, ID, etc.)
2. Loads only new/changed records since that value
3. Updates the HWM after successful load
4. Avoids reprocessing existing data

**Benefits**:
✅ Faster loads (only new data)  
✅ Reduced warehouse costs  
✅ Lower network traffic  
✅ Enables real-time/near-real-time pipelines  

---

## 📁 Project Structure

```
Query_Result_To_Scalar_Demo/
├── README.md                         # This file
├── COMPONENT_DEEP_DIVE.md           # Technical details
├── 01_Setup_Demo_Data.orch.yaml     # Creates tables and sample data
├── 02_Initial_Load.orch.yaml        # Initial full load + set HWM
├── 03_Incremental_Load_HWM.orch.yaml # Incremental load using HWM
└── 04_Simulate_New_Data.orch.yaml   # Adds new data for testing
```

---

## 🚀 Quick Start

### Step 1: Setup (One-Time)

Run: `01_Setup_Demo_Data.orch.yaml`

**Creates**:
- `SOURCE_TRANSACTIONS` - Source table with 8 transactions
- `TARGET_TRANSACTIONS` - Empty target table
- `HWM_CONTROL` - Control table tracking high-water marks

**Result**: ✅ 3 tables created

### Step 2: Initial Load

Run: `02_Initial_Load.orch.yaml`

**What it does**:
1. Loads ALL data from source to target (8 rows)
2. **Uses Query Result to Scalar** to get MAX(TRANSACTION_DATE)
3. Stores max date in `max_transaction_date` variable
4. Updates HWM_CONTROL with initial high-water mark

**Result**: ✅ 8 rows loaded, HWM set to `2024-01-02 15:30:00`

### Step 3: Simulate New Data

Run: `04_Simulate_New_Data.orch.yaml`

**What it does**:
- Adds 5 new transactions (IDs 9-13) to SOURCE_TRANSACTIONS
- Dates: 2024-01-03 and 2024-01-04

**Result**: ✅ 5 new rows in source (13 total)

### Step 4: Incremental Load

Run: `03_Incremental_Load_HWM.orch.yaml`

**What it does** (The HWM Pattern!):
1. **Query Result to Scalar**: Gets current HWM from control table
2. Loads ONLY records WHERE date > HWM
3. **Query Result to Scalar**: Gets new max date
4. **Query Result to Scalar**: Gets row count loaded
5. Updates HWM to new max date
6. Logs results

**Result**: ✅ 5 new rows loaded, HWM updated to `2024-01-04 11:30:00`

### Step 5: Verify (Optional)

Run incremental load again: `03_Incremental_Load_HWM.orch.yaml`

**Result**: ✅ 0 rows loaded (no new data since last HWM)

---

## 📊 Tables Created

### SOURCE_TRANSACTIONS (Source System)
```sql
TRANSACTION_ID   NUMBER       PRIMARY KEY
CUSTOMER_ID      NUMBER       
AMOUNT           NUMBER(12,2) 
TRANSACTION_DATE TIMESTAMP    -- Used for HWM
STATUS           VARCHAR(20)  
```

### TARGET_TRANSACTIONS (Data Warehouse)
```sql
TRANSACTION_ID   NUMBER       PRIMARY KEY
CUSTOMER_ID      NUMBER       
AMOUNT           NUMBER(12,2) 
TRANSACTION_DATE TIMESTAMP    
STATUS           VARCHAR(20)  
LOAD_TIMESTAMP   TIMESTAMP    -- Audit column
```

### HWM_CONTROL (Metadata Table)
```sql
TABLE_NAME    VARCHAR(100)  PRIMARY KEY
HWM_COLUMN    VARCHAR(100)  -- Which column to track
HWM_VALUE     VARCHAR(500)  -- Current HWM value
LAST_UPDATE   TIMESTAMP     -- When HWM was updated
```

### INCREMENTAL_LOAD_LOG (Audit Table)
```sql
LOAD_TIMESTAMP  TIMESTAMP    
PREVIOUS_HWM    VARCHAR(500) 
NEW_HWM         VARCHAR(500) 
ROWS_LOADED     NUMBER       
LOAD_TYPE       VARCHAR(50)  
```

---

## 🔬 How Query Result to Scalar Works

### Example 1: Get Current HWM

**Component**: Get Current HWM

```yaml
Query Result to Scalar:
  Mode: Advanced
  Query:
    SELECT HWM_VALUE as current_hwm
    FROM HWM_CONTROL
    WHERE TABLE_NAME = 'SOURCE_TRANSACTIONS'
  
  Scalar Variable Mapping:
    Variable: current_hwm_value
    Column: CURRENT_HWM
```

**What happens**:
1. Query executes: `SELECT HWM_VALUE...`
2. Returns single value: `"2024-01-02 15:30:00"`
3. Stores in variable: `current_hwm_value = "2024-01-02 15:30:00"`
4. Variable used in next component

### Example 2: Get Max Date

**Component**: Get New Max Date

```yaml
Query Result to Scalar:
  Mode: Advanced
  Query:
    SELECT MAX(TRANSACTION_DATE) as max_date
    FROM SOURCE_TRANSACTIONS
    WHERE TRANSACTION_DATE > TO_TIMESTAMP_NTZ('${current_hwm_value}')
  
  Scalar Variable Mapping:
    Variable: new_max_date
    Column: MAX_DATE
```

**What happens**:
1. Query uses variable: `WHERE ... > '2024-01-02 15:30:00'`
2. Returns max date from new records: `"2024-01-04 11:30:00"`
3. Stores in variable: `new_max_date = "2024-01-04 11:30:00"`
4. Used to update HWM control table

### Example 3: Get Row Count

**Component**: Get Rows Loaded Count

```yaml
Query Result to Scalar:
  Mode: Advanced
  Query:
    SELECT COUNT(*) as row_count
    FROM SOURCE_TRANSACTIONS
    WHERE TRANSACTION_DATE > TO_TIMESTAMP_NTZ('${current_hwm_value}')
  
  Scalar Variable Mapping:
    Variable: rows_loaded
    Column: ROW_COUNT
```

**What happens**:
1. Counts new records
2. Returns: `5`
3. Stores in variable: `rows_loaded = 5`
4. Logged for auditing

---

## 💡 Key Concepts

### High-Water Mark Pattern Flow

```
┌─────────────────────────────────────────────┐
│ 1. Get Current HWM                          │
│    Query Result to Scalar ──> hwm_value     │
└────────────────┬────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────┐
│ 2. Load New Data                            │
│    WHERE date > hwm_value                   │
└────────────────┬────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────┐
│ 3. Get New Max Value                        │
│    Query Result to Scalar ──> new_max       │
└────────────────┬────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────┐
│ 4. Update HWM                               │
│    SET hwm_value = new_max                  │
└─────────────────────────────────────────────┘
```

### Why Use Scalar Variables?

**Scalar Variables**:
- Store single values (text, number, date)
- Can be used in SQL queries: `WHERE col > $variable`
- Pass between components in pipeline
- Enable dynamic, data-driven logic

**vs Grid Variables**:
- Grid = multiple rows/columns (tabular data)
- Scalar = single value

---

## 🎯 Use Cases

### 1. Timestamp-Based Incremental Loads
**Pattern**: Track MAX(updated_at) or MAX(created_at)
```sql
SELECT MAX(updated_at) FROM target
-- Use to load WHERE source.updated_at > max_value
```

### 2. ID-Based Incremental Loads
**Pattern**: Track MAX(id) for auto-increment columns
```sql
SELECT MAX(id) FROM target
-- Use to load WHERE source.id > max_id
```

### 3. Configuration-Driven Pipelines
**Pattern**: Query config values to control pipeline behavior
```sql
SELECT batch_size FROM config WHERE pipeline = 'etl_daily'
-- Use batch_size in LIMIT clauses
```

### 4. Data Quality Checks
**Pattern**: Query metrics and make decisions
```sql
SELECT COUNT(*) FROM data WHERE quality_flag = 'FAIL'
-- If count > threshold, send alert
```

### 5. Dynamic Partitioning
**Pattern**: Query date ranges for partition processing
```sql
SELECT MIN(date), MAX(date) FROM staging
-- Use dates to create/update partitions
```

---

## ⚙️ Component Properties

### Mode Options

**Basic Mode**:
- UI-driven query builder
- Select table, columns, filters
- Limited to simple queries

**Advanced Mode** (Recommended for HWM):
- Write custom SQL
- Use aggregations (MAX, MIN, COUNT)
- Use WHERE clauses with variables
- More flexible

### Scalar Variable Mapping

**Required**:
- **Input Column Name**: Column from query result
- **Scalar Variable Name**: Pipeline variable to store value

**Important**:
- Variable must exist BEFORE Query Result to Scalar runs
- Define variables in pipeline variables section
- Column names are UPPERCASE (Snowflake default)

---

## 🌟 Best Practices

### 1. HWM Column Selection

✅ **Good HWM Columns**:
- Monotonically increasing (always grows)
- Indexed for performance
- Never updates (immutable)
- Examples: created_at, insert_timestamp, auto_increment_id

❌ **Bad HWM Columns**:
- Can decrease or change (updated_at that can go backwards)
- Not indexed (slow queries)
- Nullable columns

### 2. HWM Initial Value

```sql
-- Set to ancient past to capture all data on first load
INSERT INTO HWM_CONTROL VALUES 
  ('SOURCE_TABLE', 'CREATED_AT', '1900-01-01 00:00:00')
```

### 3. Handle NULL Results

What if no new records?
```sql
SELECT COALESCE(MAX(date), '${current_hwm}') as max_date
FROM source
WHERE date > '${current_hwm}'
```

This ensures you get a value even if query returns NULL.

### 4. Add Auditing

- Log every incremental load
- Track rows loaded, time taken, HWM values
- Monitor for issues (unexpected row counts)

### 5. Error Handling

- Add failure transitions
- Don't update HWM if load fails
- Implement retry logic if needed

### 6. Testing

- Test with no new data (should load 0 rows)
- Test with new data
- Test with data gaps
- Verify HWM updates correctly

---

## 🔍 Troubleshooting

### Issue: No Rows Loaded on First Run

**Cause**: HWM not initialized properly

**Solution**: Check HWM_CONTROL has correct initial value:
```sql
SELECT * FROM HWM_CONTROL WHERE TABLE_NAME = 'YOUR_TABLE'
```

Should show ancient date like `1900-01-01` for first run.

### Issue: Duplicate Rows

**Cause**: Using `>=` instead of `>` in WHERE clause

**Solution**: Always use `>` (greater than), not `>=`:
```sql
-- ✅ Correct
WHERE date > $hwm

-- ❌ Wrong (loads HWM row again)
WHERE date >= $hwm
```

### Issue: Missing Rows

**Cause**: HWM updated but load failed

**Solution**: 
- Only update HWM AFTER successful load
- Use transitions: Load (success) → Update HWM
- Never update HWM on failure path

### Issue: Variable Not Found

**Cause**: Variable not defined in pipeline

**Solution**: 
1. Go to pipeline variables section
2. Add variable with correct name
3. Set type (TEXT, NUMBER, etc.)
4. Set default value

---

## 📈 Performance Tips

### 1. Index HWM Column
```sql
CREATE INDEX idx_transaction_date 
ON SOURCE_TRANSACTIONS(TRANSACTION_DATE)
```

### 2. Partition Large Tables
```sql
CREATE TABLE SOURCE_TRANSACTIONS (
  ...
  transaction_date TIMESTAMP
)
PARTITION BY DATE_TRUNC('DAY', transaction_date)
```

### 3. Batch Processing

For very large incremental loads:
```sql
-- Load in chunks
WHERE date > $hwm 
  AND date <= DATEADD(day, 1, $hwm)
LIMIT 10000
```

---

## ✅ Success Criteria

- [ ] Setup pipeline creates 3 tables
- [ ] Initial load loads 8 rows, sets HWM
- [ ] Simulate pipeline adds 5 new rows to source
- [ ] Incremental load loads exactly 5 rows
- [ ] HWM updates to new max date
- [ ] Running incremental again loads 0 rows
- [ ] INCREMENTAL_LOAD_LOG shows history

---

## 🎓 Learning Path

### Beginner
1. Run all 4 pipelines in order
2. Query HWM_CONTROL to see values
3. Query INCREMENTAL_LOAD_LOG to see history
4. Understand the flow

### Intermediate
1. Modify queries to use different HWM column
2. Add more audit columns
3. Implement error handling
4. Add notification on failure

### Advanced
1. Implement multi-table HWM pattern
2. Add CDC (Change Data Capture) logic
3. Create reusable HWM framework
4. Implement SCD Type 2 with HWM

---

## 📚 Additional Resources

- [Query Result to Scalar Documentation](https://docs.matillion.com/data-productivity-cloud/designer/docs/query-to-scalar)
- [Variables Documentation](https://docs.matillion.com/data-productivity-cloud/designer/docs/variables)
- [SQL Executor](https://docs.matillion.com/data-productivity-cloud/designer/docs/sql-executor)
- [Create Table](https://docs.matillion.com/data-productivity-cloud/designer/docs/create-table-v2)

---

## 🎉 Summary

You now understand:
- ✅ How Query Result to Scalar works
- ✅ High-Water Mark incremental loading pattern
- ✅ Scalar variable usage
- ✅ Building production-ready incremental loads
- ✅ Best practices for HWM implementation

**This pattern is used in 80%+ of production data pipelines!**

Happy building! 🚀