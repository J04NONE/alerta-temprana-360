# 08 — Product Backlog: Alerta Temprana 360

**Metodología:** SCRUM · **Priorización:** MoSCoW  
**Versión:** 1.0 · **Fecha:** 2026-04-30  
**Trazabilidad:** Cada ítem referencia su HU en `06_historias_usuario.md` y su RF en `03_matriz_requisitos.md`.

---

## Criterio de Priorización MoSCoW

| Nivel | Criterio de inclusión |
| :--- | :--- |
| **Must Have** | Sin esta funcionalidad el sistema no cumple su propósito ni la Ley 1581/2012. Bloquea los demás ítems. |
| **Should Have** | Alto valor. Resuelve el problema de Privilegios Residuales y mejora la calidad del reporte SED. Entra al Sprint 2. |
| **Could Have** | Mejora la experiencia pero no es crítico para el MVP académico. Entra al Sprint 3+ o queda como backlog. |
| **Won't Have (MVP)** | Fuera del alcance del semestre académico. Documentado como deuda técnica futura. |

---

## Épica 1 — Seguridad y Gestión de Acceso

| ID | Historia de Usuario | MoSCoW | SP | Sprint | RF | Dependencias |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| PB-01 | **HU-01** Registro de Docente por el Administrador | Must Have | 3 | Sprint 1 | RF-01 | Ninguna |
| PB-02 | **HU-02** Vinculación de Padre de Familia mediante OTP | Must Have | 5 | Sprint 1 | RF-01 | PB-01 |
| PB-03 | **HU-04** Revocación del acceso de Padre de Familia | Must Have | 3 | Sprint 1 | RF-09 | PB-02 |
| PB-04 | **HU-03** Confirmación Activa de Vigencia de Grupo | Should Have | 5 | Sprint 2 | RF-11 | PB-01 |
| PB-05 | **HU-11** Traspaso de Responsabilidad (Hand-off) | Should Have | 5 | Sprint 2 | RF-09 | PB-03, PB-04 |

**Total Épica 1:** 21 SP · Must Have: 11 SP · Should Have: 10 SP

---

## Épica 2 — Alertas SED (6 Módulos)

| ID | Historia de Usuario | MoSCoW | SP | Sprint | RF | Dependencias |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| PB-06 | **HU-05** Registro de alerta en módulo SED | Must Have | 5 | Sprint 1 | RF-02 | PB-01 |
| PB-07 | **HU-09** Registrar intervención en bitácora de seguimiento | Must Have | 3 | Sprint 1 | RF-05 | PB-06 |
| PB-08 | **HU-10** Escalamiento automático de alerta no atendida | Must Have | 5 | Sprint 1 | RF-07 | PB-06 |
| PB-09 | **HU-06** Checklist SED obligatorio (tiempo/modo/lugar) | Should Have | 3 | Sprint 2 | RF-10 | PB-06 |
| PB-10 | **HU-07** Triage automático de criticidad | Should Have | 3 | Sprint 2 | RF-10 | PB-09 |
| PB-11 | **HU-08** Visto bueno del Coordinador (firma digital) | Should Have | 5 | Sprint 2 | RF-10 | PB-10 |

**Total Épica 2:** 24 SP · Must Have: 13 SP · Should Have: 11 SP

---

## Épica 3 — Ingesta de Datos y Triage Automático

| ID | Historia de Usuario | MoSCoW | SP | Sprint | RF | Dependencias |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| PB-12 | **HU-12** Carga masiva CSV — Triage Automático (MOD-06) | Must Have | 5 | Sprint 1 | RF-06, RF-03 | PB-01 |
| PB-13 | **HU-13** Delta-CSV incremental (novedades de matrícula) | Should Have | 5 | Sprint 2 | RF-06, RF-11 | PB-12, PB-04 |

**Total Épica 3:** 10 SP · Must Have: 5 SP · Should Have: 5 SP

---

## Épica 4 — Dashboard y Visibilidad

