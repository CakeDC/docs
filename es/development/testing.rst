Pruebas
#######

CakePHP viene con soporte integral de pruebas incorporado. CakePHP viene con
integración para `PHPUnit <https://phpunit.de>`_. Además de las funciones que
ofrece PHPUnit, CakePHP ofrece algunas funciones adicionales para facilitar
las pruebas. Esta sección cubrirá la instalación de PHPUnit, cómo empezar con
las pruebas unitarias y cómo puedes utilizar las extensiones que ofrece CakePHP.

Instalando PHPUnit
==================

CakePHP utiliza PHPUnit como marco de pruebas subyacente. PHPUnit es el estándar
de facto para pruebas unitarias en PHP. Ofrece un conjunto profundo y potente de
funciones para garantizar que tu código hace lo que crees que hace. PHPUnit se
puede instalar utilizando un `paquete PHAR <https://phpunit.de/#download>`__ o
`Composer <https://getcomposer.org>`_.

Instalar PHPUnit con Composer
-----------------------------

Para instalar PHPUnit con Composer:

.. code-block:: console

    $ php composer.phar require --dev phpunit/phpunit:"^10.1"

Esto añadirá la dependencia a la sección ``require-dev`` de tu
``composer.json`` y luego instalará PHPUnit junto con las dependencias.

Ahora puedes ejecutar PHPUnit utilizando:

.. code-block:: console

    $ vendor/bin/phpunit

Utilizando el Archivo PHAR
--------------------------

Después de haber descargado el archivo **phpunit.phar**, puedes utilizarlo
para ejecutar tus test:

.. code-block:: console

    php phpunit.phar

.. tip::

    Para tu comodidad, puedes hacer que phpunit.phar esté disponible
    globalmente en Unix o Linux con lo siguiente:

    .. code-block:: shell

          chmod +x phpunit.phar
          sudo mv phpunit.phar /usr/local/bin/phpunit
          phpunit --version

    Consulta la documentación sobre PHPUnit para obtener instrucciones sobre
    `Instalación global de PHPUnit PHAR en Windows <https://phpunit.de/manual/current/en/installation.html#installation.phar.windows>`__.

Configuración de Test Database
==============================

Recuerda tener habilitada la depuración en tu archivo **config/app_local.php** antes
de ejecutar cualquier prueba. Antes de ejecutar cualquier prueba, debes asegurarte de
agregar una configuración de fuente de datos ``test`` a tu **config/app_local.php**.
CakePHP utiliza esta configuración para fijar tablas y datos::

    'Datasources' => [
        'test' => [
            'datasource' => 'Cake\Database\Driver\Mysql',
            'persistent' => false,
            'host' => 'dbhost',
            'username' => 'dblogin',
            'password' => 'dbpassword',
            'database' => 'test_database',
        ],
    ],

.. note::

    Es una buena idea hacer que tu base de datos de prueba y la base de
    datos real sean bases de datos diferentes. Esto evitará errores
    embarazosos más adelante.

Comprobación de la Configuración de Pruebas
===========================================

Después de instalar PHPUnit y configurar tu fuente de datos de ``test``, puedes
asegurarte de que estás listo para escribir y ejecutar tus propias pruebas
ejecutando las pruebas de tu aplicación:

.. code-block:: console

    # For phpunit.phar
    $ php phpunit.phar

    # For Composer installed phpunit
    $ vendor/bin/phpunit

Lo anterior debería ejecutar cualquier prueba que tengas o informarle que no se
ejecutó ninguna prueba.
Para ejecutar una prueba específica, puedes proporcionar la ruta a la prueba como
parámetro de PHPUnit. Por ejemplo, si tuvieras un caso de prueba para la clase
ArticlesTable, podrías ejecutarlo con:

.. code-block:: console

    $ vendor/bin/phpunit tests/TestCase/Model/Table/ArticlesTableTest

Debería ver una barra verde con alguna información adicional sobre las pruebas
ejecutadas y el número aprobado.

.. note::

    Si utilizas un sistema Windows probablemente no verás ningún color.

Convenciones de Casos de Prueba
===============================

Como la mayoría de cosas en CakePHP, los casos de prueba tienen algunas
convenciones. En cuanto a las pruebas:

#. Los archivos PHP que contienen pruebas deben estar en sus directorios
   ``tests/TestCase/[Type]``.
#. Los nombres de estos archivos deben terminar en **Test.php** en lugar
   de sólo .php.
#. Las clases que contienen pruebas deben extender ``Cake\TestSuite\TestCase``,
   ``Cake\TestSuite\IntegrationTestCase`` o ``\PHPUnit\Framework\TestCase``.
#. Al igual que otros nombres de clases, los nombres de clase del caso de prueba
   deben coincidir con el nombre del archivo. **RouterTest.php** debe contener
   ``class RouterTest extends TestCase``.
#. El nombre de cualquier método que contenga una prueba, es decir, que contenga
   una afirmación, debe comenzar con ``test``, como en ``testPublishing()``.
   También puedes utilizar la anotación ``@test`` para marcar métodos como métodos
   de prueba.

Creando Tu Primer Caso de Prueba
================================

