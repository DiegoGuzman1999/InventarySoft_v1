# ADR-0002: Comunicación REST frente a gRPC entre servicios y gateway

## Estado

Accepted

## Fecha

2026-04-03

## Autores

Equipo InventarySoft

## Contexto breve

El `gateway` expone la API hacia los clientes del sistema de ventas e inventario y enruta hacia `auth-service`, `inventory-service` y `sales-service`. Cada uno de estos componentes vive en **su propio repositorio** y se despliega de forma independiente; la comunicación entre ellos debe ser interoperable, observable y compatible con **FastAPI** y el ecosistema HTTP estándar.

En desarrollo, **docker-compose** permite levantar el conjunto de contenedores; en cada repositorio, el servicio publica su interfaz HTTP sin asumir acceso al código fuente de los demás.

## Problema

¿Cuál era el problema?

Había que fijar el protocolo por defecto para la API pública y para el tráfico **gateway ↔ microservicios**, de modo que los equipos puedan **evolucionar y desplegar cada repositorio por separado** sin compilar ni versionar juntos los binarios, equilibrando rendimiento, claridad de contratos y curva de aprendizaje en un stack Python.

## Opciones consideradas

- **REST sobre HTTP con JSON (enfoque adoptado en FastAPI)**  
  - *Pros:* encaje natural con FastAPI, OpenAPI automático o documentado por servicio en **su propio repo**; depuración con herramientas HTTP habituales; el gateway actúa como cliente HTTP sin acoplarse al código interno de cada dominio.  
  - *Contras:* posible sobrecarga de serialización frente a binarios; riesgo de APIs conversadoras si no se diseñan agregaciones en el gateway.

- **gRPC con Protobuf**  
  - *Pros:* contratos fuertes y buen rendimiento en llamadas internas muy frecuentes.  
  - *Contras:* curva adicional en el proyecto; integración con clientes HTTP del gateway suele exigir traducción o librerías extra; mayor complejidad al **publicar y versionar esquemas** entre repositorios independientes frente al flujo OpenAPI ya alineado con FastAPI.

- **Híbrido: REST en el borde y gRPC interno**  
  - *Pros:* combina exposición HTTP con eficiencia interna teórica.  
  - *Contras:* dos líneas de contrato y operación por funcionalidad; coste de mantenimiento elevado para el alcance académico y de producto de InventarySoft.

## Decisión tomada

Se mantiene **REST + JSON** como estilo único para la API expuesta por el `gateway` y para la comunicación síncrona hacia `auth-service`, `inventory-service` y `sales-service`, documentando contratos con **OpenAPI** en el ámbito de cada repositorio (o publicando el esquema generado como artefacto de release).

gRPC queda fuera del alcance actual y solo se reevaluaría mediante un ADR futuro si aparecen cuellos de botella medidos y justificados.

## Consecuencias

**Ganamos:** coherencia con FastAPI; **independencia de despliegue** preservada (cada repo versiona su API sin tocar los demás); trazabilidad HTTP homogénea en logs y proxies; facilita la integración académica y las pruebas con herramientas estándar.

**Costo / compromiso:** disciplina de **versionado de API** entre repositorios (cambios rompedores coordinados por comunicación y pruebas contractuales, no por refactors en un solo árbol); política explícita de errores y paginación compartida como **estándar del ecosistema**, no como código compartido obligatorio.

## Notas / Próximos pasos

- Publicar o adjuntar OpenAPI por servicio en cada repositorio y referenciar en el gateway las rutas y versiones soportadas.  
- Definir guía de errores HTTP y formato de cuerpo de error común para InventarySoft.  
- Medir latencia y cardinalidad de llamadas entre gateway y dominios antes de plantear protocolos adicionales.
