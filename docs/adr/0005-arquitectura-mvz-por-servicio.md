# ADR-0005: Patrón MVZ (Model, View, Zone) por servicio

## Estado

Accepted

## Fecha

2026-04-03

## Autores

Equipo InventarySoft

## Contexto breve

Cada microservicio de InventarySoft (`auth-service`, `inventory-service`, `sales-service`) y el `gateway` residen en **repositorios Git independientes**. Aun así, conviene una **convención estructural común** para que quien revise distintos servicios reconozca rápidamente dónde están las reglas de negocio, los adaptadores HTTP de FastAPI y el acceso a datos. El patrón **MVZ** (Model, View, Zone) aporta ese vocabulario compartido **sin compartir código** entre repositorios: la coherencia se logra mediante **guía de arquitectura y plantillas**, no mediante un monolito de carpetas.

## Problema

¿Cuál era el problema?

Sin una convención explícita, cada repositorio podría organizarse de forma distinta, dificultando la lectura cruzada en evaluaciones y en trabajo en equipo, y favoreciendo que la lógica de rutas FastAPI, la persistencia PostgreSQL y las reglas de dominio queden mezcladas en los mismos módulos.

## Opciones consideradas

- **MVZ por servicio, replicado en cada repositorio según la misma guía**  
  - *Pros:* estructura predecible al abrir cualquier repo del ecosistema; separación entre modelo de dominio o datos, capa View (rutas, DTOs, serialización FastAPI) y Zone (casos de uso por contexto); refuerzo del principio de un servicio, un despliegue y un estilo de código reconocible.  
  - *Contras:* requiere disciplina y revisión para no reinterpretar Zone de distinta forma en cada equipo; no hay un único árbol donde forzar la convención por herramienta, sino por **revisión y documentación**.

- **Arquitectura hexagonal o clean architecture sin nombre MVZ**  
  - *Pros:* flexibilidad y literatura abundante.  
  - *Contras:* mayor riesgo de divergencia entre repositorios en la nomenclatura de paquetes; menos útil como **estándar único de proyecto** para documentación académica unificada.

- **Estructura plana tipo controllers / services / repositories**  
  - *Pros:* fácil de explicar en un solo servicio.  
  - *Contras:* tendencia al dominio anémico y controladores sobrecargados si no se aplica rigor; menos mapeo claro con zonas de caso de uso en un sistema de ventas e inventario con reglas no triviales.

## Decisión tomada

Se adopta **MVZ como patrón de referencia en cada repositorio**: **Model** para entidades, agregados y acceso a PostgreSQL; **View** para la capa HTTP de FastAPI (routers, esquemas Pydantic, respuestas); **Zone** para casos de uso y orquestación que no deben residir en handlers ni en detalles de persistencia.

El `gateway` replica la idea: **Model** orientado a configuración de rutas y clientes HTTP hacia los demás servicios; **Zone** para políticas de enrutamiento, agregación o rate limiting, según evolucione el diseño, **siempre dentro de su propio repositorio**.

## Consecuencias

**Ganamos:** homogeneidad conceptual entre repos **sin acoplamiento de código**; revisiones y memorias técnicas más comparables; facilita explicar InventarySoft como conjunto de microservicios con prácticas internas alineadas.

**Costo / compromiso:** mantener una **guía breve o plantilla** (por ejemplo repositorio de plantilla o wiki del curso) que cada servicio siga al crearse; evitar dependencias circulares entre capas; tiempo de alineación inicial del equipo en la semántica exacta de Zone frente a Model.

## Notas / Próximos pasos

- Publicar un esqueleto MVZ de referencia para FastAPI (como plantilla o repo cookiecutter) que cada microservicio pueda adoptar al iniciar.  
- Validar en revisión de código que los nuevos endpoints no incorporen acceso directo a la base desde la capa View.  
- Documentar alias aceptables si el framework sugiere nombres distintos, manteniendo el mapa MVZ en el README de cada repositorio.
