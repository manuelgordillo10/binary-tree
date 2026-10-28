# Plan: App didáctica de Árboles Binarios

> **Para agentes:** SUB-SKILL REQUERIDA: usar superpowers:subagent-driven-development (recomendado) o superpowers:executing-plans para implementar este plan tarea por tarea. Cada paso lleva una casilla (`- [ ]`) para marcar el avance.

**Objetivo:** Construir la app web con sus tres módulos: un simulador animado de un árbol binario de búsqueda, una sección de teoría con glosario dinámico y un quiz de recorridos. Todo funciona sobre un motor `ArbolBinario` propio.

**Arquitectura:** El motor (`src/logica/`) no depende de React. Cada operación modifica el árbol y devuelve una lista de pasos. Un reproductor (`useReproductor`) convierte esos pasos en estado visual, y `VistaArbol` lo dibuja en SVG usando una instantánea del árbol de antes y otra de después de la operación.

**Tecnologías:** Vite, React, React Router (`react-router`), Motion (`motion`) y CSS Modules, todo en JavaScript.

**Spec:** `docs/superpowers/specs/2026-10-27-arboles-binarios-design.md`, secciones 5 a 8. El despliegue (sección 9) queda fuera de este plan.

## Restricciones globales

- **Lenguaje:** JavaScript con módulos ES (`"type": "module"`), sin TypeScript.
- **Dependencias:** solo `react`, `react-dom`, `react-router` y `motion`; como dependencias de desarrollo, `vite` y `@vitejs/plugin-react`. Para agregar cualquier otra hay que preguntar al usuario.
- **Sin Vitest ni archivos de prueba.** Las comprobaciones son comandos `node` que se corren una vez en Git Bash desde la raíz del proyecto, o listas de revisión en el navegador. Ninguna de ellas se guarda en el repo.
- **Cero caja negra:**
  - Dentro de `src/logica/`, el árbol existe solo como objetos `Nodo` enlazados por `izquierdo` y `derecho`, y los recorridos son recursivos.
  - Está prohibido representar el árbol con arreglos, `Map`, `Set` o librerías.
  - Sí se pueden usar arreglos para guardar pasos y resultados.
- **Motor independiente:** `src/logica/` no importa nada de React ni del DOM.
- **Idioma:** el código y la interfaz van en español, con los identificadores del spec (`ArbolBinario`, `Nodo`, `insertar`, `izquierdo`…).
- **Color:** el tono de cada nodo es `(id × 137.508) mod 360`. El color de un nodo no cambia nunca, ni siquiera cuando sube como sucesor.
- **Convenciones:** la raíz está en el nivel 0, la profundidad es igual al nivel, una hoja tiene altura 0 y el árbol vacío tiene altura −1.
- **Animaciones:**
  - Las micro-animaciones duran menos de 300 ms.
  - Se respeta `prefers-reduced-motion` mediante `MotionConfig reducedMotion="user"`.
- **Dispositivos:** escritorio y tableta. No se optimiza para teléfono.
- **Sin persistencia:** no hay backend ni `localStorage`.
- **Commits:** uno por tarea, con solo el mensaje que indica la tarea, sin `Co-Authored-By` ni otros contribuidores. No se hace `git push` automáticamente: solo cuando el usuario lo pide.

## Foco de revisión

Estos son los casos que más probablemente le fallen a un estudiante. Cada uno tiene su comprobación en la tarea indicada.

1. **Eliminar la raíz** en sus tres casos, incluido el de dejar el árbol vacío. El árbol debe seguir ordenado, el sucesor debe conservar su color y el tipo del árbol debe liberarse. Se comprueba en la **Tarea 5**.
2. **Entradas raras** en el campo de valor, como `+5`, `05`, `-0`, `3.5`, `ab`, `ñ`, el campo vacío o solo espacios. Cada una se acepta o se rechaza con un mensaje, sin romper la app. Se comprueba en la **Tarea 3**.
3. **Árbol degenerado**, con los valores 1 a 30 insertados en orden. La vista debe ajustarse, el zoom, el arrastre y "Centrar" deben funcionar, y la rueda no debe desplazar la página. Se comprueba en la **Tarea 11**.
4. **Cambiar de módulo a mitad de una animación** y volver al simulador. No debe haber errores, el árbol debe conservarse y el temporizador debe quedar limpio. Se comprueba en las **Tareas 9 y 12**.
5. **Respuestas del quiz con formato raro**: comas al final, comas dobles, espacios de más o números de más o de menos. Cada respuesta se lee o se rechaza con un mensaje claro. Se comprueba en la **Tarea 17**.

## Mapa de archivos

```
index.html, vite.config.js, package.json, .gitignore
docs/bocetos/                      bocetos HTML y decisiones.md (Tarea 2)
src/
  main.jsx, App.jsx                arranque, router, MotionConfig, rutas
  estilos/global.css, tema.css     reset y variables del tema
  logica/                          motor; sin React
    Nodo.js  colores.js  formato.js  ArbolBinario.js
  utilidades/
    parsearValor.js  layout.js  aleatorio.js
  hooks/
    estadoVisual.js                pasos → estado visual (puro)
    useReproductor.js              reproducir, pausar, paso a paso, velocidad
  componentes/
    Navegacion/  VistaArbol/  Consola/  ControlesReproductor/  PilaLlamadas/
  modulos/
    simulador/   PaginaSimulador.jsx, arbolDelSimulador.js
    teoria/      PaginaTeoria.jsx, conceptos.js, TarjetaConcepto.jsx, TextoConEnlaces.jsx, IconoArbol.jsx
    quiz/        PaginaQuiz.jsx, logicaQuiz.js
```

## Bloques

| Bloque | Tareas | Depende de |
|---|---|---|
| 0. Base del proyecto | 1 | — |
| 1. Bocetos ⏸ | 2 | 1 |
| 2. Motor lógico | 3, 4, 5, 6, 7 | 1 |
| 3. Navegación | 8 | 2 aprobada |
| 4. Reproductor y vista | 9, 10, 11 | 2, 3 |
| 5. Simulador | 12, 13 | 4 |
| 6. Teoría ⏸ | 14, 15, 16 | 4 |
| 7. Quiz | 17, 18 | 4, 12 |

⏸ marca los puntos donde se espera la revisión del usuario. Mientras se esperan los bocetos, el Bloque 2 puede avanzar.

---

## Bloque 0: Base del proyecto

### Tarea 1: Proyecto base

**Archivos:** crear `package.json`, `vite.config.js`, `index.html`, `.gitignore`, `src/main.jsx`, `src/App.jsx` y `src/estilos/global.css`.

- [x] **Paso 1:** Correr `git init` en la raíz y crear un `.gitignore` con `node_modules/` y `dist/`.
- [x] **Paso 2:** Correr `npm init -y` y dejar `package.json` con:
  - `"name": "arboles-binarios"`, `"private": true` y `"type": "module"`;
  - los scripts `"dev": "vite"`, `"build": "vite build"` y `"preview": "vite preview"`;
  - sin los campos `main` ni `test`.
