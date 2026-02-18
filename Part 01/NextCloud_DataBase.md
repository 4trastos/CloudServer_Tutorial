# Manual de Administración: Base de Datos Nextcloud (Snap)

Este manual contiene los comandos esenciales para gestionar la base de datos de Nextcloud desde la terminal de Ubuntu mediante el cliente MySQL integrado en el paquete Snap.

## Acceso Inicial
Para entrar en la consola de la base de datos:
```bash
sudo nextcloud.mysql-client

```

---

## Comandos de Navegación (Básicos)

*Recuerda que todos los comandos dentro de MySQL deben terminar en `;`.*

| Comando | Acción |
| --- | --- |
| `SHOW DATABASES;` | Lista todas las bases de datos disponibles. |
| `USE nextcloud;` | Selecciona la base de datos de Nextcloud (Obligatorio). |
| `SHOW TABLES;` | Lista todas las tablas (ej: `oc_users`, `oc_bruteforce_attempts`). |
| `DESCRIBE nombre_tabla;` | Muestra las columnas y tipos de datos de una tabla. |
| `exit` | Sale de la consola MySQL y vuelve a Ubuntu. |

---

## Seguridad y Desbloqueos (Brute Force)

Si recibes el error *"Demasiadas peticiones desde su red"*, usa estos comandos:

1. **Ver quién está bloqueado:**
```sql
SELECT ip, COUNT(*) FROM oc_bruteforce_attempts GROUP BY ip;

```


2. **Borrar todos los bloqueos (Limpiar tabla):**
```sql
DELETE FROM oc_bruteforce_attempts;

```


3. **Borrar una IP específica:**
```sql
DELETE FROM oc_bruteforce_attempts WHERE ip = '192.168.1.1';

```



---

## Gestión de Usuarios

Consultas útiles para verificar el estado de las cuentas:

* **Listar todos los usuarios y sus nombres:**
```sql
SELECT uid, displayname FROM oc_users;

```


* **Ver el correo electrónico asociado a un usuario:**
```sql
SELECT uid, configvalue FROM oc_preferences WHERE appid = 'settings' AND configkey = 'email';

```



---

## Almacenamiento y Archivos

*Nota: No modifiques estas tablas manualmente a menos que sepas lo que haces, ya que puedes corromper el índice de archivos.*

* **Ver el tamaño de almacenamiento por usuario (en MB):**
```sql
SELECT user, ROUND(SUM(size) / 1048576, 2) AS size_mb FROM oc_filecache GROUP BY user;

```


* **Listar almacenamientos externos configurados:**
```sql
SELECT * FROM oc_storages;

```



---

## Configuración de Aplicaciones (`oc_appconfig`)

Ideal para desactivar apps que causan errores de inicio (como el error 500).

* **Ver configuración de una app específica:**
```sql
SELECT * FROM oc_appconfig WHERE appid = 'bruteforcesettings';

```


* **Desactivar el mantenimiento (si se queda atascado):**
```sql
UPDATE oc_appconfig SET configvalue = 'no' WHERE appid = 'core' AND configkey = 'maintenance';

```



---

## Reglas de Oro

1. **SELECT antes que DELETE:** Antes de borrar una fila, haz un `SELECT` con el mismo filtro para confirmar qué vas a borrar.
2. **Flecha `->`:** Si ves este símbolo, es que olvidaste cerrar el comando con `;`. Escríbelo y pulsa Enter.
3. **Ctrl + C:** Si te bloqueas dentro de la consola, pulsa esta combinación para forzar la salida.

---

*Manual creado para el servidor de Australair - 2026*

```

---
Como estamos en un entorno de producción, antes de hacer cambios grandes en la base de datos (como un `DELETE` masivo), hay que hacer un backup rápido de todo el Nextcloud con este comando de Snap:
```bash
sudo nextcloud.export

```

Esto guardará una copia de seguridad completa (incluida la base de datos) en `/var/snap/nextcloud/common/backups/`.
