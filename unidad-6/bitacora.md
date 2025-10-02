# Evidencias de la unidad 6

## Actividad 01.
<img width="474" height="474" alt="image" src="https://github.com/user-attachments/assets/f14613e5-f3bd-46ba-aaa4-95655ec0684e" />

. Por qué me llama la atención: me gusta cómo esas líneas, aunque son muchas, tienen una dirección orgánica. Hay zonas donde se separan, otras donde se juntas; parecen corrientes visuales que se doblan suavemente. Esa tensión entre orden y variación me parece muy poderosa.

<img width="474" height="474" alt="image" src="https://github.com/user-attachments/assets/e8db6a53-9550-4d96-bb71-da2811b0338c" />

.Por qué me llama la atención: aquí se acentúa más la densidad, el contraste y el movimiento. Las líneas parecen viento, polvo, o una fuerza invisible dibujando en el espacio. Esa textura me inspira porque siento que “escucho” el movimiento mientras miro.

## Actividad 02.

### 1. ¿Qué es una fuerza de dirección (steering force)?
Una steering force es básicamente una fuerza que le da dirección a un agente dentro de una simulación. No es solo moverse porque sí, sino que permite que el agente “decida” hacia dónde ir, como si tuviera un objetivo o un comportamiento particular. Es la que hace que un objeto no se mueva en línea recta todo el tiempo, sino que pueda girar, seguir un punto, evitar un obstáculo o alinearse con otros.

### 2. ¿Qué diferencia tiene este tipo de fuerza con las que ya hemos estudiado en el contexto de la simulación de agentes?
La diferencia principal es que las fuerzas que habíamos visto antes son más básicas, como mover un agente aplicando velocidad, aceleración o atracción a un punto fijo. En cambio, la steering force no solo mueve al agente, sino que también le da un sentido de comportamiento. Es decir, en lugar de ser un movimiento “mecánico”, se convierte en algo más parecido a una decisión: seguir a otro, mantenerse dentro de un grupo, esquivar choques, etc.

### 3. ¿Qué relación tiene la steering force con Craig Reynolds y su trabajo en simulación de comportamiento animal?
Craig Reynolds fue quien desarrolló el modelo de los “boids”, que son simulaciones de bandadas de pájaros o bancos de peces. Allí, las steering forces son la base para que cada agente siga reglas simples (como alinearse, separarse o cohesionarse) y, cuando todos los agentes lo hacen, aparece un comportamiento colectivo muy parecido al de los animales reales. En otras palabras, sin las steering forces no se lograría esa sensación natural y emergente que caracteriza el trabajo de Reynolds.

## Actividad 03. 
Actividad 03 — Bitácora
### 1. ¿Cómo está guardado el campo de flujo y cómo se generan sus vectores?
El campo es una cuadrícula. En código es un arreglo indexado por filas y columnas (en p5 se maneja como 1D con index = x + y * cols, pero mentalmente es un 2D).
Cada celda guarda un vector dirección (ángulo + magnitud ≈ 1).
En el ejemplo base del libro, ese vector se genera con Perlin noise: se toma noise(xoff, yoff, zoff), se convierte a ángulo, y de ahí sale el vector unitario de la celda. Así, celdas vecinas apuntan parecido y aparece ese “flujo suave” característico.

### 2. ¿Cómo usa el agente el campo para calcular su steering force?
Mi lectura (versión simple):
Ubico mi celda: con mi posición (pos.x, pos.y) la mapeo a índices de la grilla (col = floor(pos.x / resol), row = floor(pos.y / resol)).
Leo el vector deseado de esa celda (el “viento” local).
Lo convierto en “deseo”: ajusto su magnitud a maxspeed.
Steering = desired – vel: la diferencia entre lo que quiero (desired) y lo que llevo (vel) es mi fuerza de dirección.
Limito esa fuerza con maxforce para que el giro no sea brusco, y la aplico como aceleración.
En corto: el agente “huele” el vector del campo donde está parado y corrige su rumbo suavemente hacia allá.

### 3. Parámetros clave que identifiqué

- Resolución del campo (res, tamaño de cada celda).
- maxspeed del agente (qué tan rápido puede ir).
- maxforce del agente (qué tan fuerte puede corregir el rumbo).

### 4. Modificación que hice (y qué pasó)
Cambio grande 1: reemplacé el noise por un campo en remolino (vórtice)
En vez de usar noise() para el ángulo, generé un vector tangencial alrededor del centro. Eso convierte el flujo en un giro continuo (como corrientes orbitando).
Efecto observado (comportamiento):
Los agentes empezaron a orbitar el centro, dibujando trayectorias curvas y bastante limpias.
El movimiento se vuelve coherente globalmente (todos giran en el mismo sentido), menos errático que con noise.
Si subo maxspeed, las órbitas se alargan y sienten más “energía”; si bajo maxforce, tardan más en alinearse con la curva.

<img width="800" height="600" alt="flowfield_capture" src="https://github.com/user-attachments/assets/42467749-ad6e-45d1-9a52-9ed69eba6461" />

