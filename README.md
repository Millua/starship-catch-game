# starship-catch-game
Simulação 2D da captura do booster da Starship
[index.html](https://github.com/user-attachments/files/23111658/index.html)
<!doctype html>

<html lang="pt-BR">

<head>

<meta charset="utf-8" />

<meta name="viewport" content="width=device-width,initial-scale=1" />

<title>Starship Cinematic Edition — Smile Luã (Aprimorado)</title>

<style>

  /* ---------------------------

     Estilos principais

     --------------------------- */

  :root{

    --bg:#050519;

    --hud-bg: rgba(0,0,0,0.6);

    --accent: #FF6400;

    --rcs: #00BFFF;

    --text: #fff;

    --platform: #4A4A4A;

  }

  html,body{height:100%;margin:0;font-family:Inter, Arial, sans-serif;background:var(--bg);color:var(--text);-webkit-font-smoothing:antialiased;}

  #wrap{position:relative;height:100vh;overflow:hidden;}

  canvas{display:block;margin:0 auto;background:var(--bg);width:100%;height:100%;}

  /* UI containers */

  .hud{

    position:absolute;left:12px;top:12px;background:var(--hud-bg);padding:10px;border-radius:8px;font-family: "Courier New", monospace;z-index:40;

    min-width:180px;

  }

  .controls{

    position:absolute;left:0;right:0;bottom:12px;display:flex;justify-content:center;gap:12px;z-index:40;padding:0 12px;

  }

  .button{

    background:rgba(0,0,0,0.6);padding:12px 16px;border-radius:10px;border:2px solid rgba(255,255,255,0.12);cursor:pointer;user-select:none;touch-action:manipulation;

    transition: background 0.1s;

  }

  .button:hover{background:rgba(0,0,0,0.8);}

  .rcs-controls{position:absolute;left:50%;transform:translateX(-50%);bottom:96px;display:flex;gap:12px;z-index:40;}

  .rcs-button{width:60px;height:60px;border-radius:50%;display:flex;align-items:center;justify-content:center;background:rgba(0,0,0,0.6);border:2px solid rgba(255,255,255,0.08);font-size:18px;cursor:pointer;transition: background 0.1s;}

  .rcs-button:hover{background:rgba(0,0,0,0.8);}

  .tilt-indicator{position:absolute;left:50%;transform:translateX(-50%);bottom:170px;width:220px;height:40px;border-radius:20px;background:rgba(0,0,0,0.5);display:flex;align-items:center;justify-content:center;border:2px solid rgba(255,255,255,0.1);z-index:40}

  .tilt-bar{position:absolute;top:5px;left:50%;transform:translateX(-50%);width:4px;height:30px;background:var(--accent);border-radius:2px;transition:transform 0.08s, background 0.08s}

  .message-overlay{

    position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);background:rgba(0,0,0,0.85);padding:18px;border-radius:12px;

    border:2px solid rgba(255,255,255,0.08);z-index:60;min-width:260px;text-align:center;

  }

  .muter{position:absolute;right:12px;top:12px;background:var(--hud-bg);padding:8px;border-radius:8px;z-index:40}

  .small{font-size:12px;opacity:0.9}

  

  /* Estilo para o modal de som */

  #soundModal label{display:block; margin: 4px 0;}

  #soundModal input[type="text"]{

    background:#111; border:1px solid #333; color:var(--text); padding: 8px; border-radius: 4px; margin-bottom: 8px;

  }

  @media (max-width:600px){

    .rcs-button{width:50px;height:50px;font-size:16px}

    .tilt-indicator{bottom:140px}

    .rcs-controls{bottom:80px}

  }

</style>

</head>

<body>

  <div id="wrap">

    <canvas id="gameCanvas"></canvas>

    <div id="hud" class="hud"></div>

    <div id="tiltIndicator" class="tilt-indicator" aria-hidden="true">

      <div class="tilt-center" style="position:absolute;left:50%;top:0;transform:translateX(-50%);width:2px;height:100%;background:rgba(255,255,255,0.2)"></div>

      <div class="tilt-bar" id="tiltBar"></div>

      <div style="position:absolute;top:-18px;width:100%;display:flex;justify-content:space-between;font-size:12px;color:#fff;">

        <span>◀ ESQ</span><span>DIR ▶</span>

      </div>

    </div>

    <div id="rcsControls" class="rcs-controls"></div>

    <div class="controls" id="controls"></div>

    <div id="message" class="message-overlay" style="display:none"></div>

    <div id="muter" class="muter">

      <div style="display:flex;gap:8px;align-items:center">

        <label style="display:flex;flex-direction:column;align-items:flex-end">

          <span class="small">Som</span>

          <input id="soundToggle" type="checkbox" checked>

        </label>

        <button id="soundUrlBtn" class="button small" title="Carregar sons externos (opcional)">Sons extern.</button>

      </div>

    </div>

    

    <div id="soundModal" class="message-overlay" style="display:none; text-align:left; min-width: 300px;">

      <h3>Carregar Sons Externos</h3>

      <label>Engine URL:</label><input type="text" id="engUrl" placeholder="Ex: https://meusons.com/motor.mp3" style="width:calc(100% - 16px);"><br>

      <label>RCS URL:</label><input type="text" id="rcsUrl" placeholder="URL do jato RCS" style="width:calc(100% - 16px);"><br>

      <label>Explosion URL:</label><input type="text" id="expUrl" placeholder="URL da explosão" style="width:calc(100% - 16px);"><br>

      <label>Landing URL:</label><input type="text" id="lanUrl" placeholder="URL do pouso" style="width:calc(100% - 16px);"><br><br>

      <button id="loadSoundUrlsBtn" class="button">Carregar</button>

      <button id="closeSoundModalBtn" class="button" style="float:right">Cancelar</button>

    </div>

  </div>

<script>

/* ============================

   Starship Cinematic Edition

   Mantém jogabilidade original

   + sons, câmera shake, otimizações

   + IMPROVED: Aerodinâmica, UX Sound Loading, Parallax Stars

   Autor: ChatGPT (gerado para Smile Luã)

   ============================ */

/* --------- Config e constantes (mantive física original) ---------- */

const canvas = document.getElementById('gameCanvas');

const ctx = canvas.getContext('2d');

let DPR = Math.max(1, window.devicePixelRatio || 1);

function resizeCanvas() {

  const w = window.innerWidth;

  const h = window.innerHeight;

  canvas.style.width = w + 'px';

  canvas.style.height = h + 'px';

  canvas.width = Math.floor(w * DPR);

  canvas.height = Math.floor(h * DPR);

  ctx.setTransform(DPR,0,0,DPR,0,0);

  // Ajusta zonas dependentes do tamanho:

  GROUND_LEVEL = canvas.height/DPR - 50;

  PLATFORM_ZONE = [canvas.width/DPR * 0.4, canvas.width/DPR * 0.6];

}

