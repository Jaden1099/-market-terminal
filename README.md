[file-2.html](https://github.com/user-attachments/files/26358280/file-2.html)
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Market Terminal</title>
  <script src="[cdn.jsdelivr.net](https://cdn.jsdelivr.net/npm/chart.js)"></script>
  <script src="[cdn.jsdelivr.net](https://cdn.jsdelivr.net/npm/chartjs-adapter-date-fns/dist/chartjs-adapter-date-fns.bundle.min.js)"></script>
  <script src="[cdn.jsdelivr.net](https://cdn.jsdelivr.net/npm/chartjs-plugin-zoom@2.0.1/dist/chartjs-plugin-zoom.min.js)"></script>
  <link href="[fonts.googleapis.com](https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap)" rel="stylesheet">
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    :root {
      --bg: #0a0a0a; --surface: #111111; --card: #1a1a1a; --border: #2a2a2a;
      --text: #e2e8f0; --muted: #6b7280; --green: #10b981; --red: #ef4444;
      --blue: #3b82f6; --yellow: #f59e0b; --accent: #8b5cf6;
    }
    body { font-family: 'Inter', sans-serif; background: var(--bg); color: var(--text); min-height: 100vh; }
    ::-webkit-scrollbar { width: 6px; height: 6px; }
    ::-webkit-scrollbar-track { background: var(--bg); }
    ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }

    /* TICKER — FIXED */
    .ticker-bar {
      background: var(--surface);
      border-bottom: 1px solid var(--border);
      padding: 10px 0;
      overflow: hidden;
      position: relative;
      top: 0;
      z-index: 1;
      pointer-events: none;
    }
    .ticker-track {
      display: flex;
      gap: 40px;
      animation: ticker 50s linear infinite;
      white-space: nowrap;
      width: max-content;
      pointer-events: none;
    }
    @keyframes ticker { from { transform: translateX(0); } to { transform: translateX(-50%); } }
    .ticker-item { display: flex; gap: 8px; align-items: center; font-size: 0.85em; }
    .ticker-symbol { font-weight: 600; }
    .ticker-price { color: var(--muted); }
    .up { color: var(--green) !important; }
    .down { color: var(--red) !important; }

    /* LAYOUT */
    .topbar { padding: 20px 24px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid var(--border); position: relative; z-index: 2; }
    .topbar h1 { font-size: 1.3em; font-weight: 700; }
    .topbar-right { display: flex; gap: 10px; align-items: center; }
    .time-display { font-size: 0.85em; color: var(--muted); }
    .mode-badge { background: var(--accent); color: white; padding: 4px 10px; border-radius: 20px; font-size: 0.75em; font-weight: 600; }

    /* NAV */
    .nav { background: var(--surface); border-bottom: 1px solid var(--border); padding: 0 24px; display: flex; gap: 4px; overflow-x: auto; position: relative; z-index: 2; }
    .nav-btn { padding: 12px 16px; font-size: 0.82em; font-weight: 500; color: var(--muted); border: none; background: transparent; cursor: pointer; border-bottom: 2px solid transparent; white-space: nowrap; transition: all 0.2s; font-family: inherit; }
    .nav-btn:hover { color: var(--text); }
    .nav-btn.active { color: var(--text); border-bottom-color: var(--accent); }

    /* SECTIONS */
    .section { display: none; padding: 24px; max-width: 1600px; margin: 0 auto; position: relative; z-index: 2; }
    .section.active { display: block; }
    .section-grid { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 20px; }
    @media(max-width: 1100px) { .section-grid { grid-template-columns: 1fr 1fr; } }
    @media(max-width: 700px) { .section-grid { grid-template-columns: 1fr; } }

    /* CARDS */
    .card { background: var(--card); border: 1px solid var(--border); border-radius: 12px; padding: 20px; }
    .card-title { font-size: 0.75em; font-weight: 600; color: var(--muted); text-transform: uppercase; letter-spacing: 1px; margin-bottom: 16px; }
    .span-2 { grid-column: span 2; }
    .span-3 { grid-column: span 3; }
    @media(max-width: 1100px) { .span-3 { grid-column: span 2; } }
    @media(max-width: 700px) { .span-2, .span-3 { grid-column: span 1; } }

    /* PRICES GRID */
    .prices-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(140px, 1fr)); gap: 12px; }
    .price-card { background: var(--surface); border: 1px solid var(--border); border-radius: 8px; padding: 12px; }
    .price-symbol { font-size: 0.8em; font-weight: 600; color: var(--muted); margin-bottom: 4px; }
    .price-value { font-size: 1.2em; font-weight: 700; margin-bottom: 2px; }
    .price-change { font-size: 0.8em; }

    /* FEAR & GREED */
    .fg-container { display: flex; flex-direction: column; align-items: center; gap: 12px; }
    .fg-score { font-size: 3em; font-weight: 700; }
    .fg-label { font-size: 1em; color: var(--muted); }

    /* BRIEFING */
    .briefing-text { font-size: 0.9em; line-height: 1.7; color: var(--text); }
    .briefing-text.loading { color: var(--muted); font-style: italic; }

    /* WATCHLIST */
    .watchlist-item { display: flex; justify-content: space-between; align-items: center; padding: 10px 0; border-bottom: 1px solid var(--border); }
    .watchlist-item:last-child { border-bottom: none; }
    .watch-symbol { font-weight: 600; font-size: 0.95em; }
    .watch-note { font-size: 0.75em; color: var(--muted); margin-top: 2px; }
    .watch-price { text-align: right; }

    /* NEWS */
    .news-item { padding: 12px 0; border-bottom: 1px solid var(--border); }
    .news-item:last-child { border-bottom: none; }
    .news-headline { font-size: 0.9em; line-height: 1.5; margin-bottom: 4px; }
    .news-meta { font-size: 0.75em; color: var(--muted); }
    .news-headline a { color: var(--text); text-decoration: none; }
    .news-headline a:hover { color: var(--blue); }

    /* PAPER TRADING */
    .trade-stats { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; margin-bottom: 16px; }
    .trade-stat { background: var(--surface); border-radius: 8px; padding: 12px; text-align: center; }
    .trade-stat-value { font-size: 1.4em; font-weight: 700; }
    .trade-stat-label { font-size: 0.75em; color: var(--muted); margin-top: 4px; }
    .trade-form { display: flex; gap: 10px; flex-wrap: wrap; margin-bottom: 16px; }
    .trade-table { width: 100%; border-collapse: collapse; font-size: 0.85em; }
    .trade-table th { text-align: left; padding: 8px; color: var(--muted); border-bottom: 1px solid var(--border); font-weight: 500; }
    .trade-table td { padding: 8px; border-bottom: 1px solid var(--border); }

    /* ANALYZER */
    .analyzer-form { display: flex; gap: 10px; margin-bottom: 16px; }
    .analyzer-result { background: var(--surface); border-radius: 8px; padding: 16px; font-size: 0.9em; line-height: 1.7; min-height: 80px; }

    /* PRACTICE CHART */
    .practice-controls { display: flex; gap: 10px; flex-wrap: wrap; margin-bottom: 16px; align-items: center; }
    .chart-wrap { height: 400px; position: relative; background: var(--surface); border-radius: 8px; overflow: hidden; }
    .practice-stats { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; margin: 16px 0; }
    .practice-stat { background: var(--surface); border-radius: 8px; padding: 12px; text-align: center; }
    .practice-stat-value { font-size: 1.2em; font-weight: 700; }
    .practice-stat-label { font-size: 0.75em; color: var(--muted); margin-top: 4px; }

    /* GLOSSARY */
    .glossary-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 12px; }
    .glossary-item { background: var(--surface); border-radius: 8px; padding: 14px; }
    .glossary-term { font-weight: 600; font-size: 0.9em; margin-bottom: 6px; color: var(--blue); }
    .glossary-def { font-size: 0.82em; color: var(--muted); line-height: 1.5; }

    /* AI CHAT */
    .chat-messages { height: 280px; overflow-y: auto; background: var(--surface); border-radius: 8px; padding: 14px; margin-bottom: 12px; display: flex; flex-direction: column; gap: 10px; }
    .chat-msg { padding: 10px 14px; border-radius: 8px; font-size: 0.88em; line-height: 1.6; max-width: 85%; }
    .chat-msg.user { background: var(--accent); color: white; align-self: flex-end; }
    .chat-msg.ai { background: var(--card); border: 1px solid var(--border); align-self: flex-start; }
    .chat-input-row { display: flex; gap: 10px; }

    /* SETTINGS */
    .settings-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
    @media(max-width: 600px) { .settings-grid { grid-template-columns: 1fr; } }
    .settings-group { background: var(--surface); border-radius: 8px; padding: 16px; }
    .settings-label { font-size: 0.8em; color: var(--muted); margin-bottom: 6px; font-weight: 500; }
    .settings-status { font-size: 0.78em; margin-top: 6px; }

    /* INPUTS / BUTTONS */
    input, select, textarea {
      background: var(--surface); color: var(--text); border: 1px solid var(--border);
      border-radius: 6px; padding: 8px 12px; font-family: inherit; font-size: 0.88em;
      outline: none; transition: border 0.2s;
    }
    input:focus, select:focus, textarea:focus { border-color: var(--accent); }
    input[type="password"] { letter-spacing: 2px; }
    button {
      background: var(--accent); color: white; border: none; border-radius: 6px;
      padding: 8px 16px; font-family: inherit; font-size: 0.88em; font-weight: 600;
      cursor: pointer; transition: opacity 0.2s;
    }
    button:hover { opacity: 0.85; }
    button.secondary { background: var(--surface); color: var(--text); border: 1px solid var(--border); font-weight: 500; }
    button.green { background: var(--green); }
    button.red { background: var(--red); }
    .full-width { width: 100%; }

    /* TABS */
    .tabs { display: flex; gap: 4px; background: var(--surface); border-radius: 8px; padding: 4px; margin-bottom: 16px; }
    .tab { flex: 1; padding: 7px; text-align: center; border-radius: 6px; cursor: pointer; font-size: 0.82em; font-weight: 500; color: var(--muted); transition: all 0.2s; border: none; background: transparent; }
    .tab.active { background: var(--card); color: var(--text); }

    /* LOADING SPINNER */
    .spinner { display: inline-block; width: 16px; height: 16px; border: 2px solid var(--border); border-top-color: var(--accent); border-radius: 50%; animation: spin 0.7s linear infinite; vertical-align: middle; margin-right: 6px; }
    @keyframes spin { to { transform: rotate(360deg); } }

    /* STRATEGY CARDS */
    .strategy-card { background: var(--surface); border-radius: 8px; padding: 16px; border-left: 3px solid var(--accent); }
    .strategy-title { font-weight: 600; margin-bottom: 8px; }
    .strategy-desc { font-size: 0.85em; color: var(--muted); line-height: 1.6; }
    .strategy-rules { margin-top: 10px; }
    .strategy-rules li { font-size: 0.82em; color: var(--text); margin-left: 16px; margin-top: 4px; line-height: 1.5; }
  </style>
