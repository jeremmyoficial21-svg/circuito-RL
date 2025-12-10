# circuito-RL
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>RL — EDO 1º orden (calculadora con memoria paso a paso)</title>
<style>
  :root{
    --bg:#ffffff; --card:#ffffff; --accent:#2563eb; --muted:#475569; --line:#e6eef8;
  }
  *{box-sizing:border-box}
  body{margin:0;font-family:Inter, Arial, Helvetica, sans-serif;background:var(--bg);color:#0f172a}
  header{padding:18px;border-bottom:1px solid #e6eef8}
  .container{max-width:1200px;margin:18px auto;padding:18px}
  .card{background:var(--card);border:1px solid #e6eef8;border-radius:10px;padding:16px;margin-bottom:16px}
  .grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:12px}
  label{display:block;color:var(--muted);font-size:14px;margin-bottom:6px}
  input[type=number],select{width:100%;padding:10px;border:1px solid #e6eef8;border-radius:8px;font-size:14px}
  button{padding:10px 14px;background:var(--accent);color:white;border:none;border-radius:8px;font-weight:700;cursor:pointer}
  .layout{display:grid;grid-template-columns:1fr 1.1fr;gap:16px}
  @media(max-width:920px){ .layout{grid-template-columns:1fr} }
  canvas{width:100%;height:360px;border:1px solid #dbe7ff;border-radius:10px;background:white}
  .resistor-box{display:flex;align-items:center;gap:16px;margin-top:12px}
  .wire{width:34px;height:4px;background:#111;border-radius:2px}
  .body{width:180px;height:36px;background:#f3f4f6;border-radius:6px;display:flex;align-items:center;justify-content:center;position:relative}
  .band{width:14px;height:100%;margin:0 4px;border-radius:2px}
  .res-info{font-size:15px;color:#0f172a}
  .results-title{font-weight:700;color:var(--accent);margin-bottom:8px}
  .results-box{border-left:4px solid var(--accent);padding:12px;background:#fbfdff;border-radius:6px}
  .memory{margin-top:12px;padding:12px;background:#fbfdff;border-radius:8px;border:1px solid #eef6ff}
  pre{white-space:pre-wrap;font-family:monospace;font-size:13px}
  table{width:100%;border-collapse:collapse;margin-top:8px}
  th,td{padding:8px;border:1px solid #e6eef8;text-align:right;font-size:13px}
  th{text-align:left;background:#f8fbff;color:#0f172a}
  .muted{color:var(--muted);font-size:13px}
</style>
</head>
<body>
<header>
  <h1>Calculadora RL — EDO 1º orden</h1>
  <div class="muted">Ingresa datos, grafica la respuesta I(t), y muestra memoria de cálculo paso a paso.</div>
</header>

<main class="container">
  <!-- Parámetros -->
  <section class="card">
    <h2 style="margin:0 0 8px 0">Parámetros del circuito</h2>
    <div class="grid">
      <div>
        <label for="R_val">Resistencia R</label>
        <div style="display:flex;gap:8px">
          <input id="R_val" type="number" placeholder="ej. 100" step="any">
          <select id="R_unit" style="width:110px">
            <option value="ohm">Ω</option>
            <option value="kohm">kΩ</option>
          </select>
        </div>
      </div>

      <div>
        <label for="L_val">Inductancia L</label>
        <div style="display:flex;gap:8px">
          <input id="L_val" type="number" placeholder="ej. 10" step="any">
          <select id="L_unit" style="width:110px">
            <option value="H">H</option>
            <option value="mH">mH</option>
            <option value="uH">µH</option>
          </select>
        </div>
      </div>

      <div>
        <label for="I0">Corriente inicial I₀ (A)</label>
        <input id="I0" type="number" placeholder="ej. 1" step="any">
      </div>

      <div>
        <label for="tmax">Tiempo máximo t_max (s)</label>
        <input id="tmax" type="number" placeholder="ej. 0.5" step="any">
      </div>

      <div>
        <label for="dt">Paso Δt (s) — opcional</label>
        <input id="dt" type="number" placeholder="ej. 0.005" step="any">
      </div>

      <div>
        <label for="Iobj">Corriente objetivo I_obj (A) — opcional</label>
        <input id="Iobj" type="number" placeholder="ej. 0.1" step="any">
      </div>
    </div>

    <div style="margin-top:12px;display:flex;gap:8px;flex-wrap:wrap">
      <button id="runBtn">Calcular y graficar</button>
      <button id="resetBtn" style="background:#f3f4f6;color:#0f172a">Limpiar</button>
      <div id="msg" class="muted" style="align-self:center"></div>
    </div>
  </section>

  <!-- Layout: resultados y grafica -->
  <div class="layout">
    <!-- Izq: Resistencia + memoria + resultados -->
    <div>
      <section class="card">
        <h3 style="margin:0 0 8px 0">Resistencia utilizada</h3>
        <div class="resistor-box">
          <div class="wire"></div>
          <div class="body" id="resBody">
            <!-- bandas aquí -->
          </div>
          <div class="wire"></div>
        </div>
        <div class="res-info" id="resText" style="margin-top:8px">Ninguna aún</div>
      </section>

      <section class="card">
        <h3 style="margin:0 0 8px 0">Resultados numéricos</h3>
        <div class="results-box" id="resultsBox">
          Presiona "Calcular y graficar" para ver resultados.
        </div>
      </section>

      <section class="card">
        <h3 style="margin:0 0 8px 0">Memoria de cálculo (paso a paso)</h3>
        <div class="memory" id="memoryBox">
          Aquí aparecerá el desarrollo detallado con los datos ingresados.
        </div>
      </section>
    </div>

    <!-- Der: Grafica -->
    <div>
      <section class="card">
        <h3 style="margin:0 0 8px 0">Gráfica I(t) — cuadrícula</h3>
        <canvas id="plot" aria-label="Gráfica de corriente I(t)"></canvas>
        <div style="margin-top:8px" class="muted">Eje X: tiempo (s) — Eje Y: corriente (A). Línea punteada = τ.</div>

        <!-- Tabla opcional -->
        <div id="tableContainer" style="margin-top:12px"></div>
      </section>
    </div>
  </div>
</main>

<script>
/* ---------- utilidades ---------- */
function toOhm(val, unit){
  if(!isFinite(val)) return NaN;
  if(unit==='kohm') return val * 1e3;
  return val;
}
function toHenry(val, unit){
  if(!isFinite(val)) return NaN;
  if(unit==='mH') return val * 1e-3;
  if(unit==='uH') return val * 1e-6;
  return val;
}

/* ---------- elementos ---------- */
const R_val = document.getElementById('R_val');
const R_unit = document.getElementById('R_unit');
const L_val = document.getElementById('L_val');
const L_unit = document.getElementById('L_unit');
const I0_in = document.getElementById('I0');
const tmax_in = document.getElementById('tmax');
const dt_in = document.getElementById('dt');
const Iobj_in = document.getElementById('Iobj');
const runBtn = document.getElementById('runBtn');
const resetBtn = document.getElementById('resetBtn');
const msg = document.getElementById('msg');

const resBody = document.getElementById('resBody');
const resText = document.getElementById('resText');
const resultsBox = document.getElementById('resultsBox');
const memoryBox = document.getElementById('memoryBox');
const plot = document.getElementById('plot');
const tableContainer = document.getElementById('tableContainer');

/* ---------- canvas helpers ---------- */
const ctx = plot.getContext('2d');
function resizeCanvas(){
  const dpr = window.devicePixelRatio || 1;
  const rect = plot.getBoundingClientRect();
  const w = rect.width || 800;
  const h = rect.height || 360;
  plot.width = w * dpr;
  plot.height = h * dpr;
  ctx.setTransform(dpr,0,0,dpr,0,0);
}
window.addEventListener('load', resizeCanvas);
window.addEventListener('resize', resizeCanvas);

/* ---------- resistor colors (4-band) ---------- */
const colorNames = ["black","brown","red","orange","yellow","green","blue","violet","gray","white"];
const colorHex = {
  black:"#000", brown:"#7b3f00", red:"#dc2626", orange:"#fb923c", yellow:"#facc15",
  green:"#16a34a", blue:"#2563eb", violet:"#7c3aed", gray:"#6b7280", white:"#f8fafc", gold:"#d4af37"
};

function computeBandsFromOhms(R){
  // handle small/large values robustly
  if(R <= 0) return null;
  // Express R as significant digits and multiplier (4-band: 2 digits + multiplier)
  // Find exponent (10^exp) such that first two digits are integer
  const exp = Math.floor(Math.log10(R));
  // get multiplier power such that we express R = digits * 10^multip
  // target digits has 2 digits length (10..99)
  const digits = Math.round(R / Math.pow(10, exp-1));
  let d1 = Math.floor(digits/10) % 10;
  let d2 = digits % 10;
  let multiplier = exp-1;
  // if multiplier outside 0..9, clamp and adapt (for very large/small values)
  multiplier = Math.max(Math.min(multiplier,9), -2); // allow -2..9 (we won't show gold/silver multipliers)
  // handle negative multiplier by mapping to small resistor bands (rare), keep it simple
  return {d1,d2,multiplier};
}

function renderResistorBands(R){
  const body = document.getElementById('resBody');
  body.innerHTML = '';
  const bands = computeBandsFromOhms(R);
  if(!bands){ body.textContent = 'Valor no válido'; return; }
  // create bands elements inside .body
  const bodyInner = document.createElement('div');
  bodyInner.style.display = 'flex';
  bodyInner.style.alignItems = 'stretch';
  bodyInner.style.height = '100%';
  bodyInner.style.padding = '0 6px';
  // space left
  const left = document.createElement('div');
  left.style.width = '8px';
  bodyInner.appendChild(left);
  // first digit
  const b1 = document.createElement('div');
  b1.className = 'band';
  b1.style.width = '14px';
  b1.style.background = colorHex[colorNames[bands.d1]] || '#000';
  bodyInner.appendChild(b1);
  // second digit
  const b2 = document.createElement('div');
  b2.className = 'band';
  b2.style.width = '14px';
  b2.style.background = colorHex[colorNames[bands.d2]] || '#000';
  bodyInner.appendChild(b2);
  // multiplier
  const multIndex = Math.max(0, Math.min(9, bands.multiplier));
  const bm = document.createElement('div');
  bm.className = 'band';
  bm.style.width = '14px';
  bm.style.background = colorHex[colorNames[multIndex]] || '#000';
  bodyInner.appendChild(bm);
  // tolerance (gold)
  const tol = document.createElement('div');
  tol.className = 'band';
  tol.style.width = '10px';
  tol.style.background = colorHex['gold'];
  bodyInner.appendChild(tol);

  // append to container
  body.appendChild(bodyInner);
}

/* ---------- draw grid + axes + curve ---------- */
function drawPlot(data, I0, tView, tau){
  resizeCanvas();
  ctx.clearRect(0,0,plot.clientWidth, plot.clientHeight);
  const w = plot.clientWidth, h = plot.clientHeight;
  const margin = {l:64, r:24, t:24, b:56};
  const pw = w - margin.l - margin.r;
  const ph = h - margin.t - margin.b;

  // light grid
  ctx.lineWidth = 1;
  ctx.strokeStyle = '#eef6ff';
  for(let i=0;i<=10;i++){
    const x = margin.l + (i/10)*pw;
    ctx.beginPath(); ctx.moveTo(x, margin.t); ctx.lineTo(x, margin.t + ph); ctx.stroke();
  }
  for(let j=0;j<=10;j++){
    const y = margin.t + (j/10)*ph;
    ctx.beginPath(); ctx.moveTo(margin.l, y); ctx.lineTo(margin.l + pw, y); ctx.stroke();
  }

  // axes
  ctx.strokeStyle = '#0f172a'; ctx.lineWidth = 1.5;
  ctx.beginPath(); ctx.moveTo(margin.l, margin.t); ctx.lineTo(margin.l, margin.t+ph); ctx.lineTo(margin.l+pw, margin.t+ph); ctx.stroke();

  // ticks Y (current) - 5 ticks
  ctx.fillStyle = '#0f172a'; ctx.font = '13px monospace';
  const yTicks = 5;
  const Imax = I0;
  for(let i=0;i<=yTicks;i++){
    const frac = i/yTicks;
    const Ival = Imax * frac;
    const y = margin.t + ph - frac*ph;
    ctx.fillText(Ival.toFixed(3) + ' A', 6, y+4);
  }

  // ticks X (time) - 6 ticks
  const xTicks = 6;
  for(let i=0;i<=xTicks;i++){
    const frac = i/xTicks;
    const tval = tView * frac;
    const x = margin.l + frac*pw;
    ctx.fillText(tval.toFixed(3) + ' s', x-16, margin.t + ph + 22);
  }

  // curve
  ctx.beginPath();
  ctx.lineWidth = 2.2;
  ctx.strokeStyle = '#2563eb';
  for(let i=0;i<data.length;i++){
    const pt = data[i];
    const x = margin.l + (pt.t / tView) * pw;
    const y = margin.t + ph - (pt.I / I0) * ph;
    if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
  }
  ctx.stroke();

  // mark tau (if visible in tView window)
  if(isFinite(tau) && tau > 0){
    const tauX = margin.l + Math.min(tau / tView, 1) * pw;
    ctx.setLineDash([6,6]);
    ctx.strokeStyle = '#22c1ee';
    ctx.beginPath(); ctx.moveTo(tauX, margin.t); ctx.lineTo(tauX, margin.t + ph); ctx.stroke();
    ctx.setLineDash([]);
    ctx.fillStyle = '#0f172a';
    ctx.fillText('τ', tauX - 6, margin.t - 6);
  }

  // annotate I(0) and I(tView)
  ctx.fillStyle = '#0f172a';
  ctx.fillText('I(0)=' + data[0].I.toFixed(4) + ' A', margin.l + 6, margin.t + 14);
  ctx.fillText('I(' + tView.toFixed(3) + ' s)=' + data[data.length-1].I.toFixed(6) + ' A', margin.l + 6, margin.t + 34);
}

/* ---------- generate table HTML ---------- */
function generateTable(data){
  let html = '<table><thead><tr><th>t (s)</th><th>I(t) (A)</th></tr></thead><tbody>';
  data.forEach(pt => {
    html += `<tr><td style="text-align:left">${pt.t.toFixed(6)}</td><td>${pt.I.toFixed(6)}</td></tr>`;
  });
  html += '</tbody></table>';
  return html;
}

/* ---------- main calculate handler ---------- */
runBtn.addEventListener('click', ()=>{
  msg.textContent = '';
  // read inputs
  const Rv = parseFloat(R_val.value);
  const Runit = R_unit.value;
  const Lv = parseFloat(L_val.value);
  const Lunit = L_unit.value;
  const I0v = parseFloat(I0_in.value);
  const tmaxv = parseFloat(tmax_in.value);
  const dtv = parseFloat(dt_in.value);
  const Iobjv = parseFloat(Iobj_in.value);

  // convert
  const R = toOhm(Rv, Runit);
  const L = toHenry(Lv, Lunit);
  const I0 = isFinite(I0v) ? I0v : NaN;
  const tmax = isFinite(tmaxv) ? tmaxv : NaN;
  const dt = isFinite(dtv) ? dtv : NaN;
  const Iobj = isFinite(Iobjv) ? Iobjv : NaN;

  // validations
  if(!isFinite(R) || R <= 0){ msg.textContent = 'Ingrese R válido (>0)'; return; }
  if(!isFinite(L) || L <= 0){ msg.textContent = 'Ingrese L válido (>0)'; return; }
  if(!isFinite(I0) || I0 < 0){ msg.textContent = 'Ingrese I₀ válido (≥0)'; return; }
  if(!isFinite(tmax) || tmax <= 0){ msg.textContent = 'Ingrese t_max válido (>0)'; return; }

  // resistor visual
  renderResistorBands(R);
  resText.innerHTML = `<strong>R = ${R.toLocaleString('en-US')} Ω</strong> (entrada: ${Rv} ${Runit})`;

  // compute tau
  const tau = L / R;

  // steps logic
  let steps;
  if(isFinite(dt) && dt > 0){
    steps = Math.min(20000, Math.max(20, Math.ceil(tmax / dt)));
  } else {
    steps = 500; // default resolution
  }

  // decide tView for plot: use min(tmax, 5*tau) unless tmax << tau
  let tView;
  if(tmax <= 8 * tau) tView = tmax;
  else tView = Math.min(tmax, 5 * tau);

  // generate data for plot (using tView)
  const data = [];
  for(let k=0;k<=steps;k++){
    const t = (k/steps) * tView;
    const I = I0 * Math.exp(-t / tau);
    data.push({t, I});
  }

  // also generate table for some points (downsample if too many)
  const tableData = data.length <= 200 ? data : data.filter((_,i)=> i % Math.ceil(data.length/200) === 0);

  // draw
  drawPlot(data, I0, tView, tau);
  tableContainer.innerHTML = '<div style="margin-top:8px"><strong>Tabla (muestra)</strong></div>' + generateTable(tableData);

  // numerical results
  const I_tmax = I0 * Math.exp(-tmax / tau);
  let t_obj_text = '—';
  if(isFinite(Iobj)){
    if(Iobj <= 0 || Iobj > I0) t_obj_text = 'I_obj fuera de rango';
    else {
      const t_obj = -tau * Math.log(Iobj / I0);
      t_obj_text = t_obj.toFixed(6) + ' s';
    }
  }

  resultsBox.innerHTML = `
    <div class="results-title">Resultados</div>
    <div><strong>Datos ingresados:</strong> R=${Rv} ${Runit} (convertido ${R.toLocaleString('en-US')} Ω), L=${Lv} ${Lunit} (convertido ${L.toExponential(6)} H), I₀=${I0} A, t_max=${tmax} s, Δt=${isFinite(dt)?dt+' s':'(auto)'}</div>
    <div style="margin-top:8px"><strong>Constante de tiempo:</strong> τ = L / R = ${tau.toExponential(6)} s</div>
    <div style="margin-top:6px"><strong>Función:</strong> I(t) = ${I0} · e<sup>-t/(${tau.toExponential(6)})</sup> A</div>
    <ul style="margin-top:8px">
      <li>I(0) = ${I0} A</li>
      <li>I(t_max) = ${I_tmax.toFixed(6)} A</li>
      <li>t_objetivo = ${t_obj_text}</li>
    </ul>
  `;

  // memory (step-by-step)
  const memoryHtml = [];
  memoryHtml.push('<strong>1) Datos ingresados</strong>');
  memoryHtml.push(`<div class="muted">R (entrada) = ${Rv} ${Runit} → R = ${R.toLocaleString('en-US')} Ω</div>`);
  memoryHtml.push(`<div class="muted">L (entrada) = ${Lv} ${Lunit} → L = ${L.toExponential(6)} H</div>`);
  memoryHtml.push(`<div class="muted">I₀ = ${I0} A</div>`);
  memoryHtml.push(`<div class="muted">t_max = ${tmax} s</div>`);
  memoryHtml.push('<br>');

  memoryHtml.push('<strong>2) Cálculo de la constante de tiempo</strong>');
  memoryHtml.push(`<div class="muted">τ = L / R</div>`);
  memoryHtml.push(`<pre>τ = ${L.toExponential(6)} / ${R.toFixed(6)} = ${tau.toExponential(6)} s</pre>`);
  memoryHtml.push('<br>');

  memoryHtml.push('<strong>3) Resolución de la EDO (separación de variables)</strong>');
  memoryHtml.push('<div class="muted">Partimos de: L·dI/dt + R·I = 0</div>');
  memoryHtml.push('<div class="muted">⇒ dI/dt = -(R/L) I</div>');
  memoryHtml.push('<div class="muted">Separando variables: (1/I) dI = -(R/L) dt</div>');
  memoryHtml.push('<div class="muted">Integrando: ∫(1/I) dI = ∫ -(R/L) dt</div>');
  memoryHtml.push('<pre>ln|I| = -(R/L) t + C</pre>');
  memoryHtml.push('<div class="muted">Aplicando condición inicial I(0)=I₀ → C = ln(I₀)</div>');
  memoryHtml.push('<div class="muted">Por tanto: I(t) = I₀ · e^{-(R/L) t} = I₀ · e^{-t/τ}</div>');
  memoryHtml.push('<br>');

  memoryHtml.push('<strong>4) Sustitución numérica (ejemplos)</strong>');
  memoryHtml.push(`<div class="muted">I(0) = ${I0} A</div>`);
  memoryHtml.push(`<div class="muted">I(τ) = I₀ · e^{-1} = ${ (I0*Math.exp(-1)).toFixed(6) } A</div>`);
  memoryHtml.push(`<div class="muted">I(t_max) = I₀ · e^{-t_max/τ} = ${ (I_tmax).toFixed(6) } A</div>`);
  if(isFinite(Iobj)) memoryHtml.push(`<div class="muted">t para I = ${Iobj} A → t = -τ ln(I_obj / I₀) = ${t_obj_text}</div>`);

  memoryHtml.push('<br>');
  memoryHtml.push('<strong>5) Tabla de valores (muestra)</strong>');
  memoryHtml.push(generateTable(tableData));

  memoryBox.innerHTML = memoryHtml.join('');
}); // end runBtn

// reset
resetBtn.addEventListener('click', ()=>{
  [R_val, L_val, I0_in, tmax_in, dt_in, Iobj_in].forEach(el => el.value = '');
  msg.textContent = '';
  resBody.innerHTML = '';
  resText.innerHTML = 'Ninguna aún';
  resultsBox.innerHTML = 'Presiona "Calcular y graficar" para ver resultados.';
  memoryBox.innerHTML = 'Aquí aparecerá el desarrollo detallado con los datos ingresados.';
  tableContainer.innerHTML = '';
  resizeCanvas();
  ctx.clearRect(0,0,plot.clientWidth, plot.clientHeight);
});

/* init: set placeholders with example values */
window.addEventListener('load', ()=>{
  // set example default values for convenience if fields empty
  if(!R_val.value) R_val.value = 100;
  if(!L_val.value) L_val.value = 10;
  if(!I0_in.value) I0_in.value = 1;
  if(!tmax_in.value) tmax_in.value = 0.5;
  resizeCanvas();
});
</script>
</body>
</html>
