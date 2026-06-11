<!DOCTYPE html>
<html lang="ar" dir="ltr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>RF Link Planner TWSG </title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <style>
    :root {
      --primary:#2563eb; --secondary:#64748b; --danger:#ef4444;
      --dark:#0f172a; --bg:#f1f5f9; --panel-bg:#ffffff;
      --success:#16a34a; --warning:#d97706;
    }
    *{box-sizing:border-box;font-family:'Inter',sans-serif;}
    body{margin:0;background:var(--bg);color:#334155;overflow:hidden;}
    header{background:var(--dark);color:white;padding:.6rem 1.2rem;display:flex;
      justify-content:space-between;align-items:center;
      box-shadow:0 2px 8px rgba(0,0,0,.2);z-index:1000;position:relative;}
    header h1{margin:0;font-size:1.1rem;font-weight:700;display:flex;align-items:center;gap:8px;}
    .hbadge{font-size:.65rem;background:#1e40af;padding:2px 8px;border-radius:20px;font-weight:600;}
    .elev-status{font-size:.7rem;padding:3px 10px;border-radius:20px;font-weight:600;transition:all .3s;}
    .elev-ok  {background:#166534;color:#bbf7d0;}
    .elev-warn{background:#854d0e;color:#fef9c3;}
    .elev-err {background:#991b1b;color:#fecaca;}
    .layout{display:grid;grid-template-columns:380px 1fr;height:calc(100vh - 48px);}
    .panel{background:var(--panel-bg);padding:16px;overflow-y:auto;
      border-right:1px solid #e2e8f0;box-shadow:4px 0 15px rgba(0,0,0,.05);
      z-index:999;scrollbar-width:thin;}
    .section{margin-bottom:14px;padding:13px;background:#f8fafc;
      border:1px solid #e2e8f0;border-radius:12px;}
    .section h2{font-size:.78rem;text-transform:uppercase;letter-spacing:.06em;
      margin:0 0 10px;color:var(--secondary);
      border-bottom:1px solid #e2e8f0;padding-bottom:6px;}
    label{display:block;font-size:.72rem;font-weight:600;margin:8px 0 4px;color:#475569;}
    input,select{width:100%;padding:7px 10px;border:1px solid #cbd5e1;
      border-radius:6px;font-size:.85rem;background:white;transition:border-color .2s;}
    input:focus,select:focus{outline:none;border-color:var(--primary);
      box-shadow:0 0 0 3px rgba(37,99,235,.1);}
    .row{display:grid;grid-template-columns:1fr 1fr;gap:10px;}
    .input-with-unit{display:grid;grid-template-columns:1fr 85px;gap:6px;}
    button{width:100%;padding:9px;background:var(--primary);color:white;
      border:none;border-radius:6px;font-weight:600;cursor:pointer;margin-top:8px;
      transition:all .2s;font-size:.85rem;}
    button:hover{filter:brightness(1.1);}
    button:active{transform:scale(.98);}
    button.danger{background:#fee2e2;color:var(--danger);border:1px solid #fecaca;}
    button.danger:hover{background:var(--danger);color:white;}
    #map{width:100%;height:100%;}
    .results-grid{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-top:12px;}
    .card{background:white;padding:10px 12px;border-radius:8px;border:1px solid #e2e8f0;transition:all .3s;}
    .card span{display:block;font-size:.62rem;color:#94a3b8;text-transform:uppercase;letter-spacing:.04em;margin-bottom:3px;}
    .card strong{font-size:.92rem;color:var(--dark);font-weight:700;}
    .status-bar{padding:10px;border-radius:8px;margin:12px 0;font-weight:700;text-align:center;font-size:.88rem;border:1px solid transparent;transition:all .4s;}
    .excellent{background:#dcfce7;color:#166534;border-color:#bbf7d0;}
    .good{background:#dbeafe;color:#1e40af;border-color:#bfdbfe;}
    .poor{background:#fee2e2;color:#991b1b;border-color:#fecaca;}
    .formula-box{background:white;padding:11px;border-radius:8px;border:1px solid #e2e8f0;font-size:.77rem;line-height:1.8;color:#334155;}
    .formula-box code{background:#f1f5f9;padding:2px 5px;border-radius:4px;color:#0f172a;font-size:.75rem;}
    .toggle-row{display:flex;align-items:center;justify-content:space-between;margin:6px 0;}
    .toggle-label{font-size:.73rem;font-weight:600;color:#475569;display:flex;align-items:center;gap:6px;}
    .tog{position:relative;width:36px;height:19px;flex-shrink:0;}
    .tog input{opacity:0;width:0;height:0;}
    .sl{position:absolute;cursor:pointer;inset:0;background:#cbd5e1;border-radius:20px;transition:.3s;}
    .sl:before{position:absolute;content:"";height:13px;width:13px;left:3px;bottom:3px;background:white;border-radius:50%;transition:.3s;}
    input:checked+.sl{background:var(--primary);}
    input:checked+.sl:before{transform:translateX(17px);}
    .fp{margin-bottom:14px;padding:13px;background:#f8fafc;border:1px solid #e2e8f0;border-radius:12px;}
    .fp h2{font-size:.78rem;text-transform:uppercase;letter-spacing:.06em;margin:0 0 10px;color:var(--secondary);border-bottom:1px solid #e2e8f0;padding-bottom:6px;}
    .fcards{display:grid;grid-template-columns:1fr 1fr;gap:7px;margin-top:8px;}
    .fc{background:white;padding:8px 10px;border-radius:8px;border:1px solid #e2e8f0;}
    .fc span{display:block;font-size:.6rem;color:#94a3b8;text-transform:uppercase;}
    .fc strong{font-size:.85rem;color:var(--dark);}
    .los-bar{margin-top:8px;padding:8px 12px;border-radius:8px;font-size:.78rem;font-weight:700;text-align:center;border:1px solid transparent;}
    .los-ok  {background:#dcfce7;color:#166534;border-color:#bbf7d0;}
    .los-warn{background:#fef9c3;color:#854d0e;border-color:#fde68a;}
    .los-bad {background:#fee2e2;color:#991b1b;border-color:#fecaca;}
    #tcWrap{margin-top:8px;background:white;border:1px solid #e2e8f0;border-radius:8px;overflow:hidden;}
    #tcWrap canvas{display:block;width:100%;height:110px;}
    .spn{display:flex;align-items:center;gap:8px;font-size:.72rem;color:#64748b;margin-top:6px;}
    .spin{width:13px;height:13px;border:2px solid #e2e8f0;border-top-color:var(--primary);border-radius:50%;animation:sp .7s linear infinite;flex-shrink:0;}
    @keyframes sp{to{transform:rotate(360deg);}}
    .prog-wrap{margin-top:6px;height:4px;background:#e2e8f0;border-radius:2px;overflow:hidden;}
    .prog-bar{height:100%;background:var(--primary);border-radius:2px;transition:width .3s;width:0%;}
    .remote-row{padding:8px;background:#f8fafc;border:1px solid #e2e8f0;border-radius:8px;margin-bottom:6px;}
    .remote-row-header{display:flex;align-items:center;gap:6px;margin-bottom:5px;}
    .remote-dot{width:10px;height:10px;border-radius:50%;flex-shrink:0;}
    .remote-row-label{font-size:.72rem;font-weight:700;color:#334155;flex:1;}
    .remote-row input{margin-bottom:0;font-size:.8rem;}
    .remote-rm{width:24px;height:24px;background:#fee2e2;color:#ef4444;border:1px solid #fecaca;border-radius:5px;cursor:pointer;font-size:12px;font-weight:700;padding:0;flex-shrink:0;display:flex;align-items:center;justify-content:center;transition:all .2s;}
    .remote-rm:hover{background:#ef4444;color:white;}
    .mode-toggle{display:flex;background:#e2e8f0;border-radius:8px;padding:3px;margin-bottom:14px;gap:3px;}
    .mode-btn{flex:1;padding:7px 0;text-align:center;font-size:.78rem;font-weight:700;border-radius:6px;cursor:pointer;border:none;background:transparent;color:#64748b;transition:all .2s;}
    .mode-btn.active{background:var(--primary);color:white;box-shadow:0 2px 6px rgba(37,99,235,.25);}
    .mode-badge{display:inline-block;font-size:.6rem;background:#1e40af;color:white;padding:1px 6px;border-radius:10px;margin-left:4px;vertical-align:middle;}
    .export-btn{width:100%;padding:10px;background:#16a34a;color:white;border:none;border-radius:6px;font-weight:700;cursor:pointer;font-size:.85rem;display:flex;align-items:center;justify-content:center;gap:8px;transition:all .2s;margin-top:6px;}
    .export-btn:hover{background:#15803d;}
    .export-btn:active{transform:scale(.98);}
    .export-section{margin-top:14px;padding:13px;background:#f0fdf4;border:1px solid #bbf7d0;border-radius:12px;}
    .export-section h2{font-size:.78rem;text-transform:uppercase;letter-spacing:.06em;margin:0 0 10px;color:#16a34a;border-bottom:1px solid #bbf7d0;padding-bottom:6px;}
    @media(max-width:860px){
      body{overflow:auto;}
      .layout{grid-template-columns:1fr;height:auto;}
      .panel{border-right:none;border-bottom:1px solid #e2e8f0;max-height:55vh;overflow-y:auto;}
      #map{height:45vh;}
      main{position:relative;height:45vh;}
    }
  </style>
</head>
<body>

<header>
  <h1>📡 RF Link Planner <span class="hbadge">TWSG</span></h1>
  <div id="elevStatus" class="elev-status elev-warn">Elevation: checking…</div>
</header>

<div class="layout">
<aside class="panel">

  <div class="mode-toggle">
    <button class="mode-btn active" id="btnP2P" onclick="setMode('p2p')">
      📡 Point-to-Point <span class="mode-badge">P2P</span>
    </button>
    <button class="mode-btn" id="btnP2MP" onclick="setMode('p2mp')">
      🌐 Multipoint <span class="mode-badge">P2MP</span>
    </button>
  </div>

  <!-- P2P Sites -->
  <div class="section" id="p2pSitesSection">
    <h2>📍 Site Locations</h2>
    <div class="row">
      <div>
        <label>Site A (Lat, Lon)</label>
        <input id="coordA" placeholder="24.7136, 46.6753" onchange="updateMarkerFromInput(0)"/>
      </div>
      <div>
        <label>Site B (Lat, Lon)</label>
        <input id="coordB" placeholder="26.4207, 50.0888" onchange="updateMarkerFromInput(1)"/>
      </div>
    </div>
    <label>Distance (km)</label>
    <input id="distance" readonly placeholder="calculated from map"/>
    <div style="display:grid;grid-template-columns:1fr 40px;gap:6px;margin-top:8px;">
      <button class="danger" onclick="clearMap()" style="margin-top:0;">🗑 Reset Map</button>
      <button id="undoBtn" onclick="undoLast()" title="Undo"
        style="margin-top:0;padding:9px;background:#fef9c3;color:#854d0e;
        border:1px solid #fde68a;border-radius:6px;font-size:1rem;cursor:pointer;
        font-weight:700;transition:all .2s;opacity:.4;pointer-events:none;" disabled>↩</button>
    </div>
    <label style="margin-top:12px;">Fixed Site Presets</label>
    <select id="presetSites" onchange="loadPresetSites()">
      <option value="">Select Preset Pair</option>
      <option value="0">Riyadh City Center ↔ Al Malaz</option>
      <option value="1">Dammam Port ↔ Al Khobar</option>
      <option value="2">Jeddah North ↔ Jeddah Airport</option>
      <option value="3">Madinah Center ↔ Madinah East</option>
      <option value="4">Tabuk Center ↔ Tabuk North</option>
      <option value="5">Abha Center ↔ Abha East</option>
      <option value="6">Hail Center ↔ Hail East</option>
    </select>
  </div>

  <!-- P2MP Sites -->
  <div class="section" id="p2mpSitesSection" style="display:none;">
    <h2>🌐 Hub & Remote Sites</h2>
    <label>Hub Location (Lat, Lon)</label>
    <div style="display:grid;grid-template-columns:1fr 40px;gap:6px;">
      <input id="hubCoord" placeholder="Click map or type: 24.7136, 46.6753" onchange="placeHubFromInput()"/>
      <button id="undoBtnP2MP" onclick="undoLast()" title="Undo"
        style="padding:0;background:#fef9c3;color:#854d0e;border:1px solid #fde68a;
        border-radius:6px;font-size:1rem;cursor:pointer;font-weight:700;transition:all .2s;
        opacity:.4;pointer-events:none;display:flex;align-items:center;justify-content:center;" disabled>↩</button>
    </div>
    <div id="hubPlacedBadge" style="display:none;margin-top:6px;padding:6px 10px;
      background:#eff6ff;border:1px solid #bfdbfe;border-radius:7px;font-size:.72rem;font-weight:700;color:#1e40af;">
      Hub placed — click map or type below to add remotes
    </div>
    <div id="remotesContainer" style="margin-top:10px;display:none;">
      <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:6px;">
        <label style="margin:0;">Remote Sites</label>
        <button onclick="addRemoteRow()" style="width:auto;padding:4px 10px;font-size:.72rem;
          background:#eff6ff;color:#1e40af;border:1px solid #bfdbfe;border-radius:6px;
          margin:0;cursor:pointer;font-weight:600;">+ Add Remote</button>
      </div>
      <div id="remoteRowsList"></div>
    </div>
    <div id="p2mpSelectedInfo" style="display:none;margin-top:8px;padding:8px 10px;
      background:#eff6ff;border:1px solid #bfdbfe;border-radius:8px;
      font-size:.72rem;color:#1e40af;font-weight:600;text-align:center;">
      👆 Click a Remote on the map to see its link budget
    </div>
    <div style="margin-top:10px;">
      <button class="danger" onclick="clearP2MPAll()" style="margin-top:0;">🗑 Reset</button>
    </div>
  </div>

  <!-- Config -->
  <div class="section">
    <h2>⚙️ Configuration</h2>
    <label>Frequency</label>
    <div class="input-with-unit">
      <input id="frequency" type="number" value="5.8" step="0.1" oninput="calculateLink();if(currentMode==='p2mp')recalcAllRemotes();"/>
      <select id="frequencyUnit" onchange="calculateLink();if(currentMode==='p2mp')recalcAllRemotes();">
        <option value="GHz" selected>GHz</option>
        <option value="MHz">MHz</option>
      </select>
    </div>
    <label>Transmit Power</label>
    <div class="input-with-unit">
      <input id="txPower" type="number" value="1" step="0.1" oninput="calculateLink();if(currentMode==='p2mp')recalcAllRemotes();"/>
      <select id="powerUnit" onchange="calculateLink();if(currentMode==='p2mp')recalcAllRemotes();">
        <option value="W" selected>Watt</option>
        <option value="dBm">dBm</option>
      </select>
    </div>
    <div class="row">
      <div>
        <label>Antenna Gain</label>
        <input id="antennaGain" type="number" value="23" step="0.1" oninput="calculateLink();if(currentMode==='p2mp')recalcAllRemotes();"/>
        <label style="margin-top:3px;color:#94a3b8;">dBi</label>
      </div>
      <div>
        <label>Cable Loss</label>
        <input id="cableLoss" type="number" value="2" step="0.1" oninput="calculateLink();if(currentMode==='p2mp')recalcAllRemotes();"/>
        <label style="margin-top:3px;color:#94a3b8;">dB</label>
      </div>
    </div>
    <label>Bandwidth</label>
    <div class="input-with-unit">
      <input id="bandwidth" type="number" value="20" step="1" oninput="calculateLink();if(currentMode==='p2mp')recalcAllRemotes();"/>
      <select id="bandwidthUnit" onchange="calculateLink();if(currentMode==='p2mp')recalcAllRemotes();">
        <option value="Hz">Hz</option>
        <option value="MHz" selected>MHz</option>
        <option value="GHz">GHz</option>
      </select>
    </div>
    <div class="row">
      <div>
        <label id="labelAntA">Antenna Ht A</label>
        <input id="antHA" type="number" value="30" step="1" min="1" oninput="calculateLink();if(currentMode==='p2mp')recalcAllRemotes();"/>
        <label style="margin-top:3px;color:#94a3b8;">m AGL</label>
      </div>
      <div>
        <label id="labelAntB">Antenna Ht B</label>
        <input id="antHB" type="number" value="30" step="1" min="1" oninput="calculateLink();if(currentMode==='p2mp')recalcAllRemotes();"/>
        <label style="margin-top:3px;color:#94a3b8;">m AGL</label>
      </div>
    </div>
  </div>

  <!-- Results -->
  <div id="p2mpActiveBanner" style="display:none;padding:8px 12px;border-radius:8px;
    margin:8px 0 4px;font-weight:700;text-align:center;font-size:.82rem;
    border:2px solid var(--primary);background:#eff6ff;color:#1e40af;">—</div>
  <div id="qualityBox" class="status-bar">📍 Click two points on the map to begin</div>
  <div class="results-grid">
    <div class="card"><span>FSPL</span><strong id="resFSPL">--</strong></div>
    <div class="card"><span>RSL</span><strong id="resRSL">--</strong></div>
    <div class="card"><span>Noise Floor</span><strong id="resNoise">--</strong></div>
    <div class="card"><span>SNR</span><strong id="resSNR">--</strong></div>
    <div class="card"><span>EIRP</span><strong id="resEIRP">--</strong></div>
    <div class="card"><span>Max Range</span><strong id="resMaxRange">--</strong></div>
  </div>

  <!-- Formulas -->
  <div class="section" style="margin-top:14px;">
    <h2>📘 Formulas Used</h2>
    <div class="formula-box">
      <strong>FSPL:</strong><br>
      <code>FSPL = 92.44 + 20log₁₀(d km) + 20log₁₀(f GHz)</code><br><br>
      <strong>Tx Power:</strong> <code>dBm = 10log₁₀(W × 1000)</code><br><br>
      <strong>RSL:</strong> <code>RSL = Tx(dBm) + Gain − Loss − FSPL</code><br><br>
      <strong>Noise Floor:</strong> <code>NF = −174 + 10log₁₀(BW Hz)</code><br><br>
      <strong>SNR:</strong> <code>RSL − NF</code><br><br>
      <strong>Max Range:</strong><br>
      <code>Range = 10^[(MaxFSPL − 92.44 − 20log₁₀f) / 20]</code>
    </div>
  </div>

  <!-- Fresnel -->
  <div class="fp">
    <h2>🌐 Fresnel Zone & LOS</h2>
    <div class="toggle-row">
      <span class="toggle-label">
        <span style="width:14px;height:10px;display:inline-block;background:rgba(234,179,8,.55);border:1.5px solid #ca8a04;border-radius:2px;"></span>
        1st Fresnel Zone
      </span>
      <label class="tog"><input type="checkbox" id="tog1" checked onchange="redrawAll()"><span class="sl"></span></label>
    </div>
    <div class="toggle-row">
      <span class="toggle-label">
        <span style="width:14px;height:10px;display:inline-block;background:rgba(34,197,94,.35);border:1.5px solid #16a34a;border-radius:2px;"></span>
        60% Clearance Zone
      </span>
      <label class="tog"><input type="checkbox" id="tog60" checked onchange="redrawAll()"><span class="sl"></span></label>
    </div>
    <div class="toggle-row">
      <span class="toggle-label">
        <span style="width:14px;height:10px;display:inline-block;background:rgba(239,68,68,.3);border:1.5px dashed #ef4444;border-radius:2px;"></span>
        Terrain Obstructions
      </span>
      <label class="tog"><input type="checkbox" id="togBlk" checked onchange="redrawAll()"><span class="sl"></span></label>
    </div>
    <!-- Heatmap Toggle -->
    <div class="toggle-row" style="margin-top:8px;">
      <span class="toggle-label">
        <span style="width:14px;height:10px;display:inline-block;background:linear-gradient(90deg,#22c55e,#eab308,#ef4444);border-radius:2px;"></span>
        Coverage Heatmap
      </span>
      <label class="tog"><input type="checkbox" id="togHeatmap" onchange="toggleHeatmapMode()"><span class="sl"></span></label>
    </div>
    <div id="heatmapLegend" style="display:none;margin-top:6px;padding:6px 8px;background:white;border:1px solid #e2e8f0;border-radius:8px;">
      <div style="font-size:.65rem;color:#64748b;margin-bottom:4px;font-weight:600;">COVERAGE LEVEL</div>
      <div style="display:flex;gap:4px;align-items:center;font-size:.65rem;color:#475569;">
        <span style="width:12px;height:12px;border-radius:50%;background:#22c55e;display:inline-block;"></span> Excellent (&gt;25dB)
        <span style="width:12px;height:12px;border-radius:50%;background:#eab308;display:inline-block;margin-left:6px;"></span> Good (15-25dB)
        <span style="width:12px;height:12px;border-radius:50%;background:#ef4444;display:inline-block;margin-left:6px;"></span> Poor (&lt;15dB)
      </div>
    </div>
    <div id="losBar" class="los-bar" style="display:none;"></div>
    <div id="spnWrap" class="spn" style="display:none;">
      <div class="spin"></div><span id="spnMsg">Computing…</span>
      <div class="prog-wrap" style="flex:1"><div class="prog-bar" id="progBar"></div></div>
    </div>
    <div id="tcWrap" style="display:none;"><canvas id="tcCanvas"></canvas></div>
    <div class="fcards">
      <div class="fc"><span>Fresnel r₁ max</span><strong id="fR1">--</strong></div>
      <div class="fc"><span>60% Clear r</span><strong id="fR60">--</strong></div>
      <div class="fc"><span>EIRP</span><strong id="fEIRP">--</strong></div>
      <div class="fc"><span>RF Range</span><strong id="fRange">--</strong></div>
      <div class="fc"><span>LOS Status</span><strong id="fLOS">--</strong></div>
      <div class="fc"><span>Obstructions</span><strong id="fObs">--</strong></div>
    </div>
  </div>

  <!-- AI Analysis -->
  <div class="section" style="margin-top:14px;padding:13px;background:#faf5ff;border:1px solid #e9d5ff;border-radius:12px;">
    <h2 style="font-size:.78rem;text-transform:uppercase;letter-spacing:.06em;margin:0 0 10px;color:#7c3aed;border-bottom:1px solid #e9d5ff;padding-bottom:6px;">🤖 AI Analysis</h2>
    <button id="analyzeBtn" onclick="handleAnalyzeBtn()"
      style="width:100%;padding:10px;background:linear-gradient(135deg,#7c3aed,#4f46e5);color:white;border:none;border-radius:8px;font-weight:700;cursor:pointer;font-size:.88rem;margin-top:0;">
      🤖 Analyze This Link
    </button>
    <div id="aiThinking" style="display:none;margin-top:8px;font-size:.72rem;color:#7c3aed;">
      ⏳ Analyzing...
    </div>
    <div id="aiResult" style="display:none;margin-top:10px;padding:12px;background:white;border:1px solid #e9d5ff;border-radius:8px;font-size:.78rem;line-height:1.7;color:#334155;"></div>
  </div>

  <!-- Export -->
  <div class="export-section">
    <h2>📥 Export Report</h2>
    <button class="export-btn" onclick="exportExcel()">📊 Export to Excel</button>
  </div>

</aside>
<main style="position:relative;">
  <div id="map"></div>
</main>
</div>

<script>
const CACHE_KEY='rf_elev_v3', CACHE_MAX=300;
function cacheGet(k){try{return(JSON.parse(sessionStorage.getItem(CACHE_KEY)||'{}'))[k]||null;}catch{return null;}}
function cacheSet(k,v){try{let s=JSON.parse(sessionStorage.getItem(CACHE_KEY)||'{}');const ks=Object.keys(s);if(ks.length>=CACHE_MAX)ks.slice(0,50).forEach(x=>delete s[x]);s[k]=v;sessionStorage.setItem(CACHE_KEY,JSON.stringify(s));}catch{}}
function elevCacheKey(l){return l.map(x=>`${x.latitude.toFixed(3)},${x.longitude.toFixed(3)}`).join('|');}

let elevApiOk=null;
const ELEV_APIS=[
  async(locs)=>{const r=await fetch('https://api.open-elevation.com/api/v1/lookup',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({locations:locs}),signal:AbortSignal.timeout(8000)});if(!r.ok)throw new Error('HTTP '+r.status);return(await r.json()).results.map(x=>x.elevation);},
  async(locs)=>{const la=locs.map(l=>l.latitude.toFixed(4)).join(','),ln=locs.map(l=>l.longitude.toFixed(4)).join(',');const r=await fetch(`https://api.open-meteo.com/v1/elevation?latitude=${la}&longitude=${ln}`,{signal:AbortSignal.timeout(8000)});if(!r.ok)throw new Error('HTTP '+r.status);const d=await r.json();return Array.isArray(d.elevation)?d.elevation:locs.map(()=>0);}
];
function setElevStatus(s,m){const el=document.getElementById('elevStatus');el.className='elev-status '+(s==='ok'?'elev-ok':s==='warn'?'elev-warn':'elev-err');el.textContent=m;}
async function fetchElevWithCache(locs){const k=elevCacheKey(locs),c=cacheGet(k);if(c)return c;for(let i=0;i<ELEV_APIS.length;i++){try{const r=await ELEV_APIS[i](locs);if(r&&r.length===locs.length){cacheSet(k,r);elevApiOk=true;setElevStatus('ok',' Elevation: live data');return r;}}catch(e){console.warn(`Elev API ${i+1}:`,e.message);}}elevApiOk=false;setElevStatus('err',' Elevation: offline');return locs.map(()=>0);}
async function fetchElev(A,B,n){const locs=[];for(let i=0;i<n;i++){const t=i/(n-1),p=interp(A,B,t);locs.push({latitude:+p.lat.toFixed(4),longitude:+p.lng.toFixed(4)});}return fetchElevWithCache(locs);}

const map=L.map('map').setView([24.7136,46.6753],6);
L.tileLayer('https://{s}.google.com/vt/lyrs=y&x={x}&y={y}&z={z}',{maxZoom:20,subdomains:['mt0','mt1','mt2','mt3'],attribution:'© Google Maps'}).addTo(map);
let markers=[],polyline=null,previewLine=null;
const lyr={f1:null,f60:null,blk:[]};
let CACHE=null;

function addOrUpdateMarker(i,ll){
  if(markers[i]){markers[i].setLatLng(ll);return;}
  const icon=L.divIcon({className:'',html:`<div style="width:28px;height:28px;border-radius:50%;background:${i===0?'#2563eb':'#ef4444'};border:3px solid white;box-shadow:0 2px 8px rgba(0,0,0,.4);display:flex;align-items:center;justify-content:center;color:white;font-weight:700;font-size:13px;">${i===0?'A':'B'}</div>`,iconSize:[28,28],iconAnchor:[14,14]});
  const m=L.marker(ll,{draggable:true,icon}).addTo(map);
  m.bindTooltip(i===0?'Site A':'Site B',{permanent:false});
  m.on('drag',()=>{updateInputs();calculateLink();});
  markers[i]=m;
}
function updateMarkerFromInput(i){const v=document.getElementById(i===0?'coordA':'coordB').value.trim().split(',');if(v.length!==2)return;const la=parseFloat(v[0]),ln=parseFloat(v[1]);if(isNaN(la)||isNaN(ln))return;addOrUpdateMarker(i,L.latLng(la,ln));map.setView(L.latLng(la,ln),12);if(markers.length===2&&markers[0]&&markers[1]){calculateLink();map.fitBounds(L.latLngBounds([markers[0].getLatLng(),markers[1].getLatLng()]),{padding:[50,50]});}}
function updateInputs(){if(markers[0])document.getElementById('coordA').value=markers[0].getLatLng().lat.toFixed(4)+', '+markers[0].getLatLng().lng.toFixed(4);if(markers[1])document.getElementById('coordB').value=markers[1].getLatLng().lat.toFixed(4)+', '+markers[1].getLatLng().lng.toFixed(4);}

map.on('click',e=>{
  if(heatmapMode){drawHeatmap(e.latlng);return;}
  if(currentMode==='p2mp'){if(previewLine){map.removeLayer(previewLine);previewLine=null;}handleP2MPClick(e.latlng);return;}
  if(markers.length>=2)clearMap();
  const idx=markers.length;
  addOrUpdateMarker(idx,e.latlng);
  undoStack.push({type:idx===0?'p2p_a':'p2p_b',latlng:e.latlng});
  updateUndoBtn();updateInputs();
  if(markers.length===2){if(previewLine){map.removeLayer(previewLine);previewLine=null;}calculateLink();}
});
map.on('mousemove',e=>{
  if(currentMode==='p2mp'){if(previewLine){map.removeLayer(previewLine);previewLine=null;}if(p2mpHub){previewLine=L.polyline([p2mpHub.getLatLng(),e.latlng],{color:'#94a3b8',weight:2,dashArray:'5,5',opacity:.6}).addTo(map);}return;}
  if(markers.length===1){if(previewLine)map.removeLayer(previewLine);previewLine=L.polyline([markers[0].getLatLng(),e.latlng],{color:'#94a3b8',weight:3,dashArray:'6,6'}).addTo(map);}
});

const cvtF=(v,u)=>u==='MHz'?v/1000:v;
const cvtB=(v,u)=>u==='GHz'?v*1e9:u==='MHz'?v*1e6:v;
const cvtP=(v,u)=>u==='W'?10*Math.log10(v*1000):v;

function clearMap(){
  markers.forEach(m=>{if(m)map.removeLayer(m);});markers=[];
  if(polyline){map.removeLayer(polyline);polyline=null;}
  if(previewLine){map.removeLayer(previewLine);previewLine=null;}
  clearLayers();CACHE=null;
  ['coordA','coordB','distance'].forEach(id=>document.getElementById(id).value='');
  document.getElementById('qualityBox').className='status-bar';
  document.getElementById('qualityBox').innerText='📍 Click two points on the map to begin';
  ['resFSPL','resRSL','resNoise','resSNR','resEIRP','resMaxRange','fR1','fR60','fEIRP','fRange','fLOS','fObs'].forEach(id=>document.getElementById(id).innerText='--');
  document.getElementById('losBar').style.display='none';
  document.getElementById('tcWrap').style.display='none';
  document.getElementById('presetSites').value='';
  document.getElementById('aiResult').style.display='none';
  undoStack=[];updateUndoBtn();
  clearAllHeatmaps();
  if(currentMode==='p2mp')clearP2MPAll();
}
function clearLayers(){if(lyr.f1){map.removeLayer(lyr.f1);lyr.f1=null;}if(lyr.f60){map.removeLayer(lyr.f60);lyr.f60=null;}lyr.blk.forEach(l=>map.removeLayer(l));lyr.blk=[];}

function calculateLink(){
  if(markers.length<2||!markers[0]||!markers[1])return;
  const pA=markers[0].getLatLng(),pB=markers[1].getLatLng();
  const dKm=map.distance(pA,pB)/1000;
  document.getElementById('distance').value=dKm.toFixed(3);
  const fGHz=cvtF(parseFloat(document.getElementById('frequency').value),document.getElementById('frequencyUnit').value);
  const txDbm=cvtP(parseFloat(document.getElementById('txPower').value),document.getElementById('powerUnit').value);
  const gain=parseFloat(document.getElementById('antennaGain').value);
  const loss=parseFloat(document.getElementById('cableLoss').value);
  const bHz=cvtB(parseFloat(document.getElementById('bandwidth').value),document.getElementById('bandwidthUnit').value);
  if(!fGHz||!bHz||dKm<=0)return;
  const fspl=92.44+20*Math.log10(dKm)+20*Math.log10(fGHz);
  const rsl=txDbm+gain-loss-fspl;
  const nf=-174+10*Math.log10(bHz);
  const snr=rsl-nf;
  const eirp=txDbm+gain-loss;
  const maxFSPL=eirp-(-90)-10;
  let maxR=Math.pow(10,(maxFSPL-92.44-20*Math.log10(fGHz))/20);
  if(!isFinite(maxR)||maxR>500)maxR=500;maxR=Math.max(maxR,0.1);
  document.getElementById('resFSPL').innerText=fspl.toFixed(1)+' dB';
  document.getElementById('resRSL').innerText=rsl.toFixed(1)+' dBm';
  document.getElementById('resNoise').innerText=nf.toFixed(1)+' dBm';
  document.getElementById('resSNR').innerText=snr.toFixed(1)+' dB';
  document.getElementById('resEIRP').innerText=eirp.toFixed(1)+' dBm';
  document.getElementById('resMaxRange').innerText=maxR.toFixed(2)+' km';
  const qb=document.getElementById('qualityBox');
  if(snr>=25){qb.innerText=' Excellent Link';qb.className='status-bar excellent';}
  else if(snr>=15){qb.innerText=' Good Link';qb.className='status-bar good';}
  else{qb.innerText=' Poor Link';qb.className='status-bar poor';}
  if(polyline)map.removeLayer(polyline);
  polyline=L.polyline([pA,pB],{color:snr>=15?'#2563eb':'#ef4444',weight:4,dashArray:snr>=25?null:snr>=15?null:'8,4'}).addTo(map);
  const aHA=parseFloat(document.getElementById('antHA').value)||30;
  const aHB=parseFloat(document.getElementById('antHB').value)||30;
  runAnalysis(pA,pB,dKm,fGHz,txDbm,gain,loss,aHA,aHB,maxR,eirp);
}

function interp(A,B,t){return L.latLng(A.lat+(B.lat-A.lat)*t,A.lng+(B.lng-A.lng)*t);}
function brg(A,B){const f1=A.lat*Math.PI/180,l1=A.lng*Math.PI/180,f2=B.lat*Math.PI/180,l2=B.lng*Math.PI/180;return Math.atan2(Math.sin(l2-l1)*Math.cos(f2),Math.cos(f1)*Math.sin(f2)-Math.sin(f1)*Math.cos(f2)*Math.cos(l2-l1));}
function dest(la,ln,b,d){const R=6371000,f=la*Math.PI/180,l=ln*Math.PI/180,dd=d/R;const f2=Math.asin(Math.sin(f)*Math.cos(dd)+Math.cos(f)*Math.sin(dd)*Math.cos(b));const l2=l+Math.atan2(Math.sin(b)*Math.sin(dd)*Math.cos(f),Math.cos(dd)-Math.sin(f)*Math.sin(f2));return[f2*180/Math.PI,l2*180/Math.PI];}
function frR1(t,d,lam){if(t<=0||t>=1)return 0;return Math.sqrt(lam*(t*d)*((1-t)*d)/d);}

function fresnelPoly(A,B,dM,lam,sc){const N=64,b=brg(A,B),u=[],lo=[];for(let i=0;i<=N;i++){const t=i/N,m=interp(A,B,t),r=frR1(t,dM,lam)*sc;u.push(dest(m.lat,m.lng,b+Math.PI/2,r));lo.push(dest(m.lat,m.lng,b-Math.PI/2,r));}return u.concat(lo.slice().reverse());}
function findBlocked(el,hA,hB,dM,lam){const n=el.length,out=[];let inB=false,bS=0;for(let i=0;i<n;i++){const t=i/(n-1),los=hA+(hB-hA)*t,need=los-frR1(t,dM,lam)*0.6;if(el[i]>need){if(!inB){inB=true;bS=t;}}else{if(inB){inB=false;out.push({s:bS,e:t});}}}if(inB)out.push({s:bS,e:1});return out;}
function drawBlocked(A,B,dM,lam,segs){const b=brg(A,B);segs.forEach(seg=>{const u=[],lo=[];for(let i=0;i<=12;i++){const t=seg.s+(seg.e-seg.s)*i/12,m=interp(A,B,t),r=frR1(t,dM,lam);u.push(dest(m.lat,m.lng,b+Math.PI/2,r));lo.push(dest(m.lat,m.lng,b-Math.PI/2,r));}const p=L.polygon(u.concat(lo.slice().reverse()),{color:'#ef4444',weight:1.5,fillColor:'#ef4444',fillOpacity:.30,dashArray:'4,3'}).addTo(map);p.bindTooltip('Terrain obstruction',{sticky:true});lyr.blk.push(p);});}

function drawTerrainChart(el,hA,hB,dM,lam,blk){
  const wrap=document.getElementById('tcWrap'),cvs=document.getElementById('tcCanvas');
  wrap.style.display='block';
  const W=wrap.clientWidth||340,H=110;
  cvs.width=W*devicePixelRatio;cvs.height=H*devicePixelRatio;cvs.style.width=W+'px';cvs.style.height=H+'px';
  const c=cvs.getContext('2d');c.scale(devicePixelRatio,devicePixelRatio);
  const n=el.length,minE=Math.min(...el,hA,hB),maxE=Math.max(...el,hA,hB);
  const pad={t:10,b:22,l:42,r:8},cW=W-pad.l-pad.r,cH=H-pad.t-pad.b;
  const xp=i=>pad.l+(i/(n-1))*cW,yp=a=>pad.t+cH-((a-minE)/(maxE-minE+1))*cH;
  c.fillStyle='#f8fafc';c.fillRect(0,0,W,H);
  c.beginPath();for(let i=0;i<n;i++){const t=i/(n-1),r=frR1(t,dM,lam)*.6,l=hA+(hB-hA)*t;if(!i)c.moveTo(xp(i),yp(l+r));else c.lineTo(xp(i),yp(l+r));}
  for(let i=n-1;i>=0;i--){const t=i/(n-1),r=frR1(t,dM,lam)*.6,l=hA+(hB-hA)*t;c.lineTo(xp(i),yp(l-r));}
  c.closePath();c.fillStyle='rgba(34,197,94,.18)';c.fill();
  c.beginPath();c.moveTo(xp(0),yp(hA));c.lineTo(xp(n-1),yp(hB));c.strokeStyle='#2563eb';c.lineWidth=1.5;c.setLineDash([5,3]);c.stroke();c.setLineDash([]);
  c.beginPath();c.moveTo(xp(0),yp(minE));for(let i=0;i<n;i++)c.lineTo(xp(i),yp(el[i]));c.lineTo(xp(n-1),yp(minE));c.closePath();c.fillStyle='rgba(120,100,60,.55)';c.fill();c.strokeStyle='#78634a';c.lineWidth=1.5;c.stroke();
  blk.forEach(seg=>{c.fillStyle='rgba(239,68,68,.25)';c.fillRect(pad.l+seg.s*cW,pad.t,(seg.e-seg.s)*cW,cH);});
  c.fillStyle='#94a3b8';c.font='9px sans-serif';c.textAlign='right';
  c.fillText(Math.round(maxE)+'m',pad.l-3,pad.t+6);c.fillText(Math.round(minE)+'m',pad.l-3,H-pad.b+2);
  c.textAlign='center';c.fillText('A',xp(0),H-4);c.fillText('B',xp(n-1),H-4);c.fillText('← Terrain Profile →',W/2,H-4);
  if(elevApiOk===false){c.fillStyle='rgba(254,226,226,.85)';c.fillRect(pad.l,pad.t,cW,18);c.fillStyle='#991b1b';c.font='bold 9px sans-serif';c.fillText('Flat terrain — elevation API offline',W/2,pad.t+12);}
}

function setProgress(p){document.getElementById('progBar').style.width=p+'%';}
function spin(on,msg=''){document.getElementById('spnWrap').style.display=on?'flex':'none';if(msg)document.getElementById('spnMsg').innerText=msg;if(!on)setProgress(0);}

async function runAnalysis(pA,pB,dKm,fGHz,txDbm,gain,loss,aHA,aHB,maxR,eirp){
  const dM=dKm*1000,lam=3e8/(fGHz*1e9);
  const r1=Math.sqrt(lam*(dM/2)*(dM/2)/dM),r60=r1*0.6;
  document.getElementById('fR1').innerText=r1.toFixed(1)+' m';
  document.getElementById('fR60').innerText=r60.toFixed(1)+' m';
  document.getElementById('fEIRP').innerText=eirp.toFixed(1)+' dBm';
  document.getElementById('fRange').innerText=maxR.toFixed(1)+' km';
  clearLayers();
  if(document.getElementById('tog1').checked){const pts=fresnelPoly(pA,pB,dM,lam,1.0);lyr.f1=L.polygon(pts,{color:'#ca8a04',weight:2,fillColor:'#eab308',fillOpacity:.14,dashArray:'5,4'}).addTo(map);lyr.f1.bindTooltip('1st Fresnel Zone — max r: '+r1.toFixed(1)+' m',{sticky:true});}
  if(document.getElementById('tog60').checked){const pts=fresnelPoly(pA,pB,dM,lam,0.6);lyr.f60=L.polygon(pts,{color:'#16a34a',weight:2,fillColor:'#22c55e',fillOpacity:.10,dashArray:'4,3'}).addTo(map);lyr.f60.bindTooltip('60% Clearance — r: '+r60.toFixed(1)+' m',{sticky:true});}
  spin(true,'Fetching terrain…');setProgress(10);
  const el=await fetchElev(pA,pB,60);
  const hA=el[0]+aHA,hB=el[59]+aHB;
  const blk=findBlocked(el,hA,hB,dM,lam);
  setProgress(90);
  if(document.getElementById('togBlk').checked)drawBlocked(pA,pB,dM,lam,blk);
  drawTerrainChart(el,hA,hB,dM,lam,blk);
  setProgress(100);spin(false);
  const lb=document.getElementById('losBar');lb.style.display='block';
  const frac=blk.reduce((s,b)=>s+(b.e-b.s),0);
  if(blk.length===0){lb.className='los-bar los-ok';lb.innerText=' Line-of-Sight Clear — 60% Fresnel unobstructed';document.getElementById('fLOS').innerText=' Clear';}
  else if(frac<0.15){lb.className='los-bar los-warn';lb.innerText=' Marginal LOS — partial obstruction';document.getElementById('fLOS').innerText=' Marginal';}
  else{lb.className='los-bar los-bad';lb.innerText=' Fresnel Blocked — expect significant loss';document.getElementById('fLOS').innerText=' Blocked';}
  document.getElementById('fObs').innerText=blk.length===0?'None ':blk.length+' zone(s)';
  CACHE={pA,pB,dM,lam,r1,r60,blk,el,hA,hB,aHA,aHB};
}

function redrawAll(){clearLayers();if(!CACHE)return;const{pA,pB,dM,lam,r1,r60,blk}=CACHE;if(document.getElementById('tog1').checked){const pts=fresnelPoly(pA,pB,dM,lam,1.0);lyr.f1=L.polygon(pts,{color:'#ca8a04',weight:2,fillColor:'#eab308',fillOpacity:.14,dashArray:'5,4'}).addTo(map);lyr.f1.bindTooltip('1st Fresnel Zone — max r: '+r1.toFixed(1)+' m',{sticky:true});}if(document.getElementById('tog60').checked){const pts=fresnelPoly(pA,pB,dM,lam,0.6);lyr.f60=L.polygon(pts,{color:'#16a34a',weight:2,fillColor:'#22c55e',fillOpacity:.10,dashArray:'4,3'}).addTo(map);lyr.f60.bindTooltip('60% Clearance — r: '+r60.toFixed(1)+' m',{sticky:true});}if(document.getElementById('togBlk').checked&&blk)drawBlocked(pA,pB,dM,lam,blk);}

const presetPairs=[{a:[24.7136,46.6753],b:[24.7743,46.7386]},{a:[26.4207,50.0888],b:[26.3927,49.9777]},{a:[21.4858,39.1925],b:[21.5433,39.1728]},{a:[24.5247,39.5692],b:[24.4672,39.6111]},{a:[28.3838,36.5662],b:[28.3998,36.5789]},{a:[18.2465,42.5117],b:[18.2164,42.5053]},{a:[17.5650,44.2289],b:[17.6101,44.4192]}];
function loadPresetSites(){const idx=document.getElementById('presetSites').value;if(idx==='')return;const s=presetPairs[idx],A=L.latLng(s.a[0],s.a[1]),B=L.latLng(s.b[0],s.b[1]);addOrUpdateMarker(0,A);addOrUpdateMarker(1,B);updateInputs();if(previewLine){map.removeLayer(previewLine);previewLine=null;}calculateLink();map.fitBounds(L.latLngBounds([A,B]),{padding:[50,50]});}

(async()=>{setElevStatus('warn',' Elevation: checking…');try{await ELEV_APIS[0]([{latitude:24.7,longitude:46.7}]);setElevStatus('ok',' Elevation: ready');elevApiOk=true;}catch{setElevStatus('warn',' Elevation: using fallback');elevApiOk=null;}})();

// ═══ MODE TOGGLE ═══
let currentMode='p2p',p2mpHub=null,p2mpRemotes=[],p2mpRemoteCounter=0,undoStack=[];
const REMOTE_COLORS=['#7c3aed','#d97706','#0891b2','#be185d','#4d7c0f','#dc2626','#0369a1','#b45309'];

function setMode(mode){
  currentMode=mode;
  document.getElementById('btnP2P').className='mode-btn'+(mode==='p2p'?' active':'');
  document.getElementById('btnP2MP').className='mode-btn'+(mode==='p2mp'?' active':'');
  // Update analyze button text
  const aBtn=document.getElementById('analyzeBtn');
  aBtn.innerText=mode==='p2mp'?'🤖 Analyze All Links':'🤖 Analyze This Link';
  document.getElementById('aiResult').style.display='none';
  document.getElementById('aiThinking').style.display='none';
  if(mode==='p2mp'){
    document.getElementById('p2pSitesSection').style.display='none';
    document.getElementById('p2mpSitesSection').style.display='block';
    document.getElementById('qualityBox').style.display='none';
    document.querySelector('.results-grid').style.display='none';
    document.getElementById('p2mpActiveBanner').style.display='none';
    document.getElementById('labelAntA').innerHTML='Antenna Ht — <strong>Hub</strong>';
    document.getElementById('labelAntB').innerHTML='Antenna Ht — <strong>Remote</strong>';
    markers.forEach(m=>{if(m)map.removeLayer(m);});markers=[];
    if(polyline){map.removeLayer(polyline);polyline=null;}
    if(previewLine){map.removeLayer(previewLine);previewLine=null;}
    clearLayers();undoStack=[];
  } else {
    document.getElementById('p2pSitesSection').style.display='block';
    document.getElementById('p2mpSitesSection').style.display='none';
    document.getElementById('qualityBox').style.display='block';
    document.querySelector('.results-grid').style.display='grid';
    document.getElementById('p2mpActiveBanner').style.display='none';
    document.getElementById('labelAntA').textContent='Antenna Ht A';
    document.getElementById('labelAntB').textContent='Antenna Ht B';
    clearP2MPAll();undoStack=[];
  }
  updateUndoBtn();
}

function resetP2MPUI(){
  ['hubPlacedBadge','remotesContainer'].forEach(id=>{const el=document.getElementById(id);if(el)el.style.display='none';});
  const rl=document.getElementById('remoteRowsList');if(rl)rl.innerHTML='';
  const hc=document.getElementById('hubCoord');if(hc)hc.value='';
  p2mpRemoteCounter=0;undoStack=[];
}
function clearP2MP(){if(p2mpHub){map.removeLayer(p2mpHub);p2mpHub=null;}p2mpRemotes.forEach(r=>{if(r.marker)map.removeLayer(r.marker);if(r.line)map.removeLayer(r.line);});p2mpRemotes=[];}
function clearP2MPAll(){clearP2MP();resetP2MPUI();undoStack=[];updateUndoBtn();}

function handleP2MPClick(ll){
  if(!p2mpHub){placeP2MPHub(ll);}
  else{const pend=p2mpRemotes.find(r=>!r.placed);if(pend){placeP2MPRemote(ll,pend.id);}else{addRemoteRow();const np=p2mpRemotes.find(r=>!r.placed);if(np)placeP2MPRemote(ll,np.id);}}
  updateUndoBtn();
}

function placeHubFromInput(){const v=document.getElementById('hubCoord').value.trim().split(',');if(v.length!==2)return;const la=parseFloat(v[0]),ln=parseFloat(v[1]);if(isNaN(la)||isNaN(ln))return;placeP2MPHub(L.latLng(la,ln));map.setView([la,ln],12);updateUndoBtn();}

let activeRemoteInputId=null;

function addRemoteRow(){
  p2mpRemoteCounter++;
  const id=p2mpRemoteCounter,color=REMOTE_COLORS[(id-1)%REMOTE_COLORS.length];
  const div=document.createElement('div');div.className='remote-row';div.id='remoteRow_'+id;
  div.innerHTML=`<div class="remote-row-header" style="cursor:pointer;" onclick="showRemoteResults(${id})">
    <span class="remote-dot" style="background:${color};"></span>
    <span class="remote-row-label">Remote ${id}</span>
    <span id="remoteRowQuality_${id}" style="font-size:.62rem;font-weight:700;padding:1px 6px;border-radius:8px;background:#e2e8f0;color:#64748b;">—</span>
    <button class="remote-rm" onclick="event.stopPropagation();removeRemoteById(${id})">✕</button>
  </div>
  <input id="remoteInput_${id}" placeholder="Type: Lat, Lon  OR  click map" onchange="placeRemoteFromInput(${id})"/>
  <div id="remoteInfo_${id}" style="display:none;margin-top:4px;padding:4px 7px;background:white;border:1px solid #e2e8f0;border-radius:6px;font-size:.68rem;color:#475569;line-height:1.7;"></div>`;
  document.getElementById('remoteRowsList').appendChild(div);
  activeRemoteInputId=id;
  p2mpRemotes.push({id,marker:null,line:null,color,placed:false});
}

function placeP2MPHub(ll){
  if(p2mpHub)map.removeLayer(p2mpHub);
  const icon=L.divIcon({className:'',html:`<div style="width:36px;height:36px;border-radius:10px;background:#1e40af;border:3px solid white;box-shadow:0 2px 12px rgba(0,0,0,.45);display:flex;align-items:center;justify-content:center;color:white;font-weight:800;font-size:11px;">HUB</div>`,iconSize:[36,36],iconAnchor:[18,18]});
  p2mpHub=L.marker(ll,{draggable:true,icon}).addTo(map);
  p2mpHub.bindTooltip('Hub',{permanent:true,direction:'top',offset:[0,-20]});
  p2mpHub.on('dragend',()=>{p2mpRemotes.forEach(r=>{if(r.line&&r.marker){map.removeLayer(r.line);r.line=L.polyline([p2mpHub.getLatLng(),r.marker.getLatLng()],{color:r.color,weight:2.5,dashArray:'7,5',opacity:.85}).addTo(map);}});document.getElementById('hubCoord').value=p2mpHub.getLatLng().lat.toFixed(4)+', '+p2mpHub.getLatLng().lng.toFixed(4);recalcAllRemotes();});
  document.getElementById('hubCoord').value=ll.lat.toFixed(4)+', '+ll.lng.toFixed(4);
  document.getElementById('hubPlacedBadge').style.display='block';
  document.getElementById('remotesContainer').style.display='block';
  if(p2mpRemotes.length===0)addRemoteRow();
  undoStack.push({type:'hub'});
  const info=document.getElementById('p2mpSelectedInfo');if(info)info.style.display='block';
  document.getElementById('p2mpActiveBanner').style.display='none';
  document.getElementById('qualityBox').style.display='none';
  document.querySelector('.results-grid').style.display='none';
}

function placeP2MPRemote(ll,targetId){
  let id=targetId||activeRemoteInputId;
  let ex=p2mpRemotes.find(r=>r.id===id);
  if(!ex){addRemoteRow();id=p2mpRemoteCounter;ex=p2mpRemotes.find(r=>r.id===id);}
  if(!ex)return;
  const color=ex.color;
  if(ex.marker)map.removeLayer(ex.marker);if(ex.line)map.removeLayer(ex.line);
  const line=L.polyline([p2mpHub.getLatLng(),ll],{color,weight:2.5,dashArray:'7,5',opacity:.85}).addTo(map);
  const icon=L.divIcon({className:'',html:`<div style="width:28px;height:28px;border-radius:50%;background:${color};border:3px solid white;box-shadow:0 2px 8px rgba(0,0,0,.4);display:flex;align-items:center;justify-content:center;color:white;font-weight:700;font-size:11px;">R${id}</div>`,iconSize:[28,28],iconAnchor:[14,14]});
  const marker=L.marker(ll,{draggable:true,icon,interactive:true}).addTo(map);
  marker.bindTooltip(`Remote ${id}`,{permanent:true,direction:'top',offset:[0,-18]});
  marker.on('click',(e)=>{L.DomEvent.stopPropagation(e);showRemoteResults(id);});
  marker.on('dragend',()=>{const r2=p2mpRemotes.find(x=>x.id===id);if(r2&&r2.line)map.removeLayer(r2.line);if(r2){r2.line=L.polyline([p2mpHub.getLatLng(),marker.getLatLng()],{color,weight:2.5,dashArray:'7,5',opacity:.85}).addTo(map);const ll2=marker.getLatLng();const inp=document.getElementById('remoteInput_'+id);if(inp)inp.value=ll2.lat.toFixed(4)+', '+ll2.lng.toFixed(4);showRemoteResults(id);}});
  ex.marker=marker;ex.line=line;ex.placed=true;ex.latlng=ll;
  const inp=document.getElementById('remoteInput_'+id);if(inp)inp.value=ll.lat.toFixed(4)+', '+ll.lng.toFixed(4);
  undoStack.push({type:'remote',id});
  const np=p2mpRemotes.find(r=>!r.placed);activeRemoteInputId=np?np.id:null;
  showRemoteResults(id);
  const info=document.getElementById('p2mpSelectedInfo');if(info)info.style.display='none';
}

function placeRemoteFromInput(id){const v=document.getElementById('remoteInput_'+id)?.value.trim().split(',');if(!v||v.length!==2)return;const la=parseFloat(v[0]),ln=parseFloat(v[1]);if(isNaN(la)||isNaN(ln))return;placeP2MPRemote(L.latLng(la,ln),id);map.setView([la,ln],12);}
function removeRemoteById(id){const r=p2mpRemotes.find(x=>x.id===id);if(r){if(r.marker)map.removeLayer(r.marker);if(r.line)map.removeLayer(r.line);p2mpRemotes=p2mpRemotes.filter(x=>x.id!==id);undoStack=undoStack.filter(u=>!(u.type==='remote'&&u.id===id));}const row=document.getElementById('remoteRow_'+id);if(row)row.remove();updateUndoBtn();}
function renderP2MPResultCards(){}
function recalcAllRemotes(){if(window._activeRemoteId!=null)showRemoteResults(window._activeRemoteId);}

function undoLast(){
  if(!undoStack.length)return;const last=undoStack.pop();
  if(last.type==='remote'){const r=p2mpRemotes.find(x=>x.id===last.id);if(r){if(r.marker)map.removeLayer(r.marker);if(r.line)map.removeLayer(r.line);r.marker=null;r.line=null;r.placed=false;r.results=null;const inp=document.getElementById('remoteInput_'+last.id);if(inp)inp.value='';}}
  else if(last.type==='hub'){clearP2MP();resetP2MPUI();}
  else if(last.type==='p2p_b'){if(markers[1]){map.removeLayer(markers[1]);markers.splice(1,1);}if(polyline){map.removeLayer(polyline);polyline=null;}clearLayers();document.getElementById('coordB').value='';document.getElementById('distance').value='';document.getElementById('qualityBox').className='status-bar';document.getElementById('qualityBox').innerText='📍 Click two points on the map to begin';['resFSPL','resRSL','resNoise','resSNR','resEIRP','resMaxRange'].forEach(id=>document.getElementById(id).innerText='--');document.getElementById('losBar').style.display='none';document.getElementById('tcWrap').style.display='none';CACHE=null;}
  else if(last.type==='p2p_a'){if(markers[0]){map.removeLayer(markers[0]);markers.splice(0,1);}document.getElementById('coordA').value='';}
  updateUndoBtn();
}
function updateUndoBtn(){const has=undoStack.length>0;['undoBtn','undoBtnP2MP'].forEach(bid=>{const btn=document.getElementById(bid);if(!btn)return;btn.disabled=!has;btn.style.opacity=has?'1':'0.4';btn.style.pointerEvents=has?'auto':'none';});}

function showRemoteResults(id){
  window._activeRemoteId=id;
  const r=p2mpRemotes.find(x=>x.id===id);
  if(!r||!r.placed||!p2mpHub)return;
  p2mpRemotes.forEach(rx=>{if(rx.line)rx.line.setStyle({weight:rx.id===id?4:2,opacity:rx.id===id?1:.45,dashArray:rx.id===id?null:'7,5'});});
  const pA=p2mpHub.getLatLng(),pB=r.marker.getLatLng();
  const dKm=map.distance(pA,pB)/1000;
  const fGHz=cvtF(parseFloat(document.getElementById('frequency').value),document.getElementById('frequencyUnit').value);
  const txDbm=cvtP(parseFloat(document.getElementById('txPower').value),document.getElementById('powerUnit').value);
  const gain=parseFloat(document.getElementById('antennaGain').value)||23;
  const loss=parseFloat(document.getElementById('cableLoss').value)||2;
  const bHz=cvtB(parseFloat(document.getElementById('bandwidth').value),document.getElementById('bandwidthUnit').value);
  const aHA=parseFloat(document.getElementById('antHA').value)||30;
  const aHB=parseFloat(document.getElementById('antHB').value)||30;
  if(!fGHz||!bHz||dKm<=0)return;
  const fspl=92.44+20*Math.log10(dKm)+20*Math.log10(fGHz);
  const rsl=txDbm+gain-loss-fspl,nf=-174+10*Math.log10(bHz),snr=rsl-nf,eirp=txDbm+gain-loss;
  const maxFSPL=eirp-(-90)-10;let maxR=Math.pow(10,(maxFSPL-92.44-20*Math.log10(fGHz))/20);
  if(!isFinite(maxR)||maxR>500)maxR=500;maxR=Math.max(maxR,0.1);
  document.getElementById('distance').value=dKm.toFixed(3);
  document.getElementById('resFSPL').innerText=fspl.toFixed(1)+' dB';
  document.getElementById('resRSL').innerText=rsl.toFixed(1)+' dBm';
  document.getElementById('resNoise').innerText=nf.toFixed(1)+' dBm';
  document.getElementById('resSNR').innerText=snr.toFixed(1)+' dB';
  document.getElementById('resEIRP').innerText=eirp.toFixed(1)+' dBm';
  document.getElementById('resMaxRange').innerText=maxR.toFixed(2)+' km';
  const qb=document.getElementById('qualityBox');
  if(snr>=25){qb.innerText=' Excellent Link';qb.className='status-bar excellent';}
  else if(snr>=15){qb.innerText=' Good Link';qb.className='status-bar good';}
  else{qb.innerText=' Poor Link';qb.className='status-bar poor';}
  const qLabel=document.getElementById('remoteRowQuality_'+id);
  if(qLabel){const qt=snr>=25?'Excellent':snr>=15?'Good':'Poor',qbg=snr>=25?'#dcfce7':snr>=15?'#dbeafe':'#fee2e2',qc=snr>=25?'#166534':snr>=15?'#1e40af':'#991b1b';qLabel.textContent=qt;qLabel.style.background=qbg;qLabel.style.color=qc;}
  const ib=document.getElementById('remoteInfo_'+id);
  if(ib){ib.style.display='block';ib.innerHTML=`<span style="color:#94a3b8;">Coords:</span> ${pB.lat.toFixed(4)}, ${pB.lng.toFixed(4)} &nbsp;&nbsp;<span style="color:#94a3b8;">Distance:</span> <strong style="color:#0f172a;">${dKm.toFixed(3)} km</strong>`;}
  document.querySelectorAll('.remote-row').forEach(row=>{row.style.borderColor=row.id==='remoteRow_'+id?r.color:'#e2e8f0';row.style.background=row.id==='remoteRow_'+id?r.color+'10':'#f8fafc';});
  document.getElementById('labelAntA').innerHTML='Antenna Ht — <strong>Hub</strong>';
  document.getElementById('labelAntB').innerHTML=`<span style="display:inline-block;width:8px;height:8px;border-radius:50%;background:${r.color};margin-right:4px;vertical-align:middle;"></span>Antenna Ht — <strong style="color:${r.color};">Remote ${id}</strong>`;
  const banner=document.getElementById('p2mpActiveBanner');
  banner.style.display='block';banner.style.borderColor=r.color;banner.style.background=r.color+'18';banner.style.color=r.color;
  banner.innerHTML=`<span style="display:inline-block;width:10px;height:10px;border-radius:50%;background:${r.color};margin-right:6px;vertical-align:middle;"></span>Remote ${id} ↔ Hub &nbsp;·&nbsp; ${dKm.toFixed(2)} km`;
  document.getElementById('qualityBox').style.display='block';
  document.querySelector('.results-grid').style.display='grid';
  const info=document.getElementById('p2mpSelectedInfo');if(info)info.style.display='none';
  runAnalysis(pA,pB,dKm,fGHz,txDbm,gain,loss,aHA,aHB,maxR,eirp);
  setTimeout(()=>{document.getElementById('qualityBox').scrollIntoView({behavior:'smooth',block:'nearest'});},200);
}

// ═══ AI ═══
function handleAnalyzeBtn(){
  if(currentMode==='p2mp') analyzeAllRemotes();
  else analyzeWithAI();
}

async function analyzeWithAI(){
  const fspl=document.getElementById('resFSPL').innerText;
  if(fspl==='--'){alert('Calculate a link first!');return;}
  const btn=document.getElementById('analyzeBtn'),thinking=document.getElementById('aiThinking'),result=document.getElementById('aiResult');
  btn.disabled=true;thinking.style.display='block';result.style.display='none';
  const prompt=`RF Link Analysis:
- Distance: ${document.getElementById('distance').value} km
- Frequency: ${document.getElementById('frequency').value} ${document.getElementById('frequencyUnit').value}
- FSPL: ${document.getElementById('resFSPL').innerText}
- RSL: ${document.getElementById('resRSL').innerText}
- SNR: ${document.getElementById('resSNR').innerText}
- EIRP: ${document.getElementById('resEIRP').innerText}
- Link Quality: ${document.getElementById('qualityBox').innerText.trim()}
- LOS Status: ${document.getElementById('fLOS').innerText}
Provide: 1) Assessment 2) Improvement recommendations 3) Warnings`;
  try{
    const response=await fetch('https://rf-ai.anood-j12.workers.dev',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({prompt})});
    const data=await response.json();
    const text=data?.content?.[0]?.text||JSON.stringify(data);
    result.innerHTML=text
      .replace(/^## (.*?)$/gm,'<h3 style="margin:10px 0 5px;color:#4c1d95;font-size:.82rem;font-weight:700;">$1</h3>')
      .replace(/^- (.*?)$/gm,'<div style="display:flex;gap:6px;margin:3px 0;"><span style="color:#7c3aed;flex-shrink:0;">•</span><span>$1</span></div>')
      .replace(/⚠️ (.*?)$/gm,'<div style="background:#fef9c3;border:1px solid #fde68a;border-radius:6px;padding:5px 8px;margin:4px 0;font-size:.75rem;color:#854d0e;">⚠️ $1</div>')
      .replace(/\*\*(.*?)\*\*/g,'<strong>$1</strong>')
      .replace(/\n/g,'');
    result.style.display='block';
  }catch(err){result.innerHTML='⚠️ Error: '+err.message;result.style.display='block';}
  thinking.style.display='none';btn.disabled=false;
}

async function analyzeAllRemotes(){
  const placed=p2mpRemotes.filter(r=>r.placed&&r.marker);
  if(!placed.length||!p2mpHub){alert('Place a hub and at least one remote first!');return;}
  const btn=document.getElementById('analyzeBtn'),thinking=document.getElementById('aiThinking'),result=document.getElementById('aiResult');
  btn.disabled=true;thinking.style.display='block';result.style.display='none';
  const fGHz=cvtF(parseFloat(document.getElementById('frequency').value),document.getElementById('frequencyUnit').value);
  const txDbm=cvtP(parseFloat(document.getElementById('txPower').value),document.getElementById('powerUnit').value);
  const gain=parseFloat(document.getElementById('antennaGain').value)||23;
  const loss=parseFloat(document.getElementById('cableLoss').value)||2;
  const bHz=cvtB(parseFloat(document.getElementById('bandwidth').value),document.getElementById('bandwidthUnit').value);
  const summaries=placed.map(r=>{
    const pA=p2mpHub.getLatLng(),pB=r.marker.getLatLng();
    const dKm=map.distance(pA,pB)/1000;
    const fspl=92.44+20*Math.log10(dKm)+20*Math.log10(fGHz);
    const rsl=txDbm+gain-loss-fspl,nf=-174+10*Math.log10(bHz),snr=rsl-nf;
    const q=snr>=25?'Excellent':snr>=15?'Good':'Poor';
    return `Remote ${r.id}: distance=${dKm.toFixed(2)}km, SNR=${snr.toFixed(1)}dB, RSL=${rsl.toFixed(1)}dBm, FSPL=${fspl.toFixed(1)}dB, quality=${q}`;
  }).join('\n');
  const prompt=`You are an expert RF engineer reviewing a Point-to-Multipoint network.
Hub: Frequency=${document.getElementById('frequency').value} ${document.getElementById('frequencyUnit').value}, Tx Power=${document.getElementById('txPower').value} ${document.getElementById('powerUnit').value}, Antenna Gain=${gain}dBi, Cable Loss=${loss}dB

Link results for all ${placed.length} remote sites:
${summaries}

Structure your response EXACTLY like this:

## 1) Assessment
- Overall network status and SNR for each remote

## 2) Improvement Recommendations
- Specific fixes for weakest links
- Hardware upgrades needed

## 3) Warnings
- ⚠️ Critical deployment risks
- ⚠️ Links that must not be deployed

Use ## headers and - bullets only. Keep it under 250 words.`;
  try{
    const response=await fetch('https://rf-ai.anood-j12.workers.dev',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({prompt})});
    const data=await response.json();
    const text=data?.content?.[0]?.text||JSON.stringify(data);
    const cleanText=text.replace(/##/g,'\n##').replace(/- /g,'\n- ').replace(/⚠️/g,'\n⚠️');
    result.innerHTML=cleanText
      .replace(/^## (.*?)$/gm,'<h3 style="margin:10px 0 5px;color:#4c1d95;font-size:.82rem;font-weight:700;">$1</h3>')
      .replace(/^- (.*?)$/gm,'<div style="display:flex;gap:6px;margin:3px 0;"><span style="color:#7c3aed;flex-shrink:0;">•</span><span>$1</span></div>')
      .replace(/⚠️ (.*?)$/gm,'<div style="background:#fef9c3;border:1px solid #fde68a;border-radius:6px;padding:5px 8px;margin:4px 0;font-size:.75rem;color:#854d0e;">⚠️ $1</div>')
      .replace(/\*\*(.*?)\*\*/g,'<strong>$1</strong>')
      .replace(/\n(?!<)/g,'');
    result.style.display='block';
  }catch(err){result.innerHTML='⚠️ Error: '+err.message;result.style.display='block';}
  thinking.style.display='none';btn.disabled=false;
}

// ═══ HEATMAP ═══
let heatmapCircles = [];
let heatmapMode = false;

function toggleHeatmapMode() {
  heatmapMode = document.getElementById('togHeatmap').checked;
  document.getElementById('heatmapLegend').style.display = heatmapMode ? 'block' : 'none';
  if (!heatmapMode) clearAllHeatmaps();
}

function clearAllHeatmaps() {
  heatmapCircles.forEach(group => group.forEach(c => map.removeLayer(c)));
  heatmapCircles = [];
}

function drawHeatmap(center) {
  const fGHz = cvtF(parseFloat(document.getElementById('frequency').value), document.getElementById('frequencyUnit').value);
  const txDbm = cvtP(parseFloat(document.getElementById('txPower').value), document.getElementById('powerUnit').value);
  const gain = parseFloat(document.getElementById('antennaGain').value) || 23;
  const loss = parseFloat(document.getElementById('cableLoss').value) || 2;
  const bHz = cvtB(parseFloat(document.getElementById('bandwidth').value), document.getElementById('bandwidthUnit').value);
  const rxSens = -90; // receiver sensitivity dBm

  if (!fGHz || !bHz) return;

  // Calculate max range for each quality level
  // SNR = RSL - NF, RSL = txDbm + gain - loss - FSPL
  // FSPL = 92.44 + 20log(d) + 20log(f)
  // Coverage = RSL - rxSens
  const nf = -174 + 10 * Math.log10(bHz);
  const eirp = txDbm + gain - loss;

  // Range for SNR thresholds: excellent(25dB), good(15dB), poor(0dB)
  function snrToRange(minSNR) {
    // RSL needed = nf + minSNR
    // FSPL = eirp - RSL_needed
    const maxFSPL = eirp - (nf + minSNR);
    let r = Math.pow(10, (maxFSPL - 92.44 - 20 * Math.log10(fGHz)) / 20);
    if (!isFinite(r) || r > 300) r = 300;
    return Math.max(r, 0.1) * 1000; // meters
  }

  const rExcellent = snrToRange(25);
  const rGood = snrToRange(15);
  const rPoor = snrToRange(0);

  // Check if this center already has a heatmap → remove it
  const existingIdx = heatmapCircles.findIndex(group => {
    if (!group.length) return false;
    const c = group[0].getLatLng();
    return Math.abs(c.lat - center.lat) < 0.0005 && Math.abs(c.lng - center.lng) < 0.0005;
  });

  if (existingIdx !== -1) {
    heatmapCircles[existingIdx].forEach(c => map.removeLayer(c));
    heatmapCircles.splice(existingIdx, 1);
    return;
  }

  const circles = [];

  // Draw from largest to smallest (poor → good → excellent)
  if (rPoor > rGood) {
    circles.push(L.circle(center, {
      radius: rPoor, color: '#ef4444', weight: 1,
      fillColor: '#ef4444', fillOpacity: 0.10
    }).addTo(map).bindTooltip(`Poor coverage — up to ${(rPoor/1000).toFixed(1)} km`, {sticky:true}));
  }

  if (rGood > rExcellent) {
    circles.push(L.circle(center, {
      radius: rGood, color: '#eab308', weight: 1,
      fillColor: '#eab308', fillOpacity: 0.15
    }).addTo(map).bindTooltip(`Good coverage — up to ${(rGood/1000).toFixed(1)} km`, {sticky:true}));
  }

  circles.push(L.circle(center, {
    radius: rExcellent, color: '#22c55e', weight: 1.5,
    fillColor: '#22c55e', fillOpacity: 0.22
  }).addTo(map).bindTooltip(`Excellent coverage — up to ${(rExcellent/1000).toFixed(1)} km`, {sticky:true}));

  heatmapCircles.push(circles);
}

// ═══ EXPORT ═══
async function exportExcel(){
  const mode=currentMode==='p2mp'?'Point-to-Multipoint (P2MP)':'Point-to-Point (P2P)',now=new Date().toLocaleString();
  const fGHz=cvtF(parseFloat(document.getElementById('frequency').value),document.getElementById('frequencyUnit').value);
  const txDbm=cvtP(parseFloat(document.getElementById('txPower').value),document.getElementById('powerUnit').value);
  const gain=parseFloat(document.getElementById('antennaGain').value)||23,loss=parseFloat(document.getElementById('cableLoss').value)||2;
  const bHz=cvtB(parseFloat(document.getElementById('bandwidth').value),document.getElementById('bandwidthUnit').value);
  let rows=[['RF Link Planner — Link Budget Report'],['Generated',now],['Mode',mode],[]];
  rows.push(['RF CONFIGURATION'],['Frequency',document.getElementById('frequency').value+' '+document.getElementById('frequencyUnit').value],['Transmit Power',document.getElementById('txPower').value+' '+document.getElementById('powerUnit').value],['Antenna Gain',gain+' dBi'],['Cable Loss',loss+' dB'],['Bandwidth',document.getElementById('bandwidth').value+' '+document.getElementById('bandwidthUnit').value],['Antenna Height A',document.getElementById('antHA').value+' m'],['Antenna Height B',document.getElementById('antHB').value+' m'],[]);
  if(currentMode==='p2mp'&&p2mpHub){
    const hubLL=p2mpHub.getLatLng();
    rows.push(['HUB INFORMATION'],['Hub Coordinates',hubLL.lat.toFixed(4)+', '+hubLL.lng.toFixed(4)],['Total Remotes',p2mpRemotes.filter(r=>r.placed).length],[]);
    rows.push(['REMOTE LINK BUDGETS'],['Remote','Coordinates','Distance (km)','FSPL (dB)','RSL (dBm)','Noise Floor (dBm)','SNR (dB)','EIRP (dBm)','Max Range (km)','Link Quality']);
    p2mpRemotes.filter(r=>r.placed).forEach(r=>{const pA=hubLL,pB=r.marker.getLatLng(),dKm=map.distance(pA,pB)/1000;if(!fGHz||!bHz||dKm<=0)return;const fspl=92.44+20*Math.log10(dKm)+20*Math.log10(fGHz),rsl=txDbm+gain-loss-fspl,nf=-174+10*Math.log10(bHz),snr=rsl-nf,eirp=txDbm+gain-loss;const maxFSPL=eirp-(-90)-10;let maxR=Math.pow(10,(maxFSPL-92.44-20*Math.log10(fGHz))/20);if(!isFinite(maxR)||maxR>500)maxR=500;maxR=Math.max(maxR,0.1);rows.push(['Remote '+r.id,pB.lat.toFixed(4)+', '+pB.lng.toFixed(4),dKm.toFixed(3),fspl.toFixed(1),rsl.toFixed(1),nf.toFixed(1),snr.toFixed(1),eirp.toFixed(1),maxR.toFixed(2),snr>=25?'Excellent':snr>=15?'Good':'Poor']);});
    rows.push([]);
  } else {
    const pA=markers[0]?markers[0].getLatLng():null,pB=markers[1]?markers[1].getLatLng():null;
    rows.push(['SITE INFORMATION'],['Site A',pA?pA.lat.toFixed(4)+', '+pA.lng.toFixed(4):'--'],['Site B',pB?pB.lat.toFixed(4)+', '+pB.lng.toFixed(4):'--'],['Distance (km)',document.getElementById('distance').value||'--'],[]);
    rows.push(['CALCULATED RESULTS'],['FSPL (dB)',document.getElementById('resFSPL').innerText],['RSL (dBm)',document.getElementById('resRSL').innerText],['Noise Floor (dBm)',document.getElementById('resNoise').innerText],['SNR (dB)',document.getElementById('resSNR').innerText],['EIRP (dBm)',document.getElementById('resEIRP').innerText],['Max Range (km)',document.getElementById('resMaxRange').innerText],['Link Quality',document.getElementById('qualityBox').innerText.trim()],[]);
    rows.push(['FRESNEL & LOS'],['LOS Status',document.getElementById('fLOS').innerText],['Fresnel r₁ Max (m)',document.getElementById('fR1').innerText],['60% Clearance (m)',document.getElementById('fR60').innerText],['Obstructions',document.getElementById('fObs').innerText],[]);
  }
  rows.push(['FORMULAS REFERENCE'],['Formula','Expression','Unit','Notes'],['FSPL','92.44 + 20·log₁₀(d_km) + 20·log₁₀(f_GHz)','dB','Free Space Path Loss'],['RSL','Tx(dBm) + Gain − Loss − FSPL','dBm','Received Signal Level'],['Noise Floor','−174 + 10·log₁₀(BW_Hz)','dBm','Thermal noise'],['SNR','RSL − Noise Floor','dB','≥25 Excellent | 15–25 Good | <15 Poor'],['EIRP','Tx(dBm) + Gain − Loss','dBm','Effective Isotropic Radiated Power']);
  const csv='\uFEFF'+rows.map(r=>r.map(c=>'"'+String(c||'').replace(/"/g,'""')+'"').join(',')).join('\r\n');
  const blob=new Blob([csv],{type:'text/csv;charset=utf-8'}),url=URL.createObjectURL(blob),a=document.createElement('a');
  a.href=url;a.download='RF_Link_Budget_'+new Date().toISOString().slice(0,10)+'.csv';a.click();URL.revokeObjectURL(url);
}
</script>
</body>
</html>
