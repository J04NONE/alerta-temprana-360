# 🚀 GUÍA MAESTRA: CONTEXTO, ARQUITECTURA Y SINCRONIZACIÓN LOCAL - ALERTA TEMPRANA 360 (AT360)

> **Directiva de la Junta de Arquitectura para el Agente Autónomo del IDE (Antigravity IDE)**
> **Estándar de Referencia:** ISO/IEC/IEEE 29148:2018 | Ley 1581 de 2012 (Habeas Data) | Ley 1620 de 2013 (Convivencia Escolar)
> **Estado del Repositorio:** Sincronizado vía API con Taiga. Siguiente paso: Implementación del Sprint 1.

---

## 1. RECONSTRUCCIÓN TOTAL DEL CONTEXTO (AT360 v2.0 - PIVOT SED)

### 1.1 El Cambio de Paradigma: De lo Académico a la Convivencia
Originalmente concebido como un sistema para la detección de riesgo académico (pérdida de materias, bajas calificaciones, inasistencias), el proyecto sufrió un **pivot estratégico radical de alcance** solicitado por el cliente (el docente de la cátedra de Gestión de Requisitos). El sistema se ha transformado en un **Sistema de Soporte a Decisiones (DSS) para la Convivencia Escolar**, adoptando el modelo normativo y operativo de la **Secretaría de Educación del Distrito (SED) de Bogotá**.

Este nuevo enfoque traslada el procesamiento de datos cuantitativos sencillos (promedios) hacia la gestión cualitativa y el ruteo de alertas de alta sensibilidad humana y legal. El sistema ahora se estructura en **seis módulos de alerta crítica**:

1. **Abuso y Violencia (MOD-01):** Detección y reporte de presuntos casos de maltrato físico, psicológico, negligencia o violencia sexual dentro o fuera del entorno escolar.
2. **Accidentalidad Escolar (MOD-02):** Registro inmediato y activación del protocolo de emergencia ante accidentes físicos que involucren a estudiantes dentro de la institución o en actividades escolares.
3. **Conducta Suicida (MOD-03):** Identificación de factores de riesgo vital (ideación, intentos, autolesiones) y enrutamiento prioritario para atención médica y psicológica urgente.
4. **Consumo de SPA - Sustancias Psicoactivas (MOD-04):** Monitoreo de riesgos de adicciones y vulnerabilidades asociadas a la salud pública.
5. **Maternidad y Paternidad Tempranas (MOD-05):** Acompañamiento pedagógico y adaptaciones institucionales para garantizar la permanencia del estudiante gestante o lactante en el sistema educativo.
6. **Trastornos de Aprendizaje y Comportamiento (MOD-06):** Registro de alertas pedagógicas para coordinar apoyos personalizados bajo el marco de educación inclusiva.

### 1.2 El Problema Legal y la Restricción de Interoperabilidad
El sistema opera bajo la **Restricción de No Interoperabilidad Directa** con el sistema central oficial (SIAU / SIMAT). No contamos con APIs en tiempo real para jalar la matrícula base ni la patria potestad de los acudientes. Todo el aprovisionamiento de datos se realiza mediante **Cargas Masivas de Archivos CSV (RF-04 y RF-06)**.

Dado que gestionamos información de carácter sensible de menores de edad (salud mental, presuntas vulneraciones, historial clínico-disciplinario), el sistema está legalmente sujeto al cumplimiento de la **Ley 1581 de 2012 (Habeas Data)** en Colombia. Cualquier fuga de datos o acceso no autorizado (ej. un docente que ve casos de abuso de un grupo que no es el suyo) constituye una violación legal grave y el fracaso inmediato del proyecto.

---

## 2. ARQUITECTURA DE SEGURIDAD Y PRIVACIDAD POR DISEÑO (PbD)

Para mitigar los riesgos derivados de la Ley 1581 y las limitaciones del CSV estático, hemos diseñado tres salvaguardas arquitectónicas críticas en el backend:

