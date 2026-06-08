# Lógica de Triage Inicial — Complemento para el Skill primavera-unifier

## Propósito

Este complemento se ejecuta **antes** de activar el skill `primavera-unifier`. Su función es hacer preguntas iniciales para diagnosticar el caso, reducir ambigüedad y rutear al dominio correcto dentro del skill.

---

## Árbol de Triage

### Paso 1 — Determinar el módulo

Pregunta al usuario:
> ¿Sobre qué módulo de Unifier necesitas ayuda?

| Opción | Ruteo |
|--------|-------|
| **BP / Formularios** | → Paso 2A |
| **Workflows** | → Paso 2B |
| **Cost Manager / CBS / Cost Sheet** | → Paso 2C |
| **Fórmulas / Lógica de datos** | → Paso 2D |
| **Permisos / Seguridad** | → Paso 2E |
| **Reportes / Data Views / UDRs** | → Paso 2F |
| **Integraciones (P6, SAP, etc.)** | → Paso 2G |
| **Shells / Jerarquía** | → Paso 2H |
| **Document Manager** | → Paso 2I |
| **Gates / Lifecycle** | → Paso 2J |
| **No sé / No estoy seguro** | → Paso 1B |

### Paso 1B — Exploración guiada

> Cuéntame brevemente qué necesitas lograr (máx. 3 líneas). Identificaré el módulo por ti.

Analiza la entrada del usuario y mapea a uno de los pasos 2A–2J.

---

### Paso 2A — BP / Formularios

> ¿Qué tipo de BP necesitas diseñar o modificar?

| Opción | Ruteo |
|--------|-------|
| **Cost BP** (Commit, Spend, Budget) | → Pregunta 3A |
| **Document BP** (Transmittal, RFI, Submittal) | → Pregunta 3B |
| **Line Item BP** (con grilla de líneas) | → Pregunta 3C |
| **Simple BP** (sin líneas) | → Pregunta 3D |
| **Text BP** (notas, reportes) | → Pregunta 3E |
| **Non-workflow BP** (solo almacena datos, sin aprobación) | → Pregunta 3F |

**Pregunta 3A (Cost BP):**
> ¿Es de tipo Commit (contrato/OC), Spend (factura/pago) o Budget (cambio presupuestario)?
> *(Cada uno tiene distinto impacto en la Cost Sheet)*

**Pregunta 3B (Document BP):**
> ¿Necesita revision control? ¿Requiere respuesta formal del destinatario?

**Pregunta 3C (Line Item BP):**
> ¿Cuántas columnas tiene la grilla? ¿Requiere múltiples tabs para organizar líneas?

**Pregunta 3D (Simple BP):**
> ¿Cuántos campos aprox.? ¿Alguno requiere fórmula o auto-populate?

**Pregunta 3E (Text BP):**
> ¿Necesita Text Entry Area o Response List?

**Pregunta 3F (Non-workflow):**
> ¿Necesita terminal status o los registros deben quedar editables siempre?

---

### Paso 2B — Workflows

> ¿Qué tipo de workflow necesitas diseñar o depurar?

| Opción | Ruteo |
|--------|-------|
| **Diseñar workflow desde cero** | → Pregunta 4A |
| **Modificar workflow existente** | → Pregunta 4B |
| **Depurar workflow que no funciona** | → Pregunta 4C |
| **Conditional / routing lógico** | → Pregunta 4D |

**Pregunta 4A:**
> ¿Cuántos pasos (steps) tiene el flujo? ¿Quiénes son los actores? ¿Hay aprobación en paralelo (multiple approvers)?

**Pregunta 4B:**
> ¿Qué necesitas cambiar? (agregar step, cambiar actor, modificar acción, ajustar routing)

**Pregunta 4C:**
> Describe el error: ¿el registro no avanza? ¿salta al paso equivocado? ¿no llega notificación?

**Pregunta 4D:**
> ¿Qué campo condiciona el ruteo? (monto, tipo, región, etc.) ¿Cuántas ramas tiene?

---

### Paso 2C — Cost Manager

> ¿Sobre qué componente de Cost Manager necesitas ayuda?

| Opción | Ruteo |
|--------|-------|
| **CBS (Cost Breakdown Structure)** | → Pregunta 5A |
| **Cost Sheet** (columnas, configuración) | → Pregunta 5B |
| **SBS (Summary Budget Sheet)** | → Pregunta 5C |
| **Forecast / EAC** | → Pregunta 5D |
| **Rollups desde BPs a Cost Sheet** | → Pregunta 5E |
| **Budget Changes** | → Pregunta 5F |

**Pregunta 5A:**
> ¿Cuántos niveles de jerarquía tiene el CBS? ¿Es fijo o variable por shell?

**Pregunta 5B:**
> ¿Qué columna necesitas configurar? ¿Es Budget, Commitment, Actual, o Forecast?

**Pregunta 5C:**
> ¿El SBS consolida desde Cost Sheets de shells hijos? ¿Tiene fórmulas entre columnas?