| ID | Historia de Usuario | MoSCoW | SP | Sprint | RF | Dependencias |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| PB-14 | **HU-14** Historial 360 del estudiante (Coordinador) | Must Have | 5 | Sprint 1 | RF-08 | PB-06 |
| PB-15 | Exportar historial a PDF con membrete institucional | Could Have | 3 | Sprint 3 | RF-08 | PB-14 |
| PB-16 | Métricas agregadas por sede (Rector) | Could Have | 5 | Sprint 3 | RF-08 | PB-14 |

**Total Épica 4:** 13 SP · Must Have: 5 SP · Could Have: 8 SP

---

## Épica 5 — Infraestructura Técnica (No visible al usuario)

| ID | Tarea técnica | MoSCoW | SP | Sprint | RNF | Descripción |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| PB-17 | Configuración RBAC + RLS en Supabase | Must Have | 5 | Sprint 1 | RNF-03 | Políticas RLS por sede_id, grupo_id y caso_id |
| PB-18 | Implementación JWT + token blacklist (Redis) | Must Have | 3 | Sprint 1 | RNF-03 | Invalidación instantánea de sesiones |
| PB-19 | Configuración Celery + Redis para colas asíncronas | Must Have | 5 | Sprint 1 | RNF-01, RNF-06 | Motor de notificaciones y escalamiento |
| PB-20 | Canal de respaldo push in-app (resiliencia SMTP) | Should Have | 3 | Sprint 2 | RNF-06 | Fallback si SMTP no disponible |
| PB-21 | Read Replicas en Supabase para dashboards | Could Have | 3 | Sprint 3 | RNF-04 | Distribución de carga de lectura |

**Total Épica 5:** 19 SP · Must Have: 13 SP · Should Have: 3 SP · Could Have: 3 SP

---

## Backlog Futuro — Won't Have (MVP)

| ID | Descripción | Razón de exclusión |
| :--- | :--- | :--- |
| PB-W1 | Integración con Web Services SDS (Secretaría Distrital de Salud) | RE-05: fuera del alcance del MVP académico |
| PB-W2 | App móvil nativa (iOS/Android) | Stack definido como HTML/CSS/JS — no nativo |
| PB-W3 | Módulo de estadísticas predictivas (ML) | Requiere datos históricos que no existen aún |
| PB-W4 | Integración directa con SIAU | RE-04: restricción explícita del sistema |
| PB-W5 | Multi-idioma (inglés/lenguas nativas) | No requerido por la SED para el MVP |

---

## Resumen de capacidad por Sprint

| Sprint | Alcance | SP Comprometidos | Épicas cubiertas |
| :--- | :--- | :--- | :--- |
| **Sprint 1** | MVP funcional: acceso + alertas + seguimiento + datos + dashboard | **39 SP** | 1, 2, 3, 4, 5 (infra) |
| **Sprint 2** | Calidad y compliance: confirmación activa + checklist SED + delta + resiliencia | **26 SP** | 1, 2, 3, 5 |
| **Sprint 3+** | Mejoras de visibilidad y optimización | **11 SP** | 4, 5 |
| **Total MVP** | Sprint 1 + Sprint 2 | **65 SP** | Todas |

> **Nota para el Scrum Master:** La capacidad del equipo académico es aproximadamente 20–25 SP por sprint (3 personas × ~8h de trabajo efectivo por semana). El Sprint 1 de 39 SP es ambicioso para un contexto académico — se recomienda negociar el alcance con el profesor antes del primer sprint planning.

---

## Definición de "Listo para Sprint" (Definition of Ready)

Un ítem del Product Backlog está listo para entrar al Sprint cuando:
- [ ] La HU tiene al menos 2 criterios de aceptación en Gherkin
- [ ] El RF asociado tiene especificación completa en `05_especificacion_requisitos.md`
- [ ] Las dependencias del backlog están identificadas
- [ ] El equipo puede estimar los Story Points con confianza

---

*Documento preparado por Michael (Scrum Master) — Alerta Temprana 360.*  
*Versión 1.0 — Se actualiza en cada Sprint Review con el feedback del profesor (stakeholder).*
