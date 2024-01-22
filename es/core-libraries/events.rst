Sistema de eventos
##################

Crear aplicaciones mantenibles es tanto una ciencia como un arte. Es bien sabido que una clave para tener un código de buena calidad es hacer que tus objetos estén débilmente acoplados y fuertemente cohesivos al mismo tiempo. La cohesión significa que todos los métodos y propiedades de una clase están fuertemente relacionados con la clase en sí y no están tratando de hacer el trabajo que deberían hacer otros objetos, mientras que el acoplamiento débil es la medida de cuán "conectada" está una clase a objetos externos y cuánto depende de ellos.

Existen casos particulares en los que necesitas comunicarte de manera limpia con otras partes de una aplicación, sin tener que codificar dependencias de manera rígida, perdiendo así cohesión e incrementando el acoplamiento de clases. Utilizar el patrón Observer, que permite que los objetos notifiquen a otros objetos y a oyentes anónimos sobre cambios, es un patrón útil para lograr este objetivo.

Los oyentes en el patrón Observer pueden suscribirse a eventos y elegir actuar sobre ellos si son relevantes. Si has utilizado JavaScript, es probable que ya estés familiarizado con la programación basada en eventos.

CakePHP emula varios aspectos de cómo se desencadenan y gestionan eventos en bibliotecas populares de JavaScript como jQuery. En la implementación de CakePHP, se despacha un objeto de evento a todos los oyentes. El objeto de evento contiene información sobre el evento y proporciona la capacidad de detener la propagación del evento en cualquier momento. Los oyentes pueden registrarse ellos mismos o pueden delegar esta tarea a otros objetos y tienen la oportunidad de alterar el estado y el propio evento para el resto de las devoluciones de llamada.

El subsistema de eventos es el corazón de las devoluciones de llamada del Modelo, Comportamiento, Controlador, Vista y Ayudante. Si alguna vez has utilizado alguno de ellos, ya estás algo familiarizado con los eventos en CakePHP.

Uso Ejemplar de Eventos
=======================

Supongamos que estás construyendo un complemento de Carrito y te gustaría centrarte solo en manejar la lógica de pedidos. Realmente no quieres incluir la lógica de envío, enviar correos electrónicos al usuario o disminuir el ítem del stock, pero estas son tareas importantes para las personas que utilizan tu complemento. Si no estuvieras usando eventos, podrías intentar implementar esto adjuntando comportamientos a modelos o agregando componentes a tus controladores. Hacerlo representa un desafío la mayor parte del tiempo, ya que tendrías que idear el código para cargar externamente esos comportamientos o adjuntar ganchos a tus controladores de complementos.

En cambio, puedes usar eventos para permitirte separar limpiamente las preocupaciones de tu código y permitir que preocupaciones adicionales se conecten a tu complemento mediante eventos. Por ejemplo, en tu complemento de Carrito tienes un modelo Orders que se encarga de crear pedidos. Te gustaría notificar al resto de la aplicación que se ha creado un pedido. Para mantener limpio tu modelo Orders, podrías usar eventos::

    // Cart/Model/Table/OrdersTable.php
    namespace Cart\Model\Table;

    use Cake\Event\Event;
    use Cake\ORM\Table;

    class OrdersTable extends Table
    {
        public function place($order)
        {
            if ($this->save($order)) {
                $this->Cart->remove($order);
                $event = new Event('Order.afterPlace', $this, [
                    'order' => $order
                ]);
                $this->getEventManager()->dispatch($event);
                return true;
            }
            return false;
        }
    }

El código anterior te permite notificar a otras partes de la aplicación que se ha creado un pedido. Luego puedes realizar tareas como enviar notificaciones por correo electrónico, actualizar el stock, registrar estadísticas relevantes y otras tareas en objetos separados que se centran en esas preocupaciones.

Acceso a los Gestores de Eventos
================================

En CakePHP, los eventos se desencadenan contra gestores de eventos. Los gestores de eventos están disponibles en cada Table, View y Controller mediante ``getEventManager()``::

    $events = $this->getEventManager();

Cada modelo tiene un gestor de eventos separado, mientras que la Vista y el Controlador comparten uno. Esto permite que los eventos del modelo sean independientes y permitan que componentes o controladores actúen sobre eventos creados en la vista si es necesario.

Gestor de Eventos Global
------------------------

