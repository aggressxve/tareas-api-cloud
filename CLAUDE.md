# CLAUDE.md

Este archivo tiene las indicaciones para Claude Code (claude.ai/code) cuando trabaje en este repo.

## ¿Qué es este proyecto?

**tareas-api** es un sistema para manejar tareas pendientes en Java 17. Lo hicimos como práctica para la mentoría "Programar con Claude" (Generation México, CH70 Java). Es una lista de pendientes (TODO list) con prioridades, fechas límite y para marcar lo que ya se completó.

## Comandos que vas a usar

```bash
# Correr todas las pruebas
mvn test

# Correr solo una clase de pruebas
mvn test -Dtest=TareaServicioTest

# Correr un solo método de prueba
mvn test -Dtest=TareaServicioTest#completarMarcaLaTarea

# Correr la demo de consola
mvn -q exec:java

# Limpiar y compilar
mvn clean compile
```

## Arquitectura del proyecto

El proyecto tiene tres capas bien definidas:

1. **Capa de Dominio** (`Tarea`, `Prioridad`)
   - `Tarea`: Las propiedades son inmutables (titulo, descripcion, prioridad, fechaLimite) excepto el flag `completada`
   - `Prioridad`: Enum con BAJA, MEDIA, ALTA (se comparan por ordinal)

2. **Capa de Repositorio** (`TareaRepositorio`)
   - Guarda todo en memoria con un `LinkedHashMap<Integer, Tarea>`
   - Los IDs se asignan automáticamente empezando desde 1
   - `buscar(id)` devuelve null si no existe (no avienta excepción)

3. **Capa de Servicio** (`TareaServicio`)
   - Aquí va toda la lógica de negocio
   - Los métodos que necesitan una Tarea avientan `TareaNoEncontradaException` si el ID no existe
   - El método privado `obtener(id)` convierte el null en excepción

## Bugs conocidos y TODOs

El código tiene varios bugs que ya están documentados en los tests y comentarios:

1. **TareaRepositorio:48** - `buscarPorTitulo()` usa `contains()` que distingue mayúsculas/minúsculas, pero el Javadoc dice que no debería importar. El test en TareaRepositorioTest:35 está deshabilitado por esto.

2. **TareaServicio:67** - `diasRestantes()` tiene los parámetros al revés en `ChronoUnit.DAYS.between(fechaLimite, hoy)`. Por eso devuelve valores negativos para fechas futuras (ej: -3 para "en 3 días"). Checa el test en TareaServicioTest:64.

3. **TareaServicio:51** - `listarPorPrioridadMinima()` usa `ordinal() >` en lugar de `ordinal() >=`, entonces excluye las tareas que tienen exactamente la prioridad mínima. Si pides MEDIA, solo regresa ALTA, no MEDIA+ALTA.

4. **Tarea:19** - Comentario TODO: falta validar que el título no venga vacío ni en blanco.

5. **TareaServicioTest:39** - Falta el test: `completar(999)` debería aventar `TareaNoEncontradaException`.

6. **TareaServicioTest:102** - No hay tests para el método `eliminar(int)`.

## Patrones de testing

- Framework: JUnit 5 (5.10.2)
- Las clases de test tienen el mismo nombre que las de producción pero con sufijo `Test`
- Cada test crea instancias frescas (servicio, repositorio) en el `@BeforeEach`
- Usa una constante de fecha fija `HOY = LocalDate.of(2026, 10, 8)` para los tests que dependen del tiempo
- Los tests que no pasan están marcados con `@Disabled` y tienen explicación en el TODO

## Dónde está cada cosa

```
tareas-api/
├── src/main/java/mx/generation/tareas/
│   ├── App.java                       # Demo de consola (punto de entrada para mvn exec:java)
│   ├── Tarea.java                     # Modelo de dominio
│   ├── Prioridad.java                 # Enum: BAJA, MEDIA, ALTA
│   ├── TareaRepositorio.java          # Almacenamiento en memoria
│   ├── TareaServicio.java             # Capa de lógica de negocio
│   └── TareaNoEncontradaException.java
└── src/test/java/mx/generation/tareas/
    ├── TareaRepositorioTest.java
    └── TareaServicioTest.java
```

## Notas sobre la configuración de Maven

- Se necesita Java 17 (`maven.compiler.release=17`)
- El `maven-compiler-plugin` está puesto explícitamente en la versión 3.13.0 para evitar un bug gacho del Maven 3.8 que viene en Ubuntu/Debian (la versión vieja del plugin ignora la propiedad release y compila como Java 5)
- Clase principal: `mx.generation.tareas.App`
