# Fixed Flow Component - Complete Business Use Cases

## 🎯 Overview

The **[Fixed Flow](https://docs.matillion.com/data-productivity-cloud/designer/docs/fixed-flow)** is a transformation component that generates static, hard-coded data tables for use in your pipelines. This is incredibly powerful for real-world business scenarios.

### What is Fixed Flow?

**Fixed Flow** allows you to:
- Generate tables of data without connecting to external sources
- Create static reference/lookup tables inline
- Provide test data during development
- Define configuration data directly in pipelines
- Create data dictionaries and mapping tables

---

## 📊 Complete Business Use Cases

### Use Case 1: Currency Conversion Lookup

**Business Problem**: International e-commerce company needs to convert all transactions to USD for reporting, but doesn't want to maintain a separate database table for exchange rates.

**Solution**: Use Fixed Flow to provide currency conversion rates directly in the transformation pipeline.

**Fixed Flow Configuration**:
```yaml
Columns:
  - CURRENCY_CODE (VARCHAR, 3)
  - CURRENCY_NAME (VARCHAR, 50)
  - USD_CONVERSION_RATE (FLOAT, scale 4)
  - REGION (VARCHAR, 50)

Values:
  USD | US Dollar         | 1.0000 | North America
  EUR | Euro             | 1.0850 | Europe
  GBP | British Pound    | 1.2650 | Europe
  JPY | Japanese Yen     | 0.0067 | Asia Pacific
  CAD | Canadian Dollar  | 0.7350 | North America
  AUD | Australian Dollar | 0.6550 | Asia Pacific
```

**Pipeline Flow**:
```
Transactions Table Input
         +
         |
Fixed Flow (Currency Rates)
         |
         v
    Join on CURRENCY_CODE
         |
         v
Calculate (AMOUNT * USD_CONVERSION_RATE)
         |
         v
   Output: Standardized USD Amounts
```

**Business Benefits**:
- ✅ No external currency API needed
- ✅ Rates controlled by data team
- ✅ Easy to update (edit pipeline, no database changes)
- ✅ Self-documenting (rates visible in pipeline)
- ✅ Version controlled with pipeline code

**When to Update**: Monthly or as exchange rates change significantly

---

### Use Case 2: Product Category Mapping

**Business Problem**: Sales system uses cryptic product codes ("LAP-001", "PHN-015"), but reporting needs friendly categories and business units.

**Solution**: Fixed Flow provides product code to category mappings.

**Fixed Flow Configuration**:
```yaml
Columns:
  - PRODUCT_CODE (VARCHAR, 20)
  - PRODUCT_NAME (VARCHAR, 100)
  - CATEGORY (VARCHAR, 50)
  - BUSINESS_UNIT (VARCHAR, 50)
  - MARGIN_PERCENT (FLOAT)

Values:
  LAPTOP    | Laptop Pro 15    | Electronics | Hardware | 22.5
  PHONE     | Smartphone X     | Electronics | Mobile   | 35.0
  TABLET    | Tablet Air       | Electronics | Mobile   | 28.0
  MONITOR   | 27" Display      | Accessories | Hardware | 18.5
  KEYBOARD  | Wireless KB      | Accessories | Hardware | 45.0
  MOUSE     | Optical Mouse    | Accessories | Hardware | 50.0
  HEADSET   | Bluetooth Set    | Accessories | Audio    | 40.0
  WEBCAM    | HD Camera        | Accessories | Hardware | 38.0
```

**Pipeline Flow**:
```
Sales Data (with codes)
         |
         v
Fixed Flow (Category Mapping)
         |
         v
    Join on PRODUCT_CODE
         |
         v
Group By CATEGORY, BUSINESS_UNIT
         |
         v
Output: Sales by Category Report
```

**Business Benefits**:
- ✅ Enriches transaction data with business context
- ✅ Enables category-level reporting
- ✅ Single source of truth for product mappings
- ✅ Easy to add new products
- ✅ No database schema changes needed

---

### Use Case 3: Business Rules & Thresholds

**Business Problem**: Different customer segments have different discount rules, credit limits, and processing priorities.

**Solution**: Fixed Flow defines business rules that drive pipeline logic.

**Fixed Flow Configuration**:
```yaml
Columns:
  - CUSTOMER_SEGMENT (VARCHAR, 20)
  - DISCOUNT_PERCENT (FLOAT)
  - CREDIT_LIMIT (NUMBER)
  - PRIORITY_LEVEL (NUMBER)
  - APPROVAL_REQUIRED (BOOLEAN)

Values:
  Enterprise | 15.0 | 100000 | 1 | FALSE
  Corporate  | 10.0 |  50000 | 2 | FALSE
  SMB        |  5.0 |  10000 | 3 | TRUE
  Retail     |  0.0 |   1000 | 4 | TRUE
```

**Pipeline Flow**:
```
Customer Orders
         |
         v
Fixed Flow (Business Rules)
         |
         v
    Join on CUSTOMER_SEGMENT
         |
         v
Calculator:
  - Apply discount
  - Check credit limit
  - Flag for approval
         |
         v
Filter: Route based on APPROVAL_REQUIRED
         |
         v
Output: Auto-approved vs Manual Review
```

**Business Benefits**:
- ✅ Centralized business rules
- ✅ Easy to update thresholds
- ✅ Consistent application across pipelines
- ✅ Clear documentation of rules
- ✅ Audit trail via pipeline version control

---

### Use Case 4: Region & Tax Configuration

**Business Problem**: Sales tax varies by state/region, and need to apply correct rates and remittance schedules.

**Solution**: Fixed Flow provides regional tax configuration.

**Fixed Flow Configuration**:
```yaml
Columns:
  - STATE_CODE (VARCHAR, 2)
  - STATE_NAME (VARCHAR, 50)
  - TAX_RATE (FLOAT)
  - TAX_JURISDICTION (VARCHAR, 50)
  - REMITTANCE_SCHEDULE (VARCHAR, 20)

Values:
  CA | California  | 7.25 | State Board of Eq | Monthly
  NY | New York    | 4.00 | NY Dept of Tax   | Monthly
  TX | Texas       | 6.25 | TX Comptroller   | Quarterly
  FL | Florida     | 6.00 | FL Dept Revenue  | Monthly
  WA | Washington  | 6.50 | WA Dept Revenue  | Monthly
  IL | Illinois    | 6.25 | IL Dept Revenue  | Monthly
```

**Pipeline Flow**:
```
Sales Transactions
         |
         v
Fixed Flow (Tax Configuration)
         |
         v
    Join on STATE_CODE
         |
         v
Calculator:
  - Calculate tax amount
  - Determine remittance period
         |
         v
Group By: TAX_JURISDICTION, REMITTANCE_SCHEDULE
         |
         v
Output: Tax Liability Report
```

**Business Benefits**:
- ✅ Accurate tax calculations
- ✅ Automated remittance scheduling
- ✅ Compliance reporting
- ✅ Easy to update when rates change
- ✅ Supports multi-state operations

---

### Use Case 5: Test Data Generation

**Business Problem**: During development, need sample data to test transformations without accessing production databases.

**Solution**: Fixed Flow provides realistic test data.

**Fixed Flow Configuration**:
```yaml
Columns:
  - CUSTOMER_ID (NUMBER)
  - CUSTOMER_NAME (VARCHAR, 100)
  - SEGMENT (VARCHAR, 20)
  - ANNUAL_REVENUE (NUMBER)
  - ACTIVE (BOOLEAN)

Values:
  1001 | Acme Corp       | Enterprise | 5000000 | TRUE
  1002 | Beta Industries | Corporate  | 1200000 | TRUE
  1003 | Gamma LLC       | SMB        |  250000 | TRUE
  1004 | Delta Inc       | Retail     |   50000 | FALSE
  1005 | Epsilon Co      | Corporate  |  800000 | TRUE
```

**Pipeline Flow**:
```
Fixed Flow (Test Customers)
         |
         v
    Your Transformations
         |
         v
    Filter, Join, Calculate, etc.
         |
         v
Output: Test Results
```

**Development Benefits**:
- ✅ Test without production access
- ✅ Consistent test scenarios
- ✅ Fast iteration (no database queries)
- ✅ Edge cases covered (active/inactive, various segments)
- ✅ Shareable test data with team

---

### Use Case 6: Data Quality Rules

**Business Problem**: Need to validate incoming data against known good values and flag anomalies.

**Solution**: Fixed Flow defines acceptable values for validation.

**Fixed Flow Configuration**:
```yaml
Columns:
  - VALID_STATUS (VARCHAR, 20)
  - CATEGORY (VARCHAR, 50)
  - IS_FINAL (BOOLEAN)
  - REQUIRES_APPROVAL (BOOLEAN)

Values:
  New        | Pending    | FALSE | FALSE
  Processing | In-Flight  | FALSE | FALSE
  Approved   | Complete   | TRUE  | TRUE
  Shipped    | Complete   | TRUE  | FALSE
  Delivered  | Complete   | TRUE  | FALSE
  Cancelled  | Terminated | TRUE  | TRUE
  Refunded   | Terminated | TRUE  | TRUE
```

**Pipeline Flow**:
```
Incoming Data
         |
         v
Fixed Flow (Valid Statuses)
         |
         v
Left Join on STATUS
         |
         v
Filter: WHERE Valid_Status IS NULL
         |
         v
Output: Invalid Status Records (Data Quality Issues)
```

**Business Benefits**:
- ✅ Automated data quality checks
- ✅ Flag unexpected values
- ✅ Prevents downstream errors
- ✅ Clear definition of valid values
- ✅ Easy to extend validation rules

---

### Use Case 7: Hierarchy & Rollup Definitions

**Business Problem**: Need to roll up sales from stores → districts → regions → divisions.

**Solution**: Fixed Flow defines organizational hierarchy.

**Fixed Flow Configuration**:
```yaml
Columns:
  - STORE_ID (NUMBER)
  - STORE_NAME (VARCHAR, 100)
  - DISTRICT (VARCHAR, 50)
  - REGION (VARCHAR, 50)
  - DIVISION (VARCHAR, 50)

Values:
  101 | Downtown SF    | Bay Area   | West    | USA
  102 | San Jose Mall  | Bay Area   | West    | USA
  103 | Seattle Center | Northwest  | West    | USA
  201 | Manhattan 5th  | NYC        | East    | USA
  202 | Brooklyn Plaza | NYC        | East    | USA
  301 | London Oxford  | UK South   | Europe  | INTL
  302 | Paris Champs   | France     | Europe  | INTL
```

**Pipeline Flow**:
```
Store-Level Sales
         |
         v
Fixed Flow (Store Hierarchy)
         |
         v
    Join on STORE_ID
         |
         v
Group By: DISTRICT, REGION, DIVISION
         |
         v
Output: Multi-Level Sales Rollup
```

**Business Benefits**:
- ✅ Consistent hierarchy across reports
- ✅ Multiple aggregation levels
- ✅ Easy to reorganize structure
- ✅ Supports what-if scenarios
- ✅ Central definition of org structure

---

## 🔧 Component Properties

### Columns Definition

**Properties**:
- **Name**: Column name (must be unique)
- **Type**: Data type (VARCHAR, NUMBER, FLOAT, BOOLEAN, DATE, TIMESTAMP, TIME, VARIANT)
- **Size**: For VARCHAR (max characters), NUMBER (precision)
- **Scale**: For FLOAT/NUMBER (decimal places)

### Values Grid

- Dynamically generated based on columns defined
- One row per data record
- Enter values matching column data types
- Can use pipeline variables for dynamic values

### Key Features

✅ **No External Dependencies**: Data embedded in pipeline  
✅ **Version Controlled**: Changes tracked with pipeline  
✅ **Self-Documenting**: Data visible in component  
✅ **Fast Execution**: In-memory, no database queries  
✅ **Easy Updates**: Edit and redeploy pipeline  

---

## 🌟 Best Practices

### When to Use Fixed Flow

✅ **Good Use Cases**:
- Static reference data (< 1000 rows)
- Lookup tables that rarely change
- Configuration parameters
- Business rules and thresholds
- Test data
- Data quality validation rules
- Small dimension tables

❌ **Not Recommended**:
- Large datasets (>1000 rows)
- Frequently changing data
- Data that needs separate governance
- Historical data requiring auditing
- Data shared across many pipelines

### Design Guidelines

**1. Keep It Small**
- Limit to 100-500 rows
- Larger datasets → use database tables

**2. Document Purpose**
- Add clear component name
- Document in pipeline description
- Explain when/why to update

**3. Version Control**
- Treat updates like code changes
- Review changes in PR/merge requests
- Document update reasons in commits

**4. Update Process**
- Define ownership (who can update)
- Set update schedule (monthly, quarterly)
- Test after changes

**5. Alternative Approaches**
```
Fixed Flow → Good for: Static, < 500 rows, rarely changes
Database Table → Good for: Large, frequently updated, shared
Variable → Good for: Single values, simple configs
External API → Good for: Real-time, third-party data
```

---

## 💼 Real-World Implementation Examples

### Example 1: E-Commerce (Complete Flow)

**Scenario**: Online retailer needs to enrich orders with category and apply segment-specific discounts.

```yaml
Pipeline: Order_Processing

Step 1: Table Input
  Source: RAW_ORDERS
  Columns: order_id, customer_id, product_code, amount

Step 2: Fixed Flow (Product Categories)
  Data:
    LAPTOP | Electronics | Hardware
    PHONE  | Electronics | Mobile
    DESK   | Furniture   | Office

Step 3: Fixed Flow (Customer Segments)
  Data:
    Enterprise | 15% discount
    Retail     | 5% discount

Step 4: Join Orders → Products (on product_code)

Step 5: Join Result → Segments (on customer_id → segment)

Step 6: Calculator
  - Apply discount
  - Calculate final amount
  - Add category labels

Step 7: Table Output
  Target: PROCESSED_ORDERS
```

### Example 2: Financial Services

**Scenario**: Bank needs to route transactions based on risk rules.

```yaml
Pipeline: Transaction_Routing

Step 1: Table Input
  Source: PENDING_TRANSACTIONS

Step 2: Fixed Flow (Risk Rules)
  Data:
    amount < 1000    | auto_approve
    amount < 10000   | manager_review
    amount >= 10000  | executive_review

Step 3: Fixed Flow (Country Risk Scores)
  Data:
    USA | Low    | 1
    GBR | Low    | 1
    CHN | Medium | 2
    RUS | High   | 3

Step 4: Join transactions with risk rules
Step 5: Apply routing logic
Step 6: Output to appropriate queues
```

---

## 📈 Performance Considerations

### Execution Speed

✅ **Fast**: In-memory data generation  
✅ **No Latency**: No network calls  
✅ **Predictable**: Consistent performance  

### Resource Usage

- **Memory**: Minimal (data size × row count)
- **Compute**: Negligible
- **Network**: None

### Scalability

**Row Count Guidelines**:
- < 100 rows: Excellent
- 100-500 rows: Very good
- 500-1000 rows: Good
- 1000+ rows: Consider database table

---

## 🔄 Maintenance & Updates

### Update Workflow

1. **Identify Need**: Rate change, new category, etc.
2. **Edit Pipeline**: Update Fixed Flow values
3. **Test**: Sample component output
4. **Review**: Code review if required
5. **Deploy**: Commit and push changes
6. **Validate**: Run pipeline, check results

### Change Tracking

```bash
git log Fixed_Flow_Pipeline.yaml

commit abc123
Date: 2024-01-15
Updated EUR exchange rate from 1.08 to 1.085

commit def456
Date: 2023-12-01  
Added new product category: Smart Home
```

---

## ✅ Success Checklist

- [ ] Fixed Flow used for appropriate use case (< 500 rows, static)
- [ ] Column names are clear and descriptive
- [ ] Data types match actual data
- [ ] Values are accurate and current
- [ ] Component sampled successfully
- [ ] Joins work correctly downstream
- [ ] Update process documented
- [ ] Ownership assigned
- [ ] Changes version controlled

---

## 🎓 Summary

The **Fixed Flow** component is perfect for:

✅ **Reference Data**: Currency rates, product categories, region codes  
✅ **Business Rules**: Thresholds, discounts, validation rules  
✅ **Configuration**: System settings, processing rules  
✅ **Test Data**: Development and debugging  
✅ **Lookup Tables**: Small, static enrichment data  

**Key Advantages**:
- No external data sources needed
- Version controlled with pipeline
- Self-documenting
- Fast execution
- Easy to update

**When NOT to use**: Large datasets, frequently changing data, data requiring separate governance.

---

## 📚 Additional Resources

- [Fixed Flow Documentation](https://docs.matillion.com/data-productivity-cloud/designer/docs/fixed-flow)
- [Join Component](https://docs.matillion.com/data-productivity-cloud/designer/docs/join)
- [Calculator Component](https://docs.matillion.com/data-productivity-cloud/designer/docs/calculator)
- [Variables Documentation](https://docs.matillion.com/data-productivity-cloud/designer/docs/variables)

---

**Fixed Flow empowers data teams to embed reference data directly in pipelines, reducing dependencies and speeding up development!** 🚀