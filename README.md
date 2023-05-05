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

#### 4.1 Instalación de Moodle
![m1](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-1.png)

![m2](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-2.png)

![m3](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-3.png)

#### 4.2 Instalación de VPL JAIL SERVER
![m4](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-4.png)

![m5](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-5.png)

![m6](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-6.png)

![m7](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-7.png)

![m8](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-8.png)

![m9](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-9.png)

![m10](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-10.png)

![m11](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-11.png)

![m12](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-12.png)

![m13](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-13.png)

![m14](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-14.png)

![m15](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-15.png)

![m16](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-16.png)

![m17](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-17.png)

![m18](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/Moodle/Moodle-18.png)
### 5. Balanceador de carga
![bl](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-5.png)

![bl1](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-6.png)

![bl3](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-7.png)

![bl4](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-8.png)

![bl5](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-9.png)

![bl10](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-10.png)

![bl11](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-11.png)

![bl12](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-12.png)

![bl13](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-13.png)

![bl14](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-14.png)

![bl15](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-15.png)

![bl16](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-16.png)

![bl17](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-17.png)

![bl18](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-18.png)

![bl19](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-19.png)

![bl20](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-20.png)

![bl21](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-21.png)

![bl22](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-22.png)

![bl23](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-23.png)

![bl24](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-24.png)

![bl25](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-25.png)

![bl26](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-26.png)

![bl27](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-27.png)

![bl28](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-28.png)

![bl29](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-29.png)

![bl30](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-30.png)

![bl31](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-31.png)

![bl32](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-32.png)

![bl33](https://raw.githubusercontent.com/juan9572/TET_Reto_4/main/LB/LB-33.png)
#### 5.1 Creando grupo de balanceo

#### 5.2 Creando certificado SSL

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

- [How to Configure Apache Load Balancer](https://www.inmotionhosting.com/support/server/apache/apache-load-balancer/)
- [Get Certbot (Docker)](https://eff-certbot.readthedocs.io/en/stable/install.html#alternative-1-docker)
- [Setup a NFS Server With Docker](https://blog.ruanbekker.com/blog/2020/09/20/setup-a-nfs-server-with-docker/)
- [Install Docker Desktop on Ubuntu](https://docs.docker.com/desktop/install/ubuntu/)
