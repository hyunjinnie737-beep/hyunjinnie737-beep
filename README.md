<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>DeMergencias — Simulador y Emergencia</title>
<!-- Tailwind CDN -->
<script src="https://cdn.tailwindcss.com"></script>
<style>
  html,body { height:100%; }
  body { font-family: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; 
         @apply bg-gradient-to-r from-indigo-50 via-pink-50 to-yellow-50; }
  .big { font-size: 1.35rem; }
  .huge { font-size: 1.6rem; font-weight:700; }
  .card { @apply bg-white/80 backdrop-blur p-5 rounded-2xl shadow-lg; }
  .danger { @apply bg-gradient-to-r from-red-500 to-rose-500 text-white font-bold; }
  .calm { @apply bg-gradient-to-r from-sky-400 to-indigo-500 text-white; }
  .muted { color:#6b7280; }
  .chip { @apply inline-block px-3 py-1 rounded-full text-sm bg-gray-100 border; }
  button { min-height:48px; @apply transition-all hover:scale-105; }
</style>
</head>
<body>
<div class="min-h-screen flex items-center justify-center p-6">
  <div class="max-w-4xl w-full grid grid-cols-1 md:grid-cols-3 gap-6">
    <!-- Left: main card -->
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
        <button id="openSimulator" class="calm rounded-xl big flex items-center justify-center gap-2">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 17v-6h6v6m2 4H7a2 2 0 01-2-2V7a2 2 0 012-2h5l2 2h5a2 2 0 012 2v10a2 2 0 01-2 2z" />
          </svg>
          SIMULADOR
        </button>

        <button id="panicVisible" class="danger rounded-xl big flex items-center justify-center gap-2">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 animate-pulse" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
          ¡EMERGENCIA!
        </button>

        <button id="panicSilent" class="bg-black text-white rounded-xl big flex items-center justify-center gap-2">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12H9m12 0a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
          Panic Silencioso
        </button>
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
        <div class="mt-3 flex gap-3">
          <button id="sendAlertNow" class="bg-emerald-500 text-white rounded-xl px-4 py-2 big">Simular enviar alertas</button>
          <button id="downloadEvidence" class="bg-indigo-600 text-white rounded-xl px-4 py-2 big">Descargar evidencia (ZIP)</button>
          <button id="copyAlert" class="bg-gray-700 text-white rounded-xl px-4 py-2 big">Copiar alerta</button>
        </div>
      </div>

      <!-- AI / Detector -->
      <div class="mt-6">
        <h3 class="text-lg font-semibold mb-2">Detector rápido (IA básica gratis)</h3>
        <div class="text-sm muted mb-2">Este detector es local, sin claves: usa palabras clave para marcar riesgo. Si querés IA externa (mejor), ingresá tu token de Hugging Face abajo.</div>
        <textarea id="textToAnalyze" placeholder="Pega aquí un mensaje para analizar..." class="w-full p-3 rounded-lg border h-28"></textarea>
        <div class="flex gap-3 mt-3">
          <button id="analyzeBtn" class="bg-yellow-500 text-black rounded-xl px-4 py-2 big">Analizar (rápido)</button>
          <button id="hfAnalyze" class="bg-sky-500 text-white rounded-xl px-4 py-2 big">Analizar con Hugging Face (opcional)</button>
        </div>
        <div id="analysisResult" class="mt-3 p-3 bg-gray-50 rounded-lg big"></div>

        <div class="mt-4 p-3 bg-white/60 rounded-lg border">
          <label class="block muted mb-1 big">Token Hugging Face (opcional, gratuita cuenta HF)</label>
          <input id="hfToken" class="w-full p-3 rounded-lg border" placeholder="Pega tu token aquí si querés usar su inference API" />
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
        <button id="addContact" class="w-full bg-indigo-500 text-white rounded-xl px-4 py-2 big">Añadir contacto</button>
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
// Aquí va TODO tu JS original sin cambios funcionales (simulador, panic, análisis de texto, contactos, HF...)
</script>
</body>
</html>

