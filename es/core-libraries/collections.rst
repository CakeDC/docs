Colecciones
###########

.. php:namespace:: Cake\Collection

.. php:class:: Collection

Las clases de colección proporcionan un conjunto de herramientas para manipular matrices u objetos ``Traversable``. Si alguna vez has utilizado underscore.js, tienes una idea de lo que puedes esperar de las clases de colección.

Las instancias de colección son inmutables; modificar una colección generará en su lugar una nueva colección. Esto hace que trabajar con objetos de colección sea más predecible, ya que las operaciones no tienen efectos secundarios.

Ejemplo Rápido
==============

Las colecciones se pueden crear usando una matriz u objeto ``Traversable``. También interactuarás con colecciones cada vez que interactúes con el ORM en CakePHP. Un uso simple de una colección sería::

    use Cake\Collection\Collection;

    $items = ['a' => 1, 'b' => 2, 'c' => 3];
    $coleccion = new Collection($items);

    // Crea una nueva colección que contiene elementos
    // con un valor mayor que uno.
    $mayorUno = $coleccion->filter(function ($valor, $clave, $iterador) {
        return $valor > 1;
    });

También puedes usar la función ``collection()`` en lugar de ``new
Collection()``::

    $items = ['a' => 1, 'b' => 2, 'c' => 3];

    // Ambos crean una instancia de Collection.
    $coleccionA = new Collection($items);
    $coleccionB = collection($items);

La ventaja del método de ayuda es que es más fácil encadenar que
``(new Collection($items))``.

El :php:trait:`~Cake\\Collection\\CollectionTrait` te permite integrar
características similares a las de una colección en cualquier objeto ``Traversable`` que tengas en tu
aplicación.

Lista de Métodos
================

.. csv-table::
    :class: docutils internal-toc

    :php:meth:`append`, :php:meth:`appendItem`, :php:meth:`avg`,
    :php:meth:`buffered`, :php:meth:`chunk`, :php:meth:`chunkWithKeys`
    :php:meth:`combine`, :php:meth:`compile`, :php:meth:`contains`
    :php:meth:`countBy`, :php:meth:`each`, :php:meth:`every`
    :php:meth:`extract`, :php:meth:`filter`, :php:meth:`first`
    :php:meth:`firstMatch`, :php:meth:`groupBy`, :php:meth:`indexBy`
    :php:meth:`insert`, :php:meth:`isEmpty`, :php:meth:`last`
    :php:meth:`listNested`, :php:meth:`map`, :php:meth:`match`
    :php:meth:`max`, :php:meth:`median`, :php:meth:`min`
    :php:meth:`nest`, :php:meth:`prepend`, :php:meth:`prependItem`
    :php:meth:`reduce`, :php:meth:`reject`, :php:meth:`sample`
    :php:meth:`shuffle`, :php:meth:`skip`, :php:meth:`some`
    :php:meth:`sortBy`, :php:meth:`stopWhen`, :php:meth:`sumOf`
    :php:meth:`take`, :php:meth:`through`, :php:meth:`transpose`
    :php:meth:`unfold`, :php:meth:`zip`

Iteración
=========

.. php:method:: each($callback)

Las colecciones pueden ser iteradas y/o transformadas en nuevas colecciones con los métodos ``each()`` y ``map()``. El método ``each()`` no creará una nueva colección, pero te permitirá modificar cualquier objeto dentro de la colección::

    $coleccion = new Collection($elementos);
    $coleccion = $coleccion->each(function ($valor, $clave) {
        echo "Elemento $clave: $valor";
    });

El retorno de ``each()`` será el objeto de la colección. Each iterará la colección aplicando inmediatamente el callback a cada valor en la colección.

.. php:method:: map($callback)

El método ``map()`` creará una nueva colección basada en la salida del callback aplicado a cada objeto en la colección original::

    $elementos = ['a' => 1, 'b' => 2, 'c' => 3];
    $coleccion = new Collection($elementos);

    $nueva = $coleccion->map(function ($valor, $clave) {
        return $valor * 2;
    });

    // $resultado contiene [2, 4, 6];
    $resultado = $nueva->toList();

    // $resultado contiene ['a' => 2, 'b' => 4, 'c' => 6];
    $resultado = $nueva->toArray();

El método ``map()`` creará un nuevo iterador que crea perezosamente los elementos resultantes cuando se itera.

.. php:method:: extract($path)

Uno de los usos más comunes de una función ``map()`` es extraer una sola columna de una colección. Si estás buscando construir una lista de elementos que contengan los valores de una propiedad específica, puedes usar el método ``extract()``::

    $coleccion = new Collection($personas);
    $nombres = $coleccion->extract('nombre');

    // $resultado contiene ['mark', 'jose', 'barbara'];
    $resultado = $nombres->toList();

Como con muchas otras funciones en la clase de colección, se te permite especificar un camino separado por puntos para extraer columnas. Este ejemplo devolverá una colección que contiene los nombres de los autores de una lista de artículos::

    $coleccion = new Collection($articulos);
    $nombres = $coleccion->extract('autor.nombre');

    // $resultado contiene ['Maria', 'Stacy', 'Larry'];
    $resultado = $nombres->toList();

Finalmente, si la propiedad que estás buscando no se puede expresar como un camino, puedes usar una función de devolución de llamada para obtenerla::

    $coleccion = new Collection($articulos);
    $nombres = $coleccion->extract(function ($articulo) {
        return $articulo->autor->nombre . ', ' . $articulo->autor->apellido;
    });

