# CODE REVIEW
​
## Pre-revisión
​
- [✓]  El repositorio es público
​
### README.MD
​
- [×]  Descripción general del proyecto.
- [✓]  Tecnologías implementadas en el proyecto, versión y objetivo con el cual se realizó la implementación.
- [✓]  Enlace al demo.
- [✓]  Imagen de preview (si aplica).
- [✓]  Enlace a la documentación desplegada (si aplica).
- [✓]  Enlace a documento con los requisitos del sistema.
- [×]  Nombre del coach que asignó y revisó el proyecto
- [✓]  Información del autor del proyecto: nombre, usuario en Slack, enfoque (frontend, backend, ds).
​
### GIT
​
- [×]  Los commits deben estar 100% en inglés.
- [×]  Los commits deben ser descriptivos y puntuales.
- [✓]  Debe existir al menos una rama adicional a main para desarrollo.
- [×]  Los commits siguen un flujo de trabajo para PR (git flow)
​
## Revisión
<hr>

**Ruta:** /src/app.js:3

**Código:**

```js
var cors = require("cors");
```
**Comentarios:**
Es preferente utilizar `const` si la variable es declarada al comienzo del archivo y esta no cambiará su valor.

<hr>

**Ruta:** /src/app.js:15-20

**Código:**

```js
const db = mysql.createPool({
  host: process.env.DATABASE_HOST,
  user: process.env.DATABASE_USER,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE,
});
```
**Comentarios:**
Reutilizar archivo `database.js` para manejar las conexiones entre Mongo y MySQL y dejarle la responsabilidad a este mismo.

<hr>

**Ruta:** /src/app.js:44-46

**Código:**

```js
app.use("/auth", require("./routes/auth"));
app.use("/celebrities", require("./routes/celebrities"));
app.use("/songsArtists", require("./routes/songsArtists"));
```
**Comentarios:**
Se podría encapsular el routing dentro de una función (pe: `setRoutes`) para separar funcionalidades necesarias para levantar el servidor y que sea descriptivo para el desarrollador.

<hr>

**Ruta:** /src/app.js:9

**Código:**

```js
const auth = require("./middleware/auth");
```
**Comentarios:**
Esta importación no se utiliza dentro del archivo.

<hr>

**Ruta:** /src/app.js:22-25

**Código:**

```js
var corsOptions = {
  origin: "*",
  optionsSuccessStatus: 200,
};
```
**Comentarios:**
Si los valores de la variable son constantes, utilizar `const` en lugar de `var`

<hr>

**Ruta:** /src/database.js:22-25

**Código:**

```js
require("dotenv").config();
const USER = encodeURIComponent(process.env.DB_USER);
const PASSWORD = encodeURIComponent(process.env.DB_PASSWORD);
const DB_NAME = process.env.DB_NAME;
```
**Comentarios:**
Se puede crear un archivo con las variables configuradas por fuera y que este sea importado en donde sea necesario. Con esto podemos:
- Evitar repetir la carga de variables de entorno cada vez que se importe.
- Configurar valores por defecto en las variables, por ejemplo
```js
export const config {
    database: {
        user: process.env.DB_USER || "",
        pass: process.env.DB_PASS || "",
        name: process.env.DB_NAME || "",
    }
}
```
- `encodeURIComponent` recibe valores `string`, `number` y `boolean`. Una variable de entorno no configurada sería `undefined` y puede causar algún problema.

<hr>

**Ruta:** /src/controllers/auth.js:5-10

**Código:**

```js
const db = mysql.createPool({
  host: process.env.DATABASE_HOST,
  user: process.env.DATABASE_USER,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE,
});
```
**Comentarios:**
El código está duplicado en `src/app.js`.

<hr>

**Ruta:** /src/controllers/auth.js:14

**Código:**

