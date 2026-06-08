
# Necesidades de Negocio

**Proyecto:** Motor de Documentos e Impresión basado en DSL
**Documento:** NB-03-Cambios-sin-codigo.v1.0.md
**Versión:** 1.0
**Estado:** Propuesto
**Fecha:** 2026-03-28
**Autor:** Área de Arquitectura / Análisis Técnico

## NB-03 Permitir cambios de diseño sin modificar código

La organización necesita poder modificar la estructura y el diseño de los documentos sin requerir cambios en el código de las aplicaciones ni nuevos despliegues.

Para ello, el sistema debe basarse en plantillas DSL externas y versionables que puedan ser actualizadas de forma independiente, permitiendo adaptar rápidamente los documentos a nuevos requerimientos de negocio.

Este enfoque reduce el impacto operativo de los cambios y mejora la capacidad de respuesta ante necesidades de evolución.

**Ejemplo:**
Agregar un nuevo campo o modificar el formato de un ticket actualizando únicamente la plantilla DSL, sin recompilar la aplicación.

**Impacto:**
Plantillas dinámicas

---

### Problema Específico

Cada cambio en el diseño de un documento requiere:
- Modificación de código fuente
- Recompilación y redespliegue de la aplicación
- Ciclo de testing completo

Esto genera alto costo operativo y tiempos de respuesta lentos ante cambios de negocio.

### Criterios de Éxito

| Criterio | Métrica | Target |
|----------|---------|--------|
| Cambios sin deploy | Documentos modificables sin recompilar | ≥ 90% |
| Tiempo de cambio | Tiempo promedio para modificar un documento | < 30 min |
| Versionado | Plantillas versionables independientemente | 100% |

### Stakeholders

- Equipo Funcional (propietario)
- Equipo de Operaciones (beneficiario)

### Trazabilidad a Casos de Uso

| CU | Descripción | Relación |
|----|-------------|----------|
| [CU-13](../../02_especificacion_funcional/casos-de-uso/CU-13-cargar-plantilla-dsl_v1.0.md) | Cargar plantilla DSL | Plantillas externas cargables |
| [CU-24](../../02_especificacion_funcional/casos-de-uso/CU-24-proveer-plantilla-motor_v2.0.md) | Proveer plantilla al motor | Provisión dinámica sin código |
| [CU-25](../../02_especificacion_funcional/casos-de-uso/CU-25-proveer-datos-motor_v2.0.md) | Proveer datos al motor | Datos dinámicos sin código |

### Dependencias

- Depende de NB-01 (desacople) y NB-02 (estandarización)