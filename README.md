<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Análisis de Circuitos AC - Ruta RPG</title>
    <!-- KaTeX CDN para matemáticas impecables -->
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
                radial-gradient(circle at 50% 30%, rgba(0, 243, 255, 0.05) 0%, transparent 70%),
                linear-gradient(to right, rgba(255, 255, 255, 0.02) 1px, transparent 1px),
                linear-gradient(to bottom, rgba(255, 255, 255, 0.02) 1px, transparent 1px);
            background-size: 100% 100%, 35px 35px, 35px 35px;
            color: var(--text-main);
            font-family: 'Rajdhani', sans-serif;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
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

        /* --- CAMINO PRINCIPAL DE CAPÍTULOS --- */
        #path-container {
            max-width: 900px;
            width: 100%;
            margin: 40px auto;
            padding: 0 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 0;
            position: relative;
        }

        .chapter-node {
            width: 100%;
            background: var(--panel-bg);
            border: 2px solid var(--panel-border);
            border-radius: 12px;
            overflow: hidden;
            transition: all 0.3s ease;
            box-shadow: 0 5px 25px rgba(0,0,0,0.6);
            z-index: 2;
        }

        .chapter-node.unlocked { border-color: var(--cyan); box-shadow: 0 0 20px rgba(0,243,255,0.2); }
        .chapter-node.completed { border-color: var(--green); box-shadow: 0 0 20px rgba(0,255,102,0.2); }
        .chapter-node.locked { opacity: 0.5; filter: grayscale(1); border-color: var(--text-muted); }

        .chapter-header {
            padding: 18px 25px;
            background: rgba(0, 243, 255, 0.05);
            display: flex;
            justify-content: space-between;
            align-items: center;
            cursor: pointer;
        }

        .chapter-title {
            font-family: 'Orbitron', sans-serif;
            font-size: 1.15rem;
            color: #fff;
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .chapter-status-tag {
            font-family: 'Orbitron', sans-serif;
            font-size: 0.75rem;
            padding: 4px 10px;
            border-radius: 12px;
            border: 1px solid var(--cyan);
            color: var(--cyan);
        }
        .completed .chapter-status-tag { border-color: var(--green); color: var(--green); }
        .locked .chapter-status-tag { border-color: var(--text-muted); color: var(--text-muted); }

        /* CAMINO CONECTOR ENTRE CAPÍTULOS */
        .path-connector {
            width: 6px;
            height: 40px;
            background: rgba(255, 255, 255, 0.1);
            z-index: 1;
            transition: background 0.3s;
        }
        .path-connector.active { background: var(--cyan); box-shadow: 0 0 10px var(--cyan); }

        /* SUBMISIONES */
        .subtopics-grid {
            display: none;
            padding: 20px;
            background: rgba(0, 0, 0, 0.3);
            border-top: 1px solid rgba(255,255,255,0.05);
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 12px;
        }

        .chapter-node.open .subtopics-grid { display: grid; }

        .subtopic-btn {
            background: rgba(13, 20, 36, 0.8);
            border: 1px solid rgba(255, 255, 255, 0.15);
            border-radius: 8px;
            padding: 12px 15px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            cursor: pointer;
            transition: all 0.2s;
        }
        .subtopic-btn:hover:not(.locked) { transform: translateY(-2px); border-color: var(--cyan); box-shadow: 0 0 12px rgba(0, 243, 255, 0.2); }
        .subtopic-btn.completed { border-color: var(--green); background: rgba(0, 255, 102, 0.05); }
        .subtopic-btn.locked { opacity: 0.4; cursor: not-allowed; }

        /* --- MODALES --- */
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
            width: 90%; max-width: 850px;
            max-height: 90vh;
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

        .step-view { display: none; flex-direction: column; gap: 15px; }
        .step-view.active { display: flex; }

        .theory-box {
            background: rgba(0, 0, 0, 0.4);
            border-left: 3px solid var(--cyan);
            padding: 15px; border-radius: 0 6px 6px 0;
            font-size: 1rem; line-height: 1.5;
        }

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
            display: flex; align-items: center; gap: 8px; margin-top: 10px;
        }
        .nav-btn:hover { background: var(--cyan); color: #000; box-shadow: 0 0 15px var(--cyan); }

        /* --- SUBMENÚ CODEX DE PERSONAJES (VARIABLES) --- */
        .codex-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(110px, 1fr));
            gap: 15px;
            padding: 10px 0;
        }

        .char-card {
            background: rgba(255,255,255,0.03);
            border: 1px solid rgba(255,255,255,0.1);
            border-radius: 8px;
            padding: 15px 10px;
            display: flex; flex-direction: column; align-items: center;
            cursor: pointer; transition: all 0.2s;
            position: relative;
        }
        .char-card.unlocked { border-color: var(--purple); background: rgba(157,0,255,0.08); }
        .char-card.unlocked:hover { transform: scale(1.05); box-shadow: 0 0 15px var(--purple); }
        .char-card.locked { opacity: 0.4; filter: grayscale(1); cursor: not-allowed; }

        .char-symbol {
            font-family: 'Orbitron', sans-serif;
            font-size: 1.8rem;
            font-weight: 900;
            color: var(--purple);
            margin-bottom: 5px;
        }
        .unlocked .char-symbol { color: var(--cyan); }
        .char-name { font-size: 0.75rem; text-align: center; color: var(--text-muted); }

        .char-detail-box {
            background: rgba(157, 0, 255, 0.1);
            border: 1px solid var(--purple);
            border-radius: 8px;
            padding: 15px;
            display: none;
            flex-direction: column;
            gap: 8px;
            margin-top: 15px;
        }
        .char-detail-box.active { display: flex; }
        .char-detail-title { font-family: 'Orbitron', sans-serif; color: var(--yellow); font-size: 1rem; }

        /* --- POPUP DE NUEVO PERSONAJE DESBLOQUEADO --- */
        .reveal-card {
            background: linear-gradient(135deg, #0d1424, #1a0933);
            border: 2px solid var(--purple);
            box-shadow: 0 0 50px var(--purple);
            border-radius: 12px;
            padding: 30px;
            text-align: center;
            display: flex; flex-direction: column; align-items: center; gap: 15px;
            max-width: 400px;
            animation: popIn 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        @keyframes popIn { 0% { transform: scale(0.5); opacity: 0; } 100% { transform: scale(1); opacity: 1; } }

        .reveal-symbol {
            font-family: 'Orbitron', sans-serif;
            font-size: 3.5rem;
            font-weight: 900;
            color: var(--cyan);
            text-shadow: 0 0 20px var(--cyan);
        }
    </style>
</head>
<body>

    <!-- HUD SUPERIOR -->
    <header>
        <div class="hud-title">⚡ CIRCUITY RPG</div>
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

    <!-- CAMINO PRINCIPAL DE CAPÍTULOS -->
    <div id="path-container">
        
        <!-- CAPÍTULO 1 -->
        <div class="chapter-node unlocked" id="chap-1">
            <div class="chapter-header" onclick="toggleChapter('chap-1')">
                <div class="chapter-title"><span>📘 CAP. 1:</span> Análisis Senoidal en Estado Estable</div>
                <span class="chapter-status-tag" id="tag-chap-1">EN PROGRESO</span>
            </div>
            <div class="subtopics-grid">
                <div class="subtopic-btn unlocked" id="sub-1-1" onclick="openQuest('1-1')">
                    <span>1.1 Onda Senoidal y Fasores</span> <span>➔</span>
                </div>
                <div class="subtopic-btn locked" id="sub-1-2" onclick="openQuest('1-2')">
                    <span>1.2 Impedancia y Resonancia RLC</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-1-3" onclick="openQuest('1-3')">
                    <span>1.3 Leyes de Kirchhoff en AC</span> <span>🔒</span>
                </div>
            </div>
        </div>

        <div class="path-connector" id="conn-1-2"></div>

        <!-- CAPÍTULO 2 -->
        <div class="chapter-node locked" id="chap-2">
            <div class="chapter-header" onclick="toggleChapter('chap-2')">
                <div class="chapter-title"><span>💡 CAP. 2:</span> Potencia en Estado Estacionario</div>
                <span class="chapter-status-tag" id="tag-chap-2">BLOQUEADO</span>
            </div>
            <div class="subtopics-grid">
                <div class="subtopic-btn locked" id="sub-2-1" onclick="openQuest('2-1')">
                    <span>2.1 Potencia Instantánea y RMS</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-2-2" onclick="openQuest('2-2')">
                    <span>2.2 Triángulo de Potencia Compleja</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-2-3" onclick="openQuest('2-3')">
                    <span>2.3 Corrección del Factor de Potencia</span> <span>🔒</span>
                </div>
            </div>
        </div>

        <div class="path-connector" id="conn-2-3"></div>

        <!-- CAPÍTULO 3 -->
        <div class="chapter-node locked" id="chap-3">
            <div class="chapter-header" onclick="toggleChapter('chap-3')">
                <div class="chapter-title"><span>🧲 CAP. 3:</span> Circuitos Acoplados Magnéticamente</div>
                <span class="chapter-status-tag" id="tag-chap-3">BLOQUEADO</span>
            </div>
            <div class="subtopics-grid">
                <div class="subtopic-btn locked" id="sub-3-1" onclick="openQuest('3-1')">
                    <span>3.1 Inductancia Mutua y Acoplo</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-3-2" onclick="openQuest('3-2')">
                    <span>3.2 Regla de Puntos en Bobinas</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-3-3" onclick="openQuest('3-3')">
                    <span>3.3 El Transformador Ideal</span> <span>🔒</span>
                </div>
            </div>
        </div>

        <div class="path-connector" id="conn-3-4"></div>

        <!-- CAPÍTULO 4 -->
        <div class="chapter-node locked" id="chap-4">
            <div class="chapter-header" onclick="toggleChapter('chap-4')">
                <div class="chapter-title"><span>🏢 CAP. 4:</span> Sistemas Polifásicos</div>
                <span class="chapter-status-tag" id="tag-chap-4">BLOQUEADO</span>
            </div>
            <div class="subtopics-grid">
                <div class="subtopic-btn locked" id="sub-4-1" onclick="openQuest('4-1')">
                    <span>4.1 Generador Trifásico y Secuencias</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-4-2" onclick="openQuest('4-2')">
                    <span>4.2 Conexión Estrella Y / Delta Δ</span> <span>🔒</span>
                </div>
                <div class="subtopic-btn locked" id="sub-4-3" onclick="openQuest('4-3')">
                    <span>4.3 Cargas Trifásicas Desequilibradas</span> <span>🔒</span>
                </div>
            </div>
        </div>

    </div>

    <!-- MODAL DE MISIÓN (TEORÍA -> SIMULADOR) -->
    <div class="modal-overlay" id="quest-modal">
        <div class="modal-card">
            <div class="modal-header">
                <h3 id="modal-title">Título de Misión</h3>
                <button class="close-btn" onclick="closeQuest()">&times;</button>
            </div>
            <div class="modal-body">
                
                <!-- PASO TEORÍA -->
                <div class="step-view active" id="step-theory">
                    <div class="theory-box" id="theory-text"></div>
                    
                    <div class="math-card">
                        <div class="math-title">Fórmula Básica</div>
                        <div class="math-eq" id="eq-1"></div>
                    </div>

                    <div class="math-card">
                        <div class="math-title">Ecuación Final de Análisis</div>
                        <div class="math-eq" id="eq-2"></div>
                    </div>

                    <button class="nav-btn" onclick="switchStep('sim')">
                        Continuar al Simulador ➔
                    </button>
                </div>

                <!-- PASO SIMULADOR -->
                <div class="step-view" id="step-sim">
                    <div class="sim-box">
                        <canvas id="sim-canvas" width="650" height="220"></canvas>
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

    <!-- MODAL CÓDEX DE PERSONAJES (VARIABLES) -->
    <div class="modal-overlay" id="codex-modal">
        <div class="modal-card" style="max-width: 650px;">
            <div class="modal-header">
                <h3>👾 CÓDEX DE PERSONAJES (VARIABLES AC)</h3>
                <button class="close-btn" onclick="closeCodex()">&times;</button>
            </div>
            <div class="modal-body">
                <p style="font-size: 0.9rem; color: var(--text-muted);">
                    Cada variable eléctrica posee habilidades únicas. Haz clic en un personaje desbloqueado para examinar su función:
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
        /* --- BASE DE DATOS DE PERSONAJES (VARIABLES ELÉCTRICAS) --- */
        const charactersDB = {
            V: { symbol: 'V', name: 'Voltaje (Tensión)', desc: '<b>Clase:</b> Impulsor.<br><b>Función:</b> Fuerza electromotriz que impulsa el flujo de electrones a través del circuito.', introKey: '1-1' },
            I: { symbol: 'I', name: 'Corriente (Intensidad)', desc: '<b>Clase:</b> Torrente.<br><b>Función:</b> Flujo de carga eléctrica que oscila continuamente en el tiempo.', introKey: '1-1' },
            w: { symbol: 'ω', name: 'Frecuencia Angular', desc: '<b>Clase:</b> Marcapasos.<br><b>Función:</b> Determina la velocidad de rotación fasorial en rad/s (ω = 2πf).', introKey: '1-1' },
            Z: { symbol: 'Z', name: 'Impedancia Compleja', desc: '<b>Clase:</b> Guardián.<br><b>Función:</b> Oposición total al paso de corriente en AC (Resistencia + Reactancia).', introKey: '1-2' },
            K: { symbol: 'K', name: 'Leyes de Kirchhoff', desc: '<b>Clase:</b> Malla/Nodo.<br><b>Función:</b> Dicta la conservación de carga y energía mediante matrices phasoriales.', introKey: '1-3' },
            P: { symbol: 'P', name: 'Potencia Activa', desc: '<b>Clase:</b> Convertidor Útil.<br><b>Función:</b> Transformación real de energía en trabajo mecánico o calor (Watts).', introKey: '2-1' },
            S: { symbol: 'S', name: 'Potencia Aparente', desc: '<b>Clase:</b> Envolvente.<br><b>Función:</b> Magnitud total de la potencia suministrada por la fuente (VA).', introKey: '2-2' },
            FP: { symbol: 'FP', name: 'Factor de Potencia', desc: '<b>Clase:</b> Centinela de Eficiencia.<br><b>Función:</b> Relación cos(θ) entre la potencia útil P y la aparente S.', introKey: '2-3' },
            M: { symbol: 'M', name: 'Inductancia Mutua', desc: '<b>Clase:</b> Enlace Magnético.<br><b>Función:</b> Transfiere energía entre bobinas separadas mediante flujo magnético.', introKey: '3-1' },
            VL: { symbol: 'VL', name: 'Tensión Trifásica', desc: '<b>Clase:</b> Trinidad.<br><b>Función:</b> Tensión entre dos líneas vivas en un sistema polifásico (√3 · VP).', introKey: '4-1' }
        };

        /* --- BASE DE DATOS DE SUBMISIONES --- */
        const questsDB = {
            '1-1': {
                title: 'Misión 1.1: Onda Senoidal y Fasores',
                chap: 'chap-1', next: '1-2', introChar: 'V',
                theory: 'Las señales AC cambian continuamente. Convertimos la onda temporal a un <b>Fasor</b> (vector complejo) para analizar amplitud y desfase fácilmente.',
                eq1: 'v(t) = V_m \\cdot \\cos(\\omega t + \\phi)',
                eq2: '\\mathbf{V} = \\frac{V_m}{\\sqrt{2}} \\angle \\phi',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Fasor V:</b> Vector de magnitud $V_m$ y ángulo $\\phi$.</div><div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Onda AC:</b> Resultado de la rotación en el tiempo.</div>',
                controls: `<div class="control-group"><label>Vm (V): <span id="val-vm">100</span></label><input type="range" id="input-vm" min="30" max="150" value="100" oninput="drawSenoidal()"></div>`,
                draw: () => drawSenoidal()
            },
            '1-2': {
                title: 'Misión 1.2: Impedancia y Resonancia RLC',
                chap: 'chap-1', next: '1-3', introChar: 'Z',
                theory: 'La <b>Impedancia ($Z$)</b> combina Resistencia y Reactancia. En la frecuencia de <b>Resonancia ($f_0$)</b>, las reactancias $X_L$ y $X_C$ se anulan.',
                eq1: '\\mathbf{Z} = R + j(\\omega L - \\frac{1}{\\omega C})',
                eq2: 'f_0 = \\frac{1}{2\\pi \\sqrt{LC}}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Curva |Z|:</b> Impedancia mínima en resonancia.</div>',
                controls: `<div class="control-group"><label>L (mH): <span id="val-l">10</span></label><input type="range" id="input-l" min="1" max="50" value="10" oninput="drawRLC()"></div>`,
                draw: () => drawRLC()
            },
            '1-3': {
                title: 'Misión 1.3: Leyes de Kirchhoff en AC',
                chap: 'chap-1', next: '2-1', introChar: 'K',
                theory: 'Aplicamos la Ley de Tensiones (Mallas) y Corrientes (Nodos) sustituyendo resistencias simples por impedancias complejas $\\mathbf{Z}$.',
                eq1: '\\sum \\mathbf{V}_k = 0, \\quad \\sum \\mathbf{I}_k = 0',
                eq2: '[\\mathbf{Y}] \\cdot [\\mathbf{V}] = [\\mathbf{I}]',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Nodos/Mallas:</b> Matrices de admitancia complejas.</div>',
                controls: `<div class="control-group"><label>Impedancia Z (Ω): <span id="val-z">20</span></label><input type="range" id="input-z" min="5" max="50" value="20" oninput="drawKCL()"></div>`,
                draw: () => drawKCL()
            },
            '2-1': {
                title: 'Misión 2.1: Potencia Instantánea y RMS',
                chap: 'chap-2', next: '2-2', introChar: 'P',
                theory: 'La potencia varia el doble de rápido que la tensión. El valor <b>RMS</b> representa el equivalente en calentamiento DC.',
                eq1: 'p(t) = v(t) \\cdot i(t)',
                eq2: 'P_{prom} = V_{rms} \\cdot I_{rms} \\cdot \\cos(\\theta_v - \\theta_i)',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Línea P:</b> Potencia Promedio útil.</div>',
                controls: `<div class="control-group"><label>Vrms: <span id="val-vrms">120</span></label><input type="range" id="input-vrms" min="50" max="220" value="120" oninput="drawPinst()"></div>`,
                draw: () => drawPinst()
            },
            '2-2': {
                title: 'Misión 2.2: Triángulo de Potencia Compleja',
                chap: 'chap-2', next: '2-3', introChar: 'S',
                theory: 'La potencia vectorial $\\mathbf{S}$ une la Potencia Real $P$ (Watts) y la Reactiva $Q$ (VAR) en un triángulo rectángulo.',
                eq1: '\\mathbf{S} = P + jQ',
                eq2: '|S| = \\sqrt{P^2 + Q^2}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Vector S:</b> Potencia aparente total suministrada.</div>',
                controls: `<div class="control-group"><label>Q (VAR): <span id="val-q">200</span></label><input type="range" id="input-q" min="50" max="400" value="200" oninput="drawTriang()"></div>`,
                draw: () => drawTriang()
            },
            '2-3': {
                title: 'Misión 2.3: Corrección del Factor de Potencia',
                chap: 'chap-2', next: '3-1', introChar: 'FP',
                theory: 'Un $FP < 0.95$ recarga la red. Añadimos un banco de capacitores $Q_C$ para reducir la potencia reactiva requerida.',
                eq1: 'FP = \\frac{P}{S} = \\cos(\\theta)',
                eq2: 'Q_C = P(\\tan\\theta_1 - \\tan\\theta_2)',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Q Reducida:</b> Corrección mediante capacitores.</div>',
                controls: `<div class="control-group"><label>Compensación QC: <span id="val-qc">150</span></label><input type="range" id="input-qc" min="0" max="300" value="150" oninput="drawFP()"></div>`,
                draw: () => drawFP()
            },
            '3-1': {
                title: 'Misión 3.1: Inductancia Mutua y Acoplo',
                chap: 'chap-3', next: '3-2', introChar: 'M',
                theory: 'La corriente en una bobina induce un voltaje en otra cercana mediante la inductancia mutua $M = k\\sqrt{L_1 L_2}$.',
                eq1: 'v_2(t) = M \\frac{di_1}{dt}',
                eq2: 'M = k \\cdot \\sqrt{L_1 L_2}',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Flujo Magnético:</b> Enlace entre inductores.</div>',
                controls: `<div class="control-group"><label>Acoplo k: <span id="val-k">0.8</span></label><input type="range" id="input-k" min="0.1" max="1" step="0.1" value="0.8" oninput="drawMutua()"></div>`,
                draw: () => drawMutua()
            },
            '3-2': {
                title: 'Misión 3.2: Regla de Puntos en Bobinas',
                chap: 'chap-3', next: '3-3', introChar: 'M',
                theory: 'Los puntos polares indican si la tensión inducida por acoplo magnético se suma o se resta a la tensión propia.',
                eq1: 'V_1 = j\\omega L_1 I_1 \\pm j\\omega M I_2',
                eq2: 'V_2 = j\\omega L_2 I_2 \\pm j\\omega M I_1',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Puntos de Polaridad:</b> Referencia de fase.</div>',
                controls: `<div class="control-group"><label>Fase i2: <span id="val-fi">0</span>°</label><input type="range" id="input-fi" min="0" max="180" step="180" value="0" oninput="drawDots()"></div>`,
                draw: () => drawDots()
            },
            '3-3': {
                title: 'Misión 3.3: El Transformador Ideal',
                chap: 'chap-3', next: '4-1', introChar: 'M',
                theory: 'Dispositivo sin pérdidas que escala el voltaje según la relación de vueltas $a = N_1 / N_2$.',
                eq1: '\\frac{V_1}{V_2} = \\frac{N_1}{N_2} = a',
                eq2: 'I_1 \\cdot N_1 = I_2 \\cdot N_2',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>V2 Salida:</b> Voltaje transformado.</div>',
                controls: `<div class="control-group"><label>N1/N2 Ratio: <span id="val-a">2</span></label><input type="range" id="input-a" min="0.5" max="4" step="0.5" value="2" oninput="drawTransfo()"></div>`,
                draw: () => drawTransfo()
            },
            '4-1': {
                title: 'Misión 4.1: Generador Trifásico y Secuencias',
                chap: 'chap-4', next: '4-2', introChar: 'VL',
                theory: 'Tres fuentes AC desfasadas $120^\\circ$ entre sí generan mayor eficiencia en generación y distribución.',
                eq1: '\\mathbf{V}_{aN} = V_p \\angle 0^\\circ, \\mathbf{V}_{bN} = V_p \\angle -120^\\circ',
                eq2: 'V_L = \\sqrt{3} \\cdot V_P',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>3 Fases:</b> Fasores a 120°.</div>',
                controls: `<div class="control-group"><label>Vp: <span id="val-vp">120</span></label><input type="range" id="input-vp" min="60" max="240" value="120" oninput="drawTriGen()"></div>`,
                draw: () => drawTriGen()
            },
            '4-2': {
                title: 'Misión 4.2: Conexión Estrella Y / Delta Δ',
                chap: 'chap-4', next: '4-3', introChar: 'VL',
                theory: 'En Estrella ($Y$) $V_L = \\sqrt{3}V_P$. En Delta ($\\Delta$) la corriente de línea es $I_L = \\sqrt{3}I_P$.',
                eq1: 'Y: V_L = \\sqrt{3}V_P \\angle 30^\\circ',
                eq2: '\\Delta: I_L = \\sqrt{3}I_P \\angle -30^\\circ',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Línea vs Fase:</b> Factor √3.</div>',
                controls: `<div class="control-group"><label>Ip Carga: <span id="val-ip">10</span></label><input type="range" id="input-ip" min="2" max="20" value="10" oninput="drawYDelta()"></div>`,
                draw: () => drawYDelta()
            },
            '4-3': {
                title: 'Misión 4.3: Cargas Trifásicas Desequilibradas',
                chap: 'chap-4', next: null, introChar: 'VL',
                theory: 'Cuando las impedancias de cada fase son distintas, aparece una corriente de neutro $I_N = I_a + I_b + I_c \\neq 0$.',
                eq1: '\\mathbf{I}_N = \\mathbf{I}_A + \\mathbf{I}_B + \\mathbf{I}_C',
                eq2: 'P_{total} = P_A + P_B + P_C',
                legend: '<div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>IN Neutro:</b> Corriente de desequilibrio.</div>',
                controls: `<div class="control-group"><label>Desequilibrio Fase A: <span id="val-des">50</span>%</label><input type="range" id="input-des" min="0" max="100" value="50" oninput="drawDeseb()"></div>`,
                draw: () => drawDeseb()
            }
        };

        /* --- ESTADO DEL JUGADOR Y DESBLOQUEOS --- */
        const gameState = {
            xp: 0, level: 1,
            unlockedChaps: ['chap-1'],
            completedQuests: [],
            unlockedQuests: ['1-1'],
            unlockedChars: ['V', 'I', 'w']
        };

        let activeQuest = null;

        /* --- INICIALIZACIÓN --- */
        window.onload = () => {
            updateUI();
        };

        function updateUI() {
            // Stats HUD
            document.getElementById('player-lvl').innerText = `Nivel ${gameState.level}`;
            const ranks = ["Novato AC", "Analista de Fasores", "Maestro de Impedancias", "Soberano Trifásico"];
            document.getElementById('player-rank').innerText = ranks[Math.min(gameState.level - 1, ranks.length - 1)];
            document.getElementById('xp-bar').style.width = `${Math.min((gameState.xp / (gameState.level * 200)) * 100, 100)}%`;

            // Actualizar Capítulos
            ['chap-1', 'chap-2', 'chap-3', 'chap-4'].forEach((cId, idx) => {
                const el = document.getElementById(cId);
                const tag = document.getElementById(`tag-${cId}`);
                const isUnlocked = gameState.unlockedChaps.includes(cId);
                
                if (isUnlocked) {
                    el.classList.remove('locked');
                    el.classList.add('unlocked');
                    tag.innerText = "DESBLOQUEADO";
                } else {
                    el.classList.add('locked');
                    tag.innerText = "BLOQUEADO 🔒";
                }

                // Conectores
                if (idx < 3) {
                    const conn = document.getElementById(`conn-${idx+1}-${idx+2}`);
                    if (conn && isUnlocked) conn.classList.add('active');
                }
            });

            // Actualizar Misiones
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
        }

        function toggleChapter(id) {
            if (!gameState.unlockedChaps.includes(id)) return;
            document.getElementById(id).classList.toggle('open');
        }

        /* --- MANEJO DE MISIONES --- */
        function openQuest(key) {
            if (!gameState.unlockedQuests.includes(key)) return;
            activeQuest = key;
            const q = questsDB[key];

            // Verificar si hay personaje nuevo
            if (q.introChar && !gameState.unlockedChars.includes(q.introChar)) {
                gameState.unlockedChars.push(q.introChar);
                showCharReveal(q.introChar);
            }

            document.getElementById('modal-title').innerText = q.title;
            document.getElementById('theory-text').innerHTML = q.theory;

            katex.render(q.eq1, document.getElementById('eq-1'), { displayMode: true, throwOnError: false });
            katex.render(q.eq2, document.getElementById('eq-2'), { displayMode: true, throwOnError: false });

            document.getElementById('sim-legend').innerHTML = q.legend;
            document.getElementById('sim-controls').innerHTML = q.controls;

            switchStep('theory');
            document.getElementById('quest-modal').classList.add('active');
        }

        function switchStep(step) {
            document.getElementById('step-theory').classList.remove('active');
            document.getElementById('step-sim').classList.remove('active');

            if (step === 'theory') {
                document.getElementById('step-theory').classList.add('active');
            } else {
                document.getElementById('step-sim').classList.add('active');
                setTimeout(() => {
                    questsDB[activeQuest].draw();
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
            document.getElementById('quest-modal').classList.remove('active');
            activeQuest = null;
        }

        function completeCurrentQuest() {
            if (!activeQuest || gameState.completedQuests.includes(activeQuest)) {
                closeQuest();
                return;
            }

            gameState.completedQuests.push(activeQuest);
            const q = questsDB[activeQuest];

            // Desbloquear siguiente misión
            if (q.next && !gameState.unlockedQuests.includes(q.next)) {
                gameState.unlockedQuests.push(q.next);
                
                // Si la siguiente misión pertenece a otro capítulo, desbloquear capítulo
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

        /* --- CÓDEX DE PERSONAJES --- */
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

        function closeCodex() {
            document.getElementById('codex-modal').classList.remove('active');
        }

        function showCharReveal(cKey) {
            const c = charactersDB[cKey];
            document.getElementById('reveal-symbol').innerText = c.symbol;
            document.getElementById('reveal-title').innerText = c.name;
            document.getElementById('reveal-desc').innerHTML = c.desc;
            document.getElementById('reveal-modal').classList.add('active');
        }

        function closeReveal() {
            document.getElementById('reveal-modal').classList.remove('active');
        }

        /* --- DIBUJO DE SIMULADORES (CANVAS) --- */
        function drawSenoidal() {
            const Vm = parseFloat(document.getElementById('input-vm').value);
            document.getElementById('val-vm').innerText = Vm;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0, 0, cv.width, cv.height);

            // Vector
            ctx.strokeStyle = '#ff0055'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(100, 110); ctx.lineTo(100 + Vm*0.5, 60); ctx.stroke();

            // Onda
            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 2;
            ctx.beginPath();
            for(let x=220; x<620; x++) {
                const y = 110 - (Vm*0.5) * Math.sin((x-220)*0.03);
                if(x===220) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();
        }

        function drawRLC() {
            const L = parseFloat(document.getElementById('input-l').value);
            document.getElementById('val-l').innerText = L;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0,0,cv.width,cv.height);

            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 2; ctx.beginPath();
            for(let x=50; x<600; x++) {
                const f = (x-50)*2;
                const Z = Math.sqrt(20*20 + Math.pow(f*L*0.1 - 2000/(f+1), 2));
                const y = 200 - Math.min(Z*0.8, 180);
                if(x===50) ctx.moveTo(x,y); else ctx.lineTo(x,y);
            }
            ctx.stroke();
        }

        function drawKCL() {
            const Z = document.getElementById('input-z').value;
            document.getElementById('val-z').innerText = Z;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0,0,cv.width,cv.height);
            ctx.fillStyle = '#00f3ff'; ctx.font = '14px Orbitron';
            ctx.fillText(`Malla 1: (10 + j${Z}) I1 - (j${Z}) I2 = 120∠0°`, 150, 100);
        }

        function drawPinst() {
            const V = document.getElementById('input-vrms').value;
            document.getElementById('val-vrms').innerText = V;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0,0,cv.width,cv.height);
            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 2; ctx.beginPath();
            ctx.moveTo(50, 110); ctx.lineTo(600, 110); ctx.stroke();
            ctx.fillStyle = '#ffb700'; ctx.fillText(`P_prom = ${(V*2).toFixed(0)} W`, 270, 95);
        }

        function drawTriang() {
            const Q = document.getElementById('input-q').value;
            document.getElementById('val-q').innerText = Q;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0,0,cv.width,cv.height);
            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 3; ctx.beginPath();
            ctx.moveTo(100,180); ctx.lineTo(300,180); ctx.lineTo(300, 180 - Q*0.3); ctx.closePath(); ctx.stroke();
        }

        function drawFP() {
            const QC = document.getElementById('input-qc').value;
            document.getElementById('val-qc').innerText = QC;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0,0,cv.width,cv.height);
            ctx.fillStyle = '#00f3ff'; ctx.font = '14px Orbitron';
            ctx.fillText(`Q Compensada = ${300 - QC} VAR | FP mejorado`, 180, 110);
        }

        function drawMutua() {
            const k = document.getElementById('input-k').value;
            document.getElementById('val-k').innerText = k;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0,0,cv.width,cv.height);
            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = k * 6;
            ctx.beginPath(); ctx.arc(200,110,40,0,Math.PI*2); ctx.stroke();
            ctx.beginPath(); ctx.arc(450,110,40,0,Math.PI*2); ctx.stroke();
        }

        function drawDots() {
            const fi = document.getElementById('input-fi').value;
            document.getElementById('val-fi').innerText = fi;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0,0,cv.width,cv.height);
            ctx.fillStyle = '#00f3ff'; ctx.beginPath(); ctx.arc(200,70,6,0,Math.PI*2); ctx.fill();
            ctx.beginPath(); ctx.arc(450,70,6,0,Math.PI*2); ctx.fill();
        }

        function drawTransfo() {
            const a = document.getElementById('input-a').value;
            document.getElementById('val-a').innerText = a;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0,0,cv.width,cv.height);
            ctx.fillStyle = '#fff'; ctx.font = '14px Orbitron';
            ctx.fillText(`V1 = 120V  ===>  V2 = ${(120/a).toFixed(1)}V`, 220, 110);
        }

        function drawTriGen() {
            const Vp = document.getElementById('input-vp').value;
            document.getElementById('val-vp').innerText = Vp;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0,0,cv.width,cv.height);
            ctx.strokeStyle = '#ff0055'; ctx.beginPath(); ctx.moveTo(325,110); ctx.lineTo(325+Vp*0.5, 110); ctx.stroke();
            ctx.strokeStyle = '#00f3ff'; ctx.beginPath(); ctx.moveTo(325,110); ctx.lineTo(325-Vp*0.25, 110+Vp*0.4); ctx.stroke();
            ctx.strokeStyle = '#00ff66'; ctx.beginPath(); ctx.moveTo(325,110); ctx.lineTo(325-Vp*0.25, 110-Vp*0.4); ctx.stroke();
        }

        function drawYDelta() {
            const Ip = document.getElementById('input-ip').value;
            document.getElementById('val-ip').innerText = Ip;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0,0,cv.width,cv.height);
            ctx.fillStyle = '#fff'; ctx.font = '14px Orbitron';
            ctx.fillText(`Corriente de Línea IL = ${(Ip * 1.732).toFixed(1)} A`, 200, 110);
        }

        function drawDeseb() {
            const des = document.getElementById('input-des').value;
            document.getElementById('val-des').innerText = des;
            const cv = document.getElementById('sim-canvas'), ctx = cv.getContext('2d');
            ctx.clearRect(0,0,cv.width,cv.height);
            ctx.fillStyle = '#ffb700'; ctx.font = '14px Orbitron';
            ctx.fillText(`Corriente de Neutro IN = ${(des * 0.15).toFixed(2)} A`, 200, 110);
        }
    </script>
</body>
</html>