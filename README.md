<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>corridor-table</title>
  <style>
    :root { --bg:#0b0f14; --fg:#e6edf3; --muted:#9aa4af; --card:#111826; --line:#233044; --accent:#6ee7ff; --bad:#ff6b6b; --ok:#6bff95; }
    body { margin:0; font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; background: var(--bg); color: var(--fg); }
    header { padding: 18px 18px 10px; border-bottom:1px solid var(--line); }
    h1 { margin:0; font-size: 22px; letter-spacing: .3px; }
    .sub { margin-top:6px; color: var(--muted); font-size: 14px; line-height:1.35; }
    .wrap { display:grid; grid-template-columns: 1.35fr .95fr; gap: 14px; padding: 14px; }
    @media (max-width: 980px){ .wrap{ grid-template-columns:1fr; } }
    .card { background: var(--card); border:1px solid var(--line); border-radius: 14px; padding: 12px; }
    .card h2 { margin:0 0 10px; font-size: 16px; color: var(--fg); }
    .hint { color: var(--muted); font-size: 13px; margin: 0 0 10px; line-height:1.35; }
    .pt { width:100%; overflow:auto; padding-bottom:6px; }
    .grid {
      display:grid;
      grid-template-columns: repeat(18, minmax(44px, 1fr));
      gap: 6px;
      min-width: 792px; /* 18*44 */
    }
    .rowlabel {
      grid-column: 1 / span 18;
      color: var(--muted);
      font-size: 12px;
      margin: 8px 0 2px;
    }
    .el {
      user-select:none;
      cursor:pointer;
      border:1px solid var(--line);
      border-radius: 10px;
      padding: 8px 6px;
      text-align:center;
      background: rgba(255,255,255,.03);
      transition: transform .04s ease, border-color .12s ease, background .12s ease;
    }
    .el:hover { transform: translateY(-1px); border-color: rgba(110,231,255,.55); }
    .el.sel { outline: 2px solid rgba(110,231,255,.65); border-color: rgba(110,231,255,.65); }
    .el.filled { background: rgba(107,255,149,.06); }
    .el.warn { background: rgba(255,107,107,.06); }
    .sym { font-weight: 700; font-size: 14px; }
    .z { font-size: 11px; color: var(--muted); margin-top: 2px; }
    .name { font-size: 11px; color: var(--muted); margin-top: 4px; white-space: nowrap; overflow:hidden; text-overflow: ellipsis; }
    .kv { display:grid; grid-template-columns: 1fr 1fr; gap: 10px; }
    .k { color: var(--muted); font-size: 12px; }
    .v { font-size: 14px; margin-top: 2px; }
    .mono { font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; }
    details { margin-top: 10px; }
    textarea { width:100%; min-height: 160px; border-radius: 10px; border:1px solid var(--line); background:#0b1220; color:var(--fg); padding:10px; box-sizing:border-box; }
    footer { padding: 10px 14px 18px; color: var(--muted); font-size: 12px; }
    a { color: var(--accent); text-decoration: none; }
    a:hover { text-decoration: underline; }
    .pill { display:inline-block; padding:2px 8px; border:1px solid var(--line); border-radius: 999px; color: var(--muted); font-size: 12px; margin-left: 8px; }
    .btnbar { display:flex; gap:8px; flex-wrap:wrap; margin: 8px 0 0; }
    button {
      background: #0b1220; color: var(--fg);
      border: 1px solid var(--line); border-radius: 10px;
      padding: 8px 10px; cursor: pointer; font-size: 13px;
    }
    button:hover { border-color: rgba(110,231,255,.55); }
    .stat { margin-top: 8px; font-size: 12px; color: var(--muted); line-height: 1.4; }
    .good { color: var(--ok); }
    .bad { color: var(--bad); }
    code { background: rgba(255,255,255,.05); padding: 1px 6px; border-radius: 8px; }
  </style>
</head>
<body>
  <header>
    <h1>corridor-table <span class="pill">GitHub Pages</span></h1>
    <div class="sub">
      Tabela periódica interativa para índices de “corredor” (modelo 2×2). Cole CSV/JSON (um por elemento)
      e o site calcula <span class="mono">R</span>, <span class="mono">Δmix</span> e, se você não der,
      estima <span class="mono">leak = sin²φ</span>.
    </div>
  </header>

  <div class="wrap">
    <section class="card">
      <h2>Tabela periódica (118)</h2>
      <p class="hint">
        Clique em um símbolo. Elementos com dados aparecem em verde suave.
        Lantanídeos/actinídeos ficam em linhas separadas (layout clássico).
      </p>
      <div class="pt">
        <div id="pt-grid" class="grid" aria-label="Periodic table"></div>
      </div>

      <h2 style="margin-top:12px;">Colar dados (CSV ou JSON)</h2>
      <p class="hint">
        <b>CSV</b> esperado: <code>symbol,delta,v,leak,comment</code> (leak/comment opcionais). Ex.:<br/>
        <span class="mono">Pd,0.12,0.18,,promo corridor</span><br/>
        <b>JSON</b> esperado: objeto por símbolo. Ex.:<br/>
        <span class="mono">{"Pd":{"delta":0.12,"v":0.18,"comment":"..."}}</span>
      </p>
      <textarea id="in-data" class="mono" placeholder="Cole aqui CSV ou JSON..."></textarea>
      <div class="btnbar">
        <button id="btn-apply">Aplicar dados</button>
        <button id="btn-export">Exportar CORRIDOR (JSON)</button>
        <button id="btn-clear">Limpar dados</button>
        <button id="btn-demo">Inserir demo</button>
      </div>
      <div class="stat" id="statline">Nenhum dado carregado.</div>
    </section>

    <aside class="card">
      <h2>Selecione um elemento</h2>
      <p class="hint" id="sel-hint">Clique em um símbolo na tabela.</p>

      <div class="kv">
        <div>
          <div class="k">Δ (eV)</div>
          <div class="v mono" id="out-delta">—</div>
        </div>
        <div>
          <div class="k">V (eV)</div>
          <div class="v mono" id="out-v">—</div>
        </div>
        <div>
          <div class="k">R = |Δ|/|V|</div>
          <div class="v mono" id="out-r">—</div>
        </div>
        <div>
          <div class="k">Leakage = sin²φ</div>
          <div class="v mono" id="out-leak">—</div>
        </div>
      </div>

      <h2 style="margin-top:14px;">Métricas derivadas</h2>
      <div class="kv">
        <div>
          <div class="k">Regime</div>
          <div class="v" id="out-reg">—</div>
        </div>
        <div>
          <div class="k">Δmix = √(Δ²+4V²) (eV)</div>
          <div class="v mono" id="out-dmix">—</div>
        </div>
      </div>
      <div style="margin-top:10px;">
        <div class="k">Comentário</div>
        <div class="v" id="out-cmt">—</div>
      </div>

      <details>
        <summary>Ver JSON bruto do elemento</summary>
        <textarea id="raw" class="mono" readonly></textarea>
      </details>

      <details>
        <summary>Convenções (importante)</summary>
        <div class="hint" style="margin-top:8px;">
          Este painel usa o Hamiltoniano efetivo 2×2 padrão (dois caracteres diabáticos com acoplamento V).
          <span class="mono">Δmix</span> vem do gap de autovalores.
          A <span class="mono">leak</span> é tomada como a fração de mistura (sin²φ) no sentido “quanto mais perto do corredor, maior”.
          Se você já tem leak do seu pipeline (Feshbach/downfolding/fit), forneça e ele prevalece.
        </div>
      </details>
    </aside>
  </div>

  <footer>
    Site estático. Sem backend. Sem drama. (Mas com álgebra linear.)
  </footer>

<script>
/* =========================
   118 elementos com posições (grupo/período) + f-bloco
   Layout:
     - Grade principal: períodos 1..7, grupos 1..18
     - f-bloco: La..Lu e Ac..Lr em linhas separadas
   ========================= */

const MAIN = [
  // Período 1
  {Z:1,sym:"H",name:"Hydrogen",period:1,group:1},
  {Z:2,sym:"He",name:"Helium",period:1,group:18},
  // Período 2
  {Z:3,sym:"Li",name:"Lithium",period:2,group:1},
  {Z:4,sym:"Be",name:"Beryllium",period:2,group:2},
  {Z:5,sym:"B",name:"Boron",period:2,group:13},
  {Z:6,sym:"C",name:"Carbon",period:2,group:14},
  {Z:7,sym:"N",name:"Nitrogen",period:2,group:15},
  {Z:8,sym:"O",name:"Oxygen",period:2,group:16},
  {Z:9,sym:"F",name:"Fluorine",period:2,group:17},
  {Z:10,sym:"Ne",name:"Neon",period:2,group:18},
  // Período 3
  {Z:11,sym:"Na",name:"Sodium",period:3,group:1},
  {Z:12,sym:"Mg",name:"Magnesium",period:3,group:2},
  {Z:13,sym:"Al",name:"Aluminum",period:3,group:13},
  {Z:14,sym:"Si",name:"Silicon",period:3,group:14},
  {Z:15,sym:"P",name:"Phosphorus",period:3,group:15},
  {Z:16,sym:"S",name:"Sulfur",period:3,group:16},
  {Z:17,sym:"Cl",name:"Chlorine",period:3,group:17},
  {Z:18,sym:"Ar",name:"Argon",period:3,group:18},
  // Período 4
  {Z:19,sym:"K",name:"Potassium",period:4,group:1},
  {Z:20,sym:"Ca",name:"Calcium",period:4,group:2},
  {Z:21,sym:"Sc",name:"Scandium",period:4,group:3},
  {Z:22,sym:"Ti",name:"Titanium",period:4,group:4},
  {Z:23,sym:"V",name:"Vanadium",period:4,group:5},
  {Z:24,sym:"Cr",name:"Chromium",period:4,group:6},
  {Z:25,sym:"Mn",name:"Manganese",period:4,group:7},
  {Z:26,sym:"Fe",name:"Iron",period:4,group:8},
  {Z:27,sym:"Co",name:"Cobalt",period:4,group:9},
  {Z:28,sym:"Ni",name:"Nickel",period:4,group:10},
  {Z:29,sym:"Cu",name:"Copper",period:4,group:11},
  {Z:30,sym:"Zn",name:"Zinc",period:4,group:12},
  {Z:31,sym:"Ga",name:"Gallium",period:4,group:13},
  {Z:32,sym:"Ge",name:"Germanium",period:4,group:14},
  {Z:33,sym:"As",name:"Arsenic",period:4,group:15},
  {Z:34,sym:"Se",name:"Selenium",period:4,group:16},
  {Z:35,sym:"Br",name:"Bromine",period:4,group:17},
  {Z:36,sym:"Kr",name:"Krypton",period:4,group:18},
  // Período 5
  {Z:37,sym:"Rb",name:"Rubidium",period:5,group:1},
  {Z:38,sym:"Sr",name:"Strontium",period:5,group:2},
  {Z:39,sym:"Y",name:"Yttrium",period:5,group:3},
  {Z:40,sym:"Zr",name:"Zirconium",period:5,group:4},
  {Z:41,sym:"Nb",name:"Niobium",period:5,group:5},
  {Z:42,sym:"Mo",name:"Molybdenum",period:5,group:6},
  {Z:43,sym:"Tc",name:"Technetium",period:5,group:7},
  {Z:44,sym:"Ru",name:"Ruthenium",period:5,group:8},
  {Z:45,sym:"Rh",name:"Rhodium",period:5,group:9},
  {Z:46,sym:"Pd",name:"Palladium",period:5,group:10},
  {Z:47,sym:"Ag",name:"Silver",period:5,group:11},
  {Z:48,sym:"Cd",name:"Cadmium",period:5,group:12},
  {Z:49,sym:"In",name:"Indium",period:5,group:13},
  {Z:50,sym:"Sn",name:"Tin",period:5,group:14},
  {Z:51,sym:"Sb",name:"Antimony",period:5,group:15},
  {Z:52,sym:"Te",name:"Tellurium",period:5,group:16},
  {Z:53,sym:"I",name:"Iodine",period:5,group:17},
  {Z:54,sym:"Xe",name:"Xenon",period:5,group:18},
  // Período 6 (grupo 3 vira “La*” na tabela principal; f-bloco separado)
  {Z:55,sym:"Cs",name:"Cesium",period:6,group:1},
  {Z:56,sym:"Ba",name:"Barium",period:6,group:2},
  {Z:57,sym:"La",name:"Lanthanum",period:6,group:3},
  {Z:72,sym:"Hf",name:"Hafnium",period:6,group:4},
  {Z:73,sym:"Ta",name:"Tantalum",period:6,group:5},
  {Z:74,sym:"W",name:"Tungsten",period:6,group:6},
  {Z:75,sym:"Re",name:"Rhenium",period:6,group:7},
  {Z:76,sym:"Os",name:"Osmium",period:6,group:8},
  {Z:77,sym:"Ir",name:"Iridium",period:6,group:9},
  {Z:78,sym:"Pt",name:"Platinum",period:6,group:10},
  {Z:79,sym:"Au",name:"Gold",period:6,group:11},
  {Z:80,sym:"Hg",name:"Mercury",period:6,group:12},
  {Z:81,sym:"Tl",name:"Thallium",period:6,group:13},
  {Z:82,sym:"Pb",name:"Lead",period:6,group:14},
  {Z:83,sym:"Bi",name:"Bismuth",period:6,group:15},
  {Z:84,sym:"Po",name:"Polonium",period:6,group:16},
  {Z:85,sym:"At",name:"Astatine",period:6,group:17},
  {Z:86,sym:"Rn",name:"Radon",period:6,group:18},
  // Período 7 (grupo 3 vira “Ac*” na tabela principal; f-bloco separado)
  {Z:87,sym:"Fr",name:"Francium",period:7,group:1},
  {Z:88,sym:"Ra",name:"Radium",period:7,group:2},
  {Z:89,sym:"Ac",name:"Actinium",period:7,group:3},
  {Z:104,sym:"Rf",name:"Rutherfordium",period:7,group:4},
  {Z:105,sym:"Db",name:"Dubnium",period:7,group:5},
  {Z:106,sym:"Sg",name:"Seaborgium",period:7,group:6},
  {Z:107,sym:"Bh",name:"Bohrium",period:7,group:7},
  {Z:108,sym:"Hs",name:"Hassium",period:7,group:8},
  {Z:109,sym:"Mt",name:"Meitnerium",period:7,group:9},
  {Z:110,sym:"Ds",name:"Darmstadtium",period:7,group:10},
  {Z:111,sym:"Rg",name:"Roentgenium",period:7,group:11},
  {Z:112,sym:"Cn",name:"Copernicium",period:7,group:12},
  {Z:113,sym:"Nh",name:"Nihonium",period:7,group:13},
  {Z:114,sym:"Fl",name:"Flerovium",period:7,group:14},
  {Z:115,sym:"Mc",name:"Moscovium",period:7,group:15},
  {Z:116,sym:"Lv",name:"Livermorium",period:7,group:16},
  {Z:117,sym:"Ts",name:"Tennessine",period:7,group:17},
  {Z:118,sym:"Og",name:"Oganesson",period:7,group:18},
];

const LANTH = [
  {Z:58,sym:"Ce",name:"Cerium"},
  {Z:59,sym:"Pr",name:"Praseodymium"},
  {Z:60,sym:"Nd",name:"Neodymium"},
  {Z:61,sym:"Pm",name:"Promethium"},
  {Z:62,sym:"Sm",name:"Samarium"},
  {Z:63,sym:"Eu",name:"Europium"},
  {Z:64,sym:"Gd",name:"Gadolinium"},
  {Z:65,sym:"Tb",name:"Terbium"},
  {Z:66,sym:"Dy",name:"Dysprosium"},
  {Z:67,sym:"Ho",name:"Holmium"},
  {Z:68,sym:"Er",name:"Erbium"},
  {Z:69,sym:"Tm",name:"Thulium"},
  {Z:70,sym:"Yb",name:"Ytterbium"},
  {Z:71,sym:"Lu",name:"Lutetium"},
];

const ACT = [
  {Z:90,sym:"Th",name:"Thorium"},
  {Z:91,sym:"Pa",name:"Protactinium"},
  {Z:92,sym:"U",name:"Uranium"},
  {Z:93,sym:"Np",name:"Neptunium"},
  {Z:94,sym:"Pu",name:"Plutonium"},
  {Z:95,sym:"Am",name:"Americium"},
  {Z:96,sym:"Cm",name:"Curium"},
  {Z:97,sym:"Bk",name:"Berkelium"},
  {Z:98,sym:"Cf",name:"Californium"},
  {Z:99,sym:"Es",name:"Einsteinium"},
  {Z:100,sym:"Fm",name:"Fermium"},
  {Z:101,sym:"Md",name:"Mendelevium"},
  {Z:102,sym:"No",name:"Nobelium"},
  {Z:103,sym:"Lr",name:"Lawrencium"},
];

const ALLMETA = (() => {
  const m = new Map();
  for (const e of MAIN) m.set(e.sym, e);
  for (const e of LANTH) m.set(e.sym, { ...e, period: 6, group: null, block: "f", series:"Lanthanides" });
  for (const e of ACT) m.set(e.sym, { ...e, period: 7, group: null, block: "f", series:"Actinides" });
  return m;
})();

/* =========================
   Dados do corredor (você cola via UI)
   Estrutura:
     CORRIDOR["Pd"] = { delta: 0.12, v: 0.18, leak: 0.20, comment: "..." }
   ========================= */
let CORRIDOR = {};

/* =========================
   Funções: formatação e física 2×2
   ========================= */
function fmt(x, digits=3){
  if (x === null || x === undefined || Number.isNaN(x)) return "—";
  return Number(x).toFixed(digits);
}
function regimeFromR(R){
  if (!isFinite(R)) return "—";
  if (R >= 3) return "Estável (R ≫ 1)";
  if (R >= 1) return "Intermediário (R ~ 1–3)";
  return "Corredor/competição (R ≲ 1)";
}
function dmix(delta, v){
  if (!isFinite(delta) || !isFinite(v)) return NaN;
  return Math.sqrt(delta*delta + 4*v*v);
}
function leakFrom2x2(delta, v){
  // leak = sin^2 φ = 1/2 (1 - |Δ|/sqrt(Δ^2 + 4V^2))
  // “mínimo” e monotônico: Δ→∞ => 0; Δ→0 => 1/2
  if (!isFinite(delta) || !isFinite(v)) return NaN;
  const D = Math.abs(delta);
  const denom = Math.sqrt(delta*delta + 4*v*v);
  if (!(denom > 0)) return NaN;
  return 0.5*(1 - D/denom);
}

/* =========================
   Renderização da tabela
   ========================= */
const grid = document.getElementById("pt-grid");

function cell(sym, Z, name){
  const div = document.createElement("div");
  div.className = "el";
  div.dataset.sym = sym;
  div.innerHTML = `<div class="sym">${sym}</div><div class="z">${Z}</div><div class="name">${name}</div>`;
  div.addEventListener("click", () => selectElement(sym));
  return div;
}

function makeMainGrid(){
  // Map period-group -> element
  const index = new Map();
  for (const e of MAIN) index.set(`${e.period}-${e.group}`, e);

  for (let p=1; p<=7; p++){
    for (let g=1; g<=18; g++){
      const key = `${p}-${g}`;
      const e = index.get(key);
      const div = document.createElement("div");
      div.style.gridRow = p;
      div.style.gridColumn = g;

      if (!e){
        div.style.visibility = "hidden";
        div.textContent = ".";
      } else {
        const c = cell(e.sym, e.Z, e.name);
        c.style.gridRow = p;
        c.style.gridColumn = g;
        div.replaceWith?.(c); // no-op in some browsers
        grid.appendChild(c);
        continue;
      }
      grid.appendChild(div);
    }
  }
}

function addSeries(label, arr, rowStart){
  const lab = document.createElement("div");
  lab.className = "rowlabel";
  lab.textContent = label;
  lab.style.gridRow = rowStart;
  grid.appendChild(lab);

  // linha com 18 colunas: colocamos 14 células começando na coluna 3 (alinha sob grupo 3..16)
  const startCol = 3;
  for (let i=0; i<18; i++){
    const col = i+1;
    const spot = document.createElement("div");
    spot.style.gridRow = rowStart+1;
    spot.style.gridColumn = col;

    const j = col - startCol; // 0..13
    if (j >= 0 && j < arr.length){
      const e = arr[j];
      const c = cell(e.sym, e.Z, e.name);
      c.style.gridRow = rowStart+1;
      c.style.gridColumn = col;
      grid.appendChild(c);
    } else {
      spot.style.visibility = "hidden";
      spot.textContent = ".";
      grid.appendChild(spot);
    }
  }
}

function refreshCellClasses(){
  document.querySelectorAll(".el").forEach(el => {
    const sym = el.dataset.sym;
    const d = CORRIDOR[sym];
    el.classList.remove("filled","warn");
    if (!d) return;
    const hasDelta = isFinite(d.delta);
    const hasV = isFinite(d.v) && Math.abs(d.v) > 0;
    if (hasDelta && hasV) el.classList.add("filled");
    else el.classList.add("warn");
  });
}

function makeGrid(){
  grid.innerHTML = "";
  makeMainGrid();
  // adiciona 2 linhas extras (row 9..12, fora da grade 7). Em CSS grid funciona.
  addSeries("Lantanídeos (La–Lu)", LANTH, 9);
  addSeries("Actinídeos (Ac–Lr)", ACT, 11);
  refreshCellClasses();
}

/* =========================
   Seleção e painel
   ========================= */
function selectElement(sym){
  document.querySelectorAll(".el").forEach(x => x.classList.toggle("sel", x.dataset.sym === sym));

  const meta = ALLMETA.get(sym);
  const d = CORRIDOR[sym] || {};

  const delta = (d.delta !== undefined) ? Number(d.delta) : NaN;
  const v     = (d.v     !== undefined) ? Number(d.v)     : NaN;

  const R = (isFinite(delta) && isFinite(v) && Math.abs(v) > 0) ? Math.abs(delta)/Math.abs(v) : NaN;
  const dm = (isFinite(delta) && isFinite(v)) ? dmix(delta, v) : NaN;

  // leak: se não vier, estima do 2×2
  let leak = (d.leak !== undefined && d.leak !== null && d.leak !== "") ? Number(d.leak) : NaN;
  if (!isFinite(leak)) leak = leakFrom2x2(delta, v);

  document.getElementById("sel-hint").textContent =
    `${sym} — ${meta ? meta.name : ""} (Z=${meta ? meta.Z : "?"})`;

  document.getElementById("out-delta").textContent = isFinite(delta) ? fmt(delta) : "—";
  document.getElementById("out-v").textContent     = isFinite(v)     ? fmt(v)     : "—";
  document.getElementById("out-r").textContent     = isFinite(R)     ? fmt(R)     : "—";
  document.getElementById("out-leak").textContent  = isFinite(leak)  ? fmt(leak)  : "—";

  document.getElementById("out-dmix").textContent  = isFinite(dm) ? fmt(dm) : "—";
  document.getElementById("out-reg").textContent   = regimeFromR(R);
  document.getElementById("out-cmt").textContent   = d.comment || "—";

  const raw = {
    symbol: sym,
    ...(meta ? meta : {}),
    corridor: {
      delta_eV: isFinite(delta) ? delta : null,
      v_eV:     isFinite(v)     ? v     : null,
      R:        isFinite(R)     ? R     : null,
      leakage:  isFinite(leak)  ? leak  : null,
      deltaMix_eV: isFinite(dm) ? dm : null,
      regime: regimeFromR(R),
      comment: d.comment || null
    }
  };
  document.getElementById("raw").value = JSON.stringify(raw, null, 2);
}

/* =========================
   Import: CSV ou JSON
   ========================= */
function parseMaybeJSON(txt){
  const t = txt.trim();
  if (!t) return null;
  if (t.startsWith("{") || t.startsWith("[")){
    return JSON.parse(t);
  }
  return null;
}

function parseCSV(txt){
  const lines = txt.split(/\r?\n/).map(s => s.trim()).filter(Boolean);
  if (lines.length === 0) return {};
  let start = 0;
  const head = lines[0].toLowerCase();
  const hasHeader = head.includes("symbol") || head.includes("delta") || head.includes("leak");
  if (hasHeader) start = 1;

  const out = {};
  for (let i=start; i<lines.length; i++){
    const parts = lines[i].split(",").map(s => s.trim());
    if (parts.length < 3) continue;
    const symbol = parts[0];
    const delta = parts[1] !== "" ? Number(parts[1]) : NaN;
    const v     = parts[2] !== "" ? Number(parts[2]) : NaN;
    const leak  = (parts[3] !== undefined && parts[3] !== "") ? Number(parts[3]) : undefined;
    const comment = (parts[4] !== undefined && parts.slice(4).join(",").trim() !== "") ? parts.slice(4).join(",").trim() : undefined;

    if (!ALLMETA.has(symbol)) continue; // ignora lixo
    out[symbol] = { };
    if (isFinite(delta)) out[symbol].delta = delta;
    if (isFinite(v)) out[symbol].v = v;
    if (isFinite(leak)) out[symbol].leak = leak;
    if (comment !== undefined) out[symbol].comment = comment;
  }
  return out;
}

function normalizeJSON(obj){
  // aceita:
  // 1) {"Pd":{"delta":...,"v":...}}
  // 2) [{symbol:"Pd",delta:...,v:...}, ...]
  const out = {};
  if (Array.isArray(obj)){
    for (const row of obj){
      if (!row) continue;
      const symbol = row.symbol || row.sym;
      if (!symbol || !ALLMETA.has(symbol)) continue;
      const d = {};
      if (row.delta !== undefined && isFinite(Number(row.delta))) d.delta = Number(row.delta);
      if (row.v !== undefined && isFinite(Number(row.v))) d.v = Number(row.v);
      if (row.leak !== undefined && isFinite(Number(row.leak))) d.leak = Number(row.leak);
      if (row.comment !== undefined) d.comment = String(row.comment);
      out[symbol] = d;
    }
    return out;
  }
  if (obj && typeof obj === "object"){
    for (const [symbol, val] of Object.entries(obj)){
      if (!ALLMETA.has(symbol)) continue;
      const row = val || {};
      const d = {};
      if (row.delta !== undefined && isFinite(Number(row.delta))) d.delta = Number(row.delta);
      if (row.v !== undefined && isFinite(Number(row.v))) d.v = Number(row.v);
      if (row.leak !== undefined && isFinite(Number(row.leak))) d.leak = Number(row.leak);
      if (row.comment !== undefined) d.comment = String(row.comment);
      out[symbol] = d;
    }
    return out;
  }
  return {};
}

function stats(){
  const syms = Array.from(ALLMETA.keys());
  let nAny=0, nComplete=0, nBad=0;
  for (const s of syms){
    const d = CORRIDOR[s];
    if (!d) continue;
    nAny++;
    const ok = isFinite(d.delta) && isFinite(d.v) && Math.abs(d.v) > 0;
    if (ok) nComplete++; else nBad++;
  }
  return { total: syms.length, nAny, nComplete, nBad };
}

function updateStatline(){
  const { total, nAny, nComplete, nBad } = stats();
  const el = document.getElementById("statline");
  el.innerHTML =
    `Elementos: <span class="mono">${total}</span> • ` +
    `com algum dado: <span class="mono">${nAny}</span> • ` +
    `com (Δ,V) válidos: <span class="mono good">${nComplete}</span> • ` +
    `incompletos: <span class="mono bad">${nBad}</span>`;
}

/* =========================
   Botões
   ========================= */
document.getElementById("btn-apply").addEventListener("click", () => {
  const txt = document.getElementById("in-data").value;
  let obj = null;
  try { obj = parseMaybeJSON(txt); } catch(e){ obj = null; }

  let parsed = {};
  try {
    if (obj !== null) parsed = normalizeJSON(obj);
    else parsed = parseCSV(txt);
  } catch(e){
    alert("Falha ao parsear dados. Se for JSON, verifique vírgulas/aspas. Se for CSV, use vírgula como separador.");
    return;
  }

  // merge (não destrói o que já existe)
  for (const [k,v] of Object.entries(parsed)) CORRIDOR[k] = { ...(CORRIDOR[k]||{}), ...v };

  refreshCellClasses();
  updateStatline();
});

document.getElementById("btn-export").addEventListener("click", async () => {
  const txt = JSON.stringify(CORRIDOR, null, 2);
  try {
    await navigator.clipboard.writeText(txt);
    alert("COPIADO para a área de transferência (JSON do CORRIDOR).");
  } catch(e){
    // fallback: joga no textarea
    document.getElementById("in-data").value = txt;
    alert("Não consegui acessar o clipboard. Coloquei o JSON no textarea pra você copiar manualmente.");
  }
});

document.getElementById("btn-clear").addEventListener("click", () => {
  CORRIDOR = {};
  refreshCellClasses();
  updateStatline();
  document.getElementById("in-data").value = "";
  document.getElementById("raw").value = "";
  document.getElementById("sel-hint").textContent = "Clique em um símbolo na tabela.";
  ["out-delta","out-v","out-r","out-leak","out-reg","out-dmix","out-cmt"].forEach(id => {
    document.getElementById(id).textContent = "—";
  });
});

document.getElementById("btn-demo").addEventListener("click", () => {
  // demo pequeno só pra ver o mecanismo (você vai colar sua planilha real depois)
  const demo = [
    {symbol:"Pd", delta:0.12, v:0.18, comment:"demo: corredor forte"},
    {symbol:"Ni", delta:0.55, v:0.10, comment:"demo: mais estável"},
    {symbol:"Fe", delta:0.22, v:0.20, comment:"demo: intermediário"},
    {symbol:"Au", delta:0.08, v:0.12, comment:"demo: SOC/mistura provável"},
    {symbol:"U",  delta:0.10, v:0.25, comment:"demo: f-bloco brincando de mágica"},
  ];
  document.getElementById("in-data").value = JSON.stringify(demo, null, 2);
});

/* =========================
   Boot
   ========================= */
makeGrid();
updateStatline();
</script>
</body>
</html>