window.addEventListener('resize', resizeCanvas);

/* --- Game constants (mantidos e levemente ajustados) --- */

const GRAVITY = 0.15;

const THRUST = 0.5;

const ROTATION_SPEED = 1.5;

const STABILIZATION_FACTOR = 0.08;

const FUEL_CONSUMPTION = 1.2;

const ROTATION_FUEL_CONSUMPTION = 0.3;

const RCS_FUEL_CONSUMPTION = 0.4;

const RCS_THRUST = 0.2;

// NOVOS Fatores de Arrasto Aerodinâmico para simular o Belly Flop

const AIR_DENSITY = 0.0005; // Ajuste para a taxa de desaceleração

const MAX_DRAG_COEF = 0.8;

const MIN_DRAG_COEF = 0.1;

const INITIAL_FUEL = 1500.0;

const LANDING_GEAR_DEPLOY_ALT = 600;

const TOWER_HEIGHT = 180;

/* Colors / visuals */

const SKY_BLUE = '#050519';

const GROUND_COLOR = '#003500';

const PLATFORM_COLOR = '#4A4A4A';

const SHIP_BODY = '#D3D3D3';

const SHIP_TILES = '#000000';

const FLAME_ORANGE = '#FF6400';

const FLAME_RED = '#FF0000';

const RCS_BLUE = '#00BFFF';

const MOUNTAIN_BROWN = '#453528';

const LANDING_LEG_COLOR = '#8B4513';

const TOWER_ARM_COLOR = '#C0C0C0';

/* --------- Game state ---------- */

let state = 'difficulty'; // 'difficulty' | 'game' | 'end'

let attempts = 0;

let successes = 0;

let difficulty = 1;

let fuel, wind_force, ship_x, ship_y, vx, vy, angle;

let thrusting=false, rotating_left=false, rotating_right=false;

let rcs_left=false, rcs_right=false, rcs_up=false, rcs_down=false;

let particles = [];

let stars = [];

let landed=false, crashed=false;

let message = '';

let lastTime = 0;

let gameTime = 0;

let cameraY = 0;

let flipManeuver = false;

let show_tips = true;

let tips_timer = 5.0;

let isCenteredOnPlatform = false;

/* Limits / optimizations */

const MAX_PARTICLES = 400; // limite para evitar travamentos

const PARTICLE_POOL_TRIM = 50;

/* Platform zone dynamic */

let PLATFORM_ZONE = [320, 480];

let GROUND_LEVEL = 550;

/* Key handling (keyboard + mobile) */

const keys = {};

document.addEventListener('keydown', e => { keys[e.key] = true; if(['ArrowUp','ArrowDown','ArrowLeft','ArrowRight',' ','a','d','w','s','f','r','R'].includes(e.key)) e.preventDefault();});

document.addEventListener('keyup', e => { keys[e.key] = false; });

/* UI refs */

const controlsDiv = document.getElementById('controls');

const hudDiv = document.getElementById('hud');

const tiltBar = document.getElementById('tiltBar');

const rcsControls = document.getElementById('rcsControls');

const messageOverlay = document.getElementById('message');

const soundToggle = document.getElementById('soundToggle');

const soundUrlBtn = document.getElementById('soundUrlBtn');

// NOVOS: Refs para o Modal de Som

const soundModal = document.getElementById('soundModal');

const loadSoundUrlsBtn = document.getElementById('loadSoundUrlsBtn');

const closeSoundModalBtn = document.getElementById('closeSoundModalBtn');

/* ------------------

   Utility classes

   ------------------ */

class Particle {

  constructor(x,y,vx,vy,life,color,size=2){

    this.x=x;this.y=y;this.vx=vx;this.vy=vy;this.life=life;this.maxLife=life;this.color=color;this.size=size;

  }

  update(dt){

    // gravity applied to particles for realism

    this.vy += GRAVITY * dt * 0.6;

    this.x += this.vx * dt * 60;

    this.y += this.vy * dt * 60;

    this.life -= dt*60;

    return this.life > 0;

  }

  draw(ctx,cameraY){

    const a = Math.max(0, this.life / this.maxLife);

    // allow color like '#RRGGBB' or named; we'll draw with globalAlpha for simplicity

    ctx.save();

    ctx.globalAlpha = a;

    ctx.fillStyle = this.color;

    ctx.beginPath();

    ctx.arc(this.x, this.y - cameraY, this.size, 0, Math.PI*2);

    ctx.fill();

    ctx.restore();

  }

}

class Star {

  constructor(x,y,size,brightness){

    this.x=x;this.y=y;this.size=size;this.brightness=brightness;

    this.twinkleSpeed = Math.random()*0.04 + 0.02;

    this.twinkleOffset = Math.random()*Math.PI*2;

    this.z = Math.random()*0.8 + 0.2; // NOVO: Profundidade para efeito Parallax (0.2 a 1.0)

  }

  update(t){

    this.brightness = 0.6 + 0.4 * Math.sin(t * this.twinkleSpeed + this.twinkleOffset);

  }

  draw(ctx, cameraY){ // NOVO: Recebe cameraY

    // Estrelas mais distantes (z=1.0) se movem menos.

    const parallaxY = cameraY * this.z * 0.1; // Ajuste o fator 0.1 para a intensidade do paralaxe

    

    ctx.fillStyle = `rgba(255,255,255,${this.brightness})`;

    ctx.beginPath();

    // Aplica o offset de paralaxe

    ctx.arc(this.x, this.y - parallaxY, this.size, 0, Math.PI*2); 

    ctx.fill();

  }

}

/* -----------------------

   Sound manager (WebAudio)

   ----------------------- */

class SoundManager {

  constructor(){

    this.ctx = null;

    this.master = null;

    this.engineNode = null;

    this.gain = 0.8;

    this.enabled = true;

    this.external = {engine:null, rcs:null, explosion:null, landing:null}; // optional AudioBuffers

    this.usingExternal = false;

    // NOVO: Track RCS synth node (para som de pulso contínuo)

    this.rcsNode = null;

  }

  async init(){

    if (this.ctx) return;

    try{

      const AudioContext = window.AudioContext || window.webkitAudioContext;

      this.ctx = new AudioContext();

      this.master = this.ctx.createGain();

      this.master.gain.value = this.gain;

      this.master.connect(this.ctx.destination);

      // precreate synth nodes for engine

      this._createEngineSynth();

    }catch(e){

      console.warn('WebAudio não disponível', e);

      this.ctx = null;

    }

  }

