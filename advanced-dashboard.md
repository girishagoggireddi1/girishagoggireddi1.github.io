---
layout: page
title: Advanced Analytics Dashboard
permalink: /advanced-dashboard/
---

# Advanced Business Analytics Dashboard

This interactive dashboard combines Power BI-style visuals with SQL querying. Write a SQL query on the sample business data below—KPIs and charts will update based on your query!

---

<style>
.kpi-cards {
  display: flex;
  flex-wrap: wrap;
  gap: 2em;
  margin-bottom: 2em;
}
.kpi-card {
  background: linear-gradient(135deg, #6366f1 60%, #3b82f6 100%);
  color: #fff;
  border-radius: 16px;
  padding: 1.5em 2em;
  min-width: 180px;
  box-shadow: 0 4px 16px rgba(99,102,241,0.08);
  flex: 1 1 180px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: center;
}
.kpi-label {
  font-size: 1.1em;
  opacity: 0.85;
  margin-bottom: 0.3em;
}
.kpi-value {
  font-size: 2.2em;
  font-weight: bold;
  letter-spacing: -1px;
}
.dashboard-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 2em;
  justify-content: space-between;
}
.dashboard-cell {
  flex: 1 1 350px;
  min-width: 320px;
}
.narrative {
  background: #f1f5f9;
  border-radius: 12px;
  padding: 1.2em 1.5em;
  margin-top: 2em;
  font-size: 1.1em;
  color: #334155;
  box-shadow: 0 2px 8px rgba(99,102,241,0.06);
}
.sql-box {
  margin-bottom: 1.5em;
}
.sql-box textarea {
  width: 100%;
  font-family: monospace;
  font-size: 1em;
  padding: 0.5em;
  border-radius: 8px;
  border: 1px solid #d1d5db;
  margin-bottom: 0.5em;
}
.sql-box button {
  background: #6366f1;
  color: #fff;
  border: none;
  border-radius: 6px;
  padding: 0.5em 1.2em;
  font-size: 1em;
  cursor: pointer;
  margin-right: 1em;
}
.sql-box button:hover {
  background: #3b82f6;
}
</style>

<div class="sql-box">
  <label for="sql-input"><b>SQL Query:</b></label><br>
  <textarea id="sql-input" rows="3">SELECT * FROM customers;</textarea><br>
  <button onclick="runSQL()">Run Query</button>
  <span id="sql-error" style="color:red;"></span>
</div>

<div id="sql-results"></div>

<div class="kpi-cards" id="kpi-cards"></div>

<div class="dashboard-grid">
  <div class="dashboard-cell" id="revenue-bar"></div>
  <div class="dashboard-cell" id="churn-pie"></div>
</div>
<div class="dashboard-grid" style="margin-top:2em;">
  <div class="dashboard-cell" id="profit-line"></div>
  <div class="dashboard-cell" id="top-customers"></div>
</div>

<div class="narrative" id="narrative"></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/sql.js/1.6.2/sql-wasm.js"></script>
<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
<script>
let db, lastResult = null;
const baseData = [
  {id:1, name:'Acme Corp', segment:'Enterprise', revenue:12000, churn_risk:'High', month:'Jan', profit:3000},
  {id:2, name:'Beta LLC', segment:'SMB', revenue:10500, churn_risk:'Medium', month:'Jan', profit:2500},
  {id:3, name:'Gamma Inc', segment:'Startup', revenue:9800, churn_risk:'Low', month:'Feb', profit:2200},
  {id:4, name:'Delta Co', segment:'SMB', revenue:8900, churn_risk:'Medium', month:'Feb', profit:2100},
  {id:5, name:'Epsilon Ltd', segment:'Enterprise', revenue:8200, churn_risk:'High', month:'Mar', profit:2000},
  {id:6, name:'Zeta Group', segment:'Individual', revenue:6000, churn_risk:'Loyal', month:'Mar', profit:1500},
  {id:7, name:'Eta LLC', segment:'Startup', revenue:7000, churn_risk:'Low', month:'Apr', profit:1800},
  {id:8, name:'Theta Inc', segment:'SMB', revenue:7500, churn_risk:'Medium', month:'Apr', profit:1900},
  {id:9, name:'Iota Co', segment:'Enterprise', revenue:11000, churn_risk:'Loyal', month:'May', profit:3200},
  {id:10, name:'Kappa Ltd', segment:'Individual', revenue:5000, churn_risk:'Loyal', month:'Jun', profit:1200}
];

