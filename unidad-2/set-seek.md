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

## Actividad 04. 

### ¿Para qué sirve el método mag()?
El método .mag() me permite obtener la magnitud (o módulo) de un vector, es decir, su longitud desde el origen hasta su posición en el espacio. Lo uso cuando necesito saber qué tan grande es un vector, sin importar su dirección.

### ¿Qué diferencia hay con magSq() y cuál es más eficiente?
El método .magSq() calcula el cuadrado de la magnitud, sin aplicar la raíz cuadrada.
Esto lo hace más eficiente que .mag() porque evita la raíz cuadrada. Uso .magSq() cuando quiero comparar magnitudes entre vectores sin necesidad de saber la magnitud exacta, por ejemplo para saber cuál es mayor

### ¿Para qué sirve el método normalize()?
El método .normalize() sirve para convertir un vector en unitario, que mantenga su dirección pero tenga magnitud igual a 1. Esto es muy útil cuando quiero trabajar con direcciones puras, sin que la longitud afecte el resultado. Lo uso, por ejemplo, para definir la dirección del movimiento o para combinarlo con otra magnitud.

### Si un periodista me pregunta: “¿Para qué sirve el método dot()?”
El método dot() sirve para calcular qué tanto apuntan dos vectores en la misma dirección y es clave para saber si son paralelos, perpendiculares o apuntan en sentido opuesto.

### ¿Cuál es la diferencia entre dot() como método de instancia y como método estático?
Versión de instancia:
Se escribe así: v1.dot(v2)
Se aplica sobre un vector y recibe otro como parámetro. Calcula el producto punto entre v1 y v2.

Versión estática:
Se escribe así: p5.Vector.dot(v1, v2)
No necesito tener un objeto principal, paso directamente los dos vectores como argumentos.

Ambas hacen lo mismo, solo cambia la forma en la que las llamo. La versión estática es útil cuando trabajo con vectores sin instancias previas o en cálculos más generales.

### El periodista: “¿Cuál es la interpretación geométrica del producto cruz?”
El producto cruz de dos vectores en 3D genera un nuevo vector perpendicular a ambos, cuya dirección sigue la regla de la mano derecha y cuya magnitud representa el área del paralelogramo formado por los vectores originales.

* Magnitud: igual al área del paralelogramo que forman los dos vectores.

* Dirección: perpendicular a ambos (normal al plano).

* Orientación: sigue la regla de la mano derecha (si a apunta hacia la derecha y b hacia adelante, a × b apunta hacia arriba).

### ¿Para qué me puede servir el método dist()?
El método .dist() sirve para calcular la distancia entre dos vectores (o puntos en el espacio). Es muy útil, por ejemplo, cuando quiero saber si un objeto está cerca de otro, como detectar colisiones, seguir un objetivo o medir separación entre entidades.

Ejemplo:
```js
let d = p5.Vector.dist(pos1, pos2);
if (d < 50) {
  // los objetos están cerca
}
```

### ¿Para qué sirven los métodos normalize() y limit()?
normalize() lo uso para obtener solo la dirección del vector, manteniendo su forma pero con magnitud igual a 1.

limit(valor) me permite restringir la magnitud máxima del vector. Si el vector es más largo que ese valor, se recorta, pero si es más corto, se deja igual.

Limitan la velocidad máxima de un objeto o controlar la dirección del movimiento sin cambiar su magnitud.

## Actividad 05.
### Código que genera el resultado solicitado
En este ejemplo, modifiqué el código original para hacer múltiples interpolaciones entre dos vectores (v1 y v2) y al mismo tiempo interpolar colores usando lerpColor(). Esto me permitió visualizar cómo evoluciona tanto la dirección como el color entre ambos vectores:

```js
let v1, v2, v3;
let t = 0;
let speed = 0.01;

function setup() {
  createCanvas(400, 400);
  v1 = createVector(50, 50);     // origen
  v2 = createVector(300, 0);     // vector rojo (horizontal)
  v3 = createVector(0, 300);     // vector azul (vertical)
}

function draw() {
  background(200);

  // Vectores rojo y azul desde el mismo origen
  drawArrow(v1, v2, 'red');   // horizontal
  drawArrow(v1, v3, 'blue');  // vertical

  // Flecha verde desde punta del rojo hacia punta del azul
  let puntaRojo = p5.Vector.add(v1, v2);
  let direccionVerde = p5.Vector.sub(v3, v2); // v2 - v1
  drawArrow(puntaRojo, direccionVerde, 'green');

  // Interpolación entre v1 y v2 (desde el origen)
  let interpolado = p5.Vector.lerp(v2, v3, t);
  let colorIntermedio = lerpColor(color(255, 0, 0), color(0, 0, 255), t);
  drawArrow(v1, interpolado, colorIntermedio);

  // Actualizar valor de t
  t += speed;
  if (t > 1 || t < 0) {
    speed *= -1;
  }
}

function drawArrow(base, vec, myColor) {
  push();
  stroke(myColor);
  strokeWeight(3);
  fill(myColor);
  translate(base.x, base.y);
  line(0, 0, vec.x, vec.y);
  rotate(vec.heading());
  let arrowSize = 7;
  translate(vec.mag() - arrowSize, 0);
  triangle(0, arrowSize / 2, 0, -arrowSize / 2, arrowSize, 0);
  pop();
}
```

### ¿Cómo funciona lerp() y lerpColor()?
En este ejercicio estoy usando interpolación lineal para mover una flecha entre dos vectores (v2 y v3), y cambiar su color en el proceso. Estos son los dos métodos clave:

p5.Vector.lerp(v2, v3, t)

Este método genera un nuevo vector que está entre v2 y v3, dependiendo del valor t (entre 0 y 1):

* Si t = 0, el resultado es igual a v2.

* Si t = 1, el resultado es igual a v3.

* Si t = 0.5, el resultado es el punto medio entre ambos.

En mi código:
js
Copiar
Editar
let interpolado = p5.Vector.lerp(v2, v3, t);
Esto crea una flecha dinámica que se mueve suavemente de un vector al otro.

🔹 lerpColor(color(255, 0, 0), color(0, 0, 255), t)
Este método hace lo mismo pero con colores. Interpola el color entre rojo puro y azul puro según el mismo valor t.

En el código:

```js
let colorIntermedio = lerpColor(color(255, 0, 0), color(0, 0, 255), t);
```
Esto me permite hacer que la flecha cambie de rojo a azul conforme se mueve entre v2 y v3. Cuando está en la mitad, el color es morado (mezcla entre ambos).

### ¿Cómo se dibuja una flecha usando drawArrow()?
Este es el método que uso para dibujar cualquier flecha en el canvas. Recibe tres parámetros:

* base: punto de inicio (tipo p5.Vector)

* vec: vector que indica la dirección y magnitud

* myColor: el color de la flecha

```js
function drawArrow(base, vec, myColor) {
  push();                         // Guarda el estado del dibujo
  stroke(myColor);                // Color de línea
  strokeWeight(3);                // Grosor de la línea
  fill(myColor);                  // Color de la punta
  translate(base.x, base.y);     // Mueve el origen a la base del vector
  line(0, 0, vec.x, vec.y);       // Dibuja la línea del vector
  rotate(vec.heading());         // Rota el sistema a la dirección del vector
  let arrowSize = 7;             
  translate(vec.mag() - arrowSize, 0); // Se posiciona al final del vector
  triangle(                      // Dibuja la punta de flecha
    0, arrowSize / 2,
    0, -arrowSize / 2,
    arrowSize, 0
  );
  pop();                          // Restaura el estado del dibujo
}
```
Con translate() y rotate(), las flechas siempre apuntan en la dirección correcta sin importar el vector.














