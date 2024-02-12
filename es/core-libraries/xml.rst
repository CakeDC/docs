XML
###

.. php:namespace:: Cake\Utility

.. php:class:: Xml

La clase Xml te permite transformar arrays en objetos SimpleXMLElement o
DOMDocument, y viceversa.

Cargando documentos XML
=======================

.. php:staticmethod:: build($input, array $options = [])

Puedes cargar datos parecidos a XML usando ``Xml::build()``. Dependiendo de tu
parámetro ``$options``, este método devolverá un objeto SimpleXMLElement (predeterminado)
o DOMDocument. Puedes usar ``Xml::build()`` para construir objetos XML
a partir de diversas fuentes. Por ejemplo, puedes cargar XML desde
cadenas::

    $text = '<?xml version="1.0" encoding="utf-8"?>
    <post>
        <id>1</id>
        <title>Best post</title>
        <body> ... </body>
    </post>';
    $xml = Xml::build($text);

También puedes construir objetos Xml desde archivos locales sobrescribiendo la opción predeterminada::

    // Archivo local
    $xml = Xml::build('/home/awesome/unicorns.xml', ['readFile' => true]);

También puedes construir objetos Xml usando un array::

    $data = [
        'post' => [
            'id' => 1,
            'title' => 'Best post',
            'body' => ' ... ',
        ]
    ];
    $xml = Xml::build($data);

Si tu entrada no es válida, la clase Xml lanzará una excepción::

    $xmlString = '¿Qué es XML?';
    try {
        $xmlObject = Xml::build($xmlString); // Aquí lanzará una excepción
    } catch (\Cake\Utility\Exception\XmlException $e) {
        throw new InternalErrorException();
    }

.. nota::

    `DOMDocument <https://php.net/domdocument>`_ y
    `SimpleXML <https://php.net/simplexml>`_ implementan APIs diferentes.
    Asegúrate de usar los métodos correctos en el objeto que solicitas de Xml.

Cargando documentos HTML
========================

Los documentos HTML se pueden analizar en objetos ``SimpleXmlElement`` o ``DOMDocument``
con ``loadHtml()``::

    $html = Xml::loadHtml($htmlString, ['return' => 'domdocument']);

Por defecto, la carga de entidades y el análisis de documentos enormes están deshabilitados. Estos modos
se pueden habilitar con las opciones ``loadEntities`` y ``parseHuge``, respectivamente.

Transformando una cadena XML en un arreglo
==========================================

.. php:staticmethod:: toArray($obj);

Convertir cadenas XML en arrays también es sencillo con la clase Xml. Por
defecto, obtendrás un objeto SimpleXml::

    $xmlString = '<?xml version="1.0"?><root><child>value</child></root>';
    $xmlArray = Xml::toArray(Xml::build($xmlString));

Si tu XML no es válido, se generará una excepción ``Cake\Utility\Exception\XmlException``.

Transformando un arreglo en una Cadena XML
==========================================

::

    $xmlArray = ['root' => ['child' => 'value']];
    // También puedes usar Xml::build().
    $xmlObject = Xml::fromArray($xmlArray, ['format' => 'tags']);
    $xmlString = $xmlObject->asXML();

Tu array debe tener solo un elemento en el "nivel superior" y no puede ser
numérico. Si el array no tiene este formato, Xml lanzará una excepción.
Ejemplos de arrays no válidos::

    // Nivel superior con clave numérica
    [
        ['key' => 'value']
    ];

    // Múltiples claves en el nivel superior
    [
        'key1' => 'primer valor',
        'key2' => 'otro valor'
    ];

Por defecto, los valores de array se mostrarán como etiquetas XML. Si deseas definir
atributos o valores de texto, puedes agregar el prefijo ``@`` a las claves que se supone
que serán atributos. Para el texto del valor, usa ``@`` como clave::

    $xmlArray = [
        'project' => [
            '@id' => 1,
            'name' => 'Nombre del proyecto, como etiqueta',
            '@' => 'Valor del proyecto',
        ],
    ];
    $xmlObject = Xml::fromArray($xmlArray);
    $xmlString = $xmlObject->asXML();

El contenido de ``$xmlString`` será::

    <?xml version="1.0"?>
    <project id="1">Valor del proyecto<name>Nombre del proyecto, como etiqueta</name></project>

Usando Espacios de Nombres
--------------------------

Para usar Espacios de Nombres XML, crea una clave en tu array con el nombre ``xmlns:``
en un espacio de nombres genérico o ingresa el prefijo ``xmlns:`` en un espacio de nombres personalizado. Consulta
los ejemplos::

    $xmlArray = [
        'root' => [
            'xmlns:' => 'https://cakephp.org',
            'child' => 'value',
        ]
    ];
    $xml1 = Xml::fromArray($xmlArray);

    $xmlArray(
        'root' => [
            'tag' => [
                'xmlns:pref' => 'https://cakephp.org',
                'pref:item' => [
                    'item 1',
                    'item 2'
                ]
            ]
        ]
    );
    $xml2 = Xml::fromArray($xmlArray);

El valor de ``$xml1`` y ``$xml2`` será, respectivamente::

    <?xml version="1.0"?>
    <root xmlns="https://cakephp.org"><child>value</child>

    <?xml version="1.0"?>
    <root><tag xmlns:pref="https://cakephp.org"><pref:item>item 1</pref:item><pref:item>item 2</pref:item></tag></root>

Creando un Hijo
---------------

Después de haber creado tu documento XML, simplemente usa las interfaces nativas para
tu tipo de documento para agregar, eliminar o manipular nodos hijos::

    // Usando SimpleXML
    $myXmlOriginal = '<?xml version="1.0"?><root><child>value</child></root>';
    $xml = Xml::build($myXmlOriginal);
    $xml->root->addChild('young', 'nuevo valor');

    // Usando DOMDocument
    $myXmlOriginal = '<?xml version="1.0"?><root><child>value</child></root>';
    $xml = Xml::build($myXmlOriginal, ['return' => 'domdocument']);
    $child = $xml->createElement('young', 'nuevo valor');
    $xml->firstChild->appendChild($child);


.. meta::
    :title lang=es: Xml
    :keywords lang=es: arreglo php, clase Xml, objetos Xml, XML de publicación, objeto Xml, cadena de URL, cadena de datos, analizador XML, PHP 5, constructor, PHP XML, CakePHP, archivo PHP, método
