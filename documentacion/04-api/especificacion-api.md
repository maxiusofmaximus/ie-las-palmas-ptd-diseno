# Especificación de API (REST/JSON)

**Sistema:** PTD — Planes de Trabajo Docentes (IE Las Palmas)
**Versión:** 1.0 · **Fecha:** Julio 2026
**SRS de referencia:** `01-requerimientos/SRS-IEEE-29148.md`

> API REST/JSON sobre HTTPS. Autenticación heredada (cookie/SSO del instituto + validación de claims por el adaptador `IdentityProvider`). Versionado URL: `/api/v1/**`. Codificación UTF-8. Idioma cabeceras `Accept-Language: es-CO`.

---

## 1. Convenciones

- **Verbos**: GET (lectura), POST (creación/acción), PUT/PATCH (reemplazo/edición parcial), DELETE (inactivar — borrado lógico).
- **Respuestas estándar**:
  - `200 OK` con payload.
  - `201 Created` con `Location` + payload.
  - `204 No Content`.
  - `400 Bad Request` — errores de validación; cuerpo `ProblemDetails` (RFC 7807) + `errors[]`.
  - `401 Unauthorized` — identidad inválida/expirada.
  - `403 Forbidden` — sin permiso *(RN-002)*.
  - `404 Not Found`.
  - `409 Conflict` (p. ej. transición de estado inválida).
  - `422 Unprocessable Entity` — entidad presente pero reglas de negocio fallidas.
  - `413 Payload Too Large`, `415 Unsupported Media Type`.
  - `429 Too Many Requests` (rate limit / `IA_QUOTA_EXCEEDED`).
  - `500..599` errores.
- **Codigos de negocio** en el campo `code` del `ProblemDetails`: *PTD-XXX*.
- **Paginación**: `?page=1&page_size=20`. Respuesta: `{ items, page, page_size, total }`.
- **Auditoría**: cada escritura genera entrada en `aud_log` *(RN-080/081)*.
- **Restricciones**: todos los endpoints requieren autenticación salvo `/healthz` y `/readyz`.

---

## 2. Identidad y sesión

| Método | Path | Rol | Descripción | Trazo RF |
|---|---|---|---|---|
| GET | `/me` | cualquiera | Devuelve usuario + roles + permisos efectivos. | RF-001, RF-003 |
| POST | `/logout` (proxy) | cualquiera | Pasa logout al proveedor institucional. | RF-001 |

`GET /me` → `200` `{ id, documento_redacted?, nombre, email, roles[], permisos[] }`

---

## 3. Administración de maestros (Institution)

### Años lectivos
| Método | Path | Rol |
|---|---|---|
| GET | `/academic-years` | cualquiera autenticado |
| POST | `/academic-years` | RECTOR |
| PATCH | `/academic-years/{id}` | RECTOR |
| POST | `/academic-years/{id}/activate` | RECTOR *(RN-010)* |
| DELETE | `/academic-years/{id}` | RECTOR (inactiva) |

Request POST `/academic-years`:
```json
{ "nombre": "2026", "fecha_inicio": "2026-01-19", "fecha_cierre": "2026-11-30", "activar": true }
```

### Periodos
| GET | `/academic-years/{id}/periods` |
| POST | `/academic-years/{id}/periods` | RECTOR/COORDINACION *(RN-011)* |
| PATCH | `/periods/{id}` |
| DELETE | `/periods/{id}` (inactiva) |

### Grados / cursos / áreas / nodos / asignaturas
| GET | `/grades` `/courses` `/areas` `/nodes` `/subjects` |
| POST/PATCH/DELETE | idem — RECTOR/ADMIN_SISTEMA *(RN-012, RN-013)* |

`POST /subjects`:
```json
{ "codigo": "LEN-601", "nombre": "Lengua Castellana", "area_id": "uuid|null", "node_id": "uuid|null" }
```
> Se valida: `area_id` o `node_id` presente *(RN-013)* → `422 PTD-013-NO_HIERARCHY`.

---

## 4. Personas, roles y permisos (IAM)

