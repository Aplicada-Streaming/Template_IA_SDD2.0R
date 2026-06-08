# Entornos y Estrategia de Distribución

**Archivo:** entornos-deploy_v1.0.md
**Versión:** 2.0
**Fecha:** 2026-04-25
**Autor / Owner:** Release Engineering
**Estado:** Aprobado

---

## 1. Propósito y alcance

Este documento define el modelo de distribución del Motor DSL como **librería .NET** y delimita el concepto de "entorno" para los proyectos del repositorio.

**Decide:**

- Modelo de distribución de la librería: feeds + sufijos SemVer.
- Catálogo de feeds vigentes (Preview / Stable) sobre GitHub Packages.
- Política de promoción Preview → Stable.
- Entornos clásicos (DEV / QA / STAGING / PROD) aplicados **exclusivamente** a las sample apps consumers del repositorio.
- Configuración por contexto del consumer.
- Métricas de feed y métricas de consumer, separadas.
- Riesgos del flujo de publicación y mitigaciones.

**Aclaración conceptual obligatoria:** una librería distribuida vía paquetes **no tiene "entornos"**. No existe una "instancia productiva" del Motor DSL que se pueda apagar, reiniciar, escalar u observar en runtime. La librería se **publica** a un feed; sus consumers la **integran** en su propio build y son ellos los que tienen entornos. Aplicar el modelo DEV / QA / STAGING / PROD al artefacto `.nupkg` es un **anti-patrón** que confunde "publicación de paquete" con "despliegue de servicio". Este documento corrige ese error conceptual respecto de versiones previas (v1.0, v1.1, v1.2).

**No decide (ver documentos referenciados):**

- Reglas de versionado SemVer, Conventional Commits, breaking changes, yanking → [`estrategia-versionado_v1.0.md`](estrategia-versionado_v1.0.md).
- Pipeline CI/CD, jobs, gates de calidad → [`pipeline-ci-cd_v1.0.md`](pipeline-ci-cd_v1.0.md).
- Mecánica de empaquetado, metadata `.csproj`, consumo desde un proyecto cliente → [`guia-publicacion-nuget_v1.0.md`](guia-publicacion-nuget_v1.0.md).
- Definición de los 4 paquetes y sus dependencias internas → [`../05_arquitectura_tecnica/arquitectura-solucion_v1.0.md`](../05_arquitectura_tecnica/arquitectura-solucion_v1.0.md).

---

## 2. Audiencia

| Rol | Uso del documento |
|---|---|
| Release manager | Decide cuándo promover de Preview a Stable; ejecuta §4.3 y §8. |
| Mantenedores del paquete | Entienden por qué la librería no se "deploya" y aplican el modelo correcto en discusiones de release. |
| Mantenedores de sample apps (`MultaApp`, `MotorDsl.MultaApp.Nuget`) | Definen y operan los entornos DEV/QA/STAGING/PROD de su app. |
| Integradores externos | Saben qué feed configurar y cómo distinguir versiones Preview de Stable. |
| Arquitectura | Audita que el modelo se respete y que no reaparezcan referencias a "entornos de la librería". |

---

## 3. Modelo conceptual: feeds + consumers

### 3.1 Por qué una librería NO se "deploya"

Una aplicación web o un servicio tiene **instancias** corriendo: un proceso vivo, escuchando puertos, consumiendo CPU, atendiendo tráfico. Cuando se habla de "entorno productivo" de un servicio, se habla de un set concreto de máquinas, configuración y datos contra los que ese proceso opera. Promover de QA a PROD significa cambiar a qué máquinas se enruta el tráfico real.

Una librería **no tiene proceso propio**. El `.nupkg` publicado es un artefacto inerte: una zip con DLLs, metadata y dependencias declaradas. No corre. No escucha. No consume recursos hasta que un consumer lo referencia, lo compila dentro de su propio binario y ejecuta ese binario en su propio entorno. La librería no tiene "PROD" porque no tiene runtime: el runtime lo aporta el consumer.

Lo que sí tiene la librería es:

- **Versiones publicadas** a uno o más feeds (canales de distribución).
- **Audiencia diferenciada** por canal: integradores tempranos vs. consumers en producción.
- **Trazabilidad** entre tag Git, paquete publicado y CHANGELOG.

El modelo correcto es entonces **feeds + consumers downstream**, no entornos.

### 3.2 Diagrama de distribución

```text
┌─────────────────────────────────────────────────────────────────┐
│  REPOSITORIO MOTOR DSL                                          │
│  src/MotorDsl.Core, .Parser, .Rendering, .Extensions            │
│           │                                                     │
│           │  tag v*-alpha.N / -beta.N / -rc.N                   │
│           │  tag v*  (estable)                                  │
│           ▼                                                     │
│  ┌─────────────────────────────────────────┐                    │
│  │       Pipeline CI/CD (cd-nuget.yml)     │                    │
│  └────────────────┬────────────────────────┘                    │
│                   │ dotnet nuget push                           │
│                   ▼                                             │
│  ┌─────────────────────────────────────────┐                    │
│  │  GitHub Packages (feed único físico)    │                    │
│  │  https://nuget.pkg.github.com/<owner>/  │                    │
│  │                                         │                    │
│  │   Canal Preview     Canal Stable        │                    │
│  │   ─────────────     ────────────        │                    │
│  │   1.2.0-alpha.3     1.0.0               │                    │
│  │   1.2.0-rc.1        1.0.1               │                    │
│  │   2.0.0-beta.2      1.0.2               │                    │
│  └──┬──────────────────────────┬───────────┘                    │
└─────┼──────────────────────────┼────────────────────────────────┘
      │                          │
      │  IncludePrerelease=true  │  PackageReference estándar
      │                          │
      ▼                          ▼
┌─────────────────────┐   ┌──────────────────────────────┐
│ Consumers Preview   │   │ Consumers Stable             │
│                     │   │                              │
│ - MultaApp (DEV/QA) │   │ - MultaApp (STAGING/PROD)    │
│ - MotorDsl.MultaApp │   │ - Integraciones de terceros  │
│   .Nuget (sample    │   │ - MotorDsl.MultaApp.Nuget    │
│   E2E)              │   │   (validación de Stable)     │
│ - Integradores      │   │                              │
│   tempranos         │   │                              │
└─────────────────────┘   └──────────────────────────────┘
```

Los **consumers** son donde reaparece el modelo clásico de entornos: una app MAUI sí tiene DEV local, QA en CI, STAGING interno y PROD en Google Play / App Store. La librería no.

---

## 4. Feeds de distribución

### 4.1 GitHub Packages — Canal Preview

| Atributo | Valor |
|---|---|
| Feed físico | GitHub Packages del repositorio |
| URL | `https://nuget.pkg.github.com/<owner>/index.json` |
| Distinción | Versiones con sufijo SemVer pre-release (`-alpha.N`, `-beta.N`, `-rc.N`) |
| Publicación | Automática al pushear tag `vX.Y.Z-<sufijo>.N` |
| Audiencia | Integradores tempranos, equipo interno, sample app `MotorDsl.MultaApp.Nuget`, CI de proyectos pioneros |
| Consumo | Requiere `IncludePrerelease=true` en `PackageReference` o flag `--prerelease` en `dotnet add package` |
| Estabilidad esperada | Variable: `alpha` (baja) → `beta` (media) → `rc` (alta). Detalle en [`estrategia-versionado_v1.0.md §5`](estrategia-versionado_v1.0.md). |
| SLA de soporte | Ninguno. Las versiones pre-release no reciben fixes retroactivos. La acción correctiva siempre es "subir a la siguiente preview o al estable". |

**Cuándo se publica una preview:**

- `alpha.N`: durante desarrollo de un MINOR/MAJOR todavía no completo.
- `beta.N`: cuando la funcionalidad está completa pero pendiente de validación amplia.
- `rc.N`: candidato a release; solo se admiten bug fixes hasta el estable.

**Política de retención:** las versiones preview **no se borran** salvo yanking por CVE crítico (ver [`estrategia-versionado_v1.0.md §13`](estrategia-versionado_v1.0.md)). Permanecen indefinidamente en el feed para auditoría y reproducibilidad de builds históricos. No hay "limpieza" de previews antiguas.

