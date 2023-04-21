# Protección CSRF

## Introducción

Las falsificaciones de petición en sitios cruzados son un tipo de exploit malicioso mediante el cual se ejecutan comandos no autorizados en nombre de un usuario autenticado. Afortunadamente, Laravel hace que sea fácil de proteger su aplicación de [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site\_request\_forgery) (CSRF) ataques.

#### Una explicación de la vulnerabilidad

En caso de que no estés familiarizado con las falsificaciones de peticiones entre sitios, vamos a ver un ejemplo de cómo se puede explotar esta vulnerabilidad. Imagina que tu aplicación tiene una ruta `/user/email` que acepta una petición `POST` para cambiar la dirección de correo electrónico del usuario autenticado. Lo más probable es que esta ruta espere un campo de entrada `email` que contenga la dirección de correo electrónico que el usuario quiere empezar a utilizar.

Sin protección CSRF, un sitio web malicioso podría crear un formulario HTML que apunte a la ruta `/user/email` de su aplicación y envíe la propia dirección de correo electrónico del usuario malicioso:

```atom
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="malicious-email@example.com">
</form>
 
<script>
    document.forms[0].submit();
</script>
```

Si el sitio web malicioso envía automáticamente el formulario cuando se carga la página, el usuario malicioso sólo tiene que atraer a un usuario desprevenido de su aplicación para que visite su sitio web y su dirección de correo electrónico se cambiará en su aplicación.

Para prevenir esta vulnerabilidad, necesitamos inspeccionar cada petición entrante `POST`, `PUT`, `PATCH`, o `DELETE` en busca de un valor de sesión secreto al que la aplicación maliciosa no pueda acceder.

## Prevención de solicitudes CSRF

Laravel genera automáticamente un "token" CSRF para cada [sesión de usuario](https://laravel.com/docs/10.x/session) activa gestionada por la aplicación. Este token se utiliza para verificar que el usuario autenticado es la persona que realmente realiza las peticiones a la aplicación. Como este token se almacena en la sesión del usuario y cambia cada vez que se regenera la sesión, una aplicación maliciosa no puede acceder a él.

Se puede acceder al token CSRF de la sesión actual a través de la sesión de la solicitud o a través de la función de ayuda `csrf_token`:

```php
use Illuminate\Http\Request;
 
Route::get('/token', function (Request $request) {
    $token = $request->session()->token();
 
    $token = csrf_token();
 
    // ...
});
```

Cada vez que defina un formulario HTML "POST", "PUT", "PATCH" o "DELETE" en su aplicación, debe incluir un campo oculto CSRF `_token` en el formulario para que el middleware de protección CSRF pueda validar la petición. Para mayor comodidad, puede utilizar la directiva `@csrf` Blade para generar el campo de entrada de token oculto:

```atom
<form method="POST" action="/profile">
    @csrf
 
    <!-- Equivalent to... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

El `App\Http\Middleware\VerifyCsrfToken` [middleware](https://laravel.com/docs/10.x/middleware), que está incluido en el grupo middleware `web` por defecto, verificará automáticamente que el token de la petición coincide con el token almacenado en la sesión. Cuando estos dos tokens coinciden, sabemos que el usuario autenticado es el que inicia la petición.

### Tokens CSRF y SPA

Si estás construyendo una SPA que utiliza Laravel como API backend, deberías consultar la [Laravel Sanctum documentation](https://laravel.com/docs/10.x/sanctum) para obtener información sobre la autenticación con tu API y la protección contra vulnerabilidades CSRF.

### Excluir URIs de la protección CSRF

A veces puede que desee excluir un conjunto de URIs de la protección CSRF. Por ejemplo, si está utilizando [Stripe](https://stripe.com) para procesar pagos y está utilizando su sistema de webhook, necesitará excluir su ruta de controlador de webhook de Stripe de la protección CSRF ya que Stripe no sabrá qué token CSRF enviar a sus rutas.

Normalmente, deberías colocar este tipo de rutas fuera del grupo de middleware `web` que el `App\Providers\RouteServiceProvider` aplica a todas las rutas en el fichero `routes/web.php`. Sin embargo, también puede excluir las rutas añadiendo sus URIs a la propiedad `$except` del middleware `VerifyCsrfToken`:

```php
<?php
 
namespace App\Http\Middleware;
 
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;
 
class VerifyCsrfToken extends Middleware
{
    /**
     * The URIs that should be excluded from CSRF verification.
     *
     * @var array
     */
    protected $except = [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
    ];
}
```

{% hint style="info" %}
Por comodidad, el middleware CSRF se desactiva automáticamente para todas las rutas al [ejecutar pruebas](https://laravel.com/docs/10.x/testing).
{% endhint %}

## X-CSRF-TOKEN

Además de comprobar el token CSRF como parámetro POST, el middleware `App\Http\Middleware\VerifyCsrfToken` también comprobará la cabecera de petición `X-CSRF-TOKEN`. Podría, por ejemplo, almacenar el token en una etiqueta HTML `meta`:

```atom
<meta name="csrf-token" content="{{ csrf_token() }}">
```

A continuación, puede indicar a una biblioteca como jQuery que añada automáticamente el token a todas las cabeceras de las solicitudes. Esto proporciona una protección CSRF simple y conveniente para sus aplicaciones basadas en AJAX que utilizan tecnología JavaScript heredada:

```javascript
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

## X-XSRF-TOKEN

Laravel almacena el token CSRF actual en una cookie encriptada `XSRF-TOKEN` que se incluye con cada respuesta generada por el framework. Puedes usar el valor de la cookie para establecer la cabecera de petición `X-XSRF-TOKEN`.

Esta cookie se envía principalmente para comodidad del desarrollador, ya que algunos frameworks y bibliotecas de JavaScript, como Angular y Axios, colocan automáticamente su valor en el encabezado `X-XSRF-TOKEN` en las solicitudes del mismo origen.

{% hint style="info" %}
Por defecto, el archivo `resources/js/bootstrap.js` incluye la librería Axios HTTP que enviará automáticamente la cabecera `X-XSRF-TOKEN` por ti.
{% endhint %}

