Http Client
###########

.. php:namespace:: Cake\Http

.. php:class:: Client(mixed $config = [])

CakePHP incluye un cliente HTTP compatible con PSR-18 que se puede utilizar para realizar solicitudes. Es una excelente manera de comunicarse con servicios web y APIs remotas.

Realizar Solicitudes
====================

Realizar solicitudes es simple y directo. Hacer una solicitud GET se ve así::

    use Cake\Http\Client;

    $http = new Client();

    // Simple GET
    $response = $http->get('http://example.com/prueba.html');

    // GET simple con cadena de consulta
    $response = $http->get('http://example.com/buscar', ['q' => 'widget']);

    // GET simple con cadena de consulta y encabezados adicionales
    $response = $http->get('http://example.com/buscar', ['q' => 'widget'], [
      'headers' => ['X-Requested-With' => 'XMLHttpRequest'],
    ]);

Realizar solicitudes POST y PUT es igualmente simple::

    // Enviar una solicitud POST con datos codificados como application/x-www-form-urlencoded
    $http = new Client();
    $response = $http->post('http://example.com/posts/agregar', [
      'title' => 'prueba',
      'body' => 'contenido en la publicación',
    ]);

    // Enviar una solicitud PUT con datos codificados como application/x-www-form-urlencoded
    $response = $http->put('http://example.com/posts/agregar', [
      'title' => 'prueba',
      'body' => 'contenido en la publicación',
    ]);

    // Otros métodos también.
    $http->delete(...);
    $http->head(...);
    $http->patch(...);

Si has creado un objeto de solicitud PSR-7, puedes enviarlo usando ``sendRequest()``::

    use Cake\Http\Client;
    use Cake\Http\Client\Request as ClientRequest;

    $solicitud = new ClientRequest(
        'http://example.com/buscar',
        ClientRequest::METHOD_GET
    );
    $cliente = new Client();
    $respuesta = $cliente->sendRequest($solicitud);

Creación de Solicitudes Multipartes con Archivos
================================================

Puedes incluir archivos en los cuerpos de las solicitudes incluyendo un identificador de archivo en el arreglo::

    $http = new Client();
    $respuesta = $http->post('http://example.com/api', [
      'imagen' => fopen('/ruta/a/un/archivo', 'r'),
    ]);

El identificador de archivo se leerá hasta su fin; no se rebobinará antes de leerlo.

Construcción de Cuerpos de Solicitudes Multipartes
--------------------------------------------------

Puede haber ocasiones en las que necesites construir un cuerpo de solicitud de una manera muy específica. En estas situaciones, a menudo puedes usar ``Cake\Http\Client\FormData`` para crear la solicitud HTTP multipartes específica que deseas::

    use Cake\Http\Client\FormData;

    $datos = new FormData();

    // Crear una parte XML
    $xml = $datos->newPart('xml', $cadenaXML);
    // Establecer el tipo de contenido.
    $xml->type('application/xml');
    $datos->add($xml);

    // Crear una carga de archivo con addFile()
    // Esto añadirá el archivo también a los datos del formulario.
    $archivo = $datos->addFile('carga', fopen('/algún/archivo.txt', 'r'));
    $archivo->contentId('abc123');
    $archivo->disposition('adjunto');

    // Enviar la solicitud.
    $respuesta = $http->post(
        'http://example.com/api',
        (string)$datos,
        ['headers' => ['Content-Type' => $datos->contentType()]]
    );

Envío de Cuerpos de Solicitudes
===============================

Cuando trabajas con APIs REST, a menudo necesitas enviar cuerpos de solicitud que no están codificados como formularios. Http\\Cliente expone esto a través de la opción tipo::

    // Enviar un cuerpo de solicitud JSON.
    $http = new Client();
    $respuesta = $http->post(
      'http://example.com/tareas',
      json_encode($datos),
      ['type' => 'json']
    );

La clave ``type`` puede ser uno de 'json', 'xml' o un tipo mime completo. Cuando uses la opción ``type``, debes proporcionar los datos como una cadena. Si estás haciendo una solicitud GET que necesita tanto parámetros de cadena de consulta como un cuerpo de solicitud, puedes hacer lo siguiente::

    // Enviar un cuerpo JSON en una solicitud GET con parámetros de cadena de consulta.
    $http = new Client();
    $respuesta = $http->get(
      'http://example.com/tareas',
      ['q' => 'prueba', '_content' => json_encode($datos)],
      ['type' => 'json']
    );

.. _http_client_request_options:

Opciones del Método de Solicitud
================================

