# Sesión de Codex

- **ID:** `01a0d8a3-6665-7dd3-bff4-6b79a661b515`
- **Cliente:** Codex Desktop
- **Modelo y esfuerzo:** `gpt-6-luna`, `max`
- **Servicio:** priority (Fast 1.5x)
- **Turnos:** 1 `root_turn_id`; sin subagentes ni branches

## Uso de tokens

47 solicitudes en total.

| Métrica | Tokens | Proporción |
| --- | ---: | ---: |
| Total | 4,288,210 | |
| Entrada | 4,224,829 | |
| Entrada cacheada | 4,066,944 | 96.3% de la entrada |
| Entrada no cacheada | 157,885 | 3.7% de la entrada |
| Salida | 63,381 | |
| Razonamiento | 32,969 | 52.0% de la salida |
| Visible | 30,412 | |
| Escrituras de caché | 0 | |

- **Cache hit rate:** 96.3%, muy alta.
- El prompt creció de 28K a 130K; de los 157,885 tokens sin caché, 2,653 correspondieron a la última solicitud. El costo real de contexto vino de hits de caché.

## Tiempo

- **Duración:** 24.6 min (`09:56:38`–`10:21:12`, `America/Santiago`)
- **Promedio por solicitud:** 32.1 s; **máximo:** 378.4 s; **mínimo:** 2.5 s
- **Ritmo:** 1.9 solicitudes/min; aproximadamente 2,580 tokens de salida/min
- **Contexto final:** 130,525 tokens (13% de una ventana de 1M)
