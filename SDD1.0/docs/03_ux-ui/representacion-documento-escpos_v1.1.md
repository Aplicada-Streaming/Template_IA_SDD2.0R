# Representación de Documento en ESC/POS

**Proyecto:** Motor DSL de Documentos para Impresión
**Documento:** representacion-documento-escpos_v1.1.md
**Versión:** 1.1
**Estado:** Aprobado
**Fecha:** 2026-04-25
**Autor:** Equipo de Arquitectura / Renderización
**Glosario:** [glosario_v1.0.md](glosario_v1.0.md)

---

# 1. Introducción

Este documento describe la **representación de documentos en formato ESC/POS** dentro del motor DSL. Define cómo una *representación abstracta* es transformada en una secuencia de comandos compatibles con impresoras térmicas que utilizan el estándar EPSON ESC/POS.

La representación ESC/POS constituye una de las salidas principales del motor y está orientada a **dispositivos físicos** con restricciones de ancho, encoding y capacidades específicas. Los términos *representación abstracta*, *renderizador*, *perfil de dispositivo* siguen las definiciones de [glosario_v1.0.md](glosario_v1.0.md).

---

# 2. Alcance

Este documento cubre:

* Transformación de la representación abstracta a comandos ESC/POS
* Mapeo de elementos del documento a comandos canónicos del estándar
* Manejo de layout en impresoras térmicas
* Matriz de encoding y comportamiento ante caracteres no representables
* Algoritmo de columnado de tablas simuladas
* Construcción de secuencias de impresión

No incluye:

* Implementación interna del parser DSL
* Lógica de resolución de datos
* Renderizadores distintos a ESC/POS

---

# 3. Concepto de representación ESC/POS

La representación ESC/POS consiste en una **secuencia de bytes** que contiene comandos específicos entendidos por impresoras térmicas compatibles. Estos comandos permiten controlar:

* Texto y formato (negrita, alineación, tamaño)
* Imágenes raster
* Saltos de línea y avance de papel
* Corte de papel
* Códigos de barras y QR (según soporte)

El motor traduce la representación abstracta en esta secuencia respetando el perfil del dispositivo.

---

# 4. Estructura general de la salida

La salida ESC/POS se compone de:

1. Inicialización de la impresora
2. Configuración de encoding (code page)
3. Configuración inicial de estilos
4. Renderizado secuencial de bloques
5. Inserción de elementos especiales (QR, imagen, barcode)
6. Finalización del documento
7. Comando de corte (opcional según perfil)

---

# 5. Mapeo de elementos abstractos a comandos ESC/POS

Esta sección reemplaza al pseudocódigo `[NEGRITA]…[/NEGRITA]` por las **secuencias canónicas** del estándar EPSON. Si una variante difiere por modelo de impresora, se anota.

| Elemento | Mnemónico | Hex | Propósito | Notas |
|---|---|---|---|---|
| Inicialización | `ESC @` | `1B 40` | Reset al estado por defecto | Recomendado al inicio de cada job |
| Alineación izquierda | `ESC a 0` | `1B 61 00` | Justifica a la izquierda | |
| Alineación centro | `ESC a 1` | `1B 61 01` | Centra el contenido | Afecta texto e imágenes |
| Alineación derecha | `ESC a 2` | `1B 61 02` | Justifica a la derecha | |
| Negrita ON | `ESC E 1` | `1B 45 01` | Activa enfatizado | |
| Negrita OFF | `ESC E 0` | `1B 45 00` | Desactiva enfatizado | |
| Subrayado ON | `ESC - 1` / `ESC - 2` | `1B 2D 01` / `1B 2D 02` | Subrayado 1 o 2 puntos | |
| Subrayado OFF | `ESC - 0` | `1B 2D 00` | Desactiva subrayado | |
| Tamaño normal | `GS ! 0` | `1D 21 00` | Restablece x1 | `GS !` combina ancho/alto |
| Doble alto | `GS ! 0x01` | `1D 21 01` | Alto x2, ancho x1 | Mapea a `fontSize: "large"` |
| Doble ancho | `GS ! 0x10` | `1D 21 10` | Ancho x2, alto x1 | |
| Doble alto+ancho | `GS ! 0x11` | `1D 21 11` | Ambos x2 | |
| Salto de línea | `LF` | `0A` | Avanza una línea | |
| Avance n líneas | `ESC d n` | `1B 64 n` | Avance vertical (0–255) | `ESC J n` avanza n puntos verticales |
| Code page | `ESC t n` | `1B 74 n` | Selecciona tabla de caracteres | `n=0` CP437, `n=19` CP858, `n=16` WPC1252 (varía por modelo) |
| Bitmap raster | `GS v 0 m xL xH yL yH d…` | `1D 76 30 m xL xH yL yH …` | Imprime imagen raster | `m`: 0 normal, 1 doble ancho, 2 doble alto, 3 cuádruple |
| QR — modelo | `GS ( k` (fn 65) | `1D 28 6B 04 00 31 41 32 00` | Define modelo QR = 2 | |
| QR — tamaño módulo | `GS ( k` (fn 67) | `1D 28 6B 03 00 31 43 n` | `n` típico 1–16 | |
| QR — corrección | `GS ( k` (fn 69) | `1D 28 6B 03 00 31 45 n` | `n=48..51` (L,M,Q,H) | |
| QR — datos | `GS ( k` (fn 80) | `1D 28 6B pL pH 31 50 30 d…` | Carga datos en buffer | `pL/pH` = longitud + 3 |
| QR — imprimir | `GS ( k` (fn 81) | `1D 28 6B 03 00 31 51 30` | Imprime QR del buffer | Requiere firmware compatible (TM-T20+) |
| CODE128 | `GS k 73 n d…` | `1D 6B 49 n d…` | Imprime barcode | Datos requieren selector `{A`, `{B` o `{C` |
| Corte total | `GS V 0` | `1D 56 00` | Corte completo | Variante con avance: `GS V 65 n` |
| Corte parcial | `GS V 1` | `1D 56 01` | Corte parcial | Disponibilidad depende del cutter |
| Buzzer | `ESC B n t` | `1B 42 n t` | Beep `n` veces | No estándar; muchas Epson no lo traen |

