# Runner Chase

Implementación del juego **Runner Chase** como entorno multiagente, desarrollada para el proyecto PRAISE del grupo de investigación de la UTN – Facultad Regional Concepción del Uruguay (UTN FRCU).

El juego está construido sobre el framework de agentes provisto por los profesores (agentes, sensores, actuadores, entornos simulados y state buffers), y enfrenta a dos roles — **Captor** y **Fugitivo** — en una pista compartida, con bots o un jugador humano (agente humano pendiente de revisión), en consola o con interfaz gráfica (PyGame).

## Tabla de contenidos
- [Reglas del juego](#reglas-del-juego)
- [Instalación](#️-instalación)
- [Cómo ejecutarlo](#️-cómo-ejecutarlo)
- [Comportamiento del bot por defecto](#comportamiento-del-bot-por-defecto)
- [Estructura del repositorio](#-estructura-del-repositorio)
- [Créditos](#-créditos)

## Reglas del juego

### Participantes
- **Fugitivo** (`Role.CRIMINAL`): corre en una fila fija de la pista.
- **Captor** (`Role.PLAYER`): corre en una fila que se acerca o se aleja de la del Fugitivo según cómo va la persecución.

### La pista
- Grilla de 8 filas × 3 columnas (3 carriles).
- Un obstáculo a la vez cae desde la fila más alta hacia la más baja, a razón de 2 filas por segundo (una fila cada 0.5s, ciclo completo de ~4s por obstáculo).
- Tipos de obstáculo y la acción que hay que ejecutar contra cada uno:

| Obstáculo | Acción correcta | Qué representa |
|---|---|---|
| `none` | `run` | Carril libre |
| `block` | `jump` | Pared completa, hay que saltarla |
| `low_bar` | `slide` | Barra baja, hay que deslizarse |
| `ledge_left` | `go_left` | Carriles derecho/centro bloqueados |
| `ledge_right` | `go_right` | Carriles izquierdo/centro bloqueados |

### Turnos y errores
- En cada ciclo, cada agente elige una acción (`run`, `jump`, `slide`, `go_left`, `go_right`).
- Si la acción no coincide con la correcta para el obstáculo del momento, cuenta como **error**.
- Cada error mueve la distancia entre los dos personajes: un error del Captor **aumenta** la distancia (el Fugitivo se aleja), un error del Fugitivo la **reduce** (el Captor se acerca).

### Dificultad creciente
- Cada agente tiene una probabilidad base de equivocarse (`mistake_rate`).
- Esa probabilidad sube con el tiempo real de partida: cada pocos segundos (`MISTAKE_RATE_INTERVAL`) se suma un incremento, hasta un tope máximo — así ninguna partida se estanca para siempre.

### Fin del juego y ganador
- Si la distancia llega a 0 (o menos), el Captor atrapó al Fugitivo → **gana el Captor**.
- Si la distancia llega al máximo (`MAX_DISTANCE`), el Fugitivo se escapó → **gana el Fugitivo**.

## ⚙️ Instalación

**Requisitos**
- Python 3.10 o superior (se usa la sintaxis `str | None`)
- PyGame (solo para la interfaz gráfica)

```
git clone <URL-DEL-REPOSITORIO>
cd <NOMBRE-DEL-REPOSITORIO>

pip install pygame
```

## ▶️ Cómo ejecutarlo

```
python main_runner.py [opciones]
```

| Opción | Descripción | Por defecto |
|---|---|---|
| `--console` | Corre en modo consola en vez de PyGame | desactivado (PyGame) |
| `--stats` | Muestra las estadísticas acumuladas entre partidas y termina | — |
| `--reset` | Reinicia las estadísticas acumuladas y termina | — |

**Ejemplos**
```
# Partida en PyGame (modo por defecto)
python main_runner.py

# Partida en consola
python main_runner.py --console

# Ver estadísticas acumuladas (partidas, atrapadas, escapes)
python main_runner.py --stats

# Reiniciar las estadísticas
python main_runner.py --reset
```

**Controles** (para el Captor, cuando lo controla una persona): `W` saltar, `S` deslizar, `A` carril izquierdo, `D` carril derecho, vacío/Enter avanza el tiempo sin actuar. Funcionan igual tipeando en consola o apretando la tecla en la ventana de PyGame.

> Hoy, para jugar vos como Captor en vez del bot, hay que reemplazar en `main_runner.py` la línea `player = CaptorAgent(env, base_mistake_rate=0.10)` por `player = PlayerAgent(env)` (está comentada al lado, lista para descomentar) — todavía no es un flag de línea de comandos.

## Comportamiento del bot por defecto

`CriminalAgent` (Fugitivo) y `CaptorAgent` (Captor) juegan la misma política probabilística:
- Calculan cuál es la acción correcta para el obstáculo que tienen encima.
- Con una probabilidad (`mistake_rate`, creciente con el tiempo) ignoran esa acción y eligen una incorrecta al azar entre un par plausible (por ejemplo, frente a un `block` se pueden confundir y hacer `run` o `slide` en vez de `jump`).
- `CaptorAgent` arranca con una tasa de error más baja (0.10 contra 0.15 del Fugitivo) pero esa tasa sube el doble de rápido con el tiempo — así ninguno de los dos domina la partida solo por el valor inicial.

Estos bots son un punto de partida: están pensados para ser reemplazados por agentes más inteligentes.

## 📁 Estructura del repositorio

```
.
├── main_runner.py        # Punto de entrada: arma el entorno, los agentes y corre los hilos
├── runnerworld.py         # Entorno: grilla, obstáculos, distancia, turnos y condición de victoria
├── runneragents.py        # Agentes (CriminalAgent, CaptorAgent, PlayerAgent), sensores y actuador
├── runnerrenderers.py     # Renderers de consola y PyGame
├── runnerbuffer.py        # RunnerStateBuffer (implementa IStateBuffer)
├── Runnerstats.py         # Estadísticas persistentes entre sesiones (stats.json)
├── agents.py               # Framework base: Agent, Sensor, Actuator
├── environments.py         # Framework base: SimulatedEnvironment
├── renderers.py             # Interfaz IRenderer
├── statebuffer.py           # Interfaz IStateBuffer
└── Vacuum_version/          # Ejemplo de referencia (Vacuum World) de la cátedra
```

Los archivos `agents.py`, `environments.py`, `renderers.py`, `statebuffer.py` y la carpeta `Vacuum_version/` provienen del material provisto por los profesores a cargo del grupo de investigación GIICOS.

## 🎓 Créditos

Proyecto desarrollado en el marco de PRAISE, grupo de investigación GIICOS de la UTN FRCU (Universidad Tecnológica Nacional – Facultad Regional Concepción del Uruguay).

- Framework de agentes y entornos: cátedra / profesores a cargo del grupo.
- Implementación del juego Runner Chase: Sarlinga Matias.
