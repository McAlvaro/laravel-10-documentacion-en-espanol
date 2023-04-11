# Contenedor de Servicio

## Introducción

El contenedor de servicios de Laravel es una potente herramienta para gestionar las dependencias de clases y realizar la inyección de dependencias. La inyección de dependencias es una frase elegante que esencialmente significa esto: las dependencias de clase se "inyectan" en la clase a través del constructor o, en algunos casos, métodos "setter".

Veamos un ejemplo sencillo:

```php
<?php
 
namespace App\Http\Controllers;
 
use App\Http\Controllers\Controller;
use App\Repositories\UserRepository;
use App\Models\User;
use Illuminate\View\View;
 
class UserController extends Controller
{
    /**
     * Create a new controller instance.
     */
    public function __construct(
        protected UserRepository $users,
    ) {}
 
    /**
     * Show the profile for the given user.
     */
    public function show(string $id): View
    {
        $user = $this->users->find($id);
 
        return view('user.profile', ['user' => $user]);
    }
}
```

En este ejemplo, el `UserController` necesita recuperar usuarios de una fuente de datos. Por lo tanto, vamos a **inyectar** un servicio que sea capaz de recuperar usuarios. En este contexto, lo más probable es que nuestro `UserRepository` utilice [Eloquent](https://laravel.com/docs/10.x/eloquent) para recuperar la información de los usuarios de la base de datos. Sin embargo, dado que el repositorio está inyectado, podemos cambiarlo fácilmente por otra implementación. También podemos fácilmente "mock", o crear una implementación ficticia del `UserRepository` cuando probamos nuestra aplicación.

Un profundo conocimiento del contenedor de servicios de Laravel es esencial para construir una aplicación potente y de gran tamaño, así como para contribuir al propio núcleo de Laravel.

### Resolución de Configuración Cero

Si una clase no tiene dependencias o sólo depende de otras clases concretas (no de interfaces), no es necesario indicar al contenedor cómo resolver esa clase. Por ejemplo, puede colocar el siguiente código en su archivo `routes/web.php`:

```php
<?php
 
class Service
{
    // ...
}
 
Route::get('/', function (Service $service) {
    die(get_class($service));
});
```

En este ejemplo, al pulsar la ruta `/` de tu aplicación se resolverá automáticamente la clase `Service` y se inyectará en el manejador de tu ruta. Esto cambia las reglas del juego. Significa que puedes desarrollar tu aplicación y aprovechar las ventajas de la inyección de dependencias sin preocuparte por los archivos de configuración hinchados.

Afortunadamente, muchas de las clases que escribirás cuando construyas una aplicación Laravel reciben automáticamente sus dependencias a través del contenedor, incluyendo [controllers](https://laravel.com/docs/10.x/controllers), [event listeners](https://laravel.com/docs/10.x/events), [middleware](https://laravel.com/docs/10.x/middleware), y más. Además, puedes escribir dependencias en el método `handle` de [queued jobs](https://laravel.com/docs/10.x/queues). Una vez que pruebas el poder de la inyección de dependencias automática y sin configuración, parece imposible desarrollar sin ella.

### Cuándo Utilizar el Contenedor

Gracias a la resolución de configuración cero, a menudo escribirás dependencias en rutas, controladores, escuchadores de eventos, y en otros lugares sin tener que interactuar manualmente con el contenedor. Por ejemplo, puedes escribir el objeto `Illuminate\Http\Request` en tu definición de ruta para poder acceder fácilmente a la petición actual. A pesar de que nunca tenemos que interactuar con el contenedor para escribir este código, es la gestión de la inyección de estas dependencias detrás de las escenas:

```php
use Illuminate\Http\Request;
 
Route::get('/', function (Request $request) {
    // ...
});
```

En muchos casos, gracias a la inyección automática de dependencias y [facades](https://laravel.com/docs/10.x/facades), puedes construir aplicaciones Laravel sin **nunca** vincular o resolver manualmente nada desde el contenedor. **Entonces, ¿cuándo interactuarías manualmente con el contenedor?** Examinemos dos situaciones.

En primer lugar, si escribes una clase que implementa una interfaz y deseas escribir esa interfaz en una ruta o constructor de clase, debes indicar al contenedor cómo resolver esa interfaz. En segundo lugar, si estás [escribiendo un paquete Laravel](https://laravel.com/docs/10.x/packages) que planeas compartir con otros desarrolladores Laravel, puede que necesites enlazar los servicios de tu paquete en el contenedor.

## Binding

### Conceptos Básicos de Binding

#### Simple Bindings

Casi todas las vinculaciones del contenedor de servicios se registrarán en [service providers](https://laravel.com/docs/10.x/providers), por lo que la mayoría de estos ejemplos mostrarán el uso del contenedor en ese contexto.

Dentro de un proveedor de servicios, siempre tienes acceso al contenedor a través de la propiedad `$this->app`. Podemos registrar un enlace utilizando el método `bind`, pasando el nombre de la clase o interfaz que deseamos registrar junto con un closure que devuelva una instancia de la clase:

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;
 
$this->app->bind(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

Tenga en cuenta que recibimos el propio contenedor como argumento para el resolver. A continuación, podemos utilizar el contenedor para resolver las subdependencias del objeto que estamos construyendo.

Como se ha mencionado, normalmente interactuará con el contenedor dentro de los proveedores de servicios; sin embargo, si desea interactuar con el contenedor fuera de un proveedor de servicios, puede hacerlo a través de la `App` [Facade](https://laravel.com/docs/10.x/facades):

```php
use App\Services\Transistor;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\App;
 
App::bind(Transistor::class, function (Application $app) {
    // ...
});
```

{% hint style="info" %}
No es necesario enlazar clases en el contenedor si no dependen de ninguna interfaz. El contenedor no necesita instrucciones sobre cómo construir estos objetos, ya que puede resolverlos automáticamente mediante reflexión.
{% endhint %}

#### Binding A Singleton

El método `singleton` vincula una clase o interfaz al contenedor que sólo debe resolverse una vez. Una vez resuelto un enlace singleton, se devolverá la misma instancia de objeto en las siguientes llamadas al contenedor:

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;
 
$this->app->singleton(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

#### Binding Scoped Singletons

El método `scoped` vincula una clase o interfaz en el contenedor que sólo debe resolverse una vez dentro de un determinado ciclo de vida de solicitud / trabajo de Laravel. Si bien este método es similar al método `singleton`, las instancias registradas utilizando el método `scoped` se vaciarán cada vez que la aplicación Laravel inicie un nuevo "ciclo de vida", como cuando un trabajador [Laravel Octane](https://laravel.com/docs/10.x/octane) procesa una nueva solicitud o cuando un trabajador [queue worker](https://laravel.com/docs/10.x/queues) procesa un nuevo trabajo:

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;
 
$this->app->scoped(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

#### Binding Instances

También puede enlazar una instancia de objeto existente en el contenedor utilizando el método `instance`. La instancia dada siempre será devuelta en las siguientes llamadas al contenedor:

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
 
$service = new Transistor(new PodcastParser);
 
$this->app->instance(Transistor::class, $service);
```

### Binding de Interfaces a Implementaciones

Una característica muy potente del contenedor de servicios es su capacidad para vincular una interfaz a una implementación dada. Por ejemplo, supongamos que tenemos una interfaz `EventPusher` y una implementación `RedisEventPusher`. Una vez que hemos codificado nuestra implementación `RedisEventPusher` de esta interfaz, podemos registrarla con el contenedor de servicios de la siguiente manera:

```php
use App\Contracts\EventPusher;
use App\Services\RedisEventPusher;
 
$this->app->bind(EventPusher::class, RedisEventPusher::class);
```

Esta sentencia indica al contenedor que debe inyectar el `RedisEventPusher` cuando una clase necesite una implementación de `EventPusher`. Ahora podemos inyectar la interfaz `EventPusher` en el constructor de una clase que sea resuelta por el contenedor. Recuerda, controladores, escuchadores de eventos, middleware, y varios otros tipos de clases dentro de las aplicaciones Laravel siempre se resuelven utilizando el contenedor:

```php
use App\Contracts\EventPusher;
 
/**
 * Create a new class instance.
 */
public function __construct(
    protected EventPusher $pusher
) {}
```

### Binding contextual

A veces puede tener dos clases que utilizan la misma interfaz, pero desea inyectar diferentes implementaciones en cada clase. Por ejemplo, dos controladores pueden depender de diferentes implementaciones del `Illuminate\Contracts\Filesystem\Filesystem` [contract](https://laravel.com/docs/10.x/contracts). Laravel proporciona una interfaz sencilla y fluida para definir este comportamiento:

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\UploadController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;
use Illuminate\Support\Facades\Storage;
 
$this->app->when(PhotoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('local');
          });
 
$this->app->when([VideoController::class, UploadController::class])
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });
```

### Binding Primitives

A veces puedes tener una clase que recibe algunas clases inyectadas, pero también necesita un valor primitivo inyectado como un entero. Puede utilizar fácilmente la vinculación contextual para inyectar cualquier valor que su clase pueda necesitar:

```php
use App\Http\Controllers\UserController;
 
$this->app->when(UserController::class)
          ->needs('$variableName')
          ->give($value);
```

A veces una clase puede depender de un array de instancias tagged. Usando el método `giveTagged`, puedes inyectar fácilmente todos los enlaces del contenedor con esa etiqueta:

```php
$this->app->when(ReportAggregator::class)
    ->needs('$reports')
    ->giveTagged('reports');
```

Si necesitas inyectar un valor de uno de los ficheros de configuración de tu aplicación, puedes utilizar el método `giveConfig`:

```php
$this->app->when(ReportAggregator::class)
    ->needs('$timezone')
    ->giveConfig('app.timezone');
```

### Binding de variables tipificadas

Ocasionalmente, puedes tener una clase que recibe un array de objetos tipados usando un argumento variadic del constructor:

```php
<?php
 
use App\Models\Filter;
use App\Services\Logger;
 
class Firewall
{
    /**
     * The filter instances.
     *
     * @var array
     */
    protected $filters;
 
    /**
     * Create a new class instance.
     */
    public function __construct(
        protected Logger $logger,
        Filter ...$filters,
    ) {
        $this->filters = $filters;
    }
}
```

Utilizando la vinculación contextual, puede resolver esta dependencia proporcionando al método `give` un closure que devuelva un array de instancias de `Filter` resueltas:

```php
$this->app->when(Firewall::class)
          ->needs(Filter::class)
          ->give(function (Application $app) {
                return [
                    $app->make(NullFilter::class),
                    $app->make(ProfanityFilter::class),
                    $app->make(TooLongFilter::class),
                ];
          });
```

Para mayor comodidad, también puede proporcionar una matriz de nombres de clase que el contenedor resolverá cada vez que `Firewall` necesite instancias de `Filter`:

```php
$this->app->when(Firewall::class)
          ->needs(Filter::class)
          ->give([
              NullFilter::class,
              ProfanityFilter::class,
              TooLongFilter::class,
          ]);
```

#### Dependencias de etiquetas variádicas

A veces una clase puede tener una dependencia variada que se indica como una clase determinada (`Informe ...$informes`). Utilizando los métodos `needs` y `giveTagged`, puede inyectar fácilmente todos los enlaces de contenedor con ese tag para la dependencia dada:

```php
$this->app->when(ReportAggregator::class)
    ->needs(Report::class)
    ->giveTagged('reports');
```

### Etiquetado

Ocasionalmente, puede que necesites resolver todas las vinculaciones de una determinada "categoría". Por ejemplo, puede que estés construyendo un analizador de informes que reciba un array de diferentes implementaciones de la interfaz `Report`. Después de registrar las implementaciones de `Report`, puedes asignarles una etiqueta utilizando el método `tag`:

```php
$this->app->bind(CpuReport::class, function () {
    // ...
});
 
$this->app->bind(MemoryReport::class, function () {
    // ...
});
 
$this->app->tag([CpuReport::class, MemoryReport::class], 'reports');
```

Una vez etiquetados los servicios, puedes resolverlos todos fácilmente a través del método `tagged` del contenedor:

```php
$this->app->bind(ReportAnalyzer::class, function (Application $app) {
    return new ReportAnalyzer($app->tagged('reports'));
});
```

### Extender Bindings

El método `extend` permite modificar los servicios resueltos. Por ejemplo, cuando se resuelve un servicio, puedes ejecutar código adicional para decorar o configurar el servicio. El método `extend` acepta dos argumentos, la clase de servicio que estás extendiendo y un cierre que debe devolver el servicio modificado. El cierre recibe el servicio que se está resolviendo y la instancia del contenedor:

```php
$this->app->extend(Service::class, function (Service $service, Application $app) {
    return new DecoratedService($service);
});
```

