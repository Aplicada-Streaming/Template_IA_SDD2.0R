# Definition of Done (DoD)

**Proyecto:** Motor DSL de Generación de Documentos
**Documento:** definition-of-done_v1.0.md
**Versión:** 1.0
**Estado:** Aprobado
**Fecha:** 2026-03-28
**Autor:** Equipo Técnico

---

# 1. Propósito

Establecer los criterios unificados que determinan cuándo un ítem del backlog (historia de usuario, tarea técnica o bug fix) se considera **terminado**. Este documento es la fuente única de verdad para el DoD del proyecto.

---

# 2. Criterios de Terminado

Una funcionalidad, historia o tarea se considera **Done** cuando cumple TODOS los siguientes criterios:

## 2.1 Código

- [ ] El código compila correctamente sin warnings
- [ ] El código sigue las convenciones del proyecto (naming, estructura)
- [ ] No existen dependencias cruzadas indebidas entre módulos
- [ ] El código ha sido revisado por al menos un miembro del equipo (PR aprobado)

## 2.2 Testing

- [ ] Tiene pruebas unitarias que cubren los escenarios principales
- [ ] Tiene pruebas de integración (cuando aplica)
- [ ] Cobertura de código ≥ 70%
- [ ] Todos los tests del pipeline CI pasan (verde)
- [ ] No se introducen regresiones en tests existentes

## 2.3 Calidad

- [ ] Validaciones DSL exitosas (cuando aplica)
- [ ] No existen defectos con etiqueta "blocker" o "release-blocker" abiertos
- [ ] El rendimiento cumple los SLAs definidos:
  - Parser DSL: < 100 ms para plantillas < 10 kB
  - Layout Engine: < 50 ms para documentos < 1000 nodos
  - Renderizado ESC/POS: < 500 ms para documentos < 5 kB
  - Renderizado completo end-to-end: < 500 ms en dispositivo móvil estándar

## 2.4 Documentación

- [ ] Documentación técnica actualizada (cuando hay cambios en APIs, contratos o comportamiento)
- [ ] Decisiones de arquitectura registradas (si se tomaron decisiones relevantes)

## 2.5 Integración

- [ ] El código está integrado en la branch correspondiente
- [ ] El pipeline CI/CD ejecuta exitosamente
- [ ] Se respeta la separación de responsabilidades definida en la arquitectura

---

# 3. Criterios Adicionales para Releases

Para considerar un release como **Done**, adicionalmente:

- [ ] Todas las historias del sprint cumplen el DoD individual
- [ ] Pruebas de regresión completas ejecutadas
- [ ] Versión etiquetada según SemVer
- [ ] Release notes documentadas
- [ ] Paquetes NuGet publicados (cuando aplica)

---

# 4. Notas

- Este documento reemplaza cualquier definición de DoD incluida en documentos de sprint individuales.
- Los sprints plans pueden referenciar este documento pero NO deben redefinir el DoD.
- Cualquier cambio al DoD debe ser aprobado por el equipo y versionado en este documento.

---

# 5. Control de Cambios

| Versión | Fecha      | Descripción     |
| ------- | ---------- | --------------- |
| 1.0     | 2026-03-28 | Versión inicial — consolidación de criterios dispersos |

---

**Fin del documento**
