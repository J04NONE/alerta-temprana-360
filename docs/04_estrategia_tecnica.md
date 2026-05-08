# 04 — Estrategia Técnica y Elasticidad: Alerta Temprana 360

Este documento aborda los desafíos estratégicos de elicitación y la arquitectura de nube necesaria para soportar la demanda proyectada de 10,000 usuarios concurrentes.

## 1. Descubrimiento de Requerimientos "Emergentes"

Para identificar necesidades que el cliente aún no visualiza (aquellas que surgen con el escalado), implementaremos las siguientes técnicas:

* **Prototipado Evolutivo:** Entregas semanales de prototipos funcionales de alta fidelidad. Al interactuar con el dashboard, los coordinadores descubrirán la necesidad de filtros complejos o reportes de tendencias que no se ven en una entrevista inicial.
* **Análisis de Escenarios de Excepción:** Talleres de "qué pasa si...". Ejemplo: "¿Qué pasa si un docente reporta un accidente un domingo a las 11 PM?". Esto revela requerimientos de disponibilidad y flujos de notificación fuera de horario.
* **Historias de Usuario "Shadow":** Observación directa de cómo se manejan hoy las alertas en papel o Excel para detectar fricciones que el usuario ha normalizado y no menciona como requisito.

## 2. Especificación de Elasticidad (Requirement 10k)

La elasticidad es la capacidad del sistema para adaptarse a la carga en tiempo real. Se especifica como un requisito verificable mediante las siguientes métricas de Cloud Computing:

### Métricas de Verificación

| Métrica | Objetivo / Umbral | Herramienta de Medición |
| :--- | :--- | :--- |
| **Tiempo de Respuesta (p99)** | < 800ms durante picos de 10k usuarios. | Locust / JMeter |
| **Tasa de Error** | < 0.1% durante procesos de Auto-scaling. | Sentry / CloudWatch |
| **Latencia de Escalado** | Nuevas instancias de Django deben estar activas en < 60 segundos tras superar el 70% de CPU. | AWS CloudWatch / Azure Monitor |
| **Throughput** | El sistema debe procesar al menos 500 transacciones por segundo (TPS). | Locust |

### Arquitectura para Elasticidad

1. **Capa de Backend (Django):** Despliegue en contenedores (Docker) gestionados por un clúster (Kubernetes o AWS ECS) con reglas de *Auto-Scaling Group* basadas en uso de CPU y RAM.
2. **Capa de Datos (Supabase/PostgreSQL):** Uso de *Read Replicas* para distribuir la carga de consultas de los dashboards de directivos, manteniendo la instancia primaria para escrituras de alertas.
3. **Capa de Aplicación:** Implementación de colas de tareas (Celery + Redis) para que el envío de 10,000 notificaciones simultáneas no bloquee el hilo principal de la aplicación.

## 3. Estrategia de Prueba de Carga

Para asegurar que el sistema no colapse si 10,000 padres acceden al mismo tiempo:

* Se realizará una prueba de carga (*Stress Test*) usando **Locust**, simulando el flujo completo: Login -> Consulta de Alerta -> Carga de Seguimiento.
* Se validará que el balanceador de carga distribuya el tráfico equitativamente y que el sistema "se encoja" automáticamente cuando la carga baje, optimizando costos.

---

## 4. Arquitectura Hexagonal (Ports & Adapters)

AT360 organiza su código en tres capas concéntricas para garantizar que la lógica
de negocio no dependa de frameworks, bases de datos ni canales de entrega.

### 4.1 Capas

| Capa | Contenido | Regla |
| --- | --- | --- |
| **Dominio** | Entidades, Value Objects, Servicios de Dominio | Sin imports de Django ni de Supabase |
| **Puertos** | Interfaces (ABCs) para repos y notificadores | Sin implementaciones concretas |
| **Adaptadores** | DRF Views, Supabase repos, Email/Push senders | Dependen hacia adentro, nunca al revés |

### 4.2 Estructura de Directorios

