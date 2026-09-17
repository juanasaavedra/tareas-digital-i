```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'background': '#ffffff',
    'primaryColor': '#ffffff',
    'primaryTextColor': '#000000',
    'primaryBorderColor': '#000000',
    'lineColor': '#000000',
    'tertiaryColor': '#ffffff'
  }
}}%%
graph TD
    %% Flechas negras gruesas
    linkStyle default stroke:#000000,stroke-width:3.5px;

    classDef inicio fill:#e3f2fd,stroke:#1565c0,stroke-width:2px,color:#000;
    classDef proceso fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px,color:#000;
    classDef decision fill:#fff8e1,stroke:#f57f17,stroke-width:2px,color:#000;
    classDef io fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000;
    classDef alerta fill:#ffebee,stroke:#c62828,stroke-width:2px,color:#000;

    %% ETAPA 1: VERIFICACIÓN DE HARDWARE
    subgraph E1 [Etapa 1: Verificación del funcionamiento del hardware]
        A(["Encender consola"]):::inicio --> B{{"Iniciar sistema en la FPGA"}}:::proceso
        B --> C["Cargar programa inicial"]:::io
        C --> D["Probar pantalla y mando: Mostrar mensaje 'Press X'"]:::io
        
        D --> E{"¿Se presiona una tecla?"}:::decision
        E -- Sí --> F["Probar sonido: Emitir tono al pulsar tecla"]:::proceso
        
        E -- No --> G{"¿Va por la mitad del tiempo límite?"}:::decision
        G -- Sí --> H["Sonar alarma de tiempo de espera"]:::alerta
        H --> I{"¿Se agotó el tiempo total de espera?"}:::decision
        G -- No --> I
        
        I -- No --> E
        I -- Sí --> J(["Bloquear sistema: No se detectó mando"]):::alerta
        
        F --> K{"¿El sonido funciona correctamente?"}:::decision
        K -- Sí --> L["Activar sonido por defecto"]:::proceso
        K -- No --> M["Desactivar sonido por defecto"]:::proceso
        
        L --> N{"¿Revisar datos guardados?"}:::decision
        M --> N
        
        N -. Opcional .-> O["Leer datos de la memoria"]:::io
        O --> P{"¿Los datos están correctos?"}:::decision
        P -- Sí --> Q["Permitir guardar partidas"]:::proceso
        P -- No --> R["Desactivar guardado de partidas"]:::proceso
        
        Q --> S1(( S )):::inicio
        R --> S1
        N -. Omitir .-> S1
    end

    %% ETAPA 2: INTERFAZ DE MENÚ
    subgraph E2 [Etapa 2: Interfaz del menú principal]
        S1 --> S_Menu["Mostrar Menú Principal: Cuadrícula con 4 juegos (ID 0, 1, 2, 3)"]:::io
        S_Menu --> T["Leer botón del mando"]:::io
        
        T --> U{"¿Qué botón se presionó?"}:::decision
        
        %% Programación detallada de los 4 movimientos de la Cruceta
        U -- Cruceta --> V{"¿Hacia qué dirección?"}:::decision
        V -- Arriba / Abajo --> V1["Mover selector verticalmente entre filas"]:::proceso
        V -- Izquierda / Derecha --> V2["Mover selector horizontalmente entre columnas"]:::proceso
        
        V1 --> V3["Actualizar selección y asignar nuevo juego: ID 0, 1, 2 ó 3"]:::proceso
        V2 --> V3
        
        U -- Botón B --> W["Mostrar tutorial del juego seleccionado"]:::io
        U -- Select --> Y["Ingresar a la configuración del sistema"]:::proceso
        
        U -- Botón A --> X{"¿El sonido está encendido?"}:::decision
        X -- Sí --> X1["Apagar sonido y quitar ícono de parlante"]:::proceso
        X -- No --> X2["Encender sonido y mostrar ícono de parlante"]:::proceso
        
        %% Retorno limpio y unificado
        V3 --> Z1["Actualizar la pantalla del menú"]:::proceso
        W --> Z1
        Y --> Z1
        X1 --> Z1
        X2 --> Z1
        
        Z1 --> T
        
        %% Salida al presionar Start
        U -- Start --> Z2{"¿Está seleccionado el modo Multijugador?"}:::decision
        Z2 -- No --> AA
        Z2 -- Sí --> Z3["Validar conexión de las 4 pantallas para detectar cuáles están activas"]:::proceso
        Z3 --> Z4["Mostrar en pantalla cuáles de las 4 consolas están unidas"]:::io
        Z4 --> AA
    end

    %% ETAPA 3: BUCLE PRINCIPAL DE JUEGO (GAME LOOP)
    subgraph E3 [Etapa 3: Bucle del juego]
        AA{{"Cargar vidas, puntos e imagen del juego seleccionado según su ID"}}:::proceso --> AB
        
        AB["Leer botones durante el juego"]:::io --> AC{"¿Se presiona Select?"}:::decision
        
        AC -- Sí --> AD["Salir del juego actual"]:::proceso
        AD --> S_Exit1(( S )):::inicio
        
        AC -- No --> AE{"¿Se presiona Start?"}:::decision
        AE -- Sí --> AF["Cambiar estado de pausa"]:::proceso
        AF --> AG{"¿El juego está en pausa?"}:::decision
        AE -- No --> AG
        
        AG -- Sí --> AH["Congelar juego y mostrar mensaje de 'Pausa'"]:::io
        AH --> AB
        
        %% Desglose detallado del movimiento y físicas
        AG -- No --> AI1["Calcular posición del personaje según los botones presionados"]:::proceso
        AI1 --> AI2["Mover objetos o pelota automáticamente según su velocidad y dirección"]:::proceso
        AI2 --> AI3{"¿Los objetos o la pelota chocan con los bordes de la pantalla?"}:::decision
        AI3 -- Sí --> AI4["Invertir dirección de movimiento (Rebote)"]:::proceso
        AI3 -- No --> AJ
        AI4 --> AJ
        
        AJ{"¿El personaje choca con un enemigo u obstáculo?"}:::decision
        AJ -- Sí --> AK["Perder 1 vida"]:::alerta
        AK --> AL["Sonar efecto de daño"]:::alerta
        AL --> AM["Regresar personaje a su posición inicial"]:::proceso
        AM --> AN{"¿Se quedaron sin vidas?"}:::decision
        AN -- Sí --> AO["Mostrar pantalla de 'Juego Terminado'"]:::alerta
        AO --> S_Exit2(( S )):::inicio
        AN -- No --> AP["Actualizar la imagen en la pantalla"]:::io
        
        AJ -- No --> AQ{"¿El personaje atrapa un ítem o punto?"}:::decision
        AQ -- Sí --> AR["Sumar puntos al marcador"]:::proceso
        AR --> AS["Sonar efecto de recompensa"]:::proceso
        AS --> AT{"¿Alcanzó la puntuación para ganar el nivel?"}:::decision
        
        AQ -- No --> AT
        
        AT -- Sí --> AU["Mostrar pantalla de '¡Ganaste!'"]:::io
        AU --> S_Exit3(( S )):::inicio
        
        AT -- No --> AP
        AP --> AB
    end

    %% Estilos de contenedores
    style E1 fill:#ffffff,stroke:#cccccc,stroke-width:1px;
    style E2 fill:#ffffff,stroke:#cccccc,stroke-width:1px;
    style E3 fill:#ffffff,stroke:#cccccc,stroke-width:1px;
