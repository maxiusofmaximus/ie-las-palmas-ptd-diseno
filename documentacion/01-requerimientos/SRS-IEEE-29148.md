# Especificación de Requisitos de Software (SRS)

## Sistema Web de Gestión de Planes de Trabajo Docentes — IE Las Palmas (Envigado)

**Norma de referencia:** IEEE Std 29148-2018 (Systems and software engineering — Life cycle processes — Requirements engineering)
**Versión del SRS:** 1.0
**Estado:** Aprobado para implementación
**Fecha:** Julio de 2026
**Elaborado por:** Equipo de Análisis — Proyecto Codex IE Las Palmas
**Basado en:** *Documento de Análisis de Requerimientos de Software v1.0* y análisis de las plantillas reales `PLAN DE AULA 2026 ESPAÑOL.xlsx` / `PLAN DE AULA 2026 PENSAMIENTO SOCIAL.xlsx`

> Documentos relacionados en este repositorio:
> - Casos de uso e historias de usuario: `01-requerimientos/casos-de-uso-e-historias.md`
> - Reglas de negocio: `01-requerimientos/reglas-de-negocio.md`
> - Modelo de dominio: `02-arquitectura/modelo-de-dominio.md`
> - Diagrama de arquitectura: `02-arquitectura/diagrama-de-arquitectura.md`
> - Esquema de base de datos: `03-datos/esquema-base-de-datos.md`
> - Especificación de API: `04-api/especificacion-api.md`
> - Backlog inicial: `05-backlog/backlog-inicial.md`

---

## Tabla de contenido

1. Propósito
2. Alcance
3. Definiciones, acrónimos y abreviaturas
4. Referencias
5. Visión general del documento
6. Descripción general (IEEE §3)
   6.1 Perspectiva del producto
   6.2 Funciones del producto
   6.3 Características de los usuarios
   6.4 Restricciones
   6.5 Suposiciones y dependencias
7. Requisitos (IEEE §4)
   7.1 Requisitos funcionales
   7.2 Requisitos de idoneidad (capacidad)
   7.3 Requisitos no funcionales
   7.4 Requisitos de cumplimiento
   7.5 Requisitos de datos
   7.6 Requisitos de interfaces externas
8. Requisitos de verificabilidad y aceptación
9. Trazabilidad con el documento de Análisis de Requerimientos
10. Glosario funcional y estructura real del Plan de Aula

---

## 1. Propósito

Este SRS define, de forma completa, no ambigua, verificable y trazable, los requisitos del **Módulo de Planes de Trabajo Docentes** que se incorporará al dominio institucional de la IE Las Palmas. Sirve como base contractual y técnica para que la herramienta de implementación (Codex) genere el código, los esquemas y las pruebas con la menor ambigüedad posible, sustituyendo y ampliando al *Documento de Análisis de Requerimientos v1.0*.

## 2. Alcance

El sistema es un **módulo web** (de aquí en más, *el Módulo* o *PTD — Planes de Trabajo Docentes*) que se integra al sitio institucional existente sin reemplazarlo. Reutiliza la autenticación vigente, agrega una capa de datos transaccional, una capa de IA asistiva y un subsistema de importación desde Excel para migración de históricos.

**Dentro del alcance:**
- Diligenciamiento, edición, envío, evaluación y consulta de Planes de Aula.
- Administración de usuarios, roles, asignaciones (docente/curso/grado/área/asignatura/periodo), parámetros y plantillas.
- Evaluación y seguimiento por parte de Coordinación y Rectoría.
- Dashboards, indicadores y exportación (PDF/Excel/CSV).
- Asistente de IA configurable y desacoplado (capa `AIService`).
- Importación desde Excel con validación, transformación y trazabilidad.
- Integración con la página institucional vía un botón/redirección.

**Fuera del alcance:**
- Reemplazo del sitio institucional.
- Cambio del sistema de autenticación.
- Migración completa del portal.
- Otros módulos futuros (Observador, Convivencia, Notas, etc.) — solo se deja la arquitectura preparada.

## 3. Definiciones, acrónimos y abreviaturas

