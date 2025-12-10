# circuito-RL
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>RL – Análisis Completo</title>
<meta name="viewport" content="width=device-width, initial-scale=1">

<style>
body{
  font-family: Arial, Helvetica, sans-serif;
  background: #ffffff;
  color:#000;
  margin:0;
}
h1,h2,h3{margin:6px 0}
.container{
  max-width:1400px;
  margin:auto;
  padding:20px;
}
.card{
  border:1px solid #ddd;
  border-radius:10px;
  padding:15px;
  margin-bottom:15px;
}
input,select{
  padding:8px;
  border-radius:6px;
  border:1px solid #999;
  width:100%;
}
button{
  background:#2563eb;
  color:white;
  border:none;
  padding:10px 16px;
  border-radius:8px;
  font-weight:bold;
  cursor:pointer;
}
.grid{
  display:grid;
  grid-template-columns:repeat(auto-fit,minmax(200px,1fr));
  gap:10px;
}
.layout{
  display:grid;
  grid-template-columns:1fr 1.4fr;
  gap:15px;
}
canvas{
  width:100%;
  height:500px;
  border:1px solid #ccc;
  border-radius:8px;
}
.res-body{
  width:240px;
  height:40px;
  background:#f5f5f5;
  border-radius:8px;
  display:flex;
  align-items:center;
  justify-content:center;
}
.band{
  width:14px;
  height:70%;
  margin:4px;
}
table{
  width:100%;
  border-collapse:collapse;
}
th,td{
  border:1px solid #bbb;
  padding:6px;
  text-align:center;
}
th{background:#eee}
.muted{color:#555}
</style>
</head>

<body>
<div class="container">

<h1>📘 Circuito RL – Resolución Completa</h1>

<div class="card">
<h2>Datos de entrada</h2>
<div class="grid">
  <div><label>R (Ω)</label><input id="R" value="100"></div>
  <div><label>L (H)</label><input id="L" value="10"></div>
  <div><label>I₀ (A)</label><input id="I0" value="1"></div>
  <div><label>t<sub>max</sub> (s)</label><input id="tmax" value="0.5"></div>
</div>
<br>
<button onclick="calcular()">Calcular</button>
</div>

<div class="layout">

<!-- IZQUIERDA -->
<div>
  <div class="card">
    <h3>Resistencia utilizada</h3>
    <div class="res-body" id="res"></div>
    <div id="resTxt" class="muted"></div>
  </div>

  <div class="card">
    <h3>Análisis matemático</h3>
    <div id="analisis"></div>
  </div>

  <div class="card">
    <h3>Tabla de valores</h3>
    <div id="tabla"></div>
  </div>
</div>

<!-- DERECHA -->
<div>
  <div class="card">
    <h3>Gráfica I(t)</h3>
    <canvas id="grafica"></canvas>
  </div>

  <div class="card">
    <h3>Memoria de cálculo</h3>
    <div id="memoria"></div>
  </div>
</div>

</div>
</div>

<script>
const colores = ["black","brown","red","orange","yellow","green","blue","violet","gray","white"];
const hex = {
 black:"#000", brown:"#7B3F00", red:"#dc2626",
 orange:"#f97316", yellow:"#eab308",
 green:"#16a34a", blue:"#2563eb",
 violet:"#7c3aed", gray:"#6b7280"
};

function dibujarRes(R){
  const res=document.getElementById("res");
  res.innerHTML="";
  let exp=Math.floor(Math.log10(R))-1;
  let dig=Math.round(R/10**exp);
  let d1=Math.floor(dig/10),d2=dig%10;

  [d1,d2,exp].forEach(x=>{
    let b=document.createElement("div");
    b.className="band";
    b.style.background=hex[colores[x]];
    res.appendChild(b);
  });
  document.getElementById("resTxt").innerHTML=`R = ${R} Ω`;
}

function calcular(){
  const R=+R.value;
  const L=+L.value;
  const I0=+I0.value;
  const tmax=+tmax.value;
  const tau=L/R;

  dibujarRes(R);

  // análisis
  document.getElementById("analisis").innerHTML=`
  <b>Ecuación:</b> L dI/dt + RI = 0<br>
  dI/I = -(R/L) dt<br>
  <b>Solución:</b> I(t)=I₀·e<sup>-t/τ</sup><br>
  τ = L/R = ${tau.toExponential(3)} s
  `;

  // tabla corta
  document.getElementById("tabla").innerHTML=`
  <table>
  <tr><th>t (s)</th><th>I(t) (A)</th></tr>
  <tr><td>0</td><td>${I0.toFixed(4)}</td></tr>
  <tr><td>${tau.toExponential(3)}</td><td>${(I0*Math.exp(-1)).toFixed(4)}</td></tr>
  <tr><td>${tmax}</td><td>${(I0*Math.exp(-tmax/tau)).toFixed(4)}</td></tr>
  </table>
  `;

  // memoria
  document.getElementById("memoria").innerHTML=`
  Datos: R=${R}Ω, L=${L}H, I₀=${I0}A<br>
  τ=${tau.toExponential(4)} s<br>
  I(tmax)=${(I0*Math.exp(-tmax/tau)).toFixed(6)} A
  `;

  graficar(I0,tau,tmax);
}

function graficar(I0,tau,tmax){
  const c=document.getElementById("grafica");
  const ctx=c.getContext("2d");
  c.width=1000;
  c.height=500;
  ctx.clearRect(0,0,c.width,c.height);

  const m=70;
  const w=c.width-m-30;
  const h=c.height-m-60;

  ctx.strokeStyle="#ddd";
  for(let i=0;i<=10;i++){
    ctx.beginPath();
    ctx.moveTo(m+i*w/10,m);
    ctx.lineTo(m+i*w/10,m+h);
    ctx.stroke();
  }
  for(let i=0;i<=10;i++){
    ctx.beginPath();
    ctx.moveTo(m,m+i*h/10);
    ctx.lineTo(m+w,m+i*h/10);
    ctx.stroke();
  }

  ctx.strokeStyle="#000";
  ctx.beginPath();
  ctx.moveTo(m,m);
  ctx.lineTo(m,m+h);
  ctx.lineTo(m+w,m+h);
  ctx.stroke();

  ctx.strokeStyle="#2563eb";
  ctx.lineWidth=2;
  ctx.beginPath();
  for(let i=0;i<=500;i++){
    let t=i/500*tmax;
    let I=I0*Math.exp(-t/tau);
    let x=m+t/tmax*w;
    let y=m+h-(I/I0)*h;
    i==0?ctx.moveTo(x,y):ctx.lineTo(x,y);
  }
  ctx.stroke();
}
</script>
</body>
</html>
