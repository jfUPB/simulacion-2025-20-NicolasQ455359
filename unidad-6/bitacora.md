# Evidencias de la unidad 6

## Actividad 01.
<img width="474" height="474" alt="image" src="https://github.com/user-attachments/assets/f14613e5-f3bd-46ba-aaa4-95655ec0684e" />
. Por qué me llama la atención: me gusta cómo esas líneas, aunque son muchas, tienen una dirección orgánica. Hay zonas donde se separan, otras donde se juntas; parecen corrientes visuales que se doblan suavemente. Esa tensión entre orden y variación me parece muy poderosa.

<img width="474" height="474" alt="image" src="https://github.com/user-attachments/assets/e8db6a53-9550-4d96-bb71-da2811b0338c" />
Por qué me llama la atención: aquí se acentúa más la densidad, el contraste y el movimiento. Las líneas parecen viento, polvo, o una fuerza invisible dibujando en el espacio. Esa textura me inspira porque siento que “escucho” el movimiento mientras miro.

## Actividad 02.

### 1. ¿Qué es una fuerza de dirección (steering force)?
Una steering force es básicamente una fuerza que le da dirección a un agente dentro de una simulación. No es solo moverse porque sí, sino que permite que el agente “decida” hacia dónde ir, como si tuviera un objetivo o un comportamiento particular. Es la que hace que un objeto no se mueva en línea recta todo el tiempo, sino que pueda girar, seguir un punto, evitar un obstáculo o alinearse con otros.

### 2. ¿Qué diferencia tiene este tipo de fuerza con las que ya hemos estudiado en el contexto de la simulación de agentes?
La diferencia principal es que las fuerzas que habíamos visto antes son más básicas, como mover un agente aplicando velocidad, aceleración o atracción a un punto fijo. En cambio, la steering force no solo mueve al agente, sino que también le da un sentido de comportamiento. Es decir, en lugar de ser un movimiento “mecánico”, se convierte en algo más parecido a una decisión: seguir a otro, mantenerse dentro de un grupo, esquivar choques, etc.

### 3. ¿Qué relación tiene la steering force con Craig Reynolds y su trabajo en simulación de comportamiento animal?
Craig Reynolds fue quien desarrolló el modelo de los “boids”, que son simulaciones de bandadas de pájaros o bancos de peces. Allí, las steering forces son la base para que cada agente siga reglas simples (como alinearse, separarse o cohesionarse) y, cuando todos los agentes lo hacen, aparece un comportamiento colectivo muy parecido al de los animales reales. En otras palabras, sin las steering forces no se lograría esa sensación natural y emergente que caracteriza el trabajo de Reynolds.

## Actividad 03. 
Actividad 03 — Bitácora
### 1. ¿Cómo está guardado el campo de flujo y cómo se generan sus vectores?
El campo es una cuadrícula. En código es un arreglo indexado por filas y columnas (en p5 se maneja como 1D con index = x + y * cols, pero mentalmente es un 2D).
Cada celda guarda un vector dirección (ángulo + magnitud ≈ 1).
En el ejemplo base del libro, ese vector se genera con Perlin noise: se toma noise(xoff, yoff, zoff), se convierte a ángulo, y de ahí sale el vector unitario de la celda. Así, celdas vecinas apuntan parecido y aparece ese “flujo suave” característico.

### 2. ¿Cómo usa el agente el campo para calcular su steering force?
Mi lectura (versión simple):
Ubico mi celda: con mi posición (pos.x, pos.y) la mapeo a índices de la grilla (col = floor(pos.x / resol), row = floor(pos.y / resol)).
Leo el vector deseado de esa celda (el “viento” local).
Lo convierto en “deseo”: ajusto su magnitud a maxspeed.
Steering = desired – vel: la diferencia entre lo que quiero (desired) y lo que llevo (vel) es mi fuerza de dirección.
Limito esa fuerza con maxforce para que el giro no sea brusco, y la aplico como aceleración.
En corto: el agente “huele” el vector del campo donde está parado y corrige su rumbo suavemente hacia allá.

### 3. Parámetros clave que identifiqué

- Resolución del campo (res, tamaño de cada celda).
- maxspeed del agente (qué tan rápido puede ir).
- maxforce del agente (qué tan fuerte puede corregir el rumbo).

### 4. Modificación que hice (y qué pasó)
Cambio grande 1: reemplacé el noise por un campo en remolino (vórtice)
En vez de usar noise() para el ángulo, generé un vector tangencial alrededor del centro. Eso convierte el flujo en un giro continuo (como corrientes orbitando).
Efecto observado (comportamiento):
Los agentes empezaron a orbitar el centro, dibujando trayectorias curvas y bastante limpias.
El movimiento se vuelve coherente globalmente (todos giran en el mismo sentido), menos errático que con noise.
Si subo maxspeed, las órbitas se alargan y sienten más “energía”; si bajo maxforce, tardan más en alinearse con la curva.








