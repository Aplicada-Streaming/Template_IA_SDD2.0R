# Caso de Uso: Adaptar Documento al Perfil de Dispositivo

**Código:** CU-12
**Archivo:** CU-12-adaptar-documento-perfil_v1.1.md
**Versión:** 1.1
**Estado:** Aprobado
**Fecha:** 2026-04-25
**Autor:** Equipo Funcional / Arquitectura
**Glosario:** [docs/03_ux-ui/glosario_v1.0.md](../../03_ux-ui/glosario_v1.0.md)

---

# 1. Propósito

Este caso de uso describe el proceso mediante el cual el motor DSL adapta el documento a las capacidades y restricciones definidas en el perfil del dispositivo seleccionado.

El objetivo es garantizar que el documento sea compatible con el hardware de destino, ajustando contenido, formato y comportamiento según las limitaciones del dispositivo. La adaptación incluye **ancho útil**, **encoding**, **columnado de tablas** y **fallbacks** para elementos no soportados.

---

# 2. Actores

## Actor Primario

**Motor DSL** — Responsable de adaptar el documento.

## Actores Secundarios

**Perfil de Dispositivo** — Define restricciones y capacidades.
**Motor de Layout** — Aplica ajustes estructurales (incluido el algoritmo de columnado).
**Renderizadores** — Consumen el documento adaptado.

---

# 3. Precondiciones

* Existe un documento con datos resueltos (CU-02, CU-15).
* Existe un perfil de dispositivo seleccionado (CU-11, CU-21).
* El motor DSL se encuentra inicializado.

---

# 4. Postcondiciones

## En caso de éxito

* El documento es adaptado según el perfil del dispositivo.
* Las tablas tienen anchos de columna calculados según el algoritmo canónico (§5).
* Los caracteres no representables en el encoding del perfil están transliterados o sustituidos.
* Los elementos no soportados (QR, imagen) tienen fallback aplicado o están marcados.
* El documento queda listo para layout final y renderizado.

## En caso de fallo

* No se logra adaptar el documento.
* El motor emite diagnósticos canónicos (`PROFILE-SUPPORT-*`, `RENDER-ENCODING-*`, `RENDER-LAYOUT-*`, `DSL-VALIDATION-005`) según corresponda.
* Si algún diagnóstico es `Halts: true`, el flujo se detiene.

---

# 5. Flujo Principal

1. El sistema inicia el proceso de adaptación.
2. El motor DSL recibe:

   * DocumentoConDatosResueltos
   * PerfilDispositivo
3. El sistema analiza las capacidades del perfil:

   * ancho útil en caracteres
   * encoding default y matriz soportada
   * soporte de QR / imagen / corte / buzzer
   * estilos disponibles
4. El sistema recorre el documento y, para cada nodo:

   * verifica compatibilidad con el dispositivo
   * ajusta propiedades (alineación, formato, tamaño)
5. **Adaptación de tablas:** para cada `TableNode`, el motor aplica el algoritmo de columnado canónico definido en [docs/03_ux-ui/representacion-documento-escpos_v1.1.md](../../03_ux-ui/representacion-documento-escpos_v1.1.md) §10:

   * Normalización de `width` (fijo, proporcional, mixto)
   * Cálculo de anchos crudos con largest-remainder
   * Detección de overflow y reducción proporcional con piso `MIN_COL = 3`
   * Aplicación de política `overflow` por columna (`wrap` / `truncate` / `ellipsis`)
   * Generación de líneas multilínea consistentes
6. **Adaptación de encoding:** para cada texto del documento, el motor aplica el pipeline de conversión definido en [docs/03_ux-ui/representacion-documento-escpos_v1.1.md](../../03_ux-ui/representacion-documento-escpos_v1.1.md) §8.3:

   * Intentar codificar en `encoding.default` del perfil
   * Si falla, transliterar (con Warning `RENDER-ENCODING-002`)
   * Si la transliteración no resuelve, sustituir por `?` (con Warning `RENDER-ENCODING-003`)
