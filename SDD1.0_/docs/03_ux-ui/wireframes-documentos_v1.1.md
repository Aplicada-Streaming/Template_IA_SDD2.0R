# Wireframes de Documentos

**Proyecto:** Motor DSL de Documentos para Impresión
**Documento:** wireframes-documentos_v1.1.md
**Versión:** 1.1
**Estado:** Aprobado
**Fecha:** 2026-04-25
**Autor:** Equipo de Arquitectura / Funcional
**Glosario:** [glosario_v1.0.md](glosario_v1.0.md)

---

# 1. Introducción

Este documento describe los **wireframes conceptuales** de los documentos soportados por el motor DSL. Su objetivo es representar visualmente, a nivel lógico, cómo se estructuran los elementos de un documento antes de su renderizado final en distintos formatos (ESC/POS, vista previa UI, texto plano, PDF futuro).

Los wireframes **no representan estilos gráficos finales**, sino la **organización jerárquica** y la distribución de componentes. Para los términos usados en este documento (*plantilla DSL*, *modelo interno*, *representación abstracta*, *renderizador*), ver [glosario_v1.0.md](glosario_v1.0.md).

---

# 2. Alcance

Este documento cubre:

* Wireframes lógicos de arquetipos de documento
* Mapeo entre wireframe y nodos DSL reales
* Principios de diseño de plantillas
* Adaptación de wireframes a perfiles de dispositivo

No incluye:

* Estilos visuales finales (tipografía, color, espaciado en píxeles)
* Implementación del motor de layout
* Comandos ESC/POS específicos (ver [representacion-documento-escpos_v1.1.md](representacion-documento-escpos_v1.1.md))

---

# 3. Objetivo

Definir una representación simplificada de los documentos para:

* Visualizar la estructura jerárquica
* Facilitar el diseño de plantillas DSL
* Servir como guía para implementadores del motor
* Asegurar consistencia entre distintos renderizadores

---

# 4. Principios de diseño

* **Jerarquía clara:** cada documento se organiza en bloques anidados.
* **Independencia del renderizador:** el wireframe no depende del formato final.
* **Abstracción visual:** representa posiciones relativas, no píxeles exactos.
* **Modularidad:** cada sección del documento es reutilizable.
* **Adaptabilidad:** los bloques deben poder reordenarse según el perfil de dispositivo.
* **Trazabilidad a la DSL:** cada bloque del wireframe debe corresponder a uno o más nodos DSL definidos en [definicion-dsl_v1.0.md](../02_especificacion_funcional/definicion-dsl_v1.0.md).

---

# 5. Estructura general de un documento

Un documento típico se compone de tres regiones lógicas:

* **Header** (encabezado)
* **Body** (contenido principal)
* **Footer** (pie de página)

Wireframe lógico:

```text
+--------------------------------------+
|              HEADER                  |
+--------------------------------------+
|                                      |
|              BODY                    |
|                                      |
+--------------------------------------+
|              FOOTER                  |
+--------------------------------------+
```

**Nota:** estas tres regiones son convención de diseño. La DSL **no impone** una sección `header`/`body`/`footer` separada — todo el contenido vive bajo `body[]` con orden secuencial. La separación visual se logra agrupando con nodos `group` o `separator`.

---

# 6. Wireframe de elementos comunes

Cada subsección incluye el wireframe ASCII y el **mapeo a nodos DSL reales**.

## 6.1 Texto simple

```text
[ Texto ]
```

Mapeo DSL:

```json
{ "type": "text", "content": "Total: {{orden.total}}" }
```

---

## 6.2 Bloque con alineación

```text
| Izquierda        Centro        Derecha |
```

Mapeo DSL:

```json
{ "type": "text", "content": "Izquierda", "style": { "alignment": "left" } }
{ "type": "text", "content": "Centro",    "style": { "alignment": "center" } }
{ "type": "text", "content": "Derecha",   "style": { "alignment": "right" } }
```

**Nota:** ESC/POS no permite tres alineaciones distintas en la misma línea de texto. Si se requiere ese efecto, debe modelarse como una `table` de tres columnas (ver §6.4).

---

## 6.3 Lista de ítems (iteración)

Wireframe lógico:

```text
[FOR cada item en orden.items]
  Item.Nombre   Cant   Precio
[END FOR]
```

