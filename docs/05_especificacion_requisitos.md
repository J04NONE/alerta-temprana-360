# 05 — Especificación de Requisitos: Alerta Temprana 360

**Estándar:** ISO/IEC/IEEE 29148:2018  
**Versión:** 1.0  
**Fecha:** 2026-04-30  
**Estado:** Borrador — pendiente validación del profesor (stakeholder)  
**Trazabilidad:** Cada requisito aquí especificado se deriva de la Matriz de Requisitos (`03_matriz_requisitos.md`).

---

## Convenciones de redacción

| Término | Significado |
| :--- | :--- |
| **debe** | Requisito obligatorio. Su incumplimiento constituye un fallo del sistema. |
| **no debe** | Prohibición absoluta. El sistema nunca puede realizar esta acción. |
| **debería** | Recomendado. Incumplible solo con justificación documentada. |
| **Actor** | Rol que inicia o recibe el resultado de la función. Definidos en `02_matriz_actores.md`. |
| **RLS** | Row-Level Security — aislamiento de datos a nivel de base de datos (Supabase/PostgreSQL). |
| **RBAC** | Control de acceso basado en roles (Role-Based Access Control). |
| **SED** | Secretaría de Educación del Distrito de Bogotá — modelo de alertas de referencia. |
| **SPA** | Sustancias Psicoactivas. |

---

## RF-01 — Gestión de Usuarios y Roles

| Campo | Detalle |
| :--- | :--- |
| **Código** | RF-01 |
| **Nombre** | Gestión de Usuarios y Roles (RBAC) |
| **Tipo** | Funcional |
| **Prioridad** | Alta |
| **Actor(es)** | Administrador de Sistema (TI), Coordinador de Convivencia, todos los roles |

**Descripción:**  
El sistema debe permitir el registro, autenticación y administración de usuarios bajo un modelo de control de acceso basado en roles (RBAC) con cinco niveles de privilegio, garantizando que cada usuario acceda exclusivamente a los datos y funciones correspondientes a su jurisdicción institucional.

**Precondición:**  
- El Administrador de Sistema está autenticado con Nivel 5 (acceso técnico completo).
- La institución educativa ya existe en el sistema con su `sede_id` configurado.

**Postcondición:**  
- El nuevo usuario existe en la base de datos con rol asignado, sede vinculada y contraseña cifrada (bcrypt).
- Se genera un registro de auditoría con: actor que creó la cuenta, timestamp, rol asignado y sede.
- El usuario puede autenticarse mediante JWT con expiración configurable.

**Criterios de aceptación:**

```gherkin
Escenario 1: Registro exitoso de un Docente
  Dado que el Administrador de Sistema está autenticado
  Cuando registra un nuevo usuario con rol "Docente" y asigna la sede "IED Norte"
  Entonces el sistema crea la cuenta con Nivel 2 de acceso
  Y el Docente solo puede ver alertas de estudiantes vinculados a su curso en "IED Norte"
  Y se genera un log de auditoría con el evento de creación

Escenario 2: Intento de acceso fuera de jurisdicción
  Dado que un Docente de "IED Norte" está autenticado
  Cuando intenta consultar el perfil de un estudiante de "IED Sur"
  Entonces el sistema rechaza la solicitud con código HTTP 403
  Y no expone ningún dato del estudiante de "IED Sur"
  Y registra el intento fallido en el log de auditoría

Escenario 3: Vinculación de Padre de Familia mediante OTP
  Dado que la institución ha generado un Token OTP no reutilizable para el acudiente verificado
  Cuando el Padre ingresa el token en el proceso de registro
  Entonces el sistema valida el token, crea la relación ParentStudent
  Y el token queda marcado como utilizado y no puede reutilizarse
  Y el Padre accede únicamente al historial del estudiante vinculado (Nivel 1 Restringido)
```

**Reglas de negocio:**
- RB-01: Un token OTP para vinculación de padres tiene una vigencia máxima de 72 horas desde su generación.
- RB-02: El sistema no almacena documentos de identidad del acudiente; solo registra el hash de verificación del OTP.
- RB-03: El aislamiento de datos (RLS) se implementa a nivel de base de datos, no solo a nivel de aplicación. Un error en la capa de negocio no debe exponer datos de otra sede.
- RB-04: Las contraseñas deben cumplir una política mínima: 8 caracteres, al menos una mayúscula, un número y un carácter especial.
- RB-05: Los tokens JWT tienen una expiración de 8 horas para sesiones de usuario y 24 horas para aplicaciones móviles.

---

## RF-02 — Registro de Alertas SED

| Campo | Detalle |
| :--- | :--- |
| **Código** | RF-02 |
| **Nombre** | Registro de Alertas — Módulos SED |
| **Tipo** | Funcional |
| **Prioridad** | Alta |
| **Actor(es)** | Docente (registro), Coordinador de Convivencia (gestión), Estudiante (autorreporte) |

**Descripción:**  
El sistema debe permitir reportar y gestionar incidentes categorizados en seis módulos de alerta definidos por la Secretaría de Educación del Distrito (SED), registrando para cada alerta: el módulo, el estudiante afectado, la descripción del incidente, la fecha/hora, el actor reportante y el estado de atención.

**Módulos de alerta:**

| Código Módulo | Nombre | Nivel de Riesgo Base |
| :--- | :--- | :--- |
| MOD-01 | Abuso y Violencia | Crítico |
| MOD-02 | Accidentalidad Escolar | Alto |
| MOD-03 | Conducta Suicida | Crítico |
| MOD-04 | Consumo de Sustancias Psicoactivas (SPA) | Alto |
| MOD-05 | Maternidad y Paternidad Tempranas | Medio |
| MOD-06 | Trastornos de Aprendizaje y Comportamiento | Medio |

**Precondición:**
- El Docente está autenticado con Nivel 2 de acceso.
- El estudiante reportado existe en el sistema y está vinculado al curso del Docente.

**Postcondición:**
- La alerta queda registrada en la base de datos con estado `Activa` y visible para el Docente creador y el Coordinador de Convivencia de la sede.
- Se activa automáticamente el flujo de notificación (ver RF-04).
- Se genera un log de auditoría que registra: quién creó la alerta, sobre qué estudiante, en qué módulo y cuándo.

**Criterios de aceptación:**

```gherkin
Escenario 1: Registro exitoso de alerta crítica
  Dado que un Docente está autenticado y navega a su lista de estudiantes
  Cuando selecciona un estudiante, elige el módulo "Conducta Suicida"
    y completa los campos obligatorios (descripción, fecha observación)
  Entonces el sistema guarda la alerta con estado "Activa" y nivel "Crítico"
  Y el Coordinador de Convivencia recibe una notificación inmediata (< 2 minutos)
  Y la alerta es visible en el dashboard del Coordinador con indicador visual de nivel Crítico

Escenario 2: Intento de registro sin datos obligatorios
  Dado que un Docente intenta registrar una alerta en el módulo "SPA"
  Cuando omite el campo "Descripción del incidente"
  Entonces el sistema rechaza el guardado
  Y muestra el mensaje de error específico: "La descripción del incidente es obligatoria"
  Y no crea ningún registro parcial en la base de datos

Escenario 3: Visibilidad restringida por RLS
  Dado que existen alertas registradas en "IED Norte" y en "IED Sur"
  Cuando un Coordinador de "IED Norte" consulta el panel de alertas activas
  Entonces el sistema muestra únicamente las alertas de "IED Norte"
  Y no expone en ninguna respuesta de API datos de alertas de "IED Sur"
```

**Reglas de negocio:**
- RB-06: Los módulos MOD-01 (Abuso) y MOD-03 (Conducta Suicida) tienen clasificación de riesgo `Crítico` y activan el escalamiento automático si no son atendidos en 24 horas (no en 48h como los demás módulos).
- RB-07: Una alerta no puede ser eliminada del sistema. Solo puede cambiar de estado: `Activa` → `En Seguimiento` → `Cerrada`.
- RB-08: El campo "Descripción del incidente" tiene un límite máximo de 2,000 caracteres y no admite HTML ni scripts para prevenir XSS.
- RB-09: Cada acceso de lectura a una alerta de módulo Crítico genera automáticamente un registro de auditoría (quién leyó, cuándo).

---

## RF-03 — Motor de Detección de Indicadores Indirectos

| Campo | Detalle |
| :--- | :--- |
| **Código** | RF-03 |
| **Nombre** | Motor de Detección de Indicadores Indirectos de Riesgo de Convivencia |
| **Tipo** | Funcional |
| **Prioridad** | Media |
| **Actor(es)** | Administrador de Institución (carga datos), Sistema (procesamiento automático), Docente (receptor de alerta MOD-06) |

**Descripción:**
El sistema debe procesar automáticamente los datos académicos cargados vía archivo (ver RF-06) e identificar indicadores indirectos de riesgo de convivencia bajo dos señales: (a) tres o más matrículas registradas en la misma asignatura, y (b) presencia de nota 0.0 confirmada en el semestre activo, diferenciándola de una nota aún no registrada. Al detectar cualquiera de estas señales, el sistema genera automáticamente una alerta en el módulo **MOD-06 — Trastornos de Aprendizaje y Comportamiento**, que sigue el mismo flujo de atención, notificación y escalamiento que una alerta registrada manualmente por un Docente (RF-02).

**Precondición:**

* Los datos académicos del semestre activo han sido cargados exitosamente mediante RF-06.
* El semestre activo está configurado en el sistema con fecha de inicio y fin.

**Postcondición:**

* Se genera una alerta MOD-06 para cada estudiante con indicadores detectados, visible en el panel del Docente responsable y del Coordinador de Convivencia.
* Se activa el flujo de notificación (RF-04) con la misma prioridad que una alerta MOD-06 manual.
* La detección queda registrada con: indicador activado, fecha de detección, semestre de referencia.

**Criterios de aceptación:**

