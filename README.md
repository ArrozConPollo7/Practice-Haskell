# Sistema de Registro Universitario

> **Práctica I** — Asignatura ST0244 Lenguajes de Programación · Universidad EAFIT
>
> **Integrantes:** Juan David Giraldo Regino · Juan José Duque Marín

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
| 1 | **Check In** | Verifica que el estudiante no esté ya dentro, registra ID y hora de entrada en formato `HH:MM` |
| 2 | **Search by Student ID** | Muestra ficha del estudiante si está actualmente en campus (`salida = -1`) |
| 3 | **Time Calculation** | Calcula y muestra el tiempo de permanencia de un estudiante que ya realizó Check Out |
| 4 | **List Students** | Recarga `University.txt` desde disco en el momento de elegir la opción, almacena en lista y la muestra |
| 5 | **Check Out** | Registra la hora de salida, calcula la duración y guarda en archivo |

### Análisis Técnico

**1. Modelo de Datos e Inmutabilidad**

Los estudiantes se representan con una estructura `Estudiante` definida con `data`. Ninguna función modifica la lista existente — cada operación genera una nueva lista que se pasa como argumento a la siguiente llamada recursiva del menú.

**2. Gestión del Tiempo (Opción 2)**

La hora se ingresa en formato `HH:MM`, validada por `parsearHora`, y se almacena internamente como minutos desde `00:00`. La permanencia se calcula con una resta simple:

```
08:30  →  510 minutos
Permanencia = TiempoSalida - TiempoEntrada
```

**3. Opción 4 — Carga desde archivo en tiempo real**

`studentsList` llama a `cargarArchivo` en el momento en que el usuario elige esta opción, cumpliendo el requerimiento de la guía: *load → store in list → display*. La lista retornada reemplaza la que estaba en memoria.

**4. Persistencia**

`University.txt` se carga al iniciar `main`. En cada Check In o Check Out, `guardarArchivo` reescribe el archivo completo con el estado actualizado. Si el archivo no existe al arrancar, se crea automáticamente vacío.

**5. Menú con Recursividad Paramétrica**

El menú recibe la lista actual como argumento (`menu :: [Estudiante] -> IO ()`) y se llama recursivamente con la lista actualizada. El "estado" del programa vive en el parámetro, sin variables mutables.

```haskell
"1" -> checkIn lista      >>= menu
"4" -> studentsList       >>= menu   -- recarga desde disco
"5" -> checkOut lista     >>= menu
```

### Instrucciones de Ejecución

**Requisitos:** [GHC — Glasgow Haskell Compiler](https://www.haskell.org/ghc/)

1. Coloque `Main.hs` en su carpeta de trabajo.
2. Abra una terminal en esa carpeta y ejecute:

```bash
runhaskell Main.hs
```

> Si `University.txt` no existe, el programa lo crea automáticamente al iniciar.

### Formato de Almacenamiento

```haskell
Estudiante {idEst = "12345678", entrada = 510, salida = -1}
Estudiante {idEst = "87654321", entrada = 555, salida = 740}
```

> Un valor de `-1` en `salida` indica que el estudiante aún permanece en la universidad.

---

## Prolog — Versión B

### Funcionalidades

| # | Predicado | Descripción |
|---|---|---|
| 1 | `registrar_entrada/0` | Valida que el ID sea entero y que no esté ya dentro, pide hora en `HH:MM` y persiste con `assertz` |
| 2 | `buscar_estudiante/0` | Si está dentro muestra hora de entrada; si ya salió muestra entrada, salida y duración |
| 3 | `calcular_tiempo/3` | Calcula la permanencia como `Salida - Entrada` en minutos |
| 4 | `listar_estudiantes/0` | Recarga `University.txt` desde disco (`cargar_datos`) y muestra todos los registros |
| 5 | `registrar_salida/0` | Valida que la hora de salida sea posterior a la entrada, actualiza con `retract` + `assertz` |

### Análisis Técnico

**1. Base de Conocimiento Dinámica**

Los estudiantes se representan como hechos dinámicos. Un valor de `0` en el tercer argumento indica que el estudiante sigue dentro:

```prolog
:- dynamic estudiante/3. % estudiante(Id, Entrada, Salida) — 0 si aún está dentro
```

**2. Gestión del Tiempo (Opción 2 — consistente con Haskell)**

Igual que en Haskell, la hora se ingresa en formato `HH:MM` entre comillas simples y se almacena como minutos desde `00:00`. `parsear_hora/2` y `minutos_a_hora/2` hacen la conversión:

```prolog
parsear_hora('08:30', 510).   % H*60 + M
minutos_a_hora(510, '08:30').
```

**3. Opción 4 — Recarga desde archivo**

`listar_estudiantes/0` llama a `cargar_datos` antes de mostrar, recargando `University.txt` en ese momento — cumple el mismo requerimiento que en Haskell: *load → store → display*.

**4. Persistencia**

Al arrancar, `menu/0` llama `cargar_datos/0` que limpia la base con `retractall` y repobla desde el archivo. En cada Check In o Check Out, `guardar_datos/0` reescribe el archivo con `format/3`.

**5. Menú con Recursividad (`menu_loop`)**

En lugar del `repeat/0` con corte, el menú usa un predicado recursivo `menu_loop/0`. Al finalizar cada operación, el predicado `ejecutar/1` llama a `menu_loop` nuevamente, excepto en la opción 5:

```prolog
ejecutar(1) :- registrar_entrada, menu_loop.
ejecutar(5) :- writeln('Saliendo del sistema. Hasta luego.').
ejecutar(_) :- writeln('Opcion no valida.'), menu_loop.
```

**6. Validaciones**

- Check In: verifica que el ID sea entero y que no haya un registro activo (`salida = 0`) para ese ID.
- Check Out: si el formato de hora es inválido o la salida es anterior a la entrada, restaura el hecho original con `assertz` y muestra el error.

### Instrucciones de Ejecución

**Requisitos:** [SWI-Prolog](https://www.swi-prolog.org/)

1. Coloque `registro.pl` y `University.txt` en la misma carpeta.
2. Abra una terminal en esa carpeta y ejecute:

```bash
swipl registro.pl
```

El sistema arranca automáticamente gracias a la directiva:

```prolog
:- initialization(menu, main).
```

### Formato de Almacenamiento

```prolog
estudiante(1001, 510, 0).
estudiante(1002, 480, 740).
```

> Un valor de `0` en el tercer argumento indica que el estudiante aún permanece en la universidad.

---
