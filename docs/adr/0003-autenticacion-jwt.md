# ADR-0003: Autenticación basada en JWT

## Estado

Accepted

## Fecha

2026-04-03

## Autores

Equipo InventarySoft

## Contexto breve

InventarySoft distribuye la responsabilidad entre `gateway`, `auth-service`, `inventory-service` y `sales-service` en **repositorios separados**. La autenticación no puede depender de un único proceso monolítico ni de sesiones compartidas en memoria entre despliegues. El `auth-service`, en su propio repositorio y base **PostgreSQL** dedicada, concentra la emisión y la política de credenciales; los demás servicios deben **validar tokens** sin acoplarse a su código fuente.

El stack elegido (REST, contenedores Docker, despliegues independientes) favorece tokens firmados y verificables en cada instancia de FastAPI.

## Problema

¿Cuál era el problema?

Se requería un mecanismo de autenticación que escale con **varios artefactos desplegables**, minimice la necesidad de llamar al `auth-service` en cada petición solo para conocer la identidad (más allá de políticas de revocación), y sea estándar para APIs HTTP, **sin introducir bibliotecas compartidas privadas entre repositorios** como condición para funcionar.

## Opciones consideradas

- **JWT firmados (por ejemplo RS256 con par de claves o HS256 en entornos controlados)**  
  - *Pros:* validación local en `gateway`, `inventory-service` y `sales-service` usando la clave pública o el secreto configurado por entorno; encaja con REST y FastAPI; desacopla el runtime de los consumidores del código del emisor.  
  - *Contras:* revocación inmediata exige diseño (TTL cortos, lista de denegación, rotación de claves documentada entre repos); riesgo si los claims son excesivos o se filtran datos sensibles en el payload.

- **Sesiones server-side con almacén compartido (por ejemplo Redis)**  
  - *Pros:* revocación sencilla invalidando la sesión.  
  - *Contras:* introduce **infraestructura compartida** y acoplamiento operativo entre equipos o repos; va en contra del modelo de fronteras autónomas que persigue el multirepo de InventarySoft, salvo que se justifique como servicio transversal explícito.

- **OAuth2 / OpenID Connect con introspección de token opaco en cada petición**  
  - *Pros:* modelo maduro para federación.  
  - *Contras:* latencia y disponibilidad dependientes de un punto de introspección; complejidad adicional para el alcance del proyecto si ya basta JWT bien acotado.

## Decisión tomada

Se adopta **JWT** como formato principal de token tras autenticación en `auth-service`, con políticas claras de **expiración**, **firma** y **claims mínimos** para autorización en `inventory-service` y `sales-service`.

El `gateway` (en su repositorio) puede validar el JWT en el borde y propagar la identidad mediante cabeceras internas o contexto de petición hacia los microservicios, según el diseño de FastAPI, **configurando en cada repo** las claves o el JWKS necesarios, sin dependencias de código cruzadas entre repositorios.

## Consecuencias

**Ganamos:** modelo alineado con microservicios HTTP y multirepo; cada servicio puede desplegarse y escalar por separado validando el mismo contrato de token; integración natural con FastAPI y middleware de seguridad.

**Costo / compromiso:** **gobernanza de claves** (rotación, distribución segura de secretos o claves públicas en CI/CD por repositorio); definición de TTL y, si aplica, refresh tokens; plan de revocación; nunca confiar en claims no verificados. Los cambios en el formato de token requieren **coordinación explícita** entre mantenedores de repos afectados.

## Notas / Próximos pasos

- Documentar en el repositorio de `auth-service` el esquema de claims y el mecanismo de firma; documentar en consumidores cómo obtener la clave pública o el secreto de verificación de forma segura.  
- Acordar si el `gateway` centraliza solo autenticación o también parte de la autorización por ruta.  
- Evaluar política de refresh tokens y almacenamiento en clientes web del sistema de ventas e inventario.
