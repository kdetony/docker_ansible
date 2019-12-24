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

1. Guardar la salida de id_rsa.pub en un archivo de texto ( de preferencia ) 
2. Ejecutar: echo "id_rsa.pub" > authorized_keys ( copia de llave SSH ) 


### Creacion de nodos Slave 

1. En la carpeta **nodo_centos** encontramos un Dockerfile, con el, vamos a crear nuestro primer nodo slave, de la siguiente manera:

> docker build -t nodoc .

2. Al igual que en punto 1, repetimos lo mismo en la carpeta **nodo_ubuntu**: 

> docker build -t nodou .

3. Creamos los contenedores de la siguiente manera: 

> docker run -dit -p :22 --name nodoc nodoc

> docker run -dit -p :22 --name nodou nodou 

OBS.

1. Entrar a cada contenedor y ejecutar: 

> ssh-keygen -t rsa 

2. En el directorio: **/root/.ssh** de cada contenedor ejecutar lo siguiente:

> echo "id_pub.rsa" > authorized_keys 

Donde **id_pub.rsa** es la llave del servidor master, la cual copiamos en el punto : **Creacion nodo Master / punto4**


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

1. Entramos en el contenedor nodo master para ejecutar los primeros comandos de Ansible, y nos ubicamos en la carpeta /etc/ansible, aqui vamos a crear el archivo **hosts** con el sgt contenido: 

```
[all]
172.17.0.2
172.17.0.3
172.17.0.4 

[nodos]
172.17.0.3
172.17.0.4
```

2. Ahora vamos a ejecutar lo siguiente: 

> ansible all -m ping -u root

3. Otras consultas mediante ansible: 

> ansible all -a "uptime" 

> ansible all -a "ps fax"

> ansible all -a "hostname"

> ansible all -m shell -a "uname -a"

OBS:

- El módulo shell nos permite ejecutar comandos sobre cada uno de los nodos.
- En este caso el modificador **-m**  nos permite indicar el módulo que queremos utilizar y el 
  modificador **-a** nos permite indicar el comando.
- Si no indicamos nigun modulo, por defecto Ansible utilizara el modulo **shell**.  
- Por lo tanto, el siguiente ejemplo realizaría la misma acción que el comando anterio: 

> ansible all -a "uname -a" 

4. Vamos ahora a cambiar el archivo **hosts** con lo siguiente: 

[webserver]
nodec ansible_host=172.17.0.4 

Ejecutamos:

> ansible -m ping nodoc

Volvemos a modificar el archivo **hosts** con lo siguiente:

[webserver]
nodec ansible_connection=ssh ansible_host=172.17.0.4 ansible_port=22

Donde: 

-webserver: Nombres de grupos, que se utilizan en la clasificación de los sistemas.

-nodec: alias del equipo (no es necesario que sea el verdadero nombre del host).

-ansible_connection: tipo de conexión, éste puede ser el nombre de cualquiera de los plugins de conexión de ansible(local, docker), en nuestro caso SSH.

-ansible_host: host a conectar, en nuestro caso la ip 172.17.0.4.

-ansible_port: puerto de conexión (en caso de ssh tener otro puerto), en nuestro caso 22.

Ejecutamos: 

> ansible -m ping nodec

6. Pongamos el sgt. ejemplo, las conexión deben efectuarse desde el usuario al cual se le configura la relación de confianza con la conexión SSH, en el caso operar con un usuario distinto al usuario administrador de ansible (root), se puede configurar la conexión de la siguiente forma, para lo cual, volvemos a modificar el archivo **hosts**

[webserver]
nodec ansible_connection=ssh ansible_host=172.17.0.4 ansible_port=22 ansible_user=root ansible_ssh_private_key_file=/root/.ssh/id_rsa


5. Vamos ahora a ejecutar nuestro primer playbook, para ello, vamos a crear el archivo: play.yml

```
---
- hosts: srv_redhat
  tasks:
    - name: Validar la ultima version de VIM 
      yum: name=vim state=latest 
```
OBS.

Vamos a modificar nuestro arhcivo **hosts**  el cual debe quedar asi: 

```
[all]
172.17.0.2
172.17.0.3
172.17.0.4

[nodos]
172.17.0.3
172.17.0.4

[srv_redhat]
172.17.0.2
172.17.0.4

[srv_ubuntu]
172.17.0.3

[webserver]
nodec ansible_connection=ssh ansible_host=172.17.0.4 ansible_port=22
```

Y ejecutamos nuestro playbook de la siguiente manera: 

> ansible-playbook play.yml

6. Vamos a crear otro playbook, el cual se llamara paquetes.yml y tendra el siguiente contenido: 
```
---
- hosts: srv_redhat
  tasks:
    - name: Instalacion de Paquetes
      yum: name={{ item }} state=latest
      with_items:
        - net-tools
        - elinks
        - git 
```

Lo ejecutamos

> ansible-playbook paquetes.yml

Veamos otro playbook, el cual se llamara user.yml 
```
---
- hosts: all
  tasks:
  - name: Creacion de Usuario 
    user:
      name: bbva
      groups: 
       - adm
       - users
      state: present
      shell: /bin/bash       
      system: no             
      createhome: yes        
      home: /home/bbva  
```

7. Un punto fuerte de Ansible son los **Roles** para tener todo mas organizado. Los roles no son mas que playbooks que son llamados modularmente pero divididos en vez de en secciones en carpetas. Para hacerse una idea es similar a un modulo de **puppet** :) 

```
roles/
   common/
     files/
     templates/
     tasks/
     handlers/
     vars/
     defaults/
     meta/
```

Como se puede apreciar, el nombre del rol es common y luego de el cuelgan todas las carpetas donde habrá que meter los playbooks que queramos

Para hacer un PoC de esto, vamos a instalar apache. Para comenzar, hay que generar la estructura de rol. 

mkdir /etc/ansible/roles/apache && cd /etc/ansible/roles/apache && mkdir files handlers meta templates tasks vars

9. Vemos un ejemplo de **Task** para ello creamos un directorio llamado task y dentro de el **main.yml** 
```
---
- name: Instalacion de Apache
  yum: name=httpd state=latest
  notify:
    - start httpd

- name: Eliminar el index.html
  command: rm /var/www/html/index.html
  register: html_old
  ignore_errors: True

- name: Subir el index.html que queramos
  copy: src=index.html dest=/var/www/html mode=0644
  when: html_old|success
```
