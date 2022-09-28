# Modernización de Aplicaciones / Lab 2 OpenShift




<p align="center">
  <img src="/././images/logoDojo.png" width="500">
</p>


## Agregar dependencias y una Base de datos a nuestra Aplicación


En este laboratorio veremos cómo agregarle dependencias y una base de datos a una aplicación que estará desplegada en nuestro cluster de OpenShift. En el laboratorio anterior implementamos una aplicación muy simple, en este utilizaremos una aplicación un poco más compleja para lo cual utilizaremos un ejemplo del libro "OpenShift for Developer", que consiste en una aplicación que aleatoriamente muestra insultos de Elizabeth de Shakespeare. El insulto consiste en dos adjetivos y un sustantivo y se verá más o menos así:


```html
Thou art a Mewling Motley-minded Vassal!
```
Comencemos! 


### Aplicación Base


El código de la aplicación base que utilizaremos se encuentra en la carpeta */code* de este laboratorio. Lo que haremos será subirlo a un repositorio tal cual hicimos en la laboratorio anterior. 


Primero creamos un nuevo proyecto en donde trabajar con esta aplicación. En el laboratorio anterior ya vimos como hacerlo desde la consola web, ahora hagámoslo desde la terminal.


```
$ oc new-project lab-dojo --display-name="labs"
```


Una vez creado el proyecto ejecutemos el siguiente comando para trabajar sobre este nuevo proyecto


```
$ oc project lab-dojo
```


Ahora ejecutemos el siguiente comando con el cual crearemos un wildfly con el código de nuestro repositorio y el nombre que le daremos al despliegue


```
$ oc new-app wildfly:latest~https://github.com/<USUARIO>/book-insultapp.git
--name='app'
```


En mi caso, el resultado fue el siguiente:


```
jpanizza@jpanizaa:~$ oc new-app wildfly:latest~https://github.com/jpanizza/book-insultapp.git --name='app'
--> Found image af69006 (3 months old) in image stream "openshift/wildfly" under tag "latest" for "wildfly:latest"


    WildFly 13.0.0.Final 
    -------------------- 
    Platform for building and running JEE applications on WildFly 13.0.0.Final


    Tags: builder, wildfly, wildfly13


    * A source build using source code from https://github.com/gshipley/book-insultapp.git will be created
      * The resulting image will be pushed to image stream tag "app:latest"
      * Use 'start-build' to trigger a new build
    * This image will be deployed in deployment config "app"
    * Port 8080/tcp will be load balanced by service "app"
      * Other containers can access this service through the hostname "app"


--> Creating resources ...
    imagestream.image.openshift.io "app" created
    buildconfig.build.openshift.io "app" created
    deploymentconfig.apps.openshift.io "app" created
    service "app" created
--> Success
    Build scheduled, use 'oc logs -f bc/app' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/app' 
    Run 'oc status' to view your app.


```


Ahora expongamos una ruta para acceder a ella


```
$ oc expose service app


route.route.openshift.io/app exposed


```


Accedemos a la ruta de nuestra aplicación, para saber cual es ejecutemos:


```
$ oc get routes


NAME      HOST/PORT                                                                                                   PATH      SERVICES   PORT       TERMINATION   WILDCARD
app       app-lab-dojo.openshift-jose2-5290c8c8e5797924dc1ad5d1b85b37c0-0001.us-south.containers.appdomain.cloud                app        8080-tcp                 None


```


Puede acceder desde su navegador para probarla! 


El código que genera los insultos es bastante sencillo, ya que elige aleatoriamente dos adjetivos y un sustantivo de un array de strings. Esto se muestra en el siguiente código, que se puede encontrar en el archivo InsultGenerator.java en el directorio /src/main/java/org/openshift:


```java
package org.openshift;
import java.util.Random;
public class InsultGenerator {
    public String generateInsult() {
        String words[][] = {{"Artless", "Bawdy", "Beslubbering"},
        {"Base-court", "Bat-fowling", "Beef-witted"}, {"Apple-john",
        "Baggage", "Barnacle"}};
        String vowels = "AEIOU";
        String article = "an";
        String firstAdjective = words[0][
        new Random().nextInt(words[0].length)];
        String secondAdjective = words[1][
        new Random().nextInt(words[1].length)];
        String noun = words[2][new Random().nextInt(words[2].length)];
        if (vowels.indexOf(firstAdjective.charAt(0)) == -1) {
            article = "a";
        }
        return String.format("Thou art %s %s %s %s!", article,
        firstAdjective, secondAdjective, noun);
    }
}
```


