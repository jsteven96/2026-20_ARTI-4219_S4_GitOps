
# Análisis del Provider PostgreSQL

## Provider: tages/provider-postgresql v0.1.0

### 1. Managed Resources disponibles

| Kind | Group | Version | Descripción |
| --- | --- | --- | --- |
| Database | postgresql.postgresql.upbound.io | v1alpha1 | Es el esquema para el API de Databases. Crea y maneja una base de datos en un servidor PostgreSQL |
| Extension | postgresql.postgresql.upbound.io | v1alpha1 | Crea y maneja una extensión en un servidor PostgreSQL |
| Function | postgresql.postgresql.upbound.io | v1alpha1 | Crea y maneja una función en un servidor PostgreSQL |
| Grant | postgresql.postgresql.upbound.io | v1alpha1 | Crea y maneja privilegios dados a un usuario para un esquema de base de datos. |
| Mapping | user.postgresql.upbound.io | v1alpha1 | Crea y maneja un mapeo de usuario en un servidor PostgreSQL |
| Privileges | default.postgresql.upbound.io | v1alpha1 | Crea y maneja privilegios por defecto dados a un usuario para un esquema de base de datos. |
| ProviderConfig | postgresql.upbound.io | v1alpha1 | Configura un proveedor de PostgreSQL |
| ProviderConfigUsage | postgresql.upbound.io | v1alpha1 | Indica que un recurso está usando un ProviderConfig |
| Publication | postgresql.postgresql.upbound.io | v1alpha1 | Crea y maneja una publicación en un servidor de base de datos PostgreSQL |
| ReplicationSlot | physical.postgresql.upbound.io | v1alpha1 | Crea y maneja un slot de réplica física para un servidor PostgreSQL |
| Role | grant.postgresql.upbound.io | v1alpha1 | Crea y maneja membresía en un rol a uno o más roles. |
| Role | postgresql.postgresql.upbound.io | v1alpha1 | Crea y maneja un rol en un servidor PostgreSQL |
| Schema | 	postgresql.postgresql.upbound.io | v1alpha1 | Crea y maneja un esquema dentro de una base de datos PostgreSQL |
| Server | postgresql.postgresql.upbound.io | v1alpha1 | Crea y maneja un servidor externo de PostgreSQL |
| Slot | replication.postgresql.upbound.io | v1alpha1 | Crea y maneja un slot de réplica en un servidor PostgreSQL |
| StoreConfig | postgresql.upbound.io | v1alpha1 | Configura cómo el controlador GCP debería guardar detalles de conexión. |
| Subscription | 	postgresql.postgresql.upbound.io | v1alpha1 | Esquema para el API de suscripciones. |

### 2. Campos requeridos del recurso Database

En el recurso ```Database```, en particular el objeto ```spec```es obligatorio

