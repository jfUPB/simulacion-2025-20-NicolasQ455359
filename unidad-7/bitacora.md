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








