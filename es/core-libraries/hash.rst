Hash
####

.. php:namespace:: Cake\Utility

.. php:class:: Hash

La gestión de matrices, si se hace correctamente, puede ser una herramienta muy potente y útil para construir código más inteligente y optimizado. CakePHP ofrece un conjunto muy útil de utilidades estáticas en la clase Hash que te permiten hacer exactamente eso.

La clase Hash de CakePHP se puede llamar desde cualquier modelo o controlador de la misma manera que se llama a Inflector. Ejemplo: :php:meth:`Hash::combine()`.

.. _hash-path-syntax:

Sintaxis de Ruta de Hash
========================

La sintaxis de ruta descrita a continuación es utilizada por todos los métodos en ``Hash``. No todas las partes de la sintaxis de ruta están disponibles en todos los métodos. Una expresión de ruta está compuesta por cualquier número de tokens. Los tokens están compuestos por dos grupos. Las expresiones se utilizan para recorrer los datos de la matriz, mientras que los matchers se utilizan para calificar elementos. Aplicas matchers a los elementos de la expresión.

Tipos de Expresiones
--------------------

+--------------------------------+--------------------------------------------+
| Expresión                      | Definición                                 |
+================================+============================================+
| ``{n}``                        | Representa una clave numérica. Coincidirá  |
|                                | con cualquier clave de cadena o numérica.  |
+--------------------------------+--------------------------------------------+
| ``{s}``                        | Representa una cadena. Coincidirá con      |
|                                | cualquier valor de cadena, incluidos los   |
|                                | valores de cadena numérica.                |
+--------------------------------+--------------------------------------------+
| ``{*}``                        | Coincide con cualquier valor.              |
+--------------------------------+--------------------------------------------+
| ``Foo``                        | Coincide con claves con el mismo valor.    |
+--------------------------------+--------------------------------------------+

Todos los elementos de expresión son compatibles con todos los métodos. Además de los elementos de expresión, puedes usar la coincidencia de atributos con ciertos métodos. Son ``extract()``, ``combine()``, ``format()``, ``check()``, ``map()``, ``reduce()``, ``apply()``, ``sort()``, ``insert()``, ``remove()`` y ``nest()``.

Tipos de Coincidencia de Atributos
----------------------------------

+--------------------------------+--------------------------------------------+
| Matcher                        | Definición                                 |
+================================+============================================+
| ``[id]``                       | Coincidir con elementos con una clave de   |
|                                | matriz dada.                               |
+--------------------------------+--------------------------------------------+
| ``[id=2]``                     | Coincidir con elementos con id igual a 2.  |
+--------------------------------+--------------------------------------------+
| ``[id!=2]``                    | Coincidir con elementos con id diferente   |
|                                | de 2.                                      |
+--------------------------------+--------------------------------------------+
| ``[id>2]``                     | Coincidir con elementos con id mayor que   |
|                                | 2.                                         |
+--------------------------------+--------------------------------------------+
| ``[id>=2]``                    | Coincidir con elementos con id mayor o     |
|                                | igual a 2.                                 |
+--------------------------------+--------------------------------------------+
| ``[id<2]``                     | Coincidir con elementos con id menor que   |
|                                | 2.                                         |
+--------------------------------+--------------------------------------------+
| ``[id<=2]``                    | Coincidir con elementos con id menor o     |
|                                | igual a 2.                                 |
+--------------------------------+--------------------------------------------+
| ``[text=/.../]``               | Coincidir con elementos que tienen valores |
|                                | que coinciden con la expresión regular     |
|                                | dentro de ``...``.                         |
+--------------------------------+--------------------------------------------+

.. php:staticmethod:: get(array|\ArrayAccess $data, $path, $default = null)

    ``get()`` es una versión simplificada de ``extract()``, solo admite expresiones de ruta directas. Las rutas con ``{n}``, ``{s}``, ``{*}`` o matchers no son compatibles. Usa ``get()`` cuando quieras exactamente un valor de una matriz. Si no se encuentra una ruta coincidente, se devolverá el valor predeterminado.

