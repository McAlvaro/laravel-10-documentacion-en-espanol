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

#### Directiva `@verbatim`

Si está mostrando variables JavaScript en una gran parte de su plantilla, puede envolver el HTML en la directiva `@verbatim` para no tener que prefijar cada sentencia Blade echo con un símbolo `@`:

```php
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

## Directivas Blade

Además de la herencia de plantillas y la visualización de datos, Blade también proporciona cómodos atajos para estructuras de control PHP comunes, como sentencias condicionales y bucles. Estos atajos proporcionan una forma muy limpia y concisa de trabajar con las estructuras de control de PHP sin dejar de ser familiar a sus homólogos de PHP.

### Sentencias If

Puede construir sentencias `if` utilizando las directivas `@if`, `@elseif`, `@else` y `@endif`. Estas directivas funcionan de forma idéntica a sus equivalentes en PHP:

```php
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

Para mayor comodidad, Blade también proporciona una directiva `@unless`:

```php
@unless (Auth::check())
    You are not signed in.
@endunless
```

Además de las directivas condicionales ya discutidas, las directivas `@isset` y `@empty` pueden ser usadas como convenientes atajos para sus respectivas funciones PHP:

```php
@isset($records)
    // $records is defined and is not null...
@endisset
 
@empty($records)
    // $records is "empty"...
@endempty
```

#### Directivas de autenticación