A menudo, las propiedades que necesitas extraer son una clave común presente en múltiples matrices u objetos que están profundamente anidados dentro de otras estructuras. Para esos casos, puedes usar el comodín ``{*}`` en la clave del camino. Este comodín es útil cuando se coinciden datos de asociaciones HasMany y BelongsToMany::

    $datos = [
        [
            'nombre' => 'James',
            'numeros_telefonicos' => [
                ['numero' => 'numero-1'],
                ['numero' => 'numero-2'],
                ['numero' => 'numero-3'],
            ],
        ],
        [
            'nombre' => 'James',
            'numeros_telefonicos' => [
                ['numero' => 'numero-4'],
                ['numero' => 'numero-5'],
            ],
        ],
    ];

    $numeros = (new Collection($datos))->extract('numeros_telefonicos.{*}.numero');
    $resultado = $numeros->toList();
    // $resultado contiene ['numero-1', 'numero-2', 'numero-3', 'numero-4', 'numero-5']

Este último ejemplo usa ``toList()`` a diferencia de otros ejemplos, lo cual es importante cuando obtenemos resultados con claves posiblemente duplicadas. Al usar ``toList()``, nos aseguraremos de obtener todos los valores incluso si hay claves duplicadas.

A diferencia de :php:meth:`Cake\\Utility\\Hash::extract()`, este método solo admite el comodín ``{*}``. Ningún otro comodín o coincidente de atributos es compatible.

.. php:method:: combine($keyPath, $valuePath, $groupPath = null)

Las colecciones te permiten crear una nueva colección a partir de claves y valores en una colección existente. Tanto el camino de la clave como el de los valores pueden especificarse con notación de puntos::

    $elementos = [
        ['id' => 1, 'name' => 'foo', 'parent' => 'a'],
        ['id' => 2, 'name' => 'bar', 'parent' => 'b'],
        ['id' => 3, 'name' => 'baz', 'parent' => 'a'],
    ];
    $combinada = (new Collection($elementos))->combine('id', 'name');
    $resultado = $combinada->toArray();

    // $resultado contiene
    [
        1 => 'foo',
        2 => 'bar',
        3 => 'baz',
    ];

También puedes opcionalmente usar un ``groupPath`` para agrupar resultados basados en un camino::

    $combinada = (new Collection($elementos))->combine('id', 'name', 'parent');
    $resultado = $combinada->toArray();

    // $resultado contiene
    [
        'a' => [1 => 'foo', 3 => 'baz'],
        'b' => [2 => 'bar']
    ];

Finalmente, puedes usar *cierres* para construir caminos de claves/valores/grupos dinámicamente, por ejemplo, cuando trabajas con entidades y fechas (convertidas a instancias de ``I18n\DateTime`` por el ORM) es posible que desees agrupar los resultados por fecha::

    $combinada = (new Collection($entidades))->combine(
        'id',
        function ($entidad) { return $entidad; },
        function ($entidad) { return $entidad->date->toDateString(); }
    );
     $resultado = $combinada->toArray();

    // $resultado contiene
    [
        'cadena de fecha como 2015-05-01' => ['entidad1->id' => entidad1, 'entidad2->id' => entidad2, ..., 'entidadN->id' => entidadN]
        'cadena de fecha como 2015-06-01' => ['entidad1->id' => entidad1, 'entidad2->id' => entidad2, ..., 'entidadN->id' => entidadN]
    ]

.. php:method:: stopWhen(callable $c)

Puedes detener la iteración en cualquier punto usando el método ``stopWhen()``. Llamarlo en una colección creará una nueva que dejará de generar resultados si el callable pasado devuelve true para uno de los elementos::

    $elementos = [10, 20, 50, 1, 2];
    $coleccion = new Collection($elementos);

    $nueva = $coleccion->stopWhen(function ($valor, $clave) {
        // Detener en el primer valor mayor que 30
        return $valor > 30;
    });

    // $resultado contiene [10, 20];
    $resultado = $nueva->toList();

.. php:method:: unfold(callable $callback)

A veces, los elementos internos de una colección contendrán matrices o iteradores con más elementos. Si deseas aplanar la estructura interna para iterar una vez sobre todos los elementos, puedes usar el método ``unfold()``. Creará una nueva colección que generará cada elemento único anidado en la colección::

    $elementos = [[1, 2, 3], [4, 5]];
    $coleccion = new Collection($elementos);
    $nueva = $coleccion->unfold();

    // $resultado contiene [1, 2, 3, 4, 5];
    $resultado = $nueva->toList();

Cuando pasas un callable a ``unfold()``, puedes controlar qué elementos se desplegarán de cada elemento en la colección original. Esto es útil para devolver datos de servicios paginados::

    $paginas = [1, 2, 3, 4];
    $coleccion = new Collection($paginas);
    $elementos = $coleccion->unfold(function ($pagina, $clave) {
        // Un servicio web imaginario que devuelve una página de resultados
        return MyService::fetchPage($pagina)->toList();
    });

    $todosLosElementosDeLasPaginas = $elementos->toList();

Si estás usando PHP 5.5+, puedes usar la palabra clave ``yield`` dentro de ``unfold()`` para devolver tantos elementos para cada elemento en la colección como puedas necesitar::

    $numerosImpares = [1, 3, 5, 7];
    $coleccion = new Collection($numerosImpares);
    $nueva = $coleccion->unfold(function ($numeroImpar) {
        yield $numeroImpar;
        yield $numeroImpar + 1;
    });

    // $resultado contiene [1, 2, 3, 4, 5, 6, 7, 8];
    $resultado = $nueva->toList();

.. php:method:: chunk($chunkSize)

