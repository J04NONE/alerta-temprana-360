# Alerta Temprana 360 — Propuesta Técnica Inicial
### Primera Sesión de Elicitación de Requisitos

> **Equipo:** Joan (Líder técnico), Andrés (Diseño), Iván (Análisis), Michael (SM/Documentación)  
> **Fecha:** [Fecha de la reunión]  
> **Metodología de documentación:** ISO/IEC/IEEE 29148:2018

---

## Apertura — Qué somos y qué traemos

*(Michael abre la sesión)*

Profesor, el equipo ha analizado la problemática de deserción estudiantil y llegamos con una propuesta concreta, no con preguntas abiertas. Le vamos a presentar nuestra visión del sistema, los requisitos que ya tenemos definidos, y le pediremos su validación y los criterios que aún necesitamos de su parte para completar el documento.

El sistema se llama **Alerta Temprana 360**. Es un motor de reglas para detección proactiva de estudiantes en riesgo académico y financiero, con un panel de seguimiento para facultades y docentes.

---

## 1. Descripción del Problema

Las universidades pierden estudiantes que nadie vio venir. Un estudiante cursa su tercera matrícula en Cálculo, tiene dos notas en cero en el semestre activo, y su beca está a punto de agotarse. Esa información existe en el sistema de notas, en tesorería y en bienestar — pero está dispersa. Nadie la cruza. Nadie actúa.

El resultado: el estudiante abandona. El sistema registra la baja. Nadie supo antes.

**Alerta Temprana 360** cruza esa información, activa una alerta cuando los indicadores lo justifican, y le da al docente o directivo el contexto necesario para intervenir a tiempo. No reemplaza al SIAU ni a ningún sistema existente. Trabaja con lo que ya hay.

---

## 2. Identificación de Actores

| Actor | Rol en el sistema | Nivel de acceso |
|---|---|---|
| **Estudiante** | Sujeto de seguimiento. Puede consultar su propio estado de riesgo | Solo lectura de su perfil |
| **Docente** | Registra seguimientos y recibe alertas de sus estudiantes | Lectura de casos asignados + escritura de seguimientos |
| **Director de Programa / Directivo** | Supervisión general. Ve el estado de todos los casos activos | Lectura amplia + escalamiento |
| **Administrador de Facultad** | Carga datos académicos al sistema (CSV/Excel desde el SIAU) | Carga de datos + gestión de usuarios |
| **Padre de Familia** *(opcional, sujeto a autorización)* | Recibe notificaciones si el estudiante autoriza el acceso | Solo notificaciones — no historial completo |

> **Nota abierta para el profesor:** necesitamos confirmar si el padre de familia es un actor dentro del alcance de este proyecto o si lo dejamos fuera de la versión inicial.

---

## 3. Requisitos Funcionales (RF)

*Cinco requisitos para la versión inicial. Sujetos a ajuste según retroalimentación.*

---

**RF-01 — Detección automática de riesgo por matrículas**  
El sistema debe identificar automáticamente a los estudiantes que registren tres o más matrículas en una misma materia, y generar una alerta de riesgo académico en el panel del docente y del Director de Programa.

**Criterio de aceptación:** Dado un estudiante con 3 registros de matrícula para la misma materia, el sistema genera la alerta en menos de 1 minuto desde que se cargan los datos.

---

**RF-02 — Detección de nota en cero en período activo**  
El sistema debe identificar a los estudiantes que tengan al menos una nota de 0.0 registrada en el semestre activo, diferenciando entre nota no subida por el docente y nota perdida confirmada.

**Criterio de aceptación:** El sistema distingue y etiqueta los dos estados. No agrupa ambos casos bajo el mismo indicador.

> **Pregunta abierta para el profesor (Iván):** ¿"Nota en cero" significa una nota no subida por el docente o una calificación de 0.0 registrada formalmente? La definición cambia el motor de reglas.

---

**RF-03 — Notificación automática al activarse un indicador**  
El sistema debe enviar una notificación (correo electrónico y/o notificación en la app) al docente responsable y al Director de Programa cuando se active cualquier indicador de riesgo para un estudiante bajo su seguimiento.

**Criterio de aceptación:** La notificación llega en menos de 5 minutos desde la activación del indicador. Incluye: nombre del estudiante, indicador activado y enlace directo al perfil.

---

**RF-04 — Carga masiva de datos académicos**  
El sistema debe permitir la carga de datos académicos mediante archivos CSV o Excel. Antes de importar, debe validar la integridad del archivo y reportar los errores encontrados sin cargar datos inconsistentes.

**Criterio de aceptación:** Si el archivo tiene filas con datos faltantes o formatos incorrectos, el sistema rechaza la carga y muestra un reporte de errores por fila. No carga parcialmente.

