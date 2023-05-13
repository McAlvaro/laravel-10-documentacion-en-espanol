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

### La directiva `@once`

La directiva `@once` permite definir una parte de la plantilla que sólo se evaluará una vez por ciclo de renderizado. Esto puede ser útil para empujar una determinada pieza de JavaScript en el header de la página usando stacks. Por ejemplo, si está renderizando un componente determinado dentro de un bucle, puede que desee enviar el JavaScript a la cabecera sólo la primera vez que se renderice el componente:

```php
@once
    @push('scripts')
        <script>
            // Your custom JavaScript...
        </script>
    @endpush
@endonce
```

Dado que la directiva `@once` se utiliza a menudo junto con las directivas `@push` o `@prepend`, las directivas `@pushOnce` y `@prependOnce` están disponibles para su comodidad:

```php
@pushOnce('scripts')
    <script>
        // Your custom JavaScript...
    </script>
@endPushOnce
```

#### PHP crudo

En algunas situaciones, es útil incrustar código PHP en las vistas. Puedes usar la directiva Blade `@php` para ejecutar un bloque de PHP plano dentro de tu plantilla:

```php
@php
    $counter = 1;
@endphp
```

Si sólo necesita escribir una única sentencia PHP, puede incluir la sentencia dentro de la directiva `@php`:

```php
@php($counter = 1)
```

### Comentarios

Blade también permite definir comentarios en las vistas. Sin embargo, a diferencia de los comentarios HTML, los comentarios de Blade no se incluyen en el HTML devuelto por su aplicación:

```php
{{-- This comment will not be present in the rendered HTML --}}
```

## Componentes

Los componentes y las ranuras proporcionan ventajas similares a las de las secciones, los diseños y los includes; sin embargo, algunos pueden encontrar el modelo mental de los componentes y las ranuras más fácil de entender. Existen dos enfoques para escribir componentes: componentes basados en clases y componentes anónimos.

Para crear un componente basado en una clase, puede utilizar el comando Artisan `make:component`. Para ilustrar el uso de componentes, crearemos un simple componente `Alert`. El comando `make:component` colocará el componente en el directorio `app/View/Components`:

```sh
php artisan make:component Alert
```

El comando `make:component` también creará una plantilla de vista para el componente. La vista se colocará en el directorio `resources/views/components`. Cuando escribes componentes para tu propia aplicación, los componentes se descubren automáticamente en el directorio `app/View/Components` y en el directorio `resources/views/components`, por lo que normalmente no es necesario registrar más componentes.

También puede crear componentes dentro de subdirectorios:

```
php artisan make:component Forms/Input
```

El comando anterior creará un componente `Input` en el directorio `app/View/Components/Forms` y la vista se colocará en el directorio `resources/views/components/forms`.

Si desea crear un componente anónimo (un componente con sólo una plantilla Blade y sin clase), puede utilizar la bandera `--view` al invocar el comando `make:component`:

```sh
php artisan make:component forms.input --view
```

El comando anterior creará un archivo Blade en `resources/views/components/forms/input.blade.php` que puede ser renderizado como un componente a través de `<x-forms.input />`.

#### Registro manual de componentes de paquetes

Al escribir componentes para su propia aplicación, los componentes se descubren automáticamente en el directorio `app/View/Components` y en el directorio `resources/views/components`.

Sin embargo, si está creando un paquete que utiliza componentes Blade, tendrá que registrar manualmente su clase de componente y su alias de etiqueta HTML. Normalmente debe registrar sus componentes en el método `boot` del proveedor de servicios de su paquete:

```php
use Illuminate\Support\Facades\Blade;
 
/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::component('package-alert', Alert::class);
}
```

Una vez registrado el componente, puede renderizarse utilizando su alias de etiqueta:

```php
<x-package-alert/>
```

Alternativamente, puede utilizar el método `componentNamespace` para autocargar clases de componentes por convención. Por ejemplo, un paquete `Nightshade` puede tener componentes `Calendar` y `ColorPicker` que residen dentro del espacio de nombres `Package\Views\Components`:

```php
use Illuminate\Support\Facades\Blade;
 
/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

Esto permitirá el uso de componentes de paquete por su espacio de nombres de proveedor utilizando la sintaxis `nombre-paquete::`:

```php
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade detectará automáticamente la clase vinculada a este componente escribiendo el nombre del componente en pascal. También se admiten subdirectorios utilizando la notación "punto".

