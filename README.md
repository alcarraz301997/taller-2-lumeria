# Taller 2 - Spec + Plan + Pruebas (Three Amigos)

## Equipo L1

### Integrantes

| Integrante                       | Cargo             | Rol Three Amigos |
| -------------------------------- | ----------------- | ---------------- |
| Eduardo Arrieta                  | Product Owner     | Product          |
| Junior Guillermo Alcarraz Montes | Backend Developer | Tech Lead        |
| Melisa Gutiérrez                 | QA Engineer       | QA               |

---

## Feature seleccionada

**Automatización de Preguntas NQ en segundo plano**

### Descripción

Actualmente la generación de preguntas NQ requiere ejecución manual y seguimiento operativo. Esta propuesta busca permitir que la generación se ejecute de manera asíncrona mediante procesamiento en segundo plano, permitiendo al usuario continuar con otras actividades mientras el sistema procesa la solicitud.

Los detalles funcionales se encuentran documentados en:

* `spec.md`
* `plan.md`
* `test-cases.md`
* `constitution.md`

---

# Coverage Matrix

| Requisito | Plan | Casos de prueba | Estado |
|-----------|------|----------------|--------|
| HU-1 AC-1 — Ejecución automática | Plan §2 (Registro de faltantes) | TC-01, TC-15 | ✅ |
| HU-1 AC-2 — FIFO cronológico | Plan §2 (Gestor FIFO) | TC-03 | ✅ |
| HU-1 AC-3 — Cola FIFO | Plan §2 (Gestor FIFO) | TC-03 | ✅ |
| HU-1 AC-4 — Cursos habilitados | Plan §2 (Integración con NQ) | TC-04, TC-10 | ✅ |
| HU-1 AC-5 — Atributos payload | Plan §2 (Integración con NQ) | TC-04 | ✅ |
| HU-1 AC-6 — División por bloques | Plan §1 (Flujo general) | TC-02, TC-20 | ✅ |
| HU-1 AC-7 — API existente | Plan §2 (Integración con NQ) | TC-01, TC-02, TC-05 | ✅ |
| HU-1 AC-8 — Error → PENDING | Plan §1 (Flujo general) | TC-11 | ✅ |
| HU-2 AC-1 — Recepción automática | Plan §2 (Tabla temporal) | TC-05 | ✅ |
| HU-2 AC-2 — Validación duplicidad | Plan §2 (Validador de duplicidad) | TC-05, TC-08, TC-22 | ✅ |
| HU-2 AC-3 — Descarte duplicados | Plan §2 (Validador de duplicidad) | TC-05, TC-08, TC-22 | ✅ |
| HU-2 AC-4 — Reposición automática | Plan §2 (Control de reposición) | TC-08 | ✅ |
| HU-2 AC-5 — Reenvío bajo bloques | Plan §1 (Flujo general) | TC-08, TC-20 | ✅ |
| HU-2 AC-6 — Tabla temporal | Plan §2 (Tabla temporal) | TC-05 | ✅ |
| HU-2 AC-7 — Conservación flujo | Plan §2 (Tabla temporal) | TC-05 | ✅ |
| HU-2 AC-8 — Reintento hasta completar | Plan §2 (Control de reposición) | TC-08 | ✅ |
| Constitution Art. 2 — Seguridad | — | TC-18, TC-21 | ✅ |
| Constitution Art. 3 — Idempotencia | — | TC-17, TC-09 | ✅ |
| Constitution Art. 7 — Trazabilidad | Plan §2 (Auditoría) | TC-16, TC-13, TC-19 | ✅ |

> Matriz completada y validada. Total: 22 casos de prueba cubriendo 16 ACs + 5 casos borde + 3 NFRs + 3 artículos de constitución.

---

# Gate de Claridad

## Grupo revisor

Equipo L2

## Checklist

| Categoría    | Resultado | Observación breve |
| ------------ | --------- | ----------------- |
| Completitud  | ⚠️ Parcial | Faltan NFR cuantitativos (latencia, RPS), manejo HTTP 429, timeouts y Redis indisponible.
| Claridad     | ⚠️ Parcial | Ambigüedades en trigger (inmediato vs batch), y en la cantidad a solicitar por faltante.
| Consistencia | ⚠️ Parcial | Mezcla de idioma (constitución: inglés técnicos; spec: español). "3 ciclos" aparece en TCs; falta en spec.
| Testabilidad | ✅ Parcial | TCs detallados; faltan cargas/429/timeouts/Redis.

## Hallazgos clave

- Las US y TCs son mayoritariamente medibles (DB counts, estados, invocaciones).  
- Falta especificar NFRs numéricos (ej. SLA/latencia, RPS, contención por worker).  
- Faltan TCs para HTTP 429, connection/read timeouts y ausencia de Redis.  
- Ambigüedad crítica: ¿el envío se ejecuta inmediatamente al registrar el faltante o por batch programado?  
- Consistencia: unificar convención para identificadores técnicos (course_id, estados, nombres en código).

## Acciones recomendadas (prioridad alta → baja)

1. Añadir en spec.md: NFR cuantitativos (latencia objetivo por operación / throughput), y política para HTTP 429 y timeouts. (Alta)
2. Incorporar en spec.md la regla "máximo 3 ciclos de reposición" (HU-2) para evitar ambigüedad. (Alta)
3. Añadir TCs para HTTP 429, connection/read timeout y Redis indisponible; incluir test de carga/volumen. (Alta)
4. Decidir convención de idioma para términos técnicos y propagarlas a constitution.md y spec.md. (Media)
5. Definir trigger de envío (inmediato vs batch) y consolidación de faltantes (si aplica). (Bloqueante para diseño)

## Preguntas abiertas (necesitan respuesta del PO)

1. ¿Cuántas preguntas debe solicitar el sistema a NQ por cada faltante? (Ej: solicitar la cantidad faltante, dividida en bloques ≤5 — recomendado).  
2. ¿Consolidar registros repetidos antes de enviar o procesarlos individualmente?  
3. ¿Enviar inmediatamente al crearse el faltante o procesar por batch programado?

---

# Historial de refinamiento

El equipo trabajó siguiendo el enfoque Spec-Driven Development (SDD), refinando progresivamente los artefactos mediante iteraciones y revisiones internas entre los roles Product, Tech Lead y QA.

Los cambios y refinamientos pueden consultarse en el historial de commits del repositorio.
