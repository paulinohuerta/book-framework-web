# CodeIgniter Clase 2

## Configurar parámetros para la Base de datos

Hay un fichero de configuración en application/config/database.php que contiene valores como username, password, nombre de la database, los que sirven como
parámetros de conexión con la base de datos.   

Disponemos de un array asociativo para almacenar los valores de conexión

## Configurar rutas

Como puede haber una relación uno a uno entre el string de una URL y el controlador correspondiente clase/método. El segmento en una URI sigue el siguiente patrón:   
 
example.com/class/function/id/    
 
En ciertos casos, podría interesarte renunciar a esta relación, y necesitar
invocar a otra función, en lugar de la correspondiente a la URL.

Por ejemplo tus URLs tendrán este prototipo.
 
example.com/product/1/   
example.com/product/2/   

Normalmente el segundo segmento de la URI está reservado al nombre de función
pero en este caso es un id de producto. CodeIgniter permite modificar manipulando el mapeo de la URI.   

Las reglas de enrutamiento están definidas en application/config/routes.php.   
En un array llamado $routes se asignan nuestro propios criterios de rutas.
Pueden usarse expresiones regulares y/o metacaracteres para asignar las rutas.

Ejemplo:   

$route['product/:num'] = "catalog/product_lookup";

La key contiene la URI a ser cotejada, y el valor contiene el destino, al cual
debería ser re-enrutado.

(:num) coteja un segmento conteniendo sólo números.   
(:any) coteja un segmento conteniendo cualquier caracter.

## Trabajando sobre el controlador

Cuando analizamos el nivel o capa del *controlador* salta a la vista que es aquí
donde se enlazan las vistas y los modelos de nuestra aplicación.    
Nos vamos a esforzar en encontrar convenciones para producir un autoload de nuestras vistas y modelos intentando obtener un mecanismo aún más convencional.
Así también podemos ver la posibilidad de usar filtros con la idea de ejecutar
código pre- y post- acción.

1. Autoloading de vistas

Reflexionemos sobre esta posible convención:

Las vistas deberían ser puestas en una carpeta que sea llamada con el nombre del controlador y luego debería agregarse el nombre de la acción.    

Es decir, si usamos la URI /users/list tiene sentido cargar la vista en views/users/list.php. De hecho, puedes ya estar familiarizado con este modelo.

Con esta convención en mente podemos reducir la necesidad de llamar a $this->load->view() y hacer que esta se cargue automáticamente después que ejecuta el controlador. Esto nos llevará a tener un controlador más sucinto y claro.

Vamos a necesitar una característica de CodeIgniter, se trata de la función *_remap()*; es un método que de existir en un controlador, será llamado por CodeIgniter en lugar de la acción del controlador. Con este mecanismo, el desarrollador puede ejecutar funcionalidades antes y después del proceso de cada *acción*.    

{:lang="php"}
    <?
    class MY_Controller extends CI_Controller {
      public function __construct()
      {
        parent:__construc();
      }
    }
    ?>

Definimos nuestra función _remap(), recibe dos parámetros, $method que es el nombre de la acción,
y $parameters que son los parámetros obtenidos de los distintos segmentos de la ruta URL los que son pasados al controlador de la acción.

{:lang="php"}
    <?
    public function _remap($method, $parameters) {
      call_user_func_array(array($this,
                  $method), $parameters);
    }
    ?>

Deberíamos verificar que el método *exista*, es necesario asegurarlo, si esto no
ocurre, sería preciso responder con código 404.

