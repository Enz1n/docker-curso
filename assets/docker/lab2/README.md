# Modernización de Aplicaciones - Dojo / Lab 2 Docker

<p align="center">
  <img src="/././images/logoprincipal.png" width="500">
</p>



Ahora está listo para llegar a lo real, implementaremos una aplicación web con Docker.

## Ejecutar un sitio web estático en un contenedor

Primero, usaremos Docker para ejecutar un sitio web estático en un contenedor. El sitio web se basa en una imagen existente. Extraemos una imagen de Docker y ejecutaremos el contenedor y veremos lo fácil que es configurar un servidor web.

La imagen que vamos a utilizar es de un sitio web de una sola página que ya se creó para este lab y está disponible en Docker ```dockersamples/static-site```. Puede descargar y ejecutar la imagen directamente de una vez usando ```docker run```.

```
$ docker run -d dockersamples/static-site
```
**IMPORTANTE**

```
Esta imagen no fue creada para este dojo, su versión actual no se ejecuta sin el flag -d. 
Este flag habilita el modo descontado, lo que separa el contenedor en ejecución de la terminal y devuelve su mensaje una vez que el contenedor se inicia.
```

Entonces, ¿qué sucede cuando ejecutamos ese comando?

Como la imagen no existe en su host Docker, el Docker demond primero la descarga de docker hub y luego la ejecuta como un contenedor. Pero esto ya lo tienen claro, ¿No?
 
Ahora que el servidor se está ejecutando, ¿ves el sitio web? ¿En qué puerto se está ejecutando? Y lo que es más importante, ¿cómo se accede al contenedor directamente desde nuestro navegador?

Intente, en base a lo que aprendió en la sección anterior responder a esas preguntas e intentar ingresar en su app. Si no pueden, sigan los siguientes pasos:

Vuelva a ejecutar el comando con algunos flags nuevos para publicar puertos y pasar su nombre al contenedor para personalizar el mensaje que se muestra. Usaremos la opción -d nuevamente para ejecutar el contenedor en "modo aislado".

Primero, detenga el contenedor que acaba de lanzar. Para hacer esto, necesitamos la ID del contenedor.

Cómo ejecutamos el contenedor en modo separado, no tenemos que iniciar otro terminal para hacerlo. Ejecute ```docker ps``` para ver los contenedores en ejecución.

```
jpanizza@jpanizaa:~/Desktop$ docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS               NAMES
6c35b746257b        dockersamples/static-site   "/bin/sh -c 'cd /usr…"   8 seconds ago       Up 7 seconds        80/tcp, 443/tcp     ecstatic_gates

```

Miremos el CONTAINER ID. Deberá usar el valor de CONTAINER ID, una secuencia larga de caracteres, para identificar el contenedor que desea detener y luego eliminarlo. El siguiente ejemplo proporciona el CONTAINER ID en nuestro sistema; debes usar el valor que ves en tu terminal!

```
jpanizza@jpanizaa:~/Desktop$ docker stop 6c35b746257b
6c35b746257b
jpanizza@jpanizaa:~/Desktop$ docker rm 6c35b746257b
6c35b746257b
```

Ahora, iniciemos un contenedor como se muestra a continuación:

```
$ docker run --name static-site -e AUTHOR="Nuestro Nombre" -d -P dockersamples/static-site
```

En mi caso:
```
jpanizza@jpanizaa:~/Desktop$ docker run --name static-site -e AUTHOR="José" -d -P dockersamples/static-site
0f821a1539f8684cb1c40b68451aadeb52dd3f980a12a4bf6cfcdb7a0540578d
```

Analicemos los flags del comando anterior:

-d creará un contenedor en segundo plano
-P publicará todos los puertos del contenedor expuestos en puertos aleatorios en el host Docker
-e es cómo pasar variables de entorno al contenedor
--name le permite especificar un nombre de contenedor
AUTHOR es el nombre de una variable de entorno. 

Ahora puede ver los puertos ejecutando el comando docker port.

```
$ docker port static-site
jpanizza@jpanizaa:~/Desktop$ docker port static-site
80/tcp -> 0.0.0.0:32769
443/tcp -> 0.0.0.0:32768
```

Ahora podemos acceder a nuestra app:

 http://localhost: [Puerto para 80/tcp]. 
 En mi caso: http://localhost:32769.



