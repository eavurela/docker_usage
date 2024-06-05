<h1 id="uso-de-contenedor-php--apache2">Uso de contenedor PHP + Apache2</h1>
<p>En este documento se detallan algunos comandos útiles y casos de uso para el contenedor de docker con Apache y PHP 7.4</p>
<h1 id="verificación-de-servicios-activos">Verificación de servicios activos</h1>
<p>Para verficar los servicios corriendo de docker, se utilizará el siguiente comando:</p>
<pre><code>root@web01:~# docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
</code></pre>
<p>Si no hay ningun contenedor activo, se deberá iniciar uno.</p>
<h1 id="configuración-del-servicio">Configuración del servicio</h1>
<p>El contenedor consta de un archivo en dónde se declara la imagen que genera y otro en dónde se detallan opciones de puertos, mapeo de rutas y demás. En este caso el archivo de generación de imagen ya está armado.</p>
<p>Se podrá necesitar modificar mapeo de directorios y puertos.  Para eso se editará el docker-compose.yml</p>
<pre><code>version: '3.8'

services:
apache_app:
  build: .
  volumes:
   -  /var/www/sitio1:/var/www/html/sitio1/
   -  /var/www/institucional:/var/www/html/institucional
  restart: always
  ports:
   - '8010:80'
</code></pre>
<p>En esta configuración se genera la imagen, tomando el Dockerfile y mapean rutas del sistema con rutas dentro del contenedor y puertos del sistema con puertos del contenedor.</p>
<p>La imagen de apache2 por defecto usa como documentroot /var/www/html/ por lo tanto cualquier directorio en dónde se haga el mapeo a esa ruta o subrutas será disponible vía web.</p>
<pre><code>   volumes:
   -  /var/www/sitio1:/var/www/html/sitio1/
   -  /var/www/institucional:/var/www/html/institucional
</code></pre>
<p>En este caso, esta parte de la configuración hace que /var/www/sitio1 sea expuesto dentro del contenedor en /var/www/html/sitio1/<br>
Se podría decir que estamos “mostrando” desde el contenedor y desde el servidor apache todo lo que esté dentro en el host local de /var/www/sitio1</p>
<p>Respecto al puerto de ejecución:</p>
<pre><code>  ports:
       - '8010:80'
</code></pre>
<p>Estamos diciendo que el puerto por defecto de apache “80” será expuesto mediante el puerto del servidor 8010. Hay que tener en cuenta que el puerto 8010 no esté en uso en el servidor porque fallará.</p>
<p>Entonces para ver por el navegador el contenido web que tengamos dentro de /var/www/sitio1 deberíamos ingresar a:</p>
<p><a href="http://192.168.1.135:8010/sitio1/">http://192.168.1.135:8010/sitio1/</a></p>
<p>En el caso de que tengamos otra ruta a mapear o utilizar se pueden ir agregando puntos de mapeo.</p>
<p>Ejemplo:</p>
<pre><code>version: '3.8'

services:
apache_app:
  build: .
  volumes:
   -  /var/www/sitio1:/var/www/html/sitio1/
   -  /var/www/institucional:/var/www/html/institucional
   -  /root/projecto01:/var/www/projecto1 (en este caso no se verá por web dado que apache toma a partir del /var/www/html/ )
   -  /usr/lib/libreriaNN:/var/www/libreriaNN 
   -  ruta_host:ruta_contenedor
  restart: always
  ports:
   - '8010:80'
