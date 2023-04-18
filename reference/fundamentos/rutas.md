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

```php
// Generating URLs...
$url = route('profile');
 
// Generating Redirects...
return redirect()->route('profile');
 
return to_route('profile');
```

Si la ruta definida contiene parámetros, puede pasarlos como segundo argumento a la función `route`. Los parámetros dados se insertarán automáticamente en la URL generada en sus posiciones correctas:

```php
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');
 
$url = route('profile', ['id' => 1]);
```

Si pasa parámetros adicionales en la matriz, esos pares clave/valor se añadirán automáticamente a la cadena de consulta de la URL generada:

```php
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');
 
$url = route('profile', ['id' => 1, 'photos' => 'yes']);
 
// /user/1/profile?photos=yes
```

{% hint style="info" %}
A veces, es posible que desee especificar valores predeterminados para los parámetros de la URL, como la configuración regional actual. Para ello, puede utilizar el método [`URL::defaults`](https://laravel.com/docs/10.x/urls#default-values).
{% endhint %}

#### Inspeccionando la ruta actual

Si desea determinar si la petición actual ha sido enrutada a una ruta con nombre, puede utilizar el método `named` en una instancia de Route. Por ejemplo, puede comprobar el nombre de la ruta actual desde un middleware de ruta:

```php
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;
 
/**
 * Handle an incoming request.
 *
 * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
 */
public function handle(Request $request, Closure $next): Response
{
    if ($request->route()->named('profile')) {
        // ...
    }
 
    return $next($request);
}
```

## Grupos de rutas

Los grupos de rutas permiten compartir atributos de ruta, como el middleware, entre un gran número de rutas sin necesidad de definir esos atributos en cada ruta individual.

Los grupos anidados intentan "fusionar" inteligentemente los atributos con su grupo padre. Las condiciones de middleware y `where` se fusionan mientras que los nombres y prefijos se añaden. Los delimitadores de espacios de nombres y las barras oblicuas de los prefijos URI se añaden automáticamente cuando procede.

### Middleware

Para asignar [middleware](https://laravel.com/docs/10.x/middleware) a todas las rutas de un grupo, puede utilizar el método `middleware` antes de definir el grupo. Los middleware se ejecutan en el orden en que aparecen en el array:

```php
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // Uses first & second middleware...
    });
 
    Route::get('/user/profile', function () {
        // Uses first & second middleware...
    });
});
```

### Controladores

Si un grupo de rutas utilizan todas el mismo [controlador](https://laravel.com/docs/10.x/controllers), puede utilizar el método `controller` para definir el controlador común para todas las rutas del grupo. Entonces, cuando defina las rutas, sólo necesitará proporcionar el método del controlador que invocan:

```php
use App\Http\Controllers\OrderController;
 
Route::controller(OrderController::class)->group(function () {
    Route::get('/orders/{id}', 'show');
    Route::post('/orders', 'store');
});
```

### Enrutamiento de subdominios

Los grupos de rutas también pueden utilizarse para gestionar el enrutamiento de subdominios. A los subdominios se les pueden asignar parámetros de ruta igual que a los URI de ruta, lo que permite capturar una parte del subdominio para utilizarla en la ruta o el controlador. El subdominio puede especificarse llamando al método `domain` antes de definir el grupo:

```php
Route::domain('{account}.example.com')->group(function () {
    Route::get('user/{id}', function (string $account, string $id) {
        // ...
    });
});
```

{% hint style="info" %}
Para asegurarse de que sus rutas de subdominio son accesibles, debe registrar las rutas de subdominio antes de registrar las rutas de dominio raíz. Esto evitará que las rutas de dominio raíz sobrescriban las rutas de subdominio que tengan la misma ruta URI.
{% endhint %}

### Prefijos de rutas

El método `prefix` se puede utilizar para prefijar cada ruta del grupo con un URI determinado. Por ejemplo, es posible que desee prefijar todos los URI de ruta dentro del grupo con `admin`:

```php
Route::prefix('admin')->group(function () {
    Route::get('/users', function () {
        // Matches The "/admin/users" URL
    });
});
```

### Prefijos de nombre de ruta

El método `name` se puede utilizar para prefijar cada nombre de ruta en el grupo con una cadena dada. Por ejemplo, puede prefijar los nombres de todas las rutas del grupo con `admin`. La cadena dada se antepone al nombre de la ruta exactamente como se especifica, por lo que nos aseguraremos de proporcionar el carácter `.` al final del prefijo:

```php
Route::name('admin.')->group(function () {
    Route::get('/users', function () {
        // Route assigned name "admin.users"...
    })->name('users');
});
```

## Vinculación del modelo de ruta (Route Model Binding)

Cuando se inyecta un ID de modelo a una ruta o acción de controlador, a menudo se consulta la base de datos para recuperar el modelo que corresponde a ese ID. Laravel route model binding proporciona una forma conveniente de inyectar automáticamente las instancias del modelo directamente en tus rutas. Por ejemplo, en lugar de inyectar el ID de un usuario, puedes inyectar toda la instancia del modelo `User` que coincida con el ID dado.

### Vinculación implícita

Laravel resuelve automáticamente los modelos Eloquent definidos en rutas o acciones de controlador cuyos nombres de variables de tipo coinciden con un nombre de segmento de ruta. Por ejemplo:

```php
use App\Models\User;
 
Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```

Dado que la variable `$user` es del tipo `App\Models\User` del modelo Eloquent y el nombre de la variable coincide con el segmento URI `{user}`, Laravel inyectará automáticamente la instancia del modelo que tenga un ID que coincida con el valor correspondiente del URI de la petición. Si no se encuentra una instancia de modelo coincidente en la base de datos, se generará automáticamente una respuesta HTTP 404.

Por supuesto, la vinculación implícita también es posible cuando se utilizan métodos de controlador. De nuevo, observe que el segmento URI `{user}` coincide con la variable `$user` del controlador, que contiene una sugerencia de tipo `App\Models\User`:

```php
use App\Http\Controllers\UserController;
use App\Models\User;
 
// Route definition...
Route::get('/users/{user}', [UserController::class, 'show']);
 
// Controller method definition...
public function show(User $user)
{
    return view('user.profile', ['user' => $user]);
}
```

#### Modelos suaves eliminados (Soft Deleted Models)

Normalmente, la vinculación implícita de modelos no recuperará modelos que hayan sido [soft deleted](https://laravel.com/docs/10.x/eloquent#soft-deleting). Sin embargo, puede ordenar al enlace implícito que recupere estos modelos encadenando el método `withTrashed` en la definición de su ruta:

```php
use App\Models\User;
 
Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```

#### Personalización de la clave

A veces puede que desee resolver modelos Eloquent utilizando una columna distinta de `id`. Para ello, puede especificar la columna en la definición del parámetro de ruta:

```php
use App\Models\Post;
 
Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});
```

Si deseas que el enlace del modelo utilice siempre una columna de la base de datos distinta de `id` al recuperar una clase de modelo determinada, puedes anular el método `getRouteKeyName` del modelo de Eloquent:

```php
/**
 * Get the route key for the model.
 */
public function getRouteKeyName(): string
{
    return 'slug';
}
```

#### Claves personalizadas y ámbito

Cuando se vinculan implícitamente varios modelos Eloquent en una única definición de ruta, es posible que desee delimitar el alcance del segundo modelo Eloquent para que sea hijo del modelo Eloquent anterior. Por ejemplo, considere esta definición de ruta que recupera una entrada de blog por slug para un usuario específico:

```php
use App\Models\Post;
use App\Models\User;
 
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
});
```

Cuando se utiliza un enlace implícito con clave personalizada como parámetro de ruta anidado, Laravel delimitará automáticamente la consulta para recuperar el modelo anidado por su padre utilizando convenciones para adivinar el nombre de la relación en el padre. En este caso, se asumirá que el modelo `User` tiene una relación llamada `posts` (la forma plural del nombre del parámetro de ruta) que se puede utilizar para recuperar el modelo `Post`.

Si lo deseas, puedes indicar a Laravel que haga scope de los bindings "hijos" incluso cuando no se proporcione una clave personalizada. Para ello, puede invocar el método `scopeBindings` al definir su ruta:

```php
use App\Models\Post;
use App\Models\User;
 
Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();
```

También puede indicar a todo un grupo de definiciones de ruta que utilicen enlaces de ámbito:

```php
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    });
});
```

Del mismo modo, puedes indicar explícitamente a Laravel que no haga scope de los bindings invocando el método `withoutScopedBindings`:

```php
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
})->withoutScopedBindings();
```

#### Personalización del comportamiento del modelo ausente

Normalmente, se generará una respuesta HTTP 404 si no se encuentra un modelo vinculado implícitamente. Sin embargo, puede personalizar este comportamiento llamando al método `missing` cuando defina su ruta. El método `missing` acepta un cierre que será invocado si no se encuentra un modelo vinculado implícitamente:

```php
use App\Http\Controllers\LocationsController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;
 
Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
        ->name('locations.view')
        ->missing(function (Request $request) {
            return Redirect::route('locations.index');
        });
```

### Vinculación Enum implícita

PHP 8.1 introdujo soporte para [Enums](https://www.php.net/manual/en/language.enumerations.backed.php). Para complementar esta característica, Laravel te permite teclear un [string-backed Enum](https://www.php.net/manual/en/language.enumerations.backed.php) en tu definición de ruta y Laravel sólo invocará la ruta si ese segmento de ruta corresponde a un valor Enum válido. En caso contrario, se devolverá automáticamente una respuesta HTTP 404. Por ejemplo, dado el siguiente Enum:

```php
<?php
 
namespace App\Enums;
 
enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}
```

Puedes definir una ruta que sólo será invocada si el segmento de ruta `{category}` es `fruits` o `people`. De lo contrario, Laravel devolverá una respuesta HTTP 404:

```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;
 
Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

### Vinculación explícita

No es necesario utilizar la resolución de modelos implícita y basada en convenciones de Laravel para utilizar la vinculación de modelos. También puede definir explícitamente cómo los parámetros de ruta se corresponden con los modelos. Para registrar una vinculación explícita, utiliza el método `model` del enrutador para especificar la clase de un parámetro determinado. Debes definir tus enlaces explícitos al principio del método `boot` de tu clase `RouteServiceProvider`:

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;
 
/**
 * Define your route model bindings, pattern filters, etc.
 */
public function boot(): void
{
    Route::model('user', User::class);
 
    // ...
}
```

A continuación, defina una ruta que contenga un parámetro `{user}`:

```php
use App\Models\User;
 
Route::get('/users/{user}', function (User $user) {
    // ...
});
```

Dado que hemos vinculado todos los parámetros `{user}` al modelo `App\Models\User`, una instancia de esa clase será inyectada en la ruta. Así, por ejemplo, una solicitud a `users/1` inyectará la instancia `User` de la base de datos que tiene un ID de `1`.

Si no se encuentra una instancia de modelo coincidente en la base de datos, se generará automáticamente una respuesta HTTP 404.

#### Personalización de la lógica de resolución

Si desea definir su propia lógica de resolución de enlace de modelo, puede utilizar el método `Route::bind`. El cierre que pases al método `bind` recibirá el valor del segmento URI y debería devolver la instancia de la clase que debería ser inyectada en la ruta. De nuevo, esta personalización debería tener lugar en el método `boot` del `RouteServiceProvider` de tu aplicación:

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;
 
/**
 * Define your route model bindings, pattern filters, etc.
 */
public function boot(): void
{
    Route::bind('user', function (string $value) {
        return User::where('name', $value)->firstOrFail();
    });
 
    // ...
}
```

Alternativamente, puedes anular el método `resolveRouteBinding` en tu modelo Eloquent. Este método recibirá el valor del segmento URI y devolverá la instancia de la clase que debe inyectarse en la ruta:

```php
/**
 * Retrieve the model for a bound value.
 *
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($value, $field = null)
{
    return $this->where('name', $value)->firstOrFail();
}
```

Si una ruta utiliza implicit binding scoping, se utilizará el método `resolveChildRouteBinding` para resolver el enlace hijo del modelo padre:

```php
/**
 * Retrieve the child model for a bound value.
 *
 * @param  string  $childType
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveChildRouteBinding($childType, $value, $field)
{
    return parent::resolveChildRouteBinding($childType, $value, $field);
}
```

## Rutas fallback

Usando el método `Route::fallback`, puede definir una ruta que se ejecutará cuando ninguna otra ruta coincida con la petición entrante. Normalmente, las peticiones no gestionadas mostrarán automáticamente una página "404" a través del gestor de excepciones de tu aplicación. Sin embargo, dado que normalmente definirías la ruta `fallback` dentro de tu archivo `routes/web.php`, todo el middleware en el grupo de middleware `web` se aplicará a la ruta. Eres libre de añadir middleware adicional a esta ruta según sea necesario:

```php
Route::fallback(function () {
    // ...
});
```

{% hint style="info" %}
La ruta alternativa debe ser siempre la última ruta registrada por su aplicación.
{% endhint %}

## Limite de rango

### Definición de los limitadores de rango

Laravel incluye potentes y personalizables servicios de limitación de tasa que puedes utilizar para restringir la cantidad de tráfico de una ruta o grupo de rutas determinado. Para empezar, debes definir las configuraciones del limitador de tasa que satisfagan las necesidades de tu aplicación. Típicamente, esto debería hacerse dentro del método `configureRateLimiting` de la clase `App\Providers\RouteServiceProvider` de su aplicación, que ya incluye una definición de limitador de tasa que se aplica a las rutas en el archivo `routes/api.php` de su aplicación:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
 
/**
 * Configure the rate limiters for the application.
 */
protected function configureRateLimiting(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });
}
```

Los limitadores de tráfico se definen utilizando el método `for` de la facade `RateLimiter`. El método `for` acepta un nombre de limitador de tasa y un closure que devuelve la configuración de límite que debería aplicarse a las rutas asignadas al limitador de tráfico. La configuración de límites son instancias de la clase `Illuminate\Cache\RateLimiting\Limit`. Esta clase contiene útiles métodos "constructores" para que pueda definir rápidamente su límite. El nombre del limitador de tráfico puede ser cualquier cadena que desee:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
 
/**
 * Configure the rate limiters for the application.
 */
protected function configureRateLimiting(): void
{
    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });
}
```

Si la petición entrante excede el límite de tráfico especificado, Laravel devolverá automáticamente una respuesta con un código de estado HTTP 429. Si desea definir su propia respuesta que debe ser devuelto por un límite de velocidad, puede utilizar el método `response`:

```php
RateLimiter::for('global', function (Request $request) {
    return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
        return response('Custom response...', 429, $headers);
    });
});
```

Dado que los callbacks del limitador de tráfico reciben la instancia de la petición HTTP entrante, puede construir el límite de tráfico apropiado dinámicamente basándose en la petición entrante o en el usuario autenticado:

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
                ? Limit::none()
                : Limit::perMinute(100);
});
```
