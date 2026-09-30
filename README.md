<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Análisis de Circuitos AC - Árbol de Habilidades RPG</title>
    <!-- KaTeX CSS para renderizado matemático impecable -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.css">
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800;900&family=Rajdhani:wght@500;600;700&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --bg-dark: #080c14;
            --panel-bg: rgba(13, 20, 36, 0.85);
            --cyan-glow: #00f3ff;
            --magenta-glow: #ff0055;
            --yellow-glow: #ffb700;
            --green-glow: #00ff66;
            --text-main: #e2e8f0;
            --text-muted: #94a3b8;
            --border-neon: rgba(0, 243, 255, 0.3);
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
        }

        body {
            background-color: var(--bg-dark);
            background-image: 
                radial-gradient(circle at 50% 50%, rgba(0, 243, 255, 0.05) 0%, transparent 80%),
                linear-gradient(to right, rgba(255, 255, 255, 0.02) 1px, transparent 1px),
                linear-gradient(to bottom, rgba(255, 255, 255, 0.02) 1px, transparent 1px);
            background-size: 100% 100%, 40px 40px, 40px 40px;
            color: var(--text-main);
            font-family: 'Rajdhani', sans-serif;
            min-height: 100vh;
            overflow-x: hidden;
            display: flex;
            flex-direction: column;
        }

        /* --- HEADER / HUD RPG --- */
        header {
            background: rgba(8, 12, 20, 0.95);
            border-bottom: 2px solid var(--cyan-glow);
            box-shadow: 0 0 20px rgba(0, 243, 255, 0.2);
            padding: 15px 30px;
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
            font-size: 1.4rem;
            font-weight: 800;
            color: #fff;
            text-shadow: 0 0 10px var(--cyan-glow);
            letter-spacing: 2px;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .hud-stats {
            display: flex;
            align-items: center;
            gap: 25px;
        }

        .stat-box {
            display: flex;
            flex-direction: column;
            align-items: flex-end;
        }

        .stat-label {
            font-size: 0.8rem;
            color: var(--text-muted);
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .stat-value {
            font-family: 'Orbitron', sans-serif;
            font-size: 1.2rem;
            color: var(--cyan-glow);
            font-weight: 700;
        }

        .xp-container {
            width: 180px;
            height: 12px;
            background: rgba(255, 255, 255, 0.1);
            border: 1px solid var(--cyan-glow);
            border-radius: 6px;
            overflow: hidden;
            position: relative;
            margin-top: 4px;
        }

        .xp-bar {
            height: 100%;
            width: 0%;
            background: linear-gradient(90deg, var(--cyan-glow), var(--green-glow));
            box-shadow: 0 0 10px var(--cyan-glow);
            transition: width 0.5s ease;
        }

        /* --- TREE CANVAS CONTAINER --- */
        #tree-container {
            flex: 1;
            position: relative;
            width: 100%;
            max-width: 1200px;
            margin: 40px auto;
            min-height: 650px;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        #connections-canvas {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 1;
        }

        .tree-grid {
            position: relative;
            z-index: 2;
            width: 100%;
            height: 100%;
            display: grid;
            grid-template-rows: repeat(3, 1fr);
            gap: 60px;
            padding: 20px;
        }

        .tree-tier {
            display: flex;
            justify-content: space-around;
            align-items: center;
        }

        /* --- SKILL NODES --- */
        .skill-node {
            width: 110px;
            height: 110px;
            background: var(--panel-bg);
            border: 2px solid var(--text-muted);
            border-radius: 50%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            position: relative;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.8);
            text-align: center;
            padding: 10px;
        }

        .skill-node::before {
            content: '';
            position: absolute;
            top: -6px; left: -6px; right: -6px; bottom: -6px;
            border-radius: 50%;
            border: 1px dashed var(--text-muted);
            transition: all 0.3s ease;
        }

        .skill-node.unlocked {
            border-color: var(--cyan-glow);
            box-shadow: 0 0 20px rgba(0, 243, 255, 0.4);
            background: radial-gradient(circle, rgba(0, 243, 255, 0.15) 0%, var(--panel-bg) 70%);
        }

        .skill-node.unlocked::before {
            border-color: var(--cyan-glow);
            animation: spin 12s linear infinite;
        }

        .skill-node.completed {
            border-color: var(--green-glow);
            box-shadow: 0 0 25px rgba(0, 255, 102, 0.5);
            background: radial-gradient(circle, rgba(0, 255, 102, 0.2) 0%, var(--panel-bg) 70%);
        }

        .skill-node.completed::before {
            border-color: var(--green-glow);
            border-style: solid;
        }

        .skill-node.locked {
            opacity: 0.5;
            filter: grayscale(1);
            cursor: not-allowed;
        }

        .skill-node:hover:not(.locked) {
            transform: scale(1.12);
            box-shadow: 0 0 30px var(--cyan-glow);
        }

        .node-icon {
            font-size: 2rem;
            margin-bottom: 4px;
        }

        .node-title {
            font-family: 'Orbitron', sans-serif;
            font-size: 0.75rem;
            font-weight: 700;
            line-height: 1.1;
            color: #fff;
        }

        .node-status {
            position: absolute;
            bottom: -8px;
            background: #000;
            border: 1px solid var(--cyan-glow);
            border-radius: 10px;
            padding: 2px 8px;
            font-size: 0.65rem;
            font-family: 'Orbitron', sans-serif;
            color: var(--cyan-glow);
        }

        .completed .node-status {
            border-color: var(--green-glow);
            color: var(--green-glow);
        }

        @keyframes spin {
            100% { transform: rotate(360deg); }
        }

        /* --- MODAL DE MISION / CONTENIDO --- */
        .modal-overlay {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(4, 6, 12, 0.85);
            backdrop-filter: blur(8px);
            z-index: 1000;
            display: flex;
            justify-content: center;
            align-items: center;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s ease;
        }

        .modal-overlay.active {
            opacity: 1;
            pointer-events: all;
        }

        .modal-card {
            background: var(--panel-bg);
            border: 2px solid var(--cyan-glow);
            border-radius: 12px;
            box-shadow: 0 0 40px rgba(0, 243, 255, 0.3);
            width: 90%;
            max-width: 900px;
            max-height: 90vh;
            display: flex;
            flex-direction: column;
            overflow: hidden;
            transform: scale(0.9);
            transition: transform 0.3s ease;
        }

        .modal-overlay.active .modal-card {
            transform: scale(1);
        }

        .modal-header {
            padding: 20px 30px;
            border-bottom: 1px solid var(--border-neon);
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: rgba(0, 243, 255, 0.05);
        }

        .modal-header h2 {
            font-family: 'Orbitron', sans-serif;
            color: var(--cyan-glow);
            font-size: 1.4rem;
            letter-spacing: 1px;
        }

        .close-btn {
            background: none;
            border: none;
            color: var(--text-muted);
            font-size: 2rem;
            cursor: pointer;
            transition: color 0.2s;
        }

        .close-btn:hover {
            color: var(--magenta-glow);
        }

        .modal-body {
            padding: 30px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 25px;
        }

        /* ESTILO PARA FÓRMULAS Y PASOS MATEMÁTICOS */
        .math-derivation-box {
            background: rgba(0, 0, 0, 0.6);
            border-left: 4px solid var(--cyan-glow);
            padding: 20px;
            border-radius: 0 8px 8px 0;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        .math-step {
            background: rgba(255, 255, 255, 0.03);
            padding: 12px 16px;
            border-radius: 6px;
            border: 1px solid rgba(255, 255, 255, 0.05);
        }

        .math-step-title {
            font-family: 'Orbitron', sans-serif;
            font-size: 0.85rem;
            color: var(--yellow-glow);
            margin-bottom: 6px;
            text-transform: uppercase;
        }

        .math-equation {
            font-size: 1.1rem;
            padding: 8px 0;
            color: #fff;
            overflow-x: auto;
        }

        .sim-container {
            background: #000;
            border: 1px solid var(--border-neon);
            border-radius: 8px;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 15px;
        }

        canvas {
            background: #050811;
            border-radius: 4px;
            border: 1px solid rgba(0, 243, 255, 0.2);
            max-width: 100%;
        }

        .controls-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
            width: 100%;
        }

        .control-group {
            display: flex;
            flex-direction: column;
            gap: 5px;
        }

        .control-group label {
            font-size: 0.85rem;
            color: var(--text-muted);
            display: flex;
            justify-content: space-between;
        }

        input[type="range"] {
            accent-color: var(--cyan-glow);
            cursor: pointer;
        }

        .btn-action {
            font-family: 'Orbitron', sans-serif;
            background: linear-gradient(135deg, rgba(0,243,255,0.2), rgba(0,255,102,0.2));
            border: 2px solid var(--cyan-glow);
            color: #fff;
            padding: 14px 28px;
            border-radius: 6px;
            font-size: 1rem;
            font-weight: 700;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 0 15px rgba(0,243,255,0.2);
            align-self: center;
            margin-top: 10px;
        }

        .btn-action:hover {
            background: linear-gradient(135deg, var(--cyan-glow), var(--green-glow));
            color: #000;
            box-shadow: 0 0 25px var(--cyan-glow);
            transform: translateY(-2px);
        }

        /* --- RESPONSIVE --- */
        @media (max-width: 768px) {
            header { flex-direction: column; gap: 10px; align-items: flex-start; }
            .hud-stats { width: 100%; justify-content: space-between; }
            .tree-grid { gap: 30px; }
            .skill-node { width: 85px; height: 85px; }
            .node-icon { font-size: 1.5rem; }
            .node-title { font-size: 0.65rem; }
        }
    </style>
</head>
<body>

    <!-- HUD / BARRA SUPERIOR RPG -->
    <header>
        <div class="hud-title">
            <span>⚡</span> CIRCUITY RPG: ANALISIS AC
        </div>
        <div class="hud-stats">
            <div class="stat-box">
                <span class="stat-label">Rango Ingeniero</span>
                <span class="stat-value" id="player-rank">Novato AC</span>
            </div>
            <div class="stat-box">
                <span class="stat-label">Nivel / Experiencia</span>
                <span class="stat-value" id="player-lvl">Nivel 1</span>
                <div class="xp-container">
                    <div class="xp-bar" id="xp-bar"></div>
                </div>
            </div>
        </div>
    </header>

    <!-- ÁRBOL DE HABILIDADES TIPO RPG -->
    <div id="tree-container">
        <canvas id="connections-canvas"></canvas>
        <div class="tree-grid">
            
            <!-- TIER 1: FUNDAMENTOS SENOIDALES -->
            <div class="tree-tier">
                <div class="skill-node unlocked" id="node-senoidal" onclick="openQuest('senoidal')">
                    <div class="node-icon">🌊</div>
                    <div class="node-title">Onda Senoidal & Fasores</div>
                    <div class="node-status">DISPONIBLE</div>
                </div>
            </div>

            <!-- TIER 2: IMPEDANCIA Y POTENCIA -->
            <div class="tree-tier">
                <div class="skill-node locked" id="node-impedancia" onclick="openQuest('impedancia')">
                    <div class="node-icon">⚡</div>
                    <div class="node-title">Impedancia & RLC</div>
                    <div class="node-status">BLOQUEADO</div>
                </div>
                <div class="skill-node locked" id="node-potencia" onclick="openQuest('potencia')">
                    <div class="node-icon">💡</div>
                    <div class="node-title">Potencia Compleja & FP</div>
                    <div class="node-status">BLOQUEADO</div>
                </div>
            </div>

            <!-- TIER 3: ACOPLAMIENTO Y TRIFÁSICOS -->
            <div class="tree-tier">
                <div class="skill-node locked" id="node-acoplamiento" onclick="openQuest('acoplamiento')">
                    <div class="node-icon">🧲</div>
                    <div class="node-title">Acoplamiento Magnético</div>
                    <div class="node-status">BLOQUEADO</div>
                </div>
                <div class="skill-node locked" id="node-trifasicos" onclick="openQuest('trifasicos')">
                    <div class="node-icon">🏢</div>
                    <div class="node-title">Sistemas Trifásicos</div>
                    <div class="node-status">BLOQUEADO</div>
                </div>
            </div>

        </div>
    </div>

    <!-- MODAL INTERACTIVO DE MISIÓN -->
    <div class="modal-overlay" id="quest-modal">
        <div class="modal-card">
            <div class="modal-header">
                <h2 id="modal-title">Título de la Misión</h2>
                <button class="close-btn" onclick="closeQuest()">&times;</button>
            </div>
            <div class="modal-body" id="modal-body">
                <!-- Se inyecta dinámicamente con explicaciones paso a paso y simuladores -->
            </div>
        </div>
    </div>

    <!-- KaTeX JS para renderizar matemática -->
    <script src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/contrib/auto-render.min.js"></script>

    <script>
        /* --- ESTADO DEL JUGADOR Y PROGRESIÓN RPG --- */
        const gameState = {
            xp: 0,
            level: 1,
            completedNodes: [],
            unlockedNodes: ['senoidal']
        };

        const ranks = ["Novato AC", "Analista de Fasores", "Maestro de Impedancias", "Soberano Trifásico"];

        /* --- BASE DE DATOS DE HABILIDADES Y DESGLOSE MATEMÁTICO --- */
        const questData = {
            senoidal: {
                title: "Unidad 1: Onda Senoidal y Transformada Fasorial",
                xpReward: 100,
                unlocks: ['impedancia', 'potencia'],
                content: `
                    <p>En corriente alterna (AC), la tensión y la corriente varían en forma de onda senoidal a lo largo del tiempo. Para analizar estos circuitos sin resolver complejas ecuaciones diferenciales, transformamos el dominio del tiempo al <b>dominio de la frecuencia (Fasores)</b>.</p>

                    <div class="math-derivation-box">
                        <div class="math-step">
                            <div class="math-step-title">Paso 1: Señal Senoidal en el Tiempo</div>
                            <p>Cualquier señal AC periódica se describe como:</p>
                            <div class="math-equation">$$v(t) = V_m \cdot \cos(\omega t + \phi)$$</div>
                            <small>Donde $V_m$ es la amplitud máxima, $\omega = 2\pi f$ es la frecuencia angular (rad/s) y $\phi$ es el desfase inicial.</small>
                        </div>

                        <div class="math-step">
                            <div class="math-step-title">Paso 2: Identidad de Euler (Fundamento Matemático)</div>
                            <p>Según la fórmula de Euler, un número complejo exponencial se relaciona con senos y cosenos:</p>
                            <div class="math-equation">$$e^{j\theta} = \cos(\theta) + j \cdot \sin(\theta)$$</div>
                            <p>Por lo tanto, la parte real es: $\cos(\theta) = \text{Re}\{e^{j\theta}\}$.</p>
                        </div>

                        <div class="math-step">
                            <div class="math-step-title">Paso 3: Transformación al Fasor</div>
                            <p>Escribimos $v(t) = \text{Re}\{V_m e^{j(\omega t + \phi)}\} = \text{Re}\{V_m e^{j\phi} e^{j\omega t}\}$. Al aislar la parte estática independiente del tiempo, obtenemos el **Fasor** ($\mathbf{V}$):</p>
                            <div class="math-equation">$$\mathbf{V} = V_{rms} \angle \phi = \frac{V_m}{\sqrt{2}} e^{j\phi}$$</div>
                        </div>
                    </div>

                    <h3>Simulador Interactivo: Generador de Onda y Vector Fasorial</h3>
                    <div class="sim-container">
                        <canvas id="sim-canvas" width="600" height="220"></canvas>
                        <div class="controls-grid">
                            <div class="control-group">
                                <label>Amplitud $V_m$ (V): <span id="val-vm">100</span></label>
                                <input type="range" id="input-vm" min="20" max="150" value="100" oninput="updateSimSenoidal()">
                            </div>
                            <div class="control-group">
                                <label>Desfase $\phi$ (grados): <span id="val-phi">45</span>°</label>
                                <input type="range" id="input-phi" min="-180" max="180" value="45" oninput="updateSimSenoidal()">
                            </div>
                        </div>
                    </div>
                `
            },
            impedancia: {
                title: "Unidad 1.2: Impedancia Compleja y Resonancia RLC",
                xpReward: 150,
                unlocks: ['acoplamiento'],
                content: `
                    <p>La **Impedancia ($Z$)** es la oposición total al flujo de corriente alterna en un circuito pasivo RLC. Es un número complejo compuesto por una parte real (Resistencia $R$) y una parte imaginaria (Reactancia $X$).</p>

                    <div class="math-derivation-box">
                        <div class="math-step">
                            <div class="math-step-title">Paso 1: Ley de Ohm Básica en AC</div>
                            <div class="math-equation">$$\mathbf{V} = \mathbf{I} \cdot \mathbf{Z} \implies \mathbf{Z} = \frac{\mathbf{V}}{\mathbf{I}}$$</div>
                        </div>

                        <div class="math-step">
                            <div class="math-step-title">Paso 2: Comportamiento de cada elemento pasivo</div>
                            <p>• Resistor: $Z_R = R$ (Sin desfase)<br>
                               • Inductancia: $Z_L = j\omega L$ (La corriente se atrasa $90^\circ$)<br>
                               • Capacitancia: $Z_C = \frac{1}{j\omega C} = -j \frac{1}{\omega C}$ (La corriente se adelanta $90^\circ$)</p>
                        </div>

                        <div class="math-step">
                            <div class="math-step-title">Paso 3: Impedancia Total RLC Serie</div>
                            <div class="math-equation">$$\mathbf{Z}_{total} = R + j\left(\omega L - \frac{1}{\omega C}\right) = R + j(X_L - X_C)$$</div>
                        </div>

                        <div class="math-step">
                            <div class="math-step-title">Paso 4: Deducción de la Frecuencia de Resonancia ($f_0$)</div>
                            <p>En resonancia, las reactancias se cancelan ($X_L = X_C$), haciendo que la impedancia sea puramente resistiva y mínima:</p>
                            <div class="math-equation">$$\omega_0 L = \frac{1}{\omega_0 C} \implies \omega_0^2 = \frac{1}{LC} \implies f_0 = \frac{1}{2\pi \sqrt{LC}}$$</div>
                        </div>
                    </div>

                    <h3>Simulador: Curva de Impedancia vs Frecuencia</h3>
                    <div class="sim-container">
                        <canvas id="sim-canvas" width="600" height="220"></canvas>
                        <div class="controls-grid">
                            <div class="control-group">
                                <label>Inductancia $L$ (mH): <span id="val-l">10</span></label>
                                <input type="range" id="input-l" min="1" max="50" value="10" oninput="updateSimRLC()">
                            </div>
                            <div class="control-group">
                                <label>Capacitancia $C$ ($\mu$F): <span id="val-c">10</span></label>
                                <input type="range" id="input-c" min="1" max="50" value="10" oninput="updateSimRLC()">
                            </div>
                        </div>
                    </div>
                `
            },
            potencia: {
                title: "Unidad 2: Potencia Compleja y Factor de Potencia",
                xpReward: 150,
                unlocks: ['trifasicos'],
                content: `
                    <p>En AC, la potencia no es solo el producto de Tensión y Corriente. Debido al desfase introducido por cargas inductivas o capacitivas, la potencia se divide en **Activa** (trabajo útil) y **Reactiva** (energía oscilante en campos magnéticos/eléctricos).</p>

                    <div class="math-derivation-box">
                        <div class="math-step">
                            <div class="math-step-title">Paso 1: Ecuación de Potencia Compleja ($\mathbf{S}$)</div>
                            <div class="math-equation">$$\mathbf{S} = \mathbf{V} \cdot \mathbf{I}^* = P + jQ$$</div>
                            <small>Donde $\mathbf{I}^*$ es el conjugado complejo de la corriente.</small>
                        </div>

                        <div class="math-step">
                            <div class="math-step-title">Paso 2: Componentes del Triángulo de Potencia</div>
                            <p>• **Potencia Activa ($P$):** $P = S \cdot \cos(\theta)$ [Watts, W]<br>
                               • **Potencia Reactiva ($Q$):** $Q = S \cdot \sin(\theta)$ [VAR]<br>
                               • **Potencia Aparente ($S$):** $S = |\mathbf{S}| = \sqrt{P^2 + Q^2}$ [VA]</p>
                        </div>

                        <div class="math-step">
                            <div class="math-step-title">Paso 3: Factor de Potencia ($FP$) y Corrección</div>
                            <div class="math-equation">$$FP = \cos(\theta) = \frac{P}{S}$$</div>
                            <p>Un $FP < 0.95$ acarrea penalizaciones industriales. Para corregirlo, añadimos un banco de condensadores con potencia reactiva $Q_C$:</p>
                            <div class="math-equation">$$Q_C = P \cdot \left(\tan(\theta_1) - \tan(\theta_2)\right) \implies C = \frac{Q_C}{\omega V_{rms}^2}$$</div>
                        </div>
                    </div>

                    <h3>Simulador: Triángulo de Potencias e Inyección Capacitiva</h3>
                    <div class="sim-container">
                        <canvas id="sim-canvas" width="600" height="220"></canvas>
                        <div class="controls-grid">
                            <div class="control-group">
                                <label>Carga Inductiva $Q_L$ (VAR): <span id="val-ql">300</span></label>
                                <input type="range" id="input-ql" min="50" max="500" value="300" oninput="updateSimPotencia()">
                            </div>
                            <div class="control-group">
                                <label>Compensación Capacitiva $Q_C$: <span id="val-qc">100</span></label>
                                <input type="range" id="input-qc" min="0" max="500" value="100" oninput="updateSimPotencia()">
                            </div>
                        </div>
                    </div>
                `
            },
            acoplamiento: {
                title: "Unidad 3: Acoplamiento Magnético y Transformadores",
                xpReward: 200,
                unlocks: [],
                content: `
                    <p>Cuando dos bobinas están próximas, el flujo magnético generado por una induce una tensión en la otra. Este fenómeno se rige por la **Inductancia Mutua ($M$)** y es la base de los transformadores.</p>

                    <div class="math-derivation-box">
                        <div class="math-step">
                            <div class="math-step-title">Paso 1: Ley de Faraday e Inductancia Mutua</div>
                            <div class="math-equation">$$v_1(t) = L_1 \frac{di_1}{dt} \pm M \frac{di_2}{dt}$$</div>
                            <div class="math-equation">$$v_2(t) = L_2 \frac{di_2}{dt} \pm M \frac{di_1}{dt}$$</div>
                            <small>El valor máximo de inductancia mutua es $M = k \sqrt{L_1 L_2}$, donde $k$ es el coeficiente de acoplamiento ($0 \le k \le 1$).</small>
                        </div>

                        <div class="math-step">
                            <div class="math-step-title">Paso 2: Relación de Transformación Ideal</div>
                            <p>En un transformador ideal sin pérdidas ($k = 1$):</p>
                            <div class="math-equation">$$a = \frac{N_1}{N_2} = \frac{\mathbf{V}_1}{\mathbf{V}_2} = \frac{\mathbf{I}_2}{\mathbf{I}_1}$$</div>
                        </div>
                    </div>

                    <h3>Simulador: Transformador de Aislamiento y Muestreo de Tensión</h3>
                    <div class="sim-container">
                        <canvas id="sim-canvas" width="600" height="200"></canvas>
                        <div class="controls-grid">
                            <div class="control-group">
                                <label>Vueltas Primario ($N_1$): <span id="val-n1">500</span></label>
                                <input type="range" id="input-n1" min="100" max="1000" value="500" oninput="updateSimTransfo()">
                            </div>
                            <div class="control-group">
                                <label>Vueltas Secundario ($N_2$): <span id="val-n2">250</span></label>
                                <input type="range" id="input-n2" min="50" max="1000" value="250" oninput="updateSimTransfo()">
                            </div>
                        </div>
                    </div>
                `
            },
            trifasicos: {
                title: "Unidad 4: Sistemas Trifásicos (Estrella Y y Delta Δ)",
                xpReward: 200,
                unlocks: [],
                content: `
                    <p>La generación y distribución industrial de energía eléctrica se realiza mediante sistemas trifásicos equilibrados con tres corrientes alternas desfasadas $120^\circ$ entre sí.</p>

                    <div class="math-derivation-box">
                        <div class="math-step">
                            <div class="math-step-title">Paso 1: Fasores de Fase Equilibrados</div>
                            <div class="math-equation">$$\mathbf{V}_{aN} = V_p \angle 0^\circ, \quad \mathbf{V}_{bN} = V_p \angle -120^\circ, \quad \mathbf{V}_{cN} = V_p \angle 120^\circ$$</div>
                        </div>

                        <div class="math-step">
                            <div class="math-step-title">Paso 2: Relación en Conexión Estrella (Y)</div>
                            <p>La tensión entre dos líneas ($\mathbf{V}_{ab} = \mathbf{V}_{aN} - \mathbf{V}_{bN}$) guarda una relación de $\sqrt{3}$ con la tensión de fase:</p>
                            <div class="math-equation">$$V_{linea} = \sqrt{3} \cdot V_{fase} \approx 1.732 \cdot V_{fase}$$</div>
                        </div>

                        <div class="math-step">
                            <div class="math-step-title">Paso 3: Potencia Trifásica Total</div>
                            <div class="math-equation">$$P_{total} = \sqrt{3} \cdot V_L \cdot I_L \cdot \cos(\theta)$$</div>
                        </div>
                    </div>

                    <h3>Simulador: Diagrama Fasorial Trifásico</h3>
                    <div class="sim-container">
                        <canvas id="sim-canvas" width="600" height="240"></canvas>
                        <div class="controls-grid">
                            <div class="control-group">
                                <label>Tensión de Fase $V_p$ (V): <span id="val-vp">120</span></label>
                                <input type="range" id="input-vp" min="50" max="240" value="120" oninput="updateSimTrifasico()">
                            </div>
                        </div>
                    </div>
                `
            }
        };

        /* --- MANEJO DEL SISTEMA Y EVENTOS --- */
        let currentActiveQuest = null;

        function updateHUD() {
            document.getElementById('player-lvl').innerText = `Nivel ${gameState.level}`;
            document.getElementById('player-rank').innerText = ranks[Math.min(gameState.level - 1, ranks.length - 1)];
            
            const xpMax = gameState.level * 200;
            const pct = Math.min((gameState.xp / xpMax) * 100, 100);
            document.getElementById('xp-bar').style.width = `${pct}%`;

            // Actualizar estado visual de los nodos
            Object.keys(questData).forEach(key => {
                const node = document.getElementById(`node-${key}`);
                if (!node) return;

                if (gameState.completedNodes.includes(key)) {
                    node.className = "skill-node completed";
                    node.querySelector('.node-status').innerText = "COMPLETADO";
                } else if (gameState.unlockedNodes.includes(key)) {
                    node.className = "skill-node unlocked";
                    node.querySelector('.node-status').innerText = "DISPONIBLE";
                } else {
                    node.className = "skill-node locked";
                    node.querySelector('.node-status').innerText = "BLOQUEADO";
                }
            });

            drawConnections();
        }

        function openQuest(key) {
            if (!gameState.unlockedNodes.includes(key)) return;

            currentActiveQuest = key;
            const data = questData[key];
            
            const modal = document.getElementById('quest-modal');
            document.getElementById('modal-title').innerText = data.title;
            
            const isCompleted = gameState.completedNodes.includes(key);
            const btnText = isCompleted ? "Misión Ya Completada" : `Completar Misión (+${data.xpReward} XP)`;
            
            document.getElementById('modal-body').innerHTML = `
                ${data.content}
                <button class="btn-action" onclick="completeQuest('${key}')">${btnText}</button>
            `;

            modal.classList.add('active');

            // Renderizar matemática limpia con KaTeX
            renderMathInElement(document.getElementById('modal-body'), {
                delimiters: [
                    {left: '$$', right: '$$', display: true},
                    {left: '$', right: '$', display: false}
                ],
                throwOnError : false
            });

            // Inicializar simuladores
            setTimeout(() => {
                if (key === 'senoidal') updateSimSenoidal();
                if (key === 'impedancia') updateSimRLC();
                if (key === 'potencia') updateSimPotencia();
                if (key === 'acoplamiento') updateSimTransfo();
                if (key === 'trifasicos') updateSimTrifasico();
            }, 50);
        }

        function closeQuest() {
            document.getElementById('quest-modal').classList.remove('active');
            currentActiveQuest = null;
        }

        function completeQuest(key) {
            if (!gameState.completedNodes.includes(key)) {
                gameState.completedNodes.push(key);
                const data = questData[key];
                
                // Desbloquear siguientes nodos
                data.unlocks.forEach(u => {
                    if (!gameState.unlockedNodes.includes(u)) {
                        gameState.unlockedNodes.push(u);
                    }
                });

                // Sumar Experiencia y calcular Nivel
                gameState.xp += data.xpReward;
                if (gameState.xp >= gameState.level * 200) {
                    gameState.level++;
                }

                updateHUD();
            }
            closeQuest();
        }

        /* --- DIBUJO DE LÍNEAS ENTRE NODOS (CANVAS BACKGROUND) --- */
        function drawConnections() {
            const canvas = document.getElementById('connections-canvas');
            const ctx = canvas.getContext('2d');
            
            canvas.width = canvas.offsetWidth;
            canvas.height = canvas.offsetHeight;
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            const connections = [
                { from: 'senoidal', to: 'impedancia' },
                { from: 'senoidal', to: 'potencia' },
                { from: 'impedancia', to: 'acoplamiento' },
                { from: 'potencia', to: 'trifasicos' }
            ];

            connections.forEach(conn => {
                const elFrom = document.getElementById(`node-${conn.from}`);
                const elTo = document.getElementById(`node-${conn.to}`);
                if (!elFrom || !elTo) return;

                const rectA = elFrom.getBoundingClientRect();
                const rectB = elTo.getBoundingClientRect();
                const containerRect = document.getElementById('tree-container').getBoundingClientRect();

                const x1 = rectA.left + rectA.width/2 - containerRect.left;
                const y1 = rectA.top + rectA.height/2 - containerRect.top;
                const x2 = rectB.left + rectB.width/2 - containerRect.left;
                const y2 = rectB.top + rectB.height/2 - containerRect.top;

                const isUnlocked = gameState.unlockedNodes.includes(conn.to);

                ctx.beginPath();
                ctx.moveTo(x1, y1);
                ctx.lineTo(x2, y2);
                ctx.strokeStyle = isUnlocked ? '#00f3ff' : 'rgba(255, 255, 255, 0.1)';
                ctx.lineWidth = isUnlocked ? 3 : 1;
                if (isUnlocked) {
                    ctx.shadowColor = '#00f3ff';
                    ctx.shadowBlur = 10;
                } else {
                    ctx.shadowBlur = 0;
                }
                ctx.stroke();
            });
        }

        /* --- LÓGICA DE SIMULADORES GRÁFICOS (CANVAS) --- */

        // 1. SIMULADOR ONDA SENOIDAL
        function updateSimSenoidal() {
            const Vm = parseFloat(document.getElementById('input-vm').value);
            const phiDeg = parseFloat(document.getElementById('input-phi').value);
            document.getElementById('val-vm').innerText = Vm;
            document.getElementById('val-phi').innerText = phiDeg;

            const canvas = document.getElementById('sim-canvas');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Ejes del Osciloscopio
            ctx.strokeStyle = 'rgba(255, 255, 255, 0.1)';
            ctx.beginPath();
            ctx.moveTo(0, 110); ctx.lineTo(600, 110);
            ctx.moveTo(150, 0); ctx.lineTo(150, 220);
            ctx.stroke();

            // Dibujar Fasor (Izquierda)
            const phiRad = (phiDeg * Math.PI) / 180;
            const r = Vm * 0.6;
            const fx = 150 + r * Math.cos(-phiRad);
            const fy = 110 + r * Math.sin(-phiRad);

            ctx.strokeStyle = '#ff0055';
            ctx.lineWidth = 3;
            ctx.beginPath();
            ctx.moveTo(150, 110);
            ctx.lineTo(fx, fy);
            ctx.stroke();

            // Dibujar Onda Senoidal (Derecha)
            ctx.strokeStyle = '#00f3ff';
            ctx.beginPath();
            for (let x = 150; x < 600; x++) {
                const t = (x - 150) * 0.02;
                const y = 110 - (Vm * 0.6) * Math.sin(t + phiRad);
                if (x === 150) ctx.moveTo(x, y);
                else ctx.lineTo(x, y);
            }
            ctx.stroke();
        }

        // 2. SIMULADOR RESONANCIA RLC
        function updateSimRLC() {
            const L = parseFloat(document.getElementById('input-l').value) * 1e-3;
            const C = parseFloat(document.getElementById('input-c').value) * 1e-6;
            document.getElementById('val-l').innerText = document.getElementById('input-l').value;
            document.getElementById('val-c').innerText = document.getElementById('input-c').value;

            const canvas = document.getElementById('sim-canvas');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            const f0 = 1 / (2 * Math.PI * Math.sqrt(L * C));

            // Dibujar Curva de Impedancia |Z| vs f
            ctx.strokeStyle = '#00ff66';
            ctx.lineWidth = 2;
            ctx.beginPath();
            for (let x = 0; x < 600; x++) {
                const f = x * 5; // Rango de 0 a 3000 Hz
                const w = 2 * Math.PI * f;
                const Xl = w * L;
                const Xc = w > 0 ? 1 / (w * C) : 1000;
                const Z = Math.sqrt(10*10 + Math.pow(Xl - Xc, 2));

                const y = 200 - Math.min(Z * 1.5, 180);
                if (x === 0) ctx.moveTo(x, y);
                else ctx.lineTo(x, y);
            }
            ctx.stroke();

            // Marca de Frecuencia de Resonancia
            const xRes = f0 / 5;
            ctx.strokeStyle = '#ffb700';
            ctx.setLineDash([5, 5]);
            ctx.beginPath();
            ctx.moveTo(xRes, 0); ctx.lineTo(xRes, 220);
            ctx.stroke();
            ctx.setLineDash([]);

            ctx.fillStyle = '#ffb700';
            ctx.font = '12px Orbitron';
            ctx.fillText(`Resonancia f0 = ${Math.round(f0)} Hz`, Math.min(xRes + 10, 420), 30);
        }

        // 3. SIMULADOR POTENCIA
        function updateSimPotencia() {
            const QL = parseFloat(document.getElementById('input-ql').value);
            const QC = parseFloat(document.getElementById('input-qc').value);
            document.getElementById('val-ql').innerText = QL;
            document.getElementById('val-qc').innerText = QC;

            const canvas = document.getElementById('sim-canvas');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            const P = 300; // W constante
            const Qnet = QL - QC;
            const S = Math.sqrt(P*P + Qnet*Qnet);
            const FP = (P / S).toFixed(2);

            // Triángulo de Potencia
            const ox = 100, oy = 180;
            const scale = 0.35;

            // Cateto P (Horizontal)
            ctx.strokeStyle = '#00f3ff'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P * scale, oy); ctx.stroke();

            // Cateto Q (Vertical)
            ctx.strokeStyle = '#ff0055';
            ctx.beginPath(); ctx.moveTo(ox + P * scale, oy); ctx.lineTo(ox + P * scale, oy - Qnet * scale); ctx.stroke();

            // Hipotenusa S
            ctx.strokeStyle = '#00ff66';
            ctx.beginPath(); ctx.moveTo(ox, oy); ctx.lineTo(ox + P * scale, oy - Qnet * scale); ctx.stroke();

            // Texto de métricas
            ctx.fillStyle = '#fff';
            ctx.font = '14px Orbitron';
            ctx.fillText(`Potencia Aparente S = ${Math.round(S)} VA`, 320, 60);
            ctx.fillText(`Factor de Potencia (FP) = ${FP}`, 320, 90);
        }

        // 4. SIMULADOR TRANSFORMADOR
        function updateSimTransfo() {
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

            // Núcleo del Transformador
            ctx.fillStyle = '#1e293b';
            ctx.fillRect(200, 40, 200, 120);
            ctx.clearRect(240, 70, 120, 60);

            // Bobinados
            ctx.strokeStyle = '#ffb700'; ctx.lineWidth = 4;
            ctx.beginPath(); ctx.arc(195, 100, 30, 0, Math.PI * 2); ctx.stroke();
            ctx.strokeStyle = '#00f3ff';
            ctx.beginPath(); ctx.arc(405, 100, 30, 0, Math.PI * 2); ctx.stroke();

            ctx.fillStyle = '#fff';
            ctx.font = '14px Orbitron';
            ctx.fillText(`V1 = ${V1} V`, 80, 105);
            ctx.fillText(`V2 = ${V2} V`, 460, 105);
        }

        // 5. SIMULADOR TRIFÁSICO
        function updateSimTrifasico() {
            const Vp = parseFloat(document.getElementById('input-vp').value);
            document.getElementById('val-vp').innerText = Vp;

            const canvas = document.getElementById('sim-canvas');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            const cx = 300, cy = 120;
            const r = Vp * 0.7;

            const angles = [0, -120, 120];
            const colors = ['#ff0055', '#00f3ff', '#00ff66'];

            angles.forEach((ang, i) => {
                const rad = (ang * Math.PI) / 180;
                const x = cx + r * Math.cos(rad);
                const y = cy + r * Math.sin(rad);

                ctx.strokeStyle = colors[i];
                ctx.lineWidth = 3;
                ctx.beginPath();
                ctx.moveTo(cx, cy);
                ctx.lineTo(x, y);
                ctx.stroke();
            });

            const VL = (Math.sqrt(3) * Vp).toFixed(1);
            ctx.fillStyle = '#fff';
            ctx.font = '14px Orbitron';
            ctx.fillText(`Tensión de Línea (V_ab) = ${VL} V`, 20, 30);
        }

        // Inicialización al cargar la ventana
        window.addEventListener('resize', drawConnections);
        window.onload = () => {
            updateHUD();
        };
    </script>
</body>
</html>
