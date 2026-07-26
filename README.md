# Personal-notes-and-reactions-app
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
<title>Записи</title>
<style>
  :root{
    --bg:#EEF0EE;
    --surface:#FFFFFF;
    --ink:#2B2E2C;
    --ink-2:#6B7169;
    --accent:#5B6F6B;
    --hairline:#D8DBD6;
    --danger:#A15C50;
  }
  @media (prefers-color-scheme: dark){
    :root{
      --bg:#1B1F1D;
      --surface:#242927;
      --ink:#E7E9E6;
      --ink-2:#9CA39D;
      --accent:#8FA6A1;
      --hairline:#3A403D;
      --danger:#C98679;
    }
  }
  *{box-sizing:border-box;}
  html,body{margin:0;padding:0;}
  body{
    background:var(--bg);
    color:var(--ink);
    font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;
    line-height:1.5;
    -webkit-tap-highlight-color:transparent;
  }
  .mono{font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;}
  #app{max-width:640px;margin:0 auto;padding:20px 16px 60px;}
  header{padding:4px 0 18px;}
  header h1{font-size:17px;font-weight:600;margin:0;color:var(--ink);}
  header p{margin:4px 0 0;font-size:13px;color:var(--ink-2);}

  #composer{
    background:var(--surface);
    border:1px solid var(--hairline);
    border-radius:10px;
    padding:12px;
    margin-bottom:22px;
  }
  textarea{
    width:100%;
    border:none;
    background:transparent;
    color:var(--ink);
    font-size:16px;
    font-family:inherit;
    resize:vertical;
    min-height:44px;
    padding:2px 2px 8px;
    outline:none;
  }
  textarea::placeholder{color:var(--ink-2);}
  .composer-actions{display:flex;justify-content:flex-end;}
  button{
    font-family:inherit;
    font-size:13px;
    border:none;
    background:none;
    color:var(--ink-2);
    cursor:pointer;
    padding:6px 4px;
  }
  button:hover{color:var(--ink);}
  button:focus-visible, textarea:focus-visible{
    outline:2px solid var(--accent);
    outline-offset:2px;
  }
  .btn-primary{
    background:var(--accent);
    color:var(--surface);
    padding:7px 14px;
    border-radius:7px;
    font-weight:500;
  }
  .btn-primary:hover{opacity:0.9;color:var(--surface);}
  .btn-primary:disabled{opacity:0.4;cursor:default;}

  #feed{display:flex;flex-direction:column;gap:14px;}
  #empty{color:var(--ink-2);font-size:14px;padding:20px 4px;text-align:center;}

  .node{}
  .node-row{
    background:var(--surface);
    border:1px solid var(--hairline);
    border-radius:9px;
    padding:10px 12px;
  }
  .node-top{
    display:flex;
    align-items:flex-start;
    gap:6px;
  }
  .toggle-btn{
    flex:none;
    padding:0 2px;
    font-size:12px;
    line-height:1.6;
    color:var(--ink-2);
    width:14px;
    text-align:center;
  }
  .node-content{
    font-size:15px;
    white-space:pre-wrap;
    word-break:break-word;
    flex:1;
  }
  .node-meta{
    display:flex;
    align-items:center;
    gap:10px;
    margin-top:6px;
  }
  .ts{font-size:11.5px;color:var(--ink-2);}
  .node-meta button{font-size:12.5px;padding:2px 2px;}

  .reply-box{margin-top:6px;padding-left:2px;}
  .reply-box textarea{
    background:var(--surface);
    border:1px solid var(--hairline);
    border-radius:8px;
    min-height:36px;
    padding:8px 10px;
    font-size:15px;
  }
  .reply-actions{display:flex;gap:6px;justify-content:flex-end;margin-top:4px;}

  .children{
    margin-left:10px;
    padding-left:14px;
    border-left:1px solid var(--hairline);
    margin-top:8px;
    display:flex;
    flex-direction:column;
    gap:8px;
  }
  .children.collapsed{display:none;}

  #export-section{margin-top:34px;border-top:1px solid var(--hairline);padding-top:14px;}
  #export-toggle{color:var(--ink-2);font-size:13px;padding:0;}
  #export-body{margin-top:10px;display:none;}
  #export-body.open{display:block;}
  #csv-out{
    width:100%;
    min-height:120px;
    font-size:12px;
    background:var(--surface);
    border:1px solid var(--hairline);
    border-radius:8px;
    padding:8px;
    color:var(--ink);
  }
  #export-actions{display:flex;gap:8px;margin-top:6px;}
  .status{font-size:12px;color:var(--ink-2);margin-top:6px;min-height:14px;}
