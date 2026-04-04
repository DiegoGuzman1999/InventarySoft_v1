# ADR-0001: Monorepo frente a multirepo para microservicios

## Estado

Accepted

## Fecha

2026-04-03

## Autores

Equipo InventarySoft

## Contexto breve

InventarySoft es un sistema web de ventas e inventario que evoluciona desde un monolito hacia microservicios con **separación estricta por dominio**. Los componentes `auth-service`, `inventory-service`, `sales-service` y `gateway` se implementan con **FastAPI (Python)**, se comunican por **HTTP (REST)** y se empaquetan con **Docker**, cada uno con su propio ciclo de vida.

Por **requisito académico y de arquitectura**, la propiedad del código debe reflejar la autonomía de despliegue: cada microservicio reside en **su propio repositorio en GitHub**, con **runtime y pipeline de despliegue independientes**. El uso de **docker-compose** queda acotado al **entorno de desarrollo** para integrar localmente los contenedores construidos desde cada repositorio, sin implicar un monorepo productivo.

## Problema

¿Cuál era el problema?

Era necesario decidir si la migración debía concentrarse en un único repositorio (monorepo) o en **repositorios aislados (multirepo)**, de modo que la documentación y la práctica del proyecto demuestren **límites de dominio reales**: propiedad del código, versionado y despliegue alineados con cada servicio, sin depender de un árbol de fuentes compartido que pudiera ocultar acoplamientos indebidos.

## Opciones consideradas

- **Monorepo con servicios aislados por carpeta**  
  - *Pros:* un solo clon; cambios transversales visibles en un único pull request; plantillas de CI unificadas.  
  - *Contras:* los límites de despliegue y propiedad pueden volverse ambiguos frente a evaluadores o equipos; mayor riesgo de dependencias cruzadas por comodidad; no cumple el criterio de **repositorio independiente por microservicio** exigido para el trabajo académico.

- **Multirepo: un repositorio GitHub por microservicio y por el gateway**  
  - *Pros:* correspondencia directa entre dominio, historial Git y despliegue; permisos y secretos por repositorio; pipelines específicos por artefacto Docker; demuestra arquitectura de microservicios **desacoplada en el origen**.  
  - *Contras:* mayor esfuerzo al coordinar cambios que afectan contratos entre servicios; duplicación potencial de configuración (mitigable con plantillas de organización o documentación); el desarrollador debe clonar o referenciar varios repos para el stack completo.

- **Monorepo transitorio con extracción posterior a multirepo**  
  - *Pros:* transición gradual desde el legado monolítico.  
  - *Contras:* deja una fase intermedia que contradice el requisito de **separación completa por dominio** desde el planteamiento docente; arrastra deuda de corte entre repositorios.

## Decisión tomada

Se adopta **multirepo**: **cuatro repositorios independientes** (uno para `auth-service`, uno para `inventory-service`, uno para `sales-service` y uno para `gateway`), cada uno con su propio **Dockerfile**, variables de entorno, pipeline de integración y despliegue, y documentación de API acotada a su contexto.

La integración local del ecosistema InventarySoft se resuelve mediante **docker-compose de desarrollo** (por ejemplo en un repositorio de entorno de desarrollo o documentación operativa) que **orquesta imágenes o contextos de build** provenientes de cada repositorio, **sin** convertir el monorepo en la forma oficial de entrega.

## Consecuencias

**Ganamos:** alineación explícita entre **dominio, repositorio y despliegue**; autonomía de equipos o módulos académicos por servicio; trazabilidad de incidentes y releases por artefacto; cumplimiento del enunciado de microservicios con **fronteras reales en el control de versiones**.

**Costo / compromiso:** gobernanza de **contratos REST** (versionado, compatibilidad hacia atrás, comunicación entre mantenedores de repos); posible duplicación de **plantillas CI/CD** y estándares de calidad, que debe compensarse con guías comunes o plantillas de GitHub; curva de onboarding algo mayor hasta documentar el flujo de clonar varios repositorios y usar compose de desarrollo.

## Notas / Próximos pasos

- Nombrar y publicar en GitHub los cuatro repositorios con convención clara (por ejemplo prefijo `inventarysoft-`).  
- Documentar el procedimiento de desarrollo local: construcción de imágenes por repo y archivo compose que las ensambla.  
- Establecer reglas de cambio en API (deprecación, versiones en ruta o cabecera) aplicables a todos los servicios, sin fusionar código en un solo repositorio.
