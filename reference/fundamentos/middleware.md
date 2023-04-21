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

