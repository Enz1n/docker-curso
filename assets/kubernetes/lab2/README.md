# Modernización de Aplicaciones - Dojo

<p align="center">
  <img src="/././images/logoprincipal.png" width="500">
</p>

## Lab 2 - Kubernetes 



El propósito de este laboratorio es que aprenda cómo actualizar la cantidad de instancias que tiene un despliegue y cómo implementar de manera segura una actualización de su aplicación en Kubernetes.



### Prerrequisitos

Haber completado el lab1 de Kubernetes y tener desplegado el guestbook. 



### Escalado de aplicaciones con réplicas

Una réplica es una copia de un pod que contiene un servicio en ejecución. Al tener múltiples réplicas de un pod, nos podemos asegurar de que nuestro despliegue tenga los recursos disponibles para manejar la carga creciente de nuestra aplicación.

**1- Comando scale**

kubectl proporciona un comando scale para cambiar el tamaño de un deployment existente. Aumentemos nuestra capacidad desde una sola instancia en ejecución de guestbook hasta 10 instancias:

```
$ kubectl scale --replicas=10 deployment guestbook
```
Kubernetes ahora intentará que la realidad coincida con el estado deseado de 10 réplicas al iniciar 9 pods nuevos con la misma configuración que el primero.

**2- Estado del escalado*

Podemos ver el estado de nuestro escalado ejecutando el siguiente comando:

```
$ kubectl rollout status deployment guestbook
```
En caso de que el escalado aún no haya terminado, allí podremos ir viendo el estado. 

**3- Estado de los pods**

Una vez terminado el escalado, si nos preguntamos por nuestros pods veremos que ahora tenemos 9 pods nuevos ejecutándose en nuestro cluster

```
$ kubectl get pods

NAME                        READY   STATUS    RESTARTS   AGE
guestbook-f7cbbccd7-4s6zm   1/1     Running   0          2m5s
guestbook-f7cbbccd7-7ftdm   1/1     Running   0          2m5s
guestbook-f7cbbccd7-f75h5   1/1     Running   0          2m5s
guestbook-f7cbbccd7-g46xd   1/1     Running   0          2m5s
guestbook-f7cbbccd7-ljfv7   1/1     Running   0          2m5s
guestbook-f7cbbccd7-mkrfv   1/1     Running   0          89m
guestbook-f7cbbccd7-nf7kl   1/1     Running   0          2m5s
guestbook-f7cbbccd7-p9l6n   1/1     Running   0          2m5s
guestbook-f7cbbccd7-w9bdx   1/1     Running   0          2m5s
guestbook-f7cbbccd7-wwg29   1/1     Running   0          2m5s

```
### Update y RollBack de nuestra app

Kubernetes le permite hacer una actualización progresiva de su aplicación a una nueva imagen. Esto nos permite actualizar fácilmente la imagen que estamos usando para la ejecución de nuestros pods y también nos permite deshacer fácilmente un deployment si se descubre un problema durante o después del despliegue.

En el laboratorio anterior, usamos una imagen con una etiqueta *v1*. Para nuestra actualización usaremos la imagen con la misma imagen, pero con la etiqueta *v2*.

**1 - Actualizar nuestra imagen**

Usando kubectl, ahora podemos actualizar nuestro despliegue para usar la v2 de la imagen. kubectl permite cambiar detalles sobre los recursos existentes con el comando set.

```

$ kubectl set image deployment/guestbook guestbook=ibmcom/guestbook:v2

```

Tenga en cuenta que un pod puede tener varios contenedores, cada uno con su propio nombre. Cada imagen se puede cambiar individualmente o todas a la vez refiriéndose al nombre. 

**2 - Estado de la actualización**

Para comprobar el estado de nuestra actualización podemos usar el siguiente comando

```
kubectl rollout status deployment/guestbook
```

En mi caso, fui viendo desde el punto en el que ejecute el comando hasta que acabo la actualización, todos los pasos que se fueron dando. Aquí debajo dejo los mismos a modo de ejemplo:

```
Waiting for deployment "guestbook" rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 7 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 7 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 7 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 7 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 7 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "guestbook" rollout to finish: 3 old replicas are pending termination...
Waiting for deployment "guestbook" rollout to finish: 3 old replicas are pending termination...
Waiting for deployment "guestbook" rollout to finish: 3 old replicas are pending termination...
Waiting for deployment "guestbook" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "guestbook" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "guestbook" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "guestbook" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "guestbook" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "guestbook" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "guestbook" rollout to finish: 8 of 10 updated replicas are available...
Waiting for deployment "guestbook" rollout to finish: 9 of 10 updated replicas are available...
deployment "guestbook" successfully rolled out

```

**3 - Acceder a nuestro guestbook**

Ahora podemos acceder a nuestro despliegue y ver los cambios en nuestra app



<p align="center">
  <img src="/./././images/guestbook2.png" width="500">
</p>

Si al cambiar la imagen no se actualiza la página, probar recargando la página borrando el cache. Por ejemplo con Shift + F5

O buscar como hacer un "Hard Refresh" en su navegador

[← Volver al índice](/././README.md)
 
[→ Volver a Sección(Kubernetes)](/pages/3/#kubernetes)
