Timestamp
#########

.. php:namespace:: Cake\ORM\Behavior

.. php:class:: TimestampBehavior

El comportamiento del behavior timestamp permite a tus objetos de tabla actualizar uno o más
marcas de tiempo en cada evento del modelo. Esto se utiliza principalmente para rellenar datos
en los campos ``created`` y ``modified``. Sin embargo, con alguna configuración adicional,
puedes actualizar cualquier columna de fecha/hora en cualquier evento que publique una tabla.

Uso básico
==========

El behavior de timestamp se activa como cualquier otro behavior::

    class ArticlesTable extends Table
    {
        public function initialize(array $config): void
        {
            $this->addBehavior('Timestamp');
        }
    }

La configuración por defecto hará lo siguiente:

- Cuando se guarda una nueva entidad, los campos ``created`` y ``modified`` se ajustan a la hora actual.
- Cuando se actualiza una entidad, el campo ``modified`` se establece con la hora actual.

Uso y configuración del behavior
================================

Si necesita modificar campos con nombres diferentes, o desea actualizar campos de
timestamp adicionales en eventos personalizados puede utilizar alguna configuración adicional::

    class OrdersTable extends Table
    {
        public function initialize(array $config): void
        {
            $this->addBehavior('Timestamp', [
                'events' => [
                    'Model.beforeSave' => [
                        'created_at' => 'new',
                        'updated_at' => 'always',
                    ],
                    'Orders.completed' => [
                        'completed_at' => 'always'
                    ]
                ]
            ]);
        }
    }

Como puede ver arriba, además del evento estándar ``Model.beforeSave``,
también estamos actualizando la columna ``completed_at`` cuando se completan los pedidos.

Actualización de timestamp en entidades
=======================================

A veces querrá actualizar sólo las marcas de tiempo de una entidad sin cambiar
ninguna otra propiedad. Esto es algunas veces referido como 'tocar' un registro.
En CakePHP puedes usar el método ``touch()`` para hacer exactamente esto::

    // Touch based on the Model.beforeSave event.
    $articles->touch($article);

    // Touch based on a specific event.
    $orders->touch($order, 'Orders.completed');

Una vez guardada la entidad, el campo se actualiza.

Tocar registros puede ser útil cuando se desea señalar que un recurso padre
ha cambiado cuando se crea/actualiza un recurso hijo.
Por ejemplo: actualizar un artículo cuando se añade un nuevo comentario.

Guardar actualizaciones sin modificar timestamps
================================================

Para desactivar la modificación automática de la columna ``modified`` al
guardar una entidad puedes marcar el atributo como 'dirty'::

    // Marcar la columna modificada como dirty haciendo
    // que el valor actual se establezca en la actualización.
    $order->setDirty('modified', true);

.. meta::
    :title lang=es: Timestamp
    :keywords lang=es: maintenance branch,community interaction,community feature,necessary feature,stable release,ticket system,advanced feature,power users,feature set,chat irc,leading edge,router,new features,members,attempt,development branches,branch development
