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

## Primeros pasos en macOS

Si estás desarrollando en un Mac y [Docker Desktop](https://www.docker.com/products/docker-desktop) ya está instalado, puedes utilizar un simple comando de terminal para crear un nuevo proyecto Laravel. Por ejemplo, para crear una nueva aplicación Laravel en un directorio llamado "example-app", puedes ejecutar el siguiente comando en tu terminal:

```shell
curl -s "https://laravel.build/example-app" | bash
```

Por supuesto, puedes cambiar "example-app" en esta URL por lo que quieras sólo asegúrate de que el nombre de la aplicación sólo contiene caracteres alfanuméricos, guiones y guiones bajos. El directorio de la aplicación Laravel se creará dentro del directorio desde el que ejecutes el comando.

La instalación de Sail puede tardar varios minutos mientras se crean los contenedores de aplicaciones de Sail en su máquina local.

Una vez creado el proyecto, puede navegar hasta el directorio de la aplicación e iniciar Laravel Sail. Laravel Sail proporciona una sencilla interfaz de línea de comandos para interactuar con la configuración Docker por defecto de Laravel:

```shell
cd example-app
 
./vendor/bin/sail up
```

Una vez iniciados los contenedores Docker de la aplicación, puedes acceder a la aplicación en tu navegador web en: [http://localhost](http://localhost).

{% hint style="info" %}
Para seguir aprendiendo más sobre Laravel Sail, revisa su [documentación completa](https://laravel.com/docs/10.x/sail).
{% endhint %}

## Primeros pasos en Windows

Antes de crear una nueva aplicación Laravel en tu máquina Windows, asegúrate de instalar [Docker Desktop](https://www.docker.com/products/docker-desktop). A continuación, debes asegurarte de que Windows Subsystem for Linux 2 (WSL2) está instalado y habilitado. WSL le permite ejecutar ejecutables binarios de Linux de forma nativa en Windows 10. Encontrará información sobre cómo instalar y habilitar WSL2 en la [documentación del entorno de desarrollo de Microsoft](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

Después de instalar y habilitar WSL2, debe asegurarse de que Docker Desktop está [configurado para utilizar el backend WSL2](https://docs.docker.com/docker-for-windows/wsl/).

A continuación, estás listo para crear tu primer proyecto Laravel. Inicia [Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?rtc=1\&activetab=pivot:overviewtab) y comienza una nueva sesión de terminal para tu sistema operativo Linux WSL2. A continuación, puedes utilizar un simple comando de terminal para crear un nuevo proyecto Laravel. Por ejemplo, para crear una nueva aplicación Laravel en un directorio llamado "example-app", puedes ejecutar el siguiente comando en tu terminal:

```shell
curl -s https://laravel.build/example-app | bash
```

Por supuesto, puedes cambiar "example-app" en esta URL por lo que quieras sólo asegúrate de que el nombre de la aplicación sólo contiene caracteres alfanuméricos, guiones y guiones bajos. El directorio de la aplicación Laravel se creará dentro del directorio desde el que ejecutes el comando.

La instalación de Sail puede tardar varios minutos mientras se crean los contenedores de aplicaciones de Sail en su máquina local.

Una vez creado el proyecto, puede navegar hasta el directorio de la aplicación e iniciar Laravel Sail. Laravel Sail proporciona una sencilla interfaz de línea de comandos para interactuar con la configuración Docker por defecto de Laravel:

```shell
cd example-app
 
./vendor/bin/sail up
```

Una vez iniciados los contenedores Docker de la aplicación, puedes acceder a la aplicación en tu navegador web en: [http://localhost](http://localhost).

{% hint style="info" %}
Para seguir aprendiendo más sobre Laravel Sail, revisa su [documentación completa.](https://laravel.com/docs/10.x/sail)
{% endhint %}

### Desarrollo dentro de WSL2

Por supuesto, necesitarás poder modificar los archivos de la aplicación Laravel que fueron creados dentro de tu instalación WSL2. Para ello, te recomendamos que utilices el editor [Visual Studio Code](https://code.visualstudio.com) de Microsoft y su extensión para [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack).

Una vez instaladas estas herramientas, puede abrir cualquier proyecto Laravel ejecutando el comando `code .` desde el directorio raíz de su aplicación utilizando el Terminal de Windows.

## Primeros pasos en Linux

Si estás desarrollando en Linux y [Docker Compose](https://docs.docker.com/compose/install/) ya está instalado, puedes utilizar un simple comando de terminal para crear un nuevo proyecto Laravel. Por ejemplo, para crear una nueva aplicación Laravel en un directorio llamado "example-app", puedes ejecutar el siguiente comando en tu terminal:

```shell
curl -s https://laravel.build/example-app | bash
```

Por supuesto, puedes cambiar "example-app" en esta URL por lo que quieras sólo asegúrate de que el nombre de la aplicación sólo contiene caracteres alfanuméricos, guiones y guiones bajos. El directorio de la aplicación Laravel se creará dentro del directorio desde el que ejecutes el comando.

La instalación de Sail puede tardar varios minutos mientras se crean los contenedores de aplicación de Sail en su máquina local.

Una vez creado el proyecto, puede navegar hasta el directorio de la aplicación e iniciar Laravel Sail. Laravel Sail proporciona una sencilla interfaz de línea de comandos para interactuar con la configuración Docker por defecto de Laravel:

```shell
cd example-app
 
./vendor/bin/sail up
```

Una vez iniciados los contenedores Docker de la aplicación, puedes acceder a la aplicación en tu navegador web en: [http://localhost](http://localhost).

{% hint style="info" %}
Para seguir aprendiendo más sobre Laravel Sail, revisa su [documentación completa.](https://laravel.com/docs/10.x/sail)
{% endhint %}

## Elegir sus servicios Sail

Al crear una nueva aplicación Laravel a través de Sail, puedes utilizar la variable `with` query string para elegir qué servicios deben configurarse en el archivo `docker-compose.yml` de tu nueva aplicación. Los servicios disponibles son `mysql`, `pgsql`, `mariadb`, `redis`, `memcached`, `meilisearch`, `minio`, `selenium` y `mailpit`:

```shell
curl -s "https://laravel.build/example-app?with=mysql,redis" | bash
```

Si no especifica qué servicios desea configurar, se configurará una pila por defecto de `mysql`, `redis`, `meilisearch`, `mailpit` y `selenium`.

Puede indicar a Sail que instale un [Devcontainer](https://laravel.com/docs/10.x/sail#using-devcontainers) por defecto añadiendo el parámetro `devcontainer` a la URL:

```shell
curl -s "https://laravel.build/example-app?with=mysql,redis&devcontainer" | bash
```

## Configuración inicial

Todos los archivos de configuración para el framework Laravel se almacenan en el directorio `config`. Cada opción está documentada, así que no dudes en echar un vistazo a los archivos y familiarizarte con las opciones disponibles.

Laravel casi no necesita configuración adicional fuera de la caja. ¡Eres libre de empezar a desarrollar! Sin embargo, es posible que desees revisar el archivo `config/app.php` y su documentación. Contiene varias opciones como `timezone` y `locale` que puede que desees cambiar de acuerdo a tu aplicación.

### Configuración basada en el entorno

Dado que muchos de los valores de las opciones de configuración de Laravel pueden variar dependiendo de si tu aplicación se está ejecutando en tu máquina local o en un servidor web de producción, muchos valores de configuración importantes se definen utilizando el archivo `.env` que existe en la raíz de tu aplicación.

Su archivo `.env` no debe ser enviado al control de código fuente de su aplicación, ya que cada desarrollador / servidor que utilice su aplicación podría requerir una configuración de entorno diferente. Además, esto supondría un riesgo de seguridad en caso de que un intruso accediera a tu repositorio de control de código fuente, ya que cualquier credencial sensible quedaría expuesta.

{% hint style="info" %}
Para obtener más información sobre el archivo `.env` y la configuración basada en entornos, consulte la [documentación de configuración completa](https://laravel.com/docs/10.x/configuration#environment-configuration).
{% endhint %}

### Bases de datos y migraciones

Ahora que has creado tu aplicación Laravel, probablemente quieras almacenar algunos datos en una base de datos. Por defecto, el fichero de configuración `.env` de tu aplicación especifica que Laravel interactuará con una base de datos MySQL y accederá a la base de datos en `127.0.0.1`. Si estás desarrollando en macOS y necesitas instalar MySQL, Postgres o Redis localmente, puede que te resulte conveniente utilizar [DBngin](https://dbngin.com/).

Si no quieres instalar MySQL o Postgres en tu máquina local, siempre puedes utilizar una base de datos [SQLite](https://www.sqlite.org/index.html). SQLite es un motor de base de datos pequeño, rápido y autónomo. Para empezar, crea una base de datos SQLite creando un archivo SQLite vacío. Típicamente, este archivo existirá dentro del directorio `database` de tu aplicación Laravel:

```shell
touch database/database.sqlite
```

A continuación, actualiza tu fichero de configuración `.env` para utilizar el controlador de base de datos `sqlite` de Laravel. Puedes eliminar las otras opciones de configuración de la base de datos:

```
DB_CONNECTION=sqlite
 
DB_CONNECTION=mysql 
DB_HOST=127.0.0.1 
DB_PORT=3306 
DB_DATABASE=laravel 
DB_USERNAME=root 
DB_PASSWORD= 
```

Una vez que haya configurado su base de datos SQLite, puede ejecutar las [migraciones de base de datos](https://laravel.com/docs/10.x/migrations) de su aplicación, que creará las tablas de la base de datos de su aplicación:

```shell
php artisan migrate
```

## Próximos pasos

Ahora que has creado tu proyecto Laravel, puede que te estés preguntando qué aprender a continuación. En primer lugar, te recomendamos que te familiarices con el funcionamiento de Laravel leyendo la siguiente documentación:

* [Ciclo de vida de las peticiones](https://laravel.com/docs/10.x/lifecycle)
* [Configuración](https://laravel.com/docs/10.x/configuration)
* [Estructura del directorios](https://laravel.com/docs/10.x/structure)
* [Frontend](https://laravel.com/docs/10.x/installation#initial-configuration)
* [Contenedor de Servicio](https://laravel.com/docs/10.x/container)
* [Facades](https://laravel.com/docs/10.x/facades)

Cómo desea utilizar Laravel también dictará los próximos pasos en su viaje. Hay una variedad de maneras de utilizar Laravel, y vamos a explorar dos casos de uso principales para el marco de abajo.

{% hint style="info" %}
¿Es nuevo en Laravel? Echa un vistazo a la [Laravel Bootcamp](https://bootcamp.laravel.com/) para un recorrido práctico del marco mientras te guiamos a través de la construcción de su primera aplicación Laravel.
{% endhint %}

## Laravel  Framework Full Stack&#x20;

Laravel puede servir como un framework full stack. Por framework "full stack" queremos decir que vas a usar Laravel para enrutar peticiones a tu aplicación y renderizar tu frontend vía [Blade templates](https://laravel.com/docs/10.x/blade) o una tecnología híbrida de aplicación de una sola página como [Inertia](https://inertiajs.com). Esta es la forma más común de utilizar el framework Laravel y, en nuestra opinión, la forma más productiva de utilizar Laravel.

Si esta es la forma en la que planeas usar Laravel, puede que quieras consultar nuestra documentación sobre [desarrollo frontend](https://laravel.com/docs/10.x/frontend), [routing](https://laravel.com/docs/10.x/routing), [views](https://laravel.com/docs/10.x/views), o el [Eloquent ORM](https://laravel.com/docs/10.x/eloquent). Además, puede que te interese conocer paquetes de la comunidad como [Livewire](https://laravel-livewire.com) e [Inertia](https://inertiajs.com). Estos paquetes te permiten usar Laravel como un framework completo mientras disfrutas de muchos de los beneficios de la interfaz de usuario que proporcionan las aplicaciones JavaScript de una sola página.

{% hint style="info" %}
Si quiere empezar a crear su aplicación, eche un vistazo a uno de nuestros [kits de inicio de aplicaciones](https://laravel.com/docs/10.x/starter-kits) oficiales.
{% endhint %}

## Laravel API Backend

Laravel también puede servir como API backend para una aplicación JavaScript de una sola página o una aplicación móvil. Por ejemplo, puedes usar Laravel como API backend para tu aplicación [Next.js](https://nextjs.org). En este contexto, puede utilizar Laravel para proporcionar [autenticación](https://laravel.com/docs/10.x/sanctum) y almacenamiento / recuperación de datos para su aplicación, al mismo tiempo que aprovecha los potentes servicios de Laravel como colas, correos electrónicos, notificaciones y más.

Si esta es la forma en la que planeas usar Laravel, puede que quieras consultar nuestra documentación sobre [routing](https://laravel.com/docs/10.x/routing), [Laravel Sanctum](https://laravel.com/docs/10.x/sanctum), y el [Eloquent ORM](https://laravel.com/docs/10.x/eloquent).

{% hint style="info" %}
¿Necesitas un scaffolding inicial para tu backend Laravel y tu frontend Next.js? Laravel Breeze ofrece una [API stack](https://laravel.com/docs/10.x/starter-kits#breeze-and-next) así como una [Next.js frontend implementation](https://github.com/laravel/breeze-next) para que puedas empezar en cuestión de minutos.
{% endhint %}

