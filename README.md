![LOGO][def2]

# Taller de Linux

## 🔢 Tabla de contenidos:
- [Taller de Linux](#taller-de-linux)
  - [🔢 Tabla de contenidos:](#-tabla-de-contenidos)
  - [Presentación del problema *`Letra`*](#presentación-del-problema-letra)
    - [PARTE A](#parte-a)
    - [PARTE B](#parte-b)
    - [PARTE C](#parte-c)
    - [PARTE D](#parte-d)
  - [Contenido de este Repositorio](#contenido-de-este-repositorio)
- [Propuesta](#propuesta)
- [Implementación](#implementación)
  - [INVENTARIO](#inventario)
    - [Controller](#controller)
      - [Generamos el usuario app-admin para gestionar todos los servidores con permiso de administrador.](#generamos-el-usuario-app-admin-para-gestionar-todos-los-servidores-con-permiso-de-administrador)
    - [Instancias EC2: Servidores Web 1a y 1b](#instancias-ec2-servidores-web-1a-y-1b)
        - [Bloque de configuración user\_data:](#bloque-de-configuración-user_data)
        - [Archivo Dockerfile utilizado para la construcción de la imagen:](#archivo-dockerfile-utilizado-para-la-construcción-de-la-imagen)
      - [Autoscaling-Group en AWS 🚀](#autoscaling-group-en-aws-)
    - [Base de Datos RDS *(Relational Database Service)*](#base-de-datos-rds-relational-database-service)
      - [Backup de la base de datos](#backup-de-la-base-de-datos)
    - [EFS *(Amazon Elastic File System)*](#efs-amazon-elastic-file-system)
    - [Servidor de backup con persistencia](#servidor-de-backup-con-persistencia)
        - [Configuración de AWS backup](#configuración-de-aws-backup)
    - [Grupos de Seguridad en AWS](#grupos-de-seguridad-en-aws)
    - [Seguridad](#seguridad)
    - [Conexion de las instancias EC2](#conexion-de-las-instancias-ec2)
    - [Networking en AWS](#networking-en-aws)
    - [Zonas de Disponibilidad](#zonas-de-disponibilidad)
    - [Ejemplo de Configuración Terraform](#ejemplo-de-configuración-terraform)
- [Pruebas de funcionamiento](#pruebas-de-funcionamiento)
      - [Despliegue sin errores de terraform apply](#despliegue-sin-errores-de-terraform-apply)
      - [Autoscling - Detalles](#autoscling---detalles)
      - [Launch Configuration](#launch-configuration)
      - [Load Balance - Detalles](#load-balance---detalles)
      - [Verificación de tablas en base de datos](#verificación-de-tablas-en-base-de-datos)
      - [Verificando conexión ssh a servidores mediante clave keypair](#verificando-conexión-ssh-a-servidores-mediante-clave-keypair)
      - [Verificamos el Fileserver Montado](#verificamos-el-fileserver-montado)
      - [Verificamos la creación del BackUp-plan](#verificamos-la-creación-del-backup-plan)
      - [Probamos el Load Balancer](#probamos-el-load-balancer)
      - [Pruebas de Creación de Usuario](#pruebas-de-creación-de-usuario)
- [Conclusiones](#conclusiones)
- [Anexo](#anexo)
  - [Listado de la infraestructura creada *(extraído de terraform-docs)*](#listado-de-la-infraestructura-creada-extraído-de-terraform-docs)
  - [Referencias externas](#referencias-externas)
---



## Presentación del problema *` Letra `*
### PARTE A

- Debe tener listo un servidor controlador para poder utilizar Ansible. Este servidor se preparará
  durante el taller. Debe tener los paquetes y librerías necesarias para utilizar Ansible y Git. El
  usuario que ejecute los automatismos debe contar con sus claves pública/privada SSH. Debe
  contar con un repositorio de código en Github o en Gitlab para trabajar con su equipo.

### PARTE B

- Instalar 2 servidores, ambos con un disco de 13GB el siguiente diseño de particiones:
  
       ● Partición de 1GB para /boot
       ● LVM de 7GB para /
       ● LVM de 3GB para /var
       ● LVM de 2GB para SWAP

- Cada servidor tendrá 1CPU y 2 GB RAM.
- Un servidor debe ser instalado con una distribución de la familia Red Hat (se sugiere CentOS Stream 8 o 9) y el otro debe tener Ubuntu 24.04
- Cada servidor debe tener 2 interfaces de red, 1 conectada a NAT y la otra a una red Interna o Host-Only que le permita conectarse al servidor controlador con Ansible.
- Cada equipo debe contar con un usuario NO root, con permisos para ejecutar comandos como administrador (ansible o sysadmin) y debe copiarse la clave pública del Servidor Controlador, para poder ejecutar los automatismos.

### PARTE C

Tareas a realizar mediante Ansible:

1) En el servidor Red Hat debe instalar la aplicación ToDo usada para el obligatorio del curso
Administración de Servidores Linux
Instalar el JDK de Java, Tomcat y la aplicación con su configuración a la Base de Datos
Tomcat debe iniciarse como servicio mediante SystemD. Los puertos usados por Tomcat deben estar
habilitados en el Firewall

2) En el servidor Ubuntu debe instalar la Base de Datos, y configurarla para la aplicación.
Instalar el servidor MariaDB y asegurarlo con los procedimientos de mysql-secure-installation. Crear
el usuario para la aplicación, y asegurar que el servidor esté levantado. En el Firewall debe estar
permitido el acceso a la Base de Datos.

### PARTE D

Todo el contenido del obligatorio debe estar un un repositorio Git. El repositorio debe tener un
README que describa cómo se usan los playbooks desarrollados.
Debe incluir un directorio Documentación que contenga un documento con toda la descripción de las
tareas realizadas y prueba de ejecución de los playbook y funcionamiento de las aplicaciones.
El repositorio se puede descargar como Zip, y usarse como entrega.

---
## Contenido de este Repositorio
  - ```Docs/```
    - Letra el obligatorio y documentos extras
  - ```imagenes/```
    - Diagramas de arquitectura, screenshots de pruebas y logo del README.md.
  - ```./```
    - Archivos .gitignore y README.md.

---
# Propuesta
  Ante los requerimientos planteados en la letra, proponemos una migración hacia la nube de AWS mediante un despliegue completamente automatizado, tanto de la infraestructura como de la aplicación, utilizando Terraform y Docker.
  
  - Para el despliegue de la aplicación se crean como base dos instancias de EC2 utilizando un launch configuration de autoscaling group, las cuales al momento de crearse, lanzan un contenedor de Docker haciendo uso de una imagen creada por nosotros, que contiene la aplicación, publicada en el puerto 80.
    >***Nota:** Para que la aplicación se conecte a la base de datos, editamos desde la instancia recién creada el archivo config.php cambiando el valor del db_host al endpoint nuevo creado por terraform, y posteriormente se copia hacia adentro del ya lanzado contenedor.*

  - Para la base de datos relacional, utilizamos el servicio el servicio RDS de AWS.

  - En el caso del servidor de documentos, lo reemplazamos por el servicio EFS de AWS.

  - Para los resplaods, con AWS Backup se automatizarán los respaldos de EFS, y se usarán snapshots para respaldar la base de datos.


# Implementación
  La arquitectura diseñada para la migración del frontend a AWS se compone de lo siguiente:
  - Instancias EC2 para el servidor web lanzadas mediante AutoscalingGroup.
  - Un Load Balancer de aplicación, que balanceará la carga de ambas instancias a la vez que sirve de presentador para el acceso desde internet.
  - RDS para la base de datos.
  - Un fileserver implementado en EFS para el almacenamiento de archivos fijos.
  - AWS-Backup para el plan de almacenamiento del EFS, y snapshots para el RDS.
  


  Todos estos recursos de AWS se desplegarán utilizando la herramienta Terraform, cuyos archivos están disponibles en la carpeta [```despliegue```](https://github.com/roxdzp/ObligatorioCloud2024/tree/main/despliegue)

  ![Despliegue][def]

## INVENTARIO

### Controller

- **Descripción**: El controller despliega ansible en los servidores.


| Servidor                  | Linux              | Descripción                                    | Subred          |                   |
|--------------------------------|-------------------|------------------------------------------------|-----------------|--------------------|
| Controller                     | Centos 9 Stream          | Despliega Ansible                              | 192.168.56.10         | -                  |
| WServidor01           | Centos 9 Stream              |  Aplicación ToDo                           | 192.168.56.20        | -                  |
| DBServidor01                | Ubuntu      | Base de datos                              | 192.168.56.30        | -                  |



#### Generamos el usuario app-admin para gestionar todos los servidores con permiso de administrador.

 ```markdown
  stickiness {
    type            = "lb_cookie"
    cookie_duration = 3600 # Duración de la cookie en segundos (opcional, por defecto es 1 día)
  }
 ```

### Instancias EC2: Servidores Web 1a y 1b
Al crear las instancias mediante terraform un bloque user-data que nos permite ejecutar commandos al momento de la creación como si fuera un script. Esto nos permite preparar nuestra instancia EC2 para que se instalen con el pre-requisitos, se modificación el archivo config.php de la app para la conexión a la base de datos, así como el despliegue del contenedor con la aplicación en Docker, y la creación de la base de datos en el RDS.

##### Bloque de configuración user_data:
  ```
  (...)
    user_data = <<-EOF
                #!/bin/bash
                sudo yum update -y
                sudo yum install -y docker git mysql amazon-efs-utils
                sudo mkdir /mnt/efs
                sudo mount -t efs -o tls ${aws_efs_file_system.file-srv-docs.id}:/ /mnt/efs
                sudo systemctl start docker
                sudo systemctl enable docker
                curl https://raw.githubusercontent.com/roxdzp/ecommerce-sql/main/config.php > config.php
                curl https://raw.githubusercontent.com/roxdzp/ecommerce-sql/main/dump.sql > dump.sql
                sudo sed -i 's/db-endpoint/${aws_db_instance.db_web.address}/g' config.php
                sudo sed -i 's/db-password/${var.db_password}/g' config.php
                sudo mysql -h ${aws_db_instance.db_web.address} -u admin -ppasswordort -e 'CREATE DATABASE IF NOT EXISTS db_web; USE db_web; SOURCE dump.sql;'
                sudo docker run -d -p 80:80 --name ecommercepod rdiazpe/obligatorio-ecommerce
                sudo docker cp config.php ecommercepod:/var/www/html
                EOF
  (...)
  ```
  La imagen de Docker utilizada para el lanzamiento del contenedor fue creada por nosotros para éste fin.
  A partir de ella se lanza un centos 7 con los servicios apache,  php, php-mysql y git ya instalados, y con un clon del repositorio git [app-simple-ecommer](https://github.com/roxdzp/app-simple-ecomme) en el directorio `/var/www/html`. También se realiza la publicación en el puerto 80.

  > *Imagen disponible en [Docker hub: *obligatorio-ecommerce*](https://hub.docker.com/r/rdiazpe/obligatorio-ecommerce/tags)*
  ##### Archivo Dockerfile utilizado para la construcción de la imagen:

  ```
  FROM centos:centos7
  RUN yum install -y httpd php php-mysql git
  RUN git clone https://github.com/roxdzp/app-simple-ecomme.git
  RUN mv /app-simple-ecomme/* /var/www/html
  EXPOSE 80
  ENTRYPOINT ["/usr/sbin/httpd", "-D", "FOREGROUND"]
  ```

#### Autoscaling-Group en AWS 🚀
Para el lanzamiento de estas instancias EC2, usamos el servicio de Autoscaling-group de AWS.
El **autoscaling** en AWS es una funcionalidad que permite automáticamente ajustar el número de instancias EC2 basado en la demanda de la aplicación. Esto se logra mediante la configuración de un **Autoscaling Group** que incluye:
| Nombre | Tipo |
|------|------|
| [aws_launch_configuration.autoscaling_imagen](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/launch_configuration) | resource |
| [aws_autoscaling_group.autoscaling](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/autoscaling_group) | resource |


  - **Launch Configuration**: Define las especificaciones de cada instancia EC2, como la imagen AMI, tipo de instancia, y configuración inicial.
  ```
  resource "aws_launch_configuration" "autoscaling_imagen" {
    depends_on      = [aws_db_instance.db_web]
    name_prefix     = "EcommerceService_imagen"
    image_id        = "ami-03ededff12e34e59e"
    instance_type   = "t2.micro"
    security_groups = [aws_security_group.acceso_web.id]
    key_name = "key_srv"
  }
  ```
  >***Nota:** elegimos instancias del tipo t2.micro, para segurarnos que funcione todo lo que configuramos.*

  - **Autoscaling Group**: Establece los parámetros de escalado, incluyendo el número deseado (`desired_capacity`), mínimo (`min_size`) y máximo (`max_size`) de instancias.
  ```
  resource "aws_autoscaling_group" "autoscaling" {
    depends_on  = [aws_db_instance.db_web]
    name_prefix = "ASG_GROUP"

    desired_capacity = 2 // Ponemos 2 para lanzar dos instancias!
    min_size         = 2
    max_size         = 4 // Número máximo de instancias activas

    launch_configuration = aws_launch_configuration.autoscaling_imagen.name
    vpc_zone_identifier = [aws_subnet.subnet_privada1a.id, aws_subnet.subnet_privada1b.id]

    health_check_type         = "ELB"
    health_check_grace_period = 300

    target_group_arns = [aws_lb_target_group.lb-tg.arn]  // agregamos las instancias que se crean al aws_lb_target_group

    lifecycle {
      create_before_destroy = true
    }
  }
  ```

Al monitorear métricas, como la utilización de la CPU o el tráfico de red, AWS puede agregar o eliminar automáticamente instancias para mantener el rendimiento y la disponibilidad de la aplicación sin intervención manual.

Este proceso mejora la escalabilidad y la resiliencia de las aplicaciones en la nube al permitir respuestas dinámicas a cambios en la carga de trabajo.

El autoscaling es fundamental para optimizar costos y mejorar la experiencia del usuario al asegurar un rendimiento constante de la aplicación en AWS.

### Base de Datos RDS *(Relational Database Service)*

- **Descripción**: Gestiona los datos dinámicos de la aplicación.
- **Recursos AWS**: `aws_db_instance`, `aws_db_subnet_group`.
- **Nombre**: `bdb-web`.


| Recurso AWS        | Name            | Descripción                                    | Subred          | SecutityGroup      |
|--------------------|-----------------|------------------------------------------------|-----------------|--------------------|
| aws_db_instance    | db_web          | Instancia AWS RDS                              |                 | acceso_3306_sg     |
| aws_db_subnet_group| subnet_db       | Instancia AWS RDS                              |                 | acceso_3306_sg     |

#### Backup de la base de datos

Utilizamos la configuración de snapshots propia de AWS RDS

```markdown
  maintenance_window      = "Sat:01:00-Sat:04:00"
  backup_window           = "00:00-00:50"
  backup_retention_period = 7
  apply_immediately       = "true"
```
Los snapshots de RDS se almacenan en Amazon S3 y son independientes de la instancia de RDS. Esto significa que si la instancia de RDS se pierde o se elimina, los snapshots permanecen disponibles y se pueden utilizar para restaurar la base de datos a una nueva instancia de RDS.

### EFS *(Amazon Elastic File System)*
Es un servicio de almacenamiento de archivos que usamos para los documentos que manejan los servidores.

EFS permite escalabilidad automática, sin necesidad de aprovisionar ni administrar la infraestructura subyacente, es ideal para compartir datos de archivos en entornos de nube y locales y se puede montar tanto en instancias de Amazon EC2 como en servidores locales.
| Nombre | Tipo |
|------|------|
| [aws_efs_file_system.file-srv-docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/efs_file_system) | resource |
| [aws_efs_mount_target.file-srv-1a](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/efs_mount_target) | resource |
| [aws_efs_mount_target.file-srv-1b](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/efs_mount_target) | resource |


### Servidor de backup con persistencia
AWS Backup está diseñado para ser un servicio resistente y confiable. Los archivos en el vault de backup están protegidos y se replican en múltiples zonas de disponibilidad para asegurar su durabilidad y disponibilidad. De esta manera, incluso si el servicio de AWS Backup enfrenta problemas, los datos en los vaults de backup permanecen seguros y accesibles.

| Nombre | Tipo |
|------|------|
| [aws_autoscaling_group.autoscaling](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/autoscaling_group) | resource |
| [aws_backup_plan.backup_plan_efs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/backup_plan) | resource |
| [aws_backup_selection.efs_backup](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/backup_selection) | resource |
| [aws_backup_vault.example](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/backup_vault) | resource |

##### Configuración de AWS backup
```markdown
resource "aws_backup_plan" "backup_plan_efs" {
  name = "backup_plan_efs"

  rule {
    rule_name         = "efs-rule"
    target_vault_name = aws_backup_vault.example.name
    schedule          = "cron(0 12 * * ? *)" // Realiza el backup diariamente a las 12 PM UTC
    lifecycle {
      delete_after = 30 // Elimina los backups después de 30 días
    }
  }
}
```

### Grupos de Seguridad en AWS

Los Grupos de seguridad, actúan como un firewall virtual para las instancias EC2, controlando el tráfico entrante y saliente.
Creamos nuevos grupos de seguridad con reglas que nos permitan acceder por http al load balancer y a los servidores por SSH y HTTP. También creamos un grupo de seguridad para poder acceder a la base de datos.

- **Función**: Actúan como un firewall virtual para las instancias EC2, controlando el tráfico entrante y saliente.
- **Reglas**: Posibilidad de `asignar` múltiples instancias a un mismo grupo y establecer reglas aplicables a todas.
- **Flexibilidad**: Varias instancias pueden apuntar al mismo grupo para mantener una política de seguridad coherente.
- **Configuración**: Se especifican rangos de IP permitidos y puertos de destino.
- **Recursos AWS**: `aws_security_group`.
### Seguridad


| Recurso AWS        | Name            | Descripción                                    | input (port)    | Output (block-Cdir)|
|--------------------|-----------------|------------------------------------------------|-----------------|--------------------|
| [aws_security_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group) | acceso_web      | Permitir el acceso al Servidor Web y al SSH    | 80 y 22         | 0.0.0.0/0  - all   |
| [aws_security_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group) | acceso_3306_sg  | Permitir el acceso al Servidor de Base de Datos| 3306            | 0.0.0.0/0  - all   |
| [aws_security_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group) | lb_sg           | Permite acceso desde internet al Load Balancer | 80 y 22         | 0.0.0.0/0  - all   |


### Conexion de las instancias EC2

![Keypair](./imagenes/AuthenticationSSH.png)

En este paso tenemos que seleccionar la clave pública SSH que le vamos a inyectar a la instancia EC2.
Aquí podemos generar un par de claves SSH o podemos utilizar las claves SSH que ya están creadas y asociadas a nuestra cuenta de usuario en la plataforma AWS Learner Lab.
En este ejemplo vamos a generar la clave utilizando el recurso mencionado a continuación para obtener la clave publica.  para poder conectarnos por SSH con la instancia EC2.

- **Keypair para conexiones a servidores mediante SSH**: Generamos keypair en el despliegue de terraform para conectarnos a los servidores
- **Recursos AWS**: `aws_key_pair Terraform`.

```markdown
resource "aws_key_pair" "key_srv" {
  key_name   = "key_srv"
  public_key = tls_private_key.rsa.public_key_openssh
}
```
```markdown
resource "tls_private_key" "rsa" {
  algorithm = "RSA"
  rsa_bits  = 4096
}
```
```markdown
resource "local_file" "key_srv" {
  content  = tls_private_key.rsa.private_key_pem
  filename = "keysrv"
}
```


### Networking en AWS
Para nuestra red privada, hemos elegido el rango de direcciones IP o **CIDR** (Classless Inter-Domain Routing) 10.0.0.0/16. Este rango nos proporciona suficiente espacio para el crecimiento futuro sin limitaciones iniciales, lo cual es ideal para la escalabilidad de nuestra infraestructura.

Las subredes dividen una red mayor en segmentos más pequeños, mejorando la organización y seguridad de la infraestructura.

En nuestro diseño:

  - **aws_vpc `vpc_web`**: Define una VPC (Virtual Private Cloud) con un rango de direcciones 10.0.0.0/16.
  - **aws_subnet `privada1a` y `privada1b`**: Subredes dentro de la VPC que permiten una mejor segmentación y seguridad. Cada subred tiene un rango de /24, lo que permite 256 direcciones IP por subred. Cada una ubicada en distinta zonas de disponibilidad.


| Recurso AWS        | Name            | Descripción                                    | Subred          | SecurityGroup      |
|--------------------|-----------------|------------------------------------------------|-----------------|--------------------|
| `aws_vpc`          | `vpc_web`       | VPC para la infraestructura web                | 10.0.0.0/16     | -                  |
| `aws_subnet`       | `privada1a`     | Subred privada para el servidor web            | 10.0.1.0/24     | `acceso_web`       |
| `aws_subnet`       | `privada1b`     | Subred privada para el servidor web            | 10.0.2.0/24     | `acceso_web`       |



  -  **Variables Definidas**: Para la región, bloque CIDR de la VPC y bloques CIDR de las subredes.
  - **Asignación de IP Pública**: `map_public_ip_on_launch` como `true` para asignación automática de IP pública.


  - **Route Table**: Ruta predeterminada (para acceso a Internet `0.0.0.0/0` a través de `aws_internet_gateway.internet_gateway.id`
  - **Internet Gateway**: "web-gateway"

  ```markdown
  resource "aws_internet_gateway" "internet_gateway" {
    vpc_id = aws_vpc.vpc_web.id
    tags = {
      Name = "web-gateway"
    }
  }

  resource "aws_default_route_table" "route_table" {
    default_route_table_id = aws_vpc.vpc_web.default_route_table_id

    route {
      cidr_block = "0.0.0.0/0"
      gateway_id = aws_internet_gateway.internet_gateway.id
    }

    tags = {
      Name = "Salida a internet"
    }
  }
  ```


  - **Recursos AWS**: `aws_vpc`, `aws_subnet`, `aws_internet_gateway`, `aws_default_route_table`

| Nombre | Tipo |
|------|------|
| [aws_db_subnet_group.subnet_db](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_subnet_group) | resource |
| [aws_default_route_table.route_table](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/default_route_table) | resource |
| [aws_internet_gateway.internet_gateway](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/internet_gateway) | resource |
| [aws_subnet.subnet_privada1a](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet) | resource |
| [aws_subnet.subnet_privada1b](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet) | resource |
| [aws_vpc.vpc_web](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc) | resource |




### Zonas de Disponibilidad
Para aprovechar la alta disponibilidad y redundancia que ofrece AWS, hemos decidido operar en dos zonas de disponibilidad distintas.

- **Zonas de Disponibilidad (AZs)** son ubicaciones discretas dentro de las regiones de AWS que están diseñadas para aislar fallos y proporcionar redundancia. Operar en múltiples AZs permite alta disponibilidad y tolerancia a fallos.
- **Limitaciones en AWS Academy**: En la cuenta de AWS academy no es posible utilizar configuraciones multi-AZ para ciertos servicios como RDS, lo cual limita nuestra capacidad de desplegar algunas configuraciones de alta disponibilidad que normalmente usaríamos en un entorno de producción.

### Ejemplo de Configuración Terraform
  ```hcl
  resource "aws_vpc" "vpc_web" {
    cidr_block = var.vpc_cidr // variable.tf
    tags = {
      Name = "Red privada virtual"
    }
  }

  resource "aws_subnet" "subnet_privada1a" {
    vpc_id                  = aws_vpc.vpc_web.id
    cidr_block              = var.subnet_privada1a_cidr // variable.tf
    availability_zone       = var.az-1a
    map_public_ip_on_launch = true
    tags = {
      Name = "VPC US-East-1a"
    }
  }

  resource "aws_subnet" "subnet_privada1b" {
    vpc_id                  = aws_vpc.vpc_web.id
    cidr_block              = var.subnet_privada1b_cidr // variable.tf
    availability_zone       = var.az-1b
    map_public_ip_on_launch = true
    tags = {
      Name = "VPC US-east-1b"
    }
  }

  ```
# Pruebas de funcionamiento

 -  **VIDEO DE LA DEMO CON EL DESPLIEGUE DE LA SOLUCIÓN EN AWS ACADEMY**:

https://youtu.be/6VQnj_c17d4

#### Despliegue sin errores de terraform apply

  ![Arquitectura](./imagenes/pruebas/Creacion_sin_errores.png)

#### Autoscling - Detalles

  ![Autoscaling](./imagenes/pruebas/Autoscaling-group.jpg)

#### Launch Configuration

  ![Autoscaling](./imagenes/pruebas/Autoscaling-launch-configuration%20y%20user-data.jpg)

#### Load Balance - Detalles

  ![LoadBalancer](./imagenes/pruebas/LoadBalancer_detalles.jpg)

  ![LoadBalancer](./imagenes/pruebas/LoadBalancer_detalles2.jpg)

#### Verificación de tablas en base de datos

  ![BasedeDatos](./imagenes/pruebas/Verificación%20de%20base%20de%20datos.jpg)


#### Verificando conexión ssh a servidores mediante clave keypair
  
  ![SSH](./imagenes/pruebas/Prueba%20con%20Autoscaling_25_resources-added.jpg)

  ![SSH](./imagenes/pruebas/Prueba%20conexión%20a%20servidor%20mediante%20SSH%20a%20ambos%20servidores.jpg)

#### Verificamos el Fileserver Montado

  ![Fileserver](./imagenes/pruebas/Creamos%20Archivos%20en%20el%20Fileserver.jpg)

#### Verificamos la creación del BackUp-plan

  ![backup](./imagenes/pruebas/Verifico%20la%20creación%20de%20Backup-Plan.jpg)

#### Probamos el Load Balancer

  ![Pagina](./imagenes/pruebas/LoadBalancer_pagina.jpg)

#### Pruebas de Creación de Usuario

  ![Pagina](./imagenes/pruebas/App_Registro%20exitoso.jpg)

  ![Pagina](./imagenes/pruebas/App_logon%20exitoso.jpg)


# Conclusiones

hiiugiuh


---
---
# Anexo

  ## Listado de la infraestructura creada *(extraído de terraform-docs)*

hihiuhuh

## Referencias externas

- [Repo en GIT](<https://github.com/orlandiego/Obligatorio-TL-2024>)

- ***Material de clase y ejercicios realizados**: Material del curso, reutilización de codigo de los trabajos realizados*

---


[def]: ./imagenes/despliegue.jpg
[def2]: ./imagenes/logo.png