</head>
<body>

<!-- TICKER BAR -->
<div class="ticker-bar">
  <div class="ticker-track" id="tickerTrack">
    <span class="ticker-item"><span class="ticker-symbol">SPY</span><span class="ticker-price" id="t-SPY">--</span><span class="ticker-change" id="tc-SPY">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">QQQ</span><span class="ticker-price" id="t-QQQ">--</span><span class="ticker-change" id="tc-QQQ">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">AAPL</span><span class="ticker-price" id="t-AAPL">--</span><span class="ticker-change" id="tc-AAPL">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">NVDA</span><span class="ticker-price" id="t-NVDA">--</span><span class="ticker-change" id="tc-NVDA">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">BTC</span><span class="ticker-price" id="t-BTC">--</span><span class="ticker-change" id="tc-BTC">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">ETH</span><span class="ticker-price" id="t-ETH">--</span><span class="ticker-change" id="tc-ETH">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">GOLD</span><span class="ticker-price" id="t-GOLD">--</span><span class="ticker-change" id="tc-GOLD">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">OIL</span><span class="ticker-price" id="t-OIL">--</span><span class="ticker-change" id="tc-OIL">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">SPY</span><span class="ticker-price" id="t-SPY2">--</span><span class="ticker-change" id="tc-SPY2">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">QQQ</span><span class="ticker-price" id="t-QQQ2">--</span><span class="ticker-change" id="tc-QQQ2">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">AAPL</span><span class="ticker-price" id="t-AAPL2">--</span><span class="ticker-change" id="tc-AAPL2">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">NVDA</span><span class="ticker-price" id="t-NVDA2">--</span><span class="ticker-change" id="tc-NVDA2">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">BTC</span><span class="ticker-price" id="t-BTC2">--</span><span class="ticker-change" id="tc-BTC2">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">ETH</span><span class="ticker-price" id="t-ETH2">--</span><span class="ticker-change" id="tc-ETH2">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">GOLD</span><span class="ticker-price" id="t-GOLD2">--</span><span class="ticker-change" id="tc-GOLD2">--</span></span>
    <span class="ticker-item"><span class="ticker-symbol">OIL</span><span class="ticker-price" id="t-OIL2">--</span><span class="ticker-change" id="tc-OIL2">--</span></span>
  </div>
</div>

<!-- TOP BAR -->
<div class="topbar">
  <h1>📈 My Market Terminal</h1>
  <div class="topbar-right">
    <span class="time-display" id="clockDisplay"></span>
    <span class="mode-badge">PAPER TRADING</span>
  </div>
</div>