7. **Fallbacks de elementos:** en caso de contenido no soportado por el perfil:

   * QR → bitmap raster (si `supports.image`) o placeholder textual
   * Imagen → placeholder textual `[IMG: nombre.png]` si `supports.image == false`
   * Corte → omitido si `supports.cutter == false`
   * Cada fallback genera un Warning `PROFILE-SUPPORT-002`
8. Se aplican reglas de adaptación globales.
9. Se valida la consistencia del documento adaptado.
10. El sistema confirma la adaptación exitosa.
11. El caso de uso finaliza.

---

# 6. Flujos Alternativos

## FA-01 Elemento no compatible

**En el paso 7**

* Si el dispositivo no soporta el elemento:

  * El motor aplica fallback configurado.
  * Emite Warning `PROFILE-SUPPORT-002`.

---

## FA-02 Perfil incompleto

**En el paso 3**

* Si faltan datos en el perfil:

  * El motor aplica valores por defecto y emite Warning.
  * Si faltan campos críticos (ancho, encoding) emite Error `PROFILE-SUPPORT-001` (Halts).

---

## FA-03 Tabla con anchos desbordados (modo fijo)

**En el paso 5**

* Si en modo *fijo* la suma de `width` excede el ancho útil del perfil:

  * El motor emite Error `DSL-VALIDATION-005` (Halts).
  * Sugerencia: usar porcentajes para portabilidad cross-perfil.

---

## FA-04 Tabla con anchos desbordados (modo proporcional o mixto)

**En el paso 5**

* Si tras la reducción proporcional con piso `MIN_COL = 3` la suma sigue desbordando:

  * El motor trunca columnas desde la derecha hasta caber.
  * Emite Warning `RENDER-LAYOUT-002`.

---

## FA-05 Carácter no representable

**En el paso 6**

* Tras transliterar, si el carácter sigue sin caber en ningún encoding del fallback chain:

  * Se sustituye por `?`.
  * Emite Warning `RENDER-ENCODING-003` con `Context = { char, codepoint, position }`.

---

## FA-06 Error técnico

**En cualquier punto**

* El motor captura excepciones internas (violaciones de contrato).
* Emite Error genérico y registra el evento.
* El caso de uso finaliza.

---

# 7. Reglas de Negocio Relacionadas

* RN-01: Todo documento debe adaptarse al perfil antes de renderizarse.
* RN-02: Los elementos deben respetar las capacidades del dispositivo.
* RN-03: Se deben aplicar reglas de degradación controlada.
* RN-06: El perfil de dispositivo condiciona la salida final.

---

# 8. Datos Utilizados

## Entrada

* DocumentoConDatosResueltos (`DocumentNode`)
* PerfilDispositivo (`DeviceProfile`)

## Salida

* DocumentoAdaptado (`DocumentNode`) — listo para renderizado
* Diagnósticos acumulados (`IList<EngineDiagnostic>`)

---

# 9. Criterios de Aceptación

## CA-01 Adaptación exitosa

**Dado** un documento y perfil válidos
**Cuando** se adapta
**Entonces** el documento es compatible con el dispositivo y no hay diagnósticos `Halts: true`.

---

## CA-02 Manejo de limitaciones

**Dado** un dispositivo limitado (ej. sin soporte de QR)
**Cuando** se procesa un documento con QR
**Entonces** el QR se reemplaza por bitmap o placeholder y se emite Warning `PROFILE-SUPPORT-002`.

---

## CA-03 Algoritmo de columnado proporcional

**Dado** una tabla con `width: ["50%", "30%", "20%"]` y un perfil de 32 caracteres
**Cuando** se adapta
**Entonces** los anchos resultantes son `[15, 9, 6]` (con largest-remainder), separados por 1 char, totalizando 30 caracteres útiles.

---

## CA-04 Algoritmo de columnado: overflow con wrap

