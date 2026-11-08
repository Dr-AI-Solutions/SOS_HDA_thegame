<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HDA: Shock Room - Reanimación en Tiempo Real</title>
    <!-- Tailwind CSS para diseño moderno -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Inter:wght@400;600;800&display=swap');

        .font-mono-tech {
            font-family: 'Share Tech Mono', monospace;
        }

        /* Animación de pulso de monitor */
        @keyframes pulse-red {
            0%, 100% { opacity: 1; filter: drop-shadow(0 0 10px rgba(239, 68, 68, 0.8)); }
            50% { opacity: 0.4; filter: drop-shadow(0 0 2px rgba(239, 68, 68, 0.2)); }
        }
        .vital-danger {
            animation: pulse-red 1s infinite;
        }

        /* Shake effect en sangrado masivo */
        .shake {
            animation: shake 0.3s cubic-bezier(.36,.07,.19,.97) both;
        }
        @keyframes shake {
            10%, 90% { transform: translate3d(-2px, 0, 0); }
            20%, 80% { transform: translate3d(4px, 0, 0); }
            30%, 50%, 70% { transform: translate3d(-6px, 0, 0); }
            40%, 60% { transform: translate3d(6px, 0, 0); }
        }

        /* Barra de volumen de sangre intraluminal */
        .blood-pool {
            transition: height 0.3s ease-out, background-color 0.3s ease;
        }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 flex flex-col h-screen overflow-hidden select-none font-sans">

    <!-- Top Bar: Monitor de Servicio de Urgencias y Marcadores -->
    <header class="bg-slate-900 border-b border-slate-800 px-6 py-3 flex items-center justify-between shrink-0 shadow-lg">
        <div class="flex items-center gap-3">
            <div class="w-3.5 h-3.5 rounded-full bg-red-600 animate-ping"></div>
            <h1 class="text-xl font-bold tracking-wider uppercase text-slate-100 flex items-center gap-2">
                <span class="text-red-500 font-mono-tech">EMERGENCY:</span> HDA RESUSCITATION
            </h1>
            <span class="bg-red-950/80 text-red-400 text-xs px-2.5 py-0.5 rounded-full border border-red-700/50 font-medium">REANIMACIÓN ACTIVA</span>
        </div>

        <div class="flex items-center gap-6 font-mono-tech">
            <div class="text-right">
                <div class="text-xs text-slate-400">PACIENTE</div>
                <div id="patient-id" class="text-sm font-bold text-amber-400">CARLOS M. (63a)</div>
            </div>
            <div class="text-right">
                <div class="text-xs text-slate-400">TIEMPO VIVO</div>
                <div id="game-timer" class="text-lg font-bold text-emerald-400">00:00</div>
            </div>
            <div class="text-right">
                <div class="text-xs text-slate-400">PUNTAJE CLÍNICO</div>
                <div id="score-display" class="text-lg font-bold text-cyan-400">0 PTS</div>
            </div>
        </div>
    </header>

    <!-- Main Game Frame -->
    <main class="flex-1 grid grid-cols-12 gap-4 p-4 overflow-hidden">

        <!-- Columna Izquierda: Monitor Multiparámetro Hemodinámico y Fisiopatología en Vivo -->
        <section class="col-span-3 bg-slate-900/90 rounded-xl border border-slate-800 p-4 flex flex-col justify-between shadow-2xl relative">
            <div>
                <div class="flex items-center justify-between border-b border-slate-800 pb-2 mb-3">
                    <span class="text-xs uppercase font-bold tracking-wider text-slate-400 font-mono-tech">MONITOR HEMODINÁMICO</span>
                    <span class="text-xs bg-slate-800 text-slate-300 px-2 py-0.5 rounded font-mono-tech">DERIVACIÓN II</span>
                </div>

                <!-- Vitales -->
                <div class="space-y-3 font-mono-tech">
                    <!-- Presión Arterial -->
                    <div class="bg-slate-950 p-2.5 rounded-lg border border-slate-800">
                        <div class="text-xs text-slate-400 flex justify-between">
                            <span>PA / PAM (Meta ≥65)</span>
                            <span id="pam-val" class="font-bold text-emerald-400">PAM 58 mmHg</span>
                        </div>
                        <div class="text-2xl font-bold flex items-baseline gap-1" id="bp-display">
                            <span id="pas-val" class="text-red-400 text-3xl">82</span>
                            <span class="text-slate-500 text-lg">/</span>
                            <span id="pad-val" class="text-red-400 text-2xl">46</span>
                            <span class="text-xs text-slate-500 font-sans ml-1">mmHg</span>
                        </div>
                    </div>

                    <!-- Frecuencia Cardíaca -->
                    <div class="bg-slate-950 p-2.5 rounded-lg border border-slate-800">
                        <div class="text-xs text-slate-400 flex justify-between">
                            <span>FC (Taquicardia refleja)</span>
                            <span class="text-slate-500">lpm</span>
                        </div>
                        <div class="text-3xl font-bold text-emerald-400" id="hr-display">124</div>
                    </div>

                    <!-- Saturación O2 & Vía Aérea -->
                    <div class="bg-slate-950 p-2.5 rounded-lg border border-slate-800">
                        <div class="text-xs text-slate-400 flex justify-between">
                            <span>SatO2 & VÍA AÉREA</span>
                            <span id="airway-status" class="text-amber-400 font-bold">BIGOTERA (Vulnerable)</span>
                        </div>
                        <div class="text-2xl font-bold text-cyan-400" id="spo2-display">93%</div>
                    </div>

                    <!-- Nivel de Hemoglobina y Umbral Transfusional -->
                    <div class="bg-slate-950 p-2.5 rounded-lg border border-slate-800">
                        <div class="text-xs text-slate-400 flex justify-between">
                            <span>HEMOGLOBINA (Gatillo &lt;7.0)</span>
                            <span class="text-slate-500">g/dL</span>
                        </div>
                        <div class="flex items-baseline justify-between">
                            <span class="text-3xl font-bold text-rose-500" id="hb-display">6.4</span>
                            <span class="text-xs text-slate-400 font-sans" id="hb-target">Meta: 7.0 - 8.0</span>
                        </div>
                    </div>

                    <!-- pH Intragástrico y Coagulación -->
                    <div class="bg-slate-950 p-2.5 rounded-lg border border-slate-800">
                        <div class="text-xs text-slate-400 flex justify-between">
                            <span>pH GÁSTRICO (Inactivar Pepsina ≥6.0)</span>
                            <span id="ph-warning" class="text-red-400 text-[10px] font-bold">LISIS ACTIVA COÁGULO</span>
                        </div>
                        <div class="text-2xl font-bold text-amber-300" id="ph-display">2.2</div>
                    </div>
                </div>
            </div>

            <!-- Panel de Fisiopatología Oculta (Diagnóstico Etiológico) -->
            <div class="mt-3 bg-slate-950/70 p-3 rounded-lg border border-slate-800 font-mono-tech text-xs space-y-1.5">
                <div class="text-slate-400 font-bold border-b border-slate-800 pb-1">PISTAS DEL EXPEDIENTE</div>
                <div class="text-slate-300 flex justify-between">
                    <span>AINEs crónicos:</span> <span id="hint-nsaid" class="text-amber-400 font-bold">Naproxeno + Ketorolaco</span>
                </div>
                <div class="text-slate-300 flex justify-between">
                    <span>Estigmas Hepáticos:</span> <span id="hint-dhc" class="text-emerald-400 font-bold">Ausentes (No DHC)</span>
                </div>
                <div class="text-slate-300 flex justify-between">
                    <span>BUN / Creatinina:</span> <span id="hint-bun" class="text-purple-400 font-bold">54 / 1.1 (Ratio 49:1)</span>
                </div>
            </div>
        </section>

        <!-- Columna Central: Escenario de Acción (Gástrico / Endoscopía Interactiva) -->
        <section id="game-arena" class="col-span-6 bg-slate-900 rounded-xl border border-slate-800 p-4 flex flex-col justify-between shadow-2xl relative overflow-hidden">
            
            <!-- Header de Estado de la Hemorragia -->
            <div class="flex items-center justify-between z-10">
                <div class="flex items-center gap-2">
                    <span class="text-xs font-mono-tech bg-slate-800 px-2 py-1 rounded text-slate-300">CAMPO VISUAL INTRA-LUMINAL</span>
                    <span id="bleeding-badge" class="text-xs font-bold font-mono-tech bg-red-600/90 text-white px-2 py-1 rounded animate-pulse">SANGRADO ACTIVO: EN CHORRO (Forrest Ia)</span>
                </div>
                <div class="text-xs font-mono-tech text-slate-400">
                    VISIBILIDAD EDA: <span id="visibility-val" class="font-bold text-rose-400">20% (Borra/Sangre)</span>
                </div>
            </div>

            <!-- Área Visual del Órgano y Hemostasia (Canvas de Acción) -->
            <div class="relative w-full h-80 my-auto rounded-lg bg-slate-950 border border-slate-800 overflow-hidden flex items-center justify-center">
                
                <!-- Sangre acumulada en estómago (simulación) -->
                <div id="stomach-blood" class="absolute bottom-0 left-0 w-full bg-gradient-to-t from-red-950 via-red-900/80 to-transparent blood-pool" style="height: 65%;"></div>

                <!-- Representación de Mucosa y Vaso Erosionado -->
                <div class="relative z-10 flex flex-col items-center">
                    <div id="ulcer-site" class="w-32 h-32 rounded-full border-4 border-red-900 bg-red-950/70 flex items-center justify-center relative cursor-crosshair shadow-inner group">
                        
                        <!-- Chorro o fuga activa -->
                        <div id="blood-jet" class="w-3 h-16 bg-red-500 rounded-full blur-[1px] absolute -top-8 animate-bounce"></div>
                        
                        <!-- Coágulo o vaso -->
                        <div id="vessel-target" class="w-8 h-8 rounded-full bg-red-700 border-2 border-red-400 flex items-center justify-center font-mono-tech text-[10px] text-white shadow-lg">
                            VASO
                        </div>

                        <!-- Clips aplicados dinámicamente -->
                        <div id="applied-clips" class="absolute inset-0 flex items-center justify-center pointer-events-none"></div>
                    </div>
                    <div id="action-prompt" class="mt-4 font-mono-tech text-xs bg-black/80 px-3 py-1 rounded border border-slate-700 text-slate-300">
                        Haz clic en el vaso para aplicar terapia mecánica una vez despejado el campo.
                    </div>
                </div>

                <!-- Efecto de hematemesis masiva repentina -->
                <div id="vomit-overlay" class="absolute inset-0 bg-red-900/80 flex items-center justify-center opacity-0 pointer-events-none transition-opacity duration-200">
                    <div class="text-center font-mono-tech">
                        <div class="text-3xl font-extrabold text-white tracking-widest uppercase">¡HEMATEMESIS MASIVA!</div>
                        <div class="text-sm text-red-200 mt-1">Vía aérea comprometida. Broncoaspiración inminente.</div>
                    </div>
                </div>
            </div>

            <!-- Feed de Eventos Clínicos Dinámicos -->
            <div class="bg-slate-950 rounded-lg p-3 border border-slate-800 h-28 overflow-y-auto font-mono-tech text-xs space-y-1" id="console-log">
                <div class="text-slate-500">[00:00] Paciente ingresa diaforético, taquicárdico, hipotenso. Hematemesis en curso.</div>
                <div class="text-amber-400">[00:00] ALERTA: Alto riesgo por Glasgow-Blatchford > 2. pH ácido destruyendo factores de coagulación.</div>
            </div>
        </section>

        <!-- Columna Derecha: Consola de Decisión y Acciones de Reanimación Rápida -->
        <section class="col-span-3 bg-slate-900/90 rounded-xl border border-slate-800 p-4 flex flex-col justify-between shadow-2xl">
            <div>
                <div class="border-b border-slate-800 pb-2 mb-3">
                    <span class="text-xs uppercase font-bold tracking-wider text-slate-400 font-mono-tech">INTERVENCIONES CLÍNICAS (HOTKEYS: 1-8)</span>
                </div>

                <!-- Botones de Acción Inmediata Agrupados -->
                <div class="space-y-2 font-mono-tech text-xs">
                    
                    <!-- 1. Vía Aérea -->
                    <button onclick="actionIntubate()" class="w-full bg-slate-800 hover:bg-slate-700 active:bg-cyan-900 border border-slate-700 hover:border-cyan-500 p-2.5 rounded-lg flex items-center justify-between transition group">
                        <span class="font-bold text-slate-200 group-hover:text-cyan-400">[1] IOT + Vía Aérea</span>
                        <span class="text-[10px] bg-slate-900 text-slate-400 px-1.5 py-0.5 rounded">Proteger</span>
                    </button>

                    <!-- 2. Reanimación Cristaloides -->
                    <button onclick="actionFluids()" class="w-full bg-slate-800 hover:bg-slate-700 active:bg-blue-900 border border-slate-700 hover:border-blue-500 p-2.5 rounded-lg flex items-center justify-between transition group">
                        <span class="font-bold text-slate-200 group-hover:text-blue-400">[2] Bolo SF 0.9% (500 cc)</span>
                        <span class="text-[10px] bg-slate-900 text-slate-400 px-1.5 py-0.5 rounded">Titular PAM</span>
                    </button>

                    <!-- 3. Transfusión Restrictiva -->
                    <button onclick="actionTransfuse()" class="w-full bg-slate-800 hover:bg-slate-700 active:bg-red-900 border border-slate-700 hover:border-red-500 p-2.5 rounded-lg flex items-center justify-between transition group">
                        <span class="font-bold text-slate-200 group-hover:text-red-400">[3] Transfundir 1U GR</span>
                        <span class="text-[10px] bg-slate-900 text-slate-400 px-1.5 py-0.5 rounded">Gatillo &lt;7.0</span>
                    </button>

                    <!-- 4. Inhibidor Bomba Protones -->
                    <button onclick="actionOmeprazole()" class="w-full bg-slate-800 hover:bg-slate-700 active:bg-amber-900 border border-slate-700 hover:border-amber-500 p-2.5 rounded-lg flex items-center justify-between transition group">
                        <span class="font-bold text-slate-200 group-hover:text-amber-400">[4] Omeprazol 80mg EV</span>
                        <span class="text-[10px] bg-slate-900 text-slate-400 px-1.5 py-0.5 rounded">Subir pH &gt;6</span>
                    </button>

                    <!-- 5. Procinético Pre-EDA -->
                    <button onclick="actionErythromycin()" class="w-full bg-slate-800 hover:bg-slate-700 active:bg-purple-900 border border-slate-700 hover:border-purple-500 p-2.5 rounded-lg flex items-center justify-between transition group">
                        <span class="font-bold text-slate-200 group-hover:text-purple-400">[5] Eritromicina 250mg</span>
                        <span class="text-[10px] bg-slate-900 text-slate-400 px-1.5 py-0.5 rounded">Limpiar campo</span>
                    </button>

                    <!-- 6. Vasoactivos (Dilema Variceal vs No Variceal) -->
                    <button onclick="actionTerlipressin()" class="w-full bg-slate-800 hover:bg-slate-700 active:bg-rose-900 border border-slate-700 hover:border-rose-500 p-2.5 rounded-lg flex items-center justify-between transition group">
                        <span class="font-bold text-slate-200 group-hover:text-rose-400">[6] Terlipresina 2mg EV</span>
                        <span class="text-[10px] bg-slate-900 text-slate-400 px-1.5 py-0.5 rounded">Vasoactivo</span>
                    </button>

                    <!-- 7. Terapia Endoscópica Combinada -->
                    <button onclick="actionEndoTherapy()" class="w-full bg-slate-800 hover:bg-slate-700 active:bg-emerald-900 border border-slate-700 hover:border-emerald-500 p-2.5 rounded-lg flex items-center justify-between transition group">
                        <span class="font-bold text-slate-200 group-hover:text-emerald-400">[7] Clip + Adrenalina (EDA)</span>
                        <span class="text-[10px] bg-slate-900 text-slate-400 px-1.5 py-0.5 rounded">Hemostasia</span>
                    </button>

                    <!-- 8. Antibiótico Profiláctico -->
                    <button onclick="actionCeftriaxone()" class="w-full bg-slate-800 hover:bg-slate-700 border border-slate-700 hover:border-teal-500 p-2.5 rounded-lg flex items-center justify-between transition group">
                        <span class="font-bold text-slate-200 group-hover:text-teal-400">[8] Ceftriaxona 1g EV</span>
                        <span class="text-[10px] bg-slate-900 text-slate-400 px-1.5 py-0.5 rounded">Profilaxis PBE</span>
                    </button>
                </div>
            </div>

            <!-- Botón de Pausa / Reinicio Rápido -->
            <div class="pt-2">
                <button onclick="resetGame()" class="w-full py-2 bg-slate-800 hover:bg-slate-700 text-slate-300 rounded font-mono-tech text-xs tracking-wider border border-slate-700 transition">
                    REINICIAR ESCENARIO (ESC)
                </button>
            </div>
        </section>
    </main>

    <!-- Modal de Fin de Partida / Retroalimentación Fisiopatológica Inmediata -->
    <div id="modal-gameover" class="fixed inset-0 bg-slate-950/90 backdrop-blur-md z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-slate-900 border border-slate-700 rounded-2xl max-w-lg w-full p-6 shadow-2xl font-mono-tech text-left">
            <div id="modal-title" class="text-2xl font-bold mb-2">PACIENTE FALLECIDO EN BOX</div>
            <div id="modal-desc" class="text-sm text-slate-300 mb-4 leading-relaxed font-sans">
                El paciente desarrolló fibrilación ventricular secundaria a shock hipovolémico descompensado y acidosis profunda.
            </div>

            <!-- Fisiopatología del Error -->
            <div class="bg-slate-950 p-4 rounded-xl border border-slate-800 mb-4 space-y-2 text-xs">
                <div class="font-bold text-amber-400 uppercase tracking-wide">ANÁLISIS DEL INTERNO DOCENTE:</div>
                <div id="modal-critique" class="text-slate-300 font-sans leading-normal">
                    - Administraste Terlipresina en un paciente no cirrótico con úlcera inducida por AINEs, generando isquemia coronaria innecesaria.<br>
                    - No lograste elevar el pH gástrico por encima de 6.0 antes de intentar la hemostasia mecánica, por lo que la pepsina disolvió la agregación plaquetaria.
                </div>
            </div>

            <div class="flex items-center justify-between pt-2">
                <div>
                    <span class="text-xs text-slate-500">Puntaje Final:</span>
                    <span id="final-score" class="text-lg font-bold text-cyan-400 ml-1">0 PTS</span>
                </div>
                <button onclick="resetGame()" class="px-5 py-2.5 bg-red-600 hover:bg-red-500 text-white font-bold rounded-lg transition tracking-wider">
                    JUGAR DE NUEVO
                </button>
            </div>
        </div>
    </div>

    <!-- Lógica del Motor de Juego (Vanilla JS) -->
    <script>
        /* ================= ESTADO CLÍNICO GLOBAL ================= */
        let state = {
            active: true,
            timeSec: 0,
            score: 0,
            
            // Hemodinamia
            pas: 82,
            pad: 46,
            pam: 58,
            hr: 124,
            spo2: 93,
            hb: 6.4,
            ph: 2.2,

            // Variables de Entorno y Manejo
            intubated: false,
            fluidsGivenCount: 0,
            unitsTransfused: 0,
            omeprazoleActive: false,
            erythromycinActive: false,
            terlipressinActive: false,
            ceftriaxoneGiven: false,
            clipped: false,
            stomachBloodLevel: 65, // %
            bleedingSeverity: 2.5, // Velocidad de sangrado activa
            clotStability: 10,     // 0 a 100%
            
            // Fisiopatología del Paciente (Generación Procedural: Úlcera por AINEs vs Várices)
            etiology: 'ulcer', // 'ulcer' o 'variceal'
        };

        let gameInterval = null;

        /* ================= INICIO Y BUCLE PRINCIPAL ================= */
        function initGame() {
            state.active = true;
            state.timeSec = 0;
            state.score = 100;
            state.pas = 82;
            state.pad = 46;
            state.pam = Math.round((state.pas + 2 * state.pad) / 3);
            state.hr = 124;
            state.spo2 = 93;
            state.hb = 6.4;
            state.ph = 2.2;
            state.intubated = false;
            state.fluidsGivenCount = 0;
            state.unitsTransfused = 0;
            state.omeprazoleActive = false;
            state.erythromycinActive = false;
            state.terlipressinActive = false;
            state.ceftriaxoneGiven = false;
            state.clipped = false;
            state.stomachBloodLevel = 65;
            state.bleedingSeverity = 2.5;
            state.clotStability = 10;
            
            // 60% de los casos úlcera por AINEs, 40% várices
            state.etiology = Math.random() > 0.4 ? 'ulcer' : 'variceal';
            setupPatientCase();

            document.getElementById('modal-gameover').classList.add('hidden');
            document.getElementById('applied-clips').innerHTML = '';
            document.getElementById('console-log').innerHTML = '';
            
            log(`[00:00] Ingreso: Paciente en SHOCK hipovolémico. Sospecha de HDA activa. Glasgow-Blatchford de alto riesgo.`);
            
            clearInterval(gameInterval);
            gameInterval = setInterval(gameTick, 1000);
            updateUI();
        }

        function setupPatientCase() {
            if (state.etiology === 'ulcer') {
                document.getElementById('patient-id').textContent = "DON CARLOS (63a) - ÚLCERA";
                document.getElementById('hint-nsaid').textContent = "Naproxeno + Ketorolaco (+) habitual";
                document.getElementById('hint-dhc').textContent = "Ausentes / Hígado indemne";
                document.getElementById('hint-bun').textContent = "62 / 1.1 (Ratio > 30:1 alto)";
            } else {
                document.getElementById('patient-id').textContent = "DON ERNESTO (58a) - DHC";
                document.getElementById('hint-nsaid').textContent = "Niega uso de analgésicos";
                document.getElementById('hint-dhc').textContent = "Ascitis leve, Telangiectasias, Plaquetas 48k";
                document.getElementById('hint-bun').textContent = "48 / 1.3 (Disfunción hepatorrenal)";
            }
        }

        /* ================= TICK DEL JUEGO (CADA SEGUNDO) ================= */
        function gameTick() {
            if (!state.active) return;
            state.timeSec++;

            // 1. Dinámica de Sangrado y Estabilidad
            if (!state.clipped) {
                // Sangrado activo continua llenando el estómago
                state.stomachBloodLevel = Math.min(100, state.stomachBloodLevel + state.bleedingSeverity);
                
                // Pérdida sostenida de volumen y glóbulos rojos
                state.hb = Math.max(3.0, +(state.hb - 0.04).toFixed(2));
                state.pas = Math.max(40, state.pas - 1);
                state.pad = Math.max(25, state.pad - 0.5);
                state.hr = Math.min(160, state.hr + 1);
            } else {
                // Hemostasia lograda: reabsorción o vaciamiento paulatino
                state.stomachBloodLevel = Math.max(10, state.stomachBloodLevel - 1.5);
            }

            // 2. Efecto de la Secreción Ácida y Pepsina sobre el Coágulo
            if (state.ph < 6.0) {
                // A pH ácido la agregación plaquetaria es disuelta
                state.clotStability = Math.max(0, state.clotStability - 3);
            } else {
                state.clotStability = Math.min(100, state.clotStability + 2);
            }

            // 3. Ventana de Hematemesis Masiva y Vía Aérea
            if (state.stomachBloodLevel >= 90) {
                triggerHematemesisAlert();
                if (!state.intubated) {
                    state.spo2 = Math.max(50, state.spo2 - 6);
                    if (state.spo2 < 70) {
                        endGame(false, "PARO RESPIRATORIO POR BRONCOASPIRACIÓN", 
                        "El estómago sobrepasó el volumen de tolerancia y el paciente vomitó profusamente sin protección de vía aérea. Debiste intubar (IOT) precozmente ante el sangrado activo incontrolado.");
                        return;
                    }
                }
            } else {
                if (state.intubated) {
                    state.spo2 = Math.min(99, state.spo2 + 1);
                }
            }

            // 4. Cálculo de PAM
            state.pam = Math.round((state.pas + 2 * state.pad) / 3);

            // 5. Criterios de Shock Irreversible y Muerte
            if (state.hb < 4.5 || state.pam < 45) {
                endGame(false, "SHOCK HIPOVOLÉMICO IRREVERSIBLE", 
                "El volumen intravascular efectivo cayó por debajo de los límites viables de perfusión coronaria y cerebral. Faltó reanimación con bolos medidos de cristaloides o transfusión restrictiva oportuna.");
                return;
            }

            // 6. Criterio de Victoria / Estabilización Exitosa
            if (state.clipped && state.pam >= 65 && state.hb >= 7.0 && state.ph >= 6.0 && state.stomachBloodLevel < 35) {
                state.score += 250;
                endGame(true, "¡PACIENTE ESTABILIZADO CON ÉXITO!", 
                `Excelente enfrentamiento. Lograste hemostasia combinada definitiva, mantuviste el pH gástrico ≥6.0 para proteger el trombo primario, reanimaste con estrategia restrictiva sin sobrecargar al paciente y protegiste la vía aérea.`);
                return;
            }

            updateUI();
        }

        /* ================= ACCIONES CLÍNICAS ================= */

        // [1] Intubación Orotraqueal
        function actionIntubate() {
            if (!state.active) return;
            if (state.intubated) {
                log(`[${timeStr()}] Advertencia: El paciente ya está intubado con TOT fijado.`);
                return;
            }
            state.intubated = true;
            state.score += 20;
            log(`[${timeStr()}] IOT realizada con éxito. Vía aérea protegida contra aspiración.`);
            updateUI();
        }

        // [2] Cristaloides (SF 0.9%)
        function actionFluids() {
            if (!state.active) return;
            state.fluidsGivenCount++;
            
            // Fisiopatología de fluidos: Reanima si está en shock, pero sobrecarga daña si se abusa
            if (state.pam < 65) {
                state.pas += 8;
                state.pad += 4;
                state.hr = Math.max(90, state.hr - 6);
                state.score += 15;
                log(`[${timeStr()}] Bolo SF 0.9% 500cc infundido. PAM mejorada hacia meta.`);
            } else {
                // Sobreexpansión perjudicial: hemodilución y aumento de presión de rotura
                state.hb = +(state.hb - 0.3).toFixed(2);
                state.score -= 20;
                log(`[${timeStr()}] PELIGRO: Sobrehidratación en paciente sin hipotensión grave. Hemodilución inducida y lavado de factores.`);
                if (state.etiology === 'variceal') {
                    state.bleedingSeverity += 0.5;
                    log(`[${timeStr()}] Aumento de presión portal hidrostática: sangrado variceal intensificado.`);
                }
            }
            updateUI();
        }

        // [3] Transfusión de Glóbulos Rojos
        function actionTransfuse() {
            if (!state.active) return;
            
            // Estrategia Restrictiva (Villanueva et al.): Gatillo <7.0, Meta 7-8
            if (state.hb < 7.0) {
                state.hb = +(state.hb + 1.0).toFixed(2);
                state.pas += 5;
                state.unitsTransfused++;
                state.score += 30;
                log(`[${timeStr()}] Transfusión 1U GR correcta (Gatillo <7.0 cumplido). Hb actual: ${state.hb} g/dL.`);
            } else {
                // Sobretransfusión (Estrategia liberal perjudicial)
                state.hb = +(state.hb + 1.0).toFixed(2);
                state.score -= 40;
                log(`[${timeStr()}] ERROR TÁCTICO: Sobretransfusión con Hb ≥ 7.0. ↑ Resangrado por aumento de presión e hiperviscosidad.`);
                if (state.etiology === 'variceal') {
                    state.bleedingSeverity += 1.0;
                }
            }
            updateUI();
        }

        // [4] Omeprazol en Altas Dosis
        function actionOmeprazole() {
            if (!state.active) return;
            if (state.omeprazoleActive) {
                log(`[${timeStr()}] Omeprazol ya se encuentra en infusión continua.`);
                return;
            }
            state.omeprazoleActive = true;
            state.ph = 6.4;
            state.score += 25;
            log(`[${timeStr()}] Bolo Omeprazol 80mg EV infundido. pH gástrico elevado a ${state.ph}. Pepsina inactivada.`);
            updateUI();
        }

        // [5] Eritromicina Pre-EDA
        function actionErythromycin() {
            if (!state.active) return;
            if (state.erythromycinActive) {
                log(`[${timeStr()}] Eritromicina ya administrada recientemente.`);
                return;
            }
            state.erythromycinActive = true;
            state.stomachBloodLevel = Math.max(10, state.stomachBloodLevel - 35);
            state.score += 20;
            log(`[${timeStr()}] Eritromicina 250mg EV infundida. Vaciamiento gástrico acelerado, visibilidad endoscópica mejorada.`);
            updateUI();
        }

        // [6] Terlipresina EV
        function actionTerlipressin() {
            if (!state.active) return;
            
            if (state.etiology === 'variceal') {
                state.terlipressinActive = true;
                state.bleedingSeverity = Math.max(0.8, state.bleedingSeverity - 1.5);
                state.score += 35;
                log(`[${timeStr()}] EXCELENTE: Terlipresina indicada oportunamente para sangrado por hipertensión portal.`);
            } else {
                // Error grave: Úlcera péptica no variceal tratada con vasoconstrictor esplácnico
                state.score -= 50;
                state.hr += 10;
                state.pas -= 10; // Vasoespasmo o isquemia
                log(`[${timeStr()}] IATROGENIA: Paciente con úlcera por AINEs (no cirrótico). Terlipresina causó isquemia periférica y riesgo coronario.`);
            }
            updateUI();
        }

        // [7] Terapia Endoscópica (Clip + Adrenalina / Bandas)
        function actionEndoTherapy() {
            if (!state.active) return;

            // Se requiere visibilidad adecuada y estabilidad mínima
            let visibility = 100 - state.stomachBloodLevel;
            if (visibility < 40) {
                log(`[${timeStr()}] FALLA TÉCNICA: Campo endoscópico inundado de sangre (${visibility}% visibilidad). Usa Eritromicina o aspira primero.`);
                state.score -= 10;
                return;
            }

            if (state.ph < 6.0) {
                log(`[${timeStr()}] ADVERTENCIA: Intentas hemostasia con pH ${state.ph}. El coágulo será lisado por la pepsina activa.`);
            }

            state.clipped = true;
            state.bleedingSeverity = 0;
            state.score += 60;

            // Visual clip en la úlcera
            const clipsDiv = document.getElementById('applied-clips');
            clipsDiv.innerHTML = `
                <div class="w-6 h-6 rounded bg-amber-400 border border-white flex items-center justify-center font-bold text-[9px] text-black shadow-lg animate-pulse">
                    CLIP
                </div>
            `;
            document.getElementById('blood-jet').classList.add('hidden');
            log(`[${timeStr()}] ÉXITO: Clip hemostático desplegado en el vaso visible (Forrest Ia -> Resuelto). Hemorragia detenida.`);
            updateUI();
        }

        // [8] Ceftriaxona
        function actionCeftriaxone() {
            if (!state.active) return;
            if (state.ceftriaxoneGiven) return;
            state.ceftriaxoneGiven = true;

            if (state.etiology === 'variceal') {
                state.score += 25;
                log(`[${timeStr()}] Profilaxis antibiótica con Ceftriaxona iniciada: Previene translocación bacteriana y resangrado en cirrótico.`);
            } else {
                state.score -= 15;
                log(`[${timeStr()}] Uso indiscriminado de Ceftriaxona en paciente no cirrótico sin foco séptico.`);
            }
            updateUI();
        }

        /* ================= HELPERS Y UI ================= */
        function triggerHematemesisAlert() {
            const overlay = document.getElementById('vomit-overlay');
            overlay.style.opacity = '1';
            document.getElementById('game-arena').classList.add('shake');
            setTimeout(() => {
                overlay.style.opacity = '0';
                document.getElementById('game-arena').classList.remove('shake');
            }, 1000);
        }

        function log(msg) {
            const consoleBox = document.getElementById('console-log');
            const div = document.createElement('div');
            div.textContent = msg;
            consoleBox.appendChild(div);
            consoleBox.scrollTop = consoleBox.scrollHeight;
        }

        function timeStr() {
            let m = Math.floor(state.timeSec / 60).toString().padStart(2, '0');
            let s = (state.timeSec % 60).toString().padStart(2, '0');
            return `${m}:${s}`;
        }

        function updateUI() {
            document.getElementById('game-timer').textContent = timeStr();
            document.getElementById('score-display').textContent = `${state.score} PTS`;

            // Vitales
            document.getElementById('pas-val').textContent = Math.round(state.pas);
            document.getElementById('pad-val').textContent = Math.round(state.pad);
            document.getElementById('pam-val').textContent = `PAM ${state.pam} mmHg`;
            document.getElementById('hr-display').textContent = Math.round(state.hr);
            document.getElementById('spo2-display').textContent = `${Math.round(state.spo2)}%`;
            document.getElementById('hb-display').textContent = state.hb.toFixed(1);
            document.getElementById('ph-display').textContent = state.ph.toFixed(1);

            // Vía Aérea
            const airwayEl = document.getElementById('airway-status');
            if (state.intubated) {
                airwayEl.textContent = "TOT PROTEGIDO";
                airwayEl.className = "text-emerald-400 font-bold";
            } else {
                airwayEl.textContent = state.stomachBloodLevel > 70 ? "¡RIESGO INMINENTE!" : "BIGOTERA (Vulnerable)";
                airwayEl.className = state.stomachBloodLevel > 70 ? "text-red-500 font-bold animate-pulse" : "text-amber-400 font-bold";
            }

            // pH Alert
            const phWarning = document.getElementById('ph-warning');
            if (state.ph >= 6.0) {
                phWarning.textContent = "PEPSINA INHIBIDA (HEMOSTASIA ÓPTIMA)";
                phWarning.className = "text-emerald-400 text-[10px] font-bold";
            } else {
                phWarning.textContent = "LISIS ACTIVA COÁGULO (PEPSINA +)";
                phWarning.className = "text-red-400 text-[10px] font-bold";
            }

            // Campo gástrico y sangre
            document.getElementById('stomach-blood').style.height = `${state.stomachBloodLevel}%`;
            let visibility = Math.max(5, 100 - Math.round(state.stomachBloodLevel));
            document.getElementById('visibility-val').textContent = `${visibility}% (${visibility < 40 ? 'Crítica/Inundado' : 'Aceptable'})`;
            document.getElementById('visibility-val').className = visibility < 40 ? "font-bold text-rose-500" : "font-bold text-emerald-400";

            // Forrest Badge
            const badge = document.getElementById('bleeding-badge');
            if (state.clipped) {
                badge.textContent = "HEMOSTASIA LOGRADA (Forrest III/Fibrina)";
                badge.className = "text-xs font-bold font-mono-tech bg-emerald-700 text-white px-2 py-1 rounded";
            } else {
                badge.textContent = "SANGRADO ACTIVO: EN CHORRO (Forrest Ia)";
                badge.className = "text-xs font-bold font-mono-tech bg-red-600/90 text-white px-2 py-1 rounded animate-pulse";
            }
        }

        function endGame(victory, title, critique) {
            state.active = false;
            clearInterval(gameInterval);
            
            const modal = document.getElementById('modal-gameover');
            const titleEl = document.getElementById('modal-title');
            const descEl = document.getElementById('modal-desc');
            const critEl = document.getElementById('modal-critique');
            const scoreEl = document.getElementById('final-score');

            titleEl.textContent = title;
            titleEl.className = victory ? "text-2xl font-bold mb-2 text-emerald-400" : "text-2xl font-bold mb-2 text-rose-500";
            descEl.textContent = victory ? "Completaste la estabilización integral según las metas ESGE y Baveno VII." : "El caso clínico culminó con desenlace adverso debido a brechas en la reanimación.";
            critEl.innerHTML = critique;
            scoreEl.textContent = `${Math.max(0, state.score)} PTS`;

            modal.classList.remove('hidden');
        }

        function resetGame() {
            initGame();
        }

        // Manejo por teclado directo (Hotkeys 1-8 y ESC)
        window.addEventListener('keydown', (e) => {
            if (e.key === '1') actionIntubate();
            if (e.key === '2') actionFluids();
            if (e.key === '3') actionTransfuse();
            if (e.key === '4') actionOmeprazole();
            if (e.key === '5') actionErythromycin();
            if (e.key === '6') actionTerlipressin();
            if (e.key === '7') actionEndoTherapy();
            if (e.key === '8') actionCeftriaxone();
            if (e.key === 'Escape') resetGame();
        });

        // Click directo en el vaso para clip
        document.getElementById('ulcer-site').addEventListener('click', () => {
            actionEndoTherapy();
        });

        // Cargar al inicio
        window.onload = initGame;
    </script>
</body>
</html>