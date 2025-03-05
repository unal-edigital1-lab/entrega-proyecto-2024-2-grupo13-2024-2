[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=17800245&assignment_repo_type=AssignmentRepo)
# Entrega 1 proyecto: Juego Arcade

* Jesus Antonio Lopez Trigozo
* Julian Camilo Casallas

# Proyecto Final: Implementación de Juego con Control Bluetooth en FPGA

## Descripción del Proyecto

Este proyecto tiene como objetivo el desarrollo de un juego en FPGA, específicamente un juego tipo **Pong**, en el cual el control del juego se realizará a través de una **aplicación móvil** conectada al sistema mediante un módulo **Bluetooth HC-05**. El juego se implementará en una FPGA y se mostrará en una pantalla a través de una salida **VGA**. El objetivo de este diseño es proporcionar una experiencia de juego interactiva sin necesidad de cables adicionales o periféricos complicados, utilizando únicamente la FPGA, el módulo Bluetooth y el teléfono móvil como control.

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
   - **Funcionalidad**: La pantalla VGA se utilizará para mostrar la salida gráfica del juego, que incluye los objetos (paddle, bola).
   - **Conexión**: La FPGA generará las señales necesarias para la salida VGA, con la correcta sincronización de las señales HSync y VSync para visualizar el juego en la pantalla.

### Funcionalidad del Juego

El juego será un clásico tipo **Pong** en el que el jugador controlará un paddle en la parte lateral de la pantalla, moviéndolo hacia arriba o abajp para evitar que la pelota se salga o se destruya. El juego deberá tener características como:

- Movimiento de los objetos en la pantalla (bola, paddle).
- Detención de la pelota al llegar a los límites o interacción con los objetos.
- Puntaje en la pantalla.
- Interactividad mediante el control del teléfono móvil.

## Plan Inicial de la Arquitectura del Sistema

```plaintext
       ┌──────────────────────────────┐
       │          vga_demo            │───────────┐
       │           (Main)             │           │
       ▲───────────▲──────────────────┤           │
       │           │                  │           │ 
       ▼           ▼                  ▼           │
┌──────────┐  ┌─────────────┐   ┌─────────────┐   │
│   TOP    │  │ ball_logic  │   │    VGA      │   │
│ (Serial) │  │ (Lógica)    │◄─►│ (Video Out) │   │
└──────────┘  └────▲──────▲─┘   └─────────────┘   │
        ▲          │      │───────────────┐       │
        │          │                      │       │
        ▼          ▼                      ▼       ▼
┌──────────┐    ┌──────────────┐    ┌─────────────────────┐
│ Bluetooth│    │ rectangle_1  │    │ rectangle_2         │
│(uart_rx) │---▶│ (Jugador 1)  │    │ (Jugador 2 autónomo)│
└──────────┘    └──────────────┘    └─────────────────────┘
  
```

### a) **Especificación de los Componentes del Proyecto**

1. **FPGA (Plataforma de Implementación)**
   - Descripción Funcional: La FPGA implementará toda la lógica del juego, incluida la creación de la interfaz de usuario (gráficos en la pantalla VGA) y la gestión de las entradas del control (a través de Bluetooth).


2. **Módulo Bluetooth HC-05**
   - Descripción Funcional: El HC-05 se encarga de la transmisión y recepción de datos entre el teléfono móvil y la FPGA. El teléfono enviará comandos que serán interpretados por la FPGA para mover el paddle y ajustar otros aspectos del juego.

3. **Teléfono Móvil**
   - Descripción Funcional: El teléfono móvil envía comandos de control para mover el paddle y posiblemente otras configuraciones del juego . La app será responsable de transmitir estos comandos por Bluetooth.

4. **Pantalla VGA**
   - Descripción Funcional: La pantalla VGA visualizará el estado del juego, incluidos los objetos del juego y el puntaje.

### b) **Lenguaje Adecuado y Descripción del Sistema**

