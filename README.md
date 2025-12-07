# FTP-MySQL
Proyecto de acceso a ficheros con FTP+MySQL de 2ºASIR en SRI



Primero, prepararemos donde se guardaran nuestros usuarios para la autenticación de nuestro server FTP.

Instalaremos MariaDB/MySQL en Debian.

<img width="548" height="134" alt="imagen" src="https://github.com/user-attachments/assets/6eaa6329-28bb-4b52-aa9c-e242dccb2989" />

Por defecto, la base de datos solo se escucha a si misma y hay que dejarle entrar a la maquina del FTP con la siguiente sintaxis en la ruta /etc/mysql/mariadb.conf.d/50-server.cnf

<img width="661" height="83" alt="imagen" src="https://github.com/user-attachments/assets/5acf7055-10da-44c5-8ed7-d1d74a9537cd" />


