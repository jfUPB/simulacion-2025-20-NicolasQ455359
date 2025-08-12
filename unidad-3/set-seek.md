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





