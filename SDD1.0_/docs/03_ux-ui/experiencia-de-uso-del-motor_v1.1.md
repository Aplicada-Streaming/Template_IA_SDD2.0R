# Experiencia de Uso del Motor DSL (Developer Experience)

**Proyecto:** Motor DSL de Documentos para Impresión
**Documento:** experiencia-de-uso-del-motor_v1.1.md
**Versión:** 1.1
**Estado:** Aprobado
**Fecha:** 2026-04-25
**Autor:** Equipo de Arquitectura / Funcional
**Glosario:** [glosario_v1.0.md](glosario_v1.0.md)

---

# 1. Introducción

Este documento describe la **experiencia de uso del motor DSL** desde la perspectiva de los sistemas consumidores (developers). Define cómo una aplicación cliente interactúa con el motor para generar documentos a partir de plantillas, datos y perfiles de dispositivo.

A diferencia de un sistema tradicional de UX/UI orientado a usuarios finales, este documento se centra en la **Developer Experience (DX)** — la experiencia de integración, configuración y ejecución del motor como librería reutilizable. Para definiciones precisas de *plantilla DSL*, *modelo interno*, *representación abstracta*, *renderizador* y *perfil de dispositivo*, ver [glosario_v1.0.md](glosario_v1.0.md).

---

# 2. Alcance

Este documento cubre:

* Personas y journeys del developer integrador
* Flujo de uso del motor
* Modelo de errores y diagnósticos canónicos
* Principios operacionalizables de DX
* Modos de uso (impresión, preview, debug)
* Configuración del motor

No incluye:

* Interfaces gráficas de usuario final
* Diseño visual de aplicaciones consumidoras
* Implementación interna del motor (ver [docs/05_arquitectura_tecnica/](../05_arquitectura_tecnica/))

---

# 3. Personas

## 3.1 Persona primaria — Developer integrador

**Nombre simulado:** *Diego, dev MAUI senior*.
**Rol:** desarrollador en una empresa que integra el motor en una app móvil de punto de venta.
**Conocimientos:** C# / .NET MAUI fluido; familiaridad con JSON; nunca trabajó con ESC/POS antes.
**Objetivo principal:** que su app imprima un ticket correcto en una térmica Bluetooth en el menor tiempo posible.
**Frustraciones típicas:** docs incompletas, errores opacos, tener que leer código fuente para entender una API, divergencia preview ↔ impresora.

**Métrica clave:** **TTFP (Time To First Print)** — tiempo desde "agregué el paquete NuGet" hasta "salió un ticket correcto en la impresora real". Objetivo: < 60 minutos para un perfil típico.

---

## 3.2 Persona secundaria — Diseñador de plantillas

**Nombre simulado:** *Carla, líder funcional / analista*.
**Rol:** define el contenido y disposición de tickets, recibos y comandas.
**Conocimientos:** estructura de tickets, requisitos de impresión fiscal; lee JSON pero no programa.
**Objetivo principal:** modificar plantillas sin pedir un release de la app.
**Frustraciones típicas:** errores de validación de plantilla sin ubicación del problema, no poder previsualizar el resultado.

**Métrica clave:** **iteraciones a plantilla válida** — cantidad de intentos hasta que la plantilla pasa validación. Objetivo: ≤ 3.

---

## 3.3 Persona terciaria — Operador de caja (usuario final)

**Nombre simulado:** *Mateo, cajero*.
**Rol:** dispara la impresión desde la app.
**Conocimientos:** ninguno técnico.
**Relación con el motor:** **indirecta**. No interactúa con el motor; pero un mal diseño de DX se refleja en mensajes de error visibles en la app.

**Implicación de diseño:** los mensajes de error del motor deben ser **suficientemente descriptivos** para que el desarrollador (Diego) los traduzca a algo accionable por Mateo (ej. "Impresora no responde — verificá Bluetooth").

---

# 4. Developer Journeys

## 4.1 Journey A — Primera integración (TTFP)

```text
1. Instalación      → Diego agrega NuGet al proyecto MAUI
2. Configuración    → Crea Engine y declara perfil de dispositivo
3. Primera plantilla→ Copia ejemplo de la doc (ticket-venta)
4. Primeros datos   → Inyecta JSON de prueba
5. Render preview   → Ve resultado en pantalla
6. Render real      → Manda a impresora BT
7. Iteración        → Ajusta plantilla, repite 3-5
```

**Puntos críticos:**