Cuando se trata de grandes cantidades de elementos en una colección, puede tener sentido procesar los elementos en lotes en lugar de uno por uno. Para dividir una colección en varios conjuntos de un tamaño determinado, puedes usar la función ``chunk()``::

    $elementos = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
    $coleccion = new Collection($elementos);
    $troceada = $coleccion->chunk(2);
    $troceada->toList(); // [[1, 2], [3, 4], [5, 6], [7, 8], [9, 10], [11]]

La función ``chunk`` es particularmente útil al realizar procesamiento por lotes, por ejemplo, con un resultado de base de datos::

    $coleccion = new Collection($artículos);
    $coleccion->map(function ($artículo) {
            // Cambiar una propiedad en el artículo
            $artículo->propiedad = 'cambiada';
        })
        ->chunk(20)
        ->each(function ($lote) {
            myBulkSave($lote); // Esta función se llamará para cada lote
        });

.. php:method:: chunkWithKeys($chunkSize)

Al igual que :php:meth:`chunk()`, ``chunkWithKeys()`` te permite dividir
una colección en lotes más pequeños pero conservando las claves. Esto es útil cuando se dividen arrays asociativos::

    $coleccion = new Collection([
        'a' => 1,
        'b' => 2,
        'c' => 3,
        'd' => [4, 5]
    ]);
    $troceada = $coleccion->chunkWithKeys(2);
    $resultado = $troceada->toList();

    // $resultado contiene
    [
        ['a' => 1, 'b' => 2],
        ['c' => 3, 'd' => [4, 5]]
    ]

Filtrado
========

.. php:method:: filter($callback)

Las colecciones te permiten filtrar y crear nuevas colecciones basadas en el resultado de funciones de devolución de llamada. Puedes usar ``filter()`` para crear una nueva colección de elementos que coincidan con un criterio de devolución de llamada::

    $coleccion = new Collection($personas);
    $damas = $coleccion->filter(function ($persona, $clave) {
        return $persona->gender === 'female';
    });
    $caballeros = $coleccion->filter(function ($persona, $clave) {
        return $persona->gender === 'male';
    });

.. php:method:: reject(callable $c)

La inversa de ``filter()`` es ``reject()``. Este método hace un filtro negativo, eliminando elementos que coincidan con la función de filtro::

    $coleccion = new Collection($personas);
    $damas = $coleccion->reject(function ($persona, $clave) {
        return $persona->gender === 'male';
    });

.. php:method:: every($callback)

Puedes hacer pruebas de verdad con funciones de filtro. Para ver si cada elemento en una colección coincide con una prueba, puedes usar ``every()``::

    $coleccion = new Collection($personas);
    $todosJóvenes = $coleccion->every(function ($persona) {
        return $persona->age < 21;
    });

.. php:method:: some($callback)

Puedes ver si la colección contiene al menos un elemento que coincida con una función de filtro usando el método ``some()``::

    $coleccion = new Collection($personas);
    $hayJóvenes = $coleccion->some(function ($persona) {
        return $persona->age < 21;
    });

.. php:method:: match($conditions)

Si necesitas extraer una nueva colección que contenga solo los elementos que contienen un conjunto dado de propiedades, debes usar el método ``match()``::

    $coleccion = new Collection($comentarios);
    $comentariosDeMark = $coleccion->match(['user.name' => 'Mark']);

.. php:method:: firstMatch($conditions)

El nombre de la propiedad puede ser un camino separado por puntos. Puedes navegar hacia entidades anidadas y coincidir con los valores que contienen. Cuando solo necesitas el primer elemento que coincide de una colección, puedes usar ``firstMatch()``::

    $coleccion = new Collection($comentarios);
    $comentario = $coleccion->firstMatch([
        'user.name' => 'Mark',
        'active' => true
    ]);

Como se puede ver en lo anterior, tanto ``match()`` como ``firstMatch()`` te permiten proporcionar múltiples condiciones para coincidir. Además, las condiciones pueden ser para diferentes caminos, lo que te permite expresar condiciones complejas a las que coincidir.

Agregación
==========

.. php:method:: reduce($callback, $initial)

El contraparte de una operación ``map()`` suele ser un ``reduce``. Esta función te ayudará a construir un resultado único a partir de todos los elementos en una colección::

    $precioTotal = $coleccion->reduce(function ($acumulado, $lineaDePedido) {
        return $acumulado + $lineaDePedido->precio;
    }, 0);

En el ejemplo anterior, ``$precioTotal`` será la suma de todos los precios individuales contenidos en la colección. Observa que el segundo argumento para la función ``reduce()`` toma el valor inicial para la operación de reducción que estás realizando::

    $todasLasEtiquetas = $coleccion->reduce(function ($acumulado, $articulo) {
        return array_merge($acumulado, $articulo->etiquetas);
    }, []);

.. php:method:: min(string|callable $callback, $type = SORT_NUMERIC)

Para extraer el valor mínimo de una colección basado en una propiedad, simplemente utiliza la función ``min()``. Esto devolverá el elemento completo de la colección y no solo el valor más pequeño encontrado::

    $coleccion = new Collection($personas);
    $másJoven = $coleccion->min('edad');

    echo $másJoven->nombre;

También puedes expresar la propiedad a comparar proporcionando un camino o una función de devolución de llamada::

    $coleccion = new Collection($personas);
    $personaConHijoMásJoven = $coleccion->min(function ($persona) {
        return $persona->hijo->edad;
    });

    $personaConPadreMásJoven = $coleccion->min('padre.edad');

.. php:method:: max(string|callable $callback, $type = SORT_NUMERIC)

