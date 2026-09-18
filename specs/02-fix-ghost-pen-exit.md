# SPEC 02 — Salida estable del pen (celdas dispersas + fallback)

> **Estado:** Aprovado
> **Depende de:** SPEC 01
> **Fecha:** 2026-09-18
> **Objetivo:** Cada fantasma aparece en una celda interior del pen (no apilados) y, mientras siga dentro de esa caja, su objetivo es la puerta (13, 12); al cruzar el umbral vuelve a su personalidad.

## Alcance

**Incluye:**

- `GHOST_STARTS` en `src/js/maze.js` con 4 entradas en celdas interiores distintas del pen, todas en la fila 14 (la fila central del pen, no en el pasillo por encima): `(12, 14), (13, 14), (14, 14), (15, 14)`. Mantener `kind` y `releaseIn` (0/90/180/270) definidos por SPEC 01.
- Restaurar en `ghostTarget` (`src/js/game.js`) la rama "si el fantasma está dentro del pen interior (filas 13–15, cols 11–16) → objetivo = puerta (13, 12)". Es un fallback indispensable: sin él, la elección codiciosa oscila en el fondo del pen porque su destino (Pacman) suele estar más abajo y el fondo es muro.
- `createGame`/`resetPositions` sin cambios: ya copian desde `GHOST_STARTS` y `resetPositions` restaura `releaseIn` + `speed`.

**Fuera de alcance (futuros specs):**

- Spawns en celdas exteriores (fila 11, por encima del pen) — probados visualmente y descartados por el usuario porque no quedan centrados.
- Spawns individuales en fila 13 o fila 15, o distribuciones 2×2 — descartados por simplicidad.
- Cambios en `releaseIn`, `speed`, personalidades, `decideGhost` (más allá del fallback), `render.js`, `index.html`.
- Movimiento guionizado dentro del pen (sube/baja) o trayectoria forzada de salida — siguen descartados.
- Liberación por dots comidos o por temporizadores de chase/scatter.

## Modelo de datos

```js
// src/js/maze.js — 4 entradas en el interior del pen, dispersas horizontalmente
const GHOST_STARTS = [
  { x: 12, y: 14, kind: 'chaser',     releaseIn:   0 },
  { x: 13, y: 14, kind: 'ambusher',   releaseIn:  90 },
  { x: 14, y: 14, kind: 'strategist', releaseIn: 180 },
  { x: 15, y: 14, kind: 'flaky',      releaseIn: 270 },
];
```

```js
// src/js/game.js, dentro de ghostTarget, al inicio de la función
// Mientras el fantasma esté dentro del pen interior, su objetivo es la puerta.
// Sin esto, la elección codiciosa oscila en el fondo: el destino (Pacman)
// casi siempre está más abajo y el fondo es muro.
const gx = Math.round( g.x );
const gy = Math.round( g.y );
if ( gy >= 13 && gy <= 15 && gx >= 11 && gx <= 16 ) return { x: 13, y: 12 };
```

Sin nuevas estructuras: `game.ghosts` (`{ x, y, dir, speed, kind, releaseIn }`) sigue idéntico.

## Plan de implementación

1. `specs/02-fix-ghost-pen-exit.md`: reescribir el spec a esta versión (dispersas + fallback). Reflejar el diseño real que se va a implementar.
2. `src/js/game.js`, en `ghostTarget`: restaurar la rama "dentro del pen" arriba del todo. Tras este paso, `createGame`/`update`/`moveGhost`/`decideGhost` ya funcionan: los fantasmas eligen automáticamente la puerta mientras estén en el pen y, al cruzar, vuelven a la personalidad.

Cada paso es committable por sí solo. `src/js/maze.js` no requiere cambios (las celdas `(12, 14)–(15, 14)` ya están aplicadas por edición previa).

## Criterios de aceptación

- [ ] Las 4 entradas de `GHOST_STARTS` tienen `x ∈ {12, 13, 14, 15}` y `y === 14`.
- [ ] `releaseIn` sigue siendo `[0, 90, 180, 270]` en `createGame()`.
- [ ] `grep "dentro de la pen" src/js/game.js` devuelve al menos 1 coincidencia.
- [ ] En una simulación de ~6000 frames con Pacman en movimiento, ningún fantasma queda en el pen interior de forma permanente (los frames en pen ≪ frames totales).
- [ ] Los fantasmas salen por la puerta en orden con ~1,5 s de separación.
- [ ] Tras perder una vida, `resetPositions` devuelve a los 4 a `(12–15, 14)` con `releaseIn` y `speed` originales.
- [ ] El resto de los criterios de SPEC 01 siguen siendo válidos: personalidades diferenciadas, sin giros de 180° salvo callejón, puerta del pen usable, túnel sin bloqueos.

## Decisiones

- **Sí:** celdas internas dispersas (fila 14, cols 12–15). Visualmente centradas en el pen y separadas entre sí; evita el apilamiento en `(13,12)` que se probó antes.
- **Sí:** restaurar la rama `if (dentro del pen) target = puerta`. Es la única forma de evitar el atascamiento con celdas internas sin guionizar el movimiento (descartado).
- **No:** spawn exterior (fila 11). El usuario lo descartó visualmente por "quedar horrible y no centrado en el pen".
- **No:** distribución 2×2 dentro del pen. Más dispersión complica la sincronización visual sin un beneficio claro.
- **No:** cambios en `releaseIn`/`speed`/personalidades. La personalización actual de SPEC 01 se mantiene tal cual.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| El fallback se queda obsoleto si más adelante movemos los starts al exterior | El test de aceptación deja un `grep` que localiza la rama; mientras esté activa, basta con borrarla. |
| Conflicto con la rama "dentro del pen" en otras mecánicas futuras (entrar con Pacman en pen, frightened, etc.) | Es esperable: una mecánica que meta a Pacman al pen necesitará revisar la regla. Se documenta aquí. |

## Lo que **no** entra en este spec

- Cambios en `releaseIn`, `speed`, las personalidades o el orden de salida escalonado.
- Cambios en `createGame`, `resetPositions`, `collides`, `update`, `moveGhost`, `decideGhost` (más allá del bloque indicado).
- Cambios en `src/index.html` o `src/js/render.js`.
- Spawns exteriores, distribuciones 2×2, spawn por dots comidos o temporizadores de chase/scatter.
- Cualquier animación adicional al spawn.
