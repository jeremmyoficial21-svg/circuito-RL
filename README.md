# circuito-RL
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Circuito RL – EDO Primer Orden</title>

<style>
body{
  margin:0;
  font-family: Arial, sans-serif;
  background:#ffffff;
  color:#000;
}
header{
  padding:20px;
  border-bottom:2px solid #ddd;
}
.container{
  max-width:1200px;
  margin:auto;
  padding:20px;
}
.card{
  border:1px solid #ccc;
  padding:20px;
  border-radius:8px;
  margin-bottom:20px;
}
.layout{
  display:grid;
  grid-template-columns:1fr 1.2fr;
  gap:20px;
}
label{
  display:block;
  margin-top:10px;
}
input{
  width:100%;
  padding:8px;
}
button{
  margin-top:15px;
  padding:10px;
  background:#2563eb;
  color:#fff;
  border:none;
  font-weight:bold;
  border-radius:5px;
  cursor:pointer;
}
canvas{
  width:100%;
  height:350px;
  border:1px solid #999;
}
.resistor-box{
  background:#f5f5f5;
  padding:15px;
  border-left:6px solid #2563eb;
}
.resistor{
  display:flex;
  align-items:center;
  margin:15px 0;
}
.resistor .wire{
  width:30px;
  height:4px;
  background:black;
}
.resistor .body{
  width:120px;
  height:30px;
  background:#fbbf24;
  display:flex;
  align-items:center;
}
.band{
  width:12px;
  height:100%;
  margin:0 4px;
}
.grid-note{
  font-size:13px;
  color:#333;
}
@media(max-width:900px){
  .layout{grid-template-columns:1fr}
}
</style>
</head>

<body>

<header>
<h1>Circuito RL – Ecuación Diferencial de Primer Orden</h1>
</header>

<div class="container">

<div class="card">
<h2>Parámetros</h2>
<label>Resistencia R (Ω)</label>
<input id="R" type="number" value="100">

<label>Inductancia L (H)</label>
<input id="L" type="number" value="10">

<label>Corriente inicial I₀ (A)</label>
<input id="I0" type="number" value="1">

<label>Tiempo máximo (s)</label>
<input id="tmax" type="number" value="0.5">

<button onclick="calcular()">Calcular</button>
</div>

<div class="layout">

<!-- GRAFICA -->
<div class="card">
<h2>Gráfica I(t)</h2>
<canvas id="graf"></canvas>
<div class="grid-note">
Eje X: Tiempo (s) · Eje Y: Corriente (A) · Cuadrícula para mayor precisión
</div>
</div>

<!-- RESULTADOS -->
<div>

<div class="card resistor-box">
<h3>Resistencia utilizada</h3>

<div class="resistor">
  <div class="wire"></div>
  <div class="body" id="bandas"></div>
  <div class="wire"></div>
</div>

<div id="res-info"></div>
</div>

<div class="card">
<h3>Análisis matemático</h3>
<div id="info"></div>
</div>

</div>
</div>
</div>

<script>
const colores = [
  "black","brown","red","orange","yellow",
  "green","blue","violet","gray","white"
];

function crearBandas(R){
  const cuerpo = document.getElementById("bandas");
  cuerpo.innerHTML = "";

  let exp = Math.floor(Math.log10(R));
  let base = Math.round(R / Math.pow(10, exp-1));

  let b1 = Math.floor(base/10);
  let b2 = base%10;
  let mult = exp-1;

  [b1,b2,mult].forEach(i=>{
    let div=document.createElement("div");
    div.className="band";
    div.style.background=colores[i];
    cuerpo.appendChild(div);
  });
}

function calcular(){
  const R=+Rinput.value, L=+Linput.value, I0=+I0input.value, tmax=+tmaxinput.value;
  if(!R||!L||!I0||!tmax)return;

  crearBandas(R);

  document.getElementById("res-info").innerHTML=
    `<b>Valor:</b> ${R} Ω<br>Código de colores según norma.`

  const tau=L/R;

  document.getElementById("info").innerHTML=`
  Ecuación diferencial:<br>
  dI/dt + (R/L)·I = 0<br><br>
  Solución:<br>
  I(t) = I₀ · e<sup>-t/τ</sup><br><br>
  τ = ${tau.toFixed(4)} s
  `;

  graficar(R,L,I0,tmax);
}

function graficar(R,L,I0,tmax){
  const c=graf,ctx=c.getContext("2d");
  c.width=c.clientWidth;
  c.height=350;

  const m={l:60,r:20,t:20,b:50};
  const w=c.width-m.l-m.r;
  const h=c.height-m.t-m.b;

  ctx.clearRect(0,0,c.width,c.height);

  // CUADRICULA
  ctx.strokeStyle="#e5e7eb";
  for(let i=0;i<=10;i++){
    let x=m.l+w*i/10;
    let y=m.t+h*i/10;
    ctx.beginPath();ctx.moveTo(x,m.t);ctx.lineTo(x,m.t+h);ctx.stroke();
    ctx.beginPath();ctx.moveTo(m.l,y);ctx.lineTo(m.l+w,y);ctx.stroke();
  }

  // EJES
  ctx.strokeStyle="#000";
  ctx.beginPath();
  ctx.moveTo(m.l,m.t);
  ctx.lineTo(m.l,m.t+h);
  ctx.lineTo(m.l+w,m.t+h);
  ctx.stroke();

  ctx.font="12px Arial";
  for(let i=0;i<=5;i++){
    ctx.fillText((tmax*i/5).toFixed(2)+" s",m.l+w*i/5-10,m.t+h+20);
    ctx.fillText((I0*i/5).toFixed(2)+" A",5,m.t+h-h*i/5);
  }

  // CURVA
  ctx.strokeStyle="#2563eb";
  ctx.lineWidth=2;
  ctx.beginPath();
  for(let i=0;i<=500;i++){
    let t=tmax*i/500;
    let I=I0*Math.exp(-t/(L/R));
    let x=m.l+w*t/tmax;
    let y=m.t+h*(1-I/I0);
    i?ctx.lineTo(x,y):ctx.moveTo(x,y);
  }
  ctx.stroke();
}

const Rinput=R,Linput=L,I0input=I0,tmaxinput=tmax;
</script>

</body>
</html>
