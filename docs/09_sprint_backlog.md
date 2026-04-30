# 09 — Sprint Backlog: Sprint 1 — Alerta Temprana 360

**Sprint:** 1 de 3  
**Duración:** 1 semana (contexto académico)  
**Fecha inicio:** [A definir con el profesor]  
**Fecha cierre:** [Fecha inicio + 7 días]  
**Scrum Master:** Michael  
**Equipo:** Michael · Iván · Andrés  
**Versión:** 1.0 · **Fecha doc:** 2026-04-30

---

## Sprint Goal

> **"Al final de este Sprint, el sistema permite que un Administrador registre usuarios con roles diferenciados, un Padre se vincule mediante token OTP, un Docente reporte una alerta en cualquiera de los 6 módulos SED, y un Coordinador visualice el historial consolidado del estudiante."**

Este goal cubre los dos pilares de valor confirmados por el análisis de la asignatura:
1. **Cumplimiento Ley 1581** — vinculación segura del padre sin almacenar datos sensibles (HU-02)
2. **Valor nuclear del sistema** — el primer reporte de alerta con notificación real al Coordinador (HU-05)

---

## Definition of Done (DoD) — Sprint 1

Una tarea está "terminada" cuando cumple **todos** los criterios siguientes:

- [ ] El código está en el repositorio en la rama `feature/sprint-1`
- [ ] Los criterios de aceptación de la HU pasan manualmente en el entorno local
- [ ] El Docente/Coordinador/Padre correspondiente puede ejecutar el flujo completo sin errores visibles
- [ ] Los campos sensibles (datos del estudiante, contenido de alertas) no son accesibles desde roles no autorizados (prueba manual de RLS)
- [ ] El log de auditoría registra el evento correspondiente en la base de datos
- [ ] Andrés ha revisado la pantalla en modo desktop y mobile
- [ ] Michael ha documentado cualquier cambio en los requisitos surgido durante la implementación

---

## Capacidad del Equipo — Sprint 1

| Integrante | Rol | Horas disponibles | Foco principal |
| :--- | :--- | :--- | :--- |
| **Michael** | SM / Documentador | 8h | Coordinación, pruebas de aceptación, documentación de cambios |
| **Iván** | Analista / Backend | 8h | Django: modelos, APIs, lógica de negocio, RLS |
| **Andrés** | Diseñador / Frontend | 8h | HTML/CSS/JS: vistas, formularios, conexión a API |
| **Total** | | **24h** | |

---

## Ítems del Sprint 1

> Seleccionados del Product Backlog (`08_product_backlog.md`). Total: **39 SP** (comprometidos para entrega académica).

---

### Infraestructura base — Prerrequisito de todo lo demás

| ID Tarea | Descripción | Responsable | Estimación | Estado |
| :--- | :--- | :--- | :--- | :--- |
| T-00.1 | Configurar proyecto Django 5.x con estructura de apps: `usuarios`, `alertas`, `seguimiento`, `datos` | Iván | 1h | ⬜ Por hacer |
| T-00.2 | Conectar Django a Supabase/PostgreSQL (`.env` con DATABASE_URL) | Iván | 0.5h | ⬜ Por hacer |
| T-00.3 | Configurar políticas RLS en Supabase: aislamiento por `sede_id`, `grupo_id`, `caso_id` | Iván | 1.5h | ⬜ Por hacer |
| T-00.4 | Configurar JWT con django-rest-framework-simplejwt + tabla de token blacklist | Iván | 1h | ⬜ Por hacer |
| T-00.5 | Configurar Celery + Redis para colas asíncronas de notificaciones | Iván | 1h | ⬜ Por hacer |
| T-00.6 | Crear plantilla base HTML (layout, navbar por rol, sistema de alertas visuales) | Andrés | 1h | ⬜ Por hacer |

**Subtotal infraestructura:** ~6h

---

### PB-17/18/19 — Infraestructura RBAC + JWT + Celery (5+3+5 SP)

> Ver tareas T-00.3, T-00.4, T-00.5 arriba.

---

### PB-01 — HU-01: Registro de Docente (3 SP)

| ID Tarea | Descripción | Responsable | Estimación | Estado |
| :--- | :--- | :--- | :--- | :--- |
| T-01.1 | Modelo `Usuario` con campos: correo, rol, sede_id, nivel_acceso | Iván | 0.5h | ⬜ Por hacer |
| T-01.2 | Endpoint `POST /api/usuarios/` con validación de rol y sede | Iván | 0.5h | ⬜ Por hacer |
| T-01.3 | Vista de registro de usuario para Admin TI | Andrés | 0.5h | ⬜ Por hacer |
| T-01.4 | Prueba de aceptación: registro exitoso + rechazo de correo duplicado | Michael | 0.5h | ⬜ Por hacer |

