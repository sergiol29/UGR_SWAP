# Práctica 5 - Replicación de bases de datos MySQL

> **Autor: Sergio López Ballesteros**

### 1. Introducción
 
Para disponer de **copias de seguridad de las bases de datos MySQL** de nuestro servidor en producción, se suele usar la opción **réplica maestro - esclavo**, de manera que nuestro **servidor en producción hace de maestro y otro servidor de backup hace de esclavo**.

**Disponer de  una réplica de nuestras bases de datos** en otro servidor esclavo, **añade fiabilidad** ante fallos del servidor en producción y posibles interrupciones. 

Con esto **conseguimos evitar cualquier escenario catastrófico** que nos podamos imaginar.
Y **evitar tener decenas de clientes y servicios parados sin posibilidad de recuperar sus datos** si no hemos preparado un buen plan de contingencias

### 2. Crear una BD, insertar y consultar datos
Para la configuración **réplica maestro - esclavo** entre dos servidores, es decir, **nuestro servidor en producción hace de maestro y otro servidor de backup hace de esclavo** **crearemos una BD MySQL de prueba** (si no disponemos de ninguna) **e insertaremos algunos datos** mediante la interfaz de línea de comandos, así tendremos datos con los cuales hacer las copias de seguridad.

Para ello, **debemos entrar en un terminal y ejecutar los siguientes comandos**:

```sh
# mysql -uroot -p
mysql> create database contactos;
mysql> use contactos;
mysql> show tables;
mysql> create table datos(nombre varchar(100),tlf int);
mysql> show tables;
mysql> insert into datos(nombre,tlf) values ("Sergio",900900900);
mysql> insert into datos(nombre,tlf) values ("Lopez",600600600);
mysql> select * from datos;
mysql> describe datos;
mysql> quit;
```

Como podemos observar en la siguiente imagen.

![Creación BD en Master][captura1]

### 3. Replicar Base de Datos (BD) MySQL con mysqldump

MySQL ofrece la **herramienta *"mysqldump"* para clonar BD** que tenemos en nuestro servidor. Esta herramienta **puede ser utilizada para generar copias de seguridad** de BD. 

**Puede utilizarse para volcar una o varias BD, para copia de seguridad o para transferir datos a otro servidor SQL** (no necesariamente un servidor MySQL).

Esta herramienta **soporta una gran cantidad de opciones**, las opciones *"--quick"* o *"--opt"* hacen que MySQL **cargue el resultado entero en memoria antes de volcarlo a fichero**, lo que puede ser un problema si se trata de una BD grande.

**Antes de volcar la BD a un fichero** tenemos que tener en cuenta que los **datos pueden estar actualizándose constantemente en la BD a copiar del servidor en producción**, por tanto, **antes de volcar la copia de seguridad** en el archivo *".SQL"* debemos **bloquear cualquier escritura en las tablas de la BD**.

Mediante **las siguientes instrucciones**:

```sh
# mysql -uroot -p
mysql> FLUSH TABLES WITH READ LOCK;
mysql> quit
```

Como podemos observar en la siguiente imagen.

![Bloqueo BD][captura2]

Una vez bloqueada la escritura en la BD, **volcamos la copia de la BD en un fichero *".sql"***.

```sh
# mysqldump contactos -u root -p > /home/sergio/contactos.sql
```

**Una vez realizado el volcado desbloqueamos las tablas** para que se pueda seguir escribiendo en ellas.

```sh
# mysql -uroot -p
mysql> UNLOCK TABLES;
mysql> quit
```

Como podemos observar en la siguiente imagen.

![Backup BD][captura3]

Ya podemos **copiar el fichero** ***".sql"***  generado al realizar el volcado de la BD, en el servidor que actuará como esclavo en la **réplica maestro - esclavo**, mediante el comando: 

```sh
# scp sergio@192.168.30.137:/home/sergio/contactos.sql /home/sergio/
```

Como podemos observar en la siguiente imagen.

![Copia SCP BD][captura4]