| Método | Path | Rol |
|---|---|---|
| GET | `/persons?rol=DOCENTE&q=` | RECTOR/COORDINACION |
| POST | `/persons` | RECTOR |
| PATCH | `/persons/{id}` | RECTOR |
| DELETE | `/persons/{id}` | RECTOR (inactiva) *(RN-102)* |
| PUT | `/persons/{id}/roles` | RECTOR *(RN-003)* |
| GET | `/roles` | RECTOR |
| GET | `/permissions` | RECTOR |

`POST /persons`:
```json
{ "documento": "1234567890", "nombre": "Carolina", "apellido": "Saldarriaga",
  "email": "csaldarriaga@ielaspamas.edu.co", "roles": ["DOCENTE"] }
```

---

## 5. Asignación docente (Assignment)

| Método | Path | Rol |
|---|---|---|
| GET | `/assignments?docente_id=&anio=&periodo=` | COORDINACION/RECTOR/DOCENTE *(siempre 自己)* |
| POST | `/assignments` | COORDINACION/RECTOR *(RN-022)* |
| PATCH | `/assignments/{id}` | COORDINACION/RECTOR |
| POST | `/assignments/{id}/dupla` | COORDINACION *(RN-021)* |
| DELETE | `/assignments/{id}` (inactiva) | COORDINACION/RECTOR |

`POST /assignments`:
```json
{
  "docente_id": "uuid",
  "anio_id": "uuid",
  "periodo_id": "uuid",
  "grado_id": "uuid",
  "asignatura_id": "uuid",
  "curso_ids": ["uuid","uuid"],
  "fecha_desde": "2026-01-19",
  "fecha_hasta": null,
  "dupla_con_docente_ids": []
}
```

`POST /assignments/{id}/dupla` → `{ "docente_id": "uuid" }` crea `asm_dupla`.

---

## 6. Plantilla del Plan (PlanTemplate)

| Método | Path | Rol |
|---|---|---|
| GET | `/plan-templates` | cualquiera autenticado |
| GET | `/plan-templates/active` | cualquiera |
| POST | `/plan-templates` | ADMIN_SISTEMA/RECTOR |
| POST | `/plan-templates/{id}/activate` | ADMIN_SISTEMA/RECTOR *(RN-031)* |
| PATCH | `/plan-templates/{id}/sections/{sectionId}` | ADMIN_SISTEMA *(RN-030)* |

`POST /plan-templates`:
```json
{ "version": "v2026.1",
  "sections": [
    { "clave": "CARACTERIZACION", "obligatoria": true, "activa": true },
    { "clave": "COMPETENCIAS", "obligatoria": true, "activa": true }
  ]}
```

PATCH `/plan-templates/{id}/sections/{sectionId}`:
```json
{ "obligatoria": false, "activa": false }
```

---

## 7. Plan de Aula (Plan) — núcleo

| Método | Path | Rol |
|---|---|---|
| GET | `/plans?anio=&periodo=&grado=&asignatura=&estado=&origin=&page=` | según jerarquía |
| GET | `/plans/{id}` | docente de Plan o Coordinación en jurisd. |
| POST | `/plans` | DOCENTE *(RN-020)* |
| PATCH | `/plans/{id}` | DOCENTE de Plan (estado BORRADOR/DEVUELTO) *(RN-042)* |
| POST | `/plans/{id}/send` | DOCENTE *(RN-041)* |
| POST | `/plans/{id}/duplicate` | DOCENTE *(convenience entre periodos)* |
| GET | `/plans/{id}/versions` | DOCENTE/COORDINACION/RECTOR *(RN-082)* |
| GET | `/plans/{id}/audit` | RECTOR/COORDINACION *(RN-081)* |
| GET | `/plans/{id}/pdf` | DOCENTE/COORDINACION/RECTOR *(RF-025)* |
| GET | `/plans/{id}/xlsx` | COORDINACION/RECTOR |
| GET | `/plans/{id}/csv` | COORDINACION/RECTOR *(RN-103)* |
| DELETE | `/plans/{id}` | RECTOR (inactiva, conserva historial) |

