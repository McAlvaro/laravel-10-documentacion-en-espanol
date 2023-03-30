# Configuración

## Introducción

Todos los archivos de configuración para el framework Laravel se almacenan en el directorio `config`. Cada opción está documentada, así que no dudes en echar un vistazo a los archivos y familiarizarte con las opciones disponibles.

Estos archivos de configuración le permiten configurar cosas como la información de conexión a su base de datos, la información de su servidor de correo, así como otros valores de configuración básicos como la zona horaria de su aplicación y la clave de encriptación.

### Visión general de la aplicación

¿Tienes prisa? Puedes obtener una rápida visión general de la configuración, controladores y entorno de tu aplicación a través del comando `about` Artisan:

```shell
php artisan about
```

Si sólo le interesa una sección concreta de la vista general de la aplicación, puede filtrarla con la opción `--only`:

```shell
php artisan about --only=environmenth
```

## Configuración del entorno

A menudo es útil tener diferentes valores de configuración basados en el entorno donde se ejecuta la aplicación. Por ejemplo, es posible que desee utilizar un controlador de caché diferente a nivel local que en su servidor de producción.

Para hacer esto más fácil, Laravel utiliza la librería PHP [DotEnv](https://github.com/vlucas/phpdotenv). En una nueva instalación de Laravel, el directorio raíz de tu aplicación contendrá un archivo `.env.example` que define muchas variables de entorno comunes. Durante el proceso de instalación de Laravel, este archivo se copiará automáticamente a `.env`.

El archivo `.env` por defecto de Laravel contiene algunos valores de configuración comunes que pueden diferir en función de si tu aplicación se ejecuta localmente o en un servidor web de producción. Estos valores se recuperan de varios archivos de configuración de Laravel dentro del directorio `config` utilizando la función `env` de Laravel.

Si estás desarrollando con un equipo, puede que quieras seguir incluyendo un archivo `.env.example` con tu aplicación. Al poner valores de marcador de posición en el archivo de configuración de ejemplo, otros desarrolladores de tu equipo pueden ver claramente qué variables de entorno son necesarias para ejecutar tu aplicación.

{% hint style="info" %}
Cualquier variable de su archivo `.env` puede ser anulada por variables de entorno externas, como las variables de entorno a nivel de servidor o a nivel de sistema.
{% endhint %}

#### Seguridad de Archivos de Entorno

Su archivo `.env` no debe ser enviado al control de código fuente de su aplicación, ya que cada desarrollador / servidor que utilice su aplicación podría requerir una configuración de entorno diferente. Además, esto supondría un riesgo de seguridad en caso de que un intruso accediera a tu repositorio de control de código fuente, ya que cualquier credencial sensible quedaría expuesta.

Sin embargo, es posible cifrar tu fichero de entorno usando el [cifrado de entorno](https://laravel.com/docs/10.x/configuration#encrypting-environment-files) integrado de Laravel. Los archivos de entorno cifrados se pueden colocar en el control de código fuente de forma segura.

#### Archivos de entorno adicionales

Antes de cargar las variables de entorno de tu aplicación, Laravel determina si se ha proporcionado externamente una variable de entorno `APP_ENV` o si se ha especificado el argumento CLI `--env`. Si es así, Laravel intentará cargar un archivo `.env.[APP_ENV]` si existe. Si no existe, se cargará el fichero `.env` por defecto.

### Tipos de variables de entorno

Todas las variables en sus archivos `.env` son típicamente analizadas como cadenas, por lo que se han creado algunos valores reservados para permitirle devolver un rango más amplio de tipos desde la función `env()`:

| .env Value | env() Value  |
| ---------- | ------------ |
| true       | (bool) true  |
| (true)     | (bool) true  |
| false      | (bool) false |
| (false)    | (bool) false |
| empty      | (string) ''  |
| (empty)    | (string) ''  |
| null       | (null) null  |
| (null)     | (null) null  |

Si necesita definir una variable de entorno con un valor que contenga espacios, puede hacerlo encerrando el valor entre comillas dobles:

```php
APP_NAME="My Application"
```

### Recuperación de la configuración del entorno

Todas las variables listadas en el fichero `.env` serán cargadas en el superglobal PHP `$_ENV` cuando su aplicación reciba una petición. Sin embargo, puedes utilizar la función `env` para recuperar los valores de estas variables en tus archivos de configuración. De hecho, si revisas los archivos de configuración de Laravel, notarás que muchas de las opciones ya utilizan esta función:

```php
'debug' => env('APP_DEBUG', false),
```

El segundo valor que se pasa a la función `env` es el "valor por defecto". Este valor se devolverá si no existe ninguna variable de entorno para la clave dada.

### Determinar el entorno actual

El entorno actual de la aplicación se determina a través de la variable `APP_ENV` de tu fichero `.env`. Puedes acceder a este valor a través del método `environment` de la [Facade](https://laravel.com/docs/10.x/facades) `App`:

```php
use Illuminate\Support\Facades\App;
 
$environment = App::environment();
```

También puede pasar argumentos al método `environment` para determinar si el entorno coincide con un valor dado. El método devolverá `true` si el entorno coincide con alguno de los valores dados:

```php
if (App::environment('local')) {
    // The environment is local
}
 
if (App::environment(['local', 'staging'])) {
    // The environment is either local OR staging...
}
```

{% hint style="info" %}
La detección del entorno actual de la aplicación puede anularse definiendo una variable de entorno `APP_ENV` a nivel de servidor.
{% endhint %}

### Cifrar archivos de entorno

Los archivos de entorno sin cifrar nunca deben almacenarse en el control de código fuente. Sin embargo, Laravel te permite cifrar tus archivos de entorno para que puedan ser añadidos de forma segura al control de código fuente con el resto de tu aplicación.

#### Cifrado

Para cifrar un archivo de entorno, puede utilizar el comando `env:encrypt`:

```shell
php artisan env:encrypt
```

Ejecutar el comando `env:encrypt` encriptará tu archivo `.env` y colocará el contenido encriptado en un archivo `.env.encrypted`. La clave de descifrado se muestra en la salida del comando y debe guardarse en un gestor de contraseñas seguro. Si desea proporcionar su propia clave de cifrado puede utilizar la opción `--key` al invocar el comando:

```shell
php artisan env:encrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

{% hint style="info" %}
La longitud de la clave proporcionada debe coincidir con la longitud de clave requerida por el cifrado utilizado. Por defecto, Laravel utilizará el cifrado `AES-256-CBC` que requiere una clave de 32 caracteres. Eres libre de utilizar cualquier [cifrado](https://laravel.com/docs/10.x/encryption) soportado por el cifrador de Laravel pasando la opción `--cipher` al invocar el comando.
{% endhint %}

#### Descifrado

Para desencriptar un fichero de entorno, puedes usar el comando `env:decrypt`. Este comando requiere una clave de descifrado, que Laravel recuperará de la variable de entorno `LARAVEL_ENV_ENCRYPTION_KEY`:

```shell
php artisan env:decrypt
```

O bien, la clave puede proporcionarse directamente al comando mediante la opción `--key`:

```shell
php artisan env:decrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

Cuando se invoca el comando `env:decrypt`, Laravel descifrará el contenido del fichero `.env.encrypted` y colocará el contenido descifrado en el fichero `.env`.

Se puede proporcionar la opción `--cipher` al comando `env:decrypt` para utilizar un cifrado personalizado:

```shell
php artisan env:decrypt --key=qUWuNRdfuImXcKxZ --cipher=AES-128-CBC
```

Si su aplicación tiene varios archivos de entorno, como `.env` y `.env.staging`, puede especificar el archivo de entorno que debe descifrarse proporcionando el nombre del entorno mediante la opción `--env`:

```shell
php artisan env:decrypt --env=staging
```

Para sobrescribir un archivo de entorno existente, puede proporcionar la opción `--force` al comando `env:decrypt`:

```sh
php artisan env:decrypt --force
```

## Acceder a los valores de configuración

Puede acceder fácilmente a sus valores de configuración utilizando la función global `config` desde cualquier parte de su aplicación. Puede acceder a los valores de configuración utilizando la sintaxis "dot", que incluye el nombre del archivo y la opción a la que desea acceder. También se puede especificar un valor por defecto, que se devolverá si la opción de configuración no existe:

```php
$value = config('app.timezone');
 
// Retrieve a default value if the configuration value does not exist...
$value = config('app.timezone', 'Asia/Seoul');
```

Para establecer valores de configuración en tiempo de ejecución, pase un array a la función `config`:

```php
config(['app.timezone' => 'America/Chicago']);
```

## Almacenamiento en caché de la configuración

Para aumentar la velocidad de tu aplicación, debes cachear todos tus archivos de configuración en un solo archivo usando el comando `config:cache` de Artisan. Esto combinará todas las opciones de configuración para su aplicación en un solo archivo que puede ser cargado rápidamente por el framework.

Normalmente debería ejecutar el comando `php artisan config:cache` como parte de su proceso de despliegue de producción. El comando no debe ser ejecutado durante el desarrollo local, ya que las opciones de configuración necesitarán ser cambiadas frecuentemente durante el curso del desarrollo de su aplicación.

Una vez que la configuración ha sido almacenada en caché, el archivo `.env` de tu aplicación no será cargado por el framework durante las peticiones o comandos de Artisan; por lo tanto, la función `env` sólo devolverá variables de entorno externas, a nivel de sistema.

Por esta razón, debes asegurarte de que sólo llamas a la función `env` desde dentro de los ficheros de configuración (`config`) de tu aplicación. Puedes ver muchos ejemplos de esto examinando los archivos de configuración por defecto de Laravel.

Se puede acceder a los valores de configuración desde cualquier parte de su aplicación utilizando la función `config` [descrita anteriormente](https://laravel.com/docs/10.x/configuration#accessing-configuration-values).

{% hint style="info" %}
Si ejecutas el comando `config:cache` durante tu proceso de despliegue, debes asegurarte de que sólo estás llamando a la función `env` desde dentro de tus ficheros de configuración. Una vez que la configuración se ha almacenado en caché, el archivo `.env` no se cargará; por lo tanto, la función `env` sólo devolverá variables de entorno externas, a nivel de sistema.
{% endhint %}

## Modo Debug

La opción `debug` de su fichero de configuración `config/app.php` determina cuánta información sobre un error se muestra realmente al usuario. Por defecto, esta opción está configurada para respetar el valor de la variable de entorno `APP_DEBUG`, que se almacena en su archivo `.env`.

Para el desarrollo local, debe establecer la variable de entorno `APP_DEBUG` en `true`. **En su entorno de producción, este valor debe ser siempre `false`. Si la variable se establece en `true` en producción, corres el riesgo de exponer valores de configuración sensibles a los usuarios finales de tu aplicación.**

## Modo de mantenimiento

Cuando su aplicación está en modo de mantenimiento, se mostrará una vista personalizada para todas las peticiones que entren en su aplicación. Esto hace que sea fácil de "desactivar" su aplicación mientras se está actualizando o cuando se está realizando el mantenimiento. Se incluye una comprobación del modo de mantenimiento en la pila de middleware predeterminada para su aplicación. Si la aplicación está en modo de mantenimiento, una instancia `Symfony\Component\HttpKernel\Exception\HttpException` será lanzada con un código de estado 503.

Para activar el modo de mantenimiento, ejecute el comando Artisan `down`:

```sh
php artisan down
```

Si desea que la cabecera HTTP `Refresh` se envíe con todas las respuestas del modo de mantenimiento, puede proporcionar la opción `refresh` al invocar el comando `down`. La cabecera `Refresh` indicará al navegador que actualice automáticamente la página tras el número de segundos especificado:

```sh
php artisan down --refresh=15
```

También puede proporcionar una opción `retry` al comando `down`, que se establecerá como valor de la cabecera HTTP `Retry-After`, aunque los navegadores suelen ignorar esta cabecera:

```shell
php artisan down --retry=60
```

#### Evitar el modo de mantenimiento

Para permitir que el modo de mantenimiento sea evitado usando un token secreto, puedes usar la opción `secret` para especificar un token de evasión del modo de mantenimiento:

```shell
php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"
```

Después de poner la aplicación en modo de mantenimiento, puede navegar a la URL de la aplicación que coincida con este token y Laravel emitirá una cookie de bypass de modo de mantenimiento a su navegador:

```shell
https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515
```

Cuando acceda a esta ruta oculta, será redirigido a la ruta `/` de la aplicación. Una vez que la cookie haya sido emitida a su navegador, podrá navegar por la aplicación normalmente como si no estuviera en modo de mantenimiento.

{% hint style="info" %}
Su secreto de modo de mantenimiento debe consistir normalmente en caracteres alfanuméricos y, opcionalmente, guiones. Debe evitar utilizar caracteres que tengan un significado especial en las URL, como `?`.
{% endhint %}

#### Vista previa del modo de mantenimiento

Si utilizas el comando `php artisan down` durante el despliegue, tus usuarios pueden encontrarse ocasionalmente con errores si acceden a la aplicación mientras tus dependencias de Composer u otros componentes de la infraestructura se están actualizando. Esto ocurre porque una parte significativa del framework Laravel debe arrancar para determinar que tu aplicación está en modo mantenimiento y renderizar la vista de modo mantenimiento usando el motor de plantillas.

Por esta razón, Laravel te permite pre-renderizar una vista en modo mantenimiento que será devuelta al principio del ciclo de petición. Esta vista se renderiza antes de que ninguna de las dependencias de tu aplicación se haya cargado. Puedes pre-renderizar una plantilla de tu elección usando la opción `render` del comando `down`:

```shell
php artisan down --render="errors::503"
```

#### Redireccionamiento de solicitudes en modo de mantenimiento

Mientras esté en modo mantenimiento, Laravel mostrará la vista de modo mantenimiento para todas las URLs de la aplicación a las que el usuario intente acceder. Si lo desea, puede indicar a Laravel que redirija todas las peticiones a una URL específica. Esto puede lograrse utilizando la opción `redirect`. Por ejemplo, es posible que desee redirigir todas las solicitudes a la `/` URI:

```shell
php artisan down --redirect=/
```

#### Desactivar el modo de mantenimiento

Para desactivar el modo de mantenimiento, utilice el comando `up`:

```shell
php artisan up
```

{% hint style="info" %}
Puede personalizar la plantilla predeterminada del modo de mantenimiento definiendo su propia plantilla en `resources/views/errors/503.blade.php`.
{% endhint %}

#### Modo de mantenimiento y colas

Mientras su aplicación esté en modo de mantenimiento, no se gestionarán [trabajos en cola](https://laravel.com/docs/10.x/queues). Los trabajos seguirán gestionándose con normalidad una vez que la aplicación salga del modo de mantenimiento.

#### Alternativas al modo de mantenimiento

Dado que el modo de mantenimiento requiere que su aplicación tenga varios segundos de tiempo de inactividad, considere alternativas como [Laravel Vapor](https://vapor.laravel.com) y [Envoyer](https://envoyer.io) para lograr un despliegue sin tiempo de inactividad con Laravel.
