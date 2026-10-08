# Mapa del Proyecto tareas-api

## 1. ¿Qué hace el proyecto?

Sistema de gestión de tareas pendientes (TODO list) en Java 17 con persistencia en memoria. Permite crear tareas con prioridad y fecha límite, marcarlas como completadas, filtrarlas por estado o prioridad, y generar reportes de progreso.

## 2. Tabla de clases

| Clase | Responsabilidad | Depende de |
|-------|----------------|------------|
| `App` | Demo de consola que crea tareas de ejemplo y muestra el reporte | `TareaServicio`, `TareaRepositorio`, `Prioridad` |
| `Tarea` | Modelo de dominio que representa una tarea con título, descripción, prioridad, fecha límite y estado de completitud | `Prioridad` |
| `Prioridad` | Enum que define los tres niveles de prioridad (BAJA, MEDIA, ALTA) | ninguna |
| `TareaRepositorio` | Almacenamiento en memoria usando LinkedHashMap, asigna IDs autoincrementales | `Tarea` |
| `TareaServicio` | Lógica de negocio: crear, completar, eliminar, listar, filtrar tareas y generar reportes | `TareaRepositorio`, `Tarea`, `Prioridad`, `TareaNoEncontradaException` |
| `TareaNoEncontradaException` | Excepción lanzada cuando se busca una tarea por ID que no existe | ninguna |
| `TareaRepositorioTest` | Tests unitarios para el repositorio | `TareaRepositorio`, `Tarea`, `Prioridad` |
| `TareaServicioTest` | Tests unitarios para la capa de servicio | `TareaServicio`, `TareaRepositorio`, `Tarea`, `Prioridad` |

## 3. Cómo se compila y se prueba

```bash
# Limpiar y compilar el proyecto
mvn clean compile

# Correr todas las pruebas
mvn test

# Correr solo una clase de pruebas
mvn test -Dtest=TareaServicioTest

# Correr un solo método de prueba
mvn test -Dtest=TareaServicioTest#completarMarcaLaTarea

# Ejecutar la demo de consola
mvn -q exec:java
```

## 4. Tres cosas sospechosas o incompletas

### 1. Posible NullPointerException en completar()
**Archivo:** `TareaServicio.java:24-26`  
El método `completar(int id)` llama a `repositorio.buscar(id)` que puede devolver `null`, pero luego llama directamente a `tarea.marcarCompletada()` sin validar. Esto lanzaría `NullPointerException` en lugar de la esperada `TareaNoEncontradaException`. El método privado `obtener(int id)` en la línea 125 sí maneja esto correctamente, pero no se usa aquí.

### 2. Búsqueda por título distingue mayúsculas cuando no debería
**Archivo:** `TareaRepositorio.java:44`  
El método `buscarPorTitulo()` usa `contains()` que es case-sensitive, pero el Javadoc en las líneas 37-39 promete que "sin distinguir mayúsculas de minúsculas". Por ejemplo, buscar "informe" no encontrará "INFORME". Hay un test deshabilitado (`@Disabled`) en `TareaRepositorioTest.java:35` que documenta este bug.

### 3. Generación de reporte usa concatenación ineficiente
**Archivo:** `TareaServicio.java:74-122`  
El método `generarReporte()` concatena strings usando el operador `+` repetidamente (líneas 74-122), creando múltiples objetos String intermedios. Para un reporte con muchas tareas, esto es ineficiente. Debería usar `StringBuilder` para mejor rendimiento y claridad.