---

### PB-02 — HU-02: Vinculación Padre OTP (5 SP) ⭐ Prioritario Ley 1581

| ID Tarea | Descripción | Responsable | Estimación | Estado |
| :--- | :--- | :--- | :--- | :--- |
| T-02.1 | Modelo `OTPToken` con campos: token_hash, estudiante_id, valid_until (72h), usado | Iván | 0.5h | ⬜ Por hacer |
| T-02.2 | Endpoint `POST /api/vinculos/generar-otp/` (solo Admin Institución) | Iván | 0.5h | ⬜ Por hacer |
| T-02.3 | Endpoint `POST /api/auth/registro-padre/` — valida OTP, crea `ParentStudent`, marca OTP usado | Iván | 1h | ⬜ Por hacer |
| T-02.4 | Lógica de rechazo: OTP vencido (>72h) · OTP ya utilizado · OTP inexistente | Iván | 0.5h | ⬜ Por hacer |
| T-02.5 | Vista de registro para Padre con campo OTP y mensajes de error claros | Andrés | 0.5h | ⬜ Por hacer |
| T-02.6 | Prueba de aceptación: flujo completo OTP + 3 escenarios de rechazo | Michael | 0.5h | ⬜ Por hacer |

---

### PB-03 — HU-04: Revocación de Vínculo Padre (3 SP)

| ID Tarea | Descripción | Responsable | Estimación | Estado |
| :--- | :--- | :--- | :--- | :--- |
| T-03.1 | Endpoint `DELETE /api/vinculos/{id}/` — requiere campo razón (min 20 chars) | Iván | 0.5h | ⬜ Por hacer |
| T-03.2 | Invalidar JWT activo del Padre: INSERT en `token_blacklist` | Iván | 0.5h | ⬜ Por hacer |
| T-03.3 | Vista de revocación para Coordinador con campo de razón obligatorio | Andrés | 0.5h | ⬜ Por hacer |
| T-03.4 | Prueba de aceptación: revocación exitosa + acceso denegado al Padre + log auditado | Michael | 0.5h | ⬜ Por hacer |

---

### PB-06 — HU-05: Registro de Alerta SED (5 SP) ⭐ Valor Nuclear del Sistema

| ID Tarea | Descripción | Responsable | Estimación | Estado |
| :--- | :--- | :--- | :--- | :--- |
| T-05.1 | Modelo `Alerta` con campos: modulo, estudiante_id, docente_id, descripcion, estado, nivel_riesgo, timestamp | Iván | 0.5h | ⬜ Por hacer |
| T-05.2 | Endpoint `POST /api/alertas/` con validación RLS (estudiante debe pertenecer al Docente) | Iván | 1h | ⬜ Por hacer |
| T-05.3 | Lógica de triage automático: MOD-01/MOD-03 → `Crítico`, MOD-02/MOD-04 → `Alto`, MOD-05/MOD-06 → `Medio` | Iván | 0.5h | ⬜ Por hacer |
| T-05.4 | Tarea Celery: envío de notificación urgente al Coordinador (email + log) | Iván | 0.5h | ⬜ Por hacer |
| T-05.5 | Vista formulario de nueva alerta: selector de módulo + campo descripción + validación | Andrés | 1h | ⬜ Por hacer |
| T-05.6 | Prueba de aceptación: alerta creada + Coordinador notificado + rechazo de estudiante ajeno | Michael | 0.5h | ⬜ Por hacer |

---

### PB-07 — HU-09: Bitácora de Seguimiento (3 SP)

| ID Tarea | Descripción | Responsable | Estimación | Estado |
| :--- | :--- | :--- | :--- | :--- |
| T-09.1 | Modelo `Seguimiento` con campos: alerta_id, actor_id, tipo_intervencion, observaciones, timestamp (servidor) | Iván | 0.5h | ⬜ Por hacer |
| T-09.2 | Endpoint `POST /api/alertas/{id}/seguimientos/` — solo INSERT, sin UPDATE/DELETE | Iván | 0.5h | ⬜ Por hacer |
| T-09.3 | Vista de historial de seguimientos dentro del perfil de alerta | Andrés | 0.5h | ⬜ Por hacer |
| T-09.4 | Prueba de aceptación: registro exitoso + rechazo de edición de entrada existente | Michael | 0.5h | ⬜ Por hacer |

---

### PB-08 — HU-10: Escalamiento Automático (5 SP)

