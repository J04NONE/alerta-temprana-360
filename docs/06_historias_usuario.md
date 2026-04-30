# 06 — Historias de Usuario: Alerta Temprana 360

**Estándar:** ISO/IEC/IEEE 29148:2018 + criterios INVEST  
**Versión:** 1.0  
**Fecha:** 2026-04-30  
**Trazabilidad:** Cada HU referencia su RF de origen en `03_matriz_requisitos.md`.

---

## Convenciones

**Criterios INVEST:**
- **I**ndependiente: implementable sin depender de otra HU
- **N**egociable: define el qué, no el cómo
- **V**aliosa: entrega valor real al actor
- **E**stimable: el equipo puede calcular su esfuerzo
- **S**mall: cabe en un Sprint de 1 semana
- **T**esteable: tiene criterios de aceptación verificables

**Escala de estimación (Story Points):** 1 (trivial) · 2 (pequeña) · 3 (media) · 5 (grande) · 8 (muy grande)

**Prioridad MoSCoW:** Must Have · Should Have · Could Have · Won't Have (MVP)

---

## Épica 1 — Gestión de Acceso y Seguridad

---

### HU-01 | Registro de Docente por el Administrador

**Como** Administrador de Sistema,  
**quiero** registrar a un nuevo Docente con su sede y nivel de acceso asignados,  
**para** que pueda autenticarse y ver únicamente los estudiantes y alertas de su jurisdicción.

**INVEST:** ✅ Independiente · ✅ Negociable · ✅ Valiosa · ✅ Estimable · ✅ Small · ✅ Testeable

**Criterios de aceptación:**
```gherkin
Dado que el Administrador de Sistema está autenticado
Cuando registra un nuevo Docente con correo, sede "IED Norte" y rol "Docente"
Entonces el sistema crea la cuenta con Nivel 2 de acceso vinculado a esa sede
Y el Docente puede autenticarse y ver solo los estudiantes de "IED Norte"
Y se genera un log de auditoría con el evento de creación

Dado que el Administrador intenta registrar un Docente con un correo ya existente
Cuando envía el formulario con ese correo duplicado
Entonces el sistema rechaza la operación con mensaje:
  "El correo ya está registrado en el sistema"
Y no crea ninguna cuenta duplicada
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 3 |
| **Prioridad** | Must Have |
| **Sprint** | Sprint 1 |
| **RF relacionado** | RF-01 |

---

### HU-02 | Vinculación de Padre de Familia mediante OTP

**Como** Padre de Familia / Acudiente,  
**quiero** registrarme en el sistema usando el token que recibí de la institución,  
**para** recibir notificaciones y consultar el estado de las alertas de mi hijo sin que mis datos de identidad queden almacenados en el sistema.

**INVEST:** ✅ Independiente · ✅ Negociable · ✅ Valiosa · ✅ Estimable · ✅ Small · ✅ Testeable

> Esta HU es el **eje central del cumplimiento de la Ley 1581 (Habeas Data)** para el acceso de terceros.

**Criterios de aceptación:**
```gherkin
Dado que la institución generó un OTP para el acudiente verificado de "Juan García"
Cuando el Padre ingresa el OTP en el formulario de registro
Entonces el sistema valida el token, crea la relación ParentStudent con "Juan García"
Y el token queda marcado como "utilizado" y no puede reutilizarse
Y el Padre accede solo al historial de alertas de "Juan García" (Nivel 1 Restringido)
Y se genera log de auditoría con: quién se vinculó, cuándo y a qué estudiante

Dado que el OTP tiene más de 72 horas desde su generación
Cuando el Padre intenta usarlo para registrarse
Entonces el sistema rechaza el token con mensaje:
  "El token de vinculación ha vencido. Solicite uno nuevo en la institución."
Y no crea ningún vínculo ni cuenta

Dado que un Padre intenta usar un OTP que ya fue utilizado previamente
Cuando ingresa ese token
Entonces el sistema lo rechaza con mensaje:
  "Este token ya fue utilizado. Cada token es de un solo uso."
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 5 |
| **Prioridad** | Must Have |
| **Sprint** | Sprint 1 |
| **RF relacionado** | RF-01 |

---

### HU-03 | Confirmación Activa de Vigencia de Grupo