```gherkin
Escenario 1: Generación de alerta MOD-06 por matrículas reiteradas
  Dado que los datos cargados incluyen a un estudiante
    con 3 registros de matrícula para "Cálculo Diferencial"
  Cuando el sistema procesa el archivo de carga
  Entonces genera una alerta MOD-06 "Trastornos de Aprendizaje y Comportamiento"
    vinculada al indicador "Matrícula reiterada — Cálculo Diferencial"
  Y la alerta aparece en el panel del Docente responsable en menos de 1 minuto
  Y el sistema notifica al Docente con la misma prioridad que una alerta MOD-06 manual

Escenario 2: Distinción entre nota no subida y nota cero confirmada
  Dado que el archivo cargado contiene dos estudiantes:
    - Estudiante A: nota "0.0" registrada formalmente en el sistema de notas
    - Estudiante B: casilla de nota vacía (docente no ha subido nota)
  Cuando el sistema procesa el archivo
  Entonces el Estudiante A genera alerta MOD-06 vinculada al indicador "Nota Cero Confirmada"
  Y el Estudiante B recibe etiqueta "Nota Pendiente de Registro" — sin generar alerta MOD-06
  Y ambas etiquetas son visualmente distintas en el dashboard del Docente

Escenario 3: No generación de alertas MOD-06 duplicadas
  Dado que ya existe una alerta MOD-06 activa por indicador "Matrícula reiterada" para un estudiante
  Cuando se carga un nuevo archivo con los mismos datos académicos
  Entonces el sistema no genera una segunda alerta MOD-06 duplicada
  Y actualiza la alerta existente con la fecha de última confirmación del indicador
```

**Reglas de negocio:**

* RB-10: El umbral de "matrícula reiterada" es de 3 o más registros en la misma asignatura. Este valor es configurable por el Administrador de Institución sin requerir cambios en el código.
* RB-11: La definición de "nota cero confirmada" requiere que el campo de calificación contenga el valor numérico `0.0` explícito en el archivo importado. Un campo vacío no es equivalente a nota cero.
* RB-12: La detección de indicadores indirectos no opera en tiempo real; se ejecuta como proceso batch inmediatamente después de cada carga masiva de datos (RF-06).
* RB-12b: Las alertas MOD-06 generadas automáticamente por el motor de indicadores indirectos son funcionalmente idénticas a las alertas MOD-06 registradas manualmente por un Docente — siguen el mismo flujo de notificación (RF-04), escalamiento automático (RF-07) y bitácora de seguimiento (RF-05).

---

## RF-04 — Notificaciones en Tiempo Real

| Campo | Detalle |
| :--- | :--- |
| **Código** | RF-04 |
| **Nombre** | Notificaciones Automáticas al Activarse un Indicador de Riesgo |
| **Tipo** | Funcional |
| **Prioridad** | Alta |
| **Actor(es)** | Sistema (emisor), Docente, Coordinador de Convivencia, Padre de Familia (receptor) |

**Descripción:**  
El sistema debe enviar notificaciones automáticas por correo electrónico y por notificación en la aplicación a los actores responsables dentro de los 5 minutos siguientes a la activación de cualquier indicador de riesgo. Cada notificación debe incluir: nombre del estudiante, tipo de alerta activada y enlace directo al perfil del caso.

**Precondición:**
- Una alerta ha sido creada o escalada (RF-02, RF-03 o RF-07).
- El actor destinatario tiene una dirección de correo registrada y activa en el sistema.
- El servidor de cola de tareas (Celery + Redis) está operativo.

**Postcondición:**
- El destinatario recibe la notificación por el canal configurado (correo y/o app) en menos de 5 minutos.
- El sistema registra el estado de entrega: `enviado`, `fallido` o `pendiente`.
- Si el envío falla, el sistema reintenta automáticamente hasta 3 veces con backoff exponencial.

**Criterios de aceptación:**

```gherkin
Escenario 1: Notificación exitosa al Docente
  Dado que se activa una alerta de "Riesgo Académico — Nota Cero" para un estudiante
  Cuando el sistema procesa la notificación
  Entonces el Docente responsable recibe un correo en menos de 5 minutos
  Y el correo contiene: nombre del estudiante, indicador activado y enlace al perfil
  Y la notificación in-app aparece en el panel del Docente al hacer login

Escenario 2: Notificación de alerta Crítica al Coordinador
  Dado que se registra una alerta de módulo "Conducta Suicida" por un Docente
  Cuando el sistema procesa la notificación
  Entonces el Coordinador de Convivencia recibe la notificación en menos de 2 minutos
  Y la notificación tiene marcador visual de nivel "CRÍTICO"
  Y el Docente que registró la alerta recibe copia de confirmación

Escenario 3: Fallo en el envío de notificación
  Dado que el servidor de correo no está disponible temporalmente
  Cuando el sistema intenta enviar una notificación
  Entonces reintenta automáticamente tras 1 minuto, 3 minutos y 9 minutos
  Y si los 3 reintentos fallan, registra el estado "fallido" con código de error
  Y el administrador de sistema recibe un resumen de fallos de entrega cada hora
```

**Reglas de negocio:**
- RB-13: Las notificaciones de módulos `Crítico` (MOD-01 y MOD-03) tienen prioridad de cola máxima y no pueden postergarse por carga del sistema.
- RB-14: Un actor puede configurar sus preferencias de notificación (solo correo, solo app, ambos), pero no puede desactivar completamente las notificaciones de módulos Críticos.
- RB-15: El contenido de la notificación no debe incluir el detalle clínico o descripción completa del incidente — solo el tipo de alerta y el enlace al perfil. Esto protege la confidencialidad en caso de que el correo sea interceptado.
- RB-16: El envío de notificaciones se ejecuta de forma asíncrona mediante colas de tareas (Celery + Redis) para no bloquear el hilo principal de la aplicación durante picos de demanda.

---

## RF-05 — Registro de Seguimiento (Bitácora)

| Campo | Detalle |
| :--- | :--- |
| **Código** | RF-05 |
| **Nombre** | Registro de Seguimiento por Actor Responsable |
| **Tipo** | Funcional |
| **Prioridad** | Alta |
| **Actor(es)** | Docente, Coordinador de Convivencia (escritura), Director de Programa / Rector (lectura) |

