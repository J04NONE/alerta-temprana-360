# 01 — Descripción del Problema: Alerta Temprana 360

## 1. El Escenario Actual

En el entorno educativo de Bogotá, tanto en instituciones oficiales como privadas, la detección de situaciones que vulneran los derechos de los niños, niñas y adolescentes (NNA) enfrenta un desafío crítico de fragmentación. Actualmente, los reportes de convivencia, accidentalidad, y riesgos socioemocionales se manejan a menudo de forma manual o en sistemas aislados que no permiten una trazabilidad efectiva.

### Puntos de Dolor Identificados

* **Información Fragmentada:** Los datos sobre presuntas vulneraciones (abuso, violencia, conducta suicida) residen en diferentes dependencias o formatos físicos, impidiendo una visión de 360 grados del estudiante.
* **Intervención Reactiva:** Sin un sistema de alertas proactivas, las instituciones actúan cuando el evento ya ha escalado o causado daños irreparables, en lugar de detectar factores de riesgo tempranos.
* **Falta de Trazabilidad:** No existe un historial unificado y seguro de los seguimientos realizados por docentes y coordinadores, lo que dificulta la continuidad del acompañamiento cuando el estudiante cambia de grado o institución.
* **Riesgos de Privacidad:** El manejo de información sensible (según Ley 1581 de 2012) sin protocolos técnicos de aislamiento (como RLS) pone en riesgo la intimidad de los menores.

## 2. La Evolución del Problema

Originalmente concebido para el ámbito universitario enfocado en el riesgo académico (matrículas y notas), el proyecto ha pivotado para abordar la necesidad urgente de un sistema que integre el modelo de alertas de la **Secretaría de Educación del Distrito (SED)**. Esta necesidad se ampara en la **Ley 1620 de 2013**, que exige la creación de sistemas de información unificados para la convivencia escolar.

### El Dato Académico como "Indicador Silencioso"

El componente de datos académicos (RF-03) se mantiene en el sistema, pero en un rol subordinado: actúa como **capa de inteligencia automática** que detecta señales de alerta que el ojo humano puede no percibir antes de que un incidente sea visible. Un estudiante que acumula tres matrículas en la misma asignatura o que registra una nota 0.0 confirmada no necesariamente tiene un problema académico — con frecuencia está atravesando una situación de convivencia no reportada (ausentismo por acoso, bloqueo emocional, situación familiar crítica).

Por eso, el sistema no genera "alertas académicas" independientes: convierte esos indicadores en alertas del módulo **MOD-06 — Trastornos de Aprendizaje y Comportamiento**, que sigue exactamente el mismo flujo de atención, escalamiento y bitácora que cualquier alerta registrada manualmente por un Docente. Esto demuestra que AT360 no gestiona formularios — **procesa información para prevenir riesgos antes de que sean evidentes al ojo humano**.

## 3. Definición de la Solución: Alerta Temprana 360

El sistema se propone como una herramienta de observación y análisis que consolida seis módulos de alerta específicos:

1. **Abuso y violencia:** Detección y reporte de presuntas vulneraciones.
2. **Accidentalidad escolar:** Gestión y seguimiento de incidentes físicos.
3. **Conducta suicida:** Monitoreo de factores de riesgo vital.
4. **Consumo de SPA:** Alertas sobre uso de sustancias psicoactivas.
5. **Maternidad y paternidad tempranas:** Acompañamiento a estudiantes en gestación.
6. **Trastornos de aprendizaje:** Apoyo especializado para el comportamiento y aprendizaje — incluyendo alertas generadas automáticamente por el Motor de Indicadores Indirectos (RF-03).

## 4. Impacto Esperado

Al implementar **Alerta Temprana 360**, las instituciones lograrán:

* Reducir el tiempo de respuesta ante alertas críticas.
* Garantizar el cumplimiento legal de la Ley 1620 y Ley 1581.
* Proveer a los directivos y coordinadores una visibilidad inmediata (dashboard) sobre el estado real del acompañamiento estudiantil.
* Escalar de forma automática casos no atendidos, asegurando que ningún riesgo sea ignorado por el sistema.
* Detectar indicadores indirectos de riesgo de convivencia a partir de datos académicos, sin esperar a que el docente observe el incidente manualmente.
