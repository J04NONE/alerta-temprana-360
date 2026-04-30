# 02 — Identificación de Actores: Alerta Temprana 360

Siguiendo el estándar ISO/IEC/IEEE 29148:2018, se definen los actores que interactúan con el sistema, sus responsabilidades y niveles de acceso a la información sensible.

| Actor | Responsabilidad Principal | Nivel de Acceso (RBAC) |
| :--- | :--- | :--- |
| **Estudiante** | Sujeto de protección. Puede reportar autovulneraciones o incidentes. | **Nivel 1:** Consulta de su propio historial de alertas y estados de seguimiento. No ve detalles de otros estudiantes. |
| **Docente** | Identificador de primera línea. Registra alertas observadas y realiza seguimientos directos. | **Nivel 2:** Creación de alertas y registro de seguimientos para estudiantes a su cargo. No ve alertas de otros cursos/grados. |
| **Padre de Familia / Acudiente** | Corresponsable en el proceso de formación. Recibe notificaciones y consulta el estado de alertas de su hijo. | **Nivel 1 (Restringido):** Solo acceso a alertas y seguimientos del estudiante vinculado. Sujeto a validación de parentesco y Habeas Data. |
| **Coordinador de Convivencia** | Gestor de casos. Supervisa las alertas de su sede, asigna responsables y escala casos críticos. | **Nivel 3:** Visibilidad completa de alertas en su sede/localidad. Capacidad de edición y escalamiento. |
| **Administrador de Institución / Rector** | Supervisión estratégica. Consulta métricas agregadas y asegura que se cumplan los protocolos. | **Nivel 4:** Acceso a reportes estadísticos y auditoría. No necesariamente accede al detalle íntimo a menos que el protocolo lo exija. |
| **Administrador de Sistema (TI)** | Mantenimiento técnico, gestión de roles y auditoría de logs. | **Nivel 5:** Gestión técnica. Acceso a logs de auditoría (quién accedió a qué), pero restringido de ver contenido sensible por RLS. |
| **Michael (SM/Dev)** | Gestión del equipo, documentación técnica y desarrollo de lógica de negocio en Backend. Responsable de la integridad de reglas de RLS en Django/Supabase y coordinación de sprints. | **Nivel 5:** Acceso total a infraestructura y código. Co-responsable con Iván de la arquitectura técnica. |
| **Psicólogo / Orientador Escolar** | Stakeholder de Especialidad. Recibe remisiones formales de casos MOD-01 (Abuso) y MOD-03 (Conducta Suicida). Registra seguimientos clínicos y actúa como enlace con la Ruta de Atención Integral (RAI) definida por la Ley 1620 de 2013. | **Nivel 2 Especializado:** Acceso por `caso_id` (remisión explícita), **no** por `grupo_id`. No puede consultar el historial de un NNA sin una remisión activa a su nombre. |

## Consideraciones de Seguridad y Acceso

*   **Aislamiento de Datos (RLS):** Los registros de alertas están aislados a nivel de base de datos. Un Docente solo puede consultar filas vinculadas a su `sede_id` o `grupo_id`. El Psicólogo/Orientador accede únicamente por `caso_id` de remisión.

*   **Ciclo de Vida de Vinculación (Ley 1581 — Privilegios Residuales):**
    La relación `TeacherCourse` incluye los campos `valid_until` (fecha de vigencia del vínculo) y `last_verified_at` (última confirmación activa del Docente). Si `last_verified_at` supera 15 días sin renovación, el RLS bloquea preventivamente el acceso del Docente a los detalles sensibles de las alertas hasta que realice la Confirmación Activa (RF-11).
    *   **UI de Semáforo:** El dashboard del Docente clasifica su lista de estudiantes en: 🟢 verde (sin cambios), 🟡 amarillo (novedad detectada en carga Delta-CSV) y 🔴 rojo (validación vencida — acceso bloqueado).
    *   **Principio Fail-Safe:** Si el periodo académico cambia sin nueva carga CSV, el acceso a detalles sensibles se restringe por defecto.

*   **Token de Vinculación Institucional (Padre de Familia):**
    Para cumplir la Ley 1581 sin interoperabilidad con el SIAU, el registro de Padres usa autenticación fuera de banda (*Out-of-band*). La institución genera un Token OTP no reutilizable (vigencia 72h) entregado físicamente al acudiente verificado. El sistema establece la relación `ParentStudent` sin almacenar documentos de identidad sensibles. Cada vinculación genera un log de auditoría inalterable.

*   **Escalamiento Automático y Escalamiento por Abandono:**
    Si una alerta crítica (MOD-01, MOD-03) no es atendida en 24h, escala al Coordinador. Si ningún actor acepta el Hand-off de un caso activo en 48h tras ser transferido, el sistema genera una alerta de **"Riesgo de Ruptura de Ruta"** al Rector/Administrador de Institución.

*   **Estado de Remisión en el flujo de alerta:**
    Las alertas pueden transitar al estado `Remitido a Orientación` cuando el Coordinador asigna el caso al Psicólogo/Orientador. Este estado es visible para el Docente original y para el Rector.