**Como** Docente,  
**quiero** confirmar periódicamente que mi lista de estudiantes asignados es correcta y vigente,  
**para** garantizar que no mantengo acceso a información sensible de menores que ya no están bajo mi cargo, cumpliendo la Ley 1581.

**INVEST:** ✅ Independiente · ✅ Negociable · ✅ Valiosa · ✅ Estimable · ✅ Small · ✅ Testeable

**Criterios de aceptación:**
```gherkin
Dado que han pasado 15 días desde mi última confirmación
Cuando accedo al dashboard
Entonces el sistema muestra mis estudiantes con indicador rojo (confirmación vencida)
Y oculta los detalles de alertas activas (muestra solo nombre y módulo)
Y presenta el banner: "Verifica tu lista de estudiantes para restablecer el acceso completo"

Dado que realizo la confirmación y todos mis estudiantes son vigentes
Cuando selecciono "Confirmar — lista correcta"
Entonces el campo last_verified_at se actualiza con el timestamp del servidor
Y los indicadores semáforo vuelven a verde
Y recupero acceso completo a los detalles de las alertas

Dado que identifico un estudiante transferido durante la confirmación
  y ese estudiante no tiene alertas activas
Cuando lo marco como "Ya no está en mi grupo"
Entonces el sistema cierra mi vínculo con ese estudiante inmediatamente
Y pierdo acceso a su historial
Y el coordinador recibe notificación de que el estudiante quedó sin docente asignado
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 5 |
| **Prioridad** | Should Have |
| **Sprint** | Sprint 2 |
| **RF relacionado** | RF-11 |

---

### HU-04 | Revocación del Acceso de un Padre de Familia

**Como** Coordinador de Convivencia,  
**quiero** revocar inmediatamente el acceso de un Padre de Familia al historial de su hijo,  
**para** proteger la privacidad del estudiante en casos de pérdida de custodia o error en la vinculación, cumpliendo la Ley 1581.

**INVEST:** ✅ Independiente · ✅ Negociable · ✅ Valiosa · ✅ Estimable · ✅ Small · ✅ Testeable

**Criterios de aceptación:**
```gherkin
Dado que el Coordinador identifica un vínculo incorrecto entre un Padre y un estudiante
Cuando ejecuta "Revocar acceso" y registra la razón "Error en vinculación inicial"
Entonces el sistema desactiva la relación ParentStudent en menos de 5 segundos
Y si el Padre tiene sesión activa, esta se invalida inmediatamente (token blacklist)
Y el Padre al intentar acceder recibe: "Su acceso ha sido revocado. Contacte a la institución."
Y se genera log de auditoría inmutable con: quién revocó, vínculo afectado, razón y timestamp

Dado que el Coordinador intenta revocar sin documentar la razón
Cuando deja el campo "Razón de revocación" vacío
Entonces el sistema bloquea la operación con mensaje:
  "Debe documentar la razón. Este registro es de carácter legal."
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 3 |
| **Prioridad** | Must Have |
| **Sprint** | Sprint 1 |
| **RF relacionado** | RF-09 |

---

## Épica 2 — Registro y Gestión de Alertas SED

---

### HU-05 | Registro de una Alerta en Módulo SED

**Como** Docente,  
**quiero** reportar un incidente de un estudiante seleccionando el módulo SED correspondiente,  
**para** que el sistema active el flujo de atención institucional y el Coordinador sea notificado de inmediato.

**INVEST:** ✅ Independiente · ✅ Negociable · ✅ Valiosa · ✅ Estimable · ✅ Small · ✅ Testeable

> Esta HU es el **valor nuclear del sistema** — sin ella, AT360 no tiene razón de ser.

