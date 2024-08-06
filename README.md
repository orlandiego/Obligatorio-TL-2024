# Taller Linux 2024
Estos son playbooks de ansible que hacemos para el taller de Servidores Linux y una especie de guia para poder ejecutar todo desde el servidor creado para el despliegue.

## Instalacion
Comentamos como armamos nuestro controller para el depliegue de ansible en los otros equipos y las herramientas utilizadas en el mismo.

 - Ejecutamos con sudo lo siguiente

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

## Instalo Git y bajo mi repositorio para desplegar mis playbooks

Subimos nuestros repositorio a Github para poder desplegarlos
- Debemos instalar git en nuestro controller para hacerlo:
`sudo dnf install git`
- Luego bajamos el repo:
```
git clone https://github.com/orlandiego/Obligatorio-TL-2024.git
```
---
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
## Averiguo las IP de los servidores donde vamos a desplegar

Para poder hacer el despliegue en los 2 servidores, (Centos para webserver y Ubuntu para database), debo saber las IP de los mismos para configurar correcamente mi archivo `servidores-toml` 

```
[centos]
webserver01   ansible_host=192.168.56.102

[ubuntu]
dbserver    ansible_host=192.168.56.103
```

---
## Ejecución o despliegue de ansible:

- Con esto resuelto, verifico que puedo acceder a los mismos, antes tengo que tener generada mi ssh-keygen y correctamente configurada la carpeta donde lo obtengo en hardening.yml
`/home/sadmin/.ssh/id_rsa.pub`

 - Ejecuto el siguiente comando hacia las direcciones de los servidores:
 
```ssh
ssh-copy-id -f sysadmin@192.168.56.XXX    me pide la contraseña de sysadmin 
```
- Verifico que puedo acceder y ejecutar un modulo en el servidor remoto

```
`ansible -i 192.168.56.20, all -m ping`   `--ask-pass`
```

  `--ask-pass` es para que me pida la contraseña de ssh en caso de tener una establecida **"PASSWORD"**

 - Luego despliego el hardenig.yml
```
$ ansible-playbook -i inventario/servidores.toml hardening.yml --ask-become-pass
```

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
│   └── servidores.toml
├── hardening.yml
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
$ ansible-playbook -i inventory/servidores.toml instalar.todo.yml --ask-become-pass
```


## Despliegue del playbook en el dbserver

Para poder desplegar este playbook debemos instalar las herramientas necesarias para poder manejar base de datos

```
sudo dnf install mysql
pipx inject ansible-core pymysql
```

Para deplegar el playbook dy montar el `dbserver` tengo que ejecutar lo siguiente en mi controller

```
$ ansible-playbook -i inventory/servidores.toml install.mariadb.yml --ask-become-pass
```

## CREACION de la base de datos todo dbserver

```
$ ansible-playbook -i inventory/servidores.toml mysql.yml --ask-become-pass
```
## CONCLUSIONES

- Como posibilidad de mejora, podriamos utilizar variables para no tener que modificar en todos lados las direcciones de los servidores al desplegar.
- Modificamos el archivo de creación de base de datos para que cuando ejecutamos elimine la misma si es que hay una ya creada:

`DROP DATABASE IF EXISTS todo;`

### RESULTADO FINAL DEL DESPLIEGUE:

![RESULTADO_FINAL](./Documentos/Imagenes/pruebas/webserver%20funcionando_registro_usuario-logeado.JPG)

## Referencias

### Para acceder por SSH

[Modulo para enviar clae publica SSH](https://docs.ansible.com/ansible/latest/collections/ansible/posix/authorized_key_module.html)

---
### Referencias utilizadas para el despliegue del webserver
---
#### Primero instalo JAVA

[INSTALO-JAVA](https://www.geeksforgeeks.org/how-to-install-java-using-ansible-playbook/)

#### Instalo Tomcat

 [INSTALO-TOMCAT](https://github.com/jmutai/tomcat-ansible/blob/master/tomcat-setup.yml)   

#### abro puertos necesarios

[ABRO PUERTOS](https://docs.ansible.com/ansible/latest/collections/ansible/posix/firewalld_module.html)

#### Despliego app todo.war

[COPIO APP](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html#ansible-collections-ansible-builtin-copy-module)

#### cargo la configuración de todo.war

[COPIO CONFIG](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html#ansible-collections-ansible-builtin-template-module)

---
### Referencias utilizadas para el despliegue de la base de datos
---

- [GITHUB-EMVERDES](https://github.com/emverdes/TallerJulio2024.git)

- [REPO DEL OBLIGATORIO EN GIT](<https://github.com/orlandiego/Obligatorio-TL-2024>)