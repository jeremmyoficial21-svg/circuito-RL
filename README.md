# circuito-RL
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>RL Serie — Aplicación educativa (EDO 1º orden)</title>

  <style>
    :root{
      --bg:#0b1220; --card:#071024; --accent:#38bdf8; --accent2:#60a5fa; --muted:#99a3b3;
      --glass: rgba(255,255,255,0.04); --success:#22c55e;
      font-family: Inter, system-ui, -apple-system, 'Segoe UI', Roboto, Arial, Helvetica, sans-serif;
    }
    *{box-sizing:border-box}
    body{margin:0; background: radial-gradient(circle at 10% 10%, rgba(56,189,248,0.06), transparent 20%), linear-gradient(180deg,#041028 0%, #071024 100%); color:#e6eef8; min-height:100vh}
    header{padding:28px 24px;border-bottom:1px solid rgba(255,255,255,0.03); display:flex; align-items:center; gap:18px; justify-content:space-between}
    header h1{margin:0;font-size:20px;color:var(--accent)}
    header p{margin:0;color:var(--muted)}

    .container{max-width:1100px;margin:28px auto;padding:18px}
    .grid{display:grid;grid-template-columns:1fr 420px;gap:18px}
    .card{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); border:1px solid rgba(255,255,255,0.03); border-radius:12px;padding:18px; box-shadow:0 8px 30px rgba(2,6,23,0.6)}
    label{display:block;font-size:13px;color:var(--muted);margin-bottom:6px}
    input[type=number], select, input[type=text]{width:100%;padding:10px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:var(--accent2);font-family:monospace}
    .form-row{display:grid;grid-template-columns:repeat(auto-fit,minmax(160px,1fr));gap:12px}
    .small{width:120px}
    .actions{display:flex;gap:8px;margin-top:12px;flex-wrap:wrap}
    button{padding:10px 14px;border-radius:10px;border:0;background:linear-gradient(90deg,var(--accent),var(--accent2));color:#021025;font-weight:700;cursor:pointer}
    button.secondary{background:transparent;color:var(--accent);border:1px solid rgba(56,189,248,0.12)}
    .muted{color:var(--muted);font-size:13px}
    .note{font-size:13px;color:var(--muted);margin-top:10px}
    canvas{width:100%;height:320px;border-radius:10px;background:linear-gradient(180deg,#021222,#04162b);display:block}
    table{width:100%;border-collapse:collapse;margin-top:12px;font-size:13px;color:#dff3ff}
    th,td{padding:8px;border-bottom:1px dashed rgba(255,255,255,0.03);text-align:right}
    th{text-align:left;color:var(--muted);font-weight:600}
    .kpis{display:flex;gap:10px;margin-top:10px;flex-wrap:wrap}
    .kpi{background:var(--glass);padding:10px;border-radius:8px;color:var(--muted);min-width:120px;text-align:center}
    .kpi strong{display:block;color:var(--accent2);font-size:16px;margin-top:6px}
    .math{background:rgba(255,255,255,0.02);padding:10px;border-radius:8px;margin-top:10px;font-family:monospace;color:#d7eefc}
    footer{text-align:center;color:var(--muted);margin-top:18px;padding-bottom:30px}
    @media(max-width:980px){.grid{grid-template-columns:1fr}.small{width:100%} canvas{height:260px}}
  </style>
</head>
<body>
  <header>
    <div>
      <h1>Aplicación educativa — Circuito RL en Serie</h1>
      <p class="muted">EDO 1º orden homogénea · modelo, cálculo y visualización</p>
    </div>
    <div class="muted">Guarda: archivo único HTML · Funciona sin internet</div>
  </header>

  <main class="container">
    <div class="grid">
      <section class="card" aria-labelledby="formtitle">
        <h2 id="formtitle" style="margin:0 0 10px;color:var(--accent2)">Ingresar parámetros</h2>

        <div class="form-row">
          <div>
            <label>Resistencia R</label>
            <div style="display:flex;gap:8px">
              <input id="R_val" type="number" step="any" min="0" placeholder="ej. 100" value="100">
              <select id="R_unit" class="small"><option value="ohm">Ω</option><option value="kohm">kΩ</option></select>
            </div>
          </div>

          <div>
            <label>Inductancia L</label>
            <div style="display:flex;gap:8px">
              <input id="L_val" type="number" step="any" min="0" placeholder="ej. 10" value="10">
              <select id="L_unit" class="small"><option value="H">H</option><option value="mH">mH</option><option value="uH">µH</option></select>
            </div>
          </div>

          <div>
            <label>Corriente inicial I₀ (A)</label>
            <input id="I0" type="number" step="any" min="0" placeholder="ej. 1.5" value="1">
          </div>

          <div>
            <label>Tiempo máximo t_max (s)</label>
            <input id="tmax" type="number" step="any" min="0" placeholder="ej. 0.5" value="0.5">
          </div>

          <div>
            <label>Paso Δt (s)</label>
            <input id="dt" type="number" step="any" min="1e-6" placeholder="ej. 0.001" value="0.005">
          </div>

          <div>
            <label>Corriente objetivo I_obj (A) — opcional</label>
            <input id="Iobj" type="number" step="any" min="0" placeholder="ej. 0.1">
          </div>

          <div>
            <label>t medido (s) — opcional</label>
            <input id="tmeas" type="number" step="any" min="0" placeholder="ej. 0.02">
          </div>

          <div>
            <label>I medido (A) — opcional</label>
            <input id="Imeas" type="number" step="any" min="0" placeholder="ej. 0.35">
          </div>
        </div>

        <div class="actions">
          <button id="run" type="button">Calcular y graficar</button>
          <button id="reset" type="button" class="secondary">Limpiar</button>
          <button id="exportCsv" type="button" class="secondary">Exportar CSV</button>
        </div>

        <div id="messages" class="note" aria-live="polite"></div>

        <div class="math" style="margin-top:14px">
          <strong>Modelo:</strong><br>
          L · dI/dt + R · I = 0 &nbsp; → &nbsp;
          <span style="color:var(--accent2);font-weight:700">I(t) = I₀ · e<sup>-t / τ</sup></span>
          <div style="color:var(--muted);margin-top:6px">τ = L / R (constante de tiempo)</div>
        </div>
      </section>

      <aside class="card">
        <h3 style="margin-top:0;color:var(--accent2)">Gráfica I(t)</h3>
        <canvas id="plot" role="img" aria-label="Gráfica de corriente en función del tiempo"></canvas>

        <div class="kpis">
          <div class="kpi">τ <strong id="lbl_tau">— s</strong></div>
          <div class="kpi">I(0) <strong id="lbl_I0">— A</strong></div>
          <div class="kpi">I(t_max) <strong id="lbl_Iend">— A</strong></div>
        </div>

        <div class="note" style="margin-top:12px" id="interpretacion">La corriente decae exponencialmente: mayor R → τ menor → decaimiento más rápido.</div>
      </aside>
    </div>

    <section class="card" style="margin-top:18px">
      <h3 style="color:var(--accent2);margin-top:0">Resultados numéricos y memoria de cálculo</h3>
      <div id="results" class="note"></div>
      <div id="memoria" class="note" style="margin-top:10px"></div>
    </section>

    <footer>Proyecto educativo · EDO 1º orden · circuito RL — Puedes descargar/hostear el archivo como index.html</footer>
  </main>

<script>
/* --- unidades --- */
function toOhm(val, unit){
  if(isNaN(val)) return NaN;
  if(unit==='kohm') return val*1e3;
  return val;
}
function toHenry(val, unit){
  if(isNaN(val)) return NaN;
  if(unit==='mH') return val*1e-3;
  if(unit==='uH') return val*1e-6;
  return val;
}

/* --- canvas --- */
const plotCanvas = document.getElementById('plot');
const ctx = plotCanvas.getContext('2d');
function resizeCanvas(){
  const ratio = window.devicePixelRatio||1;
  plotCanvas.width = plotCanvas.clientWidth * ratio;
  plotCanvas.height = plotCanvas.clientHeight * ratio;
  ctx.setTransform(ratio,0,0,ratio,0,0);
}
resizeCanvas();
window.addEventListener('resize', resizeCanvas);
function clearPlot(){ ctx.clearRect(0,0,plotCanvas.width,plotCanvas.height); }

/* --- generar tabla --- */
function generateTable(data){
  let html = '<h4>Tabla de valores</h4>';
  html += '<table><thead><tr><th>t (s)</th><th>I(t) (A)</th></tr></thead><tbody>';
  data.forEach(d=> html += `<tr><td style="text-align:left">${d.t.toFixed(6)}</td><td>${d.I.toFixed(6)}</td></tr>`);
  html += '</tbody></table>';
  return html;
}

/* --- export CSV --- */
function exportCSVContent(data){
  let csv = 't(s),I(A)\n';
  data.forEach(d=> csv += `${d.t},${d.I}\n`);
  return csv;
}

/* --- dibujar --- */
function drawPlot(data, opts={}){
  // adaptative
  resizeCanvas();
  const w = plotCanvas.clientWidth, h = plotCanvas.clientHeight;
  ctx.clearRect(0,0,w,h);
  const m = {left:52,right:12,top:18,bottom:40};
  const pw = w - m.left - m.right, ph = h - m.top - m.bottom;
  // axes
  ctx.strokeStyle = 'rgba(255,255,255,0.06)'; ctx.lineWidth = 1;
  ctx.beginPath(); ctx.moveTo(m.left,m.top); ctx.lineTo(m.left,m.top+ph); ctx.lineTo(m.left+pw,m.top+ph); ctx.stroke();
  // ranges
  const tmin = data[0].t, tmax = data[data.length-1].t;
  const Imax = Math.max(...data.map(d=>d.I)), Imin = 0;
  // y ticks
  ctx.fillStyle = '#9fb6c8'; ctx.font = '12px monospace';
  const yTicks = 4;
  for(let i=0;i<=yTicks;i++){
    const y = m.top + ph - (i/yTicks)*ph;
    const val = (Imin + (i/yTicks)*(Imax - Imin));
    ctx.fillText(val.toFixed(3), 6, y+4);
    ctx.strokeStyle = 'rgba(255,255,255,0.02)'; ctx.beginPath(); ctx.moveTo(m.left,y); ctx.lineTo(m.left+pw,y); ctx.stroke();
  }
  // x ticks
  const xTicks = 6;
  for(let i=0;i<=xTicks;i++){
    const x = m.left + (i/xTicks)*pw;
    const val = tmin + (i/xTicks)*(tmax-tmin);
    ctx.fillText(val.toFixed(3), x-12, m.top+ph+22);
  }
  // curve
  ctx.beginPath();
  data.forEach((pt,i)=>{
    const x = m.left + ((pt.t - tmin)/(tmax-tmin || 1))*pw;
    const y = m.top + ph - ((pt.I - Imin)/(Imax - Imin || 1))*ph;
    if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
  });
  ctx.strokeStyle = '#38bdf8'; ctx.lineWidth = 2.2; ctx.stroke();
  // mark initial and optional objective
  const I0 = data[0].I;
  ctx.fillStyle = '#a7f3d0'; ctx.beginPath();
  const x0 = m.left + ((data[0].t - tmin)/(tmax-tmin || 1))*pw;
  const y0 = m.top + ph - ((I0 - Imin)/(Imax - Imin || 1))*ph;
  ctx.arc(x0,y0,3,0,Math.PI*2); ctx.fill();
  if(opts.Iobj != null){
    const idx = data.findIndex(d=>d.I <= opts.Iobj);
    if(idx>0){
      const pt = data[idx];
      const x = m.left + ((pt.t - tmin)/(tmax-tmin || 1))*pw;
      const y = m.top + ph - ((pt.I - Imin)/(Imax - Imin || 1))*ph;
      ctx.fillStyle = '#ffd166'; ctx.beginPath(); ctx.arc(x,y,3,0,Math.PI*2); ctx.fill();
    }
  }
}

/* --- main --- */
document.getElementById('run').addEventListener('click', ()=>{
  const Rv = parseFloat(document.getElementById('R_val').value);
  const Runit = document.getElementById('R_unit').value;
  const Lv = parseFloat(document.getElementById('L_val').value);
  const Lunit = document.getElementById('L_unit').value;
  const I0 = parseFloat(document.getElementById('I0').value);
  const tmax = parseFloat(document.getElementById('tmax').value);
  const dt = parseFloat(document.getElementById('dt').value);
  const Iobj = parseFloat(document.getElementById('Iobj').value);
  const tmeas = parseFloat(document.getElementById('tmeas').value);
  const Imeas = parseFloat(document.getElementById('Imeas').value);
  const messages = document.getElementById('messages'); messages.textContent='';

  const R = toOhm(Rv, Runit);
  const L = toHenry(Lv, Lunit);
  if(!isFinite(R) || R <= 0){ messages.textContent = 'Error: R debe ser un número positivo mayor que 0.'; return; }
  if(!isFinite(L) || L <= 0){ messages.textContent = 'Error: L debe ser un número positivo mayor que 0.'; return; }
  if(!isFinite(I0) || I0 < 0){ messages.textContent = 'Error: I₀ debe ser un número no negativo.'; return; }
  if(!isFinite(tmax) || tmax <= 0){ messages.textContent = 'Error: t_max debe ser mayor que 0.'; return; }
  if(!isFinite(dt) || dt <= 0){ messages.textContent = 'Error: Δt debe ser mayor que 0.'; return; }
  if(dt > tmax){ messages.textContent = 'Aviso: Δt es mayor a t_max, sólo se calculará en t=0 y t_max.'; }

  const tau = L / R;
  const steps = Math.min(20000, Math.max(2, Math.ceil(tmax / dt)));
  const data = [];
  for(let k=0;k<=steps;k++){
    const t = (k/steps) * tmax;
    const I = I0 * Math.exp(-t / tau);
    data.push({t, I});
  }

  const I_tmax = I0 * Math.exp(-tmax / tau);
  let t_obj = null, t_obj_text = '—';
  if(!isNaN(Iobj) && isFinite(Iobj)){
    if(Iobj <= 0 || Iobj > I0){ t_obj_text = 'No físicamente coherente (I_obj debe ser 0 < I_obj ≤ I₀).'; }
    else{ t_obj = -tau * Math.log(Iobj / I0); t_obj_text = t_obj.toFixed(6) + ' s'; }
  }

  let comparacionHtml = '';
  if(!isNaN(tmeas) && !isNaN(Imeas) && isFinite(tmeas) && isFinite(Imeas)){
    const I_at_tmeas = I0 * Math.exp(-tmeas / tau);
    const errRel = Math.abs((Imeas - I_at_tmeas) / (I_at_tmeas || 1)) * 100;
    comparacionHtml = `<div><strong>Medición experimental:</strong> I(t=${tmeas}s) modelo = ${I_at_tmeas.toFixed(6)} A — medido = ${Imeas} A — error relativo = ${errRel.toFixed(2)} %</div>`;
  }

  const results = document.getElementById('results');
  let html = `<div><strong>Datos ingresados:</strong> R=${Rv} ${Runit === 'kohm' ? 'kΩ' : 'Ω'} (convertido ${R.toFixed(6)} Ω), L=${Lv} ${Lunit} (convertido ${L.toExponential(6)} H), I₀=${I0} A, t_max=${tmax} s, Δt=${dt} s</div>`;
  html += `<div style="margin-top:8px"><strong>Constante de tiempo:</strong> τ = L/R = ${tau.toExponential(6)} s</div>`;
  html += `<div style="margin-top:6px"><strong>Función I(t):</strong> I(t) = ${I0} · e^{ - t / (${tau.toExponential(6)}) } A</div>`;
  html += `<div style="margin-top:8px"><strong>Resultados clave:</strong><ul style='padding-left:18px'><li>I(0) = ${I0} A</li><li>I(t_max) = ${I_tmax.toFixed(6)} A</li><li>t_objetivo = ${t_obj_text}</li></ul></div>`;
  if(comparacionHtml) html += `<div style="margin-top:8px">${comparacionHtml}</div>`;
  html += generateTable(data.slice(0, Math.min(data.length, 5000)));
  results.innerHTML = html;

  const memoria = document.getElementById('memoria');
  let mem = `<ol>`;
  mem += `<li>Se convierten unidades: R = ${R.toFixed(6)} Ω; L = ${L.toExponential(6)} H.</li>`;
  mem += `<li>Se calcula τ = L/R = ${tau.toExponential(6)} s.</li>`;
  mem += `<li>Se aplica I(t) = I₀ e^{-t/τ}. Por ejemplo, I(${tmax}) = ${I_tmax.toFixed(6)} A.</li>`;
  if(t_obj) mem += `<li>Para alcanzar I_obj = ${Iobj} A: t = -τ ln(I_obj/I₀) = ${t_obj.toFixed(6)} s.</li>`;
  if(comparacionHtml) mem += `<li>Comparación experimental: ${comparacionHtml}</li>`;
  mem += `</ol>`;
  memoria.innerHTML = mem;

  document.getElementById('lbl_tau').textContent = `τ = ${tau.toExponential(6)} s`;
  document.getElementById('lbl_I0').textContent = `I(0) = ${I0} A`;
  document.getElementById('lbl_Iend').textContent = `I(t_max) = ${I_tmax.toFixed(6)} A`;
  document.getElementById('interpretacion').textContent = `La corriente decae exponencialmente con constante de tiempo τ = ${tau.toExponential(6)} s. Si R aumenta (con L fijo), τ disminuye y la corriente decae más rápido. Si L aumenta (con R fijo), τ aumenta y la corriente decae más lentamente.`;

  try{ drawPlot(data, {Iobj: isFinite(Iobj) ? Iobj : null}); }catch(e){ console.error(e); }
  window._lastData = data;
});

document.getElementById('reset').addEventListener('click', ()=>{
  document.getElementById('R_val').value=''; document.getElementById('L_val').value=''; document.getElementById('I0').value='';
  document.getElementById('tmax').value=''; document.getElementById('dt').value=''; document.getElementById('Iobj').value='';
  document.getElementById('tmeas').value=''; document.getElementById('Imeas').value=''; document.getElementById('messages').textContent='';
  document.getElementById('results').innerHTML=''; document.getElementById('memoria').innerHTML='';
  document.getElementById('lbl_tau').textContent='— s'; document.getElementById('lbl_I0').textContent='— A'; document.getElementById('lbl_Iend').textContent='— A';
  clearPlot(); window._lastData = null;
});

document.getElementById('exportCsv').addEventListener('click', ()=>{
  const data = window._lastData;
  if(!data){ alert('No hay datos. Primero ejecute \"Calcular y graficar\".'); return; }
  const csv = exportCSVContent(data);
  const blob = new Blob([csv],{type:'text/csv'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a'); a.href = url; a.download = 'RL_I_vs_t.csv'; document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
});

// init empty
clearPlot();
</script>
</body>
</html>
