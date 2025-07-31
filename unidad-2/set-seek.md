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