**Notas generales:**

* `ESC @` no afecta NV bit images ni cutter.
* En hardware sin soporte de QR nativo (anterior a ~2008), el motor renderiza el QR como bitmap raster generado por software (fallback `PROFILE-SUPPORT-002` Warning).
* Los códigos de page para `ESC t` difieren entre familias TM-T; consultar el manual específico del modelo.

---

# 6. Representación de elementos del documento

## 6.1 Texto

* Codificación según code page del perfil (ver §8)
* Soporte de estilos: negrita, subrayado, tamaño
* Alineación: izquierda / centro / derecha

**Ejemplo de mapeo:**

```text
DSL: { "type": "text", "content": "Total: $100.00", "style": { "bold": true } }
ESC/POS:
  1B 45 01                   // Negrita ON
  54 6F 74 61 6C ...         // "Total: $100.00" en CP858
  1B 45 00                   // Negrita OFF
  0A                         // LF
```

---

## 6.2 Alineación

Activada vía `ESC a n`. Afecta a todo el texto/imagen subsiguiente hasta el próximo `ESC a`.

---

## 6.3 Líneas y separadores

Se generan dinámicamente según el ancho del perfil:

```text
ancho=32 → "--------------------------------"
ancho=42 → "------------------------------------------"
```

---

## 6.4 Bloques de contenido

Representan secciones (encabezado, detalle, totales, pie). Cada bloque se renderiza secuencialmente respetando el layout calculado por el motor.

---

## 6.5 Tablas simuladas

ESC/POS no soporta tablas nativas; se simulan con espaciado manual. Ver el algoritmo completo en §10.

---

## 6.6 Imágenes

* Conversión a bitmap monocromo
* Escalado al ancho del perfil (preserva aspect ratio)
* Comando: `GS v 0`

---

## 6.7 Códigos QR y barras

* QR: secuencia `GS ( k` (ver §5)
* Barras: `GS k 73`
* Si el perfil declara `supports.qr = false`, el motor emite `PROFILE-SUPPORT-002` y degrada a placeholder textual o a bitmap del QR (configurable).

---

## 6.8 Saltos de línea

* `LF` (`0x0A`) entre bloques
* `ESC d n` para avance de varias líneas

---

## 6.9 Corte de papel

* Total: `GS V 0`
* Parcial: `GS V 1`
* Solo si el perfil declara `supports.cutter = true`

---

# 7. Flujo de renderizado a ESC/POS

```text
Plantilla DSL + Datos + Perfil
        ↓
Modelo interno
        ↓
Motor de Layout
        ↓
Representación abstracta
        ↓
Renderizador ESC/POS
        ↓
Secuencia de comandos (bytes)
        ↓
Impresora
```

---

# 8. Encoding y matriz de compatibilidad

## 8.1 Decisión vigente

