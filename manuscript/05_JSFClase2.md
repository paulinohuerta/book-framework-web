# JSF Clase 2

Conociendo que una petición es procesada por JSF, a través de varias fases,
intentaremos analizar alguna de éstas y además aprenderemos acerca del concepto
JSF PhaseListener.

[ejemplo fases JSF](http://www.itcuties.com/j2ee/jsf-phaselistener/)

Es posible poner un *phase listener* a cada una de las fases. Podríamos modificar los datos enviados en la petición, si así nos lo propusiéramos, o bien podríamos
modificar la vista que será renderizada. En este caso nos proponemos desactivar o bien esconder (disable/hide) un componente InputText presente en la vista.
En concreto pretendemos que cuando el usuario introduzca _hide me_ como texto y
el formulario sea enviado a la aplicación en el servidor, nuestro PhaseListener
debería poner el atributo *rendered* de dicho componente a *false*.
Mientras que introducir _disable me_ como texto del componente en cuestión, este
debería ser *disabled*

Escribimos la clase FieldDisableListener.java

{:lang="java"}
    package mypackage;
    
    import javax.faces.component.UIComponent;
    import javax.faces.component.html.HtmlInputText;
    import javax.faces.event.PhaseEvent;
    import javax.faces.event.PhaseId;
    import javax.faces.event.PhaseListener;
    
    public class FieldDisableListener implements Pha
    seListener {
      private static final long serialVersionUID = -7607159318721947672L;
       // La fase que interesa, en ésta es llamado el listener
      private PhaseId phaseId = PhaseId.RENDER_RESPONSE;
       /**
        * Recorrido recursivo del árbol de componentes 
        */
      public void beforePhase(PhaseEvent event) {
         processViewTree(event.getFacesContext().getViewRoot());	
      }
      public void afterPhase(PhaseEvent event) {
        // No nos interesa hacer algo
      }
      public PhaseId getPhaseId() {
        return phaseId;
      }
      private void processViewTree(UIComponent component) {
        // Go to every child
        for (UIComponent child: component.getChildren()) {
          // Display el ID y tipo de componente
          System.out.println("+ " + child.getId() +
                           " ["+child.getClass()+"]");
          // Si el componente es el nuestro, coteja ID, entonces 
          // verifica si debe esconderlo
          if ("dummy-text-id".equals(child.getId())) {
            // Obtiene el texto del componente
            HtmlInputText inputText = (HtmlInputText)child;
            String  inputTextValue = (String)inputText.getValue();
            // Si coteja el texto con "hide me", esconde el campo
            if ("hide me".equals(inputTextValue))
              inputText.setRendered(false);
               // Disable el campo si el texto coteja "disable me"
            if ("disable me".equals(inputTextValue))
              inputText.setReadonly(true);
          }
          // Process next node
          processViewTree(child);
        }
      }
    }

Nuestra clase implementa la interfaz PhaseListener.   
¿Cuando es ejecutado el código?   
Es ejecutado antes de la última fase RENDER_RESPONSE
En el método beforePhase el *view tree* es procesado hacia abajo de forma
recursiva. Cuando el InputText componente es encontrado, usa el ID para cotejar,
nuestro valor a encontrar es *dummy-text-id*, entonces es cuando verifica
si el el valor del texto es *hide me* o *disable me*, con lo que el componente
podrá ser modificado.

El uso de un Phaselistener exige un detalle de configuración en el fichero
faces-config.xml
Para esto usamos la *tag*
En caso de usar más de un listener, el orden en que aparecen en el fichero
de configuración es tenido en cuenta como órden de ejecución.

{:lang="xhtml"}
    <?xml version="1.0" encoding="UTF-8"?>
    <faces-config xmlns="http://java.sun.com/xml/ns/javaee"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
         http://java.sun.com/xml/ns/javaee/web-facesconfig_2_1.xsd"
       version="2.1">
     
       <lifecycle>
          <phase-listener>mypackage.FieldDisableListener</phase-listener>
       </lifecycle>
    </faces-config>

El siguiente es el contenido del fichero *index.xhtml*, este es la vista
de nuestra aplicación, podemos apreciar el componente InputText *dummy-text-id*

{:lang="xhtml"}
    <?xml version="1.0" encoding="UTF-8"?> 
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" 
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"> 
    <html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:f="http://java.sun.com/jsf/core"      
      xmlns:h="http://java.sun.com/jsf/html"> 
    <h:head> 
      <title>Phase Listener example</title> 
    </h:head> 
    <h:body> 
      <h3>Phase Listener example</h3> 
      <h:form> 
        <h:inputText id="dummy-text-id"/> 
    	<h:commandButton id="button" value="Submit" /> 
        <h:messages /> 
    </h:body> 
    </html>

Y el fichero web.xml que no tiene nada especial para esta aplicación

{:lang="xhtml"}
    <!DOCTYPE web-app PUBLIC
     "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
     "http://java.sun.com/dtd/web-app_2_3.dtd" >
    <web-app>
        <display-name>Ejemmplo JSF 2 Phase listener</display-name>
        <!-- The welcome page -->
        <welcome-file-list>
            <welcome-file>index.xhtml</welcome-file>
        </welcome-file-list>
        <!-- Start JSF -->
        <servlet>
            <servlet-name>Faces Servlet</servlet-name>
            <servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
            <load-on-startup>1</load-on-startup>
        </servlet>
        <!-- JSF URL mapping -->
        <servlet-mapping>
            <servlet-name>Faces Servlet</servlet-name>
            <url-pattern>*.xhtml</url-pattern>
        </servlet-mapping>
    </web-app>
