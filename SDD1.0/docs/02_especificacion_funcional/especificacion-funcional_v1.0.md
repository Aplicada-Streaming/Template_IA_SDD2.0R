# Especificación Funcional

**Proyecto:** Motor DSL de Documentos para Impresión
**Documento:** especificacion-funcional_v1.0.md
**Estado:** Aprobado
**Fecha:** 2026-03-28
**Autor:** Equipo de Arquitectura / Funcional

---

# 1. Introducción

Este documento describe el comportamiento funcional del motor DSL de documentos orientado a la generación y renderizado de salidas para distintos dispositivos de impresión y visualización.

Su propósito es detallar qué debe hacer el sistema desde la perspectiva funcional, incluyendo la interpretación de plantillas DSL, la resolución de datos, la construcción de un modelo interno del documento y su posterior renderizado a distintos formatos.

La especificación establece los procesos principales, reglas de negocio, actores involucrados y criterios de aceptación que validan que el sistema cumple con los requerimientos del dominio.

---

# 2. Alcance

El sistema permitirá:

* Cargar y validar plantillas DSL.
* Resolver datos dinámicos desde fuentes externas o internas.
* Construir un modelo interno del documento.
* Ejecutar un motor de layout.
* Renderizar el documento a múltiples formatos (ESC/POS, texto plano, vista previa UI).
* Adaptar la salida según perfiles de dispositivos.

Quedan fuera del alcance en esta versión:

* Integración con sistemas de pagos.
* Gestión de usuarios finales compleja (autenticación avanzada).
* Renderizado a formatos no definidos (PDF queda como futuro).

---

# 3. Actores

## 3.1 Sistema Cliente

Aplicación que consume el motor DSL para generar documentos.

**Ejemplo:** Aplicación .NET MAUI que envía datos y solicita renderizado.

---

## 3.2 Motor DSL

Componente central encargado de interpretar plantillas, procesar datos y generar salidas.

**Ejemplo:** Procesa una plantilla DSL y genera un ticket de impresión.

---

## 3.3 Fuente de Datos

Origen de los datos utilizados por el documento (API, base de datos, objetos en memoria).

**Ejemplo:** API REST que provee información de una orden.

---

## 3.4 Dispositivo de Salida

Hardware o software destino donde se renderiza el documento.

**Ejemplo:** Impresora Bluetooth ESC/POS o visor en pantalla.

---

# 4. Descripción General del Sistema

El sistema funciona como un motor de transformación de documentos basado en DSL.

El flujo general es:

1. Carga de plantilla DSL.
2. Validación de la plantilla.
3. Carga y validación de datos.
4. Resolución de referencias dinámicas.
5. Construcción del modelo interno del documento.
6. Ejecución del motor de layout.
7. Generación de representación abstracta.
8. Renderizado según el tipo de salida (ESC/POS, texto, UI).
9. Aplicación de perfil de dispositivo.
10. Envío al dispositivo de salida.

El sistema debe garantizar consistencia entre la plantilla, los datos y el dispositivo destino.

---

# 5. Funcionalidades Principales

## 5.1 Carga y Validación de Plantilla DSL

El sistema debe permitir cargar una plantilla DSL y validar su estructura conforme al esquema definido.

**Ejemplo:**
Se carga una plantilla de ticket de venta y se valida su sintaxis.

---

## 5.2 Resolución de Datos

El sistema debe resolver referencias a datos definidos en la plantilla utilizando fuentes externas o internas.

**Ejemplo:**
Una variable {{cliente.nombre}} se reemplaza por el valor real proveniente de la API.

---

## 5.3 Construcción del Modelo Interno

El sistema debe construir una representación estructurada del documento a partir de la plantilla y los datos resueltos.

**Ejemplo:**
Un árbol jerárquico que representa secciones, bloques y elementos del documento.

---

## 5.4 Evaluación de Lógica de Plantilla

El sistema debe evaluar condiciones e iteraciones definidas en la plantilla DSL.

**Ejemplo:**
Mostrar un bloque solo si el total es mayor a cero.

---

## 5.5 Motor de Layout

El sistema debe calcular la disposición visual o estructural de los elementos del documento.

**Ejemplo:**
Alinear columnas, ajustar saltos de línea, distribuir contenido según ancho disponible.

---

## 5.6 Generación de Representación Abstracta

El sistema debe producir una representación intermedia independiente del renderizador.

**Ejemplo:**
Un modelo abstracto que puede ser convertido a ESC/POS, texto o UI.

---

## 5.7 Renderizado a ESC/POS

El sistema debe convertir el documento a comandos compatibles con impresoras ESC/POS.

**Ejemplo:**
Generación de comandos para imprimir texto, líneas y códigos de barras.

---

## 5.8 Renderizado a Texto Plano

El sistema debe generar una representación textual del documento útil para debugging o logs.

**Ejemplo:**
Salida en consola con estructura legible del documento.

---

## 5.9 Renderizado a Vista Previa (UI)

El sistema debe permitir visualizar el documento en una interfaz gráfica.

**Ejemplo:**
Preview en pantalla dentro de una aplicación MAUI.

