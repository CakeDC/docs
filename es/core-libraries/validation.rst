Validación
##########

.. php:namespace:: Cake\Validation

El paquete de validación en CakePHP proporciona características para construir validadores que pueden validar matrices arbitrarias de datos con facilidad. Puedes encontrar una `lista de reglas de validación disponibles en la API
<https://api.cakephp.org/5.x/class-Cake.Validation.Validation.html>`__.

.. _creating-validators:

Creación de Validadores
=======================

.. php:class:: Validator

Los objetos Validator definen las reglas que se aplican a un conjunto de campos. Los objetos Validator contienen un mapeo entre campos y conjuntos de validación. A su vez, los conjuntos de validación contienen una colección de reglas que se aplican al campo al que están adjuntas. Crear un validador es simple::

    use Cake\Validation\Validator;

    $validator = new Validator();

Una vez creado, puedes comenzar a definir conjuntos de reglas para los campos que deseas validar::

    $validator
        ->requirePresence('title')
        ->notEmptyString('title', 'Por favor, completa este campo')
        ->add('title', [
            'length' => [
                'rule' => ['minLength', 10],
                'message' => 'Los títulos deben tener al menos 10 caracteres de longitud',
            ]
        ])
        ->allowEmptyDateTime('published')
        ->add('published', 'boolean', [
            'rule' => 'boolean',
        ])
        ->requirePresence('body')
        ->add('body', 'length', [
            'rule' => ['minLength', 50],
            'message' => 'Los artículos deben tener un cuerpo sustancial.',
        ]);

Como se muestra en el ejemplo anterior, los validadores se construyen con una interfaz fluida que te permite definir reglas para cada campo que desees validar.

Hubo algunos métodos llamados en el ejemplo anterior, así que repasemos las diversas características. El método ``add()`` te permite agregar nuevas reglas a un validador. Puedes agregar reglas individualmente o en grupos, como se muestra arriba.

Exigir la Presencia del Campo
-----------------------------

El método ``requirePresence()`` requiere que el campo esté presente en cualquier
matriz validada. Si el campo está ausente, la validación fallará. El
método ``requirePresence()`` tiene 4 modos:

* ``true`` La presencia del campo siempre es requerida.
* ``false`` La presencia del campo no es requerida.
* ``create`` La presencia del campo es requerida al validar una operación de **creación**.
* ``update`` La presencia del campo es requerida al validar una operación de **actualización**.

Por defecto, se utiliza ``true``. La presencia de la clave se verifica utilizando
``array_key_exists()`` para que los valores nulos cuenten como presentes. Puedes configurar
el modo usando el segundo parámetro::

    $validator->requirePresence('author_id', 'create');

Si tienes múltiples campos que son requeridos, puedes definirlos como una lista::

    // Define varios campos para la creación
    $validator->requirePresence(['author_id', 'title'], 'create');

    // Define varios campos para modos mixtos
    $validator->requirePresence([
        'author_id' => [
            'mode' => 'create',
            'message' => 'Se requiere un autor.',
        ],
        'published' => [
            'mode' => 'update',
            'message' => 'Se requiere el estado publicado.',
        ],
    ]);

Permitir Campos Vacíos
----------------------

Los validadores ofrecen varios métodos para controlar qué campos aceptan valores vacíos y
qué valores vacíos son aceptados y no se envían a otras reglas de validación para
el campo nombrado. CakePHP proporciona soporte para valores vacíos para diferentes formas
de datos:

#. ``allowEmptyString()`` Debe ser utilizado cuando solo deseas aceptar
   una cadena vacía.
#. ``allowEmptyArray()`` Debe ser utilizado cuando deseas aceptar un arreglo.
#. ``allowEmptyDate()`` Debe ser utilizado cuando deseas aceptar una cadena vacía,
   o un arreglo que se convierte en un campo de fecha.
