# laravel-nginx-docker
Docker with Nginx for Laravel
___
Un flujo de trabajo Docker Compose bastante simplificado que configura una red LEMP de contenedores para el desarrollo local de Laravel. Puedes ver el artículo completo que inspiró este repo. [Enlace](https://dev.to/aschmelyun/the-beauty-of-docker-for-local-laravel-development-13c0)

Nginx, MySQL, PHP, Redis, Grafana, Composer, NPM, Artisan y Mailhog

## Uso

Para empezar, asegúrese de que tiene [Docker instalado](https://docs.docker.com/docker-for-mac/install/) en el sistema, y luego clone este repositorio.

Luego, navega en tu terminal hasta el directorio que has clonado, y agrega el proyecto Laravel en una carpeta. Edita el docker-compose.yml cambiando donde dice CARPETA_DEL_PROYECTO por el nombre que le colocaste al proyecto Laravel. 

Una vez agregado el proyecto Laravel dentro del directorio raiz del docker, arranca los contenedores para el servidor web ejecutando `docker-compose up -d --build app`.

**Nota**: El nombre de host de tu base de datos MySQL debe ser `mysql`, **no** `localhost`. El nombre de usuario y la base de datos deben ser `laravel` con una contraseña `secret`.

Trayendo la red Docker Compose con `app` en lugar de sólo usar `up`, se asegura de que sólo los contenedores de nuestro sitio se traen al principio, en lugar de todos los contenedores de comandos también. Los siguientes son construidos para nuestro servidor web, con sus puertos expuestos detallados:

- **nginx** - `:80`
- **mysql** - `:3306`
- **php** - `:9000`
- **redis** - `:6379`
- **mailhog** - `:8025` 

Se incluyen tres contenedores adicionales que manejan los comandos de Composer, NPM y Artisan *sin* tener estas plataformas instaladas en tu ordenador local. Utiliza los siguientes ejemplos de comandos desde la raíz de tu proyecto, modificándolos para adaptarlos a tu caso de uso particular.

- `docker-compose run --rm composer update`
- `docker-compose run --rm npm run dev`
- `docker-compose run --rm artisan migrate`

## Problemas de Permisos

Si se encuentra algún problema con los permisos del sistema de archivos mientras entra a su aplicación o ejecuta un comando de contenedor, intente completar uno de los siguientes pasos.

**Si estás usando tu servidor o entorno local como usuario root:**

- Baje cualquier contenedor con `docker-compose down`
- Reemplace cualquier instancia de `php.dockerfile` en el archivo docker-compose.yml con `php.root.dockerfile`
- Reconstruye los contenedores ejecutando `docker-compose build --no-cache`

**Si está utilizando su servidor o entorno local como un usuario que no es root:**

- Cierra los contenedores con `docker-compose down`
- En el terminal, ejecutar `export UID=$(id -u)` y luego `export GID=$(id -g)`
- Si ves algún error sobre variables de sólo lectura en el paso anterior, puedes ignorarlo y continuar.
- Reconstruye los contenedores ejecutando `docker-compose build --no-cache`.

A continuación, vuelva a ejecutar el comando que se estaba intentando antes, y vea si se soluciona.

## Almacenamiento MySQL persistente

Por defecto, cada vez que haga un Down de Docker Network, sus datos MySQL serán eliminados después de que los contenedores sean destruidos. Si desea tener datos persistentes que permanezcan después de bajar y volver a subir los contenedores, haga lo siguiente:

1. Crea una carpeta `mysql` en la raíz del proyecto si no existe, junto a las carpetas `docker` y `grafana`.
2. Bajo el servicio mysql en tu archivo `docker-compose.yml`, añade las siguientes líneas:

```
volumes:
  - ./mysql:/var/lib/mysql
```

## Compiling Assets

Esta configuración debe ser capaz de compilar assets con [laravel mix](https://laravel-mix.com/) y con [vite](https://vitejs.dev/). Para empezar, primero tiene que añadir ` --host 0.0.0.0` al final del comando dev correspondiente en `package.json`. Así, por ejemplo, con un proyecto Laravel utilizando Vite:

```json
"scripts": {
  "dev": "vite --host 0.0.0.0",
  "build": "vite build"
},
```

A continuación, ejecute los siguientes comandos para instalar sus dependencias e iniciar el servidor de desarrollo:

- `docker-compose run --rm npm install`
- `docker-compose run --rm --service-ports npm run dev`

Después, se debe ser capaz de utilizar las directivas de `@vite` para habilitar la recarga de módulos en caliente en su aplicación local Laravel.

## MailHog

A partir de la version 9 de Laravel, utiliza MailHog como la aplicación por defecto para probar el envío de correo electrónico y el trabajo SMTP general durante el desarrollo local. Usando la imagen Docker Hub proporcionada, conseguir una instancia configurada y lista es simple. El servicio se incluye en el archivo `docker-compose.yml`, y se inicia junto con el servidor web y los servicios de base de datos.

Para ver el panel de control y los correos electrónicos que llegan a través del sistema, visite [localhost:8025](http://localhost:8025) después de ejecutar `docker-compose up`.