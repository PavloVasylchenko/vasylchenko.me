---
title: "Primeros pasos con Hugo"
date: 2020-10-19T19:00:00Z
categories:
  - "Manuals"
draft: false
---

Cómo crear un blog con Hugo.
En los últimos años, los sitios estáticos se han vuelto muy populares.
Probablemente se deba a que son bastante fáciles de mantener y existen muchas plataformas donde publicarlos.
Si antes era necesario trabajar directamente con HTML para crear un sitio estático, hoy basta con conocer Markdown para resolver la mayoría de las tareas.
Los sitios creados con Hugo son una gran opción para páginas personales cuyo contenido no necesita actualizarse con frecuencia.
Hugo es un conjunto de herramientas que permite generar un sitio estático a partir de archivos Markdown y plantillas.
También resulta muy útil contar con imágenes Docker preparadas, ya que permiten crear y compilar el sitio sin instalar Go, Hugo ni otros componentes en el equipo.

Empecemos.
Si utilizas Linux y tienes Docker instalado ([docker.com](https://www.docker.com)), solo necesitas seguir unos pocos pasos.
Crea un sitio vacío en el directorio `vasylchenko.me` con este comando:

```bash
docker run --rm -it -v $(pwd)/:/src klakegg/hugo:ext new site vasylchenko.me
```

Después, entra en la carpeta con `cd vasylchenko.me`.
A continuación, inicializa un repositorio Git para controlar las versiones:

```bash
git init
```

Ha llegado el momento de elegir un tema.
Puedes consultar una lista de temas populares en [themes.gohugo.io](https://themes.gohugo.io).
Para este sitio, por ejemplo, elegí el tema [Devise](https://themes.gohugo.io/devise/).
Abre la página del tema y ejecuta el siguiente comando:

```bash
git submodule add https://github.com/austingebauer/devise themes/devise
```

El comando añadirá el repositorio del tema a la carpeta `themes` como un submódulo de Git.

En este punto tenemos el directorio del sitio `vasylchenko.me` y el del tema `vasylchenko.me/themes/devise`.
Ahora debemos indicar en la configuración de Hugo qué tema queremos utilizar.
Puedes editar el archivo a mano o ejecutar:

```bash
echo 'theme = "devise"' >> config.toml
```

Por último, genera el sitio y comprueba el resultado:

```bash
docker run --rm -it -v $(pwd):/src klakegg/hugo:ext
```

Cuando termine la compilación, entra en `vasylchenko.me/public` y abre `index.html` en el navegador.
Si el sitio se ve como esperabas, ya puedes subir el contenido de ese directorio a un servicio de alojamiento estático.
