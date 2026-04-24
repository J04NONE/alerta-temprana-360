# Alerta Temprana 360 — Documento Interno del Equipo

> **Para:** Andrés, Iván, Michael  
> **Estado:** Trabajo en progreso — se actualiza después de cada reunión con el profesor  
> **Objetivo de este doc:** Que cada uno sepa qué tiene que llegar a hacer, por qué, y cómo.

---

## Contexto rápido (si alguien no leyó lo anterior)

Estamos construyendo un sistema de soporte a decisiones para retención estudiantil. Detecta estudiantes en riesgo (3+ matrículas, notas en cero, becas agotadas) y le da visibilidad a quienes pueden intervenir.

La primera reunión con el profesor es básicamente nuestra sesión de elicitación de requisitos. Él actúa como cliente/stakeholder. Vamos con una propuesta inicial — no a pedirle permiso para empezar.

---

## Lo que llevamos a esa reunión

Una propuesta estructurada en tres partes:

1. **Descripción del problema** — por qué existe el sistema
2. **Identificación de actores** — quién interactúa con él y para qué
3. **Requisitos iniciales** — 5 RF + 5 RNF + restricciones conocidas

Los RF ya están redactados (ver documento de reunión). Los RNF los construye Iván antes de la sesión. Las restricciones las completa Michael con lo que ya sabemos.

---

## Tareas por rol

### Andrés — Diseño y flujo de decisión

Tu trabajo antes de la reunión no es hacer mockups todavía. Es responder una sola pregunta:

**¿Cómo identifica un directivo la alerta más grave cuando hay 200 activas?**

Eso es un problema de jerarquía visual, no de colores. Piensa en niveles de urgencia, agrupación por tipo de riesgo, y qué acción puede tomar el directivo directamente desde la vista principal. Llega a la reunión con ese flujo en papel o digital — aunque sea un esquema a mano.

Para la sesión, tú haces estas preguntas al profesor:

- "Cuando el decano abre el sistema y ve 200 alertas, ¿qué necesita ver primero para actuar en menos de 10 segundos?"
- "¿Los padres verían el historial completo de notas o solo la notificación de riesgo?"
- "¿El docente registra seguimientos desde el celular (en la tutoría) o principalmente desde computador?"

Las respuestas te van a definir la arquitectura de información del dashboard. Sin eso, cualquier diseño que hagas es un supuesto.

---

### Iván — Análisis y requisitos

Tienes dos entregas concretas antes de la reunión:

#### Entrega 1: 5 Requisitos No Funcionales (RNF)

Un RNF bien escrito tiene esta estructura:

```
ID:          RNF-XX
Nombre:      [Categoría de calidad]
Descripción: El sistema debe [condición medible] bajo [contexto específico].
Métrica:     [Cómo se verifica — número, porcentaje, herramienta]
Prioridad:   Alta / Media / Baja
```

Lo más importante: **la descripción tiene que ser verificable**. "El sistema debe ser rápido" no sirve. "El sistema debe responder en menos de 500ms bajo carga de 10,000 usuarios concurrentes" sí sirve — porque se puede medir con Locust.

Aquí tienes los 5 RNF que debes redactar con esa estructura (estos son los temas, la redacción formal es tuya):

| ID | Categoría | Qué debe expresar |
|----|-----------|-------------------|
| RNF-01 | Rendimiento | Tiempo de respuesta máximo bajo carga concurrente |
| RNF-02 | Disponibilidad | Uptime mínimo esperado del sistema |
| RNF-03 | Seguridad | Autenticación, cifrado y control de acceso por roles |
| RNF-04 | Usabilidad | Tiempo máximo para que un directivo identifique la alerta más crítica |
| RNF-05 | Escalabilidad | Capacidad de crecer a múltiples sedes sin cambios en el código |

Cuando los redactes, usa verbos concretos: *debe soportar*, *debe responder en*, *debe registrar*, *no debe exceder*. Evita *garantizará*, *permitirá que*, *buscará asegurar* — eso es lenguaje de intención, no de requisito.

#### Entrega 2: Ideas de casos de uso (al menos 3)

No hace falta el diagrama UML todavía. Necesitamos el listado en texto con esta forma:

```
CU-XX | Nombre del caso de uso
Actor principal: [quién inicia la acción]
Descripción: [qué hace el actor y qué responde el sistema]
Precondición: [qué debe ser verdad antes de que empiece]
Resultado: [qué cambia en el sistema al terminar]
```

Tres casos de uso que puedes empezar (desarrolla los que más te convenzan):

- **CU-01:** Detección automática de estudiante en riesgo
- **CU-02:** Registro de seguimiento por parte del docente
- **CU-03:** Carga masiva de datos académicos (CSV/Excel)
- **CU-04:** Consulta del historial de riesgo de un estudiante
- **CU-05:** Escalamiento de alerta no atendida al Director de Programa

En la reunión, tú haces estas preguntas:

- "¿Cuál es el criterio exacto de 'Riesgo Crítico'? ¿Solo matrículas o también promedio ponderado?"
- "La carga por Excel: ¿la hace cada docente o un administrador de facultad?"
- "¿Existen indicadores silenciosos que el sistema debería detectar, como ausencia prolongada o beca agotada antes de fin de semestre?"

---

### Michael — Scrum Master y documentación

Tu rol en la reunión es el de moderador y registrador. Abre la sesión, controla el tiempo y documenta todo lo que el profesor diga como decisión o restricción.

Antes de la reunión, prepara:

- Un registro de las restricciones conocidas (sin SIAU, sin pasarela de pagos, etc.)
- La propuesta de metodología híbrida para presentarle al profesor (ver documento de reunión)
- La tabla de decisiones donde vas a anotar en vivo las respuestas

En la reunión, tus preguntas son:

- "¿Cómo maneja el sistema el consentimiento cuando un padre quiere ver datos de un estudiante adulto? ¿Necesitamos un módulo de autorización digital?"
- "Los 10,000 usuarios concurrentes: ¿son para un despliegue nacional o para picos durante cierre de semestre?"
- "¿Podemos actualizar el documento de requisitos incrementalmente por Sprint en lugar de entregarlo todo al inicio?"

Después de la reunión, convierte las respuestas en dos artefactos:
1. **Restricciones del sistema** — lo que el profesor dijo que no puede cambiar
2. **Decisiones de arquitectura** — lo que acordamos y por qué

---

## Qué NO hacer en la reunión

- No pregunten qué tecnología usar. Eso lo decidimos nosotros.
- No lleguen a pedir que él defina los requisitos. Lleguen con los requisitos y que él los corrija o apruebe.
- Si el profesor pregunta por los 10,000 usuarios, la respuesta es Locust — prueba de carga, 500ms de latencia máxima, arquitectura en la nube.

---

*Próxima actualización: después de la reunión con el profesor*
