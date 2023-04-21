# Middleware

## Introducción

Los middleware proporcionan un mecanismo conveniente para inspeccionar y filtrar las peticiones HTTP que entran en tu aplicación. Por ejemplo, Laravel incluye un middleware que verifica que el usuario de tu aplicación está autenticado. Si el usuario no está autenticado, el middleware redirigirá al usuario a la pantalla de login de tu aplicación. Sin embargo, si el usuario está autenticado, el middleware permitirá que la solicitud continúe en la aplicación.

Se puede escribir middleware adicional para realizar una variedad de tareas además de la autenticación. Por ejemplo, un middleware de registro podría registrar todas las peticiones entrantes a tu aplicación. Hay varios middleware incluidos en el framework Laravel, incluyendo middleware para autenticación y protección CSRF. Todos estos middleware se encuentran en el directorio `app/Http/Middleware`.

## Definiendo un middleware

Para crear un nuevo middleware, utiliza el comando Artisan `make:middleware`:

```sh
php artisan make:middleware EnsureTokenIsValid
```

Este comando colocará una nueva clase `EnsureTokenIsValid` dentro de tu directorio `app/Http/Middleware`. En este middleware, sólo permitiremos el acceso a la ruta si la entrada `token` suministrada coincide con un valor especificado. En caso contrario, redirigiremos a los usuarios al URI `home`:

```php
<?php
 
namespace App\Http\Middleware;
 
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;
 
class EnsureTokenIsValid
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('token') !== 'my-secret-token') {
            return redirect('home');
        }
 
        return $next($request);
    }
}
```

Como puedes ver, si el `token` dado no coincide con nuestro token secreto, el middleware devolverá una redirección HTTP al cliente; de lo contrario, la petición será pasada más adentro de la aplicación. Para pasar la petición más adentro de la aplicación (permitiendo al middleware "pass"), deberías llamar al callback `$next` con el `$request`.

Lo mejor es imaginar el middleware como una serie de "capas" por las que deben pasar las peticiones HTTP antes de llegar a la aplicación. Cada capa puede examinar la solicitud e incluso rechazarla por completo.

