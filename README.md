# InventarySoft

Documentación central del ecosistema **InventarySoft**: arquitectura de microservicios, convenciones de entornos y referencia a los repositorios de implementación. El código ejecutable vive en los repositorios de cada servicio; este repositorio concentra diseño, diagramas y evidencias.

---

## Tabla de contenidos

- [Introducción](#introducción)
- [Objetivos](#objetivos)
- [Visión de arquitectura](#visión-de-arquitectura)
- [Componentes del sistema](#componentes-del-sistema)
- [Flujo de solicitudes](#flujo-de-solicitudes)
- [Entornos de ejecución](#entornos-de-ejecución)
- [Puertos y URLs por entorno](#puertos-y-urls-por-entorno)
- [Instalación](#instalación)
- [Cómo navegar este repositorio](#cómo-navegar-este-repositorio)
- [Topología de repositorios](#topología-de-repositorios)
- [Metodología de trabajo](#metodología-de-trabajo)
- [Alcance de la documentación](#alcance-de-la-documentación)
- [Estructura de carpetas](#estructura-de-carpetas)
- [Sugerencias de evolución arquitectónica](#sugerencias-de-evolución-arquitectónica)

---

## Introducción

InventarySoft es una plataforma empresarial **distribuida** orientada a **inventario**, **ventas** y **control de acceso**. La solución evolucionó de un enfoque monolítico hacia **microservicios** para permitir despliegues independientes, mejor mantenibilidad y escalado horizontal por dominio.

Los límites entre servicios siguen el **dominio de negocio**; el tráfico externo se concentra en un **API Gateway**, que enruta hacia los microservicios correspondientes. El desarrollo se organiza en **Scrum**, con repositorios independientes por componente.

> **Nota de nomenclatura:** el producto se denomina **InventarySoft**. El servicio de dominio de existencias se denomina **Inventory Service** (convención en inglés para el bounded context de inventario).

---

## Objetivos

- Gestionar inventarios y catálogo de productos.
- Soportar procesos de ventas y flujos transaccionales.
- Administrar autenticación y autorización de forma centralizada en su servicio.
- Mantener separación clara entre **frontend**, **gateway**, **microservicios** y **base de datos**.
- Preservar un ecosistema alineado con arquitectura de microservicios y despliegue por componentes.

---

## Visión de arquitectura

| Principio | Descripción |
|-----------|-------------|
| Microservicios | Servicios desplegables por separado, acotados por dominio. |
| API Gateway | Punto de entrada único para el cliente; enrutamiento y políticas transversales. |
| Persistencia | PostgreSQL; cada servicio accede a los datos según sus responsabilidades (idealmente con esquemas o bases acotadas por servicio). |
| Frontend | Aplicación web que consume solo el gateway (no llama directamente a microservicios internos). |

---

## Componentes del sistema

Cada componente tiene una responsabilidad única y comunicación bien definida con el resto del ecosistema.

| Componente | Rol |
|------------|-----|
| **Frontend** | Interfaz de usuario; expone la experiencia web al usuario final. |
| **API Gateway** | Recibe peticiones HTTP del frontend, aplica reglas de entrada y delega en el microservicio adecuado. |
| **Inventory Service** | Inventario, catálogo y reglas de negocio asociadas al stock y productos. |
| **Sales Service** | Ventas, transacciones y flujos relacionados con el ciclo comercial. |
| **Auth Service** | Autenticación, autorización y emisión/validación de credenciales o tokens según el diseño adoptado. |
| **PostgreSQL** | Almacenamiento relacional; persistencia consultada y actualizada por los servicios según sus contratos. |

**Interacción resumida:** el usuario solo interactúa con el frontend; el frontend habla con el gateway; el gateway habla con uno o más microservicios; los microservicios persisten en PostgreSQL.

---

## Flujo de solicitudes

```text
Usuario → Frontend → API Gateway → Microservicio(s) → PostgreSQL
```

1. El usuario opera la interfaz web (**Frontend**).
2. El frontend envía peticiones HTTP/HTTPS al **API Gateway**.
3. El gateway **enruta** la petición al microservicio correcto (Inventory, Sales o Auth).
4. El microservicio ejecuta la **lógica de negocio** y, si aplica, lee o escribe en **PostgreSQL**.
5. La respuesta asciende por la misma cadena hasta el navegador.

---

## 🧪 Entornos de ejecución

Se definen **tres perfiles** para aislar configuración y puertos en máquina local o en despliegues de prueba.

| Entorno | Propósito |
|---------|-----------|
| **Main** | Perfil de referencia “producción-like” en local: puertos base documentados para el ecosistema completo. |
| **QA** | Validación integrada, pruebas de regresión y comprobación entre servicios antes de promover cambios. |
| **Dev** | Desarrollo activo; permite convivir con Main/QA usando **puertos distintos** para bases y servicios. |

**Buenas prácticas:** usar variables de entorno o ficheros `.env` por perfil en cada repositorio; no mezclar credenciales de un entorno en otro; documentar en cada repo cómo activar `MAIN`, `QA` o `DEV`.

---

## Puertos y URLs por entorno

### Base de datos (PostgreSQL)

| Entorno | Host:puerto |
|---------|-------------|
| Main | `localhost:5432` |
| QA | `localhost:5433` |
| Dev | `localhost:5434` |

### Microservicios

| Servicio | Main | QA | Dev |
|----------|------|----|-----|
| Inventory Service | `http://localhost:8081` | `http://localhost:8082` | `http://localhost:8083` |
| Sales Service | `http://localhost:9000` | `http://localhost:9001` | `http://localhost:9002` |
| Auth Service | `http://localhost:8888` | `http://localhost:8889` | `http://localhost:8890` |

### API Gateway

| Entorno | URL |
|---------|-----|
| Main | `http://localhost:8000` |
| QA | `http://localhost:8001` |
| Dev | `http://localhost:8002` |

### Frontend

| Entorno | URL |
|---------|-----|
| Main | `http://localhost:80` |
| QA | `http://localhost:81` |
| Dev | `http://localhost:82` |

### Vista consolidada (referencia rápida)

| Entorno | Frontend | Gateway | Inventory | Sales | Auth | PostgreSQL |
|---------|----------|---------|-----------|-------|------|------------|
| Main | `:80` | `:8000` | `:8081` | `:9000` | `:8888` | `:5432` |
| QA | `:81` | `:8001` | `:8082` | `:9001` | `:8889` | `:5433` |
| Dev | `:82` | `:8002` | `:8083` | `:9002` | `:8890` | `:5434` |

En Windows, el puerto **80** puede requerir permisos elevados; si aplica, usar otro puerto en local y actualizar la configuración del frontend o del proxy.

---

## 📦 Instalación

Este repositorio **no contiene** el código de los microservicios ni del frontend. La instalación del ecosistema completa es **multi-repositorio**.

### Requisitos generales

- Git
- Docker (recomendado para PostgreSQL y orquestación local, según cada repo)
- JDK y/o runtime indicados en cada servicio (ver README de cada repositorio)
- Node.js o stack del frontend (según `InventarySoft-frontend`)

### Pasos base

1. **Clonar** este repositorio para disponer de diagramas y documentación de arquitectura.
2. **Clonar** los repositorios listados en [Topología de repositorios](#topología-de-repositorios) en carpetas hermanas o según vuestra convención de workspace.
3. En cada servicio, seguir su README: levantar **PostgreSQL** con el puerto del entorno elegido (Main/QA/Dev), arrancar el **microservicio**, luego el **gateway** y por último el **frontend**, alineando variables de entorno con la tabla de puertos.
4. Verificar conectividad: frontend → gateway → servicio → base de datos.

Los detalles concretos (Maven, Gradle, npm, compose, etc.) deben tomarse de cada repositorio; aquí se define la **topología** y los **contratos de puertos** entre piezas.

---

## 🧭 Cómo navegar este repositorio

| Ubicación | Contenido |
|-----------|-----------|
| Raíz | Este `README` como índice del ecosistema. |
| `docs/diagrams/` | Diagramas de arquitectura, casos de uso, secuencia, despliegue, clases, paquetes, BPMN, C4. |
| `docs/mockups.pdf` | Prototipos / mockups del portal. |
| `LICENSE` | Licencia del repositorio de documentación. |

Para implementación, depuración y CI/CD, abrir el repositorio específico del componente (gateway, auth, etc.).

---

## Topología de repositorios

| Repositorio (GitHub) | Responsabilidad |
|----------------------|-----------------|
| [Sales-service](https://github.com/DiegoGuzman1999/Sales-service) | Microservicio de ventas |
| [Inventory-service](https://github.com/DiegoGuzman1999/Inventory-service) | Microservicio de inventario |
| [Auth-service](https://github.com/DiegoGuzman1999/Auth-service) | Microservicio de autenticación |
| [Gateway-Service](https://github.com/DiegoGuzman1999/Gateway-Service) | API Gateway |
| [inventorysoft-database](https://github.com/DiegoGuzman1999/inventorysoft-database) | Esquemas, migraciones o scripts de base de datos |
| [InventarySoft-frontend](https://github.com/DiegoGuzman1999/InventarySoft-frontend) | Interfaz web |

Nombres de repositorio en GitHub pueden combinar mayúsculas según el remoto; al clonar, usar la URL exacta de cada proyecto.

---

## Metodología de trabajo

- **Scrum:** entregas iterativas, transparencia con stakeholders y control del alcance.
- **Épicas** para objetivos de alto nivel; **historias de usuario** para incrementos; **sprints** para planificación y revisión.
- **Jira** para backlog, tablero y trazabilidad.

**Tablero del proyecto:** [InventarySoft – SCRUM Board](https://inventarysoft.atlassian.net/jira/software/projects/SCRUM/boards/1/backlog)

---

## Alcance de la documentación

En **este** repositorio se incluye:

- Arquitectura y decisiones de diseño a nivel ecosistema.
- Referencias y evidencias de evaluación (según lo añadáis en `docs/`).
- Prototipos y diagramas.
- Documentación de sprints y retrospectivas (cuando se incorporen).

La **implementación** y los pipelines de cada pieza se mantienen en sus repositorios correspondientes.

---

## Estructura de carpetas

```text
InventarySoft_v1-1/
├── README.md
├── LICENSE
└── docs/
    ├── mockups.pdf
    └── diagrams/
        ├── bpmn-diagram.png
        ├── c4-diagram.png
        ├── class-diagram.png
        ├── deploy-diagram.png
        ├── package-diagram.png
        ├── secuence_diagram.png   # nombre actual en repo (ortografía: sequence)
        └── use_case_diagram.png
```

Podéis añadir subcarpetas bajo `docs/` (por ejemplo `reference/`, `retrospectives/`) sin cambiar el rol de este repositorio como **hub de documentación**.

---

## Sugerencias de evolución arquitectónica

Breves líneas de mejora alineadas con sistemas distribuidos:

- **Observabilidad:** trazas distribuidas (trace ID desde gateway), métricas y logs estructurados por servicio.
- **Salud y resiliencia:** endpoints `/health` y `/ready`; timeouts y circuit breakers en el gateway hacia servicios.
- **Contratos:** versionado explícito de APIs (p. ej. prefijo `/v1`) y documentación OpenAPI por servicio.
- **Seguridad:** TLS entre componentes en entornos reales; políticas de secretos (no en repositorio); validación de tokens en gateway o delegación coherente con Auth.
- **Datos:** afinar límites de persistencia (schema-per-service o DB-per-service) para reducir acoplamiento a largo plazo.

---

*Última revisión orientada a documentación de arquitectura y convenciones de entorno. Para versiones de dependencias y comandos de arranque, consultar cada microservicio y el frontend.*
