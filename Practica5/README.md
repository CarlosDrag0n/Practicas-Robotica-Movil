Mi objetivo principal en este ejercicio de Navegación Global era implementar el algoritmo Wave Front (Gradient Path Planning) para que el robot pudiera navegar de forma autónoma a cualquier punto del mapa. Al igual que en ejercicios anteriores, no busqué la solución final de inmediato, sino que fui construyendo el sistema por capas: desde entender las coordenadas hasta refinar el control del movimiento.

Primer código: comprensión del mapa y coordenadas.

Aquí mi prioridad fue entender qué estaba "pisando" el robot y cómo traducir eso a código. Cargué la imagen del mapa y creé un script sencillo para inspeccionar los valores de las celdas.

Esto me sirvió para entender la estructura de la matriz (los valores de las pareeds y los de los caminos) y validar si el objetivo seleccionado por el ratón caía en una zona válida o prohibida. El problema aquí era evidente: el robot sabía dónde estaba el objetivo, pero no tenía ni idea de cómo llegar ni de la distancia real.

Segundo código: conversión de mundos y distancias.

Decidí entonces a conectar el mundo físico (metros en Gazebo) con el mundo digital (píxeles en la matriz). Implementé las fórmulas de conversión de coordenadas y el cálculo de la distancia euclidiana.

Con esto conseguí que el sistema me dijera en tiempo real a cuántos metros estaba el objetivo y en qué celda de la matriz caía. Sin embargo, esta distancia era en línea recta, lo cual no servía para navegar por un laberinto de calles, ya que ignoraba las paredes.

Tercer código: algoritmo Wave Front y visualización.

Aquí di el salto al Path Planning. Implementé el algoritmo de llenado por inundación (BFS), propagando una "ola" desde el objetivo hacia todo el mapa. Para visualizarlo, normalicé los valores de 0 a 255, creando un gradiente donde el negro era la meta y el blanco lo más lejano.

En esta etapa también introduje un concepto clave: el margen de seguridad. Me di cuenta de que si el robot pasaba muy cerca de una pared (valor 0), podía chocar. Por eso, "engordé" las paredes artificialmente en la matriz antes de calcular el gradiente.

Cuarto código: primera navegación básica.

Con el mapa de calor generado, tocaba mover el coche. La lógica fue descender por el gradiente: el robot miraba a sus 8 vecinos y se dirigía al que tuviera el valor más bajo (el camino más corto a la meta).

El robot lograba llegar, pero el movimiento era muy tosco ("robotizado"). Al usar velocidades fijas y giros bruscos, el coche avanzaba a trompicones. Además, si el gradiente no era perfecto, se quedaba atascado en mínimos locales o esquinas.

Quinto código: robustez en la selección del objetivo.

Me encontré con un problema de usabilidad: si hacía clic un píxel dentro de una pared, el algoritmo fallaba o no calculaba nada. Para solucionarlo, implementé una función "imán" (encontrar_camino_cercano).

Si el usuario selecciona una pared, el código busca en espiral el píxel de carretera más cercano y reubica el objetivo allí. Esto hizo el sistema mucho más amigable y robusto ante errores humanos, asegurando que siempre hubiera un camino válido que calcular.

Sexto código: implementación del PID y modo escape.

Para suavizar la conducción, sustituí la lógica básica por un controlador PID completo. Ahora, el error no era solo la dirección, sino que el PID ajustaba la velocidad angular (W) basándose en la diferencia de ángulo hacia el siguiente punto.

Aquí surgió un problema crítico: a veces, por inercia, el coche "pisaba" una pared virtual (valor infinito) y el algoritmo se rompía. Para solucionarlo, creé un "Modo Escape": si el coche detecta que está fuera del camino válido, ignora el gradiente normal y busca desesperadamente el valor numérico más cercano para reincorporarse a la carretera.

Séptimo código: anticipación (Lookahead) y centrado.

Aunque el PID mejoró la suavidad, el robot tendía a "comerse" las curvas, girando demasiado tarde. Decidí mejorar dos aspectos:

Paredes más gruesas: Aumenté el RADIO_SEGURIDAD a 3 píxeles. Esto obligó al algoritmo a generar caminos que van más por el centro de la calle, lejos de los bordes.

Lookahead (Zanahoria): Implementé una función que no mira al vecino inmediato, sino 3 pasos por delante en el gradiente.

Esto permitió que el robot "anticipara" las curvas. En lugar de apuntar al vértice de la esquina, apunta a la salida de la curva, logrando trazas mucho más fluidas y rápidas.

Último código: reestructuración y arquitectura.

Finalmente, con toda la lógica funcionando, noté que el código era difícil de leer y mantener. Así que reestructuré todo el programa para que fuera más legible.

De esta forma, conseguí un sistema que:

Es robusto ante clics erróneos (imán).

Genera trayectorias seguras alejadas de las paredes.

Conduce de forma fluida gracias al PID y al Lookahead.

Se recupera automáticamente si se sale de la pista.

Con cada versión fui resolviendo desde la comprensión del entorno hasta la optimización fina del control, logrando una navegación autónoma fiable y elegante.

Dejo un vídeo a modo de demostración del programa. 


[![Ver video](https://img.youtube.com/vi/fyr2ybg8Zgk/maxresdefault.jpg)](https://youtu.be/fyr2ybg8Zgk)
