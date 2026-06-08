# Acuerdo de Equipo (Team Charter / Working Agreements)

**Proyecto:** Motor DSL de Generación de Documentos
**Documento:** acuerdo-equipo_v1.0.md
**Versión:** 1.1
**Estado:** Borrador
**Fecha:** 2026-04-25
**Autor:** Equipo Técnico

---

# 1. Propósito

Establecer los acuerdos de trabajo, roles y compromisos del equipo. Este documento sirve como referencia compartida para las normas y prácticas que el equipo se compromete a seguir.

---

# 2. Equipo y Roles Scrum

| Rol | Persona | Notas |
| --- | ------- | ----- |
| Product Owner | [Pendiente de asignar] | Responsable del backlog y priorización |
| Scrum Master | [Pendiente de asignar] | Facilitador del proceso Scrum |
| Equipo de Desarrollo | [Pendiente de completar] | Listar nombres y especialización |

### Miembros del equipo de desarrollo

| Nombre | Especialización | Notas |
| ------ | --------------- | ----- |
| [Nombre 1] | [Área de expertise] | - |
| [Nombre 2] | [Área de expertise] | - |
| [Nombre 3] | [Área de expertise] | - |

---

# 3. Cadencia de Ceremonias

| Ceremonia | Cuándo | Duración | Notas |
| --------- | ------ | -------- | ----- |
| Sprint Planning | Día 1 de cada sprint | 2 horas | Todo el equipo Scrum |
| Daily Scrum | Diario | 15 minutos | Formato async o sincrónico [definir] |
| Sprint Review | Último día del sprint | 1 hora | Demo a stakeholders |
| Sprint Retrospective | Después del Review | 1 hora | Solo el equipo Scrum |
| Backlog Refinement | Semanal, mitad del sprint | 1 hora | Preparación de ítems futuros |

---

# 4. Acuerdos de Trabajo

## 4.1 Branching Strategy

- **Modelo:** GitHub Flow estricto (decisión D3).
- **Rama principal:** `main` — siempre liberable, protegida (no se permite push directo).
- **Ramas de trabajo:** `feature/<scope>-<descripcion-corta>` creadas desde `main`.
- **Integración:** PR obligatorio hacia `main` con **squash merge** y al menos 1 aprobación.
- **Sin ramas persistentes adicionales:** no se utilizan `develop`, `release/*` ni `hotfix/*` como ramas de larga duración.
- **Hotfixes urgentes:** se crea una rama efímera `feature/hotfix-<scope>` desde `main` con PR fast-track (review acelerada); no es una rama persistente.
- **Convención de nombres:** `feature/US-XX-descripcion-corta` (o `feature/hotfix-<scope>` para correcciones urgentes).
- **Detalles operativos:** ver [`09_devops/estrategia-versionado_v1.0.md` §7](../09_devops/estrategia-versionado_v1.0.md) como autoridad para reglas detalladas de branching y merge.

## 4.2 Code Review

- Mínimo **1 aprobación** antes de merge a `main`.
- Las revisiones se realizan en un máximo de 24 horas hábiles.
- El autor del PR es responsable de resolver los comentarios.

## 4.3 Comunicación

- **Canal principal:** [Canal Slack/Teams — pendiente de definir]
- **Bloqueos:** Comunicar inmediatamente al equipo, no esperar al Daily.
- **Decisiones técnicas:** Documentar en ADRs cuando sean significativas.

## 4.4 Horario Core

- **Horario core:** [Pendiente de definir]
- Durante el horario core, todos los miembros están disponibles para consultas.

## 4.5 Documentación

- Todo cambio relevante se refleja en `docs/`.
- La documentación se actualiza como parte del Definition of Done.

## 4.6 Convenciones de Commits

- Se adopta **Conventional Commits** (decisión D5) para mensajes de commits y títulos de PR.
- Detalles y formato: ver [`09_devops/estrategia-versionado_v1.0.md` §8](../09_devops/estrategia-versionado_v1.0.md).

---

# 5. Definition of Done

Ver [definition-of-done_v1.0.md](../08_calidad_y_pruebas/definition-of-done_v1.0.md)

---

# 6. Definition of Ready

Ver [definition-of-ready_v1.0.md](../06_backlog-tecnico/definition-of-ready_v1.0.md)

---

# 7. Herramientas

| Categoría | Herramienta | Notas |
| --------- | ----------- | ----- |
| IDE | Visual Studio / VS Code | Según preferencia del desarrollador |
| Repositorio | Git | Hosted en GitHub |
| CI/CD | GitHub Actions | Pipelines de build, test y deploy |
| Tracking | [Pendiente de definir] | GitHub Projects, Jira u otra |
| Comunicación | [Pendiente de definir] | Slack, Teams u otra |

---

# 8. Control de Cambios

| Versión | Fecha      | Autor                | Descripción                                                                                                                                                            |
| ------- | ---------- | -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.0     | 2026-03-28 | Equipo Técnico       | Versión inicial                                                                                                                                                        |
| 1.1     | 2026-04-25 | Tech Lead / Equipo   | Alineación de branching strategy con GitHub Flow. Eliminadas ramas `develop` y `hotfix/*` como ramas persistentes; alineado con `09_devops/estrategia-versionado_v1.0.md` §7. |

---

**Fin del documento**
