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

    Autoloading de vistas

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

    Autoloading de modelos

Podemos emplear una técnica bastante simple para seguir limpiando y poniendo más claro el código del controlador, consistiría en proveer un pequeña interfaz model-autoloading.

La idea es cargar modelo basado en convenciones, algunas de estas ya hemos usado.   
Recordemos: *singular_recurso_model.php* lo que sería para una tabla *users* user_model.php; mientras que para un modelo que manipula un ficheros debería ser file_model.php.

Además sabemos que accedemos a nuestro modelo así:   
$this->user->get_all();    
$this->file->upload();    

