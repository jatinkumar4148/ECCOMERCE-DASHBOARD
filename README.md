# E-Commerce Dashboard Project

This repository contains a retail analytics dashboard built on a multi-table e-commerce dataset. The dashboard visuals are stored as images, and the underlying CSV files provide the fact table plus customer, item, time, store, and transaction dimensions.

## Project structure

- `EXCEL DATASET/`
  - `customer_dim.csv`
  - `fact_table.csv`
  - `item_dim.csv`
  - `store_dim.csv`
  - `time_dim.csv`
  - `Trans_dim.csv`
- `Dashboard/`
  - `MONTHLY ANALYSIS 1.png` — monthly sales and revenue summary
  - `CUSTOMERS & PRODUCT INSIGHTS 2.png` — customer retention and product performance insights

## What this project contains

### Datasets
The dataset uses a star schema centered on `fact_table.csv`:
- `fact_table.csv` stores sales transactions with keys to customers, products, time, stores, and payment records.
- `customer_dim.csv` stores customer details.
- `item_dim.csv` stores product metadata.
- `store_dim.csv` stores store geography.
- `time_dim.csv` stores date and time breakdowns.
- `Trans_dim.csv` stores payment method and bank information.

### Dashboard images
The repository includes two dashboard snapshots:
1. **MONTHLY ANALYSIS** — a monthly revenue/quantity trend dashboard with key metrics, top-selling items, and geographic sales distribution.
2. **CUSTOMERS & PRODUCT INSIGHTS** — a customer segmentation and product performance dashboard with retention metrics, product KPIs, bucketed sales, and yearly quantity summaries.

## Dataset schema summary

### `customer_dim.csv`
Columns:
- `coustomer_key` — customer ID
- `name` — customer name
- `contact_no` — phone number
- `nid` — national ID number

Sample rows:
- `C000001, sumit, 8801920345851, 7505075708899`
- `C000002, tammanne, 8801817069329, 1977731324842`

### `fact_table.csv`
Columns:
- `payment_key` — payment reference ID
- `coustomer_key` — customer foreign key
- `time_key` — time dimension foreign key
- `item_key` — item foreign key
- `store_key` — store foreign key
- `quantity` — number of units sold
- `unit` — quantity unit label
- `unit_price` — sale price per unit
- `total_price` — line total revenue

Sample rows:
- `P026, C004510, T049189, I00177, S00307, 1, ct, 35, 35`
- `P022, C008967, T041209, I00248, S00595, 1, rolls, 26, 26`

### `item_dim.csv`
Columns:
- `item_key` — item ID
- `item_name` — product name
- `desc` — product category or description
- `unit_price` — list/unit price
- `man_country` — manufacturer country
- `supplier` — supplier name
- `unit` — product unit type

Sample rows:
- `I00001, A&W Root Beer - 12 oz cans, a. Beverage - Soda, 11.5, Netherlands, Bolsius Boxmeer, cans`
- `I00002, A&W Root Beer Diet - 12 oz cans, a. Beverage - Soda, 6.75, poland, CHROMADURLIN S.A.S, cans`

### `store_dim.csv`
Columns:
- `store_key` — store ID
- `division` — region/division
- `district` — district name
- `upazila` — sub-district name

Sample rows:
- `S0001, SYLHET, HABIGANJ, AJMIRIGANJ`
- `S0002, SYLHET, HABIGANJ, BAHUBAL`

### `time_dim.csv`
Columns:
- `time_key` — time ID
- `date` — full date/time string
- `hour` — hour of day
- `day` — day of month
- `week` — weekly bucket
- `month` — month number
- `quarter` — quarter label
- `year` — year

Sample rows:
- `T00001, 20-05-2017 14:56, 14, 20, 3rd Week, 05, Q2, 2017`
- `T00002, 30-01-2015 22:14, 22, 30, 4th Week, 01, Q1, 2015`

### `Trans_dim.csv`
Columns:
- `payment_key` — payment reference ID
- `trans_type` — cash or card
- `bank_name` — bank name when card payment

Sample rows:
- `P001, cash, None`
- `P002, card, AB Bank Limited`

## What I learned from the data and dashboards

- The project uses a clean star schema where `fact_table.csv` can be joined to each dimension table using foreign keys.
- The dashboards are designed to show monthly revenue/quantity trends, top-performing items, geographic concentration, and customer/product performance.
- Key dashboard signals:
  - strong monthly sales trend with seasonal patterns;
  - top-selling items drive most quantities;
  - a dominant location appears in the geographic view;
  - customer retention appears low, making customer loyalty a useful focus area;
  - product insights highlight unit price, total price, and quantity distribution.

## Recommended analysis flow

1. Load the datasets.
2. Join `fact_table.csv` to dimensions:
   - customer details via `coustomer_key`
   - product details via `item_key`
   - date details via `time_key`
   - store geography via `store_key`
   - payment method via `payment_key`
3. Create metrics:
   - revenue by month, quarter, and year
   - quantity and revenue by product
   - revenue by division / district
   - customer purchase frequency and retention
   - payment type mix by bank and transaction method

## Example code snippet

```python
import pandas as pd

customers = pd.read_csv('EXCEL DATASET/customer_dim.csv')
items = pd.read_csv('EXCEL DATASET/item_dim.csv')
stores = pd.read_csv('EXCEL DATASET/store_dim.csv')
time = pd.read_csv('EXCEL DATASET/time_dim.csv')
trans = pd.read_csv('EXCEL DATASET/Trans_dim.csv')
orders = pd.read_csv('EXCEL DATASET/fact_table.csv')

# Normalize date
orders = orders.merge(time, on='time_key', how='left')
orders['date'] = pd.to_datetime(orders['date'], format='%d-%m-%Y %H:%M', errors='coerce')

# Enrich orders
orders = (
    orders
    .merge(customers, on='coustomer_key', how='left')
    .merge(items, on='item_key', how='left')
    .merge(stores, on='store_key', how='left')
    .merge(trans, on='payment_key', how='left')
)

# Monthly revenue
monthly_revenue = (
    orders
    .groupby(pd.Grouper(key='date', freq='M'))
    .agg(total_revenue=('total_price', 'sum'), total_quantity=('quantity', 'sum'))
    .reset_index()
)
print(monthly_revenue.head())
```

## Interactive dashboard recommendation

To convert this repo into an interactive dashboard:
- use **Streamlit** or **Dash** for web deployment;
- use **Plotly** for charts and filters;
- add interactive selectors for date ranges, product categories, store locations, and payment methods;
- show a KPI panel for revenue, quantity, average unit price, and customer retention.

## How to run a quick local prototype

```bash
python -m venv .venv
.venv\Scripts\activate
pip install pandas streamlit plotly
streamlit run app.py
```

> If you want, I can also scaffold the complete interactive `app.py` now using the current dataset and dashboard concepts.

## Notes

- The dataset exhibits a typo in the customer key field: `coustomer_key`.
- `time_dim.csv` uses `dd-mm-yyyy HH:MM` formatting, so normalizing dates is important.
- `Trans_dim.csv` includes a transaction channel and bank data, which is useful for payment analytics.
