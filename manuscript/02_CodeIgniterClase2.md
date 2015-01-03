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

{:lang="php"}
