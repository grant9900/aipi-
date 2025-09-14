# aipi-
ai assistant project
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>AI PI Lite — Control Hub</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root { --bg:#0b0e14; --card:#111520; --ink:#e8eefc; --ink2:#9bb0d3; --acc:#6aa0ff; --mut:#1a2030; }
    html,body { margin:0; padding:0; background:var(--bg); color:var(--ink); font-family: system-ui, -apple-system, Segoe UI, Roboto, Inter, Arial, sans-serif; }
    header { display:flex; gap:12px; align-items:center; padding:16px; border-bottom:1px solid #1f2a44; position:sticky; top:0; background:linear-gradient(180deg, #0b0e14 0%, #0b0e14e6 100%); backdrop-filter: blur(6px); z-index:10; }
    h1 { font-size:18px; margin:0; letter-spacing:.5px; }
    .pill { padding:8px 14px; border-radius:999px; background:var(--mut); color:var(--ink2); border:1px solid #243049; cursor:pointer; }
    .pill.primary { background:var(--acc); color:#081226; border-color:#6aa0ff; }
    .grid { display:grid; grid-template-columns: 1fr; gap:16px; padding:16px; max-width:1100px; margin:0 auto; }
    @media(min-width: 960px){ .grid{ grid-template-columns: 1.1fr .9fr; } }
    .card { background:var(--card); border:1px solid #1c253b; border-radius:14px; padding:14px; }
    .card h2 { margin:6px 0 10px; font-size:16px; color:#d6e3ff; }
    .row { display:flex; gap:8px; flex-wrap:wrap; align-items:center; }
    input,select,button,textarea { background:#0f1422; color:var(--ink); border:1px solid #243049; border-radius:10px; padding:10px 12px; }
    button { cursor:pointer; }
    .ok { color:#92f2b6; }
    .warn { color:#ffd782; }
    .err { color:#ffa7a7; }
    .mono { font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; font-size: 12px; }
    .log { height:180px; overflow:auto; background:#0b0f1a; border-radius:10px; padding:10px; border:1px solid #1a2236; white-space:pre-wrap; }
    .pill.small { padding:6px 10px; font-size:12px; }
    .tabbar { display:flex; gap:8px; flex-wrap:wrap; margin-bottom:8px; }
    .tabbar .pill[data-active="true"]{ background:#29467a; color:white; }
    .hidden { display:none !important; }
    .kbd { padding:2px 6px; border-radius:6px; background:#0f1422; border:1px solid #243049; font-size:12px; }
    .muted { color:#99a9cc; font-size:13px; }
  </style>
</head>
<body>
<header>
  <h1>AI PI Lite — Control Hub</h1>
  <span id="connStatus" class="muted">Not connected</span>
  <div style="flex:1"></div>
  <button id="btnConnect" class="pill primary">Connect</button>
  <button id="btnDisconnect" class="pill hidden">Disconnect</button>
</header>

<main class="grid">
  <!-- LEFT COLUMN -->
  <section class="card">
    <div class="tabbar">
      <button class="pill small" data-tab="dashboard" data-active="true">Dashboard</button>
      <button class="pill small" data-tab="voice">Voice</button>
      <button class="pill small" data-tab="files">Files</button>
      <button class="pill small" data-tab="wifi">Wi-Fi</button>
      <button class="pill small" data-tab="lan">LAN Control</button>
      <button class="pill small" data-tab="github">GitHub</button>
      <button class="pill small" data-tab="settings">Settings</button>
    </div>

    <!-- DASHBOARD -->
    <div id="tab-dashboard">
      <h2>Status</h2>
      <div class="row">
        <span>Device:</span> <span id="devName" class="mono">—</span>
        <span>•</span>
        <span>Transport:</span> <span id="transport" class="mono">—</span>
        <span>•</span>
        <span>Battery (if exposed):</span> <span id="battery" class="mono">—</span>
      </div>
      <div class="row" style="margin-top:10px;">
        <button id="btnPing" class="pill small">Ping</button>
        <button id="btnScreenHello" class="pill small">Screen: Hello</button>
        <button id="btnReboot" class="pill small">Reboot</button>
      </div>
      <h2 style="margin-top:14px;">Logs</h2>
      <div id="log" class="log mono"></div>
    </div>

    <!-- VOICE -->
    <div id="tab-voice" class="hidden">
      <h2>Voice Assistant</h2>
      <div class="row">
        <button id="btnMic" class="pill">🎤 Start Listening</button>
        <span id="micState" class="muted">idle</span>
      </div>
      <p class="muted">Say things like: <span class="kbd">play track one</span>, <span class="kbd">pause</span>, <span class="kbd">volume up</span>, <span class="kbd">lights off</span>, <span class="kbd">roku home</span>, <span class="kbd">status</span>.</p>
      <h2>Wake Words</h2>
      <div class="row">
        <input id="wakeWord" placeholder="wake phrase (e.g., 'hey lite')" />
        <button id="btnSetWake" class="pill small">Set Wake Word</button>
      </div>
    </div>

    <!-- FILES -->
    <div id="tab-files" class="hidden">
      <h2>Transfer Files to Device</h2>
      <div class="row">
        <input type="file" id="fileInput" multiple />
        <button id="btnSendFiles" class="pill">Send to Device</button>
        <button id="btnList" class="pill small">List Device Files</button>
      </div>
      <div class="row" style="margin-top:6px;">
        <label>Remote path:</label>
        <input id="remoteDir" placeholder="/music" value="/music" />
      </div>
      <div id="fileList" class="log mono" style="height:120px; margin-top:10px;"></div>
    </div>

    <!-- WIFI -->
    <div id="tab-wifi" class="hidden">
      <h2>Wi-Fi Auto-Connect</h2>
      <p class="muted">Add known networks; the device will try them in order when it boots.</p>
      <div class="row">
        <input id="ssid" placeholder="SSID" />
        <input id="pw" placeholder="Password" />
        <button id="btnAddNet" class="pill small">Add</button>
        <button id="btnPushWifi" class="pill small">Push to Device</button>
      </div>
      <pre id="netList" class="log mono" style="height:120px;"></pre>
    </div>

    <!-- LAN -->
    <div id="tab-lan" class="hidden">
      <h2>LAN Device Control</h2>
      <p class="muted">Roku example (enable local network permission when prompted).</p>
      <div class="row">
        <input id="rokuIp" placeholder="Roku IP (e.g., 192.168.1.50)" />
        <button id="btnRokuHome" class="pill small">Roku Home</button>
        <button id="btnRokuPlay" class="pill small">Roku Play/Pause</button>
        <button id="btnRokuUp" class="pill small">Roku Up</button>
      </div>
      <h2>Generic HTTP (for bulbs, relays, etc.)</h2>
      <div class="row" style="gap:6px;">
        <select id="httpMethod">
          <option>GET</option><option>POST</option>
        </select>
        <input id="httpUrl" placeholder="http://device.local/action" style="min-width:260px;" />
        <input id="httpBody" placeholder='{"on":true}' />
        <button id="btnHttpSend" class="pill small">Send</button>
      </div>
      <pre id="lanOut" class="log mono" style="height:140px;"></pre>
    </div>

    <!-- GITHUB -->
    <div id="tab-github" class="hidden">
      <h2>Optional: Upload to Repo</h2>
      <p class="muted">Paste a temporary GitHub token (fine-grained) to upload files to your repo’s <span class="mono">inbox/</span>.</p>
      <div class="row">
        <input id="ghOwner" placeholder="owner (e.g., Grant9900)" />
        <input id="ghRepo" placeholder="repo (e.g., AI_PI_Lite_Project)" />
      </div>
      <div class="row" style="margin-top:6px;">
        <input id="ghToken" placeholder="token (not saved in code)" />
        <input type="file" id="ghFile" />
        <button id="btnGhUpload" class="pill small">Upload to inbox/</button>
      </div>
      <pre id="ghOut" class="log mono" style="height:120px;"></pre>
    </div>

    <!-- SETTINGS -->
    <div id="tab-settings" class="hidden">
      <h2>Transport</h2>
      <div class="row">
        <label><input type="radio" name="transport" value="serial" checked /> Web Serial</label>
        <label><input type="radio" name="transport" value="bluetooth" /> Web Bluetooth</label>
      </div>
      <h2>Companion AI (browser-based)</h2>
      <p class="muted">If your device is bound to a cloud agent, use that for LLM. If you have a browser-safe token, paste it below (stored in session).</p>
      <div class="row">
        <input id="agentUrl" placeholder="https://agent.example/ws or /api" style="min-width:320px;" />
        <input id="agentToken" placeholder="short-lived token" />
        <button id="btnAgentSave" class="pill small">Save</button>
      </div>
      <p class="muted">This page never commits tokens—stored only in your browser.</p>
    </div>
  </section>

  <!-- RIGHT COLUMN: HELP -->
  <aside class="card">
    <h2>Quick Help</h2>
    <ul>
      <li><b>Connect:</b> Click <span class="kbd">Connect</span>. Choose Serial or Bluetooth in <i>Settings → Transport</i>.</li>
      <li><b>Voice:</b> Click 🎤, say “<i>play track one</i>”, “<i>pause</i>”, “<i>status</i>”, “<i>roku home</i>”.</li>
      <li><b>Files → Device:</b> Pick files → <span class="kbd">Send to Device</span>. They’ll go to <span class="mono">/music</span> (changeable).</li>
      <li><b>Wi-Fi:</b> Add networks → <span class="kbd">Push to Device</span>. (Saves to device if firmware supports it.)</li>
      <li><b>LAN:</b> Enter Roku IP → keys, or send generic HTTP to bulbs/relays.</li>
      <li><b>GitHub (optional):</b> Paste token → upload to <span class="mono">inbox/</span> in your repo.</li>
    </ul>
    <p class="muted">USB “mass storage” mounting depends on the device’s firmware. Serial/BLE file pushes work now; MSC shows up automatically if your firmware exposes it.</p>
  </aside>
</main>

<script>
/* ---------------------------- Small UI helpers ---------------------------- */
const $ = sel => document.querySelector(sel);
const $$ = sel => Array.from(document.querySelectorAll(sel));
const logEl = $('#log');
function log(msg, cls='') {
  const t = new Date().toLocaleTimeString();
  logEl.innerHTML += `[${t}] ${msg}\n`;
  logEl.scrollTop = logEl.scrollHeight;
}
function setConn(status, transport='—', name='—') {
  $('#connStatus').textContent = status;
  $('#transport').textContent = transport;
  $('#devName').textContent = name;
  $('#btnConnect').classList.toggle('hidden', status!=='Not connected' && status!=='Connecting...');
  $('#btnDisconnect').classList.toggle('hidden', status==='Not connected' || status==='Connecting...');
}

/* ------------------------------ Tabs ------------------------------------- */
$$('.tabbar .pill').forEach(btn=>{
  btn.addEventListener('click', ()=>{
    $$('.tabbar .pill').forEach(b=>b.dataset.active='false');
    btn.dataset.active='true';
    const tab = btn.dataset.tab;
    ['dashboard','voice','files','wifi','lan','github','settings'].forEach(id=>{
      const el = $('#tab-'+id);
      el.classList.toggle('hidden', id!==tab);
    });
  });
});

/* ---------------------------- Transport layer ----------------------------- */
/* We implement a tiny line-based protocol over Serial/BLE:
   > JSON per line, e.g. {"op":"ping"}\n
   Device replies JSON lines: {"ok":true, "data":"pong"}
   File send uses {"op":"begin","path":"/music/a.mp3","size":12345}
   then raw base64 chunks: {"op":"chunk","data":"..."} repeated
   then {"op":"end"}
   You can adapt on the device firmware side to match.
*/
const Transport = {
  mode: 'serial', // 'serial' or 'bluetooth'
  serial: { port:null, reader:null, writer:null, decoder: new TextDecoder(), encoder: new TextEncoder() },
  ble: { device:null, server:null, service:null, rx:null, tx:null, decoder:new TextDecoder(), encoder:new TextEncoder() },
  onMessage: (obj)=>{ /* set later */ },
  async connect(){
    setConn('Connecting...');
    this.mode = document.querySelector('input[name="transport"]:checked').value;
    if (this.mode==='serial') return this.connectSerial();
    else return this.connectBLE();
  },
  async disconnect(){
    if (this.mode==='serial' && this.serial.port){
      try { await this.serial.reader?.cancel(); } catch(e){}
      try { await this.serial.port.close(); } catch(e){}
      this.serial = { port:null, reader:null, writer:null, decoder:new TextDecoder(), encoder:new TextEncoder() };
    }
    if (this.mode==='bluetooth' && this.ble.device){
      try { await this.ble.device.gatt.disconnect(); } catch(e){}
      this.ble = { device:null, server:null, service:null, rx:null, tx:null, decoder:new TextDecoder(), encoder:new TextEncoder() };
    }
    setConn('Not connected');
    log('Disconnected','warn');
  },
  async connectSerial(){
    if (!('serial' in navigator)) { log('Web Serial not supported (use Chrome/Edge).','err'); setConn('Not connected'); return; }
    const port = await navigator.serial.requestPort();
    await port.open({ baudRate: 115200 });
    this.serial.port = port;
    this.serial.writer = port.writable.getWriter();
    const reader = port.readable.getReader();
    this.serial.reader = reader;
    setConn('Connected','Serial','PI-Lite (serial)');
    log('Serial connected.','ok');
    (async ()=>{
      let buf='';
      for(;;){
        const { value, done } = await reader.read();
        if (done) break;
        buf += this.serial.decoder.decode(value);
        let idx;
        while((idx = buf.indexOf('\n'))>=0){
          const line = buf.slice(0, idx).trim(); buf = buf.slice(idx+1);
          if (!line) continue;
          try { const obj = JSON.parse(line); this.onMessage?.(obj); }
          catch(e){ log('RX (text): '+line); }
        }
      }
    })().catch(e=> log('Serial read error: '+e.message,'err'));
  },
  async connectBLE(){
    if (!('bluetooth' in navigator)) { log('Web Bluetooth not supported.','err'); setConn('Not connected'); return; }
    // NOTE: You must match these UUIDs to your firmware GATT service/characteristics.
    const SERVICE_UUID = '0000ffff-0000-1000-8000-00805f9b34fb';
    const TX_UUID = '0000ff01-0000-1000-8000-00805f9b34fb'; // write
    const RX_UUID = '0000ff02-0000-1000-8000-00805f9b34fb'; // notify
    const device = await navigator.bluetooth.requestDevice({
      filters: [{ namePrefix: 'PI-Lite' }], optionalServices: [SERVICE_UUID]
    });
    const server = await device.gatt.connect();
    const service = await server.getPrimaryService(SERVICE_UUID);
    const tx = await service.getCharacteristic(TX_UUID);
    const rx = await service.getCharacteristic(RX_UUID);
    await rx.startNotifications();
    rx.addEventListener('characteristicvaluechanged', (ev)=>{
      const txt = new TextDecoder().decode(ev.target.value);
      txt.split('\n').forEach(line=>{
        const s=line.trim(); if(!s) return;
        try { const obj=JSON.parse(s); this.onMessage?.(obj); }
        catch(e){ log('RX (text): '+s); }
      });
    });
    this.ble = { device, server, service, rx, tx, decoder:new TextDecoder(), encoder:new TextEncoder() };
    setConn('Connected','Bluetooth', device.name || 'PI-Lite (BLE)');
    log('Bluetooth connected.','ok');
  },
  async send(obj){
    const line = JSON.stringify(obj) + '\n';
    if (this.mode==='serial' && this.serial.writer){
      await this.serial.writer.write(this.serial.encoder.encode(line));
    } else if (this.mode==='bluetooth' && this.ble.tx){
      // BLE packets must be small; split into chunks
      const bytes = this.ble.encoder.encode(line);
      const MTU = 180;
      for (let i=0;i<bytes.length;i+=MTU){
        await this.ble.tx.writeValueWithoutResponse(bytes.slice(i, i+MTU));
      }
    } else {
      log('Not connected.','err');
    }
  }
};

Transport.onMessage = (msg)=>{
  if (msg.ok) log('OK: '+(msg.msg||''), 'ok');
  if (msg.err) log('ERR: '+msg.err, 'err');
  if (msg.data) log('DATA: '+(typeof msg.data==='string'? msg.data : JSON.stringify(msg.data)));
  if (msg.battery!==undefined) $('#battery').textContent = msg.battery+'%';
};

/* ------------------------------ Buttons ----------------------------------- */
$('#btnConnect').addEventListener('click', ()=> Transport.connect());
$('#btnDisconnect').addEventListener('click', ()=> Transport.disconnect());
$('#btnPing').addEventListener('click', ()=> Transport.send({op:'ping'}));
$('#btnScreenHello').addEventListener('click', ()=> Transport.send({op:'screen', text:'Hello from Hub!'}));
$('#btnReboot').addEventListener('click', ()=> Transport.send({op:'reboot'}));

/* ------------------------------ Voice ------------------------------------- */
let rec=null, listening=false;
function ensureSpeech(){
  const SR = window.SpeechRecognition || window.webkitSpeechRecognition;
  if (!SR) { log('Web Speech API not supported.','err'); return null; }
  const r = new SR(); r.lang='en-US'; r.interimResults=false; r.continuous=false; return r;
}
function handleCommand(text){
  log('Voice: '+text);
  const t = text.toLowerCase().trim();
  if (t.includes('status')) Transport.send({op:'status'});
  else if (t.includes('play') && t.includes('track')) {
    const n = (t.match(/\b(one|two|three|1|2|3)\b/)||[])[0]||'one';
    const map={one:1,two:2,three:3}; const idx = +map[n]||parseInt(n)||1;
    Transport.send({op:'play', index: idx});
  } else if (t.includes('pause')||t.includes('stop')) Transport.send({op:'pause'});
  else if (t.includes('volume up')) Transport.send({op:'volume', delta:+10});
  else if (t.includes('volume down')) Transport.send({op:'volume', delta:-10});
  else if (t.includes('lights on')) lanHttp('GET', $('#httpUrl').value || 'http://light.local/on');
  else if (t.includes('lights off')) lanHttp('GET', $('#httpUrl').value || 'http://light.local/off');
  else if (t.includes('roku') && t.includes('home')) rokuKey('Home');
  else if (t.includes('roku') && (t.includes('play')||t.includes('pause'))) rokuKey('Play');
  else Transport.send({op:'nlc', text}); // free-form to device/cloud agent
}
$('#btnMic').addEventListener('click', ()=>{
  if (!rec){ rec = ensureSpeech(); if(!rec) return; }
  if (listening){ rec.abort(); listening=false; $('#micState').textContent='idle'; $('#btnMic').textContent='🎤 Start Listening'; return; }
  rec.onresult = (e)=>{ const text = e.results[0][0].transcript; handleCommand(text); };
  rec.onend = ()=>{ listening=false; $('#micState').textContent='idle'; $('#btnMic').textContent='🎤 Start Listening'; };
  rec.onerror = (e)=>{ log('Mic error: '+e.error,'err'); listening=false; $('#micState').textContent='idle'; };
  rec.start(); listening=true; $('#micState').textContent='listening…'; $('#btnMic').textContent='🛑 Stop';
});

/* ------------------------------ Files → Device ---------------------------- */
$('#btnList').addEventListener('click', ()=> Transport.send({op:'list', path: $('#remoteDir').value || '/'}));
$('#btnSendFiles').addEventListener('click', async ()=>{
  const files = $('#fileInput').files;
  if (!files || !files.length){ log('Pick files first.','err'); return; }
  for (const f of files) await sendFileToDevice(f, $('#remoteDir').value || '/');
  log('All files sent.','ok');
});
async function sendFileToDevice(file, dir){
  const path = (dir.endsWith('/')? dir: dir+'/') + file.name;
  await Transport.send({op:'begin', path, size:file.size});
  const CHUNK = 8*1024;
  let sent = 0;
  const reader = file.stream().getReader();
  for(;;){
    const { value, done } = await reader.read();
    if (done) break;
    sent += value.byteLength;
    const b64 = btoa(String.fromCharCode(...new Uint8Array(value)));
    await Transport.send({op:'chunk', data:b64});
    log(`Sending ${file.name}: ${((sent/file.size)*100).toFixed(1)}%`);
  }
  await Transport.send({op:'end'});
  log(`Done: ${file.name}`, 'ok');
}
Transport.onMessage = (msg)=>{
  if (msg.list){ $('#fileList').textContent = msg.list.map(x=>`${x.name} ${x.size}B`).join('\n'); }
  if (msg.ok) log('OK: '+(msg.msg||''),'ok');
  if (msg.err) log('ERR: '+msg.err,'err');
  if (msg.battery!==undefined) $('#battery').textContent = msg.battery+'%';
  if (msg.data && typeof msg.data==='string') log('DATA: '+msg.data);
};

/* ------------------------------ Wi-Fi manager ----------------------------- */
const nets = JSON.parse(localStorage.getItem('aipi:nets')||'[]');
function renderNets(){ $('#netList').textContent = JSON.stringify(nets, null, 2); }
renderNets();
$('#btnAddNet').addEventListener('click', ()=>{
  const ssid = $('#ssid').value.trim(); const pw = $('#pw').value;
  if (!ssid) return;
  nets.push({ssid, pw});
  localStorage.setItem('aipi:nets', JSON.stringify(nets));
  $('#ssid').value=''; $('#pw').value='';
  renderNets();
});
$('#btnPushWifi').addEventListener('click', ()=> Transport.send({op:'wifi', nets}));

/* ------------------------------ LAN control ------------------------------- */
async function rokuKey(key){
  const ip = $('#rokuIp').value.trim(); if(!ip){ log('Set Roku IP.','err'); return; }
  try{
    const r = await fetch(`http://${ip}:8060/keypress/${encodeURIComponent(key)}`, { method:'POST' });
    $('#lanOut').textContent = 'Roku '+key+': '+r.status;
  }catch(e){ $('#lanOut').textContent = 'Roku error: '+e.message; }
}
$('#btnRokuHome').addEventListener('click', ()=> rokuKey('Home'));
$('#btnRokuPlay').addEventListener('click', ()=> rokuKey('Play'));
$('#btnRokuUp').addEventListener('click', ()=> rokuKey('Up'));
async function lanHttp(method, url, body){
  try{
    const r = await fetch(url, { method, headers: body? {'Content-Type':'application/json'}:{}, body: body||undefined });
    const txt = await r.text(); $('#lanOut').textContent = `HTTP ${r.status}\n`+txt.slice(0,4000);
  }catch(e){ $('#lanOut').textContent = 'HTTP error: '+e.message; }
}
$('#btnHttpSend').addEventListener('click', ()=>{
  const m = $('#httpMethod').value;
  const u = $('#httpUrl').value;
  const b = $('#httpBody').value.trim();
  lanHttp(m, u, b? b: undefined);
});

/* ------------------------------ GitHub upload (optional) ------------------ */
$('#btnGhUpload').addEventListener('click', async ()=>{
  const owner = $('#ghOwner').value.trim(), repo = $('#ghRepo').value.trim(), token = $('#ghToken').value.trim();
  const file = $('#ghFile').files?.[0];
  if (!owner || !repo || !token || !file){ $('#ghOut').textContent='Fill owner/repo/token and pick a file.'; return; }
  const path = `inbox/${encodeURIComponent(file.name)}`;
  const content = await file.arrayBuffer();
  const b64 = btoa(String.fromCharCode(...new Uint8Array(content)));
  const url = `https://api.github.com/repos/${owner}/${repo}/contents/${path}`;
  try{
    const r = await fetch(url, { method:'PUT', headers:{ 'Authorization': 'Bearer '+token, 'Content-Type':'application/json' },
      body: JSON.stringify({ message:`upload ${file.name}`, content:b64 }) });
    const out = await r.json();
    $('#ghOut').textContent = 'Uploaded: '+(out.content?.path || JSON.stringify(out).slice(0,400));
  }catch(e){ $('#ghOut').textContent = 'Error: '+e.message; }
});

/* ------------------------------ Agent (cloud) ----------------------------- */
$('#btnAgentSave').addEventListener('click', ()=>{
  sessionStorage.setItem('aipi:agentUrl', $('#agentUrl').value.trim());
  sessionStorage.setItem('aipi:agentToken', $('#agentToken').value.trim());
  log('Agent settings saved (session only).','ok');
});

/* ------------------------------ Wake word --------------------------------- */
$('#btnSetWake').addEventListener('click', ()=>{
  const w = $('#wakeWord').value.trim();
  if (!w) return;
  Transport.send({op:'wake', word:w});
});
</script>
</body>
</html>
