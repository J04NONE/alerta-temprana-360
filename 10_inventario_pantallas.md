# 10 — Inventario de Pantallas: Alerta Temprana 360

**Estándar:** ISO/IEC/IEEE 29148:2018  
**Versión:** 1.0  
**Fecha:** 2026-05-08  
**Alcance:** Dashboard, Módulos SED, Confirmación de Listas (HU-03, RF-11)

---

## 1. Autenticación

### P-01 | Login
- **URL:** `/auth/login/`
- **Roles:** Todos
- **Función:** Autenticación con correo + contraseña. JWT de 8h.
- **Campos:** correo, contraseña
- **Acciones:** "Recordarme" (expansión a 24h para móvil), link "¿Olvidó contraseña?"
- **UX:** Muestra error específico si credenciales inválidas. Token enviado por email para recuperación.

### P-02 | Logout
- **URL:** `/auth/logout/`
- **Roles:** Todos
- **Función:** Invalidar sesión, limpiar token blacklist, redirigir a login.

### P-03 | Recuperar Contraseña
- **URL:** `/auth/password-reset/`
- **Roles:** Ninguno autenticado
- **Función:** Solicitar email → envío de enlace con token de un solo uso (15 min).

---

## 2. Dashboard por Rol

### P-04 | Dashboard Docente
- **URL:** `/dashboard/docente/`
- **Rol:** N2 - Docente
- **Widgets:**
  - **Contador alertas:** Mis estudiantes con alertas activas, agrupadas por módulo
  - **Lista estudiantes:** Mi grupo actual con semáforo (verde/amarillo/rojo por confirmación vencida)
  - **Banner de confirmación:** Visible si `last_verified_at > 15 días`. Oculta detalles de alertas hasta confirmar.
  - **Alertas escaladas:** Casos donde soy responsable que fueron escalados por inactividad
  - **Notificaciones recientes:** Últimas 5 notificaciones
- **Acciones rápidas:** Buscar estudiante, Ver alerta, Registrar bitácora

### P-05 | Dashboard Coordinador
- **URL:** `/dashboard/coordinador/`
- **Rol:** N3 - Coordinador de Convivencia
- **Widgets:**
  - **Panel alertas críticas:** Todas las alertas MOD-01 y MOD-03 de mi sede, ordenadas por antigüedad
  - **Alertas sin validar:** Checklist incompleto o pendiente visto bueno
  - **Escalamientos activos:** Casos que escalaron a mi nivel
  - **Vínculos padres pendientes:** OTP generados sin usar (72h)
  - **Tendencia mensual:** Gráfico de alertas por módulo
- **Acciones rápidas:** Aprobar alerta, Revocar vínculo padre, Ver historial 360

### P-06 | Dashboard Rector
- **URL:** `/dashboard/rector/`
- **Rol:** N4 - Rector/Admin de Institución
- **Widgets:**
  - **Métricas globales sede:** Total alertas activas, cerradas, tasa de cierre
  - **Riesgo académico:** Estudiantes con RF-03 detectados, separados por indicador
  - **Alertas escaladas:** Casos que llegaron a mi nivel (inacción Docente + Coordinador)
  - **Carga masiva:** Acceso rápido a carga CSV
  - **Ruptura de ruta:** Hand-offs sin receptor en 48h (RB-38)
- **Acciones rápidas:** Exportar reportes, Ver historial 360, Asignar Coordinador

### P-07 | Dashboard Admin TI
- **URL:** `/dashboard/admin/`
- **Rol:** N5 - Administrador de Sistema
- **Widgets:**
  - **Usuarios por sede:** Listado con roles y último login
  - **Logs de auditoría:** Últimos eventos del sistema
  - **Estado técnicos:** Conexión Supabase, Redis, Celery
  - **Revocaciones pendientes:** Vínculos desactivados sin resolver
- **Acciones rápidas:** Crear usuario, Ver log, Reiniciar cola

---

## 3. Módulo de Registro de Alertas (MOD-01 a MOD-06)

