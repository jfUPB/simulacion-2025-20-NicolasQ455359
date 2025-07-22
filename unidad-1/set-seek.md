# Unidad 1

## 🔎 Fase: Set + Seek

## ¿Qué aprendí sobre la aleatoriedad?
La aleatoriedad es fundamental para crear comportamientos impredecibles y naturales tanto en arte generativo como en experiencias interactivas. Permite romper la repetición y generar variabilidad, aportando naturalidad y vida a sistemas visuales o interactivos.

## Ejemplo Sodía Lezcano:
En la obra de Sofía Lezcano, la aleatoriedad permite que su visualización nunca sea la misma. El uso de ruido Perlin y otras técnicas hace que cada iteración sea única.

## Diferencia entre distribuciones:
Una distribución uniforme reparte los valores por igual, mientras que una no uniforme favorece ciertas zonas. Esto permite manipular los resultados a conveniencia para lograr ciertos efectos visuales o de comportamiento.

## Sobre el ruido Perlin:
El ruido Perlin produce cambios suaves y progresivos, ideal para simular naturaleza. Es diferente a la aleatoriedad pura, que genera cambios bruscos.

## En mi perfil profesional:
Como estudiante de Ingeniería en Diseño de Entretenimiento Digital, mi interés está enfocado en la creación de experiencias interactivas, especialmente en entornos como Unreal Engine, donde la aleatoriedad es clave para diseñar mundos más inmersivos y creíbles.

Por ejemplo:
En la generación de niveles o entornos, se utiliza aleatoriedad para crear terrenos, bosques o ciudades que nunca sean iguales en dos partidas distintas. Esto mantiene el interés del jugador y amplía la rejugabilidad.
En la simulación de comportamientos de NPCs (personajes no jugables), se pueden incluir patrones aleatorios para que reaccionen de manera diferente a situaciones similares, haciendo que la experiencia sea menos predecible y más natural.
En sistemas de partículas, física o materiales interactivos, la aleatoriedad permite simular fenómenos realistas como el fuego, el humo, el agua o incluso la dispersión de luz en ciertos materiales, mejorando el impacto visual de la experiencia.


## Actividad 3 - Caminata Aleatoria
### Antes:
Esperaba que el punto se moviera aleatoriamente sin alejarse demasiado del centro.
### Después:
El resultado fue como esperaba, pero modificar el rango de paso cambió el comportamiento de forma más agresiva de lo que suponía.
### Aprendizaje:
Pequeños cambios en los valores aleatorios alteran mucho el resultado visual.

### Código:
```javascript
let x, y;

function setup() {
  createCanvas(600, 600);
  background(220);
  x = width / 2;
  y = height / 2;
  stroke(0, 100, 200);
}

function draw() {
  point(x, y);
  let stepX = int(random(-10, 10));
  let stepY = int(random(-10, 10));
  x += stepX;
  y += stepY;
  x = constrain(x, 0, width);
  y = constrain(y, 0, height);
}
```
<img width="1872" height="879" alt="image" src="https://github.com/user-attachments/assets/dcfe85e8-7112-4311-aae2-61f7703cad91" />

### Explicación: El punto se mueve en pasos aleatorios pequeños en cualquier dirección. Cambié el rango de movimiento para que a veces se desplace más rápido.

