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

## Actividad 03.

### ¿Qué resultado esperaba obtener en el programa anterior?
Antes de ejecutar el código, esperaba que la consola imprimiera el vector original position como (6, 9), luego se llamara la función playingVector() para cambiar sus valores, y finalmente se imprimiera como (20, 30). También esperaba ver el mensaje "Only once" desde el draw().

### ¿Qué resultado obtuve?
La consola muestra:

```pgsql
p5.Vector Object with properties [x: 6, y: 9]
p5.Vector Object with properties [x: 20, y: 30]
Only once
```
Esto confirma que sí se modificó el vector original dentro de la función playingVector(), y luego se reflejó ese cambio fuera de ella.

### Ejemplo
Paso por valor:
```js
function cambia(n) {
  n = 100;
}
let x = 5;
cambia(x);
console.log(x); // Resultado: 5 (no cambia)
```
Paso por referencia:
```js
function cambiaObjeto(obj) {
  obj.valor = 100;
}
let miObjeto = { valor: 5 };
cambiaObjeto(miObjeto);
console.log(miObjeto.valor); // Resultado: 100 (sí cambia)
```
### ¿Qué tipo de paso se está realizando en el código?
En el código de la actividad se está realizando un paso por referencia. El objeto position (un p5.Vector) se pasa a la función playingVector(), y dentro de esa función se modifican directamente sus propiedades .x y .y. Por eso, el cambio se conserva incluso fuera de la función.

### ¿Qué aprendí?
Aprendí que en JavaScript, los objetos como p5.Vector se pasan por referencia. Eso significa que si modifico sus propiedades dentro de una función, esos cambios se reflejan en el objeto original. Esto es útil pero también requiere precaución, porque puede generar efectos secundarios no deseados si no se controla bien. También reforcé la diferencia entre paso por valor y paso por referencia, y cómo JavaScript maneja cada uno dependiendo del tipo de dato.

