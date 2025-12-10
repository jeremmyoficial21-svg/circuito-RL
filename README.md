# circuito-RL
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Circuito RL – EDO 1º Orden</title>

<style>
body{
  margin:0;
  font-family:Arial, sans-serif;
  background:#fff;
  color:#000;
}
header{
  padding:16px;
  border-bottom:2px solid #ddd;
}
.container{
  max-width:1200px;
  margin:auto;
  padding:20px;
}
.card{
  border:1px solid #ccc;
  border-radius:8px;
  padding:16px;
  margin-bottom:16px;
}
.layout{
  display:grid;
  grid-template-columns:1fr 1.1fr;
  gap:20px;
}
@media(max-width:900px){
  .layout{grid-template-columns:1fr}
}
label{margin-top:10px;display:block}
input{
  width:100%;
  padding:8px;
}
button{
  margin-top:14px;
  padding:10px;
  background:#2563eb;
  color:#fff;
  border:none;
  border-radius:5px;
  cursor:pointer;
}
canvas{
  width:100%;
  height:360px;
  border:1px solid #999;
}
h3{margin-top:0}
.axis{
  font-size:12px;
}
</style>
</head>

<body>

<header>
<h1>Circuito RL en Serie – Ecuación Diferencial de Primer Orden</h1>
</header>

<div class="container">

<div class="card">
<h2>Datos de entrada</h2>

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

<div class="card">
<h3>Gráfica I(t)</h3>
<canvas id="graf"></canvas>
</div>

<div class="card">
<h3>Valores relevantes</h3>
<div id="info"></div>
</div>

</div>
</div>

<script>
function calcular(){
  const R=+Rinput.value;
  const L=+Linput.value;
  const I0=+I0input.value;
  const tmax=+tmaxinput.value;
  if(!R||!L||!I0||!tmax)return;

  const tau=L/R;
  info.innerHTML=`
    τ = ${tau.toFixed(4)} s<br>
    I(τ) = ${(I0*Math.exp(-1)).toFixed(4)} A
  `;
  graficar(R,L,I0,tmax);
}

function graficar(R,L,I0,tmax){
  const c=graf,ctx=c.getContext("2d");
  c.width=c.clientWidth;
  c.height=360;
  ctx.clearRect(0,0,c.width,c.height);

  const m={l:70,r:20,t:20,b:60};
  const w=c.width-m.l-m.r;
  const h=c.height-m.t-m.b;
  const tau=L/R;

  // Cuadrícula
  ctx.strokeStyle="#e5e7eb";
  ctx.lineWidth=1;
  for(let i=0;i<=5;i++){
    let x=m.l+w*i/5;
    let y=m.t+h*i/5;
    ctx.beginPath();ctx.moveTo(x,m.t);ctx.lineTo(x,m.t+h);ctx.stroke();
    ctx.beginPath();ctx.moveTo(m.l,y);ctx.lineTo(m.l+w,y);ctx.stroke();
  }

  // Ejes
  ctx.strokeStyle="#000";
  ctx.beginPath();
  ctx.moveTo(m.l,m.t);
  ctx.lineTo(m.l,m.t+h);
  ctx.lineTo(m.l+w,m.t+h);
  ctx.stroke();

  ctx.font="12px Arial";
  ctx.fillStyle="#000";

  // Eje Y
  for(let i=0;i<=5;i++){
    let I=I0*i/5;
    let y=m.t+h-h*i/5;
    ctx.fillText(I.toFixed(2)+" A",10,y+4);
  }

  // Eje X
  for(let i=0;i<=5;i++){
    let t=tmax*i/5;
    let x=m.l+w*i/5;
    ctx.fillText(t.toFixed(2)+" s",x-12,m.t+h+25);
  }

  // Curva
  ctx.strokeStyle="#2563eb";
  ctx.lineWidth=2;
  ctx.beginPath();
  for(let i=0;i<=400;i++){
    let t=tmax*i/400;
    let I=I0*Math.exp(-t/tau);
    let x=m.l+w*t/tmax;
    let y=m.t+h*(1-I/I0);
    i?ctx.lineTo(x,y):ctx.moveTo(x,y);
  }
  ctx.stroke();

  // Punto en τ
  const xt=m.l+w*(tau/tmax);
  const yt=m.t+h*(1-Math.exp(-1));
  ctx.fillStyle="red";
  ctx.beginPath();
  ctx.arc(xt,yt,4,0,2*Math.PI);
  ctx.fill();

  ctx.fillText("τ",xt+6,yt-6);
}

const Rinput=R,Linput=L,I0input=I0,tmaxinput=tmax;
</script>

</body>
</html>
