# DevOps — Pipeline, Versionado, Publicación

**Archivo:** README.md  
**Versión:** 1.0  
**Fecha:** 2026-04-25  
**Autor / Owner:** DevOps / Release Engineering  
**Estado:** Aprobado

---

Esta sección consolida la operación de entrega del Motor DSL: cómo se construyen los artefactos, cómo se versionan, en qué entornos viven y cómo se publican y consumen como paquetes NuGet en GitHub Packages. Es la autoridad operativa para release engineering del producto y la fuente de verdad para las decisiones D1–D5 listadas más abajo.

El alcance abarca pipeline CI/CD (GitHub Actions), estrategia de versionado (SemVer 2.0.0 + Conventional Commits + MinVer), modelo de distribución (feeds Preview/Stable + consumers downstream), y empaquetado/distribución de los cuatro paquetes publicables (`MotorDsl.Core`, `MotorDsl.Parser`, `MotorDsl.Rendering`, `MotorDsl.Extensions`). Quedan fuera: gates funcionales de calidad (sección 08), arquitectura interna del motor (sección 05), y documentación de consumo orientada a integradores (sección 10).

---

## Audiencia

| Rol | Uso principal |
|---|---|
| Release manager | Procedimiento operativo de release, tagging, publicación, comunicación, yanking. |
| Mantenedores del paquete | Mapeo de cambios a SemVer, política de breaking changes, deprecaciones. |
| Integradores / consumidores externos | Configuración de feed, instalación, ventana de soporte. |
| DevOps / Plataforma | Workflows, secretos, runners, observabilidad del pipeline. |
| Tech Lead | Aprobación de releases con breaking changes, decisiones de soporte y yanking. |
| Security | Aprobación de yanking ante CVE crítico. |

---

## Documentos de la sección

| # | Documento | Propósito | Cuándo consultarlo |
|---|---|---|---|
| 1 | [pipeline-ci-cd_v1.0.md](pipeline-ci-cd_v1.0.md) | Define triggers, etapas, gates de calidad, secretos y workflows de GitHub Actions. | Antes de modificar workflows, agregar etapas, ajustar reglas de fail del pipeline o diagnosticar runs rotos. |
| 2 | [estrategia-versionado_v1.0.md](estrategia-versionado_v1.0.md) | Especifica SemVer 2.0.0, Conventional Commits, MinVer, GitHub Flow, política de breaking changes, deprecaciones, soporte y yanking. | Antes de cada release, al evaluar el bump de un PR, al planificar la deprecación de una API o al gestionar un CVE. |
| 3 | [entornos-deploy_v1.0.md](entornos-deploy_v1.0.md) | Define el modelo de distribución (feeds Preview/Stable + consumers downstream) y los entornos DEV/QA/STAGING/PROD aplicados a las sample apps consumers, con reglas de promoción Preview→Stable. | Al planificar la promoción de un cambio entre canales del feed o al revisar la configuración por entorno de una sample app consumer. |
| 4 | [guia-publicacion-nuget_v1.0.md](guia-publicacion-nuget_v1.0.md) | Procedimiento operativo para empaquetar (`dotnet pack`), publicar en GitHub Packages y consumir desde un proyecto .NET. | Al configurar metadata de paquetes, publicar manualmente, configurar el feed en un proyecto consumidor o resolver errores de instalación. |

---

## Orden de lectura recomendado

1. **`estrategia-versionado_v1.0.md`** primero. Establece el lenguaje común (SemVer, Conventional Commits, MinVer, GitHub Flow) que aparece referenciado por los otros tres documentos. Sin este marco, el pipeline y la guía NuGet pierden contexto.
2. **`pipeline-ci-cd_v1.0.md`** después. Concreta la estrategia de versionado en workflows ejecutables y define los gates que un cambio debe atravesar antes de llegar al feed.
3. **`entornos-deploy_v1.0.md`** en tercer lugar. Posiciona el resultado del pipeline en el modelo de feeds y consumers, y formaliza la regla de promoción `tag v* → publicación al feed Stable`.
4. **`guia-publicacion-nuget_v1.0.md`** al final. Es el documento más operativo: asume conocidas las reglas de versionado, los gates del pipeline y la topología de entornos para detallar el día a día de empaquetado y consumo.

