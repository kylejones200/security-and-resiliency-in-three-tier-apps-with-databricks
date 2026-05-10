---
author: "Kyle Jones"
date_published: "September 18, 2025"
date_exported_from_medium: "November 10, 2025"
canonical_link: "https://medium.com/@kyle-t-jones/security-and-resiliency-in-three-tier-apps-with-databricks-593003409466"
---

# Security and Resiliency in Three-Tier Apps with Databricks Three-tier architecture has lasted because it creates separation of
concerns. The presentation tier talks to the application tier. The...

### Security and Resiliency in Three-Tier Apps with Databricks 

Three-tier architecture has lasted because it creates separation of concerns. The presentation tier talks to the application tier. The application tier talks to the data tier. Each layer is controlled. Each layer is secure. Each layer can fail independently.

But in practice, the data tier is the hardest to secure and the easiest to break. Poor access control exposes sensitive data. Batch pipelines fail. Outages ripple upward to apps and users.

Databricks flips this script. By unifying data, governance, and compute on one platform, it makes the data tier the strongest part of the three-tier stack. Security is enforced end-to-end. Resiliency is built in at scale.

### Security by Design
Here's the modern three-tier app with Databricks and Unity Catalog:


Unity Catalog is the foundation. It applies fine-grained, role-based access control (RBAC) across tables, models, and features. That means there are no direct queries to raw data and service accounts only have access to the assets they need. Unity Catalog maintains full tracking for lineage so you know how data flows into and out of apps. This is important for insuring comliance and for enforcing data quality standards.

### Example: Secure Access with Unity Catalog
Imagine you have a fraud detection app \[[like this](https://medium.com/@kyle-t-jones/hosting-web-apps-directly-on-databricks-e932b7400c0c)\]. You want the app to query transactions but not write them. You also want data scientists to train models but not deploy apps. Unity Catalog makes this explicit.

``` 
-- Create groups
CREATE GROUP app_users;
CREATE GROUP data_scientists;

-- Grant read-only access to app
GRANT SELECT ON TABLE transactions TO app_users;
-- Grant ML privileges to data scientists
GRANT SELECT, MODIFY ON TABLE transactions TO data_scientists;
GRANT EXECUTE ON FUNCTION churn_model TO data_scientists;
-- Deny all else by default
```

This ensures least-privilege access. The application only sees what it needs. The data scientists get broader rights, but still within Databricks.

### Built-In Resiliency
Databricks inherits the resiliency of the underlying cloud (AWS, Azure, GCP). That means multi-zone clusters, automatic failover, and elastic scaling. But resiliency goes deeper in the data tier itself.

- **Delta Lake** guarantees ACID transactions. If a pipeline fails mid-write, the table remains consistent.
- **Checkpointing** in structured streaming ensures no double-counting or data loss.
- **Auto-scaling clusters** handle bursts of load without downtime.
- **Versioned data** enables time travel queries for fast recovery.

### Example: Resilient Streaming Pipeline
```python
from pyspark.sql.types import StructType, StringType, DoubleType, TimestampType

schema = StructType() \
    .add("transactionId", StringType()) \
    .add("userId", StringType()) \
    .add("amount", DoubleType()) \
    .add("timestamp", TimestampType())
stream = spark.readStream \
    .format("cloudFiles") \
    .option("cloudFiles.format", "json") \
    .schema(schema) \
    .load("/mnt/transactions")
stream.writeStream \
    .format("delta") \
    .option("checkpointLocation", "/mnt/checkpoints/transactions") \
    .outputMode("append") \
    .table("transactions")
```

If the pipeline fails, Databricks resumes from the checkpoint. Transactions are not lost. Delta ensures atomicity. The app querying the `transactions` table never sees partial results.

### Architecture in Practice
Let's look at the secure, resilient three-tier app on Databricks:


This design keeps all sensitive data and ML models in the governed environment. The app consumes only the secure outputs. If a pipeline stalls, checkpoints and Delta keep the data consistent. If a node fails, the cluster self-heals. Security and resiliency are why the three-tier model remains strong. But where older architectures falter in the data tier, Databricks strengthens it. With Unity Catalog, Delta Lake, and built-in resiliency, the data tier becomes the most secure and reliable part of your stack.

That means your apps are safer, faster, and more dependable.
