# Estrategia de Versionado

**Archivo:** estrategia-versionado_v1.0.md
**Versión:** 1.0
**Fecha:** 2026-04-25
**Autor / Owner:** Equipo DevOps / Release Engineering
**Estado:** Aprobado

---

## 1. Propósito y alcance

Este documento es la autoridad operativa para versionar y liberar los paquetes del Motor DSL. Define cómo se asigna la versión, cómo se materializa en Git y en el feed de paquetes, qué constituye un cambio mayor y qué proceso debe seguir cada release a producción.

**Decide:**

- Esquema de numeración (SemVer 2.0.0) y reglas de incremento.
- Mapeo entre cambios de código y bumps de versión.
- Política de pre-releases, build metadata y promoción.
- Estrategia de branching (GitHub Flow) y reglas de protección.
- Convención de commits (Conventional Commits) y herramienta de auto-versioning (MinVer).
- Política de breaking changes, deprecaciones, soporte y yanking.
- Procedimiento operativo de release y RACI asociado.

**No decide (ver documentos referenciados):**

- Definición del pipeline CI/CD → [`pipeline-ci-cd_v1.0.md`](pipeline-ci-cd_v1.0.md).
- Modelo de feeds Preview/Stable y consumers downstream → [`entornos-deploy_v1.0.md`](entornos-deploy_v1.0.md).
- Mecánica de empaquetado y consumo del feed → [`guia-publicacion-nuget_v1.0.md`](guia-publicacion-nuget_v1.0.md).
- Gates de calidad funcional → [`definition-of-done_v1.0.md`](../08_calidad_y_pruebas/definition-of-done_v1.0.md).

---

## 2. Audiencia

| Rol | Uso del documento |
|---|---|
| Release manager | Procedimiento operativo (§15), RACI (§16), yanking (§13). |
| Mantenedores del paquete | Mapeo SemVer↔DSL (§4), breaking change policy (§10), deprecaciones (§11). |
| Autores de PR | Conventional Commits (§8), criterios de bump. |
| Reviewers | Validación de tipo de bump declarado en el commit. |
| Integradores / consumidores externos | Ventana de soporte (§12), política de TFM (§14). |
| Security | Yanking ante CVE (§13). |

---

## 3. SemVer 2.0.0 — especificación adoptada

Se adopta **Semantic Versioning 2.0.0** sin modificaciones. Especificación normativa: <https://semver.org/spec/v2.0.0.html>.

### 3.1 Formato

```text
MAJOR.MINOR.PATCH[-PRERELEASE][+BUILDMETADATA]
```

| Componente | Tipo | Significado |
|---|---|---|
| `MAJOR` | entero ≥ 0 | Cambios incompatibles en la API pública. |
| `MINOR` | entero ≥ 0 | Funcionalidad nueva retrocompatible. |
| `PATCH` | entero ≥ 0 | Correcciones retrocompatibles. |
| `PRERELEASE` | identificadores separados por `.` | Pre-release; versión inestable previa al estable. |
| `BUILDMETADATA` | identificadores separados por `.` | Metadatos de build; **no** afecta precedencia. |

### 3.2 Reglas de incremento

| Tipo de cambio | Bump | Reset |
|---|---|---|
| Cambio incompatible en API pública o contrato | MAJOR | MINOR=0, PATCH=0 |
| Funcionalidad nueva retrocompatible | MINOR | PATCH=0 |
| Bug fix retrocompatible | PATCH | — |
| Solo cambio interno sin impacto público | PATCH | — |

Versiones `0.y.z` (`MAJOR=0`) son consideradas inestables: cualquier cambio puede ser breaking. **El Motor DSL parte directamente desde `1.0.0`** — la API pública ya está estabilizada y la disciplina SemVer aplica desde el primer release.

### 3.3 Ejemplos del Motor DSL

| Cambio | Versión previa | Versión nueva | Justificación |
|---|---|---|---|
| Se agrega keyword `@if` al DSL, plantillas existentes siguen parseando | `1.2.0` | `1.3.0` | Funcionalidad retrocompatible. |
| Se corrige un off-by-one en `LayoutEngine.Wrap()` | `1.3.0` | `1.3.1` | PATCH; no cambia API ni comportamiento documentado. |
| Se renombra `IRenderer.Render(template)` a `IRenderer.RenderAsync(template, ct)` | `1.3.1` | `2.0.0` | Cambio en firma pública. |
| Se ajusta el orden de bytes en el rendering ESC/POS para alinear con perfil EPSON TM-T20III | `2.0.0` | `3.0.0` | Cambia bytes de salida; rompe trazabilidad y tests de cliente. |
| Se mejora performance del parser sin cambiar resultados | `3.0.0` | `3.0.1` | PATCH (`perf:` → PATCH; ver §8). |
| Se agrega overload nuevo `Parser.Parse(string, ParserOptions)` | `3.0.1` | `3.1.0` | API nueva retrocompatible. |
| Se elimina overload deprecado `Parser.ParseLegacy()` | `3.1.0` | `4.0.0` | Remoción de API pública. |

---

## 4. Mapeo del versionado al DSL

