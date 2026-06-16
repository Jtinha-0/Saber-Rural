<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Saber Rural · Diário de Plantio</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #050b17; --card: rgba(14,22,38,0.8); --text: #e8edf5; --text2: #8895b3;
            --green: #10b981; --green-bright: #34d399; --gold: #f59e0b; --gold-bright: #fbbf24;
            --blue: #3b82f6; --red: #ef4444; --border: rgba(255,255,255,0.07);
            --shadow: 0 12px 40px rgba(0,0,0,0.55); --radius: 14px; --trans: 0.3s ease;
        }
        * { margin:0; padding:0; box-sizing:border-box; }
        body { font-family:'Inter',sans-serif; background:#030812; color:var(--text); height:100vh; overflow:hidden; }
        ::-webkit-scrollbar { width:4px; }
        ::-webkit-scrollbar-thumb { background:rgba(255,255,255,0.08); border-radius:8px; }

        .login-overlay { position:fixed; inset:0; z-index:9999; display:flex; align-items:center; justify-content:center; background:rgba(2,6,15,0.92); backdrop-filter:blur(25px); }
        .login-card { background:var(--card); backdrop-filter:blur(45px); border:1px solid var(--border); border-radius:var(--radius); padding:44px 32px; width:390px; box-shadow:var(--shadow); text-align:center; animation:slideUp 0.55s ease; }
        @keyframes slideUp { from{opacity:0;transform:translateY(45px);} to{opacity:1;transform:translateY(0);} }
        .login-logo { width:68px;height:68px;background:linear-gradient(135deg,var(--green),#059669);border-radius:20px;display:flex;align-items:center;justify-content:center;font-size:34px;margin:0 auto 18px;box-shadow:0 10px 35px rgba(16,185,129,0.35);animation:float 3s ease-in-out infinite; }
        @keyframes float { 0%,100%{transform:translateY(0);} 50%{transform:translateY(-7px);} }
        .login-card h2 { font-size:26px;font-weight:800;color:#fff;margin-bottom:4px; }
        .login-card .sub { color:var(--text2);font-size:12px;margin-bottom:26px; }
        .field { margin-bottom:15px;text-align:left;position:relative; }
        .field label { font-size:10px;font-weight:700;color:var(--text2);display:block;margin-bottom:5px;text-transform:uppercase;letter-spacing:0.6px; }
        .field input, .field select { width:100%;padding:12px 42px 12px 14px;background:rgba(255,255,255,0.025);border:1px solid var(--border);border-radius:8px;color:#fff;font-size:13px;outline:none;transition:var(--trans); appearance:none; }
        .field select { cursor:pointer; }
        .field select option { background:#0f172a; color:#fff; }
        .field input:focus, .field select:focus { border-color:var(--green);box-shadow:0 0 0 4px rgba(16,185,129,0.12);background:rgba(255,255,255,0.04); }
        .eye-btn { position:absolute;right:12px;bottom:10px;background:none;border:none;cursor:pointer;font-size:18px;color:var(--text2);transition:var(--trans); }
        .eye-btn:hover { color:#fff; }
        .btn { width:100%;padding:13px;border:none;border-radius:8px;font-weight:700;font-size:13px;cursor:pointer;margin-top:6px;transition:var(--trans);letter-spacing:0.3px; }
        .btn-green { background:linear-gradient(135deg,var(--green),#059669);color:#fff;box-shadow:0 8px 25px rgba(16,185,129,0.3); }
        .btn-green:hover { transform:translateY(-2px);box-shadow:0 12px 35px rgba(16,185,129,0.4);filter:brightness(1.1); }
        .btn-ghost { background:transparent;color:var(--text2);border:1px solid var(--border); }
        .btn-ghost:hover { background:rgba(255,255,255,0.03);color:#fff;border-color:rgba(255,255,255,0.2); }
        .btn-red { background:rgba(239,68,68,0.15);color:#fca5a5;border:1px solid rgba(239,68,68,0.3); }
        .btn-red:hover { background:rgba(239,68,68,0.25); }
        .link { color:var(--green);cursor:pointer;font-size:11px;font-weight:500;transition:var(--trans); }
        .link:hover { color:var(--green-bright); }
        .hidden { display:none !important; }

        .app { display:none;height:100vh;flex-direction:column; }
        .app.active { display:flex; }
        .topbar { background:rgba(9,17,30,0.82);backdrop-filter:blur(35px);border-bottom:1px solid var(--border);padding:10px 24px;display:flex;align-items:center;justify-content:space-between;z-index:100; }
        .logo { display:flex;align-items:center;gap:13px; }
        .logo-icon { width:42px;height:42px;background:linear-gradient(135deg,var(--green),#059669);border-radius:13px;display:flex;align-items:center;justify-content:center;font-size:23px;box-shadow:0 5px 18px rgba(16,185,129,0.35);animation:float 3s ease-in-out infinite; }
        .logo-text h1 { font-size:20px;font-weight:800;color:#fff; }
        .logo-text span { font-size:10px;color:var(--green-bright);font-weight:600; }
        .badge { background:rgba(255,255,255,0.04);border:1px solid var(--border);padding:7px 15px;border-radius:22px;font-size:11px;color:var(--text2);font-weight:500; }
        .btn-logout { background:rgba(239,68,68,0.08);border:1px solid rgba(239,68,68,0.2);color:#fca5a5;padding:7px 15px;border-radius:22px;cursor:pointer;font-size:11px;font-weight:600;transition:var(--trans); }
        .btn-logout:hover { background:rgba(239,68,68,0.18);color:#fecaca; }

        .content { display:flex;flex:1;overflow:hidden; }
        .map-panel { flex:1.6;position:relative; }
        #map { width:100%;height:100%;background:#040b16; }
        .side-panel { width:445px;background:rgba(9,17,30,0.65);backdrop-filter:blur(45px);border-left:1px solid var(--border);display:flex;flex-direction:column; }
        .tabs { display:flex;border-bottom:1px solid var(--border);padding:5px;gap:4px; flex-wrap:wrap; }
        .tab { flex:1; min-width:55px; padding:10px 4px;text-align:center;font-size:10px;font-weight:700;cursor:pointer;color:var(--text2);border:none;background:transparent;border-radius:var(--radius);transition:var(--trans);letter-spacing:0.2px; }
        .tab:hover { color:#fff;background:rgba(255,255,255,0.03); }
        .tab.active { color:#fff;background:rgba(16,185,129,0.12);box-shadow:inset 0 0 0 1px rgba(16,185,129,0.35),0 4px 14px rgba(16,185,129,0.12); }
        .tab-content { display:none;flex:1;overflow-y:auto;padding:10px 12px; }
        .tab-content.active { display:block; }

        .card { background:var(--card);backdrop-filter:blur(35px);border:1px solid var(--border);border-radius:var(--radius);padding:17px;margin-bottom:9px;box-shadow:0 4px 18px rgba(0,0,0,0.35);transition:var(--trans); }
        .card:hover { border-color:rgba(255,255,255,0.15); }
        .card h3 { font-size:15px;font-weight:700;color:#fff;margin-bottom:9px;display:flex;align-items:center;gap:8px; }
        .tag { display:inline-block;padding:4px 11px;border-radius:18px;font-size:10px;font-weight:700;margin:2px;text-transform:uppercase;letter-spacing:0.4px; }
        .tag-green { background:rgba(16,185,129,0.16);color:var(--green-bright); }
        .tag-gold { background:rgba(245,158,11,0.16);color:var(--gold-bright); }
        .tag-blue { background:rgba(59,130,246,0.16);color:#60a5fa; }
        .tag-red { background:rgba(239,68,68,0.16);color:#fca5a5; }

        .clima-card { background:linear-gradient(150deg,rgba(10,45,35,0.75),rgba(6,28,48,0.75));border:1px solid rgba(16,185,129,0.22);border-radius:var(--radius);padding:20px;margin-bottom:10px;display:flex;align-items:center;gap:18px;backdrop-filter:blur(25px);box-shadow:0 8px 30px rgba(0,0,0,0.4); }
        .clima-icon { font-size:56px;filter:drop-shadow(0 5px 10px rgba(0,0,0,0.4));animation:climaPulse 2.5s ease-in-out infinite; }
        @keyframes climaPulse { 0%,100%{transform:scale(1);} 50%{transform:scale(1.07);} }
        .clima-temp { font-size:52px;font-weight:300;line-height:1; }
        .clima-grid { display:grid;grid-template-columns:1fr 1fr;gap:7px;margin-top:12px; }
        .clima-item { background:rgba(255,255,255,0.05);border:1px solid rgba(255,255,255,0.04);padding:11px;border-radius:10px;text-align:center;font-size:11px; }
        .clima-item .val { font-size:19px;font-weight:700; }
        .clima-item .lbl { font-size:9px;opacity:0.5;text-transform:uppercase; }

        .search-box { position:sticky;top:0;z-index:10;padding:7px 0;margin-bottom:7px; }
        .search-box input { width:100%;padding:11px 17px;border:1px solid var(--border);border-radius:26px;background:rgba(255,255,255,0.025);color:#fff;font-size:12px;outline:none;transition:var(--trans); }
        .search-box input:focus { border-color:var(--green); }
        .mun-list { display:grid;grid-template-columns:1fr 1fr;gap:4px;max-height:350px;overflow-y:auto; }
        .mun-item { background:rgba(255,255,255,0.018);border:1px solid var(--border);padding:8px 10px;border-radius:8px;font-size:10px;cursor:pointer;border-left:3px solid var(--green);color:var(--text2);transition:var(--trans);white-space:nowrap;overflow:hidden;text-overflow:ellipsis; }
        .mun-item:hover { background:rgba(16,185,129,0.1);border-left-color:var(--gold);color:#fff;transform:translateX(3px); }
        .mun-item.capital { border-left-color:var(--gold);background:rgba(245,158,11,0.07);font-weight:700; }
        .mun-item.hidden-item { display:none; }
        .btn-back { background:rgba(255,255,255,0.03);border:1px solid var(--border);color:var(--text2);padding:6px 15px;border-radius:20px;cursor:pointer;font-size:11px;font-weight:600;margin-bottom:9px;display:inline-flex;align-items:center;gap:6px; }
        .btn-back:hover { background:rgba(255,255,255,0.07);color:#fff; }
        .spinner { width:30px;height:30px;border:3px solid rgba(255,255,255,0.04);border-top-color:var(--green);border-radius:50%;animation:spin 0.65s linear infinite;margin:35px auto; }
        @keyframes spin { to{transform:rotate(360deg);} }
        .empty { text-align:center;padding:45px 20px;color:var(--text2); }

        /* COMUNIDADE */
        .community-wrap { display:flex;flex-direction:column;height:100%; }
        .community-header { background:linear-gradient(135deg,rgba(245,158,11,0.15),rgba(245,158,11,0.05));border-bottom:1px solid rgba(245,158,11,0.2);padding:14px 16px; }
        .community-header h4 { font-size:14px;font-weight:700;color:var(--gold-bright);display:flex;align-items:center;gap:8px; }
        .community-msgs { flex:1;overflow-y:auto;padding:12px;background:linear-gradient(to bottom,rgba(5,10,20,0.45),rgba(10,18,32,0.55)); }
        .community-msg { margin-bottom:14px;animation:fadeIn 0.4s ease; }
        @keyframes fadeIn { from{opacity:0;transform:translateY(12px);} to{opacity:1;transform:translateY(0);} }
        .community-msg .msg-header { display:flex;align-items:center;gap:8px;margin-bottom:4px; }
        .community-msg .msg-avatar { width:32px;height:32px;border-radius:50%;display:flex;align-items:center;justify-content:center;font-size:14px;flex-shrink:0; }
        .community-msg .msg-avatar.user-av { background:linear-gradient(135deg,var(--gold),#d97706); }
        .community-msg .msg-avatar.bot-av { background:linear-gradient(135deg,var(--green),#047857); }
        .community-msg .msg-name { font-weight:700;font-size:12px;color:var(--gold-bright); }
        .community-msg .msg-name.bot-name { color:var(--green-bright); }
        .community-msg .msg-time { font-size:10px;color:var(--text2); }
        .community-msg .msg-body { background:var(--card);border:1px solid var(--border);padding:10px 14px;border-radius:12px;font-size:12px;line-height:1.5;color:var(--text);margin-left:40px; }
        .community-msg.bot-msg .msg-body { border-color:rgba(16,185,129,0.3);background:rgba(16,185,129,0.08); }
        .community-input { display:flex;gap:8px;padding:12px;border-top:1px solid var(--border);background:rgba(10,18,30,0.7); }
        .community-input input { flex:1;padding:11px 16px;border-radius:22px;border:1px solid var(--border);background:rgba(255,255,255,0.025);color:#fff;font-size:12px;outline:none; }
        .community-input input:focus { border-color:var(--gold); }
        .community-input button { background:linear-gradient(135deg,var(--gold),#d97706);color:#000;border:none;padding:11px 20px;border-radius:22px;cursor:pointer;font-weight:700;font-size:12px; }

        /* CALCULADORA */
        .calc-wrap { padding:10px; }
        .calc-card { background:linear-gradient(150deg,rgba(14,22,38,0.9),rgba(20,30,48,0.8));border:1px solid var(--border);border-radius:var(--radius);padding:18px;margin-bottom:12px; }
        .calc-card h3 { font-size:15px;font-weight:700;color:var(--gold-bright);margin-bottom:12px;display:flex;align-items:center;gap:8px; }
        .calc-row { display:flex;align-items:center;gap:10px;margin-bottom:10px; }
        .calc-row label { font-size:12px;font-weight:600;color:var(--text);min-width:90px; }
        .calc-row input { flex:1;padding:10px 14px;border:1px solid var(--border);border-radius:8px;background:rgba(255,255,255,0.03);color:#fff;font-size:13px;outline:none;text-align:right; }
        .calc-row input:focus { border-color:var(--green); }
        .calc-row .suffix { font-size:12px;color:var(--text2);min-width:60px;text-align:left; }
        .calc-result { background:rgba(16,185,129,0.1);border:1px solid rgba(16,185,129,0.3);border-radius:10px;padding:14px;margin-top:12px;text-align:center; }
        .calc-result .total { font-size:28px;font-weight:800;color:var(--green-bright); }
        .calc-result .lbl { font-size:10px;color:var(--text2);text-transform:uppercase;letter-spacing:0.5px; }
        .calc-grid { display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-top:10px; }
        .calc-item { background:rgba(255,255,255,0.03);border:1px solid var(--border);padding:10px;border-radius:8px;text-align:center;cursor:pointer;transition:var(--trans); }
        .calc-item:hover { background:rgba(16,185,129,0.1);border-color:var(--green); }
        .calc-item .grain { font-size:24px;margin-bottom:4px; }
        .calc-item .name { font-size:11px;font-weight:600;color:var(--text); }
        .calc-item .price { font-size:10px;color:var(--text2); }

        /* DIÁRIO DE PLANTIO */
        .diary-wrap { padding:8px; }
        .diary-entry { background:var(--card);border:1px solid var(--border);border-radius:var(--radius);padding:14px;margin-bottom:8px; }
        .diary-entry .entry-header { display:flex;justify-content:space-between;align-items:center;margin-bottom:8px; }
        .diary-entry .entry-date { font-size:10px;color:var(--text2); }
        .diary-entry .entry-crop { font-weight:700;font-size:13px;color:var(--gold-bright); }
        .diary-entry .entry-row { display:flex;justify-content:space-between;font-size:11px;padding:3px 0;color:var(--text2); }
        .diary-entry .entry-total { font-size:13px;font-weight:700;padding-top:6px;margin-top:6px;border-top:1px solid var(--border); }
        .diary-entry .profit { color:var(--green-bright); }
        .diary-entry .loss { color:#fca5a5; }
        .summary-card { background:linear-gradient(150deg,rgba(14,22,38,0.95),rgba(20,30,48,0.9));border:2px solid var(--border);border-radius:var(--radius);padding:18px;margin-bottom:10px;text-align:center; }
        .summary-card .big-number { font-size:36px;font-weight:800; }
        .summary-card .profit { color:var(--green-bright); }
        .summary-card .loss { color:#fca5a5; }
        .summary-grid { display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-top:10px; }
        .summary-item { background:rgba(255,255,255,0.03);padding:12px;border-radius:10px;text-align:center; }
        .summary-item .val { font-size:20px;font-weight:700; }
        .summary-item .lbl { font-size:9px;color:var(--text2);text-transform:uppercase; }

        .legend { position:absolute;bottom:14px;left:14px;background:rgba(9,17,30,0.88);backdrop-filter:blur(35px);border:1px solid var(--border);border-radius:var(--radius);padding:13px 17px;font-size:10px;z-index:1000;max-height:250px;overflow-y:auto; }
        .legend h4 { color:var(--gold-bright);margin-bottom:9px;font-size:11px;font-weight:800;text-transform:uppercase; }
        .legend-row { display:flex;align-items:center;gap:9px;margin:4px 0;cursor:pointer;padding:5px 9px;border-radius:7px;transition:var(--trans);color:var(--text2); }
        .legend-row:hover { background:rgba(255,255,255,0.05);color:#fff; }
        .legend-color { width:13px;height:13px;border-radius:3px;flex-shrink:0;box-shadow:0 0 9px currentColor; }

        .typing-dots { display:flex;gap:5px;padding:5px 0; }
        .typing-dots span { width:7px;height:7px;background:var(--text2);border-radius:50%;animation:bounce 1.3s infinite; }
        .typing-dots span:nth-child(2){ animation-delay:0.2s; }
        .typing-dots span:nth-child(3){ animation-delay:0.4s; }
        @keyframes bounce { 0%,60%,100%{transform:translateY(0);} 30%{transform:translateY(-13px);} }
        @media(max-width:768px){ .content{flex-direction:column;} .side-panel{width:100%;max-height:320px;} }
    </style>
</head>
<body>
    <div class="login-overlay" id="loginScreen">
        <div class="login-card">
            <div class="login-logo">🌾</div><h2>Saber Rural</h2><p class="sub">O conhecimento que faz o campo produzir mais</p>
            <div id="loginFields">
                <div class="field"><label>Email</label><input type="email" id="loginEmail" placeholder="seu@email.com"></div>
                <div class="field"><label>Senha</label><input type="password" id="loginPass"><button class="eye-btn" onclick="toggleEye('loginPass',this)">👁️</button></div>
                <button class="btn btn-green" onclick="doLogin()">🔐 Entrar</button>
                <button class="btn btn-ghost" onclick="showReg()">📝 Criar Conta</button>
                <div style="margin-top:10px;"><span class="link" onclick="recoverPass()">Esqueci a senha</span></div>
            </div>
            <div id="regFields" class="hidden">
                <div class="field"><label>Nome</label><input type="text" id="regName" placeholder="Seu nome completo"></div>
                <div class="field"><label>Email</label><input type="email" id="regEmail" placeholder="seu@email.com"></div>
                <div class="field"><label>Senha</label><input type="password" id="regPass"><button class="eye-btn" onclick="toggleEye('regPass',this)">👁️</button></div>
                <button class="btn btn-green" onclick="doRegister()">✅ Criar Conta</button>
                <button class="btn btn-ghost" onclick="showLogin()">← Voltar ao Login</button>
            </div>
        </div>
    </div>

    <div class="app" id="app">
        <div class="topbar">
            <div class="logo"><div class="logo-icon">🌾</div><div class="logo-text"><h1>Saber Rural</h1><span>Diário de Plantio</span></div></div>
            <div style="display:flex;align-items:center;gap:10px;"><span class="badge" id="userBadge">👤 Usuário</span><button class="btn-logout" onclick="logout()">Sair</button></div>
        </div>
        <div class="content">
            <div class="map-panel"><div id="map"></div><div class="legend" id="legend"><h4>🌿 Biomas</h4></div></div>
            <div class="side-panel">
                <div class="tabs">
                    <button class="tab active" onclick="switchTab('info')">📋 Mapa</button>
                    <button class="tab" onclick="switchTab('community')">💬 Comunidade</button>
                    <button class="tab" onclick="switchTab('chat')">🤖 AgroIA</button>
                    <button class="tab" onclick="switchTab('calc')">🧮 Calculadora</button>
                    <button class="tab" onclick="switchTab('diary')">📔 Diário</button>
                </div>
                <div class="tab-content active" id="tabInfo"><div id="infoContent"><div class="empty"><p>🗺️ Clique em um estado para ver seus municípios</p></div></div></div>
                <div class="tab-content" id="tabCommunity">
                    <div class="community-wrap">
                        <div class="community-header"><h4>👨‍🌾 Comunidade Rural</h4><p style="font-size:10px;color:var(--text2);margin-top:2px;">Compartilhe suas experiências ou mencione @AgroIA para falar com a IA</p></div>
                        <div class="community-msgs" id="communityMsgs"></div>
                        <div class="community-input">
                            <input id="communityInput" placeholder="Digite sua mensagem ou @AgroIA para perguntar..." onkeydown="if(event.key==='Enter')sendCommunityMsg()">
                            <button onclick="sendCommunityMsg()">Enviar</button>
                        </div>
                    </div>
                </div>
                <div class="tab-content" id="tabChat">
                    <div class="chat-wrap" style="display:flex;flex-direction:column;height:100%;">
                        <div class="chat-header" style="background:linear-gradient(135deg,rgba(16,185,129,0.18),rgba(5,150,105,0.09));border-bottom:1px solid rgba(16,185,129,0.22);padding:15px 17px;display:flex;align-items:center;gap:11px;">
                            <div class="ai-avatar" style="width:42px;height:42px;background:linear-gradient(135deg,var(--green),#047857);border-radius:50%;display:flex;align-items:center;justify-content:center;font-size:21px;box-shadow:0 5px 20px rgba(16,185,129,0.35);">🤖</div>
                            <div><h4 style="font-size:15px;font-weight:700;">AgroIA</h4><div class="status" style="font-size:10px;color:var(--green-bright);">🟢 Online</div></div>
                        </div>
                        <div class="chat-msgs" id="chatMsgs" style="flex:1;overflow-y:auto;padding:13px;background:linear-gradient(to bottom,rgba(5,10,20,0.45),rgba(10,18,32,0.55));"></div>
                        <div class="chat-row" style="display:flex;gap:9px;padding:13px 15px;border-top:1px solid var(--border);">
                            <input id="chatInput" placeholder="O que você quer saber?" onkeydown="if(event.key==='Enter')sendMsg()" style="flex:1;padding:12px 17px;border-radius:24px;border:1px solid var(--border);background:rgba(255,255,255,0.025);color:#fff;font-size:12px;outline:none;">
                            <button onclick="sendMsg()" style="background:linear-gradient(135deg,var(--green),#059669);color:#fff;border:none;padding:12px 22px;border-radius:24px;cursor:pointer;font-weight:700;">➤</button>
                        </div>
                    </div>
                </div>
                <div class="tab-content" id="tabCalc">
                    <div class="calc-wrap">
                        <div class="calc-card">
                            <h3>🧮 Calculadora de Grãos</h3>
                            <p style="font-size:11px;color:var(--text2);margin-bottom:12px;">Valores por saca de 60kg</p>
                            <div class="calc-grid">
                                <div class="calc-item" onclick="selectGrain('Soja',110.00,'🫘')"><div class="grain">🫘</div><div class="name">Soja</div><div class="price">R$ 110,00/saca</div></div>
                                <div class="calc-item" onclick="selectGrain('Milho',64.21,'🌽')"><div class="grain">🌽</div><div class="name">Milho</div><div class="price">R$ 64,21/saca</div></div>
                                <div class="calc-item" onclick="selectGrain('Feijão Carioca',271.00,'🫘')"><div class="grain">🫘</div><div class="name">Feijão Carioca</div><div class="price">R$ 271,00/saca</div></div>
                                <div class="calc-item" onclick="selectGrain('Feijão Preto',192.00,'🫘')"><div class="grain">🫘</div><div class="name">Feijão Preto</div><div class="price">R$ 192,00/saca</div></div>
                            </div>
                            <div class="calc-row" style="margin-top:14px;"><label>🌾 Grão:</label><input type="text" id="calcGrain" readonly placeholder="Selecione um grão"></div>
                            <div class="calc-row"><label>📦 Sacas:</label><input type="number" id="calcSacas" placeholder="Quantidade de sacas" oninput="calcular()" min="0" step="1"><span class="suffix">sacas</span></div>
                            <div class="calc-row"><label>💵 Preço/saca:</label><input type="text" id="calcPrice" readonly placeholder="R$ 0,00"></div>
                            <div class="calc-result" id="calcResult"><div class="lbl">VALOR TOTAL</div><div class="total">R$ 0,00</div></div>
                        </div>
                    </div>
                </div>
                <div class="tab-content" id="tabDiary">
                    <div class="diary-wrap">
                        <div id="diaryContent"></div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
    const DB = JSON.parse(localStorage.getItem('saberrural_db') || '{"users":{}}');
    const COMMUNITY = JSON.parse(localStorage.getItem('saberrural_community') || '[]');
    const DIARY = JSON.parse(localStorage.getItem('saberrural_diary') || '[]');
    let user = null;
    const ctx = { region:null, state:null, biome:null, city:null, altitude:null, history:[], cityCache:{}, polygonCache:{} };

    const CAPITAIS = {AC:{nome:'Rio Branco',alt:153,chuva:2100},AL:{nome:'Maceió',alt:7,chuva:1800},AP:{nome:'Macapá',alt:14,chuva:2500},AM:{nome:'Manaus',alt:92,chuva:2300},BA:{nome:'Salvador',alt:8,chuva:1900},CE:{nome:'Fortaleza',alt:16,chuva:1500},DF:{nome:'Brasília',alt:1172,chuva:1500},ES:{nome:'Vitória',alt:12,chuva:1200},GO:{nome:'Goiânia',alt:749,chuva:1500},MA:{nome:'São Luís',alt:24,chuva:2100},MT:{nome:'Cuiabá',alt:165,chuva:1400},MS:{nome:'Campo Grande',alt:532,chuva:1400},MG:{nome:'Belo Horizonte',alt:852,chuva:1400},PA:{nome:'Belém',alt:10,chuva:2800},PB:{nome:'João Pessoa',alt:40,chuva:1800},PR:{nome:'Curitiba',alt:934,chuva:1500},PE:{nome:'Recife',alt:4,chuva:2200},PI:{nome:'Teresina',alt:72,chuva:1300},RJ:{nome:'Rio de Janeiro',alt:2,chuva:1200},RN:{nome:'Natal',alt:30,chuva:1600},RS:{nome:'Porto Alegre',alt:10,chuva:1400},RO:{nome:'Porto Velho',alt:83,chuva:2200},RR:{nome:'Boa Vista',alt:90,chuva:1800},SC:{nome:'Florianópolis',alt:3,chuva:1500},SP:{nome:'São Paulo',alt:760,chuva:1400},SE:{nome:'Aracaju',alt:4,chuva:1600},TO:{nome:'Palmas',alt:230,chuva:1700}};
    const CHUVA = {AC:2100,AL:1800,AP:2500,AM:2300,BA:1200,CE:900,DF:1500,ES:1200,GO:1500,MA:1800,MT:1400,MS:1400,MG:1400,PA:2800,PB:1600,PR:1500,PE:1800,PI:1100,RJ:1200,RN:1200,RS:1400,RO:2200,RR:1800,SC:1500,SP:1400,SE:1400,TO:1700};
    const TEMP = {Norte:{min:22,max:32},Nordeste:{min:20,max:33},CentroOeste:{min:18,max:32},Sudeste:{min:15,max:28},Sul:{min:8,max:24}};
    const BIOMAS = {Amazônia:{color:'#1a5632',desc:'Floresta Tropical Úmida',area:'49%',crops:'Mandioca, Açaí, Cupuaçu, Guaraná',clima:'22-32°C, 2500mm/ano',estados:'AC,AM,AP,PA,RO,RR,MT,MA,TO',solo:'Latossolo Amarelo, Argissolo',relevo:'Planícies e baixos planaltos'},Cerrado:{color:'#c2b280',desc:'Savana Tropical',area:'24%',crops:'Soja, Milho, Algodão, Sorgo',clima:'18-32°C, 1500mm/ano',estados:'GO,DF,MS,MT,TO,MG,BA,MA,PI,SP,PR',solo:'Latossolo Vermelho-Amarelo',relevo:'Planaltos e chapadas'},MataAtlântica:{color:'#4a7c3f',desc:'Floresta Costeira',area:'13%',crops:'Café, Cana, Citros, Banana',clima:'15-28°C, 1400mm/ano',estados:'SP,RJ,MG,ES,BA,SE,AL,PE,PB,RN,PR,SC,RS,GO,MS',solo:'Argissolo, Latossolo',relevo:'Serras e planaltos'},Caatinga:{color:'#e8d44d',desc:'Semiárido',area:'10%',crops:'Milho, Feijão, Palma, Algodão',clima:'20-33°C, 800mm/ano',estados:'BA,PI,CE,RN,PB,PE,AL,SE,MG',solo:'Neossolo Litólico, Luvissolo',relevo:'Depressões e planaltos'},Pampa:{color:'#8fbc5a',desc:'Campos Temperados',area:'2%',crops:'Soja, Trigo, Pecuária',clima:'8-24°C, 1300mm/ano',estados:'RS',solo:'Chernossolo, Planossolo',relevo:'Planícies e coxilhas'},Pantanal:{color:'#a5d6a5',desc:'Planície Alagável',area:'2%',crops:'Pecuária, Arroz',clima:'20-35°C, 1100mm/ano',estados:'MT,MS',solo:'Planossolo, Gleissolo',relevo:'Planície alagável'}};

    // ==================== CALCULADORA ====================
    let currentGrain={name:'',price:0};
    function selectGrain(name,price,emoji){currentGrain={name,price};document.getElementById('calcGrain').value=emoji+' '+name;document.getElementById('calcPrice').value='R$ '+price.toFixed(2).replace('.',',');calcular();}
    function calcular(){const sacas=parseFloat(document.getElementById('calcSacas').value)||0;const total=sacas*currentGrain.price;document.getElementById('calcResult').innerHTML=`<div class="lbl">${currentGrain.name?currentGrain.name+' - VALOR TOTAL':'Selecione um grão'}</div><div class="total">R$ ${total.toFixed(2).replace('.',',')}</div>`;}

    // ==================== DIÁRIO DE PLANTIO ====================
    function saveDiary(){localStorage.setItem('saberrural_diary',JSON.stringify(DIARY));}
    function loadDiary(){
        const cont=document.getElementById('diaryContent');
        // Cabeçalho com resumo
        let totalCost=0,totalRevenue=0;
        DIARY.forEach(d=>{totalCost+=d.totalCost||0;totalRevenue+=d.totalRevenue||0;});
        const balance=totalRevenue-totalCost;
        let h='';
        h+=`<div class="summary-card">
            <div class="lbl" style="color:var(--text2);margin-bottom:4px;">SALDO CONSOLIDADO</div>
            <div class="big-number ${balance>=0?'profit':'loss'}">R$ ${balance.toFixed(2).replace('.',',')}</div>
            <div class="summary-grid">
                <div class="summary-item"><div class="val" style="color:#fca5a5;">R$ ${totalCost.toFixed(2).replace('.',',')}</div><div class="lbl">💸 Total Gastos</div></div>
                <div class="summary-item"><div class="val" style="color:var(--green-bright);">R$ ${totalRevenue.toFixed(2).replace('.',',')}</div><div class="lbl">💰 Total Receitas</div></div>
            </div>
            ${DIARY.length>0?`<p style="font-size:11px;color:var(--text2);margin-top:8px;">📊 ${DIARY.length} plantio(s) registrado(s) · ${balance>=0?'🟢 Lucro geral':'🔴 Prejuízo geral'}</p>`:''}
        </div>`;

        // Botão novo plantio
        h+=`<button class="btn btn-green" onclick="showNewDiaryForm()" style="margin-bottom:10px;">📝 Novo Registro de Plantio</button>`;

        // Lista de entradas
        if(!DIARY.length){h+='<div class="empty"><p>📔 Nenhum plantio registrado ainda.<br>Comece clicando em "Novo Registro de Plantio"!</p></div>';}
        else{
            DIARY.slice().reverse().forEach((d,i)=>{
                const originalIdx=DIARY.length-1-i;
                const b=d.totalRevenue-d.totalCost;
                h+=`<div class="diary-entry">
                    <div class="entry-header">
                        <span class="entry-crop">${d.emoji||'🌱'} ${d.crop||'Sem nome'}</span>
                        <span class="entry-date">${d.date}</span>
                    </div>
                    <div class="entry-row"><span>🏔️ Área:</span><span>${d.area||'?'} hectares</span></div>
                    <div class="entry-row"><span>💸 Custos:</span><span>R$ ${(d.totalCost||0).toFixed(2).replace('.',',')}</span></div>
                    <div class="entry-row"><span>💰 Receita:</span><span>R$ ${(d.totalRevenue||0).toFixed(2).replace('.',',')}</span></div>
                    <div class="entry-total ${b>=0?'profit':'loss'}">${b>=0?'🟢 Lucro':'🔴 Prejuízo'}: R$ ${b.toFixed(2).replace('.',',')}</div>
                    <button class="btn btn-red" style="margin-top:8px;padding:6px 12px;font-size:10px;" onclick="deleteDiaryEntry(${originalIdx})">🗑️ Excluir</button>
                </div>`;
            });
        }
        cont.innerHTML=h;
    }

    function showNewDiaryForm(){
        const cont=document.getElementById('diaryContent');
        cont.innerHTML=`
        <button class="btn-back" onclick="loadDiary()">← Voltar ao Diário</button>
        <div class="card"><h3>📝 Novo Registro de Plantio</h3>
        <p style="font-size:11px;color:var(--text2);margin-bottom:12px;">Registre todos os custos e receitas deste plantio</p>
        <div class="field"><label>🌾 Cultura</label><input type="text" id="diaryCrop" placeholder="Ex: Soja, Milho, Café..."></div>
        <div class="field"><label>🏔️ Área (hectares)</label><input type="number" id="diaryArea" placeholder="Ex: 10" min="0" step="0.1"></div>
        <div class="field"><label>📅 Data de Plantio</label><input type="date" id="diaryDate"></div>
        <h4 style="color:var(--text);margin:12px 0 8px;">💸 DESPESAS</h4>
        <div class="field"><label>🌱 Sementes (R$)</label><input type="number" id="costSeeds" placeholder="0,00" min="0" step="0.01" value="0"></div>
        <div class="field"><label>🧪 Fertilizantes (R$)</label><input type="number" id="costFertilizer" placeholder="0,00" min="0" step="0.01" value="0"></div>
        <div class="field"><label>🐛 Defensivos (R$)</label><input type="number" id="costPesticides" placeholder="0,00" min="0" step="0.01" value="0"></div>
        <div class="field"><label>⛽ Combustível (R$)</label><input type="number" id="costFuel" placeholder="0,00" min="0" step="0.01" value="0"></div>
        <div class="field"><label>👷 Mão de obra (R$)</label><input type="number" id="costLabor" placeholder="0,00" min="0" step="0.01" value="0"></div>
        <div class="field"><label>📦 Outros custos (R$)</label><input type="number" id="costOther" placeholder="0,00" min="0" step="0.01" value="0"></div>
        <h4 style="color:var(--text);margin:12px 0 8px;">💰 RECEITAS</h4>
        <div class="field"><label>📦 Sacas colhidas</label><input type="number" id="revSacas" placeholder="0" min="0" step="1" value="0"></div>
        <div class="field"><label>💵 Preço por saca (R$)</label><input type="number" id="revPrice" placeholder="0,00" min="0" step="0.01" value="0"></div>
        <button class="btn btn-green" onclick="finalizeDiary()" style="margin-top:12px;">✅ Finalizar Plantio</button>
        </div>`;
        document.getElementById('diaryDate').value=new Date().toISOString().split('T')[0];
    }

    function finalizeDiary(){
        const crop=document.getElementById('diaryCrop').value.trim()||'Sem nome';
        const area=parseFloat(document.getElementById('diaryArea').value)||0;
        const date=document.getElementById('diaryDate').value||new Date().toISOString().split('T')[0];
        const costSeeds=parseFloat(document.getElementById('costSeeds').value)||0;
        const costFertilizer=parseFloat(document.getElementById('costFertilizer').value)||0;
        const costPesticides=parseFloat(document.getElementById('costPesticides').value)||0;
        const costFuel=parseFloat(document.getElementById('costFuel').value)||0;
        const costLabor=parseFloat(document.getElementById('costLabor').value)||0;
        const costOther=parseFloat(document.getElementById('costOther').value)||0;
        const totalCost=costSeeds+costFertilizer+costPesticides+costFuel+costLabor+costOther;
        const revSacas=parseFloat(document.getElementById('revSacas').value)||0;
        const revPrice=parseFloat(document.getElementById('revPrice').value)||0;
        const totalRevenue=revSacas*revPrice;
        const emojis={soja:'🫘',milho:'🌽',café:'☕',feijão:'🫘',arroz:'🍚',trigo:'🌾',algodão:'🧶'};
        let emoji='🌱';
        for(const[k,v]of Object.entries(emojis)){if(crop.toLowerCase().includes(k)){emoji=v;break;}}
        const formattedDate=new Date(date+'T00:00:00').toLocaleDateString('pt-BR');
        DIARY.push({crop,area,date:formattedDate,totalCost,totalRevenue,emoji,costs:{seeds:costSeeds,fertilizer:costFertilizer,pesticides:costPesticides,fuel:costFuel,labor:costLabor,other:costOther},revenue:{sacas:revSacas,price:revPrice}});
        saveDiary();
        loadDiary();
    }

    function deleteDiaryEntry(idx){
        if(confirm('Tem certeza que deseja excluir este registro?')){
            DIARY.splice(idx,1);
            saveDiary();
            loadDiary();
        }
    }

    function moon(){const p=['🌑 Nova','🌒 Crescente','🌓 Quarto Crescente','🌔 Gibosa','🌕 Cheia','🌖 Gibosa Minguante','🌗 Quarto Minguante','🌘 Minguante'];return p[new Date().getDate()%8];}
    function clima(regiao,alt=500){const b=TEMP[regiao]||TEMP.Sudeste;const v=()=>Math.floor(Math.random()*5)-2;const adj=Math.floor(alt/200)*-1;const t=Math.floor((b.min+b.max)/2)+v()+adj;const conds=[{d:'Céu limpo',i:'☀️',u:35+Math.floor(Math.random()*25)},{d:'Parcialmente nublado',i:'⛅',u:50+Math.floor(Math.random()*25)},{d:'Nublado',i:'☁️',u:68+Math.floor(Math.random()*18)},{d:'Chuva leve',i:'🌦️',u:78+Math.floor(Math.random()*14)},{d:'Chuva forte',i:'🌧️',u:85+Math.floor(Math.random()*10)}];const c=conds[Math.floor(Math.random()*conds.length)];return {temp:t,min:b.min+v()+adj,max:b.max+v()+adj,umidade:c.u,desc:c.d,icon:c.i,vento:4+Math.floor(Math.random()*28),sensacao:t-1+Math.floor(Math.random()*3),uv:Math.floor(Math.random()*11),pressao:1010+Math.floor(Math.random()*20)};}

    let map, stateLayer, biomesLayer, selectedState=null, estadoGeom={}, biomeMap={}, cityLayer=null, cityPolyMap={}, currentCityPolygons=[];

    function initMap(){map=L.map('map',{center:[-14,-52],zoom:5,zoomControl:true,preferCanvas:true});L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Shaded_Relief/MapServer/tile/{z}/{y}/{x}',{maxZoom:17}).addTo(map);L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Terrain_Base/MapServer/tile/{z}/{y}/{x}',{maxZoom:17,opacity:0.55}).addTo(map);L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Hillshade/MapServer/tile/{z}/{y}/{x}',{maxZoom:17,opacity:0.28}).addTo(map);loadStates();buildLegend();}

    async function loadStates(){try{const r=await fetch('https://servicodados.ibge.gov.br/api/v1/localidades/estados');const estados=await r.json();const feats=[];for(const e of estados){try{const gr=await fetch(`https://servicodados.ibge.gov.br/api/v3/malhas/estados/${e.sigla}?formato=application/vnd.geo+json&qualidade=intermediaria`);if(gr.ok){const gd=await gr.json();if(gd.features?.length){gd.features[0].properties={sigla:e.sigla,nome:e.nome,regiao:e.regiao.nome};feats.push(gd.features[0]);estadoGeom[e.sigla]=gd.features[0];}}}catch(_){}await new Promise(r=>setTimeout(r,35));}if(stateLayer)map.removeLayer(stateLayer);stateLayer=L.geoJSON({type:'FeatureCollection',features:feats},{style:()=>({color:'#f59e0b',weight:2.5,fillOpacity:0.04,dashArray:'5 4'}),onEachFeature:(f,l)=>{const cap=CAPITAIS[f.properties.sigla];const cl=clima(f.properties.regiao,cap?.alt||500);l.bindTooltip(`<b>${f.properties.nome}</b><br>🏛️ ${cap?.nome||'N/D'}<br>🌡️ ${cl.temp}°C ${cl.icon}<br>🏔️ ${cap?.alt||'?'}m`);l.on('click',()=>{if(selectedState){selectedState.setStyle({fillOpacity:0.04,weight:2.5});}selectedState=l;l.setStyle({fillOpacity:0.28,weight:5,fillColor:'#fbbf24'});map.fitBounds(l.getBounds(),{padding:[45,45],maxZoom:8});ctx.state=f.properties.nome;ctx.region=f.properties.regiao;showStateInfo(f.properties.sigla,f.properties.nome,f.properties.regiao);});}}).addTo(map);drawBiomes();}catch(e){console.error(e);}}

    function drawBiomes(){if(biomesLayer)map.removeLayer(biomesLayer);biomeMap={};const all=[];Object.entries(BIOMAS).forEach(([name,data])=>{const siglas=data.estados.split(',');const feats=siglas.map(s=>estadoGeom[s]).filter(f=>f);if(!feats.length)return;const geo={type:'FeatureCollection',features:feats.map(f=>({type:'Feature',geometry:f.geometry}))};const layer=L.geoJSON(geo,{style:{color:data.color,weight:3.5,fillColor:data.color,fillOpacity:0.1},onEachFeature:(f,l)=>{l.bindTooltip(`<b>🌿 ${name}</b><br>${data.desc}<br>📐 ${data.area} do Brasil`);l.on('click',()=>{ctx.biome=name;showBiomeInfo(name);});l.on('mouseover',function(){l.setStyle({fillOpacity:0.3,weight:5});});l.on('mouseout',function(){l.setStyle({fillOpacity:0.1,weight:3.5});});}});biomeMap[name]=layer;all.push(layer);});biomesLayer=L.layerGroup(all).addTo(map);if(stateLayer)stateLayer.bringToFront();}
    function showBiomeOnMap(name){const l=biomeMap[name];if(l){map.fitBounds(l.getBounds(),{padding:[35,35],maxZoom:7});l.setStyle({fillOpacity:0.4,weight:6});setTimeout(()=>l.setStyle({fillOpacity:0.1,weight:3.5}),2200);}}
    function buildLegend(){const leg=document.getElementById('legend');leg.innerHTML='<h4>🌿 Biomas</h4>';Object.entries(BIOMAS).forEach(([name,b])=>{const row=document.createElement('div');row.className='legend-row';row.innerHTML=`<div class="legend-color" style="background:${b.color};box-shadow:0 0 10px ${b.color};"></div>${name}`;row.onclick=()=>{showBiomeOnMap(name);showBiomeInfo(name);};leg.appendChild(row);});}

    async function showStateInfo(uf,name,regiao){const cont=document.getElementById('infoContent');cont.innerHTML='<div class="spinner"></div>';if(ctx.cityCache[uf]){renderStateView(uf,name,regiao,ctx.cityCache[uf]);loadCityTerritories(uf,ctx.cityCache[uf]);return;}try{const r=await fetch(`https://servicodados.ibge.gov.br/api/v1/localidades/estados/${uf}/municipios`);const cities=await r.json();ctx.cityCache[uf]=cities;renderStateView(uf,name,regiao,cities);loadCityTerritories(uf,cities);}catch(_){cont.innerHTML='<div class="empty"><p>Erro ao carregar</p></div>';}}
    async function loadCityTerritories(uf,cities){if(cityLayer){map.removeLayer(cityLayer);cityLayer=null;}cityPolyMap={};currentCityPolygons=[];const feats=[];const sample=cities.slice(0,80);for(const c of sample){if(ctx.polygonCache[c.id]){feats.push(ctx.polygonCache[c.id]);continue;}try{const gr=await fetch(`https://servicodados.ibge.gov.br/api/v3/malhas/municipios/${c.id}?formato=application/vnd.geo+json&qualidade=simplificada`);if(gr.ok){const gd=await gr.json();if(gd.features?.length){gd.features[0].properties={nome:c.nome,id:c.id};feats.push(gd.features[0]);ctx.polygonCache[c.id]=gd.features[0];}}}catch(_){}await new Promise(r=>setTimeout(r,15));}if(feats.length){cityLayer=L.geoJSON({type:'FeatureCollection',features:feats},{style:()=>({color:'#60a5fa',weight:1.5,fillColor:'#3b82f6',fillOpacity:0.06,dashArray:'3 3'}),onEachFeature:(f,l)=>{const nome=f.properties.nome;cityPolyMap[nome]=l;currentCityPolygons.push({layer:l,nome:nome});l.bindTooltip(`<b>🏙️ ${nome}</b><br>Clique para zoom`,{sticky:true});l.on('click',function(e){L.DomEvent.stopPropagation(e);showCityInfo(nome,uf,ctx.region,CAPITAIS[uf]?.alt||500);zoomToCity(l,nome);});l.on('mouseover',function(){l.setStyle({fillOpacity:0.22,weight:3,color:'#fbbf24'});});l.on('mouseout',function(){l.setStyle({fillOpacity:0.06,weight:1.5,color:'#60a5fa'});});}}).addTo(map);}}
    function zoomToCity(layer,nome){map.fitBounds(layer.getBounds(),{padding:[40,40],maxZoom:13});currentCityPolygons.forEach(c=>{if(c.layer&&c.layer.setStyle){c.layer.setStyle({fillOpacity:0.06,weight:1.5,color:'#60a5fa'});}});layer.setStyle({fillOpacity:0.45,weight:5,color:'#fbbf24',fillColor:'#f59e0b'});layer.bringToFront();setTimeout(()=>{layer.setStyle({fillOpacity:0.06,weight:1.5,color:'#60a5fa',fillColor:'#3b82f6'});},3000);document.querySelectorAll('.mun-item').forEach(i=>i.classList.remove('capital'));document.querySelectorAll('.mun-item').forEach(i=>{if(i.textContent.includes(nome)){i.classList.add('capital');i.scrollIntoView({behavior:'smooth',block:'center'});}});}
    function showCityInfo(name,uf,regiao,alt){ctx.city=name;ctx.altitude=alt;const cl=clima(regiao,alt);document.getElementById('infoContent').innerHTML=`<button class="btn-back" onclick="resetInfo()">← Voltar</button><div class="card"><h3>🏙️ ${name} (${uf})</h3><span class="tag tag-blue">${regiao}</span><span class="tag tag-gold">🏔️ ${alt}m</span><span class="tag tag-green">🌧️ ${CHUVA[uf]||'?'}mm/ano</span><span class="tag tag-red">🌡️ ${cl.temp}°C</span><p style="margin-top:6px;font-size:11px;color:var(--text2);">🌙 ${moon()} · UV: ${cl.uv} · Pressão: ${cl.pressao}hPa</p></div><div class="clima-card"><div class="clima-icon">${cl.icon}</div><div><div class="clima-temp">${cl.temp}°C</div><div>${cl.desc}</div><div class="clima-grid"><div class="clima-item"><div class="val">${cl.min}°/${cl.max}°</div><div class="lbl">Mín/Máx</div></div><div class="clima-item"><div class="val">${cl.umidade}%</div><div class="lbl">Umidade</div></div><div class="clima-item"><div class="val">${cl.vento}km/h</div><div class="lbl">Vento</div></div><div class="clima-item"><div class="val">${cl.sensacao}°C</div><div class="lbl">Sensação</div></div></div></div></div><div class="search-box"><input type="text" id="citySearch" placeholder="🔍 Pesquisar município..." oninput="filterCities()"><div class="results-count" id="resultsCount">${ctx.cityCache[uf]?.length||'?'} municípios</div></div><div class="mun-list" id="munList">${renderCityListItems(uf)}</div>`;const poly=cityPolyMap[name];if(poly){zoomToCity(poly,name);}else if(cityLayer){cityLayer.eachLayer(l=>{if(l.feature&&l.feature.properties.nome===name){zoomToCity(l,name);}});}}
    function renderCityListItems(uf){const cities=ctx.cityCache[uf]||[];const cap=CAPITAIS[uf];const capNome=cap?.nome||cities[0]?.nome||'N/D';return cities.map(c=>{const isCap=c.nome===capNome||c.nome===ctx.city;return `<div class="mun-item${isCap?' capital':''}" data-nome="${c.nome.toLowerCase()}" onclick="showCityInfo('${c.nome.replace(/'/g,"\\'")}','${uf}','${ctx.region}',${cap?.alt||500})">${isCap?'⭐':''}${c.nome}</div>`;}).join('');}
    function renderStateView(uf,name,regiao,cities){const cap=CAPITAIS[uf];const capNome=cap?.nome||cities[0]?.nome||'N/D';const chuv=CHUVA[uf]||1500;const cl=clima(regiao,cap?.alt||500);const cont=document.getElementById('infoContent');let h=`<button class="btn-back" onclick="resetInfo()">← Voltar</button><div class="card"><h3>📍 ${name} (${uf})</h3><span class="tag tag-green">${cities.length} municípios</span><span class="tag tag-gold">🏛️ ${capNome}</span><span class="tag tag-blue">🌧️ ${chuv}mm/ano</span><span class="tag tag-red">🌡️ ${cl.temp}°C</span><p style="margin-top:6px;font-size:11px;color:var(--text2);">🏔️ ${cap?.alt||'?'}m · 🌙 ${moon()} · UV: ${cl.uv}</p></div><div class="clima-card"><div class="clima-icon">${cl.icon}</div><div><div class="clima-temp">${cl.temp}°C</div><div>${cl.desc} · ${regiao}</div><div class="clima-grid"><div class="clima-item"><div class="val">${cl.min}°/${cl.max}°</div><div class="lbl">Mín/Máx</div></div><div class="clima-item"><div class="val">${cl.umidade}%</div><div class="lbl">Umidade</div></div><div class="clima-item"><div class="val">${cl.vento}km/h</div><div class="lbl">Vento</div></div><div class="clima-item"><div class="val">${cl.sensacao}°C</div><div class="lbl">Sensação</div></div></div></div></div><div class="search-box"><input type="text" id="citySearch" placeholder="🔍 Pesquisar município..." oninput="filterCities()"><div class="results-count" id="resultsCount">${cities.length} municípios</div></div><div class="mun-list" id="munList">${renderCityListItems(uf)}</div>`;cont.innerHTML=h;setTimeout(()=>{const si=document.getElementById('citySearch');if(si)si.focus();},100);}
    window.filterCities=function(){const t=document.getElementById('citySearch')?.value.toLowerCase()||'';const items=document.querySelectorAll('#munList .mun-item');let c=0;items.forEach(i=>{if(i.getAttribute('data-nome')?.includes(t)){i.classList.remove('hidden-item');c++;}else{i.classList.add('hidden-item');}});const rc=document.getElementById('resultsCount');if(rc)rc.textContent=t?`${c} encontrado(s)`:`${items.length} municípios`;};
    function showBiomeInfo(name){const b=BIOMAS[name];if(!b)return;showBiomeOnMap(name);const cl=clima('CentroOeste',500);document.getElementById('infoContent').innerHTML=`<button class="btn-back" onclick="resetInfo()">← Voltar</button><div class="card" style="border-left:5px solid ${b.color};"><h3>🌿 ${name}</h3><span class="tag tag-green">${b.desc}</span><span class="tag tag-gold">📐 ${b.area} do Brasil</span><p style="margin-top:8px;font-size:12px;">🌤️ ${b.clima}</p><p style="font-size:12px;">🌾 Culturas: ${b.crops}</p><p style="font-size:11px;color:var(--text2);">🧪 Solo: ${b.solo}</p><p style="font-size:11px;color:var(--text2);">⛰️ Relevo: ${b.relevo}</p><p style="font-size:11px;color:var(--text2);">🌙 ${moon()} · ${b.estados.split(',').length} estados</p></div><div class="clima-card"><div class="clima-icon">${cl.icon}</div><div><div class="clima-temp">${cl.temp}°C</div><div>${cl.desc}</div><div class="clima-grid"><div class="clima-item"><div class="val">${cl.min}°/${cl.max}°</div><div class="lbl">Mín/Máx</div></div><div class="clima-item"><div class="val">${cl.umidade}%</div><div class="lbl">Umidade</div></div><div class="clima-item"><div class="val">${cl.vento}km/h</div><div class="lbl">Vento</div></div><div class="clima-item"><div class="val">${cl.sensacao}°C</div><div class="lbl">Sensação</div></div></div></div></div>`;}
    function resetInfo(){ctx.biome=null;ctx.state=null;ctx.city=null;if(selectedState){selectedState.setStyle({fillOpacity:0.04,weight:2.5});selectedState=null;}if(cityLayer){map.removeLayer(cityLayer);cityLayer=null;}cityPolyMap={};currentCityPolygons=[];document.getElementById('infoContent').innerHTML='<div class="empty"><p>🗺️ Clique em um estado para ver seus municípios</p></div>';}

    // ==================== COMUNIDADE ====================
    function saveCommunity(){localStorage.setItem('saberrural_community',JSON.stringify(COMMUNITY));}
    function loadCommunity(){const container=document.getElementById('communityMsgs');container.innerHTML='';if(!COMMUNITY.length){container.innerHTML='<div class="empty"><p>👨‍🌾 Nenhuma mensagem ainda.<br><small>Use @AgroIA para falar com a IA</small></p></div>';return;}COMMUNITY.forEach(msg=>{const div=document.createElement('div');div.className=`community-msg${msg.type==='bot'?' bot-msg':''}`;const isBot=msg.type==='bot';const inicial=isBot?'🤖':(msg.name?msg.name.charAt(0).toUpperCase():'?');const nameDisplay=isBot?'AgroIA':(msg.name||'Anônimo');div.innerHTML=`<div class="msg-header"><div class="msg-avatar ${isBot?'bot-av':'user-av'}">${inicial}</div><span class="msg-name ${isBot?'bot-name':''}">${nameDisplay}</span><span class="msg-time">${msg.time}</span></div><div class="msg-body">${msg.text.replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/\n/g,'<br>')}</div>`;container.appendChild(div);});container.scrollTop=container.scrollHeight;}
    async function sendCommunityMsg(){const inp=document.getElementById('communityInput');const text=inp.value.trim();if(!text||!user)return;const now=new Date();const time=now.toLocaleDateString('pt-BR')+' '+now.toLocaleTimeString('pt-BR',{hour:'2-digit',minute:'2-digit'});if(text.startsWith('@AgroIA')){const pergunta=text.replace('@AgroIA','').trim();COMMUNITY.push({name:user.name,text:text,time:time,type:'user'});if(pergunta){const resposta=aiReply(pergunta);const respTime=new Date().toLocaleDateString('pt-BR')+' '+new Date().toLocaleTimeString('pt-BR',{hour:'2-digit',minute:'2-digit'});COMMUNITY.push({name:'AgroIA',text:resposta,time:respTime,type:'bot'});}if(COMMUNITY.length>300)COMMUNITY.splice(0,COMMUNITY.length-300);saveCommunity();inp.value='';loadCommunity();return;}COMMUNITY.push({name:user.name,text:text,time:time,type:'user'});if(COMMUNITY.length>300)COMMUNITY.splice(0,COMMUNITY.length-300);saveCommunity();inp.value='';loadCommunity();}

    // ==================== IA ====================
    function aiReply(q){const ql=q.toLowerCase().trim();const ci=ctx;if(/^(oi|olá|ola|bom dia|boa tarde|boa noite|e aí|hey)/i.test(ql)&&q.length<35){const h=new Date().getHours();let s='Olá';if(h<12)s='Bom dia';else if(h<18)s='Boa tarde';else s='Boa noite';let r=`${s}! 👋 Bem-vindo ao **Saber Rural**!\n\n`;if(ci.state){const cl=clima(ci.region||'Sudeste',ci.altitude||500);r+=`🌍 Em **${ci.state}** agora: ${cl.temp}°C ${cl.icon} ${cl.desc}\n🌧️ Chuva anual: ${CHUVA[Object.keys(CAPITAIS).find(k=>CAPITAIS[k].nome===ci.state)||'SP']||'?'}mm\n\n`;}r+='Posso ajudar com:\n🌾 Culturas e plantio\n🌤️ Clima\n🌿 Biomas\n💧 Irrigação\n🐛 Pragas\n💰 Mercado\n🧮 Calculadora de grãos\n📔 Diário de plantio\n\n**O que você quer saber?**';return r;}if(ql.includes('clima')||ql.includes('temperatura')||ql.includes('quantos graus')){if(ci.state||ci.city){const cl=clima(ci.region||'Sudeste',ci.altitude||500);return `🌤️ **Clima em ${ci.city||ci.state}**\n\n🌡️ **${cl.temp}°C** ${cl.icon}\n📊 Mín: ${cl.min}°C / Máx: ${cl.max}°C\n💧 Umidade: ${cl.umidade}%\n🌬️ Vento: ${cl.vento}km/h\n🤔 Sensação: ${cl.sensacao}°C\n☀️ UV: ${cl.uv}\n🔵 Pressão: ${cl.pressao}hPa\n📝 ${cl.desc}`;}return '🌤️ Selecione um estado ou município no mapa!';}if(ql.includes('bioma')){for(const[n,b]of Object.entries(BIOMAS)){if(ql.includes(n.toLowerCase())){return `🌿 **${n}**\n\n📝 ${b.desc}\n📐 ${b.area} do Brasil\n🌤️ ${b.clima}\n🌾 Culturas: ${b.crops}\n🧪 Solo: ${b.solo}\n⛰️ Relevo: ${b.relevo}`;}}let r='🌿 **Biomas:**\n\n';Object.entries(BIOMAS).forEach(([n,b])=>r+=`• **${n}** (${b.area}): ${b.desc}\n`);return r;}if(ql.includes('plantar')||ql.includes('soja')||ql.includes('milho')||ql.includes('café')){if(ql.includes('soja'))return '🫘 **Soja**\n🌡️ 20-30°C | 🌧️ 450-800mm\n📅 Plantio: Out-Nov\n💰 R\\$ 110,00/saca (60kg)\n📊 Use nossa calculadora na aba 🧮!';if(ql.includes('milho'))return '🌽 **Milho**\n🌡️ 20-30°C | 🌧️ 500-800mm\n📅 Safra: Out-Nov / Safrinha: Jan-Fev\n💰 R\\$ 64,21/saca (60kg)\n📊 Use nossa calculadora na aba 🧮!';if(ql.includes('café'))return '☕ **Café**\n🌡️ 18-22°C | 🌧️ 1200-1800mm\n📅 Plantio: Out-Nov\n🏔️ Ideal acima de 800m\n💰 R\\$ 8-15 mil/ha';}if(/^(obrigad|valeu)/i.test(ql))return '😊 De nada! 🌾';return `🤔 Sobre **"${q}"**:\n\n${ci.state?`📍 ${ci.state}: ${clima(ci.region,ci.altitude||500).temp}°C`:'🗺️ Selecione um estado!'}\n\nPosso ajudar com clima, biomas, culturas, calculadora, diário de plantio e mais!`;}
    function addMsg(text,type){const c=document.getElementById('chatMsgs');const d=document.createElement('div');d.className=`msg ${type}`;d.innerHTML=`<div class="avatar">${type==='bot'?'🤖':'👤'}</div><div class="bubble">${text.replace(/\n/g,'<br>').replace(/\*\*(.*?)\*\*/g,'<b>$1</b>')}</div>`;c.appendChild(d);c.scrollTop=c.scrollHeight;}
    async function sendMsg(){const inp=document.getElementById('chatInput');const q=inp.value.trim();if(!q)return;addMsg(q,'user');inp.value='';ctx.history.push({role:'user',content:q});const typing=document.createElement('div');typing.className='msg bot';typing.innerHTML='<div class="avatar">🤖</div><div class="bubble"><div class="typing-dots"><span></span><span></span><span></span></div></div>';document.getElementById('chatMsgs').appendChild(typing);await new Promise(r=>setTimeout(r,700+Math.random()*1800));typing.remove();const resp=aiReply(q);ctx.history.push({role:'assistant',content:resp});addMsg(resp,'bot');}

    function saveDB(){localStorage.setItem('saberrural_db',JSON.stringify(DB));}
    function toggleEye(id,btn){const inp=document.getElementById(id);inp.type=inp.type==='password'?'text':'password';btn.textContent=inp.type==='password'?'👁️':'🙈';}
    function showReg(){document.getElementById('loginFields').classList.add('hidden');document.getElementById('regFields').classList.remove('hidden');}
    function showLogin(){document.getElementById('regFields').classList.add('hidden');document.getElementById('loginFields').classList.remove('hidden');}
    function doLogin(){const e=document.getElementById('loginEmail').value.trim();const p=document.getElementById('loginPass').value.trim();if(!e||!p)return alert('Preencha os campos!');const u=DB.users[e];if(!u||u.pass!==btoa(p))return alert('Credenciais inválidas!');user={email:e,name:u.name};localStorage.setItem('saberrural_user',JSON.stringify(user));enterApp();}
    function doRegister(){const n=document.getElementById('regName').value.trim();const e=document.getElementById('regEmail').value.trim();const p=document.getElementById('regPass').value.trim();if(!n||!e||p.length<4)return alert('Preencha corretamente!');if(DB.users[e])return alert('Email já cadastrado!');DB.users[e]={name:n,pass:btoa(p),createdAt:new Date().toISOString()};saveDB();alert('✅ Conta criada com sucesso!');showLogin();}
    function recoverPass(){const e=prompt('Digite seu email:');if(!e)return;const u=DB.users[e];if(!u)return alert('Email não encontrado!');alert(`🔑 Sua senha: ${atob(u.pass)}`);}
    function logout(){user=null;localStorage.removeItem('saberrural_user');document.getElementById('app').classList.remove('active');document.getElementById('loginScreen').classList.remove('hidden');showLogin();}
    function enterApp(){document.getElementById('loginScreen').classList.add('hidden');document.getElementById('app').classList.add('active');document.getElementById('userBadge').textContent='👤 '+user.name.split(' ')[0];if(!map)initMap();loadCommunity();loadDiary();}
    function switchTab(tab){
        document.querySelectorAll('.tab').forEach(b=>b.classList.remove('active'));
        document.querySelectorAll('.tab-content').forEach(c=>c.classList.remove('active'));
        const tabs=['info','community','chat','calc','diary'];
        const idx=tabs.indexOf(tab);
        if(idx>=0){
            document.querySelectorAll('.tab')[idx].classList.add('active');
            const tabIds={info:'tabInfo',community:'tabCommunity',chat:'tabChat',calc:'tabCalc',diary:'tabDiary'};
            document.getElementById(tabIds[tab]).classList.add('active');
        }
        if(tab==='community')loadCommunity();
        if(tab==='diary')loadDiary();
    }

    window.doLogin=doLogin;window.doRegister=doRegister;window.showReg=showReg;window.showLogin=showLogin;
    window.recoverPass=recoverPass;window.logout=logout;window.toggleEye=toggleEye;window.switchTab=switchTab;
    window.sendMsg=sendMsg;window.resetInfo=resetInfo;window.showCityInfo=showCityInfo;window.showBiomeInfo=showBiomeInfo;
    window.filterCities=filterCities;window.sendCommunityMsg=sendCommunityMsg;
    window.selectGrain=selectGrain;window.calcular=calcular;
    window.showNewDiaryForm=showNewDiaryForm;window.finalizeDiary=finalizeDiary;window.deleteDiaryEntry=deleteDiaryEntry;window.loadDiary=loadDiary;

    const saved=localStorage.getItem('saberrural_user');
    if(saved){user=JSON.parse(saved);enterApp();}
    </script>
</body>
</html>
