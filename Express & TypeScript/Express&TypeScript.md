> # ***Modulo 3 - Clase 3: Express & TypeScript***

> ## ***Objetivos***

* ### *Repasar en qué consisten express y TypeScript.*

* ### *Aprender a inicializar un proyecto con NPM y TypeScript.*

* ### *Crear un CRUD básico y mejorar la seguridad del código.*

* ### *Comprender de qué manera los middlewares con express y TypeScript nos permiten hacer más eficiente el proceso de desarrollo.*

> ## ***Express & TypeScript***

* ### **Breve repaso de herramientas**

  * #### **Express**

    Express es un framework minimalista para node, ideal para construir aplicaciones web y APIs. Su sencillez y flexibilidad lo convierten en una opción popular para el desarrollo backend con JavaScript o TypeScript.

    ```javascript
    const express = require('express');
    const app = express();

    app.get('/', (req, res) => {
        res.send('¡Hola, Express!');
    });

    app.listen(3000, () => {
        console.log('Servidor iniciado en el puerto 3000')
    });
    ```

  * #### **TypeScript**

    TypeScrip es un lenguaje de programación que potencia las funcionalidades de JavaScript y que nos permite definir tipados estáticos, los cuales nos ayudarán a reducir comportamientos inesperados en la ejecución del código y que podemos prever en tiempo de compilación.

    ```typescript
    function saludar(nombre: string): string {
      return `¡Hola, ${nombre}!`;
    }

    const mensaje = saludar('Mario');
    console.log(mensaje);//"¡Hola, Mario!"
    ```

* ### **¿Cómo se complementan?**

  La combinación de ambas tecnologías implica la creación de aplicaciones del lado servidor más mantenibles, escalables y eficientes. Podremos dar uso a los tipos estáticos de TS dentro de los controladores y las rutas para detectar posibles errores y prevenir comportamientos inesperados en nuestros proyectos.

  ```typescript
  PORT = "Gato"

  const express = require('express');
  const dotenv = require('dotenv');
  dotenv.config();

  const app = express();

  const PORT: number = process.env.PORT; // Error: type 'string | undefined' is not assignable to type 'number'

  app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
  });
  ```

  PORT recibe un valor string en lugar de uno numérico así que TSC nos notifica que es preciso realizar la corrección para poder levantar el servidor.

  ```typescript
  import { Request, Response } from "express";

  interface Usuario {
    id: number;
    nombre: string;
  }

  const crearUsuario = async (req: Request, res: Response) => {
    const { id, nombre } = req.body;
    const nuevoUsuario: Usuario = { id, nombre };
    nuevoUsuario.extra = "extra"; // Error: property 'extra' does not exist on type 'Usuario'
    
    res.status(201).json({ usuario: nuevoUsuario });
  };
  ```
  
  Validamos la estructura de un nuevo usuario a partir de una interfaz Usuario, logrando ver errores en tiempo de compilación. En este caso, la propiedad extra no hace parte de la interfaz y de allí el error.

> ## ***Estructura de proyecto***

* ### **Inicialización de proyecto**

  1. Lo primero será crear un nuevo directorio con el comando mkdir mi-proyecto e ingresar al directorio del proyecto con cd mi-proyecto.

  2. Una vez dentro, inicializamos el proyecto utilizando npm init -y.

  3. Para instalar TypeScript, ingresa el comando npm install --save-dev typescript @types/express @types/node nodemon ts-node express.

* ### **Archivos y carpetas**

  ```
  mi-proyecto
  |-- node_moudles
  |-- dist
  |-- src
  |    |-- config
  |    |      |-- envs.ts
  |    |-- controllers
  |    |      |-- recursosController.ts
  |    |-- interfaces
  |    |      |-- IRecurso.ts
  |    |-- middlewares
  |    |      |-- autenticación.ts
  |    |-- routes
  |    |      |-- recursosRoute.ts
  |    |-- services
  |    |      |-- recursosService.ts
  |    |-- index.ts
  |    |-- server.ts
  |-- .env
  |-- .gitignore
  |-- nodemon.json
  |-- package-lock.json
  |-- package.json
  |-- tsconfig.json
  ```

  * #### **A tener en cuenta: tsconfig.json**

    1. Los archivos de TS a compilar estarán ubicados en la carpeta src y el resultado de la compilación estará en una carpeta dist que se creará en su momento.
        
        ```json
        "rootDir": "./src",
        "outDir": "./dist",
        ```

    2. El target señala que se compilará para la versión ES5 y se utilizará el sistema de módulos CommonJS. 
        
        ```json
        "target": "ES5",
        ```
    
    3. Con strict indicamos que se harán restricciones más rigurosas del código, lo que implica mayores comprobaciones por parte de TSC.

        ```json
        "strict": true,
        ```      

    4. Se incluirá en la compilación todos aquellos archivos dentro de src y sus directorios internos con extensión .ts y se omite la carpeta de node_modules para la compilación.

        ```json
        "include": ["src/**/*.ts"],
        "exclude": ["node_modules"]
        ```

  Por defecto, node no puede ejecutar código TypeScript. Por lo tanto, dentro del archivo package.json agregaremos un script build que ejecutará a TSC.

  Y tambien, agregaremos un script start para nodemon.

  ```json
  "scripts": {
    "build": "tsc",
    "start": "nodemon src/index.ts"
  }
  ```

  En el directorio raiz crearemos un archivo "nodemon.json" para poder ejecutar codigo typescript directamente en Node con ts-node

  ```json
  {
    "watch": ["src"],
    "ext": "ts",
    "exec": "ts-node src/index.ts"
  }
  ```

