# Evidencias de la unidad 7

## Actividad 01
 1. Análisis de ejemplos de Ji Lee

### Ejemplo 1: “EXIT”
Ji Lee transforma la palabra “EXIT” inclinando la letra E hacia la izquierda, como si estuviera saliendo del resto de la palabra.
 Interpretación: El movimiento de la E comunica literalmente la idea de “salir”.
 Elementos usados: inclinación tipográfica y posición espacial para expresar acción.

### Ejemplo 2: “ILLUSION”
En este diseño, las letras LL están ligeramente desalineadas, creando una sensación óptica confusa.
 Interpretación: La distorsión genera la ilusión visual que representa el propio significado.
 Elementos usados: desplazamiento sutil y repetición para generar ambigüedad perceptiva.

### Ejemplo 3: “HORIZON”
Aquí, la palabra se divide horizontalmente: la mitad superior de las letras está arriba de una línea y la inferior, abajo, simulando un horizonte real.
 Interpretación: El diseño refuerza visualmente el concepto de “línea del horizonte”.
 Elementos usados: línea divisoria, alineación y simetría.

### Ejemplo 4: “LIFT”
En este caso, las letras I y F están elevadas sobre el resto, como si “flotaran”.
 Interpretación: Se percibe visualmente la acción de “levantar”.
 Elementos usados: variación vertical y jerarquía en altura.

## 2. Propuestas propias — Palabras como imagen

### Palabra 1: “DESHIELO”

Idea visual: Las letras podrían estar formadas por bloques de hielo que se derriten; las últimas letras comienzan a gotear o deformarse.

Elementos: degradado azul a transparente, gotas cayendo, letras inclinadas o disueltas parcialmente.

Mensaje visual: Representa el proceso de derretirse y cambio climático.

### Palabra 2: “SILENCIO”

Idea visual: Las letras podrían ir desapareciendo gradualmente, como si el sonido se apagara.

Elementos: opacidad decreciente, espaciado entre letras, colores grises o blancos.

Mensaje visual: La ausencia de ruido y la calma total.

### Palabra 3: “VELOCIDAD”

Idea visual: Las letras se deforman hacia la derecha, con líneas de movimiento que simulen rapidez.

Elementos: inclinación, desenfoque de movimiento, degradado horizontal.

Mensaje visual: Sensación de desplazamiento rápido, energía y acción

## Actividad 02
### Experimento 1 — Mundo con gravedad, cuerpos dinámicos y MouseConstraint

<img width="851" height="505" alt="image" src="https://github.com/user-attachments/assets/d1701ed8-fcc4-4783-a440-b1debe747c26" />