<!-- NAV -->
<nav class="nav">
  <button class="nav-btn active" onclick="showSection('dashboard', this)">Dashboard</button>
  <button class="nav-btn" onclick="showSection('prices', this)">Live Prices</button>
  <button class="nav-btn" onclick="showSection('paper', this)">Paper Trading</button>
  <button class="nav-btn" onclick="showSection('practice', this)">SPY Practice</button>
  <button class="nav-btn" onclick="showSection('analyzer', this)">Analyzer</button>
  <button class="nav-btn" onclick="showSection('learn', this)">Learn</button>
  <button class="nav-btn" onclick="showSection('chat', this)">AI Mentor</button>
  <button class="nav-btn" onclick="showSection('settings', this)">⚙️ Settings</button>
</nav>

<!-- DASHBOARD -->
<div class="section active" id="sec-dashboard">
  <div class="section-grid">
    <div class="card span-2">
      <div class="card-title">🤖 AI Market Briefing</div>
      <div id="briefingText" class="briefing-text loading"><span class="spinner"></span> Loading your daily briefing — add your Anthropic key in Settings to activate.</div>
      <button class="secondary" style="margin-top:14px; font-size:0.8em;" onclick="loadBriefing()">↻ Refresh Briefing</button>
    </div>
    <div class="card">
      <div class="card-title">😱 Fear &amp; Greed Index</div>
      <div class="fg-container">
        <canvas id="fgCanvas" width="200" height="110"></canvas>
        <div id="fgScore" class="fg-score" style="color:var(--yellow);">--</div>
        <div id="fgLabel" class="fg-label">Calculating...</div>
      </div>
    </div>
    <div class="card">
      <div class="card-title">👁 What To Watch Today</div>
      <div class="watchlist-item"><div><div class="watch-symbol">BTC</div><div class="watch-note">Crypto leading sentiment</div></div><div class="watch-price"><div id="w-BTC">--</div></div></div>
      <div class="watchlist-item"><div><div class="watch-symbol">SPY</div><div class="watch-note">Broad market direction</div></div><div class="watch-price"><div id="w-SPY">--</div></div></div>
      <div class="watchlist-item"><div><div class="watch-symbol">NVDA</div><div class="watch-note">AI sector bellwether</div></div><div class="watch-price"><div id="w-NVDA">--</div></div></div>
      <div class="watchlist-item"><div><div class="watch-symbol">GOLD</div><div class="watch-note">Risk-off indicator</div></div><div class="watch-price"><div id="w-GOLD">--</div></div></div>
      <div class="watchlist-item"><div><div class="watch-symbol">OIL</div><div class="watch-note">Macro inflation signal</div></div><div class="watch-price"><div id="w-OIL">--</div></div></div>
    </div>
    <div class="card span-2">
      <div class="card-title">📰 Market News</div>
      <div id="newsContainer"><div class="news-item"><div class="news-headline" style="color:var(--muted);">Add your Twelve Data key in Settings to load live news.</div></div></div>
    </div>
    <div class="card">
      <div class="card-title">🌡 Market Mood</div>
      <div style="text-align:center; padding:20px 0;">
        <div style="font-size:3em; margin-bottom:8px;" id="moodEmoji">🤔</div>
        <div style="font-size:1.1em; font-weight:600;" id="moodText">Analyzing...</div>
        <div style="font-size:0.82em; color:var(--muted); margin-top:8px;" id="moodSub"></div>
      </div>
    </div>
  </div>
</div>

<!-- LIVE PRICES -->
<div class="section" id="sec-prices">
  <div class="card">
    <div class="card-title">📊 Live Market Prices</div>
    <div class="prices-grid">
      <div class="price-card"><div class="price-symbol">SPY</div><div class="price-value" id="p-SPY">--</div><div class="price-change" id="pc-SPY">--</div></div>
      <div class="price-card"><div class="price-symbol">QQQ</div><div class="price-value" id="p-QQQ">--</div><div class="price-change" id="pc-QQQ">--</div></div>
      <div class="price-card"><div class="price-symbol">AAPL</div><div class="price-value" id="p-AAPL">--</div><div class="price-change" id="pc-AAPL">--</div></div>
      <div class="price-card"><div class="price-symbol">NVDA</div><div class="price-value" id="p-NVDA">--</div><div class="price-change" id="pc-NVDA">--</div></div>
      <div class="price-card"><div class="price-symbol">TSLA</div><div class="price-value" id="p-TSLA">--</div><div class="price-change" id="pc-TSLA">--</div></div>
      <div class="price-card"><div class="price-symbol">MSFT</div><div class="price-value" id="p-MSFT">--</div><div class="price-change" id="pc-MSFT">--</div></div>
      <div class="price-card"><div class="price-symbol">AMZN</div><div class="price-value" id="p-AMZN">--</div><div class="price-change" id="pc-AMZN">--</div></div>
      <div class="price-card"><div class="price-symbol">META</div><div class="price-value" id="p-META">--</div><div class="price-change" id="pc-META">--</div></div>
      <div class="price-card"><div class="price-symbol">BTC</div><div class="price-value" id="p-BTC">--</div><div class="price-change" id="pc-BTC">--</div></div>
      <div class="price-card"><div class="price-symbol">ETH</div><div class="price-value" id="p-ETH">--</div><div class="price-change" id="pc-ETH">--</div></div>
      <div class="price-card"><div class="price-symbol">GOLD</div><div class="price-value" id="p-GOLD">--</div><div class="price-change" id="pc-GOLD">--</div></div>
      <div class="price-card"><div class="price-symbol">OIL</div><div class="price-value" id="p-OIL">--</div><div class="price-change" id="pc-OIL">--</div></div>
    </div>
    <div style="margin-top:14px; font-size:0.78em; color:var(--muted);">Auto-refreshes every 60s. Last update: <span id="lastUpdate">--</span></div>
  </div>
</div>

<!-- PAPER TRADING -->
<div class="section" id="sec-paper">
  <div class="card">
    <div class="card-title">💼 Paper Trading Tracker</div>
    <div class="trade-stats">
      <div class="trade-stat"><div class="trade-stat-value" id="ptPnl" style="color:var(--green);">$0.00</div><div class="trade-stat-label">Total P&amp;L</div></div>
      <div class="trade-stat"><div class="trade-stat-value" id="ptWins">0/0</div><div class="trade-stat-label">Win / Loss</div></div>
      <div class="trade-stat"><div class="trade-stat-value" id="ptOpenRate">0%</div><div class="trade-stat-label">Win Rate</div></div>
    </div>
    <div class="trade-form">
      <input id="ptSymbol" type="text" placeholder="Symbol e.g. SPY" style="width:120px;" />
      <select id="ptDirection"><option value="LONG">🟢 LONG</option><option value="SHORT">🔴 SHORT</option></select>
      <input id="ptShares" type="number" placeholder="Shares" value="100" style="width:90px;" />
      <input id="ptEntry" type="number" placeholder="Entry $" style="width:100px;" step="0.01" />
      <button class="green" onclick="openTrade()">Open Trade</button>
    </div>
    <table class="trade-table">
      <thead><tr><th>#</th><th>Symbol</th><th>Direction</th><th>Shares</th><th>Entry</th><th>Exit</th><th>P&amp;L</th><th>Status</th><th>Action</th></tr></thead>
      <tbody id="tradeTableBody"><tr><td colspan="9" style="color:var(--muted); text-align:center; padding:20px;">No trades yet.</td></tr></tbody>
    </table>
  </div>