| Término | Definición |
|---|---|
| **PTD** | Planes de Trabajo Docentes (el Módulo objeto de este SRS). |
| **Plan de Aula** | Documento pedagógico por docente, grado, asignatura y periodo. Equivale al "Plan de Trabajo" en el documento de análisis. |
| **Nodo / Área / Asignatura** | Jerarquía curricular. Un *Nodo* agrupa asignaturas (modelo de primaria); un *Área* agrupa asignaturas (modelo de secundaria). El sistema debe soportar ambos. |
| **Periodo académico** | Fracción del año lectivo (en las plantillas se observan hasta 11 periodos numerados "1°"…"11°"). |
| **DUA** | Diseño Universal para el Aprendizaje. Estructura fija del Plan de Aula con principios: *Representación*, *Acción y Expresión*, *Compromiso*. |
| **Coord.** | Coordinación Académica. |
| **IA** | Inteligencia artificial. En este SRS, asistente de escritura. |
| **AIService** | Capa de abstracción de IA (puerto/interfaz) para desacoplar proveedores. |
| **Inforge.dev** | Infraestructura indicada en el documento de análisis para IA y servicios asociados, administrada desde su CLI oficial. Se accede a través de `AIService`. |
| **RAG** | Retrieval-Augmented Generation. |
| **RF / RNF / RC / RD / RI** | Requisito Funcional / No Funcional / Cumplimiento / Datos / Interfaz. |
| **SRS** | Software Requirements Specification (esta, según IEEE 29148). |
| **KPI** | Indicador clave de desempeño. |
| **UoW** | Unit of Work (patrón transaccional). |
| **CQRS / DDD / SOLID** | Principios solicitados por el área de Arquitectura (ver §6.4). |

## 4. Referencias

1. IEEE Std 29148-2018 — *Systems and software engineering — Life cycle processes — Requirements engineering*.
2. ISO/IEC 25010:2011 — *Systems and software Quality Requirements and Evaluation (SQuaRE)*.
3. NIST SP 800-53 rev.5 — controles de seguridad aplicables a logs/auditoría.
4. OWASP Top 10 (vigente) — prevención de XSS, CSRF, SQLi.
5. Documento de Análisis de Requerimientos de Software v1.0 (entrada del proyecto).
6. Plantillas institucionales reales: `PLAN DE AULA 2026 ESPAÑOL.xlsx`, `PLAN DE AULA 2026 PENSAMIENTO SOCIAL.xlsx` (estudio de estructura, ver §10).

## 5. Visión general del documento

Este SRS sigue la estructura sugerida por IEEE 29148. **§6** describe el producto; **§7** contiene los requisitos (identificados de forma única y verificable); **§8** define criterios de aceptación; **§9** mapea cada requisito con el *Documento de Análisis de Requerimientos v1.0*; **§10** documenta la estructura real del Plan de Aula observada en las plantillas, que es la base del modelo de datos.

Los requisitos se numeran con prefijo de categoría y correlativo (p. ej. `RF-001`). Cuando un requisito proviene del documento de análisis, se referencia como `(origen: RF-001 del ARS)` en la trazabilidad de §9.

---

## 6. Descripción general

### 6.1 Perspectiva del producto

El Módulo es un **subproducto** acoplado al sitio institucional por *autenticación compartida* (SSO o reutilización del proveedor de identidad existente) y por *dominio compartido* (mismo hostname o subdominio), pero **desplegado y versionado de forma independiente**. La comunicación con el portal existente se limita a:
- Lectura de la identidad del usuario autenticado (claims).
- Un enlace de entrada (botón) en el portal hacia el Módulo.
- (Opcional) endpoint de disco/SSO para iniciar sesión sin revalidación.

El Módulo expone un **API REST/JSON backend** y un **frontend SPA** (administrable y modular). La persistencia se divide en **(a) base relacional transaccional** y **(b) capacidades de IA/vecctorial provistas por Inforge.dev, accedidas a través de `AIService`.

Diagrama de bloques: ver `02-arquitectura/diagrama-de-arquitectura.md`.

### 6.2 Funciones del producto

Agrupadas por *módulo funcional* (cada macrofunción tiene sus propios requisitos funcionales en §7.1):