</style>
</head>
<body>
<div id="app">
  <header>
    <h1>Записи</h1>
    <p>Факт — запись без родителя. Реакция — запись, привязанная к любой другой.</p>
  </header>

  <div id="composer">
    <textarea id="new-content" placeholder="Что произошло?" rows="2"></textarea>
    <div class="composer-actions">
      <button class="btn-primary" id="save-new" disabled>Записать</button>
    </div>
  </div>

  <div id="feed"></div>
  <div id="empty" hidden>Пока пусто. Первая запись — всегда факт.</div>

  <div id="export-section">
    <button id="export-toggle">Экспорт CSV ▾</button>
    <div id="export-body">
      <textarea id="csv-out" readonly></textarea>
      <div id="export-actions">
        <button id="copy-csv">Скопировать</button>
        <button id="refresh-csv">Обновить</button>
      </div>
    </div>
    <div class="status" id="status"></div>
  </div>
</div>

<script>
let nodes = [];
let confirmDeleteId = null;

function uid(){
  if (crypto.randomUUID) return crypto.randomUUID();
  return 'id-' + Date.now() + '-' + Math.random().toString(16).slice(2);
}

function escapeHtml(s){
  return s.replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
}

function formatTime(iso){
  const d = new Date(iso);
  const pad = n => String(n).padStart(2,'0');
  return `${pad(d.getDate())}.${pad(d.getMonth()+1)} ${pad(d.getHours())}:${pad(d.getMinutes())}`;
}

function setStatus(msg, isError){
  const el = document.getElementById('status');
  el.textContent = msg || '';
  el.style.color = isError ? 'var(--danger)' : 'var(--ink-2)';
  if (msg) setTimeout(() => { if (el.textContent === msg) el.textContent = ''; }, 3000);
}

async function loadNodes(){
  try {
    const res = await window.storage.get('nodes', false);
    nodes = res && res.value ? JSON.parse(res.value) : [];
  } catch (e) {
    nodes = [];
  }
}

async function saveNodes(){
  try {
    const res = await window.storage.set('nodes', JSON.stringify(nodes), false);
    if (!res) setStatus('Не удалось сохранить, попробуйте ещё раз', true);
  } catch (e) {
    setStatus('Ошибка сохранения: ' + e.message, true);
  }
}

function addNode(content, parentId){
  const node = {
    id: uid(),
    parent_id: parentId || null,
    content: content,
    created_at: new Date().toISOString()
  };
  nodes.push(node);
  return node;
}

function deleteNodeCascade(id){
  const toDelete = new Set([id]);
  let changed = true;
  while (changed){
    changed = false;
    for (const n of nodes){
      if (n.parent_id && toDelete.has(n.parent_id) && !toDelete.has(n.id)){
        toDelete.add(n.id);
        changed = true;
      }
    }
  }
  nodes = nodes.filter(n => !toDelete.has(n.id));
}

function childrenOf(id){
  return nodes.filter(n => n.parent_id === id)
              .sort((a,b) => a.created_at.localeCompare(b.created_at));
}

// --- цвет записи ---

const TIME_SLOTS = [
  {start:6,  end:11, hex:'#fac5ac'},
  {start:11, end:14, hex:'#fceb9f'},
  {start:14, end:17, hex:'#eefaac'},
  {start:17, end:21, hex:'#acf9fa'},
  {start:21, end:23, hex:'#acd1fa'},
  {start:23, end:30, hex:'#c2acfa'} // 23:00–06:00, конец сдвинут на +24
];

function clamp8(v){ return Math.max(0, Math.min(255, Math.round(v))); }

function hexToRgb(hex){
  hex = hex.replace('#','');
  return { r: parseInt(hex.substr(0,2),16), g: parseInt(hex.substr(2,2),16), b: parseInt(hex.substr(4,2),16) };
}

function rgbToHex(rgb){
  return '#' + [rgb.r, rgb.g, rgb.b].map(v => clamp8(v).toString(16).padStart(2,'0')).join('');
}

function mixRgb(a, b, t){
  return { r: a.r + (b.r-a.r)*t, g: a.g + (b.g-a.g)*t, b: a.b + (b.b-a.b)*t };
}

function darkenRgb(rgb, amount){
  return { r: clamp8(rgb.r-amount), g: clamp8(rgb.g-amount), b: clamp8(rgb.b-amount) };
}

// детерминированное псевдослучайное число [0,1) по строке — чтобы цвет записи не менялся при перерисовке
function seededRandom(seed){
  let h = 2166136261;
  for (let i=0;i<seed.length;i++){
    h ^= seed.charCodeAt(i);
    h = Math.imul(h, 16777619);
  }
  h += h << 13; h ^= h >>> 7; h += h << 3; h ^= h >>> 17; h += h << 5;
  return ((h >>> 0) % 100000) / 100000;
}

function timeSlotIndex(hourDecimal){
  let h = hourDecimal < 6 ? hourDecimal + 24 : hourDecimal;
  for (let i=0;i<TIME_SLOTS.length;i++){
    if (h >= TIME_SLOTS[i].start && h < TIME_SLOTS[i].end) return i;
  }
  return TIME_SLOTS.length - 1;
}

// базовый цвет записи по времени создания + лёгкий сдвиг в сторону ближайшего соседнего слота
function hueBaseRgb(node){
  const d = new Date(node.created_at);
  let h = d.getHours() + d.getMinutes()/60;
  if (!isFinite(h)) return hexToRgb(TIME_SLOTS[0].hex); // некорректная дата — берём цвет слота без сдвига
  const idx = timeSlotIndex(h);
  const slot = TIME_SLOTS[idx];
  const hh = h < 6 ? h + 24 : h;
  const progress = Math.min(Math.max((hh - slot.start) / (slot.end - slot.start), 0), 1);
  if (!isFinite(progress)) return hexToRgb(slot.hex);
  const prev = TIME_SLOTS[(idx - 1 + TIME_SLOTS.length) % TIME_SLOTS.length];
  const next = TIME_SLOTS[(idx + 1) % TIME_SLOTS.length];
  const neighborHex = progress < 0.5 ? prev.hex : next.hex;
  const edgeDistance = progress < 0.5 ? (0.5 - progress) : (progress - 0.5); // 0..0.5, ближе к границе слота — больше
  const proximityWeight = edgeDistance * 0.5;                     // до 0.25 у самой границы
  const baselineWeight = seededRandom(node.id + ':base') * 0.22;  // 0..0.22, есть всегда, даже в середине слота
  const weight = Math.min(proximityWeight + baselineWeight, 0.45);
  return mixRgb(hexToRgb(slot.hex), hexToRgb(neighborHex), weight);
}

function getSurfaceRgb(){
  const v = getComputedStyle(document.documentElement).getPropertyValue('--surface').trim();
  return hexToRgb(v || '#ffffff');
}

// осветление к цвету фона: глубина 1 — чистый цвет, глубина 20+ — фон.
// корень вместо линейной шкалы — иначе разница между соседними уровнями почти незаметна
function lightenTowardSurface(rgb, depth){
  const linear = Math.min(Math.max((depth - 1) / 19, 0), 1);
  const factor = Math.sqrt(linear);
  return mixRgb(rgb, getSurfaceRgb(), factor);
}

function nodeColors(node, depth, hasKids){
  const baseRgb = depth <= 1
    ? hueBaseRgb(node)              // факт — всегда цвет по времени суток
    : (hasKids ? hueBaseRgb(node) : hexToRgb('#b3b3b3')); // ответ — серый, если лист
  const bgRgb = lightenTowardSurface(baseRgb, depth);
  const borderRgb = darkenRgb(bgRgb, 30);
  return { bg: rgbToHex(bgRgb), border: rgbToHex(borderRgb) };
}

function buildNodeEl(node, depth){
  depth = depth || 0;
  const wrap = document.createElement('div');
  wrap.className = 'node';
  wrap.dataset.id = node.id;

  const kids = childrenOf(node.id);
  const hasKids = kids.length > 0;

  wrap.innerHTML = `
    <div class="node-row">
      <div class="node-top">
        ${hasKids ? `<button class="toggle-btn" data-id="${node.id}" aria-label="Свернуть или развернуть вложенные реакции">▾</button>` : `<span class="toggle-btn"></span>`}
        <div class="node-content">${escapeHtml(node.content)}</div>
      </div>
      <div class="node-meta">
        <span class="ts mono">${formatTime(node.created_at)}</span>
        <button class="reply-btn" data-id="${node.id}">Ответить</button>
        <button class="delete-btn" data-id="${node.id}">Удалить</button>
      </div>
    </div>
    <div class="reply-box" data-id="${node.id}" hidden>
      <textarea class="reply-input" data-id="${node.id}" rows="2" placeholder="Реакция..."></textarea>
      <div class="reply-actions">
        <button class="btn-primary save-reply-btn" data-id="${node.id}">Сохранить</button>
        <button class="cancel-reply-btn" data-id="${node.id}">Отмена</button>
      </div>
    </div>
  `;

  const colors = nodeColors(node, depth, hasKids);
  if (colors){
    const rowEl = wrap.querySelector('.node-row');
    rowEl.style.background = colors.bg;
    rowEl.style.borderColor = colors.border;
  }

  if (hasKids){
    const childrenEl = document.createElement('div');
    childrenEl.className = 'children';
    childrenEl.dataset.id = node.id;
    kids.forEach(k => childrenEl.appendChild(buildNodeEl(k, depth + 1)));
    wrap.appendChild(childrenEl);
  }

  return wrap;
}

