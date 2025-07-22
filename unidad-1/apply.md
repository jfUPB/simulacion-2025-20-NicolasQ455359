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
[Ver mi sketch en p5.js - Actividad 05](https://editor.p5js.org/NicolasQ455359/sketches/0hkbpPBPA)