</div>

<!-- SPY PRACTICE -->
<div class="section" id="sec-practice">
  <div class="card">
    <div class="card-title">🧠 SPY Historical Practice Mode</div>
    <p style="font-size:0.85em; color:var(--muted); margin-bottom:16px;">Practice reading real SPY charts. Scroll to zoom, drag to pan. Future price is hidden.</p>
    <div class="practice-controls">
      <select id="practiceInterval">
        <option value="1min">1 Minute</option>
        <option value="5min">5 Minutes</option>
        <option value="15min">15 Minutes</option>
        <option value="1h">1 Hour</option>
        <option value="1day" selected>Daily</option>
        <option value="1week">Weekly</option>
      </select>
      <select id="practiceOutputSize">
        <option value="50">50 bars</option>
        <option value="100" selected>100 bars</option>
        <option value="200">200 bars</option>
      </select>
      <button onclick="loadPracticeChart()">Load Chart</button>
      <button class="secondary" onclick="practiceChart && practiceChart.resetZoom()">Reset Zoom</button>
    </div>
    <div class="chart-wrap"><canvas id="practiceChartCanvas"></canvas></div>
    <div class="practice-stats">
      <div class="practice-stat"><div class="practice-stat-value" id="prCurrent">$--</div><div class="practice-stat-label">Current</div></div>
      <div class="practice-stat"><div class="practice-stat-value" id="prHigh" style="color:var(--green);">$--</div><div class="practice-stat-label">Period High</div></div>
      <div class="practice-stat"><div class="practice-stat-value" id="prLow" style="color:var(--red);">$--</div><div class="practice-stat-label">Period Low</div></div>
      <div class="practice-stat"><div class="practice-stat-value" id="prChange">--</div><div class="practice-stat-label">Period Change</div></div>
    </div>
    <div style="background:var(--surface); padding:16px; border-radius:8px;">
      <div style="font-weight:600; margin-bottom:12px;">📋 Your Call</div>
      <div style="display:flex; gap:10px; flex-wrap:wrap; align-items:center;">
        <select id="prDirection"><option value="LONG">🟢 LONG</option><option value="SHORT">🔴 SHORT</option></select>
        <input id="prShares" type="number" value="100" placeholder="Shares" style="width:90px;" />
        <button class="green" onclick="logPracticeTrade()">Submit Call</button>
      </div>
      <div id="practiceTradeLog" style="margin-top:14px; font-size:0.85em; max-height:140px; overflow-y:auto; display:flex; flex-direction:column; gap:6px;"></div>
    </div>
  </div>
</div>

<!-- ANALYZER -->
<div class="section" id="sec-analyzer">
  <div class="card">
    <div class="card-title">🔍 AI Stock Analyzer</div>
    <p style="font-size:0.85em; color:var(--muted); margin-bottom:16px;">Enter any stock symbol for an AI-powered analysis including sentiment, risk level, and key levels to watch.</p>
    <div class="analyzer-form">
      <input id="analyzerSymbol" type="text" placeholder="e.g. AAPL, TSLA, NVDA" style="flex:1;" />
      <select id="analyzerRisk"><option value="conservative">Conservative</option><option value="moderate" selected>Moderate</option><option value="aggressive">Aggressive</option></select>
      <button onclick="runAnalyzer()">Analyze</button>
    </div>
    <div class="analyzer-result" id="analyzerResult" style="color:var(--muted);">Enter a symbol above and click Analyze.</div>
  </div>
</div>

<!-- LEARN -->
<div class="section" id="sec-learn">
  <div class="tabs">
    <button class="tab active" onclick="switchLearnTab('glossary', this)">📖 Glossary</button>
    <button class="tab" onclick="switchLearnTab('strategies', this)">🎯 Strategies</button>
  </div>
  <div id="learnGlossary"><div class="glossary-grid" id="glossaryGrid"></div></div>
  <div id="learnStrategies" style="display:none;"><div class="section-grid" id="strategiesGrid"></div></div>
</div>

<!-- AI MENTOR CHAT -->
<div class="section" id="sec-chat">
  <div class="card" style="max-width:800px; margin:0 auto;">
    <div class="card-title">🤖 AI Mentor</div>
    <p style="font-size:0.85em; color:var(--muted); margin-bottom:16px;">Ask anything about trading, markets, strategy, or how to read charts.</p>
    <div class="chat-messages" id="chatMessages">
      <div class="chat-msg ai">Hey! I'm your AI trading mentor. Ask me anything — chart patterns, risk management, how options work, what a candlestick means... I'm here to help you learn. 📈</div>
    </div>
    <div class="chat-input-row">
      <input id="chatInput" type="text" placeholder="Ask me anything about trading..." style="flex:1;" onkeydown="if(event.key==='Enter') sendChat()" />
      <button onclick="sendChat()">Send</button>
    </div>
  </div>
</div>

<!-- SETTINGS -->
<div class="section" id="sec-settings">
  <div class="card" style="max-width:800px; margin:0 auto;">
    <div class="card-title">⚙️ Settings &amp; API Keys</div>
    <p style="font-size:0.85em; color:var(--muted); margin-bottom:20px;">Keys are saved to your browser only (localStorage). Never sent anywhere.</p>
    <div class="settings-grid">
      <div class="settings-group">
        <div class="settings-label">🤖 Anthropic API Key</div>
        <input id="anthropicInput" type="password" placeholder="sk-ant-..." class="full-width" />
        <div style="margin-top:8px; display:flex; gap:8px;">
          <button onclick="saveKey('anthropic', 'anthropicInput', 'anthropicStatus')" style="font-size:0.8em;">Save</button>
          <button class="secondary" onclick="clearKey('anthropic', 'anthropicInput', 'anthropicStatus')" style="font-size:0.8em;">Clear</button>
        </div>
        <div id="anthropicStatus" class="settings-status"></div>
      </div>
      <div class="settings-group">
        <div class="settings-label">📊 Twelve Data API Key</div>
        <input id="twelveInput" type="password" placeholder="your_key_here" class="full-width" />
        <div style="margin-top:8px; display:flex; gap:8px;">
          <button onclick="saveKey('twelve_key', 'twelveInput', 'twelveStatus')" style="font-size:0.8em;">Save</button>
          <button class="secondary" onclick="clearKey('twelve_key', 'twelveInput', 'twelveStatus')" style="font-size:0.8em;">Clear</button>
        </div>
        <div id="twelveStatus" class="settings-status"></div>
      </div>
      <div class="settings-group">
        <div class="settings-label">🪙 CoinGecko API Key <span style="color:var(--green); font-size:0.85em;">(free tier works without key)</span></div>
        <input id="geckoInput" type="password" placeholder="Optional" class="full-width" />
        <div style="margin-top:8px; display:flex; gap:8px;">
          <button onclick="saveKey('coingecko_key', 'geckoInput', 'geckoStatus')" style="font-size:0.8em;">Save</button>
          <button class="secondary" onclick="clearKey('coingecko_key', 'geckoInput', 'geckoStatus')" style="font-size:0.8em;">Clear</button>
        </div>
        <div id="geckoStatus" class="settings-status"></div>
      </div>
      <div class="settings-group">
        <div class="settings-label">🔗 Get Your Keys</div>
        <div style="display:flex; flex-direction:column; gap:8px; font-size:0.83em;">
          <a href="[console.anthropic.com](https://console.anthropic.com)" target="_blank" style="color:var(--blue);">→ Anthropic Console</a>
          <a href="[twelvedata.com](https://twelvedata.com)" target="_blank" style="color:var(--blue);">→ Twelve Data</a>
          <a href="[coingecko.com](https://www.coingecko.com/en/api)" target="_blank" style="color:var(--blue);">→ CoinGecko API</a>
        </div>
      </div>
    </div>
  </div>
