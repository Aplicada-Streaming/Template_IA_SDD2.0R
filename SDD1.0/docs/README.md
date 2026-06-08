# 🧠 Motor DSL de Generación de Documentos

Sistema de generación y procesamiento de documentos basado en un motor DSL (Domain Specific Language), diseñado para interpretar plantillas, aplicar reglas de negocio y renderizar salidas estructuradas.

Construido con **.NET 10**, orientado a arquitectura extensible, versionado semántico y ejecución determinística.

---

## 📌 Descripción

El Motor DSL es una librería y conjunto de componentes que permiten:

- Definir documentos mediante un lenguaje DSL estructurado  
- Interpretar nodos y expresiones  
- Aplicar reglas de negocio y evaluación dinámica  
- Renderizar documentos en múltiples formatos  
- Extender funcionalidades mediante plugins  
- Versionar y mantener compatibilidad entre cambios  

El repositorio incluye documentación técnica, especificaciones del DSL, reglas de validación, pruebas y guías de extensibilidad.

---

## 🎯 Objetivos del producto

- Definir un lenguaje DSL flexible y extensible  
- Permitir generación de documentos parametrizados  
- Separar contenido, lógica y presentación  
- Facilitar reutilización de plantillas  
- Garantizar trazabilidad y versionado  
- Soportar evolución del DSL sin romper compatibilidad  

---

## 🧱 Stack tecnológico

**Core**

- .NET 10  
- C#  
- Arquitectura modular / orientada a componentes  

**Procesamiento**

- Evaluación de expresiones  
- Motor de reglas  
- Renderizado de documentos  

**DevOps**

- Git  
- Pipeline CI/CD  
- Versionado SemVer  
- Publicación de paquetes (NuGet o equivalente)  

---

## 🏗️ Arquitectura (resumen)

El sistema se organiza en capas y componentes desacoplados:

- Núcleo del motor (interpretación DSL)  
- Evaluador de expresiones  
- Renderizadores  
- Sistema de extensiones/plugins  
- Capa de validación  
- Contratos de entrada/salida  

📄 Ver detalle: [Arquitectura de solución](05_arquitectura_tecnica/arquitectura-solucion_v1.0.md)

---

## 📂 Estructura del repositorio

```text
/docs
  00_contexto/              — Visión, alcance, roadmap, acuerdo de equipo
  01_necesidades_negocio/   — Necesidades de negocio (NB-01 a NB-06)
  02_especificacion_funcional/ — CU, reglas de negocio, modelo de datos, DSL
  03_ux-ui/                 — Experiencia de uso del motor, wireframes
  04_prompts_ai/            — Prompts para clasificación IA
  05_arquitectura_tecnica/  — ADRs, contratos, flujo de ejecución, extensibilidad
  06_backlog-tecnico/       — Product backlog, backlog técnico, DoR
  07_plan-sprint/           — Sprint plans (01-08), velocity, templates
  08_calidad_y_pruebas/     — Estrategia QA, DoD, casos de prueba, matriz
  09_devops/                — CI/CD, versionado, deploy, publicación NuGet
  10_developer_guide/       — Guía de integración, formatos, ejemplos
  11_examples/              — Ejemplos progresivos (simple → avanzado)
/devs                       — Herramientas y documentación para contribuidores
```

---

## 📖 Índice de Carpetas

| Carpeta | Responsabilidad | Orden de Lectura |
|---------|----------------|:----------------:|
| [00_contexto](00_contexto/) | Visión, alcance, roadmap, acuerdo de equipo | 1 |
| [01_necesidades_negocio](01_necesidades_negocio/) | Necesidades de negocio y trazabilidad | 2 |
| [02_especificacion_funcional](02_especificacion_funcional/) | Casos de uso, reglas de negocio, DSL, modelo de datos | 3 |
| [03_ux-ui](03_ux-ui/) | Experiencia de uso del motor, representaciones de salida | 4 |
| [04_prompts_ai](04_prompts_ai/) | Prompts de IA para clasificación de documentos | 5 |
| [05_arquitectura_tecnica](05_arquitectura_tecnica/) | Arquitectura, decisiones, contratos, extensibilidad | 6 |
| [06_backlog-tecnico](06_backlog-tecnico/) | Product backlog, backlog técnico, Definition of Ready | 7 |
| [07_plan-sprint](07_plan-sprint/) | Planes de sprint, velocity, ceremonias | 8 |
| [08_calidad_y_pruebas](08_calidad_y_pruebas/) | Estrategia QA, DoD, casos de prueba, cobertura | 9 |
| [09_devops](09_devops/) | Pipeline CI/CD, versionado, entornos, publicación | 10 |
| [10_developer_guide](10_developer_guide/) | Guía de integración para desarrolladores | 11 |
| [11_examples](11_examples/) | Ejemplos funcionales progresivos | 12 |

---

## 🚶 Primer Viaje del Desarrollador

Si sos nuevo en el proyecto, seguí este recorrido:

1. **Entender el contexto** (2 min) → [Visión de producto](00_contexto/vision-producto_v1.0.md)
2. **Conocer el DSL** (5 min) → [Definición del DSL](02_especificacion_funcional/definicion-dsl_v1.0.md)
3. **Ver la arquitectura** (10 min) → [Arquitectura de solución](05_arquitectura_tecnica/arquitectura-solucion_v1.0.md) y [README de arquitectura](05_arquitectura_tecnica/README.md)
4. **Integrar la librería** (15 min) → [Guía de uso](05_arquitectura_tecnica/guia-uso-libreria_v1.0.md)
5. **Probar un ejemplo** (10 min) → [Ejemplo simple](11_examples/ejemplo-01-simple.md)

---

## 🚀 Cómo ejecutar el motor

### Prerrequisitos

* .NET 10 SDK
* Entorno de desarrollo (Visual Studio / VS Code)

---

### 1. Clonar repositorio

```bash
git clone <repo-url>
cd Motor_DSL
```

---

### 2. Restaurar dependencias

```bash
dotnet restore
```

---

### 3. Compilar solución

```bash
dotnet build
```

---

### 4. Ejecutar pruebas

```bash
dotnet test
```

**Cobertura mínima requerida:** 70%

---

## 🧪 Ejecución conceptual del motor

El motor procesa una entrada DSL siguiendo este flujo:

```text
DSL → Parseo → Validación → Evaluación → Render → Output
```

Componentes involucrados:

* Parser DSL
* Validator
* Evaluador de nodos
* Motor de reglas
* Renderizador

---

## 🔌 Conceptos principales

### DSL

Lenguaje estructurado que define:

* Estructura del documento
* Nodos lógicos
* Expresiones
* Reglas de composición

📄 Ver detalle: [Definición del DSL](02_especificacion_funcional/definicion-dsl_v1.0.md)

---

### Nodos

Elementos básicos del DSL que representan:

* Texto
* Condiciones
* Repeticiones
* Expresiones
* Componentes compuestos

---

### Evaluador

Encargado de:

* Resolver expresiones
* Ejecutar lógica condicional
* Interpretar contexto de ejecución

---

### Renderizadores

Transforman el modelo interpretado en una salida concreta:

* Texto plano
* HTML
* PDF
* Otros formatos extensibles

---

### Extensiones

Permiten ampliar el motor mediante:

* Nuevos nodos
* Funciones personalizadas
* Evaluadores adicionales
* Renderizadores específicos

📄 Ver detalle: [Extensibilidad del motor](05_arquitectura_tecnica/extensibilidad-motor_v1.0.md)

---

## 🔄 Flujo de desarrollo

1. Crear branch `feature/*`
2. Implementar cambios en motor / DSL
3. Agregar pruebas unitarias
4. Ejecutar validaciones
5. Crear Pull Request
6. Pipeline CI en verde
7. Merge a `main`
8. Generación de versión

---

## ✅ Definition of Done

Una funcionalidad se considera completa cuando:

* Código compila correctamente
* Tests pasan
* Cobertura ≥ 70%
* Validaciones DSL exitosas
* Documentación actualizada
* PR aprobado

📄 Ver detalle: [Definition of Done](08_calidad_y_pruebas/definition-of-done_v1.0.md)

---

## 🧭 Roadmap

Ver [Roadmap detallado del producto](00_contexto/roadmap-producto_v1.0.md) con trazabilidad Fases → Épicas → Sprints → Releases.

---

## 🧪 Calidad y testing

El proyecto incluye:

* Casos de prueba referenciales
* Matriz de cobertura
* Validaciones del motor
* Estrategias de testing automatizado

📄 Ver documentación: [Calidad y pruebas](08_calidad_y_pruebas/)

---

## 🚀 DevOps

El proyecto cuenta con estrategia de:

* Pipeline CI/CD con SBOM (CycloneDX), SCA (Dependabot), firma de paquete y matriz OS multi-runner
* Versionado SemVer 2.0.0 + Conventional Commits + MinVer (auto-versioning)
* Modelo de distribución: feeds Preview/Stable + consumers downstream
* Publicación en GitHub Packages

📄 Ver documentación: [DevOps](09_devops/)

Incluye:

* [README de sección](09_devops/README.md) — índice, glosario, RACI, decisiones D1–D5
* [pipeline-ci-cd_v1.0.md](09_devops/pipeline-ci-cd_v1.0.md)
* [estrategia-versionado_v1.0.md](09_devops/estrategia-versionado_v1.0.md)
* [entornos-deploy_v1.0.md](09_devops/entornos-deploy_v1.0.md)
* [guia-publicacion-nuget_v1.0.md](09_devops/guia-publicacion-nuget_v1.0.md)

---

## 🤝 Contribución

1. Fork del repositorio
2. Crear branch descriptivo
3. Implementar cambios
4. Agregar pruebas
5. Asegurar validaciones en verde
6. Crear Pull Request

---

## 📜 Licencia

MIT — ver [`LICENSE`](../LICENSE) en la raíz del repositorio.

---

## 📞 Contacto

Equipo de arquitectura y desarrollo del Motor DSL.

---
