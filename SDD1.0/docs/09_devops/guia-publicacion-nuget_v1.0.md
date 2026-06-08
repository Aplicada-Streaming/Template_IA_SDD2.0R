# Guía de Publicación y Consumo de Paquetes NuGet  
**Archivo:** guia-publicacion-nuget_v1.0.md  
**Proyecto:** Motor DSL (Librería de Generación de Documentos)  
**Versión:** v1.2  
**Estado:** Aprobado  
**Fecha:** 2026-04-25  
**Owner:** Arquitectura / DevOps  

---

## 1. Propósito

Este documento describe los pasos necesarios para empaquetar, publicar y consumir los componentes del Motor DSL como paquetes NuGet, permitiendo su reutilización en cualquier proyecto .NET (consola, API, MAUI, Blazor, etc.).

---

## 2. Conceptos fundamentales

### 2.1 ¿Qué es un paquete NuGet?

NuGet es el administrador de paquetes oficial para .NET. Un paquete NuGet (`.nupkg`) es un archivo comprimido que contiene:

- Las DLLs compiladas de la librería
- Metadata (autor, versión, descripción, licencia)
- Declaración de dependencias hacia otros paquetes

Cuando un consumidor instala un paquete, NuGet descarga automáticamente ese paquete **y todas sus dependencias transitivas**.

### 2.2 ¿Qué es una dependencia transitiva?

Si el paquete **A** depende de **B**, y **B** depende de **C**, al instalar **A** se descargan automáticamente **B** y **C**. El consumidor no necesita instalarlos manualmente.

```
MotorDsl.Extensions
  └── MotorDsl.Core
  └── MotorDsl.Parser       → depende de Core
  └── MotorDsl.Rendering    → depende de Core
```

Al instalar `MotorDsl.Extensions`, NuGet trae transitivamente Core, Parser y Rendering.

### 2.3 Estrategias de empaquetado: monolítico vs granular

Existen dos enfoques principales para publicar una librería compuesta por varios proyectos:

| Estrategia | Descripción | Ejemplo de la industria |
|---|---|---|
| **Monolítica** | Un solo paquete con todo | `Newtonsoft.Json` |
| **Granular** | Varios paquetes con dependencias entre sí | `Microsoft.Extensions.DependencyInjection` + `Microsoft.Extensions.DependencyInjection.Abstractions` |

#### Ventajas del enfoque granular
- El consumidor instala solo lo que necesita (ej: solo los contratos/interfaces)
- Menor huella de dependencias en proyectos livianos
- Permite versionado independiente por componente

#### Ventajas del enfoque monolítico
- Menor complejidad operativa (1 versión, 1 publicación)
- El consumidor instala un solo paquete
- No hay problemas de sincronización de versiones entre paquetes

### 2.4 Cuadro comparativo de opciones para este proyecto

| Opción | Paquetes publicados | El consumidor instala | Complejidad |
|---|---|---|---|
| **A) 4 paquetes** (actual) | `Core`, `Parser`, `Rendering`, `Extensions` | `MotorDsl.Extensions` | Alta — 4 versiones sincronizadas |
| **B) 1 paquete** | `MotorDsl` (todo junto) | `MotorDsl` | Baja — 1 versión, 1 artefacto |
| **C) 2 paquetes** | `MotorDsl.Core` (contratos) + `MotorDsl` (todo) | `MotorDsl` | Media — separación contratos/implementación |

### 2.5 Decisión adoptada

Se adopta la **opción A (4 paquetes)** con `MotorDsl.Extensions` como punto de entrada único recomendado para el consumidor. Esto permite flexibilidad futura (consumo granular) manteniendo una experiencia simple para el caso de uso principal.

> **Regla para el consumidor:** instalar únicamente `MotorDsl.Extensions`. Este paquete trae todo lo necesario de forma transitiva.

---

## 3. Arquitectura de paquetes

### 3.1 Diagrama de dependencias

