
# Node.js Kick-off Workshop: Proyecto Real

## Introducción
Bienvenidos al proyecto de introducción a Node.js. Este proyecto está diseñado para aplicar los conceptos básicos de Node.js y Express que has aprendido durante la primera semana del curso. Crearás una API RESTful que interactúa con el sistema de archivos en lugar de una base de datos.

## Instrucciones de Entrega
1. Crea un repositorio en GitHub llamado `nodejs-first-project`.
2. Sigue las instrucciones y completa los objetivos establecidos en este documento.
3. Sube tu proyecto a GitHub y comparte el enlace del repositorio.

## Objetivos
- Configurar un entorno de desarrollo Node.js.
- Crear un servidor web utilizando Node.js y Express.
- Implementar rutas y middlewares en Express.
- Leer y escribir datos en el sistema de archivos utilizando el módulo `fs`.
- Manejar errores y asegurar la API con middlewares.

## Descripción del Proyecto
Crearás una API RESTful para gestionar una lista de tareas (To-Do List). Las tareas se almacenarán en un archivo JSON en el sistema de archivos.

## Historias de Usuario

### 1. Como usuario, quiero poder crear una nueva tarea para agregarla a mi lista de tareas.
- **Ruta:** POST `/tasks`
- **Cuerpo de la solicitud:**
  ```json
  {
    "title": "Nombre de la tarea",
    "description": "Descripción de la tarea"
  }
  ```
- **Respuesta:**
  ```json
  {
    "message": "Tarea creada exitosamente",
    "task": {
      "id": 1,
      "title": "Nombre de la tarea",
      "description": "Descripción de la tarea"
    }
  }
  ```

### 2. Como usuario, quiero poder ver todas mis tareas para revisarlas.
- **Ruta:** GET `/tasks`
- **Respuesta:**
  ```json
  [
    {
      "id": 1,
      "title": "Nombre de la tarea",
      "description": "Descripción de la tarea"
    },
    ...
  ]
  ```

### 3. Como usuario, quiero poder ver una tarea específica por su ID para conocer sus detalles.
- **Ruta:** GET `/tasks/:id`
- **Respuesta:**
  ```json
  {
    "id": 1,
    "title": "Nombre de la tarea",
    "description": "Descripción de la tarea"
  }
  ```

### 4. Como usuario, quiero poder actualizar una tarea existente para modificar su información.
- **Ruta:** PUT `/tasks/:id`
- **Cuerpo de la solicitud:**
  ```json
  {
    "title": "Nuevo nombre de la tarea",
    "description": "Nueva descripción de la tarea"
  }
  ```
- **Respuesta:**
  ```json
  {
    "message": "Tarea actualizada exitosamente",
    "task": {
      "id": 1,
      "title": "Nuevo nombre de la tarea",
      "description": "Nueva descripción de la tarea"
    }
  }
  ```

### 5. Como usuario, quiero poder eliminar una tarea para mantener mi lista organizada.
- **Ruta:** DELETE `/tasks/:id`
- **Respuesta:**
  ```json
  {
    "message": "Tarea eliminada exitosamente"
  }
  ```

## Requisitos del Proyecto

