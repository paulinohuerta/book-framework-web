# CodeIgniter Clase 1

## Directos al patrón MVC

En esta clase nos centramos en la M de MVC, vemos que podría ser lo mejor en el
modelo e intentaremos aprender el poder de este nivel y lo que debería ser; incluso
podemos inspirarnos en los patrones de Rails, y obtener algunas conclusiones acerca
de como las cosas pueden ser más eficientes.
Podemos hacer un rápido análisis de los errores más comunes en el contexto de CodeIgniter,
para solucionarlos e implementar mejores soluciones.

Dos conceptos que provienen del mundo Rails: (1) modelo grueso, controlador ligero.
(2) convención sobre configuración.

Nos dice que nosotros necesitamos escribir las aplicaciones alrededor de la lógica de
nuestros modelos, y de aquí viene esto de... cualquier código que no puedas justificar
ponerlo en alguna parte debería estar en el modelo.

El patrón de diseño MVC, nos enseña una serie de reglas para construir aplicaciones
más robustas y estructuradas.
Lo más importante de éstas es que el modelo contiene todo el código relacionado al proceso
de datos.
Nuestros modelos controlan los cambios y el estado de los datos.

La M del MVC significa cualquier código que almacena datos en la BD, que valide datos,
que envía emails, que conecta con APIs, que calcula estadísticas, etc.

¿Hay un Modelo estandard?

Detrás de cualquier app hay un conjunto de convenciones.
Los desarrolladores sólo especifican aspectos no convencionales de la app.
El resultado es que el desarrollador deba tomar menos decisiones.

Como desarrolladores podemos abstraernos de varias funcionalidades, y sabemos que
conseguiremos una app consistente en distintos ámbitos.

En estas convenciones se asumen cuestiones bastantes razonables, atendamos a estas:

1. Cada base de datos detrás del modelo mapea a una tabla de la base de datos. 
2. El nombre del modelo es singular, el nombre de la tabla plural.
3. Cada tabla contien una columna id.
4. Cada tabla contine dos columnas una created_at y otra updated_at.

Atendamos primero una codificación particular donde no usamos framework ni el patrón
MVC.

{:lang="php"}
    <html>
     <head>
      <title>Test Page</title>
     </head>
     <body>
      <?php
        $serverip="localhost";
        $username="user1";
        $password="xxxxx";
        $database="rebbin";
        $con=mysql_connect($serverip,$username,$password) or die("no puedo conectar con mysql server");
        @mysql_select_db($database) or die("no puedo seleccionar DB");
        $sql="select id from pastes order by created_on desc";
        $result=mysql_query($sql) or die("No puedo grab pastes");
        if(!mysql_num_rows($result)){
         echo "<p>No hay pastes en la BD</p>";
        }
        else {
          while($row=mysql_fetch_array($result)){
            print_r($row);
          }
        }
      ?>
     </body>
    </html>

## ¿Cómo sería esto empleando CI?

{:lang="php"}
    <?
     class Pastes_model Extends CI_Model{
      public function __construct()
      {
        $this->load->database();
      }

      function getpastes()
      {
        $this->db->select('id')->from('pastes')->order_by('id','desc');
        $query=$this->db->get();
        return $query->result_array();
      }
     }
    ?>

Observamos que el código es OO y hace uso de herencia, éste extiende una clase del
framework, el nombre que usamos es *Pastes_model*; el nombre de la tabla de la BD en
el backend es *pastes*.

Para el caso que nos ocupa basta definir un método getpastes().
Varias preguntas caben aquí.

1. Sobre que BD está trabajando nuestro código.
2. Cuál es el nombre del fichero que contiene esta clase.
3. ¿Es todo lo necesario?

Sobre (1) decir ...
Sobre (2) ....
Sobre (3) ....