Quien sólo necesite consumir los paquetes desde un proyecto puede ir directo al §7 de la guía NuGet. Quien deba ejecutar un release lee versionado §15 y guía NuGet §6 en ese orden.

---

## Glosario DevOps

| Término | Definición operativa |
|---|---|
| **CI** (Continuous Integration) | Práctica de mergear cambios a `main` con frecuencia ejecutando build, tests y análisis automático en cada PR. En este repo: workflow `ci.yml`. |
| **CD** (Continuous Delivery / Deployment) | Automatización de empaquetado y publicación de artefactos versionados. En este repo: workflows `cd-nuget.yml`, `cd-android.yml`, `cd-ios-*.yml`. |
| **SemVer** | Semantic Versioning 2.0.0. Esquema `MAJOR.MINOR.PATCH[-PRERELEASE][+BUILDMETADATA]` con reglas estrictas de incremento según el tipo de cambio. |
| **GitHub Flow** | Estrategia de branching: `main` siempre liberable, ramas `feature/*` cortas, PR obligatorio, squash merge, sin `develop` ni `release/*`. |
| **Conventional Commits** | Convención de mensajes `<tipo>(<scope>)!: <descripción>` que mapea automáticamente a bumps SemVer (`feat`→MINOR, `fix`→PATCH, `!`/`BREAKING CHANGE:`→MAJOR). |
| **MinVer** | Herramienta .NET que calcula la versión del paquete a partir de Git tags durante el build. Determinista, sin servidor, sin commits de bump. |
| **Tag firmado** | Tag Git creado con `git tag -s` y verificado con clave GPG. Requerido en releases producción para trazabilidad criptográfica. |
| **`.nupkg`** | Archivo de paquete NuGet (ZIP con DLLs, metadata y manifiesto `.nuspec`). Producto de `dotnet pack`. |
| **`.snupkg`** | Paquete de símbolos NuGet (debugging symbols, `.pdb`). Complementa el `.nupkg` para depuración por consumidores. Publicado junto al `.nupkg` desde pipeline v1.3 vía `<SymbolPackageFormat>snupkg</SymbolPackageFormat>`. |
| **Source Link** | Mecanismo de Microsoft que incrusta en el `.nupkg` referencias a la fuente en GitHub para que un debugger pueda saltar al código original. Implementado desde pipeline v1.3 vía `Microsoft.SourceLink.GitHub`. |
| **SBOM** (Software Bill of Materials) | Inventario formal de componentes y dependencias del paquete. Generado en formato CycloneDX por cada release desde pipeline v1.3 (`Microsoft.Sbom.Tool`); cumple SLSA Level 2 y NIST SSDF PS.3. |
| **SCA** (Software Composition Analysis) | Escaneo automático de dependencias para detectar vulnerabilidades conocidas (CVEs). Implementado desde pipeline v1.3 mediante Dependabot + `dotnet list package --vulnerable --include-transitive` con gate bloqueante de severidad ≥ High. |
| **SAST** (Static Application Security Testing) | Análisis estático de seguridad sobre el código fuente. Cubierto parcialmente desde pipeline v1.3 vía Roslyn analyzers (`Microsoft.CodeAnalysis.NetAnalyzers`) con `TreatWarningsAsErrors`. SAST formal dedicado queda fuera de alcance. |
| **Yanking** | Eliminación de una versión publicada del feed. En GitHub Packages es un delete destructivo que rompe lockfiles de consumidores. Sólo procede ante CVE crítico (CVSS ≥ 9.0) con versión de reemplazo disponible. |
| **Unlisting** | Semántica de NuGet.org (versión oculta pero instalable). No aplica en GitHub Packages, que sólo soporta delete. Documentado por consistencia conceptual. |
| **TFM** (Target Framework Moniker) | Identificador del framework destino del proyecto (`net10.0`). Decisión D4: TFM único, sin multi-targeting. |
| **Squash merge** | Modalidad de merge que colapsa todos los commits de una rama feature en uno solo en `main`. Obligatorio en este repo para mantener historia linear y commit-message-as-contract. |
| **Inmutabilidad de versión** | Una vez publicada, una versión `X.Y.Z` no se sobreescribe ni se reemplaza. Cualquier corrección requiere una nueva versión. Aplica a tags, paquetes y entradas de CHANGELOG. |
| **Diamond dependency** | Conflicto que ocurre cuando dos dependencias del consumidor requieren versiones MAJOR distintas de un mismo paquete transitivo. Por eso un bump MAJOR de dependencia interna es siempre breaking. |
| **Pre-release** | Versión inestable previa al estable, identificada por sufijo `-alpha.N`, `-beta.N` o `-rc.N`. Tiene precedencia menor que la versión sin sufijo. |
| **Build metadata** | Sufijo `+identificadores` (ej. `+sha.a1b2c3d`) que aporta trazabilidad pero no afecta precedencia ni resolución de versiones. |
| **GITHUB_TOKEN** | Token efímero provisto automáticamente por GitHub Actions a cada workflow run. Para publicar a GitHub Packages requiere permiso explícito `packages: write`. No se rota manualmente; expira al finalizar el run. |
| **PAT** (Personal Access Token) | Token de usuario GitHub usado por consumidores para autenticar contra el feed. Scope mínimo `read:packages` para consumo, `write:packages` para publicación manual (restringido a release manager). |
| **Workflow dispatch** | Trigger manual de un workflow desde la UI o API de GitHub Actions con inputs parametrizados. Usado para publicaciones manuales con versión explícita. |
| **Artifact** | Archivo producido por un workflow run y persistido temporalmente en GitHub Actions (retención configurable). Distinto del paquete publicado al feed: el artifact es interno al run. |
| **Inmutabilidad de tag** | Política de no reescribir un tag Git una vez creado y pusheado. Reforzada por `protected tags` en GitHub. Recrear un tag requiere bump de versión. |

