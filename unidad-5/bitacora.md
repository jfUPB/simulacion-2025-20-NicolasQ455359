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





























