# OVERVIEW

A production-grade BigQuery SQL model for measuring customer purchasing velocity and time lag (in days and hours) from initial website discovery (`first_visit`) to initial conversion (`purchase`) using Google Analytics 4 (GA4) export data.

## REPOSITORY

### 1. 1.0_time-lag-first-visit-to-purchase

- **Approach:** Multi-CTE isolation with unbiased customer-level joining and dual timestamp latency evaluation.
- **Output Metrics:** `days_to_purchase` (`DATE_DIFF`) and `hours_to_purchase` (`TIMESTAMP_DIFF`).
- **Best Used For:** Measuring channel acquisition velocity, evaluating marketing remarketing windows, and separating same-day impulse buys from multi-day consideration cycles.

## SQL OPTIMISATION & PERFORMANCE 

The query in this repository follows production-grade performance and hygiene standards:

1. **Unbiased Identity Join:** Joins events strictly on `v.customer_id = p.customer_id` without filtering on traffic source strings, preventing conversion record drops when a user's campaign tags change between discovery and purchase.
2. **Cross-Device Stitching:** Integrates `COALESCE(user_id, user_pseudo_id)` across both visit and conversion CTEs to unify guest client cookies with authenticated user accounts.
3. **Partition Pruning:** Enforces strict `_TABLE_SUFFIX` date range bounds to minimize BigQuery byte scans over raw event tables.
4. **Chronological Safeguards:** Restricts outputs to valid transactions (`ecommerce.transaction_id IS NOT NULL`) where conversion occurred on or after initial discovery (`p.purchase_timestamp >= v.first_visit_timestamp`).

## SUMMARY OUTPUT SCHEME

Running this query generates a clean transaction-level latency log:

| Column | Type | Description |
| :--- | :--- | :--- |
| `customer_id` | STRING | Stitched user identifier (`user_id` or `user_pseudo_id`). |
| `first_visit_date` | DATE | Calendar date of initial website discovery. |
| `first_visit_source` | STRING | Acquisition source captured on first visit. |
| `first_visit_medium` | STRING | Acquisition medium captured on first visit. |
| `first_visit_campaign` | STRING | Acquisition campaign name captured on first visit. |
| `order_id` | STRING | Unique transaction identifier of the first purchase. |
| `purchase_date` | DATE | Calendar date of the initial conversion. |
| `revenue` | NUMERIC | Transaction revenue generated on first purchase. |
| `days_to_purchase` | INTEGER | Calendar days elapsed from first visit to purchase. |
| `hours_to_purchase` | INTEGER | Precise elapsed hours from first visit to purchase. |
