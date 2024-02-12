Text
####

.. php:namespace:: Cake\Utility

.. php:class:: Text

La clase Text incluye métodos de conveniencia para crear y manipular cadenas y normalmente se accede estáticamente. Por ejemplo: ``Text::uuid()``.

Si necesitas funcionalidades de :php:class:`Cake\\View\\Helper\\TextHelper` fuera de una ``View``, utiliza la clase ``Text``::

    namespace App\Controller;

    use Cake\Utility\Text;

    class UsersController extends AppController
    {
        public function initialize(): void
        {
            parent::initialize();
            $this->loadComponent('Auth')
        };

        public function afterLogin()
        {
            $message = $this->Users->find('new_message')->first();
            if (!empty($message)) {
                // Notifica al usuario de un nuevo mensaje
                $this->Flash->success(__(
                    'Tienes un nuevo mensaje: {0}',
                    Text::truncate($message['Message']['body'], 255, ['html' => true])
                ));
            }
        }
    }

Convertir Cadenas en ASCII
==========================

.. php:staticmethod:: transliterate($string, $transliteratorId = null)

Transliterate convierte de forma predeterminada todos los caracteres en la cadena proporcionada en caracteres ASCII equivalentes.
El método espera codificación UTF-8. La conversión de caracteres puede ser controlada utilizando identificadores de transliteración
que puedes pasar utilizando el argumento ``$transliteratorId`` o cambiar la cadena de identificador predeterminada
utilizando ``Text::setTransliteratorId()``. Los identificadores de transliteración de ICU son básicamente
de la forma ``<script de origen>:<script de destino>`` y puedes especificar múltiples pares de conversión
separados por ``;``. Puedes encontrar más información sobre los identificadores de
transliteración `aquí <https://unicode-org.github.io/icu/userguide/transforms/general/#transliterator-identifiers>`_::

    // puré de manzana
    Text::transliterate('apple purée');

    // Übermensch (solo los caracteres latinos se transliteran)
    Text::transliterate('Übérmensch', 'Latin-ASCII;');

Crear Cadenas Seguras para URL
==============================

.. php:staticmethod:: slug($string, $options = [])

La función slug translitera todos los caracteres a versiones ASCII y convierte los caracteres no coincidentes y espacios en guiones. La función slug espera una codificación UTF-8.

Puedes proporcionar un array de opciones que controla la función slug. ``$options`` también puede ser una cadena, en cuyo caso se utilizará como cadena de reemplazo. Las opciones admitidas son:

* ``replacement``: Cadena de reemplazo, por defecto es '-'.
* ``transliteratorId``: Una cadena de ID de transliterador válida. Si es ``null`` por defecto, se usará ``Text::$_defaultTransliteratorId``.
  Si es ``false``, no se realizará transliteración, solo se eliminarán los caracteres no válidos.
* ``preserve``: Carácter no válido específico para preservar. Por defecto es ``null``.
  Por ejemplo, esta opción se puede establecer en '.' para generar nombres de archivo limpios::

    // purée-de-pomme
    Text::slug('purée de pomme');

    // purée_de_pomme
    Text::slug('purée de pomme', '_');

    // foo-bar.tar.gz
    Text::slug('foo bar.tar.gz', ['preserve' => '.']);

Generación de UUID
==================

.. php:staticmethod:: uuid()

El método UUID se utiliza para generar identificadores únicos según :rfc:`4122`. El UUID es una cadena de 128 bits en el formato de ``485fc381-e790-47a3-9794-1337c0a8fe68``. ::

    Text::uuid(); // 485fc381-e790-47a3-9794-1337c0a8fe68

Análisis Simple de Cadenas
==========================

.. php:staticmethod:: tokenize($data, $separator = ',', $leftBound = '(', $rightBound = ')')

Tokeniza una cadena utilizando ``$separator``, ignorando cualquier instancia de ``$separator`` que aparezca entre ``$leftBound`` y ``$rightBound``.

Este método puede ser útil al dividir datos que tienen un formato regular, como listas de etiquetas::

    $data = "cakephp 'gran framework' php";
    $result = Text::tokenize($data, ' ', "'", "'");

    // El resultado contiene
    ['cakephp', "'gran framework'", 'php'];

