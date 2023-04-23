# Controladores

## Introducción

En lugar de definir toda la lógica de gestión de solicitudes como closures en sus archivos de ruta, puede que desee organizar este comportamiento utilizando clases "controlador". Los controladores pueden agrupar la lógica de gestión de peticiones relacionadas en una única clase. Por ejemplo, una clase `UserController` podría manejar todas las peticiones entrantes relacionadas con usuarios, incluyendo mostrar, crear, actualizar y borrar usuarios. Por defecto, los controladores se almacenan en el directorio `app/Http/Controllers`.

## Escribiendo Controladores

### Controladores básicos

Para generar rápidamente un nuevo controlador, puedes ejecutar el comando `make:controller` de Artisan. Por defecto, todos los controladores de su aplicación se almacenan en el directorio `app/Http/Controllers`:

```sh
php artisan make:controller UserController
```

Veamos un ejemplo de controlador básico. Un controlador puede tener cualquier número de métodos públicos que responderán a las peticiones HTTP entrantes:

```php
<?php
 
namespace App\Http\Controllers;
 
use App\Models\User;
use Illuminate\View\View;
 
class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Una vez que haya escrito una clase y un método de controlador, puede definir una ruta al método del controlador de la siguiente manera:

```php
use App\Http\Controllers\UserController;
 
Route::get('/user/{id}', [UserController::class, 'show']);
```

Cuando una petición entrante coincide con la ruta URI especificada, el método `show` de la clase `App\Http\Controllers\UserController` será invocado y los parámetros de la ruta serán pasados al método.

{% hint style="info" %}
No es **necesario** que los controladores extiendan una clase base. Sin embargo, no tendrán acceso a funciones prácticas como los métodos `middleware` y `authorize`.
{% endhint %}

### Controladores de acción simple

Si una acción del controlador es particularmente compleja, puede que te resulte conveniente dedicar toda una clase de controlador a esa única acción. Para ello, puedes definir un único método `__invoke` dentro del controlador:

```php
<?php
 
namespace App\Http\Controllers;
 
use App\Models\User;
use Illuminate\Http\Response;
 
class ProvisionServer extends Controller
{
    /**
     * Provision a new web server.
     */
    public function __invoke()
    {
        // ...
    }
}
```

Al registrar rutas para controladores de acción única, no es necesario especificar un método de controlador. En su lugar, puede simplemente pasar el nombre del controlador al enrutador:

```php
use App\Http\Controllers\ProvisionServer;
 
Route::post('/server', ProvisionServer::class);
```

Puedes generar un controlador invocable utilizando la opción `--invokable` del comando Artisan `make:controller`:

```sh
php artisan make:controller ProvisionServer --invokable
```

{% hint style="info" %}
Los stubs de los controladores pueden personalizarse usando [stub publishing](https://laravel.com/docs/10.x/artisan#stub-customization).
{% endhint %}

## Middleware de controlador

[Middleware](https://laravel.com/docs/10.x/middleware) pueden asignarse a las rutas del controlador en sus archivos de ruta:

```php
Route::get('profile', [UserController::class, 'show'])->middleware('auth');
```

También puede ser conveniente especificar el middleware en el constructor del controlador. Usando el método `middleware` dentro del constructor de tu controlador, puedes asignar middleware a las acciones del controlador:

```php
class UserController extends Controller
{
    /**
     * Instantiate a new controller instance.
     */
    public function __construct()
    {
        $this->middleware('auth');
        $this->middleware('log')->only('index');
        $this->middleware('subscribed')->except('store');
    }
}
```

Los controladores también permiten registrar middleware utilizando un closure. Esto proporciona una manera conveniente de definir un middleware en línea para un solo controlador sin definir una clase entera de middleware:

```php
use Closure;
use Illuminate\Http\Request;
 