## Codigo: 
```javascript
let flow;
let vehicles = [];
let capturer = null;   // CCapture instance (opcional)
let recording = false; // estado de grabación GIF

// Parámetros clave
const RES = 24;        // resolución del campo (tamaño de celda)
const NUM_VEH = 250;   // cantidad de agentes
const MAX_SPEED = 3.0; // velocidad máxima
const MAX_FORCE = 0.08;// fuerza máxima

function setup() {
  createCanvas(800, 600);
  pixelDensity(1);

  // Campo de flujo tipo vórtice
  flow = new FlowField(RES);

  // Crear vehículos
  for (let i = 0; i < NUM_VEH; i++) {
    vehicles.push(new Vehicle(random(width), random(height), MAX_SPEED, MAX_FORCE));
  }

  // UI: botones
  const btnPNG = createButton("📸 Capturar PNG");
  btnPNG.mousePressed(() => saveCanvas("flowfield_capture", "png"));

  const btnStartGIF = createButton("🎬 Iniciar GIF (opcional)");
  btnStartGIF.mousePressed(startGifCapture);

  const btnStopGIF = createButton("⏹️ Detener y guardar GIF");
  btnStopGIF.mousePressed(stopGifCapture);

  background(250);
}

function draw() {
  // Redibuja fondo muy suave para efecto “persistencia” (opcional)
  noStroke();
  fill(250, 250, 250, 10);
  rect(0, 0, width, height);

  // Mostrar el campo muy tenue (si quieres visualizar la grilla, descomenta)
  // flow.show();

  // Actualizar y dibujar agentes
  for (let v of vehicles) {
    v.follow(flow);
    v.update();
    v.edges();
    v.show();
  }

  // Si estamos grabando GIF (requiere CCapture.js)
  if (recording && capturer) {
    // Captura el frame actual del canvas
    capturer.capture(canvas);
  }
}

class FlowField {
  constructor(resolution) {
    this.res = resolution;
    this.cols = floor(width / this.res);
    this.rows = floor(height / this.res);
    this.field = new Array(this.cols * this.rows);
    this.generateVortex();
  }

  index(x, y) {
    return x + y * this.cols;
  }

  generateVortex() {
    const cx = width * 0.5;
    const cy = height * 0.5;

    for (let y = 0; y < this.rows; y++) {
      for (let x = 0; x < this.cols; x++) {
        const index = this.index(x, y);

        // centro de la celda
        const px = x * this.res + this.res * 0.5;
        const py = y * this.res + this.res * 0.5;

        // vector radial
        const rx = px - cx;
        const ry = py - cy;

        // vector tangencial (perpendicular a radial)
        let tx = -ry;
        let ty =  rx;

        // normalizar para evitar magnitud 0
        const m = Math.hypot(tx, ty) || 1;
        tx /= m;
        ty /= m;

        const v = createVector(tx, ty);
        v.setMag(1); // magnitud unitaria
        this.field[index] = v;
      }
    }
  }

  lookup(pos) {
    const col = constrain(floor(pos.x / this.res), 0, this.cols - 1);
    const row = constrain(floor(pos.y / this.res), 0, this.rows - 1);
    const index = this.index(col, row);
    return this.field[index].copy();
  }

  show() {
    stroke(0, 40);
    strokeWeight(1);
    for (let y = 0; y < this.rows; y++) {
      for (let x = 0; x < this.cols; x++) {
        const index = this.index(x, y);
        const v = this.field[index];
        push();
        translate(x * this.res + this.res * 0.5, y * this.res + this.res * 0.5);
        rotate(v.heading());
        line(0, 0, this.res * 0.5, 0);
        pop();
      }
    }
  }
}

class Vehicle {
  constructor(x, y, maxspeed, maxforce) {
    this.pos = createVector(x, y);
    this.vel = p5.Vector.random2D();
    this.acc = createVector(0, 0);

    this.maxspeed = maxspeed;
    this.maxforce = maxforce;

    // apariencia
    this.size = 3.5;
    this.trailAlpha = 80;
  }

  applyForce(f) {
    this.acc.add(f);
  }

  follow(flow) {
    const desired = flow.lookup(this.pos); // vector del campo local
    desired.setMag(this.maxspeed);

    // steering = desired - velocity
    const steer = p5.Vector.sub(desired, this.vel);
    steer.limit(this.maxforce);
    this.applyForce(steer);
  }

  update() {
    this.vel.add(this.acc);
    this.vel.limit(this.maxspeed);
    this.pos.add(this.vel);
    this.acc.mult(0);
  }

  edges() {
    // wrap-around
    if (this.pos.x < -this.size) this.pos.x = width + this.size;
    if (this.pos.x > width + this.size) this.pos.x = -this.size;
    if (this.pos.y < -this.size) this.pos.y = height + this.size;
    if (this.pos.y > height + this.size) this.pos.y = -this.size;
  }

  show() {
    // trazo sutil
    stroke(0, this.trailAlpha);
    strokeWeight(1);
    point(this.pos.x, this.pos.y);

    // cuerpo (triangulito orientado)
    push();
    translate(this.pos.x, this.pos.y);
    rotate(this.vel.heading());
    noStroke();
    fill(0, 180);
    triangle(this.size, 0, -this.size * 1.2, this.size * 0.8, -this.size * 1.2, -this.size * 0.8);
    pop();
  }
}

function startGifCapture() {
  if (typeof CCapture === "undefined") {
    alert("Para grabar GIF necesitas incluir CCapture.js en index.html.\nAun así puedes usar 📸 Capturar PNG.");
    return;
  }
  if (!capturer) {
    capturer = new CCapture({
      format: "gif",
      workersPath: "",   // si usas CDN, puede ir vacío
      framerate: 30,
      verbose: true,
      quality: 20
    });
  }
  if (!recording) {
    recording = true;
    capturer.start();
    console.log("GIF recording started");
  }
}

function stopGifCapture() {
  if (capturer && recording) {
    recording = false;
    capturer.stop();
    capturer.save(); // descarga el GIF
    console.log("GIF saved");
  }
}
```