[Ver mi sketch en p5.js - Actividad 3](https://editor.p5js.org/NicolasQ455359/sketches/2IGevG_j_)

## Actividad 4 - Distribución no uniforme
### Antes:
Esperaba ver más puntos hacia la derecha.
### Después:
Funcionó como lo planeé. Los puntos se agruparon hacia la derecha por la distribución no uniforme.
### Aprendizaje:
Controlar la distribución cambia el comportamiento visual drásticamente.

### Código:
```javascript
function setup() { 
  createCanvas(100, 100);
  background(200);
}

function draw() {
  noStroke();
  fill(0, 10);

  // Distribución no uniforme: favorece la derecha
  let r = random(1);
  let x;
  if (r < 0.8) {
    x = random(50, 100);
  } else {
    x = random(0, 50);
  }
  let y = 25;
  circle(x, y, 5);

  // Distribución Gaussiana desplazada hacia la derecha
  x = randomGaussian(70, 5);
  y = 50;
  circle(x, y, 5);

  // Otra Gaussiana aún más hacia la derecha
  x = randomGaussian(80, 10);
  y = 75;
  circle(x, y, 5);
}
```
<img width="1879" height="925" alt="image" src="https://github.com/user-attachments/assets/3a9ae627-0ff5-4444-90d8-35dea358c4b0" />

### Explicación: El código favorece valores hacia la derecha mediante una distribución no uniforme y Gaussianas desplazadas. Los puntos tienden a acumularse en la zona derecha del canvas.

[Ver mi sketch en p5.js - Actividad 4](https://editor.p5js.org/NicolasQ455359/sketches/0hkbpPBPA)

## Actividad 5 - Distribución Normal
### Antes:
Esperaba ver acumulación de puntos en el centro.
### Después:
El código mostró justo eso, además los tamaños de los círculos reforzaron la idea visualmente.
### Aprendizaje:
Las Gaussianas generan acumulaciones naturales, perfectas para representar fenómenos reales.

### Código:
```javascript
function setup() {
  createCanvas(600, 200);
  background(240);
  noStroke();
  fill(100, 150, 200, 150);
}

function draw() {
  let x = randomGaussian(width / 2, 60);
  let distanceToCenter = abs(width / 2 - x);
  let diameter = map(distanceToCenter, 0, width / 2, 20, 2);
  let y = random(height);
  ellipse(x, y, diameter);
}
```
<img width="1879" height="926" alt="image" src="https://github.com/user-attachments/assets/b9426891-f00e-4cf3-b133-268350966750" />

### Explicación: El código genera puntos con distribución normal centrada en el medio del canvas. Los círculos son más grandes cerca del centro para reforzar visualmente la acumulación. La visualización muestra cómo los valores se concentran alrededor de la media.

[Ver mi sketch en p5.js - Actividad 5](https://editor.p5js.org/NicolasQ455359/sketches/38CVLT4Az)


## Actividad 06 - Distribución Personalizada: Lévy Flight

### ¿Por qué usé esta técnica y qué resultados esperaba obtener?

El Lévy Flight genera un movimiento donde predominan los pasos pequeños, pero de forma inesperada ocurren saltos largos. Esto hace que la caminata sea más natural y orgánica, imitando comportamientos que vemos en la naturaleza, como los patrones de búsqueda de los animales.

Esperaba obtener un resultado donde el punto se desplazara lentamente, pero de vez en cuando hiciera saltos grandes, creando un recorrido mucho más visualmente dinámico e impredecible que una caminata aleatoria tradicional.

### 💻 Código:
```javascript
let position;

function setup() {
  createCanvas(600, 400);
  background(240);
  position = createVector(width / 2, height / 2);
  stroke(0, 50);
  strokeWeight(2);
}

function draw() {
  point(position.x, position.y);
  let step = p5.Vector.random2D();
  let r = pow(random(1), 4) * 100;
  step.mult(r);
  position.add(step);
  position.x = constrain(position.x, 0, width);
  position.y = constrain(position.y, 0, height);
}
```
<img width="1867" height="855" alt="image" src="https://github.com/user-attachments/assets/1730ede3-a770-4e4a-acb6-8a1a3c7ba457" />

[Ver mi sketch en p5.js - Actividad 6](https://editor.p5js.org/NicolasQ455359/sketches/Bayi-B5b5)

## Actividad 07 - Ruido Perlin

### Concepto, expectativas y resultados esperados:

El ruido Perlin genera valores aleatorios suaves, donde cada nuevo valor es cercano al anterior, lo que crea una sensación de continuidad natural. Es muy útil para simular fenómenos orgánicos, como terrenos, nubes, olas o movimiento fluido.

Al analizar la figura 0.4 del material, entendí que el ruido Perlin produce cambios suaves, a diferencia del ruido aleatorio tradicional que genera saltos bruscos.
Esperaba obtener una visualización con una línea continua y ondulante que muestre claramente esa progresión orgánica de valores.

Código
```javascript
let xoff = 0;
let yoff = 0;
let y;

function setup() {
  createCanvas(600, 200);
  background(240);
  stroke(50, 100, 200);
  noFill();
}

function draw() {
  background(240);
  beginShape();
  xoff = 0;
  for (let x = 0; x < width; x++) {
    let noiseVal = noise(xoff, yoff);
    y = noiseVal * height;
    vertex(x, y);
    xoff += 0.02;
  }
  endShape();
  yoff += 0.01;
}
```
<img width="1868" height="882" alt="image" src="https://github.com/user-attachments/assets/bdff5aa3-796c-4b5e-ab53-80f3cc6ae78d" />

[Ver mi sketch en p5.js - Actividad 7](https://editor.p5js.org/NicolasQ455359/sketches/4ISjLNsx8)












