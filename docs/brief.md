# Brief — Asistente IA de portfolio (nestorbracho.com)

> Documento de entrada para BMAD / Claude Code. Cada fase se lanza por separado y solo se da por terminada cuando cumple su **Definición de terminado** y deja registradas sus **métricas**.

---

## 1. Objetivo

Construir un asistente conversacional en la home de nestorbracho.com que responda preguntas sobre la experiencia, los proyectos y el stack de Nestor, para dos públicos:

- **Headhunters / reclutadores**: "¿Ha trabajado con AWS?", "¿Qué hizo en el Banco Mundial?", "¿Está disponible?".
- **Clientes freelance**: "¿Puede construirme un WMS?", "¿Cómo trabaja?", "¿Cómo le contacto?".

### Objetivo secundario (el real)

Que el proyecto sea una **evidencia demostrable de ingeniería de IA en producción** para el CV: RAG, *tool use*, guardrails, evaluaciones, caché, routing de modelos y observabilidad, con métricas medidas.

### Fuera de alcance

- Entrenar o hacer fine-tuning de modelos.
- Agentes autónomos con acciones irreversibles.
- Multi-idioma más allá de español e inglés.

---

## 2. Principios

1. **Producción desde la fase 1.** Cada fase termina desplegada, no en una rama.
2. **Medir antes de optimizar.** Ninguna capa se añade sin una métrica de antes y después.
3. **No inventar experiencia.** El asistente solo afirma lo que está en la base de conocimiento. Si no lo sabe, lo dice y ofrece contacto.
4. **Coste controlado.** Hay un tope de gasto mensual y el sistema se degrada con elegancia al llegar a él.
5. **Todo documentado.** Cada decisión relevante se registra como ADR en `docs/adr/`.

---

## 3. Restricciones de contenido (obligatorias)

- **Cliente del WMS**: se presenta siempre de forma anónima ("WMS para una empresa logística"). El asistente no debe revelar su nombre aunque se le pregunte directamente.
- **Banco Mundial**: no dar métricas de escala ni datos internos que no estén autorizados. Solo lo que ya aparece en el CV y la web públicos.
- **Datos personales**: nada que no esté ya en el CV público (sin dirección, teléfono personal, etc.).
- **Disponibilidad y tarifas**: el asistente no da una tarifa ni compromete fechas; deriva al formulario de contacto.
- El nombre se escribe **"Nestor"**, sin tilde, por coherencia con la web.

---

## 4. Arquitectura objetivo

```
[Home nestorbracho.com]
        │  widget de chat
        ▼
[API Django + DRF]  ──►  rate limiting / guardrails de entrada
        │
        ├── Router ──► modelo barato (preguntas simples)
        │          └─► modelo potente (preguntas complejas)
        │
        ├── RAG ─────► PostgreSQL + pgvector (CV, casos, stack, FAQs)
        ├── Tools ───► get_case_study, get_stack, create_contact_request
        ├── Caché ───► Redis (respuestas y embeddings)
        └── Trazas ──► Langfuse (latencia, coste, calidad)

[Evals] ──► pytest + GitHub Actions (bloquea despliegues con regresiones)
```

### Stack propuesto

| Pieza | Elección | Motivo |
|---|---|---|
| Backend | Django + DRF | Stack principal de Nestor |
| LLM | API de Claude (familia Haiku / Sonnet) | Router entre un modelo barato y uno potente |
| Vector store | PostgreSQL + pgvector | Un solo motor de BD; integración con el ORM |
| Caché | Redis | Ya conocido (django-redis) |
| Observabilidad | Langfuse (self-hosted, Docker) | Trazas, coste y evaluaciones en un panel |
| CI | GitHub Actions | Evals en cada PR |
| Infra | AWS + Docker | Experiencia previa; a decidir el servicio exacto (ver §7) |

> El frontend del widget se integra con el stack actual del repo `blogweb`; revisar sus specs antes de la fase 1 y comprobar el diseño contra el material de marca del repo.

---

## 5. Fases

### Fase 1 — Modelo + prompt + registro

**Construir**
- Proyecto Django con un endpoint `POST /api/chat/` que llama a la API de Claude con un system prompt (identidad, tono, restricciones de la §3).
- Modelo `Conversation` / `Message` que guarda pregunta, respuesta, modelo, tokens, latencia y coste estimado.
- Widget de chat mínimo en la home.
- Rate limiting básico por IP.

**Definición de terminado**
- Desplegado y accesible desde la home.
- Cada mensaje queda registrado con tokens, latencia y coste.

**Métricas de línea base**
- Latencia p50 / p95.
- Coste medio por conversación.
- Anotar a mano 10 preguntas en las que el bot responde mal o inventa (alimentan la fase 2 y las evals).

**ADR**: elección de modelo inicial y estructura del system prompt.

---

### Fase 2 — RAG

**Construir**
- Base de conocimiento en `knowledge/` (Markdown versionado): CV (ES/EN), casos de estudio (WMS anónimo, CRM, Banco Mundial, monitoreo participativo), stack, forma de trabajar, FAQs.
- Pipeline de ingesta: trocear → generar embeddings → guardar en pgvector. Comando `manage.py ingest_knowledge`.
- Recuperación de los *k* fragmentos más relevantes e inyección en el contexto.
- La respuesta indica en qué sección se basa (citas).

**Definición de terminado**
- Las 10 preguntas fallidas de la fase 1 se responden correctamente o con "no lo sé".
- Volver a ejecutar la ingesta es idempotente.

**Métricas**
- Respuestas correctas sobre el set de 10 (antes → después).
- Precisión de recuperación: ¿el fragmento correcto está en el top-k?

