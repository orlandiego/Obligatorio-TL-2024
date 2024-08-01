# Taller Linux 2024
Estos son playbooks de ansible que hacemos para el taller de Servidores Linux

## Instalacion
Instalamos Ansible usando pipx
```
# dnf install python3-pip
$ pip install pipx
$ pipx ensurepath
$ pipx install ansible-lint
```
## Modulos para poder ejecutar ansible

- Estos modulos debemos instalar para poder ejecutar ansible, los mismos los cargamos en la carpeta Collections con el nombre de `requirements.yml`

```
collections:
  - name: ansible.posix
  - name: community.general
  - name: community.mysql
```
### Instalamos estas dependencias ejecutando:
```
 ansible-galaxy collection install -r collections/requirements.yml 
```


## 


## Ejecución
```
$ ansible-playbook -i inventario/servidores.toml hardening.yml
```
En este primer despliegue preparo los servidores para recibir conexiones ssh y establecer su IP como fija.


## Organización de las carpetas de trabajo

```
Obligatorio-TL-2024
├── Collections/
│   └── requiremets.yml
├── Documentos/
├── Files/
│   ├── todo.dql
│   ├── tomcat.conf
│   └── virtualhost.conf
├── inventory/
│   ├── group_vars/
│   │   ├── centos.yml
│   │   └── ubuntu.yml
│   ├── host_vars/
│   │   ├── Webserver.yml
│   │   └── DBServer.yml
│   └── servidores.toml
├── Template/
│   └── todo.war
├── webserver.yml
├── database.yml
|   hardening.yml
└── README.md

```

## Despliegue de playbook en el webserver

Para deplegar el playbook dy montar el `webserver` tengo que ejecutar lo siguiente en mi controller
```
$ ansible-playbook -i inventario/servidores.toml webserver.yml
```


## Despliegue del playbook en el dbserver

Para poder desplegar este playbook debemos instalar las herramientas necesarias para poder manejar base de datos

```
sudo dnf install mysql
pipx inject ansible-core pymysql
```

Para deplegar el playbook dy montar el `dbserver` tengo que ejecutar lo siguiente en mi controller

```
$ ansible-playbook -i inventario/servidores.toml database.yml
```

## Referencias

### Para acceder por SSH y establecer IP Fija

[Modulo para enviar clae publica SSH](https://docs.ansible.com/ansible/latest/collections/ansible/posix/authorized_key_module.html)

---
### Referencias utilizadas para el despliegue del webserver
---
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
---
### Referencias utilizadas para el despliegue de la base de datos
---