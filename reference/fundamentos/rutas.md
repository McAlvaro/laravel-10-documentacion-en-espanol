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

## Parámetros de ruta

### Parámetros Requeridos

A veces necesitará capturar segmentos del URI dentro de su ruta. Por ejemplo, puede que necesite capturar el ID de un usuario de la URL. Puede hacerlo definiendo parámetros de ruta:

```php
Route::get('/user/{id}', function (string $id) {
    return 'User '.$id;
});
```

Puede definir tantos parámetros de ruta como requiera su ruta:

```php
Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
    // ...
});
```

Los parámetros de ruta van siempre entre llaves `{}` y deben estar formados por caracteres alfabéticos. Los guiones bajos (`_`) también son aceptables en los nombres de parámetros de ruta. Los parámetros de ruta se inyectan en las retrollamadas/los controladores de ruta en función de su orden: los nombres de los argumentos de las retrollamadas/los controladores de ruta no importan.

#### Parámetros e inyección de dependencias

Si tu ruta tiene dependencias que quieres que el contenedor de servicios de Laravel inyecte automáticamente en el callback de tu ruta, debes listar los parámetros de tu ruta después de tus dependencias:

```php
use Illuminate\Http\Request;
 
Route::get('/user/{id}', function (Request $request, string $id) {
    return 'User '.$id;
});
```

### Parámetros opcionales

En ocasiones puede ser necesario especificar un parámetro de ruta que no siempre está presente en el URI. Puede hacerlo colocando una marca `?` después del nombre del parámetro. Asegúrese de dar a la variable correspondiente de la ruta un valor por defecto:

```php
Route::get('/user/{name?}', function (string $name = null) {
    return $name;
});
 
Route::get('/user/{name?}', function (string $name = 'John') {
    return $name;
});
```

### Restricciones con expresiones regulares

Puede restringir el formato de los parámetros de ruta utilizando el método `where` en una instancia de ruta. El método `where` acepta el nombre del parámetro y una expresión regular que define cómo se debe restringir el parámetro:

```php
Route::get('/user/{name}', function (string $name) {
    // ...
})->where('name', '[A-Za-z]+');
 
Route::get('/user/{id}', function (string $id) {
    // ...
})->where('id', '[0-9]+');
 
Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

Para mayor comodidad, algunos patrones de expresiones regulares de uso común tienen métodos de ayuda que le permiten añadir rápidamente restricciones de patrones a sus rutas:

```php
Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->whereNumber('id')->whereAlpha('name');
 
Route::get('/user/{name}', function (string $name) {
    // ...
})->whereAlphaNumeric('name');
 
Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUuid('id');
 
Route::get('/user/{id}', function (string $id) {
    //
})->whereUlid('id');
 
Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', ['movie', 'song', 'painting']);
```

Si la solicitud entrante no coincide con las restricciones del patrón de ruta, se devolverá una respuesta HTTP 404.

#### Restricciones globales

Si desea que un parámetro de ruta esté siempre limitado por una expresión regular dada, puede utilizar el método `pattern`. Debe definir estos patrones en el método `boot` de su clase `App\Providers\RouteServiceProvider`:

```php
/**
 * Define your route model bindings, pattern filters, etc.
 */
public function boot(): void
{
    Route::pattern('id', '[0-9]+');
}
```

Una vez definido el patrón, se aplica automáticamente a todas las rutas que utilicen ese nombre de parámetro:

```php
Route::get('/user/{id}', function (string $id) {
    // Only executed if {id} is numeric...
});
```

#### Slashes codificados

El componente de enrutamiento de Laravel permite que todos los caracteres excepto `/` estén presentes en los valores de los parámetros de ruta. Debe permitir explícitamente que `/` forme parte de su marcador de posición utilizando una expresión regular de condición `where`:

```php
Route::get('/search/{search}', function (string $search) {
    return $search;
})->where('search', '.*');
```

{% hint style="info" %}
Las barras diagonales codificadas sólo se admiten dentro del último segmento de ruta.
{% endhint %}

## Rutas con nombre

Las rutas con nombre permiten generar cómodamente URL o redirecciones para rutas específicas. Puede especificar un nombre para una ruta encadenando el método `name` en la definición de la ruta:

```php
Route::get('/user/profile', function () {
    // ...
})->name('profile');
```

También puede especificar nombres de ruta para las acciones del controlador:

```php
Route::get(
    '/user/profile',
    [UserProfileController::class, 'show']
)->name('profile');
```

{% hint style="info" %}
Los nombres de las rutas deben ser siempre únicos.
{% endhint %}

#### Generación de URL para rutas con nombre

Una vez que hayas asignado un nombre a una ruta dada, puedes usar el nombre de la ruta cuando generes URLs o redirecciones a través de las funciones de ayuda `route` y `redirect` de Laravel:
