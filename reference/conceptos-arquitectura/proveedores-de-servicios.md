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

