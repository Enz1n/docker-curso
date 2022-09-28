# Modernización de Aplicaciones 2.0 - Dojo / Helm

<p align="center">
  <img src="/././images/logoprincipal.png" width="500">
</p>

* [← Volver al índice](/././README.md) 

## Helm 

En esta sección vamos a trabajar con Helm, el gestor de paquetes para Kubernetes, es la mejor manera de encontrar, compartir y usar software creado para Kubernetes.

<p align="center">
  <img src="/././images/helm.png" width="200">
</p>

## Introducción


Helm es un administrador de paquetes para Kubernetes. Los administradores de paquetes automatizan el proceso de instalación, configuración, actualización y eliminación de aplicaciones. Por ejemplos algunos administradores de paquetes son Red Hat Package Manager (RPM), Homebrew y Windows PackageManagement.

Una aplicación en Kubernetes generalmente consta de al menos dos tipos de recursos: un recurso de despliegue, que describe un conjunto de pods que se implementarán juntos, y un recurso de servicio, que define gateways para acceder a las API en esos pods. Además de un despliegue y un servicio, una aplicación generalmente incluirá otros tipos de recursos de Kubernetes como ConfigMaps, Secrets e Ingress.

Para cualquier aplicación en Kubernetes, deberemos de ejecutar varios comandos *kubectl* para crear y configurar nuestros recursos. Con Helm, en lugar de crear manualmente cada recurso por separado, puede crear muchos recursos con un solo comando  *helm install*. Esto simplifica enormemente el proceso como una sola unidad llamada tabla Helm.

Otra manera de definir Helm es la siguiente :

## ¿Qué es Helm?

Primero tenemos tres grandes conceptos que hay que saber.

Un **Chart** es un paquete de Helm. Contiene todas las definiciones de recursos necesarias para ejecutar una aplicación, herramienta o servicio dentro de un clúster de Kubernetes.

Un **Repositorio** es el lugar donde se pueden recopilar y compartir Charts.

Un **Release** es una instancia de un Chart que se ejecuta en un clúster de Kubernetes. A menudo, un Chart se puede instalar muchas veces en el mismo clúster. Y cada vez que se instala, se crea un nuevo release. 

Una vez que tengamos bien claros estos conceptos entonces podremos definir de la siguiente manera lo que es la herramienta de Helm.

Helm instala **Charts** en Kubernetes, creando nuevos **Release** para cada instalacion. Y para encontrar nuevos charts, puedes buscar en los **Repositorios** de charts de Helm.


## Instalar Helm 

Para los usuarios de linux es utilizar un simple comando : 
```
https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
Para los usarios de Windows sigan los siguientes pasos:

* Entramos al siguiente [repositorio]() y descargamos la ultima versión de Helm 
* Extraemos los archivos del zip y copiamos el archivo **Helm.exe**
* Luego buscamos en Archivos de Programa el directorio IBM seguimos el patron 

  C:\Program Files\IBM\Cloud\bin

  Y pegamos el archivo.
  
En el caso de de no tener la Cli de IBM instalada sigan el siguiente paso a paso --> https://cloud.ibm.com/docs/cli?topic=cli-install-ibmcloud-cli
  
---


[Otras maneras de instalar Helm para distintos sistemas operativos los pueden encontrar aqui](https://helm.sh/docs/intro/install/
)


## Iniciar Helm en el Cluster

Lo primero que haremos será Iniciar Helm en nuestro Clúster. 

Asegurémonos de estar conectados a nuestro clúster de kubernetes, recuerden que para chequear esto pueden ejecutar ```kubectl get nodes```·

```
$ kubectl apply -f https://raw.githubusercontent.com/IBM-Cloud/kube-samples/master/rbac/serviceaccount-tiller.yaml

```

## Instalar un Helm Chart de ejemplo

Para instalar un chart, puede ejecutar el comando *helm install*. Helm tiene varias formas de encontrar e instalar un chart, pero la más fácil es usar uno de los charts oficiales.
En nuestro caso no utilizaremos un chart oficial, así que debemos agregar el repo desde donde descargamos nuestro chart. 

```
$  helm repo add pl-helm-charts https://pranay-lonkar.github.io/helm-charts

"pl-helm-charts" has been added to your repositories
```
¡ Se preguntaran de donde obtuvimos el repo !

Bueno desde los documentos oficiales de helm se recomienda utilizar la plataforma de [ARTIFACTHUB](https://artifacthub.io/) 

<p align="center">
  <img src="/././images/artifacthub.png" width="750">
</p>
Bien ¿Qué es ArtifactHub?

ArtifactHub es el nuevo Helm Hub

Artifact Hub es una aplicación (open source) que permite buscar, instalar y publicar paquetes de Helm y configuraciones para proyectos Cloud Native. 
De una manera muy sencilla y facil de aplicar.

Previo a instalar el chart tendremos que actualizar el repo para que este todo funcional con el siguiente comando 

```
$ helm repo update


Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "pl-helm-charts" chart repository
Update Complete. ⎈Happy Helming!⎈
```
Ahora instalemos una app de ejemplo en Nodejs
```
$ helm install my-sample-node-app pl-helm-charts/sample-node-app --version 1.0.2