  _createEngineSynth(){

    if(!this.ctx) return;

    // Engine: multiple detuned sawtooth LFO + lowpass

    const osc1 = this.ctx.createOscillator();

    const osc2 = this.ctx.createOscillator();

    const osc3 = this.ctx.createOscillator();

    osc1.type = 'sawtooth'; osc2.type = 'sawtooth'; osc3.type = 'sawtooth';

    osc1.frequency.value = 60; osc2.frequency.value = 62; osc3.frequency.value = 58;

    const oscGain = this.ctx.createGain(); oscGain.gain.value = 0.2;

    const filter = this.ctx.createBiquadFilter(); filter.type='lowpass'; filter.frequency.value = 400;

    const comp = this.ctx.createDynamicsCompressor();

    osc1.connect(oscGain); osc2.connect(oscGain); osc3.connect(oscGain);

    oscGain.connect(filter); filter.connect(comp); comp.connect(this.master);

    // noise layer (whisper) to add rumble

    const bufferSize = 2*this.ctx.sampleRate;

    const noiseBuffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);

    const data = noiseBuffer.getChannelData(0);

    for(let i=0;i<bufferSize;i++) data[i] = (Math.random()*2-1)*0.2;

    const noiseSource = this.ctx.createBufferSource(); noiseSource.buffer = noiseBuffer; noiseSource.loop = true;

    const noiseFilter = this.ctx.createBiquadFilter(); noiseFilter.type='lowpass'; noiseFilter.frequency.value=800;

    const noiseGain = this.ctx.createGain(); noiseGain.gain.value = 0.04;

    noiseSource.connect(noiseFilter); noiseFilter.connect(noiseGain); noiseGain.connect(this.master);

    // store nodes

    this.engineNode = {osc:[osc1,osc2,osc3], oscGain, filter, noiseSource, noiseGain, running:false};

    // start (in suspended state)

    osc1.start(); osc2.start(); osc3.start();

    noiseSource.start();

    this.engineNode.running = true;

    // set initial engine gain to 0

    oscGain.gain.value = 0.0;

    noiseGain.gain.value = 0.0;

  }

  setEnabled(flag){

    this.enabled = !!flag;

    this.master && (this.master.gain.value = this.enabled ? this.gain : 0);

  }

  // engine level: 0..1 (throttle)

  setEngineLevel(level){

    if(!this.ctx || !this.engineNode) return;

    // Smooth ramp

    const now = this.ctx.currentTime;

    const target = Math.max(0, Math.min(1, level));

    // scale oscillator gain and noise

    this.engineNode.oscGain.gain.cancelScheduledValues(now);

    this.engineNode.oscGain.gain.linearRampToValueAtTime(0.03 + 0.4*target, now + 0.08);

    this.engineNode.noiseGain.gain.cancelScheduledValues(now);

    this.engineNode.noiseGain.gain.linearRampToValueAtTime(0.01 + 0.05*target, now + 0.08);

    // sweep filter frequency a bit with throttle

    const freq = 300 + 1200 * target;

    this.engineNode.filter.frequency.cancelScheduledValues(now);

    this.engineNode.filter.frequency.linearRampToValueAtTime(freq, now+0.1);

  }

  // RCS pulse (short) — we play a short noise burst (melhorado para ser reutilizado)

  playRCSPulse(){

    if(!this.ctx || !this.enabled) return;

    if(this.usingExternal && this.external.rcs){

      this._playBuffer(this.external.rcs, 0.6);

      return;

    }

    // Cria um pulso de ruído curto

    const buffer = this.ctx.createBuffer(1, this.ctx.sampleRate*0.1, this.ctx.sampleRate); // Pulso mais curto

    const d = buffer.getChannelData(0);

    for(let i=0;i<d.length;i++){

      d[i] = (Math.random()*2-1) * (1 - i/d.length)**2 * 0.7; // Decay mais rápido

    }

    const src = this.ctx.createBufferSource(); src.buffer = buffer;

    const f = this.ctx.createBiquadFilter(); f.type='highpass'; f.frequency.value = 800; // Mais agudo

    const g = this.ctx.createGain(); g.gain.value = 0.5;

    src.connect(f); f.connect(g); g.connect(this.master);

    src.start();

  }

  // explosion / big event

  playExplosion(){

    if(!this.ctx || !this.enabled) return;

    if(this.usingExternal && this.external.explosion) { this._playBuffer(this.external.explosion,1.0); return; }

    const buffer = this.ctx.createBuffer(1, this.ctx.sampleRate*1.2, this.ctx.sampleRate);

    const d = buffer.getChannelData(0);

    // burst + decaying noise

    for(let i=0;i<d.length;i++){

      d[i] = (Math.random()*2-1) * (1 - i/d.length) * (Math.random()*1.5);

    }

    const src = this.ctx.createBufferSource(); src.buffer = buffer;

    const f = this.ctx.createBiquadFilter(); f.type='lowpass'; f.frequency.value = 1600;

    const g = this.ctx.createGain(); g.gain.value = 1.0;

    src.connect(f); f.connect(g); g.connect(this.master);

    src.start();

  }

  // landing thud

  playLanding(){

    if(!this.ctx || !this.enabled) return;

    if(this.usingExternal && this.external.landing) { this._playBuffer(this.external.landing,0.8); return; }

    const buffer = this.ctx.createBuffer(1, this.ctx.sampleRate*0.6, this.ctx.sampleRate);

    const d = buffer.getChannelData(0);

    for(let i=0;i<d.length;i++){

      d[i] = (Math.random()*2-1) * Math.exp(-i/ (this.ctx.sampleRate*0.3));

    }

    const src = this.ctx.createBufferSource(); src.buffer = buffer;

    const f = this.ctx.createBiquadFilter(); f.type='lowpass'; f.frequency.value = 500;

    const g = this.ctx.createGain(); g.gain.value = 0.6;

    src.connect(f); f.connect(g); g.connect(this.master);

    src.start();

  }

  _playBuffer(buffer, gain=1.0){

    const src = this.ctx.createBufferSource();

    src.buffer = buffer;

    const g = this.ctx.createGain(); g.gain.value = gain;

    src.connect(g); g.connect(this.master);

    src.start();

  }

  // load external URL (wav/mp3) into buffer

  async loadExternal(url){

    if(!this.ctx) await this.init();

    try{

      const res = await fetch(url);

      const ab = await res.arrayBuffer();

      const buf = await this.ctx.decodeAudioData(ab);

      return buf;

    } catch(e){

      console.warn('Falha ao carregar som externo', url, e);

      return null;

    }

  }

  // user can provide external buffers for more realism

  async setExternalSounds({engineUrl, rcsUrl, explosionUrl, landingUrl}){

    try {

      await this.init();

      const load = async (u)=>u?await this.loadExternal(u):null;

      const [e,r,ex,l] = await Promise.all([load(engineUrl), load(rcsUrl), load(explosionUrl), load(landingUrl)]);

      this.external.engine = e; this.external.rcs = r; this.external.explosion = ex; this.external.landing = l;

      // if at least one loaded -> use external

      this.usingExternal = !!(e||r||ex||l);

      return this.usingExternal;

    } catch(e){ console.warn(e); return false; }

  }

}

