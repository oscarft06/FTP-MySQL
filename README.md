# FTP-MySQL
Proyecto de acceso a ficheros con FTP+MySQL de 2ºASIR en SRI



Primero, prepararemos donde se guardaran nuestros usuarios para la autenticación de nuestro server FTP. Para ello necesitaremos dos maquinas que puedan escucharse entre si. Empezaremos por la MV2 donde se encontrará la base de datos.

Instalaremos MariaDB/MySQL en Debian.

<img width="548" height="134" alt="imagen" src="https://github.com/user-attachments/assets/6eaa6329-28bb-4b52-aa9c-e242dccb2989" />

Por defecto, la base de datos solo se escucha a si misma y hay que dejarle entrar a la maquina del FTP con la siguiente sintaxis en la ruta /etc/mysql/mariadb.conf.d/50-server.cnf

<img width="636" height="80" alt="imagen" src="https://github.com/user-attachments/assets/83498270-8f8d-43cd-a465-f6d895c699a3" />

Reiniciaremos el servicio:

<img width="700" height="129" alt="imagen" src="https://github.com/user-attachments/assets/58fc63eb-3b0c-4636-9515-aba21f4c82a1" />

Crearemos la base de datos y el usuario para la base de datos con la siguiente sintaxis:

-- Crear la base de datos

CREATE DATABASE vsftpd;

-- Crear la tabla de usuarios

CREATE TABLE vsftpd.usuarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    passwd VARCHAR(50) NOT NULL
);

-- Crear un usuario de prueba (contraseña '1234')
-- Usamos la función PASSWORD() o texto plano para simplificar la conexión PAM inicial

INSERT INTO vsftpd.usuarios (nombre, password) VALUES ('alumno', PASSWORD('1234'));

-- Crear el usuario del sistema DB que usará el FTP para conectarse
-- Le damos permiso desde la IP de la MV 1

CREATE USER 'vsftpd_db_user'@'192.168.1.10' IDENTIFIED BY 'clave_secreta_db';
GRANT SELECT ON vsftpd.* TO 'vsftpd_db_user'@'192.168.1.10';
FLUSH PRIVILEGES;
EXIT;

Se veria algo asi:

<img width="520" height="161" alt="imagen" src="https://github.com/user-attachments/assets/7755845d-f90a-486d-a5f7-da1ad647376a" />

Ahora instalaremos el servicio VSFPTD en la MV1:

<img width="576" height="132" alt="imagen" src="https://github.com/user-attachments/assets/e39106aa-2af6-426a-ad95-c620fd5e2e6c" />


Los usuarios virtuales, necesitan mapearse a un usuario local de Linux que tenga pocos permisos:

<img width="792" height="88" alt="imagen" src="https://github.com/user-attachments/assets/18391a40-875f-407c-9c87-661dc83cf2cd" />