Cada método HTTP toma un parámetro ``$options`` que se utiliza para proporcionar información adicional de la solicitud. Las siguientes claves se pueden usar en ``$options``:

- ``headers`` - Matriz de encabezados adicionales
- ``cookie`` - Matriz de cookies a usar.
- ``proxy`` - Matriz de información de proxy.
- ``auth`` - Matriz de datos de autenticación, la clave ``type`` se usa para delegar a una estrategia de autenticación. Por defecto se utiliza la autenticación Básica.
- ``ssl_verify_peer`` - por defecto a ``true``. Establecer a ``false`` para deshabilitar la verificación de certificación SSL (no recomendado).
- ``ssl_verify_peer_name`` - por defecto a ``true``. Establecer a ``false`` para deshabilitar la verificación del nombre de host al verificar los certificados SSL (no recomendado).
- ``ssl_verify_depth`` - por defecto a 5. Profundidad para recorrer en la cadena de CA.
- ``ssl_verify_host`` - por defecto a ``true``. Validar el certificado SSL contra el nombre de host.
- ``ssl_cafile`` - por defecto al archivo cafile incorporado. Sobrescribir para usar conjuntos de CA personalizados.
- ``timeout`` - Duración a esperar antes de agotar el tiempo en segundos.
- ``type`` - Enviar un cuerpo de solicitud en un tipo de contenido personalizado. Requiere que ``$datos`` sea una cadena, o que se establezca la opción ``_content`` al hacer solicitudes GET.
- ``redirect`` - Número de redirecciones a seguir. Por defecto a ``false``.
- ``curl`` - Una matriz de opciones curl adicionales (si se utiliza el adaptador curl), por ejemplo, ``[CURLOPT_SSLKEY => 'clave.pem']``.

El parámetro de opciones es siempre el 3er parámetro en cada uno de los métodos HTTP. También se pueden usar al construir ``Client`` para crear :ref

:`clientes con alcance <http_client_scoped_client>`.

Autenticación
=============

``Cake\Http\Client`` admite varios sistemas de autenticación diferentes. Los desarrolladores pueden agregar diferentes estrategias de autenticación. Las estrategias de autenticación se llaman antes de enviar la solicitud y permiten agregar encabezados al contexto de la solicitud.

Usando Autenticación Básica
---------------------------

Un ejemplo de autenticación básica::

    $http = new Client();
    $respuesta = $http->get('http://example.com/perfil/1', [], [
      'auth' => ['username' => 'mark', 'password' => 'secreto'],
    ]);

Por defecto, ``Cake\Http\Client`` utilizará la autenticación básica si no hay una clave ``'type'`` en la opción de autenticación.

Usando Autenticación Digest
---------------------------

Un ejemplo de autenticación digest::

    $http = new Client();
    $respuesta = $http->get('http://example.com/perfil/1', [], [
        'auth' => [
            'type' => 'digest',
            'username' => 'mark',
            'password' => 'secreto',
            'realm' => 'mi_realm',
            'nonce' => 'valor_único',
            'qop' => 1,
            'opaque' => 'algún_valor',
        ],
    ]);

Al establecer la clave 'type' en 'digest', indicas al subsistema de autenticación que utilice la autenticación digest. La autenticación digest admite los siguientes algoritmos:

* MD5
* SHA-256
* SHA-512-256
* MD5-sess
* SHA-256-sess
* SHA-512-256-sess

El algoritmo se elegirá automáticamente según el desafío del servidor.

Autenticación OAuth 1
---------------------

Muchos servicios web modernos requieren autenticación OAuth para acceder a sus APIs. La autenticación OAuth incluida asume que ya tienes tu clave de consumidor y tu secreto de consumidor::

    $http = new Client();
    $respuesta = $http->get('http://example.com/perfil/1', [], [
        'auth' => [
            'type' => 'oauth',
            'consumerKey' => 'clave_grande',
            'consumerSecret' => 'secreto',
            'token' => '...',
            'tokenSecret' => '...',
            'realm' => 'tickets',
        ],
    ]);

Autenticación OAuth 2
---------------------

Dado que OAuth2 es a menudo un solo encabezado, no hay un adaptador de autenticación especializado. En su lugar, puedes crear un cliente con el token de acceso::

    $http = new Client([
        'headers' => ['Authorization' => 'Bearer ' . $accessToken],
    ]);
    $respuesta = $http->get('https://example.com/api/perfil/1');

Autenticación de Proxy
----------------------

