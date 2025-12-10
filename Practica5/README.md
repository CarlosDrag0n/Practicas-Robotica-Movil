Mi objetivo principal en este ejercicio de Mapeado y Navegación Reactiva era lograr que el robot explorara un entorno desconocido de forma autónoma, construyendo un mapa fiable en tiempo real. No he buscado la solución final de inmediato, sino que fui construyendo el sistema por capas: desde la lectura cruda del sensor hasta una arquitectura robusta de toma de decisiones capaz de escapar de bloqueos.

Primer código: inspección del sensor láser. Aquí mi prioridad fue entender qué estaba "viendo" el robot. Antes de intentar moverme, necesitaba saber cómo me devolvía la información el sensor LiDAR. Creé un código para destripar el objeto data y entender su estructura (values, ranges). Esto me sirvió para confirmar que el sensor devolvía distancias en metros y para calibrar cómo filtrar los valores inf (infinitos) o nan (errores) que podían romper los cálculos matemáticos posteriores.

Segundo código: navegación sistemática reactiva. Decidí entonces dar los primeros pasos. Implementé una máquina de estados muy sencilla (FORWARD, TURN) basada en una lógica puramente reactiva: "si ves algo cerca, gira; si no, avanza". El robot se movía, pero era una navegación "ciega". No sabía dónde estaba ni a dónde iba, simplemente rebotaba por el entorno evitando colisiones inmediatas. Además, el giro era arbitrario, lo que a menudo lo dejaba atrapado en bucles simples.

Tercer código: mapeado básico y localización. Necesitaba que el robot recordara dónde había estado. Implementé las fórmulas de conversión de coordenadas polares (del láser) a cartesianas (del mundo) y de ahí a la matriz del mapa (píxeles). Con esto conseguí pintar en tiempo real los obstáculos detectados sobre un lienzo negro. Sin embargo, el sistema era computacionalmente pesado y el robot se movía muy despacio, ya que procesaba cada rayo del láser en cada iteración.

Cuarto código: optimización y limpieza visual. Me encontré con un problema de rendimiento. Para solucionarlo, optimicé el bucle de lectura del láser, procesando solo uno de cada tres rayos ("skip steps"). También invertí la lógica visual: partí de un lienzo blanco (vacío) para pintar los obstáculos en negro, lo cual era más intuitivo. El robot ahora mapeaba más rápido, pero la navegación seguía siendo tosca: paraba en seco y giraba sobre su eje de forma muy mecánica.

Quinto código: lógica probabilística (Bayes). Introduje un concepto clave: la incertidumbre. Un solo rayo láser puede ser ruido, así que implementé un modelo probabilístico simplificado (Rejilla de Ocupación). En lugar de marcar un obstáculo como "verdad absoluta" (0 o 255), usé una matriz de flotantes (prob_map) inicializada en 0.5 (desconocido). Cada vez que el láser detectaba algo, aumentaba la probabilidad de ocupación (+0.3). Esto permitió generar un mapa mucho más limpio y fiable, filtrando lecturas erróneas esporádicas.

Sexto código: cerebro explorador y "Ray Casting". Con el mapa probabilístico funcionando, el problema era la curiosidad: el robot no sabía a dónde ir. Implementé una función cerebro_decidir_giro que contaba la cantidad de píxeles "desconocidos" (gris) a su izquierda y derecha. Además, mejoré el mapeado usando el algoritmo de Bresenham (método eficiente para dibujar líneas rectas en gráficos de computadora, la clave es que usa solo operaciones enteras, evitando cálculos con decimales o flotantes, lo que lo hace rápido y perfecto para gráficos en tiempo real). Ahora, no solo pintaba el obstáculo final, sino que "limpiaba" (bajaba la probabilidad) de todas las celdas entre el robot y el obstáculo, dibujando pasillos libres. El robot empezaba a comportarse de forma inteligente, buscando zonas no exploradas.

Séptimo código: navegación dirigida por fronteras. Para suavizar la conducción, sustituí los giros bruscos de 90º por un buscador de ángulos (buscar_mejor_angulo). El robot ahora escanea un abanico de posibilidades frente a él y elige el ángulo que le dirige hacia la mayor concentración de terreno desconocido. Como se ve en la transcripción de la terminal ("Ganador: -44.59º"), el robot ya no reaccionaba al azar, sino que calculaba activamente un target_yaw preciso. Esto permitió trazas mucho más fluidas y una exploración más eficiente.

Octavo código: sistema de puntuación y memoria. Aunque navegaba bien, el robot tendía a volver a zonas que ya había visitado. Para solucionarlo, implementé una memoria de celdas visitadas (visited) con un factor de decaimiento. Creé un sistema de scoring complejo para elegir la ruta: sumaba puntos por ir a lo desconocido, pero restaba muchos puntos si la zona ya había sido visitada o si el camino estaba bloqueado en el mapa probabilístico. Esto obligó al robot a priorizar siempre áreas nuevas y evitar quedarse dando vueltas en círculos en habitaciones ya mapeadas.

Noveno código: robustez extrema y modos de escape. Finalmente, noté que en situaciones complejas (como esquinas afiladas o "trampas" en U), el robot detectaba un obstáculo pero intentaba corregir su trayectoria con cambios de ángulo tan pequeños que no lograba salir (deriva). Para blindar el sistema, añadí detectores de estancamiento (is_angle_drift_stuck) y un estado de emergencia: STATE_ESCAPE_ROTATE. Si el robot detecta que no avanza o que sus objetivos oscilan sin progreso, deja de "pensar" y entra en un modo reactivo puro: gira sobre sí mismo buscando el hueco libre más grande en el láser para salir del atolladero a toda costa.

De esta forma, conseguí un sistema que:

Genera mapas probabilísticos limpios usando Bayes y Bresenham.

Prioriza la exploración de zonas desconocidas de forma inteligente.

Evita bucles infinitos gracias a su memoria de zonas visitadas.

Posee una "consciencia" de fallo que le permite auto-rescatarse si se queda atascado.

Con cada versión fui resolviendo desde la comprensión del sensor hasta la autonomía total en la toma de decisiones, logrando un explorador robusto y fiable.


Dejo un video con la simulación:


[![Ver video](https://img.youtube.com/vi/U9Y1F1qcj1E/hqdefault.jpg)](https://youtu.be/U9Y1F1qcj1E)