initSqlJs({ locateFile: file => `https://cdnjs.cloudflare.com/ajax/libs/sql.js/1.6.2/${file}` }).then(SQL => {
  db = new SQL.Database();
  db.run(`CREATE TABLE customers (id INTEGER, name TEXT, segment TEXT, revenue INTEGER, churn_risk TEXT, month TEXT, profit INTEGER);`);
  baseData.forEach(row => {
    db.run(`INSERT INTO customers VALUES (?, ?, ?, ?, ?, ?, ?);`, [row.id, row.name, row.segment, row.revenue, row.churn_risk, row.month, row.profit]);
  });
  runSQL();
});

function runSQL() {
  const sql = document.getElementById('sql-input').value;
  let html = '';
  let error = '';
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
    document.getElementById('sql-error').innerText = '';
  } catch (e) {
    html = '';
    error = e.message;
    lastResult = null;
  }
  document.getElementById('sql-results').innerHTML = html;
  document.getElementById('sql-error').innerText = error;
  updateDashboard();
}

function getDataFromResult() {
  // If lastResult is a SELECT * FROM customers or similar, convert to array of objects
  if (!lastResult || lastResult.length === 0) return [];
  const cols = lastResult[0].columns;
  const vals = lastResult[0].values;
  // Only use if all base columns are present
  const baseCols = ['id','name','segment','revenue','churn_risk','month','profit'];
  if (baseCols.every(c => cols.includes(c))) {
    return vals.map(row => Object.fromEntries(cols.map((c,i) => [c, row[i]])));
  }
  // Otherwise, return empty (charts/KPIs won't update)
  return [];
}

function updateKPI(filtered) {
  const totalRevenue = filtered.reduce((sum, d) => sum + d.revenue, 0);
  const totalProfit = filtered.reduce((sum, d) => sum + d.profit, 0);
  const customerCount = filtered.length;
  const churnMap = {High: 3, Medium: 2, Low: 1, Loyal: 0};
  const avgChurn = filtered.length ? (filtered.reduce((sum, d) => sum + (churnMap[d.churn_risk]||0), 0) / filtered.length) : 0;
  let churnLabel = 'Loyal';
  if (avgChurn > 2.5) churnLabel = 'High';
  else if (avgChurn > 1.5) churnLabel = 'Medium';
  else if (avgChurn > 0.5) churnLabel = 'Low';
  document.getElementById('kpi-cards').innerHTML = `
    <div class='kpi-card'><div class='kpi-label'>Total Revenue</div><div class='kpi-value'>$${totalRevenue.toLocaleString()}</div></div>
    <div class='kpi-card'><div class='kpi-label'>Total Profit</div><div class='kpi-value'>$${totalProfit.toLocaleString()}</div></div>
    <div class='kpi-card'><div class='kpi-label'>Customer Count</div><div class='kpi-value'>${customerCount}</div></div>
    <div class='kpi-card'><div class='kpi-label'>Avg. Churn Risk</div><div class='kpi-value'>${churnLabel}</div></div>
  `;
}

function updateNarrative(filtered) {
  if (filtered.length === 0) {
    document.getElementById('narrative').innerText = 'No data for the current SQL query.';
    return;
  }
  // Top segment by revenue
  const segs = [...new Set(filtered.map(d => d.segment))];
  const segRev = segs.map(seg => ({seg, rev: filtered.filter(d => d.segment === seg).reduce((s, d) => s + d.revenue, 0)}));
  segRev.sort((a,b) => b.rev - a.rev);
  const topSeg = segRev[0];
  // Month with highest profit
  const months = [...new Set(filtered.map(d => d.month))];
  const monthProf = months.map(m => ({m, p: filtered.filter(d => d.month === m).reduce((s, d) => s + d.profit, 0)}));
  monthProf.sort((a,b) => b.p - a.p);
  const topMonth = monthProf[0];
  // Top customer
  const topCust = filtered.slice().sort((a,b) => b.revenue - a.revenue)[0];
  document.getElementById('narrative').innerHTML = `
    <b>Insight:</b> <br>
    <ul>
      <li><b>${topSeg.seg}</b> is the top revenue segment ($${topSeg.rev.toLocaleString()})</li>
      <li><b>${topMonth.m}</b> had the highest profit ($${topMonth.p.toLocaleString()})</li>
      <li>Top customer: <b>${topCust.name}</b> ($${topCust.revenue.toLocaleString()})</li>
    </ul>
  `;
}

