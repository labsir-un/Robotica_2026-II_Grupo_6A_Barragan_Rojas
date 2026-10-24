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

<img width="5420" height="5300" alt="untitled@2x(2)" src="https://github.com/user-attachments/assets/ea0a45d4-256f-4bc6-933b-5c4d252282cd" />



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
 <img width="709" height="601" alt="image" src="https://github.com/user-attachments/assets/6930fef0-e083-4df0-814d-02b01c8ef502" />

3.  **Tipos de movimiento.**
    -   `MoveL` para los tramos rectos.
    -   `MoveC` para curvas (letras redondas y partes del dibujo).
    -   `MoveJ` para los desplazamientos de aproximación y salida de la zona de trabajo.
4.  **Levantamiento del marcador.** Cada trazo continuo empieza con un punto de aproximación sobre la superficie (Z = −50), baja al punto de contacto (Z = 0) y sube al terminar. El marcador no se arrastra entre letras. En las letras con trazo interior (por ejemplo "D" y "o") el lápiz se levanta entre el contorno exterior y el interior. El punto de la "i" se dibuja al final del nombre, en un trazo aparte.
5.  **Esquinas en ángulo recto.** El robot hace una pausa corta (`WaitTime 0.5`) en las esquinas y en los puntos de apoyo y levantamiento. Así el trazo no se redondea y las esquinas quedan en ángulo recto, y el lápiz no se levanta antes de terminar el trazo, evitando interrupciones.
6.  **Condiciones de la guía.** Todos los movimientos de decoración usan velocidad `v100` y zona `z10`, dentro de los límites del enunciado. 


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

No se usó ningún sensor: la posición del pastel se controla solo con **tiempos**. Los tiempos finales son 3.5 s (entrada) y 2 s (salida).

### Calibración de los tiempos

Se caracterizó la banda con un perfil de velocidad **trapezoidal** (aceleración, velocidad constante y desaceleración) para estimar el tiempo necesario hasta la posición de decoración. Los datos son: `[velocidad nominal, rampa, tiempo teórico]`.

En las pruebas reales el tiempo calculado quedó **bastante lejos** del requerido. Hubo que ajustarlo por prueba y error hasta que el pastel quedara bien ubicado: `[tiempo teórico]` s frente a `[tiempo real]` s. Entre las causas probables están la rampa real del variador, el deslizamiento de la banda y la carga.




## 9.Video de la práctica (Evidencia de funcionamiento)



## Anexo A: Código RAPID completo
[Ver Archivo RAPID](Module1.mod)

## Anexo B: CAD Herramienta

[Ver Archivo CAD](Herramienta.stl)