**Dado** una celda con texto `"Café con leche extra"` en una columna de 15 caracteres y `overflow: "wrap"`
**Cuando** se adapta
**Entonces** el texto se divide en dos líneas: `"Café con leche "` y `"extra          "` (padding a 15).

---

## CA-05 Algoritmo de columnado: overflow con ellipsis

**Dado** una celda con texto `"Café con leche extra"` en una columna de 15 caracteres y `overflow: "ellipsis"`
**Cuando** se adapta
**Entonces** el texto se trunca a `"Café con leche…"` (14 chars + `…`).

---

## CA-06 Algoritmo de columnado: error en modo fijo desbordado

**Dado** una tabla con `width: [20, 20, 20]` y un perfil de 32 caracteres
**Cuando** se adapta
**Entonces** se emite Error `DSL-VALIDATION-005` (Halts) y la salida es `null`.

---

## CA-07 Encoding: transliteración de acentos

**Dado** un perfil con `encoding.default = "CP437"` y texto `"Café"`
**Cuando** se adapta
**Entonces** el texto se transforma a `"Cafe"` (transliteración) y se emite Warning `RENDER-ENCODING-002`.

---

## CA-08 Encoding: sustitución por `?`

**Dado** un perfil con encoding sin soporte para `€` ni transliteración configurada
**Cuando** se adapta texto con `€`
**Entonces** se sustituye por `?` y se emite Warning `RENDER-ENCODING-003` con `Context.char = "€"`.

---

## CA-09 Manejo de errores

**Dado** un problema en la adaptación
**Cuando** ocurre
**Entonces** el motor emite el diagnóstico canónico apropiado y, si es `Halts: true`, detiene el flujo.

---

# 10. Cambios respecto a versión 1.0

* **Algoritmo de columnado canónico (§5 paso 5):** se incorpora la referencia al algoritmo definido en [docs/03_ux-ui/representacion-documento-escpos_v1.1.md](../../03_ux-ui/representacion-documento-escpos_v1.1.md) §10 con largest-remainder, reducción proporcional con piso 3, y políticas `wrap` / `truncate` / `ellipsis`.
* **Pipeline de encoding (§5 paso 6):** se incorpora la referencia al pipeline de transliteración + fallback chain definido en el mismo documento §8.3.
* **Fallbacks de elementos (§5 paso 7):** se documentan reglas explícitas para QR, imagen, corte y buzzer.
* **Diagnósticos canónicos:** los flujos alternativos referencian códigos del catálogo (`DSL-VALIDATION-005`, `RENDER-LAYOUT-002`, `RENDER-ENCODING-002/003`, `PROFILE-SUPPORT-001/002`).
* **Criterios de aceptación verificables:** CA-03 a CA-08 incorporan ejemplos numéricos verificables por test.
* **Terminología:** se alinea con [docs/03_ux-ui/glosario_v1.0.md](../../03_ux-ui/glosario_v1.0.md) ("perfil de dispositivo", "renderizador").

---

# 11. Notas

Este caso de uso es fundamental para garantizar la **portabilidad** del sistema (NB-04: multi-dispositivo), permitiendo que un mismo documento funcione correctamente en diferentes dispositivos sin necesidad de redefinir su estructura. La adopción de columnado proporcional con `width` porcentual es la clave de esa portabilidad — el modo de ancho fijo se conserva por compatibilidad pero **se desaconseja** para plantillas que deban operar sobre múltiples perfiles.

---

# 12. Historial de Versiones

| Versión | Fecha      | Autor               | Cambios                                                                                                                                                       |
| ------- | ---------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.0     | 2026-03-28 | Equipo Funcional    | Versión inicial del caso de uso                                                                                                                               |
| 1.1     | 2026-04-25 | Equipo Funcional    | Algoritmo de columnado canónico; pipeline de encoding con transliteración; fallbacks explícitos; criterios de aceptación verificables; alineación con glosario y catálogo de diagnósticos. |

---

**Fin del documento**