El Motor DSL expone **dos superficies públicas** que deben tratarse como contrato:

1. **API .NET pública** — tipos, métodos, propiedades, eventos, atributos públicos de los ensamblados `MotorDsl.Core`, `MotorDsl.Parser`, `MotorDsl.Rendering`, `MotorDsl.Extensions`.
2. **Sintaxis y semántica del DSL** — gramática JSON, nombres de nodos, atributos reconocidos, comportamiento de evaluación, perfiles de dispositivo y bytes generados por los renderers.

Ambas superficies aplican SemVer. Reglas obligatorias:

| Cambio | Bump | Notas |
|---|---|---|
| Cambio en gramática que rompe plantillas existentes (renombrar nodo, cambiar tipo de atributo, eliminar keyword) | **MAJOR** | Requiere CHANGELOG `Changed` + nota de migración. |
| Nuevo nodo, keyword o atributo retrocompatible (plantillas viejas siguen parseando con el mismo resultado) | **MINOR** | Documentar en `Added`. |
| Nueva opción de configuración con default que preserva el comportamiento previo | **MINOR** | Default = comportamiento histórico. |
| Bug fix en evaluación que **alinea** el comportamiento con el documentado | **PATCH** | Explicar en CHANGELOG por qué no es breaking. |
| Bug fix en evaluación que **cambia** un comportamiento previamente documentado o esperado | **MAJOR** | Aunque el comportamiento previo fuera incorrecto, si está documentado, romperlo es breaking. |
| Cambio en mensajes de error o severidad de logging | **PATCH** | Los mensajes no son contrato salvo declaración explícita. |
| Cambio en código de excepción / tipo de excepción lanzada | **MAJOR** | Los consumidores capturan tipos específicos. |
| Cambio en perfil de dispositivo (ESC/POS) que altera bytes de salida para una plantilla idéntica | **MAJOR** | Rompe trazabilidad de impresión, tests de byte-equality del consumidor y certificaciones fiscales. |
| Agregar un nuevo perfil de dispositivo sin tocar los existentes | **MINOR** | El perfil default no cambia. |
| Cambio interno (refactor, optimización) sin impacto observable | **PATCH** | Validar con tests de byte-equality antes del release. |
| Cambio en el formato serializado de `DocumentTemplate` (round-trip) | **MAJOR** | Es contrato si está expuesto. |
| Cambio en dependencias NuGet con bump MAJOR de la dependencia | **MAJOR** | Diamond dependency en consumidores. |
| Bump MINOR/PATCH de dependencia NuGet | **MINOR** o **PATCH** | Según el bump propio del cambio. |

### 4.1 Definición operativa de "API pública"

Es API pública todo lo que esté declarado con visibilidad `public` o `protected` en cualquiera de los ensamblados publicados. Lo declarado `internal` **no** es contrato, salvo que se marque explícitamente con `[InternalsVisibleTo]` para terceros.

### 4.2 Definición operativa de "DSL público"

Es DSL público todo nodo, keyword, atributo o regla descrita en la documentación oficial del DSL (`docs/04_diseño/` y reference de gramática). Las extensiones experimentales deben marcarse con prefijo `x-` y declararse explícitamente como inestables; no aplican SemVer.

---

## 5. Pre-releases

### 5.1 Sufijos permitidos

```text
MAJOR.MINOR.PATCH-alpha.N
MAJOR.MINOR.PATCH-beta.N
MAJOR.MINOR.PATCH-rc.N
```

| Sufijo | Significado | Estabilidad esperada |
|---|---|---|
| `-alpha.N` | Iteración temprana, API puede cambiar sin aviso. | Baja. Solo dogfooding interno. |
| `-beta.N` | Funcionalidad completa, falta validación amplia. | Media. Aceptado para integración temprana de socios. |
| `-rc.N` | Candidato a release. Sin cambios funcionales esperados. | Alta. Solo bug fixes admitidos hasta el estable. |

`N` es un entero ≥ 1, sin ceros a la izquierda.

### 5.2 Precedencia (SemVer §11)

```text
1.0.0-alpha < 1.0.0-alpha.1 < 1.0.0-alpha.2 < 1.0.0-beta < 1.0.0-beta.1 <
1.0.0-rc.1 < 1.0.0
```

Reglas formales:

- Una versión con pre-release tiene **menor** precedencia que la misma versión sin pre-release.
- Identificadores numéricos se comparan numéricamente; alfanuméricos lexicográficamente.

### 5.3 Política de promoción

| Etapa | Criterio para avanzar |
|---|---|
| `alpha` → `beta` | Funcionalidad completa, tests verdes, sin tareas de feature pendientes. |
| `beta` → `rc` | Sin bugs críticos abiertos, documentación actualizada, integradores piloto sin issues nuevos. |
| `rc` → estable | RC vive ≥ 7 días sin reportes de bug críticos; review final del Tech Lead. |

Los pre-releases se publican por tag (`v1.4.0-rc.1`) y siguen el mismo pipeline CI/CD que un release estable. **No** se promueve un pre-release renombrándolo: cada estable tiene su propio tag distinto del último RC.

### 5.4 Identificación con Git tags

