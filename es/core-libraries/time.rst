Fecha y Hora
##############

.. php:namespace:: Cake\I18n

.. php:class:: DateTime

Si necesitas funcionalidades de :php:class:`TimeHelper` fuera de una ``Vista``,
utiliza la clase ``DateTime``::

    use Cake\I18n\DateTime;

    class UsersController extends AppController
    {
        public function initialize(): void
        {
            parent::initialize();
            $this->loadComponent('Authentication.Authentication');
        }

        public function afterLogin()
        {
            $identity = $this->Authentication->getIdentity();
            $time = new DateTime($identity->date_of_birth);
            if ($time->isToday()) {
                // Saluda al usuario con un mensaje de cumpleaños feliz
                $this->Flash->success(__('¡Feliz cumpleaños!'));
            }
        }
    }

POr debajo, CakePHP utiliza `Chronos <https://github.com/cakephp/chronos>`_
para alimentar su utilidad ``DateTime``. Todo lo que puedas hacer con ``Chronos``
y con ``DateTime`` de PHP, puedes hacerlo con ``DateTime`` y ``Date``.

Para obtener más detalles sobre Chronos, consulta `la documentación de la API
<https://api.cakephp.org/chronos/1.0/>`_.

.. start-time

Creación de Instancias de DateTime
==================================

Las instancias de ``DateTime`` son objetos inmutables que son útiles cuando quieres evitar cambios accidentales en los datos, o cuando deseas evitar problemas de dependencia basados en el orden.

Existen algunas formas de crear instancias de ``DateTime``::

    use Cake\I18n\DateTime;

    // Crear desde una cadena de fecha y hora.
    $time = DateTime::createFromFormat(
        'Y-m-d H:i:s',
        '2021-01-31 22:11:30',
        'America/New_York'
    );

    // Crear desde una marca de tiempo y establecer la zona horaria.
    $time = DateTime::createFromTimestamp(1612149090, 'America/New_York');

    // Obtener la hora actual.
    $time = DateTime::now();

    // O simplemente usar 'new'
    $time = new DateTime('2021-01-31 22:11:30', 'America/New_York');

    $time = new DateTime('hace 2 horas');

El constructor de la clase ``DateTime`` puede tomar cualquier parámetro que la clase interna de PHP ``DateTimeImmutable`` pueda. Cuando se pasa un número o una cadena numérica, se interpretará como una marca de tiempo UNIX.

En casos de prueba, puedes simular ``now()`` usando ``setTestNow()``::

    // Fijar la hora.
    $time = new DateTime('2021-01-31 22:11:30');
    DateTime::setTestNow($time);

    // Muestra '2021-01-31 22:11:30'
    $now = DateTime::now();
    echo $now->i18nFormat('yyyy-MM-dd HH:mm:ss');

    // Muestra '2021-01-31 22:11:30'
    $now = DateTime::parse('now');
    echo $now->i18nFormat('yyyy-MM-dd HH:mm:ss');

Manipulación
============

Recuerda, una instancia de ``DateTime`` siempre devuelve una nueva instancia desde los métodos establecedores en lugar de modificarla a sí misma::

    $time = DateTime::now();

    // Crear y reasignar una nueva instancia
    $newTime = $time->year(2013)
        ->month(10)
        ->day(31);
    // Muestra '2013-10-31 22:11:30'
    echo $newTime->i18nFormat('yyyy-MM-dd HH:mm:ss');

También puedes usar los métodos proporcionados por la clase ``DateTime`` incorporada de PHP::

    $time = $time->setDate(2013, 10, 31);

No volver a asignar las nuevas instancias de ``DateTime`` resultará en el uso de la instancia original, no modificada::

    $time->year(2013)
        ->month(10)
        ->day(31);
    // Muestra '2021-01-31 22:11:30'
    echo $time->i18nFormat('yyyy-MM-dd HH:mm:ss');

