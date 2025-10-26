<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>DeMergencias — Simulador y Emergencia</title>
<script src="https://cdn.tailwindcss.com"></script>
<script src="https://kit.fontawesome.com/a076d05399.js" crossorigin="anonymous"></script>
<style>
  html, body { height:100%; margin:0; font-family: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; }
  body { background-color: #f3f4f6; } /* fondo limpio */
  .big { font-size: 1.35rem; }
  .huge { font-size: 1.6rem; font-weight:700; }
  .card { background-color: #ffffff; padding: 20px; border-radius: 1rem; box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
  .btn { min-height:48px; font-weight:600; border-radius:12px; transition:all 0.2s ease-in-out; display:flex; align-items:center; justify-content:center; gap:8px; cursor:pointer; }
  .btn:hover { transform: scale(1.05); }
  .btn-primary { background-color:#3b82f6; color:white; }
  .btn-danger { background-color:#ef4444; color:white; }
  .btn-silent { background-color:#111827; color:white; }
  .btn-analyze { background-color:#f59e0b; color:white; }
  .chip { display:inline-block; padding:0.25rem 0.75rem; border-radius:9999px; background:#e5e7eb; font-size:0.875rem; }
  .muted { color:#6b7280; }
</style>
</head>
<body>
<div class="min-h-screen flex items-center justify-center p-6">
  <div class="max-w-4xl w-full grid grid-cols-1 md:grid-cols-3 gap-6">
    <!-- Left main card -->
    <div class="md:col-span-2 card">
      <div class="flex items-center justify-between mb-4">
        <div>
          <div class="huge">DeMergencias</div>
          <div class="muted big">Simulador · Botón Panic · Contactos · IA básica</div>
        </div>
        <div class="text-right">
          <div class="chip">Diseñado para jóvenes y familias</div>
          <div class="text-sm muted mt-1">Modo offline posible • Guardado local</div>
        </div>
      </div>

      <!-- Buttons -->
      <div class="grid grid-cols-1 sm:grid-cols-3 gap-3 mb-4">
        <button id="openSimulator" class="btn btn-primary big"><i class="fas fa-play"></i> SIMULADOR</button>
        <button id="panicVisible" class="btn btn-danger big"><i class="fas fa-exclamation-triangle"></i> ¡EMERGENCIA!</button>
        <button id="panicSilent" class="btn btn-silent big"><i class="fas fa-user-secret"></i> Panic Silencioso</button>
      </div>

      <!-- Simulator area -->
      <div id="simArea" class="mt-4 hidden">
        <h3 class="text-xl font-semibold mb-2">Simulador interactivo</h3>
        <div class="mb-3">
          <label class="block muted mb-1 big">Escenario</label>
          <select id="scenarioSelect" class="w-full p-3 rounded-lg border"></select>
        </div>
        <div id="scenarioBox" class="p-4 bg-gray-50 rounded-lg">
          <div id="scenarioText" class="big mb-3">Selecciona un escenario para comenzar.</div>
          <div id="scenarioOptions" class="space-y-3"></div>
          <div id="scenarioResult" class="mt-4"></div>
        </div>
      </div>

      <!-- Panic flow / status -->
      <div id="panicArea" class="mt-4 hidden">
        <h3 class="text-xl font-semibold mb-2">Estado de Emergencia</h3>
        <div id="panicStatus" class="p-3 bg-gray-50 rounded-lg big">Lista para usar.</div>
        <div class="mt-3 flex flex-col sm:flex-row gap-3">
          <button id="sendAlertNow" class="btn btn-primary px-4 py-2 big"><i class="fas fa-paper-plane"></i> Simular enviar alertas</button>
          <button id="downloadEvidence" class="btn btn-primary px-4 py-2 big"><i class="fas fa-download"></i> Descargar evidencia</button>
          <button id="copyAlert" class="btn btn-primary px-4 py-2 big"><i class="fas fa-copy"></i> Copiar alerta</button>
        </div>
      </div>

      <!-- AI / Detector -->
      <div class="mt-6">
        <h3 class="text-lg font-semibold mb-2">Detector rápido (IA básica gratis)</h3>
        <div class="text-sm muted mb-2">Analiza palabras clave para marcar riesgo.</div>
        <textarea id="textToAnalyze" placeholder="Pega aquí un mensaje para analizar..." class="w-full p-3 rounded-lg border h-28"></textarea>
        <div class="flex gap-3 mt-3 flex-col sm:flex-row">
          <button id="analyzeBtn" class="btn btn-analyze px-4 py-2 big"><i class="fas fa-magnifying-glass"></i> Analizar (rápido)</button>
          <button id="hfAnalyze" class="btn btn-analyze px-4 py-2 big"><i class="fas fa-robot"></i> Analizar Hugging Face</button>
        </div>
        <div id="analysisResult" class="mt-3 p-3 bg-gray-50 rounded-lg big"></div>

        <div class="mt-4 p-3 bg-white rounded-lg border">
          <label class="block muted mb-1 big">Token Hugging Face (opcional)</label>
          <input id="hfToken" class="w-full p-3 rounded-lg border" placeholder="Pega tu token aquí" />
          <div class="text-sm muted mt-1">Si lo usas, el análisis será más avanzado.</div>
        </div>
      </div>
    </div>

    <!-- Right: contacts & help -->
    <div class="card">
      <h3 class="text-lg font-semibold mb-2">Contactos de confianza</h3>
      <div id="contactsList" class="space-y-3 mb-3"></div>
      <div class="mb-3">
        <input id="cname" placeholder="Nombre" class="w-full p-3 rounded-lg border mb-2" />
        <input id="cemail" placeholder="Correo (recomendado)" class="w-full p-3 rounded-lg border mb-2" />
        <input id="cphone" placeholder="Teléfono (opcional)" class="w-full p-3 rounded-lg border mb-2" />
        <button id="addContact" class="btn btn-primary w-full big"><i class="fas fa-plus"></i> Añadir contacto</button>
      </div>
      <div class="mt-4">
        <h4 class="font-semibold">Instrucciones rápidas</h4>
        <ol class="list-decimal pl-5 mt-2 muted text-sm">
          <li>Agrega 2–4 contactos de confianza.</li>
          <li>Usá SIMULADOR para practicar decisiones.</li>
          <li>En emergencia: presioná EMERGENCIA o Panic silencioso.</li>
          <li>Al enviar alertas se abrirá tu cliente de correo o se copiará el texto listo.</li>
        </ol>
      </div>
    </div>
  </div>
</div>

<script>
// Todo tu JS original va aquí tal cual para mantener la funcionalidad
</script>
</body>
</html>