</div>

<script>
// ── KEYS ──────────────────────────────────────────────────
const getAnthropicKey = () => localStorage.getItem('anthropic') || '';
const getTwelveKey    = () => localStorage.getItem('twelve_key') || '';
const getCoinGeckoKey = () => localStorage.getItem('coingecko_key') || '';

function saveKey(storageKey, inputId, statusId) {
  const val = document.getElementById(inputId).value.trim();
  if (!val) return;
  localStorage.setItem(storageKey, val);
  document.getElementById(statusId).innerHTML = `<span style="color:var(--green);">✓ Saved</span>`;
  document.getElementById(inputId).value = '';
  init();
}

function clearKey(storageKey, inputId, statusId) {
  localStorage.removeItem(storageKey);
  document.getElementById(inputId).value = '';
  document.getElementById(statusId).innerHTML = `<span style="color:var(--muted);">Cleared</span>`;
}

function refreshSettingsStatus() {
  if (getAnthropicKey()) document.getElementById('anthropicStatus').innerHTML = `<span style="color:var(--green);">✓ Key saved</span>`;
  if (getTwelveKey())    document.getElementById('twelveStatus').innerHTML    = `<span style="color:var(--green);">✓ Key saved</span>`;
  if (getCoinGeckoKey()) document.getElementById('geckoStatus').innerHTML     = `<span style="color:var(--green);">✓ Key saved</span>`;
}

// ── CLOCK ─────────────────────────────────────────────────
function updateClock() {
  const now = new Date();
  document.getElementById('clockDisplay').textContent = now.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: true, timeZoneName: 'short' });
}
setInterval(updateClock, 1000);
updateClock();

// ── NAV ───────────────────────────────────────────────────
function showSection(id, btn) {
  document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
  document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
  document.getElementById('sec-' + id).classList.add('active');
  if (btn) btn.classList.add('active');
  if (id === 'settings') refreshSettingsStatus();
  if (id === 'learn') renderLearn();
}

// ── LIVE PRICES ───────────────────────────────────────────
let cachedPrices = {};

async function fetchStockPrices() {
  const key = getTwelveKey();
  if (!key) return;
  const symbols = ['SPY','QQQ','AAPL','NVDA','TSLA','MSFT','AMZN','META','GOLD','CL=F'];
  try {
    const res  = await fetch(`[api.twelvedata.com](https://api.twelvedata.com/quote?symbol=${symbols.join()',')}&apikey=${key}`);
    const data = await res.json();
    symbols.forEach(sym => {
      const d = data[sym] || data;
      if (!d || d.status === 'error') return;
      const price  = parseFloat(d.close || d.price);
      const change = parseFloat(d.percent_change);
      if (isNaN(price)) return;
      const label = sym === 'CL=F' ? 'OIL' : sym;
      cachedPrices[label] = { price, change };
    });
    updatePriceDisplays();
    document.getElementById('lastUpdate').textContent = new Date().toLocaleTimeString();
  } catch(e) { console.error('Stock price error', e); }
}

async function fetchCryptoPrices() {
  try {
    const res  = await fetch('[api.coingecko.com](https://api.coingecko.com/api/v3/simple/price?ids=bitcoin,ethereum&vs_currencies=usd&include_24hr_change=true)');
    const data = await res.json();
    if (data.bitcoin)  cachedPrices['BTC'] = { price: data.bitcoin.usd,  change: data.bitcoin.usd_24h_change  };
    if (data.ethereum) cachedPrices['ETH'] = { price: data.ethereum.usd, change: data.ethereum.usd_24h_change };
    updatePriceDisplays();
  } catch(e) { console.error('Crypto price error', e); }
}

function updatePriceDisplays() {
  const fmt  = p => p > 1000 ? `$${p.toLocaleString('en-US', {minimumFractionDigits:2, maximumFractionDigits:2})}` : `$${p.toFixed(2)}`;
  const fmtC = c => c >= 0 ? `▲ ${c.toFixed(2)}%` : `▼ ${Math.abs(c).toFixed(2)}%`;

  Object.entries(cachedPrices).forEach(([sym, {price, change}]) => {
    const cls = change >= 0 ? 'up' : 'down';
    ['', '2'].forEach(sfx => {
      const pe = document.getElementById(`t-${sym}${sfx}`);
      const ce = document.getElementById(`tc-${sym}${sfx}`);
      if (pe) pe.textContent = fmt(price);
      if (ce) { ce.textContent = fmtC(change); ce.className = `ticker-change ${cls}`; }
    });
    const pv = document.getElementById(`p-${sym}`);
    const pc = document.getElementById(`pc-${sym}`);
    if (pv) { pv.textContent = fmt(price); pv.className = `price-value ${cls}`; }
    if (pc) { pc.textContent = fmtC(change); pc.className = `price-change ${cls}`; }
    const wv = document.getElementById(`w-${sym}`);
    if (wv) { wv.textContent = fmt(price); wv.className = cls; }
  });
  updateMood();
  updateFearGreed();
}

// ── MARKET MOOD ───────────────────────────────────────────
function updateMood() {
  const prices = Object.values(cachedPrices);
  if (!prices.length) return;
  const avg = prices.reduce((a, b) => a + b.change, 0) / prices.length;
  let emoji, text, sub;
  if      (avg >  1.5) { emoji = '🚀'; text = 'Strong Bull';    sub = 'Markets ripping — momentum high'; }
  else if (avg >  0.5) { emoji = '📈'; text = 'Bullish';        sub = 'Buyers in control today'; }
  else if (avg >  0.1) { emoji = '😐'; text = 'Slightly Green'; sub = 'Cautious optimism'; }
  else if (avg > -0.1) { emoji = '🤔'; text = 'Neutral';        sub = 'No clear direction'; }
  else if (avg > -0.5) { emoji = '😟'; text = 'Slightly Red';   sub = 'Mild selling pressure'; }
  else if (avg > -1.5) { emoji = '📉'; text = 'Bearish';        sub = 'Bears in control'; }
  else                  { emoji = '🔥'; text = 'Heavy Selloff';  sub = 'Risk-off — stay cautious'; }
  document.getElementById('moodEmoji').textContent = emoji;
  document.getElementById('moodText').textContent  = text;
  document.getElementById('moodSub').textContent   = sub;
}

