<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Análisis de Circuitos AC - Ruta RPG</title>
    <!-- KaTeX CDN -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.css">
    <script src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/contrib/auto-render.min.js"></script>
    <!-- Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700;900&family=Rajdhani:wght@500;600;700&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --bg-dark: #060911;
            --panel-bg: #0b1220;
            --panel-border: rgba(0, 243, 255, 0.3);
            --cyan: #00f3ff;
            --green: #00ff66;
            --yellow: #ffb700;
            --red: #ff0055;
            --purple: #9d00ff;
            --copper-light: #e5a059;
            --copper-mid: #b87333;
            --copper-dark: #5c3a1e;
            --text-main: #e2e8f0;
            --text-muted: #94a3b8;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }

        body {
            background-color: var(--bg-dark);
            background-image: 
                radial-gradient(circle at 50% 20%, rgba(0, 243, 255, 0.05) 0%, transparent 70%),
                linear-gradient(to right, rgba(255, 255, 255, 0.02) 1px, transparent 1px),
                linear-gradient(to bottom, rgba(255, 255, 255, 0.02) 1px, transparent 1px);
            background-size: 100% 100%, 35px 35px, 35px 35px;
            color: var(--text-main);
            font-family: 'Rajdhani', sans-serif;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            overflow-x: hidden;
        }

        /* --- HEADER RPG HUD --- */
        header {
            background: rgba(6, 9, 17, 0.95);
            border-bottom: 2px solid var(--cyan);
            box-shadow: 0 0 20px rgba(0, 243, 255, 0.2);
            padding: 12px 25px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            position: sticky;
            top: 0;
            z-index: 100;
            backdrop-filter: blur(10px);
        }

        .hud-title {
            font-family: 'Orbitron', sans-serif;
            font-size: 1.2rem;
            font-weight: 800;
            color: #fff;
            text-shadow: 0 0 10px var(--cyan);
        }

        .hud-controls { display: flex; gap: 15px; align-items: center; }

        .btn-codex {
            font-family: 'Orbitron', sans-serif;
            background: rgba(157, 0, 255, 0.2);
            border: 1px solid var(--purple);
            color: #fff;
            padding: 6px 14px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 0.85rem;
            transition: all 0.2s;
            box-shadow: 0 0 10px rgba(157, 0, 255, 0.3);
        }
        .btn-codex:hover { background: var(--purple); box-shadow: 0 0 20px var(--purple); }

        .hud-stats { display: flex; gap: 20px; align-items: center; }
        .stat-box { display: flex; flex-direction: column; align-items: flex-end; }
        .stat-label { font-size: 0.7rem; color: var(--text-muted); text-transform: uppercase; }
        .stat-value { font-family: 'Orbitron', sans-serif; font-size: 1.05rem; color: var(--cyan); font-weight: 700; }
        
        .xp-container {
            width: 120px; height: 8px;
            background: rgba(255,255,255,0.1);
            border: 1px solid var(--cyan);
            border-radius: 4px; overflow: hidden; margin-top: 3px;
        }
        .xp-bar { height: 100%; width: 0%; background: linear-gradient(90deg, var(--cyan), var(--green)); transition: width 0.4s; }

        /* --- MAPA PRINCIPAL Y CABLE DE COBRE --- */
        #map-wrapper {
            position: relative;
            max-width: 900px;
            width: 100%;
            margin: 30px auto;
            padding: 40px 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        #cable-svg {
            position: absolute;
            top: 0; left: 0;
            width: 100%; height: 100%;
            pointer-events: none;
            z-index: 1;
        }

        /* TEXTURA DE CABLE DE COBRE */
        .copper-base {
            fill: none;
            stroke: var(--copper-dark);
            stroke-width: 14;
            stroke-linecap: round;
        }
        .copper-core {
            fill: none;
            stroke: url(#copper-grad);
            stroke-width: 8;
            stroke-linecap: round;
            filter: drop-shadow(0 0 6px rgba(184, 115, 51, 0.6));
        }
        .copper-texture {
            fill: none;
            stroke: var(--copper-light);
            stroke-width: 2;
            stroke-dasharray: 8, 5;
            opacity: 0.7;
        }

        /* SPRITE ELECTRÓN CON SÍMBOLO - */
        .electron-core {
            fill: #00f3ff;
            filter: drop-shadow(0 0 8px #00f3ff) drop-shadow(0 0 16px #00f3ff);
        }
        .electron-text {
            font-family: 'Orbitron', sans-serif;
            font-size: 14px;
            font-weight: 900;
            fill: #060911;
            text-anchor: middle;
            dominant-baseline: central;
        }

        /* SIMBOLOS DE EXTREMO */
        .terminal-node {
            position: relative;
            z-index: 2;
            width: 70px;
            height: 70px;
            border-radius: 50%;
            background: #0b1220;
            border: 2px solid var(--copper-light);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            box-shadow: 0 0 20px rgba(184, 115, 51, 0.4);
            margin: 10px 0;
        }
        .terminal-label {
            font-family: 'Orbitron', sans-serif;
            font-size: 0.65rem;
            color: var(--copper-light);
            margin-top: 4px;
        }

        /* ESTAQUIS Y NODOS DE CAPÍTULOS */
        .chapter-stage {
            position: relative;
            z-index: 2;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 40px 0;
            width: 100%;
        }

        .chapter-circle {
            width: 150px;
            height: 150px;
            border-radius: 50%;
            position: relative;
            overflow: hidden;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            border: 3px solid var(--panel-border);
            box-shadow: 0 0 25px rgba(0,0,0,0.8);
            transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            background: #000;
        }

        .chapter-circle.unlocked { border-color: var(--cyan); box-shadow: 0 0 30px rgba(0, 243, 255, 0.3); }
        .chapter-circle.completed { border-color: var(--green); box-shadow: 0 0 30px rgba(0, 255, 102, 0.3); }
        .chapter-circle.locked { opacity: 0.45; filter: grayscale(1); cursor: not-allowed; border-color: var(--text-muted); }

        .chapter-circle:hover:not(.locked) { transform: scale(1.08); box-shadow: 0 0 40px var(--cyan); }

        .chapter-cover-img {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background-size: cover; background-position: center;
            filter: blur(2.5px) brightness(0.45);
            transition: all 0.3s;
        }
        .chapter-circle:hover .chapter-cover-img { filter: blur(0px) brightness(0.65); transform: scale(1.1); }

        .chapter-overlay-content { position: relative; z-index: 3; text-align: center; padding: 10px; }
        .chapter-num { font-family: 'Orbitron', sans-serif; font-size: 0.75rem; color: var(--yellow); }
        .chapter-name-title { font-family: 'Orbitron', sans-serif; font-size: 0.9rem; font-weight: 800; color: #fff; margin: 3px 0; }
        .chapter-badge { font-family: 'Orbitron', sans-serif; font-size: 0.6rem; padding: 2px 6px; border-radius: 8px; background: rgba(0,0,0,0.8); border: 1px solid var(--cyan); color: var(--cyan); }

        /* DESPLEGABLE SUBTEMAS */
        .subtopics-drawer {
            display: none; width: 100%; max-width: 650px; margin-top: 15px; padding: 15px;
            background: rgba(11, 18, 32, 0.95); border: 1px solid var(--panel-border);
            border-radius: 10px; box-shadow: 0 10px 30px rgba(0,0,0,0.8);
            grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 12px;
        }
        .chapter-stage.open .subtopics-drawer { display: grid; }

        .subtopic-btn {
            background: rgba(255, 255, 255, 0.03); border: 1px solid rgba(255, 255, 255, 0.15);
            border-radius: 8px; padding: 12px; display: flex; justify-content: space-between;
            align-items: center; cursor: pointer; transition: all 0.2s; font-size: 0.9rem;
        }
        .subtopic-btn:hover:not(.locked) { transform: translateY(-2px); border-color: var(--cyan); box-shadow: 0 0 12px rgba(0, 243, 255, 0.2); }
        .subtopic-btn.completed { border-color: var(--green); background: rgba(0, 255, 102, 0.05); }
        .subtopic-btn.locked { opacity: 0.4; cursor: not-allowed; }

        /* --- BARRA DE PROGRESO DE MISIÓN / PASOS --- */
        .subtopic-progress-bar {
            display: flex; gap: 8px; width: 100%; background: rgba(0,0,0,0.4);
            padding: 10px 15px; border-radius: 8px; border: 1px solid rgba(255,255,255,0.08);
            align-items: center; justify-content: space-between;
        }
        .step-pill {
            flex: 1; height: 8px; background: rgba(255,255,255,0.1);
            border-radius: 4px; transition: all 0.3s; position: relative;
        }
        .step-pill.active { background: var(--cyan); box-shadow: 0 0 8px var(--cyan); }
        .step-pill.completed { background: var(--green); }
        .step-pill.boss { border: 1px solid var(--red); }
        .step-pill.boss.active { background: var(--red); box-shadow: 0 0 10px var(--red); }

        /* MODALES */
        .modal-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(4, 6, 12, 0.9); backdrop-filter: blur(8px);
            z-index: 1000; display: flex; justify-content: center; align-items: center;
            opacity: 0; pointer-events: none; transition: opacity 0.25s ease;
        }
        .modal-overlay.active { opacity: 1; pointer-events: all; }

        .modal-card {
            background: var(--panel-bg); border: 2px solid var(--cyan);
            border-radius: 12px; width: 90%; max-width: 900px; max-height: 92vh;
            display: flex; flex-direction: column; overflow: hidden;
            box-shadow: 0 0 35px rgba(0, 243, 255, 0.25);
        }

        .modal-header {
            padding: 15px 25px; border-bottom: 1px solid var(--panel-border);
            display: flex; justify-content: space-between; align-items: center;
            background: rgba(0, 243, 255, 0.05);
        }
        .modal-header h3 { font-family: 'Orbitron', sans-serif; color: var(--cyan); font-size: 1.1rem; }
        .close-btn { background: none; border: none; color: var(--text-muted); font-size: 1.8rem; cursor: pointer; }
        .close-btn:hover { color: var(--red); }

        .modal-body { padding: 25px; overflow-y: auto; display: flex; flex-direction: column; gap: 18px; }

        .narrative-box {
            background: rgba(0, 0, 0, 0.6); border: 1px solid var(--cyan);
            border-radius: 8px; padding: 16px; cursor: pointer; min-height: 80px;
        }
        .narrative-header { font-family: 'Orbitron', sans-serif; font-size: 0.75rem; color: var(--yellow); margin-bottom: 6px; display: flex; justify-content: space-between; }
        .narrative-text { font-size: 1rem; line-height: 1.4; color: #fff; }

        .schema-box {
            background: #03060d; border: 1px solid rgba(255,255,255,0.1);
            border-radius: 8px; padding: 10px; display: flex; flex-direction: column; align-items: center; gap: 6px;
        }
        .schema-title { font-family: 'Orbitron', sans-serif; font-size: 0.8rem; color: var(--cyan); align-self: flex-start; }

        .math-card {
            background: rgba(255, 255, 255, 0.02); border: 1px solid rgba(255, 255, 255, 0.08);
            padding: 10px 15px; border-radius: 6px; display: flex; flex-direction: column; gap: 6px;
        }
        .math-title { font-family: 'Orbitron', sans-serif; font-size: 0.75rem; color: var(--yellow); }
        .math-eq { font-size: 1.05rem; padding: 2px 0; overflow-x: auto; }

        .challenge-box {
            background: rgba(0, 243, 255, 0.05); border: 1px solid var(--cyan);
            border-radius: 8px; padding: 15px; display: flex; flex-direction: column; gap: 8px;
        }
        .challenge-box.boss { background: rgba(255, 0, 85, 0.08); border-color: var(--red); }
        .challenge-title { font-family: 'Orbitron', sans-serif; font-size: 0.9rem; color: var(--yellow); }
        .boss .challenge-title { color: var(--red); }

        .sim-box {
            background: #000; border: 1px solid var(--panel-border);
            border-radius: 8px; padding: 12px; display: flex; flex-direction: column; align-items: center; gap: 10px;
        }

        canvas { background: #050811; border-radius: 4px; border: 1px solid rgba(0, 243, 255, 0.2); max-width: 100%; }

        .legend-box {
            width: 100%; background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.08);
            border-radius: 6px; padding: 8px 12px; font-size: 0.82rem; display: flex; flex-direction: column; gap: 4px;
        }
        .legend-item { display: flex; align-items: center; gap: 8px; }
        .legend-color { width: 10px; height: 10px; border-radius: 2px; }

        .controls-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(170px, 1fr)); gap: 10px; width: 100%; }
        .control-group { display: flex; flex-direction: column; gap: 3px; }
        .control-group label { font-size: 0.78rem; color: var(--text-muted); display: flex; justify-content: space-between; }
        input[type="range"] { accent-color: var(--cyan); cursor: pointer; }

        .nav-btn {
            font-family: 'Orbitron', sans-serif;
            background: linear-gradient(135deg, rgba(0,243,255,0.2), rgba(0,255,102,0.2));
            border: 1px solid var(--cyan); color: #fff;
            padding: 10px 20px; border-radius: 6px; font-size: 0.9rem; font-weight: 700;
            cursor: pointer; transition: all 0.2s; align-self: flex-end;
            display: flex; align-items: center; gap: 8px;
        }
        .nav-btn:hover { background: var(--cyan); color: #000; box-shadow: 0 0 15px var(--cyan); }
        .nav-btn.boss-btn { border-color: var(--red); background: rgba(255, 0, 85, 0.2); }
        .nav-btn.boss-btn:hover { background: var(--red); color: #fff; box-shadow: 0 0 15px var(--red); }

        /* CODEX */
        .codex-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(100px, 1fr)); gap: 12px; padding: 10px 0; }
        .char-card {
            background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.1);
            border-radius: 8px; padding: 12px 8px; display: flex; flex-direction: column; align-items: center;
            cursor: pointer; transition: all 0.2s;
        }
        .char-card.unlocked { border-color: var(--purple); background: rgba(157,0,255,0.08); }
        .char-card.unlocked:hover { transform: scale(1.05); box-shadow: 0 0 15px var(--purple); }
        .char-card.locked { opacity: 0.4; filter: grayscale(1); cursor: not-allowed; }
        .char-symbol { font-family: 'Orbitron', sans-serif; font-size: 1.6rem; font-weight: 900; color: var(--purple); margin-bottom: 4px; }
        .unlocked .char-symbol { color: var(--cyan); }
        .char-name { font-size: 0.7rem; text-align: center; color: var(--text-muted); }

        .char-detail-box {
            background: rgba(157, 0, 255, 0.1); border: 1px solid var(--purple);
            border-radius: 8px; padding: 12px; display: none; flex-direction: column; gap: 6px; margin-top: 10px;
        }
        .char-detail-box.active { display: flex; }
        .char-detail-title { font-family: 'Orbitron', sans-serif; color: var(--yellow); font-size: 0.95rem; }

        .reveal-card {
            background: linear-gradient(135deg, #0d1424, #1a0933);
            border: 2px solid var(--purple); box-shadow: 0 0 50px var(--purple);
            border-radius: 12px; padding: 25px; text-align: center;
            display: flex; flex-direction: column; align-items: center; gap: 12px;
            max-width: 380px; animation: popIn 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        @keyframes popIn { 0% { transform: scale(0.5); opacity: 0; } 100% { transform: scale(1); opacity: 1; } }
        .reveal-symbol { font-family: 'Orbitron', sans-serif; font-size: 3rem; font-weight: 900; color: var(--cyan); text-shadow: 0 0 20px var(--cyan); }
    </style>
</head>
<body>

    <!-- HUD SUPERIOR -->
    <header>
        <div class="hud-title">⚡ CIRCUITY RPG: ANALISIS AC</div>
        <div class="hud-controls">
            <button class="btn-codex" onclick="openCodex()">👾 CÓDEX DE PERSONAJES</button>
            <div class="hud-stats">
                <div class="stat-box">
                    <span class="stat-label">Rango</span>
                    <span class="stat-value" id="player-rank">Novato AC</span>
                </div>
                <div class="stat-box">
                    <span class="stat-label">Progreso</span>
                    <span class="stat-value" id="player-lvl">Nivel 1</span>
                    <div class="xp-container"><div class="xp-bar" id="xp-bar"></div></div>
                </div>
            </div>
        </div>
    </header>

    <!-- MAPA PRINCIPAL CON CABLE DE COBRE -->
    <div id="map-wrapper">
        
        <!-- CANVAS SVG OVERLAY -->
        <svg id="cable-svg">
            <defs>
                <!-- DEGRADADO METÁLICO DE COBRE -->
                <linearGradient id="copper-grad" x1="0%" y1="0%" x2="100%" y2="100%">
                    <stop offset="0%" stop-color="#e5a059" />
                    <stop offset="50%" stop-color="#b87333" />
                    <stop offset="100%" stop-color="#8b5a2b" />
                </linearGradient>
            </defs>

            <!-- CAPAS DEL CABLE DE COBRE -->
            <path id="path-copper-base" class="copper-base" />
            <path id="path-copper-core" class="copper-core" />
            <path id="path-copper-texture" class="copper-texture" />

            <!-- SPRITE ELECTRÓN CON SIGNO - -->
            <g id="electron-group">
                <circle r="12" class="electron-core" />
                <text class="electron-text">-</text>
            </g>
        </svg>

        <!-- NODO TIERRA (ORIGEN) -->
        <div class="terminal-node" id="node-ground">
            <svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="#e5a059" stroke-width="2.5">
                <line x1="12" y1="3" x2="12" y2="13" />
                <line x1="4" y1="13" x2="20" y2="13" />
                <line x1="7" y1="17" x2="17" y2="17" />
                <line x1="10" y1="21" x2="14" y2="21" />
            </svg>
            <span class="terminal-label">TIERRA</span>
        </div>

        <!-- CAPÍTULO 1 -->
        <div class="chapter-stage unlocked" id="chap-1">
            <div class="chapter-circle unlocked" onclick="toggleChapter('chap-1')">
                <div class="chapter-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%23081026\'/><path d=\'M 10 100 Q 50 20 100 100 T 190 100\' stroke=\'%2300f3ff\' stroke-width=\'8\' fill=\'none\'/></svg>');"></div>
                <div class="chapter-overlay-content">
                    <span class="chapter-num">CAPÍTULO 1</span>
                    <h4 class="chapter-name-title">Análisis Senoidal</h4>
                    <span class="chapter-badge" id="tag-chap-1">EN PROGRESO</span>
                </div>
            </div>
            <div class="subtopics-drawer">
                <div class="subtopic-btn unlocked" id="sub-1-1" onclick="openQuest('1-1')">
                    <span>1.1 Onda Senoidal & Fasores</span> <span>➔</span>
                </div>
                <div class="subtopic-btn locked" id="sub-1-2" onclick="openQuest('1-2')">
                    <span>1.2 Impedancia & Resonancia</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-1-3" onclick="openQuest('1-3')">
                    <span>1.3 Leyes de Kirchhoff AC</span> <span>🔒</span>
                </div>
            </div>
        </div>

        <!-- CAPÍTULO 2 -->
        <div class="chapter-stage locked" id="chap-2">
            <div class="chapter-circle locked" onclick="toggleChapter('chap-2')">
                <div class="chapter-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%231a0b2e\'/><polygon points=\'30,170 170,170 170,30\' stroke=\'%23ff0055\' stroke-width=\'6\' fill=\'none\'/></svg>');"></div>
                <div class="chapter-overlay-content">
                    <span class="chapter-num">CAPÍTULO 2</span>
                    <h4 class="chapter-name-title">Potencia AC</h4>
                    <span class="chapter-badge" id="tag-chap-2">BLOQUEADO</span>
                </div>
            </div>
            <div class="subtopics-drawer">
                <div class="subtopic-btn locked" id="sub-2-1" onclick="openQuest('2-1')">
                    <span>2.1 Potencia Instantánea & RMS</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-2-2" onclick="openQuest('2-2')">
                    <span>2.2 Triángulo de Potencia</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-2-3" onclick="openQuest('2-3')">
                    <span>2.3 Corrección del FP</span> <span>🔒</span>
                </div>
            </div>
        </div>

        <!-- CAPÍTULO 3 -->
        <div class="chapter-stage locked" id="chap-3">
            <div class="chapter-circle locked" onclick="toggleChapter('chap-3')">
                <div class="chapter-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%230b1e10\'/><circle cx=\'70\' cy=\'100\' r=\'40\' stroke=\'%23ffb700\' stroke-width=\'6\' fill=\'none\'/><circle cx=\'130\' cy=\'100\' r=\'40\' stroke=\'%2300ff66\' stroke-width=\'6\' fill=\'none\'/></svg>');"></div>
                <div class="chapter-overlay-content">
                    <span class="chapter-num">CAPÍTULO 3</span>
                    <h4 class="chapter-name-title">Acoplo Magnético</h4>
                    <span class="chapter-badge" id="tag-chap-3">BLOQUEADO</span>
                </div>
            </div>
            <div class="subtopics-drawer">
                <div class="subtopic-btn locked" id="sub-3-1" onclick="openQuest('3-1')">
                    <span>3.1 Inductancia Mutua</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-3-2" onclick="openQuest('3-2')">
                    <span>3.2 Regla de Puntos</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-3-3" onclick="openQuest('3-3')">
                    <span>3.3 El Transformador Ideal</span> <span>🔒</span>
                </div>
            </div>
        </div>

        <!-- CAPÍTULO 4 -->
        <div class="chapter-stage locked" id="chap-4">
            <div class="chapter-circle locked" onclick="toggleChapter('chap-4')">
                <div class="chapter-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%23201408\'/><line x1=\'100\' y1=\'100\' x2=\'170\' y2=\'100\' stroke=\'%23ff0055\' stroke-width=\'6\'/><line x1=\'100\' y1=\'100\' x2=\'65\' y2=\'160\' stroke=\'%2300f3ff\' stroke-width=\'6\'/><line x1=\'100\' y1=\'100\' x2=\'65\' y2=\'40\' stroke=\'%2300ff66\' stroke-width=\'6\'/></svg>');"></div>
                <div class="chapter-overlay-content">
                    <span class="chapter-num">CAPÍTULO 4</span>
                    <h4 class="chapter-name-title">Sistemas Polifásicos</h4>
                    <span class="chapter-badge" id="tag-chap-4">BLOQUEADO</span>
                </div>
            </div>
            <div class="subtopics-drawer">
                <div class="subtopic-btn locked" id="sub-4-1" onclick="openQuest('4-1')">
                    <span>4.1 Generador Trifásico</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-4-2" onclick="openQuest('4-2')">
                    <span>4.2 Conexión Y / Δ</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-4-3" onclick="openQuest('4-3')">
                    <span>4.3 Cargas Desequilibradas</span> <span>🔒</span>
                </div>
            </div>
        </div>

        <!-- NODO FUENTE DE VOLTAJE (FINAL) -->
        <div class="terminal-node" id="node-source">
            <svg width="34" height="34" viewBox="0 0 24 24" fill="none" stroke="#e5a059" stroke-width="2">
                <circle cx="12" cy="12" r="9" />
                <path d="M 7 12 Q 9.5 7 12 12 T 17 12" />
            </svg>
            <span class="terminal-label">FUENTE AC</span>
        </div>

    </div>

    <!-- MODAL DE MISIÓN COMPLETO -->
    <div class="modal-overlay" id="quest-modal">
        <div class="modal-card">
            <div class="modal-header">
                <h3 id="modal-title">Título de Misión</h3>
                <button class="close-btn" onclick="closeQuest()">&times;</button>
            </div>
            <div class="modal-body">
                
                <!-- BARRA DE PROGRESO DEL SUBTEMA -->
                <div class="subtopic-progress-bar">
                    <span style="font-family:'Orbitron'; font-size:0.75rem; color:var(--text-muted);" id="progress-step-text">Paso 1 de 6</span>
                    <div style="display:flex; gap:6px; flex:1; margin:0 15px;">
                        <div class="step-pill" id="pill-0"></div>
                        <div class="step-pill" id="pill-1"></div>
                        <div class="step-pill" id="pill-2"></div>
                        <div class="step-pill" id="pill-3"></div>
                        <div class="step-pill" id="pill-4"></div>
                        <div class="step-pill boss" id="pill-5"></div>
                    </div>
                </div>

                <!-- PASO 0: TEORÍA -->
                <div class="step-view" id="step-0">
                    <div class="narrative-box" onclick="skipTypewriter()">
                        <div class="narrative-header">
                            <span>TRANSMISIÓN DE DATOS ACADÉMICOS</span>
                            <span style="font-size:0.65rem; color:var(--text-muted);">(Haz clic para mostrar todo)</span>
                        </div>
                        <div class="narrative-text typing-cursor" id="narrative-text-el"></div>
                    </div>

                    <div class="schema-box">
                        <div class="schema-title">🔍 ESQUEMA ORIGEN DE LAS VARIABLES</div>
                        <canvas id="schema-canvas" width="650" height="130"></canvas>
                    </div>
                    
                    <div class="math-card">
                        <div class="math-title">Ecuación Fundamental</div>
                        <div class="math-eq" id="eq-1"></div>
                    </div>

                    <div class="math-card">
                        <div class="math-title">Formulación Final de Análisis</div>
                        <div class="math-eq" id="eq-2"></div>
                    </div>

                    <button class="nav-btn" onclick="nextSubstep()">
                        Continuar al Ejemplo ➔
                    </button>
                </div>

                <!-- PASOS 1 A 5: SIMULADOR (DEMO, EJERCICIOS Y BOSS) -->
                <div class="step-view" id="step-sim-view">
                    
                    <div class="challenge-box" id="challenge-banner">
                        <div class="challenge-title" id="challenge-title">Modo Ejemplo Guiado</div>
                        <div id="challenge-desc" style="font-size:0.9rem; color:#fff;">Observa cómo responde el simulador automáticamente al variarse sus parámetros.</div>
                    </div>

                    <div class="sim-box">
                        <canvas id="sim-canvas" width="650" height="210"></canvas>
                        <div class="legend-box" id="sim-legend"></div>
                        <div class="controls-grid" id="sim-controls"></div>
                    </div>

                    <div style="display: flex; justify-content: space-between; width: 100%; align-items: center;">
                        <button class="nav-btn" style="background: rgba(255,255,255,0.1); border-color: var(--text-muted);" onclick="prevSubstep()">
                            ⬅ Anterior
                        </button>
                        <button class="nav-btn" id="btn-next-step" onclick="nextSubstep()">
                            Continuar ➔
                        </button>
                    </div>
                </div>

            </div>
        </div>
    </div>

    <!-- MODAL CÓDEX DE PERSONAJES -->
    <div class="modal-overlay" id="codex-modal">
        <div class="modal-card" style="max-width: 650px;">
            <div class="modal-header">
                <h3>👾 CÓDEX DE PERSONAJES (VARIABLES AC)</h3>
                <button class="close-btn" onclick="closeCodex()">&times;</button>
            </div>
            <div class="modal-body">
                <p style="font-size: 0.85rem; color: var(--text-muted);">
                    Haz clic en un personaje descubierto para examinar sus datos:
                </p>
                <div class="codex-grid" id="codex-grid"></div>
                <div class="char-detail-box" id="char-detail-box">
                    <div class="char-detail-title" id="char-detail-name">Selecciona un personaje</div>
                    <div id="char-detail-desc" style="font-size: 0.9rem;"></div>
                </div>
            </div>
        </div>
    </div>

    <!-- MODAL REVELACIÓN DE PERSONAJE -->
    <div class="modal-overlay" id="reveal-modal">
        <div class="reveal-card">
            <div style="font-family: 'Orbitron'; font-size: 0.75rem; color: var(--yellow);">¡NUEVO PERSONAJE DESBLOQUEADO!</div>
            <div class="reveal-symbol" id="reveal-symbol">V</div>
            <h3 id="reveal-title" style="color: #fff; font-family: 'Orbitron';">Nombre del Personaje</h3>
            <p id="reveal-desc" style="font-size: 0.85rem; color: var(--text-muted);"></p>
            <button class="nav-btn" onclick="closeReveal()" style="align-self: center;">¡Entendido!</button>
        </div>
    </div>

    <script>
        /* --- BASE DE DATOS DE PERSONAJES Y SUBTEMAS CON EJERCICIOS --- */
        const charactersDB = {
            V: { symbol: 'V', name: 'Voltaje (Tensión)', desc: '<b>Clase:</b> Impulsor.<br><b>Función:</b> Fuerza electromotriz senoidal que impulsa los electrones.', introKey: '1-1' },
            I: { symbol: 'I', name: 'Corriente (Intensidad)', desc: '<b>Clase:</b> Torrente.<br><b>Función:</b> Flujo eléctrico que oscila en el tiempo con desfase.', introKey: '1-1' },
            w: { symbol: 'ω', name: 'Frecuencia Angular', desc: '<b>Clase:</b> Marcapasos.<br><b>Función:</b> Determina la velocidad angular de rotación de los fasores (ω = 2πf).', introKey: '1-1' },
            Z: { symbol: 'Z', name: 'Impedancia Compleja', desc: '<b>Clase:</b> Guardián.<br><b>Función:</b> Oposición total en AC (Resistencia + Reactancia).', introKey: '1-2' },
            K: { symbol: 'K', name: 'Leyes de Kirchhoff', desc: '<b>Clase:</b> Red de Mallas.<br><b>Función:</b> Mantiene la conservación de energía mediante matrices fasoriales.', introKey: '1-3' },
            P: { symbol: 'P', name: 'Potencia Activa', desc: '<b>Clase:</b> Convertidor Real.<br><b>Función:</b> Energía promedio transformada en trabajo útil (Watts).', introKey: '2-1' },
            S: { symbol: 'S', name: 'Potencia Aparente', desc: '<b>Clase:</b> Envolvente Total.<br><b>Función:</b> Capacidad total suministrada por la red eléctrica (VA).', introKey: '2-2' },
            FP: { symbol: 'FP', name: 'Factor de Potencia', desc: '<b>Clase:</b> Centinela de Eficiencia.<br><b>Función:</b> Razón cos(θ) entre la energía útil y la energía entregada.', introKey: '2-3' },
            M: { symbol: 'M', name: 'Inductancia Mutua', desc: '<b>Clase:</b> Enlace Magnético.<br><b>Función:</b> Induce tensión entre bobinas separadas mediante flujo magnético.', introKey: '3-1' },
            VL: { symbol: 'VL', name: 'Tensión Trifásica', desc: '<b>Clase:</b> Trinidad Industrial.<br><b>Función:</b> Voltaje de línea entre fases de un sistema polifásico (√3 · VP).', introKey: '4-1' }
        };

        const questsDB = {
            '1-1': {
                title: 'Misión 1.1: Onda Senoidal y Fasores', chap: 'chap-1', next: '1-2', introChar: 'V',
                narrative: '¡Saludos, Ingeniero! Cuando una espira gira dentro de un campo magnético, genera una tensión senoidal en el tiempo. Transformamos esta onda variante en un <b>Fasor</b> (vector complejo) para analizar amplitud y fase fácilmente.',
                eq1: 'v(t) = V_m \\cdot \\cos(\\omega t + \\phi)', eq2: '\\mathbf{V} = V_{rms} \\angle \\phi = \\frac{V_m}{\\sqrt{2}} e^{j\\phi}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Fasor V:</b> Magnitud $V_m$ y ángulo $\\phi$.</div><div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Onda Senoidal:</b> Evolución temporal $v(t)$.</div>',
                controls: `<div class="control-group"><label>Amplitud Vm: <span id="val-vm">100</span> V</label><input type="range" id="input-vm" min="30" max="150" value="100" oninput="drawSimSenoidal()"></div><div class="control-group"><label>Desfase ϕ: <span id="val-phi">45</span>°</label><input type="range" id="input-phi" min="-180" max="180" value="45" oninput="drawSimSenoidal()"></div>`,
                drawSchema: (ctx) => drawSchemaSenoidal(ctx), drawSim: () => drawSimSenoidal(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "El simulador varía Vm y el desfase ϕ en tiempo real." },
                    { title: "Ejercicio 1 de 3", desc: "Ajusta la Amplitud Vm exactamente a 120 V.", check: () => parseFloat(document.getElementById('input-vm').value) === 120 },
                    { title: "Ejercicio 2 de 3", desc: "Ajusta el Desfase ϕ en cuadratura positiva a +90°.", check: () => parseFloat(document.getElementById('input-phi').value) === 90 },
                    { title: "Ejercicio 3 de 3", desc: "Coloca la señal en oposición de fase completa (-180°).", check: () => parseFloat(document.getElementById('input-phi').value) === -180 },
                    { title: "👾 DESAFÍO BOSS: Generar Onda Vrms = 100V", desc: "Para obtener Vrms = 100V, ajusta la amplitud de pico Vm a 141 V y desfase a +45°.", check: () => parseFloat(document.getElementById('input-vm').value) === 141 && parseFloat(document.getElementById('input-phi').value) === 45 }
                ]
            },
            '1-2': {
                title: 'Misión 1.2: Impedancia y Resonancia RLC', chap: 'chap-1', next: '1-3', introChar: 'Z',
                narrative: 'La <b>Impedancia ($Z$)</b> combina Resistencia y Reactancia. En la <b>Resonancia ($f_0$)</b>, las reactancias inductiva y capacitiva se anulan exactamente ($X_L = X_C$), reduciendo la oposición a su valor resistivo puro.',
                eq1: '\\mathbf{Z} = R + j\\left(\\omega L - \\frac{1}{\\omega C}\\right)', eq2: 'f_0 = \\frac{1}{2\\pi \\sqrt{LC}}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Curva |Z|:</b> Mínimo de impedancia en resonancia.</div>',
                controls: `<div class="control-group"><label>Inductancia L: <span id="val-l">10</span> mH</label><input type="range" id="input-l" min="1" max="50" value="10" oninput="drawSimRLC()"></div><div class="control-group"><label>Capacitancia C: <span id="val-c">10</span> µF</label><input type="range" id="input-c" min="1" max="50" value="10" oninput="drawSimRLC()"></div>`,
                drawSchema: (ctx) => drawSchemaRLC(ctx), drawSim: () => drawSimRLC(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Observa cómo cambia la frecuencia de resonancia f0." },
                    { title: "Ejercicio 1 de 3", desc: "Fija L = 25 mH.", check: () => parseFloat(document.getElementById('input-l').value) === 25 },
                    { title: "Ejercicio 2 de 3", desc: "Ajusta C = 40 µF.", check: () => parseFloat(document.getElementById('input-c').value) === 40 },
                    { title: "Ejercicio 3 de 3", desc: "Iguala ambos parámetros a 20 (L = 20 mH, C = 20 µF).", check: () => parseFloat(document.getElementById('input-l').value) === 20 && parseFloat(document.getElementById('input-c').value) === 20 },
                    { title: "👾 DESAFÍO BOSS: Sintonía F0 ≈ 1000 Hz", desc: "Busca la combinación L = 5 mH y C = 5 µF.", check: () => parseFloat(document.getElementById('input-l').value) === 5 && parseFloat(document.getElementById('input-c').value) === 5 }
                ]
            },
            '1-3': {
                title: 'Misión 1.3: Leyes de Kirchhoff en AC', chap: 'chap-1', next: '2-1', introChar: 'K',
                narrative: 'Las leyes de nodos y mallas aplican en AC operando con fasores complejos. Resolvemos sistemas de mallas agrupa matricialmente las impedancias del circuito.',
                eq1: '\\sum \\mathbf{V}_k = 0', eq2: '[\\mathbf{Y}] \\cdot [\\mathbf{V}] = [\\mathbf{I}]',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Fasores de Corriente:</b> Matrices de admitancia complejas.</div>',
                controls: `<div class="control-group"><label>Impedancia Z2: <span id="val-z2">20</span> Ω</label><input type="range" id="input-z2" min="5" max="50" value="20" oninput="drawSimKCL()"></div>`,
                drawSchema: (ctx) => drawSchemaKCL(ctx), drawSim: () => drawSimKCL(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Cambio continuo en la matriz de admitancia." },
                    { title: "Ejercicio 1 de 3", desc: "Ajusta la impedancia Z2 a 15 Ω.", check: () => parseFloat(document.getElementById('input-z2').value) === 15 },
                    { title: "Ejercicio 2 de 3", desc: "Aumenta Z2 a 35 Ω.", check: () => parseFloat(document.getElementById('input-z2').value) === 35 },
                    { title: "Ejercicio 3 de 3", desc: "Coloca Z2 en su valor máximo de 50 Ω.", check: () => parseFloat(document.getElementById('input-z2').value) === 50 },
                    { title: "👾 DESAFÍO BOSS: Equilibrar Nodal V1 = 180V", desc: "Ajusta Z2 a 10 Ω exactamente.", check: () => parseFloat(document.getElementById('input-z2').value) === 10 }
                ]
            },
            '2-1': {
                title: 'Misión 2.1: Potencia Instantánea y RMS', chap: 'chap-2', next: '2-2', introChar: 'P',
                narrative: 'La potencia p(t) oscila al doble de la frecuencia. El valor **RMS** mide el efecto térmico real en trabajo útil.',
                eq1: 'p(t) = v(t) \\cdot i(t)', eq2: 'P_{prom} = V_{rms} \\cdot I_{rms} \\cos\\theta',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>P Promedio:</b> Nivel continuo útil.</div>',
                controls: `<div class="control-group"><label>Voltaje Vrms: <span id="val-vrms">120</span> V</label><input type="range" id="input-vrms" min="60" max="240" value="120" oninput="drawSimPinst()"></div>`,
                drawSchema: (ctx) => drawSchemaPinst(ctx), drawSim: () => drawSimPinst(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Variación de la potencia promedio." },
                    { title: "Ejercicio 1 de 3", desc: "Ajusta el voltaje residencial estándar Vrms = 110 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 110 },
                    { title: "Ejercicio 2 de 3", desc: "Suba a voltaje industrial ligero Vrms = 220 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 220 },
                    { title: "Ejercicio 3 de 3", desc: "Disminuya a Vrms = 80 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 80 },
                    { title: "👾 DESAFÍO BOSS: Potencia P_prom = 360W", desc: "Para alcanzar P_prom = 360W, ajusta Vrms a 240 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 240 }
                ]
            },
            '2-2': {
                title: 'Misión 2.2: Triángulo de Potencia Compleja', chap: 'chap-2', next: '2-3', introChar: 'S',
                narrative: 'La potencia vectorial $\\mathbf{S}$ une la Potencia Real $P$ (Watts) y la Reactiva $Q$ (VAR) en un triángulo rectángulo.',
                eq1: '\\mathbf{S} = P + jQ', eq2: '|\\mathbf{S}| = \\sqrt{P^2 + Q^2}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Vector S:</b> Potencia aparente total.</div>',
                controls: `<div class="control-group"><label>Reactiva QL: <span id="val-ql">250</span> VAR</label><input type="range" id="input-ql" min="50" max="450" value="250" oninput="drawSimTriang()"></div>`,
                drawSchema: (ctx) => drawSchemaTriang(ctx), drawSim: () => drawSimTriang(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Crecimiento de la hipotenusa de potencia S." },
                    { title: "Ejercicio 1 de 3", desc: "Ajusta QL = 100 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 100 },
                    { title: "Ejercicio 2 de 3", desc: "Aumenta la carga inductiva QL = 300 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 300 },
                    { title: "Ejercicio 3 de 3", desc: "Coloca QL = 400 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 400 },
                    { title: "👾 DESAFÍO BOSS: Triángulo Isósceles P = Q", desc: "Pista: Sabiendo que P = 300W, ajusta QL exactamente a 300 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 300 }
                ]
            },
            '2-3': {
                title: 'Misión 2.3: Corrección del Factor de Potencia', chap: 'chap-2', next: '3-1', introChar: 'FP',
                narrative: 'Añadimos un banco de condensadores en paralelo para inyectar potencia capacitiva ($Q_C$) y reducir la reactiva total.',
                eq1: 'FP = \\cos(\\theta) = \\frac{P}{S}', eq2: 'Q_C = P(\\tan\\theta_1 - \\tan\\theta_2)',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Triángulo Reducido:</b> Menor consumo reactivo.</div>',
                controls: `<div class="control-group"><label>Inyección QC: <span id="val-qc">150</span> VAR</label><input type="range" id="input-qc" min="0" max="350" value="150" oninput="drawSimFP()"></div>`,
                drawSchema: (ctx) => drawSchemaFP(ctx), drawSim: () => drawSimFP(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Efecto de la inyección de capacitores en el vector S." },
                    { title: "Ejercicio 1 de 3", desc: "Inyecta una compensación inicial QC = 100 VAR.", check: () => parseFloat(document.getElementById('input-qc').value) === 100 },
                    { title: "Ejercicio 2 de 3", desc: "Sube la inyección a QC = 200 VAR.", check: () => parseFloat(document.getElementById('input-qc').value) === 200 },
                    { title: "Ejercicio 3 de 3", desc: "Compensa totalmente la reactiva (QC = 300 VAR).", check: () => parseFloat(document.getElementById('input-qc').value) === 300 },
                    { title: "👾 DESAFÍO BOSS: Lograr FP Óptimo ≥ 0.95", desc: "Inyecta exactamente QC = 210 VAR para alcanzar FP = 0.96.", check: () => parseFloat(document.getElementById('input-qc').value) === 210 }
                ]
            },
            '3-1': {
                title: 'Misión 3.1: Inductancia Mutua y Acoplo', chap: 'chap-3', next: '3-2', introChar: 'M',
                narrative: 'El flujo magnético alterno de una bobina induce tensión en otra por inductancia mutua $M = k\sqrt{L_1 L_2}$.',
                eq1: 'v_2(t) = M \\frac{di_1}{dt}', eq2: 'M = k \\sqrt{L_1 L_2}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Flujo Φ12:</b> Campo compartido.</div>',
                controls: `<div class="control-group"><label>Coeficiente k: <span id="val-k">0.7</span></label><input type="range" id="input-k" min="0.1" max="1" step="0.05" value="0.7" oninput="drawSimMutua()"></div>`,
                drawSchema: (ctx) => drawSchemaMutua(ctx), drawSim: () => drawSimMutua(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Variación del acoplamiento magnético k." },
                    { title: "Ejercicio 1 de 3", desc: "Establece un acoplamiento débil k = 0.3.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 0.3) < 0.01 },
                    { title: "Ejercicio 2 de 3", desc: "Aumente a un acoplamiento medio k = 0.6.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 0.6) < 0.01 },
                    { title: "Ejercicio 3 de 3", desc: "Ajuste acoplamiento fuerte k = 0.9.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 0.9) < 0.01 },
                    { title: "👾 DESAFÍO BOSS: Acoplamiento Perfecto Ideal", desc: "Lleve el coeficiente k a su límite perfecto k = 1.0.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 1.0) < 0.01 }
                ]
            },
            '3-2': {
                title: 'Misión 3.2: Regla de Puntos en Bobinas', chap: 'chap-3', next: '3-3', introChar: 'M',
                narrative: 'Los puntos polares indican si la tensión inducida mutuamente se suma (+M) o se resta (-M).',
                eq1: '\\mathbf{V}_1 = j\\omega L_1 \\mathbf{I}_1 \\pm j\\omega M \\mathbf{I}_2', eq2: '\\mathbf{V}_2 = j\\omega L_2 \\mathbf{I}_2 \\pm j\\omega M \\mathbf{I}_1',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Puntos Polares:</b> Referencia de concordancia.</div>',
                controls: `<div class="control-group"><label>Dirección I2: <span id="val-dir">Mismo Punto</span></label><input type="range" id="input-dir" min="0" max="1" step="1" value="1" oninput="drawSimDots()"></div>`,
                drawSchema: (ctx) => drawSchemaDots(ctx), drawSim: () => drawSimDots(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Inversión de polaridad en el punto secundario." },
                    { title: "Ejercicio 1 de 3", desc: "Coloca la corriente en Punto Opuesto (0).", check: () => parseInt(document.getElementById('input-dir').value) === 0 },
                    { title: "Ejercicio 2 de 3", desc: "Cambia la corriente a Mismo Punto (1).", check: () => parseInt(document.getElementById('input-dir').value) === 1 },
                    { title: "Ejercicio 3 de 3", desc: "Retorna a Punto Opuesto (0).", check: () => parseInt(document.getElementById('input-dir').value) === 0 },
                    { title: "👾 DESAFÍO BOSS: Tensión Sumativa (+M)", desc: "Asegura la configuración de adición de flujo (1).", check: () => parseInt(document.getElementById('input-dir').value) === 1 }
                ]
            },
            '3-3': {
                title: 'Misión 3.3: El Transformador Ideal', chap: 'chap-3', next: '4-1', introChar: 'M',
                narrative: 'Escala voltajes según la razón de vueltas $a = N_1 / N_2$ aislando etapas de potencia.',
                eq1: 'a = \\frac{N_1}{N_2} = \\frac{\\mathbf{V}_1}{\\mathbf{V}_2}', eq2: '\\mathbf{Z}_{in} = a^2 \\mathbf{Z}_L',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>V2 Salida:</b> Transformada por el factor $a$.</div>',
                controls: `<div class="control-group"><label>Razón a: <span id="val-a">2.0</span></label><input type="range" id="input-a" min="0.5" max="4" step="0.5" value="2" oninput="drawSimTransfo()"></div>`,
                drawSchema: (ctx) => drawSchemaTransfo(ctx), drawSim: () => drawSimTransfo(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Efecto elevador y reductor de voltaje." },
                    { title: "Ejercicio 1 de 3", desc: "Ajuste un transformador reductor a la mitad (a = 2.0).", check: () => parseFloat(document.getElementById('input-a').value) === 2.0 },
                    { title: "Ejercicio 2 de 3", desc: "Configure un transformador elevador al doble (a = 0.5).", check: () => parseFloat(document.getElementById('input-a').value) === 0.5 },
                    { title: "Ejercicio 3 de 3", desc: "Ajuste un transformador de aislamiento 1:1 (a = 1.0).", check: () => parseFloat(document.getElementById('input-a').value) === 1.0 },
                    { title: "👾 DESAFÍO BOSS: Salida V2 = 30 V (Entrada 120V)", desc: "Ajuste la razón a = 4.0 para reducir 120V a 30V.", check: () => parseFloat(document.getElementById('input-a').value) === 4.0 }
                ]
            },
            '4-1': {
                title: 'Misión 4.1: Generador Trifásico y Secuencias', chap: 'chap-4', next: '4-2', introChar: 'VL',
                narrative: 'Tres devanados desfasados $120^\circ$ generan mayor densidad de potencia en distribución industrial.',
                eq1: '\\mathbf{V}_{aN} = V_p \\angle 0^\\circ', eq2: 'V_{linea} = \\sqrt{3} \\cdot V_p',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Fases A-B-C:</b> Desfasadas 120°.</div>',
                controls: `<div class="control-group"><label>Voltaje Vp: <span id="val-vp">120</span> V</label><input type="range" id="input-vp" min="60" max="240" value="120" oninput="drawSimTriGen()"></div>`,
                drawSchema: (ctx) => drawSchemaTriGen(ctx), drawSim: () => drawSimTriGen(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Rotación simétrica del sistema de 3 fases." },
                    { title: "Ejercicio 1 de 3", desc: "Ajusta la tensión de fase Vp = 100 V.", check: () => parseFloat(document.getElementById('input-vp').value) === 100 },
                    { title: "Ejercicio 2 de 3", desc: "Aumenta la tensión a Vp = 200 V.", check: () => parseFloat(document.getElementById('input-vp').value) === 200 },
                    { title: "Ejercicio 3 de 3", desc: "Disminuye la tensión a Vp = 150 V.", check: () => parseFloat(document.getElementById('input-vp').value) === 150 },
                    { title: "👾 DESAFÍO BOSS: Tensión entre Líneas VL ≈ 208 V", desc: "Ajusta la tensión de fase a Vp = 120 V (VL = 120·√3 ≈ 208V).", check: () => parseFloat(document.getElementById('input-vp').value) === 120 }
                ]
            },
            '4-2': {
                title: 'Misión 4.2: Conexión Estrella Y / Delta Δ', chap: 'chap-4', next: '4-3', introChar: 'VL',
                narrative: 'En Estrella $V_L = \sqrt{3}V_P$. En Delta $I_L = \sqrt{3}I_P$.',
                eq1: 'Y: V_L = \\sqrt{3}V_p', eq2: '\\Delta: I_L = \\sqrt{3}I_p',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Línea vs Fase:</b> Factor √3.</div>',
                controls: `<div class="control-group"><label>Corriente Ip: <span id="val-ip">10</span> A</label><input type="range" id="input-ip" min="2" max="25" value="10" oninput="drawSimYDelta()"></div>`,
                drawSchema: (ctx) => drawSchemaYDelta(ctx), drawSim: () => drawSimYDelta(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Escalado vectorial √3 en corrientes de línea." },
                    { title: "Ejercicio 1 de 3", desc: "Ajusta corriente Ip = 5 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 5 },
                    { title: "Ejercicio 2 de 3", desc: "Sube Ip = 15 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 15 },
                    { title: "Ejercicio 3 de 3", desc: "Aumente Ip = 20 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 20 },
                    { title: "👾 DESAFÍO BOSS: Corriente de Línea IL ≈ 17.3 A", desc: "Ajusta Ip = 10 A para obtener IL = 10·√3 = 17.32 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 10 }
                ]
            },
            '4-3': {
                title: 'Misión 4.3: Cargas Trifásicas Desequilibradas', chap: 'chap-4', next: null, introChar: 'VL',
                narrative: 'Si las cargas no son iguales, la corriente de retorno por el neutro es distinta de cero ($I_N \neq 0$).',
                eq1: '\\mathbf{I}_N = \\mathbf{I}_A + \\mathbf{I}_B + \\mathbf{I}_C', eq2: 'P_{total} = P_A + P_B + P_C',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Vector IN:</b> Retorno por neutro.</div>',
                controls: `<div class="control-group"><label>Desequilibrio A: <span id="val-des">60</span>%</label><input type="range" id="input-des" min="0" max="100" value="60" oninput="drawSimDeseb()"></div>`,
                drawSchema: (ctx) => drawSchemaDeseb(ctx), drawSim: () => drawSimDeseb(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Variación de la corriente de desequilibrio en neutro." },
                    { title: "Ejercicio 1 de 3", desc: "Ajuste un desequilibrio de la Fase A al 20%.", check: () => parseFloat(document.getElementById('input-des').value) === 20 },
                    { title: "Ejercicio 2 de 3", desc: "Sube el desequilibrio al 50%.", check: () => parseFloat(document.getElementById('input-des').value) === 50 },
                    { title: "Ejercicio 3 de 3", desc: "Coloca la línea equilibrada al 0%.", check: () => parseFloat(document.getElementById('input-des').value) === 0 },
                    { title: "👾 DESAFÍO BOSS: Máxima Sobrecarga en Neutro", desc: "Sature el desequilibrio al 100% en la Fase A.", check: () => parseFloat(document.getElementById('input-des').value) === 100 }
                ]
            }
        };

        /* --- ESTADO DEL JUGADOR Y CONTROL DE PASOS --- */
        const gameState = {
            xp: 0, level: 1,
            unlockedChaps: ['chap-1'],
            completedQuests: [],
            unlockedQuests: ['1-1'],
            unlockedChars: ['V', 'I', 'w']
        };

        let activeQuest = null;
        let activeSubstep = 0; // 0: Teoría, 1: Demo, 2: Ej1, 3: Ej2, 4: Ej3, 5: Boss
        let typewriterTimeout = null;
        let currentFullText = "";
        let electronAnimProgress = 0;
        let demoInterval = null;

        window.onload = () => {
            updateUI();
            updateCablesAndElectron();
            window.addEventListener('resize', updateCablesAndElectron);
            requestAnimationFrame(animateElectron);
        };

        function updateUI() {
            document.getElementById('player-lvl').innerText = `Nivel ${gameState.level}`;
            const ranks = ["Novato AC", "Analista de Fasores", "Maestro de Impedancias", "Soberano Trifásico"];
            document.getElementById('player-rank').innerText = ranks[Math.min(gameState.level - 1, ranks.length - 1)];
            document.getElementById('xp-bar').style.width = `${Math.min((gameState.xp / (gameState.level * 200)) * 100, 100)}%`;

            ['chap-1', 'chap-2', 'chap-3', 'chap-4'].forEach((cId) => {
                const el = document.getElementById(cId);
                const tag = document.getElementById(`tag-${cId}`);
                const circle = el.querySelector('.chapter-circle');
                const isUnlocked = gameState.unlockedChaps.includes(cId);
                
                if (isUnlocked) {
                    el.classList.remove('locked'); el.classList.add('unlocked');
                    circle.className = "chapter-circle unlocked"; tag.innerText = "DESBLOQUEADO";
                } else {
                    el.classList.add('locked');
                    circle.className = "chapter-circle locked"; tag.innerText = "BLOQUEADO 🔒";
                }
            });

            Object.keys(questsDB).forEach(qKey => {
                const btn = document.getElementById(`sub-${qKey}`);
                if (!btn) return;

                if (gameState.completedQuests.includes(qKey)) {
                    btn.className = "subtopic-btn completed";
                    btn.querySelector('span:last-child').innerText = "✔";
                } else if (gameState.unlockedQuests.includes(qKey)) {
                    btn.className = "subtopic-btn unlocked";
                    btn.querySelector('span:last-child').innerText = "➔";
                } else {
                    btn.className = "subtopic-btn locked";
                    btn.querySelector('span:last-child').innerText = "🔒";
                }
            });

            updateCablesAndElectron();
        }

        function toggleChapter(id) {
            if (!gameState.unlockedChaps.includes(id)) return;
            document.getElementById(id).classList.toggle('open');
            setTimeout(updateCablesAndElectron, 300);
        }

        /* --- CABLE SVG Y ANIMACIÓN DE ELECTRÓN --- */
        function updateCablesAndElectron() {
            const wrapper = document.getElementById('map-wrapper');
            const svg = document.getElementById('cable-svg');
            const rectW = wrapper.getBoundingClientRect();

            svg.setAttribute('width', rectW.width);
            svg.setAttribute('height', rectW.height);

            const nodes = ['node-ground', 'chap-1', 'chap-2', 'chap-3', 'chap-4', 'node-source'];
            let pathD = "";

            for (let i = 0; i < nodes.length - 1; i++) {
                const elA = document.getElementById(nodes[i]);
                const elB = document.getElementById(nodes[i+1]);

                const targetA = elA.querySelector('.chapter-circle') || elA;
                const targetB = elB.querySelector('.chapter-circle') || elB;

                const rA = targetA.getBoundingClientRect();
                const rB = targetB.getBoundingClientRect();

                const x1 = rA.left + rA.width / 2 - rectW.left;
                const y1 = rA.top + rA.height / 2 - rectW.top;
                const x2 = rB.left + rB.width / 2 - rectW.left;
                const y2 = rB.top + rB.height / 2 - rectW.top;

                const deltaY = y2 - y1;
                const curveOffset = (i % 2 === 0) ? 120 : -120;

                if (i === 0) pathD += `M ${x1} ${y1} `;
                pathD += `C ${x1 + curveOffset} ${y1 + deltaY * 0.5}, ${x2 - curveOffset} ${y1 + deltaY * 0.5}, ${x2} ${y2} `;
            }

            document.getElementById('path-copper-base').setAttribute('d', pathD);
            document.getElementById('path-copper-core').setAttribute('d', pathD);
            document.getElementById('path-copper-texture').setAttribute('d', pathD);
        }

        function animateElectron() {
            electronAnimProgress += 0.003;
            if (electronAnimProgress > 1) electronAnimProgress = 0;

            const path = document.getElementById('path-copper-core');
            const electronGroup = document.getElementById('electron-group');

            if (path && path.getTotalLength) {
                const len = path.getTotalLength();
                const pt = path.getPointAtLength(electronAnimProgress * len);
                electronGroup.setAttribute('transform', `translate(${pt.x}, ${pt.y})`);
            }

            requestAnimationFrame(animateElectron);
        }

        /* --- MANEJO DE MISIONES Y SUBPASOS --- */
        function openQuest(key) {
            if (!gameState.unlockedQuests.includes(key)) return;
            activeQuest = key;
            activeSubstep = 0;
            const q = questsDB[key];

            if (q.introChar && !gameState.unlockedChars.includes(q.introChar)) {
                gameState.unlockedChars.push(q.introChar);
                showCharReveal(q.introChar);
            }

            document.getElementById('modal-title').innerText = q.title;

            katex.render(q.eq1, document.getElementById('eq-1'), { displayMode: true, throwOnError: false });
            katex.render(q.eq2, document.getElementById('eq-2'), { displayMode: true, throwOnError: false });

            document.getElementById('sim-legend').innerHTML = q.legend;
            document.getElementById('sim-controls').innerHTML = q.controls;

            renderSubstepView();
            document.getElementById('quest-modal').classList.add('active');

            currentFullText = q.narrative;
            startTypewriter('narrative-text-el', q.narrative);

            setTimeout(() => {
                const schemaCv = document.getElementById('schema-canvas');
                if (schemaCv && q.drawSchema) q.drawSchema(schemaCv.getContext('2d'));
            }, 50);
        }

        function renderSubstepView() {
            if (demoInterval) { clearInterval(demoInterval); demoInterval = null; }

            const q = questsDB[activeQuest];
            
            // Actualizar barra de pills
            for (let i = 0; i < 6; i++) {
                const pill = document.getElementById(`pill-${i}`);
                pill.className = "step-pill" + (i === 5 ? " boss" : "");
                if (i < activeSubstep) pill.classList.add('completed');
                if (i === activeSubstep) pill.classList.add('active');
            }

            document.getElementById('progress-step-text').innerText = `Paso ${activeSubstep + 1} de 6`;

            // Ocultar vistas
            document.getElementById('step-0').style.display = 'none';
            document.getElementById('step-sim-view').style.display = 'none';

            if (activeSubstep === 0) {
                document.getElementById('step-0').style.display = 'flex';
            } else {
                document.getElementById('step-sim-view').style.display = 'flex';
                const challenge = q.challenges[activeSubstep - 1];
                
                const banner = document.getElementById('challenge-banner');
                banner.className = "challenge-box" + (activeSubstep === 5 ? " boss" : "");
                document.getElementById('challenge-title').innerText = challenge.title;
                document.getElementById('challenge-desc').innerText = challenge.desc;

                const btnNext = document.getElementById('btn-next-step');
                btnNext.className = "nav-btn" + (activeSubstep === 5 ? " boss-btn" : "");
                btnNext.innerText = activeSubstep === 5 ? "Completar Misión 👾" : "Continuar ➔";

                setTimeout(() => {
                    q.drawSim();
                    renderMathInLegend();

                    // Modo Demo Auto-Animado
                    if (activeSubstep === 1) {
                        let t = 0;
                        demoInterval = setInterval(() => {
                            t += 0.08;
                            const inputs = document.querySelectorAll('#sim-controls input[type="range"]');
                            inputs.forEach((inp, idx) => {
                                const min = parseFloat(inp.min), max = parseFloat(inp.max);
                                inp.value = min + (max - min) * (0.5 + 0.5 * Math.sin(t + idx));
                                inp.dispatchEvent(new Event('input'));
                            });
                        }, 50);
                    }
                }, 50);
            }
        }

        function nextSubstep() {
            const q = questsDB[activeQuest];

            // Validar ejercicio actual si estamos entre los pasos 2 a 5
            if (activeSubstep >= 2 && activeSubstep <= 5) {
                const challenge = q.challenges[activeSubstep - 1];
                if (challenge.check && !challenge.check()) {
                    alert("⚠️ ¡El simulador no está posicionado en los valores requeridos! Revisa las instrucciones.");
                    return;
                }
            }

            if (activeSubstep < 5) {
                activeSubstep++;
                renderSubstepView();
            } else {
                completeCurrentQuest();
            }
        }

        function prevSubstep() {
            if (activeSubstep > 0) {
                activeSubstep--;
                renderSubstepView();
            }
        }

        function startTypewriter(elementId, text, speed = 18) {
            if (typewriterTimeout) clearTimeout(typewriterTimeout);
            const el = document.getElementById(elementId);
            el.innerHTML = "";
            let i = 0;

            function type() {
                if (i < text.length) {
                    if (text.substring(i, i + 4) === "<br>") {
                        el.innerHTML += "<br>"; i += 4;
                    } else if (text.charAt(i) === "<") {
                        let closeIdx = text.indexOf(">", i);
                        if (closeIdx !== -1) {
                            el.innerHTML += text.substring(i, closeIdx + 1); i = closeIdx + 1;
                        } else { el.innerHTML += text.charAt(i); i++; }
                    } else {
                        el.innerHTML += text.charAt(i); i++;
                    }
                    typewriterTimeout = setTimeout(type, speed);
                }
            }
            type();
        }

        function skipTypewriter() {
            if (typewriterTimeout) clearTimeout(typewriterTimeout);
            document.getElementById('narrative-text-el').innerHTML = currentFullText;
        }

        function renderMathInLegend() {
            const leg = document.getElementById('sim-legend');
            if (window.renderMathInElement && leg) {
                renderMathInElement(leg, { delimiters: [{left: '$', right: '$', display: false}], throwOnError: false });
            }
        }

        function closeQuest() {
            if (typewriterTimeout) clearTimeout(typewriterTimeout);
            if (demoInterval) clearInterval(demoInterval);
            document.getElementById('quest-modal').classList.remove('active');
            activeQuest = null;
        }

        function completeCurrentQuest() {
            if (!activeQuest || gameState.completedQuests.includes(activeQuest)) {
                closeQuest(); return;
            }

            gameState.completedQuests.push(activeQuest);
            const q = questsDB[activeQuest];

            if (q.next && !gameState.unlockedQuests.includes(q.next)) {
                gameState.unlockedQuests.push(q.next);
                const nextChap = questsDB[q.next].chap;
                if (!gameState.unlockedChaps.includes(nextChap)) {
                    gameState.unlockedChaps.push(nextChap);
                }
            }

            gameState.xp += 100;
            if (gameState.xp >= gameState.level * 200) gameState.level++;

            updateUI();
            closeQuest();
        }

        /* --- CODEX & REVEAL MODAL --- */
        function openCodex() {
            const grid = document.getElementById('codex-grid');
            grid.innerHTML = '';

            Object.keys(charactersDB).forEach(cKey => {
                const c = charactersDB[cKey];
                const isUn = gameState.unlockedChars.includes(cKey);

                const card = document.createElement('div');
                card.className = `char-card ${isUn ? 'unlocked' : 'locked'}`;
                card.innerHTML = `<div class="char-symbol">${c.symbol}</div><div class="char-name">${isUn ? c.name.split(' ')[0] : 'Incógnito'}</div>`;
                
                if (isUn) {
                    card.onclick = () => {
                        document.getElementById('char-detail-box').classList.add('active');
                        document.getElementById('char-detail-name').innerText = `${c.symbol} - ${c.name}`;
                        document.getElementById('char-detail-desc').innerHTML = c.desc;
                    };
                }
                grid.appendChild(card);
            });

            document.getElementById('codex-modal').classList.add('active');
        }

        function closeCodex() { document.getElementById('codex-modal').classList.remove('active'); }

        function showCharReveal(cKey) {
            const c = charactersDB[cKey];
            document.getElementById('reveal-symbol').innerText = c.symbol;
            document.getElementById('reveal-title').innerText = c.name;
            document.getElementById('reveal-desc').innerHTML = c.desc;
            document.getElementById('reveal-modal').classList.add('active');
        }

        function closeReveal() { document.getElementById('reveal-modal').classList.remove('active'); }

        /* --- ESQUEMAS ORIGEN EN TEORÍA --- */
        function drawSchemaSenoidal(ctx) {
            ctx.clearRect(0,0,650,130);
            ctx.fillStyle='#1e293b'; ctx.fillRect(50,20,50,80); ctx.fillRect(230,20,50,80);
            ctx.fillStyle='#ff0055'; ctx.font='14px Orbitron'; ctx.fillText('N', 70, 65);
            ctx.fillStyle='#00f3ff'; ctx.fillText('S', 250, 65);
            ctx.strokeStyle='#ffb700'; ctx.lineWidth=3; ctx.beginPath(); ctx.arc(165,60,25,0,Math.PI*2); ctx.stroke();
            ctx.fillStyle='#fff'; ctx.font='11px Rajdhani'; ctx.fillText('Generador AC (ω)', 125, 110);
            ctx.strokeStyle='#00ff66'; ctx.lineWidth=2; ctx.beginPath(); ctx.moveTo(300,60); ctx.lineTo(370,60); ctx.stroke();
            ctx.fillStyle='#00ff66'; ctx.fillText('v(t) = Vm·cos(ωt+ϕ)', 380, 65);
        }

        function drawSchemaRLC(ctx) {
            ctx.clearRect(0,0,650,130);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2; ctx.strokeRect(90,20,440,80);
            ctx.clearRect(110,10,40,20); ctx.clearRect(270,10,40,20); ctx.clearRect(430,10,40,20);
            ctx.fillStyle='#ffb700'; ctx.font='12px Orbitron';
            ctx.fillText('R', 125, 25); ctx.fillText('L', 285, 25); ctx.fillText('C', 445, 25);
            ctx.fillStyle='#00ff66'; ctx.fillText('Impedancia Z = R + j(XL - XC)', 200, 85);
        }

        function drawSchemaKCL(ctx) {
            ctx.clearRect(0,0,650,130);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.arc(325,65,7,0,Math.PI*2); ctx.fillStyle='#ff0055'; ctx.fill();
            ctx.fillStyle='#fff'; ctx.font='12px Orbitron'; ctx.fillText('Nodo Complejo ΣI = 0', 260, 35);
            ctx.beginPath(); ctx.moveTo(220,65); ctx.lineTo(318,65); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(332,65); ctx.lineTo(430,35); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(332,65); ctx.lineTo(430,95); ctx.stroke();
        }

        function drawSchemaPinst(ctx) {
            ctx.clearRect(0,0,650,130);
            ctx.fillStyle='#1e293b'; ctx.fillRect(100,30,110,60);
            ctx.fillStyle='#00f3ff'; ctx.font='12px Orbitron'; ctx.fillText('Fuente AC', 120, 65);
            ctx.strokeStyle='#00ff66'; ctx.lineWidth=2; ctx.strokeRect(400,30,110,60);
            ctx.fillStyle='#00ff66'; ctx.fillText('Carga ZL', 430, 65);
            ctx.beginPath(); ctx.moveTo(210,45); ctx.lineTo(400,45); ctx.moveTo(210,75); ctx.lineTo(400,75); ctx.stroke();
        }

        function drawSchemaTriang(ctx) {
            ctx.clearRect(0,0,650,130);
            ctx.strokeStyle='#ff0055'; ctx.lineWidth=2; ctx.strokeRect(150,25,90,80);
            ctx.fillStyle='#ff0055'; ctx.font='11px Orbitron'; ctx.fillText('Motor Inductivo', 152, 65);
            ctx.strokeStyle='#00f3ff'; ctx.strokeRect(350,25,90,80);
            ctx.fillStyle='#00f3ff'; ctx.fillText('Resistencia R', 355, 65);
            ctx.fillStyle='#ffb700'; ctx.fillText('Absorbe Q (VAR)', 145, 120);
            ctx.fillStyle='#00f3ff'; ctx.fillText('Absorbe P (W)', 355, 120);
        }

        function drawSchemaFP(ctx) {
            ctx.clearRect(0,0,650,130);
            ctx.strokeStyle='#00ff66'; ctx.lineWidth=2; ctx.strokeRect(80,25,110,70);
            ctx.fillStyle='#00ff66'; ctx.font='11px Orbitron'; ctx.fillText('Fábrica', 110, 65);
            ctx.strokeStyle='#ff0055'; ctx.strokeRect(300,25,110,70);
            ctx.fillStyle='#ff0055'; ctx.fillText('Capacitancia', 310, 65);
            ctx.fillStyle='#ffb700'; ctx.fillText('Compensación -QC => FP ≥ 0.95', 430, 65);
        }

        function drawSchemaMutua(ctx) {
            ctx.clearRect(0,0,650,130);
            ctx.strokeStyle='#ffb700'; ctx.lineWidth=3;
            ctx.beginPath(); ctx.arc(200,65,30,0,Math.PI*2); ctx.stroke();
            ctx.beginPath(); ctx.arc(450,65,30,0,Math.PI*2); ctx.stroke();
            ctx.strokeStyle='rgba(0,243,255,0.6)'; ctx.lineWidth=1.5; ctx.setLineDash([4,4]);
            ctx.beginPath(); ctx.moveTo(230,65); ctx.lineTo(420,65); ctx.stroke(); ctx.setLineDash([]);
        }

        function drawSchemaDots(ctx) {
            ctx.clearRect(0,0,650,130);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.arc(220,65,25,0,Math.PI*2); ctx.stroke();
            ctx.beginPath(); ctx.arc(430,65,25,0,Math.PI*2); ctx.stroke();
            ctx.fillStyle='#ff0055'; ctx.beginPath(); ctx.arc(220,30,5,0,Math.PI*2); ctx.fill();
            ctx.beginPath(); ctx.arc(430,30,5,0,Math.PI*2); ctx.fill();
        }

        function drawSchemaTransfo(ctx) {
            ctx.clearRect(0,0,650,130);
            ctx.fillStyle='#1e293b'; ctx.fillRect(220,20,210,90); ctx.clearRect(260,40,130,50);
            ctx.fillStyle='#ffb700'; ctx.font='11px Orbitron'; ctx.fillText('N1', 180, 65);
            ctx.fillStyle='#00f3ff'; ctx.fillText('N2', 450, 65);
        }

        function drawSchemaTriGen(ctx) {
            ctx.clearRect(0,0,650,130);
            const colors = ['#ff0055', '#00f3ff', '#00ff66'], angles = [0, 120, 240];
            angles.forEach((ang, i) => {
                const rad = (ang * Math.PI) / 180, x = 325 + 40 * Math.cos(rad), y = 65 + 40 * Math.sin(rad);
                ctx.strokeStyle = colors[i]; ctx.lineWidth = 3;
                ctx.beginPath(); ctx.moveTo(325,65); ctx.lineTo(x,y); ctx.stroke();
            });
        }

        function drawSchemaYDelta(ctx) {
            ctx.clearRect(0,0,650,130);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.moveTo(150,30); ctx.lineTo(150,65); ctx.lineTo(120,95); ctx.moveTo(150,65); ctx.lineTo(180,95); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(480,30); ctx.lineTo(440,95); ctx.lineTo(520,95); ctx.closePath(); ctx.stroke();
        }

        function drawSchemaDeseb(ctx) {
            ctx.clearRect(0,0,650,130);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.moveTo(100,65); ctx.lineTo(550,65); ctx.stroke();
            ctx.fillStyle='#ffb700'; ctx.font='12px Orbitron'; ctx.fillText('Retorno por Neutro IN ≠ 0', 240, 100);
        }

        /* --- SIMULADORES EN CANVA --- */
        function drawSimSenoidal() {
            const Vm = parseFloat(document.getElementById('input-vm').value);
            const phiDeg = parseFloat(document.getElementById('input-phi').value);
            document.getElementById('val-vm').innerText = Vm;
            document.getElementById('val-phi').innerText = phiDeg;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const cy = 105, cxPhasor = 110, phiRad = (phiDeg * Math.PI) / 180, r = Vm * 0.55;

            ctx.strokeStyle = 'rgba(255,255,255,0.08)'; ctx.lineWidth = 1;
            for(let x=0; x<cv.width; x+=30) { ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x,cv.height); ctx.stroke(); }
            for(let y=0; y<cv.height; y+=30) { ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(cv.width,y); ctx.stroke(); }

            ctx.strokeStyle = 'rgba(255,255,255,0.2)'; ctx.lineWidth = 1.5;
            ctx.beginPath(); ctx.moveTo(15, cy); ctx.lineTo(205, cy); ctx.moveTo(cxPhasor, 15); ctx.lineTo(cxPhasor, 195); ctx.stroke();

            const fx = cxPhasor + r * Math.cos(-phiRad), fy = cy + r * Math.sin(-phiRad);
            ctx.strokeStyle = '#ff0055'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(cxPhasor, cy); ctx.lineTo(fx, fy); ctx.stroke();

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 1.5; ctx.setLineDash([4, 4]);
            ctx.beginPath(); ctx.moveTo(fx, fy); ctx.lineTo(240, fy); ctx.stroke(); ctx.setLineDash([]);

            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 2.5; ctx.beginPath();
            for (let x = 240; x <= 630; x++) {
                const t = (x - 240) * 0.02;
                const y = cy - r * Math.sin(t + phiRad);
                if (x === 240) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();
        }

        function drawSimRLC() {
            const L = parseFloat(document.getElementById('input-l').value) * 1e-3;
            const C = parseFloat(document.getElementById('input-c').value) * 1e-6;
            document.getElementById('val-l').innerText = document.getElementById('input-l').value;
            document.getElementById('val-c').innerText = document.getElementById('input-c').value;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const f0 = 1 / (2 * Math.PI * Math.sqrt(L * C));

            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 2.5; ctx.beginPath();
            for (let x = 40; x < 610; x++) {
                const f = (x - 40) * 5, w = 2 * Math.PI * f;
                const Xl = w * L, Xc = w > 0 ? 1 / (w * C) : 1000;
                const Z = Math.sqrt(15*15 + Math.pow(Xl - Xc, 2));
                const y = 180 - Math.min(Z * 1.1, 160);
                if (x === 40) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();

            const xRes = 40 + (f0 / 5);
            if (xRes >= 40 && xRes <= 610) {
                ctx.strokeStyle = '#ffb700'; ctx.setLineDash([4, 4]);
                ctx.beginPath(); ctx.moveTo(xRes, 10); ctx.lineTo(xRes, 190); ctx.stroke(); ctx.setLineDash([]);
                ctx.fillStyle = '#ffb700'; ctx.font = '12px Orbitron';
                ctx.fillText(`f0 = ${Math.round(f0)} Hz`, Math.min(xRes + 8, 500), 25);
            }
        }

        function drawSimKCL() {
            const Z2 = parseFloat(document.getElementById('input-z2').value);
            document.getElementById('val-z2').innerText = Z2;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            ctx.fillStyle = '#00f3ff'; ctx.font = '13px Orbitron';
            ctx.fillText(`Matriz de Admitancia [Y]:`, 40, 35);
            ctx.fillStyle = '#fff'; ctx.font = '12px Rajdhani';
            ctx.fillText(`[  (1/10 + 1/${Z2})   -1/${Z2}  ]  ·  [ V1 ]  =  [ 10∠0° ]`, 40, 70);
            ctx.fillText(`[     -1/${Z2}          1/${Z2}     ]     [ V2 ]     [   0   ]`, 40, 100);

            const V1 = (120 * (1 + 10/Z2)).toFixed(1);
            ctx.fillStyle = '#00ff66'; ctx.font = '13px Orbitron';
            ctx.fillText(`V1 = ${V1} V ∠ 0°`, 40, 150);
        }

        function drawSimPinst() {
            const Vrms = parseFloat(document.getElementById('input-vrms').value);
            document.getElementById('val-vrms').innerText = Vrms;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const Pprom = (Vrms * 1.5).toFixed(0);

            ctx.strokeStyle = '#ff0055'; ctx.lineWidth = 2; ctx.beginPath();
            for(let x=50; x<600; x++) {
                const t = (x-50)*0.03, p = Pprom * (1 + Math.cos(t));
                const y = 180 - p * 0.35;
                if(x===50) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 2; ctx.setLineDash([5,5]);
            const yP = 180 - Pprom * 0.35;
            ctx.beginPath(); ctx.moveTo(50, yP); ctx.lineTo(600, yP); ctx.stroke(); ctx.setLineDash([]);

            ctx.fillStyle = '#ffb700'; ctx.font = '12px Orbitron';
            ctx.fillText(`P_prom = ${Pprom} W`, 480, yP - 8);
        }

        function drawSimTriang() {
            const QL = parseFloat(document.getElementById('input-ql').value);
            document.getElementById('val-ql').innerText = QL;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const P = 300, ox = 80, oy = 175, sc = 0.35;
            const S = Math.sqrt(P*P + QL*QL);

            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 3; ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P*sc, oy); ctx.stroke();
            ctx.strokeStyle = '#ff0055'; ctx.beginPath(); ctx.moveTo(ox + P*sc, oy); ctx.lineTo(ox + P*sc, oy - QL*sc); ctx.stroke();
            ctx.strokeStyle = '#00ff66'; ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P*sc, oy - QL*sc); ctx.stroke();

            ctx.fillStyle = '#fff'; ctx.font = '12px Orbitron';
            ctx.fillText(`|S| = ${Math.round(S)} VA`, 320, 70);
            ctx.fillText(`FP = ${(P/S).toFixed(2)}`, 320, 100);
        }

        function drawSimFP() {
            const QC = parseFloat(document.getElementById('input-qc').value);
            document.getElementById('val-qc').innerText = QC;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const P = 300, QL = 300, Qnet = Math.max(0, QL - QC);
            const S = Math.sqrt(P*P + Qnet*Qnet);
            const FP = (P / S).toFixed(2);

            const ox = 80, oy = 175, sc = 0.35;

            ctx.strokeStyle = 'rgba(255,255,255,0.2)'; ctx.lineWidth = 1;
            ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P*sc, oy - QL*sc); ctx.stroke();

            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P*sc, oy); ctx.lineTo(ox + P*sc, oy - Qnet*sc); ctx.closePath(); ctx.stroke();

            ctx.fillStyle = '#00ff66'; ctx.font = '13px Orbitron';
            ctx.fillText(`FP Resultante = ${FP}`, 320, 70);
            ctx.fillStyle = FP >= 0.95 ? '#00ff66' : '#ffb700';
            ctx.fillText(FP >= 0.95 ? '✔ FP EFICIENTE (SIN PENALIZACIÓN)' : '⚠️ FP BAJO (REQUIERE MÁS QC)', 320, 105);
        }

        function drawSimMutua() {
            const k = parseFloat(document.getElementById('input-k').value);
            document.getElementById('val-k').innerText = k;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 4; ctx.beginPath(); ctx.arc(180, 105, 40, 0, Math.PI*2); ctx.stroke();
            ctx.strokeStyle = '#00f3ff'; ctx.beginPath(); ctx.arc(470, 105, 40, 0, Math.PI*2); ctx.stroke();

            ctx.strokeStyle = `rgba(0, 255, 102, ${k})`; ctx.lineWidth = k * 5;
            for(let r = 20; r <= 70; r += 20) {
                ctx.beginPath(); ctx.ellipse(325, 105, 145, r, 0, 0, Math.PI*2); ctx.stroke();
            }
        }

        function drawSimDots() {
            const dir = parseInt(document.getElementById('input-dir').value);
            document.getElementById('val-dir').innerText = dir === 1 ? 'Mismo Punto' : 'Punto Opuesto';

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 3;
            ctx.strokeRect(180, 50, 70, 100); ctx.strokeRect(390, 50, 70, 100);

            ctx.fillStyle = '#ff0055';
            ctx.beginPath(); ctx.arc(215, 35, 6, 0, Math.PI*2); ctx.fill();
            ctx.beginPath(); ctx.arc(425, dir === 1 ? 35 : 165, 6, 0, Math.PI*2); ctx.fill();

            ctx.fillStyle = '#fff'; ctx.font = '12px Orbitron';
            ctx.fillText(dir === 1 ? 'Voltajes en FASE (+M)' : 'Voltajes en OPOSICIÓN (-M)', 180, 195);
        }

        function drawSimTransfo() {
            const a = parseFloat(document.getElementById('input-a').value);
            document.getElementById('val-a').innerText = a;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const V1 = 120, V2 = (V1 / a).toFixed(1);

            ctx.fillStyle = '#1e293b'; ctx.fillRect(220, 35, 210, 110); ctx.clearRect(260, 60, 130, 60);

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 2; ctx.beginPath();
            for(let x=30; x<200; x++) {
                const y = 90 - 25 * Math.sin((x-30)*0.05);
                if(x===30) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();

            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 2; ctx.beginPath();
            for(let x=450; x<620; x++) {
                const y = 90 - (25/a) * Math.sin((x-450)*0.05);
                if(x===450) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();

            ctx.fillStyle = '#fff'; ctx.font = '12px Orbitron';
            ctx.fillText(`V1 = ${V1} V`, 80, 145); ctx.fillText(`V2 = ${V2} V`, 500, 145);
        }

        function drawSimTriGen() {
            const Vp = parseFloat(document.getElementById('input-vp').value);
            document.getElementById('val-vp').innerText = Vp;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const cx = 325, cy = 105, r = Vp * 0.55;
            const angles = [0, -120, 120], colors = ['#ff0055', '#00f3ff', '#00ff66'];

            angles.forEach((ang, i) => {
                const rad = (ang * Math.PI) / 180, x = cx + r * Math.cos(rad), y = cy + r * Math.sin(rad);
                ctx.strokeStyle = colors[i]; ctx.lineWidth = 3;
                ctx.beginPath(); ctx.moveTo(cx, cy); ctx.lineTo(x, y); ctx.stroke();
            });

            const VL = (Math.sqrt(3) * Vp).toFixed(1);
            ctx.fillStyle = '#fff'; ctx.font = '12px Orbitron';
            ctx.fillText(`Tensión de Línea VL = √3 · ${Vp} = ${VL} V`, 30, 25);
        }

        function drawSimYDelta() {
            const Ip = parseFloat(document.getElementById('input-ip').value);
            document.getElementById('val-ip').innerText = Ip;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const IL = (Math.sqrt(3) * Ip).toFixed(1);

            ctx.fillStyle = '#00f3ff'; ctx.font = '13px Orbitron';
            ctx.fillText(`Corriente de Fase Ip = ${Ip} A`, 100, 70);
            ctx.fillStyle = '#00ff66';
            ctx.fillText(`Corriente de Línea En Delta IL = √3 · Ip = ${IL} A`, 100, 115);
        }

        function drawSimDeseb() {
            const des = parseFloat(document.getElementById('input-des').value);
            document.getElementById('val-des').innerText = des;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const IN = (des * 0.12).toFixed(2);

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(100, 105); ctx.lineTo(100 + des * 3, 105); ctx.stroke();

            ctx.fillStyle = '#ffb700'; ctx.font = '13px Orbitron';
            ctx.fillText(`Corriente de Retorno por Neutro IN = ${IN} A`, 100, 70);
        }
    </script>
</body>
</html>