NAME: my-sample-node-app
LAST DEPLOYED: Tue Dec 21 12:50:18 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

```

Cada vez que instala un chart, se crea una nueva versión. Por lo tanto, se puede instalar un mismo chart varias veces en el mismo clúster, en donde cada uno se puede gestionar y actualizar de forma independiente.

Ahora podemos ejecutar el siguiente comando para consultar los Helm Chart instalados en nuestro Clúster. 

```
$ helm list
```

## Estado de nuestro cluster

En el paso anterior instalamos un helm chart en nuestro clúster, veamos que recursos se han creado. 

Consultemos por los deployments:

```
$ kubectl get deployments

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
sample-node-app        1/1     1            1           6m58s
```

Consultemos por nuestros pods:

```
$ kubectl get pods
NAME                                                     READY   STATUS    RESTARTS   AGE
sample-node-app-5457d6f559-5ch2l        1/1     Running            0          7m58s
```
Ahora consultemos por los servicios:

```
$ kubectl get services

NAME                                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
node-svc                    NodePort       172.21.136.42    <none>                                 80:30081/TCP   8m17s

```

Como podemos ver, tenemos un despliegue con un solo pod y el mismo está expuesto como NodePort en el puerto 30081. ¡¡Todo esto con un solo comando!! *Helm install* fue lo único que ejecutamos y creo este despliegue en nuestro clúster. Ingresemos ahora a nuestra app de ejemplo. Recuerde que como el mismo esta expuesto como NodePort, la forma de acceder es  CLUSTER-IP : PORT 


[Como se menciono en el primer laboratorio de Kubernetes](https://github.ibm.com/IBMInnovationLabUy/dojo-modernizacion-aplicaciones-2.0/tree/master/assets/kubernetes/lab1#:~:text=3%20%2D%20Obtener%20el%20puerto%20de%20nuestra%20app) 

<p align="center">
  <img src="/././images/node.png" width="750">
</p>

## Estado del Helm Chart

Como vimos anteriormente, es un poco molesto tener que chequear el estado “a mano” del despliegue en nuestro cluster, ahora que teníamos solo una app desplegada a partir de un helm chart no hay problema, pero imaginen si tuviéramos decenas, ¡complicado! 
Desde Helm podemos ver todo el estado de nuestro despliegue utilizando *helm status* y el nombre del despliegue de nuestro chart.

```
$ helm status  my-sample-node-app
NAME: my-sample-node-app
LAST DEPLOYED: Mon Nov 22 16:47:31 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

```

## Manifiesto 

Con el uso del siguiente comando vamos a poder ver que acciones realiza el Helm chart para poder desplegar nuestra aplicacion.

```
helm get manifest my-sample-node-app
---
# Source: sample-node-app/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: node-svc
spec:
  selector:
    app: sample-node-app
  type: NodePort
  ports:
    - name: tcp
      protocol: TCP
      port: 80
      targetPort: 3000
      nodePort: 30081
---
# Source: sample-node-app/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-node-app
  labels:
    chart: "sample-node-app"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample-node-app
  template:
    metadata:
      labels:
        app: sample-node-app
    spec:
      containers:
        - name: sample-node-app
          image: "pranaylonkar19/sample-node-app:latest"
          imagePullPolicy: Always
```
Como podemos apreciar en este caso son dos simples archivos Yamls los cuales invocan un service y un deployment.
## Apéndice

En esta sección encontrará material extra que puede utilizar para profundizar en el tema. Además, está el link a el lab de esta sección donde podrá poner en práctica lo visto en esta sección y aprender cosas nuevas. 

### Hands-On 

- [Lab 1 - Crear y Ejecutar nuestro propio Helm Chart](https://github.ibm.com/IBMInnovationLabUy/dojo-modernizacion-aplicaciones-2.0/blob/master/assets/helm/lab1/REDME.MD#:~:text=Modernizaci%C3%B3n%20de%20Aplicaciones%20%2D%20Dojo%20/%20Lab%201%20%2D%20Helm)

### Material extra

* [Documentación Oficial de Helm](https://helm.sh/docs)
* [Catalogo de Helm Charts en IBM Cloud](https://cloud.ibm.com/kubernetes/helm)
* [ArtifactHub](https://artifacthub.io/)
* [Link al repo de sample-node-app](https://artifacthub.io/packages/helm/pl-helm-charts/sample-node-app)
* [Expliaciones sencillas Helm parte 1](https://www.youtube.com/watch?v=CPjfb-I_BKU&t=276s)
* [Expliaciones sencillas Helm parte 2](https://www.youtube.com/watch?v=jScW2XaS8uI&t=296s)



[← Volver al índice](/././README.md)
 
 [→ Siguiente Sección(MicroServicios)](/pages/4#microservicios)