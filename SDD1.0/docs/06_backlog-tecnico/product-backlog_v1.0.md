# Product Backlog

**Proyecto:** Motor DSL de Generación de Documentos
**Documento:** product-backlog_v1.0.md
**Versión:** 1.0
**Estado:** Aprobado
**Fecha:** 2026-03-28
**Autor:** Equipo Técnico

---

# 1. Propósito

Backlog de producto orientado al valor de negocio. Este documento centraliza todas las historias de usuario del proyecto, clasificadas por prioridad y con trazabilidad a los casos de uso y sprints correspondientes.

---

# 2. Formato

Las User Stories siguen el formato estándar:

> **Como** [rol], **quiero** [acción], **para** [beneficio]

Cada historia incluye su clasificación MoSCoW, sprint asignado, trazabilidad a casos de uso y estado actual.

---

# 3. Clasificación MoSCoW

- **Must Have:** Core pipeline (parsing, AST, rendering básico) — funcionalidades imprescindibles para el MVP.
- **Should Have:** Multi-formato, extensibilidad, perfiles de impresora — funcionalidades importantes pero no bloqueantes.
- **Could Have:** Optimización de performance, métricas avanzadas — deseables si hay capacidad.
- **Won't Have (v1.0):** Editor visual, scripting avanzado, integración fiscal — fuera de alcance para esta versión.

---

# 4. Backlog de Producto

| ID    | Historia de Usuario | Prioridad MoSCoW | Sprint | Trazabilidad CU | Estado |
| ----- | ------------------- | ----------------- | ------ | ---------------- | ------ |
| US-01 | Como desarrollador, quiero inicializar la estructura base del motor, para tener una base sólida de desarrollo. | Must | Sprint 01 | CU-28 | Done |
| US-02 | Como desarrollador, quiero definir el modelo AST base, para representar documentos de forma estructurada. | Must | Sprint 01 | CU-03 | Done |
| US-03 | Como desarrollador, quiero un parser DSL básico, para convertir plantillas JSON en un AST. | Must | Sprint 01 | CU-01 | Done |
| US-04 | Como desarrollador, quiero procesar una plantilla DSL simple, para generar un AST válido. | Must | Sprint 01 | CU-01, CU-03 | Done |
| US-05 | Como desarrollador, quiero resolver datos dinámicos en el AST, para vincular datos externos a plantillas. | Must | Sprint 02 | CU-02, CU-15 | Done |
| US-06 | Como desarrollador, quiero evaluar condiciones en el DSL, para generar documentos condicionales. | Must | Sprint 02 | CU-17 | Done |
| US-07 | Como desarrollador, quiero ejecutar iteraciones sobre listas, para generar contenido repetitivo. | Must | Sprint 02 | CU-16 | Done |
| US-08 | Como desarrollador, quiero un motor de layout básico, para posicionar elementos en el documento. | Must | Sprint 02 | CU-04 | Done |
| US-09 | Como desarrollador, quiero renderizar a ESC/POS, para imprimir en impresoras térmicas. | Must | Sprint 03 | CU-06 | Done |
| US-10 | Como desarrollador, quiero renderizar a texto plano, para debugging y logs. | Should | Sprint 03 | CU-07 | Done |
| US-11 | Como desarrollador, quiero renderizar vista previa UI, para previsualizar documentos. | Should | Sprint 03 | CU-08 | Done |
| US-12 | Como desarrollador, quiero soporte para perfiles de impresora, para adaptar salida a hardware específico. | Must | Sprint 03 | CU-11, CU-21 | Done |
| US-13 | Como desarrollador, quiero validar plantillas DSL, para detectar errores antes del procesamiento. | Must | Sprint 04 | CU-14 | Done |
| US-14 | Como desarrollador, quiero validar estructura de datos, para garantizar consistencia. | Must | Sprint 04 | CU-19 | Done |
| US-15 | Como desarrollador, quiero mapear datos a plantilla, para automatizar la vinculación. | Must | Sprint 04 | CU-20 | Done |
| US-16 | Como desarrollador, quiero adaptar documentos al perfil del dispositivo, para respetar capacidades del hardware. | Should | Sprint 04 | CU-12, CU-23 | Done |
| US-17 | Como desarrollador, quiero cargar plantillas desde el motor, para proveer contenido dinámico. | Should | Sprint 05 | CU-24 | Done |
| US-18 | Como desarrollador, quiero cargar datos desde el motor, para alimentar el pipeline. | Should | Sprint 05 | CU-25 | Done |
| US-19 | Como desarrollador, quiero cargar perfiles desde el motor, para configurar dispositivos. | Should | Sprint 05 | CU-26 | Done |
| US-20 | Como desarrollador, quiero integrar el motor con .NET MAUI, para usarlo en apps móviles. | Must | Sprint 05 | CU-27 | Done |
| US-21 | Como desarrollador, quiero configurar el motor via DI, para integración limpia. | Must | Sprint 05 | CU-28 | Done |
| US-22 | Como desarrollador, quiero extender el motor con nuevos renderizadores, para soportar formatos adicionales. | Should | Sprint 06 | CU-29 | Done |
| US-23 | Como desarrollador, quiero manejar errores de plantilla, para diagnósticos claros. | Must | Sprint 06 | CU-30 | Done |
| US-24 | Como desarrollador, quiero manejar errores de datos, para debugging efectivo. | Must | Sprint 06 | CU-31 | Done |
| US-25 | Como desarrollador, quiero manejar errores de impresión, para recuperación ante fallos. | Must | Sprint 06 | CU-32 | Done |
| US-26 | Como desarrollador, quiero enviar documentos a impresora Bluetooth, para impresión móvil. | Must | Sprint 07 | CU-10 | Done |
| US-27 | Como desarrollador, quiero renderizar a PDF, para generar documentos digitales. | Could | Sprint 07 | CU-09 | Done |
| US-28 | Como desarrollador, quiero soporte de imágenes térmicas, para logos y firmas. | Should | Sprint 07 | - | Done |
| US-29 | Como desarrollador, quiero perfiles de impresoras reales, para configuración productiva. | Should | Sprint 08 | - | Done |
| US-30 | Como desarrollador, quiero documentación de integración, para facilitar onboarding. | Should | Sprint 08 | - | Done |
| US-31 | Como desarrollador, quiero ejemplos de aplicación completos, para referencia práctica. | Should | Sprint 08 | - | Done |

---

# 5. Trazabilidad

- **Descomposición técnica:** Ver [backlog-tecnico_v1.0.md](backlog-tecnico_v1.0.md) para la descomposición en tareas técnicas (BT-XXX).
- **Ejecución por sprint:** Ver los documentos de plan de sprint en [07_plan-sprint](../07_plan-sprint/) para el detalle de ejecución.
- **Casos de uso:** Ver [casos-de-uso_v1.0.md](../02_especificacion_funcional/casos-de-uso_v1.0.md) para la especificación funcional completa.

---

# 6. Control de Cambios

| Versión | Fecha      | Descripción     |
| ------- | ---------- | --------------- |
| 1.0     | 2026-03-28 | Versión inicial — consolidación de historias de usuario de todos los sprints |

---

**Fin del documento**