En el siguiente ejemplo, crearemos un caso de prueba para un método muy simple
de in helper. El helper que vamos a probar será el formateo de la barra de
progreso HTML. Nuestro helper se parece a::

    namespace App\View\Helper;

    use Cake\View\Helper;

    class ProgressHelper extends Helper
    {
        public function bar($value)
        {
            $width = round($value / 100, 2) * 100;
            return sprintf(
                '<div class="progress-container">
                    <div class="progress-bar" style="width: %s%%"></div>
                </div>', $width);
        }
    }

Este es un ejemplo muy simple pero será útil para mostrar cómo puedes crear un
caso de prueba simple. Después de crear y guardar nuestro helper, crearemos el
archivo del caso de prueba en **tests/TestCase/View/Helper/ProgressHelperTest.php**.
En ese archivo comenzaremos con lo siguiente::

    namespace App\Test\TestCase\View\Helper;

    use App\View\Helper\ProgressHelper;
    use Cake\TestSuite\TestCase;
    use Cake\View\View;

    class ProgressHelperTest extends TestCase
    {
        public function setUp(): void
        {
        }

        public function testBar(): void
        {
        }
    }

Daremos cuerpo a este esqueleto en un minuto. Hemos añadido dos métodos para
empezar. El primero es ``setUp()``. Este método se llama antes de cada método
*test* en una clase de caso de prueba. Los métodos de configuración deben
inicializar los objetos necesarios para la prueba y realizar cualquier
configuración necesaria. En nuestro método de configuración agregaremos lo
siguiente::

    public function setUp(): void
    {
        parent::setUp();
        $View = new View();
        $this->Progress = new ProgressHelper($View);
    }

Llamar al método principal es importante en casos de prueba, ya que
``TestCase::setUp()`` hace varias cosas como hacer una copia de seguridad
de los valores en :php:class:`~Cake\\Core\\Configure` y almacenar las rutas en
:php:class:`~Cake\\Core\\App`.

A continuación, completaremos el método de prueba. Usaremos algunas afirmaciones
para asegurarnos que nuestro código crea el resultado que esperamos::

    public function testBar(): void
    {
        $result = $this->Progress->bar(90);
        $this->assertStringContainsString('width: 90%', $result);
        $this->assertStringContainsString('progress-bar', $result);

        $result = $this->Progress->bar(33.3333333);
        $this->assertStringContainsString('width: 33%', $result);
    }

La prueba anterior es sencilla pero muestra el beneficio potencial de utilizar
casos de prueba. Utilizamos ``assertStringContainsString()`` para asegurarnos de
que nuestro helper devuelva una cadena que contenga el contenido que esperamos.
Si el resultado no tuviera el contenido esperado el test fallaría y sabríamos
que nuestro código es incorrecto.

Utilizando los casos de prueba puedes describir la relación entre un conjunto de
entradas conocidas y su salida esperada. Esto te ayudará a tener más confianza en
el código que estás escribiendo ya que puedes asegurar que el código escrito cumple
las expectativas y afirmaciones que hacen las pruebas. Además, debido a que las
pruebas son código, se pueden volver a ejecutar cada vez que realices un cambio.
Esto ayuda a prevenit la creación de nuevos errores.

.. note::

    EventManager se actualiza para cada método de prueba. Esto significa que cuando
    ejecutes varias pruebas a la vez, perderás los detectores de eventos que se
    registraron en config/bootstrap.php ya que el arranque solo se ejecuta una vez.

.. _running-tests:

Ejecución de Pruebas
====================

Una vez hayas instalado PHPUnit y escrito algunos casos de prueba, querrás ejecutar
los casos de prueba con mucha frecuencia. Es una buena idea ejecutar pruebas antes
de realizar cualquier cambio para asegurarte de que no se haya roto nada.

Utilizando ``phpunit`` puedes ejecutar las pruebas de tu aplicación. Para ejecutar
las pruebas de tu aplicación simplemente puedes ejecutar:

.. code-block:: console

    vendor/bin/phpunit

    php phpunit.phar

Si has clonado el código fuente de `CakePHP source from GitHub <https://github.com/cakephp/cakephp>`__
y deseas ejecutar las pruebas unitarias de CakePHP, no olvides ejecutar el siguiente comando
``Composer`` antes de ejecutar ``phpunit`` para que se instalen las dependencias:

.. code-block:: console

    composer install

Desde el directorio raíz de tu aplicación. Para ejecutar las pruebas para un plugin
que es parte del código de tu aplicación, primero ingresa ``cd`` en el directorio
del plugin y luego utiliza el comando ``phpunit`` que coincida con la forma en que
instalaste phpunit:

.. code-block:: console

    cd plugins

    ../vendor/bin/phpunit

    php ../phpunit.phar

Para ejecutar pruebas en un plugin independiente, primero debes instalar el
proyecto en un directorio deparado e instalar sus dependencias:

.. code-block:: console

    git clone git://github.com/cakephp/debug_kit.git
    cd debug_kit
    php ~/composer.phar install
    php ~/phpunit.phar

Filtrado de Casos de Prueba
---------------------------

Cuando tengas casos de prueba más grandes, a menudo querrás ejecutar un subconjunto
de métodos de prueba cuando intentes trabajar en un único caso fallido. Con el
corredor CLI puedes utilizar una opción para filtar métodos de prueba:

.. code-block:: console

    $ phpunit --filter testSave tests/TestCase/Model/Table/ArticlesTableTest

El parámetro de filtro se utiliza como expresión regular que distingue entre
mayúsculas y minúsculas para filtrar qué métodos de prueba ejecutar.

Generando Cobertura de Código
-----------------------------

Puedes generar informes de cobertura de código desde la línea de comandos utilizando
las herramientas de cobertura de código integradas de PHPUnit. PHPUnit generará
un conjunto de archivos HTML estáticos que contienen los resultados de la cobertura.
Puedes generar covertura para un caso de prueba haciendo lo siguiente:

.. code-block:: console

    $ phpunit --coverage-html webroot/coverage tests/TestCase/Model/Table/ArticlesTableTest

Esto colocará los resultados de la cobertura en el directorio raíz de tu aplicación.
Deberías poder ver los resultados yendo a ``http://localhost/your_app/coverage``.

También puedes utilizar ``phpdbg`` para generar cobertura en lugar de xdebug.
``phpdbgp`` es generalmente más rápido a la hora de generar cobertura:

.. code-block:: console

    $ phpdbg -qrr phpunit --coverage-html webroot/coverage tests/TestCase/Model/Table/ArticlesTableTest

Combinando Conjuntos de Pruebas para Plugins
--------------------------------------------

Muchas veces tu aplicación estará compuesta por varios plugins. En estas
situaciones, puede resultar bastante tedioso ejecutar pruebas para cada plugin.
Puedes realizar pruebas en ejecución para cada uno de los plugins que componen
tu aplicación agregando secciones ``<testsuite>`` adicionales al archivo
**phpunit.xml.dist** de tu aplicación:

.. code-block:: xml

    <testsuites>
        <testsuite name="app">
            <directory>./tests/TestCase/</directory>
        </testsuite>

        <!-- Agrega tus conjuntos de plugins -->
        <testsuite name="forum">
            <directory>./plugins/Forum/tests/TestCase/</directory>
        </testsuite>
    </testsuites>

Cualquier conjunto de pruebas adicional agregado al elemento ``<testsuites>``
se ejecutará automáticamente cuando utilices ``phpunit``.

Si estás utilizando ``<testsuites>`` para utilizar fixtures desde plugins que
has instalado con composer, el archivo ``composer.json`` del plugin debe agregar
el espacio de nombres del accesorio al la sección de carga automática. Ejemplo::

    "autoload-dev": {
        "psr-4": {
            "PluginName\\Test\\Fixture\\": "tests/Fixture/"
        }
    },

Devoluciones de Llamada del Ciclo de Vida del Caso de Prueba
============================================================

Los casos de prueba tienen varias devoluciones de llamadas del ciclo de vida que
puede utilizar al realizar pruebas:

* ``setUp`` se llama antes de cada método de prueba. Debe utilizarse para crear
  los objetos que se van a probar e inicializar los datos para la prueba.
  Recuerda siempre llamar a ``parent::setUp()``
* ``tearDown`` se llama después de cada método de prueba. Debe utilizarse para
  limpiar una vez finalizada la prueba. Recuerda siempre llamar a ``parent::tearDown()``.
* ``setupBeforeClass`` se llama una vez antes de que se inicien los métodos de prueba en
  un caso. Este método debe ser *estático*.
* ``tearDownAfterClass`` se llama una vez después de que se inician los métodos de prueba
  en un caso. Este método debe ser *estático*.

.. _test-fixtures:

Fixtures
========

Al probar código que depende de los modelos y la base de datos, se pueden utilizar
**fixtures** como una forma de crear un estado inicial para las pruebas de tu
aplicación. Al utilizar datos de accesorios, puedes reducir los pasos de configuración
repetitivos en tus pruebas. Los fixtures se adaptan bien a los datos que son comunes
o compartidos entre muchas o todas tus pruebas. Los datos que sólo se necesitan en un
subconjunto de pruebas se deben crear en las pruebas según sea necesario.

CakePHP utiliza la conexión llamada ``test`` en tu archivo de configuración
**config/app.php**. Si esta conexión no se puede utilizar, se geneará una
excepción y no podrás utilizar los fixtures de la base de datos.

CakePHP realiza lo siguiente durante el transcurso de una ejecución de prueba:

#. Crea tablas para cada uno de los fixtures necesarios.
#. Llena las tablas con datos.
#. Ejecuta los métodos de prueba.
#. Vacía las tablas de los accesorios.

El esquema para los fixtures se crea al comienzo de una ejecución de prueba
mediante migraciones o un archivo de volcado SQL.

Conexiones de Prueba
--------------------

De forma predeterminada, CakePHP asignará un alias a cada conexión en tu
aplicación. Cada conexión definida en el arranque de tu aplicación que no
comience con ``test_`` tendrá un alias con el prefijo ``test_`` creado.
Las conexiones de alias garantizan que no utilice accidentalmente la
conexión incorrecta en los casos de prueba.
El alias de conexión es transparente para el resto de tu aplicación. Por ejemplo,
si utilizas la conexión 'default' obtendrá la conexión ``test`` en los casos de
prueba. Si utilzas la conexión 'replica', el conjunto de pruebas intentará utilizar
'test_replica'.

.. _fixture-phpunit-configuration:

Configuración de PHPUnit
------------------------

Andes de poder utilizar fixtures, debes verificar que tu ``phpunit.xml``
contiene la extensión del fixture:

.. code-block:: xml

    <!-- en phpunit.xml -->
    <!-- Configurar la extensión para los fixtures -->
    <extensions>
        <extension class="\Cake\TestSuite\Fixture\PHPUnitExtension" />
    </extensions>

La extensión está incluida en tu aplicación y los plugins generados por ``bake``
de forma predeterminada.

.. _creating-test-database-schema:

Crear Esquema en Pruebas
------------------------

Puedes generar un esquema de la base de datos de test a través de las migraciones
de CakePHP, cargando un archivo de volcado de SQL o utilizando otra herramienta
de administración de esquemas externa. Debes crear tu esquema en el fichero

Creando Esquemas con Migraciones
--------------------------------

Si usas el :doc:`migrations plugin </migrations>` de CakePHP para administrar
el esquema de tu aplicación, también puedes reutilizar estas migraciones para generar
el esquema de tu base de datos de prueba::

    // en tests/bootstrap.php
    use Migrations\TestSuite\Migrator;

    $migrator = new Migrator();

    // Configuración sencilla sin plugins
    $migrator->run();

    // Ejecuta migraciones para múltiples complementos
    $migrator->run(['plugin' => 'Contacts']);

    // Ejecutar las migraciones de Documents en la conexión test_docs.
    $migrator->run(['plugin' => 'Documents', 'connection' => 'test_docs']);

Si necesitas ejecutar varios conjuntos de migraciones, puedes ejecutarlos
de la siguiente manera::

    $migrator->runMany([
        // Ejecuta migraciones de aplicaciones en una conexión de prueba.
        ['connection' => 'test'],
        // Ejecuta migraciones de Contacts en una conexión de prueba.
        ['plugin' => 'Contacts'],
        // Ejecuta migraciones de Documents en la conexión test_docs.
        ['plugin' => 'Documents', 'connection' => 'test_docs']
    ]);

El uso de ``runMany()`` garantizará que los plugins que comparten una base de
datos no eliminen tablas a medida que se ejecuta cada conjunto de migraciones.

El plugin de migraciones sólo ejecutará migraciones no aplicadas y las restablecerá
si su encabezado de migración actual difiere de las migraciones aplicadas.

También puedes configurar cómo se deben ejecutar las migraciones en las pruebas
en la configuración de tus fuentes de datos. Consulta :doc:`migrations docs </migrations>`
para obtener más información.

Crear Esquema con Esquema Abstracto
-----------------------------------

Para los plugins que necesitan definir un esquema en las pruebas, pero no necesitan
o quieren tener dependencias en las migraciones, puedes definir el esquema como
una matriz estructurada de tablas. Este formato no se recomienda para el desarrollo
de aplicaciones, ya que su mantenimiento puede llevar mucho tiempo.

Cada tabla puede definir ``columnas``, ``restricciones`` e ``índices``.
Una tabla de ejemplo sería::

     return [
       'articles' => [
          'columns' => [
              'id' => [
                  'type' => 'integer',
              ],
              'author_id' => [
                  'type' => 'integer',
                  'null' => true,
              ],
              'title' => [
                  'type' => 'string',
                  'null' => true,
              ],
              'body' => 'text',
              'published' => [
                  'type' => 'string',
                  'length' => 1,
                  'default' => 'N',
              ],
          ],
          'constraints' => [
              'primary' => [
                  'type' => 'primary',
                  'columns' => [
                      'id',
                  ],
              ],
          ],
       ],
       // Más tables.
    ];

Las opciones disponibles para ``columnas``, ``índices`` y ``restricciones``
coinciden con los atributos que están disponibles en las API de reflexión de
esquema de CakePHP. Las tablas se crean de forma incremental y debes asegurarte
de que se crean antes de que se hagan referencias a claves externas. Una vez que
se haya creado tu archivo de esquema, puedes cargarlo en tu ``tests/bootstrap.php``
con::

    $loader = new SchemaLoader();
    $loader->loadInternalFile($pathToSchemaFile);

Crear Esquema con Archivos de Volcado de SQL
--------------------------------------------

Para cargar un archivo de volcado de SQL puedes utilizar lo siguiente::

    // en tests/bootstrap.php
    use Cake\TestSuite\Fixture\SchemaLoader;

    // Carga uno o más archivos SQL.
    (new SchemaLoader())->loadSqlFiles('path/to/schema.sql', 'test');

Al principio de cada prueba, ``SchemaLoader`` eliminará todas las tablas en la
conexión y las reconstruirá basándose en el archivo de esquema proporcionado.

.. _fixture-state-management:

Gestores de Estado de Fixtures
------------------------------

De forma predeterminada, CakePHP restablece el estado de los fixtures al final
de cada prueba truncando todas las tablas en la base de datos. Esta operación
puede resultar costosa a medida que su aplicación crece. Al utilizar
``TransactionStrategy``, cada método de prueba se ejecutará dentro de una
transacción que se revertirá al final de la prueba. Esto puede mejorar el
rendimiento, pero requiere que tus pruebas no dependan en gran medida de datos
de fixtures estáticos ya que los valores de incremento automático no se
restablecen antes de cada prueba.

La estrategia de gestión del estado de fixture se puede definir dentro del
caso de prueba::

    use Cake\TestSuite\TestCase;
    use Cake\TestSuite\Fixture\FixtureStrategyInterface;
    use Cake\TestSuite\Fixture\TransactionStrategy;

    class ArticlesTableTest extends TestCase
    {
        /**
         * Crea la estrategia de fixture utilizada para este caso de prueba.
         * Puedes utilizar un class/trait base para cambiar varias clases.
         */
        protected function getFixtureStrategy(): FixtureStrategyInterface
        {
            return new TransactionStrategy();
        }
    }

Cerando Fixtures
----------------

Los fixtures definen registros que se insertarán en la base de datos de test al
comienzo de cada prueba. Creemos nuestro primer fixture, que se utilizará para
crear nuestro propio modelo Article. Crea un archivo llamado **ArticlesFixture.php**
en tu directorio **tests/Fixture** con el siguiente contenido::

    namespace App\Test\Fixture;

    use Cake\TestSuite\Fixture\TestFixture;

    class ArticlesFixture extends TestFixture
    {
          // Opcional. Establezca esta propiedad para cargar fixtures en una fuente
          // de datos de prueba diferente
          public $connection = 'test';

          public $records = [
              [
                  'title' => 'First Article',
                  'body' => 'First Article Body',
                  'published' => '1',
                  'created' => '2007-03-18 10:39:23',
                  'modified' => '2007-03-18 10:41:31'
              ],
              [
                  'title' => 'Second Article',
                  'body' => 'Second Article Body',
                  'published' => '1',
                  'created' => '2007-03-18 10:41:23',
                  'modified' => '2007-03-18 10:43:31'
              ],
              [
                  'title' => 'Third Article',
                  'body' => 'Third Article Body',
                  'published' => '1',
                  'created' => '2007-03-18 10:43:23',
                  'modified' => '2007-03-18 10:45:31'
              ]
          ];
     }

.. note::

    Se recomienda no agregar valores manualmente a las columnas con auto
    incrementales, ya que interfiere con la generación de secuencia en
    PostgreSQL y SQLServer.

La propiedad ``$connection`` define la fuente de datos que utilizará el fixture.
Si tu aplicación utiliza múltiples fuentes de datos, debes hacer que los fixtures
coincidan con las fuentes de datos del modelo pero con el prefijo ``test_``.
Por ejemplo, si tu module utiliza la fuente de datos ``mydb``, tu fixture debe
utilizar la fuente de datos ``test_mydb``. Si la conexión ``test_mydb`` no existe,
tus modulos utilizarán la fuente de datos predeterminada ``test``. Las fuentes de
datos de los fixture deben tener el prefijo ``test`` para reducir la posibilidad
de truncar accidentalmente todos los datos de su aplicación al ejecutar pruebas.

Podemos definir un conjunto de registros que se completarán después de crear la
tabla de fixtures. El formato es bastante sencillo, ``$records`` es una matriz de
registros. Cada elemento de ``$records`` debe ser una sola fila. Dentro de cada
fila, debe haber una matriz asociativa de las columnas y valores de cada fila.
Sólo ten en cuenta que cada registro en la matriz ``$records`` debe tener las
mismas claves que las filas que se insertan de forma masiva.

Datos Dinámicos
---------------

Para utilizar funciones u otros datos dinámicos en los registros de tu fixture,
puedes definir sus registros en el método ``init()`` del fixture::

    namespace App\Test\Fixture;

    use Cake\TestSuite\Fixture\TestFixture;

    class ArticlesFixture extends TestFixture
    {
        public function init(): void
        {
            $this->records = [
                [
                    'title' => 'First Article',
                    'body' => 'First Article Body',
                    'published' => '1',
                    'created' => date('Y-m-d H:i:s'),
                    'modified' => date('Y-m-d H:i:s'),
                ],
            ];
            parent::init();
        }
    }

.. note::
    Cuando sobreescribas ``init()`` recuerda llamar siempre a ``parent::init()``.

Cargando Fixtures en tus Casos de Prueba
----------------------------------------

Una vez hayas creado tus fixtures, querrás utilizarlos en tus casos de prueba.
En cada caso de prueba debes cargar los fixtures que necesitarás. Debe cargar
un fixture para cada modelo al que se ejecutará una consulta. Para cargar
fixtures, define la propiedad ``$fixtures`` en tu modelo::

    class ArticlesTest extends TestCase
    {
        protected $fixtures = ['app.Articles', 'app.Comments'];
    }

A partir de 4.1.0 puedes utilizar ``getFixtures()`` para definir tu lista
de fixtures con un método::

    public function getFixtures(): array
    {
        return [
            'app.Articles',
            'app.Comments',
        ];
    }

Lo anterior cargará los fixtures de Article and Comment desde el directorio
Fixture de tu aplicación. También puedes cargar fixtures desde el núcleo de
CakePHP, o plugins::

    class ArticlesTest extends TestCase
    {
        protected $fixtures = [
            'plugin.DebugKit.Articles',
            'plugin.MyVendorName/MyPlugin.Messages',
            'core.Comments',
        ];
    }

Usar el prefijo ``core`` cargará fixtures desde CakePHP y usar el nombre de un
plugin como prefijo, cargará el fixture desde el plugin nombrado.

Puedes cargar fixtures en subdirectorios. El uso de varios directorios puede
facilitar la organización de tus fixtures si tienes una aplicación muy grande.
Para cargar fixtures en subdirectorios, simplemente incluye el nombre del
subdirectorio en el nombre del fixture::

    class ArticlesTest extends CakeTestCase
    {
        protected $fixtures = ['app.Blog/Articles', 'app.Blog/Comments'];
    }

En el ejemplo anterior, ambos fixtures se cargarían desde
``tests/Fixture/Blog/``.

Factorías de Fixture
--------------------

A medida que tu aplicación crece, también crece el número y el tamaño de tus
fixtures de prueba. Puede que le resulte difícil mantenerlos y realizar un
seguimiento de su contenido. Los `fixture factories plugin
<https://github.com/vierge-noire/cakephp-fixture-factories>`_ propone una
alternativa para aplicaciones de gran tamaño.

El plugin utiliza el `test suite light plugin <https://github.com/vierge-noire/cakephp-test-suite-light>`_
para truncar todas las tablas sucias antes de cada prueba.

El siguiente comando te ayudará a ordenar tus factorías::

    bin/cake bake fixture_factory -h

Una vez que sus factorías estén
`sincronizadas <https://github.com/vierge-noire/cakephp-fixture-factories/blob/main/docs/factories.md>`_,
estará listo para crear fixtures de prueba en poco tiempo.

La interacción innecesaria con la base de datos ralentizará sus pruebas y su
aplicación. Puedes crear fixtures de prueba sin conservarlos, lo que puede
resultar útil para probar métodos sin interacción con la base de datos::

    $article = ArticleFactory::make()->getEntity();

Para persistir::

    $article = ArticleFactory::make()->persist();

Las factorías también ayudan a crear fixtures asociados.
Suponiendo que los artículos pertenecen a muchos autores, ahora podemos,
por ejemplo, crear 5 artículos cada uno con 2 autores::

    $articles = ArticleFactory::make(5)->with('Authors', 2)->getEntities();

Ten en cuenta que las factorías de fixture no requieren ninguna creación o
declaración de fixtures. Aun así, son totalmente compatibles con los fixtures que
vienen con cakephp. Encontrarás información y documentación adionales `aquí
<https://github.com/vierge-noire/cakephp-fixture-factories>`_.

Cargando Rutas en Pruebas
=========================

Si estás probando correos electrónicos, los componentes de controlador u otras
clases que requieren rutas y resolución de URL, necesitarás cargar rutas. Durante
el ``setUP()`` de una clase o durante los métodos de prueba individuales, puedes
usar ``loadRoutes()`` para asegurar que las rutas de tu aplicación estén cargadas::

    public function setUp(): void
    {
        parent::setUp();
        $this->loadRoutes();
    }

Este método creará una instancia de tu ``Application`` y llamará al método ``routes()``
en ella. Si tu clase ``Application`` requiere parámetros de constructor especializados,
puedes proporcionarlos a ``loadRoutes($constructorArgs)``.

Creando Rutas en Pruebas
------------------------

A veces puede ser necesario añadir rutas dinámicamente en las pruebas, por ejemplo
al desarrollar plugins or aplicaciones que sean extensibles.

Al igual que la carga de rutas de aplicación existentes, esto se puede hacer durante
el ``setup()`` de un método de prueba y/o en los propios métodos de prueba individuales::

    use Cake\Routing\Route\DashedRoute;
    use Cake\Routing\RouteBuilder;
    use Cake\Routing\Router;
    use Cake\TestSuite\TestCase;

    class PluginHelperTest extends TestCase
    {
        protected RouteBuilder $routeBuilder;

        public function setUp(): void
        {
            parent::setUp();

            $this->routeBuilder = Router::createRouteBuilder('/');
            $this->routeBuilder->scope('/', function (RouteBuilder $routes) {
                $routes->setRouteClass(DashedRoute::class);
                $routes->get(
                    '/test/view/{id}',
                    ['controller' => 'Tests', 'action' => 'view']
                );
                // ...
            });

            // ...
        }
    }

Esto creará una nueva instancia del generador de rutas que fusionará las rutas
conectadas en la misma colección de rutas utilizada por todas las demás instancias
del generador de rutas que ya existan o que aún no se hayan creado en el entorno.

Cargando Plugins en Pruebas
---------------------------

Si tu aplicación carga complementos dinámicamente, puedes utilizar
``loadPlugins()`` para cargar uno o más plugins durante las pruebas::

    public function testMethodUsingPluginResources()
    {
        $this->loadPlugins(['Company/Cms']);
        // Lógica de pruebas que requiere que se cargue Company/Cms.
    }

Probando Clases de Tabla
========================

Digamos que ya tenemos nuestra clase Articles Table definida
en **src/Model/Table/ArticlesTable.php**, y se ve así::

    namespace App\Model\Table;

    use Cake\ORM\Table;
    use Cake\ORM\Query\SelectQuery;

    class ArticlesTable extends Table
    {
        public function findPublished(SelectQuery $query): SelectQuery
        {
            $query->where([
                $this->getAlias() . '.published' => 1
            ]);

            return $query;
        }
    }

Ahora queremos configurar una prueba que pruebe esta clase de tabla. Ahora creemos un
archivo llamado **ArticlesTableTest.php** en tu directorio **tests/TestCase/Model/Table**
con el siguiente contenido::

    namespace App\Test\TestCase\Model\Table;

    use App\Model\Table\ArticlesTable;
    use Cake\TestSuite\TestCase;

    class ArticlesTableTest extends TestCase
    {
        protected $fixtures = ['app.Articles'];
    }

En la variable ``$fixtures`` de nuestros casos de prueba definimos el conjunto
de fixtures que utilizaremos. Debes recordar incluir todos los fixtures sobre
los que se realizarán consultas.

Crear un método de prueba
-------------------------

Ahora agreguemos un método para probar la función ``published()`` en la tabla Articles.
Edita el archivo **tests/TestCase/Model/Table/ArticlesTableTest.php** para que ahora
tenga este aspecto::

    namespace App\Test\TestCase\Model\Table;

    use App\Model\Table\ArticlesTable;
    use Cake\TestSuite\TestCase;

    class ArticlesTableTest extends TestCase
    {
        protected $fixtures = ['app.Articles'];

        public function setUp(): void
        {
            parent::setUp();
            $this->Articles = $this->getTableLocator()->get('Articles');
        }

        public function testFindPublished(): void
        {
            $query = $this->Articles->find('published')->select(['id', 'title']);
            $this->assertInstanceOf('Cake\ORM\Query\SelectQuery', $query);
            $result = $query->enableHydration(false)->toArray();
            $expected = [
                ['id' => 1, 'title' => 'First Article'],
                ['id' => 2, 'title' => 'Second Article'],
                ['id' => 3, 'title' => 'Third Article']
            ];

            $this->assertEquals($expected, $result);
        }
    }

Puedes ver que hemos añadido un método llamado ``testFindPublished()``. Comenzamos
creando una instancia de nuestra clase ``ArticlesTable`` y luego ejecutamos nuestro
método ``find('published')``. En ``$expected`` establecemos lo que esperamos que
sea el resultado adecuado (lo cual sabemos porque hemos definido qué registros se
completan inicialmente en la tabla de artículos). Probamos que el resultado sea
igual a nuestras expectativas usanto el método ``assertEquals()``. Consulta la
sección :ref:`running-tests` para más información sobre cómo ejecutar tu caso
de prueba.

Utilizando las factorías de fixtures, la prueba ahora se vería así::

    namespace App\Test\TestCase\Model\Table;

    use App\Test\Factory\ArticleFactory;
    use Cake\TestSuite\TestCase;

    class ArticlesTableTest extends TestCase
    {
        public function testFindPublished(): void
        {
            // Persisten 3 artículos publicados
            $articles = ArticleFactory::make(['published' => 1], 3)->persist();
            // Persisten 2 artículos publicados
            ArticleFactory::make(['published' => 0], 2)->persist();

            $result = ArticleFactory::find('published')->find('list')->toArray();

            $expected = [
                $articles[0]->id => $articles[0]->title,
                $articles[1]->id => $articles[1]->title,
                $articles[2]->id => $articles[2]->title,
            ];

            $this->assertEquals($expected, $result);
        }
    }

No es necesario cargar fixtures. Los 5 artículos creados existirán sólo en esta prueba. El
método estático ``::find()`` consultará la base de datos sin utilizar la tabla ``ArticlesTable``
y sus eventos.

Símulación de Métodos de Modelos
--------------------------------

Habrá ocasiones en las que quieras simular métodos en modelos cuando los pruebes.
Deberías usar ``getMockForModel`` para crear simulaciones de prueba de clases de
tabla. Esto evita problemas con las propiedades reflejas que tienen los modelos
normales::

    public function testSendingEmails(): void
    {
        $model = $this->getMockForModel('EmailVerification', ['send']);
        $model->expects($this->once())
            ->method('send')
            ->will($this->returnValue(true));

        $model->verifyEmail('test@example.com');
    }

Asegúrate de eliminar la simulación en tu método ``tearDown()`` con::

    $this->getTableLocator()->clear();

.. _integration-testing:

Pruebas de Integración del Controlador
======================================

Si bien puedes probar clases de controlador de manera similar a los helpers,
modelos y componentes, CakePHP ofrece un trait especializado ``IntegrationTestTrait``.
El uso de este trait en los casos de prueba de tu controlador te permite probar
controladores desde un alto nivel.

Si no estás familiarizado con las pruebas de integración, es un enfoque de prueba
que te permite probar varias unidades al mismo tiempo. Las funciones de prueba de
integración en CakePHP simulan una solicitud HTTP manejada por tu aplicación.
Por ejemplo, probar tu controlador también probará todos los componentes, modelos
y helpers involucrados en el manejor de una solicitud determinada. Esto te da una
prueba de mayor nivel de tu apicación y todas sus partes. funcionales.

Supongamos que tienes el típico ArticlesController y su modelo correspondiente.
El código del controlador se ve así::

    namespace App\Controller;

    use App\Controller\AppController;

    class ArticlesController extends AppController
    {
        public $helpers = ['Form', 'Html'];

        public function index($short = null)
        {
            if ($this->request->is('post')) {
                $article = $this->Articles->newEntity($this->request->getData());
                if ($this->Articles->save($article)) {
                    // Redirigir según el patrón PRG
                    return $this->redirect(['action' => 'index']);
                }
            }
            if (!empty($short)) {
                $result = $this->Articles->find('all', fields: ['id', 'title'])->all();
            } else {
                $result = $this->Articles->find()->all();
            }

            $this->set([
                'title' => 'Articles',
                'articles' => $result
            ]);
        }
    }

Crea in archivo llamado **ArticlesControllerTest.php** en tu directorio
**tests/TestCase/Controller**  y coloca lo siguiente dentro::

    namespace App\Test\TestCase\Controller;

    use Cake\TestSuite\IntegrationTestTrait;
    use Cake\TestSuite\TestCase;

    class ArticlesControllerTest extends TestCase
    {
        use IntegrationTestTrait;

        protected $fixtures = ['app.Articles'];

        public function testIndex(): void
        {
            $this->get('/articles');

            $this->assertResponseOk();
            // Más afirmaciones.
        }

        public function testIndexQueryData(): void
        {
            $this->get('/articles?page=1');

            $this->assertResponseOk();
            // Más afirmaciones.
        }

        public function testIndexShort(): void
        {
            $this->get('/articles/index/short');

            $this->assertResponseOk();
            $this->assertResponseContains('Articles');
            // Más afirmaciones.
        }

        public function testIndexPostData(): void
        {
            $data = [
                'user_id' => 1,
                'published' => 1,
                'slug' => 'new-article',
                'title' => 'New Article',
                'body' => 'New Body'
            ];
            $this->post('/articles', $data);

            $this->assertResponseSuccess();
            $articles = $this->getTableLocator()->get('Articles');
            $query = $articles->find()->where(['title' => $data['title']]);
            $this->assertEquals(1, $query->count());
        }
    }

Este ejemplo muestra algunos de los métodos de envío de solicitudes y algunas
de las afirmaciones que proporciona ``IntegrationTestTrait``. Antes de poder
realizar cualquier afirmación, debes enviar una solicitud. Puedes utilizar uno
de los siguientes métodos para enviar una solicitud::

* ``get()`` Envía una petición GET.
* ``post()`` Envía una petición POST.
* ``put()`` Envía una petición PUT.
* ``delete()`` Envía una petición DELETE.
* ``patch()`` Envía una petición PATCH.
* ``options()`` Envía una petición OPTIONS.
* ``head()`` Envía una petición HEAD.

Todos los métodos excepto ``get()`` y ``delete()`` aceptan un segundo parámetro
que te permite enviar un cuerpo de solicitud. Después de enviar una solicitud,
puedes utilizar las diversas afirmaciones proporcionadas por ``IntegrationsTestTraitp``
o PHPUnit para asegurarte de que tu solicitud tenga los efectos secundarios correctos.

Configurar la Solicitud
-----------------------

El trait ``IntegrationTestTrair`` viene con una serie de helpers para
configurar las solicitudes que enviarás a tu aplicación bajo prueba::

    // Establecer cookies
    $this->cookie('name', 'Uncle Bob');

    // Establecer datos de sesión
    $this->session(['Auth.User.id' => 1]);

    // Configurar cabeceras
    $this->configRequest([
        'headers' => ['Accept' => 'application/json']
    ]);

El estado establecido por estos métodos auxiliares se restablece en el método
``tearDown()``.

Acciones de Prueba Protegidas por CsrfComponent o SecurityComponent
-------------------------------------------------------------------

Al probar acciones protegidas por SecurityComponent o CsfrComponent, puedes
habilitar la generación automática de tokens para garantizar que tus pruebas
no fallen debido a discrepancias de tokens::

    public function testAdd(): void
    {
        $this->enableCsrfToken();
        $this->enableSecurityToken();
        $this->post('/posts/add', ['title' => 'Exciting news!']);
    }

También es importante habilitar la depuración en las pruebas que utilizan tokens
para evitar que SecurityComponent piense que el token de depuración se está
utilizando en un entorno que no es de depuración. Al probar con otros métodos
como ``requireSecure()`` puedes usar ``configRequest()`` para configurar las
variables de entorno correctas::

    // Falsificar conexiones SSL.
    $this->configRequest([
        'environment' => ['HTTPS' => 'on']
    ]);

Si tu acción requiere campos desbloqueados, puedes declararlos con
``setUnlockedFields()``::

    $this->setUnlockedFields(['dynamic_field']);

Pruebas de Integración del Middleware PSR-7
-------------------------------------------

Las pruebas de integración también se pueden utilizar para probar toda tu
aplicación PSR-7 y :doc:`/controllers/middleware`. De manera predeterminada,
``IntegrationTestTrait`` detectará automáticamente la presencia de una clase
``App\Application`` y habilitará automáticamente las pruebas de integración
de tu aplicación.

Puedes personalizar el nombre de la calse de aplicación utilizada y los
argumentos del constructor utilizando el método ``configApplication()``::

    public function setUp(): void
    {
        $this->configApplication('App\App', [CONFIG]);
    }

También debes tener cuidado de intengar utilizar :ref:`application-bootstrap`
para cargar cualquier plugin que contenga eventos/rutas. Al hacerlo, se
asegurará de que sus eventos/rutas estén conectados para cada caso de prueba.
Alternativamente, si deseas cargar plugins manualmente en una prueba, puedes
utilizar el método ``loadPlugins()``.

Pruebas con Cookies Cifradas
----------------------------

SI utilizas :ref:`encrypted-cookie-middleware` en tu aplicación, existen métodos
auxiliares para configurar cookies cifradas en sus casos de prueba::

    // Establecer una cookie utilizando AES y la clave predeterminada.
    $this->cookieEncrypted('my_cookie', 'Algunos valores secretos');

    // Supongamos que esta acción modifica la cookie.
    $this->get('/articles/index');

    $this->assertCookieEncrypted('Un valor actualizado', 'my_cookie');

Prueba de Mensajes Flash
------------------------

Si deseas afirmar la presencia de mensajes flash en la sesión y no el HTML
rednderizado, puedes usa ``enableRetainFlashMessages()`` en tus pruebas para
retener mensajes flash en la sesión para poder escribir afirmaciones::

    // Habilia la retención de mensajes flash en lugar de consumirlos.
    $this->enableRetainFlashMessages();
    $this->get('/articles/delete/9999');

    $this->assertSession('That article does not exist', 'Flash.flash.0.message');

    // Afirmar un mensaje flash en la clave 'flash'.
    $this->assertFlashMessage('Article deleted', 'flash');

    // Afirmar el segundo mensaje flash, también en la clave 'flash'.
    $this->assertFlashMessageAt(1, 'Article really deleted');

    // Afirmar un mensaje flash en la clave 'auth' en la primera posición.
    $this->assertFlashMessageAt(0, 'You are not allowed to enter this dungeon!', 'auth');

    // Afirmar que un mensaje flash utiliza el elemento de error
    $this->assertFlashElement('Flash/error');

    // Afirma el segundo elemento del mensaje flash.
    $this->assertFlashElementAt(1, 'Flash/error');

Prueba de un Controlador de Respues JSON
----------------------------------------

JSON es un formato amigable y común para usar al crear un servicio web. Probar
los puntos finales de tu servicio web es muy sencillo con CakePHP. Comencemos
con un ejemplo simple de controlar que rsponde en JSON::

    use Cake\View\JsonView;

    class MarkersController extends AppController
    {
        public function viewClasses(): array
        {
            return [JsonView::class];
        }

        public function view($id)
        {
            $marker = $this->Markers->get($id);
            $this->set('marker', $marker);
            $this->viewBuilder()->setOption('serialize', ['marker']);
        }
    }

Ahora creamos el archivo **test/TestCase/Controller/MarkersControllerTest.php**
y nos aseguramos de que nuestro servicio web devuelva la respuesta adecuada::

    class MarkersControllerTest extends IntegrationTestCase
    {
        use IntegrationTestTrait;

        public function testGet(): void
        {
            $this->configRequest([
                'headers' => ['Accept' => 'application/json']
            ]);
            $this->get('/markers/view/1.json');

            // Comprueba que la respuesta fue un 200
            $this->assertResponseOk();

            $expected = [
                ['id' => 1, 'lng' => 66, 'lat' => 45],
            ];
            $expected = json_encode($expected, JSON_PRETTY_PRINT);
            $this->assertEquals($expected, (string)$this->_response->getBody());
        }
    }

Utilizamos la opción ``JSON_PRETTY_PRINT`` ya que el JsonView integrado en CakePHP
usará esa opción cuando ``debug`` esté habilitado.

Pruebas con Cargas de Archivos
------------------------------

Simular la carga de archivos es sencillo cuando se utiliza el modo predeterminado
":ref:`uploaded files as objects <request-file-uploads>`". Simplemente puedes crear
instancias que implementen `\\Psr\\Http\\Message\\UploadedFileInterface <https://www.php-fig.org/psr/psr-7/#16-uploaded-files>`__
(la implementación predeterminada utilizada por CakePHP es ``\Laminas\Diactoros\UploadedFile``)
y pasarlos en los datos de tu solicitud de prueba. En el entorno CLI, dichos
objectos pasarán de forma predeterminada comprobaciones de validación que prueban
si el archivo se cargó a través de HTTP. No ocurre lo mismo con los datos de estilo
de matriz que se encuentran en ``$_FILES``; no pasaría la verificación.

Para simular exactamente cómo los objectos de archivos cargados estarían presentes
en una solicitud normal, no solo necesias pasarlos en los datos de la solicitud,
sino que también debes pasarlos a la configuración de la solicitud de prueba a través
de la opción ``archivos``. Sin embargo, no es técnicamente necesario a menos que tu
código acceda a los archivos cargados a través de los métodos
:php:meth:`Cake\\Http\\ServerRequest::getUploadedFile()` o
:php:meth:`Cake\\Http\\ServerRequest::getUploadedFiles()`.

Supongamos que los artículos tienen una imagen teaser y una asociación
``Articles hasMany Attachments``, el formulario se vería así, donde aceptaría un
archivo de imagen y múltiples archivos adjuntos::

    <?= $this->Form->create($article, ['type' => 'file']) ?>
    <?= $this->Form->control('title') ?>
    <?= $this->Form->control('teaser_image', ['type' => 'file']) ?>
    <?= $this->Form->control('attachments.0.attachment', ['type' => 'file']) ?>
    <?= $this->Form->control('attachments.0.description']) ?>
    <?= $this->Form->control('attachments.1.attachment', ['type' => 'file']) ?>
    <?= $this->Form->control('attachments.1.description']) ?>
    <?= $this->Form->button('Submit') ?>
    <?= $this->Form->end() ?>

