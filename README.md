<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>corridor-table</title>
  <style>
    :root { --bg:#0b0f14; --fg:#e6edf3; --muted:#9aa4af; --card:#111826; --line:#233044; --accent:#6ee7ff; --bad:#ff6b6b; --ok:#6bff95; --mid:#ffd166; }
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
      min-width: 792px;
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

    /* estado de dados */
    .el.filled { background: rgba(107,255,149,.06); }
    .el.warn   { background: rgba(255,107,107,.06); }

    /* regime (derivado) */
    .el.r_ok  { box-shadow: inset 0 0 0 9999px rgba(107,255,149,.045); }
    .el.r_mid { box-shadow: inset 0 0 0 9999px rgba(255,209,102,.045); }
    .el.r_bad { box-shadow: inset 0 0 0 9999px rgba(255,107,107,.045); }

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
    .mid  { color: var(--mid); }
    .bad  { color: var(--bad); }
    code { background: rgba(255,255,255,.05); padding: 1px 6px; border-radius: 8px; }
  </style>
</head>
<body>
  <header>
    <h1>corridor-table <span class="pill">GitHub Pages</span></h1>
    <div class="sub">
      Tabela periódica interativa para índices de “corredor” (modelo 2×2).
      Cole CSV/JSON (um por elemento) e o site calcula <span class="mono">R</span>, <span class="mono">Δmix</span> e
      <span class="mono">leak = sin²φ</span>. <b>Nesta versão</b>, se você não fornecer <span class="mono">Δ</span> e <span class="mono">V</span>
      para um elemento, o site <b>auto-estima</b> um par (heurístico) e calcula tudo para os 118.
    </div>
  </header>

  <div class="wrap">
    <section class="card">
      <h2>Tabela periódica (118)</h2>
      <p class="hint">
        Clique em um símbolo. Verde = estável; amarelo = intermediário; vermelho = corredor.
        Você pode colar dados reais apenas para alguns elementos: o resto fica auto-preenchido (e pode ser sobrescrito depois).
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
        <button id="btn-apply">Aplicar (sobrescreve apenas o que você colou)</button>
        <button id="btn-autofill">Auto-preencher (118) agora</button>
        <button id="btn-export">Exportar TUDO (JSON: dados + derivados)</button>
        <button id="btn-export-raw">Exportar só entrada (JSON CORRIDOR)</button>
        <button id="btn-clear">Limpar dados</button>
        <button id="btn-demo">Inserir demo</button>
      </div>
      <div class="stat" id="statline">Nenhum dado carregado.</div>

      <details>
        <summary>Heurística de auto-estima (curta e honesta)</summary>
        <div class="hint" style="margin-top:8px;">
          Para elementos sem dados colados, usamos um gerador determinístico (por Z/bloco/período e alguns “suspeitos usuais” de quase-degenerescência em d/f).
          Isso é só um <b>prior</b> para preencher 118 casas e deixar o front-end funcionando como “painel de diagnóstico”.
          Assim que você colar valores do seu pipeline (Feshbach/downfolding/fit), eles prevalecem.
        </div>
      </details>
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
        <div class="k">Origem do Δ,V</div>
        <div class="v" id="out-src">—</div>
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
          Painel usa Hamiltoniano efetivo 2×2 (dois caracteres diabáticos com acoplamento V).
          <span class="mono">Δmix</span> é o gap de autovalores.
          <span class="mono">leak</span> é a fração de mistura <span class="mono">sin²φ</span>, monotônica com proximidade do corredor.
          Se você fornecer <span class="mono">leak</span> do seu pipeline, ele prevalece; senão estimamos via 2×2.
        </div>
      </details>
    </aside>
  </div>

  <footer>
    Site estático. Sem backend. Sem drama. (Mas com álgebra linear e 118 palpites determinísticos quando você não dá dados.)
  </footer>

<script>
/* =========================
   118 elementos com posições (grupo/período) + f-bloco
   ========================= */

const MAIN = [
  {Z:1,sym:"H",name:"Hydrogen",period:1,group:1},
  {Z:2,sym:"He",name:"Helium",period:1,group:18},
  {Z:3,sym:"Li",name:"Lithium",period:2,group:1},
  {Z:4,sym:"Be",name:"Beryllium",period:2,group:2},
  {Z:5,sym:"B",name:"Boron",period:2,group:13},
  {Z:6,sym:"C",name:"Carbon",period:2,group:14},
  {Z:7,sym:"N",name:"Nitrogen",period:2,group:15},
  {Z:8,sym:"O",name:"Oxygen",period:2,group:16},
  {Z:9,sym:"F",name:"Fluorine",period:2,group:17},
  {Z:10,sym:"Ne",name:"Neon",period:2,group:18},
  {Z:11,sym:"Na",name:"Sodium",period:3,group:1},
  {Z:12,sym:"Mg",name:"Magnesium",period:3,group:2},
  {Z:13,sym:"Al",name:"Aluminum",period:3,group:13},
  {Z:14,sym:"Si",name:"Silicon",period:3,group:14},
  {Z:15,sym:"P",name:"Phosphorus",period:3,group:15},
  {Z:16,sym:"S",name:"Sulfur",period:3,group:16},
  {Z:17,sym:"Cl",name:"Chlorine",period:3,group:17},
  {Z:18,sym:"Ar",name:"Argon",period:3,group:18},
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
   Entrada (do usuário) e derivados (sempre 118)
   ========================= */
let CORRIDOR = {};          // entrada: {delta, v, leak?, comment?, _src?}
let DERIVED = {};           // sempre completo: {delta, v, R, leak, dmix, regime, src}

/* =========================
   Física 2×2 e utilidades
   ========================= */
function fmt(x, digits=3){
  if (x === null || x === undefined || Number.isNaN(x)) return "—";
  return Number(x).toFixed(digits);
}
function regimeFromR(R){
  if (!isFinite(R)) return "—";
  if (R >= 3) return "Estável (R ≥ 3)";
  if (R >= 1) return "Intermediário (1 ≤ R < 3)";
  return "Corredor/competição (R < 1)";
}
function dmix(delta, v){
  if (!isFinite(delta) || !isFinite(v)) return NaN;
  return Math.sqrt(delta*delta + 4*v*v);
}
function leakFrom2x2(delta, v){
  // leak = sin^2 φ = 1/2 (1 - |Δ|/sqrt(Δ^2 + 4V^2))
  if (!isFinite(delta) || !isFinite(v)) return NaN;
  const D = Math.abs(delta);
  const denom = Math.sqrt(delta*delta + 4*v*v);
  if (!(denom > 0)) return NaN;
  return 0.5*(1 - D/denom);
}

/* =========================
   Heurística determinística para preencher Δ,V quando faltam
   (prior simples: bloco+período+Z e “suspeitos” d/f)
   ========================= */
const SUSPECT = new Set([
  "Cr","Cu","Nb","Mo","Ru","Rh","Pd","Ag","Pt","Au", // anomalias/competição clássica
  "Ce","Eu","Yb","Th","U","Pu"                      // f-bloco: degenerescências/valência
]);

function inferBlock(meta){
  if (meta.block === "f") return "f";
  const g = meta.group;
  if (g === null || g === undefined) return "x";
  if (g <= 2) return "s";
  if (g >= 13) return "p";
  return "d";
}

function clamp(x, lo, hi){ return Math.max(lo, Math.min(hi, x)); }

function estimateDeltaV(sym){
  const meta = ALLMETA.get(sym);
  const Z = meta?.Z ?? 0;
  const p = meta?.period ?? 1;
  const block = inferBlock(meta || {});
  // escalas “de ordem de grandeza” (eV), só para preencher painel
  const base = {
    s: { delta: 1.20, v: 0.06 },
    p: { delta: 1.00, v: 0.07 },
    d: { delta: 0.55, v: 0.14 },
    f: { delta: 0.35, v: 0.18 },
    x: { delta: 0.80, v: 0.10 }
  }[block];

  // tendência suave com Z (cresce SOC e hibridização efetiva) + período (camadas)
  const zfac = 1 + 0.0025*(Z - 1);
  const pfacD = 1 + 0.08*(p - 1);
  const pfacV = 1 + 0.04*(p - 1);

  // “corredor-prior”: suspeitos têm Δ menor e V ligeiramente maior
  const sFacD = SUSPECT.has(sym) ? 0.55 : 1.0;
  const sFacV = SUSPECT.has(sym) ? 1.25 : 1.0;

  // d10 (grupo 10-12) tende a competição s/d e reorganizações: baixa Δ
  const g = meta?.group ?? 0;
  const d10Fac = (block === "d" && g >= 10 && g <= 12) ? 0.78 : 1.0;

  // gás nobre: Δ grande, V pequeno (quase ninguém “brinca” com ele)
  const noble = new Set(["He","Ne","Ar","Kr","Xe","Rn","Og"]);
  const nobleD = noble.has(sym) ? 1.9 : 1.0;
  const nobleV = noble.has(sym) ? 0.55 : 1.0;

  // resultado
  let delta = base.delta * zfac * pfacD * sFacD * d10Fac * nobleD;
  let v     = base.v     * zfac * pfacV * sFacV * nobleV;

  // limites pragmáticos (evita lixo numérico na UI)
  delta = clamp(delta, 0.03, 3.0);
  v     = clamp(v,     0.01, 0.60);

  return { delta, v, src: "auto" };
}

/* =========================
   Construir DERIVED para TODOS os 118
   ========================= */
function recomputeAllDerived(){
  DERIVED = {};
  for (const sym of ALLMETA.keys()){
    const user = CORRIDOR[sym] || {};
    let delta = (user.delta !== undefined && isFinite(Number(user.delta))) ? Number(user.delta) : NaN;
    let v     = (user.v     !== undefined && isFinite(Number(user.v)))     ? Number(user.v)     : NaN;

    let src = "auto";
    if (isFinite(delta) && isFinite(v)) src = "user";
    else {
      const est = estimateDeltaV(sym);
      if (!isFinite(delta)) delta = est.delta;
      if (!isFinite(v))     v     = est.v;
      src = (user.delta !== undefined || user.v !== undefined) ? "misto" : "auto";
    }

    const R  = (isFinite(delta) && isFinite(v) && Math.abs(v) > 0) ? Math.abs(delta)/Math.abs(v) : NaN;
    const dm = (isFinite(delta) && isFinite(v)) ? dmix(delta, v) : NaN;

    let leak = (user.leak !== undefined && user.leak !== null && user.leak !== "" && isFinite(Number(user.leak)))
      ? Number(user.leak)
      : leakFrom2x2(delta, v);

    DERIVED[sym] = {
      delta, v, R,
      dmix: dm,
      leak,
      regime: regimeFromR(R),
      src,
      comment: (user.comment !== undefined) ? String(user.comment) : null
    };
  }
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
        grid.appendChild(div);
      } else {
        const c = cell(e.sym, e.Z, e.name);
        c.style.gridRow = p;
        c.style.gridColumn = g;
        grid.appendChild(c);
      }
    }
  }
}

