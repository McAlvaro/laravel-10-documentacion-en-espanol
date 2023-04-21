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

