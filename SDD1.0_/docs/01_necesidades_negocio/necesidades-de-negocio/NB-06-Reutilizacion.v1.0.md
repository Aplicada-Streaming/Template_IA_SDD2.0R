
# Necesidades de Negocio

**Proyecto:** Motor de Documentos e Impresión basado en DSL
**Documento:** NB-06-Reutilizacion.v1.0.md
**Versión:** 1.0
**Estado:** Propuesto
**Fecha:** 2026-03-28
**Autor:** Área de Arquitectura / Análisis Técnico

## NB-06 Habilitar reutilización entre proyectos

La organización necesita contar con una solución centralizada que permita reutilizar la lógica de generación y renderizado de documentos en múltiples aplicaciones y contextos, evitando desarrollos aislados.

El motor debe implementarse como una librería desacoplada, con interfaces claras y extensibles, que pueda integrarse fácilmente en distintos proyectos sin depender de una aplicación específica.

Este enfoque favorece la consistencia técnica, reduce la duplicación de esfuerzos y facilita la evolución del sistema a lo largo del tiempo.

**Ejemplo:**
Una misma librería de generación de documentos utilizada tanto en una aplicación móvil .NET MAUI como en un servicio backend.

**Impacto:**
Integración en múltiples aplicaciones

---

### Problema Específico

No existe una librería centralizada de generación de documentos:
- Cada aplicación tiene su propia implementación
- No hay contratos ni interfaces comunes
- La lógica de negocio de documentos está embebida en cada app

### Criterios de Éxito

| Criterio | Métrica | Target |
|----------|---------|--------|
| Integración | Aplicaciones usando la librería compartida | ≥ 2 apps |
| Abstracción | APIs públicas bien definidas | 100% interfaces documentadas |
| Independencia | Librería sin dependencia de app específica | 0 dependencias app-specific |

### Stakeholders

- Equipo de Arquitectura (propietario)
- Equipos de Desarrollo de productos (consumidores)

### Trazabilidad a Casos de Uso

| CU | Descripción | Relación |
|----|-------------|----------|
| [CU-27](../../02_especificacion_funcional/casos-de-uso/CU-27-integrar-motor-maui_v1.0.md) | Integrar motor en MAUI | Primer caso de integración |
| [CU-28](../../02_especificacion_funcional/casos-de-uso/CU-28-configurar-motor-dsl_v1.0.md) | Configurar motor DSL | Configuración cross-app |
| [CU-29](../../02_especificacion_funcional/casos-de-uso/CU-29-extender-motor-renderizadores_v1.0.md) | Extender motor | Extensibilidad cross-app |

### Dependencias

- Depende de NB-01 (desacople)