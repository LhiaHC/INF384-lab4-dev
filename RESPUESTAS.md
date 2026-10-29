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
- Job y etapa:
- Problema que causa en el pipeline:
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):
- Evidencia (dato observado, con link al run o fragmento de la salida local):

## Parte 2 · Implementación
- Link al run en verde en main:
- Link al pull request:
- Defecto de la aplicación introducido (archivo y descripción):
- Job que debería detenerlo:
- Link al run fallido:

## Parte 3 · Decisión bajo restricción
Link al run medido:

| Job | Duración (min:seg) |
|---|---|
| unit | |
| integration | |
| sast | |
| sca | |
| build-push | |
| deploy | |
| smoke | |
| e2e | |
| load | |
| **Total** | |

Resultado de load en ese run:
| Métrica | Valor |
|---|---|
| p90 | |
| p95 | |
| Máximo | |
| Tasa de error | |
| Requests por segundo | |

Conclusión sobre el comportamiento bajo carga:

SLO propuesto (percentil, valor, ventana) y justificación:

SLA propuesto y diferencia con el SLO:

Cambios para cumplir el presupuesto:
| Cambio | Job | Minutos recuperados |
|---|---|---|

Total resultante: [ ] min
Link al run en main con los cambios:

Riesgo aceptado (defecto de la aplicación concreto y efecto en la métrica DORA asignada):

Decisión sobre sast (se queda / se mueve a otra etapa o evento / informativo) y razón:

## Declaración de uso de IA
Herramientas usadas y para qué:
