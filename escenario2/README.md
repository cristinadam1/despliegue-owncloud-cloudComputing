# Documentación de la práctica - Escenario 2 

### Nombre del alumno: Cristina del Águila Martín

## Entorno de desarrollo y de producción utilizado

### Entorno de desarrollo:
- **Sistema Operativo**: macOS Sonoma 14.2
- **Versión de Docker**: 27.3.1
- **Gestor de contenedores**: Docker

## Descripción de la práctica
### 2.1. Objetivo
El objetivo de esta práctica es desplegar un servicio OwnCloud en contenedores con alta disponibilidad, incluyendo los siguientes componentes clave:

- **Balanceo de carga con HAProxy**.
- **Servidores web OwnCloud**.
- **Base de datos MariaDB** con replicación.
- **Servicio de cache Redis**.
- **Autenticación LDAP** para gestionar usuarios y grupos.

### 2.2. Problema a resolver
Lo que hay que hacer es orquestar un sistema de contenedores que se encargue de:
- Gestionar la autenticación de usuarios mediante LDAP.
- Mantener la alta disponibilidad de todos los servicios a través de balanceadores de carga y replicación.
- Gestionar grandes volúmenes de datos de usuarios (documentos, archivos) de forma eficiente y segura.

El diagrama que representa la arquitectura que se propone es el siguiente:

![cap12](/escenario2/img/c12.png)

## 3. Servicios desplegados y su configuración

### 3.1. Arquitectura general

La arquitectura se distribuye de la siguiente forma:
- **Balanceador de carga (HAProxy):** Redirige las solicitudes de los usuarios a los servidores web OwnCloud.
- **Servidores web OwnCloud (2 réplicas):** Se despliegan en contenedores Docker para manejar las peticiones de los usuarios.
- **Base de datos MariaDB (replicación maestro-esclavo):** Almacena los datos de OwnCloud y se replica para asegurar la disponibilidad.
- **Redis (Cache):** Se utiliza como almacenamiento en caché para mejorar el rendimiento de OwnCloud.
- **LDAP:** Se configura para gestionar la autenticación y los permisos de los usuarios.

### 3.2. Docker Compose

En primer lugar, creo una red de docker 

    docker network create owncloud_network

El archivo `docker-compose.yml` describe todos los servicios necesarios para la configuración de la infraestructura.
A continuación lo explico detalladamente: 

#### **Redes y Volúmenes**

- Redes: he creado una red llamada `owncloud_network` usando el controlador `bridge`, que permite que los contenedores se comuniquen entre sí a través de una red virtual privada. Al usar este tipo de red, los contenedores pueden interactuar entre sí de manera eficiente sin exponerlos directamente a la red externa.

- Volúmenes:
He definido cuatro volúmenes persistentes: `owncloud_data`, `mariadb_data`, `ldap_dat`, y `ldap_config`. Los volúmenes son esenciales para la persistencia de los datos dentro de los contenedores. Esto significa que los datos generados o almacenados por los contenedores no se perderán si los contenedores se detenidos o eliminan. Cada volumen se asocia con un servicio específico:
  - owncloud_data: Almacena los datos de OwnCloud.
  - mariadb_data: Almacena los datos de la base de datos MariaDB.
  - ldap_data: Almacena los datos de LDAP.
  - ldap_config: Almacena la configuración de LDAP.

    ![cap15](/escenario2/img/c15.png)

#### **Servicios**

El bloque de servicios define todos los componentes que despliego como contenedores en el entorno de Docker.

- HAProxy (Balanceador de Carga)
Distribuye el tráfico entre dos instancias de OwnCloud (owncloud1 y owncloud2). Usa la imagen `haproxytech/haproxy-alpine:2.4`, y está conectado a la red `owncloud_network`. Expone los puertos `80`, `443` y `1936`. Además, lo he montado en un archivo de configuración personalizado (`haproxy.cfg`) para controlar el balanceo de carga.