---

## Stack y herramientas

| Categoría | Herramienta | Rol |
|---|---|---|
| CI/CD | **GitHub Actions** | Ejecutor de los 5 workflows (`ci.yml`, `cd-nuget.yml`, `cd-android.yml`, `cd-ios-sampleapp.yml`, `cd-ios-multaapp.yml`). |
| Feed de paquetes | **GitHub Packages** (NuGet) | Único destino de publicación (D1). URL: `https://nuget.pkg.github.com/<owner>/index.json`. |
| Build .NET | **.NET 10 SDK** | Build, test, pack. TFM único `net10.0` (D4). |
| Versionado | **MinVer 6.0.0** | Cálculo de versión desde Git tags. Configurado en `src/Directory.Build.props`. |
| Convención de commits | **Conventional Commits 1.0.0** | Determina el bump SemVer en cada squash merge. |
| Changelog | **Keep a Changelog 1.1.0** | Formato del `CHANGELOG.md`. |
| Análisis de calidad | Análisis estático integrado en `ci.yml` | Code smells, naming, validación de arquitectura. |
| Gestión de dependencias | **Dependabot** (recomendado) | PRs automáticos de bumps de dependencias NuGet. Categorizadas vía Conventional Commits con scope `deps`. |
| Firma de tags | **GPG** | Tags firmados con `git tag -s` requeridos en releases producción. |
| Runners | `ubuntu-latest` (build/tests/Android/NuGet), `macos-15` con Xcode 26.0 (iOS) | Hosted runners de GitHub. |
| Validación end-to-end | `samples/MotorDsl.MultaApp.Nuget` | Smoke test de integración: app MAUI consumiendo los 4 paquetes desde GitHub Packages post-release. |