```
┌──────────────────────────────────┐
│       MotorDsl.Extensions        │  ← El consumidor instala ESTE
│  (DI, fluent API, AddMotorDsl)   │
└──────┬──────────┬──────────┬─────┘
       │          │          │
       ▼          ▼          ▼
┌──────────┐ ┌──────────┐ ┌──────────────┐
│  .Core   │ │  .Parser │ │  .Rendering  │
│(contratos│ │(JSON DSL │ │(EscPos, Text │
│ modelos) │ │ parser)  │ │  renderers)  │
└──────────┘ └────┬─────┘ └──────┬───────┘
                  │              │
                  ▼              ▼
             ┌──────────────────────┐
             │     MotorDsl.Core    │
             └──────────────────────┘
```

### 3.2 Tabla de paquetes

| Paquete | Contenido | Dependencias NuGet | Dependencias internas |
|---|---|---|---|
| `MotorDsl.Core` | Contratos, modelos, evaluador, layout engine | Ninguna | — |
| `MotorDsl.Parser` | Parser JSON DSL → `DocumentTemplate` | — | → Core |
| `MotorDsl.Rendering` | `EscPosRenderer`, `TextRenderer` | `System.Text.Encoding.CodePages` | → Core |
| `MotorDsl.Extensions` | DI, fluent API `AddMotorDslEngine()` | `Microsoft.Extensions.DependencyInjection.Abstractions` | → Core, Parser, Rendering |

### 3.3 ¿Qué instala el consumidor?

```bash
dotnet add package MotorDsl.Extensions --version 1.0.0
```

Esto descarga transitivamente: `MotorDsl.Core`, `MotorDsl.Parser`, `MotorDsl.Rendering` y sus dependencias externas.

---

## 4. Prerrequisitos

### 4.1 Herramientas

- .NET SDK 10.0 o superior
- Git (para tags de versionado)

### 4.2 Cuentas y tokens

| Destino | Cuenta requerida | Token / API Key |
|---|---|---|
| **GitHub Packages** (único feed de publicación) | Cuenta GitHub con acceso al repositorio | `GITHUB_TOKEN` (provisto automáticamente en GitHub Actions, requiere permiso `packages: write` en el workflow) o PAT con scope `read:packages` (consumo) / `write:packages` (publicación, restringido a release manager) |

---

## 5. Metadata NuGet en los proyectos

### 5.1 Estado actual

Los archivos `.csproj` actuales **no tienen metadata de paquete**. Ejemplo de `MotorDsl.Core.csproj`:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
</Project>
```

Faltan propiedades esenciales: `PackageId`, `Description`, `Authors`, `License`, `RepositoryUrl`, etc.

### 5.2 Recomendación: crear `Directory.Build.props`

Crear un archivo `src/Directory.Build.props` para centralizar las propiedades compartidas entre los 4 proyectos empaquetables. La configuración debe cubrir identidad, licenciamiento, símbolos, builds reproducibles, Source Link y validación anti-breaking-change.

#### Propiedades — justificación

| Propiedad | Valor | Justificación |
|---|---|---|
| `PackageIcon` | `icon.png` | Branding visual en clientes NuGet (VS, nuget.org, GitHub). El archivo `icon.png` (≤ 1 MB, 128×128 PNG recomendado) debe incluirse en el paquete vía `<None Include="..\..\assets\icon.png" Pack="true" PackagePath="\" />`. |
| `PackageReleaseNotes` | `See CHANGELOG.md` | Apunta al changelog versionado del repo. Alternativa industrial: extracción automática por versión en CI con `awk '/^## \['"$VERSION"'\]/,/^## \[/' CHANGELOG.md` y pasarlo vía `-p:PackageReleaseNotes="..."` en `dotnet pack`. |
| `IncludeSymbols` | `true` | Genera paquete de símbolos en cada `dotnet pack`. |
| `SymbolPackageFormat` | `snupkg` | Formato moderno recomendado por NuGet, reemplaza el legacy `.symbols.nupkg`. Compatible con symbol servers de NuGet/GitHub. |
| `Deterministic` | `true` | Builds reproducibles bit-a-bit; pre-requisito de Source Link y supply-chain integrity. |
| `ContinuousIntegrationBuild` | `true` (cuando `GITHUB_ACTIONS == 'true'`) | Normaliza paths absolutos en PDBs y desactiva optimizaciones que rompen reproducibilidad. Solo se activa en CI para no degradar la experiencia local. |
| `EmbedUntrackedSources` | `true` | Source Link captura archivos generados (T4, source generators) que no están en git. |
| `PublishRepositoryUrl` | `true` | Incrusta `RepositoryUrl` y `RepositoryCommit` en el `.nuspec` del paquete; Visual Studio usa estos metadatos para el step-into. |
| `RepositoryType` | `git` | Tipo de VCS — requerido por Source Link. |
| `RepositoryCommit` | `$(GITHUB_SHA)` | SHA exacto desde CI; ancla el paquete al commit reproducible. |
| `MinClientVersion` | `5.0.0` | Versión mínima de cliente NuGet requerida (compatible con SDK-style projects, license expressions, README embebido). |
| `PackageReadmeFile` | `README.md` | README embebido visible en clientes NuGet. Requiere `<None Include="..\..\README.md" Pack="true" PackagePath="\" />`. |
| `EnablePackageValidation` | `true` | Validación automática anti-breaking-change desde .NET 8+; falla el `pack` si la API pública rompe compatibilidad respecto del baseline. |