Lo mismo se puede aplicar a la función ``max()``, que devolverá un solo elemento de la colección con el valor de propiedad más alto::

    $coleccion = new Collection($personas);
    $mayor = $coleccion->max('edad');

    $personaConHijoMayor = $coleccion->max(function ($persona) {
        return $persona->hijo->edad;
    });

    $personaConPadreMayor = $coleccion->max('padre.edad');

.. php:method:: sumOf($path = null)

Finalmente, el método ``sumOf()`` devolverá la suma de una propiedad de todos los elementos::

    $coleccion = new Collection($personas);
    $sumaDeEdades =  $coleccion->sumOf('edad');

    $sumaDeEdadesDeHijos = $coleccion->sumOf(function ($persona) {
        return $persona->hijo->edad;
    });

    $sumaDeEdadesDePadres = $coleccion->sumOf('padre.edad');

.. php:method:: avg($path = null)

Calcula el valor promedio de los elementos en la colección. Opcionalmente, proporciona un camino de coincidencia o una función para extraer valores y generar el promedio::

    $elementos = [
       ['factura' => ['total' => 100]],
       ['factura' => ['total' => 200]],
    ];

    // $promedio contiene 150
    $promedio = (new Collection($elementos))->avg('factura.total');

.. php:method:: median($path = null)

Calcula el valor mediano de un conjunto de elementos. Opcionalmente, proporciona un camino de coincidencia o una función para extraer valores y generar la mediana::

    $elementos = [
      ['factura' => ['total' => 400]],
      ['factura' => ['total' => 500]],
      ['factura' => ['total' => 100]],
      ['factura' => ['total' => 333]],
      ['factura' => ['total' => 200]],
    ];

    // $mediana contiene 333
    $mediana = (new Collection($elementos))->median('factura.total');

Agrupación y Conteo
-------------------

.. php:method:: groupBy($callback)

Los valores de una colección se pueden agrupar por diferentes claves en una nueva colección cuando comparten el mismo valor para una propiedad::

    $estudiantes = [
        ['nombre' => 'Mark', 'grado' => 9],
        ['nombre' => 'Andrew', 'grado' => 10],
        ['nombre' => 'Stacy', 'grado' => 10],
        ['nombre' => 'Barbara', 'grado' => 9]
    ];
    $coleccion = new Collection($estudiantes);
    $estudiantesPorGrado = $coleccion->groupBy('grado');
    $resultado = $estudiantesPorGrado->toArray();

    // $resultado contiene
    [
      10 => [
        ['nombre' => 'Andrew', 'grado' => 10],
        ['nombre' => 'Stacy', 'grado' => 10]
      ],
      9 => [
        ['nombre' => 'Mark', 'grado' => 9],
        ['nombre' => 'Barbara', 'grado' => 9]
      ]
    ]

Como de costumbre, es posible proporcionar ya sea un camino separado por puntos para propiedades anidadas o tu propia función de devolución de llamada para generar los grupos dinámicamente::

    $comentariosPorIdDeUsuario = $comentarios->groupBy('usuario.id');

    $resultadosDeClase = $estudiantes->groupBy(function ($estudiante) {
        return $estudiante->grado > 6 ? 'aprobado' : 'denegado';
    });

.. php:method:: countBy($callback)

Si solo deseas conocer el número de ocurrencias por grupo, puedes hacerlo mediante el método ``countBy()``. Toma los mismos argumentos que ``groupBy``, así que ya debería ser familiar para ti::

    $resultadosDeClase = $estudiantes->countBy(function ($estudiante) {
        return $estudiante->grado > 6 ? 'aprobado' : 'denegado';
    });

    // El resultado podría parecerse a esto cuando se convierte a un array:
    ['aprobado' => 70, 'denegado' => 20]

.. php:method:: indexBy($callback)

Habrá casos en los que sepas que un elemento es único para la propiedad por la que deseas agrupar. Si deseas un solo resultado por grupo, puedes usar la función ``indexBy()``::

    $usuariosPorId = $usuarios->indexBy('id');

    // Cuando se convierte a un array, el resultado podría parecerse a
    [
        1 => 'markstory',
        3 => 'jose_zap',
        4 => 'jrbasso'
    ]

Al igual que la función ``groupBy()``, también puedes usar un camino de propiedad o una devolución de llamada::

    $articulosPorIdDeAutor = $articulos->indexBy('autor.id');

    $archivosPorHash = $archivos->indexBy(function ($archivo) {
        return md5($archivo);
    });

.. php:method:: zip($items)

Los elementos de diferentes colecciones se pueden agrupar juntos usando el método ``zip()``. Devolverá una nueva colección que contiene una matriz que agrupa los elementos de cada colección que están ubicados en la misma posición::

    $impares = new Collection([1, 3, 5]);
    $pares = new Collection([2, 4, 6]);
    $combinados = $impares->zip($pares)->toList(); // [[1, 2], [3, 4], [5, 6]]

También puedes agrupar múltiples colecciones a la vez::

    $años = new Collection([2013, 2014, 2015, 2016]);
    $salarios = [1000, 1500, 2000, 2300];
    $incrementos = [0, 500, 500, 300];

    $filas = $años->zip($salarios, $incrementos);
    $resultado = $filas->toList();

    // $resultado contiene
    [
        [2013, 1000, 0],
        [2014, 1500, 500],
        [2015, 2000, 500],
        [2016, 2300, 300]
    ]

Como ya puedes ver, el método ``zip()`` es muy útil para transponer matrices multidimensionales::

    $datos = [
        2014 => ['ene' => 100, 'feb' => 200],
        2015 => ['ene' => 300, 'feb' => 500],
        2016 => ['ene' => 400, 'feb' => 600],
    ];

    // Obteniendo datos de enero y febrero juntos

    $primerAño = new Collection(array_shift($datos));
    $resultado = $primerAño->zip($datos[0], $datos[1])->toList();

    // O $primerAño->zip(...$datos) en PHP >= 5.6

    // $resultado contiene
    [
        [100, 300, 400],
        [200, 500, 600]
    ]

