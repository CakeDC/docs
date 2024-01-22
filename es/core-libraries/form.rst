Formularios sin Modelo
######################

.. php:namespace:: Cake\Form

.. php:class:: Form (Formulario)

La mayoría de las veces, tendrás formularios respaldados por :doc:`entidades ORM </orm/entities>`
y :doc:`tablas ORM </orm/table-objects>` u otras fuentes persistentes,
pero hay momentos en los que necesitarás validar la entrada del usuario y luego realizar
una acción si los datos son válidos. El ejemplo más común de esto es un formulario de contacto.

Creación de un Formulario
=========================

Generalmente, al utilizar la clase Form, querrás usar una subclase para definir tu
formulario. Esto facilita las pruebas y te permite reutilizar tu formulario. Los formularios se colocan
en **src/Form** y generalmente tienen ``Form`` como sufijo de la clase. Por ejemplo,
un formulario de contacto simple se vería así::

    // en src/Form/ContactForm.php
    namespace App\Form;

    use Cake\Form\Form;
    use Cake\Form\Schema;
    use Cake\Validation\Validator;

    class ContactForm extends Form
    {
        protected function _buildSchema(Schema $schema): Schema
        {
            return $schema->addField('name', 'string')
                ->addField('email', ['type' => 'string'])
                ->addField('body', ['type' => 'text']);
        }

        public function validationDefault(Validator $validator): Validator
        {
            $validator->minLength('name', 10)
                ->email('email');

            return $validator;
        }

        protected function _execute(array $data): bool
        {
            // Enviar un correo electrónico.
            return true;
        }
    }

En el ejemplo anterior, vemos los 3 métodos de gancho que proporcionan los formularios:

* ``_buildSchema`` se utiliza para definir los datos del esquema que utiliza FormHelper
  para crear un formulario HTML. Puedes definir el tipo de campo, longitud y precisión.
* ``validationDefault`` obtiene una instancia de :php:class:`Cake\\Validation\\Validator`
  a la que puedes adjuntar validadores.
* ``_execute`` te permite definir el comportamiento que deseas que ocurra cuando
  se llama a ``execute()`` y los datos son válidos.

Siempre puedes definir métodos públicos adicionales según sea necesario.

Procesamiento de Datos de Solicitud
===================================

Una vez que hayas definido tu formulario, puedes usarlo en tu controlador para procesar
y validar los datos de la solicitud::

    // En un controlador
    namespace App\Controller;

    use App\Controller\AppController;
    use App\Form\ContactForm;

    class ContactController extends AppController
    {
        public function index()
        {
            $contact = new ContactForm();
            if ($this->request->is('post')) {
                if ($contact->execute($this->request->getData())) {
                    $this->Flash->success('Nos pondremos en contacto contigo pronto.');
                } else {
                    $this->Flash->error('Hubo un problema al enviar tu formulario.');
                }
            }
            $this->set('contact', $contact);
        }
    }

En el ejemplo anterior, usamos el método ``execute()`` para ejecutar el método
``_execute()`` de nuestro formulario solo cuando los datos son válidos, y establecemos mensajes flash
según corresponda. Si queremos usar un conjunto de validación no predeterminado, podemos usar la opción ``validate``::

    if ($contact->execute($this->request->getData(), 'update')) {
        // Manejar el éxito del formulario.
    }

Esta opción también se puede establecer en ``false`` para desactivar la validación.

También podríamos haber usado el método ``validate()`` para validar solo
los datos de la solicitud::

    $isValid = $form->validate($this->request->getData());

    // También puedes usar otros conjuntos de validación. Lo siguiente
    // utilizaría las reglas definidas por `validationUpdate()`
    $isValid = $form->validate($this->request->getData(), 'update');

Establecer Valores del Formulario
=================================

Puedes establecer valores predeterminados para formularios sin modelo utilizando el método ``setData()``.
Los valores establecidos con este método sobrescribirán los datos existentes en el objeto del formulario::

    // En un controlador
    namespace App\Controller;

    use App\Controller\AppController;
    use App\Form\ContactForm;

    class ContactController extends AppController
    {
        public function index()
        {
            $contact = new ContactForm();
            if ($this->request->is('post')) {
                if ($contact->execute($this->request->getData())) {
                    $this->Flash->success('Nos pondremos en contacto contigo pronto.');
                } else {
                    $this->Flash->error('Hubo un problema al enviar tu formulario.');
                }
            }

            if ($this->request->is('get')) {
                $contact->setData([
                    'name' => 'John Doe',
                    'email' => 'john.doe@example.com'
                ]);
            }

            $this->set('contact', $contact);
        }
    }

Los valores solo deben definirse si el método de solicitud es GET; de lo contrario,
sobrescribirás los datos anteriores del POST que podrían tener errores de validación
que necesitan correcciones. Puedes usar ``set()`` para agregar o reemplazar campos individuales
o un subconjunto de campos::

    // Establecer un campo.
    $contact->set('name', 'John Doe');

    // Establecer varios campos;
    $contact->set([
        'name' => 'John Doe',
        'email' => 'john.doe@example.com',
    ]);

Obtener Errores del Formulario
==============================

Una vez que un formulario ha sido validado, puedes recuperar los errores de él::

    $errors = $form->getErrors();
    /* $errors contiene
    [
        'name' => ['length' => 'El nombre debe tener al menos dos caracteres de longitud'],
        'email' => ['format' => 'Se requiere una dirección de correo electrónico válida'],
    ]
    */

    $error = $form->getError('email');
    /* $error contiene
    [
        'format' => 'Se requiere una dirección de correo electrónico válida',
    ]
    */

Invalidar Campos Individuales del Formulario desde el Controlador
=================================================================

Es posible invalidar campos individuales desde el controlador sin la
necesidad de usar la clase Validator. El caso de uso más común para esto es cuando la
validación se realiza en un servidor remoto. En tal caso, debes invalidar manualmente
los campos de acuerdo con la retroalimentación del servidor remoto::

    // en src/Form/ContactForm.php
    public function setErrors($errors)
    {
        $this->_errors = $errors;
    }

Según la forma en que la clase validadora habría devuelto los errores, ``$errors``
debe tener este formato::

    ['nombreCampo' => ['nombreValidador' => 'El mensaje de error a mostrar']]

Ahora podrás invalidar campos del formulario configurando el nombre del campo y
luego establecer los mensajes de error::

    // En un controlador
    $contact = new ContactForm();
    $contact->setErrors(['email' => ['_required' => 'Tu correo electrónico es obligatorio']]);

Continúa con la creación de HTML con FormHelper para ver los resultados.

Creación de HTML con FormHelper
===============================

Una vez que hayas creado una clase Form, es probable que desees crear un formulario HTML para
ella. FormHelper comprende objetos Form al igual que las entidades ORM::

    echo $this->Form->create($contact);
    echo $this->Form->control('name');
    echo $this->Form->control('email');
    echo $this->Form->control('body');
    echo $this->Form->button('Enviar');
    echo $this->Form->end();

Lo anterior crearía un formulario HTML para el ``ContactForm`` que definimos anteriormente.
Los formularios HTML creados con FormHelper utilizarán el esquema y el validador definidos para
determinar los tipos de campos, longitudes máximas y errores de validación.