**Pregunta 5D:**
> ¿El Forecast es manual o se calcula desde BPs? ¿Usan EAC = Actual + ETC?

**Pregunta 5E:**
> ¿Qué tipo de BP (Commit/Spend) postea a la Cost Sheet? ¿En qué paso del workflow se dispara el posteo?

**Pregunta 5F:**
> ¿Los budget changes afectan el presupuesto original o el aprobado?

---

### Paso 2D — Fórmulas / Lógica

> ¿Qué tipo de lógica necesitas configurar?

| Opción | Ruteo |
|--------|-------|
| **Auto-populate** (llenar campo desde otra fuente) | → Pregunta 6A |
| **Advanced Formula** (cálculo con if-then-else, string, fecha) | → Pregunta 6B |
| **Validation Rule** (validar datos antes de guardar) | → Pregunta 6C |
| **Conditional Routing** (ruteo condicional en workflow) | → Pregunta 6D |
| **Fórmula en SBS / Cost Sheet** | → Pregunta 6E |
| **No sé la diferencia entre Auto-populate y Advanced Formula** | → Explicar: AP se ejecuta 1 vez al crear; AF se re-evalúa siempre. Luego preguntar 6A vs 6B. |

**Pregunta 6A:**
> ¿La fuente es desde otro campo del mismo formulario, un BP relacionado, el Shell padre, un DDS, o un valor constante?

**Pregunta 6B:**
> ¿Es numérica (total = cantidad × precio), de texto (concatenar campos), de fecha (dateDiff, addDays) o condicional (if-then-else)?

**Pregunta 6C:**
> ¿Es un campo required condicional, rango de valores, validación cruzada entre campos, o suma debe ser 100%?

**Pregunta 6D:**
> ¿Qué campo evalúa la condición? ¿Cuántas ramas de salida tiene?

**Pregunta 6E:**
> ¿Es entre columnas del SBS/Cost Sheet o rollup desde BPs?

---

### Paso 2E — Permisos

> ¿A qué nivel necesitas configurar permisos?

| Opción | Ruteo |
|--------|-------|
| **Company-level** | Contexto global |
| **Shell-level** (proyecto/programa) | → Pregunta 7A |
| **BP-level** (quién crea/ve/edita registros) | → Pregunta 7B |
| **Record-level** (acceso por registro individual) | → Pregunta 7C |
| **Field-level** (campo editable vs read-only por rol) | → Pregunta 7D |

**Pregunta 7A:**
> ¿Es un nuevo shell o necesitas modificar permisos existentes? ¿Usan Roles o usuarios directos?

**Pregunta 7B:**
> ¿Quién debe poder crear? ¿Quién debe poder ver todos los registros vs solo los suyos?

**Pregunta 7C:**
> ¿El acceso depende del creador, del paso del workflow, o de un campo del formulario?

**Pregunta 7D:**
> ¿Qué campos cambian según el rol? ¿En qué paso del workflow?

---

### Paso 2F — Reportes

> ¿Qué tipo de reporte necesitas?

| Opción | Ruteo |
|--------|-------|
| **Data View** (consulta tipo SQL sobre datos de Unifier) | → Pregunta 8A |
| **UDR (User Defined Report)** | → Pregunta 8B |
| **Dashboard / Portlets** | → Pregunta 8C |
| **Log View** (lista de registros de un BP) | → Pregunta 8D |

**Pregunta 8A:**
> ¿Qué BP(s) involucra? ¿Necesitas filtros, agrupaciones, o campos calculados?

**Pregunta 8B:**
> ¿Es tabular o resumen? ¿Qué columnas debe mostrar?

**Pregunta 8C:**
> ¿Qué KPI debe mostrar? ¿Gráfico, tabla, indicador?

**Pregunta 8D:**
> ¿Qué columnas debe mostrar el log? ¿Qué filtros por defecto?

---

### Paso 2G — Integraciones

> ¿Qué sistema externo necesitas integrar con Unifier?

| Opción | Ruteo |
|--------|-------|
| **Oracle P6** (Schedule / Cost) | → Pregunta 9A |
| **SAP / ERP** | → Informar: integración vía REST API o archivos planos |
| **Otro (GIS, SharePoint, etc.)** | → Preguntar por API disponible |

**Pregunta 9A:**
> ¿Es integración de Schedule (actividades), Cost (CBS ↔ WBS) o Resource? ¿Flujo unidireccional o bidireccional?

---

### Paso 2H — Shells

> ¿Necesitas crear un nuevo tipo de Shell, modificar uno existente, o configurar la jerarquía?

**Preguntas:**
> - ¿Cuántos niveles de jerarquía? (Programa → Proyecto → Subproyecto)
> - ¿Heredan atributos del shell padre?
> - ¿Usan auto-creación desde un BP?

---

### Paso 2I — Document Manager

> ¿Necesitas configurar la estructura de carpetas, revision control, o distribution lists?

**Preguntas:**
> - ¿La estructura de carpetas es fija o varía por shell?
> - ¿Requieren auto-publish desde Document BPs?

---