---

## 5.10 Selección de Perfil de Dispositivo

El sistema debe permitir seleccionar un perfil de dispositivo que defina capacidades y restricciones.

**Ejemplo:**
Perfil para impresora de 58mm vs 80mm.

---

## 5.11 Adaptación al Dispositivo

El sistema debe adaptar el documento según el perfil del dispositivo seleccionado.

**Ejemplo:**
Reducir ancho de columnas o truncar contenido si es necesario.

---

## 5.12 Envío a Impresora Bluetooth

El sistema debe enviar la salida renderizada a una impresora Bluetooth compatible.

**Ejemplo:**
Envío de bytes ESC/POS a una impresora portátil.

---

# 6. Reglas de Negocio

* RN-01: Toda plantilla debe cumplir el esquema DSL definido.
* RN-02: Solo se permiten elementos soportados por el motor.
* RN-03: La estructura del documento debe ser jerárquica.
* RN-04: Debe registrarse trazabilidad de ejecución y renderizado.
* RN-05: El documento abstracto no depende del renderizador.
* RN-06: El perfil del dispositivo condiciona la salida final.

**Ejemplo práctico:**
Una plantilla con elementos no soportados debe ser rechazada en validación.

---

# 7. Datos Principales

## 7.1 Documento

* DocumentoId
* PlantillaId
* DatosAsociados
* ModeloInterno
* RepresentacionAbstracta
* PerfilDispositivo
* ResultadoRenderizado

**Ejemplo:**
Documento de ticket generado para una orden de compra.

---

## 7.2 Plantilla DSL

* PlantillaId
* ContenidoDSL
* Version
* Estado

**Ejemplo:**
Plantilla de ticket con secciones de encabezado, detalle y totales.

---

## 7.3 Perfil de Dispositivo

* PerfilId
* TipoDispositivo
* AnchoCaracteres
* SoporteImagen
* SoporteQR

**Ejemplo:**
Perfil 58mm para impresoras térmicas.

---

# 8. Criterios de Aceptación

## CA-01 Carga de plantilla

**Dado** una plantilla DSL válida
**Cuando** se carga en el sistema
**Entonces** el sistema la acepta y la valida correctamente

---

## CA-02 Resolución de datos

**Dado** una plantilla con referencias a datos
**Cuando** se ejecuta el proceso
**Entonces** todas las referencias son resueltas correctamente

---

## CA-03 Renderizado exitoso

**Dado** un documento válido
**Cuando** se renderiza a un formato soportado
**Entonces** se genera una salida consistente con el perfil del dispositivo

---

## CA-04 Adaptación a dispositivo

**Dado** un perfil de dispositivo limitado
**Cuando** se renderiza un documento complejo
**Entonces** el sistema adapta la salida sin romper la estructura

---

## CA-05 Envío a impresora

**Dado** una salida ESC/POS válida
**Cuando** se envía a una impresora Bluetooth
**Entonces** el documento se imprime correctamente

---

# 9. Supuestos y Restricciones

Se asume que:

* Las plantillas DSL cumplen con el esquema definido.
* Las fuentes de datos están disponibles al momento de la ejecución.
* Los dispositivos de impresión soportan ESC/POS u otros formatos definidos.

El sistema deberá operar de forma multiplataforma y ser integrable con aplicaciones cliente como .NET MAUI.

---

# 10. Trazabilidad

