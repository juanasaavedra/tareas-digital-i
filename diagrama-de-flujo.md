```mermaid
graph TD
    %% Etapa 1: Verificación del funcionamiento del hardware
    A([ Power on ]) --> B{{ Energización de la fpga con inicio en 0x00000000 }}
    B --> C[/ Lectura del firmware de arranque /]
    C --> D[\ Verificación del módulo de video: muestra 'press x' y timer de timeout /]
    
    D --> E{ ¿Transcurre la porción del timeout sin pulsación? }
    E -- Sí --> F[ Se activa alerta de audio de timeout ]
    F --> G[/ Se detecta la pulsación de la tecla /]
    E -- No --> G
    
    G --> H[ Verificación del módulo de audio: al presionar una tecla se reproduce sonido ]
    H --> I{ ¿El firmware evalúa que se está enviando sonido? }
    
    I -- Sí --> J[ Se establece Audio = 1 ]
    I -- No --> K[ Se establece Audio = 0 y juega normal ]
    
    J --> L{ ¿Se realiza la validación de información en memoria? }
    K --> L
    
    %% Ruta opcional mediante líneas punteadas
    L -. Opcional .-> M[/ Lectura del magic byte en la eeprom /]
    M --> N{ ¿La estructura de datos está bien? }
    N -- Sí --> O[ Save_ok = 1: estructura bien /]
    N -- No --> P[ Save_ok = 0: modo no guardar /]
    
    O --> Q
    P --> Q
    L -. Saltar proceso .-> Q

    %% Etapa 2: Interfaz de menú
    Q[\ Dibujar menú principal: muestra la lista con los 4 juegos, dibuja 4 íconos e inicia en ID = 0 /]
    Q --> R[/ Leer entrada del mando /]
    
    R --> S{ ¿Qué comando se detecta? }
    
    S -- Cruceta --> T[ Cambiar fila o columna y mover marco selector ]
    T --> R
    
    S -- Botón B --> U[\ Mostrar pantalla de tutorial /]
    U --> R
    
    S -- Botón A --> V{ ¿Configuración de audio? }
    V --> R
    
    S -- Select --> W[ Ingresar a configuración ]
    W --> R
    
    S -- Start --> X{ ¿Evaluar jugadores y validar otras pantallas? }

    %% Inicializar el videojuego
    X -- Confirmar inicio --> Y{{ Reinicio de variables: puntaje y vidas, asignando coordenadas iniciales }}

    %% Game loop
    Y --> Z[/ Lectura de entradas en el juego /]
    
    Z --> AA{ Select: ¿romper el juego? }
    AA -- Sí --> AB[ Salir al menú principal ]
    AB --> Q
    
    AA -- No --> AC{ Start: ¿pausar el juego? }
    AC -- Sí --> AD[ Conmutar estado de pausa ]
    AD --> AE{ ¿El juego se encuentra en pausa? }
    AC -- No --> AE
    
    AE -- Sí --> AF[\ Congelar pantalla y mostrar texto de pausa /]
    AF --> Z
    
    AE -- No --> AG[ Actualización de posición: coordenadas de jugador, obstáculo y enemigo ]
    
    %% Detector de colisiones y lógica de perder
    AG --> AH{ Jugador vs enemigo u obstáculo: ¿coinciden coordenadas? }
    AH -- Sí --> AI[ Resta una vida ]
    AI --> AJ[ Emite tono de daño ]
    AJ --> AK[ Reinicia coordenadas x,y del jugador ]
    AK --> AL{ ¿Cantidad de vidas == 0? }
    AL -- Sí --> AM[\ Lógica de perder: despliega pantalla de juego terminado /]
    AM --> Q
    AL -- No --> AN[\ Renderizar fotograma en la matriz /]
    
    %% Colisión con ítem y lógica de ganar
    AH -- No --> AO{ Jugador vs ítem o punto: ¿coinciden coordenadas? }
    AO -- Sí --> AP[ Incremento de puntaje ]
    AP --> AQ[ Reproduce sonido de recompensa ]
    AQ --> AR{ ¿Se cumple la condición para ganar el nivel? }
    
    AO -- No --> AR
    
    AR -- Sí --> AS[\ Lógica de ganar: despliega pantalla de victoria /]
    AS --> Q
    AR -- No --> AN
    
    AN --> Z
