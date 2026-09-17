```mermaid
graph TD
    %% Etapa 1: Verificación del funcionamiento del hardware
    subgraph Etapa 1: Verificación del funcionamiento del hardware
        A([ Power on ]) --> B{{ Energización de la fpga con inicio en 0x00000000 }}
        B --> C[/ Lectura del firmware de arranque /]
        C --> D[\ Verificación del módulo de video y periféricos PS2/NES/mouse: muestra 'press x' /]
        
        D --> E{ ¿Se detecta la pulsación de la tecla? }
        E -- Sí --> F[ Verificación del módulo de audio: al presionar una tecla se reproduce sonido ]
        
        E -- No --> G{ ¿Se alcanza la mitad del timer de timeout? }
        G -- Sí --> H[ Emite alerta de audio de timeout ]
        H --> I{ ¿Expira el tiempo límite total sin tecla? }
        G -- No --> I
        
        I -- No --> E
        I -- Sí --> J([ Bloqueo del sistema: detiene el arranque por falta de entrada ])
        
        F --> K{ ¿El firmware evalúa que se está enviando sonido? }
        K -- Sí --> L[ Se establece Audio = 1 ]
        K -- No --> M[ Se establece Audio = 0 y juega normal ]
        
        L --> N{ ¿Se realiza la validación de información en memoria? }
        M --> N
        
        N -. Opcional .-> O[/ Lectura del magic byte en la eeprom /]
        O --> P{ ¿La estructura de datos está bien? }
        P -- Sí --> Q[ Save_ok = 1: estructura bien ]
        P -- No --> R[ Save_ok = 0: modo no guardar ]
        
        Q --> S
        R --> S
        N -. Saltar proceso .-> S
    end

    %% Etapa 2: Interfaz de menú
    subgraph Etapa 2: Interfaz de menú
        S[\ Dibujar menú principal: muestra la lista con los 4 juegos, dibuja 4 íconos e inicia en ID = 0 /]
        S --> T[/ Leer entrada del mando /]
        
        T --> U{ ¿Qué comando se detecta? }
        
        U -- Cruceta --> V[ Cambiar fila o columna y mover marco selector ]
        U -- Botón B --> W[\ Mostrar pantalla de tutorial /]
        U -- Select --> Y[ Ingresar a configuración del sistema ]
        
        U -- Botón A --> X{ ¿El audio se encuentra activo Audio == 1? }
        X -- Sí --> X1[ Desactivar sonido: Audio = 0 ]
        X -- No --> X2[ Activar sonido: Audio = 1 ]
        
        %% Nodo de retorno unificado para evitar cruce de líneas
        V --> Z1[ Actualizar interfaz visual y esperar entrada ]
        W --> Z1
        Y --> Z1
        X1 --> Z1
        X2 --> Z1
        Z1 --> T
        
        %% Evaluación limpia para Start
        U -- Start --> Z2{ ¿Se seleccionó modo multijugador? }
        Z2 -- No --> AA
        Z2 -- Sí --> Z3[ Ejecutar protocolo de comunicación entre pantallas ]
        Z3 --> Z4[\ Desplegar visualmente en la matriz las pantallas conectadas /]
        Z4 --> AA
    end

    %% Etapa 3: Inicializar y bucle de juego
    subgraph Etapa 3: Inicializar y bucle de juego
        AA{{ Reinicio de variables: puntaje y vidas, asignando coordenadas iniciales }} --> AB[/ Lectura de entradas en el juego /]
        
        AB --> AC{ Select: ¿romper el juego? }
        AC -- Sí --> AD[ Salir al menú principal ]
        AD --> S
        
        AC -- No --> AE{ Start: ¿pausar el juego? }
        AE -- Sí --> AF[ Conmutar estado de pausa ]
        AF --> AG{ ¿El juego se encuentra en pausa? }
        AE -- No --> AG
        
        AG -- Sí --> AH[\ Congelar pantalla y mostrar texto de pausa /]
        AH --> AB
        
        AG -- No --> AI[ Actualización de posición: coordenadas de jugador, obstáculo y enemigo ]
        
        AI --> AJ{ Jugador vs enemigo u obstáculo: ¿coinciden coordenadas? }
        AJ -- Sí --> AK[ Resta una vida ]
        AK --> AL[ Emite tono de daño ]
        AL --> AM[ Reinicia coordenadas x,y del jugador ]
        AM --> AN{ ¿Cantidad de vidas == 0? }
        AN -- Sí --> AO[\ Lógica de perder: despliega pantalla de juego terminado /]
        AO --> S
        AN -- No --> AP[\ Renderizar fotograma en la matriz /]
        
        AJ -- No --> AQ{ Jugador vs ítem o punto: ¿coinciden coordenadas? }
        AQ -- Sí --> AR[ Incremento de puntaje ]
        AR --> AS[ Reproduce sonido de recompensa ]
        AS --> AT{ ¿Se cumple la condición para ganar el nivel? }
        
        AQ -- No --> AT
        
        AT -- Sí --> AU[\ Lógica de ganar: despliega pantalla de victoria /]
        AU --> S
        AT -- No --> AP
        
        AP --> AB
    end