Además de los gestores de eventos a nivel de instancia, CakePHP proporciona un gestor de eventos global que te permite escuchar cualquier evento disparado en una aplicación. Esto es útil cuando adjuntar oyentes a una instancia específica puede ser engorroso o difícil. El gestor global es una instancia única de :php:class:`Cake\\Event\\EventManager`. Los oyentes adjuntos al despachador global se activarán antes que los oyentes de instancia con la misma prioridad. Puedes acceder al gestor global mediante un método estático::

    // En cualquier archivo de configuración o fragmento de código que se ejecute antes del evento
    use Cake\Event\EventManager;

    EventManager::instance()->on(
        'Order.afterPlace',
        $unCallback
    );

Una cosa importante que debes considerar es que hay eventos que se dispararán con el mismo nombre pero con sujetos diferentes, por lo que generalmente se requiere verificarlo en el objeto de evento en cualquier función que se adjunte globalmente para evitar algunos errores. Recuerda que con la flexibilidad de usar el gestor global, se incurre en cierta complejidad adicional.

El método :php:meth:`Cake\\Event\\EventManager::dispatch()` acepta el objeto de evento como argumento y notifica a todos los oyentes y devoluciones de llamada pasando este objeto.

.. _seguimiento-eventos:

Seguimiento de Eventos
----------------------

Para mantener una lista de eventos que se disparan en un determinado ``EventManager``, puedes habilitar el seguimiento de eventos. Para hacerlo, simplemente adjunta una :php:class:`Cake\\Event\\EventList` al gestor::

    EventManager::instance()->setEventList(new EventList());

Después de disparar un evento en el gestor, puedes recuperarlo de la lista de eventos::

    $eventosDisparados = EventManager::instance()->getEventList();
    $primerEvento = $eventosDisparados[0];

El seguimiento se puede desactivar eliminando la lista de eventos o llamando a :php:meth:`Cake\\Event\\EventList::trackEvents(false)`.

Eventos Principales
===================

Hay una serie de eventos principales dentro del framework a los que tu aplicación puede escuchar. Cada capa de CakePHP emite eventos que puedes utilizar en tu aplicación.

* :ref:`Eventos de ORM/Modelo <table-callbacks>`
* :ref:`Eventos de Controlador <controller-life-cycle>`
* :ref:`Eventos de Vista <view-events>`

.. _registro-oyentes-eventos:

Registro de Oyentes
===================

Los oyentes son la forma preferida de registrar devoluciones de llamada para un evento. Esto se hace implementando la interfaz :php:class:`Cake\\Event\\EventListenerInterface` en cualquier clase que desees registrar algunas devoluciones de llamada. Las clases que la implementen deben proporcionar el método ``implementedEvents()``. Este método debe devolver una matriz asociativa con todos los nombres de eventos que la clase manejará.

Para continuar con nuestro ejemplo anterior, imaginemos que tenemos una clase UserStatistic encargada de calcular el historial de compras de un usuario y compilar estadísticas globales del sitio. Este es un buen lugar para usar una clase oyente. Al hacerlo, puedes concentrar la lógica de estadísticas en un solo lugar y reaccionar a eventos según sea necesario. Nuestro oyente ``UserStatistics`` podría comenzar así::

    namespace App\Event;

    use Cake\Event\EventListenerInterface;

    class UserStatistic implements EventListenerInterface
    {
        public function implementedEvents(): array
        {
            return [
                // Nombres de eventos personalizados te permiten diseñar los eventos de tu aplicación
                // según sea necesario.
                'Order.afterPlace' => 'updateBuyStatistic',
            ];
        }

        public function updateBuyStatistic($event)
        {
            // Código para actualizar las estadísticas
        }
    }

    // Desde tu controlador, adjunta el objeto UserStatistic al gestor de eventos del Pedido
    $estadisticas = new UserStatistic();
    $this->Orders->getEventManager()->on($estadisticas);

Como puedes ver en el código anterior, la función ``on()`` aceptará instancias de la interfaz ``EventListener``. Internamente, el gestor de eventos utilizará ``implementedEvents()`` para adjuntar las devoluciones de llamada correctas.

Registro de Oyentes Anónimos
----------------------------

Si bien los objetos de oyente de eventos son generalmente una mejor manera de implementar oyentes, también puedes vincular cualquier "callable" como un oyente de eventos. Por ejemplo, si quisiéramos poner todos los pedidos en los archivos de registro, podríamos usar una función anónima simple para hacerlo::

    use Cake\Log\Log;

    // Desde un controlador, o durante el inicio de la aplicación.
    $this->Orders->getEventManager()->on('Order.afterPlace', function ($event) {
        Log::write(
            'info',
            'Se realizó un nuevo pedido con id: ' . $event->getSubject()->id
        );
    });