const sound = new SoundManager();

/* -----------------------------------

   Initialization helpers (stars, etc.)

   ----------------------------------- */

function initStars(){

  stars = [];

  for(let i=0;i<180;i++){

    // Instancia Star com a nova lógica de 'z'

    stars.push(new Star(Math.random()*canvas.width/DPR, Math.random()*canvas.height/DPR*0.7, Math.random()*1.6+0.4, Math.random()*0.6+0.4));

  }

}

/* --------------

   Controls UI

   -------------- */

function createButton(text, onDown, onUp, className=''){

  const btn = document.createElement('div');

  btn.className = 'button ' + className;

  btn.innerText = text;

  let pressed = false;

  // NOVO: Adicione 'active' class para feedback visual

  const start = (e)=>{ e.preventDefault(); pressed=true; onDown && onDown(); btn.classList.add('active'); };

  const end = (e)=>{ if(pressed){ e.preventDefault(); pressed=false; onUp && onUp(); } btn.classList.remove('active'); };

  btn.addEventListener('touchstart', start, {passive:false}); btn.addEventListener('touchend', end, {passive:false});

  btn.addEventListener('mousedown', start); document.addEventListener('mouseup', end);

  btn.addEventListener('mouseleave', end);

  return btn;

}

function createRCSButton(text,onDown,onUp){

  const btn = document.createElement('div');

  btn.className = 'rcs-button';

  btn.innerText = text;

  let pressed=false;

  // NOVO: Remove sound.playRCSPulse() daqui. O som agora será loopado no update().

  const start = (e)=>{ e.preventDefault(); pressed=true; onDown && onDown(); btn.classList.add('active'); };

  const end = (e)=>{ if(pressed){ e.preventDefault(); pressed=false; onUp && onUp(); } btn.classList.remove('active'); };

  btn.addEventListener('touchstart', start, {passive:false}); btn.addEventListener('touchend', end, {passive:false});

  btn.addEventListener('mousedown', start); document.addEventListener('mouseup', end);

  btn.addEventListener('mouseleave', end);

  return btn;

}

function setupControls(){

  controlsDiv.innerHTML=''; rcsControls.innerHTML='';

  if(state==='difficulty'){

    const container = document.createElement('div');

    container.style.display='flex'; container.style.flexDirection='column'; container.style.alignItems='center';

    const title = document.createElement('div'); title.innerText = 'Starship Simulator — Cinematic Edition'; title.style.fontSize='20px'; title.style.color='#FF9A4D'; title.style.marginBottom='8px';

    container.appendChild(title);

    const easy = createButton('Fácil (1)', ()=>startGame(1));

    const med = createButton('Médio (2)', ()=>startGame(2));

    const hard = createButton('Difícil (3)', ()=>startGame(3));

    [easy,med,hard].forEach(b=>{ b.style.margin='6px'; b.style.width='220px'; container.appendChild(b); });

    const stats = document.createElement('div'); stats.className='small'; stats.style.marginTop='8px'; stats.innerText = `Tentativas: ${attempts}  |  Sucessos: ${successes}`;

    container.appendChild(stats);

    controlsDiv.appendChild(container);

    document.getElementById('tiltIndicator').style.display='none';

    rcsControls.style.display='none';

  } else if(state==='game'){

    // main controls bottom

    const leftBtn = createButton('◀', ()=>rotating_left=true, ()=>rotating_left=false);

    const thrustBtn = createButton('RAPTOR', ()=>thrusting=true, ()=>thrusting=false);

    const rightBtn = createButton('▶', ()=>rotating_right=true, ()=>rotating_right=false);

    // Botão Flip agora é um toggle, com feedback visual na HUD

    const flipBtn = createButton('FLIP', ()=>{ flipManeuver = !flipManeuver; }, ()=>{}); 

    [leftBtn, thrustBtn, rightBtn, flipBtn].forEach(b=> b.style.minWidth='80px');

    controlsDiv.appendChild(leftBtn); controlsDiv.appendChild(thrustBtn); controlsDiv.appendChild(rightBtn); controlsDiv.appendChild(flipBtn);

    // RCS circle

    const rL = createRCSButton('←', ()=>{ rcs_left = true; }, ()=> rcs_left=false);

    const rU = createRCSButton('↑', ()=>{ rcs_up = true; }, ()=> rcs_up=false);

    const rD = createRCSButton('↓', ()=>{ rcs_down = true; }, ()=> rcs_down=false);

    const rR = createRCSButton('→', ()=>{ rcs_right = true; }, ()=> rcs_right=false);

    rcsControls.appendChild(rL); rcsControls.appendChild(rU); rcsControls.appendChild(rD); rcsControls.appendChild(rR);

    rcsControls.style.display='flex';

    document.getElementById('tiltIndicator').style.display='block';

  } else {

    const restart = createButton('Reiniciar (R)', ()=>resetToDifficulty());

    controlsDiv.appendChild(restart);

    document.getElementById('tiltIndicator').style.display='none';

    rcsControls.style.display='none';

  }

}

/* ------------------------------

   Start / Reset / Core routines

   ------------------------------ */

function resetToDifficulty(){

  state='difficulty';

  setupControls();

  messageOverlay.style.display='none';

  soundModal.style.display='none'; // Garante que o modal esteja fechado

}

function startGame(diff){

  difficulty = diff;

  const fuelMultiplier = (4 - difficulty)/2.0;

  fuel = INITIAL_FUEL * fuelMultiplier;

  wind_force = (Math.random()*0.5*difficulty - 0.25*difficulty);

  ship_x = canvas.width/DPR / 2 + (Math.random()*200 - 100);

  ship_y = 100 + Math.random()*40;

  vx = Math.random()*2 - 1;

  vy = 0.5;

  angle = 0;

  thrusting = rotating_left = rotating_right = false;

  rcs_left = rcs_right = rcs_up = rcs_down = false;

  flipManeuver = false;

  particles = [];

  landed = crashed = false;

  show_tips = true; tips_timer = 5.0;

  message = '';

  cameraY = 0;

  initStars();

  state='game';

  setupControls();

  // resume audio context in user gesture

  sound.init().then(()=>{ if(soundToggle.checked){ sound.setEnabled(true); } else { sound.setEnabled(false); } });

}

