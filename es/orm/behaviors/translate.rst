Translate
#########

.. php:namespace:: Cake\ORM\Behavior

.. php:class:: TranslateBehavior

El behavior translate te permite crear y recuperar copias de tus entidades en varios idiomas.

.. warning::

    TranslateBehavior no admite claves primarias compuestas en este momento.

Estrategias de traducción
=========================

El behavior ofrece dos estrategias para almacenar traducciones.

1. Estrategia de tabla sombra: Esta estrategia utiliza una "tabla sombra" independientep para cada
   objeto Tabla para almacenar la traducción de todos los campos traducidos de esa tabla.
   Es la estrategia por defecto.
2. Estrategia Eav: Esta estrategia utiliza una tabla ``i18n`` donde almacena la traducción de
   cada uno de los campos de cualquier objeto Table al que esté vinculada.

Estrategia de tabla sombra
==========================

Supongamos que tenemos una tabla ``articles`` y queremos traducir sus campos ``title`` y ``body``.
Para ello creamos una tabla sombra ``articles_translations``:

.. code-block:: sql

    CREATE TABLE `articles_translations` (
        `id` int(11) NOT NULL,
        `locale` varchar(5) NOT NULL,
        `title` varchar(255) NOT NULL,
        `body` text NOT NULL,
        PRIMARY KEY (`id`,`locale`)
    );

La tabla sombra necesita las columnas ``id`` y ``locale`` que juntas
forman la clave primaria y otras columnas con el mismo nombre que la tabla primaria
que necesitan ser traducidas.

Nota sobre las abreviaturas de idiomas: El behavior Translate no impone ninguna restricción
restrictions on the language identifier, los posibles valores sólo están restringidos
por el tipo/tamaño de la columna ``locale``. ``locale`` se define como ``varchar(6)`` en
caso de que desee utilizar abreviaturas como ``es-419`` (Español para América Latina,
abreviatura del idioma con el prefijo `UN M.49 <https://en.wikipedia.org/wiki/UN_M.49>`_).

.. tip::

    Es aconsejable utilizar las mismas abreviaturas lingüísticas que se requieren para
    :doc:`Internacionalización y localización </core-libraries/internationalization-and-localization>`.
    De esta forma eres consistente y cambiar el idioma funciona idéntico para ambos,
    el behavior ``Translate`` y ``Internacionalización y localización``.

Por eso se recomienda utilizar el código ISO de dos letras del idioma como
``en``, ``fr``, ``de`` o el nombre completo de la localización como ``fr_FR``, ``es_AR``,
``da_DK``, que contiene tanto el idioma como el país donde se habla.

Estrategia Eav
==============

Para utilizar la estrategia Eav, es necesario crear una tabla ``i18n``
con el esquema correcto. Actualmente la única forma de cargar la tabla ``i18n``
es ejecutando manualmente el siguiente script SQL en tu base de datos:

.. code-block:: sql

    CREATE TABLE i18n (
        id int NOT NULL auto_increment,
        locale varchar(6) NOT NULL,
        model varchar(255) NOT NULL,
        foreign_key int(10) NOT NULL,
        field varchar(255) NOT NULL,
        content text,
        PRIMARY KEY	(id),
        UNIQUE INDEX I18N_LOCALE_FIELD(locale, model, foreign_key, field),
        INDEX I18N_FIELD(model, foreign_key, field)
    );

El esquema también está disponible como archivo sql en **/config/schema/i18n.sql**.

Adjuntar el behavior Translate a las tablas
===========================================

Adjuntar el behavior se puede hacer en el método ``initialize()`` en su clase Tabla::

    class ArticlesTable extends Table
    {
        public function initialize(array $config): void
        {
            // Por defecto se utilizará ShadowTable.
            $this->addBehavior('Translate', ['fields' => ['title', 'body']]);
        }
    }

Para la estrategia de tabla sombra especificar la clave ``fields`` es opcional ya que
el comportamiento puede inferir los campos a partir de las columnas de la tabla sombra.

Si quieres usar el ``EavStrategy`` entonces tienes que configurar el comportamiento como::

    class ArticlesTable extends Table
    {
        public function initialize(array $config): void
        {
            $this->addBehavior('Translate', [
                'strategyClass' => \Cake\ORM\Behavior\Translate\EavStrategy::class,
                'fields' => ['title', 'body'],
            ]);
        }
    }

