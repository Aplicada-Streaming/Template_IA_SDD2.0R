# Contratos del Motor — Librería DSL de Renderizado

**Archivo:** contratos-del-motor_v1.1.md
**Versión:** 1.1
**Fecha:** 2026-04-25
**Autor:** Equipo Técnico
**Estado:** Borrador
**Glosario:** [docs/03_ux-ui/glosario_v1.0.md](../03_ux-ui/glosario_v1.0.md)

---

## 1. Propósito

Este documento define los contratos públicos del Motor DSL como librería. Establece las interfaces, modelos de entrada/salida y puntos de extensión que permiten a aplicaciones consumidoras integrar el motor de renderizado de documentos.

Los contratos aquí definidos constituyen la fuente de verdad para:

- Uso del motor desde aplicaciones externas (.NET MAUI, servicios, etc.)
- Implementación de renderizadores personalizados
- Integración con perfiles de dispositivo
- Testing e interoperabilidad

---

## 2. Convenciones generales

**Namespace base sugerido**

```text
MotorDsl
```

**Principios**

* Interfaces orientadas a contratos (DIP)
* Modelos independientes de infraestructura
* Sin dependencia de UI o frameworks específicos
* Extensible mediante inyección de dependencias
* Tipado fuerte en todos los contratos

**Reglas**

* No acoplar contratos a implementaciones concretas
* No exponer detalles internos del pipeline
* Mantener modelos serializables cuando aplique
* Evitar dependencias circulares

---

## 3. Contrato principal — Motor de documentos

### IDocumentEngine

```csharp
public interface IDocumentEngine
{
    RenderResult Render(
        string templateDsl,
        object data,
        DeviceProfile profile);

    RenderResult Render(
        DocumentTemplate template,
        object data,
        DeviceProfile profile);
}
```

**Responsabilidad**

* Orquestar todo el pipeline de renderizado
* Aceptar DSL o plantilla ya parseada
* Devolver el resultado final renderizado

---

## 4. Modelo de entrada

### DocumentTemplate

```csharp
public class DocumentTemplate
{
    public string Id { get; set; }
    public string Version { get; set; }
    public DocumentNode Root { get; set; }
}
```

**Responsabilidad**

Representar una plantilla ya parseada y validada.

---

### DeviceProfile

```csharp
public class DeviceProfile
{
    public string Name { get; set; }
    public int Width { get; set; }
    public string RenderTarget { get; set; } // escpos | ui | text | pdf
    public Dictionary<string, object> Capabilities { get; set; }
    public EncodingSpec Encoding { get; set; }
}

public class EncodingSpec
{
    public string Default { get; set; }                  // ej. "CP858"
    public IList<string> Supported { get; set; }         // ej. ["CP437","CP858","ISO-8859-1"]
    public IList<string> FallbackChain { get; set; }     // ej. ["CP858","ISO-8859-1","ASCII"]
    public bool SupportsUtf8 { get; set; }               // raro en térmicas
}
```

**Responsabilidad**

Definir las capacidades, restricciones y matriz de encoding del dispositivo destino. La matriz de encoding sigue lo definido en [representacion-documento-escpos_v1.1.md](../03_ux-ui/representacion-documento-escpos_v1.1.md) §8.

---

## 5. Modelo abstracto del documento

### DocumentNode

```csharp
public abstract class DocumentNode
{
    public string Type { get; set; }
    public List<DocumentNode> Children { get; set; }
}
```

---

### Tipos derivados (ejemplos)

```csharp
public class TextNode : DocumentNode
{
    public string Text { get; set; }
}

public class TableNode : DocumentNode
{
    public List<string> Headers { get; set; }
    public List<List<string>> Rows { get; set; }
    public List<ColumnSpec> Columns { get; set; }   // anchos y overflow por columna
}

public class ColumnSpec
{
    public string Width { get; set; }       // numérico ("12") o porcentual ("30%")
    public string Overflow { get; set; }    // "wrap" | "truncate" | "ellipsis"
    public string Alignment { get; set; }   // "left" | "center" | "right"
}

public class ConditionalNode : DocumentNode
{
    public string Expression { get; set; }
    public DocumentNode TrueBranch { get; set; }
    public DocumentNode FalseBranch { get; set; }
}
```

---

## 6. Contrato de renderización

### IRenderer

```csharp
public interface IRenderer
{
    string Target { get; }
    RenderResult Render(DocumentNode document, DeviceProfile profile);
}
```

**Responsabilidad**

* Transformar el modelo abstracto en una salida específica
* Cada renderizador soporta un target distinto

---

### RenderResult