La prueba que simularía la solicitud correspondiente podría verse así::

    public function testAddWithUploads(): void
    {
        $teaserImage = new \Laminas\Diactoros\UploadedFile(
            '/path/to/test/file.jpg', // secuencia o ruta al archivo que representa el archivo temporal
            12345,                    // el tamaño del archivo en bytes
            \UPLOAD_ERR_OK,           // el estado de carga/error
            'teaser.jpg',             // el nombre del archivo enviado por el cliente
            'image/jpeg'              // el tipo mime tal como lo envió el cliente
        );

        $textAttachment = new \Laminas\Diactoros\UploadedFile(
            '/path/to/test/file.txt',
            12345,
            \UPLOAD_ERR_OK,
            'attachment.txt',
            'text/plain'
        );

        $pdfAttachment = new \Laminas\Diactoros\UploadedFile(
            '/path/to/test/file.pdf',
            12345,
            \UPLOAD_ERR_OK,
            'attachment.pdf',
            'application/pdf'
        );

        // Estos son lso datos accesibles a través de `$this->request->getUploadedFile()`
        // y `$this->request->getUploadedFiles()`.
        $this->configRequest([
            'files' => [
                'teaser_image' => $teaserImage,
                'attachments' => [
                    0 => [
                        'attachment' => $textAttachment,
                    ],
                    1 => [
                        'attachment' => $pdfAttachment,
                    ],
                ],
            ],
        ]);

        // Estos son los datos accesibles a través de `$this->request->getData()`.
        $postData = [
            'title' => 'New Article',
            'teaser_image' => $teaserImage,
            'attachments' => [
                0 => [
                    'attachment' => $textAttachment,
                    'description' => 'Text attachment',
                ],
                1 => [
                    'attachment' => $pdfAttachment,
                    'description' => 'PDF attachment',
                ],
            ],
        ];
        $this->post('/articles/add', $postData);

        $this->assertResponseOk();
        $this->assertFlashMessage('The article was saved successfully');
        $this->assertFileExists('/path/to/uploads/teaser.jpg');
        $this->assertFileExists('/path/to/uploads/attachment.txt');
        $this->assertFileExists('/path/to/uploads/attachment.pdf');
    }

