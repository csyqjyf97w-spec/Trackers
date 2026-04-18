# Trackers
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover" />
  <title>Tracker 60 Meses</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    :root{
      --bg:#0b0f1a;
      --card:#121826;
      --accent:#22c55e;
      --text:#e5e7eb;
      --muted:#94a3b8;
    }
    body {margin:0;font-family:-apple-system;background:var(--bg);color:var(--text);}    
    .container{padding:20px;max-width:420px;margin:auto;}
    h1{text-align:center;margin-bottom:15px;}
    .card{background:var(--card);border-radius:20px;padding:15px;margin-bottom:15px;}
    input{width:100%;padding:12px;border-radius:12px;border:none;background:#0f172a;color:white;margin-top:10px;}
    .stats{display:flex;justify-content:space-between;margin-top:10px;color:var(--muted);}    
    .bar{height:10px;background:#1e293b;border-radius:10px;overflow:hidden;margin:10px 0;}
    .fill{height:100%;background:linear-gradient(90deg,#22c55e,#4ade80);width:0%;}
    .counter{text-align:center;color:var(--muted);}    
    .grid{display:grid;grid-template-columns:repeat(3,1fr);gap:10px;margin-top:10px;}
    .month{padding:12px;background:#1e293b;border-radius:12px;text-align:center;}
    .done{background:var(--accent);color:#000;text-decoration:line-through;}
    canvas{margin-top:15px;}
  </style>
</head>
<body>
<div class="container">
  <h1>Tracker 60 Meses</h1>

  <div class="card">
    <input type="number" id="amount" placeholder="Monto mensual">
    <div class="stats">
      <div>Pagado: $<span id="paid">0</span></div>
      <div>Total: $<span id="total">0</span></div>
    </div>
  </div>

  <div class="bar"><div class="fill" id="fill"></div></div>
  <div class="counter" id="count">0 / 60</div>

  <canvas id="chart"></canvas>

  <div class="grid" id="grid"></div>
</div>

<script>
let data = JSON.parse(localStorage.getItem("tracker")) || {
  months: Array(60).fill(false),
  amount: 0
};

const grid = document.getElementById("grid");
const count = document.getElementById("count");
const fill = document.getElementById("fill");
const paid = document.getElementById("paid");
const total = document.getElementById("total");
const input = document.getElementById("amount");

input.value = data.amount;

input.oninput = () => {
  data.amount = parseFloat(input.value) || 0;
  save();
  render();
};

function save(){
  localStorage.setItem("tracker", JSON.stringify(data));
}

let chart;

function render(){
  grid.innerHTML="";
  let done=0;

  data.months.forEach((m,i)=>{
    const div=document.createElement("div");
    div.className="month "+(m?"done":"");
    div.innerText="Mes "+(i+1);

    if(m) done++;

    div.onclick=()=>{
      data.months[i]=!data.months[i];
      save();
      render();
    };

    grid.appendChild(div);
  });

  count.innerText=done+" / 60";
  fill.style.width=(done/60)*100+"%";

  const paidAmount = done * data.amount;
  const totalAmount = 60 * data.amount;

  paid.innerText = paidAmount.toFixed(0);
  total.innerText = totalAmount.toFixed(0);

  updateChart(done, paidAmount, totalAmount);
}

function updateChart(done, paidAmount, totalAmount){
  const ctx = document.getElementById('chart');

  if(chart) chart.destroy();

  chart = new Chart(ctx, {
    type: 'doughnut',
    data: {
      labels: ['Pagado', 'Restante'],
      datasets: [{
        data: [paidAmount, totalAmount - paidAmount],
        backgroundColor: ['#22c55e', '#1e293b'],
        borderWidth:0
      }]
    },
    options: {
      plugins: {
        legend: {display:false}
      },
      cutout: '70%'
    }
  });
}

render();
</script>
</body>
</html>