### 4.2 GitHub Packages — Canal Stable

| Atributo | Valor |
|---|---|
| Feed físico | **El mismo** que Preview — GitHub Packages del repositorio |
| URL | `https://nuget.pkg.github.com/<owner>/index.json` (idéntica) |
| Distinción | Versiones SemVer **sin** sufijo pre-release (`1.0.0`, `1.0.1`, `1.0.2`, `1.2.3`) |
| Publicación | Automática al pushear tag `vX.Y.Z` (sin sufijo) |
| Audiencia | Todos los consumers, incluidos los productivos. Default cuando un proyecto referencia el paquete sin habilitar prerelease. |
| Consumo | `PackageReference` estándar. NuGet selecciona la última versión estable salvo que se pinee. |
| Estabilidad esperada | Alta. Cumple criterios de promoción §4.3. |
| SLA de soporte | Última MAJOR: features + fixes + security. Penúltima MAJOR: solo security 6 meses. Ver [`estrategia-versionado_v1.0.md §12`](estrategia-versionado_v1.0.md). |

**Decisión de diseño:** Preview y Stable comparten **un único feed físico**. La distinción es exclusivamente por sufijo SemVer y, opcionalmente, por la flag `prerelease=true` que NuGet propaga en el listado. No se crea un segundo feed (`-preview` o equivalente) porque GitHub Packages ya distingue pre-releases nativamente y duplicar el feed sería overhead operativo sin beneficio.

**Política de no-republicación:** GitHub Packages rechaza por diseño el push de una versión ya existente. No hay forma de "sobrescribir" `1.0.2` con un binario distinto. Si una versión publicada tiene un bug, se publica una nueva PATCH. Si tiene un CVE crítico, ver yanking.

### 4.3 Promoción Preview → Stable

La promoción es el acto de pasar una línea de versión de canal Preview a canal Stable. **No** se hace renombrando un paquete preview: cada estable tiene su propio tag Git distinto.

**Criterios obligatorios para promover `X.Y.Z-rc.N` a `X.Y.Z`:**

| Criterio | Cómo se valida |
|---|---|
| RC vivió ≥ 7 días en Preview | Fecha del tag `vX.Y.Z-rc.N` en GitHub |
| Sample apps de referencia consumieron el RC sin issues | `MotorDsl.MultaApp.Nuget` y `MultaApp` builds verdes contra el RC |
| Sin bug crítico abierto contra el RC | Tracker de issues filtrado por label `release-blocker` |
| Tests del repositorio en verde sobre `main` | Último run de `ci.yml` |
| CHANGELOG `[Unreleased]` cerrado a `[X.Y.Z]` | PR de release mergeado |
| Tech Lead aprobó | Comentario en issue de tracking del release |

Cumplidos los criterios, el release manager crea el tag estable firmado y el pipeline publica los 4 paquetes en el canal Stable. Detalle operativo en [`estrategia-versionado_v1.0.md §15`](estrategia-versionado_v1.0.md).

**Observación importante:** la promoción de canal **no es** un copy del binario preview al canal stable. Es un nuevo build con un tag nuevo. Los binarios `1.0.0-rc.1` y `1.0.0` son **artefactos distintos** aunque su contenido funcional sea idéntico.

### 4.4 Política de retención

| Caso | Acción |
|---|---|
| Versión publicada, sin issues | Permanece indefinidamente. No se borra. |
| Versión publicada, con bug | Permanece. Se publica nueva PATCH con el fix. |
| Versión publicada, con CVE crítico (CVSS ≥ 9.0 o alto con vector remoto sin auth) | Yanking conforme a [`estrategia-versionado_v1.0.md §13`](estrategia-versionado_v1.0.md). Requiere versión sustituta publicada simultáneamente y aprobación Tech Lead + Security. |
| Preview obsoleta (varias versiones más nuevas en el canal) | Permanece. No hay limpieza programada. |
| Versión publicada por error en metadata (no funcional) | Se publica nueva versión con metadata correcta. La versión errónea no se yankea salvo que cause fallas de resolución en consumers existentes. |