### `POST /plans`
```json
{
  "anio_id": "uuid",
  "periodo_id": "uuid",
  "asignatura_id": "uuid",
  "grado_id": "uuid",
  "curso_ids": ["uuid","uuid"],
  "docente_ids": ["self-uuid","dupla-uuid?"],
  "encabezado": {
    "docentes_texto": "Carolina Saldarriaga - Elizabeth Zuluaga",
    "nodo_area_asignatura": "Lengua Castellana - Comprensión lectora",
    "fecha_inicio": "2026-01-19",
    "fecha_cierre": "2026-04-13",
    "grado_texto": "6°1 - 6°2",
    "num_semanas": 13,
    "num_clases": 65
  }
}
```
`201 Created`:
```json
{ "id": "uuid", "estado": "BORRADOR", "origen": "MANUAL", "plantilla_version": "v2026.1", "_links": { "self": "/api/v1/plans/id" } }
```

### `PATCH /plans/{id}`
Cuerpo: cambios por secciones (similar al snapshot completo). La respuesta es el Plan actualizado con `version_plantilla` constante.

### `POST /plans/{id}/send`
Sin cuerpo. `200` con el Plan nuevo estado `ENVIADO`. Si fallan validaciones `RN-041#1..#4`:
`422 PTD-041-VALIDATE` con `errors`:
```json
{
  "type": "https://ptd.ilaspalmas/Error",
  "title": "Validación de Envío",
  "code": "PTD-041-VALIDATE",
  "errors": [
    { "field": "indicadores.porcentaje_total", "code": "PTD-041-INDICADORES_SUM", "message": "La suma de indicadores debe ser 100%" },
    { "field": "ejos.clases", "code": "PTD-041-CLASES_INCONSISTENT", "message": "Suma de clases de ejes excede num_clases" }
  ]
}
```

### Plan completo (GET)
```json
{
  "id": "uuid", "estado": "BORRADOR", "origen": "MANUAL",
  "anio_id": "...", "periodo_id": "...", "asignatura_id": "...", "area_id": "...", "nodo_id": null,
  "grado_id": "...", "curso_ids": ["...","..."], "docentes": ["uuid","uuid"],
  "plantilla_version": "v2026.1",
  "encabezado": { "docentes_texto": "...", "fecha_inicio": "2026-01-19", "fecha_cierre": "2026-04-13",
                  "grado_texto": "6°1 - 6°2", "num_semanas": 13, "num_clases": 65 },
  "metadatos": { "fecha_creacion_original": null, "fecha_carga": "...", "fecha_ultima_edicion": "...",
                 "usuario_carga_id": "...", "usuario_ultima_edicion_id": "...", "usuario_validacion_id": null },
  "secciones": {
    "CARACTERIZACION":     { "por_curso": { "curso_id_1": "texto", "curso_id_2": "texto" } },
    "COMPETENCIAS":        { "especificas": [ "...1", "...2" ], "socioemocionales": [ "...1", "...2", "...3" ] },
    "INDICADORES":         { "conceptuales":   { "porcentaje": 23, "items": ["..."] },
                              "procedimentales":{ "porcentaje": 23, "items": ["..."] },
                              "actitudinales": { "porcentaje": 54, "items": ["..."] } },
    "ACTIVIDADES":         { "ejes": [ {
        "numero": 1,
        "nombre": "Escritura creativa (Producción textual)",
        "act_inicio": "...", "act_nueva_info": "...", "act_finalizacion": "...",
        "evaluacion": "...", "criterios_valoracion": "...",
        "num_clases": 10,
        "seguimientos_por_curso": { "curso_id_1": "...", "curso_id_2": "..." },
        "pautas_dua": ["COMPROMISO_CAPTAR_INTERES","REPRESENTACION_PERCEPCION"]
    } ] },
    "SEGUIMIENTO_COORDINACION": { "registros": [ { "fecha":"2026-05-06","responsable_id":"...","observacion":"...","pautas":["..."] } ] },
    "SEGUIMIENTO_PSICOPEDAGOGICO": { "registros": [ { "fecha":"...","texto":"..." } ] }
  },
  "_links": { "self": "/api/v1/plans/.../x", "pdf": "/api/v1/plans/.../pdf", "versions": "/api/v1/plans/.../versions" }
}
```

