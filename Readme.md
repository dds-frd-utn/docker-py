# Imágenes en Docker

Crear una imagen a partir del proyecto

Desde la consola, ingresar al directorio donde están los 3 archivos del proyecto:

- app.py
- Dockerfile
- requirements.txt

Es posible que para crear una imagen sea necesario primero descargar la imagen de base (si da error la primera vez, probar con esto). Esto se puede realizar simplemente levantando el contenedor o realizando el pull de la imagen:

```
> docker pull python:3.4-alpine
3.4-alpine: Pulling from library/python
8e402f1a9c57: Already exists
cda9ba2397ef: Already exists
aafecf9bbbfd: Already exists
bc2e7e266629: Already exists
e1977129b756: Already exists
Digest: sha256:c210b660e2ea553a7afa23b41a6ed112f85dbce25cbcb567c75dfe05342a4c4b
Status: Downloaded newer image for python:3.4-alpine
docker.io/library/python:3.4-alpine
```


Luego, creamos la imagen de nuestro sistema ejecutando el comando:

```
> docker build -t pydds .
```

donde **pydds** será el nombre de la imagen.

```
> docker build -t pydds .    
[+] Building 16.8s (9/9) FINISHED
 => [internal] load build definition from Dockerfile                                                                                    0.1s 
 => => transferring dockerfile: 32B                                                                                                     0.0s 
 => [internal] load .dockerignore                                                                                                       0.1s 
 => => transferring context: 2B                                                                                                         0.0s 
 => [internal] load metadata for docker.io/library/python:3.4-alpine                                                                    0.0s 
 => CACHED [1/4] FROM docker.io/library/python:3.4-alpine                                                                               0.0s 
 => [internal] load build context                                                                                                       0.1s 
 => => transferring context: 1.35kB                                                                                                     0.0s 
 => [2/4] WORKDIR /app                                                                                                                  0.2s 
 => [3/4] ADD . /app                                                                                                                    0.2s 
 => [4/4] RUN pip install --trusted-host pypi.python.org -r requirements.txt                                                           15.4s 
 => exporting to image                                                                                                                  0.6s 
 => => exporting layers                                                                                                                 0.5s 
 => => writing image sha256:5c03900f6bf77c943691fb77f216cd4b4fa2f1a8cd7acdd765cc6f3e26fd4c6a                                            0.0s 
 => => naming to docker.io/library/pydds                                                                                                0.0s 

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
```

Ahora revisamos las imágenes locales y veremos que aparece la creada recientemente:

```
> docker image ls
REPOSITORY                               TAG          IMAGE ID       CREATED         SIZE
pydds                                    latest       5c03900f6bf7   3 minutes ago   84.6MB
```

Ahora podemos levantar la aplicación desde la imagen creada corriendo:

```
docker run -p 80:5000 pydds
```

Si accedemos a `http://localhost` del navegador podremos ver la aplicación corriendo. No requiere poner el puerto porque toma el default de http que es el 80 y cuando levantamos el docker estamos diciendo que el puerto interno del docker (el 5000 que levanta Flask) lo redirija al 80 en la salida del contenedor.

# Docker Hub

Antes de continuar es necesario crear una cuenta en `https://hub.docker.com` 

Una vez creada, desde nuestra terminal, vamos a loguearnos. Para ello debemos escribir el comando:

```
> docker login
```

Allí se nos pedirá las credenciales y quedaremos autenticados en nuestra consola para poder subir nuestras imágenes.

Ahora vamos a crear un TAG para nuestra imagen utilizando la forma docker `tag image username/repository:tag`

En el caso de este tutorial sería:

```
docker tag pydds utnfrd/pydds:1.0.0
```

Podemos verificar si la imagen es correcta (ver que ambas imagenes tienen el mismo ID, pero con diferente descripción):

```
> docker image ls
REPOSITORY                               TAG          IMAGE ID       CREATED          SIZE
pydds                                    latest       6b91ceba0fbc   4 hours ago      84.6MB
utnfrd/pydds                             1.0.0        6b91ceba0fbc   4 hours ago      84.6MB
```

Y ahora podemos proceder a subirla a docker hub realizando un push:

```
> docker push utnfrd/pydds:1.0.0   
The push refers to repository [docker.io/utnfrd/pydds]
59ee1ffd2502: Pushed
89f45c3664ec: Pushed
11d8f3bde487: Pushed
62de8bcc470a: Mounted from library/python
58026b9b6bf1: Mounted from library/python
fbe16fc07f0d: Mounted from library/python
aabe8fddede5: Mounted from library/python
bcf2f368fe23: Mounted from library/python
1: digest: sha256:c083833f20e050fe6f0eaf5b154764881de9a723a569b8e78b42d7bbe16dc5b4 size: 1993
```

Se puede verificar en `https://hub.docker.com` que la imagen se cargó correctamente navegando hasta la sección de repositorios.

Ahora podemos levantar un docker desde la imagen que acabamos de subir:

```
> docker run -p 80:5000 utnfrd/pydds:1.0.0
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 907-259-245
```

Como vemos, la misma se generó desde la imagen local, pero si eliminamos la imagen local lo va a tomar de docker hub.

Para tomar la imagen desde docker hub debemos eliminar la imagen local. Para esto, paramos y eliminamos los contenedores creados a partir de esa imagen:

```
> docker ps -a
Obtener los ID de los contenedores

> docker rm 892795b1
```

Ahora podemos eliminar la imagen del contenedor:

```
> docker image ls
```

Para obtener el ID de la imagen y luego:

```
> docker image rm 6b91ceba0fbc
Untagged: pydds:latest
Deleted: sha256:6b91ceba0fbcf35c454ab350b6ca52f33526cb1b19e782b04b0ad62085907222
```

Ahora, si volvemos a correr el comando para levantar el contenedor:

```
> docker run -p 80:5000 utnfrd/pydds:1.0.0  
Unable to find image 'utnfrd/pydds:1.0.0' locally
1: Pulling from utnfrd/pydds
8e402f1a9c57: Already exists
cda9ba2397ef: Already exists
aafecf9bbbfd: Already exists
bc2e7e266629: Already exists
e1977129b756: Already exists
8e3a833e3507: Already exists
3d933c19e5c0: Already exists
84ce3c47777a: Already exists
Digest: sha256:c083833f20e050fe6f0eaf5b154764881de9a723a569b8e78b42d7bbe16dc5b4
Status: Downloaded newer image for utnfrd/pydds:1.0.0
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 720-590-725
```

En la primer linea, se puede ver que realizó un pull desde docker hub.
