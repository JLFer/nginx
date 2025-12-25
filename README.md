# Proyecto Nginx
Práctica HTTP IV – Nginx (ASIR)

## Índice
- [1. Introducción](#1-introducción)
- [2. Comparativa con Apache](#2-comparativa-con-apache)
- [3. Esquema de red](#3-esquema-de-red)
- [4. Instalación](#4-instalación)
- [5. Casos prácticos](#5-casos-prácticos)
- [6. Referencias](#6-referencias)

## 1. Introducción
En esta práctica se monta un servidor Nginx como alternativa a Apache para ofrecer servicios web en una empresa ficticia.
Nginx es un servidor web/proxy inverso ligero de alto rendimiento, pensado para manejar muchas conexiones simultáneas consumiendo pocos recursos. 
El objetivo es practicar su instalación y configuración básica: virtual hosts, control de acceso por red, autenticación básica y acceso seguro mediante HTTPS, documentándolo todo en este repositorio.

## 2. Comparativa con Apache
Apache ha sido durante muchos años el servidor web “clásico”, pero su forma de trabajar con procesos e hilos hace que le cueste más cuando hay muchas peticiones al mismo tiempo.
Nginx en cambio, está pensado justo para eso, con pocos procesos es capaz de manejar muchas conexiones a la vez, consume menos RAM y CPU y se nota mucho sirviendo contenido estático (HTML, imágenes, CSS, JS, etc.).
A nivel de configuración también hay diferencias: Apache permite usar `.htaccess` y tiene un montón de módulos que se pueden activar y desactivar, mientras que en Nginx todo se lleva en sus propios ficheros y los módulos se definen al compilar.

## 3. Esquema de red
El servidor Nginx tiene tres interfaces de red configuradas en la máquina virtual: 
- enp0s9 (interna): 192.168.3.1/24  ← red interna
- enp0s8 (externa): 10.16.0.1/16    ← red externa
- enp0s3: 192.168.1.137/24          ← interfaz con conexión a internet

<img width="1919" height="1029" alt="image" src="https://github.com/user-attachments/assets/186c3bb5-0f4a-43e0-98ab-1a8ceae2270c" />


## 4. Instalación
El servidor web se ha montado sobre una máquina virtual con Ubuntu Server 22.04.
Pasos seguidos:
- Actualizar los repositorios del sistema: `sudo apt update && sudo apt upgrade -y`
- Instalar el paquete `nginx` desde los repositorios oficiales: `sudo apt install nginx -y`
- Comprobar que el servicio Nginx queda activo y habilitado al arrancar:
  - `sudo systemctl status nginx`
  - `sudo systemctl is-enabled nginx`
- Verificar desde un navegador que se muestra la página por defecto de Nginx en el puerto 80 accediendo a `http://192.168.3.1` y `http://10.16.0.1`

<img width="1917" height="1033" alt="image" src="https://github.com/user-attachments/assets/4984b995-5487-44f9-bc8e-7e89a5c153a1" />



## 5. Casos prácticos
### a) Versión de Nginx instalado
Para comprobar la versión instalada de Nginx en el servidor se ha usado el comando `nginx -v`: muestra la versión exacta `nginx version: nginx/1.18.0 (Ubuntu)`, que es la que se ha utilizado en la práctica.

<img width="1919" height="1033" alt="image" src="https://github.com/user-attachments/assets/dfb15ab1-de8f-44ff-8c21-6a1d290d5aa5" />


### b) servicio asociado
Nginx se gestiona con `systemd` mediante la unidad `nginx.service`.
Los comandos más utilizados han sido:
- `sudo systemctl status nginx`
- `sudo systemctl restart nginx`
- `sudo systemctl reload nginx`

### c) ficheros de configuración.
En esta práctica no se ha modificado el fichero principal `/etc/nginx/nginx.conf`; se ha dejado con la configuración por defecto de Ubuntu.
Los sitios virtuales se han definido en `/etc/nginx/sites-available/` y se han activado mediante enlaces simbólicos en `/etc/nginx/sites-enabled/`, verificando la sintaxis con `nginx -t` antes de recargar el servicio.

### d) Página por defecto
Se ha comprobado el correcto funcionamiento de Nginx accediendo desde el cliente interno a `http://192.168.3.1`, donde se muestra la página de bienvenida por defecto del servidor.

Esa página no se ha personalizado porque en el momento de redactar la memoria, el sitio por defecto ya estaba deshabilitado.

<img width="1917" height="1033" alt="image" src="https://github.com/user-attachments/assets/eb8f8ab7-9cc3-4c35-a17d-c2212dc48f1c" />

### e) Virtual hosting: web1 y web2
Se han creado dos sitios: `/var/www/web1/index.html` y `/var/www/web2/index.html`, cada uno con un mensaje de bienvenida indicando si es la web1 o la web2 y el nombre del alumno.
Para servir ambas webs en el mismo servidor se han definido dos bloques `server` en Nginx con distinto `server_name` (`www.web1.org` y `www.web2.org`) y su `root` correspondiente.

<img width="1919" height="1033" alt="image" src="https://github.com/user-attachments/assets/7cd2d7f6-3f80-4a3f-8a58-9840375717e9" />

Después se han habilitado los sitios y recargado el servicio, y en el cliente interno se ha editado `/etc/hosts` para que ambos nombres apunten a la IP del servidor.

<img width="1919" height="1029" alt="image" src="https://github.com/user-attachments/assets/d5ad4dbc-131b-4a02-bfe5-5ac9a07932d7" />

Las capturas muestran el acceso a `http://www.web1.org` y `http://www.web2.org`, donde cada dominio carga su página correspondiente.

<img width="1919" height="1030" alt="image" src="https://github.com/user-attachments/assets/c747dbc5-755f-4a43-9428-de81a5e327bc" />

### f) Autentificación, Autorización y Control de acceso
Aquí se controla el acceso por red a `www.web1.org` y `www.web2.org`.
Se ha dejado `www.web1.org` accesible desde cualquier red y en `www.web2.org` se han añadido `allow 192.168.3.0/24;` y `deny all;` en el bloque `server` para limitarlo solo a la red interna.

<img width="1919" height="1031" alt="image" src="https://github.com/user-attachments/assets/56f392d3-9412-45a6-bda6-aece4df730d9" />

Tras recargar Nginx, desde la red interna se accede a ambos sitios y desde la red externa solo responde `www.web1.org`.

- Acceso desde Red Interna:
<img width="1919" height="1031" alt="image" src="https://github.com/user-attachments/assets/5ad177bc-4cd4-4679-a238-d24690dfbd28" />

- Acceso desde Red Externa:
<img width="1919" height="1030" alt="image" src="https://github.com/user-attachments/assets/a0c405d5-c565-494e-8ca5-b50fa97a1eb7" />

### g) Autentificación, Autorización y Control de acceso
En `www.web1.org` se ha creado el directorio `/privado` protegido con autenticación básica para que solo entren usuarios válidos.
Se ha generado `/var/www/web1/privado` con su `index.html` y el fichero `/etc/nginx/.htpasswd`, y en el `location /privado/` se han añadido `auth_basic "Zona privada";` y `auth_basic_user_file /etc/nginx/.htpasswd;`.

<img width="1919" height="1030" alt="image" src="https://github.com/user-attachments/assets/8f78b29d-1589-4be4-8f9d-908df0ce942c" />

Desde el cliente interno, al acceder a `http://www.web1.org/privado` aparece la ventana de login y, tras introducir credenciales correctas, se muestra la página privada.

<img width="1919" height="1030" alt="image" src="https://github.com/user-attachments/assets/ab135c2d-c042-4fe4-91f7-b82efe8464a6" />

<img width="1919" height="1033" alt="image" src="https://github.com/user-attachments/assets/9c730e86-09d0-47e7-8936-f5d2edf11fda" />

  
### h) Autentificación, Autorización y Control de acceso

En este apartado se ha ajustado el directorio `/privado` de `www.web1.org` para que desde la red interna no se pidan credenciales y desde la red externa sí.
Para conseguirlo se ha modificado el bloque `location /privado/` añadiendo `satisfy any;` junto con `allow 192.168.3.0/24;` y `deny all;`, manteniendo la autenticación básica con `auth_basic` y `auth_basic_user_file /etc/nginx/.htpasswd;`.

<img width="1919" height="1029" alt="image" src="https://github.com/user-attachments/assets/fd97acad-6c88-420a-84fb-eaf1426ef0e6" />

Con esta combinación, los equipos de la red interna quedan autorizados solo por su IP y entran en `http://www.web1.org/privado` sin ventana de login, mientras que los equipos de la red externa ven el cuadro de usuario y contraseña y solo acceden a la zona privada tras autenticarse correctamente.

- Acceso desde Red Interna:

<img width="1919" height="1029" alt="image" src="https://github.com/user-attachments/assets/59da0443-ba1e-4d68-a2ed-36bb67d7989d" />

- Acceso desde Red Externa:

<img width="1919" height="1032" alt="image" src="https://github.com/user-attachments/assets/972fe5e3-a320-4d0c-b1cb-a4d2b6adb78c" />

<img width="1919" height="1033" alt="image" src="https://github.com/user-attachments/assets/b94adc85-e2ea-4512-b609-48544ab91fe6" />

 
### i) Seguridad
En este apartado se ha protegido el sitio `www.web1.org` añadiendo soporte HTTPS en el puerto 443 con un certificado autofirmado.
El par de claves generado se ha guardado en `/etc/ssl/certs/web1.crt` y `/etc/ssl/private/web1.key`, y se ha usado en el bloque `server` de Nginx dedicado a este virtual host.

En la configuración del sitio se ha añadido un bloque que escucha en `listen 443 ssl` para `www.web1.org`, indicando las rutas del certificado y de la clave privada y reutilizando el mismo `root` y la misma `location /` que en el sitio HTTP.

<img width="1919" height="1031" alt="image" src="https://github.com/user-attachments/assets/5c91e616-9d6c-4218-9b16-7783bda47f83" />

Desde el cliente interno se accede a `https://www.web1.org/`; el navegador avisa de que el certificado no es de una autoridad reconocida (al ser autofirmado), se acepta la excepción y se comprueba que la página carga correctamente y que el certificado presentado tiene como CN `www.web1.org`.

<img width="1919" height="1030" alt="image" src="https://github.com/user-attachments/assets/65beb928-cb02-4114-bb61-618633ce0d24" />

## 6. Referencias
- Documentación oficial de Nginx: https://nginx.org/en/docs/
- ¿Qué es Nginx y cómo funciona? (Raiola Networks):
-  https://raiolanetworks.com/blog/nginx-vs-apache-cual-es-mejor-servidor-web/
- Guía de instalación y configuración de Nginx en Ubuntu Server 22.04:
- https://ubuntu.com/tutorials/install-and-configure-nginx
- Configuración de autenticación básica:
- https://picodotdev.github.io/blog-bitix/2020/08/configurar-autenticacion-basica-en-los-servidores-web-nginx-y-apache/#funcionamiento-y-cabecera-de-la-autenticaci%C3%B3n-b%C3%A1sica
- Configuración HTTPS:
- https://nginx.org/en/docs/http/configuring_https_servers.html
