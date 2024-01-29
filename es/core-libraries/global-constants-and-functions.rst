Constantes y Funciones
######################

Mientras que la mayoría de tu trabajo diario en CakePHP será utilizando clases y métodos principales, CakePHP cuenta con una serie de funciones globales de conveniencia que pueden resultar útiles. Muchas de estas funciones son para su uso con clases de CakePHP (cargando clases de modelo o componente), pero muchas otras facilitan el trabajo con matrices o cadenas.

También cubriremos algunas de las constantes disponibles en las aplicaciones de CakePHP. El uso de estas constantes ayudará a que las actualizaciones sean más fluidas, pero también son formas convenientes de apuntar a ciertos archivos o directorios en tu aplicación de CakePHP.

Funciones Globales
==================

Aquí están las funciones globalmente disponibles de CakePHP. La mayoría de ellas son simplemente envoltorios de conveniencia para otras funcionalidades de CakePHP, como depuración y traducción de contenido. Por defecto, solo las funciones con espacio de nombres están cargadas automáticamente, sin embargo, opcionalmente puedes cargar alias globales agregando::

    require CAKE . 'functions.php';

A tu archivo ``config/bootstrap.php`` de la aplicación. Hacer esto cargará alias globales para *todas* las funciones listadas a continuación.

.. php:namespace:: Cake\I18n

.. php:function:: \_\_(string $string_id, [$formatArgs])

    Esta función maneja la localización en las aplicaciones de CakePHP. El ``$string_id`` identifica el ID para una traducción. Puedes proporcionar argumentos adicionales para reemplazar marcadores de posición en tu cadena::

        __('Tienes {0} mensajes sin leer', $number);

    También puedes proporcionar un array de reemplazos indexados por nombre::

        __('Tienes {unread} mensajes sin leer', ['unread' => $number]);

    .. note::

        Consulta la sección de :doc:`/core-libraries/internationalization-and-localization` para obtener más información.

.. php:function:: __d(string $domain, string $msg, mixed $args = null)

    Permite anular el dominio actual para una búsqueda de mensaje única.

    Útil al internacionalizar un plugin:

    ``echo __d('nombre_del_plugin', 'Este es mi plugin');``

    .. note::

        Asegúrate de usar la versión con guiones bajos del nombre del plugin aquí como dominio.

.. php:function:: __dn(string $domain, string $singular, string $plural, integer $count, mixed $args = null)

    Permite anular el dominio actual para una búsqueda de mensaje plural única. Devuelve la forma plural correcta del mensaje identificado por ``$singular`` y ``$plural`` para el recuento ``$count`` desde el dominio ``$domain``.

.. php:function:: __dx(string $domain, string $context, string $msg, mixed $args = null)

    Permite anular el dominio actual para una búsqueda de mensaje única. También te permite especificar un contexto.

    El contexto es un identificador único para la cadena de traducciones que lo hace único dentro del mismo dominio.

.. php:function:: __dxn(string $domain, string $context, string $singular, string $plural, integer $count, mixed $args = null)

    Permite anular el dominio actual para una búsqueda de mensaje plural única. También te permite especificar un contexto. Devuelve la forma plural correcta del mensaje identificado por ``$singular`` y ``$plural`` para el recuento ``$count`` desde el dominio ``$domain``. Algunos idiomas tienen más de una forma para mensajes plurales dependientes del recuento.

    El contexto es un identificador único para la cadena de traducciones que lo hace único dentro del mismo dominio.

.. php:function:: __n(string $singular, string $plural, integer $count, mixed $args = null)

    Devuelve la forma plural correcta del mensaje identificado por ``$singular`` y ``$plural`` para el recuento ``$count``. Algunos idiomas tienen más de una forma para mensajes plurales dependientes del recuento.

.. php:function:: __x(string $context, string $msg, mixed $args = null)

    El contexto es un identificador único para la cadena de traducciones que lo hace único dentro del mismo dominio.

.. php:function:: __xn(string $context, string $singular, string $plural, integer $count, mixed $args = null)

    Devuelve la forma plural correcta del mensaje identificado por ``$singular`` y ``$plural`` para el recuento ``$count`` desde el dominio ``$domain``. También te permite especificar un contexto. Algunos idiomas tienen más de una forma para mensajes plurales dependientes del recuento.

    El contexto es un identificador único para la cadena de traducciones que lo hace único dentro del mismo dominio.

.. php:namespace:: Cake\Collection

.. php:function:: collection(mixed $items)

    Envoltorio de conveniencia para instanciar un nuevo objeto :php:class:`Cake\\Collection\\Collection`, envolviendo el argumento pasado. El parámetro ``$items`` puede tomar un objeto ``Traversable`` o una matriz.

