# Robótica 2026-II – Laboratorio No. 01: Robótica Industrial – Trayectorias, Entradas y Salidas Digitales

## Índice
1.  [Resumen](#1-resumen)
    -   [Diagrama de flujo de la secuencia completa del sistema](#diagrama-de-flujo-de-la-secuencia-completa-del-sistema)
2.  [Diseño de la herramienta y TCP](#2-dise%C3%B1o-de-la-herramienta-y-tcp)
    -   [Diseño detallado de la herramienta](#dise%C3%B1o-detallado-de-la-herramienta)
    -   [Fabricación](#fabricaci%C3%B3n)
    -   [Integración en RobotStudio](#integraci%C3%B3n-en-robotstudio)
    -   [Comparación de la calibración: RobotStudio vs. robot real](#comparaci%C3%B3n-de-la-calibraci%C3%B3n-robotstudio-vs-robot-real)
3.  [Work object](#3-work-object)
4.  [Generación de las trayectorias](#4-generaci%C3%B3n-de-las-trayectorias)
    -   [Configuraciones del robot](#configuraciones-del-robot)
5.  [Estructura del programa RAPID](#5-estructura-del-programa-rapid)
    -   [5.1 Rutina principal (`main`)](#51-rutina-principal-main)
    -   [5.2 Ciclo de decoración (`DI_01`)](#52-ciclo-de-decoraci%C3%B3n-di_01)
    -   [5.3 Pose de mantenimiento (`DI_02`)](#53-pose-de-mantenimiento-di_02)
    -   [5.4 Farmear Aura (`DI_03`)](#54-farmear-aura-di_03)
    -   [5.5 NOTA: `IF` en vez de interrupciones](#55-nota-if-en-vez-de-interrupciones)
6.  [Instrucciones y funciones de RAPID utilizadas](#7-instrucciones-y-funciones-de-rapid-utilizadas)
    -   [Estructura del programa](#estructura-del-programa)
    -   [Instrucciones de movimiento](#instrucciones-de-movimiento)
    -   [Control de flujo y tiempo](#control-de-flujo-y-tiempo)
    -   [Entradas y salidas digitales](#entradas-y-salidas-digitales)
    -   [Tipos de datos](#tipos-de-datos)
7.  [Control de la banda transportadora](#8-control-de-la-banda-transportadora)
    -   [Calibración de los tiempos](#calibraci%C3%B3n-de-los-tiempos)
8.  [Dimensiones del montaje y plano de planta](#9-dimensiones-del-montaje-y-plano-de-planta)
9.  [Evidencia de funcionamiento / Video de la práctica](#11-evidencia-de-funcionamiento--video-de-la-pr%C3%A1ctica)
10.  [Anexo A: Código RAPID completo](#anexo-a-c%C3%B3digo-rapid-completo)

## 1. Resumen

Se desarrolló una celda simplificada de decoración automática de un pastel con un robot ABB IRB 140 (controlador IRC5) y una banda transportadora. El pastel se representa con una caja de `[largo × ancho × alto]` mm, que viaja sobre la banda hasta la estación de trabajo. Allí el robot escribe con un marcador los nombres de los integrantes (**Julieta** y **Diego**) y traza un dibujo personalizado. Después el pastel decorado sale por el otro extremo de la banda.

<img width="1148" height="1531" alt="image" src="https://github.com/user-attachments/assets/0cd97c2c-0269-4644-a143-a3dd91c9c86f" />


Un único programa RAPID (`Module1`) atiende tres modos de operación, seleccionados por entradas digitales:

| Entrada | Modo | Salida testigo |
|---|---|---|
| `DI_01` | Ciclo automático de transporte y decoración | `DO_01` |
| `DI_02` | Pose de mantenimiento | `DO_02` |
| `DI_03` | Rutina "Farmear Aura" | `DO_03` |

Las salidas `DO_01`, `DO_02` y `DO_03` son testigos físicos. Cada una permanece en 1 solo mientras se ejecuta su rutina y vale 0 en cualquier otro momento.

### Diagrama de flujo de la secuencia completa del sistema.
El siguiente diagrama describe la secuencia completa: arranque, espera en HOME, selección de modo y los tres modos de operación:

	<img width="5420" height="5300" alt="untitled@2x(2)" src="https://github.com/user-attachments/assets/160cfdd0-a4de-45f4-b8a8-d4e22a6e490f" />


## 2. Diseño de la herramienta y TCP

### Diseño detallado de la herramienta

La herramienta que sostiene el marcador se modeló en Onshape a partir los siguientes requisitos:

1.  **Acople con el flange.** Las dimensiones de la interfaz siguen el plano del manual del ABB IRB 140, para que la pieza asiente correctamente en el robot.
2.  **Evitar ángulos de gimbal lock.** El marcador no se orientó a 0°, 45° ni 90° respecto al flange, ya que esas orientaciones pueden producir singularidades en la representación de la orientación y movimientos bruscos del robot.
3.  **Tolerancia de hundimiento del marcador.** El diseño deja un recorrido en el que el marcador puede hundirse sin dañarse y sin dañar la herramienta. Esto deja crierta tolerancia en cuanto a  los errores de altura entre la punta y la superficie del pastel.


<img width="768" height="549" alt="image" src="https://github.com/user-attachments/assets/01da3543-a835-419d-9b2a-c98033cd1a63" />

[Ver modelo 3D interactivo](Herramienta.stl)


#### Fabricación

La herramienta se imprimió en dos partes en PETG. Se eligió este material por ser menos rígido que el PLA, lo que ayuda a absorber pequeños contactos del marcador contra la superficie.

<img width="1132" height="1046" alt="image" src="https://github.com/user-attachments/assets/f319e6e8-675d-43c5-9898-a6f421d9e625" />


#### Integración en RobotStudio

Todas las piezas se unificaron en un solo cuerpo en Onshape. Ese cuerpo se exportó como `.sat` y se importó en RobotStudio. Al importarlo, el TCP quedó ubicado exactamente en la punta del marcador, y de ahí se obtuvo el `tooldata` **`Pen_TCP`**:

| Parámetro | Valor |
|---|---|
| Posición del TCP respecto al flange (x, y, z) | 87.213, −0.079, 113.628 mm |
| Orientación (cuaternión) | `[0.8315, 0, 0.5556, 0]` |
| `robhold` | `TRUE` |
| Carga | Declarada en el `tooldata` |

El cuaternión equivale a una rotación de unos 67.5° alrededor del eje Y del flange. Con ella el eje Z de la herramienta queda alineado con la dirección del marcador, y el ángulo no coincide con ninguno de los valores evitados (0°, 45°, 90°). Se usa `robhold = TRUE` porque la herramienta va sujeta al propio robot.

Todos los movimientos de decoración y de reposo usan `Pen_TCP`, así que las trayectorias del stencil quedan referidas a la punta del marcador.





### Comparación de la calibración: RobotStudio vs. robot real

**En Robotsudio:** El TCP de `Pen_TCP` obtenido en RobotStudio proviene del modelo CAD, es decir, de una herramienta ideal, se asigna directamente a partir de la geometría del modelo de la herramienta, sin considerar errores de fabricación, de montaje en la brida ni de holgura.

**En el robot real**: El TCP se calibró de forma manual en el FlexPendant con el método de TCP de 4 puntos. Como referencia fija se usó una punta metálica fija en el espacio de trabajo, y se llevó la punta del marcador (la punta de la herramienta) hasta tocar ese mismo punto en 4 aproximaciones, cada una con una orientación diferentedel robot. Con estas 4 poses, el controlador resuelve la posición del TCP de modo que la punta de la herramienta coincida con el punto de referencia en todas ellas.

El TCP del CAD es el valor nominal, mientras que el calibrado en el robot real incorpora las desviaciones físicas de la herramienta montada, por lo que difeiere ligeramente del de RobotStudio.


## 3. Work object

Los targets se definen respecto al work object **`Workobject_2`**, no respecto al sistema base del robot. Su origen se tomó a partir de una **esquina de la caja** (el pastel). Así los targets se expresan como coordenadas sobre la superficie: X e Y recorren el plano del pastel y Z = 0 es el contacto con el marcador. Los puntos de aproximación y de levantamiento tienen Z = −50 mm (−30 mm para el punto de la "i"), es decir, el marcador separado de la superficie.

`Workobject_2` corresponde al cuadrante de la estación sobre la banda, que se definió para el dibujo. Calibrar este único frame en el robot real evitó tener que reubicar todos los puntos. La trayectoria se reutiliza cambiando solo el `wobjdata`, como pide la guía.

## 4. Generación de las trayectorias

1.  **Stencil en Onshape.** Se diseñó el dibujo (nombres y figura) como un stencil y se importó a RobotStudio. Su silueta sirvió de guía para ubicar los targets de la trayectoria.
2.  **Tipos de movimiento.**
    -   `MoveL` para los tramos rectos.
    -   `MoveC` para curvas (letras redondas y partes del dibujo).
    -   `MoveJ` para los desplazamientos de aproximación y salida de la zona de trabajo.
3.  **Levantamiento del marcador.** Cada trazo continuo empieza con un punto de aproximación sobre la superficie (Z = −50), baja al punto de contacto (Z = 0) y sube al terminar. El marcador no se arrastra entre letras. En las letras con trazo interior (por ejemplo "D" y "o") el lápiz se levanta entre el contorno exterior y el interior. El punto de la "i" se dibuja al final del nombre, en un trazo aparte.
4.  **Esquinas en ángulo recto.** El robot hace una pausa corta (`WaitTime 0.5`) en las esquinas y en los puntos de apoyo y levantamiento. Así el trazo no se redondea y las esquinas quedan en ángulo recto, y el lápiz no se levanta antes de terminar el trazo, evitando interrupciones.
5.  **Condiciones de la guía.** Todos los movimientos de decoración usan velocidad `v100` y zona `z10`, dentro de los límites del enunciado. 

> `[IMAGEN: Stencil (nombres y dibujo) con los targets de la trayectoria sobre la silueta.]`

> `[IMAGEN: Resultado del trazo sobre el pastel (simulación y robot real).]`

### Configuraciones del robot

Durante las pruebas se observó que en algunos targets el robot cambiaba bruscamente de configuración (`confdata`), con giros indeseados de las articulaciones. Para evitarlo, se revisó y **seleccionó manualmente la configuración** de cada target problemático. 

## 5. Estructura del programa RAPID

### 5.1 Rutina principal (`main`)

Al iniciar, `main` fija el estado seguro de las señales:

-   `FWD_Conveyor = 1` (sentido de la banda).
-   `BWD_Conveyor = 0` (motor apagado).
-   `DO_01`, `DO_02` y `DO_03` en 0.

Después manda el robot a HOME (`Path_Home`) y entra en un ciclo `WHILE TRUE` con dos partes:

-   **Espera.** Mientras `DI_01`, `DI_02` y `DI_03` valgan 0, el robot vuelve a HOME y se pone `DO_02` en 0. Esto garantiza que el robot descansa en HOME y que el testigo de mantenimiento se apaga al salir de esa rutina.
-   **Selección de modo.** Tres estructuras `IF`, cada una con condición excluyente. Por ejemplo, el ciclo de decoración se ejecuta solo si `DI_01 = 1`, `DI_02 = 0` y `DI_03 = 0`. Así solo se ejecuta una rutina a la vez.

### 5.2 Ciclo de decoración (`DI_01`)

1.  `Path_Home`: el robot parte de HOME.
2.  **Transporte de entrada.** Se activa `BWD_Conveyor` (motor) por 3.5 s y luego se apaga. La banda queda detenida con el pastel dentro del espacio de trabajo.
3.  `DO_01 = 1` y se ejecutan `Path_Cake1` (nombres) y `Path_Cake2` (dibujo). Los desplazamientos hacia y desde la zona de trabajo se hacen con `MoveJ` a puntos intermedios seguros, para no chocar con la banda ni con el pastel.
4.  `DO_01 = 0`.
5.  `Path_aux` y `Path_Home`: el robot se retira de la zona de interacción y regresa a HOME **antes** de volver a mover la banda.
6.  **Transporte de salida.** Se activa `BWD_Conveyor` por 2 s para llevar el pastel decorado al extremo final y luego se apaga. El ciclo termina.

La banda nunca se mueve mientras el robot dibuja, y solo se vuelve a mover cuando el robot ya está en HOME.

### 5.3 Pose de mantenimiento (`DI_02`)

Con `DI_02 = 1` se enciende `DO_02` y el robot se mueve a la pose de mantenimiento (`Path_Man`, un `MoveJ` a `Maintenance`). Esa pose deja la herramienta accesible para instalarla, retirarla, inspeccionarla o cambiar el marcador. Mientras `DI_02` siga activa, el robot permanece allí. Cuando las entradas vuelven a 0, el ciclo de espera apaga `DO_02` y regresa el robot a HOME.

### 5.4 Farmear Aura (`DI_03`)

Con `DI_03 = 1` se enciende `DO_03` y se ejecuta tres veces (`FOR i FROM 1 TO 3`) el procedimiento `Path_Aura`. Este recorre, solo con `MoveJ`, **tres configuraciones articulares** del robot, cada una con tres puntos. Se mueve así de una pose a otra de forma vistosa. Al terminar se apaga `DO_03` y el ciclo de espera regresa el robot a HOME.

La rutina usa velocidad `v100` por los límites de la guía de laboratorio. Es más vistosa a velocidades mayores: se probó en el simulador con `v1000`.

### 5.5 NOTA: `IF` en vez de interrupciones

Se probó implementar el modo de mantenimiento como interrupción (`TRAP`). Funcionó mal: al dispararse, interrumpía lo que estuviera haciendo el robot, lo dejaba detenido en mitad del movimiento y generaba un error. Se consultó al monitor, quien indicó que no se requiere una rutina de interrupción, ya que no deben activarse otras entradas mientras se ejecuta una rutina. Por eso los tres modos se implementaron con estructuras `IF` exclusivas, evaluadas cuando el robot está en reposo en HOME.



	

## 7. Instrucciones y funciones de RAPID utilizadas

### Estructura del programa

| Elemento | Para qué se usó |
|---|---|
| `MODULE Module1` | Contiene todo el programa: datos (targets, `Pen_TCP`, `Workobject_2`) y rutinas. |
| `PROC main()` | Punto de entrada. Deja las salidas en estado inicial (`FWD_Conveyor` en 1, `BWD_Conveyor` y `DO_01` a `DO_03` en 0), va a HOME con `Path_Home` y entra al ciclo de selección de modo. |
| `PROC Path_Home` | Va a `Home` con `MoveJ`. |
| `PROC Path_Cake1` | Escribe el texto "Julieta Diego" en la torta con los targets `Target_10` a `Target_1950`. |
| `PROC Path_Cake2` | Hace el dibujo con los targets `Target_2000` a `Target_3950`. |
| `PROC Path_aux` | Retira el robot de la zona de trabajo (`Target_3970`, `Target_3960`, `Target_3980`), en orden inverso a la entrada de `Path_Cake1`. |
| `PROC Path_Man` | Lleva el robot a la pose `Maintenance`. |
| `PROC Path_Aura` | Barrido de 3 poses (`Target_2700`, `_2710`, `_2720`) repetido en tres planos. |

### Instrucciones de movimiento

| Instrucción | Para qué se usó | Línea real del programa |
|---|---|---|
| `MoveJ` | Aproximación y salida de la zona de trabajo (inicio de `Path_Cake1`, `Path_aux`), cambio de letra en `Target_1610`, HOME, pose de mantenimiento y `Path_Aura`. Es movimiento articular, así que la punta no sigue una línea recta. | `MoveJ Target_3980,v100,z10,Pen_TCP\WObj:=Workobject_2;`<br>`MoveJ Home,v100,z10,Pen_TCP\WObj:=wobj0;` |
| `MoveL` | Tramos rectos de las letras y del dibujo, además de bajar y subir el lápiz (±50 mm en Z). | `MoveL Target_20,v100,z10,Pen_TCP\WObj:=Workobject_2;` |
| `MoveC` | Curvas de las letras redondas y del dibujo. Recibe un punto intermedio y uno final. | `MoveC Target_60,Target_70,v100,z10,Pen_TCP\WObj:=Workobject_2;` |

> `Path_Home` y `Path_Man` usan `wobj0`. El resto de las rutinas usa `Workobject_2`.

### Control de flujo y tiempo

| Instrucción | Para qué se usó | Línea real del programa |
|---|---|---|
| `WHILE TRUE DO ... ENDWHILE` | Ciclo infinito de `main`. | `WHILE TRUE DO` |
| `WHILE ... DO ... ENDWHILE` (interno) | Espera en reposo: mientras no hay ninguna entrada activa, vuelve a HOME y apaga `DO_02`. | `WHILE DI_01=0 AND DI_02=0 AND DI_03=0 DO` |
| `IF ... THEN ... ENDIF` | Selección exclusiva de modo: decoración (`DI_01`), mantenimiento (`DI_02`) o Aura (`DI_03`). | `IF DI_01=1 AND DI_02=0 AND DI_03=0 THEN` |
| `FOR i FROM 1 TO 3 DO ... ENDFOR` | Repite tres veces `Path_Aura`. | `FOR i FROM 1 TO 3 DO` |
| `WaitTime` | Pausa de 0.5 s tras bajar el lápiz y antes de subirlo en cada trazo. También son los tiempos fijos de la banda: 3.5 s y 2 s, marcados `!por definir` en el código. | `WaitTime 0.5;`<br>`WaitTime 3.5; !por definir`<br>`WaitTime 2; !por definir` |

### Entradas y salidas digitales

| Instrucción / función | Para qué se usó | Línea real del programa |
|---|---|---|
| `SetDO` | Salidas testigo de modo: `DO_01` (decoración), `DO_02` (mantenimiento) y `DO_03` (Aura). También las señales de la banda: `FWD_Conveyor` se pone en 1 al inicio y `BWD_Conveyor` se activa durante `WaitTime` para llevar la torta y retirarla. | `SetDO DO_01,1;`<br>`SetDO BWD_Conveyor,1;` |
| Lectura de `DI_xx` | Se leen directamente en las condiciones de los `WHILE` e `IF`; no hay instrucción de lectura aparte. | `IF DI_03=1 AND DI_02=0 AND DI_01=0 THEN` |

### Tipos de datos

| Tipo | Uso en el programa | Declaración real |
|---|---|---|
| `robtarget` | Todos los targets, incluidos `Home`, `Maintenance` y los de `Path_Aura`. Los de dibujo están expresados respecto a `Workobject_2` y llevan posición, orientación y `confdata`. | `CONST robtarget Target_20:=[[13.553,180.658,0],[1,0,0,0],[-2,-1,0,1],[9E+09,...]];` |
| `tooldata` | `Pen_TCP`: TCP en `[87.213,-0.079,113.628]`, `robhold` en TRUE y carga de 1 kg con centro de gravedad en `[10.74,0,140]`. | `PERS tooldata Pen_TCP:=[TRUE,[[87.213,-0.079,113.628],...` |
| `wobjdata` | `Workobject_2`: `ufprog` en TRUE, con marco de usuario y marco de objeto definidos. | `TASK PERS wobjdata Workobject_2:=[FALSE,TRUE,"",...` |
| `speeddata` | `v100` en todas las instrucciones de movimiento, incluidas HOME y mantenimiento. | `...,v100,z10,...` |
| `zonedata` | `z10` en todas las instrucciones de movimiento. | `...,v100,z10,...` |
| `confdata` | Configuración de cada target, dentro del `robtarget`. Los de dibujo usan `[-2,-1,0,1]` y `Home` usa `[-1,0,-1,0]`. | `[-2,-1,0,1]` |


## 8. Control de la banda transportadora

-   `FWD_Conveyor` funciona como **selector de sentido**: en 1 el movimiento es hacia un lado, en 0 hacia el otro.
-   `BWD_Conveyor` **enciende el motor**.
-   El programa deja `FWD_Conveyor = 1` y mueve la banda solo con `BWD_Conveyor`, con tiempos fijos (`WaitTime`).
-   `[Explicar cómo se verificó el movimiento en ambos sentidos (FWD = 0), con la banda detenida antes del cambio.]`

No se usó ningún sensor: la posición del pastel se controla solo con **tiempos**. Los tiempos finales son 3.5 s (entrada) y 2 s (salida).

### Calibración de los tiempos

Se caracterizó la banda con un perfil de velocidad **trapezoidal** (aceleración, velocidad constante y desaceleración) para estimar el tiempo necesario hasta la posición de decoración. Los datos son: `[velocidad nominal, rampa, tiempo teórico]`.

En las pruebas reales el tiempo calculado quedó **bastante lejos** del requerido. Hubo que ajustarlo por prueba y error hasta que el pastel quedara bien ubicado: `[tiempo teórico]` s frente a `[tiempo real]` s. Entre las causas probables están la rampa real del variador, el deslizamiento de la banda y la carga.


## 9. Dimensiones del montaje y plano de planta

-   Pastel (caja): `[largo × ancho × alto]` mm.
-   Área de dibujo: `[dimensiones]` mm.

> `[IMAGEN: Plano de planta acotado con la ubicación del robot, la banda transportadora, la superficie de trabajo (pastel), el origen de Workobject_2 y demás elementos utilizados (mesa, controlador, FlexPendant, etc.).]`



## 11. Evidencia de funcionamiento / Video de la práctica

Enlace al video: 

## Anexo A: Código RAPID completo



```
MODULE Module1  
CONST robtarget Target_10:=[[13.553,180.658,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_20:=[[13.553,180.658,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_30:=[[16.554,180.658,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_40:=[[16.554,174.309,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_50:=[[31.294,174.309,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_60:=[[33.649,175.247,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_70:=[[34.434,178.062,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_80:=[[34.434,180.759,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_90:=[[37.435,180.759,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_100:=[[37.129,175.162,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_110:=[[35.799,172.71,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_120:=[[31.737,171.105,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_130:=[[13.553,171.105,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
TASK PERS wobjdata Workobject_2:=[FALSE,TRUE,"",[[281.59,-134.65409559,-224.574233072],[0.706676031,-0.024677671,-0.024677671,-0.706676031]],[[500,-320,100],[0,0,1,0]]];  
CONST robtarget Home:=[[561.089885827,-0.079,547.15732646],[0.065403129,0,-0.997858923,0],[-1,0,-1,0],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
PERS tooldata Pen_TCP:=[TRUE,[[87.213,-0.079,113.628],[0.831469612,0,0.555570233,0]],[1,[10.7435139,0,140],[1,0,0,0],0,0,0]];  
CONST robtarget Target_140:=[[21.059,158.892,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_150:=[[21.059,158.892,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_160:=[[30.303,158.892,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_170:=[[35.222,157.84,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_180:=[[37.154,154.465,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_190:=[[37.435,145.314,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_200:=[[21.059,145.314,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_210:=[[21.059,148.316,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_220:=[[34.567,148.316,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_230:=[[34.567,152.207,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_240:=[[33.496,155.035,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_250:=[[30.271,155.891,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_260:=[[21.059,155.891,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_270:=[[13.553,133.101,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_280:=[[13.553,133.101,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_290:=[[30.953,133.101,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_300:=[[35.969,131.515,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_310:=[[37.069,129.37,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_320:=[[37.435,126.107,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_330:=[[34.567,126.107,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_340:=[[34.04,128.837,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_350:=[[30.953,130.031,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_360:=[[13.553,130.031,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_370:=[[21.059,115.941,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_380:=[[21.059,115.941,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_390:=[[23.927,117.306,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_400:=[[23.927,114.507,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_410:=[[37.435,114.507,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_420:=[[37.435,111.506,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_430:=[[21.059,111.506,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_440:=[[35.868,85.406,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_450:=[[35.868,85.406,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_460:=[[37.777,90.966,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_470:=[[35.49,97.177,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_480:=[[29.588,99.496,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_490:=[[23.071,97.209,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_500:=[[20.718,90.966,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_510:=[[22.04,86.616,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_520:=[[25.526,85.166,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_530:=[[29.8,87.035,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_540:=[[31.225,92.64,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_550:=[[31.225,94.037,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_560:=[[28.597,94.037,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_570:=[[28.422,90.463,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_580:=[[26.966,88.56,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_590:=[[24.636,88.535,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_600:=[[23.975,89.124,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_610:=[[23.447,91.137,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_620:=[[25.153,94.94,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_630:=[[29.247,96.324,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_640:=[[33.341,94.839,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_650:=[[34.909,90.966,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_660:=[[33.853,87.453,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_670:=[[23.927,75.512,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_680:=[[23.927,75.512,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_690:=[[17.818,69.403,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_700:=[[21.059,69.403,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_710:=[[21.059,64.797,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_720:=[[23.927,64.797,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_730:=[[23.927,69.403,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_740:=[[30.953,69.403,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_750:=[[33.6,68.53,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_760:=[[34.466,66.455,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_770:=[[34.567,64.797,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_780:=[[37.435,64.797,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_790:=[[37.06,68.209,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_800:=[[33.939,72.004,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_810:=[[30.953,72.473,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_820:=[[23.927,72.473,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_830:=[[13.553,53.607,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_840:=[[13.553,53.607,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_850:=[[13.553,50.606,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_860:=[[21.059,50.606,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_870:=[[21.059,47.706,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_880:=[[22.184,42.279,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_890:=[[28.192,40.03,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_900:=[[37.435,40.03,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_910:=[[37.435,43.031,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_920:=[[28.224,43.031,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_930:=[[25.485,43.553,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_940:=[[24.008,45.768,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_950:=[[23.927,50.606,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_960:=[[37.435,50.606,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_970:=[[37.435,53.607,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_980:=[[18.191,113.281,-30],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_990:=[[18.191,113.281,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1000:=[[15.731,111.585,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1010:=[[14.577,113.281,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1020:=[[16.384,115.088,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1030:=[[146.809,172.435,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1040:=[[146.809,172.435,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1050:=[[146.809,167.376,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1060:=[[147.66,162.028,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1080:=[[154.322,155.412,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1090:=[[159.836,154.535,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1100:=[[165.349,155.412,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1110:=[[169.457,157.975,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1120:=[[172.011,162.028,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1130:=[[172.862,167.376,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1140:=[[172.862,172.435,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1150:=[[150.083,168.934,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1160:=[[150.083,168.934,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1170:=[[150.083,166.928,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1180:=[[150.767,163.273,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1190:=[[152.819,160.58,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1200:=[[155.941,158.902,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1210:=[[159.836,158.292,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1220:=[[163.73,158.902,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1230:=[[166.852,160.58,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1240:=[[168.904,163.273,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1250:=[[169.588,166.928,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1260:=[[169.588,168.934,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1070:=[[150.214,157.975,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1270:=[[158.126,142.887,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1280:=[[158.126,142.887,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1290:=[[154.997,141.398,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1300:=[[154.997,136.559,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1310:=[[172.862,136.559,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1320:=[[172.862,139.833,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1330:=[[158.126,139.833,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1340:=[[168.954,110.32,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1350:=[[168.954,110.32,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1360:=[[170.106,114.152,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1370:=[[168.396,118.377,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1380:=[[163.93,119.997,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1390:=[[159.463,118.488,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1400:=[[157.603,114.338,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1410:=[[158.178,112.143,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1420:=[[159.911,111.285,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1430:=[[161.441,111.527,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1440:=[[162.458,112.254,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1450:=[[163.22,115.717,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1460:=[[163.22,117.502,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1470:=[[166.087,117.502,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1480:=[[166.087,115.978,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1490:=[[165.724,112.513,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1500:=[[164.634,110,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1510:=[[162.667,108.369,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1520:=[[159.871,107.825,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1530:=[[155.65,110,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1540:=[[154.625,114.152,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1550:=[[155.191,118.012,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1560:=[[157.193,120.962,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1570:=[[164.302,123.457,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1580:=[[170.74,120.927,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1590:=[[173.235,114.152,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1600:=[[172.357,110,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1610:=[[181.347,92.49,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1620:=[[181.347,92.49,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1630:=[[181.347,88.326,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1640:=[[180.155,82.702,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1650:=[[175.354,78.553,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1660:=[[154.997,78.125,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1670:=[[154.997,86.872,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1680:=[[157.62,93.368,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1690:=[[164.116,95.99,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1700:=[[170.405,93.385,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1710:=[[172.862,87.134,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1720:=[[171.746,82.813,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1730:=[[168.396,82.813,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1740:=[[169.734,87.134,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1750:=[[168.227,91.021,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1760:=[[164.116,92.641,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1770:=[[159.763,91.076,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1780:=[[158.126,86.872,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1790:=[[158.126,81.475,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1800:=[[172.711,81.475,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1810:=[[175.524,82.107,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1820:=[[177.404,84.005,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1830:=[[178.224,88.657,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1840:=[[178.224,92.49,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1850:=[[154.625,56.242,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1860:=[[154.625,56.242,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1870:=[[163.93,65.36,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1880:=[[173.235,56.242,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1890:=[[163.93,47.123,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1900:=[[157.754,56.242,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1910:=[[157.754,56.242,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1920:=[[163.93,62.011,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1930:=[[170.106,56.242,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1940:=[[163.93,50.473,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1950:=[[147.926,138.496,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1960:=[[147.926,138.496,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1970:=[[149.897,136.524,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1980:=[[151.868,138.496,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_1990:=[[149.897,140.467,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2000:=[[66.143,145.52,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2010:=[[66.143,145.52,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2020:=[[45.803,145.215,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2030:=[[49.561,124.516,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2040:=[[50.999,110.2,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2050:=[[54.33,99.636,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2060:=[[51.107,79.573,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2070:=[[70.562,77.356,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2080:=[[78.265,70.356,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2090:=[[88.355,61.758,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2100:=[[98.478,54.555,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2110:=[[110.9,48.433,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2120:=[[127.931,46.922,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2130:=[[139.583,61.747,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2140:=[[140.755,103.306,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2150:=[[138.731,107.126,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2160:=[[134.144,108.681,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2170:=[[127.123,103.904,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2180:=[[124.829,96.824,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2190:=[[124.829,96.824,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2200:=[[111.64,99.661,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2210:=[[101.149,83.616,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2220:=[[106.076,68.368,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2230:=[[116.775,57.459,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2240:=[[119.655,58.486,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2250:=[[118.922,62.035,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2260:=[[108.597,74.735,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2270:=[[106.651,87.401,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2280:=[[112.135,93.995,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2290:=[[122.491,92.738,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2300:=[[132.899,75.539,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2310:=[[135.133,62.723,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2320:=[[130.082,53.299,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2330:=[[120.965,50.326,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2340:=[[108.372,54.248,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2350:=[[96.998,60.856,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2360:=[[87.08,68.705,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2370:=[[77.354,77.312,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2380:=[[73.596,80.751,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2390:=[[69.815,82.973,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2400:=[[67.436,80.663,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2410:=[[54.696,82.37,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2420:=[[57.871,96.778,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2430:=[[59.281,99.454,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2440:=[[58.572,101.281,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2450:=[[55.479,110.995,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2460:=[[54.676,124.344,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2470:=[[54.313,127.895,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2480:=[[51.309,128.717,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2490:=[[48.808,141.799,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2500:=[[62.644,142.611,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2510:=[[64.51,140.579,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2520:=[[67.019,141.055,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2530:=[[73.979,142.39,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2540:=[[83.637,143.893,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2550:=[[104.284,145.199,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2560:=[[121.862,142.834,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2570:=[[132.798,135.05,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2580:=[[134.144,108.681,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2590:=[[137.952,129.596,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2600:=[[137.263,135.926,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2610:=[[135.432,139.439,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2620:=[[130.738,143.454,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2630:=[[125.472,146.368,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2640:=[[117.905,148.56,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2650:=[[106.937,149.486,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2660:=[[84.921,148.658,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2670:=[[74.498,147.107,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2690:=[[138.731,107.126,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2700:=[[377.507588906,574.849237101,-516.157175065],[0.884794151,-0.463881565,0.041985454,0.013799496],[-1,-1,1,0],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2710:=[[501.79721715,610.065504884,-493.406426534],[0.844170683,-0.315389522,0.19075209,-0.389254348],[-1,0,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2720:=[[472.899293209,647.510385883,-615.868169729],[0.448646387,-0.739711922,0.487102155,0.119474612],[-1,0,1,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2700_3:=[[306.796910787,645.559915219,-516.157175065],[0.884794151,-0.463881565,0.041985454,0.013799496],[-1,-1,1,0],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2700_4:=[[377.507588906,433.427880863,-516.157175065],[0.884794151,-0.463881565,0.041985454,0.013799496],[-1,0,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2710_3:=[[431.086539031,680.776183003,-493.406426534],[0.844170683,-0.315389522,0.19075209,-0.389254348],[-1,0,1,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2710_4:=[[501.79721715,468.644148647,-493.406426534],[0.844170683,-0.315389522,0.19075209,-0.389254348],[-1,-1,1,0],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2720_3:=[[402.188615091,718.221064002,-615.868169729],[0.448646387,-0.739711922,0.487102155,0.119474612],[-1,0,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2720_4:=[[472.899293209,506.089029646,-615.868169729],[0.448646387,-0.739711922,0.487102155,0.119474612],[-1,0,1,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2730:=[[135.559,83.046,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2740:=[[135.559,83.046,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2750:=[[129.535,94.232,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2760:=[[130.226,99.034,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2770:=[[131.717,101.847,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2780:=[[134.269,103.497,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2790:=[[135.948,102.189,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2800:=[[76.746,49.156,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2810:=[[76.746,49.156,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2820:=[[74.174,74.055,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2830:=[[72.751,95.48,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2840:=[[71.19,96.466,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2850:=[[71.315,98.796,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2860:=[[72.435,114.515,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2870:=[[67.575,130.375,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2880:=[[68.179,133.207,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2890:=[[70.177,134.312,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2900:=[[70.383,141.711,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2910:=[[70.714,146.408,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2920:=[[71.89,156.873,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2930:=[[59.659,155.189,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2940:=[[57.808,163.91,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2950:=[[61.466,168.068,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2960:=[[66.905,170.457,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2970:=[[56.374,174.395,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2980:=[[55.452,182.225,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_2990:=[[61.809,186.509,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3000:=[[69.086,183.285,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3010:=[[73.078,193.076,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3020:=[[79.26,194.745,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3030:=[[83.313,189.404,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3040:=[[81.889,182.667,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3050:=[[90.088,185.865,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3060:=[[97.282,181.9,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3070:=[[94.644,173.194,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3080:=[[84.611,169.433,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3090:=[[91.246,162.499,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3100:=[[89.518,153.24,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3110:=[[82.418,152.36,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3120:=[[75.692,157.631,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3130:=[[74.498,147.107,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3140:=[[73.979,142.39,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3150:=[[73.641,129.005,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3160:=[[77.094,114.697,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3170:=[[76.735,100.417,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3180:=[[77.023,89.49,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3190:=[[77.354,77.312,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3200:=[[78.265,70.356,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3210:=[[80.018,54.87,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3220:=[[80.545,50.281,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3230:=[[78.635,48.257,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3240:=[[69.057,167.577,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3250:=[[69.057,167.577,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3260:=[[64.083,165.089,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3270:=[[60.94,161.963,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3280:=[[61.731,158.544,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3290:=[[67.506,157.786,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3300:=[[70.506,161.782,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3310:=[[71.637,165.089,-0.103],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3320:=[[70.024,165.883,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3390:=[[70.432,174.921,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3380:=[[64.398,174.967,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3330:=[[70.432,174.921,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3340:=[[66.264,180.457,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3350:=[[61.602,182.339,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3360:=[[59.007,180.373,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3370:=[[59.261,177.007,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3400:=[[74.986,178.634,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3410:=[[74.986,178.634,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3420:=[[78.191,182.832,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3430:=[[79.867,187.542,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3440:=[[78.115,190.975,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3450:=[[74.289,189.091,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3460:=[[73.09,184.034,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3470:=[[81.338,176.155,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3480:=[[81.338,176.155,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3490:=[[88.232,181.598,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3500:=[[93.928,180.655,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3510:=[[92.162,175.715,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3520:=[[85.552,173.577,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3530:=[[81.894,173.933,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3540:=[[80.785,165.883,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3550:=[[80.785,165.883,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3560:=[[84.323,164.988,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3570:=[[86.855,162.049,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3580:=[[87.885,157.026,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3590:=[[83.062,156.181,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3600:=[[79.633,159.078,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3610:=[[77.861,164.087,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3620:=[[79.549,164.616,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3630:=[[79.041,171.758,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3640:=[[79.041,171.758,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3650:=[[76.206,175.049,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3660:=[[72.625,171.242,-1.338],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3670:=[[76.282,167.3,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3680:=[[123.718,128.545,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3690:=[[123.718,128.545,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3700:=[[120.968,131.169,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3710:=[[123.778,133.908,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3720:=[[132.768,135.197,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3730:=[[141.277,136.826,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3740:=[[144.711,134.238,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3750:=[[141.241,130.391,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3760:=[[137.952,129.596,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3770:=[[133.407,129.355,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3780:=[[130.738,143.454,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3790:=[[130.738,143.454,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3800:=[[138.713,148.374,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3810:=[[138.503,152.074,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3820:=[[135.741,152.532,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3830:=[[130.839,149.464,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3840:=[[125.472,146.368,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3850:=[[125.472,146.368,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3860:=[[51.906,136.181,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3870:=[[51.906,136.181,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3880:=[[56.078,140.353,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3890:=[[60.249,136.181,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3900:=[[56.078,132.01,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3910:=[[58.86,88.515,-50],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3920:=[[58.86,88.515,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3930:=[[62.989,92.644,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3940:=[[67.118,88.515,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3950:=[[62.989,84.386,0],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Manteinance:=[[-280.007026554,338.650422229,-485.070880198],[0.833296726,0.507308068,-0.182218756,-0.122684211],[0,-1,0,0],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3960:=[[0,0,-650],[1,0,0,0],[-2,-2,1,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3970:=[[0,0,-100],[1,0,0,0],[-2,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Target_3980:=[[300,300,-650],[1,0,0,0],[-1,-1,0,0],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
CONST robtarget Maintenance:=[[780.492172606,193.239242354,607.889123487],[0.254905697,-0.251858167,0.831139646,-0.42520282],[0,-1,0,1],[9E+09,9E+09,9E+09,9E+09,9E+09,9E+09]];  
!***********************************************************  
!  
! Module: Module1  
!  
! Description:  
! <Insert description here>  
!  
! Author: Nemito  
!  
! Version: 1.0  
!  
!***********************************************************  
  
PROC main()  
  
SetDO FWD_Conveyor,1;  
SetDO BWD_Conveyor,0;  
SetDO DO_01,0;  
SetDO DO_02,0;  
SetDO DO_03,0;  
Path_Home;  
  
WHILE TRUE DO  
  
WHILE DI_01=0 AND DI_02=0 AND DI_03=0 DO  
Path_Home;  
SetDO DO_02,0;  
ENDWHILE  
  
IF DI_01=1 AND DI_02=0 AND DI_03=0 THEN  
! rutina de decoracion  
Path_Home;  
SetDO BWD_Conveyor,1;  
WaitTime 3.5; !por definir  
SetDO BWD_Conveyor,0;  
SetDO DO_01,1;  
Path_Cake1;  
Path_Cake2;  
SetDO DO_01,0;  
Path_aux;  
Path_Home;  
SetDO BWD_Conveyor,1;  
WaitTime 2; !por definir  
SetDO BWD_Conveyor,0;  
ENDIF  
  
IF DI_02=1 AND DI_03=0 AND DI_01=0 THEN  
SetDO DO_02,1;  
Path_Man;  
ENDIF  
  
IF DI_03=1 AND DI_02=0 AND DI_01=0 THEN  
SetDO DO_03,1;  
! farmeando aura  
FOR i FROM 1 TO 3 DO  
Path_Aura;  
ENDFOR  
SetDO DO_03,0;  
ENDIF  
  
ENDWHILE  
ENDPROC  
  
PROC Path_Cake1()  
MoveJ Target_3980,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_3960,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_3970,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_10,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_20,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_30,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_40,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_50,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_60,Target_70,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_80,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_90,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_100,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_110,Target_120,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_130,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_20,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
  
MoveL Target_10,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
!termina la J  
  
MoveL Target_140,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_150,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
  
MoveL Target_160,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_170,Target_180,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_190,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_200,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_210,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_220,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_230,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_240,Target_250,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_260,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_150,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
  
MoveL Target_140,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
!termina la u  
  
MoveL Target_270,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_280,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
  
MoveL Target_290,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveC Target_300,Target_310,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_320,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_330,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_340,Target_350,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_360,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_280,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
  
MoveL Target_270,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
!termina la l  
  
MoveL Target_370,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_380,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
  
MoveL Target_390,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_400,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_410,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_420,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_430,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_380,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_370,v100,z10,Pen_TCP\WObj:=Workobject_2;  
!termina la i sin punto  
  
MoveL Target_440,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_450,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
  
MoveC Target_460,Target_470,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_480,Target_490,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_500,Target_510,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_520,Target_530,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_540,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_550,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_560,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveC Target_570,Target_580,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_590,Target_600,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_610,Target_620,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_630,Target_640,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_650,Target_660,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_450,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_440,v100,z10,Pen_TCP\WObj:=Workobject_2;  
!termina la e  
  
MoveL Target_670,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_680,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
  
MoveL Target_690,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_700,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_710,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_720,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_730,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_740,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveC Target_750,Target_760,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_770,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_780,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_790,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveC Target_800,Target_810,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_820,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_680,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_670,v100,z10,Pen_TCP\WObj:=Workobject_2;  
!termina la t  
  
MoveL Target_830,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_840,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
  
MoveL Target_850,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_860,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_870,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveC Target_880,Target_890,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_900,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_910,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_920,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveC Target_930,Target_940,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_950,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_960,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_970,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_840,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_830,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
!aqui va el punto de la i  
  
MoveL Target_980,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_990,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_1000,Target_1010,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1020,Target_990,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_980,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
!se acaba el punto de la i  
  
MoveL Target_1030,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1040,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_1050,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1060,Target_1070,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1080,Target_1090,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1100,Target_1110,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1120,Target_1130,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1140,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1040,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_1030,v100,z10,Pen_TCP\WObj:=Workobject_2;  
!ahora la parte de adentro de la D  
MoveL Target_1150,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1160,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_1170,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1180,Target_1190,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1200,Target_1210,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1220,Target_1230,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1240,Target_1250,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1260,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1160,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_1150,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
!termina la D  
  
MoveL Target_1270,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1280,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_1290,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1300,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1310,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1320,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1330,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1280,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_1270,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
!termina la i  
  
MoveL Target_1340,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1350,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_1360,Target_1370,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1380,Target_1390,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1400,Target_1410,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1420,Target_1430,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1440,Target_1450,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1460,v100,z10,Pen_TCP\WObj:=Workobject_2;  
!aqui normal  
MoveL Target_1470,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1480,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1490,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1500,Target_1510,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1520,Target_1530,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1540,Target_1550,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1560,Target_1570,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1580,Target_1590,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1600,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1350,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_1340,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
!termina la e  
  
MoveJ Target_1610,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1620,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_1630,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveC Target_1640,Target_1650,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1660,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1670,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1680,Target_1690,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1700,Target_1710,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1720,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1730,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1740,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1750,Target_1760,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1770,Target_1780,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1790,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1800,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1810,Target_1820,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1830,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1840,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1620,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_1610,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
!termina la g  
  
MoveL Target_1850,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1860,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_1870,Target_1880,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1890,Target_1860,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_1850,v100,z10,Pen_TCP\WObj:=Workobject_2;  
!la parte de adentro de la o  
MoveL Target_1900,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1910,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_1920,Target_1930,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1940,Target_1910,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_1900,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
!termina la o  
  
MoveL Target_1950,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_1960,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_1970,Target_1980,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_1990,Target_1960,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_1950,v100,z10,Pen_TCP\WObj:=Workobject_2;  
!y termina le punto de la i  
  
ENDPROC  
  
PROC Path_aux()  
MoveJ Target_3970,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_3960,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_3980,v100,z10,Pen_TCP\WObj:=Workobject_2;  
ENDPROC  
  
PROC Path_Home()  
MoveJ Home,v100,z10,Pen_TCP\WObj:=wobj0;  
ENDPROC  
  
PROC Path_Cake2()  
!este es el dibujo  
MoveJ Target_2000,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2010,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_2020,Target_2030,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_2040,Target_2050,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_2060,Target_2070,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_2080,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2090,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2100,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2110,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_2120,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2130,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2140,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveC Target_2150,Target_2160,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2170,Target_2180,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2190,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_2180,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveC Target_2200,Target_2210,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2220,Target_2230,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_2240,Target_2250,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_2260,Target_2270,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_2280,Target_2290,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2300,Target_2310,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2320,Target_2330,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2340,Target_2350,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2360,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2370,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2380,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2390,Target_2400,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_2410,Target_2420,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2430,Target_2440,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveC Target_2450,Target_2460,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2470,Target_2480,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2490,Target_2500,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2510,Target_2520,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveC Target_2530,Target_2540,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2550,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2560,Target_2570,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2160,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
  
MoveL Target_2580,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2690,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2150,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
  
MoveL Target_2590,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2600,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2610,Target_2620,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2630,Target_2640,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2650,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2660,Target_2670,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2010,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_2000,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_2730,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2740,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_2750,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2760,Target_2770,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2780,Target_2790,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2740,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_2730,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_2800,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2810,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_2820,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_2830,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_2840,Target_2850,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2860,Target_2870,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveC Target_2880,Target_2890,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
  
MoveL Target_2900,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2910,Target_2920,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_2930,Target_2940,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2950,Target_2960,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_2970,Target_2980,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_2990,Target_3000,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3010,Target_3020,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3030,Target_3040,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3050,Target_3060,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3070,Target_3080,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3090,Target_3100,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3110,Target_3120,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_3130,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3140,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3150,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3160,Target_3170,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3180,Target_3190,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3200,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3210,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3220,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3230,Target_2810,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_2800,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_3240,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3250,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3260,Target_3270,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3280,Target_3290,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3300,Target_3310,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3320,Target_3250,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_3240,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_3390,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3330,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3340,Target_3350,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3360,Target_3370,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3380,Target_3330,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_3390,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_3400,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3410,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3420,Target_3430,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3440,Target_3450,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3460,Target_3410,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_3400,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_3470,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3480,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_3490,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3500,Target_3510,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3520,Target_3530,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3480,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_3470,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_3540,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3550,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3560,Target_3570,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3580,Target_3590,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3600,Target_3610,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3620,Target_3550,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_3540,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_3630,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3640,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3650,Target_3660,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3670,Target_3640,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_3630,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_3680,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3690,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3700,Target_3710,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3720,Target_3730,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3740,Target_3750,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3760,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3770,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3690,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_3680,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_3780,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3790,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_3800,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3810,Target_3820,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3830,Target_3840,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_3850,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_3860,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3870,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3880,Target_3890,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3900,Target_3870,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_3860,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
MoveL Target_3910,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveL Target_3920,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveC Target_3930,Target_3940,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveC Target_3950,Target_3920,v100,z10,Pen_TCP\WObj:=Workobject_2;  
WaitTime 0.5;  
MoveL Target_3910,v100,z10,Pen_TCP\WObj:=Workobject_2;  
  
ENDPROC  
  
PROC Path_Aura()  
MoveJ Target_2700,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_2710,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_2720,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_2720,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_2700_3,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_2710_3,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_2720_3,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_2700_4,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_2710_4,v100,z10,Pen_TCP\WObj:=Workobject_2;  
MoveJ Target_2720_4,v100,z10,Pen_TCP\WObj:=Workobject_2;  
ENDPROC  
  
PROC Path_Man()  
MoveJ Maintenance,v100,z10,Pen_TCP\WObj:=wobj0;  
ENDPROC  
ENDMODULE

```