**Descripción:**  
El sistema debe permitir a los Docentes y Coordinadores registrar cada intervención realizada sobre un caso de riesgo activo, creando una bitácora inmutable de seguimiento vinculada al historial del estudiante. Cada entrada debe registrar: fecha y hora, tipo de intervención, actor que intervino y observaciones. Los registros no pueden ser eliminados ni modificados una vez guardados.

**Precondición:**
- Existe una alerta activa o en seguimiento para el estudiante.
- El Docente o Coordinador está autenticado y tiene acceso al caso según su jurisdicción.

**Postcondición:**
- La nueva entrada de seguimiento queda persistida en la base de datos vinculada al `alerta_id` y al `estudiante_id`.
- La entrada es inmediatamente visible para el Coordinador de Convivencia y el Rector/Administrador de Institución.
- El estado de la alerta puede cambiar de `Activa` a `En Seguimiento` si es el primer registro de intervención.

**Criterios de aceptación:**

```gherkin
Escenario 1: Registro exitoso de una intervención
  Dado que un Docente tiene una alerta activa asignada a un estudiante
  Cuando registra una entrada de seguimiento con tipo "Llamada al acudiente"
    y la descripción "Se contactó al padre. Confirmó conocimiento de la situación."
  Entonces el sistema guarda la entrada con timestamp del servidor (no del cliente)
  Y la entrada aparece en el historial cronológico del estudiante
  Y el estado de la alerta cambia a "En Seguimiento"

Escenario 2: Intento de modificación de un registro existente
  Dado que existe una entrada de seguimiento guardada hace 2 días
  Cuando el Docente intenta editar el texto de esa entrada
  Entonces el sistema rechaza la operación con mensaje:
    "Los registros de seguimiento son inmutables. Añada una nueva entrada si necesita actualizar."
  Y el registro original no es modificado

Escenario 3: Visibilidad jerárquica del historial
  Dado que existen 5 entradas de seguimiento para el estudiante X
  Cuando el Rector/Administrador de Institución consulta el historial del estudiante X
  Entonces visualiza las 5 entradas en orden cronológico descendente
  Y puede exportar el historial completo en PDF para efectos de reporte institucional
```

**Reglas de negocio:**
- RB-17: El timestamp de cada entrada de seguimiento se genera en el servidor al momento del guardado, no en el cliente. Esto previene manipulación de fechas.
- RB-18: Los registros de seguimiento son inmutables. Solo se permiten operaciones INSERT, nunca UPDATE ni DELETE sobre esta tabla.
- RB-19: El tipo de intervención debe seleccionarse de una lista controlada por el sistema: `Llamada telefónica`, `Reunión presencial`, `Citación al acudiente`, `Remisión a orientación`, `Seguimiento académico`, `Otro`. Si se selecciona `Otro`, el campo de observaciones es obligatorio.
- RB-20: Un Docente no puede ver los registros de seguimiento realizados por otro Docente en el mismo caso si no es el responsable asignado. El Coordinador sí tiene visibilidad completa de todos los registros del caso.

---

## RF-06 — Carga Masiva de Datos

| Campo | Detalle |
| :--- | :--- |
| **Código** | RF-06 |
| **Nombre** | Importación Masiva de Datos Académicos y Demográficos |
| **Tipo** | Funcional |
| **Prioridad** | Alta |
| **Actor(es)** | Administrador de Institución |

**Descripción:**  
El sistema debe permitir al Administrador de Institución importar datos académicos y demográficos de estudiantes mediante archivos CSV o Excel (.xlsx). Antes de procesar la importación, el sistema debe validar la integridad estructural y de contenido del archivo, reportando todos los errores encontrados por fila sin cargar datos parciales ni inconsistentes.

**Precondición:**
- El Administrador de Institución está autenticado con Nivel 4 de acceso.
- El archivo cumple con el formato de plantilla provisto por el sistema (headers definidos).
- El semestre activo está configurado en el sistema.

**Postcondición:**
- Si la validación es exitosa: todos los registros del archivo son importados, el sistema confirma el número de registros procesados y activa el motor de detección de RF-03.
- Si la validación falla: ningún registro es importado. Se genera un reporte de errores descargable con número de fila, columna y descripción del error.

**Criterios de aceptación:**

```gherkin
Escenario 1: Carga exitosa de archivo válido
  Dado que el Administrador sube un archivo CSV con 500 filas correctamente formateadas
  Cuando el sistema completa la validación sin errores
  Entonces importa los 500 registros y muestra el mensaje:
    "500 estudiantes importados exitosamente. Procesando detección de riesgo..."
  Y activa automáticamente el motor de detección de RF-03
  Y registra en auditoría: actor, archivo, timestamp, número de registros importados

Escenario 2: Rechazo de archivo con errores
  Dado que el Administrador sube un archivo con 10 filas donde
    la fila 4 tiene el campo "Código Estudiante" vacío
    y la fila 9 tiene una fecha de nacimiento en formato inválido
  Cuando el sistema finaliza la validación
  Entonces rechaza la importación completa (ninguna fila se guarda)
  Y genera un reporte de errores que indica:
    "Fila 4 — Columna 'Código Estudiante': campo obligatorio vacío"
    "Fila 9 — Columna 'Fecha Nacimiento': formato inválido. Use YYYY-MM-DD"
  Y el Administrador puede descargar el reporte en CSV para corregir el archivo

Escenario 3: Intento de carga de formato no soportado
  Dado que el Administrador intenta subir un archivo .PDF o .ZIP
  Cuando el sistema recibe el archivo
  Entonces lo rechaza antes de procesarlo con mensaje:
    "Formato no soportado. Solo se aceptan archivos .csv y .xlsx"
  Y no almacena ningún dato del archivo rechazado
```