```csharp
public class RenderResult
{
    public string Target { get; set; }
    public object Output { get; set; }
    public IList<EngineDiagnostic> Diagnostics { get; set; }

    // Vistas filtradas (azúcar sintáctico, no almacenamiento separado)
    public IEnumerable<EngineDiagnostic> Errors
        => Diagnostics?.Where(d => d.Severity == DiagnosticSeverity.Error)
           ?? Enumerable.Empty<EngineDiagnostic>();

    public IEnumerable<EngineDiagnostic> Warnings
        => Diagnostics?.Where(d => d.Severity == DiagnosticSeverity.Warning)
           ?? Enumerable.Empty<EngineDiagnostic>();

    public bool HasErrors => Errors.Any();
    public bool HasWarnings => Warnings.Any();
}
```

**Responsabilidad**

Contener el resultado del renderizado junto con la colección unificada de diagnósticos. Las propiedades `Errors` y `Warnings` son vistas filtradas; el almacenamiento real está en `Diagnostics`.

> **Cambio respecto a v1.0:** se reemplazó `List<string> Warnings` y `List<string> Errors` por una colección única tipada `IList<EngineDiagnostic>`. Las vistas filtradas mantienen accesibilidad pero **el cliente debe migrar** a leer códigos estables (`Diagnostic.Code`) en lugar de comparar substrings de mensajes.

---

## 7. Modelo canónico de diagnósticos

### EngineDiagnostic

```csharp
public class EngineDiagnostic
{
    public string Code { get; set; }                          // Ej. "DSL-VALIDATION-002"
    public DiagnosticSeverity Severity { get; set; }
    public string Message { get; set; }                       // i18n-able
    public string Hint { get; set; }                          // remedio sugerido (opcional)
    public bool Halts { get; set; }                           // ¿detiene el pipeline?
    public IDictionary<string, object> Context { get; set; }  // path JSON, línea, char, codepoint, ...
}

public enum DiagnosticSeverity
{
    Info,
    Warning,
    Error
}
```

### Convención de naming

`<DOMINIO>-<CATEGORÍA>-<NNN>` con `DOMINIO ∈ {DSL, DATA, PROFILE, RENDER, IO}`, 3 dígitos.

### Catálogo canónico (v1.0 del catálogo)

El catálogo completo de códigos vive en [docs/03_ux-ui/experiencia-de-uso-del-motor_v1.1.md](../03_ux-ui/experiencia-de-uso-del-motor_v1.1.md) §8.3 y comprende 17 entradas iniciales en los dominios DSL, DATA, PROFILE, RENDER e IO.

### Principios

* **No ejecución parcial silenciosa:** si un diagnóstico es `Halts: true`, el pipeline se detiene y `RenderResult.Output` es `null`.
* **Trazabilidad:** todos los diagnósticos llevan `Context` con ubicación precisa.
* **Estabilidad de códigos:** los `Code` son estables entre versiones menores. Una baja se anuncia con dos versiones de anticipación.
* **Testeabilidad:** los tests deben asertar contra `Code`, **nunca** contra `Message`.

---

## 8. Contrato del parser DSL

### IDslParser

```csharp
public interface IDslParser
{
    DocumentTemplate Parse(string dsl);
}
```

**Responsabilidad**

* Convertir DSL en modelo estructurado (`DocumentTemplate`)
* Validar sintaxis básica
* Errores se reportan vía `EngineDiagnostic` con códigos `DSL-VALIDATION-*`

---

## 9. Contrato del motor de layout

### ILayoutEngine

```csharp
public interface ILayoutEngine
{
    DocumentNode ApplyLayout(DocumentNode document, DeviceProfile profile);
}
```

**Responsabilidad**

* Aplicar reglas de distribución visual
* Ajustar estructura según perfil del dispositivo
* Implementar el algoritmo de columnado de tablas (ver [representacion-documento-escpos_v1.1.md](../03_ux-ui/representacion-documento-escpos_v1.1.md) §10)
* Preparar el documento para renderizado final

---

## 10. Contrato de resolución de datos

### IDataResolver

```csharp
public interface IDataResolver
{
    object Resolve(object data, string path);
}
```

**Responsabilidad**

* Resolver referencias de datos dentro del documento
* Mapear datos externos a la plantilla
* Errores se reportan vía `EngineDiagnostic` con códigos `DATA-RESOLUTION-*`

---

## 11. Contrato de perfiles de dispositivo

### IDeviceProfileProvider

```csharp
public interface IDeviceProfileProvider
{
    DeviceProfile? GetProfile(string name);
    IEnumerable<DeviceProfile> GetAll();
    void Add(DeviceProfile profile);
}
```

**Responsabilidad**

* Proveer perfiles de dispositivo registrados
* Permitir selección dinámica del target

