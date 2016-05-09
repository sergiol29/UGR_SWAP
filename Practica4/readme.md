# Práctica 4 - Comprobar el rendimiento de servidores web
> **Autor: Sergio López Ballesteros**

### Introducción
Existen diversas **herramientas para comprobar el rendimiento de servidores web** basadas en línea de comandos o interfaz gráfica.

Con estas herramientas **podemos analizar el rendimiento de servidores** Apache, Internet Information Services (IIS), nginx, etc.

Lo habitual es **usar programas de línea de comandos que sobrecarguen lo mínimo** posible las máquinas que estamos usando. 

Utilizaremos las siguientes **herramientas para comprobar el rendimiento de nuestra granja web**:

 - Apache Benchmark
 - Siege

**Estas herramientas de rendimiento deben utilizarse en una máquina independiente a las que forman la granja web**, es decir, **una máquina que realizará la tarea de cliente para aplicar las tareas de rendimiento** hacia las máquinas servidoras finales y la máquina balanceadora de carga, por tanto quedaría una estructura de máquinas como la siguiente:

![Estructura Granja Web][captura1]



### Rendimiento de servidores web con Apache Benchmark

**Apache Benchmark (ab) es una utilidad del servidor Apache y permite comprobar el rendimiento de un servidor web**.

Si necesitamos **comprobar el rendimiento de un servidor web, debemos entrar en un terminal y ejecutar el comando "ab"**:

```sh
$ ab -n <nº peticiones> -c <nº concurrencia> <dirección>
```

Por ejemplo, con la **siguiente instrucción**:

```sh
$ ab -n 1000 -c 10 http://192.168.30.138/test.php
```

Indicamos que se solicite la pagina con dirección *"http://192.168.30.138/test.php"*, *"1000 veces"* **( -n 1000 indica el número de peticiones )** y hacer esas **peticiones concurrentemente** de *"10 en 10"* **( -c 10 indica el nivel de concurrencia ).**

Si no disponemos de Apache Benchmark, **instalamos el paquete con la orden**:

```sh
# apt-get install apache2-utils
```


#### Pruebas rendimiento con Apache Benchmark
Por tanto, vamos a someter a **pruebas de rendimiento a nuestras máquinas servidoras finales y máquina balanceadora de carga con el software de balanceo** *nginx* y *haproxy* en funcionamiento, alternativamente.

----------

Para las **pruebas de rendimiento hacia el servidor final y balanceador de carga con diferentes software de balanceo, ejecutaremos 5000 peticiones de 10 en 10**, mediante la siguiente instrucción cambiando la IP por la necesaria:

```sh
$ ab -n 5000 -c 10 http://192.168.30.137
```

Dicha **instrucción nos devuelve diferentes parámetros de la prueba ejecutada**, como podemos observar en las siguientes imágenes.

![Comando AB][captura2]

![Ejecución AB][captura3]
  
Tras realizar varias **medidas sobre nuestras máquinas servidoras finales y máquina balanceadora de carga con el software de balanceo** *nginx* y *haproxy* y obtener la información que necesitamos valorar, los **resultados obtenidos** son los siguientes:

![Datos ejecución AB][captura4]

Realizamos una **comparación entre las tres máquinas y obtenemos las siguientes gráficas**:

![Gráfica Time Taken Test][captura5]

En la **gráfica anterior** *"Time Taken Test"* podemos observar una **comparación del tiempo que tarda cada máquina en realizar los test** y podemos ver como **las pruebas contra el servidor solo, son el doble más rápidas que los balanceadores**. 

Esto se debe a que **los test van directos hacia el servidor final sin tener que pasar por el balanceador de carga**.

En cambio, la **comparación de ambos balanceadores de carga** *"haproxy"* es un poco más rápido que *nginx*.

![Gráfica Request Per Second][captura6]

En la **gráfica anterior** *"Request Per Second"* podemos observar una **comparación del número de peticiones que responde por segundo cada máquina** y podemos ver como **el servidor solo responde muchas más peticiones** que ambos balanceadores.

Esto se debe a que como se ha explicado antes las **peticiones contra el servidor solo son más rápidas**. 

### Rendimiento de servidores web con Siege
**Siege es una herramienta de generación de carga HTTP para benchmarking**, a través de línea de comandos. Permite **realizar baterías de tests contra varias URLs diferentes del mismo servidor**, en lugar de usar la misma URL.

Dispone de **diferentes modos de ejecución**: 

 - **-b**: Ejecuta los **tests sin pausas** con lo que comprobaremos el rendimiento general. Sin la opción -b se inserta un segundo de pausa entre las diferentes peticiones.
 
 - **-t**: **Tiempo** que estará **ejecutándose** la herramienta. *( Ej: -t60S; -t1H; -t120M )*
 
 - **-v**: Muestra información extendida.

