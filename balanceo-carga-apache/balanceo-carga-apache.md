# Balanceo de carga en Apache

![logo-apache](capturas/logo-apache.png)

## 1. Introducción.
Aparte de poder usar apache como servidor Web, también podemos usar con otras características, como proxy inverso, la que vamos a explicar en esta guía.

## 2. Activar módulos.
Primero vamos a configurar apache para que permita usarse como proxy inverso, para ello vamos a instalar una serie de módulos en Apache. Los módulos son:
```
sudo a2enmod proxy proxy_http proxy_ajp rewrite deflate headers proxy_balancer proxy_connect proxy_html lbmethod_byrequests
```

![01-instalacion-modulos](capturas/01-instalacion-modulos.png)


Con esto realizado, vamos a reiniciar el apache.
```
sudo systemctl restart apache2
```

![02-restart-apache](capturas/02-restart-apache.png)


Ahora vamos a configurar el apache como balanceador de carga, para esto vamos fichero 000-default.conf en el directorio /etc/apache2/sites-available y añadimos las siguientes lineas.
```
<VirtualHost *:80>
  # Dejamos la configuración del VirtualHost como estaba
  # sólo hay que añadir las siguiente directivas: Proxy y ProxyPass

  <Proxy balancer://mycluster>
      # Server 1
      BalancerMember http://IP-HTTP-SERVER-1:8081

      # Server 2
      BalancerMember http://IP-HTTP-SERVER-2:8082

      # Server 3
      BalancerMember http://IP-HTTP-SERVER-3:8083

      # Server 4
      BalancerMember http://IP-HTTP-SERVER-4:8084
  </Proxy>

  ProxyPass / balancer://mycluster/
</VirtualHost>
```
Donde IP-HTTP-SERVER corresponde con el nombre o ip de nuestro servidor.

![03-proxy-conf](capturas/03-proxy-conf.png)


Y tras esto volvemos a reiniciar apache.

![02-restart-apache](capturas/02-restart-apache.png)



## 3. Lanzar entorno.
Con lo que hemos realizado ya tenemos el servidor apache configurado para realizar un balanceo de carga con los puertos 8081,8082,8083,8084. Ahora necesitamos 4 jboss en los puertos. Para ello vamos a crear los contenedores con docker de los 4 wildfly con la aplicación.

![04-contenedores-jboss](capturas/04-contenedores-jboss.png)


Aplicación web básica con un index.jsp que muestra el puerto.

![05-index.jsp](capturas/05-index.jsp.png)


Estructura del proyecto con el fichero docker-compose. La configuración de WildFly es la que hemos usado en practicas anteriores.

![05-tree](/capturas/05-tree.png)


Con esto vamos a levantar el entrono.

![06-levantar-compose](/capturas/06-levantar-compose.png)


Ahora si accedemos al localhost seguido del nombre de la aplicación podremos ver como apache va redireccionado a puertos diferentes cada vez que recargamos la página.

![07-comprobacion1](/capturas/07-comprobacion1.png)

![07-comprobacion2](/capturas/07-comprobacion2.png)

![07-comprobacion3](/capturas/07-comprobacion3.png)

![07-comprobacion4](/capturas/07-comprobacion4.png)