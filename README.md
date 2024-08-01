# Taller Linux 2024
Estos son playbooks de ansible que hacemos para el taller de Servidores Linux

## Instalacion
Instalamos Ansible usando pipx
```
# dnf install python3-pip
$ pip install pipx
$ pipx ensurepath
```
## Modulos instalados para poder ejecutar ansible
```
collections:
  - name: ansible.posix
  - name: community.general
  - name: community.mysql
```
## 


## Ejecución
```
$ ansible-playbook -i inventario/servidores site.yml
```

## Organización de las carpetas de trabajo

```
Obligatorio-TL-2024
├── Collections/
│   └── requiremets.yml
├── Documentos/
├── Files/
│   ├── todo.war
│   ├── todo.sql
│   └── tomcat.conf
├── inventory/
│   ├── group_vars/
│   │   ├── centos.yml
│   │   └── ubuntu.yml
│   ├── host_vars/
│   │   ├── Webserver.yml
│   │   └── DBServer.yml
│   └── servidores.toml
├── webserver.yml
├── database.yml
└── README.md

```
## Referencias

### Referencias utilizadas para el despliegue del webserver

#### Primero instalo JAVA
    - name: Instalo el JAVA SDK

#### Primero bajo y extraigo el Tomcat
    - name: Bajo el Tomcat y extrae en el directorio /opt/.

#### abro puertos necesarios
    - name: abrir los puertos 8080 y 8443 en el firewall.

#### Despliego app todo.war
    - name: Despliegue de la aplicación ToDo.war en Tomcat.

#### cargo la configuración de todo.war
    - name: Se configura la aplicación ToDo.war mediante un archivo de configuración.