.. tip::

    Si configuras la solicitud de prueba con archivos, entonces *debe* coincidir
    con la estructura de tus datos POST (pero solo incluye los objetos de archivo cargados)

Del mismo modo, puedes simular `errores de carga <https://www.php.net/manual/en/features.file-upload.errors.php>`_
o archivos no válidos que no pasan la validación::

    public function testAddWithInvalidUploads(): void
    {
        $missingTeaserImageUpload = new \Laminas\Diactoros\UploadedFile(
            '',
            0,
            \UPLOAD_ERR_NO_FILE,
            '',
            ''
        );

        $uploadFailureAttachment = new \Laminas\Diactoros\UploadedFile(
            '/path/to/test/file.txt',
            1234567890,
            \UPLOAD_ERR_INI_SIZE,
            'attachment.txt',
            'text/plain'
        );

        $invalidTypeAttachment = new \Laminas\Diactoros\UploadedFile(
            '/path/to/test/file.exe',
            12345,
            \UPLOAD_ERR_OK,
            'attachment.exe',
            'application/vnd.microsoft.portable-executable'
        );

        $this->configRequest([
            'files' => [
                'teaser_image' => $missingTeaserImageUpload,
                'attachments' => [
                    0 => [
                        'file' => $uploadFailureAttachment,
                    ],
                    1 => [
                        'file' => $invalidTypeAttachment,
                    ],
                ],
            ],
        ]);

        $postData = [
            'title' => 'New Article',
            'teaser_image' => $missingTeaserImageUpload,
            'attachments' => [
                0 => [
                    'file' => $uploadFailureAttachment,
                    'description' => 'Upload failure attachment',
                ],
                1 => [
                    'file' => $invalidTypeAttachment,
                    'description' => 'Invalid type attachment',
                ],
            ],
        ];
        $this->post('/articles/add', $postData);

        $this->assertResponseOk();
        $this->assertFlashMessage('The article could not be saved');
        $this->assertResponseContains('A teaser image is required');
        $this->assertResponseContains('Max allowed filesize exceeded');
        $this->assertResponseContains('Unsupported file type');
        $this->assertFileNotExists('/path/to/uploads/teaser.jpg');
        $this->assertFileNotExists('/path/to/uploads/attachment.txt');
        $this->assertFileNotExists('/path/to/uploads/attachment.exe');
    }

