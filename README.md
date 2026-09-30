<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Circuitos AC, Filtros & Protoboard RPG</title>
    <!-- KaTeX CDN -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.css">
    <script src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/contrib/auto-render.min.js"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700;900&family=Rajdhani:wght@500;600;700&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --bg-dark: #060911;
            --panel-bg: #0b1220;
            --panel-border: rgba(0, 243, 255, 0.35);
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

        html, body {
            width: 100%;
            height: 100%;
            overflow-x: hidden;
            background-color: var(--bg-dark);
            color: var(--text-main);
            font-family: 'Rajdhani', sans-serif;
        }

        /* --- HEADER GLOBAL HUD --- */
        header {
            background: rgba(6, 9, 17, 0.95);
            border-bottom: 2px solid var(--cyan);
            box-shadow: 0 0 20px rgba(0, 243, 255, 0.2);
            padding: 10px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            position: fixed;
            top: 0; left: 0; width: 100%;
            z-index: 500;
            backdrop-filter: blur(10px);
            transition: transform 0.5s ease, opacity 0.5s ease;
        }

        header.hidden-hud {
            transform: translateY(-100%);
            opacity: 0;
            pointer-events: none;
        }

        .hud-title {
            font-family: 'Orbitron', sans-serif;
            font-size: 1.1rem;
            font-weight: 800;
            color: #fff;
            text-shadow: 0 0 10px var(--cyan);
        }

        .hud-controls { display: flex; gap: 10px; align-items: center; }

        .btn-hud {
            font-family: 'Orbitron', sans-serif;
            background: rgba(157, 0, 255, 0.2);
            border: 1px solid var(--purple);
            color: #fff;
            padding: 6px 12px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 0.78rem;
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
        .stat-value { font-family: 'Orbitron', sans-serif; font-size: 0.95rem; color: var(--cyan); font-weight: 700; }
        
        .xp-container {
            width: 90px; height: 6px;
            background: rgba(255,255,255,0.1);
            border: 1px solid var(--cyan);
            border-radius: 4px; overflow: hidden; margin-top: 3px;
        }
        .xp-bar { height: 100%; width: 0%; background: linear-gradient(90deg, var(--cyan), var(--green)); transition: width 0.4s; }

        /* --- CONTENEDOR DE MAPA Y ZOOM CENTRADO --- */
        #viewport-stage {
            position: relative;
            width: 100vw;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 80px 0 60px 0;
            overflow: hidden;
            margin: 0 auto;
        }

        #world-container {
            position: relative;
            width: 100%;
            max-width: 800px;
            margin: 0 auto;
            transition: transform 0.8s cubic-bezier(0.25, 1, 0.5, 1);
            transform-origin: center center;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        /* CAPAS DE CABLE DE COBRE SVG */
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
        .electron-text { font-family: 'Orbitron', sans-serif; font-size: 14px; font-weight: 900; fill: #060911; text-anchor: middle; dominant-baseline: central; }

        /* TERMINALES DE EXTREMO */
        .terminal-node {
            position: relative;
            z-index: 2;
            width: 60px; height: 60px;
            border-radius: 50%; background: #0b1220;
            border: 2px solid var(--copper-light);
            display: flex; flex-direction: column;
            justify-content: center; align-items: center;
            box-shadow: 0 0 20px rgba(184, 115, 51, 0.4);
            margin: 10px 0;
            transition: opacity 0.5s;
        }
        .terminal-label { font-family: 'Orbitron', sans-serif; font-size: 0.6rem; color: var(--copper-light); margin-top: 3px; }

        /* CIRCULOS DE MUNDO EN MAPA GLOBAL */
        .chapter-stage {
            position: relative;
            z-index: 2;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 35px 0;
            width: 100%;
            transition: all 0.5s ease;
        }

        .chapter-circle {
            width: 140px; height: 140px;
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

        .chapter-circle.unlocked { border-color: var(--cyan); box-shadow: 0 0 30px rgba(0, 243, 255, 0.3); }
        .chapter-circle.completed { border-color: var(--green); box-shadow: 0 0 30px rgba(0, 255, 102, 0.3); }
        .chapter-circle.locked { opacity: 0.45; filter: grayscale(1); cursor: not-allowed; border-color: var(--text-muted); }

        .chapter-circle:hover:not(.locked) { transform: scale(1.08); box-shadow: 0 0 40px var(--cyan); }

        .chapter-cover-img {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background-size: cover; background-position: center;
            filter: blur(2.5px) brightness(0.45); transition: all 0.3s;
        }
        .chapter-circle:hover .chapter-cover-img { filter: blur(0px) brightness(0.65); transform: scale(1.1); }

        .chapter-overlay-content { position: relative; z-index: 3; text-align: center; padding: 10px; }
        .chapter-num { font-family: 'Orbitron', sans-serif; font-size: 0.7rem; color: var(--yellow); }
        .chapter-name-title { font-family: 'Orbitron', sans-serif; font-size: 0.85rem; font-weight: 800; color: #fff; margin: 3px 0; }
        .chapter-badge { font-family: 'Orbitron', sans-serif; font-size: 0.58rem; padding: 2px 6px; border-radius: 8px; background: rgba(0,0,0,0.8); border: 1px solid var(--cyan); color: var(--cyan); }

        /* PROTOBOARD MUNDO REAL (EN ZOOM EXTREMO) */
        .protoboard-world-container {
            display: none;
            width: 95%;
            max-width: 750px;
            background-color: #f8fafc;
            /* Textura de Protoboard Real con Carriles de Alimentación */
            background-image: 
                radial-gradient(circle, #475569 2.5px, transparent 3px),
                linear-gradient(to right, #ef4444 2.5px, transparent 2.5px),
                linear-gradient(to right, #3b82f6 2.5px, transparent 2.5px);
            background-size: 20px 20px, 100% 100%, 100% 100%;
            background-position: 0 0, 12px 0, 738px 0;
            border: 4px solid #cbd5e1;
            border-radius: 16px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.9), inset 0 0 12px rgba(0,0,0,0.15);
            padding: 20px 25px;
            margin-top: 20px;
            animation: fadeIn 0.4s ease;
            position: relative;
            z-index: 10;
        }

        .chapter-stage.active-zoom .protoboard-world-container {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .protoboard-title {
            font-family: 'Orbitron', sans-serif;
            font-size: 0.8rem;
            color: #0f172a;
            font-weight: 800;
            background: #e2e8f0;
            padding: 4px 14px;
            border-radius: 20px;
            border: 1px solid #cbd5e1;
            margin-bottom: 16px;
        }

        .protoboard-pins-grid {
            display: flex;
            justify-content: space-around;
            align-items: center;
            width: 100%;
            gap: 12px;
        }

        .pin-node-card {
            background: #0f172a;
            border: 2px solid var(--panel-border);
            border-radius: 10px;
            padding: 12px;
            width: 170px;
            display: flex; flex-direction: column; align-items: center; gap: 6px;
            cursor: pointer; transition: all 0.3s;
            box-shadow: 0 8px 18px rgba(0,0,0,0.4);
        }

        .pin-node-card:hover:not(.locked) {
            transform: translateY(-5px) scale(1.04);
            border-color: var(--cyan);
            box-shadow: 0 0 22px var(--cyan);
        }
        .pin-node-card.completed { border-color: var(--green); background: #062318; }
        .pin-node-card.locked { opacity: 0.4; cursor: not-allowed; filter: grayscale(1); }

        .pin-icon { font-size: 1.6rem; }
        .pin-name { font-family: 'Orbitron', sans-serif; font-size: 0.68rem; text-align: center; color: #fff; }

        /* BOTÓN FLOTANTE PARA SALIR DEL ZOOM */
        .btn-exit-zoom {
            position: fixed;
            bottom: 25px; right: 25px;
            z-index: 600;
            font-family: 'Orbitron', sans-serif;
            background: rgba(255, 0, 85, 0.25);
            border: 2px solid var(--red);
            color: #fff;
            padding: 10px 20px;
            border-radius: 30px;
            cursor: pointer;
            font-size: 0.82rem;
            display: none;
            box-shadow: 0 0 20px rgba(255, 0, 85, 0.5);
            backdrop-filter: blur(8px);
            transition: all 0.2s;
        }
        .btn-exit-zoom:hover { background: var(--red); box-shadow: 0 0 30px var(--red); }

        /* --- MODAL DE MISIÓN CON ESQUEMA GRANDE ARRIBA Y TEXTO TIPO DIÁLOGO ABAJO --- */
        .modal-overlay {
            position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
            background: rgba(4, 6, 12, 0.93); backdrop-filter: blur(10px);
            z-index: 1000; display: flex; justify-content: center; align-items: center;
            opacity: 0; pointer-events: none; transition: opacity 0.25s ease;
        }
        .modal-overlay.active { opacity: 1; pointer-events: all; }

        .modal-card {
            background: var(--panel-bg);
            border: 2px solid var(--cyan);
            border-radius: 14px;
            width: 92vw; max-width: 900px;
            max-height: 94vh;
            display: flex; flex-direction: column;
            overflow: hidden;
            box-shadow: 0 0 45px rgba(0, 243, 255, 0.3);
        }

        .modal-header {
            padding: 12px 20px;
            border-bottom: 1px solid var(--panel-border);
            display: flex; justify-content: space-between; align-items: center;
            background: rgba(0, 243, 255, 0.05);
        }
        .modal-header h3 { font-family: 'Orbitron', sans-serif; color: var(--cyan); font-size: 1.05rem; }
        .close-btn { background: none; border: none; color: var(--text-muted); font-size: 1.8rem; cursor: pointer; }
        .close-btn:hover { color: var(--red); }

        .modal-body {
            padding: 16px 20px;
            overflow-y: auto;
            display: flex; flex-direction: column; gap: 14px;
        }

        /* BARRA DE PASOS */
        .subtopic-progress-bar {
            display: flex; gap: 8px; width: 100%; background: rgba(0,0,0,0.4);
            padding: 8px 14px; border-radius: 8px; border: 1px solid rgba(255,255,255,0.08);
            align-items: center; justify-content: space-between;
        }
        .step-pill { flex: 1; height: 7px; background: rgba(255,255,255,0.1); border-radius: 4px; transition: all 0.3s; }
        .step-pill.active { background: var(--cyan); box-shadow: 0 0 8px var(--cyan); }
        .step-pill.completed { background: var(--green); }
        .step-pill.boss { border: 1px solid var(--red); }
        .step-pill.boss.active { background: var(--red); box-shadow: 0 0 10px var(--red); }

        /* SECCIÓN DEL ESQUEMA GRANDE (OCUPA LA PARTE SUPERIOR DE LA VENTANA) */
        .schema-box-large {
            background: #03060d;
            border: 1px solid var(--panel-border);
            border-radius: 10px;
            padding: 10px;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 6px;
            width: 100%;
            box-shadow: inset 0 0 20px rgba(0,0,0,0.8);
        }
        .schema-header-tag {
            font-family: 'Orbitron', sans-serif;
            font-size: 0.78rem;
            color: var(--cyan);
            align-self: flex-start;
            display: flex; gap: 8px; align-items: center;
        }

        /* ECUACIONES Y FORMULACIÓN */
        .math-summary-row {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            width: 100%;
        }
        .math-card-compact {
            background: rgba(255, 255, 255, 0.02);
            border: 1px solid rgba(255, 255, 255, 0.08);
            padding: 8px 12px;
            border-radius: 6px;
            display: flex; flex-direction: column; gap: 2px;
        }
        .math-title { font-family: 'Orbitron', sans-serif; font-size: 0.72rem; color: var(--yellow); }
        .math-eq { font-size: 0.95rem; overflow-x: auto; }

        /* SUBVENTANA TIPO DIÁLOGO DE PERSONAJE EN LA PARTE INFERIOR */
        .rpg-dialog-box {
            background: rgba(11, 18, 32, 0.95);
            border: 2px solid var(--cyan);
            border-radius: 12px;
            padding: 14px 18px;
            display: flex;
            gap: 15px;
            align-items: center;
            box-shadow: 0 0 20px rgba(0, 243, 255, 0.2);
            cursor: pointer;
            position: relative;
        }

        .rpg-avatar-slot {
            width: 58px; height: 58px;
            border-radius: 50%;
            background: #060911;
            border: 2px solid var(--yellow);
            display: flex; justify-content: center; align-items: center;
            font-family: 'Orbitron', sans-serif; font-size: 1.5rem; color: var(--cyan);
            flex-shrink: 0;
            box-shadow: 0 0 15px rgba(255, 183, 0, 0.4);
        }

        .rpg-dialog-content {
            display: flex; flex-direction: column; gap: 4px; width: 100%;
        }
        .rpg-speaker-name {
            font-family: 'Orbitron', sans-serif; font-size: 0.78rem; color: var(--yellow); text-transform: uppercase;
        }
        .rpg-text-narrative {
            font-size: 0.92rem; line-height: 1.4; color: #fff;
        }

        /* RETOS, SIMULADORES Y COMPLETAR ESPACIOS EN BLANCO */
        .fill-blank-card {
            background: rgba(157, 0, 255, 0.08);
            border: 1px solid var(--purple);
            border-radius: 8px;
            padding: 14px;
            display: flex; flex-direction: column; gap: 10px;
        }

        .fill-blank-input {
            background: #040812;
            border: 1px solid var(--cyan);
            color: var(--cyan);
            font-family: 'Orbitron', sans-serif;
            font-size: 0.95rem;
            padding: 8px 12px;
            border-radius: 6px;
            outline: none;
            width: 220px;
        }

        .challenge-box {
            background: rgba(0, 243, 255, 0.05); border: 1px solid var(--cyan);
            border-radius: 8px; padding: 12px; display: flex; flex-direction: column; gap: 4px;
        }
        .challenge-box.boss { background: rgba(255, 0, 85, 0.08); border-color: var(--red); }
        .challenge-title { font-family: 'Orbitron', sans-serif; font-size: 0.82rem; color: var(--yellow); }
        .boss .challenge-title { color: var(--red); }

        .sim-box {
            background: #000; border: 1px solid var(--panel-border);
            border-radius: 8px; padding: 10px; display: flex; flex-direction: column; align-items: center; gap: 8px;
        }

        canvas { background: #050811; border-radius: 4px; border: 1px solid rgba(0, 243, 255, 0.2); max-width: 100%; }

        .legend-box {
            width: 100%; background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.08);
            border-radius: 6px; padding: 6px 10px; font-size: 0.8rem; display: flex; flex-direction: column; gap: 3px;
        }
        .legend-item { display: flex; align-items: center; gap: 8px; }
        .legend-color { width: 10px; height: 10px; border-radius: 2px; }

        .controls-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr)); gap: 8px; width: 100%; }
        .control-group { display: flex; flex-direction: column; gap: 2px; }
        .control-group label { font-size: 0.75rem; color: var(--text-muted); display: flex; justify-content: space-between; }
        input[type="range"] { accent-color: var(--cyan); cursor: pointer; }

        .nav-btn {
            font-family: 'Orbitron', sans-serif;
            background: linear-gradient(135deg, rgba(0,243,255,0.2), rgba(0,255,102,0.2));
            border: 1px solid var(--cyan); color: #fff;
            padding: 8px 18px; border-radius: 6px; font-size: 0.82rem; font-weight: 700;
            cursor: pointer; transition: all 0.2s; align-self: flex-end;
            display: flex; align-items: center; gap: 6px;
        }
        .nav-btn:hover { background: var(--cyan); color: #000; box-shadow: 0 0 15px var(--cyan); }
        .nav-btn.boss-btn { border-color: var(--red); background: rgba(255, 0, 85, 0.2); }
        .nav-btn.boss-btn:hover { background: var(--red); color: #fff; box-shadow: 0 0 15px var(--red); }

        /* REVELACIÓN DE PERSONAJE */
        .reveal-card {
            background: linear-gradient(135deg, #0d1424, #1a0933);
            border: 2px solid var(--purple); box-shadow: 0 0 50px var(--purple);
            border-radius: 12px; padding: 22px; text-align: center;
            display: flex; flex-direction: column; align-items: center; gap: 10px;
            max-width: 360px; animation: popIn 0.35s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        @keyframes popIn { 0% { transform: scale(0.5); opacity: 0; } 100% { transform: scale(1); opacity: 1; } }
        .reveal-symbol { font-family: 'Orbitron', sans-serif; font-size: 2.8rem; font-weight: 900; color: var(--cyan); text-shadow: 0 0 20px var(--cyan); }

        /* FÓRMULAS ESTÁTICAS */
        .formulas-list-sidebar {
            display: grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); gap: 10px; padding: 8px 0;
        }
        .formula-card-static {
            background: rgba(255,255,255,0.03); border: 1px solid rgba(0, 243, 255, 0.2);
            border-radius: 8px; padding: 10px; display: flex; flex-direction: column; gap: 6px;
        }

        /* CODEX */
        .codex-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(95px, 1fr)); gap: 10px; padding: 8px 0; }
        .char-card {
            background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.1);
            border-radius: 8px; padding: 10px 6px; display: flex; flex-direction: column; align-items: center;
            cursor: pointer; transition: all 0.2s;
        }
        .char-card.unlocked { border-color: var(--purple); background: rgba(157,0,255,0.08); }
        .char-card.unlocked:hover { transform: scale(1.05); box-shadow: 0 0 15px var(--purple); }
        .char-card.locked { opacity: 0.4; filter: grayscale(1); cursor: not-allowed; }
        .char-symbol { font-family: 'Orbitron', sans-serif; font-size: 1.5rem; font-weight: 900; color: var(--purple); margin-bottom: 2px; }
        .unlocked .char-symbol { color: var(--cyan); }
        .char-name { font-size: 0.68rem; text-align: center; color: var(--text-muted); }
    </style>
</head>
<body>

    <!-- HUD GLOBAL -->
    <header id="global-hud">
        <div class="hud-title">⚡ CIRCUITY RPG: AC & FILTROS PASIVOS</div>
        <div class="hud-controls">
            <button class="btn-hud btn-formulas" onclick="openFormulasModal()">📜 FÓRMULAS</button>
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

    <!-- BOTÓN RECTANGULAR PARA SALIR DE ZOOM -->
    <button class="btn-exit-zoom" id="btn-exit-zoom" onclick="exitWorldZoom()">🔍 SALIR DEL MUNDO</button>

    <!-- ESCENARIO GLOBAL DE MUNDOS -->
    <div id="viewport-stage">
        <div id="world-container">
            
            <!-- OVERLAY SVG CABLE DE COBRE -->
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

            <!-- NODO TIERRA -->
            <div class="terminal-node" id="node-ground">
                <svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="#e5a059" stroke-width="2.5">
                    <line x1="12" y1="3" x2="12" y2="13" />
                    <line x1="4" y1="13" x2="20" y2="13" />
                    <line x1="7" y1="17" x2="17" y2="17" />
                    <line x1="10" y1="21" x2="14" y2="21" />
                </svg>
                <span class="terminal-label">TIERRA</span>
            </div>

            <!-- CAPÍTULO 1 -->
            <div class="chapter-stage unlocked" id="chap-1">
                <div class="chapter-circle unlocked" onclick="zoomIntoWorld('chap-1')">
                    <div class="chapter-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%23081026\'/><path d=\'M 10 100 Q 50 20 100 100 T 190 100\' stroke=\'%2300f3ff\' stroke-width=\'8\' fill=\'none\'/></svg>');"></div>
                    <div class="chapter-overlay-content">
                        <span class="chapter-num">CAPÍTULO 1</span>
                        <h4 class="chapter-name-title">Senoidales & Fasores</h4>
                        <span class="chapter-badge" id="tag-chap-1">ACTIVO</span>
                    </div>
                </div>
                <div class="protoboard-world-container">
                    <div class="protoboard-title">🍞 PROTOBOARD REAL - MUNDO 1</div>
                    <div class="protoboard-pins-grid">
                        <div class="pin-node-card unlocked" id="sub-1-1" onclick="openQuest('1-1')">
                            <div class="pin-icon">🌊</div>
                            <div class="pin-name">1.1 Onda & Fasores</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-1-2" onclick="openQuest('1-2')">
                            <div class="pin-icon">🔢</div>
                            <div class="pin-name">1.2 Complejos & Z</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-1-3" onclick="openQuest('1-3')">
                            <div class="pin-icon">🕸️</div>
                            <div class="pin-name">1.3 Leyes de Kirchhoff</div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- CAPÍTULO 2 -->
            <div class="chapter-stage locked" id="chap-2">
                <div class="chapter-circle locked" onclick="zoomIntoWorld('chap-2')">
                    <div class="chapter-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%231a0b2e\'/><polygon points=\'30,170 170,170 170,30\' stroke=\'%23ff0055\' stroke-width=\'6\' fill=\'none\'/></svg>');"></div>
                    <div class="chapter-overlay-content">
                        <span class="chapter-num">CAPÍTULO 2</span>
                        <h4 class="chapter-name-title">Filtros Pasivos</h4>
                        <span class="chapter-badge" id="tag-chap-2">BLOQUEADO</span>
                    </div>
                </div>
                <div class="protoboard-world-container">
                    <div class="protoboard-title">🍞 PROTOBOARD REAL - MUNDO 2</div>
                    <div class="protoboard-pins-grid">
                        <div class="pin-node-card locked" id="sub-2-1" onclick="openQuest('2-1')">
                            <div class="pin-icon">📉</div>
                            <div class="pin-name">2.1 Filtro Paso Bajo</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-2-2" onclick="openQuest('2-2')">
                            <div class="pin-icon">📈</div>
                            <div class="pin-name">2.2 Filtro Paso Alto</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-2-3" onclick="openQuest('2-3')">
                            <div class="pin-icon">📊</div>
                            <div class="pin-name">2.3 Filtro Paso Banda</div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- CAPÍTULO 3 -->
            <div class="chapter-stage locked" id="chap-3">
                <div class="chapter-circle locked" onclick="zoomIntoWorld('chap-3')">
                    <div class="chapter-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%230b1e10\'/><circle cx=\'70\' cy=\'100\' r=\'40\' stroke=\'%23ffb700\' stroke-width=\'6\' fill=\'none\'/><circle cx=\'130\' cy=\'100\' r=\'40\' stroke=\'%2300ff66\' stroke-width=\'6\' fill=\'none\'/></svg>');"></div>
                    <div class="chapter-overlay-content">
                        <span class="chapter-num">CAPÍTULO 3</span>
                        <h4 class="chapter-name-title">Potencia AC</h4>
                        <span class="chapter-badge" id="tag-chap-3">BLOQUEADO</span>
                    </div>
                </div>
                <div class="protoboard-world-container">
                    <div class="protoboard-title">🍞 PROTOBOARD REAL - MUNDO 3</div>
                    <div class="protoboard-pins-grid">
                        <div class="pin-node-card locked" id="sub-3-1" onclick="openQuest('3-1')">
                            <div class="pin-icon">⚡</div>
                            <div class="pin-name">3.1 Potencia RMS</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-3-2" onclick="openQuest('3-2')">
                            <div class="pin-icon">📐</div>
                            <div class="pin-name">3.2 Triángulo P-Q-S</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-3-3" onclick="openQuest('3-3')">
                            <div class="pin-icon">🏭</div>
                            <div class="pin-name">3.3 Corrección FP</div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- CAPÍTULO 4 -->
            <div class="chapter-stage locked" id="chap-4">
                <div class="chapter-circle locked" onclick="zoomIntoWorld('chap-4')">
                    <div class="chapter-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%23201408\'/><line x1=\'100\' y1=\'100\' x2=\'170\' y2=\'100\' stroke=\'%23ff0055\' stroke-width=\'6\'/><line x1=\'100\' y1=\'100\' x2=\'65\' y2=\'160\' stroke=\'%2300f3ff\' stroke-width=\'6\'/><line x1=\'100\' y1=\'100\' x2=\'65\' y2=\'40\' stroke=\'%2300ff66\' stroke-width=\'6\'/></svg>');"></div>
                    <div class="chapter-overlay-content">
                        <span class="chapter-num">CAPÍTULO 4</span>
                        <h4 class="chapter-name-title">Acoplo Magnético</h4>
                        <span class="chapter-badge" id="tag-chap-4">BLOQUEADO</span>
                    </div>
                </div>
                <div class="protoboard-world-container">
                    <div class="protoboard-title">🍞 PROTOBOARD REAL - MUNDO 4</div>
                    <div class="protoboard-pins-grid">
                        <div class="pin-node-card locked" id="sub-4-1" onclick="openQuest('4-1')">
                            <div class="pin-icon">🧲</div>
                            <div class="pin-name">4.1 Inductancia Mutua</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-4-2" onclick="openQuest('4-2')">
                            <div class="pin-icon">🔴</div>
                            <div class="pin-name">4.2 Regla de Puntos</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-4-3" onclick="openQuest('4-3')">
                            <div class="pin-icon">🔌</div>
                            <div class="pin-name">4.3 Transformador Ideal</div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- CAPÍTULO 5 -->
            <div class="chapter-stage locked" id="chap-5">
                <div class="chapter-circle locked" onclick="zoomIntoWorld('chap-5')">
                    <div class="chapter-cover-img" style="background-image: url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><rect width=\'200\' height=\'200\' fill=\'%231a0b2e\'/><circle cx=\'100\' cy=\'100\' r=\'50\' stroke=\'%2300ff66\' stroke-width=\'6\' fill=\'none\'/></svg>');"></div>
                    <div class="chapter-overlay-content">
                        <span class="chapter-num">CAPÍTULO 5</span>
                        <h4 class="chapter-name-title">Sistemas Trifásicos</h4>
                        <span class="chapter-badge" id="tag-chap-5">BLOQUEADO</span>
                    </div>
                </div>
                <div class="protoboard-world-container">
                    <div class="protoboard-title">🍞 PROTOBOARD REAL - MUNDO 5</div>
                    <div class="protoboard-pins-grid">
                        <div class="pin-node-card locked" id="sub-5-1" onclick="openQuest('5-1')">
                            <div class="pin-icon">🌀</div>
                            <div class="pin-name">5.1 Generador Trifásico</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-5-2" onclick="openQuest('5-2')">
                            <div class="pin-icon">🔺</div>
                            <div class="pin-name">5.2 Conexión Y / Δ</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-5-3" onclick="openQuest('5-3')">
                            <div class="pin-icon">⚖️</div>
                            <div class="pin-name">5.3 Cargas Desequilibradas</div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- FUENTE DE VOLTAJE -->
            <div class="terminal-node" id="node-source">
                <svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="#e5a059" stroke-width="2">
                    <circle cx="12" cy="12" r="9" />
                    <path d="M 7 12 Q 9.5 7 12 12 T 17 12" />
                </svg>
                <span class="terminal-label">FUENTE AC</span>
            </div>

        </div>
    </div>

    <!-- MODAL DE MISIÓN CON ESQUEMA GRANDE ARRIBA Y TEXTO TIPO DIÁLOGO ABAJO -->
    <div class="modal-overlay" id="quest-modal">
        <div class="modal-card">
            <div class="modal-header">
                <h3 id="modal-title">Título de Misión</h3>
                <button class="close-btn" onclick="closeQuest()">&times;</button>
            </div>
            
            <div class="modal-body">
                
                <div class="subtopic-progress-bar">
                    <span style="font-family:'Orbitron'; font-size:0.75rem; color:var(--text-muted);" id="progress-step-text">Paso 1 de 6</span>
                    <div style="display:flex; gap:6px; flex:1; margin:0 12px;">
                        <div class="step-pill" id="pill-0"></div>
                        <div class="step-pill" id="pill-1"></div>
                        <div class="step-pill" id="pill-2"></div>
                        <div class="step-pill" id="pill-3"></div>
                        <div class="step-pill" id="pill-4"></div>
                        <div class="step-pill boss" id="pill-5"></div>
                    </div>
                </div>

                <!-- PASO 0: ESQUEMA GRANDE ARRIBA Y TEXTO DE DIÁLOGO ABAJO -->
                <div class="step-view" id="step-0" style="display:flex; flex-direction:column; gap:12px;">
                    
                    <!-- ESQUEMA CIRCUITAL GRANDE Y CLARO -->
                    <div class="schema-box-large">
                        <div class="schema-header-tag">
                            <span>🔍 ESQUEMA TÉCNICO DEL CIRCUITO</span>
                        </div>
                        <canvas id="schema-canvas" width="700" height="210"></canvas>
                    </div>

                    <!-- FORMULACIÓN MATEMÁTICA Y DEDUCCIÓN -->
                    <div class="math-summary-row">
                        <div class="math-card-compact">
                            <div class="math-title">Ecuación Fundamental de Origen</div>
                            <div class="math-eq" id="eq-1"></div>
                        </div>
                        <div class="math-card-compact">
                            <div class="math-title">Análisis Fasorial Resultante</div>
                            <div class="math-eq" id="eq-2"></div>
                        </div>
                    </div>

                    <!-- SUBVENTANA INFERIOR DE DIÁLOGO DE PERSONAJE -->
                    <div class="rpg-dialog-box" onclick="skipTypewriter()">
                        <div class="rpg-avatar-slot" id="rpg-dialog-avatar">V</div>
                        <div class="rpg-dialog-content">
                            <div class="rpg-speaker-name" id="rpg-dialog-speaker">GUÍA TÉCNICO AC</div>
                            <div class="rpg-text-narrative typing-cursor" id="narrative-text-el">
                                Cargando información de la transmisión...
                            </div>
                        </div>
                    </div>

                    <button class="nav-btn" onclick="nextSubstep()">
                        Continuar al Ejemplo ➔
                    </button>
                </div>

                <!-- PASOS 1 A 5: SIMULADOR, RETO O COMPLETAR ESPACIO EN BLANCO -->
                <div class="step-view" id="step-sim-view" style="display:none; flex-direction:column; gap:12px;">
                    
                    <div class="challenge-box" id="challenge-banner">
                        <div class="challenge-title" id="challenge-title">Modo Ejemplo Guiado</div>
                        <div id="challenge-desc" style="font-size:0.88rem; color:#fff;">Observa el comportamiento automático.</div>
                    </div>

                    <!-- PREGUNTA COMPLETAR ESPACIO EN BLANCO -->
                    <div class="fill-blank-card" id="fill-blank-container" style="display:none;">
                        <div style="font-family:'Orbitron'; color:var(--yellow); font-size:0.85rem;">COMPLETA EL CONCEPTO:</div>
                        <div id="fill-blank-text" style="font-size:0.95rem; color:#fff;"></div>
                        <div style="display:flex; gap:10px; align-items:center;">
                            <input type="text" class="fill-blank-input" id="fill-blank-input" placeholder="Escribe la palabra..." autocomplete="off">
                            <span id="fill-blank-feedback" style="font-size:0.85rem; font-weight:700;"></span>
                        </div>
                    </div>

                    <div class="sim-box" id="sim-canvas-box">
                        <canvas id="sim-canvas" width="680" height="200"></canvas>
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

    <!-- MODAL REVELACIÓN DE PERSONAJE NUEVO -->
    <div class="modal-overlay" id="reveal-modal">
        <div class="reveal-card">
            <div style="font-family: 'Orbitron'; font-size: 0.72rem; color: var(--yellow);">¡NUEVA VARIABLE DESBLOQUEADA!</div>
            <div class="reveal-symbol" id="reveal-symbol">V</div>
            <h3 id="reveal-title" style="color: #fff; font-family: 'Orbitron';">Nombre del Personaje</h3>
            <p id="reveal-desc" style="font-size: 0.85rem; color: var(--text-muted);"></p>
            <button class="nav-btn" onclick="closeReveal()" style="align-self: center;">¡Entendido!</button>
        </div>
    </div>

    <!-- MODAL EQUIPO DE FÓRMULAS -->
    <div class="modal-overlay" id="formulas-modal">
        <div class="modal-card" style="max-width: 800px;">
            <div class="modal-header">
                <h3>📜 EQUIPO DE FÓRMULAS & DERIVACIONES</h3>
                <button class="close-btn" onclick="closeFormulasModal()">&times;</button>
            </div>
            <div class="modal-body">
                <div class="formulas-list-sidebar" id="formulas-list-container"></div>
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
                <div class="codex-grid" id="codex-grid"></div>
                <div class="char-detail-box" id="char-detail-box" style="display:none; background:rgba(157,0,255,0.1); border:1px solid var(--purple); border-radius:8px; padding:12px; margin-top:10px;">
                    <div id="char-detail-name" style="font-family:'Orbitron'; color:var(--yellow); font-size:0.95rem;"></div>
                    <div id="char-detail-desc" style="font-size:0.88rem; margin-top:4px;"></div>
                </div>
            </div>
        </div>
    </div>

    <script>
        /* --- BASE DE DATOS DE FÓRMULAS ESTÁTICAS --- */
        const formulasDB = [
            { title: "Divisor de Tensión -> Filtro RC Paso Bajo", eq: "H(j\\omega) = \\frac{1}{1 + j\\omega RC}", dev: "De $V_{out} = V_{in} \\frac{Z_C}{R + Z_C}$ con $Z_C = \\frac{1}{j\\omega C}$. Frecuencia de corte $\\omega_c = \\frac{1}{RC}$." },
            { title: "Divisor de Tensión -> Filtro RC Paso Alto", eq: "H(j\\omega) = \\frac{j\\omega RC}{1 + j\\omega RC}", dev: "De $V_{out} = V_{in} \\frac{R}{R + Z_C}$ con $Z_C = \\frac{1}{j\\omega C}$. Permite altas frecuencias." },
            { title: "Resonancia RLC Paso Banda", eq: "f_0 = \\frac{1}{2\\pi \\sqrt{LC}}, \\quad B = \\frac{f_0}{Q}", dev: "Las reactancias $X_L = \\omega L$ y $X_C = \\frac{1}{\\omega C}$ se anulan en la frecuencia de resonancia $f_0$." },
            { title: "Fasor de Voltaje Senoidal", eq: "\\mathbf{V} = \\frac{V_m}{\\sqrt{2}} e^{j\\phi}", dev: "Transformación $v(t) = V_m \\cos(\\omega t + \\phi)$ usando Euler. Valor RMS $V_{rms} = V_m / \\sqrt{2}$." },
            { title: "Impedancia Compleja RLC", eq: "\\mathbf{Z} = R + j\\left(\\omega L - \\frac{1}{\\omega C}\\right)", dev: "Suma fasorial de Resistencia real $R$ y Reactancias $X_L = \\omega L$ y $X_C = -\\frac{1}{\\omega C}$." },
            { title: "Potencia Compleja Vectorial", eq: "\\mathbf{S} = P + jQ = \\mathbf{V}_{rms} \\mathbf{I}_{rms}^*", dev: "Producto del voltaje RMS por el conjugado de la corriente RMS. Módulo $|\\mathbf{S}| = \\sqrt{P^2 + Q^2}$ [VA]." },
            { title: "Inductancia Mutua", eq: "M = k \\sqrt{L_1 L_2}", dev: "Grado de acoplamiento magnético $k \\in [0, 1]$ entre dos bobinas que comparten flujo." },
            { title: "Tensión de Línea Trifásica", eq: "V_L = \\sqrt{3} V_p \\angle 30^\\circ", dev: "Diferencia vectorial entre fases desfasadas $120^\\circ$ en conexión Estrella." }
        ];

        /* --- BASE DE DATOS DE PERSONAJES Y SUBTEMAS --- */
        const charactersDB = {
            V: { symbol: 'V', name: 'Voltaje (Tensión)', desc: 'Fuerza electromotriz senoidal que impulsa los electrones.', introKey: '1-1' },
            I: { symbol: 'I', name: 'Corriente (Intensidad)', desc: 'Flujo eléctrico que oscila en el tiempo con desfase.', introKey: '1-1' },
            w: { symbol: 'ω', name: 'Frecuencia Angular', desc: 'Velocidad angular de rotación fasorial (ω = 2πf).', introKey: '1-1' },
            Z: { symbol: 'Z', name: 'Impedancia Compleja', desc: 'Oposición total en AC (Resistencia + Reactancia).', introKey: '1-2' },
            K: { symbol: 'K', name: 'Leyes de Kirchhoff', desc: 'Mantiene conservación de carga con matrices fasoriales.', introKey: '1-3' },
            H: { symbol: 'H(jω)', name: 'Función Transferencia', desc: 'Relación Vout/Vin en magnitud y fase de un filtro.', introKey: '2-1' },
            P: { symbol: 'P', name: 'Potencia Activa', desc: 'Energía promedio convertida en trabajo real (Watts).', introKey: '3-1' },
            S: { symbol: 'S', name: 'Potencia Aparente', desc: 'Capacidad total suministrada por la red (VA).', introKey: '3-2' },
            FP: { symbol: 'FP', name: 'Factor de Potencia', desc: 'Razón cos(θ) de eficiencia energética.', introKey: '3-3' },
            M: { symbol: 'M', name: 'Inductancia Mutua', desc: 'Tensión inducida magnéticamente entre bobinas.', introKey: '4-1' },
            VL: { symbol: 'VL', name: 'Tensión Trifásica', desc: 'Voltaje de línea en sistemas polifásicos (√3 · VP).', introKey: '5-1' }
        };

        const questsDB = {
            '1-1': {
                title: 'Misión 1.1: Onda Senoidal y Fasores', chap: 'chap-1', next: '1-2', introChar: 'V',
                speaker: 'Fasor Voltaje V',
                narrative: 'Una fuente AC produce una onda senoidal $v(t) = V_m \\cos(\\omega t + \\phi)$. Para operarla matemáticamente sin derivadas complicadas, la transformamos al **Fasor** complejo $\\mathbf{V} = V_{rms} \\angle \\phi$.',
                eq1: 'v(t) = V_m \\cdot \\cos(\\omega t + \\phi)', eq2: '\\mathbf{V} = \\frac{V_m}{\\sqrt{2}} e^{j\\phi} = V_{rms} \\angle \\phi',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Fasor V:</b> Vector de magnitud y fase.</div><div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Onda Senoidal v(t):</b> Dominio del tiempo.</div>',
                controls: `<div class="control-group"><label>Amplitud Vm: <span id="val-vm">100</span> V</label><input type="range" id="input-vm" min="30" max="150" value="100" oninput="drawSimSenoidal()"></div><div class="control-group"><label>Desfase ϕ: <span id="val-phi">45</span>°</label><input type="range" id="input-phi" min="-180" max="180" value="45" oninput="drawSimSenoidal()"></div>`,
                drawSchema: (ctx) => drawSchemaGen(ctx), drawSim: () => drawSimSenoidal(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Observa la equivalencia entre el vector fasorial y la onda de tiempo." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "El valor equivalente en DC de una senoidal es el valor ________.", answer: "rms" },
                    { title: "Ejercicio 2: Ajuste Vm", type: "sim", desc: "Ajusta la Amplitud Vm exactamente a 120 V.", check: () => parseFloat(document.getElementById('input-vm').value) === 120 },
                    { title: "Ejercicio 3: Desfase", type: "sim", desc: "Coloca el desfase en +90°.", check: () => parseFloat(document.getElementById('input-phi').value) === 90 },
                    { title: "👾 DESAFÍO BOSS: Generar Vrms = 100V", type: "sim", desc: "Fija Vm = 141 V y desfase a +45°.", check: () => parseFloat(document.getElementById('input-vm').value) === 141 && parseFloat(document.getElementById('input-phi').value) === 45 }
                ]
            },
            '1-2': {
                title: 'Misión 1.2: Números Complejos e Impedancia Z', chap: 'chap-1', next: '1-3', introChar: 'Z',
                speaker: 'Impedancia Z',
                narrative: 'La **Impedancia Compleja** $\\mathbf{Z} = R + jX$ se representa mediante cajas de bloques en el circuito. $j = \\sqrt{-1}$ rota vectores $90^\\circ$ en el plano complejo.',
                eq1: '\\mathbf{Z} = R + j\\left(\\omega L - \\frac{1}{\\omega C}\\right)', eq2: '|\\mathbf{Z}| = \\sqrt{R^2 + (X_L - X_C)^2}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Magnitud |Z|:</b> Oposición total al flujo AC.</div>',
                controls: `<div class="control-group"><label>Inductancia L: <span id="val-l">10</span> mH</label><input type="range" id="input-l" min="1" max="50" value="10" oninput="drawSimRLC()"></div><div class="control-group"><label>Capacitancia C: <span id="val-c">10</span> µF</label><input type="range" id="input-c" min="1" max="50" value="10" oninput="drawSimRLC()"></div>`,
                drawSchema: (ctx) => drawSchemaRLC(ctx), drawSim: () => drawSimRLC(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Variación de la impedancia total RLC." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "La reactancia de un condensador disminuye cuando aumenta la ________.", answer: "frecuencia" },
                    { title: "Ejercicio 2: Ajuste L", type: "sim", desc: "Fija L = 25 mH.", check: () => parseFloat(document.getElementById('input-l').value) === 25 },
                    { title: "Ejercicio 3: Ajuste C", type: "sim", desc: "Ajusta C = 40 µF.", check: () => parseFloat(document.getElementById('input-c').value) === 40 },
                    { title: "👾 DESAFÍO BOSS: Sintonía L = C = 20", type: "sim", desc: "Iguala L = 20 mH y C = 20 µF.", check: () => parseFloat(document.getElementById('input-l').value) === 20 && parseFloat(document.getElementById('input-c').value) === 20 }
                ]
            },
            '1-3': {
                title: 'Misión 1.3: Leyes de Kirchhoff en AC', chap: 'chap-1', next: '2-1', introChar: 'K',
                speaker: 'Maestro Kirchhoff',
                narrative: 'Analizamos un circuito completo de dos mallas. En el nodo principal, la suma de corrientes es nula: $\\mathbf{I}_1 = \\mathbf{I}_2 + \\mathbf{I}_3$, resolviendo mediante la matriz de admitancias $[\\mathbf{Y}][\\mathbf{V}] = [\\mathbf{I}]$.',
                eq1: '\\sum \\mathbf{I}_{nodo} = 0 \\implies \\mathbf{I}_1 - \\mathbf{I}_2 - \\mathbf{I}_3 = 0', eq2: '[\\mathbf{Y}] \\cdot [\\mathbf{V}] = [\\mathbf{I}], \\quad \\mathbf{V}_1 = \\frac{\\mathbf{I}_s}{\\mathbf{Y}_{total}}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Fasores de Corriente:</b> Conservación de carga nodal.</div>',
                controls: `<div class="control-group"><label>Impedancia Z2: <span id="val-z2">20</span> Ω</label><input type="range" id="input-z2" min="5" max="50" value="20" oninput="drawSimKCL()"></div>`,
                drawSchema: (ctx) => drawSchemaKCL(ctx), drawSim: () => drawSimKCL(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Matriz de admitancia nodal en tiempo real." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "La suma de corrientes que entran a un nodo es igual a cero según la ley de ________.", answer: "kirchhoff" },
                    { title: "Ejercicio 2: Impedancia Z2", type: "sim", desc: "Ajusta Z2 = 15 Ω.", check: () => parseFloat(document.getElementById('input-z2').value) === 15 },
                    { title: "Ejercicio 3: Incremento Z2", type: "sim", desc: "Aumenta Z2 = 35 Ω.", check: () => parseFloat(document.getElementById('input-z2').value) === 35 },
                    { title: "👾 DESAFÍO BOSS: Equilibrar Nodal Z2 = 10Ω", type: "sim", desc: "Ajusta Z2 a 10 Ω exactamente.", check: () => parseFloat(document.getElementById('input-z2').value) === 10 }
                ]
            },
            '2-1': {
                title: 'Misión 2.1: Filtro Paso Bajo RC (LPF)', chap: 'chap-2', next: '2-2', introChar: 'H',
                speaker: 'Filtro H(jω)',
                narrative: 'Utilizando el divisor de tensión sobre la caja de impedancia capacitiva $\\mathbf{Z}_C$, obtenemos la función de transferencia $H(j\\omega) = \\frac{1}{1 + j\\omega RC}$. Atenúa las altas frecuencias.',
                eq1: 'H(j\\omega) = \\frac{\\mathbf{V}_{out}}{\\mathbf{V}_{in}} = \\frac{\\mathbf{Z}_C}{R + \\mathbf{Z}_C} = \\frac{1}{1 + j\\omega RC}', eq2: 'f_c = \\frac{1}{2\\pi RC} \\implies |H(j f_c)| = \\frac{1}{\\sqrt{2}} \\approx -3\\text{ dB}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Respuesta LPF:</b> Caída de -20 dB/década.</div>',
                controls: `<div class="control-group"><label>Resistencia R: <span id="val-r">1000</span> Ω</label><input type="range" id="input-r" min="100" max="5000" step="100" value="1000" oninput="drawSimLPF()"></div><div class="control-group"><label>Capacitancia C: <span id="val-c">0.1</span> µF</label><input type="range" id="input-c" min="0.01" max="1" step="0.01" value="0.1" oninput="drawSimLPF()"></div>`,
                drawSchema: (ctx) => drawSchemaLPF(ctx), drawSim: () => drawSimLPF(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Respuesta en frecuencia LPF." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "En la frecuencia de corte la potencia de salida cae a la ________.", answer: "mitad" },
                    { title: "Ejercicio 2: Resistencia R", type: "sim", desc: "Fija R = 2000 Ω.", check: () => parseFloat(document.getElementById('input-r').value) === 2000 },
                    { title: "Ejercicio 3: Capacitancia C", type: "sim", desc: "Ajusta C = 0.5 µF.", check: () => Math.abs(parseFloat(document.getElementById('input-c').value) - 0.5) < 0.01 },
                    { title: "👾 DESAFÍO BOSS: Frecuencia fc ≈ 1591 Hz", type: "sim", desc: "Ajusta R = 1000 Ω y C = 0.1 µF.", check: () => parseFloat(document.getElementById('input-r').value) === 1000 && Math.abs(parseFloat(document.getElementById('input-c').value) - 0.1) < 0.01 }
                ]
            },
            '2-2': {
                title: 'Misión 2.2: Filtro Paso Alto RC (HPF)', chap: 'chap-2', next: '2-3', introChar: 'H',
                speaker: 'Filtro H(jω)',
                narrative: 'Tomando la salida sobre la resistencia $\\mathbf{Z}_R$, bloqueamos la corriente continua (DC). La señal pasa cuando $\\omega \\gg \\omega_c$.',
                eq1: 'H(j\\omega) = \\frac{\\mathbf{Z}_R}{\\mathbf{Z}_R + \\mathbf{Z}_C} = \\frac{j\\omega RC}{1 + j\\omega RC}', eq2: 'f_c = \\frac{1}{2\\pi RC}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Respuesta HPF:</b> Pasa altas frecuencias.</div>',
                controls: `<div class="control-group"><label>Resistencia R: <span id="val-rh">1000</span> Ω</label><input type="range" id="input-rh" min="100" max="5000" step="100" value="1000" oninput="drawSimHPF()"></div><div class="control-group"><label>Capacitancia C: <span id="val-ch">0.1</span> µF</label><input type="range" id="input-ch" min="0.01" max="1" step="0.01" value="0.1" oninput="drawSimHPF()"></div>`,
                drawSchema: (ctx) => drawSchemaHPF(ctx), drawSim: () => drawSimHPF(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Respuesta en frecuencia HPF." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "A frecuencia cero (DC), el condensador actúa como un circuito ________.", answer: "abierto" },
                    { title: "Ejercicio 2: Ajuste R", type: "sim", desc: "Fija R = 1500 Ω.", check: () => parseFloat(document.getElementById('input-rh').value) === 1500 },
                    { title: "Ejercicio 3: Ajuste C", type: "sim", desc: "Ajusta C = 0.05 µF.", check: () => Math.abs(parseFloat(document.getElementById('input-ch').value) - 0.05) < 0.005 },
                    { title: "👾 DESAFÍO BOSS: Frecuencia fc ≈ 318 Hz", type: "sim", desc: "Configura R = 5000 Ω y C = 0.1 µF.", check: () => parseFloat(document.getElementById('input-rh').value) === 5000 && Math.abs(parseFloat(document.getElementById('input-ch').value) - 0.1) < 0.01 }
                ]
            },
            '2-3': {
                title: 'Misión 2.3: Filtro Paso Banda RLC', chap: 'chap-2', next: '3-1', introChar: 'H',
                speaker: 'Filtro H(jω)',
                narrative: 'Unimos las cajas de impedancia inductiva y capacitiva en serie. En la frecuencia de resonancia $f_0$, $X_L = X_C$, permitiendo la máxima transferencia de potencia.',
                eq1: 'f_0 = \\frac{1}{2\\pi \\sqrt{LC}}', eq2: 'B = \\frac{f_0}{Q} = \\frac{R}{2\\pi L}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Paso Banda:</b> Máximo pico en resonancia f0.</div>',
                controls: `<div class="control-group"><label>Inductancia L: <span id="val-l2">10</span> mH</label><input type="range" id="input-l2" min="1" max="50" value="10" oninput="drawSimRLC()"></div><div class="control-group"><label>Capacitancia C: <span id="val-c2">10</span> µF</label><input type="range" id="input-c2" min="1" max="50" value="10" oninput="drawSimRLC()"></div>`,
                drawSchema: (ctx) => drawSchemaRLC(ctx), drawSim: () => drawSimRLC(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Respuesta Paso Banda RLC." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "El factor de calidad Q mide la selectividad del filtro en la frecuencia de ________.", answer: "resonancia" },
                    { title: "Ejercicio 2: Ajuste L", type: "sim", desc: "Fija L = 10 mH.", check: () => parseFloat(document.getElementById('input-l2').value) === 10 },
                    { title: "Ejercicio 3: Ajuste C", type: "sim", desc: "Ajusta C = 10 µF.", check: () => parseFloat(document.getElementById('input-c2').value) === 10 },
                    { title: "👾 DESAFÍO BOSS: Resonancia f0 ≈ 1000 Hz", type: "sim", desc: "Ajusta L = 5 mH y C = 5 µF.", check: () => parseFloat(document.getElementById('input-l2').value) === 5 && parseFloat(document.getElementById('input-c2').value) === 5 }
                ]
            },
            '3-1': {
                title: 'Misión 3.1: Potencia Instantánea y RMS', chap: 'chap-3', next: '3-2', introChar: 'P',
                speaker: 'Potencia P',
                narrative: 'La potencia instantánea oscila al doble de frecuencia. El valor eficaz $V_{rms} = V_m / \\sqrt{2}$ determina el trabajo real disipado en las impedancias resistivas.',
                eq1: 'p(t) = v(t) \\cdot i(t)', eq2: 'P_{prom} = V_{rms} I_{rms} \\cos\\theta',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Potencia Promedio P:</b> Trabajo útil real.</div>',
                controls: `<div class="control-group"><label>Voltaje Vrms: <span id="val-vrms">120</span> V</label><input type="range" id="input-vrms" min="60" max="240" value="120" oninput="drawSimPinst()"></div>`,
                drawSchema: (ctx) => drawSchemaPinst(ctx), drawSim: () => drawSimPinst(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Variación de la potencia promedio." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "La sigla RMS en español significa Raíz Cuadrada ________.", answer: "media" },
                    { title: "Ejercicio 2: Vrms Residencial", type: "sim", desc: "Establece Vrms = 110 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 110 },
                    { title: "Ejercicio 3: Vrms Industrial", type: "sim", desc: "Sube a Vrms = 220 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 220 },
                    { title: "👾 DESAFÍO BOSS: Potencia P_prom = 360W", type: "sim", desc: "Ajusta Vrms a 240 V para lograr P = 360W.", check: () => parseFloat(document.getElementById('input-vrms').value) === 240 }
                ]
            },
            '3-2': {
                title: 'Misión 3.2: Triángulo de Potencia Vectorial', chap: 'chap-3', next: '3-3', introChar: 'S',
                narrative: 'Unimos la Potencia Activa $P$ (Watts) y la Reactiva $Q$ (VAR) en la Potencia Aparente Compleja $\\mathbf{S} = P + jQ$ (VA).',
                eq1: '\\mathbf{S} = P + jQ', eq2: '|\\mathbf{S}| = \\sqrt{P^2 + Q^2}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Vector S:</b> Potencia aparente entregada.</div>',
                controls: `<div class="control-group"><label>Reactiva QL: <span id="val-ql">250</span> VAR</label><input type="range" id="input-ql" min="50" max="450" value="250" oninput="drawSimTriang()"></div>`,
                drawSchema: (ctx) => drawSchemaTriang(ctx), drawSim: () => drawSimTriang(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Crecimiento del vector aparente S." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "La potencia reactiva inductiva se mide en la unidad de ________.", answer: "var" },
                    { title: "Ejercicio 2: Reactiva Baja", type: "sim", desc: "Fija QL = 100 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 100 },
                    { title: "Ejercicio 3: Reactiva Alta", type: "sim", desc: "Sube QL = 300 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 300 },
                    { title: "👾 DESAFÍO BOSS: Triángulo Isósceles P = Q", type: "sim", desc: "Como P = 300 W, ajusta QL exactamente a 300 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 300 }
                ]
            },
            '3-3': {
                title: 'Misión 3.3: Corrección del Factor de Potencia', chap: 'chap-3', next: '4-1', introChar: 'FP',
                narrative: 'Inyectamos potencia reactiva capacitiva $-j Q_C$ mediante bancos de capacitores en paralelo para llevar $FP = \\cos\\theta \\ge 0.95$.',
                eq1: 'FP = \\frac{P}{|\\mathbf{S}|} = \\cos(\\theta)', eq2: 'Q_C = P (\\tan\\theta_1 - \\tan\\theta_2)',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Triángulo Compensado:</b> Reducción de S.</div>',
                controls: `<div class="control-group"><label>Inyección QC: <span id="val-qc">150</span> VAR</label><input type="range" id="input-qc" min="0" max="350" value="150" oninput="drawSimFP()"></div>`,
                drawSchema: (ctx) => drawSchemaFP(ctx), drawSim: () => drawSimFP(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Compensación de reactiva." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "Para corregir un factor de potencia atrasado se conectan capacitores en ________.", answer: "paralelo" },
                    { title: "Ejercicio 2: Inyección QC", type: "sim", desc: "Ajusta QC = 100 VAR.", check: () => parseFloat(document.getElementById('input-qc').value) === 100 },
                    { title: "Ejercicio 3: Inyección Alta", type: "sim", desc: "Sube QC = 200 VAR.", check: () => parseFloat(document.getElementById('input-qc').value) === 200 },
                    { title: "👾 DESAFÍO BOSS: Lograr FP ≥ 0.95", type: "sim", desc: "Ajusta QC = 210 VAR para lograr FP = 0.96.", check: () => parseFloat(document.getElementById('input-qc').value) === 210 }
                ]
            },
            '4-1': {
                title: 'Misión 4.1: Inductancia Mutua M', chap: 'chap-4', next: '4-2', introChar: 'M',
                speaker: 'Inductancia Mutua M',
                narrative: 'Dos bobinas acopladas magnéticamente comparten flujo, induciendo una tensión mutua $M = k \\sqrt{L_1 L_2}$.',
                eq1: 'v_2(t) = M \\frac{di_1}{dt}', eq2: 'M = k \\sqrt{L_1 L_2}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Flujo k:</b> Coeficiente magnético.</div>',
                controls: `<div class="control-group"><label>Coeficiente k: <span id="val-k">0.7</span></label><input type="range" id="input-k" min="0.1" max="1" step="0.05" value="0.7" oninput="drawSimMutua()"></div>`,
                drawSchema: (ctx) => drawSchemaMutua(ctx), drawSim: () => drawSimMutua(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Variación del acoplamiento." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "El coeficiente k de acoplamiento perfecto ideal es exactamente ____.", answer: "1" },
                    { title: "Ejercicio 2: Acoplo Débil", type: "sim", desc: "Ajusta k = 0.3.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 0.3) < 0.01 },
                    { title: "Ejercicio 3: Acoplo Medio", type: "sim", desc: "Ajusta k = 0.6.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 0.6) < 0.01 },
                    { title: "👾 DESAFÍO BOSS: Acoplo Perfecto", type: "sim", desc: "Fija k = 1.0.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 1.0) < 0.01 }
                ]
            },
            '4-2': {
                title: 'Misión 4.2: Regla de Puntos en Bobinas', chap: 'chap-4', next: '4-3', introChar: 'M',
                speaker: 'Inductancia Mutua M',
                narrative: 'Los puntos polares en el diagrama indican la polaridad relativa de la tensión inducida mutuamente.',
                eq1: '\\mathbf{V}_1 = j\\omega L_1 \\mathbf{I}_1 \\pm j\\omega M \\mathbf{I}_2', eq2: '\\mathbf{V}_2 = j\\omega L_2 \\mathbf{I}_2 \\pm j\\omega M \\mathbf{I}_1',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Puntos Polares:</b> Referencias.</div>',
                controls: `<div class="control-group"><label>Dirección I2: <span id="val-dir">Mismo Punto</span></label><input type="range" id="input-dir" min="0" max="1" step="1" value="1" oninput="drawSimDots()"></div>`,
                drawSchema: (ctx) => drawSchemaDots(ctx), drawSim: () => drawSimDots(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Inversión de puntos." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "Si la corriente entra por el punto, la tensión inducida en el otro punto es ________.", answer: "positiva" },
                    { title: "Ejercicio 2: Punto Opuesto", type: "sim", desc: "Coloca la corriente en Punto Opuesto (0).", check: () => parseInt(document.getElementById('input-dir').value) === 0 },
                    { title: "Ejercicio 3: Mismo Punto", type: "sim", desc: "Cambia a Mismo Punto (1).", check: () => parseInt(document.getElementById('input-dir').value) === 1 },
                    { title: "👾 DESAFÍO BOSS: Configuración Sumativa (+M)", type: "sim", desc: "Establece la adición de flujo (1).", check: () => parseInt(document.getElementById('input-dir').value) === 1 }
                ]
            },
            '4-3': {
                title: 'Misión 4.3: Transformador Ideal', chap: 'chap-4', next: '5-1', introChar: 'M',
                speaker: 'Inductancia Mutua M',
                narrative: 'Escala tensiones y corrientes según la relación de vueltas $a = N_1 / N_2$, reflejando impedancias como $\\mathbf{Z}_{in} = a^2 \\mathbf{Z}_L$.',
                eq1: 'a = \\frac{N_1}{N_2} = \\frac{\\mathbf{V}_1}{\\mathbf{V}_2}', eq2: '\\mathbf{Z}_{in} = a^2 \\mathbf{Z}_L',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>V2 Salida:</b> Escalado.</div>',
                controls: `<div class="control-group"><label>Razón a: <span id="val-a">2.0</span></label><input type="range" id="input-a" min="0.5" max="4" step="0.5" value="2" oninput="drawSimTransfo()"></div>`,
                drawSchema: (ctx) => drawSchemaTransfo(ctx), drawSim: () => drawSimTransfo(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Transformación de voltaje." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "Un transformador ideal no consume potencia activa porque no tiene ________.", answer: "perdidas" },
                    { title: "Ejercicio 2: Reductor", type: "sim", desc: "Ajusta reductor a = 2.0.", check: () => parseFloat(document.getElementById('input-a').value) === 2.0 },
                    { title: "Ejercicio 3: Elevador", type: "sim", desc: "Ajusta elevador a = 0.5.", check: () => parseFloat(document.getElementById('input-a').value) === 0.5 },
                    { title: "👾 DESAFÍO BOSS: Obtener V2 = 30 V", type: "sim", desc: "Ajusta a = 4.0 para reducir 120V a 30V.", check: () => parseFloat(document.getElementById('input-a').value) === 4.0 }
                ]
            },
            '5-1': {
                title: 'Misión 5.1: Generador Trifásico y Secuencias', chap: 'chap-5', next: '5-2', introChar: 'VL',
                speaker: 'Tensión de Línea VL',
                narrative: 'Tres ondas senoidales simétricas desfasadas $120^\circ$. El voltaje entre dos líneas activas es $V_L = \\sqrt{3} V_p$.',
                eq1: '\\mathbf{V}_{aN} = V_p \\angle 0^\\circ, \\quad \\mathbf{V}_{bN} = V_p \\angle -120^\\circ', eq2: 'V_L = \\sqrt{3} V_p \\approx 1.732 \\cdot V_p',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Fases ABC:</b> Desfasadas 120°.</div>',
                controls: `<div class="control-group"><label>Voltaje Vp: <span id="val-vp">120</span> V</label><input type="range" id="input-vp" min="60" max="240" value="120" oninput="drawSimTriGen()"></div>`,
                drawSchema: (ctx) => drawSchemaTriGen(ctx), drawSim: () => drawSimTriGen(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Rotación simétrica trifásica." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "El ángulo de desfase simétrico entre las fases trifásicas es de ____ grados.", answer: "120" },
                    { title: "Ejercicio 2: Ajuste Vp", type: "sim", desc: "Ajusta Vp = 100 V.", check: () => parseFloat(document.getElementById('input-vp').value) === 100 },
                    { title: "Ejercicio 3: Incremento Vp", type: "sim", desc: "Sube Vp = 200 V.", check: () => parseFloat(document.getElementById('input-vp').value) === 200 },
                    { title: "👾 DESAFÍO BOSS: Tensión VL ≈ 208 V", type: "sim", desc: "Ajusta Vp = 120 V (VL = 120·√3 = 207.8 V).", check: () => parseFloat(document.getElementById('input-vp').value) === 120 }
                ]
            },
            '5-2': {
                title: 'Misión 5.2: Conexión Estrella Y / Delta Δ', chap: 'chap-5', next: '5-3', introChar: 'VL',
                speaker: 'Tensión de Línea VL',
                narrative: 'En conexión Delta ($\Delta$), $V_L = V_p$, pero la corriente de línea se multiplica por $I_L = \\sqrt{3} I_p$.',
                eq1: 'Y: V_L = \\sqrt{3} V_p', eq2: '\\Delta: I_L = \\sqrt{3} I_p',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Multiplicador:</b> Factor √3.</div>',
                controls: `<div class="control-group"><label>Corriente Ip: <span id="val-ip">10</span> A</label><input type="range" id="input-ip" min="2" max="25" value="10" oninput="drawSimYDelta()"></div>`,
                drawSchema: (ctx) => drawSchemaYDelta(ctx), drawSim: () => drawSimYDelta(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Relación de corrientes en Delta." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "En una conexión Estrella equilibrada, el cable neutro lleva corriente de valor ________.", answer: "cero" },
                    { title: "Ejercicio 2: Ajuste Ip", type: "sim", desc: "Ajusta Ip = 5 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 5 },
                    { title: "Ejercicio 3: Incremento Ip", type: "sim", desc: "Sube Ip = 15 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 15 },
                    { title: "👾 DESAFÍO BOSS: Corriente IL ≈ 17.3 A", type: "sim", desc: "Ajusta Ip = 10 A para obtener IL = 17.32 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 10 }
                ]
            },
            '5-3': {
                title: 'Misión 5.3: Cargas Trifásicas Desequilibradas', chap: 'chap-5', next: null, introChar: 'VL',
                speaker: 'Tensión de Línea VL',
                narrative: 'Cuando las cajas de impedancia de fase son desiguales, la suma de corrientes no se anula, generando una corriente de retorno por el neutro $\\mathbf{I}_N \\neq 0$.',
                eq1: '\\mathbf{I}_N = \\mathbf{I}_A + \\mathbf{I}_B + \\mathbf{I}_C', eq2: 'P_{total} = P_A + P_B + P_C',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Corriente IN:</b> Retorno por el neutro.</div>',
                controls: `<div class="control-group"><label>Desequilibrio A: <span id="val-des">60</span>%</label><input type="range" id="input-des" min="0" max="100" value="60" oninput="drawSimDeseb()"></div>`,
                drawSchema: (ctx) => drawSchemaDeseb(ctx), drawSim: () => drawSimDeseb(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Corriente por el neutro." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "El desequilibrio de cargas causa una corriente de retorno a través del cable de ________.", answer: "neutro" },
                    { title: "Ejercicio 2: Desequilibrio 20%", type: "sim", desc: "Ajusta desequilibrio al 20%.", check: () => parseFloat(document.getElementById('input-des').value) === 20 },
                    { title: "Ejercicio 3: Desequilibrio 50%", type: "sim", desc: "Sube desequilibrio al 50%.", check: () => parseFloat(document.getElementById('input-des').value) === 50 },
                    { title: "👾 DESAFÍO BOSS: Sobrecarga Total de Neutro", type: "sim", desc: "Lleve el desequilibrio al 100%.", check: () => parseFloat(document.getElementById('input-des').value) === 100 }
                ]
            }
        };

        /* --- ESTADO Y NAVEGACIÓN --- */
        const gameState = {
            xp: 0, level: 1,
            unlockedChaps: ['chap-1'],
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
            initFormulasList();
        };

        function updateUI() {
            document.getElementById('player-lvl').innerText = `Nivel ${gameState.level}`;
            const ranks = ["Novato AC", "Analista de Filtros", "Maestro de Impedancias", "Soberano Trifásico"];
            document.getElementById('player-rank').innerText = ranks[Math.min(gameState.level - 1, ranks.length - 1)];
            document.getElementById('xp-bar').style.width = `${Math.min((gameState.xp / (gameState.level * 200)) * 100, 100)}%`;

            ['chap-1', 'chap-2', 'chap-3', 'chap-4', 'chap-5'].forEach((cId) => {
                const el = document.getElementById(cId);
                const isUnlocked = gameState.unlockedChaps.includes(cId);
                const circle = el.querySelector('.chapter-circle');
                const tag = document.getElementById(`tag-${cId}`);

                if (isUnlocked) {
                    el.classList.remove('locked'); el.classList.add('unlocked');
                    circle.className = "chapter-circle unlocked"; tag.innerText = "ACTIVO";
                } else {
                    el.classList.add('locked');
                    circle.className = "chapter-circle locked"; tag.innerText = "BLOQUEADO 🔒";
                }
            });

            Object.keys(questsDB).forEach(qKey => {
                const btn = document.getElementById(`sub-${qKey}`);
                if (!btn) return;

                if (gameState.completedQuests.includes(qKey)) {
                    btn.className = "pin-node-card completed";
                } else if (gameState.unlockedQuests.includes(qKey)) {
                    btn.className = "pin-node-card unlocked";
                } else {
                    btn.className = "pin-node-card locked";
                }
            });

            updateCablesAndElectron();
        }

        /* --- ZOOM EXTREMO CENTRADO EN PROTOBOARD REAL --- */
        function zoomIntoWorld(chapId) {
            if (!gameState.unlockedChaps.includes(chapId)) return;
            if (currentZoomWorld === chapId) return;

            currentZoomWorld = chapId;
            const container = document.getElementById('world-container');
            const targetEl = document.getElementById(chapId);

            targetEl.classList.add('active-zoom');
            document.getElementById('global-hud').classList.add('hidden-hud');
            document.getElementById('btn-exit-zoom').style.display = 'block';

            ['chap-1', 'chap-2', 'chap-3', 'chap-4', 'chap-5'].forEach(id => {
                if (id !== chapId) document.getElementById(id).style.opacity = '0.05';
            });
            document.getElementById('node-ground').style.opacity = '0.05';
            document.getElementById('node-source').style.opacity = '0.05';

            const rect = targetEl.getBoundingClientRect();
            const containerRect = container.getBoundingClientRect();
            const offsetY = (containerRect.height / 2) - (targetEl.offsetTop + rect.height / 2);

            container.style.transform = `translateY(${offsetY}px) scale(1.45)`;

            setTimeout(updateCablesAndElectron, 400);
        }

        function exitWorldZoom() {
            currentZoomWorld = null;
            const container = document.getElementById('world-container');
            container.style.transform = 'translateY(0) scale(1)';

            document.getElementById('global-hud').classList.remove('hidden-hud');
            document.getElementById('btn-exit-zoom').style.display = 'none';

            ['chap-1', 'chap-2', 'chap-3', 'chap-4', 'chap-5'].forEach(id => {
                const el = document.getElementById(id);
                el.classList.remove('active-zoom');
                el.style.opacity = '1';
            });
            document.getElementById('node-ground').style.opacity = '1';
            document.getElementById('node-source').style.opacity = '1';

            setTimeout(updateCablesAndElectron, 400);
        }

        /* --- CABLES Y ELECTRÓN --- */
        function updateCablesAndElectron() {
            const container = document.getElementById('world-container');
            const svg = document.getElementById('cable-svg');
            const rectW = container.getBoundingClientRect();

            svg.setAttribute('width', rectW.width);
            svg.setAttribute('height', rectW.height);

            let pathD = "";

            if (currentZoomWorld) {
                const activeEl = document.getElementById(currentZoomWorld);
                const subCards = activeEl.querySelectorAll('.pin-node-card');

                if (subCards.length > 0) {
                    subCards.forEach((card, idx) => {
                        const sR = card.getBoundingClientRect();
                        const sX = sR.left + sR.width/2 - rectW.left;
                        const sY = sR.top + sR.height/2 - rectW.top;

                        if (idx === 0) pathD = `M ${sX} ${sY} `;
                        else {
                            const prevR = subCards[idx-1].getBoundingClientRect();
                            const pX = prevR.left + prevR.width/2 - rectW.left;
                            const pY = prevR.top + prevR.height/2 - rectW.top;
                            const midX = (pX + sX) / 2;
                            pathD += `C ${midX} ${pY}, ${midX} ${sY}, ${sX} ${sY} `;
                        }
                    });
                }
            } else {
                const nodes = ['node-ground', 'chap-1', 'chap-2', 'chap-3', 'chap-4', 'chap-5', 'node-source'];
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
                    const curveOffset = (i % 2 === 0) ? 130 : -130;

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

        /* --- APERTURA DE MISIÓN CON SUBVENTANA INFERIOR DE DIÁLOGO --- */
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

            // Configurar cuadro de diálogo estilo RPG
            const charObj = charactersDB[q.introChar] || { symbol: 'V', name: 'Guía Técnico AC' };
            document.getElementById('rpg-dialog-avatar').innerText = charObj.symbol;
            document.getElementById('rpg-dialog-speaker').innerText = q.speaker || charObj.name;

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
                const fillCard = document.getElementById('fill-blank-container');
                const simBox = document.getElementById('sim-canvas-box');

                banner.className = "challenge-box" + (activeSubstep === 5 ? " boss" : "");
                document.getElementById('challenge-title').innerText = challenge.title;
                document.getElementById('challenge-desc').innerText = challenge.desc;

                if (challenge.type === "fill_blank") {
                    fillCard.style.display = 'flex';
                    simBox.style.display = 'none';
                    document.getElementById('fill-blank-text').innerText = challenge.desc;
                    document.getElementById('fill-blank-input').value = "";
                    document.getElementById('fill-blank-feedback').innerText = "";
                } else {
                    fillCard.style.display = 'none';
                    simBox.style.display = 'flex';
                }

                const btnNext = document.getElementById('btn-next-step');
                btnNext.className = "nav-btn" + (activeSubstep === 5 ? " boss-btn" : "");
                btnNext.innerText = activeSubstep === 5 ? "Completar Misión 👾" : "Continuar ➔";

                setTimeout(() => {
                    if (challenge.type !== "fill_blank") {
                        q.drawSim();
                        renderMathInLegend();
                    }

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

            if (activeSubstep >= 1 && activeSubstep <= 5) {
                const challenge = q.challenges[activeSubstep - 1];

                if (challenge.type === "fill_blank") {
                    const userAns = document.getElementById('fill-blank-input').value.trim().toLowerCase();
                    if (userAns !== challenge.answer.toLowerCase()) {
                        document.getElementById('fill-blank-feedback').style.color = 'var(--red)';
                        document.getElementById('fill-blank-feedback').innerText = '❌ Incorrecto. Revisa el concepto.';
                        return;
                    } else {
                        document.getElementById('fill-blank-feedback').style.color = 'var(--green)';
                        document.getElementById('fill-blank-feedback').innerText = '✔ ¡Correcto!';
                    }
                } else if (challenge.check && !challenge.check()) {
                    alert("⚠️ ¡El simulador no está en la posición requerida!");
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

        function startTypewriter(elementId, text, speed = 16) {
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

            setTimeout(() => { exitWorldZoom(); }, 300);
        }

        function initFormulasList() {
            const container = document.getElementById('formulas-list-container');
            container.innerHTML = '';

            formulasDB.forEach(f => {
                const card = document.createElement('div');
                card.className = 'formula-card-static';
                card.innerHTML = `
                    <div style="font-family:'Orbitron'; color:var(--yellow); font-size:0.82rem;">${f.title}</div>
                    <div class="math-eq" style="color:var(--cyan); font-size:0.95rem;"></div>
                    <div style="font-size:0.8rem; color:var(--text-muted); line-height:1.35;">${f.dev}</div>
                `;
                container.appendChild(card);
                katex.render(f.eq, card.querySelector('.math-eq'), { displayMode: true, throwOnError: false });
            });
        }

        function openFormulasModal() { document.getElementById('formulas-modal').classList.add('active'); }
        function closeFormulasModal() { document.getElementById('formulas-modal').classList.remove('active'); }

        function showCharReveal(cKey) {
            const c = charactersDB[cKey];
            document.getElementById('reveal-symbol').innerText = c.symbol;
            document.getElementById('reveal-title').innerText = c.name;
            document.getElementById('reveal-desc').innerHTML = c.desc;
            document.getElementById('reveal-modal').classList.add('active');
        }
        function closeReveal() { document.getElementById('reveal-modal').classList.remove('active'); }

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
                        const box = document.getElementById('char-detail-box');
                        box.style.display = 'block';
                        document.getElementById('char-detail-name').innerText = `${c.symbol} - ${c.name}`;
                        document.getElementById('char-detail-desc').innerHTML = c.desc;
                    };
                }
                grid.appendChild(card);
            });

            document.getElementById('codex-modal').classList.add('active');
        }
        function closeCodex() { document.getElementById('codex-modal').classList.remove('active'); }

        /* --- DIBUJO DE ESQUEMAS: DIBUJO DE CAJAS DE IMPEDANCIA Y CIRCUITO COMPLETO DE KIRCHHOFF --- */
        function drawPillLabel(ctx, text, x, y, color) {
            ctx.font = '11px Orbitron';
            const w = ctx.measureText(text).width + 12;
            ctx.fillStyle = 'rgba(6, 9, 17, 0.9)';
            ctx.fillRect(x - w/2, y - 9, w, 18);
            ctx.strokeStyle = color; ctx.lineWidth = 1; ctx.strokeRect(x - w/2, y - 9, w, 18);
            ctx.fillStyle = color; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
            ctx.fillText(text, x, y);
        }

        // DIBUJA UNA CAJA DE IMPEDANCIA RECTANGULAR Z
        function drawZBox(ctx, x, y, width, height, labelText, color) {
            ctx.fillStyle = '#0f172a';
            ctx.fillRect(x, y, width, height);
            ctx.strokeStyle = color || 'var(--cyan)';
            ctx.lineWidth = 2;
            ctx.strokeRect(x, y, width, height);

            ctx.fillStyle = color || 'var(--cyan)';
            ctx.font = 'bold 12px Orbitron';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText(labelText, x + width/2, y + height/2);
        }

        function drawACSource(ctx, cx, cy, radius, label) {
            ctx.fillStyle = '#0f172a';
            ctx.beginPath(); ctx.arc(cx, cy, radius, 0, Math.PI*2); ctx.fill();
            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 2; ctx.stroke();
            
            ctx.beginPath();
            ctx.arc(cx - radius/3, cy, radius/3, Math.PI, 0, false);
            ctx.arc(cx + radius/3, cy, radius/3, 0, Math.PI, true);
            ctx.stroke();

            if (label) drawPillLabel(ctx, label, cx, cy + radius + 15, '#00f3ff');
        }

        function drawSchemaGen(ctx) {
            ctx.clearRect(0,0,700,210);
            ctx.fillStyle='#131d30'; ctx.fillRect(80,45,60,120); ctx.fillRect(360,45,60,120);
            drawPillLabel(ctx, 'Polo Norte N', 110, 105, '#ff0055');
            drawPillLabel(ctx, 'Polo Sur S', 390, 105, '#00f3ff');
            
            ctx.strokeStyle='#ffb700'; ctx.lineWidth=3; ctx.beginPath(); ctx.arc(250,105,35,0,Math.PI*2); ctx.stroke();
            drawPillLabel(ctx, 'Generador AC v(t)', 250, 175, '#00ff66');
        }

        // ESQUEMA MEJORADO DE KIRCHHOFF: CIRCUITO COMPLETO DE DOS MALLAS Y CAJAS DE IMPEDANCIA
        function drawSchemaKCL(ctx) {
            ctx.clearRect(0,0,700,210);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;

            // Malla Izquierda con Fuente V_s
            drawACSource(ctx, 80, 105, 22, 'Vs = 120V');

            // Conexiones de malla
            ctx.beginPath();
            ctx.moveTo(80, 83); ctx.lineTo(80, 40); ctx.lineTo(170, 40);
            ctx.moveTo(230, 40); ctx.lineTo(350, 40); ctx.lineTo(520, 40);
            ctx.moveTo(350, 40); ctx.lineTo(350, 75);
            ctx.moveTo(520, 40); ctx.lineTo(520, 75);
            
            ctx.moveTo(80, 127); ctx.lineTo(80, 170); ctx.lineTo(520, 170);
            ctx.moveTo(350, 135); ctx.lineTo(350, 170);
            ctx.moveTo(520, 135); ctx.lineTo(520, 170);
            ctx.stroke();

            // Cajas de Impedancias
            drawZBox(ctx, 170, 25, 60, 30, 'Z1', '#ffb700');
            drawZBox(ctx, 320, 75, 60, 60, 'Z2', '#00ff66');
            drawZBox(ctx, 490, 75, 60, 60, 'Z3', '#ff0055');

            // Nodo Principal y Flechas de Corriente
            ctx.beginPath(); ctx.arc(350, 40, 6, 0, Math.PI*2); ctx.fillStyle='#ff0055'; ctx.fill();
            drawPillLabel(ctx, 'Nodo V1 (ΣI = 0)', 350, 18, '#ff0055');

            // Flechas de corriente I1, I2, I3
            ctx.fillStyle='#00f3ff'; ctx.font='bold 11px Orbitron';
            ctx.fillText('I1 ➔', 125, 30);
            ctx.fillText('I2 🠗', 370, 60);
            ctx.fillText('I3 🠗', 540, 60);

            // Tierra
            ctx.beginPath(); ctx.moveTo(340, 170); ctx.lineTo(360, 170); ctx.moveTo(345, 175); ctx.lineTo(355, 175); ctx.stroke();
        }

        // ESQUEMA DE IMPEDANCIA RLC CON CAJAS INDIVIDUALES
        function drawSchemaRLC(ctx) {
            ctx.clearRect(0,0,700,210);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;

            drawACSource(ctx, 80, 105, 22, 'Vin AC');

            ctx.beginPath();
            ctx.moveTo(80, 83); ctx.lineTo(80, 40); ctx.lineTo(160, 40);
            ctx.moveTo(230, 40); ctx.lineTo(310, 40);
            ctx.moveTo(380, 40); ctx.lineTo(460, 40); ctx.lineTo(460, 170); ctx.lineTo(80, 170);
            ctx.moveTo(80, 127); ctx.lineTo(80, 170);
            ctx.stroke();

            drawZBox(ctx, 160, 25, 70, 30, 'ZR (R)', '#ffb700');
            drawZBox(ctx, 310, 25, 70, 30, 'ZL (jωL)', '#00ff66');
            drawZBox(ctx, 425, 80, 70, 40, 'ZC (-j/ωC)', '#00f3ff');

            drawPillLabel(ctx, 'Impedancia Total Z_eq = ZR + ZL + ZC', 270, 120, '#ffb700');
        }

        function drawSchemaLPF(ctx) {
            ctx.clearRect(0,0,700,210);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;

            drawACSource(ctx, 80, 105, 22, 'Vin(t)');

            ctx.beginPath();
            ctx.moveTo(80, 83); ctx.lineTo(80, 40); ctx.lineTo(210, 40);
            ctx.moveTo(290, 40); ctx.lineTo(420, 40); ctx.lineTo(540, 40);
            ctx.moveTo(420, 40); ctx.lineTo(420, 80);
            ctx.moveTo(420, 130); ctx.lineTo(420, 170); ctx.lineTo(80, 170);
            ctx.moveTo(80, 127); ctx.lineTo(80, 170);
            ctx.stroke();

            drawZBox(ctx, 210, 25, 80, 30, 'ZR (R)', '#ffb700');
            drawZBox(ctx, 380, 80, 80, 50, 'ZC (1/jωC)', '#00f3ff');

            drawPillLabel(ctx, 'Vout (Bajas Frecuencias)', 540, 40, '#00ff66');
        }

        function drawSchemaHPF(ctx) {
            ctx.clearRect(0,0,700,210);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;

            drawACSource(ctx, 80, 105, 22, 'Vin(t)');

            ctx.beginPath();
            ctx.moveTo(80, 83); ctx.lineTo(80, 40); ctx.lineTo(200, 40);
            ctx.moveTo(290, 40); ctx.lineTo(420, 40); ctx.lineTo(540, 40);
            ctx.moveTo(420, 40); ctx.lineTo(420, 80);
            ctx.moveTo(420, 130); ctx.lineTo(420, 170); ctx.lineTo(80, 170);
            ctx.moveTo(80, 127); ctx.lineTo(80, 170);
            ctx.stroke();

            drawZBox(ctx, 200, 25, 90, 30, 'ZC (1/jωC)', '#00f3ff');
            drawZBox(ctx, 380, 80, 80, 50, 'ZR (R)', '#ffb700');

            drawPillLabel(ctx, 'Vout (Altas Frecuencias)', 540, 40, '#00ff66');
        }

        function drawSchemaPinst(ctx) {
            ctx.clearRect(0,0,700,210);
            drawACSource(ctx, 120, 105, 25, 'Fuente AC RMS');

            ctx.strokeStyle='#00ff66'; ctx.lineWidth=2;
            ctx.beginPath();
            ctx.moveTo(120, 80); ctx.lineTo(120, 45); ctx.lineTo(440, 45); ctx.lineTo(440, 75);
            ctx.moveTo(120, 130); ctx.lineTo(120, 165); ctx.lineTo(440, 165); ctx.lineTo(440, 135);
            ctx.stroke();

            drawZBox(ctx, 400, 75, 80, 60, 'Carga ZL', '#00ff66');
        }

        function drawSchemaTriang(ctx) {
            ctx.clearRect(0,0,700,210);
            drawZBox(ctx, 160, 60, 140, 80, 'Motor Inductivo (Q)', '#ff0055');
            drawZBox(ctx, 400, 60, 140, 80, 'Resistencia Util (P)', '#00f3ff');
        }

        function drawSchemaFP(ctx) {
            ctx.clearRect(0,0,700,210);
            drawZBox(ctx, 140, 60, 150, 80, 'Planta Inductiva QL', '#00ff66');
            drawZBox(ctx, 390, 60, 150, 80, 'Capacitores -QC', '#ff0055');
        }

        function drawSchemaMutua(ctx) {
            ctx.clearRect(0,0,700,210);
            drawZBox(ctx, 180, 70, 70, 70, 'L1', '#ffb700');
            drawZBox(ctx, 430, 70, 70, 70, 'L2', '#00f3ff');

            ctx.strokeStyle='#00ff66'; ctx.lineWidth=2; ctx.setLineDash([5,5]);
            ctx.beginPath(); ctx.moveTo(250, 105); ctx.lineTo(430, 105); ctx.stroke(); ctx.setLineDash([]);
            drawPillLabel(ctx, 'Inductancia Mutua M = k√(L1 L2)', 340, 105, '#00ff66');
        }

        function drawSchemaDots(ctx) {
            ctx.clearRect(0,0,700,210);
            drawZBox(ctx, 200, 65, 80, 80, 'L1', '#00f3ff');
            drawZBox(ctx, 420, 65, 80, 80, 'L2', '#00f3ff');

            ctx.fillStyle='#ff0055';
            ctx.beginPath(); ctx.arc(240, 45, 7, 0, Math.PI*2); ctx.fill();
            ctx.beginPath(); ctx.arc(460, 45, 7, 0, Math.PI*2); ctx.fill();
            drawPillLabel(ctx, 'Puntos Polares de Referencia', 350, 175, '#ff0055');
        }

        function drawSchemaTransfo(ctx) {
            ctx.clearRect(0,0,700,210);
            ctx.fillStyle='#1e293b'; ctx.fillRect(260, 40, 180, 130); ctx.clearRect(300, 65, 100, 80);
            drawPillLabel(ctx, 'Primario N1', 200, 105, '#ffb700');
            drawPillLabel(ctx, 'Secundario N2', 500, 105, '#00f3ff');
        }

        function drawSchemaTriGen(ctx) {
            ctx.clearRect(0,0,700,210);
            const colors = ['#ff0055', '#00f3ff', '#00ff66'], angles = [0, 120, 240];
            angles.forEach((ang, i) => {
                const rad = (ang * Math.PI) / 180, x = 350 + 50 * Math.cos(rad), y = 105 + 50 * Math.sin(rad);
                ctx.strokeStyle = colors[i]; ctx.lineWidth = 3;
                ctx.beginPath(); ctx.moveTo(350,105); ctx.lineTo(x,y); ctx.stroke();
            });
            drawPillLabel(ctx, 'Generador Trifásico Simétrico 120°', 350, 180, '#ffb700');
        }

        function drawSchemaYDelta(ctx) {
            ctx.clearRect(0,0,700,210);
            drawZBox(ctx, 160, 70, 110, 70, 'Estrella Y', '#00f3ff');
            drawZBox(ctx, 420, 70, 110, 70, 'Delta Δ', '#00ff66');
        }

        function drawSchemaDeseb(ctx) {
            ctx.clearRect(0,0,700,210);
            drawZBox(ctx, 220, 70, 260, 70, 'Cargas Desequilibradas (ZA ≠ ZB ≠ ZC)', '#ffb700');
            drawPillLabel(ctx, 'Corriente por Neutro IN ≠ 0', 350, 170, '#ff0055');
        }

        /* --- SIMULADORES CANVA --- */
        function drawSimSenoidal() {
            const Vm = parseFloat(document.getElementById('input-vm').value);
            const phiDeg = parseFloat(document.getElementById('input-phi').value);
            document.getElementById('val-vm').innerText = Vm;
            document.getElementById('val-phi').innerText = phiDeg;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const cy = 100, cxPhasor = 110, phiRad = (phiDeg * Math.PI) / 180, r = Vm * 0.55;

            ctx.strokeStyle = 'rgba(255,255,255,0.08)'; ctx.lineWidth = 1;
            for(let x=0; x<cv.width; x+=30) { ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x,cv.height); ctx.stroke(); }

            ctx.strokeStyle = 'rgba(255,255,255,0.2)'; ctx.lineWidth = 1.5;
            ctx.beginPath(); ctx.moveTo(15, cy); ctx.lineTo(205, cy); ctx.moveTo(cxPhasor, 15); ctx.lineTo(cxPhasor, 185); ctx.stroke();

            const fx = cxPhasor + r * Math.cos(-phiRad), fy = cy + r * Math.sin(-phiRad);
            ctx.strokeStyle = '#ff0055'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(cxPhasor, cy); ctx.lineTo(fx, fy); ctx.stroke();

            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 2.5; ctx.beginPath();
            for (let x = 240; x <= 650; x++) {
                const t = (x - 240) * 0.02;
                const y = cy - r * Math.sin(t + phiRad);
                if (x === 240) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();
        }

        function drawSimRLC() {
            const L = parseFloat(document.getElementById('input-l').value || document.getElementById('input-l2').value) * 1e-3;
            const C = parseFloat(document.getElementById('input-c').value || document.getElementById('input-c2').value) * 1e-6;
            if(document.getElementById('val-l')) document.getElementById('val-l').innerText = (L*1e3).toFixed(0);
            if(document.getElementById('val-c')) document.getElementById('val-c').innerText = (C*1e6).toFixed(0);

            const f0 = 1 / (2 * Math.PI * Math.sqrt(L * C));

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 2.5; ctx.beginPath();
            for (let x = 40; x < 640; x++) {
                const f = (x - 40) * 5, w = 2 * Math.PI * f;
                const Xl = w * L, Xc = w > 0 ? 1 / (w * C) : 1000;
                const Z = Math.sqrt(15*15 + Math.pow(Xl - Xc, 2));
                const y = 170 - Math.min(Z * 1.1, 150);
                if (x === 40) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();

            const xRes = 40 + (f0 / 5);
            if (xRes >= 40 && xRes <= 640) {
                ctx.strokeStyle = '#ffb700'; ctx.setLineDash([4, 4]);
                ctx.beginPath(); ctx.moveTo(xRes, 10); ctx.lineTo(xRes, 180); ctx.stroke(); ctx.setLineDash([]);
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
            ctx.fillText(`Matriz de Admitancia Nodal [Y]:`, 40, 35);
            ctx.fillStyle = '#fff'; ctx.font = '12px Rajdhani';
            ctx.fillText(`[  (1/10 + 1/${Z2})   -1/${Z2}  ]  ·  [ V1 ]  =  [ 10∠0° ]`, 40, 70);
            ctx.fillText(`[     -1/${Z2}          1/${Z2}     ]     [ V2 ]     [   0   ]`, 40, 100);

            const V1 = (120 * (1 + 10/Z2)).toFixed(1);
            ctx.fillStyle = '#00ff66'; ctx.font = '13px Orbitron';
            ctx.fillText(`Voltaje Nodal V1 = ${V1} V ∠ 0°`, 40, 150);
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
            for(let x=40; x<640; x++) {
                const f = Math.pow(10, (x - 40) / 140);
                const H = 1 / Math.sqrt(1 + Math.pow(f / fc, 2));
                const y = 170 - H * 130;
                if(x===40) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();

            const xFc = 40 + Math.log10(fc) * 140;
            if (xFc >= 40 && xFc <= 640) {
                ctx.strokeStyle = '#ffb700'; ctx.setLineDash([4, 4]);
                ctx.beginPath(); ctx.moveTo(xFc, 10); ctx.lineTo(xFc, 180); ctx.stroke(); ctx.setLineDash([]);
                ctx.fillStyle = '#ffb700'; ctx.font = '12px Orbitron';
                ctx.fillText(`fc = ${Math.round(fc)} Hz (-3dB)`, Math.min(xFc + 8, 500), 25);
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
            for(let x=40; x<640; x++) {
                const f = Math.pow(10, (x - 40) / 140);
                const H = (f / fc) / Math.sqrt(1 + Math.pow(f / fc, 2));
                const y = 170 - H * 130;
                if(x===40) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();

            const xFc = 40 + Math.log10(fc) * 140;
            if (xFc >= 40 && xFc <= 640) {
                ctx.strokeStyle = '#ffb700'; ctx.setLineDash([4, 4]);
                ctx.beginPath(); ctx.moveTo(xFc, 10); ctx.lineTo(xFc, 180); ctx.stroke(); ctx.setLineDash([]);
                ctx.fillStyle = '#ffb700'; ctx.font = '12px Orbitron';
                ctx.fillText(`fc = ${Math.round(fc)} Hz (-3dB)`, Math.min(xFc + 8, 500), 25);
            }
        }

        function drawSimPinst() {
            const Vrms = parseFloat(document.getElementById('input-vrms').value);
            document.getElementById('val-vrms').innerText = Vrms;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const Pprom = (Vrms * 1.5).toFixed(0);

            ctx.strokeStyle = '#ff0055'; ctx.lineWidth = 2; ctx.beginPath();
            for(let x=50; x<630; x++) {
                const t = (x-50)*0.03, p = Pprom * (1 + Math.cos(t));
                const y = 170 - p * 0.35;
                if(x===50) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 2; ctx.setLineDash([5,5]);
            const yP = 170 - Pprom * 0.35;
            ctx.beginPath(); ctx.moveTo(50, yP); ctx.lineTo(630, yP); ctx.stroke(); ctx.setLineDash([]);

            ctx.fillStyle = '#ffb700'; ctx.font = '12px Orbitron';
            ctx.fillText(`P_prom = ${Pprom} W`, 500, yP - 8);
        }

        function drawSimTriang() {
            const QL = parseFloat(document.getElementById('input-ql').value);
            document.getElementById('val-ql').innerText = QL;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const P = 300, ox = 80, oy = 165, sc = 0.35;
            const S = Math.sqrt(P*P + QL*QL);

            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 3; ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P*sc, oy); ctx.stroke();
            ctx.strokeStyle = '#ff0055'; ctx.beginPath(); ctx.moveTo(ox + P*sc, oy); ctx.lineTo(ox + P*sc, oy - QL*sc); ctx.stroke();
            ctx.strokeStyle = '#00ff66'; ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P*sc, oy - QL*sc); ctx.stroke();

            ctx.fillStyle = '#fff'; ctx.font = '12px Orbitron';
            ctx.fillText(`|S| = ${Math.round(S)} VA`, 320, 60);
            ctx.fillText(`FP = ${(P/S).toFixed(2)}`, 320, 90);
        }

        function drawSimFP() {
            const QC = parseFloat(document.getElementById('input-qc').value);
            document.getElementById('val-qc').innerText = QC;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const P = 300, QL = 300, Qnet = Math.max(0, QL - QC);
            const S = Math.sqrt(P*P + Qnet*Qnet);
            const FP = (P / S).toFixed(2);

            const ox = 80, oy = 165, sc = 0.35;

            ctx.strokeStyle = 'rgba(255,255,255,0.2)'; ctx.lineWidth = 1;
            ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P*sc, oy - QL*sc); ctx.stroke();

            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P*sc, oy); ctx.lineTo(ox + P*sc, oy - Qnet*sc); ctx.closePath(); ctx.stroke();

            ctx.fillStyle = '#00ff66'; ctx.font = '13px Orbitron';
            ctx.fillText(`FP Resultante = ${FP}`, 320, 60);
        }

        function drawSimMutua() {
            const k = parseFloat(document.getElementById('input-k').value);
            document.getElementById('val-k').innerText = k;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 4; ctx.beginPath(); ctx.arc(180, 95, 35, 0, Math.PI*2); ctx.stroke();
            ctx.strokeStyle = '#00f3ff'; ctx.beginPath(); ctx.arc(490, 95, 35, 0, Math.PI*2); ctx.stroke();

            ctx.strokeStyle = `rgba(0, 255, 102, ${k})`; ctx.lineWidth = k * 5;
            for(let r = 15; r <= 60; r += 15) {
                ctx.beginPath(); ctx.ellipse(335, 95, 155, r, 0, 0, Math.PI*2); ctx.stroke();
            }
        }

        function drawSimDots() {
            const dir = parseInt(document.getElementById('input-dir').value);
            document.getElementById('val-dir').innerText = dir === 1 ? 'Mismo Punto' : 'Punto Opuesto';

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 3;
            ctx.strokeRect(180, 40, 70, 90); ctx.strokeRect(410, 40, 70, 90);

            ctx.fillStyle = '#ff0055';
            ctx.beginPath(); ctx.arc(215, 25, 6, 0, Math.PI*2); ctx.fill();
            ctx.beginPath(); ctx.arc(445, dir === 1 ? 25 : 145, 6, 0, Math.PI*2); ctx.fill();
        }

        function drawSimTransfo() {
            const a = parseFloat(document.getElementById('input-a').value);
            document.getElementById('val-a').innerText = a;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const V1 = 120, V2 = (V1 / a).toFixed(1);

            ctx.fillStyle = '#1e293b'; ctx.fillRect(230, 30, 220, 100); ctx.clearRect(270, 50, 140, 60);

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 2; ctx.beginPath();
            for(let x=30; x<210; x++) {
                const y = 80 - 25 * Math.sin((x-30)*0.05);
                if(x===30) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();

            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 2; ctx.beginPath();
            for(let x=470; x<640; x++) {
                const y = 80 - (25/a) * Math.sin((x-470)*0.05);
                if(x===470) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();

            ctx.fillStyle = '#fff'; ctx.font = '12px Orbitron';
            ctx.fillText(`V1 = ${V1} V`, 80, 135); ctx.fillText(`V2 = ${V2} V`, 520, 135);
        }

        function drawSimTriGen() {
            const Vp = parseFloat(document.getElementById('input-vp').value);
            document.getElementById('val-vp').innerText = Vp;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const cx = 340, cy = 95, r = Vp * 0.5;
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
            ctx.fillText(`Corriente de Fase Ip = ${Ip} A`, 100, 60);
            ctx.fillStyle = '#00ff66';
            ctx.fillText(`Corriente de Línea Delta IL = √3 · Ip = ${IL} A`, 100, 105);
        }

        function drawSimDeseb() {
            const des = parseFloat(document.getElementById('input-des').value);
            document.getElementById('val-des').innerText = des;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const IN = (des * 0.12).toFixed(2);

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(100, 95); ctx.lineTo(100 + des * 3, 95); ctx.stroke();

            ctx.fillStyle = '#ffb700'; ctx.font = '13px Orbitron';
            ctx.fillText(`Corriente de Retorno Neutro IN = ${IN} A`, 100, 60);
        }
    </script>
</body>
</html>