### 1. Configuración del Entorno
- Descarga e instala Node.js desde [nodejs.org](https://nodejs.org/). Recomendamos la versión LTS.
- Verifica la instalación con los siguientes comandos:
  ```sh
  node -v
  npm -v
  ```

### 2. Inicialización del Proyecto
- Inicia un nuevo proyecto Node.js:
  ```sh
  mkdir nodejs-first-project
  cd nodejs-first-project
  npm init -y
  ```

### 3. Instalación de Dependencias
- Instala Express:
  ```sh
  npm install express
  ```

### 4. Creación de la API RESTful

#### Estructura del Proyecto
```
nodejs-first-project/
├── data/
│   └── tasks.json
├── src/
│   ├── routes/
│   │   └── tasks.js
│   ├── middlewares/
│   │   └── errorHandler.js
│   └── app.js
├── package.json
└── index.js
```

#### 1. Crear el archivo `tasks.json`
- Crea la carpeta `data` y el archivo `tasks.json`:
  ```sh
  mkdir data
  echo "[]" > data/tasks.json
  ```

#### 2. Crear el Servidor con Express
- Crea la carpeta `src` y el archivo `app.js`:
  ```js
  const express = require("express"); // Importamos Express
  const tasksRoutes = require("./routes/tasks"); // Importamos las rutas de la API
  const errorHandler = require("./middlewares/errorHandler"); // Importamos el middleware para manejo de errores

  const app = express(); // Instanciamos Express
  const PORT = 3000; // Puerto del servidor en donde se ejecutará la API

  app.use(express.json()); // Middleware para parsear el cuerpo de las solicitudes en formato JSON. Tambien conocido como middleware de aplicación.
  app.use("/tasks", tasksRoutes); // Middleware para manejar las rutas de la API. Tambien conocido como middleware de montaje o de enrutamiento.
  app.use(errorHandler); // Middleware para manejar errores.

  app.listen(PORT, () => {
    console.log(`Server running at http://localhost:${PORT}/`);
  });
  ```

#### 3. Crear las Rutas de la API
- Crea la carpeta `routes` y el archivo `tasks.js`:
  ```js
  const express = require("express");
  const fs = require("fs");
  const path = require("path");

  const router = express.Router();
  const tasksFilePath = path.join(__dirname, "../../data/tasks.json");

  // Leer tareas desde el archivo
  const readTasks = () => {
    const tasksData = fs.readFileSync(tasksFilePath); // Leer el archivo. Este poderoso metodo nos permite leer archivos de manera sincrona.
    return JSON.parse(tasksData); // Retornar los datos en formato JSON.
  };

  // Escribir tareas en el archivo
  const writeTasks = (tasks) => {
    fs.writeFileSync(tasksFilePath, JSON.stringify(tasks, null, 2)); // Escribir los datos en el archivo. Este poderoso metodo nos permite escribir archivos de manera sincrona.
  };

  // Crear una nueva tarea
  router.post("/", (req, res) => {
    const tasks = readTasks();
    const newTask = {
      id: tasks.length + 1, // simulamos un id autoincrementable
      title: req.body.title, // obtenemos el titulo de la tarea desde el cuerpo de la solicitud
      description: req.body.description, // obtenemos la descripcion de la tarea desde el cuerpo de la solicitud
    };
    tasks.push(newTask);
    writeTasks(tasks);
    res.status(201).json({ message: "Tarea creada exitosamente", task: newTask });
  });

  // Obtener todas las tareas
  router.get("/", (req, res) => {
    const tasks = readTasks();
    res.json(tasks);
  });

  // Obtener una tarea por ID
  router.get("/:id", (req, res) => {
    const tasks = readTasks();
    const task = tasks.find((t) => t.id === parseInt(req.params.id));
    if (!task) {
      return res.status(404).json({ message: "Tarea no encontrada" });
    }
    res.json(task);
  });

  // Actualizar una tarea por ID
  router.put("/:id", (req, res) => {
    const tasks = readTasks();
    const taskIndex = tasks.findIndex((t) => t.id === parseInt(req.params.id));
    if (taskIndex === -1) {
      return res.status(404).json({ message: "Tarea no encontrada" });
    }
    const updatedTask = {
      ...tasks[taskIndex],
      title: req.body.title,
      description: req.body.description,
    };
    tasks[taskIndex] = updatedTask;
    writeTasks(tasks);
    res.json({ message: "Tarea actualizada exitosamente", task: updatedTask });
  });

  // Eliminar una tarea por ID
  router.delete("/:id", (req, res) => {
    const tasks = readTasks();
    const newTasks = tasks.filter((t) => t.id !== parseInt(req.params.id));
    if (tasks.length === newTasks.length) {
      return res.status(404).json({ message: "Tarea no encontrada" });
    }
    writeTasks(newTasks);
    res.json({ message: "Tarea eliminada exitosamente" });
  });

  module.exports = router;
  ```

#### 4. Crear Middleware para Manejo de Errores
- Crea la carpeta `middlewares` y el archivo `errorHandler.js`:
  ```js
  const errorHandler = (err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({ message: "Ocurrió un error en el servidor" });
  };

  module.exports = errorHandler;
  ```

#### 5. Archivo de Entrada del Proyecto
- Crea el archivo `index.js` en la raíz del proyecto:
  ```js
  require("./src/app");
  ```

## 5. Ejecución del Proyecto
- Ejecuta el proyecto con el siguiente comando:
  ```sh
  node index.js
  ```

- Deberás ver un mensaje similar al siguiente:
  ```
  Server running at http://localhost:3000/
  ```

- Abre tu navegador y accede a [http://localhost:3000/tasks](http://localhost:3000/tasks) para probar la API.

## 6. Probando la API (Ejemplo con POST)

Para probar nuestra API, podemos utilizar herramientas como [Postman](https://www.postman.com/) o [Insomnia](https://insomnia.rest/). Descarga e instala una de estas aplicaciones para realizar las siguientes pruebas.

Para este ejemplo, consideraremos que estamos utilizando Postman. Una vez descargado e instalado, sigue los pasos a continuación:

1. Abre Postman y encontrarás una interfaz similar a la siguiente:

<img src="./assets/postman_1.png" alt="Postman" width="500">

2. Le daremos click al boton `new` para crear una colección de peticiones. 

3. Se nos abrirá una venta donde elegiremos colección. En postman, las colecciones son grupos de peticiones que se pueden ejecutar juntas. Es mas que todo para organizar las peticiones que haremos a nuestra API y poder compartirlas con otros desarrolladores.

<img src="./assets/postman_2.png" alt="Postman" width="500">

4. Le damos un nombre a nuestra colección, en este caso `Node.js Kick-off Project`. A este punto puedes indagar más sobre funcionalidades de postman como variables de entorno, autenticación, etc. Te dejaré un link a la documentación oficial de postman [aquí](https://learning.postman.com/docs/getting-started/introduction/). Y un video de youtube que explica como usar postman mas a profundidad [aquí](https://www.youtube.com/watch?v=iFDQ3NFs95M&list=PLDbrnXa6SAzUsLG1gjECgFYLSZDov09fO).

<img src="./assets/postman_3.png" alt="Postman" width="500">

<img src="./assets/postman_4.png" alt="Postman" width="500">

5. Ahora crearemos una petición para crear una tarea. Para ello, le daremos click al boton `...` que se encuentra al lado de la colección que acabamos de crear y seleccionaremos `Add request`.

<img src="./assets/postman_5.png" alt="Postman" width="500">

6. Se nos abrirá una ventana donde podremos configurar nuestra petición. Le daremos un nombre a nuestra petición, en este caso `Create Task`. En el campo `Request URL` colocaremos la dirección de nuestra API, en este caso `http://localhost:3000/tasks`. En el campo `Request type` seleccionaremos `POST`. En el campo `Body` seleccionaremos `raw` y en el tipo de dato seleccionaremos `JSON` y en el campo inferior colocaremos el siguiente JSON:

```json
{
  "title": "Comprar leche",
  "description": "Ir al supermercado y comprar leche"
}
```

<img src="./assets/postman_6.png" alt="Postman" width="500">

<img src="./assets/postman_7.png" alt="Postman" width="500">

7. Ahora le daremos click al boton `Send` y veremos la respuesta de nuestra API.

<img src="./assets/postman_8.png" alt="Postman" width="500">

A este punto, ya debes suponer que el archivo `tasks.json` creado en la carpeta `data` contiene la tarea que acabamos de crear. 

<img src="./assets/tasks.json_1.png" alt="Postman" width="500">

Si no ves una respuesta similar a la que se muestra en la imagen, es posible que haya un error en tu código. Revisa el paso a paso y asegúrate de que todo esté configurado correctamente.

## 7. Probando la API (Resto de Verbos HTTP)

Como trabajo autonomo, prueba el resto de los verbos HTTP que se mencionan en las historias de usuario los cuales son:

- GET `/tasks`
- GET `/tasks/:id`
- PUT `/tasks/:id`
- DELETE `/tasks/:id`

## 8. Preguntas de Reflexión y trabajo investigativo

1. ¿Qué es el filesystem (fs) en Node.js y para qué se utiliza?

Este módulo permite realizar operaciones como leer archivos, escribir en archivos, crear archivos, modificar permisos de archivos y directorios, entre otras acciones relacionadas con el manejo de archivos y directorios.