Deshabilitar el Middleware de Manejo de Errores en las Pruebas
--------------------------------------------------------------

Al depurar pruebas que fallan porque su aplicación esncuentra errores, puede
ser útil deshabilitar temporalmente el middleware de manejo de errores para
permitir que el error subyacente surja. Puedes usar ``disableErrorHandlerMiddleware()``
para hacer esto::

    public function testGetMissing(): void
    {
        $this->disableErrorHandlerMiddleware();
        $this->get('/markers/not-there');
        $this->assertResponseCode(404);
    }

En el ejemplo anterior, la prueba fallaría y se mostrarían el mensaje de excepción
subyacente y el seguimiento de la pila en lugar de verificar la página de error
representada.

Métodos de Afirmación
---------------------

El trait ``IntegrationTestTrait`` proporciona una serie de métodos de afirmación
que simplifican mucho las pruebas de respuestas. Algunos ejemplos son::

    // Comprueba si hay un código 2xx en la respuesta
    $this->assertResponseOk();

    // Comprueba si hay un código 2xx/3xx en la respuesta
    $this->assertResponseSuccess();

    // Comprueba si hay un código 4xx en la respuesta
    $this->assertResponseError();

    // Comprueba si hay un código 5xx en la respuesta
    $this->assertResponseFailure();

    // Comprueba si hay un código específico en la respueat, por ejemplo, 200
    $this->assertResponseCode(200);

    // Comprueba la ubicación del encabezado
    $this->assertRedirect(['controller' => 'Articles', 'action' => 'index']);

    // Comprueba que no se haya establecido ninguna ubicación en el encabezado
    $this->assertNoRedirect();

    // Comprueba una parte de la ubicación del encabezado
    $this->assertRedirectContains('/articles/edit/');

    // Comprueba que la ubicación en el encabezado no contiene
    $this->assertRedirectNotContains('/articles/edit/');

    // Afirmar contenido de la respuesta no vacío
    $this->assertResponseNotEmpty();

    // Afirmar contenido de la respuesta vacío
    $this->assertResponseEmpty();

    // Afirmar contenido de la respuesta
    $this->assertResponseEquals('Yeah!');

    // Afirmar que el contenido de la respuesta no es igual a
    $this->assertResponseNotEquals('No!');

    // Afirmar contenido parcial de la respuesta
    $this->assertResponseContains('You won!');
    $this->assertResponseNotContains('You lost!');

    // Afirmar archivo devuelto
    $this->assertFileResponse('/absolute/path/to/file.ext');

    // Afirmar layout
    $this->assertLayout('default');

    // Afimar qué template se representó (si corresponde)
    $this->assertTemplate('index');

    // Afirmar datos en la sesión
    $this->assertSession(1, 'Auth.User.id');

    // Afirmar encabezado de la respuesta
    $this->assertHeader('Content-Type', 'application/json');
    $this->assertHeaderContains('Content-Type', 'html');

    // Afirmar que el encabezado de tipo de contenido no contiene xml
    $this->assertHeaderNotContains('Content-Type', 'xml');

    // Afirmar variables de la vista
    $user =  $this->viewVariable('user');
    $this->assertEquals('jose', $user->username);

    // Afirmar los valores de las cookies en la respuesta
    $this->assertCookie('1', 'thingid');

    // Afirmar que una cookie está presente o no
    $this->assertCookieIsSet('remember_me');
    $this->assertCookieNotSet('remember_me');

    // Comprobar el tipo de contenido
    $this->assertContentType('application/json');

