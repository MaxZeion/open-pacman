# SPEC 01 — Cuatro fantasmas con personalidades clásicas

> **Estado:** Aprovado
> **Depende de:** — (juego base preexistente, sin spec)
> **Fecha:** 2026-09-15
> **Objetivo:** Cuatro fantasmas con comportamientos clásicos del arcade (perseguidor agresivo, emboscada, estratega y voluble) con liberación escalonada desde la pen.

## Alcance

**Incluye:**

- 4 fantasmas definidos en `GHOST_STARTS` (`src/js/maze.js`), todos dentro de la pen, con `kind`: `chaser`, `ambusher`, `strategist`, `flaky`.
- IA por personalidad basada en "celda objetivo + avance codicioso" (mínima distancia euclidiana al cuadrado), sin retroceso de 180° salvo callejón sin salida (regla actual de `decideGhost`).
- `chaser`: apunta siempre a la celda actual de Pacman y es el único con velocidad 1/9 (~0.111).
- `ambusher`: apunta 4 celdas por delante de Pacman según su `dir`.
- `strategist`: punto 2 celdas por delante de Pacman (`P2`); objetivo = `P2` + (`P2` − posición del `chaser`).
- `flaky`: si la distancia Manhattan a Pacman > 8 celdas apunta a Pacman; si no, a su esquina casa (0, 30).
- Liberación escalonada: retardo por fantasma `releaseIn` = 0, 90, 180, 270 frames (~1,5 s de separación); mientras `releaseIn > 0` el fantasma no se mueve.
- Reset total al perder una vida: posiciones a la pen, `releaseIn` y velocidades restaurados.

**Fuera de alcance (para futuros specs):**

- Ciclos scatter/chase con temporizadores (decidido y retirado en la definición).
- Power-ups y modo asustado/frightened.
- Movimiento dentro de la pen (sube/baja) mientras esperan liberación.
- Trayectoria forzada de salida del pen hacia la puerta.

## Modelo de datos

```js
// src/js/maze.js — 4 entradas (celdas concretas de interior de pen, filas 13-15)
const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'chaser',     releaseIn:   0 },
  { x: 14, y: 14, kind: 'ambusher',   releaseIn:  90 },
  { x: 13, y: 13, kind: 'strategist', releaseIn: 180 },
  { x: 14, y: 15, kind: 'flaky',      releaseIn: 270 },
];

// src/js/game.js — cada fantasma en game.ghosts
// { x, y, dir, speed, kind, releaseIn }
// speed: chaser 1/9, resto GHOST_SPEED = 0.1
```

Convenciones: coordenadas en celdas fraccionarias (igual que Pacman); `releaseIn` en frames de `update()` (≈60 fps).

## Plan de implementación

1. `src/js/maze.js`: ampliar `GHOST_STARTS` a 4 entradas con `kind` y `releaseIn` (verificar celdas transitables del interior de la pen leyendo `MAZE_STR`).
2. `src/js/game.js` `createGame`: copiar `kind`, `releaseIn` y asignar `speed` por `kind` (chaser = 1/9). El juego ya es funcional: los 4 se mueven con la IA actual (2 comportamientos existentes como fallback).
3. `src/js/game.js` `update`/`moveGhost`: decrementar `releaseIn` y saltar el movimiento del fantasma mientras sea > 0. Prueba manual: los 4 salen del pen de uno en uno.
4. `src/js/game.js` `decideGhost`: reescribir como selección de celda objetivo por `kind` (fórmulas del alcance) + elección codiciosa de dirección a mínima distancia euclidiana al cuadrado, con desempate determinista up > left > down > right. Sin retroceso salvo callejón. Prueba manual: los 4 se comportan distinto.
5. `src/js/game.js` `resetPositions`: restaurar posiciones, `dir = 'up'`, `releaseIn` y `speed` desde `GHOST_STARTS`. Prueba manual: tras morir, salida escalonada de nuevo.

## Criterios de aceptación

- [ ] El juego arranca sin errores en consola y se ven 4 fantasmas con los 4 colores de `GHOST_COLORS`.
- [ ] Al iniciar partida, los fantasmas salen de la pen en orden con ~1,5 s de separación.
- [ ] El `chaser` recorre pasillos siguiendo a Pacman de forma directa y visiblemente más rápido que los otros tres.
- [ ] El `ambusher` tiende a anticiparse cortando el camino por delante de Pacman.
- [ ] El `strategist` cambia de objetivo cuando se cruza con la línea del `chaser`.
- [ ] El `flaky` persigue a larga distancia pero se desvía hacia la esquina inferior izquierda cuando Pacman se acerca (<8 celdas).
- [ ] Ningún fantasma hace giro de 180° salvo en callejón sin salida.
- [ ] Al perder una vida, los fantasmas vuelven a la pen y repiten la liberación escalonada.
- [ ] Los fantasmas cruzan la puerta de la pen (código 3) y usan el túnel sin bloquearse.

## Decisiones

- **Sí:** personalidades clásicas del arcade traducidas a "celda objetivo" — unifica la IA y es verificable.
- **Sí:** `chaser` a velocidad 1/9 — mantiene alineación cada 9 frames (regla de `PACMAN_SPEED`/alineación).
- **No:** ciclo scatter/chase temporal — se propuso y el usuario lo retiró; va a spec futuro.
- **No:** liberación por dots comidos — acopla pen y dots; mejor por tiempo.
- **No:** frightened/power-ups, movimiento interno en pen — otros specs.
- **Sí:** reset total de timers al morir — más simple y predecible.
- **Sí:** desempate determinista de direcciones (up > left > down > right) — comportamiento reproducible, como el arcade.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| Velocidad 1/9 con acumulación de punto flotante desalinee | `aligned()` usa tolerancia 1e-3 y `Math.round` al alinear; verificar en navegador el chaser en pasillos largos |
| Celdas de inicio dentro de la pen no transitables | El paso 1 verifica contra `MAZE_STR` antes de continuar |
| IA codiciosa con objetivo inalcanzable (p. ej. (0,30) es muro) oscile | El objetivo solo guía la elección de dirección; el fantasma siempre elige una dirección válida, no se bloquea |

## Lo que **no** entra en este spec

- Modo asustado ni power-ups.
- Ciclos scatter/chase por tiempo.
- Animación de fantasma dentro de la pen.
- Cualquier cambio en `src/js/render.js` (ya soporta 4 colores) o en `src/index.html`.