### Renderizando Componentes

Para mostrar un componente, puede utilizar una etiqueta de componente Blade dentro de una de sus plantillas Blade. Las etiquetas de componente Blade comienzan con la cadena `x-` seguida del nombre en mayúsculas y minúsculas de la clase de componente:

```php
<x-alert/>
 
<x-user-profile/>
```

Si la clase del componente está anidada a mayor profundidad dentro del directorio `app/View/Components`, puede utilizar el carácter `.` para indicar la anidación de directorios. Por ejemplo, si asumimos que un componente está localizado en `app/View/Components/Inputs/Button.php`, podemos renderizarlo así:

```php
<x-inputs.button/>
```

Si desea renderizar condicionalmente su componente, puede definir un método `shouldRender` en su clase componente. Si el método `shouldRender` devuelve `false` el componente no será renderizado:

```php
use Illuminate\Support\Str;
 
/**
 * Whether the component should be rendered
 */
public function shouldRender(): bool
{
    return Str::length($this->message) > 0;
}
```

### Pasar datos a los componentes

Puede pasar datos a los componentes Blade utilizando atributos HTML. Los valores primitivos codificados pueden pasarse al componente mediante cadenas de atributos HTML simples. Las expresiones y variables PHP deben pasarse al componente mediante atributos que utilicen el carácter `:` como prefijo:

```php
<x-alert type="error" :message="$message"/>
```

Debe definir todos los atributos de datos del componente en su constructor de clase. Todas las propiedades públicas de un componente se pondrán automáticamente a disposición de la vista del componente. No es necesario pasar los datos a la vista desde el método `render` del componente:

```php
<?php
 
namespace App\View\Components;
 
use Illuminate\View\Component;
use Illuminate\View\View;
 
class Alert extends Component
{
    /**
     * Create the component instance.
     */
    public function __construct(
        public string $type,
        public string $message,
    ) {}
 
    /**
     * Get the view / contents that represent the component.
     */
    public function render(): View
    {
        return view('components.alert');
    }
}
```

Cuando su componente es renderizado, puede mostrar el contenido de las variables públicas de su componente haciendo eco de las variables por su nombre:

```atom
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

#### Casing

Los argumentos de los constructores de componentes deben especificarse utilizando `camelCase`, mientras que `kebab-case` debe utilizarse cuando se haga referencia a los nombres de los argumentos en los atributos HTML. Por ejemplo, dado el siguiente constructor de componente:

```php
/**
 * Create the component instance.
 */
public function __construct(
    public string $alertType,
) {}
```

El argumento `$alertType` puede proporcionarse al componente del siguiente modo:

```atom
<x-alert alert-type="danger" />
```

#### Sintaxis corta de atributos

Al pasar atributos a los componentes, también puede utilizar una sintaxis de "atributo corto". Esto suele ser conveniente, ya que los nombres de los atributos suelen coincidir con los nombres de las variables a las que corresponden:

```atom
{{-- Short attribute syntax... --}}
<x-profile :$userId :$name />
 
{{-- Is equivalent to... --}}
<x-profile :user-id="$userId" :name="$name" />
```

#### Escapar de la renderización de atributos

Dado que algunos frameworks JavaScript como Alpine.js también utilizan atributos con prefijo de dos puntos, puede utilizar un prefijo de dos puntos dobles (`::`) para informar a Blade de que el atributo no es una expresión PHP. Por ejemplo, dado el siguiente componente:

```atom
<x-button ::class="{ danger: isDeleting }">
    Submit
</x-button>
```

El siguiente HTML será renderizado por Blade:

```atom
<button :class="{ danger: isDeleting }">
    Submit
</button>
```

#### Métodos de los componentes

Además de que las variables públicas estén disponibles para su plantilla de componentes, cualquier método público del componente puede ser invocado. Por ejemplo, imagine un componente que tiene un método `isSelected`:

```php
/**
 * Determine if the given option is the currently selected option.
 */
