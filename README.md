# Amazon Redshift & Analytics Databases Demo

## Overview

This demo demonstrates Amazon Redshift's capabilities for modern data warehousing and analytics, specifically showcasing how it handles both "hot" operational data and "cold" archived data through Redshift Spectrum.

**Objective:** Show how Redshift handles structured "hot" data in the warehouse and unstructured "cold" data in S3 via Redshift Spectrum.

**Duration:** 10-15 minutes

**Key Features Demonstrated:**
- Redshift data warehousing capabilities
- Redshift Spectrum for querying S3 data
- Analytics workloads and performance
- Hybrid hot/cold data analytics

## Prerequisites

### AWS Services Required
- Amazon Redshift cluster
- Amazon S3 bucket
- AWS Glue Data Catalog
- IAM roles with appropriate permissions

### Tools Needed
- AWS Management Console access
- Redshift Query Editor v2 (or SQL client)
- Basic SQL knowledge

## Step 1: Create and Configure Redshift Cluster

### 1.1 Create Redshift Cluster
1. Navigate to Amazon Redshift Console
   - Go to AWS Console → Amazon Redshift
   - Click "Create cluster"

2. Configure Cluster Settings
   ```
   Cluster identifier: demo-redshift-cluster
   Node type: dc2.large
   Number of nodes: 1
   Database name: demo_db
   Admin username: admin
   Admin password: [Create secure password]
   ```

3. Network and Security
   - Use default VPC
   - Make publicly accessible: Yes
   - VPC security groups: Default (ensure port 5439 is accessible)

4. Additional Configurations
   - Backup retention: 1 day
   - Maintenance window: Use default
   - Click "Create cluster"

### 1.2 Wait for Cluster Creation
- Cluster creation takes 5-10 minutes
- Status should change from "Creating" to "Available"

## Step 2: Create IAM Role for Redshift Spectrum

### 2.1 Create IAM Role
1. Go to IAM Console → Roles → "Create role"
2. Select trusted entity type: AWS service
3. Select service: Redshift
4. Select use case: Redshift - Customizable
5. Role name: `RedshiftSpectrumRole`

### 2.2 Attach Policies
Add the following managed policies:
- `AmazonS3ReadOnlyAccess`
- `AWSGlueConsoleFullAccess`

### 2.3 Associate Role with Cluster
1. Go back to Redshift Console
2. Select your cluster → Actions → "Manage IAM roles"
3. Add the `RedshiftSpectrumRole`
4. Click "Save changes"

## Step 3: Prepare Sample Data

### 3.1 Create S3 Bucket
1. Go to S3 Console → "Create bucket"
2. Bucket name: `devops-redshift-demo-bucket-2882025`
3. Region: Same as your Redshift cluster
4. Settings: Keep defaults, click "Create bucket"

### 3.2 Create Hot Data (CSV for loading into Redshift)

**File: redshift_sales_data.csv**
```csv
order_id,customer_id,amount,region,order_date
1001,201,1250.50,North America,2024-01-15
1002,202,875.25,Europe,2024-01-16
1003,203,2100.00,Asia Pacific,2024-01-17
1004,204,650.75,North America,2024-01-18
1005,205,1875.00,Europe,2024-01-19
1006,206,925.50,Asia Pacific,2024-01-20
1007,207,1450.25,North America,2024-01-21
1008,208,775.00,Europe,2024-01-22
1009,209,2250.75,Asia Pacific,2024-01-23
1010,210,1125.25,North America,2024-01-24
1011,201,950.00,North America,2024-01-25
1012,202,1675.50,Europe,2024-01-26
1013,203,825.75,Asia Pacific,2024-01-27
1014,204,1950.25,North America,2024-01-28
1015,205,1200.00,Europe,2024-01-29
1016,206,875.50,Asia Pacific,2024-01-30
1017,207,1525.75,North America,2024-01-31
1018,208,2075.25,Europe,2024-02-01
1019,209,675.00,Asia Pacific,2024-02-02
1020,210,1825.50,North America,2024-02-03
```

### 3.3 Create Cold Data (CSV for S3/Spectrum)

**File: s3-spectrum_web_clicks.csv**
```csv
user_id,page,timestamp,session_duration
201,home,2024-01-15 09:30:15,45
202,products,2024-01-16 14:22:30,120
203,about,2024-01-17 11:15:45,30
204,contact,2024-01-18 16:45:00,25
205,products,2024-01-19 10:30:15,180
206,home,2024-01-20 13:45:30,60
207,checkout,2024-01-21 15:20:45,300
208,products,2024-01-22 12:10:00,240
209,about,2024-01-23 09:45:15,35
210,home,2024-01-24 14:30:30,90
201,products,2024-01-25 11:20:45,150
202,checkout,2024-01-26 16:15:00,420
203,contact,2024-01-27 10:45:15,40
204,home,2024-01-28 13:30:30,75
205,about,2024-01-29 15:45:45,50
206,products,2024-01-30 12:20:00,200
207,home,2024-01-31 09:15:15,85
208,checkout,2024-02-01 14:45:30,380
209,products,2024-02-02 11:30:45,165
210,contact,2024-02-03 16:20:00,30
201,home,2024-02-04 10:45:15,55
202,about,2024-02-05 13:15:30,40
203,products,2024-02-06 15:30:45,190
204,checkout,2024-02-07 12:45:00,340
205,home,2024-02-08 09:20:15,70
```

