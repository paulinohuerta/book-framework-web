# JSF Clase 1

## JSF en pocas palabras

No usamos el viejo estilo de sintaxis jsp; nuestras páginas usarán xhtml (1), 
en el que definimos un namespace  xmlns:h="http://java.sun.com/jsf/html".
Se usan componentes JSF como h:head, h:body, h:form. También los componentes tienen facetas
estás se definen en otro namespace xmlns:f...
Más adelante veremos que tambi podemos usar elementos aún de otro namespace  xmlns:ui...
Las páginas que no contienen input pueden omitir h:form.
No es necesaria una entrada @taglib como en páginas jsp.
Recuerda que la URL no coteja el real filename, usarás tipos de nombres como *blala.xhtml* para páginas a mostrar al usuario.   
Veamos un ejemplo,    

{:lang="xhtml"}
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
       "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
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
        <center>
         <h:commandButton value="submit" action="suc"/>
        </center>
       </h:form>
      </h:body>
    </html>

Una vez visto como vienen organizadas nuestras páginas usando componentes del framework, pasemos a introducir otros conceptos como *beans*.   
Los _beans administrados_, aunque mejor usaremos el término *Managed Beans*, son clases java que bien pueden instanciarse mediante definición en el fichero
faces-config.xml o mediante anotaciones Java como  esta *@ManagedBean*

1. Son normalmente POJOs.
2. Tienen pares de métodos getter y setter correspondientes a cada elemento de input en el formulario
3. Tienen un método *action controller* que no toma argumento y retorna un string.  Este método está ligado a la acción del h:commandButton en el formulario.
4. Normalmente tienen marcadores para propiedades derivadas, lo que es información a tenerse en cuenta en el proceso de los datos de entrada.

Si escogemos anotación, podríamos tener:
{:lang="java"}
    @ManagedBean 
    public class SomeName { ... }

Con esto nos referimos a un bean del backend con *#{someName.blah}*, donde el
nombre del bean es el nombre de la clase con la primer letra pasada a minúscula.
Request es el ámbito por defecto, y "blah" es un nombre de método o una
propiedad la que enlaza con un método getter o setter; esto último ocurre en
h:inputText, mientras que la action de h:commandButton es un exacto nombre de método.

Acerca del valor de retorno del método *action controller* tenemos que conocer que hay un mecanimo detrás de la escena que consiste en que si
el método retorna _foo_ o _bar_ y no hay un explícito mapeo en el fichero faces-config.xml, entonces
la página resultante es foo.xhtml ó bar.xhtml, desde la misma carpeta que está contenido el formulario.

Por ejemplo, si en una página inicial tenemos un botón así:
{:lang="xhtml"}
    <h:commandButton action="#{navigator.choosePage}"/>

Y un bean como el siguiente

{:lang="java"}
    @ManagedBean 
    Class  Navigator {...}

Podríamos plantear que el método _choosePage_ retorne tres posibles strings *page1*, *page2*, ó *page3*.   
Los nombres cotejados por el retorno del método choosePage serán page1.xhtml, page2.xhtml, y page3.xhtml. Esto significa que son tres vistas, escritas en xhtml, formando parte de tu aplicación, diseñadas y escritas para ser enviadas como 
respuesta al cliente.

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

Esta es una declaración de un *managed bean*, sin requerir una entrada en faces-config.xml, ya que se ha usado una *Java annotation*.
Debido a que no se da un nombre será usado el nombre de la clase con la primer letra en minúscula.
Podemos dar un nombre al bean, que creará el framework para nosotros con la
siguiente notación,
{:lang="java"}
    @ManagedBean(name="someName")   
El ámbito no se ha dado entonces, por defecto el *scope* es *request*. Podemos indicar el
ámbito del bean también con notación, esto podría ser así,
{:lang="java"}
    @SessionScoped 
Puesto que en nuestro ejemplos no agregamos explícitas reglas de navegación
en el fichero faces-config.xml, el cual ni estaría presente en esta supuesta aplicación ejemplo.
Los valores de retorno corresponden a page1.xhtml, page2.xhtml, y page3.xhtml.

## Fases del framework

Antes de pasar al detalle de las fases del *ciclo de vida* interesa tener presente como vas a intervenir como desarrollador en todo este proceso de escribir una aplicación.
Como primera cuestión habrás instalado una *implementación* de la especificación de referencia
de JSF, conocida como JSF RI. Mojarra es el nombre de la actual JSF RI, y es la implementación que viene con servidores como Glassfish entre otros.

Cuando alguien comienza a estudiar y comprender como trabaja JSF, ya viene con alguna práctica realizada
con la API servlet y el uso de técnicas que pretenden eliminar código Java de las vistas, es decir, que quien comienza con JSF, normalmente conoce lo que es un scriptlet; la cuestión aquí es hacer incapié en el hecho
de la existencia de estándares Java destinados ha organizar mejor el código y 
hacer más productivo el desarrollo de aplicaciones, nos referimos a estándares como Expression Language y la librería JSTL.    
Antes de estudiar JSF, tal ves sea preciso formularnos la siguiente pregunta:   
*Los servlets, ¿no tienen damasiadas responsibilidades?*       
En en su respuesta estaría la justificación del por qué estudiamos JSF.

