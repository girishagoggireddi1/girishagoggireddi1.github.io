---
layout: page
title: SQL-Powered Analytics Dashboard
permalink: /sql-dashboard/
---

# SQL-Powered Analytics Dashboard

This page lets you run SQL queries on a sample business database and instantly visualize the results as interactive charts. Perfect for demonstrating real data analyst/business analyst workflows!

---

## 1. Write and Run a SQL Query

<textarea id="sql-input" rows="4" style="width:100%;font-family:monospace;">SELECT segment, SUM(revenue) as total_revenue FROM customers GROUP BY segment;</textarea>
<button onclick="runSQL()" style="margin-top:0.5em;">Run Query</button>

## 2. Query Results
<div id="sql-results" style="margin-top:1em;"></div>

## 3. Visualize as Chart
<select id="chart-type" style="margin-bottom:0.5em;">
  <option value="bar">Bar</option>
  <option value="pie">Pie</option>
  <option value="line">Line</option>
</select>
<button onclick="visualizeChart()">Visualize</button>
<div id="chart" style="margin-top:1em;"></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/sql.js/1.6.2/sql-wasm.js"></script>
<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
<script>
let db;
let lastResult = null;

initSqlJs({ locateFile: file => `https://cdnjs.cloudflare.com/ajax/libs/sql.js/1.6.2/${file}` }).then(SQL => {
  db = new SQL.Database();
  db.run(`CREATE TABLE customers (id INTEGER, name TEXT, segment TEXT, revenue INTEGER, churn_risk TEXT, month TEXT, profit INTEGER);
    INSERT INTO customers VALUES (1, 'Acme Corp', 'Enterprise', 12000, 'High', 'Jan', 3000);
    INSERT INTO customers VALUES (2, 'Beta LLC', 'SMB', 10500, 'Medium', 'Jan', 2500);
    INSERT INTO customers VALUES (3, 'Gamma Inc', 'Startup', 9800, 'Low', 'Feb', 2200);
    INSERT INTO customers VALUES (4, 'Delta Co', 'SMB', 8900, 'Medium', 'Feb', 2100);
    INSERT INTO customers VALUES (5, 'Epsilon Ltd', 'Enterprise', 8200, 'High', 'Mar', 2000);
    INSERT INTO customers VALUES (6, 'Zeta Group', 'Individual', 6000, 'Loyal', 'Mar', 1500);
    INSERT INTO customers VALUES (7, 'Eta LLC', 'Startup', 7000, 'Low', 'Apr', 1800);
    INSERT INTO customers VALUES (8, 'Theta Inc', 'SMB', 7500, 'Medium', 'Apr', 1900);
    INSERT INTO customers VALUES (9, 'Iota Co', 'Enterprise', 11000, 'Loyal', 'May', 3200);
    INSERT INTO customers VALUES (10, 'Kappa Ltd', 'Individual', 5000, 'Loyal', 'Jun', 1200);
  `);
  runSQL();
});

function runSQL() {
  const sql = document.getElementById('sql-input').value;
  let html = '';
  try {
    const res = db.exec(sql);
    lastResult = res;
    if (res.length > 0) {
      html += '<table border=1 style="border-collapse:collapse;width:100%"><tr>';
      res[0].columns.forEach(col => html += `<th style=\"background:#f1f5f9;padding:4px;\">${col}</th>`);
      html += '</tr>';
      res[0].values.forEach(row => {
        html += '<tr>';
        row.forEach(val => html += `<td style=\"padding:4px;\">${val}</td>`);
        html += '</tr>';
      });
      html += '</table>';
    } else {
      html = 'Query executed. No results to display.';
    }
  } catch (e) {
    html = `<span style="color:red;">Error: ${e.message}</span>`;
    lastResult = null;
  }
  document.getElementById('sql-results').innerHTML = html;
  document.getElementById('chart').innerHTML = '';
}

function visualizeChart() {
  if (!lastResult || lastResult.length === 0) {
    document.getElementById('chart').innerHTML = '<span style="color:red;">No data to visualize. Run a query first.</span>';
    return;
  }
  const chartType = document.getElementById('chart-type').value;
  const columns = lastResult[0].columns;
  const values = lastResult[0].values;
  if (columns.length < 2) {
    document.getElementById('chart').innerHTML = '<span style="color:red;">Need at least two columns to plot a chart.</span>';
    return;
  }
  const x = values.map(row => row[0]);
  const y = values.map(row => row[1]);
  let data;
  if (chartType === 'bar') {
    data = [{ x, y, type: 'bar', marker: {color: '#6366f1'} }];
  } else if (chartType === 'pie') {
    data = [{ labels: x, values: y, type: 'pie' }];
  } else if (chartType === 'line') {
    data = [{ x, y, type: 'scatter', mode: 'lines+markers', line: {color: '#3b82f6', width: 3} }];
  }
  Plotly.newPlot('chart', data, { title: `${chartType.charAt(0).toUpperCase() + chartType.slice(1)} Chart of ${columns[0]} vs ${columns[1]}` });
}
</script>

---

## Example SQL Queries

- Revenue by segment:
  ```sql
  SELECT segment, SUM(revenue) as total_revenue FROM customers GROUP BY segment;
  ```
- Churn risk count:
  ```sql
  SELECT churn_risk, COUNT(*) as num_customers FROM customers GROUP BY churn_risk;
  ```
- Monthly profit trend:
  ```sql
  SELECT month, SUM(profit) as total_profit FROM customers GROUP BY month;
  ```
- Top 5 customers by revenue:
  ```sql
  SELECT name, revenue FROM customers ORDER BY revenue DESC LIMIT 5;
  ```

---

*This dashboard uses sql.js (SQLite in the browser) and Plotly.js for true interactive analytics!* 