### P-08 | Listado de Estudiantes
- **URL:** `/estudiantes/`
- **Roles:** N2 Docente, N3 Coordinador
- **Función:** Ver todos los estudiantes asignados/visibles. Filtros: módulo, estado, riesgo académico.
- **Columnas:** Nombre, Código, Semáforo confirmación, Alertas activas, Riesgo académico
- **Acciones:** Click en fila → Perfil estudiante (P-11)

### P-09 | Perfil Detalle del Estudiante
- **URL:** `/estudiantes/{id}/`
- **Roles:** N2 Docente (solo sus estudiantes), N3 Coordinador (todos de su sede), N4 Rector (todos)
- **Función:** Ver datos del estudiante + todas sus alertas y seguimientos
- **Secciones:**
  - Datos básicos (nombre, código, curso, contacto acudiente)
  - Historial de alertas (todas, agrupadas por módulo)
  - Estado de riesgo académico (RF-03)
  - Bitácora de seguimientos (HU-09)
  - Documentos/evidencias adjuntas
- **Restricción:** Si docente no confirmó en 15 días, oculta detalles de alertas (solo muestra módulo y estado)
- **Acciones:** Registrar alerta, Añadir seguimiento, Exportar historial, Hand-off

### P-10 | Registrar Nueva Alerta
- **URL:** `/estudiantes/{id}/alertas/nueva/`
- **Roles:** N2 Docente (solo sus estudiantes), N3 Coordinador
- **Función:** Crear alerta SED con checklist obligatorio (RF-10)
- **Campos obligatorios:**
  - Módulo (MOD-01 a MOD-06) — selector
  - Descripción de hechos (tiempo/modo/lugar) — textarea, 2000 chars
  - Estado del NNA al momento — selector: Estable / Alterado / En riesgo inminente
  - Fecha y hora del incidente — datetime picker
- **Campos opcionales:**
  - Testigos / Evidencias — textarea
  - Registro de acciones — textarea (entidad notificada, fecha, forma)
- **Validación:** Botón "Guardar" deshabilitado hasta completar campos obligatorios
- **Post-guardado:** Notificación inmediata al Coordinador (< 2 min). Triage automático asigna criticidad.

### P-11 | Triage Automático de Criticidad
- **Implementación:** Backend (no pantalla separada)
- **Lógica:**
  - MOD-01, MOD-03 → Crítico (indicador rojo)
  - MOD-02, MOD-04 → Alto (indicador naranja)
  - MOD-05, MOD-06 → Medio (indicador amarillo)
- **Visible en:** Panel del Coordinador como color del badge de la alerta

### P-12 | Ver Detalle de Alerta
- **URL:** `/alertas/{id}/`
- **Roles:** N2 Docente (si es su estudiante), N3 Coordinador, N4 Rector
- **Función:** Ver estado completo de una alerta con toda su información
- **Secciones:**
  - Datos de registro (creado por, fecha, módulo, criticidad)
  - Checklist SED completado
  - Historial de seguimientos (bitácora, inmutable)
  - Estado actual (Activa / En Seguimiento / Validada / Cerrada)
  - Visto bueno del Coordinador (si aplica)
- **Acciones:** Añadir seguimiento, Aprobar para despacho (solo Coordinador)

### P-13 | Visto Bueno del Coordinador
- **URL:** `/alertas/{id}/aprobar/`
- **Rol:** N3 Coordinador
- **Función:** Firmar digitalmente la validación de una alerta para despacho externo (RF-10)
- **Flujo:**
  1. Ver checklist completo
  2. Seleccionar "Aprobar para despacho"
  3. Confirmar con PIN institucional
  4. Sistema registra: hash del contenido, quién aprobó, timestamp
- **Restricción:** No permite aprobar si checklist incompleto

### P-14 | Registro de Seguimiento (Bitácora)
- **URL:** `/alertas/{id}/seguimiento/nuevo/`
- **Roles:** N2 Docente, N3 Coordinador
- **Función:** Añadir entrada a la bitácora de seguimiento
- **Campos:**
  - Tipo de intervención — selector: Llamada telefónica / Reunión presencial / Citación al acudiente / Remisión a orientación / Seguimiento académico / Otro
  - Observaciones — textarea (obligatorio si tipo "Otro")