Ordenación
==========

.. php:method:: sortBy($callback, $order = SORT_DESC, $sort = SORT_NUMERIC)

Los valores de una colección se pueden ordenar en orden ascendente o descendente basándose en una columna o función personalizada. Para crear una nueva colección ordenada a partir de los valores de otra, puedes usar ``sortBy``::

    $colección = new Collection($personas);
    $ordenada = $colección->sortBy('edad');

Como se muestra arriba, puedes ordenar pasando el nombre de una columna o propiedad que esté presente en los valores de la colección. También puedes especificar un camino de propiedad en lugar de usar la notación de punto. El siguiente ejemplo ordenará los artículos por el nombre de su autor::

    $colección = new Collection($artículos);
    $ordenada = $colección->sortBy('autor.nombre');

El método ``sortBy()`` es lo suficientemente flexible como para permitirte especificar una función de extracción que te permitirá seleccionar dinámicamente el valor a usar para comparar dos valores diferentes en la colección::

    $colección = new Collection($artículos);
    $ordenada = $colección->sortBy(function ($artículo) {
        return $artículo->autor->nombre . '-' . $artículo->título;
    });

Para especificar en qué dirección debe ordenarse la colección, debes proporcionar ya sea ``SORT_ASC`` o ``SORT_DESC`` como el segundo parámetro para ordenar en dirección ascendente o descendente, respectivamente. Por defecto, las colecciones se ordenan en dirección descendente::

    $colección = new Collection($personas);
    $ordenada = $colección->sortBy('edad', SORT_ASC);

A veces necesitarás especificar qué tipo de datos estás tratando de comparar para obtener resultados consistentes. Para este propósito, debes suministrar un tercer argumento en la función ``sortBy()`` con una de las siguientes constantes:

- **SORT_NUMERIC**: Para comparar números.
- **SORT_STRING**: Para comparar valores de cadena.
- **SORT_NATURAL**: Para ordenar cadenas que contienen números y deseas que esos números se ordenen de manera natural. Por ejemplo: mostrar "10" después de "2".
- **SORT_LOCALE_STRING**: Para comparar cadenas basadas en la configuración regional actual.

Por defecto, se utiliza ``SORT_NUMERIC``::

    $colección = new Collection($artículos);
    $ordenada = $colección->sortBy('título', SORT_ASC, SORT_NATURAL);

.. warning::

    A menudo es costoso iterar sobre colecciones ordenadas más de una vez. Si planeas hacerlo, considera convertir la colección en un array o simplemente usa el método ``compile()`` en ella.

Trabajando con Datos en Forma de Árbol
======================================

.. php:method:: nest($idPath, $parentPath, $nestingKey = 'children')

No todos los datos están destinados a ser representados de manera lineal. Las colecciones facilitan la construcción y aplanamiento de estructuras jerárquicas o anidadas. Crear una estructura anidada donde los hijos están agrupados por una propiedad de identificación del padre se puede hacer con el método ``nest()``.

Se requieren dos parámetros para esta función. El primero es la propiedad que representa la identificación del elemento. El segundo parámetro es el nombre de la propiedad que representa la identificación del elemento padre::

    $colección = new Collection([
        ['id' => 1, 'parent_id' => null, 'nombre' => 'Aves'],
        ['id' => 2, 'parent_id' => 1, 'nombre' => 'Aves Terrestres'],
        ['id' => 3, 'parent_id' => 1, 'nombre' => 'Águila'],
        ['id' => 4, 'parent_id' => 1, 'nombre' => 'Gaviota'],
        ['id' => 5, 'parent_id' => 6, 'nombre' => 'Pez Payaso'],
        ['id' => 6, 'parent_id' => null, 'nombre' => 'Peces'],
    ]);
    $anidada = $colección->nest('id', 'parent_id');
    $resultado = $anidada->toList();

    // $resultado contiene
    [
        [
            'id' => 1,
            'parent_id' => null,
            'nombre' => 'Aves',
            'children' => [
                ['id' => 2, 'parent_id' => 1, 'nombre' => 'Aves Terrestres', 'children' => []],
                ['id' => 3, 'parent_id' => 1, 'nombre' => 'Águila', 'children' => []],
                ['id' => 4, 'parent_id' => 1, 'nombre' => 'Gaviota', 'children' => []],
            ],
        ],
        [
            'id' => 6,
            'parent_id' => null,
            'nombre' => 'Peces',
            'children' => [
                ['id' => 5, 'parent_id' => 6, 'nombre' => 'Pez Payaso', 'children' => []],
            ],
        ],
    ];

Los elementos hijos están anidados dentro de la propiedad ``children`` dentro de cada uno de los elementos en la colección. Este tipo de representación de datos es útil para renderizar menús o recorrer elementos hasta cierto nivel en el árbol.

.. php:method:: listNested($order = 'desc', $nestingKey = 'children')

El inverso de ``nest()`` es ``listNested()``. Este método te permite aplanar una estructura de árbol de nuevo en una estructura lineal. Toma dos parámetros; el primero es el modo de recorrido (asc, desc o leaves), y el segundo es el nombre de la propiedad que contiene los hijos para cada elemento en la colección.