1. **Identidad y acceso** — reutilizar autenticación institucional; control de acceso por rol/permiso.
2. **Administración** — gestión de entidades maestras (docentes, coordinadores, rector, cursos, grados, áreas, nodos, asignaturas, periodos, años lectivos), plantillas configurables del Plan de Aula, permisos, configuración de IA, notificaciones, parámetros generales.
3. **Docente** — crear/editar/guardar/enviar/consultar Planes de Aula; uso de IA; descarga de PDF.
4. **Coordinación** — revisar, calificar, aprobar/rechazar, observar, devolver, generar reportes.
5. **Rectoría** — administración de permisos, estadísticas, indicadores, reportes institucionales, parámetros y validación de históricos.
6. **IA** — asistente de escritura configurable, desacoplado por `AIService`.
7. **Importación Excel** — carga, validación estructural, transformación, persistencia y trazabilidad de históricos.
8. **Reportes y dashboards** — KPIs, gráficos, exportación PDF/Excel/CSV.
9. **Auditoría y trazabilidad** — historial de cambios, identificación de origen (manual vs importado), conservación de auditoría.

### 6.3 Características de los usuarios

| Actor | Perfil | Necesidades clave |
|---|---|---|
| **Rector** | Acceso total. Autoridad institucional. | Gobernanza, indicadores globales, parámetros, validación de históricos. |
| **Coordinación Académica** | Supervisor operativo. | Calificar/devolver/aprobar planes, asignar cursos, observaciones, reportes. |
| **Docente** | Autor del Plan de Aula. | Diligenciar rápido, recibir asistencia IA, ver observaciones, descargar PDF. |
| **Apoyo Psicopedagógico** | (Implícito en DUA) | Registro del seguimiento psicopedagógico en el Plan. |
| **Administrador del sistema** | (Rector o delegado) | Carga de Excel, configuración IA, permisos, parámetros. |

### 6.4 Restricciones

- **C1** El sistema **no** debe reemplazar la autenticación institucional; debe **integrarse** con ella.
- **C2** El dominio de despliegue pertenece al instituto (mismo host o subdominio).
- **C3** HTTPS obligatorio en todo el flujo.
- **C4** Arquitectura conforme a SOLID, DRY, KISS, YAGNI, Clean Architecture, Hexagonal (Ports & Adapters), DDD donde aporte valor, Repository Pattern, Unit of Work, DI, CQRS cuando sea necesario, programación orientada a interfaces, servicios desacoplados, alta cohesión y bajo acoplamiento.
- **C5** La IA se accede **siempre** por `AIService`; se prohíbe invocar a un proveedor concreto desde el dominio/aplicación.
- **C6** La infraestructura de IA es Inforge.dev, administrada desde su CLI oficial.
- **C7** Los datos operativos y los datos de IA **deben** mantenerse separados en su almacenamiento.
- **C8** La estructura del formulario del Plan de Aula se mantiene idéntica para todos los grados/cursos; los campos son **configurables** desde administración.
- **C9** La importación masiva solo la ejecutan roles autorizados (Rector o Administrador del sistema).
- **C10** Trazabilidad obligatoria de todo cambio (auditoría de escritura).

### 6.5 Suposiciones y dependencias

- **S1** El sistema institucional de autenticación expone identidad/claims consumibles (token/JWT/cookie firmada). En caso contrario, se define un conector adaptador con su dueño.
- **S2** Los archivos Excel históricos respetan alguna de las plantillas conocidas (FORMATO + periodos "1°".."n°"). El sistema debe tolerar variaciones menores y reportar diferencias.
- **S3** Inforge.dev está disponible y aprovisionado con claves/permisos para producción.
- **S4** El año lectivo y la malla curricular (grados/áreas/asignaturas) se cargan como datos maestros antes de la importación/explotación.
- **S5** Se dispone de un canal de correo SMTP institucional para notificaciones (o servicio equivalente).

---

## 7. Requisitos

### 7.1 Requisitos funcionales

> Cada requisito incluye: *descripción*, *actor*, *precondición*, *flujo principal*, *postcondición*, *prioridad* (MoSCoW) y *criterio de verificación* ipso facto.

#### Identidad y acceso