- [x] **Paso 3:** Instalar dependencias con `npm install react react-dom react-router motion` y `npm install -D vite @vitejs/plugin-react`.
- [x] **Paso 4:** Crear los archivos de arranque:
  - `vite.config.js` con `@vitejs/plugin-react`.
  - `index.html` con `lang="es"`, `<title>Árboles Binarios</title>`, `<div id="root">` y `<script type="module" src="/src/main.jsx">`.
  - `src/main.jsx`, que monta `<App />` dentro de `<StrictMode>` e importa `global.css`.
  - `src/App.jsx`, que muestra un `<h1>Árboles Binarios</h1>` provisional.
  - `global.css` con `box-sizing: border-box` y márgenes en 0.
- [x] **Paso 5:** Verificar:
  - `npm run build` termina con `✓ built in …`.
  - Con `npm run dev`, `http://localhost:5173` muestra el título y la consola del navegador no tiene errores.
- [x] **Paso 6:** Hacer commit con el mensaje `chore: proyecto base con Vite y React`.

---

## Bloque 1: Bocetos

### Tarea 2: Bocetos de las pantallas ⏸ revisión del usuario

**Archivos:** crear en `docs/bocetos/` los archivos `index.html`, `simulador-a.html`, `simulador-b.html`, `teoria.html`, `quiz.html`, `bocetos.css` y `decisiones.md`. Crear también `src/estilos/tema.css`.

**Produce:**
- `src/estilos/tema.css` con las variables que usan todas las tareas de interfaz: `--color-fondo`, `--color-superficie`, `--color-texto`, `--color-texto-suave`, `--color-borde`, `--color-acento`, `--estado-camino`, `--estado-visitado`, `--estado-encontrado`, `--estado-por-eliminar`, `--estado-sucesor`, `--estado-nuevo`, `--estado-destacado`, `--estado-secundario`, `--estado-referencia`, `--fuente-ui`, `--fuente-mono` y `--radio`.
- `docs/bocetos/decisiones.md` con todo lo que el usuario apruebe.

- [ ] **Paso 1:** Hacer bocetos en HTML estático, sin React. Todos llevan la barra de navegación de los tres módulos, textos reales y el árbol `[50, 30, 70, 20, 40, 60, 80]` dibujado a mano en SVG.
  - `simulador-a.html`: el árbol ocupa todo el ancho arriba. Debajo, en tres columnas, van los controles, la consola y la pila de llamadas.
  - `simulador-b.html`: el árbol ocupa cerca del 70 % a la izquierda. A la derecha, un panel con los controles, la consola y la pila.
  - `teoria.html`: un buscador arriba y la lista de tarjetas, con una abierta que muestra la definición y un árbol con nodos resaltados y el resto atenuado.
  - `quiz.html`: una pregunta en curso y, debajo, la pantalla de resultado con la calificación y el repaso.
  - En todos se ven los estados de los nodos: el relleno es siempre el color propio del nodo, el estado se marca con un anillo exterior y el puntero es un anillo más grande.
  - `index.html` enlaza los cinco bocetos.
- [ ] **Paso 2:** Abrir los bocetos con `start docs/bocetos/index.html` y preguntar al usuario:
  - qué simulador prefiere, A o B;
  - qué cambiaría en cada pantalla;
  - qué paleta y tipografía quiere;
  - si `/` debe ser una portada o redirigir a `/simulador`;
  - si quiere modo oscuro.
- [ ] **Paso 3:** Repetir los bocetos hasta que el usuario los apruebe. **El Bloque 3 no empieza sin esa aprobación.**
- [ ] **Paso 4:** Escribir `decisiones.md`, con una sección por pantalla más paleta, tipografía y portada, y `tema.css` con los valores aprobados.
- [ ] **Paso 5:** Hacer commit con el mensaje `docs: bocetos aprobados y tema visual`.

---

## Bloque 2: Motor lógico

### Tarea 3: Nodo, colores, formato y lectura de valores

**Archivos:** crear `src/logica/Nodo.js`, `src/logica/colores.js`, `src/logica/formato.js` y `src/utilidades/parsearValor.js`.

**Produce:**
- `class Nodo { constructor(id: number, valor: number|string, color: { fondo, texto }) }`, con `izquierdo = null` y `derecho = null`. Solo guarda datos y no tiene métodos.
- `colorParaId(id: number) → { fondo: string, texto: '#ffffff' | '#1f2937' }`.
- `mostrarValor(valor) → string`: un número se muestra tal cual (`50`) y un carácter entre comillas simples (`'a'`).
- `parsearValor(texto: string) → { ok: true, valor: number|string } | { ok: false, mensaje: string }`.

- [ ] **Paso 1:** Escribir `Nodo.js` y `formato.js`.
- [ ] **Paso 2:** Escribir `colores.js`:
  - `tono = (id * 137.508) % 360` y `fondo = hsl(tono 70% 55%)`.
  - `texto` es, de los dos colores posibles, el que tenga mayor contraste WCAG con el fondo. Para calcularlo, se convierte HSL a sRGB y se usa la luminancia relativa.
- [ ] **Paso 3:** Escribir `parsearValor.js`:
  - Primero se recortan los espacios.
  - Si el texto cumple `/^[+-]?\d+$/`, es un número. `-0` se guarda como `0`. Si el número no es `Number.isSafeInteger`, el mensaje es `"El número es demasiado grande."`.
  - Si no es número y `Array.from(t).length === 1`, es un carácter.
  - En cualquier otro caso, incluido el texto vacío, el mensaje es `"Escribe un número entero o un solo carácter."`.
- [ ] **Paso 4:** Verificar (foco de revisión 2):

```bash
node --input-type=module -e "
import { parsearValor } from './src/utilidades/parsearValor.js';
import { colorParaId } from './src/logica/colores.js';
for (const t of ['  42 ', '-7', '+5', '05', '-0', 'a', 'ñ', '-', 'ab', '3.5', '', '   ']) console.log(JSON.stringify(t), JSON.stringify(parsearValor(t)));
console.log([1, 2, 3, 4].map(i => colorParaId(i).fondo).join(' | '));
"
```

Resultado esperado:
- `ok: true` con los valores `42`, `-7`, `5`, `5`, `0`, `"a"`, `"ñ"` y `"-"`.
- `ok: false` para `ab`, `3.5`, el texto vacío y el de solo espacios.
- Cuatro valores de `fondo` distintos.

- [ ] **Paso 5:** Hacer commit con el mensaje `feat: Nodo, colores y lectura de valores`.

### Tarea 4: ArbolBinario — insertar, buscar e instantánea

**Archivos:** crear `src/logica/ArbolBinario.js`.

**Consume:** `Nodo`, `colorParaId` y `mostrarValor`, de la Tarea 3.

**Produce:**
- `class ArbolBinario` con estos campos:
  - `raiz: Nodo|null`;
  - `tipo: 'numero'|'caracter'|null`;
  - `#siguienteId`, que empieza en 1 y **no** se reinicia con `limpiar`. Así Motion puede distinguir los nodos viejos de los nuevos.
- `static desdeValores(valores) → ArbolBinario`: inserta los valores en orden e ignora los pasos.
- `insertar(valor) → { pasos, exito, mensaje }`.
- `buscar(valor) → { pasos, exito, mensaje }`.
- `instantanea() → { raizId: number|null, tipo, nodos: NodoPlano[] }`, con `nodos` **en orden inorden**. Cada `NodoPlano` es `{ id, valor, color, izquierdoId, derechoId, padreId, profundidad }`.
- `limpiar() → void`: deja `raiz = null` y `tipo = null`.
- Los tipos de paso de esta tarea. Su forma vale para todo el plan:

| tipo | campos | mensaje de ejemplo |
|---|---|---|
| `comparar` | `nodoId, valor, direccion: 'izquierda'\|'derecha'\|'igual'` | `35 < 50 → izquierda` · `30 = 30` |
| `encontrado` | `nodoId` | `Se encontró 35` |
| `no-encontrado` | — | `99 no está en el árbol (se llegó a un hueco vacío)` · `El árbol está vacío` |
| `cambio-estructura` | `nodoId: number\|null` | `Se inserta 35 como hijo izquierdo de 40` · `35 es la nueva raíz` |
| `error` | — | `30 ya existe: no se insertan repetidos` · `El árbol es de números: no se puede insertar 'a'` |

Todo `mensaje` usa `mostrarValor`. El `exito` y el `mensaje` del resultado resumen el último paso.

- [ ] **Paso 1:** Escribir el método privado `#comparar(a, b)`. Con números devuelve `a - b`; con caracteres, `a.codePointAt(0) - b.codePointAt(0)`.
- [ ] **Paso 2:** Escribir la validación de tipo:
  - El tipo de un valor es `'numero'` si `typeof valor === 'number'` y `'caracter'` en otro caso.
  - Si el árbol ya tiene tipo y no coincide, la operación devuelve un único paso `error` cuyo mensaje usa el verbo de la operación (insertar, buscar o eliminar).
  - Si el `tipo` del árbol es `null`, `insertar` lo fija.
- [ ] **Paso 3:** Escribir `insertar` y `buscar`:
  - Ambos bajan desde la raíz y generan un `comparar` por nodo, con dirección `igual` cuando dan con el valor.
  - `buscar` termina con `encontrado` o con `no-encontrado`.
  - `insertar` termina con `cambio-estructura`, que lleva el `nodoId` del nodo nuevo, o con `error` si el valor ya existía.
- [ ] **Paso 4:** Escribir `instantanea` (un recorrido inorden recursivo que arma los `NodoPlano`), `limpiar` y `desdeValores`.
- [ ] **Paso 5:** Verificar:

```bash
node --input-type=module -e "
import { ArbolBinario } from './src/logica/ArbolBinario.js';
const t = r => r.exito + ' ' + r.pasos.map(p => p.tipo + (p.direccion ? ':' + p.direccion : '')).join(' ');
const a = ArbolBinario.desdeValores([50, 30, 70, 20, 40]);
const r = a.insertar(35); console.log(t(r), r.pasos.at(-1).nodoId);
console.log(a.instantanea().nodos.map(n => n.valor + '@' + n.profundidad).join(' '));
console.log(t(a.insertar(30)));
console.log(t(a.insertar('a')), a.insertar('a').mensaje);
console.log(t(a.buscar(35)));
console.log(t(a.buscar(99)));
console.log(t(new ArbolBinario().buscar(5)));
a.limpiar(); console.log(a.instantanea().raizId, a.tipo, t(a.insertar('m')));
"
```

Resultado esperado:
```
true comparar:izquierda comparar:derecha comparar:izquierda cambio-estructura 6
20@2 30@1 35@3 40@2 50@0 70@1
false comparar:izquierda comparar:igual error
false error El árbol es de números: no se puede insertar 'a'
true comparar:izquierda comparar:derecha comparar:izquierda comparar:igual encontrado
false comparar:derecha comparar:derecha no-encontrado
false no-encontrado
null null true cambio-estructura
```

- [ ] **Paso 6:** Hacer commit con el mensaje `feat: ArbolBinario con insertar, buscar e instantánea`.

### Tarea 5: ArbolBinario — eliminar

**Archivos:** modificar `src/logica/ArbolBinario.js`.

**Produce:** `eliminar(valor) → { pasos, exito, mensaje }` y dos tipos de paso nuevos:

| tipo | campos | mensaje de ejemplo |
|---|---|---|
| `caso-eliminacion` | `nodoId, caso: 'hoja'\|'un-hijo'\|'dos-hijos'` | `50 tiene dos hijos: se busca su sucesor inorden (el menor del subárbol derecho)` |
| `sucesor` | `nodoId, esElSucesor: boolean` | `Bajamos por la izquierda: 70` · `60 es el sucesor` |

- [ ] **Paso 1:** Bajar por el árbol igual que en `buscar`, generando pasos `comparar`. Si el valor no existe, terminar con `no-encontrado`. Si existe, generar `caso-eliminacion`.
- [ ] **Paso 2:** Reenganchar los nodos. El sucesor se mueve físicamente; su valor nunca se copia a otro nodo.

```
N = nodo a eliminar, P = su padre (o la raíz)
hoja:      el enlace de P hacia N pasa a null
un hijo:   el enlace de P hacia N pasa al único hijo de N
dos hijos: S = mínimo de N.derecho, SP = padre de S
           (un paso 'sucesor' por cada nodo desde N.derecho hasta S; esElSucesor solo en S)
           si SP !== N: SP.izquierdo = S.derecho; S.derecho = N.derecho
           S.izquierdo = N.izquierdo
           el enlace de P hacia N pasa a S
```

- [ ] **Paso 3:** Terminar con un paso `cambio-estructura`:
  - Su `nodoId` es S en el caso de dos hijos, el hijo que sube en el caso de un hijo y `null` en el caso de hoja.
  - El mensaje sigue el formato `Se eliminó 50; 60 ocupa su lugar`.
  - Si el árbol queda vacío, `tipo` vuelve a `null`.
- [ ] **Paso 4:** Verificar (foco de revisión 1):

```bash
node --input-type=module -e "
import { ArbolBinario } from './src/logica/ArbolBinario.js';
const B = [50, 30, 70, 20, 40, 60, 80, 65];
const t = r => r.pasos.map(p => p.tipo + (p.direccion ? ':' + p.direccion : '') + (p.caso ? ':' + p.caso : '')).join(' ');
const io = a => a.instantanea().nodos.map(n => n.valor + '@' + n.profundidad).join(' ');
let a = ArbolBinario.desdeValores(B); console.log(t(a.eliminar(20))); console.log(io(a));
a = ArbolBinario.desdeValores(B); console.log(t(a.eliminar(60))); console.log(io(a));
a = ArbolBinario.desdeValores(B); const c = a.instantanea().nodos.find(n => n.valor === 60).color.fondo;
console.log(t(a.eliminar(50))); console.log(io(a), a.instantanea().raizId, a.instantanea().nodos.find(n => n.id === 6).color.fondo === c);
a = ArbolBinario.desdeValores([50, 30, 70, 80]); console.log(t(a.eliminar(50))); console.log(io(a));
a = ArbolBinario.desdeValores([50, 30, 20]); a.eliminar(50); console.log(io(a));
a = ArbolBinario.desdeValores([5]); a.eliminar(5); console.log(a.instantanea().raizId, a.tipo, a.insertar('a').exito);
a = ArbolBinario.desdeValores(B); console.log(t(a.eliminar(99)));
"
```