* Paso 2: claridad del API de configuración (¿obligatorio? ¿con defaults?).
* Paso 5: fidelidad del preview respecto al output real (ver [representacion-vista-previa-ui_v1.1.md](representacion-vista-previa-ui_v1.1.md) §7).
* Paso 6: claridad de errores de hardware.

---

## 4.2 Journey B — Diagnóstico de plantilla rota

```text
1. Edita plantilla  → Carla modifica un campo de la DSL
2. Carga al motor   → Diego prueba en la app
3. Falla validación → Motor devuelve diagnóstico
4. Localización     → Diego identifica nodo + línea afectada
5. Corrección       → Carla ajusta JSON
6. Re-validación    → Loop hasta éxito
```

**Punto crítico:** el diagnóstico (paso 3) debe ser **accionable**: código estable, mensaje, ubicación (path JSON), remedio sugerido. Ver §8.

---

## 4.3 Journey C — Cambio de perfil de dispositivo

```text
1. Negocio cambia   → De térmica 58mm a 80mm
2. Diego ajusta     → Cambia config: profile = "thermal-80mm-cp858"
3. Misma plantilla  → No requiere modificaciones
4. Misma data       → No requiere modificaciones
5. Render           → Salida adaptada al nuevo ancho automáticamente
```

**Punto crítico:** la **portabilidad de plantillas** entre perfiles es una promesa explícita del producto (NB-04). El algoritmo de columnado de tablas (ver [representacion-documento-escpos_v1.1.md](representacion-documento-escpos_v1.1.md) §10) preserva esa promesa.

---

# 5. Actores del sistema

## 5.1 Aplicación cliente

Sistema que utiliza el motor DSL para generar documentos.

**Ejemplo:** Aplicación .NET MAUI que imprime tickets desde una interfaz de usuario.

---

## 5.2 Motor DSL

Librería encargada de interpretar plantillas, procesar datos y generar representaciones renderizadas.

**Ejemplo:** Motor que convierte una plantilla DSL en comandos ESC/POS.

---

## 5.3 Fuente de datos

Origen de los datos utilizados por el documento.

**Ejemplo:** API REST, base de datos o modelo en memoria.

---

## 5.4 Dispositivo de salida

Destino final del documento renderizado.

**Ejemplo:** Impresora térmica Bluetooth o visor en pantalla.

---

# 6. Flujo de uso del motor

## 6.1 Visión general

La aplicación cliente sigue un flujo estructurado de etapas para generar un documento:

1. Inicialización del motor
2. Carga de plantilla DSL
3. Validación de plantilla
4. Provisión de datos
5. Selección de perfil de dispositivo
6. Construcción del documento
7. Renderizado
8. Consumo de la salida

Este flujo permite desacoplar completamente el contenido del documento de su representación final.

---

## 6.2 Orden de operaciones

* Los pasos **3 (validación)** y **6 (construcción)** son **obligatorios** y se ejecutan en ese orden.
* Los pasos **2 (plantilla)**, **4 (datos)** y **5 (perfil)** pueden cargarse en cualquier orden, pero los tres deben estar provistos antes del paso 6.
* La aplicación **puede** llamar a `Render(formato)` múltiples veces sobre el mismo modelo construido, cambiando solo el formato (ESC/POS, preview, texto plano).
* Cambiar el **perfil de dispositivo** requiere **reconstruir el documento** (volver al paso 6); cambiar **datos** requiere también reconstrucción; cambiar **plantilla** requiere validación + reconstrucción.

---

## 6.3 Inicialización

```text
Motor.Initialize(configuración)
```

**Objetivo:** preparar el entorno del motor para procesar documentos.

---

## 6.4 Carga de plantilla DSL

```text
Motor.CargarPlantilla(plantillaDSL)
```

**Resultado esperado:** la plantilla queda registrada y lista para validación.

---

## 6.5 Validación de plantilla

El motor valida que la plantilla cumpla con el esquema DSL.

* Si es válida: se acepta.
* Si no es válida: se devuelve un diagnóstico (ver §8) y **se detiene el flujo**.

---

## 6.6 Provisión de datos

```text
Motor.CargarDatos(datos)
```

**Consideraciones:**

* Los datos deben cumplir con la estructura esperada por las referencias `{{...}}` de la plantilla.
* Pueden provenir de APIs o fuentes internas.

---

## 6.7 Selección de perfil de dispositivo

```text
Motor.SeleccionarPerfil(perfilDispositivo)
```