Tomando la entrada de la colección anidada construida en el ejemplo anterior, podemos aplanarla::

    $resultado = $anidada->listNested()->toList();

    // $resultado contiene
    [
        ['id' => 1, 'parent_id' => null, 'nombre' => 'Aves', 'children' => [...]],
        ['id' => 2, 'parent_id' => 1, 'nombre' => 'Aves Terrestres'],
        ['id' => 3, 'parent_id' => 1, 'nombre' => 'Águila'],
        ['id' => 4, 'parent_id' => 1, 'nombre' => 'Gaviota'],
        ['id' => 6, 'parent_id' => null, 'nombre' => 'Peces', 'children' => [...]],
        ['id' => 5, 'parent_id' => 6, 'nombre' => 'Pez Payaso']
    ]

De forma predeterminada, el árbol se recorre desde la raíz hasta las hojas. También puedes indicar que solo devuelva los elementos hoja en el árbol::

    $resultado = $anidada->listNested('leaves')->toList();

    // $resultado contiene
    [
        ['id' => 2, 'parent_id' => 1, 'nombre' => 'Aves Terrestres', 'children' => [], ],
        ['id' => 3, 'parent_id' => 1, 'nombre' => 'Águila', 'children' => [], ],
        ['id' => 4, 'parent_id' => 1, 'nombre' => 'Gaviota', 'children' => [], ],
        ['id' => 5, 'parent_id' => 6, 'nombre' => 'Pez Payaso', 'children' => [], ],
    ]


Una vez que has convertido un árbol en una lista anidada, puedes usar el método ``printer()``
para configurar cómo debe formatearse la salida de la lista::

    $resultado = $anidada->listNested()->printer('nombre', 'id', '--')->toArray();

    // $resultado contiene
    [
        1 => 'Aves',
        2 => '--Aves Terrestres',
        3 => '--Águila',
        4 => '--Gaviota',
        6 => 'Peces',
        5 => '--Pez Payaso',
    ]

El método ``printer()`` también te permite usar una devolución de llamada para generar las claves y/o valores::

    $anidada->listNested()->printer(
        function ($el) {
            return $el->nombre;
        },
        function ($el) {
            return $el->id;
        }
    );

Otros Métodos
=============

.. php:method:: isEmpty()

Te permite ver si una colección contiene algún elemento::

    $colección = new Collection([]);
    // Devuelve true
    $colección->isEmpty();

    $colección = new Collection([1]);
    // Devuelve false
    $colección->isEmpty();

.. php:method:: contains($value)

Las colecciones te permiten verificar rápidamente si contienen un valor en particular: utilizando el método ``contains()``::

    $elementos = ['a' => 1, 'b' => 2, 'c' => 3];
    $colección = new Collection($elementos);
    $tieneTres = $colección->contains(3);

Las comparaciones se realizan utilizando el operador ``===``. Si deseas realizar comparaciones menos estrictas, puedes usar el método ``some()``.

.. php:method:: shuffle()

A veces puede que desees mostrar una colección de valores en un orden aleatorio. Para crear una nueva colección que devolverá cada valor en una posición aleatoria, utiliza ``shuffle``::

    $colección = new Collection(['a' => 1, 'b' => 2, 'c' => 3]);

    // Esto podría devolver [2, 3, 1]
    $colección->shuffle()->toList();

.. php:method:: transpose()

Cuando transpones una colección, obtienes una nueva colección que contiene una fila hecha de cada una de las columnas originales::

    $elementos = [
        ['Productos', '2012', '2013', '2014'],
        ['Producto A', '200', '100', '50'],
        ['Producto B', '300', '200', '100'],
        ['Producto C', '400', '300', '200'],
    ];
    $transpuesta = (new Collection($elementos))->transpose();
    $resultado = $transpuesta->toList();

    // $resultado contiene
    [
        ['Productos', 'Producto A', 'Producto B', 'Producto C'],
        ['2012', '200', '300', '400'],
        ['2013', '100', '200', '300'],
        ['2014', '50', '100', '200'],
    ]

Extracción de Elementos
-----------------------

.. php:method:: sample($length = 10)

Barajar una colección a menudo es útil al realizar un análisis estadístico rápido. Otra operación común al hacer este tipo de tarea es extraer algunos valores aleatorios de una colección para realizar más pruebas con ellos. Por ejemplo, si quisieras seleccionar 5 usuarios aleatorios a los que le aplicarás algunas pruebas A/B, puedes usar la función ``sample()``::

    $colección = new Collection($personas);

    // Extraer como máximo 20 usuarios aleatorios de esta colección
    $sujetosPrueba = $colección->sample(20);

``sample()`` tomará como máximo el número de valores que especifiques en el primer
argumento. Si no hay suficientes elementos en la colección para satisfacer la
muestra, se devolverá la colección completa en un orden aleatorio.

.. php:method:: take($length, $offset)

Cuando desees tomar un segmento de una colección, utiliza la función ``take()``, creará una nueva colección con como máximo el número de valores que especifiques en el primer argumento, comenzando desde la posición indicada en el segundo argumento::

    $cincoPrimeros = $colección->sortBy('edad')->take(5);

    // Tomar 5 personas de la colección empezando desde la posición 4
    $próximosCinco = $colección->sortBy('edad')->take(5, 4);

Las posiciones son de base cero, por lo tanto, el primer número de posición es ``0``.

.. php:method:: skip($length)

Mientras que el segundo argumento de ``take()`` puede ayudarte a saltar algunos elementos antes de obtenerlos de la colección, también puedes usar ``skip()`` para el mismo propósito como una manera de tomar el resto de los elementos después de cierta posición::

    $colección = new Collection([1, 2, 3, 4]);
    $todosExceptoPrimerosDos = $colección->skip(2)->toList(); // [3, 4]

.. php:method:: first()

