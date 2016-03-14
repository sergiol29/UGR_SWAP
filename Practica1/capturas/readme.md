# Práctica 1 - Preparación de las herramientas
> **Autor: Sergio López Ballesteros**

Tras la instalación del software de virtualización VMware, instalamos dos máquinas virtuales Linux con Ubuntu Server 12.04.1 LTS.

Activamos la cuenta root del sistema cambiando el password del mismo por un password propio, mediante el comando:
```sh
$ sudo passwd root
```

Si durante la instalación del sistema Ubuntu no hemos instalado los servicios de ***Lamp (Apache, PHP, MySQL)*** y ***OpenSSH***, realizamos la instalación con los siguientes comandos respectivamente.
```sh
$ sudo apt install lamp-server^
```

```sh
$ sudo apt install openssh-server
```

Tras la instalación del servicio ***Lamp***, comprobamos si dicho el servicio ***Apache*** está funcionando correctamente, para ello accedemos desde el navegador de la máquina anfitriona e indicamos la IP del servidor Ubuntu y nos debe aparecer un mensaje de bienvenida como la siguiente imagen el cual nos indica que ***Apache*** está funcionando correctamente.

![Servicio Apache en funcionamiento][captura1]



### Version Apache
Para comprobar la versión de ***Apache*** que hemos instalado y tenemos funcionando en nuestro sistema, ejecutamos el siguiente comando y nos aparecerá información como la siguiente imagen.

```sh
$ apache2 -v
```

![Version Servidor Apache][captura2]

### Apache en ejecución
Para comprobar si el ***servicio Apache*** esta en ejecución ejecutamos el siguiente comando y nos aparecerá un listado con los procesos del mismo que se encuentra en ejecución, como vemos en la siguiente imagen.

```sh
$ ps aux | grep apache
```

![Servicio Apache en ejecucion][captura3]

   [captura1]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica1/capturas/Ejercicio1_ApacheRun.PNG "Servicio Apache en funcionamiento"
   [captura2]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica1/capturas/Ejercicio1_VersionApache.PNG "Versión servidor Apache"
   [captura3]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica1/capturas/Ejercicio1_ServicioApache.PNG "Servicio Apache en ejecucion"