---

## 8. Evaluación

| Método | Path | Rol |
|---|---|---|
| GET | `/plans/{id}/evaluations` | DOCENTE/COORDINACION/RECTOR |
| POST | `/plans/{id}/evaluations` | COORDINACION en jurisdicción *(RN-043, RN-044)* |
| GET | `/evaluations/{id}` | idem |

`POST /plans/{id}/evaluations`:
```json
{ "decision": "APROBAR", "calificacion": 92.5, "observacion": null, "pautas": ["COMPROMISO_CAPTAR_INTERES"] }
```
`200` devuelve la decisión + el nuevo `estado` del Plan. `422 PTD-044-OBS_REQUIRED` si `(RECHAZAR|DEVOLVER)` y `observacion` vacío.
`409 PTD-040-INVALID_TRANSITION` si el Plan no está en `ENVIADO`/`EN_REVISION`.

---

## 9. IA (Intelligence)

| Método | Path | Rol |
|---|---|---|
| GET | `/ai/configuration` | ADMIN_SISTEMA/RECTOR |
| PUT | `/ai/configuration` | ADMIN_SISTEMA/RECTOR *(RN-051)* |
| GET | `/ai/prompts` `/ai/prompts/{accion}` | ADMIN_SISTEMA |
| PUT | `/ai/prompts/{accion}` | ADMIN_SISTEMA |
| GET | `/ai/limits` | ADMIN_SISTEMA |
| PUT | `/ai/limits/{rol}` | ADMIN_SISTEMA |
| GET | `/ai/invocations?desde=&hasta=&usuario=` | RECTOR/ADMIN_SISTEMA *(RN-052)* |
| POST | `/ai/suggest` | DOCENTE con permiso `ia.use` *(RN-050,RN-053,RN-054)* |

`POST /ai/suggest`:
```json
{
  "plan_id": "uuid",
  "accion": "BORRADOR",
  "campo": "CARACTERIZACION",
  "curso_id": "uuid",
  "contexto_extra": "Eje 1: Escritura creativa"
}
```
`200`:
```json
{ "sugerencia": "Es un grupo que se caracteriza por...",
  "tokens_in": 320, "tokens_out": 410,
  "modelo": "...", "invocacion_id": 12345,
  "advertencias": [] }
```
`429 PTD-053-IA_QUOTA_EXCEEDED` con `Retry-After`.

`PUT /ai/configuration`:
```json
{ "proveedor": "inforge", "modelo": "...", "temperatura": 0.20, "tokens_max": 1024, "costo_por_token": 0.0000012 }
```

---

## 10. Importación (Excel)

| Método | Path | Rol |
|---|---|---|
| POST | `/imports/validate` | ADMIN_SISTEMA/RECTOR permiso `excel.import` *(RN-070,RN-072)* |
| POST | `/imports` | ADMIN_SISTEMA/RECTOR *(RN-074)* |
| GET | `/imports?page=&estado=` | RECTOR |
| GET | `/imports/{id}` | RECTOR/ADMIN_SISTEMA |
| POST | `/imports/{id}/approve` | RECTOR *(RN-076)* |
| POST | `/imports/{id}/reject` | RECTOR |
| POST | `/imports/{id}/revert` | RECTOR *(RN-077)* |
| GET | `/imports/{id}/report` | RECTOR/ADMIN_SISTEMA — JSON con `errors`, `warnings`, `summary` |
| GET | `/imports/{id}/report.csv` | RECTOR/ADMIN_SISTEMA |

### `POST /imports/validate`
`multipart/form-data`:
- `file`: `.xlsx`
- `anio_id`: uuid
- `periodo_id?`: uuid
`200`:
```json
{ "archivo": { "nombre": "PLAN DE AULA 2026 ESPAÑOL.xlsx", "tamano": 178211, "sha256": "..." },
  "hojas_detectadas": [ "FORMATO", "1°", "2°" ],
  "errores": [
     { "hoja":"1°", "fila": 14, "campo": "docentes", "code": "PTD-IMP-DOC_REQUIRED", "bloqueante": true }
  ],
  "advertencias": [
     { "hoja":"2°", "fila": 3, "code": "PTD-IMP-CLASES_MINOR", "mensaje": "N° clases planeadas < ejes" }
  ],
  "resumen": { "filas_total": 80, "aceptadas": 78, "omitidas": 2 } }
```
`415 PTD-IMP-MIMETYPE` si no es `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`.
`413 PTD-IMP-SIZE` si supera el máximo.

