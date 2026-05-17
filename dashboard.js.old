// ============ SUPABASE ============
const SUPABASE_URL = 'https://ujswyksemvfljescqiuj.supabase.co';
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InVqc3d5a3NlbXZmbGplc2NxaXVqIiwicm9sZSI6ImFub24iLCJpYXQiOjE3Nzg0MzE2ODksImV4cCI6MjA5NDAwNzY4OX0.tFABBDoOuwXKF0RKTssM-kG6DFJLS_kNh_xWlFP_1xk';
const sb = window.supabase.createClient(SUPABASE_URL, SUPABASE_KEY);

let allData = [];
let globalSemester = '';
const chartInstances = {};
const COLORS = ['#6366f1','#8b5cf6','#06b6d4','#10b981','#f59e0b','#ef4444','#ec4899','#14b8a6','#f97316','#84cc16','#a78bfa','#fb923c'];

// ============ INIT ============
document.addEventListener('DOMContentLoaded', loadData);

// ============ NORMALISASI ASAL JURUSAN ============
const JURUSAN_MAP = [
  { canonical: 'RPL (Rekayasa Perangkat Lunak)', patterns: ['rpl', 'rekayasa perangkat lunak', 'smk rpl', 'smk rekayasa perangkat lunak'] },
  { canonical: 'IPA', patterns: ['ipa', 'sma ipa', 'mipa', 'sma mipa', 'ilmu pengetahuan alam', 'sma ipa/mipa'] },
  { canonical: 'IPS', patterns: ['ips', 'sma ips', 'ilmu pengetahuan sosial'] },
  { canonical: 'TKJ (Teknik Komputer & Jaringan)', patterns: ['tkj', 'teknik komputer jaringan', 'teknik komputer dan jaringan', 'smk tkj'] },
  { canonical: 'Multimedia', patterns: ['multimedia', 'mm', 'smk multimedia', 'desain multimedia'] },
  { canonical: 'Akuntansi', patterns: ['akuntansi', 'smk akuntansi', 'akutansi'] },
  { canonical: 'Administrasi Perkantoran', patterns: ['administrasi perkantoran', 'ap', 'otkp', 'smk administrasi'] },
  { canonical: 'Teknik Informatika', patterns: ['teknik informatika', 'ti', 'informatika'] },
  { canonical: 'Bahasa', patterns: ['bahasa', 'sma bahasa', 'bahasa dan sastra'] },
  { canonical: 'DKV (Desain Komunikasi Visual)', patterns: ['dkv', 'desain komunikasi visual', 'desain grafis'] },
  { canonical: 'Pemasaran', patterns: ['pemasaran', 'smk pemasaran', 'bisnis daring dan pemasaran', 'bdp'] },
];

function normalizeJurusan(val) {
  if (!val) return val;
  const lower = val.toString().trim().toLowerCase();
  for (const entry of JURUSAN_MAP) {
    if (entry.patterns.includes(lower)) return entry.canonical;
  }
  // Title case fallback for unmapped values
  return val.trim().replace(/\b\w/g, c => c.toUpperCase());
}

function normalizeData(data) {
  return data.map(r => ({
    ...r,
    asal_jurusan: normalizeJurusan(r.asal_jurusan)
  }));
}

// ============ INIT ============
async function loadData() {
  const { data, error } = await sb.from('responses').select('*').order('created_at', { ascending: false });
  document.getElementById('loading').style.display = 'none';

  if (error || !data || data.length === 0) {
    document.getElementById('empty').style.display = '';
    if (error) console.error(error);
    return;
  }

  allData = normalizeData(data);
  document.getElementById('dashboard').style.display = '';
  document.getElementById('last-updated').textContent = `${data.length} responden • ${new Date().toLocaleDateString('id-ID')}`;

  buildSemesterChips();
  renderAll();
  bindTableFilters();
}

// ============ GLOBAL SEMESTER FILTER ============
function getGlobalFiltered() {
  if (!globalSemester) return allData;
  return allData.filter(r => r.semester === globalSemester);
}

