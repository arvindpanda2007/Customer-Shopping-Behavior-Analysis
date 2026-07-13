# Customer Shopping Behavior Analysis

An end-to-end retail analytics project that takes 3,900 raw customer transactions through cleaning and feature engineering in Python, structured business-question analysis in SQL, and an interactive Power BI dashboard — wrapping up in a written report and stakeholder presentation.

## Business problem

A retail company wants to understand shifts in customer purchasing patterns across demographics, product categories, and channels, and to know which levers — discounts, reviews, seasonality, payment method — actually drive repeat purchases and loyalty. The goal of this project is to turn the raw transaction log into concrete, defensible recommendations a marketing or product team could act on.

## Dataset

`customer_shopping_behavior.csv` — 3,900 rows, 18 columns:

| Column | Description |
|---|---|
| Customer ID | Unique customer identifier |
| Age | Customer age |
| Gender | Customer gender |
| Item Purchased | Product name |
| Category | Product category (Clothing, Footwear, Outerwear, Accessories) |
| Purchase Amount (USD) | Transaction value |
| Location | US state |
| Size / Color / Season | Product attributes |
| Review Rating | 1–5 product rating (37 missing values, imputed) |
| Subscription Status | Whether the customer is subscribed |
| Shipping Type | Standard, Express, Next Day Air, Free Shipping, etc. |
| Discount Applied / Promo Code Used | Whether a discount/promo was used on the order (found to be fully redundant — see below) |
| Previous Purchases | Count of prior orders by that customer |
| Payment Method | Card, PayPal, Venmo, Cash, etc. |
| Frequency of Purchases | Self-reported cadence (Weekly, Fortnightly, Monthly, ...) |

## Repo contents

| File | Purpose |
|---|---|
| `Customer_Shopping_Behavior_Analysis.ipynb` | Data cleaning, feature engineering, and DB load in Python (pandas + SQLAlchemy) |
| `customer_behavior_sql_queries.sql` | 10 business-question SQL queries against the cleaned data |
| `customer_behavior_dashboard.pbix` | Power BI dashboard |
| `Customer Shopping Behavior Analysis.pdf` | Written project report |
| `Customer-Shopping-Behavior-Analysis.pptx` | Stakeholder presentation |
| `Business Problem  Document.pdf` | Original business problem brief and deliverables |
| `customer_shopping_behavior.csv` | Raw dataset |

## Pipeline

1. **Python (cleaning + feature engineering)**
   - Imputed 37 missing `Review Rating` values with the median rating for that product's category (not the global median), to avoid skewing ratings across categories with different baseline satisfaction.
   - Standardized all columns to snake_case (`purchase_amount_(usd)` → `purchase_amount`, etc.).
   - Engineered `age_group` (quartile-based bins: Young Adult / Adult / Middle-aged / Senior) and `purchase_frequency_days` (mapping cadence labels like "Fortnightly" to a numeric day count) to make the categorical fields usable in aggregate SQL queries.
   - Verified `discount_applied` and `promo_code_used` were identical for every row and dropped the duplicate column.
   - Loaded the cleaned frame into PostgreSQL (with example connection code for MySQL and SQL Server as alternatives) via SQLAlchemy.

2. **SQL (business questions)** — answered directly in `customer_behavior_sql_queries.sql`:
   - Revenue split by gender
   - Customers who used a discount but still spent above the average purchase amount
   - Top 5 products by average review rating
   - Standard vs. Express shipping: average spend comparison
   - Subscribers vs. non-subscribers: spend and revenue comparison
   - Top 5 products most dependent on discounts
   - Customer segmentation into New / Returning / Loyal by purchase history
   - Top 3 products per category by order volume
   - Whether repeat buyers (>5 previous purchases) are more likely to subscribe
   - Revenue contribution by age group

3. **Power BI** — `customer_behavior_dashboard.pbix` visualizes the above for a non-technical audience.

## Key recommendations (from the written report)

- **Boost subscriptions** by promoting exclusive subscriber benefits.
- **Build a loyalty program** to move repeat buyers into the "Loyal" segment.
- **Revisit discount policy** — some products sell almost entirely on discount, which caps margin.
- **Lead marketing with top-rated, best-selling products** rather than spreading spend evenly across the catalog.
- **Target high-revenue age groups and Express-shipping users** specifically, rather than broad demographic campaigns.

## Running it yourself

```bash
pip install pandas sqlalchemy psycopg2-binary
```

Open `Customer_Shopping_Behavior_Analysis.ipynb` and run top to bottom. The notebook loads `customer_shopping_behavior.csv`, cleans it, and (optionally) pushes the result into a local Postgres/MySQL/SQL Server instance so you can run `customer_behavior_sql_queries.sql` against it.

**Before you push this further:** the notebook currently has DB credentials (user/password/host) hardcoded inline for the PostgreSQL/MySQL/SQL Server connection cells. That's fine for a local one-off run, but swap it for environment variables (e.g. `os.environ["DB_PASSWORD"]`) or a `.env` file (git-ignored) before this goes anywhere more permanent — right now anyone with the notebook has the credentials that were used when it was written.

## Possible next steps

- Add a `requirements.txt` / `environment.yml` so the notebook runs without guessing which libraries are needed.
- Parameterize the DB connection cells (see caveat above) instead of hardcoding credentials per-engine.
- Export the 10 SQL query results as CSVs or a small results section in this README, so the findings are visible without opening Postgres or Power BI.
- Add a short "how the dashboard maps to the SQL questions" section, since right now the `.pbix` and the `.sql` file aren't explicitly cross-referenced.