Además de los métodos de afimarción ateriores, también puedes utilizar
todas las afirmaciones en `TestSuite
<https://api.cakephp.org/5.x/class-Cake.TestSuite.TestCase.html>`_ y las que
se encuentran en `PHPUnit
<https://phpunit.de/manual/current/en/appendixes.assertions.html>`__.

Comparar los Resultados de la Prueba con un Archivo
---------------------------------------------------

Para algunos tipos de pruebas, puede ser más fácil comparar el resultado de una
prueba con el contenido de un archivo, por ejemplo, al probar la salida renderizada
de una vista. ``StringCompareTrait`` agrega un método de afirmación simple para
este propósito.

El uso implica utilizar el trait, establecer la ruta base de comparación y llamar a
``assertSameAsFile``::

    use Cake\TestSuite\StringCompareTrait;
    use Cake\TestSuite\TestCase;

    class SomeTest extends TestCase
    {
        use StringCompareTrait;

        public function setUp(): void
        {
            $this->_compareBasePath = APP . 'tests' . DS . 'comparisons' . DS;
            parent::setUp();
        }

        public function testExample(): void
        {
            $result = ...;
            $this->assertSameAsFile('example.php', $result);
        }
    }

El ejemplo anterior comparará ``$result`` con el contenido del archivo
``APP/tests/comparisons/example.php``.

