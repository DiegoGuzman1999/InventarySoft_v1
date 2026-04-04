# ADR-0004: Bases de datos por servicio con PostgreSQL

## Estado

Accepted

## Fecha

2026-04-03

## Autores

Equipo InventarySoft

## Contexto breve

En InventarySoft, `auth-service`, `inventory-service` y `sales-service` son **dominios con repositorios y despliegues propios**. La persistencia debe respetar el patrón **database per service**: cada servicio es dueño de su esquema y de sus migraciones, sin acceso directo a las tablas de otro contexto. Se utiliza **PostgreSQL** por entorno, coherente con FastAPI y contenedores.

Para **desarrollo**, **docker-compose** puede levantar varias instancias o bases lógicas y enlazar cada contenedor de aplicación con su cadena de conexión; en **producción**, cada servicio se configura contra su instancia o base dedicada según la política de infraestructura, **sin compartir credenciales entre repositorios** más allá de lo que imponga el entorno seguro.

## Problema

¿Cuál era el problema?

Había que evitar el antipatrón de **base de datos compartida**, que acoplaría ciclos de despliegue y esquemas entre equipos y repositorios, y elegir un motor **único por convención** del proyecto para reducir heterogeneidad operativa en la fase académica y de producto.

## Opciones consideradas

- **Una instancia PostgreSQL compartida con esquemas separados por servicio**  
  - *Pros:* menos procesos en máquinas pequeñas de laboratorio.  
  - *Contras:* riesgo de fugas de acoplamiento (conexiones cruzadas); responsabilidad de backup y migración menos clara; contradice la imagen de **propiedad total del dato por servicio** en un multirepo bien gobernado.

- **PostgreSQL dedicado por servicio (instancia o base lógica claramente asignada)**  
  - *Pros:* límites nítidos de esquema; migraciones versionadas **dentro del repositorio del servicio**; escalado y restauración alineados con el dominio; coherencia con despliegues independientes.  
  - *Contras:* más recursos en desarrollo; operaciones distribuidas entre ventas e inventario requieren **APIs o patrones de saga/eventos**, no transacciones únicas.

- **Motores heterogéneos por servicio**  
  - *Pros:* optimización puntual por caso de uso.  
  - *Contras:* sobrecarga de conocimiento y operación para el alcance actual de InventarySoft.

## Decisión tomada

Se elige **PostgreSQL dedicado por servicio de datos** para `auth-service`, `inventory-service` y `sales-service`: cada uno define su conexión, migraciones y políticas de acceso en **su propio repositorio**, sin que otro servicio ejecute SQL contra su almacén.

La coordinación entre dominios de ventas e inventario se realiza mediante **llamadas REST** (y, si en el futuro se justifica, mensajería), nunca mediante JOINs entre bases ajenas.

## Consecuencias

**Ganamos:** alineación entre **multirepo, bounded context y datos**; libertad para evolucionar el esquema de inventario sin bloquear el despliegue de ventas, y viceversa; claridad documental para evaluación académica.

**Costo / compromiso:** mayor número de contenedores o bases en local; necesidad de **scripts o compose de desarrollo** que levanten N PostgreSQL o bases con nombres y puertos acordados; monitorización y copias de seguridad **por servicio**; diseño explícito de consistencia eventual donde antes había una sola transacción monolítica.

## Notas / Próximos pasos

- Documentar en cada repositorio variables de entorno (`DATABASE_URL`, etc.) y estrategia de migraciones (por ejemplo Alembic por servicio).  
- Definir en el entorno de desarrollo con docker-compose los nombres de servicio de base de datos y redes para que coincidan con la documentación de cada repo.  
- Identificar procesos transversales que requieran sagas u outbox y registrarlos en diseño o ADRs posteriores si aplica.
