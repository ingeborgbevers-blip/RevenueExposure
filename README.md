# Ecommerce Revenue Exposure & Payment Reconciliation Control

![Power BI Dashboard](Finance%20investigation%20view.png)

## Overview

This project explores revenue exposure, payment reconciliation, and order-status control within an ecommerce environment.

Using a public ecommerce order and supply chain dataset from Kaggle, the project was built as a SQL-based reporting workflow using PostgreSQL and Power BI. The original analysis started as a standard ecommerce dashboard exercise, looking at order status, customer location, shipping charges, product activity, and sales movement.

However, once payment data was brought into the analysis, the direction changed.

Every order had a payment record, but recorded payment value did not reconcile cleanly to recorded order value. The project therefore moved from a general sales and fulfilment dashboard into a revenue exposure and payment control case study.

The project demonstrates a realistic business intelligence workflow:

CSV files → PostgreSQL staging schema → SQL reporting tables/views → Power BI dashboards → executive case study

The analysis was designed as a practical reporting and decision-support case study suitable for:

* SME ecommerce environments,
* finance and revenue-quality analysis,
* order-status and payment-control reporting,
* ERP / SAP-style operational reporting demonstrations,
* PostgreSQL and Power BI portfolio work,
* and finance/operations improvement discussions.

---

# The Problem

Headline revenue can give a misleading view of business performance when payment values do not reconcile to order values.

In this dataset, the management dashboard identified:

* £34.40M recorded order value
* £24.00M recorded payment value
* -£10.41M net payment position
* 69.75% payment coverage
* £21.33M underpaid exposure
* £10.92M overpaid exposure
* £20.98M delivered orders underpaid
* £495.94K paid value not delivered

The first assumption was that this might be a standard sales, delivery, and customer-order analysis.

The data initially supported questions such as:

* which orders were delivered, shipped, cancelled, unavailable, approved, invoiced or processing?
* which customer states carried the most activity?
* whether shipping charges followed any obvious pattern;
* and which product or order groups appeared most commercially significant.

But the payment data changed the business question.

A check for multiple payment records per order returned no duplicate order IDs in the payment table. This meant payment value could be treated as the recorded payment total for each order, rather than a value that needed to be aggregated across several payment rows.

The issue was therefore not missing payment aggregation.

The issue was that recorded order value and recorded payment value did not match cleanly at order level.

This reflects a serious reporting and control challenge:

> Growing recorded revenue is not enough if the business cannot clearly reconcile order value, payment value, and order status.

A business can report millions in order value and still carry material financial risk if the grip on overpayments, underpayments, settlement, refunds, receivables, or payment-control processes is too loose.

---

# The Approach

The project used a multi-table ecommerce dataset containing:

* customers,
* orders,
* order items,
* products,
* payments,
* and supporting order-status/payment fields.

The source CSV files were imported into PostgreSQL using a structured staging schema. Clean reporting tables and views were then created to prepare Power BI-ready outputs.

The analysis developed in stages.

## Stage 1: Sales, Delivery and Customer View

The first stage looked at the dataset from a standard commercial and operational perspective:

* order status,
* customer state,
* shipping charges,
* product category,
* order value,
* payment type,
* and monthly order activity.

This stage showed useful activity, but it did not yet explain whether the business could trust the money trail.

## Stage 2: Payment and Revenue Exposure

The second stage compared recorded order value against recorded payment value.

Order value was calculated from:

* item value,
* plus shipping charges.

Payment value was taken from the payment table at order level.

This revealed a major reconciliation issue:

* only one order matched exactly;
* many orders were classified as underpaid;
* many orders were classified as overpaid;
* delivered orders carried the largest underpaid exposure;
* and paid value also existed against orders not marked as delivered.

The focus of the dashboard therefore shifted from “what happened operationally?” to “where is financial control not visible?”

---

# The Solution

The reporting environment combines PostgreSQL and Power BI to create a practical revenue-control BI workflow.

## PostgreSQL Layer

The PostgreSQL database was structured with:

* a `staging` schema for raw imported CSV tables,
* a `reporting` schema for cleaned reporting tables and views,
* source alphanumeric IDs preserved for traceability,
* surrogate keys created for cleaner reporting relationships,
* adjusted data types for timestamps, payment values, item values, and shipping charges,
* and reporting logic for payment reconciliation.

Key reporting objects included:

* `reporting.dim_customer`
* `reporting.dim_product`
* `reporting.fact_order`
* `reporting.fact_order_item`
* `reporting.vw_order_item_analysis`
* `reporting.vw_payment_reconciliation`

