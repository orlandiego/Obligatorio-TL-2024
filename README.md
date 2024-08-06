# Taller Linux 2024
Estos son playbooks de ansible que hacemos para el taller de Servidores Linux

## Instalacion
Comentamos como armamos nuestro controller para el depliegue de ansible en los otros equipos y las herramientas utilizadas en el mismo.

---

Ejecutamos con sudo lo siguiente

```
dnf install python3-pip
pip install pipx
pipx ensurepath
pipx install ansible-core
pipx inject ansible-core argcomplete
pipx inject ansible-core ansible-lint   
pipx install ansible-lint               // no quedo instalado antes
activate-global-python-argcomplete --user
```

## Modulos para poder ejecutar ansible

- Estos modulos debemos instalar para poder ejecutar ansible, los mismos los cargamos en la carpeta Collections con el nombre de `requirements.yml`

```
collections:
  - name: ansible.posix
  - name: community.general
  - name: community.mysql
```
 - Instalamos estas dependencias ejecutando:

```
 ansible-galaxy collection install -r collections/requirements.yml 
```
### Con esto instalo ansible en Cent0s 9
---
## Instalo Git y bajo mi repositorio para desplegar mis playbooks

Subimos nuestros repositorio a Github para poder desplegarlos
- Debemos instalar git en nuestro controller para hacerlo:
`sudo dnf install git`
- Luego bajamos el repo:
```
git clone https://github.com/orlandiego/Obligatorio-TL-2024.git
```
---
## Averigu las Ip de los servidores donde vamos a desplegar

Para poder hacer el despliegue en los 2 servidores, (Centos para webserver y Ubuntu para database), debo saber las IP de los mismos para configurar correcamente mi archivo `servidores-toml` 

```
[centos]
webserver01   ansible_host=192.168.56.102

[ubuntu]
dbserver    ansible_host=192.168.56.103
```

---
## Ejecución o despliegue de ansible:

- Con esto resuelto, verifico que puedo acceder a los mismos, antes tengo que tener generada mi ssh-keygen y correctamente configurada la carpeta donde lo obtengo
`/home/sadmin/.ssh/id_rsa.pub`

```
$ ansible-playbook -i inventario/servidores.toml hardening.yml --ask-become-pass
```
- Copio la clave pública a los servidores remotos para poder ingresar
- Tambien podemos copiarla ejecutando el siguiente comando ssh hacia cada servidor, indico el usuario donde copiarla `sysadmin`

```ssh
ssh-copy-id -f sysadmin@192.168.56.XXX    me pide la contraseña de sysadmin 
```

- Verifico que puedo acceder y ejecutar un modulo en el servidor remoto

```
`ansible -i 192.168.56.20, all -m ping`   `--ask-pass`
```

  `--ask-pass` es para que me pida la contraseña de ssh en caso de tener una establecida **"PASSWORD"**

## Organización de las carpetas de trabajo


```
Obligatorio-TL-2024
├── Collections/
│   └── requiremets.yml
├── Documentos/
├── Files/
│   ├── app.properties
│   ├── todo.sql
│   └── todo.war
├── inventory/
│   ├── group_vars/
│   │   ├── centos.yml
│   │   └── ubuntu.yml
│   ├── host_vars/
│   │   ├── database.yml
│   │   └── webserver.yml
│   └── servidores.toml
├── instalar_todo.yml
├── install_mariaDB.yml
|   mysql.yml
└── README.md

```

## Despliegue de playbook en el webserver

Para deplegar el playbook dy montar el `webserver` tengo que ejecutar lo siguiente en mi controller

 - Verificar que el archivo app.properties apunte a la IP de la base de datos antes de desplegar!

```
tipoDB=mysql
jdbcURL=jdbc:mysql://192.168.56.103:3306/todo
jdbcUsername=sysadmin
jdbcPassword=tlxadmin
```

```
$ ansible-playbook -i inventario/servidores.toml instalar.todo.yml --ask-become-pass
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

### Para acceder por SSH

[Modulo para enviar clae publica SSH](https://docs.ansible.com/ansible/latest/collections/ansible/posix/authorized_key_module.html)

---
### Referencias utilizadas para el despliegue del webserver
---
#### Primero instalo JAVA

1. Instalo el JAVA SDK

[INSTALO-JAVA](https://www.geeksforgeeks.org/how-to-install-java-using-ansible-playbook/)

#### Primero bajo y extraigo el Tomcat

2. Instalo Tomcat

 [INSTALO-TOMCAT](https://github.com/jmutai/tomcat-ansible/blob/master/tomcat-setup.yml)   

#### abro puertos necesarios

3. Abro los puertos 8080 y 8443 en el firewall.

[ABRO PUERTOS](https://docs.ansible.com/ansible/latest/collections/ansible/posix/firewalld_module.html)

#### Despliego app todo.war

4. Despliegue de la aplicación ToDo.war en Tomcat.

[COPIO APP](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html#ansible-collections-ansible-builtin-copy-module)

#### cargo la configuración de todo.war

5. Se configura la aplicación ToDo.war mediante un archivo de configuración.

[CCOPIO CONFIG](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html#ansible-collections-ansible-builtin-template-module)

---
### Referencias utilizadas para el despliegue de la base de datos
---