function buildSemesterChips() {
  const sems = [...new Set(allData.map(r => r.semester).filter(Boolean))].sort();
  const container = document.getElementById('semester-chips');
  container.innerHTML = `<button class="chip active" data-semester="" onclick="setGlobalSemester('')">Semua (${allData.length})</button>`;
  sems.forEach(s => {
    const count = allData.filter(r => r.semester === s).length;
    container.innerHTML += `<button class="chip" data-semester="${s}" onclick="setGlobalSemester('${s}')">Semester ${s} (${count})</button>`;
  });

  // Populate table semester filter too
  const sel = document.getElementById('filter-semester');
  sel.innerHTML = '<option value="">Semua Semester</option>';
  sems.forEach(s => { sel.innerHTML += `<option value="${s}">Semester ${s}</option>`; });
}

function setGlobalSemester(sem) {
  globalSemester = sem;

  // Update chip active states
  document.querySelectorAll('#semester-chips .chip').forEach(c => {
    c.classList.toggle('active', c.dataset.semester === sem);
  });

  // Sync table semester filter
  document.getElementById('filter-semester').value = sem;

  renderAll();
}

function renderAll() {
  const filtered = getGlobalFiltered();
  renderStats(filtered);
  renderCharts(filtered);
  renderOpenResponses(filtered);
  renderTable();
  renderInsights(filtered);
}

// ============ STATS ============
function getAgeRanges(d) {
  const ranges = { '<20': 0, '20-25': 0, '26-30': 0, '>30': 0 };
  d.forEach(r => {
    const u = r.usia;
    if (!u) return;
    if (u < 20) ranges['<20']++;
    else if (u <= 25) ranges['20-25']++;
    else if (u <= 30) ranges['26-30']++;
    else ranges['>30']++;
  });
  return ranges;
}

function renderStats(d) {
  const n = d.length;
  const ages = getAgeRanges(d);
  const codingYa = d.filter(r => r.pernah_coding && r.pernah_coding.toString().trim().toLowerCase() === 'ya').length;
  const codingTidak = n - codingYa;
  const pctCoding = n ? Math.round(codingYa / n * 100) : 0;
  const avgMath = n ? (d.reduce((s, r) => s + (r.kemampuan_matematika || 0), 0) / n).toFixed(1) : 0;
  const avgConf = n ? (d.reduce((s, r) => s + (r.confidence || 0), 0) / n).toFixed(1) : 0;
  const avgMinat = n ? (d.reduce((s, r) => s + (r.minat || 0), 0) / n).toFixed(1) : 0;

  const semLabel = globalSemester ? `Semester ${globalSemester}` : 'Semua Semester';

  document.getElementById('stats-grid').innerHTML = `
    <div class="stat-card accent"><div class="icon">👥</div><div class="value">${n}</div><div class="label">Total Responden</div><div class="sub-label">${semLabel}</div></div>
    <div class="stat-card cyan"><div class="icon">👶</div><div class="value">${ages['<20']}</div><div class="label">Usia < 20</div></div>
    <div class="stat-card success"><div class="icon">🧑</div><div class="value">${ages['20-25']}</div><div class="label">Usia 20-25</div></div>
    <div class="stat-card warning"><div class="icon">👨</div><div class="value">${ages['26-30']}</div><div class="label">Usia 26-30</div></div>
    <div class="stat-card pink"><div class="icon">🧔</div><div class="value">${ages['>30']}</div><div class="label">Usia > 30</div></div>
    <div class="stat-card cyan"><div class="icon">💻</div><div class="value">${pctCoding}%</div><div class="label">Pernah Coding</div><div class="sub-label">${codingYa} dari ${n} mahasiswa</div></div>
    <div class="stat-card accent"><div class="icon">📐</div><div class="value">${avgMath}/5</div><div class="label">Avg Matematika</div></div>
    <div class="stat-card pink"><div class="icon">🎯</div><div class="value">${avgMinat}/5</div><div class="label">Avg Minat</div></div>
    <div class="stat-card success"><div class="icon">💪</div><div class="value">${avgConf}/5</div><div class="label">Avg Confidence</div></div>
  `;
}

