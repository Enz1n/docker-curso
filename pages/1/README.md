# Modernización de Aplicaciones 2.0 - Dojo / página 1

<p align="center">
  <img src="/././images/logoprincipal.png" width="500">
</p>

* [← Volver al índice](/././README.md)

## ¿Qué es un contenedor?
Un contenedor es una unidad de software ejecutable que puede correrse en cualquier lugar. Se virtualiza a nivel del sistema operativo. Es pequeño, rápido y portable. A diferencia de una máquina virtual, no incluye un sistema operativo huésped en cada instancia, sino que aprovecha el sistema operativo hospedador.

Es una estandarización a cómo empaquetamos y entregamos software. Además, habilita una arquitectura de microservicios para mejorar y facilitar escalamiento.

En la siguiente sección podremos ver la diferencia entre una máquina virtual y un contenedor:

<p align="center">
  <img src="/././images/vms-vs-containers.jpg" width="500">
</p>

### ¿Máquinas virtuales o contenedores?

Antes de comenzar a experimentar con contenedores, es fundamental tener ciertos conceptos claros. Por este motivo en esta sección veremos algunos fundamentos teóricos sobre máquinas virtuales y contenedores, para concluir analizando sus diferencias. Luego, en la siguiente sección del dojo comenzaremos a trabajar con Docker. 

<p align="center">
  <img src="/././images/duda2.png" width="400">
</p>

### -> Máquinas Virtuales, ¿Qué son?


A grandes rasgos podemos decir que una máquina virtual es un entorno basado en software diseñado con el fin de simular un entorno basado en hardware. Cuando hablamos de aplicaciones convencionales, nos referimos a todas aquellas diseñadas para ser administradas por un sistema operativo y ejecutadas por un conjunto de núcleos del procesador. Estas aplicaciones son capaces de ejecutarse dentro de una máquina virtual sin que se tenga que realizar una re-arquitectura de las mismas. El hipervisor es un componente de software que juega un rol clave dentro de una máquina virtual ya que actúa como un agente entre el entorno de dicha máquina virtual y el hardware subyacente, proporcionando una capa de abstracción necesaria.
El uso de las máquinas virtuales es sumamente amplio ya que, al permitir emular casi cualquier sistema operativo estándar, no hay limitación en cuanto a los programas que se pueden ejecutar.



<p align="center">
  <img src="/././images/vm.png" width="160">
</p>


### -> Contenedores


Cuando hablamos de contenedores pensamos en una tecnología nueva y reciente, pero lo cierto es que los contenedores existen desde hace mucho tiempo, por ejemplo, los LXC fueron contenedores de Linux que se introdujeron hace más de una década. Dada su dificultad de usar y que estaban limitados a desarrolladores Linux, LXC no tuvo mucho éxito. El uso de contenedores dio un rotundo giro con la llegada de **Docker**, una plataforma de contenedores de código abierto que hace que esta tecnología sea fácil de usar. 
En los contenedores se almacenan aplicaciones y sus dependencias con el fin de desacoplarlas del sistema operativo, esto permite que la aplicación alojada en un contenedor pueda ser transportada y ejecutada independientemente del SO. Para lograrlo, los contenedores suelen contener aplicaciones y todas sus dependencias (como bibliotecas y marcos), en definitiva, un contenedor está compuesto por:

* Una aplicación
* Todas sus dependencias, librerías y demás archivos binarios
* Archivos de configuración que son necesarios para ejecutarlo

Es importante destacar que dentro de un mismo host pueden ser ejecutados *n* cantidad de contenedores, permitiendo, por ejemplo, tener un SO Linux completamente limpio y mínimo, y ejecutar todo lo demás en contenedores. 



<p align="center">
  <img src="/././images/contenedores-barco.png" width="500">
</p>

Así como hace más de 60 años los contenedores marítimos revolucionaron la industria de importación y exportación al hacer que la carga de envío sea estandarizada (de allí la analogía), los contenedores (informáticos) también revolucionaron el despliegue de aplicaciones al permitir un tipo de aislamiento sin la sobrecarga del hipervisor. Podemos concluir en que un contenedor es un entorno aislado donde se pueden ejecutar uno o más procesos, centrándose en el aislamiento y la contención de dicho proceso en vez de emular una máquina física por completo.


### ¿Qué es Docker? 


<p align="center">
  <img src="/././images/docker2.png" width="150">
</p>

Ahora que vimos el concepto de contenedores, podemos comenzar a ver un poco de Docker. 
Como se mencionó más arriba, Docker es una plataforma de contenedores de código abierto que permite crear y ejecutar contenedores ligeros y portables independientemente del sistema operativo del host en donde se esté trabajando. 