Resultado esperado:
```
comparar:izquierda comparar:izquierda comparar:igual caso-eliminacion:hoja cambio-estructura
30@1 40@2 50@0 60@2 65@3 70@1 80@2
comparar:derecha comparar:izquierda comparar:igual caso-eliminacion:un-hijo cambio-estructura
20@2 30@1 40@2 50@0 65@2 70@1 80@2
comparar:igual caso-eliminacion:dos-hijos sucesor sucesor cambio-estructura
20@2 30@1 40@2 60@0 65@2 70@1 80@2 6 true
comparar:igual caso-eliminacion:dos-hijos sucesor cambio-estructura
30@1 70@0 80@1
20@1 30@0
null null true
comparar:derecha comparar:derecha comparar:derecha no-encontrado
```

- [ ] **Paso 5:** Hacer commit con el mensaje `feat: eliminar con sucesor inorden reenganchado`.

### Tarea 6: ArbolBinario — recorridos

**Archivos:** modificar `src/logica/ArbolBinario.js`.

**Produce:** `preorden()`, `inorden()` y `postorden()`, que devuelven `{ pasos, secuencia: Array<number|string> }`. Usan estos tipos de paso:

| tipo | campos | mensaje |
|---|---|---|
| `llamar` | `nodoId, valor, funcion` | `inorden(50)` |
| `visitar` | `nodoId, valor` | `Imprime 50` |
| `retornar` | `nodoId, valor, funcion` | `Termina inorden(50)`. El de la raíz añade `. Salida: 20 30 40 50 70` |

- [ ] **Paso 1:** Escribir un solo método privado recursivo, `#recorrer(nodo, funcion, pasos, secuencia)`:
  - Si `nodo === null`, retorna sin generar pasos.
  - Si no, genera `llamar`, luego recorre los hijos y genera `visitar` en el orden que corresponde a `funcion`, y al final genera `retornar`.
  - No usa una pila propia: la recursión es la de JavaScript.
- [ ] **Paso 2:** Con el árbol vacío, el resultado es `{ pasos: [{ tipo: 'error', mensaje: 'El árbol está vacío' }], secuencia: [] }`.
- [ ] **Paso 3:** Verificar:

```bash
node --input-type=module -e "
import { ArbolBinario } from './src/logica/ArbolBinario.js';
const a = ArbolBinario.desdeValores([50, 30, 70, 20, 40]);
for (const f of ['preorden', 'inorden', 'postorden']) { const r = a[f](); console.log(f, r.secuencia.join(' '), r.pasos.length); }
console.log(a.inorden().pasos.slice(0, 4).map(p => p.tipo + ':' + p.valor).join(' '));
console.log(a.inorden().pasos.at(-1).mensaje);
console.log(JSON.stringify(new ArbolBinario().inorden()));
"
```

Resultado esperado:
```
preorden 50 30 20 40 70 15
inorden 20 30 40 50 70 15
postorden 20 40 30 70 50 15
llamar:50 llamar:30 llamar:20 visitar:20
Termina inorden(50). Salida: 20 30 40 50 70
{"pasos":[{"tipo":"error","mensaje":"El árbol está vacío"}],"secuencia":[]}
```

- [ ] **Paso 4:** Hacer commit con el mensaje `feat: recorridos recursivos con pasos`.

### Tarea 7: ArbolBinario — consultas para la teoría

**Archivos:** modificar `src/logica/ArbolBinario.js`.

**Produce:** todas las consultas reciben un `valor` y devuelven ids. Si el valor no existe, devuelven `null` o `[]`.

| consulta | devuelve |
|---|---|
| `idDe(valor)` | `number\|null` |
| `altura()` | `number`: −1 si el árbol está vacío, 0 para una hoja |
| `hojas()` | `number[]` |
| `internos()` | `number[]` |
| `padreDe(valor)` | `number\|null` |
| `hijosDe(valor)` | `number[]` |
| `hermanoDe(valor)` | `number\|null` |
| `subarbolDe(valor)` | `number[]`, incluido el propio nodo |
| `ancestrosDe(valor)` | `number[]`, de la raíz hacia abajo y sin el nodo |
| `descendientesDe(valor)` | `number[]`, sin el nodo |
| `caminoMasLargo()` | `number[]`, de la raíz a la hoja más profunda. Si hay empate, gana la que se encuentra primero yendo por la izquierda |
| `gradoDe(valor)` | `0\|1\|2\|null` |

- [ ] **Paso 1:** Implementar las consultas con recorridos recursivos sobre `Nodo`, más un auxiliar privado `#buscarConPadre(valor) → { nodo, padre, ancestros }` que no genera pasos.
- [ ] **Paso 2:** Verificar:

```bash
node --input-type=module -e "
import { ArbolBinario } from './src/logica/ArbolBinario.js';
const a = ArbolBinario.desdeValores([50, 30, 70, 20, 40, 60, 80, 65, 62]);
const n = new Map(a.instantanea().nodos.map(x => [x.id, x.valor]));
const v = ids => ids === null ? 'null' : Array.isArray(ids) ? ids.map(i => n.get(i)).join(' ') : n.get(ids);
const s = ids => ids.map(i => n.get(i)).sort((x, y) => x - y).join(' ');
console.log(a.altura(), new ArbolBinario().altura());
console.log(s(a.hojas()), '|', s(a.internos()));
console.log(v(a.padreDe(65)), v(a.padreDe(50)), '|', s(a.hijosDe(70)), '|', v(a.hermanoDe(20)), v(a.hermanoDe(65)));
console.log(s(a.subarbolDe(70)), '|', v(a.ancestrosDe(62)), '|', s(a.descendientesDe(60)));
console.log(v(a.caminoMasLargo()), '|', a.gradoDe(50), a.gradoDe(60), a.gradoDe(20), a.gradoDe(99));
"
```

Resultado esperado:
```
4 -1
20 40 62 80 | 30 50 60 65 70
60 null | 60 80 | 40 null
60 62 65 70 80 | 50 70 60 65 | 62 65
50 70 60 65 62 | 2 1 0 null
```

- [ ] **Paso 3:** Hacer commit con el mensaje `feat: consultas del árbol para la teoría`.

---

## Bloque 3: Navegación

### Tarea 8: Rutas y navegación

**Archivos:**
- Modificar `src/main.jsx` y `src/App.jsx`.
- Crear `src/componentes/Navegacion/Navegacion.jsx` y `Navegacion.module.css`.
- Crear las páginas provisionales `src/modulos/simulador/PaginaSimulador.jsx`, `src/modulos/teoria/PaginaTeoria.jsx` y `src/modulos/quiz/PaginaQuiz.jsx`, cada una con su título.

**Consume:** `tema.css` y `decisiones.md`, de la Tarea 2.

- [ ] **Paso 1:** En `main.jsx`, envolver `<App />` con `<BrowserRouter>` (de `react-router`) y con `<MotionConfig reducedMotion="user">` (de `motion/react`). Importar `tema.css`.
- [ ] **Paso 2:** En `App.jsx`, poner `<Navegacion />` y estas `<Routes>`:
  - `/simulador`, `/teoria` y `/quiz`, con su página;
  - `/`, que hace `<Navigate to="/simulador" replace />`, o muestra la portada si `decisiones.md` la aprobó;
  - `*`, que hace `<Navigate to="/" replace />`.
