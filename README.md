<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Console de Engenharia e Simulação Avançada - Submarinos</title>
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
            display: grid; grid-template-columns: 800px 420px; gap: 20px; width: 1240px;
        }
        .modulo-tela {
            border: 2px solid var(--neon-green); border-radius: 4px;
            background-color: var(--panel-bg); box-shadow: 0 0 15px rgba(0, 255, 102, 0.1);
            padding: 10px; box-sizing: border-box;
        }
        
        #arenaTactica {
            width: 780px; height: 430px; background-color: #020503;
            position: relative; overflow: hidden; border: 1px solid #00441a;
        }
        #camada-ceu {
            position: absolute; top: 0; left: 0; width: 100%; height: 200px; background-color: #020604;
        }
        #camada-mar {
            position: absolute; top: 200px; left: 0; width: 100%; height: 230px; background-color: #0c1424;
            border-top: 2px solid var(--neon-green);
        }

        /* OBJETOS FÍSICOS */
        #submarino {
            position: absolute; width: 95px; height: 20px; background-color: #1b2621;
            border: 1px solid var(--neon-green); left: 60px; top: 340px; transition: top 0.2s linear;
        }
        #submarino-vela {
            position: absolute; width: 18px; height: 8px; background-color: #1b2621;
            border: 1px solid var(--neon-green); border-bottom: none; left: 40px; top: -9px;
        }
        #mhd-rastro {
            position: absolute; width: 25px; height: 6px; background-color: #006633; left: -26px; top: 7px;
        }
        #caca {
            position: absolute; width: 25px; height: 12px; background-color: #332222;
            border: 1px solid var(--alert-red); left: -40px; top: 50px;
        }
        #bunker {
            position: absolute; width: 90px; height: 45px; background-color: #0d1310;
            border: 1px solid var(--alert-red); right: 10px; top: 155px; text-align: center; font-size: 8px; color: var(--alert-red);
        }
        .missil-sams { position: absolute; width: 6px; height: 6px; background-color: #ffaa00; border-radius: 50%; }
        .missil-hiper { position: absolute; width: 8px; height: 3px; background-color: #ffffff; box-shadow: 0 0 8px #ff3333; }
        .missil-defesa { position: absolute; width: 5px; height: 5px; background-color: #ff00ff; border-radius: 50%; }

        /* CONTROLES */
        .painel-botoes {
            display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; margin-top: 15px;
        }
        button {
            background: #00220d; border: 1px solid var(--neon-green); color: var(--neon-green);
            padding: 12px; font-family: 'Consolas', monospace; font-weight: bold; cursor: pointer;
            border-radius: 3px; font-size: 0.8rem; text-transform: uppercase; transition: all 0.2s;
        }
        button:hover { background: var(--neon-green); color: #000; box-shadow: 0 0 10px var(--neon-green); }
        .btn-alerta { border-color: var(--alert-red); color: var(--alert-red); background: #200000; }
        .btn-alerta:hover { background: var(--alert-red); color: #000; box-shadow: 0 0 10px var(--alert-red); }
        .btn-info { border-color: var(--neon-blue); color: var(--neon-blue); background: #00222b; }
        .btn-info:hover { background: var(--neon-blue); color: #000; box-shadow: 0 0 10px var(--neon-blue); }
        
        /* EXIBIÇÃO DE CÁLCULOS */
        .painel-calculos {
            display: flex; flex-direction: column; gap: 12px; font-size: 0.8rem; overflow-y: auto; height: 535px;
        }
        .bloco-formula {
            border-left: 3px solid var(--neon-blue); padding-left: 10px;
            background: rgba(0, 242, 255, 0.03); padding: 8px; border-radius: 0 4px 4px 0;
        }
        .formula-matematica { color: #fff; font-style: italic; margin: 4px 0; font-size: 0.85rem;}
        .resultado-dinamico { color: var(--neon-blue); font-weight: bold; }
    </style>
</head>
<body>

    <h2 style="margin: 0 0 15px 0; letter-spacing: 2px; text-transform: uppercase; font-size: 1.3rem;">Sistemas Militares de Operações Subterrâneas e Superfície - V2.0</h2>

    <div id="painel-global">
        
        <!-- COLUNA ESQUERDA: ARENA VISUAL + BOTOES -->
        <div>
            <div class="modulo-tela">
                <div id="arenaTactica">
                    <div id="camada-ceu">
                        <div id="caca"></div>
                        <div id="bunker"><br>BUNKER HQ<br><span id="bunker-status" style="color:#00ff66;">DEFESA: OK</span></div>
                    </div>
                    <div id="camada-mar">
                        <div id="submarino">
                            <div id="submarino-vela"></div>
                            <div id="mhd-rastro"></div>
                        </div>
                    </div>
                </div>
            </div>
            
            <div class="painel-botoes">
                <button id="btn-elevar">▲ Elevar Casco</button>
                <button id="btn-mhd">Alternar MHD</button>
                <button id="btn-sams">Disparar S.A.M.S.</button>
                <button class="btn-alerta" id="btn-hiper">Lançar Hipersônico</button>
                <button id="btn-submergir">▼ Submergir Casco</button>
                <button id="btn-sros">Mastro SROS</button>
                <button class="btn-info" id="btn-report">Exportar Telemetria</button>
            </div>
        </div>

        <!-- COLUNA DIREITA: MEMORIAL DE CÁLCULO CO-SÍNCRONO -->
        <div class="modulo-tela painel-calculos">
            <h3 style="margin: 0; color: var(--neon-blue); border-bottom: 1px solid var(--neon-blue); padding-bottom: 5px;">MÓDULO MATEMÁTICO AVANÇADO</h3>
            
            <!-- SONAR EQUATION -->
            <div class="bloco-formula" style="border-left-color: #ffaa00; background: rgba(255,170,0,0.02);">
                <strong>[SONAR] EQUAÇÃO ACÚSTICA PASSIVA</strong>
                <div class="formula-matematica">SNR = SL - TL - (NL - DI)</div>
                <div>Ruído de Auto-Indução (NL): <span id="calc-sonar-nl">45 dB</span></div>
                <div>Perda por Propagação (TL): <span id="calc-sonar-tl">32 dB</span></div>
                <div>Razão Sinal-Ruído Real: <span class="resultado-dinamico" id="calc-sonar-snr" style="color:#ffaa00;">+28 dB (Ótimo)</span></div>
            </div>

            <!-- FORÇA DE LORENTZ -->
            <div class="bloco-formula">
                <strong>[MHD] EQUAÇÃO DA FORÇA DE LORENTZ</strong>
                <div class="formula-matematica">F_L = J × B • V</div>
                <div>Densidade Corrente (J): <span id="calc-mhd-j">450 A/m²</span></div>
                <div>Indução Magnética (B): <span id="calc-mhd-b">1.5 T</span></div>
                <div>Força Resultante Real: <span class="resultado-dinamico" id="calc-mhd-f">1012.5 N</span></div>
            </div>

            <!-- TERMODINÂMICA HIPERSÔNICA -->
            <div class="bloco-formula" style="border-left-color: var(--alert-red); background: rgba(255,51,51,0.02);">
                <strong>[HIPERSÔNICO] FLUXO E DEFESA DE TERRA</strong>
                <div class="formula-matematica">T_0 = T_∞ • (1 + 0.2 • M²)</div>
                <div>Velocidade Escoamento: <span id="calc-m-mach">Mach 1.0</span></div>
                <div>Temp. Parede Cinética: <span class="resultado-dinamico" id="calc-m-temp">288.1 K</span></div>
                <div>Alerta Contramedida: <span id="calc-defesa-status" style="color:#00ff66;">Sem Ameaças</span></div>
            </div>

            <!-- TELEMETRIA GERAL -->
            <div class="bloco-formula" style="border-left-color: var(--neon-green); background: rgba(0,255,102,0.02);">
                <strong>[TELEMETRIA] HIDROSTÁTICA E COORDENADAS</strong>
                <div>Eixo Coordenada Y-Raw: <span id="calc-geo-y">340px</span></div>
                <div>Pressão Hidrostática (P): <span class="resultado-dinamico" id="calc-geo-p">2.45 MPa</span></div>
            </div>
        </div>

    </div>

    <script>
        window.onload = function() {
            const elSub = document.getElementById("submarino");
            const elRastro = document.getElementById("mhd-rastro");
            const elCaca = document.getElementById("caca");
            const elBunker = document.getElementById("bunker");
            const elBunkerStatus = document.getElementById("bunker-status");
            const arena = document.getElementById("arenaTactica");

            // Telemetria Física Co-síncrona
            let subX = 60; let subY = 340; let velX = 0.8;
            let tesla = 1.5; let correnteJ = 450;
            let mhdAltaPotencia = false; let srosElevado = false;

            let cacaX = -40; let cacaY = 50; let cacaAbatido = false;

            let listaSams = []; let listaHiper = []; let listaDefesaInimiga = [];

            // Banco de dados dinâmico para exportação do relatório
            let históricoMhd = []; let históricoMach = [];

            // --- ACIONADORES DE SISTEMAS ---
            document.getElementById("btn-elevar").onclick = function() {
                subY -= 20; if (subY < 205) subY = 205;
                elSub.style.top = subY + "px";
            };

            document.getElementById("btn-submergir").onclick = function() {
                subY += 20; if (subY > 400) subY = 400;
                elSub.style.top = subY + "px";
            };

