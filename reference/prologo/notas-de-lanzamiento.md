# Notas de Lanzamiento

## Esquema de Versionamiento

Laravel y sus otros paquetes siguen el [Versionado Semántico](https://semver.org/). Las versiones mayores del framework se publican cada año (\~Q1), mientras que las versiones menores y de parche pueden publicarse cada semana. Las versiones menores y de parche nunca deben contener cambios de última hora.

Cuando hagas referencia al framework Laravel o a sus componentes desde tu aplicación o paquete, deberías usar siempre una restricción de versión como `^10.0`, ya que las versiones mayores de Laravel incluyen cambios de última hora. Sin embargo, nos esforzamos para asegurar que siempre se puede actualizar a una nueva versión principal en un día o menos.

### Argumentos con Nombre

Los argumentos con nombre no están cubiertos por las directrices de compatibilidad con versiones anteriores de Laravel. Podemos optar por cambiar el nombre de los argumentos de función cuando sea necesario con el fin de mejorar la base de código Laravel. Por lo tanto, el uso de argumentos con nombre al llamar a métodos Laravel debe hacerse con cautela y con la comprensión de que los nombres de los parámetros pueden cambiar en el futuro.

## Políticas de Soporte

Para todas las versiones de Laravel, se proporcionan correcciones de errores durante 18 meses y correcciones de seguridad durante 2 años. Para todas las bibliotecas adicionales, incluyendo Lumen, sólo la última versión principal recibe correcciones de errores. Además, por favor revise las versiones de bases de datos soportadas por Laravel.

| Versión | PHP (\*)  | Lanzamiento             | Corrección de errores hasta | Corrección de seguridad hasta |
| ------- | --------- | ----------------------- | --------------------------- | ----------------------------- |
| 8       | 7.3 - 8.1 | 8 de septiembre de 2020 | 26 de julio de 2022         | 24 de enero de 2023           |
| 9       | 8.0 - 8.2 | 8 de febrero de 2022    | 8 de agosto de 2023         | 6 de febrero de 2024          |
| 10      | 8.1 - 8.2 | 14 de febrero de 2023   | 6 de agosto de 2024         | 4 de febrero de 2025          |
| 11      | 8.2       | T1 2024                 | 5 de agosto de 2025         | 3 de febrero de 2026          |

## Laravel 10

Como ya sabrás, Laravel hizo la transición a versiones anuales con el lanzamiento de Laravel 8. Anteriormente, las versiones principales se lanzaban cada 6 meses. Esta transición tiene como objetivo aliviar la carga de mantenimiento de la comunidad y desafiar a nuestro equipo de desarrollo para enviar nuevas características sorprendentes y potentes sin introducir cambios de última hora. Por lo tanto, hemos enviado una variedad de características robustas a Laravel 9 sin romper la compatibilidad con versiones anteriores.

Por lo tanto, este compromiso de lanzar grandes novedades durante la versión actual probablemente hará que las futuras versiones "mayores" se utilicen principalmente para tareas de "mantenimiento", como la actualización de las dependencias aguas arriba, como puede verse en estas notas de la versión.

Laravel 10 continúa las mejoras realizadas en Laravel 9.x introduciendo argumentos y tipos de retorno a todos los métodos del esqueleto de la aplicación, así como a todos los archivos stub utilizados para generar clases en todo el framework. Además, se ha introducido una nueva capa de abstracción fácil de usar por los desarrolladores para iniciar procesos externos e interactuar con ellos. Además, Laravel Pennant se ha introducido para proporcionar un enfoque maravilloso para la gestión de su aplicación "feature flags".

## PHP 8.1

Laravel 10.x requiere una versión mínima de PHP 8.1.

## Tipos

[Nuno Maduro](https://github.com/nunomaduro) ha contribuido con el esqueleto de la aplicación y las sugerencias de tipo.

En su lanzamiento inicial, Laravel utilizó todas las características de type-hinting disponibles en PHP en ese momento. Sin embargo, muchas nuevas características han sido añadidas a PHP en los años posteriores, incluyendo sugerencias de tipos primitivos adicionales, tipos de retorno y tipos de unión.

Laravel 10.x actualiza a fondo el esqueleto de la aplicación y todos los stubs utilizados por el framework para introducir argumentos y tipos de retorno a todas las firmas de métodos. Además, se ha eliminado la información de sugerencia de tipo "doc block".

Este cambio es totalmente compatible con las aplicaciones existentes. Por lo tanto, las aplicaciones existentes que no tengan estas sugerencias de tipo seguirán funcionando con normalidad.

## Laravel Pennant

_Laravel Pennant fue desarrollado por_ [_Tim MacDonald_](https://github.com/timacdonald)_._

Un nuevo paquete de primera parte, Laravel Pennant, ha sido liberado. Laravel Pennant ofrece un enfoque ligero y optimizado para la gestión de las banderas de características de su aplicación. Pennant incluye un controlador de `array` en memoria y un controlador de `database` para el almacenamiento persistente de características.

Las características pueden definirse fácilmente mediante el método Feature::define:

```php
use Laravel\Pennant\Feature;
use Illuminate\Support\Lottery;
 
Feature::define('new-onboarding-flow', function () {
    return Lottery::odds(1, 10);
});
```

Una vez definida una función, puede determinar fácilmente si el usuario actual tiene acceso a la función en cuestión:

```php
if (Feature::active('new-onboarding-flow')) {
    // ...
}
```

Por supuesto, para mayor comodidad, también están disponibles las directivas Blade:

```php
@feature('new-onboarding-flow')
    <div>
        <!-- ... -->
    </div>
@endfeature
```

Pennant ofrece una variedad de funciones y API más avanzadas. Para más información, consulte la completa [documentación de Pennant](https://laravel.com/docs/10.x/pennant).

## Interacción de Procesos

_La capa de abstracción de procesos fue aportada por_ [_Nuno Maduro_](https://github.com/nunomaduro) _y_ [_Taylor Otwell_](https://github.com/taylorotwell)_._

Laravel 10.x introduce una hermosa capa de abstracción para iniciar e interactuar con procesos externos a través de una nueva fachada `Process`:

```php
use Illuminate\Support\Facades\Process;
 
$result = Process::run('ls -la');
 
return $result->output();
```

Los procesos pueden incluso iniciarse en pools, lo que permite una cómoda ejecución y gestión de procesos concurrentes:

```php
use Illuminate\Process\Pool;
use Illuminate\Support\Facades\Process;
 
[$first, $second, $third] = Process::concurrently(function (Pool $pool) {
    $pool->command('cat first.txt');
    $pool->command('cat second.txt');
    $pool->command('cat third.txt');
});
 
return $first->output();
```

Además, los procesos pueden falsificarse para facilitar las pruebas:

```php
Process::fake();
 
// ...
 
Process::assertRan('ls -la');
```

Para más información sobre la interacción con los procesos, [consulte la documentación completa de los mismos](https://laravel.com/docs/10.x/processes).

## Perfiles de Pruebas

[N_uno Maduro_](https://github.com/nunomaduro) _ha contribuido a la elaboración de los perfiles de las pruebas._

El comando de `test` Artisan ha recibido una nueva opción `--profile` que le permite identificar fácilmente las pruebas más lentas de su aplicación:

```shell
php artisan test --profile
```

Para mayor comodidad, las pruebas más lentas se mostrarán directamente en la salida CLI:

<figure><img src="https://user-images.githubusercontent.com/5457236/217328439-d8d983ec-d0fc-4cde-93d9-ae5bccf5df14.png" alt=""><figcaption></figcaption></figure>

