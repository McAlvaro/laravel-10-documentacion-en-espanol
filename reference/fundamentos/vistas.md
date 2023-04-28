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
