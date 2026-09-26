# Benchmark Three.js — Codex Desktop vs OMP Fast vs OMP Custom vs OMP Custom V2

Benchmark comparativo de cuatro ejecuciones del mismo tipo de tarea: construir un juego 3D de tranvía aéreo/coastal line con Three.js en un único HTML.

**Snapshot actualizado:** 26-09-2026  
**Commit que añadió V2:** `7d341897e4fc076fca371da34af7bc6369083c23`

> La evaluación es estricta y separa dos cosas que no deben confundirse: **calidad del artefacto final** y **eficiencia del harness que lo produjo**. La misma rúbrica y los mismos pesos de la evaluación anterior se conservan para no cambiar el criterio después de ver V2.

## Archivos auditados

| Archivo | Rol | Tamaño GitHub | Líneas | Observación |
|---|---|---:|---:|---|
| [`train_codex.html`](./train_codex.html) | Codex Desktop | 48.0 KB | 406 | Implementación más compacta |
| [`OMP_fast_mode.html`](./OMP_fast_mode.html) | OMP Fast Mode | 61.4 KB | 720 | Prioriza rendimiento gráfico |
| [`OMP_custom.html`](./OMP_custom.html) | OMP Custom V1 | 77.5 KB | 902 | Mejor flujo de workshop de V1 |
| [`OMP_CUSTOM_V2.html`](./OMP_CUSTOM_V2.html) | OMP Custom V2 | 77.6 KB | 1,122 | Harness endurecido, UI/escena rehechas |
| [`SESSION_INFO.md`](./SESSION_INFO.md) | Telemetría Codex | 1.1 KB | — | Base de métricas para Codex |

## Resumen ejecutivo

- **Mejor HTML según la rúbrica estricta:** **OMP Custom V1** (8.9 vs 8.8). V1 conserva mejor gameplay y performance; **V2** gana en presentación, audio, responsive/safe-area y robustez de startup.
- **Mejor gameplay/workshop:** **OMP Custom V1**. Cloudworks está restringido correctamente a Saltlight y las dos mejoras requieren interacción secuencial; V2 automatiza el refit.
- **Mejor rendimiento gráfico:** **OMP Fast Mode**. Mantiene el uso más disciplinado de instancing, Points, DPR bajo y escena barata.
- **Mejor eficiencia de harness:** **Codex Desktop**, todavía por amplio margen.
- **Mejor estabilidad del OMP custom:** **V2**. Ya no cambia de modelo, Luna permanece en priority y el hit rate sube a 96.25%.
- **Ganador compuesto con la ponderación histórica 70% artefacto / 30% ejecución:** **OMP Custom V2**, pero por un margen pequeño. Si la eficiencia recibe más peso, Codex y OMP Fast pasan a ser más atractivos.

---

# Telemetría normalizada

Para Codex, `SESSION_INFO.md` registra los tokens cacheados dentro de la entrada total. Para compararlo homogéneamente:

- **Input fresco Codex:** 157,885
- **Cache read Codex:** 4,066,944
- **Output Codex:** 63,381
- **Total comparable:** 4,288,210

| Métrica | Codex Desktop | OMP Fast | OMP Custom V1 | **OMP Custom V2** |
|---|---:|---:|---:|---:|
| Modelo principal | GPT-6 Luna max | GPT-6 Luna max | GPT-6 Luna max | **GPT-6 Luna max** |
| Service tier | no reportado | priority | priority / luego fallback | **priority toda la sesión** |
| Cambio de modelo | no reportado | No | **Sí: GPT-6 → GPT-5.6** | **No reportado** |
| Tiempo útil comparable | **24.6 min** | 33.2 min | 61.2 min | **47.1 min** |
| Requests / turnos | 47 | 102 | 75 | **113** |
| Tool calls | no desglosado | 103 | 93 | **139** |
| Input fresco | **157,885** | 251,901 | 807,979 | **532,730** |
| Cache read | 4,066,944 | 10,620,544 | 8,313,216 | **13,663,616** |
| Output | 63,381 | 94,400 | 94,978 | **133,455** |
| Reasoning | 32,969 | 63,243 | 57,002 | **88,474** |
| Total tokens | **4,288,210** | 10,966,845 | 9,216,173 | **14,329,801** |
| Cache hit | 96.3% | **97.68%** | 91.14% | **96.25%** |
| Contexto pico | 130,525 final | no reportado | 179,311 | **228,822** |
| Compaction | no reportado | no reportado | 0 | **remote, 1 outlier de 430.5 s** |
| Coste registrado | no disponible | $0.3572 | $0.3127 | **$0.5133** |
| Coste comparable estimado* | ~$0.1763 | $0.3572 | $0.3127 | $0.5133 |

