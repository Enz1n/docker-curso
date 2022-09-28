# Modernización de Aplicaciones 2.0 - Dojo / página 4

<p align="center">
  <img src="/././images/logoDojo.png" width="500">
</p>

* [← Volver al índice](/././README.md)

## Microservicios

En las secciones anteriores estuvimos viendo que es Kubernetes y como nos sirve para orquestar contenedores y lo sencillo que es instalar aplicaciones y distribuirlas utilizando Helm. En esta sección veremos una introducción a los microservicios.

<p align="center">
  <img src="/././images/micro-logo.png" width="300">
</p>

**Índice de la sección:**

- [Modernización de Aplicaciones 2.0 - Dojo / página 4](#modernización-de-aplicaciones-20---dojo--página-4)
  - [Microservicios](#microservicios)
  - [¿Qué son los microservicios?](#qué-son-los-microservicios)
  - [Arquitectura de microservicios](#arquitectura-de-microservicios)
    - [Arquitectura Monolítica vs Arquitectura de Microservicios.](#arquitectura-monolítica-vs-arquitectura-de-microservicios)
    - [Microservicios y IBM Cloud Kubernetes Services.](#microservicios-y-ibm-cloud-kubernetes-services)
      - [Ejemplo de una Arquitectura](#ejemplo-de-una-arquitectura)
    - [Jerarquía y subtipos de microservicios](#jerarquía-y-subtipos-de-microservicios)
    - [Dependencia de microservicios: Typical](#dependencia-de-microservicios-typical)
    - [Dependencia de microservicios: Death Star](#dependencia-de-microservicios-death-star)
  - [Comunicación entre servicios](#comunicación-entre-servicios)
    - [Comunicación síncrona frente a asíncrona.](#comunicación-síncrona-frente-a-asíncrona)
  - [Malla de Servicios (Service Mesh)](#malla-de-servicios-service-mesh)
    - [¿Que es una Malla de Servicios?](#que-es-una-malla-de-servicios)
    - [Registro de servicios (Service Registry)](#registro-de-servicios-service-registry)
    - [Servicio de Descubrimiento (Service Discovery) y Servicio de Proxy.](#servicio-de-descubrimiento-service-discovery-y-servicio-de-proxy)
      - [Descubrimiento del lado del cliente](#descubrimiento-del-lado-del-cliente)
      - [Descubrimiento del lado del servidor](#descubrimiento-del-lado-del-servidor)
    - [Pruebas automatizadas](#pruebas-automatizadas)
    - [Interruptores de Circuito (Circuit Breaker)](#interruptores-de-circuito-circuit-breaker)
  - [Apendice](#apendice)
    - [Hands-on](#hands-on)
      - [Lab 1 - Modernizando una aplicación monolítica](#lab-1---modernizando-una-aplicación-monolítica)
    - [Material Extra](#material-extra)

## ¿Qué son los microservicios?

Los microservicios son un estilo de arquitectura de aplicación que divide una aplicación en componentes, donde cada componente es una aplicación completa, pero en miniatura que se enfoca en producir una tarea única. Se aplica el principio de SRP (principio de una única responsabilidad).

Un diseño de microservicios implementa tareas de principio a fin: desde la interfaz de usuario hasta a la base de datos, o al menos desde la API del servicio a la base de datos para que diferentes interfaces de usuario y aplicaciones de clientes puedan reutilizar la misma funcionalidad. 

Cada microservicio tiene una interfaz y dependencias bien definidas (por ejemplo, a otros microservicios y recursos externos), de modo que el microservicio puede ejecutarse de manera bastante independiente, y el equipo puede desarrollarlo puede trabajar de una forma muy independiente al resto de los equipos que trabajan sobre el mismo sistema.

Los microservicios hacen que los desarrolladores sean más eficientes ya que aceleran la entrega al minimizar la comunicación y la coordinación entre las personas, y reducir el alcance y riesgo de cambios. Esto gracias a que un pequeño equipo es responsable por el desarrollo de un microservicio.

## Arquitectura de microservicios

El objetivo de una arquitectura de microservicios es desacoplar completamente los componentes de las aplicaciones entre sí, de modo que se puedan mantener, escalar y más.

Estos principios comprueban si la arquitectura de una aplicación X está basada en microservicios. 

* Las grandes arquitecturas monolíticas se dividen en muchos pequeños servicios:
  - Cada servicio se ejecuta en su propio proceso.
  - Un servicio por contenedor.
* Los servicios están optimizados para una sola función:
  - Solo hay una función de negocio por servicio.
  - El principio de responsabilidad única
* La comunicación es a través de API REST y los agentes de mensajes:
  - Evite el acoplamiento introducido por la comunicación a través de una base de datos.
* La integración continua y la implementación continua (CI/CD) se definen por servicio:
  - Los servicios evolucionan a diferentes ritmos.
  - El sistema evoluciona, pero establece principios arquitectónicos para guiar esa evolución.
* Las decisiones de alta disponibilidad (HA) y de agrupamiento se definen por servicio:
  - Una política de tamaño o escala no es apropiada para todos.
  - No todos los servicios necesitan escalar; otros requieren autoescalado hasta grandes números.

### Arquitectura Monolítica vs Arquitectura de Microservicios.

El siguiente diagrama muestra la arquitectura de una aplicación monolítica (a la izquierda) en comparación con cómo dicha aplicación puede convertirse en un conjunto de microservicios (a la derecha).

<p align="center">
  <img src="/././images/micro-1.png" width="750">
</p>

### Microservicios y IBM Cloud Kubernetes Services.

Pasar de una arquitectura monolítica a una arquitectura de microservicio no es tan difícil como parece. Por ejemplo, puede extender la funcionalidad de una aplicación monolítica agregando servicios en la nube. También puede comenzar a utilizar herramientas operativas, como Grafana para su monitoreo, Istio para una malla de servicios y App ID de IBM Cloud para la autenticación de usuarios.

Entonces, ¿cómo funciona esta evolución? Con IBM Cloud Kubernetes Service comience por empaquetar su aplicación (incluso monolítica) en contenedores, por ejemplo:

*  Extraiga una aplicación Java existente y prepárela para WebSphere Liberty en la nube. 
*  Configure Liberty y la base de datos como microservicios en Kubernetes. 
*  Contenga la aplicación como está y depliguela en un clúster. 
*  Agregue un servicio en la nube, como AI, Cloud Object Storage, App ID, etc. 
*  Utilice las herramientas de operación en la nube para monitorear la aplicación.

Considere también estos otros aspectos para la evolución de su aplicación:

* Herramientas de CI/CD nativas para implementaciones rápidas: orquestación de contenedores con Kubernetes, pruebas de Istio canary y despliegues com Helm.

* Infraestructura de escalamiento flexible: cuando un solo centro de datos y una sola instancia de aplicación no son suficientes

* Seguridad incorporada: IBM Vulnerability Advisor + X-Force Exchange para obtener información; tráfico de red estrechamente controlado; aislamiento para nodo maestro, nodos trabajadores; y certificados TLS en IBM Certificate Manager

#### Ejemplo de una Arquitectura

En el siguiente diagrama vemos la arquitectura de una app de una aerolínea, podemos ver cómo se puede descomponer por funcionalidades una aplicación monolítica para usar microservicios individuales. 

<p align="center">
  <img src="/././images/micro-2.png" width="500">
</p>

Cada servicio puede verse como una app monolítica mucho más pequeña

<p align="center">
  <img src="/././images/micro-3.png" width="500">
</p>

### Jerarquía y subtipos de microservicios

Todos los servicios de negocio (Business Services, en la imagen anterior) son prácticamente iguales, pero hay especializaciones de subtipos dependiendo de los despachadores. Un despachador (en inglés dispatcher), es la parte de una aplicación encargada de ejecutar un proceso en el servidor en un entorno cliente/servidor. Cuando hablamos de jerarquía, no nos referimos a una jerarquía de clase o herencia, sino que de especializacion. 

La cantidad de subtipos de servicios que se necesitan implementar en nuestra aplicación va a depender de qué tipos de clientes admite y qué tan especializados son esos clientes. Es posible que un despachador móvil sea compatible con todos los dispositivos o quizás necesite uno para Android. Un despachador de iOS puede ser insuficiente, quizás puede que necesite uno separado para iPhone y otro para iPad. También es posible que un distribuidor web sea suficiente o que necesite uno para los navegadores HTML5 y otro para los más antiguos.

<p align="center">
  <img src="/././images/micro-4.png" width="500">
</p>

### Dependencia de microservicios: Typical

Los servicios de negocio pueden depender de otros servicios, solo hay asegúrese de que cada servicio sea una tarea comercial completa. Resista la tentación de separar la capa de acceso a la base de datos en su propio servicio.
Si cada servicio es una tarea de negocio completa, contar con la colaboración de los servicios de negocios está bien y puede ser útil.

Observe las dependencias que se muestran a continuación. Este es un gráfico dirigido acíclico, lo que significa que, si el servicio A depende del servicio B, entonces B no depende también de A. Estos tipos de dependencias son más fáciles de administrar.

<p align="center">
  <img src="/././images/micro-6.png" width="500">
</p>

### Dependencia de microservicios: Death Star

Esta es una "estrella de la muerte" de dependencias entre microservicios, donde todos dependen de todos los demás. Busque microservicios death star en [Internet](https://www.google.com/search?biw=1745&bih=916&tbm=isch&sa=1&ei=4AA1XfWWAb2w5OUP1dqDkAM&q=microservices+death+star&oq=microser+death+star&gs_l=img.3.0.0i7i30j0i8i7i30l2.1610.2409..4087...0.0..0.90.651.8......0....1..gws-wiz-img.......0i7i5i30.O6OcOvzJ9yk#imgrc=TGFxKMEOLVPQFM:), y encontrará imágenes y descripciones de aplicaciones con cientos de servicios, todo en un círculo con líneas que los conectan entre sí.

¿Cómo se implementa tal arquitectura? ¿Cuál implementas primero? Si tiene que cambiar la API en una (una nueva versión), ¿cuáles otras necesitan cambiar? ¡Mejor trate de evitar esta topología de Death Star en su arquitectura!

<p align="center">
  <img src="/././images/micro-7.png" width="500">
</p>

## Comunicación entre servicios

Ahora bien, si tenemos muchos microservicios, ¿cómo los conectamos entre sí?

La comunicación entre los servicios debe ser neutral en cuanto al lenguaje. Se pueden implementar diferentes microservicios en diferentes lenguajes (ahora o en el futuro), así que no se trabe en una tecnología de integración específica del lenguaje, por ejemplo, sockets de Java.

El estándar en estos días es JSON/REST. Para la integración asíncrona, siga un estándar abierto y compatible con la nube como IBM Event Streams con Apache Kafka. Otros ejemplos de protocolos asíncronos son AMQP, MQ Light, RabbitMQ.

Resista la tentación de utilizar otros enfoques como XML o serialización. Eso lleva a una explosión combinatoria de protocolos que cada proveedor de API y consumidor deben admitir.

Cómo ya hemos visto en el dojo, con IBM Cloud Kubernetes Service la comunicación del servicio de microservicio se realiza a través de redes de área local virtuales (VLAN). De forma predeterminada, el servicio IBM Cloud Kubernetes proporciona una comunicación segura y fluida que puede cambiar para las comunicaciones entrantes y salientes desde los nodos trabajadores. 
 
Un servicio Kubernetes es un grupo de pods y proporciona conexiones de red a estos pods para otros microservicios en el clúster sin exponer la dirección IP privada real de cada pod. Del mismo modo, puede optar por mantener los microservicios privados y utilizar las funciones de seguridad integradas.

### Comunicación síncrona frente a asíncrona.

Usando REST sincrónico suele ser más fácil hacer que algo funcione, así que es recomendable para comenzar. Al ser sincrónico, todo el bucle (solicitante, proveedor, mensajes) debe seguir funcionando durante toda la vida de la invocación.

<p align="center">
  <img src="/././images/micro-8.png" width="500">
</p>

Luego, considere la posibilidad de convertir estratégicamente algunos puntos a asíncronos, ya sea por la naturaleza de la solicitud (ejecución prolongada, ejecuciones en segundo plano) o para hacer que la integración sea más confiable y robusta. 

Al ser asíncrono, la invocación se divide en 3-4 partes. Si alguno falla, el sistema puede reintentar automáticamente. Y debido a que el solicitante no tiene estado, la instancia que recibe la respuesta ni siquiera necesita ser la instancia que envió la solicitud.

<p align="center">
  <img src="/././images/micro-9.png" width="500">
</p>

La comunicación asíncrona puede hacer que los microservicios sean más robustos ya que:
* El solicitante no tiene que bloquearse mientras se ejecuta.
* Una instancia diferente de solicitante puede manejar la respuesta.
* El sistema de mensajería retiene la acción y el resultado.

## Malla de Servicios (Service Mesh)

Ahora veremos lo que son las mallas de servicios (Service Mesh) Estas pueden ayudarnos con las implementaciones de microservicios.

### ¿Que es una Malla de Servicios?

Una malla de servicios puede ayudarnos a resolver los problemas que surgen cuando estamos implementando una aplicación de microservicios. Aquí hay una lista con algunas funciones que proporciona una malla de servicios.

* **Registro de servicios:** se utiliza para realizar un seguimiento de la ubicación y el estado de los servicios, por lo que un servicio de solicitud puede dirigirse a un servicio de un proveedor de manera confiable.

* **Descubrimiento de servicios:** utiliza la lista de servicios disponibles y las instancias de esos servicios en el registro de servicios, y dirige las solicitudes a la instancia adecuada en función del equilibrio de carga puro o cualquier otra regla que se haya configurado.

* **Pruebas automatizadas:** esto nos permite simular la falla o la indisponibilidad temporal de uno o más servicios en nuestra aplicación y asegurarnos de que las funciones de resistencia que diseñamos funcionen para mantener la aplicación funcional. Dos funciones que pueden ayudar a proporcionar esta resistencia:
  - **Interruptores de circuito**
  - **Bulkhead**
 
A continuación veamos un poco más sobre cada una de estas funciones.

### Registro de servicios (Service Registry)

El registro de servicios es un simple almacén de datos clave-valor, que mantiene una lista actual de todas las instancias de servicio en funcionamiento y sus ubicaciones.

A medida que se inicia una instancia de servicio, se registra en el registro de servicios, ya sea a través de su propio código o a través de una función de registro de terceros. Por ejemplo, las instancias de servicio podrían implementarse en contenedores desplegados en un cluster de Kubernetes, en este caso Kubernetes proporciona un servicio de registro de terceros para esos contenedores.

Después de que una instancia de servicio se haya registrado, el registro de servicios usará un mecanismo de "latido", ya sea entrante o saliente, para mantener su lista de instancias actualizada.

<p align="center">
  <img src="/././images/micro-10.png" width="350">
</p>

### Servicio de Descubrimiento (Service Discovery) y Servicio de Proxy.

Las funciones de registro y descubrimiento de servicios son las que permiten a cada servicio encontrar y conectarse con los otros servicios que necesitan para hacer su trabajo. 

Dependiendo del diseño general de la aplicación, cada instancia de servicio puede ser un cliente de servicio, solicitar datos o funciones de otros servicios, un proveedor de servicios, proporcionar datos o funciones a clientes de servicio, o ambos. Cuando un servicio necesita otro servicio, utiliza la función de descubrimiento de servicio para encontrar una instancia de ese servicio.

<p align="center">
  <img src="/././images/micro-11.png" width="400">
</p>

Hay dos tipos básicos de funciones de descubrimiento de servicios que una malla determinada podría emplear, del lado del cliente o del lado del servidor. El gráfico que se muestra arriba ilustra los dos tipos de descubrimiento de servicios y varias formas de lograr cada uno.

#### Descubrimiento del lado del cliente

El primer tipo de descubrimiento de servicio es el descubrimiento del lado del cliente. El servicio solicitante o el cliente del servicio, trabaja directamente con el registro de servicios para determinar todas las instancias disponibles del servicio que necesita y también es responsable de decidir cuál de esas instancias de servicio usará para su solicitud.

Cuando utiliza el descubrimiento del lado del cliente, el servicio del cliente es responsable de tomar la decisión y de seguir la decisión de comunicarse con la instancia de servicio apropiada. El cliente utilizará algún código propio para que eso suceda.

En la siguiente ilustración, la instancia de servicio A, ha descubierto en el registro de servicios que hay tres instancias del servicio que necesita. Usando la regla que ha elegido, el servicio solicitante ha decidido comunicarse con la instancia de servicio B a la derecha para solicitarla.

<p align="center">
  <img src="/././images/micro-12.png" width="500">
</p>

#### Descubrimiento del lado del servidor

El segundo tipo de descubrimiento de servicios se denomina descubrimiento del lado del servidor. Este método introduce otro componente, etiquetado como el equilibrador de carga en la siguiente ilustración. Es comúnmente llamado el proxy de servicio.

La tarea de consultar el registro de servicios y dirigir una solicitud a la instancia de servicio adecuada se delega en el equilibrador de carga. En algunas implementaciones, como **Istio**, el equilibrador de carga (load balancer) se puede programar con reglas específicas para controlar su elección de instancia de servicio.

<p align="center">
  <img src="/././images/micro-13.png" width="500">
</p>

### Pruebas automatizadas

Una de las premisas de las arquitecturas nativas y de microservicios en la nube es que deben diseñarse para ejecutarse en sistemas poco confiables. Debería esperar que los componentes fallen.

Uno de los puntos básicos de diseño de esta estrategia es tener siempre más de una instancia de un servicio determinado para que su aplicación pueda sobrevivir al fallo de una instancia determinada y continúe funcionando mientras se reinicia esa instancia (Como vimos en la sección de kubernetes).

Las herramientas de prueba automatizadas son útiles para asegurar la resistencia de su aplicación de microservicios. Los ejemplos de estas herramientas incluyen Netflix Chaos Monkey, Netflix Simian Army y las funciones integradas en Istio.

Con IBM Cloud Kubernetes Service, puede administrar la implementación de sus cambios de manera automatizada y controlada. ¡¡¡Si sus pruebas de implementación no van bien, puede revertir su implementación a la revisión anterior!!!

Los siguientes pasos muestran cómo utilizar Kubernetes para gestionar fácilmente los problemas de aplicaciones durante las pruebas:

1. Antes de comenzar, cree un despliegue.
2. Despliegue un cambio a un microservicio. Por ejemplo, es posible que desee cambiar la imagen que utilizó en su implementación inicial. Cuando implementa microservicios con Kubernetes, el cambio se aplica de inmediato y se registra en el historial de despliegue.
3. Pruebe el estado de su implementación con sus herramientas operativas y de prueba.
4. Vea el historial de despliegue e identifique el número de revisión de su último despliegue.
5. Cuando esté listo, vuelva a la versión anterior o especifique una revisión.

### Interruptores de Circuito (Circuit Breaker)

El interruptor de circuito es una herramienta de programación incluida en Istio y Netflix Hystrix. Los interruptores automáticos funcionan para evitar retrasos en la respuesta de una dependencia del servicio causada por una falla o latencia, para que las demoras no causen latencias y fallas más amplias en toda la aplicación.

El interruptor le permite establecer umbrales de tiempo de espera y umbrales de falla que le permiten fallar rápidamente en tales situaciones con la expectativa de construir un plan alternativo en su aplicación.

Puede definir números de umbral para las solicitudes de tiempo de espera y tiempo de espera agotado. A medida que realiza llamadas de servicio a través de Hystrix y se superan los umbrales, los disparos del circuito y las solicitudes subsiguientes fallan inmediatamente. Esto le permite seguir el plan alternativo que debería haber incorporado a su aplicación.

<p align="center">
  <img src="/././images/micro-14.png" width="500">
</p>

## Apendice 

En esta sección encontrará material extra que puede utilizar para profundizar en el tema. Además, están los links a los laboratorios de este dojo donde podrá poner en práctica lo visto en esta sección y aprender cosas nuevas. 

### Hands-on

#### Lab 1 - Modernizando una aplicación monolítica

Para esta sección se armó de un laboratorio donde vamos a migrar una app monolitica a microservicios. 
La app consiste en un registro de jugadores de fútbol incluyendo sus posiciones y una foto. Actualmente la app almacena la información en una base de datos local y las imágenes en una carpeta del servidor.
Vamos a crear una imagen de la aplicación y vamos a desplegarla en un cluster de Kubernetes en IBM Cloud. También desplegaremos en ese cluster una base de datos MySQL para que pueda utilizar nuestra aplicación. Por último, haremos uso de IBM Cloud Object Storage para almacenar las imágenes de los jugadores. 

[Clic aquí para ir al lab](https://github.ibm.com/IBMInnovationLabUy/dojo-modernizacion-aplicaciones/tree/master/assets/microservicios)

### Material Extra

- [IBM Docs](https://www.ibm.com/es-es/cloud/learn/microservices)
  
