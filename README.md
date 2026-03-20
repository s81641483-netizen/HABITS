<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Habit Tracker - Premium</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
body{margin:0;font-family:Arial;background:#0b0f1a;color:#fff}
.container{max-width:1500px;margin:auto;padding:20px}

h2{margin-bottom:5px}
.user{opacity:.7;margin-bottom:20px}

.dashboard{display:grid;grid-template-columns:250px 1fr 300px;gap:20px}

.sidebar{background:#111827;padding:15px;border-radius:10px}
.sidebar input{width:100%;padding:8px;margin-bottom:10px;border-radius:6px;border:none}
.sidebar button{width:100%;padding:8px;border:none;border-radius:6px;background:#22c55e;color:#000;font-weight:bold}

.card{background:#111827;padding:15px;border-radius:10px;margin-bottom:15px}

.grid{display:grid;grid-template-columns:1fr 1fr;gap:15px}

.analysis-bar{height:8px;background:#333;border-radius:5px;margin-top:5px}
.analysis-fill{height:8px;background:#22c55e;border-radius:5px}

table{width:100%;border-collapse:collapse;font-size:11px}
td,th{border:1px solid #333;padding:4px;text-align:center}

</style>
</head>
<body>

<div class="container">
<h2>Habit Tracker - January</h2>
<div class="user">User: Deniz Bahtiyaroğlu</div>

<div class="dashboard">

<div class="sidebar">
<input id="habitInput" placeholder="Yeni alışkanlık">
<button onclick="addHabit()">+ Add</button>
</div>

<div>

<div class="grid">
<div class="card">
<h4>Daily Progress</h4>
<canvas id="dailyChart"></canvas>
</div>

<div class="card">
<h4>Weekly Progress</h4>
<canvas id="weeklyChart"></canvas>
</div>
</div>

<div class="card">
<table id="table"></table>
</div>

<div class="grid">
<div class="card">
<h4>Goal</h4>
<div id="goal"></div>
</div>

<div class="card">
<h4>Overall</h4>
<canvas id="donutChart"></canvas>
</div>
</div>

</div>

<!-- 🔥 ANALYSIS PANEL -->
<div>
<div class="card">
<h4>Analysis</h4>
<div id="analysis"></div>
</div>

<div class="card">
<h4>Top Habits</h4>
<div id="topHabits"></div>
</div>
</div>

</div>
</div>

<script>

let habits = JSON.parse(localStorage.getItem("habits")) || [];
let logs = JSON.parse(localStorage.getItem("logs")) || {};

const days = 31;

let dailyChart, weeklyChart, donutChart;

function save(){
 localStorage.setItem("habits", JSON.stringify(habits));
 localStorage.setItem("logs", JSON.stringify(logs));
}

function addHabit(){
 const input = document.getElementById("habitInput");
 const val = input.value.trim();

 if(!val){ alert("Habit gir"); return; }
 if(habits.includes(val)){ alert("Zaten var"); return; }

 habits.push(val);
 input.value="";
 save();
 render();
}

function toggle(habit,day,val){
 logs[habit+"_"+day] = val;
 save();
 render();
}

function render(){
 const table = document.getElementById("table");
 table.innerHTML="";

 if(habits.length===0){
  table.innerHTML = "<tr><td style='padding:20px'>📌 Önce habit ekle</td></tr>";
  return;
 }

 let head = "<tr><th>Habit</th>";
 for(let i=1;i<=days;i++) head += `<th>${i}</th>`;
 head += "</tr>";
 table.innerHTML += head;

 let done = 0;
 let habitStats = {};

 habits.forEach(h=>{
  habitStats[h]=0;

  let row = `<tr><td>${h}</td>`;

  for(let d=0;d<days;d++){
    const key = h+"_"+d;
    const checked = logs[key] || false;

    if(checked){
      done++;
      habitStats[h]++;
    }

    row += `<td><input type="checkbox" ${checked?"checked":""} onchange="toggle('${h}',${d},this.checked)"></td>`;
  }

  row += "</tr>";
  table.innerHTML += row;
 });

 const total = habits.length * days;
 document.getElementById("goal").innerText = total;

 drawCharts(done,total);
 drawAnalysis(habitStats);
}

function drawAnalysis(stats){
 const container = document.getElementById("analysis");
 const top = document.getElementById("topHabits");

 container.innerHTML="";
 top.innerHTML="";

 const sorted = Object.entries(stats).sort((a,b)=>b[1]-a[1]);

 sorted.forEach(([habit,count])=>{
  const percent = Math.round((count/31)*100);

  container.innerHTML += `
    <div style="margin-bottom:10px">
      <div>${habit} (${percent}%)</div>
      <div class="analysis-bar">
        <div class="analysis-fill" style="width:${percent}%"></div>
      </div>
    </div>
  `;
 });

 sorted.slice(0,3).forEach(([habit])=>{
  top.innerHTML += `<div>🔥 ${habit}</div>`;
 });
}

function drawCharts(done,total){
 const daily = [];

 for(let d=0;d<days;d++){
  let count=0;
  habits.forEach(h=>{ if(logs[h+"_"+d]) count++; });
  daily.push(count);
 }

 if(dailyChart) dailyChart.destroy();
 dailyChart = new Chart(document.getElementById("dailyChart"),{
  type:"bar",
  data:{labels:daily.map((_,i)=>i+1),datasets:[{data:daily}]}
 });

 const weekly=[0,0,0,0,0];
 daily.forEach((v,i)=>weekly[Math.floor(i/7)]+=v);

 if(weeklyChart) weeklyChart.destroy();
 weeklyChart = new Chart(document.getElementById("weeklyChart"),{
  type:"bar",
  data:{labels:["W1","W2","W3","W4","W5"],datasets:[{data:weekly}]}
 });

 if(donutChart) donutChart.destroy();
 donutChart = new Chart(document.getElementById("donutChart"),{
  type:"doughnut",
  data:{labels:["Done","Left"],datasets:[{data:[done,total-done]}]}
 });
}

render();

</script>

</body>
</html>
