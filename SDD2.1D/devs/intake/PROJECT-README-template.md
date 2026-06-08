# PROJECT-README-template

Plantilla metodológica SDD 2.0 para producir el documento `PROJECT-README-<nombre-kebab>_v1.0.md` durante la Fase 1.B del bootstrap. Captura las decisiones técnicas de construcción del sistema descripto en `PROJECT-BRIEF`. Este artefacto alimenta downstream a las categorías `05_arquitectura_tecnica/`, `09_devops/` y `11_examples/`, y el campo `Tipo de proyecto` gobierna las variantes de especialidad de las 12 categorías SDD.

## Guía de uso de esta plantilla

1. Copiar este archivo como `PROJECT-README-<nombre-kebab>_v1.0.md` en la raíz del repositorio del proyecto.
2. Reemplazar el bloque de cabecera con los valores reales del proyecto.
3. Recorrer cada sección §1 a §17 en orden, contestando las preguntas guía marcadas con `(*)` (bloqueantes) antes de avanzar a la siguiente sección.
4. Borrar los bloques `Ejemplo genérico` y `Lo que NO va en esta sección` una vez completado cada apartado, dejando solo el contenido productivo del proyecto.
5. Validar el §17 (checklist) antes de cambiar el estado a Aprobado.
6. Versionar el documento siempre como `_v1.0` en su primera emisión; los cambios posteriores siguen la política de versionado del propio repositorio.

---

## Cabecera del documento

Bloque obligatorio al inicio del documento generado. Reproducir el siguiente bloque y completar los placeholders:

| Campo | Valor |
|---|---|
| Nombre del proyecto | [Nombre legible del proyecto] |
| Repositorio | [Repositorio URL] |
| Lead técnico | [Nombre y rol] |
| Documento | `PROJECT-README-<nombre-kebab>_v1.0.md` |
| Versión | 1.0 |
| Fecha | [YYYY-MM-DD] |
| Stack principal | [Lenguaje + framework principal] |
| Estado | Borrador / En revisión / Aprobado |

> Este documento captura cómo se va a construir el sistema descrito en PROJECT-BRIEF.
> Contiene decisiones técnicas de stack, arquitectura, testing, CI/CD, samples.
> NO repite las necesidades del cliente: las asume conocidas vía PROJECT-BRIEF.

---

## §1 Tipo de proyecto

**Instrucción:** Seleccionar exactamente uno de los 8 valores cerrados D8. Este campo es bloqueante para el resto del documento porque define las variantes de especialidad de las 12 categorías SDD y el contenido obligatorio de `/src`, `/samples` y los documentos de `05_arquitectura_tecnica/` y `09_devops/`. No mezclar dos tipos: si el sistema combina varios artefactos desplegables (por ejemplo, un servicio REST más una CLI), elegir el tipo dominante y declarar el secundario como sub-proyecto en §5.

**Preguntas guía:**
- (*) ¿Cuál de los 8 valores `library`, `web-monolith`, `web-microservices`, `desktop-app`, `mobile-app-maui`, `rest-api`, `cli-tool`, `worker-service` describe mejor el artefacto principal entregable?
- (*) ¿Hay sub-proyectos secundarios que se publiquen como artefactos separados?
- ¿El tipo elegido va a evolucionar en el roadmap hacia otro tipo (por ejemplo, monolito a microservicios)?
- ¿Quién consume el artefacto principal: usuario final, otro sistema, otro desarrollador?

**Ejemplo aplicado (tabla con los 8 tipos D8):**

| Tipo | Documentos obligatorios | Documentos recomendados | Omitibles | Carpetas `/src` típicas | Contenido de `/samples` | Stack-hint orientativo |
|---|---|---|---|---|---|---|
| `library` | 00, 01, 02, 05, 06, 08, 09, 10, 11 | 03 (DX), 07 | 04 | `src/<Lib>.Core/`, `src/<Lib>.Abstractions/` | Consumidor mínimo, intermedio y avanzado que usa la librería vía package manager | C#/.NET, Java, Rust, Python, TypeScript |
| `web-monolith` | 00, 01, 02, 03, 05, 06, 07, 08, 09, 13 (NFR) | 04 | 11 si es interno | `src/Web/`, `src/Domain/`, `src/Infrastructure/` | App de demostración del frontend con datos seed | ASP.NET Core, Django, Rails, Spring Boot, Laravel |
| `web-microservices` | 00, 01, 02, 03, 05, 06, 07, 08, 09, 13 | 04 | — | `src/Services/<Servicio>/`, `src/Gateway/`, `src/BuildingBlocks/` | Compose con N servicios más un cliente de prueba | .NET + YARP, Go + gRPC, Java + Spring Cloud, Node + NestJS |
| `desktop-app` | 00, 01, 02, 03, 05, 06, 08, 09, 12 | 11 si hay plugins | — | `src/<App>.UI/`, `src/<App>.Core/` | Proyecto skin con tema custom o plugin de ejemplo | WPF/WinUI, Avalonia, Electron, Tauri, Qt |
| `mobile-app-maui` | 00, 01, 02, 03, 05, 06, 08, 09, 12 | 11 | — | `src/<App>.Mobile/`, `src/<App>.Shared/` | Proyecto demo multiplataforma con storyboard de pantallas | Flutter, React Native, .NET MAUI, Xamarin legacy, Kotlin Multiplatform |
| `rest-api` | 00, 01, 02, 05, 06, 07, 08, 09, 10, 13 | 03 (DX), 04 | 11 si solo es interno | `src/Api/`, `src/Application/`, `src/Domain/`, `src/Infrastructure/` | Cliente HTTP de referencia, Postman collection, SDK generado | ASP.NET Core, FastAPI, Express, Gin, Quarkus |
| `cli-tool` | 00, 01, 02, 05, 06, 08, 09, 10, 11, 12 | 03 | 04, 07 | `src/<Cli>.Cli/`, `src/<Cli>.Core/` | Recetas de uso por OS, scripts batch/bash de demostración | Cobra (Go), Click (Python), System.CommandLine (.NET), oclif (Node) |
| `worker-service` | 00, 01, 02, 05, 06, 08, 09, 13 | 04, 07 | 11 si no hay API | `src/Worker/`, `src/Domain/`, `src/Infrastructure/` | Compose con broker (RabbitMQ/Kafka) y productor de prueba | .NET Worker Service, Sidekiq, Celery, Temporal, Hangfire |

