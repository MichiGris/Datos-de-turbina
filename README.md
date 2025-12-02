<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="theme-color" content="#0f172a">
    <meta name="description" content="Monitor de sensores ESP32 en tiempo real">
    
    <!-- PWA Meta Tags -->
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="Monitor ESP32">
    
    <!-- Manifest -->
    <link rel="manifest" href="data:application/json;base64,ewogICJuYW1lIjogIk1vbml0b3IgRVNQMzIiLAogICJzaG9ydF9uYW1lIjogIkVTUDMyIiwKICAiZGVzY3JpcHRpb24iOiAiTW9uaXRvciBkZSBzZW5zb3JlcyBFU1AzMiBlbiB0aWVtcG8gcmVhbCIsCiAgInN0YXJ0X3VybCI6ICIuLyIsCiAgImRpc3BsYXkiOiAic3RhbmRhbG9uZSIsCiAgImJhY2tncm91bmRfY29sb3IiOiAiIzBmMTcyYSIsCiAgInRoZW1lX2NvbG9yIjogIiMwZjE3MmEiLAogICJvcmllbnRhdGlvbiI6ICJwb3J0cmFpdCIsCiAgImljb25zIjogWwogICAgewogICAgICAic3JjIjogImRhdGE6aW1hZ2Uvc3ZnK3htbCwlM0NzdmclMjB4bWxucyUzRCdodHRwJTNBJTJGJTJGd3d3LnczLm9yZyUyRjIwMDAlMkZzdmcnJTIwdmlld0JveCUzRCcwJTIwMCUyMDI0JTIwMjQnJTNFJTNDcGF0aCUyMGZpbGwlM0QnJTIzM2I4MmY2JyUyMGQlM0QnTTEzJTIwMTBWM0w0JTIwMTRoN3Y3bDktMTFoLTd6JyUyRiUzRSUzQyUyRnN2ZyUzRSIsCiAgICAgICJzaXplcyI6ICIxOTJ4MTkyIiwKICAgICAgInR5cGUiOiAiaW1hZ2Uvc3ZnK3htbCIKICAgIH0sCiAgICB7CiAgICAgICJzcmMiOiAiZGF0YTppbWFnZS9zdmcreG1sLCUzQ3N2ZyUyMHhtbG5zJTNEJ2h0dHAlM0ElMkYlMkZ3d3cudzMub3JnJTJGMjAwMCUyRnN2ZyclMjB2aWV3Qm94JTNEJzAlMjAwJTIwMjQlMjAyNCclM0UlM0NwYXRoJTIwZmlsbCUzRCclMjMzYjgyZjYnJTIwZCUzRCdNMTMlMjAxMFYzTDQlMjAxNGg3djdsMzklMjAtMTFoLTd6JyUyRiUzRSUzQyUyRnN2ZyUzRSIsCiAgICAgICJzaXplcyI6ICI1MTJ4NTEyIiwKICAgICAgInR5cGUiOiAiaW1hZ2Uvc3ZnK3htbCIKICAgIH0KICBdCn0=">
    
    <title>Monitor ESP32</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }
        .animate-pulse { animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite; }
        body { 
            margin: 0; 
            padding: 0; 
            font-family: system-ui, -apple-system, sans-serif;
            overscroll-behavior: none;
            -webkit-tap-highlight-color: transparent;
        }
        
        /* Optimizaciones para móvil */
        * {
            -webkit-touch-callout: none;
            -webkit-user-select: none;
            user-select: none;
        }
        
        input, button {
            -webkit-user-select: text;
            user-select: text;
        }
        
        /* Botón de instalación */
        #installButton {
            position: fixed;
            bottom: 20px;
            right: 20px;
            z-index: 1000;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
        }
        
        /* Animación de carga */
        .loader {
            border: 3px solid rgba(59, 130, 246, 0.3);
            border-radius: 50%;
            border-top: 3px solid #3b82f6;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        /* Ocultar scrollbar pero mantener funcionalidad */
        ::-webkit-scrollbar {
            width: 4px;
        }
        
        ::-webkit-scrollbar-track {
            background: rgba(15, 23, 42, 0.5);
        }
        
        ::-webkit-scrollbar-thumb {
            background: rgba(59, 130, 246, 0.5);
            border-radius: 2px;
        }
    </style>
</head>
<body class="bg-gradient-to-br from-slate-900 via-blue-900 to-slate-900 text-white min-h-screen">
    
    <!-- Botón de instalación (solo visible si PWA no está instalada) -->
    <button id="installButton" class="hidden px-6 py-3 bg-blue-600 hover:bg-blue-700 rounded-full font-semibold flex items-center gap-2 transition-all">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" />
        </svg>
        Instalar App
    </button>

    <div id="app" class="p-4 pb-20"></div>

    <script>
        // ========================================
        // PWA - SERVICE WORKER Y INSTALACIÓN
        // ========================================
        let deferredPrompt;
        const installButton = document.getElementById('installButton');

        // Registrar Service Worker
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                // Service Worker inline para evitar archivos externos
                const swCode = `
                    const CACHE_NAME = 'esp32-monitor-v1';
                    
                    self.addEventListener('install', (event) => {
                        self.skipWaiting();
                    });
                    
                    self.addEventListener('activate', (event) => {
                        event.waitUntil(clients.claim());
                    });
                    
                    self.addEventListener('fetch', (event) => {
                        event.respondWith(
                            fetch(event.request).catch(() => {
                                return new Response('Offline', {
                                    headers: { 'Content-Type': 'text/plain' }
                                });
                            })
                        );
                    });
                `;
                
                const blob = new Blob([swCode], { type: 'application/javascript' });
                const swUrl = URL.createObjectURL(blob);
                
                navigator.serviceWorker.register(swUrl)
                    .then(reg => console.log('✅ Service Worker registrado'))
                    .catch(err => console.log('❌ Error SW:', err));
            });
        }

        // Capturar evento de instalación
        window.addEventListener('beforeinstallprompt', (e) => {
            e.preventDefault();
            deferredPrompt = e;
            installButton.classList.remove('hidden');
        });

        // Manejar clic en botón de instalación
        installButton.addEventListener('click', async () => {
            if (!deferredPrompt) return;
            
            deferredPrompt.prompt();
            const { outcome } = await deferredPrompt.userChoice;
            
            if (outcome === 'accepted') {
                console.log('✅ App instalada');
                installButton.classList.add('hidden');
            }
            
            deferredPrompt = null;
        });

        // Ocultar botón si ya está instalada
        window.addEventListener('appinstalled', () => {
            installButton.classList.add('hidden');
            console.log('✅ PWA instalada exitosamente');
        });

        // ========================================
        // CONFIGURACIÓN - CAMBIA ESTA IP
        // ========================================
        let ESP32_IP = 'http://192.168.1.100';

        // ========================================
        // ESTADO DE LA APLICACIÓN
        // ========================================
        let state = {
            connected: false,
            ipAddress: localStorage.getItem('esp32_ip') || '192.168.1.100',
            sensorData: {
                caudal: { rpm: 0, flujo: 0, estado: 'SIN FLUJO' },
                temperatura: { valor: 0, estado: 'NORMAL' },
                corriente: { valor: 0 },
                turbidez: { estado: 'Normal', alerta: false },
                nivel: { estado: 'Normal', alerta: false }
            },
            historial: [],
            error: null,
            loading: false
        };

        let updateInterval = null;

        // ========================================
        // FUNCIONES DE CONEXIÓN
        // ========================================
        function conectar() {
            ESP32_IP = `http://${state.ipAddress}`;
            localStorage.setItem('esp32_ip', state.ipAddress);
            state.connected = true;
            state.error = null;
            state.historial = [];
            state.loading = true;
            render();
            
            updateInterval = setInterval(fetchData, 1000);
            fetchData();
        }

        function desconectar() {
            state.connected = false;
            state.loading = false;
            if (updateInterval) {
                clearInterval(updateInterval);
                updateInterval = null;
            }
            render();
        }

        async function fetchData() {
            try {
                const response = await fetch(`${ESP32_IP}/data`, {
                    method: 'GET',
                    mode: 'cors',
                    cache: 'no-cache'
                });
                
                if (!response.ok) throw new Error('Error al conectar');
                
                const data = await response.json();
                state.sensorData = data;
                state.error = null;
                state.loading = false;
                
                state.historial.push({
                    timestamp: new Date().toLocaleTimeString(),
                    ...data
                });
                
                if (state.historial.length > 50) {
                    state.historial.shift();
                }
                
                render();
            } catch (error) {
                console.error('Error:', error);
                state.error = 'No se puede conectar al ESP32. Verifica la IP y conexión WiFi.';
                state.loading = false;
                render();
            }
        }

        // ========================================
        // FUNCIONES DE RENDERIZADO MÓVIL
        // ========================================
        function render() {
            const app = document.getElementById('app');
            app.innerHTML = `
                <!-- Header Compacto -->
                <div class="mb-6">
                    <div class="bg-slate-800/90 backdrop-blur rounded-2xl p-4 border border-blue-500/30 shadow-lg">
                        <h1 class="text-xl font-bold mb-3 flex items-center gap-2">
                            <svg class="w-6 h-6 text-blue-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z" />
                            </svg>
                            Monitor ESP32
                        </h1>
                        
                        <!-- Panel de Conexión -->
                        <div class="space-y-3">
                            <input
                                type="text"
                                id="ipInput"
                                value="${state.ipAddress}"
                                placeholder="IP del ESP32"
                                ${state.connected ? 'disabled' : ''}
                                class="w-full px-4 py-3 bg-slate-700 rounded-xl border border-slate-600 focus:border-blue-500 outline-none disabled:opacity-50 text-white"
                                onchange="state.ipAddress = this.value"
                            />
                            
                            <div class="flex gap-2">
                                <button
                                    onclick="${state.connected ? 'desconectar()' : 'conectar()'}"
                                    class="flex-1 px-4 py-3 rounded-xl font-semibold flex items-center justify-center gap-2 transition-all ${
                                        state.connected 
                                            ? 'bg-red-600 active:bg-red-700' 
                                            : 'bg-green-600 active:bg-green-700'
                                    }"
                                >
                                    ${state.loading ? '<div class="loader"></div>' : `
                                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                            ${state.connected 
                                                ? '<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />'
                                                : '<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8.111 16.404a5.5 5.5 0 017.778 0M12 20h.01m-7.08-7.071c3.904-3.905 10.236-3.905 14.141 0M1.394 9.393c5.857-5.857 15.355-5.857 21.213 0" />'}
                                        </svg>
                                        ${state.connected ? 'Desconectar' : 'Conectar'}
                                    `}
                                </button>
                                
                                <div class="flex items-center gap-2 px-4 py-3 rounded-xl ${
                                    state.connected ? 'bg-green-600/20 text-green-400' : 'bg-slate-700 text-slate-400'
                                }">
                                    <div class="w-2 h-2 rounded-full ${state.connected ? 'bg-green-400 animate-pulse' : 'bg-slate-500'}"></div>
                                </div>
                            </div>
                        </div>
                        
                        ${state.error ? `
                            <div class="mt-3 p-3 bg-red-600/20 border border-red-600/50 rounded-xl text-red-400 text-sm">
                                ⚠️ ${state.error}
                            </div>
                        ` : ''}
                    </div>
                </div>

                <!-- Sensores en Grid 2x3 -->
                <div class="grid grid-cols-2 gap-3 mb-6">
                    ${renderCaudalMobile()}
                    ${renderTemperaturaMobile()}
                    ${renderCorrienteMobile()}
                    ${renderTurbidezMobile()}
                    ${renderNivelMobile()}
                </div>

                <!-- Historial Compacto -->
                ${renderHistorialMobile()}
            `;
        }

        function renderCaudalMobile() {
            const d = state.sensorData.caudal;
            return `
                <div class="bg-slate-800/90 backdrop-blur rounded-2xl p-4 border border-blue-500/30 shadow-lg">
                    <div class="flex items-center gap-2 mb-2">
                        <svg class="w-5 h-5 text-blue-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 6l3 1m0 0l-3 9a5.002 5.002 0 006.001 0M6 7l3 9M6 7l6-2m6 2l3-1m-3 1l-3 9a5.002 5.002 0 006.001 0M18 7l3 9m-3-9l-6-2m0-2v2m0 16V5m0 16H9m3 0h3" />
                        </svg>
                        <h3 class="text-sm font-bold">Caudal</h3>
                    </div>
                    <div class="text-2xl font-bold text-blue-400 mb-1">
                        ${d.flujo.toFixed(0)}
                    </div>
                    <div class="text-xs text-slate-400">L/min</div>
                    <div class="mt-2 inline-block px-2 py-1 rounded text-xs font-semibold ${
                        d.estado === 'ALTO' ? 'bg-green-600' :
                        d.estado === 'MODERADO' ? 'bg-yellow-600' : 'bg-blue-600'
                    }">
                        ${d.estado}
                    </div>
                </div>
            `;
        }

        function renderTemperaturaMobile() {
            const d = state.sensorData.temperatura;
            return `
                <div class="bg-slate-800/90 backdrop-blur rounded-2xl p-4 border border-orange-500/30 shadow-lg">
                    <div class="flex items-center gap-2 mb-2">
                        <svg class="w-5 h-5 text-orange-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z" />
                        </svg>
                        <h3 class="text-sm font-bold">Temp.</h3>
                    </div>
                    <div class="text-2xl font-bold text-orange-400 mb-1">
                        ${d.valor.toFixed(1)}
                    </div>
                    <div class="text-xs text-slate-400">°C</div>
                    <div class="mt-2 inline-block px-2 py-1 rounded text-xs font-semibold ${
                        d.estado === 'CALIENTE' || d.estado === 'ALTA' ? 'bg-red-600' :
                        d.estado === 'FRÍO' ? 'bg-blue-600' : 'bg-green-600'
                    }">
                        ${d.estado}
                    </div>
                </div>
            `;
        }

        function renderCorrienteMobile() {
            const d = state.sensorData.corriente;
            return `
                <div class="bg-slate-800/90 backdrop-blur rounded-2xl p-4 border border-yellow-500/30 shadow-lg">
                    <div class="flex items-center gap-2 mb-2">
                        <svg class="w-5 h-5 text-yellow-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z" />
                        </svg>
                        <h3 class="text-sm font-bold">Corriente</h3>
                    </div>
                    <div class="text-2xl font-bold text-yellow-400 mb-1">
                        ${d.valor.toFixed(2)}
                    </div>
                    <div class="text-xs text-slate-400">A RMS</div>
                </div>
            `;
        }

        function renderTurbidezMobile() {
            const d = state.sensorData.turbidez;
            return `
                <div class="bg-slate-800/90 backdrop-blur rounded-2xl p-4 border border-cyan-500/30 shadow-lg">
                    <div class="flex items-center gap-2 mb-2">
                        <svg class="w-5 h-5 text-cyan-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 15a4 4 0 004 4h9a5 5 0 10-.1-9.999 5.002 5.002 0 10-9.78 2.096A4.001 4.001 0 003 15z" />
                        </svg>
                        <h3 class="text-sm font-bold">Turbidez</h3>
                    </div>
                    <div class="text-xl font-bold ${d.alerta ? 'text-red-400' : 'text-cyan-400'} mb-1">
                        ${d.estado}
                    </div>
                    ${d.alerta ? `
                        <div class="flex items-center gap-1 text-red-400 text-xs">
                            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
                            </svg>
                            <span>Alerta</span>
                        </div>
                    ` : '<div class="text-xs text-slate-400">Normal</div>'}
                </div>
            `;
        }

        function renderNivelMobile() {
            const d = state.sensorData.nivel;
            return `
                <div class="bg-slate-800/90 backdrop-blur rounded-2xl p-4 border border-purple-500/30 shadow-lg">
                    <div class="flex items-center gap-2 mb-2">
                        <svg class="w-5 h-5 text-purple-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z" />
                        </svg>
                        <h3 class="text-sm font-bold">Nivel</h3>
                    </div>
                    <div class="text-xl font-bold ${d.alerta ? 'text-red-400' : 'text-purple-400'} mb-1">
                        ${d.estado}
                    </div>
                    ${d.alerta ? `
                        <div class="flex items-center gap-1 text-red-400 text-xs animate-pulse">
                            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
                            </svg>
                            <span>ALARMA</span>
                        </div>
                    ` : '<div class="text-xs text-slate-400">Normal</div>'}
                </div>
            `;
        }

        function renderHistorialMobile() {
            return `
                <div class="bg-slate-800/90 backdrop-blur rounded-2xl p-4 border border-blue-500/30 shadow-lg">
                    <h2 class="text-lg font-bold mb-3">Historial</h2>
                    <div class="overflow-x-auto">
                        <div class="max-h-64 overflow-y-auto">
                            ${state.historial.length === 0 ? `
                                <div class="text-slate-400 text-center py-8 text-sm">
                                    ${state.connected ? 'Esperando datos...' : 'Conecta para ver historial'}
                                </div>
                            ` : `
                                <table class="w-full text-xs">
                                    <thead class="bg-slate-700 sticky top-0">
                                        <tr>
                                            <th class="px-2 py-2 text-left">Hora</th>
                                            <th class="px-2 py-2 text-left">Caudal</th>
                                            <th class="px-2 py-2 text-left">Temp</th>
                                            <th class="px-2 py-2 text-left">A</th>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        ${state.historial.slice(-10).reverse().map(item => `
                                            <tr class="border-b border-slate-700">
                                                <td class="px-2 py-2">${item.timestamp}</td>
                                                <td class="px-2 py-2">${item.caudal.flujo.toFixed(0)}</td>
                                                <td class="px-2 py-2">${item.temperatura.valor.toFixed(1)}°</td>
                                                <td class="px-2 py-2">${item.corriente.valor.toFixed(2)}</td>
                                            </tr>
                                        `).join('')}
                                    </tbody>
                                </table>
                            `}
                        </div>
                    </div>
                </div>
            `;
        }

        // ========================================
        // INICIALIZACIÓN
        // ========================================
        render();
        
        // Vibración al conectar (solo en móviles)
        const originalConectar = conectar;
        conectar = function() {
            if ('vibrate' in navigator) {
                navigator.vibrate(50);
            }
            originalConectar();
        };
    </script>
</body>
</html>