Herramientas implementadas desde pipeline v1.3: firma de paquetes (`dotnet nuget sign`), SBOM CycloneDX, SCA (Dependabot + auditoría en CI), Source Link, símbolos `.snupkg`, matriz OS multi-runner, observabilidad con métricas DORA. Fuera de alcance: SAST formal dedicado, publicación multi-feed, AAB para Google Play, keystore propio para APK.

### Workflows y triggers

| Workflow | Archivo | Trigger | Runner | Propósito |
|---|---|---|---|---|
| CI | `.github/workflows/ci.yml` | PR + push a `main` | `ubuntu-latest` | Build, 185 tests, análisis de calidad. Gate de merge. |
| CD NuGet | `.github/workflows/cd-nuget.yml` | Tag `v*` + `workflow_dispatch` | `ubuntu-latest` | Publicación de los 4 paquetes a GitHub Packages. |
| CD Android | `.github/workflows/cd-android.yml` | Push `main` + tag `v*` | `ubuntu-latest` | APK de SampleApp y MultaApp como artifacts (retención 7 días). |
| CD iOS SampleApp | `.github/workflows/cd-ios-sampleapp.yml` | Push `main` (paths filtrados) | `macos-15`, Xcode 26.0 | `.app.zip` para `iossimulator-arm64` (retención 1 día). |
| CD iOS MultaApp | `.github/workflows/cd-ios-multaapp.yml` | Push `main` (paths filtrados) | `macos-15`, Xcode 26.0 | `.app.zip` para `iossimulator-arm64` (retención 1 día). |

---

## Decisiones arquitectónicas DevOps

| ID | Decisión | Justificación operativa | Documento autoritativo |
|---|---|---|---|
| **D1** | GitHub Packages como único feed | Reduce superficie operativa, integra autenticación con GitHub identity, evita conflictos de naming con NuGet.org público. | [`guia-publicacion-nuget_v1.0.md`](guia-publicacion-nuget_v1.0.md) §6.5; [`entornos-deploy_v1.0.md`](entornos-deploy_v1.0.md) §4 |
| **D2** | Licencia MIT | Maximiza reutilización académica e industrial; declarada vía `<PackageLicenseExpression>MIT</PackageLicenseExpression>`. | [`guia-publicacion-nuget_v1.0.md`](guia-publicacion-nuget_v1.0.md) §5.2 |
| **D3** | GitHub Flow estricto | `main` siempre liberable, sin `develop`/`release/*`/`hotfix/*`. PR obligatorio + squash merge + linear history. | [`estrategia-versionado_v1.0.md`](estrategia-versionado_v1.0.md) §7 |
| **D4** | TFM único `net10.0` | Aprovecha static abstract members, generic math, primary constructors, collection expressions. Sin polyfills. Cualquier cambio de TFM es breaking. | [`estrategia-versionado_v1.0.md`](estrategia-versionado_v1.0.md) §14 |
| **D5** | Conventional Commits + MinVer | Bump SemVer determinista desde commits; versión calculada desde Git tags sin commits de bump ni archivos de versión versionados. | [`estrategia-versionado_v1.0.md`](estrategia-versionado_v1.0.md) §8 |
| **D7** | Tono industrial en documentación | Sin TODOs didácticos, sin notas pedagógicas. Documentos operables como contrato. | Convención editorial transversal de la sección. |

D6 está reservado y no aplica a esta sección.

---

## RACI del proceso de release

R = Responsable de ejecutar · A = Aprobador · C = Consultado antes · I = Informado del resultado.

