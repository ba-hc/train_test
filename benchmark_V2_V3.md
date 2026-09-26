# OMP Custom: V2 frente a V3

26 de septiembre de 2026. Se generó V3 desde un directorio vacío con el mismo prompt original del benchmark, GPT-6 Luna `max` y servicio `priority`. La medición termina cuando OMP entrega su HTML. El refinamiento manual posterior se entrega por separado y **no** forma parte de las cifras de V3.

| Métrica | V2 | V3 bruto | Cambio |
|---|---:|---:|---:|
| Duración de sesión | 49,5 min | 27,0 min | −45,4 % |
| Solicitudes al modelo | 113 | 83 | −26,5 % |
| Llamadas a herramientas | 139 | 88 | −36,7 % |
| Input fresco | 532.730 | 507.238 | −4,8 % |
| Cache read | 13.663.616 | 7.437.568 | −45,6 % |
| Output | 133.455 | 86.158 | −35,4 % |
| Tokens totales | 14.329.801 | 8.030.964 | −44,0 % |
| Coste registrado | $0,5133 | $0,3364 | −34,5 % |
| Compactaciones | 1 remota | 0 | — |

La duración de V2 aquí es la diferencia entre el primer mensaje y la última respuesta de su sesión (2968,1 s), la misma definición aplicada a V3 (1619,7 s). El README del repositorio también publica 47,1 min de «tiempo útil» para V2; es otra métrica. El cache hit de V3 fue 93,62 %, inferior al 96,25 % de V2, pero el tráfico total de caché bajó con menos llamadas.

## Calidad del artefacto

V3 bruto recuperó parte de la identidad que V2 perdió: tranvía verde oscuro más reconocible, vía prominente, faro y pueblo mejor situados, nubes y una escena más cálida. La comparación visual favorece a V3 frente a V2, aunque V1 sigue siendo una referencia fuerte de profundidad y atmósfera. V3 bruto tenía defectos funcionales: Cloudworks se ofrecía en Mango Tide y la mejora se completaba con un solo clic. En móvil, Power y Brake medían 37×38 px.

El HTML final refinado corrige esos puntos. Cloudworks solo abre en Saltlight con las puertas abiertas y el intercambio concluido; Mango Tide permite salir directamente hacia Saltlight. Hearth leaves y Little Companion son dos acciones separadas, con cambios visibles a 50 % y 100 %. El cielo usa un gradiente más azul y púrpura, hay estrellas más visibles y las ventanas iluminadas se ven en varios lados de las casas. Los botones de Power, Brake y apertura de puertas alcanzan al menos 44 px de alto en la vista móvil probada.

La prueba externa en Chromium confirmó aceleración, frenado, cámara, ambas direcciones, pasajeros, bloqueo del taller en tránsito y en Mango Tide, acceso en Saltlight, las dos mejoras, salida del taller, vista móvil de 390×844 sin desbordamiento horizontal y cero errores de consola. Para acortar la prueba de ruta, el arnés adelantó el estado a cada llegada mediante una copia instrumentada; las transiciones de puertas, taller, upgrades y salida se hicieron con los controles de la interfaz. Three.js se carga desde jsDelivr, de modo que el HTML necesita conexión al abrirse.

## Qué enseñó el cambio de setup

Las reglas de trabajo en bloques y aceptación redujeron coste y duración en esta ejecución, pero Luna aún hizo 38 evaluaciones de navegador y 14 ediciones: no desaparecieron los microciclos. La salida bruta tampoco invocó al `designer` a pesar de la regla en `AGENTS.md`. Al pedir esa revisión explícitamente en una segunda sesión, OMP sí lanzó `task` con `agent: designer` y una captura, pero quedó esperando más de 14 minutos sin respuesta. Se interrumpió esa sesión sin alterar la medición bruta y se añadió a `AGENTS.md` un límite de espera de 90 segundos para no bloquear futuros trabajos.

La prueba respalda mantener las reglas de aceptación, la verificación funcional/visual y el navegador aislado (`browser.relay: false`). No demuestra que cada cambio por separado causara la mejora: solo hay una ejecución V2 y una V3, con variación propia de generación y varios cambios de configuración simultáneos. No se cambió el orden de compactación ni el umbral de artifact spill: la pausa de 430,5 s de V2 ocurrió antes de la compactación, y V3 ni siquiera compactó. `computer.enabled: false` reduce la superficie de herramientas para trabajo web normal; no hay evidencia de que haya ahorrado tokens en esta prueba.

**Veredicto:** vale la pena conservar el setup actualizado y el HTML final refinado. Repetir al menos una vez el mismo benchmark permitiría estimar si el ahorro de V3 se reproduce; el comportamiento del `designer` requiere atención antes de confiar en él como paso obligatorio.
