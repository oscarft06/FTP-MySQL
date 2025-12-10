# FTP-MySQL
Proyecto de acceso a ficheros con FTP + MySQL de 2º ASIR en SRI.

---

## 1. Configuración de las máquinas en AWS

### MV VSFTPD
**Security Group:**

<p align="center">
  <img src="https://github.com/user-attachments/assets/4a6c58ef-c559-4bb1-b587-34904adc3c67" width="800" alt="Security Group VSFTPD" />
</p>

**IP elástica asociada a nuestra MV VSFTPD:**

<p align="center">
  <img src="https://github.com/user-attachments/assets/10b305ea-d5d8-4470-a0e2-59414362517a" width="800" alt="IP Elastica" />
</p>

### MV MariaDB
**Security Group:**

<p align="center">
  <img src="https://github.com/user-attachments/assets/fab2880e-7a9d-4b1f-96e9-ca3faccf0488" width="800" alt="Security Group MariaDB" />
</p>

---

## 2. Configuración de la Base de Datos (MV2)

Primero prepararemos el almacenamiento de usuarios para la autenticación del servidor FTP. Necesitaremos que las dos máquinas tengan conectividad entre sí. Empezaremos por la MV2 donde se alojará la base de datos.

### Instalación de MariaDB/MySQL
Instalamos el servicio en Debian:

<p align="center">
  <img src="https://github.com/user-attachments/assets/6eaa6329-28bb-4b52-aa9c-e242dccb2989" width="800" alt="Instalación MariaDB" />
</p>

### Configuración de escucha remota
Por defecto, la base de datos solo escucha conexiones locales (localhost). Debemos permitir la conexión desde la máquina del FTP editando la ruta `/etc/mysql/mariadb.conf.d/50-server.cnf`:

<p align="center">
  <img src="https://github.com/user-attachments/assets/83498270-8f8d-43cd-a465-f6d895c699a3" width="800" alt="Configuración bind-address" />
</p>

Reiniciamos el servicio para aplicar cambios:

<p align="center">
  <img src="https://github.com/user-attachments/assets/58fc63eb-3b0c-4636-9515-aba21f4c82a1" width="800" alt="Reinicio servicio" />
</p>

### Creación de Base de Datos y Usuarios
Ejecutamos las sentencias SQL para crear la estructura y los permisos necesarios:

```sql
-- Crear la base de datos
CREATE DATABASE vsftpd;

-- Crear la tabla de usuarios
CREATE TABLE vsftpd.usuarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    passwd VARCHAR(50) NOT NULL
);

-- Crear un usuario de prueba (contraseña 'oscar')
INSERT INTO vsftpd.usuarios (nombre, passwd) VALUES ('oscar', PASSWORD('oscar'));

-- Crear el usuario del sistema DB que usará el FTP para conectarse
-- Le damos permiso desde la IP de la MV 1 (Sustituir ip_MVvsftpd por la IP real)
CREATE USER 'vsftpduser'@'ip_MVvsftpd' IDENTIFIED BY 'ftp';
GRANT SELECT ON vsftpd.* TO 'vsftpd_db_user'@'ip_MVvsftpd';
FLUSH PRIVILEGES;
EXIT;
```

Resultado visual en la terminal:

<p align="center">
  <img src="https://github.com/user-attachments/assets/7755845d-f90a-486d-a5f7-da1ad647376a" width="800" alt="Resultado SQL" />
</p>

---

## 3. Instalación y Configuración del Servicio FTP (MV1)

### Instalación de VSFTPD
Procedemos a instalar el servidor FTP:

<p align="center">
  <img src="https://github.com/user-attachments/assets/891db895-04f8-465f-bff2-544bfeb75a29" width="800" alt="Instalación VSFTPD" />
</p>

### Mapeo de Usuarios Virtuales
Los usuarios virtuales necesitan mapearse a un usuario local de Linux con permisos restringidos. Creamos el usuario:

<p align="center">
  <img src="https://github.com/user-attachments/assets/5009b230-8f86-49a9-aeca-37dcf0cca506" width="800" alt="Creación usuario local" />
</p>

### Configuración de Directorios
Creamos el directorio para el usuario de la base de datos.
> **Nota:** Por cada usuario en la BD con acceso a FTP, debemos crear su directorio dentro de la carpeta del usuario mapeado.

Asignamos permisos de escritura y cambiamos el propietario al usuario de la base de datos (heredará permisos del usuario invitado):

<p align="center">
  <img src="https://github.com/user-attachments/assets/c20636af-1def-4fca-b105-7cc0932c1083" width="800" alt="Permisos directorios" />
</p>

### Edición de `vsftpd.conf`
Editamos `/etc/vsftpd.conf` para:
1. Enjaular usuarios.
2. Permitir escritura.
3. Activar modo pasivo.
4. Forzar UTF-8.
5. Definir la IP elástica (para evitar problemas de conexión en AWS al reiniciar).

<p align="center">
  <img src="https://github.com/user-attachments/assets/7e13044e-eb0d-49af-92fa-460c1ec3937e" width="800" alt="Configuración vsftpd.conf" />
</p>

---

## 4. Conexión PAM con MySQL

Instalamos el módulo `libpam-mysql` para permitir la comunicación entre VSFTPD y la base de datos:

<p align="center">
  <img src="https://github.com/user-attachments/assets/69d7d1da-2fd0-4e3c-8ede-ba45b1941400" width="800" alt="Instalación libpam" />
</p>

Configuramos el fichero `/etc/pam.d/vsftpd` agregando las líneas necesarias para que el módulo PAM verifique la autenticación contra los datos almacenados en MySQL:

<p align="center">
  <img src="https://github.com/user-attachments/assets/74d64d17-7458-4fed-853a-dcc22b602985" width="800" alt="Configuración PAM" />
</p>