**Lo que NO va en esta sección:**
- Stack concreto, versiones, frameworks (eso va a §2).
- Patrones arquitectónicos como Clean Architecture o CQRS (eso va a §3).
- Estructura de carpetas (eso va a §5).

---

## §2 Stack tecnológico

**Instrucción:** Declarar lenguaje principal, versión, runtime, framework de aplicación y plataformas target. Incluir tabla de dependencias core (no transitivas) con su justificación. Excluir dependencias de desarrollo o testing (van a §9 o §11).

**Preguntas guía:**
- (*) ¿Cuál es la versión mínima del lenguaje y del runtime?
- (*) ¿Cuáles son las 5-10 dependencias core sin las cuales el proyecto no compila?
- ¿Hay restricciones del cliente sobre licencias (MIT/Apache, sin GPL)?
- ¿El stack permite cumplir los NFR declarados en §13?

**Ejemplo genérico (microservicio de pagos en Go):**
> Lenguaje: Go 1.23. Framework HTTP: Gin v1.10. Cliente DB: pgx v5 contra PostgreSQL 16. Mensajería: confluent-kafka-go v2 contra Kafka 3.7. Observabilidad: OpenTelemetry SDK + Prometheus client. Tracing distribuido: Jaeger vía OTLP. Plataforma target: Linux amd64/arm64 en contenedores Docker. Las dependencias se justifican así: pgx por su soporte nativo de tipos PostgreSQL, Gin por ser estándar de facto en el ecosistema, OpenTelemetry por requisito de observabilidad cross-servicio.

**Lo que NO va en esta sección:**
- Decisiones sobre patrones arquitectónicos (van a §3).
- Frameworks de testing (van a §9).
- Herramientas de CI (van a §11).

---

## §3 Estilo arquitectónico

**Instrucción:** Elegir entre `monolito`, `monolito modular`, `microservicios`, `libreria`, `pipeline`, `event-driven`, `clean architecture`. Justificar la elección contra al menos dos alternativas descartadas. La decisión se materializa luego como ADR-001 en el Sprint 0. La elección debe ser coherente con §1: por ejemplo, `library` implica usualmente clean architecture o pipeline, no microservicios.

**Preguntas guía:**
- (*) ¿Qué estilo arquitectónico se adopta y por qué?
- (*) ¿Qué dos alternativas se descartaron y por qué motivo?
- ¿El estilo elegido permite la escalabilidad del equipo prevista (número de devs en 6 meses)?
- ¿La complejidad operativa del estilo es proporcional al tamaño del problema?

**Ejemplo genérico (API de identidad, clean architecture):**
> Se adopta Clean Architecture con 4 capas concéntricas: Domain (entidades User, Tenant, Role; reglas de negocio puras), Application (commands/queries vía MediatR, validación vía FluentValidation), Infrastructure (EF Core + PostgreSQL, integraciones con SMTP y SMS gateway), API (controllers REST, autenticación JWT, OpenAPI spec). El dominio no depende de frameworks. Alternativas descartadas: monolito en capas tradicional (acopla lógica de negocio al ORM, dificulta los tests unitarios del dominio) y microservicios (over-engineering para un equipo de 4 personas con un solo dominio acotado).

| Factor | Monolito | Monolito modular | Microservicios | Librería | Pipeline | Event-driven | Clean architecture |
|---|---|---|---|---|---|---|---|
| Equipo recomendado | 1-5 | 3-15 | 10+ | 1-5 | 1-5 | 5+ | 3-15 |
| Deploy independiente | No | Deseable | Requerido | N/A | No | Por consumidor | No |
| Complejidad operativa | Baja | Media | Alta | Baja | Baja | Alta | Media |
| Time to market | Rápido | Medio | Lento | Medio | Rápido | Medio | Medio |
| Aplica con tipo D8 | web-monolith, rest-api | web-monolith, rest-api | web-microservices | library | library, cli-tool | worker-service, web-microservices | rest-api, web-monolith |

**Lo que NO va en esta sección:**
- Estructura concreta de carpetas (va a §5).
- Detalles de comunicación entre servicios (va a §6).
- Esquema de persistencia (va a §7).

---

## §4 Esquema de descomposición y delivery

**Instrucción:** Declarar la estrategia de descomposición del trabajo en el tiempo. Optar por vertical slicing, horizontal slicing, walking skeleton o thin slice / tracer bullet, y justificar. El criterio bloqueante es que el primer sprint entregue valor demostrable end-to-end (vertical slicing). Si la estrategia elegida no cumple ese criterio, redefinir.

