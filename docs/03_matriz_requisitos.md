# 03 — Matriz de Requisitos: Alerta Temprana 360

Este documento especifica los requisitos funcionales, no funcionales y las restricciones del sistema, siguiendo la estructura de la ISO/IEC/IEEE 29148:2018.

## 1. Requisitos Funcionales (RF)

| ID | Nombre | Descripción | Prioridad |
| :--- | :--- | :--- | :--- |
| **RF-01** | Gestión de Usuarios y Roles | El sistema debe permitir el registro y autenticación de usuarios (Docente, Estudiante, Padre, Coordinador) con control de acceso basado en roles (RBAC). | Alta |
| **RF-02** | Registro de Alertas SED | El sistema debe permitir reportar incidentes en 6 módulos específicos: Abuso, Accidentalidad, Conducta Suicida, SPA, Maternidad Temprana y Trastornos de Aprendizaje. | Alta |
| **RF-03** | Detección de Riesgo Académico | El sistema debe identificar automáticamente estudiantes con 3+ matrículas en una materia o notas de 0.0 (basado en carga masiva). | Media |
| **RF-04** | Notificaciones en Tiempo Real | El sistema debe enviar alertas automáticas (Email/App) a los actores responsables inmediatamente después de activarse un indicador de riesgo crítico. | Alta |
| **RF-05** | Registro de Seguimiento | Permite a los docentes y coordinadores registrar cada intervención realizada sobre un caso activo (bitácora de seguimiento). | Alta |
| **RF-06** | Carga Masiva de Datos | Permite a los administradores importar archivos CSV/Excel con datos académicos y demográficos, validando la integridad antes de procesar. | Alta |
| **RF-07** | Escalamiento Automático | Si una alerta no es atendida en un periodo definido (ej. 48h), el sistema debe escalar la notificación automáticamente al nivel jerárquico superior. | Media |
| **RF-08** | Consulta de Historial 360 | Permite a los usuarios autorizados ver el historial consolidado de alertas y seguimientos de un estudiante a través del tiempo. | Alta |
| **RF-09** | Revocación de Vínculo | El Coordinador de Convivencia debe tener la capacidad de revocar el acceso de un Padre de Familia de forma inmediata ante pérdida de custodia o error en la vinculación. | Alta |

## 2. Requisitos No Funcionales (RNF)

| ID | Categoría | Requisito / Métrica | Prioridad |
| :--- | :--- | :--- | :--- |
| **RNF-01** | Rendimiento | El sistema debe responder en menos de 500ms bajo una carga de hasta 10,000 usuarios concurrentes. | Alta |
| **RNF-02** | Disponibilidad | El sistema debe garantizar un tiempo de actividad (Uptime) del 99.5% mensual. | Alta |
| **RNF-03** | Seguridad (Aislamiento) | Debe implementarse *Row-Level Security* (RLS) para asegurar que ningún usuario acceda a datos sensibles fuera de su jurisdicción (sede/curso). | Alta |
| **RNF-04** | Elasticidad | El sistema debe escalar sus recursos automáticamente ante picos de demanda (ej. cierre de semestre) sin degradación del servicio. | Alta |
| **RNF-05** | Usabilidad | Un usuario directivo debe poder identificar la alerta más crítica del dashboard en menos de 10 segundos. | Media |

## 3. Restricciones (RE)

| ID | Nombre | Descripción |
| :--- | :--- | :--- |
| **RE-01** | Protección de Datos | Cumplimiento estricto de la **Ley 1581 de 2012** (Habeas Data). Auditoría completa de cada acceso a datos sensibles. |
| **RE-02** | Marco Legal Educativo | Alineación con la **Ley 1620 de 2013** (Sistema Nacional de Convivencia Escolar). |
| **RE-03** | Stack Tecnológico | Uso mandatorio de Django (Backend), Supabase/PostgreSQL (DB) y HTML/CSS/JS (Frontend). |
| **RE-04** | Sin Integración Directa | Los datos externos entran exclusivamente vía carga masiva de archivos; no hay conexión directa a bases de datos SIAU externas. |
