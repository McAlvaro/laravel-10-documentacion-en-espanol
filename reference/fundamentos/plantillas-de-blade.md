# Plantillas de Blade

## Introducción

Blade es el simple, pero potente motor de plantillas que se incluye con Laravel. A diferencia de otros motores de plantillas PHP, Blade no restringe el uso de código PHP plano en las plantillas. De hecho, todas las plantillas Blade se compilan en código PHP plano y se almacenan en caché hasta que se modifican, lo que significa que Blade añade esencialmente cero sobrecarga a su aplicación. Los archivos de plantilla de Blade utilizan la extensión de archivo `.blade.php` y normalmente se almacenan en el directorio `resources/views`.

Las vistas Blade pueden ser devueltas desde rutas o controladores utilizando el helper global `view`. Por supuesto, como se menciona en la documentación sobre [views](https://laravel.com/docs/10.x/views), se pueden pasar datos a la vista Blade utilizando el segundo argumento del helper `view`:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'Finn']);
});
```

### Supercarga de Blade con Livewire

¿Quieres llevar tus plantillas Blade al siguiente nivel y construir interfaces dinámicas con facilidad? Echa un vistazo a [Laravel Livewire](https://laravel-livewire.com). Livewire le permite escribir componentes Blade que se aumentan con la funcionalidad dinámica que normalmente sólo sería posible a través de frameworks frontend como React o Vue, proporcionando un gran enfoque para la construcción de frontends modernos y reactivos sin las complejidades, renderización del lado del cliente, o pasos de construcción de muchos frameworks JavaScript.

## Visualización de datos

Puede mostrar los datos que se pasan a sus vistas Blade envolviendo la variable entre llaves. Por ejemplo, dada la siguiente ruta:

```php
Route::get('/', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```

Puede mostrar el contenido de la variable `nombre` de la siguiente manera:

```php
Hello, {{ $name }}
```

{% hint style="info" %}
Las sentencias eco `{{ }}` de Blade se envían automáticamente a través de la función `htmlspecialchars` de PHP para evitar ataques XSS.
{% endhint %}

No está limitado a mostrar el contenido de las variables pasadas a la vista. También puede hacer eco de los resultados de cualquier función PHP. De hecho, puede poner cualquier código PHP que desee dentro de una sentencia Blade echo:

```php
The current UNIX timestamp is {{ time() }}.
```

### Codificación de entidades HTML

Por defecto, Blade (y el helper `e` de Laravel) codificarán doblemente las entidades HTML. Si quieres desactivar la doble codificación, llama al método `Blade::withoutDoubleEncoding` desde el método `boot` de tu `AppServiceProvider`:

```php
<?php
 
namespace App\Providers;
 
use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;
 
class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::withoutDoubleEncoding();
    }
}
```

#### Visualización de datos sin escapar

Por defecto, las sentencias Blade `{{ }}` se envían automáticamente a través de la función `htmlspecialchars` de PHP para evitar ataques XSS. Si no desea que sus datos sean escapados, puede utilizar la siguiente sintaxis:

```php
Hello, {!! $name !!}.
```

{% hint style="info" %}
Tenga mucho cuidado cuando se haga eco del contenido suministrado por los usuarios de su aplicación. Por lo general, debe utilizar la sintaxis de doble llave escapada para evitar ataques XSS al mostrar los datos suministrados por el usuario.
{% endhint %}

### Blade y frameworks JavaScript

Dado que muchos frameworks de JavaScript también utilizan llaves "curly" para indicar que una determinada expresión debe mostrarse en el navegador, puede utilizar el símbolo `@` para informar al motor de renderizado de Blade de que una expresión debe permanecer intacta. Por ejemplo

```atom
<h1>Laravel</h1>
 
Hello, @{{ name }}.
```

En este ejemplo, el símbolo `@` será eliminado por Blade; sin embargo, la expresión `{{ nombre }}` permanecerá intacta por el motor de Blade, permitiendo que sea renderizada por su framework JavaScript.

El símbolo `@` también puede utilizarse para escapar de las directivas Blade:

```php
{{-- Blade template --}}
@@if()
 
<!-- HTML output -->
@if()
```

#### Renderización de JSON

A veces puedes pasar un array a tu vista con la intención de renderizarlo como JSON para inicializar una variable JavaScript. Por ejemplo:

```php
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

Sin embargo, en lugar de llamar manualmente a `json_encode`, puede utilizar la directiva del método `Illuminate\Support\Js::from`. El método `from` acepta los mismos argumentos que la función `json_encode` de PHP; sin embargo, se asegurará de que el JSON resultante se escapa correctamente para su inclusión entre comillas HTML. El método `from` devolverá una sentencia JavaScript `JSON.parse` que convertirá el objeto o arreglo dado en un objeto JavaScript válido:

```php
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

Las últimas versiones del esqueleto de aplicaciones Laravel incluyen una fachada `Js`, que proporciona un cómodo acceso a esta funcionalidad dentro de tus plantillas Blade:

```php
<script>
    var app = {{ Js::from($array) }};
</script>
```

{% hint style="info" %}
Sólo debe utilizar el método `Js::from` para renderizar variables existentes como JSON. La plantilla de Blade se basa en expresiones regulares y los intentos de pasar una expresión compleja a la directiva pueden provocar fallos inesperados.
{% endhint %}

#### Directiva \`@verbatim

Si está mostrando variables JavaScript en una gran parte de su plantilla, puede envolver el HTML en la directiva `@verbatim` para no tener que prefijar cada sentencia Blade echo con un símbolo `@`:

```php
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

