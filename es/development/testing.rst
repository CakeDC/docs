Testing
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
-------------------

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

[Continuar]
Unnecessary interaction with the database will slow down your tests as well as
your application. You can create test fixtures without persisting them which can
be useful for testing methods without DB interaction::

    $article = ArticleFactory::make()->getEntity();

In order to persist::

    $article = ArticleFactory::make()->persist();

The factories help creating associated fixtures too.
Assuming that articles belongs to many authors, we can now, for example,
create 5 articles each with 2 authors::

    $articles = ArticleFactory::make(5)->with('Authors', 2)->getEntities();

Note that the fixture factories do not require any fixture creation or
declaration. Still, they are fully compatible with the fixtures that come with
cakephp. You will find additional insights and documentation `here
<https://github.com/vierge-noire/cakephp-fixture-factories>`_.

Loading Routes in Tests
=======================

If you are testing mailers, controller components or other classes that require
routes and resolving URLs, you will need to load routes. During
the ``setUp()`` of a class or during individual test methods you can use
``loadRoutes()`` to ensure your application routes are loaded::

    public function setUp(): void
    {
        parent::setUp();
        $this->loadRoutes();
    }

This method will build an instance of your ``Application`` and call the
``routes()`` method on it. If your ``Application`` class requires specialized
constructor parameters you can provide those to ``loadRoutes($constructorArgs)``.

Creating Routes in Tests
------------------------

Sometimes it may be be necessary to dynamically add routes in tests, for example
when developing plugins, or applications that are extensible.

Just like loading existing application routes, this can be done during ``setup()``
of a test method, and/or in the individual test methods themselves::

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

This will create a new route builder instance that will merge connected routes
into the same route collection used by all other route builder instances that
may already exist, or are yet to be created in the environment.

Loading Plugins in Tests
------------------------

If your application would dynamically load plugins, you can use
``loadPlugins()`` to load one or more plugins during tests::

    public function testMethodUsingPluginResources()
    {
        $this->loadPlugins(['Company/Cms']);
        // Test logic that requires Company/Cms to be loaded.
    }

Testing Table Classes
=====================

Let's say we already have our Articles Table class defined in
**src/Model/Table/ArticlesTable.php**, and it looks like::

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

We now want to set up a test that will test this table class. Let's now create
a file named **ArticlesTableTest.php** in your **tests/TestCase/Model/Table** directory,
with the following contents::

    namespace App\Test\TestCase\Model\Table;

    use App\Model\Table\ArticlesTable;
    use Cake\TestSuite\TestCase;

    class ArticlesTableTest extends TestCase
    {
        protected $fixtures = ['app.Articles'];
    }

In our test cases' variable ``$fixtures`` we define the set of fixtures that
we'll use. You should remember to include all the fixtures that will have
queries run against them.

Creating a Test Method
----------------------

Let's now add a method to test the function ``published()`` in the Articles
table. Edit the file **tests/TestCase/Model/Table/ArticlesTableTest.php** so it
now looks like this::

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

You can see we have added a method called ``testFindPublished()``. We start by
creating an instance of our ``ArticlesTable`` class, and then run our
``find('published')`` method. In ``$expected`` we set what we expect should be
the proper result (that we know since we have defined which records are
initially populated to the article table.) We test that the result equals our
expectation by using the ``assertEquals()`` method. See the :ref:`running-tests`
section for more information on how to run your test case.

