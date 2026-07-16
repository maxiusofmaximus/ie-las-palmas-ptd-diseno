# Backlog Inicial

**Sistema:** PTD — Planes de Trabajo Docentes (IE Las Palmas)
**Versión:** 1.0 · **Fecha:** Julio 2026
**Marco:** Épicas + Historias de Usuario (HU) priorizadas MoSCoW; estimación relativa P (pequeña), M (mediana), G (grande), XL. Trazabilidad a RF/UC.

> Concebido para ser implementado por la herramienta Codex. El equipo (humano) debe descomponer/encajar las HU en tareas técnicas (migración, endpoint, agregado, test) en su herramienta de tracking.

---

## 1. Épicas (resumen)

| ID | Épica | Outcome principal |
|---|---|---|
| E0 | Fundaciones del proyecto | Repositorios, CI, esqueletos backend/frontend, contenerización. |
| E1 | Identidad y Acceso | Acceder al Módulo desde el portal, sesiones, permisos. |
| E2 | Administración de maestros | Malla curricular, años lectivos, personas, roles. |
| E3 | Asignación docente | Coordinación asigna docentes↔cursos/grados/asignaturas/periodos; duplas. |
| E4 | Plantilla configurable | Plantillas versionables y secciones configurable. |
| E5 | Diligenciamiento del Plan de Aula | Crear/editar/autoguardar/enviar/devolver; estructura de §10 SRS. |
| E6 | IA (Asistente de escritura) | AIService + Inforge.dev + trazabilidad + UI. |
| E7 | Evaluación (Coordinación) | Revisar, calificar, aprobar/rechazar/devolver, pautas DUA. |
| E8 | Rectoría / Gobernanza | Permisos, parámetros, indikes, dashboards, validación de históricos. |
| E9 | Importación Excel / históricos | Carga, validación, transformación, batches, reversión, trazabilidad. |
| E10 | Reportes, auditoría y exportación | KPIs, gráficos, PDF/Excel/CSV, auditoría, historial. |
| E11 | Calidad (pruebas, accesibilidad, seguridad) | QA plan, pruebas unitarias/integración/importación; SAST; WCAG AA. |
| E12 | Operación y despliegue | Infra, backups, observabilidad, monitoreo IA. |

---

## 2. Backlog por épica

### E0 — Fundaciones del proyecto

| HU/Task | Descripción | Prioridad | Tam. | RF |
|---|---|---|---|---|
| E0.1 | Crear repos mono/mono-repo con estructura: `src/modules/{IAM,Institution,Assignment,PlanTemplate,Plan,Evaluation,Intelligence,Import,Reporting,Notification,Audit}` | Must | M | C4 |
| E0.2 | Configurar pipelines CI: build, test, lint, SAST. | Must | M | §13 |
| E0.3 | Dockerfiles multi-stage + docker-compose dev/staging. | Must | P | RNF-042 |
| E0.4 | Esquema de migraciones DB con Flyway/Liquibase. Seed: roles/permisos/acción IA/secciones. | Must | M | — |
| E0.5 | Configurar bóveda para secretos y variables de entorno por ambiente. | Must | P | RN-111 |
| E0.6 | Observabilidad base: logs JSON con correlationId, métricas básicas. | Should | M | RNF-060/061 |
| E0.7 | Clientes SDK desde OpenAPI generado (frontend + tipos). | Should | P | — |

### E1 — Identidad y Acceso *(RF-001..RF-003)*

| HU | Resumen | Prioridad | Tam. | UC |
|---|---|---|---|---|
| 1.1 | Como Docente, entrar al Módulo sin relogin. | Must | M | UC-001 |
| 1.2 | Como Admin, denegar acceso sin permiso y audit auría de denegación. | Must | P | UC-001 |
| 1.3 | Adaptador `IdentityProvider` (mock + implementación con SSO institución). | Must | M | — |
| 1.4 | Middleware de autorización por permiso (Command-level). | Must | P | — |

### E2 — Administración de maestros *(RF-010..RF-015)*

