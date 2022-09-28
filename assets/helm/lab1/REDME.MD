# Modernización de Aplicaciones - Dojo / Lab 1  - Helm

<p align="center">
  <img src="/././images/logoprincipal.png" width="500">
</p>

* [← Volver al índice](/././README.md) 

## Introducción 

En este lab llevaremos a la práctica los conceptos de helm que vimos en la sección del dojo e incorporaremos nuevos.




## Paso 1: Clonar el codigo y generar la imagen 

El objetivo en este paso es: Clonar el repositorio de Github que contiene el código, crear una imagen Open Liberty de la aplicación y luego subirla a Container Registry de IBM Cloud Kubernetes

* 1 -Inicie sesión en su cuenta de Github

* 2 - En la barra de búsqueda en la parte superior izquierda, escriba app-modernization-plants-by-websphere-jee6

* 3 - Seleccione el repositorio IBMAppModernization\app-modernization-plants-by-websphere-jee6y y luego haga clic en el icono de Fork

* 4 - Haga clic en el botón Clonar o descargar y copie la URL HTTPS.

* 5 - Desde la terminal, clone el repositorio de Git con los siguientes comandos agregando la URL HTTPS desde su portapapeles

```
$ git clone [HTTPS URL for NEW REPO]
$ cd app-modernization-plants-by-websphere-jee6
```

* 6 - Construya el archivo .ear de la aplicación usando Maven usando el siguiente comando:

```
$ mvn package
```

* 7 - Inicie sesión en Container Registry y creó un repo con el siguiente comando:

```
$ ibmcloud  cr  namespace-add  <REPOSITORIO>
```





* 8 - Cree una imagen acoplable escribiendo los siguientes comandos (¡¡¡sin los comentarios!!!)



```
$docker build -t us.icr.io/REPOSITORIO/USUARIO/pbw-mariadb-web:1.0.0 .
```

* 9 - Suba la imagen a Container Registry de IBM:

```
docker push us.icr.io/REPOSITORIO/USUARIO/pbw-mariadb-web:1.0.0
```

## Paso 2: Revisar la estructura del Helm Chart

El objetivo en este paso es mirar la estructura del Helm Chaet para la aplicación Plants By WebSphere y comprender cómo está conformada. 

Deténgase y revise todos los archivos que se encuentran el repositorio ***app-modernization-plants-by-websphere-jee6*** Del cual hizo un fork.

Este es un chart de Helm con charts secundarios para la aplicación web y MariaDB como bd de la aplicación. Como ya hay un chart publicado para MariaDB, aparece como un chart secundario obligatorio en el archivo require.yaml y no teneos que crear un chart para MariaDB.

## Paso 3: Agregar el chart al repositorio local de Helm



* 1 - Desde su la terminal ejecutar:

```
# MariaDB
helm repo add ibmcom https://raw.githubusercontent.com/IBM/charts/master/repo/stable
helm dependency update chart/pbw-liberty-mariadb    
helm package chart/pbw-liberty-mariadb -d ~/.helm/repository/local
```

## Paso 4: Configure Helm para brindar el repositorio a través de HTTP

* 1: En la terminal, escriba el siguiente comando para iniciar el repositorio Helm de prueba local sustituyendo [PORT_NUMBER]. Use cualquier número de puerto libre.

```
$ helm serve --address 127.0.0.1:[PORT_NUMBER] &
```

* 2: En su terminal, escriba el siguiente comando. Verifique que se devuelvan los contenidos de index.yaml y que contenga el archivo gráfico que acaba de agregar. Use el mismo número de puerto que usó anteriormente.

```
curl http://127.0.0.1:[PORT_NUMBER]/charts/index.yaml
```

## Paso 5: Desplegar la aplicación desde el repositorio de Helm

En la ventana de su terminal, escriba el siguiente comando, sustituyendo su usuario por [YOUR_USERNAME. Nota: los charts de helm se pueden implementar varias veces, pero cada implementación debe tener un nombre único

```
$ helm install --name pbw-liberty-mariadb --set liberty.image.registry=us.icr.io \
--set liberty.image.namespace=NAMESPACE/USUARIO local/pbw-liberty-mariadb
```

## Paso 6: Iniciar la aplicación desplegada

Usaremos los siguientes comandos para obtener el gateway y el número de puerto de tu versión de Helm desplegada.

* 1 - Ejecute el siguiente comando para obtener el número de puerto de su aplicación

```
$ echo `kubectl --namespace default get service pbw-liberty-mariadb-liberty -o jsonpath='{.spec.ports[0].nodePort}'`
```

* 2 - Ejecute el siguiente comando para obtener la dirección IP externa del nodo

```
ibmcloud ks trabajadores $ USERNAME -cluster | grep -v ' ^ * '  | egrep -v " (ID | OK) "  | awk ' {print $ 2;} '  | cabeza -n1
```

* 3 - En la barra de direcciones de su navegador ingrese la URL de su aplicación. La URL será la dirección IP externa del nodo del clúster seguido de dos puntos y luego el número de puerto de su aplicación. Por ejemplo, si su IP externa es 169.61.73.182 y el puerto es 30961, la URL seráhttp://169.61.73.182:30961



## Cierre del lab

A lo largo de este lab hemos visto cómo implementar nuestro propio chart de Helm. Como introducción considero que esta bueno, pero ahora hay que ir por más. Intente agarrar alguna aplicación de labs anteriores y crear un chart de helm que permite desplegarla. 

[← Volver al índice](/././README.md)

[← Volver a Sección(Helm)](/pages/7/#helm)