**RF-001 — Reutilizar autenticación institucional** *(origen ARS RF-001, RF-002)*
- El sistema debe autenticar usuarios usando el sistema de identidad institucional vigente; no debe crear un segundo proveedor de usuarios.
- Actor: cualquier usuario del instituto.
- Prioridad: Must.
- Verificación: un usuario del instituto accede al Módulo sin crear cuenta nueva; los permisos derivan de claims/rol locales.

**RF-002 — Integración en el dominio institucional**
- El Módulo debe publicarse en el dominio (o subdominio) institucional, accesible mediante un botón del portal actual.
- Prioridad: Must.
- Verificación: el enlace del portal redirige a la URL del Módulo con identidad transmitida.

**RF-003 — Control de acceso por rol y permiso**
- El sistema debe autorizar funciones según el rol (Rector, Coordinación, Docente, Apoyo Psicopedagógico, Administrador) y permisos granulares asignables.
- Prioridad: Must.
- Verificación: un docente no puede acceder a endpoints de calificación; un coordinador no puede asignar permisos globales.

#### Administración de maestros

**RF-010 — Administrar docentes, coordinadores y rector**
- CRUD de personas con rol vinculado y estado (activo/inactivo).
- Prioridad: Must.

**RF-011 — Administrar años lectivos**
- CRUD de años lectivos con periodo activo y periodos académicos asociados (1°..n°).
- Prioridad: Must.

**RF-012 — Administrar grados, cursos, áreas, nodos y asignaturas**
- CRUD y relaciones jerárquicas. Un *curso* pertenece a un *grado*; una *asignatura* pertenece a un *área* y/o *nodo*.
- Soporte de ambos modelos (primaria por nodo, secundaria por área).
- Prioridad: Must.

**RF-013 — Asignación docente↔curso/grado/asignatura/periodo**
- Coordinación asigna a cada docente los cursos/grados/asignaturas por año lectivo y periodo.
- Prioridad: Must.

**RF-014 — Plantilla configurable del Plan de Aula**
- Los campos del Plan de Aula (secciones y subsecciones) son configurables desde administración, conservando la estructura base observada en §10.
- Prioridad: Must.

**RF-015 — Versionado de plantilla**
- El sistema debe soportar versiones de la plantilla y mantener compatibilidad con Planes de Aula creados con versiones anteriores.
- Prioridad: Should.

#### Docente — Plan de Aula

**RF-020 — Crear Plan de Aula**
- Cada docente crea Planes de Aula por año académico, asociados a docente(s), curso(s), grado, área/nodo/asignatura y periodo académico. Permite planes en dupla cuando dos docentes comparten grado (ver plantilla real §10).
- Prioridad: Must.
 *(origen ARS RF-003, RF-004, RF-006)*

**RF-021 — Estructura uniforme y configurable**
- El formulario usa la misma estructura para todos los grados/cursos, con los ítems de secundaria conservados y campos configurables.
- Prioridad: Must. *(origen ARS RF-005, RF-006, RF-007)*

**RF-022 — Editar, guardar borrador y enviar**
- El docente puede guardar borrador, editar y enviar el Plan para evaluación de Coordinación.
- Prioridad: Must.

**RF-023 — Visibilidad limitada a asignaciones**
- El docente solo visualiza cursos/asignaturas/periodos asignados a él.
- Prioridad: Must. *(origen ARS RF-010)*

**RF-024 — Uso de IA en el diligenciamiento**
- El docente puede invocar funciones de IA (ver RF-040) en cada campo de texto del Plan.
- Prioridad: Must.

**RF-025 — Descarga PDF del Plan de Aula**
- El docente puede descargar su Plan de Aula en PDF respetando el formato institucional.
- Prioridad: Must. *(origen ARS RF-018)*

#### Coordinación — Evaluación

**RF-030 — Revisión del Plan**
- Coordinación visualiza los planes enviados, con filtros por docente, grado, asignatura y periodo.
- Prioridad: Must. *(origen ARS RF-009, RF-011)*

**RF-031 — Calificar, aprobar, rechazar, devolver**
- Coordinación califica, aprueba, rechaza, hace observaciones y devuelve para corrección.
- Prioridad: Must. *(origen ARS RF-011, RF-012)*

**RF-032 — Registro DUA / seguimiento**
- Coordinación marca las pautas DUA evidentes (checklist de la plantilla) y registra observación.
- Prioridad: Must.