// ============ CHART HELPERS ============
Chart.defaults.color = '#94a3b8';
Chart.defaults.borderColor = 'rgba(255,255,255,0.06)';
Chart.defaults.font.family = 'Inter';

function countField(data, field) {
  const map = {};
  data.forEach(r => {
    const v = r[field];
    if (Array.isArray(v)) v.forEach(i => { if (i) map[i] = (map[i] || 0) + 1; });
    else if (v) map[v] = (map[v] || 0) + 1;
  });
  return Object.entries(map).sort((a, b) => b[1] - a[1]);
}

function destroyChart(id) {
  if (chartInstances[id]) { chartInstances[id].destroy(); delete chartInstances[id]; }
}

function makePie(id, data, field) {
  destroyChart(id);
  const entries = countField(data, field);
  if (!entries.length) return;
  chartInstances[id] = new Chart(document.getElementById(id), {
    type: 'doughnut',
    data: { labels: entries.map(e => e[0]), datasets: [{ data: entries.map(e => e[1]), backgroundColor: COLORS, borderWidth: 0 }] },
    options: { responsive: true, plugins: { legend: { position: 'bottom', labels: { boxWidth: 12, padding: 8, font: { size: 11 } } } } }
  });
}

function makeBar(id, data, field) {
  destroyChart(id);
  const entries = countField(data, field);
  if (!entries.length) return;
  chartInstances[id] = new Chart(document.getElementById(id), {
    type: 'bar',
    data: { labels: entries.map(e => e[0]), datasets: [{ data: entries.map(e => e[1]), backgroundColor: COLORS.slice(0, entries.length), borderRadius: 6, borderWidth: 0 }] },
    options: {
      indexAxis: 'y', responsive: true,
      plugins: { legend: { display: false } },
      scales: { x: { grid: { display: false } }, y: { grid: { display: false } } }
    }
  });
}

function makeRatingBar(id, data, field) {
  destroyChart(id);
  const map = { 1: 0, 2: 0, 3: 0, 4: 0, 5: 0 };
  data.forEach(r => { const v = r[field]; if (v >= 1 && v <= 5) map[v]++; });
  chartInstances[id] = new Chart(document.getElementById(id), {
    type: 'bar',
    data: { labels: ['1', '2', '3', '4', '5'], datasets: [{ data: Object.values(map), backgroundColor: ['#ef4444', '#f97316', '#f59e0b', '#10b981', '#6366f1'], borderRadius: 6, borderWidth: 0 }] },
    options: { responsive: true, plugins: { legend: { display: false } }, scales: { x: { grid: { display: false } }, y: { grid: { display: false }, beginAtZero: true } } }
  });
}

function makeAgeRangeBar(id, data) {
  destroyChart(id);
  const ranges = getAgeRanges(data);
  chartInstances[id] = new Chart(document.getElementById(id), {
    type: 'bar',
    data: {
      labels: ['< 20', '20-25', '26-30', '> 30'],
      datasets: [{ data: Object.values(ranges), backgroundColor: ['#06b6d4', '#10b981', '#f59e0b', '#ec4899'], borderRadius: 6, borderWidth: 0 }]
    },
    options: { responsive: true, plugins: { legend: { display: false } }, scales: { x: { grid: { display: false } }, y: { grid: { display: false }, beginAtZero: true } } }
  });
}

// ============ RENDER CHARTS ============
function renderCharts(d) {
  makePie('chart-gender', d, 'jenis_kelamin');
  makeAgeRangeBar('chart-usia', d);
  makePie('chart-semester', d, 'semester');
  makeBar('chart-jurusan', d, 'asal_jurusan');
  makeRatingBar('chart-math', d, 'kemampuan_matematika');
  makePie('chart-coding', d, 'pernah_coding');
  makeBar('chart-lang', d, 'bahasa_pemrograman');
  makeBar('chart-motivasi', d, 'motivasi');
  makePie('chart-pemahaman', d, 'pemahaman');
  makeBar('chart-bidang', d, 'bidang_minat');
  makeBar('chart-tools', d, 'tools');
  makeBar('chart-kesulitan', d, 'kesulitan');
  makeBar('chart-industri', d, 'industri');
  makeBar('chart-medsos', d, 'media_sosial');
  makePie('chart-internet', d, 'penggunaan_internet');
}