#### `Directory.Build.props` completo

```xml
<Project>
  <PropertyGroup>
    <!-- Identidad y licencia -->
    <Authors>UTN - Tecnicatura Universitaria en Programación</Authors>
    <Company>UTN</Company>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <PackageReadmeFile>README.md</PackageReadmeFile>
    <PackageIcon>icon.png</PackageIcon>
    <PackageTags>thermal-printer;escpos;dsl;dotnet;maui</PackageTags>
    <PackageReleaseNotes>See CHANGELOG.md</PackageReleaseNotes>

    <!-- Repositorio y Source Link -->
    <RepositoryUrl>https://github.com/Aplicada-Streaming/Ejemplo_IA_SDD_Template</RepositoryUrl>
    <RepositoryType>git</RepositoryType>
    <RepositoryCommit>$(GITHUB_SHA)</RepositoryCommit>
    <PublishRepositoryUrl>true</PublishRepositoryUrl>
    <EmbedUntrackedSources>true</EmbedUntrackedSources>

    <!-- Símbolos -->
    <IncludeSymbols>true</IncludeSymbols>
    <SymbolPackageFormat>snupkg</SymbolPackageFormat>

    <!-- Builds reproducibles -->
    <Deterministic>true</Deterministic>
    <ContinuousIntegrationBuild Condition="'$(GITHUB_ACTIONS)' == 'true'">true</ContinuousIntegrationBuild>

    <!-- Compatibilidad y validación -->
    <MinClientVersion>5.0.0</MinClientVersion>
    <EnablePackageValidation>true</EnablePackageValidation>
  </PropertyGroup>

  <ItemGroup>
    <None Include="..\..\README.md" Pack="true" PackagePath="\" />
    <None Include="..\..\assets\icon.png" Pack="true" PackagePath="\" />
  </ItemGroup>
</Project>
```

Y en cada `.csproj` agregar la propiedad específica:

```xml
<PropertyGroup>
  <PackageId>MotorDsl.Core</PackageId>
  <Description>Contratos, modelos y motor de evaluación del DSL para impresión térmica.</Description>
</PropertyGroup>
```

### 5.3 Source Link y debugging

**Qué hace Source Link.** Permite a los consumidores del paquete debuggear con step-into hasta el código fuente exacto del commit publicado, sin necesidad de clonar el repo. El IDE descarga los archivos `.cs` desde GitHub raw vía URLs incrustadas en los PDBs.

