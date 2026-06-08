  # Pipeline CI/CD — Motor DSL  
**Archivo:** pipeline-ci-cd_v1.0.md  
**Proyecto:** Motor DSL para generación y renderizado de documentos  
**Versión:** v1.3  
**Estado:** Aprobado  
**Fecha:** 2026-04-25  
**Owner:** DevOps  

---

## 1. Propósito

Este documento define el pipeline de Integración Continua y Entrega Continua (CI/CD) del Motor DSL. Su objetivo es automatizar la construcción, validación, empaquetado y publicación del motor como paquete **NuGet**, asegurando calidad, trazabilidad y consistencia en cada versión liberada.

---

## 2. Alcance

El pipeline cubre:

- Build de librerías .NET  
- Ejecución de tests  
- Análisis de calidad  
- Empaquetado como NuGet (.nupkg)  
- Publicación en registry (NuGet local o remoto)  
- Build y publicación de apps Android (APK)  
- Build de app iOS (simulador .app.zip)  
- Scripts de desarrollo local (.bat)  

Fuera de alcance en v1.3:

- Publicación multi-registry  
- Validación automática de breaking changes (planeado v2.x)  
- Generación automática de changelog (planeado v2.x)  

---

## 3. Estrategia general

El pipeline se ejecuta ante:

- Pull Requests  
- Push a ramas principales  
- Tags de release  

Flujo de alto nivel:

```text
Commit → Build → Tests → Quality → Pack → Publish (NuGet)
```

---

## 4. Disparadores (triggers)

### 4.1 Matriz de triggers y runners

| Trigger      | Runners                              | Etapas                                                       |
| ------------ | ------------------------------------ | ------------------------------------------------------------ |
| PR a `main`  | `ubuntu-latest`, `windows-latest`    | build, test, lint, analyzer, SCA                             |
| Push a `main`| `ubuntu-latest`                      | build, test                                                  |
| Tag `v*`     | `ubuntu-latest`                      | build, test, pack, sign, SBOM, publish                       |