* La cifra de Codex no aparece en `SESSION_INFO.md`; es una estimación histórica aplicando las mismas tarifas implícitas usadas en la primera evaluación. No representa necesariamente cobro real de suscripción.

## Qué cambió realmente de Custom V1 a V2

| Métrica | V1 | V2 | Cambio |
|---|---:|---:|---:|
| Tiempo útil | 61.2 min | **47.1 min** | **-23.0%** |
| Input fresco | 807,979 | **532,730** | **-34.1%** |
| Cache hit | 91.14% | **96.25%** | **+5.11 pp** |
| Cache read | 8.31M | **13.66M** | **+64.4%** |
| Output | 94,978 | **133,455** | **+40.5%** |
| Tokens totales | 9.22M | **14.33M** | **+55.5%** |
| Coste | $0.3127 | **$0.5133** | **+64.1%** |
| LLM calls | 75 | **113** | **+50.7%** |
| Tool calls | 93 | **139** | **+49.5%** |

### Interpretación

El hardening resolvió el problema correcto:

- ya no hubo una migración GPT-6 → GPT-5.6;
- Luna permaneció en `priority`;
- el cache hit volvió a un nivel sano;
- el input fresco cayó de 808K a 533K;
- el tiempo útil bajó unos 14 minutos.

Pero **V2 no es más económico en tokens**. El agente iteró mucho más. La caché funciona mejor, pero se reutiliza sobre un contexto cada vez mayor durante 113 llamadas. Por eso el cache read sube a 13.66M tokens y se convierte en el mayor componente del coste.

La lección es importante: **mejor cache hit no equivale a menor gasto absoluto**. V2 arregló el churn de caché; el siguiente cuello de botella es el número de iteraciones sobre contexto grande.

### Remote compaction

V2 alcanzó un pico de **228,822 tokens**, coherente con la nueva política de contexto. La remote compaction produjo el outlier de **430.5 s**. Después, los turns reiniciaron aproximadamente desde ~23K cache read en vez de ~230K.

Eso tiene dos efectos opuestos:

1. evita seguir arrastrando indefinidamente un contexto enorme;
2. rompe la continuidad del cache caliente e introduce un cliff grande de latencia.

Por tanto, la compaction funcionó como mecanismo de seguridad, pero no fue gratuita.

---

# Tool usage

| Herramienta | OMP Fast | OMP Custom V1 | **OMP Custom V2** |
|---|---:|---:|---:|
| read | 35 | 39 | **45** |
| eval/browser | 40 | 20 | **43** |
| edit | 18 | 6 | **23** |
| grep | — | 2 | **13** |
| bash | 3 | 3 | **7** |
| todo | 4 | 21 | **5** |
| glob | 1 | — | **2** |
| write | 2 | 2 | **1** |
| **Total** | **103** | **93** | **139** |

V2 muestra un loop de trabajo razonable de **read → edit → browser/eval**, pero también mucha más iteración que V1. Para un único HTML, 45 reads, 43 evals y 23 edits son suficientes para explicar gran parte del aumento de cache traffic.

---

# Auditoría estricta de los cuatro HTML

## 1. Codex Desktop — `train_codex.html`

### Fortalezas

- Mucha funcionalidad en solo 406 líneas.
- Cielo con shader, estrellas, mar, dos islas, tranvía, pasajeros, clima, comfort, scoring y workshop.
- Manejo explícito de `error` y `unhandledrejection`.
- Controles keyboard/pointer, resize y delta time.
- Menor coste de generación del benchmark.

### Problemas

1. Se puede abandonar el terminal sin completar adecuadamente el flujo de puertas.
2. El workshop puede abrirse a mitad de la ruta.
3. Una asignación del HUD de comfort/progreso es pisada inmediatamente por otra.
4. Accesibilidad débil: sin ARIA relevante, `:focus-visible` ni reduced motion.
5. No usa instancing; muchas entidades son meshes individuales.
6. Estado muy disperso en variables globales.

