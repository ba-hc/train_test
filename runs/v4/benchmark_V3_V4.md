# Benchmark OMP Custom V3 → V4

26 de septiembre de 2026. V4 se ejecutó desde cero con el prompt original, GPT-6 Luna `max` y `priority`. Se instalaron tres skills enfocadas, se añadió aceptación previa, un presupuesto de QA, puerta de finalización y criterios Three.js, se ocultó Impeccable del listado automático y se bajó el spill de artefactos a 30 KiB. La configuración versionada está en el repositorio privado `omp-stack-private`, commit `0fcce20`.

## Eficiencia medida

| Métrica | V3 bruto | V4 bruto | Cambio V4 |
| --- | ---: | ---: | ---: |
| Duración de sesión | 27,0 min | 40,4 min | **+49,6 %** |
| Solicitudes | 83 | 104 | **+25,3 %** |
| Herramientas | 88 | 103 | **+17,0 %** |
| `eval`/browser | 38 | 47 | **+23,7 %** |
| `edit` | 14 | 16 | **+14,3 %** |
| Entrada fresca | 507.238 | 335.720 | −33,8 % |
| Lectura de caché | 7.437.568 | 11.100.544 | +49,2 % |
| Salida | 86.158 | 105.097 | +22,0 % |
| Tokens totales | 8.030.964 | 11.541.361 | **+43,7 %** |
| Coste registrado | USD 0,3364 | USD 0,3943 | **+17,2 %** |
| Compactaciones | 0 | 0 | — |

La tasa de caché de V4 subió a 97,06 % (V3: 93,62 %), pero el contexto cacheado se leyó muchas más veces. El menor input fresco no compensó el incremento de solicitudes. Los objetivos de 22–27 min, 60–75 solicitudes, 10–16 `eval` y 5,5–7 M tokens **no se cumplieron**. No hubo delegación al `designer`; sí se leyeron `frontend-design` y `threejs`, pero no `web-design-guidelines`.

## Calidad del HTML bruto

La [verificación externa](./OMP_CUSTOM_V4_verification.json) abrió el HTML medido en Chromium. Escritorio y móvil cargaron el canvas sin errores de consola ni desbordamiento horizontal a 390×844. La escena usa instancing para nubes, traviesas, pilares y suelo del taller. El HUD muestra velocidad, pasajeros, condición, comodidad y estaciones. La vista inicial, sin embargo, deja la parte trasera del tranvía y la vía en primer plano, tapando gran parte del pueblo; las [capturas de escritorio](./OMP_CUSTOM_V4_desktop.png) y [móvil](./OMP_CUSTOM_V4_mobile.png) permiten juzgarlo directamente.

Tres criterios importantes quedaron sin cumplir:

1. **Ubicación del taller:** Cloudworks no se ofrece en Saltlight tras abrir puertas y completar el embarque; aparece en Mango Tide. La prueba de la interfaz confirmó ambos estados. La ubicación Saltlight venía de la rúbrica V1–V3; el prompt original decía «Oliver's home island» sin asignarle inequívocamente una estación. Por eso es una regresión frente a esa rúbrica, no una contradicción textual inequívoca del prompt.
2. **Dos mejoras interactivas:** un clic en «Begin the tune-up» completa automáticamente Hearth leaves y Little Companion en unos 4,8 s. La interfaz termina con 100 % y habilita continuar sin una segunda acción. Esto no alcanza el flujo de dos acciones separadas del V3 refinado.
3. **Controles móviles:** Power y Brake miden 64×36 px; «Open doors» mide 183×37 px. La meta práctica de al menos 44 px no se alcanzó.

Además, el código limita DPR a 1,7, aunque el nuevo finish gate sugería empezar cerca de 1,25–1,5 para escenas densas. `prefers-reduced-motion` reduce parte del reloj ambiental, pero sigue habiendo movimiento de cámara y taller. Una instantánea externa de `renderer.info` durante el taller registró 199 draw calls, 6.814 triángulos, 521 geometrías y 4 texturas; no se fijó un umbral de rendimiento ni se hizo un perfil GPU.

Para acortar la prueba externa del recorrido, se usó una copia instrumentada que forzó la llegada a Mango Tide. Puertas, embarque, entrada al taller y el único clic de mejora se ejercieron con los controles de la interfaz. **El HTML publicado no se editó**. Three.js se carga desde jsDelivr, así que abrir el archivo requiere conexión.

## Conclusión

En esta ejecución, V4 **no mejoró el benchmark**: costó más tiempo, solicitudes, herramientas, tokens y dinero registrados. Tampoco alcanzó la calidad funcional del V3 refinado. La regla de dos rondas de QA no limitó el número de llamadas `eval`, y la skill de auditoría final no se activó. Estos son resultados de una sola ejecución por versión y de varios cambios simultáneos; no permiten atribuir la regresión a una skill o parámetro individual. Para uso inmediato, V3 refinado sigue siendo la mejor entrega de este conjunto.
