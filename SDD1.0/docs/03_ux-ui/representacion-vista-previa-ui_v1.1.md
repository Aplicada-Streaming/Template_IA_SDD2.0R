# Representación de Documento en Vista Previa UI

**Proyecto:** Motor DSL de Documentos para Impresión
**Documento:** representacion-vista-previa-ui_v1.1.md
**Versión:** 1.1
**Estado:** Aprobado
**Fecha:** 2026-04-25
**Autor:** Equipo de Arquitectura / Renderización
**Glosario:** [glosario_v1.0.md](glosario_v1.0.md)

---

# 1. Introducción

Este documento describe la **representación de documentos en vista previa UI** dentro del motor DSL. Define cómo una *representación abstracta* es visualizada en una interfaz gráfica, permitiendo a los usuarios validar la salida **antes** de su impresión o exportación.

La vista previa UI constituye una representación **no física** del documento, orientada a simulación visual en pantalla con tolerancia formal definida (ver §7) respecto al output final del dispositivo destino. Para los términos *renderizador*, *perfil de dispositivo*, *representación abstracta*, ver [glosario_v1.0.md](glosario_v1.0.md).

---

# 2. Alcance

Este documento cubre:

* Representación visual del documento en UI (.NET MAUI)
* Simulación de layout con paridad de caracteres
* Renderizado de elementos estructurados
* Adaptación visual según perfil de dispositivo
* Tolerancia formal preview ↔ ESC/POS (§7)
* Accesibilidad (WCAG 2.2)
* Interacción básica de visualización

No incluye:

* Renderizado a impresoras físicas
* Generación de comandos ESC/POS
* Persistencia de datos
* Lógica de negocio del documento

---

# 3. Concepto de vista previa

La vista previa UI es una representación gráfica del documento que permite:

* Visualizar el layout final con tolerancia acotada
* Validar distribución de elementos
* Detectar errores de formato y truncado
* Simular condiciones reales de impresión

A diferencia del renderizador ESC/POS, esta representación está orientada a **interfaces gráficas (pantallas)**, no a dispositivos físicos.

---

# 4. Estructura general de la vista

La vista previa del documento se organiza en:

1. Contenedor principal (viewport)
2. Área del documento (con ancho equivalente al perfil)
3. Secciones estructuradas (header / body / footer)
4. Elementos individuales
5. Indicadores visuales opcionales (margen de papel, línea de corte)

---

# 5. Representación de elementos

## 5.1 Texto

* **Tipografía obligatoriamente monoespaciada** (Cascadia Mono / Consolas / Menlo / fallback `monospace`).
* Tamaño fijo del char (14 px por defecto, configurable).
* Estilos visuales: negrita, subrayado, doble alto/ancho.
* Alineación: izquierda / centro / derecha.

**Justificación de la fuente monoespaciada:** garantiza que el conteo de caracteres por línea coincida con el cálculo del renderizador ESC/POS. Una fuente proporcional **rompería la simulación** (un texto de 32 chars podría caber visualmente en preview pero no en la impresora física).

---

## 5.2 Alineación

La alineación se refleja con respecto al ancho útil del perfil (no al ancho del viewport):

* Izquierda → alineado al margen izquierdo del documento
* Centro → centrado dentro del ancho del perfil
* Derecha → alineado al margen derecho del documento

---

## 5.3 Bloques de contenido

Los bloques representan secciones (encabezado, detalle, totales, pie). Cada bloque se renderiza como una unidad visual separada usando el mismo motor de layout que ESC/POS.

---

## 5.4 Separadores

Se visualizan como líneas horizontales de caracteres (`-`, `=`, `*`) con la misma longitud que tendrían en ESC/POS.

---

## 5.5 Tablas simuladas

Las tablas usan exactamente el **mismo algoritmo de columnado** que el renderer ESC/POS (ver [representacion-documento-escpos_v1.1.md](representacion-documento-escpos_v1.1.md) §10). Esto garantiza que los anchos de columna y wrapping coincidan byte-a-byte con el output físico.

---

## 5.6 Imágenes

Se representan como elementos visuales:

* Logos cargados como `Image` de MAUI
* Escalado proporcional al ancho del documento
* **Bounding box vertical equivalente** al que ocuparían en ESC/POS (alto en líneas estimado)
* Fallback a placeholder textual `[IMG: nombre.png]` si el archivo no carga

---

## 5.7 Códigos QR

* Generados gráficamente con la misma cadena que iría a ESC/POS
* Tamaño en píxeles tal que el bounding box vertical ocupe las mismas líneas que en ESC/POS
* Si el perfil declara `supports.qr = false`, mostrar placeholder `[QR: contenido]` para reflejar que en la impresora **no aparecerá**

---

## 5.8 Códigos de barras

Análogo a QR. Renderizado visual con la misma altura aproximada en líneas.

---

## 5.9 Espaciado y layout

El layout en UI **simula** el comportamiento del dispositivo final:

* Márgenes proporcionales al perfil
* Padding interno equivalente
* Espaciado entre elementos según `spacer`
* Saltos de línea explícitos

---

# 6. Flujo de renderizado a vista previa

```text
Plantilla DSL + Datos + Perfil
        ↓
Modelo interno
        ↓
Motor de Layout                 ← compartido con renderizador ESC/POS
        ↓
Representación abstracta        ← misma estructura que ESC/POS
        ↓
Renderizador UI
        ↓
Componentes visuales (MAUI)
        ↓
Pantalla
```

**Punto clave:** los pasos hasta "representación abstracta" son **idénticos** entre los renderizadores UI y ESC/POS. Solo difieren en el último paso. Esto es lo que permite la tolerancia ±0 en caracteres por línea.

---

# 7. Tolerancia formal preview ↔ ESC/POS

Esta sección reemplaza al disclaimer "no garantiza fidelidad absoluta" por **criterios verificables**.

## 7.1 Decisión vigente (default v1.1)

| Aspecto | Tolerancia | Verificación |
|---|---|---|
| Caracteres por línea (texto) | **±0** | Diff exacto contra el renderer de texto plano (debug) |
| Anchos de columna en tablas | **±0** | Mismo algoritmo de columnado que ESC/POS (§10 del doc ESC/POS) |
| Wrapping de líneas largas | **±0** | Misma lógica greedy word-wrap |
| Truncado / ellipsis | **±0** | Misma política `overflow` declarada en la columna |
| Altura vertical (líneas) | **±0** para texto puro | Conteo de saltos de línea idéntico |
| Altura vertical con QR/imagen | **±1 línea** | Estimación basada en módulo QR / aspect ratio de imagen |
| Bytes ESC/POS | N/A | El preview no genera bytes ESC/POS — la fidelidad se mide a nivel de cadena monoespaciada |
| Tipografía visual | Aproximada | Fuente del sistema, no la fuente del firmware |
| Color del papel | N/A | Fondo blanco fijo |

## 7.2 Test de fidelidad

```text
text_debug = engine.Render(plantilla, datos, perfil, formato=TEXTO_PLANO)
preview     = engine.Render(plantilla, datos, perfil, formato=PREVIEW_UI)

assert ExtractTextSkeleton(preview) == text_debug
```

Es decir: el "esqueleto textual" del preview (caracteres, sin estilos) debe ser **byte-exacto** al output del renderer de texto plano para el mismo input. Esto se verifica en CI mediante snapshot tests.

## 7.3 Casos donde la tolerancia se relaja

* **Imágenes raster:** la calidad visual del preview es de pantalla; la impresora térmica imprime monocromo a 180–203 dpi. La forma será reconocible, no idéntica.
* **QR:** el código contiene la misma cadena, pero el patrón visual difiere por algoritmos de generación distintos (QRCoder en preview vs algoritmo del firmware). Lo crítico es que **el QR es escaneable en ambos**.
* **Buzzer / corte de papel:** no se simulan en preview. Pueden representarse con un ícono opcional al final del documento.

## 7.4 Alternativa registrada (no adoptada)

**Opción mínima descartada:** preview con paridad estricta de caracteres pero sin renderizado gráfico de QR/imágenes (placeholders textuales `[QR]`, `[IMG]`). Se rechazó porque la persona "Carla, diseñadora de plantillas" pierde la confirmación visual de que el QR generado es legible.

---

# 8. Adaptación visual según perfil

La vista previa se adapta según:

* Ancho del perfil (32 / 42 / 48 caracteres → ancho del viewport calculado)
* Resolución de pantalla
* Tipo de dispositivo (desktop / mobile / tablet)
* Capacidades del perfil (`supports.qr`, `supports.image`, `supports.cutter`)

---

# 9. Simulación visual del dispositivo

La UI **debe** incluir indicadores que comuniquen las restricciones del perfil:

* Banda visual del ancho de papel (ej. simular 58mm o 80mm)
* Línea punteada indicando la posición del corte
* Indicador del perfil activo (texto: "Perfil: thermal-58mm-cp858")
* Iconos opcionales para corte/buzzer si aplican

---

# 10. Interacción

## 10.1 Funcionalidades mínimas

* **Scroll vertical** — Documentos largos
* **Zoom** — 50% / 75% / 100% / 150% / 200% (mantiene fuente monoespaciada y ratios)
* **Ajustar a ancho** — Encaja el ancho del documento al viewport disponible
* **Refrescar** — Re-renderiza con los datos actuales
* **Toggle indicadores** — Mostrar/ocultar márgenes de papel y línea de corte

## 10.2 Funcionalidades opcionales

* Inspección de elementos (debug visual): hover sobre un elemento muestra su nodo DSL de origen
* Comparar lado a lado con texto plano (split view)
* Exportar el preview como imagen PNG

## 10.3 Atajos de teclado

