# Regla de Negocio: Perfil del Dispositivo Condiciona la Salida

**Código:** RN-06
**Archivo:** RN-06-perfil-dispositivo-condiciona-salida_v1.0.md
**Versión:** 1.0
**Estado:** Aprobada
**Fecha:** 2026-03-28
**Autor:** Equipo Funcional / Arquitectura

---

# 1. Descripción

Esta regla de negocio establece que el perfil del dispositivo debe condicionar las características de la salida final del renderizado.

El objetivo es garantizar que el motor adapte automáticamente la salida según las capacidades y restricciones del hardware destino.

---

# 2. Motivación de Negocio

Distintos dispositivos tienen diferentes capacidades (ancho, soporte de imágenes, QR, corte). El motor debe adaptar la salida automáticamente.

La adaptación basada en perfil permite:

* soportar múltiples modelos de impresoras con una única lógica de renderizado
* evitar errores de impresión por envío de comandos no soportados
* optimizar la salida para cada dispositivo específico
* facilitar la incorporación de nuevos dispositivos sin cambios en el motor

---

# 3. Definición de la Regla

El perfil del dispositivo (DeviceProfile) define los parámetros y capacidades del hardware destino. El renderizador debe consultar este perfil para adaptar la salida (truncar texto, omitir imágenes, ajustar anchos).

El perfil debe contemplar:

* ancho de impresión (en caracteres o milímetros)
* soporte de imágenes
* soporte de códigos QR
* soporte de códigos de barras
* soporte de corte de papel
* codificación de caracteres soportada
* comandos disponibles

---

# 4. Condiciones de Aplicación

La regla aplica en los siguientes casos de uso:

* CU-11 Configurar perfil de dispositivo
* CU-12 Seleccionar perfil de dispositivo
* CU-21 Adaptar salida según ancho
* CU-22 Adaptar salida según capacidades
* CU-23 Validar compatibilidad de salida

Debe aplicarse durante la fase de renderizado, cuando el documento abstracto se transforma en salida concreta.

---

# 5. Resultados Esperados

* La salida se adapta al ancho del dispositivo.
* Las características no soportadas se omiten o reemplazan.
* Dos dispositivos distintos producen salidas diferentes para el mismo documento.
* No se envían comandos no soportados al dispositivo.

---

# 6. Excepciones

## EX-01 Perfil no especificado

Si no se proporciona un perfil de dispositivo:

* El motor debe usar un perfil por defecto.
* Debe registrarse una advertencia indicando el uso del perfil por defecto.

---

## EX-02 Capacidad desconocida

Si el documento requiere una capacidad no definida en el perfil:

* El renderizador debe omitir el elemento o usar un fallback seguro.
* Debe registrarse el evento para diagnóstico.

---

# 7. Criterios de Validación

## CV-01 Ajuste de ancho por perfil

**Dado** un perfil de 58mm
**Cuando** se renderiza un documento
**Entonces** el ancho de línea se ajusta a 32 caracteres.

---

## CV-02 Omisión de imágenes no soportadas

**Dado** un perfil sin soporte de imagen
**Cuando** el documento tiene imágenes
**Entonces** se omiten o reemplazan por texto alternativo.

---

## CV-03 Diferenciación entre perfiles

**Dado** dos perfiles distintos
**Cuando** se renderiza el mismo documento
**Entonces** las salidas difieren según las capacidades de cada perfil.

---

# 8. Impacto

Esta regla impacta directamente en:

* CU-11 Configurar perfil de dispositivo
* CU-12 Seleccionar perfil de dispositivo
* CU-21 Adaptar salida según ancho
* CU-22 Adaptar salida según capacidades
* CU-23 Validar compatibilidad de salida

La correcta aplicación del perfil del dispositivo es esencial para la calidad de la salida impresa.

---

# 9. Control de Cambios

| Versión | Fecha      | Descripción     |
| ------- | ---------- | --------------- |
| 1.0     | 2026-03-28 | Versión inicial |

---

**Fin del documento**
