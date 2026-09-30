<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Análisis de Circuitos AC - Ruta RPG & Filtros</title>
    <!-- KaTeX CDN -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.css">
    <script src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/contrib/auto-render.min.js"></script>
    <!-- Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700;900&family=Rajdhani:wght@500;600;700&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --bg-dark: #040711;
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
            background: rgba(4, 7, 17, 0.95);
            border-bottom: 2px solid var(--cyan);
            box-shadow: 0 0 20px rgba(0, 243, 255, 0.2);
            padding: 12px 25px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            position: sticky;
            top: 0;
            z-index: 200;
            backdrop-filter: blur(10px);
        }

        .hud-title {
            font-family: 'Orbitron', sans-serif;
            font-size: 1.15rem;
            font-weight: 800;
            color: #fff;
            text-shadow: 0 0 10px var(--cyan);
        }

        .hud-controls { display: flex; gap: 12px; align-items: center; }

        .btn-hud {
            font-family: 'Orbitron', sans-serif;
            background: rgba(157, 0, 255, 0.2);
            border: 1px solid var(--purple);
            color: #fff;
            padding: 6px 12px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 0.8rem;
            transition: all 0.2s;
            box-shadow: 0 0 10px rgba(157, 0, 255, 0.3);
        }
        .btn-hud:hover { background: var(--purple); box-shadow: 0 0 18px var(--purple); }

        .btn-formulas {
            background: rgba(0, 243, 255, 0.15);
            border-color: var(--cyan);
            box-shadow: 0 0 10px rgba(0, 243, 255, 0.3);
        }
        .btn-formulas:hover { background: var(--cyan); color: #000; box-shadow: 0 0 20px var(--cyan); }

        .hud-stats { display: flex; gap: 15px; align-items: center; }
        .stat-box { display: flex; flex-direction: column; align-items: flex-end; }
        .stat-label { font-size: 0.65rem; color: var(--text-muted); text-transform: uppercase; }
        .stat-value { font-family: 'Orbitron', sans-serif; font-size: 1rem; color: var(--cyan); font-weight: 700; }
        
        .xp-container {
            width: 100px; height: 7px;
            background: rgba(255,255,255,0.1);
            border: 1px solid var(--cyan);
            border-radius: 4px; overflow: hidden; margin-top: 3px;
        }
        .xp-bar { height: 100%; width: 0%; background: linear-gradient(90deg, var(--cyan), var(--green)); transition: width 0.4s; }

        /* --- CONTENEDOR DE ZOOM Y MUNDOS --- */
        #viewport-stage {
            position: relative;
            width: 100%;
            flex: 1;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
            padding: 40px 0;
        }

        #world-container {
            position: relative;
            width: 900px;
            transition: transform 0.8s cubic-bezier(0.25, 1, 0.5, 1);
            transform-origin: center center;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        /* CAPAS DE CABLES DE COBRE SVG */
        #cable-svg {
            position: absolute;
            top: 0; left: 0;
            width: 100%; height: 100%;
            pointer-events: none;
            z-index: 1;
        }

        .copper-base { fill: none; stroke: var(--copper-dark); stroke-width: 14; stroke-linecap: round; }
        .copper-core { fill: none; stroke: url(#copper-grad); stroke-width: 8; stroke-linecap: round; filter: drop-shadow(0 0 6px rgba(184, 115, 51, 0.6)); }
        .copper-texture { fill: none; stroke: var(--copper-light); stroke-width: 2; stroke-dasharray: 8, 5; opacity: 0.7; }

        .electron-core { fill: #00f3ff; filter: drop-shadow(0 0 8px #00f3ff) drop-shadow(0 0 16px #00f3ff); }
        .electron-text { font-family: 'Orbitron', sans-serif; font-size: 14px; font-weight: 900; fill: #040711; text-anchor: middle; dominant-baseline: central; }

        /* TERMINALES DE EXTREMO */
        .terminal-node {
            position: relative;
            z-index: 2;
            width: 65px; height: 65px;
            border-radius: 50%; background: #0b1220;
            border: 2px solid var(--copper-light);
            display: flex; flex-direction: column;
            justify-content: center; align-items: center;
            box-shadow: 0 0 20px rgba(184, 115, 51, 0.4);
            margin: 10px 0;
            transition: opacity 0.5s;
        }
        .terminal-label { font-family: 'Orbitron', sans-serif; font-size: 0.6rem; color: var(--copper-light); margin-top: 3px; }

        /* MUNDOS / CAPÍTULOS */
        .world-stage {
            position: relative;
            z-index: 2;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 50px 0;
            width: 100%;
            transition: all 0.5s ease;
        }

        .world-circle {
            width: 150px; height: 150px;
            border-radius: 50%;
            position: relative;
            overflow: hidden;
            display: flex; flex-direction: column;
            justify-content: center; align-items: center;
            cursor: pointer;
            border: 3px solid var(--panel-border);
            box-shadow: 0 0 25px rgba(0,0,0,0.8);
            transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            background: #000;
        }

        .world-circle.unlocked { border-color: var(--cyan); box-shadow: 0 0 30px rgba(0, 243, 255, 0.3); }
        .world-circle.completed { border-color: var(--green); box-shadow: 0 0 30px rgba(0, 255, 102, 0.3); }
        .world-circle.locked { opacity: 0.45; filter: grayscale(1); cursor: not-allowed; border-color: var(--text-muted); }

        .world-circle:hover:not(.locked) { transform: scale(1.08); box-shadow: 0 0 40px var(--cyan); }

        .world-cover-img {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background-size: cover; background-position: center;
            filter: blur(2.5px) brightness(0.45); transition: all 0.3s;
        }
        .world-circle:hover .world-cover-img { filter: blur(0px) brightness(0.65); transform: scale(1.1); }

        .world-overlay-content { position: relative; z-index: 3; text-align: center; padding: 10px; }
        .world-num { font-family: 'Orbitron', sans-serif; font-size: 0.75rem; color: var(--yellow); }
        .world-title { font-family: 'Orbitron', sans-serif; font-size: 0.9rem; font-weight: 800; color: #fff; margin: 3px 0; }
        .world-badge { font-family: 'Orbitron', sans-serif; font-size: 0.6rem; padding: 2px 6px; border-radius: 8px; background: rgba(0,0,0,0.8); border: 1px solid var(--cyan); color: var(--cyan); }

        /* NIVELES HORIZONTALES DENTRO DEL MUNDO ZOOM */
        .horizontal-levels-container {
            display: none;
            flex-direction: row;
            justify-content: center;
            align-items: center;
            gap: 60px;
            margin-top: 35px;
            width: 100%;
            padding: 20px;
            animation: fadeIn 0.5s ease;
        }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

        .world-stage.active-zoom .horizontal-levels-container { display: flex; }

        .sublevel-card {
            background: rgba(11, 18, 32, 0.95);
            border: 2px solid var(--panel-border);
            border-radius: 12px;
            padding: 16px;
            width: 170px;
            display: flex; flex-direction: column; align-items: center; gap: 8px;
            cursor: pointer; transition: all 0.3s;
            box-shadow: 0 10px 25px rgba(0,0,0,0.8);
            position: relative; z-index: 5;
        }
        .sublevel-card:hover:not(.locked) { transform: translateY(-6px); border-color: var(--cyan); box-shadow: 0 0 20px rgba(0, 243, 255, 0.3); }
        .sublevel-card.completed { border-color: var(--green); background: rgba(0, 255, 102, 0.08); }
        .sublevel-card.locked { opacity: 0.4; cursor: not-allowed; filter: grayscale(1); }

        .sublevel-icon { font-size: 1.8rem; margin-bottom: 2px; }
        .sublevel-name { font-family: 'Orbitron', sans-serif; font-size: 0.75rem; text-align: center; color: #fff; }

        /* BOTÓN PARA SALIR DEL ZOOM DE MUNDO */
        .btn-exit-zoom {
            position: fixed;
            bottom: 30px;
            right: 30px;
            z-index: 300;
            font-family: 'Orbitron', sans-serif;
            background: rgba(255, 0, 85, 0.2);
            border: 2px solid var(--red);
            color: #fff;
            padding: 12px 22px;
            border-radius: 30px;
            cursor: pointer;
            font-size: 0.85rem;
            display: none;
            box-shadow: 0 0 20px rgba(255, 0, 85, 0.4);
            transition: all 0.2s;
        }
        .btn-exit-zoom:hover { background: var(--red); box-shadow: 0 0 30px var(--red); }

        /* MODALES */
        .modal-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(4, 6, 12, 0.92); backdrop-filter: blur(8px);
            z-index: 1000; display: flex; justify-content: center; align-items: center;
            opacity: 0; pointer-events: none; transition: opacity 0.25s ease;
        }
        .modal-overlay.active { opacity: 1; pointer-events: all; }

        .modal-card {
            background: var(--panel-bg); border: 2px solid var(--cyan);
            border-radius: 12px; width: 92%; max-width: 920px; max-height: 92vh;
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

        .modal-body { padding: 22px; overflow-y: auto; display: flex; flex-direction: column; gap: 16px; }

        /* BARRA DE PASOS EN SUBTEMA */
        .subtopic-progress-bar {
            display: flex; gap: 8px; width: 100%; background: rgba(0,0,0,0.4);
            padding: 10px 15px; border-radius: 8px; border: 1px solid rgba(255,255,255,0.08);
            align-items: center; justify-content: space-between;
        }
        .step-pill { flex: 1; height: 8px; background: rgba(255,255,255,0.1); border-radius: 4px; transition: all 0.3s; }
        .step-pill.active { background: var(--cyan); box-shadow: 0 0 8px var(--cyan); }
        .step-pill.completed { background: var(--green); }
        .step-pill.boss { border: 1px solid var(--red); }
        .step-pill.boss.active { background: var(--red); box-shadow: 0 0 10px var(--red); }

        .narrative-box {
            background: rgba(0, 0, 0, 0.6); border: 1px solid var(--cyan);
            border-radius: 8px; padding: 15px; cursor: pointer; min-height: 80px;
        }
        .narrative-header { font-family: 'Orbitron', sans-serif; font-size: 0.75rem; color: var(--yellow); margin-bottom: 6px; display: flex; justify-content: space-between; }
        .narrative-text { font-size: 0.98rem; line-height: 1.45; color: #fff; }

        .schema-box {
            background: #03060d; border: 1px solid rgba(255,255,255,0.1);
            border-radius: 8px; padding: 10px; display: flex; flex-direction: column; align-items: center; gap: 6px;
        }
        .schema-title { font-family: 'Orbitron', sans-serif; font-size: 0.78rem; color: var(--cyan); align-self: flex-start; }

        .math-card {
            background: rgba(255, 255, 255, 0.02); border: 1px solid rgba(255, 255, 255, 0.08);
            padding: 10px 14px; border-radius: 6px; display: flex; flex-direction: column; gap: 4px;
        }
        .math-title { font-family: 'Orbitron', sans-serif; font-size: 0.75rem; color: var(--yellow); }
        .math-eq { font-size: 1.05rem; padding: 2px 0; overflow-x: auto; }

        .challenge-box {
            background: rgba(0, 243, 255, 0.05); border: 1px solid var(--cyan);
            border-radius: 8px; padding: 14px; display: flex; flex-direction: column; gap: 6px;
        }
        .challenge-box.boss { background: rgba(255, 0, 85, 0.08); border-color: var(--red); }
        .challenge-title { font-family: 'Orbitron', sans-serif; font-size: 0.88rem; color: var(--yellow); }
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
            padding: 10px 20px; border-radius: 6px; font-size: 0.85rem; font-weight: 700;
            cursor: pointer; transition: all 0.2s; align-self: flex-end;
            display: flex; align-items: center; gap: 8px;
        }
        .nav-btn:hover { background: var(--cyan); color: #000; box-shadow: 0 0 15px var(--cyan); }
        .nav-btn.boss-btn { border-color: var(--red); background: rgba(255, 0, 85, 0.2); }
        .nav-btn.boss-btn:hover { background: var(--red); color: #fff; box-shadow: 0 0 15px var(--red); }

        /* EQUIPO DE FÓRMULAS SECCIÓN */
        .formulas-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
            gap: 15px; padding: 10px 0;
        }
        .formula-card {
            background: rgba(255,255,255,0.03);
            border: 1px solid rgba(0, 243, 255, 0.2);
            border-radius: 8px; padding: 14px;
            display: flex; flex-direction: column; gap: 8px;
        }
        .formula-card-title { font-family: 'Orbitron', sans-serif; color: var(--yellow); font-size: 0.85rem; }
        .formula-card-derivation { font-size: 0.8rem; color: var(--text-muted); line-height: 1.35; }

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
    </style>
</head>
<body>

    <!-- HUD SUPERIOR -->
    <header>
        <div class="hud-title">⚡ CIRCUITY RPG: AC & FILTROS</div>
        <div class="hud-controls">
            <button class="btn-hud btn-formulas" onclick="openFormulasModal()">📜 EQUIPO DE FÓRMULAS</button>
            <button class="btn-hud" onclick="openCodex()">👾 CÓDEX</button>
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

    <!-- ESCENARIO PRINCIPAL CON ZOOM -->
    <div id="viewport-stage">
        <div id="world-container">
            
            <!-- OVERLAY SVG PARA CABLES DE COBRE -->
            <svg id="cable-svg">
                <defs>
                    <linearGradient id="copper-grad" x1="0%" y1="0%" x2="100%" y2="100%">
                        <stop offset="0%" stop-color="#e5a059" />
                        <stop offset="50%" stop-color="#b87333" />
                        <stop offset="100%" stop-color="#8b5a2b" />
                    </linearGradient>
                </defs>

                <path id="path-copper-base" class="copper-base" />
                <path id="path-copper-core" class="copper-core" />
                <path id="path-copper-texture" class="copper-texture" />

                <!-- SPRITE ELECTRÓN CON SIGNO - -->
                <g id="electron-group">
                    <circle r="12" class="electron-core" />
                    <text class="electron-text">-</text>
                </g>
            </svg>

            <!-- TIERRA (ORIGEN PRINCIPAL) -->
            <div class="terminal-node" id="node-ground">
                <svg width="30" height="30" viewBox="0 0 24 24" fill="none" stroke="#e5a059" stroke-width="2.5">
                    <line x1="12" y1="3" x2="12" y2="13" />
                    <line x1="4" y1="13" x2="20" y2="13" />
                    <line x1="7" y1="17" x2="17" y2="17" />
                    <line x1="10" y1="21" x2="14" y2="21" />
                </svg>
                <span class="terminal-label">TIERRA</span>
            </div>

            <!-- MUNDO 1: ANÁLISIS SENOIDAL Y FILTROS -->
            <div class="world-stage unlocked" id="world-1">
                <div class="world-circle unlocked" onclick="zoomIntoWorld('world-1')">
                    <div class="world-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%23081026\'/><path d=\'M 10 100 Q 50 20 100 100 T 190 100\' stroke=\'%2300f3ff\' stroke-width=\'8\' fill=\'none\'/></svg>');"></div>
                    <div class="world-overlay-content">
                        <span class="world-num">MUNDO 1</span>
                        <h4 class="world-title">Senoidales & Filtros</h4>
                        <span class="world-badge" id="tag-world-1">ACTIVO</span>
                    </div>
                </div>
                <!-- SUBTEMAS HORIZONTALES (EN ZOOM EXTREMO) -->
                <div class="horizontal-levels-container">
                    <div class="sublevel-card unlocked" id="sub-1-1" onclick="openQuest('1-1')">
                        <div class="sublevel-icon">🌊</div>
                        <div class="sublevel-name">1.1 Onda Senoidal & Fasores</div>
                    </div>
                    <div class="sublevel-card locked" id="sub-1-2" onclick="openQuest('1-2')">
                        <div class="sublevel-icon">📉</div>
                        <div class="sublevel-name">1.2 Filtro Paso Bajo RC (LPF)</div>
                    </div>
                    <div class="sublevel-card locked" id="sub-1-3" onclick="openQuest('1-3')">
                        <div class="sublevel-icon">📈</div>
                        <div class="sublevel-name">1.3 Filtro Paso Alto RC (HPF)</div>
                    </div>
                </div>
            </div>

            <!-- MUNDO 2: POTENCIA AC -->
            <div class="world-stage locked" id="world-2">
                <div class="world-circle locked" onclick="zoomIntoWorld('world-2')">
                    <div class="world-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%231a0b2e\'/><polygon points=\'30,170 170,170 170,30\' stroke=\'%23ff0055\' stroke-width=\'6\' fill=\'none\'/></svg>');"></div>
                    <div class="world-overlay-content">
                        <span class="world-num">MUNDO 2</span>
                        <h4 class="world-title">Potencia AC</h4>
                        <span class="world-badge" id="tag-world-2">BLOQUEADO</span>
                    </div>
                </div>
                <div class="horizontal-levels-container">
                    <div class="sublevel-card locked" id="sub-2-1" onclick="openQuest('2-1')">
                        <div class="sublevel-icon">⚡</div>
                        <div class="sublevel-name">2.1 Potencia RMS & Promedio</div>
                    </div>
                    <div class="sublevel-card locked" id="sub-2-2" onclick="openQuest('2-2')">
                        <div class="sublevel-icon">📐</div>
                        <div class="sublevel-name">2.2 Triángulo de Potencia</div>
                    </div>
                    <div class="sublevel-card locked" id="sub-2-3" onclick="openQuest('2-3')">
                        <div class="sublevel-icon">🏭</div>
                        <div class="sublevel-name">2.3 Corrección del FP</div>
                    </div>
                </div>
            </div>

            <!-- MUNDO 3: ACOPLO MAGNÉTICO -->
            <div class="world-stage locked" id="world-3">
                <div class="world-circle locked" onclick="zoomIntoWorld('world-3')">
                    <div class="world-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%230b1e10\'/><circle cx=\'70\' cy=\'100\' r=\'40\' stroke=\'%23ffb700\' stroke-width=\'6\' fill=\'none\'/><circle cx=\'130\' cy=\'100\' r=\'40\' stroke=\'%2300ff66\' stroke-width=\'6\' fill=\'none\'/></svg>');"></div>
                    <div class="world-overlay-content">
                        <span class="world-num">MUNDO 3</span>
                        <h4 class="world-title">Acoplo Magnético</h4>
                        <span class="world-badge" id="tag-world-3">BLOQUEADO</span>
                    </div>
                </div>
                <div class="horizontal-levels-container">
                    <div class="sublevel-card locked" id="sub-3-1" onclick="openQuest('3-1')">
                        <div class="sublevel-icon">🧲</div>
                        <div class="sublevel-name">3.1 Inductancia Mutua</div>
                    </div>
                    <div class="sublevel-card locked" id="sub-3-2" onclick="openQuest('3-2')">
                        <div class="sublevel-icon">🔴</div>
                        <div class="sublevel-name">3.2 Regla de Puntos</div>
                    </div>
                    <div class="sublevel-card locked" id="sub-3-3" onclick="openQuest('3-3')">
                        <div class="sublevel-icon">🔌</div>
                        <div class="sublevel-name">3.3 El Transformador Ideal</div>
                    </div>
                </div>
            </div>

            <!-- MUNDO 4: SISTEMAS TRIFÁSICOS -->
            <div class="world-stage locked" id="world-4">
                <div class="world-circle locked" onclick="zoomIntoWorld('world-4')">
                    <div class="world-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%23201408\'/><line x1=\'100\' y1=\'100\' x2=\'170\' y2=\'100\' stroke=\'%23ff0055\' stroke-width=\'6\'/><line x1=\'100\' y1=\'100\' x2=\'65\' y2=\'160\' stroke=\'%2300f3ff\' stroke-width=\'6\'/><line x1=\'100\' y1=\'100\' x2=\'65\' y2=\'40\' stroke=\'%2300ff66\' stroke-width=\'6\'/></svg>');"></div>
                    <div class="world-overlay-content">
                        <span class="world-num">MUNDO 4</span>
                        <h4 class="world-title">Sistemas Trifásicos</h4>
                        <span class="world-badge" id="tag-world-4">BLOQUEADO</span>
                    </div>
                </div>
                <div class="horizontal-levels-container">
                    <div class="sublevel-card locked" id="sub-4-1" onclick="openQuest('4-1')">
                        <div class="sublevel-icon">🌀</div>
                        <div class="sublevel-name">4.1 Generador Trifásico</div>
                    </div>
                    <div class="sublevel-card locked" id="sub-4-2" onclick="openQuest('4-2')">
                        <div class="sublevel-icon">🔺</div>
                        <div class="sublevel-name">4.2 Conexión Y / Δ</div>
                    </div>
                    <div class="sublevel-card locked" id="sub-4-3" onclick="openQuest('4-3')">
                        <div class="sublevel-icon">⚖️</div>
                        <div class="sublevel-name">4.3 Cargas Desequilibradas</div>
                    </div>
                </div>
            </div>

            <!-- FUENTE DE VOLTAJE (FINAL) -->
            <div class="terminal-node" id="node-source">
                <svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="#e5a059" stroke-width="2">
                    <circle cx="12" cy="12" r="9" />
                    <path d="M 7 12 Q 9.5 7 12 12 T 17 12" />
                </svg>
                <span class="terminal-label">FUENTE AC</span>
            </div>

        </div>
    </div>

    <!-- BOTÓN RECTANGULAR FLOTANTE PARA SALIR DEL ZOOM -->
    <button class="btn-exit-zoom" id="btn-exit-zoom" onclick="exitWorldZoom()">🔍 SALIR DEL MUNDO</button>

    <!-- MODAL DE MISIÓN / SUBTEMA -->
    <div class="modal-overlay" id="quest-modal">
        <div class="modal-card">
            <div class="modal-header">
                <h3 id="modal-title">Título de Misión</h3>
                <button class="close-btn" onclick="closeQuest()">&times;</button>
            </div>
            <div class="modal-body">
                
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

                <!-- PASO 0: TEORÍA Y DERIVACIÓN DESDE LO BÁSICO -->
                <div class="step-view" id="step-0">
                    <div class="narrative-box" onclick="skipTypewriter()">
                        <div class="narrative-header">
                            <span>TRANSMISIÓN DE DATOS ACADÉMICOS</span>
                            <span style="font-size:0.65rem; color:var(--text-muted);">(Clic para revelar todo)</span>
                        </div>
                        <div class="narrative-text typing-cursor" id="narrative-text-el"></div>
                    </div>

                    <div class="schema-box">
                        <div class="schema-title">🔍 ESQUEMA TÉCNICO DEL CIRCUITO</div>
                        <canvas id="schema-canvas" width="650" height="140"></canvas>
                    </div>
                    
                    <div class="math-card">
                        <div class="math-title">Ecuación de Origen Básica</div>
                        <div class="math-eq" id="eq-1"></div>
                    </div>

                    <div class="math-card">
                        <div class="math-title">Derivación y Formulación Avanzada</div>
                        <div class="math-eq" id="eq-2"></div>
                    </div>

                    <button class="nav-btn" onclick="nextSubstep()">
                        Continuar al Ejemplo ➔
                    </button>
                </div>

                <!-- PASOS 1 A 5: SIMULADOR (DEMO, EJERCICIOS Y BOSS) -->
                <div class="step-view" id="step-sim-view" style="display:none;">
                    
                    <div class="challenge-box" id="challenge-banner">
                        <div class="challenge-title" id="challenge-title">Modo Ejemplo Guiado</div>
                        <div id="challenge-desc" style="font-size:0.88rem; color:#fff;">Observa cómo responde el simulador automáticamente.</div>
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

    <!-- MODAL EQUIPO DE FÓRMULAS -->
    <div class="modal-overlay" id="formulas-modal">
        <div class="modal-card" style="max-width: 800px;">
            <div class="modal-header">
                <h3>📜 SECCIÓN DE EQUIPOS & FÓRMULAS</h3>
                <button class="close-btn" onclick="closeFormulasModal()">&times;</button>
            </div>
            <div class="modal-body">
                <p style="font-size:0.85rem; color:var(--text-muted);">
                    Consulta el desarrollo deductivo desde las leyes fundamentales de Ohm y Kirchhoff hasta las ecuaciones avanzadas de Filtros, Potencia Compleja y Trifásica.
                </p>
                <div class="formulas-grid" id="formulas-grid"></div>
            </div>
        </div>
    </div>

    <!-- MODAL CÓDEX DE PERSONAJES -->
    <div class="modal-overlay" id="codex-modal">
        <div class="modal-card" style="max-width: 650px;">
            <div class="modal-header">
                <h3>👾 CÓDEX DE PERSONAJES</h3>
                <button class="close-btn" onclick="closeCodex()">&times;</button>
            </div>
            <div class="modal-body">
                <div class="codex-grid" id="codex-grid"></div>
                <div class="char-detail-box" id="char-detail-box">
                    <div class="char-detail-title" id="char-detail-name">Selecciona un personaje</div>
                    <div id="char-detail-desc" style="font-size: 0.88rem;"></div>
                </div>
            </div>
        </div>
    </div>

    <script>
        /* --- BASE DE DATOS DE FÓRMULAS --- */
        const formulasDB = [
            { title: "Divisor de Tensión -> Filtro RC Paso Bajo", eq: "H(j\\omega) = \\frac{1}{1 + j\\omega RC}", dev: "De $V_{out} = V_{in} \\frac{Z_C}{R + Z_C}$ reemplazando $Z_C = \\frac{1}{j\\omega C}$. Frecuencia de corte $\\omega_c = \\frac{1}{RC}$." },
            { title: "Divisor de Tensión -> Filtro RC Paso Alto", eq: "H(j\\omega) = \\frac{j\\omega RC}{1 + j\\omega RC}", dev: "De $V_{out} = V_{in} \\frac{R}{R + Z_C}$ reemplazando $Z_C = \\frac{1}{j\\omega C}$. Deja pasar altas frecuencias." },
            { title: "Fasor de Voltaje Senoidal", eq: "\\mathbf{V} = \\frac{V_m}{\\sqrt{2}} e^{j\\phi}", dev: "Transformación del dominio del tiempo $v(t) = V_m \\cos(\\omega t + \\phi)$ al dominio de la frecuencia mediante la identidad de Euler." },
            { title: "Impedancia Compleja RLC", eq: "\\mathbf{Z} = R + j\\left(\\omega L - \\frac{1}{\\omega C}\\right)", dev: "Suma fasorial de Resistencia real $R$, Reactancia Inductiva $X_L = \\omega L$ y Capacitiva $X_C = -\\frac{1}{\\omega C}$." },
            { title: "Potencia Compleja Vectorial", eq: "\\mathbf{S} = P + jQ = \\mathbf{V}_{rms} \\mathbf{I}_{rms}^*", dev: "Producto del voltaje RMS por el conjugado complejo de la corriente RMS. Modulo $|\\mathbf{S}| = \\sqrt{P^2 + Q^2}$ [VA]." },
            { title: "Inductancia Mutua", eq: "M = k \\sqrt{L_1 L_2}", dev: "Grado de acoplamiento magnético $k \\in [0, 1]$ entre dos devanados acoplados por flujo compartido." },
            { title: "Relación de Línea Trifásica", eq: "V_L = \\sqrt{3} V_p \\angle 30^\\circ", dev: "Demostración geométrica de la diferencia vectorial entre dos fases con desfase de $120^\\circ$ en conexión Estrella." }
        ];

        /* --- BASE DE DATOS DE PERSONAJES Y SUBTEMAS --- */
        const charactersDB = {
            V: { symbol: 'V', name: 'Voltaje (Tensión)', desc: 'Fuerza electromotriz senoidal que impulsa los electrones.' },
            I: { symbol: 'I', name: 'Corriente (Intensidad)', desc: 'Flujo eléctrico que oscila en el tiempo con desfase.' },
            w: { symbol: 'ω', name: 'Frecuencia Angular', desc: 'Velocidad angular de rotación fasorial (ω = 2πf).' },
            H: { symbol: 'H(jω)', name: 'Función Transferencia', desc: 'Relación Vout/Vin en magnitud y fase de un filtro.' },
            Z: { symbol: 'Z', name: 'Impedancia Compleja', desc: 'Oposición total en AC (Resistencia + Reactancia).' },
            P: { symbol: 'P', name: 'Potencia Activa', desc: 'Energía promedio convertida en trabajo real (Watts).' },
            S: { symbol: 'S', name: 'Potencia Aparente', desc: 'Capacidad total suministrada por la red (VA).' },
            FP: { symbol: 'FP', name: 'Factor de Potencia', desc: 'Razón cos(θ) de eficiencia energética.' },
            M: { symbol: 'M', name: 'Inductancia Mutua', desc: 'Tensión inducida magnéticamente entre bobinas.' },
            VL: { symbol: 'VL', name: 'Tensión Trifásica', desc: 'Voltaje de línea en sistemas polifásicos (√3 · VP).' }
        };

        const questsDB = {
            '1-1': {
                title: 'Misión 1.1: Onda Senoidal y Fasores', world: 'world-1', next: '1-2', introChar: 'V',
                narrative: 'Cuando una espira gira dentro de un campo magnético, produce una tensión senoidal $v(t) = V_m \\cos(\\omega t + \\phi)$. Para simplificar cálculos sin resolver ecuaciones diferenciales, la transformamos en un <b>Fasor</b> $\\mathbf{V} = V_{rms} \\angle \\phi$.',
                eq1: 'v(t) = V_m \\cdot \\cos(\\omega t + \\phi)', eq2: '\\mathbf{V} = \\frac{V_m}{\\sqrt{2}} \\angle \\phi',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Fasor V:</b> Magnitud y fase.</div><div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Señal Senoidal:</b> Muestra en osciloscopio.</div>',
                controls: `<div class="control-group"><label>Amplitud Vm: <span id="val-vm">100</span> V</label><input type="range" id="input-vm" min="30" max="150" value="100" oninput="drawSimSenoidal()"></div><div class="control-group"><label>Desfase ϕ: <span id="val-phi">45</span>°</label><input type="range" id="input-phi" min="-180" max="180" value="45" oninput="drawSimSenoidal()"></div>`,
                drawSchema: (ctx) => drawSchemaGen(ctx), drawSim: () => drawSimSenoidal(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "El simulador varía Vm y ϕ dinámicamente." },
                    { title: "Ejercicio 1 de 3", desc: "Ajusta la Amplitud Vm a 120 V.", check: () => parseFloat(document.getElementById('input-vm').value) === 120 },
                    { title: "Ejercicio 2 de 3", desc: "Ajusta el Desfase ϕ a +90°.", check: () => parseFloat(document.getElementById('input-phi').value) === 90 },
                    { title: "Ejercicio 3 de 3", desc: "Ajusta oposición de fase completa (-180°).", check: () => parseFloat(document.getElementById('input-phi').value) === -180 },
                    { title: "👾 DESAFÍO BOSS: Onda con Vrms = 100V", desc: "Fija Vm = 141 V y desfase ϕ = +45°.", check: () => parseFloat(document.getElementById('input-vm').value) === 141 && parseFloat(document.getElementById('input-phi').value) === 45 }
                ]
            },
            '1-2': {
                title: 'Misión 1.2: Filtro Paso Bajo RC (1er Orden)', world: 'world-1', next: '1-3', introChar: 'H',
                narrative: 'Un filtro RC Paso Bajo atenúa las altas frecuencias dejando pasar las bajas. <b>Derivación:</b> Usando el Divisor de Tensión $V_{out} = V_{in} \\frac{Z_C}{R + Z_C}$ con $Z_C = \\frac{1}{j\\omega C}$, obtenemos la función de transferencia $H(j\\omega) = \\frac{1}{1 + j\\omega RC}$. La frecuencia de corte $f_c = \\frac{1}{2\\pi RC}$ marca la atenuación de $-3\\text{ dB}$.',
                eq1: 'V_{out} = V_{in} \\cdot \\frac{\\frac{1}{j\\omega C}}{R + \\frac{1}{j\\omega C}}', eq2: 'H(j\\omega) = \\frac{1}{1 + j\\omega RC} \\implies f_c = \\frac{1}{2\\pi RC}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Respuesta LPF:</b> Caída de -20 dB/década tras fc.</div>',
                controls: `<div class="control-group"><label>Resistencia R: <span id="val-r">1000</span> Ω</label><input type="range" id="input-r" min="100" max="5000" step="100" value="1000" oninput="drawSimLPF()"></div><div class="control-group"><label>Capacitancia C: <span id="val-c">0.1</span> µF</label><input type="range" id="input-c" min="0.01" max="1" step="0.01" value="0.1" oninput="drawSimLPF()"></div>`,
                drawSchema: (ctx) => drawSchemaLPF(ctx), drawSim: () => drawSimLPF(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Variación de la frecuencia de corte fc del filtro." },
                    { title: "Ejercicio 1 de 3", desc: "Fija la Resistencia R = 2000 Ω.", check: () => parseFloat(document.getElementById('input-r').value) === 2000 },
                    { title: "Ejercicio 2 de 3", desc: "Ajusta C = 0.5 µF.", check: () => Math.abs(parseFloat(document.getElementById('input-c').value) - 0.5) < 0.01 },
                    { title: "Ejercicio 3 de 3", desc: "Establece R = 3000 Ω y C = 0.2 µF.", check: () => parseFloat(document.getElementById('input-r').value) === 3000 && Math.abs(parseFloat(document.getElementById('input-c').value) - 0.2) < 0.01 },
                    { title: "👾 DESAFÍO BOSS: Frecuencia de Corte fc ≈ 1591 Hz", desc: "Ajusta R = 1000 Ω y C = 0.1 µF para obtener fc = 1591.5 Hz.", check: () => parseFloat(document.getElementById('input-r').value) === 1000 && Math.abs(parseFloat(document.getElementById('input-c').value) - 0.1) < 0.01 }
                ]
            },
            '1-3': {
                title: 'Misión 1.3: Filtro Paso Alto RC (1er Orden)', world: 'world-1', next: '2-1', introChar: 'H',
                narrative: 'Al intercambiar la posición del condensador y la resistencia, obtenemos un Filtro Paso Alto. <b>Derivación:</b> $V_{out} = V_{in} \\frac{R}{R + Z_C} = V_{in} \\frac{j\\omega RC}{1 + j\\omega RC}$. Bloquea la corriente continua (DC, $\\omega = 0$) y deja pasar las componentes de alta frecuencia.',
                eq1: 'V_{out} = V_{in} \\cdot \\frac{R}{R + \\frac{1}{j\\omega C}}', eq2: 'H(j\\omega) = \\frac{j\\omega RC}{1 + j\\omega RC}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Respuesta HPF:</b> Pasa altas frecuencias.</div>',
                controls: `<div class="control-group"><label>Resistencia R: <span id="val-rh">1000</span> Ω</label><input type="range" id="input-rh" min="100" max="5000" step="100" value="1000" oninput="drawSimHPF()"></div><div class="control-group"><label>Capacitancia C: <span id="val-ch">0.1</span> µF</label><input type="range" id="input-ch" min="0.01" max="1" step="0.01" value="0.1" oninput="drawSimHPF()"></div>`,
                drawSchema: (ctx) => drawSchemaHPF(ctx), drawSim: () => drawSimHPF(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Comportamiento en altas frecuencias del HPF." },
                    { title: "Ejercicio 1 de 3", desc: "Fija R = 1500 Ω.", check: () => parseFloat(document.getElementById('input-rh').value) === 1500 },
                    { title: "Ejercicio 2 de 3", desc: "Ajuste C = 0.05 µF.", check: () => Math.abs(parseFloat(document.getElementById('input-ch').value) - 0.05) < 0.005 },
                    { title: "Ejercicio 3 de 3", desc: "Ajuste R = 4000 Ω y C = 0.1 µF.", check: () => parseFloat(document.getElementById('input-rh').value) === 4000 && Math.abs(parseFloat(document.getElementById('input-ch').value) - 0.1) < 0.01 },
                    { title: "👾 DESAFÍO BOSS: Filtro de Cierre fc ≈ 318 Hz", desc: "Consigue fc ≈ 318 Hz ajustando R = 5000 Ω y C = 0.1 µF.", check: () => parseFloat(document.getElementById('input-rh').value) === 5000 && Math.abs(parseFloat(document.getElementById('input-ch').value) - 0.1) < 0.01 }
                ]
            },
            '2-1': {
                title: 'Misión 2.1: Potencia Instantánea y RMS', world: 'world-2', next: '2-2', introChar: 'P',
                narrative: 'La potencia instantánea oscila al doble de frecuencia. El valor RMS ($V_{rms} = V_m / \\sqrt{2}$) representa el equivalente en continua que produce el mismo trabajo térmico.',
                eq1: 'p(t) = v(t) \\cdot i(t)', eq2: 'P_{prom} = V_{rms} \\cdot I_{rms} \\cdot \\cos(\\theta)',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Potencia Promedio P:</b> Trabajo útil real.</div>',
                controls: `<div class="control-group"><label>Voltaje Vrms: <span id="val-vrms">120</span> V</label><input type="range" id="input-vrms" min="60" max="240" value="120" oninput="drawSimPinst()"></div>`,
                drawSchema: (ctx) => drawSchemaPinst(ctx), drawSim: () => drawSimPinst(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Oscilación de p(t) y línea de P promedio." },
                    { title: "Ejercicio 1 de 3", desc: "Establece Vrms = 110 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 110 },
                    { title: "Ejercicio 2 de 3", desc: "Suba a Vrms = 220 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 220 },
                    { title: "Ejercicio 3 de 3", desc: "Baje a Vrms = 80 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 80 },
                    { title: "👾 DESAFÍO BOSS: Potencia P_prom = 360W", desc: "Ajusta Vrms a 240 V para alcanzar P = 360W.", check: () => parseFloat(document.getElementById('input-vrms').value) === 240 }
                ]
            },
            '2-2': {
                title: 'Misión 2.2: Triángulo de Potencia Compleja', world: 'world-2', next: '2-3', introChar: 'S',
                narrative: 'Unimos la Potencia Activa $P$ (Watts) y la Reactiva $Q$ (VAR) en el número complejo $\\mathbf{S} = P + jQ$ de magnitud $|\\mathbf{S}| = \\sqrt{P^2 + Q^2}$.',
                eq1: '\\mathbf{S} = P + jQ', eq2: '|\\mathbf{S}| = \\sqrt{P^2 + Q^2} \\quad [\\text{VA}]',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Vector S:</b> Potencia aparente entregada.</div>',
                controls: `<div class="control-group"><label>Reactiva QL: <span id="val-ql">250</span> VAR</label><input type="range" id="input-ql" min="50" max="450" value="250" oninput="drawSimTriang()"></div>`,
                drawSchema: (ctx) => drawSchemaTriang(ctx), drawSim: () => drawSimTriang(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Crecimiento del vector de potencia aparente S." },
                    { title: "Ejercicio 1 de 3", desc: "Fija QL = 100 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 100 },
                    { title: "Ejercicio 2 de 3", desc: "Sube QL = 300 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 300 },
                    { title: "Ejercicio 3 de 3", desc: "Aumenta QL = 400 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 400 },
                    { title: "👾 DESAFÍO BOSS: Triángulo Isósceles (P = Q)", desc: "Como P = 300 W, ajusta QL a 300 VAR para formar 45°.", check: () => parseFloat(document.getElementById('input-ql').value) === 300 }
                ]
            },
            '2-3': {
                title: 'Misión 2.3: Corrección del Factor de Potencia', world: 'world-2', next: '3-1', introChar: 'FP',
                narrative: 'Inyectamos potencia capacitiva $Q_C$ con bancos de condensadores en paralelo para llevar el Factor de Potencia $FP = \\cos\\theta$ cerca de 1.0.',
                eq1: 'FP = \\frac{P}{S} = \\cos(\\theta)', eq2: 'Q_C = P (\\tan\\theta_1 - \\tan\\theta_2)',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Triángulo Compensado:</b> Reducción de S.</div>',
                controls: `<div class="control-group"><label>Inyección QC: <span id="val-qc">150</span> VAR</label><input type="range" id="input-qc" min="0" max="350" value="150" oninput="drawSimFP()"></div>`,
                drawSchema: (ctx) => drawSchemaFP(ctx), drawSim: () => drawSimFP(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Efecto de la compensación de inductancia." },
                    { title: "Ejercicio 1 de 3", desc: "Inyecta QC = 100 VAR.", check: () => parseFloat(document.getElementById('input-qc').value) === 100 },
                    { title: "Ejercicio 2 de 3", desc: "Sube a QC = 200 VAR.", check: () => parseFloat(document.getElementById('input-qc').value) === 200 },
                    { title: "Ejercicio 3 de 3", desc: "Compensación total QC = 300 VAR.", check: () => parseFloat(document.getElementById('input-qc').value) === 300 },
                    { title: "👾 DESAFÍO BOSS: FP Eficiente ≥ 0.95", desc: "Inyecta QC = 210 VAR para lograr FP = 0.96.", check: () => parseFloat(document.getElementById('input-qc').value) === 210 }
                ]
            },
            '3-1': {
                title: 'Misión 3.1: Inductancia Mutua y Acoplo', world: 'world-3', next: '3-2', introChar: 'M',
                narrative: 'Dos bobinas acopladas magnéticamente comparten flujo, induciendo tensión $M = k \\sqrt{L_1 L_2}$.',
                eq1: 'v_2(t) = M \\frac{di_1}{dt}', eq2: 'M = k \\sqrt{L_1 L_2}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Líneas de Flujo k:</b> Densidad acoplada.</div>',
                controls: `<div class="control-group"><label>Coeficiente k: <span id="val-k">0.7</span></label><input type="range" id="input-k" min="0.1" max="1" step="0.05" value="0.7" oninput="drawSimMutua()"></div>`,
                drawSchema: (ctx) => drawSchemaMutua(ctx), drawSim: () => drawSimMutua(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Acoplamiento magnético k." },
                    { title: "Ejercicio 1 de 3", desc: "Ajuste k = 0.3.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 0.3) < 0.01 },
                    { title: "Ejercicio 2 de 3", desc: "Ajuste k = 0.6.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 0.6) < 0.01 },
                    { title: "Ejercicio 3 de 3", desc: "Ajuste k = 0.9.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 0.9) < 0.01 },
                    { title: "👾 DESAFÍO BOSS: Acoplo Perfecto Ideal", desc: "Fija k = 1.0 al máximo.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 1.0) < 0.01 }
                ]
            },
            '3-2': {
                title: 'Misión 3.2: Regla de Puntos en Bobinas', world: 'world-3', next: '3-3', introChar: 'M',
                narrative: 'Los puntos establecen la referencia de polaridad de la tensión inducida por la corriente que entra al terminal.',
                eq1: '\\mathbf{V}_1 = j\\omega L_1 \\mathbf{I}_1 \\pm j\\omega M \\mathbf{I}_2', eq2: '\\mathbf{V}_2 = j\\omega L_2 \\mathbf{I}_2 \\pm j\\omega M \\mathbf{I}_1',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Puntos Polares:</b> Referencias.</div>',
                controls: `<div class="control-group"><label>Dirección I2: <span id="val-dir">Mismo Punto</span></label><input type="range" id="input-dir" min="0" max="1" step="1" value="1" oninput="drawSimDots()"></div>`,
                drawSchema: (ctx) => drawSchemaDots(ctx), drawSim: () => drawSimDots(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Inversión de concordancia de puntos." },
                    { title: "Ejercicio 1 de 3", desc: "Coloca la corriente en Punto Opuesto (0).", check: () => parseInt(document.getElementById('input-dir').value) === 0 },
                    { title: "Ejercicio 2 de 3", desc: "Cambia a Mismo Punto (1).", check: () => parseInt(document.getElementById('input-dir').value) === 1 },
                    { title: "Ejercicio 3 de 3", desc: "Retorna a Punto Opuesto (0).", check: () => parseInt(document.getElementById('input-dir').value) === 0 },
                    { title: "👾 DESAFÍO BOSS: Configuración Sumativa (+M)", desc: "Establece concordancia sumativa (1).", check: () => parseInt(document.getElementById('input-dir').value) === 1 }
                ]
            },
            '3-3': {
                title: 'Misión 3.3: El Transformador Ideal', world: 'world-3', next: '4-1', introChar: 'M',
                narrative: 'El transformador ideal adapta niveles de voltaje según la razón de vueltas $a = N_1 / N_2$.',
                eq1: 'a = \\frac{N_1}{N_2} = \\frac{\\mathbf{V}_1}{\\mathbf{V}_2}', eq2: '\\mathbf{Z}_{in} = a^2 \\mathbf{Z}_L',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Secundario V2:</b> Transformado.</div>',
                controls: `<div class="control-group"><label>Razón a: <span id="val-a">2.0</span></label><input type="range" id="input-a" min="0.5" max="4" step="0.5" value="2" oninput="drawSimTransfo()"></div>`,
                drawSchema: (ctx) => drawSchemaTransfo(ctx), drawSim: () => drawSimTransfo(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Escalado de voltaje de entrada a salida." },
                    { title: "Ejercicio 1 de 3", desc: "Ajusta un transformador reductor a = 2.0.", check: () => parseFloat(document.getElementById('input-a').value) === 2.0 },
                    { title: "Ejercicio 2 de 3", desc: "Ajusta elevador a = 0.5.", check: () => parseFloat(document.getElementById('input-a').value) === 0.5 },
                    { title: "Ejercicio 3 de 3", desc: "Ajusta relación de aislamiento a = 1.0.", check: () => parseFloat(document.getElementById('input-a').value) === 1.0 },
                    { title: "👾 DESAFÍO BOSS: Obtener V2 = 30 V", desc: "Ajusta a = 4.0 para reducir 120V a 30V.", check: () => parseFloat(document.getElementById('input-a').value) === 4.0 }
                ]
            },
            '4-1': {
                title: 'Misión 4.1: Generador Trifásico y Secuencias', world: 'world-4', next: '4-2', introChar: 'VL',
                narrative: 'Tres ondas senoidales desfasadas $120^\circ$ entre sí. El voltaje entre dos líneas activas es $V_L = \\sqrt{3} V_p$.',
                eq1: '\\mathbf{V}_{aN} = V_p \\angle 0^\\circ, \\quad \\mathbf{V}_{bN} = V_p \\angle -120^\\circ', eq2: 'V_L = \\sqrt{3} V_p \\approx 1.732 \\cdot V_p',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Fases ABC:</b> Desfasadas 120°.</div>',
                controls: `<div class="control-group"><label>Voltaje Vp: <span id="val-vp">120</span> V</label><input type="range" id="input-vp" min="60" max="240" value="120" oninput="drawSimTriGen()"></div>`,
                drawSchema: (ctx) => drawSchemaTriGen(ctx), drawSim: () => drawSimTriGen(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Rotación simétrica trifásica." },
                    { title: "Ejercicio 1 de 3", desc: "Ajusta Vp = 100 V.", check: () => parseFloat(document.getElementById('input-vp').value) === 100 },
                    { title: "Ejercicio 2 de 3", desc: "Sube Vp = 200 V.", check: () => parseFloat(document.getElementById('input-vp').value) === 200 },
                    { title: "Ejercicio 3 de 3", desc: "Disminuye Vp = 150 V.", check: () => parseFloat(document.getElementById('input-vp').value) === 150 },
                    { title: "👾 DESAFÍO BOSS: Tensión de Línea VL ≈ 208 V", desc: "Ajusta Vp = 120 V (VL = 120·√3 = 207.8 V).", check: () => parseFloat(document.getElementById('input-vp').value) === 120 }
                ]
            },
            '4-2': {
                title: 'Misión 4.2: Conexión Estrella Y / Delta Δ', world: 'world-4', next: '4-3', introChar: 'VL',
                narrative: 'En conexión Delta ($\Delta$), el voltaje de línea equivale al de fase, pero la corriente de línea es $I_L = \\sqrt{3} I_p$.',
                eq1: 'Y: V_L = \\sqrt{3} V_p', eq2: '\\Delta: I_L = \\sqrt{3} I_p',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Multiplicador:</b> Factor √3.</div>',
                controls: `<div class="control-group"><label>Corriente Ip: <span id="val-ip">10</span> A</label><input type="range" id="input-ip" min="2" max="25" value="10" oninput="drawSimYDelta()"></div>`,
                drawSchema: (ctx) => drawSchemaYDelta(ctx), drawSim: () => drawSimYDelta(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Relación de corrientes de fase y línea." },
                    { title: "Ejercicio 1 de 3", desc: "Ajusta Ip = 5 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 5 },
                    { title: "Ejercicio 2 de 3", desc: "Sube Ip = 15 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 15 },
                    { title: "Ejercicio 3 de 3", desc: "Aumenta Ip = 20 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 20 },
                    { title: "👾 DESAFÍO BOSS: Corriente de Línea IL ≈ 17.3 A", desc: "Ajusta Ip = 10 A para obtener IL = 17.32 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 10 }
                ]
            },
            '4-3': {
                title: 'Misión 4.3: Cargas Trifásicas Desequilibradas', world: 'world-4', next: null, introChar: 'VL',
                narrative: 'Cuando las impedancias difieren, la corriente de retorno por el neutro es distinta de cero ($I_N = I_A + I_B + I_C \\neq 0$).',
                eq1: '\\mathbf{I}_N = \\mathbf{I}_A + \\mathbf{I}_B + \\mathbf{I}_C', eq2: 'P_{total} = P_A + P_B + P_C',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Corriente IN:</b> Retorno por el cable de neutro.</div>',
                controls: `<div class="control-group"><label>Desequilibrio A: <span id="val-des">60</span>%</label><input type="range" id="input-des" min="0" max="100" value="60" oninput="drawSimDeseb()"></div>`,
                drawSchema: (ctx) => drawSchemaDeseb(ctx), drawSim: () => drawSimDeseb(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", desc: "Efecto del desequilibrio de carga en el neutro." },
                    { title: "Ejercicio 1 de 3", desc: "Ajusta desequilibrio al 20%.", check: () => parseFloat(document.getElementById('input-des').value) === 20 },
                    { title: "Ejercicio 2 de 3", desc: "Sube desequilibrio al 50%.", check: () => parseFloat(document.getElementById('input-des').value) === 50 },
                    { title: "Ejercicio 3 de 3", desc: "Equilibre totalmente a 0%.", check: () => parseFloat(document.getElementById('input-des').value) === 0 },
                    { title: "👾 DESAFÍO BOSS: Sobrecarga Total de Neutro", desc: "Lleve el desequilibrio al 100%.", check: () => parseFloat(document.getElementById('input-des').value) === 100 }
                ]
            }
        };

        /* --- ESTADO Y NAVEGACIÓN --- */
        const gameState = {
            xp: 0, level: 1,
            unlockedWorlds: ['world-1'],
            completedQuests: [],
            unlockedQuests: ['1-1'],
            unlockedChars: ['V', 'I', 'w']
        };

        let currentZoomWorld = null;
        let activeQuest = null;
        let activeSubstep = 0;
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
            const ranks = ["Novato AC", "Analista de Filtros", "Maestro de Impedancias", "Soberano Trifásico"];
            document.getElementById('player-rank').innerText = ranks[Math.min(gameState.level - 1, ranks.length - 1)];
            document.getElementById('xp-bar').style.width = `${Math.min((gameState.xp / (gameState.level * 200)) * 100, 100)}%`;

            ['world-1', 'world-2', 'world-3', 'world-4'].forEach((wId) => {
                const el = document.getElementById(wId);
                const tag = document.getElementById(`tag-${wId}`);
                const circle = el.querySelector('.world-circle');
                const isUnlocked = gameState.unlockedWorlds.includes(wId);
                
                if (isUnlocked) {
                    el.classList.remove('locked'); el.classList.add('unlocked');
                    circle.className = "world-circle unlocked"; tag.innerText = "ACTIVO";
                } else {
                    el.classList.add('locked');
                    circle.className = "world-circle locked"; tag.innerText = "BLOQUEADO 🔒";
                }
            });

            Object.keys(questsDB).forEach(qKey => {
                const btn = document.getElementById(`sub-${qKey}`);
                if (!btn) return;

                if (gameState.completedQuests.includes(qKey)) {
                    btn.className = "sublevel-card completed";
                } else if (gameState.unlockedQuests.includes(qKey)) {
                    btn.className = "sublevel-card unlocked";
                } else {
                    btn.className = "sublevel-card locked";
                }
            });

            updateCablesAndElectron();
        }

        /* --- MECÁNICA DE ZOOM DE MUNDO Y RECORRIDO HORIZONTAL --- */
        function zoomIntoWorld(worldId) {
            if (!gameState.unlockedWorlds.includes(worldId)) return;
            if (currentZoomWorld === worldId) return;

            currentZoomWorld = worldId;
            const container = document.getElementById('world-container');
            const targetEl = document.getElementById(worldId);

            targetEl.classList.add('active-zoom');
            document.getElementById('btn-exit-zoom').style.display = 'block';

            // Ocultar otros mundos para vista limpia
            ['world-1', 'world-2', 'world-3', 'world-4'].forEach(id => {
                if (id !== worldId) document.getElementById(id).style.opacity = '0.1';
            });
            document.getElementById('node-ground').style.opacity = '0.1';
            document.getElementById('node-source').style.opacity = '0.1';

            // Aplicar escala y centrado en el elemento
            const rect = targetEl.getBoundingClientRect();
            const containerRect = container.getBoundingClientRect();
            const offsetY = (containerRect.height / 2) - (targetEl.offsetTop + rect.height / 2);

            container.style.transform = `translateY(${offsetY}px) scale(1.15)`;

            setTimeout(updateCablesAndElectron, 400);
        }

        function exitWorldZoom() {
            currentZoomWorld = null;
            const container = document.getElementById('world-container');
            container.style.transform = 'translateY(0) scale(1)';

            ['world-1', 'world-2', 'world-3', 'world-4'].forEach(id => {
                const el = document.getElementById(id);
                el.classList.remove('active-zoom');
                el.style.opacity = '1';
            });
            document.getElementById('node-ground').style.opacity = '1';
            document.getElementById('node-source').style.opacity = '1';

            document.getElementById('btn-exit-zoom').style.display = 'none';
            setTimeout(updateCablesAndElectron, 400);
        }

        /* --- DIBUJO DE CABLES DE COBRE SVG Y ELECTRÓN --- */
        function updateCablesAndElectron() {
            const container = document.getElementById('world-container');
            const svg = document.getElementById('cable-svg');
            const rectW = container.getBoundingClientRect();

            svg.setAttribute('width', rectW.width);
            svg.setAttribute('height', rectW.height);

            let pathD = "";

            if (currentZoomWorld) {
                // TRAYECTORIA HORIZONTAL EN MUNDO ZOOM
                const activeWorldEl = document.getElementById(currentZoomWorld);
                const circle = activeWorldEl.querySelector('.world-circle');
                const subCards = activeWorldEl.querySelectorAll('.sublevel-card');

                if (subCards.length > 0) {
                    const cR = circle.getBoundingClientRect();
                    const cX = cR.left + cR.width/2 - rectW.left;
                    const cY = cR.top + cR.height/2 - rectW.top;

                    pathD = `M ${cX} ${cY} `;

                    subCards.forEach((card, idx) => {
                        const sR = card.getBoundingClientRect();
                        const sX = sR.left + sR.width/2 - rectW.left;
                        const sY = sR.top + sR.height/2 - rectW.top;
                        
                        const midX = (cX + sX) / 2;
                        pathD += `C ${midX} ${cY}, ${midX} ${sY}, ${sX} ${sY} `;
                    });
                }
            } else {
                // TRAYECTORIA VERTICAL PRINCIPAL ENTRE MUNDOS
                const nodes = ['node-ground', 'world-1', 'world-2', 'world-3', 'world-4', 'node-source'];
                for (let i = 0; i < nodes.length - 1; i++) {
                    const elA = document.getElementById(nodes[i]);
                    const elB = document.getElementById(nodes[i+1]);

                    const targetA = elA.querySelector('.world-circle') || elA;
                    const targetB = elB.querySelector('.world-circle') || elB;

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

            if (path && path.getTotalLength && path.getTotalLength() > 0) {
                const len = path.getTotalLength();
                const pt = path.getPointAtLength(electronAnimProgress * len);
                electronGroup.setAttribute('transform', `translate(${pt.x}, ${pt.y})`);
            }

            requestAnimationFrame(animateElectron);
        }

        /* --- MANEJO DE MISIÓN Y SUBPASOS --- */
        function openQuest(key) {
            if (!gameState.unlockedQuests.includes(key)) return;
            activeQuest = key;
            activeSubstep = 0;
            const q = questsDB[key];

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
            
            for (let i = 0; i < 6; i++) {
                const pill = document.getElementById(`pill-${i}`);
                pill.className = "step-pill" + (i === 5 ? " boss" : "");
                if (i < activeSubstep) pill.classList.add('completed');
                if (i === activeSubstep) pill.classList.add('active');
            }

            document.getElementById('progress-step-text').innerText = `Paso ${activeSubstep + 1} de 6`;

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

            if (activeSubstep >= 2 && activeSubstep <= 5) {
                const challenge = q.challenges[activeSubstep - 1];
                if (challenge.check && !challenge.check()) {
                    alert("⚠️ ¡El simulador no está en la posición exacta requerida! Revisa las instrucciones.");
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
                const nextWorld = questsDB[q.next].world;
                if (!gameState.unlockedWorlds.includes(nextWorld)) {
                    gameState.unlockedWorlds.push(nextWorld);
                }
            }

            gameState.xp += 100;
            if (gameState.xp >= gameState.level * 200) gameState.level++;

            updateUI();
            closeQuest();

            // SALIR DEL ZOOM AUTOMÁTICAMENTE AL VENCER AL JEFE
            setTimeout(() => {
                exitWorldZoom();
            }, 300);
        }

        /* --- MODAL SECCIÓN EQUIPO DE FÓRMULAS --- */
        function openFormulasModal() {
            const grid = document.getElementById('formulas-grid');
            grid.innerHTML = '';

            formulasDB.forEach(f => {
                const card = document.createElement('div');
                card.className = 'formula-card';
                card.innerHTML = `
                    <div class="formula-card-title">${f.title}</div>
                    <div class="math-eq" style="color:var(--cyan); font-size:1rem;"></div>
                    <div class="formula-card-derivation">${f.dev}</div>
                `;
                grid.appendChild(card);
                katex.render(f.eq, card.querySelector('.math-eq'), { displayMode: true, throwOnError: false });
            });

            document.getElementById('formulas-modal').classList.add('active');
        }

        function closeFormulasModal() {
            document.getElementById('formulas-modal').classList.remove('active');
        }

        /* --- CODEX --- */
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

        /* --- AUXILIARES DIBUJO CANVAS CON ANTI-SUPERPOSICIÓN --- */
        function drawPillLabel(ctx, text, x, y, color) {
            ctx.font = '12px Orbitron';
            const w = ctx.measureText(text).width + 12;
            ctx.fillStyle = 'rgba(4, 7, 17, 0.85)';
            ctx.fillRect(x - w/2, y - 10, w, 20);
            ctx.strokeStyle = color; ctx.lineWidth = 1;
            ctx.strokeRect(x - w/2, y - 10, w, 20);
            ctx.fillStyle = color; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
            ctx.fillText(text, x, y);
        }

        function drawResistor(ctx, x1, y1, x2, y2) {
            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 2.5;
            ctx.beginPath(); ctx.moveTo(x1, y1);
            const dx = (x2 - x1) / 6, dy = 8;
            ctx.lineTo(x1 + dx, y1 - dy);
            ctx.lineTo(x1 + dx*2, y1 + dy);
            ctx.lineTo(x1 + dx*3, y1 - dy);
            ctx.lineTo(x1 + dx*4, y1 + dy);
            ctx.lineTo(x1 + dx*5, y1 - dy);
            ctx.lineTo(x2, y2); ctx.stroke();
        }

        function drawCapacitor(ctx, x, y) {
            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(x - 12, y - 15); ctx.lineTo(x - 12, y + 15); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(x + 12, y - 15); ctx.lineTo(x + 12, y + 15); ctx.stroke();
        }

        /* --- ESQUEMAS DIBUJO --- */
        function drawSchemaGen(ctx) {
            ctx.clearRect(0,0,650,140);
            ctx.fillStyle='#1e293b'; ctx.fillRect(60,25,50,80); ctx.fillRect(240,25,50,80);
            drawPillLabel(ctx, 'N', 85, 65, '#ff0055');
            drawPillLabel(ctx, 'S', 265, 65, '#00f3ff');
            ctx.strokeStyle='#ffb700'; ctx.lineWidth=3; ctx.beginPath(); ctx.arc(175,65,25,0,Math.PI*2); ctx.stroke();
            drawPillLabel(ctx, 'Generador v(t)', 175, 120, '#00ff66');
        }

        function drawSchemaLPF(ctx) {
            ctx.clearRect(0,0,650,140);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.moveTo(60,70); ctx.lineTo(150,70); ctx.stroke();
            drawResistor(ctx, 150, 70, 320, 70);
            ctx.beginPath(); ctx.moveTo(320,70); ctx.lineTo(520,70); ctx.moveTo(320,70); ctx.lineTo(320,95); ctx.stroke();
            drawCapacitor(ctx, 320, 110);
            ctx.beginPath(); ctx.moveTo(320,125); ctx.lineTo(320,135); ctx.stroke();
            drawPillLabel(ctx, 'Vin(t)', 80, 40, '#ff0055');
            drawPillLabel(ctx, 'Resistencia R', 235, 40, '#ffb700');
            drawPillLabel(ctx, 'Condensador C', 410, 110, '#00f3ff');
            drawPillLabel(ctx, 'Vout(t) [Bajo]', 540, 70, '#00ff66');
        }

        function drawSchemaHPF(ctx) {
            ctx.clearRect(0,0,650,140);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.moveTo(60,70); ctx.lineTo(210,70); ctx.stroke();
            drawCapacitor(ctx, 225, 70);
            ctx.beginPath(); ctx.moveTo(240,70); ctx.lineTo(380,70); ctx.lineTo(520,70); ctx.stroke();
            drawResistor(ctx, 380, 70, 380, 125);
            drawPillLabel(ctx, 'Vin(t)', 80, 40, '#ff0055');
            drawPillLabel(ctx, 'Condensador C', 225, 35, '#00f3ff');
            drawPillLabel(ctx, 'Resistencia R', 460, 100, '#ffb700');
            drawPillLabel(ctx, 'Vout(t) [Alto]', 540, 70, '#00ff66');
        }

        function drawSchemaPinst(ctx) {
            ctx.clearRect(0,0,650,140);
            ctx.fillStyle='#1e293b'; ctx.fillRect(100,35,110,60);
            drawPillLabel(ctx, 'Fuente AC', 155, 65, '#00f3ff');
            ctx.strokeStyle='#00ff66'; ctx.lineWidth=2; ctx.strokeRect(400,35,110,60);
            drawPillLabel(ctx, 'Carga ZL', 455, 65, '#00ff66');
            ctx.beginPath(); ctx.moveTo(210,50); ctx.lineTo(400,50); ctx.moveTo(210,80); ctx.lineTo(400,80); ctx.stroke();
        }

        function drawSchemaTriang(ctx) {
            ctx.clearRect(0,0,650,140);
            ctx.strokeStyle='#ff0055'; ctx.lineWidth=2; ctx.strokeRect(150,25,90,75);
            drawPillLabel(ctx, 'Motor Q (VAR)', 195, 62, '#ff0055');
            ctx.strokeStyle='#00f3ff'; ctx.strokeRect(350,25,90,75);
            drawPillLabel(ctx, 'Resistencia P (W)', 395, 62, '#00f3ff');
        }

        function drawSchemaFP(ctx) {
            ctx.clearRect(0,0,650,140);
            ctx.strokeStyle='#00ff66'; ctx.lineWidth=2; ctx.strokeRect(80,25,110,70);
            drawPillLabel(ctx, 'Fábrica', 135, 60, '#00ff66');
            ctx.strokeStyle='#ff0055'; ctx.strokeRect(300,25,110,70);
            drawPillLabel(ctx, 'Banco -QC', 355, 60, '#ff0055');
        }

        function drawSchemaMutua(ctx) {
            ctx.clearRect(0,0,650,140);
            ctx.strokeStyle='#ffb700'; ctx.lineWidth=3;
            ctx.beginPath(); ctx.arc(200,65,30,0,Math.PI*2); ctx.stroke();
            ctx.beginPath(); ctx.arc(450,65,30,0,Math.PI*2); ctx.stroke();
            drawPillLabel(ctx, 'Inductancia Mutua M', 325, 65, '#00f3ff');
        }

        function drawSchemaDots(ctx) {
            ctx.clearRect(0,0,650,140);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.arc(220,65,25,0,Math.PI*2); ctx.stroke();
            ctx.beginPath(); ctx.arc(430,65,25,0,Math.PI*2); ctx.stroke();
            ctx.fillStyle='#ff0055'; ctx.beginPath(); ctx.arc(220,30,5,0,Math.PI*2); ctx.fill();
            ctx.beginPath(); ctx.arc(430,30,5,0,Math.PI*2); ctx.fill();
            drawPillLabel(ctx, 'Puntos Polares', 325, 110, '#ff0055');
        }

        function drawSchemaTransfo(ctx) {
            ctx.clearRect(0,0,650,140);
            ctx.fillStyle='#1e293b'; ctx.fillRect(220,20,210,90); ctx.clearRect(260,40,130,50);
            drawPillLabel(ctx, 'Primario N1', 160, 65, '#ffb700');
            drawPillLabel(ctx, 'Secundario N2', 480, 65, '#00f3ff');
        }

        function drawSchemaTriGen(ctx) {
            ctx.clearRect(0,0,650,140);
            const colors = ['#ff0055', '#00f3ff', '#00ff66'], angles = [0, 120, 240];
            angles.forEach((ang, i) => {
                const rad = (ang * Math.PI) / 180, x = 325 + 40 * Math.cos(rad), y = 65 + 40 * Math.sin(rad);
                ctx.strokeStyle = colors[i]; ctx.lineWidth = 3;
                ctx.beginPath(); ctx.moveTo(325,65); ctx.lineTo(x,y); ctx.stroke();
            });
            drawPillLabel(ctx, 'Secuencia Trifásica 120°', 325, 120, '#ffb700');
        }

        function drawSchemaYDelta(ctx) {
            ctx.clearRect(0,0,650,140);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.moveTo(150,30); ctx.lineTo(150,65); ctx.lineTo(120,95); ctx.moveTo(150,65); ctx.lineTo(180,95); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(480,30); ctx.lineTo(440,95); ctx.lineTo(520,95); ctx.closePath(); ctx.stroke();
            drawPillLabel(ctx, 'Estrella Y', 150, 115, '#00f3ff');
            drawPillLabel(ctx, 'Delta Δ', 480, 115, '#00ff66');
        }

        function drawSchemaDeseb(ctx) {
            ctx.clearRect(0,0,650,140);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.moveTo(100,65); ctx.lineTo(550,65); ctx.stroke();
            drawPillLabel(ctx, 'Retorno por Neutro IN ≠ 0', 325, 100, '#ffb700');
        }

        /* --- SIMULADORES EN CANVAS --- */
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

            ctx.strokeStyle = 'rgba(255,255,255,0.2)'; ctx.lineWidth = 1.5;
            ctx.beginPath(); ctx.moveTo(15, cy); ctx.lineTo(205, cy); ctx.moveTo(cxPhasor, 15); ctx.lineTo(cxPhasor, 195); ctx.stroke();

            const fx = cxPhasor + r * Math.cos(-phiRad), fy = cy + r * Math.sin(-phiRad);
            ctx.strokeStyle = '#ff0055'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(cxPhasor, cy); ctx.lineTo(fx, fy); ctx.stroke();

            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 2.5; ctx.beginPath();
            for (let x = 240; x <= 630; x++) {
                const t = (x - 240) * 0.02;
                const y = cy - r * Math.sin(t + phiRad);
                if (x === 240) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();
        }

        function drawSimLPF() {
            const R = parseFloat(document.getElementById('input-r').value);
            const C = parseFloat(document.getElementById('input-c').value) * 1e-6;
            document.getElementById('val-r').innerText = R;
            document.getElementById('val-c').innerText = document.getElementById('input-c').value;

            const fc = 1 / (2 * Math.PI * R * C);

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 2.5; ctx.beginPath();
            for(let x=40; x<610; x++) {
                const f = Math.pow(10, (x - 40) / 140); // Escala logarítmica
                const H = 1 / Math.sqrt(1 + Math.pow(f / fc, 2));
                const y = 180 - H * 140;
                if(x===40) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();

            const xFc = 40 + Math.log10(fc) * 140;
            if (xFc >= 40 && xFc <= 610) {
                ctx.strokeStyle = '#ffb700'; ctx.setLineDash([4, 4]);
                ctx.beginPath(); ctx.moveTo(xFc, 10); ctx.lineTo(xFc, 190); ctx.stroke(); ctx.setLineDash([]);
                ctx.fillStyle = '#ffb700'; ctx.font = '12px Orbitron';
                ctx.fillText(`fc = ${Math.round(fc)} Hz (-3dB)`, Math.min(xFc + 8, 480), 30);
            }
        }

        function drawSimHPF() {
            const R = parseFloat(document.getElementById('input-rh').value);
            const C = parseFloat(document.getElementById('input-ch').value) * 1e-6;
            document.getElementById('val-rh').innerText = R;
            document.getElementById('val-ch').innerText = document.getElementById('input-ch').value;

            const fc = 1 / (2 * Math.PI * R * C);

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 2.5; ctx.beginPath();
            for(let x=40; x<610; x++) {
                const f = Math.pow(10, (x - 40) / 140);
                const H = (f / fc) / Math.sqrt(1 + Math.pow(f / fc, 2));
                const y = 180 - H * 140;
                if(x===40) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();

            const xFc = 40 + Math.log10(fc) * 140;
            if (xFc >= 40 && xFc <= 610) {
                ctx.strokeStyle = '#ffb700'; ctx.setLineDash([4, 4]);
                ctx.beginPath(); ctx.moveTo(xFc, 10); ctx.lineTo(xFc, 190); ctx.stroke(); ctx.setLineDash([]);
                ctx.fillStyle = '#ffb700'; ctx.font = '12px Orbitron';
                ctx.fillText(`fc = ${Math.round(fc)} Hz (-3dB)`, Math.min(xFc + 8, 480), 30);
            }
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
            ctx.fillText(`Corriente de Línea Delta IL = √3 · Ip = ${IL} A`, 100, 115);
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
            ctx.fillText(`Corriente de Retorno Neutro IN = ${IN} A`, 100, 70);
        }
    </script>
</body>
</html>