Descripción breve: Se crea un motor físico con gravedad, un suelo/paredes estáticos y cuerpos aleatorios (círculos y cajas) que caen y colisionan. Se añade MouseConstraint para agarrar/arrastrar cuerpos con el mouse.
### Codigo
```javascript
const Engine = Matter.Engine;
const World = Matter.World;
const Bodies = Matter.Bodies;
const Composite = Matter.Composite;
const Constraint = Matter.Constraint;
const Mouse = Matter.Mouse;
const MouseConstraint = Matter.MouseConstraint;

let engine, world, mConstraint;
let mode = 1; // 1 = Experimento 1, 2 = Experimento 2
let bob, link, anchor;

// --- Configuración inicial ---
function setup() {
  createCanvas(800, 500);
  engine = Engine.create();
  world = engine.world;
  world.gravity.y = 1;

  setupExperiment1(); // por defecto
}

// --- Cambiar entre experimentos desde botones del index ---
function setExperiment(num) {
  mode = num;
  World.clear(world);
  Composite.clear(world, false);
  if (mode === 1) setupExperiment1();
  else setupExperiment2();
}

// === EXPERIMENTO 1 ===
function setupExperiment1() {
  // Suelo y paredes estáticas
  const ground = Bodies.rectangle(width / 2, height - 20, width, 40, { isStatic: true });
  const wallL = Bodies.rectangle(-20, height / 2, 40, height, { isStatic: true });
  const wallR = Bodies.rectangle(width + 20, height / 2, 40, height, { isStatic: true });
  Composite.add(world, [ground, wallL, wallR]);

  // Cuerpos dinámicos
  for (let i = 0; i < 10; i++) {
    const x = random(100, width - 100);
    const y = random(-200, -20);
    if (random() < 0.5) {
      Composite.add(world, Bodies.circle(x, y, random(15, 35), { restitution: 0.6, friction: 0.1 }));
    } else {
      Composite.add(world, Bodies.rectangle(x, y, random(20, 60), random(20, 60), { restitution: 0.2, friction: 0.3 }));
    }
  }

  // Interacción con mouse
  const mouse = Mouse.create(canvas.elt);
  mConstraint = MouseConstraint.create(engine, {
    mouse,
    constraint: { stiffness: 0.2 }
  });
  Composite.add(world, mConstraint);
}

// === EXPERIMENTO 2 === (se define abajo)
// ...
// --- Dibujo (común) ---
function draw() {
  background(250);
  Engine.update(engine, 1000 / 60);

  noStroke();
  fill(40);
  const bodies = Composite.allBodies(world);
  for (const body of bodies) {
    beginShape();
    for (const v of body.vertices) vertex(v.x, v.y);
    endShape(CLOSE);
  }

  // Visual del constraint en el experimento 2
  if (mode === 2 && link && bob) {
    stroke(0);
    line(anchor.x, anchor.y, bob.position.x, bob.position.y);
    noStroke();
    fill(200, 0, 0);
    circle(anchor.x, anchor.y, 8);
  }

  // Línea del mouse al cuerpo agarrado
  if (mConstraint && mConstraint.body) {
    stroke(0);
    const pos = mConstraint.body.position;
    const m = mConstraint.mouse.position;
    line(pos.x, pos.y, m.x, m.y);
  }
}
```
Experimento 2 — Péndulo con Constraint (punto fijo → cuerpo)

<img width="858" height="506" alt="image" src="https://github.com/user-attachments/assets/403a0627-f84b-4278-86a4-2e17a37d4fef" />


Descripción breve: Se crea un péndulo uniendo un punto fijo del mundo (pointA) con un cuerpo circular (bodyB) mediante Constraint. Se puede empujar con el mouse usando MouseConstraint.

### codigo
```javascript
// === EXPERIMENTO 2 ===
function setupExperiment2() {
  anchor = { x: width / 2, y: 80 }; // punto fijo
  bob = Bodies.circle(width / 2 + 150, 300, 25, { restitution: 0.1, friction: 0.02 });
  Composite.add(world, bob);

  link = Constraint.create({
    pointA: anchor,     // punto fijo en el mundo
    bodyB: bob,         // cuerpo unido
    length: 220,
    stiffness: 0.9,
    damping: 0.02
  });
  Composite.add(world, link);

  const ground = Bodies.rectangle(width / 2, height - 20, width, 40, { isStatic: true });
  Composite.add(world, ground);

  const mouse = Mouse.create(canvas.elt);
  mConstraint = MouseConstraint.create(engine, { mouse, constraint: { stiffness: 0.2 } });
  Composite.add(world, mConstraint);
}
```
### Nota
Nota: En el index.html agregué dos botones para alternar:
```html
<button onclick="setExperiment(1)">Experimento 1</button>
<button onclick="setExperiment(2)">Experimento 2</button>
```

### Conceptos clave 

- Engine (motor): Sistema que avanza la simulación (posiciones, velocidades, colisiones) en cada paso de tiempo. En p5, se actualiza con Engine.update(engine, 1000/60) dentro de draw().

- World (mundo): Contenedor de todo lo físico (cuerpos, constraints, gravedad). Se accede con engine.world. Es el “escenario” de la simulación.

- Bodies (cuerpos): Objetos con masa/forma que pueden moverse o ser estáticos. Comunes: Bodies.rectangle, Bodies.circle, Bodies.polygon. Se configuran con propiedades como isStatic, restitution (rebote), friction, density.

- Constraint (restricción): Conecta dos cuerpos o un cuerpo con un punto fijo, como cuerda, barra o resorte. Parámetros clave: length, stiffness (rigidez), damping (amortiguación).