Para este proyecto, la FPGA se programará utilizando un lenguaje de descripción de hardware (HDL), como  **Verilog**, para implementar la lógica del juego, la generación de las señales VGA y la comunicación con el módulo Bluetooth HC-05.


# Código implementado 

### vga_demo
- este es el Código principal , el cual llama a todos los demas, y hace que funcionen en conjungo

```verilog
module vga_demo(
    input clk_50,                    // 50 MHz / 20 ns
    input button_left,               // Entrada de botón para mover a la izquierda
    input button_right,              // Entrada de botón para mover a la derecha
    input rx,                        // Entrada de datos UART (RX) para controlar el LED
    output red, green, blue,         // VGA color outputs
    output hsync, vsync,             // VGA sync outputs
    output led                       // Salida para el LED
);

    wire [9:0] hpos, vpos;           // Posición actual del píxel
    wire active;                     // Bandera de área activa
    wire pixel_tick;                 // Pulso generado cuando la posición del píxel es estable (25 MHz)
    wire frame_tick;                 // Pulso generado cuando entra en el área de blanking (60 Hz)

    reg [2:0] pixel;                 // Color RGB del píxel actual

    // Parámetros del rectángulo
    localparam RECTANGLE_WIDTH = 10;   // Ancho original del rectángulo
    localparam RECTANGLE_HEIGHT = 100; // Alto original del rectángulo
    localparam SCREEN_WIDTH = 640;     // Ancho de la pantalla VGA
    localparam SCREEN_HEIGHT = 480;    // Alto de la pantalla VGA

    // VGA generator (modulo vga)
    vga vga_gen(
        .clk(clk_50),
        .reset(1'b0),                 // No hay reset en este módulo
        .pixel_rgb(pixel),
        .hsync(hsync),
        .vsync(vsync),
        .red(red),
        .green(green),
        .blue(blue),
        .active(active),
        .ptick(pixel_tick),
        .xpos(hpos),
        .ypos(vpos),
        .ftick(frame_tick)
    );

    // Lógica de la pelota (ahora de 4x4 píxeles)
    wire [9:0] square_hpos;
    wire [9:0] square_vpos;
    wire move_up, move_right;
    wire reset_game;  // Señal de reset para todo el juego
    ball_logic ball(
         .clk(frame_tick),
         .reset(reset_game),  // Usar reset de la pelota y raquetas
         .rectangle1_vpos(rectangle1_vpos), // Posición de la raqueta izquierda
         .rectangle2_vpos(rectangle2_vpos), // Posición de la raqueta derecha
         .rectangle_width(RECTANGLE_WIDTH), // Ancho de las raquetas
         .rectangle_height(RECTANGLE_HEIGHT), // Alto de las raquetas
         .square_hpos(square_hpos),
         .square_vpos(square_vpos),
         .move_up(move_up),
         .move_right(move_right),
         .reset_game(reset_game)  // Pasar la señal de reset
    );


    // Lógica del primer rectángulo que se mueve (a la izquierda)
    wire [9:0] rectangle1_vpos; // Posición vertical del primer rectángulo
    rectangle_logic rectangle1(
        .clk(frame_tick),
        .reset(1'b0),  // No se necesita reset
        .move_left(!led_from_top),  // Mover hacia arriba cuando el LED está apagado
        .move_right(led_from_top), // Mover hacia abajo cuando el LED está encendido
        .rectangle_vpos(rectangle1_vpos)
    );

    // Lógica del segundo rectángulo que se mueve (a la derecha) con el nuevo módulo rectangle_logic2
    wire [9:0] rectangle2_vpos; // Posición vertical del segundo rectángulo
    rectangle_logic2 rectangle2(  // Nuevo módulo para la raqueta derecha
        .clk(frame_tick),
        .reset(reset_game),  // Usar reset de la pelota y raquetas
        .square_vpos(square_vpos), // Pasar la posición de la pelota
        .square_hpos(square_hpos), // Pasar la posición horizontal de la pelota
        .move_right(move_right), // Indicar si la pelota se mueve a la derecha
        .rectangle_vpos(rectangle2_vpos)
    );


    // Instanciación del módulo TOP para controlar el LED
    wire led_from_top;   // Señal para el LED proveniente de TOP
    TOP u_top (
        .clk(clk_50),      // Reloj de 50 MHz
        .rx(rx),           // Entrada UART RX
        .led(led_from_top) // Salida LED controlada por UART
    );

    // Lógica para la salida del LED en VGA demo
    assign led = led_from_top;  // Conectar la señal del LED al módulo VGA

    // Dibujar el primer rectángulo (movimiento vertical)
    always @(posedge pixel_tick) begin
        if (!active)
            pixel <= 3'b0;  // Apagar el píxel si está fuera del área activa
        else begin
            // Dibujar la pelota (4x4 píxeles)
            if ((hpos >= square_hpos) && (hpos < (square_hpos + 6)) &&
                (vpos >= square_vpos) && (vpos < (square_vpos + 6))) begin
                pixel <= 3'b111; // Color blanco (RGB: 111)
            end
            // Dibujar el primer rectángulo (movimiento vertical)
            else if ((hpos >= 0) && (hpos < RECTANGLE_WIDTH) &&
                (vpos >= rectangle1_vpos) && (vpos < (rectangle1_vpos + RECTANGLE_HEIGHT))) begin
                pixel <= 3'b100; // Color blanco (RGB: 111)
            end
            // Dibujar el segundo rectángulo (derecha)
            else if ((hpos >= SCREEN_WIDTH - RECTANGLE_WIDTH) && (hpos < SCREEN_WIDTH) &&
                     (vpos >= rectangle2_vpos) && (vpos < (rectangle2_vpos + RECTANGLE_HEIGHT))) begin
                pixel <= 3'b001; // Color blanco (RGB: 111)
            end
            else begin
                pixel <= 3'b0; // Fondo negro
            end
        end
    end

endmodule

```

