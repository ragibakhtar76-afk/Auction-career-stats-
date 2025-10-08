<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Cricket Player Manager</title>
<style>
  /* Simple, clean styles */
  :root{--bg:#f7fafc;--card:#fff;--muted:#6b7280;--accent:#2563eb}
  body{font-family:Inter,ui-sans-serif,system-ui,Segoe UI,Roboto,"Helvetica Neue",Arial; margin:0;background:var(--bg);color:#111}
  .wrap{max-width:1100px;margin:28px auto;padding:18px}
  header{display:flex;justify-content:space-between;align-items:center;gap:12px;margin-bottom:18px;}
  h1{font-size:20px;margin:0}
  .btn{background:var(--accent);color:#fff;padding:8px 12px;border-radius:8px;border:0;cursor:pointer}
  .btn.gray{background:#e5e7eb;color:#111}
  .controls{display:flex;gap:8px;flex-wrap:wrap}
  .card{background:var(--card);padding:12px;border-radius:10px;box-shadow:0 1px 4px rgba(2,6,23,0.06)}
  .grid{display:grid;grid-template-columns:1fr 300px;gap:12px}
  .filters{display:flex;gap:8px;margin-bottom:12px}
  input[type=text],input[type=number],select{padding:8px;border-radius:8px;border:1px solid #e6e6e6;min-width:0}
  .player{display:flex;gap:12px;align-items:center;border-radius:8px;padding:8px;border:1px solid #f1f5f9}
  .player img{width:56px;height:56px;border-radius:999px;object-fit:cover}
  .small{font-size:13px;color:var(--muted)}
  .meta{display:flex;gap:8px;flex-wrap:wrap;margin-top:6px}
  .actions{display:flex;gap:6px}
  .link-like{background:transparent;border:0;color:var(--accent);cursor:pointer;padding:4px}
  form .row{display:flex;gap:8px}
  form .col{flex:1}
  footer{margin-top:18px;text-align:center;color:var(--muted);font-size:13px}
  @media(max-width:900px){ .grid{grid-template-columns:1fr} .filters{flex-direction:column} .controls{width:100%} }
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1>Cricket Player Manager</h1>
    <div class="controls">
      <button id="addBtn" class="btn">+ Add Player</button>
      <button id="exportBtn" class="btn">Export CSV</button>
      <label class="btn gray" style="cursor:pointer">
        Import CSV <input id="fileInput" type="file" accept=".csv" style="display:none">
      </label>
      <button id="resetBtn" class="btn gray">Reset</button>
    </div>
  </header>

  <div class="grid">
    <main>
      <div class="card">
        <div class="filters">
          <input id="search" type="text" placeholder="Search name / team / phone">
          <select id="teamFilter"><option>All</option></select>
          <select id="roleFilter"><option>All</option></select>
        </div>

        <div id="playersList" style="display:grid;gap:10px"></div>
        <div id="noPlayers" class="small" style="display:none;padding:10px">No players found.</div>
      </div>
    </main>

    <aside>
      <div class="card">
        <h3 style="margin:0 0 8px 0">Stats</h3>
        <div>Total players: <strong id="totalCount">0</strong></div>
        <div>Filtered: <strong id="filteredCount">0</strong></div>
        <hr style="margin:12px 0">
        <div class="small">Tip: CSV columns accepted: Name,Phone,Team,Role,Photo,Matches,Runs,Average,StrikeRate</div>
      </div>

      <div id="formWrap" style="margin-top:12px;display:none">
        <div class="card">
          <h3 id="formTitle">Add Player</h3>
          <form id="playerForm">
            <input type="hidden" id="playerId">
            <div class="row" style="margin-bottom:8px">
              <input id="name" class="col" type="text" placeholder="Full name" required>
              <input id="phone" class="col" type="text" placeholder="Phone">
            </div>
            <div class="row" style="margin-bottom:8px">
              <input id="team" class="col" type="text" placeholder="Team">
              <input id="role" class="col" type="text" placeholder="Role (Batsman/Bowler)">
            </div>
            <div style="margin-bottom:8px">
              <input id="photo" type="text" placeholder="Photo URL (optional)" style="width:100%;padding:8px;border-radius:8px;border:1px solid #eee">
            </div>

            <div class="row" style="margin-bottom:8px">
              <input id="matches" type="number" placeholder="Matches" class="col">
              <input id="runs" type="number" placeholder="Runs" class="col">
            </div>
            <div class="row" style="margin-bottom:8px">
              <input id="average" type="number" step="0.1" placeholder="Average" class="col">
              <input id="sr" type="number" step="0.1" placeholder="Strike Rate" class="col">
            </div>

            <div style="display:flex;gap:8px">
              <button class="btn" type="submit">Save</button>
              <button id="cancelBtn" type="button" class="btn gray">Cancel</button>
            </div>
          </form>
        </div>
      </div>

    </aside>
  </div>

  <footer>
    Made with ❤️ — save locally or deploy to GitHub Pages / Netlify (instructions below).
  </footer>

  <template id="playerTpl">
    <div class="player">
      <img src="" alt="photo">
      <div style="flex:1">
        <div style="display:flex;justify-content:space-between;align-items:start">
          <div>
            <div class="name" style="font-weight:600"></div>
            <div class="small teamRole"></div>
          </div>
          <div class="small matches"></div>
        </div>
        <div class="meta small"></div>
        <div class="actions" style="margin-top:8px">
          <button class="edit link-like">Edit</button>
          <button class="del link-like">Delete</button>
        </div>
      </div>
    </div>
  </template>
</div>

<script>
/* Simple app logic (vanilla JS) */
const STORAGE_KEY = 'dhanbad_players_v1';
const sample = [
  {id:'p1',name:'Manshi Sharma',phone:'9876543210',team:'Dhanbad T10',role:'Batsman',photo:'https://i.pravatar.cc/150?img=12',matches:12,runs:420,average:35,strikeRate:120.5},
  {id:'p2',name:'Rahul Verma',phone:'9123456780',team:'Mixed XI',role:'All-Rounder',photo:'https://i.pravatar.cc/150?img=5',matches:10,runs:250,average:31.25,strikeRate:110.2}
];

let players = load();
let editingId = null;

/* DOM refs */
const playersList = document.getElementById('playersList');
const search = document.getElementById('search');
const teamFilter = document.getElementById('teamFilter');
const roleFilter = document.getElementById('roleFilter');
const totalCount = document.getElementById('totalCount');
const filteredCount = document.getElementById('filteredCount');
const noPlayers = document.getElementById('noPlayers');

const addBtn = document.getElementById('addBtn');
const exportBtn = document.getElementById('exportBtn');
const fileInput = document.getElementById('fileInput');
const resetBtn = document.getElementById('resetBtn');

const formWrap = document.getElementById('formWrap');
const playerForm = document.getElementById('playerForm');
const formTitle = document.getElementById('formTitle');
const cancelBtn = document.getElementById('cancelBtn');

const ids = ['playerId','name','phone','team','role','photo','matches','runs','average','sr'];
const els = {};
ids.forEach(id => els[id]=document.getElementById(id));

/* init */
render();
populateFilters();

/* events */
search.addEventListener('input', render);
teamFilter.addEventListener('change', render);
roleFilter.addEventListener('change', render);

addBtn.addEventListener('click', ()=> openForm());
cancelBtn.addEventListener('click', closeForm);

playerForm.addEventListener('submit', e=>{
  e.preventDefault();
  saveForm();
});

exportBtn.addEventListener('click', exportCSV);
fileInput.addEventListener('change', handleFileImport);
resetBtn.addEventListener('click', ()=>{
  if(!confirm('Reset to sample data?')) return;
  players = sample.slice();
  save();
  render();
  populateFilters();
});

/* functions */
function uid(prefix='p'){ return prefix + Date.now().toString(36) + Math.random().toString(36).slice(2,7); }

function load(){
  try{
    const raw = localStorage.getItem(STORAGE_KEY);
    if(raw) return JSON.parse(raw);
  }catch(e){}
  return sample.slice();
}
function save(){ localStorage.setItem(STORAGE_KEY, JSON.stringify(players)); }

function render(){
  const q = search.value.trim().toLowerCase();
  const tf = teamFilter.value;
  const rf = roleFilter.value;
  const filtered = players.filter(p=>{
    if(tf!=='All' && (p.team||'No Team')!==tf) return false;
    if(rf!=='All' && (p.role||'Unknown')!==rf) return false;
    if(!q) return true;
    return (p.name||'').toLowerCase().includes(q) || (p.team||'').toLowerCase().includes(q) || (p.role||'').toLowerCase().includes(q) || (p.phone||'').includes(q);
  });

  playersList.innerHTML='';
  if(filtered.length===0){ noPlayers.style.display='block'; } else { noPlayers.style.display='none'; }

  filtered.forEach(p=>{
    const tpl = document.getElementById('playerTpl').content.cloneNode(true);
    const root = tpl.querySelector('.player');
    tpl.querySelector('img').src = p.photo || 'https://i.pravatar.cc/80';
    tpl.querySelector('.name').textContent = p.name || 'Unnamed';
    tpl.querySelector('.teamRole').textContent = (p.team||'No Team') + ' • ' + (p.role||'Unknown');
    tpl.querySelector('.matches').textContent = 'Matches: ' + (p.matches||0);
    tpl.querySelector('.meta').textContent = 'Runs: ' + (p.runs||0) + ' • Avg: ' + (p.average||0) + ' • SR: ' + (p.strikeRate||0);
    tpl.querySelector('.edit').addEventListener('click', ()=> openForm(p.id));
    tpl.querySelector('.del').addEventListener('click', ()=> { if(confirm('Delete player?')){ players = players.filter(x=>x.id!==p.id); save(); render(); populateFilters(); }});
    playersList.appendChild(tpl);
  });

  totalCount.textContent = players.length;
  filteredCount.textContent = filtered.length;
}

function populateFilters(){
  const teams = ['All', ...Array.from(new Set(players.map(p=>p.team||'No Team')))];
  const roles = ['All', ...Array.from(new Set(players.map(p=>p.role||'Unknown')))];
  teamFilter.innerHTML = teams.map(t => `<option>${escapeHtml(t)}</option>`).join('');
  roleFilter.innerHTML = roles.map(r => `<option>${escapeHtml(r)}</option>`).join('');
}

function openForm(id=null){
  editingId = id;
  formWrap.style.display='block';
  if(id){
    const p = players.find(x=>x.id===id);
    formTitle.textContent = 'Edit Player';
    els.playerId.value = p.id;
    els.name.value = p.name || '';
    els.phone.value = p.phone || '';
    els.team.value = p.team || '';
    els.role.value = p.role || '';
    els.photo.value = p.photo || '';
    els.matches.value = p.matches || 0;
    els.runs.value = p.runs || 0;
    els.average.value = p.average || 0;
    els.sr.value = p.strikeRate || 0;
  } else {
    formTitle.textContent = 'Add Player';
    els.playerId.value = '';
    els.name.value = '';
    els.phone.value = '';
    els.team.value = '';
    els.role.value = '';
    els.photo.value = '';
    els.matches.value = 0;
    els.runs.value = 0;
    els.average.value = 0;
    els.sr.value = 0;
  }
  window.scrollTo({top:0,behavior:'smooth'});
}

function closeForm(){ editingId = null; formWrap.style.display='none'; playerForm.reset(); }

function saveForm(){
  const id = els.playerId.value || uid('p');
  const obj = {
    id,
    name: els.name.value.trim(),
    phone: els.phone.value.trim(),
    team: els.team.value.trim(),
    role: els.role.value.trim(),
    photo: els.photo.value.trim(),
    matches: Number(els.matches.value) || 0,
    runs: Number(els.runs.value) || 0,
    average: Number(els.average.value) || 0,
    strikeRate: Number(els.sr.value) || 0
  };
  const idx = players.findIndex(p=>p.id===id);
  if(idx>=0) players[idx] = obj; else players.unshift(obj);
  save();
  populateFilters();
  render();
  closeForm();
}

/* CSV import/export */
function arrayToCSV(arr){
  if(!arr.length) return '';
  const headers = Object.keys(arr[0]);
  const lines = [headers.join(',')];
  for(const r of arr){
    lines.push(headers.map(h=>{
      const v = r[h] ?? '';
      if(typeof v === 'string' && v.includes(',')) return `"${v.replace(/"/g,'""')}"`;
      return v;
    }).join(','));
  }
  return lines.join('\\n');
}
function exportCSV(){
  const csv = arrayToCSV(players.map(p=>({
    Name:p.name,Phone:p.phone,Team:p.team,Role:p.role,Photo:p.photo,Matches:p.matches,Runs:p.runs,Average:p.average,StrikeRate:p.strikeRate
  })));
  const blob = new Blob([csv],{type:'text/csv'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download='players.csv'; document.body.appendChild(a); a.click(); a.remove();
  URL.revokeObjectURL(url);
}

function handleFileImport(e){
  const f = e.target.files[0];
  if(!f) return;
  const reader = new FileReader();
  reader.onload = ()=>{
    const csv = reader.result;
    const arr = parseCSV(csv);
    const mapped = arr.map(row=>({
      id: uid('p'),
      name: row.Name || row.name || '',
      phone: row.Phone || row.phone || '',
      team: row.Team || row.team || '',
      role: row.Role || row.role || '',
      photo: row.Photo || row.photo || '',
      matches: Number(row.Matches || row.matches || 0),
      runs: Number(row.Runs || row.runs || 0),
      average: Number(row.Average || row.average || 0),
      strikeRate: Number(row.StrikeRate || row.strikeRate || 0)
    }));
    players = [...mapped, ...players];
    save(); populateFilters(); render();
    alert('Imported '+mapped.length+' players');
    e.target.value = '';
  };
  reader.readAsText(f);
}

/* Simple CSV parser handling quoted fields */
function parseCSV(text){
  const rows = [];
  const lines = text.split(/\\r?\\n/).filter(l=>l.trim()!=='');
  if(!lines.length) return [];
  const headers = parseLine(lines[0]);
  for(let i=1;i<lines.length;i++){
    const cols = parseLine(lines[i]);
    const obj = {};
    headers.forEach((h,idx)=> obj[h.trim()] = (cols[idx] !== undefined) ? cols[idx] : '');
    rows.push(obj);
  }
  return rows;
}
function parseLine(line){
  const out=[]; let cur=''; let inQuotes=false;
  for(let i=0;i<line.length;i++){
    const ch=line[i];
    if(inQuotes){
      if(ch==='\"' && line[i+1]==='\"'){ cur += '\"'; i++; continue; }
      if(ch==='\"'){ inQuotes=false; continue; }
      cur += ch;
    } else {
      if(ch==='\"'){ inQuotes=true; continue; }
      if(ch===','){ out.push(cur); cur=''; continue; }
      cur+=ch;
    }
  }
  out.push(cur);
  return out;
}

function escapeHtml(s){ return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }

function handleFileDrop(){}
</script>
</body>
</html>
