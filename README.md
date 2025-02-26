# Practica-IAW-5.2: Despliegue de WordPress con Docker y Docker Compose


## Estructura de la Práctica

![Estructura](./Imagenes/Estructura.png)


## Objetivo de la Práctica

El objetivo de esta práctica es desplegar un sitio Wordpress en AWS utilizando Docker y Docker Compose, dónde automatizaremos el despligue de Wordpress incluyendo la base de datos de MySQL, PHPMyAdmin y con portal HTTPS aseguraremos una conexión segura a través de nuestro dominio con un certificado SSL.

>**NOTA**  
>Para esta práctica, usaremos para los servicios a implementar, imágenes de Docker Hub de forma que para cada servicio se usen las siguientes:

- **wordpress:** bitnami/wordpress.
- **mysql:** mysql.
- **phpmyadmin:** phpmyadmin/phpmyadmin.
- **https-portal:** steveltn/https-portal.


## Recordatorio de cambio de la dirección IP para actualizar el nombre de dominio de  nuestra página web

>[!IMPORTANT]  
>Para esta práctica, crearemos un nuevo nombre de dominio personalizado dónde asociaremos la dirección IP de nuestra máquina, para posteriormente establecer el dominio cómo seguro.

![Hostnames](./Imagenes/Hostnames.png)


## Desarrollo de la práctica 


- **`docker-compose.yml`:** Este archivo permite ejecutar un entorno completo con MySQL, PHPMyAdmin, Wordpress y HTTPS-PORTAL de manera automatizada.


**Contenido del archivo:** 
   
   ```bash

  version: '3.4'

  services:
    mysql:
      image: mysql:9.1
      ports: 
        - 3306:3306
      environment: 
        - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
        - MYSQL_DATABASE=${MYSQL_DATABASE}
        - MYSQL_USER=${MYSQL_USER}
        - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      volumes: 
        - mysql_data:/var/lib/mysql
      networks: 
        - backend-network
      restart: always
    
    phpmyadmin:
      image: phpmyadmin:5.2.1
      ports:
        - 8080:80
      environment: 
        - PMA_ARBITRARY=1
      networks: 
        - backend-network
        - frontend-network
      restart: always
      depends_on: 
        - mysql

    prestashop:
      image: prestashop/prestashop:8
      environment: 
        - DB_SERVER=mysql
      volumes:
        - prestashop_data:/var/www/html
      networks: 
        - backend-network
        - frontend-network
      restart: always
      depends_on: 
        - mysql

    https-portal:
      image: steveltn/https-portal:1
      ports:
        - 80:80
        - 443:443
      restart: always
      environment:
        DOMAINS: "${DOMAIN} -> http://prestashop:80"
        STAGE: 'production'
      networks:
        - frontend-network

  volumes:
    mysql_data:
    prestashop_data:

  networks: 
    backend-network:
    frontend-network:

   ```


Dado el contenido del archivo entero iremos desglosándo poco a poco explicando la función de cada bloque de comandos.

![MySQL](./Imagenes/MySQL.png)

En primer bloque, declaramos que el servicio va a ser llamado "mysql", le especificaremos la imagen oficial de MySQL, que en este caso será la versión 9.1 de Docker Hub.

Especificaremos el puerto correspondiente que en este caso será el 3306 típico de MySQL. Especificando el mismo puerto tanto para la máquina host cómo para el puerto de dentro del contenedor.

Le indicaremos el nombre de usuario de MySQL, contraseña así cómo nombre de base de datos.


Definiremos también el volumen para la persistencia de datos, que en este caso será el directorio "/var/lib/mysql".

Indicaremos la red de conexión, que en este caso será la red backend y por último ajustaremos la política de reinicio, indicando que si el contenedor falla o si el servidor se reinicia, se reinicie automáticamente.


![PHPMyAdmin](./Imagenes/PHPMyAdmin.png)

En este caso, definiremos el servicio phpmyadmin, dónde le especificamos la imagen de phpmyadmin así cómo versión, le indicaremos los puertos de forma que en el host (navegador) su puerto sea el 8080, pero dentro del contenedor su puerto sea el 80.

Siguiendo, mediante la opción "PMA_ARBITRARY=1", permitirá conectarse a cualquier servidor MySQL.

En siguiente lugar, le especificaremos las redes que en este caso utilizará la red backend y frontend.

Al igual que MySQL, le aplicaremos la correspondiente política de reinicio por si el sistema falla.

