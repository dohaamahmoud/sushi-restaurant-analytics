## 2. Data
Six tables, 2015–2024:

| Table | Description |
| orders | One row per order: date, channel, customer segment, party size, revenue, COGS, gross profit, discount, tip, delivery fee, rating |
| order_items | Items inside each order: item, category, unit price, COGS, gross profit |
| customers | Customer segment, join date, local/non-local, alcohol preference |
| daily_pnl | Daily covers, revenue, COGS, labour cost, fixed costs, operating profit |
| reviews | Customer ratings (1–5) by date and channel |
| staff | Hire/termination dates and base hourly rate |

## 3. Data Cleaning & Feature Engineering (Python)
**Quality checks (all 6 tables)**
- Missing values: the only significant gap was `rating` in orders (not every order
  gets a review). It was kept as null and a `has_review` flag was added, instead of
  filling it with made-up ratings.
- Duplicates: checked per table, and for order_items at the order + item level.
- Data types: converted all date columns to datetime.
- Outliers: detected with the IQR method. Capped copies were created for revenue,
  COGS, gross profit and tip, and the original columns were kept.
- Category consistency: checked unique values of channel, segment, day, month.

**Business-rule validation**
- gross_profit = revenue − cogs
- total_collected = revenue − discount + tip + delivery_fee
- operating_profit = gross_profit − labour_cost − fixed_ops_cost
- order_items totals reconcile with orders revenue and profit
- Delivery fees only on delivery orders; ratings between 1 and 5; party size > 0;
  termination date after hire date

**Feature engineering**
- Time/context: season, `covid_period` (Lockdown: Mar–May 2020, Recovery: Jun 2020–Dec 2021)
- Orders: revenue per person, discount %, tip %, party group, items per person, has_alcohol
- Customers: RFM scores and segments (Low / Mid / High / Champions);
  **churned = no order in the last 180 days**
- Daily P&L: labour %, operating margin %, revenue per cover, loss-day flag

## 7. (replace the last insight with this)
7. **Prediction model:** a high-value order is one with revenue above the median.
   Logistic Regression identifies them with 89.5% accuracy and 90.2% precision.

## 9. Decisions & Trade-offs
- **Target definition:** high-value = revenue ≥ median revenue, calculated on the
  training set only, so the test set does not influence the threshold.
- **Data leakage:** removed revenue, total_collected, gross_profit, COGS, items count
  and average item price from the features, since they reveal the order value directly.
- **Model choice:** four models were compared (Logistic Regression, Decision Tree,
  Random Forest, SVM) and all scored within 0.3%, so Logistic Regression was chosen
  as the simplest and most interpretable.
- **Split:** 80/20 train/test (~17K test orders).
- **SVM** was trained on a 20K-row sample because of training time.
- **Why Python for cleaning:** ~84K orders and ~1M item rows, with reproducible
  validation checks and ML in the same workflow.
- **Why Power BI for the dashboard:** interactive filters and DAX KPIs for management.
