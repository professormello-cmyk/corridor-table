<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>corridor-table</title>
  <style>
    :root { --bg:#0b0f14; --fg:#e6edf3; --muted:#9aa4af; --card:#111826; --line:#233044; --accent:#6ee7ff; }
    body { margin:0; font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; background: var(--bg); color: var(--fg); }
    header { padding: 18px 18px 8px; border-bottom:1px solid var(--line); }
    h1 { margin:0; font-size: 22px; letter-spacing: .3px; }
    .sub { margin-top:6px; color: var(--muted); font-size: 14px; }
    .wrap { display:grid; grid-template-columns: 1.35fr .95fr; gap: 14px; padding: 14px; }
    @media (max-width: 980px){ .wrap{ grid-template-columns:1fr; } }
    .card { background: var(--card); border:1px solid var(--line); border-radius: 14px; padding: 12px; }
    .card h2 { margin:0 0 10px; font-size: 16px; color: var(--fg); }
    .hint { color: var(--muted); font-size: 13px; margin: 0 0 10px; }
    .pt { width:100%; overflow:auto; }
    .grid {
      display:grid;
      grid-template-columns: repeat(18, minmax(44px, 1fr));
      gap: 6px;
      min-width: 18*44px;
    }
    .el {
      user-select:none;
      cursor:pointer;
      border:1px solid var(--line);
      border-radius: 10px;
      padding: 8px 6px;
      text-align:center;
      background: rgba(255,255,255,.03);
      transition: transform .04s ease, border-color .12s ease;
    }
    .el:hover { transform: translateY(-1px); border-color: rgba(110,231,255,.55); }
    .el.sel { outline: 2px solid rgba(110,231,255,.65); border-color: rgba(110,231,255,.65); }
    .sym { font-weight: 700; font-size: 14px; }
    .z { font-size: 11px; color: var(--muted); margin-top: 2px; }
    .name { font-size: 11px; color: var(--muted); margin-top: 4px; white-space: nowrap; overflow:hidden; text-overflow: ellipsis; }
    .kv { display:grid; grid-template-columns: 1fr 1fr; gap: 10px; }
    .k { color: var(--muted); font-size: 12px; }
    .v { font-size: 14px; margin-top: 2px; }
    .mono { font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; }
    details { margin-top: 10px; }
    textarea { width:100%; min-height: 150px; border-radius: 10px; border:1px solid var(--line); background:#0b1220; color:var(--fg); padding:10px; }
    footer { padding: 10px 14px 18px; color: var(--muted); font-size: 12px; }
    a { color: var(--accent); text-decoration: none; }
    a:hover { text-decoration: underline; }
    .pill { display:inline-block; padding:2px 8px; border:1px solid var(--line); border-radius: 999px; color: var(--muted); font-size: 12px; margin-left: 8px; }
  </style>
</head>
<body>
  <header>
    <h1>corridor-table <span class="pill">GitHub Pages</span></h1>
    <div class="sub">Tabela periódica interativa: clique no elemento para ver índices de corredor (Δ, V, R, leakage) e métricas derivadas.</div>
  </header>

  <div class="wrap">
    <section class="card">
      <h2>Tabela periódica</h2>
      <p class="hint">Clique em um símbolo. (Por enquanto os valores são placeholders; depois você cola os dados do seu artigo.)</p>
      <div class="pt">
        <div id="pt-grid" class="grid" aria-label="Periodic table"></div>
      </div>
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
          <div class="k">Δmix ≈ 2|V| (eV)</div>
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
    </aside>
  </div>

  <footer>
    GitHub Pages (site estático). Sem backend. Sem drama.
  </footer>

<script>
/*
  Layout (grupo, período) -> colunas (1..18). Se faltar algum, você me xinga depois.
  Isso desenha a Tabela Periódica como grade 18 colunas.
*/
const ELEMENTS = [
  // period 1
  {Z:1,  sym:"H",  name:"Hydrogen",    period:1, group:1},
  {Z:2,  sym:"He", name:"Helium",      period:1, group:18},
  // period 2
  {Z:3,  sym:"Li", name:"Lithium",     period:2, group:1},
  {Z:4,  sym:"Be", name:"Beryllium",   period:2, group:2},
  {Z:5,  sym:"B",  name:"Boron",       period:2, group:13},
  {Z:6,  sym:"C",  name:"Carbon",      period:2, group:14},
  {Z:7,  sym:"N",  name:"Nitrogen",    period:2, group:15},
  {Z:8,  sym:"O",  name:"Oxygen",      period:2, group:16},
  {Z:9,  sym:"F",  name:"Fluorine",    period:2, group:17},
  {Z:10, sym:"Ne", name:"Neon",        period:2, group:18},
  // period 3
  {Z:11, sym:"Na", name:"Sodium",      period:3, group:1},
  {Z:12, sym:"Mg", name:"Magnesium",   period:3, group:2},
  {Z:13, sym:"Al", name:"Aluminum",    period:3, group:13},
  {Z:14, sym:"Si", name:"Silicon",     period:3, group:14},
  {Z:15, sym:"P",  name:"Phosphorus",  period:3, group:15},
  {Z:16, sym:"S",  name:"Sulfur",      period:3, group:16},
  {Z:17, sym:"Cl", name:"Chlorine",    period:3, group:17},
  {Z:18, sym:"Ar", name:"Argon",       period:3, group:18},
  // period 4
  {Z:19, sym:"K",  name:"Potassium",   period:4, group:1},
  {Z:20, sym:"Ca", name:"Calcium",     period:4, group:2},
  {Z:21, sym:"Sc", name:"Scandium",    period:4, group:3},
  {Z:22, sym:"Ti", name:"Titanium",    period:4, group:4},
  {Z:23, sym:"V",  name:"Vanadium",    period:4, group:5},
  {Z:24, sym:"Cr", name:"Chromium",    period:4, group:6},
  {Z:25, sym:"Mn", name:"Manganese",   period:4, group:7},
  {Z:26, sym:"Fe", name:"Iron",        period:4, group:8},
  {Z:27, sym:"Co", name:"Cobalt",      period:4, group:9},
  {Z:28, sym:"Ni", name:"Nickel",      period:4, group:10},
  {Z:29, sym:"Cu", name:"Copper",      period:4, group:11},
  {Z:30, sym:"Zn", name:"Zinc",        period:4, group:12},
  {Z:31, sym:"Ga", name:"Gallium",     period:4, group:13},
  {Z:32, sym:"Ge", name:"Germanium",   period:4, group:14},
  {Z:33, sym:"As", name:"Arsenic",     period:4, group:15},
  {Z:34, sym:"Se", name:"Selenium",    period:4, group:16},
  {Z:35, sym:"Br", name:"Bromine",     period:4, group:17},
  {Z:36, sym:"Kr", name:"Krypton",     period:4, group:18},
  // period 5
  {Z:37, sym:"Rb", name:"Rubidium",    period:5, group:1},
  {Z:38, sym:"Sr", name:"Strontium",   period:5, group:2},
  {Z:39, sym:"Y",  name:"Yttrium",     period:5, group:3},
  {Z:40, sym:"Zr", name:"Zirconium",   period:5, group:4},
  {Z:41, sym:"Nb", name:"Niobium",     period:5, group:5},
  {Z:42, sym:"Mo", name:"Molybdenum",  period:5, group:6},
  {Z:43, sym:"Tc", name:"Technetium",  period:5, group:7},
  {Z:44, sym:"Ru", name:"Ruthenium",   period:5, group:8},
  {Z:45, sym:"Rh", name:"Rhodium",     period:5, group:9},
  {Z:46, sym:"Pd", name:"Palladium",   period:5, group:10},
  {Z:47, sym:"Ag", name:"Silver",      period:5, group:11},
  {Z:48, sym:"Cd", name:"Cadmium",     period:5, group:12},
  {Z:49, sym:"In", name:"Indium",      period:5, group:13},
  {Z:50, sym:"Sn", name:"Tin",         period:5, group:14},
  {Z:51, sym:"Sb", name:"Antimony",    period:5, group:15},
  {Z:52, sym:"Te", name:"Tellurium",   period:5, group:16},
  {Z:53, sym:"I",  name:"Iodine",      period:5, group:17},
  {Z:54, sym:"Xe", name:"Xenon",       period:5, group:18},
  // period 6 (lanthanoids ficam “fora” no layout clássico; aqui colocamos La/… no grupo 3 como placeholder)
  {Z:55, sym:"Cs", name:"Cesium",      period:6, group:1},
  {Z:56, sym:"Ba", name:"Barium",      period:6, group:2},
  {Z:57, sym:"La", name:"Lanthanum",   period:6, group:3},
  {Z:72, sym:"Hf", name:"Hafnium",     period:6, group:4},
  {Z:73, sym:"Ta", name:"Tantalum",    period:6, group:5},
  {Z:74, sym:"W",  name:"Tungsten",    period:6, group:6},
  {Z:75, sym:"Re", name:"Rhenium",     period:6, group:7},
  {Z:76, sym:"Os", name:"Osmium",      period:6, group:8},
  {Z:77, sym:"Ir", name:"Iridium",     period:6, group:9},
  {Z:78, sym:"Pt", name:"Platinum",    period:6, group:10},
  {Z:79, sym:"Au", name:"Gold",        period:6, group:11},
  {Z:80, sym:"Hg", name:"Mercury",     period:6, group:12},
  {Z:81, sym:"Tl", name:"Thallium",    period:6, group:13},
  {Z:82, sym:"Pb", name:"Lead",        period:6, group:14},
  {Z:83, sym:"Bi", name:"Bismuth",     period:6, group:15},
  {Z:84, sym:"Po", name:"Polonium",    period:6, group:16},
  {Z:85, sym:"At", name:"Astatine",    period:6, group:17},
  {Z:86, sym:"Rn", name:"Radon",       period:6, group:18},
  // period 7 (actinoids idem: Ac no grupo 3 placeholder)
  {Z:87, sym:"Fr", name:"Francium",    period:7, group:1},
  {Z:88, sym:"Ra", name:"Radium",      period:7, group:2},
  {Z:89, sym:"Ac", name:"Actinium",    period:7, group:3},
  {Z:104,sym:"Rf", name:"Rutherfordium", period:7, group:4},
  {Z:105,sym:"Db", name:"Dubnium",       period:7, group:5},
  {Z:106,sym:"Sg", name:"Seaborgium",    period:7, group:6},
  {Z:107,sym:"Bh", name:"Bohrium",       period:7, group:7},
  {Z:108,sym:"Hs", name:"Hassium",       period:7, group:8},
  {Z:109,sym:"Mt", name:"Meitnerium",    period:7, group:9},
  {Z:110,sym:"Ds", name:"Darmstadtium",  period:7, group:10},
  {Z:111,sym:"Rg", name:"Roentgenium",   period:7, group:11},
  {Z:112,sym:"Cn", name:"Copernicium",   period:7, group:12},
  {Z:113,sym:"Nh", name:"Nihonium",      period:7, group:13},
  {Z:114,sym:"Fl", name:"Flerovium",     period:7, group:14},
  {Z:115,sym:"Mc", name:"Moscovium",     period:7, group:15},
  {Z:116,sym:"Lv", name:"Livermorium",   period:7, group:16},
  {Z:117,sym:"Ts", name:"Tennessine",    period:7, group:17},
  {Z:118,sym:"Og", name:"Oganesson",     period:7, group:18},
];

/*
  Aqui entram os seus dados do ARTIGO.
  Você só precisa preencher (quando quiser) por símbolo:
  Δ (eV), V (eV), leakage (0..1), e um comentário.
*/
const CORRIDOR = {
  // exemplo:
  // "Pd": { delta: 0.12, v: 0.18, leak: 0.20, comment: "placeholder; preencher do artigo" },
};

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

const grid = document.getElementById("pt-grid");

function makeGrid(){
  // cria células vazias 7 períodos x 18 grupos
  const index = new Map();
  for (const e of ELEMENTS) index.set(`${e.period}-${e.group}`, e);

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
        div.className = "el";
        div.dataset.sym = e.sym;
        div.innerHTML = `<div class="sym">${e.sym}</div><div class="z">${e.Z}</div><div class="name">${e.name}</div>`;
        div.addEventListener("click", () => selectElement(e.sym));
      }
      grid.appendChild(div);
    }
  }
}

