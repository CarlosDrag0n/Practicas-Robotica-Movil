# Práctica de Localización Visual y Navegación con AprilTags

## Objetivo Principal
El propósito de este ejercicio fue desarrollar un sistema de navegación autónoma capaz de localizarse en un mapa global utilizando una cámara y marcadores **AprilTags** como fuente principal de información.

La solución no se buscó de inmediato, sino que se construyó **por capas**: desde la lectura cruda de píxeles hasta una arquitectura robusta de fusión sensorial que combina visión artificial, odometría y láser para tolerar fallos y ruido en tiempo real.

---

## Evolución del Desarrollo (Roadmap)

El algoritmo final es el resultado de una iteración progresiva a través de múltiples versiones de código, resolviendo problemas específicos en cada etapa:

### 1. Inspección Visual y el Salto al 3D
* **Enfoque inicial:** Comprender qué "ve" el robot.
* **Implementación:** Se comenzó dibujando líneas sobre los tags en 2D. Posteriormente, se integró la matriz intrínseca de la cámara y el algoritmo **PnP (Perspective-n-Point)**.
* **Logro:** Conversión de coordenadas de imagen (píxeles) a coordenadas espaciales 3D relativas, obteniendo la distancia ($Z$) y la orientación del tag respecto al robot.

### 2. Navegación Reactiva y Memoria
* **Problema:** El robot entraba en bucles infinitos al llegar a un objetivo, retroceder y volver a detectarlo.
* **Implementación:** Desarrollo de un controlador **PID** para el encaramiento y la aproximación. Se añadió una **memoria a corto plazo** (`ultimo_id_visitado`) para ignorar tags recién completados y forzar la exploración de nuevas áreas.

### 3. Consciencia del Entorno y Seguridad (Láser)
* **Problema:** La navegación puramente visual era "ciega" a las paredes no marcadas, provocando colisiones.
* **Implementación:** Integración del sensor LiDAR como un "parachoques virtual" bajo una arquitectura de subsunción (el láser tiene prioridad sobre la visión).
* **Detector de Estancamiento:** Se añadió un monitor de progreso (`stuck_timer`). Si el motor empuja pero la distancia visual no cambia, el sistema diagnostica un atasco físico y activa una maniobra de escape ciega.

### 4. El Salto Matemático: Localización Global
* **Desafío:** Ubicar al robot en el mapa absoluto del mundo, no solo relativamente al tag.
* **Implementación:**
    * Carga de configuración del mapa (`YAML`).
    * Desarrollo de la cadena de transformaciones matriciales:
      $$T_{Robot} = T_{Mundo \to Tag} \times T_{Tag \to Cámara} \times T_{Cámara \to Robot}$$
    * **Calibración:** Ajuste manual de la distancia focal (`FOCAL_LENGTH`) y el *offset* del centro del robot para alinear la realidad física con la proyección matemática.

### 5. Fusión Sensorial (Visión + Odometría)
* **Problema:** Al girar y perder de vista los tags, el robot "dejaba de existir" para el sistema de localización.
* **Solución:** Implementación de un sistema híbrido.
    * **Con Tags:** La visión resetea y corrige la posición absoluta (Ground Truth).
    * **Sin Tags:** Entra en juego el **Dead Reckoning** (odometría de ruedas). Se aplicaron factores de corrección (`ODOM_FACTOR = 0.35`) para compensar el deslizamiento de las ruedas en la simulación.

### 6. Inteligencia de Exploración (Máquina de Estados)
* **Implementación:** Diseño de una FSM (*Finite State Machine*) para gestionar la autonomía total:
    * `S_SEARCH`: Búsqueda sistemática de objetivos.
    * `S_APPROACH`: Aproximación fina al tag.
    * `S_SPIN_360`: Barrido completo de recuperación si se pierde el objetivo.
    * `S_RANDOM_DIR`: Desbloqueo probabilístico mediante giros aleatorios si falla la búsqueda sistemática.

### 7. Refinamiento Final: Filtros y Estabilidad
* **Problema:** El ruido inherente a la estimación visual provocaba "saltos" y *jitter* en el mapa.
* **Mejoras:**
    * **Filtro de Salto Máximo:** Descarta lecturas físicamente imposibles (teletransportes).
    * **Suavizado Exponencial:** Aplica un factor (`SMOOTH_FACTOR`) para dar fluidez al movimiento de la pose estimada.
    * **Histéresis (`tag_locked`):** Bloqueo temporal de tags visitados hasta que el robot se aleja una distancia considerable (`REENGAGE_DIST`), eliminando falsos positivos al maniobrar.

---

## Características del Sistema Final

El código resultante (`vigesimo codigo`) ofrece:

1.  **Localización Global Robusta:** Fusión de visión artificial y odometría para una ubicación continua.
2.  **Estabilidad Visual:** Filtrado de ruido para evitar saltos en la estimación de la pose.
3.  **Comportamiento Inteligente:** Alternancia entre búsqueda sistemática y exploración aleatoria para cubrir todo el mapa.
4.  **Auto-Rescate:** Capacidad de detectar si está atascado físicamente (por ejemplo, en una esquina o pata de mesa) y liberarse sin intervención humana.

---

## Dependencias

* `Robotics Academy HAL & WebGUI`
* `OpenCV (cv2)`
* `NumPy`
* `PyAprilTags`
* `PyYAML`

---

## Demo

[Insertar enlace al video aquí]

[![Ver video](https://img.youtube.com/vi/EmlyWCKJOvI/hqdefault.jpg)](https://youtu.be/EmlyWCKJOvI)
