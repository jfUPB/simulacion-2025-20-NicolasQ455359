# Unidad 1

## 🛠 Fase: Apply

## Actividad 3 - Caminata Aleatoria

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

## Actividad 4 - Distribución No Uniforme

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






