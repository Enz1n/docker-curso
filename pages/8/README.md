# Modernización de Aplicaciones 2.0 - Dojo / Monitoreo de OpenShift



<p align="center">
  <img src="/././images/logoDojo.png" width="500">
</p>


* [← Volver al índice](/././README.md) 


## Monitoreo de un cluster OpenShift




En esta sección del dojo estaremos trabajando con algunas herramientas que nos permitirán monitorear y administrar nuestro cluster.




## Grafana y Prometheus 




Como bien dice el titulo de este dojo, todo lo que hemos visto anteriormente se centra en un solo concepto, el de modernización de aplicaciones, pero hay algo que nos esta quedando pendiente y es como administrar y monitorear todo lo que ocurre en nuestro cluster. Es en este punto en el que las herramientas para le monitoreo de contenedores como Prometheus y Grafana son protagonistas. 




OpenShift Container Platform cuenta con un stack para el monitoreo preconfigurado y de actualización automática que se basa principalmente en Prometheus. Proporciona monitoreo de los componentes del clúster y se envía con un conjunto de alertas para notificar inmediatamente al administrador del clúster sobre cualquier problema que ocurra y un conjunto de paneles de visualización de Grafana para interpretar todos estos datos.








<p align="center">
  <img src="/././images/openshift-monitoring.png" width="800">
</p>




En este diagrama podemos ver cómo interactúan estas herramientas dentro del cluster de kubernetes. El operador Prometheus (PO) crea, configura y gestiona instancias de Prometheus y Alertmanager. También genera automáticamente configuraciones de monitoreo basadas en distintas consultas utilizando tags de Kubernetes.




Además de Prometheus y Alertmanager, OpenShift Container Platform Monitoring también incluye Node-exporter y kube-state-metrics. Node-exporter es implementado en cada nodo para recopilar métricas sobre él. Por otro lado kube-state-metrics convierte los objetos de Kubernetes en métricas procesables por Prometheus.




Los objetivos monitoreados como parte de la supervisión del clúster son:




* Prometheus en si mismo
* Prometheus-Operator
* cluster-monitoring-operator
* Alertmanager cluster instances
* Kubernetes apiserver
* kubelets (kubelet incorpora cAdvisor para metricas por container)
* kube-controllers
* kube-state-metrics
* node-exporter
* etcd (siempre y cuando el monitoreo etcd este activado




*Los componentes de esta lista se actualizan de forma automatica!*




## Acceso a herramientas de monitoreo preconfiguradas




Tanto Grafana como Prometheus pueden ser instaladas dentro de un proyecto en nuestro cluster de OpenShift, pero además, por defecto ya vienen instaladas en nuestro cluster así que veamos como acceder a ellas desde la consola web.




Lo primero es acceder a la consola del cluster, haciendo clic sobre "Cluster Console" en el menú desplegable de la barra superior como se puede ver a continuación.




<p align="center">
  <img src="/././images/ocm1.png" width="800">
</p>




Luego, haciendo clic sobre "Monitoring" en el menú lateral izquierdo, se desplegaran 3 opciones:




* **Metrics:** Para acceder al dashboard de Prometheus 
* **Alerts:** Desde aquí podremos configurar el Alertmanager
* **Dashboards:** Seremos redireccionados a Grafana






## Utilizar Grafana para monitorear nuestro cluster


Cuando accedamos a grafana podemos ver que ya hay varios dashboard preconfigurados para realizar el monitoreo de algunos componentes.




<p align="center">
  <img src="/././images/gr1.png" width="800">
</p>


En un principio hagamos clic sobre *K8s/Compute Resources/Cluster* para acceder a un dashboard con métricas de nuestro cluster de openshift. Por ejemplo podemos ver el estado de la CPU y el consumo de cada proyecto. 




<p align="center">
  <img src="/././images/gr2.png" width="800">
</p>




<p align="center">
  <img src="/././images/gr4.png" width="800">
</p>


Haciendo clic sobre *K8s/Compute Resources/Cluster*, podemos cambiar de tablero de visualización, hagamos clic sobre *K8s/Compute Resources/Namespace*. 




Para ir familiarizándonos con la interfaz, podemos ver que en todos estos dashboards hay una opcion llamada *datasource* donde indica prometheus como la fuente de los datos. Más adelante entraremos en detalle en esto, de hecho en el laboratorio de esta sección veremos cómo configurar un datasource propio. 


En este dashboard tenemos la posibilidad de filtrar por namespace como podemos ver la siguiente imagen


<p align="center">
  <img src="/././images/gr6.png" width="800">
</p>


Una vez que indicamos un filtro, se le comienza a enviar cada 10s consultas al datasource para que devuelva los datos que hayan sido previamente configurados para este dashboard aplicando el filtro correspondiente. 


Al igual que en el caso del dashboard anterior también podemos ver métricas de la CPU y la memoria de los pods en un namespace de Kubernetes. 






<p align="center">
  <img src="/././images/gr7.png" width="800">
</p>




Haciendo clic en la sección donde se indica el tiempo de actualización de los datos, se abrirá un panel desde donde podemos configurar estos valores y así indicar cada cuando queremos que se actualicen los datos además de poder configurar el rango de fechas del cual queremos visualizarlos. 


<p align="center">
  <img src="/././images/grf.png" width="800">
</p>


Otro de los dashboard que ya vienen configurados en OpenShift es el de monitoreo de un Pod en específico. Como vemos a continuación, en los filtros iniciales además de indicar el *datasource* y el *namespace* también se debe indicar el pod.




<p align="center">
  <img src="/././images/gr9.png" width="800">
</p>


Por ultimo, tambien esta bueno ver el siguiente dashboard, el del Nodo. Desde aquí podremos visualizar el estado general de cada nodo de nuestro cluster. 


<p align="center">
  <img src="/././images/gr10.png" width="800">
</p>


Durante el laboratorio configuraremos un nuevo datasource y crearemos un dahsboard basados en un archivo JSON. 



## Prometheus 


## Alertmanager



# Lab 1 - Configurar el monitoreo para una aplicación en 6 pasos
BuildsBuilds


En este laboratorio veremos como configurar el monitoreo para una aplicación demo en 6 pasos.  Haremos uso de una aplicación que fue diseñada para este fin por "https://labs.consol.de" e instalaremos Grafana y Prometheus en nuevos proyectos e independientes. 

## Paso 1 -  Despleguemos la aplicación demo

El primer paso será implementar una aplicación demo muy simple. Es una aplicación SpringBoot que proporciona dos servicios RESTful ( "/ hello-world" y "/ metrics" ) y ya facilita el monitoreo con Prometheus. 
La aplicación proporciona métricas para contar las solicitudes del servicio "hello-world", así como las métricas de endpoints de Java. 


En el siguiente fragmento de código podemos ver una muestra  de cómo se implementan las métricas de Prometheus en los servicios de Restful:


```java
@Component
@Path("/")
public class Metrics {


    private final Logger logger = LogManager.getLogger(Metrics.class);


    private final Counter promRequestsTotal = Counter.build()
                    .name("requests_total")
                    .help("Total number of requests.")
                    .register();
  {
    DefaultExports.initialize();
  }
  @GET()
  @Path("/hello-world")
  @Produces(MediaType.TEXT_PLAIN)
  public String sayHello() {
    promRequestsTotal.inc();
    return "hello, world";
  }


  @GET()
  @Path("/metrics")
  @Produces(MediaType.TEXT_PLAIN)
  public StreamingOutput metrics() {
    logger.info("Starting service for metrics");
    return output -> {
      try (Writer writer = new OutputStreamWriter(output)) {
        TextFormat.write004(writer, CollectorRegistry.defaultRegistry.metricFamilySamples());
      }
    };
  }
}
```


Para implementar la aplicación demo, primero vamos a crear un proyecto en OpenShift llamado "demoapp" y desplegaremos la aplicación demo en ese proyecto.


```bash
$ oc new-project demoapp 
```
Para desplegar la aplicación debemos ejecutar el siguiente comando:


```bash
$ oc new-app -f https://raw.githubusercontent.com/ConSol/springboot-monitoring-example/master/templates/restservice_template.yaml -n demoapp
```
Puede corroborar el estado de su aplicacion desde la consola web y ver alli la ruta para acceder.


<p align="center">
  <img src="/././images/labm1.png" width="800">
</p>


En mi caso la ruta de acceso es: 


*http://restservice-demoapp.openshift-jose2-5290c8c8e5797924dc1ad5d1b85b37c0-0001.us-south.containers.appdomain.cloud*






<p align="center">
  <img src="/././images/labm2.png" width="800">
</p>




Puedo acceder desde alli tanto a "Hello World":




<p align="center">
  <img src="/././images/labm3.png" width="800">
</p>


Como a Metrics




<p align="center">
  <img src="/././images/labm4.png" width="800">
</p>




## Paso 2: Desplegar Prometheus en un proyecto separado




Como ya hemos visto, Prometheus viene configurado en nuestro cluster pero ahora vamos a desplegar una instancia separada en un nuevo proyecto. 
Vamos a implementar Prometheus con contenedores sidecar para OAuth (para el navegador y las "cuentas de servicio") y para alertas. El inconveniente con el uso de OAuth en este momento es que las cuentas de servicio (es decir, de Grafana) no pueden autenticarse con un token a menos que la cuenta de servicio sidecar OAuth tenga asignado el rol "system:auth-delegator".


 Desafortunadamente, el rol de clúster "system:auth-delegator" no está asignado por defecto a los roles administrator y view. Otra alternativa es comunicarse directamente con Prometheus a través del endpoint del servicio. Por lo tanto, Grafana puede consultar datos de Prometheus sin autenticación. Cualquier otra cuenta de usuario, grupo o servicio que conozca el endpoint del servicio también puede omitir la autenticación. En la mayoría de los casos, eso no debería ser un problema ya que la cuenta de servicio de Grafana solo debería tener acceso de lectura (rol "view") al proyecto con Prometheus.


 Creemos un nuevo proyecto y llamemoslo *demoprometheus* :


 ```
$ oc new-project prometheuslab
 ```


 En el siguiente paso, desplegaremos Prometheus con el rol "system:auth-delegator" con el siguiente comando:


 ```
$ oc new-app -f https://raw.githubusercontent.com/ConSol/springboot-monitoring-example/master/templates/prometheus3.7_with_clusterrole.yaml -p NAMESPACE=prometheuslab
 ```


o sin dicho rol:


```
$ oc new-app -f https://raw.githubusercontent.com/ConSol/springboot-monitoring-example/master/templates/prometheus3.7_without_clusterrole.yaml -p NAMESPACE=prometheuslab
```


En nuestro  caso vamos a hacerlo sin el rol "system:auth-delegator"


Una vez haya hecho lo anterior podemos ver en nuestro proyecto el despliegue y ademas hacer clic sobre la ruta para acceder al dashboard de Prometheus.


<p align="center">
  <img src="/././images/labm5.png" width="800">
</p>


En mi caso la ruta de acceso es:




*https://prometheus-prometheuslab.openshift-jose2-5290c8c8e5797924dc1ad5d1b85b37c0-0001.us-south.containers.appdomain.cloud/*


<p align="center">
  <img src="/././images/labm6.png" width="800">
</p>


Podra ver que solo está analizando pods y servicios en el proyecto "demoprometheus", ya en el paso 4 cambiaremos para para incluir a nuestra aplicación. 


## Paso 3: Desplegar Grafana en un nuevo proyecto


Comencemos creando un nuevo proyecto para grafa:


```bash
$ oc new-project grafanalab
```


Una vez creado el proyecto podemos desplegar grafana en el con el siguiente comando:


```bash
$ oc new-app -f https://raw.githubusercontent.com/ConSol/springboot-monitoring-example/master/templates/grafana.yaml -p NAMESPACE=grafanalab
```


A continuación, debemos otorgar a Grafana acceso "view" al proyecto "demoprometeo" con el siguiente comando, para que Grafana pueda mostrar los datos de Prometeo:


```bash
$ oc policy add-role-to-user view system:serviceaccount:grafanalab:grafana-ocp -n prometheuslab


```


Ahora accedemos a nuestro dashboard de grafana 


<p align="center">
  <img src="/././images/labm7.png" width="800">
</p>


Como podemos ver en la imagen, la ruta de mi dashboard de grafana es la siguiente:


*https://grafana-ocp-demografana.openshift-jose2-5290c8c8e5797924dc1ad5d1b85b37c0-0001.us-south.containers.appdomain.cloud*


Al acceder hagamos clic en "Add data Source"


<p align="center">
  <img src="/././images/labm8.png" width="800">
</p>


### Configurar la fuente de datos de Prometheus sin el rol "sistem:auth-delegator"




Si instaló Prometheus sin la función "sistem:auth-delegator" (como yo) , debemos proporcionar un nombre para la fuente de datos. Vamos a utilizar el nombre "Demo-Promtheus-OCP" y en type seleccionemos "Prometheus".
Establezca en la URL el endpoint del servicio Prometheus. Esta dirección IP se puede obtener con el siguiente comando:


```bash
$ oc describe service prometheus -n prometheuslab


```
Por ejemplo en mi caso:


```
Name:              prometheus
Namespace:         prometheuslab
Labels:            name=prometheus
Annotations:       openshift.io/generated-by=OpenShiftNewApp
                   prometheus.io/scheme=https
                   prometheus.io/scrape=true
                   service.alpha.openshift.io/serving-cert-secret-name=prometheus-tls
                   service.alpha.openshift.io/serving-cert-signed-by=openshift-service-serving-signer
Selector:          app=prometheus
Type:              ClusterIP
IP:                172.21.163.118
Port:              prometheus-https  443/TCP
TargetPort:        8443/TCP
Endpoints:         172.30.53.130:8443
Port:              prometheus-http  9090/TCP
TargetPort:        9090/TCP
Endpoints:         172.30.53.130:9090
Session Affinity:  None
Events:            <none>


```


El endpoint es  172.30.53.130:9090, a la hora de escribirlo es fundamental agregar "http://" para que funcione.


Luego haga clic en "Save and Test", deberia obtener un mensaje como el siguiente:


<p align="center">
  <img src="/././images/labm9.png" width="800">
</p>


## Paso 4 - Actualizar la configuración de Prometheus para usar la aplicación demo


Por el momento, Prometheus solo está trabajando con los pod y servicios dentro del proyecto "prometheuslab". Vamos a cambiar eso para que también incorpore a nuestra app demo.
 Para esto necesitamos adaptar la configuración de Prometheus que está almacenada en el "Config Maps" dentro del proyecto Prometheus.


En mi caso voy a editarlo utilizando la consola web, para eso lo primero es acceder desde la consola web al proyecto "prometheuslab" y una vez allí hacer clic en el menu lateral izquierdo en donde dice "Resources" y luego en "Config Maps".


Una vez allí hacemos clic sobre "prometheus" y luego editemos el yaml haciendo clic sobre "Action" y "Edit YAML"


Lo primero es buscar esta sección del documento:


```yaml
    - job_name: 'kubernetes-pods'


      kubernetes_sd_configs:
      - role: pod
        namespaces:
          names:
          - prometheuslab
```


Alli debajo de prometheuslab agreguemos demoapp, deberia quedar asi:


```yaml
    - job_name: 'kubernetes-pods'


      kubernetes_sd_configs:
      - role: pod
        namespaces:
          names:
          - prometheuslab
          - demoapp
```


Luego busquemos esta otra sección:


```yaml
      kubernetes_sd_configs:
      - role: service
        namespaces:
          names:
          - prometheuslab
```


Y hagamos lo mismo:


```yaml
      kubernetes_sd_configs:
      - role: service
        namespaces:
          names:
          - prometheuslab
          - demoapp
```


Guardemos el archivo con los nuevo cambios. 


Como ahora ya hemos agregado la aplicación demo en el mapa de configuración, debemos otorgar acceso a la cuenta de servicio promethes al proyecto demoapp. Para ello ejecutemos:


```
$ oc policy add-role-to-user view system:serviceaccount:prometheuslab:prometheus -n demoapp


```
## Paso 5: Actualizar la configuración del despliegue de la aplicación demo para permitir la supervisión de Prometheus


Anteriormente actualizamos la configuración de Prometheus. Ahora necesitamos recargar la configuración en Prometheus para que pueda encontrar los pods y servicios en el proyecto agregado. En nuestro caso, simplemente podríamos matar el pod de Prometheus con el comando:


```
$ oc delete pod prometheus-0 --grace-period=0 --force
```


Pero para un entorno en producción, no es una buena idea simplemente matar a Prometheus para volver a cargar la configuración ya que esto interrumpe la supervisión. 
Hay una forma alternativa, esta implica enviar un POST vacío a la URL de Prometheus con el path "/reload". Una de las formas de hacer esto es conectarse al pod y hacer la peticion POST como localhost:


```
$ oc exec prometheus-0 -c prometheus -- curl -X POST http://localhost:9090/-/reload
```


Después de ejecutar este comando, deberíamos ver en "Status-> Target" los pods y los servicios de la demo app. En caso de que no veas los servicios como "Activos", es posible que debas actualizar la pagina un par de veces.




## Paso 6: Mostrar algunas métricas de ejemplo en Grafana


La única parte que falta es mostrar las métricas recopiladas de la demoapp en Grafana. Solo necesitamos importar un tablero para ver algunas métricas. Importemos el siguiente JSON en Grafana

**El acrhivo se encuentra en la carpeta assets de este repositorio, */aseets/dashboard.json***


 Para utilizar la plantilla, seleccione en el navegador con Grafana el logotipo de Grafana (arriba a la izquierda) -> Tableros de instrumentos-> Importar .


Para poder realizar pruebas, podemos correr en varias terminales el siguiente comando:
```
for i in {0..1000}; do curl -s  http://restservice-demoapplication.openshift-jose2-5290c8c8e5797924dc1ad5d1b85b37c0-0001.us-south.containers.appdomain.cloud/hello-world; done
```

El resultado del dashboard despues de unos minutos:

<p align="center">
  <img src="/././images/gr12.png" width="800">
</p>



# Lab 2 -  Configurar alertas por email 

En esta lab, configuraremos OpenShift Prometheus para enviar alertas por email!. Además, configuraremos el dashboard de Grafana para mostrar algunas métricas básicas. Todos los componentes (Prometheus, NodeExporter y Grafana) se crearán en proyectos separados. La interfaz de usuario web de Prometheus y la interfaz de usuario de AlertManager se utilizarán solo para la configuración y las pruebas. 

```
$ oc adm new-project openshift-metrics-node-exporter 
$ oc project openshift-metrics-node-exporter
$ oc create -f https://raw.githubusercontent.com/openshift/origin/master/examples/prometheus/node-exporter.yaml -n openshift-metrics-node-exporter
$ oc adm policy add-scc-to-user -z prometheus-node-exporter -n openshift-metrics-node-exporter hostaccess
```

Para implementar Grafana, utilizaremos un proyecto ya existente 

```
$ git clone https://github.com/mrsiano/grafana-ocp
cd grafana-ocp
./setup-grafana.sh prometheus-ocp openshift-metrics true
```




