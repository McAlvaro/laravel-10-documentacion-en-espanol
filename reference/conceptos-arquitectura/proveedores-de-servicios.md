# Proveedores de Servicios

## Introducción

Los proveedores de servicios son el lugar central de todo el bootstrapping de aplicaciones Laravel. Tu propia aplicación, así como todos los servicios centrales de Laravel, se arrancan a través de proveedores de servicios.

Pero, ¿qué queremos decir con "bootstrapped"? En general, nos referimos a **registrar** cosas, incluyendo el registro de enlaces de contenedores de servicios, escuchadores de eventos, middleware e incluso rutas. Los proveedores de servicios son el lugar central para configurar tu aplicación.

Si abres el archivo `config/app.php` incluido con Laravel, verás un array `providers`. Estas son todas las clases de proveedores de servicios que se cargarán en tu aplicación. Por defecto, un conjunto de proveedores de servicios del núcleo de Laravel se enumeran en esta matriz. Estos proveedores arrancan los componentes principales de Laravel, como el mailer, la cola, la caché y otros. Muchos de estos proveedores son proveedores "diferidos", lo que significa que no se cargarán en cada petición, sino sólo cuando los servicios que proporcionan sean realmente necesarios.

En esta visión general, usted aprenderá cómo escribir sus propios proveedores de servicios y registrarlos con su aplicación Laravel.

{% hint style="info" %}
Si quieres saber más sobre cómo Laravel gestiona las peticiones y funciona internamente, consulta nuestra documentación sobre el [ciclo de vida de las peticiones de Laravel](https://laravel.com/docs/10.x/lifecycle).
{% endhint %}

## Escribiendo Proveedores de Servicios

Todos los proveedores de servicios extienden la clase `Illuminate\Support\ServiceProvider`. La mayoría de los proveedores de servicios contienen un método `register` y un método `boot`. Dentro del método `register`, deberías **sólo enlazar cosas en el** [**contenedor de servicio**](https://laravel.com/docs/10.x/container). Nunca debes intentar registrar escuchadores de eventos, rutas o cualquier otra funcionalidad dentro del método `register`.

La CLI de Artisan puede generar un nuevo proveedor mediante el comando `make:provider`:

```sh
php artisan make:provider RiakServiceProvider
```

### El método register

Como se mencionó anteriormente, dentro del método `register`, sólo debe enlazar cosas en el [contenedor de servicio](https://laravel.com/docs/10.x/container). Nunca se debe intentar registrar escuchadores de eventos, rutas o cualquier otra funcionalidad dentro del método `register`. De lo contrario, puede utilizar accidentalmente un servicio proporcionado por un proveedor de servicios que aún no se ha cargado.

Echemos un vistazo a un proveedor de servicios básico. Dentro de cualquiera de sus métodos de proveedor de servicios, siempre tiene acceso a la propiedad `$app` que proporciona acceso al contenedor de servicios:

```php
<?php
 
namespace App\Providers;
 
use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\ServiceProvider;
 
class RiakServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection(config('riak'));
        });
    }
}
```

Este proveedor de servicios sólo define un método `register`, y utiliza ese método para definir una implementación de `App\Services\Riak\Connection` en el contenedor de servicios. Si aún no estás familiarizado con el contenedor de servicios de Laravel, consulta [su documentación](https://laravel.com/docs/10.x/container).

#### Propiedades `bindings` y `singletons`

Si tu proveedor de servicios registra muchos enlaces simples, puedes utilizar las propiedades `bindings` y `singletons` en lugar de registrar manualmente cada enlace del contenedor. Cuando el framework cargue el proveedor de servicios, buscará automáticamente estas propiedades y registrará sus enlaces:

```php
<?php
 
namespace App\Providers;
 
use App\Contracts\DowntimeNotifier;
use App\Contracts\ServerProvider;
use App\Services\DigitalOceanServerProvider;
use App\Services\PingdomDowntimeNotifier;
use App\Services\ServerToolsProvider;
use Illuminate\Support\ServiceProvider;
 
class AppServiceProvider extends ServiceProvider
{
    /**
     * All of the container bindings that should be registered.
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];
 
    /**
     * All of the container singletons that should be registered.
     *
     * @var array
     */
    public $singletons = [
        DowntimeNotifier::class => PingdomDowntimeNotifier::class,
        ServerProvider::class => ServerToolsProvider::class,
    ];
}
```

### El método Boot

Entonces, ¿qué pasa si necesitamos registrar un [view composer](https://laravel.com/docs/10.x/views#view-composers) dentro de nuestro proveedor de servicios? Esto debería hacerse dentro del método `boot`. **Este método es llamado después de que todos los demás proveedores de servicios han sido registrados**, lo que significa que tienes acceso a todos los demás servicios que han sido registrados por el framework:

```php
<?php
 
namespace App\Providers;
 
use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;
 
class ComposerServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        View::composer('view', function () {
            // ...
        });
    }
}
```

#### Inyección de dependencia en método Boot

Puedes inyectar dependencias para el método `boot` de tu proveedor de servicios. El [contenedor de servicios](https://laravel.com/docs/10.x/container) inyectará automáticamente cualquier dependencia que necesites:

```php
use Illuminate\Contracts\Routing\ResponseFactory;
 
/**
 * Bootstrap any application services.
 */
public function boot(ResponseFactory $response): void
{
    $response->macro('serialized', function (mixed $value) {
        // ...
    });
}
```

## Registrando Proveedores

Todos los proveedores de servicios se registran en el archivo de configuración `config/app.php`. Este fichero contiene un array `providers` donde puedes listar los nombres de las clases de tus proveedores de servicios. Por defecto, un conjunto de proveedores de servicios del núcleo de Laravel se enumeran en esta matriz. Estos proveedores arrancan los componentes principales de Laravel, como el mailer, la cola, la caché y otros.

Para registrar su proveedor, añádalo a la matriz:

```php
'providers' => [
    // Other Service Providers
 
    App\Providers\ComposerServiceProvider::class,
],
```

## Proveedores diferidos

Si tu proveedor **sólo** registra enlaces en el [contenedor de servicios](https://laravel.com/docs/10.x/container), puedes optar por aplazar su registro hasta que uno de los enlaces registrados sea realmente necesario. Aplazar la carga de un proveedor de este tipo mejorará el rendimiento de su aplicación, ya que no se carga desde el sistema de archivos en cada solicitud.

Laravel compila y almacena una lista de todos los servicios suministrados por los proveedores de servicios diferidos, junto con el nombre de su clase de proveedor de servicios. Entonces, sólo cuando se intenta resolver uno de estos servicios Laravel carga el proveedor de servicios.

Para aplazar la carga de un proveedor, implemente la interfaz `Illuminate\Contracts\Support\DeferrableProvider` y defina un método `provides`. El método `provides` debe devolver los enlaces del contenedor de servicios registrados por el proveedor:

```php
<?php
 
namespace App\Providers;
 
use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Contracts\Support\DeferrableProvider;
use Illuminate\Support\ServiceProvider;
 
class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection($app['config']['riak']);
        });
    }
 
    /**
     * Get the services provided by the provider.
     *
     * @return array<int, string>
     */
    public function provides(): array
    {
        return [Connection::class];
    }
}
```