## Actividad 04
### 1. Las tres reglas

- Separación (Separation)
Objetivo: no pegarnos. Mantener “espacio personal” para que el grupo no colapse.

Cómo calcula la fuerza: reviso quiénes están muy cerca (d < radio). Por cada vecino saco un vector lejos de él (miPos − suPos) y lo peso más fuerte si está pegadísimo. Promedio esos vectores, lo llevo a mi maxspeed y hago steering = desired − vel, limitado por maxforce.

- Alineación (Alignment)
Objetivo: apuntar más o menos en la misma dirección que mis vecinos.

Cómo calcula la fuerza: promedia la velocidad de los vecinos cercanos. Ese promedio lo uso como desired (con maxspeed). La fuerza es steering = desired − vel, limitada por maxforce.

- Cohesión (Cohesion)
Objetivo: no perder al grupo; moverme hacia el centro de mis vecinos.

Cómo calcula la fuerza: promedia la posición de los vecinos (centroide), arma un seek hacia ese punto (lleva a maxspeed) y hace steering = desired − vel, limitado por maxforce.

### 2. Parámetros clave que identifiqué
perceptionRadius (y a veces ángulo): hasta dónde “veo” para contar a alguien como vecino.

Pesos de reglas: wSep, wAli, wCoh (qué tanto influye cada comportamiento en la mezcla).

maxspeed y maxforce: topo de velocidad y giro (suaviza o vuelve brusco el movimiento).

### 3. Mi modificación y lo que pasó
Hice dos pruebas, pero reporto como principal la del target global (todos además siguen un objetivo con un seek):

Modificación: añadí un target controlado por mí y sumé su fuerza de seek a la mezcla (con peso propio).

Ajustes que usé: wSep = 1.2, wAli = 1.0, wCoh = 0.9, perception = 70, maxspeed = 3.0, maxforce = 0.08, wTarget = 0.7.

Efecto observado: el enjambre se organiza y “persigue” el punto. Se nota una cabeza que va hacia el objetivo y una cola que se estira; cuando muevo el target, el grupo dobla con fluidez sin perder la forma (Separación evita choques, Cohesión mantiene la masa y Alineación hace que apunten parecido).

<img width="900" height="600" alt="flocking_capture" src="https://github.com/user-attachments/assets/da94d2dc-550f-4655-b4e5-93bf91d522eb" />

<img width="900" height="600" alt="flocking_capture (1)" src="https://github.com/user-attachments/assets/c0924e4a-9ddd-4ff7-b45f-f9e6561894f6" />


## Actividad 05.
La idea central de este proyecto fue crear una pieza de arte generativo interactivo inspirada en la canción “Mbappé” de Eladio Carrión, utilizando algoritmos de flow fields para dirigir el movimiento de los agentes visuales. Lo que quería mostrar era que una canción no solo puede escucharse, sino también verse y tocarse, convirtiendo la experiencia musical en un espectáculo visual que reacciona en tiempo real.

Quise que las visuales fueran dinámicas, llamativas y coherentes con la energía de la canción, con colores que cambian de acuerdo a los distintos momentos de la música y efectos visuales que acompañan los beats principales (bajos, cajas, hi-hats). Además, incorporé interactividad para que el usuario pueda “jugar” con la obra, disparando ondas, cometas o haces de luz al ritmo, como si estuviera tocando un instrumento visual junto a la pista.