**Regla:** no se borran versiones publicadas. La inmutabilidad del feed es propiedad explícita del modelo de distribución.

---

## 5. Paquetes publicados

### 5.1 Catálogo

| Paquete | Versión actual | Feed | URL del paquete |
|---|---|---|---|
| `MotorDsl.Core` | 1.0.2 | GitHub Packages | `https://github.com/<owner>/?tab=packages&q=MotorDsl.Core` |
| `MotorDsl.Parser` | 1.0.2 | GitHub Packages | `https://github.com/<owner>/?tab=packages&q=MotorDsl.Parser` |
| `MotorDsl.Rendering` | 1.0.2 | GitHub Packages | `https://github.com/<owner>/?tab=packages&q=MotorDsl.Rendering` |
| `MotorDsl.Extensions` | 1.0.2 | GitHub Packages | `https://github.com/<owner>/?tab=packages&q=MotorDsl.Extensions` |

**Feed común:** `https://nuget.pkg.github.com/<owner>/index.json`.
**Trigger de publicación:** tag `v*` en GitHub (ej. `v1.0.2`, `v1.1.0-rc.1`).
**Workflow:** `.github/workflows/cd-nuget.yml`.
**Secret:** `GITHUB_TOKEN` (provisto automáticamente por GitHub Actions, requiere permiso `packages: write`).

Las dependencias internas entre los 4 paquetes y la decisión de empaquetado granular están detalladas en [`guia-publicacion-nuget_v1.0.md §3`](guia-publicacion-nuget_v1.0.md).

### 5.2 Trazabilidad

Para cada versión publicada existe una cadena de identificadores que **debe** mantenerse íntegra:

```text
Tag Git           Release GitHub        Paquetes en feed                  CHANGELOG
v1.0.2 (firmado)  Release "v1.0.2"      MotorDsl.Core.1.0.2.nupkg         ## [1.0.2] - 2026-04-XX
                                        MotorDsl.Parser.1.0.2.nupkg
                                        MotorDsl.Rendering.1.0.2.nupkg
                                        MotorDsl.Extensions.1.0.2.nupkg
```

**Reglas:**

1. Toda versión publicada tiene un tag Git firmado correspondiente.
2. Todo tag Git de release tiene un GitHub Release asociado con notas.
3. Los 4 paquetes se publican **simultáneamente** con la misma versión. No existe un release donde solo 2 de los 4 estén publicados.
4. Toda versión figura en `CHANGELOG.md` con su sección `[X.Y.Z] - YYYY-MM-DD`.

Detalle de la cadena en [`estrategia-versionado_v1.0.md §9`](estrategia-versionado_v1.0.md).

---

## 6. Consumers downstream y sus entornos

Las **sample apps** del repositorio sí tienen entornos clásicos. La distinción es importante: cuando se habla de "PROD" en este documento de aquí en adelante, se habla del entorno productivo de **una app**, nunca de la librería.

### 6.1 Sample apps del repositorio

| App consumer | Tipo | Consume | DEV (local) | QA (CI) | STAGING (interno) | PROD (Google Play / App Store) |
|---|---|---|---|---|---|---|
| `MultaApp` | App MAUI Android/iOS de demo / referencia | Proyectos del repo vía `ProjectReference` o el paquete publicado | Sí | Sí | Sí | Sí |
| `MotorDsl.MultaApp.Nuget` | Sample E2E que valida el flujo de publicación | Paquete publicado en GitHub Packages (consumer externo simulado) | Sí | Sí | — | — |

`MultaApp` representa el caso de uso real que un cliente productivo tendría. `MotorDsl.MultaApp.Nuget` es una variante diseñada para validar que el paquete publicado es consumible exactamente como lo haría un proyecto externo (autenticación al feed, resolución transitiva de dependencias, restore desde cero). Esta segunda app deliberadamente **no** llega a PROD: su propósito es ser test de integración del flujo de release, no una app de mercado.

### 6.2 Entornos de las sample apps MAUI