**RF-033 — Seguimiento a la ejecución**
- El Plan de Aula contempla el bloque "Seguimiento del docente a la ejecución del eje/unidad/dimensión" por grupo/subgrupo, diligenciable por docente y revisable por Coordinación.
- Prioridad: Must.

#### Rectoría — Gobernanza

**RF-040 — Administración de permisos y parámetros**
- Rectoría administra permisos globales y parámetros generales del Módulo.
- Prioridad: Must.

**RF-041 — Estadísticas y dashboards institucionales**
- Rectoría visualiza KPIs: docentes pendientes/aprobados, % diligenciado, tiempo promedio, cursos/grados pendientes, gráficos de barras/circulares/líneas de tiempo/KPIs.
- Prioridad: Must. *(origen ARS RF-014, RF-015)*

**RF-042 — Validación de históricos importados**
- Rectoría valida/aprueba datos importados desde Excel (liga con RF-070).
- Prioridad: Must. *(origen ARS RF-023)*

#### IA

**RF-050 — Funciones del asistente IA**
- Disponibles: generar borrador, mejorar redacción, corregir ortografía, adaptar lenguaje pedagógico, resumir, expandir, proponer actividades, generar competencias, generar indicadores, generar observaciones, responder dudas sobre diligenciamiento.
- El docente conserva control total: la IA solo *sugiere*; el docente acepta/edita.
- Prioridad: Must. *(origen ARS §8)*

**RF-051 — Administración de IA**
- Configurar proveedor, modelo, prompts, temperatura, tokens, costos, límites, permisos.
- Prioridad: Must. *(origen ARS §8 Administración de IA)*

**RF-052 — Capa de abstracción AIService**
- Toda invocación de IA en la aplicación pasa por `AIService`. Cambiar de proveedor no debe tocar el dominio.
- Prioridad: Must. *(origen ARS §8 Infraestructura IA, §12)*

**RF-053 — Infraestructura Inforge.dev**
- El `AIService` se implementa sobre Inforge.dev, administrado por CLI oficial. Se conserva trazabilidad de llamadas (prompt/modelo/costo).
- Prioridad: Must. *(origen ARS §8, §9)*

#### Notificaciones

**RF-060 — Correos automáticos**
- El sistema envía correos cuando: apertura de diligenciamiento, recordatorios, retrasos, llamados de atención, aprobación, rechazo, devolución.
- Prioridad: Must. *(origen ARS RF-013)*

#### Importación Excel / migración de históricos

**RF-070 — Carga de archivos Excel**
- Administrador/Rector cargan archivos Excel con información organizada según la(s) plantilla(s) conocidas. Se admite un archivo por año/periodo o un archivo multi-periodo.
- Prioridad: Must. *(origen ARS RF-019, RF-023)*

**RF-071 — Validación estructural previa**
- Se valida estructura, campos obligatorios, tipos de datos y consistencia. El resultado es un *reporte de validación* con errores de bloqueo y advertencias.
- Prioridad: Must. *(origen ARS RF-020)*

**RF-072 — Transformación y mapeo a modelo de dominio**
- El sistema transforma las filas a entidades de dominio (Plan de Aula y subsecciones) y ejecuta la carga en una **transacción** (UoW) con *rollback* si hay error de bloqueo.
- Prioridad: Must.

**RF-073 — Trazabilidad de auditoría del importado**
- Cada registro importado conserva: fecha de redacción/creación original (si aplica), fecha de carga, fecha de última edición, usuario que cargó, usuario de última edición, usuario que validó/aprobó. *(origen ARS RF-022)*
- Prioridad: Must.

**RF-074 — Origen del registro (manual vs importado)**
- Cada registro lleva un flag `origin` (`MANUAL` | `IMPORTED`) y referencia al import batch.
- Prioridad: Must. *(origen ARS RF-024)*

**RF-075 — Edición posterior de importados sin perder historial**
- Los registros importados son editables en el Módulo, conservando el historial de cambios y la trazabilidad de auditoría.
- Prioridad: Must. *(origen ARS RF-021, RF-025)*

