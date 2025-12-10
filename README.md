# circuito-RL
<!DOCTYPE html>
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
  --tau:#22d3ee;
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
  padding:20px;
  border-radius:14px;
  margin-bottom:20px;
  box-shadow:0 10px 25px rgba(0,0,0,.35);
}

label{
  color:var(--muted);
  font-size:15px;
  margin-bottom:6px;
  display:block;
}

input,select{
  width:100%;
  padding:10px;
  border-radius:8px;
  border:1px solid #334155;
  background:#020617;
  color:white;
}

.form{
  display:grid;
  grid-template-columns:repeat(auto-fit,minmax(200px,1fr));
  gap:14px;
}

.row{
  display:grid;
  grid-template-columns:1fr 80px;
  gap:8px;
}

button{
  margin-top:15px;
  padding:12px;
  font-size:15px;
  font-weight:bold;
  background:var(--accent);
  border:none;
  border-radius:10px;
  cursor:pointer;
}

canvas{
  width:100%;
  height:320px;
  background:#020617;
  border-radius:12px;
  border:1px solid #334155;
}

.results{
  background:#020617;
  border-left:5px solid var(--accent);
  padding:18px;
  border-radius:12px;
  font-size:16px;
  line-height:1.6;
}

.results h2{margin-top:0;color:var(--accent)}
footer{text-align:center;color:var(--muted);font-size:13px}
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
      <select id="Ru"><option value="ohm">Ω</option><option value="kohm">kΩ</option></select>
    </div>
  </div>

  <div>
    <label>Inductancia L</label>
    <div class="row">
      <input id="L" type="number" value="10">
      <select id="Lu"><option value="H">H</option><option value="mH">mH</option></select>
    </div>
  </div>

  <div>
    <label>Corriente inicial I₀ (A)</label>
    <input id="I0" type="number" value="1">
  </div>

  <div>
    <label>Tiempo máximo tₘₐₓ (s)</label>
    <input id="tmax" type="number" value="0.5">
  </div>
</div>

<button onclick="calcular()">Calcular y graficar</button>
</div>

<div class="card">
<h2>Gráfica I(t)</h2>
<canvas id="graf"></canvas>
</div>

<div class="results" id="resultados">
<h2>Resultados</h2>
<p>Presiona <strong>Calcular</strong>.</p>
</div>

<footer>
Aplicación educativa – Cálculo Diferencial
</footer>

</div>

<script>
const canvas = document.getElementById("graf");
const ctx = canvas.getContext("2d");

function ajustarCanvas(){
  const dpr = window.devicePixelRatio || 1;
  const rect = canvas.getBoundingClientRect();
  canvas.width = rect.width * dpr;
  canvas.height = rect.height * dpr;
  ctx.setTransform(dpr,0,0,dpr,0,0);
}
window.addEventListener("load", ajustarCanvas);
window.addEventListener("resize", ajustarCanvas);

function calcular(){
  let R = +document.getElementById("R").value;
  let L = +document.getElementById("L").value;
  const I0 = +document.getElementById("I0").value;
  const tmax = +document.getElementById("tmax").value;

  if(document.getElementById("Ru").value==="kohm") R*=1000;
  if(document.getElementById("Lu").value==="mH") L*=1e-3;

  const tau = L/R;
  const tView = Math.min(tmax,5*tau);

  ajustarCanvas();
  ctx.clearRect(0,0,canvas.width,canvas.height);

  const w = canvas.clientWidth;
  const h = canvas.clientHeight;
  const m = {l:60,r:20,t:20,b:50};
  const pw = w-m.l-m.r;
  const ph = h-m.t-m.b;

  // Ejes
  ctx.strokeStyle="#94a3b8";
  ctx.beginPath();
  ctx.moveTo(m.l,m.t);
  ctx.lineTo(m.l,m.t+ph);
  ctx.lineTo(m.l+pw,m.t+ph);
  ctx.stroke();

  // Escala Y
  ctx.fillStyle="#e5e7eb";
  ctx.font="13px Arial";
  for(let i=0;i<=5;i++){
    const I=(I0*i/5).toFixed(2);
    const y=m.t+ph-(i/5)*ph;
    ctx.fillText(I+" A",5,y+4);
  }

  // Escala X
  for(let i=0;i<=5;i++){
    const t=(tView*i/5).toFixed(3);
    const x=m.l+(i/5)*pw;
    ctx.fillText(t+" s",x-10,m.t+ph+22);
  }

  // Curva
  ctx.beginPath();
  ctx.strokeStyle=varAccent="#38bdf8";
  ctx.lineWidth=2;
  for(let i=0;i<=400;i++){
    const t=(i/400)*tView;
    const I=I0*Math.exp(-t/tau);
    const x=m.l+(t/tView)*pw;
    const y=m.t+ph-(I/I0)*ph;
    i===0?ctx.moveTo(x,y):ctx.lineTo(x,y);
  }
  ctx.stroke();

  // Marca τ
  const xTau=m.l+(tau/tView)*pw;
  ctx.setLineDash([6,6]);
  ctx.strokeStyle="#22d3ee";
  ctx.beginPath();
  ctx.moveTo(xTau,m.t);
  ctx.lineTo(xTau,m.t+ph);
  ctx.stroke();
  ctx.setLineDash([]);
  ctx.fillStyle="#22d3ee";
  ctx.fillText("τ",xTau-4,m.t-5);

  // Resultados
  document.getElementById("resultados").innerHTML=`
    <h2>Resultados numéricos y memoria de cálculo</h2>
    <p><strong>Constante de tiempo:</strong> τ = L/R = ${tau.toExponential(3)} s</p>
    <p><strong>Función:</strong> I(t) = ${I0} · e<sup>-t / ${tau.toExponential(3)}</sup></p>
    <ul>
      <li>I(0) = ${I0} A</li>
      <li>I(tₘₐₓ) = ${(I0*Math.exp(-tmax/tau)).toFixed(6)} A</li>
    </ul>
  `;
}
</script>

</body>
</html>
