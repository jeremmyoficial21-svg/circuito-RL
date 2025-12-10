# circuito-RL
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Circuito RL – EDO de Primer Orden</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
  margin:0;
  font-family:Arial, Helvetica, sans-serif;
  background:#ffffff;
  color:#1e293b;
}
header{
  background:#f1f5f9;
  padding:20px;
  text-align:center;
  border-bottom:2px solid #e2e8f0;
}
.container{
  max-width:1100px;
  margin:auto;
  padding:20px;
}
.card{
  background:#ffffff;
  padding:20px;
  border-radius:10px;
  border:1px solid #e5e7eb;
  margin-bottom:20px;
}
h1,h2{
  color:#0f172a;
}
label{
  font-size:14px;
  color:#334155;
}
input,select{
  width:100%;
  padding:10px;
  margin-top:6px;
  border-radius:6px;
  border:1px solid #cbd5f5;
  font-size:14px;
}
button{
  margin-top:15px;
  padding:12px 16px;
  font-weight:bold;
  background:#2563eb;
  color:white;
  border:none;
  border-radius:8px;
  cursor:pointer;
}
button:hover{
  background:#1d4ed8;
}
.grid{
  display:grid;
  grid-template-columns:repeat(auto-fit,minmax(200px,1fr));
  gap:16px;
}
canvas{
  width:100%;
  height:300px;
  border:1px solid #cbd5f5;
  border-radius:8px;
}
.note{
  font-size:14px;
  color:#475569;
  margin-top:10px;
}
.resistor-box{
  display:flex;
  align-items:center;
  gap:20px;
  margin-top:15px;
}
.resistor-label{
  font-size:16px;
  font-weight:bold;
}
.footer{
  text-align:center;
  font-size:13px;
  color:#64748b;
  margin-top:30px;
}
</style>
</head>

<body>

<header>
  <h1>Circuito RL – Aplicación de Ecuaciones Diferenciales</h1>
  <div class="note">Modelo real + matemática (EDO de primer orden)</div>
</header>

<div class="container">

<!-- PARÁMETROS -->
<section class="card">
<h2>Parámetros del circuito</h2>

<div class="grid">
  <div>
    <label>Resistencia R</label>
    <input type="number" id="R" placeholder="Ej: 470">
    <select id="Runit">
      <option value="ohm">Ω</option>
      <option value="kohm">kΩ</option>
    </select>
  </div>

  <div>
    <label>Inductancia L (H)</label>
    <input type="number" id="L" placeholder="Ej: 1">
  </div>

  <div>
    <label>Corriente inicial I₀ (A)</label>
    <input type="number" id="I0" placeholder="Ej: 1">
  </div>

  <div>
    <label>Tiempo máximo t (s)</label>
    <input type="number" id="tmax" placeholder="Ej: 0.5">
  </div>
</div>

<button onclick="calcular()">Calcular y graficar</button>
<div id="msg" class="note"></div>
</section>

<!-- RESISTOR -->
<section class="card">
<h2>Resistencia equivalente (código de colores)</h2>

<div class="resistor-box">
<svg id="resistorSVG" width="320" height="80"></svg>
<div class="resistor-label" id="resistorText">—</div>
</div>

<div class="note">
Código de colores estándar (4 bandas): 2 dígitos + multiplicador + tolerancia.
</div>
</section>

<!-- GRAFICA -->
<section class="card">
<h2>Gráfica I(t)</h2>
<canvas id="graf"></canvas>
<div class="note">Línea muestra la solución de la ecuación diferencial.</div>
</section>

<!-- RESULTADOS -->
<section class="card">
<h2>Resultados</h2>
<div id="res"></div>
</section>

<div class="footer">
Herramienta educativa – Cálculo diferencial aplicado a ingeniería eléctrica
</div>

</div>

<script>
const colores = [
  "black","brown","red","orange","yellow",
  "green","blue","violet","gray","white"
];

function colorHex(c){
  const map={
    black:"#000", brown:"#7c2d12", red:"#dc2626",
    orange:"#f97316", yellow:"#fde047", green:"#16a34a",
    blue:"#2563eb", violet:"#7c3aed", gray:"#9ca3af",
    white:"#f8fafc", gold:"#d97706"
  };
  return map[c];
}

function dibujarResistor(valor){
  let v=Math.round(valor);
  let exp=Math.floor(Math.log10(v));
  let base=Math.floor(v/10**(exp-1));
  let d1=Math.floor(base/10);
  let d2=base%10;
  let mult=exp-1;

  let bands=[colores[d1],colores[d2],colores[mult],"gold"];

  let svg=document.getElementById("resistorSVG");
  svg.innerHTML=`
  <rect x="10" y="35" width="60" height="10" fill="#64748b"/>
  <rect x="250" y="35" width="60" height="10" fill="#64748b"/>
  <rect x="70" y="20" rx="15" ry="15" width="180" height="40" fill="#e5e7eb" stroke="#475569"/>

  ${bands.map((c,i)=>
    `<rect x="${100+i*30}" y="20" width="14" height="40" fill="${colorHex(c)}"/>`
  ).join("")}
  `;

  document.getElementById("resistorText").textContent =
    `R = ${valor} Ω → bandas: ${bands.join(" – ")}`;
}

function calcular(){
  const Rv=+document.getElementById("R").value;
  const Ru=document.getElementById("Runit").value;
  const L=+document.getElementById("L").value;
  const I0=+document.getElementById("I0").value;
  const tmax=+document.getElementById("tmax").value;

  if(!Rv||!L||!I0||!tmax){
    msg.textContent="Complete todos los campos";
    return;
  }

  let R=(Ru==="kohm")?Rv*1000:Rv;
  let tau=L/R;

  dibujarResistor(R);

  let ctx=graf.getContext("2d");
  graf.width=graf.clientWidth;
  graf.height=300;
  ctx.clearRect(0,0,graf.width,graf.height);

  ctx.beginPath();
  ctx.strokeStyle="#2563eb";
  ctx.lineWidth=2;

  for(let i=0;i<=300;i++){
    let t=tmax*i/300;
    let I=I0*Math.exp(-t/tau);
    let x=50+(graf.width-80)*t/tmax;
    let y=260-200*I/I0;
    i===0?ctx.moveTo(x,y):ctx.lineTo(x,y);
  }
  ctx.stroke();

  res.innerHTML=`
  τ = L/R = <b>${tau.toExponential(3)} s</b><br>
  I(t) = I₀·e<sup>-t/τ</sup><br>
  I(${tmax}s) = <b>${(I0*Math.exp(-tmax/tau)).toFixed(6)} A</b>
  `;
}
</script>

</body>
</html> presione Calcular', 18, 22);
</script>
</body>
</html>
