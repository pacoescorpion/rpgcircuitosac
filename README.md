<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Análisis de Circuitos AC, Filtros & Números Complejos</title>
    <!-- KaTeX CDN -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.css">
    <script src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/contrib/auto-render.min.js"></script>
    <!-- Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700;900&family=Rajdhani:wght@500;600;700&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --bg-dark: #070c14;
            --panel-bg: #0d1524;
            --panel-border: rgba(0, 243, 255, 0.35);
            --cyan: #00f3ff;
            --green: #00ff66;
            --yellow: #ffb700;
            --red: #ff0055;
            --purple: #9d00ff;
            --copper-light: #e5a059;
            --copper-mid: #b87333;
            --copper-dark: #5c3a1e;
            --proto-pin: #1e293b;
            --text-main: #e2e8f0;
            --text-muted: #94a3b8;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }

        body {
            background-color: var(--bg-dark);
            color: var(--text-main);
            font-family: 'Rajdhani', sans-serif;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        /* --- HEADER GLOBAL HUD --- */
        header {
            background: rgba(7, 12, 20, 0.95);
            border-bottom: 2px solid var(--cyan);
            box-shadow: 0 0 20px rgba(0, 243, 255, 0.2);
            padding: 10px 25px;
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

        /* --- ESCENARIO DE PROTOBOARD CON ZOOM EXTREMO --- */
        #viewport-stage {
            position: relative;
            width: 100vw;
            height: 100vh;
            overflow: hidden;
            background-color: #0b101d;
            /* Textura de Protoboard */
            background-image: 
                radial-gradient(circle, #25334d 2.5px, transparent 3px),
                linear-gradient(to right, rgba(255,255,255,0.02) 1px, transparent 1px),
                linear-gradient(to bottom, rgba(255,255,255,0.02) 1px, transparent 1px);
            background-size: 24px 24px, 24px 24px, 24px 24px;
            background-position: center;
        }

        #world-container {
            position: absolute;
            top: 50%; left: 50%;
            width: 1100px; height: 1600px;
            transform: translate(-50%, -50%) scale(1);
            transform-origin: center center;
            transition: transform 0.8s cubic-bezier(0.25, 1, 0.5, 1);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-around;
            padding: 80px 0;
        }

        #cable-svg {
            position: absolute;
            top: 0; left: 0;
            width: 100%; height: 100%;
            pointer-events: none;
            z-index: 1;
        }

        .copper-base { fill: none; stroke: var(--copper-dark); stroke-width: 12; stroke-linecap: round; }
        .copper-core { fill: none; stroke: url(#copper-grad); stroke-width: 7; stroke-linecap: round; filter: drop-shadow(0 0 6px rgba(184, 115, 51, 0.6)); }
        .copper-texture { fill: none; stroke: var(--copper-light); stroke-width: 2; stroke-dasharray: 6, 4; opacity: 0.7; }

        .electron-core { fill: #00f3ff; filter: drop-shadow(0 0 8px #00f3ff) drop-shadow(0 0 16px #00f3ff); }
        .electron-text { font-family: 'Orbitron', sans-serif; font-size: 13px; font-weight: 900; fill: #070c14; text-anchor: middle; dominant-baseline: central; }

        .terminal-node {
            position: relative;
            z-index: 2;
            width: 60px; height: 60px;
            border-radius: 50%; background: #0f172a;
            border: 2px solid var(--copper-light);
            display: flex; flex-direction: column;
            justify-content: center; align-items: center;
            box-shadow: 0 0 20px rgba(184, 115, 51, 0.4);
        }
        .terminal-label { font-family: 'Orbitron', sans-serif; font-size: 0.6rem; color: var(--copper-light); margin-top: 2px; }

        /* MUNDOS Y ISLAS TIPO GEOMETRY DASH WORLD */
        .world-island-stage {
            position: relative;
            z-index: 2;
            width: 90%;
            min-height: 260px;
            background: rgba(15, 23, 42, 0.85);
            border: 2px solid var(--panel-border);
            border-radius: 20px;
            box-shadow: 0 0 35px rgba(0,0,0,0.8), inset 0 0 15px rgba(0, 243, 255, 0.05);
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
            transition: all 0.5s ease;
        }

        .world-island-stage.locked { opacity: 0.45; filter: grayscale(1); }

        .world-header-badge {
            font-family: 'Orbitron', sans-serif;
            font-size: 1.1rem;
            font-weight: 800;
            color: var(--yellow);
            text-shadow: 0 0 10px rgba(255, 183, 0, 0.5);
            margin-bottom: 15px;
            display: flex; gap: 10px; align-items: center;
        }

        /* RECORRIDO DE NIVELES EN ZIGZAG DENTRO DE CADA ISLA */
        .island-levels-path {
            display: flex;
            justify-content: space-around;
            align-items: center;
            width: 100%;
            flex-wrap: wrap;
            gap: 20px;
            margin-top: 10px;
        }

        .level-pin-node {
            width: 120px;
            height: 120px;
            border-radius: 50%;
            background: #090e17;
            border: 3px solid var(--panel-border);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            position: relative;
            z-index: 5;
            transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            box-shadow: 0 0 20px rgba(0,0,0,0.9);
        }

        .level-pin-node:hover:not(.locked) {
            transform: scale(1.12);
            border-color: var(--cyan);
            box-shadow: 0 0 30px var(--cyan);
        }

        .level-pin-node.completed { border-color: var(--green); box-shadow: 0 0 25px var(--green); }
        .level-pin-node.locked { opacity: 0.35; cursor: not-allowed; border-color: var(--text-muted); }

        .level-icon { font-size: 2rem; margin-bottom: 4px; }
        .level-title-tag { font-family: 'Orbitron', sans-serif; font-size: 0.65rem; text-align: center; color: #fff; padding: 0 5px; }

        /* BOTÓN DE SALIDA DE ZOOM */
        .btn-exit-zoom {
            position: fixed;
            top: 25px; left: 25px;
            z-index: 600;
            font-family: 'Orbitron', sans-serif;
            background: rgba(255, 0, 85, 0.25);
            border: 2px solid var(--red);
            color: #fff;
            padding: 10px 20px;
            border-radius: 20px;
            cursor: pointer;
            font-size: 0.85rem;
            display: none;
            box-shadow: 0 0 20px rgba(255, 0, 85, 0.5);
            backdrop-filter: blur(8px);
            transition: all 0.2s;
        }
        .btn-exit-zoom:hover { background: var(--red); box-shadow: 0 0 30px var(--red); }

        /* --- MODAL CON COLUMNA IZQUIERDA DE 1/4 PARA ESQUEMA Y DERECHA DE 3/4 PARA CONTENIDO --- */
        .modal-overlay {
            position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
            background: rgba(4, 7, 14, 0.93); backdrop-filter: blur(10px);
            z-index: 1000; display: flex; justify-content: center; align-items: center;
            opacity: 0; pointer-events: none; transition: opacity 0.25s ease;
        }
        .modal-overlay.active { opacity: 1; pointer-events: all; }

        .modal-card-wide {
            background: var(--panel-bg);
            border: 2px solid var(--cyan);
            border-radius: 14px;
            width: 95vw; max-width: 1250px;
            height: 90vh;
            display: flex; flex-direction: column;
            overflow: hidden;
            box-shadow: 0 0 45px rgba(0, 243, 255, 0.3);
        }

        .modal-header {
            padding: 12px 25px;
            border-bottom: 1px solid var(--panel-border);
            display: flex; justify-content: space-between; align-items: center;
            background: rgba(0, 243, 255, 0.05);
        }
        .modal-header h3 { font-family: 'Orbitron', sans-serif; color: var(--cyan); font-size: 1.1rem; }
        .close-btn { background: none; border: none; color: var(--text-muted); font-size: 1.8rem; cursor: pointer; }
        .close-btn:hover { color: var(--red); }

        /* ESTRUCTURA DOS COLUMNAS (1/4 IZQ, 3/4 DER) */
        .modal-body-layout {
            display: flex;
            width: 100%;
            height: calc(100% - 55px);
            overflow: hidden;
        }

        /* COLUMNA IZQUIERDA (25% ANCHO EXCLUSIVO PARA ESQUEMAS SIN ENCOGER) */
        .modal-sidebar-left {
            flex: 0 0 25%;
            background: #040812;
            border-right: 1px solid var(--panel-border);
            padding: 15px;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 15px;
            overflow-y: auto;
        }

        .sidebar-schema-box {
            width: 100%;
            background: #070d1a;
            border: 1px solid rgba(255,255,255,0.1);
            border-radius: 8px;
            padding: 10px;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 8px;
        }

        .sidebar-schema-title {
            font-family: 'Orbitron', sans-serif;
            font-size: 0.75rem;
            color: var(--cyan);
            align-self: flex-start;
        }

        /* COLUMNA DERECHA (75% ANCHO PARA CONTENIDO, TEORÍA, SIMULADOR Y RETOS) */
        .modal-content-right {
            flex: 1;
            padding: 20px;
            display: flex;
            flex-direction: column;
            gap: 15px;
            overflow-y: auto;
        }

        /* BARRA DE PROGRESO DE SUBTEMA */
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

        .math-card {
            background: rgba(255, 255, 255, 0.02); border: 1px solid rgba(255, 255, 255, 0.08);
            padding: 10px 14px; border-radius: 6px; display: flex; flex-direction: column; gap: 4px;
        }
        .math-title { font-family: 'Orbitron', sans-serif; font-size: 0.75rem; color: var(--yellow); }
        .math-eq { font-size: 1.05rem; padding: 2px 0; overflow-x: auto; }

        /* PREGUNTA COMPLETAR ESPACIO EN BLANCO */
        .fill-blank-card {
            background: rgba(157, 0, 255, 0.08);
            border: 1px solid var(--purple);
            border-radius: 8px;
            padding: 18px;
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .fill-blank-input {
            background: #040812;
            border: 1px solid var(--cyan);
            color: var(--cyan);
            font-family: 'Orbitron', sans-serif;
            font-size: 1rem;
            padding: 8px 12px;
            border-radius: 6px;
            outline: none;
            width: 220px;
        }

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

        /* SUBMENÚ INTERACTIVO DE FÓRMULAS CON DERIVACIÓN NARRATIVA */
        .formulas-submenu-container {
            display: flex; height: 100%; gap: 15px;
        }
        .formulas-list-sidebar {
            flex: 0 0 32%; background: #040812; border-right: 1px solid var(--panel-border);
            padding: 10px; display: flex; flex-direction: column; gap: 8px; overflow-y: auto;
        }
        .formula-item-btn {
            background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.1);
            border-radius: 6px; padding: 10px; cursor: pointer; transition: all 0.2s;
            font-family: 'Orbitron', sans-serif; font-size: 0.8rem; color: var(--text-main);
        }
        .formula-item-btn:hover { border-color: var(--cyan); background: rgba(0,243,255,0.08); }
        .formula-item-btn.active { border-color: var(--cyan); background: rgba(0,243,255,0.15); color: var(--cyan); }

        .formula-derivation-panel {
            flex: 1; background: #070d1a; border: 1px solid var(--panel-border);
            border-radius: 8px; padding: 18px; display: flex; flex-direction: column; gap: 12px; overflow-y: auto;
        }
    </style>
</head>
<body>

    <!-- HUD GLOBAL (SE OCULTA EN ZOOM) -->
    <header id="global-hud">
        <div class="hud-title">⚡ CIRCUITY RPG: AC, FILTROS & DERIVACIONES</div>
        <div class="hud-controls">
            <button class="btn-hud btn-formulas" onclick="openFormulasModal()">📜 EQUIPO DE FÓRMULAS</button>
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

    <!-- BOTÓN RECTANGULAR PARA SALIR DEL ZOOM DE MUNDO -->
    <button class="btn-exit-zoom" id="btn-exit-zoom" onclick="exitWorldZoom()">🔍 SALIR DEL MUNDO</button>

    <!-- ESCENARIO PRINCIPAL (PROTOBOARD) -->
    <div id="viewport-stage">
        <div id="world-container">
            
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

            <!-- NODO TIERRA (PIN INICIAL) -->
            <div class="terminal-node" id="node-ground">
                <svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="#e5a059" stroke-width="2.5">
                    <line x1="12" y1="3" x2="12" y2="13" />
                    <line x1="4" y1="13" x2="20" y2="13" />
                    <line x1="7" y1="17" x2="17" y2="17" />
                    <line x1="10" y1="21" x2="14" y2="21" />
                </svg>
                <span class="terminal-label">TIERRA</span>
            </div>

            <!-- ISLA / MUNDO 1: SENOIDALES, FILTROS Y COMPLEJOS -->
            <div class="world-island-stage" id="world-1">
                <div class="world-header-badge" onclick="zoomIntoWorld('world-1')">
                    <span>🌴 MUNDO 1: Senoidales, Complejos & Filtros Pasivos</span>
                </div>
                <div class="island-levels-path">
                    <div class="level-pin-node unlocked" id="sub-1-1" onclick="openQuest('1-1')">
                        <div class="level-icon">🌊</div>
                        <div class="level-title-tag">1.1 Onda & Fasores</div>
                    </div>
                    <div class="level-pin-node locked" id="sub-1-2" onclick="openQuest('1-2')">
                        <div class="level-icon">📉</div>
                        <div class="level-title-tag">1.2 Filtro RC Paso Bajo</div>
                    </div>
                    <div class="level-pin-node locked" id="sub-1-3" onclick="openQuest('1-3')">
                        <div class="level-icon">📈</div>
                        <div class="level-title-tag">1.3 Filtro RC Paso Alto</div>
                    </div>
                </div>
            </div>

            <!-- ISLA / MUNDO 2: POTENCIA Y COMPENSACIÓN -->
            <div class="world-island-stage locked" id="world-2">
                <div class="world-header-badge" onclick="zoomIntoWorld('world-2')">
                    <span>⚡ MUNDO 2: Potencia Compleja Vectorial</span>
                </div>
                <div class="island-levels-path">
                    <div class="level-pin-node locked" id="sub-2-1" onclick="openQuest('2-1')">
                        <div class="level-icon">🔋</div>
                        <div class="level-title-tag">2.1 Potencia RMS</div>
                    </div>
                    <div class="level-pin-node locked" id="sub-2-2" onclick="openQuest('2-2')">
                        <div class="level-icon">📐</div>
                        <div class="level-title-tag">2.2 Triángulo P-Q-S</div>
                    </div>
                    <div class="level-pin-node locked" id="sub-2-3" onclick="openQuest('2-3')">
                        <div class="level-icon">🏭</div>
                        <div class="level-title-tag">2.3 Corrección FP</div>
                    </div>
                </div>
            </div>

            <!-- ISLA / MUNDO 3: ACOPLO Y TRANSFORMADORES -->
            <div class="world-island-stage locked" id="world-3">
                <div class="world-header-badge" onclick="zoomIntoWorld('world-3')">
                    <span>🧲 MUNDO 3: Acoplamiento Magnético</span>
                </div>
                <div class="island-levels-path">
                    <div class="level-pin-node locked" id="sub-3-1" onclick="openQuest('3-1')">
                        <div class="level-icon">🌀</div>
                        <div class="level-title-tag">3.1 Inductancia Mutua</div>
                    </div>
                    <div class="level-pin-node locked" id="sub-3-2" onclick="openQuest('3-2')">
                        <div class="level-icon">🔴</div>
                        <div class="level-title-tag">3.2 Regla de Puntos</div>
                    </div>
                    <div class="level-pin-node locked" id="sub-3-3" onclick="openQuest('3-3')">
                        <div class="level-icon">🔌</div>
                        <div class="level-title-tag">3.3 Transformador Ideal</div>
                    </div>
                </div>
            </div>

            <!-- NODO FUENTE AC (PIN FINAL) -->
            <div class="terminal-node" id="node-source">
                <svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="#e5a059" stroke-width="2">
                    <circle cx="12" cy="12" r="9" />
                    <path d="M 7 12 Q 9.5 7 12 12 T 17 12" />
                </svg>
                <span class="terminal-label">FUENTE AC</span>
            </div>

        </div>
    </div>

    <!-- MODAL DE MISIÓN / SUBTEMA CON COLUMNA IZQUIERDA DEL 25% -->
    <div class="modal-overlay" id="quest-modal">
        <div class="modal-card-wide">
            <div class="modal-header">
                <h3 id="modal-title">Título de Misión</h3>
                <button class="close-btn" onclick="closeQuest()">&times;</button>
            </div>
            
            <div class="modal-body-layout">
                
                <!-- COLUMNA IZQUIERDA 25% EXCLUSIVA PARA ESQUEMA TéCNICO Y VARIABLES -->
                <div class="modal-sidebar-left">
                    <div class="sidebar-schema-box">
                        <div class="sidebar-schema-title">🔍 ESQUEMA CIRCUITAL</div>
                        <canvas id="schema-canvas" width="240" height="200"></canvas>
                    </div>
                    <div class="math-card" style="width:100%;">
                        <div class="math-title">Variables Activas</div>
                        <div id="sidebar-vars-info" style="font-size:0.8rem; color:var(--text-muted);">
                            Revisa el comportamiento gráfico.
                        </div>
                    </div>
                </div>

                <!-- COLUMNA DERECHA 75% PARA CONTENIDO, TEORÍA Y RETOS -->
                <div class="modal-content-right">
                    
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
                                <span style="font-size:0.65rem; color:var(--text-muted);">(Clic para revelar todo)</span>
                            </div>
                            <div class="narrative-text typing-cursor" id="narrative-text-el"></div>
                        </div>

                        <div class="math-card">
                            <div class="math-title">Ecuación de Origen Básica</div>
                            <div class="math-eq" id="eq-1"></div>
                        </div>

                        <div class="math-card">
                            <div class="math-title">Derivación / Formulación de Análisis</div>
                            <div class="math-eq" id="eq-2"></div>
                        </div>

                        <button class="nav-btn" onclick="nextSubstep()">
                            Continuar al Ejemplo ➔
                        </button>
                    </div>

                    <!-- PASO 1 A 5: SIMULADOR, FILL-IN-BLANK Y EJERCICIOS -->
                    <div class="step-view" id="step-sim-view" style="display:none; flex-direction:column; gap:12px;">
                        
                        <div class="challenge-box" id="challenge-banner">
                            <div class="challenge-title" id="challenge-title">Modo Ejemplo Guiado</div>
                            <div id="challenge-desc" style="font-size:0.88rem; color:#fff;">Observa cómo responde el simulador dinámicamente.</div>
                        </div>

                        <!-- BANNER PREGUNTA COMPLETAR ESPACIO EN BLANCO -->
                        <div class="fill-blank-card" id="fill-blank-container" style="display:none;">
                            <div style="font-family:'Orbitron'; color:var(--yellow); font-size:0.85rem;">COMPLETA EL CONCEPTO:</div>
                            <div id="fill-blank-text" style="font-size:0.95rem; color:#fff;"></div>
                            <div style="display:flex; gap:10px; align-items:center;">
                                <input type="text" class="fill-blank-input" id="fill-blank-input" placeholder="Escribe aquí..." autocomplete="off">
                                <span id="fill-blank-feedback" style="font-size:0.85rem; font-weight:700;"></span>
                            </div>
                        </div>

                        <div class="sim-box" id="sim-canvas-box">
                            <canvas id="sim-canvas" width="700" height="210"></canvas>
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
    </div>

    <!-- MODAL EQUIPO DE FÓRMULAS CON SUBMENÚ INTERACTIVO NARRATIVO -->
    <div class="modal-overlay" id="formulas-modal">
        <div class="modal-card-wide" style="max-width: 1000px; height: 85vh;">
            <div class="modal-header">
                <h3>📜 EQUIPO DE FÓRMULAS & DERIVACIONES MATEMÁTICAS</h3>
                <button class="close-btn" onclick="closeFormulasModal()">&times;</button>
            </div>
            <div class="formulas-submenu-container">
                <div class="formulas-list-sidebar" id="formulas-list-sidebar">
                    <!-- Lista de fórmulas se genera dinámicamente -->
                </div>
                <div class="formula-derivation-panel">
                    <h4 id="formula-detail-title" style="font-family:'Orbitron'; color:var(--yellow); font-size:0.95rem;">
                        Selecciona una fórmula
                    </h4>
                    <div id="formula-detail-eq" class="math-eq" style="color:var(--cyan);"></div>
                    <div class="narrative-box" onclick="skipFormulaTypewriter()">
                        <div class="narrative-header">
                            <span>DEMOSTRACIÓN PASO A PASO DESDE LO BÁSICO</span>
                        </div>
                        <div class="narrative-text typing-cursor" id="formula-narrative-text">
                            Haz clic en cualquier término de la izquierda para ver de dónde sale matemáticamente desde las leyes más fundamentales.
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        /* --- BASE DE DATOS DE FÓRMULAS DERIVADAS PASO A PASO --- */
        const formulasDB = [
            {
                id: 'lpf',
                title: "Filtro RC Paso Bajo (LPF)",
                eq: "H(j\\omega) = \\frac{1}{1 + j\\omega RC} = \\frac{1}{1 + j\\frac{\\omega}{\\omega_c}}",
                derivation: "<b>1. Ley de Kirchhoff (LTK):</b> $V_{in} = I R + V_{out}$<br>" +
                            "<b>2. Impedancia del Condensador:</b> $Z_C = \\frac{1}{j\\omega C} = -\\frac{j}{\\omega C}$<br>" +
                            "<b>3. Divisor de Tensión Complejo:</b> $V_{out} = V_{in} \\frac{Z_C}{R + Z_C} = V_{in} \\frac{\\frac{1}{j\\omega C}}{R + \\frac{1}{j\\omega C}}$<br>" +
                            "<b>4. Simplificación multiplicando por $j\\omega C$:</b> $H(j\\omega) = \\frac{V_{out}}{V_{in}} = \\frac{1}{1 + j\\omega RC}$<br>" +
                            "<b>5. Frecuencia de corte y Normalización:</b> Definimos $\\omega_c = \\frac{1}{RC}$ y frecuencia normalizada $\\Omega = \\frac{\\omega}{\\omega_c} = \\frac{f}{f_c}$, de modo que $H(j\\Omega) = \\frac{1}{1 + j\\Omega}$."
            },
            {
                id: 'hpf',
                title: "Filtro RC Paso Alto (HPF)",
                eq: "H(j\\omega) = \\frac{j\\omega RC}{1 + j\\omega RC} = \\frac{j\\frac{\\omega}{\\omega_c}}{1 + j\\frac{\\omega}{\\omega_c}}",
                derivation: "<b>1. Divisor de Tensión en la Resistencia:</b> $V_{out} = V_{in} \\frac{R}{R + Z_C}$<br>" +
                            "<b>2. Sustituyendo $Z_C = \\frac{1}{j\\omega C}$:</b> $V_{out} = V_{in} \\frac{R}{R + \\frac{1}{j\\omega C}}$<br>" +
                            "<b>3. Multiplicando numerador y denominador por $j\\omega C$:</b> $H(j\\omega) = \\frac{j\\omega RC}{1 + j\\omega RC}$<br>" +
                            "<b>4. Comportamiento Límite:</b> A $\\omega \\to 0$ (DC), $H \\to 0$ (Bloquea). A $\\omega \\to \\infty$, $H \\to 1$ (Pasa)."
            },
            {
                id: 'complex',
                title: "Números Imaginarios & Operaciones",
                eq: "z = a + j b = |z| e^{j\\theta} = |z|(\\cos\\theta + j\\sin\\theta)",
                derivation: "<b>1. Unidad Imaginaria:</b> $j = \\sqrt{-1} \\implies j^2 = -1, \\quad \\frac{1}{j} = -j$<br>" +
                            "<b>2. Magnitud (Módulo):</b> $|z| = \\sqrt{a^2 + b^2}$<br>" +
                            "<b>3. Ángulo de Fase:</b> $\\theta = \\arctan\\left(\\frac{b}{a}\\right)$<br>" +
                            "<b>4. Identidad de Euler:</b> $e^{j\\theta} = \\cos\\theta + j\\sin\\theta$<br>" +
                            "<b>5. División Compleja:</b> $\\frac{a + jb}{c + jd} = \\frac{(a+jb)(c-jd)}{c^2 + d^2}$."
            },
            {
                id: 'phasor',
                title: "Transformación Fasorial",
                eq: "v(t) = V_m \\cos(\\omega t + \\phi) \\iff \\mathbf{V} = V_{rms} \\angle \\phi",
                derivation: "<b>1. Forma Senoidal en el Tiempo:</b> $v(t) = V_m \\cos(\\omega t + \\phi)$<br>" +
                            "<b>2. Notación de Parte Real de Euler:</b> $v(t) = \\text{Re}\\{V_m e^{j(\\omega t + \\phi)}\\} = \\text{Re}\\{V_m e^{j\\phi} e^{j\\omega t}\\}$<br>" +
                            "<b>3. Extracción del Fasor:</b> Suprimiendo la rotación $e^{j\\omega t}$, obtenemos el Fasor de Pico $\\mathbf{V}_m = V_m e^{j\\phi} = V_m \\angle \\phi$. En análisis eléctrico usaremos el valor RMS: $\\mathbf{V} = \\frac{V_m}{\\sqrt{2}} \\angle \\phi$."
            },
            {
                id: 'impedance',
                title: "Impedancia Compleja RLC",
                eq: "\\mathbf{Z} = R + j\\left(\\omega L - \\frac{1}{\\omega C}\\right)",
                derivation: "<b>1. Resistencia Puro:</b> $\\mathbf{Z}_R = R \\angle 0^\\circ = R$<br>" +
                            "<b>2. Inductor:</b> $v_L(t) = L \\frac{di}{dt} \\implies \\mathbf{V}_L = j\\omega L \\mathbf{I} \\implies \\mathbf{Z}_L = j\\omega L$<br>" +
                            "<b>3. Condensador:</b> $i_C(t) = C \\frac{dv}{dt} \\implies \\mathbf{I}_C = j\\omega C \\mathbf{V} \\implies \\mathbf{Z}_C = \\frac{1}{j\\omega C} = -j\\frac{1}{\\omega C}$<br>" +
                            "<b>4. Serie RLC:</b> $\\mathbf{Z}_{total} = \\mathbf{Z}_R + \\mathbf{Z}_L + \\mathbf{Z}_C = R + j\\left(\\omega L - \\frac{1}{\\omega C}\\right)$."
            },
            {
                id: 'power',
                title: "Potencia Compleja Vectorial",
                eq: "\\mathbf{S} = P + jQ = \\mathbf{V}_{rms} \\mathbf{I}_{rms}^*",
                derivation: "<b>1. Potencia Promedio Activa:</b> $P = V_{rms} I_{rms} \\cos(\\theta_v - \\theta_i)$ [Watts]<br>" +
                            "<b>2. Potencia Reactiva:</b> $Q = V_{rms} I_{rms} \\sin(\\theta_v - \\theta_i)$ [VAR]<br>" +
                            "<b>3. Forma Compleja:</b> $\\mathbf{S} = \\mathbf{V}_{rms} \\mathbf{I}_{rms}^* = (V_{rms}\\angle\\theta_v)(I_{rms}\\angle -\\theta_i) = V_{rms} I_{rms} \\angle(\\theta_v - \\theta_i)$<br>" +
                            "<b>4. Magnitud Aparente:</b> $|\\mathbf{S}| = \\sqrt{P^2 + Q^2}$ [VA]."
            }
        ];

        /* --- BASE DE DATOS DE SUBTEMAS Y RETOS --- */
        const questsDB = {
            '1-1': {
                title: 'Misión 1.1: Onda Senoidal, Fasores & Números Complejos', world: 'world-1', next: '1-2',
                narrative: 'Una fuente AC produce una onda senoidal $v(t) = V_m \\cos(\\omega t + \\phi)$. Para operarla matemáticamente, utilizamos el **Fasor** complejo $\\mathbf{V} = V_{rms} \\angle \\phi$. Recuerda que $j = \\sqrt{-1}$ rota vectores $90^\\circ$ en el plano complejo.',
                eq1: 'v(t) = V_m \\cdot \\cos(\\omega t + \\phi)', eq2: '\\mathbf{V} = \\frac{V_m}{\\sqrt{2}} e^{j\\phi} = V_{rms} \\angle \\phi',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Fasor V:</b> Vector de magnitud y fase.</div><div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Onda Senoidal v(t):</b> Dominio del tiempo.</div>',
                controls: `<div class="control-group"><label>Amplitud Vm: <span id="val-vm">100</span> V</label><input type="range" id="input-vm" min="30" max="150" value="100" oninput="drawSimSenoidal()"></div><div class="control-group"><label>Desfase ϕ: <span id="val-phi">45</span>°</label><input type="range" id="input-phi" min="-180" max="180" value="45" oninput="drawSimSenoidal()"></div>`,
                drawSchema: (ctx) => drawSchemaGen(ctx), drawSim: () => drawSimSenoidal(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Observa la equivalencia entre el vector fasorial y la onda de tiempo." },
                    { title: "Ejercicio 1: Pregunta Conceptual", type: "fill_blank", desc: "El operador imaginario j equivale a la raíz cuadrada de ____.", answer: "-1" },
                    { title: "Ejercicio 2: Ajuste de Amplitud", type: "sim", desc: "Ajusta la Amplitud Vm exactamente a 120 V.", check: () => parseFloat(document.getElementById('input-vm').value) === 120 },
                    { title: "Ejercicio 3: Cuadratura", type: "sim", desc: "Coloca el desfase en cuadratura positiva (+90°).", check: () => parseFloat(document.getElementById('input-phi').value) === 90 },
                    { title: "👾 DESAFÍO BOSS: Generar Vrms = 100V", type: "sim", desc: "Fija Vm = 141 V y desfase a +45°.", check: () => parseFloat(document.getElementById('input-vm').value) === 141 && parseFloat(document.getElementById('input-phi').value) === 45 }
                ]
            },
            '1-2': {
                title: 'Misión 1.2: Filtro Paso Bajo RC (LPF) de 1er Orden', world: 'world-1', next: '1-3',
                narrative: 'Partiendo del divisor de tensión $V_{out} = V_{in} \\frac{Z_C}{R + Z_C}$ con $Z_C = \\frac{1}{j\\omega C}$, derivamos $H(j\\omega) = \\frac{1}{1 + j\\omega RC}$. Definimos la **frecuencia de corte** $f_c = \\frac{1}{2\\pi RC}$ donde la ganancia cae a $-3\\text{ dB}$ ($0.707$).',
                eq1: 'H(j\\omega) = \\frac{V_{out}}{V_{in}} = \\frac{1}{1 + j\\omega RC}', eq2: 'H(j\\Omega) = \\frac{1}{1 + j\\Omega}, \\quad \\Omega = \\frac{f}{f_c}, \\quad f_c = \\frac{1}{2\\pi RC}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Respuesta en Frecuencia LPF:</b> Atenuación de -20 dB/década.</div>',
                controls: `<div class="control-group"><label>Resistencia R: <span id="val-r">1000</span> Ω</label><input type="range" id="input-r" min="100" max="5000" step="100" value="1000" oninput="drawSimLPF()"></div><div class="control-group"><label>Capacitancia C: <span id="val-c">0.1</span> µF</label><input type="range" id="input-c" min="0.01" max="1" step="0.01" value="0.1" oninput="drawSimLPF()"></div>`,
                drawSchema: (ctx) => drawSchemaLPF(ctx), drawSim: () => drawSimLPF(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "El simulador altera la frecuencia de corte fc dinámicamente." },
                    { title: "Ejercicio 1: Pregunta Conceptual", type: "fill_blank", desc: "En la frecuencia de corte, la ganancia de potencia cae a ____ dB.", answer: "-3" },
                    { title: "Ejercicio 2: Ajuste de Resistencia", type: "sim", desc: "Fija la Resistencia R = 2000 Ω.", check: () => parseFloat(document.getElementById('input-r').value) === 2000 },
                    { title: "Ejercicio 3: Ajuste de Capacitancia", type: "sim", desc: "Ajusta C = 0.5 µF.", check: () => Math.abs(parseFloat(document.getElementById('input-c').value) - 0.5) < 0.01 },
                    { title: "👾 DESAFÍO BOSS: Frecuencia de Corte fc ≈ 1591 Hz", type: "sim", desc: "Busca la combinación R = 1000 Ω y C = 0.1 µF.", check: () => parseFloat(document.getElementById('input-r').value) === 1000 && Math.abs(parseFloat(document.getElementById('input-c').value) - 0.1) < 0.01 }
                ]
            },
            '1-3': {
                title: 'Misión 1.3: Filtro Paso Alto RC (HPF) de 1er Orden', world: 'world-1', next: '2-1',
                narrative: 'Tomando la salida sobre la resistencia, $V_{out} = V_{in} \\frac{R}{R + Z_C} = V_{in} \\frac{j\\omega RC}{1 + j\\omega RC}$. Usando frecuencia normalizada $\\Omega = \\omega/\\omega_c$, $H(j\\Omega) = \\frac{j\\Omega}{1 + j\\Omega}$. Bloquea la corriente continua DC.',
                eq1: 'H(j\\omega) = \\frac{j\\omega RC}{1 + j\\omega RC}', eq2: 'H(j\\Omega) = \\frac{j\\Omega}{1 + j\\Omega}, \\quad \\Omega = \\frac{f}{f_c}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Respuesta en Frecuencia HPF:</b> Permite altas frecuencias.</div>',
                controls: `<div class="control-group"><label>Resistencia R: <span id="val-rh">1000</span> Ω</label><input type="range" id="input-rh" min="100" max="5000" step="100" value="1000" oninput="drawSimHPF()"></div><div class="control-group"><label>Capacitancia C: <span id="val-ch">0.1</span> µF</label><input type="range" id="input-ch" min="0.01" max="1" step="0.01" value="0.1" oninput="drawSimHPF()"></div>`,
                drawSchema: (ctx) => drawSchemaHPF(ctx), drawSim: () => drawSimHPF(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Variación de la respuesta HPF." },
                    { title: "Ejercicio 1: Pregunta Conceptual", type: "fill_blank", desc: "A frecuencia cero (DC), la impedancia del condensador es ________.", answer: "infinita" },
                    { title: "Ejercicio 2: Ajuste de R", type: "sim", desc: "Fija R = 1500 Ω.", check: () => parseFloat(document.getElementById('input-rh').value) === 1500 },
                    { title: "Ejercicio 3: Ajuste de C", type: "sim", desc: "Ajusta C = 0.05 µF.", check: () => Math.abs(parseFloat(document.getElementById('input-ch').value) - 0.05) < 0.005 },
                    { title: "👾 DESAFÍO BOSS: Frecuencia de Corte fc ≈ 318 Hz", type: "sim", desc: "Configura R = 5000 Ω y C = 0.1 µF.", check: () => parseFloat(document.getElementById('input-rh').value) === 5000 && Math.abs(parseFloat(document.getElementById('input-ch').value) - 0.1) < 0.01 }
                ]
            },
            '2-1': {
                title: 'Misión 2.1: Potencia RMS y Promedio', world: 'world-2', next: '2-2',
                narrative: 'El valor RMS ($V_{rms} = V_m / \\sqrt{2}$) mide la capacidad de una tensión alterna para realizar trabajo equivalente en corriente continua.',
                eq1: 'p(t) = v(t) \\cdot i(t)', eq2: 'P_{prom} = V_{rms} \\cdot I_{rms} \\cdot \\cos(\\theta)',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Potencia Promedio P:</b> Trabajo útil.</div>',
                controls: `<div class="control-group"><label>Voltaje Vrms: <span id="val-vrms">120</span> V</label><input type="range" id="input-vrms" min="60" max="240" value="120" oninput="drawSimPinst()"></div>`,
                drawSchema: (ctx) => drawSchemaPinst(ctx), drawSim: () => drawSimPinst(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Variación del nivel de potencia promedio." },
                    { title: "Ejercicio 1: Pregunta Conceptual", type: "fill_blank", desc: "La sigla RMS en español significa Raíz Cuadrada ________.", answer: "media" },
                    { title: "Ejercicio 2: Voltaje Residencial", type: "sim", desc: "Ajusta Vrms = 110 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 110 },
                    { title: "Ejercicio 3: Voltaje Industrial", type: "sim", desc: "Sube a Vrms = 220 V.", check: () => parseFloat(document.getElementById('input-vrms').value) === 220 },
                    { title: "👾 DESAFÍO BOSS: Potencia P_prom = 360W", type: "sim", desc: "Ajusta Vrms a 240 V para lograr P = 360W.", check: () => parseFloat(document.getElementById('input-vrms').value) === 240 }
                ]
            },
            '2-2': {
                title: 'Misión 2.2: Triángulo de Potencia Vectorial', world: 'world-2', next: '2-3',
                narrative: 'Unimos la Potencia Activa $P$ (Watts) y la Reactiva $Q$ (VAR) en la Potencia Aparente $\\mathbf{S} = P + jQ$ (VA).',
                eq1: '\\mathbf{S} = P + jQ', eq2: '|\\mathbf{S}| = \\sqrt{P^2 + Q^2}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Vector S:</b> Potencia aparente.</div>',
                controls: `<div class="control-group"><label>Reactiva QL: <span id="val-ql">250</span> VAR</label><input type="range" id="input-ql" min="50" max="450" value="250" oninput="drawSimTriang()"></div>`,
                drawSchema: (ctx) => drawSchemaTriang(ctx), drawSim: () => drawSimTriang(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Crecimiento del vector aparente S." },
                    { title: "Ejercicio 1: Pregunta Conceptual", type: "fill_blank", desc: "La potencia activa útil se mide en la unidad de ________.", answer: "watts" },
                    { title: "Ejercicio 2: Ajuste Inductivo", type: "sim", desc: "Fija QL = 100 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 100 },
                    { title: "Ejercicio 3: Incremento Reactivo", type: "sim", desc: "Sube QL = 300 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 300 },
                    { title: "👾 DESAFÍO BOSS: Triángulo Isósceles (P = Q)", type: "sim", desc: "Como P = 300 W, ajusta QL exactamente a 300 VAR.", check: () => parseFloat(document.getElementById('input-ql').value) === 300 }
                ]
            },
            '2-3': {
                title: 'Misión 2.3: Corrección del Factor de Potencia', world: 'world-2', next: '3-1',
                narrative: 'Añadimos condensadores en paralelo para inyectar $-j Q_C$ y elevar el Factor de Potencia $FP = \\cos\\theta \\ge 0.95$.',
                eq1: 'FP = \\frac{P}{S} = \\cos(\\theta)', eq2: 'Q_C = P (\\tan\\theta_1 - \\tan\\theta_2)',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Compensación:</b> Disminución de S.</div>',
                controls: `<div class="control-group"><label>Inyección QC: <span id="val-qc">150</span> VAR</label><input type="range" id="input-qc" min="0" max="350" value="150" oninput="drawSimFP()"></div>`,
                drawSchema: (ctx) => drawSchemaFP(ctx), drawSim: () => drawSimFP(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Inyección de capacitores." },
                    { title: "Ejercicio 1: Pregunta Conceptual", type: "fill_blank", desc: "Para corregir un factor de potencia inductivo se conectan capacitores en ________.", answer: "paralelo" },
                    { title: "Ejercicio 2: Inyección Inicial", type: "sim", desc: "Ajusta QC = 100 VAR.", check: () => parseFloat(document.getElementById('input-qc').value) === 100 },
                    { title: "Ejercicio 3: Inyección Media", type: "sim", desc: "Sube QC = 200 VAR.", check: () => parseFloat(document.getElementById('input-qc').value) === 200 },
                    { title: "👾 DESAFÍO BOSS: Lograr FP ≥ 0.95", type: "sim", desc: "Ajusta QC = 210 VAR para alcanzar FP = 0.96.", check: () => parseFloat(document.getElementById('input-qc').value) === 210 }
                ]
            },
            '3-1': {
                title: 'Misión 3.1: Inductancia Mutua y Acoplo', world: 'world-3', next: '3-2',
                narrative: 'Dos bobinas acopladas comparten flujo magnético induciendo una tensión mutua $M = k \\sqrt{L_1 L_2}$.',
                eq1: 'v_2(t) = M \\frac{di_1}{dt}', eq2: 'M = k \\sqrt{L_1 L_2}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Flujo k:</b> Coeficiente magnético.</div>',
                controls: `<div class="control-group"><label>Coeficiente k: <span id="val-k">0.7</span></label><input type="range" id="input-k" min="0.1" max="1" step="0.05" value="0.7" oninput="drawSimMutua()"></div>`,
                drawSchema: (ctx) => drawSchemaMutua(ctx), drawSim: () => drawSimMutua(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Variación del acoplamiento." },
                    { title: "Ejercicio 1: Pregunta Conceptual", type: "fill_blank", desc: "El coeficiente de acoplamiento ideal máximo vale ____.", answer: "1" },
                    { title: "Ejercicio 2: Acoplo Débil", type: "sim", desc: "Ajusta k = 0.3.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 0.3) < 0.01 },
                    { title: "Ejercicio 3: Acoplo Medio", type: "sim", desc: "Ajusta k = 0.6.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 0.6) < 0.01 },
                    { title: "👾 DESAFÍO BOSS: Acoplo Perfecto", type: "sim", desc: "Fija k = 1.0.", check: () => Math.abs(parseFloat(document.getElementById('input-k').value) - 1.0) < 0.01 }
                ]
            },
            '3-2': {
                title: 'Misión 3.2: Regla de Puntos en Bobinas', world: 'world-3', next: '3-3',
                narrative: 'Los puntos polares indican la concordancia de fase de la tensión inducida.',
                eq1: '\\mathbf{V}_1 = j\\omega L_1 \\mathbf{I}_1 \\pm j\\omega M \\mathbf{I}_2', eq2: '\\mathbf{V}_2 = j\\omega L_2 \\mathbf{I}_2 \\pm j\\omega M \\mathbf{I}_1',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Puntos Polares:</b> Referencias.</div>',
                controls: `<div class="control-group"><label>Dirección I2: <span id="val-dir">Mismo Punto</span></label><input type="range" id="input-dir" min="0" max="1" step="1" value="1" oninput="drawSimDots()"></div>`,
                drawSchema: (ctx) => drawSchemaDots(ctx), drawSim: () => drawSimDots(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Inversión de puntos." },
                    { title: "Ejercicio 1: Pregunta Conceptual", type: "fill_blank", desc: "Si la corriente entra por el punto, produce una tensión inducida ________ en el otro punto.", answer: "positiva" },
                    { title: "Ejercicio 2: Punto Opuesto", type: "sim", desc: "Coloca la corriente en Punto Opuesto (0).", check: () => parseInt(document.getElementById('input-dir').value) === 0 },
                    { title: "Ejercicio 3: Mismo Punto", type: "sim", desc: "Cambia a Mismo Punto (1).", check: () => parseInt(document.getElementById('input-dir').value) === 1 },
                    { title: "👾 DESAFÍO BOSS: Configuración Sumativa (+M)", type: "sim", desc: "Asegura la configuración de adición de flujo (1).", check: () => parseInt(document.getElementById('input-dir').value) === 1 }
                ]
            },
            '3-3': {
                title: 'Misión 3.3: Transformador Ideal', world: 'world-3', next: null,
                narrative: 'Escala voltajes según la razón de vueltas $a = N_1 / N_2$.',
                eq1: 'a = \\frac{N_1}{N_2} = \\frac{\\mathbf{V}_1}{\\mathbf{V}_2}', eq2: '\\mathbf{Z}_{in} = a^2 \\mathbf{Z}_L',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>V2 Salida:</b> Escalado.</div>',
                controls: `<div class="control-group"><label>Razón a: <span id="val-a">2.0</span></label><input type="range" id="input-a" min="0.5" max="4" step="0.5" value="2" oninput="drawSimTransfo()"></div>`,
                drawSchema: (ctx) => drawSchemaTransfo(ctx), drawSim: () => drawSimTransfo(),
                challenges: [
                    { title: "Ejemplo Auto-Animado 🎬", type: "sim", desc: "Transformación de voltaje." },
                    { title: "Ejercicio 1: Pregunta Conceptual", type: "fill_blank", desc: "En un transformador ideal la potencia de entrada es igual a la potencia de ________.", answer: "salida" },
                    { title: "Ejercicio 2: Reductor", type: "sim", desc: "Ajusta un transformador reductor a = 2.0.", check: () => parseFloat(document.getElementById('input-a').value) === 2.0 },
                    { title: "Ejercicio 3: Elevador", type: "sim", desc: "Ajusta elevador a = 0.5.", check: () => parseFloat(document.getElementById('input-a').value) === 0.5 },
                    { title: "👾 DESAFÍO BOSS: Obtener V2 = 30 V", type: "sim", desc: "Ajusta a = 4.0 para reducir 120V a 30V.", check: () => parseFloat(document.getElementById('input-a').value) === 4.0 }
                ]
            }
        };

        /* --- ESTADO DEL JUGADOR --- */
        const gameState = {
            xp: 0, level: 1,
            unlockedWorlds: ['world-1'],
            completedQuests: [],
            unlockedQuests: ['1-1']
        };

        let currentZoomWorld = null;
        let activeQuest = null;
        let activeSubstep = 0;
        let typewriterTimeout = null;
        let currentFullText = "";
        let electronAnimProgress = 0;
        let demoInterval = null;
        let formulaTypewriterTimeout = null;

        window.onload = () => {
            updateUI();
            updateCablesAndElectron();
            window.addEventListener('resize', updateCablesAndElectron);
            requestAnimationFrame(animateElectron);
            initFormulasSubmenu();
        };

        function updateUI() {
            document.getElementById('player-lvl').innerText = `Nivel ${gameState.level}`;
            const ranks = ["Novato AC", "Analista de Filtros", "Maestro de Impedancias", "Soberano Trifásico"];
            document.getElementById('player-rank').innerText = ranks[Math.min(gameState.level - 1, ranks.length - 1)];
            document.getElementById('xp-bar').style.width = `${Math.min((gameState.xp / (gameState.level * 200)) * 100, 100)}%`;

            ['world-1', 'world-2', 'world-3'].forEach((wId) => {
                const el = document.getElementById(wId);
                const isUnlocked = gameState.unlockedWorlds.includes(wId);
                if (isUnlocked) el.classList.remove('locked');
                else el.classList.add('locked');
            });

            Object.keys(questsDB).forEach(qKey => {
                const btn = document.getElementById(`sub-${qKey}`);
                if (!btn) return;

                if (gameState.completedQuests.includes(qKey)) {
                    btn.className = "level-pin-node completed";
                } else if (gameState.unlockedQuests.includes(qKey)) {
                    btn.className = "level-pin-node unlocked";
                } else {
                    btn.className = "level-pin-node locked";
                }
            });

            updateCablesAndElectron();
        }

        /* --- ZOOM EXTREMO E INMERSIÓN EN MUNDO --- */
        function zoomIntoWorld(worldId) {
            if (!gameState.unlockedWorlds.includes(worldId)) return;
            if (currentZoomWorld === worldId) return;

            currentZoomWorld = worldId;
            const container = document.getElementById('world-container');
            const targetEl = document.getElementById(worldId);

            // Ocultar HUD global
            document.getElementById('global-hud').classList.add('hidden-hud');
            document.getElementById('btn-exit-zoom').style.display = 'block';

            // Ocultar otros mundos
            ['world-1', 'world-2', 'world-3'].forEach(id => {
                if (id !== worldId) document.getElementById(id).style.opacity = '0.05';
            });
            document.getElementById('node-ground').style.opacity = '0.05';
            document.getElementById('node-source').style.opacity = '0.05';

            // Zoom Extremo calculando la posición de la isla
            const rect = targetEl.getBoundingClientRect();
            const containerRect = container.getBoundingClientRect();
            const offsetY = (containerRect.height / 2) - (targetEl.offsetTop + rect.height / 2);

            container.style.transform = `translate(-50%, calc(-50% + ${offsetY}px)) scale(1.65)`;

            setTimeout(updateCablesAndElectron, 400);
        }

        function exitWorldZoom() {
            currentZoomWorld = null;
            const container = document.getElementById('world-container');
            container.style.transform = 'translate(-50%, -50%) scale(1)';

            document.getElementById('global-hud').classList.remove('hidden-hud');
            document.getElementById('btn-exit-zoom').style.display = 'none';

            ['world-1', 'world-2', 'world-3'].forEach(id => {
                document.getElementById(id).style.opacity = '1';
            });
            document.getElementById('node-ground').style.opacity = '1';
            document.getElementById('node-source').style.opacity = '1';

            setTimeout(updateCablesAndElectron, 400);
        }

        /* --- TRAYECTORIA SVG Y ELECTRÓN --- */
        function updateCablesAndElectron() {
            const container = document.getElementById('world-container');
            const svg = document.getElementById('cable-svg');
            const rectW = container.getBoundingClientRect();

            svg.setAttribute('width', rectW.width);
            svg.setAttribute('height', rectW.height);

            let pathD = "";

            if (currentZoomWorld) {
                // TRAYECTORIA PIN-A-PIN EN PROTOBOARD DENTRO DE LA ISLA
                const activeWorldEl = document.getElementById(currentZoomWorld);
                const subCards = activeWorldEl.querySelectorAll('.level-pin-node');

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
                // TRAYECTORIA VERTICAL PRINCIPAL
                const nodes = ['node-ground', 'world-1', 'world-2', 'world-3', 'node-source'];
                for (let i = 0; i < nodes.length - 1; i++) {
                    const elA = document.getElementById(nodes[i]);
                    const elB = document.getElementById(nodes[i+1]);

                    const targetA = elA.querySelector('.world-header-badge') || elA;
                    const targetB = elB.querySelector('.world-header-badge') || elB;

                    const rA = targetA.getBoundingClientRect();
                    const rB = targetB.getBoundingClientRect();

                    const x1 = rA.left + rA.width / 2 - rectW.left;
                    const y1 = rA.top + rA.height / 2 - rectW.top;
                    const x2 = rB.left + rB.width / 2 - rectW.left;
                    const y2 = rB.top + rB.height / 2 - rectW.top;

                    const deltaY = y2 - y1;
                    const curveOffset = (i % 2 === 0) ? 140 : -140;

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

        /* --- CONTROL DE PASOS DE MISIONES --- */
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
                const fillCard = document.getElementById('fill-blank-container');
                const simBox = document.getElementById('sim-canvas-box');

                banner.className = "challenge-box" + (activeSubstep === 5 ? " boss" : "");
                document.getElementById('challenge-title').innerText = challenge.title;
                document.getElementById('challenge-desc').innerText = challenge.desc;

                if (challenge.type === "fill_blank") {
                    fillCard.style.display = 'flex';
                    simBox.style.display = 'none';
                    document.getElementById('fill-blank-text').innerText = challenge.desc;
                    const inp = document.getElementById('fill-blank-input');
                    inp.value = "";
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
                        document.getElementById('fill-blank-feedback').innerText = '❌ Palabra incorrecta. Inténtalo de nuevo.';
                        return;
                    } else {
                        document.getElementById('fill-blank-feedback').style.color = 'var(--green)';
                        document.getElementById('fill-blank-feedback').innerText = '✔ ¡Correcto!';
                    }
                } else if (challenge.check && !challenge.check()) {
                    alert("⚠️ ¡El simulador no está en la posición requerida! Revisa las instrucciones.");
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

            setTimeout(() => { exitWorldZoom(); }, 300);
        }

        /* --- SUBMENÚ DE FÓRMULAS INTERACTIVO Y NARRATIVO --- */
        function initFormulasSubmenu() {
            const listContainer = document.getElementById('formulas-list-sidebar');
            listContainer.innerHTML = '';

            formulasDB.forEach((f, idx) => {
                const btn = document.createElement('div');
                btn.className = `formula-item-btn ${idx === 0 ? 'active' : ''}`;
                btn.innerText = f.title;
                btn.onclick = () => selectFormula(f, btn);
                listContainer.appendChild(btn);
            });

            if (formulasDB.length > 0) selectFormula(formulasDB[0], listContainer.children[0]);
        }

        function selectFormula(f, btnEl) {
            document.querySelectorAll('.formula-item-btn').forEach(b => b.classList.remove('active'));
            if (btnEl) btnEl.classList.add('active');

            document.getElementById('formula-detail-title').innerText = f.title;
            katex.render(f.eq, document.getElementById('formula-detail-eq'), { displayMode: true, throwOnError: false });

            startFormulaTypewriter(f.derivation);
        }

        function startFormulaTypewriter(htmlContent) {
            if (formulaTypewriterTimeout) clearTimeout(formulaTypewriterTimeout);
            const el = document.getElementById('formula-narrative-text');
            el.innerHTML = "";
            let i = 0;

            function type() {
                if (i < htmlContent.length) {
                    if (htmlContent.charAt(i) === "<") {
                        let closeIdx = htmlContent.indexOf(">", i);
                        if (closeIdx !== -1) {
                            el.innerHTML += htmlContent.substring(i, closeIdx + 1);
                            i = closeIdx + 1;
                        } else { el.innerHTML += htmlContent.charAt(i); i++; }
                    } else {
                        el.innerHTML += htmlContent.charAt(i); i++;
                    }
                    
                    if (window.renderMathInElement) {
                        renderMathInElement(el, { delimiters: [{left: '$', right: '$', display: false}], throwOnError: false });
                    }
                    
                    formulaTypewriterTimeout = setTimeout(type, 12);
                }
            }
            type();
        }

        function skipFormulaTypewriter() {
            if (formulaTypewriterTimeout) clearTimeout(formulaTypewriterTimeout);
            const activeObj = formulasDB.find(f => f.title === document.getElementById('formula-detail-title').innerText);
            if (activeObj) {
                const el = document.getElementById('formula-narrative-text');
                el.innerHTML = activeObj.derivation;
                if (window.renderMathInElement) {
                    renderMathInElement(el, { delimiters: [{left: '$', right: '$', display: false}], throwOnError: false });
                }
            }
        }

        function openFormulasModal() { document.getElementById('formulas-modal').classList.add('active'); }
        function closeFormulasModal() { document.getElementById('formulas-modal').classList.remove('active'); }

        /* --- DIBUJO DE ESQUEMAS EN COLUMNA IZQUIERDA DE 1/4 --- */
        function drawPillLabel(ctx, text, x, y, color) {
            ctx.font = '11px Orbitron';
            const w = ctx.measureText(text).width + 10;
            ctx.fillStyle = 'rgba(7, 13, 26, 0.9)';
            ctx.fillRect(x - w/2, y - 9, w, 18);
            ctx.strokeStyle = color; ctx.lineWidth = 1; ctx.strokeRect(x - w/2, y - 9, w, 18);
            ctx.fillStyle = color; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
            ctx.fillText(text, x, y);
        }

        function drawResistorVertical(ctx, x, y1, y2) {
            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 2.5;
            ctx.beginPath(); ctx.moveTo(x, y1);
            const dy = (y2 - y1) / 6, dx = 8;
            ctx.lineTo(x - dx, y1 + dy); ctx.lineTo(x + dx, y1 + dy*2);
            ctx.lineTo(x - dx, y1 + dy*3); ctx.lineTo(x + dx, y1 + dy*4);
            ctx.lineTo(x - dx, y1 + dy*5); ctx.lineTo(x, y2); ctx.stroke();
        }

        function drawCapacitorVertical(ctx, x, y) {
            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(x - 12, y - 6); ctx.lineTo(x + 12, y - 6); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(x - 12, y + 6); ctx.lineTo(x + 12, y + 6); ctx.stroke();
        }

        function drawSchemaGen(ctx) {
            ctx.clearRect(0,0,240,200);
            ctx.fillStyle='#131d30'; ctx.fillRect(20,40,45,120); ctx.fillRect(175,40,45,120);
            drawPillLabel(ctx, 'N', 42, 100, '#ff0055');
            drawPillLabel(ctx, 'S', 197, 100, '#00f3ff');
            ctx.strokeStyle='#ffb700'; ctx.lineWidth=3; ctx.beginPath(); ctx.arc(120,100,28,0,Math.PI*2); ctx.stroke();
            drawPillLabel(ctx, 'Generador v(t)', 120, 175, '#00ff66');
        }

        function drawSchemaLPF(ctx) {
            ctx.clearRect(0,0,240,200);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.moveTo(30,50); ctx.lineTo(30,80); ctx.stroke();
            drawResistorVertical(ctx, 30, 80, 140);
            ctx.beginPath(); ctx.moveTo(30,140); ctx.lineTo(30,170); ctx.lineTo(170,170); ctx.stroke();
            drawCapacitorVertical(ctx, 170, 110);
            ctx.beginPath(); ctx.moveTo(170,50); ctx.lineTo(170,104); ctx.moveTo(170,116); ctx.lineTo(170,170); ctx.stroke();
            drawPillLabel(ctx, 'Vin', 30, 30, '#ff0055');
            drawPillLabel(ctx, 'R', 60, 110, '#ffb700');
            drawPillLabel(ctx, 'C', 200, 110, '#00f3ff');
            drawPillLabel(ctx, 'Vout', 170, 30, '#00ff66');
        }

        function drawSchemaHPF(ctx) {
            ctx.clearRect(0,0,240,200);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.moveTo(30,50); ctx.lineTo(30,104); ctx.stroke();
            drawCapacitorVertical(ctx, 30, 110);
            ctx.beginPath(); ctx.moveTo(30,116); ctx.lineTo(30,170); ctx.lineTo(170,170); ctx.stroke();
            drawResistorVertical(ctx, 170, 80, 140);
            ctx.beginPath(); ctx.moveTo(170,50); ctx.lineTo(170,80); ctx.moveTo(170,140); ctx.lineTo(170,170); ctx.stroke();
            drawPillLabel(ctx, 'Vin', 30, 30, '#ff0055');
            drawPillLabel(ctx, 'C', 60, 110, '#00f3ff');
            drawPillLabel(ctx, 'R', 200, 110, '#ffb700');
            drawPillLabel(ctx, 'Vout', 170, 30, '#00ff66');
        }

        function drawSchemaPinst(ctx) {
            ctx.clearRect(0,0,240,200);
            ctx.fillStyle='#131d30'; ctx.fillRect(30,40,180,50);
            drawPillLabel(ctx, 'Fuente AC RMS', 120, 65, '#00f3ff');
            ctx.strokeStyle='#00ff66'; ctx.lineWidth=2; ctx.strokeRect(30,120,180,50);
            drawPillLabel(ctx, 'Carga ZL', 120, 145, '#00ff66');
        }

        function drawSchemaTriang(ctx) {
            ctx.clearRect(0,0,240,200);
            ctx.strokeStyle='#ff0055'; ctx.lineWidth=2; ctx.strokeRect(30,30,180,55);
            drawPillLabel(ctx, 'Motor Q (VAR)', 120, 57, '#ff0055');
            ctx.strokeStyle='#00f3ff'; ctx.strokeRect(30,115,180,55);
            drawPillLabel(ctx, 'Resistencia P (W)', 120, 142, '#00f3ff');
        }

        function drawSchemaFP(ctx) {
            ctx.clearRect(0,0,240,200);
            ctx.strokeStyle='#00ff66'; ctx.lineWidth=2; ctx.strokeRect(30,30,180,55);
            drawPillLabel(ctx, 'Fábrica Inductiva', 120, 57, '#00ff66');
            ctx.strokeStyle='#ff0055'; ctx.strokeRect(30,115,180,55);
            drawPillLabel(ctx, 'Capacitores -QC', 120, 142, '#ff0055');
        }

        function drawSchemaMutua(ctx) {
            ctx.clearRect(0,0,240,200);
            ctx.strokeStyle='#ffb700'; ctx.lineWidth=3; ctx.beginPath(); ctx.arc(60,100,30,0,Math.PI*2); ctx.stroke();
            ctx.strokeStyle='#00f3ff'; ctx.beginPath(); ctx.arc(180,100,30,0,Math.PI*2); ctx.stroke();
            drawPillLabel(ctx, 'Inductancia Mutua M', 120, 100, '#00f3ff');
        }

        function drawSchemaDots(ctx) {
            ctx.clearRect(0,0,240,200);
            ctx.strokeStyle='#00f3ff'; ctx.lineWidth=2;
            ctx.beginPath(); ctx.arc(60,100,25,0,Math.PI*2); ctx.stroke();
            ctx.beginPath(); ctx.arc(180,100,25,0,Math.PI*2); ctx.stroke();
            ctx.fillStyle='#ff0055'; ctx.beginPath(); ctx.arc(60,65,5,0,Math.PI*2); ctx.fill();
            ctx.beginPath(); ctx.arc(180,65,5,0,Math.PI*2); ctx.fill();
            drawPillLabel(ctx, 'Puntos Polares', 120, 150, '#ff0055');
        }

        function drawSchemaTransfo(ctx) {
            ctx.clearRect(0,0,240,200);
            ctx.fillStyle='#131d30'; ctx.fillRect(70,30,100,140); ctx.clearRect(95,55,50,90);
            drawPillLabel(ctx, 'N1', 45, 100, '#ffb700');
            drawPillLabel(ctx, 'N2', 195, 100, '#00f3ff');
        }

        /* --- SIMULADORES CANVA --- */
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
            for (let x = 240; x <= 680; x++) {
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
            for(let x=40; x<660; x++) {
                const f = Math.pow(10, (x - 40) / 150);
                const H = 1 / Math.sqrt(1 + Math.pow(f / fc, 2));
                const y = 180 - H * 140;
                if(x===40) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();

            const xFc = 40 + Math.log10(fc) * 150;
            if (xFc >= 40 && xFc <= 660) {
                ctx.strokeStyle = '#ffb700'; ctx.setLineDash([4, 4]);
                ctx.beginPath(); ctx.moveTo(xFc, 10); ctx.lineTo(xFc, 190); ctx.stroke(); ctx.setLineDash([]);
                ctx.fillStyle = '#ffb700'; ctx.font = '12px Orbitron';
                ctx.fillText(`fc = ${Math.round(fc)} Hz (-3dB)`, Math.min(xFc + 8, 520), 30);
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
            for(let x=40; x<660; x++) {
                const f = Math.pow(10, (x - 40) / 150);
                const H = (f / fc) / Math.sqrt(1 + Math.pow(f / fc, 2));
                const y = 180 - H * 140;
                if(x===40) ctx.moveTo(x, y); else ctx.lineTo(x, y);
            }
            ctx.stroke();

            const xFc = 40 + Math.log10(fc) * 150;
            if (xFc >= 40 && xFc <= 660) {
                ctx.strokeStyle = '#ffb700'; ctx.setLineDash([4, 4]);
                ctx.beginPath(); ctx.moveTo(xFc, 10); ctx.lineTo(xFc, 190); ctx.stroke(); ctx.setLineDash([]);
                ctx.fillStyle = '#ffb700'; ctx.font = '12px Orbitron';
                ctx.fillText(`fc = ${Math.round(fc)} Hz (-3dB)`, Math.min(xFc + 8, 520), 30);
            }
        }

        function drawSimPinst() {
            const Vrms = parseFloat(document.getElementById('input-vrms').value);
            document.getElementById('val-vrms').innerText = Vrms;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const Pprom = (Vrms * 1.5).toFixed(0);

            ctx.strokeStyle = '#ff0055'; ctx.lineWidth = 2; ctx.beginPath();
            for(let x=50; x<650; x++) {
                const t = (x-50)*0.03, p = Pprom * (1 + Math.cos(t));
                const y = 180 - p * 0.35;
                if(x===50) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 2; ctx.setLineDash([5,5]);
            const yP = 180 - Pprom * 0.35;
            ctx.beginPath(); ctx.moveTo(50, yP); ctx.lineTo(650, yP); ctx.stroke(); ctx.setLineDash([]);

            ctx.fillStyle = '#ffb700'; ctx.font = '12px Orbitron';
            ctx.fillText(`P_prom = ${Pprom} W`, 520, yP - 8);
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
            ctx.strokeStyle = '#00f3ff'; ctx.beginPath(); ctx.arc(500, 105, 40, 0, Math.PI*2); ctx.stroke();

            ctx.strokeStyle = `rgba(0, 255, 102, ${k})`; ctx.lineWidth = k * 5;
            for(let r = 20; r <= 70; r += 20) {
                ctx.beginPath(); ctx.ellipse(340, 105, 160, r, 0, 0, Math.PI*2); ctx.stroke();
            }
        }

        function drawSimDots() {
            const dir = parseInt(document.getElementById('input-dir').value);
            document.getElementById('val-dir').innerText = dir === 1 ? 'Mismo Punto' : 'Punto Opuesto';

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 3;
            ctx.strokeRect(200, 50, 70, 100); ctx.strokeRect(430, 50, 70, 100);

            ctx.fillStyle = '#ff0055';
            ctx.beginPath(); ctx.arc(235, 35, 6, 0, Math.PI*2); ctx.fill();
            ctx.beginPath(); ctx.arc(465, dir === 1 ? 35 : 165, 6, 0, Math.PI*2); ctx.fill();
        }

        function drawSimTransfo() {
            const a = parseFloat(document.getElementById('input-a').value);
            document.getElementById('val-a').innerText = a;

            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            const V1 = 120, V2 = (V1 / a).toFixed(1);

            ctx.fillStyle = '#131d30'; ctx.fillRect(240, 35, 220, 110); ctx.clearRect(280, 60, 140, 60);

            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 2; ctx.beginPath();
            for(let x=30; x<220; x++) {
                const y = 90 - 25 * Math.sin((x-30)*0.05);
                if(x===30) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();

            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 2; ctx.beginPath();
            for(let x=480; x<670; x++) {
                const y = 90 - (25/a) * Math.sin((x-480)*0.05);
                if(x===480) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();

            ctx.fillStyle = '#fff'; ctx.font = '12px Orbitron';
            ctx.fillText(`V1 = ${V1} V`, 80, 145); ctx.fillText(`V2 = ${V2} V`, 530, 145);
        }
    </script>
</body>
</html>