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
  height:350px;
  border:1px solid #999;
}
.resistor{
  display:flex;
  align-items:center;
  margin:10px 0;
}
.wire{width:30px;height:4px;background:black}
.body{
  width:120px;height:30px;
  background:#e5e7eb;
  display:flex;
  align-items:center;
}
.band{
  width:12px;height:100%;
  margin:0 4px;
}
table{
  width:100%;
  border-collapse:collapse;
  margin-top:10px;
}
th,td{
  border:1px solid #ccc;
  padding:6px;
  text-align:center;
}
th{background:#f3f4f6}
h3{margin-top:0}
</style>
</head>

<body>

<header>
<h1>Aplicación de Ecuaciones Diferenciales – Circuito RL en Serie</h1>
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

<!-- IZQUIERDA -->
<div>

<div class="card">
<h3>Resistencia utilizada</h3>
<div class="resistor">
  <div class="wire"></div>
  <div class="body" id="bandas"></div>
  <div class="wire"></div>
</div>
<div id="resInfo"></div>
</div>

<div class="card">
<h3>Memoria de cálculo</h3>
<div id="memoria"></div>
</div>

<div class="card">
<h3>Valores característicos</h3>
<div id="tabla"></div>
</div>

</div>

<!-- DERECHA -->
<div class="card">
<h3>Gráfica de la corriente I(t)</h3>
<canvas id="graf"></canvas>
</div>

</div>
</div>

<script>
const colores=["black","brown","red","orange","yellow","green","blue","violet","gray","white"];

function crearBandas(R){
  bandas.innerHTML="";
  let e=Math.floor(Math.log10(R));
  let b=Math.round(R/Math.pow(10,e-1));
  let d1=Math.floor(b/10), d2=b%10, m=e-1;
  [d1,d2,m].forEach(i=>{
    let div=document.createElement("div");
    div.className="band";
    div.style.background=colores[i];
    bandas.appendChild(div);
  });
}

function calcular(){
  let Rv=+R.value,Lv=+L.value,I0v=+I0.value,tv=+tmax.value;
  if(!Rv||!Lv||!I0v||!tv)return;

  crearBandas(Rv);
  resInfo.innerHTML=`R = <b>${Rv} Ω</b>`;

  let tau=Lv/Rv;

  memoria.innerHTML=`
<b>1. Datos</b><br>
R = ${Rv} Ω<br>
L = ${Lv} H<br>
I₀ = ${I0v} A<br><br>

<b>2. Ecuación diferencial</b><br>
L·dI/dt + R·I = 0<br><br>

<b>3. Resolución</b><br>
dI/I = -(R/L) dt<br>
ln(I) = -(R/L)t + C<br><br>

<b>4. Condición inicial</b><br>
I(0)=I₀ → C=ln(I₀)<br><br>

<b>5. Solución final</b><br>
I(t)= I₀·e<sup>-t/τ</sup><br>
τ = L/R = ${tau.toFixed(4)} s<br><br>

<b>6. Interpretación</b><br>
En t = τ la corriente cae al 36,8 % del valor inicial.
`;

  let puntos=[
    {n:"0",t:0},
    {n:"τ",t:tau},
    {n:"2τ",t:2*tau},
    {n:"3τ",t:3*tau},
    {n:"tₘₐₓ",t:tv}
  ];

  let html="<table><tr><th>Punto</th><th>t (s)</th><th>I(t) (A)</th></tr>";
  puntos.forEach(p=>{
    let I=I0v*Math.exp(-p.t/tau);
    html+=`<tr><td>${p.n}</td><td>${p.t.toFixed(3)}</td><td>${I.toFixed(4)}</td></tr>`;
  });
  html+="</table>";
  tabla.innerHTML=html;

  graficar(Rv,Lv,I0v,tv);
}

function graficar(R,L,I0,tmax){
  let ctx=graf.getContext("2d");
  graf.width=graf.clientWidth;
  graf.height=350;
  ctx.clearRect(0,0,graf.width,graf.height);

  let m=60,w=graf.width-m-20,h=graf.height-40;
  let tau=L/R;

  ctx.strokeStyle="#ddd";
  for(let i=0;i<=10;i++){
    let x=m+w*i/10,y=20+h*i/10;
    ctx.beginPath();ctx.moveTo(x,20);ctx.lineTo(x,20+h);ctx.stroke();
    ctx.beginPath();ctx.moveTo(m,y);ctx.lineTo(m+w,y);ctx.stroke();
  }

  ctx.strokeStyle="#000";
  ctx.beginPath();
  ctx.moveTo(m,20);ctx.lineTo(m,20+h);ctx.lineTo(m+w,20+h);
  ctx.stroke();

  ctx.strokeStyle="#2563eb";
  ctx.beginPath();
  for(let i=0;i<=400;i++){
    let t=tmax*i/400;
    let I=I0*Math.exp(-t/tau);
    let x=m+w*t/tmax;
    let y=20+h*(1-I/I0);
    if(i==0)ctx.moveTo(x,y); else ctx.lineTo(x,y);
  }
  ctx.stroke();
}
</script>

</body>
</html>