```text
at360/
├── domain/
│   ├── models/          # StudentRisk, Alert, RiskScore (dataclasses puras)
│   └── services/        # RiskCalculationService, AlertTriggerService
├── ports/
│   ├── repositories.py  # IStudentRepository, IAlertRepository (ABC)
│   └── notifiers.py     # INotifier (ABC)
└── adapters/
    ├── api/             # Django REST Framework views y serializers
    ├── db/              # Implementaciones Supabase/PostgreSQL de los repos
    └── notifications/   # Email, Push, SMS adapters
```

### 4.3 Beneficios para AT360

* **Testeabilidad:** Los servicios de dominio se prueban con mocks de los puertos — sin base de datos.
* **Intercambiabilidad:** Si Supabase se reemplaza por otro proveedor, solo cambia `adapters/db/`.
* **Cumplimiento de privacidad:** Aísla la lógica de datos de menores de la infraestructura concreta.

### 4.4 Flujo de Creación de Alerta

```text
HTTP POST /api/alerts/
    → DRF Adapter (deserializa, valida)
    → AlertTriggerService (dominio, aplica reglas de negocio)
    → IAlertRepository.save() (puerto)
    → SupabaseAlertRepository.save() (adaptador DB)
    → INotifier.send() (puerto)
    → PushNotificationAdapter.send() (adaptador notificaciones)
```

---

## 5. CQRS (Command Query Responsibility Segregation)

AT360 separa las operaciones de **escritura** (Commands) de las de **lectura** (Queries)
para cumplir el requisito de 10,000 usuarios concurrentes sin degradar la latencia.

### 5.1 Motivación

Los dashboards de directivos y coordinadores generan un volumen de lecturas 20×
superior al de escrituras (alertas nuevas, actualizaciones de seguimiento).
Mezclar ambos en el mismo modelo de datos crea contención en la BD primaria.

### 5.2 Mapa Commands / Queries

| Tipo | Nombre | Descripción | Destino BD |
| --- | --- | --- | --- |
| **Command** | `CreateAlertCommand` | Registra una nueva alerta de riesgo | Instancia primaria (escritura) |
| **Command** | `UpdateFollowUpCommand` | Actualiza el seguimiento de un caso | Instancia primaria |
| **Command** | `SendNotificationCommand` | Encola notificación (Celery) | Redis / primaria |
| **Query** | `GetStudentRiskQuery` | Dashboard individual del estudiante | Read Replica |
| **Query** | `GetDashboardStatsQuery` | KPIs para directivos (10k usuarios) | Read Replica |
| **Query** | `GetAlertHistoryQuery` | Histórico de alertas por periodo | Read Replica |

### 5.3 Integración con la Arquitectura de Elasticidad (§2)

```text
WRITES (Commands)                    READS (Queries)
─────────────────                    ──────────────────────
DRF POST/PUT/PATCH                   DRF GET
       │                                     │
CommandBus                           QueryBus
       │                                     │
Domain Services                      Read Models (optimizados)
       │                                     │
PostgreSQL PRIMARY ─── replicación → PostgreSQL READ REPLICA
(Supabase write endpoint)            (Supabase read endpoint)
```

### 5.4 Django 6.0 + CQRS + CSP

El middleware CSP de Django 6.0 (`ContentSecurityPolicyMiddleware`) se aplica en la capa
de adaptadores (DRF), aguas arriba del Command/Query Bus, garantizando que **todas** las
respuestas HTTP lleven la cabecera `Content-Security-Policy` independientemente de si
la operación es una lectura o una escritura.

### 5.5 Consideraciones de Implementación

* Los **Command Handlers** retornan solo IDs o confirmaciones — nunca entidades completas.
* Los **Query Handlers** pueden usar SQL directo (`django.db.connection.cursor`) sobre
  la Read Replica para consultas analíticas que no encajan limpiamente en el ORM.
* Celery gestiona los Commands asíncronos (`SendNotificationCommand`) para no bloquear
  el hilo HTTP bajo picos de carga (ver §2 — Celery + Redis).