Además de las funciones anónimas, puedes usar cualquier otro tipo "callable" que PHP admita::

    $eventos = [
        'envio-correo' => 'EmailSender::sendBuyEmail',
        'inventario' => [$this->InventoryManager, 'decrementar'],
    ];
    foreach ($eventos as $llamable) {
        $gestorEventos->on('Order.afterPlace', $llamable);
    }

Cuando trabajas con complementos que no desencadenan eventos específicos, puedes aprovechar los oyentes de eventos en los eventos predeterminados. Tomemos un ejemplo de un complemento 'UserFeedback' que maneja formularios de retroalimentación de usuarios. Desde tu aplicación, te gustaría saber cuándo se ha guardado un registro de retroalimentación y, en última instancia, actuar en consecuencia. Puedes escuchar el evento global ``Model.afterSave``. Sin embargo, también puedes tomar un enfoque más directo y escuchar solo el evento que realmente necesitas::

    // Puedes crear lo siguiente antes de la
    // operación de guardado, es decir, config/bootstrap.php
    use Cake\Datasource\FactoryLocator;
    // Si se envían correos electrónicos
    use Cake\Mailer\Email;

    FactoryLocator::get('Table')->get('ThirdPartyPlugin.Feedbacks')
        ->getEventManager()
        ->on('Model.afterSave', function($event, $entity)
        {
            // Por ejemplo, podemos enviar un correo electrónico al administrador
            $correo = new Email('default');
            $correo->setFrom(['info@tusitio.com' => 'Tu Sitio'])
                ->setTo('admin@tusitio.com')
                ->setSubject('Nueva Retroalimentación - Tu Sitio')
                ->send('Cuerpo del mensaje');
        });

Puedes usar este mismo enfoque para vincular objetos de oyente.

Interactuar con Oyentes Existentes
----------------------------------

Supongamos que se han registrado varios oyentes de eventos y se desea realizar alguna acción basada en la presencia o ausencia de un patrón de evento en particular::

    // Adjuntar oyentes al gestor de eventos.
    $this->getEventManager()->on('User.Registration', [$this, 'userRegistration']);
    $this->getEventManager()->on('User.Verification', [$this, 'userVerification']);
    $this->getEventManager()->on('User.Authorization', [$this, 'userAuthorization']);

    // En otro lugar de tu aplicación.
    $eventos = $this->getEventManager()->matchingListeners('Verification');
    if (!empty($eventos)) {
        // Realizar lógica relacionada con la presencia del oyente 'Verification'.
        // Por ejemplo, eliminar el oyente si está presente.
        $this->getEventManager()->off('User.Verification');
    } else {
        // Realizar lógica relacionada con la ausencia del oyente 'Verification'.
    }

.. note::

    El patrón pasado al método ``matchingListeners`` es sensible a mayúsculas y minúsculas.

.. _prioridades-eventos:

Establecimiento de Prioridades
------------------------------

En algunos casos, es posible que desees controlar el orden en que se invocan los oyentes. Por ejemplo, si volvemos a nuestro ejemplo de estadísticas de usuario, sería ideal que este oyente se llamara al final de la pila. Al colocarlo al final de la pila de oyentes, podemos asegurarnos de que el evento no se haya cancelado y de que ningún otro oyente haya generado excepciones. También podemos obtener el estado final de los objetos en el caso de que otros oyentes hayan modificado el objeto o el objeto del evento.

Las prioridades se definen como un número entero al agregar un oyente. Cuanto mayor sea el número, más tarde se ejecutará el método. La prioridad predeterminada para todos los oyentes es ``10``. Si necesitas que tu método se ejecute antes, cualquier valor por debajo de esta prioridad predeterminada funcionará. Por otro lado, si deseas que la devolución de llamada se ejecute después de las demás, utilizar un número superior a ``10`` funcionará.

