# Benchmark Three.js — Codex Desktop vs OMP Fast vs OMP Custom

Benchmark comparativo de tres ejecuciones del mismo tipo de tarea: construir un juego 3D de tranvía aéreo/coastal line con Three.js en un único HTML.

**Snapshot evaluado:** 25-09-2026  
**Commit base antes de este informe:** `31bc8c9bfb9f7a0354eff1c9105f2bb989357724`

> La evaluación es estricta y separa dos cosas que no deben confundirse: **calidad del artefacto final** y **eficiencia del harness que lo produjo**. El prompt original completo no está versionado en este repositorio, por lo que no se asignan puntos por requisitos ocultos que no puedan comprobarse en los tres archivos.

## Archivos auditados

| Archivo | Rol | Tamaño GitHub | Líneas | Observación |
|---|---|---:|---:|---|
| [`train_codex.html`](./train_codex.html) | Codex Desktop | 48.0 KB | 406 | Implementación más compacta |
| [`OMP_fast_mode.html`](./OMP_fast_mode.html) | OMP Fast Mode | 61.4 KB | 720 | Implementación orientada a velocidad/performance |
| [`OMP_custom.html`](./OMP_custom.html) | OMP Custom | 77.5 KB | 902 | Implementación más completa en UX y accesibilidad |
| [`SESSION_INFO.md`](./SESSION_INFO.md) | Telemetría Codex | 1.1 KB | — | Base de métricas para Codex |

## Resumen ejecutivo

- **Mejor HTML final:** **OMP Custom**. Tiene el flujo de juego más coherente, la interfaz más trabajada, mejor accesibilidad y el workshop mejor integrado.
- **Mejor punto medio calidad/tiempo:** **OMP Fast Mode**. Cede algo de fidelidad visual para usar instancing, DPR bajo, sin sombras y sin antialiasing, pero conserva bastante profundidad jugable.
- **Mejor eficiencia del harness:** **Codex Desktop**. Terminó mucho antes y con una fracción de los tokens, produciendo un artefacto claramente más simple pero todavía sustancial.
- **Ganador compuesto de este benchmark:** **OMP Custom**, por margen estrecho, porque el scoring final pondera 70% el artefacto y 30% la ejecución. Si la prioridad fuera coste/latencia, Codex pasaría al primer lugar.

## Telemetría normalizada

Para Codex, `SESSION_INFO.md` registra los tokens cacheados dentro de la entrada total. Para comparar de forma homogénea:

- **Input fresco Codex:** 157,885
- **Cache read Codex:** 4,066,944
- **Output Codex:** 63,381
- **Total comparable:** 4,288,210

| Métrica | Codex Desktop | OMP Fast Mode | OMP Custom |
|---|---:|---:|---:|
| Modelo principal | GPT-6 Luna max | GPT-6 Luna max | GPT-6 Luna max |
| Fallback | No reportado | No | GPT-5.6 Luna |
| Tiempo útil comparable | **24.6 min** | **33.2 min** | **61.2 min** ventana de generación |
| Tiempo activo de generación | no desglosado | no desglosado | 56.0 min |
| Wall total | 24.6 min | 33.5 min | 135.8 min, incluye 74.4 min idle |
| Requests / turnos | 47 requests | 102 respuestas con usage | 75 turnos |
| Tool calls | no desglosado | 103 | 93 |
| Input fresco | **157,885** | 251,901 | 807,979 |
| Cache read | 4,066,944 | **10,620,544** | 8,313,216 |
| Output | 63,381 | 94,400 | **94,978** |
| Reasoning | 32,969 (52.0% output) | **63,243 (67.0%)** | 57,002 (60.0%) |
| Total tokens | **4,288,210** | 10,966,845 | 9,216,173 |
| Cache hit | 96.3% | **97.68%** | 91.14% |
| Contexto | 130,525 final | no reportado | 179,311 pico |
| Cierre | no reportado | **SIGTERM** | normal / dispose |
| Coste registrado | no disponible | $0.357191 | $0.312735 |
| Coste comparable estimado* | ~$0.1763 | $0.357191 | $0.312735 |

