# Práctica 6 - Discos en RAID

> **Autor: Sergio López Ballesteros**

### 1. Introducción
 
**RAID es una forma de distribuir datos en varios discos** y que puede ser gestionada por hardware dedicado o por software. 

Existen **diversos tipos de RAID**.

**Configuraremos dos discos en RAID 1 por software**, esta configuración RAID ofrece una gran seguridad al replicar los datos en los dos discos.

### 2. Configuración del RAID por software
Se **añadirán** a un sistema ya instalado y funcionando en una máquina virtual **dos discos del mismo tipo y capacidad**, de forma que en **total tendremos tres discos** (el sistema operativo estará ya instalado en /dev/sda y añadiremos dos discos más, que serán el /dev/sdb y el /dev/sdc, para configurar el dispositivo de almacenamiento RAID en estos dos discos nuevos de igual tamaño).

Para **añadir dichos discos al sistema** utilizaremos las opciones que nos ofrece el software de virtualización, como podemos ver en las siguientes imágenes:

 1. Con la **máquina virtual apagada**, accedemos a la **configuración de la máquina virtual** y añadimos nuevo hardware. 
 ![Añadir disco][captura1]
 2. Seleccionamos **tipo de disco**. 
 ![Añadir disco][captura2]
 3. Indicamos *"crear nuevo disco virtual"* 
 ![Añadir disco][captura3]
 4. Seleccionamos la **misma capacidad que disco principal** donde se encuentra instalado el sistema operativo. 
 ![Añadir disco][captura4]
 5. Repetimos el proceso para **añadir un segundo disco al sistema** y **quedaría el sistema con tres discos instalados**. 
 ![Añadir disco][captura5]

Iniciamos el SO e **instalamos el software para configurar el RAID**:
```sh
sudo apt-get install mdadm
```

Buscamos **identificación asignada por Linux** de ambos discos:
```sh
sudo fdisk -l
```

![Fdisk][captura6]

Como podemos observar **disponemos de 3 discos** en nuestro sistema:

 1. **Disco /dev/sda**: Disco donde esta instalado el SO.
 2. **Disco /dev/sdb**: Disco añadido aún sin formato.
 3. **Disco /dev/sdc**: Disco añadido aún sin formato.

**Creamos el RAID 1**, usando el dispositivo *"/dev/md0"*, indicando el los discos a utilizar, así como su ubicación:
```sh
sudo mdadm -C /dev/md0 --level=raid1 --raid-devices=2 /dev/sdb /dev/sdc
```

![Creación RAID][captura7]

**Nota**: El dispositivo se habrá creado con el nombre *"/dev/md0"*, pero en cuanto reiniciemos la máquina, Linux lo renombrará a *"/dev/md127"*

Una vez **creado el dispositivo RAID**, damos formato a *"/dev/md0"* con el **comando mkfs** que formateara por defecto el **disco con formato ext2**:

```sh
sudo mkfs /dev/md0
```

![Formato RAID][captura8]

Creamos el **directorio en el que se montará la unidad del RAID**:

```sh
sudo mkdir /dat
sudo mount /dev/md0 /dat
```

Comprobamos que la unidad se a **montado correctamente**, mediante el comando:

```sh
sudo mount
```

![Comando mount][captura9]

Podemos observar que la **unidad** **"/dev/md0"** **se ha montado correctamente**.

Comprobamos el **estado del RAID**:

```sh
sudo mdadm --detail /dev/md0
```

![Estado RAID][captura10]

Para finalizar el proceso, **configuramos el sistema para que monte el dispositivo RAID creado al arrancar el sistema**. 

Para ello, **editamos el archivo** */etc/fstab* y añadimos la línea correspondiente para montar automáticamente dicho dispositivo.

Utilizamos el **identificador único de cada dispositivo de almacenamiento**.

Para **obtener los UUID de todos los dispositivos** de almacenamiento que tenemos:

```sh
ls -l /dev/disk/by-uuid/
```

![UUID][captura11]

**Anotamos el identificador del dispositivo RAID** creado y lo **añadimos al final del archivo** *"/etc/fstab"*  para que se monte automáticamente al iniciar la máquina.

```sh
UUID=fcd4b3d9-2aa2-4560-92c0-26680d52139c /dat ext2 defaults 0 0
```

![Fstab][captura12]

**Reiniciamos el sistema** para que los cambios surjan efecto.

### 2. Pruebas sobre unidad RAID por software
Una vez reiniciado el sistema podemos **comprobar si la unidad RAID ha sido montada correctamente** y si se ha renombrado como *"/dev/md127"*.

Podemos observar que la **unidad ha sido montada correctamente y además ha sido renombrada**.

![Mount][captura13]

Por tanto, como esta montada correctamente podemos **simular un fallo en uno de los discos**:

```sh
sudo mdadm --manage --set-faulty /dev/md127 /dev/sdb
```

![Faulty Raid][captura14]

Al **comprobar el estado de la unidad RAID**, podemos observar como la unidad *"/dev/sdb"* esta marcada con un fallo de disco.

![Estado Faulty Raid][captura15]

**Podemos retirar “en caliente”** el disco que está marcado como que ha fallado:

```sh
sudo mdadm --manage --remove /dev/md127 /dev/sdb
```

![Remove RAID][captura16]

Al **comprobar el estado de la unidad RAID**, podemos observar como la unidad *"/dev/sdb"* ha sido retirada.

![Estado Remove RAID][captura17]

**Podemos añadir “en caliente”**, un nuevo disco para reemplazar al disco que hemos retirado:

```sh
sudo mdadm --manage --add /dev/md127 /dev/sdb
```

![Añadir RAID][captura18]

Al **comprobar el estado de la unidad RAID**, podemos observar como la unidad *"/dev/sdb"* ha añadida correctamente y **marcada como reconstruida**.

![Estado Añadir RAID][captura19]


[captura1]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/1_Add_Disk.PNG "Añadir disco"

[captura2]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/2_Add_Disk.PNG "Añadir disco"

[captura3]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/3_Add_Disk.PNG "Añadir disco"

[captura4]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/4_Add_Disk.PNG "Añadir disco"

[captura5]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/5_Add_Disk.PNG "Añadir disco"

[captura6]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/6_Fdisk.PNG "Fdisk"

[captura7]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/7_CrearRaid.PNG "Creación RAID"

[captura8]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/8_FormatoRaid.PNG "Formato RAID"

[captura9]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/9_Mount.PNG "Comando Mount"

[captura10]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/10_EstadoRaid.PNG "Estado RAID"

[captura11]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/11_uuid.PNG "UUID"

[captura12]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/12_fstab.PNG "Fstab"

[captura13]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/13_mount.PNG "Mount"

[captura14]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/14_FaultyRaid.PNG "Faulty Raid"

[captura15]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/15_Estado_FaultyRaid.PNG "Estado Faulty Raid"

[captura16]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/16_RemoveRaid.PNG "Remove RAID"

[captura17]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/17_Estado_RemoveRaid.PNG "Estado Remove RAID"

[captura18]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/18_AddRaid.PNG "Añadir RAID"

[captura19]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica6/capturas/19_Estado_AddRaid.PNG "Estado Añadir RAID"