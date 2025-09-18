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



































