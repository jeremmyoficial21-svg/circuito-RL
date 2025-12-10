# circuito-RL
<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>RL Serie — Calculadora EDO 1º orden (completa)</title>
<style>
  :root{
    --bg: #ffffff; --text: #0f172a; --muted: #475569; --accent: #2563eb; --grid: #eef6ff; --tau: #ef4444;
  }
  *{box-sizing:border-box}
  body{margin:0;font-family:Inter, Arial, Helvetica, sans-serif;background:var(--bg);color:var(--text)}
  header{padding:18px;border-bottom:1px solid #eee}
  .container{max-width:1200px;margin:18px auto;padding:18px}
  .card{background:var(--bg);border:1px solid #e8eefb;border-radius:10px;padding:14px;margin-bottom:14px}
  h1,h2,h3{margin:6px 0}
  .flex{display:flex;gap:12px;align-items:center}
  .grid-in{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:12px}
  label{display:block;color:var(--muted);font-size:14px;margin-bottom:6px}
  input,select{width:100%;padding:9px;border-radius:8px;border:1px solid #dbe9ff;background:white;font-size:14px}
  button{padding:10px 14px;border-radius:8px;border:0;background:linear-gradient(90deg,var(--accent),#60a5fa);color:white;font-weight:700;cursor:pointer}
  .layout{display:grid;grid-template-columns:1fr 1.15fr;gap:14px}
  @media(max-width:980px){.layout{grid-template-columns:1fr}}
  canvas{width:100%;height:360px;border:1px solid #dbe9ff;border-radius:8px;background:white}
  .resistor-row{display:flex;gap:12px;align-items:center}
  .wire{width:36px;height:6px;background:#111;border-radius:3px}
  .res-body{width:220px;height:40px;border-radius:8px;background:#f8fafc;display:flex;align-items:center;justify-content:center;position:relative}
  .band{width:14px;height:72%;margin:0 6px;border-radius:2px}
  .results-box{border-left:4px solid var(--accent);padding:12px;background:#fbfdff;border-radius:6px}
  .memory{background:#fbfdff;padding:12px;border-radius:6px;border:1px solid #eef6ff}
  table{width:100%;border-collapse:collapse;margin-top:8px}
  th,td{border:1px solid #e6eef8;padding:8px;text-align:center}
  th{background:#f8fbff}
  .muted{color:var(--muted)}
  .small{font-size:13px;color:var(--muted)}
</style>
</head>
<body>
<header>
  <h1>Calculadora RL — Ecuación diferencial de 1º orden (completa)</h1>
  <div class="small">Ingresa R, L, I₀ y t_max. Se muestra la resistencia con código de colores, la gráfica con cuadrícula y la memoria paso a paso.</div>
</header>

<main class="container">
  <!-- Inputs -->
  <section class="card">
    <h2>Datos de entrada</h2>
    <div class="grid-in">
      <div>
        <label for="R_val">Resistencia R</label>
        <div style="display:flex;gap:8px">
          <input id="R_val" type="number" step="any" placeholder="ej. 100" />
          <select id="R_unit" style="width:110px">
            <option value="ohm">Ω</option>
            <option value="kohm">kΩ</option>
          </select>
        </div>
      </div>

      <div>
        <label for="L_val">Inductancia L</label>
        <div style="display:flex;gap:8px">
          <input id="L_val" type="number" step="any" placeholder="ej. 10" />
          <select id="L_unit" style="width:110px">
            <option value="H">H</option>
            <option value="mH">mH</option>
            <option value="uH">µH</option>
          </select>
        </div>
      </div>

      <div>
        <label for="I0">Corriente inicial I₀ (A)</label>
        <input id="I0" type="number" step="any" placeholder="ej. 1.0" />
      </div>

      <div>
        <label for="tmax">Tiempo máximo t_max (s)</label>
        <input id="tmax" type="number" step="any" placeholder="ej. 0.5" />
      </div>

      <div>
        <label for="dt">Paso Δt (s) — opcional</label>
        <input id="dt" type="number" step="any" placeholder="ej. 0.005" />
      </div>

      <div>
        <label for="Iobj">Corriente objetivo I_obj (A) — opcional</label>
        <input id="Iobj" type="number" step="any" placeholder="ej. 0.1" />
      </div>
    </div>

    <div style="margin-top:12px;display:flex;gap:10px;align-items:center;flex-wrap:wrap">
      <button id="runBtn">Calcular y graficar</button>
      <button id="resetBtn" style="background:#fff;color:var(--accent);border:1px solid #dbe9ff">Limpiar</button>
      <div id="msg" class="muted"></div>
    </div>
  </section>

  <!-- Main layout -->
  <div class="layout">
    <!-- Left: resistor, results, memory, table (compact) -->
    <div>
      <section class="card">
        <h3>Resistencia utilizada</h3>
        <div class="resistor-row">
          <div class="wire"></div>
          <div class="res-body" id="resBody">
            <!-- bands injected here -->
          </div>
          <div class="wire"></div>
        </div>
        <div id="resText" class="muted" style="margin-top:8px">Aún no hay una resistencia seleccionada.</div>
      </section>

      <section class="card">
        <h3>Resultados numéricos y análisis</h3>
        <div class="results-box" id="resultsBox">
          Presiona <strong>Calcular y graficar</strong> para ver resultados.
        </div>
      </section>

      <section class="card">
        <h3>Memoria de cálculo (resuelto paso a paso)</h3>
        <div class="memory" id="memoryBox">
          Aquí aparecerá el desarrollo paso a paso una vez calcules.
        </div>
      </section>

      <section class="card">
        <h3>Valores característicos (tabla corta)</h3>
        <div id="tableSmall" class="muted">La tabla se mostrará aquí después de calcular.</div>
      </section>
    </div>

    <!-- Right: plot and larger table sample -->
    <div>
      <section class="card">
        <h3>Gráfica I(t) — cuadrícula y ejes reales</h3>
        <canvas id="plot" aria-label="Gráfica de corriente I(t)"></canvas>
        <div style="margin-top:8px" class="small">Eje X = tiempo (s) — Eje Y = corriente (A). Línea punteada vertical = τ.</div>
      </section>

      <section class="card">
        <h3>Tabla (muestra breve)</h3>
        <div id="tableContainer" class="muted">Se mostrará una tabla corta con puntos clave.</div>
      </section>
    </div>
  </div>
</main>

<script>
/* ----------------- utilidades ----------------- */
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

/* DOM */
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
const tableSmall = document.getElementById('tableSmall');
const tableContainer = document.getElementById('tableContainer');

const plot = document.getElementById('plot');
const ctx = plot.getContext('2d');

function resizeCanvas(){
  const dpr = window.devicePixelRatio || 1;
  const rect = plot.getBoundingClientRect();
  const w = rect.width || 700;
  const h = rect.height || 360;
  plot.width = w * dpr;
  plot.height = h * dpr;
  ctx.setTransform(dpr,0,0,dpr,0,0);
  ctx.clearRect(0,0,plot.clientWidth, plot.clientHeight);
}
window.addEventListener('load', resizeCanvas);
window.addEventListener('resize', resizeCanvas);

/* resistor colors */
const colorNames = ["black","brown","red","orange","yellow","green","blue","violet","gray","white"];
const colorHex = {
  black:"#000000", brown:"#7b3f00", red:"#dc2626", orange:"#fb923c", yellow:"#facc15",
  green:"#16a34a", blue:"#2563eb", violet:"#7c3aed", gray:"#6b7280", white:"#f8fafc", gold:"#d4af37"
};

function computeBandsFromOhms(R){
  if(R <= 0 || !isFinite(R)) return null;
  // generate 2 significant digits and multiplier for 4-band
  let exp = Math.floor(Math.log10(R));
  // adjust to get two significant digits
  let multiplier = exp - 1;
  let digits = Math.round(R / Math.pow(10, multiplier));
  if(digits >= 100){ // edge case rounding caused extra digit
    digits = Math.round(digits/10);
    multiplier += 1;
  }
  // digits must be 10..99
  digits = Math.max(10, Math.min(99, digits));
  const d1 = Math.floor(digits / 10) % 10;
  const d2 = digits % 10;
  const mult = Math.max(-2, Math.min(9, multiplier)); // clamp for display
  return {d1, d2, mult};
}

function renderResistorBands(R_original){
  resBody.innerHTML = '';
  const bands = computeBandsFromOhms(R_original);
  if(!bands){
    resText.textContent = 'Valor no válido';
    return;
  }
  // left spacer
  const spacerL = document.createElement('div'); spacerL.style.width='12px';
  resBody.appendChild(spacerL);

  // band elements
  const b1 = document.createElement('div'); b1.className='band'; b1.style.background = colorHex[colorNames[bands.d1]];
  const b2 = document.createElement('div'); b2.className='band'; b2.style.background = colorHex[colorNames[bands.d2]];
  const bm = document.createElement('div'); bm.className='band'; bm.style.background = colorHex[colorNames[Math.max(0,Math.min(9,bands.mult))]];
  const tol = document.createElement('div'); tol.className='band'; tol.style.width='10px'; tol.style.background = colorHex['gold'];
  resBody.appendChild(b1); resBody.appendChild(b2); resBody.appendChild(bm); resBody.appendChild(tol);
  // right spacer
  const spacerR = document.createElement('div'); spacerR.style.width='12px';
  resBody.appendChild(spacerR);
  // text
  resText.innerHTML = `<strong>R = ${R_original.toLocaleString('en-US')} Ω</strong>`;
}

/* ---------- draw plot with grid, axes ticks, tau marker ---------- */
function drawPlot(data, I0, tmax, tau){
  resizeCanvas();
  ctx.clearRect(0,0,plot.clientWidth, plot.clientHeight);
  const w = plot.clientWidth, h = plot.clientHeight;
  const margin = {l:72, r:24, t:28, b:64};
  const pw = w - margin.l - margin.r;
  const ph = h - margin.t - margin.b;

  // grid (light)
  ctx.lineWidth = 1;
  ctx.strokeStyle = '#eef6ff';
  const gridX = 10; const gridY = 10;
  for(let i=0;i<=gridX;i++){
    const x = margin.l + (i/gridX)*pw;
    ctx.beginPath(); ctx.moveTo(x, margin.t); ctx.lineTo(x, margin.t + ph); ctx.stroke();
  }
  for(let j=0;j<=gridY;j++){
    const y = margin.t + (j/gridY)*ph;
    ctx.beginPath(); ctx.moveTo(margin.l, y); ctx.lineTo(margin.l + pw, y); ctx.stroke();
  }

  // axes
  ctx.strokeStyle = '#0f172a'; ctx.lineWidth = 1.4;
  ctx.beginPath(); ctx.moveTo(margin.l, margin.t); ctx.lineTo(margin.l, margin.t + ph); ctx.lineTo(margin.l + pw, margin.t + ph); ctx.stroke();

  // Y ticks (current) - choose 5 ticks up to I0
  const yTicks = 5;
  ctx.fillStyle = '#0f172a'; ctx.font = '13px monospace';
  for(let i=0;i<=yTicks;i++){
    const frac = i / yTicks;
    const Ival = I0 * frac;
    const y = margin.t + ph - frac * ph;
    ctx.fillText(Ival.toFixed(4) + ' A', 6, y + 4);
    // small tick
    ctx.beginPath(); ctx.moveTo(margin.l - 6, y); ctx.lineTo(margin.l, y); ctx.stroke();
  }

  // X ticks (time) - choose 6 ticks from 0..tmax
  const xTicks = 6;
  for(let i=0;i<=xTicks;i++){
    const frac = i / xTicks;
    const tval = tmax * frac;
    const x = margin.l + frac * pw;
    ctx.fillText(tval.toFixed(4) + ' s', x - 18, margin.t + ph + 22);
    ctx.beginPath(); ctx.moveTo(x, margin.t + ph); ctx.lineTo(x, margin.t + ph + 6); ctx.stroke();
  }

  // curve: assume data spans 0..tmax in its t values
  ctx.beginPath();
  ctx.lineWidth = 2.2;
  ctx.strokeStyle = '#2563eb';
  for(let i=0;i<data.length;i++){
    const pt = data[i];
    const x = margin.l + (pt.t / tmax) * pw;
    const y = margin.t + ph - (pt.I / I0) * ph;
    if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
  }
  ctx.stroke();

  // mark tau (vertical dashed) if in range
  if(isFinite(tau) && tau >= 0){
    const tauFrac = Math.min(tau / tmax, 1); // clamp to chart
    const xTau = margin.l + tauFrac * pw;
    ctx.setLineDash([6,6]);
    ctx.strokeStyle = '#ef4444';
    ctx.beginPath(); ctx.moveTo(xTau, margin.t); ctx.lineTo(xTau, margin.t + ph); ctx.stroke();
    ctx.setLineDash([]);
    ctx.fillStyle = '#ef4444';
    ctx.fillText('τ = ' + tau.toExponential(6) + ' s', xTau + 6, margin.t + 14);
  }

  // annotate I(0) and I(tmax)
  ctx.fillStyle = '#0f172a';
  ctx.fillText('I(0) = ' + data[0].I.toFixed(6) + ' A', margin.l + 6, margin.t + 14);
  ctx.fillText('I(' + tmax.toFixed(6) + ' s) = ' + data[data.length-1].I.toFixed(6) + ' A', margin.l + 6, margin.t + 34);
}

/* ---------- generate short table (key points) ---------- */
function generateShortTable(I0, tau, tmax){
  const points = [
    {label:'0', t:0},
    {label:'τ', t:tau},
    {label:'2τ', t:2*tau},
    {label:'3τ', t:3*tau},
    {label:'t_max', t:tmax}
  ];
  let html = '<table><thead><tr><th>Punto</th><th>t (s)</th><th>I(t) (A)</th></tr></thead><tbody>';
  points.forEach(p=>{
    const t = p.t;
    // compute I(t)
    const I = I0 * Math.exp(-t / tau);
    html += `<tr><td>${p.label}</td><td>${(t).toFixed(6)}</td><td>${I.toFixed(6)}</td></tr>`;
  });
  html += '</tbody></table>';
  return html;
}

/* ---------- generate sample table (downsampled) ---------- */
function generateSampleTable(data, maxRows=120){
  const step = Math.max(1, Math.ceil(data.length / maxRows));
  let html = '<table><thead><tr><th>t (s)</th><th>I(t) (A)</th></tr></thead><tbody>';
  for(let i=0;i<data.length;i+=step){
    html += `<tr><td style="text-align:left">${data[i].t.toFixed(6)}</td><td>${data[i].I.toFixed(6)}</td></tr>`;
  }
  html += '</tbody></table>';
  return html;
}

/* ---------- main handler ---------- */
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

  // conversions
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

  // render resistor bands & text
  renderResistorBands(R);
  resText.innerHTML = `<strong>R (convertido) = ${R.toLocaleString('en-US')} Ω</strong> — entrada: ${Rv} ${Runit}`;

  // compute tau
  const tau = L / R;

  // steps for data (use dt if valid, otherwise default resolution)
  let steps;
  if(isFinite(dt) && dt > 0){
    steps = Math.min(20000, Math.max(20, Math.ceil(tmax / dt)));
  } else {
    steps = 600; // default resolution for smooth curve
  }

  // generate data over full 0..tmax for accurate data and display
  const data = [];
  for(let k=0;k<=steps;k++){
    const t = (k/steps) * tmax;
    const I = I0 * Math.exp(-t / tau);
    data.push({t, I});
  }

  // draw chart using full tmax domain
  drawPlot(data, I0, tmax, tau);

  // create short table and sample table
  tableSmall.innerHTML = generateShortTable(I0, tau, tmax);
  tableContainer.innerHTML = '<div style="margin-top:6px"><strong>Tabla (muestra)</strong></div>' + generateSampleTable(data, 120);

  // numerical results
  const I_tmax = I0 * Math.exp(-tmax / tau);
  let t_obj_text = '—';
  if(isFinite(Iobj)){
    if(Iobj <= 0 || Iobj > I0) t_obj_text = 'I_obj fuera de rango (0 < I_obj ≤ I₀)';
    else {
      const t_obj = -tau * Math.log(Iobj / I0);
      t_obj_text = t_obj.toFixed(6) + ' s';
    }
  }

  resultsBox.innerHTML = `
    <div style="font-weight:700;color:var(--accent);margin-bottom:8px">Resultados</div>
    <div><strong>Datos ingresados (convertidos):</strong> R = ${R.toLocaleString('en-US')} Ω, L = ${L.toExponential(6)} H, I₀ = ${I0} A, t_max = ${tmax} s</div>
    <div style="margin-top:8px"><strong>Constante de tiempo:</strong> τ = L / R = ${tau.toExponential(6)} s</div>
    <div style="margin-top:6px"><strong>Función:</strong> I(t) = ${I0} · e<sup>-t / (${tau.toExponential(6)})</sup> A</div>
    <ul style="margin-top:8px">
      <li>I(0) = ${I0} A</li>
      <li>I(t_max) = ${I_tmax.toFixed(6)} A</li>
      <li>t_objetivo = ${t_obj_text}</li>
    </ul>
  `;

  // memory step-by-step (ordered, clear)
  const mem = [];
  mem.push('<strong>1) Datos ingresados</strong>');
  mem.push(`<div class="muted">R (entrada) = ${Rv} ${Runit} → R = ${R.toLocaleString('en-US')} Ω</div>`);
  mem.push(`<div class="muted">L (entrada) = ${Lv} ${Lunit} → L = ${L.toExponential(6)} H</div>`);
  mem.push(`<div class="muted">I₀ = ${I0} A</div>`);
  mem.push(`<div class="muted">t_max = ${tmax} s</div>`);
  mem.push('<br/>');

  mem.push('<strong>2) Definición de τ</strong>');
  mem.push(`<div class="muted">τ = L / R = ${L.toExponential(6)} / ${R.toFixed(6)} = ${tau.toExponential(6)} s</div>`);
  mem.push('<br/>');

  mem.push('<strong>3) Resolución de la EDO</strong>');
  mem.push('<div class="muted">Partimos de: L·dI/dt + R·I = 0</div>');
  mem.push('<div class="muted">⇒ dI/dt = - (R/L) I</div>');
  mem.push('<div class="muted">Separando variables: (1/I) dI = - (R/L) dt</div>');
  mem.push('<div class="muted">Integrando: ln|I| = - (R/L) t + C</div>');
  mem.push('<div class="muted">Aplicando I(0)=I₀ → C = ln(I₀)</div>');
  mem.push('<div class="muted">Finalmente: I(t) = I₀ · e^{-(R/L) t} = I₀ · e^{-t/τ}</div>');
  mem.push('<br/>');

  mem.push('<strong>4) Sustituciones numéricas (ejemplos)</strong>');
  mem.push(`<div class="muted">I(0) = ${I0} A</div>`);
  mem.push(`<div class="muted">I(τ) = I₀ · e^{-1} = ${(I0 * Math.exp(-1)).toFixed(6)} A</div>`);
  mem.push(`<div class="muted">I(2τ) = ${(I0 * Math.exp(-2)).toFixed(6)} A</div>`);
  mem.push(`<div class="muted">I(t_max) = ${I_tmax.toFixed(6)} A</div>`);
  if(isFinite(Iobj)) mem.push(`<div class="muted">t para I = ${Iobj} A → t = -τ ln(I_obj/I₀) = ${t_obj_text}</div>`);
  mem.push('<br/>');

  mem.push('<strong>5) Tabla corta (puntos clave)</strong>');
  mem.push(generateShortTable(I0, tau, tmax));

  memoryBox.innerHTML = mem.join('');
});

/* reset */
resetBtn.addEventListener('click', ()=>{
  [R_val, L_val, I0_in, tmax_in, dt_in, Iobj_in].forEach(el => el.value = '');
  msg.textContent = '';
  resBody.innerHTML = '';
  resText.innerHTML = 'Aún no hay una resistencia seleccionada.';
  resultsBox.innerHTML = 'Presiona <strong>Calcular y graficar</strong> para ver resultados.';
  memoryBox.innerHTML = 'Aquí aparecerá el desarrollo paso a paso una vez calcules.';
  tableSmall.innerHTML = 'La tabla se mostrará aquí después de calcular.';
  tableContainer.innerHTML = '';
  resizeCanvas();
  ctx.clearRect(0,0,plot.clientWidth, plot.clientHeight);
});

/* init defaults for convenience */
window.addEventListener('load', ()=>{
  if(!R_val.value) R_val.value = 100;
  if(!L_val.value) L_val.value = 10;
  if(!I0_in.value) I0_in.value = 1;
  if(!tmax_in.value) tmax_in.value = 0.5;
  resizeCanvas();
});
</script>
</body>
</html>
