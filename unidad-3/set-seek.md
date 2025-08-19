# Unidad 3

## 🔎 Fase: Set + Seek

## Actividad 01.

video: El video destaca cómo Alexander Calder transformó la escultura tradicional al incorporar movimiento y abstracción. Desde sus inicios, Calder mostró una fascinación por el movimiento, lo que lo llevó a crear esculturas móviles que desafiaban las convenciones artísticas de su tiempo. Su obra más emblemática, el "mobile", introdujo el concepto de escultura en movimiento, fusionando arte y física en una danza visual.

## Actividad 02.

El proyecto Data Structure demuestra cómo el arte y la tecnología pueden converger para hacer que conceptos abstractos, como los datos, sean comprensibles y atractivos para el público. Al crear instalaciones que combinan lo digital y lo físico, puedes ofrecer experiencias que no solo informan, sino que también inspiran y conectan emocionalmente con los visitantes.

## Actividad 03.

Texto guia.

## Actividad 04.

Resumen del Marco Motion 101:
Se usan tres factores clave:

Posición: Donde se encuentra el objeto.

Velocidad: Qué tan rápido se mueve el objeto.

Aceleración: Qué tan rápido cambia la velocidad del objeto.

Este código simula un objeto en movimiento controlado por fuerzas aplicadas a él. 

## Actividad 05.

### ¿Qué problema le ves a este planteamiento?

Lo que veo como un problema en este planteamiento es que, al utilizar el método applyForce, la aceleración del objeto se sobrescribe cada vez que se aplica una nueva fuerza. Esto significa que, si hay múltiples fuerzas actuando sobre el objeto, en lugar de sumarse, las fuerzas se están reemplazando entre sí, lo que no refleja la realidad física de cómo deberían interactuar las fuerzas.

### ¿Qué solución propones?

La solución es que en lugar de sobrescribir la aceleración con cada nueva fuerza, debemos acumular las fuerzas. Es decir, que cada vez que se aplique una fuerza, esta se agregue a la aceleración actual, y no la reemplace. Esto permitirá que todas las fuerzas aplicadas se sumen correctamente y afecten al movimiento del objeto de manera más realista.

### ¿Cómo lo implementarías en p5.js?

En p5.js, implementaría esta solución de la siguiente manera:

Modificaría el método applyForce para que acumule las fuerzas:

```javascript
applyForce(force) {
  this.acceleration.add(force);  // Acumula las fuerzas en la aceleración
}
```
En el ciclo de dibujo, aplicaría las diferentes fuerzas como la gravedad y el viento y luego llamaría a update() para actualizar la posición, velocidad y aceleración del objeto:

```javascript
let gravity = createVector(0, 0.1);  // Gravedad hacia abajo
let wind = createVector(0.1, 0);  // Viento hacia la derecha

mover.applyForce(gravity);  // Aplicamos la gravedad
mover.applyForce(wind);  // Aplicamos el viento
mover.update();  // Actualizamos la posición del objeto
```
Con esta solución, cada fuerza se va acumulando y afectando el movimiento del objeto de manera correcta, respetando la física que queremos modelar.

## Actividad 06.

### ¿Por qué es necesario multiplicar la aceleración por cero en cada frame?

Es necesario multiplicar la aceleración por cero al final de cada frame para reiniciar su valor, ya que la aceleración se calcula como la suma de las fuerzas aplicadas en el frame actual. Si no se resetea, la aceleración de los frames anteriores seguiría acumulándose, lo que afectaría incorrectamente el movimiento del objeto. Al reiniciar la aceleración, nos aseguramos de que cada frame calcule la aceleración de manera independiente, solo con las fuerzas aplicadas en ese momento.

### ¿Por qué se multiplica por cero justo al final de update()?