.. php:method:: parseFileSize(string $size, $default)

Este método desformatea un número de un tamaño de bytes legible por humanos a un número entero de bytes::

    $int = Text::parseFileSize('2GB');

Formateo de Cadenas
===================

.. php:staticmethod:: insert($string, $data, $options = [])

El método insert se utiliza para crear plantillas de cadenas y permitir reemplazos de clave/valor::

    Text::insert(
        'Mi nombre es :name y tengo :age años.',
        ['name' => 'Bob', 'age' => '65']
    );
    // Devuelve: "Mi nombre es Bob y tengo 65 años."

.. php:staticmethod:: cleanInsert($string, $options = [])

Limpia una cadena formateada con ``Text::insert`` con los ``$options`` dados dependiendo de la clave 'clean' en ``$options``. El método predeterminado usado es texto pero también está disponible html. El objetivo de esta función es reemplazar todos los espacios en blanco y marcas no necesarias alrededor de marcadores de posición que no fueron reemplazados por ``Text::insert``.

Puedes usar las siguientes opciones en el array de opciones::

    $options = [
        'clean' => [
            'method' => 'text', // o html
        ],
        'before' => '',
        'after' => ''
    ];

Envolver Texto
==============

.. php:staticmethod:: wrap($text, $options = [])

Envuelve un bloque de texto a un ancho establecido e indenta los bloques también.
Puede envolver inteligentemente el texto para que las palabras no se corten entre líneas::

    $text = 'Este es el himno que nunca termina.';
    $result = Text::wrap($text, 22);

    // Devuelve
    Este es el himno que
    nunca termina.

Puedes proporcionar un array de opciones que controlan cómo se realiza el envoltorio. Las opciones admitidas son:

* ``width`` El ancho al que envolver. Por defecto es 72.
* ``wordWrap`` Si envolver o no palabras completas. Por defecto es ``true``.
* ``indent`` El carácter para indentar las líneas con. Por defecto es ''.
* ``indentAt`` El número de línea para empezar a indentar el texto. Por defecto es 0.

.. php:staticmethod:: wrapBlock($text, $options = [])

Si necesitas asegurarte de que el ancho total del bloque generado no exceda cierta longitud incluso con sangrías internas, debes usar ``wrapBlock()`` en lugar de ``wrap()``. Esto es especialmente útil para generar texto para la consola, por ejemplo. Acepta las mismas opciones que ``wrap()``::

    $text = 'Este es el himno que nunca termina. Este es el himno que nunca termina.';
    $result = Text::wrapBlock($text, [
        'width' => 22,
        'indent' => ' → ',
        'indentAt' => 1
    ]);

    // Devuelve
    Este es el himno que
     → nunca termina. Este
     → es el himno que nunca
     → termina.

Destacando Subcadenas
=======================

.. php:method:: highlight(string $haystack, string $needle, array $options = [] )

Destaca ``$needle`` en ``$haystack`` usando la cadena ``$options['format']`` especificada o una cadena predeterminada.

Opciones:

-  ``format`` string - El fragmento de HTML con la frase que se resaltará.
-  ``html`` bool - Si es ``true``, ignorará cualquier etiqueta HTML, asegurando que solo se resalte el texto correcto.

Ejemplo::

    // Llamado como TextHelper
    echo $this->Text->highlight(
        $lastSentence,
        'usando',
        ['format' => '<span class="highlight">\1</span>']
    );

    // Llamado como Text
    use Cake\Utility\Text;

    echo Text::highlight(
        $lastSentence,
        'usando',
        ['format' => '<span class="highlight">\1</span>']
    );

Salida:

.. code-block: html

    Resalta $needle en $haystack <span class="highlight">usando</span> la cadena $options['format'] especificada o una cadena predeterminada.


Eliminación de Enlaces
======================

.. php:method:: stripLinks($text)

Elimina cualquier enlace HTML del texto suministrado ``$text``.

Truncado de Texto
=================

.. php:method:: truncate(string $text, int $length = 100, array $options)

