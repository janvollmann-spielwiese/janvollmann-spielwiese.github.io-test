<!doctype html>
<html lang="de">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>QR-Quiz</title>
<script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
<style>
  :root {
    --bg: #14123a;
    --bg2: #1e1b52;
    --card: #262262;
    --text: #f4f3ff;
    --muted: #b3b0dd;
    --accent: #7c6cff;
    --good: #30a46c;
    --bad: #e5484d;
    --warn: #f5a623;
    --radius: 16px;
  }
  * { box-sizing: border-box; }
  html, body { margin: 0; min-height: 100%; }
  body {
    background: radial-gradient(circle at 20% 0%, var(--bg2), var(--bg) 60%) fixed;
    background-color: var(--bg);
    color: var(--text);
    font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
    line-height: 1.4;
  }
  .wrap { max-width: 1100px; margin: 0 auto; padding: 24px 16px 48px; }
  .narrow { max-width: 640px; }
  .center { text-align: center; }
  h1 { font-size: clamp(1.6rem, 4vw, 2.6rem); margin: .2em 0 .4em; }
  h2 { font-size: 1.25rem; margin: 0 0 12px; display: flex; align-items: center; gap: 8px; }
  .muted { color: var(--muted); }
  .small { font-size: .85rem; color: var(--muted); word-break: break-all; }
  .card { background: var(--card); border-radius: var(--radius); padding: 20px; box-shadow: 0 10px 30px rgba(0,0,0,.25); }
  .pill { display: inline-block; background: rgba(255,255,255,.12); padding: 4px 12px; border-radius: 999px; font-size: .95rem; font-weight: 600; }
  textarea, input {
    width: 100%; font: inherit; color: var(--text); background: rgba(0,0,0,.25);
    border: 2px solid rgba(255,255,255,.15); border-radius: 12px; padding: 12px;
  }
  textarea { font-family: ui-monospace, Consolas, monospace; font-size: .95rem; resize: vertical; }
  input:focus, textarea:focus { outline: none; border-color: var(--accent); }
  label { display: block; margin: 14px 0; color: var(--muted); }
  label input { margin-top: 6px; max-width: 140px; display: block; }
  .btn {
    font: inherit; font-weight: 700; border: 0; border-radius: 12px; padding: 12px 22px;
    cursor: pointer; color: #fff; background: rgba(255,255,255,.14); transition: transform .08s, filter .15s;
  }
  .btn:hover { filter: brightness(1.12); }
  .btn:active { transform: scale(.97); }
  .btn:disabled { opacity: .45; cursor: not-allowed; }
  .btn.primary { background: var(--accent); }
  .btn.big { font-size: 1.2rem; padding: 16px 28px; width: 100%; margin-top: 12px; }
  .error { color: #ff9b9e; margin: 10px 0; min-height: 1em; }
  .warn { margin-top: 16px; background: rgba(245,166,35,.15); border: 1px solid rgba(245,166,35,.5); border-radius: 12px; padding: 12px; font-size: .9rem; text-align: left; }
  .warn input { margin-top: 8px; }

  /* Lobby */
  .lobby { display: grid; grid-template-columns: minmax(0, 1fr) minmax(0, 1fr); gap: 20px; }
  .qrcard { text-align: center; }
  #qr { display: inline-block; background: #fff; padding: 14px; border-radius: 14px; margin: 6px 0 10px; }
  #qr img, #qr canvas { display: block; max-width: 100%; height: auto; }
  .qrph { width: 260px; height: 260px; display: grid; place-items: center; color: #555; }
  .code { font-size: 1.3rem; margin-bottom: 6px; }
  .code b { letter-spacing: .15em; font-size: 1.6rem; }
  .plist { list-style: none; padding: 0; margin: 0 0 12px; display: flex; flex-wrap: wrap; gap: 8px; min-height: 40px; }
  .plist li { background: rgba(255,255,255,.12); padding: 6px 12px; border-radius: 999px; animation: pop .3s; }
  .plist li.off { opacity: .4; }
  @keyframes pop { from { transform: scale(.6); opacity: 0; } }

  /* Frage */
  .qhead { display: flex; justify-content: space-between; align-items: center; gap: 12px; flex-wrap: wrap; }
  .timer { font-size: 2.4rem; font-weight: 800; min-width: 2.5ch; text-align: center; }
  .bar { height: 10px; background: rgba(255,255,255,.12); border-radius: 99px; overflow: hidden; margin: 12px 0 20px; }
  .bar > div { height: 100%; width: 100%; background: var(--accent); transition: width .2s linear; }
  .qtext { text-align: center; margin: 24px 0 28px; }
  .opts { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 14px; }
  .opt {
    background: var(--c); border-radius: 14px; padding: 20px; font-size: clamp(1rem, 2.4vw, 1.5rem); font-weight: 700;
    display: flex; align-items: center; gap: 14px; min-height: 84px; position: relative; overflow: hidden;
  }
  .opt .shape { font-size: 1.4em; opacity: .85; }
  .opt .count { margin-left: auto; background: rgba(0,0,0,.25); border-radius: 99px; padding: 2px 12px; }
  .opt.dim { opacity: .3; }
  .opt.right { outline: 4px solid #fff; }
  .opt.right::after { content: "✓"; position: absolute; right: 14px; top: 6px; font-size: 1.2em; }
  .actions { display: flex; justify-content: flex-end; gap: 12px; margin-top: 20px; }

  /* Auflösung + Punktestand */
  .reveal { display: grid; grid-template-columns: minmax(0, 1.1fr) minmax(0, 1fr); gap: 20px; align-items: start; }
  .reveal .opts { grid-template-columns: 1fr; }
  .reveal .opt { min-height: 60px; padding: 14px 18px; }
  .board { list-style: none; padding: 0; margin: 0; counter-reset: r; }
  .board li { display: flex; align-items: center; gap: 12px; padding: 10px 12px; border-radius: 10px; animation: pop .35s both; }
  .board li:nth-child(odd) { background: rgba(255,255,255,.06); }
  .board .rk { width: 2ch; font-weight: 800; text-align: right; }
  .board .nm { flex: 1; min-width: 0; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
  .board .gain { color: #7ee2a8; font-weight: 700; font-size: .9rem; }
  .board .gain.zero { color: var(--muted); }
  .board .mv { font-size: .8rem; width: 3ch; }
  .board .mv.up { color: #7ee2a8; } .board .mv.down { color: #ff9b9e; }
  .board .sc { font-weight: 800; min-width: 5ch; text-align: right; }

  /* Podium */
  .podium { display: flex; justify-content: center; align-items: flex-end; gap: 14px; margin: 30px 0; }
  .pod { text-align: center; width: 30%; max-width: 200px; }
  .pod .blk { border-radius: 12px 12px 0 0; display: grid; place-items: center; font-size: 2.2rem; font-weight: 800; }
  .pod .nm { font-weight: 700; margin-bottom: 6px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
  .pod.p1 .blk { height: 170px; background: #d4a72c; }
  .pod.p2 .blk { height: 120px; background: #9ba1a6; }
  .pod.p3 .blk { height: 90px; background: #a86b3c; }

  /* Spieler */
  .phead { display: flex; justify-content: space-between; align-items: center; padding: 12px 16px; background: rgba(0,0,0,.2); font-weight: 600; }
  .player { padding: 20px 16px 40px; max-width: 640px; margin: 0 auto; }
  .player .opts { grid-template-columns: 1fr; }
  .player .opt { border: 0; color: #fff; font: inherit; font-weight: 700; font-size: 1.15rem; cursor: pointer; text-align: left; width: 100%; }
  .player .opt:disabled { opacity: .4; cursor: default; }
  .player .qtext { font-size: 1.3rem; margin: 10px 0 18px; }
  .big-msg { text-align: center; padding: 40px 10px; }
  .big-msg .ico { font-size: 4.5rem; line-height: 1; }
  .pts { font-size: 2.6rem; font-weight: 800; margin: 10px 0; }
  .banner { border-radius: var(--radius); padding: 28px 16px; text-align: center; margin-bottom: 18px; }
  .banner.good { background: var(--good); } .banner.bad { background: var(--bad); } .banner.none { background: #555a7a; }
  .stats { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
  .stat { background: var(--card); border-radius: 14px; padding: 16px; text-align: center; }
  .stat b { display: block; font-size: 1.8rem; }
  .spinner { width: 46px; height: 46px; border-radius: 50%; border: 5px solid rgba(255,255,255,.2); border-top-color: var(--accent); animation: spin 1s linear infinite; margin: 30px auto 14px; }
  @keyframes spin { to { transform: rotate(360deg); } }

  @media (max-width: 760px) {
    .lobby, .reveal { grid-template-columns: 1fr; }
    .opts { grid-template-columns: 1fr; }
  }
</style>
</head>
<body>
<div id="app"></div>

<script>
const SAMPLE = `Wie heißt die Hauptstadt von Australien?
Sydney
*Canberra
Melbourne
Perth

Wie viele Kontinente gibt es?
5
6
*7
8

Welches Element hat das chemische Symbol "O"?
Gold
*Sauerstoff
Osmium
Ozon

In welchem Jahr fiel die Berliner Mauer?
1987
*1989
1990
1991

Welcher Planet ist der Sonne am nächsten?
Venus
Mars
*Merkur
Erde

Wie viele Beine hat eine Spinne?
6
*8
10
12`;

const $ = s => document.querySelector(s);
const app = $('#app');
const esc = s => String(s).replace(/[&<>"']/g, c => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c]));
const COLORS = ['#e5484d', '#3e63dd', '#d18a0b', '#30a46c', '#8e4ec6', '#0f9b8e'];
const SHAPES = ['▲', '◆', '●', '■', '★', '⬟'];
const PREFIX = 'qrquiz-';
const joinCode = new URLSearchParams(location.search).get('join');

function store(key, value) {
  try {
    if (value === undefined) return localStorage.getItem(key);
    localStorage.setItem(key, value);
  } catch (e) { return null; }
}

function parseQuestions(txt) {
  const blocks = txt.replace(/\r/g, '').split(/\n\s*\n/).map(b => b.trim()).filter(Boolean);
  const qs = [], errors = [];
  blocks.forEach((block, i) => {
    const lines = block.split('\n').map(l => l.trim()).filter(Boolean);
    const options = [];
    let correct = -1;
    lines.slice(1).forEach(line => {
      if (line.startsWith('*')) { correct = options.length; line = line.slice(1).trim(); }
      options.push(line);
    });
    if (options.length < 2 || options.length > 6) errors.push(`Frage ${i + 1}: 2 bis 6 Antworten nötig.`);
    else if (correct < 0) errors.push(`Frage ${i + 1}: Richtige Antwort mit * markieren.`);
    else qs.push({ text: lines[0], options, correct });
  });
  return { qs, errors };
}

function randomCode() {
  const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
  let c = '';
  for (let i = 0; i < 5; i++) c += chars[Math.floor(Math.random() * chars.length)];
  return c;
}

function optionsHtml(q, extra) {
  return q.options.map((o, i) => {
    const e = extra ? extra(i) : { cls: '', tail: '' };
    return `<div class="opt ${e.cls}" style="--c:${COLORS[i]}"><span class="shape">${SHAPES[i]}</span><span>${esc(o)}</span>${e.tail}</div>`;
  }).join('');
}

/* =========================== HOST =========================== */

const H = { peer: null, code: '', players: new Map(), questions: [], dur: 20, qi: -1, phase: 'setup', deadline: 0, tick: null, baseUrl: '' };

function hostSetup() {
  const saved = store('qrquiz-questions') || SAMPLE;
  const dur = store('qrquiz-dur') || 20;
  app.innerHTML = `<div class="wrap narrow">
    <h1>QR-Quiz</h1>
    <p class="muted">Eine Frage pro Block, Blöcke durch eine Leerzeile trennen. Erste Zeile ist die Frage, darunter 2 bis 6 Antworten. Die richtige Antwort mit <b>*</b> markieren.</p>
    <textarea id="qtext" rows="18" spellcheck="false">${esc(saved)}</textarea>
    <label>Sekunden pro Frage <input id="dur" type="number" min="5" max="120" value="${esc(dur)}"></label>
    <div id="err" class="error"></div>
    <button class="btn primary big" id="create">Quiz erstellen</button>
    <p class="small">Punkte: richtige Antwort 500 bis 1000 Punkte, je schneller desto mehr. Falsch oder keine Antwort: 0.</p>
  </div>`;
  $('#create').onclick = () => {
    const { qs, errors } = parseQuestions($('#qtext').value);
    if (errors.length || !qs.length) {
      $('#err').innerHTML = (errors.length ? errors : ['Keine Fragen gefunden.']).map(esc).join('<br>');
      return;
    }
    store('qrquiz-questions', $('#qtext').value);
    H.dur = Math.max(5, Math.min(120, parseInt($('#dur').value, 10) || 20));
    store('qrquiz-dur', String(H.dur));
    H.questions = qs;
    openHostPeer();
  };
}

function openHostPeer() {
  H.code = randomCode();
  app.innerHTML = `<div class="wrap center"><div class="spinner"></div><p>Quiz wird erstellt …</p></div>`;
  const peer = new Peer(PREFIX + H.code.toLowerCase());
  peer.on('open', () => { H.peer = peer; H.phase = 'lobby'; hostLobby(); });
  peer.on('connection', conn => {
    conn.on('data', d => hostOnData(conn, d));
    conn.on('close', () => hostOnClose(conn));
  });
  peer.on('disconnected', () => { if (!peer.destroyed) try { peer.reconnect(); } catch (e) {} });
  peer.on('error', err => {
    if (err.type === 'unavailable-id' && !H.peer) { peer.destroy(); openHostPeer(); return; }
    if (!H.peer) {
      app.innerHTML = `<div class="wrap narrow center"><h1>Keine Verbindung</h1>
        <p class="muted">Der Verbindungsdienst ist nicht erreichbar (${esc(err.type)}). Internetverbindung prüfen.</p>
        <button class="btn primary" onclick="location.reload()">Neu laden</button></div>`;
    } else console.warn('PeerJS:', err);
  });
}

function joinUrl() { return H.baseUrl + '?join=' + H.code; }

function hostLobby() {
  const isFile = location.protocol === 'file:';
  if (!H.baseUrl) H.baseUrl = isFile ? (store('qrquiz-base') || '') : location.origin + location.pathname;
  app.innerHTML = `<div class="wrap">
    <h1>QR-Quiz</h1>
    <div class="lobby">
      <div class="card qrcard">
        <h2 style="justify-content:center">Scannen und mitspielen</h2>
        <div id="qr"></div>
        <div class="code">Code: <b>${H.code}</b></div>
        <a id="jlink" class="small" target="_blank" rel="noopener"></a>
        ${isFile ? `<div class="warn">Diese Datei ist lokal geöffnet. Handys können nur beitreten, wenn die Seite online liegt (z. B. GitHub Pages, Netlify). Adresse der gehosteten Seite:
          <input id="base" placeholder="https://…/quiz.html" value="${esc(H.baseUrl)}"></div>` : ''}
      </div>
      <div class="card">
        <h2>Teilnehmer <span id="pcount" class="pill">0</span></h2>
        <ul id="plist" class="plist"></ul>
        <p class="muted">${H.questions.length} Fragen · ${H.dur} Sekunden pro Frage</p>
        <button class="btn primary big" id="start" disabled>Quiz starten</button>
      </div>
    </div>
  </div>`;
  drawQR();
  const base = $('#base');
  if (base) base.oninput = () => { H.baseUrl = base.value.trim().split('?')[0]; store('qrquiz-base', H.baseUrl); drawQR(); };
  $('#start').onclick = nextQuestion;
  renderPlayers();
}

function drawQR() {
  const el = $('#qr'), link = $('#jlink');
  el.innerHTML = '';
  if (!H.baseUrl) { el.innerHTML = '<div class="qrph">Adresse unten eintragen</div>'; link.textContent = ''; return; }
  new QRCode(el, { text: joinUrl(), width: 260, height: 260, correctLevel: QRCode.CorrectLevel.M });
  link.href = joinUrl();
  link.textContent = joinUrl();
}

function renderPlayers() {
  if (H.phase === 'lobby') {
    const list = $('#plist');
    if (!list) return;
    const ps = [...H.players.values()];
    list.innerHTML = ps.length
      ? ps.map(p => `<li class="${p.online ? '' : 'off'}">${esc(p.name)}</li>`).join('')
      : '<li class="off">Noch niemand da …</li>';
    $('#pcount').textContent = ps.length;
    $('#start').disabled = ps.length === 0;
  } else if (H.phase === 'question') updateAnswered();
}

function send(p, msg) {
  try { if (p.conn && p.conn.open) p.conn.send(msg); } catch (e) {}
}

function hostOnData(conn, d) {
  if (!d || typeof d !== 'object') return;
  if (d.type === 'join' && typeof d.pid === 'string') {
    const name = String(d.name || '').trim().slice(0, 20) || 'Spieler';
    let p = H.players.get(d.pid);
    if (p) Object.assign(p, { conn, name, online: true });
    else { p = { pid: d.pid, name, score: 0, answer: null, gained: 0, prevRank: 0, conn, online: true }; H.players.set(d.pid, p); }
    conn.pid = d.pid;
    send(p, { type: 'welcome', name: p.name, score: p.score });
    syncPlayer(p);
    renderPlayers();
  } else if (d.type === 'answer') {
    const p = H.players.get(conn.pid);
    if (!p || H.phase !== 'question' || d.q !== H.qi || p.answer !== null) return;
    const q = H.questions[H.qi];
    const c = Number(d.choice);
    if (!(c >= 0 && c < q.options.length)) return;
    const left = Math.max(0, H.deadline - Date.now());
    p.answer = c;
    p.gained = c === q.correct ? Math.round(500 + 500 * left / (H.dur * 1000)) : 0;
    send(p, { type: 'answered' });
    updateAnswered();
    if ([...H.players.values()].every(x => x.answer !== null || !x.online)) reveal();
  }
}

function hostOnClose(conn) {
  const p = H.players.get(conn.pid);
  if (p && p.conn === conn) { p.online = false; renderPlayers(); }
}

function syncPlayer(p) {
  if (H.phase === 'question') { sendQuestion(p); if (p.answer !== null) send(p, { type: 'answered' }); }
  else if (H.phase === 'reveal') sendReveal(p);
  else if (H.phase === 'end') sendEnd(p);
  else send(p, { type: 'lobby' });
}

function sendQuestion(p) {
  const q = H.questions[H.qi];
  send(p, { type: 'question', q: H.qi, total: H.questions.length, text: q.text, options: q.options, left: H.deadline - Date.now(), dur: H.dur * 1000 });
}

function nextQuestion() {
  H.qi++;
  if (H.qi >= H.questions.length) return endGame();
  H.phase = 'question';
  H.players.forEach(p => { p.answer = null; p.gained = 0; });
  H.deadline = Date.now() + H.dur * 1000;
  H.players.forEach(sendQuestion);
  hostQuestion();
  clearInterval(H.tick);
  H.tick = setInterval(() => {
    const left = H.deadline - Date.now();
    updateTimer(left);
    if (left <= 0) reveal();
  }, 200);
}

function hostQuestion() {
  const q = H.questions[H.qi];
  app.innerHTML = `<div class="wrap">
    <div class="qhead">
      <span class="pill">Frage ${H.qi + 1} / ${H.questions.length}</span>
      <span class="timer" id="timer">${H.dur}</span>
      <span class="pill" id="acount"></span>
    </div>
    <div class="bar"><div id="tbar"></div></div>
    <h1 class="qtext">${esc(q.text)}</h1>
    <div class="opts">${optionsHtml(q)}</div>
    <div class="actions"><button class="btn" id="skip">Jetzt auflösen</button></div>
  </div>`;
  $('#skip').onclick = reveal;
  updateAnswered();
}

function updateTimer(left) {
  const t = $('#timer'), bar = $('#tbar');
  if (t) t.textContent = Math.max(0, Math.ceil(left / 1000));
  if (bar) bar.style.width = Math.max(0, left / (H.dur * 1000) * 100) + '%';
}

function updateAnswered() {
  const el = $('#acount');
  if (!el) return;
  const ps = [...H.players.values()];
  el.textContent = `${ps.filter(p => p.answer !== null).length} / ${ps.length} Antworten`;
}

function ranked() {
  return [...H.players.values()].sort((a, b) => b.score - a.score || a.name.localeCompare(b.name));
}
function rankOf(p) {
  return 1 + [...H.players.values()].filter(x => x.score > p.score).length;
}

function reveal() {
  if (H.phase !== 'question') return;
  clearInterval(H.tick);
  H.players.forEach(p => { p.prevRank = rankOf(p); });
  H.players.forEach(p => { p.score += p.gained; });
  H.phase = 'reveal';
  H.players.forEach(sendReveal);
  hostReveal();
}

function sendReveal(p) {
  const q = H.questions[H.qi];
  send(p, {
    type: 'reveal', q: H.qi, correct: q.correct, correctText: q.options[q.correct], choice: p.answer,
    gained: p.gained, score: p.score, rank: rankOf(p), of: H.players.size
  });
}

function hostReveal() {
  const q = H.questions[H.qi];
  const counts = q.options.map((_, i) => [...H.players.values()].filter(p => p.answer === i).length);
  const last = H.qi === H.questions.length - 1;
  const board = ranked().slice(0, 10).map((p, i) => {
    const r = rankOf(p), mv = p.prevRank - r;
    const mvHtml = H.qi === 0 || mv === 0 ? '<span class="mv"></span>'
      : `<span class="mv ${mv > 0 ? 'up' : 'down'}">${mv > 0 ? '▲' : '▼'}${Math.abs(mv)}</span>`;
    return `<li style="animation-delay:${i * 60}ms"><span class="rk">${r}.</span>${mvHtml}<span class="nm">${esc(p.name)}</span>
      <span class="gain ${p.gained ? '' : 'zero'}">+${p.gained}</span><span class="sc">${p.score}</span></li>`;
  }).join('');
  app.innerHTML = `<div class="wrap">
    <div class="qhead"><span class="pill">Frage ${H.qi + 1} / ${H.questions.length}</span></div>
    <h1 class="qtext">${esc(q.text)}</h1>
    <div class="reveal">
      <div class="opts">${optionsHtml(q, i => ({ cls: i === q.correct ? 'right' : 'dim', tail: `<span class="count">${counts[i]}</span>` }))}</div>
      <div class="card">
        <h2>Punktestand</h2>
        <ol class="board">${board || '<li class="muted">Keine Teilnehmer</li>'}</ol>
        ${H.players.size > 10 ? `<p class="small">Top 10 von ${H.players.size}</p>` : ''}
      </div>
    </div>
    <div class="actions"><button class="btn primary big" id="next" style="width:auto">${last ? 'Zum Endergebnis' : 'Nächste Frage'}</button></div>
  </div>`;
  $('#next').onclick = nextQuestion;
}

function sendEnd(p) {
  send(p, { type: 'end', score: p.score, rank: rankOf(p), of: H.players.size, top: ranked().slice(0, 3).map(x => ({ name: x.name, score: x.score })) });
}

function endGame() {
  H.phase = 'end';
  clearInterval(H.tick);
  H.players.forEach(sendEnd);
  const r = ranked();
  const pod = (p, n) => p ? `<div class="pod p${n}"><div class="nm">${esc(p.name)}</div><div class="small">${p.score} Punkte</div><div class="blk">${n}</div></div>` : '';
  app.innerHTML = `<div class="wrap">
    <h1 class="center">Endergebnis</h1>
    <div class="podium">${pod(r[1], 2)}${pod(r[0], 1)}${pod(r[2], 3)}</div>
    <div class="card narrow" style="margin:0 auto">
      <ol class="board">${r.map((p, i) => `<li style="animation-delay:${i * 50}ms"><span class="rk">${rankOf(p)}.</span><span class="nm">${esc(p.name)}</span><span class="sc">${p.score}</span></li>`).join('')}</ol>
    </div>
    <div class="actions" style="justify-content:center"><button class="btn primary" onclick="location.reload()">Neues Quiz</button></div>
  </div>`;
}

/* ========================== SPIELER ========================== */

const P = { peer: null, conn: null, pid: '', name: '', score: 0, state: '', q: -1, tick: null };

function playerStart() {
  P.pid = store('qrquiz-pid') || ('p' + Math.random().toString(36).slice(2, 10) + Date.now().toString(36));
  store('qrquiz-pid', P.pid);
  app.innerHTML = `<div class="wrap narrow center">
    <h1>QR-Quiz</h1>
    <p class="muted">Quiz-Code <b>${esc(joinCode)}</b></p>
    <input id="name" maxlength="20" placeholder="Dein Name" autocomplete="nickname" value="${esc(store('qrquiz-name') || '')}">
    <button class="btn primary big" id="join">Mitspielen</button>
    <div id="err" class="error"></div>
  </div>`;
  const go = () => {
    const n = $('#name').value.trim();
    if (!n) { $('#err').textContent = 'Bitte gib einen Namen ein.'; return; }
    P.name = n.slice(0, 20);
    store('qrquiz-name', P.name);
    playerConnect();
  };
  $('#join').onclick = go;
  $('#name').onkeydown = e => { if (e.key === 'Enter') go(); };
  $('#name').focus();
}

function playerScreen(inner) {
  clearInterval(P.tick);
  app.innerHTML = `<div class="phead"><span>${esc(P.name)}</span><span>${P.score} Punkte</span></div><div class="player">${inner}</div>`;
}

function playerConnect() {
  P.state = 'connecting';
  playerScreen(`<div class="big-msg"><div class="spinner"></div><p>Verbinde mit dem Quiz …</p></div>`);
  if (P.peer) try { P.peer.destroy(); } catch (e) {}
  const peer = new Peer();
  P.peer = peer;
  peer.on('open', () => {
    const conn = peer.connect(PREFIX + joinCode.toLowerCase(), { reliable: true });
    P.conn = conn;
    const timeout = setTimeout(() => { if (!conn.open) playerError('Quiz nicht gefunden. Stimmt der Code und ist das Quiz noch geöffnet?'); }, 12000);
    conn.on('open', () => { clearTimeout(timeout); conn.send({ type: 'join', pid: P.pid, name: P.name }); });
    conn.on('data', playerOnData);
    conn.on('close', () => { if (P.peer === peer) playerError('Die Verbindung wurde unterbrochen.'); });
  });
  peer.on('error', err => {
    if (P.peer !== peer) return;
    playerError(err.type === 'peer-unavailable'
      ? 'Quiz nicht gefunden. Stimmt der Code und ist das Quiz noch geöffnet?'
      : 'Verbindungsfehler (' + err.type + ').');
  });
}

function playerError(msg) {
  if (P.state === 'end' || P.state === 'error') return;
  P.state = 'error';
  playerScreen(`<div class="big-msg"><div class="ico">⚠️</div><p>${esc(msg)}</p>
    <button class="btn primary big" id="retry">Erneut verbinden</button></div>`);
  $('#retry').onclick = playerConnect;
}

function playerOnData(d) {
  if (!d || typeof d !== 'object') return;
  if (d.type === 'welcome') { P.name = d.name; P.score = d.score; }
  else if (d.type === 'lobby') {
    P.state = 'lobby';
    playerScreen(`<div class="big-msg"><div class="ico">🎉</div><h1>Du bist dabei!</h1><p class="muted">Warte, bis das Quiz startet …</p></div>`);
  }
  else if (d.type === 'question') playerQuestion(d);
  else if (d.type === 'answered') {
    P.state = 'answered';
    playerScreen(`<div class="big-msg"><div class="spinner"></div><h1>Antwort gesendet</h1><p class="muted">Warte auf die Auflösung …</p></div>`);
  }
  else if (d.type === 'reveal') {
    P.state = 'reveal';
    P.score = d.score;
    const res = d.choice === null ? 'none' : d.choice === d.correct ? 'good' : 'bad';
    const title = { good: '✓ Richtig!', bad: '✗ Leider falsch', none: '⏱ Keine Antwort' }[res];
    playerScreen(`<div class="banner ${res}"><h1 style="margin:0">${title}</h1>
        <div class="pts">+${d.gained} Punkte</div></div>
      <p class="center muted">Richtige Antwort: <b style="color:var(--text)">${esc(d.correctText)}</b></p>
      <div class="stats">
        <div class="stat"><b>${d.score}</b>Punkte gesamt</div>
        <div class="stat"><b>${d.rank}.</b>Platz von ${d.of}</div>
      </div>`);
  }
  else if (d.type === 'end') {
    P.state = 'end';
    P.score = d.score;
    const medal = ['🥇', '🥈', '🥉'][d.rank - 1] || '🏁';
    playerScreen(`<div class="big-msg"><div class="ico">${medal}</div><h1>Platz ${d.rank} von ${d.of}</h1>
        <div class="pts">${d.score} Punkte</div></div>
      <div class="card"><h2>Top 3</h2><ol class="board">${d.top.map((t, i) =>
        `<li><span class="rk">${i + 1}.</span><span class="nm">${esc(t.name)}</span><span class="sc">${t.score}</span></li>`).join('')}</ol></div>`);
  }
}

function playerQuestion(d) {
  P.state = 'question';
  P.q = d.q;
  const deadline = Date.now() + d.left;
  playerScreen(`<div class="qhead"><span class="pill">Frage ${d.q + 1} / ${d.total}</span><span class="timer" id="ptimer"></span></div>
    <div class="bar"><div id="pbar"></div></div>
    <div class="qtext">${esc(d.text)}</div>
    <div class="opts">${d.options.map((o, i) =>
      `<button class="opt" style="--c:${COLORS[i]}" data-i="${i}"><span class="shape">${SHAPES[i]}</span><span>${esc(o)}</span></button>`).join('')}</div>`);
  app.querySelectorAll('.opt').forEach(b => b.onclick = () => {
    app.querySelectorAll('.opt').forEach(x => x.disabled = true);
    try { P.conn.send({ type: 'answer', q: P.q, choice: Number(b.dataset.i) }); } catch (e) {}
  });
  const tickFn = () => {
    const left = deadline - Date.now();
    const t = $('#ptimer'), bar = $('#pbar');
    if (t) t.textContent = Math.max(0, Math.ceil(left / 1000));
    if (bar) bar.style.width = Math.max(0, left / d.dur * 100) + '%';
    if (left <= 0) {
      clearInterval(P.tick);
      app.querySelectorAll('.opt').forEach(x => x.disabled = true);
    }
  };
  tickFn();
  P.tick = setInterval(tickFn, 200);
}

/* =========================== START =========================== */

if (typeof Peer === 'undefined') {
  app.innerHTML = `<div class="wrap narrow center"><h1>Bibliothek nicht geladen</h1><p class="muted">Bitte Internetverbindung prüfen und neu laden.</p></div>`;
} else if (joinCode) playerStart();
else hostSetup();
</script>
</body>
</html>