function selectElement(sym){
  document.querySelectorAll(".el").forEach(x => x.classList.toggle("sel", x.dataset.sym === sym));

  const meta = ELEMENTS.find(e => e.sym === sym);
  const d = CORRIDOR[sym] || {};

  const delta = d.delta;
  const v = d.v;
  const R = (isFinite(delta) && isFinite(v) && Math.abs(v) > 0) ? Math.abs(delta)/Math.abs(v) : NaN;
  const leak = d.leak;

  document.getElementById("sel-hint").textContent = `${sym} — ${meta ? meta.name : ""} (Z=${meta ? meta.Z : "?"})`;
  document.getElementById("out-delta").textContent = fmt(delta);
  document.getElementById("out-v").textContent = fmt(v);
  document.getElementById("out-r").textContent = isFinite(R) ? fmt(R) : "—";
  document.getElementById("out-leak").textContent = fmt(leak);

  const dmix = isFinite(v) ? 2*Math.abs(v) : NaN;
  document.getElementById("out-dmix").textContent = isFinite(dmix) ? fmt(dmix) : "—";
  document.getElementById("out-reg").textContent = regimeFromR(R);
  document.getElementById("out-cmt").textContent = d.comment || "—";

  const raw = {
    symbol: sym,
    ...meta,
    corridor: {
      delta_eV: delta ?? null,
      v_eV: v ?? null,
      R: isFinite(R) ? R : null,
      leakage: leak ?? null,
      deltaMixApprox_eV: isFinite(dmix) ? dmix : null,
      regime: regimeFromR(R),
      comment: d.comment || null
    }
  };
  document.getElementById("raw").value = JSON.stringify(raw, null, 2);
}

makeGrid();
</script>
</body>
</html>

