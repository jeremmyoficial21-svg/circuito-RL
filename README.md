# circuito-RL
<jeremmy-gabriel>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Circuito RL en Serie – EDO Primer Orden</title>
<meta name="viewport" content="width=device-width, initial-scale=1">

<style>
:root{
  --bg:#020617;
  --card:#0f172a;
  --accent:#38bdf8;
  --text:#e5e7eb;
  --muted:#94a3b8;
  --highlight:#22d3ee;
}

*{box-sizing:border-box}

body{
  margin:0;
  font-family:Arial, Helvetica, sans-serif;
  background:linear-gradient(180deg,#020617,#020617);
  color:var(--text);
}

header{
  text-align:center;
  padding:20px;
  background:#020617;
  border-bottom:2px solid var(--accent);
}
h1{margin:0;color:var(--accent)}

.container{
  max-width:1100px;
  margin:auto;
  padding:20px;
}

.card{
  background:var(--card);
  border-radius:14px;
  padding:20px;
  margin-bottom:20px;
  box-shadow:0 10px 25px rgba(0,0,0,0.35);
}

label{
  font-size:15px;
  color:var(--muted);
  display:block;
  margin-bottom:6px;
}

input,select{
  width:100%;
  padding:10px;
  background:#020617;
  border:1px solid #334155;
  border-radius:8px;
  color:white;
}

.form{
  display:grid;
  grid-template-columns:repeat(auto-fit,minmax(200px,1fr));
  gap:15px;
}

.row{
  display:grid;
  grid-template-columns:1fr 80px;
  gap:8px;
}

button{
  margin-top:15px;
  padding:12px;
  border:none;
  border-radius:10px;
  background:var(--accent);
  font-size:15px;
  font-weight:bold;
  cursor:pointer;
}

canvas{
  width:100%;
  height:300px;
  border-radius:12px;
  background:#020617;
  border:1px solid #334155;
}

/* 🔹 RESULTADOS MEJORADOS 🔹 */
.results{
  margin-top:20px;
  padding:18px;
  background:#020617;
  border-radius:12px;
  border-left:5px solid var(--accent);
  font-size:16px;
  line-height:1.6;
}

.results h2{
  margin-top:0;
  color:var(--highlight);
  font-size:20px;
}

.results strong{
  color:white;
}

.results ul{
  margin:10px 0 15px 20px;
}

table{
  width:100%;
  border-collapse:collapse;
  font-size:15px;
  margin-top:10px;
}

th,td{
  padding:10px;
  border-bottom:1px solid #334155;
}

th{
  color:var(--highlight);
  text-align:left;
}

td{
  color:white;
}

/* Memoria */
.memory{
  margin-top:15px;
  padding:15px;
  background:#020617;
  border-radius:10px;
  font-size:15px;
  line-height:1.6;
}

footer{
  text-align:center;
  color:var(--muted);
  font-size:13px;
}
</style>
</head>

<body>

<header>
  <h1>⚡ Circuito RL en Serie</h1>
  <p>Ecuación diferencial homogénea de primer orden</p>
</header>

<div class="container">

<div class="card">
<h2>Datos del circuito</h2>

<div class="form">
  <div>
    <label>Resistencia R</label>
    <div class="row">
      <input id="R" type="number" value="100">
      <select id="Ru">
        <option value="ohm">Ω</option>
        <option value="kohm">kΩ</option>
      </select>
    </div>
  </div>

  <div>
    <label>Inductancia L</label>
    <div class="row">
      <input id="L" type="number" value="10">
      <select id="Lu">
        <option value="H">H</option>
        <option value="mH">mH</option>
      </select>
    </div>
  </div>

  <div>
    <label>Corriente inicial I₀ (A)</label>
    <input id="I0" type="number" value="1">
  </div>

  <div>
    <label>Tiempo máximo (s)</label>
    <input id="tmax" type="number" value="0.5">
  </div>
</div>

<button onclick="calcular()">Calcular</button>
</div>

<div class="card">
<h2>Gráfica I(t)</h2>
<canvas id="graf"></canvas>
</div>

<div class="results" id="resultados">
  <h2>Resultados numéricos y memoria de cálculo</h2>
  <p>Presiona <strong>Calcular</strong> para ver los resultados.</p>
</div>

<footer>
Aplicación educativa – Cálculo Diferencial
</footer>

</div>

<script>
function calcular(){
  let R = parseFloat(document.getElementById("R").value);
  let L = parseFloat(document.getElementById("L").value);
  const I0 = parseFloat(document.getElementById("I0").value);
  const tmax = parseFloat(document.getElementById("tmax").value);

  if(document.getElementById("Ru").value === "kohm") R *= 1000;
  if(document.getElementById("Lu").value === "mH") L *= 1e-3;

  if(R <= 0 || L <= 0 || I0 < 0 || tmax <= 0){
    alert("Datos no válidos");
    return;
  }

  const tau = L / R;

  ajustarCanvas();
  ctx.clearRect(0,0,canvas.width,canvas.height);

  const w = canvas.clientWidth;
  const h = canvas.clientHeight;

  const m = {l:60,r:20,t:20,b:50};
  const pw = w - m.l - m.r;
  const ph = h - m.t - m.b;

  /* ───────── EJES ───────── */
  ctx.strokeStyle="#94a3b8";
  ctx.lineWidth=1;
  ctx.beginPath();
  ctx.moveTo(m.l,m.t);
  ctx.lineTo(m.l,m.t+ph);
  ctx.lineTo(m.l+pw,m.t+ph);
  ctx.stroke();

  ctx.fillStyle="#e5e7eb";
  ctx.font="13px Arial";

  /* ───────── ESCALA Y (CORRIENTE) ───────── */
  const yTicks = 5;
  for(let i=0;i<=yTicks;i++){
    const I = (I0*i/yTicks).toFixed(2);
    const y = m.t + ph - (i/yTicks)*ph;
    ctx.fillText(I+" A", 5, y+4);
    ctx.strokeStyle="rgba(148,163,184,0.15)";
    ctx.beginPath();
    ctx.moveTo(m.l,y);
    ctx.lineTo(m.l+pw,y);
    ctx.stroke();
  }

  /* ───────── ESCALA X (TIEMPO) ───────── */
  const xTicks = 6;
  for(let i=0;i<=xTicks;i++){
    const t = (tmax*i/xTicks).toFixed(2);
    const x = m.l + (i/xTicks)*pw;
    ctx.fillText(t+" s", x-10, m.t+ph+20);
  }

  /* ───────── CURVA I(t) ───────── */
  ctx.beginPath();
  ctx.strokeStyle="#38bdf8";
  ctx.lineWidth=2;

  for(let i=0;i<=300;i++){
    const t = (i/300)*tmax;
    const I = I0*Math.exp(-t/tau);
    const x = m.l + (t/tmax)*pw;
    const y = m.t + ph - (I/I0)*ph;
    i===0 ? ctx.moveTo(x,y) : ctx.lineTo(x,y);
  }
  ctx.stroke();

  /* ───────── RESULTADOS ───────── */
  document.getElementById("resultados").innerHTML = `
    <h2>Resultados numéricos y memoria de cálculo</h2>
    <p><strong>Constante de tiempo:</strong> τ = ${tau.toExponential(3)} s</p>
    <p><strong>Ecuación:</strong> I(t) = ${I0} · e<sup>-t / ${tau.toExponential(3)}</sup></p>
    <ul>
      <li>I(0) = ${I0} A</li>
      <li>I(tₘₐₓ) = ${(I0*Math.exp(-tmax/tau)).toFixed(6)} A</li>
    </ul>
  `;
}
</script>

</body>
</html>