2. ¿Qué es un middleware en Express y cuál es su propósito?

    Ejecutar código: Puede ejecutar cualquier código, realizar modificaciones en la solicitud y/o la respuesta, y ejecutar funciones asíncronas.

    Modificar objetos de solicitud y respuesta: Tiene acceso tanto al objeto de solicitud (req) como al objeto de respuesta (res), lo que permite modificar sus propiedades y comportamiento según sea necesario.

    Finalizar el ciclo de solicitud-respuesta: Puede finalizar el ciclo de solicitud-respuesta llamando a la función next() para pasar el control al siguiente middleware en la pila o finalizar la respuesta enviando una respuesta al cliente.

Propósitos principales de los middlewares en Express:

    Gestión de solicitudes: Pueden interceptar y manejar solicitudes HTTP antes de que lleguen a las rutas definidas, permitiendo la ejecución de código común para diferentes rutas o tipos de solicitudes.

    Modularidad y reutilización: Permiten dividir la lógica de la aplicación en funciones más pequeñas y modulares que se pueden combinar y reutilizar en múltiples rutas o aplicaciones.

    Manipulación de la respuesta: Pueden modificar la respuesta antes de que sea enviada al cliente, por ejemplo, añadiendo encabezados específicos, estableciendo códigos de estado, o incluso enviando datos específicos.

    Ejecución de código asincrónico: Pueden ejecutar operaciones asíncronas, como consultas a bases de datos o llamadas a API externas, gestionando adecuadamente la respuesta antes de continuar con el siguiente middleware.

3. ¿Qué es un endpoint en una API RESTful y cuál es su función?

un endpoint se refiere a un punto final específico de una API que se accede a través de una solicitud HTTP, como GET, POST, PUT, DELETE, etc. Cada endpoint representa una operación específica que puede ser realizada en la API

Un endpoint es una URL (Uniform Resource Locator) que se utiliza para acceder a un recurso en el servidor.
Consiste en la combinación de la URL base de la API junto con un path específico que identifica el recurso y la acción que se desea realizar sobre él.

4. ¿Qué son los verbos HTTP y cuáles son los más comunes?

Los verbos HTTP son métodos que indican la acción que se realizará sobre un recurso en un servidor. Los verbos más comunes son GET, POST, PUT, PATCH, y DELETE

5. ¿Qué es JSON y por qué es utilizado en las API RESTful?

JSON es un formato de texto que permite representar datos estructurados de manera legible tanto para humanos como para máquinas. Está basado en un subconjunto del lenguaje JavaScript, pero es independiente de cualquier lenguaje de programación y es fácil de entender y de generar.

Facilidad de Uso y Lectura:

    JSON es fácil de leer y escribir tanto para desarrolladores como para sistemas. La sintaxis es simple y clara, lo que facilita la comprensión y la depuración de datos.

Interoperabilidad:

    JSON es compatible con la mayoría de los lenguajes de programación modernos. Casi todos los lenguajes tienen librerías para serializar y deserializar datos JSON, lo que facilita la integración de APIs RESTful en diversas aplicaciones.

Ligereza y Eficiencia:

    JSON es un formato de datos ligero en comparación con XML u otros formatos más verbosos. Esto significa que consume menos ancho de banda y es más rápido de analizar, lo cual es crucial en el desarrollo de aplicaciones web y móviles que deben manejar grandes volúmenes de datos.

Formato Estandarizado:

    JSON se ha convertido en un estándar de facto para el intercambio de datos en la web, especialmente en el contexto de las APIs RESTful. Su adopción generalizada y su especificación simple lo hacen ideal para la comunicación entre clientes y servidores.

