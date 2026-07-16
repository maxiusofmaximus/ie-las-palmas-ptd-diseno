# Documentación del Proyecto — PTD IE Las Palmas (Envigado)

**Sistema Web de Gestión de Planes de Trabajo Docentes**
Institución Educativa Las Palmas · Versión 1.0 · Julio 2026

---

## 1. Contexto

Este repositorio contiene la documentación técnica y funcional que **sirve de base para implementar** (en Codex u otra herramienta) el **Módulo de Planes de Trabajo Docentes** (PTD) que se incorpora al dominio institucional de la IE Las Palmas.

**Cierre de contexto construido para este proyecto:**
- Se parte del *Documento de Análisis de Requerimientos de Software v1.0* (incluido en `../ARS-v1.0.md` o provisto por el cliente).
- Se estudiaron **las plantillas reales** del Plan de Aula vigentes en la institución (`PLAN DE AULA 2026 ESPAÑOL.xlsx` y `PLAN DE AULA 2026 PENSAMIENTO SOCIAL.xlsx`). La estructura observada (encabezado, caracterización por grupo, competencias, indicadores con porcentajes, ejes con actividades, seguimientos por actor, pautas DUA y hasta 11 periodos por archivo) está registrada en el SRS §10 y en el Modelo de Dominio.
- Se estabilizaron las decisiones con `Inforge.dev` como infraestructura de IA (administrada por CLI oficial), accedida **únicamente** por el puerto `AIService`.
- Se aplican SOLID, DRY, KISS, YAGNI, Clean Architecture, Hexagonal, DDD, Repository, Unit of Work, DI y CQRS donde aporta valor.

## 2. Estructura de carpetas

```
docs/
├── README.md                              ← este archivo
├── 01-requerimientos/
│   ├── SRS-IEEE-29148.md                  ← SRS maestro (IEEE 29148)
│   ├── casos-de-uso-e-historias.md        ← UC + HU
│   └── reglas-de-negocio.md               ← RN-### trazables
├── 02-arquitectura/
│   ├── modelo-de-dominio.md                ← DDD: BCs, agregados, VOs, puertos
│   └── diagrama-de-arquitectura.md         ← C4 + hexagonal + deployment
├── 03-datos/
│   └── esquema-base-de-datos.md            ← DDL PostgreSQL + ER
├── 04-api/
│   └── especificacion-api.md               ← REST/JSON + códigos PTD-XXX
└── 05-backlog/
    └── backlog-inicial.md                  ← Épicas, HU, roadmap, DoR/DoD
```

## 3. Documentos y rol

| Documento | Rol | Audiencia |
|---|---|---|
| `01/SRS-IEEE-29148.md` | Especificación contractual y técnica completa. | Producto, Dev, QA, Cliente |
| `01/casos-de-uso-e-historias.md` | Detalle observable de comportamiento. | Producto, QA, UX |
| `01/reglas-de-negocio.md` | Invariantes verificables. | Dev, QA, Producto |
| `02/modelo-de-dominio.md` | Pilar del código DDD. | Dev (backend) |
| `02/diagrama-de-arquitectura.md` | Pilar de despliegue/cross-cutting. | DevOps, Dev |
| `03/esquema-base-de-datos.md` | DDL fuente de verdad de persistencia relacional. | Dev, DBA |
| `04/especificacion-api.md` | Contrato de servicios. | Dev (back+front), QA |
| `05/backlog-inicial.md` | Planificación inicial. | Producto, Dev |

## 4. Convenciones de identificación

- **RF-###**: Requisitos funcionales (SRS §7.1).
- **RNF-###**: Requisitos no funcionales (ISO 25010 categorizados).
- **RC-###**: Requisitos de cumplimiento.
- **RD-###**: Requisitos de datos.
- **RI-###**: Requisitos de interfaces externas.
- **RN-###**: Reglas de negocio (`reglas-de-negocio.md`).
- **UC-###**: Casos de uso (`casos-de-uso-e-historias.md`).
- **HU-x.y**: Historias de usuario por épica.
- **PTD-###-XXXX**: Códigos de error de API (`especificacion-api.md` §14).
- **E0..E12**: Épicas del backlog.

## 5. Orden de lectura sugerido

1. `01/SRS-IEEE-29148.md` (§6 Describe el producto; §10 estructura real del Plan).
2. `01/casos-de-uso-e-historias.md`.
3. `01/reglas-de-negocio.md`.
4. `02/modelo-de-dominio.md`.
5. `02/diagrama-de-arquitectura.md`.
6. `03/esquema-base-de-datos.md`.
7. `04/especificacion-api.md`.
8. `05/backlog-inicial.md`.

## 6. Próximos pasos sugeridos (Codex)

1. Generar `openapi.yaml` desde `04/especificacion-api.md` y validar con lint.
2. Scaffold del backend modular (12 módulos) según Modelo de Dominio y Arquitectura.
3. Migraciones Flyway/Liquibase a partir del DDL (`03/esquema-base-de-datos.md`).
4. Implementar por épicas siguiendo el roadmap del backlog (Sprints 0..12).
5. Cobertura, SAST y auditoría WCAG AA según el backlog E11.

## 7. Documento de entrada (ARS v1.0)

El *Documento de Análisis de Requerimientos de Software v1.0* permanece como documento de partida; este conjunto lo especializa y lo hace trazable/implementable según IEEE 29148. La matrices de trazabilidad están dentro del SRS (§9) y en cada regla/historia.

---

**Nota de licenciamiento y privacidad**: Pending del institutional/owner. Documento sujeto a protección de datos personales colombiana (Ley 1581 de 2012); no incluye PII real.