```
                  ┌─────────────────────────────────────┐
                  │          CAPA DE APLICACIÓN         │
                  │   Django REST Framework (RBAC)      │
                  └──────────────────┬──────────────────┘
                                     │ (Filtro por Rol / Contexto JWT)
                                     ▼
                  ┌─────────────────────────────────────┐
                  │       MIDDLEWARE DE SEGURIDAD       │
                  │   Confirmación Activa de Grupo      │
                  │   (Validación last_verified_at)     │
                  └──────────────────┬──────────────────┘
                                     │ (Inyección de tenant_id / grupo_id)
                                     ▼
                  ┌─────────────────────────────────────┐
                  │          CAPA DE PERSISTENCIA       │
                  │   PostgreSQL + Supabase (RLS)       │
                  └─────────────────────────────────────┘
```

### 2.1 Mitigación de Privilegios Residuales: Confirmación Activa de Grupo (RF-11)
* **El Escenario de Falla:** Un estudiante es transferido de curso o un docente es reemplazado a mitad de semestre. Si dependemos de que el administrador suba un nuevo CSV, habrá una brecha de tiempo donde el docente anterior conservará acceso a las alertas íntimas del menor, violando el principio de necesidad y confidencialidad.
* **La Solución (RF-11):** Implementamos un **Ciclo de Vida de Vinculación**. La relación en la base de datos `TeacherCourse` cuenta con los campos `valid_until` y `last_verified_at`. Cada 15 días, el docente de aula debe realizar una **Confirmación Activa de Grupo** mediante un botón simple en su dashboard que certifique que sigue a cargo de dichos estudiantes. Si pasan más de 15 días sin validación, el backend en Django bloquea preventivamente las consultas a la base de datos (mediante filtros automatizados de RLS), impidiendo visualizar el detalle de las alertas del grupo hasta que se realice la re-confirmación.

### 2.2 Vinculación Segura de Padres de Familia mediante Token/QR OTP
* **El Escenario de Falla:** ¿Cómo vincular a un padre con su hijo de forma digital para enviarle notificaciones automáticas de alertas, garantizando que el usuario realmente es el acudiente legal y sin tener conexión a la Registraduría?
* **La Solución (RF-02):** Flujo de **Token OTP de Vinculación**. Al procesar la matrícula base por CSV, el sistema genera de forma automática un hash criptográfico único, temporal y de un solo uso por estudiante. La institución educativa imprime físicamente una "Ficha de Vinculación" (con el QR que contiene la URL con dicho token) y la entrega en la reunión de padres. Al escanear el QR, el backend de Django valida el token cifrado, asocia al usuario `Padre` con el `Estudiante` en la tabla intermedia `ParentStudent`, e invalida el token de inmediato.

### 2.3 Validación de Calidad de Alertas (RF-10 - Flujo Multinivel)
Para evitar que un reporte de alta gravedad sea despachado a entes externos (como el ICBF, Comisarías de Familia o Secretaría de Salud) con información errónea o falsa, diseñamos un flujo de aprobación estricto:
1. **Checklist Obligatorio (Docente):** Un formulario inteligente basado en las directrices técnicas de la SED. El docente debe completar campos obligatorios de modo, tiempo, lugar y acciones inmediatas tomadas antes de poder procesar la alerta.
2. **Triage Automático:** El sistema evalúa el módulo y la gravedad del incidente (ej. Conducta Suicida es clasificada automáticamente como Criticidad Alta).
3. **Visto Bueno del Coordinador (Aprobador):** Las alertas clasificadas como críticas no salen del sistema hacia entidades gubernamentales externas hasta que el Coordinador de Convivencia (Nivel 3) revise la cadena de hechos y otorgue su aprobación digitalizada.

### 2.4 Alertas Huérfanas y Hand-off Obligatorio (RB-37)
* **Regla de Negocio (RB-37):** *"El sistema bajo ningún escenario permitirá desvincular o archivar a un estudiante si posee una alerta abierta o en seguimiento dentro de los módulos MOD-01 (Abuso) o MOD-03 (Suicidio)"*.
* Si un docente intenta desvincular a un estudiante de su grupo por error académico o traslado, el backend detendrá la transacción y exigirá la ejecución de un **Hand-off (Traspaso de caso)** explícito que asigne al Coordinador o al nuevo docente como custodio del expediente de acompañamiento.

