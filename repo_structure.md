# dbt Project

**1. Staging Layer `staging`**
- ****Purpose****: The staging layer is where you bring raw data from the source tables into a format that’s more suited for transformation. It typically contains very minimal business logic and is intended to clean, normalize, and standardize data to ensure consistency.
- ****Tables/Views****: These are usually raw, one-to-one mappings from source systems. The goal is to prepare the data (e.g., handle missing values, type casting, renaming columns) for downstream transformations.
- ****Examples****:
    - `stg_salesforce_accounts`
    - `stg_google_ads_campaigns`
- ****Key Characteristics****:
    - Minimal transformation.
    - Basic cleaning (e.g., type casting, renaming).
    - Typically one model per source table or object.
- ****Materialization****: `view`
    - Faster development cycles (no rebuilding of tables), ensures freshness of raw data, easy debugging
  
**1. Intermediate Layer `intermediate`**
- ****Purpose****: The intermediate layer serves as a transformation point where you apply more complex logic and business rules. You may join different staging tables, calculate derived columns, or clean up data further. These tables or views often serve as the foundation for more refined transformations.
- ****Tables/Views****: These models typically combine data from different staging tables, create intermediate aggregations, and serve as stepping stones for more complex transformations.
- ****Examples****:
    - `int_customer_sales`
    - `int_product_inventory`
- ****Key Characteristics****:
    - More complex transformations.
    - Typically used for intermediate joins and aggregations.
    - Can be used to calculate metrics or metrics components.
- ****Materialization****: `view` or `table` (depending on size and frequency)
    - For smaller or less frequently updated data, use `view`
    - For larger, frequently queried data, use `table` (especially if the transformation process is intensive).
  
**1. Mart Layer `mart`**
- ****Purpose****: The mart layer is where you create business-relevant aggregates, metrics, and KPIs. This layer prepares data specifically for business analysis and reporting. Instead of traditional fact and dimension tables, focus on creating aggregate data that answers specific business questions or provides insight into key performance indicators (KPIs)
- ****Tables/Views****: These models are generally business-relevant aggregates.
- ****Examples****:
    - `mart_customers`
    - `mart_pipeline`
- ****Key Characteristics****:
    - Aggregated business metrics and KPIs.
    - Tables optimized for reporting or analytical queries.
    - No complex dimensional modeling; focus is on business metrics
- ****Materialization****: `table` or `incremental` (depending on size and frequency)
    - For smaller regularly queried models, use `view`
    - For large models with periodic updates, use `incremental` for better performance during refreshes.

**1. Dashboard Layer `dashboard`**
- ****Purpose****: The dashboard layer is the final layer used for BI and reporting. It takes the data from previous layers and optimizes it for consumption by BI tools (e.g., Tableau). This layer will include pre-aggregated metrics, KPIs, and any additional formatting needed for dashboards or visualizations
- ****Tables/Views****: Tables or views that contain the final data required for reporting tools. These are optimized for BI tools and often include high-level aggregations, time-based calculations (e.g., year-over-year comparisons), and business summaries.
- ****Examples****:
    - `dash_sales_performance`
    - `dash_marketing_roi`
- ****Key Characteristics****:
    - Business intelligence-ready.
    - Optimized for easy visualization.
    - Pre-aggregated data tailored to KPIs and business reports.
 - ****Materialization****: `table` or `view` (depending on size and frequency)
    - For queries that are complex and require a significant amount of commputation, use `table`
    - When live data is necessary, use `view`