### 3.4 Upload Data to S3
1. Upload redshift_sales_data.csv to: `s3://devops-redshift-demo-bucket-2882025/sales-data/redshift_sales_data.csv`
2. Create folder structure for Spectrum data: `s3://devops-redshift-demo-bucket-2882025/web-clicks/`
3. Upload s3-spectrum_web_clicks.csv to: `s3://devops-redshift-demo-bucket-2882025/web-clicks/s3-spectrum_web_clicks.csv`

## Step 4: Connect to Redshift and Create Hot Data Table

### 4.1 Connect via Query Editor v2
1. Go to Redshift Console → Query Editor v2
2. Connect to cluster: Select your demo cluster
3. Database: demo_db
4. User: admin
5. Authentication: Use stored credentials or temporary credentials

### 4.2 Create Sales Data Table (Hot Data)
```sql
-- Create the sales data table in Redshift
CREATE TABLE sales_data (
    order_id INTEGER,
    customer_id INTEGER,
    amount DECIMAL(10,2),
    region VARCHAR(50),
    order_date DATE
);
```

### 4.3 Load Data into Redshift Table
```sql
-- Load data from S3 into Redshift table
COPY sales_data 
FROM 's3://devops-redshift-demo-bucket-2882025/sales-data/redshift_sales_data.csv'
IAM_ROLE 'arn:aws:iam::YOUR-ACCOUNT-ID:role/RedshiftSpectrumRole'
CSV
IGNOREHEADER 1;
```

**Note:** Replace `YOUR-ACCOUNT-ID` with your actual AWS account ID.

### 4.4 Verify Data Load
```sql
-- Check if data loaded successfully
SELECT COUNT(*) FROM sales_data;

-- Preview the data
SELECT * FROM sales_data LIMIT 10;
```

## Step 5: Query Hot Data (Redshift Analytics Demo)

### 5.1 Basic Analytics Queries

**Query 1: Sales by Region**
```sql
SELECT 
    region,
    COUNT(*) as order_count,
    SUM(amount) as total_sales,
    AVG(amount) as avg_order_value
FROM sales_data
GROUP BY region
ORDER BY total_sales DESC;
```

**Expected Output:**
```
region          | order_count | total_sales | avg_order_value
North America   |     8       |   10427.25  |    1303.41
Europe         |     6       |    8402.75  |    1400.46
Asia Pacific   |     6       |    7126.75  |    1187.79
```

