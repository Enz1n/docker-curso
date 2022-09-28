# Modernización de Aplicaciones - Dojo

<p align="center">
  <img src="/././images/logoprincipal.png" width="500">
</p>

## Lab 1 - Kubernetes 

El propósito de este laboratorio es que aprenda cómo desplegar una aplicación en un clúster Kubernetes en IBM Cloud. En este laboratorio repasamos algunas cosas que ya hemos hecho durante el dojo cuando desplegamos nginx, van a ser de utilidad para que pueda terminar de fortalecer conocimientos y comienza a agarrarle práctica. 

### Prerrequisitos

Si viene siguiendo el dojo, y ya completo toda la sección de Kubernetes entonces seguramente cumpla con los siguientes prerrequisitos:

* Tener instalado la CLI para IBM Cloud
* Contar con un cluster de Kubernetes en IBM Cloud
* Haber configurado kubectl para trabajar con su cluster

### Pasos 

En este laboratorio implementaremos una aplicación llamada guestbook que ya se ha creado y subido en DockerHub con el nombre guestbook a la cuenta ibmcom (IBM Community).

**1 - Correr la imagen en Kubernetes**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
  labels:
     app: guestbook
spec:
  replicas: 1 
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
        image: ibmcom/guestbook:v2
        ports:
        - containerPort: 80
```
En un archivo de texto pegamos el codigo de nuestro deployment. Y en una terminal que este apuntando a el directorio donde se encuentra nuestro archivo, aplicamos el siguiente comando.  
```
kubectl apply -f guestbook.yaml . 
```


Aplicando este archivo yaml, con el comando que respectivo lo que estamos haciendo es indicando a Kubernetes que debe correr la imagen de Dockerhub *ibmcom/guestbook:v1*
Para asegurarnos que todo haya salido bien, lo que podemos hacer es preguntarnos por el estado de nuestros pods. Como hemos visto anteriormente, para lograr esto ejecutamos:

```
$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
guestbook-f7cbbccd7-mkrfv   1/1     Running   0          8m51s
```
Si el estado de nuestro pod no es "running" esperamos unos segundos más.

Con esto no solamente tenemos un pod corriendo, sino que, si se fijan en los deployments, verán que también se creó uno con el fin de administrar a nuestro pod. 

Recordemos que para ver nuestros deployments debemos ejecutar:
```
$kubectl get deployments

NAME        READY   UP-TO-DATE   AVAILABLE   AGE
guestbook   1/1     1            1           19m

```

**2 - Exponer el Deployment**

Una vez que el estado de nuestro pod sea "running" podemos pasar a la etapa de exponer nuestro despliegue para acceder a él. Si recuerdan, para hacer esto debemos exponer como un servicio nuestro deployment. 
```
$ kubectl expose deployment guestbook --type "NodePort" --port=3000 
```
Si la sintaxis es correcta, debemos obtener una respuesta como la siguiente:
```
service "guestbook" exposed
```

**3 - Obtener el puerto de nuestra app**

Una vez expuesto nuestro nuestro deployment como un servicio, debemos obtener el puerto de este para poder acceder. 

```
kubectl get service guestbook
NAME        TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
guestbook   NodePort   172.21.236.111   <none>        3000:31256/TCP   59m
```
Podemos ver la asignación de puertos, 3000 dentro del pod expuesto al clúster en el puerto 31208. Este puerto en el rango 31256 se elige automáticamente y podría ser diferente en su caso.

**4 - Obtener la IP del cluster**

Ahora lo único que necesitamos para poder acceder es la IP externa de nuestro nodo. Hay varias formas de obtenerla, una de ellas es obtener toda la descripción del mismo.

```
$ kubectl describe node 10.76.193.217
```
*10.76.193.217* es el nombre de mi nodo, recuerden que para obtener la lista de los nodos pueden ejecutar ```kubectl get nodes```

Dentro del output de este comando estará la IP Externa, en mi caso:

```
Addresses:
  InternalIP:  10.76.193.217
  ExternalIP:  50.23.39.117
  Hostname:    10.76.193.217
```
Donde la IP externa es 50.23.39.117

**5 - Acceder a nuestro guestbook**

Por último, como ya tenemos la dirección IP externa del nodo y además tenemos el puerto donde está corriendo el servicio que expone nuestro deployment, ¡podemos acceder a nuestra app! 

```
<ExternalIP>:<ServicePort>
ej: 50.23.39.117:31256
```
El resultado es el siguiente:

<p align="center">
  <img src="/./././images/guestbook.png" width="400">
</p>

Para terminar, podemos borrar nuestro despliegue y el servicio con los siguientes comandos. De todas formas, si van a hacer el siguiente lab NO LOS BORREN. 

* Para eliminar el despliegue, use ```$ kubectl delete deployment guestbook.```

* Para eliminar el servicio, utilice ```$ kubectl delete service guestbook.```


[← Volver al índice](/././README.md)
 
[→ Volver a Sección(Kubernetes)](/pages/3/#kubernetes)