| HU | Resumen | Prioridad | Tam. | UC |
|---|---|---|---|---|
| 2.1 | Administrar docentes (CRUD + inactivación). | Must | M | UC-002 |
| 2.2 | Administrar malla curricular (grados/cursos/áreas/nodos/asignaturas). | Must | M | UC-003 |
| 2.3 | Administrar años lectivos y periodos. | Must | P | UC-004 |
| 2.4 | Admin permisos + roles + permisos extra por usuario. | Must | M | UC-002 |
| 2.5 | Restricción: un solo año activo (RN-010). | Must | P | — |

### E3 — Asignación docente *(RF-013, RF-023)*

| HU | Resumen | Prioridad | Tam. | UC |
|---|---|---|---|---|
| 3.1 | Asignación docente→cursos/grados/asignaturas/periodos con fechas. | Must | M | UC-005 |
| 3.2 | Modo dupla cuando comparten grado/asignatura/periodo. | Must | P | UC-005 |
| 3.3 | Docente solo ve asignaciones vigentes (UI filtrada). | Must | P | UC-007 |
| 3.4 | Validar jurisdicción coordinación *(RN-043)*. | Must | P | — |

### E4 — Plantilla configurable *(RF-014, RF-015)*

| HU | Resumen | Prioridad | Tam. | UC |
|---|---|---|---|---|
| 4.1 | Plantilla default con secciones del §10 SRS. | Must | M | UC-006 |
| 4.2 | Activar/ocultar/obligatoriedad de secciones y subsecciones (RN-030). | Must | M | UC-006 |
| 4.3 | Versionado (RN-031) — version ligada a Plan. | Must | M | UC-006 |
| 4.4 | Validar inmutabilidad de secciones base (RN-030). | Must | P | — |

### E5 — Diligenciamiento del Plan de Aula *(RF-020..RF-025)*

| HU | Resumen | Prioridad | Tam. | UC |
|---|---|---|---|---|
| 5.1 | Crear Plan a partir de asignación + plantilla vigente. | Must | M | UC-007 |
| 5.2 | Encabezado (docentes, fechas, grado, semanas/clases). | Must | P | UC-007 |
| 5.3 | Sección Caracterización por curso. | Must | P | UC-007 |
| 5.4 | Sección Competencias (específicas + socio-emocionales). | Must | P | UC-007 |
| 5.5 | Sección Indicadores (conceptuales/procedimentales/actitudinales con %). | Must | P | UC-007 |
| 5.6 | Sección Actividades/Ejes (inicio/nueva/final/evaluación/criterios, num_clases). | Must | M | UC-007 |
| 5.7 | Seguimiento docente por Eje × Curso. | Must | M | UC-007 |
| 5.8 | Checklist DUA por Eje. | Must | P | UC-007 |
| 5.9 | Seguimientos Coordinación y Psicopedagógico 1..N por Plan. | Must | M | UC-011/012 |
| 5.10 | Autoguardado de borrador. | Should | P | UC-007 |
| 5.11 | Enviar Plan (valida RN-041) y no editable mientras Enviado. | Must | M | UC-008 |
| 5.12 | Editar Plan DEVUELTO según observaciones. | Must | P | UC-011 |
| 5.13 | Descargar PDF del Plan (formato institucional). | Must | M | UC-010 |
| 5.14 | Descargar Excel/CSV del Plan. | Should | P | UC-010 |
| 5.15 | Versiones/historial del Plan. | Must | M | UC-018 |

### E6 — IA (Asistente de escritura) *(RF-024, RF-050..RF-053)*

| HU | Resumen | Prioridad | Tam. | UC |
|---|---|---|---|---|
| 6.1 | Puerto `AIService` y adaptador Inforge.dev con CLI de aprovisionamiento. | Must | M | — |
| 6.2 | Acciones IA: borrador, mejorar, ortografía, pedagógico, resumir, expandir, proponer actividades, competencias, indicadores, observaciones, preguntar. | Must | G | UC-009 |
| 6.3 | Context provider anonimiza PII (RN-054). | Must | M | — |
| 6.4 | UI diff/sugerencia; docente acepta/edita/rechaza. | Must | M | UC-009 |
| 6.5 | Trazabilidad: `ia_invocacion` (tokens/costo/latencia/resultado). | Must | P | — |
| 6.6 | Configuración IA (proveedor/modelo/prompts/temperatura/tokens/costo). | Must | M | UC-009 |
| 6.7 | Límites por rol/usuario/día (RN-053). | Must | P | UC-009 |
| 6.8 | Test de contrato AIService (cambio de proveedor aislado — RN-055). | Must | M | — |
| 6.9 | Fallback/Circuit-breaker para errores de Inforge.dev. | Should | M | — |

