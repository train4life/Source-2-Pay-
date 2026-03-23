Source-to-Pay (S2P) Procurement Analytics & Risk Suite
📋 Project Overview
This project transforms fragmented Excel procurement data into a centralized SQL-driven analytical engine and a Three-Tier Tableau Dashboard. The goal was to provide the Chief Procurement Officer (CPO) with visibility into $XM in spend, identifying $Y in "Contract Leakage," and tracking Supplier Diversity and Risk KPIs.

🛠️ Technical Stack
Data Source: Excel (Raw Purchase Orders, Invoices, and Supplier Master Data)

Database: MySQL (ETL, Table Joining, and Performance Aggregations)

Visualization: Tableau Desktop (Interactive Leaderboards, Pareto Analysis, and Trend Tracking)

🗄️ SQL Engineering & Data Modeling
I developed a robust Performance Summary table using Common Table Expressions (CTEs) to join three separate data silos. This allowed for seamless cross-functional analysis between Purchase Orders, Supplier Risk Profiles, and Invoice Payment cycles.

SQL
-- Building the Centralized Performance Summary Table
CREATE TABLE `2_s2p`.performance_summary AS
WITH purchase_supplier AS (
    SELECT p.po_id, p.supplier_id, p.department, p.po_date, p.amount, p.approval_days, p.`status`, 
           s.supplier_name, s.category, s.diversity_flag, s.risk_rating, s.country, 
           s.contract_start, s.contract_end
    FROM purchase_orders AS p
    LEFT JOIN suppliers AS s ON p.supplier_id = s.supplier_id
),
s2p_table AS (
    SELECT ps.*, i.invoice_id, i.invoice_date, i.payment_date, i.invoice_amount, i.payment_status
    FROM purchase_supplier AS ps
    LEFT JOIN invoices AS i ON ps.po_id = i.po_id
)
SELECT * FROM s2p_table;
💡 Key SQL Insights Extracted:
Spend Concentration: Ranked suppliers by total spend to identify the "Vital Few" (Pareto 80/20 Rule).

Risk Mitigation: Quantified high-risk spend share and compared payment speeds for high-risk vs. low-risk vendors.

Efficiency Audit: Calculated average PO approval times and payment cycle lags (Invoice to Payment) by department.

SQL
-- Example: Identifying Top 10 High-Risk Suppliers by Spend
SELECT supplier_name, ROUND(SUM(amount), 2) AS total_spent
FROM performance_summary
WHERE risk_rating = 'high'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
📊 Tableau Dashboard Highlights
The final delivery consisted of three interactive dashboards designed for different organizational stakeholders:

1. Executive Spending Overview
Pareto Analysis: Visualized supplier concentration to identify consolidation opportunities.

Diversity Growth: Area charts showing Quarter-over-Quarter trends in ESG/Diversity spend.

Metric Toggle: A parameter-driven switch allowing users to flip the entire dashboard between PO Amount vs. Actual Invoice Paid.

2. Operational Efficiency & Risk
The Leakage Leaderboard: A ranked list of departments spending money on Expired Contracts.

Approval Bottlenecks: Bar charts identifying departments exceeding the 7-day SLA for PO approvals.

3. Financial Integrity Audit
Price Creep Scatter Plot: Identified invoices that significantly exceeded original PO amounts (Over-billing alerts).

URL Action: Integrated a "Deep Link" feature where clicking a Supplier ID opens the corresponding digital contract folder.

🚀 Key Results
Identified Leakage: Highlighted departments with the highest spend on expired agreements.

Process Improvement: Pinpointed specific departments responsible for the longest approval delays.

Risk Visibility: Mapped 100% of "High Risk" spend to specific suppliers and categories for immediate audit.