.. php:staticmethod:: extract(array|\ArrayAccess $data, $path)

    ``Hash::extract()`` admite todos los componentes de expresiones y matchers de
    :ref:`hash-path-syntax`. Puedes usar extract para recuperar datos de matrices
    u objetos que implementen la interfaz ``ArrayAccess``, a lo largo de rutas arbitrarias
    rápidamente sin tener que recorrer las estructuras de datos. En su lugar, usas
    expresiones de ruta para calificar qué elementos quieres devolver ::

        // Uso común:
        $usuarios = [
            ['id' => 1, 'nombre' => 'mark'],
            ['id' => 2, 'nombre' => 'jane'],
            ['id' => 3, 'nombre' => 'sally'],
            ['id' => 4, 'nombre' => 'jose'],
        ];
        $resultados = Hash::extract($usuarios, '{n}.id');
        // $resultados es igual a:
        // [1,2,3,4];

.. php:staticmethod:: Hash::insert(array $data, $path, $values = null)

    Inserta ``$values`` en una matriz según lo definido por ``$path``::

        $a = [
            'páginas' => ['nombre' => 'página']
        ];
        $resultado = Hash::insert($a, 'archivos', ['nombre' => 'archivos']);
        // $resultado ahora se ve así:
        [
            [páginas] => [
                [nombre] => página
            ]
            [archivos] => [
                [nombre] => archivos
            ]
        ]

    Puedes usar rutas usando ``{n}``, ``{s}`` y ``{*}`` para insertar datos en múltiples
    puntos::

        $usuarios = Hash::insert($usuarios, '{n}.nuevo', 'valor');

    Los matchers de atributos también funcionan con ``insert()``::

        $datos = [
            0 => ['up' => true, 'Item' => ['id' => 1, 'título' => 'primero']],
            1 => ['Item' => ['id' => 2, 'título' => 'segundo']],
            2 => ['Item' => ['id' => 3, 'título' => 'tercero']],
            3 => ['up' => true, 'Item' => ['id' => 4, 'título' => 'cuarto']],
            4 => ['Item' => ['id' => 5, 'título' => 'quinto']],
        ];
        $resultado = Hash::insert($datos, '{n}[up].Item[id=4].nuevo', 9);
        /* $resultado ahora se ve así:
            [
                ['up' => true, 'Item' => ['id' => 1, 'título' => 'primero']],
                ['Item' => ['id' => 2, 'título' => 'segundo']],
                ['Item' => ['id' => 3, 'título' => 'tercero']],
                ['up' => true, 'Item' => ['id' => 4, 'título' => 'cuarto', 'nuevo' => 9]],
                ['Item' => ['id' => 5, 'título' => 'quinto']],
            ]
        */

.. php:staticmethod:: remove(array $data, $path)

    Elimina todos los elementos de una matriz que coinciden con ``$path``. ::

        $a = [
            'páginas' => ['nombre' => 'página'],
            'archivos' => ['nombre' => 'archivos']
        ];
        $resultado = Hash::remove($a, 'archivos');
        /* $resultado ahora se ve así:
            [
                [páginas] => [
                    [nombre] => página
                ]

            ]
        */

    Usando ``{n}``, ``{s}`` y ``{*}`` te permitirá eliminar varios valores a la vez.
    También puedes usar matchers de atributos con ``remove()``::

        $datos = [
            0 => ['clear' => true, 'Item' => ['id' => 1, 'título' => 'primero']],
            1 => ['Item' => ['id' => 2, 'título' => 'segundo']],
            2 => ['Item' => ['id' => 3, 'título' => 'tercero']],
            3 => ['clear' => true, 'Item' => ['id' => 4, 'título' => 'cuarto']],
            4 => ['Item' => ['id' => 5, 'título' => 'quinto']],
        ];
        $resultado = Hash::remove($datos, '{n}[clear].Item[id=4]');
        /* $resultado ahora se ve así:
            [
                ['clear' => true, 'Item' => ['id' => 1, 'título' => 'primero']],
                ['Item' => ['id' => 2, 'título' => 'segundo']],
                ['Item' => ['id' => 3, 'título' => 'tercero']],
                ['clear' => true],
                ['Item' => ['id' => 5, 'título' => 'quinto']],
            ]
        */

