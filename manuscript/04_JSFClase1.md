# JSF Clase 1

## JSF en pocas palabras

No usamos el viejo estilo de sintaxis jsp; nuestras páginas usarán xhtml (1), 
en el que definimos un namespace  xmlns:h="http://java.sun.com/jsf/html".
Se usan componentes JSF como h:head, h:body, h:form. También los componentes tienen facetas
estás se definen en otro namespace xmlns:f...
Más adelante veremos que podremos usar  xmlns:ui...
Las páginas que no contienen input pueden omitir h:form
No es necesaria una entrada @taglib
Recuerda que la URL no coteja el real filename, usarás blala.xhtml

{:lang="xhtml"}
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    <html xmlns="http://www.w3.org/1999/xhtml"
       xmlns:h="http://java.sun.com/jsf/html"
       xmlns:f="http://java.sun.com/jsf/core">
      <head>
       <title>
        Beer Selection
       </title>
      </head>
      <h:body>
       <h1 align="center">Beer Selection Page</h1>
        Select un color<p/>
        Color:
       <h:form>
        <h:selectOneMenu value="#{order.color}" >
          <f:selectItem itemValue="light" itemLabel="Es ligth" />
          <f:selectItem itemValue="amber" itemLabel="Es amber" />
          <f:selectItem itemValue="brown" itemLabel="Es brown" />
          <f:selectItem itemValue="dark" itemLabel="Es dark" />
        </h:selectOneMenu>
        <br/><br/>
        <center>
         <h:commandButton value="submit" action="suc"/>
        </center>
       </h:form>
      </h:body>
    </html>

Una vez visto como vienen organizadas nuestras páginas usando componentes del framework, seguimos introduciendo más conceptos:
Los beans administrados, aunque mejor usaremos el término *Managed Beans*, son clases java que son declarados en el file faces-config.xml o bien mediante anotaciones @ManagedBean

1. Son normalmente POJOs.
2. Tienen pares de métodos getter y setter correspondientes a cada elemento de input en el formulario
3. Tienen un método *action controller* que no toma argumento y retorna un string.  Este método está ligado a la acción del h:commandButton en el formulario.
4. Normalmente tienen marcadores para propiedades derivadas, lo que es información a tenerse en cuenta en el proceso de los datos de entrada.

Si escogemos anotación, podríamos tener:
{:lang="java"}
    @ManagedBean 
    public class SomeName { ... }

Con esto nos referimos a un bean con *#{someName.blah}*, donde el nombre del bean es el nombre de la clase con la primer letra cambiada a minúscula.
Request es el ámbito por defecto, y "blah" es un nombre de método o una propiedad la que enlaza con un método getter o setter; esto último ocurre en h:inputText, mientras que el la action de h:commandButton es un exacto nombre de método.

Acerca del valor de retorno del action controller method:
Si este método retorna "foo" ó "bar" y no hay un explícito mapeo en faces-config.xml, entonces
la página resultante es foo.xhtml ó bar.xhtml, desde la misma carpeta que está contenido el
formulario.

    Por ejemplo, si en una página inicial tenemos un botón así:

{:lang="xhtml"}
    <h:commandButton action="#{navigator.choosePage}"/>

Y un bean como el siguiente

{:lang="java"}
    @ManagedBean 
    Class  Navigator {...}

El método _choosePage_ retorna tres posibles strings *page1*, *page2*, ó *page3*.   
Los nombres cotejados por el retorno del método choosePage serán page1.xhtml, page2.xhtml, y page3.xhtml

     Veamos la clase Navigator
{:lang="java"}
    @ManagedBean                                                          
    public class Navigator {
      private String[] resultPages = { "page1", "page2", "page3" };
      public String choosePage() {
        return(RandomUtils.randomElement,resultPages);
      }
      // getter y setter
    }

Esta es una declaración de un *managed bean*, sin requerir una entrada en faces-config.xml, se usa annotación.
Debido a que no se da un nombre será usado el nombre de la clase con la primer letra en minúsicucla.
También podría usarse un nombre así:   
{:lang="java"}
    @ManagedBean(name="someName")   
El ámbito no se ha dado entoncespor defecto es *request*. Podríamos usar una anotación así:      
{:lang="java"}
    @SessionScoped 
Puesto que no hay explícitas reglas de navegación en el fichero faces-config.xml,
los valores de retorno corresponden a page1.xhtml, page2.xhtml, y page3.xhtml.

## Fases del framework

Antes de pasar al detalle de las faces del *ciclo de vida* interesa tener presente como vas a intervenir.   
Como primera cuestión habrás instalado una *implementación* de la especificación de referencia
de JSF, conocida como JSF RI. Mojarra es el nombre de la actual JSF RI, y es la implementación
de que viene con servidores como Glassfish entre otros.