Puedes crear otra instancia con fechas modificadas, mediante la resta y la adición de sus componentes::

    $time = DateTime::create(2021, 1, 31, 22, 11, 30);
    $newTime = $time->subDays(5)
        ->addHours(-2)
        ->addMonth(1);
    // Muestra '2/26/21, 8:11 PM'
    echo $newTime;

    // Usando cadenas strtotime.
    $newTime = $time->modify('+1 month -5 days -2 hours');
    // Muestra '2/26/21, 8:11 PM'
    echo $newTime;

Puedes obtener los componentes internos de una fecha accediendo a sus propiedades::

    $time = DateTime::create(2021, 1, 31, 22, 11, 30);
    echo $time->year; // 2021
    echo $time->month; // 1
    echo $time->day; // 31
    echo $time->timezoneName; // America/New_York

Formateo
========

.. php:staticmethod:: setJsonEncodeFormat($format)

Este método establece el formato predeterminado utilizado al convertir un objeto a json::

    DateTime::setJsonEncodeFormat('yyyy-MM-dd HH:mm:ss');  // Para cualquier DateTime inmutable
    Date::setJsonEncodeFormat('yyyy-MM-dd HH:mm:ss');  // Para cualquier Date mutable

    $time = DateTime::parse('2021-01-31 22:11:30');
    echo json_encode($time);   // Muestra '2021-01-31 22:11:30'

    Date::setJsonEncodeFormat(static function($time) {
        return $time->format(DATE_ATOM);
    });

.. nota::
    Este método debe ser llamado estáticamente.

.. nota::
    ¡Ten en cuenta que este no es un formato de cadena de fecha y hora de PHP! Necesitas usar una cadena de formato de fecha y hora de ICU como se especifica en el siguiente recurso:
    https://unicode-org.github.io/icu/userguide/format_parse/datetime/#datetime-format-syntax.

.. versionchanged:: 4.1.0
    Se añadió el tipo de parámetro ``callable``.


.. php:method:: i18nFormat($format = null, $timezone = null, $locale = null)

Una tarea muy común con las instancias de ``Time`` es imprimir fechas formateadas. CakePHP hace que esto sea muy fácil::

    $time = DateTime::parse('2021-01-31 22:11:30');

    // Imprime una marca de fecha y hora localizada. Muestra '1/31/21, 10:11 PM'
    echo $time;

    // Muestra '1/31/21, 10:11 PM' para la configuración regional en-US
    echo $time->i18nFormat();

    // Usa el formato de fecha y hora completo. Muestra 'Sunday, January 31, 2021 at 10:11:30 PM Eastern Standard Time'
    echo $time->i18nFormat(\IntlDateFormatter::FULL);

    // Usa fecha completa pero formato de hora corto. Muestra 'Sunday, January 31, 2021 at 10:11 PM'
    echo $time->i18nFormat([\IntlDateFormatter::FULL, \IntlDateFormatter::SHORT]);

    // Muestra '2021-Jan-31 22:11:30'
    echo $time->i18nFormat('yyyy-MMM-dd HH:mm:ss');

Es posible especificar el formato deseado para la cadena a mostrar. Puedes pasar tanto `constantes de IntlDateFormatter
<https://www.php.net/manual/en/class.intldateformatter.php>`_ como el primer argumento de esta función, o pasar una cadena de formato de fecha y hora ICU completa como se especifica en el siguiente recurso:
https://unicode-org.github.io/icu/userguide/format_parse/datetime/#datetime-format-syntax.