6. En lo que respecta al envio de datos a lo largo de los verbos http responde:
    - ¿Qué es el body de una petición?
    - ¿Qué es el body de una respuesta?
    - ¿Qué es el query de una petición?
    - ¿Qué es el params de una petición?

El body de una petición (o request body) es la parte de una solicitud HTTP que contiene los datos que el cliente desea enviar al servidor. Estos datos pueden ser en diversos formatos como JSON, XML, formularios URL-encoded, entre otros. El body de una petición se utiliza principalmente en los métodos HTTP que permiten enviar datos, como POST, PUT, y PATCH.

response body es la parte de la respuesta HTTP que contiene los datos devueltos por el servidor al cliente en respuesta a una solicitud. Este cuerpo puede contener datos estructurados en JSON, XML, texto plano, imágenes, etc., dependiendo de la naturaleza de la respuesta.

El query de una petición (o query string) es parte de la URL de una solicitud HTTP y contiene parámetros de consulta que se utilizan para filtrar, ordenar o paginar los resultados. Estos parámetros son visibles en la URL después del signo de interrogación (?) y están en formato clave=valor separados por el símbolo &.

Params de una Petición (Route Parameters):

Los params de una petición (o route parameters) son parte de la URL de una solicitud HTTP y se utilizan para identificar recursos específicos. Estos parámetros están incluidos en la propia ruta y son variables que forman parte de la estructura de la URL.

7. En lo que respecta al verbo POST responde:
    - ¿Qué es un verbo POST y cuál es su propósito?
    - ¿Cuándo se utiliza un verbo POST?
    - ¿En qué se diferencia un verbo POST de los otros verbos HTTP como GET, PUT y DELETE?
    - ¿Como se envian datos en un verbo POST?

El verbo POST es uno de los métodos de solicitud HTTP estándar definidos en la especificación HTTP. Su propósito principal es enviar datos al servidor para que sean procesados como parte del cuerpo de la solicitud. Es utilizado para crear recursos nuevos en el servidor o realizar operaciones que resulten en la modificación o creación de recursos en el servidor.

Se utiliza un verbo POST cuando se desea enviar datos al servidor para que estos sean procesados, almacenados o modificados de alguna manera en el servidor. Por ejemplo:

    Crear un nuevo usuario en una base de datos.
    Enviar datos de un formulario para ser procesados por el servidor.
    Enviar un mensaje en una aplicación de mensajería.
    Enviar datos para realizar una compra en línea.

GET: Se utiliza para solicitar datos del servidor. No modifica datos en el servidor y se espera que la solicitud sea segura e idempotente (no tiene efectos secundarios).
PUT: Se utiliza para actualizar o reemplazar un recurso existente en el servidor. Es idempotente, lo que significa que múltiples solicitudes PUT seguidas deberían tener el mismo efecto que una sola solicitud.
DELETE: Se utiliza para eliminar un recurso específico en el servidor. Es idempotente, al igual que PUT.
POST: Se utiliza para enviar datos al servidor para que sean procesados o almacenados. No es idempotente, ya que una solicitud POST múltiple puede resultar en la creación de múltiples recursos o en diferentes efectos dependiendo de la implementación del servidor.

En un verbo POST, los datos se envían en el cuerpo de la solicitud HTTP. El formato del cuerpo de la solicitud puede ser diverso y depende del tipo de datos que se estén enviando y de las convenciones de la aplicación. Comúnmente, se envían datos en formatos como JSON, formularios URL-encoded, XML, o incluso archivos binarios en caso de subir archivos.

8. En lo que respecta al verbo GET responde:
    - ¿Qué es un verbo GET y cuál es su propósito?
    - ¿Cuándo se utiliza un verbo GET?
    - ¿En qué se diferencia un verbo GET de los otros verbos HTTP como POST, PUT y DELETE?

El verbo GET es uno de los métodos de solicitud HTTP estándar definidos en la especificación HTTP. Su propósito principal es solicitar datos de un recurso específico del servidor. En otras palabras, se utiliza para recuperar información del servidor sin realizar modificaciones en los datos del servidor.