/* -------------------------

   Drawing functions

   ------------------------- */

function drawStars(){

  // NOVO: Passa cameraY para o efeito parallax

  for(let s of stars){ s.update(gameTime); s.draw(ctx, cameraY); } 

}

function drawMountains(){

  ctx.fillStyle = MOUNTAIN_BROWN;

  ctx.beginPath();

  ctx.moveTo(0, GROUND_LEVEL - cameraY);

  ctx.lineTo(canvas.width*0.15/DPR, GROUND_LEVEL - 100 - cameraY);

  ctx.lineTo(canvas.width*0.3/DPR, GROUND_LEVEL - 50 - cameraY);

  ctx.lineTo(canvas.width*0.5/DPR, GROUND_LEVEL - 150 - cameraY);

  ctx.lineTo(canvas.width*0.7/DPR, GROUND_LEVEL - 70 - cameraY);

  ctx.lineTo(canvas.width*0.85/DPR, GROUND_LEVEL - 110 - cameraY);

  ctx.lineTo(canvas.width/DPR, GROUND_LEVEL - 90 - cameraY);

  ctx.lineTo(canvas.width/DPR, canvas.height/DPR);

  ctx.lineTo(0, canvas.height/DPR);

  ctx.closePath();

  ctx.fill();

}

function drawPlatform(){

  const PLATFORM_CENTER_X = (PLATFORM_ZONE[0] + PLATFORM_ZONE[1]) / 2;

  const PLATFORM_TOP_Y = GROUND_LEVEL - TOWER_HEIGHT - cameraY;

  ctx.fillStyle = PLATFORM_COLOR;

  ctx.fillRect(PLATFORM_ZONE[0], GROUND_LEVEL - cameraY, PLATFORM_ZONE[1] - PLATFORM_ZONE[0], canvas.height/DPR - (GROUND_LEVEL - cameraY));

  // Tower

  ctx.fillStyle = PLATFORM_COLOR;

  ctx.fillRect(PLATFORM_CENTER_X - 20, PLATFORM_TOP_Y, 40, TOWER_HEIGHT);

  // Arms (chopsticks) animate slightly when landed and centered

  ctx.fillStyle = TOWER_ARM_COLOR;

  let armWidth = 100;

  if(landed && isCenteredOnPlatform){

    armWidth = 50 + Math.sin(gameTime*5)*5;

  }

  ctx.fillRect(PLATFORM_CENTER_X - armWidth/2, PLATFORM_TOP_Y - 15, armWidth, 25);

  // target light

  ctx.strokeStyle = '#FFD700';

  ctx.lineWidth = 3;

  ctx.beginPath();

  ctx.rect(PLATFORM_CENTER_X - 20, PLATFORM_TOP_Y - 15, 40, 25);

  ctx.stroke();

  const blink = Math.sin(gameTime * 5) > 0;

  if(blink){

    ctx.fillStyle = '#00FF00';

    ctx.beginPath();

    ctx.arc(PLATFORM_CENTER_X - 25, PLATFORM_TOP_Y - 7, 4, 0, Math.PI*2);

    ctx.arc(PLATFORM_CENTER_X + 25, PLATFORM_TOP_Y - 7, 4, 0, Math.PI*2);

    ctx.fill();

  }

  // guide lines

  ctx.strokeStyle = 'rgba(255,255,255,0.08)';

  ctx.setLineDash([6,6]);

  ctx.beginPath();

  ctx.moveTo(PLATFORM_ZONE[0], GROUND_LEVEL - cameraY);

  ctx.lineTo(PLATFORM_ZONE[0], canvas.height/DPR);

  ctx.moveTo(PLATFORM_ZONE[1], GROUND_LEVEL - cameraY);

  ctx.lineTo(PLATFORM_ZONE[1], canvas.height/DPR);

  ctx.stroke();

  ctx.setLineDash([]);

}

function drawShip(x,y,angleDeg,thrusting,altitude){

  ctx.save();

  ctx.translate(x, y - cameraY);

  ctx.rotate(angleDeg * Math.PI / 180);

  // body

  ctx.fillStyle = SHIP_BODY;

  ctx.beginPath();

  ctx.rect(-12, -40, 24, 80);

  ctx.fill();

  // tiles (ventral)

  ctx.fillStyle = SHIP_TILES;

  ctx.beginPath();

  ctx.rect(-12, -40, 12, 80);

  ctx.fill();

  // flaps

  // Simulação de flaps (ajuste de ângulo conforme a manobra)

  const angleToHorizontal = Math.abs(angleDeg % 360 - 90);

  const flapBase = Math.min(1, angleToHorizontal/90); // 1.0 (vertical) a 0.0 (horizontal)

  const flapAngle = 5 * flapBase + angleDeg * 0.2; // Pequeno ajuste de 5 graus + correção angular

  

  ctx.fillStyle = '#A0A0A0';

  

  // Flap 1

  ctx.save();

  ctx.rotate((flapAngle) * Math.PI / 180);

  ctx.beginPath();

  ctx.moveTo(-12, 60);

  ctx.lineTo(-20, 80);

  ctx.lineTo(-12, 70);

  ctx.closePath();

  ctx.fill();

  ctx.restore();

  // Flap 2

  ctx.save();

  ctx.rotate((-flapAngle) * Math.PI / 180);

  ctx.beginPath();

  ctx.moveTo(12, 60);

  ctx.lineTo(20, 80);

  ctx.lineTo(12, 70);

  ctx.closePath();

  ctx.fill();

  ctx.restore();

  // windows

  ctx.fillStyle = '#87CEEB';

  for(let i=0;i<3;i++){ ctx.beginPath(); ctx.arc(0, -30 + i*15, 4, 0, Math.PI*2); ctx.fill(); }

  // RCS flames (with gradient)

  if(rcs_left && fuel>0){

    const g = ctx.createLinearGradient(-18, -5, -36, -5);

    g.addColorStop(0, RCS_BLUE); g.addColorStop(1, 'rgba(0,188,255,0)');

    ctx.fillStyle = g;

    ctx.beginPath(); ctx.moveTo(-18,-5); ctx.lineTo(-36,0); ctx.lineTo(-18,5); ctx.closePath(); ctx.fill();

  }

  if(rcs_right && fuel>0){

    const g = ctx.createLinearGradient(18, -5, 36, -5);

    g.addColorStop(0, RCS_BLUE); g.addColorStop(1, 'rgba(0,188,255,0)');

    ctx.fillStyle = g;

    ctx.beginPath(); ctx.moveTo(18,-5); ctx.lineTo(36,0); ctx.lineTo(18,5); ctx.closePath(); ctx.fill();

  }

  if(rcs_up && fuel>0){

    const g = ctx.createLinearGradient(0, -40, 0, -66);

    g.addColorStop(0, RCS_BLUE); g.addColorStop(1, 'rgba(0,188,255,0)');

    ctx.fillStyle = g;

    ctx.beginPath(); ctx.moveTo(-6,-40); ctx.lineTo(0,-66); ctx.lineTo(6,-40); ctx.closePath(); ctx.fill();

  }

  if(rcs_down && fuel>0){

    const g = ctx.createLinearGradient(0, 40, 0, 72);

    g.addColorStop(0, RCS_BLUE); g.addColorStop(1, 'rgba(0,188,255,0)');

    ctx.fillStyle = g;

    ctx.beginPath(); ctx.moveTo(-6,40); ctx.lineTo(0,72); ctx.lineTo(6,40); ctx.closePath(); ctx.fill();

  }

  // landing legs

  if(altitude < LANDING_GEAR_DEPLOY_ALT){

    ctx.strokeStyle = LANDING_LEG_COLOR; ctx.lineWidth=3;

    ctx.beginPath(); ctx.moveTo(-8,40); ctx.lineTo(-16,60); ctx.stroke();

    ctx.beginPath(); ctx.moveTo(8,40); ctx.lineTo(16,60); ctx.stroke();

  }

  // main thrust flames improved (multiple engines + gradient)

  if(thrusting && fuel > 0){

    for(let i=-1;i<=1;i++){

      const flameHeight = 30 + Math.random()*8;

      const flameWidth = 8 + Math.random()*4;

      const offsetX = i * 8;

      const gradient = ctx.createLinearGradient(offsetX, 40, offsetX, 40 + flameHeight);

      gradient.addColorStop(0, FLAME_ORANGE);

      gradient.addColorStop(0.6, FLAME_RED);

      gradient.addColorStop(1, 'rgba(255,0,0,0)');

      ctx.fillStyle = gradient;

      ctx.beginPath();

      ctx.moveTo(offsetX - flameWidth/2, 40);

      ctx.lineTo(offsetX, 40 + flameHeight);

      ctx.lineTo(offsetX + flameWidth/2, 40);

      ctx.closePath(); ctx.fill();

    }

  }

  ctx.restore();

}