**Preguntas guía:**
- (*) ¿El primer sprint entrega valor demostrable end-to-end?
- (*) ¿La descomposición es vertical (rebanadas funcionales) u horizontal (capas por sprint)?
- ¿Existe un walking skeleton planificado para Sprint 0/1?
- ¿Cómo se priorizan los slices: por riesgo arquitectónico, por valor de negocio o por simplicidad técnica?

**Ejemplo genérico (librería de parsing CSV, walking skeleton):**
> Se aplica vertical slicing desde el Sprint 1. El walking skeleton del Sprint 0 incluye: lectura de un archivo CSV mínimo, detección de delimitador fijo, deserialización a un POCO, devolución de un `IEnumerable<T>`. No incluye streaming, manejo de comillas escapadas, headers configurables ni tipos custom. Cada sprint posterior agrega profundidad al pipeline manteniendo la capacidad de leer un CSV end-to-end: Sprint 2 agrega quoting RFC 4180, Sprint 3 agrega streaming vía `IAsyncEnumerable`, Sprint 4 agrega type conversion configurable. Cada slice es demostrable con un test de aceptación público.

**Lo que NO va en esta sección:**
- Plan de sprint detallado con US/BT (eso va a `07_plan-sprint/`).
- Backlog de features (va a `06_backlog-tecnico/`).
- Decisiones de framework de testing (van a §9).

---

## §5 Estructura de repositorio propuesta

**Instrucción:** Proponer el árbol `tree` completo con `/src`, `/tests`, `/samples`, `/docs`, `/devs`. La estructura debe ser coherente con §1 (tipo de proyecto) y §3 (estilo arquitectónico). Marcar con comentario inline aquellas carpetas que aplican solo a ciertos tipos. Incluir obligatoriamente la subsección §5.X sobre `/samples`.

**Preguntas guía:**
- (*) ¿La estructura sigue las convenciones del ecosistema del lenguaje?
- (*) ¿Las carpetas reflejan el estilo arquitectónico de §3?
- ¿Hay separación clara entre código productivo (`/src`), tests (`/tests`) y samples (`/samples`)?
- ¿Existe un monorepo con sub-proyectos o un repositorio plano?

**Ejemplo genérico (app móvil de inventario, estructura repo):**
```text
inventario-app/
├── src/
│   ├── Inventario.App.Mobile/         # Proyecto móvil multiplataforma
│   ├── Inventario.Shared/             # Modelos, DTOs, validators compartidos
│   ├── Inventario.Domain/             # Entidades, reglas de negocio
│   └── Inventario.Infrastructure/     # SQLite local, sync HTTP
├── tests/
│   ├── Inventario.UnitTests/
│   ├── Inventario.IntegrationTests/
│   └── Inventario.UiTests/            # Appium / UI Testing del framework móvil
├── samples/
│   ├── 01-basico-lectura/             # Lista productos hardcodeados
│   ├── 02-intermedio-sync/            # Sync con API mock
│   └── 03-avanzado-offline/           # Modo offline + conflict resolution
├── docs/
│   ├── 00_contexto/
│   ├── 01_necesidades_negocio/
│   └── ... (categorías 00-11 SDD)
├── devs/
│   ├── intake/                        # PROJECT-BRIEF, PROJECT-README
│   └── decisiones-proyecto_v1.0.md
├── PROJECT-README-inventario-app_v1.0.md
└── README.md                          # Apuntador corto al PROJECT-README
```

### §5.X Materialización de `/samples` (alimenta `11_examples/`)

**Instrucción:** Describir cómo se materializan los samples según el tipo D8 elegido. Cada sample debe ser autocontenido, ejecutable y declarar su complejidad (básico, intermedio, avanzado).

| Tipo D8 | Qué hay en `/samples` | Vínculo con `/src` |
|---|---|---|
| `library` | 3 proyectos consumidores progresivos (hola mundo, caso común, caso avanzado con extensiones) | ProjectReference durante el desarrollo, PackageReference para validar la publicación |
| `web-monolith` | Datos seed, scripts de bootstrap, opcional skin custom | Mismo solution |
| `web-microservices` | Compose con todos los servicios + cliente de prueba (curl/Postman/Bruno) | Submódulos o monorepo |
| `desktop-app` | Plugin de demostración, tema custom | Plugin vía interfaz pública |
| `mobile-app-maui` | Proyecto demo con storyboard de pantallas y datos mock | Mismo solution o consumer externo |
| `rest-api` | SDK generado, cliente HTTP de referencia, Postman/Bruno collection | Cliente externo |
| `cli-tool` | Recetas de uso por OS (Windows/Linux/macOS), scripts de demostración | Consumer externo |
| `worker-service` | Productor de prueba, compose con broker | Compose paralelo |

**Lo que NO va en esta sección:**
- Contenido detallado de los samples (eso va a `11_examples/` durante la implementación).
- Decisiones de testing (van a §9).
- Convenciones de versionado de paquetes (van a §10).

---

## §6 Comunicación e integración

**Instrucción:** Solo aplica si el proyecto integra con otros sistemas o se compone de varios servicios. Declarar protocolo (REST/gRPC/WebSocket/MQ), formato de payload (JSON/Protobuf/Avro), versión de los contratos y política de breaking changes. Si no aplica (librería, CLI pura), declarar explícitamente "No aplica" con justificación.