* **Encoding default:** `CP858` (variante de CP437 con Euro y acentos latinos).
* **Encodings declarables por perfil:** `CP437`, `CP858`, `CP850`, `ISO-8859-1`, `ISO-8859-15`.
* **UTF-8** solo en perfiles que lo declaren explícitamente con `supports.utf8 = true` (raro en hardware térmico estándar). En el resto de los renderizadores (preview UI, texto plano, PDF), UTF-8 es nativo.

## 8.2 Estructura del perfil

```text
encoding.default: "CP858"
encoding.supported: ["CP437", "CP858", "CP850", "ISO-8859-1"]
encoding.fallback_chain: ["CP858", "ISO-8859-1", "ASCII"]
```

## 8.3 Pipeline de conversión por carácter

Para cada carácter del texto a imprimir:

1. Intentar codificar en `encoding.default`.
2. Si falla, intentar **transliteración** vía tabla canónica:
   * `á → a`, `é → e`, `í → i`, `ó → o`, `ú → u`
   * `Á → A`, `Ñ → N`, etc.
   * `€ → EUR`, `° → o`, `¿ → ?`, `– → -`, `… → ...`
   * `" → "`, `' → '`
3. Si la transliteración produce un carácter representable, **usarlo y emitir Warning** `RENDER-ENCODING-002`.
4. Si tras transliterar sigue sin representarse, sustituir por `?` y emitir Warning `RENDER-ENCODING-003` con `Context = { char, codepoint, position }`.

## 8.4 Matriz de soporte por code page

| Caracter | CP437 | CP858 | CP850 | ISO-8859-1 |
|---|---|---|---|---|
| á é í ó ú | ✗ | ✓ | ✓ | ✓ |
| Ñ ñ | ✓ (parcial) | ✓ | ✓ | ✓ |
| € | ✗ | ✓ | ✗ | ✗ |
| ° | ✓ | ✓ | ✓ | ✓ |
| ¿ ¡ | ✗ | ✓ | ✓ | ✓ |
| – — (em/en dash) | ✗ | ✗ | ✗ | ✗ → translit `-` |
| " " ' ' | ✗ | ✗ | ✗ | ✗ → translit `" '` |

## 8.5 Alternativa registrada (no adoptada)

**Opción mínima descartada:** CP437 puro con sustitución silenciosa por `?`. Se rechazó porque acentos y `ñ` se perderían sin warning, generando tickets ilegibles en español.

---

# 9. Consideraciones de layout

El renderizado a ESC/POS debe considerar:

* Ancho máximo en caracteres (típicos: 32 / 42 / 48)
* Espaciado entre columnas (1 char por defecto)
* Wrapping de líneas largas (greedy word-wrap)
* Truncado con `…` si la columna lo declara
* Alineación consistente

El layout se adapta dinámicamente según el perfil del dispositivo.

---

# 10. Algoritmo de columnado de tablas simuladas

## 10.1 Inputs

* Fila con `N` columnas, cada una con `width` (numérico, porcentual u omitido) y opcionalmente `overflow ∈ {wrap, truncate, ellipsis}` (default `wrap`).
* Ancho del perfil `W` en caracteres.
* Separador entre columnas `S = 1` char.

## 10.2 Pasos

**Paso 1 — Normalización de widths**

* Si todas las columnas declaran `width` numérico → modo *fijo*.
* Si todas declaran porcentaje → modo *proporcional*.
* Si alguna omite `width` → repartir el remanente por igual entre las omitidas (modo *mixto*).
* Si hay mezcla numérico + porcentual → reservar lo numérico, repartir el resto proporcionalmente.

**Paso 2 — Ancho disponible**

```text
W_avail = W − S × (N − 1)
```

**Paso 3 — Cálculo de anchos crudos**

* Modo fijo: `w_i = width_i`.
* Modo proporcional: `w_i = floor(W_avail × pct_i)`.
* Distribuir el residuo `W_avail − Σw_i` con **largest-remainder method** (sumar 1 char a las columnas con mayor parte fraccionaria).

**Paso 4 — Detección de overflow**

Si `Σw_i > W_avail`:

1. **Reducción proporcional** con piso: `w_i = max(MIN_COL, floor(w_i × W_avail / Σw_i))` con `MIN_COL = 3`.
2. Si tras escalar sigue desbordando (porque el piso de 3 lo impide), emitir Warning `RENDER-LAYOUT-002` y truncar columnas desde la derecha hasta caber.
3. En modo fijo puro, además emitir Error `DSL-VALIDATION-005` (halts).

**Paso 5 — Render de cada celda con su `w_i`**

