# Práctica 2 - Clonar la información de un sitio web
> **Autor: Sergio López Ballesteros**

### Copia de archivos por ssh
Para realizar un backup de un archivo (comprimido o sin comprimir) y alojarlo en otro equipo por seguridad o simplemente falta de espacio, podemos **crear el backup del fichero y mediante ssh** crear dicho backup directamente en el equipo de destino.

En este caso, vamos a crear un **fichero comprimido (con el comando tar)** para realizar el backup de un directorio completo y mandar con una pipe el fichero backup hacia el equipo de destino mediante la conexión ssh entre equipos, con el comando:

```sh
$ tar czf - directorio | ssh equipodestino 'cat > ~/tar.tgz'
```

Como podemos observar en las siguientes imágenes desde un **servidor principal (UbuntuServer)** realizamos un backup del directorio ***"/var/www/html/"*** y dicho fichero comprimido se "crea" en un servidor secundario llamado ***Swap2***:

![Backup con TAR y copia con ssh][captura1]

Tras la ***compresión del directorio y copia por ssh del mismo***, el fichero comprimido ha sido copiado correctamente en el servidor secundario.

![Listado directorio servidor secundario][captura2]

### Clonado de directorios entre máquinas
Con la **herramienta rsync podemos realizar la clonación / copia de directorios entre máquinas**, con el fin de disponer del mismo contenido de una carpeta concreta en máquinas diferentes *(por ejemplo, dos servidores de una granja web que deben tener sus carpetas html con el contenido clonado para no perder datos importantes en caso de caída de uno de ellos, ya que es donde alojan las web a las que ofrecen servicio)*.

Esto se consigue teniendo instalada la **herramienta rsync** y realizando el siguiente comando en el servidor al cual queremos clonarle el contenido del servidor principal, es decir, disponemos de 2 servidores web y queremos que el ***servidor secundario (Swap1)*** tenga el mismo contenido en su carpeta html que el ***servidor principal (UbuntuServer)*** en su carpeta html respectivamente.

Para ello, ***aplicamos el siguiente comando en el servidor secundario (Swap2)***:

```sh
$ rsync -avz -e ssh root@maquina1:/var/www/ /var/www/
```

Como podemos observar en las siguientes capturas, ***disponemos de un contenido en la carpeta html del servidor principal (UbuntuServer)***.

![Listado directorio HTML de apache][captura3]

***Queremos clonar en el servidor secundario (Swap1) en el directorio html, el directorio html del servidor principal (UbuntuServer)***, ya que como podemos observar no disponen del mismo contenido.

Para ello, **aplicamos el comando rysnc explicado anteriormente en el servidor secundario (Swap1)** para clonar el contenido de la carpeta html del servidor principal (UbuntuServer) en la carpeta html del servidor secundario y vemos tras el último *"ls"* que ambas carpetas estan clonadas.   

![Clonado de directorios con herramienta rysnc][captura4]

### Configuración de ssh para acceder sin que solicite contraseña
En muchas de las **ocasiones que realizamos una conexión ssh** entre máquinas necesitamos que dicha **conexión se realice sin la necesidad de tener que indicar la contraseña cuando se realiza la conexión entre máquinas**.

Para ello, debemos **crear una autenticación** a través de clave *(ssh-keygen)* pública-privada entre las máquinas que se van a conectar por ssh.

***La clave ssh debe ser creada en la máquina secundaria y será copiada a la máquina principal***, ya que la conexión ssh se realizará desde el servidor secundario (Swap1) al servidor principal (UbuntuServer).

Por tanto, los pasos a realizar son los siguientes:

1º - ***Generamos la clave ssh en el servidor secundario (Swap1)***, mediante el siguiente comando y dejando el campo de contraseña vacio para que la conexión ssh se realice sin contraseña, como podemos observar en la siguiente imagen.

```sh
$ ssh-keygen -t dsa
```

![Creación clave ssh][captura5]

2º - Observamos que **la clave ssh ha sido creada en el directorio que nos encontramos de forma oculta**, como podemos observar en la siguiente imagen.

![Clave ssh creada en directorio][captura6]

