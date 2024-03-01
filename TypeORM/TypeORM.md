> # ***Modulo 3 - Clase 5: TypeORM***

> ## ***Objetivos***

* ### *Aprender qué es TypeORM.*

* ### *Realizar la configuración inicial de un proyecto con TypeORM.*

* ### *Conocer las operaciones básicas de TypeORM.*

* ### *Implementar la relación de tablas con TypeORM.*

> ## ***TypeORM***

* ### **¿Qué es?**

  Un ORM (Object Relational Mapping) es un tipo de herramienta que permite la interacción entre una base de datos relacional y código estructurado en objetos de JavaScript.

  TypeORM es una biblioteca para trabajar en diferentes entornos de ejecución. En nuestro caso, el de node con enfoque en TypeScript y JavaScript nos proporciona una interfaz que permite transformar objetos a tablas de una base de datos relacional.

* ### **¿Por qué utilizar TypeORM y qué ventajas ofrece?**

  A pesar de que existen múltiples opciones para incorporar como ORM a un proyecto que trabaja con bases de datos relacionales (como Hibernate, Entity, o Sequelize), TypeORM ha adquirido mucha más popularidad ya que proporciona una mayor simplificación del proceso de desarrollo y el mapeo de información.
  
  Es compatible con varios gestores de bases de datos relacionales, como el ya conocido PostgreSQL, y otros como MySQL, SQLite y SQL Server, brindando flexibilidad según la necesidad del proyecto. Además, se integra con facilidad a frameworks como express.

* ### **Instalación y Setup en un proyecto**

  Para trabajar con TypeORM es necesario instalar la librería junto con los complementos correspondientes a la base de datos con la cual trabaja nuestra aplicación (PostgreSQL).

  * #### **Configuración de Dependencias**
    
    ```bash
    npm install typeorm

    npm install reflect-metadata

    npm install pg
  
    npx typeorm init --name demoTypeORM --database postgres
    ```

  * #### **Configuración de Typescript (tsconfig.json)**
  
    ```json
    "emitDecoratorMetaData": true,
    "experimentalDecorators": true
    ```

  * #### **Configuración de la Base De Datos**
  
    ```typescript
    // ./config/appDataSource.ts

    import { DataSource } from 'typeorm'
 
    export const AppDataSource = new DataSource({
      type: "postgres",
      host: "localhost",
      port: 5432,
      username: "yourUsername",
      password: "yourPassword",
      database: "yourDatabase",
      synchronize: true,
      logging: true,
      entities: [Entity1, Entity2, EntityN],
      subscribers: [],
      migrations: [],
    })

    // ./otroModulo.ts

    import { AppDataSource } from './config/appDataSource.ts'

    AppDataSource.initialize()
      .then(res => {
         console.log("Conexion A la Base de Datos exitosa.")
    })
    ```

> ## ***Definición de modelos***

* ### **Creación de entidades y modelos**

  Las entidades representan objetos reales con propiedades y relaciones.

  Un modelo, por su parte, corresponde a una clase de TypeScript que define cómo se verán y se comportarán las entidades en tu aplicación. Estos modelos se usan para crear, leer, actualizar y eliminar datos.

  * #### **tsconfig**
  
    ```json
    "strictPropertyInitialization": false
    ```

  * #### **Entidad**
    ```typescript
    import { Entity, Column, PrimaryColumn, PrimaryGeneratedColumn } from 'typeorm'

    @Entity({ // Crea una tabla
        name: "tableName" // Le asigna este nombre a la tabla 
    })
    export class myClass {

      @PrimaryColumn() // Define la columna como la Primary Key
      id_property: number,

      @PrimaryGeneratedColumn() // Define la columna como la Primary Key y la autogenera.
      id_propertyGenrated: number,
      
      @Column() // Crea una columna con la propiedad
      property1: number,

      @Column()
      property2: number,

      @Column()
      property3: boolean,
    }
    ```

> ## ***Operaciones básicas***

* ### **Inserción de datos**

  ```typescript
  import { AppDataSource } from './config/appDataSource.ts'
  async function getAllEntities(req: Request, res: Response): void {
    const entities: myClass[] = await AppDataSource.getRepository(myClass).find() // Devuelve todas las filas de la tabla myClass

    AppDataSource.getRepository(myClass).findOneBy({property: value}) // Devuelve el primer registro con ese valor en la propiedad especificada de la tabla myClass
  }

  async function createEntity(req: Request<{ classData }>, res: Response): void {
    const entity: myClass = await AppDataSource.getRepository(myClass).create(classData) // Crea un registro con la estructura de myClass

    const result = await  AppDataSource.getRepository(myClass).save(entity) // Guarda el registro creado en la tabla myClass
  }
  ```

> ## ***Relaciones***

* ### **Relaciones entre entidades**

  En la bases de datos relacionales trabajamos con diversas tablas las cuales pueden estar relacionadas entre sí mediante diferentes tipos de cardinalidad. Dentro de TypeORM la definición del tipo de relación es muy sencilla gracias al uso de decoradores.

  * #### **Uno a uno mediante @OneToOne.**
  
    ```typescript
    import { Entity, Column, PrimaryColumn, PrimaryGeneratedColumn } from 'typeorm'

    @Entity({
        name: "tableName"
    })
    export class MyClass {

      @PrimaryColumn()
      id_property: number,
      
      @Column()
      property1: number,

      @Column()
      property2: number,

      @Column()
      property3: boolean,

      @OneToOne(() => OtherClass)
      @JoinColumn()
      otherClass: OtherClass
    }
    ```

  * #### **Uno a muchos mediante @OneToMany.**

    ```typescript
    import { Entity, Column, PrimaryColumn, PrimaryGeneratedColumn } from 'typeorm'

    @Entity({
        name: "tableName" 
    })
    export class MyClass {

      @PrimaryColumn()
      id_property: number,
      
      @Column()
      property1: number,

      @Column()
      property2: number,

      @Column()
      property3: boolean,

      @OneToMany(() => OtherClass, (otherClass) => otherClass.id_property)
      otherClassManyEntities: OtherClass[]
    }
    ```
  
  * #### **Muchos a uno mediante @ManyToOne.**

    ```typescript
    import { Entity, Column, PrimaryColumn, PrimaryGeneratedColumn } from 'typeorm'

    @Entity({
        name: "tableName"
    })
    export class MyClass {

      @PrimaryColumn()
      id_property: number,
      
      @Column()
      property1: number,

      @Column()
      property2: number,

      @Column()
      property3: boolean,

      @ManyToOne(() => MyClass, (myClass) => myClass.otherClassproperty)
      myClassOneEntity: MyClass
    }
    ```

  * #### **Muchos a muchos mediante @ManyToMany.**
***

> ## ***Cierre***

* ### En conclusión...

  * ***Exploramos la gestión de TypeORM:*** Una herramienta para el mapeo objeto-relacional entre nuestro código y una base de datos relacional, caracterizada por su robustez y eficiencia en la comunicación entre un servidor y su respectiva base de datos. 

  * ***Aprendimos a definir Modelos y Entidades:***, que se traducirán en las distintas tablas, a definir los atributos y sus tipos para cada modelo, a agregar y consultar registros de las tablas y a conocer las diferentes maneras de relacionarlas a partir de su cardinalidad.

  ![TypeORM](./cierreTypeORM.png)
***