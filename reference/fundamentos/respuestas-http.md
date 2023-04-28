# Respuestas HTTP

## Creando respuestas

#### Cadenas y arreglos

Todas las rutas y controladores deben devolver una respuesta para ser enviada de vuelta al navegador del usuario. Laravel proporciona varias formas diferentes de devolver respuestas. La respuesta más básica es devolver una cadena desde una ruta o controlador. El framework convertirá automáticamente la cadena en una respuesta HTTP completa:

```php
Route::get('/', function () {
    return 'Hello World';
});
```

Además de devolver cadenas desde sus rutas y controladores, también puede devolver matrices. El framework convertirá automáticamente el array en una respuesta JSON:

```php
Route::get('/', function () {
    return [1, 2, 3];
});
```

{% hint style="info" %}
¿Sabías que también puedes devolver [Eloquent collections](https://laravel.com/docs/10.x/eloquent-collections) desde tus rutas o controladores? Se convertirán automáticamente a JSON. ¡Pruébalo!
{% endhint %}

#### Objetos de respuesta

Típicamente, no devolverás simples cadenas y arreglos de tus acciones de ruta. En su lugar, devolverá instancias completas de `Illuminate\Http\Response` o [views](https://laravel.com/docs/10.x/views).

Devolver una instancia completa de `Response` te permite personalizar el código de estado HTTP de la respuesta y las cabeceras. Una instancia `Response` hereda de la clase `Symfony\Component\HttpFoundation\Response`, que proporciona una variedad de métodos para construir respuestas HTTP:

```php
Route::get('/home', function () {
    return response('Hello World', 200)
                  ->header('Content-Type', 'text/plain');
});
```

#### Modelos y colecciones Eloquent

También puedes devolver modelos y colecciones [Eloquent ORM](https://laravel.com/docs/10.x/eloquent) directamente desde tus rutas y controladores. Cuando lo hagas, Laravel convertirá automáticamente los modelos y colecciones a respuestas JSON respetando los [atributos ocultos](https://laravel.com/docs/10.x/eloquent-serialization#hiding-attributes-from-json) del modelo:

```php
use App\Models\User;
 
Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

### Adjuntar encabezados a las respuestas

Ten en cuenta que la mayoría de los métodos de respuesta son encadenables, lo que permite la construcción fluida de instancias de respuesta. Por ejemplo, puedes utilizar el método `header` para añadir una serie de header a la respuesta antes de enviarla de vuelta al usuario:

```php
return response($content)
            ->header('Content-Type', $type)
            ->header('X-Header-One', 'Header Value')
            ->header('X-Header-Two', 'Header Value');
```

También puede utilizar el método `withHeaders` para especificar una matriz de cabeceras que se añadirán a la respuesta:

```php
return response($content)
            ->withHeaders([
                'Content-Type' => $type,
                'X-Header-One' => 'Header Value',
                'X-Header-Two' => 'Header Value',
            ]);
```

#### Middleware de control de caché

Laravel incluye un middleware `cache.headers`, que puede ser utilizado para establecer rápidamente el header `Cache-Control` para un grupo de rutas. Las directivas deben proporcionarse usando el equivalente "snake case" de la directiva cache-control correspondiente y deben ir separadas por punto y coma. Si se especifica `etag` en la lista de directivas, se establecerá automáticamente un hash MD5 del contenido de la respuesta como identificador ETag:

```php
Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });
 
    Route::get('/terms', function () {
        // ...
    });
});
```

### Adjuntar cookies a las respuestas

Puede adjuntar una cookie a una instancia de `Illuminate\Http\Response` saliente utilizando el método `cookie`. Debe pasar a este método el nombre, el valor y el número de minutos que la cookie debe considerarse válida:

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

El método `cookie` también acepta algunos argumentos más que se usan con menos frecuencia. Generalmente, estos argumentos tienen el mismo propósito y significado que los argumentos que se darían al método nativo de PHP [setcookie](https://secure.php.net/manual/en/function.setcookie.php):

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

Si desea asegurarse de que se envía una cookie con la respuesta saliente pero todavía no tiene una instancia de esa respuesta, puede utilizar la fachada `Cookie` para "poner en cola" cookies para adjuntarlas a la respuesta cuando se envíe. El método `queue` acepta los argumentos necesarios para crear una instancia de cookie. Estas cookies se adjuntarán a la respuesta saliente antes de ser enviada al navegador:

```php
use Illuminate\Support\Facades\Cookie;
 
