# Estructura de Directorios

La estructura por defecto de la aplicación Laravel está pensada para proporcionar un gran punto de partida tanto para aplicaciones grandes como pequeñas. Pero eres libre de organizar tu aplicación como quieras. Laravel casi no impone restricciones sobre la ubicación de cualquier clase - siempre y cuando Composer pueda autocargar la clase.

```markdown
├── app
│   ├── Broadcasting
│   ├── Console
│   ├── Events
│   ├── Exceptions
│   ├── Http
│   ├── Jobs
│   ├── Listeners
│   ├── Mail
│   ├── Models
│   ├── Notifications
│   ├── Policies
│   ├── Providers
│   ├── Rules
├── bootstrap
├── config
├── database
├── public
├── resources 
├── routes
├── storage
├── tests 
└── vendor
```

{% hint style="info" %}
¿Es nuevo en Laravel? Echa un vistazo a la [Laravel Bootcamp](https://bootcamp.laravel.com/) para un recorrido práctico del marco mientras te guiamos a través de la construcción de su primera aplicación Laravel.
{% endhint %}

## Directorio Raíz

### Directorio de App

El directorio `app` contiene el código central de tu aplicación. Exploraremos este directorio en más detalle pronto; sin embargo, casi todas las clases de tu aplicación estarán en este directorio.

### El directorio de Bootstrap

El directorio `bootstrap` contiene el archivo `app.php` que arranca el framework. Este directorio también contiene un directorio `cache` que contiene archivos generados por el framework para optimizar el rendimiento, como los archivos cache de rutas y servicios. Normalmente no debería ser necesario modificar ningún archivo de este directorio.

### El directorio Config

El directorio `config`, como su nombre indica, contiene todos los archivos de configuración de tu aplicación. Es una gran idea leer todos estos archivos y familiarizarse con todas las opciones disponibles.

### El directorio de bases de datos

El directorio `database` contiene tus migraciones de bases de datos, fábricas de modelos y seeders. Si lo deseas, también puedes utilizar este directorio para alojar una base de datos SQLite.

### El Directorio Público

El directorio `public` contiene el archivo `index.php`, que es el punto de entrada para todas las peticiones que entran en tu aplicación y configura la carga automática. Este directorio también contiene tus activos, como imágenes, JavaScript y CSS.

### El Directorio de recursos

El directorio `resources` contiene sus [views](https://laravel.com/docs/10.x/views) así como sus activos sin compilar, como CSS o JavaScript.

### El directorio de rutas

El directorio `routes` contiene todas las definiciones de ruta para tu aplicación. Por defecto, varios archivos de ruta se incluyen con Laravel: `web.php`, `api.php`, `console.php`, y `channels.php`.

El archivo `web.php` contiene rutas que el `RouteServiceProvider` coloca en el grupo de middleware `web`, el cual provee estado de sesión, protección CSRF, y encriptación de cookies. Si tu aplicación no ofrece una API RESTful sin estado, lo más probable es que todas tus rutas estén definidas en el archivo `web.php`.

El archivo `api.php` contiene rutas que el `RouteServiceProvider` coloca en el grupo de middleware `api`. Estas rutas están destinadas a ser sin estado, por lo que las solicitudes que entran en la aplicación a través de estas rutas están destinadas a ser autenticadas a [través de tokens](https://laravel.com/docs/10.x/sanctum) y no tendrán acceso al estado de la sesión.

El archivo `console.php` es donde puede definir todos sus comandos de consola basados en cierres. Cada cierre está vinculado a una instancia de comando permitiendo un enfoque simple para interactuar con los métodos IO de cada comando. Aunque este archivo no define rutas HTTP, define puntos de entrada basados en consola (rutas) en su aplicación.

El archivo `channels.php` es donde puede registrar todos los canales de [difusión de eventos](https://laravel.com/docs/10.x/broadcasting) que su aplicación soporte.

### El directorio de almacenamiento

El directorio `storage` contiene tus logs, plantillas Blade compiladas, sesiones basadas en ficheros, cachés de ficheros y otros ficheros generados por el framework. Este directorio está dividido en los directorios `app`, `framework` y `logs`. El directorio `app` puede ser utilizado para almacenar cualquier archivo generado por tu aplicación. El directorio `framework` se utiliza para almacenar archivos generados por el framework y cachés. Por último, el directorio `logs` contiene los archivos de registro de tu aplicación.

El directorio `storage/app/public` puede utilizarse para almacenar archivos generados por el usuario, como avatares de perfil, que deben ser accesibles públicamente. Debes crear un enlace simbólico en `public/storage` que apunte a este directorio. Puede crear el enlace utilizando el comando `php artisan storage:link` de Artisan.

### El directorio de pruebas

El directorio `tests` contiene tus pruebas automatizadas. Ejemplo [PHPUnit](https://phpunit.de/) pruebas unitarias y pruebas de características se proporcionan fuera de la caja. Cada clase de prueba debe tener como sufijo la palabra `Test`. Puede ejecutar sus pruebas utilizando los comandos `phpunit` o `php vendor/bin/phpunit`. O, si desea una representación más detallada y hermosa de los resultados de sus pruebas, puede ejecutarlas utilizando el comando Artisan `php artisan test`.

### El Directorio de Vendor

El directorio `vendor` contiene sus dependencias [Composer](https://getcomposer.org).

## El Directorio App

La mayor parte de la aplicación se encuentra en el directorio `app`. Por defecto, este directorio tiene el nombre `App` y es autocargado por Composer utilizando el [PSR-4 autoloading standard](https://www.php-fig.org/psr/psr-4/).

El directorio `app` contiene una variedad de directorios adicionales como `Console`, `Http`, y `Providers`. Piensa en los directorios `Console` y `Http` como una API en el núcleo de tu aplicación. Tanto el protocolo HTTP como la CLI son mecanismos para interactuar con tu aplicación, pero en realidad no contienen lógica de aplicación. En otras palabras, son dos formas de emitir comandos a tu aplicación. El directorio `Console` contiene todos tus comandos Artisan, mientras que el directorio `Http` contiene tus controladores, middleware y peticiones.

Una variedad de otros directorios serán generados dentro del directorio `app` cuando utilices los comandos `make` de Artisan para generar clases. Así, por ejemplo, el directorio `app/Jobs` no existirá hasta que ejecutes el comando Artisan `make:job` para generar una clase job.

{% hint style="info" %}
Muchas de las clases en el directorio `app` pueden ser generadas por Artisan a través de comandos. Para revisar los comandos disponibles, ejecute el comando `php artisan list make` en su terminal.
{% endhint %}

