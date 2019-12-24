## Laboratorio: Docker - Ansible

Objetivos
=======

- Repaso basico Docker
- Introduccion a Ansible
- Practica - Ejercicios 

Requisitos
=======
- Conocimientos basicos en Docker ( no excluyente ) 
- Conocimientos basicos en Linux ( no excluyente )
- Paciencia
- Tener cuenta en [Docker Hub][Hub]

[Hub]: https://hub.docker.com

Estructura
========

- Dockerfiles para contruir el laboratorio: master + nodos
- Instalacion de Ansible
- Generacion de key ssh ( master ) 
- Copiado de key ssh en nodos 
- Ejemplos basicos de Ansible
- Playbooks

A la Marcha 
==========

### Creacion de Ambiente


1. Crear una instancia en [Play With Docker][Lab]  ( con las credenciales de Docker hub, pueden hacer el login )
2. Con la instancia creada, clonar el [repositorio][Repo] en : /home/ 

[Lab]: https://labs.play-with-docker.com

[Repo]: https://github.com/kdetony/docker_ansible.git

### Creacion nodo Master 

1. En la carpeta **master**, encontramos un Dockerfile, con el, vamos a crear nuestro nodo master, de la siguiente manera: 

> docker build -t master . 

2. A continuacion, vamos a crear el contenedor de la siguiente manera: 

> docker run -dit -p :22 --name master master

3. Vamos a generar las llaves ssh, para ello, entramos al contenedor para ejecutarlo: 

> docker exec -it master bash 

> ssh-keygen -t rsa 

4. En el contenedor, nos dirigimos a la carpeta **/root/.ssh** y realizamos un cat a **id_rsa.pub**

OBS.

Guardar la salida de id_rsa.pub en un archivo de texto ( de preferencia ) 

### Creacion de nodos Slave 

1. En la carpeta **nodo_centos** encontramos un Dockerfile, con el, vamos a crear nuestro primer nodo slave, de la siguiente manera:

> docker build -t nodoc .

2. Al igual que en punto 1, repetimos lo mismo en la carpeta **nodo_ubuntu**: 

> docker build -t nodou .

3. Creamos los contenedores de la siguiente manera: 

> docker run -dit -p :22 --name nodoc nodoc

> docker run -dit -p :22 --name nodou nodou 

4. Vamos a validar la coneccion por llaves desde el master, para lo cual vamos a realizar una modificacion al archivo **/etc/hosts** en el nodo master.

> ip_nodoc   nodoc 
> ip_nodou   nodou 

validamos: 

> ssh nodoc |  ssh nodou 

### Ansible - Comandos y Ejercicios 

¿Que W&$@F es Ansible?

- Es un software que automatiza el aprovisionamiento de software, la gestión de configuraciones y el despliegue de aplicaciones.
- Permite a los SysAdmins/DevOps/SRE gestionar sus servidores, configuraciones y aplicaciones de forma sencilla, robusta y paralela
- Gestiona diferentes nodos a través de SSH y únicamente requiere Python en el servidor remoto en el que se vaya a ejecutar para poder utilizarlo. 
- Usa YAML para describir acciones a realizar y las configuraciones que se deben propagar a los diferentes nodos.


### Arquitectura 

![Diagrama_Infra](https://github.com/kdetony/docker_ansible/blob/master/images/ansible.png "diagrama")

### Let's Go 

1. Entramos en el contenedor nodo master para ejecutar los primeros comandos de Ansible, 


