Comprobando la Caché HTTP
=========================

.. php:class:: CheckHttpCacheComponent(ComponentCollection $collection, array $config = [])

El modelo de validación de caché HTTP es uno de los procesos utilizados por las pasarelas de caché, también conocidas como proxies inversos, para determinar si pueden servir una copia almacenada de una respuesta al cliente. Bajo este modelo, principalmente se ahorra ancho de banda, pero cuando se utiliza correctamente, también se puede ahorrar algo de procesamiento de CPU, reduciendo los tiempos de respuesta::

    // En un Controlador
    public function initialize(): void
    {
        parent::initialize();

        $this->addComponent('CheckHttpCache');
    }

Habilitar el componente ``CheckHttpCache`` en su controlador activa automáticamente una comprobación ``beforeRender``. Esta comprobación compara los encabezados de caché establecidos en el objeto de respuesta con los encabezados de caché enviados en la solicitud para determinar si la respuesta no se ha modificado desde la última vez que el cliente la solicitó. Se utilizan los siguientes encabezados de solicitud:

    ``If-None-Match`` se compara con el encabezado ``Etag`` de la respuesta.
    ``If-Modified-Since`` se compara con el encabezado ``Last-Modified`` de la respuesta.

Si los encabezados de la respuesta coinciden con los criterios de los encabezados de la solicitud, se omite el renderizado de la vista. Esto ahorra a su aplicación la generación de una vista, ahorrando ancho de banda y tiempo. Cuando los encabezados de la respuesta coinciden, se devuelve una respuesta vacía con un código de estado ``304 No Modificado``.