.. php:staticmethod:: combine(array $data, $keyPath, $valuePath = null, $groupPath = null)

    Crea un array asociativo utilizando ``$keyPath`` como la ruta para construir sus claves,
    y opcionalmente ``$valuePath`` como la ruta para obtener los valores. Si ``$valuePath`` no está
    especificado, o no coincide con nada, los valores se inicializarán a null.
    Opcionalmente, puedes agrupar los valores por lo que se obtiene al seguir la
    ruta especificada en ``$groupPath``. ::

        $a = [
            [
                'User' => [
                    'id' => 2,
                    'group_id' => 1,
                    'Data' => [
                        'user' => 'mariano.iglesias',
                        'name' => 'Mariano Iglesias'
                    ]
                ]
            ],
            [
                'User' => [
                    'id' => 14,
                    'group_id' => 2,
                    'Data' => [
                        'user' => 'phpnut',
                        'name' => 'Larry E. Masters'
                    ]
                ]
            ],
        ];

        $result = Hash::combine($a, '{n}.User.id');
        /* $result ahora luce así:
            [
                [2] =>
                [14] =>
            ]
        */

        $result = Hash::combine($a, '{n}.User.id', '{n}.User.Data.user');
        /* $result ahora luce así:
            [
                [2] => 'mariano.iglesias'
                [14] => 'phpnut'
            ]
        */

        $result = Hash::combine($a, '{n}.User.id', '{n}.User.Data');
        /* $result ahora luce así:
            [
                [2] => [
                        [user] => mariano.iglesias
                        [name] => Mariano Iglesias
                ]
                [14] => [
                        [user] => phpnut
                        [name] => Larry E. Masters
                ]
            ]
        */

        $result = Hash::combine($a, '{n}.User.id', '{n}.User.Data.name');
        /* $result ahora luce así:
            [
                [2] => Mariano Iglesias
                [14] => Larry E. Masters
            ]
        */

        $result = Hash::combine($a, '{n}.User.id', '{n}.User.Data', '{n}.User.group_id');
        /* $result ahora luce así:
            [
                [1] => [
                        [2] => [
                                [user] => mariano.iglesias
                                [name] => Mariano Iglesias
                        ]
                ]
                [2] => [
                        [14] => [
                                [user] => phpnut
                                [name] => Larry E. Masters
                        ]
                ]
            ]
        */

        $result = Hash::combine($a, '{n}.User.id', '{n}.User.Data.name', '{n}.User.group_id');
        /* $result ahora luce así:
            [
                [1] => [
                        [2] => Mariano Iglesias
                ]
                [2] => [
                        [14] => Larry E. Masters
                ]
            ]
        */

        // Desde la versión 3.9.0 $keyPath puede ser null
        $result = Hash::combine($a, null, '{n}.User.Data.name');
        /* $result ahora luce así:
            [
                [0] => Mariano Iglesias
                [1] => Larry E. Masters
            ]
        */

    Puedes proporcionar arrays tanto para ``$keyPath`` como para ``$valuePath``. Si haces esto,
    el primer valor se usará como una cadena de formato, para los valores extraídos por las
    otras rutas::

        $result = Hash::combine(
            $a,
            '{n}.User.id',
            ['%s: %s', '{n}.User.Data.user', '{n}.User.Data.name'],
            '{n}.User.group_id'
        );
        /* $result ahora luce así:
            [
                [1] => [
                        [2] => mariano.iglesias: Mariano Iglesias
                ]
                [2] => [
                        [14] => phpnut: Larry E. Masters
                ]
            ]
        */

        $result = Hash::combine(
            $a,
            ['%s: %s', '{n}.User.Data.user', '{n}.User.Data.name'],
            '{n}.User.id'
        );
        /* $result ahora luce así:
            [
                [mariano.iglesias: Mariano Iglesias] => 2
                [phpnut: Larry E. Masters] => 14
            ]
        */

.. php:staticmethod:: format(array $data, array $paths, $format)

    Devuelve una serie de valores extraídos de un array, formateados con una cadena de formato::

        $data = [
            [
                'Person' => [
                    'first_name' => 'Nate',
                    'last_name' => 'Abele',
                    'city' => 'Boston',
                    'state' => 'MA',
                    'something' => '42'
                ]
            ],
            [
                'Person' => [
                    'first_name' => 'Larry',
                    'last_name' => 'Masters',
                    'city' => 'Boondock',
                    'state' => 'TN',
                    'something' => '{0}'
                ]
            ],
            [
                'Person' => [
                    'first_name' => 'Garrett',
                    'last_name' => 'Woodworth',
                    'city' => 'Venice Beach',
                    'state' => 'CA',
                    'something' => '{1}'
                ]
            ]
        ];

        $res = Hash::format($data, ['{n}.Person.first_name', '{n}.Person.something'], '%2$d, %1$s');
        /*
        [
            [0] => 42, Nate
            [1] => 0, Larry
            [2] => 0, Garrett
        ]
        */

        $res = Hash::format($data, ['{n}.Person.first_name', '{n}.Person.something'], '%1$s, %2$d');
        /*
        [
            [0] => Nate, 42
            [1] => Larry, 0
            [2] => Garrett, 0
        ]
        */