- [ ] **Paso 3:** Hacer `Navegacion` con tres `NavLink`: "Simulador", "Teoría" y "Quiz". El enlace activo se marca como indiquen los bocetos.
- [ ] **Paso 4:** Verificar con `npm run dev`:
  - los tres enlaces cambian de página sin recargar y el activo queda marcado;
  - `/` y `/xyz` llevan a `/simulador`;
  - recargar en `/teoria` sigue mostrando Teoría;
  - `npm run build` termina sin errores.
- [ ] **Paso 5:** Hacer commit con el mensaje `feat: rutas y navegación`.

---

## Bloque 4: Reproductor y vista

### Tarea 9: Estado visual y reproductor

**Archivos:** crear `src/hooks/estadoVisual.js` y `src/hooks/useReproductor.js`.

**Consume:** las formas de paso de las Tareas 4 a 6.

**Produce:**
- `derivarEstadoVisual(pasos, indice, antes, despues)`, que aplica los pasos `pasos[0..indice-1]` y devuelve:
  - `instantanea`;
  - `mensajes: string[]`;
  - `salida: Array<number|string>`;
  - `pila: Array<{ nodoId, valor, funcion }>`;
  - `nodoActivoId: number|null`;
  - `estados: Map<number, EstadoNodo>`, donde `EstadoNodo` es `'camino'|'visitado'|'encontrado'|'por-eliminar'|'sucesor'|'nuevo'`.
- `useReproductor()`, que devuelve:
  - `estado: 'inactivo'|'reproduciendo'|'pausado'|'terminado'`, `enCurso: boolean`, `indice`, `total` y `velocidad: 'lenta'|'normal'|'rapida'`;
  - `visual`, el resultado de `derivarEstadoVisual`;
  - los controles `cargar({ pasos, antes, despues })`, `reproducir()`, `pausar()`, `siguiente()`, `saltarAlFinal()` y `setVelocidad(v)`.

- [ ] **Paso 1:** Escribir `derivarEstadoVisual`, que es una función pura sin React. Cada paso añade su `mensaje` a `mensajes`, y además:

| paso | efecto |
|---|---|
| `comparar` | `estados[nodoId] = 'camino'`; el activo pasa a ser `nodoId` |
| `sucesor` | `estados[nodoId] = esElSucesor ? 'sucesor' : 'camino'`; el activo pasa a ser `nodoId` |
| `encontrado` | `'encontrado'`; el activo pasa a ser `nodoId` |
| `caso-eliminacion` | `'por-eliminar'`; el activo pasa a ser `nodoId` |
| `cambio-estructura` | desde este paso, `instantanea = despues`. Se vacía `estados`. Si hay `nodoId`, recibe `'nuevo'` y pasa a ser el activo; si no, el activo es `null` |
| `llamar` | se apila; el activo pasa a ser `nodoId` |
| `visitar` | `'visitado'`; `valor` se añade a `salida`; el activo pasa a ser `nodoId` |
| `retornar` | se desapila; el activo es el `nodoId` de la nueva cima, o `null` si la pila queda vacía |
| `error`, `no-encontrado` | el activo es `null` |

Antes del primer `cambio-estructura`, `instantanea = antes`.

- [ ] **Paso 2:** Escribir `useReproductor`:
  - Milisegundos por paso: lenta 1200, normal 700 y rápida 300. La velocidad inicial es normal.
  - `cargar` pone `indice = Math.min(1, total)` y arranca solo, con estado `'reproduciendo'`, o `'terminado'` si `total <= 1`.
  - Mientras el estado sea `'reproduciendo'`, un `setTimeout` avanza un paso cada vez. Al llegar a `total`, el estado pasa a `'terminado'`.
  - `siguiente()` pausa y avanza un paso.
  - `saltarAlFinal()` pone `indice = total`.
  - `enCurso` vale `estado === 'reproduciendo' || estado === 'pausado'`.
  - El efecto limpia su temporizador al desmontarse (foco de revisión 4).
  - Antes del primer `cargar`, el estado es `'inactivo'` y `visual` tiene `instantanea: null`.
- [ ] **Paso 3:** Verificar `derivarEstadoVisual`. El hook se comprueba en el navegador en la Tarea 12.

```bash
node --input-type=module -e "
import { ArbolBinario } from './src/logica/ArbolBinario.js';
import { derivarEstadoVisual } from './src/hooks/estadoVisual.js';
const a = ArbolBinario.desdeValores([50, 30, 70, 20, 40]);
const i0 = a.instantanea(); const { pasos } = a.inorden();
let e = derivarEstadoVisual(pasos, 3, i0, i0); console.log(e.pila.map(p => p.valor).join(' '), e.nodoActivoId);
e = derivarEstadoVisual(pasos, 5, i0, i0); console.log(e.pila.map(p => p.valor).join(' '), e.nodoActivoId, e.salida.join(' '));
const r = a.insertar(35); const i1 = a.instantanea();
e = derivarEstadoVisual(r.pasos, 3, i0, i1); console.log(e.instantanea === i0, [...e.estados].join(' '));
e = derivarEstadoVisual(r.pasos, 4, i0, i1); console.log(e.instantanea === i1, [...e.estados].join(' '), e.mensajes.length);
"
```

Resultado esperado:
```
50 30 20 4
50 30 2 20
true 1,camino 2,camino 5,camino
true 6,nuevo 4
```

- [ ] **Paso 4:** Hacer commit con el mensaje `feat: estado visual y reproductor de pasos`.

### Tarea 10: Vista del árbol

**Archivos:**
- Crear `src/utilidades/layout.js`, `src/componentes/VistaArbol/VistaArbol.jsx` y `VistaArbol.module.css`.
- Modificar provisionalmente `PaginaSimulador.jsx` para montar una demo.

**Produce:**
- `calcularLayout(instantanea) → { posiciones: Map<id, {x, y}>, ancho, alto }`:
  - con las constantes `SEPARACION_X = 60`, `SEPARACION_Y = 80`, `MARGEN = 40` y `RADIO_NODO = 22`;
  - `x = MARGEN + i * SEPARACION_X`, donde `i` es el índice en `nodos`, que ya viene en inorden;
  - `y = MARGEN + profundidad * SEPARACION_Y`.
- `<VistaArbol instantanea nodoActivoId estados etiquetas mostrarNiveles atenuarNoResaltados interactiva />`. Solo `instantanea` es obligatoria:
  - `estados: Map<id, EstadoNodo | 'destacado' | 'secundario' | 'referencia'>`;
  - `etiquetas: Map<id, string>`;
  - `mostrarNiveles`;
  - `atenuarNoResaltados`: si `estados.size > 0`, los nodos sin estado se dibujan con opacidad 0.35;
  - `interactiva`: activa el zoom y el arrastre de la Tarea 11. Vale `true` por defecto;
  - si `instantanea` es `null` o no tiene nodos, se muestra el texto "El árbol está vacío".