Las directivas `@auth` y `@guest` pueden utilizarse para determinar rápidamente si el usuario actual está [autenticado](https://laravel.com/docs/10.x/authentication) o es un invitado:

```php
@auth
    // The user is authenticated...
@endauth
 
@guest
    // The user is not authenticated...
@endguest
```

Si es necesario, puede especificar la guardia de autenticación que debe comprobarse al utilizar las directivas `@auth` y `@guest`:

```php
@auth('admin')
    // The user is authenticated...
@endauth
 
@guest('admin')
    // The user is not authenticated...
@endguest
```

#### Directivas Environment

Puede comprobar si la aplicación se está ejecutando en el entorno de producción utilizando la directiva `@production`:

```php
@production
    // Production specific content...
@endproduction
```

O bien, puede determinar si la aplicación se está ejecutando en un entorno específico utilizando la directiva `@env`:

```php
@env('staging')
    // The application is running in "staging"...
@endenv
 
@env(['staging', 'production'])
    // The application is running in "staging" or "production"...
@endenv
```

#### Directiva section

Puede determinar si una sección de la herencia de la plantilla tiene contenido utilizando la directiva `@hasSection`:

```php
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>
 
    <div class="clearfix"></div>
@endif
```

Puede utilizar la directiva `sectionMissing` para determinar si una sección no tiene contenido:

```php
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif
```

### Sentencia Switch

Las sentencias switch pueden construirse utilizando las directivas `@switch`, `@case`, `@break`, `@default` y `@endswitch`:

```php
@switch($i)
    @case(1)
        First case...
        @break
 
    @case(2)
        Second case...
        @break
 
    @default
        Default case...
@endswitch
```

### Bucles

Además de las sentencias condicionales, Blade proporciona directivas simples para trabajar con las estructuras de bucle de PHP. De nuevo, cada una de estas directivas funciona de forma idéntica a sus equivalentes en PHP:

```php
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor
 
@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach
 
@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse
 
@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

{% hint style="info" %}
Mientras iteras a través de un bucle `foreach`, puedes utilizar la variable de bucle para obtener información valiosa sobre el bucle, como por ejemplo si estás en la primera o en la última iteración a través del bucle.
{% endhint %}

Cuando se utilizan bucles también se puede saltar la iteración actual o finalizar el bucle utilizando las directivas `@continue` y `@break`:

```php
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif
 
    <li>{{ $user->name }}</li>
 
    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

También puede incluir la condición de continuación o ruptura dentro de la declaración de directiva:

```php
@foreach ($users as $user)
    @continue($user->type == 1)
 
    <li>{{ $user->name }}</li>
 
    @break($user->number == 5)
@endforeach
```

### La variable Loop

Mientras se itera a través de un bucle `foreach`, una variable `$loop` estará disponible dentro de tu bucle. Esta variable proporciona acceso a información útil como el índice actual del bucle y si es la primera o la última iteración del bucle:

```php
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif
 
    @if ($loop->last)
        This is the last iteration.
    @endif
 
    <p>This is user {{ $user->id }}</p>
@endforeach
```

Si estás en un bucle anidado, puedes acceder a la variable `$loop` del bucle padre a través de la propiedad `parent`:

```php
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is the first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

La variable `$loop` también contiene otras propiedades útiles:

| Property           | Description                                                     |
| ------------------ | --------------------------------------------------------------- |
| `$loop->index`     | El índice de la iteración actual del bucle (comienza en 0).     |
| `$loop->iteration` | La iteración actual del bucle (comienza en 1).                  |
| `$loop->remaining` | Las iteraciones restantes en el bucle.                          |
| `$loop->count`     | El número total de elementos de la matriz que se está iterando. |
| `$loop->first`     | Si esta es la primera iteración a través del bucle.             |
| `$loop->last`      | Si esta es la última iteración a través del bucle.              |
| `$loop->even`      | Si se trata de una iteración par a través del bucle.            |
| `$loop->odd`       | Si se trata de una iteración impar a través del bucle.          |
| `$loop->depth`     | El nivel de anidamiento del bucle actual.                       |
| `$loop->parent`    | En un bucle anidado, la variable del bucle padre.               |

### Estilos y clases condicionales

La directiva `@class` compila condicionalmente una cadena de clases CSS. La directiva acepta un array de clases donde la clave del array contiene la clase o clases que desea añadir, mientras que el valor es una expresión booleana. Si el elemento del array tiene una clave numérica, siempre se incluirá en la lista de clases renderizada:

```php
@php
    $isActive = false;
    $hasError = true;
@endphp
 
<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>
 
<span class="p-4 text-gray-500 bg-red"></span>
```

Del mismo modo, la directiva `@style` puede utilizarse para añadir condicionalmente estilos CSS en línea a un elemento HTML:

```php
@php
    $isActive = true;
@endphp
 
<span @style([
    'background-color: red',
    'font-weight: bold' => $isActive,
])></span>
 
<span style="background-color: red; font-weight: bold;"></span>
```

### Atributos adicionales

Por comodidad, puede utilizar la directiva `@checked` para indicar fácilmente si una casilla de verificación HTML está "marcada". Esta directiva se hará eco de `checked` si la condición proporcionada se evalúa como `true`:

```atom
<input type="checkbox"
        name="active"
        value="active"
        @checked(old('active', $user->active)) />
```

Del mismo modo, la directiva `@selected` puede utilizarse para indicar si una determinada opción de selección debe ser "seleccionada":

```php
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

Además, la directiva `@disabled` puede utilizarse para indicar si un elemento determinado debe estar "desactivado":

```html
<button type="submit" @disabled($errors->isNotEmpty())>Submit</button>
```

Además, la directiva `@readonly` puede utilizarse para indicar si un elemento dado debe ser "readonly":

```html
<input type="email"
        name="email"
        value="email@laravel.com"
        @readonly($user->isNotAdmin()) />
```

Además, la directiva `@required` puede utilizarse para indicar si un elemento determinado debe ser "obligatorio":

```html
<input type="text"
        name="title"
        value="title"
        @required($user->isAdmin()) />
```

### Incluir subvistas

{% hint style="info" %}
Aunque es libre de utilizar la directiva `@include`, los componentes Blade proporcionan una funcionalidad similar y ofrecen varias ventajas sobre la directiva `@include`, como la vinculación de datos y atributos.
{% endhint %}

La directiva `@include` de Blade permite incluir una vista Blade dentro de otra vista. Todas las variables disponibles para la vista padre estarán disponibles para la vista incluida:

```php
<div>
    @include('shared.errors')
 
    <form>
        <!-- Form Contents -->
    </form>
</div>
```

Aunque la vista incluida heredará todos los datos disponibles en la vista padre, también puede pasar una matriz de datos adicionales que deben estar disponibles para la vista incluida:

```php
@include('view.name', ['status' => 'complete'])
```

Si intentas `@include` una vista que no existe, Laravel arrojará un error. Si deseas incluir una vista que puede o no estar presente, debes utilizar la directiva `@includeIf`:

```php
@includeIf('view.name', ['status' => 'complete'])
```

Si desea `@incluir` una vista si una expresión booleana dada se evalúa como `true` o `false`, puede utilizar las directivas `@includeWhen` e `@includeUnless`:

```php
@includeWhen($boolean, 'view.name', ['status' => 'complete'])
 
@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

Para incluir la primera vista que exista de un conjunto dado de vistas, puede utilizar la directiva `includeFirst`:

```php
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

{% hint style="info" %}
Debe evitar utilizar las constantes `__DIR__` y `__FILE__` en sus vistas Blade, ya que se referirán a la ubicación de la vista compilada en caché.
{% endhint %}

#### Representación de vistas para colecciones

Puede combinar bucles e includes en una línea con la directiva `@each` de Blade:

```php
@each('view.name', $jobs, 'job')
```

El primer argumento de la directiva `@each` es la vista que se mostrará para cada elemento del array o colección. El segundo argumento es la matriz o colección sobre la que se desea iterar, mientras que el tercer argumento es el nombre de la variable que se asignará a la iteración actual dentro de la vista. Así, por ejemplo, si estás iterando sobre un array de `jobs`, normalmente querrás acceder a cada job como una variable `job` dentro de la vista. La clave del array para la iteración actual estará disponible como la variable `key` dentro de la vista.

También puede pasar un cuarto argumento a la directiva `@each`. Este argumento determina la vista que se mostrará si la matriz dada está vacía.

```php
@each('view.name', $jobs, 'job', 'view.empty')
```

{% hint style="info" %}
Las vistas renderizadas mediante `@each` no heredan las variables de la vista padre. Si la vista hija necesita estas variables, debe utilizar las directivas `@foreach` e `@include` en su lugar.
{% endhint %}