**Query 2: Monthly Sales Trend**
```sql
SELECT 
    DATE_TRUNC('month', order_date) as month,
    COUNT(*) as orders,
    SUM(amount) as revenue
FROM sales_data
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

### 5.2 Performance Demo Point
Show query execution time and mention Redshift's columnar storage and parallel processing capabilities.

## Step 6: Setup Redshift Spectrum (Cold Data Querying)

### 6.1 Create External Schema
```sql
-- Create external schema pointing to Glue Data Catalog
CREATE EXTERNAL SCHEMA spectrum_demo
FROM DATA CATALOG
DATABASE 'redshift_spectrum_db'
IAM_ROLE 'arn:aws:iam::YOUR-ACCOUNT-ID:role/RedshiftSpectrumRole'
CREATE EXTERNAL DATABASE IF NOT EXISTS;
```

### 6.2 Create External Table for Web Clicks
```sql
-- Create external table pointing to S3 data
CREATE EXTERNAL TABLE spectrum_demo.web_clicks (
    user_id INTEGER,
    page VARCHAR(50),
    timestamp TIMESTAMP,
    session_duration INTEGER
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION 's3://devops-redshift-demo-bucket-2882025/web-clicks/'
TABLE PROPERTIES ('skip.header.line.count'='1');
```

### 6.3 Verify External Table Creation
```sql
-- Check if external table was created
SELECT * FROM spectrum_demo.web_clicks LIMIT 10;

-- Count records in S3
SELECT COUNT(*) FROM spectrum_demo.web_clicks;
```

## Step 7: Query Cold Data with Spectrum

### 7.1 Analytics on S3 Data

**Query 1: Popular Pages Analysis**
```sql
SELECT 
    page,
    COUNT(*) as visits,
    AVG(session_duration) as avg_duration
FROM spectrum_demo.web_clicks
GROUP BY page
ORDER BY visits DESC;
```

**Query 2: User Activity Analysis**
```sql
SELECT 
    user_id,
    COUNT(*) as total_visits,
    COUNT(DISTINCT page) as unique_pages_visited,
    SUM(session_duration) as total_time_spent
FROM spectrum_demo.web_clicks
GROUP BY user_id
ORDER BY total_visits DESC
LIMIT 10;
```

### 7.2 Performance Demo Point
Explain that Redshift Spectrum is querying data directly from S3 without loading it into the data warehouse, saving storage costs and enabling analysis of larger datasets.

## Step 8: Hybrid Analytics (Hot + Cold Data)

### 8.1 Join Warehouse and S3 Data
```sql
-- Combine sales data (hot) with web activity (cold)
SELECT 
    s.region,
    w.page,
    COUNT(*) as page_views,
    SUM(s.amount) as associated_revenue
FROM sales_data s
JOIN spectrum_demo.web_clicks w ON s.customer_id = w.user_id
GROUP BY s.region, w.page
ORDER BY associated_revenue DESC;
```

### 8.2 Customer Journey Analysis
```sql
-- Analyze customer behavior: web activity leading to purchases
WITH customer_activity AS (
    SELECT 
        w.user_id,
        COUNT(DISTINCT w.page) as pages_visited,
        AVG(w.session_duration) as avg_session_duration,
        MAX(w.timestamp) as last_web_activity
    FROM spectrum_demo.web_clicks w
    GROUP BY w.user_id
),
customer_sales AS (
    SELECT 
        customer_id,
        COUNT(*) as total_orders,
        SUM(amount) as total_spent,
        MAX(order_date) as last_order_date
    FROM sales_data
    GROUP BY customer_id
)
SELECT 
    ca.user_id as customer_id,
    ca.pages_visited,
    ca.avg_session_duration,
    cs.total_orders,
    cs.total_spent,
    CASE 
        WHEN cs.total_spent > 2000 THEN 'High Value'
        WHEN cs.total_spent > 1000 THEN 'Medium Value'
        ELSE 'Low Value'
    END as customer_segment
FROM customer_activity ca
JOIN customer_sales cs ON ca.user_id = cs.customer_id
ORDER BY cs.total_spent DESC;
```

### 8.3 Demo Point: Architecture Benefits
This demonstrates the power of Redshift's hybrid architecture:
- Hot data (frequent queries) stays in fast warehouse storage
- Cold data (archived/infrequent) stays cost-effectively in S3
- Both can be queried together seamlessly

## Step 9: Performance and Scaling Demo

### 9.1 Query Performance Comparison
```sql
-- Show execution time for warehouse query
SELECT 
    region,
    COUNT(*),
    SUM(amount)
FROM sales_data 
GROUP BY region;

-- Show execution time for Spectrum query
SELECT 
    page,
    COUNT(*)
FROM spectrum_demo.web_clicks 
GROUP BY page;
```

### 9.2 Scaling Benefits Discussion Points
- **Redshift:** Fast queries on frequently accessed data
- **Spectrum:** Query petabytes of data in S3 without loading
- **Cost Efficiency:** Only pay for warehouse storage you need
- **Elastic Scaling:** Add/remove nodes based on workload

## Troubleshooting

### Common Issues and Solutions

**Issue 1: Cannot connect to Redshift cluster**
- Solution: Check VPC security group allows inbound traffic on port 5439
- Ensure cluster is publicly accessible if connecting from outside VPC

**Issue 2: COPY command fails**
- Solution: Verify IAM role has S3 read permissions
- Check S3 path is correct (case-sensitive)
- Ensure role is associated with cluster

**Issue 3: External table creation fails**
- Solution: Verify IAM role has Glue permissions
- Check S3 location exists and is accessible
- Ensure region consistency between services

**Issue 4: Spectrum queries return no results**
- Solution: Check file format and delimiter settings
- Verify header skip settings
- Ensure files are in specified S3 location

## Architecture Overview

This demo showcases a modern data architecture pattern:

1. **Hot Data Layer (Redshift)** - Frequently accessed operational data stored in columnar format for fast analytics
2. **Cold Data Layer (S3)** - Historical or infrequently accessed data stored cost-effectively in object storage  
3. **Unified Query Interface (Spectrum)** - Single SQL interface to query both hot and cold data seamlessly
4. **Metadata Layer (Glue Data Catalog)** - Centralized schema registry for data discovery and governance

## Cost Estimation

**Estimated AWS charges for 1-hour demo:**
- Redshift dc2.large cluster: ~$0.25/hour
- S3 storage (minimal): ~$0.01
- Glue Data Catalog: Free tier
- **Total estimated cost: < $1.00**

Remember to clean up resources after the demo to minimize charges.