Se proporciona un mecanismo para escribir/actualizar archivos de prueba,
configurando la variable de entorno ``UPDATE_TEST_COMPARISON_FILES``, que
creará y/o actualizará archivos de comparación de prueba a medida que se hace
referencia a ellos:

.. code-block:: console

    phpunit
    ...
    FAILURES!
    Tests: 6, Assertions: 7, Failures: 1

    UPDATE_TEST_COMPARISON_FILES=1 phpunit
    ...
    OK (6 tests, 7 assertions)

    git status
    ...
    # Changes not staged for commit:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #   modified:   tests/comparisons/example.php


Pruebas de Integración de consola
=================================

Consulta :ref:`console-integration-testing` para saber cómo probar los comandos de la consola.

Simulación de Dependencias Inyectadas
=====================================

Consulta :ref:`mocking-services-in-tests` para saber cómo reemplazar los servicios inyectados
con el contenedor de inyección de dependencia en sus pruebas de integración.

Simulación de Respuestas de Clientes HTTP
=========================================

Consulta :ref:`httpclient-testing` para saber cómo crear respuestas simuladas a API externas.

Probando Vistas
===============

Generalmetne la mayoría de aplicaciones no probarán directamente tu código HTML.
Hacerlo a menudo resulta en pruebas frágiles, difíciles de mantener y propensas
a romperse. Al escribir pruebas funcionales utilizando :php:class:`IntegrationTrait`
puedes inspeccionar el contenido de la vista renderizada configurando la opción
``return`` a `view`. Si bien es posible probar la visualización de contenido usando
``IntegratinTestTrait``, se puede lograr una prueba de integración más sólida y
fácil de mantener utilizando herramientas como `Selenium webdriver <https://www.selenium.dev/>`_.

Probando Componentes
====================

Supongamos que tenemos un componente llamado PagematronComponent in nuestra
aplicación. Este componente nos ayuda a establecer el valor límite de paginación
en todos los controladores que lo utilizan. Aquí está nuestro componente de
ejemplo ubicado en **src/Controller/Component/PagematronComponent.php**::

    class PagematronComponent extends Component
    {
        public $controller = null;

        public function setController($controller)
        {
            $this->controller = $controller;
            // Asegúrate de que el controlador esté usando paginación
            if (!isset($this->controller->paginate)) {
                $this->controller->paginate = [];
            }
        }

        public function startup(EventInterface $event)
        {
            $this->setController($event->getSubject());
        }

        public function adjust($length = 'short'): void
        {
            switch ($length) {
                case 'long':
                    $this->controller->paginate['limit'] = 100;
                break;
                case 'medium':
                    $this->controller->paginate['limit'] = 50;
                break;
                default:
                    $this->controller->paginate['limit'] = 20;
                break;
            }
        }
    }

Ahora podemos escribir pruebas para asegurarnos de que nuestro parámetro de paginación
``limit`` esté configurado correctamente mediante el método ``adjust()`` en nuestro
componente. Creamos el archivo
**tests/TestCase/Controller/Component/PagematronComponentTest.php**::

    namespace App\Test\TestCase\Controller\Component;

    use App\Controller\Component\PagematronComponent;
    use Cake\Controller\Controller;
    use Cake\Controller\ComponentRegistry;
    use Cake\Event\Event;
    use Cake\Http\ServerRequest;
    use Cake\Http\Response;
    use Cake\TestSuite\TestCase;

    class PagematronComponentTest extends TestCase
    {
        protected $component;
        protected $controller;

        public function setUp(): void
        {
            parent::setUp();
            // Configura nuestro componente y controlador de prueba falso
            $request = new ServerRequest();
            $response = new Response();
            $this->controller = $this->getMockBuilder('Cake\Controller\Controller')
                ->setConstructorArgs([$request, $response])
                ->setMethods(null)
                ->getMock();
            $registry = new ComponentRegistry($this->controller);
            $this->component = new PagematronComponent($registry);
            $event = new Event('Controller.startup', $this->controller);
            $this->component->startup($event);
        }

        public function testAdjust(): void
        {
            // Prueba nuestro método de ajuste con diferentes configuraciones de parámetros
            $this->component->adjust();
            $this->assertEquals(20, $this->controller->paginate['limit']);

            $this->component->adjust('medium');
            $this->assertEquals(50, $this->controller->paginate['limit']);

            $this->component->adjust('long');
            $this->assertEquals(100, $this->controller->paginate['limit']);
        }

        public function tearDown(): void
        {
            parent::tearDown();
            // Limpiamos después de que hayamos terminado
            unset($this->component, $this->controller);
        }
    }

Probando Helpers
================

Dado que una cantidad decente de lógica reside en las clases de
los Helper, es importante asegurarse de que esas clases estén
cubiertas por casos de prueba.

