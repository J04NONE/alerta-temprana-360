# Alerta Temprana 360

Sistema de Soporte a Decisiones (DSS) para la detección proactiva de riesgo estudiantil. 

## Stack Tecnológico

| Capa | Tecnología | Versión | Justificación |
| --- | --- | --- | --- |
| **Backend** | Django + DRF | **6.0** | CSP nativo (XSS), Python 3.12+ |
| **Base de Datos** | PostgreSQL | 15+ | Vía Supabase (managed) |
| **Infraestructura** | Cloud-native (PaaS) | — | Auto-scaling, Read Replicas |
| **Estándar de Requisitos** | ISO/IEC/IEEE 29148:2018 | — | Trazabilidad de requisitos |

### Por qué Django 6.0 (no 5.x)

Django 6.0 introduce **Content Security Policy (CSP) nativo** — una cabecera HTTP crítica que
previene ataques XSS e inyección de contenido. En versiones anteriores, CSP requería
la librería de terceros `django-csp` (mantenimiento externo, riesgo de cadena de suministro).

Con Django 6.0 el proyecto puede configurar CSP directamente:

```python
# settings.py
MIDDLEWARE = [
    "django.middleware.csp.ContentSecurityPolicyMiddleware",
    ...
]

from django.utils.csp import CSP

SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE],   # nonce por request, anti-XSS
    "img-src":    [CSP.SELF, "https:"],
}
```

Esto es especialmente relevante para AT360 porque el sistema expone dashboards con datos
de menores de edad, donde una vulnerabilidad XSS tendría impacto legal y de privacidad.

## Arquitectura

AT360 sigue **Arquitectura Hexagonal** (Ports & Adapters) con **CQRS**:

```text
┌─────────────────────────────────────────────┐
│              DOMINIO (Core)                  │
│   StudentRisk · Alert · RiskScore            │
├──────────────┬──────────────────────────────┤
│  Commands    │  Queries                     │
│  (writes)    │  (reads — Read Replicas)     │
├──────────────┴──────────────────────────────┤
│         Ports (interfaces)                   │
│  IAlertRepo · INotifier · IStudentRepo       │
├──────────────────────────────────────────────┤
│  Adapters: DRF · Supabase · Email/Push       │
└──────────────────────────────────────────────┘
```

Ver `docs/04_estrategia_tecnica.md` §4 y §5 para detalle.

## Equipo

- **Michael (Joan):** Scrum Master, Documentador & Backend Developer
- **Iván:** Analista de Requisitos & Backend Lead
- **Andrés:** Diseñador de Solución (UI/UX)

## Estado del Proyecto

Actualmente en fase de **Definición de Arquitectura e Ingeniería de Requisitos** (Django 6.0 · Hexagonal · CQRS).