| Actividad | R | A | C | I |
|---|---|---|---|---|
| Decidir tipo de bump (vía Conventional Commits del PR) | Autor del PR | Reviewer | — | Release manager |
| Validar entrada en `[Unreleased]` del CHANGELOG | Autor del PR | Reviewer | — | — |
| Aprobar PR con breaking change | Reviewer | Tech Lead | Autores impactados | Equipo, integradores |
| Merge a `main` (squash) | Reviewer | Reviewer | — | DevOps |
| Cierre de `[Unreleased]` → `[X.Y.Z] - YYYY-MM-DD` | Release manager | Tech Lead | Autores de PRs incluidos | Equipo |
| Crear tag firmado `vX.Y.Z` | Release manager | Tech Lead | — | DevOps, equipo |
| Disparar pipeline de publicación (push del tag) | Release manager | — | — | DevOps |
| Publicar GitHub Release y release notes | Release manager | — | Tech Lead (si hay breaking) | Equipo, integradores |
| Comunicar a integradores externos | Release manager | — | Producto | Integradores |
| Yanking de versión por CVE | Release manager | Tech Lead **+** Security | DevOps | Equipo, consumidores afectados |
| Decidir extender ventana de soporte de una MAJOR retirada | Tech Lead | Producto | DevOps | Equipo |

Detalle ampliado en [`estrategia-versionado_v1.0.md`](estrategia-versionado_v1.0.md) §16.

---

## Trazabilidad cross-sección

| Documento DevOps | Se conecta con |
|---|---|
| `pipeline-ci-cd_v1.0.md` | [`08_calidad_y_pruebas/definition-of-done_v1.0.md`](../08_calidad_y_pruebas/definition-of-done_v1.0.md) (gates de cobertura ≥70 %, build/test verde como precondición de merge); [`06_backlog-tecnico/definition-of-ready_v1.0.md`](../06_backlog-tecnico/definition-of-ready_v1.0.md) (criterios de entrada al sprint que el pipeline valida automáticamente). |
| `estrategia-versionado_v1.0.md` | [`06_backlog-tecnico/product-backlog_v1.0.md`](../06_backlog-tecnico/product-backlog_v1.0.md) (PBI con flag de breaking change → MAJOR); [`00_contexto/roadmap-producto_v1.0.md`](../00_contexto/roadmap-producto_v1.0.md) (alineación de releases con fases/épicas). |
| `entornos-deploy_v1.0.md` | [`08_calidad_y_pruebas/estrategia-testing-motor_v1.0.md`](../08_calidad_y_pruebas/estrategia-testing-motor_v1.0.md) (qué tipo de prueba se ejecuta en cada entorno, criterios de promoción QA→STAGING→PROD). |
| `guia-publicacion-nuget_v1.0.md` | [`05_arquitectura_tecnica/guia-uso-libreria_v1.0.md`](../05_arquitectura_tecnica/guia-uso-libreria_v1.0.md) (uso de la API pública que el paquete expone); [`10_developer_guide/`](../10_developer_guide/) (guía orientada al integrador externo, complementa el §7 "Consumo desde un proyecto .NET"). |

---

## Mapa de uso por escenario

| Escenario operativo | Documento(s) primarios | Sección de entrada |
|---|---|---|
| Abrir un PR y elegir el tipo de commit | `estrategia-versionado_v1.0.md` | §8 Conventional Commits + scopes válidos |
| Reviewar un PR (validar bump, CHANGELOG, scope) | `estrategia-versionado_v1.0.md` | §8.2, §9.2 |
| Ejecutar un release (tag + publish + GitHub Release) | `estrategia-versionado_v1.0.md` + `guia-publicacion-nuget_v1.0.md` | §15 procedimiento operativo + §6.2 publicación |
| Diagnosticar un workflow rojo en `main` | `pipeline-ci-cd_v1.0.md` | §5 etapas + §6 gates |
| Configurar el feed en un proyecto consumidor | `guia-publicacion-nuget_v1.0.md` | §7.1 `nuget.config` + PAT |
| Publicar manualmente vía `workflow_dispatch` | `guia-publicacion-nuget_v1.0.md` | §6.3 |
| Deprecar una API pública | `estrategia-versionado_v1.0.md` | §10 breaking changes + §11 deprecation policy |
| Yankear una versión por CVE | `estrategia-versionado_v1.0.md` | §13 |
| Promover un cambio entre entornos | `entornos-deploy_v1.0.md` | §10 promoción + §7 configuración por entorno |
| Hacer rollback de una versión defectuosa | `estrategia-versionado_v1.0.md` + `entornos-deploy_v1.0.md` | versionado §15.4 (publicar PATCH) + entornos §12 (rollback consumidor) |
| Validar que el release final funciona end-to-end | `entornos-deploy_v1.0.md` | §6.4 (`samples/MotorDsl.MultaApp.Nuget` como smoke test) |