**Reglas de negocio:**
- RB-21: La carga de datos no reemplaza los datos existentes del semestre activo — los agrega. Si un `Código Estudiante` ya existe para el semestre, el sistema actualiza los campos diferidos sin crear duplicados.
- RB-22: El tamaño máximo del archivo aceptado es 10 MB. Archivos de mayor tamaño deben ser divididos por el Administrador.
- RB-23: El sistema provee una plantilla descargable en formato CSV y XLSX con los headers requeridos y ejemplos de datos válidos. No se procesa ningún archivo que no use esta plantilla como base.
- RB-24: La carga se realiza de forma transaccional: o se importan todos los registros válidos o no se importa ninguno. No existe importación parcial.

---

## RF-07 — Escalamiento Automático

| Campo | Detalle |
| :--- | :--- |
| **Código** | RF-07 |
| **Nombre** | Escalamiento Automático de Alertas No Atendidas |
| **Tipo** | Funcional |
| **Prioridad** | Media |
| **Actor(es)** | Sistema (ejecutor automático), Coordinador de Convivencia (escalamiento estándar), Rector (escalamiento crítico) |

**Descripción:**  
El sistema debe escalar automáticamente la visibilidad de una alerta al nivel jerárquico superior cuando el actor responsable no registra ninguna acción de seguimiento dentro del período máximo establecido según el nivel de riesgo de la alerta.

**Períodos de escalamiento:**

| Nivel de riesgo | Módulos | Período antes de escalar |
| :--- | :--- | :--- |
| Crítico | MOD-01 (Abuso), MOD-03 (Conducta Suicida) | 24 horas |
| Alto | MOD-02 (Accidentalidad), MOD-04 (SPA) | 48 horas |
| Medio | MOD-05 (Maternidad), MOD-06 (Trastornos) | 72 horas |

**Precondición:**
- Existe una alerta en estado `Activa` o `En Seguimiento` sin nuevas entradas de bitácora desde hace más del período establecido.
- El reloj del sistema está sincronizado con NTP.

**Postcondición:**
- La alerta es visible para el nivel jerárquico superior (el Coordinador puede escalar al Rector).
- El actor original (Docente o Coordinador) recibe una notificación de que su caso fue escalado por inactividad.
- El campo `escalado_a` de la alerta queda registrado con el rol al que se escaló y el timestamp.

**Criterios de aceptación:**

```gherkin
Escenario 1: Escalamiento de alerta crítica por inactividad
  Dado que existe una alerta de "Conducta Suicida" creada hace 25 horas
    y el Docente responsable no ha registrado ningún seguimiento
  Cuando el proceso automático de verificación se ejecuta
  Entonces la alerta escala al Coordinador de Convivencia
  Y el Coordinador recibe notificación urgente con mensaje:
    "Alerta crítica sin atención — Escalada automáticamente por superar 24h sin seguimiento"
  Y el Docente recibe notificación informando que su caso fue escalado

Escenario 2: No escalamiento cuando hay seguimiento reciente
  Dado que existe una alerta de "SPA" creada hace 50 horas
    pero el Docente registró una entrada de seguimiento hace 10 horas
  Cuando el proceso automático de verificación se ejecuta
  Entonces el sistema no escala la alerta
  Y reinicia el contador de 48 horas desde la última entrada de seguimiento

Escenario 3: Escalamiento en cadena
  Dado que una alerta ya fue escalada al Coordinador hace 48 horas
    y el Coordinador tampoco ha registrado seguimiento
  Cuando el proceso automático de verificación se ejecuta
  Entonces la alerta escala al Rector/Administrador de Institución
  Y se notifica al Coordinador y al Docente del escalamiento en cadena
```

**Reglas de negocio:**
- RB-25: El proceso de verificación de escalamiento se ejecuta cada 30 minutos mediante una tarea programada (Celery Beat).
- RB-26: Una alerta puede escalar como máximo dos niveles (Docente → Coordinador → Rector). No existe escalamiento fuera de la institución.
- RB-27: El escalamiento no transfiere la responsabilidad del Docente original — ambos, el Docente y el nuevo nivel escalado, tienen visibilidad del caso y obligación de actuar.
- RB-28: El período de escalamiento se reinicia con cualquier entrada nueva en la bitácora de seguimiento (RF-05), independientemente de quién la registre.

---

## RF-08 — Consulta de Historial 360

| Campo | Detalle |
| :--- | :--- |
| **Código** | RF-08 |
| **Nombre** | Consulta de Historial Consolidado del Estudiante |
| **Tipo** | Funcional |
| **Prioridad** | Alta |
| **Actor(es)** | Coordinador de Convivencia, Rector/Administrador de Institución, Docente (acceso parcial), Padre de Familia (acceso restringido) |

**Descripción:**  
El sistema debe permitir a los usuarios autorizados consultar el historial consolidado de alertas, seguimientos e intervenciones de un estudiante a través del tiempo, agrupado por módulo y semestre, con capacidad de exportación para efectos de reporte institucional.