Para ``EavStrategy`` es necesario pasar la clave ``fields`` en el array de configuración.
Esta lista de campos es necesaria para indicar al comportamiento qué columnas podrán almacenar traducciones.

Por defecto, la configuración regional especificada en ``App.defaultLocale``
se utiliza como configuración regional por defecto para el ``TranslateBehavior``.
Puedes sobrescribir esta configuración cuando se establece el behavior mediante la configuración ``defaultLocale``
como se muestra a continuación::

    class ArticlesTable extends Table
    {
        public function initialize(array $config): void
        {
            $this->addBehavior('Translate', [
                'defaultLocale' => 'en_GB',
            ]);
        }
    }

Recorrido rápido
================

Independientemente de la estrategia de estructura de datos que elijas,
el behavior proporciona la misma API para administrar las traducciones.

Now, select a language to be used for retrieving entities by changing the application language, which will affect all translations::
Ahora, selecciona el idioma que se usará para recuperar entidades cambiando el idioma de la aplicación,
lo que afectará a todas las traducciones::

    // En el controlador de artículos. Cambia la configuración regional a español, por ejemplo
    I18n::setLocale('es');

A continuación, obtén una entidad existente::

    $article = $this->Articles->get(12);
    echo $article->title; // Echoes 'A title', not translated yet

A continuación, traduce tu entidad::

    $article->title = 'Un Artículo';
    $this->Articles->save($article);

Ahora intenta obtener tu entidad nuevamente::

    $article = $this->Articles->get(12);
    echo $article->title; // Echoes 'Un Artículo', yay piece of cake!

Trabajar con varias traducciones se puede realizar mediante el uso de un trait especial en tu clase Entity::

    use Cake\ORM\Behavior\Translate\TranslateTrait;
    use Cake\ORM\Entity;

    class Article extends Entity
    {
        use TranslateTrait;
    }

Ahora puedes encontrar todas las traducciones de una sola entidad::

    $article = $this->Articles->find('translations')->first();
    echo $article->translation('es')->title; // 'Un Artículo'

    echo $article->translation('en')->title; // 'An Article';

Y guardar varias traducciones a la vez::

    $article->translation('es')->title = 'Otro Título';
    $article->translation('fr')->title = 'Un autre Titre';
    $this->Articles->save($article);

Si quieres profundizar en cómo funciona o cómo ajustar el comportamiento a tus necesidades,
sigue leyendo el resto de este capítulo.


Uso de una tabla de traducciones separada para la estrategia Eav
----------------------------------------------------------------

Si deseas usar una tabla que no sea ``i18n`` para traducir un repositorio en particular,
puedes especificar el nombre de la clase de tabla para su tabla personalizada en la
configuración del behavior. Esto es común cuando tiene varias tablas para traducir
y desea una separación más limpia de los datos que se almacenan
para cada tabla diferente::

    class ArticlesTable extends Table
    {
        public function initialize(array $config): void
        {
            $this->addBehavior('Translate', [
                'fields' => ['title', 'body'],
                'translationTable' => 'ArticlesI18n',
            ]);
        }
    }

Debe asegurarse de que cualquier tabla personalizada que utilice tenga
las columnas ``field``, ``foreign_key``, ``locale`` y ``model``.

Lectura de contenido traducido
==============================

Como se muestra arriba, puedes usar el método ``setLocale()`` para elegirla traducción activa
para las entidades que se cargan::

    // Cargue las funciones de I18n al principio de tu controlador de artículos:
    use Cake\I18n\I18n;

    // A continuación, puedes cambiar el idioma en tu acción(action):
    I18n::setLocale('es');

    // Todas los resultados de las entidades contendrán traducción al español
    $results = $this->Articles->find()->all();

Este método funciona con cualquier buscador de tus tablas. Por ejemplo,
puedes usar TranslateBehavior con ``find('list')``::

    I18n::setLocale('es');
    $data = $this->Articles->find('list')->toArray();

    // Los datos contendrán
    [1 => 'Mi primer artículo', 2 => 'El segundo artículo', 15 => 'Otro articulo' ...]

    // Cambiar la configuración regional a francés para una única llamada de búsqueda
    $data = $this->Articles->find('list', locale: 'fr')->toArray();

Recuperar todas las traducciones de una entidad
-----------------------------------------------

Cuando se construyen interfaces para actualizar contenidos traducidos,
a menudo es útil mostrar una o más traducciones al mismo tiempo.
Para ello, puedes utilizar el buscador de ``translations``::

    // Buscar el primer artículo con todas las traducciones correspondientes
    $article = $this->Articles->find('translations')->first();