Ver [CU-21 v2.0](../02_especificacion_funcional/casos-de-uso/CU-21-cargar-perfil-impresora_v2.0.md) para el flujo completo del proveedor inyectable.

---

## 12. Flujo de uso del motor

```text
Plantilla DSL + Datos + Perfil
        ↓
IDocumentEngine.Render()
        ↓
Parse DSL → DocumentTemplate
        ↓ (DSL-VALIDATION-* en caso de fallo)
Resolver Datos
        ↓ (DATA-RESOLUTION-* en caso de fallo)
Evaluar Condiciones / Iteraciones
        ↓
Aplicar Layout (ancho, encoding, columnado)
        ↓ (RENDER-LAYOUT-* y RENDER-ENCODING-* como warnings)
Seleccionar Renderer
        ↓
RenderResult { Output, Diagnostics }
```

---

## 13. Extensibilidad

El motor permite extenderse mediante:

### Nuevos renderizadores

```csharp
public class CustomRenderer : IRenderer
{
    public string Target => "custom";

    public RenderResult Render(DocumentNode document, DeviceProfile profile)
    {
        var diagnostics = new List<EngineDiagnostic>();
        // implementación personalizada; agregar diagnostics si aplica
        return new RenderResult { Target = Target, Output = ..., Diagnostics = diagnostics };
    }
}
```

### Nuevos tipos de nodos

Extendiendo `DocumentNode` para soportar nuevos elementos DSL.

### Registro en DI

```csharp
services.AddSingleton<IRenderer, EscPosRenderer>();
services.AddSingleton<IRenderer, UiRenderer>();
```

---

## 14. Errores y manejo

El motor emite diagnósticos para:

* Errores de parsing (`DSL-VALIDATION-001..005`)
* Datos inconsistentes (`DATA-RESOLUTION-001..003`)
* Perfil no soportado (`PROFILE-SUPPORT-001..002`)
* Encoding y layout (`RENDER-ENCODING-*`, `RENDER-LAYOUT-*`)
* Hardware (`IO-PRINTER-*`)

Todos los diagnósticos se acumulan en `RenderResult.Diagnostics`. Si alguno tiene `Halts: true`, `Output` será `null` y el cliente debe inspeccionar `Errors`.

Las **excepciones C#** quedan reservadas para violaciones de contrato (ej. `null` en parámetros obligatorios), no para errores de dominio.

---

## 15. Criterios de aceptación

El motor cumple sus contratos si:

* Puede renderizar desde DSL o modelo abstracto
* Soporta múltiples renderizadores intercambiables
* Permite inyección de dependencias
* No depende de UI ni infraestructura externa
* Mantiene independencia entre capas
* Permite extensión sin modificar core
* Emite diagnósticos canónicos con códigos estables
* `RenderResult.Diagnostics` es la fuente única de verdad para errores y warnings

---

## 16. Relación con otros documentos

Este documento se alinea con:

* arquitectura-solucion_v1.0.md
* modelo-datos-logico_v1.0.md
* extensibilidad-motor_v1.0.md
* guia-uso-libreria_v1.0.md
* [docs/03_ux-ui/wireframes-documentos_v1.1.md](../03_ux-ui/wireframes-documentos_v1.1.md)
* [docs/03_ux-ui/experiencia-de-uso-del-motor_v1.1.md](../03_ux-ui/experiencia-de-uso-del-motor_v1.1.md) (catálogo de diagnósticos)
* [docs/03_ux-ui/representacion-documento-escpos_v1.1.md](../03_ux-ui/representacion-documento-escpos_v1.1.md) (encoding, algoritmo de columnado)
* [docs/03_ux-ui/representacion-vista-previa-ui_v1.1.md](../03_ux-ui/representacion-vista-previa-ui_v1.1.md)
* [docs/03_ux-ui/glosario_v1.0.md](../03_ux-ui/glosario_v1.0.md)
* CU-xx (casos de uso del motor)
* RN-02 a RN-06
* RC-01 a RC-06

---

## 17. Historial de versiones

| Versión | Fecha      | Autor          | Cambios                         |
| ------- | ---------- | -------------- | ------------------------------- |
| 1.0     | 2026-03-28 | Equipo Técnico | Definición inicial de contratos |
| 1.1     | 2026-04-25 | Equipo Técnico | Modelo canónico de diagnósticos (`EngineDiagnostic`, `DiagnosticSeverity`); `RenderResult.Diagnostics` reemplaza a `Warnings/Errors` (mantenidos como vistas filtradas); `DeviceProfile.Encoding` con `EncodingSpec`; `TableNode.Columns` con `ColumnSpec` para soportar el algoritmo de columnado; vínculo al glosario y al catálogo de códigos. |

---
