# High-Availability ownCloud Infrastructure Deployment

Este repositorio contiene el diseño y despliegue de una arquitectura de almacenamiento en la nube basada en **ownCloud**, utilizando tecnologías de contenerización y orquestación para garantizar escalabilidad, persistencia y alta disponibilidad.

---

## 👤 Información del Proyecto
* **Autor:** [Tu Nombre]
* [cite_start]**Curso:** 2024-2025 [cite: 2]
* **Stack Tecnológico:** Podman/Docker, Podman-Compose, Kubernetes, HAProxy, OpenLDAP.

---

## 🎯 Objetivos Técnicos
* [cite_start]**Interconexión de Microservicios:** Despliegue de servicios federados (Web, DB, Cache, Auth)[cite: 17].
* [cite_start]**Alta Disponibilidad (HA):** Implementación de balanceo de carga y replicación de servicios[cite: 17].
* [cite_start]**Gestión de Identidades:** Integración de autenticación centralizada mediante el protocolo LDAP[cite: 17].
* [cite_start]**Persistencia Avanzada:** Gestión de volúmenes de datos para servicios críticos como MariaDB y OpenLDAP[cite: 17].

---

## 🏗️ Arquitecturas Implementadas

### Escenario 1: Arquitectura de Microservicios Base
[cite_start]Diseñada para entornos de pequeñas dimensiones (hasta 150 usuarios)[cite: 111].
* [cite_start]**Servicio Web:** ownCloud (Frontend)[cite: 111].
* [cite_start]**Base de Datos:** MariaDB con persistencia local[cite: 111].
* [cite_start]**Caché:** Redis para optimización de bloqueos de archivos[cite: 111].
* [cite_start]**Directorio:** OpenLDAP para gestión de usuarios[cite: 111].



### Escenario 2: Alta Disponibilidad y Escalabilidad
[cite_start]Configuración orientada a empresas medianas (150-1,000 usuarios) con redundancia total[cite: 111].
* [cite_start]**Balanceador de Carga:** **HAProxy** configurado como proxy inverso para distribuir tráfico[cite: 111].
* [cite_start]**Replicación:** Estrategia de replicación en los nodos de aplicación y base de datos para evitar puntos únicos de fallo (SPOF)[cite: 111].
* [cite_start]**Health Checks:** Monitoreo activo de la salud de los servicios desde el panel de HAProxy[cite: 111].



---

## 🔧 Configuración Destacada

### Directorio Activo (LDAP)
Se ha configurado un servidor **OpenLDAP** para la gestión jerárquica de usuarios:
* [cite_start]**Estructura DIT:** Basada en `dc=example,dc=org`[cite: 111].
* [cite_start]**Persistencia:** Montaje de volúmenes en `/var/lib/ldap` y `/etc/ldap/slapd.d` para asegurar que los datos trasciendan el ciclo de vida del contenedor[cite: 111].
* [cite_start]**Integración:** Sincronización completa con ownCloud para autenticación transparente[cite: 111].

### Balanceo con HAProxy
Configuración del balanceador para distribuir tráfico entre múltiples réplicas del servidor web:
* [cite_start]**Algoritmo:** Round-robin[cite: 111].
* [cite_start]**Dashboard:** Panel de estadísticas habilitado en el puerto `8404` para supervisión en tiempo real[cite: 111].

---

## 🚀 Despliegue

El proyecto permite un despliegue escalonado según la herramienta de orquestación disponible:

1.  **Docker/Podman Compose:**
    ```bash
    podman-compose up -d
    ```
2.  **Kubernetes:**
    ```bash
    kubectl apply -f ./k8s-manifests/
    ```

---

## 📚 Referencias y Recursos
* [cite_start][Manual de Administración de ownCloud](https://doc.owncloud.com/) [cite: 111]
* [cite_start][Documentación oficial de OpenLDAP](https://www.openldap.org/) [cite: 111]
* [cite_start][Guía de configuración de HAProxy](http://docs.haproxy.org/) [cite: 111]