.. php:staticmethod:: contains(array $data, array $needle)

    Determina si un Hash o un array contiene exactamente las claves y valores de otro::

        $a = [
            0 => ['name' => 'main'],
            1 => ['name' => 'about']
        ];
        $b = [
            0 => ['name' => 'main'],
            1 => ['name' => 'about'],
            2 => ['name' => 'contact'],
            'a' => 'b',
        ];

        $result = Hash::contains($a, $a);
        // true
        $result = Hash::contains($a, $b);
        // false
        $result = Hash::contains($b, $a);
        // true

.. php:staticmethod:: check(array $data, string $path = null)

    Comprueba si una ruta particular está establecida en un array::

        $set = [
            'My Index 1' => ['First' => 'The first item']
        ];
        $result = Hash::check($set, 'My Index 1.First');
        // $result == true

        $result = Hash::check($set, 'My Index 1');
        // $result == true

        $set = [
            'My Index 1' => [
                'First' => [
                    'Second' => [
                        'Third' => [
                            'Fourth' => 'Heavy. Nesting.'
                        ]
                    ]
                ]
            ]
        ];
        $result = Hash::check($set, 'My Index 1.First.Second');
        // $result == true

        $result = Hash::check($set, 'My Index 1.First.Second.Third');
        // $result == true

        $result = Hash::check($set, 'My Index 1.First.Second.Third.Fourth');
        // $result == true

        $result = Hash::check($set, 'My Index 1.First.Seconds.Third.Fourth');
        // $result == false

.. php:staticmethod:: filter(array $data, $callback = ['Hash', 'filter'])

    Filtra los elementos vacíos de un array, excluyendo '0'. También puedes proporcionar un ``$callback`` personalizado para filtrar los elementos del array. El callback debería devolver ``false`` para eliminar elementos del array resultante::

        $data = [
            '0',
            false,
            true,
            0,
            ['una cosa', 'te puedo decir', 'es que tienes que ser', false]
        ];
        $res = Hash::filter($data);

        /* $res ahora luce así:
            [
                [0] => 0
                [2] => true
                [3] => 0
                [4] => [
                        [0] => una cosa
                        [1] => te puedo decir
                        [2] => es que tienes que ser
                ]
            ]
        */

.. php:staticmethod:: flatten(array $data, string $separator = '.')

    Colapsa un array multidimensional en una sola dimensión::

        $arr = [
            [
                'Post' => ['id' => '1', 'title' => 'Primer Post'],
                'Author' => ['id' => '1', 'user' => 'Kyle'],
            ],
            [
                'Post' => ['id' => '2', 'title' => 'Segundo Post'],
                'Author' => ['id' => '3', 'user' => 'Crystal'],
            ],
        ];
        $res = Hash::flatten($arr);
        /* $res ahora luce así:
            [
                [0.Post.id] => 1
                [0.Post.title] => Primer Post
                [0.Author.id] => 1
                [0.Author.user] => Kyle
                [1.Post.id] => 2
                [1.Post.title] => Segundo Post
                [1.Author.id] => 3
                [1.Author.user] => Crystal
            ]
        */

.. php:staticmethod:: expand(array $data, string $separator = '.')

    Expande un array que fue previamente aplanado con :php:meth:`Hash::flatten()`::

        $data = [
            '0.Post.id' => 1,
            '0.Post.title' => Primer Post,
            '0.Author.id' => 1,
            '0.Author.user' => Kyle,
            '1.Post.id' => 2,
            '1.Post.title' => Segundo Post,
            '1.Author.id' => 3,
            '1.Author.user' => Crystal,
        ];
        $res = Hash::expand($data);
        /* $res ahora luce así:
        [
            [
                'Post' => ['id' => '1', 'title' => 'Primer Post'],
                'Author' => ['id' => '1', 'user' => 'Kyle'],
            ],
            [
                'Post' => ['id' => '2', 'title' => 'Segundo Post'],
                'Author' => ['id' => '3', 'user' => 'Crystal'],
            ],
        ];
        */

