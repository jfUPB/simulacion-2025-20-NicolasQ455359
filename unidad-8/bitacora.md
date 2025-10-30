# Evidencias de la unidad 8

## Actividad 01

Estuve viendo las performances de Alba G. Corral junto a Carles Viarnès en el Sónar+D 2020 y también la que hizo con Le Parody en el Teatro Principal de Zaragoza. Me llamó mucho la atención cómo lo visual y lo sonoro se conectan tan naturalmente, como si ambos estuvieran respirando al mismo ritmo. En la primera, los paisajes digitales parecían moverse al compás del piano y los sonidos ambientales: cuando las notas eran más suaves, las formas fluían lento, y cuando subía la intensidad, todo se iluminaba y tomaba más vida. Sentí como si las imágenes fueran una extensión del sonido.

En la presentación con Le Parody, en cambio, la conexión era mucho más rítmica y energética. Los visuales reaccionaban directamente a los beats, con colores que explotaban al mismo tiempo que las percusiones. Se notaba una sincronía increíble entre los movimientos del sonido y las formas generadas. Todo el tiempo cambiaban las texturas, los colores y las líneas, sin repetirse nunca igual, lo que me hizo pensar que estaba viendo un sistema generativo que respondía en tiempo real.

Lo que más me gustó fue la sensación de “liveness”, esa idea de que lo que estaba viendo no estaba pregrabado, sino que estaba pasando ahí mismo, frente a mis ojos. Saber que la artista estaba “tocando” los visuales como si fuera un instrumento me hizo sentir más conectada con la obra. Es bonito pensar que cada presentación es única, que el código, el sonido y el movimiento se mezclan para crear algo irrepetible. Me dejó con ganas de experimentar algo así, de poder crear visuales que reaccionen al sonido y que se sientan vivos, como si tuvieran alma propia.


## Actividad 02
Pieza musical elegida
Tema: “RKO” – Eladio Carrión

Concepto visual

Quiero un show de visuales que respire con el beat: energía de club/arena, estética neón y sensación de profundidad 3D. La idea es mezclar túneles volumétricos, nubes de partículas y rejillas láser que reaccionen al bajo, caja y platillos. Visualmente, todo gira alrededor de un núcleo pulsante (como “la firma” del track) y transiciones con glitch/chroma bloom en los drops. Quiero poder “tocar” los visuales en vivo (macros, cámara, crossfade de escenas) pero que el sistema también tenga automatismos musicales para que nunca se “caiga” si dejo que corra solo.

Inputs seleccionados y por qué

Audio del tema (análisis espectral + nivel RMS)

Por qué: me da bass/mid/treble para mapear bajo→golpe de cámara/túnel, medios→densidad/velocidad de partículas, agudos→detalles/flash.

Detección de beat por bandas (kick/snare/hat)

Por qué: cuantizo acciones importantes (cambios de escena, bursts, flashes) al ritmo real.

Controles de performance (teclado)

Por qué: necesito macro-knobs en vivo: color, densidad, speed, distorsión/chroma, brillo, feedback, saturación, glitch; además crossfade A/B y cámara (rot/zoom).

XY-Pad (tecla espacio + mouse)

Por qué: control expresivo rápido: X = color, Y = glitch para “pintar” momentos.

Auto Director (LFOs + energía)

Por qué: automatiza micro-movimientos y transiciones coherentes con la música, así no dependo 100% de mis manos.

Algoritmos / técnicas que usaré 

Análisis espectral con FFT + métricas (centroid, rolloff)
→ Mapea timbre/energía a color, brillo, elevación de rejillas y tamaño del núcleo.

Detección de beats por banda (umbrales adaptativos y “hold/decay”)
→ Dispara flashes, bursts de partículas, morphs de preset y cambios de escena cuantizados.

Sistemas de partículas 3D (“Shock Nebula”)
→ Nube energética con dirección por flow noise; responde a kick (impulsos) y hat (chispa).

Volumetric Tunnel (anillos/slices con deformación por espectro)
→ Sensación de velocidad y profundidad; el bajo ensancha/contrae el túnel.

Laser Grid (rejilla paramétrica animada)
→ Geometría rítmica para secciones con hi-hats/percusión marcada.

Flow fields (Perlin noise 3D)
→ Movimiento orgánico de partículas que no se siente “random”, sino fluido.

LFOs sincronizados al tempo estimado
→ Pequeños “respiradores” visuales entre beats para que todo tenga groove.

Post-FX en shader (bloom ligero, chroma offset, vignette, glitch)
→ Aspecto de visual de concierto, acentuando drops y builds.

