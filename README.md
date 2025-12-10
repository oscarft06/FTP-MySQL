# FTP-MySQL
Proyecto de acceso a ficheros con FTP+MySQL de 2ºASIR en SRI



Configuración de las máquinas en AWS

MV VSFTPD

Security Group:

<img width="1432" height="396" alt="imagen" src="https://github.com/user-attachments/assets/4a6c58ef-c559-4bb1-b587-34904adc3c67" />











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

INSERT INTO vsftpd.usuarios (nombre, passwd) VALUES ('alumno', PASSWORD('1234'));

-- Crear el usuario del sistema DB que usará el FTP para conectarse
-- Le damos permiso desde la IP de la MV 1

CREATE USER 'vsftpd_db_user'@'192.168.1.10' IDENTIFIED BY 'admin';
GRANT SELECT ON vsftpd.* TO 'vsftpd_db_user'@'192.168.1.10';
FLUSH PRIVILEGES;
EXIT;

Se veria algo asi:

<img width="520" height="161" alt="imagen" src="https://github.com/user-attachments/assets/7755845d-f90a-486d-a5f7-da1ad647376a" />


Ahora instalaremos el servicio VSFPTD en la MV1:

<img width="646" height="165" alt="imagen" src="https://github.com/user-attachments/assets/891db895-04f8-465f-bff2-544bfeb75a29" />


Los usuarios virtuales, necesitan mapearse a un usuario local de Linux que tenga pocos permisos, entonces, crearemos el siguiente usuario con la siguiente sintaxis:

<img width="933" height="266" alt="imagen" src="https://github.com/user-attachments/assets/5009b230-8f86-49a9-aeca-37dcf0cca506" />


Crearemos el directorio para el usuario creado en la base de datos (cada usuario que tengamos en la base de datos, si queremos que tengan acceso a vsftpd, tendremos que crear su directorio dentro del directorio del usuario invitado/mapeado). También le pondremos permisos para que pueda escribir y le cambiaremos el usuario y grupo propietario al directorio del usuario de la base de datos ya que va a heredar los permisos del usuario invitado/mapeado.

<img width="497" height="138" alt="imagen" src="https://github.com/user-attachments/assets/c20636af-1def-4fca-b105-7cc0932c1083" />


Editaremos el fichero situado en /etc/vsftpd.conf con los siguientes parametros para enjaular a los usuarios de la base de datos, puedan escribir en sus directorios, activar el modo pasivo y los nombre de ficheros/carpetas deben verse correctamente(acentos, eñes...) activando utf8. Por último, tendremos que añadir la IP elástica que le hemos asociado a nuestra máquina FTP para poder conectarnos a través de un cliente FTP. Así, evitamos que cada vez que nos conectemos al Lab de AWS, no cambie la IP pública de la máquina.

<img width="668" height="528" alt="imagen" src="https://github.com/user-attachments/assets/7e13044e-eb0d-49af-92fa-460c1ec3937e" />


Ahora instalaremos el servicio libpam-mysql, porque necesitmos que el servicio vsftpd hable con MySQL:

<img width="599" height="118" alt="imagen" src="https://github.com/user-attachments/assets/69d7d1da-2fd0-4e3c-8ede-ba45b1941400" />


Ahora configuraremos el fichero /etc/pam.d/vsftpd y agregaremos las siguientes lineas para que el modulo pam pueda verificar la autenticacion y el nombre de usuario o la cuenta almacenada en la base de datos:

<img width="1300" height="88" alt="imagen" src="https://github.com/user-attachments/assets/74d64d17-7458-4fed-853a-dcc22b602985" />