- MouseConstraint: Permite agarrar/arrastrar cuerpos con el mouse creando una restricción temporal entre el cursor y el cuerpo cercano. Útil para pruebas interactivas.

### Dificultades encontradas 

- “Identifier ‘Engine’ has already been declared”

 Causa: declarar let/const Engine = Matter.Engine más de una vez (por tener dos experimentos en el mismo archivo).

 Solución: declarar las referencias a Matter.js una sola vez al inicio del sketch.js (como en el código de arriba).

- Nada se dibuja / Matter no cargado

 Causa: orden de scripts incorrecto.

 Solución: En el index.html, cargar primero p5.js, luego matter.min.js, y al final tu sketch.js.

- El mouse no arrastra cuerpos

 Causa: el mouse no se creó con el lienzo correcto o no se añadió el constraint al mundo.

 Solución: const mouse = Mouse.create(canvas.elt); y luego Composite.add(world, mConstraint);.

- Los cuerpos atraviesan bordes

 Causa: no hay límites físicos.

 Solución: crear paredes/techo/suelo estáticos (isStatic: true) como en el Experimento 1.

 ### Link
 [Ver experimento en p5.js](https://editor.p5js.org/NicolasQ455359/sketches/UWPta2M3W)


## Actividad 03

### Idea conceptual

Elegí la palabra CONECTAR porque representa unión, relación y movimiento entre partes, y me pareció interesante poder mostrar eso con física y gestos.
La animación busca que las letras no sean algo quieto, sino que parezcan “vivas”, unidas por lazos elásticos que pueden estirarse o romperse.

Cuando hago el gesto de pinza con mi mano, las letras se atraen hacia mí, como si se fortaleciera el vínculo entre ellas. Pero cuando abro la mano, los enlaces se vuelven rompibles, mostrando que la conexión también puede soltarse o perderse.
Además, al mover la palma, genero un viento digital que empuja las letras, dándole una sensación de energía o comunicación entre todas.

La idea es que el gesto humano (conectar o desconectar) controle directamente la palabra, haciendo que su comportamiento físico refleje su significado.

### Aspectos técnicos

Para hacer esto, usé p5.js junto con Matter.js y ml5.js.
Cada letra de “CONECTAR” es un cuerpo físico en forma de rectángulo, y encima se dibuja la letra.
Entre cada par de letras hay dos restricciones elásticas (arriba y abajo), lo que hace que la palabra se sienta unida pero flexible, como una cuerda o una red.

La parte de visión artificial se hizo con ml5 Handpose, que detecta la posición de mi mano en tiempo real.
El programa mide la distancia entre el pulgar y el índice:

- Si están juntos (pinza), las letras son atraídas.

- Si la mano está abierta, los enlaces pasan al modo “rompible”.

- Si muevo la mano, las letras sienten una fuerza de viento.

Las propiedades físicas que más influyen son la rigidez (stiffness), la amortiguación (damping) y el coeficiente de fricción, que ayudan a que el movimiento se vea natural.

```html
<!DOCTYPE html>
<html lang="es">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Actividad 03 — CONECTAR</title>

    <!-- Librerías -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.0/p5.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/matter-js/0.19.0/matter.min.js"></script>
    <script src="https://unpkg.com/ml5@0.12.2/dist/ml5.min.js"></script>

    <style>
      body {
        margin: 0;
        padding: 0;
        overflow: hidden;
        background: #0b0b10;
        font-family: Inter, system-ui, Arial, sans-serif;
      }
      .panel {
        position: absolute;
        top: 10px;
        left: 10px;
        background: rgba(255, 255, 255, 0.9);
        border-radius: 8px;
        padding: 10px 12px;
        font-size: 14px;
      }
    </style>
  </head>

  <body>
    <div class="panel">
      <strong>CONÉCTAR — Control con la mano</strong><br />
      Pinza = atraer letras · Mano abierta = activar o desactivar enlaces rompibles · Mover la mano = viento.<br />
      <b>Teclas:</b> G = gravedad | R = reset | 1/2/3 = tema visual
    </div>
    <script src="sketch.js"></script>
  </body>
</html>
```
```javascript
// ===== Matter refs =====
const Engine = Matter.Engine;
const World = Matter.World;
const Bodies = Matter.Bodies;
const Composite = Matter.Composite;
const Constraint = Matter.Constraint;
const Mouse = Matter.Mouse;
const MouseConstraint = Matter.MouseConstraint;

let engine, world, mConstraint;
let letterBodies = [];
let links = [];
let walls = [];
let gravityOn = true;
let breakable = true;    // ahora se controla con MANO ABIERTA (gesto), no con tecla B
let particles = [];
let theme = 2;           // look oscuro/cian por defecto

// Palabra y parámetros
const WORD = "CONECTAR";
const LETTER_W = 58;
const LETTER_H = 78;
const LETTER_SPACING = 18;
const BASE_Y = 170;
const DAMPING_LINK = 0.08;
const BREAK_RATIO = 1.65;

// --------- Visión (ml5 Handpose) ---------
let video, handpose, predictions = [];
let handReady = false;

// Señales/estado de gestos
let palmX = null, palmY = null, prevPalmX = null, prevPalmY = null;
let pinchDist = null;
const PINCH_THRESHOLD = 35; // px — por debajo = pinza
let wasOpen = false;        // para hacer "toggle por flanco" cuando la mano se abre

function setup() {
  createCanvas(980, 560);

  // Cámara
  video = createCapture({ video: true, audio: false });
  video.size(640, 480);
  video.hide();

  // Handpose
  handpose = ml5.handpose(video, () => { handReady = true; });
  handpose.on("predict", (results) => { predictions = results; });

  // Matter
  engine = Engine.create();
  world = engine.world;
  world.gravity.y = 1;

  // Bordes
  const thick = 50;
  const ground = Bodies.rectangle(width / 2, height + thick/2 - 2, width, thick, { isStatic: true });
  const roof   = Bodies.rectangle(width / 2, -thick/2 + 2, width, thick, { isStatic: true });
  const wallL  = Bodies.rectangle(-thick/2 + 2, height / 2, thick, height, { isStatic: true });
  const wallR  = Bodies.rectangle(width + thick/2 - 2, height / 2, thick, height, { isStatic: true });
  walls = [ground, roof, wallL, wallR];
  Composite.add(world, walls);

  // Palabra
  createWordBodies();
  connectLettersLadder();

  // Mouse (opcional, por si quieres arrastrar)
  const mouse = Mouse.create(canvas.elt);
  mConstraint = MouseConstraint.create(engine, { mouse, constraint: { stiffness: 0.22 } });
  Composite.add(world, mConstraint);

  textAlign(CENTER, CENTER);
  rectMode(CENTER);
}

function createWordBodies() {
  letterBodies = [];
  const totalWidth = WORD.length * LETTER_W + (WORD.length - 1) * LETTER_SPACING;
  const startX = (width - totalWidth) / 2 + LETTER_W / 2;

  for (let i = 0; i < WORD.length; i++) {
    const x = startX + i * (LETTER_W + LETTER_SPACING);
    const y = BASE_Y;

    const body = Bodies.rectangle(x, y, LETTER_W, LETTER_H, {
      restitution: 0.06,
      friction: 0.22,
      frictionAir: 0.015,
      density: 0.0028
    });
    body._init = { x, y, angle: 0 };
    body.label = `LETTER_${WORD[i]}`;
    letterBodies.push(body);
    Composite.add(world, body);
  }
}

// Doble constraint por par (arriba/abajo) para movimiento orgánico
function connectLettersLadder() {
  links = [];
  for (let i = 0; i < letterBodies.length - 1; i++) {
    const A = letterBodies[i];
    const B = letterBodies[i + 1];
    // top
    const linkTop = Constraint.create({
      bodyA: A, bodyB: B,
      pointA: { x: LETTER_W/2 - 6, y: -LETTER_H/2 + 10 },
      pointB: { x: -LETTER_W/2 + 6, y: -LETTER_H/2 + 10 },
      length: LETTER_SPACING + 8, stiffness: 0.12, damping: DAMPING_LINK
    });
    linkTop._restLen = linkTop.length;
    // bottom
    const linkBot = Constraint.create({
      bodyA: A, bodyB: B,
      pointA: { x: LETTER_W/2 - 6, y:  LETTER_H/2 - 10 },
      pointB: { x: -LETTER_W/2 + 6, y:  LETTER_H/2 - 10 },
      length: LETTER_SPACING + 8, stiffness: 0.12, damping: DAMPING_LINK
    });
    linkBot._restLen = linkBot.length;

    links.push(linkTop, linkBot);
    Composite.add(world, [linkTop, linkBot]);
  }
}

function draw() {
  drawBackgroundTheme();
  Engine.update(engine, 1000 / 60);

  // Procesar mano y aplicar controles físicos
  updateHandControls();

  // Dibujar links con color por tensión + rotura si procede
  for (let i = links.length - 1; i >= 0; i--) {
    const L = links[i];
    if (!L.bodyA || !L.bodyB) continue;
    const { ax, ay, bx, by, lenNow } = endpointsAndLength(L);
    const color = tensionColor(lenNow / (L._restLen || L.length));
    stroke(color);
    strokeWeight(2);
    line(ax, ay, bx, by);

    if (breakable && lenNow > (L._restLen || L.length) * BREAK_RATIO) {
      spawnBreakSparks((ax + bx) / 2, (ay + by) / 2);
      Composite.remove(world, L);
      links.splice(i, 1);
    }
  }

  // Dibujar letras
  for (let i = 0; i < letterBodies.length; i++) drawLetterBlock(letterBodies[i], WORD[i]);

  // Partículas
  updateAndDrawParticles();

  // Indicadores de mano
  drawHandOverlay();

  drawHUD();
}

/* ---------- Handpose: control simplificado ---------- */
function updateHandControls() {
  if (!handReady || predictions.length === 0) {
    pinchDist = null;
    wasOpen = false;
    return;
  }
  const hand = predictions[0];
  const thumb = hand.annotations.thumb[3];      // pulgar tip
  const indexF = hand.annotations.indexFinger[3];// índice tip
  const wrist = hand.landmarks[0];

  // Map de coords (video 640x480 -> canvas) y espejo
  const vx = (1 - (wrist[0] / video.width)) * width;
  const vy = (wrist[1] / video.height) * height;

  // Suavizado de palma
  if (palmX == null) { palmX = vx; palmY = vy; }
  palmX = lerp(palmX, vx, 0.4);
  palmY = lerp(palmY, vy, 0.4);

  // Distancia de pinza
  const tx = (1 - (thumb[0] / video.width)) * width;
  const ty = (thumb[1] / video.height) * height;
  const ix = (1 - (indexF[0] / video.width)) * width;
  const iy = (indexF[1] / video.height) * height;
  pinchDist = dist(tx, ty, ix, iy);

  // Viento por velocidad de palma (sustituye tecla W)
  if (prevPalmX != null && prevPalmY != null) {
    const vxPalm = palmX - prevPalmX;
    const vyPalm = palmY - prevPalmY;
    const scale = 0.0008;
    for (const b of letterBodies) {
      Matter.Body.applyForce(b, b.position, { x: vxPalm * scale, y: vyPalm * scale });
    }
  }
  prevPalmX = palmX;
  prevPalmY = palmY;

  // GESTOS:
  // Pinza => atraer fuerte a la palma
  const isPinching = pinchDist < PINCH_THRESHOLD;
  if (isPinching) {
    attractLettersTo(palmX, palmY, 0.035);
    wasOpen = false; // al cerrar, preparamos el flanco de apertura
  } else {
    // Mano abierta => "función B": toggle de breakable UNA sola vez cuando se abre
    if (!wasOpen) {
      breakable = !breakable; // T O G G L E
      wasOpen = true;
    }
    // Con la mano abierta también aplicamos leve repulsión para dar control
    repelLettersFrom(palmX, palmY, 0.008);
  }
}

function attractLettersTo(x, y, k = 0.03) {
  for (const b of letterBodies) {
    const dir = createVector(x - b.position.x, y - b.position.y);
    const d = max(40, dir.mag());
    dir.normalize().mult(k * (1 / (d / 120)));
    Matter.Body.applyForce(b, b.position, { x: dir.x, y: dir.y });
  }
}

function repelLettersFrom(x, y, k = 0.01) {
  for (const b of letterBodies) {
    const dir = createVector(b.position.x - x, b.position.y - y);
    const d = max(50, dir.mag());
    dir.normalize().mult(k * (1 / (d / 120)));
    Matter.Body.applyForce(b, b.position, { x: dir.x, y: dir.y });
  }
}

/* ---------- Dibujo & estética ---------- */
function drawBackgroundTheme() {
  if (theme === 1) {
    for (let y = 0; y < height; y++) {
      const t = y / height;
      stroke( lerpColor(color("#f2f4ff"), color("#e6faff"), t) );
      line(0, y, width, y);
    }
  } else if (theme === 2) {
    background("#0b0b10");
    noFill(); stroke(255, 255, 255, 10);
    for (let x = 0; x < width; x += 28) line(x, 0, x, height);
    for (let y = 0; y < height; y += 28) line(0, y, width, y);
  } else {
    for (let y = 0; y < height; y++) {
      const t = y / height;
      stroke( lerpColor(color("#f8e7ff"), color("#ffe9f2"), t) );
      line(0, y, width, y);
    }
  }
}

function drawLetterBlock(b, ch) {
  // sombra
  push();
  translate(b.position.x + 6, b.position.y + 8);
  rotate(b.angle);
  noStroke(); fill(0, 30);
  rect(0, 0, LETTER_W, LETTER_H, 14);
  pop();

  // bloque
  push();
  translate(b.position.x, b.position.y);
  rotate(b.angle);
  noStroke();
  const fillCol = (theme === 2) ? color("#9AE6FF") : (theme === 3 ? color("#b86cff") : color("#1f5eff"));
  fill( lerpColor(fillCol, color(255), 0.15) );
  rect(0, 0, LETTER_W, LETTER_H, 14);
  // borde
  noFill(); stroke(0, 60); strokeWeight(1.2);
  rect(0, 0, LETTER_W, LETTER_H, 14);
  // letra
  noStroke(); fill(theme === 2 ? 15 : 30);
  textSize(38); text(ch, 0, 3);
  pop();
}

function drawHandOverlay() {
  if (!handReady) return;
  // anillo en la palma: verde si pinza (atracción), naranja si abierta (toggle B)
  if (palmX != null && palmY != null) {
    stroke(pinchDist != null && pinchDist < PINCH_THRESHOLD ? "#33cc66" : "#ffaa00");
    strokeWeight(2);
    noFill();
    circle(palmX, palmY, 22);
  }
}

function drawHUD() {
  const modeTxt = !handReady ? "cargando..." :
                  (predictions.length ? (pinchDist < PINCH_THRESHOLD ? "PINZA: atraer" : "MANO ABIERTA: toggle romper") : "sin mano");
  const txt = `Handpose: ${modeTxt} · romper: ${breakable ? "ON" : "OFF"} · G: gravedad · R: reset · 1/2/3: temas`;
  const w = textWidth(txt) + 18;
  noStroke(); fill(255, 230); rect(width/2, height - 22, w, 26, 8);
  fill(30); textSize(12); text(txt, width/2, height - 22);
}

/* ---------- Utilidades físicas/partículas ---------- */
function endpointsAndLength(L) {
  const a = L.bodyA, b = L.bodyB;
  const ax = a.position.x + Math.cos(a.angle) * L.pointA.x - Math.sin(a.angle) * L.pointA.y;
  const ay = a.position.y + Math.sin(a.angle) * L.pointA.x + Math.cos(a.angle) * L.pointA.y;
  const bx = b.position.x + Math.cos(b.angle) * L.pointB.x - Math.sin(b.angle) * L.pointB.y;
  const by = b.position.y + Math.sin(b.angle) * L.pointB.x + Math.cos(b.angle) * L.pointB.y;
  const lenNow = dist(ax, ay, bx, by);
  return { ax, ay, bx, by, lenNow };
}

function tensionColor(ratio) {
  const t = constrain(map(ratio, 1.0, 1.6, 0, 1), 0, 1);
  return lerpColor(color("#33cc66"), color("#ff4d4f"), t);
}

function spawnBreakSparks(x, y) {
  for (let i = 0; i < 16; i++) {
    particles.push({ x, y, vx: random(-2, 2), vy: random(-2, -0.2), life: random(16, 30), col: color("#ff9a3c") });
  }
}

function updateAndDrawParticles() {
  for (let i = particles.length - 1; i >= 0; i--) {
    const p = particles[i];
    p.x += p.vx; p.y += p.vy; p.vy += 0.08; p.life--;
    noStroke(); fill(red(p.col), green(p.col), blue(p.col), map(p.life, 0, 30, 0, 255));
    circle(p.x, p.y, map(p.life, 0, 30, 0, 5) + 2);
    if (p.life <= 0) particles.splice(i, 1);
  }
}

/* ---------- Teclas mínimas (simplificadas) ---------- */
function keyPressed() {
  if (key === 'G' || key === 'g') { gravityOn = !gravityOn; world.gravity.y = gravityOn ? 1 : 0; }
  if (key === 'R' || key === 'r') resetWord();
  if (key === '1') theme = 1;
  if (key === '2') theme = 2;
  if (key === '3') theme = 3;
}

function resetWord() {
  // reset constraints
  for (const L of links) Composite.remove(world, L);
  links = [];
  // reset letters
  for (const b of letterBodies) {
    const s = b._init;
    Matter.Body.setPosition(b, { x: s.x, y: s.y });
    Matter.Body.setAngle(b, s.angle);
    Matter.Body.setVelocity(b, { x: 0, y: 0 });
    Matter.Body.setAngularVelocity(b, 0);
  }
  connectLettersLadder();
  particles = [];
}
```

<img width="976" height="567" alt="Captura de pantalla 2025-10-16 103307" src="https://github.com/user-attachments/assets/70c05e59-4451-438a-9ffe-8d5bc7c12efa" />


<img width="960" height="560" alt="Captura de pantalla 2025-10-16 103322" src="https://github.com/user-attachments/assets/07953b1d-b6d8-4660-85c3-f717b019e479" />


### Nota propuesta: 5.0
Actividad 01
En esta primera parte analicé el trabajo de Ji Lee y comprendí cómo la forma visual de una palabra puede comunicar su significado sin necesidad de imágenes externas.
Describí varios ejemplos con un análisis personal, y luego propuse ideas propias con sentido semántico y coherencia entre forma y significado.
Cumplí con todos los puntos solicitados (observación, análisis y creación de bocetos), y lo hice de manera reflexiva, no solo descriptiva.

Actividad 02
En esta etapa aprendí a usar Matter.js desde cero. Replicé y documenté correctamente dos experimentos básicos:

Cuerpos cayendo y colisionando.

Un sistema con restricciones y control de mouse.

Expliqué claramente qué es Engine, World, Bodies, Constraint y MouseConstraint, con mis propias palabras, y presenté el código, capturas y reflexión sobre las dificultades (principalmente los errores de variables duplicadas).
Además, solucioné los problemas del código y logré integrar p5.js con Matter.js de forma limpia.

Actividad 03
La palabra elegida CONECTAR tiene un concepto claro y coherente con la interacción: las letras unidas por enlaces representan conexiones, y los gestos de la mano (pinza y mano abierta) permiten literalmente conectar o romper.
Incorporé visión artificial con ml5.js (Handpose) y eliminé controles innecesarios, haciendo que el proyecto fuera totalmente gestual.
Además, mantuve una estética limpia, tres temas visuales, movimiento fluido y un sistema de partículas cuando los enlaces se rompen.

Integré tres tecnologías (p5.js, Matter.js y ml5.js).

Logré una relación directa entre concepto y comportamiento.

Cumplí con la parte técnica, estética y reflexiva.

Simplifiqué la interacción para que fuera más intuitiva y significativa.















