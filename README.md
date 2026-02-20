# Práctica 3.4: «Dockerizar» una web estática y publicarla en Docker Hub

## Índice
1. [Introducción](#introducción)
2. [Crear el archivo Dockerfile](#1-crear-el-archivo-dockerfile)
3. [Construir la imagen Docker](#2-construir-la-imagen-docker)
4. [Publicar la imagen en Docker Hub](#3-publicar-la-imagen-en-docker-hub)
5. [Publicar automáticamente con GitHub Actions](#4-publicar-automáticamente-con-github-actions)
6. [Despliegue en AWS EC2 con Docker Compose](#5-despliegue-en-aws-ec2-con-docker-compose)

---

## Introducción

En esta práctica se crea una imagen Docker que contiene el servicio Nginx sirviendo la aplicación web estática [2048](https://github.com/josejuansanchez/2048). La imagen se publica en Docker Hub y se despliega en una instancia de Amazon EC2 usando Docker Compose. Además, se automatiza la publicación de la imagen con GitHub Actions.

---

## 1. Crear el archivo Dockerfile

Se crea un archivo `Dockerfile` con los siguientes requisitos:

- Imagen base: última versión de `ubuntu`
- Se instala `nginx` y `git`
- Se clona el repositorio de la aplicación en `/var/www/html/`
- Se expone el puerto `80`
- El comando de inicio es `nginx -g daemon off;`

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y \
    nginx \
    git \
    && rm -rf /var/lib/apt/lists/*

RUN rm -rf /var/www/html/* && \
    git clone https://github.com/josejuansanchez/2048 /var/www/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

---

## 2. Construir la imagen Docker

Desde el directorio donde se encuentra el `Dockerfile`, se ejecuta:

```bash
docker build -t nginx-2048 .
```

Se comprueba que la imagen se ha creado correctamente:

```bash
docker images
```

Se asigna el nombre correcto con el usuario de Docker Hub y las etiquetas `1.0` y `latest`:

```bash
docker tag nginx-2048 juangg04/nginx-2048:1.0
docker tag nginx-2048 juangg04/nginx-2048:latest
```

Se verifica que las etiquetas son correctas:

```bash
docker images
```

---

## 3. Publicar la imagen en Docker Hub

Se inicia sesión en Docker Hub usando un token de acceso personal:

```bash
docker login -u juangg04
```

Se publica la imagen con ambas etiquetas:

```bash
docker push juangg04/nginx-2048:1.0
docker push juangg04/nginx-2048:latest
```

La imagen queda disponible en: `https://hub.docker.com/r/juangg04/nginx-2048`

---

## 4. Publicar automáticamente con GitHub Actions

Se configura GitHub Actions para publicar la imagen automáticamente en Docker Hub cada vez que se hace un `push` a la rama `main`.

### 4.1 Crear los Secrets en GitHub

En el repositorio de GitHub, se van a **Settings → Secrets and variables → Actions** y se crean dos secrets:

- `DOCKERHUB_USERNAME`: nombre de usuario de Docker Hub (`juangg04`)
- `DOCKERHUB_TOKEN`: token de acceso generado en **Docker Hub → Account Settings → Security → New Access Token**

### 4.2 Archivo del workflow

Se crea el archivo `.github/workflows/docker-publish.yml`:

```yaml
name: Publish Docker image

on:
  push:
    branches: ["main"]

jobs:
  push_to_registry:
    name: Push Docker image to Docker Hub
    runs-on: ubuntu-latest

    steps:
      - name: Check out the repo
        uses: actions/checkout@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            ${{ secrets.DOCKERHUB_USERNAME }}/nginx-2048:latest
            ${{ secrets.DOCKERHUB_USERNAME }}/nginx-2048:1.0
```

Cada vez que se hace `git push` a `main`, el workflow construye y publica la imagen automáticamente en Docker Hub.

---

## 5. Despliegue en AWS EC2 con Docker Compose

### 5.1 Crear la instancia EC2

Se crea una instancia EC2 en AWS con Ubuntu. En el Security Group se abre el puerto `80` para tráfico HTTP.

### 5.2 Instalar Docker y Docker Compose

```bash
sudo apt update && sudo apt install -y docker.io docker-compose
sudo usermod -aG docker ubuntu
```

### 5.3 Archivo docker-compose.yml

```yaml
services:
  web:
    image: juangg04/nginx-2048:latest
    ports:
      - "80:80"
```

### 5.4 Lanzar el contenedor

```bash
docker compose up -d
```

### 5.5 Verificar el acceso

Se obtiene la IP pública de la instancia desde la consola de AWS y se accede desde el navegador:

```
http://<IP_PUBLICA>
```

La aplicación 2048 queda accesible públicamente desde el navegador.

---

## Referencias

- [Repositorio de la aplicación 2048](https://github.com/josejuansanchez/2048)
- [Ejemplo con GitHub Actions](https://github.com/josejuansanchez/2048-github-actions)
- [Documentación oficial de GitHub Actions - Publicar imágenes Docker](https://docs.github.com/es/actions/use-cases-and-examples/publishing-packages/publishing-docker-images)
- [Docker Hub](https://hub.docker.com)
<img width="1865" height="955" alt="image" src="https://github.com/user-attachments/assets/6a01f755-9cbe-48a2-915c-89b4d9c17b93" />
<img width="1083" height="824" alt="image" src="https://github.com/user-attachments/assets/e3549e69-079e-4ef9-adb7-226ca2fd724a" />