{:lang="php"}
    if(method_exists($this,$method)
    {
      call_user_func_array(array($this, $method),
                                    $parameters);
    }
    else
    {
      show_404();
    }

Una vez llamado con éxito el método, buscaremos de construir el *nombre de la
vista* con el controlador y la acción.

{:lang="php"}
    $view = strtolower(get_class($this)) . '/' . $method;

También debemos pensar como pasamos los datos, podemos definir una variable al
nivel de instancia $this->data

{:lang="php"}
    public $data = array();

Y entonces podemos cargar la vista con:

{:lang="php"}
    $this->load->view($view, $this->data);

Ahora sí tenemos podemos tener un controlador como sigue:

{:lang="php"}
<?php
    class Pastes extends MY_Controller {
      public function __construct()
      {
        parent::__construct();
        $this->load->model('pastes_model');
      }
      public function index()
      {
        $this->data['pastes'] = $this->pastes_model->get_pastes();
        $this->data['title'] = 'Pastes archive';
      }
     }

No está mal, pues nos proponíamos *claridad*, y aún podemos ampliar la técnica para conseguir cargar el modelo 
también en el controlador.

2. Autoloading de modelos

Podemos emplear una técnica bastante simple para seguir limpiando el código del controlador, consistiría en proveer un pequeña interfaz model-autoloading.

La idea es cargar modelo basado en convenciones, algunas de estas ya hemos usado.   
Recordemos: *singular_recurso_model.php* lo que sería para una tabla *users* user_model.php; mientras que para un modelo que manipula un ficheros debería ser file_model.php.

Además sabemos que accedemos a nuestro modelo así:   
$this->user->get_all();    
$this->file->upload();    

Podríamos cargar automáticamente el modelo *asumiendo* que a nuestro controlador le corresponde un nombre de modelo en singular.    
En nuestro MY_Controller.php procedemos como sigue:    

{:lang="php"}
    public function __construct()
    {
      $this->load->helper('inflector');
      $model=strtolower(singular(get_class($this)));
       // Verificamos que el modelo exista, en este caso lo cargamos.
      if(file_exists(APPATH . 'models/' . $model . '_model.php'))
      {
         $this->load->model($model . '_model',$model);
      }
    }

Esto es lo que buscamos pero no resuelve nuestro problema ya que si necesitaramos trabajar también sobre otro modelo,¿cómo lo cargaríamos?.      
Habría una solución *agregando un array* al controlador, y que de forma automática agregara
*_model* y cargara *cada uno de los modelos puestos en el array*; llamemos $models al array:

{:lang="php"}
    public $models = array();

Y agregando un bucle:

{:lang="php"}
    foreach($this->models as $model) { 
      $this->load->model($model . '_model', $model);
    }

Es así como en nuestro controlador podemos especificar los modelos que necesitamos cargar, para ello usaremos:

{:lang="php"}
    public $models = array('user', 'project', 'benchmark');

Si comparamos esto a nuestro previo código, vemos que utilizamos el constuctor para estas acciones de carga:

{:lang="php"}
    $this->load->model('user_model','user');
    $this->load->model('project_model','project');
    $this->load->model('benchmark_model','benchmark');

Si bien es una sutil diferencia, trabajando en grandes aplicaciones, con extensos bloques de código y numerosos
ficheros, pequeños cambios como estos pueden hacer la diferencia.    
Pero lo que nos debe quedar en limpio que un código más conciso y sucinto es *más mantenible*.

## Un ejemplo de aplicación usando CI

Una vez nuestra andadura por MY_Model y MY_Controller podemos proponernos el código de
una aplicación que recoja las distintas cuestiones y aspectos hasta aquí comentados.

Trabajamos sobre la *tabla pastes* de la BD, y su estructura es la siguiente.

    CREATE TABLE pastes (
       id int(10) unsigned NOT NULL auto_increment,
       author varchar(20) NOT NULL,
       language varchar(20) NOT NULL,
       description varchar(50) default NULL,
       body text NOT NULL,
       created_on timestamp NOT NULL default CURRENT_TIMESTAMP
          on update CURRENT_TIMESTAMP,
       PRIMARY KEY  (id)
    ) ENGINE=InnoDB AUTO_INCREMENT=46 DEFAULT CHARSET=utf8;

Deseamos codificar una aplicación que tendrá en cuenta los aspectos y cuestiones siguientes,

* Usemos la convención nombre de clase del modelo en singular.
* Mostremos un índice de pastes, invocando en el modelo "get_all".
* Hagamos posible la selección de un paste, desde el índice, invocamos "get_view".
* Desde el índice dar posibilidad de poder insertar un nuevo paste.

_Modelo_: models/paste_model.php
{:lang="php"}
    <?
    class Paste_model extends MY_Model
    {
    }
    ?>

_Controlador_: controllers/paste.php
{:lang="php"}
    <?
    class Paste extends MY_Controller {
       protected $models = array( 'paste');
       protected $helpers = array( 'cookie', 'file' );
       public function index() {
            // arrastramos del ejemplo anterior el array $pp,
            // sabiendo que no tiene sentido en un índice como tal
         $pp=array('language' => 'Ruby',
             'author' => 'paulinohuerta', 'id >' => 9);
         $this->data['pastes'] = $this->paste->get_all($pp);
         $this->data['title'] = 'Pastes archive';
       }
       public function view($id) {
         $this->data['pastes_item'] = $this->paste->get($id);
         if (empty($this->data['pastes_item'])) {
            show_404();
         }
         $this->data['title'] = $this->data['pastes_item']['language'];
       }
       public function create() {
         $this->load->helper('form');
         $this->load->library('form_validation');
         $this->data['title'] = 'Create a news item';
         $this->form_validation->set_rules('author', 'Author', 'required');
         $this->form_validation->set_rules('language', 'Language', 'required');
         if ($this->form_validation->run() === FALSE)
         {   }
         else {
            $data = array (
                   'author' => $this->input->post('author'),
                   'language' => $this->input->post('language'),
                   'description' => $this->input->post('description'),
                   'body' => $this->input->post('body'),
            );
            if($this->paste->insert($data)) {
               $this->load->view('paste/success');
            }
            else {
	       echo "  Algo fue mal .....<br>";
	    }
         }
       }
    }
    ?>

_Vistas_: (1) views/paste/index.php
{:lang="php"}
    <?php foreach ($pastes as $pastes_item): ?>
     <h2><?php echo $pastes_item['author'] ?></h2>
     <div id="main">
       <?php echo $pastes_item['language'] ?>
       <?php echo $pastes_item['description'] ?>
     </div>
     <p><a href="pastes/<?php echo $pastes_item['id'] ?>">View paste</a></p>
    <?php endforeach ?>
    <hr>
    <p><a href="pastes/create">Nueva entrada</a></p>

_Vistas_: (2) views/paste/view.php
{:lang="php"}
    <?php
    echo '<h2>'.$pastes_item['description'].'</h2>';
    echo '<strong>' .$pastes_item['language'].'</strong>';
    echo '<pre><code>' .$pastes_item['body'].'</code></pre>';
    ?>

_Vistas_: (3) views/paste/create.php
{:lang="php"}
    <h2>Crear un nuevo paste</h2>
    
    <?php echo validation_errors(); ?>
    
    <?php echo form_open('pastes/create') ?>
      <label for="author">Author</label>
      <input type="input" name="author" /><br />
      <label for="language">Language</label>
      <input type="input" name="language" /><br />
      <label for="description">Description</label>
      <input type="input" name="description" /><br />
      <label for="body">Código</label>
      <textarea name="body"></textarea><br />
      <input type="submit" name="submit" value="Crear un nuevo item" />
    </form>

_Vistas_: (4) views/paste/success.php
{:lang="html"}
    <html>
      <title>Record added</title>
      <h1>Success</h1>
      <p>Insertado con éxito</p>
      <p><a href="../">Volver al índice</a></p>
    </html>