$this->middleware(function (Request $request, Closure $next) {
    return $next($request);
});
```

## Controladores de recursos

Si piensas en cada modelo Eloquent de tu aplicación como un "recurso", es típico realizar los mismos conjuntos de acciones contra cada recurso de tu aplicación. Por ejemplo, imagine que su aplicación contiene un modelo `Photo` y un modelo `Movie`. Es probable que los usuarios puedan crear, leer, actualizar o eliminar estos recursos.

Debido a este caso de uso común, el enrutamiento de recursos de Laravel asigna las rutas típicas de creación, lectura, actualización y eliminación ("CRUD") a un controlador con una sola línea de código. Para empezar, podemos utilizar la opción `--resource` del comando Artisan `make:controller` para crear rápidamente un controlador que gestione estas acciones:

```sh
php artisan make:controller PhotoController --resource
```

Este comando generará un controlador en `app/Http/Controllers/PhotoController.php`. El controlador contendrá un método para cada una de las operaciones de recursos disponibles. A continuación, puede registrar una ruta de recursos que apunte al controlador:

```php
use App\Http\Controllers\PhotoController;
 
Route::resource('photos', PhotoController::class);
```

Esta única declaración de ruta crea múltiples rutas para manejar una variedad de acciones en el recurso. El controlador generado ya tendrá métodos para cada una de estas acciones. Recuerda, siempre puedes obtener una visión rápida de las rutas de tu aplicación ejecutando el comando `route:list` de Artisan.

Puedes incluso registrar muchos controladores de recursos a la vez pasando un array al método `resources`:

```php
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

#### Acciones gestionadas por el controlador de recursos

| Verb      | URI                    | Action  | Route Name     |
| --------- | ---------------------- | ------- | -------------- |
| GET       | `/photos`              | index   | photos.index   |
| GET       | `/photos/create`       | create  | photos.create  |
| POST      | `/photos`              | store   | photos.store   |
| GET       | `/photos/{photo}`      | show    | photos.show    |
| GET       | `/photos/{photo}/edit` | edit    | photos.edit    |
| PUT/PATCH | `/photos/{photo}`      | update  | photos.update  |
| DELETE    | `/photos/{photo}`      | destroy | photos.destroy |

#### Personalización del comportamiento del modelo ausente

Normalmente, se generará una respuesta HTTP 404 si no se encuentra un modelo de recurso vinculado implícitamente. Sin embargo, puedes personalizar este comportamiento llamando al método `missing` cuando definas tu ruta de recursos. El método `missing` acepta un closure que será invocado si no se puede encontrar un modelo implícitamente enlazado para ninguna de las rutas del recurso:

```php
use App\Http\Controllers\PhotoController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;
 
Route::resource('photos', PhotoController::class)
        ->missing(function (Request $request) {
            return Redirect::route('photos.index');
        });
```

#### Modelos Soft Deleted

