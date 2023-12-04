
<h1 align="center"> Examen DNS </h1>

## Antes de empezar
Lo primero es ir a git y crear un nuevo repositorio. Después de esto, creamos nuestro nuevo proyecto en el Code. Una vez listo esto, creamos también un nuevo proyecto en Git y lo asociamos al repositorio creado anteriormente.

### 1. Métodos para 'abrir' una consola/shell a un contenedor que se está ejecutando

    $ docker exec

Permite acceder a la sesión shell de un contenedor en ejecución y ejecutar comandos sin necesidad de iniciar una nueva instancia. Hay que tener en cuenta que este comando no es persistente, lo que significa que no se volverá a ejecutar si el contenedor se apaga o se reinicia.

    $ docker run

Permite iniciar un nuevo contenedor y acceder inmediatamente a su shell. Por defecto, este contenedor no se adjunta a tu sesión de shell actual, pero se puede adjuntarlo utilizando la opción -it.

    $ docker attach

Este comando es útil para supervisar y depurar las operaciones de un contenedor. Permite conectarte a un contenedor en ejecución y ver sus flujos estándar de entrada, salida y error en tiempo real.

    $ docker compose exec

Docker Compose te permite crear y ejecutar aplicaciones Docker multicontenedor. Se puede utilizar para definir los servicios que componen la aplicación en un archivo YAML, y luego utilizar ese archivo para iniciar y gestionar todos los contenedores juntos. Es adecuado para entornos de desarrollo y pruebas en los que se necesita poner en marcha rápidamente entornos complejos. Este comando inicia un nuevo proceso dentro del contenedor que ejecuta el comando especificado. Se puede utilizar para ejecutar cualquier comando dentro del contenedor, incluidos los shells interactivos como bash.

    $ docker compose run

Del mismo modo, si quieres iniciar un nuevo contenedor utilizando Docker Compose y obtener acceso inmediato a él, ejecuta el este comando.

### 2. Opciones que tiene que haber sido arrancado el contenedor anterior para poder interactuar con las entradas y salidas del contenedor

1. `-i` o `--interactive`: Esta opción permite mantener abierta la entrada estándar (stdin) del contenedor, lo que permite enviar comandos e interactuar con él.
2. `-t` o `--tty`: Esta opción asigna una pseudo-tty (terminal) al contenedor, lo que permite una interacción más amigable y legible con el mismo.
3. `--rm` o `--detach`: Estas opciones permiten ejecutar el contenedor en modo desprendido (detach mode), es decir, se ejecutará en segundo plano en lugar de estar en primer plano. Sin embargo, con la opción `--rm`, el contenedor se eliminará automáticamente cuando se detenga su ejecución.

Por ejemplo, para arrancar un contenedor interactivo y en modo terminal, se puede utilizar el siguiente comando:

```
docker run -it <nombre>
```

Donde `<nombre>` es el nombre de la imagen de Docker que se desea utilizar para crear el contenedor.

