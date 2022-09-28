# Modernización de Aplicaciones 2.0 - Dojo / página 2

<p align="center">
  <img src="/././images/logoprincipal.png" width="500">
</p>

* [← Volver al índice](/././README.md)

## Docker

En la [sección anterior](/./pages/1/README.md) estuvimos viendo algunos conceptos básicos de docker, en esta sección profundizaremos sobre algunos de ellos y comenzaremos a trabajar de forma práctica.  


**Indice de la sección:**

- [Modernización de Aplicaciones 2.0 - Dojo / página 2](#modernización-de-aplicaciones-20---dojo--página-2)
  - [Docker](#docker)
    - [Instalación y configuración](#instalación-y-configuración)
  - [Hello World](#hello-world)
  - [Crear y ejecutar una imagen propia](#crear-y-ejecutar-una-imagen-propia)
    - [Paso 1 - dockerfile](#paso-1---dockerfile)
    - [Datos a tener en cuenta antes de armar nuestro primer dockerfile:](#datos-a-tener-en-cuenta-antes-de-armar-nuestro-primer-dockerfile)
    - [Paso 2 - dockerignore](#paso-2---dockerignore)
    - [Paso 3 - build](#paso-3---build)
    - [Paso 4 - run](#paso-4---run)
  - [Las imágenes están formadas en capas.](#las-imágenes-están-formadas-en-capas)
  - [Almacenamiento en caché de capas](#almacenamiento-en-caché-de-capas)
  - [Docker Hub](#docker-hub)
    - [¿Qué es?](#qué-es)
    - [Crear una cuenta e iniciar sesion](#crear-una-cuenta-e-iniciar-sesion)
    - [Subir nuestra imagen](#subir-nuestra-imagen)
    - [Catálogo](#catálogo)
    - [Descargar y ejecutar Mongo DB de Docker Hub](#descargar-y-ejecutar-mongo-db-de-docker-hub)
      - [¿Cómo lo utilizo?](#cómo-lo-utilizo)
  - [Container Registry IBM](#container-registry-ibm)
      - [Configuraciones iniciales](#configuraciones-iniciales)
      - [Subir nuestra imagen](#subir-nuestra-imagen-1)
  - [Apéndice](#apéndice)
    - [Hands-On](#hands-on)
    - [Material extra](#material-extra)

<p align="center">
  <img src="/././images/docker.png" width="200">
</p>

### Instalación y configuración

¡Comencemos con la instalación de docker! 

En este dojo vamos a estar trabajando con la versión CE (community edition) de docker.  A continuación, veremos los pasos a seguir para su instalación en Ubuntu, en [este link](https://docs.docker.com/install/) puede encontrar el instructivo para instalarlo en cualquier otro SO

La instalación es muy sencilla, primero realicemos los siguientes pasos para configurar el repositorio:

1. Actualizar índice *apt* de nuestro sistema 
```
 $ sudo apt-get update
```
2. Luego instalaremos los siguientes paquetes para el uso de repositorios sobre HTTPS
```
 $ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```
3. Agreguemos la GPC key oficial de docker
```
 $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

```
4. Por último debemos añadir el repositorio desde el cual bajaremos Docker
```
 $ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

Una vez completados los pasos de arriba podemos comenzar la instalación. Para instalar docker solo debemos ejecutar el siguiente comando en nuestra consola. 

```
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Por último, si queremos verificar que la instalación se haya llevado a cabo correctamente, podemos ejecutar el siguiente comando:

```
$ docker -v
```
En el caso de que la instalación haya sido exitosa, se mostrará en la consola la versión de docker que tienen instalada. 

A partir de este punto estamos listos para comenzar a usar docker

## Hello World

Ahora que tenemos instalado docker vamos a correr una imagen de prueba, pero primero recordemos el concepto de imagen que vimos en la sección anterior. 

<p align="center">
  <img src="/././images/hello-world.png" width="250">
</p>

**Imagen:** Una imagen puede considerarse como un paquete ejecutable (por docker engine), liviano y auto-contenido. Además, una imagen posee todo lo necesario para ejecutar el software, incluyendo el código del mismo, le runtime, las librerías, los archivos de configuración, etc.  
Hay dos formas de obtener imágenes para nuestros contenedores. La primera es generarlas nosotros mismos, más adelante veremos cómo. La segunda es obtenerlas al descargarlas de un repositorio privado o público. Gracias a la popularidad de Docker, la cantidad de imagenes públicas es muy amplia y podemos encontrar desde una base de datos hasta un gestor de contenidos o sistema operativo. 

```
sudo docker run hello-world
```

Este comando lo que haces es descargar una imagen de prueba desde Docker Hub y la ejecuta en un contenedor. Cuando se ejecuta el contenedor, se imprime un mensaje y luego sale.

## Crear y ejecutar una imagen propia

<p align="center">
  <img src="/././images/build-docker-image-cover.png" width="350">
</p>

La idea ahora es crear nuestra propia imagen y utilizarla para correr un contenedor de docker. Para este ejercicio haremos un simple hello world en Nodejs. En la carpeta [assets/lab1](/assets/docker/lab1/#)  encuentran los archivos utilizados. 

### Paso 1 - dockerfile

Una vez que tenemos nuestro código, lo primero es crear el dockerfile

Bien vamos muy rápido, se preguntaran ¿qué es un dockerfile ?

Dockerfile es un archivo de texto que contiene un conjunto de instrucciones o comandos para la creación de una imagen docker. Llamando a la función docker build se leen las instrucciones del Dockerfile y se construye una imagen que se guardará en el repositorio local de docker.

### Datos a tener en cuenta antes de armar nuestro primer dockerfile:

* La INSTRUCCION no es case-sensitive y por costumbre se suele usar en mayúscula.

* El orden de las instrucciones es importante ya que debe seguir el orden del proceso de construcción de la imagen.



```
touch dockerfile
```

Una vez creado lo abrimos y comenzamos a completarlo con las siguientes instrucciones:

En primer lugar, debemos indicar la imagen sobre la cual queremos que se ejecute nuestra aplicación. En este caso utilizaremos la versión 8 de node.

```
FROM node:8
```

Luego creamos un directorio donde almacenar el código dentro de la imagen. En este directorio trabajara nuestra aplicación.

```
WORKDIR /usr/src/app
```
Como la versión de node 8 que estamos instalando ya trae npm, podemos ahora instalar los paquetes necesarios. 

```
COPY package*.json ./
RUN npm install
```
Tenga en cuenta que, en lugar de copiar todo el directorio de trabajo solo estamos copiando el archivo package.json 

Luego para agrupar el código fuente de nuestra aplicación dentro de la imagen de Docker hacemos lo siguiente

```
COPY . .
```
Como en nuestro código establecimos el puerto 8080 para exponer nuestra aplicación, debemos indicarlo en el dockerfile

```
EXPOSE 8080
```

Para finalizar, solo nos queda indicarle el comando para ejecutar nuestra aplicación. 

 ```
 CMD [ "npm", "start" ]  
 ```
 
 ¡Ahora sí, nuestro dockerfile está listo! Debería haberles quedado así

<p align="center">
  <img src="/././images/dockerfile-lab1.png" width="400">
</p>

### Paso 2 - dockerignore

Ahora crearemos un .dockerignore  y agregaremos los módulos locales de node y el log.

El archivo `.dockerignore` tiene un formato como el archivo `.gitignore` y es un listado por renglón de los archivos o carpetas que son ignorados.

```
node_modules
npm-debug.log
```

<p align="center">
  <img src="/././images/dockerignore-lab1.png" width="400">
</p>

### Paso 3 - build

Ya estamos listos para construir nuestra imagen, para lograrlo hay que usar el comando *docker build*. Usaremos -t para darle un nombre a nuestra imagen y que sea más fácil de ubicar. Y pondremos el . al final para destinar a los archivos del directorio en el cual esatmos parados. 

```
$ docker build -t <usuario>/dojo-lab1 .
```

¡Listo! Ahora si consultamos por las imágenes de docker disponibles en nuestro pc podremos encontrar la que acabamos de crear:

```
$ docker images
```

Al ejecutar el comando de arriba recibiremos una lista de todas nuestras imágenes, allí veremos la de este laboratorio. 

<p align="center">
  <img src="/././images/docker-images.png" width="700">
</p>

### Paso 4 - run

Para ejecutar nuestra imagen de docker debemos usar *docker run*.  Usaremos -d para ejecutar el contenedor en segundo plano, lo que nos permitirá seguir utilizando la terminal. También agregaremos -p para redirigir un puerto público a una privado dentro del contenedor. 

```
$  docker run -p 49160:8080 -d <username>/dojo-lab1
```

Para obtener el estado de nuestro despliegue:
```
# Obtener el ID de nuestro contenedor

$ docker ps

# Mostrar en pantalla los logs de nuestro despliegue 

$ docker logs <container id>

# En nuestro caso, nuestro log es el siguiente

Running on http://0.0.0.0:8080

```
Por último si nos dirigimos a http://localhost:49160 podremos acceder a nuestro *Hello world* que hicimos en Node.js. Si no recordamos el puerto de nuestro contenedor, podemos volver a ejecutar *$docker ps* y ver el detalle del mismo. 

<p align="center">
  <img src="/././images/run-lab1.png" width="800">
</p>

## Las imágenes están formadas en capas.

Cada repositorio archiva las capas una única vez, aunque pueda ser utilizada por distintas imágenes. De la misma forma, a la hora de transferir una imagen, solo se copiarán las capas no disponibles en el repositorio destino.

```
docker image history nombre_imagen 
```
Con el comando docker image history, puede ver el comando que se ha usado para crear cada capa dentro de una imagen.

Debería obtener una salida similar a la siguiente (las fechas y los identificadores pueden ser diferentes).

```
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a78a40cbf866        18 seconds ago      /bin/sh -c #(nop)  CMD ["node" "/app/src/ind…   0B                  
f1d1808565d6        19 seconds ago      /bin/sh -c yarn install --production            85.4MB              
a2c054d14948        36 seconds ago      /bin/sh -c #(nop) COPY dir:5dc710ad87c789593…   198kB               
9577ae713121        37 seconds ago      /bin/sh -c #(nop) WORKDIR /app                  0B                  
b95baba1cfdb        13 days ago         /bin/sh -c #(nop)  CMD ["node"]                 0B                  
```
Cada una de las líneas representa una capa en la imagen. En esta pantalla se muestra la base en la parte inferior y la capa más reciente en la parte superior. Con esto, también puede ver rápidamente el tamaño de cada capa, lo que ayuda a diagnosticar imágenes grandes.

## Almacenamiento en caché de capas

Ahora que ha visto las capas en funcionamiento, debe aprender algo importante para reducir los tiempos de compilación de las imágenes de contenedor.

Después de que cambie una capa, también se deben volver a crear todas las capas inferiores

Volver a examinar el Dockerfile que ha usado antes.

<p align="center">
  <img src="/././images/dockerfile-lab1.png" width="400">
</p>

De nuevo en la salida del historial de imágenes, verá que cada comando del Dockerfile se convierte en una nueva capa de la imagen.
Es posible que al momento que se realice un cambio en la imagen, se tuvo que volver a instalar todas las dependencias. ¿Hay alguna manera de solucionar esto? No tiene mucho sentido enviar las mismas dependencias cada vez que se compila, ¿verdad?

Para solucionarlo, nuestro Dockerfile ya viene optimizado. 

En el caso de las aplicaciones basadas en Node, esas dependencias se definen en el archivo package.json. Por tanto, ¿qué ocurre si primero solo copia ese archivo, instala las dependencias y después copia todo lo demás? Es una gran idea ya que cuando haya cambios no tenga que copiar nuevamente todas esas dependencias que se encuentran en el package.json y así ahorrar una gran cantidad de tiempo al momento de subir la imagen.

```
COPY package*.json ./
```
Línea del dockerfile que copia únicamente el package.json. Siendo una de las primeras instrucciones en ser ejecutadas.

> *Give a sysadmin an image and their app will be up-to-date for a day, give a sysadmin a Dockerfile and their app will always be up-to-date*
## Docker Hub

<p align="center">
  <img src="/././images/docker_hub.png" width="400">
</p>

### ¿Qué es? 

Cuando hablamos de imágenes, se mencionó la existencia de repositorios desde donde podemos descargar imágenes públicas o privadas para utilizar en nuestros despliegues. De hecho, en la sección "Hello World" descargamos una imagen desde Docker Hub sin entrar en detalle de que era esto. Entonces, ¿Qué es Docker Hub? Es una plataforma online para alojar nuestras imágenes en repositorios y así poder compartirlas (o no), facilitando su distribución y almacenamiento, muy similar a GitHub.
Podría decirse que Docker Hub es la formas más fácil para administrar aplicaciones basadas en contenedores que existe.
La idea ahora es poder crearnos una cuenta y publicar allí la imagen que creamos más arriba. También navegaremos por la tienda y descargamos algunas imágenes para ejecutarlas en nuestra computadora.  

### Crear una cuenta e iniciar sesion

Lo primero que haremos será dirigirnos a https://hub.docker.com/ y hacer clic sobre "Get Started", allí completaremos nuestros datos para registrarnos en la plataforma. 

<p align="center">
  <img src="/././images/docker-hub-get-started.png" width="400">
</p>

Una vez que hayamos completado el registro y verificado nuestro correo, podremos ingresar en Docker Hub.

<p align="center">
  <img src="/././images/docker-hub-sign-in.png" width="400">
</p>

### Subir nuestra imagen

Ahora comencemos por subir nuestra imagen. Podemos crear nuestro repositorio desde la web o se puede crear a la hora de subir nuestra imagen, nosotros vamos a utilizar esta segunda opción. 

Lo primero es loguearnos en docker desde la consola 

```
$ docker login
```

Luego hacemos un tag de nuestra imagen a nuestro repositorio. 

```
$ docker tag your_image  tu_usuario_docker/nombre_repo

#Por ejemplo, en mi caso:

$ docker tag jpanizza/dojo-lab1 josepanizza/helloworld
```
Ahora solo debemos ejecutar el comando *docker push*

```
$ docker push YOUR_DOCKERHUB_NAME/firstimage

#Por ejemplo, en mi caso:

$ docker push josepanizza/helloworld
```

Con esto ya tenemos publicada nuestra imagen en un repositorio de Docker Hub.  Podemos confirmar que este todo correcto y acceder a la configuración de nuestro repositorio desde la web.

<p align="center">
  <img src="/././images/repolist.png" width="700">
</p>

### Catálogo

Dentro del catálogo de docker hub hay más de 2 millones de imágenes públicas que podemos descargar y correr con un par de comandos. Para acceder al catálogo debemos ir a la ventana *Explorer* del menú superior en la web de Docker Hub.

<p align="center">
  <img src="/././images/docker-hub-catalog.png" width="700">
</p>

Cualquier repositorio público podrá buscarse en esta sección. Es importante que, si no queremos que nuestra imagen aparezca publicada, debemos configurar nuestro repositorio como privado. Dentro de la cuenta que nos acabamos de crear, tenemos 1 repositorio privado gratis. En caso de requerir más debemos adquirir una suscripción premium

Por ejemplo, si buscamos mi nombre de usuario, aparecerá la imagen que acabo de subir. En su caso ocurrirá lo mismo.

<p align="center">
  <img src="/././images/docker-josepanizza.png" width="700">
</p>

### Descargar y ejecutar Mongo DB de Docker Hub

¡¡¡Como ya sabemos utilizar Docker y conocemos lo que es Docker Hub, ya estamos listos para empezar a descargar imágenes y ejecutarlas en localhost!!! (más adelante lo haremos en la nube, paciencia)

Vamos a instalar mongo db, comencemos por buscarlo en el catálogo. 

<p align="center">
  <img src="/././images/mongo-search.png" width="700">
</p>

Podemos ver que la primera opción es mongo. Además podemos apreciar otros datos, como que tiene más de 10 millones de descargas, que es una imagen oficial y que en mi caso, se actualizo por última vez hace media hora. 

<p align="center">
  <img src="/././images/mongo-detail.png" width="700">
</p>

una vez que hacemos clic en mongo podemos ver toda la documentación y a la derecha, el comando que tenemos que ejecutar para descargarlo, 

```
$ docker pull mongo

```
Ahora, en mi caso yo quiero ejecutar la versión 4.2.0-rc1 por lo que voy a correr el siguiente comando indicando cual es el tag (versión) que quiero. 

```
$ docker run --name some-mongo -d mongo:4.2.0-rc1
```

¡Y listo! ya tenemos corriendo mongo en nuestra computadora! Ahora, por ejemplo, si ejecuto *docker ps* puedo ver el estado de mi contenedor con mongo. 

```
$ docker ps
```

<p align="center">
  <img src="/././images/docker-ps-mongo.png" width="800">
</p>

#### ¿Cómo lo utilizo?

Dado que mongo no está instalado en el sistema, la terminal no va a reconocer ningún comando de mongo que quiera ejecutar. Es por esto que lo que hay que hacer es ejecutar:

```
$ docker exec -it some-mongo bash
```
Ahora estamos dentro de nuestro contenedor y podemos ejecutar los comandos de mongo

<p align="center">
  <img src="/././images/mongo-h.png" width="600">
</p>

## Container Registry IBM

<p align="center">
  <img src="/././images/cr-ibm.png" width="75">
</p>


Container Registry es un servicio de IBM Cloud para almacenar y distribuir imágenes de contenedores en un registro privado y completamente administrado por nosotros. Gracias a las herramientas de seguridad de IBM, nuestras imágenes pueden ser revisadas para detectar problemas de seguridad e informarnos de los mismo para que tomemos las precauciones del caso. 
El servicio nos permite almacenar hasta 600mb de forma gratuita, luego el costo depende del peso de nuestras imágenes. En lo que refiere al uso, es muy similar a lo que vimos con Docker Hub, ejemplo:



#### Configuraciones iniciales

Por ser la primera vez que utilizamos Container Registry tenemos que instalar un plugin y crear un namespace. En el caso que de que haya trabajado con este servicio, puede saltarse estos pasos. 

**1**-*IBM Cloud CLI y Docker CLI*

Antes de poder instalar el plugin que nos permitirá trabajar con Container Registry es necesario que tengamos instalado tanto Docker como IBM Cloud CLI (command line interface). Si siguen el dojo desde el comienzo, Docker ya lo deberían de tener instalado. Si nunca antes utilizaron la nube de IBM, pueden acceder a este [link](https://cloud.ibm.com/docs/containers?topic=containers-cs_cli_install) para instalar la CLI

**2**-*Instalar plug-in de Container Registry*

Ahora instalemos el plug-in de container registry, ejecutemos el siguiente comando para lograrlo:

```
$ ibmcloud plugin install container-registry -r Bluemix
```

**3**-*Iniciar Sesión en IBM Cloud*
```
$ ibmcloud login -a https://cloud.ibm.com
```
En caso de que tengamos un usuario federado debemos agregar el  flag *--sso* (single sign on) para loguearnos desde la CLI.

**4**-*Namespace*

Ahora creemos el namespace que utilizaremos para subir nuestras imágenes. 

```
$ ibmcloud cr namespace-add <my_namespace>
```



#### Subir nuestra imagen 

**1**-*Conectar el demon de Docker con IBM CR*

```
$ ibmcloud cr login
```

**2**-*Seleccionar un repositorio y una etiqueta para identificar la imagen*

En nuestro caso podemos utilizar la imagen del lab1 con el hello world que creamos más arriba.
```
$ docker tag jpanizza/lab1-dojo  us.icr.io/<my_namespace>/<my_repository>:<my_tag>
```
 **3**-*Subir la imagen*

Ahora subamos nuestra imagen

```
$ docker push us.icr.io/<my_namespace>/<my_repository>:<my_tag>
```

**4**-*Verificar* 

Para finalizar podemos verificar viendo la lista completa de imágenes.

```
$ ibmcloud cr image-list
```
## Apéndice

En esta sección encontrará material extra que puede utilizar para profundizar en el tema. Además, están los links a los laboratorios de este dojo donde podrá poner en práctica lo visto en esta sección y aprender cosas nuevas. 

### Hands-On 

- [Lab 0 - Lab de la sección](/assets/docker/lab1/#) 
- [Lab 1 - Crear una aplicacion web que se ejecute en Docker](/assets/docker/lab2/#) 

### Material extra

Para mas informacion de como montar una imagen y basicos de docker - https://pilasguru.gitlab.io/Docker-GuiaParaElUsuario/

Para aprender muchos detalles tecnicos - https://www.youtube.com/watch?v=CV_Uf3Dq-EU&t=4s

Para certificarte luego sobre todo lo aprendido en docker - https://cognitiveclass.ai/courses/docker-essentials


[← Volver al índice](/././README.md)
 
 [→ Siguiente Sección(Kubernetes)](/pages/3/#kubernetes)