En el ejemplo anterior obtendrás una lista de entidades que tienen la propiedad
``_translations``. Esta propiedad contendrá una lista de entidades de datos de traducción.
Por ejemplo, las siguientes propiedades serían accesibles::

    // Outputs 'en'
    echo $article->_translations['en']->locale;

    // Outputs 'title'
    echo $article->_translations['en']->field;

    // Outputs 'My awesome post!'
    echo $article->_translations['en']->body;

Una forma más elegante de tratar estos datos es añadiendo un trait a la clase de entidad que se utiliza para su tabla::

    use Cake\ORM\Behavior\Translate\TranslateTrait;
    use Cake\ORM\Entity;

    class Article extends Entity
    {
        use TranslateTrait;
    }

Este trait contiene un único método llamado ``translation``, que te permite acceder o
crear nuevas entidades de traducción sobre la marcha::

    // Outputs 'title'
    echo $article->translation('en')->title;

    // Añade una nueva entidad de datos de traducción al artículo
    $article->translation('de')->title = 'Wunderbar';

Limitar las traducciones a recuperar
------------------------------------

Puedes limitar los idiomas que se obtienen de la base de datos
para un determinado conjunto de registros::

    $results = $this->Articles->find('translations', locales: ['en', 'es']);
    $article = $results->first();
    $spanishTranslation = $article->translation('es');
    $englishTranslation = $article->translation('en');

Impedir la recuperación de traducciones vacías
----------------------------------------------

Los registros de traducción pueden contener cualquier cadena, si un registro ha sido traducido
y almacenado como una cadena vacía ('') el behavior translate tomará y usará
esto para sobrescribir el valor del campo original.

Si no lo deseas, puedes ignorar las traducciones vacías mediante
la clave de configuración ``allowEmptyTranslations``::

    class ArticlesTable extends Table
    {
        public function initialize(array $config): void
        {
            $this->addBehavior('Translate', [
                'fields' => ['title', 'body'],
                'allowEmptyTranslations' => false
            ]);
        }
    }

Lo anterior sólo cargaría los datos traducidos que tuvieran contenido.

Recuperar todas las traducciones de las asociaciones
----------------------------------------------------

También es posible encontrar traducciones para cualquier asociación en una única operación de búsqueda::

    $article = $this->Articles->find('translations')->contain([
        'Categories' => function ($query) {
            return $query->find('translations');
        }
    ])->first();

    // Outputs 'Programación'
    echo $article->categories[0]->translation('es')->name;

Esto asume que ``Categories`` tiene el TranslateBehavior asociado.
Simplemente utiliza la función query builder de la cláusula ``contain``
para utilizar el buscador personalizado ``translations`` en la asociación.

.. _retrieving-one-language-without-using-i18n-locale:

Recuperar un idioma sin utilizar I18n::setLocale
------------------------------------------------

llamar a ``I18n::setLocale('es');`` cambia la configuración regional por defecto para todos los hallazgos traducidos,
puede haber ocasiones en las que desees recuperar contenido traducido sin modificar el estado de la aplicación.
Para estos casos utiliza el método ``setLocale()`` del behavior::

    I18n::setLocale('en'); // reset for illustration

    // specific locale.
    $this->Articles->setLocale('es');

    $article = $this->Articles->get(12);
    echo $article->title; // Echoes 'Un Artículo', yay piece of cake!

Ten en cuenta que esto sólo cambia la configuración regional de la tabla Artículos,
no afectaría al idioma de los datos asociados. Para afectar a los datos asociados
es necesario llamar al método en cada tabla, por ejemplo::

    I18n::setLocale('en'); // reset for illustration

    $this->Articles->setLocale('es');
    $this->Articles->Categories->setLocale('es');

    $data = $this->Articles->find('all', contain: ['Categories']);

Este ejemplo también asume que ``Categories`` tiene el TranslateBehavior cargado.

Consulta de campos traducidos
-----------------------------

TranslateBehavior no sustituye las condiciones de búsqueda por defecto.
Es necesario utilizar el método ``translationField()`` para componer las condiciones de búsqueda en los campos traducidos::

    $this->Articles->setLocale('es');
    $query = $this->Articles->find()->where([
        $this->Articles->translationField('title') => 'Otro Título'
    ]);

Guardar en otro idioma
======================

