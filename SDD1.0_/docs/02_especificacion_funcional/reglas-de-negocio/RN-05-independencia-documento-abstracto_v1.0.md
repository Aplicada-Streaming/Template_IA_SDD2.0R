# Regla de Negocio: Independencia del Documento Abstracto

**Código:** RN-05
**Archivo:** RN-05-independencia-documento-abstracto_v1.0.md
**Versión:** 1.0
**Estado:** Aprobada
**Fecha:** 2026-03-28
**Autor:** Equipo Funcional / Arquitectura

---

# 1. Descripción

Esta regla de negocio establece que el documento abstracto generado por el motor no debe contener dependencias ni referencias a un renderizador específico.

El objetivo es garantizar que el modelo interno sea neutral respecto al formato de salida, permitiendo que un mismo documento pueda ser consumido por distintos renderizadores sin necesidad de modificaciones.

---

# 2. Motivación de Negocio

Permitir que el mismo documento abstracto sea renderizado a ESC/POS, texto plano, vista previa UI o PDF sin modificaciones.

La independencia del documento abstracto permite:

* reutilizar una única representación interna para múltiples formatos de salida
* agregar nuevos renderizadores sin impactar la lógica de generación del documento
* simplificar las pruebas al validar el AST de forma aislada
* garantizar consistencia semántica entre diferentes salidas

---

# 3. Definición de la Regla

El modelo interno (AST/DocumentModel) debe representar el contenido y layout de forma neutral. Los nodos, estilos y propiedades deben ser agnósticos al formato de salida.

Esto implica que:

* los nodos del AST no deben contener comandos específicos de ningún formato (ESC/POS, PDF, HTML, etc.)
* las propiedades de estilo deben expresarse en términos abstractos (bold, alignment, fontSize) y no en códigos de control
* la estructura del documento debe ser independiente de las capacidades del dispositivo destino
* ningún nodo debe referenciar directamente un renderizador o formato concreto

---

# 4. Condiciones de Aplicación

La regla aplica en los siguientes casos de uso:

* CU-03 Parsear plantilla DSL y generar AST
* CU-04 Evaluar expresiones y resolver datos
* CU-05 Ejecutar motor de layout

Debe aplicarse durante toda la fase de construcción del documento abstracto, antes de que intervenga cualquier renderizador.

---

# 5. Resultados Esperados

* El AST generado no contiene referencias a formatos específicos.
* Un mismo AST puede ser consumido por múltiples renderizadores.
* La adición de nuevos renderizadores no requiere cambios en la generación del AST.
* Las pruebas del AST pueden ejecutarse sin dependencia de renderizadores.

---

# 6. Excepciones

No se contemplan excepciones para esta regla. Todo documento abstracto generado por el motor debe cumplir con la independencia de formato.

---

# 7. Criterios de Validación

## CV-01 Ausencia de referencias a formatos específicos

**Dado** un AST generado
**Cuando** se inspecciona
**Entonces** no contiene referencias a ESC/POS, PDF ni ningún formato específico.

---

## CV-02 Coherencia entre renderizados

**Dado** un mismo AST
**Cuando** se renderiza a ESC/POS y a texto plano
**Entonces** ambas salidas son coherentes con el contenido original.

---

# 8. Impacto

Esta regla impacta directamente en:

* CU-03 Parsear plantilla DSL y generar AST
* CU-04 Evaluar expresiones y resolver datos
* CU-05 Ejecutar motor de layout
* CU-06 Renderizar a ESC/POS
* CU-07 Renderizar a texto plano
* CU-08 Renderizar a vista previa UI
* CU-09 Renderizar a PDF

La independencia del documento abstracto es fundamental para la extensibilidad y mantenibilidad del motor.

---

# 9. Control de Cambios

| Versión | Fecha      | Descripción     |
| ------- | ---------- | --------------- |
| 1.0     | 2026-03-28 | Versión inicial |

---

**Fin del documento**