* La cifra de Codex no aparece en `SESSION_INFO.md`. Se estima únicamente para comparación aplicando las mismas tarifas implícitas en la telemetría de OMP Fast para GPT-6 Luna priority: $0.20/M input fresco, $0.02/M cache read y $1.00/M output. No representa necesariamente un cobro real de suscripción.

### Lectura de eficiencia

Respecto de Codex:

- OMP Fast consumió **2.56×** los tokens totales, **1.60×** el input fresco y tardó **1.35×** más.
- OMP Custom consumió **2.15×** los tokens totales, **5.12×** el input fresco y tardó **2.49×** más en su ventana real de generación.
- El output de OMP Fast y OMP Custom fue casi idéntico (~94–95K), alrededor de **1.5×** el de Codex.
- OMP Custom sufrió una penalización especialmente fuerte al cambiar de GPT-6 Luna a GPT-5.6 Luna: los primeros turnos tras el fallback reingirieron grandes bloques de contexto y su hit rate global cayó a 91.14%.
- OMP Fast tuvo el mejor hit rate, pero también el mayor tráfico total de caché: **10.62M tokens** de cache read. Un hit rate alto no significa automáticamente una ejecución barata.
- El cierre por SIGTERM de OMP Fast es una señal de robustez peor que el cierre normal de OMP Custom, aunque el artefacto resultante sí está completo.

## Auditoría técnica de los tres HTML

### 1. Codex Desktop — `train_codex.html`

**Lo mejor**

- Escena ambiciosa para 406 líneas: cielo con shader, luna, estrellas, mar, dos islas, puente, tranvía con pasajeros, clima, comfort scoring y workshop.
- Tiene manejo global de `error` y `unhandledrejection`, algo que los otros dos no implementan como estado explícito de startup.
- Controles de teclado y pointer, resize correcto, delta time acotado y tone mapping ACES.
- El código es el más compacto y consigue mucho gameplay por unidad de tiempo/tokens.

**Problemas estrictos**

1. **Se puede salir del terminal sin completar el flujo de puertas.** Al pulsar W estando `docked`, `depart()` se ejecuta incluso si nunca se abrieron las puertas ni terminó el boarding inicial.
2. **El workshop puede abrirse en cualquier momento**, incluso a mitad de la ruta. `workshop()` no comprueba estación, velocidad ni `docked`.
3. **Hay una asignación de HUD contradictoria:** en el loop se asigna `comfortDot` a la posición normalizada de la ruta y acto seguido `updateUI()` vuelve a asignarlo al porcentaje de comfort. El valor de progreso queda efectivamente pisado.
4. **Accesibilidad débil:** no hay regiones `aria-live`, labels ARIA relevantes, `:focus-visible` ni `prefers-reduced-motion`.
5. **Performance gráfica poco escalable:** no usa `InstancedMesh`; estrellas, nubes y muchos props son meshes individuales. Además usa antialiasing y DPR hasta 1.8 mientras las sombras no están habilitadas.
6. El estado está repartido en muchas variables globales; funciona, pero la mantenibilidad cae frente a un objeto de estado centralizado.

**Veredicto del HTML:** gran densidad de ideas y excelente eficiencia de generación, pero los atajos en lógica de estación/workshop y accesibilidad son demasiado visibles para una nota alta de producto.

### 2. OMP Fast Mode — `OMP_fast_mode.html`

**Lo mejor**

- Es la implementación más orientada a rendimiento:
  - `InstancedMesh` para nubes, sleepers y soportes.
  - estrellas como `Points`.
  - DPR limitado a 1.15.
  - antialiasing y shadow map desactivados.
  - materiales Lambert para buena parte de la escena.
- Tiene tres cámaras: **Follow, Wide y Cabin**.
- Mejor semántica que Codex: `aria-live`, labels de botones, regiones con nombre y un announcer screen-reader-only.
- Gameplay más ordenado: fases `dwell`, `ready` y `travel`, pasajeros que suben/bajan, comfort, crosswind, tips, streak y workshop.
- El workshop tiene progreso y transición de salida automática.

**Problemas estrictos**