Docker ofrece un modelo de implementación basado en imágenes. Estas imágenes se utilizan para crear los contenedores, y nunca cambian. Se puede definir a las imágenes como un paquete ejecutable y liviano que cuenta con todo lo necesario para la ejecución del software (código, librerías, variables de ambiente,etc). Debido a la popularidad de docker hay muchas imágenes públicas que podemos utilizar para nuestro despliegue, más adelante entraremos en profundidad en este tema. 
Por otro lado, tenemos el dockerfile, un archivo de configuración que permite definir todo lo relacionado al ambiente dentro de un contenedor. En la siguiente sección del dojo veremos más en profundidad ambos conceptos.

Ahora bien, ¿Quién se ve beneficiado con el uso de Docker? ¡Todos! desde desarrolladores hasta usuarios. Esto debido a la independencia del software frente al host en el cual se quiere ejecutar y frente a los entornos donde se ejecuta. Por ejemplo, yo como desarrollador si utilizo docker no tengo que preocuparme del entorno en el cual se va a ejecutar mi aplicación ni del proceso de despliegue, solo debo centrar mi atención en el desarrollo del mismo.

En el transcurso del dojo abordaremos más sobre cada aspecto de docker y aprenderemos cómo utilizarlo. 


### Comparación entre Docker y Máquinas Virtuales 

<p align="center">
  <img src="/././images/testVS.png" width="350">
</p>

Hasta aquí ya sabemos que son las máquinas virtuales, los contenedores y Docker. ¡¡Estamos listos para compararlos!!

Primero que nada, tanto Docker como las Máquinas Virtuales tienen beneficios similares de aislamiento de recursos y asignación, pero funcionan de manera diferente. Los contenedores virtualizan el sistema operativo en lugar del hardware, lo que los vuelve mucho más portátiles y eficientes. 

Veamos un ejemplo, un servidor físico que ejecuta 3 VM, tendrá un hipervisor y tres sistemas operativos separaros ejecutándose sobre el mismo. Por el contrario, un servidor que ejecuta 3 aplicaciones en contenedores ejecutará un solo sistema operativo y cada contenedor compartirá el núcleo del mismo con el resto de los contenedores. Aquellas secciones del SO compartidas son únicamente "de lectura" mientras que cada contenedor tiene su propio despliegue. Este es uno de los motivos fundamentales para afirmar que los contenedores son mucho más ligeros y utilizan menos recursos que las máquinas virtuales. 

<p align="center">
  <img src="/././images/representacion-container-vs-vm.png" width="500">
</p>

En la representación de arriba, también podemos ver claramente la diferencia entre ejecutar una app en un contenedor y en una máquina virtual. Lo primero a destacar es que en el segundo caso no se requiere la presencia del hipervisor, otro aspecto favorable de los contenedores es que todos los recursos, como las librerías, pueden ser utilizados por más de un contenedor en simultáneo. 


### Ventajas de los contenedores

Los contenedores además de proporcionar muchas de las ventajas de las máquinas virtuales, cuentan con características que los hacen destacar.  Por ejemplo, son ligeros en términos de recursos de procesamiento (en comparación con las VM) y automatizan tareas repetitivas de configuración del entorno de desarrollo. 
En la etapa de desarrollo, el uso de docker permite a los desarrolladores centrar su atención exclusivamente en el código. Si dos componentes utilizan versiones diferente del tiempo de ejecución Java, cada componente se puede colocar en un contenedor separado con la versión que corresponda y los contenedores también se pueden colocar en capas sin conflicto para trabajar en simultáneo. En promedio, los desarrolladores que usan Docker envían software con una frecuencia **siete veces mayor!**


Listamos algunos de sus beneficios:


* Los contenedores son:
  - Ágiles
  - Pequeños
  - Portables 
* Beneficios desde el punto de vista de los desarrolladores:
  - Proporcionan un entorno de ejecución limpio, seguro y portable. 
  - Se desarrolla la aplicación solo una vez 
  - Permite ejecutar cada aplicación en su propio contenedor aislado, para que pueda ejecutar varias versiones de bibliotecas y otras dependencias 
  - Automatización de pruebas, integración y empaquetamiento. 
  - Reduce o elimina las preocupaciones sobre la compatibilidad en diferentes plataformas.
* Beneficios desde el punto de vista de las operaciones:
  - Hace que todo el ciclo de vida sea más eficiente, consistente y repetible 
  - Aumenta la calidad del código que producen los desarrolladores. 
  - Elimina las inconsistencias entre los entornos de desarrollo, prueba y producción. 
  - Apoya la segregación de funciones. 
  - Mejora significativamente la velocidad y confiabilidad de la implementación e integración continua
  
  
* [← Volver al índice](/././README.md)
* [→ Siguiente Sección (Docker)](/pages/2#docker)

