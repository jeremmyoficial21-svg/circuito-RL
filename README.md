# circuito-RL
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Circuito RL – Ecuación Diferencial</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
  margin:0;
  font-family: Arial, Helvetica, sans-serif;
  background:#ffffff;
  color:#000000;
}
header{
  padding:20px;
  border-bottom:2px solid #e5e7eb;
}
h1,h2,h3{
  margin:0 0 10px 0;
}
.container{
  max-width:1200px;
  margin:auto;
  padding:20px;
}
.card{
  border:1px solid #d1d5db;
  border-radius:8px;
  padding:20px;
  margin-bottom:20px;
}
.layout{
  display:grid;
  grid-template-columns: 1fr 1.2fr;
  gap:20px;
}
label{
  font-size:14px;
}
input{
  width:100%;
  padding:8px;
  margin-top:4px;
  margin-bottom:12px;
}
button{
  padding:10px 16px;
  background:#2563eb;
  color:white;
  border:none;
  border-radius:6px;
  font-weight:bold;
  cursor:pointer;
}
canvas{
  width:100%;
  height:320px;
  border:1px solid #cbd5f5;
  border-radius:6px;
}
.note{
  font-size:14px;
  color:#374151;
}
.resistencia-box{
  padding:12px;
  background:#f8fafc;
  border-left:5px solid #2563eb;
  font-size:15px;
}
@media(max-width:900px){
  .layout{grid-template-columns:1fr}
}
</style>
</head>

<body>

<header>
<h1>Circuito RL – Aplicación de EDO de Primer Orden</h1>
<div class="note">
Modelo matemático y físico del decaimiento de la corriente
</div>
</header>

<div class="container">

<!-- PARAMETROS -->
<section class="card">
<h2>Parámetros del circuito</h2>

<label>Resistencia R (Ω)</label>
<input id="R" type="number" value="100">

<label>Inductancia L (H)</label>
<input id="L" type="number" value="10">

<label>Corriente inicial I₀ (A)</label>
<input id="I0" type="number" value="1">

<label>Tiempo máximo (s)</label>
<input id="tmax" type="number" value="0.5">

<button onclick="calcular()">Calcular</button>
</section>

<!-- CONTENIDO PRINCIPAL -->
<div class="layout">

<!-- GRAFICA -->
<section class="card">
<h2>Gráfica I(t)</h2>
<canvas id="graf"></canvas>
<div class="note">
Eje X: tiempo (s) — Eje Y: corriente (A)
</div>
</section>

<!-- RESULTADOS -->
<section>

<!-- RESISTENCIA USADA -->
<div class="card resistencia-box">
<h3>Resistencia utilizada en el problema</h3>
<div id="resistenciaInfo">—</div>
</div>

<!-- RESULTADOS NUMERICOS -->
<div class="card">
<h3>Resultados y análisis</h3>
<div id="resultados">—</div>
</div>

</section>
</div>

</div>

<script>
function calcular(){
  const R = +document.getElementById("R").value;
  const L = +document.getElementById("L").value;
  const I0 = +document.getElementById("I0").value;
  const tmax = +document.getElementById("tmax").value;

  if(!R || !L || !I0 || !tmax) return;

  const tau = L / R;

  // Mostrar resistencia usada
  document.getElementById("resistenciaInfo").innerHTML = `
    <b>R = ${R} Ω</b><br>
    La resistencia limita el paso de corriente y controla
    la rapidez del decaimiento.
  `;

  // Resultados
  document.getElementById("resultados").innerHTML = `
    <b>Ecuación diferencial:</b><br>
    dI/dt + (R/L) · I = 0<br><br>

    <b>Solución general:</b><br>
    I(t) = I₀ · e<sup>-t/τ</sup><br><br>

    <b>Constante de tiempo:</b><br>
    τ = L / R = <b>${tau.toExponential(3)} s</b><br><br>

    <b>Corriente final:</b><br>
    I(${tmax}s) = <b>${(I0*Math.exp(-tmax/tau)).toFixed(6)} A</b>
  `;

  // GRAFICA
  const canvas = document.getElementById("graf");
  const ctx = canvas.getContext("2d");

  canvas.width = canvas.clientWidth;
  canvas.height = 320;

  const m = {l:60,r:20,t:20,b:50};
  const w = canvas.width - m.l - m.r;
  const h = canvas.height - m.t - m.b;

  ctx.clearRect(0,0,canvas.width,canvas.height);

  // Ejes
  ctx.strokeStyle="#000";
  ctx.beginPath();
  ctx.moveTo(m.l,m.t);
  ctx.lineTo(m.l,m.t+h);
  ctx.lineTo(m.l+w,m.t+h);
  ctx.stroke();

  ctx.font="13px Arial";
  ctx.fillStyle="#000";

  // Eje Y
  for(let i=0;i<=5;i++){
    let I = I0*i/5;
    let y = m.t + h - h*i/5;
    ctx.fillText(I.toFixed(2)+" A", 5, y+4);
  }

  // Eje X
  for(let i=0;i<=5;i++){
    let t = tmax*i/5;
    let x = m.l + w*i/5;
    ctx.fillText(t.toFixed(2)+" s", x-12, m.t+h+20);
  }

  // Curva
  ctx.strokeStyle="#2563eb";
  ctx.lineWidth=2;
  ctx.beginPath();
  for(let i=0;i<=300;i++){
    let t = tmax*i/300;
    let I = I0*Math.exp(-t/tau);
    let x = m.l + w*(t/tmax);
    let y = m.t + h*(1 - I/I0);
    i===0?ctx.moveTo(x,y):ctx.lineTo(x,y);
  }
  ctx.stroke();

  // Línea tau
  let xtau = m.l + w*(tau/tmax);
  ctx.setLineDash([6,6]);
  ctx.strokeStyle="#dc2626";
  ctx.beginPath();
  ctx.moveTo(xtau,m.t);
  ctx.lineTo(xtau,m.t+h);
  ctx.stroke();
  ctx.setLineDash([]);

  ctx.fillStyle="#dc2626";
  ctx.fillText("τ", xtau+4, m.t+15);
}
</script>

</body>
</html>