```js
const { email, password } = req.body;

    if (!email || !password) {
      return res.status(400).json({
        error: {
          message: "Please provide an email and password",
        },
      });
    }

    db.query(
      "SELECT * FROM users WHERE email = ?",
      ...
```
**Comentarios:**
Existe un riesgo de seguridad de [SQL Injection](https://es.wikipedia.org/wiki/Inyecci%C3%B3n_SQL). Es recomendable no solo validar si `email` o `passsword` sino realizar una sanitización de los datos para evitar un problema relacionado.

<hr>

**Ruta:** /src/controllers/auth.js:40-42

**Código:**

```js
const token = jwt.sign({ id }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRES_IN,
});
```
**Comentarios:**
En lo personal me gusta ocupar mas espacio de forma vertical que horizontal para hacer un código mas legible. Mi sugerencia es aprovecharlo como en este ejemplo:
```js
const token = jwt.sign(
    { id },
    process.env.JWT_SECRET,
    { expiresIn: process.env.JWT_EXPIRES_IN }
);
```

<hr>

**Ruta:** /src/controllers/auth.js:12

**Código:**

```js
    exports.login = ...
```
**Comentarios:**
La función `login` tiene mucho potencial para un refactor. Como programadores resolvemos problemas grandes haciéndolos mas pequeños y manejables. Lo mismo debe ser con nuestras funciones. Observé que `login` realiza estos pasos:
- Verificar si `req.body` contiene datos y si estos son válidos
- Query para buscar si existe el usuario con ese correo
- Comparar hashes entre la contraseña actual y la conversión del `password` recibido
- Crear un token de authentication
- Configurar cookies
- Actualizar último login de usuario
- Resolver solicitud en base al flujo

Mi sugerencia es separar todos los pasos en pequeñas funciones para obtener un código mantenible, escalable y fácil de leer. Algo así:

```js
    const validateBody = (body) => { ... }
    const getUser = (email) => { ... }
    const validatePassword = (password) => { ... }
    const createToken = (data) => { ... }
    const createCookies = (data) => { ... }
    const updateUser = (email) => { ... }
    const createAPIResponse = (data) => { ... }

    exports.login = (req, res) => {
        validateBody()
        getUser()
        ...
    }
```

Como adicional, nos podemos evitar un posible callback hell observado cada vez que se manda a llamar `db.query()`. Mi sugerencia es utilizar promises para encapsular este comportamiento y que sea mucho mas fácil de mantener

```js
const getUser = (user) => {
    return new Promise((resolve, reject) => {
        db.query(
            query,
            { data },
            async (err, result) => {
                if (err)
                    reject("Error")
                // Trabajar variable result para obtener la info deseada
                if (result)
                    resolve(result)
            }
        )
    });
}
```


<hr>

**Ruta:** /src/controllers/auth.js:79-83

**Código:**

```js
const { email, password, passwordConfirm } = req.body;

  db.query(
    "SELECT email FROM users WHERE email = ?",
    [email],
```
**Comentarios:**
Existe un riesgo de seguridad de [SQL Injection](https://es.wikipedia.org/wiki/Inyecci%C3%B3n_SQL). Es recomendable no solo validar si `email` o `passsword` sino realizar una sanitización de los datos para evitar un problema relacionado.

Validar si las contraseñas coinciden debe ser un trabajo realizado al 100% por frontend para evitar trabajo no necesario en el servidor. Backend solo necesita `email` y `password` para trabajar.

<hr>

**Ruta:** /src/controllers/auth.js:117-120

**Código:**

```js
return res.json({
    status: 202,
    "message:": "User registered",
});
```

**Comentarios:**
El código adecuado para esta solicitud es `201` ya que significa que un usuario fue creado con éxito en el sistema. `202` aplicaría mas en una solicitud que se resuelva correctamente pero debamos esperar a que se procese alguna cosa (pe: una solicitud para generar un reporte muy pesado y este sea enviado por correo).

<hr>

**Ruta:** /src/controllers/auth.js:79-126

**Código:**

```js
exports.register = (req, res) => { ... }
```

**Comentarios:**
La función `register` puede ser refactorizada en pasos mas pequeños. Identifiqué los siguientes:
- Validar (y sanitizar) valores recibidos en `req.body`
- Identificar si el correo recibido no fue utilizado anteriormente
- Encriptar password
- Inseerter usuario en base de datos
- Retornar solicitud de éxito o fracaso

<hr>

**Ruta:** /src/controllers/celebrities.js:3-7

**Código:**

```js
exports.info =  async(req, res) => {
  const db = await connect();
  const result = await db.collection('celebrities').find({}).toArray();
  res.json(result);
};
```

**Comentarios:**
La función debería incluir un manejo de errores en caso de que `result` no reciba un valor adecuado o el proceso retorne un error.

<hr>

**Ruta:** /src/controllers/index.js:3-8

**Código:**

```js
const db = mysql.createPool({
  host: process.env.DATABASE_HOST,
  user: process.env.DATABASE_USER,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE,
});
```

**Comentarios:**
El código está duplicado y la constante `db` no es utilizada en el archivo.

<hr>

**Ruta:** /src/controllers/songsArtist.js:3-7

**Código:**

```js
exports.info =  async(req, res) => {
  const db = await connect();
  const result = await db.collection('songsArtists').find({}).toArray();
  res.json(result);
};
```

**Comentarios:**
La función debería incluir un manejo de errores en caso de que `result` no reciba un valor adecuado o el proceso retorne un error.

<hr>

**Ruta:**
/src/graphQL/resolvers.js:6-11
/src/graphQL/resolvers.js:12-20
/src/graphQL/resolvers.js:12-20
/src/graphQL/resolvers.js:21-25
/src/graphQL/resolvers.js:26-33

**Código:**

```js
const result = await db.collection("").find({}).toArray();
```

**Comentarios:**
La función debería incluir un manejo de errores en caso de que `result` no reciba un valor adecuado o el proceso retorne un error.

<hr>

**Ruta:** /src/graphQL/resolvers.js:12

**Código:**

```js
getCelebritie: async (_, args) => { }
```

**Comentarios:**
Posible typo, debería ser `getCelebrity`

<hr>

**Ruta:** /src/utils/validateFacebookToken.js:7

**Código:**

```js
(res) => {
    if (res.statusCode === 200) {
        return true;
    }
    return false;
}
```

**Comentarios:**
Este código podría ser simplicado:
```js
(res) => res.statusCode === 200
```

<hr>

**Ruta:** /src/utils/validateGoogleToken.js:13

**Código:**

```js
const userid = payload["sub"];
if (userid) {
    return true;
}
return false;
```

**Comentarios:**
El código busca saber si userid es `truthy` o `falsy`. Se puede identificar el valor que puede tomar `userid`, ya sea `null` o `undefined` y hacer la comparación según sea el caso

```js
return userid !== undefined;
```

<hr>

## Comentarios generales
- Explicar el por qué de los archivos de configuración (babel)
- No incluye carpeta de test


## Reviewer
​
**Nombre:** Bryan Sánchez

​
**Usuario de Slack:** Bryan Sánchez [C6]
​

**Background:** Backend (Node.JS, C#, Python)