### Codigo: 
```javascript
const AUDIO_FILE = "Eladio Carrión - Mbappe (Video Oficial)  SEN2 KBRN VOL. 2.mp3";

// ---------- Audio & Beat Grid ----------
let sound, fft, amp;
let peakKick, peakSnare, peakHat;
let bpm = 130, beatDur = 60/130;
let songStartSec = null, taps = [], phase0 = 0, latencyMs = 0, lastPhase = 0;

// ---------- Flow Field ----------
let res = 22, cols, rows, zoff = 0;
let field;
const NUM_PART = 1400;
let particles = [];
let vortices = [];
let warps = [];

// ---------- Visual Events ----------
let shockwaves = [];
let comets = [];
let beams = [];
let scheduled = [];

// ---------- Paletas + color engine (MUY dinámico) ----------
/*
 Bancos:
 0 Deep Ocean   1 Black&Gold   2 Neon Pop   3 Sunset Heat
 4 Arctic Mint  5 Purple Candy
*/
const BANKS = [
  [{h:208,s:58,b:18},{h:210,s:40,b:36},{h:200,s:30,b:64},{h:195,s:16,b:92},{h:185,s:10,b:98}],
  [{h: 46,s:78,b:88},{h: 38,s:60,b:52},{h:  0,s: 0,b:10},{h: 50,s:94,b:96},{h: 10,s:18,b:22}],
  [{h:280,s:78,b:86},{h:200,s:90,b:74},{h:135,s:92,b:90},{h: 58,s:95,b:98},{h:320,s:72,b:92}],
  [{h: 10,s:84,b:90},{h: 24,s:88,b:92},{h: 40,s:86,b:94},{h:355,s:72,b:82},{h:  0,s: 0,b: 8}],
  [{h:168,s:42,b:90},{h:186,s:38,b:98},{h:158,s:52,b:78},{h:140,s:30,b:64},{h:  0,s: 0,b:10}],
  [{h:298,s:66,b:90},{h:324,s:72,b:96},{h:286,s:72,b:84},{h:260,s:42,b:92},{h:  0,s: 0,b:10}],
];
let bankIdx = 1;        // arranca gold/black (luego rota)
let hueRotate = 0;      // rotación suave de tono (con medios)
let accentHue = 48;     // acento que cambia en cada beat
let colorBeatPulse = 0; // brillo por beat

// --- helper corregido (antes se llamaba HSB y chocaba con la constante global) ---
function hsbCol(h, s, b, a = 255) {
  colorMode(HSB, 360, 100, 100, 255);
  const c = color((h % 360 + 360) % 360, constrain(s, 0, 100), constrain(b, 0, 100), a);
  colorMode(RGB, 255);
  return c;
}

function pickBank(i){ return BANKS[((i%BANKS.length)+BANKS.length)%BANKS.length]; }
function palPick(seed){
  const p = pickBank(bankIdx);
  const k = Math.floor(((seed%p.length)+p.length)%p.length);
  const c = p[k];
  return hsbCol(c.h + hueRotate, c.s, c.b, 255);
}
function palAccented(seed, swing){
  const base = palPick(seed);
  colorMode(RGB,255);
  const rb=red(base), gb=green(base), bb=blue(base);
  colorMode(HSB,360,100,100,255);
  const hb=hue(base), sb=saturation(base), bbri=brightness(base);
  const h = lerp(hb, (accentHue + swing) % 360, 0.35);
  const s = constrain(sb + 10*colorBeatPulse, 0, 100);
  const b = constrain(bbri + 12*colorBeatPulse, 0, 100);
  const out = color(h,s,b,255); // en modo HSB aquí
  colorMode(RGB,255);
  return out;
}

// ---------- Formación: SOLO "rose" (rosa polar) ----------
let formation = { type: "rose", anchors: [], mix: 1.0, showOverlay: true };
const FORM_CYCLE = ["rose","off"];

// ---------- Estado visual ----------
let holdTrails = false;

// ================= PRELOAD =================
function preload(){ sound = loadSound(AUDIO_FILE); }

// ================= SETUP =================
function setup(){
  createCanvas(960, 600);
  pixelDensity(1);
  colorMode(RGB,255);
  background(6);

  fft = new p5.FFT(0.12, 1024);
  amp = new p5.Amplitude();

  peakKick  = new p5.PeakDetect(20, 140,   0.12, 10);
  peakSnare = new p5.PeakDetect(140, 2000, 0.12, 10);
  peakHat   = new p5.PeakDetect(4000,12000,0.11, 10);

  rebuildField(); refreshField();
  for (let i=0;i<NUM_PART;i++) particles.push(new Particle());

  formation.anchors = buildAnchors(); // rosa desde inicio

  overlay();
}

function overlay(){
  push(); noStroke(); fill(255,220); textAlign(CENTER,CENTER); textSize(13);
  text(
"ESPACIO ▶/❚❚  •  T tap tempo  •  [ / ] BPM ±  •  ENTER re-alinear\n" +
"Pads: Z Boom  •  X Comets  •  C Sprint  •  B Beams\n" +
"Formación: R (toggle rose)  •  O overlay on/off  •  1..6 bancos de color  •  H hold  •  P PNG",
    width/2, height/2
  ); pop();
}

// ================= DRAW =================
function draw(){
  drawBackground();

  // --- AUDIO ---
  let bass=0, mid=0, treble=0;
  if (sound && sound.isLoaded() && sound.isPlaying()){
    fft.analyze();
    bass   = fft.getEnergy("bass");
    mid    = fft.getEnergy("mid");
    treble = fft.getEnergy("treble");

    peakKick.update(fft);
    peakSnare.update(fft);
    peakHat.update(fft);

    if (peakKick.isDetected)  onKick(bass);
    if (peakSnare.isDetected) onSnare(mid);
    if (peakHat.isDetected)   onHat(treble);
  }

  // --- Beat & Scheduler ---
  const phase = getBeatPhase();
  const crossed = (phase < lastPhase); lastPhase = phase;
  if (crossed) onBeat();
  runScheduler();

  // --- Color engine dinámico ---
  hueRotate = (hueRotate + map(mid,0,255, 0.05, 0.25)) % 360;

  // Evolución del campo
  zoff += 0.004 + map(mid,0,255, 0, 0.012);

  // decays
  vortices = vortices.filter(v => (--v.life)>0);
  warps    = warps.filter(w => (--w.life)>0);
  for (let i=shockwaves.length-1;i>=0;i--){ shockwaves[i].update(); if (shockwaves[i].dead()) shockwaves.splice(i,1); }
  for (let i=comets.length-1;i>=0;i--){ comets[i].update(); comets[i].draw(); if (comets[i].dead()) comets.splice(i,1); }
  for (let i=beams.length-1;i>=0;i--){ beams[i].update(); beams[i].draw(); if (beams[i].dead()) beams.splice(i,1); }

  refreshFieldPartial();

  // --- Partículas ---
  const beatPulse = 0.7 + 0.3*(1.0 - Math.abs(phase-0.5)*2);
  const aBase = (50 + 180*colorBeatPulse) * beatPulse;
  for (let i=0;i<particles.length;i++){
    const p = particles[i];
    p.follow(field, res);

    // anclaje a rosa (si ON)
    if (formation.type==="rose" && formation.anchors.length>0 && formation.mix>0.01){
      const idx = i % formation.anchors.length;
      const target = formation.anchors[idx];
      const toT = p5.Vector.sub(target, p.pos);
      const d = toT.mag() + 0.0001;
      toT.setMag( map(d, 0, 200, 1.0, 0.18) );
      p.applyForce(toT.mult(1.10 * formation.mix));
    }

    p.update(); p.edges();
    p.show(aBase, bass, mid, treble);
  }

  for (let s of shockwaves) s.draw();

  drawFormationOverlay(colorBeatPulse); // rosa en neón multicapa

  drawHUD(phase);
}

// ================= Eventos musicales =================
function onBeat(){
  colorBeatPulse = 1.0;
  const jumps = [0, 30, 45, 60, 90, 120, 150, 210];
  accentHue = (accentHue + random(jumps)) % 360;
  if (random() < 0.10) bankIdx = (bankIdx + 1) % BANKS.length;
  blendMode(ADD); noStroke(); fill(255, 10); rect(0,0,width,height); blendMode(BLEND);
}
function onKick(bass){
  const c = createVector(width*0.5 + random(-60,60), height*0.55 + random(-40,40));
  shockwaves.push(new Shockwave(c, map(bass,0,255, 2.0, 4.8)));
  warps.push({pos:c.copy(), strength: 1.2, life: 34});
}
function onSnare(mid){
  if (random()<0.5) beams.push(new Beam(createVector(width*0.5, height*0.5), 8+int(random(6))));
}
function onHat(treble){
  const n = int(map(treble,120,255, 2, 8));
  for (let i=0;i<n;i++) comets.push(new Comet(createVector(random(width), random(height))));
}

// ================= Beat / Scheduler =================
function getBeatPhase(){
  if (!sound || !sound.isPlaying() || songStartSec===null) return 0;
  beatDur = 60/bpm;
  const t = (millis() + latencyMs)/1000 - songStartSec + phase0;
  return ((t/beatDur)%1 + 1) % 1;
}
function scheduleNext(fn, division=1){
  if (!sound || !sound.isPlaying() || songStartSec===null){ fn(); return; }
  const tNow = (millis()+latencyMs)/1000 - songStartSec + phase0;
  const grid = beatDur*division;
  const tNext = Math.ceil(tNow/grid)*grid;
  scheduled.push({ timeSec: tNext, fn });
}
function runScheduler(){
  if (!sound || !sound.isPlaying() || songStartSec===null) return;
  const tNow = (millis()+latencyMs)/1000 - songStartSec + phase0;
  for (let i=scheduled.length-1;i>=0;i--){
    if (tNow >= scheduled[i].timeSec){ scheduled[i].fn(); scheduled.splice(i,1); }
  }
}

// ================= Flow Field =================
function rebuildField(){ cols=floor(width/res); rows=floor(height/res); field=new Array(cols*rows); }
function fieldIndex(x,y){ return x + y*cols; }
function lookupField(pos){
  const col = constrain(floor(pos.x/res),0,cols-1);
  const row = constrain(floor(pos.y/res),0,rows-1);
  const v = field[fieldIndex(col,row)];
  return v ? v.copy() : createVector(1,0);
}
function updateFieldAt(col,row){
  const k=0.085, xoff=col*k, yoff=row*k;
  const turb = 2.6;
  let angle = noise(xoff,yoff,zoff) * TWO_PI * turb;

  const px = col*res + res*0.5, py = row*res + res*0.5;

  for (let v of vortices){ const dx=px-v.x, dy=py-v.y, d2=dx*dx+dy*dy+1; angle += v.strength * 1200.0 / d2; }
  for (let w of warps){ const dx=px-w.pos.x, dy=py-w.pos.y, d2=dx*dx+dy*dy+1; angle += w.strength * 900.0 / d2; }
  for (let s of shockwaves){
    const dx=px-s.pos.x, dy=py-s.pos.y; const d = Math.sqrt(dx*dx+dy*dy);
    const ring = exp(-pow((d - s.r)/s.width, 2));
    angle += s.spin * ring * 0.8;
  }

  const vec = p5.Vector.fromAngle(angle); vec.setMag(1);
  return vec;
}
function refreshField(){ for (let y=0;y<rows;y++) for (let x=0;x<cols;x++) field[fieldIndex(x,y)] = updateFieldAt(x,y); }
function refreshFieldPartial(){
  const y = frameCount % rows;
  for (let x=0;x<cols;x+=2) field[fieldIndex(x,y)] = updateFieldAt(x,y);
}

// ================= Partículas =================
class Particle{
  constructor(){
    this.pos = createVector(random(width), random(height));
    this.vel = p5.Vector.random2D();
    this.acc = createVector(0,0);
    this.maxspeed = 3.6;
    this.seed = random(1000);
  }
  applyForce(f){ this.acc.add(f); }
  follow(field,res){
    const col = constrain(floor(this.pos.x/res),0,cols-1);
    const row = constrain(floor(this.pos.y/res),0,rows-1);
    const desired = field[fieldIndex(col,row)].copy().setMag(this.maxspeed);
    const steer = p5.Vector.sub(desired, this.vel).limit(0.1);
    this.applyForce(steer);
  }
  update(){ this.vel.add(this.acc).limit(this.maxspeed); this.pos.add(this.vel); this.acc.mult(0); }
  edges(){ if (this.pos.x<0) this.pos.x=width; if (this.pos.x>width) this.pos.x=0; if (this.pos.y<0) this.pos.y=height; if (this.pos.y>height) this.pos.y=0; }
  show(aBase,bass,mid,treble){
    const swing = map(treble,0,255, 0, 40);
    const base = palAccented(this.seed + (noise(this.pos.x*0.002,this.pos.y*0.002,frameCount*0.01)*5|0), swing);
    const sw = 0.7 + 1.6*map(bass,0,255,0,1) + 0.9*map(mid,0,255,0,1);
    blendMode(ADD);
    stroke(red(base),green(base),blue(base), aBase);
    strokeWeight(sw);
    point(this.pos.x, this.pos.y);
    blendMode(BLEND);
  }
}

// ================= Event Visuals =================
class Shockwave{
  constructor(pos, strength=3){
    this.pos=pos.copy(); this.r=10; this.strength=strength; this.width=30;
    this.spin = random([-1,1])*(0.8+random()); this.life=95;
  }
  update(){ this.r += 6*this.strength; this.width*=0.99; this.life--; }
  draw(){
    blendMode(ADD); noFill();
    const c = palAccented(frameCount*0.2, 0);
    stroke(red(c),green(c),blue(c), 120);
    strokeWeight(2.2); circle(this.pos.x,this.pos.y,this.r*2);
    blendMode(BLEND);
  }
  dead(){ return this.life<=0 || this.r>max(width,height)*1.25; }
}
class Comet{
  constructor(p){ this.pos=p.copy(); this.vel=p5.Vector.random2D().mult(random(2,4)); this.life=46; this.seed=random(1000); }
  update(){
    const f = lookupField(this.pos).setMag(2.0);
    this.vel.add(f).limit(6.5);
    this.pos.add(this.vel);
    this.life--;
    if (this.pos.x<0) this.pos.x=width; if (this.pos.x>width) this.pos.x=0;
    if (this.pos.y<0) this.pos.y=height; if (this.pos.y>height) this.pos.y=0;
  }
  draw(){
    const c = palAccented(this.seed+frameCount*0.3, 20);
    blendMode(ADD); stroke(red(c),green(c),blue(c), 220); strokeWeight(2.0); point(this.pos.x, this.pos.y); blendMode(BLEND);
  }
  dead(){ return this.life<=0; }
}
class Beam{
  constructor(pos, count=10){ this.pos=pos.copy(); this.count=count; this.life=26; this.angle=random(TWO_PI); }
  update(){ this.life--; this.angle += 0.03; }
  draw(){
    blendMode(ADD);
    const len = map(this.life, 26,0, width*0.55, width*0.1);
    const c = palAccented(frameCount*0.2, 0);
    stroke(red(c),green(c),blue(c), 70); strokeWeight(2);
    for (let i=0;i<this.count;i++){
      const a = this.angle + i*(TWO_PI/this.count);
      line(this.pos.x, this.pos.y, this.pos.x + cos(a)*len, this.pos.y + sin(a)*len);
    }
    blendMode(BLEND);
  }
  dead(){ return this.life<=0; }
}

// ================= Rosa (anchors) + Overlay =================
function buildAnchors(){
  const pts = [];
  const cx = width*0.5, cy = height*0.55;
  const a = min(width,height)*0.30, k = 5; // 5 pétalos
  const M = 1000;
  for (let i=0;i<M;i++){
    const t = map(i,0,M, 0, TWO_PI);
    const r = a * abs(sin(k*t));
    const x = cx + r*cos(t);
    const y = cy + r*sin(t);
    pts.push(createVector(x,y));
  }
  // nervaduras radiales
  for (let j=0;j<k*2;j++){
    const ang = j*PI/k;
    for (let m=0;m<70;m++){
      const rr = a*0.1 + (a*0.9)*(m/70);
      pts.push(createVector(cx + rr*cos(ang), cy + rr*sin(ang)));
    }
  }
  return pts;
}

function drawFormationOverlay(beatPulse=1){
  if (!formation.showOverlay || formation.type!=="rose" || formation.mix<0.05) return;

  const c = palAccented(frameCount*0.6, 25 + 25*beatPulse);
  const baseA = 40 + 110*beatPulse*formation.mix;

  const layers = [
    {w: 9, a: baseA*0.18},
    {w: 5, a: baseA*0.32},
    {w: 2, a: baseA*0.52}
  ];

  blendMode(ADD);
  noFill();
  for (const L of layers){
    stroke(red(c),green(c),blue(c), L.a);
    strokeWeight(L.w);
    beginShape();
    for (let i=0;i<formation.anchors.length;i+=3){
      const p = formation.anchors[i]; vertex(p.x, p.y);
    }
    endShape(CLOSE);
  }
  blendMode(BLEND);
}

// ================= Interacción =================
let dragStart = null;
function mousePressed(){ dragStart = createVector(mouseX, mouseY); }
function mouseDragged(){ if (dragStart){ push(); blendMode(ADD); stroke(255,80); strokeWeight(2); line(dragStart.x,dragStart.y, mouseX,mouseY); blendMode(BLEND); pop(); } }
function mouseReleased(){
  if (!dragStart) return;
  const end = createVector(mouseX, mouseY);
  const v = p5.Vector.sub(end, dragStart);
  const center = dragStart.copy();
  scheduleNext(()=>{ warps.push({pos:center, strength:1.3, life: 36}); shockwaves.push(new Shockwave(center, constrain(v.mag()/40, 1.8, 4.5))); }, 0.5);
  dragStart = null;
}
function mouseWheel(e){ res = constrain(res + (e.delta>0 ? 2 : -2), 12, 50); rebuildField(); refreshField(); }

function keyPressed(){
  if (key===' '){
    if (getAudioContext().state !== 'running') getAudioContext().resume();
    if (sound && sound.isLoaded()){
      if (sound.isPlaying()) sound.pause();
      else { sound.loop(); if (songStartSec===null) songStartSec=(millis()+latencyMs)/1000; }
    }
  }
  if (key==='P'||key==='p') saveCanvas('mbappe_rose','png');
  if (key==='H'||key==='h') holdTrails = !holdTrails;

  // bancos de color (1..6)
  if (key>='1' && key<='6'){ bankIdx = int(key)-1; }

  // Tap tempo y sync
  if (key==='T'||key==='t'){
    const now = millis()+latencyMs; taps.push(now); if (taps.length>8) taps.shift();
    if (taps.length>=2){ let s=0; for (let i=1;i<taps.length;i++) s+=(taps[i]-taps[i-1]); const nbpm = 60000/(s/(taps.length-1));
      if (nbpm>60 && nbpm<200){ bpm = lerp(bpm,nbpm,0.85); beatDur=60/bpm; } }
  }
  if (key==='['){ bpm=max(60,bpm-1); beatDur=60/bpm; }
  if (key===']'){ bpm=min(200,bpm+1); beatDur=60/bpm; }
  if (keyCode===ENTER){
    if (songStartSec!==null){ const t=(millis()+latencyMs)/1000 - songStartSec; const beats=Math.round(t/beatDur); phase0=beats*beatDur - t; }
  }
  if (key===';') latencyMs -= 5;
  if (key==="'") latencyMs += 5;

  // PADS (cuantizados a corchea)
  if (key==='Z'||key==='z') scheduleNext(()=>{ const c=createVector(width*0.5,height*0.55); shockwaves.push(new Shockwave(c,3.2)); warps.push({pos:c.copy(),strength:1.5,life:44}); }, 0.5);
  if (key==='X'||key==='x') scheduleNext(()=>{ for (let i=0;i<36;i++){ const pos=createVector(random(width*0.2,width*0.8), random(height*0.25,height*0.85)); comets.push(new Comet(pos)); } }, 0.5);
  if (key==='C'||key==='c') scheduleNext(()=>spawnSprintSweep(height*0.55+random(-60,60), random([0,1])), 0.5);
  if (key==='B'||key==='b') scheduleNext(()=>beams.push(new Beam(createVector(width*0.5,height*0.5), 8+int(random(6)))), 0.5);

  // Formación (toggle) y overlay
  if (key==='R'||key==='r'){ formation.type = (formation.type==="rose") ? "off" : "rose"; formation.mix = (formation.type==="rose")?1.0:0.0; }
  if (key==='O'||key==='o') formation.showOverlay = !formation.showOverlay;
}

// Sprint Sweep
function spawnSprintSweep(y, side=0){
  const fromX = side===0 ? 0 : width;
  const toX   = side===0 ? width : 0;
  const steps = 14;
  for (let i=0;i<steps;i++){
    const x = lerp(fromX, toX, i/(steps-1));
    warps.push({pos:createVector(x, y + sin(i*0.6)*18), strength: 1.1, life: 26});
  }
}

// ================= Fondo limpio =================
function drawBackground(){
  if (!holdTrails){ noStroke(); fill(6,7,12, 26); rect(0,0,width,height); }
  // viñeta + bruma
  push();
  const g = drawingContext.createRadialGradient(width*0.5,height*0.55,0, width*0.5,height*0.55, max(width,height)*0.85);
  g.addColorStop(0,'rgba(0,0,0,0)'); g.addColorStop(1,'rgba(0,0,0,0.35)');
  drawingContext.fillStyle = g; noStroke(); rect(0,0,width,height); pop();
  colorBeatPulse = lerp(colorBeatPulse, 0.35, 0.12);
}

// ================= HUD =================
function drawHUD(phase){
  noStroke(); fill(255,170); textSize(11);
  text(`BPM ${bpm.toFixed(1)} • Phase ${phase.toFixed(2)} • Res ${res}px • Bank ${bankIdx+1} • Rose ${formation.type==="rose"?"ON":"OFF"}`, 12, height-12);
}

// ================= Helpers =================
function refreshField(){ for (let y=0;y<rows;y++) for (let x=0;x<cols;x++) field[fieldIndex(x,y)] = updateFieldAt(x,y); }
function rebuildField(){ cols=floor(width/res); rows=floor(height/res); field=new Array(cols*rows); }
```