**HTML:** 7.0/10.

---

## 2. OMP Fast Mode — `OMP_fast_mode.html`

### Fortalezas

- Mejor arquitectura gráfica para rendimiento:
  - `InstancedMesh` para nubes/sleepers/soportes;
  - estrellas con `Points`;
  - DPR 1.15;
  - sin shadow map activo;
  - antialias desactivado;
  - materiales económicos.
- Tres vistas: Follow, Wide y Cabin.
- Buen sistema de fases, pasajeros, comfort, crosswind, tips y workshop.
- ARIA bastante mejor que Codex.

### Problemas

1. Cloudworks también es accesible desde Mango Tide.
2. El HUD puede conservar origen/destino atrasados durante la parada.
3. El ahorro de GPU reduce fidelidad visual.
4. No tiene `:focus-visible` ni reduced motion.
5. No tiene estado de startup failure equivalente a Codex/V2.
6. Hay flags de shadows asignados aunque el shadow map esté desactivado.

**HTML:** 8.3/10.

---

## 3. OMP Custom V1 — `OMP_custom.html`

### Fortalezas

- Flujo de estación más coherente.
- Cloudworks solo se ofrece correctamente en Saltlight.
- Dos upgrades secuenciales e interactivos.
- ARIA, progressbar, `:focus-visible`, media queries y reduced-motion CSS.
- Instancing para nubes, sleepers y pilares.
- Estado centralizado.
- Excelente dirección visual y UX.

### Problemas

1. `waterShimmer` comparte material: el intento de variar opacity por anillo termina sobrescribiendo el mismo material.
2. Reduced-motion solo afecta CSS; el mundo Three.js sigue animándose.
3. Antialias + DPR 1.65 + PCF shadows hacen la escena relativamente cara.
4. Depende de Three.js y Google Fonts sin una UI explícita de error de CDN.
5. Sigue siendo un monolito grande.

**HTML:** 8.9/10.

---

## 4. OMP Custom V2 — `OMP_CUSTOM_V2.html`

V2 no es un retoque de V1: el HTML está rehecho. Mantiene un tamaño parecido (~77.5 KB), pero cambia estructura, UI y escena.

### Fortalezas

1. **Mejor presentación integral.**
   - HUD rediseñado.
   - safe-area support.
   - responsive específico a 800, 600 y 380 px.
   - vista de workshop separada.
   - estética muy consistente entre ruta y Cloudworks.

2. **Workshop técnicamente más ambicioso.**
   - segunda `THREE.Scene`;
   - `OrthographicCamera`;
   - entorno de taller propio;
   - banco de trabajo, herramientas, iluminación y tram dedicado.

3. **Audio opcional.**
   - sonidos sintetizados mediante Web Audio;
   - desactivado por defecto;
   - degradación segura mediante `try/catch`.

4. **Mejor robustez de startup que V1/Fast.**
   Si Three.js no carga, muestra:
   `Three.js could not load. Check your internet connection and reopen this page.`

5. **Accesibilidad y mobile.**
   - 26 atributos ARIA;
   - regiones `aria-live`;
   - `:focus-visible`;
   - viewport-fit/safe areas;
   - controles pointer + keyboard;
   - reduced-motion CSS.

6. **Algunas decisiones de performance son buenas.**
   - DPR limitado a 1.35;
   - `shadowMap.autoUpdate=false`;
   - shadows se actualizan periódicamente en ruta, no cada frame;
   - una sola dependencia remota y sin Google Fonts.

### Problemas estrictos

1. **Cloudworks vuelve a ser accesible desde cualquier lugar.**

   `workshopButton` está visible durante la ruta y `enterWorkshop()` no valida:
   - `docked`;
   - estación Saltlight;
   - velocidad;
   - estado de service.

   Además, la tecla `V` abre el workshop de la misma forma. Se puede entrar en “Oliver Cloudworks · Home island” a mitad de la línea o estando en Mango Tide.

   V1 resolvía esto correctamente mostrando `Visit Cloudworks` solo cuando `currentStation === Saltlight Terminus`.