Cuando alguien comienza a estudiar y comprender como trabaja JSF, ya viene con alguna práctica realizada
con la API servlet y el uso de técnicas que pretenden eliminar código Java de las vistas, es decir, que quien comienza con JSF, normalmente conoce lo que es un scriptlet; la cuestión aquí es hacer incapié en el hecho
de la existencia de estándares Java destinados ha organizar mejor el código y hacer más productivo el desarrollo de aplicaciones, nos referimos a estándares como Expression Language y la librería JSTL.    
Al comenzar el estudio del framwork JSF es preciso hacernos esta pregunta:    
*¿Los servlets no tienen damasiadas responsibilidades?*
en su respuesta estaría la justificación del por qué estudiamos JSF.

Tomemos la pregunta anterior como excusa para repasar algunas de las acciones y tareas de las que se ocupa un servlet; es quien obtiene los datos de las peticiones HTTP, los convierte si es necesario, los valida, los pone a disposición de las páginas JSP despachándolo (forward), agregando previamente para esto último, nuevos atributos al objeto request.

Partimos del hecho que un framework como JSF, nos ofrece un  conjunto de componentes específicos para alcanzar las tareas de la aplicación, trabaja en un flujo bien concreto, y es este flujo el que se conoce
como ciclo de vida.

JSF es un framework basado en componentes y no en acciones como CodeIgniter; en el momento que JSF surgió rompió con
la costumbre de la época. JSF tenía una forma muy diferente, el "status quo" del momento, estaba basado en acciones.   

## El controlador como enlace entre usuario y aplicación

Hasta aquí, podemos ver que además de la no existencia de código Java entremezclado, el *modelo* podría estar participando, de forma
similar a lo que hemos visto en páginas JSP usando el EL, expression language, concretamente "#{order.color}" en el formulario
podría pensarse que hace referencia a un atributo de un objeto, tal vez creado por el framework JSF; con técnica de uso servlet/JSP, hubiéramos dicho que tal objeto fue creado por el servlet, *agregado* en el objeto request y enviado a la vista.

La realidad, es que que hay definido en exacto flujo de trabajo definido por el *controlador JSF*.   
El servlet Faces actúa como un enlace entre el usuario y la lógica de la aplicación JSF.   
¿Cómo participa?
Lo hace en un ciclo de vida bien definido, el cual dicta el flujo completo de eventos
entre entre la aplicación y las peticiones de usuario. Por ejemplo para acceder a la aplicación mediante una primera petición web, el servlet *controlador* Faces al manipular dicha petición, prepara
el contexto JSF, el cual es un *objeto Java* el cual 'holds' todo los datos de la app, entonces
el *controlador* enruta la petición a la página requerida. La página normalmente render los datos
de la aplicación desde el contexto JSF usando un simple Expression Language. Luego, sobre las siguientes
peticiones, el *controlador* modifica cualquier dato del Modelo, suministrando cualquier nueva input
que haya ocurrido.   
Los desarrolladore JSF tienen un acceso programático al ciclo de vida al completo en cualquier momento durante la ejecución, con lo que puede ejercer un alto grado de control sobre la conducta
de la aplicación, en el sitio y momento que se lo propoga. Esto quiere decir que es necesario que
conozca las *seis faces del ciclo de vida de JSF*

1. Restore view
2. Apply request values
3. Process validations
4. Update model values
5. Invoke application
6. Render response

