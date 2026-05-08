# 07 — Diagramas UML: Alerta Temprana 360

**Herramienta recomendada:** [PlantUML Online](https://www.plantuml.com/plantuml/uml) · Draw.io (importar como PlantUML)  
**Versión:** 1.0 · **Fecha:** 2026-04-30  
**Trazabilidad:** Los casos de uso derivan de las HUs del documento `06_historias_usuario.md`.

---

## 1. Diagrama de Casos de Uso — Sistema Completo

```plantuml
@startuml AT360_CasosDeUso

skinparam actorStyle awesome
skinparam packageStyle rectangle
left to right direction

actor "Docente" as DOC
actor "Coordinador\nde Convivencia" as COORD
actor "Padre de\nFamilia" as PAD
actor "Rector /\nAdministrador" as RECT
actor "Psicólogo /\nOrientador" as PSI
actor "Administrador\nde Sistema (TI)" as TI
actor "Admin\nInstitución" as ADMIN
actor "Sistema\n(automático)" as SIS

rectangle "Alerta Temprana 360" {

  package "Gestión de Acceso" {
    usecase "Registrar usuario\ncon rol asignado" as UC01
    usecase "Vincular Padre\nmediante OTP" as UC02
    usecase "Confirmar vigencia\nde grupo (Semáforo)" as UC03
    usecase "Revocar acceso\nde Padre" as UC04
  }

  package "Alertas SED" {
    usecase "Registrar alerta\nen módulo SED" as UC05
    usecase "Completar checklist\nSED obligatorio" as UC06
    usecase "Triage automático\nde criticidad" as UC07
    usecase "Dar visto bueno\npara despacho externo" as UC08
    usecase "Recibir remisión\nde caso especializado" as UC09
  }

  package "Seguimiento de Casos" {
    usecase "Registrar intervención\nen bitácora" as UC10
    usecase "Escalar alerta\nautomáticamente" as UC11
    usecase "Ejecutar Hand-off\nde caso" as UC12
  }

  package "Ingesta de Datos y Triage Automático" {
    usecase "Cargar CSV completo\nde datos académicos" as UC13
    usecase "Cargar Delta-CSV\n(novedades de matrícula)" as UC14
    usecase "Detectar indicadores\nindirectos (MOD-06)" as UC15
  }

  package "Dashboard y Reportes" {
    usecase "Consultar historial\n360 del estudiante" as UC16
    usecase "Exportar historial\na PDF" as UC17
    usecase "Ver métricas\nagregadas de sede" as UC18
  }

  package "Notificaciones" {
    usecase "Recibir notificación\nde alerta activa" as UC19
    usecase "Recibir notificación\nde escalamiento" as UC20
  }
}

' Asociaciones Docente
DOC --> UC05
DOC --> UC06
DOC --> UC10
DOC --> UC12
DOC --> UC03

' Asociaciones Coordinador
COORD --> UC08
COORD --> UC04
COORD --> UC12
COORD --> UC16
COORD --> UC17

' Asociaciones Padre
PAD --> UC02
PAD --> UC16

' Asociaciones Rector
RECT --> UC16
RECT --> UC18
RECT --> UC17

' Asociaciones Psicólogo
PSI --> UC09
PSI --> UC10

' Asociaciones Admin Institución
ADMIN --> UC13
ADMIN --> UC14

' Asociaciones TI
TI --> UC01

' Asociaciones Sistema (actor automático)
SIS --> UC11
SIS --> UC15

' Include/Extend
UC05 ..> UC06 : <<include>>
UC06 ..> UC07 : <<include>>
UC07 ..> UC08 : <<extend>>
UC08 ..> UC09 : <<extend>>
UC11 ..> UC20 : <<include>>
UC13 ..> UC15 : <<include>>
UC14 ..> UC15 : <<include>>
UC15 ..> UC05 : <<include>>
UC16 ..> UC17 : <<extend>>

' Notificaciones recibidas
DOC <-- UC19
COORD <-- UC19
COORD <-- UC20
PAD <-- UC19
RECT <-- UC20
PSI <-- UC19

@enduml
```

---

## 2. Diagrama de Secuencia 1 — Registro y Escalamiento de Alerta Crítica

**Escenario:** Docente registra una alerta de "Conducta Suicida". El sistema hace triage, notifica al Coordinador y Psicólogo. Si el Coordinador no atiende en 24h, escala al Rector.

```plantuml
@startuml AT360_Seq_AlertaCritica

skinparam sequenceArrowThickness 2
skinparam roundcorner 5

actor "Docente" as DOC
participant "Frontend\n(HTML/JS)" as FE
participant "Django\nBackend" as BE
participant "Celery\nWorker" as CEL
database "PostgreSQL\n(Supabase)" as DB
participant "Cola Redis" as REDIS
actor "Coordinador" as COORD
actor "Psicólogo" as PSI
actor "Rector" as RECT

DOC -> FE : Completa checklist MOD-03\n(Conducta Suicida)
FE -> BE : POST /api/alertas/\n{modulo: "MOD-03", datos: {...}}

BE -> BE : Validar RLS:\n¿estudiante pertenece al docente?
BE -> DB : INSERT alerta\n{estado: "Activa", nivel: "Critico"}
DB --> BE : alerta_id generado

BE -> BE : Triage automático:\nMOD-03 → nivel CRITICO
BE -> REDIS : Encolar notificación urgente\n(prioridad máxima)
BE --> FE : HTTP 201 Created\n{alerta_id, estado: "Activa"}
FE --> DOC : "Alerta registrada.\nCoordinador notificado."

REDIS -> CEL : Procesar notificación urgente
CEL -> COORD : Email + Push:\n"Alerta CRÍTICA — Conducta Suicida\n[Estudiante X] — Ver caso"
CEL -> PSI : Notificación preliminar:\n"Caso referido para evaluación"
CEL -> DB : UPDATE log_notificaciones\n{estado: "enviado"}

note over BE, DB : La alerta queda en "En Revisión"\nhasta visto bueno del Coordinador

... 25 horas después — sin seguimiento del Coordinador ...

CEL -> DB : SELECT alertas WHERE\nestado="Activa" AND\nsin_seguimiento > 24h
DB --> CEL : [alerta_id escalable]
CEL -> DB : UPDATE alerta\n{escalado_a: "Coordinador",\ntimestamp_escalamiento: now()}
CEL -> COORD : "Alerta CRÍTICA escalada\npor inactividad (>24h)"
CEL -> RECT : "Riesgo de Ruptura de Ruta:\nalerta sin atención — Nivel Rector"

@enduml
```

---

## 3. Diagrama de Secuencia 2 — Vinculación de Padre por OTP y Revocación

**Escenario:** La institución genera un OTP para el acudiente. El Padre se registra. Posteriormente el Coordinador revoca el acceso.

```plantuml
@startuml AT360_Seq_OTP

skinparam sequenceArrowThickness 2

actor "Admin\nInstitución" as ADMIN
participant "Django\nBackend" as BE
database "PostgreSQL\n(Supabase)" as DB
actor "Padre de\nFamilia" as PAD
actor "Coordinador" as COORD

== Generación del OTP ==
ADMIN -> BE : POST /api/vinculos/generar-otp/\n{estudiante_id, acudiente_verificado: true}
BE -> BE : Generar OTP (6 dígitos)\nvalid_until = now() + 72h
BE -> DB : INSERT otp_tokens\n{token_hash, estudiante_id, valid_until, usado: false}
DB --> BE : otp_id
BE --> ADMIN : {otp: "A7X3K2"}\n(Entrega física al acudiente)

== Registro del Padre ==
PAD -> BE : POST /api/auth/registro-padre/\n{email, password, otp: "A7X3K2"}
BE -> DB : SELECT otp WHERE token_hash=hash("A7X3K2")\nAND valid_until > now() AND usado=false
DB --> BE : Token válido — estudiante_id
BE -> DB : INSERT parent_student\n{padre_id, estudiante_id, estado: "activo"}
BE -> DB : UPDATE otp_tokens SET usado=true
BE -> DB : INSERT audit_log\n{evento: "vinculacion_padre", actor: padre_id}
BE --> PAD : JWT (8h) + Nivel 1 Restringido

== Revocación por el Coordinador ==
COORD -> BE : DELETE /api/vinculos/{vinculo_id}/\n{razon: "Pérdida de custodia legal"}
BE -> BE : Validar razón (min 20 chars)
BE -> DB : UPDATE parent_student\n{estado: "revocado", razon, timestamp}
BE -> DB : INSERT token_blacklist\n{jwt_padre, revocado_en: now()}
BE -> DB : INSERT audit_log\n{evento: "revocacion_vinculo", actor: coord_id, razon}
BE --> COORD : HTTP 200 — Acceso revocado

PAD -> BE : GET /api/historial/ (con JWT anterior)
BE -> DB : SELECT token_blacklist WHERE token=jwt_padre
DB --> BE : Token en lista negra
BE --> PAD : HTTP 401\n"Su acceso ha sido revocado.\nContacte a la institución."

@enduml
```

---

## 4. Diagrama de Secuencia 3 — Confirmación Activa de Grupo y Bloqueo RLS

**Escenario:** Han pasado 15 días sin confirmación. El Docente intenta ver detalles, el RLS los bloquea. El Docente confirma y recupera el acceso. Un estudiante fue transferido — se activa Hand-off.

```plantuml
@startuml AT360_Seq_ConfirmacionActiva

skinparam sequenceArrowThickness 2

actor "Docente" as DOC
participant "Frontend\n(HTML/JS)" as FE
participant "Django\nBackend" as BE
database "PostgreSQL RLS\n(Supabase)" as DB
actor "Coordinador" as COORD

== Intento de acceso con validación vencida ==
DOC -> FE : Abrir dashboard
FE -> BE : GET /api/dashboard/
BE -> DB : SELECT teacher_course WHERE\nprofesor_id=X AND\nlast_verified_at < now()-15d
DB --> BE : [Vinculación vencida]
BE --> FE : HTTP 200\n{alertas: [{id, modulo, nombre_estudiante,\ndetalles: "[BLOQUEADO — validación requerida]"}]}
FE --> DOC : Dashboard con semáforo ROJO\ny banner de verificación requerida

== Proceso de Confirmación Activa ==
DOC -> FE : Iniciar verificación de grupo
FE -> BE : GET /api/mis-estudiantes/
BE -> DB : SELECT estudiantes WHERE teacher_id=X
DB --> BE : Lista de estudiantes
BE --> FE : Lista completa con estado semáforo

DOC -> FE : Marcar estudiante "María López" como\n"Ya no está en mi grupo"
FE -> BE : POST /api/confirmacion-grupo/\n{confirmados: [...], retirados: ["maria_lopez_id"]}

BE -> DB : SELECT alertas_activas WHERE\nestudiante_id="maria_lopez_id"
DB --> BE : [Sin alertas activas]

BE -> DB : UPDATE teacher_course SET\nestado="inactivo" WHERE\nestudiante_id="maria_lopez_id"
BE -> DB : UPDATE teacher_course SET\nlast_verified_at=now() WHERE\nprofesor_id=X (todos los confirmados)
BE -> DB : INSERT audit_log {evento: "confirmacion_activa"}

BE --> FE : HTTP 200 — Confirmación registrada
FE --> DOC : Dashboard con semáforo VERDE\nAcceso completo restaurado

BE -> COORD : Notificación:\n"Estudiante 'María López' quedó sin\ndocente asignado — reasignar"

@enduml
```

---

## Notas de implementación

- **Herramienta recomendada** para el diagrama de casos de uso: Draw.io con importación PlantUML o [diagrams.net](https://diagrams.net).
- **Herramienta recomendada** para los diagramas de secuencia: [PlantUML Online Editor](https://www.plantuml.com/plantuml/uml) — copiar el código de cada bloque y renderizar.
- Los diagramas de secuencia usan la notación estándar UML 2.5 con `alt`, `loop` y `note` de PlantUML para mayor claridad.
- Los nombres de los endpoints (`/api/alertas/`, `/api/vinculos/`) son orientativos — la implementación en Django REST Framework puede variar en la versión final.

---

*Documento preparado por el equipo Alerta Temprana 360.*  
*Versión 1.0 — Se actualiza con cada Sprint que modifique flujos principales.*
