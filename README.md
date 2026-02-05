<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Pocket Corridor Periodic Table — Δ,V → R, sin²φ, Δmix</title>
  <style>
    :root{
      --bg:#0b0d12; --fg:#e8edf2; --muted:#a7b0bb; --card:#121725; --edge:#273047;
      --ok:#2cff8a; --mid:#ffcc33; --bad:#ff4d4d;
    }
    body{ margin:0; font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; background:var(--bg); color:var(--fg);}
    header{ padding:14px 16px; border-bottom:1px solid var(--edge); display:flex; gap:12px; align-items:center; flex-wrap:wrap;}
    header h1{ font-size:16px; margin:0; font-weight:700; letter-spacing:.2px;}
    header .sub{ color:var(--muted); font-size:12px; }
    .wrap{ display:grid; grid-template-columns: 1fr 340px; gap:14px; padding:14px; }
    @media (max-width: 980px){ .wrap{ grid-template-columns: 1fr; } }
    .card{ background:var(--card); border:1px solid var(--edge); border-radius:14px; padding:12px; }
    .grid{ display:grid; grid-template-columns: repeat(18, minmax(34px, 1fr)); gap:6px; }
    .cell{
      position:relative; height:46px; border-radius:10px; border:1px solid var(--edge);
      background: rgba(255,255,255,.02); cursor:pointer; user-select:none;
      display:flex; flex-direction:column; justify-content:center; padding:6px 6px 5px;
      transition: transform .05s ease, border-color .15s ease, box-shadow .15s ease;
    }
    .cell:hover{ transform: translateY(-1px); border-color:#3a4666; }
    .cell .sym{ font-weight:800; font-size:13px; line-height:1; }
    .cell .mini{ color:var(--muted); font-size:10px; line-height:1.1; margin-top:3px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis;}
    .cell.filled{ box-shadow: 0 0 0 1px rgba(255,255,255,.05) inset; }
    .cell.r_ok{ outline: 2px solid rgba(44,255,138,.22); }
    .cell.r_mid{ outline: 2px solid rgba(255,204,51,.22); }
    .cell.r_bad{ outline: 2px solid rgba(255,77,77,.22); }
    .cell.r_ok::after,.cell.r_mid::after,.cell.r_bad::after{
      content:""; position:absolute; right:6px; top:6px; width:8px; height:8px; border-radius:50%;
    }
    .cell.r_ok::after{ background:var(--ok); }
    .cell.r_mid::after{ background:var(--mid); }
    .cell.r_bad::after{ background:var(--bad); }

    .panel h2{ margin:0 0 10px; font-size:14px; }
    .row{ display:grid; grid-template-columns: 1fr 1fr; gap:10px; }
    .field{ display:flex; flex-direction:column; gap:6px; margin-bottom:10px; }
    label{ font-size:11px; color:var(--muted); }
    input, textarea{
      width:100%; box-sizing:border-box; border-radius:10px; border:1px solid var(--edge);
      background:#0f1422; color:var(--fg); padding:9px 10px; font-size:13px; outline:none;
    }
    input:focus, textarea:focus{ border-color:#3a4666; }
    .btns{ display:flex; gap:10px; flex-wrap:wrap; margin:10px 0 0; }
    button{
      border:1px solid var(--edge); background:#0f1422; color:var(--fg);
      padding:9px 10px; border-radius:10px; cursor:pointer; font-weight:700; font-size:13px;
    }
    button:hover{ border-color:#3a4666; }
    .kpi{ display:grid; grid-template-columns: 1fr; gap:10px; margin-top:10px; }
    .kpi .box{ border:1px solid var(--edge); border-radius:12px; padding:10px; background:rgba(255,255,255,.02); }
    .kpi .box .t{ color:var(--muted); font-size:11px; }
    .kpi .box .v{ font-size:16px; font-weight:900; margin-top:4px; }
    .small{ color:var(--muted); font-size:11px; line-height:1.35; margin-top:10px;}
    .mono{ font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", monospace; }
  </style>
</head>

<body>
<header>
  <h1>PCPT (operacional): <span class="mono">Δ,V → R, sin²φ, Δmix</span></h1>
  <div class="sub">Implementa diretamente o modelo 2×2 do seu manuscrito (Eq. <span class="mono">R_and_leakage</span>, <span class="mono">adiabatic_gap_student</span>).</div>
</header>

<div class="wrap">
  <div class="card">
    <div id="statline" class="small"></div>
    <div id="pt" class="grid" aria-label="tabela periódica"></div>
    <div class="small">
      <b>Regimes (por R = |Δ|/|V|):</b>
      <span class="mono">R ≥ 3</span> robusto · <span class="mono">1 ≤ R &lt; 3</span> transição · <span class="mono">R &lt; 1</span> corredor.
      <br/>Obs.: isto é uma régua operacional, não um teorema de Deus.
    </div>
  </div>

  <div class="card panel">
    <h2 id="selTitle">Selecione um elemento</h2>

    <div class="row">
      <div class="field">
        <label>Δ (eV) — separação diabática</label>
        <input id="inpDelta" type="number" step="0.001" placeholder="ex.: 0.50" />
      </div>
      <div class="field">
        <label>V (eV) — hibridização efetiva</label>
        <input id="inpV" type="number" step="0.001" placeholder="ex.: 0.20" />
      </div>
    </div>

    <div class="field">
      <label>Nota (opcional)</label>
      <textarea id="inpNote" rows="3" placeholder="ex.: s↔d, SOC, SC, etc."></textarea>
    </div>

    <div class="btns">
      <button id="btnSave">Salvar (este elemento)</button>
      <button id="btnClear">Limpar (este elemento)</button>
      <button id="btnAutofill">Auto-preencher 118 (prior heurístico)</button>
    </div>

    <div class="kpi">
      <div class="box">
        <div class="t">R = |Δ|/|V|</div>
        <div class="v mono" id="kpiR">—</div>
      </div>
      <div class="box">
        <div class="t">sin²φ = ½(1 − Δ/√(Δ²+4V²))</div>
        <div class="v mono" id="kpiSin2">—</div>
      </div>
      <div class="box">
        <div class="t">Δmix = √(Δ²+4V²) (e no centro: 2|V|)</div>
        <div class="v mono" id="kpiDmix">—</div>
      </div>
    </div>

    <div class="small">
      <b>Nota metodológica (sem misticismo):</b> o botão “auto” só coloca um <i>prior</i> coerente por bloco (s/p/d/f, SOC, SC),
      porque o seu artigo diz explicitamente que Δ e V devem vir de espectroscopia / downfolding. Aqui a UI serve para
      <i>materializar</i> os cálculos e o diagnóstico de regime. Se você quer valores reais por elemento, alimente Δ,V com seus dados.
    </div>
  </div>
</div>

<script>
/* =========================================================
   1) Layout (posição period/group) — simples e suficiente
   ========================================================= */
const ELEMENTS = [
  // sym, Z, period, group
  ["H",1,1,1], ["He",2,1,18],
  ["Li",3,2,1],["Be",4,2,2],["B",5,2,13],["C",6,2,14],["N",7,2,15],["O",8,2,16],["F",9,2,17],["Ne",10,2,18],
  ["Na",11,3,1],["Mg",12,3,2],["Al",13,3,13],["Si",14,3,14],["P",15,3,15],["S",16,3,16],["Cl",17,3,17],["Ar",18,3,18],
  ["K",19,4,1],["Ca",20,4,2],["Sc",21,4,3],["Ti",22,4,4],["V",23,4,5],["Cr",24,4,6],["Mn",25,4,7],["Fe",26,4,8],["Co",27,4,9],["Ni",28,4,10],["Cu",29,4,11],["Zn",30,4,12],
  ["Ga",31,4,13],["Ge",32,4,14],["As",33,4,15],["Se",34,4,16],["Br",35,4,17],["Kr",36,4,18],
  ["Rb",37,5,1],["Sr",38,5,2],["Y",39,5,3],["Zr",40,5,4],["Nb",41,5,5],["Mo",42,5,6],["Tc",43,5,7],["Ru",44,5,8],["Rh",45,5,9],["Pd",46,5,10],["Ag",47,5,11],["Cd",48,5,12],
  ["In",49,5,13],["Sn",50,5,14],["Sb",51,5,15],["Te",52,5,16],["I",53,5,17],["Xe",54,5,18],
  ["Cs",55,6,1],["Ba",56,6,2],
  ["La",57,6,3],["Hf",72,6,4],["Ta",73,6,5],["W",74,6,6],["Re",75,6,7],["Os",76,6,8],["Ir",77,6,9],["Pt",78,6,10],["Au",79,6,11],["Hg",80,6,12],
  ["Tl",81,6,13],["Pb",82,6,14],["Bi",83,6,15],["Po",84,6,16],["At",85,6,17],["Rn",86,6,18],
  ["Fr",87,7,1],["Ra",88,7,2],
  ["Ac",89,7,3],["Rf",104,7,4],["Db",105,7,5],["Sg",106,7,6],["Bh",107,7,7],["Hs",108,7,8],["Mt",109,7,9],["Ds",110,7,10],["Rg",111,7,11],["Cn",112,7,12],
  ["Nh",113,7,13],["Fl",114,7,14],["Mc",115,7,15],["Lv",116,7,16],["Ts",117,7,17],["Og",118,7,18],
];

// Lantanídeos e actinídeos (linhas separadas) — para mostrar sem bagunçar o grid principal
const LANTH = [
  ["Ce",58],["Pr",59],["Nd",60],["Pm",61],["Sm",62],["Eu",63],["Gd",64],["Tb",65],["Dy",66],["Ho",67],["Er",68],["Tm",69],["Yb",70],["Lu",71]
];
const ACT = [
  ["Th",90],["Pa",91],["U",92],["Np",93],["Pu",94],["Am",95],["Cm",96],["Bk",97],["Cf",98],["Es",99],["Fm",100],["Md",101],["No",102],["Lr",103]
];

const ALLSYMS = new Set([...ELEMENTS.map(x=>x[0]), ...LANTH.map(x=>x[0]), ...ACT.map(x=>x[0])]);

/* =========================================================
   2) Dados editáveis: CORRIDOR (entrada) e DERIVED (cálculo)
   ========================================================= */
const CORRIDOR = {};  // {sym:{delta,v,note,_src}}
const DERIVED  = {};  // {sym:{R,sin2,dmix,delta,v}}

let SELECTED = null;

/* =========================================================
   3) Fórmulas do seu artigo (modelo 2×2)
   ========================================================= */
function computeDerived(delta, v){
  const D = Number(delta);
  const V = Number(v);
  if (!isFinite(D) || !isFinite(V) || Math.abs(V) === 0) return null;

  const dmix = Math.sqrt(D*D + 4*V*V);                       // Eq. adiabatic_gap_student
  const R    = Math.abs(D) / Math.abs(V);                   // Eq. R_and_leakage
  const sin2 = 0.5 * (1 - (D / dmix));                      // Eq. mixing_angle_student  (sin^2 φ)
  return { delta:D, v:V, dmix, R, sin2 };
}

/* =========================================================
   4) Prior heurístico (só para preencher algo coerente)
      — NÃO substitui espectroscopia / downfolding.
   ========================================================= */
function blockOf(sym){
  // classificação grosseira por posição e "família"
  const Z = ZMAP[sym];
  if (!Z) return "x";
  if (Z >= 57 && Z <= 71) return "f";
  if (Z >= 89 && Z <= 103) return "f";
  const e = META[sym];
  if (!e) return "x";
  const g = e.group;
  if (g >= 3 && g <= 12) return "d";
  if (g >= 13 && g <= 18) return "p";
  if (g <= 2) return "s";
  return "x";
}

// Heurística simples em eV: f mais denso → Δ menor, V moderado;
// 5d e pesados → V maior (SOC/covalência); SC em 3d intermediário.
function estimateDeltaV(sym){
  const Z = ZMAP[sym] || 0;
  const b = blockOf(sym);
  const e = META[sym];
  const period = e?.period ?? 0;
  const group  = e?.group ?? 0;

  let delta, v, note;

  if (b === "s"){
    delta = 1.2 + 0.15*period;           // grandes separações típicas
    v     = 0.10 + 0.02*period;
    note  = "s (estável)";
  } else if (b === "p"){
    delta = 0.9 + 0.10*period;
    v     = 0.12 + 0.03*period;
    if (Z >= 49) { v += 0.08; note = "p + SOC (pesado)"; }
    else note = "p";
  } else if (b === "d"){
    // d: mais perto do corredor, especialmente tardios (s↔d e d↔d)
    const late = (group >= 9 && group <= 12);
    delta = late ? (0.35 + 0.05*(6-period)) : (0.60 + 0.06*(6-period));
    v     = late ? (0.22 + 0.03*period) : (0.18 + 0.02*period);
    note  = late ? "d (tardio): s↔d/d↔d" : "d: d↔d";
    if (period >= 6) { v += 0.12; note += " + SOC"; } // 5d
    // SC "provável" (regra tosca): Fe/Co/Ni/ Mn região
    if (["Mn","Fe","Co","Ni"].includes(sym)) note += " + SC?";
  } else if (b === "f"){
    delta = 0.20 + 0.02*period;          // multipletos densos
    v     = 0.18 + 0.03*period;          // mistura efetiva relevante
    note  = "f/rel (multipletos densos)";
  } else {
    delta = 0.8; v = 0.15; note = "prior";
  }

  // clamp razoável (só para não gerar lixo)
  delta = Math.max(0.05, Math.min(delta, 2.50));
  v     = Math.max(0.03, Math.min(v,     0.80));

  return { delta: Number(delta.toFixed(3)), v: Number(v.toFixed(3)), note };
}

/* =========================================================
   5) Montagem do grid + interação
   ========================================================= */
const META = {};   // sym -> {Z,period,group}
const ZMAP = {};   // sym -> Z
for (const [sym,Z,period,group] of ELEMENTS){
  META[sym] = {Z,period,group};
  ZMAP[sym] = Z;
}
for (const [sym,Z] of LANTH){ META[sym] = {Z,period:6,group:3}; ZMAP[sym]=Z; }
for (const [sym,Z] of ACT){   META[sym] = {Z,period:7,group:3}; ZMAP[sym]=Z; }

function makeGrid(){
  const pt = document.getElementById("pt");
  pt.innerHTML = "";

  // grid 7 períodos x 18 grupos, mas usando CSS grid por 18 colunas; colocamos por ordem
  // Para deixar bonitinho: inserimos "buracos" usando elementos vazios
  const byPos = new Map();
  for (const [sym,Z,period,group] of ELEMENTS){
    byPos.set(`${period}-${group}`, sym);
  }
  for (let period=1; period<=7; period++){
    for (let group=1; group<=18; group++){
      const sym = byPos.get(`${period}-${group}`);
      if (!sym){
        const ghost = document.createElement("div");
        ghost.style.height = "46px";
        ghost.style.border = "1px dashed rgba(255,255,255,.05)";
        ghost.style.borderRadius = "10px";
        ghost.style.background = "transparent";
        pt.appendChild(ghost);
        continue;
      }
      pt.appendChild(makeCell(sym));
    }
  }

  // adicionar uma “linha” visual para Ln/An (simples)
  const spacer = document.createElement("div");
  spacer.style.gridColumn = "1 / span 18";
  spacer.style.height = "8px";
  pt.appendChild(spacer);

  const lnLabel = document.createElement("div");
  lnLabel.textContent = "Ln";
  lnLabel.className = "cell";
  lnLabel.style.cursor = "default";
  lnLabel.innerHTML = `<div class="sym">Ln</div><div class="mini">57–71</div>`;
  pt.appendChild(lnLabel);

  for (const [sym] of LANTH) pt.appendChild(makeCell(sym));

  const anLabel = document.createElement("div");
  anLabel.textContent = "An";
  anLabel.className = "cell";
  anLabel.style.cursor = "default";
  anLabel.innerHTML = `<div class="sym">An</div><div class="mini">89–103</div>`;
  pt.appendChild(anLabel);

  for (const [sym] of ACT) pt.appendChild(makeCell(sym));

  refreshCellClasses();
  updateStatline();
}

function makeCell(sym){
  const el = document.createElement("div");
  el.className = "cell";
  el.dataset.sym = sym;
  el.innerHTML = `<div class="sym">${sym}</div><div class="mini mono" id="mini-${sym}">Δ,V: —</div>`;
  el.addEventListener("click", ()=> select(sym));
  return el;
}

function refreshCellClasses(){
  document.querySelectorAll(".cell").forEach(el=>{
    const sym = el.dataset.sym;
    if (!sym || !ALLSYMS.has(sym)) return;

    el.classList.remove("filled","r_ok","r_mid","r_bad");

    const inp = CORRIDOR[sym];
    const der = DERIVED[sym];

    const has = der && isFinite(der.delta) && isFinite(der.v) && Math.abs(der.v)>0;
    if (has) el.classList.add("filled");

    if (der && isFinite(der.R)){
      if (der.R >= 3) el.classList.add("r_ok");
      else if (der.R >= 1) el.classList.add("r_mid");
      else el.classList.add("r_bad");
    }

    const mini = document.getElementById(`mini-${sym}`);
    if (mini){
      if (has){
        mini.textContent = `Δ=${der.delta.toFixed(2)} V=${der.v.toFixed(2)} | R=${der.R.toFixed(2)}`;
      } else {
        mini.textContent = "Δ,V: —";
      }
    }
  });
}

function recomputeAllDerived(){
  for (const sym of ALLSYMS){
    const inp = CORRIDOR[sym];
    if (!inp){ delete DERIVED[sym]; continue; }
    const d = computeDerived(inp.delta, inp.v);
    if (!d){ delete DERIVED[sym]; continue; }
    DERIVED[sym] = d;
  }
}

function updateStatline(){
  let n=0, ok=0, mid=0, bad=0;
  for (const sym of ALLSYMS){
    const d = DERIVED[sym];
    if (!d || !isFinite(d.R)) continue;
    n++;
    if (d.R >= 3) ok++;
    else if (d.R >= 1) mid++;
    else bad++;
  }
  document.getElementById("statline").innerHTML =
    `<b>Status:</b> elementos com Δ,V válidos: <span class="mono">${n}</span> · robusto: <span class="mono">${ok}</span> · transição: <span class="mono">${mid}</span> · corredor: <span class="mono">${bad}</span>`;
}

/* =========================================================
   6) Painel lateral
   ========================================================= */
function select(sym){
  SELECTED = sym;
  document.getElementById("selTitle").textContent = `Elemento: ${sym} (Z=${ZMAP[sym] ?? "?"})`;

  const inp = CORRIDOR[sym] || {};
  document.getElementById("inpDelta").value = (inp.delta ?? "");
  document.getElementById("inpV").value     = (inp.v ?? "");
  document.getElementById("inpNote").value  = (inp.note ?? "");

  updateKPIs(sym);
}

function updateKPIs(sym){
  const d = DERIVED[sym];
  document.getElementById("kpiR").textContent     = d ? d.R.toFixed(6) : "—";
  document.getElementById("kpiSin2").textContent  = d ? d.sin2.toFixed(6) : "—";
  document.getElementById("kpiDmix").textContent  = d ? d.dmix.toFixed(6) : "—";
}

document.getElementById("btnSave").addEventListener("click", ()=>{
  if (!SELECTED) return alert("Selecione um elemento.");
  const delta = document.getElementById("inpDelta").value;
  const v     = document.getElementById("inpV").value;
  const note  = document.getElementById("inpNote").value;

  CORRIDOR[SELECTED] = {
    delta: delta === "" ? undefined : Number(delta),
    v:     v     === "" ? undefined : Number(v),
    note:  note ?? "",
    _src:  "user"
  };

  recomputeAllDerived();
  refreshCellClasses();
  updateStatline();
  updateKPIs(SELECTED);
});

document.getElementById("btnClear").addEventListener("click", ()=>{
  if (!SELECTED) return alert("Selecione um elemento.");
  delete CORRIDOR[SELECTED];
  delete DERIVED[SELECTED];
  document.getElementById("inpDelta").value = "";
  document.getElementById("inpV").value     = "";
  document.getElementById("inpNote").value  = "";
  refreshCellClasses();
  updateStatline();
  updateKPIs(SELECTED);
});

document.getElementById("btnAutofill").addEventListener("click", ()=>{
  for (const sym of ALLSYMS){
    const cur = CORRIDOR[sym] || {};
    const needDelta = !(cur.delta !== undefined && isFinite(Number(cur.delta)));
    const needV     = !(cur.v     !== undefined && isFinite(Number(cur.v)));
    if (needDelta || needV){
      const est = estimateDeltaV(sym);
      CORRIDOR[sym] = {
        ...cur,
        delta: needDelta ? est.delta : Number(cur.delta),
        v:     needV     ? est.v     : Number(cur.v),
        note:  (cur.note ?? "") || est.note,
        _src:  "auto"
      };
    } else {
      CORRIDOR[sym] = { ...cur, _src: cur._src ?? "user" };
    }
  }
  recomputeAllDerived();
  refreshCellClasses();
  updateStatline();
  if (SELECTED) updateKPIs(SELECTED);
  alert("Auto-preenchido: Δ,V para 118 (não sobrescreve o que você já digitou).");
});

/* =========================================================
   7) Boot
   ========================================================= */
makeGrid();
select("Fe"); // abre já num elemento sensível (por motivos óbvios)
</script>

</body>
</html>



