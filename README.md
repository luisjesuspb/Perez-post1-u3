# Perez-post1-u3


## Laboratorio 3 - Unidad 3: Arquitectura de Computadores

## 1. Introducción

En este laboratorio se trabajó con la herramienta DEBUG en el entorno DOSBox, con el propósito de explorar el funcionamiento interno del procesador, comprender el uso de los registros, observar posiciones de memoria y ejecutar instrucciones en lenguaje ensamblador.

El desarrollo se dividió en dos partes. En la primera se realizaron ejercicios de exploración de registros, memoria, ensamblado y direccionamiento directo. En la segunda se construyeron y ejecutaron pequeños programas para analizar operaciones aritméticas y ciclos de repetición.

## 2. Herramientas utilizadas

- DOSBox: emulador del entorno DOS.
- DEBUG.COM: herramienta para inspeccionar y modificar registros y memoria, ensamblar instrucciones y ejecutar programas.
- GitHub: almacenamiento del código, documentación y evidencias del laboratorio.

## 3. Parte 1: Exploración de registros y memoria

### 3.1. Exploración y modificación de registros

Se utilizó el comando `R` para consultar los registros del procesador y observar sus valores. Posteriormente, se modificó el registro AX para asignarle el valor hexadecimal `1234`.

Este ejercicio permitió reconocer la función de los registros y comprender cómo DEBUG permite consultar y modificar directamente sus valores.

**Comando utilizado:**

```text
R
R AX
```

**Resultado esperado:** el registro AX quedó establecido en `1234`.

**Evidencia:** `capturas/CP1_registros.png`

### 3.2. Exploración y modificación de memoria

Se utilizó el comando `F` para rellenar una región de memoria a partir de la dirección `DS:0200`, empleando el patrón hexadecimal `AB CD EF`.

Después se utilizó el comando `D` para visualizar el contenido de esa región y comprobar los datos almacenados. También se exploró el inicio del PSP mediante el comando `D 0 L20`.

Este ejercicio permitió observar cómo se representan los datos en memoria y cómo se pueden consultar diferentes posiciones mediante direcciones y longitudes.

**Comandos utilizados:**

```text
F DS:0200 L40 AB CD EF
D DS:0200
D 0 L20
```

**Evidencia:** `capturas/CP2_volcado_memoria.png`

### 3.3. Ensamblado y desensamblado

Se escribió un pequeño programa utilizando el comando `A`, con instrucciones para cargar valores en los registros AX y BX, sumarlos y finalizar la ejecución.

**Programa utilizado:**

```asm
MOV AX,0005
MOV BX,0003
ADD AX,BX
INT 20
```

Después se utilizó el comando `U` para desensamblar el programa y observar cómo las instrucciones escritas se representaban mediante códigos máquina.

**Resultado esperado:** AX terminó con el valor hexadecimal `0008`.

**Evidencia:** `capturas/CP3_ensamblado_desensamblado.png`

### 3.4. Memoria y direccionamiento directo

En este ejercicio se trabajó con una dirección específica de memoria. Primero se limpió la región correspondiente a `DS:0300` y luego se almacenaron los bytes `78 56`.

Posteriormente se ensambló una instrucción para leer el contenido de esa dirección y cargarlo en AX.

**Instrucciones utilizadas:**

```asm
MOV AX,[0300]
INT 20
```

Se utilizó el comando `U` para verificar las instrucciones y se modificó el registro IP para iniciar la ejecución desde la dirección `0320`.

**Resultado esperado:** AX tomó el valor `5678`, debido a que el procesador interpretó los bytes almacenados en memoria como una palabra de 16 bits, considerando el orden little-endian.

**Evidencia:** `capturas/CP4_memoria_direccionamiento.png`

## 4. Parte 2: Ejecución y análisis de programas

### 4.1. Programa de suma

Se desarrolló un programa que carga tres valores en los registros AX, BX y CX. Después realiza dos operaciones de suma para acumular los resultados en AX.

**Código ensamblador:**

```asm
MOV AX,000A
MOV BX,0005
MOV CX,0003
ADD AX,BX
ADD AX,CX
INT 20
```

**Tabla de seguimiento:**

| Instrucción | AX | BX | CX | IP |
|---|---|---|---|---|
| MOV AX,000A | 000A | - | - | 0103 |
| MOV BX,0005 | 000A | 0005 | - | 0106 |
| MOV CX,0003 | 000A | 0005 | 0003 | 0109 |
| ADD AX,BX | 000F | 0005 | 0003 | 010B |
| ADD AX,CX | 0012 | 0005 | 0003 | 010D |

**Análisis de banderas:**

Las instrucciones MOV no modifican las banderas. Después de la primera suma, el resultado fue `000F`; después de la segunda, fue `0012`.