Preset morphing (interpolación de macros en N beats)
→ Transiciones suaves de una paleta/estado a otro, alineadas con frases musicales.

## Actividad 03

## Codigo: 
```javascript
const AUDIO_PATH = "rko.mp3";  // Cambia si tu archivo se llama distinto
let USE_FX = true;             // PostFX (shader)
let SAFE_MODE = false;         // Fuerza modo “ligero” si tu equipo va justo

// ====== Audio ======
let mic, song, fft, amp, started=false, useMic=false;

// ====== UI ======
let startBtn, srcBtn;
let statusMsg = "Clic en ▶ Start / Audio";

// ====== Beat & Spectral ======
let BEAT={kick:false,snare:false,hat:false};
let cut={kick:0,snare:0,hat:0}, since={kick:99,snare:99,hat:99};
let hold={kick:12,snare:10,hat:8}, decay={kick:0.94,snare:0.94,hat:0.94};
let thr={kick:190,snare:165,hat:140};
let lastKickFrame=0, spectralCentroid=0, spectralRolloff=0;

// ====== Transport ======
let bpm=128, phase=0, framesPerBeat=60*60/bpm, beatCount=0;

// ====== Auto Director ======
let autoMode = true, energyEMA=0, dropLatch=false, lfoPhase=0;
let autoTarget = new Array(8).fill(0.5);

// ====== Mixer / Escenas ======
let sceneA=1, sceneB=2, cross=0.0;
let requestSceneA=null, requestSceneB=null;

// ====== Macros ======
let macros = new Array(8).fill(0.5);
const M = {COLOR:0, DENS:1, SPEED:2, DIST:3, BRIGHT:4, FB:5, SAT:6, GLITCH:7};
let glitchON=false;

// ====== Cámara / HUD ======
let camRotY=0, camZ=700;
let showHUD = true;

// ====== Paleta ======
let baseHue=300, hueDrift=0;

// ====== Objetos / Límites ======
let shards=[], blades=[], voxSlices=[];
let MAX_SHARDS=1500, MAX_BLADES=400, VOX_SLICES=28; // Ajusta si necesitas más/menos

// ====== PostFX ======
let fxShader, pg, flash=0;

// ====== XY-Pad ======
let xyActive=false;

// ====== Presets ======
const PRESETS=[
  [0.62,0.75,0.55,0.15,0.80,0.30,0.70,0.10],
  [0.48,0.40,0.30,0.05,0.60,0.50,0.45,0.00],
  [0.75,0.85,0.70,0.35,0.95,0.60,0.85,0.25],
  [0.30,0.60,0.20,0.65,0.50,0.80,0.35,0.70]
];
let morphTarget = PRESETS[0].slice();
let morphing=false, morphStartFrame=0, morphDurBeats=4;

// ====== Shader source ======
const VERT_SRC = `
precision mediump float;
attribute vec3 aPosition;
attribute vec2 aTexCoord;
varying vec2 vTex;
void main(){ vTex = aTexCoord; gl_Position = vec4(aPosition, 1.0); }`;

const FRAG_SRC = `
precision mediump float;
varying vec2 vTex;
uniform sampler2D tex0;
uniform float time, flash, sat, bloom, chroma, vign, glitch;
vec3 satAdjust(vec3 c, float s){
  float l = dot(c, vec3(0.2126,0.7152,0.0722));
  return mix(vec3(l), c, 0.5 + s);
}
void main(){
  vec2 uv = vTex;
  if (glitch>0.01){
    float g = step(0.5, fract(sin(uv.y*123.4+time*6.0)*345.6))*glitch*0.02;
    uv.x += g;
  }
  float a = chroma*0.004;
  vec3 col;
  col.r = texture2D(tex0, uv + vec2( a,0.0)).r;
  col.g = texture2D(tex0, uv + vec2( 0.0,0.0)).g;
  col.b = texture2D(tex0, uv + vec2(-a,0.0)).b;
  vec3 b1 = texture2D(tex0, uv+vec2(0.002,0.002)).rgb;
  vec3 b2 = texture2D(tex0, uv+vec2(-0.002,-0.002)).rgb;
  col += (b1+b2)*bloom*0.35;
  col = satAdjust(col, sat);
  vec2 d=uv-0.5; float v=1.0 - dot(d,d)*vign; col *= v;
  col += vec3(flash*0.006);
  gl_FragColor = vec4(col,1.0);
}`;

// ====== Preload: solo audio ======
function preload(){
  soundFormats('mp3','wav','ogg');
  try{
    song = loadSound(AUDIO_PATH,
      ()=>console.log('Audio OK:', AUDIO_PATH),
      (err)=>{ console.warn('No se pudo cargar el audio:', AUDIO_PATH, err); statusMsg='No se pudo cargar el audio. Prueba Mic con [M].'; }
    );
  }catch(e){
    console.warn('Error al preparar audio:', e);
    statusMsg='Error al preparar audio (revisa p5.sound y ruta).';
  }
}

let stage; // contenedor centrado
function setup() {
  // --- CSS inyectado para centrar en pantalla ---
  createElement('style', `
    html,body{margin:0;height:100vh;background:#000;overflow:hidden;}
    body{display:flex;align-items:center;justify-content:center;}
    #stage{position:relative;width:960px;height:600px;}
    #stage canvas{display:block;}
    .ui{position:absolute;}
  `);

  // --- contenedor centrado de 960×600 ---
  stage = createDiv().id('stage');

  // --- canvas fijo 960x600 (WEBGL) dentro del contenedor ---
  const cnv = createCanvas(960, 600, WEBGL);
  cnv.parent(stage);

  pixelDensity(1);
  colorMode(HSB,360,100,100,100);
  noStroke();

  // --- buffer pg del MISMO tamaño (¡importante!) ---
  pg = createGraphics(960, 600, WEBGL);
  pg.noStroke();
  pg.colorMode(HSB,360,100,100,100);

  // --- shader (con try/catch por compatibilidad) ---
  try { fxShader = createShader(VERT_SRC, FRAG_SRC); }
  catch(e){ console.warn('FX Shader OFF:', e); USE_FX=false; SAFE_MODE=true; }

  // --- UI dentro del contenedor (anclada al canvas) ---
  startBtn = createButton('▶ Start / Audio'); startBtn.parent(stage);
  startBtn.addClass('ui'); startBtn.style('left','10px'); startBtn.style('top','10px');
  startBtn.mousePressed(initAudio);

  srcBtn = createButton('Mic/File'); srcBtn.parent(stage);
  srcBtn.addClass('ui'); srcBtn.style('left','130px'); srcBtn.style('top','10px');
  srcBtn.mousePressed(toggleSource);

  // --- audio análisis ---
  fft = new p5.FFT(0.85, 1024);
  amp = new p5.Amplitude(); amp.smooth(0.92);

  // --- escenas / presets / modo ligero auto ---
  initShards(); initBlades(); initVox();
  for (let i=0;i<8;i++) macros[i]=PRESETS[0][i];
  autoTarget = PRESETS[2].slice();
  if (windowWidth < 1100 || /Mobile|Android|iP(ad|hone)/i.test(navigator.userAgent)) SAFE_MODE = true;
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


function initAudio(){
  if (started) return;
  userStartAudio();

  // Preferir FILE integrado
  if (song){
    try{
      if (!song.isPlaying()){
        song.loop();
        fft.setInput(song);
        amp.setInput(song);
        useMic=false;
        statusMsg='Reproduciendo archivo integrado.';
      }
    }catch(e){
      console.warn('No pudo iniciar el archivo integrado:', e);
      statusMsg='No se pudo iniciar el archivo. Prueba Mic con [M].';
    }
  } else {
    statusMsg='Archivo no disponible. Usa Mic con [M].';
  }

  // Prepara mic (por si alternas)
  mic=new p5.AudioIn();
  mic.start(()=>{}, (err)=>{ console.warn('Mic error:', err); });

  started=true; startBtn.html('Audio ✔');
}

function toggleSource(){
  if (!started) return;
  if (useMic){
    // pasar a FILE
    if (song){
      if (!song.isPlaying()) song.loop();
      fft.setInput(song); amp.setInput(song); useMic=false;
      statusMsg='Fuente: FILE integrado';
    } else {
      statusMsg='No hay FILE. Manteniendo MIC.';
    }
  } else {
    // pasar a MIC
    if (mic) { fft.setInput(mic); amp.setInput(mic); useMic=true; statusMsg='Fuente: MIC'; }
  }
}

function draw(){
  if (!started){
    background(0);
    resetMatrix(); translate(-width/2,-height/2);
    fill(255); textFont('monospace'); textSize(16);
    text('RKO Visual Rig — p5.js', 20, 60);
    text('Estado: '+statusMsg, 20, 90);
    text('Click en ▶ Start / Audio para habilitar sonido.', 20, 120);
    return;
  }

  framesPerBeat = 60*60/bpm;
  phase = (frameCount % framesPerBeat) / framesPerBeat;

  // Audio
  let spec=fft.analyze();
  let bass=fft.getEnergy('bass'), mid=fft.getEnergy('mid'), tre=fft.getEnergy('treble');
  let level=amp.getLevel();

  spectralCentroid = calcSpectralCentroid(spec);
  spectralRolloff  = calcSpectralRolloff(spec, 0.85);

  // Beats
  const wasKick = BEAT.kick;
  detectBand('kick', bass, thr.kick);
  detectBand('snare', mid, thr.snare);
  detectBand('hat', tre, thr.hat);

  if (BEAT.kick && !wasKick){ beatCount++; lastKickFrame=frameCount; flash=max(flash, 45); quantizedActions(); }
  if (BEAT.snare){ flash=max(flash, 20); }
  if (BEAT.hat)  { flash=max(flash, 10); }

  // LFOs (tempo)
  lfoPhase = (lfoPhase + (1/framesPerBeat)) % 1.0;
  const lfo1 = 0.5 + 0.5 * sin(TWO_PI * lfoPhase);
  const lfo2 = 0.5 + 0.5 * sin(TWO_PI * lfoPhase * 0.5);

  // Auto Director
  autoDirector(bass, mid, tre, lfo1, lfo2);

  // Morph presets
  if (morphing){
    let beats = (frameCount - morphStartFrame)/framesPerBeat;
    let t = constrain(beats/morphDurBeats, 0, 1);
    for (let i=0;i<8;i++) macros[i] = lerp(macros[i], morphTarget[i], 0.08);
    if (t>=1) morphing=false;
  }

  hueDrift = lerp(hueDrift, map(tre,0,255,0,70), 0.08);
  let H = (baseHue + hueDrift + (macros[M.COLOR]-0.5)*120) % 360;

  // Render a pg (mismo tamaño que canvas)
  pg.push();
  pg.clear();
  pg.background(0,0,0, 8 + macros[M.FB]*24);
  pg.ambientLight(0,0,30);
  pg.directionalLight(H,80,100, 0.2,0.2,-1);

  pg.translate(0,0,-camZ);
  pg.rotateY(camRotY);

  drawScene(pg, sceneA, (1.0-cross), spec, bass, mid, tre, H, level);
  drawScene(pg, sceneB, cross,        spec, bass, mid, tre, H, level);

  pg.pop();

  // PostFX (o modo seguro)
  if (USE_FX && !SAFE_MODE){
    shader(fxShader);
    fxShader.setUniform('tex0', pg);
    fxShader.setUniform('time', frameCount*0.016);
    fxShader.setUniform('flash', flash);
    fxShader.setUniform('sat',  macros[M.SAT]*1.2);
    fxShader.setUniform('bloom',macros[M.BRIGHT]*1.2);
    fxShader.setUniform('chroma',macros[M.DIST]*1.2 + (glitchON?0.08:0.0));
    fxShader.setUniform('vign', 0.85);
    fxShader.setUniform('glitch', glitchON? (0.2+macros[M.GLITCH]*0.8): macros[M.GLITCH]*0.2);
    rect(-width/2,-height/2,width,height);
    resetShader();
  } else {
    resetShader();
    imageMode(CENTER);
    image(pg, 0, 0, width, height); // sin shader: dibuja tal cual
  }

  flash*=0.90;

  if (showHUD) drawHUD(bass, mid, tre, level);
}

// ====== Auto Director ======
function autoDirector(bass, mid, tre, lfo1, lfo2){
  if (!autoMode) return;

  let energyNow = (bass*0.6 + mid*0.3 + tre*0.1)/255.0;
  energyEMA = lerp(energyEMA, energyNow, 0.08);

  cross = constrain(cross + (energyNow - 0.5) * 0.004 + (lfo2-0.5)*0.01, 0, 1);

  autoTarget[M.COLOR]  = 0.55 + 0.35*(lfo1-0.5) + 0.1*energyNow;
  autoTarget[M.DENS ]  = 0.65 + 0.25*(lfo2-0.5) + 0.1*energyNow;
  autoTarget[M.SPEED]  = 0.55 + 0.35*(lfo1-0.5) + 0.15*(spectralCentroid-0.5);
  autoTarget[M.DIST ]  = 0.25 + 0.45*(energyNow) + 0.2*(lfo2);
  autoTarget[M.BRIGHT] = 0.60 + 0.35*(energyNow) + 0.1*lfo1;
  autoTarget[M.FB   ]  = 0.30 + 0.40*(lfo2);
  autoTarget[M.SAT  ]  = 0.55 + 0.35*(lfo1);
  autoTarget[M.GLITCH]= 0.10 + 0.30*(lfo2) + 0.2*max(0, spectralRolloff-0.6);

  for (let i=0;i<8;i++) macros[i] = lerp(macros[i], autoTarget[i], 0.06);

  if (BEAT.kick){
    if (beatCount % 4 === 0){ flash = max(flash, 70); macros[M.BRIGHT]=min(1, macros[M.BRIGHT]+0.06); }
    if (beatCount % 8 === 0){ glitchON = (spectralCentroid>0.55) ? !glitchON : glitchON; }
    if (beatCount % 16 === 0){
      let next = floor(random(1,4));
      if (next===sceneA) next = (next % 3) + 1;
      requestSceneB = next; cross = min(1, cross + 0.25);
      morphTarget = PRESETS[floor(random(PRESETS.length))].slice();
      morphing = true; morphStartFrame = lastKickFrame;
    }
  }

  if (!dropLatch && energyNow > energyEMA+0.12 && BEAT.kick){
    macros[M.SPEED]=min(1, macros[M.SPEED]+0.05);
    macros[M.DENS ]=min(1, macros[M.DENS ]+0.04);
    dropLatch = true;
  }
  if (dropLatch && energyNow < energyEMA-0.08){
    macros[M.SPEED]=max(0, macros[M.SPEED]-0.04);
    dropLatch = false;
  }
}

// ====== Métricas espectrales ======
function calcSpectralCentroid(spec){
  let num=0, den=0;
  for (let i=0;i<spec.length;i++){ num += i*spec[i]; den += spec[i]; }
  if (den===0) return 0;
  return num/den/spec.length; // 0..1
}
function calcSpectralRolloff(spec, pct){
  let total=0; for (let v of spec) total+=v;
  let target=total*pct, acc=0;
  for (let i=0;i<spec.length;i++){ acc+=spec[i]; if (acc>=target) return i/spec.length; }
  return 1;
}

// ====== Beat detection ======
function detectBand(name, energy, threshold){
  since[name]++;
  let c=cut[name]||0;
  if (energy>c && energy>threshold && since[name]>(hold[name]||8)){
    BEAT[name]=true; cut[name]=energy*1.04; since[name]=0;
  }else{
    BEAT[name]=false;
    if (since[name]<=(hold[name]||8)) cut[name]*=(decay[name]||0.94);
    else cut[name]=max(cut[name]*(decay[name]||0.94), threshold);
  }
}

// ====== Acciones cuantizadas ======
function quantizedActions(){
  if (requestSceneA!=null){ sceneA=requestSceneA; requestSceneA=null; }
  if (requestSceneB!=null){ sceneB=requestSceneB; requestSceneB=null; }
}

// ====== Escenas ======
function drawScene(g, idx, w, spec, bass, mid, tre, H, level){
  if (w<=0.001) return;
  g.push();
  g.tint(255, 255*w);

  let dens = 0.5 + macros[M.DENS]*1.4;
  let speed= 0.4 + macros[M.SPEED]*2.2;

  if (idx===1) drawTunnel(g, spec, bass, mid, tre, H, dens, speed);
  if (idx===2) drawNebula(g, bass, mid, tre, H, dens, speed);
  if (idx===3) drawLaserGrid(g, bass, mid, tre, H, dens, speed);

  g.pop();
}

// --- Volumetric Tunnel ---
function initVox(){
  voxSlices=[];
  for (let i=0;i<VOX_SLICES;i++){
    voxSlices.push({z:-i*70, r:120+ i*2, rot:random(TWO_PI)});
  }
}
function drawTunnel(g, spec, bass, mid, tre, H, dens, speed){
  let segs=64;
  let baseR = 130 + map(bass,0,255,0,140)*(0.6+speed);

  g.blendMode(ADD);
  for (let s of voxSlices){
    s.z -= (3.2 + speed*2.0);
    if (s.z < -VOX_SLICES*70) { s.z = 0; s.rot = random(TWO_PI); }
    let ringBri = map(tre,0,255,40,100);
    g.push(); g.translate(0,0,s.z); g.rotateZ(s.rot);
    for (let i=0;i<segs;i+=2){
      let a = TWO_PI*(i/segs);
      let idx=floor(map(i,0,segs-1,0,spec.length-1));
      let e=spec[idx];
      let rad = baseR + map(e,0,255,0,90)*speed;
      let x=cos(a)*rad, y=sin(a)*rad;
      g.fill(H, 80, ringBri, 60);
      g.push(); g.translate(x,y,0); g.box(5, 22+map(e,0,255,0,40), 8); g.pop();
    }
    g.pop();
  }
  g.blendMode(BLEND);

  let segs2=48;
  for (let i=0;i<segs2;i++){
    let a=TWO_PI*(i/segs2);
    let idx=floor(map(i,0,segs2-1,0,spec.length-1));
    let e=spec[idx];
    let rad=baseR + map(e,0,255,0,120)*speed;
    let x=cos(a)*rad, y=sin(a)*rad;
    g.push(); g.translate(x,y,-40); g.rotateZ(a+HALF_PI);
    g.fill(H, 85, map(tre,0,255,55,100), 85);
    g.box(6, 10+map(e,0,255,0,80)*speed, 14);
    g.pop();
  }

  g.push();
  let core= 80 + map(spectralCentroid,0,1,0,50)*(0.6+speed) + map(bass,0,255,0,25);
  g.fill(H,90,100,60);
  g.sphere(core*0.25, 16, 16);
  g.pop();
}

// --- Shock Nebula ---
function initShards(){
  shards=[];
  for (let i=0;i<1200;i++) shards.push(new Shard());
}
class Shard{
  constructor(){ this.reset(true); }
  reset(rand=true){
    this.pos = rand? createVector(random(-700,700), random(-420,420), random(-1200,-120))
                   : createVector(random(-80,80), random(-60,60), random(-80,80));
    this.vel = p5.Vector.random3D().mult(random(0.25,1.6));
    this.s = random(4, 28);
    this.h = (baseHue+random(-30,30))%360;
    this.a = random(40, 90);
    this.life=random(160, 480);
  }
  update(flow, kick, hat, speed){
    if (kick) this.vel.add(p5.Vector.random3D().mult(random(1.2,4.2)*(0.8+speed)));
    if (hat) this.a = min(95, this.a+5);
    this.vel.add(flow);
    this.vel.limit(3.8*(0.6+speed));
    this.pos.add(this.vel);
    this.life--; this.a*=0.995;
    if (this.life<=0 || abs(this.pos.x)>1200||abs(this.pos.y)>900||this.pos.z>400||this.pos.z<-2000) this.reset(true);
  }
  draw(g, H, tre){
    g.push(); g.translate(this.pos.x, this.pos.y, this.pos.z);
    g.rotateZ(frameCount*0.01 + this.pos.x*0.001);
    g.rotateY(frameCount*0.012 + this.pos.y*0.001);
    g.fill(this.h, 85, map(tre,0,255,55,100), this.a);
    g.box(this.s*0.5, this.s, this.s*0.3);
    g.pop();
  }
}
function drawNebula(g, bass, mid, tre, H, dens, speed){
  let flow = createVector(
    (noise(frameCount*0.006)-0.5),
    (noise(frameCount*0.007+44)-0.5),
    (noise(frameCount*0.008+99)-0.5)
  ).mult(0.8 + map(mid,0,255,0,1.6)*(0.6+speed));

  if (BEAT.kick){
    for (let i=0;i<180;i++) shards.push(new Shard());
    if (shards.length>MAX_SHARDS) shards.splice(0, shards.length-MAX_SHARDS);
  }

  let take = int(1100 + dens*650 + map(spectralRolloff,0,1,0,160));
  take = min(take, shards.length);

  g.blendMode(ADD);
  for (let i=0;i<take; i++){
    shards[i].update(flow, BEAT.kick, BEAT.hat, speed);
    shards[i].draw(g, H, tre);
  }
  g.blendMode(BLEND);

  g.push();
  let core= 60 + map(bass,0,255,0,44)*(0.6+speed) + map(spectralCentroid,0,1,0,20);
  g.fill(H,90,100,42+map(tre,0,255,0,40));
  g.translate(0,0,-60);
  g.sphere(core*0.32, 14, 14);
  g.pop();
}

// --- Laser Grid ---
function initBlades(){
  blades=[];
  for (let i=0;i<300;i++) blades.push(new Blade());
}
class Blade{
  constructor(){ this.reset(true); }
  reset(rand=true){
    this.a=random(TWO_PI); this.r=random(140, 420);
    this.z=random(-800,-80);
    this.w=random(2,7); this.len=random(60,260);
    this.h=(baseHue+random(-40,40))%360; this.alpha=random(40,90);
    this.spin=random(-0.02,0.02); this.life=random(120,360);
    if (!rand){ this.r=random(60,160); this.z=random(-200,60); }
  }
  update(mid, snare, speed){
    this.a += this.spin * (1 + map(mid,0,255,0,1.2)*(0.6+speed));
    if (snare){ this.r += random(-14,14); this.alpha=min(95, this.alpha+12); }
    this.life--; if (this.life<=0) this.reset(true);
  }
  draw(g, tre){
    let x=cos(this.a)*this.r, y=sin(this.a)*this.r*0.62;
    g.push(); g.translate(x,y,this.z); g.rotateZ(this.a);
    g.fill(this.h, 80, map(tre,0,255,45,100), this.alpha);
    g.box(this.len, this.w, 4);
    g.pop();
  }
}
function drawLaserGrid(g, bass, mid, tre, H, dens, speed){
  let ringR = 220 + map(bass,0,255,0,100)*(0.6+speed);
  g.push(); g.noFill(); g.stroke(H,80,100,60); g.strokeWeight(2);
  for (let k=0;k<3;k++){ let rr = ringR*(1-k*0.2); g.ellipse(0,0, rr*2, rr*1.2); }
  g.noStroke(); g.pop();

  g.push();
  let rows = 10 + int(macros[M.DENS]*10);
  let cols = 18 + int(macros[M.DENS]*16);
  let hLift = map(spectralCentroid,0,1,0,160);
  for (let r=0;r<rows;r++){
    for (let c=0;c<cols;c++){
      let x = map(c,0,cols-1,-500,500);
      let y = map(r,0,rows-1,-300,300) - hLift;
      let z = -200 - (sin((c+r)*0.18 + frameCount*0.04)*60);
      g.push(); g.translate(x,y,z);
      let bri = map(tre,0,255,40,100);
      g.fill(H,85,bri, 70);
      g.box(4,4, 18 + map(mid,0,255,0,40)*(0.6+speed));
      g.pop();
    }
  }
  g.pop();

  if (BEAT.kick){
    for (let i=0;i<45;i++) blades.push(new Blade());
    if (blades.length>MAX_BLADES) blades.splice(0, blades.length-MAX_BLADES);
  }

  let take = int(240 + dens*200);
  for (let i=0;i<min(take, blades.length); i++){
    blades[i].update(mid, BEAT.snare, speed);
    blades[i].draw(g, tre);
  }

  g.push(); let core=36+map(tre,0,255,0,28)*(0.6+speed);
  g.fill(H, 90, 100, 60); g.sphere(core*0.25, 12, 12); g.pop();
}

// ====== HUD ======
function drawHUD(bass, mid, tre, level){
  resetMatrix(); translate(-width/2,-height/2,0);
  noStroke(); fill(0,0,0,70); rect(8,50, min(560, width-16),190,10);
  fill(0,0,100,95); textFont('monospace'); textSize(12);
  let src = useMic? 'Mic' : ((song && song.isPlaying())?'File':'—');
  text(
`AUTO:${autoMode?'ON':'OFF'}  FX:${(USE_FX && !SAFE_MODE)?'ON':'OFF'}  A:${sceneA}  B:${sceneB}  XFade:${nf(cross,1,2)}  SRC:${src}  BPM:${int(bpm)}
Macros: [Q..I]  C:${nf(macros[M.COLOR],1,2)} D:${nf(macros[M.DENS],1,2)} V:${nf(macros[M.SPEED],1,2)} 
         Dist:${nf(macros[M.DIST],1,2)} Br:${nf(macros[M.BRIGHT],1,2)} FB:${nf(macros[M.FB],1,2)}
         Sat:${nf(macros[M.SAT],1,2)} Glitch:${nf(macros[M.GLITCH],1,2)} (G:${glitchON?'ON':'OFF'})
K:${nf(bass,3,0)}  S:${nf(mid,3,0)}  H:${nf(tre,3,0)}  LVL:${nf(level,1,3)}
Centroid:${nf(spectralCentroid,1,3)}  Rolloff:${nf(spectralRolloff,1,3)}
[1/2/3]=A  [Z/X/C]=B  [ ]=xfade  [M]Src  [B]Burst  [N]Flash  [P]PresetMorph  [A]Auto  [F]FX
[Espacio]=XYPad(Q/I)  Arrows Cam  ThrK/S/H:${thr.kick}/${thr.snare}/${thr.hat}`,
    16,70,width-32,210
  );

  if (xyActive){
    fill(0,0,0,70); rect(width-220, 50, 200, 200, 12);
    fill(0,0,100,95);
    text('XY-Pad → Color (X), Glitch (Y)', width-208, 70);
    let bx=map(macros[M.COLOR],0,1,width-210,width-20);
    let by=map(macros[M.GLITCH],0,1,  230,  70);
    stroke(0,0,100,95); line(bx,70,bx,250); line(width-210,by,width-20,by);
    noStroke(); fill(200,80,100,95); circle(bx,by,12);
  }
}

// ====== Input ======
function keyPressed(){
  if (key==='A' || key==='a') autoMode=!autoMode;
  if (key==='F' || key==='f') USE_FX=!USE_FX;
  if (key==='1') requestSceneA=1; if (key==='2') requestSceneA=2; if (key==='3') requestSceneA=3;
  if (key==='z'||key==='Z') requestSceneB=1; if (key==='x'||key==='X') requestSceneB=2; if (key==='c'||key==='C') requestSceneB=3;
  if (key==='[') cross=max(0, cross-0.06); if (key===']') cross=min(1, cross+0.06);
  if (key==='M'||key==='m') toggleSource(); if (key==='H'||key==='h') showHUD=!showHUD;
  if (key==='G'||key==='g') glitchON=!glitchON; if (key==='N'||key==='n') flash=100;

  if (key==='B'||key==='b'){
    flash=110; lastKickFrame=frameCount;
    macros[M.BRIGHT]=min(1, macros[M.BRIGHT]+0.18);
    macros[M.DIST]=min(1, macros[M.DIST]+0.12);
  }

  let step=(keyIsDown(SHIFT)?0.02:0.06);
  if (key==='Q') macros[M.COLOR]=constrain(macros[M.COLOR]+step,0,1);
  if (key==='W') macros[M.DENS ]=constrain(macros[M.DENS ]+step,0,1);
  if (key==='E') macros[M.SPEED]=constrain(macros[M.SPEED]+step,0,1);
  if (key==='R') macros[M.DIST ]=constrain(macros[M.DIST ]+step,0,1);
  if (key==='T') macros[M.BRIGHT]=constrain(macros[M.BRIGHT]+step,0,1);
  if (key==='Y') macros[M.FB   ]=constrain(macros[M.FB   ]+step,0,1);
  if (key==='U') macros[M.SAT  ]=constrain(macros[M.SAT  ]+step,0,1);
  if (key==='I') macros[M.GLITCH]=constrain(macros[M.GLITCH]+step,0,1);

  if (keyCode===LEFT_ARROW)  camRotY-=0.06;
  if (keyCode===RIGHT_ARROW) camRotY+=0.06;
  if (keyCode===UP_ARROW)    camZ=max(300, camZ-40);
  if (keyCode===DOWN_ARROW)  camZ=min(1300, camZ+40);

  if (key==='P'||key==='p'){
    let idx=int(random(PRESETS.length));
    morphTarget=PRESETS[idx].slice();
    morphing=true; morphStartFrame=lastKickFrame;
  }

  if (key===';') bpm=max(60, bpm-1);
  if (key==="'") bpm=min(180, bpm+1);
}
function keyReleased(){
  let step=(keyIsDown(SHIFT)?0.02:0.06);
  if (key==='Q') macros[M.COLOR]=constrain(macros[M.COLOR]-step,0,1);
  if (key==='W') macros[M.DENS ]=constrain(macros[M.DENS ]-step,0,1);
  if (key==='E') macros[M.SPEED]=constrain(macros[M.SPEED]-step,0,1);
  if (key==='R') macros[M.DIST ]=constrain(macros[M.DIST ]-step,0,1);
  if (key==='T') macros[M.BRIGHT]=constrain(macros[M.BRIGHT]-step,0,1);
  if (key==='Y') macros[M.FB   ]=constrain(macros[M.FB   ]-step,0,1);
  if (key==='U') macros[M.SAT  ]=constrain(macros[M.SAT  ]-step,0,1);
  if (key==='I') macros[M.GLITCH]=constrain(macros[M.GLITCH]-step,0,1);
}

function mousePressed(){ if (keyIsDown(32)){ xyActive=true; } } // espacio
function mouseReleased(){ xyActive=false; }
function mouseDragged(){
  if (xyActive){
    let x = constrain(mouseX/(width), 0, 1);
    let y = constrain(1 - mouseY/(height), 0, 1);
    macros[M.COLOR]=x;
    macros[M.GLITCH]=y;
  }
}

// ====== Utils ======
function initVox(){
  voxSlices=[];
  for (let i=0;i<VOX_SLICES;i++){
    voxSlices.push({z:-i*70, r:120+ i*2, rot:random(TWO_PI)});
  }
}
function initShards(){
  shards=[];
  for (let i=0;i<900;i++) shards.push(new Shard());
}
function initBlades(){
  blades=[];
  for (let i=0;i<260;i++) blades.push(new Blade());
}
```

### Enlace

https://editor.p5js.org/NicolasQ455359/sketches/JvT1uyaRE


### Autoevaluación: 5

Realice todas las actividades propuestas siguiendo la rubrica, se creó el sistema de visualizacón desde cero y cumpliendo con todo lo requerido