Tomemos la pregunta como excusa para repasar algunas de las acciones y tareas de las que se ocupa
un servlet; es quien obtiene los datos de las peticiones HTTP, los convierte 
si es necesario, los valida, los pone a disposición de las páginas JSP despachándolo (forward), agregando previamente para esto 
último, nuevos atributos al objeto request; instancia objetos a partir
de definiciones que se encuentran en el modelo, y pasa datos de la petición cliente
hacia el modelo, enlazando la lógica de negocio con lo que es la input/output HTTP.

Partimos del hecho que un framework como JSF, nos va a ofrecer un conjunto de componentes específicos para alcanzar las tareas 
de la aplicación, trabajando modularmente y sobretodo nos permitirá dedicar
nuestro esfuerzo a desarrollar la lógica de negocio, y no distraer nuestra
atención en tareas repetitivas y comunes en las aplicaciones web. JSF nos obliga a trabajar en un *flujo bien concreto*, el cual se conoce como _ciclo de vida_.

JSF es un framework basado en _componentes_ y no en *acciones* como CodeIgniter u otros
similares sin distinción del lenguaje; cuando surgió JSF rompió con
la costumbre de la época. JSF tenía una forma muy diferente, el "status quo" del momento, de los framworks web, estaba basado en acciones.   

## Controlador: enlace entre usuario y aplicación

Con el código del ejemplo xhtml visto anteriormente, podemos ver el aspecto que pueden
tener las vistas, éstas además de no contar con la existencia de código 
Java entremezclado marcan a su vez una distancia con la codificación HTML;
esto en cuanto a las páginas destinada al usuario, pero ¿que pasa con el *modelo*?   
el modelo estaría participando, de similar
forma a lo visto en páginas JSP usando EL, expression language, concretamente "#{order.color}" en el formulario
podría pensarse que hace referencia a un atributo de un objeto, tal vez creado por el framework JSF; con técnica de uso servlet/JSP, hubiéramos dicho que tal objeto fue creado por el servlet, *agregado* en el objeto request y enviado a la vista.

La realidad es que el *controlador JSF* tiene definido en exacto flujo de trabajo. El controlador o servlet Faces
actúa como enlace entre el usuario y la lógica de la aplicación, siendo el
ciclo de vida un flujo de eventors entre código de aplicación y peticiones de usuario.    
La descripción del ciclo sería algo así:   
Una vez recibida una petición web, digamos una primera petición, el servlet,
*controlador* Faces, va a manipular esta petición creando el contexto JSF, el cual es un *objeto Java* que *retiene* todos los datos de la aplicación, luego 
el *controlador* enruta la petición a la página requerida. La página normalmente devuelve o retorna los datos
de la aplicación desde el contexto JSF usando un simple Expression Language. Esto acontece en cuanto a la primera petición, con las las siguientes
peticiones, el *controlador* modifica cualquier dato del Modelo, suministrando cualquier nueva input y actualizando el contexto.
Los desarrolladores JSF tienen acceso programático al ciclo descripto; acceso en cualquier momento de la ejecución del ciclo, es decir en
cualquiera de sus fases, significa que pueden ejercer un alto grado de control
sobre la conducta de la aplicación, en el sitio y momento que se lo propongan.  
Dicho esto, es de esperar que alguien que tenga pensado usar el framewor conozca las *seis faces del ciclo de vida de JSF*,
reciben los siguientes nombres,

1. Restore view
2. Apply request values
3. Process validations
4. Update model values
5. Invoke application
6. Render response

