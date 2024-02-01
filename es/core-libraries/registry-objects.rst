Objetos de Registro
###################

Las clases de registro proporcionan una forma sencilla de crear y recuperar instancias cargadas de un tipo de objeto dado. Hay clases de registro para Componentes, Ayudantes, Tareas y Comportamientos.

Si bien los ejemplos a continuación utilizarán Componentes, se puede esperar el mismo comportamiento para Ayudantes, Comportamientos y Tareas además de Componentes.

Carga de Objetos
================

Los objetos pueden cargarse sobre la marcha utilizando add<objeto-de-registro>().
Ejemplo::

    $this->loadComponent('Acl.Acl');
    $this->addHelper('Flash')

Esto resultará en la carga de la propiedad ``Acl`` y el ayudante ``Flash``.
La configuración también se puede establecer sobre la marcha. Ejemplo::

    $this->loadComponent('Cookie', ['name' => 'sweet']);

Cualquier clave y valor proporcionados se pasarán al constructor del Componente. La única excepción a esta regla es ``className``. Classname es una clave especial que se utiliza para aliasar objetos en un registro. Esto le permite tener nombres de componentes que no reflejen los nombres de las clases, lo que puede ser útil al extender componentes centrales::

    $this->Flash = $this->loadComponent('Flash', ['className' => 'MiFlashPersonalizado']);
    $this->Flash->error(); // Realmente utilizando MiFlashPersonalizado::error();

Desencadenar Callbacks
======================

Los callbacks no son proporcionados por los objetos de registro. Deberías usar el :doc:`sistema de eventos </core-libraries/events>` para despachar cualquier evento/callback para tu aplicación.

Desactivar Callbacks
====================

En versiones anteriores, los objetos de colección proporcionaban un método ``disable()`` para desactivar objetos para recibir callbacks. Deberías usar las características en el sistema de eventos para lograr esto ahora. Por ejemplo, podrías desactivar los callbacks de componente de la siguiente manera::

    // Eliminar MyComponent de los callbacks.
    $this->getEventManager()->off($this->MyComponent);

    // Re-habilitar MyComponent para los callbacks.
    $this->getEventManager()->on($this->MyComponent);

.. meta::
    :title lang=es: Objetos de Registro
    :keywords lang=es: nombre de arreglo, cargando componentes, varios tipos diferentes, API unificada, cargando objetos, nombres de componentes, clave especial, componentes principales, callbacks, prg, callback, alias, error fatal, colecciones, memoria, prioridad, prioridades