| Entorno | Configuración | Datos | Despliegue | Consume librería desde |
|---|---|---|---|---|
| **DEV** | Local, debug build, hot reload, logging verbose | Mocks y fixtures del repo | Emulador Android / simulador iOS local del developer | `ProjectReference` directo o paquete del canal Preview |
| **QA** | Build release sobre runner CI, modo strict | Datos de prueba versionados con la app | Artifacts del workflow (APK / `.app.zip`), retención según `cd-android.yml` y `cd-ios-*.yml` | Paquete del canal Preview o Stable según rama |
| **STAGING** | Build release firmado, configuración productiva | Pseudo-reales, anonimizados | TestFlight (iOS) / Firebase App Distribution o Google Play Internal Testing (Android) | Paquete del canal Stable, versión pinneada |
| **PROD** | Build release firmado con keystore productivo | Reales | Google Play release tracks / App Store production | Paquete del canal Stable, versión exacta auditable |

**Notas operativas:**

- DEV y QA pueden consumir Preview porque están en validación y se acepta romper builds ante un breaking change descubierto temprano. STAGING y PROD **deben** consumir Stable: un consumer productivo no debe estar acoplado a un sufijo `-rc.N` que SemVer considera inestable.
- El versionado de la app es independiente del versionado de la librería. La app puede subir versiones sin cambiar la versión del paquete consumido y viceversa.
- Las claves de firma productivas, las API keys de los stores y los certificados iOS son responsabilidad del equipo de la app, no del repositorio de la librería.

### 6.3 Sample `MotorDsl.MultaApp.Nuget` (E2E)

**Propósito:** validar que el paquete publicado en GitHub Packages es consumible como lo haría un consumer externo. Es prueba de integración del flujo completo de empaquetado y publicación, no una app destinada a mercado.

**Características:**

- Vive en `samples/MotorDsl.MultaApp.Nuget`.
- Tiene `nuget.config` apuntando explícitamente al feed GitHub Packages.
- Su `.csproj` declara `<PackageReference Include="MotorDsl.Extensions" Version="..." />` con versión específica, no `ProjectReference`.
- Su build se ejerce en CI tras cada release para confirmar que la última versión es consumible.

**Entornos aplicables:**

- **DEV:** un developer prueba localmente que puede restaurar y compilar.
- **QA (CI):** workflow ejerce el build con la última versión publicada.
- **STAGING / PROD:** **no aplican**. Esta app no se distribuye. Si llegara a hacerlo, dejaría de cumplir su función de "consumer externo neutro" y se transformaría en una app más con su propia política.

Esta sample es el **test funcional del feed**: si `MotorDsl.MultaApp.Nuget` no compila contra el último paquete Stable, el release está roto aunque los tests internos del repositorio hayan pasado.

---

## 7. Configuración por contexto de consumer

La configuración relevante cambia entre los entornos del consumer, no entre versiones de la librería. Esta tabla aplica al consumer (`MultaApp` y similares); la librería no participa de esta configuración salvo recibiendo opciones vía DI.

| Variable / aspecto | DEV consumer | QA consumer | STAGING consumer | PROD consumer |
|---|---|---|---|---|
| `LogLevel` (consumer) | Debug | Information | Warning | Warning |
| `LogLevel` (librería, vía opciones) | Debug | Information | Warning | Warning |
| Feed NuGet usado en restore | Preview + Stable | Preview + Stable | Stable | Stable |
| `IncludePrerelease` | `true` | `true` (configurable) | `false` | `false` |
| Versión del paquete pinneada | Flotante (`*`) o última | Flotante o pin a la PR en validación | **Pin exacto** | **Pin exacto y auditado** |
| Datos consumidos | Mocks | Fixtures de QA | Pseudo-reales | Reales |
| Crash reporting | Desactivado o local | Activado a sandbox | Activado a sandbox | Activado a backend productivo |
| Firma del binario del consumer | No requerida | Opcional | Requerida | Requerida con keystore productivo |
| Símbolos / PDB de la librería | Embebidos | Embebidos | Embebidos | Embebidos (publicación de symbols server pendiente) |

**Lo que NO cambia entre entornos del consumer:**