// ============ TABLE ============
const TABLE_COLS = [
  { key: 'nim', label: 'NIM' },
  { key: 'nama', label: 'Nama' },
  { key: 'semester', label: 'Sem' },
  { key: 'jenis_kelamin', label: 'Gender' },
  { key: 'domisili', label: 'Domisili' },
  { key: 'asal_jurusan', label: 'Asal Jurusan' },
  { key: 'kemampuan_matematika', label: 'Mat' },
  { key: 'pernah_coding', label: 'Coding' },
  { key: 'minat', label: 'Minat' },
  { key: 'confidence', label: 'PD' },
  { key: 'pekerjaan_impian', label: 'Pekerjaan Impian' },
  { key: 'created_at', label: 'Waktu' }
];

function bindTableFilters() {
  document.getElementById('search').addEventListener('input', renderTable);
  document.getElementById('filter-semester').addEventListener('change', () => {
    setGlobalSemester(document.getElementById('filter-semester').value);
  });
  document.getElementById('filter-gender').addEventListener('change', renderTable);
}

function getFilteredData() {
  const q = document.getElementById('search').value.toLowerCase();
  const gen = document.getElementById('filter-gender').value;
  let data = getGlobalFiltered();
  return data.filter(r => {
    if (q && !r.nim?.toLowerCase().includes(q) && !r.nama?.toLowerCase().includes(q)) return false;
    if (gen && r.jenis_kelamin !== gen) return false;
    return true;
  });
}

function renderTable() {
  const filtered = getFilteredData();
  const total = getGlobalFiltered().length;
  document.getElementById('row-count').textContent = `${filtered.length} dari ${total} data`;

  const thead = document.getElementById('table-head');
  thead.innerHTML = '<tr>' + TABLE_COLS.map(c => `<th>${c.label}</th>`).join('') + '</tr>';

  const tbody = document.getElementById('table-body');
  tbody.innerHTML = filtered.map(r => '<tr>' + TABLE_COLS.map(c => {
    let v = r[c.key];
    if (c.key === 'created_at') v = v ? new Date(v).toLocaleDateString('id-ID') : '-';
    else if (Array.isArray(v)) v = v.map(i => `<span class="tag">${i}</span>`).join(' ');
    else v = v ?? '-';
    return `<td>${v}</td>`;
  }).join('') + '</tr>').join('');
}

