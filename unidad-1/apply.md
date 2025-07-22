# Unidad 1

## 🛠 Fase: Apply

## Actividad 8 - Creación de obra generativa interactiva en tiempo real

### Concepto de obra generativa: Campos de viento

Se me ocurrió la idea de generar movimientos suaves y caoticos que hacen parecer corrientes de viento, será representado por medio de partículas que flotan y son arrastradas por corrientes de aire invisibles. Busco representar como pequeñas cosas intractuan con elementos naturales, en este caso el viento y que sean aleatorias para crear patrones.

Los 3 conceptos que usaré son:

Ruido Perlin para dar dirección fluida a las partículas.

Aleatoriedad inicial para su posición y velocidad.

Lévy Flight para movimientos inesperados al hacer clic.

### Interactividad:

Click empuja las partículas de forma repentina.

Teclado añade nuevas partículas aleatorias al sistema en cualquier momento.

Codigo: 

```Javascript
let particles = [];

function setup() {
  createCanvas(800, 600);
  for (let i = 0; i < 300; i++) {
    particles.push(new Particle(random(width), random(height)));
  }
  background(255);
}

function draw() {
  background(255, 20);
  for (let p of particles) {
    p.update();
    p.show();
  }
}

function mousePressed() {
  for (let p of particles) {
    let force = p5.Vector.random2D();
    force.mult(random(20, 50));
    p.applyForce(force);
  }
}

function keyPressed() {
  for (let i = 0; i < 10; i++) {
    particles.push(new Particle(random(width), random(height)));
  }
}

class Particle {
  constructor(x, y) {
    this.pos = createVector(x, y);
    this.vel = p5.Vector.random2D();
    this.acc = createVector(0, 0);
    this.t = random(1000);
  }

  applyForce(force) {
    this.acc.add(force);
  }

  update() {
    let angle = noise(this.pos.x * 0.002, this.pos.y * 0.002, this.t) * TWO_PI * 4;
    let force = p5.Vector.fromAngle(angle);
    force.mult(0.5);
    this.applyForce(force);

    // Lévy Flight ocasional
    if (random(1) < 0.002) {
      let jump = p5.Vector.random2D();
      jump.mult(pow(random(1), 4) * 100);
      this.applyForce(jump);
    }

    this.vel.add(this.acc);
    this.vel.limit(3);
    this.pos.add(this.vel);
    this.acc.mult(0);
    this.t += 0.01;

    this.edges();
  }

  show() {
    noStroke();
    fill(0, 50);
    ellipse(this.pos.x, this.pos.y, 3);
  }

  edges() {
    if (this.pos.x > width) this.pos.x = 0;
    if (this.pos.x < 0) this.pos.x = width;
    if (this.pos.y > height) this.pos.y = 0;
    if (this.pos.y < 0) this.pos.y = height;
  }
}
```
[Ver mi sketch en p5.js - Actividad 8](https://editor.p5js.org/NicolasQ455359/sketches/PMaaKLjFS)
<img width="1868" height="883" alt="image" src="https://github.com/user-attachments/assets/70630d3f-6c31-4a66-bbcc-45f1b40a66c7" />




