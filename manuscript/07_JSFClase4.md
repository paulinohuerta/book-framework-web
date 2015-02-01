# JSF Clase 4

Si bien, en esta clase nos ocupamos de modificar la clase aceptando acciones de modificación y
eliminación, al principio nos proponemos algunas indicaciones sobre el uso de parámetros.

La etiqueta _f:param_ permite pasar parámetros a un componente, en el siguiente ejemplo vemos
un uso de _f:param_ en un componente _h:outputFormat_

Tal como tenemos el código de _index.xhtml_ al final de la clase anterior, puedes agregar el
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
de la vista _cursoView.xhtml_.    
Con el componente _h:link_  siguient
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
	
---------------Podemos desarrollar @ManagedProperty-------------

Cuál sería la ddiferencia:

{:lang="java"}
    @ManagedProperty(value = "#{param.id}")
    private Integer id;

{:lang="xhtml"}
    <f:metadata>
      <f:viewParam name="id" value="#{someBean.id}"/>
    </f:metadata>

f:viewParam
Sets the value during update model values phase 

The set value is not available during @PostConstruct, so you need an additional <f:event type="preRenderView" listener="#{bean.init}" /> inside the <f:metadata> to do initialization/preloading based on the set values. Since JSF 2.2 you could use <f:viewAction> for that instead.

Ejemplo:
Allows for nested <f:converter> and <f:validator> for more fine-grained conversion/validation. Even a <h:message> can be attached.
<f:metadata>
    <f:viewParam id="id" name="id" value="#{bean.id}" required="true">
        <f:validateLongRange minimum="10" maximum="20" />
    </f:viewParam>
    <f:viewAction action="#{bean.init}" />
</f:metadata>
<h:message for="id" />

@ManagedProperty
SetsSets the value immediately after bean's construction the value immediately after bean's construction
Set value is available during @PostConstruct which allows easy initialization/preloading of other properties based on the set value.

http://bhttp://balusc.blogspot.com.es/2011/09/communication-in-jsf-20.html#ProcessingGETRequestParameters

## CRUD: estadio final 

#### Una vista, ViewScoped, add, edit y delete
