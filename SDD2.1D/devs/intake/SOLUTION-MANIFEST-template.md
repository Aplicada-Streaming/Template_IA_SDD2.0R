# SOLUTION-MANIFEST-template

Plantilla metodológica SDD 2.1 para producir el documento `SOLUTION-MANIFEST-<nombre-solucion-kebab>_v1.0.md` durante la fase de intake. El manifiesto declara la jerarquía de proyectos de una solución: enumera los proyectos, su tipo D8, su rol, sus dependencias y sus nombres de código. Es el primer insumo que lee el orquestador, antes de cualquier otro intake, y es la fuente única de verdad de la enumeración de proyectos. El `PROJECT-README` refleja esta enumeración pero no la reabre; ante divergencia, manda el manifiesto.

## Guía de uso de esta plantilla

1. Copiar este archivo como `SOLUTION-MANIFEST-<nombre-solucion-kebab>_v1.0.md` en `/SDD2.1D/devs/intake/`.
2. Completar el bloque de solución con los valores reales.
3. Enumerar todos los proyectos en la tabla de proyectos, uno por fila, declarando para cada uno su valor D8 (uno de los 8 cerrados), su rol, su bandera `redistribuible` y sus dependencias hacia otros proyectos de la misma solución.
4. Verificar que el grafo de dependencias es acíclico y que no hay colisiones de nombre.
5. Borrar los bloques `Ejemplo aplicado` y `Caso degenerado` una vez completado, dejando solo el contenido productivo de la solución.
6. Validar el checklist final antes de pasar el manifiesto al orquestador.

Una solución de un solo proyecto es válida y es el caso degenerado: reproduce el comportamiento del template de tipo único. El manifiesto, en ese caso, tiene una sola fila de proyecto.

---

## §1 Bloque de solución

Bloque obligatorio al inicio del documento. Reproducir y completar los placeholders:

| Campo | Valor |
|---|---|
| Nombre de solución | [Nombre legible de la solución] |
| `nombre-solucion-kebab` | [slug derivado del nombre, kebab-case lowercase] |
| `NombreSolucionCodigo` | [forma PascalCase del nombre de la solución] |
| Proyecto principal | [`nombre-proyecto-kebab` del proyecto cabeza] |
| Intake de negocio | `PROJECT-BRIEF-<nombre-solucion-kebab>_v1.0.md` |
| Intake técnico | `PROJECT-README-<nombre-solucion-kebab>_v1.0.md` |
| Documento | `SOLUTION-MANIFEST-<nombre-solucion-kebab>_v1.0.md` |
| Versión | 1.0 |
| Fecha | [YYYY-MM-DD] |
| Estado | Borrador / En revisión / Aprobado |

### §1.1 Perfil de convención de nombres

Configuración que el orquestador aplica de forma reproducible para derivar los nombres de código de cada proyecto. Declarar una vez por solución:

| Parámetro | Valor por defecto | Notas |
|---|---|---|
| Forma del nombre de solución en código | PascalCase | `NombreSolucionCodigo` derivado del nombre legible |
| Separador de segmentos | `.` | Separa la raíz de la solución del sufijo de rol |
| Prefijo de paquetes redistribuibles | `Aplicada` | Reemplaza la raíz de la solución cuando `redistribuible: true` |

---

## §2 Tabla de proyectos

Una fila por proyecto. Todos los campos son obligatorios salvo Dependencias, que puede quedar vacío para proyectos sin dependencias.

| `nombre-proyecto-kebab` | `nombre-proyecto-codigo` | `project_type` (D8) | Rol en la solución | `redistribuible` | Dependencias | Path `/src` |
|---|---|---|---|---|---|---|
| [kebab] | [`<NombreSolucionCodigo>.<Sufijo>`] | [uno de los 8 D8] | [una frase] | [true / false] | [lista de kebab o vacío] | [`src/<NombreProyectoCodigo>/`] |

Valores cerrados de `project_type` (D8), exactamente 8:

```text
library, web-monolith, web-microservices, desktop-app, mobile-app-maui, rest-api, cli-tool, worker-service
```

### §2.1 Regla de nombres de código

1. El nombre de código de cada proyecto se forma como `<NombreSolucionCodigo>.<Sufijo>`, donde `<Sufijo>` identifica el rol del proyecto.
2. Si `redistribuible: true`, el nombre arranca con el prefijo de organización del perfil (`Aplicada` por defecto) en lugar de la raíz de la solución, porque un paquete reusable necesita un espacio de nombres estable e independiente de la solución que lo consume. Por ejemplo, en la solución `GestionDeTurnos`, un paquete de validaciones reusable se llama `Aplicada.Validaciones`, no `GestionDeTurnos.Validaciones`.
3. El `<Sufijo>` se orienta por el `project_type` y el rol. Mapa orientativo, no cerrado:

| `project_type` | Sufijo orientativo |
|---|---|
| `rest-api` | `.WebApi` o `.Api` |
| `web-monolith` | `.Web` |
| `worker-service` | `.Worker` |
| `cli-tool` | `.Cli` |
| `desktop-app` | `.Ui` o `.Desktop` |
| `mobile-app-maui` | `.Mobile` |
| `library` | `.Core`, `.Abstractions`, `.Domain`, `.Infrastructure` u otro rol |
| `web-microservices` | un proyecto por servicio bajo `<NombreSolucionCodigo>.Services.<Servicio>` más `.Gateway` y `.BuildingBlocks` |

La regla se expresa de forma agnóstica de stack a propósito. El perfil de convención de §1.1 es donde una solución concreta materializa la convención de su ecosistema.

### §2.2 Derivación de nombres (a cargo del orquestador)

- `nombre-solucion-kebab` y cada `nombre-proyecto-kebab` se derivan con el algoritmo de normalización del `master-prompt.md` §3: lowercase, espacios a guion medio, acentos a su equivalente sin acento, eliminación de caracteres no ascii/dígito/guion, colapso de guiones, recorte de guiones extremos.
- `NombreSolucionCodigo` se obtiene en PascalCase del nombre legible.
- `nombre-proyecto-codigo` se compone por la regla de §2.1.

---

## §3 Grafo de dependencias

Las dependencias declaradas en la tabla de §2 forman un grafo dirigido. El orquestador deriva de él el orden topológico de generación y de construcción: primero los proyectos sin dependencias, luego los que dependen solo de proyectos ya resueltos. Los proyectos del mismo nivel topológico pueden generarse en paralelo.

Representar el grafo como referencia visual (opcional pero recomendado):

```text
[proyecto-sin-deps]  ->  [proyecto-que-depende]  ->  [proyecto-principal]
```

---

## §4 Validaciones bloqueantes

El orquestador detiene la cadena y reporta si alguna de estas condiciones no se cumple:

- Algún `project_type` no pertenece al conjunto cerrado D8.
- No hay exactamente un proyecto principal (hay cero o más de uno).
- Dos proyectos colisionan en `nombre-proyecto-kebab` o en `nombre-proyecto-codigo`.
- Una dependencia apunta a un proyecto que no existe en la tabla.
- El grafo de dependencias contiene un ciclo.
- El `PROJECT-README` declara una lista de proyectos o un `project_type` que diverge de este manifiesto.

---

## §5 Ejemplo aplicado (solución multi-proyecto)

Bloque de solución:

| Campo | Valor |
|---|---|
| Nombre de solución | Gestión de Turnos |
| `nombre-solucion-kebab` | `gestion-de-turnos` |
| `NombreSolucionCodigo` | `GestionDeTurnos` |
| Proyecto principal | `gestion-de-turnos-api` |
| Intake de negocio | `PROJECT-BRIEF-gestion-de-turnos_v1.0.md` |
| Intake técnico | `PROJECT-README-gestion-de-turnos_v1.0.md` |

Perfil de convención: PascalCase; separador `.`; prefijo de redistribuibles `Aplicada`.

