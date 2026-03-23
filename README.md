Source-to-Pay (S2P) Procurement Analytics & Risk Suite

Project Overview

This project transforms fragmented Excel procurement data into a centralized SQL-driven analytical engine and a Three-Tier Tableau Dashboard. The goal was to provide the Chief Procurement Officer (CPO) with visibility into $XM in spend, identifying $Y in "Contract Leakage," and tracking Supplier Diversity and Risk KPIs.

🛠️ Technical Stack
Data Source: Excel (Raw Purchase Orders, Invoices, and Supplier Master Data)

Database: MySQL (ETL, Table Joining, and Performance Aggregations)

Visualization: Tableau Desktop (Interactive Leaderboards, Pareto Analysis, and Trend Tracking)

SQL Analysis

I developed a robust Performance Summary table using Common Table Expressions (CTEs) to join three separate data silos. This allowed for seamless cross-functional analysis between Purchase Orders, Supplier Risk Profiles, and Invoice Payment cycles.

SQL

-- Building the Centralized Performance Summary Table
``` 
CREATE TABLE `2_s2p`.performance_summary AS
WITH purchase_supplier AS (
    SELECT p.po_id, p.supplier_id, p.department, p.po_date, p.amount, p.approval_days, p.`status`, 
           s.supplier_name, s.category, s.diversity_flag, s.risk_rating, s.country, 
           s.contract_start, s.contract_end
    FROM purchase_orders AS p
    LEFT JOIN suppliers AS s
        ON p.supplier_id = s.supplier_id
),
s2p_table AS (
    SELECT ps.po_id, ps.supplier_id, ps.department, ps.po_date, ps.amount, ps.approval_days, ps.`status`, 
           ps.supplier_name, ps.category, ps.diversity_flag, ps.risk_rating, ps.country, 
           ps.contract_start, ps.contract_end, i.invoice_id, i.invoice_date, i.payment_date,
           i.invoice_amount, i.payment_status
    FROM purchase_supplier AS ps
    LEFT JOIN invoices AS i
        ON ps.po_id = i.po_id
)
SELECT *
FROM s2p.s2p_performance_summary_1;

-- Finding the top 10 suppliers that spend the most with our company.
SELECT RANK() OVER(ORDER BY sum(amount) DESC) AS rn, supplier_name, ROUND(SUM(amount), 2) AS total_amount_spent
FROM performance_summary
GROUP BY 2
LIMIT 10;

-- Looking at our diversity flags and checking the percentage of diverse suppliers our company is using. 
SELECT diversity_flag, ROUND(SUM(amount)/(SELECT SUM(amount) FROM performance_summary) * 100, 2) AS percentage_spent
FROM performance_summary
GROUP BY 1
ORDER BY 1 DESC;

-- Viewing the percentage of high_risk suppliers used at the company. 
SELECT risk_rating, ROUND(sum(amount)/(SELECT SUM(amount) FROM performance_summary) * 100, 2) AS percentage_spent
FROM performance_summary
GROUP BY 1
ORDER BY 2 DESC;

-- Viewing high-risk spending at the company 
SELECT risk_rating, ROUND(sum(amount), 2) AS total_spent
FROM performance_summary
GROUP BY 1
ORDER BY 2;

-- Who are the top high-risk suppliers by spend?
SELECT supplier_name, ROUND(SUM(amount), 2) AS total_spent
FROM performance_summary
WHERE risk_rating = 'high'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;

-- Are high-risk vendors paid slower than others?
SELECT risk_rating, ROUND(AVG(DATEDIFF(payment_date, invoice_date)),1) AS days_until_paid
FROM performance_summary
WHERE payment_date != '1900-01-01 00:00:00' 
	AND payment_date IS NOT NULL AND invoice_date IS NOT NULL
GROUP BY 1
ORDER BY 2;

-- What is the average PO approval time by department?
SELECT department, ROUND(AVG(approval_days),2) AS avg_approval_time_days
FROM performance_summary
GROUP BY 1
ORDER BY 2;

-- What is the average payment cycle time (invoice to payment)?
SELECT ROUND(AVG(DATEDIFF(payment_date, invoice_date)), 1) AS average_payment_cycle
FROM performance_summary
WHERE payment_date != '1900-01-01 00:00:00' AND invoice_date != '1900-01-01 00:00:00' 
	AND payment_date IS NOT NULL AND invoice_date IS NOT NULL;

-- Which departments generate the most purchase orders?
SELECT department, COUNT(po_id) AS purchase_order_amount
FROM performance_summary
GROUP BY 1
ORDER BY 2 DESC;

-- Which departments have the highest procurement spend?
SELECT department, ROUND(SUM(amount), 2) AS total_spend
FROM performance_summary
GROUP BY 1
ORDER BY 2 DESC;

-- What is the average PO approval time by department?
SELECT department, ROUND(AVG(approval_days), 1) AS avg_approval_days
FROM performance_summary
GROUP BY 1;

-- Which suppliers get paid the fastest?
SELECT supplier_name, ROUND(AVG(DATEDIFF(payment_date, invoice_date)), 1) AS avg_days_to_payment
FROM performance_summary
WHERE payment_date NOT LIKE '1900%' AND payment_date IS NOT NULL and invoice_date IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;

-- Rank suppliers by total spend.
SELECT RANK() OVER(ORDER BY SUM(amount) DESC) AS rn, 
	supplier_name, ROUND(SUM(amount), 2) AS total_spend
FROM performance_summary
GROUP BY 2;

-- Top supplier per category 
SELECT category, supplier_name, ROUND(SUM(amount), 2) AS total_spend, 
	RANK() OVER(PARTITION BY category ORDER BY sum(amount)) AS rn 
FROM performance_summary
GROUP BY 1, 2;

-- Spend Share per Supplier
SELECT supplier_name, ROUND(SUM(amount), 2) AS total_amount, ROUND(SUM(amount) * 100/SUM(SUM(amount))  OVER(), 2) AS percentage_of_spend
FROM performance_summary
GROUP BY 1
ORDER BY 2 DESC;
```
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


🚀 Key Results
Identified Leakage: Highlighted departments with the highest spend on expired agreements.

Process Improvement: Pinpointed specific departments responsible for the longest approval delays.

Risk Visibility: Mapped 100% of "High Risk" spend to specific suppliers and categories for immediate audit.



