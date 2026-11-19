# Laboratorio No. 02 — Robótica Industrial
## Análisis y operación del manipulador Motoman MH6

**Universidad Nacional de Colombia** · Robótica 2026-2 · LabSIR

**Integrantes:**
- Diego Andrés Barragán
- Julieth Manuela Rojas

---

## Tabla de contenido

1. [Objetivos](#1-objetivos)
2. [Contenido del repositorio](#2-contenido-del-repositorio)
3. [Comparación Motoman MH6 vs ABB IRB140](#3-comparación-motoman-mh6-vs-abb-irb140)
4. [Configuraciones home del Motoman MH6](#4-configuraciones-home-del-motoman-mh6)
5. [Movimiento manual del Motoman](#5-movimiento-manual-del-motoman)
6. [Niveles de velocidad manual](#6-niveles-de-velocidad-manual)
7. [RoboDK: funcionalidades y comunicación con el manipulador](#7-robodk-funcionalidades-y-comunicación-con-el-manipulador)
8. [RoboDK vs RobotStudio](#8-robodk-vs-robotstudio)
9. [Trayectoria polar](#9-trayectoria-polar)
10. [Cómo ejecutar el código](#10-cómo-ejecutar-el-código)
11. [Referencias](#11-referencias)

---

## 1. Objetivos

- Comprender las diferencias entre las características técnicas del Motoman MH6 y el ABB IRB140.
- Identificar y describir las configuraciones iniciales (home) del Motoman MH6.
- Realizar movimientos manuales del Motoman en distintos modos (articulaciones, cartesianos, traslaciones y rotaciones).
- Cambiar y controlar los niveles de velocidad del movimiento manual.
- Comprender las aplicaciones de RoboDK y su comunicación con el manipulador.
- Comparar RobotStudio y RoboDK.
- Diseñar y ejecutar una trayectoria polar en RoboDK y llevarla al manipulador físico.

---

## 2. Contenido del repositorio

Todos los archivos están en la raíz del repositorio, junto a este README.

| Archivo | Descripción |
|---|---|
| `README.md` | Este documento |
| `corazon.py` | Script de RoboDK (Python) que dibuja la trayectoria polar (corazón) y los nombres del equipo |
| `[POR COMPLETAR]` | Video de simulación en RoboDK |
| `[POR COMPLETAR]` | Video de la ejecución en el Motoman físico |
| `[POR COMPLETAR]` | Diagrama de flujo de acciones del robot |
| `[POR COMPLETAR]` | Plano de planta de la ubicación de los elementos |

---

## 3. Comparación Motoman MH6 vs ABB IRB140

| Característica | Motoman MH6 | ABB IRB140 |
|---|---|---|
| Fabricante | Yaskawa Motoman | ABB |
| Estructura | Articulada vertical | Articulada vertical |
| Grados de libertad | 6 | 6 |
| Carga útil | 6 kg | 6 kg (versión estándar; versiones antiguas: 5 kg) |
| Alcance | 1422 mm horizontal · 2486 mm vertical | 810 mm (hasta el centro del eje 5) |
| Repetibilidad | ±0.08 mm (ISO 9283) | 0.03 mm (promedio, prueba ISO) |
| Masa aproximada | 130 kg | 98 kg |
| Montaje | Piso, pared, techo | Piso, pared, invertido (cualquier ángulo) |
| Controlador | DX100 (también DXM100) | IRC5 |
| Rango de movimiento | S ±170° · L +155°/−90° · U +250°/−175° · R ±180° · B +225°/−45° · T ±360° | Eje 1: 360° · Eje 2: 200° · Eje 3: 280° · Eje 4: 400° (por defecto, ilimitado) · Eje 5: 240° · Eje 6: 800° (por defecto, ilimitado) |
| Velocidad máxima por eje | S 220 · L 200 · U 220 · R 410 · B 410 · T 610 °/s | Ejes 1–2: 200 · Eje 3: 260 · Ejes 4–5: 360 · Eje 6: 450 °/s |
| Velocidad máxima TCP | No indicada en el manual | 2.5 m/s |
| Momento admisible en muñeca | R 11.8 · B 9.8 · T 5.9 N·m | — |
| Condiciones ambientales | 0 a 45 °C, 20–80 % HR | — |
| Aplicaciones típicas | Ensamble, dispensado, manejo de materiales, atención de máquinas, empaque, soldadura | Soldadura por arco, ensamble, atención de máquinas, manejo de materiales, empaque, automatización de laboratorio |

**Análisis.** Ambos robots tienen la misma carga útil (6 kg), pero el MH6 alcanza aproximadamente 76 % más (1422 mm frente a 810 mm), a costa de mayor masa (130 kg frente a 98 kg) y menor repetibilidad (±0.08 mm frente a 0.03 mm). Por eso el MH6 es más adecuado para celdas donde se necesita cubrir un espacio de trabajo grande, mientras que el IRB140 es más compacto y preciso, ideal para tareas finas en espacios reducidos. En velocidad angular, el MH6 es más rápido en los ejes S, R, B y T, y el IRB140 en el eje U (260 frente a 220 °/s).

> Los datos del MH6 provienen del manual *MOTOMAN-MH6 Instructions* (HW0484977). Los del IRB140, de las hojas de datos de ABB. Ver [Referencias](#11-referencias).

---

## 4. Configuraciones home del Motoman MH6

El Motoman MH6 tiene dos configuraciones home de uso distinto:

- **Posición compacta (transporte / reposo):** deja el brazo recogido, lo que reduce el espacio ocupado y es la postura usada para mover el robot (el manual la muestra en la figura *Transporting Position*, con los soportes de envío colocados).
- **Posición desplegada (operación):** deja el brazo extendido y listo para trabajar.

| Configuración | Nombre en el laboratorio | Uso | S (°) | L (°) | U (°) | R (°) | B (°) | T (°) |
|---|---|---|---|---|---|---|---|---|
| Compacta | **[POR COMPLETAR: home1 o home2]** | Transporte / reposo | [..] | [..] | [..] | [..] | [..] | [..] |
| Desplegada | **[POR COMPLETAR: home1 o home2]** | Operación | [..] | [..] | [..] | [..] | [..] | [..] |

**Nota sobre el home de calibración.** En los controladores Yaskawa, la *home position* es la postura de calibración del manipulador, marcada con indicadores de alineación en cada eje. En ella, el controlador lee **0 pulsos en cada eje**.

**¿Cuál posición es mejor?** Ninguna es mejor en términos absolutos, porque cumplen funciones distintas. La posición compacta es preferible para transportar o guardar el robot, porque minimiza el volumen y el riesgo de colisión. La posición desplegada es preferible para operar, porque deja al robot cerca de su espacio de trabajo y listo para recibir movimientos. La elección depende del uso.

---

## 5. Movimiento manual del Motoman

El movimiento manual se hace desde el *teach pendant* (consola de programación) del controlador.

**Procedimiento general**

1. Verificar que el área de trabajo esté libre y colocar el selector de modo del pendant en **TEACH**.
2. Seleccionar el grupo de control (robot) con la tecla **[ROBOT]**, si el controlador tiene varios.
3. Seleccionar el sistema de coordenadas con **[COORD]** (ver abajo).
4. Seleccionar la velocidad manual con **[FAST]** / **[SLOW]** (ver [sección 6](#6-niveles-de-velocidad-manual)).
5. Activar los servos con **[SERVO ON READY]** y mantener presionado levemente el **interruptor de habilitación (enable switch)** ubicado en la parte posterior del pendant.
6. Mantener presionada la tecla del eje deseado para mover el robot. Se pueden presionar varias teclas a la vez para mover varios ejes simultáneamente.
7. Mientras se mantiene una tecla de eje, **[HIGH SPEED]** hace que el robot se mueva más rápido que la velocidad seleccionada (no funciona en nivel INCH).

**Cambio de modo con [COORD].** Cada vez que se presiona la tecla, el sistema de coordenadas cambia en el orden: **Articulaciones (JOINT) → Cartesiano / Cilíndrico → Herramienta (TOOL) → Usuario (USER)**. El modo activo aparece en el área de estado de la pantalla.

| Modo | Teclas | Movimiento |
|---|---|---|
| Articulaciones (JOINT) | **S−/S+, L−/L+, U−/U+, R−/R+, B−/B+, T−/T+** | Mueve cada articulación individualmente |
| Cartesiano | **X−/X+, Y−/Y+, Z−/Z+** | Traslación del TCP a lo largo de los ejes X, Y, Z |
| Cartesiano (rotación) | Teclas de rotación en las posiciones de R, B, T | Rotación de la herramienta alrededor de X, Y, Z (Rx, Ry, Rz) |

> **[POR COMPLETAR / VERIFICAR EN EL PENDANT]** Confirmar la correspondencia exacta de las teclas de rotación en el pendant del laboratorio (cuáles teclas producen Rx, Ry y Rz).

---

## 6. Niveles de velocidad manual

El controlador ofrece **cuatro niveles de velocidad** para el movimiento manual:

| Nivel | Indicación en pantalla | Descripción |
|---|---|---|
| INCH | Inching | Movimiento paso a paso |
| SLOW | `SLW` | Velocidad baja |
| MEDIUM | `MED` | Velocidad media |
| FAST | `FST` | Velocidad alta |

**Cómo cambiar de nivel**

- Cada vez que se presiona **[FAST]**, la velocidad sube en el orden: INCH → SLOW → MED → FST.
- Cada vez que se presiona **[SLOW]**, la velocidad baja en el orden: FST → MED → SLOW → INCH.
- Con **[HIGH SPEED]** + una tecla de eje, el robot se mueve más rápido que la velocidad usual (deshabilitado en INCH).

**Cómo identificar el nivel en la pantalla.** La velocidad manual seleccionada se muestra en el **área de estado** del pendant, mediante un ícono con barras de nivel, similar al indicador de señal de un teléfono celular, que aumenta con cada nivel (inching, baja, media, alta).

> **[POR COMPLETAR]** Adjuntar una foto de la pantalla del pendant mostrando el indicador.

---

## 7. RoboDK: funcionalidades y comunicación con el manipulador

### Aplicaciones principales

RoboDK es un software de simulación y programación fuera de línea (*offline programming*) de robots industriales, independiente del fabricante. Sus principales usos son:

- **Simulación:** construir una estación virtual con el robot, herramientas, marcos de referencia y objetivos, y verificar alcance, colisiones y tiempos.
- **Programación fuera de línea:** generar programas para el robot real sin detener la producción, mediante *post-procesadores* (para Motoman, programas `.JBI`).
- **Programación con Python:** controlar la estación y el robot con la API de RoboDK (`robolink`, `robomath`), como en `corazon.py`.
- **Programación en línea:** mover el robot real directamente desde el PC, mientras se simula el programa.

### ¿Qué hace RoboDK para mover el manipulador?

RoboDK calcula la cinemática inversa para cada objetivo, planifica la trayectoria (movimientos articulares `MoveJ` y lineales `MoveL`) y, según el modo elegido:

1. **Fuera de línea:** convierte la trayectoria en un programa del robot (`.JBI` para Motoman) que luego se carga al controlador.
2. **En línea (*Run on robot*):** envía los movimientos al controlador a través del driver mientras el programa se ejecuta, y monitorea la posición del robot.

### ¿Cómo se comunica con el manipulador?

Para Motoman, RoboDK usa el driver **MotomanHSE**, basado en el protocolo **High-Speed Ethernet Server (HSE)**, que viene habilitado en controladores Yaskawa recientes y **no requiere la opción MotoCom**. Permite mover y monitorear el robot desde el PC a una tasa de refresco de unos 70 Hz sobre una conexión Ethernet estándar. Los controladores compatibles incluyen DX100, DX200, FS100 y NX100.

**Pasos de conexión**

1. Conectar el PC y el controlador por Ethernet, en la misma red.
2. En el pendant, colocar el selector de modo en **REMOTE**.
3. En RoboDK: **Connect → Connect to Robot**, ingresar la **dirección IP del controlador** y presionar **Connect** (el puerto se puede ignorar).
4. Ejecutar el programa con **Run on robot**.

| Dato de la conexión | Valor |
|---|---|
| Versión de RoboDK | 6.2 |
| Driver | MotomanHSE |
| IP del controlador | **[POR COMPLETAR]** |
| IP del PC | **[POR COMPLETAR]** |
| Modelo del controlador | **[POR COMPLETAR]** (el manual del MH6 indica DX100) |

> En `corazon.py` el bloque de conexión (`robot.Connect()` y `robot.ConnectedState()`) está **comentado**; la conexión al robot físico se hizo desde la interfaz de RoboDK (**[POR COMPLETAR: confirmar]**).

---

## 8. RoboDK vs RobotStudio

| Aspecto | RoboDK | RobotStudio |
|---|---|---|
| Fabricante | RoboDK Inc. | ABB |
| Marcas de robots | Multimarca (más de 50 fabricantes) | Solo robots ABB |
| Simulación | Cinemática y trayectorias del robot; no ejecuta el código nativo del controlador | Usa un **controlador virtual** que ejecuta el mismo software (RAPID) que el robot real |
| Lenguaje | Python/API y post-procesadores a cada lenguaje del fabricante | RAPID (nativo) |
| Comunicación con robot real | Drivers por fabricante (en Motoman: MotomanHSE) | Conexión nativa al controlador IRC5 |
| Fortalezas | Flexibilidad, un solo programa para varias marcas, API en Python, curva de aprendizaje corta | Alta fidelidad con el robot real, simulación fiel del controlador, herramientas específicas de ABB |
| Limitaciones | La fidelidad al controlador real depende del post-procesador y del driver | Solo funciona con ABB |

**Usos específicos**

- **RoboDK:** laboratorios y talleres con robots de distintas marcas (como este laboratorio, que usa un Motoman y un ABB), prototipado rápido y programación con Python.
- **RobotStudio:** celdas ABB donde se busca validar en simulación exactamente el programa RAPID antes de llevarlo al IRC5.

**¿Qué significa para nosotros cada herramienta?**

> **[POR COMPLETAR — opinión del equipo]** Escribir en 2 a 4 líneas qué representa para ustedes cada herramienta (por ejemplo: RoboDK como herramienta universal y flexible; RobotStudio como simulador fiel del ecosistema ABB).

---

## 9. Trayectoria polar

### Integrantes

- Diego A. Barragán
- Julieth M. Rojas

### Curva elegida

Se eligió una curva polar con forma de **corazón**, definida como:

$$
r(\theta) = e\left(2 - 2\sin\theta + \frac{\sin\theta\,\sqrt{|\cos\theta|}}{\sin\theta + 1.4}\right), \qquad \theta \in [0,\, 2\pi]
$$

Las coordenadas cartesianas en el plano del *frame* se obtienen con una rotación de −90° para que el corazón quede derecho:

$$
x = r\cos\left(\theta - \tfrac{\pi}{2}\right), \qquad y = r\sin\left(\theta - \tfrac{\pi}{2}\right)
$$

Después de dibujar el corazón, el robot escribe los nombres **"Julieth"** y **"Diego"**, generados como trazos con `matplotlib.textpath.TextPath`.

### Parámetros utilizados

| Parámetro | Valor |
|---|---|
| Escala del corazón `e` (`escala`) | 40 mm por unidad de `r` |
| Número de segmentos (`num_points`) | 50 (51 puntos, de θ = 0 a 2π) |
| Radio inicial `r(0)` | 80 mm (2·`e`) |
| Velocidad lineal (`setSpeed`) | 300 mm/s |
| Blending / redondeo (`setRounding`) | 5 mm |
| Plano de trabajo (frame) | `Frame_from_Target1` |
| Target de reposo | `Target_Home` |
| Herramienta (TCP) | La herramienta activa del robot (`robot.PoseTool()`) |
| Superficie (`z_surface`) | 0 mm (plano del frame) |
| Altura segura (`z_safe`) | 50 mm |
| Texto | Fuente de tamaño 50, desplazado 150 mm en X |
| Movimientos | `MoveJ` para aproximaciones, `MoveL` para el trazo |

### Estructura del código (`corazon.py`)

1. **Conexión a RoboDK:** crea el objeto `Robolink`, pide seleccionar el robot y valida que sea válido. El bloque de conexión al robot físico está comentado.
2. **Frame y target:** carga `Frame_from_Target1` y `Target_Home`, los asigna al robot y fija la velocidad y el redondeo.
3. **Parámetros y texto:** define los parámetros de la curva y genera con `TextPath` los trazos de "Julieth" y "Diego".
4. **Aproximación:** va a `Target_Home`, se posiciona sobre el punto inicial del corazón y baja a la superficie.
5. **Corazón:** recorre θ de 0 a 2π en 50 pasos, calcula `r`, `x`, `y` y avanza con `MoveL`. Al terminar, sube a altura segura y vuelve a `Target_Home`.
6. **Nombres:** recorre los vértices de `TextPath`. Con cada código `MOVETO` sube, se desplaza y baja; con el resto de códigos traza continuamente. Al final sube y regresa a `Target_Home`.

> **Nota:** los puntos de control de las curvas Bézier del texto se recorren como vértices con `MoveL`, por lo que las letras quedan aproximadas por segmentos rectos.

### Diagrama de flujo de acciones del robot

> **[POR COMPLETAR]** Insertar aquí la imagen del diagrama de flujo (por ejemplo `diagrama_flujo.png`).

### Plano de planta

> **[POR COMPLETAR]** Insertar aquí la imagen del plano de planta con la ubicación de cada elemento (por ejemplo `plano_planta.png`).

### Videos

Los videos deben comenzar con la introducción oficial de LabSIR.

| Video | Enlace |
|---|---|
| Simulación en RoboDK | **[POR COMPLETAR]** |
| Ejecución física en el Motoman MH6 (controlado desde el PC) | **[POR COMPLETAR]** |

---

## 10. Cómo ejecutar el código

**Requisitos**

- RoboDK instalado (versión 6.2).
- Python con las librerías `robodk` y `matplotlib`:

```bash
pip install robodk matplotlib
```

- Una estación de RoboDK con el robot Motoman MH6, un frame llamado `Frame_from_Target1` y un target llamado `Target_Home`.

**Simulación**

1. Abrir la estación en RoboDK.
2. Ejecutar `corazon.py` (desde RoboDK con **Tools → Run Script**, o desde una terminal de Python con RoboDK abierto).
3. Seleccionar el robot en la ventana emergente.

**Ejecución en el robot físico**

1. Conectar el PC al controlador por Ethernet y colocar el pendant en **REMOTE**.
2. En RoboDK: **Connect → Connect to Robot**, ingresar la IP del controlador (driver MotomanHSE) y conectar.
3. Verificar que el área de trabajo esté libre y ejecutar el programa con **Run on robot**.

> **Seguridad:** verificar que nadie esté dentro del área de trabajo del robot y mantener al alcance el botón de paro de emergencia.

---

## 11. Referencias

- Yaskawa, *MOTOMAN-MH6 Instructions* (manual HW0484977).
- Yaskawa, *DX100 Operator's Manual* (teclas del pendant, velocidad manual y sistemas de coordenadas).
- Yaskawa Motoman Knowledge Base, *Home Position, Second Home Position, and Work Home Position*.
- ABB, hojas de datos del IRB 140 (IRB 140 product data sheet, 2021).
- RoboDK, documentación del driver para Yaskawa Motoman: <https://robodk.com/doc/en/Robots-Motoman-RoboDK-driver-Yaskawa-Motoman.html>
- Universidad Nacional de Colombia, *Laboratorio No. 02 — Robótica Industrial*, 2026.