- [ ] **Paso 1:** Escribir `layout.js`.
- [ ] **Paso 2:** Escribir `VistaArbol`:
  - Usa `<svg viewBox="0 0 ancho alto" preserveAspectRatio="xMidYMid meet">`.
  - Las aristas son `motion.line` (key `padreId-hijoId`) y los nodos son `motion.g` (key `id`), dentro de `AnimatePresence`.
  - Cada nodo es un `circle` con `color.fondo` y un `text` con `color.texto`.
  - El relleno **nunca** cambia. El estado se dibuja como un anillo exterior con el color `var(--estado-<estado>)`.
  - El puntero es un `motion.circle` de radio `RADIO_NODO + 8` que viaja hasta el nodo activo.
  - Los movimientos duran 400 ms. Los nodos entran y salen con escala y opacidad.
  - Con `mostrarNiveles` se dibujan franjas horizontales con el rótulo "Nivel n". Las `etiquetas` se escriben debajo de su nodo.
- [ ] **Paso 3:** Montar una demo provisional en `PaginaSimulador`:
  - `ArbolBinario.desdeValores([50, 30, 70, 20, 40, 60, 80, 65])`, con el nodo 30 como activo y un nodo con cada tipo de estado;
  - un botón "Insertar aleatorio" que inserta un número y actualiza la instantánea.
- [ ] **Paso 4:** Verificar en el navegador:
  - los hijos izquierdos quedan a la izquierda y los derechos a la derecha;
  - cada nodo tiene un color distinto y su texto se lee bien;
  - cada estado se ve con su anillo;
  - al insertar, los nodos se deslizan a su nueva posición y el nuevo aparece;
  - con el árbol vacío se muestra el texto.
- [ ] **Paso 5:** Hacer commit con el mensaje `feat: VistaArbol en SVG con estados y puntero`.

### Tarea 11: Zoom, arrastre y centrar

**Archivos:** modificar `VistaArbol.jsx` y su CSS.

- [ ] **Paso 1:** Guardar el estado `vista = { x, y, ancho, alto }` y usarlo como `viewBox`. La vista se reajusta al contenido cuando cambian el `ancho` o el `alto` del layout, y también con el botón "Centrar", que va en la esquina superior derecha.
- [ ] **Paso 2:** Zoom con la rueda:
  - cada paso de la rueda multiplica el zoom por 1.1, centrado en el puntero;
  - el zoom queda entre 0.2× y 5× del tamaño ajustado;
  - el listener se registra con `addEventListener('wheel', …, { passive: false })` dentro de un `useEffect`. El `onWheel` de React es pasivo y no permite hacer `preventDefault`.
- [ ] **Paso 3:** Arrastre con `pointerdown`, `pointermove` y `pointerup`, usando `setPointerCapture`. El cursor es `grab` en reposo y `grabbing` mientras se arrastra.
- [ ] **Paso 4:** Con `interactiva={false}` no se registran listeners ni aparece el botón "Centrar", así la rueda desplaza la página con normalidad.
- [ ] **Paso 5:** Verificar (foco de revisión 3). En la demo, agregar un botón provisional "Cadena 1–30" que inserta los valores del 1 al 30 en orden, y comprobar que:
  - la vista se ajusta y se ve el árbol completo;
  - la rueda hace zoom hacia el cursor sin mover la página;
  - arrastrar mueve el árbol;
  - "Centrar" vuelve a la vista ajustada.
- [ ] **Paso 6:** Hacer commit con el mensaje `feat: zoom, arrastre y centrar en VistaArbol`.

---

## Bloque 5: Simulador

### Tarea 12: Simulador — insertar, buscar y eliminar

**Archivos:**
- Crear `src/modulos/simulador/arbolDelSimulador.js` y `src/utilidades/aleatorio.js`.
- Crear `src/componentes/Consola/Consola.jsx` y `src/componentes/ControlesReproductor/ControlesReproductor.jsx`, cada uno con su CSS.
- Crear `PaginaSimulador.module.css`.
- Reemplazar `PaginaSimulador.jsx`, quitando la demo.

**Consume:**
- `ArbolBinario`, de las Tareas 4 y 5;
- `parsearValor`, de la Tarea 3;
- `useReproductor`, de la Tarea 9;
- `VistaArbol`, de las Tareas 10 y 11;
- `decisiones.md`.

**Produce:**
- `arbolDelSimulador`: una única instancia `new ArbolBinario()` exportada. Como vive en el módulo, se conserva al cambiar de ruta y se pierde al recargar la página.
- `enteroAleatorio(min, max) → number`, con los extremos incluidos.
- `valoresAleatoriosDistintos(cantidad, min, max) → number[]`.
- `<Consola lineas={string[]} salida={{ nombre, valores } | null} />`: baja sola al final cuando cambian las líneas. La salida se muestra aparte, en una línea fija, por ejemplo `Inorden: 20 30 40`.
- `<ControlesReproductor estado velocidad onReproducir onPausar onSiguiente onSaltar onVelocidad />`:
  - "Reproducir" solo está activo en pausa;
  - "Pausar" solo está activo mientras se reproduce;
  - "Siguiente" y "Saltar al final" solo están activos con `enCurso`;
  - un selector de velocidad con Lenta, Normal y Rápida.

- [ ] **Paso 1:** Escribir `aleatorio.js`.
- [ ] **Paso 2:** Escribir `Consola` y `ControlesReproductor`.
- [ ] **Paso 3:** Escribir `PaginaSimulador`, con el reparto de la pantalla aprobado en los bocetos.
  - Tiene una función interna `ejecutar(nombre, operacion)` que hace, en orden:
    1. `antes = instantanea()`;
    2. `resultado = operacion(arbolDelSimulador)`;
    3. `despues = instantanea()`;
    4. pasa las líneas de la operación anterior al historial, que guarda como máximo 200 líneas;
    5. `cargar({ pasos: resultado.pasos, antes, despues })`.
  - `Consola` recibe `lineas = [...historial, ...visual.mensajes]`.
  - Al montarse llama a `cargar({ pasos: [], antes: inst, despues: inst })`, para mostrar el árbol conservado.
- [ ] **Paso 4:** Añadir el campo de valor y los botones Insertar (que también responde a Enter), Buscar y Eliminar:
  - el valor se lee con `parsearValor`;
  - si falla, se carga un único paso `error` con su mensaje;
  - tras una inserción exitosa, el campo se vacía.
- [ ] **Paso 5:** Añadir los botones "Árbol de ejemplo" y "Limpiar":
  - "Árbol de ejemplo" hace `limpiar()` e inserta sin animación `valoresAleatoriosDistintos(enteroAleatorio(7, 10), 1, 99)`. Luego carga un paso `cambio-estructura` con `nodoId: null` y un mensaje como `Árbol de ejemplo: 23 8 61 …`.
  - "Limpiar" hace `limpiar()` y carga un paso `cambio-estructura` con el mensaje `Se vació el árbol`.
- [ ] **Paso 6:** Mientras `enCurso` sea verdadero, el campo y los botones de operación quedan desactivados.
- [ ] **Paso 7:** Verificar en el navegador:
  - Al insertar 50, 30, 70, 20, 40, 35 y 45, se ve cada comparación (`35 < 50 → izquierda`…) y cada nodo nuevo aparece en su sitio con su propio color.
  - Insertar 30 avisa de que está repetido; `a` avisa de que el tipo no coincide; `ab` avisa de que la entrada no es válida.
  - Buscar 35 resalta el camino y lo encuentra; buscar 99 termina en no encontrado.
  - Eliminar 30, que tiene dos hijos: el sucesor 35 sube conservando su color. Después, eliminar 40, que tiene un hijo, y luego 20, que es una hoja.
  - Al eliminar todo hasta vaciar el árbol, insertar `m` funciona, porque el tipo se liberó.
  - Pausa, Siguiente, las tres velocidades y Saltar al final funcionan, y los botones de operación siguen desactivados mientras dura la animación.
  - "Árbol de ejemplo" y "Limpiar" funcionan.
  - Foco de revisión 4: iniciar una inserción a velocidad lenta, ir a Teoría a mitad de la animación y volver. El árbol debe tener el nodo insertado y la consola del navegador no debe mostrar errores.
