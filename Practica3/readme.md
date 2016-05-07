# Práctica 3 - Balanceo de carga
> **Autor: Sergio López Ballesteros**

### Introducción
Para **solucionar el problema de sobrecarga en los servidores**, **configuraremos una red entre varias máquinas de forma que tengamos un balanceador que reparta la carga entre varios servidores finales**, quedando una estructura como podemos observar en la siguiente imagen.

![Estructura Granja Web][captura1]

Se puede balancear cualquier protocolo, pero balancearemos los servidores HTTP que tenemos configurados.

De esta forma **conseguiremos una infraestructura redundante y de alta disponibilidad**

Instalaremos y configuraremos los **software de balanceo de carga nginx (configurado como proxy)** y **haproxy**.
Ambas alternativas software no pueden estar en funcionamiento en la misma máquina balanceadora.

### Instalación y configuración de nginx
Para la **instalación y configuración de nginx**, deberemos estar ***logueados como root en el sistema*** y seguir los siguientes pasos:

1 .- **Importaremos la clave del repositorio** de software, mediante los siguientes comandos:

```sh
# cd /tmp/
# wget http://nginx.org/keys/nginx_signing.key
# apt-key add /tmp/nginx_signing.key
```

Como podemos observar en la siguiente imagen:

![Importar Clave Nginx][captura2]

2 .- **Añadir los repositorios de nginx** al fichero *"/etc/apt/sources.list"* y **actualizamos repositorios** con las siguientes órdenes:

```sh
# echo "deb http://nginx.org/packages/ubuntu/ lucid nginx" >> /etc/apt/sources.list

# echo "deb-src http://nginx.org/packages/ubuntu/ lucid nginx" >> /etc/apt/sources.list

# apt-get update
```

Como podemos observar en la siguiente imagen:

![Añadir Repositorio Nginx][captura3]

3 .- **Instalamos paquete de nginx** con la orden:

```sh
# apt-get install nginx
```

Una vez instalado, **configuraremos nginx como balanceador de carga** utilizando un **balanceo mediante el algoritmo de round-robin**.

4 .- Para **configurar nginx como balanceador de carga** tenemos que modificar el fichero de configuración *"/etc/nginx/conf.d/default.conf"*, eliminamos todo el contenido que tenga en ese momento e **introducimos la siguiente configuración indicando los datos necesarios** . 

```sh
upstream apaches {
	server "IP servidor final de la granja web";
	server "IP servidor final de la granja web";
}

server{
	listen "Puerto de escucha servidor web";
	server_name balanceador;
	access_log /var/log/nginx/balanceador.access.log;
	error_log /var/log/nginx/balanceador.error.log;
	root /var/www/;

	location /
	{
		proxy_pass http://apaches;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_http_version 1.1;
		proxy_set_header Connection "";
	}
}
```

Como podemos observar en la siguiente imagen, **como quedaría la configuración para mi caso**:

![Fichero de configuración Nginx][captura4]

Con esta configuración hemos usado **balanceo mediante el algoritmo de “round-robin” con la misma prioridad para todos los servidores**.

5 .- **Reiniciamos servicio nginx** con la orden:

```sh
# service nginx restart
```

6 .- **Si no genera mensaje de error, está configurado correctamente** y **probaremos la configuración realizando peticiones de servicio a la IP del balanceador** desde el mismo balanceador o máquina cliente, con las siguientes órdenes:

```sh
# curl http://192.168.30.138
# curl http://192.168.30.138
```

**Debe mostrar la página de inicio de cada una de las máquinas servidoras, alternativamente**, y por tanto está repartiendo las peticiones entre ambos servidores finales correctamente.

Como podemos observar en las siguientes imágenes, que **las dos peticiones las ha repartido entre ambos servidores finales**:

![Generación de peticiones con Curl][captura5]

![Generación de peticiones con Curl][captura6]