#. ``allowEmptyTime()`` Debe ser utilizado cuando deseas aceptar una cadena vacía,
   o un arreglo que se convierte en un campo de tiempo.
#. ``allowEmptyDateTime()`` Debe ser utilizado cuando deseas aceptar una cadena vacía
   o un arreglo que se convierte en un campo de fecha y hora o de marca de tiempo.
#. ``allowEmptyFile()`` Debe ser utilizado cuando deseas aceptar un arreglo que
   contenga un archivo subido vacío.

También puedes usar los siguientes validadores específicos: ``notEmptyString()``, ``notEmptyArray()``, ``notEmptyFile()``, ``notEmptyDate()``, ``notEmptyTime()``, ``notEmptyDateTime()``.

Los métodos ``allowEmpty*`` admiten un parámetro ``when`` que te permite controlar
cuándo un campo puede o no puede estar vacío:

* ``false`` El campo no está permitido estar vacío.
* ``create`` El campo puede estar vacío al validar una operación de **creación**.
* ``update`` El campo puede estar vacío al validar una operación de **actualización**.
* Un callback que devuelve ``true`` o ``false`` para indicar si un campo está
  permitido estar vacío. Consulta la sección :ref:`conditional-validation` para obtener ejemplos sobre
  cómo usar este parámetro.

Un ejemplo de estos métodos en acción es::

    $validator->allowEmptyDateTime('published')
        ->allowEmptyString('title', 'El título no puede estar vacío', false)
        ->allowEmptyString('body', 'El cuerpo no puede estar vacío', 'update')
        ->allowEmptyFile('header_image', 'update');
        ->allowEmptyDateTime('posted', 'update');

Agregando Reglas de Validación
------------------------------

La clase ``Validator`` proporciona métodos que hacen que la construcción de validadores sea simple y expresiva. Por ejemplo, agregar reglas de validación a un nombre de usuario podría parecerse a esto::

    $validator = new Validator();
    $validator
        ->email('username')
        ->ascii('username')
        ->lengthBetween('username', [4, 8]);

Consulta la `documentación de la API de Validator
<https://api.cakephp.org/5.x/class-Cake.Validation.Validator.html>`_ para ver el conjunto completo de métodos de validación.

.. _custom-validation-rules:

Usando Reglas de Validación Personalizadas
------------------------------------------

Además de utilizar métodos en el ``Validator``, y provenientes de proveedores, también
puedes utilizar cualquier llamable, incluyendo funciones anónimas, como reglas de validación::

    // Usa una función global
    $validator->add('title', 'custom', [
        'rule' => 'validate_title',
        'message' => 'El título no es válido'
    ]);

    // Usa un llamable de arreglo que no está en un proveedor
    $validator->add('title', 'custom', [
        'rule' => [$this, 'method'],
        'message' => 'El título no es válido'
    ]);

    // Usa un cierre
    $extra = 'Alguno valor adicional necesario dentro del cierre';
    $validator->add('title', 'custom', [
        'rule' => function ($value, $context) use ($extra) {
            // Lógica personalizada que devuelve true/false
        },
        'message' => 'El título no es válido'
    ]);

    // Usa una regla de un proveedor personalizado
    $validator->add('title', 'custom', [
        'rule' => 'customRule',
        'provider' => 'custom',
        'message' => 'El título no es suficientemente único'
    ]);

Los cierres o métodos llamables recibirán 2 argumentos cuando se llamen. El primero
será el valor para el campo que se está validando. El segundo es un arreglo de contexto
que contiene datos relacionados con el proceso de validación:

- **data**: Los datos originales pasados al método de validación, útiles si
  planeas crear reglas comparando valores.
- **providers**: La lista completa de objetos proveedores de reglas, útil si
  necesitas crear reglas complejas llamando a múltiples proveedores.
- **newRecord**: Si la llamada de validación es para un nuevo registro o
  uno preexistente.

