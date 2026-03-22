Sistema de Registro Universitario - Haskell (Versión A)

Este proyecto es el desarrollo de la Versión A de la Práctica I para la asignatura ST0244 - Lenguajes de Programación en la Universidad EAFIT. Consiste en un sistema funcional para gestionar el ingreso y salida de estudiantes del campus.
Especificaciones de la Práctica

    Materia: Programación con Lenguajes de Programación.

    Docente: Alexander Narváez Berrío.

    Valor: 15% de la nota final.

    Paradigma: Funcional.

Requerimientos Funcionales

El sistema implementa las siguientes funciones exigidas en la guía:

    Check In: Registro de entrada del estudiante solicitando su ID y almacenando la hora de ingreso.

    Search by ID: Búsqueda de estudiantes que se encuentran actualmente dentro de la universidad.

    Time Calculation: Cálculo automático de la estancia en formato legible (horas y minutos).

    Students List: Carga de registros desde el archivo University.txt y visualización en terminal.

    Check Out: Registro de salida y actualización de la persistencia de datos.

Análisis Técnico para la Defensa

Para la sustentación presencial, es fundamental comprender los siguientes pilares del código:
1. Modelo de Datos e Inmutabilidad

En Haskell, no modificamos variables. Utilizamos una estructura de datos Estudiante definida con data. Para "actualizar" un registro (como en el Check Out), el programa utiliza la función map para recorrer la lista actual y generar una nueva lista con el dato cambiado, manteniendo la integridad de los datos originales.
2. Gestión de Tiempo

Se implementó la Opción 3 de la guía: el tiempo se almacena como un número entero que representa los minutos transcurridos desde las 00:00.

    Ejemplo: Las 08:30 AM se almacenan como 510 minutos.

    Esto facilita el cálculo de permanencia mediante una resta simple: TiempoSalida - TiempoEntrada.

3. Persistencia y Lazy Evaluation

El programa lee y escribe en el archivo University.txt. Debido a que Haskell es un lenguaje de evaluación perezosa (Lazy), el código fuerza la lectura completa del archivo antes de permitir una nueva escritura, evitando conflictos de acceso al archivo durante la ejecución en vivo.
4. Recursividad en el Menú

A falta de ciclos imperativos (como while), el menú principal es una función recursiva. Al finalizar cada operación, la función se llama a sí misma para mantener el programa en ejecución hasta que el usuario decida salir.
Instrucciones de Ejecución
Requisitos

    Tener instalado GHC (Glasgow Haskell Compiler).

Pasos para ejecutar

    Coloque el archivo Main.hs y University.txt en la misma carpeta.

    Abra una terminal y ejecute:
    Bash

    runhaskell Main.hs

Formato de Almacenamiento (University.txt)

El archivo guarda los datos en formato de lista de estructuras nativas de Haskell:
Estudiante {idEst = "123", entrada = 480, salida = -1}

    Un valor de -1 en salida indica que el estudiante aún permanece en la universidad.