- [ ] **Paso 8:** Hacer commit con el mensaje `feat: simulador con insertar, buscar y eliminar`.

### Tarea 13: Simulador — recorridos y pila de llamadas

**Archivos:**
- Crear `src/componentes/PilaLlamadas/PilaLlamadas.jsx` y su CSS.
- Modificar `PaginaSimulador.jsx`.

**Produce:** `<PilaLlamadas pila={Array<{ nodoId, valor, funcion }>} />`:
- la cima va arriba y cada marco se muestra como `inorden(50)`;
- los marcos entran y salen con una animación de menos de 300 ms;
- si la pila está vacía, se muestra "La pila aparece durante los recorridos".

- [ ] **Paso 1:** Escribir `PilaLlamadas`.
- [ ] **Paso 2:** Añadir los botones Preorden, Inorden y Postorden, que llaman a `ejecutar(nombre, a => a[nombre]())`. Mientras la operación actual sea un recorrido, `Consola` recibe `salida = { nombre: 'Inorden', valores: visual.salida }`, con el nombre del recorrido que corresponda.
- [ ] **Paso 3:** Verificar con el árbol 50, 30, 70, 20, 40:
  - las salidas son `50 30 20 40 70` (preorden), `20 30 40 50 70` (inorden) y `20 40 30 70 50` (postorden);
  - en inorden, el puntero pasa por 50 y por 30 antes de imprimir 20;
  - la pila crece y luego se vacía;
  - los nodos impresos quedan marcados como visitados;
  - con el árbol vacío, cada recorrido muestra "El árbol está vacío".
- [ ] **Paso 4:** Hacer commit con el mensaje `feat: recorridos y pila de llamadas en el simulador`.

---

## Bloque 6: Teoría

### Tarea 14: Contenido de la teoría

**Archivos:** crear `src/modulos/teoria/conceptos.js`, que no lleva JSX, y `src/modulos/teoria/IconoArbol.jsx`.

**Consume:** las consultas de la Tarea 7.

**Produce:**
- `VALORES_EJEMPLO = [50, 30, 70, 20, 40, 60, 80, 65, 62]`.
- `CONCEPTOS`, un arreglo de objetos `{ id, titulo, definicion, icono: number[], resaltar(arbol) }`:
  - `resaltar(arbol)` devuelve `{ estados: Map, etiquetas?: Map, mostrarNiveles?: boolean, nota?: string }`;
  - los conceptos van en este orden y con estos `id`: `raiz`, `hijo`, `padre`, `hoja`, `hermanos`, `subarbol`, `altura`, `profundidad`, `nivel`, `grado`, `nodo-interno`, `ancestros-descendientes`, `abb` y `recorridos`;
  - cada `definicion` tiene de 1 a 3 frases sencillas, tuteando y siguiendo las convenciones de las restricciones globales. Los términos que tienen tarjeta propia se marcan como `[[id|texto]]`, por ejemplo `[[hijo|hijos]]`.
- `<IconoArbol marcados={number[]} />`: un pictograma SVG de 24×24 con un árbol de 7 puntos, numerados del 0 al 6 por niveles. Los puntos marcados se pintan con `--color-acento`.

- [ ] **Paso 1:** Escribir los 14 conceptos con este resaltado:

| id | resaltar |
|---|---|
| raiz | 50 destacado |
| hijo | 70 referencia; 60 y 80 destacados |
| padre | 65 referencia; 60 destacado |
| hoja | 20, 40, 62 y 80 destacados |
| hermanos | 20 referencia; 40 destacado |
| subarbol | 70, 60, 80, 65 y 62 destacados |
| altura | el camino 50, 70, 60, 65, 62 destacado; nota `Altura del árbol: 4` |
| profundidad | 65 referencia; 50, 70 y 60 secundarios; nota `Profundidad de 65: 3` |
| nivel | sin estados; `mostrarNiveles: true` |
| grado | sin estados; etiqueta `grado n` en cada nodo |
| nodo-interno | 50, 30, 70, 60 y 65 destacados |
| ancestros-descendientes | 60 referencia; 50 y 70 secundarios; 65 y 62 destacados |
| abb | sin estados; nota `Inorden: 20 30 40 50 60 62 65 70 80 → sale ordenado` |
| recorridos | sin estados; la tarjeta trae sus propios botones (Tarea 16) |

`resaltar` obtiene los ids con las consultas de la Tarea 7. Nunca usa ids escritos a mano.

- [ ] **Paso 2:** Escribir `IconoArbol`. El `icono` de cada concepto marca los puntos que sugieren la idea, por ejemplo `hoja: [3, 4, 5, 6]`.
- [ ] **Paso 3:** Verificar:

```bash
node --input-type=module -e "
import { ArbolBinario } from './src/logica/ArbolBinario.js';
import { CONCEPTOS, VALORES_EJEMPLO } from './src/modulos/teoria/conceptos.js';
const a = ArbolBinario.desdeValores(VALORES_EJEMPLO);
const n = new Map(a.instantanea().nodos.map(x => [x.id, x.valor]));
for (const c of CONCEPTOS) { const r = c.resaltar(a); console.log(c.id.padEnd(24), [...r.estados].map(([id, e]) => n.get(id) + ':' + e).join(' '), r.nota ?? '', r.mostrarNiveles ? 'niveles' : '', r.etiquetas ? r.etiquetas.size + ' etiquetas' : ''); }
"
```

Resultado esperado: 14 líneas que coinciden con la tabla del Paso 1 (el orden dentro de cada línea puede variar). La línea de `nivel` muestra `niveles` y la de `grado` muestra `9 etiquetas`.

- [ ] **Paso 4:** Hacer commit con el mensaje `feat: contenido y resaltados de la teoría`.

### Tarea 15: Página de teoría

**Archivos:**
- Crear `src/modulos/teoria/TarjetaConcepto.jsx` con su CSS, `src/modulos/teoria/TextoConEnlaces.jsx` y `PaginaTeoria.module.css`.
- Reemplazar `PaginaTeoria.jsx`.

**Produce:** `<TextoConEnlaces texto onIrA={(id) => void} />`, que convierte cada `[[id|texto]]` en un botón con aspecto de enlace.

- [ ] **Paso 1:** Escribir `PaginaTeoria`:
  - el árbol de ejemplo se crea una sola vez, con `useMemo`;
  - las tarjetas forman un acordeón con **una sola** abierta; al entrar está abierta `raiz`;
  - el reparto de la pantalla sigue los bocetos.
