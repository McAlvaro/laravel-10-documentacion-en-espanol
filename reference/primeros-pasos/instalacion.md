# Instalación

## Conoce Laravel

Laravel es un framework de aplicaciones web con una sintaxis expresiva y elegante. Un framework web proporciona una estructura y un punto de partida para la creación de su aplicación, lo que le permite centrarse en la creación de algo increíble mientras sudamos los detalles.

Laravel se esfuerza por proporcionar una experiencia de desarrollador increíble al tiempo que proporciona características de gran alcance, tales como la inyección de dependencia a fondo, una capa de abstracción de base de datos expresiva, colas y trabajos programados, pruebas unitarias y de integración, y mucho más.

Tanto si eres nuevo en los frameworks web PHP como si tienes años de experiencia, Laravel es un framework que puede crecer contigo. Te ayudaremos a dar tus primeros pasos como desarrollador web o te daremos un empujón mientras llevas tu experiencia al siguiente nivel. No podemos esperar a ver lo que construyes.

{% hint style="info" %}
**Nota** ¿Eres nuevo en Laravel? Echa un vistazo a la [Laravel Bootcamp](https://bootcamp.laravel.com) para un recorrido práctico del marco mientras te guiamos a través de la construcción de su primera aplicación Laravel.
{% endhint %}

## ¿Por qué Laravel?

Hay una gran variedad de herramientas y frameworks disponibles para construir una aplicación web. Sin embargo, creemos que Laravel es la mejor opción para crear aplicaciones web modernas y completas.

### Un Framework progresivo

Nos gusta llamar a Laravel un framework "progresivo". Con esto queremos decir que Laravel crece contigo. Si estás dando tus primeros pasos en el desarrollo web, la vasta biblioteca de Laravel de documentación, guías y [video tutoriales](https://laracasts.com) te ayudará a aprender las cuerdas sin sentirte abrumado.

Si eres un desarrollador senior, Laravel te ofrece herramientas robustas para [inyección de dependencias](https://laravel.com/docs/10.x/container), [pruebas unitarias](https://laravel.com/docs/10.x/testing), [colas](https://laravel.com/docs/10.x/queues), [eventos en tiempo real](https://laravel.com/docs/10.x/broadcasting), y mucho más. Laravel está afinado para la construcción de aplicaciones web profesionales y listo para manejar las cargas de trabajo de la empresa.

### Un framework escalable

Laravel es increíblemente escalable. Gracias a la naturaleza escalable de PHP y al soporte integrado de Laravel para sistemas de caché rápidos y distribuidos como Redis, el escalado horizontal con Laravel es pan comido. De hecho, las aplicaciones Laravel se han escalado fácilmente para manejar cientos de millones de peticiones al mes.

¿Necesitas escalado extremo? Plataformas como [Laravel Vapor](https://vapor.laravel.com) te permiten ejecutar tu aplicación Laravel a una escala casi ilimitada en la última tecnología sin servidor de AWS.

### Un framework comunitario

Laravel combina los mejores paquetes del ecosistema PHP para ofrecer el framework más robusto y fácil de desarrollar que existe. Además, miles de desarrolladores con talento de todo el mundo han [contribuido al framework](https://github.com/laravel/framework). Quién sabe, quizás incluso te conviertas en un contribuidor de Laravel.

## Tu Primer Proyecto Laravel

Antes de crear tu primer proyecto Laravel, debes asegurarte de que tu máquina local tiene PHP y [Composer](https://getcomposer.org) instalados. Si estás desarrollando en macOS, PHP y Composer se pueden instalar a través de [Homebrew](https://brew.sh/). Además, recomendamos [instalar Node y NPM](https://nodejs.org).

Después de haber instalado PHP y Composer, puede crear un nuevo proyecto Laravel a través del comando Composer `create-project`:

```shell
composer create-project laravel/laravel example-app
```

O bien, puede crear nuevos proyectos Laravel mediante la instalación global del instalador de Laravel a través de Composer:

```shell
composer global require laravel/installer
 
laravel new example-app
```

Una vez creado el proyecto, inicia el servidor de desarrollo local de Laravel utilizando el comando `serve` de la CLI Artisan de Laravel:

```shell
cd example-app
 
php artisan serve
```

Una vez que hayas iniciado el servidor de desarrollo Artisan, tu aplicación será accesible en tu navegador web en `http://localhost:8000`. A continuación, estás listo para comenzar a dar los [siguientes pasos en el ecosistema Laravel](https://laravel.com/docs/10.x/installation#next-steps). Por supuesto, también puedes querer [configurar una base de datos](https://laravel.com/docs/10.x/installation#databases-and-migrations).

{% hint style="info" %}
Si desea una ventaja al desarrollar su aplicación Laravel, considere el uso de uno de nuestros kits de inicio. Los [kits de inicio](https://laravel.com/docs/10.x/starter-kits) de Laravel proporcionan un andamiaje de autenticación backend y frontend para tu nueva aplicación Laravel.
{% endhint %}

## Laravel y Docker

Queremos que sea lo más fácil posible empezar con Laravel independientemente de tu sistema operativo preferido. Por lo tanto, hay una variedad de opciones para desarrollar y ejecutar un proyecto Laravel en tu máquina local. Si bien es posible que desees explorar estas opciones más adelante, Laravel proporciona [Sail](https://laravel.com/docs/10.x/sail), una solución integrada para ejecutar tu proyecto Laravel utilizando [Docker](https://www.docker.com).

Docker es una herramienta para ejecutar aplicaciones y servicios en "contenedores" pequeños y ligeros que no interfieren con el software instalado o la configuración de tu máquina local. Esto significa que no tienes que preocuparte de configurar o instalar complicadas herramientas de desarrollo como servidores web y bases de datos en tu máquina local. Para empezar, sólo tienes que instalar [Docker Desktop](https://www.docker.com/products/docker-desktop).

Laravel Sail es una interfaz de línea de comandos ligera para interactuar con la configuración Docker por defecto de Laravel. Sail proporciona un gran punto de partida para la construcción de una aplicación Laravel usando PHP, MySQL y Redis sin necesidad de experiencia previa en Docker.

{% hint style="info" %}
¿Ya es un experto en Docker? No te preocupes. Todo sobre Sail se puede personalizar utilizando el archivo `docker-compose.yml` incluido con Laravel.
{% endhint %}

