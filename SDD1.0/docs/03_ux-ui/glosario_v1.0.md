# Glosario UX/UI — Motor DSL de Documentos

**Proyecto:** Motor DSL de Documentos para Impresión
**Documento:** glosario_v1.0.md
**Versión:** 1.0
**Estado:** Aprobado
**Fecha:** 2026-04-25
**Autor:** Equipo de Arquitectura / Funcional

---

# 1. Propósito

Este glosario establece la **terminología canónica** para los documentos del directorio `docs/03_ux-ui/`. Su objetivo es eliminar ambigüedades detectadas en el uso de términos del pipeline de renderizado, perfiles de dispositivo y renderizadores.

Los términos definidos aquí son de uso **obligatorio y consistente** en todos los documentos de UX/UI del proyecto. Si otro documento (especificación funcional, arquitectura, casos de uso) usa un término distinto, debe ser corregido o se considera deuda documental.

---

# 2. Términos del pipeline del motor

El motor procesa el documento en etapas con artefactos diferenciados. La distinción es importante porque **cada artefacto se transforma en el siguiente**:

## 2.1 Documento abstracto

**Definición:** Concepto lógico de "el documento que se quiere imprimir", **independiente de cualquier representación**. Es un término de dominio, no una estructura técnica.

**Ejemplo:** "Un ticket de venta con encabezado, productos y total" es un documento abstracto.

**No confundir con:** modelo interno (estructura técnica) ni representación abstracta (artefacto post-layout).

---

## 2.2 Plantilla DSL

**Definición:** Archivo JSON que describe la estructura del documento de forma declarativa, según la especificación de [definicion-dsl_v1.0.md](../02_especificacion_funcional/definicion-dsl_v1.0.md).

**Ejemplo:** `{ "metadata": {...}, "config": {...}, "body": [...] }`.

---

## 2.3 Modelo interno

**Definición:** Estructura de datos en memoria que el motor construye al **parsear y validar** una plantilla DSL junto con sus datos. Es la representación normalizada y enriquecida lista para procesar.

**Sinónimos aceptables en el código:** *DocumentModel*, *AST del documento* (en contextos de implementación).

**Ejemplo:** árbol de nodos con referencias resueltas, condicionales evaluadas e iteraciones expandidas.

**No confundir con:** plantilla DSL (input) ni representación abstracta (output del motor de layout).

---

## 2.4 Representación abstracta

**Definición:** Estructura intermedia producida por el **motor de layout** a partir del modelo interno. Contiene posiciones, anchos, líneas y bloques ya resueltos, pero **sigue siendo independiente del renderizador final** (ESC/POS, UI, PDF, texto plano).

**Ejemplo:** una secuencia de "líneas" con alineación, ancho en caracteres y contenido, lista para que cualquier renderizador la convierta a su formato.

**No confundir con:** modelo interno (pre-layout) ni salida final (post-renderizador).

---

## 2.5 Pipeline canónico

```
Plantilla DSL + Datos + Perfil
        ↓ (parse + validación)
Modelo interno
        ↓ (motor de layout)
Representación abstracta
        ↓ (renderizador específico)
Salida final (ESC/POS / UI / texto plano / PDF)
```

---

# 3. Términos de perfiles

## 3.1 Perfil de dispositivo

**Definición:** **Término canónico**. Estructura declarativa que describe las capacidades y restricciones del dispositivo de salida (ancho en caracteres o mm, encoding soportado, soporte de QR/imágenes, capacidades de corte, etc.).

**Uso:** preferido en toda documentación técnica y de UX/UI.

**Sinónimo deprecado:** *perfil de impresora* — sigue apareciendo en algunos casos de uso (CU-11, CU-21, CU-26) por compatibilidad histórica, pero **no se debe usar en documentación nueva**. Las referencias futuras deben migrar a "perfil de dispositivo".

**Ejemplo:** Perfil `thermal-58mm-cp858` con ancho 32 caracteres, encoding CP858, sin soporte de imagen color.

---

# 4. Términos de renderizado

## 4.1 Renderizador