**Criterios de aceptación:**
```gherkin
Dado que un Docente autenticado navega al perfil de uno de sus estudiantes
Cuando selecciona módulo "Conducta Suicida", completa la descripción y guarda
Entonces la alerta se crea con estado "Activa" y nivel de riesgo "Crítico"
Y el Coordinador recibe notificación en menos de 2 minutos
Y la alerta aparece en el dashboard del Coordinador con indicador visual CRÍTICO

Dado que el Docente intenta registrar una alerta sin completar la descripción
Cuando intenta guardar con ese campo vacío
Entonces el sistema bloquea el guardado con mensaje de error específico
Y no crea ningún registro parcial en la base de datos

Dado que un Docente intenta registrar una alerta de un estudiante que no es suyo
Cuando hace la petición
Entonces el sistema la rechaza con HTTP 403
Y no expone ningún dato del estudiante ajeno
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 5 |
| **Prioridad** | Must Have |
| **Sprint** | Sprint 1 |
| **RF relacionado** | RF-02 |

---

### HU-06 | Completar Checklist SED antes de Guardar Alerta

**Como** Docente,  
**quiero** completar un checklist estructurado con los campos del protocolo SED (tiempo, modo, lugar del incidente),  
**para** asegurar que el reporte tenga la calidad necesaria para ser aceptado por entes externos y no sea devuelto por datos incompletos.

**Criterios de aceptación:**
```gherkin
Dado que un Docente está registrando una alerta de "Abuso y Violencia"
Cuando deja el campo "Descripción de Hechos (tiempo/modo/lugar)" vacío
Entonces el botón "Guardar" permanece deshabilitado
Y el sistema resalta en rojo el campo faltante

Dado que el Docente completa todos los campos obligatorios del checklist
Cuando hace clic en "Guardar"
Entonces la alerta se crea exitosamente y el checklist queda guardado como parte del registro
Y el campo "Registro de Acciones" queda vacío como opcional si no se ha notificado a ninguna entidad
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 3 |
| **Prioridad** | Should Have |
| **Sprint** | Sprint 2 |
| **RF relacionado** | RF-10 |

---

### HU-07 | Triage Automático de Criticidad por el Sistema

**Como** Coordinador de Convivencia,  
**quiero** que el sistema clasifique automáticamente el nivel de criticidad de cada alerta al momento de su registro,  
**para** priorizar visualmente los casos más urgentes en mi dashboard sin tener que revisar uno por uno.

**Criterios de aceptación:**
```gherkin
Dado que se registra una alerta en módulo "Conducta Suicida" (MOD-03)
Cuando el sistema procesa el triage
Entonces clasifica la alerta como "Crítica" y la marca con indicador rojo en el dashboard
Y envía notificación preliminar al Coordinador y al Psicólogo/Orientador

Dado que se registran alertas en módulos de diferente criticidad
Cuando el Coordinador abre su dashboard
Entonces las alertas aparecen ordenadas: Críticas primero, luego Altas, luego Medias
Y puede identificar la más urgente en menos de 10 segundos
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 3 |
| **Prioridad** | Should Have |
| **Sprint** | Sprint 2 |
| **RF relacionado** | RF-10 |

---

### HU-08 | Visto Bueno del Coordinador para Despacho Externo

**Como** Coordinador de Convivencia,  
**quiero** aprobar formalmente el despacho de una alerta a entes externos mediante mi firma digital en el sistema,  
**para** asumir la responsabilidad legal institucional del reporte y garantizar la cadena de custodia.

**Criterios de aceptación:**
```gherkin
Dado que una alerta completó el checklist y el triage automático
Cuando el Coordinador revisa el caso y selecciona "Aprobar para despacho"
  y confirma con su PIN institucional
Entonces la alerta cambia a estado "Validada"
Y el log de auditoría registra: quién aprobó, timestamp y hash del contenido aprobado
Y se habilita el despacho externo de la alerta

Dado que el Coordinador intenta aprobar sin ingresar el PIN
Cuando intenta confirmar la acción sin autenticación adicional
Entonces el sistema bloquea la operación
Y muestra: "Ingrese su PIN para confirmar esta acción de responsabilidad legal"
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 5 |
| **Prioridad** | Should Have |
| **Sprint** | Sprint 2 |
| **RF relacionado** | RF-10 |

---

## Épica 3 — Seguimiento y Continuidad de Casos

---

### HU-09 | Registrar una Intervención en la Bitácora de Seguimiento

**Como** Docente o Coordinador,  
**quiero** registrar cada intervención realizada sobre un caso activo (llamada, reunión, citación),  
**para** mantener un historial inmutable que demuestre el acompañamiento institucional y proteja a la institución legalmente.

