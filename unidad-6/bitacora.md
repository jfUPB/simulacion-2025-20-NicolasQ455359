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













