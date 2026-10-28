# CLAUDE.md

App web didáctica de árboles binarios de búsqueda con tres módulos: **Simulador** animado, **Teoría** con glosario dinámico y **Quiz** de recorridos. El plan de implementación, tarea por tarea, está en `2026-10-27-arboles-binarios.md`. Es la fuente de verdad: antes de empezar una tarea, léela completa ahí y marca sus casillas (`- [x]`) al terminar.

## Comandos

```bash
npm run dev       # servidor de desarrollo en http://localhost:5173
npm run build     # build de producción; debe terminar con "✓ built in …"
npm run preview   # sirve el build
```

No hay tests automatizados. Cada tarea trae su propia comprobación: un comando `node --input-type=module -e "…"` que se corre en Git Bash desde la raíz, o una lista de revisión en el navegador. Ninguna se guarda en el repo.

## Arquitectura

- `src/logica/`: el motor (`Nodo`, `ArbolBinario`, `colores`, `formato`). Es JavaScript puro y **no importa nada de React ni del DOM**. Cada operación modifica el árbol y devuelve `{ pasos, exito, mensaje }`.
- `src/hooks/estadoVisual.js`: la función pura `derivarEstadoVisual(pasos, indice, antes, despues)`.
- `src/hooks/useReproductor.js`: reproduce los pasos (reproducir, pausar, paso a paso, velocidad).
- `src/componentes/VistaArbol/`: dibuja la instantánea del árbol en SVG con Motion.
- `src/modulos/{simulador,teoria,quiz}/`: las tres páginas. Las rutas son `/simulador`, `/teoria` y `/quiz`.

Las formas exactas de los pasos (`comparar`, `encontrado`, `no-encontrado`, `cambio-estructura`, `error`, `caso-eliminacion`, `sucesor`, `llamar`, `visitar`, `retornar`) y de la instantánea están en las Tareas 4 a 6 del plan.

## Reglas obligatorias

- **Lenguaje:** JavaScript con módulos ES (`"type": "module"`), sin TypeScript.
- **Dependencias:** solo `react`, `react-dom`, `react-router` y `motion`; en desarrollo, `vite` y `@vitejs/plugin-react`. Para agregar cualquier otra hay que **preguntar al usuario**.
- **Sin Vitest ni archivos de prueba** en el repo.
- **Cero caja negra:**
  - En `src/logica/`, el árbol existe solo como objetos `Nodo` enlazados por `izquierdo` y `derecho`.
  - Los recorridos son recursivos, sin pila propia.
  - Está prohibido representar el árbol con arreglos, `Map`, `Set` o librerías. Los arreglos solo sirven para guardar pasos y resultados.
- **Idioma:** código e interfaz en español, con los identificadores del plan (`ArbolBinario`, `Nodo`, `insertar`, `izquierdo`…).
- **Color:** el tono de cada nodo es `(id × 137.508) mod 360`. El color de un nodo no cambia nunca, ni cuando sube como sucesor. El estado del nodo se dibuja como un anillo exterior, nunca cambiando el relleno.
- **Eliminar con dos hijos:** el sucesor inorden se mueve físicamente. Nunca se copia su valor a otro nodo.
- **Convenciones:** la raíz está en el nivel 0, la profundidad es igual al nivel, una hoja tiene altura 0 y el árbol vacío tiene altura −1.
- **Ids:** `#siguienteId` empieza en 1 y no se reinicia con `limpiar()`.
- **Animaciones:**
  - Las micro-animaciones duran menos de 300 ms y los movimientos del árbol 400 ms.
  - Se respeta `prefers-reduced-motion` mediante `MotionConfig reducedMotion="user"`.
- **Dispositivos:** escritorio y tableta, no teléfono.
- **Sin persistencia:** ni backend ni `localStorage`.

## Git

- Se hace **un commit por tarea**, con exactamente el mensaje que indica el plan (p. ej. `feat: Nodo, colores y lectura de valores`).
- **Sin `Co-Authored-By`** ni ningún otro contribuidor.
- **No hacer `git push` automáticamente.** Solo se sube a `origin/main` (https://github.com/manuelgordillo10/binary-tree) cuando el usuario lo pide explícitamente.

## Puntos de revisión del usuario

- **Tarea 2:** los bocetos los aprueba el usuario. El Bloque 3 (navegación) no empieza sin esa aprobación. Mientras tanto se puede avanzar con el Bloque 2 (motor).
- **Tarea 16, paso 6:** el usuario revisa los textos de la teoría.

## Pendiente

El plan cita el spec `docs/superpowers/specs/2026-10-27-arboles-binarios-design.md`, pero todavía no está en el repo.
