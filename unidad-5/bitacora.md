# Evidencias de la unidad 5

## Actividad 02
### Ejemplo 4.2 – An Array of Particles

Gestión de creación y desaparición:
En este ejemplo se usa un array de partículas. En cada frame se crean nuevas partículas que se agregan al arreglo y las que “mueren” (cuando su vida llega a cero) se eliminan. La memoria se gestiona removiendo las partículas muertas con splice(), lo que evita acumulación en el arreglo.

Modificación:
Concepto aplicado: Transformaciones (translate, rotate).

Cómo lo apliqué: Trasladé el origen al centro de la pantalla y añadí una rotación constante para que las partículas se generaran girando en torno al punto de emisión.

Por qué: Quise repasar cómo los sistemas de coordenadas modifican la percepción espacial, creando un efecto dinámico y diferente al flujo lineal habitual de partículas.

[Sketch en p5.js – NicolasQ455359 (ID: ABsxbw5fk)](https://editor.p5js.org/NicolasQ455359/sketches/ABsxbw5fk)

### Codigo: 
```javascript
let particles = [];
let emitter;

function setup() {
  createCanvas(640, 360);
  emitter = createVector(0, 0); 

function draw() {
  background(0);
  
  // Centro y rotación constante del sistema
  push();
  translate(width/2, height/2);
  rotate(frameCount * 0.01);

  for (let i = 0; i < 3; i++) {
    particles.push(new Particle(emitter.x, emitter.y));
  }

  for (let i = particles.length - 1; i >= 0; i--) {
    let p = particles[i];
    p.update();
    p.display();
    if (p.isDead()) {
      particles.splice(i, 1); // gestión de memoria: liberar referencia en el array
    }
  }

  pop();

  noStroke();
  fill(255);
  text(`Partículas activas: ${particles.length}`, 10, 20);
}

class Particle {
  constructor(x, y) {
    this.pos = createVector(x, y);
    // Darle una velocidad que salga radialmente del centro rotado
    const angle = random(TWO_PI);
    const speed = random(1, 3);
    this.vel = p5.Vector.fromAngle(angle).mult(speed);
    this.acc = createVector(0, 0);
    this.lifespan = 255;
    this.size = random(4, 8);
  }

  applyForce(f) {
    this.acc.add(f);
  }

  update() {
    this.applyForce(createVector(0, 0.03)); 
    this.vel.add(this.acc);
    this.pos.add(this.vel);
    this.acc.mult(0);
    this.lifespan -= 2.5;
  }

  display() {
    noStroke();
    fill(200, 150, 255, this.lifespan);
    ellipse(this.pos.x, this.pos.y, this.size);
  }

  isDead() {
    return this.lifespan <= 0;
  }
}
```
<img width="856" height="566" alt="image" src="https://github.com/user-attachments/assets/2452d262-44d4-4154-a41f-c7ea7e7fd467" />

### Ejemplo 4.4 - System of Systems

Gestión de creación y desaparición:
Aquí no solo hay partículas, sino múltiples sistemas de partículas. Cada sistema crea sus partículas de manera independiente y las gestiona internamente (vida útil y eliminación).

Modificación:
Concepto aplicado: Interactividad con el usuario (mousePressed).

Cómo lo apliqué: Programé que cada clic en el lienzo genere un nuevo sistema de partículas en la posición del mouse.

Por qué: Busqué repasar la relación entre entrada del usuario y el comportamiento del sistema, mostrando cómo la interacción directa influye en la creación de elementos dentro de la simulación.

[Sketch en p5.js – NicolasQ455359 (ID: f79cPqS4Wi)](https://editor.p5js.org/NicolasQ455359/sketches/f79cPqS4Wi)

### Codigo: 
```javascript
let systems = [];

function setup() {
  createCanvas(640, 360);
  // Sistema inicial
  systems.push(new ParticleSystem(createVector(width/2, height/2)));
}

function draw() {
  background(10);

  // Actualizar y dibujar todos los sistemas
  for (let s = systems.length - 1; s >= 0; s--) {
    systems[s].run();
    // Si ya no tiene partículas y dejó de emitir, lo removemos
    if (systems[s].isEmpty()) {
      systems.splice(s, 1);
    }
  }

  fill(255);
  text(`Sistemas: ${systems.length}`, 10, 20);
}

function mousePressed() {
  // Cada clic agrega un nuevo sistema en la posición del mouse
  systems.push(new ParticleSystem(createVector(mouseX, mouseY)));
}

class ParticleSystem {
  constructor(origin) {
    this.origin = origin.copy();
    this.particles = [];
    this.emitting = true;
    this.emitFrames = 180; // emitir por 3 s aprox a 60 fps
  }

  addParticle() {
    this.particles.push(new Particle(this.origin.x, this.origin.y));
  }

  run() {
    if (this.emitting) {
      for (let i = 0; i < 2; i++) this.addParticle();
      this.emitFrames--;
      if (this.emitFrames <= 0) this.emitting = false;
    }

    for (let i = this.particles.length - 1; i >= 0; i--) {
      let p = this.particles[i];
      p.update();
      p.display();
      if (p.isDead()) this.particles.splice(i, 1); // liberar
    }
  }

  isEmpty() {
    return !this.emitting && this.particles.length === 0;
  }
}

class Particle {
  constructor(x, y) {
    this.pos = createVector(x, y);
    this.vel = p5.Vector.random2D().mult(random(1, 3));
    this.acc = createVector(0, 0.05);
    this.lifespan = 200;
    this.size = random(3, 7);
  }
  update() {
    this.vel.add(this.acc);
    this.pos.add(this.vel);
    this.lifespan -= 3;
  }
  display() {
    noStroke();
    fill(100, 200, 255, this.lifespan);
    ellipse(this.pos.x, this.pos.y, this.size);
  }
  isDead() {
    return this.lifespan <= 0;
  }
}

```

<img width="638" height="360" alt="image" src="https://github.com/user-attachments/assets/b49b87e1-e172-4a0c-99ea-be3c12a311b1" />


### Ejemplo Ejemplo 4.5 – Particle System with Inheritance and Polymorphism

Gestión de creación y desaparición:
Se usan clases hijas (por ejemplo, Confetti) que heredan de Particle. Cada partícula tiene un tiempo de vida, y cuando muere, se elimina del arreglo.

Concepto aplicado: Aleatoriedad (random).

Cómo lo apliqué: Modifiqué las subclases para que las partículas hereden propiedades con variación en tamaño y color mediante random().

Por qué: La aleatoriedad es clave en las simulaciones generativas, porque da riqueza visual y diversidad de comportamientos dentro de un mismo sistema.

[Sketch en p5.js – NicolasQ455359 (ID: sY6e5RwcK)](https://editor.p5js.org/NicolasQ455359/sketches/sY6e5RwcK)

```javascript
let ps;

function setup() {
  createCanvas(640, 360);
  ps = new ParticleSystem(createVector(width/2, height/2));
}

function draw() {
  background(12);
  ps.addParticle();        // emitir una por frame
  ps.run();

  fill(255);
  text('Pulsa espacio para alternar tipo', 10, 20);
}

function keyPressed() {
  if (key === ' ') ps.toggleType();
}

class ParticleSystem {
  constructor(origin) {
    this.origin = origin.copy();
    this.particles = [];
    this.useConfetti = false;
  }

  toggleType() {
    this.useConfetti = !this.useConfetti;
  }

  addParticle() {
    if (this.useConfetti) {
      this.particles.push(new Confetti(this.origin.x, this.origin.y));
    } else {
      this.particles.push(new Particle(this.origin.x, this.origin.y));
    }
  }

  run() {
    for (let i = this.particles.length - 1; i >= 0; i--) {
      let p = this.particles[i];
      p.update();
      p.display();
      if (p.isDead()) this.particles.splice(i, 1);
    }
  }
}

class Particle {
  constructor(x, y) {
    this.pos = createVector(x, y);
    this.vel = p5.Vector.random2D().mult(random(0.5, 2.2));
    this.acc = createVector(0, 0.03);
    this.lifespan = 255;
    this.size = random(4, 8);              // aleatoriedad en tamaño
    this.col = color(random(180,255),      // y color
                     random(100,220),
                     random(180,255));
  }

  update() {
    this.vel.add(this.acc);
    this.pos.add(this.vel);
    this.lifespan -= 2;
  }

  display() {
    noStroke();
    fill(red(this.col), green(this.col), blue(this.col), this.lifespan);
    ellipse(this.pos.x, this.pos.y, this.size);
  }

  isDead() { return this.lifespan <= 0; }
}

// Subclase: cambia la forma y rota
class Confetti extends Particle {
  constructor(x, y) {
    super(x, y);
    this.angle = random(TWO_PI);
    this.spin = random(-0.1, 0.1);
    this.w = random(4, 10);
    this.h = random(2, 6);
  }

  update() {
    super.update();
    this.angle += this.spin;
  }

  display() {
    push();
    translate(this.pos.x, this.pos.y);
    rotate(this.angle);
    noStroke();
    fill(red(this.col), green(this.col), blue(this.col), this.lifespan);
    rectMode(CENTER);
    rect(0, 0, this.w, this.h);
    pop();
  }
}
```

<img width="637" height="354" alt="Captura de pantalla 2025-09-18 084821" src="https://github.com/user-attachments/assets/063afef6-b015-44d5-b9bf-dba97979ff1b" />

<img width="636" height="355" alt="image" src="https://github.com/user-attachments/assets/59784027-d2b3-4b83-b671-f84c248e0c44" />


### Ejemplo 4.6 – Particle System with Forces

Gestión de creación y desaparición:
Cada partícula está sujeta a fuerzas (por ejemplo, gravedad). La desaparición se gestiona igual: una vez que la vida de la partícula llega a cero, se elimina del arreglo.

Concepto aplicado: Vectores y fuerzas.

Cómo lo apliqué: Además de la gravedad, añadí una fuerza de viento horizontal dependiente de la posición del mouse (map(mouseX, ...)) y la apliqué a todas las partículas.

Por qué: Quise repasar cómo se construye la dinámica de un sistema físico sumando fuerzas vectoriales, logrando que el comportamiento responda tanto a la física como a la interacción del usuario.

[Sketch en p5.js – NicolasQ455359 (ID: 1DLZplVcW)](https://editor.p5js.org/NicolasQ455359/sketches/1DLZplVcW)

```javascript
// 4.6 Forces — gravedad + viento controlado por mouseX
let ps;
let gravity;

function setup() {
  createCanvas(640, 360);
  ps = new ParticleSystem(createVector(width/2, 40));
  gravity = createVector(0, 0.08);
}

function draw() {
  background(15);

  // Viento según mouseX (-0.2 a 0.2)
  let wind = createVector(map(mouseX, 0, width, -0.2, 0.2), 0);

  // Emitir y aplicar fuerzas a cada partícula
  ps.addParticle();
  ps.applyForceAll(gravity);
  ps.applyForceAll(wind);
  ps.run();

  fill(255);
  text(`Viento x: ${wind.x.toFixed(2)}`, 10, 20);
}

class ParticleSystem {
  constructor(origin) {
    this.origin = origin.copy();
    this.particles = [];
  }

  addParticle() {
    this.particles.push(new Particle(this.origin.x, this.origin.y));
  }

  applyForceAll(force) {
    for (let p of this.particles) p.applyForce(force);
  }

  run() {
    for (let i = this.particles.length - 1; i >= 0; i--) {
      let p = this.particles[i];
      p.update();
      p.display();
      if (p.isDead()) this.particles.splice(i, 1);
    }
  }
}

class Particle {
  constructor(x, y) {
    this.pos = createVector(x, y);
    this.vel = createVector(random(-1, 1), random(-2, 0));
    this.acc = createVector(0, 0);
    this.lifespan = 220;
    this.size = random(3, 6);
  }

  applyForce(f) { this.acc.add(f); }

  update() {
    this.vel.add(this.acc);
    this.pos.add(this.vel);
    this.acc.mult(0);
    this.lifespan -= 2.2;
  }

  display() {
    noStroke();
    fill(255, 220, 120, this.lifespan);
    ellipse(this.pos.x, this.pos.y, this.size);
  }

  isDead() { return this.lifespan <= 0; }
}
```
<img width="639" height="357" alt="image" src="https://github.com/user-attachments/assets/3532ebcf-f953-430a-a07f-5f00a9b8c9e8" />

### Ejemplo 4.7 – Particle System with a Repeller

Gestión de creación y desaparición:
Además de las partículas con vida limitada, aquí se agrega un “repeller” (repulsor) que modifica las trayectorias. Las partículas siguen muriendo por vida útil y se eliminan.

Concepto aplicado: Perlin Noise.

Cómo lo apliqué: Utilicé noise() para modificar la posición inicial del emisor de partículas, generando un movimiento oscilatorio y natural en el punto de spawn.

Por qué: El ruido Perlin permite evitar trayectorias artificiales o repetitivas, aportando variación orgánica y más realismo en sistemas generativos.

[Sketch en p5.js – NicolasQ455359 (ID: IEpR4G3XE)](https://editor.p5js.org/NicolasQ455359/sketches/IEpR4G3XE)

```javascript
let ps;
let repeller;
let t = 0; // tiempo para el ruido

function setup() {
  createCanvas(640, 360);
  ps = new ParticleSystem(createVector(width/2, height/2));
  repeller = new Repeller(width/2, height/2);
}

function draw() {
  background(8);

  // Emisión con offset por ruido Perlin
  let nx = map(noise(t), 0, 1, -80, 80);
  let ny = map(noise(t + 1000), 0, 1, -80, 80);
  ps.origin.set(width/2 + nx, height/2 + ny);
  ps.addParticle();
  t += 0.01;

  // Aplicar fuerza de repulsión a cada partícula
  for (let p of ps.particles) {
    let f = repeller.repel(p);
    p.applyForce(f);
  }

  ps.run();
  repeller.display();
}

class ParticleSystem {
  constructor(origin) {
    this.origin = origin.copy();
    this.particles = [];
  }

  addParticle() {
    this.particles.push(new Particle(this.origin.x, this.origin.y));
  }

  run() {
    for (let i = this.particles.length - 1; i >= 0; i--) {
      let p = this.particles[i];
      p.update();
      p.display();
      if (p.isDead()) this.particles.splice(i, 1);
    }
  }
}

class Particle {
  constructor(x, y) {
    this.pos = createVector(x, y);
    this.vel = p5.Vector.random2D().mult(random(0.5, 1.5));
    this.acc = createVector(0, 0);
    this.lifespan = 240;
    this.size = 5;
  }

  applyForce(f) { this.acc.add(f); }

  update() {
    this.vel.add(this.acc);
    this.pos.add(this.vel);
    this.acc.mult(0);
    this.lifespan -= 2;
  }

  display() {
    noStroke();
    fill(120, 200, 255, this.lifespan);
    circle(this.pos.x, this.pos.y, this.size);
  }

  isDead() { return this.lifespan <= 0; }
}

class Repeller {
  constructor(x, y) {
    this.pos = createVector(x, y);
    this.strength = 200; // constante de repulsión
    this.r = 20;
  }

  repel(particle) {
    let dir = p5.Vector.sub(this.pos, particle.pos);
    let d = constrain(dir.mag(), 5, 100);
    dir.normalize();
    // Fuerza de repulsión (invertida)
    let force = dir.mult(-this.strength / (d * d));
    return force;
  }

  display() {
    noFill();
    stroke(255, 80);
    circle(this.pos.x, this.pos.y, this.r * 2);
  }
}
```

<img width="637" height="358" alt="image" src="https://github.com/user-attachments/assets/9d5b6c80-5f5e-46ba-9972-74d6cb336b6a" />


## Actividad 03: 
### Concepto de mi obra:
Quise crear una experiencia visual que muestre cómo todo en nuestro entorno está conectado por fuerzas invisibles. Las partículas representan energías o recuerdos que siempre están en movimiento, guiadas por corrientes que no vemos (el flujo generado por ruido) y también por puntos de atracción o rechazo (los portales).

### Lo que quiero comunicar:
Mi intención es que, al interactuar con la obra, se sienta que cada acción que hacemos deja una huella en el espacio. Al mover el mouse o arrastrar un portal, se altera todo el sistema, como pasa en la vida real cuando pequeñas decisiones generan cambios más grandes. Quiero que quien vea e interactúe con la pieza sienta que está “pintando con energía” en tiempo real, y que perciba esa idea de que el mundo es dinámico, cambiante y sensible a nuestra presencia.

### Codigo
[Sketch en p5.js – NicolasQ455359 (ID: 91dyJOoU1)](https://editor.p5js.org/NicolasQ455359/sketches/91dyJOoU1)

```javascript
let systems = [];
let mainSystem;

let particleType = 0;   // 0 Glow, 1 Shard, 2 Ribbon
let baseEmission = 3;

let t = 0;              // tiempo para ruido
let noiseScale = 0.0035;
let curlStrength = 0.35;
let attractStrength = 0.35;
let repelStrength = 1.25;

let trail;              // capa de estela
let portals = [];       // portales arrastrables
let draggingPortal = null;

function setup() {
  createCanvas(900, 560);
  colorMode(HSB, 360, 100, 100, 255);
  noStroke();

  trail = createGraphics(width, height);
  trail.colorMode(HSB, 360, 100, 100, 255);
  trail.noStroke();
  trail.background(0, 0, 0, 255);

  mainSystem = new ParticleSystem(createVector(width * 0.5, height * 0.5));
  systems.push(mainSystem);

  // Portales iniciales
  portals.push(new Portal(width * 0.28, height * 0.45, true));   // atractor
  portals.push(new Portal(width * 0.72, height * 0.55, false));  // repulsor

  textFont('monospace');
}

function draw() {
  // Fading suave en la capa de estelas
  trail.push();
  trail.fill(0, 0, 0, 20);
  trail.rect(0, 0, width, height);
  trail.pop();

  background(0);

  // Emisión
  for (let i = 0; i < baseEmission; i++) {
    mainSystem.addParticle(particleType);
  }

  // Campo de flujo + fuerzas portales + mouse
  for (let s of systems) {
    for (let p of s.particles) {
      // Curl noise
      let flow = curlNoise(p.pos.x * noiseScale, p.pos.y * noiseScale, t).mult(curlStrength);
      p.applyForce(flow);

      // Gravedad mínima
      p.applyForce(createVector(0, 0.01));

      // Fuerzas de portales
      for (let portal of portals) {
        let f = portal.forceOn(p);
        p.applyForce(f);
      }

      // Mouse: click = repulsor, sin click = atractor suave
      let m = createVector(mouseX, mouseY);
      let dir = p5.Vector.sub(m, p.pos);
      let d = constrain(dir.mag(), 1, 240);
      dir.normalize();
      if (!draggingPortal) {
        if (mouseIsPressed) {
          p.applyForce(p5.Vector.mult(dir, -repelStrength / (d * 0.03)));
        } else {
          p.applyForce(p5.Vector.mult(dir, attractStrength / (d * 0.07)));
        }
      }
    }
  }

  // Actualizar y dibujar partículas
  for (let s = systems.length - 1; s >= 0; s--) {
    systems[s].run(trail);
    if (systems[s].isEmpty()) systems.splice(s, 1);
  }

  // Composición visual
  image(trail, 0, 0);
  blendMode(ADD);
  for (let s of systems) s.drawFront();
  blendMode(BLEND);

  // Dibujar portales
  for (let portal of portals) portal.display();

  // HUD
  drawHUD();

  t += 0.0045;
}

// ---------------- Interacción ----------------
function mousePressed() {
  // ¿Arrastramos portal?
  for (let portal of portals) {
    if (portal.over(mouseX, mouseY)) {
      draggingPortal = portal;
      return;
    }
  }
  // Si no: ráfaga
  for (let i = 0; i < 40; i++) mainSystem.addParticle(particleType, true);
}

function mouseDragged() {
  if (draggingPortal) draggingPortal.set(mouseX, mouseY);
}

function mouseReleased() {
  draggingPortal = null;
}

function keyPressed() {
  if (key === 'C' || key === 'c') {
    particleType = (particleType + 1) % 3;
  }
  if (key === 'P' || key === 'p') {
    if (portals.length > 0) portals[0].isAttractor = !portals[0].isAttractor;
  }
  if (key === 'R' || key === 'r') {
    systems = [];
    mainSystem = new ParticleSystem(createVector(width * 0.5, height * 0.5));
    systems.push(mainSystem);
    trail.clear();
    trail.background(0);
  }
  if (key === '[') baseEmission = max(0, baseEmission - 1);
  if (key === ']') baseEmission += 1;
}

// ---------------- HUD ----------------
function drawHUD() {
  fill(255);
  textSize(12);
  const mode = ['Glow', 'Shard', 'Ribbon'][particleType];
  text(
`Partículas: ${mainSystem.particles.length}
Tipo: ${mode}  |  Emisión: ${baseEmission}
Controles: Click=ráfaga • arrastra portales • C=tipo • P=portal +/- • [ ]=emis • R=reset`,
     12, height - 30
  );
}

// ---------------- Curl Noise ----------------
function curlNoise(x, y, tOff = 0) {
  const eps = 0.0005;
  let n1 = noise(x + eps, y, t + tOff);
  let n2 = noise(x - eps, y, t + tOff);
  let a = (n1 - n2) / (2 * eps);

  let n3 = noise(x, y + eps, t + tOff);
  let n4 = noise(x, y - eps, t + tOff);
  let b = (n3 - n4) / (2 * eps);

  return createVector(b, -a);
}

// ---------------- Sistema y Partículas ----------------
class ParticleSystem {
  constructor(origin) {
    this.origin = origin.copy();
    this.particles = [];
  }
  addParticle(type = 0, burst = false) {
    let p;
    if (type === 0) p = new GlowParticle(this.origin.x, this.origin.y, burst);
    else if (type === 1) p = new ShardParticle(this.origin.x, this.origin.y, burst);
    else p = new RibbonParticle(this.origin.x, this.origin.y, burst);
    this.particles.push(p);
  }
  run(trailGfx) {
    for (let i = this.particles.length - 1; i >= 0; i--) {
      const p = this.particles[i];
      p.update();
      p.drawTrail(trailGfx);
      if (p.isDead() || p.outOfBounds()) this.particles.splice(i, 1);
    }
  }
  drawFront() {
    for (let p of this.particles) p.display();
  }
  isEmpty() { return this.particles.length === 0; }
}

class Particle {
  constructor(x, y, burst = false) {
    this.pos = createVector(x, y);
    this.vel = p5.Vector.random2D().mult(burst ? random(2.0, 4.2) : random(0.6, 2.0));
    this.acc = createVector(0, 0);
    this.size = random(3, 7);
    this.h = random(180, 260);
    this.s = random(45, 95);
    this.b = random(70, 100);
    this.alpha = 255;
    this.lifespan = random(160, 260);
    this.decay = random(1.1, 2.1);
    this.prev = this.pos.copy();
  }
  applyForce(f) { this.acc.add(f); }
  update() {
    this.vel.add(this.acc);
    this.pos.add(this.vel);
    this.acc.mult(0);
    this.lifespan -= this.decay;
    this.alpha = map(this.lifespan, 0, 200, 0, 255, true);
  }
  drawTrail(g) {
    g.push();
    g.stroke(this.h, this.s * 0.4, 100, 40);
    g.strokeWeight(1.2);
    g.line(this.prev.x, this.prev.y, this.pos.x, this.pos.y);
    g.pop();
    this.prev.set(this.pos);
  }
  display() {
    fill(this.h, this.s, this.b, this.alpha);
    ellipse(this.pos.x, this.pos.y, this.size);
  }
  isDead() { return this.lifespan <= 0; }
  outOfBounds() {
    const m = 250;
    return this.pos.x < -m || this.pos.x > width + m || this.pos.y < -m || this.pos.y > height + m;
  }
}

class GlowParticle extends Particle {
  constructor(x, y, burst=false) {
    super(x, y, burst);
    this.glow = random(10, 22);
  }
  drawTrail(g) {
    g.push();
    g.noStroke();
    g.fill(this.h, this.s * 0.35, 100, 18);
    g.ellipse(this.pos.x, this.pos.y, this.glow * 0.9);
    g.pop();
    super.drawTrail(g);
  }
  display() {
    fill(this.h, this.s, this.b, this.alpha);
    ellipse(this.pos.x, this.pos.y, this.size + 1);
  }
}

class ShardParticle extends Particle {
  constructor(x, y, burst=false) {
    super(x, y, burst);
    this.angle = random(TWO_PI);
    this.spin = random(-0.09, 0.09);
    this.w = random(4, 10);
    this.hRect = random(2, 6);
  }
  update() {
    super.update();
    this.angle += this.spin;
  }
  display() {
    push();
    translate(this.pos.x, this.pos.y);
    rotate(this.angle);
    noStroke();
    fill(this.h, this.s, this.b, this.alpha);
    rectMode(CENTER);
    rect(0, 0, this.w, this.hRect, 1);
    pop();
  }
}

class RibbonParticle extends Particle {
  constructor(x, y, burst=false) {
    super(x, y, burst);
    this.path = [this.pos.copy()];
    this.maxLen = 12;
  }
  update() {
    super.update();
    this.path.push(this.pos.copy());
    if (this.path.length > this.maxLen) this.path.shift();
  }
  drawTrail(g) {
    g.push();
    g.noFill();
    g.stroke(this.h, this.s * 0.6, 100, 35);
    g.strokeWeight(2);
    for (let i = 1; i < this.path.length; i++) {
      const a = map(i, 1, this.path.length - 1, 10, 60);
      g.stroke(this.h, this.s * 0.6, 100, a);
      g.line(this.path[i - 1].x, this.path[i - 1].y, this.path[i].x, this.path[i].y);
    }
    g.pop();
  }
  display() {
    fill(this.h, this.s, this.b, this.alpha);
    ellipse(this.pos.x, this.pos.y, this.size + 2);
  }
}

// ---------------- Portales ----------------
class Portal {
  constructor(x, y, isAttractor = true) {
    this.pos = createVector(x, y);
    this.isAttractor = isAttractor;
    this.r = 28;
  }
  set(x, y) { this.pos.set(x, y); }
  over(mx, my) { return p5.Vector.dist(this.pos, createVector(mx, my)) < this.r + 6; }
  forceOn(p) {
    let dir = p5.Vector.sub(this.pos, p.pos);
    let d = constrain(dir.mag(), 8, 260);
    dir.normalize();
    let k = (this.isAttractor ? 1 : -1) * 1.1 / (d * 0.06);
    return p5.Vector.mult(dir, k);
  }
  display() {
    push();
    translate(this.pos.x, this.pos.y);
    noFill();
    stroke(this.isAttractor ? color(140, 70, 100, 90) : color(0, 80, 100, 90));
    strokeWeight(2);
    circle(0, 0, this.r * 2);
    fill(this.isAttractor ? color(140, 70, 100, 160) : color(0, 80, 100, 160));
    ellipse(0, 0, 10, 10);
    pop();
  }
}
```

<img width="897" height="568" alt="Captura de pantalla 2025-09-18 103216" src="https://github.com/user-attachments/assets/75dd19ab-72db-4bc4-95d8-db2c1a3bea6c" />
<img width="898" height="554" alt="Captura de pantalla 2025-09-18 103227" src="https://github.com/user-attachments/assets/9d97a29e-6b03-4137-9397-0d2ca4c31bf6" />
<img width="904" height="563" alt="Captura de pantalla 2025-09-18 103310" src="https://github.com/user-attachments/assets/fc7fde46-32ed-4147-b71d-06a85ceb2580" />
<img width="900" height="561" alt="Captura de pantalla 2025-09-18 103327" src="https://github.com/user-attachments/assets/390ef550-1150-4615-8543-70f5ea5a4c9f" />










































