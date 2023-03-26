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