**Configuración.** Referenciar el paquete `Microsoft.SourceLink.GitHub` en cada proyecto empaquetable (o centralizar en `Directory.Build.props`):

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.SourceLink.GitHub" Version="8.0.0" PrivateAssets="All" />
</ItemGroup>
```

`PrivateAssets="All"` evita que Source Link se propague como dependencia transitiva al consumidor (es solo build-time).

**Resultado.** El `.snupkg` publicado al feed contiene PDBs con URLs apuntando a `raw.githubusercontent.com/Aplicada-Streaming/Ejemplo_IA_SDD_Template/<commit>/...`. Visual Studio y JetBrains Rider hacen step-into transparente cuando `Tools → Options → Debugging → Enable Source Link support` está activo.

**Verificación post-publish.**

```bash
# Verificar firma e integridad del paquete
nuget verify -Signatures ./nupkg/MotorDsl.Core.1.0.0.nupkg

# Inspeccionar metadata Source Link incrustada en el PDB
dotnet sourcelink test ./nupkg/MotorDsl.Core.1.0.0.snupkg
```

### 5.4 Política de licencia

| Concepto | Decisión |
|---|---|
| Licencia | **MIT** (D2) |
| Mecanismo | `<PackageLicenseExpression>MIT</PackageLicenseExpression>` (SPDX expression) |
| **NO usar** | `<PackageLicenseFile>` — deprecated para licencias estándar; reservar solo para licencias custom no-SPDX |
| Archivo `LICENSE` | Texto MIT estándar en raíz del repo (responsabilidad del bootstrap del repositorio, no de este documento) |
| Compatibilidad downstream | MIT es permisiva; compatible con consumidores GPL, Apache 2.0, BSD, propietarios |
| Third-party notices | Si en el futuro se incluyen dependencias con licencias no triviales (ej: Apache 2.0 con NOTICE), agregar `THIRD-PARTY-NOTICES.txt` al paquete vía `<None Include="..\..\THIRD-PARTY-NOTICES.txt" Pack="true" PackagePath="\" />` |

> **Nota:** estos cambios no están aplicados aún en los `.csproj`. Son la configuración objetivo para la primera publicación formal.

### 5.5 Propiedades recomendadas por paquete

| Paquete | `PackageId` | `Description` |
|---|---|---|
| Core | `MotorDsl.Core` | Contratos, modelos y motor de evaluación del DSL para impresión térmica |
| Parser | `MotorDsl.Parser` | Parser JSON DSL que transforma plantillas en DocumentTemplate |
| Rendering | `MotorDsl.Rendering` | Renderizadores ESC/POS y texto plano para documentos DSL |
| Extensions | `MotorDsl.Extensions` | Integración con DI y fluent API para el Motor DSL de impresión térmica |

---

## 6. Publicación paso a paso

### 6.1 Prueba local con `dotnet pack`

Generar los `.nupkg` localmente para verificar que el empaquetado funciona:

```bash
# Desde la raíz del repositorio
dotnet build src/MotorDsl.Extensions/MotorDsl.Extensions.csproj -c Release

dotnet pack src/MotorDsl.Core/MotorDsl.Core.csproj -c Release -o ./nupkg
dotnet pack src/MotorDsl.Parser/MotorDsl.Parser.csproj -c Release -o ./nupkg
dotnet pack src/MotorDsl.Rendering/MotorDsl.Rendering.csproj -c Release -o ./nupkg
dotnet pack src/MotorDsl.Extensions/MotorDsl.Extensions.csproj -c Release -o ./nupkg
```

Verificar el contenido:

```bash
ls ./nupkg/
# MotorDsl.Core.1.0.0.nupkg
# MotorDsl.Parser.1.0.0.nupkg
# MotorDsl.Rendering.1.0.0.nupkg
# MotorDsl.Extensions.1.0.0.nupkg
```

### 6.2 Publicación en GitHub Packages (automática vía CI/CD)

El workflow `cd-nuget.yml` se encarga automáticamente. Los pasos son:

#### Paso 1 — Crear un tag de versión

```bash
git tag v1.0.0
git push origin v1.0.0
```

#### Paso 2 — El workflow se ejecuta automáticamente