<img width="960" height="600" alt="mbappe_rose" src="https://github.com/user-attachments/assets/db6fb8a7-1c60-4ccf-b9a2-01c5e7d8349f" />

<img width="960" height="600" alt="mbappe_rose (1)" src="https://github.com/user-attachments/assets/ee46b890-8bf7-46ba-8e9d-dcda97dc7796" />

<img width="960" height="600" alt="mbappe_rose (2)" src="https://github.com/user-attachments/assets/3b3972d2-c69c-4698-b722-4d84b706c208" />

<img width="960" height="600" alt="mbappe_rose (3)" src="https://github.com/user-attachments/assets/ea055edb-0995-44a4-8be3-1811b5a6abe7" />

<img width="960" height="600" alt="mbappe_rose (4)" src="https://github.com/user-attachments/assets/bfab96a5-6fcf-44dc-aeb1-8131143b5ca4" />

<img width="960" height="600" alt="mbappe_rose (5)" src="https://github.com/user-attachments/assets/afaa85b4-a9c1-4e0e-ac18-c4718f3eda39" />

### enlace: 
[Sketch en p5.js – NicolasQ455359 (ID: 8y58u1bk7)](https://editor.p5js.org/NicolasQ455359/sketches/8y58u1bk7)


### Autoevalución:
Todas las actividades propuestas fueron realizadas y estan en la bitacora detalladamente con codigo e imagenes, la actividad 5 de arte generativo cumplió con lo propuesto utilizando flow fields y añadiendo interactividad como si se pudieran tocar las visuales.
NOTA PROPUESTA: 5.0

Defensa de la nota:

Actividad 01: Seleccioné imágenes del artista Tyler Hobbs, expliqué qué me llamaba la atención de ellas y qué me inspiraba de su trabajo.

Actividad 02: Investigué y respondí qué es una steering force, cómo se diferencia de otras fuerzas y la relación con el trabajo de Craig Reynolds.

Actividad 03: Analicé el algoritmo de flow fields, identifiqué la estructura de datos usada, expliqué cómo los agentes calculan la fuerza de dirección, listé los parámetros clave y realicé modificaciones en el código con evidencia.

Actividad 04: Estudié el comportamiento de flocking, describí con mis palabras las tres reglas (separación, alineación, cohesión), expliqué los parámetros clave, realicé modificaciones en el código y documenté los cambios con su efecto.

Actividad 05: Desarrollé una pieza de arte generativo interactivo inspirada en la canción Mbappé de Eladio Carrión, usando flow fields, sincronizada con la música, con interactividad en tiempo real que permite “tocar” las visuales como un instrumento. Documenté la idea principal, el proceso y los resultados.
























