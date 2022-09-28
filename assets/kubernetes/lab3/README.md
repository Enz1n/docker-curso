# Modernización de Aplicaciones - Dojo

<p align="center">
  <img src="/././images/logoprincipal.png" width="500">
</p>



## Lab 3 - Kubernetes 

En esta práctica de laboratorio, aprenderemos a implementar la misma aplicación de guestbook que implementamos en las prácticas anteriores. Sin embargo, en lugar de usar kubectl, implementaremos la aplicación utilizando archivos de configuración yaml. Este archivo de configuración nos permite tener un control más preciso sobre todos los recursos que se crean dentro de nuestro clúster de Kubernetes.

Antes de trabajar con la aplicación, tenga en cuenta la ubicación de los archivos con los que estaremos trabajando. Para obtenerlos debe descargar este repositorio de github y dirigirse a la siguiente carpeta:

```
$ cd assets/kubernetes/guestbook/v1
```

Si presta atención, dentro de la carpeta *guestbook* hay 2 versiones, en este laboratorio trabajaremos con v1.

## Pasos:

A continuación, realizaremos distintos pasos para escalar y actualizar aplicaciones de forma nativa. Como se mencionó anteriormente, utilizaremos la misma app con la que hemos trabajado en los laboratorios anteriores (guestbook).



### 1 - Escala las aplicaciones de forma nativa.

Como hemos visto Kubernetes puede ejecutar una aplicación a partir de un solo pod, pero cuando necesitamos escalar nuestra app para manejar una gran cantidad de solicitudes, una de las soluciones es utilizar Deployments. Un deployment maneja una colección de pods similares. Cuando solicitamos un número específico de réplicas, el controlador de Kubernetes intentará mantener esa cantidad de réplicas en todo momento. Hasta aquí todos conceptos con los que ya hemos trabajado.

Cada objeto Kubernetes que creemos debe proporcionar dos campos de objeto anidados: el objeto **spec**  y el objeto **status**. El objeto **spec** define el estado deseado y el objeto **status** contiene información proporcionada por el sistema Kubernetes sobre el estado real del recurso. Como se describió anteriormente, Kubernetes intentará reconciliar su estado deseado con el estado real del sistema.

Considere la siguiente configuración de Deployment para la del guestbook:

**guestbook.yaml** *Disponible entre los archivos de la carpeta /guestbook/v1*

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
  labels:
    app: guestbook
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
  template:
    metadata:
      labels:
        app: guestbook
    spec:
      containers:
      - name: guestbook
        image: ibmcom/guestbook:v1
        ports:
        - name: http-server
          containerPort: 3000
```

El archivo de configuración anterior crea un deployment llamado 'guestbook' con un pod que contiene un solo contenedor que ejecuta la imagen ibmcom/guestbook:v1. Además, en la configuración se especifica en 3 el número de réplicas por lo tanto Kubernetes intenta asegurarse de que al menos tres pods activos se estén ejecutando en todo momento.

* Hagamos el despliegue de nuestro guestbook con el guestbook.yaml:

```
$ kubectl create -f guestbook-deployment.yaml
deployment "guestbook" created
```
* Veamos los pods de nuestro despliegue filtrando por la etiqueta "guestbook"
Recordemos que la etiqueta de los pods la definimos en nuestro archivo de Deployment (guestbook.yaml) dentro de la seccion *spec.template.metadata.labels*

```
$ kubectl get pods -l app=guestbook
```
* Editar un nuestro Deployment.yaml

Supongamos que queremos pasar de 3 réplicas a 2, entonces editamos el campo *replicas* en nuestro guestbook.yaml. Una vez hecho esto, debemos actualizar nuestra app, para lograrlo utilizaremos el siguiente comando:

```
$ kubectl apply -f guestbook-deployment.yaml
```

Esto le pedirá a Kubernetes que haga un diff de nuestro archivo yaml con el estado actual del deployment y aplique solo esos cambios.

* Ahora creemos un servicio para acceder a nuestro Deployment

**guestbook-service.yaml**  *Disponible entre los archivos de la carpeta /guestbook/v1*

```
apiVersion: v1
kind: Service
metadata:
  name: guestbook
  labels:
    app: guestbook
spec:
  ports:
  - port: 3000
    targetPort: http-server
  selector:
    app: guestbook
  type: NodePort
```
Esta configuración crea un servicio llamado guestbook. En este caso, estamos configurando una ruta desde el puerto 3000 en el clúster hasta el puerto "http-server" en nuestra aplicación, que es el puerto 3000 según la especificación del deployment.

* Creemos el servicio:

```
$ kubectl create -f guestbook-service.yaml
```
* Probar la app:

Pruebe la aplicación del guestbook ingresando desde el navegador a la url  Cluster-IP : Node-Port 

[Como se menciono en el primer laboratorio](https://github.ibm.com/IBMInnovationLabUy/dojo-modernizacion-aplicaciones-2.0/tree/master/assets/kubernetes/lab1#:~:text=3%20%2D%20Obtener%20el%20puerto%20de%20nuestra%20app) 
  
### 2 - Conectar con un servicio de backend

El código fuente del guestbook está implementado para admitir una variedad de formas para almacenar los datos que registra (los que se ingresan en el libro de visitas). Por defecto mantendrá el registro en las entradas del libro en la memoria.
Ahora bien, necesitamos que todas las instancias de nuestra aplicación compartan el mismo almacén de datos; en este caso, vamos a utilizar una base de datos redis que implementaremos en nuestro cluster. Esta instancia de redis se definirá de manera similar a cuando definimos el Deployment para el guestbook.



**redis-master-deployment.yaml** *Disponible entre los archivos de la carpeta /guestbook/v1*

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      role: master
  template:
    metadata:
      labels:
        app: redis
        role: master
    spec:
      containers:
      - name: redis-master
        image: redis:2.8.23
        ports:
        - name: redis-server
          containerPort: 6379
```