// ── FEAR & GREED ──────────────────────────────────────────
function updateFearGreed() {
  const prices = Object.values(cachedPrices);
  if (!prices.length) return;
  const avg   = prices.reduce((a, b) => a + b.change, 0) / prices.length;
  const score = Math.min(100, Math.max(0, Math.round(50 + avg * 10)));
  let label, color;
  if      (score >= 75) { label = 'Extreme Greed'; color = '#10b981'; }
  else if (score >= 55) { label = 'Greed';          color = '#84cc16'; }
  else if (score >= 45) { label = 'Neutral';         color = '#f59e0b'; }
  else if (score >= 25) { label = 'Fear';            color = '#f97316'; }
  else                   { label = 'Extreme Fear';   color = '#ef4444'; }
  document.getElementById('fgScore').textContent = score;
  document.getElementById('fgScore').style.color = color;
  document.getElementById('fgLabel').textContent = label;
  drawFGGauge(score, color);
}

function drawFGGauge(score, color) {
  const canvas = document.getElementById('fgCanvas');
  if (!canvas) return;
  const ctx = canvas.getContext('2d');
  const cx = 100, cy = 100, r = 80;
  ctx.clearRect(0, 0, 200, 110);
  ctx.beginPath(); ctx.arc(cx, cy, r, Math.PI, 0, false);
  ctx.lineWidth = 14; ctx.strokeStyle = '#2a2a2a'; ctx.stroke();
  const angle = Math.PI + (score / 100) * Math.PI;
  ctx.beginPath(); ctx.arc(cx, cy, r, Math.PI, angle, false);
  ctx.strokeStyle = color; ctx.stroke();
  const needleAngle = Math.PI + (score / 100) * Math.PI;
  ctx.beginPath();
  ctx.moveTo(cx, cy);
  ctx.lineTo(cx + (r - 20) * Math.cos(needleAngle), cy + (r - 20) * Math.sin(needleAngle));
  ctx.lineWidth = 2; ctx.strokeStyle = '#e2e8f0'; ctx.stroke();
}

// ── AI BRIEFING ───────────────────────────────────────────
async function loadBriefing() {
  const key = getAnthropicKey();
  if (!key) {
    document.getElementById('briefingText').innerHTML = '⚠️ Add your Anthropic API key in <strong>Settings</strong> to enable AI briefings.';
    document.getElementById('briefingText').className = 'briefing-text loading';
    return;
  }
  document.getElementById('briefingText').innerHTML = '<span class="spinner"></span> Generating your market briefing...';
  document.getElementById('briefingText').className = 'briefing-text loading';
  const priceContext = Object.entries(cachedPrices).map(([s, {price, change}]) => `${s}: $${price.toFixed(2)} (${change >= 0 ? '+' : ''}${change.toFixed(2)}%)`).join(', ');
  try {
    const res = await fetch('[api.anthropic.com](https://api.anthropic.com/v1/messages)', {
      method: 'POST',
      headers: { 'x-api-key': key, 'anthropic-version': '2023-06-01', 'content-type': 'application/json', 'anthropic-dangerous-direct-browser-calls': 'true' },
      body: JSON.stringify({
        model: 'claude-opus-4-5',
        max_tokens: 400,
        messages: [{ role: 'user', content: `You are a concise market analyst. Current prices: ${priceContext || 'not yet loaded'}. Give a short 3-4 sentence market briefing for today. Focus on what's moving, key levels, and one actionable insight for a beginner paper trader. Be direct and educational.` }]
      })
    });
    const data = await res.json();
    document.getElementById('briefingText').textContent = data?.content?.[0]?.text || 'Could not generate briefing.';
    document.getElementById('briefingText').className = 'briefing-text';
  } catch(e) {
    document.getElementById('briefingText').textContent = 'Error connecting to AI. Check your key and try again.';
  }
}

// ── AI ANALYZER ───────────────────────────────────────────
async function runAnalyzer() {
  const key    = getAnthropicKey();
  const symbol = document.getElementById('analyzerSymbol').value.trim().toUpperCase();
  const risk   = document.getElementById('analyzerRisk').value;
  if (!symbol) return;
  if (!key) { document.getElementById('analyzerResult').innerHTML = '⚠️ Add your Anthropic API key in Settings.'; return; }
  document.getElementById('analyzerResult').innerHTML = `<span class="spinner"></span> Analyzing ${symbol}...`;
  const priceData = cachedPrices[symbol] ? `Current: $${cachedPrices[symbol].price.toFixed(2)} (${cachedPrices[symbol].change.toFixed(2)}%)` : 'price not loaded';
  try {
    const res = await fetch('[api.anthropic.com](https://api.anthropic.com/v1/messages)', {
      method: 'POST',
      headers: { 'x-api-key': key, 'anthropic-version': '2023-06-01', 'content-type': 'application/json', 'anthropic-dangerous-direct-browser-calls': 'true' },
      body: JSON.stringify({
        model: 'claude-opus-4-5',
        max_tokens: 450,
        messages: [{ role: 'user', content: `Analyze ${symbol} for a ${risk} paper trader. ${priceData}. Cover: 1) Current sentiment 2) Key support/resistance levels 3) Risk level 4) One trade idea with entry, stop-loss, and target. Keep it beginner-friendly and under 200 words.` }]
      })
    });
    const data = await res.json();
    document.getElementById('analyzerResult').textContent = data?.content?.[0]?.text || 'No response.';
  } catch(e) {
    document.getElementById('analyzerResult').textContent = 'Error — check your Anthropic key.';
  }
}

// ── AI CHAT ───────────────────────────────────────────────
let chatHistory = [];

async function sendChat() {
  const key   = getAnthropicKey();
  const input = document.getElementById('chatInput');
  const msg   = input.value.trim();
  if (!msg) return;
  if (!key) { appendChatMsg('ai', '⚠️ Please add your Anthropic API key in Settings to use the AI mentor.'); return; }
  appendChatMsg('user', msg);
  input.value = '';
  chatHistory.push({ role: 'user', content: msg });
  const thinkingEl = appendChatMsg('ai', '...');
  try {
    const res = await fetch('[api.anthropic.com](https://api.anthropic.com/v1/messages)', {
      method: 'POST',
      headers: { 'x-api-key': key, 'anthropic-version': '2023-06-01', 'content-type': 'application/json', 'anthropic-dangerous-direct-browser-calls': 'true' },
      body: JSON.stringify({
        model: 'claude-opus-4-5',
        max_tokens: 500,
        system: 'You are a friendly, knowledgeable trading mentor for beginners. Answer questions about stocks, crypto, chart patterns, trading strategies, risk management, and market concepts. Keep answers clear, educational, and under 150 words. Use simple language.',
        messages: chatHistory
      })
    });
    const data = await res.json();
    const reply = data?.content?.[0]?.text || 'Sorry, I could not respond.';
    thinkingEl.textContent = reply;
    chatHistory.push({ role: 'assistant', content: reply });
    if (chatHistory.length > 20) chatHistory = chatHistory.slice(-20);
  } catch(e) { thinkingEl.textContent = 'Connection error. Check your Anthropic key.'; }
}

