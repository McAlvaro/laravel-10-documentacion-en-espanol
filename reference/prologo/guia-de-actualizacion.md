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
