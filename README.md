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

        /* --- MAPA PRINCIPAL CON CABLES CURVADOS Y ELECTRÓN --- */
        #map-wrapper {
            position: relative;
            max-width: 900px;
            width: 100%;
            margin: 30px auto;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        /* CAPA SVG PARA CABLES CURVADOS Y SPRITES */
        #cable-svg {
            position: absolute;
            top: 0; left: 0;
            width: 100%; height: 100%;
            pointer-events: none;
            z-index: 1;
        }

        .cable-path-bg {
            fill: none;
            stroke: rgba(255, 255, 255, 0.08);
            stroke-width: 8;
            stroke-linecap: round;
        }

        .cable-path-active {
            fill: none;
            stroke: var(--cyan);
            stroke-width: 4;
            stroke-linecap: round;
            filter: drop-shadow(0 0 8px var(--cyan));
            transition: stroke 0.5s;
        }

        .electron-sprite {
            fill: #fff;
            filter: drop-shadow(0 0 10px var(--cyan)) drop-shadow(0 0 20px var(--cyan));
        }

        /* ESTRUCTURA DE CAPÍTULOS Y SUBTEMAS */
        .chapter-stage {
            position: relative;
            z-index: 2;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin-bottom: 90px;
            width: 100%;
        }

        /* NODO CIRCULAR CON PORTADA DIFUMINADA */
        .chapter-circle {
            width: 160px;
            height: 160px;
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

        .chapter-circle.unlocked {
            border-color: var(--cyan);
            box-shadow: 0 0 30px rgba(0, 243, 255, 0.3);
        }
        .chapter-circle.completed {
            border-color: var(--green);
            box-shadow: 0 0 30px rgba(0, 255, 102, 0.3);
        }
        .chapter-circle.locked {
            opacity: 0.45;
            filter: grayscale(1);
            cursor: not-allowed;
            border-color: var(--text-muted);
        }

        .chapter-circle:hover:not(.locked) {
            transform: scale(1.08);
            box-shadow: 0 0 40px var(--cyan);
        }

        .chapter-cover-img {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background-size: cover;
            background-position: center;
            filter: blur(2.5px) brightness(0.45);
            transition: all 0.3s;
        }

        .chapter-circle:hover .chapter-cover-img {
            filter: blur(0px) brightness(0.65);
            transform: scale(1.1);
        }

        .chapter-overlay-content {
            position: relative;
            z-index: 3;
            text-align: center;
            padding: 10px;
        }

        .chapter-num {
            font-family: 'Orbitron', sans-serif;
            font-size: 0.8rem;
            color: var(--yellow);
            text-shadow: 0 0 8px #000;
            letter-spacing: 1px;
        }

        .chapter-name-title {
            font-family: 'Orbitron', sans-serif;
            font-size: 0.95rem;
            font-weight: 800;
            color: #fff;
            text-shadow: 0 0 10px #000, 0 0 5px #000;
            margin: 4px 0;
            line-height: 1.1;
        }

        .chapter-badge {
            font-family: 'Orbitron', sans-serif;
            font-size: 0.65rem;
            padding: 2px 8px;
            border-radius: 10px;
            background: rgba(0,0,0,0.8);
            border: 1px solid var(--cyan);
            color: var(--cyan);
            margin-top: 4px;
            display: inline-block;
        }
        .completed .chapter-badge { border-color: var(--green); color: var(--green); }
        .locked .chapter-badge { border-color: var(--text-muted); color: var(--text-muted); }

        /* DESPLEGABLE DE SUBTEMAS ABAJO DEL NODO CIRCULAR */
        .subtopics-drawer {
            display: none;
            width: 100%;
            max-width: 650px;
            margin-top: 20px;
            padding: 15px;
            background: rgba(11, 18, 32, 0.95);
            border: 1px solid var(--panel-border);
            border-radius: 10px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.8);
            grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
            gap: 12px;
            animation: fadeIn 0.3s ease;
        }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(-10px); } to { opacity: 1; transform: translateY(0); } }

        .chapter-stage.open .subtopics-drawer { display: grid; }

        .subtopic-btn {
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid rgba(255, 255, 255, 0.15);
            border-radius: 8px;
            padding: 12px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            cursor: pointer;
            transition: all 0.2s;
            font-size: 0.95rem;
        }
        .subtopic-btn:hover:not(.locked) { transform: translateY(-2px); border-color: var(--cyan); box-shadow: 0 0 12px rgba(0, 243, 255, 0.2); }
        .subtopic-btn.completed { border-color: var(--green); background: rgba(0, 255, 102, 0.05); }
        .subtopic-btn.locked { opacity: 0.4; cursor: not-allowed; }

        /* --- MODALES Y NARRATIVA --- */
        .modal-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(4, 6, 12, 0.9);
            backdrop-filter: blur(8px);
            z-index: 1000;
            display: flex; justify-content: center; align-items: center;
            opacity: 0; pointer-events: none; transition: opacity 0.25s ease;
        }
        .modal-overlay.active { opacity: 1; pointer-events: all; }

        .modal-card {
            background: var(--panel-bg);
            border: 2px solid var(--cyan);
            border-radius: 12px;
            width: 90%; max-width: 900px;
            max-height: 92vh;
            display: flex; flex-direction: column;
            overflow: hidden;
            box-shadow: 0 0 35px rgba(0, 243, 255, 0.25);
        }

        .modal-header {
            padding: 15px 25px;
            border-bottom: 1px solid var(--panel-border);
            display: flex; justify-content: space-between; align-items: center;
            background: rgba(0, 243, 255, 0.05);
        }
        .modal-header h3 { font-family: 'Orbitron', sans-serif; color: var(--cyan); font-size: 1.1rem; }
        .close-btn { background: none; border: none; color: var(--text-muted); font-size: 1.8rem; cursor: pointer; }
        .close-btn:hover { color: var(--red); }

        .modal-body { padding: 25px; overflow-y: auto; display: flex; flex-direction: column; gap: 20px; }

        .step-view { display: none; flex-direction: column; gap: 18px; }
        .step-view.active { display: flex; }

        .narrative-box {
            background: rgba(0, 0, 0, 0.6);
            border: 1px solid var(--cyan);
            border-radius: 8px;
            padding: 18px;
            cursor: pointer;
            min-height: 90px;
        }
        .narrative-header {
            font-family: 'Orbitron', sans-serif;
            font-size: 0.75rem;
            color: var(--yellow);
            margin-bottom: 8px;
            display: flex; justify-content: space-between;
        }
        .narrative-text { font-size: 1.05rem; line-height: 1.5; color: #fff; }
        .typing-cursor::after { content: '▌'; color: var(--cyan); animation: blink 0.8s infinite; }
        @keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }

        .schema-box {
            background: #03060d;
            border: 1px solid rgba(255,255,255,0.1);
            border-radius: 8px; padding: 12px;
            display: flex; flex-direction: column; align-items: center; gap: 8px;
        }
        .schema-title { font-family: 'Orbitron', sans-serif; font-size: 0.8rem; color: var(--cyan); align-self: flex-start; }

        .math-card {
            background: rgba(255, 255, 255, 0.02);
            border: 1px solid rgba(255, 255, 255, 0.08);
            padding: 12px 18px; border-radius: 6px;
            display: flex; flex-direction: column; gap: 8px;
        }
        .math-title { font-family: 'Orbitron', sans-serif; font-size: 0.8rem; color: var(--yellow); }
        .math-eq { font-size: 1.1rem; padding: 4px 0; overflow-x: auto; }

        .sim-box {
            background: #000;
            border: 1px solid var(--panel-border);
            border-radius: 8px; padding: 15px;
            display: flex; flex-direction: column; align-items: center; gap: 12px;
        }

        canvas { background: #050811; border-radius: 4px; border: 1px solid rgba(0, 243, 255, 0.2); max-width: 100%; }

        .legend-box {
            width: 100%; background: rgba(255,255,255,0.03);
            border: 1px solid rgba(255,255,255,0.08);
            border-radius: 6px; padding: 10px 15px;
            font-size: 0.85rem; display: flex; flex-direction: column; gap: 6px;
        }
        .legend-item { display: flex; align-items: center; gap: 8px; }
        .legend-color { width: 12px; height: 12px; border-radius: 2px; }

        .controls-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 12px; width: 100%; }
        .control-group { display: flex; flex-direction: column; gap: 4px; }
        .control-group label { font-size: 0.8rem; color: var(--text-muted); display: flex; justify-content: space-between; }
        input[type="range"] { accent-color: var(--cyan); cursor: pointer; }

        .nav-btn {
            font-family: 'Orbitron', sans-serif;
            background: linear-gradient(135deg, rgba(0,243,255,0.2), rgba(0,255,102,0.2));
            border: 1px solid var(--cyan); color: #fff;
            padding: 12px 24px; border-radius: 6px; font-size: 0.95rem; font-weight: 700;
            cursor: pointer; transition: all 0.2s; align-self: flex-end;
            display: flex; align-items: center; gap: 8px; margin-top: 5px;
        }
        .nav-btn:hover { background: var(--cyan); color: #000; box-shadow: 0 0 15px var(--cyan); }

        /* CODEX */
        .codex-grid {
            display: grid; grid-template-columns: repeat(auto-fill, minmax(110px, 1fr));
            gap: 15px; padding: 10px 0;
        }
        .char-card {
            background: rgba(255,255,255,0.03);
            border: 1px solid rgba(255,255,255,0.1);
            border-radius: 8px; padding: 15px 10px;
            display: flex; flex-direction: column; align-items: center;
            cursor: pointer; transition: all 0.2s;
        }
        .char-card.unlocked { border-color: var(--purple); background: rgba(157,0,255,0.08); }
        .char-card.unlocked:hover { transform: scale(1.05); box-shadow: 0 0 15px var(--purple); }
        .char-card.locked { opacity: 0.4; filter: grayscale(1); cursor: not-allowed; }

        .char-symbol { font-family: 'Orbitron', sans-serif; font-size: 1.8rem; font-weight: 900; color: var(--purple); margin-bottom: 5px; }
        .unlocked .char-symbol { color: var(--cyan); }
        .char-name { font-size: 0.75rem; text-align: center; color: var(--text-muted); }

        .char-detail-box {
            background: rgba(157, 0, 255, 0.1); border: 1px solid var(--purple);
            border-radius: 8px; padding: 15px; display: none; flex-direction: column; gap: 8px; margin-top: 15px;
        }
        .char-detail-box.active { display: flex; }
        .char-detail-title { font-family: 'Orbitron', sans-serif; color: var(--yellow); font-size: 1rem; }

        .reveal-card {
            background: linear-gradient(135deg, #0d1424, #1a0933);
            border: 2px solid var(--purple); box-shadow: 0 0 50px var(--purple);
            border-radius: 12px; padding: 30px; text-align: center;
            display: flex; flex-direction: column; align-items: center; gap: 15px;
            max-width: 400px; animation: popIn 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        @keyframes popIn { 0% { transform: scale(0.5); opacity: 0; } 100% { transform: scale(1); opacity: 1; } }

        .reveal-symbol {
            font-family: 'Orbitron', sans-serif; font-size: 3.5rem; font-weight: 900;
            color: var(--cyan); text-shadow: 0 0 20px var(--cyan);
        }
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

    <!-- MAPA PRINCIPAL CON NODOS CIRCULARES Y CABLE SVG CON ELECTRÓN -->
    <div id="map-wrapper">
        
        <!-- CANVAS SVG OVERLAY PARA LOS CABLES CURVADOS -->
        <svg id="cable-svg">
            <defs>
                <filter id="glow-cyan" x="-20%" y="-20%" width="140%" height="140%">
                    <feGaussianBlur stdDeviation="4" result="blur" />
                    <feComposite in="SourceGraphic" in2="blur" operator="over" />
                </filter>
            </defs>

            <!-- CABLE 1 -> 2 -->
            <path id="path-1-2" class="cable-path-bg" />
            <path id="path-1-2-active" class="cable-path-active" />

            <!-- CABLE 2 -> 3 -->
            <path id="path-2-3" class="cable-path-bg" />
            <path id="path-2-3-active" class="cable-path-active" />

            <!-- CABLE 3 -> 4 -->
            <path id="path-3-4" class="cable-path-bg" />
            <path id="path-3-4-active" class="cable-path-active" />

            <!-- SPRITE INTERACTIVO DEL ELECTRÓN EN MOVIMIENTO -->
            <g id="electron-group">
                <circle id="electron-core" r="7" class="electron-sprite" />
                <circle id="electron-halo" r="14" fill="rgba(0, 243, 255, 0.3)" />
            </g>
        </svg>

        <!-- NODO CAPÍTULO 1 -->
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

        <!-- NODO CAPÍTULO 2 -->
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

        <!-- NODO CAPÍTULO 3 -->
        <div class="chapter-stage locked" id="chap-3">
            <div class="chapter-circle locked" onclick="toggleChapter('chap-3')">
                <div class="chapter-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%230b1e10\'/><circle cx=\'70\' cy=\'100\' r=\'40\' stroke=\'%23ffb700\' stroke-width=\'6\' fill=\'none\'/><circle cx=\'130\' cy=\'100\' r=\'40\' stroke=\'%2300ff66\' stroke-width=\'6\' fill=\'none\'/></svg>');"></div>
                <div class="chapter-overlay-content">
                    <span class="chapter-num">CAPÍTULO 3</span>
                    <h4 class="chapter-name-title">Acoplamiento Magnético</h4>
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

        <!-- NODO CAPÍTULO 4 -->
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

    </div>

    <!-- MODAL DE MISIÓN -->
    <div class="modal-overlay" id="quest-modal">
        <div class="modal-card">
            <div class="modal-header">
                <h3 id="modal-title">Título de Misión</h3>
                <button class="close-btn" onclick="closeQuest()">&times;</button>
            </div>
            <div class="modal-body">
                
                <div class="step-view active" id="step-theory">
                    <div class="narrative-box" onclick="skipTypewriter()">
                        <div class="narrative-header">
                            <span>TRANSMISIÓN DE DATOS ACADÉMICOS</span>
                            <span style="font-size:0.65rem; color:var(--text-muted);">(Haz clic para mostrar todo)</span>
                        </div>
                        <div class="narrative-text typing-cursor" id="narrative-text-el"></div>
                    </div>

                    <div class="schema-box">
                        <div class="schema-title">🔍 ESQUEMA ORIGEN DE LAS VARIABLES</div>
                        <canvas id="schema-canvas" width="650" height="150"></canvas>
                    </div>
                    
                    <div class="math-card">
                        <div class="math-title">Ecuación Fundamental</div>
                        <div class="math-eq" id="eq-1"></div>
                    </div>

                    <div class="math-card">
                        <div class="math-title">Formulación Final de Análisis</div>
                        <div class="math-eq" id="eq-2"></div>
                    </div>

                    <button class="nav-btn" onclick="switchStep('sim')">
                        Continuar al Simulador ➔
                    </button>
                </div>

                <div class="step-view" id="step-sim">
                    <div class="sim-box">
                        <canvas id="sim-canvas" width="650" height="230"></canvas>
                        <div class="legend-box" id="sim-legend"></div>
                        <div class="controls-grid" id="sim-controls"></div>
                    </div>

                    <div style="display: flex; justify-content: space-between; width: 100%;">
                        <button class="nav-btn" style="background: rgba(255,255,255,0.1); border-color: var(--text-muted);" onclick="switchStep('theory')">
                            ⬅ Volver a Teoría
                        </button>
                        <button class="nav-btn" onclick="completeCurrentQuest()">
                            Completar Misión (+100 XP) ✔
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
                <p style="font-size: 0.9rem; color: var(--text-muted);">
                    Cada variable posee funciones únicas. Haz clic en un personaje descubierto para examinar sus datos:
                </p>
                <div class="codex-grid" id="codex-grid"></div>
                <div class="char-detail-box" id="char-detail-box">
                    <div class="char-detail-title" id="char-detail-name">Selecciona un personaje</div>
                    <div id="char-detail-desc" style="font-size: 0.95rem;"></div>
                </div>
            </div>
        </div>
    </div>

    <!-- MODAL REVELACIÓN DE PERSONAJE NUEVO -->
    <div class="modal-overlay" id="reveal-modal">
        <div class="reveal-card">
            <div style="font-family: 'Orbitron'; font-size: 0.8rem; color: var(--yellow);">¡NUEVO PERSONAJE DESBLOQUEADO!</div>
            <div class="reveal-symbol" id="reveal-symbol">V</div>
            <h3 id="reveal-title" style="color: #fff; font-family: 'Orbitron';">Nombre del Personaje</h3>
            <p id="reveal-desc" style="font-size: 0.9rem; color: var(--text-muted);"></p>
            <button class="nav-btn" onclick="closeReveal()" style="align-self: center;">¡Entendido!</button>
        </div>
    </div>

    <script>
        /* --- BASE DE DATOS DE PERSONAJES Y MISIONES --- */
        const charactersDB = {
            V: { symbol: 'V', name: 'Voltaje (Tensión)', desc: '<b>Clase:</b> Impulsor.<br><b>Función:</b> Fuerza electromotriz senoidal que impulsa los electrones a través del circuito.', introKey: '1-1' },
            I: { symbol: 'I', name: 'Corriente (Intensidad)', desc: '<b>Clase:</b> Torrente.<br><b>Función:</b> Flujo eléctrico que oscila en el tiempo con desfase respecto a la tensión.', introKey: '1-1' },
            w: { symbol: 'ω', name: 'Frecuencia Angular', desc: '<b>Clase:</b> Marcapasos.<br><b>Función:</b> Determina la velocidad angular de rotación de los fasores (ω = 2πf).', introKey: '1-1' },
            Z: { symbol: 'Z', name: 'Impedancia Compleja', desc: '<b>Clase:</b> Guardián.<br><b>Función:</b> Oposición total en AC (Resistencia real + Reactancia imaginaria).', introKey: '1-2' },
            K: { symbol: 'K', name: 'Leyes de Kirchhoff', desc: '<b>Clase:</b> Red de Mallas.<br><b>Función:</b> Mantiene la conservación de energía y carga mediante matrices fasoriales.', introKey: '1-3' },
            P: { symbol: 'P', name: 'Potencia Activa', desc: '<b>Clase:</b> Convertidor Real.<br><b>Función:</b> Energía promedio transformada efectivamente en trabajo útil (Watts).', introKey: '2-1' },
            S: { symbol: 'S', name: 'Potencia Aparente', desc: '<b>Clase:</b> Envolvente Total.<br><b>Función:</b> Capacidad total suministrada por la red eléctrica (VA).', introKey: '2-2' },
            FP: { symbol: 'FP', name: 'Factor de Potencia', desc: '<b>Clase:</b> Centinela de Eficiencia.<br><b>Función:</b> Razón cos(θ) entre la energía útil y la energía entregada.', introKey: '2-3' },
            M: { symbol: 'M', name: 'Inductancia Mutua', desc: '<b>Clase:</b> Enlace Magnético.<br><b>Función:</b> Induce tensión entre bobinas separadas mediante flujo magnético.', introKey: '3-1' },
            VL: { symbol: 'VL', name: 'Tensión Trifásica', desc: '<b>Clase:</b> Trinidad Industrial.<br><b>Función:</b> Voltaje de línea entre fases de un sistema polifásico (√3 · VP).', introKey: '4-1' }
        };

        const questsDB = {
            '1-1': {
                title: 'Misión 1.1: Onda Senoidal y Fasores', chap: 'chap-1', next: '1-2', introChar: 'V',
                narrative: '¡Saludos, Ingeniero! Cuando un espira gira dentro de un campo magnético, genera una tensión senoidal en el tiempo. Para evitar resolver ecuaciones diferenciales complejas, transformamos esta onda variante en un <b>Fasor</b>: un vector estático que captura la amplitud máxima y el ángulo de desfase.',
                eq1: 'v(t) = V_m \\cdot \\cos(\\omega t + \\phi)', eq2: '\\mathbf{V} = V_{rms} \\angle \\phi = \\frac{V_m}{\\sqrt{2}} e^{j\\phi}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Vector Fasor V:</b> Magnitud $V_m$ y ángulo $\\phi$ en el plano complejo.</div><div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Proyección Flotante:</b> Trazo horizontal que genera la forma de onda.</div><div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Señal Senoidal:</b> Evolución temporal $v(t)$ medida en osciloscopio.</div>',
                controls: `<div class="control-group"><label>Amplitud Vm (V): <span id="val-vm">100</span></label><input type="range" id="input-vm" min="30" max="150" value="100" oninput="drawSimSenoidal()"></div><div class="control-group"><label>Desfase ϕ (°): <span id="val-phi">45</span>°</label><input type="range" id="input-phi" min="-180" max="180" value="45" oninput="drawSimSenoidal()"></div>`,
                drawSchema: (ctx) => drawSchemaSenoidal(ctx), drawSim: () => drawSimSenoidal()
            },
            '1-2': {
                title: 'Misión 1.2: Impedancia y Resonancia RLC', chap: 'chap-1', next: '1-3', introChar: 'Z',
                narrative: 'Al conectar una resistencia, una bobina y un condensador en serie, la oposición total al flujo de corriente pasa a llamarse <b>Impedancia ($Z$)</b>. A una frecuencia específica llamada <b>Resonancia ($f_0$)</b>, las reactancias inductiva y capacitiva se anulan exactamente, haciendo que la impedancia sea puramente resistiva y mínima.',
                eq1: '\\mathbf{Z} = R + j\\left(\\omega L - \\frac{1}{\\omega C}\\right)', eq2: 'f_0 = \\frac{1}{2\\pi \\sqrt{LC}} \\quad \\implies \\quad |\\mathbf{Z}_{min}| = R',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Curva de Impedancia |Z|:</b> Muestra el mínimo de oposición en frecuencia de resonancia.</div><div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Punto f0:</b> Frecuencia exacta donde $X_L = X_C$.</div>',
                controls: `<div class="control-group"><label>Inductancia L (mH): <span id="val-l">10</span></label><input type="range" id="input-l" min="1" max="50" value="10" oninput="drawSimRLC()"></div><div class="control-group"><label>Capacitancia C (µF): <span id="val-c">10</span></label><input type="range" id="input-c" min="1" max="50" value="10" oninput="drawSimRLC()"></div>`,
                drawSchema: (ctx) => drawSchemaRLC(ctx), drawSim: () => drawSimRLC()
            },
            '1-3': {
                title: 'Misión 1.3: Leyes de Kirchhoff en AC', chap: 'chap-1', next: '2-1', introChar: 'K',
                narrative: 'Las leyes de nodos (LCK) y mallas (LTK) se mantienen totalmente válidas en corriente alterna, con la diferencia de que ahora operamos con números complejos. Agrupamos las ecuaciones en matrices de impedancias o admitancias para resolver redes complejas de múltiples fuentes.',
                eq1: '\\sum \\mathbf{V}_k = 0, \\quad \\sum \\mathbf{I}_k = 0', eq2: '[\\mathbf{Y}] \\cdot [\\mathbf{V}_{nodos}] = [\\mathbf{I}_{fuentes}]',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Fasores de Corriente:</b> Amplitud y fase de corrientes en las mallas del circuito.</div>',
                controls: `<div class="control-group"><label>Impedancia Z2 (Ω): <span id="val-z2">20</span></label><input type="range" id="input-z2" min="5" max="50" value="20" oninput="drawSimKCL()"></div>`,
                drawSchema: (ctx) => drawSchemaKCL(ctx), drawSim: () => drawSimKCL()
            },
            '2-1': {
                title: 'Misión 2.1: Potencia Instantánea y RMS', chap: 'chap-2', next: '2-2', introChar: 'P',
                narrative: 'La potencia instantánea en un circuito AC oscila al doble de la frecuencia de la red. Para medir el efecto calórico real de la corriente alterna, utilizamos los valores **RMS** (Raíz Cuadrada Media), que equivalen a la tensión DC que produciría la misma potencia promedio.',
                eq1: 'p(t) = v(t) \\cdot i(t) = V_{rms} I_{rms} \\cos\\theta + V_{rms} I_{rms} \\cos(2\\omega t - \\theta)', eq2: 'P_{prom} = V_{rms} \\cdot I_{rms} \\cdot \\cos(\\theta_v - \\theta_i)',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Línea de Potencia Promedio P:</b> Nivel medio continuo de trabajo útil.</div><div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Onda p(t):</b> Potencia instantánea oscilante.</div>',
                controls: `<div class="control-group"><label>Voltaje Vrms: <span id="val-vrms">120</span></label><input type="range" id="input-vrms" min="60" max="240" value="120" oninput="drawSimPinst()"></div>`,
                drawSchema: (ctx) => drawSchemaPinst(ctx), drawSim: () => drawSimPinst()
            },
            '2-2': {
                title: 'Misión 2.2: Triángulo de Potencia Compleja', chap: 'chap-2', next: '2-3', introChar: 'S',
                narrative: 'Las cargas industriales absorben dos clases de energía: la **Potencia Activa ($P$)** que realiza trabajo útil (Watts), y la **Potencia Reactiva ($Q$)** consumida por inductores para crear campos magnéticos (VAR). Unidas forman la **Potencia Aparente ($S$)** en el espacio vectorial.',
                eq1: '\\mathbf{S} = P + jQ = \\mathbf{V}_{rms} \\cdot \\mathbf{I}_{rms}^*', eq2: '|\\mathbf{S}| = \\sqrt{P^2 + Q^2} \\quad [\\text{VA}]',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>P (Watts):</b> Trabajo activo real.</div><div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Q (VAR):</b> Potencia reactiva almacenada en inductores.</div><div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>S (VA):</b> Potencia aparente entregada por la red.</div>',
                controls: `<div class="control-group"><label>Reactiva QL (VAR): <span id="val-ql">250</span></label><input type="range" id="input-ql" min="50" max="450" value="250" oninput="drawSimTriang()"></div>`,
                drawSchema: (ctx) => drawSchemaTriang(ctx), drawSim: () => drawSimTriang()
            },
            '2-3': {
                title: 'Misión 2.3: Corrección del Factor de Potencia', chap: 'chap-2', next: '3-1', introChar: 'FP',
                narrative: 'Un Factor de Potencia ($FP = \\cos\\theta$) bajo significa que la fábrica sobrecarga las líneas con energía reactiva que no realiza trabajo. Para corregir el FP, se conectan bancos de condensadores en paralelo que inyectan potencia reactiva capacitiva ($Q_C$), reduciendo la corriente total demandada.',
                eq1: 'FP = \\frac{P}{S} = \\cos(\\theta)', eq2: 'Q_C = P \\cdot (\\tan\\theta_1 - \\tan\\theta_2) \\implies C = \\frac{Q_C}{\\omega V_{rms}^2}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Triángulo Reducido:</b> Disminución del vector aparente $S$ tras inyectar $Q_C$.</div>',
                controls: `<div class="control-group"><label>Inyección Capacitiva QC: <span id="val-qc">150</span></label><input type="range" id="input-qc" min="0" max="350" value="150" oninput="drawSimFP()"></div>`,
                drawSchema: (ctx) => drawSchemaFP(ctx), drawSim: () => drawSimFP()
            },
            '3-1': {
                title: 'Misión 3.1: Inductancia Mutua y Acoplo', chap: 'chap-3', next: '3-2', introChar: 'M',
                narrative: 'Cuando dos bobinas están físicamente cercanas, el flujo magnético alterno generado por la corriente de la primera atraviesa las espiras de la segunda, induciendo una tensión por **Inductancia Mutua ($M$)**. La fracción de flujo compartido se mide con el coeficiente de acoplamiento $k$.',
                eq1: 'v_2(t) = M \\frac{di_1(t)}{dt}', eq2: 'M = k \\sqrt{L_1 L_2} \\quad (0 \\le k \\le 1)',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Líneas de Flujo Φ12:</b> Densidad de campo magnético acoplado entre bobinas.</div>',
                controls: `<div class="control-group"><label>Coeficiente k: <span id="val-k">0.7</span></label><input type="range" id="input-k" min="0.1" max="1" step="0.05" value="0.7" oninput="drawSimMutua()"></div>`,
                drawSchema: (ctx) => drawSchemaMutua(ctx), drawSim: () => drawSimMutua()
            },
            '3-2': {
                title: 'Misión 3.2: Regla de Puntos en Bobinas', chap: 'chap-3', next: '3-3', introChar: 'M',
                narrative: 'Para determinar la polaridad relativa de las tensiones inducidas mutuamente sin dibujar la geometría tridimensional de los devanados, se utiliza la **Regla de los Puntos**: si una corriente entra por el terminal con punto de una bobina, produce una tensión inducida positiva en el terminal con punto de la otra.',
                eq1: '\\mathbf{V}_1 = j\\omega L_1 \\mathbf{I}_1 + j\\omega M \\mathbf{I}_2', eq2: '\\mathbf{V}_2 = j\\omega L_2 \\mathbf{I}_2 + j\\omega M \\mathbf{I}_1',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Puntos Polares:</b> Referencia de concordancia de fase entre tensiones inducidas.</div>',
                controls: `<div class="control-group"><label>Dirección I2: <span id="val-dir">Mismo Punto</span></label><input type="range" id="input-dir" min="0" max="1" step="1" value="1" oninput="drawSimDots()"></div>`,
                drawSchema: (ctx) => drawSchemaDots(ctx), drawSim: () => drawSimDots()
            },
            '3-3': {
                title: 'Misión 3.3: El Transformador Ideal', chap: 'chap-3', next: '4-1', introChar: 'M',
                narrative: 'Un transformador ideal consta de dos bobinas devanadas sobre un núcleo magnético de permeabilidad infinita sin pérdidas. Permite elevaciones o reducciones de voltaje basadas estrictamente en la razón de vueltas $a = N_1 / N_2$, aislando al mismo tiempo etapas de potencia.',
                eq1: 'a = \\frac{N_1}{N_2} = \\frac{\\mathbf{V}_1}{\\mathbf{V}_2} = \\frac{\\mathbf{I}_2}{\\mathbf{I}_1}', eq2: '\\mathbf{Z}_{in} = a^2 \\mathbf{Z}_L',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Onda Secundario V2:</b> Transformada en amplitud según la razón $a$.</div>',
                controls: `<div class="control-group"><label>Razón a (N1/N2): <span id="val-a">2.0</span></label><input type="range" id="input-a" min="0.5" max="4" step="0.5" value="2" oninput="drawSimTransfo()"></div>`,
                drawSchema: (ctx) => drawSchemaTransfo(ctx), drawSim: () => drawSimTransfo()
            },
            '4-1': {
                title: 'Misión 4.1: Generador Trifásico y Secuencias', chap: 'chap-4', next: '4-2', introChar: 'VL',
                narrative: 'Un generador trifásico posee tres devanados separados simétricamente $120^\circ$ en el estator. Al girar el rotor, produce tres voltajes iguales en amplitud pero desfasados un tercio de periodo. La **Secuencia de Fase** (positiva ABC o negativa ACB) determina la dirección de giro de motores eléctricos.',
                eq1: '\\mathbf{V}_{aN} = V_p \\angle 0^\\circ, \\quad \\mathbf{V}_{bN} = V_p \\angle -120^\\circ, \\quad \\mathbf{V}_{cN} = V_p \\angle 120^\\circ', eq2: 'V_{linea} = \\sqrt{3} \\cdot V_p \\approx 1.732 \\cdot V_p',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Fase A (0°)</b> | <div class="legend-color" style="background:#00f3ff;"></div> <b>Fase B (-120°)</b> | <div class="legend-color" style="background:#00ff66;"></div> <b>Fase C (120°)</b></div>',
                controls: `<div class="control-group"><label>Voltaje Fase Vp (V): <span id="val-vp">120</span></label><input type="range" id="input-vp" min="60" max="240" value="120" oninput="drawSimTriGen()"></div>`,
                drawSchema: (ctx) => drawSchemaTriGen(ctx), drawSim: () => drawSimTriGen()
            },
            '4-2': {
                title: 'Misión 4.2: Conexión Estrella Y / Delta Δ', chap: 'chap-4', next: '4-3', introChar: 'VL',
                narrative: 'Las cargas y generadores trifásicos se interconectan en dos topologías principales: **Estrella ($Y$)** y **Delta ($\Delta$)**. En la Estrella, el voltaje entre dos líneas activas es $\sqrt{3}$ veces mayor que el voltaje de fase. En la Delta, el voltaje de línea es idéntico al de fase pero la corriente de línea escala por $\sqrt{3}$.',
                eq1: 'Y: V_L = \\sqrt{3} V_p \\angle 30^\\circ, \\quad \\Delta: I_L = \\sqrt{3} I_p \\angle -30^\\circ', eq2: 'P_{total} = \\sqrt{3} \\cdot V_L \\cdot I_L \\cdot \\cos(\\theta)',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Relación de Fase/Línea:</b> Escalado geométrico de vectores por el factor $\sqrt{3}$.</div>',
                controls: `<div class="control-group"><label>Corriente Ip (A): <span id="val-ip">10</span></label><input type="range" id="input-ip" min="2" max="25" value="10" oninput="drawSimYDelta()"></div>`,
                drawSchema: (ctx) => drawSchemaYDelta(ctx), drawSim: () => drawSimYDelta()
            },
            '4-3': {
                title: 'Misión 4.3: Cargas Trifásicas Desequilibradas', chap: 'chap-4', next: null, introChar: 'VL',
                narrative: 'En instalaciones reales, las impedancias conectadas a cada fase no siempre son idénticas ($\mathbf{Z}_A \\neq \\mathbf{Z}_B \\neq \\mathbf{Z}_C$). Este desequilibrio altera las corrientes de línea y produce una **Corriente de Neutro ($I_N$)** no nula que regresa al centro de la Estrella.',
                eq1: '\\mathbf{I}_N = \\mathbf{I}_A + \\mathbf{I}_B + \\mathbf{I}_C', eq2: 'P_{total} = P_A + P_B + P_C',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Vector IN:</b> Corriente resultante que circula por el cable de neutro debido al desequilibrio.</div>',
                controls: `<div class="control-group"><label>Desequilibrio Fase A (%): <span id="val-des">60</span>%</label><input type="range" id="input-des" min="0" max="100" value="60" oninput="drawSimDeseb()"></div>`,
                drawSchema: (ctx) => drawSchemaDeseb(ctx), drawSim: () => drawSimDeseb()
            }
        };

        /* --- ESTADO Y LÓGICA GENERAL --- */
        const gameState = {
            xp: 0, level: 1,
            unlockedChaps: ['chap-1'],
            completedQuests: [],
            unlockedQuests: ['1-1'],
            unlockedChars: ['V', 'I', 'w']
        };

        let activeQuest = null;
        let typewriterTimeout = null;
        let currentFullText = "";
        let electronAnimProgress = 0;

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
                    circle.className = "chapter-circle unlocked";
                    tag.innerText = "DESBLOQUEADO";
                } else {
                    el.classList.add('locked');
                    circle.className = "chapter-circle locked";
                    tag.innerText = "BLOQUEADO 🔒";
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

        /* --- DIBUJO DE CABLES CURVADOS Y ANIMACIÓN DEL ELECTRÓN --- */
        function updateCablesAndElectron() {
            const wrapper = document.getElementById('map-wrapper');
            const svg = document.getElementById('cable-svg');
            const rectW = wrapper.getBoundingClientRect();

            svg.setAttribute('width', rectW.width);
            svg.setAttribute('height', rectW.height);

            const connections = [
                { from: 'chap-1', to: 'chap-2', pathBg: 'path-1-2', pathActive: 'path-1-2-active' },
                { from: 'chap-2', to: 'chap-3', pathBg: 'path-2-3', pathActive: 'path-2-3-active' },
                { from: 'chap-3', to: 'chap-4', pathBg: 'path-3-4', pathActive: 'path-3-4-active' }
            ];

            connections.forEach(conn => {
                const elA = document.getElementById(conn.from).querySelector('.chapter-circle');
                const elB = document.getElementById(conn.to).querySelector('.chapter-circle');

                const rA = elA.getBoundingClientRect();
                const rB = elB.getBoundingClientRect();

                const x1 = rA.left + rA.width / 2 - rectW.left;
                const y1 = rA.top + rA.height / 2 - rectW.top;
                const x2 = rB.left + rB.width / 2 - rectW.left;
                const y2 = rB.top + rB.height / 2 - rectW.top;

                // Curva Bézier suave tipo 'S'
                const deltaY = y2 - y1;
                const d = `M ${x1} ${y1} C ${x1 + 140} ${y1 + deltaY * 0.5}, ${x2 - 140} ${y1 + deltaY * 0.5}, ${x2} ${y2}`;

                const pBg = document.getElementById(conn.pathBg);
                const pAct = document.getElementById(conn.pathActive);

                pBg.setAttribute('d', d);
                pAct.setAttribute('d', d);

                const isUnlocked = gameState.unlockedChaps.includes(conn.to);
                pAct.style.display = isUnlocked ? 'block' : 'none';
            });
        }

        function animateElectron() {
            electronAnimProgress += 0.006;
            if (electronAnimProgress > 1) electronAnimProgress = 0;

            // Encontrar la última conexión activa
            let activePathId = 'path-1-2-active';
            if (gameState.unlockedChaps.includes('chap-4')) activePathId = 'path-3-4-active';
            else if (gameState.unlockedChaps.includes('chap-3')) activePathId = 'path-2-3-active';

            const path = document.getElementById(activePathId);
            const electronGroup = document.getElementById('electron-group');

            if (path && path.getTotalLength) {
                const len = path.getTotalLength();
                const pt = path.getPointAtLength(electronAnimProgress * len);

                electronGroup.setAttribute('transform', `translate(${pt.x}, ${pt.y})`);
            }

            requestAnimationFrame(animateElectron);
        }

        /* --- MANEJO DE MISIONES Y NARRATIVA --- */
        function openQuest(key) {
            if (!gameState.unlockedQuests.includes(key)) return;
            activeQuest = key;
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

            switchStep('theory');
            document.getElementById('quest-modal').classList.add('active');

            currentFullText = q.narrative;
            startTypewriter('narrative-text-el', q.narrative);

            setTimeout(() => {
                const schemaCv = document.getElementById('schema-canvas');
                if (schemaCv && q.drawSchema) q.drawSchema(schemaCv.getContext('2d'));
            }, 50);
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
                            el.innerHTML += text.substring(i, closeIdx + 1);
                            i = closeIdx + 1;
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

        function switchStep(step) {
            document.getElementById('step-theory').classList.remove('active');
            document.getElementById('step-sim').classList.remove('active');

            if (step === 'theory') {
                document.getElementById('step-theory').classList.add('active');
            } else {
                document.getElementById('step-sim').classList.add('active');
                setTimeout(() => {
                    questsDB[activeQuest].drawSim();
                    renderMathInLegend();
                }, 50);
            }
        }

        function renderMathInLegend() {
            const leg = document.getElementById('sim-legend');
            if (window.renderMathInElement && leg) {
                renderMathInElement(leg, { delimiters: [{left: '$', right: '$', display: false}], throwOnError: false });
            }
        }

        function closeQuest() {
            if (typewriterTimeout) clearTimeout(typewriterTimeout);
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

        /* --- ESQUEMAS ORIGEN DE LAS VARIABLES --- */
        function drawSchemaSenoidal(ctx) {
            ctx.clearRect(0,0,650,150);
            ctx.strokeStyle='#ff0055'; ctx.lineWidth=2;
            ctx.fillStyle='#1e293b'; ctx.fillRect(50,30,60,90); ctx.fillRect(250,30,60,90);
            ctx.fillStyle='#ff0055'; ctx.font='16px Orbitron'; ctx.fillText('N', 75, 80);
            ctx.fillStyle='#00f3ff'; ctx.fillText('S', 275, 80);
            ctx.strokeStyle='#ffb700'; ctx.lineWidth=3; ctx.beginPath(); ctx.arc(180,75,30,0,Math.PI*2); ctx.stroke();
            ctx.fillStyle='#fff'; ctx.font='12px Rajdhani'; ctx.fillText('Rotación Generador (ω)', 135, 130);
            ctx.strokeStyle='rgba(0,243,255,0.4)'; ctx.lineWidth=1;
            for(let y=45; y<=105; y+=20) { ctx.beginPath(); ctx.moveTo(110,y); ctx.lineTo(250,y); ctx.stroke(); }
            ctx.strokeStyle='#00ff66'; ctx.lineWidth=2; ctx.beginPath(); ctx.moveTo(330,75); ctx.lineTo(400,75); ctx.stroke();
            ctx.fillStyle='#00ff66'; ctx.fillText('Genera v(t) = Vm·cos(ωt+ϕ)', 410, 80);
        }

        function drawSchemaRLC(ctx) {
            ctx.clearRect(0,0,650,150);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2; ctx.strokeRect(100,30,450,90);
            ctx.clearRect(120,20,40,20); ctx.clearRect(280,20,40,20); ctx.clearRect(440,20,40,20);
            ctx.fillStyle='#ffb700'; ctx.font='14px Orbitron';
            ctx.fillText('R (Resistencia)', 100, 20); ctx.fillText('L (Inductancia)', 260, 20); ctx.fillText('C (Capacitancia)', 420, 20);
            ctx.fillStyle='#00ff66'; ctx.fillText('Impedancia Total Z = R + j(XL - XC)', 200, 100);
        }

        function drawSchemaKCL(ctx) {
            ctx.clearRect(0,0,650,150);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.arc(325,75,8,0,Math.PI*2); ctx.fillStyle='#ff0055'; ctx.fill();
            ctx.fillStyle='#fff'; ctx.font='12px Orbitron'; ctx.fillText('Nodo Complejo ΣI = 0', 270, 45);
            ctx.beginPath(); ctx.moveTo(220,75); ctx.lineTo(317,75); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(333,75); ctx.lineTo(430,40); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(333,75); ctx.lineTo(430,110); ctx.stroke();
            ctx.fillStyle='#00f3ff'; ctx.fillText('I1∠ϕ1', 230, 65); ctx.fillText('I2∠ϕ2', 400, 30); ctx.fillText('I3∠ϕ3', 400, 125);
        }

        function drawSchemaPinst(ctx) {
            ctx.clearRect(0,0,650,150);
            ctx.fillStyle='#1e293b'; ctx.fillRect(100,40,120,70);
            ctx.fillStyle='#00f3ff'; ctx.font='12px Orbitron'; ctx.fillText('Fuente AC', 125, 80);
            ctx.strokeStyle='#00ff66'; ctx.lineWidth=2; ctx.strokeRect(400,40,120,70);
            ctx.fillStyle='#00ff66'; ctx.fillText('Carga ZL', 435, 80);
            ctx.beginPath(); ctx.moveTo(220,55); ctx.lineTo(400,55); ctx.moveTo(220,95); ctx.lineTo(400,95); ctx.stroke();
            ctx.fillStyle='#ffb700'; ctx.fillText('v(t) · i(t) = p(t)', 270, 45);
        }

        function drawSchemaTriang(ctx) {
            ctx.clearRect(0,0,650,150);
            ctx.strokeStyle='#ff0055'; ctx.lineWidth=2; ctx.strokeRect(150,30,100,90);
            ctx.fillStyle='#ff0055'; ctx.font='12px Orbitron'; ctx.fillText('Motor Inductivo', 155, 80);
            ctx.strokeStyle='#00f3ff'; ctx.strokeRect(350,30,100,90);
            ctx.fillStyle='#00f3ff'; ctx.fillText('Resistencia R', 355, 80);
            ctx.fillStyle='#ffb700'; ctx.fillText('Absorbe Q (VAR)', 150, 140);
            ctx.fillStyle='#00f3ff'; ctx.fillText('Absorbe P (Watts)', 350, 140);
        }

        function drawSchemaFP(ctx) {
            ctx.clearRect(0,0,650,150);
            ctx.strokeStyle='#00ff66'; ctx.lineWidth=2; ctx.strokeRect(80,35,120,80);
            ctx.fillStyle='#00ff66'; ctx.font='12px Orbitron'; ctx.fillText('Planta Industrial', 90, 80);
            ctx.strokeStyle='#ff0055'; ctx.strokeRect(300,35,120,80);
            ctx.fillStyle='#ff0055'; ctx.fillText('Banco Capacitores', 305, 80);
            ctx.fillStyle='#fff'; ctx.fillText('＋', 240, 80);
            ctx.fillStyle='#ffb700'; ctx.fillText('Inyecta -QC para elevar FP ≥ 0.95', 440, 80);
        }

        function drawSchemaMutua(ctx) {
            ctx.clearRect(0,0,650,150);
            ctx.strokeStyle='#ffb700'; ctx.lineWidth=3;
            ctx.beginPath(); ctx.arc(200,75,35,0,Math.PI*2); ctx.stroke();
            ctx.beginPath(); ctx.arc(450,75,35,0,Math.PI*2); ctx.stroke();
            ctx.fillStyle='#fff'; ctx.font='12px Orbitron'; ctx.fillText('Bobina L1', 170, 130); ctx.fillText('Bobina L2', 420, 130);
            ctx.strokeStyle='rgba(0,243,255,0.6)'; ctx.lineWidth=1.5; ctx.setLineDash([4,4]);
            ctx.beginPath(); ctx.moveTo(235,75); ctx.lineTo(415,75); ctx.stroke(); ctx.setLineDash([]);
            ctx.fillStyle='#00f3ff'; ctx.fillText('Inductancia Mutua (M)', 270, 65);
        }

        function drawSchemaDots(ctx) {
            ctx.clearRect(0,0,650,150);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.arc(220,75,30,0,Math.PI*2); ctx.stroke();
            ctx.beginPath(); ctx.arc(430,75,30,0,Math.PI*2); ctx.stroke();
            ctx.fillStyle='#ff0055'; ctx.beginPath(); ctx.arc(220,35,6,0,Math.PI*2); ctx.fill();
            ctx.beginPath(); ctx.arc(430,35,6,0,Math.PI*2); ctx.fill();
            ctx.fillStyle='#fff'; ctx.font='12px Orbitron'; ctx.fillText('Punto Entrada', 180, 20); ctx.fillText('Punto Salida', 390, 20);
        }

        function drawSchemaTransfo(ctx) {
            ctx.clearRect(0,0,650,150);
            ctx.fillStyle='#1e293b'; ctx.fillRect(220,25,210,100); ctx.clearRect(260,50,130,50);
            ctx.fillStyle='#ffb700'; ctx.font='12px Orbitron'; ctx.fillText('Primario N1', 130, 80);
            ctx.fillStyle='#00f3ff'; ctx.fillText('Secundario N2', 460, 80);
            ctx.fillStyle='#00ff66'; ctx.fillText('Núcleo Magnético a = N1/N2', 245, 142);
        }

        function drawSchemaTriGen(ctx) {
            ctx.clearRect(0,0,650,150);
            ctx.fillStyle='#fff'; ctx.font='12px Orbitron'; ctx.fillText('Rotor 120°', 280, 80);
            const colors = ['#ff0055', '#00f3ff', '#00ff66'], angles = [0, 120, 240];
            angles.forEach((ang, i) => {
                const rad = (ang * Math.PI) / 180, x = 325 + 50 * Math.cos(rad), y = 75 + 50 * Math.sin(rad);
                ctx.strokeStyle = colors[i]; ctx.lineWidth = 3;
                ctx.beginPath(); ctx.moveTo(325,75); ctx.lineTo(x,y); ctx.stroke();
            });
        }

        function drawSchemaYDelta(ctx) {
            ctx.clearRect(0,0,650,150);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.moveTo(150,40); ctx.lineTo(150,80); ctx.lineTo(110,110); ctx.moveTo(150,80); ctx.lineTo(190,110); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(480,40); ctx.lineTo(440,110); ctx.lineTo(520,110); ctx.closePath(); ctx.stroke();
            ctx.fillStyle='#fff'; ctx.font='12px Orbitron'; ctx.fillText('Topología Y', 125, 135); ctx.fillText('Topología Δ', 455, 135);
        }

        function drawSchemaDeseb(ctx) {
            ctx.clearRect(0,0,650,150);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.moveTo(100,75); ctx.lineTo(550,75); ctx.stroke();
            ctx.fillStyle='#ff0055'; ctx.font='12px Orbitron'; ctx.fillText('Fase A (ZA)', 150, 60);
            ctx.fillStyle='#00f3ff'; ctx.fillText('Fase B (ZB)', 300, 60);
            ctx.fillStyle='#00ff66'; ctx.fillText('Fase C (ZC)', 450, 60);
            ctx.fillStyle='#ffb700'; ctx.fillText('Neutro con Corriente de Retorno IN ≠ 0', 220, 115);
        }

        /* --- SIMULADORES INTERACTIVOS --- */
        function drawSimSenoidal() {
            const Vm = parseFloat(document.getElementById('input-vm').value);
            const phiDeg = parseFloat(document.getElementById('input-phi').value);
            document.getElementById('val-vm').innerText = Vm;
            document.getElementById('val-phi').innerText = phiDeg;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const cy = 115, cxPhasor = 120, phiRad = (phiDeg * Math.PI) / 180, r = Vm * 0.65;

            ctx.strokeStyle = 'rgba(255,255,255,0.08)'; ctx.lineWidth = 1;
            for(let x=0; x<cv.width; x+=30) { ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x,cv.height); ctx.stroke(); }
            for(let y=0; y<cv.height; y+=30) { ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(cv.width,y); ctx.stroke(); }

            ctx.strokeStyle = 'rgba(255,255,255,0.2)'; ctx.lineWidth = 1.5;
            ctx.beginPath(); ctx.moveTo(20, cy); ctx.lineTo(220, cy); ctx.moveTo(cxPhasor, 15); ctx.lineTo(cxPhasor, 215); ctx.stroke();

            ctx.strokeStyle = 'rgba(0,243,255,0.15)'; ctx.beginPath(); ctx.arc(cxPhasor, cy, r, 0, Math.PI * 2); ctx.stroke();

            const fx = cxPhasor + r * Math.cos(-phiRad), fy = cy + r * Math.sin(-phiRad);
            ctx.strokeStyle = '#ff0055'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(cxPhasor, cy); ctx.lineTo(fx, fy); ctx.stroke();
            ctx.fillStyle = '#ff0055'; ctx.beginPath(); ctx.arc(fx, fy, 4, 0, Math.PI*2); ctx.fill();

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 1.5; ctx.setLineDash([4, 4]);
            ctx.beginPath(); ctx.moveTo(fx, fy); ctx.lineTo(260, fy); ctx.stroke(); ctx.setLineDash([]);

            ctx.strokeStyle = 'rgba(255,255,255,0.2)'; ctx.beginPath(); ctx.moveTo(260, cy); ctx.lineTo(630, cy); ctx.stroke();

            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 2.5; ctx.beginPath();
            for (let x = 260; x <= 630; x++) {
                const t = (x - 260) * 0.02;
                const y = cy - r * Math.sin(t + phiRad);
                if (x === 260) ctx.moveTo(x, y); else ctx.lineTo(x, y);
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
                const y = 200 - Math.min(Z * 1.2, 180);
                if (x === 40) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();

            const xRes = 40 + (f0 / 5);
            if (xRes >= 40 && xRes <= 610) {
                ctx.strokeStyle = '#ffb700'; ctx.setLineDash([4, 4]);
                ctx.beginPath(); ctx.moveTo(xRes, 10); ctx.lineTo(xRes, 210); ctx.stroke(); ctx.setLineDash([]);
                ctx.fillStyle = '#ffb700'; ctx.font = '12px Orbitron';
                ctx.fillText(`f0 = ${Math.round(f0)} Hz`, Math.min(xRes + 8, 500), 30);
            }
        }

        function drawSimKCL() {
            const Z2 = parseFloat(document.getElementById('input-z2').value);
            document.getElementById('val-z2').innerText = Z2;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            ctx.fillStyle = '#00f3ff'; ctx.font = '14px Orbitron';
            ctx.fillText(`Matriz de Admitancia [Y]:`, 50, 40);
            ctx.fillStyle = '#fff'; ctx.font = '13px Rajdhani';
            ctx.fillText(`[  (1/10 + 1/${Z2})   -1/${Z2}  ]  ·  [ V1 ]  =  [ 10∠0° ]`, 50, 75);
            ctx.fillText(`[     -1/${Z2}          1/${Z2}     ]     [ V2 ]     [   0   ]`, 50, 105);

            const V1 = (120 * (1 + 10/Z2)).toFixed(1);
            ctx.fillStyle = '#00ff66'; ctx.font = '14px Orbitron';
            ctx.fillText(`Solución Nodal Resultante: V1 = ${V1} V ∠ 0°`, 50, 160);
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
                const y = 200 - p * 0.4;
                if(x===50) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 2; ctx.setLineDash([5,5]);
            const yP = 200 - Pprom * 0.4;
            ctx.beginPath(); ctx.moveTo(50, yP); ctx.lineTo(600, yP); ctx.stroke(); ctx.setLineDash([]);

            ctx.fillStyle = '#ffb700'; ctx.font = '13px Orbitron';
            ctx.fillText(`P_prom = ${Pprom} W`, 480, yP - 10);
        }

        function drawSimTriang() {
            const QL = parseFloat(document.getElementById('input-ql').value);
            document.getElementById('val-ql').innerText = QL;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const P = 300, ox = 80, oy = 190, sc = 0.4;
            const S = Math.sqrt(P*P + QL*QL);

            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P*sc, oy); ctx.stroke();
            ctx.strokeStyle = '#ff0055';
            ctx.beginPath(); ctx.moveTo(ox + P*sc, oy); ctx.lineTo(ox + P*sc, oy - QL*sc); ctx.stroke();
            ctx.strokeStyle = '#00ff66';
            ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P*sc, oy - QL*sc); ctx.stroke();

            ctx.fillStyle = '#fff'; ctx.font = '13px Orbitron';
            ctx.fillText(`Potencia Aparente |S| = ${Math.round(S)} VA`, 320, 80);
            ctx.fillText(`Factor de Potencia = ${(P/S).toFixed(2)}`, 320, 110);
        }

        function drawSimFP() {
            const QC = parseFloat(document.getElementById('input-qc').value);
            document.getElementById('val-qc').innerText = QC;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const P = 300, QL = 300, Qnet = Math.max(0, QL - QC);
            const S = Math.sqrt(P*P + Qnet*Qnet);
            const FP = (P / S).toFixed(2);

            const ox = 80, oy = 190, sc = 0.4;

            ctx.strokeStyle = 'rgba(255,255,255,0.2)'; ctx.lineWidth = 1;
            ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P*sc, oy - QL*sc); ctx.stroke();

            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P*sc, oy); ctx.lineTo(ox + P*sc, oy - Qnet*sc); ctx.closePath(); ctx.stroke();

            ctx.fillStyle = '#00ff66'; ctx.font = '14px Orbitron';
            ctx.fillText(`FP Resultante = ${FP}`, 320, 80);
            ctx.fillStyle = FP >= 0.95 ? '#00ff66' : '#ffb700';
            ctx.fillText(FP >= 0.95 ? '✔ FP EFICIENTE (SIN PENALIZACIÓN)' : '⚠️ FP BAJO (REQUIERE MÁS QC)', 320, 115);
        }

        function drawSimMutua() {
            const k = parseFloat(document.getElementById('input-k').value);
            document.getElementById('val-k').innerText = k;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 4;
            ctx.beginPath(); ctx.arc(180, 115, 45, 0, Math.PI*2); ctx.stroke();
            ctx.strokeStyle = '#00f3ff';
            ctx.beginPath(); ctx.arc(470, 115, 45, 0, Math.PI*2); ctx.stroke();

            ctx.strokeStyle = `rgba(0, 255, 102, ${k})`; ctx.lineWidth = k * 5;
            for(let r = 20; r <= 80; r += 20) {
                ctx.beginPath(); ctx.ellipse(325, 115, 145, r, 0, 0, Math.PI*2); ctx.stroke();
            }
        }

        function drawSimDots() {
            const dir = parseInt(document.getElementById('input-dir').value);
            document.getElementById('val-dir').innerText = dir === 1 ? 'Mismo Punto' : 'Punto Opuesto';

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 3;
            ctx.strokeRect(180, 60, 80, 110); ctx.strokeRect(390, 60, 80, 110);

            ctx.fillStyle = '#ff0055';
            ctx.beginPath(); ctx.arc(220, 45, 7, 0, Math.PI*2); ctx.fill();
            ctx.beginPath(); ctx.arc(430, dir === 1 ? 45 : 185, 7, 0, Math.PI*2); ctx.fill();

            ctx.fillStyle = '#fff'; ctx.font = '13px Orbitron';
            ctx.fillText(dir === 1 ? 'Voltajes Inducidos en FASE (+M)' : 'Voltajes Inducidos en OPOSICIÓN (-M)', 180, 215);
        }

        function drawSimTransfo() {
            const a = parseFloat(document.getElementById('input-a').value);
            document.getElementById('val-a').innerText = a;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const V1 = 120, V2 = (V1 / a).toFixed(1);

            ctx.fillStyle = '#1e293b'; ctx.fillRect(220, 40, 210, 130); ctx.clearRect(260, 70, 130, 70);

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 2; ctx.beginPath();
            for(let x=30; x<200; x++) {
                const y = 105 - 30 * Math.sin((x-30)*0.05);
                if(x===30) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();

            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 2; ctx.beginPath();
            for(let x=450; x<620; x++) {
                const y = 105 - (30/a) * Math.sin((x-450)*0.05);
                if(x===450) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();

            ctx.fillStyle = '#fff'; ctx.font = '13px Orbitron';
            ctx.fillText(`V1 = ${V1} V`, 80, 160); ctx.fillText(`V2 = ${V2} V`, 500, 160);
        }

        function drawSimTriGen() {
            const Vp = parseFloat(document.getElementById('input-vp').value);
            document.getElementById('val-vp').innerText = Vp;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const cx = 325, cy = 115, r = Vp * 0.65;
            const angles = [0, -120, 120], colors = ['#ff0055', '#00f3ff', '#00ff66'];

            angles.forEach((ang, i) => {
                const rad = (ang * Math.PI) / 180, x = cx + r * Math.cos(rad), y = cy + r * Math.sin(rad);
                ctx.strokeStyle = colors[i]; ctx.lineWidth = 3;
                ctx.beginPath(); ctx.moveTo(cx, cy); ctx.lineTo(x, y); ctx.stroke();
            });

            const VL = (Math.sqrt(3) * Vp).toFixed(1);
            ctx.fillStyle = '#fff'; ctx.font = '13px Orbitron';
            ctx.fillText(`Tensión de Línea VL = √3 · ${Vp} = ${VL} V`, 30, 30);
        }

        function drawSimYDelta() {
            const Ip = parseFloat(document.getElementById('input-ip').value);
            document.getElementById('val-ip').innerText = Ip;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const IL = (Math.sqrt(3) * Ip).toFixed(1);

            ctx.fillStyle = '#00f3ff'; ctx.font = '14px Orbitron';
            ctx.fillText(`Corriente de Fase Ip = ${Ip} A`, 100, 80);
            ctx.fillStyle = '#00ff66';
            ctx.fillText(`Corriente de Línea En Delta IL = √3 · Ip = ${IL} A`, 100, 130);
        }

        function drawSimDeseb() {
            const des = parseFloat(document.getElementById('input-des').value);
            document.getElementById('val-des').innerText = des;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const IN = (des * 0.12).toFixed(2);

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(100, 115); ctx.lineTo(100 + des * 3, 115); ctx.stroke();

            ctx.fillStyle = '#ffb700'; ctx.font = '14px Orbitron';
            ctx.fillText(`Corriente de Retorno por Neutro IN = ${IN} A`, 100, 80);
        }
    </script>
</body>
</html>