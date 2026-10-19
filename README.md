# A/B Test Analysis for an AI Cart Recommendation Widget

## Business scenario

An e-commerce platform replaces the generic top-selling-items list on its cart page with an AI-driven personalized product recommendation widget.

**Business question:** Does the Challenger increase revenue per eligible cart, improve cart conversion, and support higher average order value (AOV) without harming checkout performance or customer experience?

| Experiment arm | Experience |
|---|---|
| Control (A) | Generic top-selling-items list |
| Challenger (B) | AI-driven personalized product recommendations |

## Project objective

Design and analyze a reproducible A/B test that evaluates the feature from experiment design through a product recommendation. The **primary decision metric** is revenue per randomized eligible cart, which includes zero revenue for carts with no purchase. Cart conversion rate and AOV among completed orders are important component metrics and must be reported with their denominators and uncertainty.

AOV is conditional on a purchase: the widget may change which shoppers convert, so AOV among purchasers should not be interpreted alone as the causal effect of the feature. Read it alongside conversion and revenue per randomized cart.

## Tools and deliverables

- **Tools:** Python, pandas, NumPy, SciPy, statsmodels, matplotlib/seaborn, and Power BI or Tableau.
- **Deliverables:** pre-analysis plan and power analysis, documented synthetic experiment data, data-quality checks, statistical analysis, a native Power BI (`.pbix`) or Tableau (`.twb`/`.twbx`) dashboard based on the supplied reference, and a product-manager decision memo. Python prepares and validates dashboard data; the final interactive dashboard is authored in the selected BI tool.
- **Estimated effort:** 18–22 hours.

This is a portfolio simulation, not evidence about a real product. Establish assumptions, sample size, significance level, and a fixed stopping rule before generating or reviewing simulated outcomes. Do not use a t-test for binary conversion; choose statistical methods suited to each metric and its randomization unit.