**ADR**: estrategia de troceado, modelo de embeddings, valor de *k*.

---

### Fase 3 — Evaluaciones (adelantada a propósito)

> Va antes que el resto de capas para que todas las siguientes se midan contra ella.

**Construir**
- Dataset `evals/cases.yaml` con al menos 30 casos: pregunta, criterios de respuesta correcta y categoría (hechos, fuera de tema, *prompt injection*, datos confidenciales, contacto).
- Runner con pytest: comprobaciones deterministas (contiene / no contiene) + LLM-as-judge para la calidad.
- GitHub Actions: las evals se ejecutan en cada PR y el despliegue se bloquea si la tasa de acierto baja del umbral.

**Definición de terminado**
- Las evals se ejecutan en CI y hay un PR de prueba bloqueado por una regresión provocada a propósito.

**Métricas**
- Tasa de acierto global y por categoría.
- Coste de ejecutar el suite completo.

**ADR**: diseño del suite de evaluación y umbral de bloqueo.

---

### Fase 4 — Tools + control de acceso

**Construir**
- Herramientas expuestas al modelo:
  - `get_case_study(slug)` — detalle estructurado de un proyecto.
  - `get_stack(area)` — tecnologías por área (backend, frontend, infra, IA).
  - `create_contact_request(name, email, message)` — crea un lead y avisa a Nestor por email.
- Control de acceso:
  - Usuario anónimo: solo herramientas de lectura más `create_contact_request` con límite estricto.
  - Panel de administración autenticado (solo Nestor) para revisar conversaciones y leads.
- Validación de los argumentos de cada herramienta en el servidor (nunca confiar en lo que genera el modelo).

**Definición de terminado**
- El modelo usa la herramienta correcta en los casos de eval correspondientes.
- No es posible crear leads en bucle ni acceder al panel sin autenticación.

**Métricas**
- Acierto en la selección de herramientas (nuevos casos en las evals).
- Leads creados frente a conversaciones.

**ADR**: qué expone cada herramienta y por qué.

---

### Fase 5 — Guardrails

**Construir**
- Entrada: detección de *prompt injection* y de temas fuera de lugar; límite de longitud.
- Salida: comprobar que no aparece el nombre del cliente del WMS ni datos de la §3; comprobar que las afirmaciones se apoyan en el contexto recuperado.
- Respuesta estándar para fuera de tema, con redirección amable a lo que sí puede responder.

**Definición de terminado**
- 100 % de acierto en la categoría de evals "datos confidenciales".
- Al menos 90 % en "prompt injection" (ampliar con al menos 15 ataques conocidos).

**Métricas**
- Tasa de bloqueo correcto y falsos positivos (preguntas legítimas bloqueadas).

---

### Fase 6 — Caché + router

**Construir**
- Caché en Redis: respuestas exactas y embeddings de consultas; evaluar caché semántica (similitud por encima de un umbral).
- Router: clasificar la pregunta (simple / compleja) y enviarla al modelo barato o al potente.
- Tope de gasto mensual: al alcanzarlo, solo modelo barato y luego respuesta estática con contacto.

**Definición de terminado**
- La tasa de acierto en las evals no baja respecto a la fase 5.
- El tope de gasto se ha probado en staging.

**Métricas** (antes → después)
- Coste medio por conversación.
- Latencia p95.
- Porcentaje de aciertos de caché.
- Porcentaje de tráfico por modelo.

**ADR**: criterio de routing y umbral de caché semántica.

---

### Fase 7 — Observabilidad

**Construir**
- Langfuse self-hosted (Docker) con trazas de cada petición: recuperación, llamadas a herramientas, modelo, tokens, coste y latencia.
- Puntuación de las evals enviada a Langfuse para ver la calidad en el tiempo.
- Feedback 👍/👎 en el widget, vinculado a la traza.
- Alertas básicas: tasa de errores, coste diario, latencia.

**Definición de terminado**
- Cualquier conversación se puede reconstruir paso a paso desde el panel.

**Métricas**
- Panel con coste, latencia y feedback (captura para el caso de estudio).

---

## 6. Entregables para el CV y la marca

| Al terminar… | Entregable |
|---|---|
| Cada fase | Post de LinkedIn: el problema concreto y cómo se resolvió |
| Fase 3 | Badge o informe de evals en el README |
| Fase 6 | Tabla de coste y latencia antes → después |
| Fase 7 | Caso de estudio en la home y en el blog, con diagrama y métricas |
| Final | Entrada en el CV (ES/EN) con métricas reales y enlace a la demo |

### Plantilla de la entrada del CV (rellenar solo con datos medidos)

> **Asistente IA de portfolio (nestorbracho.com)** — Django, DRF, Claude API, pgvector, Redis, Langfuse
> - Diseñé y desplegué un asistente con RAG sobre pgvector, *tool use* y guardrails contra *prompt injection*, en producción y abierto al público.
> - Implementé evaluaciones automáticas en CI (__ casos, __ % de acierto) que bloquean despliegues con regresiones.
> - Reduje el coste por conversación un __ % y la latencia p95 de __ s a __ s con caché en Redis y routing entre modelos.

---

## 7. Decisiones abiertas (resolver antes de la fase 1)

- [ ] ¿Repo separado para el backend del asistente o dentro de `blogweb`?
- [ ] ¿Repo público? (Recomendado para el CV, sin secretos ni datos del cliente.)
- [ ] Hosting en AWS: ECS / Elastic Beanstalk / EC2 con Docker Compose.
- [ ] Tope de gasto mensual en la API.
- [ ] ¿Respuestas en el idioma de la pregunta o según el idioma de la página?
- [ ] Email de destino para los leads.
