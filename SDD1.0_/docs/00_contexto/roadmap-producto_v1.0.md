# Roadmap de Producto

**Proyecto:** Motor DSL de Generación de Documentos
**Documento:** roadmap-producto_v1.0.md
**Versión:** 1.0
**Estado:** Aprobado
**Fecha:** 2026-03-28
**Autor:** Equipo Técnico

---

# 1. Propósito

Conectar la visión del producto con el backlog técnico y el plan de sprints. Este roadmap proporciona una vista de alto nivel de las fases de desarrollo, sus objetivos y entregables esperados.

---

# 2. Fases del Producto

## Fase 1 — MVP (Sprints 01–04)

- **Objetivo:** Motor funcional con pipeline completo DSL → AST → Render ESC/POS
- **Épicas:**
  - ÉPICA 1 — Fundaciones del proyecto
  - ÉPICA 2 — Modelo AST
  - ÉPICA 3 — Parser DSL
  - ÉPICA 4 — Data Resolution
  - ÉPICA 5 — Evaluación de expresiones
  - ÉPICA 6 — Motor de Layout
  - ÉPICA 7 parcial — Render ESC/POS
- **Entregable:** Librería MotorDsl.Core funcional con rendering ESC/POS
- **Release target:** v1.0.0
- **Historias:** US-01 a US-16

## Fase 2 — Expansión (Sprints 05–06)

- **Objetivo:** Multi-formato, extensibilidad, integración MAUI
- **Épicas:**
  - ÉPICA 7 completa — Todos los renderers (texto plano, vista previa UI)
  - ÉPICA 8 — Orquestación e integración
  - ÉPICA 9 — Extensibilidad
- **Entregable:** Soporte multi-formato, API de extensibilidad, integración .NET MAUI
- **Release target:** v1.1.0 / v1.2.0
- **Historias:** US-17 a US-25

## Fase 3 — Madurez (Sprints 07–08)

- **Objetivo:** PDF, Bluetooth, performance, documentación completa
- **Épicas:**
  - ÉPICA 10 — Testing y calidad
  - ÉPICA 11 — Performance y optimización
- **Entregable:** Render PDF, impresión Bluetooth, ejemplos, documentación de integración
- **Release target:** v1.3.0
- **Historias:** US-26 a US-31

## Futuro — Evolución

- **Objetivo:** Breaking changes, DSL v2, editor visual
- **Épicas:** Por definir según feedback de usuarios y stakeholders
- **Entregable:** Nueva versión mayor del motor con capacidades avanzadas
- **Release target:** v2.0.0

---

# 3. Matriz de Trazabilidad Fase → Épica → Sprint

| Fase | Épicas | Sprints | Release |
| ---- | ------ | ------- | ------- |
| Fase 1 — MVP | ÉPICA 1, 2, 3, 4, 5, 6, 7 (parcial) | Sprint 01–04 | v1.0.0 |
| Fase 2 — Expansión | ÉPICA 7 (completa), 8, 9 | Sprint 05–06 | v1.1.0 / v1.2.0 |
| Fase 3 — Madurez | ÉPICA 10, 11 | Sprint 07–08 | v1.3.0 |
| Futuro — Evolución | Por definir | Por definir | v2.0.0 |

---

# 4. Dependencias entre Fases

- **Fase 2 depende de Fase 1:** El pipeline core (parser, AST, render) debe estar funcional antes de agregar multi-formato y extensibilidad.
- **Fase 3 depende de Fase 2:** La orquestación e integración MAUI deben estar estables antes de agregar PDF, Bluetooth y optimización.
- **Futuro depende de Fase 3:** Las decisiones de DSL v2 se informan por el feedback obtenido durante las fases anteriores.

---

# 5. Criterios de Transición entre Fases

Para avanzar de una fase a la siguiente:

- [ ] Todas las historias **Must Have** de la fase cumplen el [Definition of Done](../08_calidad_y_pruebas/definition-of-done_v1.0.md).
- [ ] Pruebas de regresión ejecutadas sin fallos críticos.
- [ ] Release etiquetado según SemVer.
- [ ] Sprint Review aprobado por el Product Owner.
- [ ] Deuda técnica de la fase documentada y priorizada.

---

# 6. Control de Cambios

| Versión | Fecha      | Descripción     |
| ------- | ---------- | --------------- |
| 1.0     | 2026-03-28 | Versión inicial |

---

**Fin del documento**
