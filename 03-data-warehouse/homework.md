# Module 3 Homework: Data Warehouse

## Question 1:
Question 1: What is count of records for the 2024 Yellow Taxi Data?
- 65,623
- 840,402
- 20,332,093 -- Correct
- 85,431,289

```sql
-- Calculate records in 2024 --
SELECT COUNT(1) FROM copper-diorama-448706-a3.zoomcamp.yellow_tripdata_2024_non_partitoned;
```

## Question 2:
Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.</br> 
What is the **estimated amount** of data that will be read when this query is executed on the External Table and the Table?

- 18.82 MB for the External Table and 47.60 MB for the Materialized Table
- 0 MB for the External Table and 155.12 MB for the Materialized Table -- Correct
- 2.14 GB for the External Table and 0MB for the Materialized Table
- 0 MB for the External Table and 0MB for the Materialized Table

This is because BigQuery:
  - Knows materialized table size before execution
  - Cannot estimate external table size until scan
  - Shows 0 MB estimate for external tables initially

```sql
-- Count the distinct number of PULocationIDs for external table --
SELECT COUNT(DISTINCT(PULocationID)) FROM copper-diorama-448706-a3.zoomcamp.yellow_tripdata_2024_external;

-- Count the distinct number of PULocationIDs for materialized table --
SELECT COUNT(DISTINCT(PULocationID)) FROM copper-diorama-448706-a3.zoomcamp.yellow_tripdata_2024_non_partitoned;
```

## Question 3:
Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table. Why are the estimated number of Bytes different?
- BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed. -- Correct
- BigQuery duplicates data across multiple storage partitions, so selecting two columns instead of one requires scanning the table twice, doubling the estimated bytes processed.
- BigQuery automatically caches the first queried column, so adding a second column increases processing time but does not affect the estimated bytes scanned.
- When selecting multiple columns, BigQuery performs an implicit join operation between them, increasing the estimated bytes processed

155.12 MB -- PULocationID  
310.24 MB -- PULocationID, DOLocationID

```sql
-- retrieve the PULocationID from the table --
SELECT PULocationID FROM copper-diorama-448706-a3.zoomcamp.yellow_tripdata_2024_non_partitoned;

-- retrieve the PULocationID and DOLocationID from the table --
SELECT PULocationID, DOLocationID FROM copper-diorama-448706-a3.zoomcamp.yellow_tripdata_2024_non_partitoned;
```

## Question 4:
How many records have a fare_amount of 0?
- 128,210
- 546,578
- 20,188,016
- 8,333 -- Correct

```sql
-- count of records with fare_amount of 0 --
SELECT COUNT(1) FROM copper-diorama-448706-a3.zoomcamp.yellow_tripdata_2024_non_partitoned WHERE fare_amount = 0;
```

## Question 5:
What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)
- Partition by tpep_dropoff_datetime and Cluster on VendorID -- Correct
- Cluster on by tpep_dropoff_datetime and Cluster on VendorID
- Cluster on tpep_dropoff_datetime Partition by VendorID
- Partition by tpep_dropoff_datetime and Partition by VendorID

```sql
-- Create a partitioned table by tpep_dropoff_datetime and clustered by VendorID from external table --
CREATE OR REPLACE TABLE copper-diorama-448706-a3.zoomcamp.yellow_tripdata_2024_partitoned
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM copper-diorama-448706-a3.zoomcamp.yellow_tripdata_2024_external;
```


## Question 6:
Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime
2024-03-01 and 2024-03-15 (inclusive)</br>

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values? </br>

Choose the answer which most closely matches.</br> 

- 12.47 MB for non-partitioned table and 326.42 MB for the partitioned table
- 310.24 MB for non-partitioned table and 26.84 MB for the partitioned table -- Correct
- 5.87 MB for non-partitioned table and 0 MB for the partitioned table
- 310.31 MB for non-partitioned table and 285.64 MB for the partitioned table

310.24 MB -- non-partitioned table  
26.84 MB -- partitioned table

```sql
-- retrieve the distinct VendorIDs between tpep_dropoff_datetime 2024-03-01 and 2024-03-15 (inclusive) for materialized table --
SELECT DISTINCT(VendorID) FROM copper-diorama-448706-a3.zoomcamp.yellow_tripdata_2024_non_partitoned WHERE tpep_dropoff_datetime > '2024-03-01' AND tpep_dropoff_datetime <= '2024-03-15';

-- retrieve the distinct VendorIDs between tpep_dropoff_datetime 2024-03-01 and 2024-03-15 (inclusive) for partitioned table --
SELECT DISTINCT(VendorID) FROM copper-diorama-448706-a3.zoomcamp.yellow_tripdata_2024_partitoned WHERE tpep_dropoff_datetime > '2024-03-01' AND tpep_dropoff_datetime <= '2024-03-15';
```

## Question 7: 
Where is the data stored in the External Table you created?

- Big Query
- Container Registry
- GCP Bucket -- Correct
- Big Table

Source URI(s): `gs://copper-diorama-448706-a3_kestra/yellow_tripdata_2024-*.parquet` -> GCP Bucket

## Question 8:
It is best practice in Big Query to always cluster your data:
- True
- False -- Correct

Why clustering isn't always the best practice:
1. Table Size Considerations
   - Tables < 1 GB don't benefit from clustering
   - Small tables may see increased overhead
   - Cost-Benefit Analysis

2. Cost-Benefit Analysis
   - Clustering adds maintenance costs
   - Storage costs increase
   - Performance gains must justify overhead
  
3. When to Avoid Clustering:
   - Small datasets
   - Tables with frequent updates
   - Full table scans
   - Cost-sensitive scenarios


## (Bonus: Not worth points) Question 9:
No Points: Write a `SELECT count(*)` query FROM the materialized table you created. How many bytes does it estimate will be read? Why?

```sql
-- SELECT count(*) from materialized table --
SELECT COUNT(*) FROM copper-diorama-448706-a3.zoomcamp.yellow_tripdata_2024_non_partitoned;
```

It estimates 0 bytes will be read because the query is a metadata operation that doesn't require scanning the table's data.
