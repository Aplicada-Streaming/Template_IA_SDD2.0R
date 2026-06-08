# Plan de Pruebas — Motor DSL de Renderizado

**Archivo:** plan-pruebas_v1.0.md
**Versión:** 1.0
**Fecha:** 2026-03-28
**Autor:** Equipo Técnico
**Estado:** Activo

---

## 1. Propósito

Definir la estrategia y planificación de pruebas del Motor DSL de Generación de Documentos.

Este plan establece los tipos de prueba, criterios de entrada y salida, herramientas, responsabilidades y calendario necesarios para garantizar la calidad del motor a lo largo de su ciclo de desarrollo.

---

## 2. Alcance

El plan de pruebas cubre los siguientes componentes del motor:

* **Parser** — Conversión de plantillas DSL (JSON) a AST
* **AST** — Modelo de nodos del documento abstracto
* **DataResolver** — Resolución de datos dinámicos y bindings
* **Evaluator** — Evaluación de expresiones condicionales e iterativas
* **LayoutEngine** — Cálculo de layout adaptativo
* **Renderers** — Renderizadores a ESC/POS, texto plano, vista previa UI y PDF
* **Extensibilidad** — Puntos de extensión: nodos custom, renderizadores y plugins

---

## 3. Tipos de Prueba

### 3.1 Pruebas Unitarias

Validación de cada componente de forma aislada:

* Parser: conversión de JSON a nodos AST
* Nodos AST: construcción, propiedades y validación
* Evaluator: evaluación de expresiones y condicionales
* LayoutEngine: cálculo de anchos, alineación y distribución
* Renderizadores: generación de salida por formato

### 3.2 Pruebas de Integración

Validación del pipeline completo:

* DSL → Parse → Validate → Evaluate → Render → Output
* Integración entre DataResolver y Evaluator
* Integración entre LayoutEngine y Renderers
* Flujo completo con perfiles de dispositivo

### 3.3 Pruebas de Snapshot / Golden

Comparación de salidas renderizadas contra archivos de referencia:

* Se almacenan archivos golden con la salida esperada
* Cada ejecución compara la salida actual contra el golden
* Los cambios intencionales requieren actualización explícita del golden
* Aplica a todos los renderizadores

### 3.4 Pruebas de Regresión

Suite automatizada ejecutada en cada PR:

* Incluye todas las pruebas unitarias e integración
* Se ejecuta automáticamente en el pipeline de CI
* Bloquea merge si alguna prueba falla
* Garantiza que cambios nuevos no rompen funcionalidad existente

---

## 4. Criterios de Entrada

Para que las pruebas puedan ejecutarse, deben cumplirse:

* El código compila sin errores
* El Definition of Ready (DoR) del ítem está cumplido
* El ambiente de test está disponible y configurado
* Las dependencias del componente bajo prueba están resueltas

---

## 5. Criterios de Salida

Las pruebas se consideran exitosas cuando:

* 100% de los tests pasan
* Cobertura de código ≥ 70%
* 0 defectos críticos abiertos
* Todos los snapshots/golden files están actualizados
* No hay regresiones respecto a la versión anterior

---

## 6. Ambientes de Prueba

| Ambiente | Descripción |
|----------|-------------|
| Local dev | Máquina del desarrollador, ejecución manual o por IDE |
| CI pipeline | GitHub Actions, ejecución automatizada en cada PR y merge |

---

## 7. Herramientas

| Herramienta | Propósito |
|-------------|-----------|
| xUnit | Framework de pruebas unitarias e integración |
| FluentAssertions | Aserciones legibles y expresivas |
| Verify | Pruebas de snapshot / golden file |
| Coverlet | Medición de cobertura de código |

---

## 8. Responsabilidades

| Rol | Responsabilidad |
|-----|----------------|
| Desarrollador | Crear y mantener pruebas unitarias de su componente |
| Equipo | Diseñar y mantener pruebas de integración |
| CI (automatizado) | Ejecutar suite de regresión completa en cada PR |

---

## 9. Calendario

| Tipo de prueba | Frecuencia |
|----------------|------------|
| Tests unitarios | Durante cada sprint, con cada cambio de código |
| Tests de integración | Al final de cada sprint |
| Regresión completa | Pre-release y en cada PR (automatizada) |

---

## 10. Riesgos

| Riesgo | Mitigación |
|--------|------------|
| Hardware específico no disponible en CI | Uso de mocks y perfiles simulados para pruebas de renderizado |
| Variaciones entre plataformas | Ejecución de tests en múltiples OS en CI (Windows, Linux) |
| Snapshots frágiles ante cambios de formato | Revisión periódica de golden files, proceso claro de actualización |

---

## 11. Trazabilidad

Este plan de pruebas se complementa con los siguientes documentos:

* [estrategia-calidad-motor_v1.0.md](estrategia-calidad-motor_v1.0.md) — Estrategia general de calidad
* [estrategia-testing-motor_v1.0.md](estrategia-testing-motor_v1.0.md) — Estrategia de testing específica del motor
* [matriz-cobertura-pruebas_v1.0.md](matriz-cobertura-pruebas_v1.0.md) — Matriz de cobertura de pruebas por componente
* [casos-prueba-referenciales_v1.0.md](casos-prueba-referenciales_v1.0.md) — Casos de prueba referenciales

---

## 12. Control de Cambios

| Versión | Fecha      | Descripción     |
| ------- | ---------- | --------------- |
| 1.0     | 2026-03-28 | Versión inicial |

---

**Fin del documento**