Si dos devoluciones de llamada tienen el mismo valor de prioridad, se ejecutarán en el orden en que se adjuntaron. Puedes establecer prioridades utilizando el método ``on()`` para devoluciones de llamada y declarándolo en la función ``implementedEvents()`` para los objetos de escucha de eventos::

    // Estableciendo prioridad para una devolución de llamada
    $devolucionLlamada = [$this, 'hacerAlgo'];
    $this->getEventManager()->on(
        'Order.afterPlace',
        ['prioridad' => 2],
        $devolucionLlamada
    );

    // Estableciendo prioridad para un objeto de escucha
    class EstadisticaUsuario implements EventListenerInterface
    {
        public function implementedEvents()
        {
            return [
                'Order.afterPlace' => [
                    'callable' => 'actualizarEstadisticasCompra',
                    'prioridad' => 100
                ],
            ];
        }
    }

Como puedes ver, la principal diferencia para los objetos ``EventListener`` es que necesitas usar un array para especificar el método llamable y la preferencia de prioridad. La clave ``callable`` es una entrada especial de array que el gestor leerá para saber qué función de la clase debe llamar.

Obtención de Datos de Evento como Parámetros de Función
-------------------------------------------------------

Cuando los eventos tienen datos proporcionados en su constructor, los datos proporcionados se convierten en argumentos para los oyentes. Un ejemplo de la capa de Vista es la devolución de llamada ``afterRender``::

    $this->getEventManager()
        ->dispatch(new Event('View.afterRender', $this, ['vista' => $nombreArchivoVista]));

Los oyentes de la devolución de llamada ``View.afterRender`` deberían tener la siguiente firma::

    function (EventInterface $event, $nombreArchivoVista)

Cada valor proporcionado al constructor de Evento se convertirá en parámetros de función en el orden en que aparecen en la matriz de datos. Si utilizas un array asociativo, el resultado de ``array_values`` determinará el orden de los argumentos de la función.

Despachando Eventos
===================

Una vez que hayas obtenido una instancia de un gestor de eventos, puedes despachar eventos utilizando :php:meth:`~Cake\\Event\\EventManager::dispatch()`. Este método toma una instancia de la clase :php:class:`Cake\\Event\\Event`. Veamos cómo se despacha un evento:

::

    // Una escucha de eventos debe ser instanciado antes de despachar un evento.
    // Crea un nuevo evento y lanzalo.
    $event = new Event('Order.afterPlace', $this, ['order' => $order]);
    $this->getEventManager()->dispatch($event);

:php:class:`Cake\\Event\\Event` acepta 3 argumentos en su constructor. El primero es el nombre del evento; debes tratar de mantener este nombre tan único como sea posible, al mismo tiempo que lo haces legible. Sugerimos una convención como sigue: ``Capa.nombreEvento`` para eventos generales que ocurren a nivel de capa (por ejemplo, ``Controller.startup``, ``View.beforeRender``) y ``Capa.Clase.nombreEvento`` para eventos que ocurren en clases específicas en una capa, por ejemplo, ``Modelo.Usuario.despuesRegistro`` o ``Controlador.Cursos.accesoInvalido``.

El segundo argumento es el ``sujeto``, lo que significa el objeto asociado al evento; generalmente, cuando es la misma clase la que dispara eventos sobre sí misma, usar ``$this`` será el caso más común. Aunque un componente también podría activar eventos de controlador. La clase del sujeto es importante porque los escuchas tendrán acceso inmediato a las propiedades del objeto y tendrán la oportunidad de inspeccionarlas o cambiarlas sobre la marcha.

Finalmente, el tercer argumento es cualquier dato adicional del evento. Esto puede ser cualquier dato que consideres útil pasar para que los escuchas puedan actuar sobre él. Aunque esto puede ser un argumento de cualquier tipo, recomendamos pasar un array asociativo.

El método :php:meth:`~Cake\\Event\\EventManager::dispatch()` acepta un objeto de evento como argumento y notifica a todos los escuchas suscritos.

.. _stopping-events:

Deteniendo Eventos
------------------

Al igual que con los eventos del DOM, es posible que desees detener un evento para evitar que se notifiquen oyentes adicionales. Puedes ver esto en acción durante las devoluciones de llamada del modelo (por ejemplo, antes de guardar), donde es posible detener la operación de guardado si el código detecta que no puede continuar.

Para detener eventos, puedes devolver ``false`` en tus devoluciones de llamada o llamar al método ``stopPropagation()`` en el objeto de evento::

    public function hacerAlgo($evento)
    {
        // ...
        return false; // Detiene el evento
    }

    public function actualizarEstadisticasCompra($evento)
    {
        // ...
        $evento->stopPropagation();
    }