1. Ejecuta los 185 tests
2. Empaqueta los 4 proyectos con la versión del tag (ej: `1.0.0`)
3. Publica en `https://nuget.pkg.github.com/<owner>/index.json`
4. Sube los `.nupkg` como artifacts del workflow (retención 30 días)

#### Paso 3 — Verificar

- En GitHub → repositorio → Packages: deben aparecer los 4 paquetes
- En el workflow run → Artifacts: descargar los `.nupkg`

### 6.3 Publicación en GitHub Packages (manual vía workflow_dispatch)

1. Ir a GitHub → Actions → **CD NuGet - Publish MotorDsl**
2. Click en **Run workflow**
3. Ingresar versión (ej: `1.0.0`)
4. El workflow ejecuta los mismos pasos que con un tag

### 6.4 Versiones preview (automáticas)

Cada push a `main` genera paquetes con versión `0.0.0-preview.<run_number>` como artifacts descargables (no se publican al feed). Útil para pruebas internas.

### 6.5 NuGet.org — fuera de alcance

NuGet.org **no se utiliza** como destino de publicación en este proyecto. El feed único oficial es GitHub Packages (ver §6.2 y §6.3). Cualquier propuesta de distribución pública adicional debe pasar por revisión de Arquitectura y modificar este documento explícitamente.

---

## 7. Consumo desde un proyecto .NET

### 7.1 Configurar el feed (GitHub Packages)

GitHub Packages requiere autenticación. Crear un archivo `nuget.config` en la raíz del proyecto consumidor:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="github" value="https://nuget.pkg.github.com/<owner>/index.json" />
  </packageSources>
  <packageSourceCredentials>
    <github>
      <add key="Username" value="TU_USUARIO_GITHUB" />
      <add key="ClearTextPassword" value="TU_PAT_CON_READ_PACKAGES" />
    </github>
  </packageSourceCredentials>
</configuration>
```

> **Seguridad:** no commitear tokens en el repositorio. Usar variables de entorno o `dotnet nuget add source` en la máquina local. El PAT del consumidor debe tener únicamente scope `read:packages`.

### 7.2 Instalar el paquete

```bash
dotnet add package MotorDsl.Extensions --version 1.0.0
```

Esto agrega al `.csproj`:

```xml
<ItemGroup>
  <PackageReference Include="MotorDsl.Extensions" Version="1.0.0" />
</ItemGroup>
```

### 7.3 Uso en código

#### Aplicación .NET con DI (API, Worker, MAUI, etc.)

```csharp
// Program.cs o MauiProgram.cs
using MotorDsl.Extensions;

builder.Services.AddMotorDslEngine();
```

#### Uso directo (aplicación consola sin DI)

```csharp
using MotorDsl.Parser;
using MotorDsl.Rendering;

// Parsear una plantilla DSL
var parser = new DslParser();
var template = parser.Parse(jsonDsl);

// Renderizar a ESC/POS
var renderer = new EscPosRenderer();
byte[] output = renderer.Render(template, data);
```

### 7.4 Ejemplo mínimo funcional (consola)

```csharp
using MotorDsl.Extensions;
using Microsoft.Extensions.DependencyInjection;

var services = new ServiceCollection();
services.AddMotorDslEngine();
var provider = services.BuildServiceProvider();

