# Modernización de Aplicaciones 2.0 - Dojo / página 3

<p align="center">
  <img src="/././images/logoprincipal.png" width="500">
</p>

* [← Volver al índice](/././README.md)

## Kubernetes

En la [sección anterior](/./pages/2/README.md) estuvimos trabajando con contenedores usando docker. Ahora vamos a trabajar con Kubernetes, un orquestador de contenedores con el que cual desplegaremos nuestras aplicaciones en la nube.

<p align="center">
  <img src="/././images/Kubernetes-logo.png" width="300">
</p>

**Indice de la sección:**

- [Modernización de Aplicaciones 2.0 - Dojo / página 3](#modernización-de-aplicaciones-20---dojo--página-3)
  - [Kubernetes](#kubernetes)
    - [¿Que es Kubernetes?](#que-es-kubernetes)
    - [IBM Cloud Kubernetes Service](#ibm-cloud-kubernetes-service)
    - [Arquitectura de Kubernetes](#arquitectura-de-kubernetes)
      - [Nodo Maestro](#nodo-maestro)
      - [Nodo Trabajador](#nodo-trabajador)
      - [Replicas en Kubernetes](#replicas-en-kubernetes)
      - [Replica Sets](#replica-sets)
      - [Deployments](#deployments)
    - [Conectar terminal a nuestro cluster de Kuberentes](#conectar-terminal-a-nuestro-cluster-de-kuberentes)
    - [Crear un Deployment para desplegar nginx en Kubernetes](#crear-un-deployment-para-desplegar-nginx-en-kubernetes)
    - [Dashboard Kubernetes](#dashboard-kubernetes)
    - [¿Cómo se accede a las aplicaciones?](#cómo-se-accede-a-las-aplicaciones)
      - [Servicios](#servicios)
      - [Exponer nuestro despliegue de nginx](#exponer-nuestro-despliegue-de-nginx)
    - [Almacenamiento en Kubernetes](#almacenamiento-en-kubernetes)
  - [Apendice](#apendice)
    - [Hands-On](#hands-on)
    - [Material extra](#material-extra)

### ¿Que es Kubernetes?

Ahora que sabemos qué son los contenedores, definamos qué es Kubernetes. Kubernetes es un orquestador de contenedores para aprovisionar, administrar y escalar aplicaciones. En otras palabras, Kubernetes nos permite administrar el ciclo de vida de las aplicaciones en contenedores dentro de un grupo de nodos (ya hablaremos de ellos), algunos de los servicios que ofrece son:

* Administrar del clúster
* Gestionar todo el ciclo de vida de los contenedores (ej. reiniciar contenedores que fallen)
* Service Discovery (que un contenedor pueda encontrar las rutas IP/DNS de otro contenedor)
* Servicios de red y load-balancing (repartir la carga entre las distintas máquinas del clúster)
* Servicios de monitorización
* Servicios de chequeo de estado de salud
* Auto escalado

Se puede hacer referencia a Kubernetes de distintas formas. A veces se reduce a k8s (perdiendo las 8 letras internas), o simplemente kube. La palabra proviene del griego antiguo y significa "timonel". Un timonel es la persona que dirige un barco. Esperamos que pueda ver la analogía entre dirigir un barco y las decisiones tomadas para organizar contenedores en un grupo.

### IBM Cloud Kubernetes Service

Antes de comenzar a trabajar con kubernetes, es importante que tengamos creado y configurado nuestro clúster para hacer allí los despliegues.

Lo primero que debemos hacer es crear el servicio en IBM Cloud, para ello accedemos a la sección de kubernetes. Lo que debemos hacer es desplegar el menú lateral izquierdo que se encuentra en el dashboard de IBM Cloud (haciendo clic en la 'hamburguesa', botón en el margen superior izquierdo) y luego hacer clic sobre 'Kubernetes'.

<p align="center">
  <img src="/././images/interfaz.png" width="800">
</p>

Una vez alli, si no tenemos ningún cluster creado, veremos una pantalla como la imagen de aquí debajo. Hagamos clic en 'Create a Cluster'

<p align="center">
  <img src="/././images/crearclusternuevo.png" width="800">
</p>

Aquí hay dos opciones, crear un clúster bajo el plan Lite (Free) o crearlo bajo un plan Standard (pago). Las opciones para crear el clúster en el plan Standard son mucho más variadas que en el plan free, para el propósito de este dojo utilizaremos el plan Lite. 
En nuestro caso, solo debemos indicar el nombre de nuestro clúster. Asegurese de estar en 'Dallas'

<p align="center">
  <img src="/././images/clustercreate.png" width="800">
</p>

<p align="center">
  <img src="/././images/createcluster2.png" width="800">
</p>

Ahora solo hace falta esperar entre 10 y 15 minutos a que nuestro cluster esté listo para usar. Mientras esto ocurre podemos seguir con el dojo. 

El cluster que estamos creando, al usar la versión gratuita, tiene algunas limitaciones. Lo más importante a tener en cuenta es que a los 30 días se borra. Más adelante veremos otras limitaciones, por ejemplo, solo se puede usar NodePort a la hora de exponer un servicio, deployment o pod. Tampoco podremos hacer uso de los Persistent volume Storage. 

### Arquitectura de Kubernetes

Comencemos a ver un poco de la arquitectura de kubernetes, primero arranquemos con algunas definiciones:

**Nodo Master:** controla los nodos trabajadores de Kubernetes. Aquí es donde se originan todas las asignaciones de tareas.

**Nodo trabajador:** se encargan de realizar las tareas requeridas y asignadas. 

**Pod:** un grupo de uno o más contenedores implementados en un nodo único. Todos los contenedores de un pod comparten la dirección IP, la IPC, el nombre del host y otros recursos. Los pods abstraen la red y el almacenamiento del contenedor subyacente. Esto le permite mover los contenedores por el clúster con mayor facilidad.

**Controlador de replicación**(replica set): controla la cantidad de copias idénticas de un pod que deben ejecutarse en algún lugar del clúster.

**Servicio:** separa las definiciones de tareas de los pods. Los proxys de servicios de Kubernetes envían automáticamente las solicitudes de servicio al pod correspondiente, sin importar donde se traslada en el clúster, o incluso si está siendo reemplazado.

**Kubelet:** este servicio se ejecuta en los nodos y lee los manifiestos del contenedor, y garantiza que los contenedores definidos estén iniciados y ejecutándose.

**kubectl:** es la herramienta de configuración de la línea de comandos de Kubernetes.

<p align="center">
  <img src="/././images/kubernetes-arq.png" width="600">
</p>

Con la imagen de arriba ya podemos empezar a hacernos una idea de cómo es la arquitectura de un despliegue en kubernetes, ahora entremos un poco mas en profundidad en algunas de las definiciones de arriba.

#### Nodo Maestro

Como vimos más arriba, los servicios de control en un clúster de Kubernetes se encuentran en el nodo maestro. Estos componentes funcionan como los principales puntos donde administrar y también brindar servicios de administración de sistemas a nivel de clúster para los nodos de trabajo. Estos servicios de control pueden instalarse en una sola máquina o distribuirse en varias máquinas. 

Los servidores que ejecutan nodos maestros tienen una serie de servicios únicos que se utilizan para administrar la carga de trabajo del clúster y las comunicaciones directas a través del sistema, entre los cuales podemos encontrar:

* **etcd:** almacena datos de configuración que pueden ser utilizados por cada uno de los nodos en el clúster.
* **Servidor API:** permite la configuración de cargas de trabajo y unidades organizativas. Expone la API para la gestión de Kubernetes. Es utilizado por kubectrl CLI.
* **Demon del controlador:**  ejecuta los controladores, que son los subprocesos en segundo plano que manejan las tareas de rutina en el clúster.
* **Controlador de nodo:**  es responsable de responder cuando los nodos fallan
* **Controlador de endpoints:** arma los endpoint uniendo servicios y pods
* **Cuenta de servicio y controladores de token:** crean cuentas predeterminadas y tokens de acceso a la API para nuevos espacios de nombres
* **Servicio del programador:** asigna cargas de trabajo a nodos específicos en el clúster

#### Nodo Trabajador

Los nodos de trabajo se encargan de realizar las tareas requeridas. Se comunican con los nodos maestros, configuran la red para contenedores y ejecutan las cargas de trabajo asignadas. Docker se ejecuta en una subred

* **Servicio de Kubelet:** este servicio se comunica con los nodos maestros para recibir comandos y trabajar.
* **Servicio de proxy:** este proceso envía las solicitudes a los contenedores correctos, los balanceos de carga y se asegura de que el entorno de red aislado sea accesible.

#### Replicas en Kubernetes

El uso de réplicas en nuestro desarrollo de kubernetes tiene como principal objetivo cumplir con las siguientes características:

* **Confiabilidad:** por tener varias versiones de una aplicación, previene problemas si uno o más fallan. 
* **Equilibrio de carga:** tener varias instancias de un contenedor le permite enviar fácilmente tráfico a diferentes instancias para evitar la sobrecarga de una sola. 
* **Escalado:** cuando la carga se vuelve demasiado para la cantidad de instancias existentes, Kubernetes le permite escalar fácilmente su aplicación, agregando instancias adicionales según sea necesario.

En kubernetes existen 3 tipos, Replication Controller, Replica Sets, y Deployments. A continuación, analizaremos cada uno de ellos. 

#### Replica Sets 

Replica Sets es una forma de hacer una replicación de pods en Kubernetes. En otras palabras, es una estructura que nos permite crear fácilmente múltiples pods y luego asegúrese de que ese número de pods siempre exista. Si un pod sufre de algún problema, él Replica Sets lo reemplaza. También brinda otros beneficios, como la capacidad de escalar la cantidad de pods y de actualizar o eliminar múltiples pods con un solo comando.
Durante la implementación de un ReplicaSet, se puede asignar un campo llamado .spec.selector, que se utiliza para determinar qué pods administrar. Para cada pod cuya etiqueta coincida con este spec.selector de un determinado ReplicaSet, cae bajo la jurisdicción de este ReplicaSet. Esto significa que un Replica Sets puede administrar pods que no creó explícitamente.
Como resultado, en la práctica, los Replica Sets rara vez se usan de forma independiente para administrar pods. En su lugar, la opción más popular es usar "capas de encapsulación" que hagan uso de Replica Sets en forma de objetos de la API. Un ejemplo de esto último es usando Deployments.
 
#### Deployments 

Originalmente, cuando necesitábamos escalar nuestros Pods en kubernetes, usábamos los controladores de réplica. Una de las características clave de los controladores de réplica es que incluyen la funcionalidad de "actualización continua". Esto permite que los pods que son administrados por ese controlador de réplica se actualice con una interrupción mínima del servicio que los pods están brindando. Durante este proceso de actualización lo que ocurre es que las instancias de los pods antiguas se actualizan una por una. 
Sin embargo, los controladores de réplica fueron criticados por ser imperativos y por carecer de flexibilidad. Como solución, se introdujeron los Replica Sets y los Deployments.  Si bien los Replica Sets aún tienen la capacidad de administrar y escalar instancias de ciertos pods, no tienen la capacidad de realizar una actualización sucesiva. En su lugar, esta funcionalidad es administrada por un Deployment.

Los deployments encapsulan conjuntos de réplicas y pods en la jerarquía de recursos de Kubernetes y proporcionan un método declarativo para actualizar el estado de ambos. Un método para acceder a esta interfaz declarativa es a través de kubectl. 

### Conectar terminal a nuestro cluster de Kuberentes 

Para poder acceder a nuestro cluster de Kubernetes desde la linea de comandos, debemos obtener el link acceso desde el dashboard de IBM.

<p align="center">
  <img src="/././images/kubernetesdash.png" width="600">
</p>
 Para acceder a la misma lo hacemos entrando a el cluster que recien creamos.
<p align="center">
  <img src="/././images/clisash.png" width="600">
</p>
Siguiendo el instructivo que muestra como conectarnos deberiamos acceder de forma exitosa a nuestro cluster. 
<p align="center">
  <img src="/././images/cliconnect.png" width="600">
</p>

```
PS C:\Projects\dojo-modernizacion\kubernetes> ibmcloud ks cluster config --cluster c6tom2of05fu8r1a3rm0
OK
The configuration for c6tom2of05fu8r1a3rm0 was downloaded successfully.
Added context for c6tom2of05fu8r1a3rm0 to the current kubeconfig file.
You can now execute 'kubectl' commands against your cluster. For mmands might fail for a few seconds while RBAC synchronizes. (editado) 

```
Si logramos tener una salida parecida a la anterior quiere decir que ¡Estamos conectados a nuestro cluster de Kubernetes!


### Crear un Deployment para desplegar nginx en Kubernetes

Hasta aquí hemos visto bastante de teórico, es hora de comenzar con algo práctico. 
A continuación, veremos un ejemplo que se proporciona en la documentación de Kubernetes y que nos permitirá ver las funcionalidades básicas de los Deployments. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
     app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

En este ejemplo:
* Se crea un deployment llamado nginx-deployment, esto se indica en el campo metadata.name.
* El deployment crea tres Pods replicados, indicado en el campo .replicas.
* El campo selector define cómo el deployment encuentra los Pods a gestionar. En este caso, simplemente selecciona una etiqueta que se define en la plantilla Pod ( app: nginx). 
* El campo template contiene sub-campos, en ellos se indica:
* Los pods se etiquetan usando el campo labels.
* La especificación de la plantilla Pod, o el campo .template.spec, indica que los Pods ejecutan un contenedor nginx, que ejecuta la imagen de nginx de Docker Hub en la versión 1.7.9.

Hagamos el despliegue en kubernetes de nginx usando el deployment que acabamos de ver. Lo primero es copiar la configuración de arriba en un archivo con extensión yaml. En mi caso lo llamare deployment.yaml

Una vez tengamos pronto el archivo, abramos nuestra terminal y ejecutemos el siguiente comando:

```
$ kubectl apply -f deployment.yaml
```
Este comando efectúa un despliegue en base a nuestro deployment.yaml.
Una vez terminado podemos ejecutar el siguiente comando para ver nuestros deployments

```
$ kubectl get deployments
```
Si todo salio bien veremos en pantalla algo como esto:

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           03m
```
¡Nuestro despliegue se realizó con éxito!

Ahora veamos los pods que hemos creado

```sh
$ kubectl get pods
NAME                                  READY   STATUS    RESTARTS   AGE
nginx-deployment-75bd58f5c7-5h9lr   1/1     Running   0          03m
nginx-deployment-75bd58f5c7-csxns   1/1     Running   0          03m
nginx-deployment-75bd58f5c7-n8z6s   1/1     Running   0          03m
```
Si queremos ver información más detallada de un pod podemos ejecutar:

```
$ kubectl describe pod <NOMBRE DEL POD>
```
Por ejemplo:
```
$ kubectl describe pod nginx-deployment-75bd58f5c7-5h9lr

Name:               nginx-deployment-75bd58f5c7-5h9lr
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               10.76.193.217/10.76.193.217
Start Time:         Fri, 05 Jul 2019 12:24:45 -0300
Labels:             app=nginx
                    pod-template-hash=75bd58f5c7
Annotations:        kubernetes.io/psp: ibm-privileged-psp
Status:             Running
IP:                 172.30.244.8
Controlled By:      ReplicaSet/nginx-deployment-75bd58f5c7
Containers:
  nginx:
    Container ID:   containerd://23fc50e10578b38448cb5aad86017e7d142eb88e2c2cd8277c2590649e18f524
    Image:          nginx:1.15.4
    Image ID:       docker.io/library/nginx@sha256:e8ab8d42e0c34c104ac60b43ba60b19af08e19a0e6d50396bdfd4cef0347ba83
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 05 Jul 2019 12:24:55 -0300
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-kb9kj (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-kb9kj:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-kb9kj
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>
```

Dediquele un par de minutos a leer con atencion toda la información que describe al pod, notara que alli hay mucha información relevante como la IP del pod, la imagen que está corriendo, su controlador, etc.

Más adelante veremos cómo exponer nuestro despliegue para que se pueda acceder a nuestro servidor nginx. 

### Dashboard Kubernetes 

Ademas de la CLI, podemos obtener logs y visualizar el estado de nuestro cluster por medio del dashboard de Kubernetes en IBM Cloud. 

Para hacerlo debemos acceder a nuestro cluster desde IBM Cloud. Una vez allí veremos algo como lo siguiente:

<p align="center">
  <img src="/././images/kubernetesdash.png" width="800">
</p>

Una vez alli, hagamos clic sobre el boton *Kubernetes dashboard*, se abrira una nueva pestaña en nuestro navegador donde podremos ver el dashboard de Kubernetes, 

<p align="center">
  <img src="/././images/panel.png" width="800">
</p>

Como podemos ver en la imagen de arriba, en el dashboard lo primero que se ve es una representación del estado de nuestro cluster. También podemos ver nuestro nuestro deployment y si queremos podemos hacer clic sobre el y ver la información del mismo y los logs. En el caso de querer obtener información sobre nuestro deployment desde la CLI podemos utilizar el siguiente comando:

```
$ kubectl describe deployment nginx-deployment 

Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Fri, 05 Jul 2019 12:24:45 -0300
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
                        kubectl.kubernetes.io/last-applied-configuration:
                          {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx-deployment","namespace":"d...
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.15.4
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-75bd58f5c7 (3/3 replicas created)
Events:          <none>
```

<p align="center">
  <img src="/././images/dash.png" width="800">
</p>

Como vemos en la imagen de arriba, también podemos ver nuestros pods y su estado. Si hacemos clic en uno de ellos podemos obtener mucha más información.

<p align="center">
  <img src="/././images/dash3.png" width="800">
</p>

Como se ve en la imagen, arriba a la derecha aparece una opción que nos permite ver los logs de nuestro pod. Si queremos hacer esto desde la CLI debemos ejecutar el siguiente comando:

```
$ kubectl logs <NOMBRE DEL POD>
```

Le recomendamos que le dedique unos minutos a recorrer un poco el dashboard, verá que desde allí puede obtener toda la información del estado de su clúster de manera organizada y fácil de visualizar. Mas adelante, a medida que avancemos con demostraciones prácticas de algunos conceptos iremos viendo cómo varía el estado de nuestro clúster tanto desde la CLI como desde el dashboard. 

### ¿Cómo se accede a las aplicaciones?

En esta etapa del dojo veremos cómo exponer nuestro desarrollo. Volveremos a retomar algunos conceptos anteriormente mencionados y veremos las distintas formas de exponer nuestros pods y deployments.

#### Servicios 

Un **servicio** es una colección de pods expuestas como un endpoint. El servicio propaga información de estado y de red a todos los nodos de trabajo. Cada pod tiene su propia dirección IP y comparte un espacio de nombres PID, una red y un nombre de host.

<p align="center">
  <img src="/././images/service-label1.png" width="600">
</p>

El descubrimiento y enrutamiento entre los Pods dependientes (como los componentes de frontend y backend en una aplicación) son manejados por los Servicios de Kubernetes.
Los servicios coinciden con un conjunto de Pods que usan una  etiqueta en específico. Las etiquetas son pares de clave / valores adjuntos a los objetos y se pueden usar de varias maneras:
Designar objetos para desarrollo, prueba y producción.
Insertar etiquetas de versión
Clasificar un objeto utilizando etiquetas.

<p align="center">
  <img src="/././images/service-label2.png" width="600">
</p>

Hay varios tipos de exposición al servicio:
* **ClusterIP:** expone el servicio en una IP interna en el clúster. Este tipo hace que el Servicio solo sea accesible desde el clúster
* **NodePort:** expone el servicio en el mismo puerto de cada nodo seleccionado en el clúster usando NAT. Hace que un servicio sea accesible desde fuera del clúster usando <NodeIP>:<NodePort>
* **LoadBalancer:** crea un equilibrador de carga externo en la nube actual (si se admite) y asigna una IP externa fija al servicio. 
* **ExternalName:** expone el servicio usando un nombre arbitrario (especificado por external Name en la especificación) al devolver un registro CNAME con el nombre. No se utiliza ningún proxy. Este tipo requiere v1.7 o superior de *kube-dns*

<p align="center">
  <img src="/././images/cs_network_planning_dt.png" width="600">
</p>

Como podemos ver en la imagen de arriba, durante este dojo solo podremos hacer uso de *NodePort*. Recordemos que por estar trabajando en un cluster free solo disponemos de un nodo de trabajo. 

#### Exponer nuestro despliegue de nginx 

Más arriba desplegamos en kubernetes un servidor nginx, lo que no hicimos fue exponerlo. Ahora vamos a exponer ese deployment para que podamos acceder a él. Usaremos NodePort ya que es el unico disponible para nuestro tipo de cluster. 
Lo primero es obtener el nombre de nuestro deployment, si no nos acordamos de cuál era podemos usar el siguiente comando que ya hemos visto anteriormente
```
$kubectl get deployment

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           2d03m
```
Luego utilizaremos el siguiente comando para hacer un exposé de nuestro deployment del tipo NodePort

```
$ kubectl expose deployment nginx-deployment --type=NodePort --name=mi-servicio-dojo

service/mi-servicio-dojo exposed
```
**Nota:** `deployment` puede variar dependiendo de lo que queramos exponer, también puede ser un pod o un servicio. 

Con esto debemos tener nuestro servicio *mi-servicio-dojo* creado, para corroborar podemos ejecutar:

```
$kubectl get services

NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP   172.21.0.1      <none>        443/TCP        17d
mi-servicio-dojo   NodePort    172.21.254.16   <none>        80:31617/TCP   20s
```
Deberian poder algo parecido a como está arriba. Allí se indica:
* **Name:** nombre de nuestro servicio
* **Type:** el tipo de nuestro servicio, en nuestro caso NodePort
* **Cluster-IP:** nos muestra la IP interna en el clúster
* **External-IP:** nos muestra la IP  externa, como el tipo es NodePort, no aplica en este caso ya que la IP externa de nuestro servicio es la IP externa del nodo
* **Port:** es el puerto en el que está expuesto nuestro servicio

En este punto nuestro servicio está expuesto, para acceder a el debemos ingresar IP-EXTENARL-NODO:PORT
Para conocer la ip de nuestro nodo podemos ejecutar un comando que nos describe toda la información del mismo, pero antes debemos obtener el nombre de nuestro nodo:

```
$ kubectl get nodes

 NAME            STATUS   ROLES    AGE   VERSION
10.76.193.217   Ready    <none>   17d   v1.13.7+IKS
```

Ahora que sabemos el nombre ejecutemos el siguiente comando:

```
$ kubectl describe node 10.76.193.217
```

Podremos ver mucha información sobre nuestro nodo, en este momento lo único que nos interesa es la seccion donde nos muestra las direcciones de nuestro nodo, por ejemplo, en mi caso:

```
Addresses:
  InternalIP:  10.76.193.217
  ExternalIP:  50.23.39.117
  Hostname:    10.76.193.217
```

Alli tenemos la IP externa del nodo 10.76.193.217 en el que expusimos nuestro servicio mi-servicio-dojo. Para acceder entonces a nuestro servicio que expone a nginx debemos ingresa a la ExtenarlIP en el puerto de nuestro servicio, en mi caso 50.23.39.117:31617

<p align="center">
  <img src="/././images/nginx.png" width="800">
</p>

### Almacenamiento en Kubernetes

En la sección anterior hemos visto la facilidad de correr una base de datos en contenedores, en nuestro caso hicimos la prueba con mongo al bajar su imagen directo desde docker hub a nuestros pc. En ese caso no configuramos ningún tipo de almacenamiento, pero si quisiéramos correr una base de datos en kubernetes es necesario crear algún tipo de almacenamiento en nuestro clúster que nos permita guardar los datos de esa aplicación. En nuestro caso, por tener un clúster free no podremos hacer uso del almacenamiento externo al clúster.
En la imagen siguiente se muestran los componentes de almacenamiento en un clúster de Kubernetes.

<p align="center">
  <img src="/././images/storage1.png" width="200">
</p>

* **Cluster:**
De forma predeterminada, cada clúster está configurado con un complemento para aprovisionar el almacenamiento de archivos. Puede elegir instalar otros complementos, como el del almacenamiento de bloques. Para utilizar el almacenamiento en un clúster, debe crear un Persistent volume claim, un Persistent Volumen y una instancia de almacenamiento físico. Cuando elimina el clúster, tiene la opción de eliminar las instancias de almacenamiento relacionadas.

* **Persistent volume claim (PVC):** 
Una PVC es la solicitud para suministrar almacenamiento persistente con un tipo y una configuración específicos. Para especificar el tipo de almacenamiento persistente que desea, se debe utilizar las clases de almacenamiento de Kubernetes. El administrador del clúster puede definir clases de almacenamiento, o puede elegir entre una de las clases de almacenamiento predefinidas de IBM Cloud Kubernetes Service. Cuando cree una PVC, la solicitud se envía al proveedor de almacenamiento de IBM Cloud™. En función de la configuración que se haya definido en la clase de almacenamiento, el dispositivo de almacenamiento físico se solicita y se suministra en la cuenta de IaaS de IBM Cloud. Si la configuración solicitada no existe, el almacenamiento no se crea.

* **Persistent Volumen (PV):**
Un PV es una instancia de almacenamiento virtual que se añade como volumen al clúster. El PV apunta a un dispositivo de almacenamiento físico en la cuenta de IaaS de IBM Cloud y resume la API que se utiliza para comunicar con el dispositivo de almacenamiento. Para montar un PV en una app, debe tener una PVC coincidente. Los PV montados aparecen como una carpeta dentro del sistema de archivos del contenedor.

* **Almacenamiento físico:**
Una instancia de almacenamiento físico que puede utilizarse para conservar los datos. Entre los ejemplos de almacenamiento físico en IBM Cloud se incluyen Almacenamiento de archivos, Almacenamiento en bloque, Almacenamiento de objetos. IBM Cloud proporciona alta disponibilidad para las instancias de almacenamiento físico. Sin embargo, los datos que se almacenan en una instancia de almacenamiento físico no se copian automáticamente. En función del tipo de almacenamiento que utilice, existen distintos métodos para configurar soluciones de copia de seguridad y restauración.

## Apendice

En esta sección encontrará material extra que puede utilizar para profundizar en el tema. Además, están los links a los laboratorios de este dojo donde podrá poner en práctica lo visto en esta sección y aprender cosas nuevas. 

### Hands-On 

- [Lab 1 - Desplegar y exponer una app (guestbook)](/assets/kubernetes/lab1) 
- [Lab 2 - Escalar y Actualizar aplicaciones](/assets/kubernetes/lab2) 
- [Lab 3 - Escalar y Actualizar aplicaciones de forma nativa](/assets/kubernetes/lab3) 

### Material extra

- [Doc. Kubernetes](https://kubernetes.io/es/docs/home/)
- [Doc. IBM Cloud](https://cloud.ibm.com/docs/containers?topic=containers-getting-started)

- [Curso Completo](https://www.youtube.com/watch?v=DCoBcpOA7W4&t=790s)
- [Certificacion](https://cognitiveclass.ai/courses/kubernetes-course)



 [← Volver al índice](/././README.md)
 
 [→ Siguiente Sección(Helm)](/pages/7/#helm)

