# Unidad 3


## 🛠 Fase: Apply

## Actividad 10

En mi obra generativa, modelé el problema de los n-cuerpos creando varios cuerpos con masa, posición y velocidad. Cada cuerpo ejerce una fuerza gravitacional sobre los demás, calculada en cada frame usando vectores, lo que hace que las trayectorias cambien constantemente. Además, añadí interactividad para que pueda arrastrar un cuerpo con el mouse, lo que altera el movimiento de todos los demás cuerpos en tiempo real.

Los dos algoritmos que utilicé son:

Vectores: Para calcular posiciones, velocidades, aceleraciones y fuerzas entre los cuerpos.

Random: Para generar las posiciones iniciales, masas y colores de los cuerpos de manera dinámica y distinta cada vez que se ejecuta la simulación.

El sistema simula la complejidad de un n-cuerpos mientras genera un efecto visual similar a un móvil cinético, donde la posición de un elemento afecta a todo el conjunto.

[Simulación de n-cuerpos en p5.js](https://editor.p5js.org/NicolasQ455359/sketches/9CmCLTrLn)

```javascript
let bodies = [];
let G = 0.5; // constante gravitacional
let numBodies = 6;

function setup() {
  createCanvas(700, 700);
  for (let i = 0; i < numBodies; i++) {
    let mass = random(10, 25);
    let position = createVector(random(width), random(height));
    let velocity = p5.Vector.random2D().mult(random(0.5, 2));
    bodies.push(new Body(position, velocity, mass));
  }
}

function draw() {
  background(30, 30, 50);

  // Dibujar conexiones tipo móvil
  stroke(200, 150);
  for (let i = 0; i < bodies.length; i++) {
    for (let j = i + 1; j < bodies.length; j++) {
      line(bodies[i].position.x, bodies[i].position.y,
           bodies[j].position.x, bodies[j].position.y);
    }
  }

  // Actualizar cada cuerpo
  for (let i = 0; i < bodies.length; i++) {
    bodies[i].applyGravity(bodies);
    bodies[i].update();
    bodies[i].checkEdges();
    bodies[i].display();
  }
}

// Clase para los cuerpos
class Body {
  constructor(position, velocity, mass) {
    this.position = position;
    this.velocity = velocity;
    this.mass = mass;
    this.acceleration = createVector(0, 0);
    this.radius = mass;
    this.color = color(random(100, 255), random(100, 255), random(100, 255));
  }

  applyGravity(others) {
    this.acceleration.mult(0); // resetear aceleración
    for (let other of others) {
      if (other != this) {
        let force = p5.Vector.sub(other.position, this.position);
        let distance = constrain(force.mag(), 20, 200); 
        let strength = (G * this.mass * other.mass) / (distance * distance);
        force.setMag(strength);
        this.acceleration.add(force.div(this.mass));
      }
    }
  }

  update() {
    this.velocity.add(this.acceleration);
    this.position.add(this.velocity);
  }

  display() {
    noStroke();
    fill(this.color);
    ellipse(this.position.x, this.position.y, this.radius*2);
  }

  checkEdges() {
    if (this.position.x < 0 || this.position.x > width) this.velocity.x *= -1;
    if (this.position.y < 0 || this.position.y > height) this.velocity.y *= -1;
  }
}

// Interactividad: arrastrar cuerpos con el mouse
function mouseDragged() {
  for (let body of bodies) {
    let d = dist(mouseX, mouseY, body.position.x, body.position.y);
    if (d < body.radius*2) {
      body.position.x = mouseX;
      body.position.y = mouseY;
      body.velocity.mult(0); // detener mientras se arrastra
    }
  }
}
```
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/cb7bd9f7-919e-4f46-bb96-c3ccbf8064a6" />