> **Aviso:** la sintaxis `[FOR ... END FOR]` es **pseudosintaxis ilustrativa** y **no es la DSL real del motor**. La DSL canónica es JSON con un nodo `loop`.

Mapeo DSL real:

```json
{
  "type": "loop",
  "source": "orden.items",
  "itemAlias": "item",
  "children": [
    { "type": "text", "content": "{{item.nombre}}  x{{item.cantidad}}  ${{item.subtotal}}" }
  ]
}
```

---

## 6.4 Tabla lógica

Wireframe:

```text
+----------------------------------+
| Col1    | Col2    | Col3         |
+----------------------------------+
| Val1    | Val2    | Val3         |
| Val4    | Val5    | Val6         |
+----------------------------------+
```

Mapeo DSL:

```json
{
  "type": "table",
  "children": [
    {
      "type": "row",
      "children": [
        { "type": "column", "style": { "width": "50%" }, "content": "Col1" },
        { "type": "column", "style": { "width": "30%" }, "content": "Col2" },
        { "type": "column", "style": { "width": "20%" }, "content": "Col3" }
      ]
    }
  ]
}
```

**Nota:** el comportamiento de columnas (truncado, wrap, reducción proporcional) está definido en [representacion-documento-escpos_v1.1.md](representacion-documento-escpos_v1.1.md) §10.

---

## 6.5 Sección condicional

Wireframe lógico:

```text
[IF cliente.esVIP]
  Mostrar bloque VIP
[ELSE]
  Mostrar bloque estándar
[END IF]
```

> **Aviso:** la sintaxis `[IF ... END IF]` es **pseudosintaxis ilustrativa**. La DSL real usa el nodo `conditional`.

Mapeo DSL real:

```json
{
  "type": "conditional",
  "condition": "cliente.esVIP",
  "children": [
    { "type": "text", "content": "Bloque VIP" }
  ]
}
```

**Nota:** la DSL v1.0 **no soporta `else` nativo**. Para una rama alternativa se usan dos nodos `conditional` con condiciones complementarias.

---

## 6.6 Códigos QR y barras

Wireframe:

```text
+------------+
|   ##  ##   |
|  ###  ##   |
|   QR code  |
+------------+
```

Mapeo DSL:

```json
{ "type": "qr", "content": "{{urlComprobante}}", "style": { "alignment": "center" } }
{ "type": "barcode", "content": "{{codigoTicket}}", "style": { "alignment": "center" } }
```

---

## 6.7 Imagen

Wireframe:

```text
+------------+
|   [LOGO]   |
+------------+
```

Mapeo DSL:

```json
{ "type": "image", "content": "logo-empresa.png", "style": { "alignment": "center" } }
```

---

# 7. Wireframes de arquetipos de documento

## 7.1 Ticket de venta (arquetipo principal)

```text
+--------------------------------------------------+
|              [LOGO EMPRESA]                      |
|              EMPRESA S.A.                        |
|         Av. Siempreviva 742 - CABA               |
+--------------------------------------------------+
|  Ticket #00123              Fecha: 2026-04-25   |
|  Cliente: Juan Pérez                             |
+--------------------------------------------------+
|  PRODUCTOS                                       |
+--------------------------------------------------+
|  Item              Cant     Precio     Total     |
|  Café              2        100.00     200.00    |
|  Medialuna         3         50.00     150.00    |
+--------------------------------------------------+
|                       Subtotal:        350.00    |
|                       IVA 21%:          73.50    |
|                       TOTAL:           423.50    |
+--------------------------------------------------+
|              [QR comprobante AFIP]               |
|        ¡Gracias por su compra!                   |
+--------------------------------------------------+
```

Estructura jerárquica:

```text
Documento
├── Header
│   ├── Logo (image)
│   ├── Razón social (text, center, bold)
│   └── Dirección (text, center)
├── Body
│   ├── Datos transaccionales (table 2 cols)
│   ├── Separador
│   ├── Título "PRODUCTOS" (text)
│   ├── Tabla de items
│   │   └── Loop sobre orden.items
│   └── Totales (table 2 cols, alineación derecha)
└── Footer
    ├── QR (qr)
    └── Mensaje de cierre (text, center)
```

---

## 7.2 Recibo simplificado

