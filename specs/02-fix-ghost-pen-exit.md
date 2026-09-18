# SPEC 02 — Spawn directo en la celda de la puerta del pen

> **Estado:** Aprovado
> **Depende de:** SPEC 01
> **Fecha:** 2026-09-18
> **Objetivo:** Cuando expira el retardo de un fantasma o al morir, aparece en la celda de la puerta del pen (13, 12), evitando el atascamiento dentro de la jaula.

## Alcance

**Incluye:**

- Cambiar `GHOST_STARTS` en `src/js/maze.js` para que las 4 entradas tengan `(x, y) = (13, 12)`, conservando `kind` y `releaseIn` (0/90/180/270) definidos por SPEC 01.
- Eliminar en `ghostTarget` (en `src/js/game.js`) la rama `// Si aun esta dentro de la pen…`, que ya no aporta casos cuando los fantasmas activos nunca pasan por el interior del pen.
- Mantener `createGame` y `resetPositions` sin cambios: ambos inicializan desde `GHOST_STARTS`, y este arreglo los alinea con la nueva ubicación.

**Fuera de alcance (para futuros specs):**

- Spawns distintos por fantasma (p. ej. celdas separadas a la salida de la pen).
- Cambios en `releaseIn` o `speed` por `kind`.
- Modificaciones a la IA, a `decideGhost`, a la mecánica de colisión, a `render.js` o a `index.html`.
- Movimiento interno del pen (sube/baja mientras esperan) o trayectoria guionizada de salida — siguen descartados.
- Reaparición en una celda exterior más alejada (p. ej. fila 11) — la puerta del pen es la celda designada.

## Modelo de datos

```js
// src/js/maze.js — 4 entradas en la celda de la puerta
const GHOST_STARTS = [
  { x: 13, y: 12, kind: 'chaser',     releaseIn:   0 },
  { x: 13, y: 12, kind: 'ambusher',   releaseIn:  90 },
  { x: 13, y: 12, kind: 'strategist', releaseIn: 180 },
  { x: 13, y: 12, kind: 'flaky',      releaseIn: 270 },
];
```

No se introducen nuevas estructuras; `game.ghosts` (`{ x, y, dir, speed, kind, releaseIn }`) sigue siendo el modelo en memoria desde `createGame`, idéntico al definido en SPEC 01.

## Plan de implementación

1. `src/js/maze.js`: en `GHOST_STARTS`, fijar `x: 13, y: 12` en las 4 entradas (mantener los `kind` y `releaseIn`). Tras este paso, `createGame` ya coloca a los 4 fantasmas en la puerta: el juego es funcional.
2. `src/js/game.js`: en `ghostTarget`, eliminar la rama `// Si aun esta dentro de la pen, apunta a la puerta: ...` (líneas que comprueban `gy >= 13 && gy <= 15 && gx >= 11 && gx <= 16`). Tras este paso, no queda código muerto y todas las decisiones de IA pasan por la elección codiciosa hacia la celda objetivo de la personalidad.

Cada paso es committable por sí solo.

## Criterios de aceptación

- [ ] `grep "dentro de la pen" src/js/game.js` no devuelve coincidencias.
- [ ] Las 4 entradas de `GHOST_STARTS` tienen `x === 13 && y === 12`.
- [ ] Al iniciar partida, los 4 fantasmas salen del mapa por la puerta del pen en orden con ~1,5 s de separación (criterio heredado de SPEC 01).
- [ ] Tras perder una vida, `resetPositions` deja a los 4 fantasmas en `(13, 12)` con sus `releaseIn` originales (0/90/180/270) y sus `speed` originales (chaser 1/9, resto 0.1).
- [ ] El resto de los criterios de SPEC 01 siguen siendo válidos: personalidades diferenciadas, sin giros de 180° salvo callejón, puerta del pen usable, túnel sin bloqueos.

## Decisiones

- **Sí:** spawn directo en la celda de la puerta (13, 12). Resuelve el atascamiento sin recurrir a *trayectoria forzada* ni a *movimiento interno*, ambas explícitamente fuera de alcance en SPEC 01.
- **Sí:** las 4 entradas comparten la misma celda; `releaseIn` marca el orden de salida. Diferenciar celdas complica la sincronización del spawn sin aportar ventaja clara en una primera versión.
- **No:** conservar la rama `if (dentro del pen)` de `ghostTarget` como salvaguarda. Quedaría muerta y ruidosa, y abre la puerta a regresiones si una mecánica futura mete a Pacman dentro del pen.
- **No:** spawns distintos por fantasma o más alejados (fila 11, etc.). Se valora en un spec futuro si la IA de salida se queda corta visualmente.

## Lo que **no** entra en este spec

- Cambios en `releaseIn`, `speed` o las personalidades.
- Cambios en `createGame`, `resetPositions`, `collides`, `update`, `moveGhost`, `decideGhost` (más allá del borrado de la rama indicada).
- Cambios en `src/index.html` o `src/js/render.js`.
- Cualquier animación o efecto visual adicional al spawn.