El verbo GET se utiliza en situaciones donde se desea obtener datos del servidor sin cambiar el estado de los recursos en el servidor. Algunos escenarios comunes donde se utiliza GET incluyen:

    Solicitar información de una página web específica.
    Recuperar datos de un API para ser mostrados en una aplicación web o móvil.
    Obtener detalles de un artículo en una tienda en línea.
    Acceder a recursos como imágenes, archivos CSS o JavaScript necesarios para renderizar una página web.

GET: Se utiliza para solicitar datos del servidor. No modifica datos en el servidor y se espera que la solicitud sea segura e idempotente (no tiene efectos secundarios).
PUT: Se utiliza para actualizar o reemplazar un recurso existente en el servidor. Es idempotente, lo que significa que múltiples solicitudes PUT seguidas deberían tener el mismo efecto que una sola solicitud.
DELETE: Se utiliza para eliminar un recurso específico en el servidor. Es idempotente, al igual que PUT.
POST: Se utiliza para enviar datos al servidor para que sean procesados o almacenados. No es idempotente, ya que una solicitud POST múltiple puede resultar en la creación de múltiples recursos o en diferentes efectos dependiendo de la implementación del servidor.


9. En lo que respecta al verbo PUT responde:
    - ¿Qué es un verbo PUT y cuál es su propósito?
    - ¿Cuándo se utiliza un verbo PUT?
    - ¿En qué se diferencia un verbo PUT de los otros verbos HTTP como POST, GET y DELETE?

El verbo PUT es uno de los métodos de solicitud HTTP estándar definidos en la especificación HTTP. Su propósito principal es actualizar o reemplazar un recurso existente en el servidor con los datos proporcionados en el cuerpo de la solicitud. Es utilizado para realizar modificaciones específicas y completas en los recursos del servidor.

¿Cuándo se utiliza un verbo PUT?

El verbo PUT se utiliza cuando se desea actualizar completamente un recurso existente en el servidor con nuevos datos proporcionados por el cliente. Algunos escenarios comunes donde se utiliza PUT incluyen:

    Actualización de información de usuario (nombre, dirección, etc.) en un sistema de gestión.
    Modificación de datos de un artículo en un inventario en línea.
    Reemplazo de un archivo en un almacenamiento en la nube con una nueva versión.

10. En lo que respecta al verbo DELETE responde:
    - ¿Qué es un verbo DELETE y cuál es su propósito?
    - ¿Cuándo se utiliza un verbo DELETE?
    - ¿En qué se diferencia un verbo DELETE de los otros verbos HTTP como POST, GET y PUT?

El verbo DELETE es uno de los métodos de solicitud HTTP estándar definidos en la especificación HTTP. Su propósito principal es solicitar al servidor que elimine un recurso específico. Es utilizado para eliminar recursos o datos específicos en el servidor de manera permanente.

¿Cuándo se utiliza un verbo DELETE?

El verbo DELETE se utiliza cuando se desea eliminar un recurso específico identificado por una URL del servidor. Algunos escenarios comunes donde se utiliza DELETE incluyen:

    Eliminación de un usuario de una base de datos.
    Borrado de un archivo de almacenamiento en la nube.
    Remoción de un artículo de un carrito de compras en línea.
    Borrar un mensaje de una bandeja de entrada.


11. ¿Qué es un status code y cuáles son los más comunes?

es un número de tres dígitos que se devuelve en una respuesta HTTP para proporcionar información sobre el estado de la solicitud realizada por el cliente al servidor. Estos códigos de estado permiten al cliente y al servidor comunicarse de manera efectiva sobre el resultado de la solicitud y cualquier acción que deba tomarse en respuesta a esa solicitud.

