# Ciclo de Vida de una Solicitud

## Introducción

Cuando utilizas cualquier herramienta en el "mundo real", te sientes más seguro si entiendes cómo funciona esa herramienta. El desarrollo de aplicaciones no es diferente. Cuando entiendes cómo funcionan tus herramientas de desarrollo, te sientes más cómodo y seguro utilizándolas.

El objetivo de este documento es darte una buena visión general de alto nivel de cómo funciona el framework Laravel. Al conocer mejor el framework en general, todo parecerá menos "mágico" y te sentirás más seguro construyendo tus aplicaciones. Si no entiendes todos los términos a la primera, ¡no te desanimes! Simplemente intenta tener una idea básica de lo que está pasando, y tu conocimiento crecerá a medida que explores otras secciones de la documentación.

## Resumen del Ciclo de Vida

### Primeros pasos

El punto de entrada para todas las peticiones a una aplicación Laravel es el archivo `public/index.php`. Todas las peticiones son dirigidas a este archivo por la configuración de tu servidor web (Apache / Nginx). El archivo `index.php` no contiene mucho código. Más bien, es un punto de partida para cargar el resto del framework.

El archivo `index.php` carga la definición de autoloader generada por Composer, y luego recupera una instancia de la aplicación Laravel desde `bootstrap/app.php`. La primera acción que realiza Laravel es crear una instancia de la aplicación / [contenedor de servicios](https://laravel.com/docs/10.x/container).

### HTTP / Console Kernels

A continuación, la solicitud entrante se envía al kernel HTTP o al kernel de la consola, dependiendo del tipo de solicitud que esté entrando en la aplicación. Estos dos núcleos sirven como ubicación central a través de la cual fluyen todas las peticiones. Por ahora, vamos a centrarnos en el kernel HTTP, que se encuentra en `app/Http/Kernel.php`.

El kernel HTTP extiende la clase `Illuminate\Foundation\Http\Kernel`, que define un array de `bootstrappers` que se ejecutarán antes de que se ejecute la petición. Estos bootstrappers configuran el manejo de errores, configuran el registro, [detectan el entorno de la aplicación](https://laravel.com/docs/10.x/configuration#environment-configuration), y realizan otras tareas que deben hacerse antes de que la solicitud sea realmente manejada. Típicamente, estas clases manejan la configuración interna de Laravel de la que no necesitas preocuparte.

El núcleo HTTP también define una lista de [middleware](https://laravel.com/docs/10.x/middleware) HTTP por los que deben pasar todas las peticiones antes de ser gestionadas por la aplicación. Estos middleware manejan la lectura y escritura de la [sesión HTTP](https://laravel.com/docs/10.x/session), determinan si la aplicación está en modo mantenimiento, [verifican el token CSRF](https://laravel.com/docs/10.x/csrf), y más. Hablaremos más sobre esto pronto.

La firma del método `handle` del núcleo HTTP es bastante simple: recibe una `Request` y devuelve una `Response`. Piensa en el kernel como si fuera una gran caja negra que representa toda tu aplicación. Dale peticiones HTTP y te devolverá respuestas HTTP.

### Proveedores de Servicios

Una de las acciones más importantes para arrancar el kernel es cargar los [service providers](https://laravel.com/docs/10.x/providers) para tu aplicación. Los proveedores de servicios son responsables de arrancar todos los componentes del framework, como la base de datos, la cola, la validación y los componentes de enrutamiento. Todos los proveedores de servicios de la aplicación se configuran en la matriz `providers` del archivo de configuración `config/app.php`.

Laravel iterará a través de esta lista de proveedores e instanciará cada uno de ellos. Después de instanciar los proveedores, el método `register` será llamado en todos los proveedores. Entonces, una vez que todos los proveedores han sido registrados, el método `boot` será llamado en cada proveedor. Esto es para que los proveedores de servicios puedan depender de que cada contenedor esté registrado y disponible en el momento en que se ejecute su método `boot`.

Esencialmente cada característica importante ofrecida por Laravel es arrancada y configurada por un proveedor de servicios. Dado que arrancan y configuran tantas características ofrecidas por el framework, los proveedores de servicios son el aspecto más importante de todo el proceso de arranque de Laravel.

### Enrutamiento

Uno de los proveedores de servicios más importantes de tu aplicación es el `App\Providers\RouteServiceProvider`. Este proveedor de servicios carga los archivos de rutas contenidos en el directorio `routes` de tu aplicación. Adelante, abre el código del `RouteServiceProvider` y echa un vistazo a cómo funciona.

Una vez que la aplicación ha sido arrancada y todos los proveedores de servicios han sido registrados, el `Request` será entregada al enrutador para su despacho. El enrutador enviará la solicitud a una ruta o controlador, así como ejecutará cualquier middleware específico de la ruta.

Los middleware proporcionan un mecanismo conveniente para filtrar o examinar las peticiones HTTP que entran en tu aplicación. Por ejemplo, Laravel incluye un middleware que verifica si el usuario de tu aplicación está autenticado. Si el usuario no está autenticado, el middleware redirigirá al usuario a la pantalla de login. Sin embargo, si el usuario está autenticado, el middleware permitirá que la solicitud continúe en la aplicación. Algunos middleware se asignan a todas las rutas dentro de la aplicación, como los definidos en la propiedad `$middleware` de tu kernel HTTP, mientras que otros sólo se asignan a rutas específicas o grupos de rutas. Puedes aprender más sobre middleware leyendo la [documentación completa sobre middleware](https://laravel.com/docs/10.x/middleware).

Si la solicitud pasa a través de todos los middleware asignados a la ruta coincidente, se ejecutará el método de la ruta o del controlador y la respuesta devuelta por el método de la ruta o del controlador se enviará de vuelta a través de la cadena de middleware de la ruta.

### Finalizando

Una vez que el método de la ruta o controlador devuelve una respuesta, la respuesta viajará de vuelta hacia el exterior a través del middleware de la ruta, dando a la aplicación la oportunidad de modificar o examinar la respuesta saliente.

Finalmente, una vez que la respuesta viaja de vuelta a través del middleware, el método `handle` del núcleo HTTP devuelve el objeto respuesta y el archivo `index.php` llama al método `send` sobre la respuesta devuelta. El método `send` envía el contenido de la respuesta al navegador web del usuario. ¡Hemos terminado nuestro viaje a través de todo el ciclo de vida de las peticiones de Laravel!

## Centrarse en los Proveedores de Servicios

Los proveedores de servicios son realmente la clave para arrancar una aplicación Laravel. Se crea la instancia de la aplicación, se registran los proveedores de servicios, y la solicitud se entrega a la aplicación bootstrapped. Así de sencillo.

Tener una comprensión firme de cómo una aplicación Laravel se construye y arranca a través de los proveedores de servicios es muy valioso. Los proveedores de servicio predeterminados de tu aplicación se almacenan en el directorio `app/Providers`.

Por defecto, el `AppServiceProvider` está bastante vacío. Este proveedor es un buen lugar para añadir el bootstrapping propio de tu aplicación y los enlaces del contenedor de servicios. Para aplicaciones grandes, es posible que desee crear varios proveedores de servicios, cada uno con más granular bootstrapping para servicios específicos utilizados por su aplicación.