```text
+----------------------------+
|    RECIBO DE PAGO          |
+----------------------------+
|  N°: 00045                 |
|  Fecha: 2026-04-25         |
|  Concepto: Servicio mes    |
+----------------------------+
|  Monto:           $1500.00 |
+----------------------------+
|  Recibí conforme           |
|  ____________________      |
|       Firma                |
+----------------------------+
```

Estructura jerárquica:

```text
Documento
├── Header
│   └── Título (text, center, bold, large)
├── Body
│   ├── Datos del recibo (3 text)
│   ├── Separador
│   └── Monto (text, right, bold)
└── Footer
    ├── Texto "Recibí conforme"
    ├── Espaciador (spacer)
    └── Línea de firma (separator + text)
```

---

## 7.3 Comanda de cocina

```text
+----------------------------+
|       COMANDA              |
|       Mesa 7  -  19:42     |
+----------------------------+
|  >> 2x Café cortado        |
|  >> 3x Medialuna           |
|  >> 1x Tostado mixto       |
|       (sin tomate)         |
+----------------------------+
|  Mozo: Carla               |
+----------------------------+
```

Estructura jerárquica:

```text
Documento
├── Header
│   ├── Título "COMANDA"
│   └── Mesa + hora
├── Body
│   └── Loop sobre items
│       ├── Cantidad + nombre
│       └── Conditional: nota especial
└── Footer
    └── Mozo asignado
```

**Característica:** documento de **alta urgencia**, sin elementos decorativos (no hay logo, ni QR, ni totales). Fuente típica más grande (`fontSize: "large"`) para lectura rápida en cocina.

---

# 8. Representación jerárquica equivalente (genérica)

```text
Documento
├── Header
│   ├── Logo
│   └── Datos del cliente
├── Body
│   ├── Sección: Productos
│   │   ├── Tabla
│   │   └── Iteración de items
│   └── Sección: Totales
└── Footer
    └── Información adicional
```

---

# 9. Relación con el motor DSL

Los wireframes se derivan de:

* Plantillas DSL (estructura declarativa)
* Modelo interno del documento
* Datos de entrada

Y se utilizan para:

* Validación visual conceptual
* Depuración de layout
* Diseño de plantillas
* Interpretación del renderizador

---

# 10. Consideraciones para perfiles de dispositivo

Dependiendo del perfil de dispositivo:

* **ESC/POS (térmica):** layout lineal, sin posicionamiento libre, ancho 32/42/48 caracteres
* **UI (preview):** layout flexible con contenedores, simulación de ancho del perfil
* **PDF (futuro):** layout paginado con control preciso

El wireframe se adapta a restricciones como:

* Ancho de papel
* Tamaño de pantalla
* Márgenes
* Capacidades del dispositivo (QR sí/no, imagen sí/no, encoding)

---

# 11. Extensibilidad

Los wireframes pueden extenderse para incluir:

* Elementos gráficos (líneas, separadores)
* Imágenes
* Códigos QR o de barras
* Componentes personalizados (vía CU-29)

---

# 12. Casos de Uso Asociados

| Caso de Uso | Descripción                              |
| ----------- | ---------------------------------------- |
| CU-01       | Interpretar plantilla DSL                |
| CU-03       | Construir modelo interno                 |
| CU-04       | Ejecutar motor de layout                 |
| CU-05       | Generar representación abstracta         |
| CU-13       | Cargar plantilla DSL                     |
| CU-14       | Validar plantilla DSL                    |
| CU-16       | Ejecutar iteraciones (listas)            |
| CU-17       | Evaluar condiciones                      |

---

# 13. Conclusión

El wireframe de documentos permite abstraer la estructura visual de un documento antes de su renderizado, facilitando la coherencia entre distintos formatos de salida y simplificando el diseño de plantillas DSL.

Sirve como puente conceptual entre:

* Datos
* Plantillas
* Motor de renderizado
* Dispositivo final

---

# 14. Historial de Versiones

| Versión | Fecha      | Autor               | Cambios                                                                                                                            |
| ------- | ---------- | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| 1.0     | 2026-03-28 | Equipo Arquitectura | Versión inicial de wireframes                                                                                                      |
| 1.1     | 2026-04-25 | Equipo Arquitectura | Cabecera estandarizada; cajas ASCII corregidas; mapeo a DSL real para cada wireframe; aclaración de pseudosintaxis vs DSL canónica; agregados arquetipos *recibo* y *comanda*; sección de Casos de Uso Asociados; vínculo al glosario. |

---

**Fin del documento**