// Resolver servicios registrados y usar el motor
var engine = provider.GetRequiredService<IDslEngine>();
var result = engine.Process(jsonTemplate, data);
```

---

## 8. Versionado

El proyecto sigue **Semantic Versioning (SemVer)** en formato `MAJOR.MINOR.PATCH`. Reglas completas, casos de borde y política de breaking changes en [estrategia-versionado_v1.0.md](estrategia-versionado_v1.0.md).

Ejemplos rápidos: `1.0.1` (PATCH, bugfix) y `2.0.0` (MAJOR, breaking change).

### ¿Cómo se determina la versión en el workflow?

| Trigger | Versión resultante | ¿Se publica al feed? |
|---|---|---|
| Tag `v1.2.3` (sobre `main`) | `1.2.3` | Sí |
| `workflow_dispatch` con input `1.2.3` | `1.2.3` | Sí |
| Push a `main` | `0.0.0-preview.<run_number>` | No (solo artifact) |

---

## 9. Política de yanking ante CVE

### 9.1 Cuándo yankear

Los criterios de disparo están definidos en [`estrategia-versionado_v1.0.md` §13](estrategia-versionado_v1.0.md): CVSS ≥ 9.0 o vector de explotación remoto sin autenticación. Esta sección documenta únicamente el **procedimiento operativo** en GitHub Packages; los criterios de decisión no se duplican.

### 9.2 Procedimiento operativo en GitHub Packages

**Vía GitHub Web UI:**

1. Repositorio → **Packages** → seleccionar el paquete afectado
2. **Package settings** → **Manage versions**
3. Localizar la versión comprometida → **Delete this version**
4. Confirmar el typeahead del nombre

**Vía API (recomendado para procedimientos auditables):**

```bash
# Listar versiones para obtener el ID
gh api /orgs/<owner>/packages/nuget/<package>/versions

# Eliminar versión específica
gh api -X DELETE /orgs/<owner>/packages/nuget/<package>/versions/<id>
```

Para paquetes en cuentas personales, reemplazar `/orgs/<owner>/` por `/users/<owner>/`.

### 9.3 Acciones simultáneas obligatorias

El delete del paquete **no es suficiente**. Toda operación de yanking debe ejecutar en la misma ventana operativa:

| Acción | Responsable | Artefacto |
|---|---|---|
| Publicar versión PATCH/MINOR con la mitigación | Release manager | Tag `vX.Y.Z` + paquete en feed |
| Crear GitHub Security Advisory (GHSA) | Release manager | Advisory público con CVE asignado |
| Entrada `Security` en `CHANGELOG.md` describiendo el fix | Release manager | Commit en `main` |
| Marcar la versión yankeada en `CHANGELOG.md` con sufijo `[YANKED]` | Release manager | Commit en `main` |
| Notificar a consumidores conocidos (canal interno) | DevOps | Mensaje en canal de releases |

### 9.4 GitHub Packages vs NuGet.org — diferencias críticas

| Plataforma | Operación disponible | Reversibilidad | Impacto en consumidores |
|---|---|---|---|
| **GitHub Packages** | `delete` (destructivo) | Irreversible | Nuevas restauraciones fallan con 404; consumidores con la versión cacheada en `~/.nuget/packages` la conservan operativa |
| **NuGet.org** | `unlist` (no-destructivo) | Reversible | Versión queda accesible vía `<PackageReference>` explícita pero no aparece en búsquedas ni resoluciones de rango |

**Implicancia operativa.** En GitHub Packages el yanking es destructivo: una vez eliminada, la versión no puede republicarse con el mismo número (NuGet bloquea republicación incluso tras delete por consistencia de hashes). Esto refuerza la necesidad de coordinar el delete con la publicación de la versión PATCH antes de comunicar el incidente, para que los consumidores tengan ruta de actualización inmediata.

---

## 10. Troubleshooting

### 10.1 Error de autenticación en GitHub Packages

```
error: Response status code does not indicate success: 401 (Unauthorized)
```

**Solución:** verificar que el PAT tenga el scope `read:packages` (para consumir) o `write:packages` (para publicar). Regenerar si expiró.

### 10.2 Paquete no aparece en el feed

- Los paquetes pueden tardar 1-2 minutos en indexarse
- Verificar en GitHub → repositorio → **Packages** que el paquete fue publicado
- Si usó `--skip-duplicate` y la versión ya existía, no se sobrescribe (diseño intencional)

### 10.3 Versión duplicada

```
error: Conflict - The feed already contains 'MotorDsl.Core 1.0.0'
```

**Solución:** incrementar la versión. Las versiones publicadas son inmutables por diseño. Crear un nuevo tag (ej: `v1.0.1`).

### 10.4 El paquete no trae las dependencias transitivas

Verificar que los `.csproj` usen `<ProjectReference>` (no `<Reference>`). El SDK de .NET convierte automáticamente las `ProjectReference` en dependencias del paquete NuGet al hacer `dotnet pack`.

### 10.5 Error `dotnet pack` falla con `--no-build`

El paso `--no-build` requiere que se haya hecho `dotnet build -c Release` previamente. Si falla, ejecutar sin `--no-build`:

```bash
dotnet pack src/MotorDsl.Core/MotorDsl.Core.csproj -c Release -o ./nupkg
```

---

## 11. Resumen rápido

```
Publicar:
  git tag v1.0.0 && git push origin v1.0.0
  → El workflow cd-nuget.yml hace el resto

