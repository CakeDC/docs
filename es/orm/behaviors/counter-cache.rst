CounterCache Behavior
#####################

.. php:namespace:: Cake\ORM\Behavior

.. php:class:: CounterCacheBehavior

A menudo, las aplicaciones web necesitan mostrar recuentos de objetos relacionados. Por ejemplo,
al mostrar una lista de artículos, es posible que desees mostrar cuántos
comentarios tiene. O bien, al mostrar a un usuario, tal vez quieras mostrar cuántos
amigos/seguidores tiene. El comportamiento de CounterCache está pensado para estas
situaciones. CounterCache actualizará un campo en los modelos asociados asignados
en las opciones cuando sea invocado. Los campos deben existir en la base de datos y
ser del tipo INT.

Uso básico
==========

El CounterCache behavior se habilita como cualquier otro Behavior, pero no hará nada
hasta que configure algunas relaciones y los recuentos de campos que se deben almacenar en cada una de ellas.
Usando nuestro ejemplo a continuación, podríamos almacenar en caché el recuento de comentarios de cada artículo
con lo siguiente::

    class CommentsTable extends Table
    {
        public function initialize(array $config): void
        {
            $this->addBehavior('CounterCache', [
                'Articles' => ['comment_count']
            ]);
        }
    }


.. note::

    La columna ``comment_count`` bebe existir en la tabla ``articles``.

La configuración de CounterCache debe ser un mapa de nombres de relación y la
configuración específica para esa relación.

Como puedes ver, tienes que agregar el comportamiento en el "otro lado" de la asociación
donde realmente deseas que se actualice el campo. En este ejemplo, el comportamiento
se agrega a ``CommentsTable`` aunque actualiza el campo ``comment_count`` en ``ArticlesTable``.

El valor del contador se actualizará cada vez que se guarde o elimine una entidad.
El contador **no** se actualizará cuando

- guardes la entidad sin cambiar los datos o
- uses ``updateAll()`` o
- uses ``deleteAll()`` o
- ejecutes SQL que hayas escrito

Uso avanzado
============

Si necesitas mantener un contador almacenado en caché para al menos una parte de los registros relacionados,
puedes proporcionar condiciones adicionales o métodos de búsqueda para generar
un valor de contador::

    // Utilizar un método de búsqueda específico.
    // En este caso find(published)
    $this->addBehavior('CounterCache', [
        'Articles' => [
            'comment_count' => [
                'finder' => 'published',
            ],
        ],
    ]);

Si no tienes un método de búsqueda personalizado, también puedes usar un array de condiciones
para buscar registros::

    $this->addBehavior('CounterCache', [
        'Articles' => [
            'comment_count' => [
                'conditions' => ['Comments.spam' => false],
            ],
        ],
    ]);

Si quieres que CounterCache actualice múltiples campos, por ejemplo mostrando
tanto un recuento condicional como un recuento básico puedes añadir estos campos en el array::

    $this->addBehavior('CounterCache', [
        'Articles' => [
            'comment_count',
            'published_comment_count' => [
                'finder' => 'published',
            ],
        ],
    ]);

Si desea calcular el valor del campo CounterCache por tu cuenta, puede establecer
la opción ``ignoreDirty`` a ``true``.
Esto evitará que el campo sea recalculado si lo has puesto dirty antes::

    $this->addBehavior('CounterCache', [
        'Articles' => [
            'comment_count' => [
                'ignoreDirty' => true,
            ],
        ],
    ]);

Por último, si un buscador personalizado y las condiciones no son adecuadas, puedes utilizar
una función callback. La función debe devolver el valor de recuento que se va a almacenar::

    $this->addBehavior('CounterCache', [
        'Articles' => [
            'rating_avg' => function ($event, $entity, $table, $original) {
                return 4.5;
            }
        ],
    ]);

Tu función puede devolver ``false`` para omitir la actualización de la columna del contador, o
un objeto ``SelectQuery`` que generó el valor de recuento. Si devuelves un objeto
``SelectQuery``, la consulta se usará como una subconsulta en la instrucción update. El parámetro
``$table`` se refiere al objeto de la tabla que contiene el behavior (en este caso CommentsTable y no la
relación de destino ArticlesTable) por conveniencia. El callback se invoca al menos una vez con
``$original`` establecido en ``false``. Si entity-update cambia la asociación,
la devolución de llamada se invoca una *segunda* vez con ``true``, el valor devuelto
actualiza el contador del elemento asociado *anteriormente*.

.. note::

    El behavior de CounterCache solo funciona para las asociaciones ``belongsTo``.
    Por ejemplo, para "Los comentarios pertenecen a Artículos", tienes que agregar el behavior
    CounterCache a la tabla ``CommentsTable`` para generar ``comment_count`` para la tabla ``ArticlesTable`.

Usos para BelongsToMany
======================

Es posible usar CounterCache behavior en una asociación ``belongsToMany``.
En primer lugar, tienes que añadir la opciones ``through`` and ``cascadeCallbacks`` a la asociación
``belongsToMany``::

    'through'          => 'CommentsArticles',
    'cascadeCallbacks' => true

Ver también :ref:`using-the-through-option` como configurar una tabla join personalizada.

``CommentsArticles`` es el nombre de la tabla de union classname.
Si no lo tienes, debes crearlo con la herramienta CLI de bake.

En este ``src/Model/Table/CommentsArticlesTable.php`` tienes que añadir el behavior
con el mismo código descrito anteriormente.::

    $this->addBehavior('CounterCache', [
        'Articles' => ['comments_count'],
    ]);

Para terminar borra todas las cachés con ``bin/cake cache clear_all`` y pruébalo.

.. meta::
    :title lang=es: CounterCache Behavior
    :keywords lang=es: maintenance branch,community interaction,community feature,necessary feature,stable release,ticket system,advanced feature,power users,feature set,chat irc,leading edge,router,new features,members,attempt,development branches,branch development, counter cache
