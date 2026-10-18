<div align="center">

  <img src="https://github.com/synapse-racing.png" alt="Synapse Racing" width="110" style="border-radius: 8px;" />

  # Synapse Racing

  **Plataforma web de carreras 3D con conducción autónoma (NEAT) y multijugador en tiempo real**

  <br/>

  [![Proyecto Universitario](https://img.shields.io/badge/Proyecto-Universitario-475569?style=flat-square)](https://github.com/synapse-racing)
  [![Estado](https://img.shields.io/badge/Estado-En%20Desarrollo-2563eb?style=flat-square)](https://github.com/synapse-racing)
  [![Licencia](https://img.shields.io/badge/Licencia-MIT-0f172a?style=flat-square)](https://github.com/synapse-racing)

  <br/>

  <p>
    <img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white" alt="TypeScript" />
    <img src="https://img.shields.io/badge/React_19-222222?style=flat-square&logo=react&logoColor=61DAFB" alt="React" />
    <img src="https://img.shields.io/badge/Three.js-000000?style=flat-square&logo=three.js&logoColor=white" alt="Three.js" />
    <img src="https://img.shields.io/badge/Rapier_Physics-262626?style=flat-square" alt="Rapier Physics" />
    <img src="https://img.shields.io/badge/NestJS-E0234E?style=flat-square&logo=nestjs&logoColor=white" alt="NestJS" />
    <img src="https://img.shields.io/badge/Socket.IO-010101?style=flat-square&logo=socket.io&logoColor=white" alt="Socket.IO" />
    <img src="https://img.shields.io/badge/PostgreSQL-336791?style=flat-square&logo=postgresql&logoColor=white" alt="PostgreSQL" />
    <img src="https://img.shields.io/badge/Prisma-2D3748?style=flat-square&logo=prisma&logoColor=white" alt="Prisma" />
    <img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white" alt="Docker" />
  </p>

</div>

---

## Sobre el Proyecto

**Synapse Racing** es un proyecto desarrollado en el marco académico universitario para la materia de desarrollo de aplicaciones web. Como equipo decidimos plantear el trabajo práctico alrededor del desarrollo de un **videojuego de carreras en 3D** directamente ejecutable desde el navegador web, integrando simulación física, entrenamiento de agentes autónomos con algoritmos evolutivos y juego multijugador en red.

El juego propone dos modalidades principales:

1. **Modo Entrenamiento (Inteligencia Artificial):** Los autos aprenden a completar el circuito de forma autónoma mediante el algoritmo **NEAT** (*NeuroEvolution of Augmenting Topologies*). La simulación corre en el navegador y permite ver la evolución de las redes neuronales a través de las distintas generaciones.
2. **Modo Multijugador:** Creación de salas privadas para competir en tiempo real contra otros jugadores con sincronización vía WebSockets.

---

## ¿Cómo Funciona la IA en el Juego?

Los vehículos autónomos no siguen una línea predefinida en el circuito, sino que toman decisiones en base a sensores virtuales:

```
                  [ -30° Raycast ]    [ 0° Raycast ]    [ +30° Raycast ]
                          \                  |                  /
       [ -60° Raycast ] ---\                 |                 /--- [ +60° Raycast ]
                             \               |               /
                            ┌────────────────────────────────┐
                            │        Auto Autónomo           │
                            └────────────────┬───────────────┘
                                             │
                                             ▼
                            ┌────────────────────────────────┐
                            │     Red Neuronal (NEAT)        │
                            │ Topología y pesos evolutivos   │
                            └────────────────┬───────────────┘
                                             │
                            ┌────────────────┴───────────────┐
                            ▼                                ▼
                    Dirección [-1, 1]              Acelerador / Freno [-1, 1]
```

- **Entradas:** 5 sensores de distancia por raycast ($-60^\circ, -30^\circ, 0^\circ, +30^\circ, +60^\circ$) y la velocidad longitudinal del vehículo.
- **Salidas:** Dos valores continuos que controlan el ángulo de giro y la aceleración o frenado.
- **Cálculo de Rendimiento (Fitness):** Cada generación evalúa a los autos sumando puntos por distancia recorrida y checkpoints completados, premiando la velocidad media y penalizando choques contra muros o quedarse inmóvil.

---

## Arquitectura de la Aplicación

La solución está construida desacoplando el cliente web interactivo del servidor de juego y persistencia:

```mermaid
flowchart LR
    subgraph Frontend["Frontend (Cliente Web)"]
        direction TB
        UI["Interfaz Web<br/>React 19 / Zustand"]
        Game["Juego y Físicas 3D<br/>Three.js / Rapier (60 FPS)"]
        NEAT["Cálculo Evolutivo NEAT<br/>Web Workers"]
        UI --- Game
        Game --- NEAT
    end

    subgraph Backend["Backend (NestJS)"]
        direction TB
        Auth["Autenticación y Sesiones<br/>JWT / Argon2id"]
        Multiplayer["Salas y Multijugador<br/>Socket.IO WebSockets"]
        Persistence["Lógica de Negocio<br/>Prisma ORM"]
    end

    subgraph Database["Base de Datos"]
        DB[("MongoDB")]
    end

    UI -->|API REST| Auth
    Game <-->|WebSockets Tiempo Real| Multiplayer
    Auth --> Persistence
    Multiplayer --> Persistence
    Persistence --> DB
```