Si ``$text`` es más largo que ``$length``, este método lo trunca en ``$length``
y agrega un sufijo que consiste en ``'ellipsis'``, si está definido. Si se pasa
``'exact'`` como ``false``, la truncación ocurrirá en el primer espacio en blanco
después del punto en el que se excede ``$length``. Si se pasa ``'html'`` como
``true``, se respetarán las etiquetas HTML y no se cortarán.

``$options`` se utiliza para pasar todos los parámetros adicionales, y tiene las
siguientes claves posibles de forma predeterminada, todas las cuales son opcionales::

    [
        'ellipsis' => '...',
        'exact' => true,
        'html' => false
    ]

Ejemplo::

    // Llamado como TextHelper
    echo $this->Text->truncate(
        'El asesino se acercó sigilosamente y tropezó con la alfombra.',
        22,
        [
            'ellipsis' => '...',
            'exact' => false
        ]
    );

    // Llamado como Text
    use Cake\Utility\Text;

    echo Text::truncate(
        'El asesino se acercó sigilosamente y tropezó con la alfombra.',
        22,
        [
            'ellipsis' => '...',
            'exact' => false
        ]
    );

Salida::

    El asesino se acercó...

Truncado de la Cola de una Cadena
==================================

.. php:method:: tail(string $text, int $length = 100, array $options)

Si ``$text`` es más largo que ``$length``, este método elimina una subcadena inicial
con longitud que consiste en la diferencia y agrega un prefijo que consiste en
``'ellipsis'``, si está definido. Si se pasa ``'exact'`` como ``false``, la truncación
ocurrirá en el primer espacio en blanco antes del punto en el que de lo contrario se
realizaría la truncación.

``$options`` se utiliza para pasar todos los parámetros adicionales, y tiene las
siguientes claves posibles de forma predeterminada, todas las cuales son opcionales::

    [
        'ellipsis' => '...',
        'exact' => true
    ]

Ejemplo::

    $sampleText = 'Empaqué mi bolsa y en ella puse una PSP, un PS3, una TV, ' .
        'un programa en C# que puede dividir por cero, camisetas de death metal'

    // Llamado como TextHelper
    echo $this->Text->tail(
        $sampleText,
        70,
        [
            'ellipsis' => '...',
            'exact' => false
        ]
    );

    // Llamado como Text
    use Cake\Utility\Text;

    echo Text::tail(
        $sampleText,
        70,
        [
            'ellipsis' => '...',
            'exact' => false
        ]
    );

Salida::

    ...una TV, un programa en C# que puede dividir por cero, camisetas de death metal


Extrayendo un Extracto
======================

.. php:method:: excerpt(string $haystack, string $needle, integer $radius=100, string $ellipsis="...")

Extrae un extracto de ``$haystack`` que rodea al ``$needle`` con un número
de caracteres a cada lado determinado por ``$radius``, y se agrega un prefijo/sufijo
con ``$ellipsis``. Este método es especialmente útil para los resultados de búsqueda.
La cadena de consulta o las palabras clave pueden mostrarse dentro del documento resultante. ::

    // Llamado como TextHelper
    echo $this->Text->excerpt($lastParagraph, 'método', 50, '...');

    // Llamado como Text
    use Cake\Utility\Text;

    echo Text::excerpt($lastParagraph, 'método', 50, '...');

Salida::

    ...por $radius, y se agrega un prefijo/sufijo con $ellipsis. Este método es especialmente
    útil para los resultados de búsqueda. La consulta...

Convirtiendo un arreglo en Forma de Oración
===========================================

.. php:method:: toList(array $list, $and='y', $separator=', ')

Crea una lista separada por comas donde los dos últimos elementos se unen con 'y'::

    $colores = ['rojo', 'naranja', 'amarillo', 'verde', 'azul', 'añil', 'violeta'];

    // Llamado como TextHelper
    echo $this->Text->toList($colores);

    // Llamado como Text
    use Cake\Utility\Text;

    echo Text::toList($colores);

Salida::

    rojo, naranja, amarillo, verde, azul, añil y violeta

.. meta::
    :title lang=es: Text
    :keywords lang=es: arreglo de PHP, nombre del arreglo, opciones de cadena, opciones de datos, cadena de resultado, cadena de clase, datos de cadena, clase de cadena, marcadores de posición, método predeterminado, clave valor, marcado, RFC, reemplazos, conveniencia, plantillas.