**Preguntas guía:**
- (*) ¿Qué protocolos de comunicación se usan en flujos sincrónicos y asincrónicos?
- (*) ¿Cómo se versionan los contratos (URL, header, schema registry)?
- ¿Hay un schema registry para eventos?
- ¿Qué política se sigue ante breaking changes en contratos?

**Ejemplo genérico (microservicio de pagos, REST + Kafka):**
> Comunicación sincrónica: REST + JSON vía HTTPS. Esquema versionado por URL (`/v1/payments`). Documentado en OpenAPI 3.1. Comunicación asincrónica: eventos publicados en Kafka con Avro + Schema Registry (Confluent). Topics: `payments.requested`, `payments.completed`, `payments.failed`. Política de compatibilidad: backward-compatible obligatorio para cambios en minor; cambios incompatibles requieren topic nuevo `v2.payments.requested`. Retries con exponential backoff + DLQ tras 5 intentos. Idempotencia garantizada por `payment_id` como clave de evento.

**Sub-tabla de protocolos por escenario:**

| Escenario | Protocolo | Formato | Versionado |
|---|---|---|---|
| API pública de consumo externo | REST sobre HTTPS | JSON | URL (`/v1/...`) |
| API interna alta performance | gRPC sobre HTTP/2 | Protobuf | Paquete proto versionado |
| Notificación realtime al cliente | WebSocket / SSE | JSON | Header `X-API-Version` |
| Procesamiento asincrónico | Cola (RabbitMQ/SQS) o stream (Kafka) | Avro / JSON Schema | Schema Registry |
| Integración con tercero | Webhook saliente firmado | JSON + HMAC-SHA256 | Header `X-Webhook-Version` |

**Política de breaking changes (resumen):**
- Anunciar la deprecación en el changelog del consumidor con al menos una minor de anticipación.
- Mantener la versión anterior en paralelo durante un período de gracia mínimo (sugerido: 6 meses para APIs públicas, 3 meses para APIs internas).
- Documentar la guía de migración en el repositorio del productor.

**Lo que NO va en esta sección:**
- Esquema de la base de datos (va a §7).
- Política de autenticación del consumidor (va a §8).
- NFR de latencia (van a §13).

---

## §7 Persistencia

**Instrucción:** Declarar qué datos se guardan, dónde, cómo se versiona el esquema (migrations) y qué patrones de acceso se usan (Repository, Unit of Work, DAO, EventSourcing). Aclarar si el modelo es multi-tenant y cómo se aísla cada tenant. Si no hay persistencia (CLI sin estado, librería pura), declarar "No aplica".

**Preguntas guía:**
- (*) ¿Qué motor de persistencia se usa y por qué?
- (*) ¿Cómo se versiona el esquema (migrations automatizadas, scripts manuales)?
- ¿El sistema es multi-tenant? ¿Aislamiento por base, esquema o columna?
- ¿Qué patrón de acceso a datos se aplica?

**Ejemplo genérico (portal SaaS multi-tenant):**
> Motor: PostgreSQL 16 gestionado (RDS o Azure Database). Esquema versionado por migrations Flyway con archivos numerados `V001__init.sql`, `V002__add_users.sql`. Multi-tenant vía columna `tenant_id` en todas las tablas de dominio más query filter global en EF Core que se hidrata desde el claim `tenant_id` del JWT. Patrón Repository sobre EF Core, Unit of Work explícito por command handler. Índices compuestos `(tenant_id, <campo_consulta>)` en todas las tablas. Backups diarios con retención de 30 días, restore validado mensualmente.

**Sub-tabla de estrategias multi-tenant:**

| Estrategia | Aislamiento | Coste operativo | Cuándo aplicar |
|---|---|---|---|
| Database por tenant | Máximo | Alto (N motores que mantener) | Tenants enterprise con compliance estricto |
| Schema por tenant | Alto | Medio (un motor, N esquemas) | 10-100 tenants medianos |
| Columna `tenant_id` con query filter | Medio | Bajo (un motor, una BD) | SaaS B2B con muchos tenants chicos |
| Sin multi-tenancy | N/A | Mínimo | Producto single-tenant on-premise |

**Lo que NO va en esta sección:**
- Modelo de datos detallado (va a `docs/02_especificacion_funcional/modelo-datos/`).
- ADRs sobre elección de motor (van a `docs/05_arquitectura_tecnica/`).
- Estrategia de cifrado de datos (va a §8).

---

## §8 Seguridad y autenticación

**Instrucción:** Declarar mecanismos de autenticación, autorización y manejo de secretos. Especificar flujos OAuth/OIDC si aplica, formato de tokens, política de roles. Incluir tabla de decisión adaptada al tipo de cliente. Aclarar cómo se manejan los secretos en CI/CD (referenciable desde §11).

**Preguntas guía:**
- (*) ¿Qué mecanismo de autenticación se usa? ¿Dónde reside el Identity Provider?
- (*) ¿Dónde se almacenan los secretos en runtime y en CI/CD?
- ¿Qué política de roles/scopes se aplica?
- ¿Hay requisitos de compliance (GDPR, PCI, HIPAA) que afecten la seguridad?

**Ejemplo genérico (API de identidad, OAuth2 + OIDC):**

| Tipo de cliente | Flujo recomendado | Token |
|---|---|---|
| SPA navegador | Authorization Code + PKCE | JWT corto (15 min) + refresh token rotativo |
| App móvil nativa | Authorization Code + PKCE + deep links | JWT corto + refresh con device-binding |
| Servicio a servicio | Client Credentials | JWT con scope acotado, sin refresh |
| CLI administrativo | Device Code | JWT con scope amplio, MFA obligatorio |