### `POST /imports`
Idéntico `multipart`. Crea un **`ImportBatch`** en `PENDIENTE_VALIDACION` con registros pre-validados, **sin persistir Plans** todavía (los Plans se persisten transaccionalmente **al aprobar**).

### `POST /imports/{id}/approve`
`Rector` valida. Crea los `Plan` con `origin=IMPORTED` y metadatos de auditoría *(RN-076)* en una **transacción (UoW)**. `200` con `{ plan_ids: [...], plan_count: N, estado: "VALIDADO" }`. En caso de fallo técnico rever¬ y el batch queda `RECHAZADO` *(RN-074)*.

### `POST /imports/{id}/revert`
Marca batch `REVERTIDO`. Los Planes creados por el lote pasan a estado `REVERTIDO_*` (inactivos productivos, visibles en auditoría) — **no se eliminan** *(RN-077)*.

---

## 11. Reportes y dashboards

| Método | Path | Rol |
|---|---|---|
| GET | `/reports/kpis?anio=&periodo=` | RECTOR *(RF-041)* |
| GET | `/reports/charts/bars?by=estado&...` | RECTOR/COORDINACION |
| GET | `/reports/charts/timeline?...` | RECTOR |
| GET | `/reports/charts/pie?by=grado&...` | RECTOR |
| GET | `/reports/exports/plans.xlsx?filters=` | COORDINACION/RECTOR *(RF-080)* |
| GET | `/reports/exports/plans.csv?filters=` | COORDINACION/RECTOR *(RN-103)* |

`GET /reports/kpis`:
```json
{ "anio_id": "...", "periodo_id": "...",
  "kpis": {
    "DOCENTES_PENDIENTES": 12,
    "DOCENTES_APROBADOS": 37,
    "PORCENTAJE_DILIGENCIADO": 76.4,
    "TIEMPO_PROMEDIO_DILIGENCIAMIENTO_HS": 4.2,
    "CURSOS_PENDIENTES": 8,
    "GRADOS_PENDIENTES": 2
  },
  "calculo_version_hash": "sha1:...",
  "_links": { "csv": "/api/v1/reports/exports/kpis.csv?..." }
}
```

---

## 12. Notificaciones

| Método | Path | Rol |
|---|---|---|
| GET | `/notifications/inbox` | cualquiera |
| PATCH | `/notifications/inbox/{id}/read` | cualquiera |
| GET | `/admin/notifications/templates` | ADMIN_SISTEMA/RECTOR |
| PUT | `/admin/notifications/templates/{evento}` | ADMIN_SISTEMA |

Eventos (RN-090): `OPEN_DILIGENCIAMIENTO`, `RECORDATORIO`, `RETRASO`, `LLAMADO_ATENCION`, `APROBADO`, `RECHAZADO`, `DEVUELTO`.

---

## 13. Auditoría

| Método | Path | Rol |
|---|---|---|
| GET | `/audit?entidad=&entidad_id=&usuario=&desde=&hasta=&page=` | RECTOR *(RN-081)* |
| GET | `/audit/{id}` | RECTOR |

`GET /audit/{id}`:
```json
{ "id": 1234, "correlacion_id": "uuid", "fecha": "...",
  "usuario": { "id": "...", "nombre": "Carolina Saldarriaga" },
  "tipo_operacion": "UPDATE",
  "tipo_entidad": "Plan", "entidad_id": "uuid",
  "diff": { "antes": { "estado": "BORRADOR" }, "después": { "estado": "ENVIADO" } },
  "ip_origen": "190.x.x.x", "user_agent": "..." }
```

---

## 14. Errores y códigos (resumen)