.. php:namespace:: Cake\Core

.. php:function:: debug(mixed $var, boolean $showHtml = null, $showFrom = true)

    Si la variable central ``$debug`` es ``true``, se imprime ``$var``.
    Si ``$showHTML`` es ``true`` o se deja como ``null``, los datos se renderizan para ser
    amigables con el navegador. Si ``$showFrom`` no se establece en ``false``, la salida de depuración
    comenzará con la línea desde la que se llamó. También consulta
    :doc:`/development/debugging`

.. php:function:: dd(mixed $var, boolean $showHtml = null)

    Se comporta como ``debug()``, pero también se detiene la ejecución.
    Si la variable central ``$debug`` es ``true``, se imprime ``$var``.
    Si ``$showHTML`` es ``true`` o se deja como ``null``, los datos se renderizan para ser
    amigables con el navegador. También consulta :doc:`/development/debugging`

.. php:function:: pr(mixed $var)

    Envoltorio de conveniencia para ``print_r()``, con la adición de
    envolver las etiquetas ``<pre>`` alrededor de la salida.

.. php:function:: pj(mixed $var)

    Función de conveniencia para imprimir bonito en JSON, con la adición de
    envolver las etiquetas ``<pre>`` alrededor de la salida.

    Está destinado a depurar la representación JSON de objetos y matrices.

.. php:function:: env(string $key, string $default = null)

    Obtiene una variable de entorno de las fuentes disponibles. Se usa como respaldo si
    ``$_SERVER`` o ``$_ENV`` están desactivados.

    Esta función también emula ``PHP_SELF`` y ``DOCUMENT_ROOT`` en
    servidores que no lo soportan. De hecho, es una buena idea usar siempre ``env()``
    en lugar de ``$_SERVER`` o ``getenv()`` (especialmente si planeas
    distribuir el código), ya que es un envoltorio de emulación completo.

.. php:function:: h(string $text, boolean $double = true, string $charset = null)

    Envoltorio de conveniencia para ``htmlspecialchars()``.

.. php:function:: pluginSplit(string $name, boolean $dotAppend = false, string $plugin = null)

    Divide un nombre de plugin de sintaxis de punto en su plugin y nombre de clase. Si ``$name``
    no tiene un punto, entonces el índice 0 será ``null``.

    Comúnmente utilizado como ``list($plugin, $name) = pluginSplit('Usuarios.Usuario');``

.. php:function:: namespaceSplit(string $class)

    Divide el espacio de nombres del nombre de clase.

    Comúnmente utilizado como ``list($namespace, $nombreClase) = namespaceSplit('Cake\Core\App');``

Constantes de Definición Principal
==================================

La mayoría de las siguientes constantes se refieren a rutas en tu aplicación.

.. php:const:: APP

   Ruta absoluta al directorio de tu aplicación, incluyendo una barra diagonal al final.

.. php:const:: APP_DIR

    Equivale a ``app`` o al nombre de tu directorio de aplicación.

.. php:const:: CACHE

    Ruta al directorio de archivos de caché. Puede ser compartido entre hosts en una configuración multi-servidor.

.. php:const:: CAKE

    Ruta al directorio de Cake.

.. php:const:: CAKE_CORE_INCLUDE_PATH

    Ruta al directorio raíz lib.

.. php:const:: CONFIG

   Ruta al directorio de configuración.

.. php:const:: CORE_PATH

   Ruta al directorio CakePHP con la barra diagonal de directorio final.

.. php:const:: DS

    Abreviatura de ``DIRECTORY_SEPARATOR`` de PHP, que es ``/`` en Linux y ``\`` en Windows.

.. php:const:: LOGS

    Ruta al directorio de registros.

.. php:const:: RESOURCES

   Ruta al directorio de recursos.

.. php:const:: ROOT

    Ruta al directorio raíz.

.. php:const:: TESTS

    Ruta al directorio de pruebas.

.. php:const:: TMP

    Ruta al directorio de archivos temporales.

.. php:const:: WWW_ROOT

    Ruta completa al directorio raíz de la web.

.. meta::
    :title lang=es: Constantes y Funciones Globales
    :keywords lang=es: internacionalización y localización,constantes globales,configuración de ejemplo,matriz php,funciones de conveniencia,bibliotecas principales,clases de componentes,número opcional,funciones globales,cadena de caracteres,clases principales,formato de cadenas,mensajes no leídos,marcadores de posición,funciones útiles,sprintf,matrices,parámetros,existencia,traducciones