| Operación | Resultado | ZF | CF | SF |
|---|---|---|---|---|
| ADD AX,BX | 000F | 0 | 0 | 0 |
| ADD AX,CX | 0012 | 0 | 0 | 0 |

En ambas sumas el resultado fue diferente de cero, no se produjo acarreo y el bit de signo permaneció en cero.

**Conclusión:** el programa permitió comprobar cómo se realizan operaciones aritméticas utilizando registros y cómo DEBUG permite observar los cambios después de cada instrucción.

**Evidencia:** `capturas/CP1_traza_suma.png`

### 4.2. Programa utilizando LOOP

En este ejercicio se creó un ciclo utilizando la instrucción `LOOP`. El programa inicia AX en cero y CX en cuatro. En cada repetición se incrementa AX en dos unidades y se utiliza LOOP para disminuir CX y controlar la repetición.

**Código ensamblador:**

```asm
MOV AX,0000
MOV CX,0004
ADD AX,0002
LOOP 0206
INT 20
```

**Tabla de seguimiento del ciclo:**

| Iteración | AX | CX | Comportamiento |
|---|---|---|---|
| Inicial | 0000 | 0004 | Se inicializan los registros |
| 1 | 0002 | 0003 | LOOP vuelve a 0206 |
| 2 | 0004 | 0002 | LOOP vuelve a 0206 |
| 3 | 0006 | 0001 | LOOP vuelve a 0206 |
| 4 | 0008 | 0000 | El ciclo termina |

**Análisis:**

La instrucción LOOP disminuye automáticamente el registro CX y comprueba si todavía es diferente de cero. Mientras CX tenga un valor distinto de cero, el programa regresa a la dirección donde se encuentra la instrucción ADD.

Cuando CX llega a cero, el ciclo termina y la ejecución continúa con `INT 20`.

**Resultado final:** AX = `0008` y CX = `0000`.

**Evidencia:** `capturas/CP2_traza_loop.png`

### 4.3. Programa utilizando DEC y JNZ

En este ejercicio se implementó un ciclo similar al anterior, pero utilizando las instrucciones `DEC` y `JNZ` para controlar manualmente las repeticiones.

**Código ensamblador:**

```asm
MOV AX,0000
MOV CX,0004
ADD AX,0002
DEC CX
JNZ 0306
INT 20
```

**Tabla de seguimiento del ciclo:**

| Iteración | AX | CX | Comportamiento de JNZ |
|---|---|---|---|
| Inicial | 0000 | 0004 | - |
| 1 | 0002 | 0003 | Salta a 0306 |
| 2 | 0004 | 0002 | Salta a 0306 |
| 3 | 0006 | 0001 | Salta a 0306 |
| 4 | 0008 | 0000 | No realiza el salto |

**Análisis:**

La instrucción DEC disminuye CX en una unidad. Después, JNZ comprueba la bandera ZF: si el resultado anterior no fue cero, el programa vuelve a la dirección `0306`. Cuando CX llega a cero, se activa la bandera de cero y el salto deja de realizarse.

**Resultado final:** AX = `0008` y CX = `0000`.

**Evidencia:** `capturas/CP3_traza_dec_jnz.png`

## 5. Comparación entre LOOP y DEC/JNZ

Los dos programas permitieron realizar cuatro repeticiones y obtener el mismo resultado final: AX = `0008` y CX = `0000`.

La diferencia principal está en la forma de controlar el ciclo. LOOP realiza automáticamente la disminución de CX y la comprobación para repetir. En cambio, DEC y JNZ separan estas dos acciones en instrucciones diferentes.

Según el conteo realizado durante el laboratorio, LOOP requirió 11 ejecuciones paso a paso con T, mientras que DEC/JNZ requirió 15. Esto permitió observar que la cantidad de instrucciones que se deben ejecutar puede variar dependiendo de la estructura del ciclo.

## 6. Decisiones técnicas

### 6.1. Uso de registros y memoria

Durante la primera parte se decidió trabajar directamente con los registros y las posiciones de memoria para comprender mejor cómo se almacenan y manipulan los datos. Los registros permiten realizar operaciones de manera directa dentro del procesador, mientras que la memoria permite conservar información en direcciones específicas. Esta diferencia se pudo observar al modificar AX y posteriormente almacenar y recuperar datos desde una dirección de memoria. También fue importante utilizar valores hexadecimales, porque DEBUG trabaja con este sistema de numeración. De esta manera se pudo relacionar lo que se escribe en ensamblador con los valores que realmente se encuentran almacenados en los registros y en la memoria.

### 6.2. Verificación mediante ensamblado y desensamblado

Se decidió verificar las instrucciones utilizando los comandos A y U en lugar de asumir que el programa había quedado correctamente ensamblado. El comando A permitió escribir las instrucciones y U permitió revisar cómo quedaron representadas en memoria. Esta comprobación fue importante porque un error en una dirección o en una instrucción podía provocar que el programa ejecutara algo diferente a lo esperado. Además, observar los códigos máquina ayudó a entender que cada instrucción tiene una representación en bytes y ocupa una cantidad determinada de memoria. Por eso, revisar el desensamblado antes de ejecutar fue una forma de detectar posibles errores y confirmar que las instrucciones estaban ubicadas correctamente.