---

## 3. REDISTRIBUCIÓN DE ROLES Y RESPONSABILIDADES TÉCNICAS

Para optimizar la curva de aprendizaje del equipo y asegurar la entrega impecable del backend en Django, Michael asume un rol híbrido de Scrum Master y Backend Developer Principal. La distribución de complejidad técnica se divide en un **65% (Michael) - 35% (Iván)**:

### 3.1 Matriz de Asignación por Complejidad y Sprint

| ID Tarea | Descripción Técnica de la Tarea | Dificultad | Sprint | Responsable Principal |
| :--- | :--- | :--- | :--- | :--- |
| **T-01** | **Setup Base & CI/CD:** Configuración inicial del proyecto Django, inicialización de modelos de usuario extendidos (AbstractUser), estructuración del middleware para variables de entorno `.env` y creación de pipelines de GitHub Actions (Linter Black/Flake8 + Django Tests). | **Alta** | Sprint 1 | **Michael (SM/Dev)** |
| **T-02** | **Lógica Core de Seguridad (RLS):** Codificación del middleware de Django para interceptar solicitudes de datos y mapear los esquemas de Row-Level Security (RLS) según el `grupo_id` o `caso_id` del usuario autenticado. | **Alta** | Sprint 1 | **Michael (SM/Dev)** |
| **T-03** | **Criptografía OTP & QR:** Desarrollo del servicio de generación de hashes únicos de vinculación de padres, lógica de expiración de tokens en Redis o base de datos y rendering dinámico del código QR mediante `qrcode` y `Pillow`. | **Alta** | Sprint 1 | **Michael (SM/Dev)** |
| **T-04** | **Motor de Enrutamiento SED:** Lógica en Django que valida si un reporte cumple con el checklist y las reglas de triage de criticidad del RF-10 antes de despacharse. | **Alta** | Sprint 2 | **Michael (SM/Dev)** |
| **T-05** | **Modelado de Persistencia (Models):** Creación de las clases en `models.py` para Estudiantes, Docentes, Cursos, Sedes y la estructura base de los 6 módulos de alerta de la SED. | **Baja** | Sprint 1 | **Iván (Analista/Dev)** |
| **T-06** | **Carga Masiva (CSV Parser):** Servicio en Django para decodificar archivos CSV cargados por la administración, realizar validaciones sintácticas básicas y poblar la base de datos de manera masiva. | **Media** | Sprint 1 | **Iván (Analista/Dev)** |
| **T-07** | **Servicio de Notificación (SMTP):** Implementación de la pasarela de salida de correos electrónicos para despachar las alertas automáticas a los actores correspondientes. | **Baja** | Sprint 1 | **Iván (Analista/Dev)** |
| **T-08** | **Dashboard & QR Layout (UI/UX):** Prototipado y diseño visual del dashboard de criticidad escolar y la ficha imprimible de vinculación familiar. | **Media** | Sprint 1 | **Andrés (Diseñador)** |

---

## 4. GUÍA DE INSTALACIÓN Y SINCRONIZACIÓN LOCAL (Para el Agente del IDE)

Sigue estos pasos de forma estricta para sincronizar tu entorno de desarrollo local con el estado actual del repositorio oficial y preparar el espacio de trabajo:

### Paso 1: Clonar y Acceder
```bash
git clone https://github.com/j04none/alerta-temprana-360.git
cd alerta-temprana-360
```

### Paso 2: Configurar Entorno de Ejecución (Python)
```bash
# Crear entorno virtual de aislamiento
python -m venv venv

# Activar el entorno virtual:
# Windows:
.\venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate
```

### Paso 3: Instalar Dependencias del Core
```bash
# Actualizar pip e instalar los módulos requeridos por el core del backend
pip install --upgrade pip
pip install django djangorestframework psycopg2-binary python-dotenv django-cors-headers qrcode pillow

# Congelar dependencias para asegurar consistencia en el repo
pip freeze > requirements.txt
```