La filosofía detrás de TranslateBehavior es que usted tiene una entidad
que representa el idioma por defecto, y múltiples traducciones que pueden sobrescribir
ciertos campos en dicha entidad. Teniendo esto en cuenta, puedes guardar intuitivamente
traducciones para cualquier entidad dada. Por ejemplo, dada la siguiente configuración::

    // in src/Model/Table/ArticlesTable.php
    class ArticlesTable extends Table
    {
        public function initialize(array $config): void
        {
            $this->addBehavior('Translate', ['fields' => ['title', 'body']]);
        }
    }

    // in src/Model/Entity/Article.php
    class Article extends Entity
    {
        use TranslateTrait;
    }

    // In the Articles Controller
    $article = new Article([
        'title' => 'My First Article',
        'body' => 'This is the content',
        'footnote' => 'Some afterwords'
    ]);

    $this->Articles->save($article);

Así que, después de guardar tu primer artículo, ahora puedes guardar una traducción para él,
hay un par de maneras de hacerlo. La primera es establecer el idioma directamente en la entidad::

    $article->_locale = 'es';
    $article->title = 'Mi primer Artículo';

    $this->Articles->save($article);

Una vez guardada la entidad, el campo traducido también se conservará.
Hay que tener en cuenta que los valores del idioma por defecto que no se hayan modificado
se conservarán::

    // Outputs 'This is the content'
    echo $article->body;

    // Outputs 'Mi primer Artículo'
    echo $article->title;

Una vez modificado el valor, la traducción de ese campo se guardará y podrá recuperarse como de costumbre::

    $article->body = 'El contenido';
    $this->Articles->save($article);

La segunda forma de utilizar para guardar entidades en otro idioma es establecer el
idioma por defecto directamente en la tabla::

    $article->title = 'Mi Primer Artículo';

    $this->Articles->setLocale('es');
    $this->Articles->save($article);

Establecer el idioma directamente en la tabla es útil cuando necesitas
recuperar y guardar entidades para el mismo idioma o cuando necesitas
guardar varias entidades a la vez.

.. _saving-multiple-translations:

Guardar varias traducciones
============================

Es un requisito común poder añadir o editar múltiples traducciones a cualquier registro
de la base de datos al mismo tiempo. Esto se puede hacer utilizando el ``TranslateTrait``::

    use Cake\ORM\Behavior\Translate\TranslateTrait;
    use Cake\ORM\Entity;

    class Article extends Entity
    {
        use TranslateTrait;
    }

Ahora, puedes rellenar las traducciones antes de guardarlas::

    $translations = [
        'fr' => ['title' => "Un article"],
        'es' => ['title' => 'Un artículo'],
    ];

    foreach ($translations as $lang => $data) {
        $article->translation($lang)->set($data, ['guard' => false]);
    }

    $this->Articles->save($article);

Y crear controles de formulario para tus campos traducidos::

    // In a view template.
    <?= $this->Form->create($article); ?>
    <fieldset>
        <legend>French</legend>
        <?= $this->Form->control('_translations.fr.title'); ?>
        <?= $this->Form->control('_translations.fr.body'); ?>
    </fieldset>
    <fieldset>
        <legend>Spanish</legend>
        <?= $this->Form->control('_translations.es.title'); ?>
        <?= $this->Form->control('_translations.es.body'); ?>
    </fieldset>

En tu controlador, puedes serializar los datos de la forma habitual::

    $article = $this->Articles->newEntity($this->request->getData());
    $this->Articles->save($article);

Esto hará que tu artículo y las traducciones al francés y al español se mantengan.
Tendrás que acordarte de añadir ``_translations`` en los campos ``$_accessible`` de tu entidad.

Validación de entidades traducidas
----------------------------------

Cuando se carga ``TranslateBehavior`` a un modelo, se puede definir el validador
que se debe utilizar cuando los registros de traducción son creados/modificados por el
comportamiento durante ``newEntity()`` o ``patchEntity()``::

    class ArticlesTable extends Table
    {
        public function initialize(array $config): void
        {
            $this->addBehavior('Translate', [
                'fields' => ['title'],
                'validator' => 'translated',
            ]);
        }
    }

Lo anterior utilizará el validador creado por ``validationTranslated`` para validar las entidades traducidas.

.. meta::
    :title lang=es: Translate
    :keywords lang=es: maintenance branch,community interaction,community feature,necessary feature,stable release,ticket system,advanced feature,power users,feature set,chat irc,leading edge,router,new features,members,attempt,development branches,branch development