> Identity Provider: Keycloak self-hosted o Auth0 según ambiente. Tokens JWT firmados RS256, claims `sub`, `tenant_id`, `roles`. Roles modelados como `tenant:role` (por ejemplo, `acme:admin`). Secretos en runtime: HashiCorp Vault con dynamic credentials. Secretos en CI: GitHub Secrets con rotación cada 90 días. MFA obligatorio para roles admin. Auditoría de accesos a `audit_log` con retención de 1 año.

**Sub-tabla de manejo de secretos:**

| Entorno | Backend | Mecanismo | Rotación |
|---|---|---|---|
| Desarrollo local | `.env` fuera de git + `.env.example` versionado | Variables de entorno | Manual |
| CI/CD | Secret store de la plataforma (GitHub Secrets, Azure Pipelines Library, GitLab CI Variables) | Inyección en runtime del job | 90 días |
| Staging / Producción | Vault gestionado (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault) | Referencias por nombre + IAM | 30-90 días o dynamic credentials |
| Aplicación runtime | Cliente del vault con cache TTL bajo | API o sidecar | Refresco automático |

**Lo que NO va en esta sección:**
- Protocolos de comunicación (van a §6).
- Pipeline de despliegue (va a §11).
- NFR de disponibilidad (van a §13).

---

## §9 Estrategia de testing

**Instrucción:** Declarar la pirámide de testing deseada con porcentajes orientativos, cobertura mínima numérica (gate del CI), frameworks por nivel y si se aplica BDD/ATDD. La cobertura mínima es bloqueante: debe ser numérica, no "alta" o "razonable".

**Preguntas guía:**
- (*) ¿Cuál es la cobertura mínima de líneas y de branches?
- (*) ¿Qué frameworks se usan en cada nivel de la pirámide?
- ¿Se aplica BDD/ATDD con escenarios Gherkin?
- ¿Cómo se manejan los tests de contrato entre servicios o consumidores?

**Ejemplo genérico (librería utilitaria, pirámide 80/15/5):**
> Pirámide: 80 por ciento tests unitarios, 15 por ciento tests de integración contra fixtures, 5 por ciento snapshot tests sobre output canónico. Cobertura mínima de líneas: 85 por ciento. Cobertura mínima de branches: 75 por ciento. Frameworks: xUnit + FluentAssertions para unitarios, Verify para snapshots, Testcontainers cuando se necesita un servicio real. BDD no aplica (librería sin actor de negocio). Tests de contrato públicos vía golden files versionados en `tests/golden/`. Mutation testing con Stryker en CI nocturno con umbral del 70 por ciento de mutantes muertos.

**Lo que NO va en esta sección:**
- Definition of Done (va a `docs/08_calidad_y_pruebas/definition-of-done_v1.0.md`).
- Quality gates del pipeline (van a §11).
- Plan de pruebas exploratorias (va a `docs/08_calidad_y_pruebas/plan-pruebas_v1.0.md`).

---

## §10 Estrategia de versionado y release

**Instrucción:** Adoptar SemVer 2.0.0 y Conventional Commits. Declarar herramienta de cálculo automático de versión (MinVer, GitVersion, semantic-release, release-please), estrategia de branching, canales preview/stable y feed de paquetes. Esta sección alimenta directamente §11.

**Preguntas guía:**
- (*) ¿Se adopta SemVer 2.0.0 y Conventional Commits sin excepciones?
- (*) ¿Qué herramienta calcula la versión y a partir de qué (tags, commits)?
- ¿Qué canales de release existen (alpha, beta, rc, stable)?
- ¿Dónde se publican los artefactos?

**Ejemplo genérico (librería de parsing, SemVer + MinVer):**
> Versionado: SemVer 2.0.0. Commits: Conventional Commits validados con commitlint en pre-commit hook. Herramienta de cálculo: MinVer sobre tags `git`. Branching: trunk-based con feature branches cortos (vida útil máxima 3 días). Canales: `0.x.y` pre-release durante desarrollo, `1.0.0` a partir del primer release público estable. Pre-releases: `1.2.0-alpha.5`, `1.2.0-rc.1`. Feed: registry de paquetes público (según ecosistema del lenguaje) para releases estables; registry privado del equipo para pre-releases. CHANGELOG.md generado por release-please en cada merge a `main`.

**Sub-tabla de reglas de bump SemVer:**

| Cambio | Bump | Ejemplo |
|---|---|---|
| Bug fix sin cambio de API | Patch | 1.3.0 -> 1.3.1 |
| Feature backward-compatible | Minor | 1.3.0 -> 1.4.0 |
| Breaking change | Major | 1.x -> 2.0.0 |
| Pre-release iterativo | Suffix | 1.4.0-alpha.1 -> 1.4.0-alpha.2 |
| Promoción a estable | Quitar suffix | 1.4.0-rc.3 -> 1.4.0 |

**Lo que NO va en esta sección:**
- Comandos específicos del pipeline (van a §11).
- Compatibilidad de versiones consumidoras (va a §12).
- Política de breaking changes detallada (queda como sub-tabla aquí o ADR).

---

## §11 Pipeline CI/CD

**Instrucción:** Declarar stages del pipeline (build, test, lint, SCA, SBOM, firma, publicación), matriz de SO y runtime, ambientes (DEV/QA/STAGING/PROD o preview/stable). Cada stage debe tener un quality gate explícito. La cobertura mínima del CI debe coincidir con §9.

