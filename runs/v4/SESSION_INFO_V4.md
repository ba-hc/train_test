# Sesión OMP Custom V4

Benchmark del 26-09-2026 con el mismo prompt original de Three.js, desde un directorio vacío. [`OMP_CUSTOM_V4.html`](./OMP_CUSTOM_V4.html) es exactamente el HTML que entregó OMP al cerrar la sesión; no hubo refinamiento posterior del archivo publicado.

- **ID:** `01a0dfbe-1c7a-76be-8d25-f7a6a25ce78a`
- **Modelo:** GPT-6 Luna `max`, servicio `priority` durante toda la sesión; sin cambio de modelo ni compactación
- **Intervalo medido:** `2026-09-26T22:02:55.424Z`–`22:43:18.832Z`
- **Duración de sesión:** 2.423,4 s = **40,4 min** (primer mensaje hasta última respuesta)
- **Tiempo del proceso completo:** 2.426,2 s; exit code 0
- **SHA-256 del HTML medido:** `5bb42f7a5d30660a9b401b898cd61bc15fb3e220c17571fdf30443beb74faa0d`

| Uso registrado | V4 |
| --- | ---: |
| Solicitudes al modelo | 104 |
| Herramientas | 103 |
| Entrada fresca | 335.720 tokens |
| Lectura de caché | 11.100.544 tokens |
| Salida | 105.097 tokens |
| Razonamiento (incluido en salida) | 63.586 tokens |
| Total | 11.541.361 tokens |
| Cache hit sobre entrada | 97,06 % |
| Coste registrado | USD 0,3943 |
| Compactaciones | 0 |

| Herramienta | Llamadas |
| --- | ---: |
| `eval`/browser | 47 |
| `edit` | 16 |
| `read` | 14 |
| `grep` | 14 |
| `todo` | 5 |
| `bash` | 3 |
| `glob` | 2 |
| `write` | 2 |

Luna leyó `frontend-design` y `threejs`. No leyó `web-design-guidelines`, pese a la regla que pedía usarla una vez, ni invocó al `designer` (ahora opcional). Los datos estructurados están en [`OMP_CUSTOM_V4_session.json`](./OMP_CUSTOM_V4_session.json). No se publica el JSONL completo de la sesión porque contiene rutas y material de la configuración local.