[JSF ciclco de vida](https://balusc.blogspot.com.es/2006/09/debug-jsf-lifecycle.html)

## Un framework statefull

Cuando desarrollamos con un framework basado en acción tenemos 'tags' que nos ayudan en el desarrollo de la vista, y podemos pensar que está ligado, lo que si
quieres quitar las tags y hacerlo a mano lo consigues.
cuando submit un formulario basado en acción, nuestra app basada en la aciión no se fija en el estado de la vista.
no es tenido en cuenta como ella fue escrita, tampoco si es un html estático o un jsp. Se montará una petición que tiene como parámetro est e y este otro dato, que los aportó el usuario en el formulario.
los frameworks basados en ación no conocen un formulario, conocen la request y la información que está siendo pasada en ella.

cuando trabajamos con JSF,él es quien crea el formulario, es decir cuando llamamos a nuestra pagina.xhtml en lugar de ejecutar un servlet, JSF va a leer el archivo y lo monta en memoria, crea un arbol de componentes; luego ese arbol den memoria he pasado por un renderizador.
con lo que el arbol de componentes representa la estructura de la página y que JSF utilizará para escribir el HTML, en un momento del ciclo de vida.

JSF, guarda en memoria el árbol que fue usado para generar ese formulario.
Esto no ocure en un servlet, que genera automáticamente un HTML a partir de un JSP, esto lo hace el servidor para enviar datos al cliente, es para esto que que tiene en cuenta el formulario, para enviarlo, lo mismo es en los frameworks
basado en acción, mientras que JSF, al guardar en memoria la estructura de la página, en ese árbol de componentes, él va a comparar cada atributo de la petición con los campos que estaban disponibles para el usaurio.
Teniendo toda esta información JSF, podrá validar los datos enviados por el usuario. Se puede decir que JSF sabe  todo acerca de un formulario, que había que hay ahora que el usuario envió datos, JSF conoce el estado de la vista y lo mantine a través de las peticiones , por ello podemos considerarlo un framework 'statefull'.

Una buena práctica para entender el ciclo de vida de JSF es codificar una aplicación tal como lo [demuestra
Balusc](https://balusc.blogspot.com.es/2006/09/debug-jsf-lifecycle.html) (nombre de un usuario con merecido respeto en stackoverflow), es una codificación destinada exclusivamente para que un desarrollador JSF, tenga una herramienta concreta para recorrer y estudiar el flujo completo con la entrada y salida de cada una de las etapas. La aplicación de Balusc pretende que seamos avisados antes y después de cada fase, ofreciendo sus reflexiones y las posibles buenas prácticas que se derivan de esta *aplicación explicativa*

En la siguiente clase vamos a proponernos *jugar*, aunque sea mínimamente con el ciclo de vida, con el fin de
ver más de cerca la responsabilidad de alguna de las fases, utilizaremos un _PhaseListener_, y principalmente demostraremos que estamos en condiciones de obtener un *acceso granular a un punto concreto* del ciclo de vida de una petición.

{:lang="java"}
    package mypackage;
    import javax.faces.bean.ManagedBean;
    import javax.faces.bean.SessionScoped;
    import javax.faces.bean.ApplicationScoped;
    
    @ManagedBean
    @ApplicationScoped
    public class Customer {
      private String firstName;
      private String lastName;
      public String getFirstName() {
        return firstName;
      }
      public void setFirstName(String firstName) {
        this.firstName = firstName;
      }
       // resto de getter y setter
      public String save() {
        return "/showCustomer.xhtml";
      }
    }
Mostramos parcialmente el contenido de web.xml
{:lang="xml"}
    <?xml version="1.0" encoding="UTF-8"?>
    <!-- se omiten las definiciones -->
    <!-- Faces Servlet -->
    <servlet>
      <servlet-name>Faces Servlet</servlet-name>
      <servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
      <load-on-startup>1</load-on-startup>
    </servlet>
    <!-- Faces Servlet Mapping -->
    <servlet-mapping>
      <servlet-name>Faces Servlet</servlet-name>
      <url-pattern>*.jsf</url-pattern>
    </servlet-mapping>
    <!-- Welcome files -->
    <welcome-file-list>
      <welcome-file>index.html</welcome-file>
    </welcome-file-list>
    <context-param>
      <param-name>javax.faces.PROJECT_STAGE</param-name>
      <param-value>Development</param-value>
    </context-param>

El fichero _index.html_
{:lang="html"}
    <html>
      <head>
       <meta http-equiv="refresh" content="0; URL=editCustomer.jsf">
      </head>
    </html>

Mostramos el *body* de cancel.xhtml
{:lang="xhtml"}
    <?xml version="1.0" encoding="UTF-8"?>
    <body>
      <h1><h:outputText value="MyGourmet"/></h1>
      <h:outputText value="Cancelled customer editing!"/>
    </body>

Parcialmenteel fichero editar.xhtml
{:lang="xhtml"}
    <body>
      <h1><h:outputText value="MyGourmet"/></h1>
      <h2><h:outputText value="Edit Customer"/></h2>
      <h:messages showDetail="true" showSummary="false"/>
      <h:form id="form">
        <h:panelGrid id="grid" columns="2">
          <h:outputLabel value="First Name:" for="firstName"/>
          <h:inputText id="firstName" value="#{customer.firstName}"
                   required="true"/>
          <h:outputLabel value="Last Name:" for="lastName"/>
          <h:inputText id="lastName" value="#{customer.lastName}"
                   required="true"/>
        </h:panelGrid>
        <h:commandButton id="save" action="#{customer.save}" value="Save"/>
        <h:commandButton id="cancel" action="/cancelled.xhtml" value="Cancel"
                immediate="true"/>
      </h:form>
    </body>

Parcialmente show.xhtml
{:lang="xhtml"}
    <body>
      <h1><h:outputText value="MyGourmet"/></h1>
      <h2><h:outputText value="Show Customer"/></h2>
      <h:panelGrid id="grid" columns="2">
        <h:outputText value="First Name:"/>
        <h:outputText value="#{customer.firstName}"/>
        <h:outputText value="Last Name:"/>
        <h:outputText value="#{customer.lastName}"/>
      </h:panelGrid>
      <h:outputText value="Customer saved successfully!" />
    </body>