public function isSelected(string $option): bool
{
    return $option === $this->selected;
}
```

Puede ejecutar este método desde su plantilla de componentes invocando la variable que coincida con el nombre del método:

```atom
<option {{ $isSelected($value) ? 'selected' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

#### Acceso a atributos y slots dentro de clases de componentes

Los componentes Blade también le permiten acceder al nombre del componente, atributos y slot dentro del método render de la clase. Sin embargo, para acceder a estos datos, debe devolver un closure desde el método `render` de su componente. El closure recibirá un array `$data` como único argumento. Este array contendrá varios elementos que proporcionan información sobre el componente:

```php
use Closure;
 
/**
 * Get the view / contents that represent the component.
 */
public function render(): Closure
{
    return function (array $data) {
        // $data['componentName'];
        // $data['attributes'];
        // $data['slot'];
 
        return '<div>Components content</div>';
    };
}
```

El `componentName` es igual al nombre utilizado en la etiqueta HTML después del prefijo `x-`. Así, el `componentName` de `<x-alert />` será `alert`. El elemento `attributes` contendrá todos los atributos presentes en la etiqueta HTML. El elemento `slot` es una instancia de `Illuminate\Support\HtmlString` con el contenido de la ranura del componente.

El closure debe devolver una cadena. Si la cadena devuelta corresponde a una vista existente, esa vista se renderizará; en caso contrario, la cadena devuelta se evaluará como una vista Blade en línea.

#### Dependencias adicionales

Si tu componente requiere dependencias del [contenedor de servicios de Laravel](https://laravel.com/docs/10.x/container), puedes listarlas antes de cualquiera de los atributos de datos del componente y serán automáticamente inyectadas por el contenedor:

```php
use App\Services\AlertCreator;
 
/**
 * Create the component instance.
 */
public function __construct(
    public AlertCreator $creator,
    public string $type,
    public string $message,
) {}
```

#### Ocultar atributos / métodos

Si desea evitar que algunos métodos públicos o propiedades sean expuestos como variables a su plantilla de componentes, puede añadirlos a una propiedad `$except` array en su componente:

```php
<?php
 
namespace App\View\Components;
 
use Illuminate\View\Component;
 
class Alert extends Component
{
    /**
     * The properties / methods that should not be exposed to the component template.
     *
     * @var array
     */
    protected $except = ['type'];
 
    /**
     * Create the component instance.
     */
    public function __construct(
        public string $type,
    ) {}
}
```

### Atributos de los componentes

Ya hemos examinado cómo pasar atributos de datos a un componente; sin embargo, a veces puede ser necesario especificar atributos HTML adicionales, como `class`, que no forman parte de los datos necesarios para que funcione un componente. Normalmente, estos atributos adicionales se pasan al elemento raíz de la plantilla del componente. Por ejemplo, imaginemos que queremos mostrar un componente `alert` de la siguiente manera:

```atom
<x-alert type="error" :message="$message" class="mt-4"/>
```

Todos los atributos que no formen parte del constructor del componente se añadirán automáticamente a la "bolsa de atributos" del componente. Esta bolsa de atributos se pone automáticamente a disposición del componente a través de la variable `$attributes`. Todos los atributos se pueden mostrar dentro del componente haciendo eco de esta variable:

```atom
<div {{ $attributes }}>
    <!-- Component content -->
</div>
```

{% hint style="info" %}
El uso de directivas como `@env` dentro de las etiquetas de los componentes no está soportado en este momento. Por ejemplo, `<x-alert :live="@env('production')"/>` no se compilará.
{% endhint %}

#### Atributos por defecto / fusionados

A veces puede ser necesario especificar valores por defecto para los atributos o combinar valores adicionales en algunos de los atributos del componente. Para ello, puede utilizar el método `merge` de la bolsa de atributos. Este método es especialmente útil para definir un conjunto de clases CSS predeterminadas que deben aplicarse siempre a un componente:

```atom
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

Si suponemos que este componente se utiliza así:

```atom
<x-alert type="error" :message="$message" class="mb-4"/>
```

El HTML final renderizado del componente tendrá el siguiente aspecto:

```atom
<div class="alert alert-error mb-4">
    <!-- Contents of the $message variable -->
</div>
```

#### Fusionar clases condicionalmente

A veces es posible que desee combinar clases si una condición dada es `true`. Puede hacerlo mediante el método `class`, que acepta un array de clases donde la clave del array contiene la clase o clases que desea añadir, mientras que el valor es una expresión booleana. Si el elemento del array tiene una clave numérica, siempre se incluirá en la lista de clases generada:

```atom
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

Si necesita combinar otros atributos en su componente, puede encadenar el método `merge` en el método `class`:

```atom
<button {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

{% hint style="info" %}
Si necesita compilar condicionalmente clases en otros elementos HTML que no deben recibir atributos fusionados, puede utilizar la directiva `@class`.
{% endhint %}

#### Fusión de atributos sin clase

Al fusionar atributos que no son atributos `class`, los valores proporcionados al método `merge` se considerarán los valores "por defecto" del atributo. Sin embargo, a diferencia del atributo `class`, estos atributos no se fusionarán con valores de atributo inyectados. En su lugar, se sobrescribirán. Por ejemplo, la implementación de un componente `button` puede tener el siguiente aspecto:

```atom
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

Para renderizar el componente button con un `type` personalizado, se puede especificar al consumir el componente. Si no se especifica ningún tipo, se utilizará el tipo `button`:

```atom
<x-button type="submit">
    Submit
</x-button>
```

El HTML renderizado del componente `button` en este ejemplo sería:

```atom
<button type="submit">
    Submit
</button>
```

Si desea que un atributo distinto de `class` tenga su valor por defecto y los valores inyectados unidos, puede utilizar el método `prepends`. En este ejemplo, el atributo `data-controller` comenzará siempre con `profile-controller` y cualquier valor adicional inyectado de `data-controller` se colocará después de este valor por defecto:

```atom
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
```

#### Recuperación y filtrado de atributos

Puedes filtrar atributos utilizando el método `filter`. Este método acepta un closure que debe devolver `true` si deseas mantener el atributo en la bolsa de atributos:

```php
{{ $attributes->filter(fn (string $value, string $key) => $key == 'foo') }}
```

Para mayor comodidad, puede utilizar el método `whereStartsWith` para recuperar todos los atributos cuyas claves empiecen por una cadena determinada:

```php
{{ $attributes->whereStartsWith('wire:model') }}
```

Por el contrario, el método `whereDoesntStartWith` puede utilizarse para excluir todos los atributos cuyas claves empiecen por una cadena determinada:

```php
{{ $attributes->whereDoesntStartWith('wire:model') }}
```

Utilizando el método `first`, puedes renderizar el primer atributo de una bolsa de atributos dada:

```php
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

Si desea comprobar si un atributo está presente en el componente, puede utilizar el método `has`. Este método acepta el nombre del atributo como único argumento y devuelve un booleano que indica si el atributo está presente o no:

```php
@if ($attributes->has('class'))
    <div>Class attribute is present</div>
@endif
```

Puede recuperar el valor de un atributo específico utilizando el método `get`:

```php
{{ $attributes->get('class') }}
```

### Palabras clave reservadas

Por defecto, algunas palabras clave están reservadas para el uso interno de Blade con el fin de renderizar componentes. Las siguientes palabras clave no se pueden definir como propiedades públicas o nombres de métodos dentro de sus componentes:

* `data`
* `render`
* `resolveView`
* `shouldRender`
* `view`
* `withAttributes`
* `withName`

### Slots

A menudo necesitará pasar contenido adicional a su componente a través de "slots". Los "slots" de los componentes se renderizan haciendo eco de la variable `$slot`. Para explorar este concepto, imaginemos que un componente `alert` tiene el siguiente marcado:

```atom
<!-- /resources/views/components/alert.blade.php -->
 
<div class="alert alert-danger">
    {{ $slot }}
</div>
```

Podemos pasar contenido al “slot" inyectando contenido en el componente:

```atom
<x-alert>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

A veces, un componente puede necesitar mostrar varias ranuras diferentes en distintas ubicaciones dentro del componente. Modifiquemos nuestro componente de alerta para permitir la inyección de una ranura de "título":

```atom
<!-- /resources/views/components/alert.blade.php -->
 
<span class="alert-title">{{ $title }}</span>
 
<div class="alert alert-danger">
    {{ $slot }}
</div>
```

Puede definir el contenido de la ranura con la etiqueta `x-slot`. Cualquier contenido que no esté dentro de una etiqueta `x-slot` explícita se pasará al componente en la variable `$slot`:

```atom
<x-alert>
    <x-slot:title>
        Server Error
    </x-slot>
 
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```