**Precondición:**
- El usuario está autenticado y tiene acceso al estudiante consultado según su nivel RBAC y RLS.
- El estudiante tiene al menos un registro en el sistema (alerta, seguimiento o datos académicos).

**Postcondición:**
- Se muestra el historial completo según el nivel de acceso del usuario que consulta.
- Se genera un registro de auditoría con: quién consultó, qué estudiante, qué campos visualizó y cuándo.

**Criterios de aceptación:**

```gherkin
Escenario 1: Consulta completa por el Coordinador
  Dado que el Coordinador de Convivencia selecciona a un estudiante de su sede
  Cuando accede a la vista de "Historial 360"
  Entonces visualiza: todas las alertas históricas (activas y cerradas),
    todos los registros de seguimiento, los indicadores indirectos detectados (RF-03)
    y el estado actual del caso, ordenados cronológicamente
  Y el sistema registra en auditoría el acceso completo

Escenario 2: Acceso restringido del Padre de Familia
  Dado que el Padre de Familia está autenticado con su vínculo validado
  Cuando accede al historial de su hijo
  Entonces solo visualiza: el estado actual de alertas abiertas
    y las notificaciones que le fueron enviadas
  Y no puede ver la descripción detallada de los incidentes
    ni los comentarios de seguimiento de los Docentes

Escenario 3: Exportación del historial
  Dado que el Rector consulta el historial de un estudiante
  Cuando selecciona la opción "Exportar a PDF"
  Entonces el sistema genera un PDF con membrete institucional
    que incluye: datos del estudiante (sin número de documento completo),
    cronología de alertas y seguimientos
  Y el PDF queda disponible para descarga en menos de 10 segundos
  Y se registra en auditoría la exportación realizada
```

**Reglas de negocio:**
- RB-29: El historial de un estudiante es permanente y no puede ser eliminado por ningún rol, incluido el Administrador de Sistema.
- RB-30: En cumplimiento de la Ley 1581 de 2012, el estudiante (si es mayor de 18 años) puede solicitar al Administrador de Institución la consulta de los logs de auditoría que muestran quién ha accedido a su historial.
- RB-31: Los PDF exportados no incluyen el número de identificación completo del estudiante — se muestra solo el último tipo y los últimos 3 dígitos del código institucional para identificación interna.
- RB-32: La función de búsqueda de estudiantes requiere al menos 3 caracteres del nombre o código para ejecutarse, previniendo listados masivos no autorizados.

---

## RF-09 — Revocación de Vínculo

| Campo | Detalle |
| :--- | :--- |
| **Código** | RF-09 |
| **Nombre** | Revocación del Acceso de Padre de Familia |
| **Tipo** | Funcional |
| **Prioridad** | Alta |
| **Actor(es)** | Coordinador de Convivencia (ejecutor), Administrador de Sistema (respaldo técnico) |

**Descripción:**  
El sistema debe permitir al Coordinador de Convivencia revocar de forma inmediata el acceso de un Padre de Familia o acudiente al historial de un estudiante, en situaciones de pérdida de custodia legal, error en la vinculación inicial o solicitud documentada de la institución.

**Precondición:**
- Existe una relación `ParentStudent` activa entre el Padre y el estudiante en el sistema.
- El Coordinador de Convivencia está autenticado con Nivel 3 de acceso.

**Postcondición:**
- La relación `ParentStudent` queda desactivada con estado `revocada`.
- La sesión activa del Padre es invalidada inmediatamente (el JWT en uso queda en lista de revocación).
- El Padre pierde acceso al historial del estudiante en menos de 60 segundos.
- Se genera un registro de auditoría inmutable con: quién revocó, sobre qué vínculo, razón registrada y timestamp.

**Criterios de aceptación:**

```gherkin
Escenario 1: Revocación exitosa por el Coordinador
  Dado que el Coordinador identifica un vínculo incorrecto entre un Padre y un estudiante
  Cuando ejecuta la acción "Revocar acceso" y registra la razón "Error en vinculación inicial"
  Entonces el sistema desactiva la relación ParentStudent en menos de 5 segundos
  Y si el Padre tiene una sesión activa, esta es invalidada inmediatamente
  Y el Padre al intentar acceder recibe mensaje: "Su acceso ha sido revocado. Contacte a la institución."
  Y el sistema genera un log de auditoría con todos los campos requeridos

Escenario 2: Intento de revocación sin razón documentada
  Dado que el Coordinador intenta revocar un vínculo
  Cuando deja el campo "Razón de revocación" vacío
  Entonces el sistema no ejecuta la revocación
  Y muestra el mensaje: "Debe documentar la razón de revocación. Este registro es de carácter legal."

Escenario 3: Consulta del historial de vínculos revocados
  Dado que el Administrador de Sistema consulta el historial de revocaciones de una institución
  Cuando accede al log de auditoría filtrado por evento "Revocación de vínculo"
  Entonces visualiza todos los registros con: actor que revocó, vínculo afectado, razón y timestamp
  Y este historial no puede ser modificado ni eliminado
```