{% hint style="info" %}
Todos los middleware se resuelven a través del [contenedor de servicios](https://laravel.com/docs/10.x/container), por lo que puedes escribir cualquier dependencia que necesites en el constructor de un middleware.
{% endhint %}

#### Middleware y respuestas

Por supuesto, un middleware puede realizar tareas antes o después de pasar la petición a la aplicación. Por ejemplo, el siguiente middleware realizaría alguna tarea **antes** de que la solicitud sea gestionada por la aplicación:

```php
<?php
 
namespace App\Http\Middleware;
 
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;
 
class BeforeMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        // Perform action
 
        return $next($request);
    }
}
```

Sin embargo, este middleware realizaría su tarea **después** de que la solicitud sea gestionada por la aplicación:

```php
<?php
 
namespace App\Http\Middleware;
 
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;
 
class AfterMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);
 
        // Perform action
 
        return $response;
    }
}
```

## Registrando un Middleware

### Middleware global

Si quieres que un middleware se ejecute durante cada petición HTTP a tu aplicación, lista la clase del middleware en la propiedad `$middleware` de tu clase `app/Http/Kernel.php`.

### Asignación de middleware a rutas

Si desea asignar middleware a rutas específicas, puede invocar el método `middleware` al definir la ruta:

```php
use App\Http\Middleware\Authenticate;
 
Route::get('/profile', function () {
    // ...
})->middleware(Authenticate::class);
```

Puedes asignar múltiples middleware a la ruta pasando un array de nombres de middleware al método `middleware`:

```php
Route::get('/', function () {
    // ...
})->middleware([First::class, Second::class]);
```

Para mayor comodidad, puedes asignar alias al middleware en el fichero `app/Http/Kernel.php` de tu aplicación. Por defecto, la propiedad `$middlewareAliases` de esta clase contiene entradas para el middleware incluido con Laravel. Puedes añadir tu propio middleware a esta lista y asignarle un alias de tu elección:

```php
// Within App\Http\Kernel class...
 
protected $middlewareAliases = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
    'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
    'can' => \Illuminate\Auth\Middleware\Authorize::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
];
```

Una vez que se ha definido el alias de middleware en el kernel HTTP, se puede utilizar el alias al asignar middleware a las rutas:

```php
Route::get('/profile', function () {
    // ...
})->middleware('auth');
```

#### Excluyendo el middleware

Al asignar middleware a un grupo de rutas, puede que ocasionalmente necesites evitar que el middleware se aplique a una ruta individual dentro del grupo. Para ello puede utilizar el método `withoutMiddleware`:

```php
use App\Http\Middleware\EnsureTokenIsValid;
 
Route::middleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/', function () {
        // ...
    });
 
    Route::get('/profile', function () {
        // ...
    })->withoutMiddleware([EnsureTokenIsValid::class]);
});
```

También puede excluir un determinado conjunto de middleware de todo un [grupo](https://laravel.com/docs/10.x/routing#route-groups) de definiciones de ruta:

```php
use App\Http\Middleware\EnsureTokenIsValid;
 
Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/profile', function () {
        // ...
    });
});
```

El método `withoutMiddleware` sólo puede eliminar middleware de ruta y no se aplica a global middleware.

### Grupos de middleware

A veces es posible que desee agrupar varios middleware bajo una sola clave para que sea más fácil asignarlos a las rutas. Puedes conseguirlo usando la propiedad `$middlewareGroups` de tu kernel HTTP.

Laravel incluye grupos de middleware predefinidos `web` y `api` que contienen middleware común que puedes querer aplicar a tus rutas web y API. Recuerde, estos grupos de middleware se aplican automáticamente por su aplicación `App\Providers\RouteServiceProvider` proveedor de servicios a las rutas dentro de su correspondiente `web` y `api` archivos de ruta:

```php
/**
 * The application's route middleware groups.
 *
 * @var array
 */
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
 
    'api' => [
        \Illuminate\Routing\Middleware\ThrottleRequests::class.':api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];
```

Los grupos de middleware pueden asignarse a rutas y acciones de controlador utilizando la misma sintaxis que los middleware individuales. Una vez más, los grupos de middleware hacen que sea más conveniente asignar muchos middleware a una ruta a la vez:

```php
Route::get('/', function () {
    // ...
})->middleware('web');
 
Route::middleware(['web'])->group(function () {
    // ...
});
```

{% hint style="info" %}
Los grupos de middleware `web` y `api` se aplican automáticamente a los archivos correspondientes `routes/web.php` y `routes/api.php` de tu aplicación mediante el `App\Providers\RouteServiceProvider`.
{% endhint %}

### Clasificación de middleware

En raras ocasiones, puedes necesitar que tu middleware se ejecute en un orden específico pero no tener control sobre su orden cuando son asignados a la ruta. En este caso, puede especificar la prioridad de su middleware usando la propiedad `$middlewarePriority` de su archivo `app/Http/Kernel.php`. Esta propiedad puede no existir en su kernel HTTP por defecto. Si no existe, puede copiar su definición por defecto a continuación:

```php
/**
 * The priority-sorted list of middleware.
 *
 * This forces non-global middleware to always be in the given order.
 *
 * @var string[]
 */
protected $middlewarePriority = [
    \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
    \Illuminate\Cookie\Middleware\EncryptCookies::class,
    \Illuminate\Session\Middleware\StartSession::class,
    \Illuminate\View\Middleware\ShareErrorsFromSession::class,
    \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
    \Illuminate\Routing\Middleware\ThrottleRequests::class,
    \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
    \Illuminate\Contracts\Session\Middleware\AuthenticatesSessions::class,
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
    \Illuminate\Auth\Middleware\Authorize::class,
];
```

## Parámetros de middlewares

Los middleware también pueden recibir parámetros adicionales. Por ejemplo, si tu aplicación necesita verificar que el usuario autenticado tiene un "rol" determinado antes de realizar una acción determinada, podrías crear un middleware `EnsureUserHasRole` que reciba un nombre de rol como argumento adicional.

Los parámetros adicionales del middleware se pasarán al middleware después del argumento `$next`:

```php
<?php
 
namespace App\Http\Middleware;
 
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;
 
class EnsureUserHasRole
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }
 
        return $next($request);
    }
 
}
```

Los parámetros del middleware pueden especificarse al definir la ruta separando el nombre del middleware y los parámetros con un `:`. Los parámetros múltiples deben estar delimitados por comas:

```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware('role:editor');
```

## Middleware terminable

A veces un middleware puede necesitar hacer algún trabajo después de que la respuesta HTTP haya sido enviada al navegador. Si defines un método `terminate` en tu middleware y tu servidor web está usando FastCGI, el método `terminate` será llamado automáticamente después de que la respuesta sea enviada al navegador:

```php
<?php
 
namespace Illuminate\Session\Middleware;
 
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;
 
class TerminatingMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }
 
    /**
     * Handle tasks after the response has been sent to the browser.
     */
    public function terminate(Request $request, Response $response): void
    {
        // ...
    }
}
```

El método `terminate` debe recibir tanto la petición como la respuesta. Una vez que hayas definido un middleware terminable, debes añadirlo a la lista de rutas o middleware global en el archivo `app/Http/Kernel.php`.

Al llamar al método `terminate` en tu middleware, Laravel resolverá una nueva instancia del middleware desde el [service container](https://laravel.com/docs/10.x/container). Si deseas utilizar la misma instancia de middleware cuando los métodos `handle` y `terminate` son llamados, registra el middleware con el contenedor utilizando el método `singleton` del contenedor. Normalmente esto debería hacerse en el método `register` de tu `AppServiceProvider`:

```php
use App\Http\Middleware\TerminatingMiddleware;
 
/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(TerminatingMiddleware::class);
}
```