The product data required cleaning because the source product table contained repeated product rows. These were retained in staging for auditability and deduplicated in the reporting product dimension, resulting in one clean row per product ID.

Missing product categories were not invented. They were retained and labelled as `Unknown / Uncategorised` in the reporting layer where required.

The payment reconciliation view created order-level fields including:

* recorded order value,
* recorded payment value,
* payment difference,
* underpaid exposure,
* overpaid exposure,
* payment position,
* instalment status,
* delivered-underpaid flag,
* paid-value-not-delivered flag,
* and high-value mismatch flag.

This made the Power BI report easier to build and reduced the risk of mixing order-level and order-line-level calculations incorrectly.

## Power BI Layer

The Power BI report was intentionally split into two dashboard pages:

* Management / Executive Revenue Exposure
* Finance Control Detail

This avoided overloading one page with both board-level risk and operational investigation detail.

---

# Dashboard Highlights

## Management / Executive Revenue Exposure Dashboard

The management page focuses on the scale and direction of the risk.

It includes:

* recorded order value,
* recorded payment value,
* net payment position,
* payment coverage,
* underpaid exposure,
* overpaid exposure,
* delivered orders underpaid,
* paid value not delivered,
* payment reconciliation status,
* underpaid and overpaid trend over time,
* and order status by payment position.

The key finding was:

> Payment exists, but payment control is not yet visible.

The dashboard showed £34.40M recorded order value against £24.00M recorded payment value, leaving a net payment position of -£10.41M.

The strongest executive concern was the £20.98M delivered orders underpaid figure. These are orders marked as delivered where recorded payment value sits below recorded order value.

This does not prove unpaid debt, fraud, failed collection, or incorrect reporting on its own. The dataset does not include enough finance context to confirm that.

However, it does show that the available reporting data does not provide enough comfort that delivered order value and payment value reconcile cleanly.

That is exactly the type of issue a CFO would need surfaced before trusting reported revenue.

## Finance Control Detail Dashboard

The finance page supports investigation rather than executive summary.

It includes slicers for:

* order status,
* payment method,
* payment position,
* and payment structure / instalments.

It also includes order-level exception tables showing:

* order ID,
* order status,
* customer state,
* recorded order value,
* recorded payment value,
* payment difference,
* payment method,
* payment structure,
* and number of instalments.

This page allows finance users to isolate exception groups quickly and decide where to investigate first.

The distinction between the two pages is deliberate:

* Management needs the risk made visible.
* Finance needs the working detail behind it.

Good reporting does not mean showing every number to everyone. It means giving each audience the level of truth they can act on.

---

# Key Findings

The analysis identified several important findings.

## 1. Payment Records Were Present, But Not Sufficient

Every order had a payment record.

That could initially appear reassuring, but payment presence did not guarantee payment reconciliation.

The payment table contained one payment record per order, which means the reconciliation issue was not caused by missed aggregation across multiple payment rows.

## 2. Order Value and Payment Value Did Not Reconcile Cleanly

The report identified only one exactly matched order.

The remaining order base was split across underpaid and overpaid positions.

This means the available dataset cannot support a simple assumption that order value equals payment value.

## 3. Delivered Orders Carried the Main Underpaid Exposure

The largest concern was not simply open, cancelled, or unavailable orders.

A material underpaid exposure appeared against orders marked as delivered.

This creates a finance-control question:

> If the order is delivered, where is the evidence that the remaining value was collected, settled, refunded, financed, written off, or otherwise explained?

## 4. Overpaid Orders Also Required Separate Review

The analysis also identified substantial overpaid exposure.

Overpaid positions could reflect refunds, vouchers, payment-provider mechanics, credits, duplicated logic, timing differences, or missing supporting data.

They should not be treated as confirmed surplus cash without further finance evidence.

## 5. The Dataset Lacks the Finance Context Needed to Close the Question

The dataset does not include:

* invoice documents,
* tax records,
* settlement dates,
* refund records,
* chargebacks,
* credit notes,
* receivables ledger entries,
* external finance-provider information,
* or payment authorisation/capture distinction.

This limits the level of certainty available from the dashboard.

The dashboard can identify exposure and exception patterns. It cannot fully explain the cause without additional finance-system data.

---

# Operational Recommendations

The analysis led to five practical recommendations.

## 1. Introduce Payment Reconciliation Monitoring

Track recorded order value against recorded payment value at order level.

At minimum, reporting should separate:

* matched orders,
* underpaid orders,
* overpaid orders,
* and unresolved payment positions.

This should be reviewed by order status, payment method, payment structure, and month.

## 2. Prioritise Delivered Underpaid Orders

Delivered underpaid orders should be treated as a priority investigation group.

These orders have already moved through the operational process, so finance needs clear evidence of payment collection, settlement, credit terms, financing, refund treatment, or write-off logic.

## 3. Separate Executive Reporting From Finance Investigation

The management dashboard should show the scale of exposure clearly.

The finance dashboard should support filtering and exception review.

Trying to combine both into one page risks creating a dashboard that is too detailed for leadership and not detailed enough for investigation.

## 4. Add Missing Finance-System Data

To close the reconciliation question, the reporting model would need additional data sources such as:

* invoice records,
* refund records,
* chargebacks,
* settlement data,
* receivables data,
* credit notes,
* payment provider records,
* and tax/VAT documentation.

Without these sources, the dashboard can show exposure but cannot fully explain it.

## 5. Build an Exception-Based Review Process

High-value mismatches should be reviewed through a controlled workflow.

Useful exception categories include:

* delivered orders with underpaid exposure,
* cancelled or unavailable orders with payment value,
* high-value overpaid orders,
* instalment-based mismatches,
* and recurring mismatch patterns by payment method or customer state.

Exception reporting allows finance teams to focus on the orders most likely to affect cash, revenue quality, or control risk.

---

# Data Limitations

This case study is based on a public dataset and should not be treated as a complete finance-system audit.

The analysis is limited by the data available.

The dashboard does not prove:

* fraud,
* unpaid debt,
* failed collections,
* incorrect accounting,
* or business insolvency risk.

It does show that, from the available tables, order value and payment value cannot be assumed to reconcile.

The most useful conclusion is therefore:

> The available reporting data raises a material finance-control question that would require additional finance-system evidence before management could rely on the revenue position.

This distinction matters. The dashboard identifies the gap. The business would still need the underlying finance records to explain it.

---

# Demo

The repository includes:

* PostgreSQL table setup scripts
* SQL reporting table/view scripts
* Power BI dashboard screenshots
* executive case study PDF
* dashboard PDF export
* source dataset notes
* and operational recommendations

Suggested walkthrough order:

1. Review the database structure
2. Review the staging schema
3. Review the reporting schema
4. Open the SQL reporting views
5. Open the Power BI report
6. Review the Management / Executive Revenue Exposure dashboard
7. Review the Finance Control Detail dashboard
8. Read the executive case study summary

---

# How to Run

## Requirements

* PostgreSQL
* pgAdmin or psql
* Power BI Desktop
* CSV source files from the Kaggle dataset

## Steps

1. Download or clone the repository

```bash
git clone <repository-url>
```

2. Create a PostgreSQL database

Example:

```sql
CREATE DATABASE ecommerce_revenue_exposure;
```

3. Create schemas

```sql
CREATE SCHEMA staging;
CREATE SCHEMA reporting;
```

4. Create the staging tables

Run the SQL setup scripts included in the repository.

5. Import the CSV files into the staging schema

Import the source files into the relevant staging tables, including:

* customers
* orders
* order items
* products
* payments

6. Create the reporting tables and views

Run the SQL scripts for:

* dimensions
* fact tables
* reporting views
* payment reconciliation logic

7. Open the Power BI file

Connect Power BI to the PostgreSQL database and refresh the reporting views.

8. Review the dashboards and case study PDF

Use the dashboards and PDF summary to review the revenue exposure and payment reconciliation findings.

---

# Suggested Business Use

This case study demonstrates how a business could move from general ecommerce reporting to finance-control reporting.

It is relevant where managers need to understand:

* whether recorded sales can be trusted,
* whether payment values reconcile to order values,
* where delivered orders carry underpaid exposure,
* where overpaid orders need explanation,
* and whether finance teams have enough detail to investigate exceptions.

The project also demonstrates the value of not stopping at the first dashboard.

The first version showed activity.

The better version showed risk.

That is where reporting becomes decision support.

---

# Tools Used

* PostgreSQL
* SQL
* Power BI Desktop
* Power Query
* Kaggle source data
* GitHub
* PDF case study export

---

# Project Status

Completed as a portfolio case study.

Potential future improvements include:

* adding refund and settlement data if available,
* improving invoice/tax document visibility,
* modelling receivables and write-off outcomes,
* adding payment provider reconciliation,
* building a formal exception workflow,
* and creating a refreshed version using real business data where permitted.