- **Regla:** Timestamp generado por servidor, no por cliente. Registro inmutable — no se puede editar ni eliminar.
- **Post-guardado:** Cambia estado de alerta a "En Seguimiento" y reinicia contador de escalamiento.

### P-15 | Traspaso de Responsabilidad (Hand-off)
- **URL:** `/estudiantes/{id}/hand-off/`
- **Roles:** N2 Docente, N3 Coordinador
- **Función:** Transferir caso con alertas activas a otro docente o coordinador
- **Flujo:**
  1. Seleccionar estudiante con alertas activas
  2. Elegir nuevo responsable (docente de la misma sede o coordinador)
  3. Registrar motivo de traspaso (textarea, obligatorio)
  4. Enviar solicitud — receptor debe aceptar explícitamente
  5. Al aceptar: se cierra vínculo anterior, nuevo responsable tiene acceso
- **Bloqueo:** Si estudiante tiene alertas activas, no permite desvinculación manual sin Hand-off (RB-37)
- **Alerta RB-38:** Si nadie acepta en 48h, genera notificación a Rector

---

## 4. Confirmación de Listas (RF-11)

### P-16 | Pantalla de Confirmación de Grupo
- **URL:** `/confirmacion/grupo/`
- **Rol:** N2 Docente
- **Función:** Verificar vigencia de la lista de estudiantes asignados cada 15 días
- **Disparo:** Aparece automáticamente cuando `last_verified_at > 15 días`
- **Vista:**
  - Lista completa de estudiantes asignados
  - Semáforo por estudiante: Verde (vigente) / Amarillo (novedad delta-CSV) / Rojo (confirmación vencida)
  - Para cada estudiante:
    - Nombre y código
    - Si tiene alertas activas (módulo + estado)
    - Botón "Retirar de mi grupo" (solo si no tiene alertas activas)
- **Acciones:**
  - "Confirmar — todos vigentes": Actualiza `last_verified_at`, restaura acceso completo
  - "Confirmar y retirar": Confirma lista + desvincula estudiantes sin alertas
- **Resultado:**
  - Confirmación exitosa → Banner desaparece, indicadores vuelven a verde
  - Desvinculación → RLS bloquea acceso al historial de ese estudiante
  - Si estudiante tenía alertas activas: muestra mensaje de bloqueo, no permite desvincular (RB-37)

### P-17 | Gestión de Delta-CSV (Admin)
- **URL:** `/admin/carga-delta/`
- **Rol:** N4 Rector / Admin Institución
- **Función:** Cargar novedades de matrícula (traslados, retiros, nuevas asignaciones)
- **Campos:** Archivo CSV/XLSX (máx 10 MB)
- **Procesamiento:**
  - Valida estructura antes de importar
  - Si hay errores: rechaza toda la carga, genera reporte descargable
  - Si es exitoso: actualiza relaciones TeacherCourse y marca estudiantes como "novedad" (semáforo amarillo)
- **Notificación:** Si un estudiante retirado tiene alertas activas, alerta al Coordinador

---

## 5. Historial 360

### P-18 | Historial Consolidado del Estudiante
- **URL:** `/estudiantes/{id}/historial/`
- **Roles:** N3 Coordinador (completo), N4 Rector (completo), N2 Docente (restringido a sus estudiantes), N1 Padre (solo estado actual y notificaciones recibidas)
- **Función:** Ver toda la información del estudiante en una sola vista
- **Secciones:**
  - Datos del estudiante
  - Todas las alertas (activas y cerradas) ordenadas cronológicamente
  - Todos los seguimientos
  - Estado de riesgo académico
- **Exportar a PDF:** Genera documento con membrete institucional, sin ID completo del estudiante (RB-31)
- **Acceso:** Cada consulta genera log de auditoría (RB-29)

