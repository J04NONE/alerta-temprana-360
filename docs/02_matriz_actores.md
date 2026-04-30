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

## Consideraciones de Seguridad y Acceso
*   **Aislamiento de Datos (RLS):** Los registros de alertas están aislados a nivel de base de datos. Un Docente solo puede consultar filas vinculadas a su `sede_id` o `grupo_id`.
*   **Mecanismos de Control de Acceso (Gestión de Consentimiento):** 
    *   **Token de Vinculación Institucional:** Para garantizar el cumplimiento de la Ley 1581 (Habeas Data) sin interoperabilidad con el SIAU, el registro de Padres de Familia utiliza autenticación fuera de banda (*Out-of-band Authentication*). La institución genera un Token Único no reutilizable (OTP) que se entrega físicamente al acudiente verificado. El sistema valida este secreto compartido para establecer la relación `ParentStudent` sin almacenar documentos de identidad sensibles.
    *   **Trazabilidad:** Cada vinculación genera un log de auditoría inalterable.
*   **Escalamiento Automático:** Si una alerta de "Conducta Suicida" o "Abuso" no es atendida por un Docente en un tiempo parametrizado, el sistema escala automáticamente la visibilidad al Coordinador.
