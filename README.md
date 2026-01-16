<code>
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Élé-mentale</title>
    <meta name="description" content="Jeu d'apprentissage du tableau périodique">
    <style>
        :root {
            --primary: #4f46e5;
            --primary-light: #818cf8;
            --mentale: #7c3aed; /* Purple for Mentale Mode */
            --success: #16a34a;
            --warning: #d97706;
            --error: #dc2626;
            --bg: #f8fafc;
            --card: #ffffff;
            --text-main: #1e293b;
            --text-muted: #64748b;
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background-color: var(--bg);
            color: var(--text-main);
            height: 100dvh;
            margin: 0;
            display: flex;
            align-items: center;
            justify-content: center;
            overflow: hidden;
            touch-action: manipulation;
        }

        #fireworks {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            pointer-events: none; z-index: 50;
        }

        .container {
            width: 100%; max-width: 600px;
            height: 100dvh;
            display: flex; flex-direction: column;
            background: var(--card);
            position: relative; z-index: 10;
            box-shadow: 0 0 20px rgba(0,0,0,0.05);
            padding: 1rem;
            padding-bottom: calc(1rem + env(safe-area-inset-bottom));
        }

        @media (min-width: 600px) {
            .container {
                height: 90vh; max-height: 850px;
                border-radius: 16px;
                border: 1px solid #e2e8f0;
            }
        }

        h1 { font-size: 1.4rem; margin: 0; color: var(--primary); font-weight: 800; letter-spacing: -0.5px; }
        .font-serif { font-family: 'Georgia', serif; }

        .header-bar {
            display: flex; justify-content: space-between; align-items: center;
            padding-bottom: 0.8rem; border-bottom: 1px solid #f1f5f9;
            flex-shrink: 0;
        }
        
        .header-controls { display: flex; align-items: center; gap: 0.5rem; }
        
        .timer { 
            font-family: 'Courier New', Courier, monospace;
            font-variant-numeric: tabular-nums; 
            font-weight: 700; color: var(--text-main); font-size: 1.3rem;
            opacity: 1; transition: opacity 0.3s;
            background: #f1f5f9; padding: 2px 8px; border-radius: 6px;
            margin-right: 0.3rem;
        }
        .timer.hidden { opacity: 0; }

        .btn-icon {
            background: #f1f5f9; border: none; color: var(--text-muted);
            width: 40px; height: 40px; border-radius: 50%;
            font-size: 1.2rem; cursor: pointer; display: flex; align-items: center; justify-content: center;
            transition: background 0.2s, transform 0.2s;
        }
        .btn-icon:active { background: #e2e8f0; transform: scale(0.95); }
        .btn-icon.active { color: var(--primary); background: #e0e7ff; }

        .stats-row {
            display: flex; justify-content: space-around;
            padding: 0.8rem 0; font-size: 0.9rem; color: var(--text-muted);
            flex-shrink: 0;
        }
        .stat-item { display: flex; flex-direction: column; align-items: center; }
        .stat-val { font-size: 1.2rem; font-weight: 800; line-height: 1; }
        .stat-lbl { font-size: 0.7rem; text-transform: uppercase; margin-top: 2px; letter-spacing: 0.5px; }

        .config-area { flex-shrink: 0; display: flex; flex-direction: column; gap: 0.5rem; margin-bottom: 1rem; }
        .toggle-row { display: flex; gap: 0.5rem; }
        .toggle-group {
            background: #f1f5f9; padding: 3px; border-radius: 8px; display: flex; flex: 1;
        }
        .toggle-btn {
            flex: 1; background: transparent; border: none; padding: 8px;
            font-size: 0.8rem; border-radius: 6px; color: var(--text-muted); font-weight: 600;
            cursor: pointer; transition: all 0.2s; white-space: nowrap;
        }
        .toggle-btn.active { background: white; color: var(--primary); box-shadow: 0 1px 2px rgba(0,0,0,0.1); }
        
        .toggle-btn.mentale-active { background: var(--mentale); color: white; }

        .checkbox-row { display: flex; justify-content: space-between; padding: 0 4px; }
        .chk-label {
            display: flex; align-items: center; gap: 8px; font-size: 0.85rem;
            color: var(--text-muted); font-weight: 500; cursor: pointer;
        }
        .chk-label.hidden { opacity: 0; pointer-events: none; }
        .chk-label input { width: 16px; height: 16px; accent-color: var(--primary); }

        .game-stage {
            flex: 1; display: flex; flex-direction: column; position: relative;
            justify-content: space-between; min-height: 0; 
        }

        .prompt-box {
            font-size: 3.5rem; font-weight: 800; text-align: center;
            color: var(--text-main); margin: 0.5rem 0;
            height: 80px; display: flex; align-items: center; justify-content: center;
        }

        .grid-options {
            display: grid; grid-template-columns: 1fr 1fr; gap: 12px;
            flex: 1; max-height: 45vh; min-height: 220px;
        }
        
        .btn-opt {
            background: white; border: 2px solid #e2e8f0; border-radius: 12px;
            font-size: 1.4rem; font-weight: 600; color: var(--text-main);
            cursor: pointer; transition: all 0.1s;
            display: flex; align-items: center; justify-content: center;
            padding: 0; touch-action: manipulation;
        }
        .btn-opt:active:not(:disabled) { transform: scale(0.98); background: #f8fafc; }
        @media(hover: hover) { .btn-opt:hover:not(:disabled) { border-color: var(--primary-light); background: #f8fafc; } }
        
        .btn-opt.correct { background: var(--success); color: white; border-color: var(--success); }
        .btn-opt.wrong { background: var(--error); color: white; border-color: var(--error); }
        .btn-opt:disabled { opacity: 1; cursor: default; }

        .input-container {
            display: none; flex-direction: column; width: 100%; gap: 10px;
            flex: 1; justify-content: center;
        }
        .input-container.active { display: flex; }
        
        input[type="text"] {
            padding: 1rem; font-size: 1.3rem; border: 2px solid #e2e8f0;
            border-radius: 12px; text-align: center; outline: none;
            width: 100%; -webkit-appearance: none; background: white;
            color: var(--text-main);
        }
        input[type="text"]:focus { border-color: var(--primary); }
        .shake { animation: shake 0.4s cubic-bezier(.36,.07,.19,.97) both; }

        .btn-action {
            background: var(--primary); color: white; border: none;
            padding: 1rem; border-radius: 12px; font-weight: 700; font-size: 1.1rem;
            cursor: pointer; width: 100%; transition: background 0.2s;
        }
        .btn-action:active { background: var(--primary-dark); }

        .feedback-area {
            height: 110px; display: flex; flex-direction: column;
            align-items: center; justify-content: center; flex-shrink: 0;
            text-align: center; padding-top: 10px;
        }
        .fb-msg { font-size: 1.3rem; font-weight: 800; margin-bottom: 4px; min-height: 1.6rem; }
        .fb-sub { font-size: 0.95rem; color: var(--text-muted); font-style: italic; min-height: 1.2rem; }
        
        .btn-next {
            margin-top: 10px; background: var(--text-main); color: white; border: none;
            padding: 12px 32px; border-radius: 30px; font-size: 1rem; font-weight: 600;
            cursor: pointer; opacity: 0; pointer-events: none; transition: opacity 0.2s;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }
        .btn-next.visible { opacity: 1; pointer-events: auto; }

        .overlay {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: var(--card); z-index: 20;
            display: flex; flex-direction: column;
            align-items: center; justify-content: center;
            text-align: center; padding: 1rem;
            transition: opacity 0.3s, visibility 0.3s;
        }
        .overlay.hidden { opacity: 0; visibility: hidden; pointer-events: none; }

        .badges { display: flex; gap: 8px; flex-wrap: wrap; justify-content: center; margin-bottom: 1.5rem; }
        .badge {
            padding: 6px 12px; border-radius: 20px; font-size: 0.75rem; font-weight: 700;
            background: #e0e7ff; color: var(--primary); border: 1px solid #c7d2fe;
        }
        .badge.expert { background: #fef3c7; color: #b45309; border-color: #fcd34d; }
        .badge.mentale { background: var(--mentale); color: white; border-color: var(--mentale); }

        .btn-start {
            background: var(--primary); color: white; border: none;
            padding: 1.2rem 3.5rem; font-size: 1.2rem; font-weight: 800;
            border-radius: 50px; cursor: pointer;
            box-shadow: 0 8px 20px rgba(79, 70, 229, 0.3);
            animation: pulse 2s infinite;
        }

        .end-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; width: 100%; margin: 1.5rem 0; }
        .history-box {
            width: 100%; flex: 1; overflow-y: auto; border: 1px solid #e2e8f0;
            border-radius: 12px; padding: 0.5rem; margin-bottom: 1rem;
            font-size: 0.85rem; background: #f8fafc;
        }
        .hist-row {
            display: flex; justify-content: space-between; align-items: center;
            padding: 8px; border-bottom: 1px solid #e2e8f0;
        }
        .hist-row:last-child { border-bottom: none; }
        .hist-tag { font-size: 0.7rem; background: #cbd5e1; padding: 2px 6px; border-radius: 4px; font-weight: 700; margin-right: 6px; }
        .row-perfect { background-color: #ecfccb; color: #365314; }
        .row-clean { background-color: #f0fdf4; color: #166534; }

        @keyframes shake { 10%, 90% { transform: translate3d(-1px, 0, 0); } 20%, 80% { transform: translate3d(2px, 0, 0); } 30%, 50%, 70% { transform: translate3d(-4px, 0, 0); } 40%, 60% { transform: translate3d(4px, 0, 0); } }
        @keyframes pulse { 0% { transform: scale(1); box-shadow: 0 0 0 0 rgba(79, 70, 229, 0.7); } 70% { transform: scale(1.02); box-shadow: 0 0 0 15px rgba(79, 70, 229, 0); } 100% { transform: scale(1); box-shadow: 0 0 0 0 rgba(79, 70, 229, 0); } }
    </style>
</head>
<body>

<canvas id="fireworks"></canvas>

<div class="container">
    <header class="header-bar">
        <h1>Élé-mentale</h1>
        <div class="header-controls">
            <div class="timer hidden" id="timerDisplay">00:00.00</div>
            <button class="btn-icon" id="btnFullscreen" onclick="app.toggleFullscreen()" aria-label="Plein écran">
                <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M8 3H5a2 2 0 0 0-2 2v3m18 0V5a2 2 0 0 0-2-2h-3m0 18h3a2 2 0 0 0 2-2v-3M3 16v3a2 2 0 0 0 2 2h3"/></svg>
            </button>
            <button class="btn-icon" onclick="app.reset()" aria-label="Réinitialiser">
                <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8"/><path d="M3 3v5h5"/></svg>
            </button>
        </div>
    </header>

    <div class="stats-row">
        <div class="stat-item">
            <span class="stat-val" id="countLeft" style="color:var(--primary)">37</span>
            <span class="stat-lbl">Restant</span>
        </div>
        <div class="stat-item">
            <span class="stat-val" id="countAlmost" style="color:var(--warning)">0</span>
            <span class="stat-lbl">Presque</span>
        </div>
        <div class="stat-item">
            <span class="stat-val" id="countError" style="color:var(--error)">0</span>
            <span class="stat-lbl">Erreurs</span>
        </div>
    </div>

    <div class="config-area" id="configArea">
        <div class="toggle-row">
            <div class="toggle-group">
                <button class="toggle-btn active" id="btnS2N" onclick="app.setDir('symToName')">Sym → Nom</button>
                <button class="toggle-btn" id="btnN2S" onclick="app.setDir('nameToSym')">Nom → Sym</button>
            </div>
        </div>
        <div class="toggle-row">
            <div class="toggle-group">
                <button class="toggle-btn active" id="btnModeBtn" onclick="app.setMode('buttons')">Boutons</button>
                <button class="toggle-btn" id="btnModeTxt" onclick="app.setMode('text')">Clavier</button>
            </div>
        </div>
        <div class="toggle-row">
            <div class="toggle-group">
                <button class="toggle-btn active" id="btnDeck37" onclick="app.setDeck('base')">Standard (37)</button>
                <button class="toggle-btn" id="btnDeck118" onclick="app.setDeck('mentale')">Mentale (118)</button>
            </div>
        </div>

        <div class="checkbox-row">
            <label class="chk-label" id="lblExpert">
                <input type="checkbox" id="chkExpert" onchange="app.toggleExpert()">
                Expert (Pièges 118)
            </label>
            <label class="chk-label">
                <input type="checkbox" id="chkSpeed" onchange="app.toggleSpeed()">
                Speedrun
            </label>
        </div>
    </div>

    <main class="game-stage">
        <div class="overlay" id="startScreen">
            <h2 style="font-size:2rem; color:var(--primary); margin-bottom:0.5rem">Prêt ?</h2>
            <div class="badges">
                <span class="badge" id="badgeDir">SYM → NOM</span>
                <span class="badge" id="badgeMode">BOUTONS</span>
                <span class="badge expert" id="badgeExp" style="display:none">EXPERT</span>
                <span class="badge mentale" id="badgeMentale" style="display:none">MENTALE (118)</span>
                <span class="badge" id="badgeSpd" style="display:none; background:#fee2e2; color:#b91c1c; border-color:#fca5a5">SPEEDRUN</span>
            </div>
            <p id="startDesc" style="color:var(--text-muted); line-height:1.5; margin-bottom:2rem; max-width:90%">
                Description...
            </p>
            <button class="btn-start" onclick="app.start()">JOUER</button>
        </div>

        <div class="overlay hidden" id="endScreen">
            <h2 id="endTitle" style="font-size:2.2rem; margin-bottom:0.5rem">Terminé !</h2>
            
            <div class="end-grid">
                <div class="stat-item">
                    <span class="stat-val" id="endTime">--:--</span>
                    <span class="stat-lbl">Temps</span>
                </div>
                <div class="stat-item">
                    <span class="stat-val" id="endAlmost" style="color:var(--warning)">0</span>
                    <span class="stat-lbl">Presque</span>
                </div>
                <div class="stat-item">
                    <span class="stat-val" id="endErr" style="color:var(--error)">0</span>
                    <span class="stat-lbl">Erreurs</span>
                </div>
            </div>

            <div style="width:100%; text-align:left; font-size:0.8rem; font-weight:700; color:var(--text-muted); margin-bottom:4px;">Dernières parties :</div>
            <div class="history-box" id="historyList"></div>
            <button class="btn-action" onclick="app.reset()">Retour au menu</button>
        </div>

        <div class="prompt-box font-serif" id="prompt">Ag</div>

        <div class="grid-options font-serif" id="optionsGrid"></div>
        
        <div class="input-container" id="inputArea">
            <input type="text" id="txtInput" placeholder="Votre réponse..." autocomplete="off" spellcheck="false">
            <button class="btn-action" id="btnSubmit" onclick="app.checkText()">Valider</button>
        </div>

        <div class="feedback-area">
            <div class="fb-msg" id="fbMain"></div>
            <div class="fb-sub" id="fbSub"></div>
            <button class="btn-next" id="btnNext" onclick="app.next()">Suivant</button>
        </div>
    </main>
</div>

<script>
const baseElements = [
    {z:1,s:"H",n:"Hydrogène"},{z:2,s:"He",n:"Hélium"},{z:3,s:"Li",n:"Lithium"},{z:4,s:"Be",n:"Béryllium"},
    {z:5,s:"B",n:"Bore"},{z:6,s:"C",n:"Carbone"},{z:7,s:"N",n:"Azote"},{z:8,s:"O",n:"Oxygène"},
    {z:9,s:"F",n:"Fluor"},{z:10,s:"Ne",n:"Néon"},{z:11,s:"Na",n:"Sodium"},{z:12,s:"Mg",n:"Magnésium"},
    {z:13,s:"Al",n:"Aluminium"},{z:14,s:"Si",n:"Silicium"},{z:15,s:"P",n:"Phosphore"},{z:16,s:"S",n:"Soufre"},
    {z:17,s:"Cl",n:"Chlore"},{z:18,s:"Ar",n:"Argon"},{z:19,s:"K",n:"Potassium"},{z:20,s:"Ca",n:"Calcium"},
    {z:21,s:"Sc",n:"Scandium"},{z:22,s:"Ti",n:"Titane"},{z:23,s:"V",n:"Vanadium"},{z:24,s:"Cr",n:"Chrome"},
    {z:25,s:"Mn",n:"Manganèse"},{z:26,s:"Fe",n:"Fer"},{z:27,s:"Co",n:"Cobalt"},{z:28,s:"Ni",n:"Nickel"},
    {z:29,s:"Cu",n:"Cuivre"},{z:30,s:"Zn",n:"Zinc"},{z:31,s:"Ga",n:"Gallium"},{z:32,s:"Ge",n:"Germanium"},
    {z:33,s:"As",n:"Arsenic"},{z:34,s:"Se",n:"Sélénium"},{z:35,s:"Br",n:"Brome"},{z:36,s:"Kr",n:"Krypton"},
    {z:53,s:"I",n:"Iode"}
];

const fullTable = [
    ...baseElements,
    {z:37,s:"Rb",n:"Rubidium"},{z:38,s:"Sr",n:"Strontium"},{z:39,s:"Y",n:"Yttrium"},{z:40,s:"Zr",n:"Zirconium"},
    {z:41,s:"Nb",n:"Niobium"},{z:42,s:"Mo",n:"Molybdène"},{z:43,s:"Tc",n:"Technétium"},{z:44,s:"Ru",n:"Ruthénium"},
    {z:45,s:"Rh",n:"Rhodium"},{z:46,s:"Pd",n:"Palladium"},{z:47,s:"Ag",n:"Argent"},{z:48,s:"Cd",n:"Cadmium"},
    {z:49,s:"In",n:"Indium"},{z:50,s:"Sn",n:"Étain"},{z:51,s:"Sb",n:"Antimoine"},{z:52,s:"Te",n:"Tellure"},
    {z:54,s:"Xe",n:"Xénon"},{z:55,s:"Cs",n:"Césium"},{z:56,s:"Ba",n:"Baryum"},{z:57,s:"La",n:"Lanthane"},
    {z:58,s:"Ce",n:"Cérium"},{z:59,s:"Pr",n:"Praséodyme"},{z:60,s:"Nd",n:"Néodyme"},{z:61,s:"Pm",n:"Prométhium"},
    {z:62,s:"Sm",n:"Samarium"},{z:63,s:"Eu",n:"Europium"},{z:64,s:"Gd",n:"Gadolinium"},{z:65,s:"Tb",n:"Terbium"},
    {z:66,s:"Dy",n:"Dysprosium"},{z:67,s:"Ho",n:"Holmium"},{z:68,s:"Er",n:"Erbium"},{z:69,s:"Tm",n:"Thulium"},
    {z:70,s:"Yb",n:"Ytterbium"},{z:71,s:"Lu",n:"Lutécium"},{z:72,s:"Hf",n:"Hafnium"},{z:73,s:"Ta",n:"Tantale"},
    {z:74,s:"W",n:"Tungstène"},{z:75,s:"Re",n:"Rhénium"},{z:76,s:"Os",n:"Osmium"},{z:77,s:"Ir",n:"Iridium"},
    {z:78,s:"Pt",n:"Platine"},{z:79,s:"Au",n:"Or"},{z:80,s:"Hg",n:"Mercure"},{z:81,s:"Tl",n:"Thallium"},
    {z:82,s:"Pb",n:"Plomb"},{z:83,s:"Bi",n:"Bismuth"},{z:84,s:"Po",n:"Polonium"},{z:85,s:"At",n:"Astate"},
    {z:86,s:"Rn",n:"Radon"},{z:87,s:"Fr",n:"Francium"},{z:88,s:"Ra",n:"Radium"},{z:89,s:"Ac",n:"Actinium"},
    {z:90,s:"Th",n:"Thorium"},{z:91,s:"Pa",n:"Protactinium"},{z:92,s:"U",n:"Uranium"},{z:93,s:"Np",n:"Neptunium"},
    {z:94,s:"Pu",n:"Plutonium"},{z:95,s:"Am",n:"Américium"},{z:96,s:"Cm",n:"Curium"},{z:97,s:"Bk",n:"Berkélium"},
    {z:98,s:"Cf",n:"Californium"},{z:99,s:"Es",n:"Einsteinium"},{z:100,s:"Fm",n:"Fermium"},{z:101,s:"Md",n:"Mendélévium"},
    {z:102,s:"No",n:"Nobélium"},{z:103,s:"Lr",n:"Lawrencium"},{z:104,s:"Rf",n:"Rutherfordium"},{z:105,s:"Db",n:"Dubnium"},
    {z:106,s:"Sg",n:"Seaborgium"},{z:107,s:"Bh",n:"Bohrium"},{z:108,s:"Hs",n:"Hassium"},{z:109,s:"Mt",n:"Meitnerium"},
    {z:110,s:"Ds",n:"Darmstadtium"},{z:111,s:"Rg",n:"Roentgenium"},{z:112,s:"Cn",n:"Copernicium"},{z:113,s:"Nh",n:"Nihonium"},
    {z:114,s:"Fl",n:"Flérovium"},{z:115,s:"Mc",n:"Moscovium"},{z:116,s:"Lv",n:"Livermorium"},{z:117,s:"Ts",n:"Tennesse"},
    {z:118,s:"Og",n:"Oganesson"}
];

const commonMistakes = {
    "nitrogen": "Anglais pour Azote", "nitrogene": "N = Azote",
    "boron": "Anglais pour Bore", "chlorine": "Anglais pour Chlore",
    "souffre": "Soufre ne prend qu'un 'f'", "ph": "Phosphore = P"
};

const app = {
    state: {
        active: false, deck: [], current: null,
        err: 0, almost: 0, answered: false,
        dir: 'symToName', mode: 'buttons',
        mentale: false,
        expert: false, speedrun: false,
        startTS: 0, timerId: null, history: []
    },
    
    ui: {
        screens: { start: document.getElementById('startScreen'), end: document.getElementById('endScreen') },
        stats: { left: document.getElementById('countLeft'), alm: document.getElementById('countAlmost'), err: document.getElementById('countError') },
        game: { prompt: document.getElementById('prompt'), grid: document.getElementById('optionsGrid'), inpBox: document.getElementById('inputArea'), inp: document.getElementById('txtInput'), next: document.getElementById('btnNext'), sub: document.getElementById('btnSubmit') },
        fb: { main: document.getElementById('fbMain'), sub: document.getElementById('fbSub') },
        badges: { dir: document.getElementById('badgeDir'), mode: document.getElementById('badgeMode'), exp: document.getElementById('badgeExp'), spd: document.getElementById('badgeSpd'), men: document.getElementById('badgeMentale') },
        btns: { 
            s2n: document.getElementById('btnS2N'), n2s: document.getElementById('btnN2S'), 
            mBtn: document.getElementById('btnModeBtn'), mTxt: document.getElementById('btnModeTxt'),
            d37: document.getElementById('btnDeck37'), d118: document.getElementById('btnDeck118'),
            fs: document.getElementById('btnFullscreen')
        }
    },

    init() {
        this.updateStartInfo();
        document.addEventListener('keydown', (e) => {
            if(e.key !== 'Enter') return;
            if(!this.ui.screens.start.classList.contains('hidden')) this.start();
            else if(!this.ui.screens.end.classList.contains('hidden')) this.reset();
            else if(this.state.answered) this.next();
            else if(this.state.mode === 'text') this.checkText();
        });
        
        document.addEventListener('fullscreenchange', () => {
            if (document.fullscreenElement) {
                this.ui.btns.fs.classList.add('active');
            } else {
                this.ui.btns.fs.classList.remove('active');
            }
        });
    },

    toggleFullscreen() {
        if (!document.fullscreenElement) {
            document.documentElement.requestFullscreen().catch(err => {
                console.error(`Error attempting to enable full-screen mode: ${err.message}`);
            });
        } else {
            document.exitFullscreen();
        }
    },

    setDir(d) {
        this.state.dir = d;
        this.ui.btns.s2n.className = `toggle-btn ${d === 'symToName' ? 'active' : ''}`;
        this.ui.btns.n2s.className = `toggle-btn ${d === 'nameToSym' ? 'active' : ''}`;
        this.updateStartInfo();
    },
    setMode(m) {
        this.state.mode = m;
        this.ui.btns.mBtn.className = `toggle-btn ${m === 'buttons' ? 'active' : ''}`;
        this.ui.btns.mTxt.className = `toggle-btn ${m === 'text' ? 'active' : ''}`;
        this.updateStartInfo();
    },
    setDeck(type) {
        this.state.mentale = (type === 'mentale');
        this.ui.btns.d37.className = `toggle-btn ${!this.state.mentale ? 'active' : ''}`;
        this.ui.btns.d118.className = `toggle-btn ${this.state.mentale ? 'mentale-active' : ''}`;
        this.updateStartInfo();
    },

    toggleExpert() { this.state.expert = document.getElementById('chkExpert').checked; this.updateStartInfo(); },
    toggleSpeed() { this.state.speedrun = document.getElementById('chkSpeed').checked; this.updateStartInfo(); },

    updateStartInfo() {
        const { dir, mode, expert, speedrun, mentale } = this.state;
        const lblExp = document.getElementById('lblExpert');
        if (mentale) {
            lblExp.classList.add('hidden');
        } else {
            lblExp.classList.remove('hidden');
        }

        this.ui.badges.dir.textContent = dir === 'symToName' ? 'SYMBOLE → NOM' : 'NOM → SYMBOLE';
        this.ui.badges.mode.textContent = mode === 'buttons' ? 'BOUTONS' : 'CLAVIER';
        
        this.ui.badges.men.style.display = mentale ? 'inline-block' : 'none';
        this.ui.badges.exp.style.display = (!mentale && expert) ? 'inline-block' : 'none';
        this.ui.badges.spd.style.display = speedrun ? 'inline-block' : 'none';

        let desc = "";
        if (mentale) {
             desc += `<b>MODE MENTALE</b> : Le tableau complet (118). Courage. <br>`;
        } else {
             desc += `Mode <b>Standard</b> (37 éléments). `;
             if(expert) desc += ` <span style="color:var(--warning)">Expert:</span> Pièges du tableau complet. <br>`;
             else desc += `<br>`;
        }

        if(speedrun) desc += `<span style="color:var(--error)">⚡ Speedrun :</span> Chrono précis, transitions éclairs.`;
        
        document.getElementById('startDesc').innerHTML = desc;
        document.getElementById('configArea').style.display = 'block';
    },

    reset() {
        this.stopTimer();
        this.state.active = false;
        this.ui.screens.end.classList.add('hidden');
        this.ui.screens.start.classList.remove('hidden');
        document.getElementById('timerDisplay').classList.add('hidden');
        document.getElementById('timerDisplay').textContent = "00:00.00";
        this.ui.stats.left.textContent = this.state.mentale ? "118" : "37";
        this.ui.stats.err.textContent = '0';
        this.ui.stats.alm.textContent = '0';
        this.ui.game.prompt.textContent = '';
        this.ui.game.grid.innerHTML = '';
        this.ui.fb.main.textContent = ''; this.ui.fb.sub.textContent = '';
        this.ui.game.next.classList.remove('visible');
        this.updateStartInfo();
    },

    start() {
        document.getElementById('configArea').style.display = 'none';
        this.ui.screens.start.classList.add('hidden');
        this.state.deck = this.state.mentale ? [...fullTable] : [...baseElements];
        this.state.err = 0; this.state.almost = 0;
        this.state.active = true;
        this.ui.stats.err.textContent = '0';
        this.ui.stats.alm.textContent = '0';
        if(this.state.speedrun) document.getElementById('timerDisplay').classList.remove('hidden');
        this.nextQuestion(true);
        this.startTimer();
    },

    next() {
        if(this.state.deck.length === 0) return this.endGame();
        this.nextQuestion();
    },

    nextQuestion(isFirst = false) {
        if(!isFirst) {
            this.state.deck = this.state.deck.filter(e => e.z !== this.state.current.z);
            if(this.state.deck.length === 0) return this.endGame();
        }

        this.state.answered = false;
        this.ui.stats.left.textContent = this.state.deck.length;
        const idx = Math.floor(Math.random() * this.state.deck.length);
        this.state.current = this.state.deck[idx];
        this.ui.game.next.classList.remove('visible');
        this.ui.fb.main.textContent = ''; this.ui.fb.sub.textContent = '';
        this.ui.game.inp.value = ''; this.ui.game.inp.disabled = false; this.ui.game.sub.disabled = false;
        const isS2N = this.state.dir === 'symToName';
        this.ui.game.prompt.textContent = isS2N ? this.state.current.s : this.state.current.n;

        if(this.state.mode === 'text') {
            this.ui.game.grid.style.display = 'none';
            this.ui.game.inpBox.classList.add('active');
            setTimeout(() => this.ui.game.inp.focus(), 50);
        } else {
            this.ui.game.inpBox.classList.remove('active');
            this.ui.game.grid.style.display = 'grid';
            this.renderButtons(isS2N);
        }
    },

    renderButtons(isS2N) {
        this.ui.game.grid.innerHTML = '';
        const pool = (this.state.mentale || this.state.expert) ? fullTable : baseElements;
        const promptStr = this.ui.game.prompt.textContent;
        const promptFirstChar = promptStr.charAt(0).toUpperCase();
        const distractorProp = isS2N ? 'n' : 's'; 

        let distractors = pool.filter(e => 
            e.z !== this.state.current.z && 
            e[distractorProp].charAt(0).toUpperCase() === promptFirstChar
        );

        distractors.sort(() => Math.random() - 0.5);
        let options = [this.state.current, ...distractors.slice(0, 3)];

        while(options.length < 4) {
            let r = pool[Math.floor(Math.random() * pool.length)];
            if(!options.some(o => o.z === r.z)) options.push(r);
        }
        options.sort(() => Math.random() - 0.5);
        options.forEach(opt => {
            const btn = document.createElement('button');
            btn.className = 'btn-opt font-serif';
            btn.textContent = isS2N ? opt.n : opt.s;
            btn.onclick = () => this.checkBtn(opt, btn);
            this.ui.game.grid.appendChild(btn);
        });
    },

    checkBtn(selected, btnEl) {
        if(this.state.answered) return;
        if(selected.z === this.state.current.z) {
            btnEl.classList.add('correct');
            this.success("Correct !");
        } else {
            btnEl.classList.add('wrong');
            const ans = this.state.dir === 'symToName' ? this.state.current.n : this.state.current.s;
            this.fail("Faux !", `Réponse : ${ans}`, true);
        }
    },

    checkText() {
        if(this.state.answered) return;
        let val = this.ui.game.inp.value.trim();
        if(!val) return;

        if(commonMistakes[val.toLowerCase()]) {
            this.ui.game.inp.classList.add('shake');
            setTimeout(()=>this.ui.game.inp.classList.remove('shake'), 400);
            this.ui.fb.main.textContent = "Attention";
            this.ui.fb.main.style.color = "var(--warning)";
            this.ui.fb.sub.textContent = commonMistakes[val.toLowerCase()];
            return;
        }

        const isS2N = this.state.dir === 'symToName';
        const correct = isS2N ? this.state.current.n : this.state.current.s;
        if(val === correct) return this.success("Parfait !");
        const normVal = val.toLowerCase().normalize("NFD").replace(/[\u0300-\u036f]/g, "");
        const normCor = correct.toLowerCase().normalize("NFD").replace(/[\u0300-\u036f]/g, "");

        if(val.toLowerCase() === correct.toLowerCase()) {
            if(!isS2N) {
                 if(this.state.speedrun) {
                     this.ui.game.inp.classList.add('shake');
                     return this.fail("Faux (Casse) !", `Attendu : ${correct}`);
                 }
                 this.state.almost++; this.ui.stats.alm.textContent = this.state.almost;
                 return this.success("Attention Majuscule", `C'est ${correct}`, true);
            }
            return this.success("Correct !");
        }

        if(normVal === normCor) {
            if(this.state.speedrun) {
                this.ui.game.inp.classList.add('shake');
                return this.fail("Faux (Accents) !", `Attendu : ${correct}`);
            }
            this.state.almost++; this.ui.stats.alm.textContent = this.state.almost;
            return this.success("Presque...", `Accents : ${correct}`, true);
        }
        this.ui.game.inp.classList.add('shake');
        this.fail("Incorrect", `Réponse : ${correct}`);
    },

    success(msg, subMsg = "", isWarn = false) {
        this.state.answered = true;
        this.ui.fb.main.textContent = msg;
        this.ui.fb.main.style.color = isWarn ? "var(--warning)" : "var(--success)";
        this.ui.fb.sub.textContent = subMsg;
        this.lockUI();
        if(this.state.speedrun) {
            const delay = this.state.mode === 'text' ? 50 : 150;
            setTimeout(() => this.next(), delay);
        } else {
            this.ui.game.next.classList.add('visible');
            this.ui.game.next.focus();
        }
    },

    fail(msg, subMsg, highlightCorrect = false) {
        this.state.answered = true;
        this.state.err++;
        this.ui.stats.err.textContent = this.state.err;
        this.ui.fb.main.textContent = msg;
        this.ui.fb.main.style.color = "var(--error)";
        this.ui.fb.sub.textContent = subMsg;
        this.lockUI();
        if(highlightCorrect && this.state.mode === 'buttons') {
            const ans = this.state.dir === 'symToName' ? this.state.current.n : this.state.current.s;
            Array.from(this.ui.game.grid.children).forEach(b => {
                if(b.textContent === ans) b.classList.add('correct');
            });
        }
        this.ui.game.next.classList.add('visible');
        if(!this.state.speedrun) this.ui.game.next.focus();
    },

    lockUI() {
        this.ui.game.inp.disabled = true;
        this.ui.game.sub.disabled = true;
        Array.from(this.ui.game.grid.children).forEach(b => b.disabled = true);
    },

    startTimer() {
        this.stopTimer();
        this.state.startTS = Date.now();
        this.state.timerId = setInterval(() => {
            const ms = Date.now() - this.state.startTS;
            document.getElementById('timerDisplay').textContent = this.formatTime(ms);
        }, 34); 
    },
    stopTimer() { clearInterval(this.state.timerId); },
    formatTime(ms) {
        const min = Math.floor(ms/60000);
        const sec = Math.floor((ms%60000)/1000);
        const centi = Math.floor((ms%1000)/10);
        return `${min<10?'0':''}${min}:${sec<10?'0':''}${sec}.${centi<10?'0':''}${centi}`;
    },

    endGame() {
        this.stopTimer();
        this.state.active = false;
        const finalMs = Date.now() - this.state.startTS;
        const timeStr = this.formatTime(finalMs);
        document.getElementById('endTime').textContent = timeStr;
        document.getElementById('endAlmost').textContent = this.state.almost;
        document.getElementById('endErr').textContent = this.state.err;
        const title = document.getElementById('endTitle');
        if(this.state.err === 0 && this.state.almost === 0) {
            title.textContent = "MAGISTRAL !"; title.style.color = "var(--warning)";
            launchFireworks(true);
        } else if(this.state.err === 0) {
            title.textContent = "SANS FAUTE !"; title.style.color = "var(--success)";
            launchFireworks(false);
        } else {
            title.textContent = "TERMINÉ"; title.style.color = "var(--primary)";
        }
        let tag = (this.state.dir === 'symToName' ? 'S→N' : 'N→S');
        tag += (this.state.mode === 'text' ? ' Clavier' : ' QCM');
        if(this.state.mentale) tag += ' Mentale';
        else if(this.state.expert) tag += ' Exp';
        if(this.state.speedrun) tag += ' ⚡';
        this.state.history.unshift({tag, time: timeStr, err: this.state.err, alm: this.state.almost});
        this.state.history = this.state.history.slice(0, 5);
        this.renderHistory();
        this.ui.screens.end.classList.remove('hidden');
    },

    renderHistory() {
        const list = document.getElementById('historyList');
        list.innerHTML = '';
        this.state.history.forEach(h => {
            const div = document.createElement('div');
            div.className = 'hist-row';
            if(h.err === 0 && h.alm === 0) div.classList.add('row-perfect');
            else if(h.err === 0) div.classList.add('row-clean');
            div.innerHTML = `<div><span class="hist-tag">${h.tag}</span> <b>${h.time}</b></div><div style="font-size:0.8rem">Err: <b>${h.err}</b></div>`;
            list.appendChild(div);
        });
    }
};

const cvs = document.getElementById('fireworks');
const ctx = cvs.getContext('2d');
let particles = [];
window.addEventListener('resize', () => { cvs.width = window.innerWidth; cvs.height = window.innerHeight; });
window.dispatchEvent(new Event('resize'));

function launchFireworks(isGold) {
    particles = [];
    let count = 0; const max = isGold ? 15 : 6;
    const interval = setInterval(() => {
        const x = Math.random() * cvs.width;
        const y = cvs.height * 0.3 + (Math.random() * cvs.height * 0.3);
        const color = isGold ? '#fbbf24' : ['#ef4444', '#22c55e', '#3b82f6'][Math.floor(Math.random()*3)];
        for(let i=0; i<40; i++) particles.push(new Particle(x, y, color, isGold));
        if(++count >= max) clearInterval(interval);
    }, 300);
    animate();
}

class Particle {
    constructor(x, y, color, sparkle) {
        this.x = x; this.y = y; this.color = color; this.sparkle = sparkle;
        const angle = Math.random() * Math.PI * 2;
        const v = Math.random() * 4 + 2;
        this.vx = Math.cos(angle) * v; this.vy = Math.sin(angle) * v;
        this.alpha = 1; this.decay = Math.random() * 0.02 + 0.01;
    }
    update() {
        this.vx *= 0.96; this.vy *= 0.96; this.vy += 0.1; 
        this.x += this.vx; this.y += this.vy; this.alpha -= this.decay;
    }
    draw() {
        ctx.save(); ctx.globalAlpha = this.alpha; ctx.fillStyle = this.color;
        ctx.beginPath(); ctx.arc(this.x, this.y, this.sparkle ? Math.random()*3 : 2, 0, Math.PI*2);
        ctx.fill(); ctx.restore();
    }
}

function animate() {
    if(!particles.length) return;
    ctx.clearRect(0,0,cvs.width,cvs.height);
    particles.forEach((p, i) => {
        p.update(); p.draw();
        if(p.alpha <= 0) particles.splice(i, 1);
    });
    requestAnimationFrame(animate);
}

app.init();
</script>
</body>
</html>