| Acción | Atajo |
|--------|--------|
| Zoom in | `Ctrl + +` |
| Zoom out | `Ctrl + -` |
| Zoom reset (100%) | `Ctrl + 0` |
| Ajustar a ancho | `Ctrl + W` |
| Refrescar | `F5` |
| Toggle indicadores | `Ctrl + I` |

---

# 11. Accesibilidad (WCAG 2.2)

La vista previa forma parte de una app .NET MAUI accesible. Requisitos mínimos:

## 11.1 Visión

* **Contraste:** texto del documento contra fondo cumple WCAG 2.2 nivel **AA** (ratio ≥ 4.5:1 para texto normal, ≥ 3:1 para texto grande). Default: texto negro `#000000` sobre fondo blanco `#FFFFFF` (ratio 21:1).
* **Tamaño de texto:** soporta zoom hasta 200% sin pérdida de funcionalidad.
* **No depender de color:** ningún estado del preview se comunica solo por color (los warnings y errores siempre incluyen ícono y/o texto).

## 11.2 Navegación por teclado

* Toda funcionalidad accesible con teclado (sin requerir mouse o touch).
* Foco visible (outline) en todos los controles interactivos.
* Orden de tabulación lógico (top-to-bottom, left-to-right).
* `Esc` cierra cualquier diálogo modal.

## 11.3 Lectores de pantalla

* El área de preview anuncia su rol como `document` o `region` (con etiqueta "Vista previa de ticket").
* Los controles de zoom e indicadores tienen `AutomationProperties.Name` y `HelpText`.
* Los warnings y errores del motor (ver §8 del doc de experiencia) se anuncian al lector de pantalla cuando aparecen.

## 11.4 Movimiento

* Sin animaciones esenciales.
* Las transiciones de zoom respetan la preferencia del SO `prefers-reduced-motion` cuando esté disponible.

## 11.5 Limitaciones aceptadas

* El **contenido textual del documento** (ej. nombre del producto, total) es responsabilidad de la plantilla y los datos; el preview no puede garantizar el contraste de imágenes embebidas.
* El **QR generado** no es accesible por sí mismo a un lector de pantalla; debe acompañarse de un texto alternativo legible que represente la URL/cadena del código.

---

# 12. Limitaciones de la vista previa

* No reproduce exactamente la calidad visual de impresión térmica
* La tipografía visual difiere de la del firmware
* No simula el grosor del papel ni la calidad del corte
* Imágenes raster se ven más nítidas que en impresión real

Estas limitaciones son **esperadas y documentadas**, no defectos. La tolerancia formal está acotada en §7.

---

# 13. Estrategias de representación

El motor aplica:

* Mapeo de bloques abstractos a componentes UI MAUI
* Conversión de layout abstracto a layout monoespaciado
* Simulación de restricciones del perfil
* Adaptación responsiva del viewport

---

# 14. Relación con otros componentes

Esta representación depende de:

* CU-04: Ejecutar motor de layout
* CU-05: Generar representación abstracta
* CU-08: Renderizar vista previa UI
* CU-11: Seleccionar perfil de dispositivo
* CU-12: Adaptar documento al perfil
* RC-06: Perfil de dispositivo condiciona la salida

---

# 15. Ejemplo simplificado

Plantilla DSL:

```json
{
  "body": [
    { "type": "text", "content": "Ticket", "style": { "alignment": "center", "bold": true } },
    { "type": "separator" },
    { "type": "text", "content": "Total: 100", "style": { "alignment": "right" } },
    { "type": "separator" }
  ]
}
```

Vista previa UI (perfil 32 chars):

```text
┌────────────────────────────────┐
│            Ticket              │  ← bold, center
├────────────────────────────────┤
│ ------------------------------ │  ← separator (30 chars)
│                     Total: 100 │  ← right-aligned (32 chars)
│ ------------------------------ │
└────────────────────────────────┘
   ──── corte ────
```

Renderizado como una estructura visual con ancho fijo de 32 caracteres en fuente monoespaciada.

---

# 16. Casos de Uso Asociados

| Caso de Uso | Descripción                              |
| ----------- | ---------------------------------------- |
| CU-04       | Ejecutar motor de layout                 |
| CU-05       | Generar representación abstracta         |
| CU-08       | Renderizar vista previa UI               |
| CU-11       | Seleccionar perfil de dispositivo        |
| CU-12       | Adaptar documento al perfil              |
| CU-23       | Aplicar restricciones del dispositivo    |

---

# 17. Historial de Versiones

| Versión | Fecha      | Autor               | Cambios                                                                                                                                                                                       |
| ------- | ---------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.0     | 2026-03-28 | Equipo Arquitectura | Versión inicial de vista previa UI                                                                                                                                                            |
| 1.1     | 2026-04-25 | Equipo Arquitectura | Tolerancia formal ±0 caracteres y ±1 línea (§7); fuente monoespaciada obligatoria; sección de Accesibilidad WCAG 2.2 AA (§11); atajos de teclado (§10.3); test de fidelidad; vínculo al glosario. |

---

**Fin del documento**
