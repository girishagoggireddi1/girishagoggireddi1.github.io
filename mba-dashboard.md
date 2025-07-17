---
layout: page
title: Churn & Revenue Opportunity Dashboard
permalink: /mba-dashboard/
---

# Churn Risk & Revenue Opportunity Dashboard

This interactive dashboard helps visualize customer churn risk and revenue opportunities, using simulated data and the kind of analysis a business/data analyst would do with Python, SQL, and BI tools.

---

## Churn Risk by Segment

<div id="churn-pie"></div>

## Revenue by Customer Segment

<div id="revenue-bar"></div>

## Monthly Churn Trend

<div id="churn-line"></div>

## Top 5 Customers by Revenue

<table id="top-customers" style="width:100%;border-collapse:collapse;margin-top:1em;">
  <thead>
    <tr>
      <th style="border-bottom:1px solid #ccc;text-align:left;">Customer</th>
      <th style="border-bottom:1px solid #ccc;text-align:right;">Revenue ($)</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>Acme Corp</td><td style="text-align:right;">12,000</td></tr>
    <tr><td>Beta LLC</td><td style="text-align:right;">10,500</td></tr>
    <tr><td>Gamma Inc</td><td style="text-align:right;">9,800</td></tr>
    <tr><td>Delta Co</td><td style="text-align:right;">8,900</td></tr>
    <tr><td>Epsilon Ltd</td><td style="text-align:right;">8,200</td></tr>
  </tbody>
</table>

<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
<script>
// Churn Risk Pie Chart
Plotly.newPlot('churn-pie', [{
  values: [30, 20, 10, 40],
  labels: ['High Risk', 'Medium Risk', 'Low Risk', 'Loyal'],
  type: 'pie',
  marker: {colors: ['#ef4444', '#f59e42', '#3b82f6', '#22c55e']}
}], {
  title: 'Churn Risk by Segment'
});

// Revenue by Segment Bar Chart
Plotly.newPlot('revenue-bar', [{
  x: ['Enterprise', 'SMB', 'Startup', 'Individual'],
  y: [40000, 25000, 12000, 6000],
  type: 'bar',
  marker: {color: ['#6366f1', '#3b82f6', '#f59e42', '#22c55e']}
}], {
  title: 'Revenue by Customer Segment',
  yaxis: {title: 'Revenue ($)'}
});

// Monthly Churn Trend Line Chart
Plotly.newPlot('churn-line', [{
  x: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'],
  y: [5, 7, 6, 8, 4, 3],
  type: 'scatter',
  mode: 'lines+markers',
  line: {color: '#ef4444', width: 3}
}], {
  title: 'Monthly Churn Trend',
  yaxis: {title: 'Customers Lost'}
});
</script>

---

## How was this built?

- **Python**: Used for data cleaning, churn prediction, and revenue aggregation (see code below)
- **SQL**: Used for extracting and grouping customer data
- **Plotly.js**: Used for interactive dashboard visualizations

### Example Python Code

```python
import pandas as pd
# Simulate customer data
df = pd.DataFrame({
    'customer': ['Acme Corp', 'Beta LLC', 'Gamma Inc', 'Delta Co', 'Epsilon Ltd'],
    'revenue': [12000, 10500, 9800, 8900, 8200],
    'churn_risk': ['High', 'Medium', 'Low', 'Loyal', 'High']
})
# Group by churn risk
risk_summary = df.groupby('churn_risk').size()
print(risk_summary)
```

### Example SQL Query

```sql
SELECT churn_risk, COUNT(*) as num_customers
FROM customers
GROUP BY churn_risk;

SELECT segment, SUM(revenue) as total_revenue
FROM customers
GROUP BY segment;
```

---

**You can edit the data in the JavaScript arrays or table above to match your own project!**

---

## Why this dashboard is valuable

- **Business Impact:** Quickly spot at-risk customers and high-value segments
- **Actionable:** Helps prioritize retention and upsell strategies
- **Showcases:** Your ability to combine data analysis, business insight, and visualization

---

*Built with Plotly.js for easy, interactive analytics on a static portfolio!* 