### Paso 4: Configurar Variables de Entorno Locales (`.env`)
Dado que el archivo `.gitignore` ya está protegiendo tus credenciales, debes crear un archivo `.env` en la raíz de tu proyecto local para enlazar el servidor local de Django con la base de datos PostgreSQL alojada en Supabase de forma segura:

```env
# Configuración del Entorno de Desarrollo
DEBUG=True
SECRET_KEY=django-insecure-master-secret-key-at360-project-2026

# Conexión a Base de Datos Supabase (PostgreSQL elástica)
DB_NAME=postgres
DB_USER=postgres
DB_PASSWORD=tu_password_de_supabase_aqui
DB_HOST=db.your-project-id.supabase.co
DB_PORT=5432
```

### Paso 5: Protocolo de Commits y Trazabilidad (Conventional Commits)
Para garantizar la trazabilidad exigida por la norma **ISO 29148**, cada commit local debe seguir la estructura estándar y estar estrictamente asociado a su Historia de Usuario o Requerimiento Funcional:

* **Para nuevas funcionalidades backend:**
  `feat(auth): implementar middleware de confirmacion activa para mitigar privilegios residuales #HU-03`
* **Para modelado de datos:**
  `feat(models): estructurar modelo de persistencia para los 6 modulos de alertas SED #HU-05`
* **Para correcciones de código:**
  `fix(security): resolver error de bypass en las politicas de RLS de Supabase #fix-db`

---

## 5. HOJA DE RUTA Y PROTOCOLO DE ACTUALIZACIÓN DOCUMENTAL (EVOLUCIÓN DOCUMENTAL ÁGIL)

La documentación de requisitos **no es estática** (se descartó el modelo rígido de la IEEE 830). En cumplimiento con el acuerdo establecido con el profesor (cliente), la documentación se mantendrá como un **activo incremental vivo** alineado con los Sprints:

```
            Sprint Planning (Ajuste de Requisitos en docs/03)
                                │
                                ▼
            Fase de Desarrollo (Codificación + Pruebas unitarias)
                                │
                                ▼
            Definición de Done (DoD) (Actualización obligatoria en docs/)
                                │
                                ▼
            Sprint Review (Versionamiento con Tag Git en GitHub)
```

1. **Gatillo de Cambio en Planificación:** Si al iniciar una Historia de Usuario se detecta una restricción técnica o de negocio nueva, Iván debe actualizar inmediatamente el documento `docs/03_matriz_requisitos.md` y `docs/05_especificacion_requisitos.md` antes de escribir código.
2. **Gatillo en el Definition of Done (DoD):** Michael no aprobará el merge de ningún Pull Request (PR) en GitHub si el código modifica lógica de negocio que no esté reflejada y actualizada en los diagramas de `docs/07_diagramas_uml.md` o las especificaciones del archivo `docs/05`.
3. **Cierre de Ciclo (Sprint Review):** Al finalizar cada Sprint de dos semanas, Michael consolidará los avances creando un Tag de versión en Git (ej. `v0.1-sprint1-baseline`), permitiendo al profesor verificar la evolución histórica del software y sus especificaciones de requisitos de forma paralela.

---

## 6. ARCHIVO README.md MAESTRO (PRODUCCIÓN Y DOCUMENTACIÓN PÚBLICA)

Este es el archivo `README.md` que debe ser colocado en la raíz de tu proyecto para unificar la cara pública del repositorio ante el docente y la comunidad de desarrollo:

```markdown
# Alerta Temprana 360 (AT360) 🚀
> **Sistema Inteligente de Soporte a Decisiones (DSS) para la Convivencia Escolar y la Retención Estudiantil.**

Diseñado bajo las directrices del Sistema de Alertas de la **Secretaría de Educación del Distrito (SED) de Bogotá**, la **Ley 1620 de 2013 (Convivencia Escolar)** y el estándar internacional de ingeniería de requisitos **ISO/IEC/IEEE 29148:2018**.

---

## 🛠️ Stack Tecnológico
* **Backend:** Python 3.11+ / Django 5.x / Django REST Framework (DRF)
* **Base de Datos:** PostgreSQL (Alojado en **Supabase** de forma elástica)
* **Frontend:** HTML5, CSS3 (Tailwind CSS o Bootstrap) y JavaScript Vanilla (Diseño responsivo, móvil-primero)
* **Seguridad:** Row-Level Security (RLS) en Postgres, cifrado de tokens OTP, y control de acceso basado en roles (RBAC).
* **CI/CD:** GitHub Actions (Validaciones de sintaxis automáticas y ejecución de suites de pruebas unitarias).

---

## 👥 Equipo y Roles de Ingeniería

Para maximizar el aprendizaje técnico de forma transversal en el ciclo de vida del software, el equipo colabora de manera integrada en todas las etapas, liderando las siguientes áreas:

* **Michael (j04none):** Scrum Master, Documentador Principal & Desarrollador Backend (65% de complejidad técnica: Infraestructura de despliegue, RLS de base de datos, middleware de control de accesos, cifrado OTP/QR y automatización de pipelines de CI/CD).
* **Iván (ivelasco31):** Analista de Requisitos & Desarrollador Backend (35% de complejidad técnica: Capa de persistencia relacional, CRUDs de base de datos, servicio SMTP de notificaciones y parsing Delta-CSV).
* **Andrés (andresbanderas864):** Diseñador UI/UX & Frontend (Definición de maquetación responsiva, flujos de experiencia interactivos, visualización de dashboards y representación física del código QR).

---

## 🔒 Privacidad por Diseño (PbD) y Cumplimiento de Ley 1581

Debido a que AT360 administra datos altamente sensibles de menores de edad (salud mental, presuntas vulneraciones de derechos), la arquitectura se rige bajo la **Ley 1581 de 2012 (Habeas Data)** en Colombia mediante tres controles específicos:

1. **Row-Level Security (RLS):** Las consultas a la base de datos están limitadas a nivel de registro físico. Un docente o psicólogo solo puede acceder a información enlazada a su identificador de grupo (`grupo_id`) o caso asignado (`caso_id`).
2. **Confirmación Activa de Grupo:** La relación de acceso docente-grupo expira preventivamente a los 15 días. El docente debe ratificar activamente su vigencia para evitar el riesgo legal de "privilegios residuales" tras traslados escolares.
3. **Ficha de Vinculación QR (Token de Un Solo Uso):** La vinculación familiar no expone datos de acudientes en bases externas desactualizadas; se realiza de forma física y segura mediante tokens de un solo uso criptográficamente firmados por el backend.

---

## 📁 Estructura del Repositorio

```text
alerta-temprana-360/
├── .github/                # Configuración de GitHub Actions (Pipelines CI/CD)
├── docs/                   # Documentación de Requisitos (ISO/IEC/IEEE 29148:2018)
│   ├── 01_descripcion_problema.md
│   ├── 02_matriz_actores.md
│   ├── 03_matriz_requisitos.md
│   ├── 04_estrategia_tecnica.md
│   ├── 05_especificacion_requisitos.md
│   ├── 06_historias_usuario.md
│   ├── 07_diagramas_uml.md
│   ├── 08_product_backlog.md
│   └── 09_sprint_backlog.md
├── src/                    # Código Fuente del Proyecto
│   ├── backend/            # Aplicación Django (API y Core)
│   └── frontend/           # Plantillas y activos estáticos (UI/UX)
├── .gitignore              # Exclusiones de Git (Protección de .env y entornos virtuales)
├── README.md               # Este archivo descriptivo
└── LICENSE                 # Licencia del software (MIT)
```

---

## 🚀 Guía de Instalación Rápida (Local)

1. **Clonar e inicializar entorno virtual:**
   ```bash
   git clone https://github.com/j04none/alerta-temprana-360.git
   cd alerta-temprana-360
   python -m venv venv
   source venv/bin/activate  # venv\Scripts\activate en Windows
   ```

2. **Instalar dependencias:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Configurar Variables de Env:**
   Crea un archivo `.env` en la raíz del proyecto agregando las credenciales de Supabase proporcionadas por el Scrum Master.

4. **Ejecutar servidor de desarrollo:**
   ```bash
   python src/backend/manage.py runserver
   ```
```

---
