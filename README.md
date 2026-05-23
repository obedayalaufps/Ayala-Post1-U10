# Arquitectura de Computadores - Unidad 10: Post-Contenido 1

## Datos del Estudiante
* **Nombre:** Obed Ayala
* **Institución:** Universidad Francisco de Paula Santander (UFPS)
* **Programa:** Ingeniería de Sistemas
* **Año:** 2026 

## Objetivo de la Actividad
El objetivo primordial de este laboratorio es implementar y ejecutar un benchmark en lenguaje C para medir de forma empírica la latencia de acceso a la memoria utilizando arrays de diversos tamaños. A través de este experimento, se analizan los efectos prácticos de la jerarquía de memoria caché (L1, L2, L3 y RAM) sobre el rendimiento de los programas, interpretando las variaciones en función de la localidad espacial, el tamaño de las líneas de caché y los fallos de caché (*cache misses*).

## Estructura del Repositorio
De acuerdo con las especificaciones de la guía de entregables, el repositorio se estructura de la siguiente manera:
```text
Ayala-Post1-U10/
[cite_start]├── capturas/                     # Capturas de pantalla de las evidencias en la terminal 
[cite_start]│   ├── checkpoint1_topology.png   # Evidencia de la topología de cachés del sistema 
[cite_start]│   ├── checkpoint2_sequential.png # Evidencia de la tabla de resultados secuenciales 
[cite_start]│   └── checkpoint3_comparative.png# Evidencia de la comparativa secuencial vs. aleatoria
[cite_start]├── cache_bench.c                 # Código fuente del benchmark en C 
[cite_start]└── README.md                     # Documentación técnica formal del laboratorio 

```

## Prerrequisitos del Entorno

Los experimentos fueron desarrollados bajo el siguiente entorno tecnológico:

* **Sistema Operativo:** Linux (Subsistema de Windows para Linux - WSL / Ubuntu).


* **Compilador:** GCC con soporte de banderas de optimización.


* **Directorio de Trabajo:** `~/u10post1/`.



---

## Análisis Técnico de los Checkpoints

### Checkpoint 1: Configuración del Entorno y Detección de Caché

Se interactuó directamente con la interfaz del sistema de archivos del kernel de Linux mediante la ruta `/sys/devices/system/cpu/cpu0/cache/` para interrogar la topología física del procesador de la máquina.

La ejecución del script en bucle arrojó los tamaños de almacenamiento de las memorias caché internas del hardware. Estos valores representan los puntos de inflexión físicos donde el tamaño de las estructuras de datos desborda la capacidad de un nivel de memoria, forzando la búsqueda en el siguiente nivel jerárquico.

*Ver evidencia gráfica en:* `capturas/Checkpoint1.png` 

### Checkpoint 2: Benchmark de Latencia (Acceso Secuencial)

Se ejecutó la función `bench_seq` evaluando tamaños de arrays desde los 4 KB hasta los 64 MB. El programa fue compilado de forma obligatoria con la bandera de optimización deshabilitada (`-O0`) para evitar que el compilador eliminara los lazos de lectura cíclicos al detectar que el dato no se modificaba en el código.

* **Análisis de Resultados:** En la tabla de salida se observan saltos de latencia discernibles en nanosegundos por byte (`ns/byte`). Mientras el array se mantiene por debajo de los límites de la caché L1, la latencia es mínima gracias a la velocidad de la memoria SRAM estática. Al superar sucesivamente los tamaños críticos de L1, L2 y L3, el tiempo de acceso se incrementa escalonadamente debido a la penalización por la búsqueda de datos en niveles más lentos y densos.



*Ver evidencia gráfica en:* `capturas/Checkpoint2.png` 

### Checkpoint 3: Acceso Aleatorio vs. Secuencial

Se incorporó la rutina `bench_rand`, la cual implementa el algoritmo de mezcla de Fisher-Yates para desordenar por completo el vector de índices de acceso a la memoria.

* **Análisis de Resultados:** La comparación directa evidencia que el acceso aleatorio destruye el principio de **localidad espacial**, impidiendo que los mecanismos de *hardware prefetching* carguen líneas de caché continuas de forma anticipada. Como consecuencia, la probabilidad de un *cache miss* se eleva drásticamente de manera independiente al tamaño del array.


* **Impacto del TLB:** Adicionalmente, cuando el tamaño del bloque supera el umbral del búfer de traducción rápida (TLB, aproximadamente ~1-2 MB), el acceso aleatorio provoca fallos de traducción de direcciones (*TLB misses*), obligando a la MMU a realizar recorridos de tablas en la memoria principal y disparando exponencialmente la latencia medida frente al acceso secuencial ordinario.



*Ver evidencia gráfica en:* `capturas/Checkpoint3.png` 

---

## Conclusiones e Implicaciones de Diseño

1. **La Trilogía de Memoria:** Los datos obtenidos demuestran físicamente el *trade-off* de diseño arquitectónico: las memorias de alta velocidad poseen capacidades sumamente limitadas debido a su área en silicio, justificando la existencia de una jerarquía multinivel.


2. **Impacto del Patrón de Acceso:** Escribir código consciente de la caché (*cache-aware*) determina el rendimiento real de una aplicación. El acceso secuencial aprovecha las líneas de caché completas cargadas en ráfagas, mientras que los patrones de acceso desordenados degradan el rendimiento al saturar los buffers con constantes fallos de traducción y vaciados de líneas.