### E7 — Evaluación (Coordinación) *(RF-030..RF-032)*

| HU | Resumen | Prioridad | Tam. | UC |
|---|---|---|---|---|
| 7.1 | Cola de Planes enviados, filtros multi-criterio. | Must | M | UC-011 |
| 7.2 | Revisión y observación; marcar pautas DUA. | Must | M | UC-011 |
| 7.3 | Calificar/aprobar/rechazar/devolver con transición de estado (RN-040/044). | Must | M | UC-011 |
| 7.4 | Notificaciones automáticas al docente (RN-090). | Must | M | — |
| 7.5 | Apoyo psico diligencia su sección. | Should | P | UC-012 |

### E8 — Rectoría / Gobernanza *(RF-040..RF-042)*

| HU | Resumen | Prioridad | Tam. | UC |
|---|---|---|---|---|
| 8.1 | Administrar permisos globales y por usuario. | Must | M | UC-002 |
| 8.2 | Parámetros generales (umbra śles configurables). | Should | M | — |
| 8.3 | Validar lotes importados (UC-014) — ver E9. | Must | M | UC-014 |

### E9 — Importación Excel / históricos *(RF-070..RF-076)* — *feature con más riesgo*

| HU | Resumen | Prioridad | Tam. | UC |
|---|---|---|---|---|
| 9.1 | `ExcelReader` OpenXML con detección de FORMATO + periodos `1°..n°`. | Must | M | — |
| 9.2 | Validación previa: campos/tipos/obligatorios/consistencia; resumen descargable (RN-071/RN-072). | Must | G | UC-013 |
| 9.3 | Carga en storage inmutable (sha256, mime, magic-bytes). | Must | M | UC-013 |
| 9.4 | Previsualización de registros antes de confirmar. | Must | M | UC-013 |
| 9.5 | Crear `ImportBatch` con `PENDIENTE_VALIDACION`. | Must | P | UC-013 |
| 9.6 | Aprobar lote (Rector) → persistencia transaccional de Plans con `origin=IMPORTED` y metadatos (RN-074/076). | Must | G | UC-014 |
| 9.7 | Rechazar lote (Rector). | Must | P | UC-014 |
| 9.8 | Reversión de lote (no elimina, marca revertidos) (RN-077). | Should | M | UC-014 |
| 9.9 | Edición posterior de importados conservando historial (RN-078). | Must | M | UC-015 |
| 9.10 | Detección y reporte de duplicados (hash por docente/año/periodo/asignatura/grado). | Should | M | — |
| 9.11 | Tests con plantillas reales (`PLAN DE AULA 2026 ESPAÑOL.xlsx`, `…PENSAMIENTO SOCIAL.xlsx`). | Must | M | — |

### E10 — Reportes, auditoría y exportación *(RF-025, RF-080..RF-082)*

| HU | Resumen | Prioridad | Tam. | UC |
|---|---|---|---|---|
| 10.1 | KPIs institucionales (pendientes/aprobados/%/tiempo/cursos y grados pendientes) (RN-104). | Must | M | UC-016 |
| 10.2 | Gráficos: barras, circulares, línea de tiempo, KPI cards. | Must | M | UC-016 |
| 10.3 | Export PDF/Excel/CSV de Plan y reportes (RN-103). | Must | M | UC-017 |
| 10.4 | Visor de auditoría con diffs (RN-100/101/110). | Must | M | UC-018 |
| 10.5 | Visor de versiones/historial de Plan. | Must | M | UC-018 |