**RF-076 — Lote (batch) identificable y reversible**
- Toda importación queda como un *ImportBatch* con usuario, fecha, archivo original almacenado, count de exitosos/omitidos/erróneos y estado (PENDIENTE_VALIDACION | VALIDADO | RECHAZADO | REVERTIDO).
- Prioridad: Should.

#### Reportes, exportación y auditoría

**RF-080 — Exportación**
- Exportación PDF/Excel/CSV de Plan, reportes e indicadores.
- Prioridad: Must. *(origen ARS RF-018)*

**RF-081 — Auditoría de cambios**
- Todo cambio (crear/editar/eliminar, calificar, enviar, importar) genera un registro de auditoría con usuario, fecha, tipo de operación, entidad, id, *diff* antes/después.
- Prioridad: Must. *(origen ARS RF-016, RF-017)*

**RF-082 — Historial del Plan**
- Historial completo y consultable del Plan de Aula (versiones, con autor y observación).
- Prioridad: Must. *(origen ARS RF-017)*

### 7.2 Requisitos de idoneidad (capacidad / suitability — ISO 25010 Functional Suitability)

**RI-001** El sistema cubre el 100% de los *RF Must* de este SRS.
**RI-002** La importación soporta las plantillas "FORMATO + periodos 1°..n°" observadas en los archivos institucionales.

### 7.3 Requisitos no funcionales (RNF)

> Subsiguientes categorías según ISO/IEC 25010.

**Rendimiento**
- **RNF-001** Listado y filtrado de Planes: P95 ≤ 2 s con 500 docentes y 10.000 Planes.
- **RNF-002** Guardado de Plan: P95 ≤ 1 s.
- **RNF-003** Respuesta de IA: time-to-first-token ≤ 3 s; timeout configurable (default 30 s).

**Escalabilidad**
- **RNF-010** Soporta 5.000 docentes activos y 50.000 Planes históricos sin cambios de arquitectura. *(origen ARS §11)*
- **RNF-011** Diseñado para incorporar nuevos módulos (Observador, Convivencia, Notas, Planeación, Evaluaciones, Horarios, Asistencia, Biblioteca, Psicoorientación) sin reescribir el core.

**Disponibilidad**
- **RNF-020** 99,5 % de disponibilidad en horario laboral (definido por la institución).

**Seguridad (origen ARS §10)**
- **RNF-030** Autenticación institucional obligatoria; sin *login* propio.
- **RNF-031** Autorización por rol/permiso; denegación por defecto.
- **RNF-032** Auditoría y logs de seguridad.
- **RNF-033** Protección CSRF, XSS, SQLi; HTTPS obligatorio; HSTS.
- **RNF-034** Backups automáticos diarios + prueba de restauración mensual.
- **RNF-035** Validación estricta de archivos Excel (tamaño, mime firma, contenido).
- **RNF-036** Restricción de carga masiva por rol/permiso.
- **RNF-037** Almacenamiento de secretos en bóveda (no en código/config plano).

**Mantenibilidad / Portabilidad**
- **RNF-040** Clean Architecture + Hexagonal; el dominio sin dependencias de frameworks.
- **RNF-041** Cobertura mínima de pruebas unitarias: 80 % enDominio/Aplicación; 60 % en Adaptadores.
- **RNF-042** Contenerizable; despliegue reproducible.

**Usabilidad / Accesibilidad**
- **RNF-050** WCAG 2.1 AA en el frontend.
- **RNF-051** Trazado de formularios con etiquetas, validación clara y ahorrado de borrador.

**Observabilidad**
- **RNF-060** Logs estructurados (JSON) con correlación por `request_id`.
- **RNF-061** Métricas de IA (uso, latencia, costo, tokens) exportables.

**Internacionalización / localización**
- **RNF-070** UI en español (Colombia); arquitectura i18n-ready.

### 7.4 Requisitos de cumplimiento (RC)

- **RC-001** Cumplimiento de protección de datos personales (Ley 1581 de 2012 de Colombia y «Política de Tratamiento y Protección de Datos Personales» de la institución): mínimo en almacenamiento y auditoría, sin reenvío de PII a la IA sin anonimización/consentimiento.
- **RC-002** Históricos exportables a formatos abiertos (CSV) además de cerrados (Excel).
- **RC-003** Trazabilidad inmutable de las operaciones de auditoría (anexar, no sobrescribir).