> ## ***Manejo de rutas***

* ### **Creación y tipado de rutas**

  Lo primero que haremos será crear una interfaz llamada Recurso. Esta interfaz estará definida en un archivo llamado recursos.ts dentro de una nueva carpeta llamada routes en el directorio src. Esto nos permitirá realizar validaciones dentro de las rutas.

  Tambien definiremos el router con los métodos HTTP, por lo que deberemos importarlo junto con las interfaces Request y Response para manejar la solicitud y la respuesta a las peticiones, respectivamente.

  ```typescript
  //src/routes/recursos.ts
  
  import { Router, Request, Response } from 'express';

  interface IRecurso {
    id: string;
    nombre: string;
  }

  const router = Router();
  ```

* ### **CRUD de Recursos**

  * #### **Creación de recursos**
    
    Crearemos una ruta de tipo POST. Utilizaremos la interfaz de IRecurso para el tipado en la solicitud con Request< IRecurso >, lo cual significa que esperamos recibir por body un objeto con esta estructura.

  * #### **Lectura de recursos**

    Para la lectura de recursos crearemos una ruta GET que ejecutará un controlador para obtener todos los recursos. Luego responderemos con la lista completa de estos.

  * #### **Actualización de recursos**

    Crearemos una ruta PUT para actualizar recursos existentes. En el controlador de la solicitud podremos asignar no sólo el tipo del recurso, sino también el valor recibido por params. Luego de esto, actualizamos y respondemos con el recurso actualizado. Si no existe, devolvemos un mensaje de error.

  * #### **Eliminación de recursos**

    Añadiremos una ruta DELETE para eliminar recursos. En este caso, solo necesitamos validar el tipo de dato recibido por params, ya que no construiremos un nuevo recurso, por lo que la interfaz no será necesaria. Luego verificamos. Si el recurso existe, lo eliminamos y respondemos con el recurso eliminado. Si este no existe, devolvemos un mensaje de error.

> ## ***Middlewares***

* ### **¿Qué era un middleware?**

  Los middlewares son funciones intermediarias que realizan una tarea concreta antes de que la request llegue a su destino.

* ### **Tipado de middlewares**

  Vamos a crear una nueva carpeta llamada middlewares dentro de src en la cual agregaremos un archivo de nombre autenticación.ts.

  En este archivo vamos a definir un middleware para verificar si un usuario tiene o no autorización para acceder a cierto contenido de la API.

  La estructura base de un middleware es una función que recibe como argumentos a request, response y next, que nos permiten tomar la solicitud, analizarla y continuar al endpoint designado por la ruta. Para ello, importamos las interfaces Request, Response y NextFunction.

  ```typescript
  //src/middlewares/autenticacion.ts

  import { Request, Response, NextFunction } from 'express';
  ```

  También vamos a enviar un token para saber que el usuario está autenticado y que tiene permiso para acceder a la información devuelta por el método GET. Esta información se envía a través del headers de la petición.

  Un header es información adicional que acompaña a una solicitud y puede especificar, por ejemplo, el tipo de contenido que se envía (JSON, texto plano, etc.), la longitud del contenido y, en particular, datos de autenticación, como en nuestro caso.
***

> ## ***Cierre***

* ### **En conclusión...**

  * ***Exploramos la sinergia entre TypeScript y Express:*** Como la ventaja de unir estas tecnologías hasta la creación de middlewares con tipado estático. Aprendimos cómo estructurar nuestros proyectos a partir de un esquema organizado de archivos y directorios.

  * ***Vimos como configurar el entorno de trabajo con Node y TypeScript:*** Para generar adecuadamente un servidor y definir las posibles rutas. También aprovechamos el tipado estático para prevenir errores al momento de definir controladores y middlewares.
***