Using the fixture factories, the test would now look like this::

    namespace App\Test\TestCase\Model\Table;

    use App\Test\Factory\ArticleFactory;
    use Cake\TestSuite\TestCase;

    class ArticlesTableTest extends TestCase
    {
        public function testFindPublished(): void
        {
            // Persist 3 published articles
            $articles = ArticleFactory::make(['published' => 1], 3)->persist();
            // Persist 2 unpublished articles
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

No fixtures need to be loaded. The 5 articles created will exist only in this test. The
static method ``::find()`` will query the database without using the table ``ArticlesTable``
and it's events.

Mocking Model Methods
---------------------

There will be times you'll want to mock methods on models when testing them. You
should use ``getMockForModel`` to create testing mocks of table classes. It
avoids issues with reflected properties that normal mocks have::

    public function testSendingEmails(): void
    {
        $model = $this->getMockForModel('EmailVerification', ['send']);
        $model->expects($this->once())
            ->method('send')
            ->will($this->returnValue(true));

        $model->verifyEmail('test@example.com');
    }

In your ``tearDown()`` method be sure to remove the mock with::

    $this->getTableLocator()->clear();

.. _integration-testing:

Controller Integration Testing
==============================

While you can test controller classes in a similar fashion to Helpers, Models,
and Components, CakePHP offers a specialized ``IntegrationTestTrait`` trait.
Using this trait in your controller test cases allows you to
test controllers from a high level.

If you are unfamiliar with integration testing, it is a testing approach that
allows you to test multiple units in concert. The integration testing
features in CakePHP simulate an HTTP request being handled by your application.
For example, testing your controller will also exercise any components, models
and helpers that would be involved in handling a given request. This gives you a
more high level test of your application and all its working parts.

Say you have a typical ArticlesController, and its corresponding model. The
controller code looks like::

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
                    // Redirect as per PRG pattern
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

Create a file named **ArticlesControllerTest.php** in your
**tests/TestCase/Controller** directory and put the following inside::

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
            // More asserts.
        }

        public function testIndexQueryData(): void
        {
            $this->get('/articles?page=1');

            $this->assertResponseOk();
            // More asserts.
        }

        public function testIndexShort(): void
        {
            $this->get('/articles/index/short');

            $this->assertResponseOk();
            $this->assertResponseContains('Articles');
            // More asserts.
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

This example shows a few of the request sending methods and a few of the
assertions that ``IntegrationTestTrait`` provides. Before you can do any
assertions you'll need to dispatch a request. You can use one of the following
methods to send a request:

* ``get()`` Sends a GET request.
* ``post()`` Sends a POST request.
* ``put()`` Sends a PUT request.
* ``delete()`` Sends a DELETE request.
* ``patch()`` Sends a PATCH request.
* ``options()`` Sends an OPTIONS request.
* ``head()`` Sends a HEAD request.

All of the methods except ``get()`` and ``delete()`` accept a second parameter
that allows you to send a request body. After dispatching a request you can use
the various assertions provided by ``IntegrationTestTrait`` or PHPUnit to
ensure your request had the correct side-effects.

Setting up the Request
----------------------

The ``IntegrationTestTrait`` trait comes with a number of helpers to
to configure the requests you will send to your application under test::

    // Set cookies
    $this->cookie('name', 'Uncle Bob');

    // Set session data
    $this->session(['Auth.User.id' => 1]);

    // Configure headers
    $this->configRequest([
        'headers' => ['Accept' => 'application/json']
    ]);

The state set by these helper methods is reset in the ``tearDown()`` method.

Testing Actions Protected by CsrfComponent or SecurityComponent
---------------------------------------------------------------

When testing actions protected by either SecurityComponent or CsrfComponent you
can enable automatic token generation to ensure your tests won't fail due to
token mismatches::

    public function testAdd(): void
    {
        $this->enableCsrfToken();
        $this->enableSecurityToken();
        $this->post('/posts/add', ['title' => 'Exciting news!']);
    }

It is also important to enable debug in tests that use tokens to prevent the
SecurityComponent from thinking the debug token is being used in a non-debug
environment. When testing with other methods like ``requireSecure()`` you
can use ``configRequest()`` to set the correct environment variables::

    // Fake out SSL connections.
    $this->configRequest([
        'environment' => ['HTTPS' => 'on']
    ]);

If your action requires unlocked fields you can declare them with
``setUnlockedFields()``::

    $this->setUnlockedFields(['dynamic_field']);

Integration Testing PSR-7 Middleware
------------------------------------

Integration testing can also be used to test your entire PSR-7 application and
:doc:`/controllers/middleware`. By default ``IntegrationTestTrait`` will
auto-detect the presence of an ``App\Application`` class and automatically
enable integration testing of your Application.

You can customize the application class name used, and the constructor
arguments, by using the ``configApplication()`` method::

    public function setUp(): void
    {
        $this->configApplication('App\App', [CONFIG]);
    }

You should also take care to try and use :ref:`application-bootstrap` to load
any plugins containing events/routes. Doing so will ensure that your
events/routes are connected for each test case. Alternatively if you wish to
load plugins manually in a test you can use the ``loadPlugins()`` method.

Testing with Encrypted Cookies
------------------------------

If you use the :ref:`encrypted-cookie-middleware` in your
application, there are helper methods for setting encrypted cookies in your
test cases::

    // Set a cookie using AES and the default key.
    $this->cookieEncrypted('my_cookie', 'Some secret values');

    // Assume this action modifies the cookie.
    $this->get('/articles/index');

    $this->assertCookieEncrypted('An updated value', 'my_cookie');

Testing Flash Messages
----------------------

If you want to assert the presence of flash messages in the session and not the
rendered HTML, you can use ``enableRetainFlashMessages()`` in your tests to
retain flash messages in the session so you can write assertions::

    // Enable retention of flash messages instead of consuming them.
    $this->enableRetainFlashMessages();
    $this->get('/articles/delete/9999');

    $this->assertSession('That article does not exist', 'Flash.flash.0.message');

    // Assert a flash message in the 'flash' key.
    $this->assertFlashMessage('Article deleted', 'flash');

    // Assert the second flash message, also  in the 'flash' key.
    $this->assertFlashMessageAt(1, 'Article really deleted');

    // Assert a flash message in the 'auth' key at the first position
    $this->assertFlashMessageAt(0, 'You are not allowed to enter this dungeon!', 'auth');

    // Assert a flash messages uses the error element
    $this->assertFlashElement('Flash/error');

    // Assert the second flash message element
    $this->assertFlashElementAt(1, 'Flash/error');

Testing a JSON Responding Controller
------------------------------------

JSON is a friendly and common format to use when building a web service.
Testing the endpoints of your web service is very simple with CakePHP. Let us
begin with a simple example controller that responds in JSON::

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

Now we create the file **tests/TestCase/Controller/MarkersControllerTest.php**
and make sure our web service is returning the proper response::

    class MarkersControllerTest extends IntegrationTestCase
    {
        use IntegrationTestTrait;

        public function testGet(): void
        {
            $this->configRequest([
                'headers' => ['Accept' => 'application/json']
            ]);
            $this->get('/markers/view/1.json');

            // Check that the response was a 200
            $this->assertResponseOk();

            $expected = [
                ['id' => 1, 'lng' => 66, 'lat' => 45],
            ];
            $expected = json_encode($expected, JSON_PRETTY_PRINT);
            $this->assertEquals($expected, (string)$this->_response->getBody());
        }
    }

We use the ``JSON_PRETTY_PRINT`` option as CakePHP's built in JsonView will use
that option when ``debug`` is enabled.

Testing with file uploads
-------------------------

Simulating file uploads is straightforward when you use the default
":ref:`uploaded files as objects <request-file-uploads>`" mode. You can simply
create instances that implement
`\\Psr\\Http\\Message\\UploadedFileInterface <https://www.php-fig.org/psr/psr-7/#16-uploaded-files>`__
(the default implementation currently used by CakePHP is
``\Laminas\Diactoros\UploadedFile``), and pass them in your test request data.
In the CLI environment such objects will by default pass validation checks that
test whether the file was uploaded via HTTP. The same is not true for array style
data as found in ``$_FILES``, it would fail that check.

In order to simulate exactly how the uploaded file objects would be present on
a regular request, you not only need to pass them in the request data, but you also
need to pass them to the test request configuration via the ``files`` option. It's
not technically necessary though unless your code accesses uploaded files via the
:php:meth:`Cake\\Http\\ServerRequest::getUploadedFile()` or
:php:meth:`Cake\\Http\\ServerRequest::getUploadedFiles()` methods.

Let's assume articles have a teaser image, and a ``Articles hasMany Attachments``
association, the form would look like something like this accordingly, where one
image file, and multiple attachments/files would be accepted::

    <?= $this->Form->create($article, ['type' => 'file']) ?>
    <?= $this->Form->control('title') ?>
    <?= $this->Form->control('teaser_image', ['type' => 'file']) ?>
    <?= $this->Form->control('attachments.0.attachment', ['type' => 'file']) ?>
    <?= $this->Form->control('attachments.0.description']) ?>
    <?= $this->Form->control('attachments.1.attachment', ['type' => 'file']) ?>
    <?= $this->Form->control('attachments.1.description']) ?>
    <?= $this->Form->button('Submit') ?>
    <?= $this->Form->end() ?>