Los cierres deben devolver verdadero (boolean true) si la validación pasa. Si falla,
devuelve falso (boolean false) o para un mensaje de error personalizado devuelve una cadena (string), consulta la sección :ref:`Mensajes de Error Condicional/Dinámicos <dynamic_validation_error_messages>` para más detalles.

.. _dynamic_validation_error_messages:

Mensajes de Error Condicionales/Dinámicos
-----------------------------------------

Los métodos de reglas de validación, ya sean :ref:`llamables personalizados <custom-validation-rules>`,
o :ref:`métodos proporcionados por proveedores <adding-validation-providers>`, pueden devolver
un booleano, indicando si la validación fue exitosa, o pueden devolver
una cadena, lo que significa que la validación falló, y que la cadena devuelta
debe ser usada como el mensaje de error.

Los posibles mensajes de error existentes definidos a través de la opción ``message`` serán
sobrescritos por los devueltos desde el método de regla de validación::

    $validator->add('length', 'custom', [
        'rule' => function ($value, $context) {
            if (!$value) {
                return false;
            }

            if ($value < 10) {
                return 'Mensaje de error cuando el valor es menor que 10';
            }

            if ($value > 20) {
                return 'Mensaje de error cuando el valor es mayor que 20';
            }

            return true;
        },
        'message' => 'Mensaje de error genérico usado cuando se devuelve `false`'
    ]);

.. _conditional-validation:

Validación Condicional
----------------------

Al definir reglas de validación, puedes usar la clave ``on`` para definir cuándo
se debe aplicar una regla de validación. Si se deja sin definir, la regla siempre se
aplicará. Otros valores válidos son ``create`` y ``update``. Usar uno de estos
valores hará que la regla se aplique solo a operaciones de creación o actualización.

Además, puedes proporcionar una función llamable que determinará si una regla en particular
debe aplicarse o no::

    $validator->add('picture', 'file', [
        'rule' => ['mimeType', ['image/jpeg', 'image/png']],
        'on' => function ($context) {
            return !empty($context['data']['show_profile_picture']);
        }
    ]);

Puedes acceder a los otros valores de campo enviados usando el arreglo ``$context['data']``.
El ejemplo anterior hará que la regla para 'picture' sea opcional dependiendo de
si el valor de ``show_profile_picture`` está vacío. También podrías usar la
regla de validación ``uploadedFile`` para crear entradas de carga de archivos opcionales::

    $validator->add('picture', 'file', [
        'rule' => ['uploadedFile', ['optional' => true]],
    ]);

Los métodos ``allowEmpty*``, ``notEmpty*`` y ``requirePresence()`` también aceptarán
una función de devolución de llamada como su último argumento. Si está presente, la función de devolución de llamada
determina si la regla debe aplicarse o no. Por ejemplo, a veces se permite que un campo esté vacío::

    $validator->allowEmptyString('tax', 'Este campo es requerido', function ($context) {
        return !$context['data']['is_taxable'];
    });

De manera similar, un campo puede ser requerido para estar poblado cuando ciertas condiciones
se cumplen::

    $validator->notEmptyString('email_frequency', 'Este campo es requerido', function ($context) {
        return !empty($context['data']['wants_newsletter']);
    });

En el ejemplo anterior, el campo ``email_frequency`` no puede dejarse vacío si el
usuario desea recibir el boletín.

Además, también es posible requerir que un campo esté presente solo bajo ciertas
condiciones::

    $validator->requirePresence('full_name', function ($context) {
        if (isset($context['data']['action'])) {
            return $context['data']['action'] === 'subscribe';
        }
        return false;
    });
    $validator->requirePresence('email');

Esto requeriría que el campo ``full_name`` esté presente solo en caso de que el usuario
desea crear una suscripción, mientras que el campo ``email`` siempre sería
requerido.

El parámetro ``$context`` pasado a las devoluciones de llamada condicionales personalizadas contiene las
siguientes claves:

* ``data`` Los datos que se están validando.
* ``newRecord`` un booleano que indica si se está validando un registro nuevo o existente.
* ``field`` El campo actual que se está validando.
* ``providers`` Los proveedores de validación adjuntos al validador actual.


Marcando Reglas como las Últimas en Ejecutarse
----------------------------------------------

Cuando los campos tienen múltiples reglas, cada regla de validación se ejecutará incluso si el
anterior ha fallado. Esto te permite recopilar tantos errores de validación como
puedas en un solo paso. Si deseas detener la ejecución después de
que una regla específica haya fallado, puedes establecer la opción ``last`` en ``true``::

    $validator = new Validator();
    $validator
        ->add('body', [
            'minLength' => [
                'rule' => ['minLength', 10],
                'last' => true,
                'message' => 'Los comentarios deben tener un cuerpo sustancial.',
            ],
            'maxLength' => [
                'rule' => ['maxLength', 250],
                'message' => 'Los comentarios no pueden ser demasiado largos.',
            ],
        ]);

Si la regla minLength falla en el ejemplo anterior, la regla maxLength no se ejecutará.

Hacer que las Reglas sean 'last' por Defecto
============================================

Puedes hacer que la opción ``last`` se aplique automáticamente a cada regla que puedas usar
el método ``setStopOnFailure()`` para habilitar este comportamiento::

        public function validationDefault(Validator $validator): Validator
        {
            $validator
                ->setStopOnFailure()
                ->requirePresence('email', 'create')
                ->notBlank('email')
                ->email('email');

            return $validator;
        }

Cuando se habilita, todos los campos detendrán la validación en la primera regla que falle en lugar de
verificar todas las reglas posibles. En este caso, solo aparecerá un mensaje de error
bajo el campo del formulario.

Agregando Proveedores de Validación
-----------------------------------

Las clases ``Validator``, ``ValidationSet`` y ``ValidationRule`` no
proporcionan ningún método de validación por sí mismas. Las reglas de validación provienen de
'proveedores'. Puedes vincular cualquier número de proveedores a un objeto Validator.
Las instancias de Validator vienen con una configuración de proveedor 'predeterminada' automáticamente. El
proveedor predeterminado está mapeado a la clase :php:class:`~Cake\\Validation\\Validation`.
Esto hace que sea simple usar los métodos en esa clase como reglas de validación. Al usar Validadores y el ORM juntos, se configuran proveedores adicionales para los objetos de tabla y entidad. Puedes usar el método ``setProvider()``
para agregar cualquier proveedor adicional que tu aplicación necesite::

    $validator = new Validator();

    // Usa una instancia de objeto.
    $validator->setProvider('custom', $miObjeto);

    // Usa un nombre de clase. Los métodos deben ser estáticos.
    $validator->setProvider('custom', 'App\Model\Validation');

Los proveedores de validación pueden ser objetos o nombres de clases. Si se usa un nombre de clase, los
métodos deben ser estáticos. Para usar un proveedor que no sea 'predeterminado', asegúrate de establecer
la clave ``provider`` en tu regla::

    // Usa una regla del proveedor de la tabla
    $validator->add('title', 'custom', [
        'rule' => 'customTableMethod',
        'provider' => 'table'
    ]);

Si deseas agregar un ``proveedor`` a todos los objetos ``Validator`` que se creen
en el futuro, puedes usar el método ``addDefaultProvider()`` de la siguiente manera::

    use Cake\Validation\Validator;

    // Usa una instancia de objeto.
    Validator::addDefaultProvider('custom', $miObjeto);

    // Usa un nombre de clase. Los métodos deben ser estáticos.
    Validator::addDefaultProvider('custom', 'App\Model\Validation');

.. nota::

    Los ProveedoresPredeterminados deben agregarse antes de que se cree el objeto ``Validator``
    por lo tanto, **config/bootstrap.php** es el mejor lugar para configurar tus
    proveedores predeterminados.