function render(){
  confirmDeleteId = null;
  const feed = document.getElementById('feed');
  const empty = document.getElementById('empty');
  feed.innerHTML = '';
  const roots = nodes.filter(n => !n.parent_id)
                     .sort((a,b) => b.created_at.localeCompare(a.created_at));
  if (!roots.length){
    empty.hidden = false;
    return;
  }
  empty.hidden = true;
  roots.forEach(r => feed.appendChild(buildNodeEl(r, 1)));
}

function csvEscape(s){
  if (/[",\n]/.test(s)) return '"' + s.replace(/"/g,'""') + '"';
  return s;
}

function buildCsv(){
  const rows = [['id','parent_id','content','created_at']];
  nodes.slice().sort((a,b)=>a.created_at.localeCompare(b.created_at)).forEach(n=>{
    rows.push([n.id, n.parent_id || '', n.content, n.created_at]);
  });
  return rows.map(r => r.map(v => csvEscape(String(v))).join(',')).join('\n');
}

// composer
const newContent = document.getElementById('new-content');
const saveNewBtn = document.getElementById('save-new');
newContent.addEventListener('input', () => {
  saveNewBtn.disabled = newContent.value.trim().length === 0;
});
saveNewBtn.addEventListener('click', async () => {
  const val = newContent.value.trim();
  if (!val) return;
  addNode(val, null);
  newContent.value = '';
  saveNewBtn.disabled = true;
  render();
  await saveNodes();
});

// feed delegation
document.getElementById('feed').addEventListener('click', async (e) => {
  const id = e.target.dataset.id;
  if (!id) return;

  if (e.target.classList.contains('toggle-btn')){
    const childrenEl = document.querySelector(`.children[data-id="${id}"]`);
    if (childrenEl){
      const collapsed = childrenEl.classList.toggle('collapsed');
      e.target.textContent = collapsed ? '▸' : '▾';
    }
  }

  if (e.target.classList.contains('reply-btn')){
    document.querySelectorAll('.reply-box').forEach(b => { if (b.dataset.id !== id) b.hidden = true; });
    const box = document.querySelector(`.reply-box[data-id="${id}"]`);
    box.hidden = !box.hidden;
    if (!box.hidden) box.querySelector('textarea').focus();
  }

  if (e.target.classList.contains('cancel-reply-btn')){
    const box = document.querySelector(`.reply-box[data-id="${id}"]`);
    box.hidden = true;
    box.querySelector('textarea').value = '';
  }

  if (e.target.classList.contains('save-reply-btn')){
    const box = document.querySelector(`.reply-box[data-id="${id}"]`);
    const ta = box.querySelector('textarea');
    const val = ta.value.trim();
    if (!val) return;
    addNode(val, id);
    render();
    await saveNodes();
  }

  if (e.target.classList.contains('delete-btn')){
    if (confirmDeleteId === id){
      confirmDeleteId = null;
      deleteNodeCascade(id);
      render();
      await saveNodes();
    } else {
      confirmDeleteId = id;
      e.target.textContent = 'Точно?';
      e.target.style.color = 'var(--danger)';
      setTimeout(() => {
        if (confirmDeleteId === id){
          confirmDeleteId = null;
          const btn = document.querySelector(`.delete-btn[data-id="${id}"]`);
          if (btn){ btn.textContent = 'Удалить'; btn.style.color = ''; }
        }
      }, 2500);
    }
  }
});

// export
const exportToggle = document.getElementById('export-toggle');
const exportBody = document.getElementById('export-body');
const csvOut = document.getElementById('csv-out');
exportToggle.addEventListener('click', () => {
  exportBody.classList.toggle('open');
  if (exportBody.classList.contains('open')) csvOut.value = buildCsv();
});
document.getElementById('refresh-csv').addEventListener('click', () => {
  csvOut.value = buildCsv();
});
document.getElementById('copy-csv').addEventListener('click', async () => {
  try {
    await navigator.clipboard.writeText(csvOut.value);
    setStatus('Скопировано');
  } catch (e) {
    csvOut.focus();
    csvOut.select();
    setStatus('Скопируйте вручную (Ctrl/Cmd+C) — буфер обмена недоступен напрямую', true);
  }
});

// init
(async () => {
  await loadNodes();
  render();
})();
</script>
</body>
</html>