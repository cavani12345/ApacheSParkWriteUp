# ApacheSParkWriteUp

Key Methods for Parallel Extraction

JDBC / RDBMS Parallel Reading:
	• Provide a numeric partition column, lower/upper bounds, and the number of partitions so Spark spins up concurrent JDBC connections for non-overlapping data chunks.
	• Use custom predicate filters (e.g., splitting a table by date ranges or ID buckets) when a single numeric column is unavailable.