1. **Cloudworks es accesible también desde Mango Tide.** El botón se muestra en cualquier `ready/dwell` y `showWorkshop()` no exige `currentStation === 0`, aunque el propio texto lo describe como la isla de Oliver/Saltlight.
2. **El HUD de ruta puede quedar semánticamente atrasado durante la parada.** `updateRouteLabels()` deriva origen/destino de `direction`, pero `direction` no cambia al llegar; se actualiza al abandonar la estación.
3. El ahorro de rendimiento se paga visualmente: sin AA, sin sombras y DPR 1.15, la escena debería ser más estable pero menos limpia que las otras dos en pantallas densas.
4. No implementa `:focus-visible` específico ni `prefers-reduced-motion`.
5. No hay UI explícita de fallo de startup si el CDN de Three.js falla.
6. Hay propiedades `castShadow/receiveShadow` asignadas aunque `shadowMap.enabled=false`; no rompe nada, pero añade ruido de implementación.

**Veredicto del HTML:** el mejor compromiso técnico. Es bastante más completo que Codex y mucho más eficiente gráficamente que Custom, pero conserva dos inconsistencias de estado importantes.

### 3. OMP Custom — `OMP_custom.html`

**Lo mejor**

- La interfaz más madura del benchmark:
  - ARIA abundante y coherente.
  - `aria-live` para estados.
  - `role="progressbar"` con valores.
  - `:focus-visible`.
  - media queries para ancho y altura.
  - `prefers-reduced-motion` para la capa CSS.
- El flujo jugable es el más coherente:
  - llegada → abrir puertas → intercambio → estación detenida → salida.
  - Cloudworks solo se ofrece en Saltlight.
  - dos upgrades secuenciales e interactivos.
  - estado centralizado en un objeto `state`.
- Mejor sistema visual/UI: SVGs propios, route marker, comfort, condiciones, panel de llegada, workshop y estados de botones.
- Usa instancing para nubes, sleepers y pilares, evitando el peor problema de draw calls de Codex.
- Mantiene sombras, antialiasing, ACES, una escena más grande y más detalle geométrico.

**Problemas estrictos**

1. **La animación de `waterShimmer` usa un material compartido.** El loop modifica `ring.material.opacity` por anillo, pero todos apuntan al mismo `shimmerMat`; la última asignación domina visualmente y se pierde el desfase individual buscado.
2. `prefers-reduced-motion` solo frena CSS; la cámara, el tranvía, los anillos y el mundo Three.js siguen animándose.
3. Es la opción gráficamente más cara: antialiasing, DPR hasta 1.65, shadow map PCF soft y una escena más compleja.
4. Tiene más dependencias de red: Three.js más Google Fonts. Si el import falla, no hay un estado de error explícito equivalente al de Codex.
5. `workshopComplete` aparece como propiedad dinámica y no está inicializada en el objeto de estado; es un detalle menor de mantenibilidad.
6. Sigue siendo un monolito de 902 líneas: el estado está mejor organizado, pero la separación de responsabilidades ya empieza a justificar módulos si esto dejara de ser un benchmark single-file.

**Veredicto del HTML:** es el artefacto más cercano a un pequeño producto y no solo a una demo técnica. Sus defectos son más de polish/performance que de flujo principal.

## Comparación estructural

| Característica | Codex | OMP Fast | OMP Custom |
|---|---:|---:|---:|
| Three.js | 0.160.0 | 0.165.0 | 0.162.0 vía import map |
| IDs/UI state hooks | 34 | 39 | **50** |
| Botones declarados | 6 | **8** | 7 |
| SVG UI propios | 0 | 0 | **10** |
| `InstancedMesh` en código | 0 | **4** | 3 |
| Antialias | Sí | **No** | Sí |
| DPR máximo | 1.8 | **1.15** | 1.65 |
| Shadow map | No | **No** | Sí |
| Varias cámaras/vistas | Lean lateral | **3 vistas** | View trim |
| ARIA relevante | No | Sí | **Sí, más completa** |
| `:focus-visible` | No | No | **Sí** |
| Reduced motion | No | No | **Sí, solo CSS** |
| Startup error UI | **Sí** | No | No |
| Workshop restringido correctamente a Saltlight | No | No | **Sí** |
| Estado centralizado | No | Parcial | **Sí** |

## Scoring estricto del artefacto

Pesos:

- Dirección visual: 20%
- Gameplay/completitud: 25%
- UX/controles: 15%
- Accesibilidad/responsive: 15%
- Calidad técnica/performance: 15%
- Robustez/mantenibilidad: 10%

