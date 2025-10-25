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
const PATTERNS = 'ABCDEFGHIJKLMNOPQRST'.split('');
let activePattern = null;

// === Build pattern buttons ===
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
function last3Of4(s4) { return s4.slice(-3); }
function valid3DigitStr(s) { return /^\d{3}$/.test(s); }

// === FILTER RULES ===
function isTriple(num3) {
  return num3[0] === num3[1] && num3[1] === num3[2];
}
function hasForbiddenDouble(num3, lastHist3) {
  // if lastHist3 has 2 consecutive same digits, forbid same pattern position
  if (lastHist3[0] === lastHist3[1] && num3[0] === num3[1]) return true;
  if (lastHist3[1] === lastHist3[2] && num3[1] === num3[2]) return true;
  return false;
}
function forbiddenPositionsFromLast2(histArr) {
  const forb = {0:null,1:null,2:null};
  if (histArr.length < 2) return forb;
  const last2 = histArr.slice(-2).map(x => last3Of4(x));
  for (let pos=0; pos<3; pos++) {
    const a = last2[0][pos], b = last2[1][pos];
    if (a === b) forb[pos] = a;
  }
  return forb;
}
function forbiddenLast30(histArr) {
  return histArr.slice(-30).map(x => last3Of4(x));
}

// === CORE: generatePredictions3D ===
function generatePredictions3D(numbers, pola) {
  if (!PATTERNS.includes(pola)) return [];
  const results = [];
  const seen = new Set();
  const lastHist = numbers.length ? last3Of4(numbers[numbers.length - 1]) : null;
  const forbLast30 = forbiddenLast30(numbers);
  const forbPos = forbiddenPositionsFromLast2(numbers);

  const pushCandidate = (num) => {
    let n = ((num % 1000) + 1000) % 1000;
    const s = String(n).padStart(3, '0');
    if (!valid3DigitStr(s)) return;
    if (isTriple(s)) return;
    if (lastHist && hasForbiddenDouble(s, lastHist)) return;
    if (forbLast30.includes(s)) return;
    // check forbidden positions from last 2
    for (let i=0;i<3;i++) {
      if (forbPos[i] !== null && s[i] === forbPos[i]) return;
    }
    if (!seen.has(s)) { seen.add(s); results.push(s); }
  };

  // === pattern formulae ===
  for (let raw of numbers) {
    const d = raw.split('').map(x => parseInt(x, 10));
    let d1=d[0], d2=d[1], d3=d[2], d4=d[3];
    switch(pola) {
      case 'A': pushCandidate((d1+d2+d3)*10+d4); pushCandidate((d1*100+d2*10+d3)%1000); break;
      case 'B': pushCandidate(((d2+d3+d4)*Math.max(1,d1))%1000); pushCandidate((d2*100+d3*10+d4)%1000); break;
      case 'C': pushCandidate((Math.abs(d1-d4)*100+(d2+d3)%100)); pushCandidate(((d1+d4)*10+Math.abs(d2-d3))%1000); break;
      case 'D': pushCandidate(d3*100+d4*10+d1); pushCandidate((d4*100+d1*10+d2)%1000); break;
      case 'E': pushCandidate((d1*d3+d2)%1000); pushCandidate((d2*d4+d1)%1000); break;
      case 'F': pushCandidate((d4*d2)%1000); pushCandidate((d4*100+d2*10+((d1+d3)%10))%1000); break;
      case 'G': pushCandidate(((d1+d4)*Math.max(1,(d2-d3)))%1000); pushCandidate(((d1+d4)*Math.abs(d2-d3))%1000); break;
      case 'H': pushCandidate(Math.floor((d1+d2+d3+d4)/4)*11%1000); pushCandidate(Math.floor((d1+d2+d3+d4)/4)*7%1000); break;
      case 'I': pushCandidate((d1*d2*d3*(d4||1))%1000); pushCandidate(((d1+1)*(d2+1)*(d3+1)*(d4+1))%1000); break;
      case 'J': pushCandidate(((d1+d3)*(d1+d3)+(d2+d4))%1000); pushCandidate(((d2+d4)*(d2+d4)+(d1+d3))%1000); break;
      case 'K': pushCandidate(((d2*d2+d3*d3))%1000); pushCandidate(((d2*10+d3*3))%1000); break;
      case 'L': pushCandidate(d4*100+d3*10+d2); pushCandidate(d3*100+d2*10+d1); break;
      case 'M': pushCandidate((d1+d2+d3+d4)%1000); pushCandidate(((d1*3+d2*2+d3*1+d4*4))%1000); break;
      case 'N': pushCandidate((d1*d2+d3*d4)%1000); pushCandidate(((d1+d2)*(d3+d4))%1000); break;
      case 'O': pushCandidate(((d1+d3)-(d2+d4)+1000)%1000); pushCandidate((Math.abs(d1-d3)*100+Math.abs(d2-d4))%1000); break;
      case 'P': pushCandidate((Math.abs(d1-d2)*10+Math.abs(d3-d4))%1000); pushCandidate((Math.abs(d1-d2)*100+Math.abs(d3-d4))%1000); break;
      case 'Q': pushCandidate((d1*3+d2*7+d3*9+d4*5)%1000); pushCandidate((d1*11+d2*13+d3*17+d4*19)%1000); break;
      case 'R': pushCandidate(((d1+d4)*(d1+d4))%1000); pushCandidate(((d1+d4)*123)%1000); break;
      case 'S': pushCandidate((d1*d1+d2*d3+d4)%1000); pushCandidate((d1*d2+d3*d3+d4)%1000); break;
      case 'T': pushCandidate(((d1+d2+d3+d4)*(d1+d2+d3+d4))%1000); pushCandidate(((d1+d2+d3+d4)*37)%1000); break;
    }
    if (seen.size >= 100) break;
  }

  // filler
  let offsetBase = pola.charCodeAt(0) - 65 + 1;
  let i = 0;
  const existing = Array.from(seen);
  while (seen.size < 100) {
    const seed = existing.length ? parseInt(existing[i % existing.length],10) : 0;
    pushCandidate(seed + offsetBase * (i+1));
    i++; if (i>5000) break;
  }

  return results.slice(0, 100);
}