Multiplicar la aceleración por cero al final de update() asegura que no se acumule información de aceleración de frames anteriores. La aceleración debe ser calculada en cada frame en función de las fuerzas actuales, no de las pasadas. Al hacerlo, evitamos que fuerzas previas interfieran con los cálculos del siguiente frame, lo que garantizará que el objeto se mueva correctamente en cada actualización sin influencias incorrectas.

## Actividad 07. 

Lo que ocurre es que al pasar un objeto como referencia, cualquier modificación en esa referencia dentro de la función afecta al objeto original. En el caso de la fuerza, queremos modificar solo la copia de la fuerza en lugar de alterar el objeto original, por eso la solución es hacer una copia del vector force antes de dividirlo.

## Actividad 08. 

### ¿Cuál es la diferencia entre las dos líneas?

Primera línea (let friction = this.velocity.copy();): Aquí estamos creando una copia independiente de this.velocity. El método copy() crea un nuevo objeto con los mismos valores, lo que significa que friction y this.velocity son dos objetos separados. Los cambios en uno no afectarán al otro.

Segunda línea (let friction = this.velocity;): En este caso, estamos referenciando el mismo objeto. Esto significa que friction y this.velocity apuntan al mismo objeto en memoria. Cualquier cambio en uno de ellos afectará al otro, ya que ambos son el mismo objeto.

### ¿Qué podría salir mal con let friction = this.velocity;?

El problema con esta línea es que friction y this.velocity comparten el mismo objeto en memoria. Si modificas friction, también estarás modificando this.velocity, lo cual puede generar resultados inesperados si no se desea que ambos objetos estén vinculados. Si necesitas que friction sea independiente de this.velocity, esta asignación no funcionará correctamente, ya que ambas variables afectarán al mismo objeto.

### ¿Cuál es la diferencia entre copiar por VALOR y por REFERENCIA?

Copia por valor: Cuando se hace una copia por valor, se crea una nueva instancia del objeto o valor. Cualquier cambio en la copia no afecta al original. Por ejemplo, en let friction = this.velocity.copy();, friction es una copia independiente de this.velocity.

Copia por referencia: Cuando se hace una copia por referencia, las dos variables apuntan al mismo objeto en memoria. Los cambios realizados en una variable afectan a la otra, ya que ambas hacen referencia al mismo objeto. Esto ocurre en let friction = this.velocity;, donde friction y this.velocity están referenciando el mismo objeto.

En el fragmento de código, ¿cuándo es por VALOR y cuándo por REFERENCIA?
Por valor: En la línea let friction = this.velocity.copy();, this.velocity.copy() crea una copia independiente del objeto, por lo que estamos trabajando con copia por valor.

Por referencia: En la línea let friction = this.velocity;, this.velocity no se copia, sino que se asigna directamente a friction. Ambas variables apuntan al mismo objeto en memoria, lo que significa que estamos trabajando con copia por referencia.

## Actividad 09. 
### 1.1 Fricción - Cómo modelé la fuerza:
Para modelar la fricción, creé partículas que se generan cuando el usuario hace clic en el lienzo. Cada partícula se mueve de manera aleatoria, pero a medida que el tiempo pasa, la fricción actúa sobre ellas, reduciendo su velocidad gradualmente hasta que se detienen. La fricción se modela utilizando el mult(friction) sobre el vector de velocidad de cada partícula, lo que disminuye su velocidad con cada actualización del cuadro. Esto simula cómo la fricción reduce el movimiento de un objeto con el tiempo.

### 1.2 Conceptualmente cómo se relaciona la fricción con la obra:

La fricción es la fuerza que se opone al movimiento de las partículas y, al igual que en la vida real, va reduciendo su velocidad hasta que finalmente se detienen. En esta obra generativa, el usuario puede interactuar con las partículas (empujándolas al hacer clic), y al observarlas, puede ver cómo la fricción actúa lentamente sobre ellas, disipando su energía. Esto refleja visualmente cómo la fricción reduce el movimiento y el dinamismo de los objetos con el tiempo.