| ID Tarea | Descripción | Responsable | Estimación | Estado |
| :--- | :--- | :--- | :--- | :--- |
| T-10.1 | Tarea Celery Beat: verificación cada 30min de alertas sin seguimiento | Iván | 1h | ⬜ Por hacer |
| T-10.2 | Lógica de escalamiento por nivel: Crítico=24h, Alto=48h, Medio=72h | Iván | 0.5h | ⬜ Por hacer |
| T-10.3 | Envío de notificación de escalamiento al Coordinador + aviso al Docente | Iván | 0.5h | ⬜ Por hacer |
| T-10.4 | Prueba de aceptación: alerta sin seguimiento escala en tiempo correcto | Michael | 0.5h | ⬜ Por hacer |

---

### PB-12 — HU-12: Carga Masiva CSV (5 SP)

| ID Tarea | Descripción | Responsable | Estimación | Estado |
| :--- | :--- | :--- | :--- | :--- |
| T-12.1 | Endpoint `POST /api/datos/carga-masiva/` con parseo de CSV/XLSX (librería pandas u openpyxl) | Iván | 1h | ⬜ Por hacer |
| T-12.2 | Validación por fila: campos obligatorios, formatos de fecha, códigos duplicados | Iván | 0.5h | ⬜ Por hacer |
| T-12.3 | Transacción atómica: todo o nada — si hay error, rollback completo | Iván | 0.5h | ⬜ Por hacer |
| T-12.4 | Vista de carga con progreso y reporte de errores descargable | Andrés | 0.5h | ⬜ Por hacer |
| T-12.5 | Activación automática del motor de detección RF-03 al completar la carga | Iván | 0.5h | ⬜ Por hacer |

---

### PB-14 — HU-14: Historial 360 (5 SP)

| ID Tarea | Descripción | Responsable | Estimación | Estado |
| :--- | :--- | :--- | :--- | :--- |
| T-14.1 | Endpoint `GET /api/estudiantes/{id}/historial/` con filtro RBAC por nivel de acceso | Iván | 1h | ⬜ Por hacer |
| T-14.2 | Vista de historial 360 para Coordinador: alertas + seguimientos cronológicos | Andrés | 1h | ⬜ Por hacer |
| T-14.3 | Vista restringida para Padre: solo estado actual + notificaciones propias | Andrés | 0.5h | ⬜ Por hacer |
| T-14.4 | Registro en audit_log de cada acceso al historial | Iván | 0.5h | ⬜ Por hacer |

---

## Resumen de carga — Sprint 1

| Integrante | Tareas asignadas | Horas estimadas |
| :--- | :--- | :--- |
| **Iván** (backend) | T-00.1 a T-00.5, T-01.1/2, T-02.1/2/3/4, T-03.1/2, T-05.1/2/3/4, T-09.1/2, T-10.1/2/3, T-12.1/2/3/5, T-14.1/4 | ~15h |
| **Andrés** (frontend) | T-00.6, T-01.3, T-02.5, T-03.3, T-05.5, T-09.3, T-12.4, T-14.2/3 | ~6.5h |
| **Michael** (SM + QA) | T-01.4, T-02.6, T-03.4, T-05.6, T-09.4, T-10.4 + documentación de cambios | ~5h |
| **Total** | | **~26.5h** |

> La capacidad nominal es 24h. El margen de 2.5h adicionales es manejable para un sprint académico si se trabaja de forma coordinada.

---

## Riesgos identificados para Sprint 1

| Riesgo | Impacto | Mitigación |
| :--- | :--- | :--- |
| Configuración de RLS en Supabase es más compleja de lo estimado | Alto | Dedicar la primera sesión de trabajo exclusivamente a T-00.3; si bloquea, simplificar a validación solo en Django |
| Celery + Redis no están instalados en el entorno local | Medio | Usar un worker en modo eager (sin Redis) para las pruebas locales del Sprint 1 |
| El Sprint tiene demasiados SP para el tiempo disponible | Alto | Negociar con el profesor que el Sprint 1 académico cubre los Must Have de RF-01, RF-02 y RF-05 como demostración |

---

## Notas del Scrum Master (Michael)

1. La **prioridad absoluta del Sprint** es HU-02 (OTP) y HU-05 (primera alerta). Si hay bloqueos, esas dos HUs no se mueven.
2. La **demostración al profesor** (Stakeholder) debe mostrar el flujo completo: Docente registra alerta → Sistema notifica → Coordinador ve en dashboard.
3. Cualquier cambio en los requisitos durante el Sprint se documenta en `findings.md` antes de modificar el código.
4. Al finalizar el Sprint, Michael actualiza `progress.md` con el Velocity real para ajustar la capacidad del Sprint 2.

---

*Documento preparado por Michael (Scrum Master) — Alerta Temprana 360.*  
*Versión 1.0 — Se actualiza al cierre del Sprint con resultados reales vs. planificados.*