Algunos proxies requieren autenticación para usarlos. Generalmente esta autenticación es básica, pero puede ser implementada por cualquier adaptador de autenticación. Por defecto, Http\\Cliente asumirá la autenticación básica, a menos que se establezca la clave de tipo::

    $http = new Client();
    $respuesta = $http->get('http://example.com/prueba.php', [], [
        'proxy' => [
            'username' => 'mark',
            'password' => 'prueba',
            'proxy' => '127.0.0.1:8080',
        ],
    ]);

El segundo parámetro de proxy debe ser una cadena con una IP o un dominio sin protocolo. La información de nombre de usuario y contraseña se pasará a través de los encabezados de la solicitud, mientras que la cadena de proxy se pasará a través de `stream_context_create()
<https://php.net/manual/en/function.stream-context-create.php>`_.

Creación de Clientes Específicos
================================

Tener que volver a escribir el nombre de dominio, la autenticación y la configuración del proxy puede volverse tedioso y propenso a errores. Para reducir la posibilidad de errores y aliviar parte de la tediosidad, puedes crear clientes específicos::

    // Crea un cliente específico.
    $http = new Client([
        'host' => 'api.example.com',
        'scheme' => 'https',
        'auth' => ['username' => 'mark', 'password' => 'prueba'],
    ]);

    // Realiza una solicitud a api.example.com
    $respuesta = $http->get('/prueba.php');

Si tu cliente específico solo necesita información de la URL, puedes usar ``createFromUrl()``::

    $http = Client::createFromUrl('https://api.example.com/v1/prueba');

Lo anterior crearía una instancia de cliente con las opciones ``protocol``, ``host`` y
``basePath`` configuradas.

La siguiente información se puede utilizar al crear un cliente específico:

* host
* basePath
* scheme
* proxy
* auth
* port
* cookies
* timeout
* ssl_verify_peer
* ssl_verify_depth
* ssl_verify_host

Cualquiera de estas opciones puede ser anulada especificándolas al hacer solicitudes.
host, scheme, proxy, port se anulan en la URL de la solicitud::

    // Usando el cliente específico que creamos anteriormente.
    $respuesta = $http->get('http://foo.com/prueba.php');

Lo anterior reemplazará el dominio, el esquema y el puerto. Sin embargo, esta solicitud seguirá utilizando todas las otras opciones definidas cuando se creó el cliente específico.

Consulta :ref:`http_client_request_options` para obtener más información sobre las opciones soportadas.

Configuración y Gestión de Cookies
==================================

Http\\Cliente también puede aceptar cookies al realizar solicitudes. Además de aceptar cookies, también almacenará automáticamente las cookies válidas establecidas en las respuestas. Cualquier respuesta con cookies se almacenará en la instancia original de Http\\Cliente. Las cookies almacenadas en una instancia de Cliente se incluyen automáticamente en futuras solicitudes a combinaciones de dominio + ruta que coincidan::

    $http = new Client([
        'host' => 'cakephp.org'
    ]);

    // Realiza una solicitud que establece algunas cookies
    $respuesta = $http->get('/');

    // Las cookies de la primera solicitud se incluirán
    // por defecto.
    $respuesta2 = $http->get('/changelogs');

Siempre puedes anular las cookies incluidas automáticamente estableciéndolas en los parámetros ``$options`` de la solicitud::

    // Reemplazar una cookie almacenada con un valor personalizado.
    $respuesta = $http->get('/changelogs', [], [
        'cookies' => ['sessionid' => '123abc'],
    ]);

Puedes agregar objetos de cookie al cliente después de crearlo usando el método ``addCookie()``::

    use Cake\Http\Cookie\Cookie;

    $http = new Client([
        'host' => 'cakephp.org'
    ]);
    $http->addCookie(new Cookie('session', 'abc123'));

Objetos de Respuesta
====================

.. php:namespace:: Cake\Http\Client

.. php:class:: Respuesta

Los objetos de respuesta tienen varios métodos para inspeccionar los datos de respuesta.

Lectura de Cuerpos de Respuesta
-------------------------------

Puedes leer todo el cuerpo de respuesta como una cadena::

    // Lee toda la respuesta como una cadena.
    $respuesta->getStringBody();

También puedes acceder al objeto de transmisión para la respuesta y usar sus métodos::

    // Obtiene un Psr\Http\Message\StreamInterface que contiene el cuerpo de respuesta
    $transmision = $respuesta->getBody();

    // Lee una transmisión de 100 bytes a la vez.
    while (!$transmision->eof()) {
        echo $transmision->read(100);
    }

Lectura de Cuerpos de Respuesta JSON y XML
------------------------------------------