La **copia de seguridad *".sql"* incluye las sentencias SQL** para copiar los datos en la BD del servidor esclavo. Sin embargo, **la orden mysqldump no incluye en dicho fichero la sentencia para crear la BD**, por tanto, es necesario que **antes de volcar el contenido del backup ".sql" en la BD del servidor esclavo**, creemos la BD:

```sh
# mysql -uroot -p
mysql> create database contactos;
mysql> quit
```

Y después **volcamos la copia de seguridad en la BD esclava**, mediante el comando:

```sh
# mysql -u root -p contactos < /home/sergio/contactos.sql
```

Como podemos observar en la siguiente imagen.

![Volcado BD][captura5]

Comprobamos que los datos han sido volcados correctamente.

![Comprobacion Volcado BD][captura6]

Para **casos en los que ya se encuentre creada la BD en el servidor esclavo**, podemos **copiar los datos directamente desde el servidor en producción que tiene la BD original a la BD del servidor esclavo**, mediante la siguiente sintaxis de comando:

```sh
# mysqldump contactos -u root -p | ssh <IP_equipo_destino> mysql
```

### 4. Replicación de BD mediante una configuración maestro-esclavo

La **opción anterior es tediosa**, ya que para realizar una replicación de BD se debe realizar a mano. 

**MySQL tiene la opción de configurar el demonio para hacer replicación de las BD** sobre un esclavo a partir de los datos que almacena el maestro.

Se trata de un **proceso automático** que resulta muy adecuado en un entorno de producción real. 

Para **realizar la replicación de BD automática entre servidores maestro - esclavo**, realizaremos los siguientes pasos:

1 -. Debemos **tener la BD clonada** en ambas máquinas.

2 -. **Configuración del servicio MySQL** en el **servidor maestro**:

 - **Editamos como root, el fichero de configuración** de MySQL *"/etc/mysql/my.cnf"*:
 - Comentamos el **parámetro bind-address** que sirve para que escuche a un servidor

```sh
#bind-address 127.0.0.1
```

- Indicamos el archivo donde **almacenar el log de errores**:

```sh
log_error = /var/log/mysql/error.log
```

- Establecemos el identificador del servidor:

```sh
server-id = 1
```

- Indicamos el archivo donde **almacenar el registro binario**:

```sh
log_bin = /var/log/mysql/bin.log
```

- **Guardamos fichero de configuración y reiniciamos el servicio**:

```sh
# /etc/init.d/mysql restart
```

- Si el reinicio no genera **ningún error la configuración en el maestro es correcta**.

Como podemos observar en la siguiente imagen:

![Configuración MySQL en Master][captura7]

3 -. **Configuración del servicio MySQL** en el **servidor esclavo**:

 - **Editamos como root, el fichero de configuración** de MySQL *"/etc/mysql/my.cnf"*:
 
 - **Editamos los mismos parámetros que hemos modificando en el servidor maestro** anteriormente **pero indicando en el parámetro:** *"server-id = 2"*.
 
```sh
server-id = 2
```

![Configuración MySQL en Esclavo][captura8]

- **Comprobamos nuestra versión de mysql** con el comando:

```sh
# mysql --version
```

![Version MySQL][captura9]

- Si la **versión es menor a la 5.5** configuramos los datos del master en el fichero de configuración:

```sh
Master-host = IP_Server_Master
Master-user = usuario_bd
Master-password = Pass_bd
```

- Si nuestra **versión es superior a la 5.5, guardamos fichero y saltamos al paso 4** (como ocurre en mi caso) sin configurar los datos del master, ya que nos generará un error al reiniciar el servicio.

- **Guardamos fichero de configuración y reiniciamos el servicio**:

```sh
# /etc/init.d/mysql restart
```

- Si el reinicio no genera **ningún error la configuración en el maestro es correcta**.

4 -. **Creación usuario y permisos de acceso en MySQL**:
Accedemos de nuevo al servidor maestro y creamos un usuario con sus respectivos permisos para acceder a la BD, mediante los siguientes comandos:

```sh
# mysql -uroot -p
mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%'
IDENTIFIED BY 'esclavo';
mysql> FLUSH PRIVILEGES;
mysql> FLUSH TABLES;
mysql> FLUSH TABLES WITH READ LOCK;
```

Como podemos observar en la siguiente imagen.

![Creación Usuario BD en Master][captura10]

**Obtenemos los datos de la base de datos que vamos a replicar para posteriormente usarlos en la configuración del esclavo**, mediante el comando:

```sh
mysql> SHOW MASTER STATUS;
```

Como podemos observar en la siguiente imagen.

![Datos configuración Master][captura11]

Si nuestra **versión de mysql es superior a la 5.5,** debemos **configurar los datos del master los cuales no se han configurado anteriormente**, por tanto, volvemos a la máquina que trabajará como esclava y configuramos dichos datos con los siguientes comandos, indicando la información obtenida con el comando anterior:

```sh
# mysql -uroot -p
mysql> CHANGE MASTER TO MASTER_HOST='DIRECCION_IP_MASTER',
MASTER_USER='esclavo', MASTER_PASSWORD='esclavo',
MASTER_LOG_FILE='bin.000003', MASTER_PORT=3306;
```

Como podemos observar en la siguiente imagen.

![Configuración usuario Master en Esclavo][captura12]

Por último, **arrancamos el esclavo** y todo está listo para que los **demonios de MySQL** de las dos máquinas **repliquen automáticamente los datos que se introduzcan/modifiquen/borren en el servidor maestro**:

```sh
mysql> START SLAVE;
mysql> UNLOCK TABLES;
```

Como podemos observar en la siguiente imagen.

![Iniciar máquina esclava][captura13]

### 5. Pruebas en servidor maestro y replica en el esclavo automática

En este momento **podemos realizar pruebas en el maestro**, por ejemplo, **introduciendo nuevos datos en la BD maestra** y comprobando que **se replican automáticamente a la BD del servidor esclavo**.

Como podemos observar en la siguiente imagen.

![Replica Datos Maestro-Esclavo][captura14]

### 6. Asegurarse que todo funciona correctamente

Para **asegurarnos que todo funciona correctamente y que el esclavo
no tiene ningún problema para replicar la información** nos vamos al esclavo y con la siguiente orden:

```sh
# mysql -uroot -p
mysql> SHOW SLAVE STATUS\G
```

**Revisamos la variable “Seconds_Behind_Master”** si es distinta de *“null”*, todo estará funcionando perfectamente.

**Si no es así, hay algún error** y los valores de las demás variables indicarán cuál es el problema y cómo arreglarlo. 

Como podemos observar en mi caso, todo esta correcto.

![Variable Seconds Behind Master][captura15]

[captura1]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/1_Create_Database_Master.PNG "Creación BD en Master"

[captura2]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/2_Block_Database.PNG "Bloqueo BD"

[captura3]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/3_Backup_BD.PNG "Backup BD"

[captura4]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/4_Copy_File_SCP.PNG "Copia SCP BD"

[captura5]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/5_Volcado_BD.PNG "Volcado BD"

[captura6]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/5_1_Comprobacion_Volcado_BD.PNG "Comprobación Volcado BD"

[captura7]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/6_File_Configuration_Master_MySQL.png "Configuración MySQL en Master"

[captura8]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/7_File_Configuration_Slave_MySQL.PNG "Configuración MySQL en Esclavo"

[captura9]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/8_MySQL_Version.PNG "Version MySQL"

[captura10]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/9_Create_User_Master.PNG "Creación Usuario BD Master"

[captura11]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/10_Show_Master.PNG "Datos configuración master"

[captura12]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/11_Configuration_User_Slave.PNG "Configuración usuario Master en Esclavo"

[captura13]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/12_Start_Slave.PNG "Iniciar máquina Esclava"

[captura14]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/13_Master_Slave_BD.PNG "Replica Datos Maestro-Esclavo"

[captura15]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica5/capturas/14_Seconds_Behind_Master.PNG "Variable Seconds Behind Master"