Tabla de proyectos:

| `nombre-proyecto-kebab` | `nombre-proyecto-codigo` | `project_type` | Rol | `redistribuible` | Dependencias | Path `/src` |
|---|---|---|---|---|---|---|
| `gestion-de-turnos-api` | `GestionDeTurnos.WebApi` | `rest-api` | API pública de turnos (principal) | false | `gestion-de-turnos-domain`, `aplicada-validaciones` | `src/GestionDeTurnos.WebApi/` |
| `gestion-de-turnos-domain` | `GestionDeTurnos.Domain` | `library` | Dominio y reglas de negocio compartidas | false | `aplicada-validaciones` | `src/GestionDeTurnos.Domain/` |
| `gestion-de-turnos-notificaciones` | `GestionDeTurnos.Worker` | `worker-service` | Envío asincrónico de recordatorios | false | `gestion-de-turnos-domain` | `src/GestionDeTurnos.Worker/` |
| `aplicada-validaciones` | `Aplicada.Validaciones` | `library` | Paquete reusable de validaciones | true | — | `src/Aplicada.Validaciones/` |

Grafo de dependencias:

```text
aplicada-validaciones  ->  gestion-de-turnos-domain  ->  gestion-de-turnos-api
                       \                              \-> gestion-de-turnos-notificaciones
                        \-> gestion-de-turnos-api
```

Orden topológico:

```text
nivel 0: aplicada-validaciones
nivel 1: gestion-de-turnos-domain
nivel 2: gestion-de-turnos-api, gestion-de-turnos-notificaciones   (paralelizables)
```

---

## §6 Caso degenerado (solución de un proyecto)

Una solución con un único proyecto reproduce el comportamiento del template de tipo único.

Bloque de solución:

| Campo | Valor |
|---|---|
| Nombre de solución | Parser CSV |
| `nombre-solucion-kebab` | `parser-csv` |
| `NombreSolucionCodigo` | `ParserCsv` |
| Proyecto principal | `parser-csv` |
| Intake de negocio | `PROJECT-BRIEF-parser-csv_v1.0.md` |
| Intake técnico | `PROJECT-README-parser-csv_v1.0.md` |

Tabla de proyectos:

| `nombre-proyecto-kebab` | `nombre-proyecto-codigo` | `project_type` | Rol | `redistribuible` | Dependencias | Path `/src` |
|---|---|---|---|---|---|---|
| `parser-csv` | `ParserCsv.Core` | `library` | Librería de parseo (única y principal) | false | — | `src/ParserCsv.Core/` |

El orquestador recorre un solo proyecto; el resultado equivale a la ejecución actual del template contra un único `project_type`.

---

## §7 Checklist de completitud del manifiesto

Verificar antes de pasar el manifiesto al orquestador. Todos los ítems deben estar tildados.

- [ ] El bloque de solución tiene nombre, `nombre-solucion-kebab`, `NombreSolucionCodigo`, proyecto principal y referencias de intake completos.
- [ ] El perfil de convención de nombres está declarado (forma PascalCase, separador, prefijo de redistribuibles).
- [ ] La tabla de proyectos tiene al menos una fila y todos los campos obligatorios completos.
- [ ] Cada `project_type` pertenece al conjunto cerrado D8 de 8 valores.
- [ ] Hay exactamente un proyecto principal.
- [ ] No hay colisiones de `nombre-proyecto-kebab` ni de `nombre-proyecto-codigo`.
- [ ] Cada dependencia referencia un proyecto existente en la tabla.
- [ ] El grafo de dependencias es acíclico.
- [ ] Cada proyecto marcado `redistribuible: true` arranca su nombre de código con el prefijo de organización del perfil.
- [ ] El control de cambios refleja la versión y fecha del documento.

---

## Control de cambios

| Versión | Fecha | Cambios | Autor |
|---|---|---|---|
| 1.0 | [YYYY-MM-DD] | Manifiesto inicial de la solución | [Autor] |
