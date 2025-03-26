# Guia de instalación
Guia de instalación para prueba tecnica

## Instalar SQL Server
1. Instalar SQL Server
2. Ejecutar en PowerShell o CMD
   
    ```Shell
      sqlcmd -?
    ```
4. Si no se reconoce el comando, asegúrate de que el directorio de sqlcmd esté en el PATH
   
   ```Shell
   C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\Tools\Binn\sqlcmd.exe
   ```
5. Conectarse en windows
   
   ```Shell
    sqlcmd -S localhost\SQLEXPRESS -E
   ```
6. Establecer contraseña usuario sa

    ```Shell
       Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\MSSQL*.SQLEXPRESS\MSSQLServer\" -Name "LoginMode" -Value 2
       Restart-Service "MSSQL$SQLEXPRESS"
       sqlcmd -S localhost\SQLEXPRESS -E -Q "ALTER LOGIN sa WITH PASSWORD='NuevaContraseñaFuerte123', CHECK_POLICY=OFF; ALTER LOGIN sa ENABLE;"
       sqlcmd -S localhost\SQLEXPRESS -E -Q "SELECT name, is_disabled FROM sys.sql_logins WHERE name = 'sa'"
    ```
9. Conectarse con autenticacion de Windows
     
     ```Shell
         sqlcmd -S localhost\SQLEXPRESS -E
         EXEC xp_instance_regwrite N'HKEY_LOCAL_MACHINE', N'Software\Microsoft\MSSQLServer\MSSQLServer', N'LoginMode', REG_DWORD, 2;
         Restart-Service "MSSQL$SQLEXPRESS"
      ```
7. Verificar y conectarse

     ```Shell
     sqlcmd -S localhost\SQLEXPRESS -U sa -P "NuevaContraseñaFuerte123" -Q "SELECT '¡Conexión exitosa!' AS Resultado"
     ```

Activar TCP/IP 
![image](https://github.com/user-attachments/assets/c5368d5c-a948-4df1-9b03-166923d304d6)
ELIMINAR EL PUERTO DINAMICO Y DEJAR EL ESTATICO (1433)

Poner TCP PORT a 1433 y luego reiniciar
![image](https://github.com/user-attachments/assets/8a7b4023-9082-4df8-8269-7e401a7f8c72)



## Sequalize y ExpressJS
1. Instalar dependencias
```Shell
mkdir my-api
cd my-api
npm init -y

npm install express sequelize tedious mssql
npm install --save-dev typescript @types/express @types/sequelize ts-node nodemon
npm install --save-dev typescript ts-node-dev @types/express @types/node
npm install --save-dev sequelize-cli
```
2. Crear un `tsconfig.json`

```Javascript
{
  "compilerOptions": {
    "target": "ES6",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
```
 3. Continuar instalación
```Shell
npx sequelize-cli init
```
4. Configurar sequalize + Typescript
Crear un archivo `.sequelizerc`
```Javascript
const path = require('path');

module.exports = {
  config: path.resolve('config', 'config.js'),
  'models-path': path.resolve('src', 'models'),
  'seeders-path': path.resolve('seeders'),
  'migrations-path': path.resolve('migrations')
};
```
5. Configurar SQL Server conecction (eliminar config/config.json a config/config.js
```Javascript
require("dotenv").config();

module.exports = {
  development: {
    username: process.env.DB_USER || "sa",
    password: process.env.DB_PASSWORD || "NuevaContraseñaFuerte123",
    database: process.env.DB_NAME || "task_manager_db",
    host: process.env.DB_HOST || "localhost",
    port: process.env.DB_PORT || 1433,
    dialect: "mssql",
    dialectOptions: {
      options: {
        encrypt: true,
        trustServerCertificate: true,
      },
    },
  },
};

```

6. Crear la bd
```shell
npx sequelize-cli db:create
```

EXTRAS

```shell
npm install --save-dev ts-node-dev
```


7. Flujo para migraciones
Añadir el nuevo campo al modelo
Generar migracion
```
npx sequelize-cli migration:generate --name add-description-to-tasks
```
Editar el campo de up y down

✅ 1. Agregar una columna (addColumn)
```
'use strict';

/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.addColumn("Tasks", "description", {
      type: Sequelize.TEXT, // <-- Aquí usamos Sequelize.TEXT en lugar de DataTypes.TEXT
      allowNull: true,
    });
  },

  async down(queryInterface, Sequelize) {
    await queryInterface.removeColumn("Tasks", "description");
  }
};

```

✅ 2. Eliminar una columna (removeColumn)
```
up: async (queryInterface: QueryInterface) => {
  await queryInterface.removeColumn("Tasks", "old_column");
},

down: async (queryInterface: QueryInterface) => {
  await queryInterface.addColumn("Tasks", "old_column", {
    type: DataTypes.STRING,
    allowNull: true,
  });
}
```

✅ 3. Renombrar una columna (renameColumn)
```
up: async (queryInterface: QueryInterface) => {
  await queryInterface.renameColumn("Tasks", "oldName", "newName");
},

down: async (queryInterface: QueryInterface) => {
  await queryInterface.renameColumn("Tasks", "newName", "oldName");
}
```

✅ 4. Crear una nueva tabla (createTable)
```
up: async (queryInterface: QueryInterface) => {
  await queryInterface.createTable("Projects", {
    id: {
      type: DataTypes.INTEGER,
      autoIncrement: true,
      primaryKey: true,
    },
    name: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    createdAt: {
      type: DataTypes.DATE,
      allowNull: false,
    },
    updatedAt: {
      type: DataTypes.DATE,
      allowNull: false,
    }
  });
},

down: async (queryInterface: QueryInterface) => {
  await queryInterface.dropTable("Projects");
}
```
✅ 5. Agregar una clave foránea (addConstraint)

 ```
up: async (queryInterface: QueryInterface) => {
  await queryInterface.addConstraint("Tasks", {
    fields: ["projectId"],
    type: "foreign key",
    name: "fk_tasks_project",
    references: {
      table: "Projects",
      field: "id",
    },
    onDelete: "CASCADE",
    onUpdate: "CASCADE",
  });
},

down: async (queryInterface: QueryInterface) => {
  await queryInterface.removeConstraint("Tasks", "fk_tasks_project");
}
```
Subir migracion
```
npx sequelize-cli db:migrate
```