function updateDashboard() {
  const filtered = getDataFromResult();
  if (filtered.length === 0) {
    document.getElementById('kpi-cards').innerHTML = '';
    document.getElementById('revenue-bar').innerHTML = '';
    document.getElementById('churn-pie').innerHTML = '';
    document.getElementById('profit-line').innerHTML = '';
    document.getElementById('top-customers').innerHTML = '';
    document.getElementById('narrative').innerText = 'No data for the current SQL query.';
    return;
  }
  updateKPI(filtered);
  updateNarrative(filtered);

  // Revenue by Segment
  const segments = [...new Set(filtered.map(d => d.segment))];
  const revenueBySegment = segments.map(seg =>
    filtered.filter(d => d.segment === seg).reduce((sum, d) => sum + d.revenue, 0)
  );
  Plotly.newPlot('revenue-bar', [{
    x: segments,
    y: revenueBySegment,
    type: 'bar',
    marker: {color: '#6366f1'},
    hovertemplate: 'Segment: %{x}<br>Revenue: $%{y:,}<extra></extra>'
  }], {
    title: 'Revenue by Segment',
    yaxis: {title: 'Revenue ($)'},
    hovermode: 'closest'
  });

  // Churn Risk Pie
  const churns = [...new Set(filtered.map(d => d.churn_risk))];
  const churnCounts = churns.map(risk => filtered.filter(d => d.churn_risk === risk).length);
  Plotly.newPlot('churn-pie', [{
    labels: churns,
    values: churnCounts,
    type: 'pie',
    marker: {colors: ['#ef4444', '#f59e42', '#3b82f6', '#22c55e']},
    hovertemplate: '%{label}: %{value} customers<extra></extra>'
  }], {
    title: 'Churn Risk Distribution',
    legend: {orientation: 'h'}
  });

  // Monthly Profit Trend
  const months = ['Jan','Feb','Mar','Apr','May','Jun'];
  const profitByMonth = months.map(m =>
    filtered.filter(d => d.month === m).reduce((sum, d) => sum + d.profit, 0)
  );
  Plotly.newPlot('profit-line', [{
    x: months,
    y: profitByMonth,
    type: 'scatter',
    mode: 'lines+markers',
    line: {color: '#3b82f6', width: 3},
    hovertemplate: 'Month: %{x}<br>Profit: $%{y:,}<extra></extra>'
  }], {
    title: 'Monthly Profit Trend',
    yaxis: {title: 'Profit ($)'},
    hovermode: 'closest'
  });

  // Top Customers Table
  const top = filtered.slice().sort((a,b) => b.revenue - a.revenue).slice(0,5);
  let html = '<h4>Top 5 Customers by Revenue</h4>';
  html += '<table border=1 style="border-collapse:collapse;width:100%"><tr><th>Name</th><th>Segment</th><th>Revenue ($)</th><th>Profit ($)</th></tr>';
  top.forEach(row => {
    html += `<tr><td>${row.name}</td><td>${row.segment}</td><td>${row.revenue}</td><td>${row.profit}</td></tr>`;
  });
  html += '</table>';
  document.getElementById('top-customers').innerHTML = html;
}
</script>

---

**How this works:**
- Write a SQL query to filter or aggregate the data (e.g., `SELECT * FROM customers WHERE segment = 'Enterprise';`)
- If your query returns all the main columns, the dashboard will update; otherwise, only the table will show
- KPI cards, charts, and insights update based on your query result
- Data is in-browser for demo purposes, but you can connect to a real backend in a dynamic app
- Built with sql.js and Plotly.js for rich, interactive analytics

If you want to add more visuals, features, or custom SQL logic, just ask! 