# Respuestas · Laboratorio 4
Alumno: [20211729]
Presupuesto: [11] min  Métrica DORA: [change failure rate]

## Parte 1 · Diagnóstico
### Defecto 1
- Job y etapa: job (smoke), etapa (deploy)
- Problema que causa en el pipeline: El job de smoke no espera a que termine el de deploy, por lo que puede probar una versión anterior de la aplicación y no detectar errores en la nueva versión.
- Link al commit: https://github.com/LhiaHC/INF384-lab4-dev/commit/380be7835d6cbe0d89b8609c3562e7d835c6186c 
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): mover etapa (La clasificación es mover de etapa, porque smoke ahora depende de deploy y se ejecuta después de que termina el despliegue.)
- Evidencia (dato observado, con link al run o fragmento de la salida local): https://github.com/LhiaHC/INF384-lab4-dev/actions/runs/40957343514 
### Defecto 2
- Job y etapa: load, etapa post-deploy.
- Problema que causa en el pipeline: El job load también se ejecuta en los pull requests, haciendo que el pipeline demore más de lo necesario.
- Link al commit: https://github.com/LhiaHC/INF384-lab4-dev/commit/0a58f7d2304d311ca9be3675885c53532fabf10d
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Ajustar configuración. Se agrega una condición para que load se ejecute solo en main.
- Evidencia (dato observado, con link al run o fragmento de la salida local): https://github.com/LhiaHC/INF384-lab4-dev/actions/runs/40959588072
### Defecto 3
- Job y etapa:  sca, etapa pre-build.
- Problema que causa en el pipeline:  El job sca revisa también las dependencias de desarrollo, lo que puede bloquear el despliegue por vulnerabilidades de herramientas que no se incluyen en producción.
- Link al commit: https://github.com/LhiaHC/INF384-lab4-dev/commit/07d1e91a965cbe2736cbc777e980a41d3ea7ce04
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Ajustar configuración. Se quitó --include-dev-deps para que el análisis SCA no incluya las dependencias de desarrollo.
- Evidencia (dato observado, con link al run o fragmento de la salida local): https://github.com/LhiaHC/INF384-lab4-dev/actions/runs/40961627014/job/123755587764
### Defecto 4
- Job y etapa: integration, etapa pre-build.
- Problema que causa en el pipeline: Los tests usan siempre la misma base (/tmp/wallet.db) y suponen que empieza con los saldos iniciales. La primera vez pasan, pero si se vuelven a correr sobre esa base fallan aunque el código esté bien, porque los saldos ya cambiaron. Así dan falsos errores.
- Link al commit: https://github.com/LhiaHC/INF384-lab4-dev/commit/2d3417ec2b59211f916feaae12e83fbeb329127e
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): modificar tests + ajustar configuración. Cada ejecución crea su propia base temporal (mkdtemp) y se quitó el DB_PATH fijo del job.
- Evidencia (dato observado, con link al run o fragmento de la salida local): - Evidencia: Run #8, job integration (https://github.com/LhiaHC/INF384-lab4-dev/actions/runs/40963860256/job/123763225562), con base temporal por ejecución en vez de /tmp/wallet.db fijo:tests 12, pass 12, fail 0
### Defecto 5
- Job y etapa: e2e, etapa post-deploy.
- Problema que causa en el pipeline: El job instala chromium en cada run, pero los tests solo hacen llamadas HTTP y no usan el navegador. Ese paso desperdicia tiempo. Además, había tests (monto negativo, ruta 404) que ya se prueban en unit/integration.
- Link al commit: [link commit ci.yml](https://github.com/LhiaHC/INF384-lab4-dev/commit/26250eba2638ae61196b6ef08ac1f8bdaca1a3b2)] (cambio de tests: [https://github.com/LhiaHC/INF384-lab4-dev/commit/7b62478a7945644f511678a2d3f5729d2fe49b94])
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): modificar tests + eliminar. Los tests usan request (sin navegador), se eliminó el paso "Navegador" y se quitaron los tests duplicados.
- Evidencia (dato observado, con link al run o fragmento de la salida local): e2e bajó de 46 s (run #8: [https://github.com/LhiaHC/INF384-lab4-dev/actions/runs/40963860256], "Navegador" 22 s) a 21 s (run #10: [https://github.com/LhiaHC/INF384-lab4-dev/actions/runs/40964845678], 8 passed).

## Parte 2 · Implementación
- Link al run en verde en main: https://github.com/LhiaHC/INF384-lab4-dev/actions/runs/40964845678
- Link al pull request: https://github.com/LhiaHC/INF384-lab4-dev/pull/1
- Defecto de la aplicación introducido (archivo y descripción): src/util.js. validarMonto cambia "monto <= 0" por "monto < 0", así que acepta monto 0. POST /transferir con monto 0 devuelve 200 y registra un movimiento, cuando debería devolver 400
- Job que debería detenerlo: unit
- Link al run fallido: https://github.com/LhiaHC/INF384-lab4-dev/actions/runs/40966035799/job/123770665447

## Parte 3 · Decisión bajo restricción
Link al run medido: https://github.com/LhiaHC/INF384-lab4-dev/actions/runs/40964845678

| Job | Duración (min:seg) |
|---|---|
| unit | 0:10 |
| integration | 0:15 |
| sast | 0:20 |
| sca | 0:12 |
| build-push | 0:33 |
| deploy | 0:15 |
| smoke | 0:10 |
| e2e | 0:21 |
| load | 12:13 |
| **Total** | **14:29** |

Resultado de load en ese run:
| Métrica | Valor |
|---|---|
| p90 | 72,83 ms |
| p95 | 75,04 ms |
| Máximo | 3,72 s |
| Tasa de error | 0,00 % (0 de 34 910) |
| Requests por segundo | 48,45 req/s |

Conclusión sobre el comportamiento bajo carga:
La aplicación aguantó bien la carga. Con 30 usuarios durante 12 minutos, el 95 % de los requests respondió en menos de 75 ms y no hubo ningún error. Solo hubo un request lento de 3,72 s, que seguramente fue el arranque en frío de una instancia nueva de Lambda.

SLO propuesto (percentil, valor, ventana) y justificación:
El 95 % de los requests debe responder en menos de 150 ms, medido en los últimos 28 días. En el pipeline lo puse como SLO_P95 = 150.
Elegí 150 ms porque es el doble de lo que medí (75 ms). Así load no falla por variaciones normales entre runs, pero sí falla si un cambio hace que la app se vuelva mucho más lenta. El valor anterior de 500 ms era demasiado alto y casi nunca fallaría.

SLA propuesto y diferencia con el SLO:
El 99 % de los requests de cada mes responderá en menos de 1 segundo. Si no se cumple, se compensa al cliente.
El SLO es una meta interna del equipo, y el SLA es una promesa al cliente con consecuencias si no se cumple. Por eso el SLA es más flexible: si la app se pone lenta, el equipo lo nota primero con el SLO y tiene tiempo de corregirlo antes de fallarle al cliente.

Cambios para cumplir el presupuesto:
| Cambio | Job | Minutos recuperados |
|---|---|---|
| Reducir la prueba de carga: subida 15 s, carga constante 4 min, bajada 15 s (antes 1 + 10 + 1 min) | load | 7,5 (12:13 → 4:43) |

Total resultante: 7:10 min (11 + 13 + 16 + 20 + 47 + 16 + 8 + 16 + 283 s = 430 s)
Link al run en main con los cambios: https://github.com/LhiaHC/INF384-lab4-dev/actions/runs/40968793126


Riesgo aceptado (defecto de la aplicación concreto y efecto en la métrica DORA asignada):
Al reducir la prueba de carga de 10 a 4 minutos, el pipeline puede dejar pasar problemas que solo aparecen después de varios minutos. Por ejemplo, si alguien cambia db.js para calcular el saldo sumando todos los movimientos, /saldo se pone más lento a medida que se hacen transferencias. En 4 minutos todavía no se nota, pero en 10 sí. Ese cambio llegaría a producción y fallaría con los clientes.
Esto hace que el Change Failure Rate suba, porque habría más despliegues que terminan causando fallas.
Decisión sobre sast (se queda / se mueve a otra etapa o evento / informativo) y razón:
Se queda donde está. Solo demora 20 segundos y el pipeline ya cumple el presupuesto de 11 minutos sin tocarlo. Si lo quitara o lo volviera informativo, podría llegar a producción código inseguro, y eso haría subir el Change Failure Rate. Dejándolo, el problema se detiene antes de desplegarse.
## Declaración de uso de IA
Herramientas usadas y para qué:
- GPT: revisión del código yml, sugerencia de correcciones y testeos para evidencia y apoyo en la redacción.
- Yo hice los commits, ejecuté los runs, revisé los logs, obtuve las evidencias y verifiqué cada corrección.