* Si `len(texto) ≤ w_i`: aplicar padding según `alignment` de la columna.
* Si `len(texto) > w_i`: aplicar política `overflow`:
  * `wrap` (default): word-wrap greedy; si una palabra excede `w_i`, cortar por carácter.
  * `truncate`: cortar a `w_i` caracteres.
  * `ellipsis`: cortar a `w_i − 1` y agregar `…` (o `>` si CP437 sin Unicode).

**Paso 6 — Multilínea**

Para una fila, `H = max(líneas_celda_i)`. Las celdas con menos líneas se rellenan con espacios `w_i`. Se emiten `H` líneas físicas separadas por `S` espacios entre columnas.

**Paso 7 — Salida**

Cada línea termina con `LF` (`0x0A`); se aplica la alineación global del `row` (left por defecto).

## 10.3 Ejemplo

Perfil 32 chars, columnas con porcentajes `["50%", "30%", "20%"]`:

```text
W_avail = 32 − 2 = 30
w = [15, 9, 6]
```

Celda `"Café con leche extra"` en col 0 con `wrap`:

```text
Línea 1: "Café con leche "
Línea 2: "extra          "
```

## 10.4 Alternativa registrada (no adoptada)

**Opción mínima descartada:** ancho fijo declarado, truncado duro, sin wrap, sin reducción proporcional. Se rechazó porque obligaría al diseñador de plantillas a recalcular anchos por perfil, rompiendo la promesa de portabilidad de NB-04 (multi-dispositivo).

---

# 11. Limitaciones del formato ESC/POS

* No soporta layouts complejos tipo HTML
* No tiene posicionamiento absoluto
* Tipografías limitadas (las del firmware)
* Dependiente del hardware
* Variaciones entre fabricantes (especialmente en code pages y QR)

Estas limitaciones son manejadas por el motor mediante adaptación del contenido.

---

# 12. Estrategias de adaptación

El motor puede aplicar:

* Ajuste de ancho de columnas (§10)
* Reducción proporcional con piso (§10 paso 4)
* Reemplazo de elementos no soportados (QR → bitmap o placeholder)
* Fallback de imágenes a texto cuando el perfil declara `supports.image = false`
* Transliteración de caracteres (§8.3)

Las estrategias se aplican **en este orden de prioridad**: encoding → layout → fallbacks de elementos.

---

# 13. Relación con otros componentes

Esta representación depende de:

* CU-05: Generar representación abstracta
* CU-06: Renderizar a ESC/POS
* CU-11: Seleccionar perfil de dispositivo
* CU-12: Adaptar documento al perfil
* RC-06: Perfil de dispositivo condiciona la salida

---

# 14. Ejemplo simplificado

Plantilla DSL:

```json
{
  "body": [
    { "type": "text", "content": "Ticket", "style": { "alignment": "center", "bold": true } },
    { "type": "separator" },
    { "type": "text", "content": "Total: 100" },
    { "type": "separator" }
  ]
}
```

Secuencia ESC/POS aproximada (perfil 32 chars, CP858):

```text
1B 40                          // ESC @     (init)
1B 74 13                       // ESC t 19  (CP858)
1B 61 01                       // ESC a 1   (centrar)
1B 45 01                       // ESC E 1   (negrita ON)
54 69 63 6B 65 74              // "Ticket"
1B 45 00 0A                    // negrita OFF + LF
1B 61 00                       // ESC a 0   (izquierda)
"--------------------------------" 0A
"Total: 100" 0A
"--------------------------------" 0A
1D 56 00                       // GS V 0    (corte total, si perfil lo soporta)
```

---

# 15. Casos de Uso Asociados

| Caso de Uso | Descripción                              |
| ----------- | ---------------------------------------- |
| CU-05       | Generar representación abstracta         |
| CU-06       | Renderizar a ESC/POS                     |
| CU-10       | Enviar a impresora Bluetooth             |
| CU-11       | Seleccionar perfil de dispositivo        |
| CU-12       | Adaptar documento al perfil              |
| CU-22       | Validar perfil de dispositivo            |
| CU-23       | Aplicar restricciones del dispositivo    |
| CU-32       | Manejar errores de impresión             |

---

# 16. Historial de Versiones

| Versión | Fecha      | Autor               | Cambios                                                                                                                                                                                                |
| ------- | ---------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.0     | 2026-03-28 | Equipo Arquitectura | Versión inicial de representación ESC/POS                                                                                                                                                              |
| 1.1     | 2026-04-25 | Equipo Arquitectura | Mapeo a comandos ESC/POS canónicos (hex + mnemónico); matriz de encoding con CP858 default y transliteración; algoritmo de columnado de tablas con largest-remainder + reducción proporcional + wrap; vínculo al glosario. |

---

**Fin del documento**