| NB | Necesidad de negocio | Funcionalidad | Caso de uso | Versión vigente | RN asociadas | Estado |
| --- | --- | --- | --- | --- | --- | --- |
| NB-01 | Interpretar plantillas DSL | Interpretación y parsing | [CU-01](casos-de-uso/CU-01-interpretar-plantilla-dsl_v1.0.md) | v1.0 | RN-01, RN-02 | Activo |
| NB-01 | Resolver datos del documento | Resolución de datos | [CU-02](casos-de-uso/CU-02-resolver-datos-documento_v1.0.md) | v1.0 | — | Activo |
| NB-01 | Construir modelo interno | Modelo abstracto | [CU-03](casos-de-uso/CU-03-construir-modelo-interno_v1.0.md) | v1.0 | RN-03, RN-05 | Activo |
| NB-01 | Ejecutar motor de layout | Layout engine | [CU-04](casos-de-uso/CU-04-ejecutar-motor-de-layout_v1.0.md) | v1.0 | — | Activo |
| NB-01 | Generar representación abstracta | Representación intermedia | [CU-05](casos-de-uso/CU-05-generar-representacion-abstracta_v1.0.md) | v1.0 | RN-05 | Activo |
| NB-04 | Renderizar ESC/POS | Renderizado térmico | [CU-06](casos-de-uso/CU-06-renderizar-escpos_v2.0.md) | **v2.0** | RN-03, RN-06 | Activo |
| NB-04 | Renderizar texto plano | Debug rendering | [CU-07](casos-de-uso/CU-07-renderizar-texto-plano-debug_v1.0.md) | v1.0 | — | Activo |
| NB-04 | Renderizar vista previa UI | Preview en MAUI | [CU-08](casos-de-uso/CU-08-renderizar-vista-previa-ui_v1.0.md) | v1.0 | — | Activo |
| NB-04 | Renderizar PDF | Salida paginada | [CU-09](casos-de-uso/CU-09-renderizar-pdf_v2.0.md) | **v2.0** | RN-03 | Futuro |
| NB-04 | Enviar a impresora Bluetooth | Impresión BT | [CU-10](casos-de-uso/CU-10-enviar-impresora-bluetooth_v1.0.md) | v1.0 | — | Activo |
| NB-04 | Seleccionar perfil de impresora | Perfil de dispositivo | [CU-11](casos-de-uso/CU-11-seleccionar-perfil-impresora_v1.0.md) | v1.0 | RN-06 | Activo |
| NB-04 | Adaptar documento al perfil | Adaptación de salida | [CU-12](casos-de-uso/CU-12-adaptar-documento-perfil_v1.0.md) | v1.0 | RN-06 | Activo |
| NB-02 | Cargar plantilla DSL | Carga de plantilla | [CU-13](casos-de-uso/CU-13-cargar-plantilla-dsl_v1.0.md) | v1.0 | RN-01 | Activo |
| NB-02 | Validar plantilla DSL | Validación DSL | [CU-14](casos-de-uso/CU-14-validar-plantilla-dsl_v1.0.md) | v1.0 | RN-01, RN-02 | Activo |
| NB-01 | Resolver referencias de datos | Resolución de datos | [CU-15](casos-de-uso/CU-15-resolver-referencias-datos_v1.0.md) | v1.0 | — | Activo |
| NB-01 | Ejecutar iteraciones/listas | Iteraciones | [CU-16](casos-de-uso/CU-16-ejecutar-iteraciones-listas_v1.0.md) | v1.0 | — | Activo |
| NB-01 | Evaluar condiciones | Evaluación condicional | [CU-17](casos-de-uso/CU-17-evaluar-condiciones_v1.0.md) | v1.0 | — | Activo |
| NB-01 | Cargar datos del documento | Carga de datos | [CU-18](casos-de-uso/CU-18-cargar-datos-documento_v1.0.md) | v1.0 | — | Activo |
| NB-01 | Validar estructura de datos | Validación de datos | [CU-19](casos-de-uso/CU-19-validar-estructura-datos_v1.0.md) | v1.0 | — | Activo |
| NB-01 | Mapear datos a plantilla | Mapeo de datos | [CU-20](casos-de-uso/CU-20-mapear-datos-plantilla_v1.0.md) | v1.0 | — | Activo |
| NB-04 | Cargar perfil de impresora | Perfil de dispositivo | [CU-21](casos-de-uso/CU-21-cargar-perfil-impresora_v2.0.md) | **v2.0** | RN-06 | Activo |
| NB-04 | Validar perfil de dispositivo | Validación de perfil | [CU-22](casos-de-uso/CU-22-validar-perfil-dispositivo_v1.0.md) | v1.0 | RN-06 | Activo |
| NB-04 | Aplicar restricciones dispositivo | Restricciones | [CU-23](casos-de-uso/CU-23-aplicar-restricciones-dispositivo_v1.0.md) | v1.0 | RN-06 | Activo |
| NB-03 | Proveer plantilla al motor | Integración API | [CU-24](casos-de-uso/CU-24-proveer-plantilla-motor_v2.0.md) | **v2.0** | — | Activo |
| NB-03 | Proveer datos al motor | Integración API | [CU-25](casos-de-uso/CU-25-proveer-datos-motor_v2.0.md) | **v2.0** | — | Activo |
| NB-03 | Proveer perfil al motor | Integración API | [CU-26](casos-de-uso/CU-26-proveer-perfil-motor_v2.0.md) | **v2.0** | — | Activo |
| NB-05 | Integrar motor con MAUI | Integración .NET MAUI | [CU-27](casos-de-uso/CU-27-integrar-motor-maui_v1.0.md) | v1.0 | — | Activo |
| NB-05 | Configurar motor DSL | Configuración DSL | [CU-28](casos-de-uso/CU-28-configurar-motor-dsl_v1.0.md) | v1.0 | — | Activo |
| NB-06 | Extender renderizadores | Extensibilidad | [CU-29](casos-de-uso/CU-29-extender-motor-renderizadores_v1.0.md) | v1.0 | — | Activo |
| NB-06 | Manejar errores de plantilla | Validación y errores | [CU-30](casos-de-uso/CU-30-manejar-errores-plantilla_v1.0.md) | v1.0 | — | Activo |
| NB-06 | Manejar errores de datos | Diagnóstico de datos | [CU-31](casos-de-uso/CU-31-manejar-errores-datos_v1.0.md) | v1.0 | — | Activo |
| NB-06 | Manejar errores de impresión | Recuperación ante fallo | [CU-32](casos-de-uso/CU-32-manejar-errores-impresion_v1.0.md) | v1.0 | — | Activo |

> **Nota:** Los CU marcados en **negrita** poseen versión v2.0 que reemplaza a la v1.0. Los archivos v1.0 se conservan como referencia histórica.

---

**Fin del documento**
