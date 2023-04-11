# Facades

## Introducción

A lo largo de la documentación de Laravel, verás ejemplos de código que interactúa con las características de Laravel a través de “facades". Las facades proporcionan una interfaz "estática" a las clases que están disponibles en el [contenedor de servicios de la aplicación](https://laravel.com/docs/10.x/container). Laravel viene con muchas facades que proporcionan acceso a casi todas las características de Laravel.

Las facades de Laravel sirven como "proxies estáticos" a las clases subyacentes en el contenedor de servicios, proporcionando el beneficio de una sintaxis breve y expresiva, manteniendo al mismo tiempo más capacidad de prueba y flexibilidad que los métodos estáticos tradicionales. No pasa nada si no entiendes del todo cómo funcionan las facades, simplemente déjate llevar y sigue aprendiendo sobre Laravel.

Todas las facades de Laravel están definidas en el espacio de nombres `Illuminate\Support\Facades`. Por lo tanto, podemos acceder fácilmente a una fachada así:

```php
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Route;
 
Route::get('/cache', function () {
    return Cache::get('key');
});
```

A lo largo de la documentación de Laravel, muchos de los ejemplos utilizarán fachadas para demostrar diversas características del framework.

#### Funciones Helpers

Para complementar las facades, Laravel ofrece una variedad de "funciones de ayuda" globales que hacen aún más fácil interactuar con las características comunes de Laravel. Algunas de las funciones de ayuda comunes con las que puedes interactuar son `view`, `response`, `url`, `config`, y más. Cada función helper ofrecida por Laravel está documentada con su característica correspondiente; sin embargo, una lista completa está disponible en la [documentación helper dedicada](https://laravel.com/docs/10.x/helpers).

Por ejemplo, en lugar de utilizar la fachada `Illuminate\Support\Facades\Response` para generar una respuesta JSON, podemos utilizar simplemente la función `response`. Dado que las funciones helper están disponibles globalmente, no es necesario importar ninguna clase para utilizarlas:

```php
use Illuminate\Support\Facades\Response;
 
Route::get('/users', function () {
    return Response::json([
        // ...
    ]);
});
 
Route::get('/users', function () {
    return response()->json([
        // ...
    ]);
});
```

## Cuándo utilizar Facades

Las facades tienen muchos beneficios. Proporcionan una sintaxis concisa y fácil de recordar que te permite utilizar las características de Laravel sin tener que recordar largos nombres de clases que tienen que ser inyectados o configurados manualmente. Además, debido a su uso único de los métodos dinámicos de PHP, son fáciles de probar.

Sin embargo, hay que tener cuidado al utilizar facades. El principal peligro de las facades es el "scope creep" de las clases. Dado que las facades son tan fáciles de usar y no requieren inyección, puede ser fácil dejar que tus clases sigan creciendo y usar muchas facades en una sola clase. Usando inyección de dependencia, este potencial es mitigado por la retroalimentación visual que un constructor grande te da de que tu clase está creciendo demasiado. Así que, cuando uses facades, presta especial atención al tamaño de tu clase para que su ámbito de responsabilidad se mantenga estrecho. Si tu clase está creciendo demasiado, considera dividirla en múltiples clases más pequeñas.

### Facades vs inyección de dependencia

Una de las principales ventajas de la inyección de dependencias es la posibilidad de intercambiar implementaciones de la clase inyectada. Esto es útil durante las pruebas, ya que puede inyectar un simulacro o stub y afirmar que varios métodos fueron llamados en el stub.

Típicamente, no sería posible hacer un mock o stub de un método de clase verdaderamente estático. Sin embargo, dado que las facades utilizan métodos dinámicos para delegar llamadas a métodos de objetos resueltos desde el contenedor de servicios, podemos probar las facades del mismo modo que probaríamos una instancia de clase inyectada. Por ejemplo, dada la siguiente ruta:

```php
use Illuminate\Support\Facades\Cache;
 
Route::get('/cache', function () {
    return Cache::get('key');
});
```

Usando los métodos de prueba de facade de Laravel, podemos escribir la siguiente prueba para verificar que el método `Cache::get` fue llamado con el argumento que esperábamos:

```php
use Illuminate\Support\Facades\Cache;
 
/**
 * A basic functional test example.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');
 
    $response = $this->get('/cache');
 
    $response->assertSee('value');
}
```

### Facades vs Funciones Helpers

Además de las facades, Laravel incluye una variedad de funciones "helper" que pueden realizar tareas comunes como generar vistas, disparar eventos, despachar trabajos o enviar respuestas HTTP. Muchas de estas funciones helper realizan la misma función que la facade correspondiente. Por ejemplo, esta llamada a la facade y la llamada al helper son equivalentes:

```php
return Illuminate\Support\Facades\View::make('profile');
 
return view('profile');
```

No hay ninguna diferencia práctica entre las facades y las funciones de ayuda. Al utilizar funciones auxiliares, puede probarlas exactamente igual que lo haría con la facade correspondiente. Por ejemplo, dada la siguiente ruta:

```php
Route::get('/cache', function () {
    return cache('key');
});
```

El helper `cache` va a llamar al método `get` de la clase subyacente a la facade `Cache`. Así que, aunque estemos usando la función helper, podemos escribir el siguiente test para verificar que el método fue llamado con el argumento que esperábamos:

```php
use Illuminate\Support\Facades\Cache;
 
/**
 * A basic functional test example.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');
 
    $response = $this->get('/cache');
 
    $response->assertSee('value');
}
```

## Cómo funcionan las Facades

En una aplicación Laravel, una facade es una clase que proporciona acceso a un objeto desde el contenedor. La maquinaria que hace que esto funcione está en la clase `Facade`. Las facades de Laravel, y cualquier facade personalizada que crees, extenderán la clase base `Illuminate\Support\Facades\Facade`.

La clase base `Facade` hace uso del método mágico `__callStatic()` para diferir las llamadas de tu facade a un objeto resuelto desde el contenedor. En el siguiente ejemplo, se realiza una llamada al sistema de caché de Laravel. Echando un vistazo a este código, uno podría asumir que el método estático `get` está siendo llamado en la clase `Cache`:

```php
<?php
 
namespace App\Http\Controllers;
 
use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Cache;
use Illuminate\View\View;
 
class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function showProfile(string $id): View
    {
        $user = Cache::get('user:'.$id);
 
        return view('profile', ['user' => $user]);
    }
}
```

Observe que cerca de la parte superior del archivo estamos "importando" la facade `Cache`. Esta facade sirve como proxy para acceder a la implementación subyacente de la interfaz `Illuminate\Contracts\Cache\Factory`. Cualquier llamada que hagamos usando la fachada será pasada a la instancia subyacente del servicio de caché de Laravel.

Si miramos esa clase `Illuminate\Support\Facades\Cache`, verás que no hay ningún método estático `get`:

```php
class Cache extends Facade
{
    /**
     * Get the registered name of the component.
     */
    protected static function getFacadeAccessor(): string
    {
        return 'cache';
    }
}
```

En su lugar, la facade `Cache` extiende la clase base `Facade` y define el método `getFacadeAccessor()`. La función de este método es devolver el nombre de un contenedor de servicios. Cuando un usuario hace referencia a cualquier método estático de la fachada `Cache`, Laravel resuelve el enlace `cache` del [contenedor de servicios](https://laravel.com/docs/10.x/container) y ejecuta el método solicitado (en este caso, `get`) contra ese objeto.

## Facades en tiempo real

Usando facades en tiempo real, puedes tratar cualquier clase de tu aplicación como si fuera una facade. Para ilustrar cómo se puede utilizar esto, vamos a examinar primero un poco de código que no utiliza fachadas en tiempo real. Por ejemplo, supongamos que nuestro modelo `Podcast` tiene un método `publish`. Sin embargo, para publicar el podcast, necesitamos inyectar una instancia `Publisher`:

```php
<?php
 
namespace App\Models;
 
use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;
 
class Podcast extends Model
{
    /**
     * Publish the podcast.
     */
    public function publish(Publisher $publisher): void
    {
        $this->update(['publishing' => now()]);
 
        $publisher->publish($this);
    }
}
```

Inyectar una implementación del publicador en el método nos permite probar fácilmente el método de forma aislada, ya que podemos simular el publicador inyectado. Sin embargo, nos obliga a pasar siempre una instancia del publicador cada vez que llamamos al método `publish`. Usando facade en tiempo real, podemos mantener la misma capacidad de prueba sin tener que pasar explícitamente una instancia de `Publisher`. Para generar una fachada en tiempo real, anteponga al espacio de nombres de la clase importada el prefijo `Facades`:

```php
<?php
 
namespace App\Models;
 
use Facades\App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;
 
class Podcast extends Model
{
    /**
     * Publish the podcast.
     */
    public function publish(): void
    {
        $this->update(['publishing' => now()]);
 
        Publisher::publish($this);
    }
}
```

Cuando se utiliza la facade en tiempo real, la implementación del editor se resolverá fuera del contenedor de servicios utilizando la parte de la interfaz o el nombre de la clase que aparece después del prefijo `Facades`. Al realizar pruebas, podemos utilizar los ayudantes de pruebas de fachada incorporados de Laravel para simular esta llamada al método:

```php
<?php
 
namespace Tests\Feature;
 
use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;
 
class PodcastTest extends TestCase
{
    use RefreshDatabase;
 
    /**
     * A test example.
     */
    public function test_podcast_can_be_published(): void
    {
        $podcast = Podcast::factory()->create();
 
        Publisher::shouldReceive('publish')->once()->with($podcast);
 
        $podcast->publish();
    }
}
```

## Referencia de clases de Facades

A continuación encontrará cada facade y su clase subyacente. Esta es una herramienta útil para profundizar rápidamente en la documentación de la API para una determinada raíz de facade. También se incluye la clave [service container binding](https://laravel.com/docs/10.x/container) cuando procede.

| Facade               | Class                                                                                                                                | Service Container Binding |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------- |
| App                  | [Illuminate\Foundation\Application](https://laravel.com/api/10.x/Illuminate/Foundation/Application.html)                             | `app`                     |
| Artisan              | [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/10.x/Illuminate/Contracts/Console/Kernel.html)                         | `artisan`                 |
| Auth                 | [Illuminate\Auth\AuthManager](https://laravel.com/api/10.x/Illuminate/Auth/AuthManager.html)                                         | `auth`                    |
| Auth (Instance)      | [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/10.x/Illuminate/Contracts/Auth/Guard.html)                                 | `auth.driver`             |
| Blade                | [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/10.x/Illuminate/View/Compilers/BladeCompiler.html)                 | `blade.compiler`          |
| Broadcast            | [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/10.x/Illuminate/Contracts/Broadcasting/Factory.html)             |                           |
| Broadcast (Instance) | [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/10.x/Illuminate/Contracts/Broadcasting/Broadcaster.html)     |                           |
| Bus                  | [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/10.x/Illuminate/Contracts/Bus/Dispatcher.html)                         |                           |
| Cache                | [Illuminate\Cache\CacheManager](https://laravel.com/api/10.x/Illuminate/Cache/CacheManager.html)                                     | `cache`                   |
| Cache (Instance)     | [Illuminate\Cache\Repository](https://laravel.com/api/10.x/Illuminate/Cache/Repository.html)                                         | `cache.store`             |
| Config               | [Illuminate\Config\Repository](https://laravel.com/api/10.x/Illuminate/Config/Repository.html)                                       | `config`                  |
| Cookie               | [Illuminate\Cookie\CookieJar](https://laravel.com/api/10.x/Illuminate/Cookie/CookieJar.html)                                         | `cookie`                  |
| Crypt                | [Illuminate\Encryption\Encrypter](https://laravel.com/api/10.x/Illuminate/Encryption/Encrypter.html)                                 | `encrypter`               |
| Date                 | [Illuminate\Support\DateFactory](https://laravel.com/api/10.x/Illuminate/Support/DateFactory.html)                                   | `date`                    |
| DB                   | [Illuminate\Database\DatabaseManager](https://laravel.com/api/10.x/Illuminate/Database/DatabaseManager.html)                         | `db`                      |
| DB (Instance)        | [Illuminate\Database\Connection](https://laravel.com/api/10.x/Illuminate/Database/Connection.html)                                   | `db.connection`           |
| Event                | [Illuminate\Events\Dispatcher](https://laravel.com/api/10.x/Illuminate/Events/Dispatcher.html)                                       | `events`                  |
| File                 | [Illuminate\Filesystem\Filesystem](https://laravel.com/api/10.x/Illuminate/Filesystem/Filesystem.html)                               | `files`                   |
| Gate                 | [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/10.x/Illuminate/Contracts/Auth/Access/Gate.html)                     |                           |
| Hash                 | [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/10.x/Illuminate/Contracts/Hashing/Hasher.html)                         | `hash`                    |
| Http                 | [Illuminate\Http\Client\Factory](https://laravel.com/api/10.x/Illuminate/Http/Client/Factory.html)                                   |                           |
| Lang                 | [Illuminate\Translation\Translator](https://laravel.com/api/10.x/Illuminate/Translation/Translator.html)                             | `translator`              |
| Log                  | [Illuminate\Log\LogManager](https://laravel.com/api/10.x/Illuminate/Log/LogManager.html)                                             | `log`                     |
| Mail                 | [Illuminate\Mail\Mailer](https://laravel.com/api/10.x/Illuminate/Mail/Mailer.html)                                                   | `mailer`                  |
| Notification         | [Illuminate\Notifications\ChannelManager](https://laravel.com/api/10.x/Illuminate/Notifications/ChannelManager.html)                 |                           |
| Password             | [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/10.x/Illuminate/Auth/Passwords/PasswordBrokerManager.html) | `auth.password`           |
| Password (Instance)  | [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/10.x/Illuminate/Auth/Passwords/PasswordBroker.html)               | `auth.password.broker`    |
| Pipeline (Instance)  | [Illuminate\Pipeline\Pipeline](https://laravel.com/api/10.x/Illuminate/Pipeline/Pipeline.html)                                       |                           |
| Queue                | [Illuminate\Queue\QueueManager](https://laravel.com/api/10.x/Illuminate/Queue/QueueManager.html)                                     | `queue`                   |
| Queue (Instance)     | [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/10.x/Illuminate/Contracts/Queue/Queue.html)                               | `queue.connection`        |
| Queue (Base Class)   | [Illuminate\Queue\Queue](https://laravel.com/api/10.x/Illuminate/Queue/Queue.html)                                                   |                           |
| Redirect             | [Illuminate\Routing\Redirector](https://laravel.com/api/10.x/Illuminate/Routing/Redirector.html)                                     | `redirect`                |
| Redis                | [Illuminate\Redis\RedisManager](https://laravel.com/api/10.x/Illuminate/Redis/RedisManager.html)                                     | `redis`                   |
| Redis (Instance)     | [Illuminate\Redis\Connections\Connection](https://laravel.com/api/10.x/Illuminate/Redis/Connections/Connection.html)                 | `redis.connection`        |
| Request              | [Illuminate\Http\Request](https://laravel.com/api/10.x/Illuminate/Http/Request.html)                                                 | `request`                 |
| Response             | [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/10.x/Illuminate/Contracts/Routing/ResponseFactory.html)       |                           |
| Response (Instance)  | [Illuminate\Http\Response](https://laravel.com/api/10.x/Illuminate/Http/Response.html)                                               |                           |
| Route                | [Illuminate\Routing\Router](https://laravel.com/api/10.x/Illuminate/Routing/Router.html)                                             | `router`                  |
| Schema               | [Illuminate\Database\Schema\Builder](https://laravel.com/api/10.x/Illuminate/Database/Schema/Builder.html)                           |                           |
| Session              | [Illuminate\Session\SessionManager](https://laravel.com/api/10.x/Illuminate/Session/SessionManager.html)                             | `session`                 |
| Session (Instance)   | [Illuminate\Session\Store](https://laravel.com/api/10.x/Illuminate/Session/Store.html)                                               | `session.store`           |
| Storage              | [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/10.x/Illuminate/Filesystem/FilesystemManager.html)                 | `filesystem`              |
| Storage (Instance)   | [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/10.x/Illuminate/Contracts/Filesystem/Filesystem.html)           | `filesystem.disk`         |
| URL                  | [Illuminate\Routing\UrlGenerator](https://laravel.com/api/10.x/Illuminate/Routing/UrlGenerator.html)                                 | `url`                     |
| Validator            | [Illuminate\Validation\Factory](https://laravel.com/api/10.x/Illuminate/Validation/Factory.html)                                     | `validator`               |
| Validator (Instance) | [Illuminate\Validation\Validator](https://laravel.com/api/10.x/Illuminate/Validation/Validator.html)                                 |                           |
| View                 | [Illuminate\View\Factory](https://laravel.com/api/10.x/Illuminate/View/Factory.html)                                                 | `view`                    |
| View (Instance)      | [Illuminate\View\View](https://laravel.com/api/10.x/Illuminate/View/View.html)                                                       |                           |
| Vite                 | [Illuminate\Foundation\Vite](https://laravel.com/api/10.x/Illuminate/Foundation/Vite.html)                                           |                           |
