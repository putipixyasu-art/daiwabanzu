
<!DOCTYPE html>
<html lang="ja">
<head>
<!-- Playables SDK -->
<script>// Playables SDK v1.0.0
// Game lifecycle bridge: rAF-based game-ready detection + event communication
(function() {
  'use strict';

  // Idempotency: skip if already initialized (e.g., server-side injection
  // followed by client-side inject-javascript via the Bloks webview component).
  if (window.playablesSDK) return;

  var HANDLER_NAME = 'playablesGameEventHandler';
  var ANDROID_BRIDGE_NAME = '_MetaPlayablesBridge';
  var RAF_FRAME_THRESHOLD = 3;

  var gameReadySent = false;
  var firstInteractionSent = false;
  var errorSent = false;
  var frameCount = 0;
  var originalRAF = window.requestAnimationFrame;

  // --- Transport Layer ---

  function hasIOSBridge() {
    return !!(window.webkit &&
              window.webkit.messageHandlers &&
              window.webkit.messageHandlers[HANDLER_NAME]);
  }

  function hasAndroidBridge() {
    return !!(window[ANDROID_BRIDGE_NAME] &&
              typeof window[ANDROID_BRIDGE_NAME].postEvent === 'function');
  }

  function isInIframe() {
    return !!(window.parent && window.parent !== window);
  }

  function sendEvent(eventName, payload) {
    var message = {
      type: eventName,
      payload: payload || {},
      timestamp: Date.now()
    };

    if (hasIOSBridge()) {
      try {
        window.webkit.messageHandlers[HANDLER_NAME].postMessage(message);
      } catch (e) { /* ignore */ }
      return;
    }

    if (hasAndroidBridge()) {
    try {
      var p = payload || {};
      p.__secureToken = window.__fbAndroidBridgeAuthToken || '';
      window[ANDROID_BRIDGE_NAME].postEvent(
        eventName,
        JSON.stringify(p)
      );
    } catch (e) { /* ignore */ }
    return;
  }

    if (isInIframe()) {
      try {
        window.parent.postMessage(message, '*');
      } catch (e) { /* ignore */ }
      return;
    }
  }

  // --- rAF Game-Ready Detection ---

  function onFrame() {
    if (gameReadySent) return;

    frameCount++;
    if (frameCount >= RAF_FRAME_THRESHOLD) {
      gameReadySent = true;
      sendEvent('game_ready', {
        frame_count: frameCount,
        detected_at: Date.now()
      });
      return;
    }

    originalRAF.call(window, onFrame);
  }

  if (originalRAF) {
    window.requestAnimationFrame = function(callback) {
      if (!gameReadySent) {
        return originalRAF.call(window, function(timestamp) {
          frameCount++;
          if (frameCount >= RAF_FRAME_THRESHOLD && !gameReadySent) {
            gameReadySent = true;
            sendEvent('game_ready', {
              frame_count: frameCount,
              detected_at: Date.now()
            });
          }
          callback(timestamp);
        });
      }
      return originalRAF.call(window, callback);
    };
  }

  // --- First User Interaction Detection ---

  function setupFirstInteractionDetection() {
    var events = ['touchstart', 'mousedown', 'keydown'];

    function onFirstInteraction() {
      if (firstInteractionSent) return;
      firstInteractionSent = true;
      sendEvent('user_interaction_start', null);

      for (var i = 0; i < events.length; i++) {
        document.removeEventListener(events[i], onFirstInteraction, true);
      }
    }

    for (var i = 0; i < events.length; i++) {
      document.addEventListener(events[i], onFirstInteraction, true);
    }
  }

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', setupFirstInteractionDetection);
  } else {
    setupFirstInteractionDetection();
  }

  // --- Auto Error Capture ---

  window.addEventListener('error', function(event) {
    if (errorSent) return;
    errorSent = true;
    sendEvent('error', {
      message: event.message || 'Unknown error',
      source: event.filename || '',
      lineno: event.lineno || 0,
      colno: event.colno || 0,
      auto_captured: true
    });
  });

  window.addEventListener('unhandledrejection', function(event) {
    if (errorSent) return;
    errorSent = true;
    var reason = event.reason;
    sendEvent('error', {
      message: (reason instanceof Error) ? reason.message : String(reason),
      type: 'unhandled_promise_rejection',
      auto_captured: true
    });
  });

  // --- Public API ---

  window.playablesSDK = {
    complete: function(score) {
      sendEvent('game_ended', {
        score: score,
        completed: true
      });
    },

    error: function(message) {
      if (errorSent) return;
      errorSent = true;
      sendEvent('error', {
        message: message || 'Unknown error',
        auto_captured: false
      });
    },

    sendEvent: function(eventName, payload) {
      if (!eventName || typeof eventName !== 'string') return;
      sendEvent(eventName, payload);
    }
  };

  // Kick off rAF detection in case no game code calls rAF immediately
  if (originalRAF) {
    originalRAF.call(window, onFrame);
  }
})();</script>
<script>window.Intl=window.Intl||{};Intl.t=function(s){return(Intl._locale&&Intl._locale[s])||s;};</script>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>分電盤結線図作成ツール - panel_text_adjust</title>
<style>
* { box-sizing: border-box; }
body {
  margin: 0;
  padding: 20px;
  background: #f1f5f9;
  font-family: "Yu Gothic UI", "YuGothic", "Hiragino Kaku Gothic ProN", "Meiryo", sans-serif;
  color: #1f2937;
  -webkit-font-smoothing: antialiased;
}
.container { max-width: 1280px; margin: 0 auto; }
#input-section {
  background: white;
  padding: 20px 24px;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
  margin-bottom: 24px;
  border: 1px solid #e2e8f0;
}
#input-section h2 {
  margin: 0 0 16px;
  font-size: 16px;
  color: #92400e;
  background: #fffbeb;
  border-left: 4px solid #f59e0b;
  padding: 8px 12px;
  font-weight: 600;
}
.input-table {
  border-collapse: collapse;
  width: 100%;
  max-width: 980px;
  font-size: 13px;
}
.input-table th {
  width: 105px;
  text-align: left;
  padding: 6px 8px;
  background: #f8fafc;
  border: 1px solid #cbd5e1;
  font-weight: 500;
  color: #334155;
  white-space: nowrap;
}
.input-table td {
  border: 1px solid #cbd5e1;
  padding: 0;
  height: 32px;
}
.input-table td.yellow {
  background: #fff9c4;
  position: relative;
}
.input-table td.yellow:focus-within {
  background: #fff59d;
  outline: 2px solid #facc15;
  outline-offset: -2px;
}
.input-table select,
.input-table input[type="text"],
.input-table input[type="number"] {
  width: 100%;
  height: 100%;
  border: none;
  background: transparent;
  padding: 6px 8px;
  font-size: 14px;
  font-family: inherit;
  outline: none;
}
.inline { display: flex; align-items: center; gap: 6px; padding: 0 4px; height: 100%; }
.inline input { border: 1px solid transparent; border-bottom: 1px dashed #d97706; background: rgba(255,255,255,0.5); height: 26px; text-align: center; }
.inline input:focus { background: white; border: 1px solid #f59e0b; }

.btn {
  background: #2563eb;
  color: white;
  border: none;
  padding: 10px 24px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  margin-top: 14px;
  box-shadow: 0 1px 2px rgba(0,0,0,0.1);
  transition: all .2s;
}
.btn:hover { background: #1d4ed8; transform: translateY(-1px); }

#output-section { display: none; }
.toolbar {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
  flex-wrap: wrap;
}
.toolbar button {
  background: white;
  border: 1px solid #cbd5e1;
  padding: 7px 14px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
  font-family: inherit;
}
.toolbar button:hover { background: #f8fafc; }
.toolbar button.primary { background: #16a34a; color: white; border-color: #16a34a; }
.toolbar button.primary:hover { background: #15803d; }

.diagram-paper {
  background: white;
  border: 2px solid #000;
  box-shadow: 0 6px 20px rgba(0,0,0,0.12);
  padding: 16px;
}
.diagram-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 2px solid #000;
  padding-bottom: 6px;
  margin-bottom: 12px;
  font-size: 20px;
  font-weight: 700;
  letter-spacing: 0.05em;
}
.diagram-body {
  display: grid;
  grid-template-columns: 170px 1fr 230px;
  gap: 12px;
  align-items: start;
}
.panel {
  border: 1.5px solid #000;
  background: white;
  min-height: 420px;
  position: relative;
}
.panel-title {
  position: absolute;
  top: -10px;
  left: 8px;
  background: white;
  padding: 0 6px;
  font-size: 11px;
  font-weight: 600;
}
.left-panel { padding: 8px; }
.center-panel { padding: 8px; }
.right-panel { padding: 0; }

.comment-box { height: 100%; display: flex; flex-direction: column; }
.box-title {
  background: #111827;
  color: white;
  padding: 5px 8px;
  font-size: 12px;
  font-weight: 600;
}
#commentContent {
  flex: 1;
  padding: 8px 10px;
  font-size: 12.5px;
  line-height: 1.65;
  white-space: pre-wrap;
}

.circuit-table-wrap {
  margin-top: 14px;
  border: 2px solid #000;
}
.table-title {
  background: #e5e7eb;
  padding: 4px 8px;
  font-weight: 700;
  border-bottom: 1.5px solid #000;
  font-size: 13px;
}
.circuit-table {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
}
.circuit-table .col { border-right: 1px solid #000; }
.circuit-table .col:last-child { border-right: none; }
.circuit-cell {
  display: flex;
  height: 26px;
  border-bottom: 1px solid #9ca3af;
  font-size: 12px;
}
.circuit-cell:last-child { border-bottom: none; }
.circuit-cell .num {
  width: 26px;
  background: #f3f4f6;
  border-right: 1px solid #9ca3af;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  color: #374151;
  flex-shrink: 0;
}
.circuit-cell .name {
  flex: 1;
  padding: 2px 6px;
  outline: none;
  display: flex;
  align-items: center;
  min-width: 0;
  overflow: hidden;
  white-space: nowrap;
}
.circuit-cell .name:focus {
  background: #fff9c4;
  box-shadow: inset 0 0 0 1px #f59e0b;
}
.circuit-cell .name:empty::before {
  content: "—";
  color: #9ca3af;
}

svg text { font-family: "Yu Gothic", "Meiryo", sans-serif; }

/* Print */
@media print {
  @page {
    size: B5 landscape;
    margin: 0;
  }

  body {
    background: white;
    padding: 0;
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
  }

  html, body {
    width: 100%;
    height: 100%;
    overflow: hidden;
  }

  #input-section, .toolbar {
    display: none !important;
  }

  #output-section {
    display: block !important;
  }
  .diagram-paper {
    width: 247mm;
    height: 172mm;
    margin: 0;
    padding: 5mm;
    box-sizing: border-box;
    overflow: hidden;
    border-width: 1px;
    box-shadow: none;

    transform: scale(0.80);   /* ← 少し小さくする */
    transform-origin: top left;
  }

  .container { max-width: none; }

  .diagram-header {
    font-size: 14px;
    padding-bottom: 2px;
    margin-bottom: 4px;
  }

  .diagram-body {
    gap: 6px;
    grid-template-columns: 130px 1fr 170px;
  }

  .panel { min-height: 300px; }

  .panel-title {
    font-size: 9px;
    top: -7px;
  }

  .circuit-table-wrap {
    margin-top: 4px;
    border-width: 1.2px;
  }

  .table-title {
    padding: 2px 6px;
    font-size: 10px;
  }

  .circuit-cell {
    height: 14px;
    font-size: 8px;
  }

  .circuit-cell .num {
    width: 18px;
  }

  .circuit-table {
    grid-template-columns: repeat(5, 1fr);
  }

  #commentContent {
    font-size: 9px;
    padding: 4px 5px;
    line-height: 1.3;
  }

  svg text {
    font-size: 8.5px !important;
  }
}
@media (max-width: 980px) {
  .diagram-body { grid-template-columns: 1fr; }
  .panel { min-height: auto; }
  .circuit-table { grid-template-columns: repeat(3, 1fr); }
}
@media (max-width: 640px) {
  .circuit-table { grid-template-columns: repeat(2, 1fr); }
  body { padding: 10px; }
}
</style>
</head>
<body>
<div class="container">
  <section id="input-section">
    <h2>○入力画面（※黄色の枠に入力）</h2>
    <table class="input-table">
      <tr>
        <th>供給方式，電圧</th>
        <td class="yellow">
          <select id="voltage">
            <option>単相3線式100/200V</option>
            <option>単相2線式100V</option>
            <option>三相3線式200V</option>
          </select>
        </td>
        <th>契約種類</th>
        <td class="yellow">
          <select id="contract">
            <option>従量電灯B</option>
            <option selected>従量電灯C</option>
            <option>よりそう+スマート</option>
          </select>
        </td>
        <th>契約A</th>
        <td class="yellow">
          <select id="contractA">
            <option>30</option>
            <option>40</option>
            <option>50</option>
            <option>60</option>
            <option>6K</option>
            <option>8K</option>
            <option selected>10K</option>
            <option>12K</option>
          </select>
        </td>
      </tr>
      <tr>
        <th>SB</th>
        <td class="yellow">
          <select id="sbType">
            <option>なし</option>
            <option>あり</option>
          </select>
        </td>
        <th>容量</th>
        <td class="yellow">
          <div class="inline"><select id="sbCap"><option></option><option>30</option><option>40</option><option>50</option><option>60</option><option>75</option></select> A</div>
        </td>
      </tr>
      <tr>
        <th>遮断器１</th>
        <td class="yellow">
          <select id="brk1type">
            <option>ELB</option>
            <option>MCB</option>
            <option>スマートメーターSB</option>
          </select>
        </td>
        <th>容量</th>
        <td class="yellow">
          <div class="inline"><select id="brk1cap"><option>30</option><option>40</option><option selected>50</option><option>60</option><option>75</option></select> A</div>
        </td>
      </tr>
      <tr>
        <th>遮断器２</th>
        <td class="yellow"><input id="brk2type" type="text" placeholder="MCBなど"></td>
        <th>容量</th>
        <td class="yellow">
          <div class="inline"><select id="brk2cap"><option></option><option>30</option><option>40</option><option>50</option><option>60</option><option>75</option></select> A</div>
        </td>
      </tr>
      <tr>
        <th>回路数</th>
        <td class="yellow"><input id="circuitCount" type="number" min="12" max="30" value="18"></td>
        <th>電線</th>
        <td class="yellow">
          <div class="inline">種類 <select id="wireType" style="width:70px"><option selected>CVT</option></select> 太さ <select id="wireSize" style="width:70px"><option selected>8</option><option>14</option><option>22</option><option>38</option><option>60</option></select> mm²</div>
        </td>
      </tr>
      <tr>
        <th>盤名</th>
        <td class="yellow" colspan="5"><input id="banmei" type="text" value=""></td>
      </tr>
    </table>
    <button class="btn" id="generateBtn">図面を生成</button>
  </section>

  <section id="output-section">
    <div class="toolbar">
      <button id="add2" class="primary">+2回路追加</button>
      <button id="saveBtn">保存</button>
      <button id="loadBtn">読込</button>
      <button id="printBtn">印刷 (B5横)</button>
    </div>

    <div class="diagram-paper">
      <div class="diagram-header">
        <div>分電盤結線図</div>
        <div id="header-banmei"></div>
      </div>
      
      <div class="diagram-body">
        <div class="panel left-panel">
          <span class="panel-title">系統図</span>
          <div id="systemDiagram"></div>
        </div>
        
        <div class="panel center-panel">
          <span class="panel-title">単線結線図</span>
          <div id="mainDiagram"></div>
        </div>
        
        <div class="panel right-panel">
          <div class="comment-box">
            <div class="box-title">契約情報</div>
            <div id="commentContent"></div>
          </div>
        </div>
      </div>

      <div class="circuit-table-wrap">
        <div class="table-title">分岐回路 負荷名称一覧（クリックで編集可）</div>
        <div id="circuitTable" class="circuit-table"></div>
      </div>
    </div>
  </section>
</div>

<script>
const $ = id => document.getElementById(id);

let circuits = {
  1:'玄関',2:'リビング',3:'キッチン',4:'洗面所',5:'寝室2 3',6:'寝室1',
  7:'レンジ',8:'食洗器',9:'洗濯機',10:'浴室暖房',11:'セコム',12:'寝室4 AC',
  13:'寝室3 AC',14:'寝室2 AC',15:'寝室1 AC',16:'リビング AC',17:'IHヒーター',18:'',
  19:'',20:'',21:'',22:'',23:'',24:'',25:'',26:'',27:'',28:'',29:'',30:''
};

function getFormData() {
  return {
    voltage: $('voltage').value,
    contract: $('contract').value,
    contractA: $('contractA').value,
    sbType: $('sbType').value,
    sbCap: $('sbCap').value,
    brk1type: $('brk1type').value,
    brk1cap: $('brk1cap').value,
    brk2type: $('brk2type').value,
    brk2cap: $('brk2cap').value,
    circuitCount: parseInt($('circuitCount').value) || 18,
    wireType: $('wireType').value,
    wireSize: $('wireSize').value,
    banmei: $('banmei').value
  };
}

function renderCircuitTable() {
  const table = $('circuitTable');
  table.innerHTML = '';
  for(let col=0; col<5; col++) {
    const colDiv = document.createElement('div');
    colDiv.className = 'col';
    const start = col*6 + 1;
    const end = Math.min(start+5, 30);
    for(let n=start; n<=end; n++) {
      const cell = document.createElement('div');
      cell.className = 'circuit-cell';
      cell.innerHTML = `<span class="num">${n}</span><span class="name" contenteditable="true" data-num="${n}">${circuits[n]||''}</span>`;
      colDiv.appendChild(cell);
    }
    table.appendChild(colDiv);
  }
}

function renderSystemDiagram(d) {
  let y = 20;
  let svg = `<svg viewBox="0 0 150 260" width="100%" xmlns="http://www.w3.org/2000/svg">`;
  // 電力量計
  svg += `<rect x="50" y="${y}" width="50" height="26" fill="white" stroke="black" stroke-width="1.5" rx="2"/>`;
  svg += `<text x="75" y="${y+17}" text-anchor="middle" font-size="12">WH</text>`;
  y += 26;
  svg += `<line x1="75" y1="${y}" x2="75" y2="${y+18}" stroke="black" stroke-width="1.8"/>`;
  y += 18;
  // SB
  if(d.sbType === 'あり') {
    svg += `<rect x="45" y="${y}" width="60" height="26" fill="white" stroke="black" stroke-width="1.5" rx="2"/>`;
    svg += `<text x="75" y="${y+17}" text-anchor="middle" font-size="11">SB ${d.sbCap||''}A</text>`;
  } else {
    svg += `<text x="75" y="${y+14}" text-anchor="middle" font-size="11" fill="#666">SB なし</text>`;
    svg += `<line x1="75" y1="${y}" x2="75" y2="${y+26}" stroke="#999" stroke-dasharray="3 2"/>`;
  }
  y += 26;
  svg += `<line x1="75" y1="${y}" x2="75" y2="${y+18}" stroke="black" stroke-width="1.8"/>`;
  y += 18;
  // 遮断器1
  svg += `<rect x="40" y="${y}" width="70" height="30" fill="white" stroke="black" stroke-width="1.8" rx="3"/>`;
  svg += `<text x="75" y="${y+14}" text-anchor="middle" font-size="11" font-weight="600">${d.brk1type}</text>`;
  svg += `<text x="75" y="${y+26}" text-anchor="middle" font-size="11">${d.brk1cap}A</text>`;
  y += 30;
  svg += `<line x1="75" y1="${y}" x2="75" y2="${y+18}" stroke="black" stroke-width="1.8"/>`;
  y += 18;
  // 遮断器2
  if(d.brk2type) {
    svg += `<rect x="40" y="${y}" width="70" height="26" fill="white" stroke="black" stroke-width="1.5" rx="3"/>`;
    svg += `<text x="75" y="${y+17}" text-anchor="middle" font-size="11">${d.brk2type} ${d.brk2cap||''}A</text>`;
    y += 26;
    svg += `<line x1="75" y1="${y}" x2="75" y2="${y+12}" stroke="black" stroke-width="1.5"/>`;
  }
  svg += `<polygon points="70,${y+12} 80,${y+12} 75,${y+22}" fill="black"/>`;
  svg += `</svg>`;
  $('systemDiagram').innerHTML = svg;
}

function renderMainDiagram(d) {
  const count = d.circuitCount;
  const pairs = Math.ceil(count/2);
  const h = 240; // 少し小さく
  let svg = `<svg viewBox="0 0 400 ${h}" width="100%" xmlns="http://www.w3.org/2000/svg">`;
  
  // 供給方式
  svg += `<text x="200" y="16" text-anchor="middle" font-size="12" font-weight="600">${d.voltage}</text>`;
  const contractLabel = d.contractA.includes('K') ? `${d.contract} ${d.contractA}VA` : `${d.contract} ${d.contractA}A`;
  svg += `<text x="200" y="30" text-anchor="middle" font-size="10" fill="#444">${contractLabel}</text>`;
  
  // 主幹
  let y = 40;
  svg += `<rect x="165" y="${y}" width="70" height="26" fill="white" stroke="black" stroke-width="2" rx="3"/>`;
  svg += `<text x="200" y="${y+17}" text-anchor="middle" font-size="12" font-weight="600">${d.brk1type} ${d.brk1cap}A</text>`;
  y += 26;
  svg += `<line x1="200" y1="${y}" x2="200" y2="${y+8}" stroke="black" stroke-width="2.2"/>`;
  y += 8;
  
  if(d.brk2type) {
    svg += `<rect x="170" y="${y}" width="60" height="22" fill="white" stroke="black" stroke-width="1.6" rx="2"/>`;
    svg += `<text x="200" y="${y+15}" text-anchor="middle" font-size="10">${d.brk2type} ${d.brk2cap||''}A</text>`;
    y += 22;
    svg += `<line x1="200" y1="${y}" x2="200" y2="${y+6}" stroke="black" stroke-width="2"/>`;
    y += 6;
  }
  
  const startY = y + 6;
  svg += `<line x1="200" y1="${startY}" x2="200" y2="${h-10}" stroke="black" stroke-width="2.5"/>`;
  
  // 分岐回路
  const maxPairs = 15; // 30回路まで対応
  const rowHeight = (h - startY - 10) / maxPairs;
  for(let i=1; i<=count; i++) {
    const pair = Math.floor((i-1)/2);
    const yb = startY + pair * rowHeight;
    const isOdd = i % 2 === 1;
    const xEnd = isOdd ? 75 : 325;
    const bx = isOdd ? 55 : 325;
    
    svg += `<line x1="200" y1="${yb}" x2="${xEnd}" y2="${yb}" stroke="black" stroke-width="1.3"/>`;
    svg += `<line x1="${xEnd}" y1="${yb}" x2="${xEnd}" y2="${yb+8}" stroke="black"/>`;
    // ブレーカ記号
    svg += `<rect x="${bx}" y="${yb+8}" width="18" height="12" fill="white" stroke="black" stroke-width="1.2"/>`;
    svg += `<text x="${bx+9}" y="${yb+17}" text-anchor="middle" font-size="9" font-weight="600">${i}</text>`;
    
  }
  
  // 接地等表示
  svg += `<text x="200" y="${h-2}" text-anchor="middle" font-size="9" fill="#555">分岐 ${count}回路</text>`;
  svg += `</svg>`;
  $('mainDiagram').innerHTML = svg;
}

function renderRightComment(d) {
  const contractLabel = d.contractA.includes('K') ? `${d.contractA}VA` : `${d.contractA}A`;
  const html = `
<div><strong>盤名称：</strong>${d.banmei}</div>
<div><strong>供給方式：</strong>${d.voltage}</div>
<div style="margin-top:8px; padding:8px; background:#fff9c4; border:2px solid #d97706; border-radius:4px;">
  <div><strong>契約種類：</strong>${d.contract}</div>
  <div><strong>契約容量：</strong>${contractLabel}</div>
</div>
<div style="margin-top:10px"><strong>主幹：</strong>${d.brk1type} ${d.brk1cap}A</div>
<div><strong>SB：</strong>${d.sbType}${d.sbType==='あり' ? '（'+d.sbCap+'A）' : ''}</div>
<div><strong>電線：</strong>${d.wireType} ${d.wireSize}mm²</div>`;
  $('commentContent').innerHTML = html;
}

function adjustScale() {
  // 印刷時はJSスケールを無効化
  if (window.matchMedia('print').matches) return;

  const count = parseInt($('circuitCount').value) || 18;
  const paper = document.querySelector('.diagram-paper');

  if(count <= 18) {
    paper.style.transform = "scale(1)";
  } else if(count <= 24) {
    paper.style.transform = "scale(0.92)";
  } else {
    paper.style.transform = "scale(0.85)";
  }
  paper.style.transformOrigin = "top center";
}

function generate() {
  const data = getFormData();
  $('output-section').style.display = 'block';
  $('header-banmei').textContent = data.banmei;
  
  // SB容量制御
  $('sbCap').disabled = $('sbType').value === 'なし';
  
  renderCircuitTable();
  renderSystemDiagram(data);
  renderMainDiagram(data);
  renderRightComment(data);

  adjustScale(); // ←追加

  saveData();
  
  // スクロール
  setTimeout(()=>{ $('output-section').scrollIntoView({behavior:'smooth', block:'start'}); }, 100);
}

function saveData() {
  const data = getFormData();
  data.circuits = circuits;
  localStorage.setItem('panelTextAdjust_v3', JSON.stringify(data));
}

function loadData() {
  const raw = localStorage.getItem('panelTextAdjust_v3');
  if(!raw) return false;
  try {
    const data = JSON.parse(raw);
    $('voltage').value = data.voltage || '単相3線式100/200V';
    $('sbType').value = data.sbType || 'なし';
    $('sbCap').value = data.sbCap || '';
    $('brk1type').value = data.brk1type || 'ELB';
    $('brk1cap').value = data.brk1cap || '50';
    $('brk2type').value = data.brk2type || '';
    $('brk2cap').value = data.brk2cap || '';
    $('circuitCount').value = data.circuitCount || 18;
    $('wireType').value = data.wireType || 'CVT';
    $('wireSize').value = data.wireSize || '14';
    $('contract').value = data.contract || '従量電灯C';
    $('contractA').value = data.contractA || '10K';
    $('banmei').value = data.banmei || '';
    if(data.circuits) circuits = {...circuits, ...data.circuits};
    renderCircuitTable();
    return true;
  } catch(e) { console.error(e); return false; }
}

// Events
$('generateBtn').onclick = generate;
$('add2').onclick = () => {
  let c = parseInt($('circuitCount').value) || 18;
  c = Math.min(30, c + 2);
  $('circuitCount').value = c;
  generate();
};
$('saveBtn').onclick = () => { saveData(); alert('保存しました'); };
$('loadBtn').onclick = () => { if(loadData()){ alert('読込ました。図面を生成してください。'); } else { alert('保存データがありません'); } };
$('printBtn').onclick = () => { saveData(); window.print(); };
$('sbType').onchange = e => { $('sbCap').disabled = e.target.value === 'なし'; };

$('circuitTable').addEventListener('input', e => {
  if(e.target.classList.contains('name')) {
    const num = e.target.dataset.num;
    circuits[num] = e.target.textContent.trim();
    const label = document.getElementById(`circ-label-${num}`);
    if(label) label.textContent = circuits[num];
    saveData();
  }
});

// Init
document.addEventListener('DOMContentLoaded', () => {
  renderCircuitTable();
  if(!loadData()) {
    $('sbCap').disabled = true;
  } else {
    $('sbCap').disabled = $('sbType').value === 'なし';
  }
});
</script>
</body>
</html>