function appendChatMsg(role, text) {
  const el = document.createElement('div');
  el.className = `chat-msg ${role}`;
  el.textContent = text;
  const container = document.getElementById('chatMessages');
  container.appendChild(el);
  container.scrollTop = container.scrollHeight;
  return el;
}

// ── PAPER TRADING ─────────────────────────────────────────
let paperTrades = JSON.parse(localStorage.getItem('paperTrades') || '[]');

function openTrade() {
  const symbol = document.getElementById('ptSymbol').value.trim().toUpperCase();
  const dir    = document.getElementById('ptDirection').value;
  const shares = parseFloat(document.getElementById('ptShares').value);
  const entry  = parseFloat(document.getElementById('ptEntry').value);
  if (!symbol || isNaN(shares) || isNaN(entry)) return alert('Fill in all fields.');
  paperTrades.push({ id: Date.now(), symbol, dir, shares, entry, exit: null, status: 'OPEN' });
  savePaperTrades();
  renderPaperTrades();
}

function closeTrade(id) {
  const exitPrice = parseFloat(prompt('Exit price?'));
  if (isNaN(exitPrice)) return;
  const trade = paperTrades.find(t => t.id === id);
  if (!trade) return;
  trade.exit   = exitPrice;
  trade.status = 'CLOSED';
  trade.pnl    = trade.dir === 'LONG' ? (exitPrice - trade.entry) * trade.shares : (trade.entry - exitPrice) * trade.shares;
  savePaperTrades();
  renderPaperTrades();
}

function savePaperTrades() { localStorage.setItem('paperTrades', JSON.stringify(paperTrades)); }

function renderPaperTrades() {
  const tbody = document.getElementById('tradeTableBody');
  if (!paperTrades.length) {
    tbody.innerHTML = '<tr><td colspan="9" style="color:var(--muted); text-align:center; padding:20px;">No trades yet.</td></tr>';
    return;
  }
  tbody.innerHTML = paperTrades.map((t, i) => {
    const pnl = t.pnl !== undefined ? `<span class="${t.pnl >= 0 ? 'up' : 'down'}">$${t.pnl.toFixed(2)}</span>` : '--';
    return `<tr>
      <td>${i+1}</td><td>${t.symbol}</td>
      <td class="${t.dir === 'LONG' ? 'up' : 'down'}">${t.dir}</td>
      <td>${t.shares}</td><td>$${t.entry.toFixed(2)}</td>
      <td>${t.exit ? '$'+t.exit.toFixed(2) : '--'}</td>
      <td>${pnl}</td>
      <td><span style="color:${t.status==='OPEN'?'var(--yellow)':'var(--muted)'}">${t.status}</span></td>
      <td>${t.status==='OPEN' ? `<button class="red" style="font-size:0.75em;padding:4px 10px;" onclick="closeTrade(${t.id})">Close</button>` : '✓'}</td>
    </tr>`;
  }).join('');
  const closed   = paperTrades.filter(t => t.pnl !== undefined);
  const totalPnl = closed.reduce((a, t) => a + t.pnl, 0);
  const wins     = closed.filter(t => t.pnl > 0).length;
  const rate     = closed.length ? Math.round((wins / closed.length) * 100) : 0;
  document.getElementById('ptPnl').textContent      = `$${totalPnl.toFixed(2)}`;
  document.getElementById('ptPnl').className        = `trade-stat-value ${totalPnl >= 0 ? 'up' : 'down'}`;
  document.getElementById('ptWins').textContent     = `${wins}/${closed.length}`;
  document.getElementById('ptOpenRate').textContent = `${rate}%`;
}

// ── PRACTICE CHART ────────────────────────────────────────
let practiceChart;
let practiceData  = [];
let practiceTrades = [];

function initPracticeChart() {
  const ctx = document.getElementById('practiceChartCanvas').getContext('2d');
  practiceChart = new Chart(ctx, {
    type: 'line',
    data: { datasets: [{
      label: 'SPY', data: [],
      borderColor: '#10b981', backgroundColor: 'rgba(16,185,129,0.08)',
      borderWidth: 2, fill: true, tension: 0.1, pointRadius: 0, pointHoverRadius: 4
    }]},
    options: {
      responsive: true,
      maintainAspectRatio: false,
      interaction: { intersect: false, mode: 'index' },
      scales: {
        x: { type: 'time', time: { tooltipFormat: 'MMM d, yyyy' }, grid: { color: '#1f1f1f' }, ticks: { color: '#6b7280', maxTicksLimit: 8 } },
        y: { grid: { color: '#1f1f1f' }, ticks: { color: '#6b7280', callback: v => '$' + v.toFixed(0) }, beginAtZero: false }
      },
      plugins: {
        legend: { display: false },
        tooltip: { backgroundColor: '#1a1a1a', borderColor: '#2a2a2a', borderWidth: 1, callbacks: { label: ctx => ` $${ctx.parsed.y.toFixed(2)}` } },
        zoom: { zoom: { wheel: { enabled: true }, pinch: { enabled: true }, mode: 'xy' }, pan: { enabled: true, mode: 'xy' } }
      }
    },
    plugins: [ChartZoom]
  });
}

async function loadPracticeChart() {
  const key      = getTwelveKey();
  const interval = document.getElementById('practiceInterval').value;
  const size     = document.getElementById('practiceOutputSize').value;
  if (!key) { alert('Add your Twelve Data key in Settings to load live SPY data.'); return; }
  try {
    const res  = await fetch(`[api.twelvedata.com](https://api.twelvedata.com/time_series?symbol=SPY&interval=${interval}&outputsize=${size}&apikey=${key})`);
    const data = await res.json();
    if (!data?.values) throw new Error('No data');
    practiceData = data.values.reverse().map(d => ({ x: new Date(d.datetime), y: parseFloat(d.close) }));
    practiceChart.data.datasets[0].data = practiceData;
    practiceChart.resetZoom();
    practiceChart.update('none');
    updatePracticeStats();
  } catch(e) { alert('Could not load SPY data. Check your Twelve Data key.'); }
}

function updatePracticeStats() {
  if (!practiceData.length) return;
  const prices  = practiceData.map(d => d.y);
  const current = prices[prices.length - 1];
  const high    = Math.max(...prices);
  const low     = Math.min(...prices);
  const change  = ((current - prices[0]) / prices[0] * 100).toFixed(2);
  document.getElementById('prCurrent').textContent = `$${current.toFixed(2)}`;
  document.getElementById('prHigh').textContent    = `$${high.toFixed(2)}`;
  document.getElementById('prLow').textContent     = `$${low.toFixed(2)}`;
  document.getElementById('prChange').textContent  = `${change >= 0 ? '+' : ''}${change}%`;
  document.getElementById('prChange').className    = `practice-stat-value ${change >= 0 ? 'up' : 'down'}`;
}