function drawTerrain(){

  drawMountains();

  ctx.fillStyle = GROUND_COLOR;

  ctx.fillRect(0, GROUND_LEVEL - cameraY, canvas.width/DPR, canvas.height/DPR - (GROUND_LEVEL - cameraY));

  drawPlatform();

}

/* ---------------

   HUD drawing

   --------------- */

function drawHUD(){

  const altitude = Math.max(0, Math.floor(GROUND_LEVEL - ship_y));

  const verticalSpeed = vy.toFixed(1);

  const horizontalSpeed = vx.toFixed(1);

  const fuelRemaining = Math.floor(fuel);

  const shipAngle = angle.toFixed(1);

  const fuelPercent = (fuel / INITIAL_FUEL) * 100;

  // tilt bar updated

  const tiltPercent = Math.max(-100, Math.min(100, (angle/45)*100));

  tiltBar.style.transform = `translateX(${tiltPercent}%)`;

  if(Math.abs(tiltPercent) < 20) tiltBar.style.background = '#00FF00';

  else if(Math.abs(tiltPercent) < 60) tiltBar.style.background = '#FFFF00';

  else tiltBar.style.background = '#FF6400';

  // HUD html

  hudDiv.innerHTML = `

    <div>Altitude: <b>${altitude} m</b></div>

    <div>Vel.Vert: <b>${verticalSpeed} m/s</b></div>

    <div>Vel.Horiz: <b>${horizontalSpeed} m/s</b></div>

    <div>Combustível: <b>${fuelRemaining}</b></div>

    <div>Ângulo: <b>${shipAngle}°</b></div>

    <div style="color: ${flipManeuver ? '#00FF00' : '#FF6400'};">Flip: <b>${flipManeuver ? 'ATIVO' : 'OFF'}</b></div>

    <div style="margin-top:6px;">Tentativas: ${attempts} | Sucessos: ${successes}</div>

    <div style="margin-top:6px;">

      <div style="width:110px;height:8px;background:#222;border-radius:4px;display:inline-block;vertical-align:middle;">

        <div style="width:${Math.max(0,Math.min(100,(fuelPercent)))}%;height:100%;background:${fuelPercent<10? '#FF0000' : fuelPercent<30 ? '#FFFF00' : '#00FF00'};border-radius:4px"></div>

      </div>

    </div>

  `;

}

/* -----------------------

   Particle helpers

   ----------------------- */

function trimParticles(){

  if(particles.length > MAX_PARTICLES){

    particles.splice(0, PARTICLE_POOL_TRIM);

  }

}

function addRCSParticles(dirX=0, dirY=0){

  if(particles.length > MAX_PARTICLES) return;

  for(let i=0;i<3;i++){

    const pvx = vx + dirX*(3+Math.random()*3) + Math.random()*2-1;

    const pvy = vy + dirY*(3+Math.random()*3) + Math.random()*2-1;

    particles.push(new Particle(ship_x + dirX*12 + (Math.random()*6-3), ship_y + dirY*12 + (Math.random()*6-3), pvx, pvy, 20, RCS_BLUE, Math.random()*2+1));

  }

}

function addThrustParticles(){

  if(particles.length > MAX_PARTICLES) return;

  for(let i=0;i<5;i++){

    const pvx = vx + Math.random()*5 - 2.5;

    const pvy = vy + Math.random()*5 - 2.5;

    particles.push(new Particle(ship_x + (Math.random()*12-6), ship_y + 40, pvx, pvy, 25, FLAME_ORANGE, Math.random()*3+1));

  }

}

function addSuccessParticles(x,y,count,color){

  for(let i=0;i<count && particles.length < MAX_PARTICLES;i++){

    const pvx = Math.random()*8 - 4;

    const pvy = Math.random()*-12 - 6;

    particles.push(new Particle(x, y, pvx, pvy, 80, color, Math.random()*4+2));

  }

}

function addExplosionParticles(x,y){

  for(let i=0;i<80 && particles.length < MAX_PARTICLES;i++){

    const pvx = Math.random()*20 - 10;

    const pvy = Math.random()*20 - 10;

    particles.push(new Particle(x, y, pvx, pvy, 120, '#FF3300', Math.random()*5+2));

  }

  for(let i=0;i<20 && particles.length < MAX_PARTICLES;i++){

    const pvx = Math.random()*10 - 5;

    const pvy = Math.random()*5;

    particles.push(new Particle(x, y - Math.random()*40, pvx, pvy, 100, SHIP_TILES, Math.random()*3+1));

  }

}

