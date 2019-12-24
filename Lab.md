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
- Tener cuenta en [Docker Hub][https://hub.docker.com]

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


1. Crear una instancia en [Play With Docker][https://labs.play-with-docker.com]  ( con las credenciales de Docker hub, pueden hacer el login )
2. Con la instancia creada, clonar el [repositorio][https://github.com/kdetony/docker_ansible.git] en : /home/ 


### Creacion nodo Master 

1. En la carpeta master, enctramos un Dockerfile, con el, vamos a crear nuestro nodo master, de la siguiente manera: 

> docker build -t master . 

2. A continuacion, vamos a crear el contenedor de la siguiente manera: 

> docker run -dit -p :22 --name master master

3. Vamos a generar las llaves ssh, para ello, entramos al contenedor para ejecutarlo: 

> docker exec -it master bash 

> ssh-keygen -t rsa 

4. En el contenedor, nos dirigimos a la carpeta /root/.ssh y realizamos un cat a **id_rsa.pub**

