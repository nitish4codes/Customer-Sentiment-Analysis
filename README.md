# Final Project: Customer Sentiment Analysis Pipeline

By: Nitish Bhardwaj 

College: Amity University, Noida

An end-to-end production-grade data engineering pipeline built using the **Medallion Architecture (Bronze → Silver → Gold)**. This project streams and processes customer call center logs alongside historical customer account data to generate live business KPIs and sentiment tracking metrics.

Developed as the capstone project for the **Celebal Technologies Data Engineering Internship**.

## Architecture Design & Data Flow
The pipeline processes incoming telemetry data dynamically, ensuring high data fidelity, state tracking, and analytical isolation across three distinct layers:

*   **Bronze Layer:** Ingests raw, unstructured call log files incrementally from a monitored landing zone (`/Workspace/mock_stream_calls/`) alongside a historical snapshot of the raw customer change-data-capture (CDC) logs.
*   **Silver Layer:** Parses incoming nested JSON records, flattens relational paths, applies a 10-minute event-time watermark to tolerate late data, and deduplicates the streaming data. Concurrently, customer CDC updates are modeled into a **Slowly Changing Dimension Type 2 (SCD Type 2)** table to retain comprehensive profile histories over time.
*   **Gold Layer:** Joins the clean, streaming Silver call entries with the active customer profiles using a **Point-in-Time (PIT) Join condition** ($event\_ts \ge start\_date \text{ AND } event\_ts \le end\_date$). The pipeline aggregates these records to output 5 serverless-compliant business KPI tables.

## Tech Stack & Advanced Engineering Features
*   **Core Engine:** Apache Spark / PySpark (Structured Streaming API)
*   **Storage Architecture:** Delta Lake (ACID Transactions, Strict Schema Enforcement)
*   **State Management:** Streaming Deduplication & Event-Time Watermarking
*   **Data Modeling:** Slowly Changing Dimensions (SCD Type 2) & Point-in-Time Joins
*   **Compute Workspace:** Databricks Serverless Compute using a fully qualified Unity Catalog Namespace (`workspace.default.*`)

---

## Continuously Updated Business KPIs (Gold Layer)
The pipeline automatically computes and overwrites 5 specialized business metric tables inside the catalog:
1. **`gold_sentiment_daily`**: Tracks the overall shift in customer call center sentiment (Positive, Neutral, Negative) on a daily basis.
2. **`gold_call_volume_daily`**: Captures call volume scales and calculates running average call durations to evaluate support performance.
3. **`gold_high_risk_customers`**: Flags customer profiles experiencing repeated Negative sentiment interactions for proactive account outreach.
4. **`gold_region_sentiment`**: Groups sentiment trends by major metropolitan regions (e.g., Delhi, Bangalore, Mumbai) to reveal regional performance gaps.
5. **`gold_subscription_sentiment`**: Cross-references customer satisfaction matrices across subscription tiers (Premium, Standard, Free) to monitor tier health.

---

## 🚀 How to Run and Verify the Pipeline
The pipeline is engineered for strict linear reproducibility. A reviewer can execute the pipeline end-to-end with a single click:

1. Import the project notebook into your Databricks workspace.
2. Ensure the base customer data is available in the catalog as `workspace.default.customer_cdc_data_final`.
3. Click **Run All** at the top of the notebook.
4. The notebook will automatically run through 5 structured phases:
    *   **Cell 1:** Global imports, filesystem initialization, and constants mapping.
    *   **Cell 2:** Background streaming initialization for the raw Bronze ingestion stream.
    *   **Cell 3:** Mock streaming data file generation to simulate active call center records.
    *   **Cell 4:** Running the Silver parsing operations and processing the SCD Type 2 table mapping.
    *   **Cell 5:** Running the final Point-in-Time joins to compute and display live Gold KPI results.