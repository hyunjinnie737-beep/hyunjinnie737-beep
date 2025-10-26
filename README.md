<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>DeMergencias — Simulador y Emergencia</title>
<!-- Tailwind CDN -->
<script src="https://cdn.tailwindcss.com"></script>
<style>
  /* Ajustes visuales extras */
  html,body { height:100%; }
  body { font-family: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; }
  .big { font-size: 1.35rem; } /* texto grande para jóvenes/niños */
  .huge { font-size: 1.6rem; font-weight:700; }
  .card { @apply bg-white/80 backdrop-blur p-5 rounded-2xl shadow-lg; }
  .danger { @apply bg-gradient-to-r from-red-500 to-rose-500 text-white font-bold; }
  .calm { @apply bg-gradient-to-r from-sky-400 to-indigo-500 text-white; }
  .muted { color:#6b7280; }
  .chip { @apply inline-block px-3 py-1 rounded-full text-sm bg-gray-100 border; }
  /* ensure big tap targets */
  button { min-height:48px; }
</style>
</head>
<body class="bg-[url('https://images.unsplash.com/photo-1507679799987-c73779587ccf?q=80&w=1400&auto=format&fit=crop&ixlib=rb-4.0.3&s=1a9e1a9e7f3b5a0f1a1b6f1b7d2b3f2c')] bg-cover bg-center">
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
          <button id="openSimulator" class="calm rounded-xl big">SIMULADOR</button>
          <button id="panicVisible" class="danger rounded-xl big">¡EMERGENCIA! (visible)</button>
          <button id="panicSilent" class="bg-black text-white rounded-xl big">Panic Silencioso</button>
        </div>

        <!-- Simulator area -->
        <div id="simArea" class="mt-4 hidden">
          <h3 class="text-xl font-semibold mb-2">Simulador interactivo</h3>
          <div class="mb-3">
            <label class="block muted mb-1 big">Escenario</label>
            <select id="scenarioSelect" class="w-full p-3 rounded-lg border">
              <!-- options populated by JS -->
            </select>
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
            <input id="hfToken" class="w-full p-3 rounded-lg border" placeholder="Pega tu token de Hugging Face aquí si querés usar su inference API (opcional)" />
            <div class="text-sm muted mt-1">Si lo usas el análisis será más avanzado pero depende del modelo y límites de HF.</div>
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

        <div class="mt-4">
          <h4 class="font-semibold">Notas técnicas</h4>
          <p class="muted text-sm">Al abrir el archivo haciendo doble-click, todo funciona. Algunas funciones (ubicación precisa / audio) funcionan mejor si servís el archivo por HTTP (instrucciones abajo).</p>
        </div>
      </div>

    </div>
  </div>

<script>
/* -----------------------
   Datos: escenarios
   ----------------------- */
const SCENARIOS = [
  {
    id: 'acoso_chat',
    title: 'Acoso en chat (insultos y presión)',
    intro: 'Alguien te escribe insultos repetidos y quiere que envíes fotos. ¿Qué haces?',
    steps: [
      {
        text: 'Recibís un mensaje pidiéndote fotos privadas. ¿Qué hacés?',
        options: [
          { id:'send', text:'Enviar las fotos', score:-3, explain:'Compartir fotos privadas puede usarse en tu contra.' },
          { id:'ignore', text:'Ignorar y bloquear', score:+2, explain:'Bloquear reduce contacto y te protege.' },
          { id:'ask', text:'Pedir tiempo y pensar', score:0, explain:'Ganas tiempo, pero la mejor es bloquear.' }
        ]
      },
      {
        text: 'Ves que sigue insistiendo con amenazas. ¿Qué hacés?',
        options: [
          { id:'confront', text:'Enfrentar con insultos', score:-2, explain:'Puede escalar la situación.' },
          { id:'report', text:'Guardar evidencia y reportar', score:+3, explain:'Registrar y reportar es lo correcto.' },
          { id:'meet', text:'Quedar para una reunión', score:-5, explain:'Nunca quedar con desconocidos.' }
        ]
      }
    ]
  },
  {
    id: 'robo_calle',
    title: 'Intento de robo en la calle',
    intro: 'Alguien intenta quitarte la mochila mientras vas en la calle.',
    steps: [
      {
        text: 'Sospechas que quieren robarte. ¿Qué hacés?',
        options: [
          { id:'run', text:'Correr y gritar', score:+1, explain:'A veces útil; prioriza tu seguridad.' },
          { id:'entregar', text:'Entregar los objetos', score:+2, explain:'Perder cosas es preferible a perder la vida.' },
          { id:'enfrentar', text:'Enfrentar al ladrón', score:-3, explain:'No recomendable; puede ser peligroso.' }
        ]
      },
      {
        text: 'Lográs alejarte, ¿qué seguís?',
        options: [
          { id:'llamar', text:'Llamar a un contacto de confianza', score:+3, explain:'Informar ayuda y pruebas.' },
          { id:'nada', text:'Seguir caminando sin avisar', score:-1, explain:'Mejor avisar y buscar apoyo.' }
        ]
      }
    ]
  },
  {
    id: 'grooming_online',
    title: 'Grooming / adulto sospechoso en línea',
    intro: 'Un adulto comienza a hablarte de forma romántica y pide contacto fuera de la app.',
    steps: [
      {
        text: 'El adulto pide hablar por otra app y te halaga. ¿Qué haces?',
        options: [
          { id:'seguir', text:'Seguir la conversación', score:-4, explain:'Peligroso con adultos desconocidos.' },
          { id:'contar', text:'Contarle a un adulto de confianza', score:+4, explain:'Fundamental informarlo.' },
          { id:'bloquear', text:'Bloquear y reportar', score:+2, explain:'Protección inmediata.' }
        ]
      },
      {
        text: 'Te pide fotos o encuentros. ¿Qué hacés?',
        options: [
          { id:'aceptar', text:'Aceptar', score:-5, explain:'NUNCA aceptes.' },
          { id:'reportar', text:'Guardar evidencia y reportar', score:+3, explain:'Acción correcta.' }
        ]
      }
    ]
  }
];

/* -----------------------
   Utilidades de LocalStorage (contacts, evidence)
   ----------------------- */
const CONTACTS_KEY = 'dm_contacts_v1';
const EVIDENCE_KEY = 'dm_evidence_v1';

function loadContacts(){
  try{
    return JSON.parse(localStorage.getItem(CONTACTS_KEY) || '[]');
  }catch(e){ return []; }
}
function saveContacts(list){ localStorage.setItem(CONTACTS_KEY, JSON.stringify(list)); }

function loadEvidence(){
  try { return JSON.parse(localStorage.getItem(EVIDENCE_KEY) || '[]'); }catch(e){ return []; }
}
function saveEvidence(list){ localStorage.setItem(EVIDENCE_KEY, JSON.stringify(list)); }

/* -----------------------
   DOM references
   ----------------------- */
const openSimulator = document.getElementById('openSimulator');
const simArea = document.getElementById('simArea');
const panicArea = document.getElementById('panicArea');
const panicVisible = document.getElementById('panicVisible');
const panicSilent = document.getElementById('panicSilent');
const sendAlertNow = document.getElementById('sendAlertNow');
const downloadEvidence = document.getElementById('downloadEvidence');
const copyAlert = document.getElementById('copyAlert');
const scenarioSelect = document.getElementById('scenarioSelect');
const scenarioText = document.getElementById('scenarioText');
const scenarioOptions = document.getElementById('scenarioOptions');
const scenarioResult = document.getElementById('scenarioResult');
const contactsList = document.getElementById('contactsList');
const addContact = document.getElementById('addContact');
const cname = document.getElementById('cname');
const cemail = document.getElementById('cemail');
const cphone = document.getElementById('cphone');
const panicStatus = document.getElementById('panicStatus');
const textToAnalyze = document.getElementById('textToAnalyze');
const analyzeBtn = document.getElementById('analyzeBtn');
const analysisResult = document.getElementById('analysisResult');
const hfTokenInput = document.getElementById('hfToken');
const hfAnalyze = document.getElementById('hfAnalyze');

/* -----------------------
   Inicialización UI
   ----------------------- */
function renderScenarios(){
  scenarioSelect.innerHTML = '';
  SCENARIOS.forEach(s=>{
    const opt = document.createElement('option');
    opt.value = s.id; opt.textContent = s.title;
    scenarioSelect.appendChild(opt);
  });
}
function renderContacts(){
  const list = loadContacts();
  contactsList.innerHTML = '';
  if(list.length===0){
    contactsList.innerHTML = '<div class="muted">No hay contactos. Añadí 2–4 personas de confianza.</div>';
    return;
  }
  list.forEach((c,i)=>{
    const div = document.createElement('div');
    div.className='flex items-center justify-between p-2 border rounded-lg';
    div.innerHTML = `<div>
        <div class="font-semibold">${escapeHtml(c.name)}</div>
        <div class="text-sm muted">${escapeHtml(c.email)} ${c.phone?('• '+escapeHtml(c.phone)):''}</div>
      </div>
      <div class="flex gap-2">
        <button data-i="${i}" class="bg-red-500 text-white px-3 py-1 rounded-lg small">Eliminar</button>
      </div>`;
    contactsList.appendChild(div);
    div.querySelector('button').addEventListener('click',()=>{ removeContact(i); });
  });
}

function removeContact(i){
  const list = loadContacts();
  list.splice(i,1);
  saveContacts(list);
  renderContacts();
}

/* -----------------------
   Simulador: motor simple
   ----------------------- */
let currentScenario = null;
let currentStepIndex = 0;
let currentScore = 0;

function startScenario(id){
  const s = SCENARIOS.find(x=>x.id===id);
  if(!s) return;
  currentScenario = s;
  currentStepIndex = 0;
  currentScore = 0;
  showStep();
  simArea.classList.remove('hidden');
  panicArea.classList.add('hidden');
}

function showStep(){
  const s = currentScenario;
  const step = s.steps[currentStepIndex];
  scenarioText.textContent = step.text;
  scenarioOptions.innerHTML = '';
  step.options.forEach(opt=>{
    const btn = document.createElement('button');
    btn.className='w-full text-left p-3 border rounded-lg big';
    btn.textContent = opt.text;
    btn.addEventListener('click', ()=> chooseOption(opt) );
    scenarioOptions.appendChild(btn);
  });
  scenarioResult.innerHTML = `<div class="muted text-sm">${s.intro}</div>`;
}

function chooseOption(opt){
  currentScore += opt.score;
  const explain = document.createElement('div');
  explain.className='mt-3 p-3 bg-white rounded-lg border';
  explain.innerHTML = `<div class="font-semibold">Consecuencia</div><div class="muted">${escapeHtml(opt.explain)}</div>`;
  scenarioResult.innerHTML = '';
  scenarioResult.appendChild(explain);

  // next step or finish
  currentStepIndex++;
  setTimeout(()=>{
    if(currentStepIndex >= currentScenario.steps.length){
      finishScenario();
    } else {
      showStep();
    }
  }, 700);
}

function finishScenario(){
  const summary = document.createElement('div');
  summary.className='p-3 mt-3 rounded-lg';
  const risk = currentScore <= -3 ? 'ALTO' : currentScore <= 0 ? 'MEDIO' : 'BAJO';
  const color = risk==='ALTO' ? 'text-red-600' : risk==='MEDIO' ? 'text-yellow-600' : 'text-green-600';
  summary.innerHTML = `<div class="font-semibold">Resultado: <span class="${color}">${risk}</span></div>
    <div class="muted mt-2">Puntuación: ${currentScore}</div>
    <div class="mt-2">Consejos: ${risk==='ALTO' ? 'Evita confrontar. Busca a un adulto y guarda evidencia.' : risk==='MEDIO' ? 'Refuerza seguridad y reporta.' : 'Buen manejo. Mantén vigilancia.'}</div>`;
  scenarioResult.appendChild(summary);
}

/* -----------------------
   Panic: preparar alerta
   ----------------------- */
function prepareAlert({silent=false}){
  const contacts = loadContacts();
  if(contacts.length===0){
    panicStatus.textContent = 'No hay contactos configurados. Añade al menos uno en la columna derecha.';
    return {ok:false, message:'no_contacts'};
  }
  panicStatus.textContent = (silent ? 'Panic silencioso preparado...' : 'Alerta visible preparada...');
  // collect location (best-effort)
  return new Promise(async (resolve)=>{
    let lat=null,lng=null;
    if('geolocation' in navigator){
      navigator.geolocation.getCurrentPosition(p=>{
        lat = p.coords.latitude; lng = p.coords.longitude;
        resolve({ok:true, contacts, lat, lng});
      }, err=>{
        // can't get location
        resolve({ok:true, contacts, lat:null, lng:null});
      }, {timeout:3500});
    } else {
      resolve({ok:true, contacts, lat:null, lng:null});
    }
  });
}

function composeAlertPayload({contacts, lat, lng, silent=false}){
  const userLabel = 'Usuario (demo)';
  const now = new Date().toLocaleString();
  const map = lat && lng ? `https://www.google.com/maps?q=${lat},${lng}` : 'Ubicación no disponible';
  const text = `ALERTA DE EMERGENCIA
Usuario: ${userLabel}
Fecha: ${now}
Tipo: ${silent ? 'Silenciosa' : 'Visible'}
Ubicación: ${map}
Mensaje: Necesito ayuda URGENTE.
\n\nNota: este mensaje fue generado por la app DeMergencias.`;
  return {text, map, contacts, now, lat, lng};
}

/* "Enviar" alertas: en este demo abrimos mailto, copiamos texto, y guardamos evidencia local */
async function sendAlert({silent=false}){
  const prep = await prepareAlert({silent});
  if(!prep.ok) return;
  const payload = composeAlertPayload({contacts:prep.contacts, lat:prep.lat, lng:prep.lng, silent});
  // Save evidence entry
  const evidence = loadEvidence();
  evidence.push({id:'ev_'+Date.now(), payload, created:Date.now()});
  saveEvidence(evidence);
  panicStatus.textContent = 'Alerta preparada. Se abrirán mails (si están configurados) y se copiará el texto al portapapeles.';
  // For each contact, open mailto (best-effort)
  prep.contacts.forEach((c,i)=>{
    if(c.email){
      const subject = encodeURIComponent('ALERTA: necesito ayuda');
      const body = encodeURIComponent(payload.text + '\n\nContacto: ' + c.name);
      const mailto = `mailto:${encodeURIComponent(c.email)}?subject=${subject}&body=${body}`;
      // open in new tab to prompt email client
      window.open(mailto, '_blank');
    }
  });
  // copy to clipboard
  try{
    await navigator.clipboard.writeText(payload.text);
  }catch(e){}
  // show small visual
  setTimeout(()=>{ alert('Alerta preparada: mails abiertos (si tu cliente está configurado) y texto copiado.'); }, 200);
}

/* -----------------------
   Evidence: download as JSON (simple)
   ----------------------- */
function downloadEvidenceFile(){
  const evid = loadEvidence();
  if(evid.length===0){ alert('No hay evidencia guardada.'); return; }
  const blob = new Blob([JSON.stringify(evid, null, 2)], {type:'application/json'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href=url; a.download = 'evidencia_demergencias_'+Date.now()+'.json';
  document.body.appendChild(a); a.click(); a.remove();
  URL.revokeObjectURL(url);
}

/* -----------------------
   Simple keyword-based detector (local)
   ----------------------- */
const RISK_WORDS = {
  alta: ['matar','muerte','violar','atentar','te mato','te voy a matar','violación','suicidio','mata','disparar','bomba'],
  media: ['idiota','estúpido','tonto','tonta','mierda','imbécil','insulto','humillar','amenaza'],
  grooming: ['foto íntima','foto privada','envía foto','conocerte fuera','nuevo contacto','ven a verme']
};

function analyzeTextLocal(text){
  const t = text.toLowerCase();
  let score = 0;
  const hits = [];
  Object.keys(RISK_WORDS).forEach(k=>{
    RISK_WORDS[k].forEach(w=>{
      if(t.includes(w)){
        hits.push({level:k,word:w});
        if(k==='alta') score -= 3;
        if(k==='media') score -= 1;
        if(k==='grooming') score -= 2;
      }
    });
  });
  let severity = 'Bajo';
  if(score <= -4) severity = 'Alto';
  else if(score <= -2) severity = 'Medio';
  return {severity, score, hits};
}

/* -----------------------
   Hugging Face inference (opcional): uses user-provided token
   Model used: text-classification/bert-base-uncased or similar (example)
   NOTE: user must provide token with inference permission.
   ----------------------- */
async function analyzeWithHugging(text, token){
  if(!token) throw new Error('No token');
  // Example endpoint: using a text-classification model hosted on HF.
  // We'll call "https://api-inference.huggingface.co/models/{model}"
  // For general toxicity you can try "unitary/toxic-bert" or other models on HF hub.
  const model = 'unitary/toxic-bert'; // example; may vary
  const url = `https://api-inference.huggingface.co/models/${model}`;
  const res = await fetch(url, {
    method:'POST',
    headers: { Authorization: 'Bearer ' + token, 'Content-Type':'application/json' },
    body: JSON.stringify({ inputs: text })
  });
  if(res.status>=400){
    const txt = await res.text();
    throw new Error('HF error: '+txt);
  }
  const json = await res.json();
  return json;
}

/* -----------------------
   Helpers & events wiring
   ----------------------- */
function escapeHtml(s){ return String(s).replaceAll('<','&lt;').replaceAll('>','&gt;'); }

openSimulator.addEventListener('click', ()=> {
  simArea.classList.toggle('hidden');
  panicArea.classList.add('hidden');
});

panicVisible.addEventListener('click', async ()=>{
  simArea.classList.add('hidden');
  panicArea.classList.remove('hidden');
  panicStatus.textContent = 'Preparando alerta visible...';
  const data = await prepareAlert({silent:false});
  if(data.ok) panicStatus.textContent = 'Lista: pulsa "Simular enviar alertas" para abrir mails / copiar texto.';
});

panicSilent.addEventListener('click', async ()=>{
  simArea.classList.add('hidden');
  panicArea.classList.remove('hidden');
  panicStatus.textContent = 'Preparando panic silencioso... (3 taps también puede activar)';
  const data = await prepareAlert({silent:true});
  if(data.ok) panicStatus.textContent = 'Panic silencioso listo: pulsa "Simular enviar alertas".';
});

sendAlertNow.addEventListener('click', async ()=>{
  // ask whether we want silent
  const silent = confirm('¿Enviar como silencioso? (Aceptar = sí, Cancelar = visible)');
  await sendAlert({silent});
});

downloadEvidence.addEventListener('click', ()=> downloadEvidenceFile());
copyAlert.addEventListener('click', async ()=>{
  const evidence = loadEvidence();
  if(evidence.length===0){ alert('No hay evidencia.'); return; }
  try { await navigator.clipboard.writeText(JSON.stringify(evidence[evidence.length-1], null, 2)); alert('Última evidencia copiada al portapapeles.'); }
  catch(e){ alert('No se pudo copiar.'); }
});

renderScenarios();
renderContacts();

scenarioSelect.addEventListener('change', ()=> startScenario(scenarioSelect.value));
document.addEventListener('DOMContentLoaded', ()=> {
  if(SCENARIOS.length>0) startScenario(SCENARIOS[0].id);
});

/* Contacts events */
addContact.addEventListener('click', ()=>{
  const name = cname.value.trim(), email = cemail.value.trim(), phone = cphone.value.trim();
  if(!name || (!email && !phone)){ alert('Poné al menos nombre y correo o teléfono.'); return; }
  const list = loadContacts();
  list.push({name, email, phone});
  saveContacts(list);
  cname.value=''; cemail.value=''; cphone.value='';
  renderContacts();
});

/* Analysis local */
analyzeBtn.addEventListener('click', ()=>{
  const txt = textToAnalyze.value.trim();
  if(!txt){ analysisResult.innerHTML = '<div class="muted">Ingresa texto para analizar.</div>'; return; }
  const r = analyzeTextLocal(txt);
  let html = `<div class="font-semibold">Severidad: ${r.severity} • score ${r.score}</div>`;
  if(r.hits.length>0){
    html += '<div class="mt-2 muted">Palabras detectadas:</div><ul class="mt-1">';
    r.hits.forEach(h=> html += `<li>${escapeHtml(h.word)} (${h.level})</li>`);
    html += '</ul>';
  } else html += '<div class="muted mt-2">No se detectaron palabras de riesgo en el análisis rápido.</div>';
  analysisResult.innerHTML = html;
});

/* Hugging Face analyze */
hfAnalyze.addEventListener('click', async ()=>{
  const txt = textToAnalyze.value.trim();
  const token = hfTokenInput.value.trim();
  if(!txt){ alert('Pega el texto primero.'); return; }
  if(!token){ alert('Si querés usar Hugging Face, pegá tu token. Si no, usá el analizador rápido.'); return; }
  analysisResult.innerHTML = 'Analizando con Hugging Face... (puede tardar unos segundos)';
  try{
    const res = await analyzeWithHugging(txt, token);
    analysisResult.innerHTML = '<div class="font-semibold">Resultado (raw):</div><pre class="mt-2 p-2 bg-gray-100 rounded">'+escapeHtml(JSON.stringify(res, null, 2))+'</pre>';
  }catch(e){
    analysisResult.innerHTML = '<div class="text-red-600">Error: '+escapeHtml(e.message)+'</div>';
  }
});

/* load saved evidence on start (no UI list for brevity) */
if(!localStorage.getItem(EVIDENCE_KEY)) saveEvidence([]);

/* Warn if some APIs won't work on file:// */
(function(){
  // Many browsers block geolocation and media over file:// or without HTTPS.
  const insecureNotice = document.createElement('div');
  insecureNotice.className = 'mt-3 p-2 text-sm muted';
  insecureNotice.innerHTML = '<strong>Atención:</strong> Algunas funciones (geolocalización precisa, micrófono, cámara) funcionan mejor si servís el archivo por HTTP/HTTPS. Si no sabés cómo, en las instrucciones abajo tenés el comando para hacerlo rápido en tu PC.';
  document.querySelector('.card').appendChild(insecureNotice);
})();

/* -----------------------
   Final helper: quick server instructions shown to user
   ----------------------- */
(function addFooterInstructions(){
  const footer = document.createElement('div');
  footer.className='mt-6 text-sm muted';
  footer.innerHTML = `<h4 class="font-semibold">Cómo obtener permisos completos (opcional)</h4>
  <p>Si querés que la geolocalización y el acceso al micrófono funcionen sin restricciones, serví este archivo por HTTP desde tu PC:</p>
  <pre class="mt-2 p-3 bg-gray-100 rounded">Python 3 (uno de estos comandos):
# En macOS/Linux
python3 -m http.server 8000
# En Windows (PowerShell)
python -m http.server 8000
# Luego abrí en el navegador: http://localhost:8000/demergencias.html</pre>
  <p>También podés usar <code>npx serve</code> o cualquier servidor estático. Esto es seguro y gratuito.</p>
  <p class="mt-2">Para IA exterior: <strong>Hugging Face</strong> ofrece tokens gratis (plan básico). Pegá tu token en el campo correspondiente para análisis con modelos de clasificación.</p>`;
  document.querySelector('body').appendChild(footer);
})();

</script>
</body>
</html>