Puedes usar el `plugin Localized <https://github.com/cakephp/localized>`_ para
obtener proveedores basados en países. Con este plugin, podrás validar
los campos del modelo, dependiendo de un país, por ejemplo::

    namespace App\Model\Table;

    use Cake\ORM\Table;
    use Cake\Validation\Validator;

    class PostsTable extends Table
    {
        public function validationDefault(Validator $validator): Validator
        {
            // agregar el proveedor al validador
            $validator->setProvider('fr', 'Cake\Localized\Validation\FrValidation');
            // usar el proveedor en una regla de validación de campo
            $validator->add('phoneField', 'myCustomRuleNameForPhone', [
                'rule' => 'phone',
                'provider' => 'fr'
            ]);

            return $validator;
        }
    }

El plugin localizado utiliza el código ISO de dos letras de los países para
la validación, como en, fr, de.

Hay algunos métodos que son comunes a todas las clases, definidas a través de la
`interfaz ValidationInterface <https://github.com/cakephp/localized/blob/master/src/Validation/ValidationInterface.php>`_::

    phone() para verificar un número de teléfono
    postal() para verificar un código postal
    personId() para verificar un ID de persona específico del país

Anidando Validadores
--------------------

Cuando validas :doc:`/core-libraries/form` con datos anidados, o cuando trabajas
con modelos que contienen tipos de datos de arreglo, es necesario validar los
datos anidados que tienes. CakePHP hace que sea simple agregar validadores a atributos específicos.
Por ejemplo, supón que estás trabajando con una base de datos no relacional
y necesitas almacenar un artículo y sus comentarios::

    $data = [
        'title' => 'Mejor artículo',
        'comments' => [
            ['comment' => ''],
        ],
    ];

Para validar los comentarios, usarías un validador anidado::

    $validator = new Validator();
    $validator->add('title', 'not-blank', ['rule' => 'notBlank']);

    $commentValidator = new Validator();
    $commentValidator->add('comment', 'not-blank', ['rule' => 'notBlank']);

    // Conecta los validadores anidados.
    $validator->addNestedMany('comments', $commentValidator);

    // Obtén todos los errores incluyendo aquellos de validadores anidados.
    $validator->validate($data);

Puedes crear relaciones 1:1 con ``addNested()`` y relaciones 1:N con ``addNestedMany()``. Con ambos métodos, los errores del validador anidado
contribuirán a los errores del validador principal e influirán en el resultado final.
Al igual que otras características del validador, los validadores anidados admiten mensajes de error y
aplicación condicional::

    $validator->addNestedMany(
        'comments',
        $commentValidator,
        'Comentario inválido',
        'create'
    );

El mensaje de error para un validador anidado se puede encontrar en la clave ``_nested``.

Creando Validadores Reutilizables
---------------------------------

Si bien definir validadores en línea donde se utilizan hace buen código de ejemplo, no conduce a aplicaciones mantenibles. En su lugar, deberías crear subclases de ``Validator`` para tu lógica de validación reutilizable::

    // En src/Model/Validation/ContactValidator.php
    namespace App\Model\Validation;

    use Cake\Validation\Validator;

    class ContactValidator extends Validator
    {
        public function __construct()
        {
            parent::__construct();
            // Agrega reglas de validación aquí.
        }
    }

Validando Datos
===============

Ahora que has creado un validador y has agregado las reglas que deseas, puedes comenzar a usarlo para validar datos. Los validadores pueden validar datos de arreglo. Por ejemplo, si deseas validar un formulario de contacto antes de crear y enviar un correo electrónico, podrías hacer lo siguiente::

    use Cake\Validation\Validator;

    $validator = new Validator();
    $validator
        ->requirePresence('email')
        ->add('email', 'validFormat', [
            'rule' => 'email',
            'message' => 'El correo electrónico debe ser válido',
        ])
        ->requirePresence('name')
        ->notEmptyString('name', 'Necesitamos tu nombre.')
        ->requirePresence('comment')
        ->notEmptyString('comment', 'Debes dar un comentario.');

    $errors = $validator->validate($this->request->getData());
    if (empty($errors)) {
        // Envía un correo electrónico.
    }

