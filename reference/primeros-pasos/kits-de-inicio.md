# Kits de Inicio

## Introducción

Para darte una ventaja en la construcción de tu nueva aplicación Laravel, estamos encantados de ofrecerte kits de autenticación y de inicio de aplicaciones. Estos kits componen automáticamente tu aplicación con las rutas, controladores y vistas que necesitas para registrar y autenticar a los usuarios de tu aplicación.

Aunque le invitamos a utilizar estos kits de inicio, no son obligatorios. Eres libre de crear tu propia aplicación desde cero simplemente instalando una copia nueva de Laravel. De cualquier manera, ¡sabemos que construirás algo genial!

## Laravel Breeze

[Laravel Breeze](https://github.com/laravel/breeze) es una implementación mínima y sencilla de todas las [características de autenticación](https://laravel.com/docs/10.x/authentication) de Laravel, incluyendo login, registro, restablecimiento de contraseña, verificación de correo electrónico y confirmación de contraseña. Además, Breeze incluye una sencilla página de "perfil" donde el usuario puede actualizar su nombre, dirección de correo electrónico y contraseña.

La capa de vista por defecto de Laravel Breeze se compone de simples [Blade templates](https://laravel.com/docs/10.x/blade) estilizadas con [Tailwind CSS](https://tailwindcss.com). O, Breeze puede andamiar su aplicación utilizando Vue o React y [Inertia](https://inertiajs.com).

Breeze proporciona un maravilloso punto de partida para comenzar una aplicación Laravel fresca y también es una gran opción para proyectos que planean llevar sus plantillas Blade al siguiente nivel con [Laravel Livewire](https://laravel-livewire.com).

![](https://laravel.com/img/docs/breeze-register.png)

**Laravel Bootcamp**

Si usted es nuevo en Laravel, no dude en saltar en el [Laravel Bootcamp](https://bootcamp.laravel.com). El Laravel Bootcamp te guiará a través de la construcción de tu primera aplicación Laravel usando Breeze. Es una gran manera de conseguir un recorrido por todo lo que Laravel y Breeze tienen que ofrecer.

### Instalación

En primer lugar, debes [crear una nueva aplicación Laravel](https://laravel.com/docs/10.x/installation), configurar tu base de datos y ejecutar tus [migraciones de base de datos](https://laravel.com/docs/10.x/migrations). Una vez que hayas creado una nueva aplicación Laravel, puedes instalar Laravel Breeze usando Composer:

```sh
composer require laravel/breeze --dev
```

Una vez que Breeze está instalado, puede armar su aplicación utilizando una de las "pilas" Breeze que se discuten en la documentación a continuación.

### Breeze & Blade

Después de que Composer haya instalado el paquete Laravel Breeze, puedes ejecutar el comando Artisan `breeze:install`. Este comando publica las vistas de autenticación, rutas, controladores y otros recursos en tu aplicación. Laravel Breeze publica todo su código en tu aplicación para que tengas control total y visibilidad sobre sus características e implementación.

La "pila" por defecto de Breeze es la pila Blade, que utiliza simples [plantillas Blade](https://laravel.com/docs/10.x/blade) para renderizar el frontend de su aplicación. La pila Blade puede instalarse invocando el comando `breeze:install` sin otros argumentos adicionales. Una vez instalado el andamiaje de Breeze, también debes compilar los activos del frontend de tu aplicación:

```sh
php artisan breeze:install
 
php artisan migrate
npm install
npm run dev
```

A continuación, puede navegar a su aplicación `/login` o `/register` URLs en su navegador web. Todas las rutas de Breeze se definen en el archivo `routes/auth.php`.

#### Modo Dark

Si desea que Breeze incluya soporte para el "modo oscuro" al crear el frontend de su aplicación, simplemente proporcione la directiva `--dark` al ejecutar el comando `breeze:install`:

```sh
php artisan breeze:install --dark
```

{% hint style="info" %}
Para obtener más información sobre la compilación de CSS y JavaScript de tu aplicación, consulta la [documentación de Vite de Laravel](https://laravel.com/docs/10.x/vite#running-vite).
{% endhint %}

### Breeze & React / Vue

Laravel Breeze también ofrece andamiaje React y Vue a través de una implementación frontend [Inertia](https://inertiajs.com). Inertia te permite crear aplicaciones React y Vue modernas y de una sola página utilizando enrutamiento y controladores clásicos del lado del servidor.

Inertia te permite disfrutar de la potencia frontend de React y Vue combinada con la increíble productividad backend de Laravel y la rapidísima compilación [Vite](https://vitejs.dev). Para usar un stack de Inertia, especifica `vue` o `react` como tu stack deseado cuando ejecutes el comando `breeze:install` de Artisan. Después de instalar el andamiaje de Breeze, también debes compilar los activos frontend de tu aplicación:

```sh
php artisan breeze:install vue
 
# Or...
 
php artisan breeze:install react
 
php artisan migrate
npm install
npm run dev
```

A continuación, puede navegar a su aplicación `/login` o `/register` URLs en su navegador web. Todas las rutas de Breeze se definen en el archivo `routes/auth.php`.

#### Renderizado del lado del servidor

Si desea que Breeze sea compatible con [Inertia SSR](https://inertiajs.com/server-side-rendering), puede proporcionar la opción `ssr` al invocar el comando `breeze:install`:

```sh
php artisan breeze:install vue --ssr
php artisan breeze:install react --ssr
```

### Breeze & Next.js / API

Laravel Breeze también puede crear una API de autenticación que está preparada para autenticar aplicaciones JavaScript modernas como las desarrolladas por [Next](https://nextjs.org), [Nuxt](https://nuxtjs.org), y otras. Para empezar, especifica la pila `api` como tu pila deseada cuando ejecutes el comando `breeze:install` de Artisan:

```sh
php artisan breeze:install api
 
php artisan migrate
```

Durante la instalación, Breeze añadirá una variable de entorno `FRONTEND_URL` al archivo `.env` de tu aplicación. Esta URL debe ser la URL de su aplicación JavaScript. Normalmente será `http://localhost:3000` durante el desarrollo local. Además, debes asegurarte de que tu `APP_URL` está configurada en `http://localhost:8000`, que es la URL por defecto utilizada por el comando `serve` de Artisan.

#### Implementación de referencia de Next.js

Finalmente, estás listo para emparejar este backend con el frontend de tu elección. La siguiente implementación de referencia del frontend Breeze está [disponible en GitHub](https://github.com/laravel/breeze-next). Este frontend está mantenido por Laravel y contiene la misma interfaz de usuario que los stacks tradicionales Blade e Inertia proporcionados por Breeze.

## Laravel Jetstream

Mientras que Laravel Breeze proporciona un punto de partida simple y mínimo para la construcción de una aplicación Laravel, Jetstream aumenta esa funcionalidad con características más robustas y pilas de tecnología front-end adicionales. **Para los recién llegados a Laravel, se recomienda aprender las cuerdas con Laravel Breeze antes de graduarse a Laravel Jetstream.**

Jetstream proporciona un andamiaje de aplicaciones bellamente diseñado para Laravel e incluye inicio de sesión, registro, verificación de correo electrónico, autenticación de dos factores, gestión de sesiones, soporte de API a través de Laravel Sanctum y gestión de equipos opcional. Jetstream está diseñado utilizando [Tailwind CSS](https://tailwindcss.com) y ofrece la posibilidad de elegir entre [Livewire](https://laravel-livewire.com) o [Inertia](https://inertiajs.com) para el frontend.

La documentación completa para instalar Laravel Jetstream se puede encontrar en la [documentación oficial de Jetstream](https://jetstream.laravel.com/3.x/introduction.html).
