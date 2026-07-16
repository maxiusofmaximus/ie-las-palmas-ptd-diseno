# Reglas de Negocio (Business Rules)

**Sistema:** PTD — Planes de Trabajo Docentes (IE Las Palmas)
**Versión:** 1.0 · **Fecha:** Julio 2026

> Conjunto de reglas identificables por código **RN-###** y trazables a RF/HU. Cada regla incluye: enunciado, motivo, severidad (*Bloqueante*, *Error*, *Advertencia*), trazabilidad y método de verificación.

---

## 1. Identidad y acceso

**RN-001 — Sin doble autenticación**
Un usuario autenticado en el portal institucional NO debe autenticarse de nuevo en PTD.
- Severidad: Bloqueante · RF: RF-001, RF-002 · Verif: T.

**RN-002 — Denegación por defecto**
Toda funcionalidad no explícitamente autorizada para el rol/permiso del usuario se deniega. El log de denegación se audita.
- Severidad: Bloqueante · RF: RF-003 · Verif: T.

**RN-003 — Roles permitidos**
Los únicos roles de PTD son: `RECTOR`, `COORDINACION`, `DOCENTE`, `APOYO_PSICOPEDAGOGICO`, `ADMIN_SISTEMA`. Un usuario puede tener varios roles; la unión de permisos se aplica.
- Severidad: Error · RF: RF-003, RF-010 · Verif: A.

---

## 2. Maestros institucionales

**RN-010 — Año lectivo único activo**
A lo sumo un año lectivo puede estar `ACTIVO` simultáneamente. Los Planes nuevos solo pueden crearse en años activos.
- Severidad: Error · RF: RF-011 · Verif: T.

**RN-011 — Periodos ordenados**
Los periodos `(1°..n°)` pertenecen a un año lectivo, tienen fecha inicio/cierre, son consecutivos y no se solapan.
- Severidad: Error · RF: RF-011 · Verif: T.

**RN-012 — Curso dentro de grado**
Un `Curso` siempre pertenece a un `Grado`. El `Grado` pertenece al modelo educativo de la institución.
- Severidad: Bloqueante · RF: RF-012 · Verif: A.

**RN-013 — Jerarquía curricular dual**
`Asignatura` pertenece a un `Área` (modelo secundaria) y/o a un `Nodo` (modelo primaria). Al menos una de las dos afirmaciones debe cumplirse.
- Severidad: Error · RF: RF-012 · Verif: T.

---

## 3. Asignación docente

**RN-020 — Solo asignado accede**
Un Docente solo puede crear/editar Planes para las asignaciones `DocenteAsignacion` vigentes.
- Severidad: Bloqueante · RF: RF-013, RF-023 · Verif: T.

**RN-021 — Dupla docente en mismo grado**
Solo se admite el modo *dupla* entre docentes que comparten el mismo `Grado` + `Asignatura` + `Periodo`. Ambos figuran como autores y ambos pueden editar el mismo Plan.
- Severidad: Error · RF: RF-013 · Verif: T.

**RN-022 — Asignación por Coordinación**
Solo el rol `COORDINACION` (o `RECTOR`) puede crear/modificar `DocenteAsignacion`.
- Severidad: Bloqueante · RF: RF-013 · Verif: T.

---

## 4. Plantilla del Plan de Aula

**RN-030 — Estructura uniforme**
Todos los Planes de un mismo año lectivo usan la **misma** plantilla activa. La estructura base de §10 del SRS no puede suprimirse vía configuración (sí puede ocultarse con justificación, conservando la sección como inactiva).
- Severidad: Error · RF: RF-014, RF-021 · Verif: A.

**RN-031 — Versionado no retroactivo**
Un Plan creado con la versión *v* del template mantiene su forma aunque se publique *v+1*. La edición del Plan usa siempre la versión ligada al momento de su creación.
- Severidad: Error · RF: RF-015 · Verif: T.

**RN-032 — Sección obligatoria**
Las secciones marcadas como obligatorias en la plantilla deben estar completas antes de *Enviar*.
- Severidad: Error · RF: RF-021, RF-022 · Verif: T.

---

## 5. Ciclo de vida del Plan de Aula

**RN-040 — Estados válidos**
Estados permitidos: `BORRADOR`, `ENVIADO`, `EN_REVISION` (opcional), `APROBADO`, `RECHAZADO`, `DEVUELTO`. Transiciones válidas:

```
BORRADOR --enviar--> ENVIADO
ENVIADO --abrir revision--> EN_REVISION | APROBADO | RECHAZADO | DEVUELTO
DEVUELTO --editar/enviar--> ENVIADO
APROBADO --editar--> (requiere reenvío: BORRADOR o DEVUELTO según param config)
RECHAZADO --editar/enviar--> ENVIADO
```
- Severidad: Bloqueante · RF: RF-022, RF-031 · Verif: T.

**RN-041 — Envío requiere validaciones locales**
Antes de `ENVIADO`, el sistema valida:
1. Campos obligatorios del encabezado completos.
2. Indicadores (conceptual/procedimental/actitudinal) suman 100 % (o valor configurable por plantilla).
3. `N° de Clases` declarado ≥ suma de clases de los Ejes (advertencia si menor).
4. Docente(s) pertenece(n) a la asignación del Plan.
- Severidad: Error · RF: RF-022 · Verif: T.