Uno de los usos más comunes de ``take()`` es obtener el primer elemento en la colección. Un método abreviado para lograr el mismo objetivo es usando el método ``first()``::

    $colección = new Collection([5, 4, 3, 2]);
    $colección->first(); // Devuelve 5

.. php:method:: last()

De manera similar, puedes obtener el último elemento de una colección usando el método ``last()``::

    $colección = new Collection([5, 4, 3, 2]);
    $colección->last(); // Devuelve 2

Expansión de Colecciones
------------------------

.. php:method:: append(array|Traversable $items)

Puedes componer múltiples colecciones en una sola. Esto te permite recopilar datos de diversas fuentes, concatenarlos y aplicar otras funciones de colección de manera muy

 fluida. El método ``append()`` devolverá una nueva colección que contiene los valores de ambas fuentes::

    $tweetsCakePHP = new Collection($tweets);
    $miLíneaDeTiempo = $tweetsCakePHP->append($tweetsPHP);

    // Tweets que contienen `cakefest` de ambas fuentes
    $miLíneaDeTiempo->filter(function ($tweet) {
        return strpos($tweet, 'cakefest');
    });

.. php:method:: appendItem($value, $key)

Te permite agregar un elemento con una clave opcional a la colección. Si especificas una clave que ya existe en la colección, el valor no se sobrescribirá::

    $tweetsCakePHP = new Collection($tweets);
    $miLíneaDeTiempo = $tweetsCakePHP->appendItem($nuevoTweet, 99);

.. php:method:: prepend($items)

El método ``prepend()`` devolverá una nueva colección que contiene los valores de ambas fuentes::

    $tweetsCakePHP = new Collection($tweets);
    $miLíneaDeTiempo = $tweetsCakePHP->prepend($tweetsPHP);

.. php:method:: prependItem($value, $key)

Te permite agregar un elemento con una clave opcional a la colección. Si especificas una clave que ya existe en la colección, el valor no se sobrescribirá::

    $tweetsCakePHP = new Collection($tweets);
    $miLíneaDeTiempo = $tweetsCakePHP->prependItem($nuevoTweet, 99);

.. warning::

    Al agregar desde diferentes fuentes, puedes esperar que algunas claves de ambas
    colecciones sean iguales. Por ejemplo, al agregar dos arrays simples.
    Esto puede presentar un problema al convertir una colección a un array usando
    ``toArray()``. Si no quieres que los valores de una colección sobrescriban
    a otros en la colección anterior basándote en su clave, asegúrate de llamar
    a ``toList()`` para eliminar las claves y preservar todos los valores.

Modificación de Elementos
-------------------------

.. php:method:: insert($path, $items)

En ocasiones, es posible que tengas dos conjuntos de datos separados que te gustaría insertar en cada uno de los elementos del otro conjunto. Este es un caso muy común cuando obtienes datos de una fuente de datos que no admite combinación de datos o uniones de forma nativa.

Las colecciones ofrecen un método ``insert()`` que te permitirá insertar cada uno de los elementos de una colección en una propiedad dentro de cada uno de los elementos de otra colección::

    $usuarios = [
        ['username' => 'mark'],
        ['username' => 'juan'],
        ['username' => 'jose']
    ];

    $lenguajes = [
        ['PHP', 'Python', 'Ruby'],
        ['Bash', 'PHP', 'Javascript'],
        ['Javascript', 'Prolog']
    ];

    $fusionado = (new Collection($usuarios))->insert('skills', $lenguajes);
    $resultado = $fusionado->toArray();

    // $resultado contiene
    [
        ['username' => 'mark', 'skills' => ['PHP', 'Python', 'Ruby']],
        ['username' => 'juan', 'skills' => ['Bash', 'PHP', 'Javascript']],
        ['username' => 'jose', 'skills' => ['Javascript', 'Prolog']]
    ];

El primer parámetro para el método ``insert()`` es una ruta de propiedades separada por puntos para seguir, de modo que los elementos se puedan insertar en esa posición. El segundo argumento puede ser cualquier cosa que se pueda convertir en un objeto de colección.

Observa que los elementos se insertan en la posición en la que se encuentran, por lo tanto, el primer elemento del segundo conjunto se fusiona en el primer elemento del primer conjunto.

Si no hay suficientes elementos en el segundo conjunto para insertar en el primero, entonces la propiedad de destino no estará presente::

    $lenguajes = [
        ['PHP', 'Python', 'Ruby'],
        ['Bash', 'PHP', 'Javascript']
    ];

    $fusionado = (new Collection($usuarios))->insert('skills', $lenguajes);
    $resultado = $fusionado->toArray();

    // $resultado contiene
    [
        ['username' => 'mark', 'skills' => ['PHP', 'Python', 'Ruby']],
        ['username' => 'juan', 'skills' => ['Bash', 'PHP', 'Javascript']],
        ['username' => 'jose']
    ];

El método ``insert()`` puede operar en elementos de arrays u objetos que implementen la interfaz ``ArrayAccess``.

Haciendo Métodos de Colección Reutilizables
-------------------------------------------

El uso de cierres para los métodos de colección es excelente cuando el trabajo a realizar es pequeño y enfocado, pero puede volverse desordenado rápidamente. Esto se hace más evidente cuando se deben llamar a muchos métodos diferentes o cuando la longitud de los métodos de cierre es más que solo unas pocas líneas.

