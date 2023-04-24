# Solicitudes HTTP

## Introducción

La clase `Illuminate\Http\Request` de Laravel proporciona una forma orientada a objetos para interactuar con la solicitud HTTP actual que está siendo manejada por su aplicación, así como recuperar la entrada, cookies y archivos que fueron enviados con la solicitud.

## Interacción con la solicitud

### Accediendo a la solicitud

Para obtener una instancia de la solicitud HTTP actual a través de la inyección de dependencia, debe escribir la clase `Illuminate\Http\Request` en su closure de ruta o método de controlador. La instancia de solicitud entrante será inyectada automáticamente por el [contenedor de servicios de Laravel](https://laravel.com/docs/10.x/container):

```php
<?php
 
namespace App\Http\Controllers;
 
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
 
class UserController extends Controller
{
    /**
     * Store a new user.
     */
    public function store(Request $request): RedirectResponse
    {
        $name = $request->input('name');
 
        // Store the user...
 
        return redirect('/users');
    }
}
```

Como se ha mencionado, también puede escribir la clase `Illuminate\Http\Request` en un closure de ruta. El contenedor de servicios inyectará automáticamente la solicitud entrante en el closure cuando se ejecute:

```php
use Illuminate\Http\Request;
 
Route::get('/', function (Request $request) {
    // ...
});
```

#### Inyección de dependencia y parámetros de ruta

Si el método de su controlador también espera la entrada de un parámetro de ruta, debe listar los parámetros de ruta después de las otras dependencias. Por ejemplo, si tu ruta está definida así:

```php
use App\Http\Controllers\UserController;
 
Route::put('/user/{id}', [UserController::class, 'update']);
```

Aún puede escribir la sugerencia `Illuminate\Http\Request` y acceder a su parámetro de ruta `id` definiendo su método controlador de la siguiente manera:

```php
<?php
 
namespace App\Http\Controllers;
 
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
 
class UserController extends Controller
{
    /**
     * Update the specified user.
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // Update the user...
 
        return redirect('/users');
    }
}
```

### Ruta de solicitud, host y método

La instancia `Illuminate\Http\Request` proporciona una variedad de métodos para examinar la solicitud HTTP entrante y extiende la clase `Symfony\Component\HttpFoundation\Request`. Vamos a discutir algunos de los métodos más importantes a continuación.

#### Recuperación de la ruta de solicitud

El método `path` devuelve la información de la ruta de la petición. Así, si la petición entrante se dirige a `http://example.com/foo/bar`, el método `path` devolverá `foo/bar`:

```php
$uri = $request->path();
```

#### Inspección de la ruta de solicitud

El método `is` le permite verificar que la ruta de la petición entrante coincide con un patrón dado. Puede utilizar el carácter `*` como comodín al utilizar este método:

```php
if ($request->is('admin/*')) {
    // ...
}
```

Utilizando el método `routeIs`, puede determinar si la solicitud entrante ha coincidido con una [ruta con nombre](https://laravel.com/docs/10.x/routing#named-routes):

```php
if ($request->routeIs('admin.*')) {
    // ...
}
```

#### Recuperación de la URL de solicitud

Para recuperar la URL completa de la petición entrante puede utilizar los métodos `url` o `fullUrl`. El método `url` devolverá la URL sin la cadena de consulta, mientras que el método `fullUrl` incluye la cadena de consulta:

```php
$url = $request->url();
 
$urlWithQueryString = $request->fullUrl();
```

Si desea añadir datos de la cadena de consulta a la URL actual, puede llamar al método `fullUrlWithQuery`. Este método combina la matriz de variables de cadena de consulta con la cadena de consulta actual:

```php
$request->fullUrlWithQuery(['type' => 'phone']);
```

#### Recuperación del host de solicitud

Puede recuperar el "host" de la solicitud entrante mediante los métodos `host`, `httpHost` y `schemeAndHttpHost`:

```php
$request->host();
$request->httpHost();
$request->schemeAndHttpHost();
```

#### Recuperación del método de solicitud

El método `method` devolverá el verbo HTTP de la petición. Puedes utilizar el método `isMethod` para verificar que el verbo HTTP coincide con una cadena dada:

```php
$method = $request->method();
 
if ($request->isMethod('post')) {
    // ...
}
```

### Headers de solicitud

Puede recuperar un header de la petición desde la instancia `Illuminate\Http\Request` usando el método `header`. Si el header no está presente en la petición, se devolverá `null`. Sin embargo, el método `header` acepta un segundo argumento opcional que será devuelto si el header no está presente en la petición:

```php
$value = $request->header('X-Header-Name');
 
$value = $request->header('X-Header-Name', 'default');
```

El método `hasHeader` puede utilizarse para determinar si la petición contiene un header determinado:

```php
if ($request->hasHeader('X-Header-Name')) {
    // ...
}
```

Por comodidad, el método `bearerToken` puede utilizarse para recuperar un token de portador del header `Authorization`. Si no existe tal header, se devolverá una cadena vacía:

```php
$token = $request->bearerToken();
```

### Dirección IP de la solicitud

El método `ip` se puede utilizar para recuperar la dirección IP del cliente que realizó la solicitud a su aplicación:

```php
$ipAddress = $request->ip();
```

### Negociación de contenidos

Laravel proporciona varios métodos para inspeccionar los tipos de contenido solicitados a través del header `Accept`. En primer lugar, el método `getAcceptableContentTypes` devolverá un array con todos los tipos de contenido aceptados por la petición:

```php
$contentTypes = $request->getAcceptableContentTypes();
```

El método `accepts` acepta un array de tipos de contenido y devuelve `true` si alguno de los tipos de contenido es aceptado por la petición. En caso contrario, devolverá `false`:

```php
if ($request->accepts(['text/html', 'application/json'])) {
    // ...
}
```

Puede utilizar el método `prefers` para determinar qué tipo de contenido de un conjunto dado de tipos de contenido es el más preferido por la petición. Si la solicitud no acepta ninguno de los tipos de contenido proporcionados, se devolverá `null`:

```php
$preferred = $request->prefers(['text/html', 'application/json']);
```

Dado que muchas aplicaciones sólo sirven HTML o JSON, puede utilizar el método `expectsJson` para determinar rápidamente si la solicitud entrante espera una respuesta JSON:

```php
if ($request->expectsJson()) {
    // ...
}
```

### Solicitudes PSR-7

El [estándar PSR-7](https://www.php-fig.org/psr/psr-7/) especifica interfaces para mensajes HTTP, incluyendo peticiones y respuestas. Si quieres obtener una instancia de una petición PSR-7 en lugar de una petición Laravel, primero tendrás que instalar algunas librerías. Laravel utiliza el componente _Symfony HTTP Message Bridge_ para convertir las peticiones y respuestas típicas de Laravel en implementaciones compatibles con PSR-7:

```sh
composer require symfony/psr-http-message-bridge
composer require nyholm/psr7
```

Una vez que haya instalado estas bibliotecas, puede obtener una solicitud PSR-7 mediante la sugerencia de tipo de la interfaz de solicitud en su método de cierre de ruta o controlador:

```php
use Psr\Http\Message\ServerRequestInterface;
 
Route::get('/', function (ServerRequestInterface $request) {
    // ...
});
```

{% hint style="info" %}
Si devuelves una instancia de respuesta PSR-7 desde una ruta o controlador, se convertirá automáticamente en una instancia de respuesta Laravel y será mostrada por el framework.
{% endhint %}

## Input

### Recuperación de inputs

#### Recuperación de todos los datos de entrada

Puede recuperar todos los datos de entrada de la petición entrante como un `array` utilizando el método `all`. Este método se puede utilizar independientemente de si la solicitud entrante proviene de un formulario HTML o es una solicitud XHR:

```php
$input = $request->all();
```

Utilizando el método `collect`, puede recuperar todos los datos de entrada de la solicitud entrante como una [colección](https://laravel.com/docs/10.x/collections):

```php
$input = $request->collect();
```

El método `collect` también permite recuperar un subconjunto de la entrada de la solicitud entrante como una colección:

```php
$request->collect('users')->each(function (string $user) {
    // ...
});
```

#### Recuperar un valor de entrada

Usando unos pocos métodos simples, puede acceder a todas las entradas del usuario desde su instancia `Illuminate\Http\Request` sin preocuparse de qué verbo HTTP se utilizó para la solicitud. Independientemente del verbo HTTP, el método `input` se puede utilizar para recuperar la entrada del usuario:

```php
$name = $request->input('name');
```

Puede pasar un valor por defecto como segundo argumento al método `input`. Este valor se devolverá si el valor de entrada solicitado no está presente en la solicitud:

```php
$name = $request->input('name', 'Sally');
```

Cuando trabaje con formularios que contengan arrays de entrada, utilice la notación "punto" para acceder a los arrays:

```php
$name = $request->input('products.0.name');
 
$names = $request->input('products.*.name');
```

Puede llamar al método `input` sin ningún argumento para recuperar todos los valores de entrada como un array asociativo:

```php
$input = $request->input();
```

#### Obtención de datos desde Query String

Mientras que el método `input` recupera valores de toda la carga útil de la petición (incluida la cadena de consulta), el método `query` sólo recuperará valores de la cadena de consulta:

```php
$name = $request->query('name');
```

Si los datos del valor de la cadena de consulta solicitada no están presentes, se devolverá el segundo argumento de este método:

```php
$name = $request->query('name', 'Helen');
```

Puede llamar al método `query` sin ningún argumento para recuperar todos los valores de la cadena de consulta como un array asociativo:

```php
$query = $request->query();
```

#### Recuperando valores de entrada JSON

Cuando envíes peticiones JSON a tu aplicación, puedes acceder a los datos JSON a través del método `input` siempre que la cabecera `Content-Type` de la petición esté correctamente configurada como `application/json`. Incluso puede utilizar la sintaxis "dot" para recuperar valores que están anidados dentro de matrices / objetos JSON:

```php
$name = $request->input('user.name');
```

#### Recuperando valores de entrada encadenables

En lugar de recuperar los datos de entrada de la petición como una `string` primitiva, puede utilizar el método `string` para recuperar los datos de la petición como una instancia de [`Illuminate\Support\Stringable`](https://laravel.com/docs/10.x/helpers#fluent-strings):

```php
$name = $request->string('name')->trim();
```

#### Recuperación de valores booleanos de entrada

Al tratar con elementos HTML como las casillas de verificación, su aplicación puede recibir valores "de verdad" que en realidad son cadenas. Por ejemplo, "true" o "on". Por conveniencia, puede utilizar el método `boolean` para recuperar estos valores como booleanos. El método `boolean` devuelve `true` para 1, "1", true, "true", "on" y "yes". Todos los demás valores devolverán `false`:

```php
$archived = $request->boolean('archived');
```

#### Recuperación de valores de entrada de fecha

Para mayor comodidad, los valores de entrada que contienen fechas / horas pueden recuperarse como instancias de Carbon utilizando el método `date`. Si la solicitud no contiene un valor de entrada con el nombre dado, se devolverá `null`:

```php
$birthday = $request->date('birthday');
```

Los argumentos segundo y tercero aceptados por el método `date` pueden utilizarse para especificar el formato de la fecha y la zona horaria, respectivamente:

```php
$elapsed = $request->date('elapsed', '!H:i', 'Europe/Madrid');
```

Si el valor de entrada está presente pero tiene un formato inválido, se lanzará una `InvalidArgumentException`; por lo tanto, se recomienda validar la entrada antes de invocar el método `date`.

#### Recuperación de valores de entrada Enum

Los valores de entrada que corresponden a [PHP enums](https://www.php.net/manual/en/language.types.enumerations.php) también pueden ser recuperados de la petición. Si la petición no contiene un valor de entrada con el nombre dado o el enum no tiene un valor de respaldo que coincida con el valor de entrada, se devolverá `null`. El método `enum` acepta el nombre del valor de entrada y la clase enum como primer y segundo argumento:

```php
use App\Enums\Status;
 
$status = $request->enum('status', Status::class);
```

#### Recuperación de entradas mediante propiedades dinámicas

También puede acceder a la entrada del usuario utilizando propiedades dinámicas en la instancia `Illuminate\Http\Request`. Por ejemplo, si uno de los formularios de su aplicación contiene un campo `name`, puede acceder al valor del campo de esta manera:

```php
$name = $request->name;
```

Cuando se utilizan propiedades dinámicas, Laravel buscará primero el valor del parámetro en la carga útil de la petición. Si no está presente, Laravel buscará el campo en los parámetros de la ruta correspondiente.

#### Recuperación de una parte de los datos de entrada

Si necesita recuperar un subconjunto de los datos de entrada, puede utilizar los métodos `only` y `except`. Ambos métodos aceptan un único `array` o una lista dinámica de argumentos:

```php
$input = $request->only(['username', 'password']);
 
$input = $request->only('username', 'password');
 
$input = $request->except(['credit_card']);
 
$input = $request->except('credit_card');
```

{% hint style="info" %}
El método `only` devuelve todos los pares clave/valor solicitados; sin embargo, no devolverá pares clave/valor que no estén presentes en la solicitud.
{% endhint %}

### Determinar si la entrada está presente

Puede utilizar el método `has` para determinar si un valor está presente en la petición. El método `has` devuelve `true` si el valor está presente en la petición:

```php
if ($request->has('name')) {
    // ...
}
```

Cuando se le da un array, el método `has` determinará si todos los valores especificados están presentes:

```php
if ($request->has(['name', 'email'])) {
    // ...
}
```

El método `whenHas` ejecutará el closure dado si hay un valor presente en la petición:

```php
$request->whenHas('name', function (string $input) {
    // ...
});
```

Se puede pasar un segundo closure al método `whenHas` que se ejecutará si el valor especificado no está presente en la petición:

```php
$request->whenHas('name', function (string $input) {
    // The "name" value is present...
}, function () {
    // The "name" value is not present...
});
```

El método `hasAny` devuelve `true` si alguno de los valores especificados está presente:

```php
if ($request->hasAny(['name', 'email'])) {
    // ...
}
```

Si desea determinar si un valor está presente en la solicitud y no es una cadena vacía, puede utilizar el método `filled`:

```php
if ($request->filled('name')) {
    // ...
}
```

El método `whenFilled` ejecutará el closure dado si hay un valor presente en la petición y no es una cadena vacía:

```php
$request->whenFilled('name', function (string $input) {
    // ...
});
```

Se puede pasar un segundo closure al método `whenFilled` que se ejecutará si el valor especificado no está "lleno":

```php
$request->whenFilled('name', function (string $input) {
    // The "name" value is filled...
}, function () {
    // The "name" value is not filled...
});
```

Para determinar si una clave dada está ausente en la solicitud, puede utilizar los métodos `missing` y `whenMissing`:

```php
if ($request->missing('name')) {
    // ...
}
 
$request->whenMissing('name', function (array $input) {
    // The "name" value is missing...
}, function () {
    // The "name" value is present...
});
```
