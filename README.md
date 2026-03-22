# Sistema de Registro Universitario

> **Práctica I** — Asignatura ST0244 Lenguajes de Programación · Universidad EAFIT

Sistema para gestionar el **ingreso y salida de estudiantes del campus**, implementado en dos paradigmas distintos: **funcional (Haskell)** y **lógico (Prolog)**.

---

## Especificaciones

| Campo | Detalle |
|---|---|
| **Materia** | ST0244 — Lenguajes de Programación |
| **Docente** | Alexander Narváez Berrío |
| **Valor** | 15% de la nota final |
| **Paradigmas** | Funcional (Haskell) · Lógico (Prolog) |

---

## Haskell — Versión A

### Funcionalidades

| # | Función | Descripción |
|---|---|---|
| 1 | **Check In** | Registro de entrada solicitando ID y almacenando la hora de ingreso |
| 2 | **Search by ID** | Búsqueda de estudiantes actualmente dentro de la universidad |
| 3 | **Time Calculation** | Cálculo automático de la estancia en horas y minutos |
| 4 | **Students List** | Carga de registros desde `University.txt` y visualización en terminal |
| 5 | **Check Out** | Registro de salida y actualización de la persistencia de datos |

### Análisis Técnico

**1. Modelo de Datos e Inmutabilidad**

En Haskell no se modifican variables. Se utiliza una estructura `Estudiante` definida con `data`. Para "actualizar" un registro (como en el Check Out), el programa usa `map` para recorrer la lista y generar una **nueva lista** con el dato cambiado, preservando la integridad de los datos originales.

**2. Gestión de Tiempo**

El tiempo se almacena como un entero que representa los minutos transcurridos desde las `00:00` (Opción 3 de la guía).

```
08:30 AM  →  510 minutos
Permanencia = TiempoSalida - TiempoEntrada
```

**3. Persistencia y Lazy Evaluation**

El programa lee y escribe en `University.txt`. Dado que Haskell usa **evaluación perezosa**, el código fuerza la lectura completa del archivo antes de permitir una nueva escritura, evitando conflictos de acceso.

**4. Recursividad en el Menú**

A falta de ciclos imperativos, el menú principal es una **función recursiva** que se llama a sí misma al finalizar cada operación, manteniendo el programa activo hasta que el usuario decida salir.

### Instrucciones de Ejecución

**Requisitos:** [GHC — Glasgow Haskell Compiler](https://www.haskell.org/ghc/)

1. Coloque `Main.hs` y `University.txt` en la misma carpeta.
2. Abra una terminal en esa carpeta y ejecute:

```bash
runhaskell Main.hs
```

### Formato de Almacenamiento

```haskell
Estudiante {idEst = "123", entrada = 480, salida = -1}
```

> Un valor de `-1` en `salida` indica que el estudiante aún permanece en la universidad.

---

## Prolog — Versión B

### Funcionalidades

| # | Predicado | Descripción |
|---|---|---|
| 1 | `registrar_entrada/0` | Check In: solicita ID y hora de entrada, persiste con `assertz` |
| 2 | `buscar_estudiante/0` | Busca por ID si el estudiante está actualmente dentro (`salida = 0`) |
| 3 | `calcular_tiempo/3` | Calcula la permanencia como `Salida - Entrada` en minutos |
| 4 | `listar_estudiantes/0` | Lista todos los registros en memoria con ID, entrada y salida |
| 5 | `registrar_salida/0` | Check Out: retracta el hecho anterior y aserta uno nuevo con hora de salida |

### Análisis Técnico

**1. Base de Conocimiento Dinámica**

Los estudiantes se representan como hechos dinámicos:

```prolog
:- dynamic estudiante/3. % estudiante(Id, Entrada, Salida)
```

Un valor de `0` en `Salida` indica que el estudiante sigue dentro. Prolog permite agregar y retirar hechos en tiempo de ejecución con `assertz/1` y `retract/1`.

**2. Persistencia**

El sistema carga datos desde `University.txt` al iniciar usando `cargar_datos/0`, que lee los términos con `read/2` y los incorpora a la base de conocimiento. Al registrar entradas o salidas, `guardar_datos/0` reescribe el archivo con `format/3`.

```prolog
estudiante(123, 480, 0).
estudiante(456, 510, 620).
```

**3. Menú con `repeat/0`**

A diferencia de Haskell, el menú usa el predicado `repeat/0` combinado con un corte (`!`) al seleccionar la opción de salida, simulando un ciclo imperativo dentro del paradigma lógico.

**4. Gestión de Tiempo**

Idéntico al módulo Haskell: los tiempos se almacenan en minutos desde `00:00` y la permanencia se calcula con una resta simple.

```prolog
calcular_tiempo(E, S, T) :- T is S - E.
```

### Instrucciones de Ejecución

**Requisitos:** [SWI-Prolog](https://www.swi-prolog.org/)

1. Coloque `registro.pl` y `University.txt` en la misma carpeta.
2. Abra una terminal en esa carpeta y ejecute:

```bash
swipl registro.pl
```

El sistema arranca automáticamente gracias a la directiva:

```prolog
:- initialization(menu).
```

### Formato de Almacenamiento

```prolog
estudiante(123, 480, 0).
estudiante(456, 510, 620).
```

> Un valor de `0` en el tercer argumento indica que el estudiante aún permanece en la universidad.

---