/* ------------------------

   Camera update + shake

   ------------------------ */

let camShake = 0;

function updateCamera(dt){

  const targetY = ship_y - (canvas.height/DPR) * 0.3;

  cameraY += (targetY - cameraY) * 5 * dt;

  cameraY = Math.min(cameraY, GROUND_LEVEL - canvas.height/DPR);

  cameraY = Math.max(cameraY, 0);

  // camera shake decay

  camShake *= Math.pow(0.9, dt*60);

}

/* ---------------------

   Update loop

   --------------------- */

function update(dt){

  if(state !== 'game') return;

  gameTime += dt;

  updateCamera(dt);

  // apply inputs: preserve original behavior

  thrusting = keys[' '] || keys['ArrowUp'] || thrusting;

  rotating_left = keys['ArrowLeft'] || keys['a'] || rotating_left;

  rotating_right = keys['ArrowRight'] || keys['d'] || rotating_right;

  rcs_left = keys['ArrowLeft'] || keys['a'] || rcs_left;

  rcs_right = keys['ArrowRight'] || keys['d'] || rcs_right;

  rcs_up = keys['w'] || rcs_up;

  rcs_down = keys['s'] || rcs_down;

  if(keys['f'] || keys['F']) {

    // toggle flip on press; ensure not toggling continuously by clearing key

    flipManeuver = !flipManeuver; keys['f']=false; keys['F']=false;

  }

  // rotation controls

  if(rotating_left && fuel > 0){

    angle -= ROTATION_SPEED * dt * 60;

    fuel -= ROTATION_FUEL_CONSUMPTION * dt * 60;

  }

  if(rotating_right && fuel > 0){

    angle += ROTATION_SPEED * dt * 60;

    fuel -= ROTATION_FUEL_CONSUMPTION * dt * 60;

  }

  // RCS + Som Aprimorado

  let rcs_active_this_frame = false;

  if(rcs_left && fuel > 0){

    vx -= RCS_THRUST * dt * 60; fuel -= RCS_FUEL_CONSUMPTION * dt * 60; addRCSParticles(-1); rcs_active_this_frame = true;

  }

  if(rcs_right && fuel > 0){

    vx += RCS_THRUST * dt * 60; fuel -= RCS_FUEL_CONSUMPTION * dt * 60; addRCSParticles(1); rcs_active_this_frame = true;

  }

  if(rcs_up && fuel > 0){

    vy -= RCS_THRUST * dt * 60; fuel -= RCS_FUEL_CONSUMPTION * dt * 60; addRCSParticles(0,-1); rcs_active_this_frame = true;

  }

  if(rcs_down && fuel > 0){

    vy += RCS_THRUST * dt * 60; fuel -= RCS_FUEL_CONSUMPTION * dt * 60; addRCSParticles(0,1); rcs_active_this_frame = true;

  }

  // Toca o som RCS periodicamente enquanto ativo (somente se não estiver usando o RCS de rotação)

  if (rcs_active_this_frame && !(rotating_left || rotating_right)) {

      if (gameTime * 60 % 15 < dt * 60) { // A cada ~15 frames

          sound.playRCSPulse();

      }

  }

  // Flip maneuver simulation (keeps original behavior)

  if(flipManeuver && ship_y > 500){

    if(Math.abs(angle - 90) > 1){

      angle += (90 - angle) * 0.05;

    } else if(ship_y < 300 && Math.abs(angle) > 1){

      angle -= angle * 0.1;

    }

  }

  // auto-stabilize

  if(!rotating_left && !rotating_right && Math.abs(angle) > 0.5){

    angle -= angle * STABILIZATION_FACTOR * dt * 60;

  }

  // physics

  vy += GRAVITY * dt * 60;

  vx += wind_force * dt * 60;

  // NOVO: Arrasto Aerodinâmico para o Belly Flop

  const angleRad = angle * Math.PI / 180;

  // Drag Coeficiente: Máximo quando próximo de 90° (horizontal/barriga para baixo)

  let dragCoefficient = MIN_DRAG_COEF + (MAX_DRAG_COEF - MIN_DRAG_COEF) * Math.sin(angleRad)**2;

  const totalVelocity = Math.sqrt(vx*vx + vy*vy);

  if (totalVelocity > 0) {

      const DRAG_FACTOR = dragCoefficient * AIR_DENSITY * dt * 60; 

      const dragForceX = DRAG_FACTOR * vx * totalVelocity;

      const dragForceY = DRAG_FACTOR * vy * totalVelocity;

      

      vx -= dragForceX;

      vy -= dragForceY;

  }

  // FIM NOVO: Arrasto

  // thrust

  if(thrusting && fuel > 0){

    const rad = angle * Math.PI / 180;

    vx += THRUST * Math.sin(rad) * dt * 60;

    vy -= THRUST * Math.cos(rad) * dt * 60;

    fuel -= FUEL_CONSUMPTION * dt * 60;

    addThrustParticles();

    // engine sound level roughly by throttle (clamp)

    const throttle = Math.min(1, Math.max(0, fuel>0 ? 0.6 : 0));

    sound.setEngineLevel(throttle);

  } else {

    // slight idle rumble when not thrusting

    sound.setEngineLevel(0.02);

  }

  fuel = Math.max(0, fuel);

  ship_x += vx * dt * 60;

  ship_y += vy * dt * 60;

  // bounds

  const canvasW = canvas.width/DPR, canvasH = canvas.height/DPR;

  if(ship_x < 0){ ship_x = 0; vx = Math.abs(vx)*0.3; }

  if(ship_x > canvasW){ ship_x = canvasW; vx = -Math.abs(vx)*0.3; }

  // collision

  const shipBottom = ship_y + 40;

  const platformTop = GROUND_LEVEL - TOWER_HEIGHT;

  const platformCenterX = (PLATFORM_ZONE[0] + PLATFORM_ZONE[1]) / 2;

  const isOverPlatform = ship_x >= PLATFORM_ZONE[0] && ship_x <= PLATFORM_ZONE[1];

  let collisionY = GROUND_LEVEL;

  if(isOverPlatform) collisionY = platformTop;

  if(shipBottom >= collisionY){

    ship_y = collisionY - 40;

    attempts++;

    // landing criteria (kept strict)

    const verticalSpeedOk = Math.abs(vy) < 2.5;

    const horizontalSpeedOk = Math.abs(vx) < 1.5;

    const angleMod = ((angle%360)+360)%360; // normalized

    const angleOk = Math.abs(angleMod) < 5 || Math.abs(angleMod - 360) < 5;

    const isSoftLanding = verticalSpeedOk && horizontalSpeedOk && angleOk;

    isCenteredOnPlatform = Math.abs(ship_x - platformCenterX) < 20;

    if(isSoftLanding){

      landed = true; successes++;

      if(isOverPlatform && isCenteredOnPlatform){

        message = 'CAPTURADO PELA MECHAZILLA! Pouso perfeito!';

        addSuccessParticles(ship_x, ship_y, 60, '#00FF00');

        // cinematic landing sound + shake

        sound.playLanding();

        camShake = 8;

      } else if(isOverPlatform){

        crashed = true;

        message = 'Pouso na plataforma, mas desalinhado!';

        addSuccessParticles(ship_x, ship_y, 30, '#FFFF00');

        sound.playLanding();

        camShake = 5;

      } else {

        message = 'Pouso suave no solo. Boa, supervisor!';

        addSuccessParticles(ship_x, ship_y, 40, GROUND_COLOR);

        sound.playLanding();

        camShake = 4;

      }

    } else {

      crashed = true;

      message = 'Falha! Starship explodiu - tiles perdidos!';

      addExplosionParticles(ship_x, ship_y);

      sound.playExplosion();

      camShake = 18;

    }

    state='end';

    setupControls();

    showMessage(message);

    // stop engine hum

    sound.setEngineLevel(0);

    // clear input flags

    for(let k in keys) keys[k]=false;

  }

  // update particles

  for(let i=particles.length-1;i>=0;i--){

    if(!particles[i].update(dt)) particles.splice(i,1);

  }

  trimParticles();

  // tip timer

  if(show_tips){ tips_timer -= dt; if(tips_timer <= 0) show_tips=false; }

}