Tag = versión literal con prefijo `v`. Ejemplos: `v1.4.0-alpha.3`, `v2.0.0-rc.1`, `v2.0.0`.

---

## 6. Build metadata

### 6.1 Formato

```text
1.4.0+sha.a1b2c3d
1.4.0+sha.a1b2c3d.20260425
2.0.0-rc.1+sha.f0e1d2c
```

Identificadores permitidos en `+BUILDMETADATA`:

- `sha.<7 chars del commit>` — SHA corto del commit fuente.
- `<YYYYMMDD>` — fecha de build en UTC, opcional.

### 6.2 Reglas

- No afecta precedencia (`1.4.0+sha.a1b2c3d` y `1.4.0+sha.f0e1d2c` son la **misma** versión a efectos de comparación SemVer).
- No se publica como tag Git separado: el tag es la versión limpia (`v1.4.0`); el sufijo `+...` se inyecta en el assembly y en el `.nupkg` `<RepositoryCommit>` para trazabilidad.
- NuGet **ignora** `+BUILDMETADATA` al resolver versiones; está presente solo para auditoría.

### 6.3 Inyección automática

MinVer (ver §8) toma el SHA del commit del tag y lo expone en `MinVerSha`. Se mapea a `<SourceRevisionId>` y queda incrustado en el `.nuspec` y en `AssemblyInformationalVersion`.

---

## 7. Branching — GitHub Flow

Estrategia: **GitHub Flow estricto**. No GitFlow. No `develop`. No `release/*`. No `hotfix/*`.

### 7.1 Topología

```text
main  ──●───●───●───●───●───●──→  (siempre liberable)
         \         /     \   /
          ●───●───●       ●─●
       feature/parser-trim
                     feature/escpos-codepage-cp858
```

| Rama | Origen | Destino | Vida |
|---|---|---|---|
| `main` | — | — | Permanente. Siempre verde y liberable. |
| `feature/<scope>-<descripcion>` | `main` | `main` (squash merge) | Hasta el merge del PR. |

### 7.2 Reglas

1. Toda rama de feature parte de `main` actualizado.
2. Nombre: `feature/<scope>-<descripcion-kebab>`. `<scope>` debe coincidir con un scope válido de §8.4.
3. Push a `main` directo está prohibido.
4. PR obligatorio. Squash merge obligatorio (un commit por feature en `main`).
5. El mensaje de squash debe respetar Conventional Commits (§8).
6. Tras el merge, la rama se elimina automáticamente (configuración del repo).

### 7.3 Hotfix

No existe rama `hotfix/*`. Un hotfix es una rama `feature/fix-<descripcion>` con commit `fix:` (o `fix!:` si es breaking). El proceso de release es el mismo que cualquier otro PATCH: PR → merge → tag.

### 7.4 Protección de `main`

| Regla | Valor |
|---|---|
| Require pull request before merging | Sí |
| Require approvals | 1 reviewer |
| Dismiss stale reviews when new commits are pushed | Sí |
| Require status checks to pass | Sí (CI verde, ver `pipeline-ci-cd_v1.0.md` §6) |
| Require branches to be up to date before merging | Sí |
| Require conversation resolution before merging | Sí |
| Require signed commits | Recomendado |
| Require linear history | Sí (squash garantiza linealidad) |
| Restrict who can push to matching branches | Solo administradores y bot de CI |
| Allow force pushes | No |
| Allow deletions | No |

---

## 8. Conventional Commits + auto-versioning

Especificación: <https://www.conventionalcommits.org/en/v1.0.0/>.

### 8.1 Formato del commit

```text
<tipo>(<scope>)!: <descripción imperativa, ≤72 chars>

<cuerpo opcional, separado por línea en blanco>

<footer opcional>
BREAKING CHANGE: <descripción del cambio incompatible>
Refs: #123
```

- `<tipo>` es obligatorio y restringido a la tabla §8.3.
- `<scope>` es obligatorio para `feat`, `fix`, `perf`, `refactor`. Restringido a la tabla §8.4.
- `!` después del tipo/scope marca breaking change.
- `BREAKING CHANGE:` en el footer es **equivalente** a `!` y prevalece como fuente de verdad.

### 8.2 Mensaje del squash merge

El título del PR es el commit final en `main` tras squash. **El reviewer debe verificar** que el título cumple esta convención antes de aprobar el merge.

### 8.3 Tipos permitidos y mapeo a SemVer

| Tipo | Bump SemVer | Aparece en CHANGELOG en sección | Uso |
|---|---|---|---|
| `feat` | **MINOR** | `Added` | Funcionalidad nueva. |
| `fix` | **PATCH** | `Fixed` | Bug fix retrocompatible. |
| `perf` | **PATCH** | `Changed` | Mejora de performance sin cambiar comportamiento. |
| `refactor` | **PATCH** | (no listado) | Refactor interno sin impacto público. |
| `docs` | **PATCH** | (no listado) | Solo documentación. |
| `test` | **PATCH** | (no listado) | Solo tests. |
| `build` | **PATCH** | (no listado) | Sistema de build, dependencias dev. |
| `ci` | **PATCH** | (no listado) | Pipeline CI/CD. |
| `chore` | **PATCH** | (no listado) | Tareas misceláneas. |
| `style` | **PATCH** | (no listado) | Formato sin cambio funcional. |
| `revert` | según tipo revertido | `Removed` o `Fixed` | Revertir un commit previo. |
| `feat!`, `fix!`, etc. | **MAJOR** | `Changed` o `Removed` | Breaking change. |
| Footer `BREAKING CHANGE:` | **MAJOR** | `Changed` o `Removed` | Equivalente a `!`. |

