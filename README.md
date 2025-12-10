# circuito-RL
<jeremmy-gabriel>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>RL Serie — Calculadora EDO 1º orden</title>
<style>
:root{
  --bg:#020617; --card:#0f172a; --accent:#38bdf8; --muted:#94a3b8; --tau:#22d3ee; --text:#e6eef8;
}
*{box-sizing:border-box}
body{margin:0;font-family:Arial,Helvetica,sans-serif;background:linear-gradient(180deg,#020617,#071224);color:var(--text)}
header{padding:18px;text-align:center;border-bottom:2px solid rgba(56,189,248,0.06)}
h1{margin:0;color:var(--accent)}
.container{max-width:1100px;margin:22px auto;padding:18px}
.card{background:var(--card);padding:18px;border-radius:12px;margin-bottom:18px;box-shadow:0 10px 30px rgba(0,0,0,.5)}
.form{display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:12px}
.row{display:flex;gap:8px;align-items:center}
label{display:block;color:var(--muted);font-size:14px;margin-bottom:6px}
input,select{width:100%;padding:10px;border-radius:8px;border:1px solid #334155;background:transparent;color:var(--text);font-family:monospace}
button{margin-top:12px;padding:10px 14px;border-radius:10px;border:0;background:linear-gradient(90deg,var(--accent),#60a5fa);color:#021025;font-weight:700;cursor:pointer}
canvas{width:100%;height:320px;border-radius:10px;border:1px solid #334155;background:linear-gradient(180deg,#021222,#04162b)}
.results{padding:16px;border-left:4px solid var(--accent);background:rgba(255,255,255,0.01);border-radius:8px;font-size:15px}
.note{color:var(--muted);font-size:13px;margin-top:8px}
.footer{ text-align:center;color:var(--muted);margin-top:14px;font-size:13px}
@media(max-width:900px){canvas{height:260px}}
</style>
</head>
<body>
<header>
  <h1>Calculadora RL — EDO 1º orden</h1>
  <div class="note">Ingresa cualquier valor y pulsa "Calcular y graficar".</div>
</header>

<main class="container">
  <section class="card">
    <h2 style="margin-top:0;color:var(--accent)">Parámetros</h2>
    <div class="form">
      <div>
        <label for="R_val">Resistencia R</label>
        <div class="row">
          <input id="R_val" type="number" step="any" placeholder="ej. 100">
          <select id="R_unit" style="width:110px">
            <option value="ohm">Ω</option>
            <option value="kohm">kΩ</option>
          </select>
        </div>
      </div>

      <div>
        <label for="L_val">Inductancia L</label>
        <div class="row">
          <input id="L_val" type="number" step="any" placeholder="ej. 10">
          <select id="L_unit" style="width:110px">
            <option value="H">H</option>
            <option value="mH">mH</option>
            <option value="uH">µH</option>
          </select>
        </div>
      </div>

      <div>
        <label for="I0">Corriente inicial I₀ (A)</label>
        <input id="I0" type="number" step="any" placeholder="ej. 1">
      </div>

      <div>
        <label for="tmax">Tiempo máximo tₘₐₓ (s)</label>
        <input id="tmax" type="number" step="any" placeholder="ej. 0.5">
      </div>

      <div>
        <label for="dt">Paso Δt (s) — opcional</label>
        <input id="dt" type="number" step="any" placeholder="ej. 0.001">
      </div>

      <div>
        <label for="Iobj">I_obj (A) — opcional</label>
        <input id="Iobj" type="number" step="any" placeholder="ej. 0.1">
      </div>
    </div>

    <div style="margin-top:12px">
      <button id="run">Calcular y graficar</button>
      <button id="reset" style="margin-left:8px;background:transparent;border:1px solid rgba(56,189,248,0.12);color:var(--accent)">Limpiar</button>
    </div>

    <div id="messages" class="note"></div>
  </section>

  <section class="card">
    <h2 style="margin-top:0;color:var(--accent)">Gráfica I(t)</h2>
    <canvas id="plot" aria-label="Grafica de corriente"></canvas>
    <div id="legend" class="note" style="margin-top:8px">Eje X en segundos (s). Eje Y en amperios (A). Línea punteada = τ.</div>
  </section>

  <section class="card results" id="results">
    <h3 style="margin-top:0;color:var(--accent)">Resultados numéricos y memoria de cálculo</h3>
    <div id="resbody">Presiona "Calcular y graficar" para mostrar resultados aquí.</div>
  </section>

  <div class="footer">Archivo local — funciona offline y en hosting</div>
</main>

<script>
// ===== utilidades =====
function toOhm(val, unit){
  if(!isFinite(val)) return NaN;
  if(unit==='kohm') return val*1e3;
  return val;
}
function toHenry(val, unit){
  if(!isFinite(val)) return NaN;
  if(unit==='mH') return val*1e-3;
  if(unit==='uH') return val*1e-6;
  return val;
}

// ===== canvas setup =====
const canvas = document.getElementById('plot');
const ctx = canvas.getContext('2d');
function adjustCanvas(){
  const dpr = window.devicePixelRatio || 1;
  const rect = canvas.getBoundingClientRect();
  // if rect.width is 0 (hidden or not yet laid out), use fallback
  const w = rect.width || 800;
  const h = rect.height || 320;
  canvas.width = w * dpr;
  canvas.height = h * dpr;
  ctx.setTransform(dpr,0,0,dpr,0,0);
  // clear logical size
  ctx.clearRect(0,0,canvas.clientWidth, canvas.clientHeight);
}
window.addEventListener('load', adjustCanvas);
window.addEventListener('resize', adjustCanvas);

// ===== dibujo con ejes y marcas =====
function drawCurve(data, I0, tView, tau, opts={}){
  adjustCanvas();
  const w = canvas.clientWidth, h = canvas.clientHeight;
  ctx.clearRect(0,0,w,h);
  const m = {l:64, r:24, t:20, b:44};
  const pw = w - m.l - m.r, ph = h - m.t - m.b;

  // ejes
  ctx.strokeStyle = 'rgba(255,255,255,0.06)';
  ctx.lineWidth = 1;
  ctx.beginPath();
  ctx.moveTo(m.l,m.t);
  ctx.lineTo(m.l,m.t+ph);
  ctx.lineTo(m.l+pw,m.t+ph);
  ctx.stroke();

  // Y ticks (corriente)
  const yTicks = 5;
  ctx.fillStyle = '#dff3ff';
  ctx.font = '13px monospace';
  for(let i=0;i<=yTicks;i++){
    const frac = i / yTicks;
    const I = I0 * frac;
    const y = m.t + ph - frac * ph;
    ctx.fillText(I.toFixed(2) + ' A', 6, y+4);
    // grid
    ctx.strokeStyle = 'rgba(255,255,255,0.02)'; ctx.beginPath();
    ctx.moveTo(m.l, y); ctx.lineTo(m.l+pw, y); ctx.stroke();
  }

  // X ticks (tiempo)
  const xTicks = 6;
  for(let i=0;i<=xTicks;i++){
    const frac = i / xTicks;
    const t = tView * frac;
    const x = m.l + frac * pw;
    ctx.fillText(t.toFixed(3) + ' s', x-14, m.t + ph + 20);
  }

  // curve
  ctx.beginPath();
  ctx.strokeStyle = '#60a5fa';
  ctx.lineWidth = 2;
  for(let i=0;i<data.length;i++){
    const pt = data[i];
    const x = m.l + (pt.t / tView) * pw;
    const y = m.t + ph - (pt.I / I0) * ph;
    if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
  }
  ctx.stroke();

  // Mark tau (only if tau <= tView*1.5 — if tau outside, position at edge)
  if(isFinite(tau) && tau > 0){
    const xTau = m.l + Math.min(tau / tView, 1) * pw;
    ctx.setLineDash([6,6]);
    ctx.strokeStyle = '#22d3ee';
    ctx.beginPath();
    ctx.moveTo(xTau, m.t);
    ctx.lineTo(xTau, m.t + ph);
    ctx.stroke();
    ctx.setLineDash([]);
    ctx.fillStyle = '#22d3ee';
    ctx.fillText('τ', xTau - 6, m.t - 6);
  }

  // highlight initial point
  ctx.fillStyle = '#a7f3d0';
  const x0 = m.l + 0 * pw;
  const y0 = m.t + ph - (data[0].I / I0) * ph;
  ctx.beginPath(); ctx.arc(x0, y0, 3, 0, Math.PI*2); ctx.fill();

  // optional: show I0 and I(tView) numeric near top
  ctx.fillStyle = '#dff3ff';
  ctx.font = '13px monospace';
  ctx.fillText('I(0)=' + data[0].I.toFixed(3) + ' A', m.l + 6, m.t + 14);
  ctx.fillText('I(' + tView.toFixed(3) + 's)=' + data[data.length-1].I.toFixed(6) + ' A', m.l + 6, m.t + 32);
}

// ===== cálculo y validación =====
document.getElementById('run').addEventListener('click', ()=>{

  // leer entradas (permitir campos vacíos: tratar como NaN)
  const Rv = parseFloat(document.getElementById('R_val').value);
  const Runit = document.getElementById('R_unit').value;
  const Lv = parseFloat(document.getElementById('L_val').value);
  const Lunit = document.getElementById('L_unit').value;
  const I0v = parseFloat(document.getElementById('I0').value);
  const tmaxv = parseFloat(document.getElementById('tmax').value);
  const dtv = parseFloat(document.getElementById('dt').value);
  const Iobjv = parseFloat(document.getElementById('Iobj').value);

  const msg = document.getElementById('messages');
  msg.textContent = '';

  // conversion
  const R = toOhm(Rv, Runit);
  const L = toHenry(Lv, Lunit);
  const I0 = isFinite(I0v) ? I0v : NaN;
  const tmax = isFinite(tmaxv) ? tmaxv : NaN;
  const dt = isFinite(dtv) ? dtv : NaN;
  const Iobj = isFinite(Iobjv) ? Iobjv : NaN;

  // validaciones
  if(!isFinite(R) || R <= 0){ msg.textContent = 'Ingrese R válido (>0).'; return; }
  if(!isFinite(L) || L <= 0){ msg.textContent = 'Ingrese L válido (>0).'; return; }
  if(!isFinite(I0) || I0 < 0){ msg.textContent = 'Ingrese I₀ válido (≥0).'; return; }
  if(!isFinite(tmax) || tmax <= 0){ msg.textContent = 'Ingrese t_max válido (>0).'; return; }

  // si dt no es válido, calcular pasos con 200 puntos
  let steps;
  if(isFinite(dt) && dt > 0){
    steps = Math.min(20000, Math.max(4, Math.ceil(tmax / dt)));
  } else {
    // fallback: 300 puntos
    steps = 300;
  }

  // constante de tiempo
  const tau = L / R;

  // decidir ventana de vista tView: mostrar hasta min(tmax, 5*tau) pero si tmax << tau, mostrar tmax
  // Si tmax > 8*tau, mostrar 5*tau en la ventana principal (el resto se mantiene disponible en resultados)
  let tView;
  if(tmax <= 8 * tau) tView = tmax;
  else tView = Math.min(tmax, 5 * tau);

  // generar datos con base en tView para la gráfica (y en total para la tabla)
  const dataForPlot = [];
  for(let k=0;k<=steps;k++){
    const t = (k/steps) * tView;
    const I = I0 * Math.exp(-t / tau);
    dataForPlot.push({t, I});
  }

  // calcular valores numéricos importantes
  const I_tmax = I0 * Math.exp(-tmax / tau);
  let t_obj_text = '—';
  if(isFinite(Iobj)){
    if(Iobj <= 0 || Iobj > I0) t_obj_text = 'I_obj fuera de rango (0 < I_obj ≤ I₀)';
    else {
      const t_obj = -tau * Math.log(Iobj / I0);
      t_obj_text = t_obj.toFixed(6) + ' s';
    }
  }

  // mostrar resultados
  const res = document.getElementById('resbody');
  res.innerHTML = `
    <div><strong>Entradas (convertidas):</strong> R = ${R.toFixed(6)} Ω, L = ${L.toExponential(6)} H, I₀ = ${I0} A, t_max = ${tmax} s</div>
    <div style="margin-top:8px"><strong>Constante de tiempo:</strong> τ = L / R = ${tau.toExponential(6)} s</div>
    <div style="margin-top:8px"><strong>Función:</strong> I(t) = ${I0} · e<sup>-t/τ</sup></div>
    <ul style="margin-top:8px">
      <li>I(0) = ${I0} A</li>
      <li>I(t_max) = ${I_tmax.toFixed(6)} A</li>
      <li>t_objetivo = ${t_obj_text}</li>
    </ul>
    <div style="margin-top:8px" class="note">La gráfica muestra hasta ${tView.toFixed(6)} s (ventana didáctica). Si quieres la curva completa, reduce Δt o ajusta t_max.</div>
  `;

  // dibujar
  drawCurve(dataForPlot, I0, tView, tau);

}); // fin run

// reset
document.getElementById('reset').addEventListener('click', ()=>{
  ['R_val','L_val','I0','tmax','dt','Iobj'].forEach(id => document.getElementById(id).value = '');
  document.getElementById('messages').textContent = '';
  document.getElementById('resbody').textContent = 'Presiona "Calcular y graficar" para mostrar resultados aquí.';
  adjustCanvas();
  ctx.clearRect(0,0,canvas.clientWidth, canvas.clientHeight);
});

// init: draw empty axes
adjustCanvas();
ctx.fillStyle = '#7f9fb6';
ctx.font = '13px monospace';
ctx.fillText('Ingrese parámetros y presione Calcular', 18, 22);
</script>
</body>
</html>
