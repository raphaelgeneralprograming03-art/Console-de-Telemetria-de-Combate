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
        .modulo-tela {
            border: 2px solid var(--neon-green); border-radius: 4px;
            background-color: var(--panel-bg); box-shadow: 0 0 15px rgba(0, 255, 102, 0.1);
            padding: 10px; box-sizing: border-box;
        }
        
        /* NOVO PAINEL DE SIMULAÇÃO USANDO APENAS ELEMENTOS HTML/CSS REALISTAS */
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

        /* OBJETOS FÍSICOS DA SIMULAÇÃO */
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
            position: absolute; width: 90px; height: 40px; background-color: #0d1310;
            border: 1px solid var(--alert-red); right: 10px; top: 160px; text-align: center; font-size: 8px; color: var(--alert-red);
        }
        .missil-sams {
            position: absolute; width: 6px; height: 6px; background-color: #ffaa00; border-radius: 50%;
        }
        .missil-hiper {
            position: absolute; width: 8px; height: 3px; background-color: #ffffff; box-shadow: 0 0 8px #ff3333;
        }

        /* PAINEL DE BOTÕES */
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
        
        /* PAINEL MATEMÁTICO */
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
        
        <!-- VISUALIZADOR DA SIMULAÇÃO (HTML PURO) + BOTÕES -->
        <div>
            <div class="modulo-tela">
                <div id="arenaTactica">
                    <div id="camada-ceu">
                        <div id="caca"></div>
                        <div id="bunker"><br>BUNKER HQ</div>
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
                <button id="btn-elevar">▲ Elevar Submarino</button>
                <button id="btn-mhd">Alternar Propulsão MHD</button>
                <button id="btn-sams">Disparar S.A.M.S.</button>
                <button id="btn-submergir">▼ Submergir Submarino</button>
                <button id="btn-sros">Alternar Mastro SROS</button>
                <button class="btn-alerta" id="btn-hiper">Lançar Hipersônico</button>
            </div>
        </div>

        <!-- MEMORIAL DE CÁLCULO SÍNCRONO -->
        <div class="modulo-tela painel-calculos">
            <h3 style="margin: 0; color: var(--neon-blue); border-bottom: 1px solid var(--neon-blue); padding-bottom: 5px;">MÓDULO MATEMÁTICO (SÍNCRONO)</h3>
            
            <div class="bloco-formula">
                <strong>[MHD] EQUAÇÃO DA FORÇA DE LORENTZ</strong>
                <div class="formula-matematica">F_L = J × B • V</div>
                <div>Densidade Corrente (J): <span id="calc-mhd-j">450 A/m²</span></div>
                <div>Indução Magnética (B): <span id="calc-mhd-b">1.5 T</span></div>
                <div>Força Resultante Real: <span class="resultado-dinamico" id="calc-mhd-f">1012.5 N</span></div>
            </div>

            <div class="bloco-formula" style="border-left-color: var(--alert-red); background: rgba(255,51,51,0.03);">
                <strong>[HIPERSÔNICO] TERMODINÂMICA DE FLUXO</strong>
                <div class="formula-matematica">T_0 = T_∞ • (1 + ((γ - 1)/2) • M²)</div>
                <div>Regime de Velocidade: <span id="calc-m-mach">Mach 1.0</span></div>
                <div>Razão de Calores (γ): <span>1.40 (Ar Atmosférico)</span></div>
                <div>Temp. Estagnação Flutuante: <span class="resultado-dinamico" id="calc-m-temp">288.1 K</span></div>
            </div>

            <div class="bloco-formula">
                <strong>[SROS] RESOLUÇÃO DE SENSORES MULTIESPECTRAIS</strong>
                <div class="formula-matematica">θ = 1.22 • (λ / D)</div>
                <div>Comprimento de Onda (λ): <span>4.5 µm (Invermelho Médio)</span></div>
                <div>Abertura da Lente (D): <span>0.18 m</span></div>
                <div>Limite Difração Angular: <span class="resultado-dinamico" id="calc-sros-theta">0.0000305 rad</span></div>
            </div>

            <div class="bloco-formula" style="border-left-color: var(--neon-green); background: rgba(0,255,102,0.03);">
                <strong>[TELEMETRIA] HIDROSTÁTICA E COORDENADAS</strong>
                <div>Eixo Profundidade (Y-Raw): <span id="calc-geo-y">340px</span></div>
                <div>Pressão Hidrostática (P): <span class="resultado-dinamico" id="calc-geo-p">2.45 MPa</span></div>
            </div>
        </div>

    </div>

    <script>
        window.onload = function() {
            // Referências HTML estáveis dos objetos físicos
            const elSub = document.getElementById("submarino");
            const elRastro = document.getElementById("mhd-rastro");
            const elCaca = document.getElementById("caca");
            const elBunker = document.getElementById("bunker");
            const arena = document.getElementById("arenaTactica");

            // Estado Inicial das Variáveis Físicas e Geográficas
            let subX = 60;
            let subY = 340; // Coordenada de Profundidade inicial
            let velX = 0.8;
            let tesla = 1.5;
            let correnteJ = 450;
            let mhdAltaPotencia = false;
            let srosElevado = false;

            let cacaX = -40;
            let cacaY = 50;
            let cacaAbatido = false;

            // Arrays para controle dos mísseis na tela
            let listaSams = [];
            let listaHiper = [];

            // --- LÓGICA DE INTERAÇÃO DOS BOTÕES ---
            document.getElementById("btn-elevar").onclick = function() {
                subY -= 20;
                if (subY < 205) subY = 205; // Limite da superfície da água
                elSub.style.top = subY + "px";
            };

            document.getElementById("btn-submergir").onclick = function() {
                subY += 20;
                if (subY > 400) subY = 400; // Limite do fundo do mar
                elSub.style.top = subY + "px";
            };

            document.getElementById("btn-mhd").onclick = function() {
                mhdAltaPotencia = !mhdAltaPotencia;
                if(mhdAltaPotencia) {
                    velX = 3.2; tesla = 7.8; correnteJ = 1200;
                    elRastro.style.backgroundColor = "#00ffff";
                    elRastro.style.width = "40px";
                    elRastro.style.left = "-41px";
                    elSub.style.borderColor = "#ff3333";
                } else {