- OwnCloud (Instancias replicadas)
He definido dos instancias de OwnCloud (`owncloud1` y `owncloud2`), las dos usando la imagen oficial `owncloud:latest`. Están conectadas a la red `owncloud_network` y exponen el puerto `80`. Ambas dependen de la base de datos MariaDB (`mariadb_main`) y el caché Redis (`redis_cache`). Los volúmenes persistentes aseguran que los datos se mantengan disponibles entre reinicios de contenedores.

- MariaDB (BD con replicación)
La base de datos MariaDB la despliego con dos contenedores. `mariadb_main` es el contenedor principal, que gestiona los datos de OwnCloud. `mariadb_replica` es una réplica esclava para asegurar la alta disponibilidad mediante la replicación. Las dos están conectados a la red `owncloud_network` y usan volúmenes persistentes para almacenar los datos.

- Redis (Caché)
El contenedor `redis_cache` usa la imagen oficial `redis:latest` para proporcionar caché, mejorando el rendimiento de OwnCloud al reducir la carga sobre la base de datos. Está conectado a la red `owncloud_network` y lo usan las instancias de OwnCloud para almacenar datos en caché.

- LDAP 
El contenedor `openldap2`(lo he tenido que renombrar porque openldap ya estaba cogido) usa la imagen `osixia/openldap:1.5.0` para implementar un servidor LDAP. Este servicio se encarga de gestionar la autenticación y autorización de usuarios de manera centralizada. Se conecta a la red `owncloud_network` y expone los puertos `389` (LDAP) y `636` (LDAP seguro), permitiendo la integración con otros servicios como OwnCloud.

## Configuracion de HAProxy
Creo un archivo `haproxy.cfg` en el mismo directorio

## Inicio los servicios
Inicio los servicios usando los comandos (importante que sea donde está el documento de docker-compose.yml) 

    docker-compose up -d

He encontrado un error porque mariadb y redis ya estaban en uso. Por lo que he tenido que cambiar a redis_cache y mariadb_main

Comprobamos que owncloud está funcionando

    docker ps

Para acceder a OwnCloud, abro un navegador y voy a:

    http://localhost

  ![cap16](/escenario2/img/c16.png)
    
Para verificar HAProxy y su distribución de carga uso el comando:

    docker logs haproxy

## Integración de LDAP
Voy a seguir los siguientes pasos (similar al escenario 1)
1. Añadir un servidor OpenLDAP a tu docker-compose
2. Crear un archivo .ldif para importar usuarios, grupos y ou
3. Configurar la integración LDAP desde la web de OwnCloud

Importante cambiarle el nombre al contenedor, ya que tengo openldap creado del escenario 1

    docker-compose up -d

Lo copio

    docker cp ldap/ldif/groups.ldif openldap2:/tmp/groups.ldif
    docker cp ldap/ldif/users.ldif openldap2:/tmp/users.ldif
    docker cp ldap/ldif/ou.ldif openldap2:/tmp/ou.ldif

Accedo al contenedr

    docker exec -it openldap2 bash

Dentro del contenedor, ejecuto la carga:

    ldapadd -x -D "cn=admin,dc=acmetech,dc=local" -w StrongAdminPass123 -f /tmp/users.ldif
    ldapadd -x -D "cn=admin,dc=acmetech,dc=local" -w StrongAdminPass123 -f /tmp/company_structure.ldif

Verifico que el LDAP esté cargado correctamente

    ldapsearch -x -H ldap://localhost -b dc=acmetech,dc=local -D "cn=admin,dc=acmetech,dc=local" -w StrongAdminPass123

![cap4](/escenario2/img/c4.png)

Ahora integro LDAP con Owncloud siguiendo los pasos descritos en el escenario 1: 

![cap5](/escenario2/img/c5.png)
![cap6](/escenario2/img/c6.png)
![cap7](/escenario2/img/c7.png)
![cap8](/escenario2/img/c8.png)

### Estructura general de LDAP

Los archivos `.ldif` que he creado están en el directorio `/ldap/ldif`
El servidor LDAP lo he configurado con una estructura jerárquica que sigue el formato `dc=acmetech,dc=local`. La jerarquía está organizada en dos unidades organizativas principales:

- groups: los grupos de usuarios.
- users: los usuarios del sistema.

Dentro de esta estructura, he definido varios grupos (en `groups.ldif`) y usuarios (en `users.ldif`), organizados de la siguiente manera.

#### 1. Unidades Organizativas (OU)
- **OU de Grupos (ou=groups)**: Esta unidad organizativa contiene todos los grupos de usuarios definidos en el sistema. Cada grupo se representa como un objeto de clase `posixGroup`, y en ella se almacenan atributos como el `cn` (nombre común del grupo), `gidNumber` (número de identificación del grupo) y `memberUid` (miembros del grupo, que hacen referencia a los usuarios del sistema). Esta estructura permite organizar a los usuarios en grupos según su función o departamento dentro de la empresa.

- **OU de Usuarios (ou=users)**: Esta unidad organizativa contiene los objetos de los usuarios, cada uno representado por un objeto de clase `posixAccount` y `inetOrgPerson`. Los atributos típicos de un usuario incluyen uid (identificador único), cn (nombre común), sn (apellido), uidNumber (número de identificación del usuario), gidNumber (número de identificación del grupo al que pertenece), homeDirectory (directorio home), loginShell (shell de inicio), userPassword (contraseña del usuario) y mail (dirección de correo electrónico)

#### 2. Grupos de usuarios
En el archivo `groups.ldif`, he definido varios grupos, cada uno representado por un objeto de clase `posixGroup`. Estos grupos son esenciales para gestionar el acceso y las permisiones de los usuarios dentro de la infraestructura. Los grupos que he definido son los siguientes:

- Grupo IT: representa a los miembros del equipo de IT y está compuesto por los usuarios pedro y juan. Este grupo tiene un gidNumber de 3001 y su DN es cn=IT,ou=groups,dc=acmetech,dc=local.

- Grupo Marketing: Agrupa a los usuarios ana y maria bajo el departamento de marketing. Este grupo tiene un gidNumber de 3002 y su DN es cn=Marketing,ou=groups,dc=acmetech,dc=local.

- Grupo Ventas: Está compuesto por los usuarios maria y juan, que pertenecen al departamento de ventas. Su gidNumber es 3003, y el DN correspondiente es cn=Ventas,ou=groups,dc=acmetech,dc=local.

- Grupo Soporte: Este grupo contiene a los usuarios pedro y sofia, responsables del soporte técnico. Su gidNumber es 3004 y su DN es cn=Soporte,ou=groups,dc=acmetech,dc=local.

- Grupo Recursos Humanos: Representa al grupo de recursos humanos y está compuesto por los usuarios ana y admin1. Este grupo tiene un gidNumber de 3005 y su DN es cn=RecursosHumanos,ou=groups,dc=acmetech,dc=local.

- Grupo Finanzas: Este grupo agrupa a los usuarios carlos y admin2, encargados de las finanzas. Su gidNumber es 3006, y el DN asociado es cn=Finanzas,ou=groups,dc=acmetech,dc=local.

- Grupo Desarrollo: Los usuarios carlos y sofia forman parte del equipo de desarrollo. Este grupo tiene un gidNumber de 3007 y su DN es cn=Desarrollo,ou=groups,dc=acmetech,dc=local.

- Grupo Administradores: Este grupo agrupa a los usuarios admin1 y admin2, que tienen privilegios administrativos en el sistema. El gidNumber de este grupo es 3008 y su DN es cn=Administradores,ou=groups,dc=acmetech,dc=local.

Desde ownCloud, vemos que están los grupos creados:

![cap13](/escenario2/img/c13.png)

#### 3. Usuarios
En el archivo users.ldif, he definido varios usuarios, cada uno representado por un objeto que combina las clases `posixAccount` e `inetOrgPerson`. Estos objetos contienen información detallada sobre cada usuario, como su nombre, su grupo principal (a través del atributo `gidNumber`), su directorio home, su shell de inicio, su contraseña y su correo electrónico. 
Los usuarios que he definido son:

- Usuario Pedro: Tiene el uid de pedro y pertenece al grupo IT (con gidNumber 3001). Su DN es uid=pedro,ou=users,dc=acmetech,dc=local y su correo electrónico es pedro@acmetech.local.

- Usuario Ana: Tiene el uid de ana y pertenece al grupo Marketing (con gidNumber 3002). Su DN es uid=ana,ou=users,dc=acmetech,dc=local y su correo electrónico es ana@acmetech.local.

- Usuario Juan: Su uid es juan y está en el grupo IT (con gidNumber 3001). Su DN es uid=juan,ou=users,dc=acmetech,dc=local y su correo electrónico es juan@acmetech.local.

- Usuario Maria: Su uid es maria y pertenece al grupo Marketing (con gidNumber 3002). El DN asociado a este usuario es uid=maria,ou=users,dc=acmetech,dc=local, y su correo electrónico es maria@acmetech.local.

- Usuario Admin1: Es un usuario administrador con uid=admin1 y pertenece al grupo Administradores (con gidNumber 3008). Su DN es uid=admin1,ou=users,dc=acmetech,dc=local y su correo electrónico es admin1@acmetech.local.

- Usuario Admin2: Similar al usuario anterior, admin2 es también un administrador, con uid=admin2 y perteneciente al grupo Administradores (con gidNumber 3008). Su DN es uid=admin2,ou=users,dc=acmetech,dc=local y su correo electrónico es admin2@acmetech.local.

- Usuario Carlos: Tiene el uid de carlos y pertenece al grupo Desarrollo (con gidNumber 3007). Su DN es uid=carlos,ou=users,dc=acmetech,dc=local, y su correo electrónico es carlos@acmetech.local.

- Usuario Sofia: Tiene el uid de sofia y pertenece al grupo Desarrollo (con gidNumber 3007). El DN de este usuario es uid=sofia,ou=users,dc=acmetech,dc=local, y su correo electrónico es sofia@acmetech.local.

#### 4. Relación entre Grupos y Usuarios

Cada usuario está asociado con un grupo específico a través del atributo `gidNumber`, que indica el grupo principal al que pertenece. Además, los grupos contienen una lista de usuarios en el atributo `memberUid`, que define a qué usuarios pertenece cada grupo.

Por ejemplo:
- El grupo IT contiene a los usuarios pedro y juan, como se refleja en los atributos memberUid del grupo IT.
- El grupo Marketing incluye a los usuarios ana y maria.

## Estadísticas
Si nos vamos a

  http://localhost:1936/stats

Y observamos las estadísticas

![cap14](/escenario2/img/c14.png)

## Problemas encontrados
Haciendo la comprobación con los comando 

    docker inspect owncloud1 | grep -i port
    docker network inspect owncloud_network

Me doy cuenta de que no están conectados a la misma red, los conecto usando: 

    docker network connect owncloud_network haproxy
    docker network connect owncloud_network owncloud1
    docker network connect owncloud_network owncloud2

Lo compruebo y vemos que ahora sí estan todos en la misma red

    docker network inspect owncloud_network

## Referencias
- Docker: https://www.ionos.es/digitalguide/servidores/configuracion/tutorial-docker-instalacion-y-primeros-pasos/ 
- Docker Compose: https://imaginaformacion.com/tutoriales/que-es-docker-compose
- OwnCloud: https://owncloud.com
- HAProxy: https://www.haproxy.org
- LDAP: https://www.openldap.org