</code></pre>
<h1 id="incio-del-servicio">Incio del servicio</h1>
<p>Para iniciar el servicio, desde dentro del servidor deberás digirirte al directorio en dónde se encuentra compose y docker file y luego prenderlo de la siguiente forma:</p>
<p>Moverse al directorio:</p>
<pre><code>root@web01:~# 
cd /root/federica/
</code></pre>
<p>Verificar el contenido, debería estar Dockerfile y docker-compose.yml</p>
<pre><code>root@web01:~/federica# ls
docker-compose.yml  Dockerfile  nusoap-0.9.5
</code></pre>
<p>Para iniciar el contenedor deberás ejecutar el siguiente comando:</p>
<pre><code>root@web01:~/federica# docker-compose -f docker-compose.yml up -d
</code></pre>
<p>La primera vez que se ejecuté demorará un tiempito mientras instala dentro del contenedor php y sus librerías, y luego iniciará exponiendose en el puerto 8010</p>
<h2 id="apagado-del-servicio">Apagado del servicio</h2>
<p>Para pagar el servicio debemos identificaro:</p>
<pre><code>root@web01:~/federica# docker ps 
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                            NAMES
825c39504cda        federica-apache_app   "apachectl -D FOREGR…"   2 seconds ago       Up 1 second         8080/tcp, 0.0.0.0:8010-&gt;80/tcp   federica-apache_app-1
</code></pre>
<p>Una vez identificado se usa stop y el nombre del contenedor o el ID</p>
<pre><code>root@web01:~/federica# docker stop 825c39504cda  
</code></pre>
<p>Verificar que no esté listado</p>
<pre><code>root@web01:~/federica# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
</code></pre>
<h2 id="prender-servicio-apagado">Prender servicio apagado</h2>
<p>Para verificar los servicios activos pero apagados. el compando es:</p>
<pre><code>root@web01:~/federica# docker ps -a
</code></pre>
<p>En caso estén apagados y se necesite prenderlo, no crearlo, se pueden iniciar simplemente con docker start y nombre o ID del contenedor, en este caso:</p>
<pre><code>root@web01:~/federica# docker start 825c39504cda
</code></pre>
<h2 id="eliminar-un-servicio">Eliminar un servicio</h2>
<p>Se deberá primero apagarlo, buscalo en la lista de apagados y usar:</p>
<pre><code>root@web01:~/federica# docker rm 825c39504cda
</code></pre>
<h2 id="duplicado-de-servicios">Duplicado de servicios</h2>
<p>Se pueden usar varios a la vez, suponiendo que se quiera trabajar con sitios diferentes y ambientes diferentes.</p>
<p>Simplemente se puede duplicar el archivo docker-compose.yml con otro nombre docker-projecto01.yml.<br>
Se debe mapear los directorios que se requieran y asegurarse de usar otro puerto.</p>
<p>Ejemplo:</p>
<p>Dupliqué el compose, y solo hice el mapeo de un directorio y cambié el puerto. Se iniciará en el 8011</p>
<pre><code>version: '3.8'

services:
    apache_app:
      build: .
      volumes:
       -  /var/www/institucional:/var/www/html/institucional
      restart: always
      ports:
       - '8011:80'
</code></pre>
<p>Se debe configurar un nombre en el inicio y luego iniciará sin problemas.<br>
El flag “-p” se usa para nombrarlo. En este caso se llamará contenedor2</p>
<pre><code>root@web01:~/federica# docker-compose -f docker-tutorial01.yml -p contenedor2 up -d
</code></pre>
<p>Como se puede ver, han iniciado los dos independientes cada uno con nombre diferente y puerto diferente.</p>
<pre><code>root@web01:~/federica# docker-compose -f docker-tutorial01.yml -p contenedor2 up -d
[+] Building 0.0s (0/0)                                                                                                                                                                                                                        docker:default
[+] Running 1/1
 ✔ Container contenedor2-apache_app-1  Started                                                                                                                                                                                                           0.7s 
root@web01:~/federica# docker ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                            NAMES
20492f50fb80        federica-apache_app      "apachectl -D FOREGR…"   20 seconds ago      Up 9 seconds        8080/tcp, 0.0.0.0:8010-&gt;80/tcp   federica-apache_app-1
6f123f53fcdb        contenedor2-apache_app   "apachectl -D FOREGR…"   51 seconds ago      Up 2 seconds        8080/tcp, 0.0.0.0:8011-&gt;80/tcp   contenedor2-apache_app-1
</code></pre>

