# Unidad 2

## 🔎 Fase: Set + Seek

## Actividad 01. 
En p5.js, los vectores se manejan con la clase p5.Vector. Para sumarlos correctamente, no puedo usar el símbolo + como lo haría con números. En su lugar, debo usar el método .add(), que está diseñado para trabajar con objetos p5.Vector.

Por ejemplo, si tengo un vector de posición y otro de velocidad, la forma correcta de sumarlos es así:

```js
position.add(velocity);
```
Esta instrucción actualiza directamente el vector position, sumándole la velocidad componente a componente.

### ¿Por qué esta línea position = position + velocity; no funciona?
Esa línea no funciona porque en JavaScript, y específicamente en p5.js, los vectores (position y velocity) son objetos, no números. El operador + no está definido para sumar objetos p5.Vector. Lo que hace en realidad es convertir ambos objetos a cadenas de texto (o simplemente genera un resultado inválido), lo que rompe la lógica del programa.

Por eso, si quiero que la posición cambie correctamente con la velocidad en cada cuadro del sketch, debo usar el método .add() así:

```js
position.add(velocity);
```
De lo contrario, no se estaría realizando la suma vectorial que necesito para mover la pelota.

## Actividad 02.
* Implementé el walker iniciando en el centro del canvas.

* Asigné probabilidades discretas para moverme: 26 % a la derecha, 26 % abajo, 24 % izquierda, 24 % arriba.

* Aplicación de wrap-around al salir del canvas.

* Utilicé un tamaño de paso fijo (1 pixel).

* Dejé la traza visible para apreciar el sesgo en la dirección acumulada.

  ### Codigo:
  ```js
  class Walker {
  constructor() {
    this.x = width / 2;
    this.y = height / 2;
  }

  step() {
    let r = random(1);
    if (r < 0.26) {
      this.x++;           // 26% mover a la derecha
    } else if (r < 0.52) {
      this.y++;           // siguiente 26% mover hacia abajo
    } else if (r < 0.76) {
      this.x--;           // 24% mover hacia la izquierda
    } else {
      this.y--;           // 24% mover hacia arriba
    }
    // wrap por los bordes
    if (this.x < 0) this.x = width - 1;
    if (this.x >= width) this.x = 0;
    if (this.y < 0) this.y = height - 1;
    if (this.y >= height) this.y = 0;
  }

  show() {
    stroke(0);
    point(this.x, this.y);
  }
  }

  let walker;

  function setup() {
  createCanvas(640, 360);
  walker = new Walker();
  background(255);
  }

  function draw() {
  walker.step();
  walker.show();
  }
  ```

