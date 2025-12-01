import React, { useState, useEffect } from 'react';
import { Activity, Droplets, Thermometer, Zap, CloudRain, AlertTriangle, Wifi, WifiOff, Download } from 'lucide-react';

const ESP32Monitor = () => {
  const [connected, setConnected] = useState(false);
  const [ipAddress, setIpAddress] = useState('192.168.1.100');
  const [sensorData, setSensorData] = useState({
    caudal: { rpm: 0, flujo: 0, estado: 'SIN FLUJO' },
    temperatura: { valor: 0, estado: 'NORMAL' },
    corriente: { valor: 0 },
    turbidez: { estado: 'Normal', alerta: false },
    nivel: { estado: 'Normal', alerta: false }
  });
  const [historial, setHistorial] = useState([]);

  // Simular conexión WiFi y recepción de datos
  useEffect(() => {
    if (!connected) return;

    const interval = setInterval(() => {
      // Simular datos recibidos desde ESP32
      const nuevosDatos = {
        caudal: {
          rpm: Math.random() * 3000,
          flujo: Math.random() * 25000,
          estado: ['BAJO', 'MODERADO', 'ALTO'][Math.floor(Math.random() * 3)]
        },
        temperatura: {
          valor: 20 + Math.random() * 15,
          estado: ['FRÍO', 'NORMAL', 'CALIENTE'][Math.floor(Math.random() * 3)]
        },
        corriente: {
          valor: Math.random() * 5
        },
        turbidez: {
          estado: Math.random() > 0.3 ? 'Normal' : 'ALERTA',
          alerta: Math.random() > 0.3
        },
        nivel: {
          estado: Math.random() > 0.2 ? 'Normal' : 'BAJO',
          alerta: Math.random() > 0.2
        }
      };

      setSensorData(nuevosDatos);
      
      setHistorial(prev => {
        const nuevo = [...prev, {
          timestamp: new Date().toLocaleTimeString(),
          fecha: new Date().toLocaleDateString(),
          ...nuevosDatos
        }];
        return nuevo.slice(-20);
      });
    }, 1000);

    return () => clearInterval(interval);
  }, [connected]);

  const conectar = () => {
    setConnected(true);
    setHistorial([]);
  };

  const desconectar = () => {
    setConnected(false);
  };

  // Función para descargar el historial como CSV
  const descargarHistorial = () => {
    if (historial.length === 0) {
      alert('No hay datos para descargar');
      return;
    }

    // Encabezados del CSV
    const encabezados = ['Fecha', 'Hora', 'Caudal (L/min)', 'RPM', 'Estado Caudal', 'Temperatura (°C)', 'Estado Temperatura', 'Corriente (A)', 'Turbidez', 'Alerta Turbidez', 'Nivel', 'Alerta Nivel'];
    
    // Convertir datos a filas CSV
    const filasCSV = historial.map(item => [
      item.fecha,
      item.timestamp,
      item.caudal.flujo.toFixed(2),
      item.caudal.rpm.toFixed(2),
      item.caudal.estado,
      item.temperatura.valor.toFixed(2),
      item.temperatura.estado,
      item.corriente.valor.toFixed(3),
      item.turbidez.estado,
      item.turbidez.alerta ? 'SI' : 'NO',
      item.nivel.estado,
      item.nivel.alerta ? 'SI' : 'NO'
    ]);

    // Combinar encabezados y datos
    const contenidoCSV = [
      encabezados.join(','),
      ...filasCSV.map(fila => fila.join(','))
    ].join('\n');

    // Crear y descargar archivo
    const blob = new Blob([contenidoCSV], { type: 'text/csv;charset=utf-8;' });
    const link = document.createElement('a');
    const url = URL.createObjectURL(blob);
    
    link.setAttribute('href', url);
    link.setAttribute('download', `datos_esp32_${new Date().toISOString().split('T')[0]}.csv`);
    link.style.visibility = 'hidden';
    
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    URL.revokeObjectURL(url);
  };

  // Función alternativa para descargar como JSON
  const descargarHistorialJSON = () => {
    if (historial.length === 0) {
      alert('No hay datos para descargar');
      return;
    }

    const datosCompletos = {
      fechaExportacion: new Date().toISOString(),
      totalRegistros: historial.length,
      datos: historial
    };

    const contenidoJSON = JSON.stringify(datosCompletos, null, 2);
    const blob = new Blob([contenidoJSON], { type: 'application/json;charset=utf-8;' });
    const link = document.createElement('a');
    const url = URL.createObjectURL(blob);
    
    link.setAttribute('href', url);
    link.setAttribute('download', `datos_esp32_${new Date().toISOString().split('T')[0]}.json`);
    link.style.visibility = 'hidden';
    
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    URL.revokeObjectURL(url);
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-slate-900 via-blue-900 to-slate-900 text-white p-6">
      {/* Header */}
      <div className="max-w-7xl mx-auto mb-8">
        <div className="bg-slate-800/50 backdrop-blur rounded-lg p-6 border border-blue-500/30">
          <h1 className="text-3xl font-bold mb-4 flex items-center gap-3">
            <Activity className="w-8 h-8 text-blue-400" />
            Monitor ESP32 - Sistema Integrado de Sensores
          </h1>
          
          {/* Panel de Conexión */}
          <div className="flex items-center gap-4">
            <input
              type="text"
              value={ipAddress}
              onChange={(e) => setIpAddress(e.target.value)}
              placeholder="IP del ESP32"
              disabled={connected}
              className="px-4 py-2 bg-slate-700 rounded border border-slate-600 focus:border-blue-500 outline-none disabled:opacity-50"
            />
            <button
              onClick={connected ? desconectar : conectar}
              className={`px-6 py-2 rounded font-semibold flex items-center gap-2 transition-all ${
                connected 
                  ? 'bg-red-600 hover:bg-red-700' 
                  : 'bg-green-600 hover:bg-green-700'
              }`}
            >
              {connected ? (
                <>
                  <WifiOff className="w-5 h-5" />
                  Desconectar
                </>
              ) : (
                <>
                  <Wifi className="w-5 h-5" />
                  Conectar
                </>
              )}
            </button>
            <div className={`flex items-center gap-2 px-4 py-2 rounded ${
              connected ? 'bg-green-600/20 text-green-400' : 'bg-slate-700 text-slate-400'
            }`}>
              <div className={`w-3 h-3 rounded-full ${connected ? 'bg-green-400 animate-pulse' : 'bg-slate-500'}`} />
              {connected ? 'Conectado' : 'Desconectado'}
            </div>
          </div>
        </div>
      </div>

      {/* Paneles de Sensores */}
      <div className="max-w-7xl mx-auto grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 mb-8">
        
        {/* Caudal */}
        <div className="bg-slate-800/50 backdrop-blur rounded-lg p-6 border border-blue-500/30">
          <div className="flex items-center gap-3 mb-4">
            <Droplets className="w-6 h-6 text-blue-400" />
            <h2 className="text-xl font-bold">Caudal</h2>
          </div>
          <div className="space-y-2">
            <div className="text-3xl font-bold text-blue-400">
              {sensorData.caudal.flujo.toFixed(1)} <span className="text-lg">L/min</span>
            </div>
            <div className="text-slate-400">
              RPM: {sensorData.caudal.rpm.toFixed(1)}
            </div>
            <div className={`inline-block px-3 py-1 rounded text-sm font-semibold ${
              sensorData.caudal.estado === 'ALTO' ? 'bg-green-600' :
              sensorData.caudal.estado === 'MODERADO' ? 'bg-yellow-600' :
              'bg-blue-600'
            }`}>
              {sensorData.caudal.estado}
            </div>
          </div>
        </div>

        {/* Temperatura */}
        <div className="bg-slate-800/50 backdrop-blur rounded-lg p-6 border border-orange-500/30">
          <div className="flex items-center gap-3 mb-4">
            <Thermometer className="w-6 h-6 text-orange-400" />
            <h2 className="text-xl font-bold">Temperatura</h2>
          </div>
          <div className="space-y-2">
            <div className="text-3xl font-bold text-orange-400">
              {sensorData.temperatura.valor.toFixed(1)} <span className="text-lg">°C</span>
            </div>
            <div className={`inline-block px-3 py-1 rounded text-sm font-semibold ${
              sensorData.temperatura.estado === 'CALIENTE' ? 'bg-red-600' :
              sensorData.temperatura.estado === 'FRÍO' ? 'bg-blue-600' :
              'bg-green-600'
            }`}>
              {sensorData.temperatura.estado}
            </div>
          </div>
        </div>

        {/* Corriente */}
        <div className="bg-slate-800/50 backdrop-blur rounded-lg p-6 border border-yellow-500/30">
          <div className="flex items-center gap-3 mb-4">
            <Zap className="w-6 h-6 text-yellow-400" />
            <h2 className="text-xl font-bold">Corriente</h2>
          </div>
          <div className="space-y-2">
            <div className="text-3xl font-bold text-yellow-400">
              {sensorData.corriente.valor.toFixed(3)} <span className="text-lg">A RMS</span>
            </div>
            <div className="text-slate-400">Medición continua</div>
          </div>
        </div>

        {/* Turbidez */}
        <div className="bg-slate-800/50 backdrop-blur rounded-lg p-6 border border-cyan-500/30">
          <div className="flex items-center gap-3 mb-4">
            <CloudRain className="w-6 h-6 text-cyan-400" />
            <h2 className="text-xl font-bold">Turbidez</h2>
          </div>
          <div className="space-y-2">
            <div className={`text-2xl font-bold ${
              sensorData.turbidez.alerta ? 'text-red-400' : 'text-cyan-400'
            }`}>
              {sensorData.turbidez.estado}
            </div>
            {sensorData.turbidez.alerta && (
              <div className="flex items-center gap-2 text-red-400">
                <AlertTriangle className="w-5 h-5" />
                <span>Nivel por encima del umbral</span>
              </div>
            )}
          </div>
        </div>

        {/* Nivel */}
        <div className="bg-slate-800/50 backdrop-blur rounded-lg p-6 border border-purple-500/30">
          <div className="flex items-center gap-3 mb-4">
            <Activity className="w-6 h-6 text-purple-400" />
            <h2 className="text-xl font-bold">Nivel</h2>
          </div>
          <div className="space-y-2">
            <div className={`text-2xl font-bold ${
              sensorData.nivel.alerta ? 'text-red-400' : 'text-purple-400'
            }`}>
              {sensorData.nivel.estado}
            </div>
            {sensorData.nivel.alerta && (
              <div className="flex items-center gap-2 text-red-400 animate-pulse">
                <AlertTriangle className="w-5 h-5" />
                <span>ALARMA ACTIVADA</span>
              </div>
            )}
          </div>
        </div>
      </div>

      {/* Historial y Descargas */}
      <div className="max-w-7xl mx-auto">
        <div className="bg-slate-800/50 backdrop-blur rounded-lg p-6 border border-blue-500/30">
          <div className="flex justify-between items-center mb-4">
            <h2 className="text-xl font-bold">Historial de Lecturas</h2>
            
            {/* Botones de Descarga */}
            {historial.length > 0 && (
              <div className="flex gap-2">
                <button
                  onClick={descargarHistorial}
                  className="px-4 py-2 bg-green-600 hover:bg-green-700 rounded flex items-center gap-2 transition-all"
                >
                  <Download className="w-4 h-4" />
                  Descargar CSV
                </button>
                <button
                  onClick={descargarHistorialJSON}
                  className="px-4 py-2 bg-blue-600 hover:bg-blue-700 rounded flex items-center gap-2 transition-all"
                >
                  <Download className="w-4 h-4" />
                  Descargar JSON
                </button>
              </div>
            )}
          </div>
          
          <div className="overflow-x-auto">
            <div className="max-h-64 overflow-y-auto">
              {historial.length === 0 ? (
                <div className="text-slate-400 text-center py-8">
                  {connected ? 'Esperando datos...' : 'Conecta el ESP32 para ver el historial'}
                </div>
              ) : (
                <table className="w-full text-sm">
                  <thead className="bg-slate-700 sticky top-0">
                    <tr>
                      <th className="px-4 py-2 text-left">Hora</th>
                      <th className="px-4 py-2 text-left">Caudal</th>
                      <th className="px-4 py-2 text-left">Temp</th>
                      <th className="px-4 py-2 text-left">Corriente</th>
                      <th className="px-4 py-2 text-left">Turbidez</th>
                      <th className="px-4 py-2 text-left">Nivel</th>
                    </tr>
                  </thead>
                  <tbody>
                    {historial.slice().reverse().map((item, idx) => (
                      <tr key={idx} className="border-b border-slate-700 hover:bg-slate-700/50">
                        <td className="px-4 py-2">{item.timestamp}</td>
                        <td className="px-4 py-2">{item.caudal.flujo.toFixed(1)} L/min</td>
                        <td className="px-4 py-2">{item.temperatura.valor.toFixed(1)}°C</td>
                        <td className="px-4 py-2">{item.corriente.valor.toFixed(3)} A</td>
                        <td className="px-4 py-2">{item.turbidez.estado}</td>
                        <td className="px-4 py-2">{item.nivel.estado}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              )}
            </div>
          </div>
          
          {/* Información adicional */}
          {historial.length > 0 && (
            <div className="mt-4 text-slate-400 text-sm">
              <p>Total de registros: {historial.length} | Última actualización: {new Date().toLocaleTimeString()}</p>
            </div>
          )}
        </div>
      </div>
    </div>
  );
};

export default ESP32Monitor;
