
# Necesidades de Negocio

**Proyecto:** Motor de Documentos e Impresión basado en DSL
**Documento:** NB-04-Multi-dispositivo.v1.0.md
**Versión:** 1.0
**Estado:** Propuesto
**Fecha:** 2026-03-28
**Autor:** Área de Arquitectura / Análisis Técnico

## NB-04 Adaptarse a múltiples dispositivos de salida

La organización necesita que los documentos puedan generarse y renderizarse correctamente en distintos dispositivos y formatos de salida, sin requerir cambios en los datos ni en las plantillas.

El sistema debe abstraer las particularidades del hardware mediante perfiles de dispositivo, permitiendo que un mismo documento pueda adaptarse automáticamente a diferentes capacidades, resoluciones y protocolos de impresión.

Este enfoque garantiza portabilidad y extensibilidad del sistema hacia nuevos dispositivos.

**Ejemplo:**
Un mismo documento debe poder imprimirse en una impresora térmica ESC/POS, visualizarse en pantalla o exportarse a PDF sin redefinir su estructura.

**Impacto:**
Renderizadores

---

### Problema Específico

Cada dispositivo de salida requiere implementación específica de renderizado:
- Impresoras térmicas ESC/POS: comandos binarios
- Pantalla: componentes UI
- PDF: formato de documento portable

Sin abstracción, cada nuevo dispositivo multiplica el esfuerzo de desarrollo.

### Criterios de Éxito

| Criterio | Métrica | Target |
|----------|---------|--------|
| Renderizadores soportados | Formatos de salida disponibles | ≥ 3 (ESC/POS, UI, texto) |
| Extensibilidad | Tiempo para agregar nuevo renderizador | < 1 día |
| Fidelidad | Documento renderizado igual en todos los dispositivos | ≥ 95% |

### Stakeholders

- Equipo de Arquitectura (propietario)
- Equipo de Desarrollo (implementador)

### Trazabilidad a Casos de Uso

| CU | Descripción | Relación |
|----|-------------|----------|
| [CU-06](../../02_especificacion_funcional/casos-de-uso/CU-06-renderizar-escpos_v2.0.md) | Renderizar ESC/POS | Salida a impresora térmica |
| [CU-07](../../02_especificacion_funcional/casos-de-uso/CU-07-renderizar-texto-plano-debug_v1.0.md) | Renderizar texto plano | Salida debug |
| [CU-08](../../02_especificacion_funcional/casos-de-uso/CU-08-renderizar-vista-previa-ui_v1.0.md) | Renderizar vista previa UI | Salida pantalla |
| [CU-09](../../02_especificacion_funcional/casos-de-uso/CU-09-renderizar-pdf_v2.0.md) | Renderizar PDF | Salida PDF |
| [CU-29](../../02_especificacion_funcional/casos-de-uso/CU-29-extender-motor-renderizadores_v1.0.md) | Extender motor renderizadores | Extensibilidad |

### Dependencias

- Depende de NB-01 (desacople datos/diseño/dispositivo)