### Agregar una base de datos


Si bien la aplicación funciona, la idea es utilizar una base de datos para extraer las palabras y así poder importar allí una cantidad mucho mayor. 


Lo primero que haremos será crear la base de datos desde la consola web, para esto debemos dirigirnos a nuestro nuevo proyecto, seleccionar en "Add to Project" y luego en "Browse Catalog". Allí busquemos PostgreSQL y completamos los pasos para su despliegue.  En mi caso configurare:


| Campo | Contenido |
| ------------- | ------------- |
| Memory Limit  | 512Mi  |
| Database service name  | lab  |
| PostgreSQL user  | user  |
| PostgreSQL password | pass  |
| PostgreSQL database  | database  |




Para poder comunicarnos con la base de datos utilizando estas variables de entorno, debemos agregarlas a la configuración del Deployment. Agregar las variables de entorno a DeploymentConfig en lugar de agregarlas directamente en el pod en ejecución garantiza que cualquier pod nuevo se iniciará con las variables que necesita para conectarse.


Esto se puede hacer usando el siguiente comando:


```
$ oc env dc insults -e POSTGRESQL_USER=user POSTGRESQL_DATABASE=database
```


Ahora vamos a importar el esquema de nuestra base de datos.
En este punto, tenemos la aplicación y una base de datos desplegadas, pero la base de datos está vacía, por lo que debemos solucionarlo. En el repositorio del código  de la aplicación, hay un archivo llamado insult.sql que contiene todo el SQL necesario para crear nuestro esquema. 
Un extracto del SQL es el siguiente:


```sql


DROP TABLE IF EXISTS short_adjective;
DROP TABLE IF EXISTS long_adjective;
DROP TABLE IF EXISTS noun;
BEGIN;
CREATE TABLE short_adjective (id serial PRIMARY KEY, string varchar);
CREATE TABLE long_adjective (id serial PRIMARY KEY, string varchar);
CREATE TABLE noun (id serial PRIMARY KEY, string varchar);
INSERT INTO short_adjective (string) VALUES ('artless');
INSERT INTO short_adjective (string) VALUES ('bawdy');
.
.
.
```


Tenemos que importar el SQL a nuestra aplicación web por lo que utilizaremos la terminal proporcionada en la consola web de OpenShift.


- Vayamos a "Overview" dentro de nuestro proyecto


- Luego hagamos clic sobre "1 pod" en el panel de información de la aplicación (NO de la base de datos)


- Allí veremos la lista de nuestros pod, en este caso solo tenemos uno. Hagamos clic en su nombre


- Se abrirá una ventana con todas la informacion de este pod, ahora hagamos clic en la pestaña "terminal"


- Allí ejecutamos lo siguiente:


```
psql -h $POSTGRESQL_SERVICE_HOST -p $POSTGRESQL_SERVICE_PORT \ -U $POSTGRESQL_USER $POSTGRESQL_DATABASE < insults.sql
```




Y listo! ahora tenemos varias tablas creadas dentro de su base de datos, que contienen una gran cantidad de palabras. 


### Agregar la base de datos a la aplicación


Tenemos esta nueva base de datos creada y vinculada a nuestra aplicación, pero todavía no la estamos utilizando desde nuestro código. Para usar la base de datos, primero debemos agregar una dependencia para el controlador de la base de datos en nuestro archivo maven pom.xml. Abra el archivo en un editor de código y agregue la siguiente sección después de la etiqueta **</properties>:**


```java
<dependencies>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>9.4-1200-jdbc41</version>
    </dependency>
</dependencies>
```


Una vez que hayamos agregado la dependencia para el controlador PostgreSQL, necesitamos modificar el archivo InsultGenerator.java para conectarnos y ejecutar consultas en la nueva base de datos. Abra este archivo y reemplace el contenido con lo siguiente:


```java


package org.openshift;
import java.sql.Connection;
import
import
import
java.sql.DriverManager;
java.sql.ResultSet;
java.sql.Statement;
public class InsultGenerator {
    public String generateInsult() {
        String vowels = "AEIOU";    
        String article = "an";
        String theInsult = "";
        try {
            String databaseURL = "jdbc:postgresql://";
            databaseURL += System.getenv("POSTGRESQL_SERVICE_HOST");
            databaseURL += "/" + System.getenv("POSTGRESQL_DATABASE");
            String username = System.getenv("POSTGRESQL_USER");
            String password = System.getenv("PGPASSWORD");
            Connection connection = DriverManager.getConnection(databaseURL, username,
            password);
            if (connection != null) {
                String SQL = "select a.string AS first, b.string AS second, c.string AS noun
                from short_adjective a , long_adjective b, noun c ORDER BY random() limit 1";
                Statement stmt = connection.createStatement();
                ResultSet rs = stmt.executeQuery(SQL);
                while (rs.next()) {
                    if (vowels.indexOf(rs.getString("first").charAt(0)) == -1) {
                    article = "a";
                    }
                    theInsult = String.format("Thou art %s %s %s %s!", article,
                    rs.getString("first"), rs.getString("second"), rs.getString("noun"));
                }
                rs.close();
                connection.close();
            }
        }catch (Exception e) {
            return "Database connection problem!";
        }
        return theInsult;
    }
}
```




Una vez que haya realizado los cambios y los haya subido a su repositorio, debemos comenzar un nuevo build para implementar la nueva versión del código en nuestra app. Esto se puede hacer desde la consola web tal y como vimos en el laboratorio anterior o puede comenzar un nuevo build usando la terminal con el siguiente comando:


$ oc start-build app


Una vez que la compilación haya finalizado, verifique que su aplicación esté funcionando accediendo desde el navegador web.


### Agregar un endpoint REST


Estamos listos para agregar funcionalidades adicionales a nuestra aplicación. Lo último que queremos agregar a nuestra app es la capacidad de devolver un insulto en un JSON por medio de una consulta HTTP. 


Lo primero que debemos hacer es agregar una dependencia para Java EE que nos permita usar JaxRS para generar los resultado. ABra el archivo pom.xml y agregue la siguiente dependencia:




```java
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>7.0</version>
    <scope>provided</scope>
</dependency>
```


Ahora que hemos agregado la dependencia a nuestro archivo pom.xml, es hora de agregar dos clases Java que expongan el endpoint. El primero es un simple archivo JaxrsConfig.java que especifica la ruta de la aplicación en la que estarán nuestros servicios. Cree un nuevo archivo llamado JaxrsConfig.java en el directorio /src/ main/java/org/openshift y agregue el siguiente código:


```java
package org.openshift;
import javax.ws.rs.ApplicationPath;
import javax.ws.rs.core.Application;


@ApplicationPath("/api")
public class JaxrsConfig extends Application{


}
```


Lo último que debemos hacer es definir nuestro endpoint el cual devolverá el insulto. Cree un nuevo archivo llamado InsultResource.java en el directorio /src/main/java/org/openshift
con el siguiente código:


```java


package org.openshift;
import java.util.HashMap;
import javax.enterprise.context.RequestScoped;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;


@RequestScoped
@Path("/insult")
public class InsultResource {
    @GET()
    @Produces("application/json")
    public HashMap<String,String> getInsult() {
        HashMap<String, String> theInsult = new HashMap<String, String>();
        theInsult.put("insult", new InsultGenerator().generateInsult());
        return theInsult;
    }
}
```


Estos dos archivos expondrán un endpoint en la URL /api/insult y escucharán las solicitudes GET. Una vez que se realiza una solicitud, se devolverá un documento JSON con un insulto proporcionado por la base de datos PostgreSQL que agregamos anteriormente.


Una vez que haya subido código a su repositorio, no se olvide de generar un nuevo build! Luego podrá probarlo, deberia recibir un JSON como el siguiente:


```json
{"insult":"Thou art a dankish weather-bitten minnow!"}
```

Felicitaciones!! Completo el lab2 de OpenShift. 