.. php:staticmethod:: merge(array $data, array $merge[, array $n])

    Esta función puede pensarse como una combinación entre ``array_merge`` y ``array_merge_recursive`` de PHP. La diferencia con ambas es que si una clave de array contiene otro array, entonces la función se comporta recursivamente (a diferencia de ``array_merge``), pero no lo hace para claves que contienen cadenas (a diferencia de ``array_merge_recursive``).

    .. note::

        Esta función funcionará con una cantidad ilimitada de argumentos y convertirá parámetros no-array en arrays.

    ::

        $array = [
            [
                'id' => '48c2570e-dfa8-4c32-a35e-0d71cbdd56cb',
                'name' => 'mysql raleigh-workshop-08 < 2008-09-05.sql ',
                'description' => 'Importando un volcado sql',
            ],
            [
                'id' => '48c257a8-cf7c-4af2-ac2f-114ecbdd56cb',
                'name' => 'pbpaste | grep -i Unpaid | pbcopy',
                'description' => 'Eliminar todas las líneas que dicen "Unpaid".',
            ]
        ];
        $arrayB = 4;
        $arrayC = [0 => "array de prueba", "gatos" => "perros", "personas" => 1267];
        $arrayD = ["gatos" => "felinos", "perro" => "enojado"];
        $res = Hash::merge($array, $arrayB, $arrayC, $arrayD);

        /* $res ahora luce así:
        [
            [0] => [
                    [id] => 48c2570e-dfa8-4c32-a35e-0d71cbdd56cb
                    [name] => mysql raleigh-workshop-08 < 2008-09-05.sql
                    [description] => Importando un volcado sql
            ]
            [1] => [
                    [id] => 48c257a8-cf7c-4af2-ac2f-114ecbdd56cb
                    [name] => pbpaste | grep -i Unpaid | pbcopy
                    [description] => Eliminar todas las líneas que dicen "Unpaid".
            ]
            [2] => 4
            [3] => array de prueba
            [gatos] => felinos
            [personas] => 1267
            [perro] => enojado
        ]
        */

.. php:staticmethod:: numeric(array $data)

    Verifica si todos los valores en el array son numéricos::

        $data = ['uno'];
        $res = Hash::numeric(array_keys($data));
        // $res es true

        $data = [1 => 'uno'];
        $res = Hash::numeric($data);
        // $res es false

.. php:staticmethod:: dimensions (array $data)

    Cuenta las dimensiones de un array. Este método solo considerará la dimensión del primer elemento en el array::

        $data = ['uno', '2', 'tres'];
        $result = Hash::dimensions($data);
        // $result == 1

        $data = ['1' => '1.1', '2', '3'];
        $result = Hash::dimensions($data);
        // $result == 1

        $data = ['1' => ['1.1' => '1.1.1'], '2', '3' => ['3.1' => '3.1.1']];
        $result = Hash::dimensions($data);
        // $result == 2

        $data = ['1' => '1.1', '2', '3' => ['3.1' => '3.1.1']];
        $result = Hash::dimensions($data);
        // $result == 1

        $data = ['1' => ['1.1' => '1.1.1'], '2', '3' => ['3.1' => ['3.1.1' => '3.1.1.1']]];
        $result = Hash::dimensions($data);
        // $result == 2

.. php:staticmethod:: maxDimensions(array $data)

    Similar a :php:meth:`~Hash::dimensions()`, sin embargo, este método devuelve el número más profundo de dimensiones de cualquier elemento en el array::

        $data = ['1' => '1.1', '2', '3' => ['3.1' => '3.1.1']];
        $result = Hash::maxDimensions($data);
        // $result == 2

        $data = ['1' => ['1.1' => '1.1.1'], '2', '3' => ['3.1' => ['3.1.1' => '3.1.1.1']]];
        $result = Hash::maxDimensions($data);
        // $result == 3