7 .- Si **alguna de las nuestras máquinas servidoras finales es más potente que el resto**, podemos **asignarle una prioridad mayor para pasarle más tráfico que al resto de máquinas**. 
Para ello, debemos **modificar el fichero de configuración de nginx** *"/etc/nginx/conf.d/default.conf"*, e indicar en la **opción upstream el parámetro** *"weight"* para indicarle la carga que le asignamos:

```sh
upstream apaches {
	server "IP servidor final de la granja web" weight=1;
	server "IP servidor final de la granja web" weight=2;
}
```

Como podemos observar en la siguiente imagen:

![Configuración parámetro Weight en Nginx][captura7]

En esta configuración, **la primera máquina servidora es menos potente o más sobrecargada, así que le hemos asignado menos carga de trabajo que a la segunda** (*de cada tres peticiones que lleguen, la segunda máquina atiende dos y la primera atenderá una*).


### Instalación y configuración de haproxy
Para la **instalación y configuración de haproxy**, deberemos estar ***logueados como root en el sistema*** y seguir los siguientes pasos:

1 .- **Instalar haproxy**, mediante la siguiente orden:

```sh
# apt-get install haproxy
```

2 .- **Modificamos el fichero de configuración de haproxy** que se encuentra en *"/etc/haproxy/haproxy.cfg"* e **introducimos la siguiente configuración indicando los datos necesarios:** 

```sh
global
	daemon
	maxconn 256
	
defaults
	mode http
	contimeout 4000
	clitimeout 42000
	srvtimeout 43000
	
frontend http-in
	bind *:"Puerto escucha"
	default_backend servers
	
backend servers
	server m1 "IP Servidor Final":"Puerto escucha" maxconn 32
	server m2 "IP Servidor Final":"Puerto escucha" maxconn 32
```

Como podemos observar en la siguiente imagen, **como quedaría la configuración para mi caso**:

![Fichero de configuración HaProxy][captura8]

3 .- **Lanzamos el servicio haproxy** mediante el comando:

```sh
# /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg
```

4 .- **Si no genera mensaje de error, está configurado correctamente** y **probaremos la configuración realizando peticiones de servicio a la IP del balanceador** desde el mismo balanceador o máquina cliente, con las siguientes órdenes:

```sh
# curl http://192.168.30.138
# curl http://192.168.30.138
```

**Debe mostrar la página de inicio de cada una de las máquinas servidoras, alternativamente**, y por tanto está repartiendo las peticiones entre ambos servidores finales correctamente.

Como podemos observar en las siguientes imágenes, que **las dos peticiones las ha repartido entre ambos servidores finales**:

![Generación de peticiones con Curl][captura5]

![Generación de peticiones con Curl][captura6]

[captura1]: https://github.com/sergiol29/UGR_SWAP/Practica3/capturas/Estructura_GranjaWeb.png "Estructura Granja Web"

[captura2]: https://github.com/sergiol29/UGR_SWAP/Practica3/capturas/Captura1_ImportarClaveNginx.PNG "Importar Clave Nginx"
   
[captura3]: https://github.com/sergiol29/UGR_SWAP/Practica3/capturas/Captura2_AnadirRepositorio.PNG "Añadir Repositorio Nginx"

[captura4]: https://github.com/sergiol29/UGR_SWAP/Practica3/capturas/Captura3_ConfiguracionNginx.PNG "Fichero de configuración Nginx"
   
[captura5]: https://github.com/sergiol29/UGR_SWAP/Practica3/capturas/Captura4_1_Curl.PNG "Generación de peticiones con Curl"
   
[captura6]: https://github.com/sergiol29/UGR_SWAP/Practica3/capturas/Captura4_Curl.PNG "Generación de peticiones con Curl"
   
[captura7]: https://github.com/sergiol29/UGR_SWAP/Practica3/capturas/Captura5_Configuracion_Weight_Nginx.PNG "Configuración parametro Weight en Nginx"
   
[captura8]: https://github.com/sergiol29/UGR_SWAP/Practica3/capturas/Captura6_Configuracion_haproxy.PNG "Fichero de configuración HaProxy"