El paso siguiente será indicarle la dependencia con MySQL, significando que PHPMyAdmin se iniciará después de que MySQL esté en marcha.

En siguiente lugar, procedemos a la explicación de los parámetros referentes a prestashop.


![PrestaShop](./Imagenes/PrestaShop.png)

En primer lugar cómo es de esperar, declaramos el servicio prestashop, después le indicamos la imagen a usar, que será la de prestashop en su versión nº 8, le especificaremos también la variable indicando el servidor MySQL a usar.

Posteriormente definimos los volúmenes para la persistencia de datos, que en este caso, prestashop estará alojado en el directorio "/var/www/html". Le indicamos también las redes que queremos que esté conectado, en este caso será la red frontend y backend.

Ajustamos la política de reinicio y por último le especificamos también la dependencia con MySQL, para que arranque una vez que MySQL esté en marcha.


Procedemos a explicar los parámetros del servicio Portal HTTPS.

![Portal HTTPS](./Imagenes/HTTPS-PORTAL.png)

Cómo es de esperar, declaramos el servicio https-portal actuándo cómo servidor proxy inverso con soporte HTTPS.

Le indicamos la imagen a usar, en este caso es "steveltn/https-portal:1" de forma que proporcione un proxy NGINx con certificados SSL firmados automáticos.

Le indicamos los puertos, al ser una tecnología relacionada con HTTP y HTTPS, le indicaremos sus puertos correspondientes, tanto cómo el 80 y el 443.

Ajustamos la misma política de reinicio al igual que en los demás servicios.

En la sección de importar las variables de entorno, indicamos que defina el dominio que apuntará a PrestaShop, redirigiendo el tráfico hacia el contenedor de prestashop en el puerto 80, y especificándole que tome el valor de la variable para definir el dominio personalizado.

El parámetro "STAGE: 'production'" indica que el entorno es de producción, y que se usarán certificados reales de Let´s Encrypt.

Por último, indicamos que se conecte a la red frontend para comunicarse con prestashop.

En último lugar del archivo, indicaremos los volúmenes para la persistencia de datos y le indicaremos las redes, que cómo hemos mencionado anteriormente, hemos definido la red frontend y backend.

![Volumenes y redes](./Imagenes/volumenes_n.png)


Tras explicar la función del archivo, seguimos con el desarrollo de la práctica.


### Variables incluidas en el archivo ".env"

![Variables](./Imagenes/Variables.png)


### Comprobaciones generales

Una vez redactados los comandos útiles, probaremos la ejecución del script a modo que se ejecute sin errores.

![Ejecución Docker Compose](./Imagenes/Docker.png)

Tras ejecutar el script, nos dirigimos a la página de nuestro dominio y cómo podemos observar aparecerá la página de instalación de PrestaShop


![Instalador de PrestaShop](./Imagenes/Instalador.png)

Seleccionamos el idioma y seguimos.

En siguiente lugar, aparecerá la página de las condiciones, las aceptamos sin mayor importancia y avanzamos.

![Condiciones](./Imagenes/Condiciones.png)

La siguiente es el ajuste de los datos, introducimos nuestros datos así cómo un nombre para la tienda y ámbito, en mi caso he activado la casilla "Activar SSL" ya que la página va a ser segura.

El siguiente paso será ajuste de configuración del contenido de nuestra tienda, aceptamos y seguimos.

![Contenido Tienda](./Imagenes/Contenido.png)


Lo siguiente en realizar será la introducción de algunos datos de conexión hacia la base de datos, en docker, no hay que declarar IPs sino nombres de servicio, por lo que en la dirección de nuestra base de datos introducimos "mysql", introducimos los demás datos acorde con las variables definidas en nuestro archivo .env, si la comprobación aparece en verde quiere decir que la conexión se ha establecido con éxito.


Tras aceptar, prestashop empezará a crear su base de datos en el servidor de MySQL

![Creación de la base de datos](./Imagenes/Creación_BBDD.png)

Tras unos minutos, prestashop indicará que se ha instalado con éxito, nos indicará el correo electrónico configurado así cómo la contraseña, ya que estos datos son los de acceso para la administración de la tienda.


Tras aceptar el aviso, ya habrá cargado nuestra tienda.

![Tienda](./Imagenes/Tienda.png)


### Detalles de nuestro Certificado

Tras haber completado toda la instalación, visualizaremos los detalles de nuestro certificado.


![Certificado](./Imagenes/Certificado.png)