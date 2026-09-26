# Sesión OMP Custom V3

Benchmark realizado el 26-09-2026 desde un directorio vacío, con el prompt original de Three.js. La medición corresponde exclusivamente a [`OMP_CUSTOM_V3_raw.html`](./OMP_CUSTOM_V3_raw.html), la salida entregada por OMP. [`OMP_CUSTOM_V3.html`](./OMP_CUSTOM_V3.html) es una versión refinada después de cerrar esa sesión; su trabajo adicional no se incluye en estas métricas.

- **ID de sesión medida:** `01a0df50-a845-764b-89e6-428d784b3ccc`
- **Modelo:** GPT-6 Luna, esfuerzo `max`, servicio `priority`
- **Intervalo medido:** `2026-09-26 20:03:21.935Z` a `20:30:21.643Z` (primer evento de la solicitud hasta la última respuesta del asistente)
- **Duración:** 1.619,7 s (27,0 min)
- **Solicitudes al modelo:** 83
- **Llamadas a herramientas:** 88 (`eval` 38, `read` 17, `edit` 14, `grep` 9, `todo` 4, `glob` 2, `write` 2, `bash` 2)
- **Compactaciones:** 0
- **Delegación al designer en la sesión medida:** ninguna

| Tokens y coste registrados | V3 bruto |
| --- | ---: |
| Entrada fresca | 507.238 |
| Lectura de caché | 7.437.568 |
| Salida | 86.158 |
| Razonamiento (subconjunto de salida) | 54.315 |
| Total | 8.030.964 |
| Cache hit sobre entrada | 93,62 % |
| Coste registrado | USD 0,3364 |

## Comparación con V2

| Métrica | V2 | V3 bruto | Cambio |
| --- | ---: | ---: | ---: |
| Duración de sesión* | 49,5 min | 27,0 min | −45,4 % |
| Solicitudes | 113 | 83 | −26,5 % |
| Herramientas | 139 | 88 | −36,7 % |
| Tokens totales | 14.329.801 | 8.030.964 | −44,0 % |
| Coste registrado | USD 0,5133 | USD 0,3364 | −34,5 % |

\* La duración V2 aquí usa el intervalo completo entre el primer mensaje y la última respuesta (2.968,1 s), con la misma definición aplicada a V3. Los 47,1 min de «tiempo útil» publicados más abajo en el README son otra métrica.

## Alcance y resultado visual

La salida bruta recuperó una identidad visual más clara, pero permitía abrir Cloudworks en Mango Tide, completaba las mejoras con un solo clic y tenía controles móviles de 37×38 px. La versión refinada corrige esas tres cosas y mejora cielo, estrellas y ventanas iluminadas. Las capturas muestran **la versión refinada**: [escritorio](./OMP_CUSTOM_V3_desktop.png) y [móvil](./OMP_CUSTOM_V3_mobile.png).

La [verificación funcional](./OMP_CUSTOM_V3_verification.json) cubrió ida y vuelta, puertas, bloqueo del taller en tránsito y Mango Tide, dos mejoras, salida del taller, vista móvil de 390×844 sin desbordamiento horizontal y consola sin errores. Para acortar el recorrido, el arnés adelantó el estado a cada llegada mediante una copia instrumentada; las transiciones de puertas, taller y mejoras se probaron con los controles. El HTML necesita conexión para cargar Three.js desde jsDelivr.

En una sesión posterior se solicitó explícitamente una revisión al `designer`, pero quedó esperando más de 14 minutos y se interrumpió. Tampoco pertenece a las cifras de V3. Esta es una sola ejecución por versión; la diferencia observada no demuestra por sí sola que los cambios de configuración causaran la mejora. Véase el [análisis V2–V3](./benchmark_V2_V3.md) para contexto y límites.
