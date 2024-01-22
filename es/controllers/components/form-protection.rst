FormProtection
##############

.. php:class:: FormProtection(ComponentCollection $collection, array $config = [])

El Componente FormProtection proporciona protección contra la manipulación de datos de formularios.

Al igual que todos los componentes, se configura mediante varios parámetros configurables.
Todas estas propiedades se pueden establecer directamente o a través de métodos de configuración del mismo nombreen los métodos ``initialize()`` o ``beforeFilter()`` de su controlador.

Si está utilizando otros componentes que procesan datos de formularios en sus devoluciones de llamada ``startup()``, asegúrese de colocar el Componente FormProtection antes de esos componentes en su método ``initialize()``.

.. note::

    Al utilizar el Componente FormProtection, **debe** utilizar FormHelper para crear
    sus formularios. Además, **no** debe anular ninguno de los atributos "name" de los campos.
    El Componente FormProtection busca ciertos indicadores que son
    creados y gestionados por FormHelper (especialmente aquellos creados en
    :php:meth:`~Cake\\View\\Helper\\FormHelper::create()` y
    :php:meth:`~Cake\\View\\Helper\\FormHelper::end()`). Modificar dinámicamente
    los campos que se envían en una solicitud POST, como deshabilitar, eliminar
    o crear nuevos campos mediante JavaScript, probablemente causará que la validación del token de formulario
    falle.

Prevención de manipulación de formularios
=========================================

De forma predeterminada, el ``FormProtectionComponent`` evita que los usuarios manipulen
formularios de maneras específicas. Evitará las siguientes cosas:

* La acción (URL) del formulario no se puede modificar.
* No se pueden agregar campos desconocidos al formulario.
* No se pueden eliminar campos del formulario.
* No se pueden modificar los valores en los campos de entrada ocultos.

Evitar estos tipos de manipulación se logra trabajando con el ``FormHelper``
y haciendo un seguimiento de qué campos están en un formulario. También se hace un seguimiento
de los valores para los campos ocultos. Todos estos datos se combinan y se convierten en un hash y campos de token ocultos se insertan automáticamente en los formularios. Cuando se envía un formulario, el ``FormProtectionComponent`` utilizará los datos POST para construir la misma estructura y comparar el hash.

.. note::

    El ``FormProtectionComponent`` **no** evitará que se agreguen/cambien opciones de selección.
    Tampoco evitará que se agreguen/cambien opciones de radio.

Uso
===

La configuración del componente de protección de formularios se realiza generalmente en los métodos
``initialize()`` o ``beforeFilter()`` del controlador.

Las opciones disponibles son:

validate
    Establecer en ``false`` para omitir completamente la validación de solicitudes POST,
    esencialmente desactivando la validación del formulario.

unlockedFields
    Establecer a una lista de campos de formulario para excluir de la validación POST. Los campos pueden ser
    desbloqueados ya sea en el componente o con
    :php:meth:`FormHelper::unlockField()`. Los campos que han sido desbloqueados no son
    obligatorios para formar parte del POST y los campos ocultos desbloqueados no tienen
    sus valores verificados.

unlockedActions
    Acciones para excluir de las comprobaciones de validación POST.

validationFailureCallback
    Devolución de llamada para llamar en caso de fallo de validación. Debe ser un Closure válido.
    No está configurado por defecto, en cuyo caso se lanzará una excepción en caso de fallo de validación.

Desactivar las comprobaciones de manipulación de formularios
============================================================

::

    namespace App\Controller;

    use App\Controller\AppController;
    use Cake\Event\EventInterface;

    class WidgetsController extends AppController
    {
        public function initialize(): void
        {
            parent::initialize();

            $this->loadComponent('FormProtection');
        }

        public function beforeFilter(EventInterface $event)
        {
            parent::beforeFilter($event);

            if ($this->request->getParam('prefix') === 'Admin') {
                $this->FormProtection->setConfig('validate', false);
            }
        }
    }

El ejemplo anterior desactivaría la prevención de manipulación de formularios para rutas con prefijo de administrador.

Desactivar la manipulación de formularios para acciones específicas
===================================================================

Puede haber casos en los que desee desactivar la prevención de manipulación de formularios para una acción (por ejemplo, solicitudes AJAX). Puede "desbloquear" estas acciones enumerándolas en ``$this->FormProtection->setConfig('unlockedActions', ['edit']);`` en su ``beforeFilter()::``

::

    namespace App\Controller;

    use App\Controller\AppController;
    use Cake\Event\EventInterface;

    class WidgetController extends AppController
    {
        public function initialize(): void
        {
            parent::initialize();
            $this->loadComponent('FormProtection');
        }

        public function beforeFilter(EventInterface $event)
        {
            parent::beforeFilter($event);

            $this->FormProtection->setConfig('unlockedActions', ['edit']);
        }
    }

Este ejemplo desactivaría todas las comprobaciones de seguridad para la acción de edición.

Manejo de fallos de validación a través de devoluciones de llamada
==================================================================

Si falla la validación de la protección de formularios, por defecto resultará en un error 400.
Puede configurar este comportamiento estableciendo la opción de configuración ``validationFailureCallback``
a una función de devolución de llamada en el controlador.

Al configurar un método de devolución de llamada, puede personalizar cómo funciona el proceso de manejo de fallos::

    public function beforeFilter(EventInterface $event)
    {
        parent::beforeFilter($event);

        $this->FormProtection->setConfig(
            'validationFailureCallback',
            function (BadRequestException $exception) {
                // Puede devolver una instancia de respuesta o lanzar la excepción
                // recibida como argumento.
            }
        );
    }

.. meta::
    :title lang=es: FormProtection
    :keywords lang=es: parametros configurables,componente de protección de formularios,parametros de configuracion,características de protección,tighter security,clase php,método,array,envío,clase de seguridad,desactivar seguridad,desbloquear acciones
