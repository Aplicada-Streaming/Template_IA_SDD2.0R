# Definición del DSL — Motor DSL de Documentos

**Archivo:** definicion-dsl_v1.0.md
**Versión:** 1.0
**Fecha:** 2026-03-28
**Autor:** Equipo de Arquitectura / Funcional
**Estado:** Aprobado

---

# 1. Propósito

Definir la especificación del lenguaje DSL utilizado para describir documentos en el Motor DSL de Generación de Documentos.

Este documento establece la estructura, nodos, propiedades y reglas del DSL que permite a los consumidores definir plantillas de documentos de forma declarativa.

---

# 2. Formato

El formato del DSL es **JSON**. Cada plantilla DSL es un archivo JSON que describe la estructura completa de un documento.

Se eligió JSON por:

* amplio soporte en todos los lenguajes y plataformas
* facilidad de validación mediante esquemas (JSON Schema)
* legibilidad y edición manual
* compatibilidad con herramientas de desarrollo existentes

---

# 3. Estructura General

Un documento DSL contiene tres secciones principales:

```json
{
  "metadata": {
    "name": "nombre-plantilla",
    "version": "1.0",
    "description": "Descripción de la plantilla"
  },
  "config": {
    "defaultStyle": { ... },
    "pageWidth": "80mm"
  },
  "body": [
    { "type": "text", "content": "..." },
    { "type": "separator" }
  ]
}
```

* **metadata** — Información descriptiva de la plantilla (nombre, versión, descripción)
* **body** — Array de nodos que componen el contenido del documento
* **config** — Opciones de configuración globales (estilos por defecto, dimensiones)

---

# 4. Nodos Soportados

El DSL soporta los siguientes tipos de nodos:

| Nodo | Descripción |
|------|-------------|
| `text` | Bloque de texto con contenido estático o dinámico |
| `line` | Línea de texto con propiedades de estilo |
| `separator` | Línea separadora visual |
| `spacer` | Espacio vertical |
| `qr` | Código QR generado a partir de datos |
| `barcode` | Código de barras en formatos estándar |
| `image` | Imagen embebida o referenciada |
| `table` | Tabla con filas y columnas |
| `row` | Fila dentro de una tabla |
| `column` | Columna dentro de una fila |
| `conditional` | Nodo que se muestra condicionalmente |
| `loop` | Nodo que itera sobre un array de datos |
| `group` | Agrupación lógica de nodos |

---

# 5. Propiedades Comunes

Todos los nodos comparten un conjunto de propiedades comunes:

| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `type` | string | Tipo del nodo (obligatorio) |
| `style` | object | Propiedades de estilo aplicadas al nodo |
| `content` | string | Contenido textual del nodo |
| `children` | array | Nodos hijos (para nodos contenedores) |
| `condition` | string | Expresión booleana para renderizado condicional |
| `dataBinding` | string | Referencia a un campo de datos dinámico |

Ejemplo:

```json
{
  "type": "text",
  "content": "Total: {{total}}",
  "style": { "bold": true, "alignment": "right" }
}
```

---

# 6. Data Binding

El DSL utiliza la sintaxis `{{campo}}` para vincular datos dinámicos al contenido del documento.

Características:

* **Sintaxis:** `{{nombreCampo}}`
* **Path notation:** soporta acceso a propiedades anidadas con punto: `{{cliente.nombre}}`
* **Contexto:** los datos se resuelven contra el objeto de datos proporcionado al motor
* **Fallback:** si un campo no existe, se resuelve como cadena vacía (configurable)

Ejemplos:

```json
{ "type": "text", "content": "Cliente: {{cliente.nombre}}" }
{ "type": "text", "content": "Fecha: {{fechaEmision}}" }
{ "type": "text", "content": "Total: ${{orden.total}}" }
```

---

# 7. Condicionales

El nodo condicional permite incluir o excluir contenido basándose en expresiones booleanas.

```json
{
  "type": "conditional",
  "condition": "cliente.esVIP",
  "children": [
    { "type": "text", "content": "¡Cliente VIP — Descuento aplicado!" }
  ]
}
```

La propiedad `condition` evalúa una expresión booleana contra los datos proporcionados. Si la expresión es verdadera, se renderizan los nodos hijos; de lo contrario, se omiten.

Expresiones soportadas:

* Acceso a campos booleanos: `"cliente.esVIP"`
* Comparaciones: `"orden.total > 1000"`
* Existencia de datos: `"cliente.email != null"`