// === Predict Global ===
function predictGlobal() {
  const raw = document.getElementById('historyBox').value.trim();
  if (!raw) return alert('Masukkan histori terlebih dahulu.');
  const numbers = cleanInput(raw);
  if (!numbers.length) return alert('Tidak ditemukan histori 4-digit valid.');
  if (!activePattern) return alert('Pilih pola (A–T) terlebih dahulu.');

  const preds = generatePredictions3D(numbers, activePattern);
  document.getElementById('resultGlobal').textContent = preds.join('*');
  document.getElementById('globalMeta').textContent = 'Jumlah kandidat: ' + preds.length + ' (Pola ' + activePattern + ')';

  const accText = calculateAccuracyGlobal(numbers, activePattern);
  document.getElementById('accuracyBox').textContent = accText.summary;
  document.getElementById('testTableBox').innerHTML = accText.tableHtml;
}

// === Akurasi ===
function calculateAccuracyGlobal(numbers, pola) {
  if (numbers.length < 101) return { summary:"Histori terlalu sedikit (butuh >=101).", tableHtml:"" };
  let hits=0, misses=0;
  const rows=[];
  const testRange=numbers.slice(-100);
  for(let i=0;i<testRange.length-1;i++){
    const posFromEnd=testRange.length-1-i;
    const subset=numbers.slice(0,numbers.length-posFromEnd);
    const next=testRange[i+1], next3=last3Of4(next);
    const preds=generatePredictions3D(subset,pola);
    const hit=preds.includes(next3);
    if(hit) hits++; else misses++;
    rows.push(`<tr><td>${next3}</td><td class="${hit?'hit':'miss'}">${hit?'Hit':'Miss'}</td></tr>`);
  }
  const acc=((hits/(hits+misses))*100).toFixed(2);
  const summary=`Global: Hit ${hits}, Miss ${misses}, Akurasi ${acc}% (pola ${pola})`;
  return { summary, tableHtml:`<table><tr><th>Next (last3)</th><th>Global (${pola})</th></tr>${rows.join('')}</table>` };
}

// === Kombinasi Pola Terbaik ===
function evaluateAllPatterns(numbers){
  const res=[];
  const testRange=numbers.slice(-100);
  for(let pola of PATTERNS){
    const history=[];
    let hits=0,miss=0;
    for(let i=0;i<testRange.length-1;i++){
      const posFromEnd=testRange.length-1-i;
      const subset=numbers.slice(0,numbers.length-posFromEnd);
      const next=testRange[i+1];
      const next3=last3Of4(next);
      const preds=generatePredictions3D(subset,pola);
      const hit=preds.includes(next3);
      history.push(hit?'H':'M');
      if(hit) hits++; else miss++;
    }
    const acc=(hits+miss)>0?(hits/(hits+miss))*100:0;
    // new miss streak rule: count last consecutive M until hit
    let streak=0;
    for(let j=history.length-1;j>=0;j--){
      if(history[j]==='M') streak++;
      else break;
    }
    res.push({pola, acc:parseFloat(acc.toFixed(4)), longestMiss:streak});
  }
  res.sort((a,b)=> b.acc!==a.acc ? b.acc-a.acc : b.longestMiss!==a.longestMiss ? b.longestMiss-a.longestMiss : a.pola.localeCompare(b.pola));
  return res;
}

function computeCombination(){
  const raw=document.getElementById('historyBox').value.trim();
  if(!raw) return alert('Masukkan histori terlebih dahulu.');
  const numbers=cleanInput(raw);
  if(!numbers.length) return alert('Tidak ditemukan histori 4-digit valid.');
  const evals=evaluateAllPatterns(numbers);
  if(evals.length<2) return alert('Gagal mengevaluasi pola.');
  const top1=evals[0], top2=evals[1];
  const preds1=generatePredictions3D(numbers,top1.pola);
  const preds2=generatePredictions3D(numbers,top2.pola);
  const seen=new Set(), combined=[];
  for(let x of preds1){ if(!seen.has(x)){combined.push(x);seen.add(x);} }
  for(let x of preds2){ if(!seen.has(x)){combined.push(x);seen.add(x);} }
  const html=`
    <b>Pola Terbaik:</b><br>
    1️⃣ ${top1.pola} — Akurasi ${top1.acc.toFixed(2)}%, Miss streak ${top1.longestMiss}<br>
    2️⃣ ${top2.pola} — Akurasi ${top2.acc.toFixed(2)}%, Miss streak ${top2.longestMiss}<br><br>
    <b>Gabungan Prediksi (${combined.length} angka):</b><br>
    <div class='result-box'>${combined.join('*')}</div>
    <button class='btn copyBtn' onclick='copyCombo()'>Salin Gabungan</button>`;
  document.getElementById('comboResults').innerHTML=html;
  document.getElementById('comboInfo').textContent='2 Pola terbaik terpilih berdasarkan akurasi & miss streak.';
}

function resetAll(){
  document.getElementById('historyBox').value='';
  document.getElementById('resultGlobal').textContent='(Belum ada prediksi)';
  document.getElementById('globalMeta').textContent='Jumlah kandidat: -';
  document.getElementById('accuracyBox').textContent='(Akurasi belum dihitung)';
  document.getElementById('testTableBox').innerHTML='';
  document.getElementById('comboResults').innerHTML='';
  document.getElementById('comboInfo').textContent='Tekan tombol Kombinasi untuk menghitung 2 pola terbaik.';
  PATTERNS.forEach(q=>{
    const el=document.getElementById('pat_'+q);
    if(el) el.classList.remove('btn-active');
  });
  activePattern=null;
}

function copyGlobal(){
  const txt=document.getElementById('resultGlobal').textContent;
  navigator.clipboard.writeText(txt);
  alert('Hasil Global disalin.');
}
function copyCombo(){
  const el=document.querySelector('#comboResults .result-box');
  if(!el) return;
  navigator.clipboard.writeText(el.textContent);
  alert('Gabungan disalin.');
}
</script>
</body>
</html>