.. php:staticmethod:: map(array $data, $path, $function)

    Crea un nuevo array, extrayendo ``$path``, y mapeando ``$function`` a través de los resultados. Puedes usar tanto expresiones como elementos coincidentes con este método::

        // Llama a la función noop $this->noop() en cada elemento de $data
        $result = Hash::map($data, "{n}", [$this, 'noop']);

        public function noop(array $array)
        {
            // Haz algo con el array y devuelve el resultado
            return $array;
        }

.. php:staticmethod:: reduce(array $data, $path, $function)

    Crea un valor único, extrayendo ``$path``, y reduciendo los resultados extraídos con ``$function``. Puedes usar tanto expresiones como elementos coincidentes con este método.

.. php:staticmethod:: apply(array $data, $path, $function)

    Aplica un callback a un conjunto de valores extraídos usando ``$function``. La función obtendrá los valores extraídos como primer argumento::

        $data = [
            ['date' => '01-01-2016', 'booked' => true],
            ['date' => '01-01-2016', 'booked' => false],
            ['date' => '02-01-2016', 'booked' => true]
        ];
        $result = Hash::apply($data, '{n}[booked=true].date', 'array_count_values');
        /* $result ahora luce así:
            [
                '01-01-2016' => 1,
                '02-01-2016' => 1,
            ]
        */

.. php:staticmethod:: sort(array $data, $path, $dir, $type = 'regular')

    Ordena un array por cualquier valor, determinado por una :ref:`hash-path-syntax`
    Solo se admiten elementos de expresión con este método::

        $a = [
            0 => ['Person' => ['name' => 'Jeff']],
            1 => ['Shirt' => ['color' => 'black']]
        ];
        $result = Hash::sort($a, '{n}.Person.name', 'asc');
        /* $result ahora luce así:
            [
                [0] => [
                        [Shirt] => [
                                [color] => black
                        ]
                ]
                [1] => [
                        [Person] => [
                                [name] => Jeff
                        ]
                ]
            ]
        */

    ``$dir`` puede ser ``asc`` o ``desc``. ``$type``
    puede ser uno de los siguientes valores:

    * ``regular`` para ordenamiento regular.
    * ``numeric`` para ordenar los valores como sus equivalentes numéricos.
    * ``string`` para ordenar los valores como su valor de cadena.
    * ``natural`` para ordenar los valores de una manera amigable para humanos. Ordenará, por ejemplo, ``foo10`` debajo de ``foo2``.

.. php:staticmethod:: diff(array $data, array $compare)

    Calcula la diferencia entre dos arrays::

        $a = [
            0 => ['name' => 'main'],
            1 => ['name' => 'about']
        ];
        $b = [
            0 => ['name' => 'main'],
            1 => ['name' => 'about'],
            2 => ['name' => 'contact']
        ];

        $result = Hash::diff($a, $b);
        /* $result ahora luce así:
            [
                [2] => [
                        [name] => contact
                ]
            ]
        */

.. php:staticmethod:: mergeDiff(array $data, array $compare)

    Esta función fusiona dos arrays y empuja las diferencias en
    datos al final del array resultante.

    **Ejemplo 1**
    ::

        $array1 = ['ModelOne' => ['id' => 1001, 'field_one' => 'a1.m1.f1', 'field_two' => 'a1.m1.f2']];
        $array2 = ['ModelOne' => ['id' => 1003, 'field_one' => 'a3.m1.f1', 'field_two' => 'a3.m1.f2', 'field_three' => 'a3.m1.f3']];
        $res = Hash::mergeDiff($array1, $array2);

        /* $res ahora luce así:
            [
                [ModelOne] => [
                        [id] => 1001
                        [field_one] => a1.m1.f1
                        [field_two] => a1.m1.f2
                        [field_three] => a3.m1.f3
                    ]
            ]
        */

    **Ejemplo 2**
    ::

        $array1 = ["a" => "b", 1 => 20938, "c" => "cadena"];
        $array2 = ["b" => "b", 3 => 238, "c" => "cadena", ["campo_extra"]];
        $res = Hash::mergeDiff($array1, $array2);
        /* $res ahora luce así:
            [
                [a] => b
                [1] => 20938
                [c] => cadena
                [b] => b
                [3] => 238
                [4] => [
                        [0] => campo_extra
                ]
            ]
        */