**Reglas de negocio:**
- RB-33: La revocación de un vínculo no elimina el historial de accesos que el Padre tuvo mientras el vínculo estuvo activo. Este historial permanece en el log de auditoría.
- RB-34: Una revocación no puede ser deshecha automáticamente. Si fue un error, se debe crear un nuevo vínculo mediante el proceso OTP completo (RF-01).
- RB-35: El campo "Razón de revocación" es obligatorio y tiene un mínimo de 20 caracteres para asegurar que se documenta la causa real y no un texto vacío de relleno.
- RB-36: El sistema invalida el JWT activo del Padre añadiéndolo a una lista negra de tokens (token blacklist) consultada en cada request. La invalidación no puede depender únicamente de la expiración natural del token.

---

## RF-09 — Actualización: Regla de Negocio RB-37 (Hand-off)

> Agregado por análisis de /grill-me — sesión 2026-04-30

**RB-37 (ampliada):** El sistema no permite que un Docente desvincule manualmente a un estudiante si existen alertas en estado `Activa` o `En Seguimiento` asociadas a ese estudiante. Para proceder, el sistema exige un **Traspaso de Responsabilidad (Hand-off)** explícito:
- El Docente selecciona al Coordinador o al nuevo Docente responsable
- El receptor acepta explícitamente el caso
- Solo entonces se transfiere la vinculación y se corta el acceso del Docente original

**RB-38 — Escalamiento por Abandono:** Si ningún actor acepta el Hand-off en 48 horas, el sistema genera una alerta de **"Riesgo de Ruptura de Ruta"** al Rector/Administrador de Institución, garantizando que ningún caso activo quede en limbo administrativo.

---

## RF-10 — Validación Multinivel de Alertas

| Campo | Detalle |
| :--- | :--- |
| **Código** | RF-10 |
| **Nombre** | Flujo de Validación Multinivel de Alertas |
| **Tipo** | Funcional |
| **Prioridad** | Media *(Should Have — Sprint 2)* |
| **Actor(es)** | Docente (checklist), Sistema (triage), Coordinador de Convivencia (visto bueno) |

**Descripción:**
El sistema debe implementar un flujo de tres niveles de validación antes de que una alerta pueda ser despachada a entes externos o considerada formalmente cerrada: (1) checklist obligatorio basado en los protocolos de la SED Bogotá, (2) triage automático de criticidad por el sistema, y (3) visto bueno del Coordinador como responsable legal institucional.

**Precondición:**
- El Docente ha completado el registro inicial de la alerta (RF-02).
- La alerta está en estado `Activa`.

**Postcondición:**
- La alerta transita al estado `Validada` solo si los tres niveles fueron completados.
- El visto bueno del Coordinador queda registrado como firma de aprobación en el log de auditoría.
- El sistema habilita el despacho externo (notificación a entes) únicamente sobre alertas `Validadas`.

**Campos del checklist SED (Nivel 1 — Docente):**

| Campo | Obligatorio | Descripción |
| :--- | :--- | :--- |
| Identificación del NNA | Sí | Nombre completo y datos de contacto para localización inmediata |
| Descripción de Hechos | Sí | Circunstancias de **tiempo, modo y lugar** del incidente |
| Registro de Acciones | Sí | Acciones tomadas: entidad notificada (ej. Línea 123), fecha y forma de reporte |
| Testigos / Evidencias | No | Personas presentes o evidencias documentales disponibles |
| Estado del NNA al momento | Sí | Condición física y emocional observada por el Docente |

**Criterios de aceptación:**

```gherkin
Escenario 1: Checklist bloqueante antes del envío
  Dado que un Docente registra una alerta de "Abuso y Violencia"
  Cuando intenta guardar la alerta sin completar el campo "Descripción de Hechos"
  Entonces el sistema deshabilita el botón "Guardar"
  Y muestra el mensaje: "Complete todos los campos obligatorios del checklist SED antes de continuar"
  Y no crea ningún registro en la base de datos

Escenario 2: Triage automático de criticidad (Nivel 2)
  Dado que un Docente completa el checklist de una alerta MOD-01 (Abuso)
  Cuando el sistema procesa el triage de criticidad
  Entonces clasifica la alerta como "Crítica" por pertenecer a MOD-01
  Y envía una notificación preliminar inmediata al Coordinador y Psicólogo/Orientador
  Y mantiene la alerta en estado "En Revisión" hasta el visto bueno del Coordinador

Escenario 3: Visto bueno del Coordinador (Nivel 3)
  Dado que una alerta está en estado "En Revisión" con checklist completo
  Cuando el Coordinador revisa el caso y selecciona "Aprobar para despacho"
  Entonces la alerta cambia a estado "Validada"
  Y se registra en auditoría: quién aprobó, timestamp y hash del contenido aprobado
  Y el sistema habilita el despacho externo de la alerta
```

**Reglas de negocio:**
- RB-39: El triage de criticidad es automático e inmediato al completar el checklist. El Docente no puede modificar la clasificación asignada por el sistema.
- RB-40: El Coordinador no puede aprobar una alerta con campos del checklist vacíos, aunque la alerta haya pasado el triage.
- RB-41: El "visto bueno" del Coordinador no es un checkbox — es una acción confirmada con reingreso de contraseña o PIN institucional para garantizar la firma digital como acto consciente y legal.
- RB-42: Las alertas de nivel Crítico (MOD-01, MOD-03) siguen activando notificación preliminar inmediata al Coordinador/Psicólogo independientemente de si el checklist está completo o no. La notificación preliminar no espera la validación — la validación es para el despacho formal externo.

---

## RF-11 — Confirmación Activa de Grupo