<p align="center">
  <img src="/././images/lab2docker.png" width="750">
</p>

¡¡Ahora que ya hemos visto cómo ejecutar un servidor web dentro de un contenedor Docker, creemos nuestra propia imagen para ejecutar!!

Pero primero, detengámonos y eliminemos los contenedores, ya que ya no los usará:

```
$ docker stop static-site
$ docker rm static-site
```

## Crear nuestra imagen



En esta sección, profundicemos en las imágenes de Docker. Creará su propia imagen y la usará para ejecutar una aplicación localmente. 

Como ya sabemos, las imágenes de Docker son la base de los contenedores. En el ejemplo anterior, sacó las imágenes de docker hub y le pidió a Docker que ejecutara un contenedor basado en esa imagen. Para ver la lista de imágenes que están disponibles localmente en su sistema, ejecute el comando ```docker images```.

Una distinción importante con respecto a las imágenes es entre *imágenes base* e *imágenes secundarias*.

* Las imágenes base son imágenes que no tienen imágenes principales, generalmente imágenes con un sistema operativo como ubuntu, alpine o debian.

* Las imágenes secundarias son imágenes que se basan en imágenes base y agrega funcionalidad adicional.

Otro concepto clave es la idea de imágenes oficiales e imágenes de usuario (Ambas pueden ser imágenes base o imágenes secundarias).

* Las imágenes oficiales son imágenes autorizadas por Docker. Docker Inc patrocina un equipo que es responsable de revisar y publicar todo el contenido de los repositorios oficiales. Este equipo trabaja en colaboración con quienes se encargan del mantenimiento de software, expertos en seguridad y la comunidad más amplia de Docker. Estas imágenes no tienen el prefijo de una organización o nombre de usuario.

* Las imágenes de usuario son imágenes creadas y compartidas por usuarios. Se basan en imágenes base y agregan funcionalidad adicional. Por lo general, estas están formateadas como *user/image-name*. En donde user es el usuario y name es el nombre de la imagen.

**El objetivo de este ejercicio es crear una imagen Docker que ejecutará una aplicación Flask.**

Haremos esto primero juntando los componentes para un generador de imágenes de gatos al azar construido con Python Flask, luego crearemos su imagen de docker a partir de un Dockerfile. Finalmente, construiremos la imagen y luego la ejecutaremos.

## Crear una aplicación Python Flask 

Para los propósitos de este lab, usaremos una pequeña y divertida aplicación basada en Python Flask que muestra un gato al azar cada vez que se carga, algo sencillo y divertido. 

Comience creando un directorio llamado flask-app donde crearemos los siguientes archivos:

* app.py
* requirements.txt
* templates/index.html
* Dockerfile

**app.py**

Cree el archivo app.py con el siguiente contenido:

```py
from flask import Flask, render_template
import random

app = Flask(__name__)

# list of cat images
images = [
   "http://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr05/15/9/anigif_enhanced-buzz-26388-1381844103-11.gif",
    "http://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr01/15/9/anigif_enhanced-buzz-31540-1381844535-8.gif",
    "http://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr05/15/9/anigif_enhanced-buzz-26390-1381844163-18.gif",
    "http://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr06/15/10/anigif_enhanced-buzz-1376-1381846217-0.gif",
    "http://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr03/15/9/anigif_enhanced-buzz-3391-1381844336-26.gif",
    "http://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr06/15/10/anigif_enhanced-buzz-29111-1381845968-0.gif",
    "http://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr03/15/9/anigif_enhanced-buzz-3409-1381844582-13.gif",
    "http://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr02/15/9/anigif_enhanced-buzz-19667-1381844937-10.gif",
    "http://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr05/15/9/anigif_enhanced-buzz-26358-1381845043-13.gif",
    "http://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr06/15/9/anigif_enhanced-buzz-18774-1381844645-6.gif",
    "http://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr06/15/9/anigif_enhanced-buzz-25158-1381844793-0.gif",
    "http://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr03/15/10/anigif_enhanced-buzz-11980-1381846269-1.gif"
    ]

@app.route('/')
def index():
    url = random.choice(images)
    return render_template('index.html', url=url)

if __name__ == "__main__":
    app.run(host="0.0.0.0")

```

**requirements.txt**

