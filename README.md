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
Documentos/
├── Collections/
│   └── requiremets.yml
├── Files/
│   ├── todo.war
│   └── todo.sql
├── inventory/
│   ├── groupvars/
│   │   ├── centos.yml
│   │   └── ubuntu.yml
│   ├── hostvars/
│   │   ├── Webserver.yml
│   │   └── DBServer.yml
│   └── servidores.toml
├── webserver.yml
├── database.yml
└── README.md
```

## License
MIT