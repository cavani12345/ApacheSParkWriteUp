# ApacheSParkWriteUp

Key Methods for Parallel Extraction

JDBC / RDBMS Parallel Reading:
	• Provide a numeric partition column, lower/upper bounds, and the number of partitions so Spark spins up concurrent JDBC connections for non-overlapping data chunks.

• Use custom predicate filters (e.g., splitting a table by date ranges or ID buckets) when a single numeric column is unavailable.


Core Tuning Parameters

• numPartitions: Controls how many concurrent tasks read from a JDBC or source system. Match this to your cluster core capacity and database connection limits.
• spark.sql.shuffle.partitions: Controls the default parallelism for wide transformations (defaults to 200, which should be scaled down or up depending on your data size and core count).
• Fair Scheduler: Enable spark.scheduler.mode = "FAIR" to prevent a massive extraction job from choking all available cluster resources.
If you share your data source (e.g., PostgreSQL, REST API, S3/Parquet) and approximate volume, I can provide a sample code snippet tailored to your setup.



1. The Strategy: Two-Level Parallelism

To handle hundreds of 200M+ record tables without overloading your system, you must break the problem into two distinct parallel layers:
1. Table-Level Parallelism (Orchestration): Avoid extracting all hundreds of tables at the exact same moment. Instead, use an orchestrator (such as Airflow) to process tables in small concurrent batches (e.g., 5 to 10 tables running simultaneously).


2. Row-Level Parallelism (Spark Drivers/Executors): For each table being actively extracted, distribute the row reading across multiple Spark executor cores by generating non-overlapping SQL queries via JDBC slicing.


2. Core Tuning Parameters for Oracle Ingestion

Parameter	Recommended Value	Impact
partitionColumn	A numeric ID, integer, or highly uniform DATE/TIMESTAMP.	Dictates which column Spark will use to split the WHERE clauses.

numPartitions	100 to 400 (per table)	Spawns this exact number of concurrent Oracle JDBC connections per table. Do not exceed your Oracle DBA's max connection pool limits.


fetchsize	5000 to 20000	Crucial for Oracle. The Oracle JDBC driver defaults to fetching only 10 rows at a time, slowing ingestion down to a crawl. Setting this higher reduces network round trips dramatically. Reduce this toward 500–1000 if a table is incredibly "wide" (hundreds of text columns) to avoid executor OOMs.

lowerBound / upperBound	Min and Max values of the partitionColumn	Defines the boundaries Spark uses to split data evenly. Always query these values dynamically from the database right before reading.


https://www.linkedin.com/top-content/science/best-practices-for-data-management/best-practices-for-repartitioning-in-spark/

https://medium.com/@suffyan.asad1/spark-parallelization-of-reading-data-from-jdbc-sources-e8b35e94cb82


