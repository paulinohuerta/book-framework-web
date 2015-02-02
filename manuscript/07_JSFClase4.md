# JSF Clase 4

Si bien, en esta clase nos ocupamos de modificar la clase aceptando acciones de modificación y
eliminación, al principio nos proponemos algunas indicaciones sobre el uso de parámetros.

## Usando parámetros

La etiqueta _f:param_ permite pasar parámetros a un componente, en el siguiente ejemplo vemos
un uso de _f:param_ en un componente _h:outputFormat_

Tal como tenemos el código de _index.xhtml_ al final de la clase anterior, puedes agregar 
al final del body, luego del componente h:dataTable
{:lang="xhtml"}
    <ul>
      <ui:repeat value="#{viewManager.cacheList}" var="item">
        <li>
         <h:link value="View details of #{item.key}" outcome="edit">
             <f:param name="id1" value="#{item.key}" />
             <f:param name="id2" value="#{item.value}" />
         </h:link>
        </li>
      </ui:repeat>
    </ul>

Se observa el uso del componente h:link donde existe un atributo _outcome_ y dos subelementos
facetas _f:param_, esto genera el siguiente código HTML
{:lang="xhtml"}
    <a href="/.../edit.xhtml?id1=xxx&id2=zzzz">View details of xxx</a>

Según valor de _outcome_ es renderizado _edit.xhtml_, que podría tener un contenido como sigue,
{:lang="xhtml"}
    <h2>CRUD estadio dos, editando uno</h2>
    <i>Key vale #{param['id1']} y Value vale #{param['id2']}</i>

También se ha usado la faceta _ui:repeat_ para realizar el bucle sobre la lista, en este caso
no nos interesa generar una tabla HTML.

En el siguiente ejemplo usamos _f:viewParam_ dentro de una faceta _f:metadata_
de la vista _cursoView.xhtml_; entonces con este componente _h:link_
{:lang="xhtml"}
    <h:link outcome="cursoView.xhtml?code=#{v_curso.code}"
       value="#{v_curso.title}" />
será renderizado _cursoView.xhtml_, el cual tiene
{:lang="xhtml"}
    <f:metadata>
      <f:viewParam name="code" value="#{cursoHome.cargar}" />
    </f:metadata>
Donde _f:viewParam_ procesa el parámetro GET, y tal como lo hace un componente h:inputText, realiza setting,
conversión y validación, con lo que el valor del parámetro _code_ sirve para realizar el _set_ del atributo
_cargar_ del bean _cursoHome_, representado como _#{cursoHome.cargar}_ en el atributo value.    
Si el atributo value se omite, entonces queda disponible en la vista en _#{code}_
	
¿Qué diferencia existe en usar @ManagedProperty en relación a f:viewParam?
Veamos, tendríamos los siguientes  dos bloques de código:

{:lang="java"}
    @ManagedProperty(value = "#{param.id}")
    private Integer id;

{:lang="xhtml"}
    <f:metadata>
      <f:viewParam name="id" value="#{someBean.id}"/>
    </f:metadata>

*f:viewParam*

Set el valor durante la fase _update model_.

El valor asignado no está disponible durante @PostConstruct, entonces necesitas
agegar un f:event dentro de f:metadata para inicializar los valores, o bien y más simple
puedes usar f:viewAction para el mismo fin.    

Permite anidar f:converter y f:validator para una más detallada conversión/validación; pudiéndose usar un h:message. Ejemplo:
{:lang="xhtml"}
    <f:metadata>
      <f:viewParam id="id" name="id" value="#{bean.id}" required="true">
        <f:validateLongRange minimum="10" maximum="20" />
      <f:viewParam>
      <f:viewAction action="#{bean.init}" />
    </f:metadata>
<h:message for="id" />

*@ManagedProperty*

Set los valores inmediátamente luego de la construcción del bean.    

El valor está disponible durante @PostConstruct lo cual permite inicialización de otras propiedades basadas sobre los valores.