También hay casos en los que la lógica utilizada para los métodos de colección se puede reutilizar en varias partes de tu aplicación. Se recomienda considerar la posibilidad de extraer la lógica de colección compleja a clases separadas. Por ejemplo, imagina un cierre extenso como este::

        $colección
                ->map(function ($fila, $clave) {
                    if (!empty($fila['items'])) {
                        $fila['total'] = collection($fila['items'])->sumOf('price');
                    }

                    if (!empty($fila['total'])) {
                        $fila['tax_amount'] = $fila['total'] * 0.25;
                    }

                    // Más código aquí...

                    return $filaModificada;
                });

Esto se puede refactorizar creando otra clase::

        class TotalOrderCalculator
        {
                public function __invoke($fila, $clave)
                {
                    if (!empty($fila['items'])) {
                        $fila['total'] = collection($fila['items'])->sumOf('price');
                    }

                    if (!empty($fila['total'])) {
                        $fila['tax_amount'] = $fila['total'] * 0.25;
                    }

                    // Más código aquí...

                    return $filaModificada;
                }
        }

        // Utiliza la lógica en tu llamada map()
        $colección->map(new TotalOrderCalculator)

.. php:method:: through($callback)

A veces, una cadena de llamadas a métodos de colección puede volverse reutilizable en otras partes de tu aplicación, pero solo si se llaman en ese orden específico. En esos casos, puedes usar ``through()`` en combinación con una clase que implemente ``__invoke`` para distribuir tus llamadas útiles de procesamiento de datos::

        $colección
                ->map(new ShippingCostCalculator)
                ->map(new TotalOrderCalculator)
                ->map(new GiftCardPriceReducer)
                ->buffered()
               ...

Las llamadas a métodos anteriores se pueden extraer a una nueva clase para que no necesiten repetirse cada vez::

        class FinalCheckOutRowProcessor
        {
                public function __invoke($colección)
                {
                        return $colección
                                ->map(new ShippingCostCalculator)
                                ->map(new TotalOrderCalculator)
                                ->map(new GiftCardPriceReducer)
                                ->buffered()
                               ...
                }
        }

        // Ahora puedes usar el método through() para llamar a todos los métodos a la vez
        $colección->through(new FinalCheckOutRowProcessor);

Optimización de Colecciones
---------------------------

.. php:method:: buffered()

A menudo, las colecciones realizan la mayoría de las operaciones que creas usando sus funciones de manera perezosa. Esto significa que, aunque puedas llamar a una función, no significa que se ejecute de inmediato. Esto es cierto para una gran cantidad de funciones en esta clase. La evaluación perezosa te permite ahorrar recursos en situaciones en las que no usas todos los valores en una colección. Es posible que no uses todos los valores cuando la iteración se detiene temprano o cuando se alcanza un caso de excepción/error temprano.

Además, la evaluación perezosa ayuda a acelerar algunas operaciones. Considera el siguiente ejemplo::

    $colección = new Collection($unMillonDeElementos);
    $colección = $colección->map(function ($elemento) {
        return $elemento * 2;
    });
    $elementosAMostrar = $colección->take(30);

Si las colecciones no fueran perezosas, habríamos ejecutado un millón de operaciones, aunque solo quisiéramos mostrar 30 elementos. En cambio, nuestra operación de map se aplicó solo a los 30 elementos que usamos. También podemos obtener beneficios de esta evaluación perezosa para colecciones

 más pequeñas cuando realizamos más de una operación en ellas. Por ejemplo: llamando a ``map()`` dos veces y luego ``filter()``.

La evaluación perezosa también tiene sus inconvenientes. Podrías estar realizando las mismas operaciones más de una vez si optimizas una colección prematuramente. Considera este ejemplo::

    $edades = $colección->extract('age');

    $menorDe30 = $edades->filter(function ($elemento) {
        return $elemento < 30;
    });

    $mayorDe30 = $edades->filter(function ($elemento) {
        return $elemento > 30;
    });

Si iteramos tanto ``menorDe30`` como ``mayorDe30``, lamentablemente, la colección ejecutaría la operación ``extract()`` dos veces. Esto se debe a que las colecciones son inmutables y la operación de extracción perezosa se realizaría para ambos filtros.

Afortunadamente, podemos superar este problema con una sola función. Si planeas reutilizar los valores de ciertas operaciones más de una vez, puedes compilar los resultados en otra colección usando la función ``buffered()``::

    $edades = $colección->extract('age')->buffered();
    $menorDe30 = ...
    $mayorDe30 = ...

Ahora, cuando ambas colecciones se iteran, solo llamarán a la operación de extracción una vez.

Haciendo Colecciones Reiniciables
---------------------------------

El método ``buffered()`` también es útil para convertir iteradores no reiniciables en colecciones que se pueden iterar más de una vez::

    // En PHP 5.5+
    public function results()
    {
        ...
        foreach ($elementosTransitorios as $e) {
            yield $e;
        }
    }
    $reinicable = (new Collection(results()))->buffered();

Clonación de Colecciones
------------------------

.. php:method:: compile($preserveKeys = true)

A veces necesitas obtener un clon de los elementos de otra colección. Esto es útil cuando necesitas iterar el mismo conjunto desde lugares diferentes al mismo tiempo. Para clonar una colección a partir de otra, utiliza el método ``compile()``::

    $edades = $colección->extract('age')->compile();

    foreach ($edades as $edad) {
        foreach ($colección as $elemento) {
            echo h($elemento->name) . ' - ' . $edad;
        }
    }

.. meta::
    :title lang=es: Colecciones
    :keywords lang=es: colecciones, cakephp, anexar, ordenar, compilar, contiene, contarPor, cada, todos, extraer, filtrar, primero, primerCoincidencia, agruparPor, indexarPor, jsonSerialize, map, coincidir, máximo, mínimo, reducir, rechazar, muestra, mezclar, algunos, aleatorio, ordenarPor, tomar, toArray, insertar
