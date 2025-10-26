<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>DeMergencias — Simulador y Emergencia</title>
<script src="https://cdn.tailwindcss.com"></script>
<style>
  html, body { height: 100%; margin:0; font-family: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; background: linear-gradient(135deg, #f0f4ff, #d9f0ff); }
  .big { font-size: 1.35rem; }
  .huge { font-size: 1.6rem; font-weight:700; }
  .card { background: rgba(255,255,255,0.85); backdrop-blur:10px; padding:20px; border-radius:1rem; box-shadow: 0 10px 25px rgba(0,0,0,0.1); }
  .danger { background: linear-gradient(to right, #f43f5e, #f87171); color:white; font-bold; transition: transform 0.2s; }
  .danger:hover { transform: scale(1.05); }
  .calm { background: linear-gradient(to right, #3b82f6, #6366f1); color:white; transition: transform 0.2s; }
  .calm:hover { transform: scale(1.05); }
  .silent { background: linear-gradient(to right, #000000, #374151); color:white; transition: transform 0.2s; }
  .silent:hover { transform: scale(1.05); }
  .chip { display:inline-block; padding:0.25rem 0.75rem; border-radius:9999px; background:#e5e7eb; border:1px solid #d1d5db; font-size:0.875rem; }
  button { min-height:48px; font-weight:500; }
</style>
</head>
<body>
<div class="min-h-screen flex items-center justify-center p-4">
  <div class="max-w-5xl w-full grid grid-cols-1 md:grid-cols-3 gap-6">
    
    <!-- Main Card -->
    <div class="md:col-span-2 card flex flex-col gap-4">
      <div class="flex justify-between items-center">
        <div>
          <div class="huge">DeMergencias</div>
          <div class="big text-gray-500">Simulador · Botón Panic · Contactos · IA básica</div>
        </div>
        <div class="text-right">
          <div class="chip">Para jóvenes y familias</div>
          <div class="text-sm text-gray-500 mt-1">Modo offline • Guardado local</div>
        </div>
      </div>

      <!-- Buttons -->
      <div class="grid grid-cols-1 sm:grid-cols-3 gap-3">
        <button id="openSimulator" class="calm rounded-xl big">SIMULADOR</button>
        <button id="panicVisible" class="danger rounded-xl big">¡EMERGENCIA! (visible)</button>
        <button id="panicSilent" class="silent rounded-xl big">Panic Silencioso</button>
      </div>

      <!-- Simulador -->
      <div id="simArea" class="hidden flex flex-col gap-4">
        <h3 class="text-xl font-semibold">Simulador interactivo</h3>
        <select id="scenarioSelect" class="w-full p-3 rounded-lg border"></select>
        <div class="p-4 bg-gray-50 rounded-lg">
          <div id="scenarioText" class="big mb-3">Selecciona un escenario.</div>
          <div id="scenarioOptions" class="space-y-3"></div>
          <div id="scenarioResult" class="mt-4"></div>
        </div>
      </div>

      <!-- Panic -->
      <div id="panicArea" class="hidden flex flex-col gap-3">
        <h3 class="text-xl font-semibold">Estado de Emergencia</h3>
        <div id="panicStatus" class="p-3 bg-gray-50 rounded-lg big">Lista para usar.</div>
        <div class="flex flex-wrap gap-3">
          <button id="sendAlertNow" class="bg-emerald-500 text-white rounded-xl px-4 py-2 big hover:scale-105 transition">Simular alertas</button>
          <button id="downloadEvidence" class="bg-indigo-600 text-white rounded-xl px-4 py-2 big hover:scale-105 transition">Descargar evidencia</button>
          <button id="copyAlert" class="bg-gray-700 text-white rounded-xl px-4 py-2 big hover:scale-105 transition">Copiar alerta</button>
        </div>
      </div>

      <!-- Detector IA -->
      <div class="mt-4 flex flex-col gap-2">
        <h3 class="text-lg font-semibold">Detector rápido (IA local)</h3>
        <textarea id="textToAnalyze" placeholder="Pega aquí un mensaje..." class="w-full p-3 rounded-lg border h-28"></textarea>
        <div class="flex gap-3">
          <button id="analyzeBtn" class="bg-yellow-500 text-black rounded-xl px-4 py-2 big hover:scale-105 transition">Analizar (rápido)</button>
          <button id="hfAnalyze" class="bg-sky-500 text-white rounded-xl px-4 py-2 big hover:scale-105 transition">Analizar HF</button>
        </div>
        <div id="analysisResult" class="mt-3 p-3 bg-gray-50 rounded-lg big"></div>
        <input id="hfToken" placeholder="Token Hugging Face (opcional)" class="w-full p-3 rounded-lg border" />
      </div>
    </div>

    <!-- Contactos -->
    <div class="card flex flex-col gap-4">
      <h3 class="text-lg font-semibold">Contactos de confianza</h3>
      <div id="contactsList" class="space-y-3"></div>
      <input id="cname" placeholder="Nombre" class="w-full p-3 rounded-lg border" />
      <input id="cemail" placeholder="Correo" class="w-full p-3 rounded-lg border" />
      <input id="cphone" placeholder="Teléfono (opcional)" class="w-full p-3 rounded-lg border" />
      <button id="addContact" class="w-full bg-indigo-500 text-white rounded-xl px-4 py-2 big hover:scale-105 transition">Añadir contacto</button>
      <div class="mt-2 text-gray-500 text-sm">
        <ol class="list-decimal pl-5">
          <li>Agrega 2–4 contactos.</li>
          <li>Usá SIMULADOR para practicar decisiones.</li>
          <li>En emergencia: presioná EMERGENCIA o Panic silencioso.</li>
        </ol>
      </div>
    </div>

  </div>
</div>

<script>
const SCENARIOS = [
  { id:'acoso_chat', title:'Acoso en chat', intro:'Alguien te escribe insultos...', steps:[
    { text:'Recibís mensaje pidiendo fotos', options:[
      {id:'send', text:'Enviar fotos', score:-3, explain:'No enviar.'},
      {id:'ignore', text:'Ignorar y bloquear', score:2, explain:'Protege.'},
      {id:'ask', text:'Pedir tiempo', score:0, explain:'Ganas tiempo.'}
    ]},
    { text:'Sigue insistiendo', options:[
      {id:'confront', text:'Enfrentar', score:-2, explain:'Puede escalar.'},
      {id:'report', text:'Guardar evidencia y reportar', score:3, explain:'Correcto.'}
    ]}
  ]},
  { id:'robo_calle', title:'Intento de robo', intro:'Alguien intenta quitarte la mochila', steps:[
    { text:'Sospechas robo', options:[
      {id:'run', text:'Correr', score:1, explain:'Seguridad primero.'},
      {id:'entregar', text:'Entregar objetos', score:2, explain:'Evita peligro.'}
    ]}
  ]}
];

const CONTACTS_KEY='dm_contacts_v1';
const EVIDENCE_KEY='dm_evidence_v1';

function loadContacts(){ return JSON.parse(localStorage.getItem(CONTACTS_KEY)||'[]'); }
function saveContacts(list){ localStorage.setItem(CONTACTS_KEY,JSON.stringify(list)); }
function loadEvidence(){ return JSON.parse(localStorage.getItem(EVIDENCE_KEY)||'[]'); }
function saveEvidence(list){ localStorage.setItem(EVIDENCE_KEY,JSON.stringify(list)); }

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

/* Inicialización */
function renderScenarios(){
  scenarioSelect.innerHTML='';
  SCENARIOS.forEach(s=>{
    const opt=document.createElement('option');
    opt.value=s.id; opt.textContent=s.title;
    scenarioSelect.appendChild(opt);
  });
}
function renderContacts(){
  const list=loadContacts();
  contactsList.innerHTML='';
  if(list.length===0){ contactsList.innerHTML='<div class="text-gray-500">No hay contactos.</div>'; return; }
  list.forEach((c,i)=>{
    const div=document.createElement('div');
    div.className='flex items-center justify-between p-2 border rounded-lg';
    div.innerHTML=`<div><div class="font-semibold">${c.name}</div><div class="text-sm text-gray-500">${c.email} ${c.phone?('• '+c.phone):''}</div></div>
    <div><button data-i="${i}" class="bg-red-500 text-white px-3 py-1 rounded-lg">Eliminar</button></div>`;
    contactsList.appendChild(div);
    div.querySelector('button').addEventListener('click',()=>{ removeContact(i); });
  });
}
function removeContact(i){ const list=loadContacts(); list.splice(i,1); saveContacts(list); renderContacts(); }

/* Simulador */
let currentScenario=null, currentStepIndex=0, currentScore=0;
function startScenario(id){ const s=SCENARIOS.find(x=>x.id===id); if(!s) return; currentScenario=s; currentStepIndex=0; currentScore=0; showStep(); simArea.classList.remove('hidden'); panicArea.classList.add('hidden'); }
function showStep(){ const step=currentScenario.steps[currentStepIndex]; scenarioText.textContent=step.text; scenarioOptions.innerHTML=''; step.options.forEach(opt=>{ const btn=document.createElement('button'); btn.className='w-full text-left p-3 border rounded-lg'; btn.textContent=opt.text; btn.addEventListener('click',()=>chooseOption(opt)); scenarioOptions.appendChild(btn); }); scenarioResult.innerHTML=`<div class="text-gray-500 text-sm">${currentScenario.intro}</div>`; }
function chooseOption(opt){ currentScore+=opt.score; scenarioResult.innerHTML=`<div class="p-3 border rounded-lg"><div class="font-semibold">Consecuencia</div><div class="text-gray-500">${opt.explain}</div></div>`; currentStepIndex++; setTimeout(()=>{ if(currentStepIndex>=currentScenario.steps.length){ finishScenario(); }else showStep(); },700); }
function finishScenario(){ const summary=document.createElement('div'); summary.className='p-3 mt-3 rounded-lg'; const risk=currentScore<=-3?'ALTO':currentScore<=0?'MEDIO':'BAJO'; const color=risk==='ALTO'?'text-red-600':risk==='MEDIO'?'text-yellow-600':'text-green-600'; summary.innerHTML=`<div class="font-semibold">Resultado: <span class="${color}">${risk}</span></div><div class="text-gray-500 mt-2">Puntuación: ${currentScore}</div><div class="mt-2">Consejos: ${risk==='ALTO'?'Evita confrontar. Busca un adulto y guarda evidencia.':risk==='MEDIO'?'Refuerza seguridad y reporta.':'Buen manejo. Mantén vigilancia.'}</div>`; scenarioResult.appendChild(summary); }

/* Panic */
async function prepareAlert({silent=false}){ const contacts=loadContacts(); if(contacts.length===0){ panicStatus.textContent='No hay contactos'; return {ok:false}; } panicStatus.textContent=silent?'Panic silencioso preparado':'Alerta visible preparada'; return {ok:true, contacts}; }
function composeAlertPayload({contacts, silent=false}){ const now=new Date().toLocaleString(); const text=`ALERTA DE EMERGENCIA\nUsuario (demo)\nFecha: ${now}\nTipo: ${silent?'Silenciosa':'Visible'}\nMensaje: Necesito ayuda URGENTE.`; return {text, contacts, now}; }
async function sendAlert({silent=false}){ const prep=await prepareAlert({silent}); if(!prep.ok) return; const payload=composeAlertPayload({contacts:prep.contacts, silent}); const evidence=loadEvidence(); evidence.push({id:'ev_'+Date.now(), payload, created:Date.now()}); saveEvidence(evidence); panicStatus.textContent='Alerta lista. Revisá mails o portapapeles'; prep.contacts.forEach(c=>{ if(c.email){ const mailto=`mailto:${encodeURIComponent(c.email)}?subject=ALERTA&body=${encodeURIComponent(payload.text)}`; window.open(mailto,'_blank'); } }); try{ await navigator.clipboard.writeText(payload.text); }catch(e){}; alert('Alerta generada y copiada al portapapeles'); }

/* Contactos */
addContact.addEventListener('click',()=>{ const name=cname.value.trim(), email=cemail.value.trim(), phone=cphone.value.trim(); if(!name||!email){ alert('Nombre y correo son requeridos'); return; } const list=loadContacts(); list.push({name,email,phone}); saveContacts(list); cname.value=''; cemail.value=''; cphone.value=''; renderContacts(); });

/* Buttons */
openSimulator.addEventListener('click',()=>{ simArea.classList.toggle('hidden'); panicArea.classList.add('hidden'); renderScenarios(); startScenario(scenarioSelect.value); });
scenarioSelect.addEventListener('change',()=>startScenario(scenarioSelect.value));
panicVisible.addEventListener('click',()=>{ simArea.classList.add('hidden'); panicArea.classList.remove('hidden'); sendAlert({silent:false}); });
panicSilent.addEventListener('click',()=>{ simArea.classList.add('hidden'); panicArea.classList.remove('hidden'); sendAlert({silent:true}); });
sendAlertNow.addEventListener('click',()=>sendAlert({silent:false}));
downloadEvidence.addEventListener('click',()=>{ const data=loadEvidence(); const blob=new Blob([JSON.stringify(data,null,2)],{type:'application/json'}); const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download='evidencia.json'; a.click(); });
copyAlert.addEventListener('click',async()=>{ const evidence=loadEvidence(); if(evidence.length===0){ alert('No hay alertas'); return; } try{ await navigator.clipboard.writeText(JSON.stringify(evidence[evidence.length-1].payload,null,2)); alert('Última alerta copiada'); }catch(e){ alert('No se pudo copiar'); }});

/* Detector */
analyzeBtn.addEventListener('click',()=>{ const t=textToAnalyze.value.trim(); if(!t){ analysisResult.textContent='Ingresa un texto.'; return; } const containsBad=t.toLowerCase().includes('malo')?'Posible riesgo':'Sin indicios'; analysisResult.textContent=containsBad; });
hfAnalyze.addEventListener('click',()=>{ const t=textToAnalyze.value.trim(); const token=hfTokenInput.value.trim(); if(!t||!token){ analysisResult.textContent='Texto o token faltante'; return; } analysisResult.textContent='Analizando en Hugging Face... (demo)'; });

renderContacts();
renderScenarios();
</script>
</body>
</html>
