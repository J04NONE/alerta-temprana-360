# 04 — Estrategia Técnica y Elasticidad: Alerta Temprana 360

Este documento aborda los desafíos estratégicos de elicitación y la arquitectura de nube necesaria para soportar la demanda proyectada de 10,000 usuarios concurrentes.

## 1. Descubrimiento de Requerimientos "Emergentes"
Para identificar necesidades que el cliente aún no visualiza (aquellas que surgen con el escalado), implementaremos las siguientes técnicas:

*   **Prototipado Evolutivo:** Entregas semanales de prototipos funcionales de alta fidelidad. Al interactuar con el dashboard, los coordinadores descubrirán la necesidad de filtros complejos o reportes de tendencias que no se ven en una entrevista inicial.
*   **Análisis de Escenarios de Excepción:** Talleres de "qué pasa si...". Ejemplo: "¿Qué pasa si un docente reporta un accidente un domingo a las 11 PM?". Esto revela requerimientos de disponibilidad y flujos de notificación fuera de horario.
*   **Historias de Usuario "Shadow":** Observación directa de cómo se manejan hoy las alertas en papel o Excel para detectar fricciones que el usuario ha normalizado y no menciona como requisito.

## 2. Especificación de Elasticidad (Requirement 10k)
La elasticidad es la capacidad del sistema para adaptarse a la carga en tiempo real. Se especifica como un requisito verificable mediante las siguientes métricas de Cloud Computing:

### Métricas de Verificación:
| Métrica | Objetivo / Umbral | Herramienta de Medición |
| :--- | :--- | :--- |
| **Tiempo de Respuesta (p99)** | < 800ms durante picos de 10k usuarios. | Locust / JMeter |
| **Tasa de Error** | < 0.1% durante procesos de Auto-scaling. | Sentry / CloudWatch |
| **Latencia de Escalado** | Nuevas instancias de Django deben estar activas en < 60 segundos tras superar el 70% de CPU. | AWS CloudWatch / Azure Monitor |
| **Throughput** | El sistema debe procesar al menos 500 transacciones por segundo (TPS). | Locust |

### Arquitectura para Elasticidad:
1.  **Capa de Backend (Django):** Despliegue en contenedores (Docker) gestionados por un clúster (Kubernetes o AWS ECS) con reglas de *Auto-Scaling Group* basadas en uso de CPU y RAM.
2.  **Capa de Datos (Supabase/PostgreSQL):** Uso de *Read Replicas* para distribuir la carga de consultas de los dashboards de directivos, manteniendo la instancia primaria para escrituras de alertas.
3.  **Capa de Aplicación:** Implementación de colas de tareas (Celery + Redis) para que el envío de 10,000 notificaciones simultáneas no bloquee el hilo principal de la aplicación.

## 3. Estrategia de Prueba de Carga
Para asegurar que el sistema no colapse si 10,000 padres acceden al mismo tiempo:
*   Se realizará una prueba de carga (*Stress Test*) usando **Locust**, simulando el flujo completo: Login -> Consulta de Alerta -> Carga de Seguimiento.
*   Se validará que el balanceador de carga distribuya el tráfico equitativamente y que el sistema "se encoja" automáticamente cuando la carga baje, optimizando costos.