Puedes [leer más](http://balusc.blogspot.com.es/2011/09/communication-in-jsf-20.html#ProcessingGETRequestParameters) sobre el uso y proceso de parámetros GET 

## CRUD: estadio final 

#### Una vista, ViewScoped, add, edit y delete

Partiendo del último estadio de la app conseguido en la clase anterior, tendríamos nuestra única vista con dos elementos h:form,
utilizando el primero para introducir datos y el segundo destinado a permitir la _modificación_ de datos de items ya existentes.
En un tercer formulario deberíamos mostrar los datos existentes a ese momento, podemos utilizar una tabla, y permitir la edición
de cualquiera de los items utilizando un botón o un enlace; sin preocuparnos de este último formulario, el index.xhtml lo encontramos
de esta manera,
{:lang="xhtml"}
    <h:panelGroup rendered="#{empty viewManager.cacheList}">
       <p>No hay datos aún.</p>
    </h:panelGroup>
    <h:panelGroup rendered="#{!viewManager.edit}">
       <h3>Alta de Datos</h3>
       <h:form>
          <p>Key:
            <h:inputText value="#{viewManager.item.key}" />
          </p>
          <p>Value:
            <h:inputText value="#{viewManager.item.value}" />
          </p>
          <p>
            <h:commandButton value="add" action="#{viewManager.add}" />
          </p>
       </h:form>
    </h:panelGroup>
    <h:panelGroup rendered="#{viewManager.edit}">
       <h3>Editar un item #{viewManager.item.key}</h3>
       <h:form>
          <p>Key:
             <h:inputText value="#{viewManager.item.key}" />
          </p>
          <p>Value:
             <h:inputText value="#{viewManager.item.value}" />
          </p>
          <p>
             <h:commandButton value="save" action="#{viewManager.save}" />
          </p>
       </h:form>
    </h:panelGroup>
    <h:form rendered="#{not empty viewManager.cacheList}">
            !--< Mostrar los datos en una tabla -->
    </h:form>
Hasta aquí, vemos que se renderiza "No hay datos aún", en caso que el miembro lista esté vacío y pretenderíamos que en ese caso
como no hay nada para editar por ausencia de items, se diera la posibilidad de dar de alta items, para ello, en la clase, el valor
de una _variable boolena edit_ debería ser _false_.

{:lang="java"}
    private boolena edit;
    // y el método que se ejecuta cuando el valor del atributo
    // rendered en un componente es "#{viewManager.edit}"
    public boolean isEdit() {
      return edit;
    }
Mientras que si _edit_ es _true_ la renderización es del segundo formulario. Ambos admiten datos mediante un elemento h:inputText,
la diferencia está en la acción a ejecutar tras el envío del formulario, en caso de edición es _save_, cuyo código es como sigue
{:lang="java"}
    public void save() {
      item = new Property();
      edit = false;
    }
Notamos que este método será ejecutado una vez el usuario haya tenido la posibilidad de _editar un item_, para ello se prevee como
se ha indicado anteriormente un botón en lo que sería el tercer formulario de index.xhtml, que describimos más abajo.    
A la variable _edit_, se le asigna _true_ al ejecutar la acción del botón de edición, produciendo el efecto de la renderización del
formulario de edición, así pues con la acción _save_, la edición ha terminado (se editó y se grabó), entonces asignamos _false_ a _edit_,
para que sea renderizado la parte _insertar datos_   
El formulario que se encarga de renderizar la tabla con los datos y permitir la edición de un item, podría tener esta forma
{:lang="xhtml}
    <h:form rendered="#{not empty viewManager.cacheList}">
      <h:dataTable value="#{viewManager.cacheList}" var="item"
         <h:column>
           <f:facet name="header">Key</f:facet>
           <h:outputText value="#{item.key}" />
         </h:column>
         <h:column>
           <f:facet name="header">Value</f:facet>
           <h:outputText value="#{item.value}" />
         </h:column>
         <h:column>
           <h:commandButton value="edit" action="#{viewManager.edit(item)}" />
         </h:column>
         <h:column>
      </h:dataTable>
    </h:form>
Vemos que el método de la clase a ejecutar tras el envío del cliente, la acción del h:commandButton, recibirá un parámetro el cual
es un objeto item dentro de la tabla.   
En los estadios de la app desarrollados la clase anterior se optó por mostrar los datos usando ui:repeat y h:outputText.    
Este es el código del _método edit_ de la clase,
{:lang="java"}
    public void edit(Property item) {
      this.item = item;
      edit = true;
    }
Por último podemos completar las acciones CRUD, agregando la posibilidad de _eliminar un item_, será definir en la clase un método
_delete_, el cuál será ejecutado mediante atributo action de un h:commandButton que añadimos al último formulario,
{:lang="java"}
    public void delete(Property item) {
        cacheList.remove(item);
    }
Y en el formulario retocamos el elemento h:dataTable
{:lang="xhtml"}
    <h:dataTable value="#{viewManager.cacheList}" var="item"
      <h:column>
        <f:facet name="header">Key</f:facet>
        <h:outputText value="#{item.key}" />
      </h:column>
      <h:column>
        <f:facet name="header">Value</f:facet>
        <h:outputText value="#{item.value}" />
      </h:column>
      <h:column>
        <h:commandButton value="edit" action="#{viewManager.edit(item)}" />
      </h:column>
      <h:column>
        <h:commandButton value="delete"
                         action="#{viewManager.delete(item)}" />
      </h:column>
    </h:dataTable>
