
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
            z-index: 5;
        }
        #submarino-vela {
            position: absolute; width: 18px; height: 8px; background-color: #1b2621;
            border: 1px solid var(--neon-green); border-bottom: none; left: 40px; top: -9px;
        }
        #submarino-sros {
            position: absolute; width: 2px; height: 14px; background-color: var(--neon-blue);
            left: 48px; top: -23px; display: none; box-shadow: 0 0 5px var(--neon-blue);
        }
        #mhd-rastro {
            position: absolute; width: 25px; height: 6px; background-color: #006633; left: -26px; top: 7px;
            transition: all 0.3s;
        }
        #caca {
            position: absolute; width: 25px; height: 12px; background-color: #332222;
            border: 1px solid var(--alert-red); left: -40px; top: 50px; z-index: 4;
        }
        #bunker {
            position: absolute; width: 90px; height: 45px; background-color: #0d1310;
            border: 1px solid var(--alert-red); right: 10px; top: 155px; text-align: center; font-size: 8px; color: var(--alert-red);
            z-index: 4;
        }
        .missil-sams { position: absolute; width: 6px; height: 6px; background-color: #ffaa00; border-radius: 50%; z-index: 6; box-shadow: 0 0 6px #ffaa00; }
        .missil-hiper { position: absolute; width: 10px; height: 3px; background-color: #ffffff; box-shadow: 0 0 8px #ff3333; z-index: 6; }
        .missil-defesa { position: absolute; width: 5px; height: 5px; background-color: #ff00ff; border-radius: 50%; z-index: 6; box-shadow: 0 0 5px #ff00ff; }
        .explosao {
            position: absolute; width: 20px; height: 20px; border-radius: 50%;
            background-color: #ffaa00; border: 2px solid #ff3333;
            animation: animExplosao 0.4s forwards; z-index: 10;
        }

        @keyframes animExplosao {
            0% { transform: scale(0.2); opacity: 1; }
            100% { transform: scale(2.5); opacity: 0; }
        }

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
        button.ativo { background: var(--neon-green); color: #000; }
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
                            <div id="submarino-sros"></div>
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
                <div>Razão Sinal-Ruído Real: <span class="resultado-dinamico" id="calc-sonar-snr" style="color:#ffaa00;">+58 dB (Ótimo)</span></div>
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
                <div>Velocidade Escoamento: <span id="calc-m-mach">Mach 0.0</span></div>
                <div>Temp. Parede Cinética: <span class="resultado-dinamico" id="calc-m-temp">288.1 K</span></div>
                <div>Alerta Contramedida: <span id="calc-defesa-status" style="color:#00ff66;">Sem Ameaças</span></div>
            </div>

            <!-- TELEMETRIA GERAL -->
            <div class="bloco-formula" style="border-left-color: var(--neon-green); background: rgba(0,255,102,0.02);">
                <strong>[TELEMETRIA] HIDROSTÁTICA E COORDENADAS</strong>
                <div>Eixo Coordenada Y-Raw: <span id="calc-geo-y">340px</span></div>
                <div>Pressão Hidrostática (P): <span class="resultado-dinamico" id="calc-geo-p">2.38 MPa</span></div>
            </div>
        </div>

    </div>

    <script>
        window.onload = function() {
            const elSub = document.getElementById("submarino");
            const elRastro = document.getElementById("mhd-rastro");
            const elSros = document.getElementById("submarino-sros");
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
            let historicoTelemetria = [];

            // --- ACIONADORES DE SISTEMAS ---
            document.getElementById("btn-elevar").onclick = function() {
                subY -= 20; if (subY < 205) subY = 205;
                elSub.style.top = subY + "px";
            };

            document.getElementById("btn-submergir").onclick = function() {
                subY += 20; if (subY > 400) subY = 400;
                elSub.style.top = subY + "px";
            };

            document.getElementById("btn-mhd").onclick = function() {
                mhdAltaPotencia = !mhdAltaPotencia;
                this.classList.toggle("ativo", mhdAltaPotencia);
                if (mhdAltaPotencia) {
                    correnteJ = 1200; velX = 2.4;
                    elRastro.style.backgroundColor = "var(--neon-blue)";
                    elRastro.style.boxShadow = "0 0 10px var(--neon-blue)";
                    elRastro.style.width = "40px";
                    elRastro.style.left = "-41px";
                } else {
                    correnteJ = 450; velX = 0.8;
                    elRastro.style.backgroundColor = "#006633";
                    elRastro.style.boxShadow = "none";
                    elRastro.style.width = "25px";
                    elRastro.style.left = "-26px";
                }
            };

            document.getElementById("btn-sros").onclick = function() {
                srosElevado = !srosElevado;
                this.classList.toggle("ativo", srosElevado);
                elSros.style.display = srosElevado ? "block" : "none";
            };

            document.getElementById("btn-sams").onclick = function() {
                if (cacaAbatido) return;
                const missil = document.createElement("div");
                missil.className = "missil-sams";
                arena.appendChild(missil);
                listaSams.push({
                    el: missil,
                    x: subX + 45,
                    y: subY,
                    vx: 2,
                    vy: -5
                });
            };

            document.getElementById("btn-hiper").onclick = function() {
                const missil = document.createElement("div");
                missil.className = "missil-hiper";
                arena.appendChild(missil);
                
                const mObj = {
                    el: missil,
                    x: subX + 90,
                    y: subY,
                    mach: 1.0,
                    alvoX: 680,
                    alvoY: 175
                };
                listaHiper.push(mObj);

                // Disparo de contramedida do Bunker
                setTimeout(() => {
                    if (listaHiper.includes(mObj)) {
                        const def = document.createElement("div");
                        def.className = "missil-defesa";
                        arena.appendChild(def);
                        listaDefesaInimiga.push({
                            el: def,
                            x: 680,
                            y: 175,
                            alvoMissil: mObj
                        });
                        document.getElementById("calc-defesa-status").innerText = "INTERCEPTADOR LANÇADO";
                        document.getElementById("calc-defesa-status").style.color = "var(--alert-red)";
                    }
                }, 600);
            };

            document.getElementById("btn-report").onclick = function() {
                const data = new Date().toISOString().replace('T', ' ').substring(0, 19);
                let logText = `=== RELATÓRIO DE TELEMETRIA MILITAR [${data}] ===\n`;
                logText += `Coordenada Y Submarino: ${subY}px\n`;
                logText += `Estado Propulsão MHD: ${mhdAltaPotencia ? 'ALTA POTÊNCIA' : 'CRUZEIRO'}\n`;
                logText += `Densidade de Corrente (J): ${correnteJ} A/m²\n`;
                logText += `Mastro SROS: ${srosElevado ? 'ATIVO (SUPERFÍCIE)' : 'RECOLHIDO'}\n`;
                logText += `Histórico de Registros:\n`;
                historicoTelemetria.slice(-5).forEach((h, i) => {
                    logText += ` [${i+1}] Y:${h.y}px | P:${h.pressao}MPa | SNR:${h.snr}dB | F_L:${h.forca}N\n`;
                });

                const blob = new Blob([logText], { type: 'text/plain;charset=utf-8' });
                const a = document.createElement('a');
                a.href = URL.createObjectURL(blob);
                a.download = `telemetria_submarina_${Date.now()}.txt`;
                a.click();
            };

            function criarExplosao(x, y) {
                const exp = document.createElement("div");
                exp.className = "explosao";
                exp.style.left = (x - 10) + "px";
                exp.style.top = (y - 10) + "px";
                arena.appendChild(exp);
                setTimeout(() => exp.remove(), 400);
            }

            // --- LOOP PRINCIPAL DE SIMULAÇÃO (PHYSICS & TELEMETRY) ---
            function simular() {
                // 1. Movimento do Submarino
                subX += velX;
                if (subX > 780) subX = -95;
                elSub.style.left = subX + "px";
                elSub.style.top = subY + "px";

                // 2. Movimento do Caça Inimigo
                if (!cacaAbatido) {
                    cacaX += 1.5;
                    if (cacaX > 800) cacaX = -40;
                    elCaca.style.left = cacaX + "px";
                }

                // 3. Atualização de Mísseis S.A.M.S.
                for (let i = listaSams.length - 1; i >= 0; i--) {
                    let s = listaSams[i];
                    s.x += s.vx;
                    s.y += s.vy;
                    s.el.style.left = s.x + "px";
                    s.el.style.top = s.y + "px";

                    // Colisão com o caça
                    if (!cacaAbatido && Math.abs(s.x - (cacaX + 12)) < 20 && Math.abs(s.y - (cacaY + 6)) < 20) {
                        criarExplosao(cacaX + 12, cacaY + 6);
                        cacaAbatido = true;
                        elCaca.style.display = "none";
                        s.el.remove();
                        listaSams.splice(i, 1);

                        // Respawn do Caça após 6s
                        setTimeout(() => {
                            cacaAbatido = false;
                            cacaX = -40;
                            elCaca.style.display = "block";
                        }, 6000);
                        continue;
                    }

                    // Limpeza fora da tela
                    if (s.y < 0 || s.x > 800) {
                        s.el.remove();
                        listaSams.splice(i, 1);
                    }
                }

                // 4. Mísseis Hipersônicos
                for (let i = listaHiper.length - 1; i >= 0; i--) {
                    let h = listaHiper[i];
                    h.mach += 0.15; // Aceleração
                    let speed = h.mach * 2.2;
                    
                    let dx = h.alvoX - h.x;
                    let dy = h.alvoY - h.y;
                    let dist = Math.sqrt(dx*dx + dy*dy);

                    if (dist > 5) {
                        h.x += (dx / dist) * speed;
                        h.y += (dy / dist) * speed;
                        h.el.style.left = h.x + "px";
                        h.el.style.top = h.y + "px";

                        // Atualiza Cálculos Termodinâmicos
                        document.getElementById("calc-m-mach").innerText = "Mach " + h.mach.toFixed(1);
                        let tempK = 288.15 * (1 + 0.2 * Math.pow(h.mach, 2));
                        document.getElementById("calc-m-temp").innerText = tempK.toFixed(1) + " K";
                    } else {
                        // Impacto no Bunker
                        criarExplosao(h.alvoX, h.alvoY);
                        elBunkerStatus.innerText = "DANIFICADO";
                        elBunkerStatus.style.color = "var(--alert-red)";
                        h.el.remove();
                        listaHiper.splice(i, 1);

                        setTimeout(() => {
                            elBunkerStatus.innerText = "DEFESA: OK";
                            elBunkerStatus.style.color = "var(--neon-green)";
                            document.getElementById("calc-defesa-status").innerText = "Sem Ameaças";
                            document.getElementById("calc-defesa-status").style.color = "var(--neon-green)";
                            document.getElementById("calc-m-mach").innerText = "Mach 0.0";
                            document.getElementById("calc-m-temp").innerText = "288.1 K";
                        }, 4000);
                    }
                }

                // 5. Interceptadores do Bunker
                for (let i = listaDefesaInimiga.length - 1; i >= 0; i--) {
                    let d = listaDefesaInimiga[i];
                    if (listaHiper.includes(d.alvoMissil)) {
                        let dx = d.alvoMissil.x - d.x;
                        let dy = d.alvoMissil.y - d.y;
                        let dist = Math.sqrt(dx*dx + dy*dy);

                        if (dist < 12) {
                            // Interceptação bem sucedida
                            criarExplosao(d.x, d.y);
                            d.alvoMissil.el.remove();
                            let indexH = listaHiper.indexOf(d.alvoMissil);
                            if (indexH > -1) listaHiper.splice(indexH, 1);
                            
                            d.el.remove();
                            listaDefesaInimiga.splice(i, 1);

                            document.getElementById("calc-defesa-status").innerText = "AMEAÇA NEUTRALIZADA";
                            document.getElementById("calc-defesa-status").style.color = "var(--neon-blue)";
                            setTimeout(() => {
                                document.getElementById("calc-defesa-status").innerText = "Sem Ameaças";
                                document.getElementById("calc-defesa-status").style.color = "var(--neon-green)";
                                document.getElementById("calc-m-mach").innerText = "Mach 0.0";
                                document.getElementById("calc-m-temp").innerText = "288.1 K";
                            }, 3000);
                            continue;
                        }

                        d.x += (dx / dist) * 7;
                        d.y += (dy / dist) * 7;
                        d.el.style.left = d.x + "px";
                        d.el.style.top = d.y + "px";
                    } else {
                        // Alvo destruído ou perdido
                        d.el.remove();
                        listaDefesaInimiga.splice(i, 1);
                    }
                }

                // 6. Atualização em Tempo Real dos Módulos Matemáticos
                // Deep cálculo de profundidade
                let profundidadeM = Math.max(0, subY - 200);
                let pressaoMPa = (0.1013 + (profundidadeM * 1000 * 9.81) / 1000000).toFixed(2);
                document.getElementById("calc-geo-y").innerText = subY + "px";
                document.getElementById("calc-geo-p").innerText = pressaoMPa + " MPa";

                // Força de Lorentz: F = J * B * V (Considerando V = 1.5 m³)
                let forcaN = (correnteJ * tesla * 1.5).toFixed(1);
                document.getElementById("calc-mhd-j").innerText = correnteJ + " A/m²";
                document.getElementById("calc-mhd-b").innerText = tesla + " T";
                document.getElementById("calc-mhd-f").innerText = forcaN + " N";

                // Sonar Equation: SNR = SL - TL - (NL - DI)
                let sl = 120;
                let tl = (30 + profundidadeM * 0.1).toFixed(1);
                let nl = mhdAltaPotencia ? 75 : 45;
                let di = 15;
                let snr = sl - tl - (nl - di);

                document.getElementById("calc-sonar-nl").innerText = nl + " dB";
                document.getElementById("calc-sonar-tl").innerText = tl + " dB";
                const elSnr = document.getElementById("calc-sonar-snr");
                elSnr.innerText = (snr >= 0 ? "+" : "") + snr.toFixed(0) + " dB (" + (snr > 40 ? "Excelente" : snr > 10 ? "Bom" : "Ruidoso") + ")";

                // Atualização do log de telemetria
                if (Math.random() < 0.05) {
                    historicoTelemetria.push({
                        y: subY,
                        pressao: pressaoMPa,
                        snr: snr.toFixed(0),
                        forca: forcaN
                    });
                }

                requestAnimationFrame(simular);
            }

            requestAnimationFrame(simular);
        };
    </script>
</body>
</html>
