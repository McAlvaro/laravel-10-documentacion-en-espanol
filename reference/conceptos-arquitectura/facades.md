# Facades

## Introducción

A lo largo de la documentación de Laravel, verás ejemplos de código que interactúa con las características de Laravel a través de “facades". Las facades proporcionan una interfaz "estática" a las clases que están disponibles en el [contenedor de servicios de la aplicación](https://laravel.com/docs/10.x/container). Laravel viene con muchas facades que proporcionan acceso a casi todas las características de Laravel.

Las facades de Laravel sirven como "proxies estáticos" a las clases subyacentes en el contenedor de servicios, proporcionando el beneficio de una sintaxis breve y expresiva, manteniendo al mismo tiempo más capacidad de prueba y flexibilidad que los métodos estáticos tradicionales. No pasa nada si no entiendes del todo cómo funcionan las facades, simplemente déjate llevar y sigue aprendiendo sobre Laravel.

Todas las facades de Laravel están definidas en el espacio de nombres `Illuminate\Support\Facades`. Por lo tanto, podemos acceder fácilmente a una fachada así:

```php
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Route;
 
Route::get('/cache', function () {
    return Cache::get('key');
});
```

A lo largo de la documentación de Laravel, muchos de los ejemplos utilizarán fachadas para demostrar diversas características del framework.

#### Funciones Helpers

Para complementar las facades, Laravel ofrece una variedad de "funciones de ayuda" globales que hacen aún más fácil interactuar con las características comunes de Laravel. Algunas de las funciones de ayuda comunes con las que puedes interactuar son `view`, `response`, `url`, `config`, y más. Cada función helper ofrecida por Laravel está documentada con su característica correspondiente; sin embargo, una lista completa está disponible en la [documentación helper dedicada](https://laravel.com/docs/10.x/helpers).

Por ejemplo, en lugar de utilizar la fachada `Illuminate\Support\Facades\Response` para generar una respuesta JSON, podemos utilizar simplemente la función `response`. Dado que las funciones helper están disponibles globalmente, no es necesario importar ninguna clase para utilizarlas:

```php
use Illuminate\Support\Facades\Response;
 
Route::get('/users', function () {
    return Response::json([
        // ...
    ]);
});
 
Route::get('/users', function () {
    return response()->json([
        // ...
    ]);
});
```

## Cuándo utilizar Facades

Las facades tienen muchos beneficios. Proporcionan una sintaxis concisa y fácil de recordar que te permite utilizar las características de Laravel sin tener que recordar largos nombres de clases que tienen que ser inyectados o configurados manualmente. Además, debido a su uso único de los métodos dinámicos de PHP, son fáciles de probar.

Sin embargo, hay que tener cuidado al utilizar facades. El principal peligro de las facades es el "scope creep" de las clases. Dado que las facades son tan fáciles de usar y no requieren inyección, puede ser fácil dejar que tus clases sigan creciendo y usar muchas facades en una sola clase. Usando inyección de dependencia, este potencial es mitigado por la retroalimentación visual que un constructor grande te da de que tu clase está creciendo demasiado. Así que, cuando uses facades, presta especial atención al tamaño de tu clase para que su ámbito de responsabilidad se mantenga estrecho. Si tu clase está creciendo demasiado, considera dividirla en múltiples clases más pequeñas.

### Facades vs inyección de dependencia

Una de las principales ventajas de la inyección de dependencias es la posibilidad de intercambiar implementaciones de la clase inyectada. Esto es útil durante las pruebas, ya que puede inyectar un simulacro o stub y afirmar que varios métodos fueron llamados en el stub.

Típicamente, no sería posible hacer un mock o stub de un método de clase verdaderamente estático. Sin embargo, dado que las facades utilizan métodos dinámicos para delegar llamadas a métodos de objetos resueltos desde el contenedor de servicios, podemos probar las facades del mismo modo que probaríamos una instancia de clase inyectada. Por ejemplo, dada la siguiente ruta:

```php
use Illuminate\Support\Facades\Cache;
 
Route::get('/cache', function () {
    return Cache::get('key');
});
```

Usando los métodos de prueba de facade de Laravel, podemos escribir la siguiente prueba para verificar que el método `Cache::get` fue llamado con el argumento que esperábamos:

```php
use Illuminate\Support\Facades\Cache;
 
/**
 * A basic functional test example.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');
 
    $response = $this->get('/cache');
 
    $response->assertSee('value');
}
```

### Facades vs Funciones Helpers

Además de las facades, Laravel incluye una variedad de funciones "helper" que pueden realizar tareas comunes como generar vistas, disparar eventos, despachar trabajos o enviar respuestas HTTP. Muchas de estas funciones helper realizan la misma función que la facade correspondiente. Por ejemplo, esta llamada a la facade y la llamada al helper son equivalentes:

```php
return Illuminate\Support\Facades\View::make('profile');
 
return view('profile');
```

No hay ninguna diferencia práctica entre las facades y las funciones de ayuda. Al utilizar funciones auxiliares, puede probarlas exactamente igual que lo haría con la facade correspondiente. Por ejemplo, dada la siguiente ruta:

```php
Route::get('/cache', function () {
    return cache('key');
});
```

El helper `cache` va a llamar al método `get` de la clase subyacente a la facade `Cache`. Así que, aunque estemos usando la función helper, podemos escribir el siguiente test para verificar que el método fue llamado con el argumento que esperábamos:

```php
use Illuminate\Support\Facades\Cache;
 
/**
 * A basic functional test example.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');
 
    $response = $this->get('/cache');
 
    $response->assertSee('value');
}
```

