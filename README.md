# circuito-RL
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Circuito RL – EDO Primer Orden</title>
<meta name="viewport" content="width=device-width, initial-scale=1">

<style>
body{
  font-family: Arial, sans-serif;
  background:white;
  margin:0;
  color:#1e293b;
}
header{
  padding:20px;
  border-bottom:2px solid #e5e7eb;
}
.container{
  max-width:1100px;
  margin:auto;
  padding:20px;
}
.card{
  border:1px solid #d1d5db;
  border-radius:8px;
  padding:20px;
  margin-bottom:20px;
}
label{
  font-size:14px;
}
input{
  width:100%;
  padding:8px;
  margin-top:4px;
  margin-bottom:10px;
}
button{
  padding:10px 15px;
  font-weight:bold;
  background:#2563eb;
  color:white;
  border:none;
  border-radius:6px;
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
  color:#475569;
}
</style>
</head>

<body>

<header>
<h1>Gráfica I(t) – Circuito RL</h1>
<div class="note">Solución de la ecuación diferencial de primer orden</div>
</header>

<div class="container">

<section class="card">
<h2>Parámetros</h2>
<label>Resistencia R (Ω)</label>
<input id="R" type="number" value="100">

<label>Inductancia L (H)</label>
<input id="L" type="number" value="10">

<label>Corriente inicial I₀ (A)</label>
<input id="I0" type="number" value="1">

<label>Tiempo máximo (s)</label>
<input id="tmax" type="number" value="0.5">

<button onclick="calcular()">Calcular y graficar</button>
</section>

<section class="card">
<h2>Gráfica I(t)</h2>
<canvas id="graf"></canvas>
<div class="note">Eje horizontal: tiempo (s). Eje vertical: corriente (A).</div>
</section>

<section class="card">
<h2>Resultados numéricos</h2>
<div id="res"></div>
</section>

</div>

<script>
function calcular(){
  const R=+document.getElementById("R").value;
  const L=+document.getElementById("L").value;
  const I0=+document.getElementById("I0").value;
  const tmax=+document.getElementById("tmax").value;

  const tau=L/R;

  const canvas=document.getElementById("graf");
  const ctx=canvas.getContext("2d");

  canvas.width=canvas.clientWidth;
  canvas.height=320;

  ctx.clearRect(0,0,canvas.width,canvas.height);

  const margin={l:60,r:20,t:20,b:50};
  const w=canvas.width-margin.l-margin.r;
  const h=canvas.height-margin.t-margin.b;

  // Ejes
  ctx.strokeStyle="#000";
  ctx.beginPath();
  ctx.moveTo(margin.l,margin.t);
  ctx.lineTo(margin.l,margin.t+h);
  ctx.lineTo(margin.l+w,margin.t+h);
  ctx.stroke();

  // Escalas
  ctx.font="13px Arial";
  ctx.fillStyle="#000";

  // Eje Y (corriente)
  for(let i=0;i<=5;i++){
    let I=I0*i/5;
    let y=margin.t+h-(h*i/5);
    ctx.fillText(I.toFixed(2)+" A",5,y+4);
    ctx.beginPath();
    ctx.moveTo(margin.l-5,y);
    ctx.lineTo(margin.l,y);
    ctx.stroke();
  }

  // Eje X (tiempo)
  for(let i=0;i<=5;i++){
    let t=tmax*i/5;
    let x=margin.l+(w*i/5);
    ctx.fillText(t.toFixed(2)+" s",x-10,margin.t+h+20);
    ctx.beginPath();
    ctx.moveTo(x,margin.t+h);
    ctx.lineTo(x,margin.t+h+5);
    ctx.stroke();
  }

  // Curva
  ctx.strokeStyle="#2563eb";
  ctx.lineWidth=2;
  ctx.beginPath();
  for(let i=0;i<=300;i++){
    let t=tmax*i/300;
    let I=I0*Math.exp(-t/tau);
    let x=margin.l+w*(t/tmax);
    let y=margin.t+h*(1-I/I0);
    i===0?ctx.moveTo(x,y):ctx.lineTo(x,y);
  }
  ctx.stroke();

  // Línea tau
  let xtau=margin.l+w*(tau/tmax);
  ctx.setLineDash([5,5]);
  ctx.strokeStyle="#dc2626";
  ctx.beginPath();
  ctx.moveTo(xtau,margin.t);
  ctx.lineTo(xtau,margin.t+h);
  ctx.stroke();
  ctx.setLineDash([]);

  ctx.fillStyle="#dc2626";
  ctx.fillText("τ",xtau+4,margin.t+15);

  document.getElementById("res").innerHTML=`
    τ = L / R = <b>${tau.toExponential(3)} s</b><br>
    I(t) = I₀·e<sup>-t/τ</sup><br>
    I(${tmax}s) = <b>${(I0*Math.exp(-tmax/tau)).toFixed(6)} A</b>
  `;
}
</script>

</body>
</html>