// ============ EXPORT CSV ============
function exportCSV() {
  const filtered = getFilteredData();
  const allKeys = ['nim','nama','usia','jenis_kelamin','domisili','semester','asal_jurusan','kemampuan_matematika','pernah_coding','bahasa_pemrograman','motivasi','minat','pemahaman','bidang_minat','ekspektasi','waktu_belajar','metode_belajar','platform','tools','confidence','kesulitan','kebutuhan','pekerjaan_impian','industri','penggunaan_internet','media_sosial','konten_edukasi','alasan','pengalaman','kekhawatiran','created_at'];
  let csv = allKeys.join(',') + '\n';
  filtered.forEach(r => {
    csv += allKeys.map(k => {
      let v = r[k];
      if (Array.isArray(v)) v = v.join('; ');
      if (typeof v === 'string' && (v.includes(',') || v.includes('"') || v.includes('\n')))
        v = '"' + v.replace(/"/g, '""') + '"';
      return v ?? '';
    }).join(',') + '\n';
  });
  const blob = new Blob(['\uFEFF' + csv], { type: 'text/csv;charset=utf-8;' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = `survei_persona_${globalSemester ? 'sem' + globalSemester + '_' : ''}${new Date().toISOString().slice(0,10)}.csv`;
  a.click();
}

// ============ OPEN RESPONSES (Alasan, Pengalaman, Kekhawatiran) ============
const STOPWORDS = new Set([
  'yang','dan','di','ke','dari','untuk','dengan','pada','adalah','ini','itu',
  'saya','saya','aku','akan','bisa','juga','tidak','ada','sudah','lebih',
  'sangat','banyak','karena','agar','supaya','apa','bagaimana','merasa',
  'dalam','menjadi','secara','serta','oleh','hal','seperti','telah','belum',
  'dapat','harus','atau','maupun','namun','tetapi','jika','kalau','ketika',
  'setelah','sebelum','sedang','masih','pernah','mau','ingin','perlu',
  'tentang','terhadap','antara','melalui','hingga','sampai','sejak',
  'saat','waktu','sering','jarang','paling','cukup','kurang','terlalu',
  'lagi','baru','hanya','semua','setiap','beberapa','para','nya','kita',
  'kami','mereka','dia','ia','satu','dua','tersebut','yaitu','bahwa',
  'karena','sebagai','menurut','tanpa','selain','antara','baik','jadi',
  'sama','lain','begitu','pun','diri','sendiri','orang','hal','memang',
  'ya','bagi','rata','cuma','nggak','gak','banget','aja','kalo','udah',
  'nih','sih','dong','deh','lah','kok','terus','bikin','dulu','gimana',
  'kayak','kali','yg','ga','maka','dgn','kok','tp','jg','dll','dsb',
  'tak','me','se','ber','ter','per','ke','an','kan','lah','tapi'
]);

function extractKeywords(texts, limit = 20) {
  const freq = {};
  texts.forEach(text => {
    if (!text) return;
    const words = text.toString().toLowerCase()
      .replace(/[^a-zA-Z\s]/g, ' ')
      .split(/\s+/)
      .filter(w => w.length > 2 && !STOPWORDS.has(w));
    const seen = new Set();
    words.forEach(w => {
      if (!seen.has(w)) {
        freq[w] = (freq[w] || 0) + 1;
        seen.add(w);
      }
    });
  });
  return Object.entries(freq)
    .filter(e => e[1] >= 2)
    .sort((a, b) => b[1] - a[1])
    .slice(0, limit);
}

function renderKeywordCloud(containerId, keywords, total) {
  const el = document.getElementById(containerId);
  if (!el) return;
  if (!keywords.length) {
    el.innerHTML = '<span class="no-data">Belum ada data</span>';
    return;
  }
  const maxCount = keywords[0][1];
  const TAG_COLORS = [
    { bg: 'rgba(99,102,241,0.2)', border: 'rgba(99,102,241,0.4)', text: '#a5b4fc' },
    { bg: 'rgba(139,92,246,0.2)', border: 'rgba(139,92,246,0.4)', text: '#c4b5fd' },
    { bg: 'rgba(6,182,212,0.2)', border: 'rgba(6,182,212,0.4)', text: '#67e8f9' },
    { bg: 'rgba(16,185,129,0.2)', border: 'rgba(16,185,129,0.4)', text: '#6ee7b7' },
    { bg: 'rgba(245,158,11,0.2)', border: 'rgba(245,158,11,0.4)', text: '#fcd34d' },
    { bg: 'rgba(236,72,153,0.2)', border: 'rgba(236,72,153,0.4)', text: '#f9a8d4' },
  ];
  el.innerHTML = keywords.map((kw, i) => {
    const size = 0.72 + (kw[1] / maxCount) * 0.5;
    const color = TAG_COLORS[i % TAG_COLORS.length];
    const pct = Math.round(kw[1] / total * 100);
    return `<span class="kw-tag" style="font-size:${size}rem;background:${color.bg};border-color:${color.border};color:${color.text}" title="${kw[1]} responden (${pct}%)">${kw[0]} <small>${kw[1]}</small></span>`;
  }).join('');
}

function renderResponseCards(containerId, countId, data, field) {
  const el = document.getElementById(containerId);
  const countEl = document.getElementById(countId);
  if (!el) return;
  const responses = data.filter(r => r[field] && r[field].toString().trim());
  if (countEl) countEl.textContent = `(${responses.length})`;
  if (!responses.length) {
    el.innerHTML = '<div class="no-data">Belum ada respons</div>';
    return;
  }
  el.innerHTML = responses.map(r => {
    const text = r[field].toString().trim();
    const name = r.nama || 'Anonim';
    const sem = r.semester ? `Sem ${r.semester}` : '';
    return `<div class="response-item">
      <div class="response-text">"${escapeHtml(text)}"</div>
      <div class="response-meta"><span class="response-name">${escapeHtml(name)}</span>${sem ? ` <span class="response-sem">${sem}</span>` : ''}</div>
    </div>`;
  }).join('');
}

function escapeHtml(str) {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}

function renderOpenResponses(d) {
  const fields = [
    { field: 'alasan', kwId: 'keywords-alasan', respId: 'responses-alasan', countId: 'count-alasan' },
    { field: 'pengalaman', kwId: 'keywords-pengalaman', respId: 'responses-pengalaman', countId: 'count-pengalaman' },
    { field: 'kekhawatiran', kwId: 'keywords-kekhawatiran', respId: 'responses-kekhawatiran', countId: 'count-kekhawatiran' },
  ];
  fields.forEach(({ field, kwId, respId, countId }) => {
    const texts = d.map(r => r[field]).filter(Boolean);
    const keywords = extractKeywords(texts);
    renderKeywordCloud(kwId, keywords, d.length);
    renderResponseCards(respId, countId, d, field);
  });
}

// ============ INSIGHTS ============
function renderInsights(d) {
  const container = document.getElementById('insight-content');
  if (!container || !d.length) {
    if (container) container.innerHTML = '<p class="insight-empty">Tidak ada data untuk ditampilkan.</p>';
    return;
  }

  const n = d.length;
  const semLabel = globalSemester ? `Semester ${globalSemester}` : 'keseluruhan semester';

  // ---- Demographics ----
  const genderCount = {};
  d.forEach(r => { const g = r.jenis_kelamin; if (g) genderCount[g] = (genderCount[g] || 0) + 1; });
  const genderEntries = Object.entries(genderCount).sort((a, b) => b[1] - a[1]);
  const dominantGender = genderEntries.length ? genderEntries[0] : ['', 0];
  const genderPct = Math.round(dominantGender[1] / n * 100);

  const ages = { '<20': 0, '20-25': 0, '26-30': 0, '>30': 0 };
  d.forEach(r => {
    const u = r.usia;
    if (!u) return;
    if (u < 20) ages['<20']++;
    else if (u <= 25) ages['20-25']++;
    else if (u <= 30) ages['26-30']++;
    else ages['>30']++;
  });
  const ageEntries = Object.entries(ages).sort((a, b) => b[1] - a[1]);
  const topAge = ageEntries[0];
  const topAgePct = Math.round(topAge[1] / n * 100);

  // ---- Coding ----
  const codingYa = d.filter(r => r.pernah_coding && r.pernah_coding.toString().trim().toLowerCase() === 'ya').length;
  const pctCoding = Math.round(codingYa / n * 100);

  // ---- Top items helper ----
  function topItems(field, limit = 3) {
    const map = {};
    d.forEach(r => {
      const v = r[field];
      if (Array.isArray(v)) v.forEach(i => { if (i) map[i] = (map[i] || 0) + 1; });
      else if (v) map[v] = (map[v] || 0) + 1;
    });
    return Object.entries(map).sort((a, b) => b[1] - a[1]).slice(0, limit);
  }

  // ---- Skills & Tools ----
  const topLangs = topItems('bahasa_pemrograman', 3);
  const topTools = topItems('tools', 3);
  const avgMath = (d.reduce((s, r) => s + (r.kemampuan_matematika || 0), 0) / n).toFixed(1);
  const avgConf = (d.reduce((s, r) => s + (r.confidence || 0), 0) / n).toFixed(1);
  const avgMinat = (d.reduce((s, r) => s + (r.minat || 0), 0) / n).toFixed(1);

  // ---- Motivation ----
  const topMotivasi = topItems('motivasi', 3);
  const topBidang = topItems('bidang_minat', 3);

  // ---- Challenges ----
  const topKesulitan = topItems('kesulitan', 3);
  const topKebutuhan = topItems('kebutuhan', 3);

  // ---- Career ----
  const topIndustri = topItems('industri', 3);
  const topPekerjaan = topItems('pekerjaan_impian', 3);

  // ---- Digital ----
  const topMedsos = topItems('media_sosial', 3);
  const topInternet = topItems('penggunaan_internet', 1);

  // ---- Pemahaman ----
  const pemahamanEntries = topItems('pemahaman', 5);
  const topPemahaman = pemahamanEntries.length ? pemahamanEntries[0] : ['', 0];

  // ---- Metode belajar ----
  const topMetode = topItems('metode_belajar', 3);

  // Format list helper
  function fmtList(items) {
    return items.map((e, i) => {
      const pct = Math.round(e[1] / n * 100);
      return `<strong>${e[0]}</strong> (${pct}%)`;
    }).join(', ');
  }

  function fmtListCount(items) {
    return items.map(e => `<strong>${e[0]}</strong> (${e[1]} orang)`).join(', ');
  }

  // ---- Build insight paragraphs ----
  let html = '';

  // 1. Demografi
  html += `<div class="insight-block">`;
  html += `<div class="insight-title">📊 Profil Demografi</div>`;
  html += `<p>Berdasarkan data <strong>${n} responden</strong> pada <strong>${semLabel}</strong>, `;
  html += `mayoritas mahasiswa berjenis kelamin <strong>${dominantGender[0]}</strong> sebanyak <strong>${dominantGender[1]} orang (${genderPct}%)</strong>. `;
  html += `Dari segi usia, kelompok terbesar berada di rentang <strong>${topAge[0]} tahun</strong> yaitu sebanyak <strong>${topAge[1]} orang (${topAgePct}%)</strong>.`;
  if (genderEntries.length > 1) {
    const minGender = genderEntries[genderEntries.length - 1];
    const minPct = Math.round(minGender[1] / n * 100);
    html += ` Sementara mahasiswa ${minGender[0]} berjumlah <strong>${minGender[1]} orang (${minPct}%)</strong>.`;
  }
  html += `</p></div>`;

  // 2. Kemampuan & Pengalaman
  html += `<div class="insight-block">`;
  html += `<div class="insight-title">💻 Kemampuan & Pengalaman</div>`;
  html += `<p>Sebanyak <strong>${pctCoding}% (${codingYa} dari ${n})</strong> mahasiswa menyatakan pernah memiliki pengalaman coding sebelumnya. `;
  html += `Rata-rata kemampuan matematika berada di level <strong>${avgMath}/5</strong>, `;
  html += `dengan tingkat kepercayaan diri (confidence) rata-rata <strong>${avgConf}/5</strong>. `;
  if (topLangs.length) {
    html += `Bahasa pemrograman yang paling banyak dikuasai adalah ${fmtList(topLangs)}. `;
  }
  if (topTools.length) {
    html += `Tools yang paling banyak digunakan meliputi ${fmtList(topTools)}.`;
  }
  html += `</p></div>`;

  // 3. Motivasi & Minat
  html += `<div class="insight-block">`;
  html += `<div class="insight-title">🎯 Motivasi & Minat</div>`;
  html += `<p>Rata-rata tingkat minat mahasiswa terhadap Sains Data berada di <strong>${avgMinat}/5</strong>. `;
  if (topMotivasi.length) {
    html += `Motivasi utama memilih program studi ini adalah ${fmtList(topMotivasi)}. `;
  }
  html += `Tingkat pemahaman yang paling dominan adalah <strong>${topPemahaman[0]}</strong> (${Math.round(topPemahaman[1] / n * 100)}%). `;
  if (topBidang.length) {
    html += `Bidang minat yang paling diminati meliputi ${fmtList(topBidang)}.`;
  }
  html += `</p></div>`;

  // 4. Tantangan & Kebutuhan
  html += `<div class="insight-block">`;
  html += `<div class="insight-title">⚡ Tantangan & Kebutuhan</div>`;
  html += `<p>`;
  if (topKesulitan.length) {
    html += `Kesulitan utama yang dihadapi mahasiswa adalah ${fmtList(topKesulitan)}. `;
  }
  if (topKebutuhan.length) {
    html += `Untuk mendukung proses belajar, mahasiswa paling membutuhkan ${fmtList(topKebutuhan)}. `;
  }
  if (topMetode.length) {
    html += `Metode belajar yang paling disukai adalah ${fmtList(topMetode)}.`;
  }
  html += `</p></div>`;

  // 5. Karier & Industri
  html += `<div class="insight-block">`;
  html += `<div class="insight-title">💼 Aspirasi Karier</div>`;
  html += `<p>`;
  if (topPekerjaan.length) {
    html += `Pekerjaan impian yang paling banyak disebutkan adalah ${fmtListCount(topPekerjaan)}. `;
  }
  if (topIndustri.length) {
    html += `Industri yang paling diminati meliputi ${fmtList(topIndustri)}.`;
  }
  html += `</p></div>`;

  // 6. Perilaku Digital
  html += `<div class="insight-block">`;
  html += `<div class="insight-title">📱 Perilaku Digital</div>`;
  html += `<p>`;
  if (topInternet.length) {
    html += `Mayoritas mahasiswa menggunakan internet selama <strong>${topInternet[0][0]}</strong> per hari (${Math.round(topInternet[0][1] / n * 100)}%). `;
  }
  if (topMedsos.length) {
    html += `Media sosial yang paling aktif digunakan adalah ${fmtList(topMedsos)}.`;
  }
  html += `</p></div>`;

  // 7. Pertanyaan Terbuka Insight
  const alasanTexts = d.map(r => r.alasan).filter(Boolean);
  const pengalamanTexts = d.map(r => r.pengalaman).filter(Boolean);
  const kekhawatiranTexts = d.map(r => r.kekhawatiran).filter(Boolean);

  const kwAlasan = extractKeywords(alasanTexts, 5);
  const kwPengalaman = extractKeywords(pengalamanTexts, 5);
  const kwKekhawatiran = extractKeywords(kekhawatiranTexts, 5);

  html += `<div class="insight-block">`;
  html += `<div class="insight-title">✍️ Analisis Respons Terbuka</div>`;
  html += `<p>`;
  if (kwAlasan.length) {
    html += `Dari <strong>${alasanTexts.length} respons</strong> tentang alasan memilih Sains Data, tema yang paling sering muncul adalah ${kwAlasan.map(k => `<strong>${k[0]}</strong>`).join(', ')}. `;
  }
  if (kwPengalaman.length) {
    html += `Dalam pengalaman belajar (<strong>${pengalamanTexts.length} respons</strong>), kata kunci dominan meliputi ${kwPengalaman.map(k => `<strong>${k[0]}</strong>`).join(', ')}. `;
  }
  if (kwKekhawatiran.length) {
    html += `Kekhawatiran utama (<strong>${kekhawatiranTexts.length} respons</strong>) mencakup tema ${kwKekhawatiran.map(k => `<strong>${k[0]}</strong>`).join(', ')}. `;
  }
  html += `</p></div>`;

  // 8. Kesimpulan
  html += `<div class="insight-block insight-summary">`;
  html += `<div class="insight-title">📝 Kesimpulan</div>`;
  html += `<p>Secara keseluruhan, mahasiswa pada <strong>${semLabel}</strong> `;
  html += `didominasi oleh ${dominantGender[0]} (${genderPct}%) dengan rentang usia terbanyak ${topAge[0]} tahun. `;
  html += `${pctCoding}% sudah memiliki pengalaman coding, `;
  html += `namun rata-rata tingkat kepercayaan diri masih di level ${avgConf}/5. `;
  if (parseFloat(avgConf) < 3) {
    html += `Hal ini menunjukkan perlunya peningkatan program pendampingan dan latihan praktis untuk membangun kepercayaan diri mahasiswa. `;
  } else if (parseFloat(avgConf) >= 4) {
    html += `Ini menunjukkan mahasiswa cukup percaya diri dengan kemampuan mereka saat ini. `;
  }
  if (topKesulitan.length) {
    html += `Tantangan terbesar adalah <strong>${topKesulitan[0][0]}</strong>, `;
    html += `sehingga perlu perhatian khusus dalam kurikulum dan metode pengajaran.`;
  }
  html += `</p></div>`;

  container.innerHTML = html;
}