**Notas operativas:**

- `docs`, `test`, `build`, `ci`, `chore`, `style` no producen cambios en el binario publicado pero **sí** disparan PATCH si se libera entre dos features. En la práctica, no se libera por estos tipos solos.
- Un PR puede contener varios cambios; el squash merge los agrupa en un único mensaje. El bump se determina por el commit squash final, **no** por los commits intermedios de la rama feature.

### 8.4 Scopes válidos (Motor DSL)

| Scope | Cubre |
|---|---|
| `parser` | `MotorDsl.Parser`. Lexer, parser JSON DSL, validación sintáctica. |
| `evaluator` | `MotorDsl.Core` evaluación de expresiones, binding de datos. |
| `renderer` | `MotorDsl.Rendering` orquestación, `IRenderer`. |
| `escpos` | Renderer ESC/POS, perfiles de dispositivo, codepages. |
| `pdf` | Renderer PDF (cuando exista). |
| `dsl` | Gramática, definición de nodos, schema. |
| `validation` | Validaciones semánticas (post-parser). |
| `extensions` | `MotorDsl.Extensions`, DI, fluent API. |
| `core` | `MotorDsl.Core` general (modelos, contratos transversales). |
| `deps` | Bumps de dependencias NuGet. |
| `release` | Tareas administrativas de release. |

Scope no listado → el reviewer rechaza el PR o solicita extender la tabla en este documento (PR a `estrategia-versionado_v2.0.md`).

### 8.5 Footer `BREAKING CHANGE:`

Obligatorio cuando el PR introduce un cambio incompatible. Debe contener:

1. Qué cambia.
2. Por qué.
3. Migración requerida por el consumidor.

Ejemplo:

```text
feat(renderer)!: cambia firma de IRenderer.Render a async

BREAKING CHANGE: IRenderer.Render(DocumentTemplate) fue reemplazado por
IRenderer.RenderAsync(DocumentTemplate, CancellationToken). Los consumidores
que implementen IRenderer deben actualizar la firma. Los consumidores que
solo usan la implementación deben agregar `await` y propagar CancellationToken.

Refs: #487
```

### 8.6 Herramienta primaria: MinVer