Primer creamos un ejemplo de helper para probar. El ``CurrencyRendererHelper``
nos ayudará a mostrar las monedas en nuestras vistas y, para simplificar, solo
tendrá un método ``usd()``::

    // src/View/Helper/CurrencyRendererHelper.php
    namespace App\View\Helper;

    use Cake\View\Helper;

    class CurrencyRendererHelper extends Helper
    {
        public function usd($amount): string
        {
            return 'USD ' . number_format($amount, 2, '.', ',');
        }
    }

Aquí configuramos las posiciones decimales en 2, el separador decimal en
punto, el separador de miles en coma y anteponemos el número formateado
con la cadena 'USD'.

Ahora creamos nuestras pruebas::

    // tests/TestCase/View/Helper/CurrencyRendererHelperTest.php

    namespace App\Test\TestCase\View\Helper;

    use App\View\Helper\CurrencyRendererHelper;
    use Cake\TestSuite\TestCase;
    use Cake\View\View;

    class CurrencyRendererHelperTest extends TestCase
    {
        public $helper = null;

        // Aquí instanciamos nuestro helper
        public function setUp(): void
        {
            parent::setUp();
            $View = new View();
            $this->helper = new CurrencyRendererHelper($View);
        }

        // Probando la función usd()
        public function testUsd(): void
        {
            $this->assertEquals('USD 5.30', $this->helper->usd(5.30));

            // Siempre debemos tener 2 dígitos decimales
            $this->assertEquals('USD 1.00', $this->helper->usd(1));
            $this->assertEquals('USD 2.05', $this->helper->usd(2.05));

            // Probando el separador de miles
            $this->assertEquals(
              'USD 12,000.70',
              $this->helper->usd(12000.70)
            );
        }
    }

Aquí llamamos a ``usd()```con diferentes parámetros y le indicamos al conjunto
de pruebas que verifique si los valores devueltos son iguales a lo esperado.

Guarda esto y ejecuta la prueba. Deberías ver una barra verde y mensajes que
indiquen 1 aprobación y 4 afirmaciones.

Cuando estés probando un helper que utiliza otros helpers, asegúrate de simular
el método ``loadHelpers`` de la clase View.

.. _testing-events:

Probando Events
===============

El :doc:`/core-libraries/events` es una excelente manera de desacoplar el código
de tu aplicación, pero a veces, al realizar pruebas, tiendes a probar los resultados
de los eventos en los casos de prueba que ejecutan esos eventos. esta es una forma
adicional de acoplamiento que se puede eliminar usando ``asserEventFired`` y
``assertEventFiredWith`` en su lugar.

Amplando el ejemplo de Orders, digamos que tenemos las siguiente tablas::

    class OrdersTable extends Table
    {
        public function place($order): bool
        {
            if ($this->save($order)) {
                // Movida la eliminación del carrito a CartsTable
                $event = new Event('Model.Order.afterPlace', $this, [
                    'order' => $order
                ]);
                $this->getEventManager()->dispatch($event);
                return true;
            }
            return false;
        }
    }

    class CartsTable extends Table
    {
        public function implementedEvents(): array
        {
            return [
                'Model.Order.afterPlace' => 'removeFromCart'
            ];
        }

        public function removeFromCart(EventInterface $event): void
        {
            $order = $event->getData('order');
            $this->delete($order->cart_id);
        }
    }

.. note::
    Para afirmar que los eventos se activan, primero debes habilitar
    :ref:`tracking-events` en el administrador de eventos contra el
    que desea afirmar.

Para probar la ``OrdersTable`` anterior, habilitamos el seguimiento en ``setUp()``,
luego afirmamos que el evento se activó y afirmamos que la entidad ``$order`` se
pasó en los datos del evento::

    namespace App\Test\TestCase\Model\Table;

    use App\Model\Table\OrdersTable;
    use Cake\Event\EventList;
    use Cake\TestSuite\TestCase;

    class OrdersTableTest extends TestCase
    {
        protected $fixtures = ['app.Orders'];

        public function setUp(): void
        {
            parent::setUp();
            $this->Orders = $this->getTableLocator()->get('Orders');
            // habilitar seguimiento del evento
            $this->Orders->getEventManager()->setEventList(new EventList());
        }

        public function testPlace(): void
        {
            $order = new Order([
                'user_id' => 1,
                'item' => 'Cake',
                'quantity' => 42,
            ]);

            $this->assertTrue($this->Orders->place($order));

            $this->assertEventFired('Model.Order.afterPlace', $this->Orders->getEventManager());
            $this->assertEventFiredWith('Model.Order.afterPlace', 'order', $order, $this->Orders->getEventManager());
        }
    }

De forma predeterminada, el ``EventManager`` global se utiliza para las aserciones,
por lo que probar eventos globales no requiere pasar el administrador de eventos::

    $this->assertEventFired('My.Global.Event');
    $this->assertEventFiredWith('My.Global.Event', 'user', 1);


Probando el Correo Electrónico
==============================

Consulta :ref:`email-testing` para obtener información sobre cómo probar el correo electrónico.

Creando Conjuntos de Pruebas
============================

Si deseas que se ejecuten varias pruebas al mismo tiempo, puedes crear un conjunto
de pruebas. Un conjunto de pruebas se compone de varios casos de prueba. Puedes
crear conjuntos de pruebas en el archivo * * phpunit.xml** de tu aplicación. Un
ejemplo sencillo sería:

.. code-block:: xml

    <testsuites>
      <testsuite name="Models">
        <directory>src/Model</directory>
        <file>src/Service/UserServiceTest.php</file>
        <exclude>src/Model/Cloud/ImagesTest.php</exclude>
      </testsuite>
    </testsuites>

Creando Pruebas para Plugins
============================

Las pruebas para los plugins se crean en su propio directorio dentro de
los plugins. ::

    /src
    /plugins
        /Blog
            /tests
                /TestCase
                /Fixture


Funcionan como pruebas normales, pero debes recordad usar las convenciones de
nomenclatura para plugins al importar clases. Este es un ejemplo de un caso de
prueba para el modelo ``BlogPost`` del capítulo de plugins de este manual. Una
diferencia con otras pruebas está en la primera línea donde se importa 'Blog.BlogPost'.
También necesitas anteponer los fixtures de tu plugin con ``plugin.Blog.BlofPosts``::

    namespace Blog\Test\TestCase\Model\Table;

    use Blog\Model\Table\BlogPostsTable;
    use Cake\TestSuite\TestCase;

    class BlogPostsTableTest extends TestCase
    {
        // Fixtures del plugin ubicados en /plugins/Blog/tests/Fixture/
        protected $fixtures = ['plugin.Blog.BlogPosts'];

        public function testSomething(): void
        {
            // Prueba algo.
        }
    }

Si deseas utilizar fixtures de plugin en las pruebas de la aplicación,
puedes hacer referencia a ellos usando la sintaxis ``plugin.pluginName.fixtureName``
en la matrix ``$fixtures``. Además, si utilizas el nombre de un plugin de proveedor
o los directorios de fixtures, puedes usar lo siguiente:
``plugin.vendorName/pluginName.folderName/fixtureName``.

Antes de poder utilizar fixtures, debes asegurarte de tener :ref:`fixture
listener <fixture-phpunit-configuration>` configuirado en tu archivo ``phpunit.xml``.
También debes asegurarte de que tus fixtures se pueden cargar. Aegúrate de que lo
siguiente esté presente en tu archivo **composer.json**::

    "autoload-dev": {
        "psr-4": {
            "MyPlugin\\Test\\": "plugins/MyPlugin/tests/"
        }
    }

.. note::

    Recuerda ejecutar ``composer.phar dumpautoload`` cuanto agregues nuevas
    asignaciones de carga automática.

Generando Pruebas con Bake
==========================

Si usas :doc:`bake </bake/usage>` para generar estructuras, también generará
resguardos de pruebas. Si necesitas volver a generar esqueletos de casos de
pruebas o necesitas generar esqueletos de casos de pruebas par el código que
has escrito, puedes usar ``bake``:

.. code-block:: console

    bin/cake bake test <type> <name>

``<type>`` debería ser uno de:

#. Entity
#. Table
#. Controller
#. Component
#. Behavior
#. Helper
#. Shell
#. Task
#. ShellHelper
#. Cell
#. Form
#. Mailer
#. Command

Mientra que ``<name>`` debería ser el nombre del objeto del que deseas crear
un esqueleto de prueba.

.. meta::
    :title lang=es: Pruebas
    :keywords lang=es: phpunit,pruebas de base de datos,configuración de base de datos,prueba de base de datos,prueba pública,marco de pruebas,ejecución de uno,configuración de prueba,estándar de factor,pear,runners,corredores,array,matriz,arreglo,bases de datos,cakephp,php,integración
