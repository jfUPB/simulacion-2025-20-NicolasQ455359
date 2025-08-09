# Unidad 2


## 🛠 Fase: Apply

### Actividad 8. 

### Concepto
La obra representa una serie de formas geométricas (círculos concéntricos, anillos, ondas o rejillas) que se deforman, pulsan, giran o respiran como si tuvieran vida propia. Al mover el mouse, las figuras cambian su frecuencia, velocidad, tamaño o rotación, creando una ilusión óptica dinámica.

No hay partículas ni ruido visual. Todo se basa en vectores, rotaciones y transformaciones geométricas. La obra remite a lo orgánico, pero también a lo matemático. El resultado es una sensación de expansión, latido y energía contenida.

### Aplicación del marco Motion 101
Aunque no hay objetos que “viajan” en el espacio, el Motion 101 se aplica internamente en la transformación de cada línea curva o figura:

Cada punto en el anillo tiene una posición, una velocidad angular, y una aceleración angular (que depende del mouse).

Se usan vectores polares y su conversión a coordenadas cartesianas.

Esto permite que las curvas "vibren" como si tuvieran masa, dirección y fuerza.

### Algoritmo de aceleración
En lugar de usar aceleración hacia un punto, uso aceleración angular dependiente del movimiento del mouse.

θ + = ω ; ω + = α

Donde:

θ es el ángulo de rotación.

ω es la velocidad angular.

α es una aceleración que varía con mouseX.

Esto permite una sensación de “pulso controlado” que reacciona al usuario.

### Interactividad 
 * MouseX controla la aceleración angular (gira más o menos rápido).

 * Las formas se transforman en tiempo real, creando una ilusión hipnótica.

 * El color varía suavemente con frameCount, dándole vida y dinamismo.

 * La imagen nunca es igual: cada segundo es una nueva composición visual.

### Codigo

```js
let angle = 0;
let angularVelocity = 0;
let angularAcceleration = 0;

function setup() {
  createCanvas(800, 800);
  colorMode(HSB, 360, 100, 100);
  noFill();
  strokeWeight(1.5);
}

function draw() {
  background(0, 0, 0, 0.1);

  translate(width / 2, height / 2);

  // Control de aceleración con el mouse
  angularAcceleration = map(mouseX, 0, width, -0.001, 0.001);
  angularVelocity += angularAcceleration;
  angle += angularVelocity;
  angularVelocity *= 0.98; // fricción suave

  rotate(angle);

  let rings = 25;
  let points = 100;

  for (let r = 50; r < rings * 10; r += 10) {
    beginShape();
    stroke((frameCount + r) % 360, 80, 100);
    for (let i = 0; i < points; i++) {
      let theta = map(i, 0, points, 0, TWO_PI);
      let x = cos(theta) * r;
      let y = sin(theta) * r;

      // Distorsión con oscilación
      let offset = sin(theta * 6 + frameCount * 0.05) * 10;
      vertex(x + offset, y + offset);
    }
    endShape(CLOSE);
  }
}
```
### Enlace 
[https://editor.p5js.org/NicolasQ455359/sketches/c8brF6kd4](https://editor.p5js.org/NicolasQ455359/sketches/c8brF6kd4)

### Captura

<img width="1919" height="1054" alt="image" src="https://github.com/user-attachments/assets/231a11a1-4a5d-4581-8997-b07b1391fcaf" />