function logPracticeTrade() {
  if (!practiceData.length) return alert('Load a chart first.');
  const dir    = document.getElementById('prDirection').value;
  const shares = document.getElementById('prShares').value;
  const entry  = practiceData[practiceData.length - 1].y;
  const time   = new Date().toLocaleTimeString();
  practiceTrades.push({ dir, shares, entry, time });
  const log = document.getElementById('practiceTradeLog');
  const el  = document.createElement('div');
  el.style.cssText = 'background:var(--surface); padding:8px 12px; border-radius:6px; font-size:0.83em;';
  el.innerHTML = `<span class="${dir === 'LONG' ? 'up' : 'down'}">${dir}</span> ${shares} shares @ <strong>$${entry.toFixed(2)}</strong> — ${time}`;
  log.prepend(el);
}

// ── LEARN ─────────────────────────────────────────────────
const glossaryTerms = [
  { term: 'Bull Market',     def: 'A market trending upward, with prices rising 20%+ from recent lows. Investors are optimistic.' },
  { term: 'Bear Market',     def: 'A market falling 20%+ from recent highs. Pessimism and selling dominate.' },
  { term: 'Support',         def: 'A price level where buying interest is strong enough to prevent the price from falling further.' },
  { term: 'Resistance',      def: 'A price level where selling pressure prevents the price from rising further.' },
  { term: 'Volume',          def: 'The number of shares traded in a period. High volume confirms price moves.' },
  { term: 'Breakout',        def: 'When price moves above resistance or below support with strong volume.' },
  { term: 'Stop Loss',       def: 'An automatic sell order placed below your entry to limit potential losses.' },
  { term: 'Moving Average',  def: 'The average price over a set period. Used to smooth out noise and spot trends.' },
  { term: 'RSI',             def: 'Relative Strength Index. Measures momentum 0-100. Over 70 = overbought, under 30 = oversold.' },
  { term: 'Candlestick',     def: 'A chart element showing open, high, low, and close prices for a time period.' },
  { term: 'P&L',             def: 'Profit and Loss. The gain or loss on a trade or portfolio.' },
  { term: 'Long',            def: 'Buying a security expecting its price to rise.' },
  { term: 'Short',           def: "Selling a security you don't own, expecting the price to fall." },
  { term: 'Volatility',      def: 'How much a price moves up and down. Higher volatility = bigger swings, more risk.' },
  { term: 'Liquidity',       def: 'How easily an asset can be bought or sold without moving its price.' },
  { term: 'Market Cap',      def: 'Total market value of a company. Share price × total shares outstanding.' },
  { term: 'ETF',             def: 'Exchange-Traded Fund. A basket of securities that trades like a single stock.' },
  { term: 'Options',         def: 'Contracts giving the right (not obligation) to buy or sell at a set price before a date.' },
  { term: 'MACD',            def: 'Momentum indicator showing the relationship between two moving averages.' },
  { term: 'Bid/Ask Spread',  def: 'The difference between the highest buyer price and lowest seller price.' },
];

const strategies = [
  { title: '📈 Trend Following',    color: '#10b981', desc: 'Trade in the direction of the dominant trend using moving averages.',           rules: ['Use 20-day and 50-day moving averages', 'Enter when shorter MA crosses above longer MA', 'Set stop-loss below recent swing low', 'Exit when MA cross reverses'] },
  { title: '🔄 Mean Reversion',     color: '#3b82f6', desc: 'Bet that extreme price moves will snap back to the average.',                    rules: ['Look for RSI below 30 to buy', 'Look for RSI above 70 to sell', 'Use Bollinger Bands as price extremes', 'Exit near the middle of the range'] },
  { title: '💥 Breakout Trading',   color: '#f59e0b', desc: 'Enter when price breaks through a key level with high volume.',                  rules: ['Identify strong support/resistance', 'Wait for close above/below the level', 'Confirm with high volume', 'Stop-loss just inside the broken level'] },
  { title: '🕯 Candlestick Patterns', color: '#8b5cf6', desc: 'Read candles to predict short-term reversals or continuations.',               rules: ['Doji = indecision, watch for reversal', 'Hammer = potential bullish reversal', 'Engulfing = strong reversal signal', 'Always confirm with volume and trend'] },
  { title: '📊 Volume Analysis',    color: '#ef4444', desc: 'Price moves are only meaningful when backed by volume.',                         rules: ['Rising price + rising volume = strong trend', 'Rising price + falling volume = weak move', 'High volume selloff = potential bottom', 'Check volume on all breakouts'] },
  { title: '🛡 Risk Management',    color: '#10b981', desc: 'The most important strategy. Protect your capital first.',                        rules: ['Never risk more than 1-2% per trade', 'Always use a stop-loss', 'Aim for 1:2 risk/reward ratio', 'Keep a trading journal'] },
];

function renderLearn() {
  const gg = document.getElementById('glossaryGrid');
  if (!gg.children.length) {
    gg.innerHTML = glossaryTerms.map(g => `<div class="glossary-item"><div class="glossary-term">${g.term}</div><div class="glossary-def">${g.def}</div></div>`).join('');
  }
  const sg = document.getElementById('strategiesGrid');
  if (!sg.children.length) {
    sg.innerHTML = strategies.map(s => `
      <div class="strategy-card" style="border-left-color:${s.color}">
        <div class="strategy-title">${s.title}</div>
        <div class="strategy-desc">${s.desc}</div>
        <ul class="strategy-rules">${s.rules.map(r => `<li>${r}</li>`).join('')}</ul>
      </div>`).join('');
  }
}

function switchLearnTab(tab, btn) {
  document.querySelectorAll('.tabs .tab').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');
  document.getElementById('learnGlossary').style.display   = tab === 'glossary'   ? 'block' : 'none';
  document.getElementById('learnStrategies').style.display = tab === 'strategies' ? 'block' : 'none';
}

// ── NEWS ──────────────────────────────────────────────────
async function loadNews() {
  const key = getTwelveKey();
  if (!key) return;
  try {
    const res  = await fetch(`[api.twelvedata.com](https://api.twelvedata.com/news?apikey=${key})`);
    const data = await res.json();
    if (!data?.data?.length) return;
    document.getElementById('newsContainer').innerHTML = data.data.slice(0, 8).map(n => `
      <div class="news-item">
        <div class="news-headline"><a href="${n.url}" target="_blank">${n.title}</a></div>
        <div class="news-meta">${new Date(n.published_at * 1000).toLocaleDateString()}</div>
      </div>`).join('');
  } catch(e) { console.error('News error', e); }
}

// ── INIT ──────────────────────────────────────────────────
function init() {
  fetchStockPrices();
  fetchCryptoPrices();
  loadBriefing();
  loadNews();
}

document.addEventListener('DOMContentLoaded', () => {
  initPracticeChart();
  renderPaperTrades();
  init();
  setInterval(() => { fetchStockPrices(); fetchCryptoPrices(); }, 60000);
});
</script>
</body>
</html>
