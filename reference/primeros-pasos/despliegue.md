# Despliegue

## Introducción

Cuando estés listo para desplegar tu aplicación Laravel en producción, hay algunas cosas importantes que puedes hacer para asegurarte de que tu aplicación se ejecuta de la manera más eficiente posible. En este documento, vamos a cubrir algunos grandes puntos de partida para asegurarse de que su aplicación Laravel se despliega correctamente.

## Requisitos del servidor

El framework Laravel tiene algunos requisitos de sistema. Debe asegurarse de que su servidor web tiene la siguiente versión mínima de PHP y extensiones:

* PHP >= 8.1
* Ctype PHP Extension
* cURL PHP Extension
* DOM PHP Extension
* Fileinfo PHP Extension
* Filter PHP Extension
* Hash PHP Extension
* Mbstring PHP Extension
* OpenSSL PHP Extension
* PCRE PHP Extension
* PDO PHP Extension
* Session PHP Extension
* Tokenizer PHP Extension
* XML PHP Extension

## Configuración del servidor

### Nginx

Si está desplegando su aplicación en un servidor que ejecuta Nginx, puede utilizar el siguiente archivo de configuración como punto de partida para configurar su servidor web. Lo más probable es que este archivo tenga que ser personalizado dependiendo de la configuración de su servidor. **Si deseas asistencia en la gestión de tu servidor, considera el uso de un servicio de gestión y despliegue de servidores Laravel como** [**Laravel Forge**](https://forge.laravel.com)**.**

Por favor, asegúrate de que, como en la configuración de abajo, tu servidor web dirige todas las peticiones al archivo `public/index.php` de tu aplicación. Nunca debe intentar mover el archivo `index.php` a la raíz de su proyecto, ya que servir la aplicación desde la raíz del proyecto expondrá muchos archivos de configuración sensibles a la Internet pública:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /srv/example.com/public;
 
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
 
    index index.php;
 
    charset utf-8;
 
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
 
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }
 
    error_page 404 /index.php;
 
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
 
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

## Optimización

### Optimización del Autoloader

Al desplegar en producción, asegúrese de que está optimizando el mapa de autocarga de clases de Composer para que Composer pueda encontrar rápidamente el archivo adecuado para cargar una clase determinada:

```sh
composer install --optimize-autoloader --no-dev
```

{% hint style="info" %}
Además de optimizar el autoloader, siempre debes asegurarte de incluir un archivo `composer.lock` en el repositorio de control de código fuente de tu proyecto. Las dependencias de su proyecto se pueden instalar mucho más rápido cuando un archivo `composer.lock` está presente.
{% endhint %}

### Optimizar la carga de configuraciones

Cuando despliegues tu aplicación a producción, debes asegurarte de ejecutar el comando Artisan `config:cache` durante tu proceso de despliegue:

```sh
php artisan config:cache
```

Este comando combinará todos los archivos de configuración de Laravel en un único archivo en caché, lo que reduce en gran medida el número de viajes que el framework debe hacer al sistema de archivos al cargar los valores de configuración.

{% hint style="info" %}
Si ejecutas el comando `config:cache` durante tu proceso de despliegue, debes asegurarte de que sólo estás llamando a la función `env` desde dentro de tus ficheros de configuración. Una vez que la configuración se ha almacenado en caché, el archivo `.env` no se cargará y todas las llamadas a la función `env` para variables `.env` devolverán `null`.
{% endhint %}

### Optimizar la carga de rutas

Si estás construyendo una aplicación grande con muchas rutas, debes asegurarte de que estás ejecutando el comando Artisan `route:cache` durante tu proceso de despliegue:

```sh
php artisan route:cache
```

Este comando reduce todos los registros de rutas en una única llamada a un método dentro de un archivo en caché, lo que mejora el rendimiento del registro de rutas cuando se registran cientos de rutas.

### Optimizar la carga de vistas

Cuando despliegues tu aplicación a producción, debes asegurarte de ejecutar el comando Artisan `view:cache` durante tu proceso de despliegue:

```sh
php artisan view:cache
```

Este comando precompila todas tus vistas Blade para que no se compilen bajo demanda, mejorando el rendimiento de cada petición que devuelve una vista.

## Modo Debug

La opción de depuración en su archivo de configuración config/app.php determina cuánta información sobre un error se muestra realmente al usuario. Por defecto, esta opción está configurada para respetar el valor de la variable de entorno `APP_DEBUG`, que se almacena en el archivo `.env` de su aplicación.

**En su entorno de producción, este valor debe ser siempre `false`. Si la variable `APP_DEBUG` se establece en `true` en producción, corres el riesgo de exponer valores de configuración sensibles a los usuarios finales de tu aplicación.**

## Despliegue con Forge / Vapor

### Laravel Forge

Si no estás preparado para gestionar la configuración de tu propio servidor o no te sientes cómodo configurando todos los servicios necesarios para ejecutar una aplicación Laravel robusta, [Laravel Forge](https://forge.laravel.com) es una magnífica alternativa.

Laravel Forge puede crear servidores en varios proveedores de infraestructura como DigitalOcean, Linode, AWS, y más. Además, Forge instala y gestiona todas las herramientas necesarias para construir aplicaciones Laravel robustas, como Nginx, MySQL, Redis, Memcached, Beanstalk y más.

{% hint style="info" %}
¿Quieres una guía completa para desplegar con Laravel Forge? Echa un vistazo a la [Laravel Bootcamp](https://bootcamp.laravel.com/deploying) y Forge [serie de vídeos disponibles en Laracasts](https://laracasts.com/series/learn-laravel-forge-2022-edition).
{% endhint %}

### Laravel Vapor

Si quieres una plataforma de despliegue totalmente sin servidor y autoescalable para Laravel, echa un vistazo a [Laravel Vapor](https://vapor.laravel.com). Laravel Vapor es una plataforma de despliegue sin servidor para Laravel, impulsada por AWS. Lanza tu infraestructura Laravel en Vapor y enamórate de la simplicidad escalable de serverless. Laravel Vapor está ajustado por los creadores de Laravel para trabajar sin problemas con el framework para que puedas seguir escribiendo tus aplicaciones Laravel exactamente como estás acostumbrado.
