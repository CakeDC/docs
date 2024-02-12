Internacionalización y Localización
###################################

Una de las mejores formas para que una aplicación llegue a una audiencia más amplia es adaptarse a varios idiomas. Esto a menudo puede resultar una tarea desafiante, pero las características de internacionalización y localización en CakePHP hacen que sea mucho más fácil.

Primero, es importante entender algunos términos. *Internacionalización* se refiere a la capacidad de una aplicación de ser localizada. El término *localización* se refiere a la adaptación de una aplicación para cumplir con requisitos específicos de idioma (o cultura) (es decir, una "configuración regional"). La internacionalización y la localización a menudo se abrevian como i18n y l10n respectivamente; 18 y 10 son el número de caracteres entre el primer y el último caracter.

Configuración de Traducciones
=============================

Solo hay algunos pasos para pasar de una aplicación de un solo idioma a una aplicación multilingüe, el primero de los cuales es hacer uso de la función :php:func:`__()` en tu código. A continuación se muestra un ejemplo de código para una aplicación de un solo idioma:

    <h2>Artículos Populares</h2>

Para internacionalizar tu código, todo lo que necesitas hacer es envolver las cadenas en :php:func:`__()` de la siguiente manera:

    <h2><?= __('Artículos Populares') ?></h2>

Sin hacer nada más, estos dos ejemplos de código son funcionalmente idénticos: ambos enviarán el mismo contenido al navegador. La función :php:func:`__()` traducirá la cadena pasada si hay una traducción disponible, o la devolverá sin modificar.

Archivos de Idioma
------------------