| Campo | Detalle |
| :--- | :--- |
| **Código** | RF-11 |
| **Nombre** | Confirmación Activa de Vigencia de Grupo (Privilegios Residuales) |
| **Tipo** | Funcional |
| **Prioridad** | Media *(Should Have — Sprint 2)* |
| **Actor(es)** | Docente (confirmación), Sistema (bloqueo RLS), Admin Institución (carga Delta-CSV) |

**Descripción:**
El sistema debe exigir al Docente una confirmación periódica (cada 15 días) de que su lista de estudiantes asignados es vigente y correcta, con el fin de eliminar los "Privilegios Residuales" que generarían violación de Ley 1581 cuando un estudiante se transfiere o un Docente es reemplazado. La omisión de esta confirmación activa un bloqueo preventivo RLS sobre los detalles sensibles de las alertas.

**Precondición:**
- El Docente está autenticado y tiene estudiantes asignados en el sistema.
- Han transcurrido 15 o más días desde el valor de `last_verified_at` en la relación `TeacherCourse`.

**Postcondición:**
- Si confirma: `last_verified_at` se actualiza con el timestamp actual y el acceso se mantiene sin restricciones.
- Si desvincula un estudiante durante la confirmación: la relación `TeacherCourse` se cierra, el RLS bloquea el acceso al historial de ese estudiante, y se genera un log de auditoría.
- Si omite: el RLS oculta los detalles de las alertas activas hasta que se complete la confirmación.

**Criterios de aceptación:**

```gherkin
Escenario 1: Bloqueo preventivo por omisión de confirmación
  Dado que han transcurrido 15 días desde la última confirmación del Docente
  Cuando el Docente accede al dashboard
  Entonces el sistema muestra los estudiantes con indicador rojo (validación vencida)
  Y oculta los detalles de las alertas activas (solo muestra nombre y módulo)
  Y presenta el banner: "Tu lista de estudiantes requiere verificación para acceder a detalles sensibles"

Escenario 2: Confirmación exitosa sin cambios
  Dado que el Docente ve su lista de estudiantes en el proceso de confirmación
  Cuando selecciona "Confirmar — todos mis estudiantes son vigentes"
  Entonces el campo last_verified_at se actualiza con el timestamp del servidor
  Y el RLS restaura el acceso completo a todos los detalles de las alertas
  Y los indicadores semáforo vuelven a verde

Escenario 3: Desvinculación manual de estudiante transferido
  Dado que el Docente identifica en su lista un estudiante que ya no está a su cargo
  Cuando lo marca como "Retirado de mi grupo" durante el proceso de confirmación
    y el estudiante no tiene alertas activas
  Entonces el sistema cierra la relación TeacherCourse para ese estudiante
  Y el Docente pierde inmediatamente acceso al historial de ese estudiante
  Y el estudiante queda visible solo para el Coordinador hasta ser reasignado
```

**Reglas de negocio:**
- RB-43: El bloqueo RLS por omisión de confirmación es preventivo, no punitivo. El Docente puede desbloquear en cualquier momento completando la confirmación; no requiere intervención del Admin.
- RB-44: Un Docente no puede desvincular manualmente a un estudiante con alertas activas o en seguimiento — aplica la regla RB-37 (Hand-off obligatorio).
- RB-45: La carga de un Delta-CSV por el Administrador actualiza automáticamente el estado semáforo de los estudiantes afectados (de verde a amarillo), pero no reemplaza la Confirmación Activa del Docente.
- RB-46: Si el campo `last_verified_at` nunca ha sido registrado (Docente nuevo), el sistema le da un periodo de gracia de 3 días desde la primera asignación antes de activar el bloqueo.

---

## Trazabilidad de requisitos

| RF | Actores involucrados | RNF relacionados | Restricciones aplicables |
| :--- | :--- | :--- | :--- |
| RF-01 | Admin TI, todos los roles | RNF-03 (Seguridad) | RE-01 (Habeas Data), RE-03 (Stack) |
| RF-02 | Docente, Coordinador, Estudiante | RNF-01 (Rendimiento), RNF-03 | RE-01, RE-02 (Ley 1620) |
| RF-03 | Admin Institución, Docente | RNF-01, RNF-04 (Elasticidad) | RE-04 (Sin integración SIAU) |
| RF-04 | Sistema, Docente, Coordinador, Padre | RNF-01, RNF-02 (Disponibilidad), RNF-06 (Resiliencia) | RE-01, RE-03 |
| RF-05 | Docente, Coordinador, Rector | RNF-03 | RE-01, RE-02 |
| RF-06 | Admin Institución | RNF-01, RNF-04 | RE-03, RE-04 |
| RF-07 | Sistema, Coordinador, Rector | RNF-02, RNF-04, RNF-06 | RE-02 |
| RF-08 | Coordinador, Rector, Docente, Padre | RNF-01, RNF-03, RNF-05 (Usabilidad) | RE-01 |
| RF-09 | Coordinador, Admin TI | RNF-03 | RE-01 |
| RF-10 | Docente, Sistema, Coordinador | RNF-03, RNF-06 | RE-01, RE-02, RE-05 |
| RF-11 | Docente, Sistema, Admin Institución | RNF-03 | RE-01, RE-04 |

---

*Documento preparado por el equipo Alerta Temprana 360.*  
*Versión 1.0 — Se actualiza al cierre de cada Sprint con las decisiones del profesor (stakeholder).*
