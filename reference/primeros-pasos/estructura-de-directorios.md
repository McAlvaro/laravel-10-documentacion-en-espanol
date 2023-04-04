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

### El Directorio Broadcasting

El directorio `Broadcasting` contiene todas las clases de canales de difusión para su aplicación. Estas clases se generan usando el comando `make:channel`. Este directorio no existe por defecto, pero se creará para ti cuando crees tu primer canal. Para saber más sobre canales, consulta la documentación sobre [event broadcasting](https://laravel.com/docs/10.x/broadcasting).

### El Directorio Console

El directorio `Console` contiene todos los comandos personalizados de Artisan para su aplicación. Estos comandos pueden ser generados utilizando el comando `make:command`. Este directorio también contiene el kernel de tu consola, que es donde tus comandos personalizados de Artisan son registrados y tus [tareas programadas](https://laravel.com/docs/10.x/scheduling) son definidas.

### El Directorio Events

Este directorio no existe por defecto, pero será creado para usted por los comandos Artisan `event:generate` y `make:event`. El directorio `Events` contiene [clases de eventos](https://laravel.com/docs/10.x/events). Los eventos pueden ser utilizados para alertar a otras partes de tu aplicación de que una determinada acción ha ocurrido, proporcionando una gran flexibilidad y desacoplamiento.

### El Directorio Exceptions

El directorio `Exceptions` contiene el manejador de excepciones de tu aplicación y es también un buen lugar para colocar cualquier excepción lanzada por tu aplicación. Si desea personalizar la forma en que sus excepciones se registran o renderizan, debe modificar la clase `Handler` en este directorio.

### El Directorio Http

El directorio `Http` contiene tus controladores, middleware y peticiones de formulario. Casi toda la lógica para manejar las solicitudes que entran en su aplicación se colocará en este directorio.

### El Directorio Jobs

Este directorio no existe por defecto, pero será creado para usted si ejecuta el comando Artisan `make:job`. El directorio `Jobs` aloja los [trabajos en cola](https://laravel.com/docs/10.x/queues) para su aplicación. Los trabajos pueden ser puestos en cola por su aplicación o ejecutarse sincrónicamente dentro del ciclo de vida de la solicitud actual. Los trabajos que se ejecutan de forma sincrónica durante la petición actual se denominan a veces "comandos", ya que son una implementación del [patrón de comandos](https://en.wikipedia.org/wiki/Command\_pattern).

### El Directorio Listeners

Este directorio no existe por defecto, pero será creado por ti si ejecutas los comandos Artisan `event:generate` o `make:listener`. El directorio `Listeners` contiene las clases que manejan tus [eventos](https://laravel.com/docs/10.x/events). Los listeners de eventos reciben una instancia de evento y ejecutan la lógica en respuesta al evento disparado. Por ejemplo, un evento `UserRegistered` puede ser manejado por un listener `SendWelcomeEmail`.

### El Directorio de Mail

Este directorio no existe por defecto, pero será creado para usted si ejecuta el comando `make:mail` de Artisan. El directorio `Mail` contiene todas sus [clases que representan emails](https://laravel.com/docs/10.x/mail) enviados por su aplicación. Los objetos Mail le permiten encapsular toda la lógica de construcción de un correo electrónico en una única y simple clase que puede ser enviada utilizando el método `Mail::send`.

### El Directorio Models

El directorio `Models` contiene todas tus [clases de modelos](https://laravel.com/docs/10.x/eloquent) Eloquent. El ORM de Eloquent incluido con Laravel proporciona una hermosa y simple implementación de ActiveRecord para trabajar con tu base de datos. Cada tabla de la base de datos tiene su correspondiente "Modelo" que se utiliza para interactuar con esa tabla. Los modelos te permiten consultar los datos de tus tablas, así como insertar nuevos registros en la tabla.

### El directorio Notifications

Este directorio no existe por defecto, pero será creado para usted si ejecuta el comando Artisan `make:notification`. El directorio `Notifications` contiene todas las [notificaciones](https://laravel.com/docs/10.x/notifications) "transaccionales" que son enviadas por tu aplicación, como simples notificaciones sobre eventos que ocurren dentro de tu aplicación. La función de notificación de Laravel abstrae el envío de notificaciones a través de una variedad de controladores como correo electrónico, Slack, SMS, o almacenados en una base de datos.

### El directorio Policies

Este directorio no existe por defecto, pero será creado por usted si ejecuta el comando Artisan `make:policy`. El directorio `Policies` contiene las clases de [políticas de autorización](https://laravel.com/docs/10.x/authorization) para su aplicación. Las políticas son usadas para determinar si un usuario puede realizar una acción dada contra un recurso.

### El Directorio Providers

El directorio `Providers` contiene todos los [proveedores de servicios](https://laravel.com/docs/10.x/providers) para tu aplicación. Los proveedores de servicios arrancan tu aplicación vinculando servicios en el contenedor de servicios, registrando eventos, o realizando cualquier otra tarea para preparar tu aplicación para las peticiones entrantes.

En una aplicación Laravel nueva, este directorio ya contendrá varios proveedores. Eres libre de añadir tus propios proveedores a este directorio según sea necesario.

### El directorio Rules

Este directorio no existe por defecto, pero será creado por usted si ejecuta el comando Artisan `make:rule`. El directorio `Rules` contiene los objetos de reglas de validación personalizados para su aplicación. Las reglas se utilizan para encapsular lógica de validación complicada en un objeto simple. Para mayor información, revisa la [documentación de validación](https://laravel.com/docs/10.x/validation).
