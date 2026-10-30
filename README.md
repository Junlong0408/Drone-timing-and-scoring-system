<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>競賽計時與計分系統</title>
<style>
  :root {
    --red: #ef3038;
    --red-dark: #9d151b;
    --blue: #10b9d3;
    --blue-dark: #08758a;
    --green: #08b96b;
    --ink: #111827;
    --panel: #f4f5f7;
    --display: #090b0e;
    --cream: #ffe2a2;
  }
  * { box-sizing: border-box; }
  body {
    margin: 0; min-height: 100vh; padding: 16px;
    font-family: "Microsoft JhengHei", "Noto Sans TC", Arial, sans-serif;
    color: var(--ink);
    background: linear-gradient(135deg, #d9dadd, #f5f5f5 55%, #d7d9dc);
  }
  button, input { font: inherit; }
  button { cursor: pointer; border: 0; transition: transform .08s, filter .15s, opacity .15s; }
  button:active { transform: translateY(2px) scale(.99); filter: brightness(.92); }
  button:disabled { cursor: not-allowed; opacity: .42; }
  .app { width: min(1500px, 100%); margin: 0 auto; }
  .topbar { display:flex; justify-content:space-between; align-items:center; gap:12px; margin:0 4px 8px; }
  .brand { font-weight:900; letter-spacing:.08em; color:#333; font-size:clamp(14px,1.7vw,20px); }
  .top-actions { display:flex; gap:8px; }
  .small-btn { background:#252a32; color:white; padding:9px 14px; border-radius:9px; font-weight:700; }
  .timer-wrap { display:flex; justify-content:center; margin: 4px auto 14px; }
  .timer {
    width:min(920px, 100%); text-align:center; color:#050505;
    font-family: "Arial Black", Impact, sans-serif; font-weight:900;
    font-size:clamp(58px, 11.5vw, 150px); line-height:.98; letter-spacing:.025em;
    font-variant-numeric: tabular-nums;
    text-shadow: 1px 2px 0 rgba(0,0,0,.08);
  }
  .timer.urgent { color:#d71920; animation:pulse 1s infinite; }
  @keyframes pulse { 50% { opacity:.45; } }
  .status-row { display:flex; justify-content:center; margin:0 0 12px; }
  .status { padding:6px 18px; border-radius:999px; background:#e5e7eb; color:#374151; font-size:14px; font-weight:800; }
  .status.live { background:#d1fae5; color:#047857; }
  .status.paused { background:#fee2e2; color:#b91c1c; }
  .board { display:grid; grid-template-columns:minmax(180px,1fr) minmax(360px,1.7fr) minmax(180px,1fr); gap:16px; align-items:stretch; }
  .team { display:flex; flex-direction:column; gap:14px; min-width:0; }
  .team-name { text-align:center; color:white; font-weight:900; font-size:clamp(22px,3vw,34px); padding:10px; border-radius:12px 12px 0 0; letter-spacing:.08em; }
  .red .team-name { background:var(--red-dark); }
  .blue .team-name { background:var(--blue-dark); }
  .foul-card { background:white; border:3px solid #202b50; text-align:center; padding:12px 8px; min-height:120px; display:flex; flex-direction:column; justify-content:center; }
  .foul-label { font-size:clamp(17px,2vw,25px); font-weight:900; }
  .foul-count { font-size:clamp(32px,4vw,46px); font-weight:900; line-height:1.25; font-variant-numeric:tabular-nums; }
  .score-button { border-radius:60px; color:#161616; min-height:108px; font-size:clamp(22px,2.8vw,34px); font-weight:900; box-shadow: inset 0 -4px rgba(0,0,0,.18), 0 3px 7px rgba(0,0,0,.13); }
  .red .score-button { background:var(--red); }
  .blue .score-button { background:var(--blue); }
  .foul-button { background:#080808; color:white; border-radius:10px; min-height:68px; font-size:clamp(22px,2.6vw,32px); font-weight:800; }
  .adjust-row { display:flex; gap:8px; }
  .adjust-row button { flex:1; background:#e5e7eb; color:#1f2937; padding:9px 4px; border-radius:8px; font-weight:800; }
  .center { min-width:0; display:flex; flex-direction:column; gap:12px; }
  .scoreboard { display:grid; grid-template-columns:1fr 1fr; min-height:260px; border:5px solid #272727; background:#222; box-shadow:0 6px 14px rgba(0,0,0,.2); }
  .score-half { display:flex; align-items:center; justify-content:center; min-width:0; padding:10px; position:relative; overflow:hidden; }
  .score-half:first-child { background:linear-gradient(145deg,#c92b2b,#8f1118); border-right:5px solid #222; }
  .score-half:last-child { background:linear-gradient(145deg,#147eaa,#07506e); }
  .score { color:var(--cream); font-family:"Arial Black",Impact,sans-serif; font-weight:900; font-size:clamp(88px,13vw,176px); line-height:1; font-variant-numeric:tabular-nums; text-shadow:3px 4px 0 rgba(0,0,0,.45); }
  .center-controls { display:grid; grid-template-columns:1fr 1fr; gap:10px; }
  .control { min-height:72px; border:2px solid #26334d; font-size:clamp(19px,2.5vw,30px); font-weight:900; color:white; }
  .start { background:var(--green); grid-column:1 / -1; min-height:82px; font-size:clamp(26px,3.2vw,38px); }
  .pause { background:#f5222d; }
  .reset { background:#30343b; }
  .time-setting { display:flex; justify-content:center; align-items:center; gap:8px; color:#333; font-weight:700; flex-wrap:wrap; }
  .time-setting input { width:70px; padding:7px; border:1px solid #9ca3af; border-radius:7px; text-align:center; background:white; }
  .time-setting button { background:#e5e7eb; padding:8px 12px; border-radius:7px; font-weight:800; }
  .footer { text-align:center; margin:14px 0 4px; color:#4b5563; font-size:13px; }
  .keys { margin-top:10px; background:rgba(255,255,255,.68); border-radius:10px; padding:10px 14px; font-size:13px; color:#4b5563; text-align:center; }
  kbd { display:inline-block; background:#fff; border:1px solid #cbd5e1; border-bottom-width:2px; border-radius:4px; padding:1px 6px; margin:0 2px; color:#111827; font-weight:800; }
  @media (max-width:850px) {
    body { padding:10px; }
    .board { grid-template-columns:1fr 1fr; gap:12px; }
    .center { grid-column:1 / -1; grid-row:1; }
    .team { grid-row:2; }
    .scoreboard { min-height:190px; }
    .score { font-size:clamp(80px,20vw,140px); }
    .foul-card { min-height:95px; }
    .score-button { min-height:82px; }
  }
  @media (max-width:480px) {
    .topbar { align-items:flex-start; }
    .top-actions { flex-direction:column; }
    .timer { font-size:clamp(58px,16vw,90px); }
    .board { gap:8px; }
    .team-name { font-size:19px; }
    .foul-label { font-size:15px; }
    .foul-count { font-size:30px; }
    .scoreboard { min-height:145px; border-width:3px; }
    .score-half:first-child { border-right-width:3px; }
    .score { font-size:clamp(62px,19vw,90px); }
    .score-button { min-height:70px; font-size:19px; }
    .foul-button { min-height:56px; font-size:19px; }
    .control { min-height:60px; font-size:18px; }
    .start { min-height:66px; font-size:24px; }
  }
</style>
</head>
<body>
<div class="app">
  <header class="topbar">
    <div class="brand">競賽計時・即時計分系統</div>
    <div class="top-actions">
      <button class="small-btn" id="fullscreenBtn">⛶ 全螢幕</button>
      <button class="small-btn" id="soundBtn" aria-pressed="true">🔔 音效：開</button>
    </div>
  </header>

  <div class="timer-wrap"><div class="timer" id="timer">03:00.000</div></div>
  <div class="status-row"><span class="status" id="status">尚未開始</span></div>

  <main class="board">
    <section class="team red">
      <div class="team-name">紅隊</div>
      <div class="foul-card">
        <div class="foul-label">紅隊違規次數</div>
        <div class="foul-count" id="redFouls">0</div>
      </div>
      <button class="score-button" id="redScoreBtn" disabled>紅隊得分 +1</button>
      <div class="adjust-row">
        <button id="redScoreMinus" disabled>得分 −1</button>
        <button id="redFoulMinus" disabled>違規 −1</button>
      </div>
      <button class="foul-button" id="redFoulBtn" disabled>違規 +1</button>
    </section>

    <section class="center">
      <div class="scoreboard" aria-label="目前比分">
        <div class="score-half"><div class="score" id="redScore">0</div></div>
        <div class="score-half"><div class="score" id="blueScore">0</div></div>
      </div>
      <div class="center-controls">
        <button class="control start" id="startBtn">比賽開始</button>
        <button class="control pause" id="pauseBtn" disabled>時間暫停</button>
        <button class="control reset" id="resetBtn">重新計時</button>
      </div>
      <div class="time-setting">
        <label for="minutes">比賽時間</label>
        <input id="minutes" type="number" min="0" max="99" value="3" aria-label="分鐘">
        <span>分</span>
        <input id="seconds" type="number" min="0" max="59" value="0" aria-label="秒">
        <span>秒</span>
        <button id="applyTimeBtn">套用時間</button>
      </div>
    </section>

    <section class="team blue">
      <div class="team-name">藍隊</div>
      <div class="foul-card">
        <div class="foul-label">藍隊違規次數</div>
        <div class="foul-count" id="blueFouls">0</div>
      </div>
      <button class="score-button" id="blueScoreBtn" disabled>藍隊得分 +1</button>
      <div class="adjust-row">
        <button id="blueScoreMinus" disabled>得分 −1</button>
        <button id="blueFoulMinus" disabled>違規 −1</button>
      </div>
      <button class="foul-button" id="blueFoulBtn" disabled>違規 +1</button>
    </section>
  </main>

  <div class="keys">
    鍵盤快捷鍵：
    <kbd>A</kbd> 紅隊得分　
    <kbd>S</kbd> 紅隊違規　
    <kbd>K</kbd> 藍隊得分　
    <kbd>L</kbd> 藍隊違規　
    <kbd>Space</kbd> 暫停／繼續　
    <kbd>Enter</kbd> 開始
  </div>
  <div class="footer">比賽結束後會保留最終比分；重新開始會清除比分與違規紀錄。</div>
</div>

<script>
(() => {
  const $ = id => document.getElementById(id);
  const state = {
    duration: 180, remaining: 180, red: 0, blue: 0,
    redFouls: 0, blueFouls: 0, status: 'idle',
    deadline: null, sound: true, interval: null
  };
  const scoreIds = ['redScoreBtn','blueScoreBtn','redFoulBtn','blueFoulBtn','redScoreMinus','blueScoreMinus','redFoulMinus','blueFoulMinus'];

  function formatTime(total) {
    const totalMs = Math.max(0, Math.ceil(total * 1000));
    const minutes = Math.floor(totalMs / 60000);
    const seconds = Math.floor((totalMs % 60000) / 1000);
    const milliseconds = totalMs % 1000;
    return `${String(minutes).padStart(2,'0')}:${String(seconds).padStart(2,'0')}.${String(milliseconds).padStart(3,'0')}`;
  }
  function render() {
    $('timer').textContent = formatTime(state.remaining);
    $('timer').classList.toggle('urgent', state.remaining <= 30 && state.remaining > 0);
    $('redScore').textContent = state.red;
    $('blueScore').textContent = state.blue;
    $('redFouls').textContent = state.redFouls;
    $('blueFouls').textContent = state.blueFouls;

    const status = $('status');
    status.className = 'status';
    if (state.status === 'running') { status.textContent = '比賽進行中'; status.classList.add('live'); }
    else if (state.status === 'paused') { status.textContent = '時間暫停中'; status.classList.add('paused'); }
    else if (state.status === 'ended') { status.textContent = '比賽結束'; status.classList.add('paused'); }
    else status.textContent = '尚未開始';

    const active = state.status === 'running' || state.status === 'paused';
    scoreIds.forEach(id => $(id).disabled = !active);
    $('startBtn').disabled = state.status === 'running' || state.status === 'paused' || state.status === 'ended';
    $('startBtn').textContent = state.status === 'ended' ? '比賽已結束' : '比賽開始';
    $('pauseBtn').disabled = !(state.status === 'running' || state.status === 'paused');
    $('pauseBtn').textContent = state.status === 'paused' ? '繼續比賽' : '時間暫停';
    $('minutes').disabled = active;
    $('seconds').disabled = active;
    $('applyTimeBtn').disabled = active;
  }
  function beep() {
    if (!state.sound) return;
    // 時間到提示：持續 3 秒的電子蜂鳴器聲，帶有厚實的雙頻蜂鳴質感。
    try {
      const AudioCtx = window.AudioContext || window.webkitAudioContext;
      if (!AudioCtx) return;
      const ctx = new AudioCtx();
      const startAt = ctx.currentTime + 0.04;
      const master = ctx.createGain();
      master.gain.setValueAtTime(0.0001, startAt);
      master.gain.linearRampToValueAtTime(0.28, startAt + 0.06);
      master.gain.setValueAtTime(0.28, startAt + 2.78);
      master.gain.linearRampToValueAtTime(0.0001, startAt + 3.0);
      master.connect(ctx.destination);

      [390, 470].forEach((frequency, index) => {
        const osc = ctx.createOscillator();
        const toneGain = ctx.createGain();
        osc.type = 'square';
        osc.frequency.setValueAtTime(frequency, startAt);
        toneGain.gain.setValueAtTime(index === 0 ? 0.42 : 0.24, startAt);
        osc.connect(toneGain);
        toneGain.connect(master);
        osc.start(startAt);
        osc.stop(startAt + 3.02);
      });
      window.setTimeout(() => ctx.close().catch(() => {}), 3400);
    } catch(e) {
      console.warn('無法播放時間到提示音：', e);
    }
  }
  function stopClock() {
    if (state.interval) clearInterval(state.interval);
    state.interval = null;
  }
  function tick() {
    if (state.status !== 'running') return;
    state.remaining = Math.max(0, (state.deadline - performance.now()) / 1000);
    if (state.remaining <= 0) {
      state.remaining = 0; state.status = 'ended'; stopClock(); beep();
    }
    render();
  }
  function start() {
    if (state.status !== 'idle') return;
    if (state.remaining <= 0) { state.status = 'ended'; render(); return; }
    state.status = 'running';
    state.deadline = performance.now() + state.remaining * 1000;
    stopClock(); state.interval = setInterval(tick, 10);
    render();
  }
  function togglePause() {
    if (state.status === 'running') {
      tick();
      if (state.status !== 'running') return;
      state.remaining = Math.max(0, (state.deadline - performance.now()) / 1000);
      state.status = 'paused'; stopClock();
    } else if (state.status === 'paused') {
      state.status = 'running';
      state.deadline = performance.now() + state.remaining * 1000;
      state.interval = setInterval(tick, 10);
    }
    render();
  }
  function reset() {
    // 直接重新計時，不再顯示確認彈窗。
    stopClock();
    state.remaining = state.duration;
    state.red = state.blue = state.redFouls = state.blueFouls = 0;
    state.status = 'idle'; state.deadline = null;
    render();
  }
  function applyTime() {
    const m = Math.max(0, Math.min(99, Number($('minutes').value) || 0));
    const s = Math.max(0, Math.min(59, Number($('seconds').value) || 0));
    const total = m * 60 + s;
    if (total <= 0) { alert('請設定大於 0 秒的比賽時間。'); return; }
    state.duration = total; state.remaining = total; render();
  }
  function change(key, delta) {
    if (state.status !== 'running' && state.status !== 'paused') return;
    state[key] = Math.max(0, state[key] + delta);
    render();
  }

  $('startBtn').addEventListener('click', start);
  $('pauseBtn').addEventListener('click', togglePause);
  $('resetBtn').addEventListener('click', reset);
  $('applyTimeBtn').addEventListener('click', applyTime);
  $('redScoreBtn').addEventListener('click', () => change('red', 1));
  $('blueScoreBtn').addEventListener('click', () => change('blue', 1));
  $('redFoulBtn').addEventListener('click', () => change('redFouls', 1));
  $('blueFoulBtn').addEventListener('click', () => change('blueFouls', 1));
  $('redScoreMinus').addEventListener('click', () => change('red', -1));
  $('blueScoreMinus').addEventListener('click', () => change('blue', -1));
  $('redFoulMinus').addEventListener('click', () => change('redFouls', -1));
  $('blueFoulMinus').addEventListener('click', () => change('blueFouls', -1));
  $('fullscreenBtn').addEventListener('click', async () => {
    try {
      if (!document.fullscreenElement) await document.documentElement.requestFullscreen();
      else await document.exitFullscreen();
    } catch(e) { alert('目前瀏覽器不支援全螢幕功能。'); }
  });
  $('soundBtn').addEventListener('click', () => {
    state.sound = !state.sound;
    $('soundBtn').textContent = state.sound ? '🔔 音效：開' : '🔔 音效：關';
    $('soundBtn').setAttribute('aria-pressed', String(state.sound));
  });
  document.addEventListener('keydown', e => {
    if (['INPUT','TEXTAREA'].includes(document.activeElement.tagName)) return;
    if (e.code === 'Space') { e.preventDefault(); togglePause(); return; }
    if (e.key === 'Enter') { if (state.status === 'idle') start(); return; }
    const key = e.key.toLowerCase();
    const map = {a: ['red',1], s:['redFouls',1], k:['blue',1], l:['blueFouls',1]};
    if (map[key]) change(map[key][0], map[key][1]);
  });
  render();
})();
</script>
</body>
</html>