**Definición:** **Término canónico** (en español). Componente del motor que transforma una representación abstracta en una salida concreta para un formato específico.

**Sinónimo deprecado en documentación en español:** *renderer* — usar solo en código fuente, nombres de clases (`BitmapEscPosRenderer`, `PdfRenderer`) y referencias a archivos `.cs`. En prosa explicativa, usar siempre "renderizador".

**Tipos previstos:**

* Renderizador ESC/POS
* Renderizador de vista previa UI
* Renderizador de texto plano (debug)
* Renderizador PDF (futuro)

---

# 5. Términos de estructura visual

## 5.1 Layout

**Definición:** Disposición espacial de los elementos del documento (orden vertical, alineación horizontal, ancho de columnas, márgenes). Producido por el **motor de layout** a partir del modelo interno.

**Uso:** término técnico aceptado, no requiere traducción.

---

## 5.2 Wireframe

**Definición:** Representación esquemática y de baja fidelidad de la **estructura jerárquica** de un documento, usada con fines de diseño y comunicación, **sin estilos visuales finales**.

**Uso:** acotado al documento [wireframes-documentos](wireframes-documentos_v1.1.md). No debe usarse como sinónimo de "layout" ni de "representación abstracta".

---

## 5.3 Estructura

**Definición:** Término genérico. **Su uso está desaconsejado** en documentación nueva por ser polisémico. Cuando se necesite referirse a:

* La organización jerárquica de nodos → usar **"jerarquía de nodos"** o **"árbol del documento"**.
* La disposición visual → usar **"layout"**.
* La representación intermedia → usar **"representación abstracta"** o **"modelo interno"**.

---

# 6. Términos de experiencia

## 6.1 Developer Experience (DX)

**Definición:** Calidad de la experiencia que tiene un desarrollador al integrar y usar el motor como librería. Cubre: claridad de la API, modelo de errores, diagnóstico, documentación, predictibilidad y velocidad para llegar a un primer documento renderizado correctamente.

**Métricas de referencia:** tiempo a primera impresión exitosa (TTFP), tasa de errores de plantilla, número de consultas a soporte por integración.

---

## 6.2 Vista previa (preview)

**Definición:** Renderizado del documento en una superficie gráfica (pantalla) que **simula** el output del dispositivo final con tolerancia definida (ver [representacion-vista-previa-ui_v1.1.md](representacion-vista-previa-ui_v1.1.md) §7).

**No es:** WYSIWYG estricto. La fidelidad está acotada por contrato.

---

# 7. Términos transversales

| Término | Definición canónica corta | Documento de referencia |
|---------|---------------------------|--------------------------|
| DSL | Lenguaje declarativo (JSON) para describir documentos | [definicion-dsl_v1.0.md](../02_especificacion_funcional/definicion-dsl_v1.0.md) |
| ESC/POS | Estándar de comandos de Epson para impresoras térmicas | [representacion-documento-escpos_v1.1.md](representacion-documento-escpos_v1.1.md) |
| Code page | Tabla de codificación de caracteres (CP437, CP858, ...) | [representacion-documento-escpos_v1.1.md](representacion-documento-escpos_v1.1.md) §8 |
| Diagnóstico | Cualquier mensaje (error, warning, info) emitido por el motor | [experiencia-de-uso-del-motor_v1.1.md](experiencia-de-uso-del-motor_v1.1.md) §8 |

---

# 8. Convenciones de uso

1. En **documentación en español**, usar los términos canónicos en español ("renderizador", no "renderer").
2. En **código** y **nombres de archivos `.cs`**, se acepta inglés (`Renderer`, `Profile`, `LayoutEngine`).
3. Cuando se introduce un término del glosario por primera vez en un documento, **enlazarlo** a este glosario.
4. Las referencias cruzadas usan rutas relativas, no absolutas.

---

# 9. Historial de Versiones

| Versión | Fecha      | Autor               | Cambios                                  |
| ------- | ---------- | ------------------- | ---------------------------------------- |
| 1.0     | 2026-04-25 | Equipo Arquitectura | Glosario inicial para `docs/03_ux-ui/`   |

---

**Fin del documento**
