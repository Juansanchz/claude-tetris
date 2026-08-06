---
description: Crea un git worktree aislado en .trees/ y ejecuta ahí las instrucciones dadas
argument-hint: <requerimiento o instrucciones a implementar>
---

El usuario invocó `/worktree` con el siguiente requerimiento/instrucciones:

$ARGUMENTS

Sigue estos pasos, en orden:

1. **Determina un nombre corto en kebab-case** (2-4 palabras, sin acentos, minúsculas, separadas por guiones) que resuma el requerimiento anterior. Ese nombre se usará como nombre de carpeta y de rama. Ej: "arreglar el bug del game over" → `fix-game-over`; "agregar modo multijugador" → `add-multiplayer-mode`.

2. Verifica que estás en la raíz de un repo git (`git rev-parse --show-toplevel`). Si `.trees/` no existe aún, créala implícitamente al correr el comando (git la crea sola).

3. Crea el worktree aislado en una rama nueva a partir del HEAD actual:
   ```
   git worktree add .trees/<nombre> -b <nombre>
   ```
   Si ya existe una carpeta o rama con ese nombre, agrega un sufijo numérico (`<nombre>-2`, etc.) en vez de sobreescribir nada.

4. Cambia tu directorio de trabajo a `.trees/<nombre>` para todos los comandos siguientes (usa rutas absolutas o `cd` explícito por comando; no operes por accidente sobre el working tree principal).

5. Dentro de ese worktree, ejecuta de forma completa e independiente las instrucciones del requerimiento indicado arriba — lee el código, implementa, prueba lo que corresponda. No toques el directorio de trabajo principal del repo (fuera de `.trees/<nombre>`) durante esta tarea.

6. No hagas commit ni push automáticamente salvo que el usuario lo pida explícitamente en su requerimiento o lo confirme cuando termines.

7. Al terminar, resume brevemente: qué rama/carpeta se creó (`.trees/<nombre>`, rama `<nombre>`), qué cambios se hicieron ahí, y cómo revisarlos (`git -C .trees/<nombre> diff`) o limpiarlos (`git worktree remove .trees/<nombre>`) si el usuario no los quiere conservar.