**RN-042 — Plan enviado no editable por docente**
En `ENVIADO`, el docente no puede editar. Coordinación puede abrir revisión, calificar y devolver.
- Severidad: Bloqueante · RF: RF-022, RF-031 · Verif: T.

**RN-043 — Solo coordinación con jurisdicción**
Coordinación solo puede calificar/devolver/aprobar/rechazar Planes dentro de su jurisdicción (grado/asignatura configurados en su rol).
- Severidad: Bloqueante · RF: RF-031 · Verif: T.

**RN-044 — Observación obligatoria si devolver/rechazar**
`DEVUELTO` y `RECHAZADO` requieren un texto de observación.
- Severidad: Error · RF: RF-031 · Verif: T.

---

## 6. Asistencia IA

**RN-050 — IA es asistente, no autor**
La IA nunca persiste texto en el Plan por sí misma. El docente acepta/edita/rechaza la sugerencia; el texto aceptado se almacena con `origin_text = "AI_ASSISTED"` (autoría del campo sigue siendo el docente).
- Severidad: Bloqueante · RF: RF-024, RF-050 · Verif: A/T.

**RN-051 — Capa de abstracción obligatoria**
Ningún componente de Dominio/Aplicación invoca un proveedor de IA concreto. Toda invocación pasa por `AIService` (puerto).
- Severidad: Bloqueante · RF: RF-052 · Verif: A.

**RN-052 — Trazabilidad IA**
Cada invocación registra: usuario, timestamp, modelo, prompt (hash o plantilla sustituida → sin PII en log), tokens_in, tokens_out, costo, latencia, campo destino, resultado (aceptado/editado/rechazado).
- Severidad: Bloqueante · RF: RF-053, RNF-061 · Verif: T.

**RN-053 — Límites IA**
Se respetan límites por usuario/día y por rol configurados desde administración. Superado el límite, la invocación se rechaza con código `IA_QUOTA_EXCEEDED` y mensaje amable.
- Severidad: Error · RF: RF-051 · Verif: T.

**RN-054 — Privacidad de PII**
La información sensible del estudiante (nombres, documentos) NO se envía a la IA. El *context provider* anonimiza/agrega antes del prompt.
- Severidad: Bloqueante · RC-001 · Verif: A.

**RN-055 — Cambio de proveedor aislado**
Cambiar de proveedor/modelo en `AIService` no requiere modificar Dominio/Aplicación. Test de equivalencia de contrato garantiza la continuidad.
- Severidad: Error · RF: RF-052 · Verif: T.

---

## 7. Importación Excel / históricos

**RN-070 — Solo roles autorizados**
Solo `ADMIN_SISTEMA` y `RECTOR` con permiso `excel.import` pueden iniciar una carga.
- Severidad: Bloqueante · RF: RF-023, RF-070, RNF-036 · Verif: T.

**RN-071 — Validación previa indispensable**
Ninguna fila se persiste hasta que el *Reporte de validación* no tenga cero errores de bloqueo. Advertencias se permiten pero se registran en el batch.
- Severidad: Bloqueante · RF: RF-071 · Verif: T.

**RN-072 — Tipos de archivo aceptados**
Solo `.xlsx` (OpenXML). Validation de mime + magic bytes + tamaño máximo configurado. Se rechaza cualquier otra extensión/mime.
- Severidad: Bloqueante · RNF-035 · Verif: T.

**RN-073 — Hoja `FORMATO` opcional, periodos obligatorios**
El archivo puede traer una hoja `FORMATO` con orientaciones (se ignora para la carga) y al menos una hoja `1°..n°`. Si no se detectan periodos, lote = `RECHAZADO` con motivo `NO_PERIOD_SHEETS`.
- Severidad: Error · RF: RF-070 · Verif: T.

**RN-074 — Transaccionalidad del batch**
La persistencia de un batch es transaccional (UoW). Un error de bloqueo durante la fase de persistencia revierte el lote completo y el batch queda `RECHAZADO` con motivo técnico.
- Severidad: Bloqueante · RF: RF-072 · Verif: T.

**RN-075 — Origen grabado**
Todo registro creado por importación se persiste con `origin = IMPORTED` y `importBatchId` referenciado. Los creados manualmente: `origin = MANUAL`.
- Severidad: Bloqueante · RF: RF-074 · Verif: T.

**RN-076 — Trazabilidad completa**
Siempre se conservan: `fecha_creacion_original` (si viene del Excel), `fecha_carga` (batch), `fecha_ultima_edicion`, `usuario_carga`, `usuario_ultima_edicion`, `usuario_validacion` (batch aprobador). Estos campos son inmutables salvo `fecha_ultima_edicion` y `usuario_ultima_edicion` (actualizados en cada edición).
- Severidad: Bloqueante · RF: RF-073 · Verif: T.