.. php:staticmethod:: normalize(array $data, $assoc = true, $default = null)

    Normaliza un array. Si ``$assoc`` es ``true``, el array resultante será
    normalizado para ser un array asociativo. Las claves numéricas con valores
    serán convertidas a claves de cadena con valores ``$default``. Normalizar un array
    hace que usar los resultados con :php:meth:`Hash::merge()` sea más fácil::

        $a = ['Árbol', 'ContadorCache',
            'Subir' => [
                'carpeta' => 'productos',
                'campos' => ['id_imagen_1', 'id_imagen_2']
            ]
        ];
        $result = Hash::normalize($a);
        /* $result ahora luce así:
            [
                [Árbol] => null
                [ContadorCache] => null
                [Subir] => [
                        [carpeta] => productos
                        [campos] => [
                                [0] => id_imagen_1
                                [1] => id_imagen_2
                        ]
                ]
            ]
        */

        $b = [
            'Cacheable' => ['activado' => false],
            'Límite',
            'Enlazable',
            'Validador',
            'Transaccional',
        ];
        $result = Hash::normalize($b);
        /* $result ahora luce así:
            [
                [Cacheable] => [
                        [activado] => false
                ]

                [Límite] => null
                [Enlazable] => null
                [Validador] => null
                [Transaccional] => null
            ]
        */

.. versionchanged:: 4.5.0
    Se agregó el parámetro ``$default``.

.. php:staticmethod:: nest(array $data, array $options = [])

    Toma un conjunto de datos plano y crea una estructura de datos anidada o enroscada.

    **Opciones:**

    - ``children`` El nombre de clave que se usará en el conjunto de resultados para los hijos. Por defecto,
      es 'children'.
    - ``idPath`` La ruta a una clave que identifica cada entrada. Debe ser compatible con :php:meth:`Hash::extract()`.
      Por defecto, es ``{n}.$alias.id``.
    - ``parentPath`` La ruta a una clave que identifica el padre de cada entrada. Debe ser compatible con
      :php:meth:`Hash::extract()`. Por defecto, es ``{n}.$alias.parent_id``.
    - ``root`` El id del resultado superior deseado.

    Por ejemplo, si tuvieras el siguiente array de datos::

        $data = [
            ['ThreadPost' => ['id' => 1, 'parent_id' => null]],
            ['ThreadPost' => ['id' => 2, 'parent_id' => 1]],
            ['ThreadPost' => ['id' => 3, 'parent_id' => 1]],
            ['ThreadPost' => ['id' => 4, 'parent_id' => 1]],
            ['ThreadPost' => ['id' => 5, 'parent_id' => 1]],
            ['ThreadPost' => ['id' => 6, 'parent_id' => null]],
            ['ThreadPost' => ['id' => 7, 'parent_id' => 6]],
            ['ThreadPost' => ['id' => 8, 'parent_id' => 6]],
            ['ThreadPost' => ['id' => 9, 'parent_id' => 6]],
            ['ThreadPost' => ['id' => 10, 'parent_id' => 6]]
        ];

        $result = Hash::nest($data, ['root' => 6]);
        /* $result ahora luce así:
            [
                (int) 0 => [
                    'ThreadPost' => [
                        'id' => (int) 6,
                        'parent_id' => null
                    ],
                    'children' => [
                        (int) 0 => [
                            'ThreadPost' => [
                                'id' => (int) 7,
                                'parent_id' => (int) 6
                            ],
                            'children' => []
                        ],
                        (int) 1 => [
                            'ThreadPost' => [
                                'id' => (int) 8,
                                'parent_id' => (int) 6
                            ],
                            'children' => []
                        ],
                        (int) 2 => [
                            'ThreadPost' => [
                                'id' => (int) 9,
                                'parent_id' => (int) 6
                            ],
                            'children' => []
                        ],
                        (int) 3 => [
                            'ThreadPost' => [
                                'id' => (int) 10,
                                'parent_id' => (int) 6
                            ],
                            'children' => []
                        ]
                    ]
                ]
            ]
            */

.. meta::
    :title lang=es: Hash
    :keywords lang=es: arreglo, ruta de arreglo, nombre de arreglo, clave numérica, expresión regular, conjunto de resultados, nombre de persona, corchetes, sintaxis, CakePHP, elementos, PHP, establecer ruta.