El método ``getErrors()`` devolverá un array no vacío cuando haya fallas de validación. El array devuelto de errores tendrá una estructura como esta::

    $errors = [
        'email' => ['El correo electrónico debe ser válido'],
    ];

Si tienes múltiples errores en un solo campo, se devolverá un array de mensajes de error por campo. Por defecto, el método ``getErrors()`` aplica reglas para el modo 'create'. Si deseas aplicar reglas de 'update', puedes hacer lo siguiente::

    $errors = $validator->validate($this->request->getData(), false);
    if (!$errors) {
        // Envía un correo electrónico.
    }

.. nota::

    Si necesitas validar entidades, debes usar métodos como
    :php:meth:`~Cake\\ORM\\Table::newEntity()`,
    :php:meth:`~Cake\\ORM\\Table::newEntities()`,
    :php:meth:`~Cake\\ORM\\Table::patchEntity()`,
    :php:meth:`~Cake\\ORM\\Table::patchEntities()`
    ya que están diseñados para eso.

Validando Datos de Entidad
==========================

La validación está destinada a verificar los datos de solicitud que provienen de formularios u otras interfaces de usuario utilizadas para poblar las entidades.

Los datos de solicitud se validan automáticamente cuando se utilizan los métodos ``newEntity()``, ``newEntities()``, ``patchEntity()`` o ``patchEntities()`` de la clase ``Table``::

    // En la clase ArticlesController
    $article = $this->Articles->newEntity($this->request->getData());
    if ($article->getErrors()) {
        // Haz el trabajo para mostrar mensajes de error.
    }

De manera similar, cuando necesitas validar múltiples entidades al mismo tiempo, puedes
usar el método ``newEntities()``::

    // En la clase ArticlesController
    $entities = $this->Articles->newEntities($this->request->getData());
    foreach ($entities as $entity) {
        if (!$entity->getErrors()) {
            $this->Articles->save($entity);
        }
    }

Los métodos ``newEntity()``, ``patchEntity()``, ``newEntities()`` y ``patchEntities()``
te permiten especificar qué asociaciones se validan y qué
conjuntos de validación aplicar usando el parámetro ``options``::

    $valid = $this->Articles->newEntity($article, [
        'associated' => [
            'Comments' => [
                'associated' => ['User'],
                'validate' => 'special',
            ],
        ],
    ]);

Aparte de validar datos proporcionados por el usuario, mantener la integridad de los datos independientemente
de dónde provengan es importante. Para resolver este problema, CakePHP ofrece un segundo
nivel de validación que se llama "reglas de aplicación". Puedes leer más sobre
ellas en la sección :ref:`Aplicando Reglas de Aplicación <application-rules>`.

Reglas de Validación Básicas del Núcleo
=======================================

CakePHP proporciona una suite básica de métodos de validación en la clase ``Validation``. La clase Validation contiene una variedad de métodos estáticos que proporcionan
validadores para varias situaciones de validación comunes.

La `documentación de la API
<https://api.cakephp.org/5.x/class-Cake.Validation.Validation.html>`_ para la
clase ``Validation`` proporciona una buena lista de las reglas de validación que están
disponibles, y su uso básico.

Algunos de los métodos de validación aceptan parámetros adicionales para definir condiciones de límite
o opciones válidas. Puedes proporcionar estas condiciones de límite y
opciones de la siguiente manera::

    $validator = new Validator();
    $validator
        ->add('title', 'minLength', [
            'rule' => ['minLength', 10],
        ])
        ->add('rating', 'validValue', [
            'rule' => ['range', 1, 5],
        ]);

Las reglas básicas que toman parámetros adicionales deben tener un array para la
clave ``rule`` que contenga la regla como el primer elemento y los parámetros adicionales como los parámetros restantes.