### 7.5 Requisitos de datos (RD)

- **RD-001** Separación de **datos operativos** (relacional) y **datos de IA** (Inforge.dev, incluyendo embeddings cuando aplique).
- **RD-002** Esquema descrito en `03-datos/esquema-base-de-datos.md`.
- **RD-003** Reglas de consistencia referencial; borrado lógico (no físico) en todas las entidades de dominio.
- **RD-004** Conservación de los metadatos de auditoría listados en RF-073 en toda entidad de dominio editable.

### 7.6 Requisitos de interfaces externas (RI)

- **RI-010** **Interfaz con el portal institucional**: botón de entrada + endpoint *me* para leer identidad.
- **RI-020** **Interfaz de autenticación**: reutiliza el proveedor del instituto (SSO/OIDC/SAML según disponibilidad). El Módulo define un *adaptor* `IdentityProvider`.
- **RI-030** **Correo SMTP**: servicio institucional.
- **RI-040** **Inforge.dev (IA)**: CLI oficial para aprovisionamiento; API(s) por proveedor de modelos consumidas exclusivamente desde el adaptador concreto de `AIService`.
- **RI-050** **Excel (entrada)**: lecturas `.xlsx` (OpenXML).
- **RI-060** **Exportación**: generación PDF/Excel/CSV.

---

## 8. Requisitos de verificabilidad y aceptación

Cada requisito debe ser **único, no ambiguo y verificable** por al menos uno de estos métodos:

- Inspección (I)
- Demostración (D)
- Prueba de análisis/estática (A)
- Prueba dinámica (T) (unitaria/integración/funcional)

La aceptación final del proyecto requiere:
- A-01 Todos los RF *Must* cubiertos con evidencia de prueba.
- A-02 Pruebas unitarias ≥ 80 % en Dominio/Aplicación.
- A-03 Pruebas de integración/funcionales ejecutadas en verde.
- A-04 Pruebas de importación de Excel ejecutadas con plantilla institucional real.
- A-05 Revisión de seguridad pasada (SAST + revisión manual).
- A-06 Validación de accesibilidad WCAG 2.1 AA (audit + corrección de hallazgos críticos).
- A-07 Verificación de trazabilidad: registro importado editable mantiene historial.
- A-08 Integración con el portal verificada en entorno staging.

---

## 9. Trazabilidad con el Documento de Análisis de Requerimientos (ARS v1.0)

| ARS   | SRS              | Estado |
|-------|------------------|--------|
| RF-001 | RF-001           | cubierto |
| RF-002 | RF-002           | cubierto |
| RF-003 | RF-020           | cubierto |
| RF-004 | RF-020           | cubierto |
| RF-005 | RF-021           | cubierto |
| RF-006 | RF-021           | cubierto |
| RF-007 | RF-014           | cubierto |
| RF-008 | RF-015           | cubierto |
| RF-009 | RF-013, RF-030   | cubierto |
| RF-010 | RF-023           | cubierto |
| RF-011 | RF-031           | cubierto |
| RF-012 | RF-031           | cubierto |
| RF-013 | RF-060           | cubierto |
| RF-014 | RF-041           | cubierto |
| RF-015 | RF-041           | cubierto |
| RF-016 | RF-081           | cubierto |
| RF-017 | RF-082           | cubierto |
| RF-018 | RF-025, RF-080   | cubierto |
| RF-019 | RF-070           | cubierto |
| RF-020 | RF-071           | cubierto |
| RF-021 | RF-075           | cubierto |
| RF-022 | RF-073           | cubierto |
| RF-023 | RF-070           | cubierto |
| RF-024 | RF-074           | cubierto |
| RF-025 | RF-075           | cubierto |
| §8     | RF-050..RF-053   | cubierto |
| §9     | RD-001, RF-053   | cubierto |
| §10    | RNF-030..RNF-036 | cubierto |
| §11    | RNF-011          | cubierto |
| §12    | C4, RNF-040      | cubierto |
| §13    | §8, RNF-041      | cubierto |
| §14    | A-01..A-08       | cubierto |

---

## 10. Glosario funcional y estructura real del Plan de Aula

> Hallazgos del análisis de las plantillas institucionales `PLAN DE AULA 2026 ESPAÑOL.xlsx` y `PLAN DE AULA 2026 PENSAMIENTO SOCIAL.xlsx`. Cada archivo contiene una hoja `FORMATO` más hojas por periodo (`1°`..`11°`).

### 10.1 Hoja `FORMATO` — encabezado/orientaciones

- Orientaciones para diligenciar el plan de aula (texto guía).
- Tipos:
  - Planeación por grado.
  - Caracterización del grado/grupo puede elaborarse en dupla cuando dos docentes comparten grado.
  - Docentes de primaria con varias asignaturas/nodos hacen una sola caracterización.
  - Seguimiento en dupla: un solo registro por grado.

### 10.2 Encabezado por periodo

| Campo | Ejemplo |
|---|---|
| Docente(s) | `Carolina Saldarriaga - Elizabeth Zuluaga` |
| Nodo / Área / Asignatura | `Lengua Castellana - Comprensión lectora` |
| Fecha Inicio | `19 de enero de 2026` |
| Grado | `6°1 - 6°2` (admite múltiples grupos) |
| Periodo | `01`, `02`, …, `11` |
| N° de Semanas | `13` |
| N° de Clases | `65` |
| Fecha Cierre | `13 de abril de 2026` |

### 10.3 Secciones del cuerpo

1. **Caracterización del grupo o grado** — texto largo por grupo (p. ej. `6°1` y `6°2` por separado).
2. **Competencias**
   - *Específicas de Área / Nodo / Asignatura* (lista con numeración 1..4).
   - *Básicas y socioemocionales* (1. Conmigo mismo/intrapersonal; 2. Con los demás/interpersonal; 3. Proyectos de vida y metas académicas).
3. **Indicadores de desempeño** — tres categorías con porcentajes: *Conceptuales* (% 23), *Procedimentales* (% 23), *Actitudinales* (% 23).
4. **Actividades y número de clases** — tabla por *Eje temático / Unidad integradora / Dimensión*, columnas:
   - `N°`
   - Actividades y Rutinas de **Inicio** y saberes previos
   - Actividades y Rutinas de **Nueva información** y profundización
   - Actividades y Rutinas de **Finalización** y aplicación
   - Estrategias o instrumentos de **Evaluación**
   - **Criterios de valoración** de las actividades
5. **Seguimiento del docente a la ejecución del eje / unidad / dimensión** — texto por grupo (`6°1`, `6°2`, …).
6. **Principio DUA** — checklist de pa	pautas:
   - Compromiso: opciones para captar el interés; para mantener el esfuerzo y la persistencia; para la autorregulación.
   - Representación: diferentes opciones para la percepción; para el lenguaje y las expresiones matemáticas y símbolos; para la comprensión.
   - Acción y Expresión: opciones para la interacción física; para la expresión y la comunicación; para las funciones ejecutivas.
7. **Seguimiento Coordinación Académica** — texto largo con observación y *fecha* (ej. `06/05/26 …`, `27/04/2026 …`).
8. **Seguimiento Apoyo Psicopedagógico** — texto libre.

### 10.4 Implicaciones del modelo

- **Plan de Aula** = cabecera por *docente(s) + grado + área/nodo/asignatura + periodo* (un periodo = una hoja del Excel).
- **Dupla docente** — repr. *N..N* docente↔Plan en el mismo grado.
- **Múltiples grupos por Plan** — `grado` puede listar varios grupos (`6°1 - 6°2`); *seguimiento* y *caracterización* se registran **por grupo**.
- **Eje / Unidad / Dimensión** — fila de la tabla *Actividades*; se numera; cada uno con sus propias actividades (inicio/nueva/final), evaluación y criterios.
- **DUA** — vector de pautas marcadas (booleans) por sección DUA, asociadas al Plan o al Eje (decisión de diseño en `modelo-de-dominio.md`: por *Eje*.
- **Seguimientos por actor** — *Coordinación* y *Apoyo Psicopedagógico* son textos fechados; el sistema debe permitir múltiples registros de seguimiento (1..N) por Plan.

**Modelo de datos**: ver `03-datos/esquema-base-de-datos.md` y `02-arquitectura/modelo-de-dominio.md`.

---

*Fin del SRS.*
