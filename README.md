
<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Galactic Defender - UPDATE 10.5</title>
    <style>
        * { box-sizing: border-box; user-select: none; }
        body { 
            margin: 0; padding: 0; background: #000; color: #fff; 
            font-family: 'Segoe UI', system-ui, sans-serif; 
            overflow: hidden; touch-action: none; cursor: none;
        }
        canvas { display: block; position: absolute; top: 0; left: 0; z-index: 1; }
        .loader-overlay { 
            position: fixed; top: 0; left: 0; width: 100%; height: 100%; 
            background: radial-gradient(circle, #001a33 0%, #000 100%); 
            z-index: 2000; display: flex; flex-direction: column; justify-content: center; align-items: center; 
        }
        .loader-content { width: 260px; text-align: center; }
        .p-bar-outer { width: 100%; height: 8px; background: rgba(255,255,255,0.1); border-radius: 20px; border: 1px solid #00d2ff; overflow: hidden; margin-top: 12px; }
        .p-bar-inner { width: 0%; height: 100%; background: linear-gradient(90deg, #00d2ff, #00ffaa); transition: 0.1s; }
        .overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: none; flex-direction: column; justify-content: flex-start; align-items: center; z-index: 100; text-align: center; backdrop-filter: blur(6px); overflow-y: auto; padding: 20px 10px; }
        #main-hub { display: flex; background: radial-gradient(circle at center, #001533 0%, #010005 100%); z-index: 150; justify-content: flex-start; }
        #game-select-screen { display: none; background: radial-gradient(circle at center, #001533 0%, #010005 100%); z-index: 140; justify-content: flex-start; }
        #start-screen { display: none; background: radial-gradient(circle at center, #001533 0%, #010005 100%); justify-content: flex-start; position: relative; }
        #shop-screen { background: rgba(0,10,30,0.96); border: 1px solid #00d2ff; overflow-y: auto; justify-content: flex-start; }
        #rng-shop-screen { background: rgba(0,10,30,0.96); border: 1px solid #ff00ff; overflow-y: auto; justify-content: flex-start; }
        #achievements-screen { background: rgba(0,10,30,0.96); border: 1px solid gold; overflow-y: auto; justify-content: flex-start; }
        #events-screen { background: rgba(0,10,30,0.96); border: 1px solid #ff00ff; overflow-y: auto; justify-content: flex-start; }
        #skins-screen { background: rgba(0,10,30,0.96); border: 1px solid #ff66ff; overflow-y: auto; justify-content: flex-start; }
        #settings-screen { background: rgba(0,10,30,0.96); border: 1px solid #00ffaa; overflow-y: auto; justify-content: flex-start; }
        #update-log-screen { background: rgba(0,10,30,0.96); border: 1px solid #00ffaa; overflow-y: auto; justify-content: flex-start; }
        #game-over { background: rgba(40,0,0,0.96); justify-content: center; }
        #pause-screen { background: rgba(0,10,30,0.94); justify-content: center; }
        #ascend-screen { background: rgba(0,0,0,0.95); border: 2px solid gold; justify-content: center; }
        .side-btn { position: absolute; right: 10px; width: 40px; height: 40px; border-radius: 50%; background: rgba(0,30,60,0.8); border: 1px solid #00d2ff; color: #fff; font-size: 20px; display: flex; align-items: center; justify-content: center; cursor: pointer; z-index: 60; transition: 0.2s; }
        .side-btn:hover { background: rgba(0,80,120,0.9); transform: scale(1.05); }
        #ach-side-btn { top: 80px; }
        #events-side-btn { top: 130px; }
        #skins-side-btn { top: 180px; }
        #settings-side-btn { top: 230px; }
        .settings-option { background: rgba(0,0,0,0.5); border-radius: 10px; padding: 10px; margin: 8px; display: flex; justify-content: space-between; align-items: center; width: 300px; }
        .settings-toggle { width: 50px; height: 25px; background: #333; border-radius: 25px; cursor: pointer; transition: 0.2s; position: relative; }
        .settings-toggle.on { background: #00ffaa; }
        .settings-toggle.on.music-toggle-on { background: #ff66ff; }
        .settings-toggle:after { content: ""; position: absolute; width: 21px; height: 21px; background: #fff; border-radius: 50%; top: 2px; left: 3px; transition: 0.2s; }
        .settings-toggle.on:after { left: 26px; }
        .custom-alert { position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); background: linear-gradient(135deg, #001a33, #000); border: 2px solid #00d2ff; border-radius: 15px; padding: 20px; min-width: 280px; max-width: 400px; z-index: 3000; text-align: center; backdrop-filter: blur(10px); display: none; flex-direction: column; gap: 15px; }
        .custom-alert p { margin: 0; font-size: 16px; }
        .custom-alert button { background: #00d2ff; border: none; padding: 8px 20px; border-radius: 25px; color: #000; font-weight: bold; cursor: pointer; margin-top: 10px; }
        .notification-area { position: fixed; top: 80px; right: 20px; width: 280px; z-index: 2500; display: flex; flex-direction: column; gap: 8px; pointer-events: none; }
        .notification { background: linear-gradient(135deg, rgba(0,30,60,0.95), rgba(0,10,30,0.95)); border-right: 4px solid; border-radius: 10px; padding: 10px 15px; animation: slideInRight 0.3s ease-out, fadeOut 0.5s ease-out 4.5s forwards; transform-origin: right; pointer-events: none; }
        @keyframes slideInRight { from { transform: translateX(100%); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
        @keyframes fadeOut { to { opacity: 0; transform: translateX(100%); } }
        .notification-warning { border-right-color: #ff0044; }
        .notification-success { border-right-color: #00ffaa; }
        .notification-event { border-right-color: #ff00ff; }
        .notification-info { border-right-color: #00d2ff; }
        .tooltip { position: relative; display: inline-block; cursor: help; }
        .tooltip .tooltip-text { visibility: hidden; width: 200px; background-color: #001a33; color: #fff; text-align: center; border-radius: 6px; padding: 5px; position: absolute; z-index: 1; bottom: 125%; left: 50%; margin-left: -100px; opacity: 0; transition: opacity 0.3s; border: 1px solid #00d2ff; font-size: 11px; pointer-events: none; }
        .tooltip:hover .tooltip-text { visibility: visible; opacity: 1; }
        .gem-notification { position: fixed; top: 20%; left: 50%; transform: translate(-50%, -50%); background: linear-gradient(135deg, #8e44ad, #ff00ff); color: gold; padding: 10px 20px; border-radius: 30px; font-size: 20px; font-weight: bold; z-index: 2000; animation: gemPop 1s ease-out forwards; pointer-events: none; white-space: nowrap; }
        @keyframes gemPop { 0% { opacity: 0; transform: translate(-50%, -50%) scale(0.5); } 20% { opacity: 1; transform: translate(-50%, -50%) scale(1.2); } 80% { opacity: 1; transform: translate(-50%, -50%) scale(1); } 100% { opacity: 0; transform: translate(-50%, -80%) scale(0.8); } }
        #game-timer { position: absolute; bottom: 12px; left: 12px; font-size: 10px; color: #aaa; background: rgba(0,0,0,0.5); padding: 2px 8px; border-radius: 10px; z-index: 50; display: none; pointer-events: none; }
        .daily-reward-btn { background: linear-gradient(45deg, #ffaa00, #ff6600); border: none; color: #fff; padding: 8px 15px; border-radius: 25px; font-weight: bold; cursor: pointer; margin: 5px; font-size: 12px; }
        .daily-mission-card { background: rgba(0,0,0,0.5); border-radius: 10px; padding: 8px 12px; margin: 5px; border-right: 3px solid #ffaa00; text-align: right; }
        .lootbox-animation { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); z-index: 1000; display: none; justify-content: center; align-items: center; flex-direction: column; }
        .lootbox { width: 200px; height: 200px; background: #8B4513; border-radius: 20px; display: flex; justify-content: center; align-items: center; font-size: 80px; animation: shake 0.5s infinite; cursor: pointer; }
        @keyframes shake { 0%{transform:rotate(0deg);} 25%{transform:rotate(10deg);} 75%{transform:rotate(-10deg);} 100%{transform:rotate(0deg);} }
        .lootbox-opening { animation: openBox 0.5s forwards; }
        @keyframes openBox { 0%{transform:scale(1);} 50%{transform:scale(1.2);background:#ffaa00;} 100%{transform:scale(0);opacity:0;} }
        .reward-display { font-size: 24px; margin-top: 20px; animation: fadeIn 0.5s; text-align: center; }
        @keyframes fadeIn { from{opacity:0;transform:scale(0.5);} to{opacity:1;transform:scale(1);} }
        .btn { background: rgba(0,30,60,0.7); color:#fff; border:1px solid #00d2ff; padding:8px 20px; border-radius:25px; font-weight:bold; cursor:pointer; margin:5px; font-size:12px; transition:0.2s; backdrop-filter:blur(3px); }
        .btn:hover { background: rgba(0,80,120,0.8); transform:scale(1.02); }
        .btn-danger { border-color:#ff0044; background:rgba(80,0,0,0.7); }
        .btn-gem { border-color:#ff66ff; background:linear-gradient(45deg,#8e44ad,#ff00ff); }
        .game-select-card { background:rgba(0,30,60,0.8); border:2px solid #00d2ff; border-radius:20px; padding:25px; margin:15px; width:280px; cursor:pointer; transition:0.3s; }
        .game-select-card:hover { transform:scale(1.05); border-color:#ff00ff; background:rgba(0,50,100,0.9); }
        .game-select-card.coming-soon { opacity:0.6; border-color:#888; cursor:not-allowed; }
        .game-select-card.coming-soon:hover { transform:none; }
        .shop-grid { display:grid; grid-template-columns:1fr 1fr; gap:8px; margin:12px; width:90%; max-width:380px; }
        .card { background:rgba(255,255,255,0.05); border:1px solid #444; padding:6px; border-radius:10px; cursor:pointer; transition:0.2s; font-size:11px; position:relative; }
        .card:hover { background:rgba(0,210,255,0.1); border-color:#00d2ff; transform:scale(1.02); }
        .card.cant-afford { opacity:0.4; cursor:not-allowed; }
        .rarity-common { border-color:#4287f5; background:rgba(66,135,245,0.2); }
        .rarity-rare { border-color:#42f5b6; background:rgba(66,245,182,0.2); }
        .rarity-epic { border-color:#f5a742; background:rgba(245,167,66,0.2); }
        .rarity-legendary { border-color:#f542d1; background:rgba(245,66,209,0.2); }
        .rarity-mythic { border-color:#f54242; background:rgba(245,66,66,0.2); }
        .rarity-ultra { border-color:#f5e642; background:rgba(245,230,66,0.2); box-shadow:0 0 15px gold; }
        .gem-counter { position:absolute; top:10px; left:10px; background:rgba(0,0,0,0.6); border-radius:20px; padding:5px 12px; font-size:14px; color:#ff66ff; border:1px solid #ff66ff; z-index:200; }
        .gem-counter span { color:#ffcc00; font-weight:bold; }
        .update-item { background:rgba(0,0,0,0.5); border-radius:10px; padding:10px; margin:8px; text-align:right; border-right:3px solid #00ffaa; width: 90%; max-width: 500px; }
        .update-version { color:#00ffaa; font-weight:bold; font-size:14px; }
        .update-desc { color:#ccc; font-size:12px; margin-top:5px; }
        #ui-hud { position: absolute; top: 10px; left: 10px; z-index: 50; display: none; pointer-events: none; }
        .stat-group { margin-bottom: 6px; }
        .stat-label { font-size: 9px; font-weight: bold; color: #00d2ff; letter-spacing: 1px; margin-bottom: 1px; }
        .bar-bg { width: 120px; height: 6px; background: rgba(0,0,0,0.6); border-radius: 3px; border: 1px solid #444; overflow: hidden; }
        .bar-fill { height: 100%; transition: width 0.2s; }
        #hp-fill { background: linear-gradient(90deg, #ff0044, #ff5588); }
        #xp-fill { background: linear-gradient(90deg, #00ffaa, #00ffff); width: 0%; }
        #od-fill { background: linear-gradient(90deg, #8e44ad, #ff00ff); width: 0%; }
        #score-hud { position: absolute; top: 10px; right: 10px; z-index: 50; display: none; pointer-events: none; text-align: right; }
        #score-hud .score-main { font-size: 16px; font-weight: 900; color: #fff; text-shadow: 0 0 5px #00d2ff; }
        #score-hud .score-sub { font-size: 9px; color: #aaa; margin-top: 1px; }
        #combo-small { position: absolute; bottom: 12px; right: 12px; font-size: 16px; font-weight: bold; color: #ffcc00; display: none; z-index: 50; text-shadow: 0 0 3px orange; pointer-events: none; }
        .combo-meter { position: absolute; bottom: 50px; right: 12px; width: 100px; height: 8px; background: #333; border-radius: 4px; overflow: hidden; display: none; }
        .combo-meter-fill { height: 100%; width: 0%; background: linear-gradient(90deg, #ffaa00, #ff6600); transition: width 0.1s; }
        #od-btn { position: absolute; bottom: 15px; right: 15px; width: 60px; height: 60px; background: rgba(142,68,173,0.5); border: 2px solid #8e44ad; border-radius: 50%; display: none; justify-content: center; align-items: center; z-index: 500; color: #fff; font-weight: bold; cursor: pointer; font-size: 9px; line-height: 1.2; }
        #powerup-bar { position:absolute; bottom:85px; left:50%; transform:translateX(-50%); z-index:50; display:none; pointer-events:none; text-align:center; white-space:nowrap; }
        .pu-item { display:inline-block; margin:0 2px; background:rgba(0,0,0,0.7); border-radius:6px; padding:1px 5px; border:1px solid #555; font-size:9px; font-weight:bold; }
        #wave-banner { position:absolute; top:50%; left:50%; transform:translate(-50%,-50%); font-size:28px; font-weight:900; color:#00d2ff; text-shadow:0 0 15px #00d2ff; display:none; z-index:200; pointer-events:none; text-align:center; white-space:nowrap; background:rgba(0,0,0,0.6); padding:5px 15px; border-radius:30px; }
        #event-banner { position:absolute; top:40%; left:50%; transform:translate(-50%,-50%); font-size:24px; font-weight:900; display:none; z-index:200; pointer-events:none; text-align:center; white-space:nowrap; background:rgba(0,0,0,0.7); padding:5px 15px; border-radius:30px; }
        #boss-warning { position:absolute; top:50%; left:50%; transform:translate(-50%,-50%); font-size:28px; font-weight:900; color:#ff0044; text-shadow:0 0 15px #ff0044; display:none; z-index:200; pointer-events:none; animation:bossWarn 0.3s infinite alternate; background:rgba(0,0,0,0.5); padding:4px 12px; border-radius:25px; white-space:nowrap; }
        @keyframes bossWarn { from{opacity:0.8;} to{opacity:0.3;} }
        #crosshair { position:fixed; z-index:999; pointer-events:none; display:none; top:0; left:0; }
        #vignette { position:fixed; top:0; left:0; width:100%; height:100%; pointer-events:none; z-index:49; display:none; background:radial-gradient(circle,transparent 50%,rgba(255,0,0,0.25) 100%); }
        #nebulaCanvas { position:absolute; top:0; left:0; z-index:0; pointer-events:none; opacity:0.12; }
        .achievements-grid { display:grid; grid-template-columns:1fr 1fr; gap:6px; margin:12px; max-width:750px; width: 95%; padding:6px; }
        .ach-card { background:rgba(0,0,0,0.6); border:1px solid #555; border-radius:6px; padding:4px 6px; text-align:right; font-size:10px; position:relative; }
        .ach-card.locked { opacity:0.5; filter:grayscale(0.3); }
        .ach-name { color:gold; font-weight:bold; font-size:11px; }
        .ach-desc { color:#aaa; font-size:8px; }
        .ach-status { font-size:12px; margin-left:4px; }
        .ach-progress-bar { height:3px; background:#333; border-radius:2px; margin-top:3px; overflow:hidden; }
        .ach-progress-fill { height:100%; width:0%; background:linear-gradient(90deg,gold,#ffcc44); transition:width 0.2s; }
        .ach-progress-text { font-size:7px; color:#888; margin-top:2px; text-align:left; }
        .events-grid { display:grid; grid-template-columns:1fr; gap:8px; margin:15px; max-width:500px; width: 95%; padding:8px; }
        .event-card { background:rgba(0,0,0,0.7); border:1px solid #ff00ff; border-radius:10px; padding:8px 12px; text-align:right; }
        .event-name { color:#ff00ff; font-weight:bold; font-size:14px; }
        .event-chance { color:#ffaa00; font-size:11px; }
        .event-desc { color:#ccc; font-size:11px; margin-top:4px; }
        .skins-grid { display:grid; grid-template-columns:1fr 1fr; gap:12px; margin:12px; max-width:600px; width: 95%; padding:6px; }
        .skin-card { background:rgba(0,0,0,0.6); border:2px solid #555; border-radius:12px; padding:10px; text-align:center; cursor:pointer; transition:0.2s; }
        .skin-card.owned { border-color:gold; background:rgba(255,215,0,0.1); }
        .skin-card.locked { opacity:0.5; filter:grayscale(0.5); cursor:not-allowed; }
        .skin-card.equipped { border-color:#ff66ff; box-shadow:0 0 15px #ff66ff; }
        .skin-icon { font-size:48px; margin-bottom:8px; }
        .skin-name { font-weight:bold; font-size:14px; margin-bottom:4px; }
        .skin-desc { font-size:10px; color:#aaa; }
        .skin-effect { font-size:9px; color:#ffaa00; margin-top:5px; }
        .quantity-selector { display:flex; gap:4px; margin-top:6px; justify-content:center; }
        .qty-btn { background:#333; border:none; color:#fff; border-radius:4px; padding:2px 6px; font-size:9px; cursor:pointer; }
        #pause-btn { position:absolute; top:10px; right:50%; transform:translateX(50%); z-index:50; display:none; background:rgba(0,0,0,0.5); border:1px solid #666; color:#fff; padding:4px 12px; border-radius:15px; cursor:pointer; font-size:10px; pointer-events:all; }
        .start-stats { background:rgba(0,0,0,0.4); border:1px solid #00d2ff33; border-radius:12px; padding:8px 15px; margin:10px 0; }
        .reset-section { margin-top:20px; padding-top:15px; border-top:1px solid #ff0044; }
        .achievement-popup-fixed { position: fixed; bottom: 20px; left: 20px; background: linear-gradient(135deg, rgba(0,30,60,0.95), rgba(0,10,30,0.95)); border: 2px solid gold; border-radius: 12px; padding: 10px 16px; min-width: 220px; max-width: 300px; z-index: 300; display: none; animation: slideIn 0.3s ease; pointer-events: none; }
        .achievement-popup-fixed .title { color: gold; font-size: 11px; font-weight: bold; letter-spacing: 1px; }
        .achievement-popup-fixed .name { font-size: 14px; font-weight: bold; margin-top: 2px; }
        .achievement-popup-fixed .desc { font-size: 11px; color: #ccc; margin-top: 2px; }
        @keyframes slideIn { from { transform: translateX(-120%); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
        .rank-badge { position: absolute; top: 80px; left: 10px; background: linear-gradient(135deg, #ffaa00, #ff6600); border-radius: 20px; padding: 4px 12px; font-size: 10px; font-weight: bold; color: #000; z-index: 55; display: none; }
        .daily-mission-header { background: linear-gradient(135deg, #00d2ff, #00ffaa); border-radius: 10px; padding: 5px 15px; margin-bottom: 10px; color: #000; font-weight: bold; }
        .stats-panel { background: rgba(0,0,0,0.5); border-radius: 10px; padding: 8px; margin-top: 10px; font-size: 11px; display: flex; justify-content: space-between; flex-wrap: wrap; gap: 8px; }
        .critical-hit { animation: critFlash 0.2s ease-out; }
        @keyframes critFlash { 0% { text-shadow: 0 0 0px #ffaa00; } 50% { text-shadow: 0 0 20px #ffaa00; } 100% { text-shadow: 0 0 0px #ffaa00; } }
    </style>
</head>
<body>
<div id="loader-init" class="loader-overlay">
    <div class="loader-content">
        <h1 style="color:#00d2ff;letter-spacing:2px;font-size:1.5rem;">WARP INITIATED</h1>
        <p style="color:#00ffaa;font-size:9px;">CALIBRATING...</p>
        <div class="p-bar-outer"><div id="init-fill" class="p-bar-inner"></div></div>
    </div>
</div>
<div id="loader-death" class="loader-overlay" style="display:none;">
    <div class="loader-content">
        <h1 style="color:#ff0044;letter-spacing:2px;font-size:1.5rem;">CORE FAILURE</h1>
        <p style="color:#aaa;font-size:9px;">RESTORING...</p>
        <div class="p-bar-outer"><div id="death-fill" class="p-bar-inner" style="background:#ff0044;"></div></div>
    </div>
</div>

<div id="notification-area" class="notification-area"></div>
<div id="custom-alert" class="custom-alert">
    <p id="alert-message"></p>
    <button onclick="closeCustomAlert()">OK</button>
</div>
<div id="rank-badge" class="rank-badge">🏆 RANK 1</div>
<div id="gem-counter" class="gem-counter" style="display:none;">💎 GEMSTONES: <span id="gemstones-amount">0</span></div>
<div id="game-timer">⏱️ <span id="timer-display">00:00</span></div>

<!-- MAIN HUB -->
<div id="main-hub" class="overlay">
    <h1 style="font-size:48px;margin-bottom:5px;text-shadow:0 0 20px #00d2ff;">✨ GALACTIC DEFENDER ✨</h1>
    <p style="color:#ff66ff;margin-bottom:20px;">UPDATE 10.5</p>
    <div style="display:flex;flex-wrap:wrap;justify-content:center;gap:15px;margin:20px;">
        <button class="btn" style="font-size:16px;padding:12px 30px;" onclick="openGameSelect()">🎮 SELECT GAME</button>
        <button class="btn" style="font-size:16px;padding:12px 30px;border-color:#00ffaa;" onclick="openUpdateLog()">📜 UPDATE LOG</button>
        <button class="daily-reward-btn" onclick="claimDailyReward()">🎁 DAILY REWARD</button>
    </div>
    <div class="start-stats" style="background:rgba(0,0,0,0.6);border-radius:20px;padding:15px;margin:10px auto;width:90%;max-width:400px;">
        <div style="font-size:14px;">🏆 RECORD: <span id="hub-hi" style="color:gold">0</span></div>
        <div style="font-size:14px;">💰 CREDITS: <span id="hub-coins" style="color:#00ffaa">0</span></div>
        <div style="font-size:14px;">💎 GEMSTONES: <span id="hub-gems" style="color:#ff66ff">0</span></div>
        <div style="font-size:14px;">💀 KILLS: <span id="hub-kills" style="color:#ff4444">0</span></div>
        <div style="font-size:14px;">🌟 SKIN: <span id="hub-skin" style="color:gold">LOCKED</span></div>
        <div id="hub-skin-progress-bar" style="width:100%;height:6px;background:#333;border-radius:3px;margin:8px auto;overflow:hidden;"><div id="hub-skin-progress-fill" style="height:100%;width:0%;background:linear-gradient(90deg,gold,#ffcc44);transition:width 0.3s;"></div></div>
        <div id="hub-skin-percent" style="font-size:10px;color:#aaa;">0/40 ACHIEVEMENTS</div>
    </div>
    <div class="stats-panel" style="width:90%;max-width:400px;margin:10px auto;">
        <div>🎯 CRITICAL HITS: <span id="stat-crits">0</span> (<span id="stat-crit-rate">15</span>%)</div>
        <div>💥 TOTAL DAMAGE: <span id="stat-damage">0</span></div>
        <div>⚡ OVERDRIVES: <span id="stat-od">0</span></div>
        <div>💀 BOSSES SLAIN: <span id="stat-bosses">0</span></div>
    </div>
    <div style="margin-top:20px;padding:15px;background:rgba(0,0,0,0.4);border-radius:15px;width:90%;max-width:500px;">
        <h3 style="color:#ffaa00;">📋 DAILY MISSIONS</h3>
        <div id="daily-missions-container"></div>
    </div>
    <div style="margin-top:20px;padding:15px;background:rgba(0,0,0,0.4);border-radius:15px;width:90%;max-width:500px;margin-bottom:30px;">
        <h3 style="color:#00d2ff;">📋 GAME INFO</h3>
        <p style="font-size:12px;color:#ccc;"><strong>🚀 GALACTIC DEFENDER:</strong> משחק יריות בחלל עם בוסים, אירועים מיוחדים, מערכת שדרוגים, הישגים ועוד! הגן על החללית שלך והשמד את כל האויבים.</p>
        <p style="font-size:12px;color:#ccc;margin-top:8px;"><strong>❓ GAME 2 (COMING SOON):</strong> המשחק השני נמצא בשלבי פיתוח מתקדמים! צפוי לצאת בקרוב עם מכניקות חדשות ומרגשות. הישארו מעודכנים!</p>
        <p style="font-size:11px;color:#ffaa00;margin-top:8px;">✨ עדכון 10.5 - 2 אירועים חדשים + 15 הישגים + כפתור הגדרות + QOL!</p>
    </div>
    <div style="margin-top:15px;margin-bottom:30px;color:#666;font-size:9px;">© Galactic Defender - All Rights Reserved</div>
</div>

<div id="game-select-screen" class="overlay">
    <h1 style="font-size:48px;margin-bottom:10px;text-shadow:0 0 20px #00d2ff;">🎮 GAME SELECT</h1>
    <div style="display:flex;flex-wrap:wrap;justify-content:center;gap:20px;margin:20px;">
        <div class="game-select-card" onclick="selectGame('defender')">
            <div style="font-size:48px;">🚀</div>
            <h2 style="color:#00d2ff;">GALACTIC DEFENDER</h2>
            <p>המשחק הקלאסי! יריות, בוסים, אירועים והישגים</p>
            <p style="color:#00ffaa;font-size:12px;margin-top:10px;">▶ לחץ כדי לשחק</p>
            <div style="margin-top:8px;font-size:10px;color:#ffaa00;">✨ עדכון 10.5: 40 אירועים + 310+ הישגים + מערכת סקינים + פרס יומי!</div>
        </div>
        <div class="game-select-card coming-soon" onclick="showCustomAlert('משחק זה עדיין בפיתוח! יגיע בקרוב...')">
            <div style="font-size:48px;">❓</div>
            <h2 style="color:#888;">COMING SOON</h2>
            <p>משחק חדש בפיתוח... בקרוב!</p>
            <p style="color:#ffaa00;font-size:12px;margin-top:10px;">⏳ בשלבי הכנה אחרונים</p>
            <div style="margin-top:8px;font-size:10px;color:#888;">צפוי לצאת בקרוב עם מכניקות חדשות!</div>
        </div>
    </div>
    <button class="btn" style="border-color:#888;margin-bottom:30px;" onclick="backToHub()">← BACK TO HUB</button>
</div>

<!-- SETTINGS SCREEN (with Music) -->
<div id="settings-screen" class="overlay">
    <h2 style="color:#00ffaa;">⚙️ SETTINGS</h2>
    <div class="settings-option">
        <span>🔔 NOTIFICATIONS</span>
        <div id="setting-notifications" class="settings-toggle on" onclick="toggleSetting('notifications')"></div>
    </div>
    <div class="settings-option">
        <span>🔊 SOUND EFFECTS</span>
        <div id="setting-sound" class="settings-toggle on" onclick="toggleSetting('sound')"></div>
    </div>
    <div class="settings-option">
        <span>🎵 BACKGROUND MUSIC</span>
        <div id="setting-music" class="settings-toggle on" onclick="toggleMusic()"></div>
    </div>
    <div class="settings-option">
        <span>📳 SCREEN SHAKE</span>
        <div id="setting-shake" class="settings-toggle on" onclick="toggleSetting('shake')"></div>
    </div>
    <div class="settings-option">
        <span>🎨 GRAPHICS QUALITY</span>
        <div style="display:flex;gap:10px;">
            <button class="btn" style="padding:4px 12px;" onclick="setGraphicsQuality('low')">LOW</button>
            <button class="btn" style="padding:4px 12px;" onclick="setGraphicsQuality('medium')">MED</button>
            <button class="btn" style="padding:4px 12px;" onclick="setGraphicsQuality('high')">HIGH</button>
        </div>
    </div>
    <div class="settings-option">
        <span>🎯 CRITICAL HIT CHANCE</span>
        <span><span id="crit-chance-display">15</span>%</span>
    </div>
    <button class="btn" onclick="closeSettings()" style="margin-bottom:30px;">← BACK</button>
</div>

<div id="update-log-screen" class="overlay">
    <h1 style="color:#00ffaa;">📜 UPDATE LOG</h1>
    <div style="width:90%;max-width:600px;margin:10px auto;text-align:right;">
        <div class="update-item"><div class="update-version">⚡ v10.5 - MEGA UPDATE (Current)</div><div class="update-desc">• 2 אירועים חדשים! DOPPELGANGER + SUPERNOVA!<br>• 15 הישגים חדשים! (סה"כ 310+ הישגים!)<br>• כפתור הגדרות חדש!<br>• CRITICAL HITS!<br>• COMBO METER ויזואלי!<br>• STATS PANEL<br>• AUTO-SAVE כל 30 שניות!<br>• 🎵 מערכת מוזיקת רקע! מוזיקה דינמית לפי מצב המשחק!</div></div>
        <div class="update-item"><div class="update-version">🌟 v10.0 - MEGA UPDATE</div><div class="update-desc">• 10 אירועים חדשים! STARFALL, INFERNO, CHAIN LIGHTNING, BARRIER, SOUL REAPER, GAMBLER, VORTEX, KING'S BLESSING, PRISM, ABYSS!<br>• 30 הישגים חדשים!<br>• מערכת RANK חדשה! 50 רמות עם בונוסים!<br>• DAILY MISSIONS!</div></div>
        <div class="update-item"><div class="update-version">🔧 v9.1 - BUG FIX</div><div class="update-desc">• תיקון באג קריסת המשחק</div></div>
        <div class="update-item"><div class="update-version">🎁 v8.0 - DAILY REWARD</div><div class="update-desc">• פרס יומי + 3 אירועים חדשים</div></div>
        <div class="update-item"><div class="update-version">🚀 v1.0 - LAUNCH</div><div class="update-desc">• השקה ראשונית! 9 אירועים, 100 הישגים</div></div>
    </div>
    <button class="btn" onclick="backToHub()" style="margin-bottom:30px;">← BACK TO HUB</button>
</div>

<div id="start-screen" class="overlay">
    <h1>CORE</h1>
    <h2 style="color:#ff00ff;letter-spacing:4px;margin-bottom:8px;font-size:0.9rem;">RECOVERED</h2>
    <div class="start-stats">
        <div style="font-size:11px;">🏆 RECORD: <span id="menu-hi" style="color:gold">0</span></div>
        <div style="font-size:11px;">💰 CREDITS: <span id="menu-coins" style="color:#00ffaa">0</span></div>
        <div style="font-size:11px;">💀 KILLS: <span id="menu-kills" style="color:#ff4444">0</span></div>
        <div style="font-size:11px;">🌟 SKIN: <span id="skin-status-text" style="color:gold">LOCKED</span></div>
        <div id="skin-progress-bar"><div id="skin-progress-fill"></div></div>
        <div id="skin-percent" style="font-size:9px;color:#aaa;">0/40 ACHIEVEMENTS</div>
    </div>
    <button class="btn" onclick="startGame()">🚀 ENGAGE</button>
    <button class="btn" style="border-color:#ff00ff;" onclick="openShop()">⚙️ UPGRADES</button>
    <button class="btn" style="border-color:#ff66ff;" onclick="openRNGShop()">🎲 LOOTBOXES</button>
    <button class="btn" style="border-color:#888;margin-bottom:30px;" onclick="backToGameSelect()">← BACK</button>
    <div style="color:#888;font-size:8px;margin-top:8px;margin-bottom:20px;">MOUSE/TOUCH | ESC | Q bomb | O overdrive</div>
    <div id="ach-side-btn" class="side-btn" onclick="openAchievements()">🏆</div>
    <div id="events-side-btn" class="side-btn" onclick="openEvents()">📋</div>
    <div id="skins-side-btn" class="side-btn" onclick="openSkins()">🎨</div>
    <div id="settings-side-btn" class="side-btn" onclick="openSettings()">⚙️</div>
</div>

<div id="skins-screen" class="overlay">
    <h2 style="color:#ff66ff;">🎨 SKIN COLLECTION 🎨</h2>
    <div class="skins-grid" id="skins-grid-container"></div>
    <button class="btn" onclick="closeSkins()" style="margin-bottom:30px;">← BACK</button>
</div>

<div id="rng-shop-screen" class="overlay">
    <h2 style="color:#ff66ff;">🎲 LOOTBOXES & REWARDS 🎲</h2>
    <div style="margin:10px;">💎 YOUR GEMSTONES: <span id="rng-gems" style="color:#ffcc00;font-size:24px;">0</span></div>
    <div class="shop-grid" style="grid-template-columns:1fr 1fr;max-width:500px;">
        <div class="card rarity-common" onclick="openLootbox('common')"><div style="font-size:20px;">📦</div><div>COMMON LOOTBOX</div><div style="color:#4287f5;">🔵 COMMON</div><div style="color:gold;">50 💎</div><div style="font-size:9px;color:#aaa;">פריטים נדירים בסיסיים</div></div>
        <div class="card rarity-rare" onclick="openLootbox('rare')"><div style="font-size:20px;">📦</div><div>RARE LOOTBOX</div><div style="color:#42f5b6;">🟢 RARE</div><div style="color:gold;">150 💎</div><div style="font-size:9px;color:#aaa;">סיכוי לפריטים טובים יותר</div></div>
        <div class="card rarity-epic" onclick="openLootbox('epic')"><div style="font-size:20px;">📦</div><div>EPIC LOOTBOX</div><div style="color:#f5a742;">🟡 EPIC</div><div style="color:gold;">400 💎</div><div style="font-size:9px;color:#aaa;">פריטים אפיים!</div></div>
        <div class="card rarity-legendary" onclick="openLootbox('legendary')"><div style="font-size:20px;">📦</div><div>LEGENDARY LOOTBOX</div><div style="color:#f542d1;">🟠 LEGENDARY</div><div style="color:gold;">1000 💎</div><div style="font-size:9px;color:#aaa;">פריטים אגדיים!</div></div>
        <div class="card rarity-mythic" onclick="openLootbox('mythic')"><div style="font-size:20px;">📦</div><div>MYTHIC LOOTBOX</div><div style="color:#f54242;">🔴 MYTHIC</div><div style="color:gold;">2500 💎</div><div style="font-size:9px;color:#aaa;">פריטים מיתיים נדירים!</div></div>
        <div class="card rarity-ultra" onclick="openLootbox('ultra')"><div style="font-size:20px;">👑</div><div>ULTRA MYTHIC BOX</div><div style="color:#f5e642;">💎 ULTRA MYTHIC</div><div style="color:gold;">10000 💎</div><div style="font-size:9px;color:#aaa;">הפריטים הנדירים ביותר! כמות מוגבלת!</div></div>
    </div>
    <h3 style="margin-top:20px;color:#ffaa00;">🎁 BUY INDIVIDUAL REWARDS</h3>
    <div class="shop-grid" style="grid-template-columns:1fr 1fr;max-width:500px;" id="individual-rewards"></div>
    <button class="btn" onclick="closeRNGShop()" style="margin-bottom:30px;">← BACK</button>
</div>

<div id="lootbox-animation" class="lootbox-animation">
    <div id="lootbox-element" class="lootbox">📦</div>
    <div id="lootbox-result" class="reward-display" style="display:none;"></div>
    <button id="lootbox-close-btn" class="btn" style="margin-top:20px;display:none;" onclick="closeLootboxAnimation()">CLOSE</button>
</div>

<div id="ascend-screen" class="overlay">
    <h1 style="color:gold;">🌟 ASCENSION 🌟</h1>
    <p style="font-size:16px;margin:10px;">You have reached <span id="ascend-wave" style="color:#00ffaa;font-weight:bold;">100</span>!</p>
    <p>Do you wish to continue into <span style="color:#ffcc00;">ENDLESS MODE</span>?</p>
    <p style="font-size:12px;color:#aaa;">• Enemies get stronger every wave<br>• No more forced bosses<br>• Infinite progression<br>• Special achievements await!</p>
    <button class="btn btn-ascend" onclick="continueEndless()">✨ YES, ASCEND ✨</button>
    <button class="btn" onclick="quitToMenu()">BACK TO MENU</button>
</div>

<!-- GAME UI -->
<div id="combo-small">x1</div>
<div class="combo-meter" id="combo-meter"><div class="combo-meter-fill" id="combo-meter-fill"></div></div>
<div id="ui-hud">
    <div class="stat-group"><div class="stat-label">🛡️ <span id="hp-num"></span></div><div class="bar-bg"><div id="hp-fill" class="bar-fill"></div></div></div>
    <div class="stat-group"><div class="stat-label">⭐ XP</div><div class="bar-bg"><div id="xp-fill" class="bar-fill"></div></div></div>
    <div class="stat-group"><div class="stat-label">⚡ OD</div><div class="bar-bg"><div id="od-fill" class="bar-fill"></div></div></div>
    <div style="font-size:9px;">💣 <span id="bomb-count" style="color:#ffcc00;">0</span></div>
</div>
<div id="score-hud">
    <div class="score-main">SCORE: <span id="score-val">0</span></div>
    <div class="score-sub">RANK: <span id="lvl-val">1</span></div>
    <div class="score-sub">KILLS: <span id="kill-val">0</span></div>
    <div class="score-sub">WAVE: <span id="wave-val">1</span></div>
</div>
<div id="powerup-bar"></div>

<div id="shop-screen" class="overlay">
    <h2 style="color:#00d2ff;font-size:1.3rem;">🚀 GALACTIC DEFENDER - TECH HANGAR</h2>
    <div id="shop-money" style="color:gold;font-size:16px;">CREDITS: 0</div>
    <div class="shop-grid" id="shop-grid-container"></div>
    <h3 style="color:#ffaa00;margin-top:20px;">✨ SPECIAL ITEMS ✨</h3>
    <div class="shop-grid" style="margin-top:10px;">
        <div class="card tooltip" onclick="buySpecialItem('stardust')"><div style="font-size:20px;">⭐</div><div>STARDUST (One-Time)</div><div style="color:#ffaa00;">+10% ALL STATS</div><div style="color:gold;">5000c</div><div class="tooltip-text">מגדיל את כל הסטטים ב-10% לצמיתות! ניתן לקנות פעם אחת בלבד.</div></div>
        <div class="card tooltip" onclick="buySpecialItem('guardian')"><div style="font-size:20px;">🛡️</div><div>GUARDIAN ANGEL (Limited)</div><div style="color:#ffaa00;">חסינות למוות פעם אחת</div><div style="color:gold;">3000c</div><div style="font-size:9px;color:#aaa;">נותרו: <span id="guardian-count">3</span>/3</div><div class="tooltip-text">מעניק חסינות למוות פעם אחת. ניצל אותך אוטומטית!</div></div>
        <div class="card tooltip" onclick="buySpecialItem('powercore')"><div style="font-size:20px;">⚡</div><div>POWER CORE (Infinite)</div><div style="color:#ffaa00;">+5% DAMAGE</div><div style="color:gold;">2000c</div><div style="font-size:9px;color:#aaa;">ניתן לקנות עד 20 פעמים</div><div class="tooltip-text">מגדיל את הנזק ב-5% לצמיתות. ניתן לקנות עד 20 פעמים.</div></div>
        <div class="card tooltip" onclick="buySpecialItem('cosmic')"><div style="font-size:20px;">🌟</div><div>COSMIC ESSENCE (Limited)</div><div style="color:#ffaa00;">+3% FIRE RATE</div><div style="color:gold;">1500c</div><div style="font-size:9px;color:#aaa;">ניתן לקנות עד 15 פעמים</div><div class="tooltip-text">מגדיל את מהירות הירי ב-3% לצמיתות. ניתן לקנות עד 15 פעמים.</div></div>
        <div class="card tooltip" onclick="buySpecialItem('ironwill')"><div style="font-size:20px;">🛡️</div><div>IRON WILL (One-Time)</div><div style="color:#ffaa00;">50% DAMAGE REDUCTION</div><div style="color:gold;">5000c</div><div class="tooltip-text">מקנה 50% חסינות לנזק לצמיתות! מחצית מהנזק שיתקבל.</div></div>
        <div class="card tooltip" onclick="buySpecialItem('timedistortion')"><div style="font-size:20px;">⏰</div><div>TIME DISTORTION (One-Time)</div><div style="color:#ffaa00;">30% SLOW ENEMIES</div><div style="color:gold;">8000c</div><div class="tooltip-text">מאט את כל האויבים ב-30% לצמיתות!</div></div>
        <div class="card tooltip" onclick="buySpecialItem('crystalheart')"><div style="font-size:20px;">💎</div><div>CRYSTAL HEART (One-Time)</div><div style="color:#ffaa00;">2x GEMSTONES</div><div style="color:gold;">10000c</div><div class="tooltip-text">מכפיל את כל ה-GEMSTONES שיתקבלו בעתיד ב-2!</div></div>
    </div>
    <button class="btn" onclick="closeShop()" style="margin-bottom:30px;">← BACK</button>
    <div class="reset-section" style="margin-top:15px;margin-bottom:30px;">
        <button class="btn btn-danger" style="padding:5px 15px;font-size:10px;" onclick="confirmReset()">🔄 RESET UPGRADES (Refund)</button>
    </div>
</div>

<div id="achievements-screen" class="overlay">
    <h2 style="color:gold;">🏆 ACHIEVEMENTS (310+ TOTAL)</h2>
    <div id="ach-list" class="achievements-grid"></div>
    <button class="btn" onclick="closeAchievements()" style="margin-bottom:30px;">BACK</button>
</div>

<div id="events-screen" class="overlay">
    <h2 style="color:#ff00ff;">📋 EVENT LIST (40 EVENTS)</h2>
    <div class="events-grid" id="events-list-container"></div>
    <button class="btn" onclick="closeEvents()" style="margin-bottom:30px;">BACK</button>
</div>

<div id="pause-screen" class="overlay">
    <h2 style="color:#00d2ff;">PAUSED</h2>
    <button class="btn" onclick="togglePause()">RESUME</button>
    <button class="btn" onclick="quitToMenu()">MAIN MENU</button>
</div>
<div id="game-over" class="overlay">
    <h1 style="color:#ff0044;font-size:1.8rem;">MISSION ENDED</h1>
    <div id="final-stats" style="font-size:12px;margin-bottom:12px;"></div>
    <button class="btn" onclick="triggerReboot()">RE-INITIALIZE</button>
    <button class="btn" onclick="quitToMenu()">MAIN MENU</button>
</div>

<canvas id="gameCanvas"></canvas>
<canvas id="nebulaCanvas"></canvas>
<div id="vignette"></div>
<div id="boss-warning">⚠ BOSS INCOMING ⚠</div>
<div id="wave-banner"></div>
<div id="event-banner"></div>
<div id="od-btn" onclick="activateOverdrive()">OVER<br>DRIVE</div>
<button id="pause-btn" onclick="togglePause()">⏸</button>
<div id="achievement-popup" class="achievement-popup-fixed">
    <div class="title">🏆 ACHIEVEMENT UNLOCKED</div>
    <div id="ach-name" class="name"></div>
    <div id="ach-desc" class="desc"></div>
</div>
<canvas id="crosshair"></canvas>

<script>
// ============================================
// MUSIC SYSTEM - Background Music
// ============================================
let backgroundMusic = null;
let musicEnabled = localStorage.getItem('musicEnabled') !== 'false';
let currentMusicType = 'menu';
let musicVolume = 0.12;
let musicLoopInterval = null;
let musicStopRequested = false;

function stopBackgroundMusic() {
    musicStopRequested = true;
    if (backgroundMusic) {
        try {
            if (backgroundMusic.osc1) { backgroundMusic.osc1.stop(); backgroundMusic.osc1.disconnect(); }
            if (backgroundMusic.osc2) { backgroundMusic.osc2.stop(); backgroundMusic.osc2.disconnect(); }
            if (backgroundMusic.osc) { backgroundMusic.osc.stop(); backgroundMusic.osc.disconnect(); }
            if (backgroundMusic.masterGain) backgroundMusic.masterGain.disconnect();
        } catch(e) { console.log("Music stop error:", e); }
        backgroundMusic = null;
    }
    if (musicLoopInterval) {
        clearInterval(musicLoopInterval);
        musicLoopInterval = null;
    }
    setTimeout(() => { musicStopRequested = false; }, 100);
}

function playBackgroundMusic(type) {
    if (!musicEnabled) return;
    if (musicStopRequested) return;
    if (!audioCtx && settings.sound) {
        try {
            audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        } catch(e) { return; }
    }
    if (!audioCtx) return;
    if (audioCtx.state === 'suspended') {
        audioCtx.resume().catch(e => console.log("AudioContext resume failed"));
    }
    if (currentMusicType === type && backgroundMusic && backgroundMusic.masterGain) {
        return;
    }
    stopBackgroundMusic();
    currentMusicType = type;
    try {
        const now = audioCtx.currentTime;
        const masterGain = audioCtx.createGain();
        masterGain.gain.value = musicVolume;
        masterGain.connect(audioCtx.destination);
        switch(type) {
            case 'menu':
                const osc1 = audioCtx.createOscillator();
                const osc2 = audioCtx.createOscillator();
                const gain1 = audioCtx.createGain();
                const gain2 = audioCtx.createGain();
                osc1.connect(gain1);
                osc2.connect(gain2);
                gain1.connect(masterGain);
                gain2.connect(masterGain);
                osc1.type = 'sine';
                osc2.type = 'sine';
                osc1.frequency.value = 174.61;
                osc2.frequency.value = 261.63;
                gain1.gain.setValueAtTime(0, now);
                gain2.gain.setValueAtTime(0, now);
                gain1.gain.linearRampToValueAtTime(0.07, now + 1);
                gain2.gain.linearRampToValueAtTime(0.05, now + 1.5);
                osc1.start();
                osc2.start();
                backgroundMusic = { osc1, osc2, gain1, gain2, masterGain };
                let menuLFO = setInterval(() => {
                    if (!backgroundMusic || !musicEnabled || musicStopRequested) return;
                    try {
                        const freq1 = 174.61 + Math.sin(Date.now() / 6000) * 2;
                        const freq2 = 261.63 + Math.sin(Date.now() / 7000) * 1.5;
                        if (backgroundMusic.osc1) backgroundMusic.osc1.frequency.value = freq1;
                        if (backgroundMusic.osc2) backgroundMusic.osc2.frequency.value = freq2;
                    } catch(e) {}
                }, 500);
                musicLoopInterval = menuLFO;
                break;
            case 'gameplay':
                const gOsc = audioCtx.createOscillator();
                const gGain = audioCtx.createGain();
                gOsc.connect(gGain);
                gGain.connect(masterGain);
                gOsc.type = 'sawtooth';
                gOsc.frequency.value = 130.81;
                gGain.gain.setValueAtTime(0, now);
                gGain.gain.linearRampToValueAtTime(0.06, now + 1);
                gOsc.start();
                backgroundMusic = { osc: gOsc, gain: gGain, masterGain };
                let gameLFO = setInterval(() => {
                    if (!backgroundMusic || !musicEnabled || musicStopRequested) return;
                    try {
                        const notes = [130.81, 146.83, 164.81, 146.83];
                        const idx = Math.floor(Date.now() / 350) % notes.length;
                        if (backgroundMusic.osc) backgroundMusic.osc.frequency.value = notes[idx];
                    } catch(e) {}
                }, 350);
                musicLoopInterval = gameLFO;
                break;
            case 'boss':
                const bOsc = audioCtx.createOscillator();
                const bGain = audioCtx.createGain();
                bOsc.connect(bGain);
                bGain.connect(masterGain);
                bOsc.type = 'square';
                bOsc.frequency.value = 87.31;
                bGain.gain.setValueAtTime(0, now);
                bGain.gain.linearRampToValueAtTime(0.09, now + 0.5);
                bOsc.start();
                backgroundMusic = { osc: bOsc, gain: bGain, masterGain };
                let bossLFO = setInterval(() => {
                    if (!backgroundMusic || !musicEnabled || musicStopRequested) return;
                    try {
                        const pulse = 87.31 + (Math.sin(Date.now() / 250) * 8);
                        if (backgroundMusic.osc) backgroundMusic.osc.frequency.value = pulse;
                    } catch(e) {}
                }, 150);
                musicLoopInterval = bossLFO;
                break;
            case 'event':
                const eOsc = audioCtx.createOscillator();
                const eGain = audioCtx.createGain();
                eOsc.connect(eGain);
                eGain.connect(masterGain);
                eOsc.type = 'triangle';
                eOsc.frequency.value = 220.00;
                eGain.gain.setValueAtTime(0, now);
                eGain.gain.linearRampToValueAtTime(0.07, now + 0.8);
                eOsc.start();
                backgroundMusic = { osc: eOsc, gain: eGain, masterGain };
                let eventLFO = setInterval(() => {
                    if (!backgroundMusic || !musicEnabled || musicStopRequested) return;
                    try {
                        const sweep = 220 + Math.sin(Date.now() / 400) * 20;
                        if (backgroundMusic.osc) backgroundMusic.osc.frequency.value = sweep;
                    } catch(e) {}
                }, 200);
                musicLoopInterval = eventLFO;
                break;
        }
    } catch(e) { console.log("Music error:", e); }
}

function updateMusicBasedOnGameState() {
    if (!musicEnabled) { stopBackgroundMusic(); return; }
    if (gameState !== 'PLAYING') { playBackgroundMusic('menu'); return; }
    if (boss !== null || guardian !== null) { playBackgroundMusic('boss'); }
    else if (activeEvent && (apocalypseActive || cosmicCollapseActive || primordialRageActive || voidActive)) { playBackgroundMusic('event'); }
    else { playBackgroundMusic('gameplay'); }
}

function toggleMusic() {
    musicEnabled = !musicEnabled;
    localStorage.setItem('musicEnabled', musicEnabled);
    const toggle = document.getElementById('setting-music');
    if (toggle) toggle.classList.toggle('on', musicEnabled);
    if (musicEnabled) {
        updateMusicBasedOnGameState();
        if (window.showNotification) showNotification('🎵 Music ON', 'success');
        else alert('🎵 Music ON');
    } else {
        stopBackgroundMusic();
        if (window.showNotification) showNotification('🔇 Music OFF', 'info');
        else alert('🔇 Music OFF');
    }
}

// ============================================
// SETTINGS SYSTEM
// ============================================
let settings = {
    notifications: localStorage.getItem('notificationsEnabled') !== 'false',
    sound: localStorage.getItem('soundEnabled') !== 'false',
    shake: localStorage.getItem('shakeEnabled') !== 'false',
    graphics: localStorage.getItem('graphicsQuality') || 'high'
};

function initSettingsUI(){
    const notifToggle = document.getElementById('setting-notifications');
    const soundToggle = document.getElementById('setting-sound');
    const shakeToggle = document.getElementById('setting-shake');
    const musicToggle = document.getElementById('setting-music');
    if(notifToggle) notifToggle.classList.toggle('on', settings.notifications);
    if(soundToggle) soundToggle.classList.toggle('on', settings.sound);
    if(shakeToggle) shakeToggle.classList.toggle('on', settings.shake);
    if(musicToggle) musicToggle.classList.toggle('on', musicEnabled);
}

function toggleSetting(setting){
    settings[setting] = !settings[setting];
    localStorage.setItem(setting + 'Enabled', settings[setting]);
    const toggle = document.getElementById('setting-' + setting);
    if(toggle) toggle.classList.toggle('on', settings[setting]);
    if(setting === 'sound' && !settings.sound && audioCtx){}
    showNotification(`${setting.toUpperCase()} ${settings[setting] ? 'ON' : 'OFF'}`, 'info');
}

function setGraphicsQuality(quality){
    settings.graphics = quality;
    localStorage.setItem('graphicsQuality', quality);
    showNotification(`Graphics set to ${quality.toUpperCase()}`, 'info');
    window.graphicsQuality = quality;
}

function openSettings(){
    document.getElementById('start-screen').style.display='none';
    document.getElementById('settings-screen').style.display='flex';
    initSettingsUI();
}
function closeSettings(){
    document.getElementById('settings-screen').style.display='none';
    document.getElementById('start-screen').style.display='flex';
}

// Override showNotification based on settings
const originalShowNotification = window.showNotification || function(){};
window.showNotification = function(msg, type){
    if(settings.notifications) originalShowNotification(msg, type);
};

// ============================================
// CRITICAL HITS SYSTEM
// ============================================
let criticalHitsCount = 0;
let critChance = 0.15;
let totalDamageDealt = 0;

function isCriticalHit(){
    return Math.random() < critChance;
}

function applyCriticalHit(power){
    if(isCriticalHit()){
        criticalHitsCount++;
        totalDamageDealt += power * 2;
        if(player && player.x && player.y) floats.push({txt:'⚡ CRITICAL!', x:player.x-40, y:player.y-50, l:1, c:'#ffaa00', size:20});
        return power * 2;
    }
    totalDamageDealt += power;
    return power;
}

// ============================================
// COMBO METER
// ============================================
function updateComboMeter(){
    const meter = document.getElementById('combo-meter');
    const fill = document.getElementById('combo-meter-fill');
    if(!meter) return;
    if(combo > 1){
        meter.style.display = 'block';
        let percent = Math.min(100, (combo / 50) * 100);
        fill.style.width = percent + '%';
        if(combo >= 10) fill.style.background = '#ff6600';
        else if(combo >= 5) fill.style.background = '#ffaa00';
        else fill.style.background = '#ffcc00';
    } else {
        meter.style.display = 'none';
    }
}

// ============================================
// NOTIFICATION SYSTEM
// ============================================
function showNotification(message, type = 'info'){
    const area = document.getElementById('notification-area');
    if(!area) return;
    const notif = document.createElement('div');
    notif.className = `notification notification-${type}`;
    notif.innerHTML = message;
    area.appendChild(notif);
    setTimeout(() => {
        if(notif && notif.remove) notif.remove();
    }, 5000);
}

function showCustomAlert(message){
    const alertDiv = document.getElementById('custom-alert');
    document.getElementById('alert-message').innerText = message;
    alertDiv.style.display = 'flex';
}
function closeCustomAlert(){
    document.getElementById('custom-alert').style.display = 'none';
}

// AUTO-SAVE
function autoSave(){
    localStorage.setItem('totalCoins', totalCoins);
    localStorage.setItem('hiScore', hiScore);
    localStorage.setItem('totalKills', totalKills);
    localStorage.setItem('fireLevel', fireLevel);
    localStorage.setItem('dmgLevel', damageLevel);
    localStorage.setItem('droneCount', droneCount);
    localStorage.setItem('bombCount', bombCount);
    localStorage.setItem('gemstones', gemstones);
    localStorage.setItem('ownedSkins', JSON.stringify(ownedSkins));
    localStorage.setItem('achievements', JSON.stringify(achievements));
}
setInterval(autoSave, 30000);

// DAILY REWARD
let lastDailyClaim = localStorage.getItem('lastDailyClaim') || 0;
let dailyClaimCount = parseInt(localStorage.getItem('dailyClaimCount')) || 0;
function claimDailyReward(){
    const now = Date.now();
    const last = parseInt(lastDailyClaim);
    const hoursSince = (now - last) / (1000 * 60 * 60);
    if(last === 0 || hoursSince >= 24){
        const reward = 100 + Math.floor(Math.random() * 400);
        gemstones += reward;
        saveGemstones();
        lastDailyClaim = now;
        dailyClaimCount++;
        localStorage.setItem('lastDailyClaim', now);
        localStorage.setItem('dailyClaimCount', dailyClaimCount);
        showCustomAlert(`🎁 זכית ב-${reward} GEMSTONES! 🎁\nבוא שוב מחר לפרס נוסף! (${dailyClaimCount} total claims)`);
        showNotification(`🎁 Daily Reward: +${reward} GEMSTONES!`, 'success');
        updateHubUI();
        checkAchievements();
    } else {
        const remaining = Math.ceil(24 - hoursSince);
        showCustomAlert(`⏳ הפרס היומי כבר נאסף! תוכל לקחת שוב בעוד ${remaining} שעות.`);
        showNotification(`⏳ Daily reward available in ${remaining} hours`, 'warning');
    }
}

// DAILY MISSIONS
let dailyMissions = JSON.parse(localStorage.getItem('dailyMissions')) || [];
let lastMissionReset = localStorage.getItem('lastMissionReset') || 0;

function resetDailyMissions(){
    const now = Date.now();
    const last = parseInt(lastMissionReset);
    const hoursSince = (now - last) / (1000 * 60 * 60);
    if(last === 0 || hoursSince >= 24){
        dailyMissions = [
            { id: 0, name: "KILL ENEMIES", desc: "הרוג 100 אויבים", current: 0, target: 100, reward: 200, completed: false, type: "kill" },
            { id: 1, name: "COLLECT GEMS", desc: "אסוף 500 GEMSTONES", current: 0, target: 500, reward: 300, completed: false, type: "gem" },
            { id: 2, name: "PERFECT WAVES", desc: "השלם 5 גלים בלי נזק", current: 0, target: 5, reward: 250, completed: false, type: "perfect" },
            { id: 3, name: "CRITICAL HITS", desc: "בצע 50 פגיעות קריטיות", current: 0, target: 50, reward: 350, completed: false, type: "crit" }
        ];
        lastMissionReset = now;
        localStorage.setItem('lastMissionReset', now);
        localStorage.setItem('dailyMissions', JSON.stringify(dailyMissions));
    }
}

function updateDailyMissionProgress(type, amount = 1){
    for(let m of dailyMissions){
        if(!m.completed && m.type === type){
            m.current += amount;
            if(m.current >= m.target){
                m.completed = true;
                gemstones += m.reward;
                saveGemstones();
                showNotification(`✅ Mission Complete: ${m.name}! +${m.reward} GEMSTONES!`, 'success');
            }
        }
    }
    localStorage.setItem('dailyMissions', JSON.stringify(dailyMissions));
    updateDailyMissionsUI();
}

function updateDailyMissionsUI(){
    const container = document.getElementById('daily-missions-container');
    if(!container) return;
    container.innerHTML = '';
    for(const m of dailyMissions){
        const percent = (m.current / m.target) * 100;
        const div = document.createElement('div');
        div.className = 'daily-mission-card';
        div.innerHTML = `
            <div style="display:flex;justify-content:space-between;">
                <span style="color:#ffaa00;">${m.name}</span>
                <span>${m.completed ? '✅' : '⏳'}</span>
            </div>
            <div style="font-size:10px;">${m.desc}</div>
            <div class="ach-progress-bar" style="margin-top:5px;"><div class="ach-progress-fill" style="width:${percent}%;background:#ffaa00;"></div></div>
            <div style="font-size:9px;">${m.current}/${m.target} 🎁 ${m.reward}💎</div>
        `;
        container.appendChild(div);
    }
}

// RANK SYSTEM
let currentRank = parseInt(localStorage.getItem('currentRank')) || 1;
let rankXP = parseInt(localStorage.getItem('rankXP')) || 0;
const RANK_REQUIREMENTS = [0, 100, 250, 500, 1000, 2000, 3500, 5500, 8000, 11000, 15000, 20000, 26000, 33000, 41000, 50000];
const RANK_BONUS = [0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 25, 28, 32, 36, 40, 44, 48, 52, 56, 60, 65, 70, 75, 80, 85, 90, 95, 100, 105, 110, 115, 120, 125, 130, 135, 140, 145, 150, 160, 170, 180, 190, 200, 210, 220, 230, 240, 250];

function addRankXP(amount){
    rankXP += amount;
    let req = RANK_REQUIREMENTS[currentRank] || (currentRank * 500);
    while(rankXP >= req && currentRank < 50){
        rankXP -= req;
        currentRank++;
        req = RANK_REQUIREMENTS[currentRank] || (currentRank * 500);
        showNotification(`🏆 RANK UP! Reached Rank ${currentRank}! +${RANK_BONUS[currentRank]}% bonus!`, 'success');
    }
    localStorage.setItem('currentRank', currentRank);
    localStorage.setItem('rankXP', rankXP);
    updateRankUI();
}

function updateRankUI(){
    const badge = document.getElementById('rank-badge');
    if(badge) badge.innerHTML = `🏆 RANK ${currentRank}`;
    const rankBonus = RANK_BONUS[currentRank] || 0;
    skinCreditMultiplier = 1 + (rankBonus / 100);
    skinDamageMultiplier = 1 + (rankBonus / 100);
    skinFireRateMultiplier = 1 + (rankBonus / 100);
}

// SPECIAL ITEMS
let stardustPurchased = localStorage.getItem('stardustPurchased') === 'true';
let guardianAngelCount = parseInt(localStorage.getItem('guardianAngelCount')) || 3;
let powerCoreCount = parseInt(localStorage.getItem('powerCoreCount')) || 0;
let cosmicEssenceCount = parseInt(localStorage.getItem('cosmicEssenceCount')) || 0;
let ironWillPurchased = localStorage.getItem('ironWillPurchased') === 'true';
let timeDistortionPurchased = localStorage.getItem('timeDistortionPurchased') === 'true';
let crystalHeartPurchased = localStorage.getItem('crystalHeartPurchased') === 'true';
let guardianAngelUsed = false;
let damageReduction = ironWillPurchased ? 0.5 : 1;
let enemySlow = timeDistortionPurchased ? 0.7 : 1;
let gemMultiplier = crystalHeartPurchased ? 2 : 1;

function buySpecialItem(item){
    if(item === 'stardust'){
        if(stardustPurchased){
            showCustomAlert('⭐ כבר רכשת את STARDUST! ניתן לקנות רק פעם אחת.');
            return;
        }
        if(totalCoins >= 5000){
            totalCoins -= 5000;
            stardustPurchased = true;
            localStorage.setItem('stardustPurchased', 'true');
            damageLevel = Math.floor(damageLevel * 1.1);
            fireLevel = Math.floor(fireLevel * 1.1);
            localStorage.setItem('dmgLevel', damageLevel);
            localStorage.setItem('fireLevel', fireLevel);
            showCustomAlert(`⭐ STARDUST נרכש! כל הסטטים הוגדלו ב-10%! ⭐\nדמג: ${damageLevel} | Fire Rate: ${fireLevel}`);
            showNotification(`⭐ STARDUST acquired! +10% to all stats!`, 'success');
            updateShopUI();
            checkAchievements();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 5000c, יש לך ${totalCoins}c`);
        }
    } else if(item === 'guardian'){
        if(guardianAngelCount <= 0){
            showCustomAlert('🛡️ אזלו לך מלאכי המוות! אין יותר.');
            return;
        }
        if(totalCoins >= 3000){
            totalCoins -= 3000;
            guardianAngelCount--;
            localStorage.setItem('guardianAngelCount', guardianAngelCount);
            document.getElementById('guardian-count').innerText = guardianAngelCount;
            showCustomAlert(`🛡️ GUARDIAN ANGEL נרכש! (נותרו ${guardianAngelCount}/3)\nמעניק חסינות למוות פעם אחת!`);
            showNotification(`🛡️ Guardian Angel purchased! (${guardianAngelCount}/3 remaining)`, 'success');
            updateShopUI();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 3000c, יש לך ${totalCoins}c`);
        }
    } else if(item === 'powercore'){
        if(powerCoreCount >= 20){
            showCustomAlert('⚡ הגעת למקסימום של POWER CORE (20)!');
            return;
        }
        if(totalCoins >= 2000){
            totalCoins -= 2000;
            powerCoreCount++;
            damageLevel += Math.floor(damageLevel * 0.05);
            localStorage.setItem('powerCoreCount', powerCoreCount);
            localStorage.setItem('dmgLevel', damageLevel);
            showCustomAlert(`⚡ POWER CORE +5% DAMAGE! (${powerCoreCount}/20)\nדמג כעת: ${damageLevel}`);
            showNotification(`⚡ Power Core +5% damage! (${powerCoreCount}/20)`, 'success');
            updateShopUI();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 2000c, יש לך ${totalCoins}c`);
        }
    } else if(item === 'cosmic'){
        if(cosmicEssenceCount >= 15){
            showCustomAlert('🌟 הגעת למקסימום של COSMIC ESSENCE (15)!');
            return;
        }
        if(totalCoins >= 1500){
            totalCoins -= 1500;
            cosmicEssenceCount++;
            fireLevel += Math.floor(fireLevel * 0.03);
            localStorage.setItem('cosmicEssenceCount', cosmicEssenceCount);
            localStorage.setItem('fireLevel', fireLevel);
            showCustomAlert(`🌟 COSMIC ESSENCE +3% FIRE RATE! (${cosmicEssenceCount}/15)\nקצב ירי כעת: ${fireLevel}`);
            showNotification(`🌟 Cosmic Essence +3% fire rate! (${cosmicEssenceCount}/15)`, 'success');
            updateShopUI();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 1500c, יש לך ${totalCoins}c`);
        }
    } else if(item === 'ironwill'){
        if(ironWillPurchased){
            showCustomAlert('🛡️ כבר רכשת את IRON WILL! ניתן לקנות רק פעם אחת.');
            return;
        }
        if(totalCoins >= 5000){
            totalCoins -= 5000;
            ironWillPurchased = true;
            localStorage.setItem('ironWillPurchased', 'true');
            damageReduction = 0.5;
            showCustomAlert(`🛡️ IRON WILL נרכש! כל הנזק שהתקבל יחצה! 🛡️`);
            showNotification(`🛡️ Iron Will acquired! 50% damage reduction!`, 'success');
            updateShopUI();
            checkAchievements();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 5000c, יש לך ${totalCoins}c`);
        }
    } else if(item === 'timedistortion'){
        if(timeDistortionPurchased){
            showCustomAlert('⏰ כבר רכשת את TIME DISTORTION! ניתן לקנות רק פעם אחת.');
            return;
        }
        if(totalCoins >= 8000){
            totalCoins -= 8000;
            timeDistortionPurchased = true;
            localStorage.setItem('timeDistortionPurchased', 'true');
            enemySlow = 0.7;
            showCustomAlert(`⏰ TIME DISTORTION נרכש! כל האויבים הואטו ב-30%! ⏰`);
            showNotification(`⏰ Time Distortion acquired! Enemies slowed by 30%!`, 'success');
            updateShopUI();
            checkAchievements();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 8000c, יש לך ${totalCoins}c`);
        }
    } else if(item === 'crystalheart'){
        if(crystalHeartPurchased){
            showCustomAlert('💎 כבר רכשת את CRYSTAL HEART! ניתן לקנות רק פעם אחת.');
            return;
        }
        if(totalCoins >= 10000){
            totalCoins -= 10000;
            crystalHeartPurchased = true;
            localStorage.setItem('crystalHeartPurchased', 'true');
            gemMultiplier = 2;
            showCustomAlert(`💎 CRYSTAL HEART נרכש! כל ה-GEMSTONES שיתקבלו יוכפלו ב-2! 💎`);
            showNotification(`💎 Crystal Heart acquired! 2x GEMSTONES forever!`, 'success');
            updateShopUI();
            checkAchievements();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 10000c, יש לך ${totalCoins}c`);
        }
    }
}

// GAME VARIABLES
let currentGame = null;
let gemstones = parseInt(localStorage.getItem('gemstones')) || 0;
let lootboxesOpened = parseInt(localStorage.getItem('lootboxesOpened')) || 0;
let currentSkin = localStorage.getItem('currentSkin') || 'default';

// SKIN DEFINITIONS
const SKINS = [
    { id: 'default', name: 'DEFAULT', icon: '🚀', desc: 'החללית הבסיסית. פשוטה אבל אמינה!', effect: null, requirement: 0, owned: true },
    { id: 'blue', name: 'BLUE COMMON', icon: '🔵', desc: 'סקין כחול. הוא... כחול. זהו.', effect: null, requirement: 'lootbox', owned: false },
    { id: 'purple', name: 'PURPLE RARE', icon: '🟣', desc: 'סגלגל וחמוד. אויבים מפחדים מסגול?', effect: null, requirement: 'lootbox', owned: false },
    { id: 'gold', name: 'GOLDEN LEGEND', icon: '👑', desc: 'סקין זהב אגדי! מראה את השליטה שלך!', effect: '+25% CREDITS, +20% DAMAGE', requirement: 40, owned: false },
    { id: 'rainbow', name: 'RAINBOW MYTHIC', icon: '🌈', desc: 'צבעי הקשת! מסנוור את האויבים ביופי!', effect: null, requirement: 'lootbox', owned: false },
    { id: 'ultra', name: 'ULTRA MYTHIC', icon: '💎', desc: 'הסקין הנדיר ביותר ביקום! רק לעילא ולעלא!', effect: '+50% CREDITS, +50% DAMAGE, +20% FIRE RATE', requirement: 'ultra_lootbox', owned: false, limited: true },
    { id: 'legend', name: 'LEGEND RANK 50', icon: '🏆', desc: 'סקין אגדי! מושג רק על ידי הטובים ביותר!', effect: '+100% CREDITS, +100% DAMAGE, +50% FIRE RATE', requirement: 'rank50', owned: false }
];

// EVENT COUNTERS
let reaperCalls = 0, timeWarps = 0, goldRushes = 0, divineInterventions = 0, bugEvents = 0;
let stableCycleCount = 0, tidalWaveCount = 0, masqueradeCount = 0, soulHarvestCount = 0;
let frozenTimeCount = 0, crystalRainCount = 0, shadowCloneCount = 0;
let lightningStormCount = 0, luckyDrawCount = 0, mysteryBoxCount = 0, doomsDayCount = 0, royalBlessingCount = 0;
let starfallCount = 0, infernoCount = 0, chainLightningCount = 0, barrierCount = 0, soulReaperCount = 0;
let gamblerCount = 0, vortexCount = 0, kingsBlessingCount = 0, prismCount = 0, abyssCount = 0;
let doppelgangerCount = 0, supernovaCount = 0;

function showAchievementPopup(name, desc, gemsAmount = 0){
    const popup = document.getElementById('achievement-popup');
    const nameEl = document.getElementById('ach-name');
    const descEl = document.getElementById('ach-desc');
    nameEl.innerText = name;
    if(gemsAmount > 0){
        descEl.innerText = `${desc} +${gemsAmount} 💎`;
    } else {
        descEl.innerText = desc;
    }
    popup.style.display = 'block';
    setTimeout(() => {
        popup.style.display = 'none';
    }, 3000);
    showNotification(`🏆 ${name} unlocked!`, 'success');
}

function flashScreen(){
    if(!settings.shake) return;
    const flash = document.createElement('div');
    flash.style.position = 'fixed';
    flash.style.top = 0;
    flash.style.left = 0;
    flash.style.width = '100%';
    flash.style.height = '100%';
    flash.style.backgroundColor = 'rgba(255,255,255,0.5)';
    flash.style.zIndex = 9999;
    flash.style.pointerEvents = 'none';
    document.body.appendChild(flash);
    setTimeout(() => flash.remove(), 200);
}

// LOOTBOX REWARDS
const LOOTBOX_REWARDS = {
    common: [
        {name:"50 GEMSTONES", type:"gem", amount:50, rarity:"common", icon:"💎"},
        {name:"100 GEMSTONES", type:"gem", amount:100, rarity:"common", icon:"💎"},
        {name:"500 CREDITS", type:"credit", amount:500, rarity:"common", icon:"💰"},
        {name:"TEMPORARY SPEED BOOST", type:"boost", effect:"speed", duration:60, rarity:"common", icon:"⚡"},
        {name:"1 BOMB", type:"bomb", amount:1, rarity:"common", icon:"💣"}
    ],
    rare: [
        {name:"150 GEMSTONES", type:"gem", amount:150, rarity:"rare", icon:"💎"},
        {name:"250 GEMSTONES", type:"gem", amount:250, rarity:"rare", icon:"💎"},
        {name:"1000 CREDITS", type:"credit", amount:1000, rarity:"rare", icon:"💰"},
        {name:"TEMPORARY DAMAGE BOOST", type:"boost", effect:"damage", duration:90, rarity:"rare", icon:"💪"},
        {name:"2 BOMBS", type:"bomb", amount:2, rarity:"rare", icon:"💣"},
        {name:"COMMON SKIN", type:"skin", skin:"blue", rarity:"rare", icon:"🎨"}
    ],
    epic: [
        {name:"300 GEMSTONES", type:"gem", amount:300, rarity:"epic", icon:"💎"},
        {name:"500 GEMSTONES", type:"gem", amount:500, rarity:"epic", icon:"💎"},
        {name:"2500 CREDITS", type:"credit", amount:2500, rarity:"epic", icon:"💰"},
        {name:"PERMANENT DAMAGE UPGRADE", type:"perm_upgrade", stat:"damage", amount:1, rarity:"epic", icon:"🔰"},
        {name:"3 BOMBS", type:"bomb", amount:3, rarity:"epic", icon:"💣"},
        {name:"RARE SKIN", type:"skin", skin:"purple", rarity:"epic", icon:"🎨"}
    ],
    legendary: [
        {name:"600 GEMSTONES", type:"gem", amount:600, rarity:"legendary", icon:"💎"},
        {name:"1000 GEMSTONES", type:"gem", amount:1000, rarity:"legendary", icon:"💎"},
        {name:"5000 CREDITS", type:"credit", amount:5000, rarity:"legendary", icon:"💰"},
        {name:"PERMANENT FIRE RATE UPGRADE", type:"perm_upgrade", stat:"fire", amount:2, rarity:"legendary", icon:"🔥"},
        {name:"5 BOMBS", type:"bomb", amount:5, rarity:"legendary", icon:"💣"},
        {name:"LEGENDARY SKIN", type:"skin", skin:"gold", rarity:"legendary", icon:"👑"}
    ],
    mythic: [
        {name:"1500 GEMSTONES", type:"gem", amount:1500, rarity:"mythic", icon:"💎"},
        {name:"2500 GEMSTONES", type:"gem", amount:2500, rarity:"mythic", icon:"💎"},
        {name:"10000 CREDITS", type:"credit", amount:10000, rarity:"mythic", icon:"💰"},
        {name:"PERMANENT DAMAGE UPGRADE x3", type:"perm_upgrade", stat:"damage", amount:3, rarity:"mythic", icon:"🔰🔰"},
        {name:"7 BOMBS", type:"bomb", amount:7, rarity:"mythic", icon:"💣"},
        {name:"MYTHIC SKIN", type:"skin", skin:"rainbow", rarity:"mythic", icon:"🌈"}
    ],
    ultra: [
        {name:"5000 GEMSTONES", type:"gem", amount:5000, rarity:"ultra", icon:"💎💎"},
        {name:"10000 GEMSTONES", type:"gem", amount:10000, rarity:"ultra", icon:"💎💎"},
        {name:"50000 CREDITS", type:"credit", amount:50000, rarity:"ultra", icon:"💰💰"},
        {name:"ULTRA MYTHIC SKIN (LIMITED)", type:"skin", skin:"ultra", rarity:"ultra", icon:"👑👑", limited:true},
        {name:"15 BOMBS", type:"bomb", amount:15, rarity:"ultra", icon:"💣💣"},
        {name:"ALL PERMANENT UPGRADES +5", type:"perm_upgrade_all", amount:5, rarity:"ultra", icon:"⭐"}
    ]
};

const INDIVIDUAL_REWARDS = [
    {name:"COMMON SKIN", type:"skin", skin:"blue", price:200, rarity:"rare", icon:"🎨"},
    {name:"RARE SKIN", type:"skin", skin:"purple", price:500, rarity:"epic", icon:"🎨"},
    {name:"LEGENDARY SKIN", type:"skin", skin:"gold", price:1500, rarity:"legendary", icon:"👑"},
    {name:"MYTHIC SKIN", type:"skin", skin:"rainbow", price:4000, rarity:"mythic", icon:"🌈"},
    {name:"PERMANENT DAMAGE +1", type:"perm_upgrade", stat:"damage", price:800, rarity:"epic", icon:"🔰"},
    {name:"PERMANENT FIRE RATE +2", type:"perm_upgrade", stat:"fire", price:1000, rarity:"legendary", icon:"🔥"},
    {name:"1000 GEMSTONES", type:"gem", amount:1000, price:1200, rarity:"legendary", icon:"💎"},
    {name:"5000 CREDITS", type:"credit", amount:5000, price:600, rarity:"epic", icon:"💰"}
];

let ownedSkins = JSON.parse(localStorage.getItem('ownedSkins')) || {blue:false, purple:false, gold:false, rainbow:false, ultra:false, legend:false};
let ultraMythicCount = parseInt(localStorage.getItem('ultraMythicCount')) || 0;
const ULTRA_MYTHIC_LIMIT = 5;

function saveGemstones(){
    localStorage.setItem('gemstones', gemstones);
    updateGemUI();
}

function updateGemUI(){
    const gemElements = ['gemstones-amount', 'hub-gems', 'rng-gems'];
    gemElements.forEach(id => {
        const el = document.getElementById(id);
        if(el) el.innerText = formatNumber(gemstones);
    });
}

function formatNumber(num){ return num.toLocaleString(); }

// SKINS SYSTEM
function updateSkinsUI(){
    const container = document.getElementById('skins-grid-container');
    if(!container) return;
    container.innerHTML = '';
    const unlockedCount = ACHIEVEMENTS_LIST.filter(a => achievements[a.id]===true).length;
    
    for(const skin of SKINS){
        let isOwned = false;
        if(skin.id === 'default') isOwned = true;
        else if(skin.id === 'blue') isOwned = ownedSkins.blue;
        else if(skin.id === 'purple') isOwned = ownedSkins.purple;
        else if(skin.id === 'gold') isOwned = unlockedCount >= skin.requirement;
        else if(skin.id === 'rainbow') isOwned = ownedSkins.rainbow;
        else if(skin.id === 'ultra') isOwned = ownedSkins.ultra;
        else if(skin.id === 'legend') isOwned = currentRank >= 50;
        
        const isEquipped = currentSkin === skin.id;
        const card = document.createElement('div');
        card.className = `skin-card ${isOwned ? 'owned' : 'locked'} ${isEquipped ? 'equipped' : ''}`;
        card.innerHTML = `
            <div class="skin-icon">${skin.icon}</div>
            <div class="skin-name">${skin.name}</div>
            <div class="skin-desc">${skin.desc}</div>
            ${skin.effect ? `<div class="skin-effect">✨ ${skin.effect}</div>` : '<div class="skin-effect">😴 ללא יכולת מיוחדת</div>'}
            ${!isOwned ? `<div style="font-size:9px;color:#ffaa00;margin-top:5px;">🔒 ${skin.requirement === 'lootbox' ? 'נפתח בתיבות' : skin.requirement === 'ultra_lootbox' ? 'נפתח בתיבות ULTRA MYTHIC' : skin.requirement === 'rank50' ? 'דורש RANK 50' : `דורש ${skin.requirement} הישגים`}</div>` : ''}
            ${isEquipped ? '<div style="font-size:9px;color:#ff66ff;margin-top:5px;">✅ מצויד כעת</div>' : ''}
        `;
        if(isOwned && !isEquipped){
            card.onclick = () => equipSkin(skin.id);
        } else if(isEquipped){
            card.onclick = null;
        } else {
            card.onclick = null;
        }
        container.appendChild(card);
    }
}

function equipSkin(skinId){
    currentSkin = skinId;
    localStorage.setItem('currentSkin', currentSkin);
    updateSkinsUI();
    showAchievementPopup('🎨 SKIN EQUIPPED', `${skinId.toUpperCase()} skin is now active!`);
    showNotification(`🎨 ${skinId.toUpperCase()} skin equipped!`, 'info');
}

function openSkins(){
    document.getElementById('start-screen').style.display='none';
    document.getElementById('skins-screen').style.display='flex';
    updateSkinsUI();
}
function closeSkins(){
    document.getElementById('skins-screen').style.display='none';
    document.getElementById('start-screen').style.display='flex';
}

let currentReward = null;
let currentLootboxType = null;

function openLootbox(type){
    const price = {common:50, rare:150, epic:400, legendary:1000, mythic:2500, ultra:10000}[type];
    if(gemstones < price){
        showCustomAlert(`לא מספיק GEMSTONES! צריך ${price} 💎`);
        showNotification(`Not enough GEMSTONES! Need ${price} 💎`, 'warning');
        return;
    }
    if(type === 'ultra' && ultraMythicCount >= ULTRA_MYTHIC_LIMIT){
        showCustomAlert(`❗ ULTRA MYTHIC תיבות מוגבלות ל-${ULTRA_MYTHIC_LIMIT} בלבד! כבר פתחת את כולן.`);
        showNotification(`Ultra Mythic boxes limit reached (${ULTRA_MYTHIC_LIMIT}/5)`, 'warning');
        return;
    }
    gemstones -= price;
    saveGemstones();
    lootboxesOpened++;
    localStorage.setItem('lootboxesOpened', lootboxesOpened);
    
    const rewards = LOOTBOX_REWARDS[type];
    const reward = rewards[Math.floor(Math.random() * rewards.length)];
    currentReward = reward;
    currentLootboxType = type;
    
    document.getElementById('lootbox-animation').style.display='flex';
    document.getElementById('lootbox-result').style.display='none';
    document.getElementById('lootbox-close-btn').style.display='none';
    const box = document.getElementById('lootbox-element');
    box.className = 'lootbox';
    box.style.animation = 'shake 0.5s infinite';
    box.innerHTML = '📦';
    
    setTimeout(() => {
        box.style.animation = 'openBox 0.5s forwards';
        flashScreen();
        setTimeout(() => {
            box.style.display = 'none';
            applyReward(reward);
            document.getElementById('lootbox-result').style.display = 'block';
            document.getElementById('lootbox-close-btn').style.display = 'inline-block';
            if(type === 'ultra') ultraMythicCount++;
            localStorage.setItem('ultraMythicCount', ultraMythicCount);
        }, 500);
    }, 1000);
}

function applyReward(reward){
    let resultText = '';
    let finalAmount = reward.amount || 0;
    if(reward.type === 'gem') finalAmount = Math.floor(finalAmount * gemMultiplier);
    
    switch(reward.type){
        case 'gem':
            gemstones += finalAmount;
            saveGemstones();
            resultText = `✨ זכית ב-${finalAmount} GEMSTONES! ✨`;
            showNotification(`🎁 Lootbox reward: ${finalAmount} GEMSTONES!`, 'success');
            break;
        case 'credit':
            totalCoins += reward.amount;
            localStorage.setItem('totalCoins', totalCoins);
            resultText = `💰 זכית ב-${formatNumber(reward.amount)} CREDITS! 💰`;
            showNotification(`💰 Lootbox reward: ${formatNumber(reward.amount)} CREDITS!`, 'success');
            break;
        case 'bomb':
            bombCount += reward.amount;
            localStorage.setItem('bombCount', bombCount);
            resultText = `💣 זכית ב-${reward.amount} BOMBS! 💣`;
            showNotification(`💣 Lootbox reward: ${reward.amount} BOMBS!`, 'success');
            break;
        case 'boost':
            activePowerUps[reward.effect + '_boost'] = {name:reward.name, endTime:Date.now() + reward.duration*1000};
            resultText = `⚡ ${reward.name} הופעל ל-${reward.duration} שניות! ⚡`;
            showNotification(`⚡ ${reward.name} activated for ${reward.duration}s!`, 'success');
            break;
        case 'perm_upgrade':
            if(reward.stat === 'damage'){
                damageLevel += reward.amount;
                localStorage.setItem('dmgLevel', damageLevel);
                resultText = `🔰 נזק קבוע +${reward.amount}! (עכשיו רמה ${damageLevel}) 🔰`;
                showNotification(`🔰 Permanent damage +${reward.amount}!`, 'success');
            } else if(reward.stat === 'fire'){
                fireLevel += reward.amount;
                localStorage.setItem('fireLevel', fireLevel);
                resultText = `🔥 קצב ירי קבוע +${reward.amount}! (עכשיו רמה ${fireLevel}) 🔥`;
                showNotification(`🔥 Permanent fire rate +${reward.amount}!`, 'success');
            }
            break;
        case 'perm_upgrade_all':
            damageLevel += reward.amount;
            fireLevel += reward.amount;
            localStorage.setItem('dmgLevel', damageLevel);
            localStorage.setItem('fireLevel', fireLevel);
            resultText = `⭐ כל השדרוגים הקבועים +${reward.amount}! ⭐`;
            showNotification(`⭐ All permanent upgrades +${reward.amount}!`, 'success');
            break;
        case 'skin':
            if(!ownedSkins[reward.skin]){
                ownedSkins[reward.skin] = true;
                localStorage.setItem('ownedSkins', JSON.stringify(ownedSkins));
                resultText = `🎨 ${reward.name.toUpperCase()} נפתח! 🎨`;
                showNotification(`🎨 ${reward.name.toUpperCase()} skin unlocked!`, 'success');
                if(reward.skin === 'ultra'){
                    resultText = `👑👑 ULTRA MYTHIC SKIN (LIMITED) נפתחה! רק ${ULTRA_MYTHIC_LIMIT} קיימות! 👑👑`;
                }
                updateSkinsUI();
            } else {
                let duplicateGems = {blue:50, purple:100, gold:200, rainbow:400, ultra:1000}[reward.skin] || 100;
                gemstones += duplicateGems;
                saveGemstones();
                resultText = `🎨 כבר יש לך את הסקין! קיבלת ${duplicateGems} GEMSTONES במקום. 🎨`;
                showNotification(`🎨 Duplicate skin! +${duplicateGems} GEMSTONES`, 'info');
            }
            break;
    }
    document.getElementById('lootbox-result').innerHTML = `<div class="${reward.rarity}" style="font-size:20px;margin-bottom:10px;">${reward.icon}</div><div>${resultText}</div><div style="font-size:14px;color:#ffaa00;margin-top:10px;">${reward.name} (${reward.rarity.toUpperCase()})</div>`;
    updateShopUI();
    updateIndividualRewardsUI();
    checkAchievements();
}

function closeLootboxAnimation(){
    document.getElementById('lootbox-animation').style.display='none';
    const box = document.getElementById('lootbox-element');
    box.style.display = 'flex';
    box.style.animation = '';
    box.innerHTML = '📦';
    document.getElementById('lootbox-result').style.display = 'none';
    document.getElementById('lootbox-close-btn').style.display = 'none';
    currentLootboxType = null;
    currentReward = null;
}

function openRNGShop(){
    document.getElementById('start-screen').style.display='none';
    document.getElementById('rng-shop-screen').style.display='flex';
    updateGemUI();
    updateIndividualRewardsUI();
}

function closeRNGShop(){
    document.getElementById('rng-shop-screen').style.display='none';
    document.getElementById('start-screen').style.display='flex';
}

function buyIndividualReward(reward){
    if(gemstones < reward.price){
        showCustomAlert(`לא מספיק GEMSTONES! צריך ${reward.price} 💎`);
        showNotification(`Not enough GEMSTONES! Need ${reward.price} 💎`, 'warning');
        return;
    }
    gemstones -= reward.price;
    saveGemstones();
    
    if(reward.type === 'skin'){
        if(!ownedSkins[reward.skin]){
            ownedSkins[reward.skin] = true;
            localStorage.setItem('ownedSkins', JSON.stringify(ownedSkins));
            showCustomAlert(`🎨 ${reward.name.toUpperCase()} נרכש! 🎨`);
            showNotification(`🎨 ${reward.name.toUpperCase()} purchased!`, 'success');
            updateSkinsUI();
        } else {
            let duplicateGems = {blue:25, purple:50, gold:100, rainbow:200}[reward.skin] || 50;
            gemstones += duplicateGems;
            saveGemstones();
            showCustomAlert(`🎨 כבר יש לך את הסקין! קיבלת ${duplicateGems} GEMSTONES במקום. 🎨`);
            showNotification(`Duplicate skin! +${duplicateGems} GEMSTONES`, 'info');
        }
    } else if(reward.type === 'perm_upgrade'){
        if(reward.stat === 'damage'){
            damageLevel += reward.amount;
            localStorage.setItem('dmgLevel', damageLevel);
            showCustomAlert(`🔰 נזק קבוע +${reward.amount}! (עכשיו רמה ${damageLevel}) 🔰`);
            showNotification(`🔰 Permanent damage +${reward.amount}!`, 'success');
        } else if(reward.stat === 'fire'){
            fireLevel += reward.amount;
            localStorage.setItem('fireLevel', fireLevel);
            showCustomAlert(`🔥 קצב ירי קבוע +${reward.amount}! (עכשיו רמה ${fireLevel}) 🔥`);
            showNotification(`🔥 Permanent fire rate +${reward.amount}!`, 'success');
        }
    } else if(reward.type === 'gem'){
        let finalAmount = reward.amount * gemMultiplier;
        gemstones += finalAmount;
        saveGemstones();
        showCustomAlert(`✨ קיבלת ${finalAmount} GEMSTONES! ✨`);
        showNotification(`✨ +${finalAmount} GEMSTONES!`, 'success');
    } else if(reward.type === 'credit'){
        totalCoins += reward.amount;
        localStorage.setItem('totalCoins', totalCoins);
        showCustomAlert(`💰 קיבלת ${formatNumber(reward.amount)} CREDITS! 💰`);
        showNotification(`💰 +${formatNumber(reward.amount)} CREDITS!`, 'success');
    }
    updateGemUI();
    updateShopUI();
    updateIndividualRewardsUI();
    checkAchievements();
}

function updateIndividualRewardsUI(){
    const container = document.getElementById('individual-rewards');
    if(!container) return;
    container.innerHTML = '';
    for(const reward of INDIVIDUAL_REWARDS){
        const card = document.createElement('div');
        card.className = `card rarity-${reward.rarity} tooltip`;
        card.innerHTML = `<div style="font-size:20px;">${reward.icon}</div>
                         <div>${reward.name}</div>
                         <div style="color:#ffaa00;">${reward.price} 💎</div>
                         <div style="font-size:9px;color:#aaa;">${reward.rarity.toUpperCase()}</div>
                         <div class="tooltip-text">${reward.type === 'skin' ? 'פותח סקין חדש!' : reward.type === 'perm_upgrade' ? 'שדרוג קבוע!' : reward.type === 'gem' ? 'מקבל GEMSTONES!' : 'מקבל CREDITS!'}</div>`;
        card.onclick = () => buyIndividualReward(reward);
        container.appendChild(card);
    }
}

// HUB FUNCTIONS
function openGameSelect(){
    document.getElementById('main-hub').style.display='none';
    document.getElementById('game-select-screen').style.display='flex';
}
function openUpdateLog(){
    document.getElementById('main-hub').style.display='none';
    document.getElementById('update-log-screen').style.display='flex';
}
function backToHub(){
    document.getElementById('main-hub').style.display='flex';
    document.getElementById('game-select-screen').style.display='none';
    document.getElementById('update-log-screen').style.display='none';
    document.getElementById('start-screen').style.display='none';
    document.getElementById('settings-screen').style.display='none';
    updateHubUI();
}
function updateHubUI(){
    document.getElementById('hub-hi').innerText = formatNumber(hiScore);
    document.getElementById('hub-coins').innerText = formatNumber(totalCoins);
    document.getElementById('hub-gems').innerText = formatNumber(gemstones);
    document.getElementById('hub-kills').innerText = formatNumber(totalKills);
    document.getElementById('hub-skin').innerText = skinUnlocked ? 'UNLOCKED (GOLDEN)' : 'LOCKED';
    const unlockedCount = ACHIEVEMENTS_LIST.filter(a => achievements[a.id]===true).length;
    const percent = Math.min(100, (unlockedCount/40)*100);
    document.getElementById('hub-skin-progress-fill').style.width = percent+'%';
    document.getElementById('hub-skin-percent').innerHTML = unlockedCount+'/40 ACHIEVEMENTS';
    document.getElementById('stat-crits').innerText = criticalHitsCount;
    document.getElementById('stat-crit-rate').innerText = Math.floor(critChance * 100);
    document.getElementById('stat-damage').innerText = formatNumber(totalDamageDealt);
    document.getElementById('stat-od').innerText = totalOverdriveUses;
    document.getElementById('stat-bosses').innerText = bossesKilled;
}
function selectGame(game){
    if(game === 'defender'){
        currentGame = 'defender';
        document.getElementById('game-select-screen').style.display='none';
        document.getElementById('start-screen').style.display='flex';
        updateMainMenuUI();
        updateGemUI();
    } else {
        showCustomAlert('משחק זה עדיין בפיתוח! יגיע בקרוב...');
    }
}
function backToGameSelect(){
    gameState='MENU';
    isPaused=false;
    if(player) player=null;
    ['game-over','pause-screen','ui-hud','score-hud','combo-small','od-btn','pause-btn','powerup-bar','wave-banner','boss-warning','event-banner','ascend-screen','start-screen','game-timer','rank-badge','combo-meter'].forEach(id=>{
        const el=document.getElementById(id);
        if(el) el.style.display='none';
    });
    document.getElementById('vignette').style.display='none';
    document.getElementById('crosshair').style.display='none';
    document.getElementById('game-select-screen').style.display='flex';
}
function updateMainMenuUI(){
    document.getElementById('menu-hi').innerText=formatNumber(hiScore);
    document.getElementById('menu-coins').innerText=formatNumber(totalCoins);
    document.getElementById('menu-kills').innerText=formatNumber(totalKills);
    updateSkinProgressUI();
}
function confirmReset(){
    if(confirm('⚠️ אזהרה! פעולה זו תאפס את כל השדרוגים בחנות:\n- Fire Rate, Damage, Shield, Drones, Spread, Laser, Bombs\n\nכל הכסף שהושקע יוחזר לך!\n\nהישגים, שיאים וסקין מוזהב יישארו.\n\nהאם אתה בטוח?')){
        resetUpgradesOnly();
    }
}
function resetUpgradesOnly(){
    let refund = 0;
    refund += fireLevel * 150;
    refund += (damageLevel - 1) * 200;
    refund += droneCount * 500;
    refund += bombCount * 300;
    if(hasShieldUpgrade) refund += 250;
    if(hasSpreadShot) refund += 350;
    if(hasLaser) refund += 600;
    totalCoins += refund;
    fireLevel = 0;
    damageLevel = 1;
    droneCount = 0;
    bombCount = 0;
    hasShieldUpgrade = false;
    hasSpreadShot = false;
    hasLaser = false;
    localStorage.setItem('totalCoins', totalCoins);
    localStorage.setItem('fireLevel', fireLevel);
    localStorage.setItem('dmgLevel', damageLevel);
    localStorage.setItem('droneCount', droneCount);
    localStorage.setItem('bombCount', bombCount);
    localStorage.setItem('hasShieldUpgrade', hasShieldUpgrade);
    localStorage.setItem('hasSpreadShot', hasSpreadShot);
    localStorage.setItem('hasLaser', hasLaser);
    updateMainMenuUI();
    updateShopUI();
    updateSkinProgressUI();
    updateHubUI();
    showCustomAlert(`✅ כל השדרוגים אופסו!\n💰 קיבלת חזרה ${formatNumber(refund)} קרדיטים!\nסך קרדיטים נוכחי: ${formatNumber(totalCoins)}`);
    showNotification(`🔄 Upgrades reset! Refunded ${formatNumber(refund)} credits`, 'info');
}

// GAME STATE VARIABLES
const canvas=document.getElementById('gameCanvas'),ctx=canvas.getContext('2d');
const nebulaCanvas=document.getElementById('nebulaCanvas'),nCtx=nebulaCanvas.getContext('2d');
let width,height,mouseX=0,mouseY=0,kills=0;
let totalCoins=parseInt(localStorage.getItem('totalCoins'))||0;
let hiScore=parseInt(localStorage.getItem('hiScore'))||0;
let totalKills=parseInt(localStorage.getItem('totalKills'))||0;
let fireLevel=parseInt(localStorage.getItem('fireLevel'))||0;
let damageLevel=parseInt(localStorage.getItem('dmgLevel'))||1;
let hasShieldUpgrade=localStorage.getItem('hasShieldUpgrade')==='true';
let hasSpreadShot=localStorage.getItem('hasSpreadShot')==='true';
let hasLaser=localStorage.getItem('hasLaser')==='true';
let droneCount=Math.min(400, parseInt(localStorage.getItem('droneCount'))||0);
let bombCount=parseInt(localStorage.getItem('bombCount'))||0;
let achievements=JSON.parse(localStorage.getItem('achievements')||'{}');
let skinUnlocked=localStorage.getItem('skinUnlocked')==='true';
let totalUpgradesBought=parseInt(localStorage.getItem('totalUpgradesBought'))||0;
let guardianDefeated=localStorage.getItem('guardianDefeated')==='true';
let endlessMode=false;
let gameState='LOADING';
let score=0,health=100,maxHealth=100,xp=0,level=1;
let odCharge=0,isOD=false,odTimer=0;
let lastFire=0,shake=0,combo=1,comboTimer=0;
let bossWarningTimer=0,waveBannerTimer=0;
let wave=1,waveKillGoal=12,waveKills=0,waveTriggered=false;
let player=null;
let stars=[],bullets=[],enemies=[],eBullets=[],particles=[],items=[],floats=[];
let boss=null,isPaused=false;
let activePowerUps={};
let odActivations=0,waveNoDamage=true,bossesKilled=0;
let perfectWavesCount=0;
let startTime=0;
let totalOverdriveUses=0;
let totalBombsUsed=0;
let maxCombo=0;
let totalSynapseActivations=0;
let synapseKills=0;
let psychedeliaCollected=0;
let totalEventsTriggered=0;
let totalRiftsTriggered=0;
let totalMeteorsDestroyed=0;
let apocalypseTriggers=0;
let voidTriggers=0;
let blackHolePieces=0;
let cosmicCollapseCount=0;
let primordialRageCount=0;
let deathTouchCount=0;
let chaosRealmCount=0;
let eventCooldown=0;
let ascendTriggered=false;
let timerInterval = null;
let animationId = null;

// SKIN EFFECTS
let skinCreditMultiplier = 1;
let skinDamageMultiplier = 1;
let skinFireRateMultiplier = 1;

function updateSkinEffects(){
    skinCreditMultiplier = 1;
    skinDamageMultiplier = 1;
    skinFireRateMultiplier = 1;
    const rankBonus = RANK_BONUS[currentRank] || 0;
    skinCreditMultiplier = 1 + (rankBonus / 100);
    skinDamageMultiplier = 1 + (rankBonus / 100);
    skinFireRateMultiplier = 1 + (rankBonus / 100);
    if(currentSkin === 'gold'){
        skinCreditMultiplier *= 1.25;
        skinDamageMultiplier *= 1.2;
    } else if(currentSkin === 'ultra'){
        skinCreditMultiplier *= 1.5;
        skinDamageMultiplier *= 1.5;
        skinFireRateMultiplier *= 1.2;
    } else if(currentSkin === 'legend'){
        skinCreditMultiplier *= 2;
        skinDamageMultiplier *= 2;
        skinFireRateMultiplier *= 1.5;
    }
}

// MODES & EVENTS
let synapseActive=false;
let synapseTimer=0;
let synapsePsychedeliaCount=0;
let synapseBackgroundHue=0;
let activeEvent=null;
let eventTimer=0;
let meteors=[];
let riftActive=false;
let riftMultiplier=1;
let apocalypseActive=false;
let voidActive=false;
let voidTimer=0;
let guardian=null;
let cosmicCollapseActive=false;
let primordialRageActive=false;
let blackHoleActive=false;
let blackHoleCenter={x:0,y:0};
let chaosRealmActive=false;
let chaosHue=0;
let selectedQuantities={fire:1,dmg:1,drone:1,bomb:1};

// EVENT VARIABLES
let timeWarpActive=false;
let timeWarpTimer=0;
let goldRushActive=false;
let goldRushTimer=0;
let divineActive=false;
let divineTimer=0;
let bugEventActive=false;
let bugEventTimer=0;
let bugEventGlitch=false;
let stableCycleActive=false;
let stableCycleTimer=0;
let tidalWaveActive=false;
let tidalWaveTimer=0;
let masqueradeActive=false;
let masqueradeTimer=0;
let soulHarvestActive=false;
let soulHarvestTimer=0;
let soulHarvestSouls=[];
let frozenTimeActive=false;
let frozenTimeTimer=0;
let crystalRainActive=false;
let crystalRainTimer=0;
let shadowCloneActive=false;
let shadowCloneTimer=0;
let shadowClone = null;
let lightningStormActive=false;
let lightningStormTimer=0;
let lightningBolts=[];
let luckyDrawActive=false;
let mysteryBoxActive=false;
let doomsDayActive=false;
let royalBlessingActive=false;
let royalBlessingTimer=0;
let starfallActive=false;
let starfallTimer=0;
let starfallStars=[];
let infernoActive=false;
let infernoTimer=0;
let chainLightningActive=false;
let chainLightningTimer=0;
let barrierActive=false;
let barrierTimer=0;
let barrierHp=0;
let soulReaperActive=false;
let soulReaperTimer=0;
let soulReaperSoul=null;
let gamblerActive=false;
let vortexActive=false;
let vortexTimer=0;
let vortexCenter={x:0,y:0};
let kingsBlessingActive=false;
let kingsBlessingTimer=0;
let prismActive=false;
let prismTimer=0;
let abyssActive=false;
let abyssTimer=0;
let abyssCenter={x:0,y:0};
let doppelgangerActive=false;
let doppelgangerTimer=0;
let doppelgangerClone = null;
let supernovaActive=false;
let supernovaTimer=0;
let crystals = [];

// METEOR CLASS
class Meteor {
    constructor(x, y) {
        this.x = x; this.y = y; this.radius = 14; this.hp = 10; this.maxHp = 10;
        this.vx = (Math.random() - 0.5) * 2; this.vy = 3 + Math.random() * 2;
    }
    update() { this.x += this.vx; this.y += this.vy; }
    draw() {
        ctx.fillStyle = '#aa6644'; ctx.shadowBlur = 10;
        ctx.beginPath(); ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2); ctx.fill();
        ctx.fillStyle = '#ff8844'; ctx.fillRect(this.x - 4, this.y - 8, 8, 5);
        ctx.fillStyle = '#222'; ctx.fillRect(this.x - 10, this.y - 3, 20, 4);
        ctx.fillStyle = `hsl(${(this.hp / this.maxHp) * 30}, 80%, 50%)`;
        ctx.fillRect(this.x - 10, this.y - 3, (this.hp / this.maxHp) * 20, 4);
    }
}

// GUARDIAN CLASS
class Guardian {
    constructor() {
        this.x = width/2; this.y = -200; this.hp = 10000; this.maxHp = 10000;
        this.r = 120; this.speed = 0.3; this.lastShot = 0; this.angle = 0;
    }
    update() {
        this.y += this.speed; if(this.y > 150) this.y = 150;
        this.x = width/2 + Math.sin(Date.now()/600) * 80;
        this.angle += 0.02;
        if(Date.now() - this.lastShot > 400){
            this.lastShot = Date.now();
            for(let i=0;i<12;i++){
                let ang = i * Math.PI*2/12 + this.angle;
                eBullets.push({x:this.x + Math.cos(ang)*70, y:this.y + 50, vx:Math.cos(ang)*4, vy:Math.sin(ang)*4});
            }
        }
    }
    draw() {
        ctx.save(); ctx.translate(this.x, this.y);
        ctx.shadowBlur = 30; ctx.shadowColor = 'gold';
        ctx.fillStyle = '#ffd700'; ctx.beginPath(); ctx.arc(0,0,100,0,Math.PI*2); ctx.fill();
        ctx.fillStyle = '#ffaa00'; ctx.beginPath(); ctx.arc(0,0,80,0,Math.PI*2); ctx.fill();
        ctx.fillStyle = '#fff'; ctx.font = 'bold 40px Segoe UI'; ctx.fillText('👑', -25, 20);
        ctx.fillStyle = '#222'; ctx.fillRect(-100, -140, 200, 12);
        ctx.fillStyle = `hsl(${(this.hp/this.maxHp)*60},100%,50%)`;
        ctx.fillRect(-100, -140, (this.hp/this.maxHp)*200, 12);
        ctx.restore();
    }
}

// DOPPELGANGER CLONE CLASS
class DoppelgangerClone {
    constructor(){
        this.x = player.x - 60;
        this.y = player.y;
        this.r = 25;
        this.lastShot = 0;
    }
    update(){
        if(player){
            this.x = player.x - 60;
            this.y = player.y;
        }
        if(Date.now() - this.lastShot > 200){
            this.lastShot = Date.now();
            let power = damageLevel * 0.7;
            bullets.push(new Bullet(this.x, this.y-20, power, '#ff88ff'));
        }
    }
    draw(){
        ctx.save(); ctx.translate(this.x, this.y);
        ctx.fillStyle = '#ff88ff'; ctx.shadowBlur = 15;
        ctx.beginPath();
        ctx.moveTo(0,-25);ctx.lineTo(20,15);ctx.lineTo(8,15);ctx.lineTo(8,25);
        ctx.lineTo(-8,25);ctx.lineTo(-8,15);ctx.lineTo(-20,15);ctx.closePath();ctx.fill();
        ctx.fillStyle = 'rgba(255,136,255,0.5)';
        ctx.beginPath();ctx.ellipse(0,-5,4,10,0,0,Math.PI*2);ctx.fill();
        ctx.restore();
    }
}

// ACHIEVEMENTS LIST (abbreviated for space - same as before)
const ACHIEVEMENTS_LIST = [
    {id:'first_kill', name:'FIRST BLOOD', desc:'Destroy your first enemy', gems:5, difficulty:'easy'},
    {id:'combo10', name:'COMBO MASTER', desc:'Reach x10 combo', gems:10, difficulty:'easy'},
    {id:'wave5', name:'VETERAN', desc:'Survive to wave 5', gems:10, difficulty:'easy'},
    {id:'score5k', name:'RISING STAR', desc:'Score 5,000 points', gems:15, difficulty:'easy'},
    {id:'score25k', name:'GALACTIC HERO', desc:'Score 25,000 points', gems:25, difficulty:'medium'},
    {id:'overdrive3', name:'SPEED DEMON', desc:'Activate Overdrive 3 times', gems:15, difficulty:'easy'},
    {id:'nodmg_wave', name:'UNTOUCHABLE', desc:'Complete a wave without damage', gems:20, difficulty:'medium'},
    {id:'boss1', name:'BOSS SLAYER', desc:'Defeat your first boss', gems:20, difficulty:'easy'},
    {id:'rich', name:'RICH', desc:'Earn 10,000 total credits', gems:30, difficulty:'medium'},
    {id:'pyro', name:'PYROMANIAC', desc:'Fire Rate level 10', gems:25, difficulty:'medium'},
    {id:'warmonger', name:'WARMONGER', desc:'Damage level 10', gems:25, difficulty:'medium'},
    {id:'invincible', name:'INVINCIBLE', desc:'Buy Ion Shield', gems:20, difficulty:'easy'},
    {id:'drone_army', name:'DRONE ARMY', desc:'Own 5 drones', gems:25, difficulty:'medium'},
    {id:'demolition', name:'DEMOLITION', desc:'Use 10 bombs in one game', gems:30, difficulty:'medium'},
    {id:'overcharged', name:'OVERCHARGED', desc:'Activate Overdrive 10 times total', gems:35, difficulty:'medium'},
    {id:'kills100', name:'100 KILLS', desc:'100 total kills', gems:20, difficulty:'easy'},
    {id:'kills500', name:'500 KILLS', desc:'500 total kills', gems:40, difficulty:'medium'},
    {id:'legendary', name:'LEGENDARY', desc:'Reach Rank 15', gems:30, difficulty:'medium'},
    {id:'no_mercy', name:'NO MERCY', desc:'Kill 50 enemies in one wave', gems:35, difficulty:'hard'},
    {id:'laser_master', name:'LASER MASTER', desc:'Buy Laser Beam', gems:30, difficulty:'medium'},
    {id:'perfect_wave', name:'PERFECT WAVE', desc:'Complete 3 waves without damage', gems:40, difficulty:'hard'},
    {id:'speedrun', name:'SPEEDRUN', desc:'Complete 5 waves in 3 minutes', gems:35, difficulty:'hard'},
    {id:'millionaire', name:'MILLIONAIRE', desc:'Score 1,000,000 points', gems:50, difficulty:'hard'},
    {id:'boss_genocide', name:'BOSS GENOCIDE', desc:'Kill 10 bosses', gems:45, difficulty:'hard'},
    {id:'true_god', name:'TRUE GOD', desc:'Reach Rank 30', gems:50, difficulty:'hard'},
    {id:'combo_god', name:'COMBO GOD', desc:'Reach x30 combo', gems:40, difficulty:'hard'},
    {id:'max_out', name:'MAX OUT', desc:'Upgrade any stat to level 50', gems:60, difficulty:'extreme'},
    {id:'synapse_activate', name:'SYNAPSE ACTIVATED', desc:'Activate Synapse Mode 3 times', gems:45, difficulty:'hard'},
    {id:'time_lord', name:'TIME LORD', desc:'Kill 50 enemies during Synapse Mode', gems:50, difficulty:'hard'},
    {id:'psychedelic', name:'PSYCHEDELIC', desc:'Collect 10 Psychedelias', gems:30, difficulty:'medium'},
    {id:'decimator', name:'DECIMATOR', desc:'Kill 200 enemies in one game', gems:40, difficulty:'hard'},
    {id:'immortal', name:'IMMORTAL', desc:'Complete a wave without losing HP', gems:35, difficulty:'medium'},
    {id:'survivor', name:'SURVIVOR', desc:'Reach wave 15', gems:25, difficulty:'medium'},
    {id:'god_of_war', name:'GOD OF WAR', desc:'Reach Rank 50', gems:75, difficulty:'extreme'},
    {id:'infinite_power', name:'INFINITE POWER', desc:'Charge Overdrive to 200%', gems:50, difficulty:'hard'},
    {id:'shopaholic', name:'SHOPAHOLIC', desc:'Buy 50 upgrades total', gems:40, difficulty:'hard'},
    {id:'credit_farm', name:'CREDIT FARM', desc:'Collect 50,000 total credits', gems:45, difficulty:'hard'},
    {id:'meteor_slayer', name:'METEOR SLAYER', desc:'Destroy 50 meteors', gems:35, difficulty:'medium'},
    {id:'rift_walker', name:'RIFT WALKER', desc:'Trigger Dimension Rift 5 times', gems:40, difficulty:'hard'},
    {id:'event_master', name:'EVENT MASTER', desc:'Experience 10 total events', gems:30, difficulty:'medium'},
    {id:'godlike', name:'GODLIKE', desc:'Reach wave 50', gems:60, difficulty:'hard'},
    {id:'unstoppable', name:'UNSTOPPABLE', desc:'Kill 1000 enemies in one game', gems:80, difficulty:'extreme'},
    {id:'perfect_run', name:'PERFECT RUN', desc:'Complete 10 consecutive waves without damage', gems:70, difficulty:'extreme'},
    {id:'ultimate_power', name:'ULTIMATE POWER', desc:'Reach Rank 100', gems:100, difficulty:'extreme'},
    {id:'zero_damage', name:'ZERO DAMAGE', desc:'Complete a full game without taking any damage', gems:150, difficulty:'mythic'},
    {id:'speed_god', name:'SPEED GOD', desc:'Complete 30 waves in under 10 minutes', gems:80, difficulty:'extreme'},
    {id:'billionaire', name:'BILLIONAIRE', desc:'Score 10,000,000 points', gems:120, difficulty:'extreme'},
    {id:'true_survivor', name:'TRUE SURVIVOR', desc:'Play for 60 minutes straight', gems:100, difficulty:'extreme'},
    {id:'omega_boss', name:'OMEGA BOSS', desc:'Kill 50 total bosses', gems:90, difficulty:'extreme'},
    {id:'impossible', name:'IMPOSSIBLE', desc:'Reach wave 100', gems:150, difficulty:'mythic'},
    {id:'meteor_master', name:'METEOR MASTER', desc:'Destroy 200 meteors', gems:70, difficulty:'hard'},
    {id:'rift_god', name:'RIFT GOD', desc:'Trigger Dimension Rift 15 times', gems:80, difficulty:'extreme'},
    {id:'event_collector', name:'EVENT COLLECTOR', desc:'Experience 25 events', gems:60, difficulty:'hard'},
    {id:'fast_killer', name:'FAST KILLER', desc:'Kill 10 enemies within 3 seconds', gems:35, difficulty:'medium'},
    {id:'rich_lord', name:'RICH LORD', desc:'Collect 200,000 total credits', gems:70, difficulty:'hard'},
    {id:'endless', name:'ENDLESS', desc:'Reach wave 25', gems:30, difficulty:'medium'},
    {id:'combo_legend', name:'COMBO LEGEND', desc:'Reach x50 combo', gems:60, difficulty:'hard'},
    {id:'overdrive_god', name:'OVERDRIVE GOD', desc:'Activate Overdrive 50 times', gems:70, difficulty:'hard'},
    {id:'drone_master', name:'DRONE MASTER', desc:'Own 10 drones', gems:50, difficulty:'hard'},
    {id:'fire_god', name:'FIRE GOD', desc:'Fire Rate level 30', gems:60, difficulty:'hard'},
    {id:'damage_god', name:'DAMAGE GOD', desc:'Damage level 30', gems:60, difficulty:'hard'},
    {id:'drone_overlord', name:'DRONE OVERLORD', desc:'Own 400 drones (max)', gems:100, difficulty:'extreme'},
    {id:'apocalypse_survivor', name:'APOCALYPSE SURVIVOR', desc:'Survive Apocalypse Mode 3 times', gems:60, difficulty:'hard'},
    {id:'void_walker', name:'VOID WALKER', desc:'Trigger Void Mode 5 times', gems:70, difficulty:'hard'},
    {id:'guardian_slayer', name:'GUARDIAN SLAYER', desc:'Defeat The Guardian', gems:200, difficulty:'mythic'},
    {id:'true_guardian', name:'TRUE GUARDIAN', desc:'Defeat The Guardian 10 times', gems:500, difficulty:'mythic'},
    {id:'apocalypse_master', name:'APOCALYPSE MASTER', desc:'Kill 500 enemies during Apocalypse Mode', gems:80, difficulty:'extreme'},
    {id:'void_assassin', name:'VOID ASSASSIN', desc:'Kill 200 enemies during Void Mode', gems:80, difficulty:'extreme'},
    {id:'event_god', name:'EVENT GOD', desc:'Experience 50 events', gems:80, difficulty:'extreme'},
    {id:'credit_king', name:'CREDIT KING', desc:'Collect 1,000,000 total credits', gems:100, difficulty:'extreme'},
    {id:'score_king', name:'SCORE KING', desc:'Score 100,000,000 points', gems:150, difficulty:'mythic'},
    {id:'wave_warrior', name:'WAVE WARRIOR', desc:'Reach wave 200', gems:120, difficulty:'extreme'},
    {id:'immortal_god', name:'IMMORTAL GOD', desc:'Complete 50 waves without damage', gems:150, difficulty:'mythic'},
    {id:'perfect_game', name:'PERFECT GAME', desc:'Complete a full game without losing any HP', gems:200, difficulty:'mythic'},
    {id:'speed_demon', name:'SPEED DEMON', desc:'Complete 50 waves in under 15 minutes', gems:100, difficulty:'extreme'},
    {id:'overdrive_legend', name:'OVERDRIVE LEGEND', desc:'Activate Overdrive 200 times', gems:120, difficulty:'extreme'},
    {id:'bomb_master', name:'BOMB MASTER', desc:'Use 100 bombs total', gems:80, difficulty:'extreme'},
    {id:'cosmic_collapse', name:'COSMIC COLLAPSE', desc:'Experience Cosmic Collapse event', gems:50, difficulty:'hard'},
    {id:'primordial_rage', name:'PRIMORDIAL RAGE', desc:'Experience Primordial Rage event', gems:150, difficulty:'mythic'},
    {id:'black_hole', name:'BLACK HOLE', desc:'Activate Black Hole event', gems:60, difficulty:'hard'},
    {id:'cosmic_master', name:'COSMIC MASTER', desc:'Experience Cosmic Collapse 5 times', gems:80, difficulty:'extreme'},
    {id:'primordial_god', name:'PRIMORDIAL GOD', desc:'Experience Primordial Rage 3 times', gems:300, difficulty:'mythic'},
    {id:'black_hole_god', name:'BLACK HOLE GOD', desc:'Activate Black Hole 10 times', gems:120, difficulty:'extreme'},
    {id:'event_legend', name:'EVENT LEGEND', desc:'Experience 100 events', gems:120, difficulty:'extreme'},
    {id:'wave_500', name:'WAVE 500', desc:'Reach wave 500', gems:200, difficulty:'mythic'},
    {id:'rank_500', name:'RANK 500', desc:'Reach Rank 500', gems:200, difficulty:'mythic'},
    {id:'kills_10000', name:'10000 KILLS', desc:'Kill 10,000 enemies in one game', gems:250, difficulty:'mythic'},
    {id:'score_billion', name:'SCORE BILLION', desc:'Score 1,000,000,000 points', gems:300, difficulty:'mythic'},
    {id:'credit_billion', name:'CREDIT BILLION', desc:'Collect 100,000,000 total credits', gems:250, difficulty:'mythic'},
    {id:'perfect_100', name:'PERFECT 100', desc:'Complete 100 waves without damage', gems:250, difficulty:'mythic'},
    {id:'speed_100', name:'SPEED 100', desc:'Complete 100 waves in under 30 minutes', gems:200, difficulty:'mythic'},
    {id:'overdrive_1000', name:'OVERDRIVE 1000', desc:'Activate Overdrive 1000 times', gems:300, difficulty:'mythic'},
    {id:'bomb_1000', name:'BOMB 1000', desc:'Use 1000 bombs total', gems:200, difficulty:'mythic'},
    {id:'drone_400', name:'DRONE 400', desc:'Own 400 drones', gems:150, difficulty:'mythic'},
    {id:'fire_100', name:'FIRE 100', desc:'Fire Rate level 100', gems:200, difficulty:'mythic'},
    {id:'damage_100', name:'DAMAGE 100', desc:'Damage level 100', gems:200, difficulty:'mythic'},
    {id:'death_touch', name:'DEATH\'S TOUCH', desc:'Experience Death\'s Touch event', gems:250, difficulty:'mythic'},
    {id:'chaos_realm', name:'CHAOS REALM', desc:'Experience Chaos Realm event', gems:200, difficulty:'mythic'},
    {id:'death_master', name:'DEATH MASTER', desc:'Experience Death\'s Touch 3 times', gems:500, difficulty:'mythic'},
    {id:'chaos_master', name:'CHAOS MASTER', desc:'Experience Chaos Realm 3 times', gems:400, difficulty:'mythic'},
    {id:'reaper_call', name:'REAPER\'S CALL', desc:'Experience Reaper\'s Call event', gems:300, difficulty:'mythic'},
    {id:'time_warp', name:'TIME WARP', desc:'Experience Time Warp event', gems:250, difficulty:'mythic'},
    {id:'gold_rush', name:'GOLD RUSH', desc:'Experience Gold Rush event', gems:500, difficulty:'mythic'},
    {id:'divine_intervention', name:'DIVINE INTERVENTION', desc:'Experience Divine Intervention event', gems:1000, difficulty:'mythic'},
    {id:'bug_event', name:'BUG EVENT', desc:'Experience the mysterious Bug Event', gems:777, difficulty:'mythic'},
    {id:'stable_cycle', name:'STABLE CYCLE', desc:'Experience Stable Cycle event', gems:100, difficulty:'hard'},
    {id:'tidal_wave', name:'TIDAL WAVE', desc:'Experience Tidal Wave event', gems:150, difficulty:'mythic'},
    {id:'masquerade', name:'MASQUERADE', desc:'Experience Masquerade event', gems:200, difficulty:'mythic'},
    {id:'soul_harvest', name:'SOUL HARVEST', desc:'Experience Soul Harvest event', gems:250, difficulty:'mythic'},
    {id:'frozen_time', name:'FROZEN TIME', desc:'Experience Frozen Time event', gems:180, difficulty:'mythic'},
    {id:'crystal_rain', name:'CRYSTAL RAIN', desc:'Experience Crystal Rain event', gems:220, difficulty:'mythic'},
    {id:'shadow_clone', name:'SHADOW CLONE', desc:'Experience Shadow Clone event', gems:300, difficulty:'mythic'},
    {id:'lightning_storm', name:'LIGHTNING STORM', desc:'Experience Lightning Storm event', gems:200, difficulty:'mythic'},
    {id:'lucky_draw', name:'LUCKY DRAW', desc:'Experience Lucky Draw event', gems:250, difficulty:'mythic'},
    {id:'mystery_box', name:'MYSTERY BOX', desc:'Experience Mystery Box event', gems:280, difficulty:'mythic'},
    {id:'dooms_day', name:'DOOM\'S DAY', desc:'Experience Doom\'s Day event', gems:350, difficulty:'mythic'},
    {id:'royal_blessing', name:'ROYAL BLESSING', desc:'Experience Royal Blessing event', gems:400, difficulty:'mythic'},
    {id:'starfall', name:'STARFALL', desc:'Experience Starfall event', gems:180, difficulty:'mythic'},
    {id:'inferno', name:'INFERNO', desc:'Experience Inferno event', gems:200, difficulty:'mythic'},
    {id:'chain_lightning', name:'CHAIN LIGHTNING', desc:'Experience Chain Lightning event', gems:220, difficulty:'mythic'},
    {id:'barrier', name:'BARRIER', desc:'Experience Barrier event', gems:150, difficulty:'hard'},
    {id:'soul_reaper', name:'SOUL REAPER', desc:'Experience Soul Reaper event', gems:350, difficulty:'mythic'},
    {id:'gambler', name:'GAMBLER', desc:'Experience Gambler event', gems:400, difficulty:'mythic'},
    {id:'vortex', name:'VORTEX', desc:'Experience Vortex event', gems:280, difficulty:'mythic'},
    {id:'kings_blessing', name:'KING\'S BLESSING', desc:'Experience King\'s Blessing event', gems:450, difficulty:'mythic'},
    {id:'prism', name:'PRISM', desc:'Experience Prism event', gems:300, difficulty:'mythic'},
    {id:'abyss', name:'ABYSS', desc:'Experience Abyss event', gems:500, difficulty:'mythic'},
    {id:'doppelganger', name:'DOPPELGANGER', desc:'Experience Doppelganger event', gems:350, difficulty:'mythic'},
    {id:'supernova', name:'SUPERNOVA', desc:'Experience Supernova event', gems:600, difficulty:'mythic'},
    {id:'doppelganger_master', name:'DOPPELGANGER MASTER', desc:'Experience Doppelganger 5 times', gems:800, difficulty:'mythic'},
    {id:'supernova_master', name:'SUPERNOVA MASTER', desc:'Experience Supernova 5 times', gems:1200, difficulty:'mythic'},
    {id:'critical_hitter', name:'CRITICAL HITTER', desc:'Land 100 critical hits', gems:100, difficulty:'hard'},
    {id:'critical_master', name:'CRITICAL MASTER', desc:'Land 1000 critical hits', gems:300, difficulty:'extreme'},
    {id:'combo_master_50', name:'COMBO MASTER 50', desc:'Reach x50 combo', gems:150, difficulty:'hard'},
    {id:'combo_master_100', name:'COMBO MASTER 100', desc:'Reach x100 combo', gems:500, difficulty:'mythic'},
    {id:'settings_enthusiast', name:'SETTINGS ENTHUSIAST', desc:'Change graphics quality', gems:25, difficulty:'easy'},
    {id:'auto_save_hero', name:'AUTO-SAVE HERO', desc:'Trigger auto-save 10 times', gems:50, difficulty:'medium'},
    {id:'stat_enthusiast', name:'STAT ENTHUSIAST', desc:'Check stats panel 5 times', gems:30, difficulty:'easy'},
    {id:'damage_dealer', name:'DAMAGE DEALER', desc:'Deal 1,000,000 total damage', gems:200, difficulty:'extreme'},
    {id:'damage_god', name:'DAMAGE GOD', desc:'Deal 100,000,000 total damage', gems:1000, difficulty:'mythic'},
    {id:'perfect_game_plus', name:'PERFECT GAME PLUS', desc:'Complete a game without taking any damage on wave 50+', gems:500, difficulty:'mythic'},
    {id:'ultimate_collector', name:'ULTIMATE COLLECTOR', desc:'Own all skins', gems:2000, difficulty:'mythic'},
    {id:'absolute_perfection', name:'ABSOLUTE PERFECTION', desc:'Complete all achievements!', gems:10000, difficulty:'mythic'}
];

function getAchievementProgressValue(id){
    if(id==='first_kill') return kills>=1;
    if(id==='combo10') return combo>=10;
    if(id==='wave5') return wave>=5;
    if(id==='score5k') return score>=5000;
    if(id==='score25k') return score>=25000;
    if(id==='overdrive3') return odActivations>=3;
    if(id==='nodmg_wave') return waveNoDamage && wave>1;
    if(id==='boss1') return bossesKilled>=1;
    if(id==='rich') return totalCoins>=10000;
    if(id==='pyro') return fireLevel>=10;
    if(id==='warmonger') return damageLevel>=10;
    if(id==='invincible') return hasShieldUpgrade;
    if(id==='drone_army') return droneCount>=5;
    if(id==='demolition') return totalBombsUsed>=10;
    if(id==='overcharged') return totalOverdriveUses>=10;
    if(id==='kills100') return totalKills>=100;
    if(id==='kills500') return totalKills>=500;
    if(id==='legendary') return level>=15;
    if(id==='no_mercy') return waveKills>=50;
    if(id==='laser_master') return hasLaser;
    if(id==='perfect_wave') return perfectWavesCount>=3;
    if(id==='speedrun') return (wave>=5 && (Date.now()-startTime)<180000);
    if(id==='millionaire') return score>=1000000;
    if(id==='boss_genocide') return bossesKilled>=10;
    if(id==='true_god') return level>=30;
    if(id==='combo_god') return maxCombo>=30;
    if(id==='max_out') return fireLevel>=50||damageLevel>=50;
    if(id==='synapse_activate') return totalSynapseActivations>=3;
    if(id==='time_lord') return synapseKills>=50;
    if(id==='psychedelic') return psychedeliaCollected>=10;
    if(id==='decimator') return kills>=200;
    if(id==='immortal') return waveNoDamage && health===maxHealth;
    if(id==='survivor') return wave>=15;
    if(id==='god_of_war') return level>=50;
    if(id==='infinite_power') return odCharge>=200;
    if(id==='shopaholic') return totalUpgradesBought>=50;
    if(id==='credit_farm') return totalCoins>=50000;
    if(id==='meteor_slayer') return totalMeteorsDestroyed>=50;
    if(id==='rift_walker') return totalRiftsTriggered>=5;
    if(id==='event_master') return totalEventsTriggered>=10;
    if(id==='godlike') return wave>=50;
    if(id==='unstoppable') return kills>=1000;
    if(id==='perfect_run') return perfectWavesCount>=10;
    if(id==='ultimate_power') return level>=100;
    if(id==='zero_damage') return (wave>=5 && health===maxHealth);
    if(id==='speed_god') return (wave>=30 && (Date.now()-startTime)<600000);
    if(id==='billionaire') return score>=10000000;
    if(id==='true_survivor') return (Date.now()-startTime)>=3600000;
    if(id==='omega_boss') return bossesKilled>=50;
    if(id==='impossible') return wave>=100;
    if(id==='meteor_master') return totalMeteorsDestroyed>=200;
    if(id==='rift_god') return totalRiftsTriggered>=15;
    if(id==='event_collector') return totalEventsTriggered>=25;
    if(id==='fast_killer') return (waveKills>=10 && waveKills<15);
    if(id==='rich_lord') return totalCoins>=200000;
    if(id==='endless') return wave>=25;
    if(id==='combo_legend') return maxCombo>=50;
    if(id==='overdrive_god') return totalOverdriveUses>=50;
    if(id==='drone_master') return droneCount>=10;
    if(id==='fire_god') return fireLevel>=30;
    if(id==='damage_god') return damageLevel>=30;
    if(id==='drone_overlord') return droneCount>=400;
    if(id==='apocalypse_survivor') return apocalypseTriggers>=3;
    if(id==='void_walker') return voidTriggers>=5;
    if(id==='guardian_slayer') return guardianDefeated===true;
    if(id==='true_guardian') return (localStorage.getItem('guardianDefeatedCount')||0)>=10;
    if(id==='apocalypse_master') return apocalypseTriggers>=3 && kills>=500;
    if(id==='void_assassin') return voidTriggers>=5 && kills>=200;
    if(id==='event_god') return totalEventsTriggered>=50;
    if(id==='credit_king') return totalCoins>=1000000;
    if(id==='score_king') return score>=100000000;
    if(id==='wave_warrior') return wave>=200;
    if(id==='immortal_god') return perfectWavesCount>=50;
    if(id==='perfect_game') return (wave>=10 && health===maxHealth);
    if(id==='speed_demon') return (wave>=50 && (Date.now()-startTime)<900000);
    if(id==='overdrive_legend') return totalOverdriveUses>=200;
    if(id==='bomb_master') return totalBombsUsed>=100;
    if(id==='cosmic_collapse') return cosmicCollapseCount>=1;
    if(id==='primordial_rage') return primordialRageCount>=1;
    if(id==='black_hole') return blackHolePieces>=6;
    if(id==='cosmic_master') return cosmicCollapseCount>=5;
    if(id==='primordial_god') return primordialRageCount>=3;
    if(id==='black_hole_god') return (localStorage.getItem('blackHoleActivations')||0)>=10;
    if(id==='event_legend') return totalEventsTriggered>=100;
    if(id==='wave_500') return wave>=500;
    if(id==='rank_500') return level>=500;
    if(id==='kills_10000') return kills>=10000;
    if(id==='score_billion') return score>=1000000000;
    if(id==='credit_billion') return totalCoins>=100000000;
    if(id==='perfect_100') return perfectWavesCount>=100;
    if(id==='speed_100') return (wave>=100 && (Date.now()-startTime)<1800000);
    if(id==='overdrive_1000') return totalOverdriveUses>=1000;
    if(id==='bomb_1000') return totalBombsUsed>=1000;
    if(id==='drone_400') return droneCount>=400;
    if(id==='fire_100') return fireLevel>=100;
    if(id==='damage_100') return damageLevel>=100;
    if(id==='endless_master') return endlessMode && wave>=200;
    if(id==='death_touch') return deathTouchCount>=1;
    if(id==='chaos_realm') return chaosRealmCount>=1;
    if(id==='death_master') return deathTouchCount>=3;
    if(id==='chaos_master') return chaosRealmCount>=3;
    if(id==='reaper_call') return reaperCalls>=1;
    if(id==='time_warp') return timeWarps>=1;
    if(id==='gold_rush') return goldRushes>=1;
    if(id==='divine_intervention') return divineInterventions>=1;
    if(id==='bug_event') return bugEvents>=1;
    if(id==='stable_cycle') return stableCycleCount>=1;
    if(id==='tidal_wave') return tidalWaveCount>=1;
    if(id==='masquerade') return masqueradeCount>=1;
    if(id==='soul_harvest') return soulHarvestCount>=1;
    if(id==='frozen_time') return frozenTimeCount>=1;
    if(id==='crystal_rain') return crystalRainCount>=1;
    if(id==='shadow_clone') return shadowCloneCount>=1;
    if(id==='lightning_storm') return lightningStormCount>=1;
    if(id==='lucky_draw') return luckyDrawCount>=1;
    if(id==='mystery_box') return mysteryBoxCount>=1;
    if(id==='dooms_day') return doomsDayCount>=1;
    if(id==='royal_blessing') return royalBlessingCount>=1;
    if(id==='starfall') return starfallCount>=1;
    if(id==='inferno') return infernoCount>=1;
    if(id==='chain_lightning') return chainLightningCount>=1;
    if(id==='barrier') return barrierCount>=1;
    if(id==='soul_reaper') return soulReaperCount>=1;
    if(id==='gambler') return gamblerCount>=1;
    if(id==='vortex') return vortexCount>=1;
    if(id==='kings_blessing') return kingsBlessingCount>=1;
    if(id==='prism') return prismCount>=1;
    if(id==='abyss') return abyssCount>=1;
    if(id==='doppelganger') return doppelgangerCount>=1;
    if(id==='supernova') return supernovaCount>=1;
    if(id==='doppelganger_master') return doppelgangerCount>=5;
    if(id==='supernova_master') return supernovaCount>=5;
    if(id==='critical_hitter') return criticalHitsCount>=100;
    if(id==='critical_master') return criticalHitsCount>=1000;
    if(id==='combo_master_50') return maxCombo>=50;
    if(id==='combo_master_100') return maxCombo>=100;
    if(id==='settings_enthusiast') return settings.graphics !== 'high';
    if(id==='auto_save_hero') return (localStorage.getItem('autoSaveCount')||0)>=10;
    if(id==='stat_enthusiast') return (localStorage.getItem('statViewCount')||0)>=5;
    if(id==='damage_dealer') return totalDamageDealt>=1000000;
    if(id==='damage_god') return totalDamageDealt>=100000000;
    if(id==='perfect_game_plus') return (wave>=50 && health===maxHealth);
    if(id==='ultimate_collector') return ownedSkins.blue && ownedSkins.purple && ownedSkins.gold && ownedSkins.rainbow && ownedSkins.ultra && ownedSkins.legend;
    if(id==='absolute_perfection') return ACHIEVEMENTS_LIST.every(a => achievements[a.id]===true);
    return false;
}

function checkAchievements(){
    let anyNew=false;
    for(const a of ACHIEVEMENTS_LIST){
        if(!achievements[a.id] && getAchievementProgressValue(a.id)){
            achievements[a.id]=true;
            anyNew=true;
            let rewardGems = a.gems * gemMultiplier;
            gemstones += rewardGems;
            saveGemstones();
            showAchievementPopup(a.name, a.desc, rewardGems);
            addRankXP(rewardGems);
        }
    }
    if(anyNew){
        localStorage.setItem('achievements',JSON.stringify(achievements));
        checkSkinUnlock();
        updateAchievementsUI();
        updateSkinProgressUI();
        updateHubUI();
        updateSkinsUI();
    }
}

function checkSkinUnlock(){
    const unlockedCount = ACHIEVEMENTS_LIST.filter(a => achievements[a.id]===true).length;
    if(unlockedCount >= 40 && !skinUnlocked){
        skinUnlocked=true;
        localStorage.setItem('skinUnlocked','true');
        if(!ownedSkins.gold){
            ownedSkins.gold = true;
            localStorage.setItem('ownedSkins', JSON.stringify(ownedSkins));
        }
        showAchievementPopup('🌟 GOLDEN SKIN UNLOCKED', 'You can now equip the Golden Legend skin!');
        updateSkinProgressUI();
        updateHubUI();
        updateSkinsUI();
    }
    if(currentRank >= 50 && !ownedSkins.legend){
        ownedSkins.legend = true;
        localStorage.setItem('ownedSkins', JSON.stringify(ownedSkins));
        showAchievementPopup('🏆 LEGEND SKIN UNLOCKED', 'You reached Rank 50! Legend skin is yours!');
        updateSkinsUI();
    }
}

function updateSkinProgressUI(){
    const unlockedCount = ACHIEVEMENTS_LIST.filter(a => achievements[a.id]===true).length;
    const percent = Math.min(100, (unlockedCount/40)*100);
    document.getElementById('skin-progress-fill').style.width=percent+'%';
    document.getElementById('skin-percent').innerHTML=unlockedCount+'/40 ACHIEVEMENTS';
    document.getElementById('skin-status-text').innerText=skinUnlocked?'UNLOCKED (GOLDEN)':'LOCKED';
}

function getAchievementProgress(achId){
    const progressMap = {
        'kills100': {current: totalKills, target: 100},
        'kills500': {current: totalKills, target: 500},
        'kills10000': {current: kills, target: 10000},
        'meteor_slayer': {current: totalMeteorsDestroyed, target: 50},
        'meteor_master': {current: totalMeteorsDestroyed, target: 200},
        'rift_walker': {current: totalRiftsTriggered, target: 5},
        'rift_god': {current: totalRiftsTriggered, target: 15},
        'event_master': {current: totalEventsTriggered, target: 10},
        'event_collector': {current: totalEventsTriggered, target: 25},
        'event_god': {current: totalEventsTriggered, target: 50},
        'event_legend': {current: totalEventsTriggered, target: 100},
        'overdrive3': {current: odActivations, target: 3},
        'overdrive_god': {current: totalOverdriveUses, target: 50},
        'overdrive_legend': {current: totalOverdriveUses, target: 200},
        'overdrive_1000': {current: totalOverdriveUses, target: 1000},
        'bomb_master': {current: totalBombsUsed, target: 100},
        'bomb_1000': {current: totalBombsUsed, target: 1000},
        'combo10': {current: maxCombo, target: 10},
        'combo_legend': {current: maxCombo, target: 50},
        'combo_god': {current: maxCombo, target: 30},
        'wave5': {current: wave, target: 5},
        'survivor': {current: wave, target: 15},
        'godlike': {current: wave, target: 50},
        'impossible': {current: wave, target: 100},
        'wave_500': {current: wave, target: 500},
        'wave_warrior': {current: wave, target: 200},
        'perfect_wave': {current: perfectWavesCount, target: 3},
        'perfect_run': {current: perfectWavesCount, target: 10},
        'immortal_god': {current: perfectWavesCount, target: 50},
        'perfect_100': {current: perfectWavesCount, target: 100},
        'score5k': {current: score, target: 5000},
        'score25k': {current: score, target: 25000},
        'millionaire': {current: score, target: 1000000},
        'billionaire': {current: score, target: 10000000},
        'score_billion': {current: score, target: 1000000000},
        'rich': {current: totalCoins, target: 10000},
        'rich_lord': {current: totalCoins, target: 200000},
        'credit_king': {current: totalCoins, target: 1000000},
        'credit_billion': {current: totalCoins, target: 100000000},
        'pyro': {current: fireLevel, target: 10},
        'fire_god': {current: fireLevel, target: 30},
        'fire_100': {current: fireLevel, target: 100},
        'warmonger': {current: damageLevel, target: 10},
        'damage_god': {current: damageLevel, target: 30},
        'damage_100': {current: damageLevel, target: 100},
        'drone_army': {current: droneCount, target: 5},
        'drone_master': {current: droneCount, target: 10},
        'drone_overlord': {current: droneCount, target: 400},
        'drone_400': {current: droneCount, target: 400},
        'shopaholic': {current: totalUpgradesBought, target: 50},
        'legendary': {current: level, target: 15},
        'true_god': {current: level, target: 30},
        'god_of_war': {current: level, target: 50},
        'ultimate_power': {current: level, target: 100},
        'rank_500': {current: level, target: 500},
        'boss_genocide': {current: bossesKilled, target: 10},
        'omega_boss': {current: bossesKilled, target: 50},
        'decimator': {current: kills, target: 200},
        'unstoppable': {current: kills, target: 1000},
        'no_mercy': {current: waveKills, target: 50},
        'synapse_activate': {current: totalSynapseActivations, target: 3},
        'time_lord': {current: synapseKills, target: 50},
        'psychedelic': {current: psychedeliaCollected, target: 10},
        'apocalypse_survivor': {current: apocalypseTriggers, target: 3},
        'void_walker': {current: voidTriggers, target: 5},
        'apocalypse_master': {current: apocalypseTriggers, target: 3},
        'void_assassin': {current: voidTriggers, target: 5},
        'cosmic_collapse': {current: cosmicCollapseCount, target: 1},
        'cosmic_master': {current: cosmicCollapseCount, target: 5},
        'primordial_rage': {current: primordialRageCount, target: 1},
        'primordial_god': {current: primordialRageCount, target: 3},
        'death_touch': {current: deathTouchCount, target: 1},
        'chaos_realm': {current: chaosRealmCount, target: 1},
        'endless_master': {current: endlessMode && wave>=200 ? 1 : 0, target: 1},
        'reaper_call': {current: reaperCalls, target: 1},
        'time_warp': {current: timeWarps, target: 1},
        'gold_rush': {current: goldRushes, target: 1},
        'divine_intervention': {current: divineInterventions, target: 1},
        'bug_event': {current: bugEvents, target: 1},
        'stable_cycle': {current: stableCycleCount, target: 1},
        'tidal_wave': {current: tidalWaveCount, target: 1},
        'masquerade': {current: masqueradeCount, target: 1},
        'soul_harvest': {current: soulHarvestCount, target: 1},
        'frozen_time': {current: frozenTimeCount, target: 1},
        'crystal_rain': {current: crystalRainCount, target: 1},
        'shadow_clone': {current: shadowCloneCount, target: 1},
        'lightning_storm': {current: lightningStormCount, target: 1},
        'lucky_draw': {current: luckyDrawCount, target: 1},
        'mystery_box': {current: mysteryBoxCount, target: 1},
        'dooms_day': {current: doomsDayCount, target: 1},
        'royal_blessing': {current: royalBlessingCount, target: 1},
        'starfall': {current: starfallCount, target: 1},
        'inferno': {current: infernoCount, target: 1},
        'chain_lightning': {current: chainLightningCount, target: 1},
        'barrier': {current: barrierCount, target: 1},
        'soul_reaper': {current: soulReaperCount, target: 1},
        'gambler': {current: gamblerCount, target: 1},
        'vortex': {current: vortexCount, target: 1},
        'kings_blessing': {current: kingsBlessingCount, target: 1},
        'prism': {current: prismCount, target: 1},
        'abyss': {current: abyssCount, target: 1},
        'doppelganger': {current: doppelgangerCount, target: 1},
        'supernova': {current: supernovaCount, target: 1},
        'doppelganger_master': {current: doppelgangerCount, target: 5},
        'supernova_master': {current: supernovaCount, target: 5},
        'critical_hitter': {current: criticalHitsCount, target: 100},
        'critical_master': {current: criticalHitsCount, target: 1000},
        'combo_master_50': {current: maxCombo, target: 50},
        'combo_master_100': {current: maxCombo, target: 100},
        'damage_dealer': {current: totalDamageDealt, target: 1000000},
        'damage_god': {current: totalDamageDealt, target: 100000000}
    };
    return progressMap[achId] || {current: achievements[achId]?1:0, target: 1};
}

function openAchievements(){
    document.getElementById('start-screen').style.display='none';
    document.getElementById('achievements-screen').style.display='flex';
    updateAchievementsUI();
}
function closeAchievements(){
    document.getElementById('achievements-screen').style.display='none';
    document.getElementById('start-screen').style.display='flex';
}
function updateAchievementsUI(){
    const container=document.getElementById('ach-list');
    if(!container)return;
    container.innerHTML='';
    for(const a of ACHIEVEMENTS_LIST){
        const unlocked=achievements[a.id]===true;
        const progress = getAchievementProgress(a.id);
        const percent = Math.min(100, (progress.current/progress.target)*100);
        const div=document.createElement('div');
        div.className='ach-card '+(unlocked?'':'locked');
        div.innerHTML=`<div class="ach-status" style="float:left;">${unlocked?'✅':'🔒'}</div>
                       <div class="ach-name">${a.name}</div>
                       <div class="ach-desc">${a.desc}</div>
                       ${!unlocked && progress.target>1 ? `<div class="ach-progress-bar"><div class="ach-progress-fill" style="width:${percent}%"></div></div>
                       <div class="ach-progress-text">${formatNumber(progress.current)}/${formatNumber(progress.target)}</div>` : ''}
                       ${unlocked ? '<div class="ach-progress-text" style="color:gold;">✓ COMPLETED +'+a.gems+'💎</div>' : '<div class="ach-progress-text" style="color:#ffaa00;">🎁 '+a.gems+'💎</div>'}`;
        container.appendChild(div);
    }
}

function openEvents(){
    document.getElementById('start-screen').style.display='none';
    document.getElementById('events-screen').style.display='flex';
    updateEventsUI();
}
function closeEvents(){
    document.getElementById('events-screen').style.display='none';
    document.getElementById('start-screen').style.display='flex';
}
function updateEventsUI(){
    const container=document.getElementById('events-list-container');
    if(!container)return;
    container.innerHTML=`
        <div class="event-card"><div class="event-name">☄️ METEOR SHOWER</div><div class="event-chance">Chance: 60% every 4 waves (wave≥4)</div><div class="event-desc">Meteors fall from the sky. Destroy them for +200 points each. Meteor hit = 15 damage.</div></div>
        <div class="event-card"><div class="event-name">🌀 DIMENSION RIFT</div><div class="event-chance">Chance: 3% after wave 10</div><div class="event-desc">Double enemies, 2x points and credits for 15 seconds.</div></div>
        <div class="event-card"><div class="event-name">🧠 SYNAPSE MODE</div><div class="event-chance">Collect 8 Psychedelia (🧠) items</div><div class="event-desc">Time slows, 2.5x damage, trippy background for 10 seconds.</div></div>
        <div class="event-card"><div class="event-name">🔴 APOCALYPSE MODE</div><div class="event-chance">~0.5% every 5 waves (rare!)</div><div class="event-desc">Enemies doubled, faster, but 5x points. Lasts 15 seconds.</div></div>
        <div class="event-card"><div class="event-name">💜 VOID MODE</div><div class="event-chance">~0.3% after wave 15 (ultra rare!)</div><div class="event-desc">Become invincible, 10x damage for 15 seconds.</div></div>
        <div class="event-card"><div class="event-name">👑 THE GUARDIAN</div><div class="event-chance">~0.1% after wave 20 (legendary!)</div><div class="event-desc">Giant golden boss with 10,000 HP. Rewards: 50,000 points, 5,000 credits, and GUARANTEES golden skin!</div></div>
        <div class="event-card"><div class="event-name">🌌 COSMIC COLLAPSE</div><div class="event-chance">GUARANTEED every 40 waves!</div><div class="event-desc">Screen fills with enemies, 10x points, 5x credits for 20 seconds.</div></div>
        <div class="event-card"><div class="event-name">⚡ PRIMORDIAL RAGE</div><div class="event-chance">~0.003% (1 in 33,333) - EXTREMELY RARE!</div><div class="event-desc">Become giant red, invincible, massive bullet storms for 10 seconds.</div></div>
        <div class="event-card"><div class="event-name">🕳️ BLACK HOLE</div><div class="event-chance">Collect 6 Black Hole pieces (⭐) to activate</div><div class="event-desc">Sucks in all enemies, instantly killing them. All points and credits rewarded!</div></div>
        <div class="event-card"><div class="event-name">💀 DEATH'S TOUCH</div><div class="event-chance">~0.005% (1 in 20,000) - MYTHIC RARE!</div><div class="event-desc">All enemies on screen die instantly! 50x points for each enemy!</div></div>
        <div class="event-card"><div class="event-name">🌈 CHAOS REALM</div><div class="event-chance">~0.008% (1 in 12,500) - MYTHIC RARE!</div><div class="event-desc">Rainbow bullets, 5x fire rate, trippy rainbow background for 12 seconds!</div></div>
        <div class="event-card"><div class="event-name">💀 REAPER'S CALL</div><div class="event-chance">~0.002% (1 in 50,000) - MYTHIC RARE!</div><div class="event-desc">Reapers appear and automatically kill enemies for 10 seconds!</div></div>
        <div class="event-card"><div class="event-name">🌀 TIME WARP</div><div class="event-chance">~0.004% (1 in 25,000) - MYTHIC RARE!</div><div class="event-desc">All enemies slowed by 80% for 10 seconds!</div></div>
        <div class="event-card"><div class="event-name">💰 GOLD RUSH</div><div class="event-chance">~0.0005% (1 in 200,000) - LEGENDARY RARE!</div><div class="event-desc">10x credits and gemstones from all sources for 15 seconds!</div></div>
        <div class="event-card"><div class="event-name">⚡ DIVINE INTERVENTION</div><div class="event-chance">~0.00001% (1 in 10,000,000) - GODLY RARE!</div><div class="event-desc">Full heal + 30 seconds invincibility!</div></div>
        <div class="event-card"><div class="event-name">🐛 BUG EVENT</div><div class="event-chance">~0.001% (1 in 100,000) - MYSTERY EVENT!</div><div class="event-desc">The game glitches out... then rewards you with massive bonuses!</div></div>
        <div class="event-card"><div class="event-name">💫 STABLE CYCLE</div><div class="event-chance">GUARANTEED every 7 waves!</div><div class="event-desc">3x points and credits for 10 seconds!</div></div>
        <div class="event-card"><div class="event-name">🌊 TIDAL WAVE</div><div class="event-chance">~0.5% after wave 12 (rare!)</div><div class="event-desc">Massive wave of enemies, but 5x points per kill!</div></div>
        <div class="event-card"><div class="event-name">🎭 MASQUERADE</div><div class="event-chance">~0.3% after wave 15 (rare!)</div><div class="event-desc">Enemies disguise as items, but give 10x points!</div></div>
        <div class="event-card"><div class="event-name">💀 SOUL HARVEST</div><div class="event-chance">~0.1% after wave 20 (legendary!)</div><div class="event-desc">Defeated enemies release souls that attack other enemies!</div></div>
        <div class="event-card"><div class="event-name">❄️ FROZEN TIME</div><div class="event-chance">~0.2% after wave 15 (rare!)</div><div class="event-desc">All enemies frozen for 8 seconds! Take no damage from frozen enemies!</div></div>
        <div class="event-card"><div class="event-name">💎 CRYSTAL RAIN</div><div class="event-chance">~0.15% after wave 18 (rare!)</div><div class="event-desc">Crystals fall from the sky! Collect them for bonus gems and credits!</div></div>
        <div class="event-card"><div class="event-name">👥 SHADOW CLONE</div><div class="event-chance">~0.08% after wave 25 (legendary!)</div><div class="event-desc">Create a clone that fights alongside you for 15 seconds!</div></div>
        <div class="event-card"><div class="event-name">⚡ LIGHTNING STORM</div><div class="event-chance">~0.1% after wave 12 (rare!)</div><div class="event-desc">Lightning strikes random enemies, dealing massive damage!</div></div>
        <div class="event-card"><div class="event-name">🍀 LUCKY DRAW</div><div class="event-chance">~0.05% after wave 15 (rare!)</div><div class="event-desc">Get a random shop item for free!</div></div>
        <div class="event-card"><div class="event-name">🔮 MYSTERY BOX</div><div class="event-chance">~0.03% after wave 18 (rare!)</div><div class="event-desc">Open a mystery box with random rewards!</div></div>
        <div class="event-card"><div class="event-name">💀 DOOM'S DAY</div><div class="event-chance">~0.01% after wave 20 (legendary!)</div><div class="event-desc">All enemies weakened by 50% and drop 2x points!</div></div>
        <div class="event-card"><div class="event-name">👑 ROYAL BLESSING</div><div class="event-chance">~0.005% after wave 25 (mythic!)</div><div class="event-desc">3x to everything for 20 seconds!</div></div>
        <div class="event-card"><div class="event-name">🌟 STARFALL</div><div class="event-chance">~0.12% after wave 10 (rare!)</div><div class="event-desc">Stars fall from the sky! Collect them for bonuses!</div></div>
        <div class="event-card"><div class="event-name">🔥 INFERNO</div><div class="event-chance">~0.08% after wave 15 (rare!)</div><div class="event-desc">Flames spread across the screen, burning enemies!</div></div>
        <div class="event-card"><div class="event-name">⚡ CHAIN LIGHTNING</div><div class="event-chance">~0.06% after wave 18 (rare!)</div><div class="event-desc">Lightning chains between enemies, dealing massive damage!</div></div>
        <div class="event-card"><div class="event-name">🛡️ BARRIER</div><div class="event-chance">~0.2% after wave 12 (rare!)</div><div class="event-desc">Create a barrier that absorbs all damage for 10 seconds!</div></div>
        <div class="event-card"><div class="event-name">💀 SOUL REAPER</div><div class="event-chance">~0.02% after wave 25 (legendary!)</div><div class="event-desc">A giant reaper appears and harvests enemy souls!</div></div>
        <div class="event-card"><div class="event-name">🎲 GAMBLER</div><div class="event-chance">~0.01% after wave 20 (legendary!)</div><div class="event-desc">Gamble your credits for a chance to win big!</div></div>
        <div class="event-card"><div class="event-name">🌀 VORTEX</div><div class="event-chance">~0.04% after wave 22 (rare!)</div><div class="event-desc">A vortex sucks in and destroys enemies!</div></div>
        <div class="event-card"><div class="event-name">👑 KING'S BLESSING</div><div class="event-chance">~0.003% after wave 30 (mythic!)</div><div class="event-desc">The king blesses you with 5x everything for 15 seconds!</div></div>
        <div class="event-card"><div class="event-name">🌈 PRISM</div><div class="event-chance">~0.05% after wave 18 (rare!)</div><div class="event-desc">Split your bullets into rainbow colors for massive coverage!</div></div>
        <div class="event-card"><div class="event-name">🕳️ ABYSS</div><div class="event-chance">~0.002% after wave 35 (mythic!)</div><div class="event-desc">An abyss opens, swallowing all enemies!</div></div>
        <div class="event-card"><div class="event-name">🎭 DOPPELGANGER</div><div class="event-chance">~0.06% after wave 22 (rare!)</div><div class="event-desc">A clone of you appears and fights alongside you for 15 seconds!</div></div>
        <div class="event-card"><div class="event-name">💥 SUPERNOVA</div><div class="event-chance">~0.001% after wave 30 (mythic!)</div><div class="event-desc">A massive explosion kills all enemies on screen and drops massive bonuses!</div></div>
        <div class="event-card"><div class="event-name" style="color:#ffaa00;">⏱️ EVENT COOLDOWN</div><div class="event-chance">3 seconds between events</div><div class="event-desc">Events cannot trigger one after another - 3 second grace period!</div></div>
        <div class="event-card"><div class="event-name" style="color:gold;">🌟 ENDLESS MODE</div><div class="event-chance">Available after wave 100</div><div class="event-desc">Continue infinitely with progressively stronger enemies!</div></div>
    `;
}

// AUDIO FUNCTIONS
const AudioCtx=window.AudioContext||window.webkitAudioContext;
let audioCtx;
function initAudio(){if(!audioCtx && settings.sound) audioCtx=new AudioCtx();}
function playBeep(freq,dur,vol,type='square',freqEnd){
    if(!settings.sound) return;
    if(!audioCtx) return;
    try{
        const o=audioCtx.createOscillator(),g=audioCtx.createGain();
        o.connect(g);g.connect(audioCtx.destination);
        o.type=type;o.frequency.setValueAtTime(freq,audioCtx.currentTime);
        if(freqEnd) o.frequency.exponentialRampToValueAtTime(freqEnd,audioCtx.currentTime+dur);
        g.gain.setValueAtTime(vol,audioCtx.currentTime);
        g.gain.exponentialRampToValueAtTime(0.001,audioCtx.currentTime+dur);
        o.start();o.stop(audioCtx.currentTime+dur);
    }catch(e){}
}
function sfxShoot()   {playBeep(880,0.04,0.04,'sawtooth');}
function sfxHit()     {playBeep(220,0.07,0.1,'square');}
function sfxExplode() {playBeep(110,0.18,0.12,'sawtooth');}
function sfxCoin()    {playBeep(1200,0.05,0.06,'sine');}
function sfxLevelUp() {playBeep(660,0.1,0.08,'sine');setTimeout(()=>playBeep(880,0.1,0.08,'sine'),100);}
function sfxOverdrive(){playBeep(440,0.3,0.12,'sawtooth');playBeep(880,0.3,0.08,'sine');}
function sfxBomb()    {playBeep(60,0.5,0.15,'sawtooth',30);}
function sfxWave()    {playBeep(300,0.08,0.06,'sine');setTimeout(()=>playBeep(400,0.08,0.06,'sine'),90);}
function sfxPowerup() {playBeep(800,0.06,0.08,'sine');setTimeout(()=>playBeep(1000,0.06,0.08,'sine'),70);}
function sfxLaser()   {playBeep(200,0.04,0.05,'sawtooth',800);}
function sfxSynapse() {playBeep(500,0.2,0.1,'sine');playBeep(700,0.2,0.1,'sine');playBeep(900,0.3,0.1,'sine');}
function sfxEvent()   {playBeep(400,0.15,0.12,'sine');playBeep(600,0.2,0.12,'sine');}
function sfxApocalypse(){playBeep(100,0.3,0.15,'sawtooth');playBeep(150,0.3,0.15,'sawtooth');playBeep(200,0.4,0.15,'sawtooth');}
function sfxVoid(){playBeep(600,0.2,0.1,'sine');playBeep(900,0.2,0.1,'sine');playBeep(1200,0.3,0.1,'sine');}
function sfxCosmic(){playBeep(50,0.5,0.2,'sine');playBeep(100,0.5,0.2,'sine');playBeep(200,0.5,0.2,'sine');}
function sfxDeathTouch(){playBeep(40,0.8,0.2,'sawtooth');playBeep(20,0.8,0.2,'sawtooth');playBeep(10,1,0.2,'sawtooth');}
function sfxChaosRealm(){playBeep(800,0.1,0.1,'sine');playBeep(1000,0.1,0.1,'sine');playBeep(1200,0.1,0.1,'sine');playBeep(1400,0.2,0.1,'sine');}
function sfxReaper(){playBeep(300,0.2,0.15,'sawtooth');playBeep(200,0.2,0.15,'sawtooth');playBeep(100,0.3,0.15,'sawtooth');}
function sfxTimeWarp(){playBeep(600,0.2,0.1,'sine');playBeep(400,0.2,0.1,'sine');playBeep(200,0.3,0.1,'sine');}
function sfxGoldRush(){playBeep(1000,0.1,0.15,'sine');playBeep(1200,0.1,0.15,'sine');playBeep(1400,0.2,0.15,'sine');}
function sfxDivine(){playBeep(500,0.3,0.2,'sine');playBeep(800,0.3,0.2,'sine');playBeep(1200,0.4,0.2,'sine');}
function sfxBug(){playBeep(300,0.1,0.2,'sawtooth');playBeep(200,0.1,0.2,'sawtooth');playBeep(100,0.1,0.2,'sawtooth');playBeep(50,0.2,0.2,'sawtooth');}
function sfxStable(){playBeep(500,0.2,0.15,'sine');playBeep(600,0.2,0.15,'sine');}
function sfxTidal(){playBeep(100,0.3,0.2,'sawtooth');playBeep(200,0.3,0.2,'sawtooth');}
function sfxMasquerade(){playBeep(800,0.1,0.1,'sine');playBeep(900,0.1,0.1,'sine');playBeep(1000,0.2,0.1,'sine');}
function sfxSoul(){playBeep(400,0.2,0.15,'sine');playBeep(300,0.2,0.15,'sine');}
function sfxFrozen(){playBeep(300,0.2,0.15,'sine');playBeep(200,0.2,0.15,'sine');}
function sfxCrystal(){playBeep(1000,0.1,0.15,'sine');playBeep(1100,0.1,0.15,'sine');playBeep(1200,0.2,0.15,'sine');}
function sfxShadow(){playBeep(400,0.2,0.15,'sine');playBeep(300,0.2,0.15,'sine');playBeep(200,0.2,0.15,'sine');}
function sfxLightning(){playBeep(800,0.1,0.2,'sawtooth');playBeep(1000,0.1,0.2,'sawtooth');}
function sfxLucky(){playBeep(600,0.2,0.15,'sine');playBeep(800,0.2,0.15,'sine');}
function sfxMystery(){playBeep(500,0.2,0.15,'sine');playBeep(700,0.2,0.15,'sine');playBeep(900,0.2,0.15,'sine');}
function sfxDoom(){playBeep(100,0.5,0.2,'sawtooth');playBeep(80,0.5,0.2,'sawtooth');}
function sfxRoyal(){playBeep(600,0.2,0.15,'sine');playBeep(800,0.2,0.15,'sine');playBeep(1000,0.2,0.15,'sine');}
function sfxStarfall(){playBeep(800,0.1,0.15,'sine');playBeep(900,0.1,0.15,'sine');playBeep(1000,0.2,0.15,'sine');}
function sfxInferno(){playBeep(200,0.3,0.2,'sawtooth');playBeep(300,0.3,0.2,'sawtooth');}
function sfxChain(){playBeep(600,0.1,0.15,'sine');playBeep(700,0.1,0.15,'sine');playBeep(800,0.1,0.15,'sine');}
function sfxBarrier(){playBeep(400,0.2,0.15,'sine');playBeep(500,0.2,0.15,'sine');}
function sfxSoulReaper(){playBeep(100,0.5,0.2,'sawtooth');playBeep(80,0.5,0.2,'sawtooth');playBeep(60,0.5,0.2,'sawtooth');}
function sfxGambler(){playBeep(600,0.1,0.15,'sine');playBeep(800,0.1,0.15,'sine');playBeep(1000,0.2,0.15,'sine');}
function sfxVortex(){playBeep(200,0.3,0.2,'sawtooth');playBeep(150,0.3,0.2,'sawtooth');}
function sfxKings(){playBeep(500,0.2,0.2,'sine');playBeep(700,0.2,0.2,'sine');playBeep(900,0.3,0.2,'sine');}
function sfxPrism(){playBeep(800,0.1,0.15,'sine');playBeep(1000,0.1,0.15,'sine');playBeep(1200,0.2,0.15,'sine');}
function sfxAbyss(){playBeep(50,0.5,0.2,'sawtooth');playBeep(30,0.5,0.2,'sawtooth');}
function sfxDoppelganger(){playBeep(600,0.2,0.15,'sine');playBeep(700,0.2,0.15,'sine');playBeep(800,0.2,0.15,'sine');}
function sfxSupernova(){playBeep(80,0.5,0.25,'sawtooth');playBeep(60,0.5,0.25,'sawtooth');playBeep(40,0.5,0.25,'sawtooth');}

// CROSSHAIR
const crossCanvas=document.getElementById('crosshair'),cCtx=crossCanvas.getContext('2d');
crossCanvas.style.pointerEvents='none';
function resizeCross(){crossCanvas.width=window.innerWidth;crossCanvas.height=window.innerHeight;}
resizeCross();
function drawCrosshair(x,y){
    cCtx.clearRect(0,0,crossCanvas.width,crossCanvas.height);
    if(gameState!=='PLAYING')return;
    const size=12,gap=4,col=isOD?'#ff00ff':synapseActive?'#ff66ff':voidActive?'#aa66ff':primordialRageActive?'#ff0000':chaosRealmActive?`hsl(${chaosHue},100%,60%)`:apocalypseActive?'#ff0000':riftActive?'#aa66ff':bugEventActive?'#00ff00':stableCycleActive?'#88aaff':lightningStormActive?'#ffff00':royalBlessingActive?'#ffdd00':kingsBlessingActive?'#ffdd00':doppelgangerActive?'#ff88ff':'#00d2ff';
    cCtx.strokeStyle=col;cCtx.lineWidth=2;cCtx.shadowBlur=5;
    cCtx.beginPath();
    cCtx.moveTo(x-size,y);cCtx.lineTo(x-gap,y);
    cCtx.moveTo(x+gap,y); cCtx.lineTo(x+size,y);
    cCtx.moveTo(x,y-size);cCtx.lineTo(x,y-gap);
    cCtx.moveTo(x,y+gap); cCtx.lineTo(x,y+size);
    cCtx.stroke();
    cCtx.beginPath();cCtx.arc(x,y,3,0,Math.PI*2);cCtx.fillStyle=col;cCtx.fill();
}

function drawNebula(){
    if(settings.graphics === 'low') return;
    nCtx.clearRect(0,0,width,height);
    const g1=nCtx.createRadialGradient(width*0.3,height*0.4,0,width*0.3,height*0.4,width*0.5);
    g1.addColorStop(0,'rgba(0,100,180,0.25)');g1.addColorStop(1,'rgba(0,0,0,0)');
    nCtx.fillStyle=g1;nCtx.fillRect(0,0,width,height);
    const g2=nCtx.createRadialGradient(width*0.75,height*0.6,0,width*0.75,height*0.6,width*0.4);
    g2.addColorStop(0,'rgba(100,0,120,0.2)');g2.addColorStop(1,'rgba(0,0,0,0)');
    nCtx.fillStyle=g2;nCtx.fillRect(0,0,width,height);
}

function runLoader(id,barId,cb){
    const scr=document.getElementById(id),bar=document.getElementById(barId);
    scr.style.display='flex';let p=0;
    const iv=setInterval(()=>{
        p+=Math.random()*9;
        if(p>=100){p=100;clearInterval(iv);setTimeout(()=>{scr.style.display='none';cb();},400);}
        bar.style.width=p+'%';
    },70);
}

// POWER-UPS
const POWERUP_TYPES=[
    {id:'rapidfire',name:'RAPID FIRE',color:'#00d2ff',icon:'⚡',dur:7000},
    {id:'doubleDmg',name:'POWER AMP', color:'#ff4444',icon:'💪',dur:6000},
    {id:'magnet',   name:'MAGNET',    color:'gold',    icon:'🧲',dur:9000},
    {id:'regen',    name:'REGEN',     color:'#00ffaa', icon:'💚',dur:7000},
];
function spawnPowerUp(x,y){
    const t=POWERUP_TYPES[Math.floor(Math.random()*POWERUP_TYPES.length)];
    items.push({x,y,vy:2,isPowerUp:true,puType:t});
}
function spawnPsychedelia(x,y){
    items.push({x,y,vy:2,isPowerUp:false,isPsychedelia:true});
}
function spawnBlackHolePiece(x,y){
    items.push({x,y,vy:2,isPowerUp:false,isBlackHolePiece:true});
}
function applyPowerUp(t){
    activePowerUps[t.id]={name:t.name,color:t.color,icon:t.icon,endTime:Date.now()+t.dur};
    sfxPowerup();
    if(player && player.x && player.y) floats.push({txt:t.icon+' '+t.name+'!',x:player.x-50,y:player.y-35,l:1.5,c:t.color,size:18});
    updatePowerUpBar();
}
function updatePowerUpBar(){
    const bar=document.getElementById('powerup-bar'),now=Date.now();
    const active=Object.values(activePowerUps).filter(p=>p.endTime>now);
    if(active.length===0){bar.style.display='none';bar.innerHTML='';return;}
    bar.style.display='block';
    bar.innerHTML=active.map(p=>{
        const rem=Math.ceil((p.endTime-now)/1000);
        return `<span class="pu-item" style="border-color:${p.color};color:${p.color}">${p.icon} ${p.name} ${rem}s</span>`;
    }).join('');
}
function tickPowerUps(){
    const now=Date.now();
    for(const k of Object.keys(activePowerUps)) if(activePowerUps[k].endTime<=now) delete activePowerUps[k];
    if(activePowerUps.regen&&Math.random()<0.008) health=Math.min(maxHealth,health+0.5);
    updatePowerUpBar();
}

// EVENT COOLDOWN
function canTriggerEvent(){
    if(activeEvent) return false;
    if(eventCooldown > Date.now()) return false;
    return true;
}
function startEventCooldown(){ eventCooldown = Date.now() + 3000; }

// EVENT TRIGGER FUNCTIONS (abbreviated - same as before)
function triggerDoppelganger(){
    if(!canTriggerEvent()) return;
    activeEvent='doppelganger'; eventTimer=Date.now()+15000; doppelgangerCount++; totalEventsTriggered++; sfxDoppelganger();
    doppelgangerActive=true; doppelgangerTimer=Date.now()+15000;
    doppelgangerClone = new DoppelgangerClone();
    const banner=document.getElementById('event-banner');
    banner.innerHTML='🎭 DOPPELGANGER 🎭'; banner.style.display='block'; banner.style.color='#ff88ff';
    setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000);
    if(player && player.x && player.y) floats.push({txt:'🎭 DOPPELGANGER!',x:width/2-70,y:height/2-40,l:1.5,c:'#ff88ff',size:24});
    showNotification('🎭 Doppelganger event started! A clone fights for you!', 'event');
    startEventCooldown(); checkAchievements();
}

function triggerSupernova(){
    if(!canTriggerEvent()) return;
    activeEvent='supernova'; eventTimer=Date.now()+3000; supernovaCount++; totalEventsTriggered++; sfxSupernova();
    supernovaActive=true; supernovaTimer=Date.now()+3000;
    const banner=document.getElementById('event-banner');
    banner.innerHTML='💥 SUPERNOVA 💥'; banner.style.display='block'; banner.style.color='#ffaa44';
    setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000);
    let enemiesKilled = 0;
    for(let i=0;i<enemies.length;i++){
        let e=enemies[i];
        let pointBonus = e.isBoss?50000:5000*combo;
        if(goldRushActive) pointBonus*=10;
        if(stableCycleActive) pointBonus*=3;
        if(royalBlessingActive) pointBonus*=3;
        if(kingsBlessingActive) pointBonus*=5;
        pointBonus *= skinCreditMultiplier;
        score+=pointBonus; kills++; waveKills++;
        if(e.isBoss) bossesKilled++;
        enemiesKilled++;
        for(let k=0;k<30;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*20,(Math.random()-0.5)*20,'#ffaa44',1));
    }
    enemies = [];
    boss = null;
    for(let i=0;i<10+Math.floor(Math.random()*10);i++){
        items.push({x:Math.random()*width, y:height/2, vy:2, isPowerUp:false});
    }
    gemstones += 500 * gemMultiplier;
    saveGemstones();
    totalCoins += 10000;
    localStorage.setItem('totalCoins', totalCoins);
    if(player && player.x && player.y) floats.push({txt:`💥 SUPERNOVA! ${enemiesKilled} ENEMIES DIED! +500💎 +10000c`,x:width/2-150,y:height/2-40,l:1.8,c:'#ffaa44',size:26});
    showNotification(`💥 SUPERNOVA killed ${enemiesKilled} enemies! +500 GEMSTONES, +10000 CREDITS!`, 'event');
    startEventCooldown(); checkAchievements();
}

function triggerMeteorShower(){ if(!canTriggerEvent()) return; activeEvent='meteor'; eventTimer=Date.now()+10000; totalEventsTriggered++; sfxEvent(); const banner=document.getElementById('event-banner'); banner.innerHTML='☄️ METEOR SHOWER ☄️'; banner.style.display='block'; banner.style.color='#ffaa44'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); for(let i=0;i<15+Math.floor(Math.random()*15);i++) meteors.push(new Meteor(Math.random()*width, -50)); startEventCooldown(); checkAchievements(); showNotification('☄️ Meteor Shower event started!', 'event'); }
function triggerDimensionRift(){ if(!canTriggerEvent()) return; activeEvent='rift'; eventTimer=Date.now()+15000; riftActive=true; riftMultiplier=2; totalEventsTriggered++; totalRiftsTriggered++; sfxEvent(); const banner=document.getElementById('event-banner'); banner.innerHTML='🌀 DIMENSION RIFT 🌀'; banner.style.display='block'; banner.style.color='#aa66ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); for(let i=0;i<8;i++) enemies.push(new Enemy()); startEventCooldown(); checkAchievements(); showNotification('🌀 Dimension Rift event started! 2x points!', 'event'); }
function triggerApocalypseMode(){ if(!canTriggerEvent()) return; apocalypseActive=true; activeEvent='apocalypse'; eventTimer=Date.now()+15000; apocalypseTriggers++; totalEventsTriggered++; sfxApocalypse(); const banner=document.getElementById('event-banner'); banner.innerHTML='🔴 APOCALYPSE MODE 🔴'; banner.style.display='block'; banner.style.color='#ff0000'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); for(let i=0;i<12;i++) enemies.push(new Enemy()); if(player && player.x && player.y) floats.push({txt:'🔴 APOCALYPSE!',x:width/2-60,y:height/2-40,l:1.5,c:'#ff0000',size:24}); startEventCooldown(); checkAchievements(); showNotification('🔴 Apocalypse Mode event started! 5x points!', 'event'); }
function triggerVoidMode(){ if(!canTriggerEvent()) return; voidActive=true; voidTimer=Date.now()+15000; activeEvent='void'; eventTimer=Date.now()+15000; voidTriggers++; totalEventsTriggered++; sfxVoid(); const banner=document.getElementById('event-banner'); banner.innerHTML='💜 VOID MODE 💜'; banner.style.display='block'; banner.style.color='#aa66ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💜 VOID MODE!',x:width/2-50,y:height/2-40,l:1.5,c:'#aa66ff',size:24}); startEventCooldown(); checkAchievements(); showNotification('💜 Void Mode event started! Invincible!', 'event'); }
function triggerGuardian(){ if(!canTriggerEvent()) return; guardian=new Guardian(); activeEvent='guardian'; sfxApocalypse(); const banner=document.getElementById('event-banner'); banner.innerHTML='👑 THE GUARDIAN AWAKENS 👑'; banner.style.display='block'; banner.style.color='gold'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},4000); if(player && player.x && player.y) floats.push({txt:'👑 GUARDIAN!',x:width/2-50,y:height/2-50,l:2,c:'gold',size:32}); startEventCooldown(); showNotification('👑 The Guardian has awakened!', 'warning'); }
function triggerCosmicCollapse(){ if(!canTriggerEvent()) return; activeEvent='cosmic'; eventTimer=Date.now()+20000; cosmicCollapseActive=true; cosmicCollapseCount++; totalEventsTriggered++; sfxCosmic(); const banner=document.getElementById('event-banner'); banner.innerHTML='🌌 COSMIC COLLAPSE 🌌'; banner.style.display='block'; banner.style.color='#88aaff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); for(let i=0;i<30;i++) enemies.push(new Enemy()); if(player && player.x && player.y) floats.push({txt:'🌌 COSMIC COLLAPSE!',x:width/2-70,y:height/2-40,l:1.5,c:'#88aaff',size:24}); startEventCooldown(); checkAchievements(); showNotification('🌌 Cosmic Collapse event started! 10x points!', 'event'); }
function triggerPrimordialRage(){ if(!canTriggerEvent()) return; activeEvent='primordial'; eventTimer=Date.now()+10000; primordialRageActive=true; primordialRageCount++; totalEventsTriggered++; sfxApocalypse(); const banner=document.getElementById('event-banner'); banner.innerHTML='⚡ PRIMORDIAL RAGE ⚡'; banner.style.display='block'; banner.style.color='#ff0000'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player) player.r = 60; if(player && player.x && player.y) floats.push({txt:'⚡ PRIMORDIAL RAGE!',x:width/2-70,y:height/2-40,l:1.5,c:'#ff0000',size:24}); startEventCooldown(); checkAchievements(); showNotification('⚡ Primordial Rage event started! Massive damage!', 'event'); }
function triggerBlackHole(){ if(!canTriggerEvent()) return; activeEvent='blackhole'; eventTimer=Date.now()+8000; blackHoleActive=true; blackHoleCenter={x:width/2, y:height/2}; totalEventsTriggered++; sfxCosmic(); const banner=document.getElementById('event-banner'); banner.innerHTML='🕳️ BLACK HOLE 🕳️'; banner.style.display='block'; banner.style.color='#4400aa'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); let activations=parseInt(localStorage.getItem('blackHoleActivations')||0)+1; localStorage.setItem('blackHoleActivations',activations); if(player && player.x && player.y) floats.push({txt:'🕳️ BLACK HOLE!',x:width/2-50,y:height/2-40,l:1.5,c:'#4400aa',size:24}); startEventCooldown(); checkAchievements(); showNotification('🕳️ Black Hole event started!', 'event'); }
function triggerDeathTouch(){ if(!canTriggerEvent()) return; activeEvent='deathtouch'; eventTimer=Date.now()+3000; deathTouchCount++; totalEventsTriggered++; sfxDeathTouch(); const banner=document.getElementById('event-banner'); banner.innerHTML='💀 DEATH\'S TOUCH 💀'; banner.style.display='block'; banner.style.color='#440044'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); let enemiesKilled=0; for(let i=0;i<enemies.length;i++){ let e=enemies[i]; let pointBonus = e.isBoss?90000:3500*combo; if(goldRushActive) pointBonus*=10; if(stableCycleActive) pointBonus*=3; if(royalBlessingActive) pointBonus*=3; if(kingsBlessingActive) pointBonus*=5; score+=pointBonus; kills++; waveKills++; if(e.isBoss) bossesKilled++; enemiesKilled++; for(let k=0;k<20;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*15,(Math.random()-0.5)*15,'#440044',0.9)); } enemies=[]; boss=null; if(player && player.x && player.y) floats.push({txt:`💀 ${enemiesKilled} ENEMIES DIED!`,x:width/2-100,y:height/2-40,l:1.8,c:'#440044',size:28}); startEventCooldown(); checkAchievements(); showNotification(`💀 Death's Touch killed ${enemiesKilled} enemies!`, 'event'); }
function triggerChaosRealm(){ if(!canTriggerEvent()) return; activeEvent='chaos'; eventTimer=Date.now()+12000; chaosRealmActive=true; chaosRealmCount++; totalEventsTriggered++; sfxChaosRealm(); const banner=document.getElementById('event-banner'); banner.innerHTML='🌈 CHAOS REALM 🌈'; banner.style.display='block'; banner.style.color='#ff66ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🌈 CHAOS REALM!',x:width/2-70,y:height/2-40,l:1.5,c:'#ff66ff',size:24}); startEventCooldown(); checkAchievements(); showNotification('🌈 Chaos Realm event started! Rainbow bullets!', 'event'); }
function triggerReapersCall(){ if(!canTriggerEvent()) return; activeEvent='reaper'; eventTimer=Date.now()+10000; reaperCalls++; totalEventsTriggered++; sfxReaper(); const banner=document.getElementById('event-banner'); banner.innerHTML='💀 REAPER\'S CALL 💀'; banner.style.display='block'; banner.style.color='#880044'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💀 REAPER\'S CALL!',x:width/2-70,y:height/2-40,l:1.5,c:'#880044',size:24}); startEventCooldown(); checkAchievements(); showNotification('💀 Reaper\'s Call event started!', 'event'); }
function triggerTimeWarp(){ if(!canTriggerEvent()) return; activeEvent='timewarp'; eventTimer=Date.now()+10000; timeWarps++; totalEventsTriggered++; sfxTimeWarp(); timeWarpActive=true; timeWarpTimer=Date.now()+10000; const banner=document.getElementById('event-banner'); banner.innerHTML='🌀 TIME WARP 🌀'; banner.style.display='block'; banner.style.color='#44aaff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🌀 TIME WARP!',x:width/2-60,y:height/2-40,l:1.5,c:'#44aaff',size:24}); startEventCooldown(); checkAchievements(); showNotification('🌀 Time Warp event started! Enemies slowed!', 'event'); }
function triggerGoldRush(){ if(!canTriggerEvent()) return; activeEvent='goldrush'; eventTimer=Date.now()+15000; goldRushes++; totalEventsTriggered++; sfxGoldRush(); goldRushActive=true; goldRushTimer=Date.now()+15000; const banner=document.getElementById('event-banner'); banner.innerHTML='💰 GOLD RUSH 💰'; banner.style.display='block'; banner.style.color='#ffaa00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💰 GOLD RUSH!',x:width/2-60,y:height/2-40,l:1.5,c:'#ffaa00',size:24}); startEventCooldown(); checkAchievements(); showNotification('💰 Gold Rush event started! 10x credits!', 'event'); }
function triggerDivineIntervention(){ if(!canTriggerEvent()) return; activeEvent='divine'; eventTimer=Date.now()+30000; divineInterventions++; totalEventsTriggered++; sfxDivine(); divineActive=true; divineTimer=Date.now()+30000; health = maxHealth; if(player) player.invincibleTimer = 3000; const banner=document.getElementById('event-banner'); banner.innerHTML='⚡ DIVINE INTERVENTION ⚡'; banner.style.display='block'; banner.style.color='#ffdd00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},4000); if(player && player.x && player.y) floats.push({txt:'⚡ DIVINE INTERVENTION!',x:width/2-80,y:height/2-40,l:1.5,c:'#ffdd00',size:28}); startEventCooldown(); checkAchievements(); showNotification('⚡ Divine Intervention event started! Full heal!', 'event'); }
function triggerBugEvent(){ if(!canTriggerEvent()) return; activeEvent='bug'; eventTimer=Date.now()+5000; bugEvents++; totalEventsTriggered++; sfxBug(); bugEventActive=true; bugEventGlitch=true; const banner=document.getElementById('event-banner'); banner.innerHTML='🐛 BUG EVENT 🐛'; banner.style.display='block'; banner.style.color='#00ff00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🐛 BUG EVENT!',x:width/2-60,y:height/2-40,l:1.5,c:'#00ff00',size:24}); setTimeout(() => { if(gameState === 'PLAYING'){ let reward = 777 * gemMultiplier; gemstones += reward; saveGemstones(); totalCoins += 7777; localStorage.setItem('totalCoins', totalCoins); showAchievementPopup('🐛 BUG EVENT RESOLVED', `You received ${reward} GEMSTONES and 7,777 CREDITS!`, reward); bugEventActive = false; bugEventGlitch = false; } }, 3000); startEventCooldown(); checkAchievements(); showNotification('🐛 Bug Event triggered! Glitch incoming...', 'event'); }
function triggerStableCycle(){ if(!canTriggerEvent()) return; activeEvent='stable'; eventTimer=Date.now()+10000; stableCycleCount++; totalEventsTriggered++; sfxStable(); stableCycleActive=true; stableCycleTimer=Date.now()+10000; const banner=document.getElementById('event-banner'); banner.innerHTML='💫 STABLE CYCLE 💫'; banner.style.display='block'; banner.style.color='#88aaff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💫 STABLE CYCLE!',x:width/2-60,y:height/2-40,l:1.5,c:'#88aaff',size:24}); startEventCooldown(); checkAchievements(); showNotification('💫 Stable Cycle event started! 3x points!', 'event'); }
function triggerTidalWave(){ if(!canTriggerEvent()) return; activeEvent='tidal'; eventTimer=Date.now()+12000; tidalWaveCount++; totalEventsTriggered++; sfxTidal(); tidalWaveActive=true; tidalWaveTimer=Date.now()+12000; const banner=document.getElementById('event-banner'); banner.innerHTML='🌊 TIDAL WAVE 🌊'; banner.style.display='block'; banner.style.color='#44aaff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); for(let i=0;i<20;i++) enemies.push(new Enemy()); if(player && player.x && player.y) floats.push({txt:'🌊 TIDAL WAVE!',x:width/2-60,y:height/2-40,l:1.5,c:'#44aaff',size:24}); startEventCooldown(); checkAchievements(); showNotification('🌊 Tidal Wave event started! Many enemies!', 'event'); }
function triggerMasquerade(){ if(!canTriggerEvent()) return; activeEvent='masquerade'; eventTimer=Date.now()+15000; masqueradeCount++; totalEventsTriggered++; sfxMasquerade(); masqueradeActive=true; masqueradeTimer=Date.now()+15000; const banner=document.getElementById('event-banner'); banner.innerHTML='🎭 MASQUERADE 🎭'; banner.style.display='block'; banner.style.color='#aa66ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🎭 MASQUERADE!',x:width/2-60,y:height/2-40,l:1.5,c:'#aa66ff',size:24}); startEventCooldown(); checkAchievements(); showNotification('🎭 Masquerade event started! Disguised enemies!', 'event'); }
function triggerSoulHarvest(){ if(!canTriggerEvent()) return; activeEvent='soul'; eventTimer=Date.now()+15000; soulHarvestCount++; totalEventsTriggered++; sfxSoul(); soulHarvestActive=true; soulHarvestTimer=Date.now()+15000; soulHarvestSouls = []; const banner=document.getElementById('event-banner'); banner.innerHTML='💀 SOUL HARVEST 💀'; banner.style.display='block'; banner.style.color='#8800aa'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💀 SOUL HARVEST!',x:width/2-70,y:height/2-40,l:1.5,c:'#8800aa',size:24}); startEventCooldown(); checkAchievements(); showNotification('💀 Soul Harvest event started! Souls attack!', 'event'); }
function triggerFrozenTime(){ if(!canTriggerEvent()) return; activeEvent='frozen'; eventTimer=Date.now()+8000; frozenTimeCount++; totalEventsTriggered++; sfxFrozen(); frozenTimeActive=true; frozenTimeTimer=Date.now()+8000; const banner=document.getElementById('event-banner'); banner.innerHTML='❄️ FROZEN TIME ❄️'; banner.style.display='block'; banner.style.color='#88ccff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'❄️ FROZEN TIME!',x:width/2-60,y:height/2-40,l:1.5,c:'#88ccff',size:24}); startEventCooldown(); checkAchievements(); showNotification('❄️ Frozen Time event started! Enemies frozen!', 'event'); }
function triggerCrystalRain(){ if(!canTriggerEvent()) return; activeEvent='crystal'; eventTimer=Date.now()+10000; crystalRainCount++; totalEventsTriggered++; sfxCrystal(); crystalRainActive=true; crystalRainTimer=Date.now()+10000; const banner=document.getElementById('event-banner'); banner.innerHTML='💎 CRYSTAL RAIN 💎'; banner.style.display='block'; banner.style.color='#88ffaa'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💎 CRYSTAL RAIN!',x:width/2-60,y:height/2-40,l:1.5,c:'#88ffaa',size:24}); startEventCooldown(); checkAchievements(); showNotification('💎 Crystal Rain event started! Collect crystals!', 'event'); }
function triggerShadowClone(){ if(!canTriggerEvent()) return; activeEvent='shadow'; eventTimer=Date.now()+15000; shadowCloneCount++; totalEventsTriggered++; sfxShadow(); shadowCloneActive=true; shadowCloneTimer=Date.now()+15000; shadowClone = new ShadowClone(); const banner=document.getElementById('event-banner'); banner.innerHTML='👥 SHADOW CLONE 👥'; banner.style.display='block'; banner.style.color='#aa88ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'👥 SHADOW CLONE!',x:width/2-60,y:height/2-40,l:1.5,c:'#aa88ff',size:24}); startEventCooldown(); checkAchievements(); showNotification('👥 Shadow Clone event started! Clone fights for you!', 'event'); }
function triggerLightningStorm(){ if(!canTriggerEvent()) return; activeEvent='lightning'; eventTimer=Date.now()+8000; lightningStormCount++; totalEventsTriggered++; sfxLightning(); lightningStormActive=true; lightningStormTimer=Date.now()+8000; const banner=document.getElementById('event-banner'); banner.innerHTML='⚡ LIGHTNING STORM ⚡'; banner.style.display='block'; banner.style.color='#ffff00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'⚡ LIGHTNING STORM!',x:width/2-60,y:height/2-40,l:1.5,c:'#ffff00',size:24}); startEventCooldown(); checkAchievements(); showNotification('⚡ Lightning Storm event started! Lightning strikes enemies!', 'event'); }
function triggerLuckyDraw(){ if(!canTriggerEvent()) return; activeEvent='lucky'; eventTimer=Date.now()+3000; luckyDrawCount++; totalEventsTriggered++; sfxLucky(); const banner=document.getElementById('event-banner'); banner.innerHTML='🍀 LUCKY DRAW 🍀'; banner.style.display='block'; banner.style.color='#88ff88'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); const shopItems = ['fire', 'dmg', 'shield', 'drone', 'heal', 'spread', 'laser', 'bomb']; const randomItem = shopItems[Math.floor(Math.random() * shopItems.length)]; buyUpgrade(randomItem, 1); if(player && player.x && player.y) floats.push({txt:'🍀 LUCKY DRAW! Free upgrade!',x:width/2-70,y:height/2-40,l:1.5,c:'#88ff88',size:24}); startEventCooldown(); checkAchievements(); showNotification('🍀 Lucky Draw event! Free upgrade!', 'event'); }
function triggerMysteryBox(){ if(!canTriggerEvent()) return; activeEvent='mystery'; eventTimer=Date.now()+3000; mysteryBoxCount++; totalEventsTriggered++; sfxMystery(); const banner=document.getElementById('event-banner'); banner.innerHTML='🔮 MYSTERY BOX 🔮'; banner.style.display='block'; banner.style.color='#ff88ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); const rewards = [ () => { let reward = 500 * gemMultiplier; gemstones += reward; saveGemstones(); return `${reward} GEMSTONES!`; }, () => { let reward = 10000; totalCoins += reward; localStorage.setItem('totalCoins', totalCoins); return `${reward} CREDITS!`; }, () => { bombCount += 5; localStorage.setItem('bombCount', bombCount); return "5 BOMBS!"; }, () => { damageLevel += 2; localStorage.setItem('dmgLevel', damageLevel); return "PERMANENT DAMAGE +2!"; }, () => { fireLevel += 3; localStorage.setItem('fireLevel', fireLevel); return "PERMANENT FIRE RATE +3!"; } ]; const reward = rewards[Math.floor(Math.random() * rewards.length)](); if(player && player.x && player.y) floats.push({txt:`🔮 MYSTERY BOX! ${reward}`,x:width/2-80,y:height/2-40,l:1.5,c:'#ff88ff',size:24}); startEventCooldown(); checkAchievements(); showNotification('🔮 Mystery Box event! Random reward!', 'event'); }
function triggerDoomsDay(){ if(!canTriggerEvent()) return; activeEvent='doom'; eventTimer=Date.now()+15000; doomsDayCount++; totalEventsTriggered++; sfxDoom(); doomsDayActive=true; const banner=document.getElementById('event-banner'); banner.innerHTML='💀 DOOM\'S DAY 💀'; banner.style.display='block'; banner.style.color='#440000'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💀 DOOM\'S DAY!',x:width/2-60,y:height/2-40,l:1.5,c:'#440000',size:24}); startEventCooldown(); checkAchievements(); showNotification('💀 Doom\'s Day event started! Enemies weakened!', 'event'); }
function triggerRoyalBlessing(){ if(!canTriggerEvent()) return; activeEvent='royal'; eventTimer=Date.now()+20000; royalBlessingCount++; totalEventsTriggered++; sfxRoyal(); royalBlessingActive=true; royalBlessingTimer=Date.now()+20000; const banner=document.getElementById('event-banner'); banner.innerHTML='👑 ROYAL BLESSING 👑'; banner.style.display='block'; banner.style.color='#ffdd00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'👑 ROYAL BLESSING!',x:width/2-60,y:height/2-40,l:1.5,c:'#ffdd00',size:24}); startEventCooldown(); checkAchievements(); showNotification('👑 Royal Blessing event started! 3x everything!', 'event'); }
function triggerStarfall(){ if(!canTriggerEvent()) return; activeEvent='starfall'; eventTimer=Date.now()+10000; starfallCount++; totalEventsTriggered++; sfxStarfall(); starfallActive=true; starfallTimer=Date.now()+10000; starfallStars = []; const banner=document.getElementById('event-banner'); banner.innerHTML='🌟 STARFALL 🌟'; banner.style.display='block'; banner.style.color='#ffffaa'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); for(let i=0;i<20;i++){ starfallStars.push({x:Math.random()*width, y:-20, size:8+Math.random()*8, value:50+Math.floor(Math.random()*100)}); } if(player && player.x && player.y) floats.push({txt:'🌟 STARFALL!',x:width/2-60,y:height/2-40,l:1.5,c:'#ffffaa',size:24}); showNotification('🌟 STARFALL event started! Catch falling stars!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerInferno(){ if(!canTriggerEvent()) return; activeEvent='inferno'; eventTimer=Date.now()+8000; infernoCount++; totalEventsTriggered++; sfxInferno(); infernoActive=true; infernoTimer=Date.now()+8000; const banner=document.getElementById('event-banner'); banner.innerHTML='🔥 INFERNO 🔥'; banner.style.display='block'; banner.style.color='#ff4400'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🔥 INFERNO!',x:width/2-60,y:height/2-40,l:1.5,c:'#ff4400',size:24}); showNotification('🔥 INFERNO event started! Flames will burn enemies!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerChainLightning(){ if(!canTriggerEvent()) return; activeEvent='chain'; eventTimer=Date.now()+10000; chainLightningCount++; totalEventsTriggered++; sfxChain(); chainLightningActive=true; chainLightningTimer=Date.now()+10000; const banner=document.getElementById('event-banner'); banner.innerHTML='⚡ CHAIN LIGHTNING ⚡'; banner.style.display='block'; banner.style.color='#ffff00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'⚡ CHAIN LIGHTNING!',x:width/2-70,y:height/2-40,l:1.5,c:'#ffff00',size:24}); showNotification('⚡ CHAIN LIGHTNING event started! Lightning will chain between enemies!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerBarrier(){ if(!canTriggerEvent()) return; activeEvent='barrier'; eventTimer=Date.now()+10000; barrierCount++; totalEventsTriggered++; sfxBarrier(); barrierActive=true; barrierTimer=Date.now()+10000; barrierHp = 500; const banner=document.getElementById('event-banner'); banner.innerHTML='🛡️ BARRIER 🛡️'; banner.style.display='block'; banner.style.color='#88aaff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🛡️ BARRIER!',x:width/2-60,y:height/2-40,l:1.5,c:'#88aaff',size:24}); showNotification('🛡️ BARRIER event started! Damage absorption active!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerSoulReaper(){ if(!canTriggerEvent()) return; activeEvent='soulreaper'; eventTimer=Date.now()+12000; soulReaperCount++; totalEventsTriggered++; sfxSoulReaper(); soulReaperActive=true; soulReaperTimer=Date.now()+12000; soulReaperSoul = {x:width/2, y:-100, hp:500, maxHp:500}; const banner=document.getElementById('event-banner'); banner.innerHTML='💀 SOUL REAPER 💀'; banner.style.display='block'; banner.style.color='#8800aa'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},4000); if(player && player.x && player.y) floats.push({txt:'💀 SOUL REAPER!',x:width/2-70,y:height/2-40,l:1.5,c:'#8800aa',size:28}); showNotification('💀 SOUL REAPER event started! A powerful reaper appears!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerGambler(){ if(!canTriggerEvent()) return; activeEvent='gambler'; eventTimer=Date.now()+5000; gamblerCount++; totalEventsTriggered++; sfxGambler(); gamblerActive=true; const banner=document.getElementById('event-banner'); banner.innerHTML='🎲 GAMBLER 🎲'; banner.style.display='block'; banner.style.color='#ffaa00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); const outcome = Math.random(); if(outcome < 0.3){ totalCoins = Math.floor(totalCoins * 0.5); localStorage.setItem('totalCoins', totalCoins); if(player && player.x && player.y) floats.push({txt:'🎲 GAMBLER: YOU LOST 50% CREDITS!',x:width/2-100,y:height/2-40,l:1.5,c:'#ff0000',size:22}); showNotification('🎲 GAMBLER: You lost 50% of your credits!', 'warning'); } else if(outcome < 0.7){ let win = 5000; totalCoins += win; localStorage.setItem('totalCoins', totalCoins); if(player && player.x && player.y) floats.push({txt:`🎲 GAMBLER: +${win} CREDITS!`,x:width/2-100,y:height/2-40,l:1.5,c:'#00ffaa',size:22}); showNotification(`🎲 GAMBLER: You won ${win} CREDITS!`, 'success'); } else { let win = 20000; totalCoins += win; localStorage.setItem('totalCoins', totalCoins); if(player && player.x && player.y) floats.push({txt:`🎲 GAMBLER: +${win} CREDITS! JACKPOT!`,x:width/2-110,y:height/2-40,l:1.5,c:'#ffaa00',size:24}); showNotification(`🎲 GAMBLER: JACKPOT! +${win} CREDITS!`, 'success'); } startEventCooldown(); checkAchievements(); }
function triggerVortex(){ if(!canTriggerEvent()) return; activeEvent='vortex'; eventTimer=Date.now()+10000; vortexCount++; totalEventsTriggered++; sfxVortex(); vortexActive=true; vortexTimer=Date.now()+10000; vortexCenter={x:width/2, y:height/2}; const banner=document.getElementById('event-banner'); banner.innerHTML='🌀 VORTEX 🌀'; banner.style.display='block'; banner.style.color='#44aaff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🌀 VORTEX!',x:width/2-60,y:height/2-40,l:1.5,c:'#44aaff',size:24}); showNotification('🌀 VORTEX event started! Enemies will be pulled into the vortex!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerKingsBlessing(){ if(!canTriggerEvent()) return; activeEvent='kings'; eventTimer=Date.now()+15000; kingsBlessingCount++; totalEventsTriggered++; sfxKings(); kingsBlessingActive=true; kingsBlessingTimer=Date.now()+15000; const banner=document.getElementById('event-banner'); banner.innerHTML='👑 KING\'S BLESSING 👑'; banner.style.display='block'; banner.style.color='#ffdd00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},4000); if(player && player.x && player.y) floats.push({txt:'👑 KING\'S BLESSING! 5x EVERYTHING!',x:width/2-100,y:height/2-40,l:1.5,c:'#ffdd00',size:26}); showNotification('👑 KING\'S BLESSING event started! 5x points and credits!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerPrism(){ if(!canTriggerEvent()) return; activeEvent='prism'; eventTimer=Date.now()+12000; prismCount++; totalEventsTriggered++; sfxPrism(); prismActive=true; prismTimer=Date.now()+12000; const banner=document.getElementById('event-banner'); banner.innerHTML='🌈 PRISM 🌈'; banner.style.display='block'; banner.style.color='#ff88ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🌈 PRISM! Bullets split into colors!',x:width/2-80,y:height/2-40,l:1.5,c:'#ff88ff',size:24}); showNotification('🌈 PRISM event started! Bullets split into rainbow colors!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerAbyss(){ if(!canTriggerEvent()) return; activeEvent='abyss'; eventTimer=Date.now()+8000; abyssCount++; totalEventsTriggered++; sfxAbyss(); abyssActive=true; abyssTimer=Date.now()+8000; abyssCenter={x:width/2, y:height/2}; const banner=document.getElementById('event-banner'); banner.innerHTML='🕳️ ABYSS 🕳️'; banner.style.display='block'; banner.style.color='#4400aa'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🕳️ ABYSS!',x:width/2-60,y:height/2-40,l:1.5,c:'#4400aa',size:24}); showNotification('🕳️ ABYSS event started! Enemies will be swallowed by the abyss!', 'event'); startEventCooldown(); checkAchievements(); }

function checkEventTrigger(){
    if(!canTriggerEvent()) return;
    if(!endlessMode){
        if(wave>0 && wave%7===0 && !activeEvent && stableCycleCount < Math.floor(wave/7)){
            triggerStableCycle(); return;
        }
    } else {
        if(wave>0 && wave%7===0 && !activeEvent){ triggerStableCycle(); return; }
    }
    if(!endlessMode){
        if(wave>0 && wave%40===0 && !activeEvent && cosmicCollapseCount < Math.floor(wave/40)){
            triggerCosmicCollapse(); return;
        }
    } else {
        if(wave>0 && wave%40===0 && !activeEvent){ triggerCosmicCollapse(); return; }
    }
    if(wave>=22 && !activeEvent && Math.random()<0.0006){ triggerDoppelganger(); return; }
    if(wave>=30 && !activeEvent && Math.random()<0.00001){ triggerSupernova(); return; }
    if(wave>=10 && !activeEvent && Math.random()<0.0012){ triggerStarfall(); return; }
    if(wave>=15 && !activeEvent && Math.random()<0.0008){ triggerInferno(); return; }
    if(wave>=18 && !activeEvent && Math.random()<0.0006){ triggerChainLightning(); return; }
    if(wave>=12 && !activeEvent && Math.random()<0.002){ triggerBarrier(); return; }
    if(wave>=25 && !activeEvent && Math.random()<0.0002){ triggerSoulReaper(); return; }
    if(wave>=20 && !activeEvent && Math.random()<0.0001){ triggerGambler(); return; }
    if(wave>=22 && !activeEvent && Math.random()<0.0004){ triggerVortex(); return; }
    if(wave>=30 && !activeEvent && Math.random()<0.00003){ triggerKingsBlessing(); return; }
    if(wave>=18 && !activeEvent && Math.random()<0.0005){ triggerPrism(); return; }
    if(wave>=35 && !activeEvent && Math.random()<0.00002){ triggerAbyss(); return; }
    if(wave>=12 && !activeEvent && Math.random()<0.001){ triggerLightningStorm(); return; }
    if(wave>=15 && !activeEvent && Math.random()<0.0005){ triggerLuckyDraw(); return; }
    if(wave>=18 && !activeEvent && Math.random()<0.0003){ triggerMysteryBox(); return; }
    if(wave>=20 && !activeEvent && Math.random()<0.0001){ triggerDoomsDay(); return; }
    if(wave>=25 && !activeEvent && Math.random()<0.00005){ triggerRoyalBlessing(); return; }
    if(wave>=15 && !activeEvent && Math.random()<0.002){ triggerFrozenTime(); return; }
    if(wave>=18 && !activeEvent && Math.random()<0.0015){ triggerCrystalRain(); return; }
    if(wave>=25 && !activeEvent && Math.random()<0.0008){ triggerShadowClone(); return; }
    if(wave>=30 && !activeEvent && Math.random()<0.0000001){ triggerDivineIntervention(); return; }
    if(wave>=25 && !activeEvent && Math.random()<0.000005){ triggerGoldRush(); return; }
    if(wave>=20 && !activeEvent && Math.random()<0.00001){ triggerBugEvent(); return; }
    if(wave>=20 && !activeEvent && Math.random()<0.00004){ triggerTimeWarp(); return; }
    if(wave>=20 && !activeEvent && Math.random()<0.00002){ triggerReapersCall(); return; }
    if(wave>=20 && !activeEvent && Math.random()<0.001){ triggerSoulHarvest(); return; }
    if(wave>=15 && !activeEvent && Math.random()<0.003){ triggerMasquerade(); return; }
    if(wave>=12 && !activeEvent && Math.random()<0.005){ triggerTidalWave(); return; }
    if(wave>=25 && !activeEvent && Math.random()<0.00005){ triggerDeathTouch(); return; }
    if(wave>=25 && !activeEvent && Math.random()<0.00008){ triggerChaosRealm(); return; }
    if(wave>=15 && !activeEvent && Math.random()<0.00003){ triggerPrimordialRage(); return; }
    if(wave>=4 && wave%4===0 && Math.random()<0.6 && !activeEvent){ triggerMeteorShower(); return; }
    if(wave>=10 && !activeEvent && Math.random()<0.03){ triggerDimensionRift(); return; }
    if(wave>=5 && wave%5===0 && !activeEvent && Math.random()<0.005){ triggerApocalypseMode(); return; }
    if(wave>=15 && !activeEvent && Math.random()<0.003){ triggerVoidMode(); return; }
    if(!endlessMode && wave>=20 && !activeEvent && !guardian && Math.random()<0.001){ triggerGuardian(); return; }
    if(endlessMode && wave>=20 && !activeEvent && !guardian && Math.random()<0.0005){ triggerGuardian(); return; }
}

function activateSynapse(){
    if(synapseActive) return;
    synapseActive=true; synapseTimer=Date.now()+10000; totalSynapseActivations++; sfxSynapse();
    const banner=document.getElementById('event-banner');
    banner.innerHTML='🌀 SYNAPSE MODE 🌀'; banner.style.display='block'; banner.style.color='#ff66ff';
    setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},2500);
    if(player && player.x && player.y) floats.push({txt:'🌀 SYNAPSE MODE!',x:width/2-60,y:height/2-40,l:1.5,c:'#ff66ff',size:24});
    checkAchievements();
    showNotification('🌀 Synapse Mode activated! Time slows!', 'event');
}

function startWave(n){
    wave=n;waveKills=0;waveNoDamage=true;waveTriggered=false;
    if(endlessMode) waveKillGoal = 5 + wave * 2;
    else waveKillGoal = 5 + wave * 3;
    const banner=document.getElementById('wave-banner');
    banner.innerHTML=`WAVE ${n}`; banner.style.display='block'; waveBannerTimer=80; sfxWave();
    document.getElementById('wave-val').innerText=wave;
    if(player && player.x && player.y) floats.push({txt:'WAVE '+n,x:width/2-35,y:height/2-40,l:1.5,c:'#00d2ff',size:22});
    checkAchievements();
    if(!endlessMode && wave >= 100 && !ascendTriggered){
        ascendTriggered=true; gameState='ASCEND';
        document.getElementById('ascend-wave').innerText=wave;
        document.getElementById('ascend-screen').style.display='flex';
        document.getElementById('pause-btn').style.display='none';
    }
}

function continueEndless(){
    endlessMode=true; ascendTriggered=false;
    document.getElementById('ascend-screen').style.display='none';
    gameState='PLAYING'; document.getElementById('pause-btn').style.display='block';
    showAchievementPopup('🌟 ENDLESS ASCENSION', 'You have entered Endless Mode!');
    showNotification('🌟 Endless Mode unlocked! Infinite progression!', 'success');
}

// SHADOW CLONE CLASS
class ShadowClone {
    constructor(){
        this.x = player.x + 50;
        this.y = player.y;
        this.r = 25;
        this.lastShot = 0;
    }
    update(){
        if(player){
            this.x = player.x + 50;
            this.y = player.y;
        }
        if(Date.now() - this.lastShot > 200){
            this.lastShot = Date.now();
            let power = damageLevel * 0.8;
            bullets.push(new Bullet(this.x, this.y-20, power, '#aa88ff'));
        }
    }
    draw(){
        ctx.save(); ctx.translate(this.x, this.y);
        ctx.fillStyle = '#aa88ff'; ctx.shadowBlur = 15;
        ctx.beginPath();
        ctx.moveTo(0,-25);ctx.lineTo(20,15);ctx.lineTo(8,15);ctx.lineTo(8,25);
        ctx.lineTo(-8,25);ctx.lineTo(-8,15);ctx.lineTo(-20,15);ctx.closePath();ctx.fill();
        ctx.fillStyle = 'rgba(170,136,255,0.5)';
        ctx.beginPath();ctx.ellipse(0,-5,4,10,0,0,Math.PI*2);ctx.fill();
        ctx.restore();
    }
}

// CRYSTAL CLASS
class Crystal {
    constructor(x,y){
        this.x=x;this.y=y;this.r=8;this.vy=2;
    }
    update(){this.y+=this.vy;}
    draw(){
        ctx.fillStyle='#88ffaa';ctx.shadowBlur=8;
        ctx.beginPath();ctx.rect(this.x-4,this.y-4,8,8);ctx.fill();
        ctx.fillStyle='#fff';ctx.font='bold 12px Segoe UI';ctx.fillText('💎',this.x-6,this.y+5);
    }
}

// STARFALL STAR CLASS
class StarfallStar {
    constructor(x,y,size,value){
        this.x=x;this.y=y;this.size=size;this.value=value;this.vy=2;
    }
    update(){this.y+=this.vy;}
    draw(){
        ctx.fillStyle='#ffffaa';ctx.shadowBlur=8;
        ctx.beginPath();ctx.rect(this.x-this.size/2,this.y-this.size/2,this.size,this.size);ctx.fill();
        ctx.fillStyle='#fff';ctx.font='bold 12px Segoe UI';ctx.fillText('⭐',this.x-6,this.y+5);
    }
}

// SOUL CLASS
class Soul{
    constructor(x,y){
        this.x=x;this.y=y;this.r=8;this.speed=3;this.angle=Math.random()*Math.PI*2;
        this.vx=Math.cos(this.angle)*this.speed;this.vy=Math.sin(this.angle)*this.speed;
        this.life=1;
    }
    update(){
        this.x+=this.vx;this.y+=this.vy;
        this.life-=0.01;
    }
    draw(){
        ctx.fillStyle=`rgba(150,0,150,${this.life})`;ctx.shadowBlur=8;
        ctx.beginPath();ctx.arc(this.x,this.y,this.r,0,Math.PI*2);ctx.fill();
        ctx.fillStyle='#fff';ctx.font='bold 12px Segoe UI';ctx.fillText('💀',this.x-6,this.y+5);
    }
}

function useBomb(){
    if(bombCount<=0||gameState!=='PLAYING')return;
    bombCount--;sfxBomb();totalBombsUsed++;
    localStorage.setItem('bombCount',bombCount);
    for(let i=0;i<enemies.length;i++){
        let e=enemies[i];
        if(e.isBoss){
            e.hp-=100;
            if(e.hp<=0){ enemies.splice(i,1); boss=null; i--; }
        } else {
            let pointBonus = 35*combo;
            if(cosmicCollapseActive) pointBonus*=10;
            if(goldRushActive) pointBonus*=10;
            if(stableCycleActive) pointBonus*=3;
            if(royalBlessingActive) pointBonus*=3;
            if(kingsBlessingActive) pointBonus*=5;
            pointBonus *= skinCreditMultiplier;
            score+=pointBonus;kills++;
            for(let k=0;k<8;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*10,(Math.random()-0.5)*10,e.color,0.8));
            enemies.splice(i,1); i--;
        }
    }
    if(guardian){
        guardian.hp-=500;
        if(guardian.hp<=0){
            guardian=null; activeEvent=null; 
            let pointBonus = 50000;
            if(goldRushActive) pointBonus*=10;
            if(stableCycleActive) pointBonus*=3;
            if(royalBlessingActive) pointBonus*=3;
            if(kingsBlessingActive) pointBonus*=5;
            pointBonus *= skinCreditMultiplier;
            score+=pointBonus; totalCoins+=5000;
            if(!skinUnlocked){ skinUnlocked=true; localStorage.setItem('skinUnlocked','true'); showAchievementPopup('👑 GUARDIAN DEFEATED', 'Golden skin unlocked!'); }
            let count=parseInt(localStorage.getItem('guardianDefeatedCount')||0)+1; localStorage.setItem('guardianDefeatedCount',count);
            guardianDefeated=true; checkAchievements();
        }
    }
    for(let i=0;i<meteors.length;i++){
        let pointBonus = 200;
        if(goldRushActive) pointBonus*=10;
        if(bugEventActive) pointBonus*=5;
        if(stableCycleActive) pointBonus*=3;
        if(royalBlessingActive) pointBonus*=3;
        if(kingsBlessingActive) pointBonus*=5;
        pointBonus *= skinCreditMultiplier;
        score+=pointBonus; totalMeteorsDestroyed++;
        for(let k=0;k<5;k++) particles.push(new Particle(meteors[i].x,meteors[i].y,(Math.random()-0.5)*8,(Math.random()-0.5)*8,'#ff8844',0.8));
        meteors.splice(i,1); i--;
    }
    eBullets.length=0; shake=25;
    if(player && player.x && player.y) floats.push({txt:'💣 BOMB!',x:width/2-40,y:height/2,l:1.5,c:'#ff8800',size:26});
    if(player && player.x && player.y) for(let i=0;i<35;i++){const a=i*(Math.PI*2/35);particles.push(new Particle(player.x,player.y,Math.cos(a)*12,Math.sin(a)*12,'#ff8800',0.9));}
    checkAchievements();
    updateDailyMissionProgress('bomb', 1);
}

// SHOP FUNCTIONS
function buyUpgrade(type, amount=1){
    initAudio();
    const prices={fire:150,dmg:200,shield:250,drone:500,heal:100,spread:350,laser:600,bomb:300};
    if(type==='shield'&&hasShieldUpgrade) return;
    if(type==='spread'&&hasSpreadShot) return;
    if(type==='laser'&&hasLaser) return;
    let totalCost=0;
    if(type==='fire') totalCost=prices.fire*amount;
    else if(type==='dmg') totalCost=prices.dmg*amount;
    else if(type==='drone') totalCost=prices.drone*amount;
    else if(type==='bomb') totalCost=prices.bomb*amount;
    else totalCost=prices[type];
    if(totalCoins<totalCost) return;
    totalCoins-=totalCost;
    totalUpgradesBought+=amount;
    if(type==='fire'){fireLevel+=amount;localStorage.setItem('fireLevel',fireLevel);}
    if(type==='dmg'){damageLevel+=amount;localStorage.setItem('dmgLevel',damageLevel);}
    if(type==='shield'){hasShieldUpgrade=true;localStorage.setItem('hasShieldUpgrade',true);}
    if(type==='drone'){droneCount=Math.min(400, droneCount+amount);localStorage.setItem('droneCount',droneCount);}
    if(type==='heal'){health=Math.min(maxHealth,health+25);}
    if(type==='spread'){hasSpreadShot=true;localStorage.setItem('hasSpreadShot',true);}
    if(type==='laser'){hasLaser=true;localStorage.setItem('hasLaser',true);}
    if(type==='bomb'){bombCount+=amount;localStorage.setItem('bombCount',bombCount);}
    localStorage.setItem('totalCoins',totalCoins);
    localStorage.setItem('totalUpgradesBought',totalUpgradesBought);
    sfxCoin();
    updateShopUI();
    checkAchievements();
    updateDailyMissionProgress('upgrade', amount);
}
function setQuantity(type, qty){ selectedQuantities[type]=Math.max(1, Math.min(99, qty)); updateShopUI(); }
function maxOutUpgrade(type,pricePer){
    let canBuy=Math.floor(totalCoins/pricePer);
    if(type==='drone') canBuy=Math.min(canBuy, 400-droneCount);
    if(canBuy>0) buyUpgrade(type,canBuy);
}
function updateShopUI(){
    const container=document.getElementById('shop-grid-container');
    if(!container) return;
    const items=[
        {id:'fire', name:'🔥 FIRE RATE', level:fireLevel, price:150, type:'fire', infinite:true},
        {id:'dmg', name:'💥 DAMAGE', level:damageLevel, price:200, type:'dmg', infinite:true},
        {id:'shield', name:'🛡️ SHIELD', owned:hasShieldUpgrade, price:250, type:'shield'},
        {id:'drone', name:'🤖 DRONE', level:droneCount, price:500, type:'drone', infinite:true, maxLimit:400},
        {id:'heal', name:'❤️ REPAIR', price:100, type:'heal'},
        {id:'spread', name:'🎯 SPREAD', owned:hasSpreadShot, price:350, type:'spread'},
        {id:'laser', name:'⚡ LASER', owned:hasLaser, price:600, type:'laser'},
        {id:'bomb', name:'💣 BOMB', level:bombCount, price:300, type:'bomb', infinite:true}
    ];
    container.innerHTML='';
    for(const it of items){
        const card=document.createElement('div');
        card.className='card tooltip';
        if(it.owned===true) card.classList.add('cant-afford');
        let content=`<div style="font-size:14px;">${it.name}</div>`;
        if(it.level!==undefined) content+=`<div style="font-size:9px;">LVL: ${it.level}${it.maxLimit?('/'+it.maxLimit):''}</div>`;
        if(it.owned!==undefined) content+=`<div style="font-size:9px;">${it.owned?'✅':'❌'}</div>`;
        content+=`<div style="color:gold;">${formatNumber(it.price)}c</div>`;
        if(it.infinite){
            let qty=selectedQuantities[it.type]||1;
            content+=`<div class="quantity-selector">
                <button class="qty-btn" onclick="event.stopPropagation();setQuantity('${it.type}',1)">1</button>
                <button class="qty-btn" onclick="event.stopPropagation();setQuantity('${it.type}',5)">5</button>
                <button class="qty-btn" onclick="event.stopPropagation();setQuantity('${it.type}',10)">10</button>
                <button class="qty-btn" onclick="event.stopPropagation();maxOutUpgrade('${it.type}',${it.price})">MAX</button>
                <span style="margin-right:5px;">x${qty}</span>
            </div>`;
        }
        let tooltipText = it.type==='fire' ? 'מגדיל את מהירות הירי' : it.type==='dmg' ? 'מגדיל את הנזק' : it.type==='shield' ? 'מוסיף מגן סביב החללית' : it.type==='drone' ? 'מוסיף רחפן שיורה אוטומטית' : it.type==='heal' ? 'מרפא 25 נקודות חיים' : it.type==='spread' ? 'מוסיף כדור נוסף לכל ירייה' : it.type==='laser' ? 'מוסיף לייזר חזק' : 'מוסיף פצצות לניקוי מסך';
        card.innerHTML = content + `<div class="tooltip-text">${tooltipText}</div>`;
        if(!(it.owned===true)){
            let qty=selectedQuantities[it.type]||1;
            let atMax=(it.type==='drone' && droneCount+qty>400);
            if(!atMax) card.onclick=()=>buyUpgrade(it.type, selectedQuantities[it.type]||1);
            else card.onclick=null;
        }
        container.appendChild(card);
    }
    document.getElementById('shop-money').innerHTML="CREDITS: "+formatNumber(totalCoins);
    document.getElementById('guardian-count').innerText = guardianAngelCount;
}
function openShop(){ 
    document.getElementById('start-screen').style.display='none'; 
    document.getElementById('shop-screen').style.display='flex';
    updateShopUI(); 
}
function closeShop(){ 
    document.getElementById('shop-screen').style.display='none'; 
    document.getElementById('start-screen').style.display='flex';
}

// PLAYER CLASS
class Player{
    constructor(){this.x=width/2;this.y=height-120;this.tx=this.x;this.ty=this.y;this.r=25;this.invincibleTimer=0;this.deathProtection=false;}
    update(){
        if(bugEventGlitch){
            this.tx = width/2 + (Math.random() - 0.5) * 200;
            this.ty = height/2 + (Math.random() - 0.5) * 200;
        }
        this.x+=(this.tx-this.x)*0.2;this.y+=(this.ty-this.y)*0.2;
        this.x=Math.max(this.r,Math.min(width-this.r,this.x));
        this.y=Math.max(this.r,Math.min(height-this.r,this.y));
        if(this.invincibleTimer>0)this.invincibleTimer--;
        const tc=isOD?'#ff00ff':synapseActive?'#ff66ff':voidActive?'#aa66ff':primordialRageActive?'#ff0000':chaosRealmActive?`hsl(${chaosHue},100%,60%)`:apocalypseActive?'#ff0000':riftActive?'#aa66ff':bugEventActive?'#00ff00':stableCycleActive?'#88aaff':lightningStormActive?'#ffff00':royalBlessingActive?'#ffdd00':kingsBlessingActive?'#ffdd00':prismActive?'#ff88ff':doppelgangerActive?'#ff88ff':activePowerUps.rapidfire?'#00ffff':'#00d2ff';
        if(Math.random()>0.5) particles.push(new Particle(this.x,this.y+28,(Math.random()-0.5)*2,Math.random()*4+2,tc,0.7));
    }
    draw(){
        ctx.save();
        if(settings.shake && shake>0)ctx.translate((Math.random()-0.5)*shake,(Math.random()-0.5)*shake);
        if(bugEventGlitch){
            ctx.translate((Math.random()-0.5)*10, (Math.random()-0.5)*10);
        }
        ctx.translate(this.x,this.y);
        let maxDrones = Math.min(400, droneCount);
        for(let i=0;i<maxDrones;i++){
            const a=(Date.now()/400)+(i*Math.PI*2/maxDrones);
            const dx=Math.cos(a)*55,dy=Math.sin(a)*45;
            ctx.fillStyle='#00ffaa';ctx.shadowBlur=8;
            ctx.fillRect(dx-5,dy-5,10,10);
            ctx.beginPath();ctx.moveTo(0,0);ctx.lineTo(dx,dy);ctx.stroke();
        }
        if(hasShieldUpgrade){
            ctx.strokeStyle=`rgba(0,255,204,${0.3+Math.sin(Date.now()/400)*0.2})`;
            ctx.lineWidth=1.5;ctx.setLineDash([3,6]);
            ctx.beginPath();ctx.arc(0,0,55,0,Math.PI*2);ctx.stroke();ctx.setLineDash([]);
        }
        if(this.invincibleTimer>0&&Math.floor(this.invincibleTimer/5)%2===0){ctx.restore();return;}
        let col=isOD?'#ff00ff':synapseActive?'#ff66ff':voidActive?'#aa66ff':primordialRageActive?'#ff0000':chaosRealmActive?`hsl(${chaosHue},100%,60%)`:apocalypseActive?'#ff0000':riftActive?'#aa66ff':bugEventActive?'#00ff00':stableCycleActive?'#88aaff':lightningStormActive?'#ffff00':royalBlessingActive?'#ffdd00':kingsBlessingActive?'#ffdd00':prismActive?'#ff88ff':doppelgangerActive?'#ff88ff':activePowerUps.doubleDmg?'#ff8800':'#00d2ff';
        
        if(currentSkin === 'gold' && !isOD && !synapseActive && !voidActive && !apocalypseActive && !riftActive && !primordialRageActive && !chaosRealmActive && !bugEventActive && !stableCycleActive && !lightningStormActive && !royalBlessingActive && !kingsBlessingActive && !prismActive && !doppelgangerActive) col='#ffd700';
        if(currentSkin === 'blue' && !isOD && !synapseActive && !voidActive && !apocalypseActive && !riftActive && !primordialRageActive && !chaosRealmActive && !bugEventActive && !stableCycleActive && !lightningStormActive && !royalBlessingActive && !kingsBlessingActive && !prismActive && !doppelgangerActive && currentSkin !== 'gold') col='#4287f5';
        if(currentSkin === 'purple' && !isOD && !synapseActive && !voidActive && !apocalypseActive && !riftActive && !primordialRageActive && !chaosRealmActive && !bugEventActive && !stableCycleActive && !lightningStormActive && !royalBlessingActive && !kingsBlessingActive && !prismActive && !doppelgangerActive && currentSkin !== 'gold' && currentSkin !== 'blue') col='#aa44ff';
        if(currentSkin === 'rainbow' && !isOD && !synapseActive && !voidActive && !apocalypseActive && !riftActive && !primordialRageActive && !chaosRealmActive && !bugEventActive && !stableCycleActive && !lightningStormActive && !royalBlessingActive && !kingsBlessingActive && !prismActive && !doppelgangerActive && currentSkin !== 'gold' && currentSkin !== 'blue' && currentSkin !== 'purple') col=`hsl(${Date.now()/10 % 360},100%,60%)`;
        if(currentSkin === 'ultra' && !isOD && !synapseActive && !voidActive && !apocalypseActive && !riftActive && !primordialRageActive && !chaosRealmActive && !bugEventActive && !stableCycleActive && !lightningStormActive && !royalBlessingActive && !kingsBlessingActive && !prismActive && !doppelgangerActive && currentSkin !== 'gold' && currentSkin !== 'blue' && currentSkin !== 'purple' && currentSkin !== 'rainbow') col='#f5e642';
        if(currentSkin === 'legend' && !isOD && !synapseActive && !voidActive && !apocalypseActive && !riftActive && !primordialRageActive && !chaosRealmActive && !bugEventActive && !stableCycleActive && !lightningStormActive && !royalBlessingActive && !kingsBlessingActive && !prismActive && !doppelgangerActive) col='#ff6600';
        
        ctx.fillStyle=col;ctx.shadowBlur=15;
        if(primordialRageActive) ctx.scale(2,2);
        ctx.beginPath();
        ctx.moveTo(0,-32);ctx.lineTo(28,18);ctx.lineTo(10,18);ctx.lineTo(10,32);
        ctx.lineTo(-10,32);ctx.lineTo(-10,18);ctx.lineTo(-28,18);ctx.closePath();ctx.fill();
        ctx.fillStyle='rgba(255,255,255,0.3)';
        ctx.beginPath();ctx.ellipse(0,-6,5,12,0,0,Math.PI*2);ctx.fill();
        if(currentSkin === 'gold' || currentSkin === 'ultra' || currentSkin === 'legend'){
            ctx.strokeStyle='gold';ctx.lineWidth=2;ctx.beginPath();ctx.arc(0,0,38,0,Math.PI*2);ctx.stroke();
        }
        if(primordialRageActive) ctx.scale(0.5,0.5);
        ctx.restore();
    }
}

// ENEMY CLASS
class Enemy{
    constructor(isBoss=false){
        this.isBoss=isBoss;
        this.type=isBoss?'boss':['normal','zigzag','fast','tank'][Math.floor(Math.random()*4)];
        this.x=isBoss?width/2:Math.random()*(width-70)+35;
        this.y=isBoss?-120:-60;
        let lm = 1 + (level-1)*0.02 + (wave-1)*0.015;
        if(endlessMode && wave>100) lm *= (1 + (wave-100)*0.002);
        if(apocalypseActive) lm *= 1.5;
        if(cosmicCollapseActive) lm *= 1.3;
        if(doomsDayActive) lm *= 0.5;
        if(timeDistortionPurchased) lm *= enemySlow;
        if(isBoss){
            this.hp = Math.floor((100 + level*20) * lm);
        } else {
            switch(this.type){
                case 'tank': this.hp = 3; break;
                default: this.hp = 2; break;
            }
            this.hp = Math.floor(this.hp * lm);
        }
        this.maxHp=this.hp;
        this.r=isBoss?90:this.type==='tank'?35:25;
        let speedMulti = synapseActive ? 0.45 : (cosmicCollapseActive?1.2:(apocalypseActive?1.6:(riftActive?1.2:1)));
        if(timeWarpActive) speedMulti *= 0.2;
        if(frozenTimeActive) speedMulti = 0;
        if(doomsDayActive) speedMulti *= 0.5;
        if(timeDistortionPurchased) speedMulti *= enemySlow;
        if(endlessMode && wave>100) speedMulti *= (1 + (wave-100)*0.001);
        this.speed=isBoss?0.35*speedMulti:{normal:1.6,zigzag:1.6,fast:3.5,tank:0.8}[this.type]*speedMulti + Math.random()*0.3;
        this.color=isBoss?'#ff0044':{normal:'#ff4444',zigzag:'#ff8800',fast:'#00ffff',tank:'#aa44ff'}[this.type];
        this.rot=0;this.lastShot=0;this.zigDir=Math.random()<0.5?1:-1;this.zigTimer=0;
        
        if(masqueradeActive && !isBoss){
            this.disguise = 'item';
            this.originalColor = this.color;
            this.color = '#ffaa00';
        }
    }
    update(){
        if(!frozenTimeActive){
            this.y+=this.speed;
            this.rot+=0.04;
        }
        if(this.type==='zigzag'){this.zigTimer++;if(this.zigTimer%50===0)this.zigDir*=-1;this.x+=this.zigDir*1.8;this.x=Math.max(22,Math.min(width-22,this.x));}
        if(this.isBoss){
            this.x+=Math.sin(Date.now()/900)*1.2;
            let shotDelay = synapseActive?1000:650;
            if(apocalypseActive) shotDelay*=0.6;
            if(cosmicCollapseActive) shotDelay*=0.7;
            if(riftActive) shotDelay*=0.8;
            if(frozenTimeActive) shotDelay=999999;
            if(doomsDayActive) shotDelay*=1.5;
            if(timeDistortionPurchased) shotDelay*=1.3;
            if(Date.now()-this.lastShot>shotDelay){
                this.lastShot=Date.now();
                for(let s=-2;s<=2;s++)eBullets.push({x:this.x+s*28,y:this.y+40,vx:s*0.9,vy:4.5});
            }
        } else if(this.type!=='tank'&&Math.random()<0.0018+level*0.00012){
            eBullets.push({x:this.x,y:this.y+20,vx:0,vy:3.5+Math.random()*1.5});
        } else if(this.type==='tank'&&Math.random()<0.004){
            eBullets.push({x:this.x-12,y:this.y+22,vx:-1.2,vy:3.5});
            eBullets.push({x:this.x+12,y:this.y+22,vx:1.2,vy:3.5});
        }
    }
    draw(){
        ctx.save();ctx.translate(this.x,this.y);
        if(frozenTimeActive){
            ctx.fillStyle = '#88ccff'; ctx.shadowBlur = 10;
            ctx.beginPath(); ctx.arc(0,0,this.r+5,0,Math.PI*2); ctx.fill();
            ctx.fillStyle = '#fff'; ctx.font = 'bold 20px Segoe UI'; ctx.fillText('❄️', -10, 10);
        } else if(this.disguise === 'item'){
            ctx.fillStyle = '#ffaa00'; ctx.shadowBlur = 10;
            ctx.beginPath(); ctx.arc(0,0,15,0,Math.PI*2); ctx.fill();
            ctx.fillStyle = '#fff'; ctx.font = 'bold 20px Segoe UI'; ctx.fillText('?', -8, 8);
        } else {
            ctx.strokeStyle=this.color;ctx.shadowBlur=12;
            if(this.isBoss){
                ctx.lineWidth=3;ctx.rotate(this.rot*0.3);
                ctx.strokeRect(-60,-60,120,120);ctx.rotate(this.rot*0.2);ctx.strokeRect(-32,-32,64,64);
                ctx.fillStyle='rgba(255,0,68,0.2)';ctx.beginPath();ctx.arc(0,0,22,0,Math.PI*2);ctx.fill();
                ctx.fillStyle='#222';ctx.fillRect(-50,-90,100,8);
                ctx.fillStyle=`hsl(${(this.hp/this.maxHp)*120},100%,50%)`;ctx.fillRect(-50,-90,(this.hp/this.maxHp)*100,8);
            } else {
                ctx.lineWidth=this.type==='tank'?3.5:2;ctx.rotate(this.rot);
                if(this.type==='fast'){ctx.beginPath();ctx.moveTo(0,-18);ctx.lineTo(18,18);ctx.lineTo(-18,18);ctx.closePath();ctx.stroke();}
                else if(this.type==='tank'){ctx.strokeRect(-22,-22,44,44);ctx.strokeRect(-10,-10,20,20);}
                else{ctx.strokeRect(-14,-14,28,28);}
                if(this.hp<this.maxHp){
                    ctx.restore();ctx.save();ctx.translate(this.x,this.y);
                    ctx.fillStyle='#222';ctx.fillRect(-15,-32,30,5);
                    ctx.fillStyle=this.color;ctx.fillRect(-15,-32,(this.hp/this.maxHp)*30,5);
                }
            }
        }
        ctx.restore();
    }
}

// BULLET CLASS
class Bullet{
    constructor(x,y,power,color,angle=-Math.PI/2,isLaser=false){
        this.x=x;this.y=y;this.p=power;this.c=color;this.angle=angle;this.isLaser=isLaser;
        let speedMulti = synapseActive ? 1.6 : (primordialRageActive?2.5:(chaosRealmActive?5:(apocalypseActive?1.4:(riftActive?1.3:1))));
        if(bugEventActive) speedMulti *= 2;
        if(stableCycleActive) speedMulti *= 1.5;
        if(frozenTimeActive) speedMulti *= 0.5;
        if(royalBlessingActive) speedMulti *= 1.5;
        if(kingsBlessingActive) speedMulti *= 2;
        if(prismActive) speedMulti *= 1.2;
        if(doppelgangerActive) speedMulti *= 1.1;
        speedMulti *= skinFireRateMultiplier;
        this.speed=(isLaser?26:16)*speedMulti;
        this.vx=Math.cos(angle)*this.speed;this.vy=Math.sin(angle)*this.speed;
        this.size=isLaser?6:3;
    }
    update(){this.x+=this.vx;this.y+=this.vy;}
    draw(){
        let bulletColor=this.c;
        if(chaosRealmActive) bulletColor = `hsl(${chaosHue + this.x + this.y}, 100%, 60%)`;
        if(bugEventActive) bulletColor = `hsl(${Date.now()/50 % 360}, 100%, 60%)`;
        if(crystalRainActive) bulletColor = `hsl(${Date.now()/30 % 360}, 100%, 60%)`;
        if(lightningStormActive) bulletColor = `#ffff00`;
        if(royalBlessingActive) bulletColor = `#ffdd00`;
        if(kingsBlessingActive) bulletColor = `#ffaa00`;
        if(prismActive) bulletColor = `hsl(${this.x + this.y}, 100%, 60%)`;
        if(doppelgangerActive) bulletColor = `#ff88ff`;
        ctx.fillStyle=bulletColor;ctx.shadowBlur=this.isLaser?12:7;
        ctx.save();ctx.translate(this.x,this.y);ctx.rotate(this.angle+Math.PI/2);
        const h=this.isLaser?22:14;ctx.fillRect(-this.size/2,-h/2,this.size,h);
        ctx.restore();
    }
}

// PARTICLE CLASS
class Particle{
    constructor(x,y,vx,vy,color,life){this.x=x;this.y=y;this.vx=vx;this.vy=vy;this.c=color;this.l=life;this.size=Math.random()*3+1.5;}
    update(){this.x+=this.vx;this.y+=this.vy;this.vy+=0.05;this.l-=0.02;}
    draw(){ctx.globalAlpha=Math.max(0,this.l);ctx.fillStyle=this.c;ctx.fillRect(this.x,this.y,this.size,this.size);ctx.globalAlpha=1;}
}

// GAME FLOW
let gameLoopRunning = false;

function startGame(){
    initAudio();
    updateSkinEffects();
    updateRankUI();
    resetDailyMissions();
    updateDailyMissionsUI();
    updateMusicBasedOnGameState(); // MUSIC
    
    gameState='PLAYING'; endlessMode=false; ascendTriggered=false;
    score=0;health=100;xp=0;level=1;odCharge=0;combo=1;kills=0;
    bossWarningTimer=0;waveBannerTimer=0;isPaused=false;
    wave=1;waveKills=0;waveTriggered=false;
    odActivations=0;bossesKilled=0;waveNoDamage=true;perfectWavesCount=0;
    totalOverdriveUses=0;totalBombsUsed=0;maxCombo=0;
    synapseActive=false;synapsePsychedeliaCount=0;synapseKills=0;
    activeEvent=null;riftActive=false;riftMultiplier=1;meteors=[];
    apocalypseActive=false;voidActive=false;guardian=null;
    cosmicCollapseActive=false;primordialRageActive=false;blackHoleActive=false;
    chaosRealmActive=false;timeWarpActive=false;goldRushActive=false;divineActive=false;bugEventActive=false;bugEventGlitch=false;
    stableCycleActive=false;tidalWaveActive=false;masqueradeActive=false;soulHarvestActive=false;soulHarvestSouls=[];
    frozenTimeActive=false;crystalRainActive=false;shadowCloneActive=false;shadowClone=null;crystals=[];
    lightningStormActive=false;doomsDayActive=false;royalBlessingActive=false;
    starfallActive=false;starfallStars=[];infernoActive=false;chainLightningActive=false;barrierActive=false;barrierHp=0;
    soulReaperActive=false;soulReaperSoul=null;gamblerActive=false;vortexActive=false;kingsBlessingActive=false;prismActive=false;abyssActive=false;
    doppelgangerActive=false;doppelgangerClone=null;supernovaActive=false;
    blackHolePieces=0; eventCooldown=0;
    startTime=Date.now();
    enemies=[];bullets=[];eBullets=[];particles=[];items=[];floats=[];boss=null;activePowerUps={};
    player=new Player();
    document.getElementById('start-screen').style.display='none';
    document.getElementById('ascend-screen').style.display='none';
    ['ui-hud','score-hud','combo-small','powerup-bar','pause-btn','game-timer','rank-badge','combo-meter'].forEach(id=>document.getElementById(id).style.display='block');
    document.getElementById('od-btn').style.display='none';
    document.getElementById('crosshair').style.display='block';
    startWave(1);
    
    if(timerInterval) clearInterval(timerInterval);
    timerInterval = setInterval(() => {
        if(gameState === 'PLAYING' && player && player.x && player.y){
            let elapsed = Math.floor((Date.now() - startTime) / 1000);
            let minutes = Math.floor(elapsed / 60);
            let seconds = elapsed % 60;
            document.getElementById('timer-display').innerText = `${minutes.toString().padStart(2,'0')}:${seconds.toString().padStart(2,'0')}`;
        }
    }, 1000);
    
    if(animationId) cancelAnimationFrame(animationId);
    gameLoopRunning = true;
    function gameLoop(){
        if(gameLoopRunning) loop();
        animationId = requestAnimationFrame(gameLoop);
    }
    gameLoop();
    
    let autoSaveCount = parseInt(localStorage.getItem('autoSaveCount')) || 0;
    autoSaveCount++;
    localStorage.setItem('autoSaveCount', autoSaveCount);
}

function togglePause(){
    if(gameState==='PLAYING'){gameState='PAUSED';isPaused=true;document.getElementById('pause-screen').style.display='flex';document.getElementById('pause-btn').innerText='▶';}
    else if(gameState==='PAUSED'){gameState='PLAYING';isPaused=false;document.getElementById('pause-screen').style.display='none';document.getElementById('pause-btn').innerText='⏸';}
}

function quitToMenu(){
    stopBackgroundMusic();
    playBackgroundMusic('menu');
    
    gameState='MENU';isPaused=false;
    gameLoopRunning = false;
    if(player) player=null;
    if(timerInterval) clearInterval(timerInterval);
    if(animationId) cancelAnimationFrame(animationId);
    ['game-over','pause-screen','ui-hud','score-hud','combo-small','od-btn','pause-btn','powerup-bar','wave-banner','boss-warning','event-banner','ascend-screen','start-screen','game-timer','rank-badge','combo-meter'].forEach(id=>{
        const el=document.getElementById(id);
        if(el) el.style.display='none';
    });
    document.getElementById('vignette').style.display='none';
    document.getElementById('crosshair').style.display='none';
    document.getElementById('main-hub').style.display='flex';
    updateHubUI();
    
    let statViewCount = parseInt(localStorage.getItem('statViewCount')) || 0;
    statViewCount++;
    localStorage.setItem('statViewCount', statViewCount);
}

function triggerReboot(){ 
    document.getElementById('game-over').style.display='none'; 
    gameState = 'LOADING';
    gameLoopRunning = false;
    if(player) player = null;
    if(timerInterval) clearInterval(timerInterval);
    if(animationId) cancelAnimationFrame(animationId);
    enemies = [];
    bullets = [];
    eBullets = [];
    particles = [];
    items = [];
    floats = [];
    meteors = [];
    crystals = [];
    soulHarvestSouls = [];
    starfallStars = [];
    shadowClone = null;
    soulReaperSoul = null;
    doppelgangerClone = null;
    boss = null;
    guardian = null;
    activePowerUps = {};
    startGame();
}

function activateOverdrive(){
    if(odCharge<100)return;initAudio();sfxOverdrive();odActivations++;totalOverdriveUses++;
    isOD=true;odTimer=Date.now()+6000;if(settings.shake) shake=18;
    document.getElementById('od-btn').style.display='none';
    if(player && player.x && player.y) floats.push({txt:'⚡ OVERDRIVE!',x:player.x-50,y:player.y-20,l:1.5,c:'#ff00ff',size:24});
    checkAchievements();
    showNotification('⚡ Overdrive activated!', 'success');
}

function gameOver(){
    if(guardianAngelCount > 0 && !guardianAngelUsed){
        guardianAngelUsed = true;
        guardianAngelCount--;
        localStorage.setItem('guardianAngelCount', guardianAngelCount);
        health = maxHealth;
        if(player) player.invincibleTimer = 180;
        showCustomAlert(`🛡️ GUARDIAN ANGEL saved you! (${guardianAngelCount}/3 remaining)`);
        showNotification(`🛡️ Guardian Angel saved you! (${guardianAngelCount}/3 left)`, 'success');
        updateShopUI();
        return;
    }
    
    gameState='GAMEOVER';
    gameLoopRunning = false;
    const earned=Math.floor(score/10);
    totalCoins+=earned;totalKills+=kills;
    if(score>hiScore)hiScore=score;
    localStorage.setItem('totalCoins',totalCoins);localStorage.setItem('hiScore',hiScore);localStorage.setItem('totalKills',totalKills);
    if(timerInterval) clearInterval(timerInterval);
    if(animationId) cancelAnimationFrame(animationId);
    ['ui-hud','score-hud','od-btn','combo-small','pause-btn','powerup-bar','boss-warning','wave-banner','event-banner','ascend-screen','game-timer','rank-badge','combo-meter'].forEach(id=>document.getElementById(id).style.display='none');
    document.getElementById('vignette').style.display='none';
    document.getElementById('game-over').style.display='flex';
    const isNew=score>=hiScore&&score>0;
    document.getElementById('final-stats').innerHTML=
        `<div>SCORE: <span style="color:#00d2ff">${formatNumber(score)}</span>${isNew?' 🏆 NEW!':''}</div>
         <div>KILLS: <span style="color:#ff4444">${formatNumber(kills)}</span></div>
         <div>WAVE: <span style="color:#00ffaa">${wave}</span></div>
         <div>RANK: <span style="color:#00ffaa">${level}</span></div>
         <div style="color:gold">CREDITS: +${formatNumber(earned)}</div>`;
}

function fireBullets(){
    if(!player || !player.x || !player.y) return;
    const isRapid=!!activePowerUps.rapidfire,isPower=!!activePowerUps.doubleDmg;
    let power = damageLevel*(isPower?2:1);
    power = applyCriticalHit(power);
    power *= skinDamageMultiplier;
    if(synapseActive) power *= 2.5;
    if(voidActive) power *= 10;
    if(primordialRageActive) power *= 20;
    if(chaosRealmActive) power *= 3;
    if(apocalypseActive) power *= 2;
    if(cosmicCollapseActive) power *= 3;
    if(riftActive) power *= riftMultiplier;
    if(goldRushActive) power *= 2;
    if(bugEventActive) power *= 5;
    if(stableCycleActive) power *= 1.5;
    if(royalBlessingActive) power *= 3;
    if(kingsBlessingActive) power *= 5;
    if(prismActive) power *= 1.5;
    let color=isOD?'#ff00ff':synapseActive?'#ff66ff':voidActive?'#aa66ff':primordialRageActive?'#ff0000':chaosRealmActive?`hsl(${chaosHue},100%,60%)`:apocalypseActive?'#ff0000':cosmicCollapseActive?'#88aaff':riftActive?'#aa66ff':bugEventActive?'#00ff00':stableCycleActive?'#88aaff':lightningStormActive?'#ffff00':royalBlessingActive?'#ffdd00':kingsBlessingActive?'#ffaa00':prismActive?'#ff88ff':doppelgangerActive?'#ff88ff':isRapid?'#00ffff':'#00d2ff';
    const px=player.x,py=player.y-28;
    if(primordialRageActive){
        for(let i=-3;i<=3;i++){
            bullets.push(new Bullet(px+i*12,py,power,color,-Math.PI/2-0.15*i));
            bullets.push(new Bullet(px+i*12,py,power,color,-Math.PI/2+0.15*i));
        }
    } else if(chaosRealmActive){
        for(let i=-2;i<=2;i++){
            let chaosColor = `hsl(${chaosHue + i*60},100%,60%)`;
            bullets.push(new Bullet(px+i*10,py,power,chaosColor,-Math.PI/2-0.2*i));
            bullets.push(new Bullet(px+i*10,py,power,chaosColor,-Math.PI/2+0.2*i));
        }
        bullets.push(new Bullet(px,py,power,color));
    } else if(prismActive){
        for(let i=-3;i<=3;i++){
            let prismColor = `hsl(${i*60},100%,60%)`;
            bullets.push(new Bullet(px+i*8,py,power,prismColor,-Math.PI/2-0.1*i));
            bullets.push(new Bullet(px+i*8,py,power,prismColor,-Math.PI/2+0.1*i));
        }
    } else {
        bullets.push(new Bullet(px,py,power,color));
        if(hasSpreadShot){bullets.push(new Bullet(px,py,power,color,-Math.PI/2-0.22));bullets.push(new Bullet(px,py,power,color,-Math.PI/2+0.22));}
        if(hasLaser&&Math.random()<0.3){sfxLaser();bullets.push(new Bullet(px,py,power*1.3,'#ff44ff',-Math.PI/2,true));}
    }
    if(isOD){bullets.push(new Bullet(px-15,py+8,power,color));bullets.push(new Bullet(px+15,py+8,power,color));}
    sfxShoot();lastFire=Date.now();
}

// MAIN LOOP (abbreviated for space - same as before but with music update)
function loop(){
    // Update event timers
    if(starfallActive && Date.now()>starfallTimer) starfallActive=false;
    if(infernoActive && Date.now()>infernoTimer) infernoActive=false;
    if(chainLightningActive && Date.now()>chainLightningTimer) chainLightningActive=false;
    if(barrierActive && Date.now()>barrierTimer) barrierActive=false;
    if(soulReaperActive && Date.now()>soulReaperTimer) soulReaperActive=false;
    if(vortexActive && Date.now()>vortexTimer) vortexActive=false;
    if(kingsBlessingActive && Date.now()>kingsBlessingTimer) kingsBlessingActive=false;
    if(prismActive && Date.now()>prismTimer) prismActive=false;
    if(abyssActive && Date.now()>abyssTimer) abyssActive=false;
    if(doppelgangerActive && Date.now()>doppelgangerTimer) { doppelgangerActive=false; doppelgangerClone=null; }
    if(supernovaActive && Date.now()>supernovaTimer) supernovaActive=false;
    
    if(stableCycleActive && Date.now()>stableCycleTimer) stableCycleActive=false;
    if(tidalWaveActive && Date.now()>tidalWaveTimer) tidalWaveActive=false;
    if(masqueradeActive && Date.now()>masqueradeTimer) masqueradeActive=false;
    if(soulHarvestActive && Date.now()>soulHarvestTimer) soulHarvestActive=false;
    if(timeWarpActive && Date.now()>timeWarpTimer) timeWarpActive=false;
    if(goldRushActive && Date.now()>goldRushTimer) goldRushActive=false;
    if(divineActive && Date.now()>divineTimer) divineActive=false;
    if(bugEventActive && Date.now()>eventTimer) { bugEventActive=false; bugEventGlitch=false; }
    if(frozenTimeActive && Date.now()>eventTimer) frozenTimeActive=false;
    if(crystalRainActive && Date.now()>eventTimer) crystalRainActive=false;
    if(shadowCloneActive && Date.now()>eventTimer) { shadowCloneActive=false; shadowClone=null; }
    if(lightningStormActive && Date.now()>eventTimer) lightningStormActive=false;
    if(doomsDayActive && Date.now()>eventTimer) doomsDayActive=false;
    if(royalBlessingActive && Date.now()>royalBlessingTimer) royalBlessingActive=false;
    if(chaosRealmActive) chaosHue = (chaosHue + 3) % 360;
    if(synapseActive && Date.now()>synapseTimer){ synapseActive=false; document.getElementById('event-banner').style.display='none'; }
    if(voidActive && Date.now()>voidTimer){ voidActive=false; document.getElementById('event-banner').style.display='none'; }
    if(primordialRageActive && Date.now()>eventTimer){ primordialRageActive=false; if(player) player.r=25; document.getElementById('event-banner').style.display='none'; }
    if(chaosRealmActive && Date.now()>eventTimer){ chaosRealmActive=false; document.getElementById('event-banner').style.display='none'; }
    if(cosmicCollapseActive && Date.now()>eventTimer){ cosmicCollapseActive=false; document.getElementById('event-banner').style.display='none'; }
    if(blackHoleActive && Date.now()>eventTimer){ blackHoleActive=false; document.getElementById('event-banner').style.display='none'; }
    if(activeEvent && Date.now()>eventTimer){
        activeEvent=null; riftActive=false; apocalypseActive=false; cosmicCollapseActive=false;
        primordialRageActive=false; blackHoleActive=false; chaosRealmActive=false; timeWarpActive=false; goldRushActive=false; divineActive=false; bugEventActive=false; bugEventGlitch=false;
        stableCycleActive=false; tidalWaveActive=false; masqueradeActive=false; soulHarvestActive=false; frozenTimeActive=false; crystalRainActive=false; shadowCloneActive=false;
        lightningStormActive=false; doomsDayActive=false; royalBlessingActive=false; starfallActive=false; infernoActive=false; chainLightningActive=false; barrierActive=false;
        soulReaperActive=false; vortexActive=false; kingsBlessingActive=false; prismActive=false; abyssActive=false; doppelgangerActive=false; supernovaActive=false;
        riftMultiplier=1;
        document.getElementById('event-banner').style.display='none'; startEventCooldown();
    }
    
    // Starfall effect (simplified)
    if(starfallActive){
        for(let i=0;i<starfallStars.length;i++){
            starfallStars[i].update();
            starfallStars[i].draw();
            if(starfallStars[i].y>height){
                starfallStars.splice(i,1);
                i--;
            } else if(player && Math.hypot(starfallStars[i].x-player.x, starfallStars[i].y-player.y) < starfallStars[i].size+player.r){
                let bonus = starfallStars[i].value * gemMultiplier;
                gemstones += bonus;
                saveGemstones();
                particles.push(new Particle(starfallStars[i].x,starfallStars[i].y,0,0,'#ffffaa',0.8));
                starfallStars.splice(i,1);
                i--;
                showNotification(`⭐ Star collected! +${bonus} GEMSTONES!`, 'success');
            }
        }
        if(Math.random()<0.2){
            starfallStars.push(new StarfallStar(Math.random()*width, -20, 8+Math.random()*8, 50+Math.floor(Math.random()*100)));
        }
    }
    
    // Update music based on game state
    updateMusicBasedOnGameState();
    
    // Background rendering and game logic (same as original)
    if(synapseActive){
        synapseBackgroundHue = (synapseBackgroundHue + 2.5) % 360;
        ctx.fillStyle = `hsl(${synapseBackgroundHue}, 70%, 7%)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(chaosRealmActive){
        ctx.fillStyle = `hsl(${chaosHue}, 80%, 10%)`; ctx.fillRect(0,0,width,height); drawNebula();
        for(let i=0;i<50;i++){ ctx.fillStyle = `hsl(${chaosHue + i*7}, 100%, 50%)`; ctx.fillRect(Math.random()*width, Math.random()*height, 2, 2); }
    } else if(primordialRageActive){ ctx.fillStyle = `rgba(80,0,0,0.7)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(cosmicCollapseActive){ ctx.fillStyle = `rgba(0,20,60,0.6)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(blackHoleActive){ ctx.fillStyle = `rgba(20,0,40,0.8)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(voidActive){ ctx.fillStyle = `rgba(80,30,100,0.5)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(apocalypseActive){ ctx.fillStyle = `rgba(80,20,20,0.5)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(riftActive){ ctx.fillStyle = `rgba(100,30,120,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(goldRushActive){ ctx.fillStyle = `rgba(80,60,0,0.4)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(divineActive){ ctx.fillStyle = `rgba(255,200,0,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(bugEventActive){ ctx.fillStyle = `rgba(0,255,0,0.1)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(stableCycleActive){ ctx.fillStyle = `rgba(100,150,255,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(tidalWaveActive){ ctx.fillStyle = `rgba(0,100,150,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(masqueradeActive){ ctx.fillStyle = `rgba(150,100,0,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(soulHarvestActive){ ctx.fillStyle = `rgba(80,0,80,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(frozenTimeActive){ ctx.fillStyle = `rgba(100,150,200,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(crystalRainActive){ ctx.fillStyle = `rgba(100,200,150,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(shadowCloneActive){ ctx.fillStyle = `rgba(100,80,150,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(lightningStormActive){ ctx.fillStyle = `rgba(80,80,0,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(doomsDayActive){ ctx.fillStyle = `rgba(80,0,0,0.4)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(royalBlessingActive){ ctx.fillStyle = `rgba(255,200,0,0.15)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(starfallActive){ ctx.fillStyle = `rgba(100,100,50,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(infernoActive){ ctx.fillStyle = `rgba(80,30,0,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(chainLightningActive){ ctx.fillStyle = `rgba(80,80,0,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(barrierActive){ ctx.fillStyle = `rgba(68,85,170,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(soulReaperActive){ ctx.fillStyle = `rgba(68,0,85,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(vortexActive){ ctx.fillStyle = `rgba(34,85,127,0.25)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(kingsBlessingActive){ ctx.fillStyle = `rgba(127,100,0,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(prismActive){ ctx.fillStyle = `rgba(127,68,127,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(abyssActive){ ctx.fillStyle = `rgba(34,0,85,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(doppelgangerActive){ ctx.fillStyle = `rgba(127,68,127,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else { ctx.clearRect(0,0,width,height); ctx.fillStyle='#000'; ctx.fillRect(0,0,width,height); drawNebula(); }
    
    stars.forEach(s=>{
        if(gameState==='PLAYING')s.y+= (isOD||synapseActive||apocalypseActive||primordialRageActive||cosmicCollapseActive||chaosRealmActive||bugEventActive||stableCycleActive||lightningStormActive||royalBlessingActive||kingsBlessingActive||prismActive||doppelgangerActive)?s.v*5:s.v;
        if(s.y>height){s.y=0;s.x=Math.random()*width;}
        let starColor=isOD?'#ff00ff':synapseActive?`hsl(${synapseBackgroundHue},80%,55%)`:primordialRageActive?'#ff0000':chaosRealmActive?`hsl(${chaosHue},100%,60%)`:cosmicCollapseActive?'#88aaff':apocalypseActive?'#ff0000':goldRushActive?'#ffaa00':bugEventActive?'#00ff00':stableCycleActive?'#88aaff':frozenTimeActive?'#88ccff':crystalRainActive?'#88ffaa':lightningStormActive?'#ffff00':royalBlessingActive?'#ffdd00':kingsBlessingActive?'#ffaa00':prismActive?'#ff88ff':doppelgangerActive?'#ff88ff':s.v>2.5?'#aaddff':'#ffffff';
        ctx.fillStyle=starColor; ctx.globalAlpha=s.v/3; ctx.fillRect(s.x,s.y,isOD?2:1.8,isOD?10:s.v>2.5?2:1.8); ctx.globalAlpha=1;
    });

    if(gameState==='PLAYING' && player && player.x && player.y){
        player.update(); player.draw(); if(settings.shake) shake*=0.85;
        if(waveBannerTimer>0){waveBannerTimer--; document.getElementById('wave-banner').style.display='block';}
        else document.getElementById('wave-banner').style.display='none';
        if(bossWarningTimer>0){bossWarningTimer--; document.getElementById('boss-warning').style.display='block';}
        else document.getElementById('boss-warning').style.display='none';

        if(waveKills>=waveKillGoal && !waveTriggered && !boss && !guardian){
            waveTriggered=true;
            if(waveNoDamage && wave>1) perfectWavesCount++;
            setTimeout(()=>{if(gameState==='PLAYING') startWave(wave+1);},1800);
        }

        if(!endlessMode && wave>1 && wave%5===0 && !boss && !guardian && !enemies.some(e=>e.isBoss) && waveKills<3){
            if(bossWarningTimer===0 && !waveTriggered){
                bossWarningTimer=100;
                setTimeout(()=>{ if(gameState==='PLAYING' && !boss && !guardian){ boss=new Enemy(true); enemies.push(boss); floats.push({txt:'⚠ BOSS!',x:width/2-35,y:height/2,l:1.5,c:'#ff0044',size:32}); } },2500);
            }
        }

        let spawnChance = 0.012 + (wave*0.002) + (level*0.0012);
        if(apocalypseActive) spawnChance*=2; if(cosmicCollapseActive) spawnChance*=3; if(riftActive) spawnChance*=1.5;
        if(tidalWaveActive) spawnChance*=2;
        if(doomsDayActive) spawnChance*=0.5;
        if(Math.random()<spawnChance && enemies.length<20+wave) enemies.push(new Enemy());

        const isRapid=!!activePowerUps.rapidfire;
        let fr=isOD||isRapid?40:Math.max(65,240-fireLevel*38);
        if(synapseActive) fr*=0.55; if(primordialRageActive) fr*=0.3; if(chaosRealmActive) fr*=0.2;
        if(apocalypseActive) fr*=0.7; if(cosmicCollapseActive) fr*=0.5; if(riftActive) fr*=0.9;
        if(bugEventActive) fr*=0.3; if(stableCycleActive) fr*=0.7;
        if(royalBlessingActive) fr*=0.5; if(kingsBlessingActive) fr*=0.3;
        if(prismActive) fr*=0.6;
        if(doppelgangerActive) fr*=0.8;
        fr /= skinFireRateMultiplier;
        if(Date.now()-lastFire>fr){ fireBullets(); let maxDrones=Math.min(400, droneCount); for(let i=0;i<maxDrones;i++){ const a=(Date.now()/400)+(i*Math.PI*2/maxDrones); if(Math.random()<0.3) bullets.push(new Bullet(player.x+Math.cos(a)*55,player.y+Math.sin(a)*45,Math.ceil(damageLevel*0.55),'#00ffaa')); } }

        checkEventTrigger();

        if(activeEvent === 'reaper'){
            for(let i=0;i<enemies.length;i++){
                let e=enemies[i];
                let pointBonus = e.isBoss?90000:3500*combo;
                if(goldRushActive) pointBonus*=10;
                if(stableCycleActive) pointBonus*=3;
                if(royalBlessingActive) pointBonus*=3;
                if(kingsBlessingActive) pointBonus*=5;
                pointBonus *= skinCreditMultiplier;
                score+=pointBonus; kills++; waveKills++;
                if(e.isBoss) bossesKilled++;
                for(let k=0;k<15;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*12,(Math.random()-0.5)*12,'#880044',0.9));
                enemies.splice(i,1); i--;
            }
            boss=null;
            activeEvent=null;
            document.getElementById('event-banner').style.display='none';
        }

        if(blackHoleActive){
            ctx.fillStyle='rgba(0,0,0,0.3)'; ctx.beginPath(); ctx.arc(blackHoleCenter.x,blackHoleCenter.y,80,0,Math.PI*2); ctx.fill();
            ctx.fillStyle='#4400aa'; ctx.beginPath(); ctx.arc(blackHoleCenter.x,blackHoleCenter.y,40,0,Math.PI*2); ctx.fill();
            for(let i=0;i<enemies.length;i++){
                let dx = blackHoleCenter.x - enemies[i].x, dy = blackHoleCenter.y - enemies[i].y, dist = Math.hypot(dx,dy);
                if(dist<150){ let pull = (150-dist)/150 * 8; enemies[i].x += dx/dist * pull; enemies[i].y += dy/dist * pull;
                    if(dist<30){ 
                        let pointBonus = 70*combo;
                        if(goldRushActive) pointBonus*=10;
                        if(stableCycleActive) pointBonus*=3;
                        if(royalBlessingActive) pointBonus*=3;
                        if(kingsBlessingActive) pointBonus*=5;
                        pointBonus *= skinCreditMultiplier;
                        score+=pointBonus; kills++; waveKills++; xp+=10; if(xp>=100){xp=0;level++;sfxLevelUp();}
                        particles.push(new Particle(enemies[i].x,enemies[i].y,0,0,'#aa66ff',0.8)); enemies.splice(i,1); i--; 
                    }
                }
            }
        }

        for(let i=0;i<meteors.length;i++){ meteors[i].update(); meteors[i].draw();
            if(!voidActive && !primordialRageActive && !chaosRealmActive && !divineActive && !bugEventActive && !stableCycleActive && !frozenTimeActive && !royalBlessingActive && !barrierActive && Math.hypot(meteors[i].x-player.x,meteors[i].y-player.y)<meteors[i].radius+player.r){
                let dmg = 15 * damageReduction;
                health-=dmg; if(settings.shake) shake=15; player.invincibleTimer=40; sfxHit(); waveNoDamage=false; meteors.splice(i,1); i--; if(health<=0){gameOver();return;}
                if(barrierActive){
                    barrierHp -= dmg;
                    if(barrierHp<=0) barrierActive=false;
                    health += dmg;
                    if(health>maxHealth) health = maxHealth;
                }
            }
            if(meteors[i].y>height+50){ meteors.splice(i,1); i--; }
        }

        if(guardian){ guardian.update(); guardian.draw();
            if(!voidActive && !primordialRageActive && !chaosRealmActive && !divineActive && !bugEventActive && !stableCycleActive && !frozenTimeActive && !royalBlessingActive && !barrierActive && Math.hypot(guardian.x-player.x,guardian.y-player.y)<guardian.r+player.r){
                let dmg = 40 * damageReduction;
                health-=dmg; if(settings.shake) shake=20; player.invincibleTimer=60; sfxHit(); waveNoDamage=false; if(health<=0){gameOver();return;}
                if(barrierActive){
                    barrierHp -= dmg;
                    if(barrierHp<=0) barrierActive=false;
                    health += dmg;
                    if(health>maxHealth) health = maxHealth;
                }
            }
            if(guardian.y>height+200){ guardian=null; activeEvent=null; }
        }

        const magnetActive=!!activePowerUps.magnet;
        const nextBullets=[], nextEBullets=[], nextItems=[], nextParticles=[], nextFloats=[];

        const preCollide=[];
        for(const e of enemies){ e.update(); e.draw(); let dead=false;
            if(!voidActive && !primordialRageActive && !chaosRealmActive && !divineActive && !bugEventActive && !stableCycleActive && !frozenTimeActive && !royalBlessingActive && !barrierActive && Math.hypot(e.x-player.x,e.y-player.y)<e.r+18 && player.invincibleTimer===0){
                if(!isOD && !synapseActive){
                    let dmg = e.isBoss?15:8; if(apocalypseActive) dmg*=1.5; if(cosmicCollapseActive) dmg*=1.3; if(doomsDayActive) dmg*=0.5;
                    dmg *= damageReduction;
                    health-=hasShieldUpgrade?Math.floor(dmg*0.5):dmg; if(settings.shake) shake=15; player.invincibleTimer=45; sfxHit(); waveNoDamage=false;
                    if(health<=0){gameOver();return;}
                    if(barrierActive){
                        barrierHp -= dmg;
                        if(barrierHp<=0) barrierActive=false;
                        health += dmg;
                        if(health>maxHealth) health = maxHealth;
                    }
                }
                if(!e.isBoss) dead=true;
            }
            if(e.y>height+180){ if(e.isBoss) boss=null; combo=1; dead=true; }
            if(!dead) preCollide.push(e); else if(e.isBoss) boss=null;
        }

        const hitBullets=new Set(), finalEnemies=[];
        for(const e of preCollide){ let destroyed=false;
            for(let j=0;j<bullets.length;j++){
                if(hitBullets.has(j)) continue;
                const b=bullets[j];
                if(Math.hypot(b.x-e.x,b.y-e.y)<e.r){
                    if(!b.isLaser) hitBullets.add(j);
                    e.hp-=b.p;
                    if(e.hp<=0){
                        destroyed=true; sfxExplode();
                        const cnt=e.isBoss?25:12;
                        for(let k=0;k<cnt;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*12,(Math.random()-0.5)*12,e.color,0.8));
                        let pointBonus = e.isBoss?1800:70*combo;
                        if(apocalypseActive) pointBonus*=5; if(cosmicCollapseActive) pointBonus*=10; if(riftActive) pointBonus*=riftMultiplier;
                        if(goldRushActive) pointBonus*=10;
                        if(bugEventActive) pointBonus*=5;
                        if(stableCycleActive) pointBonus*=3;
                        if(royalBlessingActive) pointBonus*=3;
                        if(kingsBlessingActive) pointBonus*=5;
                        if(doomsDayActive) pointBonus*=2;
                        pointBonus *= skinCreditMultiplier;
                        score+=pointBonus; kills++; waveKills++;
                        if(synapseActive) synapseKills++;
                        xp+=e.isBoss?35:10;
                        if(xp>=100){ xp=0; level++; sfxLevelUp(); floats.push({txt:'▲ RANK UP!',x:player.x-40,y:player.y-25,l:1.5,c:'#00ffaa',size:20}); }
                        odCharge=Math.min(100,odCharge+(e.isBoss?45:3));
                        if(e.isBoss){ bossesKilled++; boss=null; }
                        const roll=Math.random();
                        if(roll>0.65) items.push({x:e.x,y:e.y,vy:2,isPowerUp:false});
                        if(roll>0.88) spawnPowerUp(e.x+20,e.y);
                        if(Math.random()<0.08) spawnPsychedelia(e.x,e.y);
                        if(Math.random()<0.05 && blackHolePieces<6) spawnBlackHolePiece(e.x,e.y);
                        
                        if(soulHarvestActive && !e.isBoss){
                            soulHarvestSouls.push(new Soul(e.x, e.y));
                        }
                        
                        combo++; if(combo>maxCombo) maxCombo=combo; comboTimer=Date.now()+2200;
                        updateComboMeter();
                        checkAchievements(); break;
                    }
                }
            }
            if(!destroyed) finalEnemies.push(e);
        }

        for(let j=0;j<bullets.length;j++){ if(hitBullets.has(j)) continue; const b=bullets[j];
            for(let m=0;m<meteors.length;m++){ if(Math.hypot(b.x-meteors[m].x,b.y-meteors[m].y)<meteors[m].radius){
                hitBullets.add(j); meteors[m].hp-=b.p;
                if(meteors[m].hp<=0){ 
                    let pointBonus = 200;
                    if(goldRushActive) pointBonus*=10;
                    if(bugEventActive) pointBonus*=5;
                    if(stableCycleActive) pointBonus*=3;
                    if(royalBlessingActive) pointBonus*=3;
                    if(kingsBlessingActive) pointBonus*=5;
                    pointBonus *= skinCreditMultiplier;
                    score+=pointBonus; totalMeteorsDestroyed++;
                    for(let k=0;k<8;k++) particles.push(new Particle(meteors[m].x,meteors[m].y,(Math.random()-0.5)*8,(Math.random()-0.5)*8,'#ff8844',0.8));
                    meteors.splice(m,1); m--;
                } break;
            }}
        }
        
        if(guardian){ for(let j=0;j<bullets.length;j++){ if(hitBullets.has(j)) continue; const b=bullets[j];
            if(Math.hypot(b.x-guardian.x,b.y-guardian.y)<guardian.r){
                hitBullets.add(j); guardian.hp-=b.p;
                if(guardian.hp<=0){ guardian=null; activeEvent=null; 
                    let pointBonus = 50000;
                    if(goldRushActive) pointBonus*=10;
                    if(bugEventActive) pointBonus*=5;
                    if(stableCycleActive) pointBonus*=3;
                    if(royalBlessingActive) pointBonus*=3;
                    if(kingsBlessingActive) pointBonus*=5;
                    pointBonus *= skinCreditMultiplier;
                    score+=pointBonus; totalCoins+=5000;
                    if(!skinUnlocked){ skinUnlocked=true; localStorage.setItem('skinUnlocked','true'); showAchievementPopup('👑 GUARDIAN DEFEATED', 'Golden skin unlocked!'); }
                    let count=parseInt(localStorage.getItem('guardianDefeatedCount')||0)+1; localStorage.setItem('guardianDefeatedCount',count);
                    guardianDefeated=true; for(let k=0;k<50;k++) particles.push(new Particle(guardian.x,guardian.y,(Math.random()-0.5)*20,(Math.random()-0.5)*20,'gold',1));
                    checkAchievements();
                } break;
            }}
        }

        for(let j=0;j<bullets.length;j++){ if(!hitBullets.has(j)){ bullets[j].update(); bullets[j].draw(); if(bullets[j].y>-45 && bullets[j].y<height+45 && bullets[j].x>-45 && bullets[j].x<width+45) nextBullets.push(bullets[j]); } }
        bullets.length=0; bullets.push(...nextBullets);
        enemies.length=0; enemies.push(...finalEnemies);

        for(const eb of eBullets){ eb.x+=eb.vx||0; eb.y+=eb.vy;
            ctx.fillStyle='#ff0044'; ctx.shadowBlur=7; ctx.beginPath(); ctx.arc(eb.x,eb.y,5,0,Math.PI*2); ctx.fill();
            let keep=true;
            if(!voidActive && !primordialRageActive && !chaosRealmActive && !divineActive && !bugEventActive && !stableCycleActive && !frozenTimeActive && !royalBlessingActive && !barrierActive && Math.hypot(eb.x-player.x,eb.y-player.y)<22 && player.invincibleTimer===0){
                if(!isOD && !synapseActive){ 
                    let dmg = (hasShieldUpgrade?4:9) * damageReduction;
                    health-=dmg; if(settings.shake) shake=10; player.invincibleTimer=22; sfxHit(); waveNoDamage=false; 
                    if(health<=0){gameOver();return;}
                    if(barrierActive){
                        barrierHp -= dmg;
                        if(barrierHp<=0) barrierActive=false;
                        health += dmg;
                        if(health>maxHealth) health = maxHealth;
                    }
                }
                keep=false;
            }
            if(eb.y>height || eb.x<-20 || eb.x>width+20) keep=false;
            if(keep) nextEBullets.push(eb);
        }
        eBullets.length=0; eBullets.push(...nextEBullets);

        for(const it of items){ it.y+=it.vy||2;
            const d=Math.hypot(it.x-player.x,it.y-player.y); const pull=magnetActive?280:100;
            if(d<pull){ it.x+=(player.x-it.x)*0.16; it.y+=(player.y-it.y)*0.16; }
            let keep=true;
            if(d<30){
                if(it.isPowerUp){ applyPowerUp(it.puType); }
                else if(it.isPsychedelia){ synapsePsychedeliaCount++; psychedeliaCollected++; sfxPowerup(); floats.push({txt:'🧠 +1',x:it.x,y:it.y,l:0.8,c:'#ff66ff',size:14});
                    if(synapsePsychedeliaCount>=8 && !synapseActive){ synapsePsychedeliaCount=0; activateSynapse(); }
                } else if(it.isBlackHolePiece){ blackHolePieces++; sfxPowerup(); floats.push({txt:'⭐ +1',x:it.x,y:it.y,l:0.8,c:'gold',size:14});
                    if(blackHolePieces>=6 && !blackHoleActive && !activeEvent){ blackHolePieces=0; triggerBlackHole(); }
                } else { let bonus=4+Math.floor(level*0.4+wave*0.3); if(apocalypseActive) bonus*=3; if(cosmicCollapseActive) bonus*=5; if(riftActive) bonus*=riftMultiplier;
                    if(goldRushActive) bonus*=10;
                    if(bugEventActive) bonus*=5;
                    if(stableCycleActive) bonus*=3;
                    if(royalBlessingActive) bonus*=3;
                    if(kingsBlessingActive) bonus*=5;
                    bonus *= skinCreditMultiplier;
                    totalCoins+=bonus; sfxCoin(); floats.push({txt:'+'+bonus+'c',x:it.x,y:it.y,l:0.8,c:'gold',size:12}); }
                keep=false;
            }
            if(it.y>height+45) keep=false;
            if(keep){
                if(it.isPowerUp){ ctx.fillStyle=it.puType.color; ctx.shadowBlur=10; ctx.font='bold 16px Segoe UI'; ctx.fillText(it.puType.icon,it.x-8,it.y+5);
                    ctx.beginPath(); ctx.arc(it.x,it.y+2,10+Math.sin(Date.now()/150)*2,0,Math.PI*2); ctx.stroke();
                } else if(it.isPsychedelia){ ctx.fillStyle='#ff66ff'; ctx.shadowBlur=10; ctx.font='bold 18px Segoe UI'; ctx.fillText('🧠',it.x-9,it.y+7);
                    ctx.beginPath(); ctx.arc(it.x,it.y+2,9+Math.sin(Date.now()/180)*2,0,Math.PI*2); ctx.strokeStyle='#ff66ff'; ctx.stroke();
                } else if(it.isBlackHolePiece){ ctx.fillStyle='gold'; ctx.shadowBlur=10; ctx.font='bold 18px Segoe UI'; ctx.fillText('⭐',it.x-9,it.y+7);
                    ctx.beginPath(); ctx.arc(it.x,it.y+2,9+Math.sin(Date.now()/180)*2,0,Math.PI*2); ctx.strokeStyle='gold'; ctx.stroke();
                } else { ctx.fillStyle='gold'; ctx.shadowBlur=8; ctx.beginPath(); ctx.arc(it.x,it.y,5,0,Math.PI*2); ctx.fill();
                    ctx.beginPath(); ctx.arc(it.x,it.y,5+Math.sin(Date.now()/160)*2,0,Math.PI*2); ctx.stroke(); }
                nextItems.push(it);
            }
        }
        items.length=0; items.push(...nextItems);

        for(const p of particles){ p.update(); p.draw(); if(p.l>0) nextParticles.push(p); }
        particles.length=0; particles.push(...nextParticles);

        for(const f of floats){ f.y-=1.6; f.l-=0.014; ctx.font=`bold ${f.size||16}px Segoe UI`; ctx.globalAlpha=Math.max(0,Math.min(1,f.l)); ctx.fillStyle=f.c; ctx.shadowBlur=6; ctx.fillText(f.txt,f.x,f.y); if(f.l>0) nextFloats.push(f); }
        ctx.globalAlpha=1; ctx.shadowBlur=0; floats.length=0; floats.push(...nextFloats);

        tickPowerUps();

        if(Date.now()>comboTimer && gameState==='PLAYING') combo=1;
        if(isOD && Date.now()>odTimer){ isOD=false; odCharge=0; document.getElementById('od-btn').style.display='none'; }
        if(odCharge>=100 && !isOD) document.getElementById('od-btn').style.display='flex';
        if(isOD){ const rem=(odTimer-Date.now())/6000; document.getElementById('od-btn').style.background=`conic-gradient(#ff00ff ${rem*360}deg,rgba(142,68,173,0.3) 0deg)`; }

        document.getElementById('hp-fill').style.width=Math.max(0,health)+'%';
        document.getElementById('hp-num').innerText=Math.max(0,Math.ceil(health))+'%';
        document.getElementById('xp-fill').style.width=xp+'%';
        document.getElementById('od-fill').style.width=odCharge+'%';
        document.getElementById('score-val').innerText=formatNumber(score);
        document.getElementById('lvl-val').innerText=level;
        document.getElementById('kill-val').innerText=formatNumber(kills);
        document.getElementById('wave-val').innerText=wave;
        document.getElementById('bomb-count').innerText=bombCount;
        document.getElementById('combo-small').innerText=combo>1?'x'+combo:'';
        const hpPct=health/maxHealth;
        document.getElementById('hp-fill').style.background=hpPct>0.5?'linear-gradient(90deg,#ff0044,#ff5588)':hpPct>0.25?'linear-gradient(90deg,#ff6600,#ffaa00)':'linear-gradient(90deg,#ff2200,#ff5500)';
        document.getElementById('vignette').style.display=hpPct<0.3?'block':'none';
        
        updateDailyMissionProgress('kill', waveKills);
        updateDailyMissionProgress('gem', 0);
        if(waveNoDamage) updateDailyMissionProgress('perfect', waveNoDamage ? 1 : 0);
        updateDailyMissionProgress('crit', criticalHitsCount);
    }
    drawCrosshair(mouseX,mouseY);
}

// INPUT
window.addEventListener('resize',()=>{ width=canvas.width=window.innerWidth; height=canvas.height=window.innerHeight; nebulaCanvas.width=width; nebulaCanvas.height=height; resizeCross(); drawNebula(); });
window.dispatchEvent(new Event('resize'));
canvas.addEventListener('mousemove',e=>{ mouseX=e.clientX; mouseY=e.clientY; if(gameState==='PLAYING' && player){ player.tx=e.clientX; player.ty=e.clientY-50; } });
canvas.addEventListener('touchmove',e=>{ e.preventDefault(); if(gameState==='PLAYING' && player){ mouseX=e.touches[0].clientX; mouseY=e.touches[0].clientY; player.tx=e.touches[0].clientX; player.ty=e.touches[0].clientY-65; } },{passive:false});
canvas.addEventListener('touchstart',e=>{ e.preventDefault(); initAudio(); },{passive:false});
canvas.addEventListener('click',()=>initAudio());
window.addEventListener('keydown',e=>{
    if(e.key==='Escape' && (gameState==='PLAYING'||gameState==='PAUSED')) togglePause();
    if((e.key==='q'||e.key==='Q') && gameState==='PLAYING') useBomb();
    if((e.key==='o'||e.key==='O') && gameState==='PLAYING') activateOverdrive();
});

// INIT
for(let i=0;i<90;i++) stars.push({x:Math.random()*(window.innerWidth||800), y:Math.random()*(window.innerHeight||600), v:Math.random()*2.2+0.8});
runLoader('loader-init','init-fill',()=>{
    document.getElementById('main-hub').style.display='flex';
    gameState='MENU';
    resetDailyMissions();
    updateDailyMissionsUI();
    updateAchievementsUI();
    updateSkinProgressUI();
    updateEventsUI();
    updateHubUI();
    updateGemUI();
    updateIndividualRewardsUI();
    updateSkinsUI();
    checkSkinUnlock();
    updateRankUI();
    initSettingsUI();
});

// Initialize music on first user interaction
function initMusicOnFirstInteraction() {
    try {
        if (audioCtx && audioCtx.state === 'suspended') {
            audioCtx.resume();
        }
        if (musicEnabled && !backgroundMusic && gameState === 'MENU') {
            playBackgroundMusic('menu');
        }
    } catch(e) {}
    document.body.removeEventListener('click', initMusicOnFirstInteraction);
    document.body.removeEventListener('touchstart', initMusicOnFirstInteraction);
}
document.body.addEventListener('click', initMusicOnFirstInteraction);
document.body.addEventListener('touchstart', initMusicOnFirstInteraction);
</script>
</body>
</html>