### E11 — Calidad (pruebas, accesibilidad, seguridad) *(§13 SRS, RNF-041, RNF-050)*

| HU/Task | Resumen | Prioridad | Tam. | RF |
|---|---|---|---|---|
| 11.1 | Plan de pruebas (unitarias/integración/funcionales) por módulo. | Must | M | §13 |
| 11.2 | Cobertura ≥ 80% en Dominio/Aplicación, ≥ 60% adaptadores (RN-130). | Must | M | RNF-041 |
| 11.3 | Pruebas de importación Excel con plantilla real. | Must | M | — |
| 11.4 | Pruebas de trazabilidad de registros importados + edición sin perder historial. | Must | M | — |
| 11.5 | Auditoría WCAG 2.1 AA y corrección de hallazgos críticos. | Must | M | RNF-050 |
| 11.6 | Revisión de seguridad (SAST + manual: XSS/CSRF/SQLi/SSRF upload). | Must | M | RNF-033..035 |
| 11.7 | Pruebas de carga (PI19: 5.000 docentes, 10.000 Planes) y P95. | Should | M | RNF-001/002/010 |
| 11.8 | Documentación técnica + manual de usuario. | Must | M | §14/13 |

### E12 — Operación y despliegue *(RNF-034/036/060/112/113)*

| HU/Task | Resumen | Prioridad | Tam. | RF |
|---|---|---|---|---|
| 12.1 | Configurar HTTPS + HSTS + WAF de borde. | Must | M | RN-112 |
| 12.2 | Backups automáticos + rutina de restore mensual con evidencia. | Must | M | RN-113 |
| 12.3 | Alertas de uso IA (presupuesto / latencia / errores). | Should | P | — |
| 12.4 | Dashboards operativos (latencia, errores, jobs de importación). | Should | M | RNF-060 |
| 12.5 | Plan de rollback para despliegues. | Must | P | — |
| 12.6 | Documento de administración y runbook. | Should | M | — |

---

## 3. Roadmap sugerido (prioridades)

> Sprints de 2 semanas. Es solo un plan inicial editable.

| Sprint | Entregables clave |
|---|---|
| 1 (Sprint 0) | E0:**fundaciones**, decisiones stack, repo, CI, migraciones base |
| 2 | E1:**acceso** + E2:**maestros** (malla + año lectivo + personas) |
| 3 | E3:**asignaciones** + E4:**plantilla default** |
| 4 | E5:**diligenciamiento v1** (creación, ediciones, autoguardado) |
| 5 | E5:**envío + estados** + E7 (evaluación v1) |
| 6 | E10 (PDF + versiones) + E2/E8 (permisos/dashboards básicos) |
| 7 | E6:**IA** (AIService + Inforge + acciones básicas + límites) |
| 8 | E9:**importación v1** (validación + carga + aprobación) |
| 9 | E9:**reversión + edición de importados** + E10 auditoría |
| 10 | E11:**calidad + accesibilidad** + E12**operación** |
| 11 | Hardening, WCSAA audit, pruebas de carga, documentación |
| 12 | UAT, fixes, release candidate |

---

## 4. Definition of Ready (DoR) — sugerencia

- HU con criterios *Given/When/Then* definidos.
- Trazabilidad a RF/UC identificada.
- Diseño UI/DDT referenciado (o nota de "sin UI crítica").
- Aceptación para "Must": al menos una prueba automática por criterio.
- Cumple RN inherentes (mencionadas explícitamente en la HU).
- Tareas técnicas estimadas (Tam. P/M/G/XL).

## 5. Definition of Done (DoD) — sugerencia

- Código revisado (peer review) y en main (con feature flags si requiere).
- Lint + tests pasando en CI.
- Cobertura exigida satisfecha (RN-130).
- Migraciones probadas en staging.
- Auditoría y logs estructura.
- Documentación técnica/usuario actualizada.
- Accesibilidad AA en pantallas nuevas.
- Seguridad: SAST sin hallazgos críticos; secretos en bóveda.
- If Import/IA: trazabilidad probada y límites configurados.

*Fin del documento.*
