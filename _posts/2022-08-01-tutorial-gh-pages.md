---
title: Tutorial Github Pages + Jekyll + Chirpy
date: 2022-08-01 17:10:00 +0800
categories: [Tutos]
tags: ["github pages", tutos]     # TAG names should always be lowercase
---

## Intro.

Buscando un espacio donde publicar manuales de cosecha propia, writeups en español y cosas por el estilo, se me ocurrió echar un ojo a [GitHub Pages](https://pages.github.com/). Si bien había oído hablar de este servicio (llevo tiempo trabajando con [GitHub](https://github.com/) como repositorio de código) nunca le había prestado demasiada atención. 

El servicio lo presentan como algo sencillo de montar y mantener, sin coste, basado en git, sin el poder de un CMS pero también sin sus complicaciones ... en resumen: un recurso ideal para disponer de una página personal en la que compartir contenido de un modo digno y asequible.

Tras un par de pruebas con los temas que ofrece el servicio por defecto me decidí por montar mi repo personal con [Jekyll](https://jekyllrb.com/) y [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy), el tema que más me gustó de los que hay disponibles [aquí](http://jekyllthemes.org/).

A continuación os dejo una guía paso a paso para aquellos que quieran intentarlo.

> **Nota importante:** el contenido de este post puede resultar demasiado básico a quienes cuenten con cierta experiencia en ámbitos como Git, Ruby o Linux, pero su objetivo es justamente ese: servir de guía a quienes se embarquen por primera vez en estas lides. Por otra parte, y para quienes quieran profundizar, dejo enlaces a la documentación original en cada apartado.

## Algunas generalidades antes de empezar ...

Como en todo proyecto basado en **Github** tenemos que tener claro los dos entornos en los que vamos a movernos:

* Por una lado **Github** como tal, al que accedemos via el navegador. Doy por hecho que disponéis de una cuenta, de no ser así, el primer paso es [registrarse](https://github.com/signup). Por simplicidad, de aquí en adelante me voy a referir a Github con la abreviatura **GH**.

* Por otro lado nuestro **entorno de desarrollo local**, disponible en nuestra propia máquina y en el que realizaremos los cambios que luego publicaremos en **GH**. En mi caso utilizo un ordenador con Kali, una distribución de Linux basada en Debian. Por simplicidad, de aquí en adelante me voy a referir a nuestra máquina como **local**. 


## Configuración de nuestro sitio en GH.

En este paso crearemos nuestro nuevo sitio en **GH** utilizando el tema [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy). Para ello ...

1. Iniciamos nuestro navegador web y nos logueamos en **GH**.

2. En la misma pestaña (o en una nueva, eso va en gustos) cargamos la siguiente URL para *clonar* el repositorio de **Chirpy** ...
```
https://github.com/cotes2020/chirpy-starter/generate
```

3. Se nos muestra la siguiente vista:
![](/assets/posts/20220801/img01.png)

4. Completamos el input ***Repository name*** con el valor: `<mi_usuarioGH>.github.io` donde `<mi_usuarioGH>` es vuestro usuario de **GH** tal como aparece en el desplegable ***Owner*** que esta a la izquierda. Es fundamental que lo escribáis tal cual.

5. Nos aseguramos que el checkbox ***Public*** este seleccionado.

6. Damos al boton **Create repository from template**.

> **NOTA:** Para mas información sobre este paso podéis visitar la página oficial de **Chirpy** [aquí](https://chirpy.cotes.page/posts/getting-started/#creating-a-new-site). En nuestro caso hemos utilizado la opcion 1.

Con esto, por el momento, hemos terminado con la parte de **GH** vía web.

## Configuración de nuestro entorno local.

Para utilizar **Jekyll** en nuestro ordenador necesitamos tener instalados los siguiente paquetes:

* **Ruby** (lo comprobamos ejecutando en nuestro terminal `$ ruby -v`)

* **RubyGems** (lo comprobamos ejecutando en nuestro terminal `$ gem -v`)

* **GCC** (lo comprobamos ejecutando en nuestro terminal `$ gcc -v` y `$ g++ -v`)

* **Make** (lo comprobamos ejecutando en nuestro terminal `$ make -v`)

En caso de necesidad, podemos instalarlos ejecutando desde nuestro terminal:

```console
$ sudo apt install ruby-full
```

... para la parte de Ruby, y ...

```console
$ sudo apt install build-essential
```

para el resto de paquetes requeridos.

> **NOTA:** Para mas información sobre los requisitos de **Jekyll** en otros sistemas operativos podéis ver la documentación oficial [aquí](https://jekyllrb.com/docs/installation/other-linux/).

Comprobados los prerequisitos de **Jekyll** pasamos a instalarlo. Esto lo haremos via `gem`, como el usuario ***root***, ejecutando el siguiente comando desde nuestro terminal:

```console
# gem install jekyll bundler
```

> **Nota:** la instalación es posible que os pida un par de veces vuestra contraseña de usuario local.

Una vez tenemos **jekyll** y **bundler** instalados en nuestro equipo, salimos de ***root*** [`Ctrl + d`] y procedemos a crear la carpeta donde alojaremos nuestro repositorio de **GH**. Para ello ejecutamos desde la ubicación que deseemos (nuestro directorio *home* por ejemplo) ...

```console
$ mkdir <mi_usuarioGH>
$ cd <mi_usuarioGH>
```
Y desde alli nos traemos nuestro repositorio de **GH** con ...

```console
$ git clone https://github.com/<mi_usuarioGH>/<mi_usuarioGH>.github.io.gi
```

Una vez finalizado el proceso, nos movemos al directorio que nos hemos descargado ...

```console
$ cd <mi_usuarioGH>.github.io.gi
```

Ejecuntando un `ls` en este directorio deberíamos ver algo asi ...

```
┌──(ewan㉿kali)-[~/<mi_usuarioGH>/<mi_usuarioGH>.github.io]
└─$ ls
Gemfile  Gemfile.lock  LICENSE  README.md  _config.yml  _data  _plugins  _posts  _tabs  assets  index.html  tools
```

Bien, ahora vamos a editar algunos ficheros de este directorio antes de generar nuestra sitio con `bundle`. Vamos a ello ...

> **Nota:** un par de líneas más abajo os explico esto de *generar* nuestro sitio y la mecánica que hay en juego.

El primer fichero que vamos a editar es el <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">_config.yml</code> que se encuentra en nuestro directorio <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">~/&lt;mi_usuarioGH&gt;/&lt;mi_usuarioGH&gt;.github.io</code> (aka nuestro directorio raíz del proyecto si lo habéis creado en el directorio *home* de vuestro equipo).

Lo abrimos en nuestro editor preferido, le echamos un ojo, y modificamos los siguientes valores:

* `baseurl: ''`&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;en nuestro caso vamos a dejarlo vacío, tal como viene.

* <p><code class="language-plaintext highlighter-rouge">lang: <span style="color:var(--filepath-text-color);">es-ES</span></code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;para que nuestro site se vea en español.</p>

> **Nota:** podéis ver los idiomas disponibles en la carpeta <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">&lt;mi_usuarioGH&gt;.github.io/_data/locales/</code>.

* <p><code class="language-plaintext highlighter-rouge">timezone: <span style="color:var(--filepath-text-color);">Europe/London</span></code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;aquí poned lo que os corresponda.</p>

* <p><code class="language-plaintext highlighter-rouge">title: <span style="color:var(--filepath-text-color);">&lt;mi_usuarioGH&gt;</span></code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;podéis poner como título del sitio lo que os apetezca, en mi caso he optado por mi nombre de usuario de <strong>GH</strong>. Este es el texto que va a aparecer, entre otros lugares, debajo de vuestro avatar.</p>

* <p><code class="language-plaintext highlighter-rouge">tagline: <span style="color:var(--filepath-text-color);">Lo que quieras</span></code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;este es el texto que va a aparecer en el site justo debajo del <code class="language-plaintext highlighter-rouge">title</code>.</p>

* <p><code class="language-plaintext highlighter-rouge">url: <span style="color:var(--filepath-text-color);">'https://&lt;mi_usuarioGH&gt;.github.io'</span></code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;muy importante este parámetro, completarlo con vuestro nombre de usuario de <strong>GH</strong> como hemos venido haciendo en pasos anteriores.</p>

* Vuestros usuarios en redes (pego pantallazo, reemplazar lo que corresponda) ...

![](/assets/posts/20220801/img02.png)

Que son valores son los que van a aparecer asociados a los iconos ubicados al final del lateral izquierdo de la página. 

* <p><code class="language-plaintext highlighter-rouge">avatar: <span style="color:var(--filepath-text-color);">/assets/common/<em>&lt;mi_avatar.jpg|png&gt;</em></span></code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;esta es la imagen que va a aparecer en el lateral izquierdo. Tened en cuenta almacenarla en la carpeta <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">~/&lt;mi_usuarioGH&gt;.github.io/assets/common/</code>.</p>

> **Nota:** deberéis crear la carpeta <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/common</code>

El resto de valores los dejamos tal cual. Guardamos los cambios.

Volvemos al terminal, nos posicionamos en nuestro directorio raíz <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">~/&lt;mi_usuarioGH&gt;/&lt;mi_usuarioGH&gt;.github.io</code> y ejecutamos:

```console
$ bundle
```

Con este comando "compilamos" nuestro sitio en local. Si echáis un ojo a las carpetas que cuelgan del directorio raíz veréis que se ha creado una nueva llamada <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">_site</code> en la que están los ficheros que forman nuestro nuevo sitio web.

Ejecutamos

```console
$ bundle exec jekyll s
```

para iniciar el servidor que trae Jekyll y poder navegar por nuestro sitio en local.

Abrimos una pestaña en el navegador y visitamos

```
http://localhost:4000
```

Si todo ha ido ok, estaremos viendo nuestro flamante sitio web basado en Jekyll y con el tema Chirpy.

Cada vez que ejecutemos `bundle` recrearemos nuestro sitio a partir de los ficheros que tengamos en nuestro directorio raíz. Si hemos iniciado el servidor y hacemos algún cambio, el proceso regenera nuestro site de manera automática, sin necesidad de pararlo y volverlo a iniciar.

> **Nota:** para parar el servidor de jekyll ejecutamos `Ctrl + d`

## Nuestro primer post.

Ahora vamos a crear nuestro primer post. Va a ser algo sencillo y no voy a entrar en muchos detalles al respecto. Apenas cuente con algo de tiempo tengo idea de publicar una segunda parte de esta guía con algo más de información. Los mas ansiosos podéis echar un ojo a la documentación oficial disponible pinchando en los links que apunté mas arriba.

En nuestro editor de texto preferido creamos un fichero nuevo y lo guardamos en la carpeta <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/_posts</code> de nuestro directorio raíz (es decir, en: <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">~/&lt;mi_usuarioGH&gt;/&lt;mi_usuarioGH&gt;.github.io/_posts/</code>) con el siguiente formato de nombre <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">YYYY-MM-DD-un-nombre.md</code>, por ejemplo: 

```
2022-08-01-hola-mundo.md
```
En el fichero copiamos el siguiente código:

```
---
title: Hola mundo_
date: 2022-08-01 14:10:00 +0800
categories: [Tutos]
tags: ["hola mundo"]     # TAG names should always be lowercase
---

Mi primer post en GH ;)
```

Guardamos y volvemos al navegador donde tenemos abierto nuestro sitio. Refrescamos y debería aparecernos nuestra nueva entrada.

> De no ver los cambios, id al terminal, cerrar la sesion del server con `Ctrl + d`, volved a ejecutar `$ bundle exec jekyll s`&nbsp;&nbsp;, recargar el site desde la ventana del navegador. Si, es cierto, a veces el tema del regenerado automático se queda tieso :(

## Subir todo a Github.

La mecánica aquí es la habitual para subir cosas a Github por lo que iré algo rápido, centrándome solo en un detalle particular que debemos considerar relacionado con Github Pages.

Tened en cuenta que para ejecutar el `push`&nbsp;&nbsp;nos va a solicitar vuestra **Personal Access Token** de Github. Si no disponéis de ella, [aqui](https://docs.github.com/es/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token#creating-a-token) tenéis la documentación oficial de Github para crearla.

Desde nuestro terminal, posicionados en nuestro directorio raíz, ejecutamos:

```
┌──(ewan㉿kali)-[~/<mi_usuarioGH>/<mi_usuarioGH>.github.io]
└─$ git init
Reinitialized existing Git repository in /home/ewan/ewan67/ewan67.github.io/.git/

┌──(ewan㉿kali)-[~/<mi_usuarioGH>/<mi_usuarioGH>.github.io]
└─$ git add .

┌──(ewan㉿kali)-[~/<mi_usuarioGH>/<mi_usuarioGH>.github.io]
└─$ git commit -m "primer commit"   
[main 2cdb01b] primer commit
... nos lista los ficheros que se van a publicar ...

┌──(ewan㉿kali)-[~/<mi_usuarioGH>/<mi_usuarioGH>.github.io]
└─$ git push
Username for 'https://github.com': <mi_usuarioGH>
Password for 'https://Ewan67@github.com': <nuestro Personal Access Token>
... inicia el proceso ...
```

Ahora nos vamos a nuestra pestaña del navegador donde tenemos abierta la web de Github, accedemos a nuestro repositorio <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">&lt;mi_usuarioGH&gt;.github.io</code>&nbsp;&nbsp;y pinchamos en ***Commit*** para ver el progreso de nuestro envío.

Una vez finalizado (según el número de cambios, el estado de la red, etc; el proceso puede tardar unos minutos, sed pacientes) veremos que se nos ha creado una nueva rama (branch) en nuestro repo llamada <code class="language-plaintext highlighter-rouge" style="color:#3361ff;">gh-pages</code>. Toca hacer la última configuración que mencionaba mas arriba para dejar todo ok.

Desde la *home* de nuestro repo cuya URL será algo como

```
https://github.com/<mi_usuarioGH>/<mi_usuarioGH>.github.io)
```

1. Damos a la opción ***Settings*** del menú horizontal.

2. Damos a la opción ***Pages*** del menú lateral izquierdo.

3. En el apartado ***Build and deployment***&nbsp;&nbsp;>&nbsp;&nbsp;***Branch*** configuramos lo siguiente:
![](/assets/posts/20220801/img04.png)

4. Le damos al boton ***Save***.

Y listo !!! ... Ya podemos crear y modificar el contenido de nuestro site en local y publicar los cambios a nuestro repo de Github Pages.

Hay varias posibilidades mas que no he mencionado en esta guía en aras de la brevedad. Los mas curiosos podéis brujulear por la red donde encontraréis contenidos muy interesantes sobre las posibilidades de Jekyll y GH.

Aún así espero que os haya valido este tuto al menos como introducción. Cualquier duda o comentario podéis escribirme a mi correo, intentaré responderos ASAP.

Sed buenos si no hay una opción mejor.