También puedes formatear fechas con calendarios no gregorianos::

    // En la versión ICU 66.1
    $time = DateTime::create(2021, 1, 31, 22, 11, 30);

    // Muestra 'Sunday, Bahman 12, 1399 AP at 10:11:30 PM Eastern Standard Time'
    echo $time->i18nFormat(\IntlDateFormatter::FULL, null, 'en-IR@calendar=persian');

    // Muestra 'Sunday, January 31, 3 Reiwa at 10:11:30 PM Eastern Standard Time'
    echo $time->i18nFormat(\IntlDateFormatter::FULL, null, 'en-JP@calendar=japanese');

    // Muestra 'Sunday, Twelfth Month 19, 2020(geng-zi) at 10:11:30 PM Eastern Standard Time'
    echo $time->i18nFormat(\IntlDateFormatter::FULL, null, 'en-CN@calendar=chinese');

    // Muestra 'Sunday, Jumada II 18, 1442 AH at 10:11:30 PM Eastern Standard Time'
    echo $time->i18nFormat(\IntlDateFormatter::FULL, null, 'en-SA@calendar=islamic');

Los siguientes tipos de calendarios son compatibles:

* japonés
* budista
* chino
* persa
* indio
* islámico
* hebreo
* copto
* etíope

.. nota::
    Para cadenas constantes es decir IntlDateFormatter::FULL Intl utiliza la biblioteca ICU que alimenta sus datos desde CLDR (https://cldr.unicode.org/) cuya versión puede variar dependiendo de la instalación de PHP y dar resultados diferentes.

.. php:method:: nice()

Imprime un formato 'bonito' predefinido::

    $time = DateTime::parse('2021-01-31 22:11:30', new \DateTimeZone('America/New_York'));

    // Muestra '31 de ene. de 2021 10:11 PM' en es-ES
    echo $time->nice();

Puedes alterar la zona horaria en la que se muestra la fecha sin alterar el
objeto ``DateTime`` en sí mismo. Esto es útil cuando almacenas fechas en una zona horaria, pero
quieres mostrarlas en la zona horaria del usuario::

    // Muestra 'lunes, 1 de feb. de 2021 a las 4:11:30 AM hora estándar de Europa central'
    echo $time->i18nFormat(\IntlDateFormatter::FULL, 'Europe/Paris');

    // Muestra 'lunes, 1 de feb. de 2021 a las 12:11:30 PM hora estándar de Japón'
    echo $time->i18nFormat(\IntlDateFormatter::FULL, 'Asia/Tokyo');

    // La zona horaria no cambia. Muestra 'America/New_York'
    echo $time->timezoneName;

Dejar el primer parámetro como ``null`` usará la cadena de formato predeterminada::

    // Muestra '1/2/21, 4:11 AM'
    echo $time->i18nFormat(null, 'Europe/Paris');

Finalmente, es posible usar una localidad diferente para mostrar una fecha::

    // Muestra 'lundi 1 févr. 2021 à 04:11:30 heure normale d’Europe centrale'
    echo $time->i18nFormat(\IntlDateFormatter::FULL, 'Europe/Paris', 'fr-FR');

    // Muestra '1 févr. 2021 à 04:11'
    echo $time->nice('Europe/Paris', 'fr-FR');

Establecimiento de la Localidad y Cadena de Formato Predeterminada
------------------------------------------------------------------

La localidad predeterminada en la que se muestran las fechas al usar ``nice``
``i18nFormat`` se toma de la directiva
`intl.default_locale <https://www.php.net/manual/en/intl.configuration.php#ini.intl.default-locale>`_.
Sin embargo, puedes modificar este predeterminado en tiempo de ejecución::

    DateTime::setDefaultLocale('es-ES');
    Date::setDefaultLocale('es-ES');

    // Muestra '31 ene. 2021 22:11'
    echo $time->nice();

A partir de ahora, las fechas se mostrarán en el formato preferido en español a menos que
se especifique una localidad diferente directamente en el método de formato.

Asimismo, es posible alterar la cadena de formato predeterminada que se utilizará para
``i18nFormat``::

    DateTime::setToStringFormat(\IntlDateFormatter::SHORT); // Para cualquier DateTime
    Date::setToStringFormat(\IntlDateFormatter::SHORT); // Para cualquier Date

    // El mismo método existe en Date, y DateTime
    DateTime::setToStringFormat([
        \IntlDateFormatter::FULL,
        \IntlDateFormatter::SHORT
    ]);
    // Muestra 'domingo, 31 de enero de 2021 a las 10:11 PM'
    echo $time;

    // El mismo método existe en Date y DateTime
    DateTime::setToStringFormat("EEEE, MMMM dd, yyyy 'a las' KK:mm:ss a");
    // Muestra 'domingo, 31 de enero de 2021 a las 10:11:30 PM'
    echo $time;

Se recomienda siempre usar las constantes en lugar de pasar directamente una cadena de formato de fecha.

.. nota::
    ¡Ten en cuenta que este no es un formato de cadena de fecha y hora de PHP! Necesitas usar una
    cadena de formato de fecha y hora ICU como se especifica en el siguiente recurso:
    https://unicode-org.github.io/icu/userguide/format_parse/datetime/#datetime-format-syntax.

.. php:method:: timeAgoInWords(array $options = [])

A menudo es útil imprimir tiempos relativos al presente::

    $time = new DateTime('31 de ene. de 2021');
    // En el 12 de jun. de 2021, esto mostraría 'hace 4 meses, 1 semana, 6 días'
    echo $time->timeAgoInWords(
        ['format' => 'MMM d, YYY', 'end' => '+1 year']
    );

La opción ``end`` te permite definir en qué momento después del cual los tiempos relativos
deben ser formateados usando la opción ``format``. La opción ``accuracy`` nos permite
controlar qué nivel de detalle se debe usar para cada intervalo::

    // Muestra 'hace 4 meses'
    echo $time->timeAgoInWords([
        'accuracy' => ['month' => 'month'],
        'end' => '1 year'
    ]);

Al establecer ``accuracy`` como una cadena, puedes especificar cuál es el nivel máximo
de detalle que deseas en la salida::

    $time = new DateTime('+23 hours');
    // Muestra 'en aproximadamente un día'
    echo $time->timeAgoInWords([
        'accuracy' => 'day'
    ]);

Conversión
==========

.. php:method:: toQuarter()

Una vez creado, puedes convertir las instancias de ``DateTime`` en marcas de tiempo o valores de trimestre::

    $time = new DateTime('2021-01-31');
    echo $time->toQuarter();  // Muestra '1'
    echo $time->toUnixString();  // Muestra '1612069200'

Comparación con el presente
===========================

.. php:method:: isYesterday()
.. php:method:: isThisWeek()
.. php:method:: isThisMonth()
.. php:method:: isThisYear()

Puedes comparar una instancia de ``DateTime`` con el presente de varias maneras::

    $time = new DateTime('+3 days');

    debug($time->isYesterday());
    debug($time->isThisWeek());
    debug($time->isThisMonth());
    debug($time->isThisYear());

Cada uno de los métodos anteriores devolverá ``true``/``false`` basado en si
la instancia de ``DateTime`` coincide o no con el presente.

Comparación con Intervalos
==========================

.. php:method:: isWithinNext($interval)

Puedes ver si una instancia de ``DateTime`` se encuentra dentro de un rango dado usando
``wasWithinLast()`` y ``isWithinNext()``::

    $time = new DateTime('+3 days');

    // Dentro de 2 días. Muestra 'false'
    debug($time->isWithinNext('2 days'));

    // Dentro de 2 próximas semanas. Muestra 'true'
    debug($time->isWithinNext('2 weeks'));

.. php:method:: wasWithinLast($interval)

También puedes comparar una instancia de ``DateTime`` dentro de un rango en el pasado::

    $time = new DateTime('-72 hours');

    // Dentro de los últimos 2 días. Muestra 'false'
    debug($time->wasWithinLast('2 days'));

    // Dentro de los últimos 3 días. Muestra 'true'
    debug($time->wasWithinLast('3 days'));

    // Dentro de las últimas 2 semanas. Muestra 'true'
    debug($time->wasWithinLast('2 weeks'));

Fecha
=====

.. php:class: Date

La clase inmutable ``Date`` en CakePHP implementa una API y métodos similares a
:php:class:`Cake\\I18n\\DateTime`. La principal diferencia entre ``DateTime``
y ``Date`` es que ``Date`` no rastrea los componentes de tiempo. Como ejemplo::

    use Cake\I18n\Date;

    $fecha = new Date('2021-01-31');

    $nuevaFecha = $fecha->modify('+2 horas');
    // Muestra '2021-01-31 00:00:00'
    echo $nuevaFecha->format('Y-m-d H:i:s');

    $nuevaFecha = $fecha->addHours(36);
    // Muestra '2021-01-31 00:00:00'
    echo $nuevaFecha->format('Y-m-d H:i:s');

    $nuevaFecha = $fecha->addDays(10);
    // Muestra '2021-02-10 00:00:00'
    echo $nuevaFecha->format('Y-m-d H:i:s');


Los intentos de modificar la zona horaria en una instancia de ``Date`` también son ignorados::

    use Cake\I18n\Date;
    $fecha = new Date('2021-01-31', new \DateTimeZone('America/New_York'));
    $nuevaFecha = $fecha->setTimezone(new \DateTimeZone('Europe/Berlin'));

    // Muestra 'America/New_York'
    echo $nuevaFecha->format('e');

.. _mutable-time:

Fechas y Tiempos Mutables
=========================

.. php:class:: Time
.. php:class:: Date

CakePHP utiliza clases de fecha y hora mutables que implementan la misma interfaz
que sus contrapartes inmutables. Los objetos inmutables son útiles cuando deseas evitar
cambios accidentales en los datos, o cuando deseas evitar problemas de dependencia
basados en el orden. Toma el siguiente código::

    use Cake\I18n\Time;
    $tiempo = new Time('2015-06-15 08:23:45');
    $tiempo->modify('+2 horas');

    // Este método también modifica la instancia $tiempo
    $this->otraFuncion($tiempo);

    // La salida aquí es desconocida.
    echo $tiempo->format('Y-m-d H:i:s');

Si la llamada al método se reordenara, o si ``otraFuncion`` cambiara,
la salida podría ser inesperada. La mutabilidad de nuestro objeto crea un
acoplamiento temporal. Si usáramos objetos inmutables, podríamos evitar este problema::

    use Cake\I18n\DateTime;
    $tiempo = new DateTime('2015-06-15 08:23:45');
    $tiempo = $tiempo->modify('+2 horas');

    // Las modificaciones de este método no cambian $tiempo
    $this->otraFuncion($tiempo);

    // La salida aquí es conocida.
    echo $tiempo->format('Y-m-d H:i:s');

Las fechas y horas inmutables son útiles en entidades ya que evitan
modificaciones accidentales, y obligan a que los cambios sean explícitos. Usar
objetos inmutables ayuda al ORM a rastrear más fácilmente los cambios y garantizar que
las columnas de fecha y hora se guarden correctamente::

    // Este cambio se perderá cuando se guarde el artículo.
    $articulo->actualizado->modify('+1 hora');

    // Reemplazando el objeto de tiempo, la propiedad se guardará.
    $articulo->actualizado = $articulo->actualizado->modify('+1 hora');

Aceptación de Datos de Solicitud Localizados
============================================

Al crear entradas de texto que manipulan fechas, probablemente quieras aceptar
y analizar cadenas de fecha y hora localizadas. Ver :ref:`parsing-localized-dates`.

Zonas Horarias Soportadas
=========================

CakePHP admite todas las zonas horarias PHP válidas. Para ver una lista de zonas horarias admitidas, `consulta esta página <https://php.net/manual/en/timezones.php>`_.

.. meta::
    :title lang=es: Fecha y Hora
    :description lang=es: La clase Time te ayuda a formatear el tiempo y hora, probar el tiempo y hora.
    :keywords lang=es: tiempo,formatear tiempo,zona horaria,época UNIX,cadenas de tiempo,desplazamiento de zona horaria,utc,gmt
