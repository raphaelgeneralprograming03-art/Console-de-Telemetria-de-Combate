<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Console de Engenharia e Simulação Tática - Submarinos</title>
    <style>
        :root {
            --neon-green: #00ff66;
            --neon-blue: #00ffff;
            --alert-red: #ff3333;
            --panel-bg: #030805;
        }
        body {
            margin: 0; padding: 20px;
            background-color: #010402; color: var(--neon-green);
            font-family: 'Consolas', 'Courier New', monospace;
            display: flex; flex-direction: column; align-items: center;
        }
        #painel-global {
            display: grid; grid-template-columns: 800px 380px; gap: 20px; width: 1200px;
        }
        /* Painéis e Telas */
        .modulo-tela {
            border: 2px solid var(--neon-green); border-radius: 4px;
            background-color: var(--panel-bg); box-shadow: 0 0 15px rgba(0, 255, 102, 0.1);
            padding: 10px; box-sizing: border-box;
        }
        canvas {
            display: block; background-color: #000201; border: 1px solid #00441a;
            width: 100%; height: 450px;
        }
        /* Painel de Controle e Botões */
        .painel-botoes {
            display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-top: 15px;
        }
        button {
            background: #00220d; border: 1px solid var(--neon-green); color: var(--neon-green);
            padding: 12px; font-family: 'Consolas', monospace; font-weight: bold; cursor: pointer;
            border-radius: 3px; font-size: 0.85rem; text-transform: uppercase; transition: all 0.2s;
        }
        button:hover { background: var(--neon-green); color: #000; box-shadow: 0 0 10px var(--neon-green); }
        button:active { transform: scale(0.98); }
        .btn-alerta { border-color: var(--alert-red); color: var(--alert-red); background: #200000; }
        .btn-alerta:hover { background: var(--alert-red); color: #000; box-shadow: 0 0 10px var(--alert-red); }
        
        /* Monitor de Cálculos Matemáticos */
        .painel-calculos {
            display: flex; flex-direction: column; gap: 15px; font-size: 0.82rem; overflow-y: auto; height: 535px;
        }
        .bloco-formula {
            border-left: 3px solid var(--neon-blue); padding-left: 10px; margin-bottom: 5px;
            background: rgba(0, 242, 255, 0.03); padding: 8px; border-radius: 0 4px 4px 0;
        }
        .formula-matematica { color: #fff; font-style: italic; margin: 4px 0; font-size: 0.9rem;}
        .resultado-dinamico { color: var(--neon-blue); font-weight: bold; }
    </style>
</head>
<body>

    <h2 style="margin: 0 0 15px 0; letter-spacing: 2px; text-transform: uppercase; font-size: 1.3rem;">Sistemas Militares de Operações Subterrâneas e Superfície</h2>

    <div id="painel-global">
        
        <!-- COLUNA ESQUERDA: VISUALIZADOR GRÁFICO DA SIMULAÇÃO + BOTÕES -->
        <div>
            <div class="modulo-tela">
                <canvas id="arenaTactica" width="800" height="450"></canvas>
            </div>
            
            <!-- CONTROLES VIA BOTÕES CLICÁVEIS -->
            <div class="painel-botoes">
                <button onclick="alterarProfundidade(-1.5)">▲ Elevar Submarino</button>
                <button onclick="alternarMhd()">Alternar Propulsão MHD</button>
                <button onclick="dispararSams()">Disparar S.A.M.S.</button>
                <button onclick="alterarProfundidade(1.5)">▼ Submergir Submarino</button>
                <button onclick="alternarSros()">Alternar Mastro SROS</button>
                <button class="btn-alerta" onclick="dispararHipersonico()">Lançar Hipersônico</button>
            </div>
        </div>

        <!-- COLUNA DIREITA: MEMORIAL DE CÁLCULO SÍNCRONO EM TEMPO REAL -->
        <div class="modulo-tela painel-calculos">
            <h3 style="margin: 0; color: var(--neon-blue); border-bottom: 1px solid var(--neon-blue); padding-bottom: 5px;">MÓDULO MATEMÁTICO (SÍNCRONO)</h3>
            
            <!-- CÁLCULOS PROPULSÃO MHD -->
            <div class="bloco-formula">
                <strong>[MHD] EQUAÇÃO DA FORÇA DE LORENTZ</strong>
                <div class="formula-matematica">F_L = J × B • V</div>
                <div>Densidade Corrente (J): <span id="calc-mhd-j">450 A/m²</span></div>
                <div>Indução Magnética (B): <span id="calc-mhd-b">1.5 T</span></div>
                <div>Força Resultante Real: <span class="resultado-dinamico" id="calc-mhd-f">1012.5 N</span></div>
            </div>

            <!-- CÁLCULOS VETOR HIPERSÔNICO -->
            <div class="bloco-formula" style="border-left-color: var(--alert-red); background: rgba(255,51,51,0.03);">
                <strong>[HIPERSÔNICO] TERMODINÂMICA DE FLUXO</strong>
                <div class="formula-matematica">T_0 = T_∞ • (1 + ((γ - 1)/2) • M²)</div>
                <div>Regime de Velocidade: <span id="calc-m-mach">Mach 1.0</span></div>
                <div>Razão de Calores (γ): <span>1.40 (Ar Atmosférico)</span></div>
                <div>Temp. Estagnação Flutuante: <span class="resultado-dinamico" id="calc-m-temp">288.1 K</span></div>
            </div>

            <!-- CÁLCULOS DETECÇÃO ÓPTICA SROS -->
            <div class="bloco-formula">
                <strong>[SROS] RESOLUÇÃO DE SENSORES MULTIESPECTRAIS</strong>
                <div class="formula-matematica">θ = 1.22 • (λ / D)</div>
                <div>Comprimento de Onda (λ): <span>4.5 µm (Invermelho Médio)</span></div>
                <div>Abertura da Lente (D): <span>0.18 m</span></div>
                <div>Limite Difração Angular: <span class="resultado-dinamico" id="calc-sros-theta">0.0000305 rad</span></div>
            </div>

            <!-- CÁLCULOS TELEMETRIA GERAL -->
            <div class="bloco-formula" style="border-left-color: var(--neon-green); background: rgba(0,255,102,0.03);">
                <strong>[TELEMETRIA] HIDROSTÁTICA E COORDENADAS</strong>
                <div>Eixo Profundidade (Y-Raw): <span id="calc-geo-y">360px</span></div>
                <div>Pressão Hidrostática (P): <span class="resultado-dinamico" id="calc-geo-p">2.45 MPa</span></div>
            </div>
        </div>

    </div>

    <script>
        const canvas = document.getElementById("arenaTactica");
        const ctx = canvas.getContext("2d");

        // Definição física de fronteira (Meio Fluido a 200px do topo)
        const CAMADA_OCEANO = 200;

        // Configurações do Submarino de Combate
        const submarino = {
            x: 60, y: 340, largura: 95, altura: 20, velX: 0.8, tesla: 1.5, correnteJ: 450
        };

        // Estados e Vetores do Sistema
        let mhdAltaPotencia = false;
        let srosElevado = false;
        let vetorSams = [];
        let vetorHipersonico = [];

        // Elementos do Cenário Operacional
        const cacaInimigo = { x: -50, y: 50, vel: 2.5, operacional: true, abatido: false, angulo: 0 };
        const alvoContinente = { x: 710, y: CAMADA_OCEANO - 40, w: 90, h: 40, neutralizado: false };

        // --- FUNÇÕES DE CONTROLE DE BOTÕES ---
        function alterarProfundidade(valor) {
            submarino.y += valor * 10;
            // Limitações físicas do casco
            if (submarino.y < CAMADA_OCEANO + 5) submarino.y = CAMADA_OCEANO + 5;
            if (submarino.y > canvas.height - 30) submarino.y = canvas.height - 30;
        }

        function alternarMhd() {
            mhdAltaPotencia = !mhdAltaPotencia;
            if(mhdAltaPotencia) {
                submarino.velX = 3.2; submarino.tesla = 7.8; submarino.correnteJ = 1200;
            } else {
                submarino.velX = 0.8; submarino.tesla = 1.5; submarino.correnteJ = 450;
            }
        }

        function alternarSros() {
            if(submarino.y <= CAMADA_OCEANO + 30) {
                srosElevado = !srosElevado;
            }
        }

        function dispararSams() {
            if (submarino.y <= CAMADA_OCEANO + 45 && cacaInimigo.operacional && !cacaInimigo.abatido) {
                vetorSams.push({ x: submarino.x + 45, y: submarino.y, vx: 2.2, vy: -5.5 });
            }
        }

        function dispararHipersonico() {
            if (submarino.y <= CAMADA_OCEANO + 15) {
                vetorHipersonico.push({ x: submarino.x + 20, y: submarino.y, vx: 0.7, vy: -6, mach: 1, rastro: [] });
            }
        }

        // --- MOTOR DE PROCESSAMENTO E ATUALIZAÇÃO SÍNCRONA (60 FPS) ---
        function simularEComputar() {
            // 1. ATUALIZAÇÃO DA INTERFACE DE CÁLCULO REAL/SÍNCRONE
            // Cálculo da Força de Lorentz (MHD)
            let fLorentz = submarino.correnteJ * submarino.tesla * 1.5; // Vol fictício de 1.5m³
            document.getElementById("calc-mhd-j").innerText = `${submarino.correnteJ} A/m²`;
            document.getElementById("calc-mhd-b").innerText = `${submarino.tesla} T`;
            document.getElementById("calc-mhd-f").innerText = `${fLorentz.toFixed(1)} N`;

            // Cálculo da Pressão Hidrostática Baseada no Sensor de Profundidade
            let profMetros = (submarino.y - CAMADA_OCEANO) * 1.8;
            let pressaoMpa = (1000 * 9.81 * profMetros) / 1000000;
            document.getElementById("calc-geo-y").innerText = `${submarino.y} px`;
            document.getElementById("calc-geo-p").innerText = `${pressaoMpa.toFixed(2)} MPa`;

            // 2. RENDERIZAÇÃO DA IMAGEM E ELEMENTOS GRÁFICOS NO CANVAS
            // Limpa canvas e desenha a estratosfera/céu
            ctx.fillStyle = "#010502"; ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Renderiza o meio aquático (Oceano Profundo)
            ctx.fillStyle = "rgba(1, 15, 32, 0.7)"; ctx.fillRect(0, CAMADA_OCEANO, canvas.width, canvas.height - CAMADA_OCEANO);
            
            // Desenha a linha divisória da superfície marítima
