### Volúmenes

![volúmenes](images/types-of-mounts-volume.png)

^^^^^^

### Volúmenes

Son el mecanismo recomendado y preferido para persistir datos en los contenedores

* No incrementan el tamaño del contenedor, cosa que sí ocurre si escribimos sobre el _container layer_
* El volumen existe siempre, no depende del ciclo de vida del contenedor

^^^^^^

#### ¿Porqué volúmenes en lugar de _bind mounts_?

* Es más fácil hacer copias de respaldo y recuperarlas
* Se pueden gestionar utilizando la línea de comandos o la API de docker
* Funcionan tanto en contenedores Linux como Windows
* Los _drivers_ nos permiten almacenar la información en hosts remotos, en proveedores cloud
  e incluso encriptar el contenido.

^^^^^^

### Uso: `-v` o `--mount`

Existen dos maneras de configurar los volúmenes al crear un contenedor:

* `-v` ó `--volume`
* `--mount`

notes:

Mi experiencia es que la syntaxis de `-v` es más cómoda para configuraciones sencillas.
Cuando la configuración del volumen es más compleja, la opción --mount es más clara.

^^^^^^

### Uso: `-v` o `--mount`

```bash
> docker run -v [NOMBRE VOLUMEN]:[RUTA EN EL CONTENEDOR]:[OPCIONES]
```

```bash
> docker run --rm -v nginx-data:/var/www/ nginx
```

^^^^^^

### Uso: `-v` o `--mount`

```bash
> docker run --mount [key=value],...
```

```bash
> docker run --rm --mount 'type=volume,src=nginx-data,dst=/var/www/' nginx
```

note:

`--mount` acepta las siguientes opciones:

* `type` que puede ser `bind`, `volume`, or `tmpfs`
* `source`: para volúmenes, esta opción indica el nombre del volumen. Si no se pone nombre
  se crea un volume anónimo y es docker quién le asigna un nombre. Se puede usar `src` como alternativa.
* `destination` ruta dentro del contenedor en la que montar el volumen. Se puede usar `dst` o
  `target`.
* `readonly` una opción que, si está presente monta el _bind mount_ en modo solo lectura
* `volume-opt` opción que se puede especificar más de una vez y que permite pasarle opciones al
  driver que se esté utilizando

[Info](https://docs.docker.com/storage/volumes/#choose-the--v-or---mount-flag)

^^^^^^

### Comandos para la gestión de volúmenes

* Crear un volumen

```bash
> docker volume create [NOMBRE]
```

notes:

[Info](https://docs.docker.com/storage/volumes/#create-and-manage-volumes)


^^^^^^

### Comandos para la gestión de volúmenes

* Ver todos los volúmenes

```bash
> docker volume ls
```

^^^^^^

### Comandos para la gestión de volúmenes

* Ver los detalles de un volumen

```bash
> docker volume inspect [NOMBRE]
```

^^^^^^

### Comandos para la gestión de volúmenes

* Borrar un volumen

```bash
> docker volume rm [NOMBRE]
```

No se puede borrar un volumen que está vinculado a un contenedor

^^^^^^

### Comandos para la gestión de volúmenes

* Borrar todos los volúmenes que no se están usando

```bash
> docker volume prune
```

^^^^^^

### 💻 Práctica 1 💻 ️

Extraer las dipositivas del módulo 5 para poder reutilizarlas en otros contenedores

* Creamos el volumen `diapositivas_modulo_5`

```bash
> docker volume create diapositivas_modulo_5
```

* Creamos un nuevo contenedor a partir de la imagen del módulo 5 utilizando 
  el volumen que acabamos de crear:

```bash
> docker run -p "8005:8005" 
         -d
         --name practica_modulo_5 
         --mount "type=volume,src=diapositivas_modulo_5,dst=/home/node/slides"
         becorecode/curso-intro-docker-modulo-5 
```

notes:

Esto crea un contenedor, habilita el puerto 9005 del host para ver las diapositivas
y monta el volumen `diapositivas_modulo_5` en la carpeta `/home/node/slides`
del contenedor

Dado que el volumen `diapositivas_modulo_5` está vacío y la carpeta 
`/home/node/slides` del contenedor no, **este comando copia los ficheros y carpetas
de `/home/node/slides` al volumen**

Si entramos en el contenedor con `bash` podemos ver el volumen con el comando `mount`

^^^^^^

#### 💻 Práctica 1 (cont.) 💻 ️

* Si inspeccionamos el volumen:

```bash
> docker volume inspect diapositivas_modulo_5
[
    {
        "CreatedAt": "2019-10-13T10:48:42Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/diapositivas_modulo_5/_data",
        "Name": "diapositivas_modulo_5",
        "Options": {},
        "Scope": "local"
    }
]
```

notes:

Si estáis en una máquina Linux, podéis ver el contenido del contenedor en la carpeta
`/var/lib/docker/volumes/diapositivas_modulo_5/_data`

Podemos ver el volumen que está usando el contenedor utilizando `docker container inspect`

```bash
> docker container inspect practica_modulo_5 --format '{{json .Mounts}}'
```

^^^^^^

#### 💻 Práctica 1 (cont.) 💻 ️

* Ahora creamos un contenedor (usando la imagen de `ubuntu`) y vamos 
a ver las dapositivas desde ese otro contenedor:

```bash
> docker run --rm -ti \
         --name practica_modulo_5_segundo_contenedor
         --mount "type=volume,src=diapositivas_modulo_5,dst=/home/node/slides" \
         ubuntu \
         bash   
```

^^^^^^

#### 💻 Práctica 1 (cont.) 💻 ️

* Creamos una nueva diapositiva desde el contenedor 
  `practica_modulo_5_segundo_contenedor`

```bash
> echo "### Nueva diapositiva" > /home/node/slides/nueva_diapositiva.md
```

* Si volvemos al contenedor `practica_modulo_5` y listamos la carpeta
  `/home/node/slides/`, veremos el nuevo fichero


^^^^^^

### Volúmenes anónimos

Podemos crear un volumen anónimo de la siguiente manera:

```backup
> docker run --rm -v /test -v data:/data ubuntu top
```

* Se crea un volumen anónimo y se monta en `/test` dentro del contenedor
* Se crea un volumen `data` y se monta en la carpeta `/data` dentro del contenedor

notes:

Al parar el contenedor, la opción `--rm` hace que se borre tanto el contenedor
como el volumen anónimo. El volumen `data` no se borra y podremos verlo
usando el comando `doker volume ls`

Si entramos en el contenedor con `bash` podemos ver el volumen con el comando `mount`

^^^^^^

### 💻 ️Práctica 2 💻


notes:

Esta práctica está insipirada en esta 
[sección de la documentación de docker](https://docs.docker.com/storage/volumes/#backup-a-container)