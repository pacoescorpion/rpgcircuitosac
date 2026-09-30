<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Circuitos AC, Potencia & Protoboard RPG</title>
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
            min-height: 100vh;
            background-color: var(--bg-dark);
            color: var(--text-main);
            font-family: 'Rajdhani', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            overflow-x: hidden;
        }

        /* --- HEADER GLOBAL CENTRADO --- */
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

        /* --- CONTENEDOR PRINCIPAL RIGUROSAMENTE CENTRADO --- */
        #viewport-stage {
            position: relative;
            width: 100%;
            max-width: 850px;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 80px 0 60px 0;
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

        /* SPRITE DEL ELECTRÓN */
        .electron-core { fill: #00f3ff; filter: drop-shadow(0 0 10px #00f3ff) drop-shadow(0 0 20px #00f3ff); }
        .electron-core.traversing { fill: #ffb700; filter: drop-shadow(0 0 15px #ffb700) drop-shadow(0 0 25px #ff0055); }
        .electron-text { font-family: 'Orbitron', sans-serif; font-size: 14px; font-weight: 900; fill: #060911; text-anchor: middle; dominant-baseline: central; }

        /* IMPEDANCIAS EN EL CAMINO DE COBRE */
        .path-impedance-group rect {
            fill: #0b1220;
            stroke: var(--cyan);
            stroke-width: 2px;
            rx: 6px;
            filter: drop-shadow(0 0 10px rgba(0,243,255,0.4));
        }
        .path-impedance-group text {
            font-family: 'Orbitron', sans-serif;
            font-size: 11px;
            font-weight: bold;
            fill: var(--yellow);
            text-anchor: middle;
            dominant-baseline: central;
        }

        /* TERMINALES EXTREMOS */
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

        /* NIVELES DE MUNDO EN MAPA GLOBAL */
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

        /* PROTOBOARD MUNDO REAL (EN ZOOM EXTREMO CENTRADO) */
        .protoboard-world-container {
            display: none;
            width: 100%;
            max-width: 780px;
            background-color: #f8fafc;
            background-image: 
                radial-gradient(circle, #475569 2.5px, transparent 3px),
                linear-gradient(to right, #ef4444 2.5px, transparent 2.5px),
                linear-gradient(to right, #3b82f6 2.5px, transparent 2.5px);
            background-size: 20px 20px, 100% 100%, 100% 100%;
            background-position: 0 0, 12px 0, 768px 0;
            border: 4px solid #cbd5e1;
            border-radius: 16px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.9), inset 0 0 12px rgba(0,0,0,0.15);
            padding: 20px;
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
            justify-content: center;
            flex-wrap: wrap;
            align-items: center;
            width: 100%;
            gap: 12px;
        }

        .pin-node-card {
            background: #0f172a;
            border: 2px solid var(--panel-border);
            border-radius: 10px;
            padding: 10px;
            width: 160px;
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

        .pin-icon { font-size: 1.5rem; }
        .pin-name { font-family: 'Orbitron', sans-serif; font-size: 0.65rem; text-align: center; color: #fff; }

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

        /* --- MODAL DE MISIÓN CENTRADO CON ESQUEMA GRANDE Y DIÁLOGO ABAJO --- */
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
            width: 92vw; max-width: 880px;
            max-height: 94vh;
            display: flex; flex-direction: column;
            overflow: hidden;
            box-shadow: 0 0 45px rgba(0, 243, 255, 0.3);
            margin: 0 auto;
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

        /* SECCIÓN DEL ESQUEMA GRANDE SUPERIOR */
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
        .math-eq { font-size: 0.92rem; overflow-x: auto; }

        /* SUBVENTANA DE DIÁLOGO RPG INFERIOR */
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
            width: 56px; height: 56px;
            border-radius: 50%;
            background: #060911;
            border: 2px solid var(--yellow);
            display: flex; justify-content: center; align-items: center;
            font-family: 'Orbitron', sans-serif; font-size: 1.4rem; color: var(--cyan);
            flex-shrink: 0;
            box-shadow: 0 0 15px rgba(255, 183, 0, 0.4);
        }

        .rpg-dialog-content { display: flex; flex-direction: column; gap: 4px; width: 100%; }
        .rpg-speaker-name { font-family: 'Orbitron', sans-serif; font-size: 0.78rem; color: var(--yellow); text-transform: uppercase; }
        .rpg-text-narrative { font-size: 0.92rem; line-height: 1.4; color: #fff; }

        /* RETOS Y SIMULADORES */
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
            margin: 0 auto;
        }
        @keyframes popIn { 0% { transform: scale(0.5); opacity: 0; } 100% { transform: scale(1); opacity: 1; } }
        .reveal-symbol { font-family: 'Orbitron', sans-serif; font-size: 2.8rem; font-weight: 900; color: var(--cyan); text-shadow: 0 0 20px var(--cyan); }

        /* FÓRMULAS Y CÓDEX */
        .formulas-list-sidebar {
            display: grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); gap: 10px; padding: 8px 0;
        }
        .formula-card-static {
            background: rgba(255,255,255,0.03); border: 1px solid rgba(0, 243, 255, 0.2);
            border-radius: 8px; padding: 10px; display: flex; flex-direction: column; gap: 6px;
        }

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

    <!-- HUD GLOBAL CENTRADO -->
    <header id="global-hud">
        <div class="hud-title">⚡ CIRCUITY RPG: AC, POTENCIA & TEORÍAS</div>
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

    <!-- BOTÓN FLOTANTE SALIR DE ZOOM -->
    <button class="btn-exit-zoom" id="btn-exit-zoom" onclick="exitWorldZoom()">🔍 SALIR DEL MUNDO</button>

    <!-- ESCENARIO GLOBAL CENTRADO -->
    <div id="viewport-stage">
        <div id="world-container">
            
            <!-- OVERLAY SVG CABLE DE COBRE E IMPEDANCIAS -->
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

                <!-- GRUPOS DE CAJAS DE IMPEDANCIA EN EL CAMINO -->
                <g id="path-impedances-container"></g>

                <!-- SPRITE ELECTRÓN CON ANIMACIÓN DE ATRAVIESO -->
                <g id="electron-group">
                    <circle id="electron-circle" r="13" class="electron-core" />
                    <text class="electron-text">-</text>
                </g>
            </svg>

            <!-- TIERRA INICIAL -->
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
                        <h4 class="chapter-name-title">Senoidales & Teoremas</h4>
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
                            <div class="pin-name">1.3 Leyes Kirchhoff</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-1-4" onclick="openQuest('1-4')">
                            <div class="pin-icon">⚡</div>
                            <div class="pin-name">1.4 Thévenin & Norton</div>
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
                            <div class="pin-name">2.1 Paso Bajo (LPF)</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-2-2" onclick="openQuest('2-2')">
                            <div class="pin-icon">📈</div>
                            <div class="pin-name">2.2 Paso Alto (HPF)</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-2-3" onclick="openQuest('2-3')">
                            <div class="pin-icon">📊</div>
                            <div class="pin-name">2.3 Paso Banda RLC</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-2-4" onclick="openQuest('2-4')">
                            <div class="pin-icon">🎯</div>
                            <div class="pin-name">2.4 Rechazo (Notch)</div>
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
                        <h4 class="chapter-name-title">Potencia Avanzada</h4>
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
                        <div class="pin-node-card locked" id="sub-3-4" onclick="openQuest('3-4')">
                            <div class="pin-icon">🧲</div>
                            <div class="pin-name">3.4 Máx. Transf. P</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-3-5" onclick="openQuest('3-5')">
                            <div class="pin-icon">🎛️</div>
                            <div class="pin-name">3.5 Método 2 Vatios</div>
                        </div>
                        <div class="pin-node-card locked" id="sub-3-6" onclick="openQuest('3-6')">
                            <div class="pin-icon">📊</div>
                            <div class="pin-name">3.6 Armónicos & THD</div>
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
                        <h4 class="chapter-name-title">Redes Trifásicas</h4>
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
                            <div class="pin-name">5.3 Cargas Desiguales</div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- FUENTE DE ALIMENTACIÓN FINAL -->
            <div class="terminal-node" id="node-source">
                <svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="#e5a059" stroke-width="2">
                    <circle cx="12" cy="12" r="9" />
                    <path d="M 7 12 Q 9.5 7 12 12 T 17 12" />
                </svg>
                <span class="terminal-label">FUENTE AC</span>
            </div>

        </div>
    </div>

    <!-- MODAL DE MISIÓN CENTRADO CON ESQUEMA GRANDE ARRIBA Y DIÁLOGO ABAJO -->
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
                    
                    <!-- ESQUEMA CIRCUITAL GRANDE -->
                    <div class="schema-box-large">
                        <div class="schema-header-tag">
                            <span>🔍 ESQUEMA TÉCNICO DEL CIRCUITO Y CAJAS Z</span>
                        </div>
                        <canvas id="schema-canvas" width="680" height="210"></canvas>
                    </div>

                    <!-- FORMULACIÓN MATEMÁTICA -->
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

                    <!-- SUBVENTANA INFERIOR DE DIÁLOGO RPG -->
                    <div class="rpg-dialog-box" onclick="skipTypewriter()">
                        <div class="rpg-avatar-slot" id="rpg-dialog-avatar">V</div>
                        <div class="rpg-dialog-content">
                            <div class="rpg-speaker-name" id="rpg-dialog-speaker">GUÍA TÉCNICO AC</div>
                            <div class="rpg-text-narrative typing-cursor" id="narrative-text-el">
                                Cargando información de transmisión...
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
                        <div id="challenge-desc" style="font-size:0.88rem; color:#fff;">Observa el comportamiento en tiempo real.</div>
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
                        <canvas id="sim-canvas" width="660" height="200"></canvas>
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

    <!-- MODAL REVELACIÓN DE PERSONAJE -->
    <div class="modal-overlay" id="reveal-modal">
        <div class="reveal-card">
            <div style="font-family: 'Orbitron'; font-size: 0.72rem; color: var(--yellow);">¡NUEVA VARIABLE DESBLOQUEADA!</div>
            <div class="reveal-symbol" id="reveal-symbol">V</div>
            <h3 id="reveal-title" style="color: #fff; font-family: 'Orbitron';">Nombre del Personaje</h3>
            <p id="reveal-desc" style="font-size: 0.85rem; color: var(--text-muted);"></p>
            <button class="nav-btn" onclick="closeReveal()" style="align-self: center;">¡Entendido!</button>
        </div>
    </div>

    <!-- MODAL FÓRMULAS -->
    <div class="modal-overlay" id="formulas-modal">
        <div class="modal-card" style="max-width: 800px;">
            <div class="modal-header">
                <h3>📜 FÓRMULAS DEL PENSUM AC & POTENCIA</h3>
                <button class="close-btn" onclick="closeFormulasModal()">&times;</button>
            </div>
            <div class="modal-body">
                <div class="formulas-list-sidebar" id="formulas-list-container"></div>
            </div>
        </div>
    </div>

    <!-- MODAL CÓDEX -->
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
        /* --- BASE DE DATOS DE FÓRMULAS AMPLIADA --- */
        const formulasDB = [
            { title: "Máxima Transferencia de Potencia AC", eq: "\\mathbf{Z}_L = \\mathbf{Z}_{Th}^* = R_{Th} - jX_{Th}", dev: "Para entregar la máxima potencia activa a la carga, la impedancia de carga debe ser el conjugado complejo de la impedancia de Thévenin. $P_{max} = \\frac{|\\mathbf{V}_{Th}|^2}{4 R_{Th}}$." },
            { title: "Método de los Dos Vatios (Trifásico)", eq: "P_T = W_1 + W_2, \\quad Q_T = \\sqrt{3}(W_1 - W_2)", dev: "Medición de potencia en sistemas trifásicos a 3 hilos mediante dos wattímetros. $W_1 = V_L I_L \\cos(30^\\circ - \\theta)$." },
            { title: "Distorsión Armónica Total (THD)", eq: "THD = \\frac{\\sqrt{\\sum_{h=2}^{\\infty} V_h^2}}{V_1} \\times 100\\%", dev: "Mide la proporción de armónicos respecto a la componente fundamental de $60\\text{ Hz}$ en ondas no senoidales." },
            { title: "Equivalente de Thévenin Complejo", eq: "\\mathbf{V}_{Th} = \\mathbf{V}_{oc}, \\quad \\mathbf{Z}_{Th} = \\frac{\\mathbf{V}_{oc}}{\\mathbf{I}_{sc}}", dev: "Reducción de cualquier red lineal pasiva AC a una sola fuente de voltaje con una caja de impedancia en serie." },
            { title: "Filtro Notch (Rechazo de Banda)", eq: "H(j\\omega) = \\frac{\\omega_0^2 - \\omega^2}{\\omega_0^2 - \omega^2 + j\\omega B}", dev: "Elimina una frecuencia específica $\\omega_0 = 1/\\sqrt{LC}$ permitiendo el paso de frecuencias inferiores y superiores." },
            { title: "Divisor de Tensión LPF", eq: "H(j\\omega) = \\frac{1}{1 + j\\omega RC}", dev: "Frecuencia de corte $\\omega_c = \\frac{1}{RC}$." },
            { title: "Potencia Compleja Vectorial", eq: "\\mathbf{S} = P + jQ = \\mathbf{V}_{rms} \\mathbf{I}_{rms}^*", dev: "Suma ortogonal de la Potencia Activa $P$ [W] y Reactiva $Q$ [VAR]." }
        ];

        /* --- PERSONAJES Y PENSUM AMPLIADO --- */
        const charactersDB = {
            V: { symbol: 'V', name: 'Voltaje Senoidal', desc: 'Fuerza electromotriz que impulsas los electrones en corriente alterna.', introKey: '1-1' },
            I: { symbol: 'I', name: 'Corriente Alterna', desc: 'Flujo de carga oscilante en el dominio del tiempo y la frecuencia.', introKey: '1-1' },
            Z: { symbol: 'Z', name: 'Impedancia Compleja', desc: 'Oposición fasorial total (Resistencia + Reactancia).', introKey: '1-2' },
            K: { symbol: 'K', name: 'Leyes de Kirchhoff', desc: 'Ecuaciones matriciales fasoriales de nodo y malla.', introKey: '1-3' },
            Th: { symbol: 'Th', name: 'Thévenin & Norton', desc: 'Reducción de circuitos complejos a impedancias equivalentes.', introKey: '1-4' },
            H: { symbol: 'H(jω)', name: 'Función Transferencia', desc: 'Respuesta en frecuencia de filtros pasivos.', introKey: '2-1' },
            P: { symbol: 'P', name: 'Potencia Activa', desc: 'Energía real convertida en trabajo o calor (Watts).', introKey: '3-1' },
            S: { symbol: 'S', name: 'Potencia Aparente', desc: 'Magnitud vectorial total de capacidad del sistema (VA).', introKey: '3-2' },
            FP: { symbol: 'FP', name: 'Factor de Potencia', desc: 'Relación de eficiencia energética cos(θ).', introKey: '3-3' },
            Pmax: { symbol: 'Pmax', name: 'Máxima Potencia', desc: 'Acoplamiento conjugado de impedancia ZL = ZTh*.', introKey: '3-4' },
            W: { symbol: 'W12', name: 'Método 2 Vatios', desc: 'Medición de potencia activa y reactiva trifásica.', introKey: '3-5' },
            THD: { symbol: 'THD', name: 'Distorsión Armónica', desc: 'Presencia de armónicos no senoidales en la red.', introKey: '3-6' },
            M: { symbol: 'M', name: 'Inductancia Mutua', desc: 'Acoplamiento magnético por flujo compartido.', introKey: '4-1' },
            VL: { symbol: 'VL', name: 'Tensión Trifásica', desc: 'Voltaje entre líneas de transmisión polifásicas.', introKey: '5-1' }
        };

        const questsDB = {
            '1-1': {
                title: 'Misión 1.1: Onda Senoidal y Fasores', chap: 'chap-1', next: '1-2', introChar: 'V',
                speaker: 'Voltaje Fasorial V',
                narrative: 'Una fuente AC genera $v(t) = V_m \\cos(\\omega t + \\phi)$. Para analizar el circuito con números complejos usamos el fasor $\\mathbf{V} = V_{rms} \\angle \\phi$.',
                eq1: 'v(t) = V_m \\cdot \\cos(\\omega t + \\phi)', eq2: '\\mathbf{V} = \\frac{V_m}{\\sqrt{2}} \\angle \\phi',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Fasor V:</b> Magnitud y fase.</div>',
                controls: `<div class="control-group"><label>Amplitud Vm: <span id="val-vm">100</span> V</label><input type="range" id="input-vm" min="30" max="150" value="100" oninput="drawSimSenoidal()"></div><div class="control-group"><label>Desfase ϕ: <span id="val-phi">45</span>°</label><input type="range" id="input-phi" min="-180" max="180" value="45" oninput="drawSimSenoidal()"></div>`,
                drawSchema: (ctx) => drawSchemaGen(ctx), drawSim: () => drawSimSenoidal(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Equivalencia entre fasor y onda senoidal." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "El valor eficaz de una onda senoidal se conoce como valor ________.", answer: "rms" },
                    { title: "Ejercicio 2: Ajuste Amplitud", type: "sim", desc: "Ajusta la Amplitud Vm a 120 V.", check: () => parseFloat(document.getElementById('input-vm').value) === 120 },
                    { title: "Ejercicio 3: Ajuste Fase", type: "sim", desc: "Fija el desfase a +90°.", check: () => parseFloat(document.getElementById('input-phi').value) === 90 },
                    { title: "👾 DESAFÍO BOSS: Generar Vrms = 100V", type: "sim", desc: "Fija Vm = 141 V y desfase a +45°.", check: () => parseFloat(document.getElementById('input-vm').value) === 141 && parseFloat(document.getElementById('input-phi').value) === 45 }
                ]
            },
            '1-2': {
                title: 'Misión 1.2: Impedancia Compleja Z', chap: 'chap-1', next: '1-3', introChar: 'Z',
                speaker: 'Impedancia Z',
                narrative: 'La **Impedancia Compleja** $\\mathbf{Z} = R + jX$ combina la resistencia real y la reactancia imaginaria. Representamos cada elemento mediante cajas en el circuito.',
                eq1: '\\mathbf{Z} = R + j\\left(\\omega L - \\frac{1}{\\omega C}\\right)', eq2: '|\\mathbf{Z}| = \\sqrt{R^2 + X^2}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Magnitud |Z|:</b> Oposición total en Ohmios.</div>',
                controls: `<div class="control-group"><label>Inductancia L: <span id="val-l">10</span> mH</label><input type="range" id="input-l" min="1" max="50" value="10" oninput="drawSimRLC()"></div><div class="control-group"><label>Capacitancia C: <span id="val-c">10</span> µF</label><input type="range" id="input-c" min="1" max="50" value="10" oninput="drawSimRLC()"></div>`,
                drawSchema: (ctx) => drawSchemaRLC(ctx), drawSim: () => drawSimRLC(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Variación de la impedancia RLC." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "La unidad de medida de la impedancia es el ________.", answer: "ohmio" },
                    { title: "Ejercicio 2: Inductancia L", type: "sim", desc: "Ajusta L = 25 mH.", check: () => parseFloat(document.getElementById('input-l').value) === 25 },
                    { title: "Ejercicio 3: Capacitancia C", type: "sim", desc: "Fija C = 40 µF.", check: () => parseFloat(document.getElementById('input-c').value) === 40 },
                    { title: "👾 DESAFÍO BOSS: Igualar Reactancias", type: "sim", desc: "Configura L = 20 mH y C = 20 µF.", check: () => parseFloat(document.getElementById('input-l').value) === 20 && parseFloat(document.getElementById('input-c').value) === 20 }
                ]
            },
            '1-3': {
                title: 'Misión 1.3: Leyes de Kirchhoff y Mallas AC', chap: 'chap-1', next: '1-4', introChar: 'K',
                speaker: 'Maestro Kirchhoff',
                narrative: 'En un circuito de dos mallas con cajas de impedancia $\\mathbf{Z}_1, \\mathbf{Z}_2, \\mathbf{Z}_3$, resolvemos sumando tensiones en mallas y corrientes en nodos: $[\\mathbf{Y}][\\mathbf{V}] = [\\mathbf{I}]$.',
                eq1: '\\sum \\mathbf{I}_{nodo} = 0 \\implies \\mathbf{I}_1 - \\mathbf{I}_2 - \\mathbf{I}_3 = 0', eq2: '[\\mathbf{Y}] \\cdot [\\mathbf{V}] = [\\mathbf{I}]',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Corriente Nodal:</b> Conservación de carga.</div>',
                controls: `<div class="control-group"><label>Impedancia Z2: <span id="val-z2">20</span> Ω</label><input type="range" id="input-z2" min="5" max="50" value="20" oninput="drawSimKCL()"></div>`,
                drawSchema: (ctx) => drawSchemaKCL(ctx), drawSim: () => drawSimKCL(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Ecuaciones matriciales de nodo." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "La primera ley de Kirchhoff establece la conservación de la ________.", answer: "carga" },
                    { title: "Ejercicio 2: Ajuste Z2", type: "sim", desc: "Ajusta Z2 = 15 Ω.", check: () => parseFloat(document.getElementById('input-z2').value) === 15 },
                    { title: "Ejercicio 3: Impedancia Alta", type: "sim", desc: "Sube Z2 = 35 Ω.", check: () => parseFloat(document.getElementById('input-z2').value) === 35 },
                    { title: "👾 DESAFÍO BOSS: Balance Nodal Z2 = 10Ω", type: "sim", desc: "Ajusta Z2 a 10 Ω exactamente.", check: () => parseFloat(document.getElementById('input-z2').value) === 10 }
                ]
            },
            '1-4': {
                title: 'Misión 1.4: Thévenin y Norton en AC', chap: 'chap-1', next: '2-1', introChar: 'Th',
                speaker: 'Thévenin & Norton',
                narrative: 'Cualquier red AC lineal vista desde dos terminales se reduce a una fuente $\\mathbf{V}_{Th}$ en serie con una caja de impedancia de Thévenin $\\mathbf{Z}_{Th} = R_{Th} + jX_{Th}$.',
                eq1: '\\mathbf{V}_{Th} = \\mathbf{V}_{oc}, \\quad \\mathbf{Z}_{Th} = R_{Th} + jX_{Th}', eq2: '\\mathbf{I}_N = \\frac{\\mathbf{V}_{Th}}{\\mathbf{Z}_{Th}}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Equivalente ZTh:</b> Reducción fasorial.</div>',
                controls: `<div class="control-group"><label>Resistencia Rth: <span id="val-rth">50</span> Ω</label><input type="range" id="input-rth" min="10" max="100" value="50" oninput="drawSimThevenin()"></div>`,
                drawSchema: (ctx) => drawSchemaThevenin(ctx), drawSim: () => drawSimThevenin(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Reducción de red a fuente e impedancia única." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "La impedancia de Thévenin se calcula desactivando todas las fuentes ________.", answer: "independientes" },
                    { title: "Ejercicio 2: Ajuste Rth", type: "sim", desc: "Ajusta Rth = 30 Ω.", check: () => parseFloat(document.getElementById('input-rth').value) === 30 },
                    { title: "Ejercicio 3: Resistencia Alta", type: "sim", desc: "Fija Rth = 80 Ω.", check: () => parseFloat(document.getElementById('input-rth').value) === 80 },
                    { title: "👾 DESAFÍO BOSS: Configurar Rth = 25 Ω", type: "sim", desc: "Ajusta Rth exactamente a 25 Ω.", check: () => parseFloat(document.getElementById('input-rth').value) === 25 }
                ]
            },
            '2-1': {
                title: 'Misión 2.1: Filtro Paso Bajo RC (LPF)', chap: 'chap-2', next: '2-2', introChar: 'H',
                speaker: 'Filtro H(jω)',
                narrative: 'Aplicando un divisor de tensión sobre la caja de impedancia capacitiva $\\mathbf{Z}_C$, atenuamos altas frecuencias: $H(j\\omega) = \\frac{1}{1 + j\\omega RC}$.',
                eq1: 'H(j\\omega) = \\frac{\\mathbf{Z}_C}{R + \\mathbf{Z}_C} = \\frac{1}{1 + j\\omega RC}', eq2: 'f_c = \\frac{1}{2\\pi RC}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Atenuación:</b> -20 dB/década.</div>',
                controls: `<div class="control-group"><label>Resistencia R: <span id="val-r">1000</span> Ω</label><input type="range" id="input-r" min="100" max="5000" step="100" value="1000" oninput="drawSimLPF()"></div><div class="control-group"><label>Capacitancia C: <span id="val-c">0.1</span> µF</label><input type="range" id="input-c" min="0.01" max="1" step="0.01" value="0.1" oninput="drawSimLPF()"></div>`,
                drawSchema: (ctx) => drawSchemaLPF(ctx), drawSim: () => drawSimLPF(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Respuesta en frecuencia LPF." },
                    { title: "Ejercicio 1: Concepto", type: "fill_blank", desc: "Un filtro paso bajo atenúa las altas ________.", answer: "frecuencias" },
                    { title: "Ejercicio 2: Ajuste R", type: "sim", desc: "Fija R = 2000 Ω.", check: () => parseFloat(document.getElementById('input-r').value) === 2000 },
                    { title: "Ejercicio 3: Ajuste C", type: "sim", desc: "Fija C = 0.5 µF.", check: () => Math.abs(parseFloat(document.getElementById('input-c').value) - 0.5) < 0.01 },
                    { title: "👾 DESAFÍO BOSS: Frecuencia fc ≈ 1591 Hz", type: "sim", desc: "Ajusta R = 1000 Ω y C = 0.1 µF.", check: () => parseFloat(document.getElementById('input-r').value) === 1000 && Math.abs(parseFloat(document.getElementById('input-c').value) - 0.1) < 0.01 }
                ]
            },
            '2-2': { title: 'Misión 2.2: Filtro Paso Alto RC', chap: 'chap-2', next: '2-3', introChar: 'H', speaker: 'Filtro H(jω)', narrative: 'Bloqueamos DC conectando la caja capacitiva en serie.', eq1: 'H(j\\omega) = \\frac{j\\omega RC}{1 + j\\omega RC}', eq2: 'f_c = \\frac{1}{2\\pi RC}', legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Paso Alto:</b> Deja pasar f > fc.</div>', controls: `<div class="control-group"><label>R: <span id="val-rh">1000</span> Ω</label><input type="range" id="input-rh" min="100" max="5000" step="100" value="1000" oninput="drawSimHPF()"></div><div class="control-group"><label>C: <span id="val-ch">0.1</span> µF</label><input type="range" id="input-ch" min="0.01" max="1" step="0.01" value="0.1" oninput="drawSimHPF()"></div>`, drawSchema: (ctx) => drawSchemaHPF(ctx), drawSim: () => drawSimHPF(), challenges: [{ title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Respuesta HPF." }, { title: "Ejercicio 1", type: "fill_blank", desc: "Un condensador bloquea la corriente ________.", answer: "continua" }, { title: "Ejercicio 2", type: "sim", desc: "Fija R = 1500 Ω.", check: () => parseFloat(document.getElementById('input-rh').value) === 1500 }, { title: "Ejercicio 3", type: "sim", desc: "Fija C = 0.05 µF.", check: () => Math.abs(parseFloat(document.getElementById('input-ch').value) - 0.05) < 0.005 }, { title: "👾 BOSS", type: "sim", desc: "Ajusta R = 5000 Ω y C = 0.1 µF.", check: () => parseFloat(document.getElementById('input-rh').value) === 5000 && Math.abs(parseFloat(document.getElementById('input-ch').value) - 0.1) < 0.01 }] },
            '2-3': { title: 'Misión 2.3: Paso Banda RLC', chap: 'chap-2', next: '2-4', introChar: 'H', speaker: 'Filtro H(jω)', narrative: 'Filtro de resonancia seleccionando una banda de paso alrededor de f0.', eq1: 'f_0 = \\frac{1}{2\\pi \\sqrt{LC}}', eq2: 'Q = \\frac{\\omega_0 L}{R}', legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Paso Banda:</b> Máximo pico en f0.</div>', controls: `<div class="control-group"><label>L: <span id="val-l2">10</span> mH</label><input type="range" id="input-l2" min="1" max="50" value="10" oninput="drawSimRLC()"></div><div class="control-group"><label>C: <span id="val-c2">10</span> µF</label><input type="range" id="input-c2" min="1" max="50" value="10" oninput="drawSimRLC()"></div>`, drawSchema: (ctx) => drawSchemaRLC(ctx), drawSim: () => drawSimRLC(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Sintonía de banda." }, { title: "Ejercicio 1", type: "fill_blank", desc: "La máxima transferencia ocurre en la frecuencia de ________.", answer: "resonancia" }, { title: "Ejercicio 2", type: "sim", desc: "Fija L = 10 mH.", check: () => parseFloat(document.getElementById('input-l2').value) === 10 }, { title: "Ejercicio 3", type: "sim", desc: "Fija C = 10 µF.", check: () => parseFloat(document.getElementById('input-c2').value) === 10 }, { title: "👾 BOSS", type: "sim", desc: "Ajusta L = 5 mH y C = 5 µF.", check: () => parseFloat(document.getElementById('input-l2').value) === 5 && parseFloat(document.getElementById('input-c2').value) === 5 }] },
            '2-4': { title: 'Misión 2.4: Filtro Rechazo de Banda (Notch)', chap: 'chap-2', next: '3-1', introChar: 'H', speaker: 'Filtro Notch', narrative: 'El filtro Rechazo de Banda elimina interferencias ruidosas fijas (ej. 60 Hz).', eq1: 'H(j\\omega) = \\frac{\\omega_0^2 - \\omega^2}{\\omega_0^2 - \\omega^2 + j\\omega B}', eq2: 'f_{notch} = \\frac{1}{2\\pi \\sqrt{LC}}', legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Muesca Notch:</b> Atenuación profunda en f0.</div>', controls: `<div class="control-group"><label>Inductancia L: <span id="val-ln">10</span> mH</label><input type="range" id="input-ln" min="1" max="50" value="10" oninput="drawSimNotch()"></div>`, drawSchema: (ctx) => drawSchemaNotch(ctx), drawSim: () => drawSimNotch(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Eliminación de ruido." }, { title: "Ejercicio 1", type: "fill_blank", desc: "Un filtro notch se utiliza para eliminar frecuencias de ________.", answer: "ruido" }, { title: "Ejercicio 2", type: "sim", desc: "Fija L = 20 mH.", check: () => parseFloat(document.getElementById('input-ln').value) === 20 }, { title: "Ejercicio 3", type: "sim", desc: "Fija L = 40 mH.", check: () => parseFloat(document.getElementById('input-ln').value) === 40 }, { title: "👾 BOSS", type: "sim", desc: "Configura L = 10 mH.", check: () => parseFloat(document.getElementById('input-ln').value) === 10 }] },
            '3-1': { title: 'Misión 3.1: Potencia RMS y Activa', chap: 'chap-3', next: '3-2', introChar: 'P', speaker: 'Potencia P', narrative: 'La potencia activa disipada en las cajas resistivas de la red.', eq1: 'P = V_{rms} I_{rms} \\cos\\theta', eq2: 'p(t) = P(1 + \\cos(2\\omega t))', legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>P:</b> Trabajo útil disipado.</div>', controls: `<div class="control-group"><label>Vrms: <span id="val-vrms">120</span> V</label><input type="range" id="input-vrms" min="60" max="240" value="120" oninput="drawSimPinst()"></div>`, drawSchema: (ctx) => drawSchemaPinst(ctx), drawSim: () => drawSimPinst(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Potencia instantánea." }, { title: "Ejercicio 1", type: "fill_blank", desc: "La potencia activa se mide en la unidad de ________.", answer: "vatios" }, { title: "Ejercicio 2", type: "sim", desc: "Ajusta Vrms = 110 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 110 }, { title: "Ejercicio 3", type: "sim", desc: "Ajusta Vrms = 220 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 220 }, { title: "👾 BOSS", type: "sim", desc: "Ajusta Vrms = 240 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 240 }] },
            '3-2': { title: 'Misión 3.2: Triángulo de Potencia', chap: 'chap-3', next: '3-3', introChar: 'S', speaker: 'Potencia S', narrative: 'Suma vectorial ortogonal: Potencia Aparente S = P + jQ.', eq1: '\\mathbf{S} = P + jQ', eq2: '|\\mathbf{S}| = \\sqrt{P^2 + Q^2}', legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Vector S:</b> Aparente total.</div>', controls: `<div class="control-group"><label>Reactiva QL: <span id="val-ql">250</span> VAR</label><input type="range" id="input-ql" min="50" max="450" value="250" oninput="drawSimTriang()"></div>`, drawSchema: (ctx) => drawSchemaTriang(ctx), drawSim: () => drawSimTriang(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Triángulo vectorial P-Q-S." }, { title: "Ejercicio 1", type: "fill_blank", desc: "La potencia aparente S se expresa en voltio-________.", answer: "amperios" }, { title: "Ejercicio 2", type: "sim", desc: "Fija QL = 100 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 100 }, { title: "Ejercicio 3", type: "sim", desc: "Fija QL = 300 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 300 }, { title: "👾 BOSS", type: "sim", desc: "Ajusta QL = 300 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 300 }] },
            '3-3': { title: 'Misión 3.3: Corrección del Factor de Potencia', chap: 'chap-3', next: '3-4', introChar: 'FP', speaker: 'Eficiencia FP', narrative: 'Inyectamos potencia reactiva capacitiva -jQC en paralelo.', eq1: 'FP = \\frac{P}{|\\mathbf{S}|} = \\cos\\theta', eq2: 'Q_C = P(\\tan\\theta_1 - \\tan\\theta_2)', legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>S Compensado:</b> Reducción de corriente.</div>', controls: `<div class="control-group"><label>Inyección QC: <span id="val-qc">150</span> VAR</label><input type="range" id="input-qc" min="0" max="350" value="150" oninput="drawSimFP()"></div>`, drawSchema: (ctx) => drawSchemaFP(ctx), drawSim: () => drawSimFP(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Compensación de reactiva." }, { title: "Ejercicio 1", type: "fill_blank", desc: "Un factor de potencia unitario implica un desfase de ________ grados.", answer: "cero" }, { title: "Ejercicio 2", type: "sim", desc: "Fija QC = 100 VAR.", check: () => parseFloat(document.getElementById('input-qc').value) === 100 }, { title: "Ejercicio 3", type: "sim", desc: "Sube QC = 200 VAR.", check: () => parseFloat(document.getElementById('input-qc').value) === 200 }, { title: "👾 BOSS", type: "sim", desc: "Ajusta QC = 210 VAR para lograr FP = 0.96.", check: () => parseFloat(document.getElementById('input-qc').value) === 210 }] },
            '3-4': { title: 'Misión 3.4: Teorema de Máxima Transferencia de Potencia', chap: 'chap-3', next: '3-5', introChar: 'Pmax', speaker: 'Máxima Potencia Pmax', narrative: 'Para entregar la máxima potencia activa a la carga ZL, esta debe ser el conjugado complejo de la impedancia interna de la fuente: ZL = ZTh*.', eq1: '\\mathbf{Z}_L = R_{Th} - jX_{Th}', eq2: 'P_{max} = \\frac{|\\mathbf{V}_{Th}|^2}{4 R_{Th}}', legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Pmax:</b> Pico en acoplamiento conjugado.</div>', controls: `<div class="control-group"><label>Resistencia RL: <span id="val-rl">50</span> Ω</label><input type="range" id="input-rl" min="10" max="100" value="50" oninput="drawSimPmax()"></div>`, drawSchema: (ctx) => drawSchemaPmax(ctx), drawSim: () => drawSimPmax(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Curva de transferencia de potencia." }, { title: "Ejercicio 1", type: "fill_blank", desc: "Para máxima potencia en AC, la impedancia de carga debe ser el conjugado ________.", answer: "complejo" }, { title: "Ejercicio 2", type: "sim", desc: "Ajusta RL = 30 Ω.", check: () => parseFloat(document.getElementById('input-rl').value) === 30 }, { title: "Ejercicio 3", type: "sim", desc: "Ajusta RL = 70 Ω.", check: () => parseFloat(document.getElementById('input-rl').value) === 70 }, { title: "👾 BOSS: Acoplamiento Perfecto RL = RTh", type: "sim", desc: "Como RTh = 50 Ω, ajusta RL = 50 Ω.", check: () => parseFloat(document.getElementById('input-rl').value) === 50 }] },
            '3-5': { title: 'Misión 3.5: Método de Medición de los Dos Vatios', chap: 'chap-3', next: '3-6', introChar: 'W', speaker: 'Wattímetros Trifásicos', narrative: 'Medimos la potencia total de una red trifásica conectando dos wattímetros entre fases.', eq1: 'P_T = W_1 + W_2', eq2: 'Q_T = \\sqrt{3}(W_1 - W_2)', legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>W1 + W2:</b> Lectura de potencia total.</div>', controls: `<div class="control-group"><label>Ángulo Carga θ: <span id="val-theta">30</span>°</label><input type="range" id="input-theta" min="0" max="80" value="30" oninput="drawSim2Watt()"></div>`, drawSchema: (ctx) => drawSchema2Watt(ctx), drawSim: () => drawSim2Watt(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Medición con dos wattímetros." }, { title: "Ejercicio 1", type: "fill_blank", desc: "El método de dos vatios funciona en sistemas trifásicos a tres ________.", answer: "hilos" }, { title: "Ejercicio 2", type: "sim", desc: "Ajusta ángulo θ = 0°.", check: () => parseFloat(document.getElementById('input-theta').value) === 0 }, { title: "Ejercicio 3", type: "sim", desc: "Ajusta ángulo θ = 60°.", check: () => parseFloat(document.getElementById('input-theta').value) === 60 }, { title: "👾 BOSS", type: "sim", desc: "Ajusta θ = 30°.", check: () => parseFloat(document.getElementById('input-theta').value) === 30 }] },
            '3-6': { title: 'Misión 3.6: Potencia con Armónicos y THD', chap: 'chap-3', next: '4-1', introChar: 'THD', speaker: 'Analizador THD', narrative: 'Las cargas no lineales generan armónicos que distorsionan la senoidal pura.', eq1: 'THD = \\frac{\\sqrt{V_3^2 + V_5^2}}{V_1} \\times 100\\%', eq2: 'S = \\sqrt{P^2 + Q^2 + D^2}', legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Distorsión D:</b> Deformación armónica.</div>', controls: `<div class="control-group"><label>Armónico 3º: <span id="val-h3">20</span>%</label><input type="range" id="input-h3" min="0" max="50" value="20" oninput="drawSimTHD()"></div>`, drawSchema: (ctx) => drawSchemaTHD(ctx), drawSim: () => drawSimTHD(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Superposición armónica." }, { title: "Ejercicio 1", type: "fill_blank", desc: "La distorsión armónica se abrevia con las siglas ________.", answer: "thd" }, { title: "Ejercicio 2", type: "sim", desc: "Fija Armónico = 0%.", check: () => parseFloat(document.getElementById('input-h3').value) === 0 }, { title: "Ejercicio 3", type: "sim", desc: "Sube Armónico = 40%.", check: () => parseFloat(document.getElementById('input-h3').value) === 40 }, { title: "👾 BOSS", type: "sim", desc: "Ajusta Armónico a 25%.", check: () => parseFloat(document.getElementById('input-h3').value) === 25 }] },
            '4-1': { title: 'Misión 4.1: Inductancia Mutua M', chap: 'chap-4', next: '4-2', introChar: 'M', speaker: 'Inductancia Mutua', narrative: 'Flujo magnético compartido M = k √(L1 L2).', eq1: 'v_2(t) = M \\frac{di_1}{dt}', eq2: 'M = k \\sqrt{L_1 L_2}', legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>k:</b> Coeficiente magnético.</div>', controls: `<div class="control-group"><label>k: <span id="val-k">0.7</span></label><input type="range" id="input-k" min="0.1" max="1" step="0.05" value="0.7" oninput="drawSimMutua()"></div>`, drawSchema: (ctx) => drawSchemaMutua(ctx), drawSim: () => drawSimMutua(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Acoplamiento mutuo." }, { title: "Ejercicio 1", type: "fill_blank", desc: "El coeficiente k ideal máximo es igual a ____.", answer: "1" }, { title: "Ejercicio 2", type: "sim", desc: "Fija k = 0.3.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 0.3) < 0.01 }, { title: "Ejercicio 3", type: "sim", desc: "Fija k = 0.6.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 0.6) < 0.01 }, { title: "👾 BOSS", type: "sim", desc: "Ajusta k = 1.0.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 1.0) < 0.01 }] },
            '4-2': { title: 'Misión 4.2: Regla de Puntos en Bobinas', chap: 'chap-4', next: '4-3', introChar: 'M', speaker: 'Regla de Puntos', narrative: 'Polaridad relativa de la tensión inducida por mutua.', eq1: '\\mathbf{V}_1 = j\\omega L_1 \\mathbf{I}_1 \\pm j\\omega M \\mathbf{I}_2', eq2: '\\mathbf{V}_2 = j\\omega L_2 \\mathbf{I}_2 \\pm j\\omega M \\mathbf{I}_1', legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Puntos Polares:</b> Referencias.</div>', controls: `<div class="control-group"><label>Dirección I2: <span id="val-dir">Mismo Punto</span></label><input type="range" id="input-dir" min="0" max="1" step="1" value="1" oninput="drawSimDots()"></div>`, drawSchema: (ctx) => drawSchemaDots(ctx), drawSim: () => drawSimDots(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Inversión de puntos." }, { title: "Ejercicio 1", type: "fill_blank", desc: "Si las corrientes entran por el punto, las tensiones inducidas se ________.", answer: "suman" }, { title: "Ejercicio 2", type: "sim", desc: "Selecciona Punto Opuesto (0).", check: () => parseInt(document.getElementById('input-dir').value) === 0 }, { title: "Ejercicio 3", type: "sim", desc: "Selecciona Mismo Punto (1).", check: () => parseInt(document.getElementById('input-dir').value) === 1 }, { title: "👾 BOSS", type: "sim", desc: "Configura Mismo Punto (1).", check: () => parseInt(document.getElementById('input-dir').value) === 1 }] },
            '4-3': { title: 'Misión 4.3: Transformador Ideal', chap: 'chap-4', next: '5-1', introChar: 'M', speaker: 'Transformador', narrative: 'Relación de transformación a = N1 / N2.', eq1: 'a = \\frac{N_1}{N_2} = \\frac{\\mathbf{V}_1}{\\mathbf{V}_2}', eq2: '\\mathbf{Z}_{in} = a^2 \\mathbf{Z}_L', legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>V2 Salida:</b> Escalado.</div>', controls: `<div class="control-group"><label>Razón a: <span id="val-a">2.0</span></label><input type="range" id="input-a" min="0.5" max="4" step="0.5" value="2" oninput="drawSimTransfo()"></div>`, drawSchema: (ctx) => drawSchemaTransfo(ctx), drawSim: () => drawSimTransfo(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Transformación de voltaje." }, { title: "Ejercicio 1", type: "fill_blank", desc: "La relación de vueltas se simboliza con la letra ____.", answer: "a" }, { title: "Ejercicio 2", type: "sim", desc: "Fija a = 2.0.", check: () => parseFloat(document.getElementById('input-a').value) === 2.0 }, { title: "Ejercicio 3", type: "sim", desc: "Fija a = 0.5.", check: () => parseFloat(document.getElementById('input-a').value) === 0.5 }, { title: "👾 BOSS", type: "sim", desc: "Ajusta a = 4.0 para V2 = 30V.", check: () => parseFloat(document.getElementById('input-a').value) === 4.0 }] },
            '5-1': { title: 'Misión 5.1: Generador Trifásico', chap: 'chap-5', next: '5-2', introChar: 'VL', speaker: 'Línea VL', narrative: 'Tensiones simétricas desfasadas 120°.', eq1: 'V_L = \\sqrt{3} V_p \\angle 30^\\circ', eq2: '\\mathbf{V}_A + \\mathbf{V}_B + \\mathbf{V}_C = 0', legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Fases ABC:</b> Desfasadas 120°.</div>', controls: `<div class="control-group"><label>Vp: <span id="val-vp">120</span> V</label><input type="range" id="input-vp" min="60" max="240" value="120" oninput="drawSimTriGen()"></div>`, drawSchema: (ctx) => drawSchemaTriGen(ctx), drawSim: () => drawSimTriGen(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Rotación trifásica." }, { title: "Ejercicio 1", type: "fill_blank", desc: "El desfase simétrico entre fases es de ____ grados.", answer: "120" }, { title: "Ejercicio 2", type: "sim", desc: "Fija Vp = 100 V.", check: () => parseFloat(document.getElementById('input-vp').value) === 100 }, { title: "Ejercicio 3", type: "sim", desc: "Fija Vp = 200 V.", check: () => parseFloat(document.getElementById('input-vp').value) === 200 }, { title: "👾 BOSS", type: "sim", desc: "Ajusta Vp = 120 V.", check: () => parseFloat(document.getElementById('input-vp').value) === 120 }] },
            '5-2': { title: 'Misión 5.2: Conexión Estrella / Delta', chap: 'chap-5', next: '5-3', introChar: 'VL', speaker: 'Línea VL', narrative: 'Relaciones de fase y línea en Estrella Y y Delta Δ.', eq1: 'Y: V_L = \\sqrt{3} V_p', eq2: '\\Delta: I_L = \\sqrt{3} I_p', legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Multiplicador:</b> √3.</div>', controls: `<div class="control-group"><label>Ip: <span id="val-ip">10</span> A</label><input type="range" id="input-ip" min="2" max="25" value="10" oninput="drawSimYDelta()"></div>`, drawSchema: (ctx) => drawSchemaYDelta(ctx), drawSim: () => drawSimYDelta(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Corrientes en Delta." }, { title: "Ejercicio 1", type: "fill_blank", desc: "En Estrella equilibrada la corriente de neutro es ____.", answer: "cero" }, { title: "Ejercicio 2", type: "sim", desc: "Fija Ip = 5 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 5 }, { title: "Ejercicio 3", type: "sim", desc: "Fija Ip = 15 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 15 }, { title: "👾 BOSS", type: "sim", desc: "Ajusta Ip = 10 A.", check: () => parseFloat(document.getElementById('input-ip').value) === 10 }] },
            '5-3': { title: 'Misión 5.3: Cargas Trifásicas Desequilibradas', chap: 'chap-5', next: null, introChar: 'VL', speaker: 'Línea VL', narrative: 'Cargas desiguales generan corriente por el cable neutro IN ≠ 0.', eq1: '\\mathbf{I}_N = \\mathbf{I}_A + \\mathbf{I}_B + \\mathbf{I}_C', eq2: 'P_T = P_A + P_B + P_C', legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Retorno IN:</b> Corriente de neutro.</div>', controls: `<div class="control-group"><label>Desequilibrio: <span id="val-des">60</span>%</label><input type="range" id="input-des" min="0" max="100" value="60" oninput="drawSimDeseb()"></div>`, drawSchema: (ctx) => drawSchemaDeseb(ctx), drawSim: () => drawSimDeseb(), challenges: [{ title: "Ejemplo 🎬", type: "sim", desc: "Retorno por el neutro." }, { title: "Ejercicio 1", type: "fill_blank", desc: "El desequilibrio de fases genera corriente por el cable de ________.", answer: "neutro" }, { title: "Ejercicio 2", type: "sim", desc: "Ajusta al 20%.", check: () => parseFloat(document.getElementById('input-des').value) === 20 }, { title: "Ejercicio 3", type: "sim", desc: "Ajusta al 50%.", check: () => parseFloat(document.getElementById('input-des').value) === 50 }, { title: "👾 BOSS", type: "sim", desc: "Lleva el desequilibrio al 100%.", check: () => parseFloat(document.getElementById('input-des').value) === 100 }] }
        };

        /* --- ESTADO DEL JUEGO --- */
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
        let impedanceNodes = []; // Almacena coordenadas de las cajas Z en el camino

        window.onload = () => {
            updateUI();
            updateCablesAndElectron();
            window.addEventListener('resize', updateCablesAndElectron);
            requestAnimationFrame(animateElectron);
            initFormulasList();
        };

        function updateUI() {
            document.getElementById('player-lvl').innerText = `Nivel ${gameState.level}`;
            const ranks = ["Novato AC", "Analista de Filtros", "Especialista en Potencia", "Maestro Trifásico"];
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

        /* --- ZOOM EXTREMO CENTRADO --- */
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

            container.style.transform = `translateY(${offsetY}px) scale(1.4)`;

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

        /* --- TRAYECTORIA DE COBRE, CAJAS Z Y DETENCIÓN DEL ELECTRÓN --- */
        function updateCablesAndElectron() {
            const container = document.getElementById('world-container');
            const svg = document.getElementById('cable-svg');
            const rectW = container.getBoundingClientRect();

            svg.setAttribute('width', rectW.width);
            svg.setAttribute('height', rectW.height);

            let pathD = "";
            impedanceNodes = [];

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

            const pathCore = document.getElementById('path-copper-core');
            document.getElementById('path-copper-base').setAttribute('d', pathD);
            pathCore.setAttribute('d', pathD);
            document.getElementById('path-copper-texture').setAttribute('d', pathD);

            // DIBUJAR CAJAS DE IMPEDANCIA Z EN CADA INTERSECCIÓN DE NIVEL
            const impContainer = document.getElementById('path-impedances-container');
            impContainer.innerHTML = '';

            if (!currentZoomWorld && pathCore.getTotalLength) {
                const totalLen = pathCore.getTotalLength();
                const numLevels = 5;
                for (let k = 1; k <= numLevels; k++) {
                    const frac = k / (numLevels + 1);
                    const pt = pathCore.getPointAtLength(frac * totalLen);
                    impedanceNodes.push({ lenPos: frac * totalLen, x: pt.x, y: pt.y, label: `Z${k}` });

                    const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
                    g.setAttribute('class', 'path-impedance-group');
                    g.innerHTML = `
                        <rect x="${pt.x - 22}" y="${pt.y - 14}" width="44" height="28" />
                        <text x="${pt.x}" y="${pt.y}">Z${k}</text>
                    `;
                    impContainer.appendChild(g);
                }
            }
        }

        /* ANIMACIÓN DEL ELECTRÓN CON PARADA EN IMPEDANCIA */
        function animateElectron() {
            const path = document.getElementById('path-copper-core');
            const electronGroup = document.getElementById('electron-group');
            const electronCircle = document.getElementById('electron-circle');

            if (path && path.getTotalLength && path.getTotalLength() > 0) {
                const totalLen = path.getTotalLength();
                let currentDist = electronAnimProgress * totalLen;

                // Detectar si está cerca de alguna caja de impedancia
                let isNearZ = false;
                impedanceNodes.forEach(zNode => {
                    if (Math.abs(currentDist - zNode.lenPos) < 18) {
                        isNearZ = true;
                    }
                });

                if (isNearZ) {
                    electronCircle.classList.add('traversing');
                    electronAnimProgress += 0.0006; // Avance lento / atravesando impedancia
                } else {
                    electronCircle.classList.remove('traversing');
                    electronAnimProgress += 0.0025; // Avance normal
                }

                if (electronAnimProgress > 1) electronAnimProgress = 0;

                const pt = path.getPointAtLength(electronAnimProgress * totalLen);
                electronGroup.setAttribute('transform', `translate(${pt.x}, ${pt.y})`);
            }

            requestAnimationFrame(animateElectron);
        }

        /* --- APERTURA DE MISIÓN Y SUBVENTANA DE DIÁLOGO --- */
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
                    <div class="math-eq" style="color:var(--cyan); font-size:0.92rem;"></div>
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

        /* --- DIBUJO DE ESQUEMAS: CAJAS Z Y CIRCUITOS AMPLIADOS --- */
        function drawPillLabel(ctx, text, x, y, color) {
            ctx.font = '11px Orbitron';
            const w = ctx.measureText(text).width + 12;
            ctx.fillStyle = 'rgba(6, 9, 17, 0.9)';
            ctx.fillRect(x - w/2, y - 9, w, 18);
            ctx.strokeStyle = color; ctx.lineWidth = 1; ctx.strokeRect(x - w/2, y - 9, w, 18);
            ctx.fillStyle = color; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
            ctx.fillText(text, x, y);
        }

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
            ctx.clearRect(0,0,680,210);
            ctx.fillStyle='#131d30'; ctx.fillRect(80,45,60,120); ctx.fillRect(360,45,60,120);
            drawPillLabel(ctx, 'Polo Norte N', 110, 105, '#ff0055');
            drawPillLabel(ctx, 'Polo Sur S', 390, 105, '#00f3ff');
            
            ctx.strokeStyle='#ffb700'; ctx.lineWidth=3; ctx.beginPath(); ctx.arc(250,105,35,0,Math.PI*2); ctx.stroke();
            drawPillLabel(ctx, 'Generador AC v(t)', 250, 175, '#00ff66');
        }

        function drawSchemaKCL(ctx) {
            ctx.clearRect(0,0,680,210);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;

            drawACSource(ctx, 80, 105, 22, 'Vs = 120V');

            ctx.beginPath();
            ctx.moveTo(80, 83); ctx.lineTo(80, 40); ctx.lineTo(170, 40);
            ctx.moveTo(230, 40); ctx.lineTo(350, 40); ctx.lineTo(520, 40);
            ctx.moveTo(350, 40); ctx.lineTo(350, 75);
            ctx.moveTo(520, 40); ctx.lineTo(520, 75);
            
            ctx.moveTo(80, 127); ctx.lineTo(80, 170); ctx.lineTo(520, 170);
            ctx.moveTo(350, 135); ctx.lineTo(350, 170);
            ctx.moveTo(520, 135); ctx.lineTo(520, 170);
            ctx.stroke();

            drawZBox(ctx, 170, 25, 60, 30, 'Z1', '#ffb700');
            drawZBox(ctx, 320, 75, 60, 60, 'Z2', '#00ff66');
            drawZBox(ctx, 490, 75, 60, 60, 'Z3', '#ff0055');

            ctx.beginPath(); ctx.arc(350, 40, 6, 0, Math.PI*2); ctx.fillStyle='#ff0055'; ctx.fill();
            drawPillLabel(ctx, 'Nodo V1 (ΣI = 0)', 350, 18, '#ff0055');
        }

        function drawSchemaThevenin(ctx) {
            ctx.clearRect(0,0,680,210);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            drawACSource(ctx, 100, 105, 22, 'Vth');

            ctx.beginPath();
            ctx.moveTo(100, 83); ctx.lineTo(100, 40); ctx.lineTo(240, 40);
            ctx.moveTo(320, 40); ctx.lineTo(480, 40);
            ctx.moveTo(480, 170); ctx.lineTo(100, 170); ctx.lineTo(100, 127);
            ctx.stroke();

            drawZBox(ctx, 240, 25, 80, 30, 'ZTh', '#ffb700');
            drawZBox(ctx, 440, 75, 80, 60, 'ZL (Carga)', '#00ff66');
            drawPillLabel(ctx, 'Circuito Equivalente de Thévenin', 290, 140, '#00f3ff');
        }

        function drawSchemaRLC(ctx) {
            ctx.clearRect(0,0,680,210);
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
            drawPillLabel(ctx, 'Impedancia Serie Z_eq = ZR + ZL + ZC', 270, 120, '#ffb700');
        }

        function drawSchemaLPF(ctx) {
            ctx.clearRect(0,0,680,210);
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
            ctx.clearRect(0,0,680,210);
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

        function drawSchemaNotch(ctx) {
            ctx.clearRect(0,0,680,210);
            drawACSource(ctx, 80, 105, 22, 'Vin');
            drawZBox(ctx, 200, 25, 80, 30, 'Z_LC (Paralelo)', '#ff0055');
            drawZBox(ctx, 380, 80, 80, 50, 'ZR (R)', '#00f3ff');
            drawPillLabel(ctx, 'Filtro Notch Elimina f0', 300, 150, '#ff0055');
        }

        function drawSchemaPinst(ctx) {
            ctx.clearRect(0,0,680,210);
            drawACSource(ctx, 120, 105, 25, 'Fuente AC');
            drawZBox(ctx, 400, 75, 80, 60, 'Carga ZL', '#00ff66');
        }

        function drawSchemaTriang(ctx) {
            ctx.clearRect(0,0,680,210);
            drawZBox(ctx, 160, 60, 140, 80, 'Motor Inductivo (Q)', '#ff0055');
            drawZBox(ctx, 400, 60, 140, 80, 'Resistencia Útil (P)', '#00f3ff');
        }

        function drawSchemaFP(ctx) {
            ctx.clearRect(0,0,680,210);
            drawZBox(ctx, 140, 60, 150, 80, 'Planta Inductiva QL', '#00ff66');
            drawZBox(ctx, 390, 60, 150, 80, 'Capacitancia -QC', '#ff0055');
        }

        function drawSchemaPmax(ctx) {
            ctx.clearRect(0,0,680,210);
            drawZBox(ctx, 180, 65, 100, 70, 'ZTh (Interna)', '#ffb700');
            drawZBox(ctx, 400, 65, 100, 70, 'ZL = ZTh*', '#00ff66');
            drawPillLabel(ctx, 'Acoplamiento Conjugado ZL = RTh - jXTh', 340, 165, '#00ff66');
        }

        function drawSchema2Watt(ctx) {
            ctx.clearRect(0,0,680,210);
            drawZBox(ctx, 160, 40, 70, 40, 'W1', '#00f3ff');
            drawZBox(ctx, 160, 120, 70, 40, 'W2', '#00f3ff');
            drawZBox(ctx, 380, 60, 120, 90, 'Carga Trifásica', '#ffb700');
        }

        function drawSchemaTHD(ctx) {
            ctx.clearRect(0,0,680,210);
            drawZBox(ctx, 220, 65, 240, 80, 'Carga No Lineal (Armónicos)', '#ff0055');
        }

        function drawSchemaMutua(ctx) {
            ctx.clearRect(0,0,680,210);
            drawZBox(ctx, 180, 70, 70, 70, 'L1', '#ffb700');
            drawZBox(ctx, 430, 70, 70, 70, 'L2', '#00f3ff');
            drawPillLabel(ctx, 'Inductancia Mutua M = k√(L1 L2)', 340, 105, '#00ff66');
        }

        function drawSchemaDots(ctx) {
            ctx.clearRect(0,0,680,210);
            drawZBox(ctx, 200, 65, 80, 80, 'L1', '#00f3ff');
            drawZBox(ctx, 420, 65, 80, 80, 'L2', '#00f3ff');
        }

        function drawSchemaTransfo(ctx) {
            ctx.clearRect(0,0,680,210);
            drawPillLabel(ctx, 'Primario N1', 200, 105, '#ffb700');
            drawPillLabel(ctx, 'Secundario N2', 500, 105, '#00f3ff');
        }

        function drawSchemaTriGen(ctx) {
            ctx.clearRect(0,0,680,210);
            drawPillLabel(ctx, 'Generador Trifásico Simétrico 120°', 340, 180, '#ffb700');
        }

        function drawSchemaYDelta(ctx) {
            ctx.clearRect(0,0,680,210);
            drawZBox(ctx, 160, 70, 110, 70, 'Estrella Y', '#00f3ff');
            drawZBox(ctx, 420, 70, 110, 70, 'Delta Δ', '#00ff66');
        }

        function drawSchemaDeseb(ctx) {
            ctx.clearRect(0,0,680,210);
            drawZBox(ctx, 220, 70, 240, 70, 'Cargas Desiguales', '#ffb700');
            drawPillLabel(ctx, 'Corriente por Neutro IN ≠ 0', 340, 170, '#ff0055');
        }

        /* --- DIBUJO DE SIMULADORES EN PASOS DE RETO --- */
        function drawSimSenoidal() {
            const Vm = parseFloat(document.getElementById('input-vm').value);
            const phiDeg = parseFloat(document.getElementById('input-phi').value);
            document.getElementById('val-vm').innerText = Vm;
            document.getElementById('val-phi').innerText = phiDeg;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const cy = 100, cxPhasor = 110, phiRad = (phiDeg * Math.PI) / 180, r = Vm * 0.55;

            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 2.5; ctx.beginPath();
            for (let x = 240; x <= 630; x++) {
                const t = (x - 240) * 0.02;
                const y = cy - r * Math.sin(t + phiRad);
                if (x === 240) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();
        }

        function drawSimRLC() {
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 2.5; ctx.beginPath();
            for (let x = 40; x < 620; x++) {
                const y = 100 + 40 * Math.sin(x * 0.05);
                if (x === 40) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();
        }

        function drawSimKCL() {
            const Z2 = parseFloat(document.getElementById('input-z2').value);
            document.getElementById('val-z2').innerText = Z2;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
            ctx.fillStyle = '#00f3ff'; ctx.font = '13px Orbitron';
            ctx.fillText(`Admitancia Nodal Z2 = ${Z2} Ω`, 40, 50);
        }

        function drawSimThevenin() {
            const Rth = parseFloat(document.getElementById('input-rth').value);
            document.getElementById('val-rth').innerText = Rth;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
            ctx.fillStyle = '#ffb700'; ctx.font = '13px Orbitron';
            ctx.fillText(`Resistencia Equivalente RTh = ${Rth} Ω`, 40, 50);
        }

        function drawSimLPF() {
            const R = parseFloat(document.getElementById('input-r').value);
            document.getElementById('val-r').innerText = R;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 2.5; ctx.beginPath();
            for(let x=40; x<620; x++) {
                const y = 40 + (x - 40) * 0.2;
                if(x===40) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();
        }

        function drawSimHPF() {
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 2.5; ctx.beginPath();
            for(let x=40; x<620; x++) {
                const y = 160 - (x - 40) * 0.2;
                if(x===40) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();
        }

        function drawSimNotch() {
            const L = parseFloat(document.getElementById('input-ln').value);
            document.getElementById('val-ln').innerText = L;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
            ctx.strokeStyle = '#ff0055'; ctx.lineWidth = 2.5; ctx.beginPath();
            for(let x=40; x<620; x++) {
                const dist = Math.abs(x - 330);
                const y = 60 + 100 * Math.exp(-dist * 0.05);
                if(x===40) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();
        }

        function drawSimPinst() {
            const Vrms = parseFloat(document.getElementById('input-vrms').value);
            document.getElementById('val-vrms').innerText = Vrms;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
            ctx.fillStyle = '#ffb700'; ctx.font = '13px Orbitron';
            ctx.fillText(`Potencia Promedio P = ${(Vrms*1.5).toFixed(0)} W`, 40, 50);
        }

        function drawSimTriang() {
            const QL = parseFloat(document.getElementById('input-ql').value);
            document.getElementById('val-ql').innerText = QL;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
            ctx.fillStyle = '#00ff66'; ctx.font = '13px Orbitron';
            ctx.fillText(`Potencia Reactiva QL = ${QL} VAR`, 40, 50);
        }

        function drawSimFP() {
            const QC = parseFloat(document.getElementById('input-qc').value);
            document.getElementById('val-qc').innerText = QC;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
            ctx.fillStyle = '#00ff66'; ctx.font = '13px Orbitron';
            ctx.fillText(`Inyección QC = ${QC} VAR`, 40, 50);
        }

        function drawSimPmax() {
            const RL = parseFloat(document.getElementById('input-rl').value);
            document.getElementById('val-rl').innerText = RL;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 2.5; ctx.beginPath();
            for(let x=40; x<620; x++) {
                const r = (x - 40) * 0.2;
                const p = (100 * r) / Math.pow(r + 50, 2);
                const y = 180 - p * 300;
                if(x===40) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();
        }

        function drawSim2Watt() {
            const theta = parseFloat(document.getElementById('input-theta').value);
            document.getElementById('val-theta').innerText = theta;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
            ctx.fillStyle = '#00f3ff'; ctx.font = '13px Orbitron';
            ctx.fillText(`Ángulo de Carga θ = ${theta}°`, 40, 50);
        }

        function drawSimTHD() {
            const h3 = parseFloat(document.getElementById('input-h3').value);
            document.getElementById('val-h3').innerText = h3;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
            ctx.strokeStyle = '#ff0055'; ctx.lineWidth = 2.5; ctx.beginPath();
            for(let x=40; x<620; x++) {
                const t = (x - 40) * 0.03;
                const y = 100 - 50 * Math.sin(t) - (h3*0.5) * Math.sin(3*t);
                if(x===40) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();
        }

        function drawSimMutua() {
            const k = parseFloat(document.getElementById('input-k').value);
            document.getElementById('val-k').innerText = k;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
        }

        function drawSimDots() {
            const dir = parseInt(document.getElementById('input-dir').value);
            document.getElementById('val-dir').innerText = dir === 1 ? 'Mismo Punto' : 'Punto Opuesto';
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
        }

        function drawSimTransfo() {
            const a = parseFloat(document.getElementById('input-a').value);
            document.getElementById('val-a').innerText = a;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
        }

        function drawSimTriGen() {
            const Vp = parseFloat(document.getElementById('input-vp').value);
            document.getElementById('val-vp').innerText = Vp;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
        }

        function drawSimYDelta() {
            const Ip = parseFloat(document.getElementById('input-ip').value);
            document.getElementById('val-ip').innerText = Ip;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
        }

        function drawSimDeseb() {
            const des = parseFloat(document.getElementById('input-des').value);
            document.getElementById('val-des').innerText = des;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);
        }
    </script>
</body>
</html>