<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="utf-8"/>
  <title>Prediksi 3 Digit (4→3) — Pola A–T Eksperimen</title>
  <meta name="viewport" content="width=device-width,initial-scale=1"/>
  <style>
    body { font-family: Arial, sans-serif; margin:20px; background:#f6f8fa; color:#222; }
    h2 { margin-bottom:6px; }
    textarea { width:100%; height:150px; padding:8px; font-family:monospace; box-sizing:border-box; }
    .pattern-row { display:flex; flex-wrap:wrap; gap:6px; margin:10px 0; }
    .btn { padding:8px 10px; border-radius:6px; border:1px solid #cfcfcf; background:white; cursor:pointer; }
    .btn:hover { background:#f0f0f0; }
    .btn-active { background:#2d83f7; color:white; border-color:#1b62d6; }
    .controls { display:flex; gap:8px; align-items:center; margin-bottom:8px; }
    .panel { background:#fff; border:1px solid #e0e0e0; padding:10px; border-radius:6px; min-height:80px; box-sizing:border-box; }
    .result-box { font-family:monospace; white-space:nowrap; overflow:auto; padding:6px; }
    .meta { font-size:13px; color:#444; margin-top:6px; }
    .small-muted { font-size:13px; color:#666; }
    .two-horizontal { display:flex; gap:10px; flex-wrap:wrap; align-items:center; }
    .copyBtn { margin-left:8px; }
    table { width:100%; border-collapse:collapse; margin-top:8px; font-size:13px; }
    th, td { border:1px solid #ddd; padding:6px; text-align:center; }
    th { background:#f3f3f3; }
    .hit { color:green; font-weight:bold; }
    .miss { color:red; font-weight:bold; }
    .note { font-size:13px; color:#333; margin-top:8px; }
    code { background:#eef; padding:2px 4px; border-radius:4px; }
  </style>
</head>
<body>
  <h2>Prediksi 3 Digit (4→3) — Pola A–T Eksperimen</h2>
  <div class="note">Masukkan histori 4-digit (pisahkan dengan spasi, koma, newline). Contoh: <code>7642, 9210 3481 5073 ...</code></div>
  <textarea id="historyBox" placeholder="Masukkan histori angka 4 digit..."></textarea>

  <div class="small-muted">Klik pola (A–T) untuk memilih pola aktif — jika ada histori, klik pola langsung menjalankan Prediksi Global.</div>
  <div id="patternRow" class="pattern-row"></div>

  <div class="controls two-horizontal" style="margin-top:8px;">
    <div style="display:flex; gap:8px;">
      <button id="predictBtn" class="btn" onclick="predictGlobal()">Prediksi Global</button>
      <button id="comboBtn" class="btn" onclick="computeCombination()">Kombinasi (2 pola terbaik)</button>
      <button class="btn" onclick="resetAll()">Reset</button>
    </div>
    <div style="margin-left:auto" class="small-muted">Kombinasi hanya menghitung saat tombol ditekan.</div>
  </div>

  <h3>Hasil Prediksi Global (100 angka 3-digit)</h3>
  <div id="resultGlobalPanel" class="panel">
    <div id="resultGlobal" class="result-box">(Belum ada prediksi)</div>
  </div>
  <div style="display:flex; justify-content:space-between; align-items:center; gap:12px; margin-top:6px;">
    <div id="globalMeta" class="meta">Jumlah kandidat: -</div>
    <div>
      <button class="btn" onclick="copyGlobal()">Salin Hasil Global</button>
    </div>
  </div>

  <h3>Akurasi & Tabel Pengujian (100 histori terakhir — per pola aktif)</h3>
  <div id="accuracyBox" class="panel small-muted">(Akurasi belum dihitung)</div>
  <div id="testTableBox" class="panel" style="margin-top:8px; font-family:inherit;"></div>

  <h3>Kotak Kombinasi Pola Terbaik (2 pola)</h3>
  <div id="comboPanel" class="panel">
    <div id="comboInfo" class="small-muted">Tekan tombol <b>Kombinasi</b> untuk menghitung 2 pola terbaik dari A–T (berdasarkan akurasi 100 histori terakhir).</div>
    <div id="comboResults" style="margin-top:10px;"></div>
  </div>

<script>
/* ============================
   STATE & HELPERS
   ============================ */
const PATTERNS = 'ABCDEFGHIJKLMNOPQRST'.split(''); // A..T
let activePattern = null;

// build pattern buttons
const patternRow = document.getElementById('patternRow');
PATTERNS.forEach(p => {
  const btn = document.createElement('button');
  btn.className = 'btn';
  btn.textContent = p;
  btn.id = 'pat_' + p;
  btn.onclick = () => selectPattern(p);
  patternRow.appendChild(btn);
});

function selectPattern(p) {
  activePattern = p;
  PATTERNS.forEach(q => {
    const el = document.getElementById('pat_' + q);
    if (el) el.classList.toggle('btn-active', q === p);
  });
  document.getElementById('accuracyBox').textContent = '(Pola aktif: ' + p + ')';
  const raw = document.getElementById('historyBox').value.trim();
  if (raw) setTimeout(() => { try { predictGlobal(); } catch(e) { console.error(e); } }, 80);
}

function cleanInput(raw) {
  return raw.split(/[^0-9]+/).map(x => x.trim()).filter(x => /^\d{4}$/.test(x));
}
function valid3DigitStr(s) {
  return /^\d{3}$/.test(s);
}
function last3Of4(s4) {
  return s4.slice(-3);
}

/* ============================
   CORE: generatePredictions3D(numbers, pola)
   - returns up to 100 unique 3-digit strings ('000'..'999')
   - uses diverse formulas per pola (A-T) based on 4 digits
   - ensures no item equals last3 of last history
   - if less than 100 unique produced, adds deterministic offsets to fill to 100
   ============================ */
function generatePredictions3D(numbers, pola) {
  if (!PATTERNS.includes(pola)) return [];
  const results = [];
  const seen = new Set();
  // get last3 of last history to avoid equal
  const lastHist = numbers.length ? last3Of4(numbers[numbers.length - 1]) : null;

  // helper to push candidate safely
  const pushCandidate = (num) => {
    // normalize to 0..999
    let n = ((num % 1000) + 1000) % 1000;
    const s = String(n).padStart(3, '0');
    if (!valid3DigitStr(s)) return;
    if (lastHist && s === lastHist) return;
    if (!seen.has(s)) {
      seen.add(s);
      results.push(s);
    }
  };

  // iterate through history and generate candidates
  for (let raw of numbers) {
    // raw is 4-digit string
    const d = raw.split('').map(x => parseInt(x, 10));
    let d1=d[0], d2=d[1], d3=d[2], d4=d[3];

    switch(pola) {
      case 'A':
        // combine (d1+d2+d3) as hundreds,take d4 as units chunk -> build many variants
        pushCandidate((d1 + d2 + d3)*10 + d4);
        pushCandidate((d1*100 + d2*10 + d3) % 1000);
        break;
      case 'B':
        pushCandidate(((d2 + d3 + d4) * Math.max(1,d1)) % 1000);
        pushCandidate((d2*100 + d3*10 + d4) % 1000);
        break;
      case 'C':
        pushCandidate((Math.abs(d1 - d4) * 100 + (d2 + d3) % 100));
        pushCandidate(((d1 + d4)*10 + (Math.abs(d2 - d3))) % 1000);
        break;
      case 'D':
        // rotation: d3 d4 d1
        pushCandidate(d3*100 + d4*10 + d1);
        pushCandidate((d4*100 + d1*10 + d2) % 1000);
        break;
      case 'E':
        pushCandidate((d1*d3 + d2) % 1000);
        pushCandidate((d2*d4 + d1) % 1000);
        break;
      case 'F':
        pushCandidate((d4 * d2) % 1000);
        pushCandidate((d4*100 + d2*10 + ((d1+d3)%10)) % 1000);
        break;
      case 'G':
        pushCandidate(((d1 + d4) * Math.max(1, (d2 - d3))) % 1000);
        pushCandidate(((d1 + d4) * Math.abs(d2 - d3)) % 1000);
        break;
      case 'H':
        pushCandidate(Math.floor((d1 + d2 + d3 + d4)/4) * 11 % 1000);
        pushCandidate(Math.floor((d1 + d2 + d3 + d4)/4) * 7 % 1000);
        break;
      case 'I':
        pushCandidate((d1 * d2 * d3 * (d4 || 1)) % 1000);
        pushCandidate(((d1+1) * (d2+1) * (d3+1) * (d4+1)) % 1000);
        break;
      case 'J':
        pushCandidate(((d1 + d3)*(d1 + d3) + (d2 + d4)) % 1000);
        pushCandidate(((d2 + d4)*(d2 + d4) + (d1 + d3)) % 1000);
        break;
      case 'K':
        pushCandidate(((d2*d2 + d3*d3) ) % 1000);
        pushCandidate(((d2*10 + d3*3) ) % 1000);
        break;
      case 'L':
        // mirror: d4 d3 d2
        pushCandidate(d4*100 + d3*10 + d2);
        pushCandidate(d3*100 + d2*10 + d1);
        break;
      case 'M':
        pushCandidate((d1 + d2 + d3 + d4) % 1000);
        pushCandidate(((d1*3 + d2*2 + d3*1 + d4*4)) % 1000);
        break;
      case 'N':
        pushCandidate((d1*d2 + d3*d4) % 1000);
        pushCandidate(((d1 + d2) * (d3 + d4)) % 1000);
        break;
      case 'O':
        pushCandidate(( (d1 + d3) - (d2 + d4) + 1000) % 1000);
        pushCandidate((Math.abs(d1 - d3)*100 + Math.abs(d2 - d4)) % 1000);
        break;
      case 'P':
        pushCandidate((Math.abs(d1-d2) * 10 + Math.abs(d3-d4)) % 1000);
        pushCandidate((Math.abs(d1-d2)*100 + Math.abs(d3-d4)) % 1000);
        break;
      case 'Q':
        pushCandidate((d1*3 + d2*7 + d3*9 + d4*5) % 1000);
        pushCandidate((d1*11 + d2*13 + d3*17 + d4*19) % 1000);
        break;
      case 'R':
        pushCandidate(((d1 + d4)*(d1 + d4)) % 1000);
        pushCandidate(((d1 + d4)*123) % 1000);
        break;
      case 'S':
        pushCandidate((d1*d1 + d2*d3 + d4) % 1000);
        pushCandidate((d1*d2 + d3*d3 + d4) % 1000);
        break;
      case 'T':
        pushCandidate(((d1 + d2 + d3 + d4)*(d1 + d2 + d3 + d4)) % 1000);
        pushCandidate(((d1 + d2 + d3 + d4)*37) % 1000);
        break;
    }
    if (seen.size >= 100) break;
  } // end for history

  // If not enough candidates, add deterministic offsets based on pola name to fill to 100
  let offsetBase = pola.charCodeAt(0) - 65 + 1; // 1..20
  let i = 0;
  const existing = Array.from(seen);
  // Use existing candidates as seeds; if none, use sequence 0..999 with offset
  while (seen.size < 100) {
    if (results.length === 0) {
      // no seed: generate offset sequence
      pushCandidate(offsetBase * i + (offsetBase + i));
    } else {
      // perturb existing seeds in round-robin
      const seed = existing[i % existing.length] || '000';
      const seedNum = parseInt(seed, 10);
      pushCandidate(seedNum + offsetBase * (i+1));
    }
    i++;
    if (i > 5000) break; // safety
  }

  return results.slice(0, 100);
}

/* ============================
   PREDICT GLOBAL
   ============================ */
function predictGlobal() {
  const raw = document.getElementById('historyBox').value.trim();
  if (!raw) return alert('Masukkan histori terlebih dahulu.');
  const numbers = cleanInput(raw);
  if (!numbers.length) return alert('Tidak ditemukan histori 4-digit valid.');
  if (!activePattern) return alert('Pilih pola (A–T) terlebih dahulu dengan klik salah satu tombol pola.');

  const preds = generatePredictions3D(numbers, activePattern);
  if (!preds.length) {
    document.getElementById('resultGlobal').textContent = '(Tidak ada kandidat valid untuk pola ' + activePattern + ')';
    document.getElementById('globalMeta').textContent = 'Jumlah kandidat: 0';
  } else {
    document.getElementById('resultGlobal').textContent = preds.join('*');
    document.getElementById('globalMeta').textContent = 'Jumlah kandidat: ' + preds.length + ' (Pola ' + activePattern + ')';
  }

  const accText = calculateAccuracyGlobal(numbers, activePattern);
  document.getElementById('accuracyBox').textContent = accText.summary;
  document.getElementById('testTableBox').innerHTML = accText.tableHtml;
}

/* ============================
   ACCURACY EVALUATION (global)
   - uses last 100 historis
   - compares predicted 3-digit set with next's last3
   ============================ */
function calculateAccuracyGlobal(numbers, pola) {
  if (numbers.length < 101) {
    return { summary: "Histori terlalu sedikit untuk evaluasi (butuh >=101).", tableHtml: "" };
  }
  let hits = 0, misses = 0;
  const rows = [];
  const testRange = numbers.slice(-100);
  for (let i = 0; i < testRange.length - 1; i++) {
    const posFromEnd = testRange.length - 1 - i;
    const subsetGlobalEnd = numbers.length - posFromEnd;
    const subsetGlobal = numbers.slice(0, subsetGlobalEnd);
    const next = testRange[i + 1];
    const nextLast3 = last3Of4(next);
    const predGlobal = generatePredictions3D(subsetGlobal, pola);
    const hit = predGlobal.includes(nextLast3);
    if (hit) hits++; else misses++;
    rows.push(`<tr><td>${nextLast3}</td><td class="${hit ? 'hit' : 'miss'}">${hit ? 'Hit' : 'Miss'}</td></tr>`);
  }
  const acc = ((hits / (hits + misses)) * 100).toFixed(2);
  const summary = `Global: Hit ${hits}, Miss ${misses}, Akurasi ${acc}% (pola ${pola})`;
  const tableHtml = `<table><tr><th>Next (last3)</th><th>Global (${pola})</th></tr>${rows.join('')}</table>`;
  return { summary, tableHtml };
}

/* ============================
   EVALUATE ALL PATTERNS (pick top2)
   - compute acc and longest miss streak for each pola
   - sort: acc desc, missStreak desc, pola asc
   ============================ */
function evaluateAllPatterns(numbers) {
  const patterns = PATTERNS.slice();
  const results = [];
  const testRange = numbers.slice(-100);
  for (let pola of patterns) {
    let hits = 0, misses = 0;
    const history = [];
    for (let i = 0; i < testRange.length - 1; i++) {
      const posFromEnd = testRange.length - 1 - i;
      const subsetGlobalEnd = numbers.length - posFromEnd;
      const subsetGlobal = numbers.slice(0, subsetGlobalEnd);
      const next = testRange[i + 1];
      const nextLast3 = last3Of4(next);
      const predGlobal = generatePredictions3D(subsetGlobal, pola);
      const globalHit = predGlobal.includes(nextLast3);
      history.push(globalHit ? 'H' : 'M');
      if (globalHit) hits++; else misses++;
    }
    const total = hits + misses;
    const acc = total > 0 ? (hits / total) * 100 : 0;
    // compute longest consecutive misses
    let longestMiss = 0, cur = 0;
    for (let s of history) {
      if (s === 'M') { cur++; if (cur > longestMiss) longestMiss = cur; }
      else { cur = 0; }
    }
    results.push({ pola, acc: parseFloat(acc.toFixed(4)), longestMiss });
  }

  results.sort((a,b) => {
    if (b.acc !== a.acc) return b.acc - a.acc;
    if (b.longestMiss !== a.longestMiss) return b.longestMiss - a.longestMiss;
    return a.pola.localeCompare(b.pola);
  });
  return results;
}

/* ============================
   COMPUTE COMBINATION (top2)
   - generate 100 preds per top pola
   - combine preserving order: preds1 then preds2 items not seen before
   - display count and joined '*' list (no spaces)
   ============================ */
function computeCombination() {
  const raw = document.getElementById('historyBox').value.trim();
  if (!raw) return alert('Masukkan histori terlebih dahulu.');
  const numbers = cleanInput(raw);
  if (!numbers.length) return alert('Tidak ditemukan histori 4-digit valid.');

  const evals = evaluateAllPatterns(numbers);
  if (!evals || evals.length < 2) return alert('Gagal mengevaluasi pola.');

  const top1 = evals[0], top2 = evals[1];
  const preds1 = generatePredictions3D(numbers, top1.pola);
  const preds2 = generatePredictions3D(numbers, top2.pola);

  const combined = [];
  const seen = new Set();
  for (let x of preds1) { if (!seen.has(x)) { combined.push(x); seen.add(x); } }
  for (let x of preds2) { if (!seen.has(x)) { combined.push(x); seen.add(x); } }

  const html = `
    <div>
      <div><b>Pola Terbaik 1:</b> ${top1.pola} — Akurasi ${(top1.acc).toFixed(2)}% — MissStreak ${top1.longestMiss}</div>
      <div><b>Pola Terbaik 2:</b> ${top2.pola} — Akurasi ${(top2.acc).toFixed(2)}% — MissStreak ${top2.longestMiss}</div>
      <div class="meta" style="margin-top:8px;"><b>Total angka gabungan (unik):</b> ${combined.length}</div>
      <div id="comboList" class="result-box" style="margin-top:8px; white-space:nowrap;">${combined.length ? combined.join('*') : '(Kosong)'}</div>
      <div style="margin-top:8px;"><button class="btn" onclick="copyCombo()">Salin Hasil Kombinasi</button></div>
    </div>
  `;
  document.getElementById('comboInfo').textContent = '';
  document.getElementById('comboResults').innerHTML = html;
}

/* ============================
   COPY FUNCTIONS
   ============================ */
function copyGlobal() {
  const el = document.getElementById('resultGlobal');
  const text = el.innerText || el.textContent || '';
  if (!text.trim()) return alert('Tidak ada hasil untuk disalin.');
  navigator.clipboard.writeText(text).then(() => {
    alert('Hasil prediksi global disalin ke clipboard.');
  }).catch(err => {
    alert('Gagal menyalin: ' + err);
  });
}
function copyCombo() {
  const listDiv = document.getElementById('comboList');
  if (!listDiv) return alert('Tidak ada hasil kombinasi untuk disalin.');
  const text = listDiv.innerText || listDiv.textContent || '';
  if (!text.trim()) return alert('Tidak ada hasil kombinasi untuk disalin.');
  navigator.clipboard.writeText(text).then(() => {
    alert('Hasil kombinasi disalin ke clipboard.');
  }).catch(err => {
    alert('Gagal menyalin: ' + err);
  });
}

/* ============================
   RESET
   ============================ */
function resetAll() {
  document.getElementById('historyBox').value = '';
  document.getElementById('resultGlobal').textContent = '(Belum ada prediksi)';
  document.getElementById('globalMeta').textContent = 'Jumlah kandidat: -';
  document.getElementById('accuracyBox').textContent = '(Akurasi belum dihitung)';
  document.getElementById('testTableBox').textContent = '';
  document.getElementById('comboInfo').textContent = 'Tekan tombol Kombinasi untuk menghitung 2 pola terbaik dari A–T (berdasarkan akurasi 100 histori terakhir).';
  document.getElementById('comboResults').innerHTML = '';
  PATTERNS.forEach(q => {
    const el = document.getElementById('pat_' + q);
    if (el) el.classList.remove('btn-active');
  });
  activePattern = null;
}
resetAll();
</script>
</body>
</html>