### 6.3. Comparación entre LOOP y DEC/JNZ

Se utilizaron dos formas diferentes de construir un ciclo para comparar su funcionamiento. Primero se trabajó con LOOP, que reúne la disminución del contador y la condición de repetición en una sola instrucción. Después se utilizó DEC junto con JNZ, lo que permitió observar por separado cómo se modifica el contador y cómo se toma una decisión mediante las banderas del procesador. Se decidió implementar ambos métodos porque, aunque producen el mismo resultado, permiten comprender diferentes mecanismos de control de flujo. La comparación también mostró que una solución puede necesitar más instrucciones que otra, dependiendo de cómo se construya el ciclo y de cómo se realice su seguimiento paso a paso.

### 6.4. Uso de T frente a la ejecución continua

Durante las pruebas se utilizó principalmente el comando T para ejecutar las instrucciones una por una. Esta decisión permitió observar los cambios en los registros después de cada operación y comprobar cuándo se realizaban los saltos de los ciclos. Aunque la ejecución continua puede ser más rápida cuando un programa ya está verificado, durante el aprendizaje resulta más útil avanzar paso a paso porque facilita identificar errores y entender el comportamiento del procesador. También se tuvo en cuenta que T permite revisar cómo cambia IP y cómo se actualizan los registros después de cada instrucción. Por esta razón, se consideró más apropiado utilizar T para documentar las trazas del laboratorio y comprender el funcionamiento de los programas.

## 7. Observaciones

- DEBUG permite consultar y modificar registros directamente.
- Los datos y las direcciones se manejan utilizando valores hexadecimales.
- El comando D permite visualizar el contenido de la memoria.
- El comando A permite ensamblar instrucciones.
- El comando U permite revisar las instrucciones y sus códigos máquina.
- El comando T permite ejecutar un programa paso a paso.
- El registro IP indica la dirección de la siguiente instrucción que se ejecutará.
- Las instrucciones LOOP y DEC/JNZ permiten implementar ciclos, aunque su funcionamiento interno es diferente.
- El orden little-endian permite comprender por qué los bytes `78 56` se interpretan como la palabra `5678`.

## 8. Conclusiones

El desarrollo del laboratorio permitió comprender de manera práctica el funcionamiento de los registros, la memoria y las instrucciones básicas del procesador. Mediante DEBUG fue posible observar cómo cambian los valores durante la ejecución y cómo se relacionan las instrucciones de ensamblador con los códigos máquina.

También se logró identificar la diferencia entre ejecutar instrucciones individualmente y utilizar ciclos de repetición. La comparación entre LOOP y DEC/JNZ permitió reconocer que existen diferentes formas de controlar un ciclo y que cada una tiene un comportamiento particular.

Finalmente, el laboratorio ayudó a reforzar los conocimientos sobre direccionamiento, operaciones aritméticas, registros, banderas y ejecución de instrucciones, utilizando ejemplos pequeños que facilitaron la observación de cada proceso.

## 9. Evidencias

Las capturas de pantalla correspondientes a los ejercicios se encuentran almacenadas en la carpeta `capturas/`.

| Archivo | Descripción |
|---|---|
| CP1_registros.png | Exploración y modificación de registros |
| CP2_volcado_memoria.png | Exploración y modificación de memoria |
| CP3_ensamblado_desensamblado.png | Ensamblado y desensamblado |
| CP4_memoria_direccionamiento.png | Memoria y direccionamiento directo |
| CP1_traza_suma.png | Traza del programa de suma |
| CP2_traza_loop.png | Traza del ciclo LOOP |
| CP3_traza_dec_jnz.png | Traza del ciclo DEC/JNZ |

## 10. Estructura del repositorio

```text
Perez-post1-u3/
│
├── capturas/
│   ├── CP1_registros.png
│   ├── CP2_volcado_memoria.png
│   ├── CP3_ensamblado_desensamblado.png
│   ├── CP4_memoria_direccionamiento.png
│   ├── CP1_traza_suma.png
│   ├── CP2_traza_loop.png
│   └── CP3_traza_dec_jnz.png
│
└── README.md
```

## 11. Historial de commits

Los cambios del laboratorio se organizan en seis commits, distribuidos entre las dos partes:

1. `Agrega estructura inicial de la Parte 1`
2. `Completa exploración de registros y memoria`
3. `Completa ensamblado y direccionamiento directo`
4. `Agrega estructura inicial de la Parte 2`
5. `Completa traza suma y traza LOOP`
6. `Completa bucle DEC/JNZ y decisiones técnicas`

