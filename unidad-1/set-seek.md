# Unidad 1

## 🔎 Fase: Set + Seek

## ¿Qué aprendí sobre la aleatoriedad?
La aleatoriedad es fundamental para crear comportamientos impredecibles y naturales tanto en arte generativo como en experiencias interactivas. Permite romper la repetición y generar variabilidad, aportando naturalidad y vida a sistemas visuales o interactivos.

## Ejemplo claro de esto:
En la obra de Sofía Lezcano, la aleatoriedad permite que su visualización nunca sea la misma. El uso de ruido Perlin y otras técnicas hace que cada iteración sea única.

## Diferencia entre distribuciones:
Una distribución **uniforme** reparte los valores por igual, mientras que una **no uniforme** favorece ciertas zonas. Esto permite manipular los resultados a conveniencia para lograr ciertos efectos visuales o de comportamiento.

## Sobre el ruido Perlin:
El ruido Perlin produce cambios suaves y progresivos, ideal para simular naturaleza. Es diferente a la aleatoriedad pura, que genera cambios bruscos.

## En mi perfil profesional:
Como estudiante de Ingeniería en Diseño de Entretenimiento Digital, mi interés está enfocado en la creación de experiencias interactivas, especialmente en entornos como Unreal Engine, donde la aleatoriedad es clave para diseñar mundos más inmersivos y creíbles.

Por ejemplo:
En la generación procedural de niveles o entornos, se utiliza aleatoriedad para crear terrenos, bosques, ciudades o mazmorras que nunca sean iguales en dos partidas distintas. Esto mantiene el interés del jugador y amplía la rejugabilidad.
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
<img width="1880" height="920" alt="image" src="https://github.com/user-attachments/assets/83a866f1-a2ad-4bcc-a619-bc68cbecae13" />

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









