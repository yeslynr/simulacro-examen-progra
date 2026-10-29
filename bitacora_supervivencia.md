# Bitácora de supervivencia — CitasSalud+

**Estudiante:** Yeslyn Rodriguuez Valverde 
**Sección:** 11-6
**Fecha:** 28 / 08 / 2026

## Escenario

Durante la ejecución de la prueba de performance (JMeter, listado de citas con
500 registros simulados — ver Anexo 1), el servidor principal de CitasSalud+
se satura y queda fuera de línea.

## 1. Identificación

La caída se detectó por tres señales en JMeter (Summary/Aggregate Report) y en
la propia aplicación:

- El tiempo de respuesta del listado de citas superó el criterio de 3 segundos
  del Anexo 1 y siguió creciendo a medida que avanzaba la prueba.
- El porcentaje de errores subió de forma sostenida: aparecieron *timeouts* y
  respuestas HTTP 500 en las solicitudes al listado.
- Al abrir la aplicación en el navegador, ya no cargaba ni siquiera el
  formulario de agendar cita, lo que confirmó que el servidor había dejado de
  responder y no solo el listado.

## 2. Contención

Se detuvo de inmediato la prueba de carga en JMeter para que el servidor dejara
de recibir solicitudes y no se agravara la saturación. Después se restringió el
acceso al módulo afectado (listado de citas y reportes), que es el que consume
más recursos con 500 registros, y se mantuvieron habilitadas únicamente las
funciones críticas. Así se limitó el impacto al listado general y se protegió la
atención de los pacientes con citas urgentes.

## 3. Recuperación

Se activó el modo **"solo emergencias"** mediante una variable de configuración
(`SOLO_EMERGENCIAS = true` en `config.json`). En este modo la aplicación solo
permite consultar y agendar citas de emergencia y deshabilita el listado
completo, de modo que el servicio esencial sigue operando mientras se corrige la
causa de fondo (listado sin paginación ni índices). Esta acción se registró en
el repositorio con el siguiente commit:

```bash
git add config.json bitacora_supervivencia.md
git commit -m "Activa modo solo-emergencias mientras se optimiza el listado de citas"
git push origin main
```

Una vez optimizado el listado (paginación, índices en la consulta de citas) y
verificado con una nueva prueba de JMeter, se puede desactivar el modo y
restablecer el servicio completo.

**Commit de recuperación:** `Activa modo solo-emergencias mientras se optimiza el listado de citas`
(hash: ________ — péguelo aquí después de ejecutar `git log --oneline -1`)

## 4. Aprendizaje / mejora

El criterio del Anexo 1 exige que el listado de citas cargue en menos de 3
segundos con 500 registros. La falla pudo anticiparse con una estrategia
complementaria combinada de **monitoreo y redundancia**:

- **Monitoreo con alertas:** medir continuamente el tiempo de respuesta del
  listado y configurar una alerta cuando se acerque a los 3 segundos (por
  ejemplo, al llegar a 2 s) o cuando suba el porcentaje de errores. Habría
  avisado del deterioro antes de la caída total.
- **Redundancia con balanceo de carga:** tener al menos un segundo servidor que
  atienda solicitudes si el principal se satura, para no depender de una única
  instancia.
- **Prevención:** paginación e índices en la base de datos, pruebas de carga
  periódicas con JMeter antes de cada entrega y respaldos verificados de las
  citas.

Estas medidas reducen el riesgo de indisponibilidad en un sistema de salud,
donde una cita de emergencia no puede quedar sin atención por una saturación
técnica.