Típicamente, la vinculación implícita de modelos no recuperará modelos que hayan sido [soft deleted](https://laravel.com/docs/10.x/eloquent#soft-deleting), y en su lugar devolverá una respuesta HTTP 404. Sin embargo, puede indicar al framework que permita modelos borrados en caliente invocando el método `withTrashed` al definir su ruta de recursos:

```php
use App\Http\Controllers\PhotoController;
 
Route::resource('photos', PhotoController::class)->withTrashed();
```

Llamar a `withTrashed` sin argumentos permitirá modelos borrados suavemente para las rutas de recursos `show`, `edit`, y `update`. Puedes especificar un subconjunto de estas rutas pasando un array al método `withTrashed`:

```php
Route::resource('photos', PhotoController::class)->withTrashed(['show']);
```

#### Especificar el modelo de recurso

Si utiliza [route model binding](https://laravel.com/docs/10.x/routing#route-model-binding) y desea que los métodos del controlador de recursos indiquen una instancia del modelo, puede utilizar la opción `--model` al generar el controlador:

```sh
php artisan make:controller PhotoController --model=Photo --resource
```

#### Generar solicitudes de formularios

Puedes proporcionar la opción `--requests` cuando generes un controlador de recursos para indicar a Artisan que genere [clases de solicitud de formulario](https://laravel.com/docs/10.x/validation#form-request-validation) para los métodos de almacenamiento y actualización del controlador:

```sh
php artisan make:controller PhotoController --model=Photo --resource --requests
```

### Rutas de recursos parciales

Al declarar una ruta de recursos, puede especificar un subconjunto de acciones que el controlador debe gestionar en lugar del conjunto completo de acciones predeterminadas:

```php
use App\Http\Controllers\PhotoController;
 
Route::resource('photos', PhotoController::class)->only([
    'index', 'show'
]);
 
Route::resource('photos', PhotoController::class)->except([
    'create', 'store', 'update', 'destroy'
]);
```

#### Rutas de recursos para API

Al declarar rutas de recursos que serán consumidas por APIs, normalmente querrás excluir las rutas que presentan plantillas HTML como `create` y `edit`. Para mayor comodidad, puede utilizar el método `apiResource` para excluir automáticamente estas dos rutas:

```php
use App\Http\Controllers\PhotoController;
 
Route::apiResource('photos', PhotoController::class);
```

Puede registrar muchos controladores de recursos de API a la vez pasando una matriz al método `apiResources`:

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\PostController;
 
Route::apiResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

Para generar rápidamente un controlador de recursos API que no incluya los métodos `create` o `edit`, utilice el modificador `--api` al ejecutar el comando `make:controller`:

```sh
php artisan make:controller PhotoController --api
```

### Recursos anidados

A veces puede ser necesario definir rutas a un recurso anidado. Por ejemplo, un recurso foto puede tener múltiples comentarios que pueden adjuntarse a la foto. Para anidar los controladores de recursos, puede utilizar la notación "punto" en su declaración de ruta:

```php
use App\Http\Controllers\PhotoCommentController;
 
Route::resource('photos.comments', PhotoCommentController::class);
```

Esta ruta registrará un recurso anidado al que se podrá acceder con URIs como las siguientes:

```php
/photos/{photo}/comments/{comment}
```

#### Determinación del alcance de los recursos anidados

La característica [implicit model binding](https://laravel.com/docs/10.x/routing#implicit-model-binding-scoping) de Laravel puede determinar automáticamente el alcance de los enlaces anidados de forma que se confirme que el modelo hijo resuelto pertenece al modelo padre. Utilizando el método `scoped` al definir tu recurso anidado, puedes habilitar el alcance automático así como indicar a Laravel por qué campo debe recuperarse el recurso hijo. Para obtener más información sobre cómo lograr esto, consulte la documentación sobre scoping resource routes.

#### Nidificación superficial

A menudo, no es del todo necesario tener tanto el ID padre como el ID hijo dentro de un URI, ya que el ID hijo ya es un identificador único. Cuando utilice identificadores únicos como claves primarias autoincrementadas para identificar sus modelos en segmentos URI, puede optar por utilizar el "anidamiento superficial":

```php
use App\Http\Controllers\CommentController;
 
Route::resource('photos.comments', CommentController::class)->shallow();
```

Esta definición de ruta definirá las siguientes rutas:

| Verb      | URI                               | Action  | Route Name             |
| --------- | --------------------------------- | ------- | ---------------------- |
| GET       | `/photos/{photo}/comments`        | index   | photos.comments.index  |
| GET       | `/photos/{photo}/comments/create` | create  | photos.comments.create |
| POST      | `/photos/{photo}/comments`        | store   | photos.comments.store  |
| GET       | `/comments/{comment}`             | show    | comments.show          |
| GET       | `/comments/{comment}/edit`        | edit    | comments.edit          |
| PUT/PATCH | `/comments/{comment}`             | update  | comments.update        |
| DELETE    | `/comments/{comment}`             | destroy | comments.destroy       |

### Nombrando rutas de recursos

Por defecto, todas las acciones del controlador de recursos tienen un nombre de ruta; sin embargo, puede anular estos nombres pasando una matriz `names` con los nombres de ruta que desee:

```php
use App\Http\Controllers\PhotoController;
 
Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build'
]);
```

### Nombrando parámetros de ruta de recursos

Por defecto, `Route::resource` creará los parámetros de ruta para tus rutas de recursos basándose en la versión "singularizada" del nombre del recurso. Puedes anular esto fácilmente para cada recurso utilizando el método `parameters`. El array pasado al método `parameters` debe ser un array asociativo de nombres de recursos y nombres de parámetros:

```php
use App\Http\Controllers\AdminUserController;
 
Route::resource('users', AdminUserController::class)->parameters([
    'users' => 'admin_user'
]);
```

El ejemplo anterior genera la siguiente URI para la ruta `show` del recurso:

```php
/users/{admin_user}
```

### Alcance de las rutas de recursos

La característica de Laravel [scoped implicit model binding](https://laravel.com/docs/10.x/routing#implicit-model-binding-scoping) puede determinar automáticamente el alcance de los enlaces anidados de forma que se confirme que el modelo hijo resuelto pertenece al modelo padre. Usando el método `scoped` al definir tu recurso anidado, puedes habilitar el alcance automático así como indicar a Laravel por qué campo debe ser recuperado el recurso hijo:

```php
use App\Http\Controllers\PhotoCommentController;
 
Route::resource('photos.comments', PhotoCommentController::class)->scoped([
    'comment' => 'slug',
]);
```

Esta ruta registrará un recurso anidado de alcance al que se podrá acceder con URIs como las siguientes:

```php
/photos/{photo}/comments/{comment:slug}
```

Cuando se utiliza un enlace implícito con clave personalizada como parámetro de ruta anidado, Laravel automáticamente delimitará la consulta para recuperar el modelo anidado por su padre utilizando convenciones para adivinar el nombre de la relación en el padre. En este caso, se asumirá que el modelo `Photo` tiene una relación llamada `comments` (el plural del nombre del parámetro de ruta) que se puede utilizar para recuperar el modelo `Comment`.

### Localización de recursos URI

Por defecto, `Route::resource` creará URIs de recursos usando verbos en inglés y reglas de plural. Si necesitas localizar los verbos de acción `create` y `edit`, puedes usar el método `Route::resourceVerbs`. Esto puede hacerse al principio del método `boot` dentro de la aplicación `App\Providers\RouteServiceProvider`:

```php
/**
 * Define your route model bindings, pattern filters, etc.
 */
public function boot(): void
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
 
    // ...
}
```

El pluralizador de Laravel soporta [varios idiomas diferentes que puedes configurar en función de tus necesidades](https://laravel.com/docs/10.x/localization#pluralization-language). Una vez personalizados los verbos y el lenguaje de pluralización, un registro de ruta de recursos como `Route::resource('publicacion', PublicacionController::class)` producirá las siguientes URIs:

```php
/publicacion/crear
 
/publicacion/{publicaciones}/editar
```

### Complemento de los controladores de recursos

Si necesitas añadir rutas adicionales a un controlador de recursos más allá del conjunto predeterminado de rutas de recursos, debes definir esas rutas antes de tu llamada al método `Route::resource`; de lo contrario, las rutas definidas por el método `resource` pueden tener prioridad involuntariamente sobre tus rutas suplementarias:

```php
use App\Http\Controller\PhotoController;
 
Route::get('/photos/popular', [PhotoController::class, 'popular']);
Route::resource('photos', PhotoController::class);
```

{% hint style="info" %}
Recuerde mantener sus controladores centrados. Si necesita métodos fuera del conjunto típico de acciones de recursos, considere la posibilidad de dividir el controlador en dos controladores más pequeños.
{% endhint %}

### Controladores de recursos Singleton

A veces, su aplicación tendrá recursos que sólo pueden tener una única instancia. Por ejemplo, el "perfil" de un usuario puede editarse o actualizarse, pero un usuario no puede tener más de un "perfil". Del mismo modo, una imagen puede tener una única "miniatura". Estos recursos se denominan "recursos singleton", lo que significa que sólo puede existir una instancia del recurso. En estos casos, puede registrar un controlador de recursos "singleton":

```php
use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;
 
Route::singleton('profile', ProfileController::class);
```

La definición de recurso singleton anterior registrará las siguientes rutas. Como puede ver, las rutas de "creación" no se registran para los recursos singleton, y las rutas registradas no aceptan un identificador ya que sólo puede existir una instancia del recurso:

| Verb      | URI             | Action | Route Name     |
| --------- | --------------- | ------ | -------------- |
| GET       | `/profile`      | show   | profile.show   |
| GET       | `/profile/edit` | edit   | profile.edit   |
| PUT/PATCH | `/profile`      | update | profile.update |

Los recursos Singleton también pueden anidarse dentro de un recurso estándar:

```php
Route::singleton('photos.thumbnail', ThumbnailController::class);
```

En este ejemplo, el recurso `photos` recibiría todas las rutas de recursos estándar; sin embargo, el recurso `thumbnail` sería un recurso singleton con las siguientes rutas:

| Verb      | URI                              | Action | Route Name              |
| --------- | -------------------------------- | ------ | ----------------------- |
| GET       | `/photos/{photo}/thumbnail`      | show   | photos.thumbnail.show   |
| GET       | `/photos/{photo}/thumbnail/edit` | edit   | photos.thumbnail.edit   |
| PUT/PATCH | `/photos/{photo}/thumbnail`      | update | photos.thumbnail.update |