2. **El workshop perdió interacción respecto de V1.**

   V1 tenía:
   - `Fit leaves`;
   - luego `Prepare`;
   - dos pasos bloqueados secuencialmente.

   V2 convierte el proceso en una animación automática de ~7.1 s. Es más cinematográfico, pero menos jugable.

3. **Regresión de draw calls.**

   V2 no usa `InstancedMesh`. Casas, árboles, personas, sleepers, soportes, clouds y decoración se construyen con meshes individuales. V1 tenía 3 usos de instancing y Fast 4.

4. **Reduced motion sigue siendo parcial.**

   `prefers-reduced-motion` elimina animaciones/transiciones CSS, pero no detiene:
   - cámara;
   - sway;
   - tram;
   - workshop animation;
   - mundo Three.js.

5. **Shadow update del workshop puede quedar visualmente atrasado.**

   `renderer.shadowMap.autoUpdate=false`. Se marca `needsUpdate` al entrar/salir del workshop, pero el tram del taller se mueve y cambia durante el refit sin pedir una actualización continua de shadows.

6. **Three.js r128 es bastante más antiguo que las versiones usadas por los otros resultados.**

   No rompe esta implementación, pero deja V2 sobre una API legacy y sacrifica mejoras posteriores del renderer/ecosistema.

7. **Monolito de 1,122 líneas.**

   Para el requisito single-file es aceptable, pero ya es el artefacto más difícil de mantener del benchmark.

---

# Comparación estructural

| Característica | Codex | OMP Fast | OMP Custom V1 | **OMP Custom V2** |
|---|---:|---:|---:|---:|
| Líneas | 406 | 720 | 902 | **1,122** |
| Three.js | 0.160 | 0.165 | 0.162 | **r128** |
| UI IDs | 34 | 39 | **50** | 40 |
| Botones | 6 | 8 | 7 | **9** |
| SVG UI propios | 0 | 0 | **10** | 0 |
| `InstancedMesh` | 0 | **4** | 3 | **0** |
| `Points` | 0 | 1 | 1 | 1 |
| Escenas Three.js | 1 | 1 | 1 | **2** |
| Cámara ortográfica | No | No | No | **Sí, workshop** |
| Audio | No | No | No | **Sí** |
| Antialias | Sí | No | Sí | Sí |
| DPR máx. | 1.8 | **1.15** | 1.65 | **1.35** |
| Shadow map | No | No | Sí | **Sí, update manual** |
| ARIA | No | Sí | **Sí** | **Sí** |
| `:focus-visible` | No | No | Sí | **Sí** |
| Safe-area mobile | No | Sí | No | **Sí** |
| Reduced motion | No | No | CSS-only | **CSS-only** |
| Startup error UI | **Sí** | No | No | **Sí** |
| Workshop correctamente restringido a Saltlight | No | No | **Sí** | **No** |
| Workshop interactivo | Bajo | Medio | **Alto** | Medio/bajo |
| Estado centralizado | No | Parcial | **Sí** | **Sí** |

---

# Scoring estricto del artefacto

Se mantienen los pesos originales:

- Dirección visual: 20%
- Gameplay/completitud: 25%
- UX/controles: 15%
- Accesibilidad/responsive: 15%
- Calidad técnica/performance: 15%
- Robustez/mantenibilidad: 10%

| Categoría | Codex | OMP Fast | OMP Custom V1 | **OMP Custom V2** |
|---|---:|---:|---:|---:|
| Visual / art direction | 8.4 | 8.1 | 9.1 | **9.3** |
| Gameplay / completitud | 7.6 | 8.5 | **9.0** | 8.6 |
| UX / controles | 7.4 | 8.2 | 9.0 | **9.2** |
| Accesibilidad / responsive | 4.2 | 7.7 | 9.2 | **9.3** |
| Técnica / performance | 6.5 | **8.8** | 8.1 | 7.9 |
| Robustez / mantenibilidad | 7.4 | 8.1 | **8.7** | 8.6 |
| **Nota HTML ponderada** | **7.0** | **8.3** | **8.9** | **8.8 / 10** |

### Lectura

V2 no supera a V1 en todo. V1 conserva mejor gameplay y una escena más eficiente. V2 compensa con mejor presentación, mobile, audio, startup resilience y una experiencia de workshop visualmente más elaborada.

