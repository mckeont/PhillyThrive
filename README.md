# PhillyThrive
Testing page


<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>EJ Connections — Indicators Edition</title>
<meta name="description" content="A Connections-style word grouping game using Environmental Justice indicators." />
<style>
  :root{
    --bg:#0b1220; --panel:#101a2e; --muted:#8aa1c1; --text:#e8eefb; --good:#2ecc71; --bad:#ff6b6b;
    --catA:#3b82f6; --catB:#ef4444; --catC:#22c55e; --catD:#f59e0b;
    --card:#0f1a30; --card2:#132240; --ring:#233b74;
  }
  *{box-sizing:border-box}
  body{
    margin:0; font-family:system-ui,-apple-system,Segoe UI,Roboto,Inter,Arial,sans-serif;
    background: radial-gradient(1200px 600px at 20% -10%, #142242 0%, #0b1220 60%) no-repeat,
                radial-gradient(1000px 500px at 120% 10%, #15254a 0%, #0b1220 60%) no-repeat,
                var(--bg); color:var(--text);
  }
  header{max-width:960px;margin:32px auto 16px;padding:0 16px}
  h1{margin:0 0 8px;font-weight:800;letter-spacing:.4px}
  .sub{color:var(--muted);font-size:.95rem}
  .wrap{max-width:960px;margin:0 auto 80px;padding:0 16px}
  .toolbar{
    display:flex;flex-wrap:wrap;gap:8px;align-items:center;justify-content:space-between;
    background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.06);
    padding:10px 12px;border-radius:16px;backdrop-filter:blur(4px)
  }
  .toolbar-left,.toolbar-right{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
  .chip{padding:8px 12px;border:1px solid rgba(255,255,255,.08);border-radius:999px;color:var(--muted)}
  button{
    background:#1a2a4d;border:1px solid var(--ring);color:var(--text);
    padding:10px 14px;border-radius:12px;font-weight:600;cursor:pointer;transition:.18s ease;
  }
  button:hover{transform:translateY(-1px)}
  button:disabled{opacity:.45;cursor:not-allowed;transform:none}
  .btn-ghost{background:transparent}
  .btn-primary{background:#2a47a8;border-color:#3b59c9}
  .grid{
    display:grid;grid-template-columns:repeat(4,1fr);gap:10px;margin:16px 0 10px
  }
  .card{
    user-select:none; padding:16px 12px; min-height:70px;
    background:linear-gradient(180deg,var(--card),var(--card2));
    border:1px solid var(--ring); border-radius:14px; display:flex; align-items:center; justify-content:center;
    text-align:center; font-weight:700; line-height:1.2; cursor:pointer; transition:.18s ease;
  }
  .card[aria-pressed="true"]{outline:3px solid #88a7ff; transform:translateY(-1px)}
  .card.locked{cursor:default; color:#0b1220; font-weight:800}
  .row{
    display:grid; grid-template-columns:repeat(4,1fr); gap:10px; margin:12px 0 0;
  }
  .group-banner{
    grid-column:1/-1; padding:14px 16px; border-radius:14px; font-weight:800; display:flex; justify-content:space-between; align-items:center;
  }
  .group-note{font-weight:600; opacity:.95}
  .A{background:linear-gradient(180deg, color-mix(in oklab, var(--catA) 35%, black), var(--catA));}
  .B{background:linear-gradient(180deg, color-mix(in oklab, var(--catB) 35%, black), var(--catB));}
  .C{background:linear-gradient(180deg, color-mix(in oklab, var(--catC) 35%, black), var(--catC));}
  .D{background:linear-gradient(180deg, color-mix(in oklab, var(--catD) 35%, black), var(--catD));}
  .footer{color:var(--muted);font-size:.9rem;margin-top:12px}
  .toast{
    position:fixed; left:50%; transform:translateX(-50%); bottom:20px; background:#142242; border:1px solid #2a3f77;
    padding:10px 14px; border-radius:12px; color:var(--text); box-shadow:0 10px 30px rgba(0,0,0,.3); opacity:0; pointer-events:none;
    transition:.2s ease; font-weight:700
  }
  .toast.show{opacity:1}
  @media (max-width:680px){
    .card{min-height:64px; font-size:.95rem}
  }
</style>
</head>
<body>
  <header>
    <h1>EJ Connections — Indicators Edition</h1>
    <div class="sub">Group the 16 tiles into four sets of Environmental Justice indicators. You get 4 mistakes.</div>
  </header>

  <main class="wrap" role="main">
    <div class="toolbar" aria-label="Game controls">
      <div class="toolbar-left">
        <div class="chip" id="livesChip" aria-live="polite">❤️ Lives: <strong id="lives">4</strong></div>
        <div class="chip">🎯 <span id="selectedCount">0</span>/4 selected</div>
      </div>
      <div class="toolbar-right">
        <button class="btn-ghost" id="langBtn" aria-label="Toggle language">🌐 Español</button>
        <button class="btn-ghost" id="shuffleBtn">🔀 Shuffle</button>
        <button id="submitBtn" class="btn-primary" disabled>Check Group</button>
        <button class="btn-ghost" id="resetBtn">↺ Reset</button>
        <button class="btn-ghost" id="shareBtn">🔗 Share</button>
      </div>
    </div>

    <!-- Active grid -->
    <section aria-label="Word grid">
      <div id="grid" class="grid"></div>
    </section>

    <!-- Solved groups appear here -->
    <section id="solvedArea"></section>

    <div class="footer">
      Built for education: categories include heat risk, air exposure, lead/water risk, and social vulnerability. After solving, hover/see notes for why each set fits.
    </div>
  </main>

  <div id="toast" class="toast" role="status" aria-live="polite"></div>

<script>
(() => {
  // Language packs
  const I18N = {
    en: {
      ui: {
        lives: "Lives",
        selected: "selected",
        check: "Check Group",
        reset: "Reset",
        shuffle: "Shuffle",
        share: "Share",
        es: "Español",
        en: "English",
        correct: "Correct group!",
        wrong: "Not quite — try again.",
        win: "All groups solved! 🎉"
      },
      cats: {
        A: {
          title: "Heat & Urban Form",
          note: "Indicators of heat vulnerability and urban heat island effects."
        },
        B: {
          title: "Air Pollution Exposure",
          note: "Common ambient air exposures relevant to EJ burdens."
        },
        C: {
          title: "Lead & Drinking Water Risk",
          note: "Indicators related to lead exposure pathways in housing/water."
        },
        D: {
          title: "Social Vulnerability",
          note: "Population characteristics linked to disproportionate impacts."
        }
      }
    },
    es: {
      ui: {
        lives: "Vidas",
        selected: "seleccionadas",
        check: "Verificar grupo",
        reset: "Reiniciar",
        shuffle: "Mezclar",
        share: "Compartir",
        es: "Español",
        en: "Inglés",
        correct: "¡Grupo correcto!",
        wrong: "No del todo — intenta de nuevo.",
        win: "¡Todos los grupos resueltos! 🎉"
      },
      cats: {
        A: {
          title: "Calor & Forma Urbana",
          note: "Indicadores de vulnerabilidad al calor y efectos de isla de calor."
        },
        B: {
          title: "Exposición a Aire Contaminado",
          note: "Exposiciones ambientales comunes relevantes para la justicia ambiental."
        },
        C: {
          title: "Riesgo de Plomo & Agua Potable",
          note: "Indicadores vinculados a vías de exposición al plomo en vivienda/agua."
        },
        D: {
          title: "Vulnerabilidad Social",
          note: "Características poblacionales asociadas con impactos desproporcionados."
        }
      }
    }
  };

  // Puzzle bank (you can add more puzzles later)
  const PUZZLE = {
    // Each group gets an id A-D, label will be translated
    A: { colorClass: 'A', words: ["Heat Index","Tree Canopy","Impervious Surface","Surface Temp"] },
    B: { colorClass: 'B', words: ["PM2.5","NO2","Diesel Routes","Highway Proximity"] },
    C: { colorClass: 'C', words: ["Lead Lines","Old Housing","Blood Lead","Tap Filters"] },
    D: { colorClass: 'D', words: ["Low Income","Linguistic Isolation","Age 65+","No Vehicle"] }
  };

  // --- State ---
  let lang = 'en';
  let lives = 4;
  let selected = new Set();
  let activeWords = []; // array of {text, group}
  let solvedOrder = []; // e.g., ['B','D',...]

  // Build a flat list and shuffle
  function seedWords(){
    activeWords = [];
    for(const [gid, group] of Object.entries(PUZZLE)){
      group.words.forEach(w => activeWords.push({ text: w, group: gid }));
    }
    shuffle(activeWords);
  }

  // Fisher-Yates shuffle
  function shuffle(arr){
    for(let i=arr.length-1; i>0; i--){
      const j = Math.floor(Math.random()*(i+1));
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
    return arr;
  }

  // Render functions
  const gridEl = document.getElementById('grid');
  const solvedArea = document.getElementById('solvedArea');
  const selectedCountEl = document.getElementById('selectedCount');
  const livesEl = document.getElementById('lives');
  const toastEl = document.getElementById('toast');
  const submitBtn = document.getElementById('submitBtn');
  const shuffleBtn = document.getElementById('shuffleBtn');
  const resetBtn = document.getElementById('resetBtn');
  const shareBtn = document.getElementById('shareBtn');
  const langBtn = document.getElementById('langBtn');

  function renderGrid(){
    gridEl.innerHTML = '';
    activeWords.forEach((w,i)=>{
      const btn = document.createElement('button');
      btn.className = 'card';
      btn.setAttribute('role','button');
      btn.setAttribute('aria-pressed', selected.has(i) ? 'true' : 'false');
      btn.textContent = w.text;
      btn.onclick = () => toggleSelect(i, btn);
      btn.onkeydown = (e)=>{ if(e.key==='Enter' || e.key===' '){ e.preventDefault(); btn.click(); } };
      gridEl.appendChild(btn);
    });
    selectedCountEl.textContent = selected.size;
    submitBtn.disabled = selected.size !== 4;
    livesEl.textContent = lives;
  }

  function toggleSelect(idx, el){
    if(el.classList.contains('locked')) return;
    if(selected.has(idx)) selected.delete(idx); else {
      if(selected.size >= 4) return; // max 4
      selected.add(idx);
    }
    renderGrid();
  }

  function showToast(msg){
    toastEl.textContent = msg;
    toastEl.classList.add('show');
    clearTimeout(showToast._t);
    showToast._t = setTimeout(()=> toastEl.classList.remove('show'), 1400);
  }

  function submitGuess(){
    if(selected.size !== 4) return;
    // Get the candidate indices and groups
    const indices = Array.from(selected);
    const groups = indices.map(i => activeWords[i].group);
    const allSame = groups.every(g => g === groups[0]);

    if(allSame){
      const gid = groups[0];
      lockGroup(gid, indices);
      showToast(I18N[lang].ui.correct);
      selected.clear();
      renderGrid();
      if(checkWin()) {
        showToast(I18N[lang].ui.win);
        confetti();
      }
    } else {
      lives -= 1;
      showToast(I18N[lang].ui.wrong);
      if(lives <= 0){
        // Auto-reveal: pick any correct group among the selected 4 if possible, else reveal a random one
        autoReveal();
      }
      selected.clear();
      renderGrid();
    }
  }

  function lockGroup(gid, indices){
    // Pull out those 4 into a banner row; remove from grid pool
    const set = new Set(indices);
    const locked = [];
    const remain = [];
    activeWords.forEach((w,i)=> (set.has(i) ? locked.push(w) : remain.push(w)) );
    activeWords = remain;
    solvedOrder.push(gid);

    // Make banner
    const groupInfo = PUZZLE[gid];
    const t = I18N[lang].cats[gid].title;
    const n = I18N[lang].cats[gid].note;

    const row = document.createElement('div');
    row.className = 'row';
    const banner = document.createElement('div');
    banner.className = `group-banner ${groupInfo.colorClass}`;
    banner.innerHTML = `<div>${t}</div><div class="group-note">${n}</div>`;
    row.appendChild(banner);

    // Add the 4 words as locked cards (for the pretty row)
    shuffle(locked); // random order in the row
    locked.forEach(w=>{
      const c = document.createElement('div');
      c.className = 'card locked';
      c.textContent = w.text;
      row.appendChild(c);
    });

    solvedArea.appendChild(row);
  }

  function autoReveal(){
    // Try to find any complete group still in grid and reveal it
    const leftGroups = new Set(activeWords.map(w=>w.group));
    const pick = [...leftGroups][0];
    // pick first 4 words of that group
    const indices = [];
    for(let i=0;i<activeWords.length;i++){
      if(activeWords[i].group === pick){ indices.push(i); if(indices.length===4) break; }
    }
    lockGroup(pick, indices);
  }

  function checkWin(){
    return solvedOrder.length === 4;
  }

  function shuffleActive(){
    shuffle(activeWords);
    selected.clear();
    renderGrid();
  }

  function resetGame(){
    lives = 4;
    selected.clear();
    solvedOrder = [];
    solvedArea.innerHTML = '';
    seedWords();
    renderGrid();
  }

  function toggleLang(){
    lang = (lang === 'en') ? 'es' : 'en';
    langBtn.textContent = lang === 'en' ? '🌐 Español' : '🌐 English';
    // re-render solved banners with translated text
    const rows = solvedArea.querySelectorAll('.row');
    rows.forEach((row, idx) => {
      const gid = solvedOrder[idx];
      const t = I18N[lang].cats[gid].title;
      const n = I18N[lang].cats[gid].note;
      const banner = row.querySelector('.group-banner');
      if(banner) banner.innerHTML = `<div>${t}</div><div class="group-note">${n}</div>`;
    });
    // update small UI strings
    document.querySelector('#submitBtn').textContent = I18N[lang].ui.check;
    document.querySelector('#shuffleBtn').textContent = '🔀 ' + I18N[lang].ui.shuffle;
    document.querySelector('#resetBtn').textContent = '↺ ' + I18N[lang].ui.reset;
    document.querySelector('#shareBtn').textContent = '🔗 ' + I18N[lang].ui.share;
    document.querySelector('#livesChip').innerHTML = `❤️ ${I18N[lang].ui.lives}: <strong id="lives">${lives}</strong>`;
    renderGrid();
  }

  function shareResult(){
    const blocks = solvedOrder.map(gid => I18N[lang].cats[gid].title);
    const mistakes = Math.max(0, 4 - lives);
    const text =
`EJ Connections — ${lang==='en'?'Indicators Edition':'Edición de Indicadores'}
Solved: ${checkWin() ? '✅' : '—'}
Mistakes: ${mistakes}
/ ${4}
Groups:
- ${blocks.join('\n- ')}
Play: (your link)
`;
    if(navigator.clipboard){
      navigator.clipboard.writeText(text).then(()=>{
        showToast('Copied to clipboard!');
      }).catch(()=>{
        showToast('Copy failed — select and copy manually.');
      });
    } else {
      showToast('Copy not supported, sorry!');
    }
  }

  // Tiny confetti
  function confetti(){
    const count = 80;
    for(let i=0;i<count;i++){
      const s = document.createElement('span');
      s.style.position='fixed';
      s.style.left = (Math.random()*100)+'vw';
      s.style.top = '-10px';
      s.style.width = s.style.height = (6+Math.random()*6)+'px';
      s.style.background = ['#7dd3fc','#fde047','#86efac','#fca5a5','#c4b5fd'][Math.floor(Math.random()*5)];
      s.style.borderRadius='2px';
      s.style.transform=`rotate(${Math.random()*360}deg)`;
      s.style.transition='transform 1.8s linear, top 1.8s linear, opacity .4s ease';
      s.style.zIndex=9999;
      document.body.appendChild(s);
      setTimeout(()=>{
        s.style.top = (90+Math.random()*10)+'vh';
        s.style.transform = `translateY(0) rotate(${180+Math.random()*180}deg)`;
      },10);
      setTimeout(()=>{ s.style.opacity='0'; },1600);
      setTimeout(()=>{ s.remove(); },2100);
    }
  }

  // Wire up events
  submitBtn.addEventListener('click', submitGuess);
  shuffleBtn.addEventListener('click', shuffleActive);
  resetBtn.addEventListener('click', resetGame);
  shareBtn.addEventListener('click', shareResult);
  langBtn.addEventListener('click', toggleLang);

  // Init
  seedWords();
  renderGrid();
  // Translate UI labels on first render (keep EN defaults in markup)
  document.querySelector('#submitBtn').textContent = I18N[lang].ui.check;
  document.querySelector('#shuffleBtn').textContent = '🔀 ' + I18N[lang].ui.shuffle;
  document.querySelector('#resetBtn').textContent = '↺ ' + I18N[lang].ui.reset;
  document.querySelector('#shareBtn').textContent = '🔗 ' + I18N[lang].ui.share;
})();
</script>
</body>
</html>