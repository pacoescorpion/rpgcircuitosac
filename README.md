<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Análisis de Circuitos AC - Árbol RPG</title>
    <!-- KaTeX CSS & JS -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.css">
    <script src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.js"></script>
    <!-- Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700;900&family=Rajdhani:wght@500;600;700&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --bg-dark: #070a12;
            --panel-bg: #0d1424;
            --panel-border: rgba(0, 243, 255, 0.3);
            --cyan: #00f3ff;
            --green: #00ff66;
            --yellow: #ffb700;
            --red: #ff0055;
            --text-main: #e2e8f0;
            --text-muted: #94a3b8;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }

        body {
            background-color: var(--bg-dark);
            background-image: 
                radial-gradient(circle at 50% 50%, rgba(0, 243, 255, 0.04) 0%, transparent 80%),
                linear-gradient(to right, rgba(255, 255, 255, 0.02) 1px, transparent 1px),
                linear-gradient(to bottom, rgba(255, 255, 255, 0.02) 1px, transparent 1px);
            background-size: 100% 100%, 30px 30px, 30px 30px;
            color: var(--text-main);
            font-family: 'Rajdhani', sans-serif;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
        }

        /* --- HEADER RPG --- */
        header {
            background: rgba(7, 10, 18, 0.95);
            border-bottom: 2px solid var(--cyan);
            box-shadow: 0 0 15px rgba(0, 243, 255, 0.2);
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
            text-shadow: 0 0 8px var(--cyan);
        }

        .hud-stats { display: flex; gap: 20px; align-items: center; }
        .stat-box { display: flex; flex-direction: column; align-items: flex-end; }
        .stat-label { font-size: 0.75rem; color: var(--text-muted); text-transform: uppercase; }
        .stat-value { font-family: 'Orbitron', sans-serif; font-size: 1.1rem; color: var(--cyan); font-weight: 700; }
        
        .xp-container {
            width: 140px; height: 10px;
            background: rgba(255,255,255,0.1);
            border: 1px solid var(--cyan);
            border-radius: 5px; overflow: hidden; margin-top: 3px;
        }
        .xp-bar { height: 100%; width: 0%; background: linear-gradient(90deg, var(--cyan), var(--green)); transition: width 0.4s ease; }

        /* --- MAIN TREE CONTAINER --- */
        #tree-container {
            max-width: 1100px;
            width: 100%;
            margin: 30px auto;
            padding: 0 20px;
            display: flex;
            flex-direction: column;
            gap: 25px;
        }

        .chapter-card {
            background: var(--panel-bg);
            border: 1px solid var(--panel-border);
            border-radius: 10px;
            overflow: hidden;
            box-shadow: 0 4px 20px rgba(0,0,0,0.5);
        }

        .chapter-header {
            padding: 18px 25px;
            background: rgba(0, 243, 255, 0.05);
            display: flex;
            justify-content: space-between;
            align-items: center;
            cursor: pointer;
            transition: background 0.2s;
        }

        .chapter-header:hover { background: rgba(0, 243, 255, 0.1); }

        .chapter-title {
            font-family: 'Orbitron', sans-serif;
            font-size: 1.1rem;
            color: var(--cyan);
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .chapter-arrow {
            font-size: 1.2rem;
            color: var(--cyan);
            transition: transform 0.3s ease;
        }

        .chapter-card.open .chapter-arrow { transform: rotate(90deg); }

        /* SUB-TREE / MISSIONS */
        .sub-tree {
            display: none;
            padding: 20px 25px;
            border-top: 1px solid rgba(255,255,255,0.05);
            background: rgba(0,0,0,0.2);
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 15px;
        }

        .chapter-card.open .sub-tree { display: grid; }

        .mission-node {
            background: rgba(13, 20, 36, 0.9);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 8px;
            padding: 15px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            cursor: pointer;
            transition: all 0.2s ease;
        }

        .mission-node.unlocked { border-color: var(--cyan); }
        .mission-node.unlocked:hover { transform: translateY(-2px); box-shadow: 0 0 15px rgba(0, 243, 255, 0.2); }
        .mission-node.completed { border-color: var(--green); background: rgba(0, 255, 102, 0.05); }
        .mission-node.locked { opacity: 0.4; cursor: not-allowed; filter: grayscale(1); }

        .mission-info { display: flex; align-items: center; gap: 12px; }
        .mission-icon { font-size: 1.5rem; }
        .mission-name { font-weight: 600; font-size: 0.95rem; color: #fff; }
        .mission-tag { font-size: 0.7rem; font-family: 'Orbitron', sans-serif; padding: 2px 6px; border-radius: 4px; }
        .unlocked .mission-tag { color: var(--cyan); border: 1px solid var(--cyan); }
        .completed .mission-tag { color: var(--green); border: 1px solid var(--green); }
        .locked .mission-tag { color: var(--text-muted); border: 1px solid var(--text-muted); }

        /* --- MODAL DE MISION --- */
        .modal-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(4, 6, 12, 0.9);
            backdrop-filter: blur(6px);
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
            box-shadow: 0 0 30px rgba(0, 243, 255, 0.25);
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

        /* STEP NAVIGATION (TEORIA -> SIMULADOR) */
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

        /* SIMULATOR & LEGEND */
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
            cursor: pointer; transition: all 0.2s ease; align-self: flex-end;
            display: flex; align-items: center; gap: 8px; margin-top: 10px;
        }
        .nav-btn:hover { background: var(--cyan); color: #000; box-shadow: 0 0 15px var(--cyan); }
    </style>
</head>
<body>

    <!-- HEADER / HUD -->
    <header>
        <div class="hud-title">⚡ ANALISIS AC: RPG ACADEMY</div>
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
    </header>

    <!-- CONTAINER DE CAPÍTULOS Y SUB-ÁRBOLES -->
    <div id="tree-container">
        
        <!-- CAPÍTULO 1 -->
        <div class="chapter-card open" id="chap-1">
            <div class="chapter-header" onclick="toggleChapter('chap-1')">
                <div class="chapter-title"><span>📘 CAPÍTULO 1:</span> Análisis Senoidal en Estado Estable</div>
                <div class="chapter-arrow">▶</div>
            </div>
            <div class="sub-tree">
                <div class="mission-node unlocked" id="node-senoidal" onclick="openQuest('senoidal')">
                    <div class="mission-info">
                        <span class="mission-icon">🌊</span>
                        <div>
                            <div class="mission-name">Onda Senoidal & Fasores</div>
                            <span class="mission-tag">MISIÓN 1.1</span>
                        </div>
                    </div>
                    <span>➔</span>
                </div>
                <div class="mission-node locked" id="node-impedancia" onclick="openQuest('impedancia')">
                    <div class="mission-info">
                        <span class="mission-icon">⚡</span>
                        <div>
                            <div class="mission-name">Impedancia & Resonancia RLC</div>
                            <span class="mission-tag">MISIÓN 1.2</span>
                        </div>
                    </div>
                    <span>➔</span>
                </div>
            </div>
        </div>

        <!-- CAPÍTULO 2 -->
        <div class="chapter-card" id="chap-2">
            <div class="chapter-header" onclick="toggleChapter('chap-2')">
                <div class="chapter-title"><span>💡 CAPÍTULO 2:</span> Potencia en Estado Estacionario</div>
                <div class="chapter-arrow">▶</div>
            </div>
            <div class="sub-tree">
                <div class="mission-node locked" id="node-potencia" onclick="openQuest('potencia')">
                    <div class="mission-info">
                        <span class="mission-icon">📐</span>
                        <div>
                            <div class="mission-name">Potencia Compleja & FP</div>
                            <span class="mission-tag">MISIÓN 2.1</span>
                        </div>
                    </div>
                    <span>➔</span>
                </div>
            </div>
        </div>

        <!-- CAPÍTULO 3 -->
        <div class="chapter-card" id="chap-3">
            <div class="chapter-header" onclick="toggleChapter('chap-3')">
                <div class="chapter-title"><span>🧲 CAPÍTULO 3:</span> Circuitos Acoplados Magnéticamente</div>
                <div class="chapter-arrow">▶</div>
            </div>
            <div class="sub-tree">
                <div class="mission-node locked" id="node-acoplamiento" onclick="openQuest('acoplamiento')">
                    <div class="mission-info">
                        <span class="mission-icon">🔄</span>
                        <div>
                            <div class="mission-name">Inducción Mutua & Transformadores</div>
                            <span class="mission-tag">MISIÓN 3.1</span>
                        </div>
                    </div>
                    <span>➔</span>
                </div>
            </div>
        </div>

        <!-- CAPÍTULO 4 -->
        <div class="chapter-card" id="chap-4">
            <div class="chapter-header" onclick="toggleChapter('chap-4')">
                <div class="chapter-title"><span>🏢 CAPÍTULO 4:</span> Sistemas Polifásicos</div>
                <div class="chapter-arrow">▶</div>
            </div>
            <div class="sub-tree">
                <div class="mission-node locked" id="node-trifasicos" onclick="openQuest('trifasicos')">
                    <div class="mission-info">
                        <span class="mission-icon">🌐</span>
                        <div>
                            <div class="mission-name">Sistemas Trifásicos Y / Δ</div>
                            <span class="mission-tag">MISIÓN 4.1</span>
                        </div>
                    </div>
                    <span>➔</span>
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
                
                <!-- PASO 1: TEORÍA BREVE -->
                <div class="step-view active" id="step-theory">
                    <div class="theory-box" id="theory-text">
                        <!-- Texto conciso -->
                    </div>
                    
                    <div class="math-card">
                        <div class="math-title">Fórmula Fundamental</div>
                        <div class="math-eq" id="eq-1"></div>
                    </div>

                    <div class="math-card">
                        <div class="math-title">Transformación / Resultado Final</div>
                        <div class="math-eq" id="eq-2"></div>
                    </div>

                    <button class="nav-btn" onclick="switchStep('sim')">
                        Continuar al Simulador ➔
                    </button>
                </div>

                <!-- PASO 2: SIMULADOR INTERACTIVO -->
                <div class="step-view" id="step-sim">
                    <div class="sim-box">
                        <canvas id="sim-canvas" width="650" height="230"></canvas>
                        
                        <!-- LEYENDA EXPLICATIVA DEL SIMULADOR -->
                        <div class="legend-box" id="sim-legend">
                            <!-- Leyenda inyectada por JS -->
                        </div>

                        <div class="controls-grid" id="sim-controls">
                            <!-- Controles inyectados por JS -->
                        </div>
                    </div>

                    <div style="display: flex; justify-content: space-between; width: 100%;">
                        <button class="nav-btn" style="background: rgba(255,255,255,0.1); border-color: var(--text-muted);" onclick="switchStep('theory')">
                            ⬅ Volver a Teoría
                        </button>
                        <button class="nav-btn" id="btn-complete" onclick="completeCurrentQuest()">
                            Completar Misión (+100 XP) ✔
                        </button>
                    </div>
                </div>

            </div>
        </div>
    </div>

    <script>
        /* --- ESTADO Y PROGRESO --- */
        const gameState = {
            xp: 0,
            level: 1,
            completedNodes: [],
            unlockedNodes: ['senoidal']
        };

        const ranks = ["Novato AC", "Analista de Fasores", "Maestro de Impedancias", "Soberano Trifásico"];

        /* --- BASE DE DATOS DE MISIONES Y ECUACIONES --- */
        const questDB = {
            senoidal: {
                title: "Unidad 1: Onda Senoidal y Fasores",
                reward: 100,
                next: ['impedancia'],
                theory: "Las señales senoidales cambian continuamente en el tiempo. Para simplificar su análisis sin resolver ecuaciones diferenciales, las convertimos a <b>Fasores</b> (vectores en el plano complejo con amplitud y ángulo).",
                eq1: "v(t) = V_m \\cdot \\cos(\\omega t + \\phi)",
                eq2: "\\mathbf{V} = V_{rms} \\angle \\phi = \\frac{V_m}{\\sqrt{2}} e^{j\\phi}",
                legend: `
                    <div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Línea Roja (Fasor V):</b> Vector giratorio en el plano complejo ($Re$, $Im$) con magnitud $V_m$ y ángulo $\\phi$.</div>
                    <div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Línea Punteada (Proyección):</b> Muestra la altura vertical instantánea transmitida hacia la onda temporal.</div>
                    <div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Línea Azul (Osciloscopio):</b> La forma de onda $v(t)$ generada al transcurrir el tiempo.</div>
                `,
                controls: `
                    <div class="control-group">
                        <label>Amplitud $V_m$ (V): <span id="val-vm">100</span></label>
                        <input type="range" id="input-vm" min="30" max="150" value="100" oninput="drawSimSenoidal()">
                    </div>
                    <div class="control-group">
                        <label>Desfase $\\phi$ (grados): <span id="val-phi">45</span>°</label>
                        <input type="range" id="input-phi" min="-180" max="180" value="45" oninput="drawSimSenoidal()">
                    </div>
                `,
                initSim: () => drawSimSenoidal()
            },
            impedancia: {
                title: "Unidad 1.2: Impedancia y Resonancia RLC",
                reward: 150,
                next: ['potencia'],
                theory: "La <b>Impedancia ($Z$)</b> representa la oposición al paso de corriente alterna. En un circuito RLC, existe una frecuencia especial de <b>Resonancia ($f_0$)</b> donde las reactancias inductiva ($X_L$) y capacitiva ($X_C$) se cancelan mutuamente.",
                eq1: "\\mathbf{Z} = R + j(X_L - X_C) = R + j\\left(\\omega L - \\frac{1}{\\omega C}\\right)",
                eq2: "f_0 = \\frac{1}{2\\pi \\sqrt{L C}} \\quad (\\text{cuando } X_L = X_C)",
                legend: `
                    <div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Curva Verde (|Z|):</b> Magnitud de la impedancia total según la frecuencia.</div>
                    <div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Línea Amarilla (f0):</b> Frecuencia exacta de resonancia donde la impedancia es mínima ($|Z| = R$).</div>
                `,
                controls: `
                    <div class="control-group">
                        <label>Inductancia L (mH): <span id="val-l">10</span></label>
                        <input type="range" id="input-l" min="1" max="50" value="10" oninput="drawSimRLC()">
                    </div>
                    <div class="control-group">
                        <label>Capacitancia C (µF): <span id="val-c">10</span></label>
                        <input type="range" id="input-c" min="1" max="50" value="10" oninput="drawSimRLC()">
                    </div>
                `,
                initSim: () => drawSimRLC()
            },
            potencia: {
                title: "Unidad 2: Potencia Compleja y Factor de Potencia",
                reward: 150,
                next: ['acoplamiento'],
                theory: "En AC, la energía se divide en <b>Potencia Activa ($P$)</b> (trabajo real) y <b>Potencia Reactiva ($Q$)</b> (campo magnético/eléctrico). El <b>Factor de Potencia ($FP$)</b> mide la eficiencia del sistema y se corrige inyectando capacitores.",
                eq1: "\\mathbf{S} = P + jQ = \\mathbf{V}_{rms} \\mathbf{I}_{rms}^*",
                eq2: "FP = \\cos(\\theta) = \\frac{P}{|S|}",
                legend: `
                    <div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Línea Azul (P):</b> Potencia Activa Útil (Watts).</div>
                    <div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Línea Roja (Q):</b> Potencia Reactiva Neta $Q_L - Q_C$ (VAR).</div>
                    <div class="legend-item"><div class="legend-color" style="background:#00ff66;"></div> <b>Línea Verde (S):</b> Potencia Aparente Total (VA).</div>
                `,
                controls: `
                    <div class="control-group">
                        <label>Carga Inductiva QL (VAR): <span id="val-ql">300</span></label>
                        <input type="range" id="input-ql" min="50" max="500" value="300" oninput="drawSimPotencia()">
                    </div>
                    <div class="control-group">
                        <label>Capacitor Compensador QC: <span id="val-qc">150</span></label>
                        <input type="range" id="input-qc" min="0" max="500" value="150" oninput="drawSimPotencia()">
                    </div>
                `,
                initSim: () => drawSimPotencia()
            },
            acoplamiento: {
                title: "Unidad 3: Inducción Mutua y Transformadores",
                reward: 200,
                next: ['trifasicos'],
                theory: "El flujo magnético variable de una bobina puede inducir tensión en una bobina cercana por <b>Inductancia Mutua ($M$)</b>. Un transformador altera el voltaje según la razón de espiras ($a = N_1/N_2$).",
                eq1: "v_2(t) = M \\frac{di_1(t)}{dt}",
                eq2: "a = \\frac{N_1}{N_2} = \\frac{V_1}{V_2}",
                legend: `
                    <div class="legend-item"><div class="legend-color" style="background:#ffb700;"></div> <b>Primario (N1):</b> Bobinado alimentado con voltaje de entrada $V_1$.</div>
                    <div class="legend-item"><div class="legend-color" style="background:#00f3ff;"></div> <b>Secundario (N2):</b> Voltaje inducido de salida $V_2$ según la relación de transformación.</div>
                `,
                controls: `
                    <div class="control-group">
                        <label>Vueltas Primario (N1): <span id="val-n1">500</span></label>
                        <input type="range" id="input-n1" min="100" max="1000" value="500" oninput="drawSimTransfo()">
                    </div>
                    <div class="control-group">
                        <label>Vueltas Secundario (N2): <span id="val-n2">250</span></label>
                        <input type="range" id="input-n2" min="50" max="1000" value="250" oninput="drawSimTransfo()">
                    </div>
                `,
                initSim: () => drawSimTransfo()
            },
            trifasicos: {
                title: "Unidad 4: Sistemas Trifásicos (Estrella Y / Delta Δ)",
                reward: 200,
                next: [],
                theory: "Los sistemas trifásicos usan tres tensiones senoidales desfasadas $120^\\circ$. En conexión Estrella (Y), la tensión entre dos líneas ($V_L$) es $\\sqrt{3}$ veces mayor que la tensión de fase ($V_P$).",
                eq1: "\\mathbf{V}_{aN} = V_P \\angle 0^\\circ, \\quad \\mathbf{V}_{bN} = V_P \\angle -120^\\circ, \\quad \\mathbf{V}_{cN} = V_P \\angle 120^\\circ",
                eq2: "V_{linea} = \\sqrt{3} \\cdot V_{fase} \\approx 1.732 \\cdot V_P",
                legend: `
                    <div class="legend-item"><div class="legend-color" style="background:#ff0055;"></div> <b>Fase A ($0^\circ$)</b> | <div class="legend-color" style="background:#00f3ff;"></div> <b>Fase B ($-120^\circ$)</b> | <div class="legend-color" style="background:#00ff66;"></div> <b>Fase C ($120^\circ$)</b></div>
                `,
                controls: `
                    <div class="control-group">
                        <label>Tensión de Fase Vp (V): <span id="val-vp">120</span></label>
                        <input type="range" id="input-vp" min="50" max="240" value="120" oninput="drawSimTrifasico()">
                    </div>
                `,
                initSim: () => drawSimTrifasico()
            }
        };

        let activeQuestKey = null;

        /* --- CONTROL DE CAPÍTULOS Y DESPLEGABLES --- */
        function toggleChapter(id) {
            document.getElementById(id).classList.toggle('open');
        }

        function updateUI() {
            document.getElementById('player-lvl').innerText = `Nivel ${gameState.level}`;
            document.getElementById('player-rank').innerText = ranks[Math.min(gameState.level - 1, ranks.length - 1)];
            
            const xpMax = gameState.level * 200;
            const pct = Math.min((gameState.xp / xpMax) * 100, 100);
            document.getElementById('xp-bar').style.width = `${pct}%`;

            Object.keys(questDB).forEach(key => {
                const node = document.getElementById(`node-${key}`);
                if (!node) return;

                const tag = node.querySelector('.mission-tag');
                if (gameState.completedNodes.includes(key)) {
                    node.className = "mission-node completed";
                    tag.innerText = "COMPLETADO ✔";
                } else if (gameState.unlockedNodes.includes(key)) {
                    node.className = "mission-node unlocked";
                    tag.innerText = "DISPONIBLE";
                } else {
                    node.className = "mission-node locked";
                    tag.innerText = "BLOQUEADO 🔒";
                }
            });
        }

        /* --- ABRIR Y NAVEGAR MISIÓN --- */
        function openQuest(key) {
            if (!gameState.unlockedNodes.includes(key)) return;
            
            activeQuestKey = key;
            const data = questDB[key];

            document.getElementById('modal-title').innerText = data.title;
            document.getElementById('theory-text').innerHTML = `<p>${data.theory}</p>`;
            
            // Render de fórmulas KaTeX limpias de error
            katex.render(data.eq1, document.getElementById('eq-1'), { displayMode: true, throwOnError: false });
            katex.render(data.eq2, document.getElementById('eq-2'), { displayMode: true, throwOnError: false });

            // Cargar controles y leyenda
            document.getElementById('sim-legend').innerHTML = data.legend;
            document.getElementById('sim-controls').innerHTML = data.controls;

            // Reset a paso teoría
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
                    questDB[activeQuestKey].initSim();
                    renderMathInLegend();
                }, 50);
            }
        }

        function renderMathInLegend() {
            // Renderizar notación KaTeX en leyendas o controles si existen
            const legendEl = document.getElementById('sim-legend');
            if (window.renderMathInElement && legendEl) {
                renderMathInElement(legendEl, {
                    delimiters: [{left: '$', right: '$', display: false}],
                    throwOnError: false
                });
            }
        }

        function closeQuest() {
            document.getElementById('quest-modal').classList.remove('active');
            activeQuestKey = null;
        }

        function completeCurrentQuest() {
            if (activeQuestKey && !gameState.completedNodes.includes(activeQuestKey)) {
                gameState.completedNodes.push(activeQuestKey);
                const data = questDB[activeQuestKey];

                data.next.forEach(n => {
                    if (!gameState.unlockedNodes.includes(n)) gameState.unlockedNodes.push(n);
                });

                gameState.xp += data.reward;
                if (gameState.xp >= gameState.level * 200) gameState.level++;

                updateUI();
            }
            closeQuest();
        }

        /* =========================================================
           SIMULADORES INTERACTIVOS (CORREGIDOS Y CLAROS)
           ========================================================= */

        // 1. SIMULADOR: ONDA SENOIDAL Y FASOR (PROYECCIÓN CORREGIDA)
        function drawSimSenoidal() {
            const Vm = parseFloat(document.getElementById('input-vm').value);
            const phiDeg = parseFloat(document.getElementById('input-phi').value);
            document.getElementById('val-vm').innerText = Vm;
            document.getElementById('val-phi').innerText = phiDeg;

            const canvas = document.getElementById('sim-canvas');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            const cy = 115;
            const cxPhasor = 120;
            const phiRad = (phiDeg * Math.PI) / 180;
            const r = Vm * 0.65;

            // Plano Complejo (Izquierda)
            ctx.strokeStyle = 'rgba(255,255,255,0.15)'; ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(20, cy); ctx.lineTo(220, cy); // Eje Real
            ctx.moveTo(cxPhasor, 15); ctx.lineTo(cxPhasor, 215); // Eje Imag
            ctx.stroke();

            // Círculo de Amplitud Max
            ctx.strokeStyle = 'rgba(0,243,255,0.1)';
            ctx.beginPath(); ctx.arc(cxPhasor, cy, r, 0, Math.PI * 2); ctx.stroke();

            // Vector Fasor (Línea Roja)
            const fx = cxPhasor + r * Math.cos(-phiRad);
            const fy = cy + r * Math.sin(-phiRad);

            ctx.strokeStyle = '#ff0055'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(cxPhasor, cy); ctx.lineTo(fx, fy); ctx.stroke();

            // Punta de flecha del fasor
            ctx.fillStyle = '#ff0055';
            ctx.beginPath(); ctx.arc(fx, fy, 4, 0, Math.PI * 2); ctx.fill();

            // Línea de Proyección Punteada (Amarilla) desde el fasor a la onda senoidal
            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 1.5; ctx.setLineDash([4, 4]);
            ctx.beginPath(); ctx.moveTo(fx, fy); ctx.lineTo(260, fy); ctx.stroke();
            ctx.setLineDash([]);

            // Ejes de Osciloscopio (Derecha)
            ctx.strokeStyle = 'rgba(255,255,255,0.15)';
            ctx.beginPath(); ctx.moveTo(260, cy); ctx.lineTo(630, cy); ctx.stroke();

            // Onda Senoidal (Azul)
            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 2.5;
            ctx.beginPath();
            for (let x = 260; x <= 630; x++) {
                const t = (x - 260) * 0.02;
                const y = cy - r * Math.sin(t + phiRad);
                if (x === 260) ctx.moveTo(x, y);
                else ctx.lineTo(x, y);
            }
            ctx.stroke();
        }

        // 2. SIMULADOR: RESONANCIA RLC
        function drawSimRLC() {
            const L = parseFloat(document.getElementById('input-l').value) * 1e-3;
            const C = parseFloat(document.getElementById('input-c').value) * 1e-6;
            document.getElementById('val-l').innerText = document.getElementById('input-l').value;
            document.getElementById('val-c').innerText = document.getElementById('input-c').value;

            const canvas = document.getElementById('sim-canvas');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            const f0 = 1 / (2 * Math.PI * Math.sqrt(L * C));

            // Curva de Impedancia
            ctx.strokeStyle = '#00ff66'; ctx.lineWidth = 2.5;
            ctx.beginPath();
            for (let x = 40; x < 610; x++) {
                const f = (x - 40) * 5;
                const w = 2 * Math.PI * f;
                const Xl = w * L;
                const Xc = w > 0 ? 1 / (w * C) : 1000;
                const Z = Math.sqrt(15*15 + Math.pow(Xl - Xc, 2));

                const y = 200 - Math.min(Z * 1.2, 180);
                if (x === 40) ctx.moveTo(x, y);
                else ctx.lineTo(x, y);
            }
            ctx.stroke();

            // Marca Resonancia f0
            const xRes = 40 + (f0 / 5);
            if (xRes >= 40 && xRes <= 610) {
                ctx.strokeStyle = '#ffb700'; ctx.setLineDash([4, 4]);
                ctx.beginPath(); ctx.moveTo(xRes, 10); ctx.lineTo(xRes, 210); ctx.stroke();
                ctx.setLineDash([]);

                ctx.fillStyle = '#ffb700'; ctx.font = '12px Orbitron';
                ctx.fillText(`f0 = ${Math.round(f0)} Hz`, Math.min(xRes + 8, 500), 30);
            }
        }

        // 3. SIMULADOR: TRIÁNGULO DE POTENCIAS
        function drawSimPotencia() {
            const QL = parseFloat(document.getElementById('input-ql').value);
            const QC = parseFloat(document.getElementById('input-qc').value);
            document.getElementById('val-ql').innerText = QL;
            document.getElementById('val-qc').innerText = QC;

            const canvas = document.getElementById('sim-canvas');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            const P = 300;
            const Qnet = QL - QC;
            const S = Math.sqrt(P*P + Qnet*Qnet);
            const FP = (P / S).toFixed(2);

            const ox = 80, oy = 180, sc = 0.45;

            // Cateto P (Azul)
            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P * sc, oy); ctx.stroke();

            // Cateto Q (Rojo)
            ctx.strokeStyle = '#ff0055';
            ctx.beginPath(); ctx.moveTo(ox + P * sc, oy); ctx.lineTo(ox + P * sc, oy - Qnet * sc); ctx.stroke();

            // Hipotenusa S (Verde)
            ctx.strokeStyle = '#00ff66';
            ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P * sc, oy - Qnet * sc); ctx.stroke();

            // Métricas
            ctx.fillStyle = '#fff'; ctx.font = '13px Orbitron';
            ctx.fillText(`Potencia Aparente S = ${Math.round(S)} VA`, 320, 70);
            ctx.fillText(`Factor de Potencia (FP) = ${FP}`, 320, 100);
        }

        // 4. SIMULADOR: TRANSFORMADOR
        function drawSimTransfo() {
            const N1 = parseInt(document.getElementById('input-n1').value);
            const N2 = parseInt(document.getElementById('input-n2').value);
            document.getElementById('val-n1').innerText = N1;
            document.getElementById('val-n2').innerText = N2;

            const canvas = document.getElementById('sim-canvas');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            const V1 = 120;
            const V2 = (V1 * (N2 / N1)).toFixed(1);

            // Núcleo
            ctx.fillStyle = '#1e293b'; ctx.fillRect(220, 40, 200, 140);
            ctx.clearRect(260, 75, 120, 70);

            // Bobinas
            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 4;
            ctx.beginPath(); ctx.arc(215, 110, 32, 0, Math.PI * 2); ctx.stroke();
            
            ctx.strokeStyle = '#00f3ff';
            ctx.beginPath(); ctx.arc(425, 110, 32, 0, Math.PI * 2); ctx.stroke();

            ctx.fillStyle = '#fff'; ctx.font = '14px Orbitron';
            ctx.fillText(`V1 = ${V1} V`, 90, 115);
            ctx.fillText(`V2 = ${V2} V`, 480, 115);
        }

        // 5. SIMULADOR: SISTEMA TRIFÁSICO
        function drawSimTrifasico() {
            const Vp = parseFloat(document.getElementById('input-vp').value);
            document.getElementById('val-vp').innerText = Vp;

            const canvas = document.getElementById('sim-canvas');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            const cx = 325, cy = 115, r = Vp * 0.65;
            const angles = [0, -120, 120];
            const colors = ['#ff0055', '#00f3ff', '#00ff66'];

            angles.forEach((ang, i) => {
                const rad = (ang * Math.PI) / 180;
                const x = cx + r * Math.cos(rad);
                const y = cy + r * Math.sin(rad);

                ctx.strokeStyle = colors[i]; ctx.lineWidth = 3;
                ctx.beginPath(); ctx.moveTo(cx, cy); ctx.lineTo(x, y); ctx.stroke();
            });

            const VL = (Math.sqrt(3) * Vp).toFixed(1);
            ctx.fillStyle = '#fff'; ctx.font = '13px Orbitron';
            ctx.fillText(`Tensión entre Líneas (VL) = ${VL} V`, 25, 30);
        }

        window.onload = () => { updateUI(); };
    </script>
</body>
</html>