- El binario de la librería. Una vez resuelto `MotorDsl.Extensions 1.0.2`, el `.dll` referenciado es bit-a-bit el mismo en DEV, QA, STAGING y PROD del consumer. La inmutabilidad del paquete es garantía operativa.
- El esquema del DSL. Un template JSON válido en DEV es válido en PROD del consumer.
- Las firmas públicas de las APIs. Si funciona en DEV, funciona en PROD; si rompe en PROD, rompe en DEV.

**Implicancia:** cualquier diferencia de comportamiento entre entornos del consumer **no** puede deberse a la librería siempre que la versión sea la misma. Es siempre configuración, datos o entorno del consumer. Esto descarga al equipo de la librería de debugging cruzado por entorno.

---

## 8. Promoción entre feeds y entre entornos del consumer

Hay **dos flujos independientes** que se sincronizan en el punto de consumo. Confundirlos lleva a malas prácticas (ej. "tagear PROD" en la librería).

### 8.1 Flujo de la librería (canales)

```text
1. PR a main con commit Conventional Commits
   ↓
2. Merge a main → CI verde
   ↓
3. (opcional) Tag vX.Y.Z-alpha.N → publish canal Preview
   ↓
4. (opcional) Tag vX.Y.Z-beta.N → publish canal Preview
   ↓
5. Tag vX.Y.Z-rc.N → publish canal Preview
   ↓ (criterios §4.3 cumplidos, ≥ 7 días, sin críticos)
   ↓
6. Tag vX.Y.Z (firmado) → publish canal Stable
```

Cada paso es disparado por push de tag. El pipeline `cd-nuget.yml` los ejecuta uniformemente.

### 8.2 Flujo del consumer (entornos)

```text
1. Developer del consumer agrega/actualiza PackageReference
   ↓
2. Build local (DEV) → smoke test
   ↓
3. PR del consumer → CI (QA del consumer)
   ↓
4. Merge → build firmado para STAGING → distribución interna
   ↓
5. Validación interna → release a PROD (Play / App Store)
```

Este flujo es responsabilidad del equipo del consumer, no del equipo de la librería.

### 8.3 Punto de sincronización

```text
Consumer .csproj:
  <PackageReference Include="MotorDsl.Extensions" Version="1.0.2" />
                                                          ↑
                                            Decisión del equipo del consumer:
                                            qué versión del feed Stable consumir
```

El equipo de la librería **garantiza** que las versiones Stable cumplen sus contratos SemVer. El equipo del consumer **decide** cuándo y a qué versión actualizar. Ningún equipo puede "empujar" un cambio al otro: la librería no fuerza al consumer a actualizar; el consumer no fuerza a la librería a publicar.

### 8.4 Casos cruzados

| Situación | Acción |
|---|---|
| Consumer encuentra bug en versión Stable consumida | Reporta issue en repo de la librería; aplica workaround o pin a versión previa hasta que haya nueva PATCH. |
| Librería publica nueva MAJOR | Consumer evalúa y planifica migración; la PATCH/MINOR de su MAJOR previa sigue soportada según [`estrategia-versionado_v1.0.md §12`](estrategia-versionado_v1.0.md). |
| Consumer necesita feature todavía no liberada | Puede consumir desde Preview en sus entornos DEV/QA, **nunca** en PROD. |
| Librería yankea versión por CVE | Consumer recibe error de restore en próxima build; debe actualizar a la versión sustituta indicada en el advisory. |

---

## 9. Versionado

El esquema, las reglas de incremento, la mecánica de pre-releases, los criterios de breaking change, la política de deprecación y la ventana de soporte están definidos exclusivamente en [`estrategia-versionado_v1.0.md`](estrategia-versionado_v1.0.md). **Este documento no duplica esas reglas.**

Lo único que aporta este documento sobre versionado es la **separación entre canales** (Preview / Stable) en términos de distribución, no en términos de cómo se calcula el número de versión.

---

## 10. Observabilidad

### 10.1 Métricas del feed (de la librería)

Las métricas relevantes de la librería son **del feed**, no de runtime. La librería no tiene runtime propio para observar.