**Preguntas guía:**
- (*) ¿Qué plataforma de CI se usa (GitHub Actions, Azure DevOps, GitLab CI)?
- (*) ¿Cuáles son los quality gates bloqueantes para mergear a `main`?
- ¿Se genera SBOM y se firma el artefacto?
- ¿Cómo se hace rollback de una publicación problemática?

**Ejemplo genérico (microservicio, build/test/SBOM/firma):**
> Plataforma: GitHub Actions. Trigger CI: pull_request a `main`. Stages: restore, build (Release), test con cobertura, lint (Semgrep + reviewdog), SCA (Trivy + Dependabot), SBOM (CycloneDX vía syft), build de imagen Docker multi-stage, escaneo de imagen (Trivy), firma de imagen (cosign con keyless OIDC), push a registry interno. Gates bloqueantes: 0 tests fallidos, cobertura mayor o igual al 80 por ciento, 0 vulnerabilidades críticas en dependencias, 0 vulnerabilidades altas en imagen. Trigger CD: push de tag `v*`. Ambientes: dev (auto en cada merge), staging (auto en cada tag pre-release), prod (manual approval). Rollback: retag de la imagen anterior + redeploy vía Argo CD; toma menos de 5 minutos.

| Stage | Tool sugerida | Quality gate |
|---|---|---|
| Build | Compilador del lenguaje | 0 errores, 0 warnings nuevos |
| Test | Framework de §9 | Cobertura ≥ [X]% |
| Lint | Semgrep / linters nativos | 0 issues high/critical |
| SCA | Trivy / Snyk / Dependabot | 0 CVEs critical/high |
| SBOM | syft / CycloneDX | SBOM adjunto al artefacto |
| Firma | cosign / GPG | Firma verificable públicamente |
| Publish | Registry de paquetes del ecosistema (npm, Maven, Crates, PyPI) o registry de imágenes (Docker Hub, GHCR) | Solo en tags SemVer válidos |

**Matriz de ambientes sugerida (adaptar al proyecto):**

| Ambiente | Trigger | Aprobación | Datos | Acceso |
|---|---|---|---|---|
| DEV | merge a `main` | Automático | Datos sintéticos | Equipo |
| QA | merge a `main` o tag pre-release | Automático | Mix sintético + anonimizado | Equipo + QA |
| STAGING | tag pre-release (`*-rc.*`) | Automático con humo manual | Anonimizado de producción | Equipo + stakeholders |
| PROD | tag estable (`v1.2.3`) | Manual con cambio aprobado | Reales | Operaciones |

**Lo que NO va en esta sección:**
- Detalles de la herramienta de versionado (va a §10).
- Definición de DoR/DoD (va a `docs/06_backlog-tecnico/` y `docs/08_calidad_y_pruebas/`).
- Políticas de IAM del cloud provider (van a `docs/09_devops/` operativo).

---

## §12 Compatibilidad y plataformas target

**Instrucción:** Declarar SO soportados, runtimes, navegadores, dispositivos y versiones mínimas. Coherente con §1: una librería declara TFMs soportados, una app móvil declara versiones de iOS/Android, un servicio declara distribuciones Linux. Toda combinación no listada se considera no soportada.

**Preguntas guía:**
- (*) ¿Cuáles son las plataformas target soportadas?
- (*) ¿Cuál es la versión mínima de cada runtime/SO/navegador?
- ¿Hay arquitecturas que NO se soportan (32-bit, ARM antiguo)?
- ¿Existe matriz de compatibilidad con versiones consumidoras?

**Ejemplo genérico (CLI de migración, multi-OS):**
> SO soportados: Windows 10 1809 o superior, Windows Server 2019+, Ubuntu 22.04 o superior, Debian 12, macOS 13 o superior (Intel y Apple Silicon). Arquitecturas: x64 y arm64. Runtime: el binario se entrega como single-file self-contained AOT, sin runtime preinstalado. No se soporta x86 32-bit ni distribuciones musl-based salvo Alpine 3.18+. Política de soporte: se mantiene la versión actual y la anterior (N y N-1) durante 12 meses post-release.

**Lo que NO va en esta sección:**
- Stack interno (va a §2).
- Política de versionado del propio paquete (va a §10).
- NFR de performance (van a §13).

---

## §13 Requerimientos no funcionales (NFR)

**Instrucción:** Declarar métricas numéricas concretas para performance (latencia p99, throughput), escalabilidad, disponibilidad, observabilidad y compliance. Cada métrica debe ser medible y auditable. NFR vagos como "rápido" o "escalable" no son aceptables.

**Preguntas guía:**
- (*) ¿Cuál es la latencia objetivo p99 y el throughput mínimo?
- (*) ¿Cuál es el SLO de disponibilidad?
- ¿Qué se loguea, qué se mide y qué se traza? ¿En qué stack de observabilidad?
- ¿Hay requisitos regulatorios (GDPR/PCI/fiscal/HIPAA)?

**Ejemplo genérico (worker de procesamiento de imágenes, NFR):**
> Performance: tiempo de procesamiento p99 menor o igual a 2 segundos por imagen de hasta 10 MB. Throughput sostenido: 500 imágenes por minuto por instancia. Escalabilidad horizontal: el worker es stateless, escala lineal hasta 20 réplicas validado en pruebas de carga. Disponibilidad: SLO del 99,5 por ciento mensual (descontando ventanas de mantenimiento programadas). Observabilidad: logs estructurados JSON enviados a Loki, métricas en Prometheus (jobs procesados, fallos, latencia por etapa), traces OpenTelemetry hacia Tempo. Dashboards Grafana versionados como código. Compliance: borrado completo de imágenes originales a los 7 días post-procesamiento por requisito GDPR. Auditoría de accesos por usuario y por tenant.