Cookie::queue('name', 'value', $minutes);
```

#### Generación de instancias de cookies

Si quieres generar una instancia `Symfony\Component\HttpFoundation\Cookie` que pueda ser adjuntada a una instancia de respuesta en un momento posterior, puedes usar el helper global `cookie`. Esta cookie no se devolverá al cliente a menos que se adjunte a una instancia de respuesta:

```php
$cookie = cookie('name', 'value', $minutes);
 
return response('Hello World')->cookie($cookie);
```

#### Caducidad anticipada de las cookies

Puede eliminar una cookie expirándola a través del método `withoutCookie` de una respuesta saliente:

```php
return response('Hello World')->withoutCookie('name');
```

Si aún no tiene una instancia de la respuesta saliente, puede utilizar el método `expire` de la fachada `Cookie` para expirar una cookie:

```php
Cookie::expire('name');
```

### Cookies y encriptación

Por defecto, todas las cookies generadas por Laravel son encriptadas y firmadas para que no puedan ser modificadas o leídas por el cliente. Si quieres deshabilitar el cifrado para un subconjunto de cookies generadas por tu aplicación, puedes usar la propiedad `$except` del middleware `App\Http\Middleware\EncryptCookies`, que se encuentra en el directorio `app/Http/Middleware`:

```php
/**
 * The names of the cookies that should not be encrypted.
 *
 * @var array
 */
protected $except = [
    'cookie_name',
];
```

## Redirecciones

Las respuestas de redirección son instancias de la clase `Illuminate\Http\RedirectResponse`, y contienen los encabezados adecuados necesarios para redirigir al usuario a otra URL. Hay varias maneras de generar una instancia `RedirectResponse`. El método más sencillo es utilizar el helper global `redirect`:

```php
Route::get('/dashboard', function () {
    return redirect('home/dashboard');
});
```

A veces es posible que desee redirigir al usuario a su ubicación anterior, como cuando un formulario enviado no es válido. Puede hacerlo utilizando la función global de ayuda `back`. Dado que esta función utiliza la [sesión](https://laravel.com/docs/10.x/session), asegúrese de que la ruta que llama a la función `back` utiliza el grupo de middleware `web`:

```php
Route::post('/user/profile', function () {
    // Validate the request...
 
    return back()->withInput();
});
```

### Redirección a rutas con nombre

Cuando se llama al helper `redirect` sin parámetros, se devuelve una instancia de `Illuminate\Routing\Redirector`, lo que permite llamar a cualquier método en la instancia `Redirector`. Por ejemplo, para generar un `RedirectResponse` a una ruta con nombre, puede utilizar el método `route`:

```php
return redirect()->route('login');
```

Si su ruta tiene parámetros, puede pasarlos como segundo argumento al método `route`:

```php
// For a route with the following URI: /profile/{id}
 
return redirect()->route('profile', ['id' => 1]);
```

#### Rellenar parámetros mediante modelos Eloquent

Si está redirigiendo a una ruta con un parámetro "ID" que se está rellenando desde un modelo de Eloquent, puede pasar el propio modelo. El ID se extraerá automáticamente:

```php
// For a route with the following URI: /profile/{id}
 