Este yaml creará una base de datos redis en nuestro cluster. Habrá una sola instancia ya que en réplicas indicamos 1. Las instancias de la aplicación del guestbook se conectarán a ella para conservar los datos, así como también realizar una lectura de los mismos. La imagen que se ejecuta en el contenedor es 'redis: 2.8.23' y expone el puerto redis estándar 6379.



* Apliquemos el Deployment de redis al igual que hicimos con el deployment del guestbook:

```
$ kubectl create -f redis-master-deployment.yaml
```

* Comprobemos el estado de nuestros pods usando la etiqueta que le asignamos en el yaml a los pods de redis master:
```
$ kubectl get pods -l app=redis, role=master 
```

* Probemos redis:

```
$ kubectl exec -it <NOMBRE DEL POD> redis-cli
```

Recordemos que al igual que al usar mongo en docker (seccion 2 del dojo), el comando ``` kubectl exec ``` iniciará un proceso secundario en el contenedor especificado. En este caso, estamos pidiendo que el comando "redis-cli" se ejecute en el <NOMBRE DEL POD>. Cuando este proceso finalice, el comando "kubectl exec" también se cerrará, pero los otros procesos en el contenedor no se verán afectados.

Una vez en el contenedor, podemos usar el comando "redis-cli" para asegurarnos de que la base de datos de redis se está ejecutando correctamente, o para configurarlo si es necesario. 

```
redis-cli-IP> ping 
PONG 
redis-cli-IP> exit
```

Ahora debemos exponer el Deployment de redis-master como un Servicio para que la aplicación guestbook pueda conectarse a ella. 

**redis-master-service.yaml** *Disponible entre los archivos de la carpeta /guestbook/v1*

```
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
spec:
  ports:
  - port: 6379
    targetPort: redis-server
  selector:
    app: redis
    role: master
```
Esto crea un objeto de servicio llamado 'redis-master' y lo configura para apuntar al puerto 6379 en todos los pods seleccionados por las etiquetas "app=redis" y "role=master"

* Creemos el servicio para acceder a redis-master

```
$ kubectl create -f redis-master-service.yaml

```
* Reiniciemos nuestra app para que ahora se conecte con la base de datos de redis

```
$ kubectl delete deploy guestbook-v1 
$ kubectl create -f guestbook-deployment.yaml

```

Ahora podemos ver que, si abrimos varios navegadores y actualizamos la página para acceder a las diferentes copias del libro de visitas, todos tienen el mismo registro. Todas las instancias escriben en el mismo almacenamiento y todas las instancias se leen desde ese almacenamiento. 

En este momento, todo el tráfico del servidor de base de datos está en un solo pod master que se encarga de la lectura y escritura de los datos. Una buena práctica para la escalabilidad y eficiencia es separar las lecturas y escrituras para que vayan a diferentes bases de datos que se replican correctamente para lograr la consistencia de los datos.

Para esto, creemos un deployment llamado 'redis-slave' que pueda comunicarse con la base de datos redis y así administrar las lecturas de datos. El deployment de 'redis-slave' está configurado para ejecutar dos réplicas (como vemos a continuación).

<p align="center">
  <img src="/./././images/representacion-lab3-kubernetes.png" width="500">
</p>


**redis-slave-deployment.yaml** *Disponible entre los archivos de la carpeta /guestbook/v1*

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  labels:
    app: redis
    role: slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
      role: slave
  template:
    metadata:
      labels:
        app: redis
        role: slave
    spec:
      containers:
      - name: redis-slave
        image: kubernetes/redis-slave:v2
        ports:
        - name: redis-server
          containerPort: 6379
```

* Crear los pods ejecutando el deployment redis-slave

```
$ kubectl create -f redis-slave-deployment.yaml
```

* Comprobemos si los pods están ejecutados correctamente

```
$ kubectl get pods -l app = redis, role = slave
```

* Accedemos a uno de los pods para chequear que podamos leer la información que se haya cargado en redis

```
$ kubectl exec -it <NOMBRE-POD-REDIS-SLAVE> redis-cli 
127.0.0.1:6379> keys *
1) "guestbook"
127.0.0.1:6379> lrange guestbook 0 10
1) "hola"
127.0.0.1:6379> lrange guestbook 0 10
1) "hola"
2) "chau"
127.0.0.1:6379> exit
```
Ahora que tenemos nuestros pods de redis-slave podemos crear el servicio que los exponga, usaremos el siguiente archivo:

**redis-slave-service.yaml** *Disponible entre los archivos de la carpeta /guestbook/v1*

```
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    app: redis
    role: slave
spec:
  ports:
  - port: 6379
    targetPort: redis-server
  selector:
    app: redis
    role: slave
```

* Creemos el servicio para acceder a redis-slave

```
$ kubectl create -f redis-slave-service.yaml

```
* Reiniciemos nuestra app para que ahora se conecte con la base de datos de redis

```
$ kubectl delete deploy guestbook-v1 
$ kubectl create -f guestbook-deployment.yaml
```

Hasta aquí llegaremos con este laboratorio. Con los siguientes comandos pueden eliminar nuestra app y la base de datos que hemos desplegado. 

```
$ kubectl delete -f guestbook-deployment.yaml
$ kubectl delete -f guestbook-service.yaml
$ kubectl delete -f redis-slave-service.yaml
$ kubectl delete -f redis-slave-deployment.yaml 
$ kubectl delete -f redis-master-service.yaml 
$ kubectl delete -f redis-master-deployment.yaml
```

[← Volver al índice](/././README.md)
 
[→ Volver a Sección(Kubernetes)](/pages/3/#kubernetes)