| Código | Origen | Significado |
|---|---|---|
| `PTD-001-AUTH_INVALID` | IAM | Identidad no valida |
| `PTD-002-FORBIDDEN` | IAM | Permiso/faltante |
| `PTD-010-ANIO_UNICO_ACTIVO` | Institution | RN-010 |
| `PTD-011-PERIODO_OVERLAP` | Institution | RN-011 |
| `PTD-013-NO_HIERARCHY` | Institution | RN-013 |
| `PTD-020-NO_ASIGNACION` | Assignment | RN-020 |
| `PTD-021-DUPLA_GRADO_INVALIDO` | Assignment | RN-021 |
| `PTD-022-ROL_NO_AUTORIZADO` | Assignment | RN-022 |
| `PTD-030-SECCION_BASE_NO_OCULTABLE` | PlanTemplate | RN-030 |
| `PTD-031-VERSION_LIGADA` | PlanTemplate | RN-031 |
| `PTD-040-INVALID_TRANSITION` | Plan | RN-040 |
| `PTD-041-VALIDATE` | Plan | RN-041 |
| `PTD-041-INDICADORES_SUM` | Plan | RN-041#2 |
| `PTD-041-CLASES_INCONSISTENT` | Plan | RN-041#3 |
| `PTD-042-NOT_EDITABLE` | Plan | RN-042 |
| `PTD-043-FUERA_JURISDICCION` | Evaluation | RN-043 |
| `PTD-044-OBS_REQUIRED` | Evaluation | RN-044 |
| `PTD-050-IA_NOT_ACCEPTED_DIRECT` | Intelligence | RN-050 (solo sistema) |
| `PTD-053-IA_QUOTA_EXCEEDED` | Intelligence | RN-053 |
| `PTD-054-PII_IN_PROMPT` | Intelligence | RN-054 |
| `PTD-070-IMPORT_NO_PERM` | Import | RN-070 |
| `PTD-071-IMP_BLOCKING_ERRORS` | Import | RN-071 |
| `PTD-072-IMP_MIMETYPE` | Import | RN-072 |
| `PTD-073-IMP_NO_PERIOD_SHEETS` | Import | RN-073 |
| `PTD-074-IMP_TXN_ROLLBACK` | Import | RN-074 |
| `PTD-077-IMP_ALREADY_REVERTED` | Import | RN-077 |
| `PTD-100-AUDIT_IMMUTABLE` | Audit | RN-100 |

---

## 15. Headers comunes

- `Authorization` heredado del instituto (cookie firmada o Bearer) — el adaptador lo interpreta.
- `Accept-Language: es-CO` (default).
- `X-Correlation-Id` propagado en logs/auditoría.
- `Idempotency-Key` on POST `/imports`, `/notifications enviar`, etc. (respetado por RN-091).

---

## 16. Trazabilidad API ↔ RF/UC

| Path | RF | UC |
|---|---|---|
| `/me`, auth | RF-001/002/003 | UC-001 |
| `/persons/roles` | RF-010, RF-040 | UC-002 |
| `/grades/*` `/courses/*` `/areas/*` `/nodes/*` `/subjects/*` | RF-012 | UC-003 |
| `/academic-years` `/periods` | RF-011 | UC-004 |
| `/assignments` | RF-013, RF-023 | UC-005 |
| `/plan-templates` | RF-014, RF-015 | UC-006 |
| `/plans` `POST/PATCH` | RF-020..022 | UC-007 |
| `/plans/{id}/send` | RF-022, RF-060 | UC-008 |
| `/ai/suggest` | RF-024, RF-050..053 | UC-009 |
| `/plans/{id}/pdf` | RF-025 | UC-010 |
| `/evaluations` | RF-031, RF-032 | UC-011 |
| `/imports/*` | RF-070..076 | UC-013/014 |
| `/reports/kpis/charts/exports` | RF-041, RF-080 | UC-016/017 |
| `/audit` | RF-081, RF-082 | UC-018 |

---

## 17. OpenAPI

El contrato completo se generará como `openapi.yaml` en una iteración siguiente; servirá para clientes SDK, mocks de prueba y validación (p. ej. `schema-based validator`). Esta especificación define los *endpoints*, *shapes* y *codes* contratuales.

*Fin del documento.*
