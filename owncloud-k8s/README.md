# Documentación de la práctica - Escenario 2 usando Kubernetes

### Nombre del alumno: Cristina del Águila Martín

## Entorno de desarrollo y de producción utilizado

### Entorno de desarrollo:
- **Sistema Operativo**: macOS Sonoma 14.2
- **Versión de Minikube**: 1.35.0
- **Versión de Docker**: 27.3.1
- **Gestor de contenedores**: Docker
- **Herramienta de línea de comandos Kubernetes**: 
    Client Version: v1.32.3
    Kustomize Version: v5.5.0
    Server Version: v1.32.0

## Descripción de la práctica y problema a resolver

### Descripción:
El objetivo de esta práctica es realizar el despliegue de un sistema propio de **OwnCloud** en un entorno Kubernetes, usando varios servicios que permiten que la aplicación sea escalable, esté distribuida y balanceada entre múltiples instancias. El despliegue usa contenedores Docker para contener los servicios y Kubernetes para la orquestación, gestión y escalabilidad.

He configurado, como se solicitan, los siguientes servicios:

- **OwnCloud**: Aplicación de almacenamiento en la nube
- **MariaDB**: Base de datos para almacenar la información de OwnClou
- **Redis**: Caché para mejorar el rendimiento de la aplicación.
- **OpenLDAP**: Servicio de autenticación para gestionar usuarios

## Servicios desplegados y su configuración

### 1. **ownCloud**
El servicio **ownCloud** lo despliego como un **Deployment** de Kubernetes. En este caso, uso dos réplicas del contenedor, como se solicita.

El código está en el archivo adjunto "owncloud-deployment.yaml".

### 2. **MariaDB**
He configurado una base de datos MariaDB para almacenar los datos de ownCloud. La base de datos está configurada para aceptar conexiones desde ownCloud

El código está en el archivo adjunto "mariadb-deployment.yaml".

### 3. **Redis**
El servicio Redis se utiliza como sistema de caché para mejorar el rendimiento de ownCloud. Este servicio también se configura como un Deployment de Kubernetes.

El código está en el archivo adjunto "redis-deployment.yaml".

### 4. **OpenLDAP**
El servicio OpenLDAP se utiliza para la autenticación de usuarios en ownCloud.
El código está en el archivo adjunto "openldap-deployment.yaml".

### 5. **Ingress Controller**
He instalado el Ingress Controller de NGINX para gestionar el tráfico externo y realizar el balanceo de carga entre las instancias de ownCloud.
Para esto he usado el comando:

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

Para configurarlo he creado el archivo owncloud-ingress.yaml para exponer el servicio ownCloud a través de un nombre de dominio personalizado.
Lo aplico con el comando:

    kubectl apply -f owncloud-ingress.yaml

### 6. **Persistencia de datos**
He usado volúmenes persistentes (PV) en Kubernetes para asegurar que los datos de MariaDB no se pierdan cuando los pods se reinicien. La configuración del volumen persistente se puede consultar en el archivos "mariadb-deployment.yaml"

## Instrucciones para lanzar la provisión de servicios
### Iniciar Minikube

    minikube start

### Desplegar los servicios:
Aplico los archivos YAML que había creado anteriormente en el clúster de Kubernetes

    kubectl apply -f mariadb-deployment.yaml
    kubectl apply -f redis-deployment.yaml
    kubectl apply -f openldap-deployment.yaml
    kubectl apply -f owncloud-deployment.yaml
    kubectl apply -f owncloud-ingress.yaml

### Configurar DNS local:
He añadido una entrada en el archivo /etc/hosts para resolver owncloud.local a la IP de Minikube:

    minikube ip

Añado en la ultima línea del archivo /etc/hosts lo siguiente:

    192.168.49.2    owncloud.local

Queda así:

    sudo nano /etc/hosts 

![cap5](/owncloud-k8s/img/c5.png)

### Acceder a OwnCloud:
Abro un navegador y accedemos a 

    http://owncloud.local

## Problemas encontrados

He configurado un clúster de Kubernetes en Minikube para ejecutar un servicio de owncloud, y he creado un recurso Ingress para exponer el servicio a través de un nombre de dominio local, owncloud.local. Sin embargo, aunque los pods de Kubernetes están corriendo sin errores, no podía acceder al servicio usando la URL configurada.

### Verificación de los pods en ejecución:
He comprobado que todos los pods del clúster están en ejecución utilizando el siguiente comando

    kubectl get pods 

Los resultados muestran que todos los pods están en estado "Running", lo que indica que no hay problemas con el despliegue de los contenedores:

![cap1](/owncloud-k8s/img/c1.png)

### Verificación del servicio Ingress: 
He revisado los servicios del clúster utilizando el comando

    kubectl get svc 

Como podemos observar en la imagen, el servicio owncloud está corriendo correctamente en el puerto 80:30080

![cap2](/owncloud-k8s/img/c2.png)

También, he comprobado que el recurso Ingress está configurado correctamente:

    kubectl get ingress

El recurso Ingress para owncloud.local está configurado con la dirección IP correcta, como se muestra en el siguiente resultado: 

![cap3](/owncloud-k8s/img/c3.png)

A pesar de que el servicio está correctamente expuesto y el Ingress está configurado, no podía acceder al servicio owncloud.local en el navegador. La URL http://owncloud.local no carga la aplicación.

Además, también he verificado si el servicio owncloud es accesible dentro del clúster, para ello hago un port-forward para redirigir el puerto 80 del servicio owncloud a un puerto disponible en mi máquina local. Esto me permite acceder al servicio como si fuera una aplicación local. Al hacer: 

    kubectl port-forward service/owncloud 8080:80

después en mi navegador uso

    http://localhost:8080

y comprobamos que funciona.

![cap4](/owncloud-k8s/img/c4.png)

## Referencias bibliográficas y recursos utilizados
- Kubernetes Documentation: https://kubernetes.io/docs/
- Ejemplo de despliege Kubernetes: https://www.youtube.com/watch?v=o7D8COVgK1o 
- ownCloud Documentation: https://doc.owncloud.com/
- Minikube: https://minikube.sigs.k8s.io/docs/
- NGINX Ingress Controller: https://kubernetes.github.io/ingress-nginx/