| Campo | Descripción | Obligatorio | Tipo |
| --- | --- | --- | --- |
| allowConnections | Si es falso, nadie se puede conectar a esta base de datos. El valor por defecto es ```true``` permitiendo conexiones (excepto cuando hay una restricción por otros mecanismos, como GRANT o REVOKE CONNECT) | No | Boolean |
| connectionLimit | Cuantas conexiones concurrentes pueden ser establecidas hacia la base de datos. Tiene un valor por defecto de -1 que significa que no hay límite. | No | Number |
| encoding | Conjunto de caracteres a ser utilizados en la base de datos. Se puede especficar a través de una constante tipo String (Por ejemplo ```UTF8, SQL_ASCII``` o un entero codificado. Si no se especifica se coloca un valor por defecto ```UTF8```) Cambiar este valor implica la creación de un nuevo recurso puesto que este valor solo se puede cambiar cuando una base de datos se está creando. | No | String |
| isTemplate | Si está en verdadero, la base de datos puede ser clonada por cualquier usuario con privilegios de CREATEDB. Si es falso (Valor por defecto) solo superusuarios o el dueño de la base de datos la puede clonar. | No | Boolean |
| lcCollate | Orden de intercalación (LC_COLLATE) a ser usada en la base de datos. Esto afecta el orden de clasificación aplicado a Strings en consultas como por ejemplo ```ORDER BY```, así como el orden usado en índices sobre columnas de texto. Si no se configura, el orden por defecto a ser usado es ```C```. Cambiar este valor obligará a la creación de un nuevo recurso puesto que este valor solo se puede cambiar en el momento de la creación de una base de datos.| No | String |
| lcCtype | Clasificación de caracteres (LC_TYPE) a ser usado en base de datos. Esto afecta la categorización de caracteres. Por ejemplo ```lower, upper, digit``` Si no se configura el valor por defecto es ```C```. Cambiar este valor obligará a la creación de un nuevo recurso puesto que este valor solo se puede cambiar en el momento de la creación de una base de datos. | No | String |
| name | El nombre de la base de datos. Debe ser único en la instancia del servidor PostgreSQL donde se configure. | Si | String |
| owner | El nombre del rol del usuario dueño de la base de datos, o el valor por defecto DEFAULT. Para crear una base de datos que peretence a otro rol o cambiar el dueño de una base de datos existente, se debe ser de forma directa o indirecta, un miembro del rol específico, o el nombre de usuario en el proveedor es un superusuario. | No | String |
| tablespaceName | El nombre de espacio de tablas que será asociado a la base de datos, o DEFAULT en caso de usar la plantilla del tablespace de la base de datos. Este tablespace será el por defecto cuando se creen objetos. | No | String |
| template | El nombre de la plantilla de base de datos desde la que se creará la base de datos, o DEFAULT cuando se use la plantilla por defecto (template0). Cambiar este valor implica la creación de un nuevo recurso debido a que su valor solo se puede cambiar cuando la base de datos es creada. | No | String |

### 3. Información requerida por el ProviderConfig

El `ProviderConfig` (```postgresql.upbound.io/v1beta1```) le indica al provider **dónde** y **con qué credenciales** conectarse al servidor PostgreSQL. Toda la configuración vive dentro de ```spec.credentials```.

| Campo | Descripción | Obligatorio | Tipo |
| --- | --- | --- | --- |
| credentials.source | Origen de las credenciales de conexión. El valor usado en este PoC es ```Secret```, es decir, las credenciales se leen desde un Secret de Kubernetes en lugar de ir hardcodeadas en el manifiesto (otros valores posibles del enum genérico de Crossplane son ```InjectedIdentity```, ```Environment```, ```Filesystem```, ```None```, pero este provider trabaja con ```Secret```). El valor es case-sensitive. | Sí | String (enum) |
| credentials.secretRef.namespace | Namespace donde vive el Secret con las credenciales. En este PoC es ```crossplane-system```. | Sí (cuando source: Secret) | String |
| credentials.secretRef.name | Nombre del Secret que contiene la cadena de conexión. En este PoC es ```postgresql-credentials```. | Sí (cuando source: Secret) | String |
| credentials.secretRef.key | Clave dentro del Secret donde está el valor de conexión. En este PoC es ```connection```. | Sí (cuando source: Secret) | String |

El valor almacenado en esa clave del Secret es un **JSON** con los parámetros reales de conexión al servidor PostgreSQL:

| Campo (dentro del JSON) | Descripción | Obligatorio | Tipo |
| --- | --- | --- | --- |
| host | Host o Service de Kubernetes donde escucha el servidor PostgreSQL (por ejemplo ```postgresql.postgresql.svc.cluster.local```). | Sí | String |
| port | Puerto TCP del servidor PostgreSQL (por defecto ```5432```). | Sí | String/Number |
| username | Usuario con el que el provider se autentica contra PostgreSQL. Debe tener privilegios suficientes para crear/gestionar bases de datos, roles, esquemas, etc. según los Managed Resources que se usen. | Sí | String |
| password | Contraseña del usuario anterior. | Sí | String |
| database | Base de datos a la que se conecta inicialmente el provider (normalmente ```postgres```, la base administrativa). | Sí | String |
| sslmode | Modo de conexión SSL/TLS (```disable```, ```require```, ```verify-ca```, ```verify-full```, etc.). En este PoC se usa ```disable``` porque el clúster es local (Kind). | No | String |

Finalmente, cada Managed Resource (```Database```, ```Role```, ```Grant```, etc.) debe referenciar este `ProviderConfig` mediante ```spec.providerConfigRef.name```, apuntando al nombre definido en el `ProviderConfig` (en este PoC: ```postgresql-config```).
