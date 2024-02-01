Clase Plugin
############

.. php:namespace:: Cake\Core

.. php:class:: Plugin

La clase Plugin es responsable de la ubicación de recursos y la gestión de rutas de los plugins.

Ubicación de Plugins
====================

.. php:staticmethod:: path(string $plugin)

Los plugins pueden ser ubicados con Plugin. Usando ``Plugin::path('DebugKit');``
por ejemplo, te dará la ruta completa al plugin DebugKit::

    $path = Plugin::path('DebugKit');

Comprobar si un Plugin está Cargado
===================================

Puedes comprobar dinámicamente dentro de tu código si un plugin específico ha sido cargado::

    $isLoaded = Plugin::isLoaded('DebugKit');

Usa ``Plugin::loaded()`` si deseas obtener una lista de todos los plugins actualmente cargados.

Encontrar Rutas a Espacios de Nombres
=====================================

.. php:staticmethod:: classPath(string $plugin)

Usado para obtener la ubicación de los archivos de clase del plugin::

    $path = App::classPath('DebugKit');

Encontrar Rutas a Recursos
==========================

.. php:staticmethod:: templatePath(string $plugin)

El método devuelve la ruta a las plantillas de los plugins::

    $path = Plugin::templatePath('DebugKit');

Lo mismo ocurre para la ruta de configuración::

    $path = Plugin::configPath('DebugKit');

.. meta::
    :title lang=es: Clase Plugin
    :keywords lang=es: implementación compatible, comportamientos de modelo, gestión de rutas, carga de archivos, clase de PHP, carga de clases, comportamiento de modelo, ubicación de clase, modelo de componente, clase de gestión, cargador automático, nombre de clase, ubicación de directorio, anulación, convenciones, librería, textile, CakePHP, clases de PHP, cargado
