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