**RN-077 — Reversión no elimina**
Revertir un batch marcó los registros como `revertido` (estado interno) y los excluye de consultas productivas, pero **no** los elimina. Se conserva auditoría completa del intento y su reversión.
- Severidad: Bloqueante · RF: RF-076 · Verif: T.

**RN-078 — Edición de importados preserva historial**
Cualquier edición posterior de un registro importado añade una nueva versión en el historial; el `origin` y `importBatchId` originales son inmutables.
- Severidad: Bloqueante · RF: RF-075, RF-082 · Verif: T.

---

## 8. Notificaciones

**RN-090 — Eventos de correo**
Eventos que producen correo: `OPEN_DILIGENCIAMIENTO`, `RECORDATORIO`, `RETRASO`, `LLAMADO_ATENCION`, `APROBADO`, `RECHAZADO`, `DEVUELTO`. Plantillas y destinatarios configurables.
- Severidad: Error · RF: RF-060 · Verif: T.

**RN-091 — Idempotencia de envío**
El disparo de un mismo evento al mismo destinatario en una ventana configurable (p. ej. 1 h) no se repite salvo `force = true` (uso admin/a posterior).
- Severidad: Advertencia · RF: RF-060 · Verif: T.

**RN-092 — No externaliza PII en asunto**
El asunto (y la cabecera visible) del correo no contiene PII sensible del Plan/estudiante.
- Severidad: Error · RC-001 · Verif: A.

---

## 9. Reportes, auditoría y exportación

**RN-100 — Auditoría inmutable**
Los registros de auditoría **solo se anexan**. Nunca se actualizan ni se eliminan salvo por una política de retención explícita.
- Severidad: Bloqueante · RF: RF-081, RC-003 · Verif: A.

**RN-101 — Diff antes/después**
Para entidades de dominio, la auditoría almacena un `diff` JSON (campo + valor anterior + valor nuevo) salvo campos sensibles (hashes/redactados).
- Severidad: Error · RF: RF-081 · Verif: T.

**RN-102 — Borrado lógico**
Las entidades maestras y de dominio se marcan como inactivas; no se borran físicamente. Las auditoría y los historiales se conservan.
- Severidad: Bloqueante · RD-003 · Verif: A.

**RN-103 — Exportación en formatos abiertos**
Todo reporte con más de 1000 filas también se ofrece en **CSV** además del formato elegido (PDF/Excel) para garantizar interoperabilidad abierta.
- Severidad: Error · RC-002, RF-080 · Verif: A.

**RN-104 — Indicadores reproducibles**
Los KPIs expuestos en dashboards se calculan con consulta reproducible (lógica documentada) y exportable en CSV subyacente al gráfico.
- Severidad: Error · RF: RF-041 · Verif: T.

---

## 10. Seguridad y compliance

**RN-110 — PII fuera de logs**
En logs/auditoría se aplica redacción sobre PII sensible (número de documento, dirección, diagnóstico psicopedagógico).
- Severidad: Bloqueante · RC-001, RNF-037 · Verif: A.

**RN-111 — Secretos en bóveda**
Claves de proveedores IA, SMTP, DB, SSO se cargan desde bóveda / variables secretas inyectadas en runtime **nunca** en repositorio.
- Severidad: Bloqueante · RNF-037 · Verif: A.

**RN-112 — HTTPS y HSTS**
Todo el tráfico va por HTTPS; `Strict-Transport-Security` activo. Cookies marcadas `Secure`, `HttpOnly`, `SameSite=Lax` ( CSRF token explícito para POST/PUT/DELETE).
- Severidad: Bloqueante · RNF-033 · Verif: A.

**RN-113 — Backups validados**
Se ejecuta backup automático diario y una restauración de prueba mensual con evidencia archivada.
- Severidad: Error · RNF-034 · Verif: A.

---

## 11. Datos operativos vs IA

**RN-120 — Separación de almacenes**
Datos operativos y administrativos viven en la **base relacional transaccional**. Datos vectoriales/embeddings (cuando aplique RAG u otros) viven en **Inforge.dev** accedidos únicamente por el adaptador de `AIService`. Dominio/Aplicación no conocen el almacén vectorial.
- Severidad: Bloqueante · RD-001, RF-053 · Verif: A.

---

## 12. Mantenibilidad / evolución

**RN-130 — Cobertura mínima**
Cobertura de tests: ≥ 80 % en Dominio/Aplicación; ≥ 60 % en Adaptadores/Infraestructura.
- Severidad: Error · RNF-041 · Verif: A.

**RN-131 — Preparado para nuevos módulos**
La arquitectura (módulos verticales, APIs en módulos propios, shared kernel mínimo) permite incorporar Observador, Convivencia, Notas, Planeación, Evaluaciones, Horarios, Asistencia, Biblioteca, Psicoorientación **sin reescribir el core**.
- Severidad: Error · RNF-011 · Verif: A.

---

## Matriz de severidades

| Severidad | Significado |
|---|---|
| Bloqueante | No se acepta la operación; el sistema rechaza con error. |
| Error | Se admite completar la operación, pero genera alerta/incidencia y no se considera conforme. |
| Advertencia | Es flexible; se registra y reporta, pero no bloquea. |

*Fin del documento.*
