## Español 🇪🇸 

# Configuración de un Servidor Ubuntu para Servicios en la Nube

Este proyecto consiste en la configuración de un servidor Ubuntu para soportar servicios en la nube, con un enfoque en la instalación y administración de **Nextcloud**, una potente solución de almacenamiento y colaboración en la nube.

## Contenidos

### 1. Instalación y Administración de Nextcloud
Nextcloud permite gestionar y almacenar archivos, sincronizar datos entre dispositivos y colaborar con otros usuarios de forma segura. En este proyecto, se implementará y administrará Nextcloud en un servidor Ubuntu.

### 2. Virtualización con VirtualBox
Se incluye la implementación y manejo de un servidor virtualizado utilizando **VirtualBox**, permitiendo crear y administrar máquinas virtuales para simular entornos de prueba y producción.

### 3. Buenas Prácticas en Seguridad y Redes
Siguiendo las recomendaciones del proyecto **Born2beroot**, se aplicarán buenas prácticas de seguridad y configuración de redes, reforzando la seguridad del servidor y su entorno.

### 4. Gestión de Redes y Servicios
Se cubrirán aspectos de la gestión de redes y servicios a nivel básico e intermedio, basados en el proyecto **Netpractice**, mejorando la comprensión y administración de redes en un servidor.

## Requisitos

- **Ubuntu Server**
- **VirtualBox** instalado en el sistema anfitrión
- Conexión a Internet
- Familiaridad con línea de comandos

---

Aquí tienes un listado completo y profesional de los **comandos `occ` (ownCloud Console Command)** disponibles en Nextcloud para gestión de usuarios y administración general — **especialmente útil en instalaciones Snap como la tuya**:

---

## 🧑‍💻 **1. Comandos para Gestión de Usuarios**

| Comando                                           | Descripción                                            |
| ------------------------------------------------- | ------------------------------------------------------ |
| `sudo nextcloud.occ user:list`                    | Lista todos los usuarios.                              |
| `sudo nextcloud.occ user:add <usuario>`           | Crea un nuevo usuario (te pedirá nombre y contraseña). |
| `sudo nextcloud.occ user:delete <usuario>`        | Elimina un usuario.                                    |
| `sudo nextcloud.occ user:resetpassword <usuario>` | Cambia la contraseña de un usuario.                    |
| `sudo nextcloud.occ user:report`                  | Muestra estadísticas generales de usuarios.            |
| `sudo nextcloud.occ user:info <usuario>`          | Muestra datos completos de usuarios.                   |

🔎 **Importante:** El `<usuario>` debe ser el **nombre de usuario interno** (no el nombre completo).

Para encontrar el nombre interno exacto, ejecuta:

```bash
sudo nextcloud.occ user:list --output=json
```

---

## 🔐 **2. Gestión de Grupos**

| Comando                                                 | Descripción                   |
| ------------------------------------------------------- | ----------------------------- |
| `sudo nextcloud.occ group:list`                         | Lista todos los grupos.       |
| `sudo nextcloud.occ group:add <grupo>`                  | Crea un grupo nuevo.          |
| `sudo nextcloud.occ group:delete <grupo>`               | Elimina un grupo.             |
| `sudo nextcloud.occ group:adduser <grupo> <usuario>`    | Añade un usuario a un grupo.  |
| `sudo nextcloud.occ group:removeuser <grupo> <usuario>` | Quita un usuario de un grupo. |

---

## 🛠️ **3. Comandos Administrativos Generales**

| Comando                                              | Descripción                                                     |
| ---------------------------------------------------- | --------------------------------------------------------------- |
| `sudo nextcloud.occ status`                          | Estado general del sistema (versión, base de datos, apps, etc). |
| `sudo nextcloud.occ check`                           | Verifica la instalación de Nextcloud.                           |
| `sudo nextcloud.occ config:list`                     | Muestra toda la configuración.                                  |
| `sudo nextcloud.occ config:system:get <clave>`       | Muestra valor específico del archivo `config.php`.              |
| `sudo nextcloud.occ maintenance:mode --on` / `--off` | Activa o desactiva modo mantenimiento.                          |
| `sudo nextcloud.occ log:tail`                        | Muestra logs en tiempo real.                                    |
| `sudo nextcloud.occ update:check`                    | Verifica si hay actualizaciones.                                |

---

## 🧩 **4. Aplicaciones (Apps)**

| Comando                                | Descripción                                  |
| -------------------------------------- | -------------------------------------------- |
| `sudo nextcloud.occ app:list`          | Lista todas las apps instaladas y su estado. |
| `sudo nextcloud.occ app:enable <app>`  | Habilita una app.                            |
| `sudo nextcloud.occ app:disable <app>` | Desactiva una app.                           |
| `sudo nextcloud.occ app:install <app>` | Instala una nueva app.                       |
| `sudo nextcloud.occ app:remove <app>`  | Elimina una app.                             |

---

## ☁️ **5. Otras Utilidades útiles**

| Comando                                   | Descripción                                                             |
| ----------------------------------------- | ----------------------------------------------------------------------- |
| `sudo nextcloud.occ files:scan --all`     | Escanea todos los archivos de todos los usuarios (reconstruye índices). |
| `sudo nextcloud.occ files:scan <usuario>` | Escanea solo los archivos de un usuario.                                |
| `sudo nextcloud.occ files:cleanup`        | Limpia archivos orfanados.                                              |
| `sudo nextcloud.occ background:cron`      | Ejecuta tareas programadas (CRON) manualmente.                          |
| `sudo nextcloud.occ encryption:disable`   | Desactiva el cifrado de archivos (si está habilitado).                  |



---

## English 🇬🇧

# Ubuntu Server Configuration for Cloud Services

This project focuses on configuring an Ubuntu server to support cloud services, with a particular focus on the installation and management of **Nextcloud**, a powerful cloud storage and collaboration solution.

## Contents

### 1. Installation and Management of Nextcloud
Nextcloud allows you to manage and store files, synchronize data between devices, and collaborate securely with other users. In this project, Nextcloud will be implemented and managed on an Ubuntu server.

### 2. Virtualization with VirtualBox
The project includes the implementation and management of a virtualized server using **VirtualBox**, allowing for the creation and administration of virtual machines to simulate testing and production environments.

### 3. Best Practices in Security and Networking
Following the guidelines of the **Born2beroot** project, best practices in security and network configuration will be applied, strengthening the server and its environment.

### 4. Network and Service Management
Basic and intermediate levels of network and service management will be covered, based on the **Netpractice** project, enhancing understanding and administration of server networks.

## Requirements

- **Ubuntu Server**
- **VirtualBox** installed on the host system
- Internet connection
- Familiarity with the command line