function addSeries(label, arr, rowStart){
  const lab = document.createElement("div");
  lab.className = "rowlabel";
  lab.textContent = label;
  lab.style.gridRow = rowStart;
  grid.appendChild(lab);

  const startCol = 3;
  for (let i=0; i<18; i++){
    const col = i+1;
    const spot = document.createElement("div");
    spot.style.gridRow = rowStart+1;
    spot.style.gridColumn = col;

    const j = col - startCol;
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
    const inp = CORRIDOR[sym];
    const der = DERIVED[sym];

    el.classList.remove("filled","warn","r_ok","r_mid","r_bad");

    // marca “tem entrada”
    if (inp){
      const hasDelta = isFinite(inp.delta);
      const hasV = isFinite(inp.v) && Math.abs(inp.v) > 0;
      if (hasDelta && hasV) el.classList.add("filled");
      else el.classList.add("warn");
    }

    // colore por regime (sempre existe em DERIVED)
    if (der && isFinite(der.R)){
      if (der.R >= 3) el.classList.add("r_ok");
      else if (der.R >= 1) el.classList.add("r_mid");
      else el.classList.add("r_bad");
    }
  });
}

function makeGrid(){
  grid.innerHTML = "";
  makeMainGrid();
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
  const d = DERIVED[sym] || {};
  const user = CORRIDOR[sym] || {};

  const delta = (d.delta !== undefined) ? Number(d.delta) : NaN;
  const v     = (d.v     !== undefined) ? Number(d.v)     : NaN;

  const R  = (d.R    !== undefined) ? Number(d.R)    : NaN;
  const dm = (d.dmix !== undefined) ? Number(d.dmix) : NaN;
  const leak = (d.leak !== undefined) ? Number(d.leak) : NaN;

  document.getElementById("sel-hint").textContent =
    `${sym} — ${meta ? meta.name : ""} (Z=${meta ? meta.Z : "?"})`;

  document.getElementById("out-delta").textContent = isFinite(delta) ? fmt(delta) : "—";
  document.getElementById("out-v").textContent     = isFinite(v)     ? fmt(v)     : "—";
  document.getElementById("out-r").textContent     = isFinite(R)     ? fmt(R)     : "—";
  document.getElementById("out-leak").textContent  = isFinite(leak)  ? fmt(leak)  : "—";

  document.getElementById("out-dmix").textContent  = isFinite(dm) ? fmt(dm) : "—";
  document.getElementById("out-reg").textContent   = d.regime || "—";
  document.getElementById("out-cmt").textContent   = (user.comment || d.comment) || "—";
  document.getElementById("out-src").textContent   = (d.src === "user") ? "fornecido (user)"
                                            : (d.src === "misto") ? "misto (user+auto)"
                                            : "auto (heurístico)";

  const raw = {
    symbol: sym,
    ...(meta ? meta : {}),
    input: (CORRIDOR[sym] ? { ...CORRIDOR[sym] } : null),
    derived: {
      delta_eV: isFinite(delta) ? delta : null,
      v_eV:     isFinite(v)     ? v     : null,
      R:        isFinite(R)     ? R     : null,
      leakage:  isFinite(leak)  ? leak  : null,
      deltaMix_eV: isFinite(dm) ? dm : null,
      regime: d.regime || null,
      src: d.src || null,
      comment: (user.comment || d.comment) || null
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
  if (t.startsWith("{") || t.startsWith("[")) return JSON.parse(t);
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
    if (!ALLMETA.has(symbol)) continue;

    const delta = parts[1] !== "" ? Number(parts[1]) : NaN;
    const v     = parts[2] !== "" ? Number(parts[2]) : NaN;
    const leak  = (parts[3] !== undefined && parts[3] !== "") ? Number(parts[3]) : undefined;
    const comment = (parts[4] !== undefined && parts.slice(4).join(",").trim() !== "") ? parts.slice(4).join(",").trim() : undefined;

    const d = {};
    if (isFinite(delta)) d.delta = delta;
    if (isFinite(v))     d.v = v;
    if (isFinite(leak))  d.leak = leak;
    if (comment !== undefined) d.comment = comment;
    out[symbol] = d;
  }
  return out;
}

function normalizeJSON(obj){
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

/* =========================
   Estatísticas e export
   ========================= */
function stats(){
  const syms = Array.from(ALLMETA.keys());
  let nAny=0, nComplete=0, nBad=0, nAuto=0, nUser=0, nMix=0;
  let nRok=0, nRmid=0, nRbad=0;

  for (const s of syms){
    const inp = CORRIDOR[s];
    if (inp) nAny++;
    if (inp){
      const ok = isFinite(inp.delta) && isFinite(inp.v) && Math.abs(inp.v) > 0;
      if (ok) nComplete++; else nBad++;
    }

    const der = DERIVED[s];
    if (der){
      if (der.src === "auto") nAuto++;
      else if (der.src === "user") nUser++;
      else if (der.src === "misto") nMix++;

      if (isFinite(der.R)){
        if (der.R >= 3) nRok++;
        else if (der.R >= 1) nRmid++;
        else nRbad++;
      }
    }
  }
  return { total: syms.length, nAny, nComplete, nBad, nAuto, nUser, nMix, nRok, nRmid, nRbad };
}

function updateStatline(){
  const s = stats();
  const el = document.getElementById("statline");
  el.innerHTML =
    `Elementos: <span class="mono">${s.total}</span> • ` +
    `com alguma entrada: <span class="mono">${s.nAny}</span> • ` +
    `entrada com (Δ,V) válidos: <span class="mono good">${s.nComplete}</span> • ` +
    `entrada incompleta: <span class="mono bad">${s.nBad}</span><br/>` +
    `origem (Δ,V): user=<span class="mono good">${s.nUser}</span>, misto=<span class="mono mid">${s.nMix}</span>, auto=<span class="mono">${s.nAuto}</span> • ` +
    `regimes: estável=<span class="mono good">${s.nRok}</span>, interm=<span class="mono mid">${s.nRmid}</span>, corredor=<span class="mono bad">${s.nRbad}</span>`;
}

async function copyOrDump(txt){
  try {
    await navigator.clipboard.writeText(txt);
    alert("COPIADO para a área de transferência.");
  } catch(e){
    document.getElementById("in-data").value = txt;
    alert("Sem acesso ao clipboard. Coloquei no textarea para copiar manualmente.");
  }
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
    alert("Falha ao parsear dados. JSON: verifique aspas/vírgulas. CSV: separador é vírgula.");
    return;
  }

  // merge: só toca nos símbolos colados
  for (const [k,v] of Object.entries(parsed)){
    CORRIDOR[k] = { ...(CORRIDOR[k]||{}), ...v };
  }

  recomputeAllDerived();
  refreshCellClasses();
  updateStatline();
});