**Justificación de matriz multi-OS en PR:** la librería será consumida desde múltiples sistemas operativos. Tests en `ubuntu-latest` + `windows-latest` permiten detectar tempranamente issues de path separator (`/` vs `\`), line endings (`LF` vs `CRLF`) y file casing (case-sensitive vs case-insensitive). En push a `main` y tags se prioriza `ubuntu-latest` por costo/tiempo, asumiendo que la matriz cruzada ya validó en PR.

**Ejemplo de tag**

```text
v1.3.0
```

---

## 5. Etapas del pipeline

---

### 5.1 Stage: Checkout

**Objetivo:** obtener el código fuente.

**Pasos**

* Clonar repositorio
* Restaurar dependencias
* Validar versión desde tag o build

---

### 5.2 Stage: Build

**Objetivo:** compilar las librerías del motor.

**Incluye**

* MotorDsl.Core
* MotorDsl.Parser
* MotorDsl.Rendering
* MotorDsl.Extensions

**Criterio de éxito**

* Build sin errores

---

### 5.3 Stage: Tests

**Objetivo:** validar calidad funcional.

**Incluye**

* Tests unitarios
* Tests de integración del pipeline DSL

**Reglas**

* Cobertura mínima: 70%
* Fallo en tests → pipeline falla

---

### 5.4 Stage: Lint / Format

**Objetivo:** garantizar consistencia de estilo previo al merge.

**Comando**

```bash
dotnet format --verify-no-changes
```

**Regla**

* Gate obligatorio en PR. Diferencia de formato → pipeline falla.
* La corrección local (`dotnet format`) es responsabilidad del autor del PR; el CI no auto-comitea.

---

### 5.5 Stage: Análisis estático (Roslyn analyzers)

**Objetivo:** detectar issues de código en tiempo de compilación.

**Configuración**

* Paquete: `Microsoft.CodeAnalysis.NetAnalyzers` (referenciado en `Directory.Build.props`).
* `<TreatWarningsAsErrors>true</TreatWarningsAsErrors>` activo en `Release`.
* Severidad mínima: `warning`. Cualquier warning rompe el build.

**Cubre**

* Code smells, convenciones de naming, reglas de seguridad CA-rules, performance.
* Validación de arquitectura mediante reglas custom si aplica.

**Regla**

* Issues de severidad `warning` o superior → pipeline falla.

---

### 5.6 Stage: SCA (Software Composition Analysis)

**Objetivo:** detectar vulnerabilidades conocidas en dependencias directas y transitivas.

**Componentes**

1. **GitHub Dependabot** habilitado vía `.github/dependabot.yml`. Contenido esperado del archivo:
   * `version: 2`
   * Entradas `package-ecosystem: nuget` con `directory: "/"`, `schedule.interval: "weekly"`.
   * Entradas `package-ecosystem: github-actions` con `directory: "/"`, `schedule.interval: "weekly"`.
   * `open-pull-requests-limit: 10` por ecosistema.
   * Labels automáticos: `dependencies`, `security`.

2. **Auditoría en CI**:
   ```bash
   dotnet list package --vulnerable --include-transitive
   ```

**Threshold de gate**

* Bloquear PR si existe vulnerabilidad de severidad `High` o superior (incluye `Critical`).
* Vulnerabilidades `Moderate` / `Low` → warning, no bloqueante; se trackean en Dependabot PRs.

**Justificación:** cumplimiento OWASP SCVS v1 §3 (Component Inventory & Vulnerability Management).

---

### 5.7 Stage: Generación de SBOM

**Objetivo:** producir un Software Bill of Materials por paquete para trazabilidad de cadena de suministro.

**Formato:** CycloneDX 1.5+ (JSON).

**Herramienta:** `Microsoft.Sbom.Tool` (preferida, output SPDX + CycloneDX) o `CycloneDX/cyclonedx-dotnet` como alternativa.

**Output**

* `sbom.json` por cada `.nupkg` generado.
* Adjuntado como asset al GitHub Release del tag `v*`.
* Subido también como artifact del workflow con retención de 90 días.

**Justificación de cumplimiento**

* **SLSA Level 2:** provenance + build service auditable + SBOM versionado por release.
* **NIST SSDF PS.3 (Archive and Protect Each Software Release):** SBOM inmutable junto al binario firmado.
* **OWASP SCVS v1 §1.2:** SBOM machine-readable por cada release.

---

### 5.8 Stage: Empaquetado (Pack)

**Objetivo:** generar el paquete NuGet.

**Salida**

* Archivo `.nupkg`
* Archivo `.snupkg` (símbolos, ver §5.11)

**Comando base**

```bash
dotnet pack -c Release --no-build
```

**Ejemplo de nombre**

```text
MotorDsl.1.3.0.nupkg
```

---

### 5.9 Stage: Firma de paquete

**Objetivo:** garantizar integridad y autenticidad del paquete publicado.

**Comando**

```bash
dotnet nuget sign MotorDsl.*.nupkg \
  --certificate-path $CERT_PFX_PATH \
  --certificate-password $NUGET_CERT_PASSWORD \
  --timestamper http://timestamp.digicert.com
```

**Secrets requeridos**

| Secret                     | Contenido                                                            |
| -------------------------- | -------------------------------------------------------------------- |
| `NUGET_CERT_PFX_BASE64`    | Certificado PFX del proyecto codificado en base64.                   |
| `NUGET_CERT_PASSWORD`      | Password del PFX.                                                    |

El step de CI decodifica `NUGET_CERT_PFX_BASE64` a un archivo temporal en `$RUNNER_TEMP/cert.pfx` y lo pasa por `--certificate-path`. El archivo se elimina al finalizar el job.

**Verificación post-publish**

```bash
dotnet nuget verify MotorDsl.*.nupkg --all
```

* Falla → republicar como PATCH siguiendo §9 (rollback).

**Justificación:** requisito SLSA L2 + NIST SSDF PW.6 (Configure the Compilation, Interpreter, and Build Processes to Improve Executable Security).

---

### 5.10 Stage: Publicación (Publish NuGet)

**Objetivo:** distribuir el paquete.

**Condición**

* Solo en tags `v*` (ej. `v1.3.0`).

**Destino**

* **GitHub Packages** (`https://nuget.pkg.github.com/<owner>/index.json`) — producción.

**Secret requerido:** `GITHUB_TOKEN` (provisto automáticamente por GitHub Actions, requiere permiso `packages: write` en el workflow).

**Metadatos requeridos en cada csproj:**
`<PackageId>`, `<Authors>`, `<Description>`, `<PackageTags>`, `<PackageLicenseExpression>`, `<RepositoryUrl>`

**Comando base**

```bash
dotnet nuget push MotorDsl.1.3.0.nupkg --api-key $GITHUB_TOKEN --source https://nuget.pkg.github.com/<owner>/index.json
```

---

### 5.11 Stage: Source Link y símbolos

**Objetivo:** habilitar debugging de consumidores contra el código fuente exacto del release.

**Configuración en `Directory.Build.props`** (referencia detallada: `guia-publicacion-nuget_v1.0.md §5`):

```xml
<PublishRepositoryUrl>true</PublishRepositoryUrl>
<EmbedUntrackedSources>true</EmbedUntrackedSources>
<IncludeSymbols>true</IncludeSymbols>
<SymbolPackageFormat>snupkg</SymbolPackageFormat>
```

**Salida adicional:** `*.snupkg` publicado al mismo feed que el `.nupkg`.

---

### 5.12 Stage: Reproducibilidad

**Objetivo:** garantizar builds determinísticos y reproducibles bit-a-bit.

**Configuración**

* `global.json` en raíz del repo:
  ```json
  {
    "sdk": {
      "version": "10.0.x",
      "rollForward": "latestFeature"
    }
  }
  ```
* En `Directory.Build.props`:
  ```xml
  <ContinuousIntegrationBuild>true</ContinuousIntegrationBuild>
  <Deterministic>true</Deterministic>
  ```

**Justificación:** elimina paths absolutos del PDB, normaliza timestamps, y permite verificación de provenance contra builds re-ejecutados (SLSA L3 prerequisite).

---

### 5.13 Stage: Caché

**Objetivo:** reducir tiempo de restore en CI.

**Configuración**

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.nuget/packages
    key: ${{ runner.os }}-nuget-${{ hashFiles('**/packages.lock.json', '**/*.csproj') }}
    restore-keys: |
      ${{ runner.os }}-nuget-
```

**Impacto medido:** reducción esperada de 50–70% en tiempo de `dotnet restore` para hits de caché. Requiere `RestorePackagesWithLockFile=true` en csproj para resultados determinísticos.

---

### 5.14 Stage: CD Android

**Trigger:** push main o tag v*

- Genera APK de SampleApp y MultaApp
- Artifact: *-APK con retención 7 días

---

### 5.15 Stage: CD iOS (simulador)

**Trigger:** push main (en paths `samples/MotorDsl.SampleApp/**` o `samples/MotorDsl.MultaApp/**`)

- SampleApp y MultaApp en workflows separados
- Genera .app.zip para simulador iossimulator-arm64
- Artifact: *.app.zip con retención 1 día
- Runner requerido: `macos-15`, Xcode 26.0

---

## 6. Gates de calidad

El pipeline falla si ocurre cualquiera de los siguientes:

* Error de compilación
* Tests fallidos
* Cobertura < 70%
* Diferencias en `dotnet format --verify-no-changes`
* Warnings de Roslyn analyzers (con `TreatWarningsAsErrors=true`)
* Vulnerabilidades de severidad `High` o `Critical` detectadas por SCA
* Fallo en empaquetado, firma o verificación post-publish del paquete

---

## 7. Versionado en el pipeline

Se utiliza **Semantic Versioning (SemVer)**.

La versión se determina por:

1. Tag git (principal)
2. Número de build (fallback)

**Ejemplos**

* v1.0.0 → versión oficial
* v1.1.0 → nueva funcionalidad
* v1.1.1 → bugfix

Fallback:

```text
1.1.0-build.45
```

---

## 8. Variables de entorno

Variables mínimas:

* `GITHUB_TOKEN`
* `NUGET_SOURCE`
* `BUILD_CONFIGURATION`
* `VERSION`
* `NUGET_CERT_PFX_BASE64` (firma de paquete)
* `NUGET_CERT_PASSWORD` (firma de paquete)
* `SLACK_WEBHOOK_URL` (notificaciones, ver §10)

**Reglas**

* Nunca hardcodear API keys
* Usar secretos del pipeline
* Rotación anual obligatoria de `NUGET_CERT_PFX_BASE64` o ante sospecha de compromiso

---

## 9. Estrategia de rollback

En caso de error en publicación:

* No sobrescribir versiones existentes (inmutabilidad)
* Publicar nueva versión corregida (PATCH)
* Mantener historial de versiones

**Ejemplo**

```text
1.1.0 (fallo)
→ 1.1.1 (corregido)
```

---

## 10. Observabilidad del pipeline

### 10.1 Notificaciones

* Webhook a Slack/Teams en estados `failure` y `cancelled` de cualquier workflow.
* Variable requerida: `SLACK_WEBHOOK_URL` (secret).
* Estados `success` no notifican (ruido evitado); excepción: publicación exitosa de tag `v*` notifica al canal de releases.

### 10.2 Métricas DORA

| Métrica                    | Definición operacional                                                              | Fuente                                    |
| -------------------------- | ----------------------------------------------------------------------------------- | ----------------------------------------- |
| Lead Time for Changes      | Tiempo entre primer commit en branch y merge a `main`.                              | GitHub API (PR `commits[0].author.date` → `merged_at`). |
| Deployment Frequency       | Cantidad de tags `v*` publicados por mes.                                           | GitHub API (releases por período).         |
| Change Failure Rate        | % de releases que requieren patch (`vX.Y.Z+1`) dentro de las 24 h post-publicación. | GitHub Releases + tags.                    |
| MTTR (Mean Time To Restore)| Tiempo entre detección de bug (issue/incident) y publicación del release de fix.    | GitHub Issues `created_at` → release `published_at`. |

### 10.3 Dashboard

* **Base:** GitHub Insights + GitHub Actions usage metrics (built-in).
* **Avanzado:** exportación de logs y métricas a Datadog o Grafana Cloud → **fuera de alcance v1.x**, evaluado en roadmap v2.x.

---

## 11. Ejemplo de workflows

### 11.1 Estructura de archivos

```text
.github/workflows/
├── ci.yml                   ← tests en cada PR y push (185 tests, matriz ubuntu+windows en PR)
├── cd-android.yml           ← APK SampleApp + MultaApp (ubuntu-latest)
├── cd-ios-sampleapp.yml     ← .app.zip SampleApp, iossimulator-arm64 (macos-15)
├── cd-ios-multaapp.yml      ← .app.zip MultaApp, iossimulator-arm64 (macos-15)
└── cd-nuget.yml             ← GitHub Packages en tags v*, GITHUB_TOKEN (ubuntu-latest)
```

### 11.2 `ci.yml` mínimo

```yaml
name: CI
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'
      - uses: actions/cache@v4
        with:
          path: ~/.nuget/packages
          key: ${{ runner.os }}-nuget-${{ hashFiles('**/packages.lock.json', '**/*.csproj') }}
          restore-keys: |
            ${{ runner.os }}-nuget-
      - run: dotnet restore
      - run: dotnet format --verify-no-changes
      - run: dotnet build --configuration Release --no-restore
      - run: dotnet test --no-build --collect:"XPlat Code Coverage"
      - run: dotnet list package --vulnerable --include-transitive
```

---

## 12. Riesgos

| Riesgo                | Impacto | Mitigación                   |
| --------------------- | ------- | ---------------------------- |
| Versionado incorrecto | Alta    | uso de tags SemVer           |
| Publicación duplicada | Alta    | versiones inmutables         |
| Tests insuficientes   | Alta    | cobertura mínima obligatoria |
| API key expuesta      | Crítica | uso de secrets seguros       |

---

## 13. Evolución futura

Implementado en v1.0 (actualización):

- ✅ Publicación automática en GitHub Packages (cd-nuget.yml con GITHUB_TOKEN)
- ✅ 4 paquetes publicados: MotorDsl.Core, Parser, Rendering, Extensions v1.0.2
- ✅ Soporte iOS: cd-ios-sampleapp.yml y cd-ios-multaapp.yml
- ✅ Sample de integración NuGet: MotorDsl.MultaApp.Nuget

Implementado en v1.3:

- ✅ Lint/Format gate (`dotnet format --verify-no-changes`)
- ✅ Análisis estático con Roslyn analyzers (`Microsoft.CodeAnalysis.NetAnalyzers`, warnings as errors)
- ✅ SCA con Dependabot + `dotnet list package --vulnerable` (gate: High/Critical)
- ✅ Generación de SBOM CycloneDX por release (cumplimiento SLSA L2 + NIST SSDF PS.3)
- ✅ Firma de paquete con `dotnet nuget sign` + verificación post-publish
- ✅ Source Link y símbolos (`.snupkg`)
- ✅ Reproducibilidad: `global.json`, `Deterministic`, `ContinuousIntegrationBuild`
- ✅ Matriz OS (`ubuntu-latest` + `windows-latest`) en PR
- ✅ Caché NuGet (`actions/cache@v4`)
- ✅ Métricas DORA y notificaciones Slack/Teams

Planeado para v2.x:

- Validación de compatibilidad (breaking changes) con `Microsoft.CodeAnalysis.PublicApiAnalyzers`
- Generación automática de changelog con `release-drafter` o `git-cliff`
- Firma de APK con keystore propio
- Publicación en Google Play (AAB)
- Exportación de métricas a Datadog/Grafana Cloud
- Scripts .bat para desarrollo local Windows

> **Nota — TFM único `net10.0`:** la migración a multi-target NO está en evolución futura. Según `estrategia-versionado_v1.0.md §14`, el TFM único es decisión arquitectónica permanente; cualquier cambio a futuro será MAJOR y reemplazará el TFM, no agregará compatibilidad multi-target.

---

## 14. Control de cambios

| Versión | Fecha      | Autor  | Descripción                             |
| ------- | ---------- | ------ | --------------------------------------- |
| v1.0    | 2026-03-29 | DevOps | Pipeline inicial para publicación NuGet |
| v1.1    | 2026-04-02 | DevOps | Publicación real en NuGet.org (NUGET_API_KEY), cd-ios-multaapp.yml, MotorDsl.MultaApp.Nuget sample |
| v1.2    | 2026-04-25 | DevOps | Reconciliación de feed: GitHub Packages como único destino de publicación. Eliminada referencia a NuGet.org. Trigger de push limitado a `main` (GitHub Flow). |
| v1.3    | 2026-04-25 | DevOps | Refuerzo de cadena de suministro: SCA, SBOM, firma de paquete, Source Link, matriz OS, caché, métricas DORA, ejemplo YAML real. Incorpora SLSA Level 2 y NIST SSDF PS.3. |

---
