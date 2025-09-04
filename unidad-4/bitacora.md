# Evidencias de la unidad 4

## Explicación conceptual de la obra

* ¿Qué concepto de la unidad 4 y cómo lo aplicaste en la obra?
> Tu respuesta aquí:
> Ángulos y rotaciones: Se usó translate() y rotate() para que las varillas de los móviles se dibujen respecto a su propio pivote.

heading() y vectores: la orientación de algunos elementos (como las ondas que emergen en un beat) se relaciona con la dirección de movimiento.

Coordenadas polares: aplicadas implícitamente en el cálculo de péndulos: cada brazo del móvil se define por un ángulo y una longitud (x = r*cos(θ), y = r*sin(θ)).

Funciones sinusoidales: el movimiento de los péndulos responde a ecuaciones sinusoidales, con frecuencia, amplitud y fase controladas por gravedad, longitud y amortiguamiento.

* ¿Qué concepto de la unidad 3 y cómo lo aplicaste en la obra?
> Tu respuesta aquí:
>El concepto central que use es el de Motion 101 con fuerzas acumulativas. Un objeto en movimiento no solo debe actualizarse con su velocidad, sino que puede recibir fuerzas externas que se acumulan en su aceleración antes de cada actualización. Luego, en cada frame, el objeto aplica todas las fuerzas, actualiza su velocidad y finalmente su posición.
```javascript
vel.add(acc);
pos.add(vel);
acc.mult(0); // reset después de aplicar
```

* ¿Qué concepto de la unidad 2 y cómo lo aplicaste en la obra?
> Tu respuesta aquí:
> Vectores: Se usaron para representar posiciones (anchor, p1, p2), velocidades y aceleraciones. Gracias a operaciones como add(), sub(), mag(), normalize() y limit(), pude controlar el movimiento suave y realista de los móviles.
Ejemplo: cuando un beat genera un impulso, se usa un vector dirección → se normaliza → se multiplica por una magnitud → se suma a la aceleración angular.

Motion 101:
El esquema clásico velocidad += aceleración; posición += velocidad; aceleración = 0; fue aplicado a las varillas de los péndulos. Así, cada móvil acumula fuerzas (gravedad, ruido Perlin, beat) en su aceleración y luego actualiza su movimiento de manera orgánica.

* ¿Qué concepto de la unidad 1 y cómo lo aplicaste en la obra?
> Tu respuesta aquí:
> De la Unidad 1 tomé principalmente el concepto de aleatoriedad controlada en el arte generativo.
Aleatoriedad en la composición inicial: Cada vez que se presiona la tecla R (resembrar), se genera una nueva semilla aleatoria (randomSeed, noiseSeed), cambiando la posición de los móviles, su longitud y sus colores.

Aleatoriedad en los colores: Los móviles escogen su tonalidad a partir de una paleta definida, pero con variaciones aleatorias dentro de un rango.

Aleatoriedad en el movimiento orgánico: Se usó ruido Perlin para introducir pequeñas variaciones en el ángulo de los móviles.

## ¿Cómo resolviste la interacción?
> Tu respuesta aquí:
> La interacción de la obra se resolvió mediante el micrófono, que actúa como disparador de beats visuales, y la participación activa del usuario con el mouse y el teclado. El sonido genera impulsos, halos y ondas en los móviles, mientras que el mouse permite arrastrar, empujar y lanzar ondas manualmente. El teclado ofrece control creativo sobre modos, efectos y configuraciones.

## Enlace a la obra en el editor de p5.js