**Objetivo:** adaptar la salida a restricciones como ancho, encoding o soporte de elementos.

---

## 6.8 Construcción del documento

El motor procesa plantilla + datos + perfil y construye el modelo interno y la representación abstracta.

**Resultado:** representación abstracta lista para uno o más renderizadores.

---

## 6.9 Renderizado

```text
Motor.Renderizar(Formato.ESC_POS)
Motor.Renderizar(Formato.PREVIEW_UI)
Motor.Renderizar(Formato.TEXTO_PLANO)
Motor.Renderizar(Formato.PDF)   // futuro
```

---

## 6.10 Consumo de la salida

La aplicación cliente recibe el resultado y lo utiliza:

* Enviar a impresora
* Mostrar en pantalla
* Guardar en archivo
* Log de debug

---

# 7. Modos de uso

## 7.1 Modo impresión

* Output: ESC/POS
* Adaptación a ancho del dispositivo
* Optimización de layout

## 7.2 Modo vista previa

* Output: componentes UI (MAUI)
* Simulación con tolerancia definida (ver [representacion-vista-previa-ui_v1.1.md](representacion-vista-previa-ui_v1.1.md))

## 7.3 Modo debug (texto plano)

* Output: string monoespaciado
* Útil para diagnóstico, snapshots y tests automatizados
* No depende de hardware

---

# 8. Modelo de errores y diagnósticos

El motor expone un modelo de **diagnósticos canónicos** estable, versionado y testeable. Esto reemplaza el manejo de errores genérico por una API explícita.

## 8.1 Estructura del diagnóstico

```text
Diagnostic {
  Code:     string      // Código estable (ej. "DSL-VALIDATION-002")
  Severity: enum         // Error | Warning | Info
  Message:  string       // Mensaje canónico (i18n-able)
  Hint:     string?      // Remedio sugerido
  Halts:    bool         // ¿Detiene el pipeline?
  Context:  dict         // path JSON, línea, char, codepoint, etc.
}
```

El resultado del motor expone una colección `Diagnostics` que contiene errores, warnings e infos juntos. La aplicación cliente filtra por severidad según necesite.

---

## 8.2 Convención de naming

`<DOMINIO>-<CATEGORÍA>-<NNN>` donde `DOMINIO ∈ {DSL, DATA, PROFILE, RENDER, IO}`, 3 dígitos.

---

## 8.3 Catálogo canónico (v1.0)

| Código | Severidad | Causa típica | Remedio | Halts |
|---|---|---|---|---|
| `DSL-VALIDATION-001` | Error | JSON malformado (parse fail) | Validar con linter JSON | Sí |
| `DSL-VALIDATION-002` | Error | Nodo desconocido (`type:"foo"`) | Usar tipos del catálogo DSL v1.0 | Sí |
| `DSL-VALIDATION-003` | Error | Atributo requerido ausente | Completar campo obligatorio | Sí |
| `DSL-VALIDATION-004` | Warning | Estilo no reconocido (`fontSize:"huge"`) | Usar `normal` \| `large` | No |
| `DSL-VALIDATION-005` | Error | Suma de `width` de columnas excede ancho del perfil (modo fijo) | Usar porcentajes o reducir anchos | Sí |
| `DATA-RESOLUTION-001` | Error | Referencia `{{x.y}}` no resuelta y sin default | Proveer dato o default | Sí |
| `DATA-RESOLUTION-002` | Warning | Loop sobre array vacío | Validar fuente de datos | No |
| `DATA-RESOLUTION-003` | Warning | Tipo inesperado (coerción aplicada) | Revisar mapeo | No |
| `PROFILE-SUPPORT-001` | Error | Perfil declarado no existe en catálogo | Cargar perfil válido | Sí |
| `PROFILE-SUPPORT-002` | Warning | Elemento `qr` / `image` no soportado por perfil | Fallback a texto/placeholder | No |
| `RENDER-ENCODING-001` | Info | Encoding default aplicado | — | No |
| `RENDER-ENCODING-002` | Warning | Carácter transliterado (ej. `á → a`) | Considerar perfil con encoding más amplio | No |
| `RENDER-ENCODING-003` | Warning | Carácter sustituido por `?` | Revisar contenido o cambiar perfil | No |
| `RENDER-LAYOUT-001` | Warning | Línea truncada por exceder ancho | Reducir contenido o cambiar perfil | No |
| `RENDER-LAYOUT-002` | Warning | Reducción proporcional de columnas aplicada | Revisar diseño de tabla | No |
| `IO-PRINTER-001` | Error | Impresora no responde / timeout | Verificar conexión BT y estado | Sí |
| `IO-PRINTER-002` | Error | Buffer rechazado por dispositivo | Reintentar o reducir tamaño | Sí |

