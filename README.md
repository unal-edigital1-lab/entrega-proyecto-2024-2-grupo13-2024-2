[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=17800245&assignment_repo_type=AssignmentRepo)
# Entrega 1 proyecto: Juego Arcade

* Jesus Antonio Lopez Trigozo
* Julian Camilo Casallas

# Proyecto Final: Implementación de Juego con Control Bluetooth en FPGA

## Descripción del Proyecto

Este proyecto tiene como objetivo el desarrollo de un juego en FPGA, específicamente un juego tipo **Pong** o **Destruir Ladrillos**, en el cual el control del juego se realizará a través de una **aplicación móvil** conectada al sistema mediante un módulo **Bluetooth HC-05**. El juego se implementará en una FPGA y se mostrará en una pantalla a través de una salida **VGA**. El objetivo de este diseño es proporcionar una experiencia de juego interactiva sin necesidad de cables adicionales o periféricos complicados, utilizando únicamente la FPGA, el módulo Bluetooth y el teléfono móvil como control.

## Especificación Detallada del Sistema

### Componentes Principales

1. **FPGA (Plataforma de Implementación)**
   - **Funcionalidad**: La FPGA será la encargada de procesar el juego y la interfaz gráfica. Implementará la lógica del juego (movimiento de objetos, colisiones, puntajes) y la salida de video VGA. 
   - **Entradas**: Señales de control recibidas desde el teléfono móvil (a través del módulo Bluetooth).
   - **Salidas**: Señal de video VGA para la visualización en pantalla.

2. **Módulo Bluetooth HC-05**
   - **Funcionalidad**: El módulo Bluetooth permitirá la comunicación inalámbrica entre la FPGA y el teléfono móvil, que servirá como control del juego.
   - **Conexión**: El HC-05 se conectará a la FPGA mediante la interfaz UART, permitiendo recibir comandos desde la aplicación móvil (como mover los controles del juego).

3. **Teléfono Móvil**
   - **Funcionalidad**: El teléfono móvil será utilizado para controlar el juego. A través de una aplicación personalizada, el usuario podrá enviar comandos de control (por ejemplo, mover el paddle en el juego) al módulo Bluetooth HC-05, que a su vez los enviará a la FPGA.
   - **Conexión**: La conexión Bluetooth se gestionará mediante una aplicación desarrollada por el usuario, la cual se encargará de enviar los datos de control a la FPGA.

4. **Pantalla VGA**
   - **Funcionalidad**: La pantalla VGA se utilizará para mostrar la salida gráfica del juego, que incluye los objetos (paddle, bola, ladrillos) y el puntaje.
   - **Conexión**: La FPGA generará las señales necesarias para la salida VGA, con la correcta sincronización de las señales HSync y VSync para visualizar el juego en la pantalla.

### Funcionalidad del Juego

El juego será un clásico tipo **Pong** o **Destruir Ladrillos** en el que el jugador controlará un paddle en la parte inferior de la pantalla, moviéndolo hacia la izquierda o derecha para evitar que la pelota (o el ladrillo) se caiga o se destruya. El juego deberá tener características como:

- Movimiento de los objetos en la pantalla (bola, paddle, ladrillos).
- Detención de la pelota al llegar a los límites o interacción con los objetos.
- Puntaje en la pantalla.
- Interactividad mediante el control del teléfono móvil.

## Plan Inicial de la Arquitectura del Sistema

### a) **Especificación de los Componentes del Proyecto**

1. **FPGA (Plataforma de Implementación)**
   - Descripción Funcional: La FPGA implementará toda la lógica del juego, incluida la creación de la interfaz de usuario (gráficos en la pantalla VGA) y la gestión de las entradas del control (a través de Bluetooth).
   - Sistema de Caja Negra: La FPGA recibe señales de entrada desde el módulo Bluetooth, genera las señales VGA y maneja la lógica del juego (control de la pelota, el paddle y el puntaje).

2. **Módulo Bluetooth HC-05**
   - Descripción Funcional: El HC-05 se encarga de la transmisión y recepción de datos entre el teléfono móvil y la FPGA. El teléfono enviará comandos que serán interpretados por la FPGA para mover el paddle y ajustar otros aspectos del juego.
   - Sistema de Caja Negra: El HC-05 recibe comandos de la aplicación móvil y los envía a la FPGA, y viceversa, asegurando una comunicación bidireccional.

3. **Teléfono Móvil**
   - Descripción Funcional: El teléfono móvil envía comandos de control para mover el paddle y posiblemente otras configuraciones del juego (como reiniciar el puntaje). La app será responsable de transmitir estos comandos por Bluetooth.
   - Sistema de Caja Negra: La aplicación en el teléfono móvil envía datos de control (por ejemplo, "mover izquierda" o "mover derecha") al módulo Bluetooth, que los pasa a la FPGA.

4. **Pantalla VGA**
   - Descripción Funcional: La pantalla VGA visualizará el estado del juego, incluidos los objetos del juego y el puntaje.
   - Sistema de Caja Negra: La FPGA generará señales VGA con la información del juego para ser mostrada en la pantalla.

### b) **Lenguaje Adecuado y Descripción del Sistema**

Para este proyecto, la FPGA se programará utilizando un lenguaje de descripción de hardware (HDL), como **VHDL** o **Verilog**, para implementar la lógica del juego, la generación de las señales VGA y la comunicación con el módulo Bluetooth HC-05.

## Documentación Inicial en el Repositorio de Git

### a) **Estructura de Documentación**

El repositorio de Git contendrá los siguientes elementos iniciales:

1. **README.md**: Este archivo, que describe la visión general del proyecto, los componentes involucrados y el objetivo general del sistema.
2. **Código Fuente**:
   - Archivos VHDL/Verilog para la implementación de la lógica del juego.
   - Archivos de configuración para el módulo Bluetooth HC-05 y la generación de las señales VGA.
3. **Diagramas**: Diagramas de bloques que describen cómo los distintos componentes interactúan entre sí (FPGA, Bluetooth, móvil y VGA).

### b) **Estructura Inicial del Proyecto en Git**

El repositorio estará organizado de la siguiente manera:

