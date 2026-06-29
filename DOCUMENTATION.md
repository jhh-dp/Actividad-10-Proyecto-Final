# Documentación Técnica: Simulador Interactivo de Laberintos y Algoritmos de Búsqueda

## 1. Descripción General del Proyecto

### ¿Qué es el proyecto?
Este proyecto consiste en un avanzado simulador interactivo de laberintos desarrollado íntegramente en Python. Cuenta con una interfaz gráfica (GUI) robusta capaz de generar mapas dinámicos y laberínticos, para posteriormente resolverlos en tiempo real visualizando el comportamiento de diversos algoritmos de búsqueda.

### ¿Por qué se hace? (Justificación)
El propósito principal de este sistema posee un alto valor académico y práctico. Permite a estudiantes, desarrolladores y analistas visualizar, auditar y comparar el rendimiento de múltiples algoritmos de búsqueda en grafos operando sobre un mismo espacio de estados idéntico, el cual posee múltiples soluciones viables. Esto facilita la comprensión de cómo diferentes heurísticas y lógicas de recorrido impactan directamente en el tiempo de ejecución y la eficiencia espacial al intentar hallar la ruta más corta en entornos controlados y altamente interconectados.

### ¿Cómo se hace? (Enfoque Tecnológico)
El desarrollo se sustenta exclusivamente en **Python puro**, aplicando de manera estricta el paradigma de **Programación Orientada a Objetos (POO)**. Se garantiza un desacoplamiento total entre el motor de procesamiento lógico (Backend algorítmico) y la renderización visual (Frontend) apoyándose en el patrón Modelo-Vista-Controlador (MVC). Además, integra librerías para la generación de gráficas estadísticas de rendimiento de forma nativa (como `matplotlib`), proporcionando un entorno de análisis visual fluido e intuitivo.

---

## 2. Arquitectura del Sistema y Árbol de Directorios

El proyecto adopta un diseño arquitectónico modular basado en un enfoque MVC adaptado a las necesidades específicas de la simulación gráfica. Cada componente tiene responsabilidades claramente definidas, favoreciendo la escalabilidad y el mantenimiento del código.

```text
maze_simulator/
│
├── main.py                 # Punto de entrada de la aplicación
├── config.py               # Configuraciones globales (colores, tamaños, Big O estáticos)
│
├── models/                 # Lógica de datos y estructuras (Modelo)
│   ├── __init__.py
│   ├── maze.py             # Clase Maze (Generación matricial, control de celdas, estado del grafo)
│   └── agent.py            # Clase Agent (Punto de recorrido, coordenadas espaciales, assets PNG)
│
├── core/                   # El "Backend": Algoritmos de resolución (Controlador Lógico)
│   ├── __init__.py
│   ├── solver_base.py      # Clase abstracta/base (MazeSolver) estandarizando estrategias
│   ├── bfs.py              # Implementación de búsqueda en Anchura (Breadth-First Search)
│   ├── dfs.py              # Implementación de búsqueda en Profundidad (Depth-First Search)
│   └── a_star.py           # Implementación del algoritmo heurístico A* (A-Estrella)
│
└── views/                  # El "Frontend": Interfaz gráfica y gráficos (Vista)
    ├── __init__.py
    ├── main_window.py      # Contenedor principal de la interfaz y layouts
    ├── menu_view.py        # Selector inicial de dimensiones (Pequeño, Mediano, Grande)
    ├── canvas_view.py      # Renderizado visual del laberinto (Soporte de Paneo y Zoom)
    └── dashboard_view.py   # Panel de métricas con Matplotlib (Gráficas Tiempo vs Eficiencia)
```

### Descripción de los Componentes:
- **`main.py`**: Archivo orquestador que inicializa la aplicación y vincula la lógica con la interfaz gráfica.
- **`config.py`**: Centraliza todas las constantes del sistema (paletas de color, resoluciones de pantalla, y diccionarios con metadatos de complejidad computacional).
- **`models/`**: Contiene la definición estructural. `Maze` encapsula el estado matricial y las reglas físicas del laberinto. `Agent` representa a la entidad que transita por el entorno.
- **`core/`**: Representa el cerebro algorítmico del simulador. La clase abstracta `MazeSolver` dicta el contrato que todo algoritmo debe cumplir, permitiendo inyectar dependencias con facilidad (`bfs.py`, `dfs.py`, `a_star.py`).
- **`views/`**: Segmenta la carga de la interfaz gráfica. Delimita la navegación en menús (`menu_view.py`), la visualización interactiva del grid (`canvas_view.py`), y la telemetría del análisis (`dashboard_view.py`).

---

## 3. Lógica de Generación del Laberinto (Múltiples Caminos)

Para garantizar un entorno desafiante donde la elección del algoritmo sea significativa, la generación del entorno consta de dos etapas secuenciales fundamentales matemáticamente diseñadas para proveer múltiples rutas válidas:

### Paso 1: Generación del Laberinto Perfecto (Árbol de Expansión)
La estructura base inicial se genera empleando el algoritmo **DFS Aleatorizado (Randomized Depth-First Search) implementado con Backtracking**. Este enfoque explora celdas pseudoaleatorias derribando las paredes que las separan. Al no permitir la re-visitación de celdas consolidadas y aplicar retroceso cuando se llega a un callejón sin salida (Backtracking), se asegura la formación de un "laberinto perfecto", es decir, un grafo acíclico donde existe exactamente un único camino válido entre dos nodos cualesquiera.

### Paso 2: Inyección de Complejidad (Ruptura de Paredes)
Un laberinto de camino único limita la comparativa entre algoritmos óptimos y subóptimos. Para solventarlo, se implementa un mecanismo probabilístico de *Ruptura de Paredes*. Tras el Paso 1, el algoritmo recorre la matriz de paredes internas restantes y, basado en un índice de permeabilidad configurable (ej. 20% a 30%), elimina bloqueos adicionales.
Esta degradación controlada introduce ciclos (bucles) y múltiples caminos concurrentes dentro del grid, convirtiendo el espacio en un laberinto imperfecto. Ahora, identificar la ruta más corta pasa a ser un problema computacionalmente rico.

### Dimensiones Adaptativas
El sistema permite configurar el área transitable según tres perfiles escalares referenciados al factor de ocupación de la pantalla del usuario:
- **Pequeño:** ~40% del área útil. Generación casi instantánea, ideal para pruebas de concepto.
- **Mediano:** ~70% del área útil. Equilibrio óptimo entre densidad y tiempos de visualización.
- **Grande:** ~100% del área útil. Prueba de estrés máxima, evidenciando las limitaciones en tiempo y espacio de algoritmos no informados.

---

## 4. Estrategias Algorítmicas y Análisis de Complejidad (Big O)

El punto inicial (start) se definirá en la esquina inferior izquierda del laberinto, y el punto objetivo (goal) en la esquina superior derecha. Se emplean y auditan tres estrategias algorítmicas clásicas:

### 1. BFS (Breadth-First Search - Búsqueda en Anchura)
Realiza una exploración por capas perimetrales, avanzando un nivel de profundidad a la vez hacia todas las ramificaciones disponibles utilizando una Cola (Queue) estándar (FIFO). En un entorno donde los desplazamientos (aristas) tienen el mismo peso (distancia = 1), BFS garantiza invariablemente el descubrimiento del camino más corto. Sin embargo, su principal debilidad radica en su alta demanda de memoria espacial para albergar extensas fronteras de expansión en mapas abiertos.

### 2. A* (A-Estrella - Búsqueda Heurística)
Es una optimización robusta que utiliza una cola de prioridad basada en el costo total estimado de una ruta. Combina el costo exacto transcurrido desde el inicio ($g(n)$) junto con una heurística de proximidad pre-calculada hacia el destino ($h(n)$). En grids que restringen el movimiento diagonal, A* utiliza la **Distancia Manhattan** como función heurística, guiando de forma determinista la expansión hacia el objetivo y logrando descartar enormes secciones de la matriz que resultarían subóptimas. 

### 3. DFS (Depth-First Search - Búsqueda en Profundidad)
Avanza explorando un camino específico hasta alcanzar su límite máximo de profundidad (callejón sin salida) antes de aplicar retroceso iterativo a través de una Pila (LIFO). Este algoritmo se emplea en la simulación principalmente para evidenciar la naturaleza de una búsqueda "ciega y sin garantía de optimalidad". DFS encuentra *un* camino, pero rara vez resultará ser el camino más corto, exhibiendo trazos serpenteantes ineficientes, útil como estándar base (baseline) de comparación negativa frente a enfoques superiores.

### Tabla Comparativa de Algoritmos (Notación Big O)

| Algoritmo | Complejidad Temporal (Peor Caso) | Complejidad Espacial (Peor Caso) | Garantía de Camino Más Corto | Tipo de Búsqueda |
| :--- | :--- | :--- | :--- | :--- |
| **BFS** | $O(V + E)$ | $O(V)$ | **Sí** | No Informada (Ciega) |
| **A\*** | $O(E \log V)$ | $O(V)$ | **Sí** | Informada (Heurística) |
| **DFS** | $O(V + E)$ | $O(V)$ | **No** | No Informada (Ciega) |

*(Donde **$V$** representa los Vértices/Celdas navegables y **$E$** representa las Aristas/Conexiones válidas del laberinto).*

---

## 5. Diseño de la Interfaz Gráfica (UI) y Experiencia de Usuario

La interfaz está orientada a mantener a los usuarios enfocados en la representación visual y las métricas computacionales:

### Ventana de Inicio (Menu)
Proporciona un punto de entrada limpio y minimalista. Está conformada por botones de acción principales que configuran el entorno base solicitando al usuario el tamaño deseado: "Laberinto Pequeño", "Laberinto Mediano" o "Laberinto Grande".

### Ventana Principal (Main Workspace)
Es el ecosistema primario de trabajo donde interactúa la simulación en sí misma:

- **Panel Superior (Controles de Ejecución):**
  Alberga las herramientas dinámicas del usuario. Integra botones con etiquetas claras para invocar la resolución: "Ejecutar BFS", "Ejecutar A*" o "Ejecutar DFS". También contendrá un control para restablecer el mapa sin perder el laberinto actual, posibilitando pruebas algorítmicas idénticas en cadena.

- **Lienzo del Laberinto (Canvas Interactivo):**
  Constituye el entorno de renderizado matricial. Empleando técnicas de trazado optimizado, representa las paredes, espacios vacíos y la ruta descubierta en tiempo real (mediante colores o texturas específicas). Para laberintos de dimensiones masivas, la clase incorpora **funciones matemáticas interactivas avanzadas de navegación visual**:
  - **Zoom Dinámico:** Integrado mediante transformaciones afines. El usuario puede emplear la rueda del ratón (`Mouse Wheel`) para escalar la escena hacia dentro o hacia fuera, enfocando sectores específicos de celdas.
  - **Desplazamiento / Paneo:** Al presionar y arrastrar usando el botón derecho o central del ratón, se calcula el vector diferencial de movimiento actualizando el origen relativo (offset) del canvas en el modelo de vista.

- **Panel de Métricas (Dashboard en Tiempo Real):**
  Un componente analítico que incorpora la potencia del dibujo gráfico de `matplotlib` incrustado en el ecosistema. Mostrará, tras la ejecución de cada algoritmo, un conjunto comparativo de gráficos de barras para evaluar visual y empíricamente:
  - *Tiempo de Ejecución:* El volumen de milisegundos reales consumidos por la función de búsqueda.
  - *Eficiencia de Espacio de Estado:* Nodos Explorados frente al tamaño en nodos del Camino Final, demostrando de forma directa el beneficio del poda espacial inherente al algoritmo A*.

---

## 6. Casos de Prueba Iniciales y Plan de Desarrollo

El sistema será validado empíricamente a lo largo de su construcción garantizando su integridad mecánica a través de escenarios de estrés.

### Escenarios de Prueba Lógicos
1. **Laberinto Bloqueado Sin Salida Válida:** Forzar un bloqueo estático integral rodeando al Agente o a la Meta para verificar que los algoritmos abortan la búsqueda de forma segura sin generar excepciones de desbordamiento (Overflow), retornando un estado "No Resuelto".
2. **Laberinto de Un Único Camino (Árbol Acíclico Puro):** Ejecutar una búsqueda en el mapa posterior al Paso 1 de generación pero previo a la ruptura de paredes. En este estado, BFS, DFS y A* deben reportar *exactamente el mismo camino resultante* y los mismos nodos dentro de este.
3. **Laberinto Masivo Altamente Interconectado:** Maximizar el parámetro de ruptura de paredes en el mapa "Grande" y observar cómo la búsqueda de profundidad de DFS falla en eficiencia al recorrer zonas inútiles enteras, y evidenciar el delta de tiempo sustancial de la cola de prioridad de A*.

### Cronograma de Desarrollo Modular (Por Fases)
El proyecto se plantea ejecutar mediante un flujo ágil progresivo:

- **Fase 1 (Modelos y Generación):**
  - Implementación de las clases lógicas `Maze` y `Agent`.
  - Desarrollo del generador DFS Aleatorio base y el sistema de inyección de complejidad (Ruptura de paredes).
- **Fase 2 (Core Algorítmico):**
  - Consolidación del contrato genérico `solver_base.py`.
  - Construcción estricta de las tres unidades de resolución independientes: `bfs.py`, `dfs.py` y `a_star.py`, preparadas para acoplarse y devolver secuencias de coordenadas trazables.
- **Fase 3 (UI e Interactividad):**
  - Implementación de la Vista Inicial (`menu_view.py`) y la Vista Principal (`main_window.py`).
  - Creación del `canvas_view.py`, añadiendo el cálculo diferencial necesario para soportar el zoom in/out interactivo y el desplazamiento espacial (paneo).
- **Fase 4 (Dashboard e Integración Final):**
  - Acoplamiento del motor algorítmico (Core) con el disparo por eventos del usuario (Views).
  - Implementación del canvas incrustado de matplotlib en el `dashboard_view.py`.
  - Validación con los casos de prueba teóricos, auditoría de código, pulido estético final y documentación.

---
*Fin del Documento Técnico.*