---

## 6. Notificaciones y Alertas

### P-19 | Centro de Notificaciones
- **URL:** `/notificaciones/`
- **Roles:** Todos
- **Función:** Ver todas las notificaciones del usuario, con filtros por estado (leídas/no leídas) y tipo
- **Canales:** Email + in-app
- **Tipos:** Nueva alerta creada, Caso escalado, Confirmación requerida, Hand-off pendiente, Revocación de acceso

### P-20 | Gestión de Vínculos Padres
- **URL:** `/admin/vinculos-padres/`
- **Rol:** N3 Coordinador
- **Función:** Ver vínculos activos, generar OTP, revocar acceso
- **Acciones:**
  - Generar OTP para nuevo acudiente
  - Ver estado de OTP (disponible/usado/vencido)
  - Revocar vínculo existente (requiere razón, mínimo 20 caracteres)
- **Restricción:** Revocación inmediata, sesión del padre invalidada en < 60 segundos (RB-36)

---

## 7. Resumen de Pantallas

| ID | Pantalla | Roles | Módulo RF |
| :--- | :--- | :--- | :--- |
| P-01 | Login | Todos | RF-01 |
| P-02 | Logout | Todos | RF-01 |
| P-03 | Recuperar contraseña | Ninguno | RF-01 |
| P-04 | Dashboard Docente | N2 | RF-08 |
| P-05 | Dashboard Coordinador | N3 | RF-08 |
| P-06 | Dashboard Rector | N4 | RF-08 |
| P-07 | Dashboard Admin TI | N5 | RF-01 |
| P-08 | Listado estudiantes | N2, N3 | RF-02 |
| P-09 | Perfil estudiante | N2, N3, N4 | RF-08 |
| P-10 | Registrar alerta | N2, N3 | RF-02, RF-10 |
| P-11 | Triage automático | Sistema | RF-10 |
| P-12 | Ver detalle alerta | N2, N3, N4 | RF-02 |
| P-13 | Visto bueno coordinador | N3 | RF-10 |
| P-14 | Registro de seguimiento | N2, N3 | RF-05 |
| P-15 | Traspaso Hand-off | N2, N3 | RF-09 |
| P-16 | Confirmación de grupo | N2 | RF-11 |
| P-17 | Carga Delta-CSV | N4 | RF-06 |
| P-18 | Historial 360 | N2, N3, N4, N1 | RF-08 |
| P-19 | Centro notificaciones | Todos | RF-04 |
| P-20 | Gestión vínculos padres | N3 | RF-09 |

**Total: 20 pantallas**

---

## 8. Matriz de Permisos

| Pantalla | N1 Estudiante | N2 Docente | N3 Coordinador | N4 Rector | N5 Admin TI |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Login/Logout | ✅ | ✅ | ✅ | ✅ | ✅ |
| Dashboard Docente | — | ✅ | — | — | — |
| Dashboard Coordinador | — | — | ✅ | — | — |
| Dashboard Rector | — | — | — | ✅ | — |
| Dashboard Admin | — | — | — | — | ✅ |
| Listado estudiantes | R | R (sus) | R | R | — |
| Registrar alerta | — | R (sus) | ✅ | — | — |
| Ver detalle alerta | — | R (sus) | ✅ | ✅ | — |
| Visto bueno | — | — | ✅ | — | — |
| Registro seguimiento | — | R | ✅ | R | — |
| Confirmación grupo | — | ✅ | — | — | — |
| Historial 360 | R (restringido) | R (sus) | ✅ | ✅ | — |
| Centro notificaciones | ✅ | ✅ | ✅ | ✅ | ✅ |
| Gestión vínculos | — | — | ✅ | R | ✅ |
| Carga Delta-CSV | — | — | — | ✅ | — |

**Leyenda:** ✅ = Crear/Editar | R = Solo lectura | — = Sin acceso

---

*Documento preparado por el equipo Alerta Temprana 360.*  
*Versión 1.0 — 20 pantallas documentadas para fase de prototipado.*