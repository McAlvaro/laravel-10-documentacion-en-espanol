# Vistas

## Introducción

Por supuesto, no es práctico devolver cadenas enteras de documentos HTML directamente desde tus rutas y controladores. Afortunadamente, las vistas proporcionan una manera conveniente de colocar todo nuestro HTML en archivos separados.

Las vistas separan tu lógica de controlador / aplicación de tu lógica de presentación y se almacenan en el directorio `resources/views`. Cuando se utiliza Laravel, las plantillas de vista se escriben normalmente utilizando el [Blade templating language](https://laravel.com/docs/10.x/blade). Una vista simple podría ser algo como esto:

```atom
<!-- View stored in resources/views/greeting.blade.php -->
 
<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

Como esta vista está almacenada en `resources/views/greeting.blade.php`, podemos devolverla usando el helper global `view` de la siguiente manera:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

{% hint style="info" %}
¿Buscas más información sobre cómo escribir plantillas Blade? Consulta la [documentación de Blade completa](https://laravel.com/docs/10.x/blade) para empezar.
{% endhint %}

### Escribiendo vistas en React / Vue

En lugar de escribir sus plantillas frontend en PHP a través de Blade, muchos desarrolladores han comenzado a preferir escribir sus plantillas utilizando React o Vue. Laravel hace esto sin dolor gracias a [Inertia](https://inertiajs.com/), una biblioteca que hace que sea un juego de niños para atar su frontend React / Vue a su backend Laravel sin las complejidades típicas de la construcción de una SPA.

Nuestros [starter kits de Breeze y Jetstream](https://laravel.com/docs/10.x/starter-kits) le ofrecen un buen punto de partida para su próxima aplicación Laravel con Inertia. Además, el [Laravel Bootcamp](https://bootcamp.laravel.com) proporciona una demostración completa de la construcción de una aplicación Laravel impulsada por Inertia, incluyendo ejemplos en Vue y React.

## Creación y renderización de vistas

Puede crear una vista colocando un archivo con la extensión `.blade.php` en el directorio `resources/views` de su aplicación. La extensión `.blade.php` informa al framework de que el archivo contiene una [plantilla Blade](https://laravel.com/docs/10.x/blade). Las plantillas Blade contienen HTML así como directivas Blade que le permiten fácilmente hacer eco de valores, crear sentencias "if", iterar sobre datos, y más.

Una vez que hayas creado una vista, puedes devolverla desde una de las rutas o controladores de tu aplicación utilizando el helper global `view`:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

Las vistas también pueden devolverse utilizando la fachada `View`:

```php
use Illuminate\Support\Facades\View;
 
return View::make('greeting', ['name' => 'James']);
```

Como puedes ver, el primer argumento pasado al helper `view` corresponde al nombre del archivo de la vista en el directorio `resources/views`. El segundo argumento es un array de datos que deben estar disponibles para la vista. En este caso, estamos pasando la variable `name`, que se muestra en la vista usando [sintaxis Blade](https://laravel.com/docs/10.x/blade).

### Directorios de vista anidados

Las vistas también pueden anidarse en subdirectorios del directorio `resources/views`. Se puede utilizar la notación "punto" para referenciar vistas anidadas. Por ejemplo, si tu vista está almacenada en `resources/views/admin/profile.blade.php`, puedes devolverla desde una de las rutas / controladores de tu aplicación así:

```php
return view('admin.profile', $data);
```

{% hint style="info" %}
Los nombres de los directorios de vista no deben contener el carácter `.`.
{% endhint %}

### Creación de la primera vista disponible

Usando el método `first` de la fachada `View`, puedes crear la primera vista que existe en un array dado de vistas. Esto puede ser útil si su aplicación o paquete permite que las vistas sean personalizadas o sobrescritas:

```php
use Illuminate\Support\Facades\View;
 
return View::first(['custom.admin', 'admin'], $data);
```

### Determinar si existe una vista

Si necesitas determinar si una vista existe, puedes utilizar la fachada `View`. El método `exists` devolverá `true` si la vista existe:

```php
use Illuminate\Support\Facades\View;
 
if (View::exists('emails.customer')) {
    // ...
}
```

## Pasar datos a vistas

Como has visto en los ejemplos anteriores, puedes pasar un array de datos a las vistas para que esos datos estén disponibles para la vista:

```php
return view('greetings', ['name' => 'Victoria']);
```

Cuando se pasa información de esta manera, los datos deben ser un array con pares clave / valor. Después de proporcionar los datos a una vista, puedes acceder a cada valor dentro de tu vista usando las claves de los datos, como `<?php echo $nombre; ?>`.

Como alternativa a pasar un array completo de datos a la función helper `view`, puedes usar el método `with` para añadir piezas individuales de datos a la vista. El método `with` devuelve una instancia del objeto vista para que pueda continuar encadenando métodos antes de devolver la vista:

```php
return view('greeting')
            ->with('name', 'Victoria')
            ->with('occupation', 'Astronaut');
```

### Compartir datos con todas las vistas

Ocasionalmente, puedes necesitar compartir datos con todas las vistas que son renderizadas por tu aplicación. Puedes hacerlo utilizando el método `share` de la fachada `View`. Típicamente, deberías colocar llamadas al método `share` dentro del método `boot` de un proveedor de servicios. Eres libre de añadirlos a la clase `App\Providers\AppServiceProvider` o generar un proveedor de servicios independiente para alojarlos:

```php
<?php
 
namespace App\Providers;
 
use Illuminate\Support\Facades\View;
 
class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }
 
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        View::share('key', 'value');
    }
}
```

## View Composers

Los view composers son callbacks o métodos de clase que se llaman cuando se renderiza una vista. Si tienes datos que quieres que se vinculen a una vista cada vez que ésta se renderiza, un compositor de vistas puede ayudarte a organizar esa lógica en una única ubicación. Los view composers pueden ser particularmente útiles si la misma vista es devuelta por múltiples rutas o controladores dentro de tu aplicación y siempre necesita un dato en particular.

Típicamente, los compositores de vistas serán registrados dentro de uno de los [proveedores de servicios de tu aplicación](https://laravel.com/docs/10.x/providers). En este ejemplo, asumiremos que hemos creado un nuevo `App\Providers\ViewServiceProvider` para alojar esta lógica.

Usaremos el método `composer` de la fachada `View` para registrar el compositor de vistas. Laravel no incluye un directorio por defecto para los compositores de vistas basados en clases, así que eres libre de organizarlos como quieras. Por ejemplo, puedes crear un directorio `app/View/Composers` para alojar todos los compositores de vistas de tu aplicación:

```php
<?php
 
namespace App\Providers;
 
use App\View\Composers\ProfileComposer;
use Illuminate\Support\Facades;
use Illuminate\Support\ServiceProvider;
use Illuminate\View\View;
 
class ViewServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }
 
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        // Using class based composers...
        Facades\View::composer('profile', ProfileComposer::class);
 
        // Using closure based composers...
        Facades\View::composer('welcome', function (View $view) {
            // ...
        });
 
        Facades\View::composer('dashboard', function (View $view) {
            // ...
        });
    }
}
```

{% hint style="info" %}
Recuerde que si crea un nuevo proveedor de servicios para contener sus registros de compositores de vistas, deberá añadir el proveedor de servicios a la matriz `providers` del archivo de configuración `config/app.php`.
{% endhint %}

Ahora que hemos registrado el compositor, el método `compose` de la clase `App\View\Composers\ProfileComposer` será ejecutado cada vez que la vista `profile` sea renderizada. Veamos un ejemplo de la clase composer:

```php
<?php
 
namespace App\View\Composers;
 
use App\Repositories\UserRepository;
use Illuminate\View\View;
 
class ProfileComposer
{
    /**
     * Create a new profile composer.
     */
    public function __construct(
        protected UserRepository $users,
    ) {}
 
    /**
     * Bind data to the view.
     */
    public function compose(View $view): void
    {
        $view->with('count', $this->users->count());
    }
}
```

Como puedes ver, todos los compositores de vistas se resuelven a través del [contenedor de servicios](https://laravel.com/docs/10.x/container), por lo que puedes escribir cualquier dependencia que necesites dentro del constructor de un compositor.

#### Adjuntar un compositor a varias vistas

Puedes adjuntar un compositor de vistas a varias vistas a la vez pasando un array de vistas como primer argumento al método `composer`:

```php
use App\Views\Composers\MultiComposer;
use Illuminate\Support\Facades\View;
 
View::composer(
    ['profile', 'dashboard'],
    MultiComposer::class
);
```

El método `composer` también acepta el carácter `*` como comodín, lo que permite adjuntar un compositor a todas las vistas:

```php
use Illuminate\Support\Facades;
use Illuminate\View\View;
 
Facades\View::composer('*', function (View $view) {
    // ...
});
```

### Creadores de vista

Los "creadores" de vistas son muy similares a los compositores de vistas; sin embargo, se ejecutan inmediatamente después de instanciar la vista en lugar de esperar a que la vista esté a punto de renderizarse. Para registrar un creador de vistas, utilice el método `creator`:

```php
use App\View\Creators\ProfileCreator;
use Illuminate\Support\Facades\View;
 
View::creator('profile', ProfileCreator::class);
```

## Optimización de vistas

Por defecto, las vistas de la plantilla Blade se compilan bajo demanda. Cuando se ejecuta una petición que renderiza una vista, Laravel determinará si existe una versión compilada de la vista. Si el fichero existe, Laravel determinará si la vista no compilada ha sido modificada más recientemente que la vista compilada. Si la vista compilada no existe, o la vista no compilada ha sido modificada, Laravel recompilará la vista.

Compilar vistas durante la petición puede tener un pequeño impacto negativo en el rendimiento, por lo que Laravel proporciona el comando Artisan `view:cache` para precompilar todas las vistas utilizadas por tu aplicación. Para aumentar el rendimiento, es posible que desees ejecutar este comando como parte de tu proceso de despliegue:

```sh
php artisan view:cache
```

Puede utilizar el comando `view:clear` para borrar la caché de vista:

```sh
php artisan view:clear
```
