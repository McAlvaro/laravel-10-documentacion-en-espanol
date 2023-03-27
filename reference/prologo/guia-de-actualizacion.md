# Guía de Actualización

## Actualización a 10.0 desde 9.x

### Tiempo estimado de actualización: 10 Minutos

{% hint style="info" %}
Intentamos documentar todos los cambios de última hora posibles. Dado que algunos de estos cambios de ruptura se encuentran en partes oscuras de la estructura, sólo una parte de estos cambios puede afectar realmente a su aplicación. ¿Quieres ahorrar tiempo? Puede utilizar [Laravel Shift](https://laravelshift.com/) para ayudar a automatizar las actualizaciones de su aplicación.
{% endhint %}

## Actualización de Dependencias

Probabilidad de impacto: Alta

**Requiere PHP 8.1.0**

Laravel ahora requiere PHP 8.1.0 o superior.

**Requiere Composer 2.2.0**

Laravel ahora requiere Composer 2.2.0 o superior.

**Dependencias de Composer**

Debe actualizar las siguientes dependencias en el archivo `composer.json` de su aplicación:

* `laravel/framework` a `^10.0`
* `laravel/sanctum` a `^3.2`
* `doctrine/dbal` a `^3.0`
* `spatie/laravel-ignition` a `^2.0`
* `laravel/passport` a `^11.0`(Guía de Actualización)

Si está actualizando a Sanctum 3.x desde la serie de versiones 2.x, consulte la [guía de actualización de Sanctum](https://github.com/laravel/sanctum/blob/3.x/UPGRADE.md).

Además, si desea utilizar [PHPUnit 10](https://phpunit.de/announcements/phpunit-10.html), debe eliminar el atributo `processUncoveredFiles` de la sección `<coverage>` del archivo de configuración `phpunit.xml` de su aplicación. A continuación, actualiza las siguientes dependencias en el archivo `composer.json` de tu aplicación:

* `nunomaduro/collision` a `^7.0`
* `phpunit/phpunit` a `^10.0`

Por último, examine cualquier otro paquete de terceros consumido por su aplicación y verifique que está utilizando la versión adecuada para la compatibilidad con Laravel 10.

### Estabilidad Mínima

Debe actualizar la configuración de estabilidad mínima en el archivo `composer.json` de su aplicación a estable. O bien, dado que el valor predeterminado de estabilidad mínima es estable, puede eliminar esta configuración del archivo `composer.json` de la aplicación:

```json
"minimum-stability": "stable"
```

## Aplicación

### Vinculación de la Ruta Pública

**Probabilidad de impacto: Baja**

Si su aplicación está personalizando su "ruta pública" vinculando `path.public` en el contenedor, debería actualizar su código para invocar el método `usePublicPath` ofrecido por el objeto `Illuminate\Foundation\Application`:

```php
app()->usePublicPath(__DIR__.'/public');
```

## Autorización

### El Método `registerPolicies`

**Probabilidad de impacto: Baja**

El método `registerPolicies` del `AuthServiceProvider` ahora es invocado automáticamente por el framework. Por lo tanto, puede eliminar la llamada a este método del método de arranque del `AuthServiceProvider` de su aplicación.

## Cache

### Etiquestas de Redis Cache

**Probabilidad de impacto: Media**

El soporte de etiquetas de caché Redis ha sido reescrito para mejorar el rendimiento y la eficiencia de almacenamiento. En versiones anteriores de Laravel, las etiquetas de caché obsoletas se acumulaban en la caché cuando se utilizaba Redis como controlador de caché de la aplicación.

Sin embargo, para podar correctamente las entradas de etiquetas de caché obsoletas, el nuevo comando de Laravel `cache:prune-stale-tags` Artisan debe ser programado en la clase `App\Console\Kernel` de tu aplicación:

```php
$schedule->command('cache:prune-stale-tags')->hourly();
```

## Base de Datos

### Expresiones de Base de Datos

**Probabilidad de impacto: Media**

Las "expresiones" de base de datos (típicamente generadas a través de `DB::raw`) han sido reescritas en Laravel 10.x para ofrecer funcionalidad adicional en el futuro. En particular, el valor de la cadena de la gramática ahora debe ser recuperado a través del método `getValue(Grammar $grammar)` de la expresión. Ya no es posible convertir una expresión en una cadena utilizando `(string)`.

**Normalmente, esto no afecta a las aplicaciones de usuario final;** sin embargo, si su aplicación está convirtiendo manualmente expresiones de base de datos en cadenas utilizando `(string)` o invocando el método `__toString` en la expresión directamente, debe actualizar su código para invocar el método `getValue` en su lugar:

```php
use Illuminate\Support\Facades\DB;
 
$expression = DB::raw('select 1');
 
$string = $expression->getValue(DB::connection()->getQueryGrammar());
```

### **Constructor de excepción de consulta**

**Probabilidad de impacto: Muy baja**

El constructor `Illuminate\Database\QueryException` ahora acepta un nombre de conexión de cadena como su primer argumento. Si su aplicación está lanzando manualmente esta excepción, debe ajustar su código en consecuencia.

## Columnas ULID

**Probabilidad de impacto: Baja**

Cuando las migraciones invocan al método `ulid` sin ningún argumento, la columna se llamará ahora `ulid`. En versiones anteriores de Laravel, al invocar este método sin argumentos se creaba una columna llamada erróneamente `uuid`:

```php
$table->ulid();
```

Para especificar explícitamente un nombre de columna al invocar el método `ulid`, puede pasar el nombre de la columna al método:

```php
$table->ulid('ulid');
```

## Eloquent

### Propiedad "Dates" del Modelo

La propiedad obsoleta `$dates` del modelo Eloquent ha sido eliminada. Su aplicación debe utilizar ahora la propiedad `$casts`:

```php
protected $casts = [
    'deployed_at' => 'datetime',
];
```

### Método `getBaseQuery` de la Relación

**Probabilidad de impacto: Muy baja**

El método `getBaseQuery` de la clase `Illuminate\Database\Eloquent\Relations\Relation` ha sido renombrado a `toBase`.

## Localización

### Directorio de Idiomas

**Probabilidad de impacto: Ninguno**

Aunque no es relevante para las aplicaciones existentes, el esqueleto de la aplicación Laravel ya no contiene el directorio `lang` por defecto. En su lugar, al escribir nuevas aplicaciones Laravel, se puede publicar utilizando el comando `lang:publish` de Artisan:

```shell
php artisan lang:publish
```

## Logging

### Monolog 3

**Probabilidad de impacto: Media**

La dependencia Monolog de Laravel ha sido actualizada a Monolog 3.x. Si estás interactuando directamente con Monolog dentro de tu aplicación, deberías revisar la [guía de actualización de Monolog](https://github.com/Seldaek/monolog/blob/main/UPGRADE.md).

Si está utilizando servicios de registro de terceros como BugSnag o Rollbar, puede que tenga que actualizar los paquetes de terceros a una versión que soporte Monolog 3.x y Laravel 10.x.

## Colas

### El Método `Bus::dispatchNow`

**Probabilidad de impacto: Baja**

Los métodos obsoletos `Bus::dispatchNow` y `dispatch_now` han sido eliminados. En su lugar, la aplicación debe utilizar los métodos `Bus::dispatchSync` y `dispatch_sync`, respectivamente.

## Enrutamiento

### Alias Middleware

**Probabilidad de impacto: Opcional**

En las nuevas aplicaciones Laravel, la propiedad `$routeMiddleware` de la clase `App\Http\Kernel` ha sido renombrada a `$middlewareAliases` para reflejar mejor su propósito. Le invitamos a cambiar el nombre de esta propiedad en sus aplicaciones existentes, sin embargo, no es necesario.

### Valores de retorno del limitador de velocidad

**Probabilidad de impacto: Baja**

Al invocar el método `RateLimiter::attempt`, el valor devuelto por el cierre proporcionado ahora será devuelto por el método. Si no se devuelve nada o `null`, el método `attempt` devolverá `true`:

```php
$value = RateLimiter::attempt('key', 10, fn () => ['example'], 1);
 
$value; // ['example']
```

### El método `Redirect::home`

**Probabilidad de impacto: Muy baja**

El método obsoleto `Redirect::home` ha sido eliminado. En su lugar, su aplicación debe redirigir a una ruta explícitamente nombrada:

```php
return Redirect::route('home');
```

## Testing

### Servicio Mocking

**Probabilidad de impacto: Media**

El trait obsoleto `MocksApplicationServices` ha sido eliminado del framework. Este trait proporcionaba métodos de prueba como `expectsEvents`, `expectsJobs` y `expectsNotifications`.

Si tu aplicación utiliza estos métodos, te recomendamos que cambies a `Event::fake`, `Bus::fake`, y `Notification::fake`, respectivamente. Puedes obtener más información sobre la simulación a través de falsificaciones en la documentación correspondiente al componente que intentas simular.

## Validación

### Mensajes de la regla de validación de Closoure

**Probabilidad de impacto: Muy baja**

Al escribir reglas de validación personalizadas basadas en closoure, si se invoca la llamada de retorno `$fail` más de una vez, los mensajes se añadirán a una matriz en lugar de sobrescribir el mensaje anterior. Normalmente, esto no afectará a su aplicación.

Además, la llamada de retorno `$fail` devuelve ahora un objeto. Si anteriormente estaba indicando el tipo de retorno de su closoure de validación, esto puede requerir que actualice su indicación de tipo:

```php
public function rules()
{
    'name' => [
        function ($attribute, $value, $fail) {
            $fail('validation.translation.key')->translate();
        },
    ],
}
```

## Varios

También le animamos a ver los cambios en el `laravel/laravel` [repositorio GitHub](https://github.com/laravel/laravel). Aunque muchos de estos cambios no son necesarios, es posible que desee mantener estos archivos sincronizados con su aplicación. Algunos de estos cambios serán cubiertos en esta guía de actualización, pero otros, como los cambios en los archivos de configuración o comentarios, no lo serán.

Puedes ver fácilmente los cambios con la [herramienta de comparación de GitHub](https://github.com/laravel/laravel/compare/9.x...10.x) y elegir qué actualizaciones son importantes para ti. Sin embargo, muchos de los cambios mostrados por la herramienta de comparación de GitHub se deben a la adopción por parte de nuestra organización de tipos nativos de PHP. Estos cambios son compatibles con versiones anteriores y su adopción durante la migración a Laravel 10 es opcional.
