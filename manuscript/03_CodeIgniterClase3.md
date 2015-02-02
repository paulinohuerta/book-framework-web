# CodeIgniter Clase 3

En esta clase preesentamos dos técnicas de aplicación en la construcción de vistas, _presenters_ y _partials_, y otra para modelos,
los _observers_.   
Al principio desarrollamos una nueva acción CRUD para la aplicación que seguimos desde las clases anteriores. Es preciso recordar que
hemos definido el controlador MY_Controller y el modelo MY_Model, lo que seguiremos teniendo en cuenta para la nueva acción que consiste
en permitir la _modificación_ de una fila en la Base de datos, que desde la persepectiva del usuario es ofrecerle la posibilidad de _editar
un item_.

### Editando un item
La nueva ruta a tener en cuenta debe ser:
{:lang="php"}
    $route['pastes/edit/(:num)'] = 'paste/edit/$1';
Procedemos a codificar el *método edit* en el _controlador paste.php_
{lang:"php"}
    <?
    public function edit($id)
    {
      $this->data['show']=TRUE;
      $this->load->library('form_validation');
      $this->data['title'] = "Editando el item";
      $this->data['id'] = $id;
      $this->form_validation->set_rules('author', 'Author', 'required');
      $this->form_validation->set_rules('language', 'Language', 'required');
      $this->data['pastes_item'] = $this->paste->get($id);
      if (empty($this->data['pastes_item'])) {
         show_404();
      }
      if ($this->form_validation->run() === FALSE)
        {}
      else {
         $data = array (
              'author' => $this->input->post('author'),
              'language' => $this->input->post('language'),
              'description' => $this->input->post('description'),
              'body' => $this->input->post('body'),
         );
         if($this->paste->update($id,$data)) {
             $this->data['show']=FALSE;
         }
      }
    }
    ?>
Con la instrucción _$this->load->library('form_validation')_ hemos cargado la clase *form_validation*.    
Sabiendo que las librerías de CodeIgniter se encuentran en la carpeta system/libraries y que normalmente la cargaríamos usando la siguiente función
de inicialización:
{:lang="php"}
    $this->load->library('class_name');
