Utilidad de Seguridad
#####################

.. php:namespace:: Cake\Utility

.. php:class:: Security

La `biblioteca de seguridad <https://api.cakephp.org/5.x/class-Cake.Utility.Security.html>`_
maneja medidas básicas de seguridad como proporcionar métodos para hashear y cifrar datos.

Cifrado y Descifrado de Datos
=============================

.. php:staticmethod:: encrypt($texto, $clave, $hmacSalt = null)
.. php:staticmethod:: decrypt($cifrado, $clave, $hmacSalt = null)

Cifra ``$texto`` usando AES-256. La ``$clave`` debe ser un valor con una gran variabilidad en los datos, similar a una buena contraseña. El resultado devuelto será el valor cifrado con una suma de verificación HMAC.

La extensión `openssl <https://php.net/openssl>`_ es necesaria para cifrar/descifrar.

Un ejemplo de uso sería::

    // Suponiendo que la clave se almacena en algún lugar donde pueda reutilizarse para
    // descifrar más tarde.
    $clave = 'wt1U5MACWJFTXGenFoZoiLwQGrLgdbHA';
    $resultado = Security::encrypt($valor, $clave);

Si no proporciona un salt HMAC, se utilizará el valor de ``Security::getSalt()``. Los valores cifrados pueden descifrarse usando :php:meth:`Cake\\Utility\\Security::decrypt()`.

Este método **nunca** debe usarse para almacenar contraseñas.

Descifra un valor previamente cifrado. Los parámetros ``$clave`` y ``$hmacSalt`` deben coincidir con los valores utilizados para cifrar o la descifrado fallará. Un ejemplo de uso sería::

    // Suponiendo que la clave se almacena en algún lugar donde pueda reutilizarse para
    // descifrar más tarde.
    $clave = 'wt1U5MACWJFTXGenFoZoiLwQGrLgdbHA';

    $cifrado = $usuario->secretos;
    $resultado = Security::decrypt($cifrado, $clave);

Si no se puede descifrar el valor debido a cambios en la clave o el salt HMAC, se devolverá ``false``.

Hashing de Datos
================

.. php:staticmethod:: hash( $cadena, $tipo = NULL, $sal = false )

Crea un hash a partir de una cadena usando el método dado. Si no está disponible,
se usa el siguiente método disponible. Si ``$salt`` está establecido en ``true``, el salt
de la aplicación será utilizada::

    // Usando el valor de salt de la aplicación
    $sha1 = Security::hash('CakePHP Framework', 'sha1', true);

    // Usando un valor de salt personalizado
    $sha1 = Security::hash('CakePHP Framework', 'sha1', 'mi-salt');

    // Usando el algoritmo de hash predeterminado
    $hash = Security::hash('CakePHP Framework');

El método ``hash()`` soporta las siguientes estrategias de hash:

- md5
- sha1
- sha256

Y cualquier otro algoritmo de hash que soporte la función ``hash()`` de PHP.

.. warning::

    No deberías estar usando ``hash()`` para contraseñas en nuevas aplicaciones.
    En su lugar, deberías usar la clase ``DefaultPasswordHasher`` que utiliza bcrypt
    de manera predeterminada.

Obtener Datos Aleatorios Seguros
================================

.. php:staticmethod:: randomBytes($longitud)

Obtiene ``$longitud`` número de bytes de una fuente de aleatoriedad segura. Esta función
extrae datos de una de las siguientes fuentes:

* La función ``random_bytes`` de PHP.
* ``openssl_random_pseudo_bytes`` de la extensión SSL.

Si ninguna fuente está disponible, se emitirá una advertencia y se utilizará un valor inseguro
por razones de compatibilidad hacia atrás.

.. php:staticmethod:: randomString($longitud)

Obtiene una cadena aleatoria de ``$longitud`` caracteres de una fuente de aleatoriedad segura. Este método
extrae de la misma fuente aleatoria que ``randomBytes()`` y codificará los datos
como una cadena hexadecimal.

.. meta::
    :title lang=es: Utilidad de Seguridad
    :keywords lang=es: API de seguridad, contraseña secreta, texto cifrado, clase de PHP, seguridad de clase, clave de texto, biblioteca de seguridad, instancia de objeto, medidas de seguridad, seguridad básica, nivel de seguridad, tipo de cadena, alternativa, hash, seguridad de datos, singleton, inactividad, cifrado de PHP, implementación, seguridad de PHP