200 OK: La solicitud se ha completado correctamente y se ha devuelto el resultado solicitado.
201 Created: La solicitud se ha completado y se ha creado un nuevo recurso como resultado.
204 No Content: La solicitud se ha completado correctamente pero no hay contenido para devolver.
301 Moved Permanently: El recurso solicitado ha sido movido permanentemente a una nueva URL.
400 Bad Request: La solicitud del cliente no pudo ser entendida o contiene sintaxis incorrecta.
401 Unauthorized: El cliente debe autenticarse para obtener la respuesta solicitada.
403 Forbidden: El servidor entiende la solicitud del cliente, pero se niega a autorizarla.
404 Not Found: El recurso solicitado no se pudo encontrar en el servidor.
500 Internal Server Error: El servidor encontró una situación inesperada que le impidió completar la solicitud.


12. ¿Cuales son los status code mas comunes para el verbo POST?
13. ¿Cuales son los status code mas comunes para el verbo GET?
14. ¿Cuales son los status code mas comunes para el verbo PUT?
15. ¿Cuales son los status code mas comunes para el verbo DELETE?

Para el verbo POST:

    200 OK: Se utiliza para indicar que la solicitud POST se ha completado satisfactoriamente.
    201 Created: Indica que la solicitud POST ha tenido éxito y ha resultado en la creación de un nuevo recurso.
    400 Bad Request: Puede ocurrir si la solicitud POST es incorrecta o tiene una sintaxis inválida.
    401 Unauthorized: Indica que se requiere autenticación para la solicitud POST.
    403 Forbidden: El servidor entiende la solicitud POST, pero se niega a procesarla debido a permisos insuficientes.
    404 Not Found: Puede ser devuelto si el recurso al que se intenta enviar la solicitud POST no se encuentra en el servidor.
    500 Internal Server Error: Indica un error interno en el servidor al procesar la solicitud POST.

Para el verbo GET:

    200 OK: Indica que la solicitud GET se ha completado satisfactoriamente y que se ha devuelto el resultado solicitado.
    304 Not Modified: Se utiliza para indicar que el recurso solicitado no ha sido modificado desde la última solicitud GET y puede ser recuperado desde la caché.
    400 Bad Request: Puede ocurrir si la solicitud GET es incorrecta o tiene una sintaxis inválida.
    401 Unauthorized: Indica que se requiere autenticación para la solicitud GET.
    403 Forbidden: El servidor entiende la solicitud GET, pero se niega a procesarla debido a permisos insuficientes.
    404 Not Found: Indica que el recurso solicitado en la solicitud GET no se encuentra en el servidor.
    500 Internal Server Error: Indica un error interno en el servidor al procesar la solicitud GET.

Para el verbo DELETE:

    200 OK: Se utiliza para indicar que la solicitud DELETE se ha completado satisfactoriamente.
    204 No Content: Indica que la solicitud DELETE se ha completado satisfactoriamente y que no hay contenido para devolver.
    400 Bad Request: Puede ocurrir si la solicitud DELETE es incorrecta o tiene una sintaxis inválida.
    401 Unauthorized: Indica que se requiere autenticación para la solicitud DELETE.
    403 Forbidden: El servidor entiende la solicitud DELETE, pero se niega a procesarla debido a permisos insuficientes.
    404 Not Found: Indica que el recurso solicitado para eliminar en la solicitud DELETE no se encuentra en el servidor.
    500 Internal Server Error: Indica un error interno en el servidor al procesar la solicitud DELETE.

Para el verbo PUT:

    200 OK: Indica que la solicitud PUT se ha completado satisfactoriamente.
    201 Created: Indica que la solicitud PUT ha tenido éxito y ha resultado en la creación de un nuevo recurso.
    204 No Content: Indica que la solicitud PUT se ha completado satisfactoriamente y que no hay contenido para devolver.
    400 Bad Request: Puede ocurrir si la solicitud PUT es incorrecta o tiene una sintaxis inválida.
    401 Unauthorized: Indica que se requiere autenticación para la solicitud PUT.
    403 Forbidden: El servidor entiende la solicitud PUT, pero se niega a procesarla debido a permisos insuficientes.
    404 Not Found: Indica que el recurso solicitado para actualizar en la solicitud PUT no se encuentra en el servidor.
    500 Internal Server Error: Indica un error interno en el servidor al procesar la solicitud PUT.