---

**RF-05 — Registro de seguimiento por el docente**  
El sistema debe permitir que el docente registre un seguimiento para cada caso de riesgo activo: fecha, tipo de intervención realizada y observaciones. Este registro debe quedar vinculado al historial del estudiante.

**Criterio de aceptación:** El historial de seguimientos es visible para el Director de Programa y no puede ser eliminado por el docente (solo añadir entradas nuevas).

---

## 4. Requisitos No Funcionales (RNF)

*Borrador inicial — Iván completa la redacción formal con métrica y prioridad antes de la reunión.*

| ID | Categoría | Descripción resumida | Métrica de verificación |
|----|-----------|----------------------|------------------------|
| RNF-01 | Rendimiento | Tiempo de respuesta bajo carga máxima | Máximo 500ms con 10,000 usuarios concurrentes (Locust) |
| RNF-02 | Disponibilidad | Uptime del sistema | 99.5% de disponibilidad mensual |
| RNF-03 | Seguridad | Autenticación y control de acceso | JWT + RBAC. Acceso denegado a datos fuera del rol. Audit Log activo |
| RNF-04 | Usabilidad | Tiempo para identificar alerta crítica | Un directivo identifica la alerta más grave en menos de 10 segundos sin instrucciones previas |
| RNF-05 | Escalabilidad | Crecimiento a múltiples sedes | Agregar una sede nueva no requiere cambios en el código, solo configuración |

> Iván: cada fila de esta tabla se convierte en un RNF con formato completo (ID, nombre, descripción, métrica, prioridad) usando la estructura del documento interno.

---

## 5. Restricciones Conocidas

Estas restricciones ya están definidas — son parte del contexto del sistema:

- **Sin integración directa al SIAU:** Los datos académicos entran al sistema por carga manual (CSV/Excel). El sistema no se conecta a la base de datos de notas oficial.
- **Sin reemplazo de sistemas existentes:** Alerta Temprana 360 no sustituye el registro de notas, la pasarela de pagos ni ningún módulo del LMS actual.
- **Habeas Data (Ley 1581 de 2012):** El sistema registra un log de auditoría por cada consulta a datos de un estudiante (quién, cuándo, qué). El acceso de terceros (padres) requiere autorización explícita del estudiante.
- **Stack tecnológico fijo:** Django/Python + PostgreSQL (Supabase). 
- **Metodología de documentación:** ISO/IEC/IEEE 29148:2018, actualización incremental por Sprint.

---

## 6. Preguntas para el Profesor

*Necesitamos estas respuestas para completar el documento. No son opcionales.*

**Sobre actores y alcance:**
1. ¿El padre de familia entra al alcance del primer entregable o lo aplazamos?
2. ¿Quién carga los datos al sistema: cada docente por separado o un administrador de facultad?

**Sobre reglas de negocio:**
3. ¿Cuál es la definición exacta de "Riesgo Crítico"? ¿Solo número de matrículas o también promedio ponderado?
4. ¿Existen indicadores que el sistema debería detectar además de los tres que tenemos (matrículas, notas, beca)?

**Sobre privacidad:**
5. ¿Es suficiente el Audit Log para cumplir Habeas Data o el estudiante debe autorizar explícitamente a cada docente?
6. ¿Cómo manejamos la transición cuando el estudiante cumple 18 años y los padres ya tenían acceso?

**Sobre calidad y documentación:**
7. ¿Acepta validar el requisito de 10,000 usuarios con una prueba de carga en Locust?
8. ¿Podemos actualizar el documento de requisitos de forma incremental por Sprint?

---

## 7. Escenario de Excepción — Para discutir con el profesor

*(Iván plantea esto en la reunión)*

> "Profesor, supongamos que el sistema detecta a un estudiante con 3 materias perdidas y envía la alerta. El docente la recibe pero no registra ningún seguimiento en 48 horas. ¿Debe el sistema escalar esa omisión automáticamente al Director de Programa?"

Esta pregunta tiene impacto directo en el motor de reglas. Si la respuesta es sí, necesitamos un RF-06 de escalamiento automático.

---

## 8. Propuesta Metodológica

Si el profesor pregunta por RUP o IEEE 830:

> "Profesor, vamos a usar ISO/IEC/IEEE 29148:2018, que es el sucesor oficial del 830. La diferencia es que el 29148 permite que la especificación de requisitos sea iterativa — el documento se actualiza en cada Sprint y siempre refleja el estado real del software. No es un documento congelado que queda obsoleto a la segunda semana."

Esto no es una trampa. Es el estándar correcto para proyectos que no trabajan en cascada.

---

*Documento preparado por el equipo previo a la primera sesión de elicitación.*  
*Versión: 1.0 — se actualiza con feedback del profesor.*
