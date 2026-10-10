# GA4 BigQuery Time Lag Analysis: First Visit to First Purchase

A production-grade BigQuery SQL model for measuring customer purchasing velocity (time lag in days and hours) from initial website discovery (`first_visit`) to initial conversion (`purchase`) using Google Analytics 4 (GA4) export data.

---

## 📌 Business Value & Use Cases

Understanding the time lag between brand acquisition and initial transaction allows e-commerce and marketing teams to:
* **Evaluate Channel Velocity:** Compare how quickly paid search, paid social, and organic traffic convert.
* **Optimize Retargeting Windows:** Set data-driven window durations for cart recovery and post-discovery remarketing campaigns.
* **Differentiate Same-Day Purchases:** Measure both `days_to_purchase` and `hours_to_purchase` to isolate instant conversions from multi-day consideration cycles.

---

## 📁 Repository Structure

├── 2.0_time-lag-first-visit-to-purchase.sql
└── README.md

---

## ⚡ Key Architecture & Optimizations

1. **Unbiased Cross-Campaign Joining:** Joins events exclusively on customer identity (`customer_id`), preventing conversion drops if a user converts under a different traffic source or campaign parameter than their initial visit.
2. **Cross-Device Stitching:** Utilizes `COALESCE(user_id, user_pseudo_id)` to unify anonymous client IDs with authenticated account logins.
3. **Partition Pruning:** Enforces strict `_TABLE_SUFFIX` date filtering to minimize BigQuery byte scanning costs.
4. **Chronological Safeguards:** Ensures purchases are evaluated chronologically on or after the first recorded visit timestamp (`p.purchase_timestamp >= v.first_visit_timestamp`).

---

## 📊 Summary Output Schema

Running the query generates a clean transaction latency log:

| Column | Type | Description |
| :--- | :--- | :--- |
| `customer_id` | STRING | Stitched user identifier (`user_id` or `user_pseudo_id`). |
| `first_visit_date` | DATE | Date of initial website visit. |
| `first_visit_source` | STRING | First-touch traffic source (e.g., google, facebook). |
| `first_visit_medium` | STRING | First-touch traffic medium (e.g., cpc, paidsocial). |
| `first_visit_campaign` | STRING | First-touch marketing campaign name. |
| `order_id` | STRING | Unique transaction ID of the first purchase. |
| `purchase_date` | DATE | Date of the initial transaction. |
| `revenue` | NUMERIC | Monetary value generated on first purchase. |
| `days_to_purchase` | INTEGER | Full calendar days elapsed between visit and purchase. |
| `hours_to_purchase` | INTEGER | Precise elapsed hours between first visit and purchase. |

---

## 🛠 Prerequisites & Setup

1. Replace `enter.tablename_123456.events_*` with your GA4 BigQuery export table path.
2. Adjust the historical query window in `_TABLE_SUFFIX` (default: last 24 months) as needed.