**MinVer** (<https://github.com/adamralph/minver>) calcula la versión a partir de Git tags durante el build. Sin servidor, sin archivos de configuración versionados, sin commits de bump.

**Por qué MinVer:**

- Configuración mínima (un PackageReference por proyecto o vía `Directory.Build.props`).
- Determinismo: misma rev de Git → misma versión.
- Compatible con `dotnet pack` sin pasos extra en CI.
- Soporta pre-releases vía tags (`v1.4.0-rc.1`) y SHA en `+BUILDMETADATA`.

**Instalación (una sola vez, en `src/Directory.Build.props`):**

```xml
<Project>

  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
    <MinVerTagPrefix>v</MinVerTagPrefix>
    <MinVerDefaultPreReleaseIdentifiers>alpha.0</MinVerDefaultPreReleaseIdentifiers>
    <MinVerVerbosity>minimal</MinVerVerbosity>
    <MinVerAutoIncrement>minor</MinVerAutoIncrement>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="MinVer" Version="6.0.0" PrivateAssets="all" />
  </ItemGroup>

</Project>
```

**Comportamiento resultante:**

| Estado del repo | Versión calculada |
|---|---|
| HEAD está en tag `v1.4.0` | `1.4.0` |
| HEAD está en tag `v1.4.0-rc.1` | `1.4.0-rc.1` |
| HEAD = tag `v1.4.0` + 5 commits | `1.5.0-alpha.0.5` (auto-increment minor + height) |
| HEAD = tag `v1.4.0` + 5 commits + dirty | `1.5.0-alpha.0.5` con flag de warning |

**Bump explícito:** se controla creando un tag con la versión deseada. Para forzar un MAJOR sin esperar a que MinVer lo infiera por commits, se crea `git tag -s v2.0.0` directamente.

### 8.7 Alternativa: Nerdbank.GitVersioning

`Nerdbank.GitVersioning` (NBGV, <https://github.com/dotnet/Nerdbank.GitVersioning>) es la alternativa indicada cuando se necesita:

- Versionado declarativo en archivo `version.json` versionado en el repo (cada rama declara su versión).
- Cálculo de versión sin tags (basado en altura de commits desde un baseline).
- Integración con `nbgv` CLI para automatizar bumps en PRs.

**Trade-off:** mayor complejidad operativa, necesidad de mantener `version.json` por rama, lock-in con su CLI. Solo elegir si el equipo necesita versionado independiente por rama o pipelines multi-versión simultáneos.

**Decisión actual:** **MinVer**. Migración a NBGV requiere ADR explícito.

---

## 9. Mapeo Git tag ↔ paquete ↔ CHANGELOG

Tres artefactos, **un mismo identificador semántico**, todos inmutables.

### 9.1 Cadena de trazabilidad

```text
Git tag             →  Paquete NuGet                →  CHANGELOG
v1.2.3 (firmado)       MotorDsl.Core.1.2.3.nupkg       ## [1.2.3] - 2026-04-25
                       MotorDsl.Parser.1.2.3.nupkg
                       MotorDsl.Rendering.1.2.3.nupkg
                       MotorDsl.Extensions.1.2.3.nupkg
                       (en GitHub Packages)
```

| Artefacto | Inmutabilidad | Mecanismo |
|---|---|---|
| Tag Git | Inmutable. No se reescribe. | `git tag -s` firmado; protected tags en GitHub. |
| Paquete `.nupkg` | Inmutable. No se sobreescribe. | GitHub Packages rechaza re-push de la misma versión. |
| Entrada CHANGELOG | Inmutable post-release. | Editar versiones publicadas requiere nueva entrada. |

### 9.2 CHANGELOG.md

Formato: <https://keepachangelog.com/en/1.1.0/>. Secciones obligatorias por release:

```markdown
## [Unreleased]

### Added
### Changed
### Deprecated
### Removed
### Fixed
### Security

## [1.2.3] - 2026-04-25

### Added
- (parser) Soporte para keyword `@if` en plantillas. (#487)

### Fixed
- (escpos) Corrige codepage CP858 en perfil EPSON TM-T20III. (#492)
```

Reglas:

1. Toda PR que afecte el binario debe agregar entrada bajo `[Unreleased]`. El reviewer lo valida.
2. Al crear el tag de release, el commit previo mueve `[Unreleased]` → `[X.Y.Z] - YYYY-MM-DD`.
3. Las entradas se ordenan por scope entre paréntesis cuando es relevante.
4. Issues y PRs se referencian con `(#N)`.

### 9.3 Firmado de tags (recomendado)

```bash
git tag -s v1.2.3 -m "Release 1.2.3"
git push origin v1.2.3
```

`-s` requiere clave GPG configurada en `git config user.signingkey`. La firma queda verificable en GitHub UI con badge `Verified`. La protección de tags en GitHub puede exigir tags firmados para releases producción.

---

## 10. Breaking change policy

### 10.1 Definición operativa

Un cambio es **breaking** si rompe **al menos uno** de:

1. **API .NET pública** — cualquiera de:
   - Eliminación de tipo, método, propiedad, evento, campo público.
   - Cambio de firma (parámetros, tipo de retorno, tipo genérico, restricciones).
   - Cambio de visibilidad de `public` a `internal` o `private`.
   - Cambio de tipo de excepción lanzada en condiciones documentadas.
   - Cambio de namespace de un tipo público.
2. **Sintaxis DSL pública** — eliminación o renombrado de nodo/atributo, cambio de tipo de un atributo, cambio en la semántica de evaluación documentada.
3. **Contrato serializado** — cambio en el formato JSON de `DocumentTemplate` que rompe round-trip con artefactos producidos por la versión anterior.
4. **Comportamiento documentado de evaluación** — cambio en el resultado para una entrada idéntica que esté cubierto por la documentación oficial o por el contrato de tests.
5. **Bytes generados por renderers** — cambio en el output binario para una plantilla idéntica con el mismo perfil de dispositivo.
6. **Bump MAJOR de una dependencia NuGet** que genere conflicto diamond en consumidores que también dependan de esa lib.

### 10.2 Período de aviso antes de remover una API deprecada

Mínimo: **2 versiones MINOR** posteriores al anuncio **o 6 meses calendario**, lo que ocurra **después**.

Ejemplo:

```text
v1.4.0 (2026-01-15)  Se deprecia Parser.ParseLegacy() con [Obsolete].
v1.5.0 (2026-03-01)  Sigue disponible. Aviso reiterado en release notes.
v1.6.0 (2026-05-10)  +120 días desde v1.4.0.
v2.0.0 (2026-08-20)  ≥ 6 meses desde v1.4.0 y ≥ 2 MINOR. Se elimina.
```

Si en el ejemplo `v2.0.0` se hubiera lanzado en `2026-04-30` (≥ 2 MINOR pero < 6 meses), no podría removerse `ParseLegacy()`. Hay que esperar al criterio temporal.

### 10.3 Comunicación

Antes de la remoción se requiere:

1. Entrada en CHANGELOG sección `Deprecated` en el release que introduce el `[Obsolete]`.
2. Atributo `[Obsolete("Use X instead. Removed in v3.0.0. See https://github.com/<org>/<repo>/issues/<N>", false)]` en el código.
3. Issue de tracking en GitHub con label `breaking-change` y la versión target en el milestone.
4. Mención en release notes de cada MINOR intermedio recordando la deprecación pendiente.

---

## 11. Deprecation policy

### 11.1 Cómo marcar

```csharp
// Fase 1 — Aviso (warning, false)
[Obsolete("Use Parser.Parse(string, ParserOptions) instead. Removed in v3.0.0. See https://github.com/<org>/<repo>/issues/487", false)]
public DocumentTemplate ParseLegacy(string json) { ... }
```

```csharp
// Fase 2 — Escalada a error (true) — solo en la última MINOR antes de la MAJOR que remueve
[Obsolete("Use Parser.Parse(string, ParserOptions) instead. Will be removed in v3.0.0.", true)]
public DocumentTemplate ParseLegacy(string json) { ... }
```

```csharp
// Fase 3 — Removido en MAJOR siguiente. La API deja de existir.
```

### 11.2 Cuándo escalar `error: true`

- En el **último release MINOR** antes de la MAJOR que remueve la API, si el período de aviso de §10.2 ya se cumplió.
- Forzar `error: true` antes del período mínimo está **prohibido**: el consumidor pierde la posibilidad de migración progresiva.

### 11.3 Documentación

Cada API deprecada debe figurar en:

| Lugar | Qué contiene |
|---|---|
| CHANGELOG sección `Deprecated` (release que introduce el aviso) | Nombre de la API, reemplazo sugerido, versión target de remoción. |
| Release notes del GitHub Release | Mismo contenido, formato narrativo. |
| Issue de tracking | Discusión técnica, alternativas, PRs relacionados. |
| Atributo `[Obsolete]` | Mensaje accionable y URL al issue. |

---

## 12. Ventana de soporte

| Banda | Política |
|---|---|
| Última MAJOR (`vN.x.x`) | Recibe features (MINOR), fixes (PATCH) y security fixes. Soporte completo. |
| Penúltima MAJOR (`v(N-1).x.x`) | Solo security fixes durante **6 meses** desde el release de `vN.0.0`. No hay backport de features. |
| Anteriores (`v(N-2).x.x` y previos) | Sin soporte. Consumidores deben migrar. |

### 12.1 Soporte intra-MAJOR

Dentro de la MAJOR vigente (`vN`), solo la **última MINOR liberada** recibe PATCH proactivos. Versiones MINOR anteriores reciben PATCH **únicamente** si:

1. Hay un **CVE crítico** (CVSS ≥ 7.0) que afecte específicamente esa MINOR, **y**
2. La nueva MINOR no contiene una mitigación trivial (ej. el consumidor puede actualizar la MINOR sin breaking).

En la práctica: el path normal es "actualizá a la última MINOR".

### 12.2 Anuncio de fin de soporte

El cierre de soporte de una MAJOR se anuncia en:

1. CHANGELOG del release de la MAJOR sucesora (sección dedicada).
2. README del repo (sección "Supported versions").
3. Release notes del último PATCH de la MAJOR retirada, marcada como "End of life".

---

## 13. Política de yanking / unlisting ante CVE

### 13.1 Definiciones

- **Yanking** (en GitHub Packages): eliminar la versión del feed. **El consumidor no puede instalarla**, los lockfiles existentes fallan a resolver.
- **Unlisting** (semántica NuGet.org): no aplica de forma idéntica en GitHub Packages, que solo soporta delete. Documentado para consistencia conceptual.

### 13.2 Criterios para yankear

Yankear es una acción **destructiva** que rompe builds de consumidores. Solo procede si **se cumplen todas** las siguientes condiciones:

1. CVE de severidad **crítica** (CVSS ≥ 9.0) o **alta** con vector remoto sin autenticación.
2. Existe una versión de reemplazo publicada **simultáneamente** con la mitigación (ej. `1.4.3` se yankea, `1.4.4` está disponible con el fix).
3. Tech Lead **y** Security aprueban (RACI §16).
4. Hay GitHub Security Advisory abierta y pública con CVE asignado o número provisional.

CVE de severidad media o baja → **no se yankea**. Se publica una nueva PATCH con el fix y se documenta en `Security`.

### 13.3 Procedimiento de yanking

1. Verificar criterios §13.2.
2. Publicar versión de reemplazo (ver §15) **antes** de yankear.
3. Eliminar la versión vulnerable: `gh api -X DELETE /orgs/<owner>/packages/nuget/<package>/versions/<version-id>` para los 4 paquetes simultáneamente.
4. Actualizar CHANGELOG: marcar la versión yankeada con `[YANKED]` y agregar nota explicando el motivo y la versión sustituta.
5. Crear / actualizar GitHub Security Advisory público referenciando la versión yankeada y la versión segura.
6. Notificar en release notes del fix (`Security` section) y en el README.

### 13.4 Registro en CHANGELOG

```markdown
## [1.4.3] - 2026-04-22 [YANKED]

Yankeada el 2026-04-23 por CVE-2026-XXXXX (RCE en deserialización del parser).
Reemplazada por 1.4.4. No usar.

### Added
- (parser) ...

## [1.4.4] - 2026-04-23

### Security
- (parser) Mitiga CVE-2026-XXXXX (RCE en deserialización). Reemplaza 1.4.3 [YANKED].
```

La entrada de la versión yankeada **no se elimina** del CHANGELOG. El historial es auditable.

---

## 14. Decisión arquitectónica: TFM único `net10.0`

### 14.1 Decisión

Los cuatro proyectos publicables (`MotorDsl.Core`, `MotorDsl.Parser`, `MotorDsl.Rendering`, `MotorDsl.Extensions`) declaran:

```xml
<TargetFramework>net10.0</TargetFramework>
```

**No se hace multi-targeting.** No hay `<TargetFrameworks>` con múltiples valores. No hay condicionales por TFM en el código.

### 14.2 Justificación

La librería está diseñada para .NET 10 LTS y aprovecha activamente:

- **Static abstract members en interfaces** (parser y evaluator usan dispatch genérico estático).
- **Generic math** (cálculos de layout y conversiones de unidades).
- **Primary constructors** (modelos del DSL).
- **Collection expressions** y mejoras de pattern matching.
- **`UnsafeAccessor`** y APIs de bajo nivel para el renderer ESC/POS.
- **Performance del JIT/AOT-ready** disponible solo en runtime moderno.

Soportar `net8.0` o `netstandard2.1` requeriría:

- Polyfills extensos o duplicación de implementaciones.
- Matriz de tests N×TFM (tests actuales: 185 → con multi-target ≥ 370).
- Análisis condicional por compilación (`#if NET10_0_OR_GREATER`).
- Coordinar releases con regresiones específicas de cada TFM.

### 14.3 Trade-offs explícitos

| Trade-off | Costo asumido | Beneficio obtenido |
|---|---|---|
| Consumers atados a .NET 10 | Excluye proyectos legacy (.NET 8, .NET Framework, Xamarin clásico). | Acceso pleno a APIs modernas; código más limpio; menor superficie de bugs. |
| No portabilidad a `netstandard` | Imposible consumir desde libs que aún targeteen netstandard. | Sin polyfills; mejor performance. |
| Releases acoplados al ciclo .NET | Cada nueva LTS de .NET requiere migración planificada. | Política de soporte simple y predecible. |
| Audiencia inicial reducida | Solo equipos en .NET 10. | Menor superficie de testing; releases más rápidos. |

### 14.4 Política de migración a futuras versiones de .NET

Cuando .NET 11 LTS esté disponible (estimado Q4 2027 según ciclo Microsoft), se aplica:

1. Se publica una **nueva MAJOR** del paquete con `<TargetFramework>net11.0</TargetFramework>`.
2. La MAJOR previa (sobre `net10.0`) entra en período de soporte de §12 (security fixes 6 meses).
3. **No se mantiene compatibilidad hacia atrás.** No se retro-publican fixes de `net11.0` a la MAJOR `net10.0` salvo CVE crítico.
4. La decisión de cuándo migrar la queda en el ADR del momento; no se anticipa en este documento.

### 14.5 Consecuencia para SemVer

Cambiar el TFM (ej. `net10.0` → `net11.0`) es **siempre breaking** y dispara MAJOR. No se discute caso por caso.

---

## 15. Proceso de release (paso a paso operativo)

### 15.1 Precondiciones

Antes de iniciar el proceso, verificar:

| Precondición | Cómo se valida |
|---|---|
| Todas las historias del sprint cumplen DoD | [`definition-of-done_v1.0.md`](../08_calidad_y_pruebas/definition-of-done_v1.0.md) §2 |
| Pipeline CI verde en `main` | GitHub Actions, último run de `ci.yml` |
| Cobertura de tests ≥ 70 % | [`pipeline-ci-cd_v1.0.md`](pipeline-ci-cd_v1.0.md) §6 |
| `[Unreleased]` del CHANGELOG tiene contenido | Inspección manual |
| No hay deprecaciones por escalar a `error: true` que estén pendientes | Búsqueda de `[Obsolete]` en src |

### 15.2 Pasos

**Paso 1 — Cierre del CHANGELOG**

PR a `main` que mueva la sección `[Unreleased]` a `[X.Y.Z] - YYYY-MM-DD`. La versión `X.Y.Z` se decide por el conjunto acumulado de Conventional Commits desde el último tag (ver §8.3).

```markdown
## [Unreleased]

## [1.2.3] - 2026-04-25

### Added
- (parser) ...
```

Squash merge con mensaje: `chore(release): prepare 1.2.3`.

**Paso 2 — CI verde post-merge**

Esperar que el run de `ci.yml` sobre `main` (post-merge) esté verde. Si falla, **no se etiqueta**: se corrige primero.

**Paso 3 — Tag firmado**

```bash
git checkout main
git pull origin main
git tag -s v1.2.3 -m "Release 1.2.3"
git push origin v1.2.3
```

El tag debe apuntar al commit del paso 1.

**Paso 4 — Release pipeline**

Push del tag dispara el workflow de publicación (ver `pipeline-ci-cd_v1.0.md` §5.10). Acciones automáticas:

- MinVer detecta el tag → versión `1.2.3` (sin pre-release, sin build metadata adicional).
- `dotnet pack -c Release` produce los 4 `.nupkg` con versión `1.2.3`.
- `dotnet nuget push` los publica en GitHub Packages.

**Paso 5 — GitHub Release**

Una vez publicados los paquetes:

```bash
gh release create v1.2.3 \
  --title "v1.2.3" \
  --notes-from-tag \
  --verify-tag
```

Las release notes se generan a partir de los Conventional Commits desde el tag previo. Editar para agregar contexto de migración si hubo deprecaciones.

**Paso 6 — Verificación**

| Verificación | Cómo |
|---|---|
| Los 4 paquetes están publicados en GitHub Packages con versión `1.2.3` | UI de Packages del repo |
| El GitHub Release está publicado y referencia el tag | UI de Releases |
| El CHANGELOG en `main` refleja la versión publicada | `git show main:CHANGELOG.md` |
| Un consumidor puede instalar `dotnet add package MotorDsl.Extensions --version 1.2.3` | Smoke test desde un proyecto sample |

**Paso 7 — Comunicación post-release**

- Anuncio en canal interno con link al GitHub Release.
- Si hay breaking changes o deprecaciones, mensaje específico a integradores.

### 15.3 Pre-releases

Mismo procedimiento, con tag `vX.Y.Z-<sufijo>.N`. La diferencia: las release notes se generan, pero el GitHub Release se marca como "pre-release" (`gh release create --prerelease`).

### 15.4 Recuperación ante falla del pipeline

| Situación | Acción |
|---|---|
| Pipeline falla en `pack` | Borrar tag local y remoto (`git tag -d` + `git push --delete`), corregir, retaguear con la **misma versión** si el commit no se pushed packages. Si **alguno** de los 4 paquetes ya se publicó, **incrementar PATCH** y publicar `1.2.4`. |
| Paquete publicado tiene un bug | **Nunca** reescribir. Publicar `1.2.4` con el fix. Si es CVE, ver §13. |
| Tag firmado con clave incorrecta | Borrar tag, recrear con la clave correcta **antes** de que se haya disparado el pipeline. Si ya disparó: avanzar con PATCH. |

---

## 16. RACI ligero del release

| Actividad | Responsable (R) | Aprobador (A) | Consultado (C) | Informado (I) |
|---|---|---|---|---|
| Decidir tipo de bump (vía Conventional Commits) | Autor del PR | Reviewer | — | Release manager |
| Validar CHANGELOG `[Unreleased]` durante el PR | Autor del PR | Reviewer | — | — |
| Cierre de `[Unreleased]` → `[X.Y.Z]` | Release manager | Tech Lead | Autores de PRs incluidos | Equipo |
| Crear tag firmado | Release manager | Tech Lead | — | Equipo |
| Disparar pipeline de publicación | Release manager (push del tag) | — | — | DevOps |
| Publicar GitHub Release y release notes | Release manager | — | Tech Lead (si hay breaking) | Equipo, integradores |
| Comunicar a integradores externos | Release manager | — | Producto | Integradores |
| Yanking de versión por CVE | Release manager | Tech Lead **+** Security | DevOps | Equipo, consumidores afectados |
| Decisión de extender ventana de soporte | Tech Lead | Producto | DevOps | Equipo |

`R` = ejecuta. `A` = autoriza. `C` = aporta antes. `I` = recibe el resultado.

---

## 17. Referencias

### 17.1 Externas

- Semantic Versioning 2.0.0 — <https://semver.org/spec/v2.0.0.html>
- Conventional Commits 1.0.0 — <https://www.conventionalcommits.org/en/v1.0.0/>
- Keep a Changelog 1.1.0 — <https://keepachangelog.com/en/1.1.0/>
- MinVer — <https://github.com/adamralph/minver>
- Nerdbank.GitVersioning — <https://github.com/dotnet/Nerdbank.GitVersioning>
- GitHub Flow — <https://docs.github.com/en/get-started/using-github/github-flow>
- GitHub Packages (NuGet) — <https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry>
- GitHub Security Advisories — <https://docs.github.com/en/code-security/security-advisories>
- .NET 10 release notes — <https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/overview>

### 17.2 Internas

- [`pipeline-ci-cd_v1.0.md`](pipeline-ci-cd_v1.0.md) — Pipeline CI/CD (§7 versionado en pipeline).
- [`entornos-deploy_v1.0.md`](entornos-deploy_v1.0.md) — Entornos (§9 versionado, §6.3 paquetes publicados).
- [`guia-publicacion-nuget_v1.0.md`](guia-publicacion-nuget_v1.0.md) — Empaquetado y consumo (§8 tabla SemVer).
- [`definition-of-done_v1.0.md`](../08_calidad_y_pruebas/definition-of-done_v1.0.md) — Gates de calidad como precondición de release.
- [`README.md`](../README.md) — Sección DevOps.

---

## 18. Control de cambios

| Versión | Fecha | Autor | Cambios |
|---|---|---|---|
| 1.0 | 2026-04-25 | DevOps / Release Engineering | Versión inicial. Reemplaza archivo corrupto previo que contenía contenido erróneo de `entornos-deploy_v1.0.md`. Define SemVer 2.0.0, Conventional Commits, MinVer, GitHub Flow, política de breaking changes, deprecación, soporte y yanking; TFM único `net10.0` como decisión arquitectónica. |

---