- [ ] **Paso 2:** Escribir `TarjetaConcepto`:
  - la cabecera lleva el `IconoArbol` y el título;
  - abierta, muestra la definición con `TextoConEnlaces`, una `VistaArbol` con `interactiva={false}` y `atenuarNoResaltados` alimentada con lo que devuelve `resaltar`, y la `nota`.
- [ ] **Paso 3:** Añadir las micro-animaciones con Motion, todas de menos de 300 ms:
  - abrir y cerrar la tarjeta con `layout`, en 250 ms;
  - al pasar el ratón, los puntos marcados del icono crecen, en 200 ms;
  - al abrir una tarjeta, los anillos resaltados hacen un pulso, en 280 ms.
- [ ] **Paso 4:** Verificar en el navegador:
  - cada tarjeta resalta lo indicado en la tabla de la Tarea 14;
  - solo hay una tarjeta abierta a la vez;
  - las animaciones se ven cortas;
  - con la opción de reducir movimiento activada en el sistema, no hay animaciones.
- [ ] **Paso 5:** Hacer commit con el mensaje `feat: página de teoría con tarjetas y micro-animaciones`.

### Tarea 16: Glosario dinámico y tarjeta de recorridos ⏸ revisión del usuario

**Archivos:** modificar `PaginaTeoria.jsx` y `TarjetaConcepto.jsx`.

- [ ] **Paso 1:** Añadir el buscador arriba de la lista:
  - filtra por título y por definición, sin las marcas `[[…]]`;
  - no distingue mayúsculas ni acentos (`normalize('NFD')` y quitar `\p{Diacritic}`);
  - si no hay resultados, muestra "No hay conceptos con «…»".
- [ ] **Paso 2:** Hacer que los enlaces funcionen. `onIrA(id)`:
  - limpia el filtro si la tarjeta destino está oculta;
  - abre esa tarjeta;
  - hace `scrollIntoView({ behavior: 'smooth', block: 'start' })` hasta ella.
- [ ] **Paso 3:** En la tarjeta `recorridos`, añadir los botones Preorden, Inorden y Postorden:
  - cada uno reproduce el recorrido sobre el árbol de ejemplo con su propio `useReproductor`, sin pila;
  - debajo se muestra una línea con la salida.
- [ ] **Paso 4:** Verificar:
  - buscar "subarbol" encuentra "Subárbol";
  - buscar "HOJA" incluye "Hoja" en los resultados;
  - al hacer clic en un enlace, se abre su tarjeta y la página baja hasta ella;
  - el inorden de la tarjeta da `20 30 40 50 60 62 65 70 80`.
- [ ] **Paso 5:** Hacer commit con el mensaje `feat: glosario dinámico y recorridos en la teoría`.
- [ ] **Paso 6:** ⏸ Pasar al usuario el título y la definición de cada concepto de `conceptos.js` para que los corrija. Aplicar sus cambios y hacer commit con el mensaje `docs: textos de teoría revisados`.

---

## Bloque 7: Quiz

### Tarea 17: Lógica del quiz

**Archivos:** crear `src/modulos/quiz/logicaQuiz.js`.

**Consume:** `ArbolBinario.desdeValores` y los recorridos; `enteroAleatorio` y `valoresAleatoriosDistintos`, de la Tarea 12.

**Produce:**
- las constantes `TOTAL_PREGUNTAS = 5` y `PUNTOS_POR_PREGUNTA = 2`;
- `generarPregunta() → { valores: number[], recorrido: 'preorden'|'inorden'|'postorden', correcta: number[] }`, con entre 5 y 9 valores distintos, cada uno entre 1 y 99;
- `parsearRespuesta(texto) → { ok: true, valores: number[] } | { ok: false, mensaje }`;
- `esCorrecta(valores, correcta) → boolean`;
- `calificar(aciertos: number) → number`.

- [ ] **Paso 1:** Escribir `parsearRespuesta`:
  - separa el texto con `/[\s,]+/` y descarta los trozos vacíos;
  - si no queda nada, el mensaje es `Escribe la secuencia de números.`;
  - si algún trozo no es un entero, el mensaje es `Usa solo números separados por espacios o comas.`.
- [ ] **Paso 2:** Escribir `esCorrecta` y `calificar`:
  - `esCorrecta` exige la misma longitud y los mismos valores en el mismo orden;
  - `calificar(aciertos)` devuelve `aciertos * PUNTOS_POR_PREGUNTA`.
- [ ] **Paso 3:** Verificar (foco de revisión 5):

```bash
node --input-type=module -e "
import { parsearRespuesta, esCorrecta, calificar, generarPregunta } from './src/modulos/quiz/logicaQuiz.js';
for (const t of ['20, 30 ,40,', ' 20   30,,40 ', '20 30 x', '', ' , ']) console.log(JSON.stringify(t), JSON.stringify(parsearRespuesta(t)));
console.log(esCorrecta([1, 2], [1, 2]), esCorrecta([1, 2, 3], [1, 2]), esCorrecta([2, 1], [1, 2]), calificar(3));
let ok = true; for (let i = 0; i < 200; i++) { const p = generarPregunta(); const s = new Set(p.valores);
  ok &&= p.valores.length >= 5 && p.valores.length <= 9 && s.size === p.valores.length && p.valores.every(v => v >= 1 && v <= 99) && p.correcta.length === p.valores.length && ['preorden', 'inorden', 'postorden'].includes(p.recorrido); }
console.log(ok);
"
```

Resultado esperado:
- Las dos primeras entradas dan `{"ok":true,"valores":[20,30,40]}`.
- `20 30 x` da el mensaje "Usa solo números…".
- `''` y `' , '` dan el mensaje "Escribe la secuencia…".
- La línea siguiente es `true false false 6`.
- La última línea es `true`.

- [ ] **Paso 4:** Hacer commit con el mensaje `feat: lógica del quiz`.

### Tarea 18: Página del quiz

**Archivos:** reemplazar `src/modulos/quiz/PaginaQuiz.jsx` y crear `PaginaQuiz.module.css`.

- [ ] **Paso 1:** Escribir la parte de preguntas:
  - las 5 preguntas se generan al entrar a la página y al pulsar "Intentar de nuevo";
  - cada pregunta muestra "Pregunta n de 5", el recorrido pedido y una `VistaArbol` del árbol (`ArbolBinario.desdeValores(valores).instantanea()`, con `interactiva={false}`);
  - hay un campo y un botón Responder, que también responde a Enter;
  - si `parsearRespuesta` falla, el mensaje aparece junto al campo y no se pasa a la siguiente pregunta;
  - durante el quiz no se dice si la respuesta fue correcta.
- [ ] **Paso 2:** Escribir la pantalla final:
  - muestra `Calificación: X / 10`;
  - muestra el repaso de cada pregunta: el árbol en pequeño, el recorrido, la respuesta del estudiante, la correcta y ✓ o ✗;
  - tiene un botón "Intentar de nuevo".
- [ ] **Paso 3:** Verificar en el navegador:
  - con 3 respuestas bien y 2 mal, la calificación es `6 / 10` y el repaso muestra las 5 preguntas;
  - con letras o con el campo vacío, no se avanza;
  - "Intentar de nuevo" trae árboles nuevos;
  - al ir a otro módulo y volver, el quiz empieza de nuevo.
- [ ] **Paso 4:** Hacer commit con el mensaje `feat: página del quiz con calificación y repaso`.
