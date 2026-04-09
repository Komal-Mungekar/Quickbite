# QuickBite — End to End Food Delivery Data Pipeline

## Problem Statement

QuickBite is a fictional food delivery platform operating across major Indian cities. Every day, thousands of orders are placed by customers ordering from restaurants and fulfilled by delivery agents. The raw order data lands as daily CSV files in a landing zone.

The goal of this project is to build a reliable, end to end batch data pipeline on Databricks that transforms raw, messy source data into clean, analytics ready Delta tables, enabling the business to answer questions like - 

- Which city generates the most revenue per month?
- Which restaurants have the highest cancellation rate?
- What is the average delivery time by city and vehicle type?
- Which payment mode is most popular?
- Who are the top 10 restaurants by revenue?

---

## Architecture

This project follows the Medallion Architecture - a modern data engineering pattern with three progressive layers - 

```
Raw CSVs → [Bronze] → [Silver] → [Gold] → Dashboard / BI Tool
```

| Layer  | Purpose                                                   |
|--------|-----------------------------------------------------------|
| Bronze | Raw ingestion — data stored exactly as received           |
| Silver | Cleaned & transformed — data quality issues resolved      |
| Gold   | Business-ready — metrics calculated, dimensions joined    |

---

## Tech Stack

| Tool            | Purpose                                      |
|-----------------|----------------------------------------------|
| Databricks      | Cloud data platform to run notebooks         |
| Apache Spark    | Distributed data processing engine           |
| PySpark         | Python API for Spark                         |
| Delta Lake      | Versioned, reliable table storage format     |
| SQL             | Querying and view creation                   |
| Python          | Data generation scripts                      |

---

## Dataset Overview

### Fact Table (daily, incremental)
| File | Description |
|------|-------------|
| `orders/landing/orders_YYYY-MM-DD.csv` | Daily order lines — one file per day, 90 days of data (Jan–Mar 2025) |

**Columns:** `dt`, `order_ts`, `order_id`, `item_seq`, `customer_id`, `restaurant_id`, `agent_id`, `item_id`, `quantity`, `item_price`, `discount_pct`, `delivery_fee`, `delivery_time_mins`, `payment_mode`, `order_status`, `city`

### Dimension Tables (static)
| File | Rows | Description |
|------|------|-------------|
| `restaurants.csv` | 20 | Restaurant master — name, city, cuisine, rating |
| `customers.csv` | 200 | Customer master — name, phone, city, signup date |
| `delivery_agents.csv` | 15 | Agent master — name, vehicle type, city |
| `menu_items.csv` | 20 | Menu item master — name, category, base price |
| `date.csv` | 365 | Date dimension — year, month, quarter, week, weekend flag |

---

## Data Quality Issues (intentionally introduced)

These are the kinds of real-world problems data engineers deal with every day:

| Table | Issue | Fix Applied |
|-------|-------|-------------|
| `orders` | Duplicate rows (same order_id + item_seq) | `dropDuplicates()` |
| `orders` | `item_price` has `₹` prefix or spaces | `regexp_replace()` + cast |
| `orders` | `delivery_time_mins` has negative values | `abs()` |
| `orders` | NULL `agent_id` for cancelled orders | `fillna("UNASSIGNED")` |
| `orders` | Mixed city casing (`mumbai`, `DELHI`) | `initcap()` |
| `restaurants` | Spelling mistakes in `cuisine_type` (`Nort Indian`, `Italain`) | `replace()` dictionary |
| `customers` | NULL `phone` values | `fillna("Not Available")` |
| `customers` | Mixed city casing | `initcap()` |
| `delivery_agents` | Mixed `vehicle_type` casing (`bike`, `Bike`) | `initcap()` |
| `date` | Negative `week_of_year` values | `abs()` |
| `date` | Mixed `day_name` casing | `initcap()` |

---

## Gold Layer — Calculated Metrics

| Column | Calculation |
|--------|-------------|
| `gross_amount` | `quantity × item_price` |
| `discount_amount` | `gross_amount × discount_pct / 100` |
| `net_amount` | `gross_amount − discount_amount + delivery_fee` |
| `is_cancelled` | `1` if `order_status = 'Cancelled'`, else `0` |
| `date_id` | Integer key (`yyyyMMdd`) for joining with date dimension |
| `hour_of_day` | Extracted from `order_ts` for peak hour analysis |

---

## Project Structure

```
quickbite/
├── README.md
├── data/
│   ├── orders/landing/          ← 90 daily CSV files (orders_2025-01-01.csv ...)
│   ├── restaurants/
│   ├── customers/
│   ├── delivery_agents/
│   ├── menu_items/
│   └── date/
└── notebooks/
    ├── 1_setup/
    ├── 2_dim_bronze/
    ├── 3_dim_silver/
    ├── 4_dim_gold/
    ├── 5_fact_bronze/
    ├── 6_fact_silver/
    └── 7_fact_gold/
```