| Métrica | Fuente | Uso |
|---|---|---|
| Downloads por versión | GitHub Packages UI / API | Detectar adopción de Stable nuevas; identificar versiones legacy aún consumidas. |
| Versiones publicadas por trimestre | Tags Git + Releases | Cadencia de release; salud del proceso. |
| Ratio Preview / Stable | API GitHub Packages | Indicador de cuánto se usa el canal de validación temprana. |
| Tiempo Preview-rc → Stable | Diferencia entre tags `vX.Y.Z-rc.N` y `vX.Y.Z` | Validar cumplimiento de la regla "≥ 7 días". |
| Vulnerabilidades reportadas (Dependabot, GHSA) | GitHub Security tab | Trigger de PATCHes de seguridad o yanking. |

**No** son métricas de la librería: tiempo de respuesta, throughput, memoria consumida, crash rate. Esas son métricas del consumer.

### 10.2 Métricas de las sample apps (de los consumers)

Cada sample app tiene su propia observabilidad, propia de aplicación móvil.

| Métrica | Aplica a | Fuente típica |
|---|---|---|
| Crash rate | `MultaApp` PROD | Firebase Crashlytics / App Store Connect |
| ANR (Android) | `MultaApp` PROD Android | Google Play Console |
| Tiempo de render de un documento DSL | `MultaApp` cualquier entorno | Telemetría propia de la app |
| Errores de parseo DSL | `MultaApp` cualquier entorno | Logs de la app |
| Fallos de restore NuGet en CI | `MotorDsl.MultaApp.Nuget` QA | Logs de GitHub Actions |

Estas métricas pueden ser **input** para el equipo de la librería (un crash recurrente en un método de `MotorDsl.Rendering` motiva un fix), pero son **propiedad** del equipo del consumer. La librería no instrumenta el runtime del consumer.

---

## 11. Responsables (RACI)

| Actividad | R (ejecuta) | A (aprueba) | C (consultado) | I (informado) |
|---|---|---|---|---|
| Decidir publicar Preview | Release manager | Tech Lead | Mantenedores | Integradores |
| Promover RC a Stable | Release manager | Tech Lead | Security (si hay fix de CVE incluido) | Equipo, integradores externos |
| Yankear versión por CVE | Release manager | Tech Lead **+** Security | DevOps | Consumers afectados |
| Mantener catálogo §5.1 actualizado | Release manager | Tech Lead | DevOps | Equipo |
| Definir entornos de `MultaApp` | Equipo de la app | Producto | DevOps | — |
| Definir entornos de `MotorDsl.MultaApp.Nuget` | DevOps | Tech Lead | Mantenedores | — |
| Decidir versión consumida por una sample app | Equipo de la sample app | Tech Lead de la sample | Release manager de la librería | — |
| Reportar bug de la librería desde un consumer | Mantenedor del consumer | — | Mantenedores de la librería | Release manager |

`R` = ejecuta. `A` = autoriza. `C` = aporta antes. `I` = recibe el resultado.

---

## 12. Riesgos y mitigaciones