**Lo que NO va en esta sección:**
- Pipeline operativo (va a §11).
- Decisiones de hosting (van a `docs/09_devops/`).
- Modelo de datos (va a `docs/02_especificacion_funcional/`).

---

## §14 Estrategia de demo / samples (alimenta `11_examples`)

**Instrucción:** Detallar qué aplicaciones de ejemplo se construyen en `/samples`, qué demuestra cada una, su nivel de complejidad (básico, intermedio, avanzado) y cómo se vinculan a `/src` (ProjectReference, PackageReference, consumer externo). Cada sample debe ser autocontenido, reproducible en un entorno limpio en 5 pasos o menos.

**Preguntas guía:**
- (*) ¿Cuántos samples se construyen y qué progresión de complejidad cubren?
- (*) ¿Cómo se vincula cada sample con el código productivo?
- ¿Cada sample documenta su resultado esperado?
- ¿Hay un sample que demuestre el punto de extensión principal del sistema?

**Ejemplo genérico (app móvil de inventario, samples):**
> Tres samples progresivos en `/samples`:
> 1. `01-basico-lectura/` (nivel básico): app móvil mínima que lista 10 productos hardcodeados en memoria, sin sync ni autenticación. Demuestra el patrón MVVM, el binding de `CollectionView` y la inyección de un repositorio in-memory. Tiempo de setup: 5 minutos.
> 2. `02-intermedio-sync/` (nivel intermedio): agrega sync con un mock server (json-server). Demuestra HttpClient tipado, manejo de estados de loading/error, refresh-pull. Requiere `dotnet run` del mock antes.
> 3. `03-avanzado-offline/` (nivel avanzado): modo offline-first con SQLite local, queue de cambios pendientes, resolución de conflictos last-write-wins. Demuestra el patrón Repository con dos backends, el uso de `Connectivity` plugin y la sincronización en background. Cubre el punto de extensión `ISyncConflictResolver`.
>
> Los tres samples referencian el código de `/src` por ProjectReference durante el desarrollo y se reconfiguran a PackageReference automáticamente en el job de validación de release.

**Lo que NO va en esta sección:**
- Tutoriales conceptuales (van a `docs/10_developer_guide/`).
- Casos de uso de negocio (van a `docs/02_especificacion_funcional/casos-de-uso/`).
- Especificación detallada de la API pública (va a `docs/05_arquitectura_tecnica/contratos*`).

---

## §15 Decisiones técnicas pre-tomadas (pre-ADR)

**Instrucción:** Listar decisiones técnicas cerradas antes del Sprint 0 con justificación. Estas decisiones se convierten formalmente en ADRs aceptados durante Sprint 0 (uno por decisión, archivos `ADR-XX-<kebab>_v1.0.md` en `docs/05_arquitectura_tecnica/`). Esta sección no reemplaza al ADR formal, lo prefigura.

**Preguntas guía:**
- (*) ¿Cuáles son las decisiones cerradas antes del Sprint 0 y por qué?
- (*) ¿Qué alternativas se evaluaron en cada decisión?
- ¿Están documentadas las consecuencias positivas y negativas de cada decisión?
- ¿Qué decisiones quedan deliberadamente abiertas para Sprint 0?

**Ejemplo genérico (API de identidad, pre-ADRs):**

| ID provisional | Decisión | Elegido | Alternativas descartadas | Motivo principal |
|---|---|---|---|---|
| pre-ADR-01 | Identity Provider | Keycloak self-hosted | Auth0 (costo a escala), IdentityServer Duende (licencia comercial), implementación propia (costo de mantenimiento) | Open source con soporte comunitario y feature parity con los SaaS |
| pre-ADR-02 | Algoritmo de firma JWT | RS256 con rotación de claves | HS256 (clave compartida no escala), ES256 (menor adopción en clientes legacy) | RS256 permite rotar claves sin coordinar con consumidores |
| pre-ADR-03 | Storage de usuarios | PostgreSQL gestionado | LDAP (rígido), MongoDB (sin transacciones fuertes en sus inicios) | Modelo relacional encaja con tenants/roles, transacciones ACID |
| pre-ADR-04 | Lenguaje | Go 1.23 | Java/Spring (más verbosidad), C# (familiaridad del equipo es media) | Footprint bajo en runtime y tiempo de arranque para escalado |

**Lo que NO va en esta sección:**
- ADRs formales (esos viven en `docs/05_arquitectura_tecnica/ADR-XX-*_v1.0.md`).
- Decisiones abiertas (van al backlog del Sprint 0).
- Justificaciones de stack básicas (van a §2).

---

## §16 Restricciones técnicas y trade-offs aceptados

**Instrucción:** Documentar a qué se renuncia conscientemente para ganar otra cosa. Esta sección evita reabrir discusiones cerradas y documenta el costo aceptado de las decisiones tomadas.

**Preguntas guía:**
- (*) ¿Qué ganancias se priorizaron y a costa de qué renuncias?
- (*) ¿Hay restricciones impuestas por el cliente o el ecosistema?
- ¿Qué features deliberadamente NO se construyen en esta versión?
- ¿Qué tipo de cargas de trabajo el sistema no soporta?

