# Modernización de Aplicaciones 2.0 - Dojo / página 6

<p align="center">
  <img src="/././images/logoDojo.png" width="500">
</p>

* [← Volver al índice](/././README.md)


## Istio


En la sección anterior estuvimos viendo microservicios, en esta sección profundizaremos sobre una herramienta que nos facilita su implementación, Istio. 


<p align="center">
  <img src="/././images/istio01.png" width="250">
</p>




**Índice de la sección:**

  - [¿Qué es Istio?](#qué-es-istio)
  - [¿Por qué usar Istio?](#por-qué-usar-istio)
  - [Arquitectura](#arquitectura)
    - [Proxy Sidecard](#proxy-sidecard)
    - [Envoy](#envoy)
    - [Mixer](#mixer)
    - [Pilot](#pilot)
    - [Citadel](#citadel)
    - [Flujo de Solicitud](#flujo-de-solicitud)
  - [¿Qué aporta Istio a una arquitectura de microservicios?](#que-aporta-istio-a-una-arquitectura-de-microservicios)
    - [Beneficios en la gestión del táfico](#beneficios-en-la-gestión-del-tráfico)
    - [Registro de servicios (Registry Service)](#registro-de-servicios)
    - [Descubrimiento de servicios (Discovery Service)](#descubrimiento-de-servicios)
    - [Manejo de fallas](#manejo-de-fallas)
    - [Inyeccion de fallas](#inyección-de-fallas)
    - [Autenticación mutua TLS](#autenticación-mutua-tls)
  - [Apendice](#apendice)
    - [Hands-on](#hands-on)
      - [Lab 1 - Instalación de Istio](#hands-on)
      - [Lab 2 - En construcción](/hands-on)
    - [Material Extra](/material-extra)



## ¿Qué es Istio?


Istio es una implementación de plataforma de servicio para conectar, administrar y proteger microservicios. Istio proporciona una manera fácil de crear una red de servicios desplegados utilizando balanceo de carga, autenticación de servicio a servicio, monitoreo y más, sin requerir ningún cambio en el código de cada servicio. Usted agrega el soporte de Istio a los servicios al implementar un proxy sidecar especial en su entorno, el cual intercepta toda la comunicación de red entre microservicios y se configura y administra mediante la funcionalidad del plano de control de Istio.


El lanzamiento de Istio fue el resultado de la colaboración entre IBM, Google y Lyft para proporcionar administración de flujo de tráfico, cumplimiento de políticas de acceso y utilización de datos de telemetría entre microservicios. Todos estos se pueden lograr sin la necesidad de realizar algún cambio en el código de nuestra aplicación. De este modo, los desarrolladores pueden centrarse en la lógica empresarial e integrar rápidamente nuevas características.


Antes de combinar fuerzas, IBM, Google y Lyft habían estado abordando partes separadas pero complementarias del problema.


* El proyecto Amalgam8 de IBM es una malla de servicios unificada que proporciona una estructura de enrutamiento de tráfico con un plano de control programable para ayudar a los clientes internos y empresariales a realizar pruebas A/B, liberaciones de canarios y probar sistemáticamente la resistencia de los servicios contra fallas.


* El Servicio de control de Google proporciona una malla de servicios con un plano de control que se enfoca en hacer cumplir políticas como las ACL, los límites de velocidad y la autenticación, además de recopilar datos de telemetría de diversos servicios y proxies.


* Lyft desarrolló el proxy Envoy para ayudar en su migración a microservicios, lo que llevó a la compañía de tener una aplicación monolítica a un sistema en producción que abarcaba 10,000 o más máquinas virtuales que manejaban 100 o más microservicios.


## ¿Por qué usar Istio?


Istio aborda muchos de los desafíos que enfrentamos los desarrolladores y operadores a medida que las aplicaciones monolíticas hacen la transición hacia una arquitectura de microservicio distribuida. A medida que la malla de servicios (service mesh) crece en tamaño y complejidad, puede ser más difícil de entender y administrar.Sus requisitos pueden incluir descubrimiento, equilibrio de carga, recuperación de fallas, métricas y monitoreo y, a menudo, requisitos operacionales más complejos, como pruebas A/B, liberaciones de canarios, limitación de velocidad, control de acceso y autenticación de extremo a extremo.


Istio proporciona una solución completa para satisfacer los diversos requisitos de las aplicaciones de microservicio al proporcionar información de comportamiento y control operativo sobre la malla de servicios en su conjunto. Proporciona varias capacidades clave uniformemente a través de una red de servicios:


* **Gestión del tráfico:** Este servicio nos permite controlar el flujo de tráfico y las llamadas a la API entre servicios, nos permite hacer que las llamadas sean más confiables y que la red sea más robusta ante condiciones adversas.


* **Observabilidad:** Nos permite obtener una comprensión de las dependencias entre los servicios y la naturaleza y el flujo de tráfico entre ellos, lo que nos proporciona la capacidad de identificar rápidamente los problemas.


* **Aplicación de políticas:** Gracias a esto podemos aplicar políticas organizativas a la interacción entre servicios y asegurarnos de que las políticas de acceso se apliquen y los recursos se distribuyan equitativamente entre los consumidores. Los cambios en las políticas se realizan configurando la malla, **no** cambiando el código de la aplicación!!!


* **Identidad y seguridad del servicio:** Podemos proporcionar servicios en la malla con una identidad verificable y proporcionar la capacidad de proteger el tráfico del servicio a medida que fluye a través de redes con diversos grados de confiabilidad.


Además de estos, Istio está diseñado para que la extensibilidad satisface diversas necesidades de implementación:


* **Soporte de plataforma:** Istio puede ejecutarse en una variedad de entornos, incluidos los que abarcan la nube, locales, Kubernetes, Mesos, etc.


* **Integración y personalización:** El componente de aplicación de políticas se puede ampliar y personalizar para integrarlo con las soluciones existentes para ACL, registro, monitoreo, cuotas, auditoría y más.


## Arquitectura 


Una malla de servicio de Istio se divide lógicamente en un plano de datos y un plano de control.
**El plano de datos** se compone de un conjunto de proxys inteligentes (Envoy) desplegados como sidecars que miden y controlan todas las comunicaciones de red entre microservicios.
**El plano de control** es responsable de administrar y configurar los proxys para enrutar el tráfico y hacer cumplir las políticas en tiempo de ejecución.


El siguiente diagrama muestra los diferentes componentes en cada plano:




<p align="center">
  <img src="/././images/istio02.png" width="600">
</p>


### Proxy Sidecard


Una implementación de malla de servicios, como Istio, utiliza proxys que normalmente se implementan como sidecars en pods. Un proxy controla el acceso a otro objeto. Istio usa proxys entre servicios y clientes. Permite que la malla de servicios gestione las interacciones.
Un sidecar agrega comportamiento a un contenedor sin cambiarlo. En ese sentido, el sidecar y el servicio se comportan como una sola unidad mejorada. Las cápsulas alojan el sidecar y el servicio como una sola unidad.


<p align="center">
  <img src="/././images/istio03.png" width="250">
</p>


### Envoy


Istio utiliza una versión extendida del proxy Envoy, un proxy de alto rendimiento desarrollado en C++ para mediar todo el tráfico entrante y saliente para todos los servicios en la malla de servicios. Istio utiliza las numerosas funciones integradas de Envoy, como el descubrimiento dinámico de servicios, el balanceo de carga, la terminación TLS, el proxy HTTP/2 y gRPC, los interruptores de circuito, los controles de estado, metricas, etc.


Envoy se implementa como un sidecar para el servicio relevante en el mismo pod Kubernetes. Esto permite que Istio extraiga una gran cantidad de señales sobre el comportamiento del tráfico como atributos, que a su vez puede usar en el Mixer para hacer cumplir las decisiones de políticas y ser enviado a los sistemas de monitoreo para proporcionar información sobre el comportamiento de toda la malla. El modelo de sidecar proxy también le permite agregar capacidades de Istio a una implementación existente sin necesidad de realizar una rearquitectura o reescribir el código.


### Mixer


El mixer es un componente independiente de la plataforma responsable de aplicar el control de acceso y las políticas de uso en la malla de servicios y recopilar datos de telemetría desde el proxy Envoy y otros servicios.El mixer incluye un modelo de complemento flexible que le permite interactuar con una variedad de entornos host y back-end de infraestructura, los cuales abstraen el proxy Envoy y los servicios administrados por Istio de estos detalles.


### Pilot 


Pilot proporciona detección de servicios para los sidecars de Envoy, capacidades de administración de tráfico para enrutamiento inteligente (por ejemplo, pruebas A/B) y resiliencia (tiempos de espera, reintentos, interruptores de circuito y más). Convierte las reglas de enrutamiento de alto nivel que controlan el comportamiento del tráfico en configuraciones específicas de Envoy y las propaga a los sidecars en tiempo de ejecución. 


Tambien abstrae los mecanismos de descubrimiento de servicios específicos de la plataforma y los sintetiza en un formato estándar consumible por cualquier sidecar que se ajuste a las API del plano de datos de Envoy . Este acoplamiento suelto permite que Istio se ejecuta en múltiples entornos (por ejemplo, Kubernetes) mientras mantiene la misma interfaz de operador para la gestión del tráfico.


### Citadel


Citadel (anteriormente denominada Istio Auth) proporciona una sólida autenticación de servicio a servicio y a usuario final mediante TLS, con identidad incorporada y gestión de credenciales. Se puede usar para actualizar el tráfico no cifrado en la malla de servicios y brinda a los operadores la capacidad de aplicar políticas que se basan en la identidad del servicio en lugar de en los controles de red. Las futuras versiones de Istio agregarán control de acceso y auditoría detallados para controlar y monitorear quién accede a su servicio, API o recurso.


### Flujo de solicitud


El flujo que se muestra aquí es genérico para cualquier malla de servicios, pero se puede considerar para Istio.
 
<p align="center">
  <img src="/././images/istio4.png" width="550">
</p>
 
Como se indicó anteriormente, una malla de servicios se puede construir utilizando sidecars. Los sidecars manejan tanto el tráfico de ingreso como el de salida hacia un servicio. De manera similar, la malla Istio administra y controla la capacidad de los servicios y los usuarios para acceder a los servicios bajo su control.


El uso de sidecars permite incluir varias propiedades deseables en la malla de servicios:


* Las funciones comunes realizadas dentro del código de la aplicación se incluyen en el sidecar. Estos incluyen la telemetría, el rastreo distribuido y la terminación/iniciación de TLS.


* También se pueden incluir características útiles en el sidecar, como interruptores automáticos, límites de velocidad, enrutamiento inteligente para pruebas A/B.


## ¿Que aporta Istio a una arquitectura de microservicios?


Ahora que tenemos un poco de conocimiento sobre que es y como funciona (a grandes rasgos) Istio, Vamos a profundizar en algunas características que demuestran cómo la malla de servicios de Istio nos ayudan a administrar cientos o miles de microservicios.


### Beneficios en la gestión del tráfico.


El uso del modelo de gestión de tráfico de Istio desacopla el flujo de tráfico y la escala de la infraestructura, lo que permite a los operadores especificar a través de Pilot qué reglas quieren que siga el tráfico en lugar de qué pods/VM específicas deben recibir tráfico. Los proxies pilot e Envoy se encargan del resto. Así, por ejemplo, puede especificar a través de Pilot que desea que el 5% del tráfico para un servicio en particular vaya a una versión en particular dependiendo del contenido de la solicitud. 


**División del tráfico**


<p align="center">
  <img src="/././images/istio5.png" width="550">
</p>


**Direccion del trafico**


<p align="center">
  <img src="/././images/istio6.png" width="550">
</p>


Desacoplar el flujo de tráfico de la infraestructura como esta, permite a Istio proporcionar una variedad de funciones de administración de tráfico que se ejecutan fuera del código de la aplicación. Además del enrutamiento dinámico para pruebas A/B, despliegues graduales, Istio también maneja la recuperación de fallas. Todas estas capacidades se realizan mediante los sidecars y proxies de Envoy desplegados en la malla de servicios.




#### Registro de servicios


Istio asume la presencia de un registro de servicios para realizar un seguimiento de los pods o máquinas virtuales de un servicio en la aplicación. También supone que las nuevas instancias de un servicio se registran automáticamente con el registro de servicios y las instancias insalubres se eliminan automáticamente. Plataformas como Kubernetes y Mesos ya proporcionan dicha funcionalidad para aplicaciones basadas en contenedores. 


#### Descubrimiento de servicios


Pilot consume información del registro de servicios y proporciona una "interfaz de descubrimiento" de servicios independiente de la plataforma. Las instancias de Envoy en la malla realizan el descubrimiento del servicio y actualizan dinámicamente sus grupos de balanceo de carga en consecuencia.


#### Manejo de fallas


Envoy proporciona un conjunto de características de recuperación de fallos que pueden ser aprovechados por los servicios en una aplicación:


* Tiempos de espera
* Reintentos limitados con tiempo de espera
* Límites en el número de conexiones simultáneas y solicitudes a servicios ascendentes
* Verificaciones de estado activas (periódicas) en cada miembro del grupo de balanceo de carga
* Interruptores automáticos de "grano fino" que se aplican por instancia en el grupo de balanceo de carga


Estas funciones se pueden configurar dinámicamente en tiempo de ejecución a través de las reglas de administración de tráfico de Istio.


#### Inyección de fallas


Si bien el sidecar y proxy de Envoy proporciona una gran cantidad de mecanismos de recuperación de fallos a los servicios que se ejecutan en Istio, aún es imprescindible probar la capacidad de recuperación de fallos de extremo a extremo de la aplicación. Las políticas de recuperación de fallas mal configuradas (por ejemplo, tiempos de espera incompatibles o restrictivos en todas las llamadas de servicio) pueden resultar en una continua falta de disponibilidad de servicios críticos en la aplicación, lo que resulta en una experiencia de usuario pobre.


Istio habilita la inyección de fallas en la red en lugar de "matar" los pods, lo que retrasa o corrompe los paquetes en la capa TCP. La razón es que las fallas observadas por la capa de aplicación son las mismas, independientemente de las fallas a nivel de la capa de red, y que se pueden inyectar fallas más significativas en la capa de la aplicación (por ejemplo, códigos de error HTTP) para ejercer la resiliencia de una aplicación.


Los operadores pueden configurar las fallas que se inyectarán en las solicitudes que coincidan con criterios específicos.


Los operadores pueden restringir aún más el porcentaje de solicitudes que deben estar sujetas a fallas. Se pueden inyectar dos tipos de fallas: retrasos y abortos. Los retrasos son fallas de tiempo, que imitan una mayor latencia de la red o un servicio ascendente sobrecargado. Los abortos son fallas de bloqueo que imitan fallas en servicios ascendentes. Los abortos generalmente se manifiestan en forma de códigos de error HTTP o fallas de conexión TCP.


#### Autenticación mutua TLS


Como vimos anteriormente, con Citadel se puede mejorar la seguridad de los microservicios y su comunicación sin requerir cambios en el código de cada servicio. Citadel es responsable de:


* Proporcionar a cada servicio una identidad sólida que represente su función para permitir la interoperabilidad entre clusters.
* Asegurar la comunicación de servicio a servicio y la comunicación de usuario final a servicio
* Proporcionar un sistema de administración de claves para automatizar la generación, distribución, rotación y revocación de claves y certificados.




## Apendice


En esta sección encontrará material extra que puede utilizar para profundizar en el tema. Además están los links a los laboratorios de este dojo donde podrá poner en práctica lo visto en esta sección y aprender cosas nuevas. 


### Hands-On


- [Lab 1 - Instalación de Istio ---> Errores con el cluster free]()
- [Lab 2 - **En Construcción**]()


### Material Extra


El siguiente link es la documentación oficial de Istio, allí puede encontrar material para profundizar o consultar los conceptos vistos durante esta sección: https://istio.io/docs/


En el caso de que le interese, en el siguiente link encontrara un curso,  **Beyond the Basics: Istio and IBM Cloud Kubernetes Service** para fortalecer algunos conceptos, aunque todo lo que verá en este curso fue mencionado en esta sección: https://cognitiveclass.ai/badges/beyond-the-basics-istio-and-ibm-cloud-kubernetes-service/


Por ultimo, el siguiente libro también puede serle de ayuda. Es claro, corto y de lectura sencilla: https://corneliu.cl/docs/istio-mesh-for-microservices.pdf
(En este libro ya se habla de OpenShift! Una plataforma que más adelante veremos en este dojo)




---- Navegacion Dojo ----