3º - **Copiamos la clave ssh creada en el servidor secundario (Swap1) al servidor principal (UbuntuServer)** mediante el siguiente comando, como podemos observar en la siguiente imagen. 

```sh
$ ssh-copy-id -i .ssh/id_dsa.pub root@maquina1
```

![Copia clave ssh entre equipos][captura7]

4º - **Comprobamos que la conexión ssh se realiza sin necesidad de indicar contraseña para la conexión**, como podemos observar en la siguiente imagen:

![Conexión ssh sin contraseña gracias a clave ssh][captura8]

### Tarea en cron ejecutada cada hora para mantener actualizado el contenido del directorio /var/www/html entre máquinas
Para la **automatización de tareas**, como por ejemplo, la *clonación del contenido del directorio /var/www/html en un servidor secundario (Swap1) respecto al directorio /var/www/html del servidor principal (UbuntuServer)* se debe hacer a través del **administrador de procesos cron**, el cual **ejecuta procesos de manera automatizada, especificados en el fichero crontab en un instante indicado**.

Para la **clonación de los directorios *"/var/www/html"* de manera automatizada y cada hora**, vamos a realizar los siguientes pasos:

1º - Creamos un **script que se encargue de clonar dicho directorio del servidor principal (UbuntuServer) hacia el servidor secundario (Swap1) con la herramienta rsync**, como podemos observar en la siguiente imagen.

![Creación script Rsync][captura9]

2º - **Asignamos permisos de ejecución al script creado**, como podemos observar en la siguiente imagen.

![Permisos script Rsync][captura10]

3º - **Añadimos una nueva tarea al administrador de procesos cron para que ejecute dicho script creado cada hora y de forma automática**, *editando el fichero "/etc/crontab"*, e insertado la siguiente instrucción (que se ejecuta todas las horas del día al minuto 50), como podemos observar en la siguiente imagen:

```sh
50 *   * * *    /directorio/script_a_ejecutar
```

![Tarea en crontab][captura11]

4º - **Una vez cumplido el minuto 50 de cada hora, el script se ejecutara automáticamente** *(sin necesidad de indicar contraseña para la conexión ssh por la clave ssh creada anteriormente)*, clonándose el contenido de ambos directorios como podemos observar en las siguientes imágenes que el servidor secundario (Swap1) a las 18:50 horas dispone del contenido clonado respecto al servidor principal (UbuntuServer).

![Contenido carpeta HTML en UbuntuServer][captura12]

![Contenido carpeta HTML en Swap1][captura13]

   [captura1]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica2/capturas/Ejercicio1_Tar_ssh.PNG "Backup con TAR y copia con ssh"
   [captura2]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica2/capturas/Ejercicio1_Tar_ssh_1.PNG "Listado directorio servidor secundario"
   
   [captura3]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica2/capturas/Ejercicio2_Rsync.PNG "Listado directorio HTML de apache"
   [captura4]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica2/capturas/Ejercicio2_Rsync_1.PNG "Clonado de directorios con herramienta rysnc"
   
   [captura5]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica2/capturas/Ejercicio3_ssh.PNG "Creación clave ssh"
   [captura6]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica2/capturas/Ejercicio3_ssh_1.PNG "Clave ssh creada en directorio"
   [captura7]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica2/capturas/Ejercicio3_ssh_2.PNG "Copia clave ssh entre equipos"
   [captura8]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica2/capturas/Ejercicio3_ssh_3.PNG "Conexión ssh sin contraseña gracias a clave ssh"
   
   [captura9]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica2/capturas/Ejercicio4_Crontab.PNG "Creación script Rsync"
   [captura10]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica2/capturas/Ejercicio4_Crontab_0.PNG "Permisos script Rsync"
   [captura11]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica2/capturas/Ejercicio4_Crontab_1.PNG "Tarea en crontab"
   [captura12]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica2/capturas/Ejercicio4_Crontab_2.PNG "Contenido carpeta HTML en UbuntuServer"
   [captura13]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica2/capturas/Ejercicio4_Crontab_3.PNG "Contenido carpeta HTML en Swap1"



