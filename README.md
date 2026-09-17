# Tareas - Electrónica Digital I

Repositorio dedicado a la presentación y documentación de las tareas y entregables de la materia **Electrónica Digital I**.

---

## 📁 Contenido del Repositorio

| Entrega / Archivo | Descripción | Acceso Directo |
| :--- | :--- | :---: |
| **Diagrama de Flujo General** | Flujo completo del sistema en FPGA: verificación de hardware, menú de 4 juegos, conexión de 4 pantallas y lógica de juego. | [📌 Abrir `diagrama-de-flujo.md`](./diagrama-de-flujo.md) |

---

## 📌 Detalle de Entregas

### 1. Diagrama de Flujo (`diagrama-de-flujo.md`)
Modelo interactivo estructurado en tres etapas principales:
* **Etapa 1 (Hardware):** Verificación inicial de periféricos, señal de video, test de audio, timeout de seguridad y validación de memoria EEPROM.
* **Etapa 2 (Menú Principal):** Navegación en cuadrícula de 4 juegos (ID 0 al 3), control de audio, ayuda y escaneo/validación de pantallas conectadas en modo multijugador.
* **Etapa 3 (Bucle de Juego):** Bucle principal (*Game Loop*), gestión de pausa, físicas de movimiento, detección de colisiones, vidas y pantallas de victoria/derrota.

> **Nota:** Haz clic en el enlace de la tabla para ver el diagrama renderizado directamente en GitHub.
