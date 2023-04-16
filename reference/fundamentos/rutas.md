# Rutas

## Rutas Básicas

Las rutas más básicas de Laravel aceptan un URI y un cierre, proporcionando un método muy simple y expresivo de definir rutas y comportamientos sin complicados archivos de configuración de rutas:

```php
use Illuminate\Support\Facades\Route;
 
Route::get('/greeting', function () {
    return 'Hello World';
});
```

### Los archivos de ruta por defecto

Todas las rutas Laravel se definen en los archivos de ruta, que se encuentran en el directorio `routes`. Estos archivos son cargados automáticamente por el `App\Providers\RouteServiceProvider` de tu aplicación. El archivo `routes/web.php` define las rutas que son para su interfaz web. A estas rutas se les asigna el grupo de middleware `web`, que proporciona características como el estado de sesión y la protección CSRF. Las rutas en `routes/api.php` no tienen estado y se les asigna el grupo de middleware `api`.

Para la mayoría de las aplicaciones, usted comenzará definiendo rutas en su archivo `routes/web.php`. Puede acceder a las rutas definidas en `routes/web.php` introduciendo la URL de la ruta definida en su navegador. Por ejemplo, puede acceder a la siguiente ruta navegando a `http://example.com/user` en su navegador:

```php
use App\Http\Controllers\UserController;
 
Route::get('/user', [UserController::class, 'index']);
```

Las rutas definidas en el archivo `routes/api.php` están anidadas dentro de un grupo de rutas por el `RouteServiceProvider`. Dentro de este grupo, el prefijo URI `/api` se aplica automáticamente, por lo que no es necesario aplicarlo manualmente a cada ruta del archivo. Puedes modificar el prefijo y otras opciones del grupo de rutas modificando tu clase `RouteServiceProvider`.

### Métodos de router disponibles

El router permite registrar rutas que respondan a cualquier verbo HTTP:

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

A veces puede necesitar registrar una ruta que responda a múltiples verbos HTTP. Puede hacerlo utilizando el método `match`. O incluso puedes registrar una ruta que responda a todos los verbos HTTP utilizando el método `any`:

```php
Route::match(['get', 'post'], '/', function () {
    // ...
});
 
Route::any('/', function () {
    // ...
});
```

{% hint style="info" %}
Cuando se definen múltiples rutas que comparten la misma URI, las rutas que utilizan los métodos `get`, `post`, `put`, `patch`, `delete` y `options` deben definirse antes que las rutas que utilizan los métodos `any`, `match` y `redirect`. Esto asegura que la petición entrante se corresponde con la ruta correcta.
{% endhint %}

### Inyección de dependencias

Puedes escribir cualquier dependencia requerida por tu ruta en la firma del callback de tu ruta. Las dependencias declaradas serán automáticamente resueltas e inyectadas en la llamada de retorno por el [contenedor de servicios de Laravel](https://laravel.com/docs/10.x/container). Por ejemplo, puedes escribir la clase `Illuminate\Http\Request` para que la petición HTTP actual se inyecte automáticamente en la llamada de retorno de tu ruta:

```php
use Illuminate\Http\Request;
 
Route::get('/users', function (Request $request) {
    // ...
});
```

### Protección CSRF

Recuerde que cualquier formulario HTML que apunte a las rutas `POST`, `PUT`, `PATCH` o `DELETE` definidas en el archivo de rutas `web` debe incluir un campo de token CSRF. En caso contrario, la solicitud será rechazada. Puedes leer más sobre la protección CSRF en la [documentación CSRF](https://laravel.com/docs/10.x/csrf):

```atom
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

## Redireccionar rutas

Si está definiendo una ruta que redirige a otro URI, puede utilizar el método `Route::redirect`. Este método proporciona un atajo conveniente para que usted no tiene que definir una ruta completa o controlador para realizar una simple redirección:

```php
Route::redirect('/here', '/there');
```

Por defecto, `Route::redirect` devuelve un código de estado `302`. Puede personalizar el código de estado utilizando el tercer parámetro opcional:

```php
Route::redirect('/here', '/there', 301);
```

También puede utilizar el método `Route::permanentRedirect` para devolver un código de estado `301`:

```php
Route::permanentRedirect('/here', '/there');
```

{% hint style="info" %}
Cuando se utilizan parámetros de ruta en las rutas de redirección, los siguientes parámetros están reservados por Laravel y no se pueden utilizar: `destination` y `status`.
{% endhint %}

## Rutas de Vista

Si su ruta sólo necesita devolver una [vista](https://laravel.com/docs/10.x/views), puede utilizar el método `Route::view`. Al igual que el método `redirect`, este método proporciona un atajo simple para que no tengas que definir una ruta o controlador completo. El método `view` acepta un URI como primer argumento y un nombre de vista como segundo argumento. Además, puedes proporcionar un array de datos para pasar a la vista como tercer argumento opcional:

```php
Route::view('/welcome', 'welcome');
 
Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

{% hint style="info" %}
Cuando se utilizan parámetros de ruta en rutas de vista, los siguientes parámetros están reservados por Laravel y no se pueden utilizar: `view`, `data`, `status`, y `headers`.
{% endhint %}

## Lista de rutas

El comando `route:list` de Artisan puede proporcionar fácilmente una visión general de todas las rutas definidas por su aplicación:

```sh
php artisan route:list
```

Por defecto, el middleware de ruta que se asigna a cada ruta no se mostrará en la salida `route:list`; sin embargo, puedes ordenar a Laravel que muestre el middleware de ruta añadiendo la opción `-v` al comando:

```sh
php artisan route:list -v
```

También puede ordenar a Laravel que sólo muestre las rutas que comienzan con un URI determinado:

```sh
php artisan route:list --path=api
```

Además, puedes indicar a Laravel que oculte cualquier ruta definida por paquetes de terceros proporcionando la opción `--except-vendor` al ejecutar el comando `route:list`:

```shell
php artisan route:list --except-vendor
```

Del mismo modo, también puedes indicar a Laravel que sólo muestre las rutas definidas por paquetes de terceros proporcionando la opción `--only-vendor` al ejecutar el comando `route:list`:

```sh
php artisan route:list --only-vendor
```