### Paso 2J — Gates / Lifecycle

> ¿Necesitas diseñar un Gate Process para controlar el ciclo de vida de un shell?

**Preguntas:**
> - ¿Cuántas fases tiene el lifecycle?
> - ¿Qué BPs deben completarse antes de pasar a la siguiente fase?
> - ¿Qué criterios determinan si un gate se aprueba o rechaza?

---

## Reglas de Implementación

### Orden de preguntas
1. Siempre empezar con **Paso 1** (módulo).
2. Si el usuario ya mencionó el módulo en su mensaje inicial, saltar al paso correspondiente.
3. Hacer **una pregunta a la vez** — no abrumar con preguntas múltiples.
4. Después de 2 respuestas del usuario, debes tener suficiente contexto para dar una respuesta asertiva.

### Cuando el usuario ya dio suficiente contexto
Si el usuario proporcionó toda la información necesaria en su mensaje inicial (ej. *"Necesito un BP para control de órdenes de cambio con flujo de aprobación por montos"*), entonces:
1. Identifica el módulo (BP → Cost BP → Commit Change).
2. No hagas preguntas redundantes.
3. Ve directo a la respuesta usando el skill `primavera-unifier`.

### Formato de respuesta asertiva
Una vez identificado el caso, la respuesta debe incluir:

```
## Contexto del caso
[Resumen de lo diagnosticado]

## Solución / Configuración
[Pasos detallados o referencias a los documentos del skill]

## Consideraciones
[Edge cases, mejores prácticas, errores comunes]
```

### Integración con el skill
- Después del triage, activa el skill `primavera-unifier` con el contexto recolectado.
- Pasa al skill las respuestas como parámetros de contexto.
- El skill usará esa información para dar una respuesta más precisa.

---

## Tabla de Mapeo Rápido

| Término del usuario | Módulo probable |
|---------------------|-----------------|
| "Crear BP", "formulario", "upper form", "detail form" | BP / Formularios |
| "Flujo", "aprobación", "step", "ruteo", "workflow" | Workflows |
| "CBS", "Cost Sheet", "hoja de costos", "SBS", "presupuesto" | Cost Manager |
| "Fórmula", "auto-populate", "cálculo", "validación" | Fórmulas / Lógica |
| "Permiso", "rol", "acceso", "seguridad" | Permisos |
| "Reporte", "Data View", "UDR", "dashboard" | Reportes |
| "Integración", "P6", "SAP", "API" | Integraciones |
| "Shell", "jerarquía", "programa", "proyecto" | Shells |
| "Document Manager", "transmittal", "submittal", "RFI" | Document Manager |
| "Gate", "lifecycle", "fase", "aprobación de fase" | Gates |

---

## Flujo de Decisión Resumido

```
Entrada del usuario
    │
    ├─ ¿Mencionó el módulo explícitamente? → Saltar a Paso 2X
    │
    └─ No → Paso 1: ¿Qué módulo?
              │
              ├─ BP         → 2A
              ├─ Workflow   → 2B
              ├─ Cost       → 2C
              ├─ Fórmulas   → 2D
              ├─ Permisos   → 2E
              ├─ Reportes   → 2F
              ├─ Integrac.  → 2G
              ├─ Shells     → 2H
              ├─ Doc Mgr    → 2I
              ├─ Gates      → 2J
              └─ No sabe    → 1B (exploración libre)
                    │
                    └─ 1-2 preguntas hasta identificar módulo
                           │
                           └─ → Paso 2X correspondiente
                                  │
                                  └─ 1-2 preguntas específicas
                                         │
                                         └─ Respuesta asertiva + skill
```

---

## Ejemplos de uso

### Ejemplo 1: Usuario vago
> *"Necesito ayuda con Unifier"*

**Triage:** Paso 1 → ¿Sobre qué módulo?
> *"Quiero crear un flujo de aprobación"*

**Triage:** Paso 2B → Workflows → Pregunta 4A: ¿Cuántos pasos? ¿Actores?
> *"3 pasos: Creador, Revisor, Aprobador. Monto > 10k va a Gerencia"*

**Triage:** Contexto suficiente → Respuesta asertiva + skill.

### Ejemplo 2: Usuario específico
> *"La fórmula de dateDiff me está dando 0 días cuando combino DatePicker con DateOnlyPicker"*

**Triage:** Se detecta "fórmula" + "dateDiff" + "DatePicker/DateOnlyPicker" → módulo Fórmulas. Sin preguntas adicionales → Respuesta directa sobre Time Zone + edge case.

### Ejemplo 3: Usuario semi-específico
> *"Necesito configurar un BP de transmittals con revision control"*

**Triage:** Se detecta "BP" + "transmittals" + "revision control" → módulo BP → Document BP → Pregunta 3B: ¿Respuesta formal requerida?

---

## Mantenimiento

Este triage evoluciona con la práctica. Si encuentras que ciertas preguntas no llevan al diagnóstico correcto, ajusta el árbol. Si hay módulos nuevos en Unifier, agrégalos al Paso 1.