document.getElementById("btn-autofill").addEventListener("click", () => {
  // garante que DERIVED tem 118 (sempre) — e opcionalmente grava em CORRIDOR os auto-estimados faltantes
  for (const sym of ALLMETA.keys()){
    if (!CORRIDOR[sym]) CORRIDOR[sym] = {}; // cria “slot” vazio para existir na entrada, se quiser
    // não escreve delta/v aqui: mantém como “sem entrada” (para você distinguir), mas DERIVED sempre terá.
  }
  recomputeAllDerived();
  refreshCellClasses();
  updateStatline();
  alert("Auto-estima ativada: todos os 118 têm (Δ,V,R,Δmix,leak) em DERIVED. Cole seus dados reais quando quiser sobrescrever.");
});

document.getElementById("btn-export").addEventListener("click", async () => {
  const payload = {
    version: "corridor-table+derived.v1",
    timestamp_iso: new Date().toISOString(),
    notes: "export contém input (CORRIDOR) e derivados (DERIVED) para os 118.",
    input_CORRIDOR: CORRIDOR,
    derived_ALL: DERIVED
  };
  await copyOrDump(JSON.stringify(payload, null, 2));
});

document.getElementById("btn-export-raw").addEventListener("click", async () => {
  await copyOrDump(JSON.stringify(CORRIDOR, null, 2));
});

document.getElementById("btn-clear").addEventListener("click", () => {
  CORRIDOR = {};
  recomputeAllDerived();
  refreshCellClasses();
  updateStatline();
  document.getElementById("in-data").value = "";
  document.getElementById("raw").value = "";
  document.getElementById("sel-hint").textContent = "Clique em um símbolo na tabela.";
  ["out-delta","out-v","out-r","out-leak","out-reg","out-dmix","out-cmt","out-src"].forEach(id => {
    document.getElementById(id).textContent = "—";
  });
});

document.getElementById("btn-demo").addEventListener("click", () => {
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
recomputeAllDerived();
makeGrid();
updateStatline();
</script>
</body>
</html>