return redirect()->route('profile', [$user]);
```

Si desea personalizar el valor que se coloca en el parámetro de ruta, puede especificar la columna en la definición del parámetro de ruta (`/profile/{id:slug}`) o puede anular el método `getRouteKey` en su modelo Eloquent:

```php
/**
 * Get the value of the model's route key.
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

### Redirección a acciones del controlador

También puede generar redirecciones a [acciones del controlador](https://laravel.com/docs/10.x/controllers). Para ello, pase el controlador y el nombre de la acción al método `action`:

```php
use App\Http\Controllers\UserController;
 
return redirect()->action([UserController::class, 'index']);
```

Si la ruta de tu controlador requiere parámetros, puedes pasarlos como segundo argumento al método `action`:

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

### Redirección a dominios externos

A veces puede necesitar redirigir a un dominio fuera de su aplicación. Puedes hacerlo llamando al método `away`, que crea un `RedirectResponse` sin ninguna codificación de URL adicional, validación o verificación:

```php
return redirect()->away('https://www.google.com');
```

### Redirección con datos de sesión flasheados

Redirigir a una nueva URL y [flashear datos a la sesión](https://laravel.com/docs/10.x/session#flash-data) suelen hacerse al mismo tiempo. Normalmente, esto se hace después de realizar con éxito una acción cuando se flashea un mensaje de éxito a la sesión. Para mayor comodidad, puede crear una instancia `RedirectResponse` y flashear datos a la sesión en una única cadena de métodos fluida:

```php
Route::post('/user/profile', function () {
    // ...
 
    return redirect('dashboard')->with('status', 'Profile updated!');
});
```

Después de redirigir al usuario, puede mostrar el mensaje flasheado de la [sesión](https://laravel.com/docs/10.x/session). Por ejemplo, utilizando [sintaxis de Blade](https://laravel.com/docs/10.x/blade):

```atom
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

#### Redirección con input

Puede utilizar el método `withInput` proporcionado por la instancia `RedirectResponse` para flashear los datos de entrada de la solicitud actual en la sesión antes de redirigir al usuario a una nueva ubicación. Esto se hace normalmente si el usuario ha encontrado un error de validación. Una vez que la entrada ha sido flasheada a la sesión, puede fácilmente [recuperarla](https://laravel.com/docs/10.x/requests#retrieving-old-input) durante la siguiente petición para repoblar el formulario:

```php
return back()->withInput();
```

## Otros tipos de respuestas

El helper `response` puede utilizarse para generar otros tipos de instancias de respuesta. Cuando se llama al helper `response` sin argumentos, se devuelve una implementación del \[contrato] `Illuminate\Contracts\Routing\ResponseFactory`(https://laravel.com/docs/10.x/contracts). Este contrato proporciona varios métodos útiles para generar respuestas.

### Respuestas de vista

Si necesita controlar el estado y las cabeceras de la respuesta pero también necesita devolver una [vista](https://laravel.com/docs/10.x/views) como contenido de la respuesta, debe utilizar el método `view`:

```php
return response()
            ->view('hello', $data, 200)
            ->header('Content-Type', $type);
```

Por supuesto, si no necesita pasar un código de estado HTTP personalizado o encabezados personalizados, puede utilizar la función helper global `view`.

### Respuestas JSON

El método `json` establecerá automáticamente el header `Content-Type` a `application/json`, así como convertirá el array dado a JSON usando la función PHP `json_encode`:

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```

Si desea crear una respuesta JSONP, puede utilizar el método `json` en combinación con el método `withCallback`:

```php
return response()
            ->json(['name' => 'Abigail', 'state' => 'CA'])
            ->withCallback($request->input('callback'));
```

### Descarga de archivos

El método `download` puede utilizarse para generar una respuesta que fuerce al navegador del usuario a descargar el fichero en la ruta dada. El método `download` acepta un nombre de fichero como segundo argumento del método, que determinará el nombre de fichero que verá el usuario que descargue el fichero. Finalmente, puede pasar un array de cabeceras HTTP como tercer argumento al método:

```php
return response()->download($pathToFile);
 
return response()->download($pathToFile, $name, $headers);
```

{% hint style="info" %}
Symfony HttpFoundation, que gestiona las descargas de archivos, requiere que el archivo que se descarga tenga un nombre de archivo ASCII.
{% endhint %}

#### Descargas en streaming

A veces puede que desee convertir la cadena de respuesta de una operación dada en una respuesta descargable sin tener que escribir el contenido de la operación en el disco. En este caso puede utilizar el método `streamDownload`. Este método acepta como argumentos una llamada de retorno, un nombre de fichero y un array opcional de cabeceras:

```php
use App\Services\GitHub;
 
return response()->streamDownload(function () {
    echo GitHub::api('repo')
                ->contents()
                ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

### Respuestas de archivos

El método `file` puede utilizarse para mostrar un archivo, como una imagen o un PDF, directamente en el navegador del usuario en lugar de iniciar una descarga. Este método acepta la ruta al archivo como primer argumento y una matriz de cabeceras como segundo argumento:

```php
return response()->file($pathToFile);
 
return response()->file($pathToFile, $headers);
```

## Macros de respuesta

Si quieres definir una respuesta personalizada que puedas reutilizar en varias de tus rutas y controladores, puedes utilizar el método `macro` de la fachada `Response`. Normalmente, deberías llamar a este método desde el método `boot` de uno de los [service providers de tu aplicación](https://laravel.com/docs/10.x/providers), como el proveedor de servicios `AppProviders\AppServiceProvider`:

```php
<?php
 
namespace App\Providers;
 
use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;
 
class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Response::macro('caps', function (string $value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

La función `macro` acepta un nombre como primer argumento y un closure como segundo. El closure de la macro se ejecutará cuando se llame al nombre de la macro desde una implementación de `ResponseFactory` o el helper `response`:

```php
return response()->caps('foo');
```
