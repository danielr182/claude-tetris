# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Tetris en JavaScript vanilla + Canvas 2D. Tres archivos (`index.html`, `style.css`, `game.js`), sin dependencias, sin `package.json`, sin build, sin linter ni tests. El `README.md` (en español) documenta mecánicas, controles y constantes ajustables.

## Ejecutar

```bash
python -m http.server 8000   # luego abrir http://localhost:8000
# o simplemente: start index.html
```

No hay suite de pruebas: los cambios se verifican jugando en el navegador.

## Arquitectura de `game.js`

- **Estado global mutable**: todo el estado de la partida (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `animId`…) vive en variables de módulo declaradas en una sola línea `let` e inicializadas en `init()`. Cualquier variable de estado nueva debe reiniciarse en `init()`, que también es el handler del botón de reinicio.
- **Un mismo índice (1–7) identifica tipo de pieza, valor de celda y color**: `PIECES[i]` contiene el valor `i` en sus celdas, `COLORS[i]` es su color, y `board[r][c]` guarda ese índice (0 = vacío). Al añadir o cambiar piezas hay que mantener alineados `PIECES` y `COLORS` (ambos con `null` en el índice 0), y `randomPiece()` asume exactamente 7 tipos.
- **Piezas como matrices cuadradas**; `rotateCW` produce una matriz nueva y `tryRotate` aplica kicks horizontales `[0, -1, 1, -2, 2]` (no es SRS).
- **`collide(shape, ox, oy)`** es la única comprobación de validez; permite `y < 0` (por encima del tablero) pero no fuera de los laterales ni del fondo. Movimiento, rotación, ghost y drops pasan todos por ella.
- **Ciclo de fijado**: `lockPiece()` → `merge()` → `clearLines()` (actualiza puntuación, nivel y `dropInterval`) → `spawn()`. El game over se detecta en `spawn()` cuando la pieza nueva ya colisiona.
- **Bucle**: `loop` con `requestAnimationFrame`, acumula `dt` en `dropAccum` y baja una fila al superar `dropInterval`; redibuja todo el tablero cada frame. Pausa y game over cancelan el frame vía `animId`; al reanudar se reinicia `lastTime` para evitar un salto de `dt`.
- **HUD**: `updateHUD()` debe llamarse explícitamente tras cambios de puntuación (el loop no lo hace). `drawNext()` solo se llama desde `spawn()`.

## Acoplamientos con HTML/CSS

- El canvas `#board` (300×600) debe medir `COLS × BLOCK` por `ROWS × BLOCK`; si se cambian esas constantes hay que actualizar `index.html`.
- `#next-canvas` (120×120) asume una rejilla de 4×4 celdas de 30 px (`NB` en `drawNext`).
- El overlay (`#overlay`, `#overlay-title`, `#overlay-score`) se reutiliza para PAUSA y GAME OVER, mostrándose/ocultándose con la clase `hidden`. Nota: `togglePause()` no vuelve a ocultar el overlay al reanudar.
- La lista de controles en `index.html` está escrita a mano; si se cambian teclas en el handler `keydown`, actualizarla junto con la tabla del README.