| Categoría | Codex | OMP Fast | OMP Custom |
|---|---:|---:|---:|
| Visual / art direction | 8.4 | 8.1 | **9.1** |
| Gameplay / completitud | 7.6 | 8.5 | **9.0** |
| UX / controles | 7.4 | 8.2 | **9.0** |
| Accesibilidad / responsive | 4.2 | 7.7 | **9.2** |
| Técnica / performance | 6.5 | **8.8** | 8.1 |
| Robustez / mantenibilidad | 7.4 | 8.1 | **8.7** |
| **Nota HTML ponderada** | **7.0 / 10** | **8.3 / 10** | **8.9 / 10** |

## Scoring del harness

Pesos:

- Tiempo: 30%
- Eficiencia de tokens: 25%
- Comportamiento de caché: 15%
- Estabilidad/recuperación: 15%
- Economía de iteraciones/herramientas: 15%

| Harness | Tiempo | Tokens | Caché | Estabilidad | Iteración | Nota ejecución |
|---|---:|---:|---:|---:|---:|---:|
| Codex Desktop | 10.0 | 10.0 | 9.6 | 9.0 | 9.2 | **9.7 / 10** |
| OMP Fast Mode | 8.3 | 5.5 | **9.8** | 6.8 | 6.5 | **7.3 / 10** |
| OMP Custom | 4.8 | 6.2 | 7.5 | 7.2 | 7.2 | **6.3 / 10** |

### Interpretación

**Codex Desktop** es el ganador claro de eficiencia. Generó un resultado de 48 KB en 24.6 minutos usando solo 4.29M tokens totales. Su problema no es el harness: es que dejó más deuda de producto en el HTML.

**OMP Fast Mode** gastó muchísimo más contexto para ganar ~1.2 puntos de calidad de artefacto sobre Codex. Aun así, su arquitectura gráfica sí demuestra trabajo extra útil: instancing, tres vistas y una máquina de estados mejor definida. Es el “sweet spot” si se busca subir calidad sin llegar al tiempo de Custom.

**OMP Custom** produjo el mejor HTML, pero el harness pagó demasiado por ello. El fallback de modelo destruyó parte del beneficio del prompt cache y elevó el input fresco a 807,979 tokens. La sesión se recuperó correctamente, pero una política de fallback que preserve modelo/cache o espere/reintente antes de cambiar habría mejorado mucho este resultado.

## Nota final

La nota final pondera **70% calidad del HTML + 30% ejecución del harness**.

| Puesto | Harness | HTML | Ejecución | **Nota final** |
|---:|---|---:|---:|---:|
| **1** | **OMP Custom** | 8.9 | 6.3 | **8.1 / 10** |
| **2** | **OMP Fast Mode** | 8.3 | 7.3 | **8.0 / 10** |
| **3** | **Codex Desktop** | 7.0 | 9.7 | **7.8 / 10** |

La distancia entre los tres es menor de lo que sugieren los tokens:

- **Si solo importa el producto final:** OMP Custom.
- **Si importa el punto medio entre calidad y tiempo:** OMP Fast Mode.
- **Si importa producir rápido, gastar poca cuota y luego iterar:** Codex Desktop.

Para un workflow real de SWE, la conclusión más importante de este benchmark no es “usar siempre el ganador”. Es que **Codex obtiene la mayor parte del valor con una fracción del presupuesto**, mientras OMP demuestra su ventaja cuando realmente aprovecha las iteraciones adicionales para cerrar UX, accesibilidad y lógica de producto. OMP Custom solo justifica su sobrecoste cuando ese último ~10–20% de polish importa.

## Auditoría de `SESSION_INFO.md`

El archivo es conciso e internamente consistente: separa entrada total, cacheada/no cacheada, output y reasoning, e informa duración y contexto final.

Para convertirlo en telemetría de benchmark completa le faltan:

- coste registrado,
- distribución de herramientas,
- TTFT,
- stop/close reason,
- contexto pico,
- errores/reintentos,
- duración activa vs idle,
- número de tool calls.

**Nota como metadata de benchmark: 7.5 / 10.** Es suficiente para comparar el baseline de Codex, pero menos granular que los reportes disponibles para OMP.
