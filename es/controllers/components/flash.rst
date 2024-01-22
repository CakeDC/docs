Flash
#####

.. php:namespace:: Cake\Controller\Component

.. php:class:: FlashComponent(ComponentCollection $collection, array $config = [])

FlashComponent proporciona una manera de establecer mensajes de notificación de una sola vez que se mostrarán después de procesar un formulario o reconocer datos. CakePHP se refiere a estos mensajes como "flash messages" (mensajes flash). FlashComponent escribe mensajes flash en ``$_SESSION``, para ser renderizados en una Vista usando
:doc:`FlashHelper </views/helpers/flash>`.

Configuración de Mensajes Flash
===============================

FlashComponent proporciona dos formas de establecer mensajes flash: su método mágico ``__call()`` y su método ``set()``. Para dotar a su aplicación de verbosidad, el método mágico ``__call()`` de FlashComponent le permite usar un nombre de método que se asigna a un elemento ubicado en el directorio templates/element/flash. Por convención, los métodos camelcased se asignarán al nombre del elemento en minúsculas y con guiones bajos::

    // Utiliza templates/element/flash/success.php
    $this->Flash->success('Esto fue un exito');

    // Utiliza templates/element/flash/great_success.php
    $this->Flash->greatSuccess('Esto fue un gran exito');

Alternativamente, para establecer un mensaje de texto plano sin renderizar un elemento, puede usar el método ``set()``::

    $this->Flash->set('Este es un mensaje');

Los mensajes flash se agregan a un array internamente. Llamadas sucesivas a
``set()`` o ``__call()`` con la misma clave agregarán los mensajes en la
``$_SESSION``. Si desea sobrescribir mensajes existentes al establecer un mensaje flash, establezca la opción clear en true al configurar el componente.

Los métodos ``__call()`` y ``set()`` de FlashComponent opcionalmente toman un segundo
parámetro, un array de opciones:

* ``key`` Por defecto es 'flash'. La clave del array que se encuentra bajo la clave Flash en la sesión.
* ``element`` Por defecto es null, pero se establecerá automáticamente al usar el método mágico __call(). El nombre del elemento a usar para renderizar.
* ``params`` Un array opcional de claves/valores para poner a disposición como variables dentro de un elemento.
* ``clear`` espera un bool y le permite borrar todos los mensajes en el stack actual y comenzar uno nuevo.

Un ejemplo de uso de estas opciones::

    // En su Controlador
    $this->Flash->success('El usuario ha sido guardado', [
        'key' => 'positivo',
        'clear' => true,
        'params' => [
            'nombre' => $usuario->nombre,
            'email' => $usuario->email,
        ],
    ]);

    // En su Vista
    <?= $this->Flash->render('positivo') ?>

    <!-- En templates/element/flash/success.php -->
    <div id="flash-<?= h($key) ?>" class="message-info success">
        <?= h($message) ?>: <?= h($params['nombre']) ?>, <?= h($params['email']) ?>.
    </div>

Tenga en cuenta que el parámetro `element` siempre se anulará al usar ``__call()``. Para recuperar un elemento específico de un plugin, debe establecer el parámetro plugin. Por ejemplo::

    // En su Controlador
    $this->Flash->warning('Mi mensaje', ['plugin' => 'NombreDelPlugin']);

El código anterior utilizará el elemento warning.php bajo
plugins/NombreDelPlugin/templates/element/flash para renderizar el mensaje flash.

.. note::

    Por defecto, CakePHP escapa el contenido en los mensajes flash para prevenir
    scripting entre sitios. Los datos de usuario en sus mensajes flash se codificarán en HTML y
    estarán seguros para ser impresos. Si desea incluir HTML en sus mensajes flash,
    debe pasar la opción ``escape`` y ajustar sus plantillas de mensajes flash
    para permitir desactivar el escape cuando se pasa la opción de escape.

HTML en Mensajes Flash
======================

Es posible imprimir HTML en mensajes flash usando la clave 'escape' en las opciones::

    $this->Flash->info(sprintf('<b>%s</b> %s', h($resaltar), h($mensaje)), ['escape' => false]);

Asegúrese de escapar manualmente la entrada, entonces. En el ejemplo anterior,
`$resaltar` y `$mensaje` son entradas no HTML y, por lo tanto, están escapadas.

Para obtener más información sobre cómo renderizar sus mensajes flash, consulte la
sección :doc:`FlashHelper </views/helpers/flash>`.
