
# Necesidades de Negocio

**Proyecto:** Motor de Documentos e Impresión basado en DSL
**Documento:** NB-02 Estandarización.v1.0.md
**Versión:** 1.0
**Estado:** Propuesto
**Fecha:** 2026-03-28
**Autor:** Área de Arquitectura / Análisis Técnico

## NB-02 Estandarizar la generación de documentos

La organización necesita establecer un modelo estándar para la definición de documentos que permita unificar la forma en que se representan, procesan y renderizan los distintos tipos de comprobantes, tickets y documentos en los sistemas.

Esta estandarización debe basarse en un DSL común que permita describir la estructura y el contenido de los documentos de manera consistente, independientemente de la aplicación o del dispositivo de salida.

El objetivo es evitar implementaciones ad-hoc y garantizar que todos los sistemas utilicen un mismo enfoque para la generación de documentos, facilitando la reutilización, el mantenimiento y la evolución del ecosistema.

**Ejemplo:**
Un mismo tipo de ticket debe definirse mediante una plantilla estándar reutilizable en distintas aplicaciones sin necesidad de redefinir su estructura.

**Impacto:**
DSL de plantillas

---

### Problema Específico

No existe un formato estándar para definir documentos. Cada aplicación implementa su propia lógica de generación, resultando en:
- Formatos incompatibles entre sistemas
- Duplicación de esfuerzo al definir estructuras similares
- Imposibilidad de reutilizar plantillas entre aplicaciones

### Criterios de Éxito

| Criterio | Métrica | Target |
|----------|---------|--------|
| Cobertura DSL | Tipos de documento representables con DSL estándar | ≥ 90% |
| Adopción | Aplicaciones usando DSL común | 100% de nuevas apps |
| Consistencia | Documentos generados con mismo formato entre apps | 100% |

### Stakeholders

- Equipo de Arquitectura (propietario)
- Equipo Funcional (validador de formatos)

### Trazabilidad a Casos de Uso

| CU | Descripción | Relación |
|----|-------------|----------|
| [CU-13](../../02_especificacion_funcional/casos-de-uso/CU-13-cargar-plantilla-dsl_v1.0.md) | Cargar plantilla DSL | Formato estándar de carga |
| [CU-14](../../02_especificacion_funcional/casos-de-uso/CU-14-validar-plantilla-dsl_v1.0.md) | Validar plantilla DSL | Validación contra esquema estándar |

### Dependencias

- Depende de NB-01 (desacople permite estandarización)

