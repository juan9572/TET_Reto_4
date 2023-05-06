# TET_Reto_N4
> En este repositorio se aloja nuestro proyecto de despliegue de Moodle utilizando servicios de Amazon Web Services (AWS) como Elastic Load Balancer (ELB), Elastic File System (EFS), Relational Database Service (RDS), Certificate Manager (ACM), EC2 y Auto Scaling Group. Este proyecto tiene como objetivo proporcionar un enfoque escalable, seguro y de alta disponibilidad para el despliegue de Moodle en la nube utilizando la infraestructura de AWS. Además, se ha implementado la seguridad adicional al servir el contenido a través de HTTPS, garantizando la integridad y confidencialidad de la información transmitida entre el servidor y el cliente.
## Tabla de contenidos:
---

1.  [Información de la asignatura](#información-de-la-asignatura)

2. [Datos de los estudiantes](#datos-de-los-estudiantes)

3. [Descripción y alcance del proyecto](#descripción-y-alcance-del-proyecto)

4. [Arquitectura de despliegue](#arquitectura-de-despliegue)

6. [Paso a paso del despliegue](#paso-a-paso-del-despliegue)
	 - [Pre-requisitos](#pre-requisitos)
	 - [NFS](#nfs)
	 - [Base de datos](#base-de-datos)
	 - [Moodle](#moodle)
		 * [Instalación de Moodle](#instalación-de-moodle)
		 * [Instalación de VPL JAIL SERVER](#instalación-de-vpl-jail-server)
	 - [Balanceador de carga](#balanceador-de-carga)
		 * [Creando grupo de balanceo](#creando-grupo-de-balanceo)
		 * [Creando certificado SSL](#creando-certificado-ssl)
	- [Grupo de escalamiento automático](#grupo-de-escalamiento-automático)
7. [Referencias](#referencias) 

## Información de la asignatura
---

 -  **Nombre de la asignatura:** Tópicos especiales en telemática.
-   **Código de la asignatura:**  C2361-ST0263-4528
-   **Departamento:** Departamento de Informática y Sistemas (Universidad EAFIT).
-   **Nombre del profesor:** Juan Carlos Montoya Mendoza.
-  **Correo electrónico del docente:** __[jcmontoy@eafit.edu.co](mailto:jcmontoy@eafit.edu.co)__.

## Datos de los estudiantes
---

-   **Nombre del estudiante:** Juan Pablo Rincon Usma.
-  **Correo electrónico del estudiante:** __[jprinconu@eafit.edu.co](mailto:jprinconu@eafit.edu.co)__.
-   **Nombre del estudiante:** Julian Gámez Benítez.
-  **Correo electrónico del estudiante:** __[jgomezb11@eafit.edu.co](mailto:jgomezb11@eafit.edu.co)__.

## Descripción y alcance del proyecto
---

El objetivo principal de este proyecto fue adquirir experiencia práctica en la implementación de una solución escalable y segura para el despliegue de Moodle en la nube utilizando la infraestructura de AWS. Además, se buscó familiarizarse con el uso de varios servicios de AWS, como ELB, EFS, RDS, ACM, EC2 y Auto Scaling Group, y comprender cómo funcionan en conjunto para proporcionar una solución completa.

El alcance del proyecto incluyó la creación de una infraestructura de AWS escalable y segura para el despliegue de Moodle, la configuración y el aprovisionamiento de instancias de EC2 y RDS, la configuración de los servicios de red de AWS, como ELB y Auto Scaling Group, y la implementación de HTTPS para garantizar la seguridad en la transferencia de datos. Tambien se abarco la configuracion de EFS como servidor NFS para todas las replicas del auto scaling group.


## Arquitectura de despliegue
---
![Arquitectura](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Arquitectura.png)

La arquitectura de este proyecto está diseñada para proporcionar una solución escalable y segura para el despliegue de Moodle en la nube utilizando la infraestructura de AWS. La arquitectura se compone de varios componentes que trabajan juntos para proporcionar una solución completa. El componente central de la arquitectura es el Elastic Load Balancer (ELB), que se encarga de distribuir el tráfico entre las instancias de EC2 en el Auto Scaling Group. El Auto Scaling Group se compone de varias instancias de EC2, cada una de las cuales contiene una plantilla de servidor Moodle.
Cada instancia de EC2 está configurada para conectarse a una base de datos RDS (mysql) como base de datos principal y se conecta a un sistema de archivos remoto EFS para garantizar que todas las réplicas estén en sincronía en terminos de datos y de archivos.
Además, la aplicación Moodle está configurada para conectarse a un VPL Server para efectos de laboratorios de programación.
Toda la arquitectura se configura para utilizar HTTPS para garantizar la seguridad en la transferencia de datos entre el servidor Moodle y el cliente. Esta solución proporciona una arquitectura escalable y segura para el despliegue de Moodle en la nube utilizando la infraestructura de AWS.
## Paso a paso del despliegue
---

### 1. Pre-requisitos

Cada uno de estos pre-requisitos es fundamental para asegurar el correcto funcionamiento de la solución desplegada, por lo que es importante llevar a cabo estos pasos antes de comenzar con el despliegue.

 - 1 instancias EC2 en AWS, usando Ubuntu 22.04 como SO y una t2.micro como CPU.
- Crear un security group el cual permita conexiones por el puerto 80 en las instancias que servirán como servidores web de Moodle.
- Crear un security group el cual permita conexiones por el puerto 443 para poder decirle al balanceador de carga que escuche peticiones HTTPS.
![sg3](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-1.png)

![sg4](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-2.png)

![sg5](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-4.png)

- Crear un security group el cual permita conexiones por el puerto 2049 para poder realizar el montaje del servidor NFS.
![sg](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/NFS/NFSP-1.png)

![sg2](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/NFS/NFSP-2.png)

- Crear un security group el cual permita conexiones por el puerto 3306 para poder realizar conexiones a la base de datos que crearemos.
Es importante destacar que aunque en este proyecto se está utilizando AWS, los pre-requisitos necesarios son similares independientemente de la plataforma en la que se despliegue la solución.
Una vez tengamos listo estos requisitos, podremos seguir con el despliegue.

### 2. NFS
Para nuestro NFS vamos a utilizar el servicio de AWS llamado EFS, para crearlo nos dirigimos a la sección de servicios -> Almacenamiento -> EFS.

![nfs1](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/NFS/NFS-1.png)

Una vez en esa sección, le damos a crear un sistema de archivos.

![nfs2](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/NFS/NFS-2.png)

Cuando nos salga la ventana le damos al boton de "Personalizar".

![nfs3](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/NFS/NFS-3.png)

Elegimos la personalizacion estandar y activamos el checkbox que nos habilita las copias de seguridad.

![nfs4](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/NFS/NFS-4.png)

En la sección de "Acceso a la red" elegimos la VPC en la que estamos desplegando los recursos de nuestro proyecto, elegimos una *avaliability zone* y le asignamos el *security group* que creamos anteriormente para el EFS.

![nfs5](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/NFS/NFS-5.png)

Despues de creado nos deberia salir una pantalla como la siguiente.	

![nfs6](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/NFS/NFS-7.png)

Luego nos dirigimos a la sección sistemas de archivos, seleccionamos el que acabamos de crear y le damos en "Ver detalles".

![nfs7](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/NFS/NFS-8.png)

Una vez ahí, le damos al boton "Asociar".

![nfs8](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/NFS/NFS-9.png)

Y seleccionamos el "Montaje a través de DNS" y nos sale ese comando. Con ese comando podemos hacer el montaje cuando estemos intalando la imagen de moodle para almacenar todos los archivos de instalacion en el NFS server.

![nfs9](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/NFS/NFS-10.png)

### 3. Base de datos

Nos dirigimos a la seccion de "Servicios".

![bd1](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/DB/DB-1.png)

Un vez ahi le damos click al item "Base de datos" y luego seleccionamos "RDS".

![bd2](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/DB/DB-2.png)

Una vez dentro, le damos al boton para crear la base de datos.

![bd3](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/DB/DB-3.png)

Elegimos la creacion estandar y seleccionamos el motor "MySQL".

![bd4](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/DB/DB-4.png)

Le damos click al checkbox de "Capa gratuita", le ponemos un nombre a nuestra base de datos y le definimos las credenciales de acceso (user y password).

![bd5](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/DB/DB-5.png)

Elegimos la opcion de "Clases con rafagas", seleccionamos la *db.t3.micro*.

![bd6](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/DB/DB-6.png)

Optamos por unas 20GiB SSD de almacenamiento con escalado automatico integrado y con un maximo de 1000 GiB.

![bd7](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/DB/DB-7.png)

Configuramos la conectividad de tal manera que no se conecte a un EC2 especifico y le asignamos la misma VPC que a los demas recursos.

![bd8](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/DB/Db-7.5.png)

Con eso deberiamos estar listos, le damos a crear la database y nos debe redireccionar al dashboard. Una vez ahi le damos click a la base de datos que acabamos de crear.

![bd9](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/DB/DB-8.png)

Y si bajamos un poco deberiamos ver el punto de enlace con el que, desde un cliente MySQL, nos  podemos conectar a la base de datos con las credenciales que definimos.

![bd10](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/DB/DB-9.png)
### 4. Moodle
Ahora procederemos a conectarnos a nuestra máquina mediante SSH, cuando estemos en esta vamos a instalar Docker para poder descarga una imagen de Moodle, estos pasos están en la documentación oficial de Docker.
1.  Desinstalar versiones antiguas de Docker usando el siguiente comando:
```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```
2.  Configurar el repositorio Ubuntu usando los siguientes comandos:
```bash
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg
```
3.  Añadir la clave GPG oficial de Docker usando el siguiente comando:
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
4. Usar el siguiente comando para configurar el repositorio:
```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
5.  Actualizar la lista de paquetes usando el siguiente comando:
```bash
sudo apt-get update
```
6. Instalar Docker Engine, containerd y Docker Compose usando el siguiente comando:
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Ahora tenemos Docker listo en nuestra máquina, necesitaremos poder acceder a nuestra instancia de la base de datos creada anteriormente y poder crear una una base de datos para que Moodle pueda hacer la configuración de todo su contenido allí, para esto necesitaremos instalar un cliente de MYSQL y creamos una base de datos.
- Creamos el cliente:
	```bash
	sudo apt-get update
	sudo apt-get install mysql-client
	```
- Nos conectamos a la db:
	```bash
	mysql -h db-moodle.c5nfjbrq8d6k.us-east-1.rds.amazonaws.com -u admin -p
	```
- Creamos la base de datos para Moodle:
	```sql
	 CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
	```
Ahora con esto si podremos instalar nuestro Moodle.

#### 4.1 Instalación de Moodle
- Creamos un [docker-compose.yml](https://github.com/juan9572/TET_Reto_4/blob/main/docker-compose.yml) de la imagen de bitnami/moodle, con el montaje del NFS:
	```yml
	version: '3'
	services:
	 moodle:
	   image: bitnami/moodle:latest
	   privileged: true
	   ports:
	     - "80:8080"
	   environment:
	     - MOODLE_DATABASE_USER=admin
	     - MOODLE_DATABASE_TYPE=mysqli
	     - MOODLE_DATABASE_PASSWORD=12345678
	     - MOODLE_DATABASE_NAME=moodle
	     - MOODLE_DATABASE_HOST=db-moodle.c5nfjbrq8d6k.us-east-1.rds.amazonaws.com
	   volumes:
	     - moodle_data:/bitnami/moodle
	     - nfs-data:/bitnami/moodle
	   restart: always
	volumes:
	 moodle_data:
	 nfs-data:
	   driver: local
	   driver_opts:
	     type: nfs
	     o: addr=fs-08a80be0aa8fe7d6d.efs.us-east-1.amazonaws.com,vers=4.1,hard,rsize=1048576,wsize=1048576,timeo=600,retrans=2,noresvport
	     device: ":/"
	```
Como se puede apreciar en __environment__ en  "MOODLE_DATABASE_HOST" se coloca el link que vimos cuando configuramos la base de datos y en el volumen __nfs-data__ en "addr" se coloca el link que nos salía para realizar el montaje.
- Corremos el docker-compose.yml
``` bash
sudo docker compose up -d
```
Después de realizar esto el contendor instalara Moodle, inicializara la base de datos y creara el montaje en el directorio de los archivos fuentes de Moodle, ahora lo que haremos es entrar a este Moodle mediante la **IP** de nuestra máquina por el **puerto 80** y vamos al apartado de **"Sing in"**.
![m1](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-1.png)
Nos loguemos con nuestro usuario, por defecto en la imagen de bitnami dice que le "usuario" es "user" y el "password" es "bitnami".
![m2](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-2.png)
Puede que nos pida registrar nuestra página le damos registrar y allí nos pedirán los datos relacionados al dueño de la página y cuando lo hagamos veremos la página funcionando correctamente.
![m3](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-3.png)
#### 4.2 Instalación de VPL JAIL SERVER
Creamos otra EC2 y siguiendo los mismo pasos de instalación de Docker, vamos a proceder hasta que ya tengamos listo el Docker.

Una vez listo le vamos a decir que corra una imagen de Docker para poder ejecutar remotamente el VPL-Jail-System que es el juez para el tema de compilar los programas y ejecutarlos.
```bash
	sudo docker run -p 80:81 --name vpl --restart=always --privileged -d gabrielbdec/vpl-jail:3.0.0
```
Ahora siguiendo desde nuestro Moodle.
Una vez logueados nos vamos a la configuración del sitio.
![m4](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-4.png)
Hay una pestaña llamada "plguins" le damos allí y le damos "Install plugins from the Moodle plugins directory", cuando le damos allí nos llevará al repositorio de Moodle donde se encuentran todos los plugins que podemos usar para la página
![m5](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-5.png)
Buscamos el plugin para el VPL
![m6](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-6.png)
Lo descargamos
![m7](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-7.png)
Y ahora podemos instalarlo, desde el zip que nos hemos descargado.
![m8](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-8.png)
Lo instalamos
![m9](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-9.png)
Le decimos que actualice la base de datos y siguiente.. siguiente..
![m10](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-10.png)
y cuando termine podremos encontrar que el plugin ya esta disponible, si le damos allí podemos ver las configuraciones.
![m11](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-11.png)
Buscamos la configuración del Execution servers config y allí ponemos la ip de nuestra máquina la cual tiene el VPL-Jail-System corriendo.
![m12](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-12.png)
Ahora volvemos al inicio y creamos un curso para nuestro sitio de Moodle
![m13](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-13.png)
Ponemos su nombre y su nombre abreviado.
![m14](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-14.png)
Cuando lo creemos deberemos ver algo así, ahora agregaremos una actividad de Moodle de VPL, para ello nos ponemos en el modo de edición:
![m15](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-15.png)
Ahora creamos una nueva actividad.
![m16](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-16.png)
Le indicamos que sea de tipo VPL
![m17](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-17.png)
Colocamos el nombre de la actividad y la creamos.
![m18](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-18.png)
Y ahora tenemos nuestra actividad de VPL lista corriendo con su remote VPL-Jail-System corriendo.
### 5. Balanceador de carga
Ahora procederemos a crear nuestro balanceador de carga el cual se integrara con el auto scaling group, para ello volvemos a nuestro AWS y buscamos en la sección de EC2, **"Balanceadores de carga"**
![bl](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-5.png)
Allí le daremos crear nuevo balanceador de carga
![bl1](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-6.png)
Veremos 3 opciones de balanceadores, en este caso seleccionamos la opción de balanceador de carga de aplicación.
![bl3](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-7.png)
Allí colocaremos su nombre, de que tipo será y como se puede conectar a este.
![bl4](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-8.png)
le indicamos que mapee todas las zonas de disponibilidad y le seleccionamos la VPC
![bl5](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-9.png)
Agregamos el sucrity group que ya creamos anteriormente.
![bl10](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-10.png)
Configuramos un listener en el puerto 443 y le damos crear grupo
![bl11](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-11.png)
#### 5.1 Creando grupo de balanceo
En la creación del grupo le indicamos que es un grupo de instancias
![bl12](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-12.png)
Ponemos el nombre del grupo, el puerto donde las máquinas de este grupo están escuchando, el vpc y el protocolo.
![bl13](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-13.png)
además indicamos cual será la forma validar de que la máquina esta funcionando correctamente
![bl14](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-14.png)
por último se nos pide indicar cuales instancias queremos para nuestro grupo de nuestro balanceador de carga, por el momento dejemos solo la que configuramos para Moodle
![bl15](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-15.png)
Ahora volviendo a nuestro balanceador de carga, recargamos lo grupos y seleccionamos el que acabamos de crear
![bl16](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-16.png)
#### 5.2 Creando certificado SSL
Después nos pedirán el certificado SSL para nuestro balanceador de carga, le damos en solicitar uno con ACM
![bl17](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-17.png)
Le damos solicitar
![bl18](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-18.png)
Le indicamos que sea público
![bl19](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-19.png)
ponemos el nombre de nuestro dominio web y el método en el cual podemos validar que ese dominio nos pertenezca, en este caso lo haremos por medio del DNS.
![bl20](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-20.png)
Cuando lo solicitemos iremos a verlos y seleccionamos el que acabamos de crear
![bl21](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-21.png)
allí encontraremos las instrucciones que debemos hacer para autenticar el certificado SSL, necesitaremos crear un registro CNAME en nuestro DNS que contenga la siguiente información.
![bl22](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-22.png)
vamos a nuestro proveedor del dominio y en nuestros DNS, agregamos el que necesitamos.
![bl23](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-23.png)
Pasado un rato, cuando los registros del DNS se hayan esparcido por internet, podremos volver a consultar el estado de nuestro certificado y ver si ya esta disponible
![bl24](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-24.png)

Si estos es correcto podremos volver a la configuración de nuestro balanceador de carga, recargamos los certificados y seleccionamos el creado anteriormente.
![bl25](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-25.png)
Por último nuestro balanceador de carga se debe ver algo así y lo creamos.
![bl26](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-26.png)
Seleccionamos nuestro balanceador de carga y podremos ver el registro A que debemos poner en nuestro DNS del dominio web
![bl27](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-27.png)
Ahora lo colocamos y ya es accesible nuestro sitio web desde el dominio web, ya que este apunta al balanceador de carga y este ya nos dará alguna instancia de Moodle
![bl28](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-28.png)
Por último activaremos el sticky section, para ello nos iremos al apartado de balanceadores de carga, seleccionamos el creado y le damos listeners.
![bl29](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-29.png)
allí veremos el grupo que habíamos creado, lo seleccionamos
![bl30](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-30.png)
y ahora en el grupo le damos attributes y edit
![bl31](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-31.png)
bajamos hasta que encontremos la opción Stickness y lo dejamos encargado al balanceador de carga y guardamos los cambios.
![bl32](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-32.png)
Ahora si accedemos a nuestra web podremos ver que todo esta funcionado bien.
![bl33](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-33.png)
### 6. Grupo de escalamiento automático

En el dashboard de EC2 nos vamos a la seccion de "Instancias", seleccionamos nuestra maquina de Moodle, click derecho y vamos a "Imagen y plantillas" -> Crear Imagen.

![asg-0](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-0.png)

Una vez ahi, le ponemos un nombre y una breve descripcion a la imagen. Habilitamos el "Sin reiniciar".

![asg-0-5](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-0-5.png)

En el dashboard de EC2 vamos a la seccion de "Plantillas de lanzamiento" y le damos al boton de "Crear plantilla de lanzamiento".

![asg-2](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-2.png)

Cuando nos redireccione le ponemos un nombre y una breve descripcion a la plantilla. Por ultimo habilitamos la opcion de "Orientacion sobre Auto Scaling".

![asg-3](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-3.png)

Especificamos que la imagen es de nuestra propiedad y le asignamos la imagen que creamos previamente.

![asg-4](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-4.png)

Elegimos la *t2.micro* que es gratis y le asignamos un key pair.

![asg-5](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-5.png)

En "Subred" elegimos "No incluir en la plantilla de lanzamiento". Seleccionamos un grupo de seguridad existente y le ponemos la que previamente habiamos creado.

![asg-6](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-6.png)

Le damos a "Crear grupo de Auto Scaling.

![asg-7](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-7.png)

Le ponemos un nombre al grupo de auto scaling. Elegimos la plantilla que creamos previamente y dejamos la version en Default.

![asg-8](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-8.png)

Seleccionamos la misma VPC que en todos los recursos y habilitamos todas las *avaliability zones* disponibles.

![asg-9](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-9.png)

Le asociamos un balanceador de carga existente. Y elegimos entre los target groups que tenemos disponibles. En nuestro caso el target group se llama "Moodle-Group | HTTP".

![asg-10](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-10.png)

Seleccionamos la opcion que dice que no existe un VPC Lattice para asociar. Y activamos las comprobaciones de estado de Elastic Load Balancing.

![asg-11](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-11.png)

Le ponemos 300 segundos en el periodo de gracia de comprobacion de estado y habilitamos la recopilacion de CloudWatch.

![asg-12](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-12.png)

En esta seccion especificamos el maximo, el minimo y el deseado de la cantidad de maquinas que queremos tener corriendo a la vez.

![asg-13](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-13.png)

En politicas de escalado seleccionamos la que es de seguimiento de destino y el tipo de metrica usada va a ser "Utilizacion de la CPU" en donde se espera que un valor de utilizacion promedio sea del 50%.

![asg-14](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-14.png)

Omitimos la opcion de recibir notificaciones.

![asg-15](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-15.png)

No agregamos etiquetas.

![asg-16](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Auto%20scaling%20group/ASG-16.png)

## Referencias
---

- [Auto Scaling group with load balancing](https://docs.aws.amazon.com/autoscaling/ec2/userguide/attach-load-balancer-asg.html)
- [Download moodle](https://docs.moodle.org/402/en/Git_for_Administrators)
- [VPL Jail System’s documentation](https://vpl.dis.ulpgc.es/documentation/vpl-jail-system-2.6.0/index.html)
- [Bitnami moodle](https://hub.docker.com/r/bitnami/moodle)
- [Install Docker Desktop on Ubuntu](https://docs.docker.com/desktop/install/ubuntu/)