Detener un evento evitará que se llamen a devoluciones de llamada adicionales. Además, el código que activa el evento puede comportarse de manera diferente según si el evento se detuvo o no. En general, no tiene sentido detener eventos 'después', pero detener eventos 'antes' se usa a menudo para evitar que ocurra toda la operación.

Para verificar si se detuvo un evento, puedes llamar al método ``isStopped()`` en el objeto de evento::

    public function realizarPedido($orden)
    {
        $evento = new Event('Order.beforePlace', $this, ['order' => $orden]);
        $this->getEventManager()->dispatch($evento);
        if ($evento->isStopped()) {
            return false;
        }
        if ($this->Orders->save($orden)) {
            // ...
        }
        // ...
    }

En el ejemplo anterior, la orden no se guardaría si el evento se detiene durante el proceso ``beforePlace``.

Obteniendo Resultados de Eventos
--------------------------------

Cada vez que una devolución de llamada devuelve un valor no nulo ni falso, se almacena en la propiedad ``$result`` del objeto de evento. Esto es útil cuando deseas permitir que las devoluciones de llamada modifiquen la ejecución del evento. Volvamos a nuestro ejemplo de ``beforePlace`` y permitamos que las devoluciones de llamada modifiquen los datos de ``$order``.

Los resultados del evento se pueden modificar tanto usando directamente la propiedad de resultado del objeto de evento como devolviendo el valor en la devolución de llamada en sí::

    // Una devolución de llamada de un escucha
    public function hacerAlgo($evento)
    {
        // ...
        $datosModificados = $evento->getData('order') + $masDatos;
        return $datosModificados;
    }

    // Otra devolución de llamada de un escucha
    public function hacerOtraCosa($evento)
    {
        // ...
        $evento->setResult(['order' => $datosModificados] + $this->result());
    }

    // Utilizando el resultado del evento
    public function realizarPedido($orden)
    {
        $evento = new Event('Order.beforePlace', $this, ['order' => $orden]);
        $this->getEventManager()->dispatch($evento);
        if (!empty($evento->getResult()['order'])) {
            $orden = $evento->getResult()['order'];
        }
        if ($this->Orders->save($orden)) {
            // ...
        }
        // ...
    }

Es posible modificar cualquier propiedad del objeto de evento y pasar los nuevos datos a la siguiente devolución de llamada. En la mayoría de los casos, proporcionar objetos como datos o resultado del evento y modificar directamente el objeto es la mejor solución, ya que la referencia se mantiene igual y las modificaciones se comparten en todas las llamadas de devolución de llamada.

Eliminación de Devoluciones de Llamada y Escuchas
-------------------------------------------------

Si por alguna razón deseas eliminar cualquier devolución de llamada del gestor de eventos, simplemente llama al método :php:meth:`Cake\\Event\\EventManager::off()` utilizando como argumentos los dos primeros parámetros que usaste para adjuntarlo::

    // Adjuntando una función
    $this->getEventManager()->on('Mi.evento', [$this, 'hacerAlgo']);

    // Desadjuntando la función
    $this->getEventManager()->off('Mi.evento', [$this, 'hacerAlgo']);

    // Adjuntando una función anónima
    $miFuncion = function ($evento) { ... };
    $this->getEventManager()->on('Mi.evento', $miFuncion);

    // Desadjuntando la función anónima
    $this->getEventManager()->off('Mi.evento', $miFuncion);

    // Agregando un EventListener
    $escucha = new MiEscuchaEvento();
    $this->getEventManager()->on($escucha);

    // Desadjuntando una clave de evento única de un escucha
    $this->getEventManager()->off('Mi.evento', $escucha);

    // Desadjuntando todas las devoluciones de llamada implementadas por un escucha
    $this->getEventManager()->off($escucha);

Los eventos son una excelente manera de separar preocupaciones en tu aplicación y hacer que las clases sean cohesivas y desacopladas entre sí. Los eventos pueden utilizarse para desacoplar el código de la aplicación y hacer que los complementos sean extensibles.

Ten en cuenta que con un gran poder viene una gran responsabilidad. El uso excesivo de eventos puede dificultar la depuración y requerir pruebas de integración adicionales.

Lecturas Adicionales
====================

* :doc:`/orm/behaviors`
* :doc:`/console-commands/commands`
* :doc:`/controllers/components`
* :doc:`/views/helpers`
* :ref:`testing-events`

.. meta::
    :title lang=es: Sistema de Eventos
    :keywords lang=es: eventos, despacho, desacoplamiento, cakephp, devoluciones de llamada, desencadenadores, ganchos, php
