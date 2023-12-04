
<h1 align="center"> Examen DNS </h1>

## Antes de empezar
Lo primero es ir a git y crear un nuevo repositorio. Después de esto, creamos nuestro nuevo proyecto en el Code. Una vez listo esto, creamos también un nuevo proyecto en Git y lo asociamos al repositorio creado anteriormente.

### `1.` Métodos para 'abrir' una consola/shell a un contenedor que se está ejecutando

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

### `2.` Opciones que tiene que haber sido arrancado el contenedor anterior para poder interactuar con las entradas y salidas del contenedor

1. `-i` o `--interactive`: Esta opción permite mantener abierta la entrada estándar (stdin) del contenedor, lo que permite enviar comandos e interactuar con él.
2. `-t` o `--tty`: Esta opción asigna una pseudo-tty (terminal) al contenedor, lo que permite una interacción más amigable y legible con el mismo.
3. `--rm` o `--detach`: Estas opciones permiten ejecutar el contenedor en modo desprendido (detach mode), es decir, se ejecutará en segundo plano en lugar de estar en primer plano. Sin embargo, con la opción `--rm`, el contenedor se eliminará automáticamente cuando se detenga su ejecución.

Por ejemplo, para arrancar un contenedor interactivo y en modo terminal, se puede utilizar el siguiente comando:

```
$ docker run -it <nombre>
```

Donde `<nombre>` es el nombre de la imagen de Docker que se desea utilizar para crear el contenedor.

### `3.` ¿Cómo sería un fichero docker-compose para que dos contenedores se comuniquen entre si en una red solo de ellos?

Este es un ejemplo de un archivo `docker-compose.yml` que crea dos contenedores en una red dedicada y les permite comunicarse entre sí:

```yaml
services:
  asir_bind9:
    container_name: asir_bind9
    image: internetsystemsconsortium/bind9:9.16
    platform: linux/amd64
    ports:
      - 53:53
    networks:
      bind9_subnet:
        ipv4_address: 172.28.5.1
    volumes:
      - ./conf:/etc/bind
      - ./zonas:/var/lib/bind
  asir_cliente:
    container_name: asir_cliente
    image: alpine
    networks:
      - bind9_subnet
    stdin_open: true 
    tty: true 
    dns:
      - 172.28.5.1 
networks:
  bind9_subnet:
    external: true
```

En este ejemplo, tienes dos servicios (`asir_bind9` y `asir_cliente`) que se construyen a partir de sus respectivos `image`. Ambos servicios están conectados a la misma red.

Cada contenedor puede comunicarse con el otro usando el nombre del servicio como hostname. Por ejemplo, dentro de `asir_bind9`, puedes acceder a `asir_cliente` usando la URL `http://asir_cliente:ip`, donde `dns` es el puerto asignado a `asir_cliente`.

### `4.` ¿Qué hay que añadir al fichero anterior para que un contenedor tenga la IP fija?

Para asignar una IP fija a un contenedor en Docker Compose, se debe añadir la siguiente configuración:

1. Se agrega una sección de `networks` en el archivo `docker-compose.yml` si aún no se tiene una:
```yaml
networks:
  my_network:
    ipam:
      driver: default
      config:
        - subnet: 172.28.5.0/24
```

2. Dentro de la definición del servicio al que se desa asignar una IP fija, se especifica la `networks` que acabas de crear y agrega una etiqueta `ipv4_address` con la IP deseada:
```yaml
version: '3'
services:
  my_service:
    image: my_image
    networks:
      my_network:
        ipv4_address: 172.28.5.1
```
En nuestro caso en el servicio bind9 añadiremos esto y en el cliente señalaremos solo `bind9_subnet` como ya está puesto en el ejemplo del ejercicio anterior.

3. Por último, hay que asegurarse de que otros servicios que necesiten comunicarse con el contenedor con IP fija también se conecten a la misma `networks`:
```yaml
version: '3'
services:
  otro_service:
    image: otra_image
    networks:
      my_network:
```

Hay que recordar que la IP que se especifique debe estar dentro del rango de la subred definida en `subnet` para evitar conflictos. En el ejemplo anterior, se utiliza `172.28.5.0/24`. 

### `5.` ¿Que comando de consola puedo usar para saber las ips de los contenedores anteriores? Filtra todo lo que puedas la salida.

Podemos utilizar el comando `docker ps -aq` para obtener una lista de los IDs de todos los contenedores anteriores. Luego, se puede filtrar la salida de la siguiente manera:

```
$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)
```

Este comando primero obtiene los IDs de los contenedores anteriores con `docker ps -aq` y luego utiliza `docker inspect` para obtener la dirección IP de cada contenedor. El flag `--format` permite especificar el formato de salida, y en este caso, utilizamos una plantilla para imprimir solo la dirección IP de cada contenedor.

Esto dará una salida donde cada línea corresponderá a la dirección IP de un contenedor anterior.

### `6.` ¿Cual es la funcionalidad del apartado "ports" en docker compose?

El apartado "ports" en Docker Compose se utiliza para mapear los puertos de los contenedores que se ejecutan a los puertos del host. 

Cada servicio definido en el archivo docker-compose.yml puede tener su propia sección "ports". Dentro de esta sección, se indica el puerto del host seguido de dos puntos ":" y luego el puerto del contenedor. Por ejemplo:

```
services:
  web:
    ...
    ports:
      - "8080:80"
```

En este caso, el puerto 8080 del host se mapearía al puerto 80 del contenedor. Esto permite acceder al servicio dentro del contenedor a través de la dirección IP del host y el puerto mapeado.

El uso de "ports" es útil cuando se desea acceder a los servicios dentro de los contenedores desde el host o desde otros contenedores en la misma red Docker. También es posible configurar la propiedad "ports" en función de la dirección IP específica del host que se utilizará para el mapeo.

### `7.` Definición del registro CNAME con un ejemplo

El registro CNAME, o alias de nombre canónico, es un tipo de registro en el sistema de nombres de dominio (DNS) que se utiliza para asociar un nombre de dominio con otro nombre de dominio. 

El CNAME se utiliza generalmente cuando se desea redirigir un nombre de dominio a otro dominio o subdominio. Por ejemplo, supongamos que tenemos el dominio "asir.com" y queremos redirigir el subdominio "www" a otro dominio llamado "otro_asir.com". En este caso, crearíamos un registro CNAME en el DNS de "asir.com" que dirija el subdominio "www" a "otro_asir.com".

El registro CNAME se vería así:
```
www.asir.com.   IN   CNAME   otro_asir.com.
```
En este ejemplo, "www.asir.com" es el nombre de dominio que queremos redirigir y "otro_asir.com" es el dominio al que queremos redirigirlo.

Cuando un usuario intenta acceder a "www.asir.com", el registro CNAME le indica que redirija la solicitud a "otro_asir.com". Esto permite que el subdominio "www" sea utilizado para acceder al contenido del dominio "otro_asir.com" sin tener que configurar un servidor web adicional para "www.asir.com".

### `8.` ¿Como puedo hacer para que la configuración de un contenedor DNS no se borre si creo otro contenedor?

1. Se crea un volumen en Docker que contenga los archivos de configuración del contenedor DNS. Por ejemplo:
   ```
   $ docker volume create dns-config
   ```

2. Se monta este volumen en el contenedor DNS al momento de crearlo. Por ejemplo:
   ```
   $ docker run -d --name contenedor-dns -v dns-config:/ruta/config contenedor-dns
   ```

   Aquí, `/ruta/config` es la ruta dentro del contenedor que almacena los archivos de configuración del DNS.

3. Al crear un nuevo contenedor, se utiliza el mismo volumen que contiene la configuración del contenedor DNS. Por ejemplo, si deseamos crear un contenedor web:
   ```
   $ docker run -d --name contenedor-web -v dns-config:/ruta/dns-config contenedor-web
   ```

   Aquí, `/ruta/dns-config` es la ruta dentro del nuevo contenedor que contendrá los archivos de configuración del DNS.

De esta manera, al utilizar el mismo volumen en ambos contenedores, la configuración del contenedor DNS estará preservada incluso al crear un nuevo contenedor.

### `9.` Añade una zona tiendadeelectronica.int en tu docker DNS

Para crear la zona tiendadeelectronica.int en el Docker DNS, se modificará el archivo de configuración del DNS en el servidor de la práctica del DNS:

1. Accedemos al servidor donde está ejecutándose el Docker DNS.
2. Abrimos el archivo de configuración del DNS. Esto puede variar dependiendo del servidor DNS que se esté utilizando. Por ejemplo, para BIND, el archivo de configuración suele ser `/etc/bind/named.conf`.
3. Dentro del archivo de configuración, buscamos la sección `zone` y añadimos la siguiente entrada:

```
zone "tiendadeelectronica.int" {
    type master;
    file "/etc/bind/zones/tiendadeelectronica.int.zone";
};
```

4. Creamos el archivo de zona `db.tiendadeelectronica.int` en la ubicación especificada anteriormente. Este archivo contendrá los registros de la zona. Añadimos los siguientes registros:

```
$TTL 1d
@    IN    SOA   ns1.tiendadeelectronica.int. admin.tiendadeelectronica.int. (
                       2021092101 ; Serial
                       3600       ; Refresh
                       1800       ; Retry
                       604800     ; Expire
                       86400      ; Minimum TTL
                       )
     IN    NS    ns1.tiendadeelectronica.int.

www  IN    A     172.16.0.1
owncloud  IN    CNAME www
     IN    TXT   "1234ASDF"
```

5. Guardamos los cambios en el archivo `db.tiendadeelectronica.int`.
6. Reiniciamos el servicio DNS para aplicar los cambios.

Ahora, podemos comprobar que todo funciona correctamente utilizando el comando `dig`. Primero nos aseguramos de haber instalado el programa `dig` en nuestro sistema. Luego, ejecutamos los siguientes comandos para obtener los registros de la zona `tiendadeelectronica.int`:

```
dig www.tiendadeelectronica.int
dig owncloud.tiendadeelectronica.int
dig TXT tiendadeelectronica.int
```

Si todo está configurado correctamente, deberíamos obtener respuestas con los valores correspondientes.

Para mostrar los logs del servicio DNS, podemos utilizar el comando específico para el servidor DNS que estemos utilizando. Por ejemplo, para BIND, el comando sería `tail -f /var/log/syslog` en sistemas Linux.

