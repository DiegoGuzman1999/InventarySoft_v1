# ADR-0006: Orquestación con Docker Compose frente a Kubernetes

## Estado

Accepted

## Fecha

2026-04-03

## Autores

Equipo InventarySoft

## Contexto breve

InventarySoft está compuesto por `auth-service`, `inventory-service`, `sales-service` y `gateway`, cada uno en **su propio repositorio**, con **Dockerfile propio** y **despliegue independiente**. Las bases **PostgreSQL** siguen el criterio de una base de datos por servicio de datos. Para el trabajo diario y la demostración integrada, hace falta un mecanismo que levante el ecosistema en una sola máquina **sin fusionar** el ciclo de vida productivo en un único artefacto monolítico.

Por decisión de arquitectura, **docker-compose se utiliza exclusivamente para desarrollo (y, si aplica, laboratorios)**, mientras que cada servicio conserva su propia imagen y pipeline hacia su entorno de ejecución.

## Problema

¿Cuál era el problema?

Se debía elegir cómo **coordinar contenedores localmente** sin imponer Kubernetes prematuramente, respetando que **cada microservicio es construido y versionado desde su repositorio**, y sin confundir el archivo compose con el modelo de despliegue de producción.

## Opciones consideradas

- **Docker Compose solo para desarrollo, referenciando builds por contexto o imágenes de cada repo**  
  - *Pros:* curva baja; permite `docker compose up` tras construir desde los clones locales o desde imágenes publicadas por cada pipeline; separación clara entre **orquestación local** y **despliegue por servicio**.  
  - *Contras:* no ofrece HA ni orquestación de clúster; el archivo compose debe mantenerse coherente cuando cambian puertos o variables entre equipos.

- **Kubernetes desde el inicio para todos los entornos**  
  - *Pros:* escalado y patrones de producción empresariales.  
  - *Contras:* complejidad operativa y cognitiva desproporcionada para la fase académica y de maduración de InventarySoft; cuatro repos implican cuatro flujos de manifiestos o Helm charts sin necesidad inmediata demostrada.

- **Compose en desarrollo y Kubernetes en producción sin política de alineación**  
  - *Pros:* separa entornos.  
  - *Contras:* deriva entre lo que el desarrollador ejecuta y lo que opera producción; más coste de mantenimiento doble.

## Decisión tomada

Se mantiene **Docker por servicio en cada repositorio** (Dockerfile, opcionalmente `.dockerignore` y documentación de variables) y **Docker Compose únicamente como herramienta de integración en desarrollo**, pudiendo vivir el manifest en un **repositorio de entorno de desarrollo**, en documentación versionada, o en la convención acordada por el equipo, siempre **subordinado** a la autonomía de build y deploy de cada microservicio.

Kubernetes u otra plataforma de orquestación en clúster se pospone hasta que existan requisitos formales de HA, multi-nodo o políticas de red avanzadas, documentando entonces un ADR específico.

## Consecuencias

**Ganamos:** claridad pedagógica y operativa: **multirepo + contenedor por servicio + compose solo local**; cada pipeline publica o despliega su imagen sin depender del compose; onboarding explícito (“cómo integro los cuatro repos en mi máquina”).

**Costo / compromiso:** el compose de desarrollo debe **actualizarse cuando cambien contratos o puertos**; los secretos y configuraciones de producción **no** deben copiarse ciegamente desde el YAML de desarrollo; cuando se adopte Kubernetes, habrá que **mapear cada repositorio** a su chart o manifiesto sin romper la independencia de despliegue.

## Notas / Próximos pasos

- Ubicar y versionar el `docker-compose` de desarrollo (repo dedicado o sección de documentación) con servicios, redes y volúmenes para cada PostgreSQL y cada FastAPI.  
- Definir healthchecks y orden de arranque razonable en compose sin acoplar lógica de negocio.  
- Si el proyecto avanza a producción compartida, planificar ADR de orquestación en clúster alineado con los cuatro repositorios y sus pipelines.