**Ejemplo genérico (librería, trade-offs):**
> Trade-offs aceptados conscientemente:
> 1. Se prioriza ergonomía de API pública sobre extensibilidad máxima: el público objetivo son desarrolladores intermedios, no implementadores de plugins. Consecuencia: agregar plugins de terceros requerirá un cambio mayor (v2.0).
> 2. Se prioriza compatibilidad con la versión LTS del runtime sobre adopción de features modernas. Consecuencia: no se usan features de preview del lenguaje aunque acortarían código.
> 3. Se prioriza coverage de tests sobre tiempo de ejecución del pipeline. Consecuencia: el CI completo demora 8-10 minutos.
> 4. No se soporta arquitectura 32-bit. Consecuencia: clientes con runtimes legacy quedan fuera.
> 5. No se entrega sincrónicamente en formato AOT en la v1; solo JIT. Consecuencia: arranque en frío mayor a 1 segundo en escenarios serverless.

**Lo que NO va en esta sección:**
- Decisiones positivas (esas están en §15).
- Plan de features futuras (va a `docs/00_contexto/roadmap-producto_v1.0.md`).
- NFR cuantitativos (van a §13).

---

## §17 Checklist de completitud técnica

Verificar antes de marcar el documento como Aprobado. Todos los ítems deben estar tildados.

- [ ] Tipo de proyecto seleccionado entre los 8 valores de D8 (§1).
- [ ] Tabla de implicancias por tipo verificada contra el tipo elegido (§1).
- [ ] Stack tecnológico declara lenguaje, versión, runtime y dependencias core con justificación (§2).
- [ ] Estilo arquitectónico elegido y justificado contra al menos dos alternativas descartadas (§3).
- [ ] Estrategia de descomposición garantiza valor demostrable end-to-end en el primer sprint (§4).
- [ ] Estructura de repositorio publicada como árbol `tree` completo y coherente con tipo y estilo (§5).
- [ ] Subsección §5.X describe materialización de `/samples` según tipo D8.
- [ ] Estrategia de testing declara cobertura mínima numérica de líneas y de branches (§9).
- [ ] Estrategia de versionado adopta SemVer 2.0.0 y Conventional Commits con herramienta automatizada (§10).
- [ ] Pipeline CI/CD enumera stages con quality gates explícitos y bloqueantes (§11).
- [ ] Compatibilidad declara SO, runtimes y versiones mínimas sin ambigüedad (§12).
- [ ] NFR expresados con métricas numéricas (latencia p99, throughput, SLO de disponibilidad) (§13).
- [ ] Estructura de `/samples` describe al menos un sample por nivel de complejidad (§14).
- [ ] Decisiones pre-ADR de §15 tienen justificación documentada y alternativas evaluadas.
- [ ] Trade-offs aceptados documentados con su contrapartida (§16).
- [ ] Control de cambios actualizado con la versión y fecha del documento.

---

## Trazabilidad downstream

Este documento alimenta las siguientes secciones SDD 2.0:

| Sección del README | Categoría SDD destino | Documento downstream típico |
|---|---|---|
| §1 Tipo de proyecto | Todas las categorías 00-11 | Selector de variantes de especialidad |
| §2 Stack tecnológico | `05_arquitectura_tecnica/` | `arquitectura-solucion_v1.0.md` |
| §3 Estilo arquitectónico | `05_arquitectura_tecnica/` | `ADR-001-estilo-arquitectonico_v1.0.md` |
| §4 Descomposición y delivery | `07_plan-sprint/` | `plan-iteracion_sprint-01_v1.0.md` |
| §5 Estructura de repositorio | `05_arquitectura_tecnica/`, `10_developer_guide/` | `arquitectura-solucion_v1.0.md`, README de carpeta |
| §6 Comunicación e integración | `05_arquitectura_tecnica/` | `contratos_v1.0.md`, OpenAPI/Proto |
| §7 Persistencia | `02_especificacion_funcional/modelo-datos/`, `05_arquitectura_tecnica/` | `modelo-datos-logico_v1.0.md` |
| §8 Seguridad | `09_devops/`, `05_arquitectura_tecnica/` | `politica-seguridad_v1.0.md` |
| §9 Testing | `08_calidad_y_pruebas/` | `estrategia-testing_v1.0.md`, `definition-of-done_v1.0.md` |
| §10 Versionado y release | `09_devops/` | `estrategia-versionado_v1.0.md` |
| §11 Pipeline CI/CD | `09_devops/` | `pipeline-ci-cd_v1.0.md`, `entornos-deploy_v1.0.md` |
| §12 Compatibilidad | `00_contexto/` | `compatibilidad-plataformas_v1.0.md` |
| §13 NFR | `00_contexto/`, `08_calidad_y_pruebas/` | `requerimientos-no-funcionales_v1.0.md` |
| §14 Samples | `11_examples/` | `README.md` y `ejemplo-01..03_v1.0.md` |
| §15 Pre-ADR | `05_arquitectura_tecnica/` | `ADR-XX-<kebab>_v1.0.md` por decisión |
| §16 Restricciones | `00_contexto/` | `restricciones-y-tradeoffs_v1.0.md` |

---

## Control de cambios

| Version | Fecha | Cambios | Autor |
|---|---|---|---|
| 1.0 | [YYYY-MM-DD] | Plantilla inicial generada durante bootstrap SDD 2.0 | Bootstrap SDD 2.0 |
