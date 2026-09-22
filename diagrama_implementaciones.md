flowchart TD
    subgraph S1 ["1. Solicitud de Entrada (Polling por CPU)"]
        START([Inicio del Ciclo / Bucle del Juego]) --> CPU_REQ["CPU (RV32I / femtorv32) emite lectura rd=1
        en la dirección 0x450000 (mem_addr)"]
    end

    subgraph S2 ["2. Decodificación y Bus de Hardware (SoC)"]
        CPU_REQ --> DEC["Address Decoder decodifica 0x450000
        y activa la señal cs6"]
        DEC -->|Línea cs6| NES["Módulo NES (nes_controller.v / NES_CTRL)"]
        DEC -->|Línea cs6| MUX["Multiplexor (MUX)"]
        
        NES -->|Envía datos del mando en d_out| MUX
        MUX -->|Entrega bus mem_rdata| CPU_READ["CPU lee los bits de los botones
        (Cruceta, A, B, Select, Start)"]
    end

    subgraph S3 ["3. Procesamiento de Lógica del Juego"]
        CPU_READ --> EVAL{"¿Qué botón está activo?"}
        
        EVAL -->|Cruceta / Dirección| MOVE["Calcular nuevas coordenadas X/Y del jugador"]
        EVAL -->|Botón Acción A / B| ACTION["Ejecutar acción (Salto / Disparo / Ataque)"]
        EVAL -->|Sin presionar| IDLE["Mantener estado inercial o reposo"]
        
        MOVE --> RAM_WRITE["CPU actualiza estado del juego en BRAM
        (bram.v @ 0x000000 - 0x3FFFFF)"]
        ACTION --> RAM_WRITE
        IDLE --> RAM_WRITE
    end

    subgraph S4 ["4. Actualización de Salidas y Periféricos"]
        RAM_WRITE --> DISP_WRITE["CPU actualiza el Framebuffer
        en display_driver.v (@ 0x480000)"]
        
        ACTION --> AUDIO_WRITE["CPU envía comandos de sonido
        al módulo de audio i2s_tx.v (@ 0x470000)"]
        
        DISP_WRITE --> VSYNC([Esperar VSync / Siguiente Frame])
        AUDIO_WRITE --> VSYNC
    end

    VSYNC --> START
