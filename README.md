# Módulo de Planes de Trabajo Docentes (PTD) — IE Las Palmas (Envigado)

> Mock-ups e informe de la idea de diseño del **Módulo de Planes de Trabajo Docentes** para la
> Institución Educativa Las Palmas de Envigado. Esta entrega es un **vistazo visual** del proyecto
> para validación con el Rector, **no** es la aplicación final.

## 🔗 Sitio público (para enviar al Rector)

- **Índice de mock-ups:** <https://maxiusofmaximus.github.io/ie-las-palmas-ptd-diseno/>
- **Informe para el Rector (HTML):** <https://maxiusofmaximus.github.io/ie-las-palmas-ptd-diseno/informe/informe-resumen-proyecto.html>
- **Informe para el Rector (PDF):** <https://maxiusofmaximus.github.io/ie-las-palmas-ptd-diseno/informe/Informe-resumen-proyecto.pdf>
- **Ayuda / FAQ (leer antes de revisar):** <https://maxiusofmaximus.github.io/ie-las-palmas-ptd-diseno/09-ayuda-faq.html>
- **Documentación técnica (IEEE 29148):** <https://maxiusofmaximus.github.io/ie-las-palmas-ptd-diseno/documentacion/index.html>
- **Repositorio GitHub:** <https://github.com/maxiusofmaximus/ie-las-palmas-ptd-diseno>

## 📋 ¿Qué hay aquí?

Estos mock-ups son **wireframes de baja fidelidad** en HTML/CSS puro que replican la identidad
visual del portal institucional `ielaspalmas.edu.co` (paleta y escudo reales). Definen estructura,
contenido y flujo; **no** el acabado visual final ni la lógica de la aplicación.

### Pantallas disponibles

| # | Pantalla | Para quién | Cubre (RF) |
|---|---|---|---|
| 1 | Portal institucional + botón de entrada | Comunidad | RF-002 |
| 2 | Acceso (SSO Colombia Digital) + aterrizaje por rol | Todos | RF-001 |
| 3 | Dashboard de Rectoría | Rector | RF-040 · RF-041 · RF-042 · RF-080 |
| 4 | Dashboard de Coordinación | Coordinación | RF-030 · RF-031 · RF-032 · RF-033 |
| 5 | Editor del Plan de Aula con IA por campo | Docente | RF-020..RF-024 · RF-050 |
| 6 | "Mis Planes" + descarga PDF | Docente | RF-022 · RF-023 · RF-025 · RF-060 |
| 7 | Importación de Excel con validación | Rector / Admin | RF-070..RF-076 |
| 8 | Administración (maestros, periodos, plantilla) | Rector / Admin | RF-010..RF-015 |
| 9 | Ayuda y FAQ — guía de revisión ★ | Todos | Soporte |

### Identidad visual

- Paleta extraída del CSS real del portal (`theme7/styles.css`):
  - Verde oscuro institucional: `#4A6F2C`
  - Verde vivo / CTA: `#488704`
  - Verde claro / fondos suaves: `#A8C98D`
  - Azul de enlaces: `#004884`
- Escudo: `logofinalpalmas300_200x200.png` (ver documentación interna).
- Tipografía: system-ui, compatible con el portal.
- Accesibilidad: WCAG 2.1 AA (contraste, etiquetas).

## 📚 Documentación técnica

La carpeta `documentacion/` contiene el respaldo técnico completo y trazable del proyecto,
elaborado bajo la norma **IEEE 29148**. Es la fuente de verdad para implementación y gobernanza.

| Documento | Descripción |
|---|---|
| `documentacion/01-requerimientos/SRS-IEEE-29148.md` | Especificación contractual completa (RF, RNF, RC, RD, RI) |
| `documentacion/01-requerimientos/casos-de-uso-e-historias.md` | Casos de uso por actor + historias de usuario |
| `documentacion/01-requerimientos/reglas-de-negocio.md` | Invariantes verificables (RN-###) |
| `documentacion/02-arquitectura/modelo-de-dominio.md` | Bounded contexts, agregados, value objects (DDD) |
| `documentacion/02-arquitectura/diagrama-de-arquitectura.md` | Vista C4 + hexagonal, despliegue |
| `documentacion/03-datos/esquema-base-de-datos.md` | DDL PostgreSQL fuente de verdad |
| `documentacion/04-api/especificacion-api.md` | Contrato REST/JSON con códigos PTD-###-XXXX |
| `documentacion/05-backlog/backlog-inicial.md` | Épicas E0..E12, roadmap y DoR/DoD |

Índice navegable en: <https://maxiusofmaximus.github.io/ie-las-palmas-ptd-diseno/documentacion/index.html>

## 🗂 Estructura del repositorio

```
.
├── index.html                    # Índice navegable de mock-ups
├── estilos-base.css              # Hoja institucional compartida
├── logofinalpalmas.png           # Escudo real de la IE
├── .nojekyll                     # Sirve Markdown crudo sin Jekyll
├── 01-portal-boton-entrada.html
├── 02-login-sso.html
├── 03-dashboard-rectoria.html
├── 04-dashboard-coordinacion.html
├── 05-editor-plan-aula.html
├── 06-mis-planes-docente.html
├── 07-importacion-excel.html
├── 08-administracion.html
├── 09-ayuda-faq.html
├── capturas.cjs                  # Script de capturas PNG (opcional)
├── *.png                         # Capturas estáticas de cada pantallazo
├── documentacion/                # Documentación técnica IEEE 29148
│   ├── index.html                # Índice navegable de la documentación
│   ├── README.md
│   ├── 01-requerimientos/
│   ├── 02-arquitectura/
│   ├── 03-datos/
│   ├── 04-api/
│   └── 05-backlog/
└── informe/
    ├── informe-resumen-proyecto.html
    └── Informe-resumen-proyecto.pdf  # Versión imprimible para el Rector
```

## 🚀 Cómo abrir en local

Los HTML funcionan abriéndolos directamente con doble clic. Si quiere regenerar las capturas PNG
necesita [Playwright](https://playwright.dev/) instalado:

```bash
npx playwright install chromium
node capturas.cjs               # genera los PNG
playwright pdf informe/informe-resumen-proyecto.html informe/Informe-resumen-proyecto.pdf --paper-format Letter
```

## 📊 Documentación técnica de respaldo

El respaldo técnico completo (SRS bajo norma IEEE 29148, modelo de dominio, arquitectura, API,
backlog) se entrega por separado en el repositorio del proyecto y se comparte con el rector a su
solicitud. Esta publicación pública contiene **solo** material de presentación de diseño.

## 👤 Contacto

- **Autor:** Pedro Abelardo
- **GitHub:** [@maxiusofmaximus](https://github.com/maxiusofmaximus)
- **Correo:** [maxlive@hotmail.es](mailto:maxlive@hotmail.es)

## 📜 Licencia y protección de datos

- Estos mock-ups están bajo revisión; la licencia final se definirá con la institución.
- Cumple con la **Ley 1581 de 2012** (protección de datos personales): aquí no se almacenan ni
  transmiten datos reales de estudiantes ni docentes; son datos ficticios para presentación.
- El escudo y nombre de la IE Las Palmas son de uso institucional.

---

**Institución Educativa Las Palmas · Envigado · 2026**