**Criterios de aceptación:**
```gherkin
Dado que un Docente tiene una alerta activa asignada
Cuando registra una entrada con tipo "Llamada al acudiente"
  y descripción "Se contactó al padre. Confirmó conocimiento de la situación."
Entonces el sistema guarda la entrada con timestamp del servidor (no del cliente)
Y el estado de la alerta cambia a "En Seguimiento"
Y el Coordinador puede ver la entrada inmediatamente

Dado que el Docente intenta editar una entrada guardada hace 2 días
Cuando intenta modificarla
Entonces el sistema rechaza con: "Los registros de bitácora son inmutables. Añada una nueva entrada."
Y el registro original no es modificado
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 3 |
| **Prioridad** | Must Have |
| **Sprint** | Sprint 1 |
| **RF relacionado** | RF-05 |

---

### HU-10 | Escalamiento Automático de Alerta No Atendida

**Como** Coordinador de Convivencia,  
**quiero** recibir automáticamente las alertas que el Docente no atendió en el tiempo establecido,  
**para** garantizar que ningún caso crítico quede sin respuesta institucional.

**Criterios de aceptación:**
```gherkin
Dado que existe una alerta de "Conducta Suicida" sin seguimiento por más de 24 horas
Cuando el proceso automático de verificación se ejecuta (cada 30 minutos)
Entonces la alerta escala al Coordinador con notificación urgente
Y el Docente recibe aviso de que su caso fue escalado por inactividad

Dado que el Docente registró una entrada de seguimiento hace 10 horas en una alerta de 50h
Cuando el proceso de verificación se ejecuta
Entonces el sistema no escala (contador reiniciado desde el último seguimiento)
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 5 |
| **Prioridad** | Must Have |
| **Sprint** | Sprint 1 |
| **RF relacionado** | RF-07 |

---

### HU-11 | Traspaso de Responsabilidad (Hand-off) de un Caso

**Como** Docente,  
**quiero** transferir formalmente la responsabilidad de un caso activo a otro Docente o al Coordinador,  
**para** garantizar que el estudiante sigue siendo atendido cuando yo dejo de ser su responsable, sin crear alertas huérfanas.

**Criterios de aceptación:**
```gherkin
Dado que un Docente intenta desvincular un estudiante con una alerta activa
Cuando selecciona "Retirar de mi grupo"
Entonces el sistema bloquea la operación con mensaje:
  "Este estudiante tiene alertas activas. Debe completar el Traspaso de Responsabilidad."
Y muestra el formulario de Hand-off con la lista de posibles receptores

Dado que el Docente selecciona al Coordinador como receptor del Hand-off
Cuando el Coordinador acepta explícitamente el caso
Entonces el vínculo del Docente original se cierra
Y el Coordinador queda como responsable del caso
Y ambos (Docente y Coordinador) reciben confirmación del traspaso con log de auditoría

Dado que nadie acepta el Hand-off en 48 horas
Cuando el proceso automático de verificación se ejecuta
Entonces el sistema genera una alerta de "Riesgo de Ruptura de Ruta" al Rector
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 5 |
| **Prioridad** | Should Have |
| **Sprint** | Sprint 2 |
| **RF relacionado** | RF-09 |

---

## Épica 4 — Datos Académicos

---

### HU-12 | Carga Masiva de Datos Académicos (CSV completo)

**Como** Administrador de Institución,  
**quiero** importar el archivo CSV completo con los datos académicos del semestre,  
**para** que el sistema detecte automáticamente los estudiantes en riesgo académico sin requerir integración directa con el SIAU.

**Criterios de aceptación:**
```gherkin
Dado que el Administrador sube un CSV válido con 500 filas
Cuando el sistema completa la validación sin errores
Entonces importa los 500 registros y confirma: "500 estudiantes importados. Procesando detección de riesgo..."
Y activa el motor de detección de RF-03 automáticamente

Dado que el archivo tiene errores en 2 filas
Cuando el sistema finaliza la validación
Entonces rechaza toda la carga y genera reporte descargable con fila, columna y descripción del error
Y ninguna fila se importa (transacción atómica)
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 5 |
| **Prioridad** | Must Have |
| **Sprint** | Sprint 1 |
| **RF relacionado** | RF-06 |

---

### HU-13 | Carga Incremental de Novedades de Matrícula (Delta-CSV)

**Como** Administrador de Institución,  
**quiero** subir solo los cambios de matrícula (traslados, retiros, nuevas asignaciones) sin tener que recargar toda la base de datos,  
**para** que el sistema actualice los permisos de acceso en tiempo real y elimine los Privilegios Residuales.