/* ---------------------

   Render / draw loop

   --------------------- */

function draw(){

  // apply camera shake by small offset

  const shakeX = (Math.random()*2-1)*camShake * 0.3;

  const shakeY = (Math.random()*2-1)*camShake * 0.3;

  ctx.save();

  ctx.clearRect(0,0,canvas.width,canvas.height);

  ctx.translate(shakeX, shakeY);

  // background

  ctx.fillStyle = SKY_BLUE;

  ctx.fillRect(0,0,canvas.width/DPR,canvas.height/DPR);

  // stars

  drawStars();

  if(state==='difficulty'){

    // show nothing else (UI covers)

    ctx.restore();

    return;

  }

  drawTerrain();

  // particles behind ship

  for(let p of particles) p.draw(ctx, cameraY);

  const altitude = GROUND_LEVEL - ship_y;

  drawShip(ship_x, ship_y, angle, thrusting && fuel>0, altitude);

  // HUD drawn in DOM, but we can show tips

  if(show_tips && state==='game'){

    ctx.fillStyle = 'rgba(255,255,255,0.9)';

    ctx.font = '14px Arial';

    ctx.textAlign = 'center';

    ctx.fillText('DESAFIO: Pouse na torre Mechazilla! Use flip para manobra real da SpaceX.', canvas.width/DPR / 2, 80);

    ctx.fillText('Agora com arrasto aerodinâmico e sons aprimorados!', canvas.width/DPR / 2, 100);

    ctx.textAlign = 'left';

  }

  if(state==='end'){

    ctx.fillStyle = 'rgba(0,0,0,0.6)';

    ctx.fillRect(0,0,canvas.width/DPR,canvas.height/DPR);

    ctx.fillStyle = crashed ? '#FF0000' : '#00FF00';

    ctx.font = '24px Arial';

    ctx.textAlign = 'center';

    ctx.fillText(message, canvas.width/DPR/2, canvas.height/DPR/2 - 30);

    ctx.fillStyle = '#FFFFFF';

    ctx.font = '16px Arial';

    ctx.fillText('Pressione R ou toque no botão para reiniciar', canvas.width/DPR/2, canvas.height/DPR/2 + 30);

    ctx.textAlign='left';

    if(keys['r'] || keys['R']) resetToDifficulty();

  }

  ctx.restore();

}

/* ---------------------

   Game loop

   --------------------- */

function loop(now){

  if(!lastTime) lastTime = now;

  const dt = Math.min((now - lastTime) / 1000, 0.05); // cap dt for stability

  lastTime = now;

  update(dt);

  draw();

  requestAnimationFrame(loop);

}

/* -----------------------

   Small UI message overlay (Aprimorado com duração)

   ----------------------- */

function showMessage(txt, duration=0){

  messageOverlay.innerText = txt;

  messageOverlay.style.display='block';

  if(duration > 0){

      // Esconde após a duração, a menos que o estado seja 'end'

      setTimeout(()=>{ 

          if(state !== 'end') messageOverlay.style.display='none';

      }, duration);

  }

}

/* -----------------------

   Event bindings + setup

   ----------------------- */

function init(){

  DPR = Math.max(1, window.devicePixelRatio || 1);

  resizeCanvas();

  initStars();

  setupControls();

  requestAnimationFrame(loop);

}

window.addEventListener('load', init);

/* sound toggle handling */

soundToggle.addEventListener('change', ()=>{

  const on = soundToggle.checked;

  if(on){

    sound.init().then(()=>sound.setEnabled(true));

  } else {

    sound.setEnabled(false);

  }

});

/* NOVO: Eventos do Modal de URL de som */

soundUrlBtn.addEventListener('click', ()=>{

  soundModal.style.display='block';

  // Esconde o overlay principal

  messageOverlay.style.display='none'; 

});

closeSoundModalBtn.addEventListener('click', ()=>{

  soundModal.style.display='none';

});

loadSoundUrlsBtn.addEventListener('click', async ()=>{

  const urls = {

    engineUrl: document.getElementById('engUrl').value,

    rcsUrl: document.getElementById('rcsUrl').value,

    explosionUrl: document.getElementById('expUrl').value,

    landingUrl: document.getElementById('lanUrl').value,

  };

  

  // Verifica se alguma URL foi fornecida

  const hasUrls = Object.values(urls).some(url => url.trim() !== '');

  if(!hasUrls) { 

      showMessage('Nenhuma URL fornecida. Mantendo sons sintetizados.', 3000);

      soundModal.style.display='none'; 

      return; 

  }

  

  const ok = await sound.setExternalSounds(urls);

  

  if(ok) showMessage('Sons externos carregados (quando disponíveis).', 3000); 

  else showMessage('Falha ao carregar alguns sons. Usando fallback sintetizado.', 3000);

  

  soundModal.style.display='none';

});

/* keyboard for restart */

document.addEventListener('keydown', (e)=>{

  if(e.key === 'r' || e.key === 'R'){

    resetToDifficulty();

  }

});

/* initial resize call */

resizeCanvas();

/* final: draw HUD periodically (not heavy) */

setInterval(()=>{ drawHUD(); }, 120);

</script>

</body>

</html>