---

# 8. Iteraciones

El nodo `loop` permite iterar sobre un array de datos para generar contenido repetitivo.

```json
{
  "type": "loop",
  "source": "orden.items",
  "itemAlias": "item",
  "children": [
    { "type": "text", "content": "{{item.nombre}} x{{item.cantidad}} — ${{item.subtotal}}" }
  ]
}
```

| Propiedad | Descripción |
|-----------|-------------|
| `source` | Path al array de datos sobre el que se itera |
| `itemAlias` | Nombre de la variable que representa cada elemento en la iteración |
| `children` | Nodos que se repiten por cada elemento del array |

---

# 9. Estilos

Propiedades de estilo soportadas por el DSL:

| Propiedad | Valores | Descripción |
|-----------|---------|-------------|
| `bold` | `true` / `false` | Texto en negrita |
| `italic` | `true` / `false` | Texto en cursiva |
| `underline` | `true` / `false` | Texto subrayado |
| `alignment` | `left` / `center` / `right` | Alineación horizontal del contenido |
| `fontSize` | `normal` / `large` | Tamaño del texto |
| `width` | número o porcentaje | Ancho del elemento |

Los estilos se aplican a nivel de nodo y pueden heredarse del estilo por defecto definido en `config.defaultStyle`.

---

# 10. Validación

Toda plantilla DSL debe ser validada antes de su procesamiento. Las reglas de validación incluyen:

* Estructura JSON válida
* Presencia de campos obligatorios (`type` en cada nodo, `body` en el documento)
* Tipos de nodo reconocidos por el motor
* Propiedades válidas para cada tipo de nodo
* Expresiones de data binding con sintaxis correcta
* Referencias de `source` en nodos loop apuntando a paths válidos

Para más detalle sobre las reglas de validación del esquema, consultar [RN-01 Validación de Esquema DSL](reglas-de-negocio/RN-01-validacion-esquema-dsl_v1.0.md).

---

# 11. Ejemplo Completo

Plantilla DSL de un ticket de venta simple:

```json
{
  "metadata": {
    "name": "ticket-venta",
    "version": "1.0",
    "description": "Ticket de venta estándar"
  },
  "config": {
    "defaultStyle": { "fontSize": "normal", "alignment": "left" }
  },
  "body": [
    {
      "type": "text",
      "content": "{{empresa.nombre}}",
      "style": { "bold": true, "alignment": "center", "fontSize": "large" }
    },
    {
      "type": "text",
      "content": "{{empresa.direccion}}",
      "style": { "alignment": "center" }
    },
    { "type": "separator" },
    {
      "type": "text",
      "content": "Fecha: {{fechaEmision}}"
    },
    {
      "type": "text",
      "content": "Ticket #{{numeroTicket}}"
    },
    { "type": "separator" },
    {
      "type": "loop",
      "source": "orden.items",
      "itemAlias": "item",
      "children": [
        {
          "type": "text",
          "content": "{{item.nombre}}  x{{item.cantidad}}  ${{item.subtotal}}"
        }
      ]
    },
    { "type": "separator" },
    {
      "type": "text",
      "content": "TOTAL: ${{orden.total}}",
      "style": { "bold": true, "alignment": "right", "fontSize": "large" }
    },
    { "type": "spacer" },
    {
      "type": "conditional",
      "condition": "cliente.esVIP",
      "children": [
        {
          "type": "text",
          "content": "¡Gracias por ser cliente VIP!",
          "style": { "alignment": "center" }
        }
      ]
    },
    {
      "type": "qr",
      "content": "{{urlComprobante}}",
      "style": { "alignment": "center" }
    }
  ]
}
```

---

# 12. Trazabilidad

Para detalles de implementación y uso práctico del formato DSL, consultar:

* [formato-dsl-templates.md](../10_developer_guide/formato-dsl-templates.md) en el Developer Guide
* [RN-01 Validación de Esquema DSL](reglas-de-negocio/RN-01-validacion-esquema-dsl_v1.0.md)
* [RN-03 Estructura Jerárquica](reglas-de-negocio/RN-03-estructura-jerarquica_v1.0.md)

---

# 13. Control de Cambios

| Versión | Fecha      | Descripción     |
| ------- | ---------- | --------------- |
| 1.0     | 2026-03-28 | Versión inicial |

---

**Fin del documento**