Dado que las respuestas JSON y XML se utilizan comúnmente, los objetos de respuesta proporcionan una forma de usar accesos para leer datos decodificados. Los datos JSON se decodifican en un array, mientras que los datos XML se decodifican en un árbol ``SimpleXMLElement``::

    // Obtén algo de XML
    $http = new Client();
    $respuesta = $http->get('http://example.com/test.xml');
    $xml = $respuesta->getXml();

    // Obtén algo de JSON
    $http = new Client();
    $respuesta = $http->get('http://example.com/test.json');
    $json = $respuesta->getJson();

Los datos de respuesta decodificados se almacenan en el objeto de respuesta, por lo que acceder a ellos varias veces no tiene un costo adicional.

Acceso a Encabezados de Respuesta
---------------------------------

Puedes acceder a los encabezados a través de algunos métodos diferentes. Los nombres de los encabezados siempre se tratan como valores insensibles a mayúsculas y minúsculas cuando se accede a ellos a través de métodos::

    // Obtén todos los encabezados como un array asociativo.
    $respuesta->getHeaders();

    // Obtén un solo encabezado como un array.
    $respuesta->getHeader('content-type');

    // Obtén un encabezado como una cadena
    $respuesta->getHeaderLine('content-type');

    // Obtén la codificación de la respuesta
    $respuesta->getEncoding();

Acceso a Datos de Cookies
-------------------------

Puedes leer cookies con algunos métodos diferentes dependiendo de cuántos datos necesites sobre las cookies::

    // Obtén todas las cookies (datos completos)
    $respuesta->getCookies();

    // Obtén el valor de una sola cookie.
    $respuesta->getCookie('session_id');

    // Obtén los datos completos de una sola cookie
    // incluye claves de valor, caducidad, ruta, httponly, seguras.
    $respuesta->getCookieData('session_id');

Comprobación del Código de Estado
---------------------------------

Los objetos de respuesta proporcionan algunos métodos para verificar los códigos de estado::

    // ¿Fue la respuesta un 20x
    $respuesta->isOk();

    // ¿Fue la respuesta un 30x
    $respuesta->isRedirect();

    // Obtén el código de estado
    $respuesta->getStatusCode();

Cambio de Adaptadores de Transporte
===================================

Por defecto, ``Http\Client`` preferirá usar un adaptador de transporte basado en ``curl``. Si la extensión de curl no está disponible, se usará un adaptador basado en transmisión en su lugar. Puedes seleccionar un adaptador de transporte específico usando una opción de constructor::

    use Cake\Http\Client\Adapter\Stream;

    $cliente = new Client(['adaptador' => Stream::class]);

.. _httpclient-testing:

Pruebas
=======

.. php:namespace:: Cake\Http\TestSuite

.. php:trait:: HttpClientTrait

En las pruebas, a menudo querrás crear respuestas simuladas para APIs externas. Puedes
usar el ``HttpClientTrait`` para definir respuestas a las solicitudes que tu aplicación
está realizando::

    use Cake\Http\TestSuite\HttpClientTrait;
    use Cake\TestSuite\TestCase;

    class PruebasDeControladorDeCarrito extends TestCase
    {
        use HttpClientTrait;

        public function testCheckout()
        {
            // Simula una solicitud POST que se realizará.
            $this->mockClientPost(
                'https://example.com/process-payment',
                $this->newClientResponse(200, [], json_encode(['ok' => true]))
            );
            $this->post("/cart/checkout");
            // Realiza aserciones.
        }
    }

Hay métodos para simular los métodos HTTP más comúnmente utilizados::

    $this->mockClientGet(...);
    $this->mockClientPatch(...);
    $this->mockClientPost(...);
    $this->mockClientPut(...);
    $this->mockClientDelete(...);

.. php:method:: newClientResponse(int $code = 200, array $headers = [], string $body = '')

Como se vio anteriormente, puedes usar el método ``newClientResponse()`` para crear respuestas
para las solicitudes que tu aplicación realizará. Los encabezados deben ser una lista de
cadenas::

    $headers = [
        'Content-Type: application/json',
        'Connection: close',
    ];
    $respuesta = $this->newClientResponse(200, $headers, $body)

.. meta::
    :title lang=es: HttpClient
    :keywords lang=es: nombre del arreglo, datos del arreglo, parámetro de consulta, cadena de consulta, clase de PHP, consulta de cadena, tipo de prueba, datos de cadena, Google, resultados de la consulta, servicios web, API, parámetros, CakePHP, método, resultados de búsqueda.
