---
source: gitlab.com/nestorbracho (perfil revisado el 2026-11-25)
status: REVISADO — confirmaciones de Nestor aplicadas el 2026-11-25
visibility: público (lo leerá el asistente de nestorbracho.com)
---

# Experiencia en GitLab (2016–2024)

Cuenta de GitLab activa desde junio de 2016. Unos 44 repositorios entre proyectos de empresa, de clientes y personales, con más de 500 commits propios. GitLab fue su plataforma principal de control de versiones durante la etapa en Turpial Development y en varios proyectos freelance, antes de pasar a GitHub.

## Prácticas de trabajo

- **Git flow con merge requests**: ramas `feature/*` y `bugfix/*` vinculadas a issues, y promoción por entornos (`develop` → `pre-dev` → `master`) mediante merge requests.
- **GitLab CI/CD**:
  - Pipelines de tests para Django con un servicio de PostgreSQL en contenedor.
  - Despliegue de sitios estáticos con GitLab Pages (Jekyll).
- **Despliegue en AWS Elastic Beanstalk** (`.ebextensions`, `.platform`) con media en S3, en varios proyectos Django.
- Versionado de releases con scripts propios.

## Proyectos destacados

### Plataforma SaaS de análisis de filtración de aire industrial (2019–2024)
- Cliente anónimo: no se nombra la empresa.
- Backend del producto durante más de 4 años, en un equipo de unas 7 personas; unos 180 commits propios.
- Stack: Python, Django, Django REST Framework, PostgreSQL, AWS (Elastic Beanstalk, S3), GitLab CI.
- Integración con sensores y registradores de datos de terceros para monitorizar la calidad del aire, con tareas programadas (cron jobs) de monitorización e informes.
- Módulos de cálculo de costes y consumo energético, comparación de propuestas y snapshots de costes, con numpy y matplotlib.
- API documentada con OpenAPI (drf-spectacular), pagos con Stripe y backoffice con AdminLTE.

### Plataforma de registro de beneficiarios con códigos QR — freelance para el Banco Mundial (2022–2023)
- Mismas reglas que el resto del trabajo con el Banco Mundial: sin métricas de escala ni datos internos no autorizados.
- Django + DRF, registro y perfiles de beneficiarios con generación de códigos QR, flujo por pasos y permisos por rol.
- Integración por webhook con KoboToolbox (recogida de datos en campo).
- Cifrado de datos sensibles (pycryptodome) y despliegue en AWS Elastic Beanstalk.

### Plataforma de gestión de inversiones — proyecto propio (2021–2022)
- Django, Celery (tareas asíncronas y programadas), PostgreSQL, S3, Elastic Beanstalk.
- Gestión de operaciones, contratos, liquidaciones y movimientos de fondos, con panel de superadministración y notificaciones por email.

### Sistema de pedidos y punto de venta para food trucks (2019)
- API REST con Django REST Framework: platos, menús, pedidos y cola de cocina, con roles de administrador, caja y cocina.
- Notificaciones push con Firebase Cloud Messaging; frontend en Angular desarrollado por otro miembro del equipo.

### Sistema de crédito para máquinas de café (2018)
- API con Django REST Framework, login con Google, MySQL y app móvil en Flutter.

### Plataforma de solicitudes de cambio de divisas (2020)
- Django, flujo completo de solicitudes de compra y venta, precio de referencia obtenido por scraping, despliegue en Elastic Beanstalk.

### Web corporativa de Turpial Development (2018)
- Sitio en Jekyll, multilenguaje, con portafolio de proyectos en Markdown, formulario de contacto y despliegue con GitLab Pages CI.

### Formación de nuevos programadores en Turpial Development (2017–2018)
- Formador en el programa interno de onboarding de desarrolladores ("Jedi Training").
- Cada nuevo programador hacía un proyecto de inducción (por ejemplo, un blog o una lista de tareas) sobre proyectos base en Django con pytest y en Angular.

## Tecnologías que aparecen en estos repositorios

- **Backend**: Python, Django, Django REST Framework, Celery, PostgreSQL, MySQL, SQLite
- **Frontend**: Angular, TypeScript, Jekyll, Bootstrap, AdminLTE
- **Móvil**: Flutter/Dart (proyectos de 2018)
- **Infraestructura**: AWS Elastic Beanstalk, S3, GitLab CI/CD, GitLab Pages
- **Integraciones**: Stripe, Firebase Cloud Messaging, Google OAuth, KoboToolbox, sensores IoT de terceros, OpenAPI