[Fricción - Proyecto en p5.js](https://editor.p5js.org/NicolasQ455359/sketches/NYPEdrOJt)

### 1.3 Codigo
```javascript
let particles = [];
let friction = 0.98; // Coeficiente de fricción

function setup() {
  createCanvas(600, 600);
}

function draw() {
  background(0);
  
  // Crear partículas cada vez que se hace clic
  if (mouseIsPressed) {
    particles.push(new Particle(mouseX, mouseY));
  }
  
  // Actualizar y dibujar todas las partículas
  for (let i = particles.length - 1; i >= 0; i--) {
    particles[i].applyFriction();
    particles[i].update();
    particles[i].display();
    
    // Eliminar partículas que se detienen
    if (particles[i].isStopped()) {
      particles.splice(i, 1);
    }
  }
}

// Clase para las partículas
class Particle {
  constructor(x, y) {
    this.pos = createVector(x, y);
    this.vel = createVector(random(-3, 3), random(-3, 3));
    this.acc = createVector(0, 0);
    this.size = random(5, 10);
  }
  
  applyFriction() {
    // La fricción reduce la velocidad de la partícula
    this.vel.mult(friction);
  }
  
  update() {
    this.pos.add(this.vel);
  }
  
  display() {
    fill(255, 150);
    noStroke();
    ellipse(this.pos.x, this.pos.y, this.size);
  }
  
  isStopped() {
    return this.vel.mag() < 0.05;
  }
}
```
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/fc4534c6-7826-48e7-945b-77a2f232bf07" />

### 2.1. Resistencia del aire y fluidos - Cómo modelé la fuerza:
Para modelar la resistencia del aire, creé un barco que se mueve por el canvas. Este barco tiene una fuerza de viento que lo impulsa en una dirección, pero la resistencia del aire actúa para desacelerarlo. La resistencia se aplica de manera que siempre intenta frenar el movimiento del barco, lo que se traduce en un cambio en su velocidad conforme se desplaza por el fluido. Además, añadí un fondo fluido que simula agua o aire en movimiento, haciendo que la interacción con el barco se vea más natural y visualmente atractiva.

### 2.2 Cómo se relaciona la fuerza con la obra generativa:
La resistencia del aire es una fuerza que se opone al movimiento de los objetos que pasan a través de un fluido. En la obra generativa, el barco se mueve por el canvas impulsado por el viento, pero a medida que avanza, la resistencia del aire lo desacelera, modificando su velocidad. El barco cambia de dirección al tocar los bordes del canvas, lo que genera un movimiento continuo e impredecible, representando cómo los objetos se ven afectados por la resistencia del fluido a lo largo del tiempo. Este comportamiento es generativo, ya que cada vez que se ejecuta, el barco se mueve de manera diferente según las fuerzas aplicadas.

[Resistencia del aire y fluidos – Proyecto en p5.js](https://editor.p5js.org/NicolasQ455359/sketches/I0ehdDuOQ)

### 2.3 Codigo
```javascript
let boat;
let wind;
let resistance = 0.1;  // Resistencia del aire ajustada para hacerlo más visual

function setup() {
  createCanvas(600, 400);
  boat = new Boat(width / 4, height / 2);  // El barco comienza en una posición inicial
  wind = createVector(0.1, 0); // Dirección del viento hacia la derecha
  noStroke();
}

function draw() {
  // Fondo dinámico que simula el agua (con un toque de fluido)
  drawFluidBackground();
  
  boat.applyForce(wind); // Aplica la fuerza del viento al barco
  boat.applyResistance(resistance); // Aplica la resistencia del aire
  boat.update(); // Actualiza la posición y el movimiento del barco
  boat.display(); // Dibuja el barco en la pantalla

  // Asegura que el barco no se salga del canvas
  boat.checkEdges();  // Verifica si toca el borde
  boat.limitToCanvas(); // Asegura que no se salga del canvas
}

// Función para dibujar un fondo que simula el agua
function drawFluidBackground() {
  let waterColor = color(0, 102, 204, 150); // Azul agua translúcido
  fill(waterColor);
  for (let i = 0; i < width; i += 20) {
    let offset = map(sin(i * 0.02 + frameCount * 0.05), -1, 1, -5, 5); // Movimiento de agua
    rect(i, 0, 20, height + offset); // Representando las ondas de agua
  }
}

// Clase para el barco
class Boat {
  constructor(x, y) {
    this.position = createVector(x, y); // Posición inicial del barco
    this.velocity = createVector(2, 1); // Velocidad inicial (aquí está moviéndose en ambas direcciones)
    this.acceleration = createVector(0, 0); // Aceleración inicial
    this.size = 50; // Tamaño del barco
    this.angle = 0; // Ángulo de inclinación
  }

  // Aplica una fuerza al barco
  applyForce(force) {
    let f = force.copy();
    this.acceleration.add(f);
  }

  // Aplica la resistencia del aire sobre el barco
  applyResistance(resistance) {
    let resistanceForce = this.velocity.copy();
    resistanceForce.normalize(); // Normaliza la velocidad
    resistanceForce.mult(-resistance); // Aplica la resistencia en dirección opuesta
    this.applyForce(resistanceForce); // Suma la resistencia
  }

  // Actualiza la posición del barco
  update() {
    // Movimiento aleatorio del barco en el fluido (simula interacción)
    let randomForce = createVector(random(-0.2, 0.2), random(-0.2, 0.2));
    this.applyForce(randomForce);

    // Actualización de la velocidad y la posición del barco
    this.velocity.add(this.acceleration);
    this.position.add(this.velocity);

    // Limita la velocidad del barco
    this.velocity.limit(3);
    this.acceleration.mult(0); // Reinicia la aceleración
  }

  // Limita el movimiento del barco para que no se salga del canvas
  limitToCanvas() {
    if (this.position.x < 0) this.position.x = 0;
    if (this.position.x > width) this.position.x = width;
    if (this.position.y < 0) this.position.y = 0;
    if (this.position.y > height) this.position.y = height;
  }

  // Cambia la dirección del barco cuando toca el borde
  checkEdges() {
    // Detecta si el barco toca el borde en los 4 lados
    if (this.position.x <= 0 || this.position.x >= width) {
      this.velocity.x *= -1; // Invertir la dirección en X cuando toca el borde horizontal
    }
    if (this.position.y <= 0 || this.position.y >= height) {
      this.velocity.y *= -1; // Invertir la dirección en Y cuando toca el borde vertical
    }
  }

  // Dibuja el barco en la pantalla
  display() {
    fill(255, 100, 100); // Color del barco
    push();
    translate(this.position.x, this.position.y); // Mueve el barco a su posición
    rotate(this.angle); // Rota el barco para simular el viento
    // Dibuja el barco (como un triángulo que representa el casco)
    triangle(-this.size / 2, this.size / 4, 
             this.size / 2, this.size / 4, 
             0, -this.size / 2);
    pop();
  }
}
```
<img width="1869" height="884" alt="image" src="https://github.com/user-attachments/assets/39cf1676-4b5d-4d3f-ba9d-55ea56c8080a" />

### 3.1 Atracción Gravitacional - Cómo modelé la fuerza:
Para modelar la atracción gravitacional, creé varios planetas que interactúan entre sí en un espacio bidimensional. La gravedad se aplica entre los planetas utilizando la ley de la gravitación universal de Newton. Cada planeta ejerce una fuerza gravitacional sobre los demás en función de sus masas y la distancia entre ellos. El movimiento de los planetas es afectado por esta fuerza, lo que hace que cambien constantemente su trayectoria. Además, agregué una función para hacer que los planetas rebote al llegar a los bordes del canvas, para que siempre permanezcan dentro del lienzo.

### 3.2 Cómo se relaciona la fuerza con la obra generativa:
La gravedad es una fuerza que atrae a los cuerpos con masa entre sí. En esta obra generativa, los planetas se mueven de manera orgánica, siendo atraídos por la gravedad de los otros planetas, lo que hace que sus trayectorias cambien constantemente. Como se trata de una obra generativa, cada vez que ejecutes el código, los planetas tendrán trayectorias diferentes, ya que la interacción entre ellos y la gravedad cambia constantemente. Además, al tocar los bordes del canvas, el planeta rebota, lo que da un efecto visual dinámico y evita que los planetas salgan del lienzo, creando una simulación siempre nueva y diferente.

[Atracción Gravitacional – Proyecto en p5.js](https://editor.p5js.org/NicolasQ455359/sketches/-g9XGSeMj)

### 3.3 Codigo
```javascript
let planets = [];
let gravityConstant = 0.1;

function setup() {
  createCanvas(600, 600);
  noStroke();

  // Crear planetas con posiciones, velocidades y masas aleatorias
  for (let i = 0; i < 5; i++) {
    let x = random(width);
    let y = random(height);
    let mass = random(5, 20);
    let velocity = createVector(random(-1, 1), random(-1, 1)); // Velocidades aleatorias
    planets.push(new Planet(x, y, mass, velocity));
  }
}

function draw() {
  background(0);

  // Dibujar estrellas de fondo para hacerlo más atractivo
  drawStars();

  // Actualizar y mostrar cada planeta
  for (let i = 0; i < planets.length; i++) {
    planets[i].update(planets);
    planets[i].display();
    planets[i].checkEdges(); // Verifica si el planeta toca el borde
  }
}

// Función para dibujar un fondo estrellado
function drawStars() {
  fill(255, 255, 255, 50);
  for (let i = 0; i < width; i += random(1, 5)) {
    ellipse(random(width), random(height), 2, 2); // Crear estrellas
  }
}

// Clase para los planetas
class Planet {
  constructor(x, y, mass, velocity) {
    this.position = createVector(x, y);
    this.velocity = velocity;
    this.mass = mass;
    this.acceleration = createVector(0, 0);
    this.radius = this.mass * 2;  // Tamaño del planeta en función de su masa
    this.color = color(random(100, 255), random(100, 255), random(100, 255));
  }

  // Actualiza la posición y la velocidad del planeta
  update(planets) {
    let force = createVector(0, 0);

    // Calcular la gravedad de cada otro planeta
    for (let i = 0; i < planets.length; i++) {
      if (planets[i] != this) {
        let direction = p5.Vector.sub(planets[i].position, this.position); // Dirección entre los planetas
        let distance = direction.mag();
        distance = constrain(distance, 20, 150); // Evitar la división por cero

        let strength = gravityConstant * (this.mass * planets[i].mass) / (distance * distance);
        direction.setMag(strength);  // Magnitud de la fuerza gravitacional
        force.add(direction);  // Sumar la fuerza al total
      }
    }

    // Aplicar la aceleración y actualizar la velocidad y la posición
    this.acceleration = force.copy().div(this.mass); // F = ma, entonces a = F / m
    this.velocity.add(this.acceleration);
    this.position.add(this.velocity);
  }

  // Muestra el planeta en la pantalla
  display() {
    fill(this.color);
    ellipse(this.position.x, this.position.y, this.radius, this.radius); // Dibujar el planeta
  }

  // Cambia la dirección del planeta cuando toca el borde del canvas
  checkEdges() {
    if (this.position.x <= 0 || this.position.x >= width) {
      this.velocity.x *= -1; // Invertir la dirección en X cuando toca el borde horizontal
    }
    if (this.position.y <= 0 || this.position.y >= height) {
      this.velocity.y *= -1; // Invertir la dirección en Y cuando toca el borde vertical
    }
  }
}
```
<img width="1919" height="1073" alt="image" src="https://github.com/user-attachments/assets/8a6432da-6142-4475-aa00-cf41c9e3dd93" />




















