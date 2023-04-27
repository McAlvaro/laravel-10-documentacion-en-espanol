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

