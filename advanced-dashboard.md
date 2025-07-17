---
layout: page
title: Advanced Analytics Dashboard
permalink: /advanced-dashboard/
---

# Advanced Business Analytics Dashboard

This interactive dashboard mimics a Power BI experience: filter by segment and month, and see all charts update instantly. Explore revenue, profit, churn, and top customers with rich visuals and interactivity—all in your browser!

---

## Dashboard Filters

Segment:
<select id="segment-filter">
  <option value="All">All</option>
  <option value="Enterprise">Enterprise</option>
  <option value="SMB">SMB</option>
  <option value="Startup">Startup</option>
  <option value="Individual">Individual</option>
</select>

Month:
<select id="month-filter">
  <option value="All">All</option>
  <option value="Jan">Jan</option>
  <option value="Feb">Feb</option>
  <option value="Mar">Mar</option>
  <option value="Apr">Apr</option>
  <option value="May">May</option>
  <option value="Jun">Jun</option>
</select>

<button onclick="updateDashboard()" style="margin-left:1em;">Apply Filters</button>

---

<div style="display:flex;flex-wrap:wrap;gap:2em;justify-content:space-between;">
  <div id="revenue-bar" style="flex:1 1 350px;min-width:320px;"></div>
  <div id="churn-pie" style="flex:1 1 350px;min-width:320px;"></div>
</div>
<div style="display:flex;flex-wrap:wrap;gap:2em;justify-content:space-between;margin-top:2em;">
  <div id="profit-line" style="flex:1 1 350px;min-width:320px;"></div>
  <div id="top-customers" style="flex:1 1 350px;min-width:320px;"></div>
</div>

<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
<script>
// Sample data
const data = [
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

function filterData() {
  const seg = document.getElementById('segment-filter').value;
  const mon = document.getElementById('month-filter').value;
  return data.filter(row =>
    (seg === 'All' || row.segment === seg) &&
    (mon === 'All' || row.month === mon)
  );
}

function updateDashboard() {
  const filtered = filterData();

  // Revenue by Segment
  const segments = [...new Set(filtered.map(d => d.segment))];
  const revenueBySegment = segments.map(seg =>
    filtered.filter(d => d.segment === seg).reduce((sum, d) => sum + d.revenue, 0)
  );
  Plotly.newPlot('revenue-bar', [{
    x: segments,
    y: revenueBySegment,
    type: 'bar',
    marker: {color: '#6366f1'}
  }], {
    title: 'Revenue by Segment',
    yaxis: {title: 'Revenue ($)'}
  });

  // Churn Risk Pie
  const churns = [...new Set(filtered.map(d => d.churn_risk))];
  const churnCounts = churns.map(risk => filtered.filter(d => d.churn_risk === risk).length);
  Plotly.newPlot('churn-pie', [{
    labels: churns,
    values: churnCounts,
    type: 'pie',
    marker: {colors: ['#ef4444', '#f59e42', '#3b82f6', '#22c55e']}
  }], {
    title: 'Churn Risk Distribution'
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
    line: {color: '#3b82f6', width: 3}
  }], {
    title: 'Monthly Profit Trend',
    yaxis: {title: 'Profit ($)'}
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

// Initial render
updateDashboard();
</script>

---

**How this works:**
- All charts and tables update instantly when you change filters—just like in Power BI!
- Data is in-browser for demo purposes, but you can connect to a real backend in a dynamic app.
- Built with Plotly.js for rich, interactive analytics.

If you want to add more visuals, filters, or features, just ask! 