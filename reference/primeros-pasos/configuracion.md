# Configuración

## Introducción

Todos los archivos de configuración para el framework Laravel se almacenan en el directorio `config`. Cada opción está documentada, así que no dudes en echar un vistazo a los archivos y familiarizarte con las opciones disponibles.

Estos archivos de configuración le permiten configurar cosas como la información de conexión a su base de datos, la información de su servidor de correo, así como otros valores de configuración básicos como la zona horaria de su aplicación y la clave de encriptación.

## Visión general de la aplicación

¿Tienes prisa? Puedes obtener una rápida visión general de la configuración, controladores y entorno de tu aplicación a través del comando `about` Artisan:

```shell
php artisan about
```

Si sólo le interesa una sección concreta de la vista general de la aplicación, puede filtrarla con la opción `--only`:

```shell
php artisan about --only=environmenth
```