**Criterios de aceptación:**
```gherkin
Dado que el Administrador sube un Delta-CSV con 5 novedades de traslado
Cuando el sistema procesa el archivo
Entonces actualiza solo las relaciones TeacherCourse afectadas
Y los Docentes de los estudiantes trasladados ven el indicador amarillo (novedad detectada)
Y no modifica los registros de estudiantes no mencionados en el delta

Dado que el delta incluye el retiro de un estudiante con alerta activa
Cuando el sistema procesa el retiro
Entonces no cierra el vínculo automáticamente
Y genera notificación al Coordinador: "Estudiante con alerta activa registrado como retirado — requiere Hand-off"
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 5 |
| **Prioridad** | Should Have |
| **Sprint** | Sprint 2 |
| **RF relacionado** | RF-06, RF-11 |

---

## Épica 5 — Dashboard y Visibilidad

---

### HU-14 | Consultar el Historial 360 de un Estudiante

**Como** Coordinador de Convivencia,  
**quiero** ver el historial consolidado de alertas y seguimientos de cualquier estudiante de mi sede en una sola vista,  
**para** tener contexto completo al tomar decisiones de intervención o escalar un caso al Rector.

**Criterios de aceptación:**
```gherkin
Dado que el Coordinador selecciona a un estudiante de su sede
Cuando accede a la vista "Historial 360"
Entonces visualiza: todas las alertas (activas y cerradas), todos los seguimientos
  y el estado de riesgo académico, ordenados cronológicamente
Y el sistema genera log de auditoría del acceso

Dado que el Coordinador selecciona "Exportar a PDF"
Cuando el sistema genera el documento
Entonces el PDF incluye: datos del estudiante (sin número de ID completo),
  cronología de alertas y seguimientos
Y queda disponible para descarga en menos de 10 segundos

Dado que el Padre de Familia accede al historial de su hijo vinculado
Cuando consulta la vista
Entonces solo ve el estado actual de alertas abiertas y las notificaciones que recibió
Y NO ve la descripción detallada de incidentes ni los comentarios de seguimiento
```

| Campo | Valor |
| :--- | :--- |
| **Story Points** | 5 |
| **Prioridad** | Must Have |
| **Sprint** | Sprint 1 |
| **RF relacionado** | RF-08 |

---

## Resumen de Historias de Usuario

| ID | Título | Épica | SP | Prioridad | Sprint | RF |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| HU-01 | Registro de Docente | Gestión de Acceso | 3 | Must Have | Sprint 1 | RF-01 |
| HU-02 | Vinculación Padre OTP | Gestión de Acceso | 5 | Must Have | Sprint 1 | RF-01 |
| HU-03 | Confirmación Activa de Grupo | Gestión de Acceso | 5 | Should Have | Sprint 2 | RF-11 |
| HU-04 | Revocación Vínculo Padre | Gestión de Acceso | 3 | Must Have | Sprint 1 | RF-09 |
| HU-05 | Registro de Alerta SED | Alertas SED | 5 | Must Have | Sprint 1 | RF-02 |
| HU-06 | Checklist SED Obligatorio | Alertas SED | 3 | Should Have | Sprint 2 | RF-10 |
| HU-07 | Triage Automático | Alertas SED | 3 | Should Have | Sprint 2 | RF-10 |
| HU-08 | Visto Bueno Coordinador | Alertas SED | 5 | Should Have | Sprint 2 | RF-10 |
| HU-09 | Bitácora de Seguimiento | Seguimiento | 3 | Must Have | Sprint 1 | RF-05 |
| HU-10 | Escalamiento Automático | Seguimiento | 5 | Must Have | Sprint 1 | RF-07 |
| HU-11 | Hand-off de Caso | Seguimiento | 5 | Should Have | Sprint 2 | RF-09 |
| HU-12 | Carga Masiva CSV | Datos Académicos | 5 | Must Have | Sprint 1 | RF-06 |
| HU-13 | Delta-CSV Incremental | Datos Académicos | 5 | Should Have | Sprint 2 | RF-06, RF-11 |
| HU-14 | Historial 360 | Dashboard | 5 | Must Have | Sprint 1 | RF-08 |
| **Total** | | | **65 SP** | | | |

---

*Documento preparado por el equipo Alerta Temprana 360.*  
*Versión 1.0 — Se actualiza al cierre de cada Sprint.*