The test that would simulate the corresponding request could look like this::

    public function testAddWithUploads(): void
    {
        $teaserImage = new \Laminas\Diactoros\UploadedFile(
            '/path/to/test/file.jpg', // stream or path to file representing the temp file
            12345,                    // the filesize in bytes
            \UPLOAD_ERR_OK,           // the upload/error status
            'teaser.jpg',             // the filename as sent by the client
            'image/jpeg'              // the mimetype as sent by the client
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

        // This is the data accessible via `$this->request->getUploadedFile()`
        // and `$this->request->getUploadedFiles()`.
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

        // This is the data accessible via `$this->request->getData()`.
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

    If you configure the test request with files, then it *must* match the
    structure of your POST data (but only include the uploaded file objects)!

Likewise you can simulate `upload errors <https://www.php.net/manual/en/features.file-upload.errors.php>`_
or otherwise invalid files that do not pass validation::

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

Disabling Error Handling Middleware in Tests
--------------------------------------------

When debugging tests that are failing because your application is encountering
errors it can be helpful to temporarily disable the error handling middleware to
allow the underlying error to bubble up. You can use
``disableErrorHandlerMiddleware()`` to do this::

    public function testGetMissing(): void
    {
        $this->disableErrorHandlerMiddleware();
        $this->get('/markers/not-there');
        $this->assertResponseCode(404);
    }

In the above example, the test would fail and the underlying exception message
and stack trace would be displayed instead of the rendered error page being
checked.

Assertion methods
-----------------

The ``IntegrationTestTrait`` trait provides a number of assertion methods that
make testing responses much simpler. Some examples are::

    // Check for a 2xx response code
    $this->assertResponseOk();

    // Check for a 2xx/3xx response code
    $this->assertResponseSuccess();

    // Check for a 4xx response code
    $this->assertResponseError();

    // Check for a 5xx response code
    $this->assertResponseFailure();

    // Check for a specific response code, for example, 200
    $this->assertResponseCode(200);

    // Check the Location header
    $this->assertRedirect(['controller' => 'Articles', 'action' => 'index']);

    // Check that no Location header has been set
    $this->assertNoRedirect();

    // Check a part of the Location header
    $this->assertRedirectContains('/articles/edit/');

    // Assert location header does not contain
    $this->assertRedirectNotContains('/articles/edit/');

    // Assert not empty response content
    $this->assertResponseNotEmpty();

    // Assert empty response content
    $this->assertResponseEmpty();

    // Assert response content
    $this->assertResponseEquals('Yeah!');

    // Assert response content doesn't equal
    $this->assertResponseNotEquals('No!');

    // Assert partial response content
    $this->assertResponseContains('You won!');
    $this->assertResponseNotContains('You lost!');

    // Assert file sent back
    $this->assertFileResponse('/absolute/path/to/file.ext');

    // Assert layout
    $this->assertLayout('default');

    // Assert which template was rendered (if any)
    $this->assertTemplate('index');

    // Assert data in the session
    $this->assertSession(1, 'Auth.User.id');

    // Assert response header.
    $this->assertHeader('Content-Type', 'application/json');
    $this->assertHeaderContains('Content-Type', 'html');

    // Assert content-type header doesn't contain xml
    $this->assertHeaderNotContains('Content-Type', 'xml');

    // Assert view variables
    $user =  $this->viewVariable('user');
    $this->assertEquals('jose', $user->username);

    // Assert cookie values in the response
    $this->assertCookie('1', 'thingid');

    // Assert a cookie is or is not present
    $this->assertCookieIsSet('remember_me');
    $this->assertCookieNotSet('remember_me');

    // Check the content type
    $this->assertContentType('application/json');

In addition to the above assertion methods, you can also use all of the
assertions in `TestSuite
<https://api.cakephp.org/5.x/class-Cake.TestSuite.TestCase.html>`_ and those
found in `PHPUnit
<https://phpunit.de/manual/current/en/appendixes.assertions.html>`__.

Comparing test results to a file
--------------------------------

For some types of test, it may be easier to compare the result of a test to the
contents of a file - for example, when testing the rendered output of a view.
The ``StringCompareTrait`` adds a simple assert method for this purpose.

Usage involves using the trait, setting the comparison base path and calling
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

The above example will compare ``$result`` to the contents of the file
``APP/tests/comparisons/example.php``.

A mechanism is provided to write/update test files, by setting the environment
variable ``UPDATE_TEST_COMPARISON_FILES``, which will create and/or update test
comparison files as they are referenced:

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


Console Integration Testing
===========================

See :ref:`console-integration-testing` for how to test console commands.

Mocking Injected Dependencies
=============================

See :ref:`mocking-services-in-tests` for how to replace services injected with
the dependency injection container in your integration tests.

Mocking HTTP Client Responses
=============================

See :ref:`httpclient-testing` to know how to create mock responses to external APIs.

Testing Views
=============

Generally most applications will not directly test their HTML code. Doing so is
often results in fragile, difficult to maintain test suites that are prone to
breaking. When writing functional tests using :php:class:`IntegrationTestTrait`
you can inspect the rendered view content by setting the ``return`` option to
'view'. While it is possible to test view content using ``IntegrationTestTrait``,
a more robust and maintainable integration/view testing can be accomplished
using tools like `Selenium webdriver <https://www.selenium.dev/>`_.

Testing Components
==================

Let's pretend we have a component called PagematronComponent in our application.
This component helps us set the pagination limit value across all the
controllers that use it. Here is our example component located in
**src/Controller/Component/PagematronComponent.php**::

    class PagematronComponent extends Component
    {
        public $controller = null;

        public function setController($controller)
        {
            $this->controller = $controller;
            // Make sure the controller is using pagination
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

Now we can write tests to ensure our paginate ``limit`` parameter is being set
correctly by the ``adjust()`` method in our component. We create the file
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
            // Setup our component and fake test controller
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
            // Test our adjust method with different parameter settings
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
            // Clean up after we're done
            unset($this->component, $this->controller);
        }
    }

Testing Helpers
===============

Since a decent amount of logic resides in Helper classes, it's
important to make sure those classes are covered by test cases.

First we create an example helper to test. The ``CurrencyRendererHelper`` will
help us display currencies in our views and for simplicity only has one method
``usd()``::

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

Here we set the decimal places to 2, decimal separator to dot, thousands
separator to comma, and prefix the formatted number with 'USD' string.

Now we create our tests::

    // tests/TestCase/View/Helper/CurrencyRendererHelperTest.php

    namespace App\Test\TestCase\View\Helper;

    use App\View\Helper\CurrencyRendererHelper;
    use Cake\TestSuite\TestCase;
    use Cake\View\View;

    class CurrencyRendererHelperTest extends TestCase
    {
        public $helper = null;

        // Here we instantiate our helper
        public function setUp(): void
        {
            parent::setUp();
            $View = new View();
            $this->helper = new CurrencyRendererHelper($View);
        }

        // Testing the usd() function
        public function testUsd(): void
        {
            $this->assertEquals('USD 5.30', $this->helper->usd(5.30));

            // We should always have 2 decimal digits
            $this->assertEquals('USD 1.00', $this->helper->usd(1));
            $this->assertEquals('USD 2.05', $this->helper->usd(2.05));

            // Testing the thousands separator
            $this->assertEquals(
              'USD 12,000.70',
              $this->helper->usd(12000.70)
            );
        }
    }

Here, we call ``usd()`` with different parameters and tell the test suite to
check if the returned values are equal to what is expected.

Save this and execute the test. You should see a green bar and messaging
indicating 1 pass and 4 assertions.

When you are testing a Helper which uses other helpers, be sure to mock the
View clases ``loadHelpers`` method.

.. _testing-events:

Testing Events
==============

The :doc:`/core-libraries/events` is a great way to decouple your application
code, but sometimes when testing, you tend to test the results of events in the
test cases that execute those events. This is an additional form of coupling
that can be removed by using ``assertEventFired`` and ``assertEventFiredWith``
instead.

Expanding on the Orders example, say we have the following tables::

    class OrdersTable extends Table
    {
        public function place($order): bool
        {
            if ($this->save($order)) {
                // moved cart removal to CartsTable
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
    To assert that events are fired, you must first enable
    :ref:`tracking-events` on the event manager you wish to assert against.

To test the ``OrdersTable`` above, we enable tracking in ``setUp()`` then assert
that the event was fired, and assert that the ``$order`` entity was passed in
the event data::

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
            // enable event tracking
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

By default, the global ``EventManager`` is used for assertions, so testing
global events does not require passing the event manager::

    $this->assertEventFired('My.Global.Event');
    $this->assertEventFiredWith('My.Global.Event', 'user', 1);

Testing Email
=============

See :ref:`email-testing` for information on testing email.

Creating Test Suites
====================

If you want several of your tests to run at the same time, you can create a test
suite. A test suite is composed of several test cases.  You can either create
test suites in your application's **phpunit.xml** file. A simple example
would be:

.. code-block:: xml

    <testsuites>
      <testsuite name="Models">
        <directory>src/Model</directory>
        <file>src/Service/UserServiceTest.php</file>
        <exclude>src/Model/Cloud/ImagesTest.php</exclude>
      </testsuite>
    </testsuites>

Creating Tests for Plugins
==========================

Tests for plugins are created in their own directory inside the plugins
folder. ::

    /src
    /plugins
        /Blog
            /tests
                /TestCase
                /Fixture

They work just like normal tests but you have to remember to use the naming
conventions for plugins when importing classes. This is an example of a testcase
for the ``BlogPost`` model from the plugins chapter of this manual. A difference
from other tests is in the first line where 'Blog.BlogPost' is imported. You
also need to prefix your plugin fixtures with ``plugin.Blog.BlogPosts``::

    namespace Blog\Test\TestCase\Model\Table;

    use Blog\Model\Table\BlogPostsTable;
    use Cake\TestSuite\TestCase;

    class BlogPostsTableTest extends TestCase
    {
        // Plugin fixtures located in /plugins/Blog/tests/Fixture/
        protected $fixtures = ['plugin.Blog.BlogPosts'];

        public function testSomething(): void
        {
            // Test something.
        }
    }

If you want to use plugin fixtures in the app tests you can
reference them using ``plugin.pluginName.fixtureName`` syntax in the
``$fixtures`` array. Additionally if you use vendor plugin name or fixture
directories you can use the following: ``plugin.vendorName/pluginName.folderName/fixtureName``.

Before you can use fixtures you should ensure you have the :ref:`fixture
listener <fixture-phpunit-configuration>` configured in your ``phpunit.xml``
file. You should also ensure that your fixtures are loadable. Ensure the
following is present in your **composer.json** file::

    "autoload-dev": {
        "psr-4": {
            "MyPlugin\\Test\\": "plugins/MyPlugin/tests/"
        }
    }

.. note::

    Remember to run ``composer.phar dumpautoload`` when adding new autoload
    mappings.

Generating Tests with Bake
==========================

If you use :doc:`bake </bake/usage>` to
generate scaffolding, it will also generate test stubs. If you need to
re-generate test case skeletons, or if you want to generate test skeletons for
code you wrote, you can use ``bake``:

.. code-block:: console

    bin/cake bake test <type> <name>

``<type>`` should be one of:

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

While ``<name>`` should be the name of the object you want to bake a test
skeleton for.

.. meta::
    :title lang=en: Testing
    :keywords lang=en: phpunit,test database,database configuration,database setup,database test,public test,test framework,running one,test setup,de facto standard,pear,runners,array,databases,cakephp,php,integration
