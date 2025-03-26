# Guia-instalación
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
  Conectarse con autenticacion de Windows
     
   ```Shell
      sqlcmd -S localhost\SQLEXPRESS -E
      EXEC xp_instance_regwrite N'HKEY_LOCAL_MACHINE', N'Software\Microsoft\MSSQLServer\MSSQLServer', N'LoginMode', REG_DWORD, 2;
      Restart-Service "MSSQL$SQLEXPRESS"
   ```
Verificar y conectarse
     ```Shell
     sqlcmd -S localhost\SQLEXPRESS -U sa -P "NuevaContraseñaFuerte123" -Q "SELECT '¡Conexión exitosa!' AS Resultado"
     ```

 
  