Esta clase tiene métodos que nos permiten validar campos de entrada de datos en base a reglas que suministramos. En este punto deberías leer
la guía CodeIgniter para la entrada [for_validation](https://ellislab.com/codeigniter/user-guide/libraries/form_validation.html).   
En nuestro caso respetando los tres argumentos del método *set_rules*, queremos que el campo author y language no se dejen en blanco, es decir
que cuando el usuario edite un registro debe obligatoriamente rellenar estos dos campos.   
El método _run()_ retorna TRUE, en el caso que todas las reglas no detecten fallo. Cuando se renderiza el formulario por primera vez, aún no hay
entrada de datos, retorna FALSE.   
El código del _método edit_ termina cargando el array $data con las entradas del formulario, se ha usado la _clase input_, ésta es inicializada
automáticamente por el sistema con lo que no debemos preocuparnos y la usamos directamente.   
El uso de 
{:lang="php"}
    $something = $this->input->post('something');
    // consigue lo mismo que
    $something = $_POST['something'];
Luego la _clase input_ sirve para dos propuestas:
- Pre-procesar datos de entrada globales.
- Ofrecer algún _helper_ para conseguir la entrada desde un formulario y pre-procesarla, nosotros hemos usado _post()_, existen otros dos
_server()_ y _cookie()_.   
Tenemos una ventaja al usar $this->input->post('something'), que consiste en el chequeo sobre la existencia del item _something_, en caso que
no exista, retorna _false_, esto no ocurre si usamos para el _fetch_ del item $_POST, en concreto deberíamos tener:
{:lang="php"} 
    if ( ! isset($_POST['something'])) {
      $something = FALSE;
    }
    else {
      $something = $_POST['something'];
    }
Por último, la codificación de _edit()_ ejecuta $this->paste->update($id,$data) del modelo con lo que tomaría efecto la modificación en la
Base de Datos.  
Como vemos en nuestro método no se invoca a ninguna vista debido a que al ser una tarea repetida, por DRY se procede con ello en el controlador
MY_Controller el cual usamos y extendemos.   

### Presenters
 
Es frecuente el caso donde las vistas se tornan un tanto confusas según la app va agregando nuevas acciones; sobre este particular resultaría
interesante  una lectura de [este post](http://jamieonsoftware.com/post/59690276326/codeigniter-view-presenters), escrito por Jamie Rumbelow y
donde se observa en el ejemplo usado violación del principio DRY y MVC.    
En general *no debería haber lógica de negocio en las vistas*.    
_Presenters_ es una técnica usada en Rails que ayuda a conseguir un nuevo nivel de abstracción proporcionando *una representación
de clase del estado de una vista*.    
Entonces los _presenters_ pueden ser un camino para _esconder_ *lógica presentacional*.
Remplazaríamos una vista como view/paste/show.php
{:lang="php"}
    <?php
     echo '<title>'.$title.'</title>' ;
     echo '<h2>'. $pastes_item->description .'</h2>';
     echo '<strong>' .$pastes_item->language .'</strong>';
     echo '<pre><code>' .$pastes_item->body.'</code></pre>';
    ?>
por esta otra
{:lang="php"}
    <div<div id="paste">
      <h1><?= $paste->title() ?></h1>
      <p class="information">
         <strong>Name Author:</strong> <?= $paste->author() ?><br />
         <strong>Language:</strong> <?= $paste->language() ?><br />
         <strong>El código:</strong> <?=$paste->body() ?>
      </p>
    </div>
Crearíamos una carpeta _application/presenters_, en la cual ponemos el fichero _paste_presenter.php_, contendrá la clase presenter para
nuestro objet paste, así podemos extrer la lógica de nuestra vista poniéndola en la clase, por ejemplo el título los extraeríamos así,
{:lang="php"}
    public function title() {
       return $this->paste->description;
    }
Con lo que la clase nos quedaría como sigue
{:lang="php"}
    <?
    class Paste_Presenter {
     public function __construct($paste) {
      $this->paste = $paste;
      $this->ci =& get_instance();
     }
     public function title()
     {
      return $this->paste->description;
     }
     public function author()
     {
      return $this->paste->author ?: "N/A";
     }
     public function language()
     {
      return $this->paste->language ?: "N/A";
     }
     public function body()
     {
      return $this->paste->body ?: "N/A";
     }
     public function description()
     {
      return $this->paste->description ?: "N/A";
     }
    }
    ?>
Mientras que en el controlador en controllers/paste.php, para el método/acción _show()_ tendríamos:
{:lang="php"}
    public function show($id)
    {
     $this->data['paste'] = new Paste_Presenter($this->paste->get($id));
     $this->load->view('paste/show', $this->data);
    }
Como se ve la idea es en lugar de pasar a la vista directamente el _objeto paste_ desde nuestro modelo, primero lo _envolvemos_
en nuestro _presenter_.    
Con estoo el _presenter_ representa la _cara pública_ de los datos recuperados de la base de datos, en este caso una fila de la
tabla paste.    
Teniendo en cuenta que necesitamos cargar nuestro presenter para que pueda ser usado por todos los métodos tiene sentido cargarlo
al inicio del _controlador_, para esto necesitamos,
{:lang="php"}
    require_once APPPATH . 'presenters/paste_presenter.php';
y habríamos acabado.    
Agregando un nivel de abstracción hemos conseguido una vista sucinta y clara estéticamente, pero lo más importante es que 
hemos practicado _DRY_.

### Partials

A veces repetimos bloques de código, por ejemplo en la construcción de una aplicación CRUD _replicaríamos_ el código del formulario
para _agregar_ y _editar_ o también podríamos estar repitiendo código ya escrito antes cuando mostramos en una tabla los datos
extraídos de la base de datos de una fila. En el _espíritu de DRY_ podemos extrer el código replicado y ponerlo en algo así como
un *partial*.

Construyendo un pratial podemos eliminar código duplicado, la convención usaada es que u nobre comience con _underscoe_, luego
distinguiremos vistas llamadas directamente desde el controlador de los _partials_ que están pensados para ser usado por más
de una vista.

Crearíamos un fichero _partial_helper.php_ en la carpeta _application/helpers_, podríamos escribir una librería, pero sólo nos
interesa renderizar una funcionalidad, luego un _helper_ es suficiente.   
En nuestro helper definimos una simple y única función, la llamaremos _partial()_ y podrá tener el siguiente contenido:
{:lang="php"}
    <?
    function partial($name, $data, $loop = FALSE)
    {
      $output = "";
      if ($loop && is_array($data))
      {
        foreach ($data as $row)
        {
          $output .= get_instance()->load->view($name,
          array( 'row' => $row ), TRUE);
        }
      }
      else
      {
        $output = get_instance()->load->view($name, $data,TRUE);
      }
      return $output;
    }
    ?>
Y pdoría ser usado en una vista de esta manera:
{:lang="php"}
    <div id="paste">
      <h1><?=$title?></h1>
      <table>
         <?= partial('paste/_row', $result, TRUE) ?>
      </table>
Es invocado en el elemento html tabla el partial paste/_row, el cual tiene acceso a $result, el cual es un array de objetos que el
controlador ha obtenido del modelo. El fichero views/paste/_row tiene:
{:lang="php"}
    <tr>
      <td><?= $row->language ?></td>
      <td><?= $row->author ?></td>
      <td><?=$row->description?></td>
      <td><pre><code><?= $row->body ?></code></pre></td>
    </tr>
La idea es que este código sea reusado en otras vistas y evitar escribirlo nuevamente cuando otra vista deba mostrar una tabla con
el resultado de una consulta sobre la Base de Datos.   
Los datos en $result que usa la vista fueron obtenidos en el método del controlador por ejemplo así:
{lang="php"}
    $this->data['result'] = $this->paste->get_all();

### Observers y callbacks

Muchas veces necesitas alterar el modelo antes que nuevos datos sean insertados.   
¿Cómo podría hacerse esto?    
El patrón MVC indica que esta clases de operaciones deberían estar en el modelo. Para facilitar esto, MY_Model contiene una serie
de eliminación de callbacks/observers, que nos son otra cosa que _métodos que serán llamadas en ciertos puntos_.   

La lista completa de _observers_ es la siguiente: 

- $before_create
- $after_create
- $before_update
- $after_update
- $before_get
- $after_get
- $before_delete
- $after_delete

Son variables de instancia que normalmente están definidas en el nivel de clase. Son _arrays de métodos_ sobre esta clase a ser
llamados en ciertos puntos.

Un simple ejemplo de uso de observers: _la columna created_on de la tabla pastes queremos que siempre valga la fecha y hora actual_.   
Tendremos en cuenta crear un observer $before_create, donde se realiza la asignación de la fecha, que luego la acción insert() debe
respetar. 

{:lang="php"}
    class Paste extends MY_Model
    {
      public $before_create = array( 'timestamps' );
      protected function timestamps($paste)
      {
       $paste['created_at'] = $paste['updated_at'] = date('Y-m-d H:i:s');
       return $paste;
      }
    }
Recordando de retornar el mismo objeto que la función recibe, en el ejemplo $row. Cada observer sobreescribe los datos del predecesor,
secuencialmente, en el orden que los _observers_ han sido definidos.    
Los _observers_ también pueden tomar parámetros en sus nombres, similar a los formularios CodeIgniter (Form Validation library).    
Los parámetros son accedidos en $this->callback_parameters:
{:lang="php"}
    public $before_create = array( 'data_process(name)' );
    public $before_update = array( 'data_process(date)' );
    protected function data_process($row)
    {
     $row[$this->callback_parameters[0]] = $this->_process($row[$this->callback_parameters[0]]);
     return $row;
    }

Para seguir con el ejemplo deberíamos repasar nuestra situación, en este momento _Paste_ extiende MY_Model, y el contenido de MY_Model es
{:lang="php"}
    class Paste_model extends MY_Model
    {
    }
Pasaría a estar con el siguiente nuevo código:
{:lang="php"}
    class Paste extends MY_Model {
      public $before_create = array( 'timestamps' );
      protected function timestamps($paste) {
        $paste['created_on'] = date('Y-m-d H:i:s');
        return $paste;
      }
    }
Y luego dos modificaciones debemos realizar en la clase MY_Model, que como sabemos la tenemos en core/MY_Model.   
En el _método insert()_, deberíamos incluir
{:lanng="php"}
    $data = $this->trigger('before_create', $data);
    // justo antes de
    $success = $this->db->insert($this->_table, $data);
    // el resto del código del método
Y la otra modificación consiste en agregar la codificación del método _tigger()_ a la clase y grabar en core/MY_Model.   
{:lang="php"}
    public function trigger($event, $data = FALSE, $last = TRUE)
    {
      if (isset($this->$event) && is_array($this->$event))
      {
       foreach($this->$event as $method) {
         if (strpos($method, '('))
         {
           preg_match('/([a-zA-Z0-9\_\-]+)(\(([a-zA-Z0-9\_\-\., ]+)\))?/', $method, $matches);
           $method = $matches[1];
           $this->callback_parameters = explode(',', $matches[3]);
         }
         $data = call_user_func_array(array($this, $method), array($data, $last));
       }
      }
      return $data;
    }
Retocando el modelo a través de la clase MY_Model y la clase Paste, hemos agregado una función que llamamos _timestamps()_, la cual se ocupa
de asignar un valor concreto al array que luego será el parámetro del _método insert()_ es decir estamos preparando con esta asignación el 
valor a ser asignado en una de las columnas de la tabla pastes al ejecutar _insert()_.   