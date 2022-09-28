#Modernización de Aplicaciones 2.0 - Dojo / página 5


<p align="center">
  <img src="/././images/logoDojo.png" width="500">
</p>


* [← Volver al índice](/././README.md)


## Openshift


En la [sección anterior](/./pages/6/README.md) estuvimos viendo Istio. Ahora vamos a trabajar con Openshift, la solución de RedHat para la orquestación de contenedores en la nube.


<p align="center">
  <img src="/././images/open-logo.png" width="150">
</p>


**Índice de la sección:**

- [Openshift](#openshift)
- [¿Qué es Openshift?](#qué-es-openshift)
- [¿Por que usar Openshift?](#por-que-usar-openshift)
- [Openshift en IBM Cloud](#openshift-en-ibm-cloud)
  - [Beneficios de usar OCP en IBM](#beneficios-de-usar-ocp-en-ibm)
  - [Openshift vs IKS](#openshift-vs-iks)
- [Arquitectura de OpenShift](#arquitectura-de-openshift)
- [Source-to-Image](#source-to-image)
- [Registro integrado y métricas](#registro-integrado-y-métricas)
- [Proyectos](#proyectos)
- [Aplicaciones en Openshift](#aplicaciones-en-openshift)
- [Proceso de compilación y despliegue](#proceso-de-compilación-y-despliegue)
- [Registro Interno de Imágenes](#registro-interno-de-imágenes)
  - [1 - Configurar la conexión](#1---configurar-la-conexión)
  - [2 - Subir una imagen local](#2---subir-una-imagen-local)
  - [3 - Desplegar nuestra imagen desde la consola web](#3---desplegar-nuestra-imagen-desde-la-consola-web)
- [IBM Container Registry con IBM Cloud](#ibm-container-registry-con-ibm-cloud)
  - [¿Cómo Autorizo a mi cluster para extraer imágenes de ICR?](#cómo-autorizo-a-mi-cluster-para-extraer-imágenes-de-icr)
  - [Configurar un imagePullSecret para acceder desde OpenShift a ICR](#configurar-un-imagepullsecret-para-acceder-desde-openshift-a-icr)
    - [a. Crear credenciales IAM y una API Key para el servicio ICR](#a-crear-credenciales-iam-y-una-api-key-para-el-servicio-icr)
    - [b. Utilizar la API Key para crear un imagePullSecret en OpenShift](#b-utilizar-la-api-key-para-crear-un-imagepullsecret-en-openshift)
    - [c. Desplegar una imagen de ICR usando nuestro imagePullSecret en OC](#c-desplegar-una-imagen-de-icr-usando-nuestro-imagepullsecret-en-oc)
- [Apendice](#apendice)
  - [Hands-On](#hands-on)
  - [Material extra](#material-extra)


## ¿Qué es Openshift?


Openshift es un servicio que utiliza contenedores para la construcción, ejecución y despliegue de aplicaciones. En versiones anteriores, estaba basado en kernel namespaces, cGroups y SELinux, pero desde de la versión 3 se basa en Docker y Kubernetes! 
Durante los últimos años Openshift a captado el interés de empresas y desarrolladores, siendo la plataforma líder de Kubernetes, con operaciones automatizadas para administrar implementaciones de nube híbrida y multicloud optimizadas para la productividad del desarrollador y la innovación sin fricción. Red Hat OpenShift se ejecuta en Red Hat Enterprise Linux, el sistema operativo de Linux comercial más implementado en la nube pública. Como dato curioso, más del 90% de las compañías de Fortune 500 confían en Red Hat y el 50% del top 100 usan Openshift.




## ¿Por que usar Openshift?


OpenShift proporciona una plataforma común para que las unidades empresariales alojen sus aplicaciones en la nube sin preocuparse por el sistema operativo subyacente. Esto hace que sea muy fácil de usar, desarrollar e implementar aplicaciones en la nube. Una de las características clave es que proporciona hardware administrado y recursos de red para todo tipo de desarrollo y pruebas. OpenShift también ofrece una versión local, OpenShift Enterprise. En OpenShift, los desarrolladores tienen la posibilidad de diseñar aplicaciones escalables y no escalables y estos diseños se implementan utilizando servidores HAproxy.


Algunas características de OpenShift son:


* Soporte de múltiples bases de datos
* Administración de versiones de código
* Despliegue con un clic
* Soporte Multi Ambiente
* Flujo de trabajo de desarrolladores estandarizado
* Escalado Automático de Aplicaciones
* Consola web
* Inicio de sesión SSH remoto para aplicaciones
* Integración continua y gestión de despliegues
* Integración IDE
* Depuración remota de aplicaciones
* Open Source


## Openshift en IBM Cloud


Para llevar a cabo la parte práctica de esta sección del dojo utilizaremos un cluster de openshift en IBM Cloud. El 1ro de Agosto de 2019, y después de tan solo dos meses en su versión beta, se lanzó una versión estable de Openshift en la nube de IBM, logro que se llevó a cabo gracias al gran número de empresas y desarrolladores que colaborarón. 


Red Hat OpenShift en IBM Cloud es una extensión del Servicio IBM Cloud Kubernetes (IKS), en donde IBM gestiona OpenShift Container Platform por nosotros.  Podremos administrar los clústeres de Kubernetes y OpenShift con la misma API, CLI y consola de IBM Cloud Kubernetes Service para obtener una mejor experiencia de usuario.


### Beneficios de usar OCP en IBM 


Con Red Hat OpenShift en IBM Cloud, los desarrolladores tenemos una forma rápida y segura de contener e implementar cargas de trabajo empresariales en clusters de Kubernetes. Los clusters OpenShift se basan en la orquestación de contenedores de Kubernetes que ofrece consistencia y flexibilidad para todas las operaciones dentro del ciclo de vida de nuestros desarrollos.


Las cargas de trabajo de OpenShift pueden escalarse fácilmente entre centros de datos y regiones multizona de IBM. Al mismo tiempo, puede monitorear, registrar y proteger de manera uniforme las aplicaciones. Debido a que IBM gestiona el servicio, podemos centrarnos en innovar con servicios y middleware de IBM Cloud, como AI y Analytics. También tenemos acceso a las herramientas de código abierto empaquetadas de Red Hat, incluidos los tiempos de ejecución, marcos, bases de datos, entre muchas otras.


### Openshift vs IKS 


Tanto los clústeres de Kubernetes como los de OpenShift son plataformas de contenedores listas para producción que se adaptan a las cargas de trabajo empresariales. La siguiente tabla compara y contrasta algunas características comunes que pueden ayudarlo a elegir qué plataforma de contenedor es la adecuada para su caso de uso.


| Caracteristica | Cluster Kubernetes| Cluster OpenShift |
| ----- | ---- | ---- |
| Experiencia completa de gestión de clústeres a través de las herramientas de automatización de IBM Cloud Kubernetes Service (API, CLI, consola) | Sí | Sí |
| Disponibilidad mundial en zonas individuales y múltiples |  Sí | Sí |
| Orquestación constante de contenedores en proveedores de nube híbrida        | Sí | Sí |
| Acceso a servicios de IBM Cloud como AI                | Sí | Sí |
| Plan Free disponible | Sí | No |
| Ultima version de Kubernetes community | Sí | No|
| CI/CD integrado con Jenkis | No | Sí |
| Contexto de seguridad de la aplicación más estricto configurado de forma predeterminada        | No | Sí
| Experiencia de desarrollador de Kubernetes simplificada, con una consola de aplicaciones | No | Sí
| Sitema operativo | Ubuntu 16.64, 18.64        | Red Hat Enterprise Linux 7|
| Enrutamiento de tráfico externo preferido        | Ingress | Router |




## Arquitectura de OpenShift


Después de todos los beneficios que vimos sobre Docker y Kubernetes, probablemente te estés preguntando donde entra OpenShift  en el juego y qué pieza proporciona la plataforma. Resulta que Kubernetes es excelente para orquestar contenedores, pero para tener una plataforma que ayude a los desarrolladores y administradores de sistemas a lidiar con la pieza más importante para sus "clientes", la aplicación, se necesita algo más.


El objetivo de OpenShift es proporcionar la mejor experiencia para desarrolladores y administradores de sistemas desarrollando, implementando y ejecutando aplicaciones. En otras palabras, OpenShift es una capa además de Docker y Kubernetes que lo hace accesible y fácil para el desarrollador y así crear aplicaciones y una plataforma que es un sueño para los operadores desplegar contener
para las cargas de trabajo de desarrollo y producción.


A continuación veremos un diagrama de la arquitectura de Open Shift que nos servirá para comenzar a comprender su funcionamiento y las herramientas que participan. 


<p align="center">
  <img src="/././images/open1.png" width="750">
</p>


Para administrar nuestro cluster tenemos dos opciones:




* **Consola Web**: OpenShift cuenta con una consola web  con características que permiten a los desarrolladores realizar las acciones necesarias para implementar y ejecutar proyectos existentes. Por ejemplo, una de las características, es una representación gráfica de una aplicación que consta de múltiples contenedores con carga equilibrada, mientras que también utiliza una base de datos como motor de almacenamiento. 


* **CLI**: OpenShift proporciona una herramienta de línea de comandos que está escrita en Go, llamada oc y ejecutable en los sistemas operativos Windows, Apple OS X y Linux. OC puede usarse para realizar cualquier operacion, no hay operaciones que se puedan realizar desde la consola web que no se puedan hacer desde la CLI. 


 
## Source-to-Image


El verdadero poder de OpenShift viene de la mano  de S2I (Source-to-Image) un proyecto que el equipo OpenShift creó como parte de las soluciones de la plataforma OpenShift. 
Este proyecto surgió de la necesidad de reducir las complicaciones y el tiempo que los desarrolladores pierden dia a dia creando Dockerfiles e imagenes y ademas realizar toda la orquestación. Dicho de forma simple, las imágenes de docker basadas en S2I permiten a los desarrolladores interactuar con la plataforma utilizando el código fuente puro. Esto implica que Openshift solo necesita saber la URL del repositorio en git donde se encuentra el código y luego se encarga automáticamente del resto.  
Supongamos que tenemos un proyecto basado en Java que utiliza Maven. Cuando se usa el servidor de aplicaciones WildFly y un repositorio git que contiene su código, la plataforma descargará la imagen base WildFly, clonara su repositorio, lo identificara como un proyecto Maven, ejecutara una un build de Maven y creara una nueva imagen de Docker que contendra el runtime WildFly y nuestra app. Finalmente,  lo desplegará y luego iniciará el contenedor resultante. Sé que esto puede sonar complicado, pero no te preocupes, la plataforma se encarga de todo el trabajo pesado.


## Registro integrado y métricas


Uno de los aspectos más importantes de cualquier proyecto de desarrollo de software es la capacidad de solucionar problemas a medida que surgen. Uno de los primeros lugares en los que un desarrollador mira al comenzar el proceso de depuración es en los logs del proyecto. OpenShift proporciona una vista completa de los registros de su aplicación, incluidos los registros de tiempo de ejecución, los registros del build y los registros del deploy. Tenerlos a su alcance a través de la consola web y la herramienta oc (CLI) simplifica enormemente el proceso de solución de problemas cuando se usan contenedores.


Otro aspecto importante son las métricas sobre nuestras aplicaciones y la utilización de los recursos del sistema por parte de estas. La plataforma OpenShift incluye soporte para esto!


## Proyectos


Otro de los nuevos conceptos que openshift agrega  es el de *proyecto*, que envuelve un namespace con acceso al espacio de nombres controlado a través del proyecto. El acceso se controla a través de un modelo de autenticación y autorización basado en usuarios y grupos. Por lo tanto, los proyectos en OpenShift proporcionan "muros" entre namespaces, asegurando que los usuarios o las aplicaciones solo puedan ver y acceder a lo que tienen permitido.


<p align="center">
  <img src="/././images/open2.png" width="750">
</p>

Esto permite el uso de múltiples desarrolladores en un clúster de OpenShift con privilegios de acceso determinados por la identidad del usuario o el equipo al que pertenecen. Los usuarios pueden asignarse a múltiples grupos y pueden heredar permisos dependiendo de la membresía del grupo. Los grupos a menudo se asignan a equipos o unidades funcionales dentro de una empresa, como desarrolladores, QA, o producción.


## Aplicaciones en Openshift


Una típica interrogante es si un namespace es lo mismo que una aplicación. Pues OpenShift no tiene un concepto formal de aplicación, lo que permite que una aplicación sea flexible según las necesidades del usuario.
Puede dedicar un namespace a todo lo relacionado con una sola aplicación. O bien, siempre que etiquete todos los recursos que componen una aplicación para saber qué va con qué, también puede usar el namespace para contener más de una.
¿Cuáles son las diferentes partes que intervienen en la creación de una aplicación? Eso es lo que vamos a ver ahora.


<p align="center">
  <img src="/././images/open3.png" width="750">
</p>


AL igual que en Kubernetes,la unidad más básica en OpenShift son los pods. Repasemos el concepto de pod y su funcionamiento. 


Un pod esta compuesto por uno o más contenedores y dichos contenedores dentro de un pod comparten una dirección IP única. Pueden comunicarse entre sí a través del "localhost" y también con todos los volúmenes compartidos (almacenamiento persistente). Los contenedores se inician desde una imagen, que en nuestro caso es una imagen de Docker.


Cuando se escala, una aplicación tendrá más de una copia de sí misma, y cada copia tendrá su propio estado local. Cada copia corresponde a una instancia diferente de un pod con los pods gestionados por el controlador de replicación. Como cada pod tiene una IP única, necesitamos una manera fácil de abordar el conjunto de todos los pod en su conjunto. Aquí es donde los servicios entra en juego. El servicio obtiene su propia IP y un nombre DNS. Al realizar una conexión a un servicio, OpenShift enrutará automáticamente la conexión a uno de los pods asociados con ese servicio


Si bien un servicio tiene un nombre DNS, sólo es accesible dentro del clúster OpenShift y no es accesible externamente. Para que un servicio sea accesible externamente, se debe crear una ruta.Al crear una ruta se configura automáticamente un haproxy o un enrutador basado en hardware, con un nombre DNS direccionable externamente, exponiendo el servicio y equilibrando la carga del tráfico entrante a través de los pods.


## Proceso de compilación y despliegue


Hacer que nuestra aplicación se ejecute y administre para nosotros está genial, pero necesitamos una forma de construir y desplegar la aplicación en primer lugar.


Las compilaciones (builds) pueden estar basadas directamente en el código fuente y usar S2I para lenguajes comunes como Java, PHP, Ruby o Python, o estar basadas en Docker y crear compilaciones desde un archivo Docker. 


El resultado del proceso de compilación es una imagen que se almacena en un registro Docker integrado listo para entregarlo a los nodos cuando se despliega la aplicación. Si ya tiene una imagen Docker existente en un registro externo como Docker Hub o Container Registry, también puede hacer referencia a ella mediante una secuencia de imágenes en lugar de construirla localmente.


Al desplegar una aplicación, la configuración del despliegue es lo que define la plantilla para un pod y administra el despliegue de nuevas imágenes o cambios de configuración cada vez que cambian. Una configuración de despliegue única suele ser análoga a un microservicio único. Una configuración de despliegue admite una serie de estrategias de implementación diferentes, que incluyen un apagado completo y luego reinicio, actualizaciones continuas o sus propios comportamientos totalmente personalizados. 


Como ya hemos visto en la sección de Kubernetes, el resultado de un despliegue es el controlador de replicación, que luego administra sus pods y los mantiene en funcionamiento.


## Registro Interno de Imágenes  

Así como en Kubernetes utilizamos Container Registry para almacenar nuestras imágenes en la nube y utilizarlas para desplegar en nuestro cluster, OpenShift proporciona un registro interno que podemos utilizar con este fin y que veremos a continuación.

### 1 - Configurar la conexión  

Una de las alternativas que tenemos para alojar nuestras imágenes en OpenShift es utilizar el registro interno.  Lo primero que haremos será obtener una descripción del mismo, para esto podemos utilizar el siguiente comando: 

$ oc describe pvc registry-backing -n default 

 
``` 

Name:          registry-backing 
Namespace:     default 
StorageClass:  ibmc-file-bronze 
Status:        Bound 
Volume:        pvc-6b9cfbee-c15d-11e9-820e-c63272ece3c7 
Labels:        billingType=hourly 
               region=us-south 
               zone=dal13 
Annotations:   ibm.io/provisioning-status=complete 
               kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"labels":{"billingType":"hourly"},"name":"registry-backing","namespace":... 
               pv.kubernetes.io/bind-completed=yes 
               pv.kubernetes.io/bound-by-controller=yes 
               volume.beta.kubernetes.io/storage-provisioner=ibm.io/ibmc-file 
Finalizers:    [kubernetes.io/pvc-protection] 
Capacity:      20Gi 
Access Modes:  RWX 
Events:        <none> 
``` 


En este caso tenemos 20G para almacenar nuestras imágenes, en caso que necesitemos más podremos ampliar su capacidad.  

Lo que haremos a continuación es configurar una ruta para conectarnos a nuestro registro, para esto lo primero que debemos hacer es chequear que el servicio docker-registry exista para el registro interno. 

```
$  oc get svc 
``` 

En mi caso: 

```
NAME               TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE 

docker-registry    ClusterIP      172.21.27.254    <none>          5000/TCP                     29d 

kubernetes         ClusterIP      172.21.0.1       <none>          443/TCP,53/UDP,53/TCP        29d 

nginx-example      ClusterIP      172.21.20.224    <none>          8080/TCP                     28d 

registry-console   ClusterIP      172.21.161.205   <none>          9000/TCP                     29d 

router             LoadBalancer   172.21.45.166    169.60.156.75   80:30343/TCP,443:30974/TCP   29d 

``` 

Creemos ahora una ruta para el servicio docker-registry usando reencrypt TLS. Con esto la ruta de conexión completa entre el usuario y el registro interno estará encriptada.   

```
$ oc create route reencrypt --service=docker-registry 
```
 
Para asegurarnos que se haya creado con éxito podemos ejecutar el siguiente comando: 
 
```
$ oc get route docker-registry 
```

En mi caso: 

 ```
NAME              HOST/PORT                                                                                                           PATH      SERVICES          PORT       TERMINATION   WILDCARD 

docker-registry   docker-registry-default.openshift-jose2-5290c8c8e5797924dc1ad5d1b85b37c0-0001.us-south.containers.appdomain.cloud             docker-registry   5000-tcp   reencrypt     None 

``` 

 

Ahora podemos activar balanceo de carga en la ruta agregando la siguiente anotación (también podemos hacerlo mediante la consola web de oc)  

```
 $ oc edit route docker-registry 
```


``` 
apiVersion: route.openshift.io/v1 
kind: Route 
metadata: 
annotations: 
    haproxy.router.openshift.io/balance: source 

... 
```
 

Ya podemos loguearnos en el registro interno de nuestro cluster por medio de la consola. Para lograrlo debemos ejecutar el siguiente comando: 

 
```
$ docker login -u $(oc whoami) -p $(oc whoami -t) docker-registry-default.<cluster_name>-<ID_string>.<region>.containers.appdomain.cloud 
```
 

Si le resulta más sencillo, puede obtener la ruta completa (para agregar en el comando anterior) utilizando el siguiente comando: 

 
```
$ oc get route docker-registry 
```
 

Si no hay ninguna complicación recibiremos un mensaje indicándonos que nos logueamos con éxito!

### 2 - Subir una imagen local 

 

Ya que tenemos configurada la ruta al registro interno, podemos subir nuestras imágenes que tengamos almacenadas de forma local. Utilicemos como prueba la imagen que creamos en el laboratorio de la sección de microservicios.  

Primero debemos loguearnos en el registro interno de nuestro clúster indicando el proyecto en el cual vamos a trabajar 

Para esto debemos ejecutar el siguiente comando: 

``` 
$ docker login -u $(oc whoami) -p $(oc whoami -t) docker-registry-default.<cluster_name>-<ID_string>.<region>.containers.appdomain.cloud/NOMBRE-DEL-PROYECTO
```

 

En mi caso voy a utilizar el proyecto "dojo-containers": 

 
```
$  docker login -u $(oc whoami) -p $(oc whoami -t)  docker-registry-default.openshift-jose2-5290c8c8e5797924dc1ad5d1b85b37c0-0001.us-south.containers.appdomain.cloud/dojo-containers 
```
 
 

Una vez que estemos logueados podemos realizar un tag a nuestra imagen: 
 
```
$docker tag miapp:latest docker-registry-default.<cluster_name>-<ID_string>.<region>.containers.appdomain.cloud/<project>/<image_name>:<tag>
```
 

En mi caso, la imaen local se llama miapp, la subiré al cluster usando la ruta que anteriormente configuramos y la asignare al proyecto dojo-containers. 

 
```
$docker tag miapp:v1 docker-registry-default.docker-registry-default.openshift-jose2-5290c8c8e5797924dc1ad5d1b85b37c0-0001.us-south.containers.appdomain.cloud/dojo-containers/dojo-containers/miapp:v1 
```
 

Ahora hagamos un push de nuestra imagen: 
 
```
$ docker push docker-registry-default.docker-registry-default.openshift-jose2-5290c8c8e5797924dc1ad5d1b85b37c0-0001.us-south.containers.appdomain.cloud/dojo-containers/dojo-containers/miapp:v1 
```

¡Listo! la imagen se encuentra en nuestro registro interno lista para que la utilicemos en nuestros proyectos.  

### 3 - Desplegar nuestra imagen desde la consola web 

Ahora que tenemos nuestra imagen almacenada en nuestro cluster, podemos proceder a desplegarla. Lo que debemos hacer es hacer clic sobre “deploy image” para que se habrá la ventana de configuración. 
Una vez allí debemos selección la opción “Image Stream Tag”, luego seleccionamos el proyecto en el cual subimos nuestra imagen (en mi caos “dojo-containers”) y luego la imagen.  
Luego de completar la configuración básica podemos hacer clic en “Deploy” para comenzar el despliegue.  

## IBM Container Registry con IBM Cloud

Si queremos usar imágenes que tengamos alojadas en nuestro registro privado de IBM Cloud (Container Registry)  debemos crear un imagePullSecret por cada región del servicio (globales y regionales). Luego, agregar ese imagePullSecret a OpenShift.  

### ¿Cómo Autorizo a mi cluster para extraer imágenes de ICR? 

Para extraer imágenes de un registro, el cluster de IKS utiliza un tipo de secreto llamado imagePullSecret. Este secreto para la extracción de imágenes almacena las credenciales para acceder a un container registry, como lo es IBM Cloud Container Registry. Por defecto al crear un cluster automáticamente se configura para extraer imágenes de nuestro ICR (en IKS), pero se puede configurar otros imagePullSecret para acceder a otros registros, por ejemplo, un ICR en otra cuenta de IBM Cloud. En el caso de OpenShift este secreto no se configura de manera automática, así que vamos a analizar cómo podemos configurarlo en nuestro cluster. En un principio lo que haremos será configurar el imagePullSecret a nivel de Kubernetes dentro del cluster de OpenShift! 


Cuando creamos un cluster este tiene un ID de servicio de IBM Cloud IAM, el cual recibe un rol de acceso al servicio de IAM Reader para ICR. Estas credenciales de servicio almacenan una API Key, la cual no caduca y se almacena en un imagePullSecret dentro del cluster. Estos imagePullSecret se almacenan en el namespaces default de Kubernetes, al usar secretos para la extracción de imágenes, los despliegues pueden extraer imágenes (acceso de solo lectura en este caso) en el registro global y regional para construir contenedores. El registro global almacena de forma segura las imágenes publicas proporcionadas por IBM a las que puede hacer referencias en los despliegues en lugar de tener diferentes referencias parea las imágenes que se almacenan en cada registro regional. El registro regional almacena de forma seguirá sus propias imágenes privadas. 

En este punto ya tenemos una idea de cómo funciona Container Registry y como es la comunicación con nuestros cluster de Kubernetes, puede que ahora tenga alguna de estas preguntas: 

 

 - *¿Puedo restringir el acceso de extracción de una imagen a un determinado registro regional?*

Pues sí, podemos editar la política IAM existente del servicio, aquella que anteriormente vimos que tenía accedo Reader.  

 

- *¿Puedo extraer imágenes en un namespace que no sean default?*

Por defecto no. Al utilizar la configuración predetermina del cluster, puede desplegar contenedores desde cualquiera imagen que este almacenada en ICR en el namespace default. Para utilizar estas imágenes en otros namespace o cuenta de IBM Cloud tendremos que crear nuestro propio imagePullSecret (lo veremos a continuacion!!!)  


### Configurar un imagePullSecret para acceder desde OpenShift a ICR 


Configuremos nuestro propio imagePullSecret en el cluster para poder desplegar contenedores usando imágenes que tengamos almacenadas en ICR. Después de crear nuestro secreto, los contenedores deberán utilizar este secreto para estar obtener autorización y extraer imágenes de ese registro. 

Para empezar, si ya tiene un registro privado, debe almacenar las credenciales del registro en un secreto de extracción de imágenes de Kubernetes (imagePullSecret) y hacer referencia a este secreto desde su archivo de configuración. 

#### a. Crear credenciales IAM y una API Key para el servicio ICR 

Comencemos por crear credenciales IAM para nuestro imagePullSecret  


```
$ ibmcloud iam service-id-create <NOMBRE> --description "<DESCRIPCION>"
```

 
```
$ jpanizza@jpanizaa:~$ ibmcloud iam service-id-create iam-cr-openshift --description "Credenciales para Container Registry OpenShift"  

Creating service ID iam-cr-openshift bound to current account as jose.panizza@ibm.com... 
OK 
Service ID iam-cr-openshift is created successfully 
Name          iam-cr-openshift    
Description   Credenciales para Container Registry OpenShift    
CRN           crn:v1:bluemix:public:iam-identity::a/7f1ce657c4db490fb6b456639f62782a::serviceid:ServiceId-51c4018c-a7d7-4946-a320-7ab3eefea95c    
Bound To      crn:v1:bluemix:public:::a/7f1ce657c4db490fb6b456639f62782a:::    
Version       1-961c278518a1c889b7975fd182f44306    
Locked        false    
UUID          ServiceId-51c4018c-a7d7-4946-a320-7ab3eefea95c   
```

Ahora creemos una política de acceso para brindarle a las credenciales que acabamos de crear acceso a IBM Cloud Container Registry 

 

```
$ ibmcloud iam service-policy-create <NOMBRE-CREDENCIAL> --roles <ROL-DE-ACCESO> --service-name container-registry [--region <REGION>] [--resource-type namespace --resource <NAMESPACE-CR>]
```

Como podemos ver, debemos indicar el rol que vamos a otorgarle a la credencial de IAM, este puede ser: Reader, Writer o Manager.  

Luego como opcional podemos indicar la región del servicio a la cual va a tener acceso, así como también el namespace en Container Registry. En mi caso no voy a indicar ninguno de estos por lo cual esta credencial va a tener acceso completo a todas las regiones y namespaces de ICR. 

 
```
$ jpanizza@jpanizaa:~$ ibmcloud iam service-policy-create iam-cr-openshift --roles Manager --service-name container-registry 
Creating policy under current account for service ID iam-cr-openshift as jose.panizza@ibm.com... 
OK 
Service policy is successfully create        
Policy ID:   93c4de6b-5043-4cfd-99eb-011ff8758b02    
Version:     1-d2e5ab9a3fb21fd698ecd559a990b9c6    
Roles:       Manager    
Resources:                             
             Service Name       container-registry       
             Service Instance          
             Region                    
             Resource Type             
             Resource     

```
Ahora creemos una clave API para el ID del servicio IAM que acabamos de crear. Asignemos un nombre similar y una descripción clara que nos ayude a recuperarla en el futuro. 

```$ ibmcloud iam service-api-key-create <NOMBRE-CREDENCIAL>-key <NOMBRE-CREDENCIAL> --description "API key para le servicoio ID <NOMBRE-CREDENCIAL> para el cluster <NOMBRE-CLUSTER> "
```
 
En mi caso:
```
$ ibmcloud iam service-api-key-create iam-cr-openshift-key iam-cr-openshift --description "API key para le servicoio ID iam-cr-openshift para el cluster openshift-jose2" 
Creating API key iam-cr-openshift-key of service iam-cr-openshift as jose.panizza@ibm.com... 
OK 
Service API key iam-cr-openshift-key is created 
Please preserve the API key! It cannot be retrieved after it's created. 
Name          iam-cr-openshift-key    
Description   API key para le servicoio ID iam-cr-openshift para el cluster openshift-jose2    
Bound To      crn:v1:bluemix:public:iam-identity::a/7f1ce657c4db490fb6b456639f62782a::serviceid:ServiceId-51c4018c-a7d7-4946-a320-7ab3eefea95c    
Created At    2019-09-17T14:33+0000    
API Key       npzh-rRA69xbTCMF8K1gfdJrL80KjzS4Ueze-CMs50sD    
Locked        false    
UUID          ApiKey-3d0d237f-3da3-48b2-bb8e-1e9a6dad1ed3   
```


**¡¡¡Hay que guardar la API KEY porque no la podremos recuperar!!!**


#### b. Utilizar la API Key para crear un imagePullSecret en OpenShift 

Ahora que tenemos nuestras credenciales para hacer uso de nuestro servicio IBM Cloud Container Registry podemos proceder a trabajar desde OpenShift.  


Vamos a crear un imagePullSecret en OpenShift y almacenar allí las credenciales de la API Key en un namespace del cluster.  


```
$ oc --namespace <NAMESPACE-KUBERNETES> create secret docker-registry <NOMBRE-DEL-SECRETO> --docker-server=<ICR-URL> --docker-username=iamapikey --docker-password=<API-KEY> --docker-email=<EMAIL>
```

Analicemos los campos de este comando: 

| Primer encabezado | Segundo encabezado |
| ------------- | ------------- |
| NAMESPACE-KUBERNETES  | El nombre del namespace donde almacenaremos el ImagePullSecret   |
| NOMBRE-DEL-SECRETO   | El nombre que utilizaremos para identificar y utilizar este ImagePullSecret   |
| ICR-URL   | La URL de nuestro Container Registry (varia por región)   |
| --docker-username   | Utilicemos iamapikey   |
| --docker-password   | Utilicemos nuestra API Key   |
| --docker-email   | Tiene que tener formato de email (ejemplo@email.com), no se utiliza para nada. |

```
jpanizza@jpanizaa:~$ oc --namespace default create secret docker-registry icrpanizza --docker-server=us.icr.io --docker-username=iamapikey --docker-password=npzh-rRA69xbTCMF8K1gfdJrL80KjzS4Ueze-CMs50sD --docker-email=jose.panizza@ibm.com 

secret/icrpanizza created 
```

#### c. Desplegar una imagen de ICR usando nuestro imagePullSecret en OC

Ahora que nuestro pullImageSecret esta creado podemos probar desplegar un POD con una imagen alojada en nuestro IBM Cloud Container Registry 

Creemos un YAML simple con la siguiente información: 

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: mypod 
spec: 
  containers: 
    - name: <container_name> 
      image: <region>.icr.io/<namespace_name>/<image_name>:<tag> 
  imagePullSecrets: 
    - name: <secret_name>
```

Allí debemos indicar no solo la ruta de nuestra imagen sino también el nombre del imagePullSecret que creamos anteriormente.   

Por ejemplo, en mi caso: 


```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: icrpanizzanginx6 
spec: 
  containers: 
    - name: icrpanizzanginx6 
      image: us.icr.io/test-openshift/nginx:v2 
  imagePullSecrets: 
    - name: icrpanizza 
```
Luego: 

 
```
$ oc apply –f NOMBRE-DEL-ARCHIVO.yaml 
```

Listo! Debería haber podido desplegar un pod con una imagen alojada en su IBM Cloud Container Registry 
 
## Apendice

En esta sección encontrará material extra que puede utilizar para profundizar en el tema. Además están los links a los laboratorios de este dojo donde podrá poner en práctica lo visto en esta sección y aprender cosas nuevas. 

### Hands-On 

- [Lab 1 - Crear una Aplicación y desplegarla en Openshift](/assets/openshift/lab1/#)
- [Lab 2 - Agregar dependencias y una Base de datos a nuestra Aplicación](/assets/openshift/lab2/#)


### Material extra

- OpenShift for Developer (Libro) [->](https://assets.openshift.com/hubfs/pdfs/OpenShift_for_Developers_Red_Hat.pdf?hsLang=en-us&extIdCarryOver=true&sc_cid=701f2000001OH7iAAG)
- Deploying to OpenShift (Libro) [->](https://assets.openshift.com/hubfs/pdfs/Deploying_to_OpenShift.pdf?hsLang=en-us&extIdCarryOver=true&sc_cid=701f2000001OH7iAAG)
- DevOps with OpenShift (Libro) [->](https://assets.openshift.com/hubfs/pdfs/DevOps_with_OpenShift.pdf?hsLang=en-us&extIdCarryOver=true&sc_cid=701f2000001OH7iAAG)