I> Buena propuesta para captar un poco mejor las [fases del framewok JSF](https://balusc.blogspot.com.es/2006/09/debug-jsf-lifecycle.html), es la de codificar
una aplicación completa, tan sólo con el propósito de entender lo que pasa con una petición web de principio a fin.

## Un framework statefull

Cuando desarrollamos con un framework *basado en acción* tenemos 'tags' que nos
ayudan en el desarrollo de la vista, y podemos pensar que está ligado, lo que 
significa que si quitamos las tags y lo hacemos a mano también lo
conseguiríamos.
Cuando submit un formulario basado en acción, nuestra aplicación basada en la ación no se fija en el estado de la vista.
No tiene en cuenta como la vista fue escrita, tampoco si es un html estático
o un jsp. Se montará una petición que tiene como parámetro tal o cual dato, que los aporta el usuario en el formulario.
Los frameworks basados en ación no conocen un formulario, conocen la *request* y la información que está siendo pasada en ella al servidor.

Cuando trabajamos con JSF, _él_ es quien crea el formulario, es decir cuando se
accede desde una petición
a la URL de una página *xhtml*, ocurre que en lugar de ejecutar un servlet,
_JSF va a leer el archivo y montarlo en memoria_, lo que es lo mismo, JSF crea un árbol de componentes; luego ese árbol en memoria será pasado por un renderizador.

El árbol de componentes representa la estructura de la página y JSF lo utilizará para escribir el código HTML, en un momento del ciclo de vida.

En definitiva JSF, guarda en memoria el árbol que fue usado para generar un formulario.    
Esto no ocure en un servlet, que genera automáticamente un HTML a partir de
una página JSP, esto lo hace el servidor para enviar datos al cliente, es para
esto que que tiene en cuenta el formulario, para enviarlo, lo mismo es en los frameworks
basado en acción, mientras que JSF, al guardar en memoria la estructura de
la página, en ese árbol de componentes, _él_ va a comparar cada atributo de
la petición con los campos que estaban disponibles para el usuario.
Teniendo toda esta información JSF, podrá validar los datos enviados por el usuario.
Se puede decir que JSF sabe todo acerca de un formulario, lo que había antes, y lo que hay ahora que el usuario envió datos, JSF conoce el estado de la vista y lo mantiene durante las peticiones, por ello podemos considerarlo un *framework statefull*.

Una herramienta útil tal como [demuestra Balusc](https://balusc.blogspot.com.es/2006/09/debug-jsf-lifecycle.html) (nombre de un usuario con merecido respeto
en stackoverflow), es destinar una aplicación JSF para tener una visión de cada una de las fases, la entrada y salida de cada una de ellas. La aplicación de 
Balusc pretende que seamos avisados antes y después de cada fase, ofrece sus reflexiones e indica las posibles buenas prácticas que se derivan de esta *aplicación explicativa*

En la _siguiente clase_ vamos a proponernos *jugar*, aunque sea mínimamente con el ciclo de vida, con el fin de
ver más de cerca la responsabilidad de alguna de las fases, utilizaremos un _PhaseListener_, y principalmente demostraremos que estamos en condiciones de obtener un *acceso granular a un punto concreto* del ciclo de vida de una petición.

Ahora veamos una clase Java, representa el modelo y este es el código que contiene
la lógica de la aplicación,

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

Mostramos parcialmente el contenido de _web.xml_
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

Del fichero web.xml destacamos el siguiente punto:    
La extensión *.jsf* normalmente se mapeaba en web.xml, así lo vemos en el  ejemplo.
Mientras que la extensión *.xhtml* es la extensión de los ficheros _actuales_
y que físicamente están en el webcontent de tu aplicación.   
La conclusión es que si invocas una página con extensión .jsf como por ejemplo
http://localhost:8080/webapp/page.jsf, entonces el 
servlet JSF debe localizar a page.xhtml y realizar el correspondiente  parse/render de los componentes JSF

Desde JSF 2.0 un mapeo a páginas *.xhtml es posible, lo que significa que la URL http://localhost:8080/webapp/page.xhtml puede ser accedida, tendremos en web.xml    

{:lang="xml">
    <url-pattern>*.xhtml</url-pattern>

Observa el parámetro *Project Stage* este es un parámetro de contexto que identifica
el *estadio* o etapa de la aplicación en su ciclo de vida. El estadio de la aplicación
puede afectar la conducta de la misma. Un ejemplo de ello puede ser mensajes de error, que se busca mostrar durante la etapa o fase de desarrollo, mientras
que será preciso suprimirlos en etapa de Producción.

Siguiendo con el ejemplo, este sería el fichero _index.html_
{:lang="html"}
    <html>
      <head>
       <meta http-equiv="refresh" content="0; URL=editCustomer.jsf">
      </head>
    </html>

Mostramos el *body* de _cancelled.xhtml_, una de las páginas de las vistas de la aplicación,
{:lang="xhtml"}
    <?xml version="1.0" encoding="UTF-8"?>
    <body>
      <h1><h:outputText value="MyGourmet"/></h1>
      <h:outputText value="Cancelled customer editing!"/>
    </body>

Parcialmente el fichero _editCustomer.xhtml_, otra de las páginas,
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

De la anterior vista resaltamos el uso del atributo *immediate* con valor true,
este produce que desde la actual fase pasará directamente a la última, sin producir cambios de ningún tipo en los datos del modelo, con lo que la renderización
desde ese punto de vista es inmediata. Mientras que el atributo *required* con
valor *true* para algún componente, en este caso los dos inputText, indica que el ciclo
de vida finalizará en la fase 3 *proccess validations* si los componentes están _vacíos_,
y el servlet controller *renderiza error* en la última etapa.

Parcialmente _showCustomer.xhtml_, la última vista,
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

Por último verás que los datos se mantienen, en los sucesivos accesos a la
aplicación, observa el *scope* que se ha usado para la creación del bean.
Puedes modificar el ámbito del bean mediante las correspondientes *anotaciones*,
verificando la conducta de la aplicación.