---

## Convenciones de la sección

- **Naming de archivos.** `<tema>_vX.Y.md` en kebab-case en español. La versión `vX.Y` corresponde al **documento**, no al paquete publicado: el control de cambios de cada doc es independiente del versionado del Motor DSL. Renombrar archivos está prohibido (rompe links cross-sección y referencias externas).
- **Control de cambios.** Toda modificación de un documento de esta sección requiere actualizar la tabla "Control de cambios" del propio documento con versión, fecha, autor y descripción concisa. La versión del documento sigue SemVer interno: bump MINOR para agregados, PATCH para correcciones, MAJOR para reorganización estructural que rompa anchors externos.
- **Commits de cambios documentales.** Se aplican Conventional Commits con scope `docs` (no impacta versión del paquete). Ejemplo: `docs(devops): clarifica política de yanking en estrategia-versionado §13`.
- **Versionado del documento vs versionado del paquete.** Son ortogonales. `pipeline-ci-cd_v1.0.md` puede estar en v1.2 mientras el paquete `MotorDsl.Core` está en `1.0.2`. No hay sincronización automática ni se busca.
- **Referencias internas.** Siempre con paths relativos al doc actual (`../08_calidad_y_pruebas/...`) y anclas explícitas (`§N`). Nunca URLs absolutas a `github.com` para contenido interno.
- **Referencias externas normativas.** Se citan con URL completa y versión de la especificación (ej. SemVer 2.0.0, Conventional Commits 1.0.0).

---

## Referencias externas

### Especificaciones normativas

- **Semantic Versioning 2.0.0** — <https://semver.org/spec/v2.0.0.html>
- **Conventional Commits 1.0.0** — <https://www.conventionalcommits.org/en/v1.0.0/>
- **Keep a Changelog 1.1.0** — <https://keepachangelog.com/en/1.1.0/>

### Frameworks y prácticas de seguridad de la cadena de suministro

- **SLSA (Supply-chain Levels for Software Artifacts)** — <https://slsa.dev/>
- **OWASP SCVS (Software Component Verification Standard)** — <https://owasp.org/www-project-software-component-verification-standard/>
- **NIST SSDF (Secure Software Development Framework, SP 800-218)** — <https://csrc.nist.gov/Projects/ssdf>

### Documentación de plataforma

- **NuGet best practices (Microsoft Docs)** — <https://learn.microsoft.com/en-us/nuget/concepts/package-authoring-best-practices>
- **GitHub Packages — NuGet registry** — <https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry>
- **GitHub Flow** — <https://docs.github.com/en/get-started/using-github/github-flow>
- **GitHub Security Advisories** — <https://docs.github.com/en/code-security/security-advisories>
- **.NET 10 — What's new** — <https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/overview>

### Herramientas

- **MinVer** — <https://github.com/adamralph/minver>
- **Nerdbank.GitVersioning** (alternativa documentada, no adoptada) — <https://github.com/dotnet/Nerdbank.GitVersioning>
- **Dependabot** — <https://docs.github.com/en/code-security/dependabot>

---

## Control de cambios

| Versión | Fecha | Autor | Cambios |
|---|---|---|---|
| 1.0 | 2026-04-25 | DevOps / Release Engineering | Versión inicial. Índice de la sección con orden de lectura, glosario operativo (24 términos), RACI consolidado de release, decisiones D1–D5+D7, trazabilidad cross-sección, mapa de uso por escenario, convenciones editoriales, referencias normativas (SemVer 2.0.0, Keep a Changelog 1.1.0, Conventional Commits 1.0.0, SLSA, OWASP SCVS, NIST SSDF). |

---

**Fin del documento**