Consumir:
  dotnet add package MotorDsl.Extensions --version 1.0.0
  → En código: services.AddMotorDslEngine();
```

---

## 12. Target Frameworks (TFMs)

Aplicación operativa de la decisión D4 (TFM único `net10.0`).

| Decisión | Valor | Justificación |
|---|---|---|
| `TargetFrameworks` | `net10.0` (único) | Acceso a APIs de .NET 10: interfaces estáticas (static abstract members), generic math, primary constructors en clases, collection expressions. Trade-off explícito: consumidores quedan atados a runtime .NET 10. |
| Multi-target | NO | Decisión arquitectónica permanente — referencia a [`estrategia-versionado_v1.0.md` §14](estrategia-versionado_v1.0.md). Migración futura será MAJOR + reemplazo del TFM (`net10.0` → `net11.0`), nunca agregado de targets de compatibilidad. |
| Compatibilidad hacia atrás | No soportada | Quien necesite ejecutar sobre .NET 8 debe consumir la última versión MAJOR previa de la librería compatible (cuando exista en la línea de releases). No se backportean fixes a TFMs descontinuados. |

**Implicancia en el `.csproj`.** Mantener `<TargetFramework>` en singular (no `<TargetFrameworks>` plural). Esto evita la generación accidental de assemblies multi-target y mantiene `dotnet pack` produciendo un único `lib/net10.0/` dentro del `.nupkg`.

---

## 13. Referencias

- [pipeline-ci-cd_v1.0.md](pipeline-ci-cd_v1.0.md) — Pipeline CI/CD completo
- [estrategia-versionado_v1.0.md](estrategia-versionado_v1.0.md) — Estrategia de versionado SemVer
- [entornos-deploy_v1.0.md](entornos-deploy_v1.0.md) — Entornos de despliegue
- [Documentación oficial NuGet](https://learn.microsoft.com/en-us/nuget/) — Microsoft Docs
- [GitHub Packages con NuGet](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry) — GitHub Docs
- [Source Link](https://github.com/dotnet/sourcelink) — dotnet/sourcelink
- [Package Validation](https://learn.microsoft.com/en-us/dotnet/fundamentals/apicompat/package-validation/overview) — .NET API compatibility tooling

---

## 14. Control de cambios

| Versión | Fecha | Autor | Descripción |
|---|---|---|---|
| v1.0 | 2026-04-02 | Arquitectura / DevOps | Guía inicial de publicación y consumo NuGet |
| v1.1 | 2026-04-25 | Arquitectura / DevOps | fix(devops): reconciliar feed a GitHub Packages como único destino. Eliminada §6.5 sobre NuGet.org "futuro". `NUGET_API_KEY` reemplazado por `GITHUB_TOKEN` (`packages: write`). Trigger de publicación limitado a `main` (GitHub Flow). Sección §8 reducida a referencia al doc dedicado de versionado. |
| v1.2 | 2026-04-25 | NuGet Specialist | Metadatos production-grade (PackageIcon, IncludeSymbols, snupkg, Deterministic, ContinuousIntegrationBuild, EmbedUntrackedSources, PublishRepositoryUrl, MinClientVersion, EnablePackageValidation). Source Link. Política MIT explícita. Política de yanking ante CVE. TFM único net10.0 documentado. Slug del repo unificado. |