**Ejemplo de ejecución**, utilizará 15 usuarios concurrentes (valor por defecto) durante 60 segundos:

```sh
$ siege -b -t60S -v http://192.168.30.137
```

Si no disponemos de Siege, **instalamos el paquete con la orden**:

```sh
# apt-get install siege
```


#### Pruebas rendimiento con Siege
Por tanto, vamos a someter a **pruebas de rendimiento a nuestras máquinas servidoras finales y máquina balanceadora de carga con el software de balanceo *nginx* y *haproxy*** en funcionamiento, alternativamente.

----------

Para las **pruebas de rendimiento hacia el servidor final y balanceador de carga con diferentes software de balanceo, ejecutaremos durante 60 segundos sin pausas entre cada y balanceador de carga con diferentes software de balanceo con 15 usuarios concurrentes**, mediante la siguiente instrucción cambiando la IP por la necesaria:

```sh
$ siege -b -t60S -v http://192.168.30.137
```


Dicha **instrucción nos devuelve diferentes parámetros de la prueba ejecutada**, como podemos observar en las siguientes imágenes.

![Comando Siege][captura7]

![Ejecución Siege][captura8]

Tras realizar varias **medidas sobre nuestras máquinas servidoras finales y máquina balanceadora de carga con el software de balanceo *nginx* y *haproxy*** y obtener la información que necesitamos valorar, los **resultados obtenidos** son los siguientes:

![Datos ejecución Siege][captura9]
![Datos ejecución Siege][captura10]

Realizamos una **comparación entre las tres máquinas y obtenemos las siguientes gráficas**:

![Gráfica Elapsed Time y Availability][captura11]

En la **gráfica anterior** *"Elapsed Time"* podemos observar una **comparación del tiempo transcurrido para la ejecución** y podemos ver como el **balanceador nginx dispone de un tiempo más alto**.

![Gráfica Response Time y Transaction Rate][captura12]

En la **gráfica anterior** *"Response Time"* podemos observar una **comparación del tiempo de respuesta para cada petición** y podemos ver como el **servidor solo responde más rápido a las peticiones**, ya que las peticiones van directamente al servidor final sin pasar por los balanceadores.

En la **gráfica anterior** *"Transaction Rate"* podemos observar una **comparación del número de peticiones atendidas por segundo** y podemos ver como el **servidor solo responde más peticiones por segundo**, ya que tiene un tiempo de respuesta menor y responde más rápido.

![Gráfica Longest Transaction][captura13]

En la **gráfica anterior** *"Longest Transaction"* podemos observar una **comparación de las peticiones más largas** y podemos ver como el **balanceador haproxy realiza las peticiones más largas**, ya que tiene un tiempo de respuesta mayor.



### Anexo
Los **datos de las mediciones** para cada máquina y **gráficas resultantes** pueden visitarse en el siguiente enlace: [Mediciones y Gráficas](https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/Resultados_Ejecuciones.xlsx "Mediciones y Gráficas")

[captura1]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/capturas/Estructura_Granja_Web.png "Estructura Granja Web"

[captura2]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/capturas/Ejecucion_1_ab.PNG "Comando AB"

[captura3]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/capturas/Ejecucion_ab.PNG "Ejecución AB"

[captura4]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/capturas/Datos_Medidas_AB.PNG "Datos Ejecución AB"

[captura5]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/capturas/Grafica_AB_Time_Taken_Test.PNG "Gráfica Time Taken Test"

[captura6]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/capturas/Grafica_AB_Request_per_second.PNG "Gráfica Request Per Second"

[captura7]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/capturas/Ejecucion_1_Siege.PNG "Comando Siege"

[captura8]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/capturas/Ejecucion_Siege.PNG "Ejecución Siege"

[captura9]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/capturas/Datos_Medidas_Siege.PNG "Datos Ejecución Siege"

[captura10]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/capturas/Datos_Medidas_Siege_1.PNG "Datos Ejecución Siege"

[captura11]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/capturas/Grafica_Siege_ElapseTime.PNG "Gráfica Elapsed Time y Availability"

[captura12]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/capturas/Grafica_Siege_ResponseTime_TransactionRate.PNG "Gráfica Response Time y Transaction Rate"

[captura13]: https://github.com/sergiol29/UGR_SWAP/blob/master/Practica4/capturas/Grafica_Siege_LongestTransaction.PNG "Gráfica Longest Transaction"