[Aquí está mi obra]([URL](https://editor.p5js.org/NicolasQ455359/sketches/y-CvZjNhg))

## Código de la obra 

``` js
let seed;
let mobiles = [];
let ripples = [];
let dust = [];

// Paletas base (HSB)
let palettes = [
  {name:"Solar",   base:[20, 40, 0]},
  {name:"Aqua",    base:[170,190,210]},
  {name:"Aurora",  base:[280,320,10]},
  {name:"Selva",   base:[90,120,140]},
  {name:"Magenta", base:[330,350,10]},
];
let palette;

// ------- Parámetros globales / visuales -------
let USE_FULLSCREEN = true;
let DENSITY_PER_16K = 0.55; // se redefine por preset
let MIN_DIST = 160;
let TRAIL_FADE = 18;
let SHOW_TRAIL = true;
let SHOW_CONNECTIONS = false;
let SHOW_BEATS = true; // por defecto encendido

let DUST_COUNT = 90;
let DUST_ALPHA = 26;

let glowDecay = 0.07;              // velocidad a la que se apaga el halo
let connectionMaxD = 200;          // radio para conexiones
let connectionMaxPerNode = 1;      // máx. vecinos por móvil
let strokeAlphaMult = 0.55;        // opacidad global de líneas

// --------- Audio (SOLO SONIDO) ----------
let mic, audioReady = false;
let levelSmooth = 0;
let levelAlpha = 0.08;     // suavizado exponencial

// Detección simple de "beat"
let beatThreshold = 0.11;  // <- Ajustable con el control abajo-derecha
let beatCooldown = 260;    // ms entre beats
let lastBeatMs = -9999;

// Flujo para polvo decorativo
let flowZ = 0, flowScale = 0.0025, flowStrength = 0.09;

// Drag/selección
let dragging = null; // {mobile, which:1|2, prevMouse}
let DOUBLECLICK_MS = 280;
let lastClickMs = -9999;

// --------- Presets ---------
const PRESETS = {
  calm: {
    density: 0.55, minDist: 160,
    dustCount: 90, dustAlpha: 26,
    trailFade: 18, showTrail: true,
    connections: false, beats: true,
    glowDecay: 0.07, connectionMaxD: 200,
    connectionMaxPerNode: 1, strokeAlphaMult: 0.55
  },
  expressive: {
    density: 0.9, minDist: 140,
    dustCount: 160, dustAlpha: 34,
    trailFade: 12, showTrail: true,
    connections: true, beats: true,
    glowDecay: 0.03, connectionMaxD: 260,
    connectionMaxPerNode: 3, strokeAlphaMult: 1.0
  }
};

function applyPreset(name='calm'){
  const p = PRESETS[name];
  DENSITY_PER_16K = p.density;
  MIN_DIST = p.minDist;
  DUST_COUNT = p.dustCount;
  DUST_ALPHA = p.dustAlpha;
  TRAIL_FADE = p.trailFade;
  SHOW_TRAIL = p.showTrail;
  SHOW_CONNECTIONS = p.connections;
  SHOW_BEATS = p.beats;
  glowDecay = p.glowDecay;
  connectionMaxD = p.connectionMaxD;
  connectionMaxPerNode = p.connectionMaxPerNode;
  strokeAlphaMult = p.strokeAlphaMult;
}

// --------- Utilidades ---------
function reseed(){ seed = Math.floor(Math.random()*1e9); randomSeed(seed); noiseSeed(seed); }
function randomFrom(a){ return a[int(random(a.length))]; }
function pickPalette(){ palette = randomFrom(palettes); }
function sizeFromWindow(){ return USE_FULLSCREEN ? {w:windowWidth,h:windowHeight} : {w:1000,h:700}; }

// Colocación aleatoria con distancia mínima (rechazo)
function placeAnchors(n, minDist){
  const pts = [];
  let attempts = n * 60;
  while (pts.length < n && attempts-- > 0){
    const margin = 60;
    const p = createVector(random(margin, width-margin), random(margin, height-margin));
    let ok = true;
    for (let q of pts){ if (p.dist(q) < minDist){ ok=false; break; } }
    if (ok) pts.push(p);
  }
  return pts;
}

// --------- Setup / Layout ---------
function setup(){
  const s = sizeFromWindow();
  createCanvas(s.w, s.h);
  pixelDensity(1);
  colorMode(HSB, 360, 100, 100, 100);
  textFont('monospace');

  reseed(); pickPalette();
  applyPreset('calm'); // inicia en modo CALM

  mic = new p5.AudioIn();

  composeScene();
}

function windowResized(){
  const s = sizeFromWindow();
  resizeCanvas(s.w, s.h);
  composeScene();
}

function composeScene(){
  mobiles = []; ripples = []; dust = [];

  // cantidad según área (controlada por preset)
  const area = width * height;
  const target = constrain(floor((area/16000) * DENSITY_PER_16K), 6, 36);

  // anclajes distribuidos con separación
  const anchors = placeAnchors(target, MIN_DIST);

  // longitudes relativas
  const len1Min = height*0.08, len1Max = height*0.16;
  const len2Min = height*0.06, len2Max = height*0.12;

  for (let an of anchors){
    mobiles.push(new Mobile(
      an,
      random(len1Min, len1Max), random(-PI/10, PI/10),
      random(len2Min, len2Max), random(-PI/7,  PI/7)
    ));
  }

  for (let i=0;i<DUST_COUNT;i++){
    dust.push(new Dust(createVector(random(width), random(height))));
  }
}

// --------- Interacción ---------
function mousePressed(){
  const now = millis();
  const isDouble = (now - lastClickMs) < DOUBLECLICK_MS;
  lastClickMs = now;

  if (!audioReady){
    userStartAudio();
    mic.start(()=> audioReady = true);
    return;
  }

  // ¿tocaste un peso? (p2 tiene prioridad)
  let best = null; let bestD = Infinity;
  for (let m of mobiles){
    const p2 = m.p2(), d2 = dist(mouseX,mouseY,p2.x,p2.y);
    const p1 = m.p1(), d1 = dist(mouseX,mouseY,p1.x,p1.y);
    if (d2 < bestD && d2 < 22){ best = {m,which:2}; bestD = d2; }
    else if (d1 < bestD && d1 < 22){ best = {m,which:1}; bestD = d1; }
  }

  // Si el clic fue sobre el control de umbral, lo manejamos aparte
  if (thresholdUIClick(mouseX, mouseY)) return;

  if (best){
    if (isDouble){
      // doble clic: ondas
      const src = (best.which===2) ? best.m.p2() : best.m.p1();
      for (let k=0;k<3;k++) ripples.push(new Ripple(src.x, src.y, best.m.hue, 1.6 + k*0.4));
    }else{
      // iniciar drag + impulso pequeño
      dragging = {mobile:best.m, which:best.which, prevMouse:createVector(mouseX,mouseY)};
      best.m.flash(1.0);
      best.m.kick(mouseX, mouseY, 0.03, best.which);
    }
  }
}

function mouseDragged(){
  if (thresholdUIDrag(mouseX, mouseY)) return; // ajustar umbral
  if (!dragging) return;

  const m = dragging.mobile;
  const p = createVector(mouseX, mouseY);
  const prev = dragging.prevMouse;
  const delta = p.copy().sub(prev);
  dragging.prevMouse = p.copy();

  const hinge = (dragging.which===2) ? m.p2() : m.p1();
  const tangent = createVector(-(p.y - hinge.y), (p.x - hinge.x)).normalize();
  const accel = constrain(delta.dot(tangent)*0.002, -0.05, 0.05);
  if (dragging.which===1) m.aVel1 += accel; else m.aVel2 += accel;

  m.flash(0.6);
}

function mouseReleased(){
  dragging = null;
  thresholdUIRelease();
}

function keyPressed(){
  if (key==='M'||key==='m'){ applyPreset('calm'); composeScene(); }
  if (key==='X'||key==='x'){ applyPreset('expressive'); composeScene(); }
  if (key==='R'||key==='r'){ reseed(); pickPalette(); composeScene(); }
  if (key==='P'||key==='p') saveCanvas('moviles_sonoros_solo_sonido','png');
  if (key==='C'||key==='c') SHOW_CONNECTIONS = !SHOW_CONNECTIONS;
  if (key==='T'||key==='t') SHOW_TRAIL = !SHOW_TRAIL;
  if (key==='B'||key==='b') SHOW_BEATS = !SHOW_BEATS;
}

// --------- Draw ---------
function draw(){
  // fondo (trail opcional)
  if (SHOW_TRAIL){
    noStroke(); fill(0,0,0,TRAIL_FADE); rect(0,0,width,height);
  } else {
    background(0);
  }

  // nivel de audio suavizado
  let level = audioReady ? mic.getLevel() : 0;
  levelSmooth = lerp(levelSmooth, level, levelAlpha);

  // animación del flujo (solo para polvo)
  flowZ += 0.004;

  // conexiones por detrás
  if (SHOW_CONNECTIONS) drawConnections();

  // actualizar/dibujar móviles
  let maxEnergy = -1, energetic = null;
  for (let m of mobiles){
    // (SIN VIENTO) → nos quedamos con ruido sutil + dinámica propia
    m.applyNoise(flowZ);
    m.update();
    m.display();
    const energy = abs(m.aVel1) + abs(m.aVel2);
    if (energy > maxEnergy){ maxEnergy = energy; energetic = m; }

    // polvo desde el extremo
    if (random() < 0.3) {
      let e = m.p2();
      dust[int(random(dust.length))]?.warp(e.x, e.y);
    }
  }

  // “Beat” visual (flash + ondas) si está activado
  if (SHOW_BEATS && audioReady){
    const now = millis();
    if (levelSmooth > beatThreshold && (now - lastBeatMs) > beatCooldown){
      lastBeatMs = now;
      energetic?.flash(1.2);
      energetic?.kick(energetic.p2().x + random(-20,20), energetic.p2().y + random(-20,20), 0.02, 2);
      const src = energetic.p2();
      ripples.push(new Ripple(src.x, src.y, energetic.hue, 1.6));
    }
  }

  // polvo decorativo
  for (let d of dust){
    let theta = noise(d.pos.x*flowScale, d.pos.y*flowScale, flowZ) * TWO_PI * 4;
    let desired = p5.Vector.fromAngle(theta).setMag(flowStrength);
    d.applyForce(p5.Vector.sub(desired, d.vel).limit(0.03));
    d.update(); d.edgesWrap(); d.display();
  }

  // ondas
  for (let i=ripples.length-1;i>=0;i--){
    ripples[i].update(); ripples[i].display();
    if (ripples[i].dead) ripples.splice(i,1);
  }

  drawHUD(levelSmooth);
  drawThresholdUI(levelSmooth); // UI del umbral (abajo-derecha)
}

// --------- Conexiones limitadas y sutiles ---------
function drawConnections(){
  for (let i=0;i<mobiles.length;i++){
    const ai = mobiles[i].p2();
    const cand = [];
    for (let j=0;j<mobiles.length;j++){
      if (i===j) continue;
      const aj = mobiles[j].p2();
      const d = dist(ai.x, ai.y, aj.x, aj.y);
      if (d < connectionMaxD) cand.push({j, d});
    }
    cand.sort((a,b)=>a.d-b.d);
    const take = cand.slice(0, connectionMaxPerNode);

    for (const {j,d} of take){
      const a = ai;
      const b = mobiles[j].p2();
      const e = (abs(mobiles[i].aVel1)+abs(mobiles[i].aVel2)+abs(mobiles[j].aVel1)+abs(mobiles[j].aVel2))*0.5;
      const alpha = map(d, 40, connectionMaxD, 50, 0, true) * strokeAlphaMult;
      stroke((mobiles[i].hue+mobiles[j].hue)/2, 60, 95, alpha);
      strokeWeight(map(e, 0, 0.3, 0.4, 1.6, true));
      line(a.x, a.y, b.x, b.y);
    }
  }
}

// --------- Clases ---------
class Mobile{
  constructor(anchor, len1, ang1, len2, ang2){
    this.anchor = anchor.copy();
    this.len1 = len1; this.len2 = len2;

    this.angle1 = ang1; this.angle2 = ang2;
    this.aVel1 = 0; this.aVel2 = 0;
    this.aAcc1 = 0; this.aAcc2 = 0;

    this.g = 0.11;
    this.damping = 0.996;
    this.maxAngVel = 0.09;

    this.hue = this.pickHue();
    this.glow = 0;
  }
  pickHue(){ const base = randomFrom(palette.base); return (base + random(-14,14) + 360) % 360; }

  flash(intensity=1){ this.glow = constrain(this.glow + intensity, 0, 2.2); }

  kick(mx, my, strength=0.03, which=2){
    const target = (which===2) ? this.p2() : this.p1();
    const dir = createVector(mx, my).sub(target).heading();
    const sgn = sin(dir) >= 0 ? 1 : -1;
    if (which===1) this.aVel1 += sgn * strength;
    else           this.aVel2 += sgn * strength;
    this.flash(0.7);
  }

  applyNoise(t){
    // deriva orgánica suave (sin viento)
    const n1 = noise(this.anchor.x*0.0015, t) - 0.5;
    const n2 = noise(this.anchor.y*0.0015, t + 100) - 0.5;
    this.aAcc1 += n1 * 0.013;
    this.aAcc2 += n2 * 0.013;
  }

  update(){
    // péndulo 1
    let torque1 = (-this.g/this.len1) * sin(this.angle1);
    this.aAcc1 += torque1;
    this.aVel1 = constrain((this.aVel1 + this.aAcc1) * this.damping, -this.maxAngVel, this.maxAngVel);
    this.angle1 += this.aVel1; this.aAcc1 = 0;

    // péndulo 2
    let torque2 = (-this.g/this.len2) * sin(this.angle2) + this.aVel1 * 0.012;
    this.aAcc2 += torque2;
    this.aVel2 = constrain((this.aVel2 + this.aAcc2) * this.damping, -this.maxAngVel, this.maxAngVel);
    this.angle2 += this.aVel2; this.aAcc2 = 0;

    // decaimiento del brillo
    this.glow = max(0, this.glow - glowDecay);
  }

  p1(){
    return p5.Vector.add(this.anchor, createVector(this.len1*sin(this.angle1), this.len1*cos(this.angle1)));
  }
  p2(){
    const p1 = this.p1();
    return p5.Vector.add(p1, createVector(this.len2*sin(this.angle2), this.len2*cos(this.angle2)));
  }

  display(){
    const p1 = this.p1();
    const p2 = this.p2();

    // varillas (opacidad global reducida por strokeAlphaMult)
    stroke(this.hue, 55, 95, 90*strokeAlphaMult); strokeWeight(2.4);
    line(this.anchor.x, this.anchor.y, p1.x, p1.y);

    push();
    translate(p1.x, p1.y);
    rotate(-this.angle1);
    strokeWeight(3.4);
    line(-this.len2*0.38, 0, this.len2*0.38, 0);
    pop();

    stroke(this.hue, 65, 95, 90*strokeAlphaMult); strokeWeight(2.0);
    line(p1.x, p1.y, p2.x, p2.y);

    // halos y pesos
    noStroke();
    let s1 = map(this.len1, height*0.08, height*0.16, 12, 22);
    let s2 = map(this.len2, height*0.06, height*0.12, 10, 18);

    if (this.glow > 0){
      fill((this.hue+20)%360, 70, 100, 14*this.glow);
      circle(p1.x, p1.y, s1*(2.2+this.glow));
      circle(p2.x, p2.y, s2*(2.2+this.glow));
    }

    fill((this.hue+18)%360, 70, 97, 95); circle(p1.x, p1.y, s1);
    fill((this.hue+200)%360,70, 97, 95); circle(p2.x, p2.y, s2);
  }
}

class Dust{
  constructor(pos){
    this.pos = pos.copy();
    this.vel = p5.Vector.random2D().mult(random(0.1,0.5));
    this.acc = createVector(0,0);
    this.maxSpeed = 1.6;
    this.size = random(0.8, 1.6);
    this.h = (randomFrom(palette.base)+random(-12,12)+360)%360;
    this.alpha = DUST_ALPHA;
  }
  applyForce(f){ this.acc.add(f); }
  update(){ this.vel.add(this.acc); this.vel.limit(this.maxSpeed); this.pos.add(this.vel); this.acc.mult(0); }
  edgesWrap(){ if (this.pos.x<0) this.pos.x=width; if (this.pos.x>width) this.pos.x=0; if (this.pos.y<0) this.pos.y=height; if (this.pos.y>height) this.pos.y=0; }
  warp(x,y){ this.pos.set(x+random(-2,2), y+random(-2,2)); this.vel.mult(0); }
  display(){ stroke(this.h,60,100,this.alpha); strokeWeight(this.size); point(this.pos.x, this.pos.y); }
}

class Ripple{
  constructor(x,y,hue, speed=1.4){
    this.x=x; this.y=y; this.h=hue; this.r=4; this.speed=speed;
    this.alpha=80; this.dead=false;
  }
  update(){ this.r += this.speed; this.alpha -= 1.8; if (this.alpha<=0) this.dead=true; }
  display(){ noFill(); stroke(this.h, 70, 100, max(0,this.alpha)); strokeWeight(2); circle(this.x, this.y, this.r*2); }
}

// --------- HUD ---------
function drawHUD(levelSm){
  const pad = 12, w=420, h=76;
  noStroke(); fill(0,0,0,60); rect(pad, pad, w, h, 8);
  fill(0,0,100,92); textSize(12);
  text(
`MÓVILES SONOROS — [M] calm  [X] expressive
Click para habilitar audio. Nivel: ${nf(levelSm,1,3)}
[R] resembrar  [P] guardar  [C] conexiones  [T] trail  [B] beats`,
    pad+12, pad+20
  );
}

// ===================== UI de Umbral (abajo-derecha) =====================
let thrUI = { x:0, y:0, w:220, h:56, dragging:false };

function drawThresholdUI(levelSm){
  // posiciona en esquina inferior derecha con margen
  const margin = 14;
  thrUI.x = width - thrUI.w - margin;
  thrUI.y = height - thrUI.h - margin;

  // fondo
  noStroke();
  fill(0,0,0,70);
  rect(thrUI.x, thrUI.y, thrUI.w, thrUI.h, 8);

  // título
  fill(0,0,100,95);
  textSize(12);
  text("Umbral de sonido (beat)", thrUI.x + 10, thrUI.y + 16);

  // barra del umbral (0.00 a 0.30 típico)
  const barX = thrUI.x + 10;
  const barY = thrUI.y + 24;
  const barW = thrUI.w - 20;
  const barH = 10;

  // barra base
  fill(200,60,90,90);
  rect(barX, barY, barW, barH, 4);

  // posición del umbral mapeada
  const maxLevel = 0.30; // rango útil
  const ux = barX + constrain(map(beatThreshold, 0, maxLevel, 0, barW), 0, barW);

  // indicador de umbral
  fill(20,90,100,95);
  rect(barX, barY, constrain(map(beatThreshold,0,maxLevel,0,barW),0,barW), barH, 4);

  // línea del nivel actual para referencia
  const lx = barX + constrain(map(levelSm, 0, maxLevel, 0, barW), 0, barW);
  stroke(50,0,100,80);
  strokeWeight(2);
  line(lx, barY-4, lx, barY+barH+4);

  noStroke();
  fill(0,0,100,92);
  text(`thr: ${nf(beatThreshold,1,3)}  lvl: ${nf(levelSm,1,3)}`, thrUI.x + 10, thrUI.y + 44);
}

function thresholdUIClick(mx,my){
  // ¿clic dentro del panel? → activar drag
  if (mx>=thrUI.x && mx<=thrUI.x+thrUI.w && my>=thrUI.y && my<=thrUI.y+thrUI.h){
    thrUI.dragging = true;
    thresholdUIDrag(mx,my);
    return true;
  }
  return false;
}
function thresholdUIDrag(mx,my){
  if (!thrUI.dragging) return false;
  const barX = thrUI.x + 10;
  const barW = thrUI.w - 20;
  const maxLevel = 0.30;
  // mapear x del mouse a beatThreshold
  const t = constrain((mx - barX)/barW, 0, 1);
  beatThreshold = t * maxLevel;
  return true;
}
function thresholdUIRelease(){ thrUI.dragging = false; }

```

## Captura de pantalla representativa

<img width="945" height="938" alt="image" src="https://github.com/user-attachments/assets/fb53cd3e-816d-4ac9-9f0b-0698a3fe044c" />