Por eso el resultado es **8.8 vs 8.9**: V2 es más “producto”, V1 está un poco mejor cerrado como juego.

---

# Scoring del harness

Se mantienen los pesos originales:

- Tiempo: 30%
- Eficiencia de tokens: 25%
- Comportamiento de caché: 15%
- Estabilidad/recuperación: 15%
- Economía de iteraciones/herramientas: 15%

| Harness | Tiempo | Tokens | Caché | Estabilidad | Iteración | **Nota ejecución** |
|---|---:|---:|---:|---:|---:|---:|
| **Codex Desktop** | **10.0** | **10.0** | 9.6 | 9.0 | **9.2** | **9.7 / 10** |
| **OMP Fast Mode** | 8.3 | 5.5 | **9.8** | 6.8 | 6.5 | **7.3 / 10** |
| OMP Custom V1 | 4.8 | 6.2 | 7.5 | 7.2 | 7.2 | **6.3 / 10** |
| **OMP Custom V2** | 6.8 | 4.4 | 9.4 | **9.3** | 5.2 | **6.7 / 10** |

### V2: por qué 6.7 y no más

**Sube mucho en estabilidad y caché:**
- un solo modelo;
- priority estable;
- 96.25% hit;
- input fresco notablemente menor que V1;
- compaction efectiva cuando el contexto supera el rango configurado.

**Pero baja en economía:**
- mayor total de tokens del benchmark;
- mayor coste registrado;
- 113 LLM calls;
- 139 tools;
- 43 browser evals;
- 23 edits para un único archivo;
- 430.5 s perdidos en remote compaction.

El hardening arregló **cache churn**, pero todavía no arregla **iteration churn**.

---

# Nota final

Se conserva la ponderación histórica:

**70% calidad del HTML + 30% ejecución del harness.**

| Puesto | Harness | HTML | Ejecución | **Nota final** |
|---:|---|---:|---:|---:|
| **1** | **OMP Custom V2** | 8.8 | 6.7 | **8.2 / 10** |
| **2** | **OMP Custom V1** | **8.9** | 6.3 | **8.1 / 10** |
| **3** | **OMP Fast Mode** | 8.3 | 7.3 | **8.0 / 10** |
| **4** | **Codex Desktop** | 7.0 | **9.7** | **7.8 / 10** |

La diferencia entre V2 y V1 es pequeña. V2 queda primero porque la pérdida mínima en gameplay/eficiencia gráfica se compensa por una ejecución mucho más estable y por mejoras de UX/robustez.

---

# Veredicto por objetivo

| Objetivo | Mejor opción |
|---|---|
| Máxima eficiencia de harness | **Codex Desktop** |
| Menor tiempo manteniendo calidad alta | **OMP Fast Mode** |
| Mejor gameplay/workshop | **OMP Custom V1** |
| Mejor UX/presentación general | **OMP Custom V2** |
| Mejor estabilidad del custom harness | **OMP Custom V2** |
| Mejor resultado compuesto 70/30 | **OMP Custom V2** |
| Mejor rendimiento gráfico | **OMP Fast Mode** |

## Conclusión

La iteración del harness **sí funcionó**, pero no de la manera “menos tokens = mejor”.

V1 tenía un problema grave de recuperación:

```text
GPT-6 Luna
→ upstream failure
→ GPT-5.6 fallback
→ cache invalidado
→ grandes reingestas de input fresco
```

V2 lo transforma en:

```text
GPT-6 Luna priority
→ misma identidad durante la sesión
→ 96%+ cache hit
→ contexto crece
→ remote compaction cerca del umbral
→ continúa en GPT-6 Luna
```

Eso es arquitectónicamente mejor.

El nuevo problema es distinto:

```text
demasiados turns
+ muchos reads
+ muchos browser evals
+ muchos edits
+ contexto grande y muy cacheado
= cache hit alto, pero 14.33M tokens procesados
```

Por tanto, **el siguiente salto no debería venir de más routing ni más context machinery**. Debería venir de hacer que Luna termine en menos ciclos: lecturas más dirigidas, browser checks agrupados y menos edición incremental.

No conviene intentar subir el cache hit mucho más: 96.25% ya es sano. El mayor potencial está en reducir **113 calls / 139 tools** sin perder el nivel visual conseguido por V2.