Para instalar los módulos de Python necesarios para nuestra aplicación, necesitamos crear un archivo llamado require.txt y agregar la siguiente línea a ese archivo:

```
Flask==0.10.1
```

**templates / index.html**

Cree un directorio llamado ```templates``` y cree allí un archivo index.html en ese directorio con el siguiente contenido:

```html
<html>
  <head>
    <style type="text/css">
      body {
        background: black;
        color: white;
      }
      div.container {
        max-width: 500px;
        margin: 100px auto;
        border: 20px solid white;
        padding: 10px;
        text-align: center;
      }
      h4 {
        text-transform: uppercase;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <h4>Cat Gif of the day</h4>
      <img src="{{url}}" />
      <p><small>Courtesy: <a href="http://www.buzzfeed.com/copyranter/the-best-cat-gif-post-in-the-history-of-cat-gifs">Buzzfeed</a></small></p>
    </div>
  </body>
</html>
```

## Escribir el Dockerfile

Queremos crear una imagen Docker con esta aplicación web. Como se mencionó anteriormente, todas las imágenes de usuario se basan en una imagen base. Como nuestra aplicación está escrita en Python, crearemos nuestra propia imagen de Python basada en Alpine. Lo haremos usando un Dockerfile.

Repasando un poco lo que vimos en la sección de docker del dojo, un Dockerfile es un archivo de texto que contiene una lista de comandos que el Docker llama mientras crea la imagen. El Dockerfile contiene toda la información que Docker necesita saber para ejecutar la aplicación: 
* Una imagen base de Docker 
* La ubicación del código de su proyecto
* Las dependencias que tiene

Los comandos que debe ejecutar al inicio. Es una forma sencilla de automatizar el proceso de creación de imágenes. La mejor parte es que los comandos que escribe en un Dockerfile son casi idénticos a sus comandos de Linux equivalentes. Esto significa que realmente no tiene que aprender una nueva sintaxis para crear sus propios Dockerfiles.

1 - Cree un archivo llamado Dockerfile y agregue contenido como se describe a continuación.

Comenzaremos especificando nuestra imagen base:

```dockerfile
FROM alpine:3.15
```

2 - El siguiente paso generalmente es escribir los comandos para copiar los archivos e instalar las dependencias. Pero primero instalaremos el paquete Python pip en la distribución alpine linux. Esto no solo instalará el paquete pip, sino también cualquier otra dependencia, que incluye el intérprete de python.

```dockerfile
RUN apk add --update py3-pip
```

3 - Ahora, agreguemos los archivos que componen la aplicación Flask.

Instale todos los requisitos de Python para que se ejecute nuestra aplicación. Esto se logrará agregando las líneas:

```
COPY requirements.txt /usr/src/app/
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt
```
Copie los archivos que ha creado anteriormente en nuestra imagen utilizando el comando COPY .

```
COPY app.py /usr/src/app/
COPY templates/index.html /usr/src/app/templates/
```

4 - Especifique el número de puerto que debe exponerse.

```
EXPOSE 5000
```

5 - El último paso es el comando para ejecutar la aplicación

```
CMD ["python", "/usr/src/app/app.py"]
```

El objetivo principal de CMD es decirle al contenedor qué comando debe ejecutar de forma predeterminada cuando se inicia.

6 - Verifique su Dockerfile

Debería quedarle algo así:

```
FROM alpine:3.15
RUN apk add --update py3-pip
COPY requirements.txt /usr/src/app/
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt
COPY app.py /usr/src/app/
COPY templates/index.html /usr/src/app/templates/
EXPOSE 5000
CMD ["python", "/usr/src/app/app.py"]
```

## Construir la imagen 

Ahora que tenemos nuestro Dockerfile, podemos construir nuestra imagen. El comando ```docker build``` hace el trabajo pesado de crear una imagen acoplable a partir de nuestro Dockerfile.

Cuando ejecute el docker build que veremos a continuación, asegúrese de reemplazar <TU_USUARIO> con su nombre de usuario. Este nombre de usuario debe ser el mismo que creó al registrarse en Docker Hub durante la sección de docker del dojo. 

El comando docker build es bastante simple: toma un nombre de etiqueta opcional con el flag -t, y la ubicación del directorio que contiene el Dockerfile "." indica el directorio actual:

```
$ docker build -t josepanizza/myfirstapp .

Sending build context to Docker daemon 9.728 kB
Step 1 : FROM alpine:latest
 ---> 0d81fc72e790
Step 2 : RUN apk add --update py-pip
 ---> Running in 8abd4091b5f5
fetch http://dl-4.alpinelinux.org/alpine/v3.3/main/x86_64/APKINDEX.tar.gz
fetch http://dl-4.alpinelinux.org/alpine/v3.3/community/x86_64/APKINDEX.tar.gz
(1/12) Installing libbz2 (1.0.6-r4)
(2/12) Installing expat (2.1.0-r2)
(3/12) Installing libffi (3.2.1-r2)
(4/12) Installing gdbm (1.11-r1)
(5/12) Installing ncurses-terminfo-base (6.0-r6)
(6/12) Installing ncurses-terminfo (6.0-r6)
(7/12) Installing ncurses-libs (6.0-r6)
(8/12) Installing readline (6.3.008-r4)
(9/12) Installing sqlite-libs (3.9.2-r0)
(10/12) Installing python (2.7.11-r3)
(11/12) Installing py-setuptools (18.8-r0)
(12/12) Installing py-pip (7.1.2-r0)
Executing busybox-1.24.1-r7.trigger
OK: 59 MiB in 23 packages
 ---> 976a232ac4ad
Removing intermediate container 8abd4091b5f5
Step 3 : COPY requirements.txt /usr/src/app/
 ---> 65b4be05340c
Removing intermediate container 29ef53b58e0f
Step 4 : RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt
 ---> Running in a1f26ded28e7
Collecting Flask==0.10.1 (from -r /usr/src/app/requirements.txt (line 1))
  Downloading Flask-0.10.1.tar.gz (544kB)
Collecting Werkzeug>=0.7 (from Flask==0.10.1->-r /usr/src/app/requirements.txt (line 1))
  Downloading Werkzeug-0.11.4-py2.py3-none-any.whl (305kB)
Collecting Jinja2>=2.4 (from Flask==0.10.1->-r /usr/src/app/requirements.txt (line 1))
  Downloading Jinja2-2.8-py2.py3-none-any.whl (263kB)
Collecting itsdangerous>=0.21 (from Flask==0.10.1->-r /usr/src/app/requirements.txt (line 1))
  Downloading itsdangerous-0.24.tar.gz (46kB)
Collecting MarkupSafe (from Jinja2>=2.4->Flask==0.10.1->-r /usr/src/app/requirements.txt (line 1))
  Downloading MarkupSafe-0.23.tar.gz
Installing collected packages: Werkzeug, MarkupSafe, Jinja2, itsdangerous, Flask
  Running setup.py install for MarkupSafe
  Running setup.py install for itsdangerous
  Running setup.py install for Flask
Successfully installed Flask-0.10.1 Jinja2-2.8 MarkupSafe-0.23 Werkzeug-0.11.4 itsdangerous-0.24
You are using pip version 7.1.2, however version 8.1.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
 ---> 8de73b0730c2
Removing intermediate container a1f26ded28e7
Step 5 : COPY app.py /usr/src/app/
 ---> 6a3436fca83e
Removing intermediate container d51b81a8b698
Step 6 : COPY templates/index.html /usr/src/app/templates/
 ---> 8098386bee99
Removing intermediate container b783d7646f83
Step 7 : EXPOSE 5000
 ---> Running in 31401b7dea40
 ---> 5e9988d87da7
Removing intermediate container 31401b7dea40
Step 8 : CMD python /usr/src/app/app.py
 ---> Running in 78e324d26576
 ---> 2f7357a0805d
Removing intermediate container 78e324d26576
Successfully built 2f7357a0805d
```
¡¡Si todo salió bien, la imagen debería estar lista!! Ejecute ```docker images``` y vea si allí está. 



## Ejecutar nuestra imagen

El siguiente paso en esta sección es ejecutar la imagen y ver si realmente funciona.

```
$ docker run -p 8888:5000 --name myfirstapp josepanizza/myfirstapp
```

¡¡Felicitaciones!! Completo el lab2 de docker exitosamente. 

* [Ir a la seccion docker](/pages/2#docker)
* [Volver al menú principal del dojo](/././README.md)