### vga

- este módulo genera señales VGA para mostrar imágenes en una pantalla con resolución 640x480 a 60 Hz.

🔹 Sincroniza la señal de video con hsync y vsync.
🔹 Controla la posición del píxel con xpos y ypos.
🔹 Activa o desactiva colores (red, green, blue) dependiendo de si el píxel está dentro del área visible.
🔹 Divide el reloj de 50 MHz a 25 MHz para ajustarse a la frecuencia de píxeles de la pantalla.


```verilog
module vga(
	input						clk,										// 50 MHz / 20 ns
	input						reset,
	input	[2:0]				pixel_rgb,								// Current pixel RGB data to be displayed

	output					hsync, vsync,							// VGA sync signals
	output					red, green, blue,						// VGA RGB data
	output					active,									// Active when pixel inside the 640 x 480 area
	output					ptick,									// Pixel clock (25 MHz - all signals are stable on front edge)
	output [9:0]			xpos, ypos,								// Current pixel position
	output					ftick										// Frame clock (60 Hz - all signals are stable on front edge)
);



// Source : https://timetoexplore.net/blog/arty-fpga-vga-verilog-01

	localparam
	HPIXELS = 640,
	HA = HPIXELS - 1,									// Hor. active area (0 to 639 = 640 pixels)
	HFP = HA + 16,										// Hor. front porch end position
	HSYNC = HFP + 96,									// Hor. sync end position
	HBP = HSYNC + 48,									// Hor. back porch end position
	
	VPIXELS = 480,
	VA = VPIXELS - 1,									// Vert. active area (0 to 479 = 480 pixels)
	VFP = VA + 11,										// Vert. front porch end position
	VSYNC = VFP + 2,									// Vert. sync end position
	VBP = VSYNC + 31;									// Vert. back porch end position

	// Instead of creating a separate 25-MHz clock domain and violating
	// the "synchronous design methodology", we generate a 25-MHz enable tick
	// to enable or pause the counting.
	
	// The tick is also routed to the p_tick port as an output signal
	// to coordinate operation of the pixel generation circuit.
	
	// Mod-2 counter

	reg		mod2_r;
	wire		mod2_next;
	wire		ptick_w;
	
	// Sync counters

	reg  [9:0]	hcount, hcount_next;
	reg  [9:0]	vcount, vcount_next;
	
	// Sync output buffers
	
	// To remove potential glitches, output buffers are inserted for the hsync and vsync signals. This leads 
	// to a one-clock-cycle delay. We also add a similar buffer for the rgb signal in the pixel 
	// generation circuit to compensate for the delay.
	
	reg			vsync_r, hsync_r;
	wire			vsync_next, hsync_next;
	
	// RGB signal buffer
	
	reg			red_r, green_r, blue_r;

	// Frame output buffered
	
	reg			ftick_r;
	wire			ftick_next;
	
	// Status signals

	wire			h_end, v_end;

	// Body
	
	// Registers

always @ (posedge clk or posedge reset) begin
	if  (reset) begin
		mod2_r <= 1'b0;
		
		vcount <= 0;
		hcount <= 0;
		
		vsync_r <= 1'b0;
		hsync_r <= 1'b0;
		
		red_r <= 1'b0;
		green_r <= 1'b0;
		blue_r <= 1'b0;
		
		ftick_r <= 1'b0;
	end
	else begin
		mod2_r <= mod2_next;
		
		vcount <= vcount_next;
		hcount <= hcount_next;
		
		vsync_r <= vsync_next;
		hsync_r <= hsync_next;
		
		red_r <= pixel_rgb[0];
		green_r <= pixel_rgb[1];
		blue_r <= pixel_rgb[2];
		
		ftick_r <= ftick_next;
	end
end

	// Mod-2 circuit to generate the 25 MHz tick
	
	assign mod2_next = ~mod2_r;
	assign ptick_w = mod2_r;

	// Status  signals

	// End of horizontal line counter (799)

	assign h_end = (hcount == HBP);

	// End of vertical (524)

	assign v_end = (vcount == VBP);

	// Next-state logic of mod-800 horizontal sync counter

	always @(*) begin
		if  (ptick_w)  // 25 MHz pixel tick
			if (h_end)	// End of line ?
				hcount_next = 0;
			else
				hcount_next = hcount + 10'd1;
		else
			hcount_next = hcount;
	end

	// Next-state logic of mod-525 vertical sync counter

	always @(*) begin
		if (ptick_w & h_end)	// 25 MHz pixel tick and end of line
			if (v_end)	// End of screen ?
				vcount_next = 0;
			else
				vcount_next = vcount + 10'd1;
		else
			vcount_next = vcount;
	end

	// Horizontal and vertical sync, and f_tick, buffered to avoid glitches
	
	// hsync_next reset between 656 and 752
	
	assign	hsync_next = ~((hcount > HFP) && (hcount <= HSYNC));

	// vsync_next reset between 491 and 493
	
	assign	vsync_next = ~((vcount > VFP) && (vcount <= VSYNC));
	
	// f_tick_next set when the current position enters the blanking area
	
	assign	ftick_next = (hcount == HPIXELS) && (vcount == VPIXELS);

	// active when the current position is inside the visible area
	
	assign	active = (hcount <= HA) && (vcount <= VA);

	// Outputs

	assign	hsync = hsync_r;
	assign	vsync = vsync_r;
	assign	xpos = hcount;
	assign	ypos = vcount;
	assign	ptick = ptick_w;
	assign	ftick = ftick_r;

	assign	red = active ? red_r : 1'b0;
	assign	green = active ? green_r : 1'b0;
	assign	blue = active ? blue_r : 1'b0;
endmodule
```