<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Console de Telemetria e Simulação Tática - MHD / SAMS / SROS</title>
    <style>
        body {
            margin: 0; padding: 20px;
            background-color: #030806; color: #00ff66;
            font-family: 'Consolas', 'Courier New', monospace;
            display: flex; flex-direction: column; align-items: center;
        }
        #painel-superior {
            width: 950px; display: grid; grid-template-columns: repeat(3, 1fr);
            gap: 15px; margin-bottom: 15px;
        }
        .bloco-dados {
            background: rgba(0, 40, 20, 0.4);
            border: 1px solid #00aa44; border-radius: 4px; padding: 10px;
        }
        .titulo-bloco { font-weight: bold; border-bottom: 1px solid #00aa44; padding-bottom: 4px; margin-bottom: 8px; color: #00ffbc;}
        .valor-telemetria { color: #ffffff; }
        #canvas-container { border: 2px solid #00ff66; background-color: #010402; }
        .controles-simulacao { margin-top: 10px; width: 950px; text-align: left; font-size: 0.85rem; color: #88aa88;}
    </style>
</head>
<body>

    <!-- TELEMETRIA DE DADOS DOS SISTEMAS -->
    <div id="painel-superior">
        <div class="bloco-dados">
            <div class="titulo-bloco">PROPULSÃO MHD (MAGNÉTICA)</div>
            <div>STATUS: <span class="valor-telemetria" id="mhd-status">Furtivo (Baixo Ruído)</span></div>
            <div>CAMP. MAGNÉTICO: <span class="valor-telemetria" id="mhd-tesla">1.5 Tesla</span></div>
            <div>EFICIÊNCIA FLUIDO: <span class="valor-telemetria">32% (Lorentz)</span></div>
        </div>
        <div class="bloco-dados">
            <div class="titulo-bloco">DEFESA S.A.M.S. & HIPERSÔNICO</div>
            <div>SAMS PRONTO: <span class="valor-telemetria" id="sams-status">Aguardando Prof. Periscópio</span></div>
            <div>MÍSSIL HIPERSÔNICO: <span class="valor-telemetria" id="missil-status">Pronto (Tubo 01)</span></div>
            <div>ASSINATURA TÉRMICA: <span class="valor-telemetria" id="plasma-status">0 MW</span></div>
        </div>
        <div class="bloco-dados">
            <div class="titulo-bloco">DIAGNÓSTICO SROS (OPTRÔNICO)</div>
            <div>MAST STATUS: <span class="valor-telemetria" id="sros-status">Retraído (Submerso)</span></div>
            <div>ALVOS TRAVADOS: <span class="valor-telemetria" id="sros-targets">0/2</span></div>
            <div>FEIXE LASER: <span class="valor-telemetria">Multiespectral IR</span></div>
        </div>
    </div>

    <!-- ÁREA DA SIMULAÇÃO CIENTÍFICA -->
    <div id="canvas-container">
        <canvas id="arenaTactica" width="950" height="500"></canvas>
    </div>

    <div class="controles-simulacao">
        <strong>COMANDOS DO OPERADOR:</strong> W/S: Alterar Profundidade | 
        Teclado [A]: Alternar Potência MHD (Furtivo vs Alta Velocidade) | 
        Teclado [R]: Elevar/Retrair Mastro SROS | 
        Teclado [F]: Disparar S.A.M.S. (Contra Aeronave) | 
        Teclado [H]: Disparar Míssil Hipersônico (Contra a Infraestrutura)
    </div>

    <script>
        const canvas = document.getElementById("arenaTactica");
        const ctx = canvas.getContext("2d");

        // Elementos DOM para atualização de Telemetria Real
        const eMhdStatus = document.getElementById("mhd-status");
        const eMhdTesla = document.getElementById("mhd-tesla");
        const eSamsStatus = document.getElementById("sams-status");
        const eMissilStatus = document.getElementById("missil-status");
        const ePlasmaStatus = document.getElementById("plasma-status");
        const eSrosStatus = document.getElementById("sros-status");
        const eSrosTargets = document.getElementById("sros-targets");

        // Variáveis de Ambiente e Coordenadas Geográficas da Simulação
        const CAMADA_OCEANO = 220;
        let mhdAltaPotencia = false;
        let srosElevado = false;

        const inputs = { w: false, s: false };
        window.addEventListener("keydown", (e) => {
            if(e.key.toLowerCase() === 'w') inputs.w = true;
            if(e.key.toLowerCase() === 's') inputs.s = true;
            
            // Alternar Modo de Indução do Propulsor MHD
            if(e.key.toLowerCase() === 'a') {
                mhdAltaPotencia = !mhdAltaPotencia;
                if(mhdAltaPotencia) {
                    submarino.velocidadeX = 3.5;
                    eMhdStatus.innerText = "VELOCIDADE MÁXIMA (Alerta MAD)";
                    eMhdStatus.style.color = "#ff3333";
                    eMhdTesla.innerText = "7.8 Tesla";
                } else {
                    submarino.velocidadeX = 1.0;
                    eMhdStatus.innerText = "Furtivo (Baixo Ruído)";
                    eMhdStatus.style.color = "#00ff66";
                    eMhdTesla.innerText = "1.5 Tesla";
                }
            }

            // Alternar Elevação do Mastro Optrônico SROS
            if(e.key.toLowerCase() === 'r') {
                if(submarino.y <= CAMADA_OCEANO + 25) { // Só eleva próximo à superfície
                    srosElevado = !srosElevado;
                    eSrosStatus.innerText = srosElevado ? "ELEVADO - ESCANEANDO" : "RETRAÍDO";
                    eSrosStatus.style.color = srosElevado ? "#00ff66" : "#ffffff";
                }
            }

            // Ativação do Sistema S.A.M.S.
            if(e.key.toLowerCase() === 'f' && submarino.y <= CAMADA_OCEANO + 40) {
                if(cacaInimigo.operacional) {
                    vetorSams.push({ x: submarino.x + 40, y: submarino.y, vx: 2, vy: -5, ativo: true });
                    eSamsStatus.innerText = "LANÇADO / TRAVADO IR";
                    eSamsStatus.style.color = "#ffaa00";
                }
            }

            // Disparo de Vetor Estratégico Hipersônico
            if(e.key.toLowerCase() === 'h' && submarino.y <= CAMADA_OCEANO + 15) {
                vetorHipersonico.push({ x: submarino.x + 20, y: submarino.y, vx: 0.8, vy: -6, mach: 1, rastro: [] });
                eMissilStatus.innerText = "EM VOO - CRUISE PHASE";
                eMissilStatus.style.color = "#ff3333";
            }
        });

        window.addEventListener("keyup", (e) => {
            if(e.key.toLowerCase() === 'w') inputs.w = false;
            if(e.key.toLowerCase() === 's') inputs.s = false;
        });

        // Configurações Físicas do Submarino Operacional
        const submarino = {
            x: 70, y: 380, largura: 85, altura: 18, velocidadeX: 1.0, velocidadeY: 1.2
        };

        // Alvos de Teste dos Algoritmos de Combate
        const cacaInimigo = { x: -40, y: 60, vel: 2.2, operacional: true, abatido: false, anguloQueda: 0 };
        const qgInimigo = { x: 840, y: CAMADA_OCEANO - 50, largura: 110, altura: 50, destruido: false };

        // Vetores Dinâmicos
        let vetorSams = [];
        let vetorHipersonico = [];

        function renderizarAlgoritmo() {
            // Limpeza e Atualização Dinâmica da Atmosfera/Água
            ctx.fillStyle = "#020503"; ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Renderizar Meio Aquático (Densidade sonar)
            ctx.fillStyle = "rgba(0, 20, 35, 0.6)";
            ctx.fillRect(0, CAMADA_OCEANO, canvas.width, canvas.height - CAMADA_OCEANO);
            
            // Interface Físico-Química da Água (Superfície)
            ctx.strokeStyle = "rgba(0, 255, 180, 0.4)"; ctx.lineWidth = 1;
            ctx.beginPath(); ctx.moveTo(0, CAMADA_OCEANO); ctx.lineTo(canvas.width, CAMADA_OCEANO); ctx.stroke();

            // --- CÁLCULO DE MOVIMENTO DO SUBMARINO ---
            if (inputs.w && submarino.y > CAMADA_OCEANO + 8) submarino.y -= submarino.velocidadeY;
            if (inputs.s && submarino.y < canvas.height - 30) submarino.y += submarino.velocidadeY;
            submarino.x += submarino.velocidadeX;
            if (submarino.x > canvas.width - 200) submarino.x = 40; // Mantém em órbita de teste espacial

            // --- MODELAGEM DOS SISTEMAS ---
            
            // 1. PROPULSÃO MHD: Rastro Térmico-Elétrico Sem Cavitação Mecânica
            if (mhdAltaPotencia) {
                // Em alta potência, o empuxo Lorentz gera ionização detectável na água
                ctx.fillStyle = "rgba(0, 255, 255, 0.2)";
                ctx.fillRect(submarino.x - 40, submarino.y + 4, 40, 8);
                // Desenhar linhas de indução do campo magnético
                ctx.strokeStyle = "rgba(255, 50, 50, 0.4)";
                ctx.strokeRect(submarino.x, submarino.y - 5, submarino.largura, submarino.altura + 10);
            } else {
                // Em baixo ruído, o rastro magnético é contido
                ctx.fillStyle = "rgba(0, 255, 100, 0.08)";
                ctx.fillRect(submarino.x - 20, submarino.y + 6, 20, 4);
            }

            // Desenhar Estrutura do Submarino
            ctx.fillStyle = "#1e2522"; ctx.strokeStyle = "#00ff66"; ctx.lineWidth = 1.5;
            ctx.beginPath(); ctx.roundRect(submarino.x, submarino.y, submarino.largura, submarino.altura, 6);
            ctx.fill(); ctx.stroke();
            ctx.fillRect(submarino.x + 35, submarino.y - 10, 16, 10); // Vela do Casco

            // 2. SISTEMA SROS: Controle do Mastro Optrônico Multi-Espectral
            if (submarino.y > CAMADA_OCEANO + 25) srosElevado = false; // Retração forçada pela pressão hidrodinâmica
            
            if (srosElevado) {
                ctx.strokeStyle = "#00ffff"; ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.moveTo(submarino.x + 43, submarino.y - 10);
                ctx.lineTo(submarino.x + 43, CAMADA_OCEANO - 12); // Projeção acima da água
                ctx.stroke();
                
                // Sensor Optrônico SROS emitindo varredura passiva infravermelha
