# Preguntas y Respuestas sobre .NET y Cassandra

## ¿Cómo gestionaste la configuración del proyecto (.NET 8)?

Utilicé el sistema de configuración basado en archivos JSON (`appsettings.json`), variables de entorno y `IConfiguration`. Esto permite cargar valores de forma dinámica y segura, como cadenas de conexión, claves secretas y configuraciones específicas por entorno.

---

## ¿Qué es Cassandra y por qué lo usaste en lugar de una base de datos relacional?

**Apache Cassandra** es una base de datos **NoSQL distribuida** orientada a columnas. La elegí por su **alta disponibilidad**, **escalabilidad horizontal** y **tolerancia a fallos**, ideal para manejar grandes volúmenes de datos en tiempo real sin cuellos de botella.

---

## ¿Cómo funciona el modelo de datos en Cassandra?

Cassandra utiliza un modelo orientado a columnas con una estructura de tabla flexible. Las filas están identificadas por una clave primaria (**clave de partición + claves de agrupamiento**), lo que permite un acceso rápido a los datos y una mejor distribución en nodos.

---

## ¿Cómo se realiza una conexión entre .NET y Cassandra?

Utilicé el driver oficial de Cassandra para .NET, proporcionado por **Datastax**. Se configura mediante `Cluster.Builder()` y `Connect()` para establecer la conexión, y luego se ejecutan consultas usando sesiones (`ISession`).

---

## ¿Qué es JWT y para qué se utiliza en .NET?

**JWT (JSON Web Token)** es un estándar abierto (RFC 7519) usado para transmitir información de forma segura entre partes como un objeto JSON firmado digitalmente.
En .NET se utiliza principalmente para implementar **autenticación sin estado (stateless)**, protegiendo rutas y validando la identidad del usuario en cada solicitud.

---

## ¿Cómo se configura JWT en una aplicación ASP.NET Core?

```csharp
builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer("Bearer", options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = "miApp",
            ValidAudience = "miAppUsuarios",
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("clave_secreta_segura"))
        };
    });

builder.Services.AddAuthorization();
```

---

## ¿Qué contiene un token JWT típico?

Un token JWT tiene 3 partes:

- **Header**: indica el algoritmo de firma (ej. HS256).
- **Payload**: contiene los _claims_ (como `sub`, `name`, `exp`, `roles`, etc.).
- **Signature**: firma digital con clave secreta para verificar integridad.

```json
{
  "sub": "123456",
  "name": "Juan Perez",
  "role": "admin",
  "exp": 1718163200
}
```

---

## ¿Cómo se protege un endpoint con `[Authorize]`?

Se usa el atributo `[Authorize]` sobre controladores o acciones:

```csharp
[Authorize]
[HttpGet("perfil")]
public IActionResult GetPerfil() => Ok("Acceso autorizado");
```

---

## ¿Qué es el CassandraCSharpDriver y para qué se utiliza?

Es el driver oficial de **DataStax** para interactuar con clústeres **Apache Cassandra** desde aplicaciones en C#.
Permite abrir conexiones, ejecutar consultas CQL (**Cassandra Query Language**) y manejar resultados de manera asincrónica o sincrónica.

---

## ¿Qué clase se usa para conectarse al clúster de Cassandra?

Se usa la clase `Cluster` para construir una conexión y luego se obtiene una instancia de `ISession`:

```csharp
var cluster = Cluster.Builder()
    .AddContactPoint("127.0.0.1")
    .Build();

var session = cluster.Connect("mi_keyspace");
```

---

## ¿Qué es `ISession` y qué rol cumple?

`ISession` representa una conexión a un _keyspace_ en Cassandra.
Es la interfaz principal que se usa para ejecutar comandos:

```csharp
var result = session.Execute("SELECT * FROM usuarios");
```

---

## ¿Cómo se ejecutan consultas parametrizadas para prevenir inyecciones en Cassandra?

Usando consultas **preparadas** (`PreparedStatement`) y _bind_ de valores:

```csharp
var stmt = session.Prepare("SELECT * FROM usuarios WHERE id = ?");
var bound = stmt.Bind(Guid.Parse("..."));
var result = session.Execute(bound);
```

---

## ¿Cómo se serializan los datos entre C# y Cassandra?

El driver hace un mapeo automático de tipos C# a tipos CQL.
Por ejemplo, `Guid`, `string`, `DateTime`, `bool`, etc., son compatibles directamente.
También se puede usar **POCO mapping** con `MappingConfiguration`.

---

## ¿Por qué no se usa Entity Framework con Cassandra?

Entity Framework está diseñado para bases de datos relacionales (SQL Server, PostgreSQL, etc.) que tienen:

- Esquemas rígidos
- Relaciones entre tablas (JOINs)
- Mapeo ORM y transacciones

**Cassandra**, como base de datos NoSQL distribuida:

- No tiene relaciones ni soporte para JOINs
- No tiene transacciones completas como en SQL
- Está basada en un modelo de columnas anchas y _query-driven design_

EF no está preparado para trabajar con este modelo.
Por eso, se necesita un **driver especializado como CassandraCSharpDriver**, que conoce la arquitectura distribuida y ofrece soporte completo para:

- CQL
- Consistencia tunable
- Failover

---

## Argumento sólido: ¿Por qué usamos CassandraCSharpDriver y no EF?

Usamos `CassandraCSharpDriver` porque **Cassandra no es una base de datos relacional**, y **Entity Framework fue creado exclusivamente para modelos relacionales**.

El driver oficial permite aprovechar al máximo las ventajas de Cassandra:

- Escalabilidad horizontal
- Baja latencia
- Modelo de consistencia tunable
- Tolerancia a fallos

EF asume que:

- Los datos están fuertemente normalizados
- Existen relaciones entre entidades
- Se pueden consultar mediante LINQ
- Se necesitan transacciones ACID

Estas características **no existen en Cassandra**, cuyo enfoque es:

- Desnormalizado
- Distribuido
- Optimizado para lectura rápida basada en la clave de partición

Por lo tanto, el uso de EF es **incompatible** con el modelo de Cassandra.
La mejor práctica es usar el **driver oficial**, que entiende su arquitectura interna, distribución, particionamiento y replicación.