Las traducciones pueden estar disponibles utilizando archivos de idioma almacenados en la aplicación. El formato predeterminado para los archivos de traducción de CakePHP es el formato `Gettext <https://es.wikipedia.org/wiki/Gettext>`_. Los archivos deben colocarse bajo **resources/locales/** y dentro de este directorio, debería haber un subdirectorio para cada idioma que la aplicación necesite soportar:

    resources/
        locales/
            en_US/
                default.po
            en_GB/
                default.po
                validation.po
            es/
                default.po

El dominio predeterminado es 'default', por lo tanto, la carpeta de la configuración regional debe contener al menos el archivo **default.po** como se muestra arriba. Un dominio se refiere a cualquier agrupación arbitraria de mensajes de traducción. Cuando no se usa ningún grupo, se selecciona el grupo predeterminado.

Los mensajes de cadenas centrales extraídos de la biblioteca CakePHP pueden almacenarse por separado en un archivo llamado **cake.po** en **resources/locales/**. La `biblioteca localizada de CakePHP <https://github.com/cakephp/localized>`_ alberga traducciones para las cadenas traducidas orientadas al cliente en el núcleo (el dominio del pastel). Para usar estos archivos, enlázalos o cópialos en su ubicación esperada: **resources/locales/<idioma>/cake.po**. Si tu configuración regional está incompleta o es incorrecta, envía un PR en este repositorio para corregirla.

Los plugins también pueden contener archivos de traducción, la convención es usar la versión con guiones bajos del nombre del plugin como el dominio para los mensajes de traducción:

    MiPlugin/
        resources/
            locales/
                fr/
                    mi_plugin.po
                    adicional.po
                de/
                    mi_plugin.po

Las carpetas de traducción pueden ser el código ISO de dos o tres letras del idioma o el nombre completo de la configuración regional de ICU como ``fr_FR``, ``es_AR``, ``da_DK`` que contiene tanto el idioma como el país donde se habla.

Consulta https://www.localeplanet.com/icu/ para obtener la lista completa de configuraciones regionales.

.. versionchanged:: 4.5.0

A partir de 4.5.0, los plugin pueden contener múltiples dominios de traducción. Usa ``MiPlugin.adicional`` para hacer referencia a los dominios del plugin.

Un archivo de traducción de ejemplo podría verse así:

.. code-block:: pot

     msgid "My name is {0}"
     msgstr "Me llamo {0}"

     msgid "I'm {0,number} years old"
     msgstr "Tengo {0,number} años"

.. nota::
    Las traducciones se almacenan en caché. ¡Asegúrate de siempre limpiar la caché después de hacer cambios en las traducciones! Puedes usar la
    :doc:`herramienta de caché </console-commands/cache>` y ejecutar, por ejemplo,
    ``bin/cake cache clear _cake_core_``, o borrar manualmente la carpeta ``tmp/cache/persistent``
    (si usas una caché basada en archivos).

Extraer Archivos Pot con I18n Shell
------------------------------------

Para crear los archivos pot a partir de `__()` y otros tipos de mensajes internacionalizados que se pueden encontrar en el código de la aplicación, puedes usar el comando i18n. Por favor, lee el :doc:`capítulo siguiente </console-commands/i18n>` para
aprender más.

Configurando la Configuración Regional Predeterminada
-----------------------------------------------------

La configuración regional predeterminada se puede establecer en tu archivo **config/app.php** configurando ``App.defaultLocale``::

    'App' => [
        ...
        'defaultLocale' => env('APP_DEFAULT_LOCALE', 'es_ES'),
        ...
    ]

Esto controlará varios aspectos de la aplicación, incluido el idioma de las traducciones predeterminadas, el formato de fecha, el formato de número y la moneda siempre que alguno de ellos se muestre utilizando las bibliotecas de localización que proporciona CakePHP.

Cambiar la Configuración Regional en Tiempo de Ejecución
--------------------------------------------------------

Para cambiar el idioma de las cadenas traducidas, puedes llamar a este método::

    use Cake\I18n\I18n;

    I18n::setLocale('de_DE');

Esto también cambiará cómo se formatean los números y las fechas cuando se usan una de las herramientas de localización.

Usando Funciones de Traducción
==============================

CakePHP proporciona varias funciones que te ayudarán a internacionalizar tu aplicación. La más utilizada es :php:func:`__()`. Esta función se utiliza para recuperar un solo mensaje de traducción o devolver la misma cadena si no se encontró ninguna traducción::

    echo __('Artículos Populares');

Si necesitas agrupar tus mensajes, por ejemplo, traducciones dentro de un plugin, puedes usar la función :php:func:`__d()` para obtener mensajes de otro dominio::

    echo __d('mi_plugin', 'Trending ahora mismo');

.. nota::

    Si deseas traducir plugins que tengan espacios de nombres de vendor, debes usar la cadena de dominio ``vendor/nombre_del_plugin``. Pero el archivo de idioma relacionado será ``plugins/<Vendor>/<NombrePlugin>/resources/locales/<configuración regional>/nombre_del_plugin.po`` dentro de la carpeta de tu plugin.

A veces, las cadenas de traducción pueden ser ambiguas para las personas que las traducen. Esto puede suceder si dos cadenas son idénticas pero se refieren a cosas diferentes. Por ejemplo, 'carta' tiene múltiples significados en inglés. Para resolver ese problema, puedes usar la función :php:func:`__x()`::

    echo __x('written communication', 'He read the first letter');

    echo __x('alphabet learning', 'He read the first letter');

El primer argumento es el contexto del mensaje y el segundo es el mensaje que se va a traducir.

.. code-block:: pot

     msgctxt "written communication"
     msgid "He read the first letter"
     msgstr "Er las den ersten Brief"

Usando Variables en Mensajes de Traducción
------------------------------------------

Las funciones de traducción te permiten interpolar variables en los mensajes utilizando marcadores especiales definidos en el mensaje mismo o en la cadena traducida::

    echo __("Hola, mi nombre es {0}, tengo {1} años", ['Sara', 12]);

Los marcadores son numéricos y corresponden a las claves en el array pasado. También puedes pasar variables como argumentos independientes a la función::

    echo __("Pequeño paso para {0}, gran salto para {1}", 'el Hombre', 'la Humanidad');

Todas las funciones de traducción admiten reemplazos de marcadores::

    __d('validación', 'El campo {0} no puede dejarse vacío', 'Nombre');

    __x('alfabeto', 'Él leyó la letra {0}', 'Z');

El carácter ``'`` (comilla simple) actúa como un código de escape en los mensajes de traducción. Cualquier variable entre comillas simples no será reemplazada y se trata como texto literal. Por ejemplo::

    __("Esta variable '{0}' será reemplazada.", 'no');

Al usar dos comillas simples adyacentes, tus variables serán reemplazadas correctamente::

    __("Esta variable ''{0}'' será reemplazada.", 'sí');

Estas funciones aprovechan el `ICU MessageFormatter <https://php.net/manual/en/messageformatter.format.php>`_
para que puedas traducir mensajes y localizar fechas, números y monedas al mismo tiempo::

    echo __(
        'Hola {0}, tu saldo el {1,date} es {2,number,currency}',
        ['Carlos', new DateTime('2014-01-13 11:12:00'), 1354.37]
    );

    // Devuelve
    Hola Carlos, tu saldo el 13 de enero de 2014, 11:12 AM es $ 1,354.37

Los números en marcadores también pueden ser formateados con un control detallado de la salida::

    echo __(
        'Has recorrido {0,number} kilómetros en {1,number,integer} semanas',
        [5423.344, 5.1]
    );

    // Devuelve
    Has recorrido 5,423.34 kilómetros en 5 semanas

    echo __('Hay {0,number,#,###} personas en la tierra', 6.1 * pow(10, 8));

    // Devuelve
    Hay 6,100,000,000 personas en la tierra

Esta es la lista de especificadores de formato que puedes colocar después de la palabra ``number``:

* ``integer``: Elimina la parte decimal
* ``currency``: Coloca el símbolo de la moneda local y redondea los decimales
* ``percent``: Formatea el número como un porcentaje

Las fechas también pueden ser formateadas usando la palabra ``date`` después del marcador numérico. Una lista de opciones adicionales sigue:

* ``short``
* ``medium``
* ``long``
* ``full``

La palabra ``time`` después del marcador numérico también es aceptada y entiende las mismas opciones que ``date``.

También puedes usar marcadores con nombres como ``{nombre}`` en las cadenas de mensaje. Cuando uses marcadores con nombres, pasa el marcador y el reemplazo en un arreglo usando pares de clave/valor, por ejemplo::

    // imprime:  ¡Hola. Mi nombre es Sara. Tengo 12 años.
    echo __("¡Hola. Mi nombre es {nombre}. Tengo {edad} años.", ['nombre' => 'Sara', 'edad' => 12]);

Plurales
--------

Una parte crucial de internacionalizar tu aplicación es obtener tus mensajes pluralizados correctamente dependiendo del idioma en el que se muestran. CakePHP proporciona un par de formas de seleccionar correctamente plurales en tus mensajes.

Usando la Selección de Plurales ICU
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

La primera es aprovechar el formato de mensaje ``ICU`` que viene por defecto en las funciones de traducción. En el archivo de traducciones podrías tener las siguientes cadenas

.. code-block:: pot

     msgid "{0,plural,=0{No se encontraron registros} =1{Se encontró 1 registro} other{Se encontraron # registros}}"
     msgstr "{0,plural,=0{Ningún resultado} =1{1 resultado} other{# resultados}}"

     msgid "{placeholder,plural,=0{No se encontraron registros} =1{Se encontró 1 registro} other{Se encontraron {1} registros}}"
     msgstr "{placeholder,plural,=0{Ningún resultado} =1{1 resultado} other{{1} resultados}}"

Y en la aplicación utiliza el siguiente código para mostrar cualquiera de las traducciones para dicha cadena::

    __('{0,plural,=0{No se encontraron registros }=1{Se encontró 1 registro} other{Se encontraron # registros}}', [0]);

    // Devuelve "No se encontraron registros" ya que el argumento {0} es 0

    __('{0,plural,=0{No se encontraron registros} =1{Se encontró 1 registro} other{Se encontraron # registros}}', [1]);

    // Devuelve "Se encontró 1 registro" porque el argumento {0} es 1

    __('{placeholder,plural,=0{No se encontraron registros} =1{Se encontró 1 registro} other{Se encontraron {1} registros}}', [0, 'muchos', 'placeholder' => 2])

    // Devuelve "muchos resultados" porque el argumento {placeholder} es 2 y
    // el argumento {1} es 'muchos'

Un vistazo más cercano al formato que acabamos de usar hará evidente cómo se construyen los mensajes::

    { [contenedor de recuento],plural, caso1{mensaje} caso2{mensaje} caso3{...} ... }

El ``[contenedor de recuento]`` puede ser el número de clave del arreglo de cualquiera de las variables que pases a la función de traducción. Se usará para seleccionar la forma plural correcta.

Ten en cuenta que para hacer referencia a ``[contenedor de recuento]`` dentro de ``{mensaje}`` debes usar ``#``.

Por supuesto, puedes usar identificadores de mensaje más simples si no quieres escribir la secuencia completa de selección plural en tu código.

.. code-block:: pot

     msgid "search.results"
     msgstr "{0,plural,=0{Ningún resultado} =1{1 resultado} other{{1} resultados}}"

Luego usa la nueva cadena en tu código::

    __('search.results', [2, 2]);

    // Devuelve: "2 resultados"

La última versión tiene la desventaja de que se necesita tener un archivo de mensajes de traducción incluso para el idioma predeterminado, pero tiene la ventaja de que hace que el código sea más legible y deja las complicadas cadenas de selección plural en los archivos de traducción.

A veces, usar coincidencias directas de números en plurales es poco práctico. Por ejemplo, idiomas como el árabe requieren un plural diferente cuando te refieres a pocas cosas y otra forma plural para muchas cosas. En esos casos, puedes usar los alias de coincidencia de ICU. En lugar de escribir::

    =0{No hay resultados} =1{...} other{...}

Puedes hacer::

    zero{No hay resultados} one{Un resultado} few{...} many{...} other{...}

Asegúrate de leer la `Guía de Reglas Plurales de Idiomas <https://unicode-org.github.io/cldr-staging/charts/37/supplemental/language_plural_rules.html>`_ para obtener una visión completa de los alias que puedes usar para cada idioma.

Usando la Selección Plural de Gettext
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

El segundo formato de selección plural aceptado es usando las capacidades integradas
de Gettext. En este caso, los plurales se almacenarán en el archivo ``.po``
creando una línea de traducción de mensaje separada por cada forma plural:

.. code-block:: pot

    # Un identificador de mensaje para singular
    msgid "Un archivo eliminado"
    # Otro para plural
    msgid_plural "{0} archivos eliminados"
    # Traducción en singular
    msgstr[0] "Un fichero eliminado"
    # Traducción en plural
    msgstr[1] "{0} ficheros eliminados"

Al usar este otro formato, se requiere usar otra función de traducción::

    // Devuelve: "10 ficheros eliminados"
    $count = 10;
    __n('Un archivo eliminado', '{0} archivos eliminados', $count, $count);

    // También es posible usarlo dentro de un dominio
    __dn('mi_plugin', 'Un archivo eliminado', '{0} archivos eliminados', $count, $count);

El número dentro de ``msgstr[]`` es el número asignado por Gettext para la forma plural
del idioma. Algunos idiomas tienen más de dos formas plurales, por ejemplo el croata:

.. code-block:: pot

    msgid "Un archivo eliminado"
    msgid_plural "{0} archivos eliminados"
    msgstr[0] "{0} datoteka je uklonjena"
    msgstr[1] "{0} datoteke su uklonjene"
    msgstr[2] "{0} datoteka je uklonjeno"

Por favor, visita la `página de idiomas de Launchpad <https://translations.launchpad.net/+languages>`_
para obtener una explicación detallada de los números de forma plural para cada idioma.

Creando Tus Propios Traductores
===============================

Si necesitas divergir de las convenciones de CakePHP con respecto a dónde y cómo se almacenan los mensajes de traducción, puedes crear tu propio cargador de mensajes de traducción. La forma más fácil de crear tu propio traductor es definiendo un cargador para un único dominio y localidad::

    use Cake\I18n\Package;
    // Antes de la versión 4.2, necesitas usar Aura\Intl\Package

    I18n::setTranslator('animales', function () {
        $package = new Package(
            'default', // La estrategia de formato (ICU)
            'default'  // El dominio de reserva
        );
        $package->setMessages([
            'Perro' => 'Chien',
            'Gato' => 'Chat',
            'Pájaro' => 'Oiseau'
            ...
        ]);

        return $package;
    }, 'fr_FR');

El código anterior se puede agregar a tu **config/bootstrap.php** para que las traducciones se puedan encontrar antes de que se utilice cualquier función de traducción. Lo mínimo absoluto que se requiere para crear un traductor es que la función de cargador debe devolver un objeto ``Cake\I18n\Package`` (antes de la versión 4.2 debería ser un objeto ``Aura\Intl\Package``). Una vez que el código está en su lugar, puedes usar las funciones de traducción como de costumbre::

    I18n::setLocale('fr_FR');
    __d('animales', 'Perro'); // Devuelve "Chien"

Como ves, los objetos ``Package`` toman los mensajes de traducción como un arreglo. Puedes pasar el método ``setMessages()`` como quieras: con código en línea, incluyendo otro archivo, llamando a otra función, etc. CakePHP proporciona algunas funciones de cargador que puedes reutilizar si solo necesitas cambiar dónde se cargan los mensajes. Por ejemplo, aún puedes usar archivos **.po**, pero cargados desde otra ubicación::

    use Cake\I18n\MessagesFileLoader as Loader;

    // Carga los mensajes desde resources/locales/folder/sub_folder/filename.po
    I18n::setTranslator(
        'animales',
        new Loader('filename', 'folder/sub_folder', 'po'),
        'fr_FR'
    );

Creación de Analizadores de Mensajes
------------------------------------

Es posible continuar utilizando las mismas convenciones que CakePHP utiliza, pero usar un analizador de mensajes distinto a ``PoFileParser``. Por ejemplo, si deseas cargar mensajes de traducción usando ``YAML``, primero necesitarás crear la clase del analizador::

    namespace App\I18n\Parser;

    class YamlFileParser
    {
        public function parse($file)
        {
            return yaml_parse_file($file);
        }
    }

El archivo debe ser creado en el directorio **src/I18n/Parser** de tu aplicación. Luego, crea el archivo de traducciones bajo **resources/locales/fr_FR/animals.yaml**

.. code-block:: yaml

    Dog: Chien
    Cat: Chat
    Bird: Oiseau

Y finalmente, configura el cargador de traducción para el dominio y la localidad::

    use Cake\I18n\MessagesFileLoader as Loader;

    I18n::setTranslator(
        'animals',
        new Loader('animals', 'fr_FR', 'yaml'),
        'fr_FR'
    );

.. _creating-generic-translators:

Creación de Traductores Genéricos
------------------------------------

Configurar traductores llamando a ``I18n::setTranslator()`` para cada dominio y
localidad que necesites soportar puede resultar tedioso, especialmente si necesitas
soportar más que unas pocas localidades diferentes. Para evitar este problema, CakePHP te permite definir cargadores de traducción genéricos para cada dominio.

Imagina que deseas cargar todas las traducciones para el dominio predeterminado y para cualquier idioma desde un servicio externo::

    use Cake\I18n\Package;
    // Antes de la versión 4.2 necesitas usar Aura\Intl\Package

    I18n::config('default', function ($domain, $locale) {
        $locale = Locale::parseLocale($locale);
        $lang = $locale['language'];
        $messages = file_get_contents("http://ejemplo.com/traducciones/$lang.json");

        return new Package(
            'default', // Formateador
            null, // Reserva (ninguna para el dominio predeterminado)
            json_decode($messages, true)
        )
    });

El ejemplo anterior llama a un servicio externo de ejemplo para cargar un archivo JSON con las traducciones y luego simplemente construye un objeto ``Package`` para cualquier localidad que se solicite en la aplicación.

Si deseas cambiar cómo se cargan los paquetes para todos los paquetes que no tienen cargadores específicos establecidos, puedes reemplazar el cargador de paquetes de reserva utilizando el paquete ``_fallback``::

    I18n::config('_fallback', function ($domain, $locale) {
        // Código personalizado que devuelve un paquete aquí.
    });

Plurales y Contexto en Traductores Personalizados
-------------------------------------------------

Los arreglos utilizados para ``setMessages()`` pueden ser diseñados para instruir al traductor a almacenar mensajes bajo diferentes dominios o para activar la selección de plurales al estilo Gettext. A continuación, se muestra un ejemplo de cómo almacenar traducciones para la misma clave en diferentes contextos::

    [
        'Él lee la letra {0}' => [
            'alfabeto' => 'Él lee la letra {0}',
            'comunicación escrita' => 'Él lee la carta {0}',
        ],
    ]

De manera similar, puedes expresar plurales al estilo Gettext utilizando el arreglo de mensajes al tener una clave de arreglo anidada por forma plural::

    [
        'He leído un libro' => 'He leído un libro',
        'He leído {0} libros' => [
            'He leído un libro',
            'He leído {0} libros',
        ],
    ]

Usando Diferentes Formateadores
-------------------------------

En ejemplos anteriores hemos visto que los Paquetes se construyen utilizando ``default`` como primer argumento, y se indicó con un comentario que correspondía al formateador a utilizar. Los formateadores son clases responsables de interpolar variables en mensajes de traducción y seleccionar la forma plural correcta.

Si estás trabajando con una aplicación heredada, o no necesitas el poder ofrecido por el formato de mensaje ICU, CakePHP también proporciona el formateador ``sprintf``::

    return Package('sprintf', 'dominio_de_reserva', $mensajes);

Los mensajes a traducir se pasarán a la función ``sprintf()`` para interpolar las variables::

    __('Hola, mi nombre es %s y tengo %d años', 'José', 29);

Es posible establecer el formateador predeterminado para todos los traductores creados por CakePHP antes de que se utilicen por primera vez. Esto no incluye los traductores creados manualmente utilizando los métodos ``setTranslator()`` y ``config()``::

    I18n::defaultFormatter('sprintf');

Localización de Fechas y Números
================================

Cuando se muestran Fechas y Números en tu aplicación, a menudo necesitas que se formateen según el formato preferido para el país o región en la que deseas que se muestre tu página.

Para cambiar cómo se muestran las fechas y los números, solo necesitas cambiar la configuración de localidad actual y utilizar las clases correctas::

    use Cake\I18n\I18n;
    use Cake\I18n\DateTime;
    use Cake\I18n\Number;

    I18n::setLocale('fr-FR');

    $fecha = new DateTime('2015-04-05 23:00:00');

    echo $fecha; // Muestra 05/04/2015 23:00

    echo Number::format(524.23); // Muestra 524,23

Asegúrate de leer las secciones :doc:`/core-libraries/time` y :doc:`/core-libraries/number` para aprender más sobre las opciones de formato.

Por defecto, las fechas devueltas para los resultados del ORM utilizan la clase ``Cake\I18n\DateTime``, por lo que mostrarlas directamente en tu aplicación se verá afectado por cambiar la localidad actual.

Análisis de Datos de Fecha y Hora Localizados
----------------------------------------------

Cuando aceptas datos localizados desde la solicitud, es bueno aceptar información de fecha y hora en el formato localizado del usuario. En un controlador, o en un :doc:`/controllers/middleware`, puedes configurar los tipos de Fecha, Hora y FechaHora para analizar formatos localizados::

    use Cake\Database\TypeFactory;

    // Habilita el análisis del formato de localización predeterminado.
    TypeFactory::build('datetime')->useLocaleParser();

    // Configura un formato de analizador de fecha y hora personalizado.
    TypeFactory::build('datetime')->useLocaleParser()->setLocaleFormat('dd-M-y');

    // También puedes usar constantes de IntlDateFormatter.
    TypeFactory::build('datetime')->useLocaleParser()
        ->setLocaleFormat([IntlDateFormatter::SHORT, -1]);

El formato de análisis predeterminado es el mismo que el formato de cadena predeterminado.

Conversión de Datos de Solicitud desde la Zona Horaria del Usuario
------------------------------------------------------------------

Cuando manejas datos de usuarios en diferentes zonas horarias, necesitarás convertir las fechas y horas en los datos de la solicitud a la zona horaria de tu aplicación. Puedes usar ``setUserTimezone()`` desde un controlador o un :doc:`/controllers/middleware` para hacer este proceso más sencillo::

    // Establece la zona horaria del usuario
    TypeFactory::build('datetime')->setUserTimezone($usuario->zona_horaria);

Una vez establecido, cuando tu aplicación cree o actualice entidades a partir de los datos de la solicitud, el ORM convertirá automáticamente los valores de fecha y hora desde la zona horaria del usuario a la zona horaria de tu aplicación. Esto asegura que tu aplicación siempre esté trabajando en la zona horaria definida en ``App.defaultTimezone``.

Si tu aplicación maneja información de fecha y hora en varias acciones, puedes usar un middleware para definir tanto la conversión de zona horaria como el análisis de localización::

    namespace App\Middleware;

    use Cake\Database\TypeFactory;
    use Psr\Http\Message\ResponseInterface;
    use Psr\Http\Message\ServerRequestInterface;
    use Psr\Http\Server\MiddlewareInterface;
    use Psr\Http\Server\RequestHandlerInterface;

    class DatetimeMiddleware implements MiddlewareInterface
    {
        public function process(
            ServerRequestInterface $request,
            RequestHandlerInterface $handler
        ): ResponseInterface {
            // Obtén el usuario desde la solicitud.
            // Este ejemplo asume que tu entidad de usuario tiene un atributo de zona horaria.
            $usuario = $request->getAttribute('identidad');
            if ($usuario) {
                TypeFactory::build('datetime')
                    ->useLocaleParser()
                    ->setUserTimezone($usuario->zona_horaria);
            }

            return $handler->handle($request);
        }
    }

Eligiendo Automáticamente el Idioma Basado en los Datos de la Solicitud
=======================================================================

Al utilizar el ``LocaleSelectorMiddleware`` en tu aplicación, CakePHP establecerá automáticamente el idioma basado en el usuario actual::

    // en src/Application.php
    use Cake\I18n\Middleware\LocaleSelectorMiddleware;

    // Actualiza la función del middleware, añadiendo el nuevo middleware
    public function middleware(MiddlewareQueue $middlewareQueue): MiddlewareQueue
    {
        // Agrega el middleware y establece los idiomas válidos
        $middlewareQueue->add(new LocaleSelectorMiddleware(['en_US', 'fr_FR']));
        // Para aceptar cualquier valor de encabezado de idioma
        $middlewareQueue->add(new LocaleSelectorMiddleware(['*']));
    }

El ``LocaleSelectorMiddleware`` utilizará el encabezado ``Accept-Language`` para establecer automáticamente el idioma preferido del usuario. Puedes usar la opción de lista de idiomas para restringir qué idiomas se utilizarán automáticamente.

Traducir Contenido/Entidades
=============================

Si deseas traducir contenido/entidades, entonces deberías consultar el :doc:`Comportamiento de Traducción </orm/behaviors/translate>`.

.. meta::
    :title lang=es: Internacionalización y Localización
    :keywords lang=es: internacionalización y localización, internacionalización y localización, aplicación de idioma, gettext, l10n, pot, i18n, traducción, idiomas
