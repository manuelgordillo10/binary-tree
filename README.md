# Árboles Binarios

App web didáctica para aprender **árboles binarios de búsqueda**. Todo funciona sobre un motor propio, `ArbolBinario`, escrito desde cero con nodos enlazados y recursión, sin librerías de estructuras de datos.

## Módulos

- **Simulador:** inserta, busca y elimina valores, que pueden ser números enteros o caracteres. Cada operación se reproduce paso a paso: cada comparación, el camino recorrido y el sucesor inorden al eliminar un nodo con dos hijos. También ejecuta los recorridos **preorden**, **inorden** y **postorden**, mostrando la pila de llamadas recursivas y la salida. La reproducción se puede pausar, avanzar paso a paso, saltar al final y ver a tres velocidades. La vista del árbol admite zoom, arrastre y centrado.
- **Teoría:** tarjetas con los conceptos clave: raíz, hijo, padre, hoja, hermanos, subárbol, altura, profundidad, nivel, grado, nodo interno, ancestros y descendientes, ABB y recorridos. Cada tarjeta resalta su concepto sobre un árbol de ejemplo. Tiene un buscador que no distingue mayúsculas ni acentos, y las definiciones enlazan entre sí.
- **Quiz:** 5 preguntas sobre árboles generados al azar, en las que hay que escribir el recorrido pedido. Al final se muestra la calificación sobre 10 y un repaso de cada respuesta.

## Tecnologías

[Vite](https://vite.dev), [React](https://react.dev), [React Router](https://reactrouter.com), [Motion](https://motion.dev) y CSS Modules, todo en JavaScript.

## Cómo ejecutarlo

Requiere [Node.js](https://nodejs.org).

```bash
npm install
npm run dev       # abre http://localhost:5173
npm run build     # build de producción en dist/
npm run preview   # sirve el build
```

## Estructura

```
src/
  main.jsx, App.jsx    arranque, router y rutas
  estilos/             reset y variables del tema
  logica/              motor del árbol (Nodo, ArbolBinario…), sin React
  utilidades/          lectura de valores, layout del árbol, aleatorios
  hooks/               estado visual y reproductor de pasos
  componentes/         Navegacion, VistaArbol, Consola, ControlesReproductor, PilaLlamadas
  modulos/             simulador/, teoria/, quiz/
docs/bocetos/          bocetos de las pantallas y decisiones de diseño
```

El motor (`src/logica/`) es independiente de la interfaz. Cada operación modifica el árbol y devuelve una lista de **pasos**. Un reproductor convierte esos pasos en estado visual, y la vista lo dibuja en SVG.

## Convenciones

- La raíz está en el **nivel 0** y la profundidad de un nodo es igual a su nivel.
- Una hoja tiene **altura 0** y el árbol vacío, **altura −1**.
- No se permiten valores repetidos, y un árbol es solo de números o solo de caracteres.
- Cada nodo tiene un color propio que no cambia nunca.

## Estado

En desarrollo. El plan completo, tarea por tarea, está en [`2026-10-27-arboles-binarios.md`](2026-10-27-arboles-binarios.md).

| Bloque | Estado |
|---|---|
| 0. Base del proyecto | ✅ |
| 1. Bocetos | pendiente |
| 2. Motor lógico | pendiente |
| 3. Navegación | pendiente |
| 4. Reproductor y vista | pendiente |
| 5. Simulador | pendiente |
| 6. Teoría | pendiente |
| 7. Quiz | pendiente |