---

## 8.4 Principios del modelo de errores

* **No ejecución parcial silenciosa:** si un error es `Halts: true`, el pipeline se detiene y no se entrega salida parcial.
* **Trazabilidad:** todos los diagnósticos llevan `Context` con ubicación precisa (path JSON, posición en cadena, perfil involucrado, etc.).
* **Estabilidad:** los `Code` son estables entre versiones menores. Una baja se anuncia con dos versiones de anticipación.
* **Testeabilidad:** los tests deben asertar contra `Code`, nunca contra el `Message` (que puede traducirse o reformularse).

---

## 8.5 Decisión vigente y alternativa registrada

**Decisión vigente (default v1.1):** el modelo robusto descrito arriba (con `Diagnostic` + 17 códigos canónicos).

**Alternativa registrada (no adoptada):** un enum plano `EngineErrorCode` con ~6 valores genéricos (`InvalidTemplate`, `MissingData`, `UnsupportedProfile`, `RenderError`, `IoError`). Más simple, menos accionable. Se descartó porque sin códigos estables no se pueden escribir tests ni documentación de troubleshooting.

---

# 9. Configuración del motor

La configuración incluye:

* Plantillas disponibles
* Fuentes de datos
* Renderizadores habilitados
* Perfiles de dispositivos
* Estrategias de layout

**Ejemplo:**

```text
Config:
- Renderizador: ESC/POS
- Perfil: thermal-58mm-cp858
- Fuente de datos: API REST
```

---

# 10. Principios operacionalizables de DX

Reescritura de "consideraciones de experiencia" en términos verificables:

| Principio | Operacionalización |
|-----------|---------------------|
| **Fácil de integrar** | TTFP < 60 min para un perfil estándar (ver §3.1). Un solo paquete NuGet, sin dependencias nativas obligatorias. |
| **API consistente** | Todos los métodos públicos siguen patrón `Verb + Noun` (`CargarPlantilla`, `SeleccionarPerfil`). Errores siempre vía `Diagnostic`, nunca por excepción `Exception` excepto en violaciones de contrato (null en parámetros obligatorios). |
| **API predecible** | Métodos puros: misma plantilla + mismos datos + mismo perfil → misma salida byte-exacta (excepto timestamps explícitos). |
| **Flujo claro y secuencial** | Las dependencias entre pasos están documentadas en §6.2. El motor expresa pre-condiciones violadas con `DSL-VALIDATION-*` o excepciones `InvalidOperationException` con mensaje guía. |
| **Separación explícita** | Plantilla, datos, perfil y formato son **cuatro inputs distintos**, nunca embebidos uno dentro de otro. |
| **Comportamiento determinista** | El motor no consulta servicios externos, no usa hora del sistema, no genera identificadores aleatorios salvo que se le pasen explícitamente. |
| **Diagnóstico accionable** | Cada `Diagnostic` lleva `Hint` y `Context` (ver §8.1). |

---

# 11. Casos de Uso Asociados

| Caso de Uso | Descripción                       |
| ----------- | --------------------------------- |
| CU-13       | Cargar plantilla DSL              |
| CU-14       | Validar plantilla DSL             |
| CU-15       | Resolver referencias a datos      |
| CU-18       | Cargar datos del documento        |
| CU-21       | Cargar perfil de dispositivo      |
| CU-29       | Extender motor con renderizadores |
| CU-06       | Renderizar a ESC/POS              |
| CU-07       | Renderizar a texto plano          |
| CU-08       | Renderizar vista previa UI        |
| CU-30       | Manejar errores de plantilla      |
| CU-31       | Manejar errores de datos          |
| CU-32       | Manejar errores de impresión      |

---

# 12. Historial de Versiones

| Versión | Fecha      | Autor               | Cambios                                                                                                                                            |
| ------- | ---------- | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.0     | 2026-03-28 | Equipo Arquitectura | Versión inicial del documento UX del motor                                                                                                         |
| 1.1     | 2026-04-25 | Equipo Arquitectura | Agregadas Personas y Developer Journeys; modelo canónico de diagnósticos con 17 códigos; reglas explícitas de orden de operaciones; principios DX operacionalizables; vínculo al glosario. |

---

**Fin del documento**