| Riesgo | Impacto | Probabilidad | Mitigación |
|---|---|---|---|
| Paquete publicado con metadata corrupta (PackageId mal, dependencias rotas) | Alto: consumers no pueden restaurar | Baja | Pipeline ejerce `dotnet pack` + restauración cruzada en `MotorDsl.MultaApp.Nuget` antes de publicar; metadata centralizada en `Directory.Build.props`. |
| Breaking change inadvertido en MINOR/PATCH | Alto: rompe consumers en producción | Media | Conventional Commits + review obligatorio; en roadmap análisis automático de breaking ([`pipeline-ci-cd_v1.0.md §13`](pipeline-ci-cd_v1.0.md)). |
| Diamond dependency con bump MAJOR de dep transitiva | Medio: incompatibilidad en consumers | Media | Política §10 de [`estrategia-versionado_v1.0.md`](estrategia-versionado_v1.0.md): bump MAJOR de dep → MAJOR propio. |
| CVE en dependencia transitiva | Crítico | Media | Dependabot + advisory; PATCH inmediato; yanking si CVSS ≥ 9.0. |
| Confusión Preview ↔ Stable en consumer productivo | Alto: app PROD consume binario inestable | Media | Tabla §7 explícita; `IncludePrerelease=false` por default en `nuget.config` del consumer productivo. |
| Tag firmado con clave incorrecta | Bajo, pero compromete trazabilidad | Baja | Procedimiento §15.4 de [`estrategia-versionado_v1.0.md`](estrategia-versionado_v1.0.md): borrar tag antes de pipeline o avanzar a PATCH. |
| Re-aparición del anti-patrón "deploy de la librería" en otros docs o discusiones | Bajo operativo, alto conceptual | Media | Este documento (§3.1) es referencia normativa; reviewers rechazan PRs que mezclen entornos con feeds. |
| `MotorDsl.MultaApp.Nuget` falla tras release | Alto: indica feed roto | Baja | Su build forma parte del gate post-release; rollback vía nueva PATCH. |
| Pérdida de acceso a GitHub Packages (downtime de GitHub) | Crítico durante el incidente | Muy baja | Aceptado: dependencia de plataforma. SLA de GitHub aplica. Plan de contingencia: mirror manual a feed alternativo es decisión de Arquitectura ad-hoc, no automatizada. |
| Versión yankeada rompe builds históricos del consumer | Medio | Baja | Yanking solo procede con versión sustituta publicada; comunicación previa a consumers conocidos. |

---

## 13. Referencias

### 13.1 Internas

- [`estrategia-versionado_v1.0.md`](estrategia-versionado_v1.0.md) — SemVer, Conventional Commits, MinVer, breaking changes, deprecación, soporte, yanking, TFM `net10.0`.
- [`pipeline-ci-cd_v1.0.md`](pipeline-ci-cd_v1.0.md) — Triggers, jobs, gates de calidad, workflows.
- [`guia-publicacion-nuget_v1.0.md`](guia-publicacion-nuget_v1.0.md) — Empaquetado, metadata, consumo desde un proyecto cliente.
- [`../05_arquitectura_tecnica/arquitectura-solucion_v1.0.md`](../05_arquitectura_tecnica/arquitectura-solucion_v1.0.md) — Definición de los 4 paquetes y dependencias internas.
- [`../08_calidad_y_pruebas/definition-of-done_v1.0.md`](../08_calidad_y_pruebas/definition-of-done_v1.0.md) — Gates de calidad como precondición de release.

### 13.2 Externas

- Semantic Versioning 2.0.0 — <https://semver.org/spec/v2.0.0.html>
- GitHub Packages (NuGet) — <https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry>
- GitHub Security Advisories — <https://docs.github.com/en/code-security/security-advisories>
- NuGet versioning (prerelease) — <https://learn.microsoft.com/en-us/nuget/concepts/package-versioning>
- Google Play release tracks — <https://support.google.com/googleplay/android-developer/answer/9859751>
- TestFlight — <https://developer.apple.com/testflight/>

---

## 14. Control de cambios

| Versión | Fecha | Autor | Cambios |
|---|---|---|---|
| v1.0 | 2026-03-28 | DevOps | Versión inicial con modelo DEV / QA / STAGING / PROD aplicado a la librería. |
| v1.1 | 2026-04-02 | DevOps | PROD = NuGet.org real; tablas de paquetes publicados; sample `MotorDsl.MultaApp.Nuget`. |
| v1.2 | 2026-04-25 | DevOps | Reconciliación de feed a GitHub Packages. Resolución de duplicado §6.4. Delegación de versionado al doc dedicado. |
| **v2.0** | **2026-04-25** | **Release Engineering** | **Reescritura conceptual completa: modelo feeds + consumers downstream. El modelo DEV / QA / STAGING / PROD se reserva para sample apps consumers, no para la librería. Resuelve el anti-patrón de "deploy de librería como si fuera servicio" presente en v1.0–v1.2. Conserva la trazabilidad de los paquetes 1.0.2 publicados y la mención de `MotorDsl.MultaApp.Nuget` como E2E de consumo.** |

---
