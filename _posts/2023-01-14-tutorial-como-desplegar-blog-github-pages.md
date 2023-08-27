---
layout: post
title: Desplegar blog personal con Github Pages.
autor: S3cureroy
date: 2022-12-26
categories: [Tutoriales, Blog]
tags: [jekyll, gem, ruby, blog, markdown]
---

Llevo tiempo queriendo escribir esta entrada explicando la manera en la que despliego el blog, tanto de manera privada para realizar los artículos, como los despliegues públicos que se ven en la web.

Llevo varios años usando el lenguaje Markdown para generar mi documentación en Github, y por eso, en su momento, pensé que generar un sitio de *Github Pages* era una buena opción para desplegar mi blog.

Así que, aprovechando que esta semana cambié la máquina donde realizo todos estos pasos, los he documentado y lo subo.

Decir, que para este entorno, he utilizado una máquina Ubuntu Server 22.04.1, aunque se puede realizar sobre otras distribuciones, Windows, o incluso, Docker.

## Instalar paquetes y dependencias.

Github Pages, como explican desde su [web](https://pages.github.com/), son repositorios de Github, que pueden ser usados para mostrar páginas web.

Existen múltiples repositorios en Github que contienen la estructura y los ficheros necesarios para poder generar una web visualmente bonita, y además con funcionalidades más avanzadas que un simple fichero HTML.

Para ello, se hace uso de [Jekyll](https://jekyllrb.com/), una solución que permite generar dichas webs a través de ficheros markdown. 

Para que Jekyll pueda generar dichas páginas, es necesario instalar, en caso de que no se dispongan ya, las dependencias necesarias.

~~~
sudo apt-get install ruby-full build-essential
~~~

Comprobar el versionado de las dependencias, ya que se necesita, al menos de **ruby en versión 2.5.0** o superior.

~~~
ruby -v
~~~

~~~
gem -v
~~~

~~~
make -v
~~~

~~~
gcc -v
~~~

~~~
g++ -v
~~~

## Buscar el tema e instalarlo.

Una vez que se han instalado todas las dependencias, se puede obtener un tema ya formado desde la web [Jekyllthemes](http://jekyllthemes.org/). 

En mi caso, yo utilizo el tema [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy/) porque te permite buscar dentro de los artículos, generar categorías y etiquetas, y otras funciones que hacen que me haya decantado por él.

> Recuerda que para descargar el repositorio, es necesario tener el comando *git* instalado.
{: .prompt-info }

~~~
git clone https://github.com/cotes2020/jekyll-theme-chirpy.git
~~~

Con el repositorio ya clonado, se debe entrar dentro de la carpeta e instalar la "Gema".

## Generación de la gema de Ruby.

Para empezar a generar la gema, lo primero es, insertar las variables de entorno en el fichero ```.bashrc``` del usuario.

~~~
echo -e "# Install Ruby Gems to ~/gems\nexport GEM_HOME='$HOME/gems'\nexport PATH='$HOME/gems/bin:$PATH'" >> ~/.bashrc
source ~/.bashrc
~~~

Con las variables cargadas, se debe instalar ```jekyll``` y ```bundler``` para poder compilar la gema.

~~~
gem install jekyll bundler
~~~

Por último, se compila la gema con la configuración del tema descargado.

~~~
sudo bundle install
~~~

## Arrancar el servicio.

Con todo instalado ya, se puede arrancar el servicio para que la página sea accesible.

~~~console
jekyll serve -H 0.0.0.0

Configuration file: /home/s3cureroy/Downloads/jekyll-theme-chirpy/_config.yml
 Theme Config file: /home/s3cureroy/Downloads/jekyll-theme-chirpy/_config.yml
            Source: /home/s3cureroy/Downloads/jekyll-theme-chirpy
       Destination: /home/s3cureroy/Downloads/jekyll-theme-chirpy/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
    /home/s3cureroy/Downloads/jekyll-theme-chirpy/_sass/addon/commons.scss 844:38  @import
    jekyll-theme-chirpy.scss 17:3                                                  @import
    /home/s3cureroy/Downloads/jekyll-theme-chirpy/assets/css/style.scss 7:9        root stylesheet
                    done in 0.798 seconds.
 Auto-regeneration: enabled for '/home/s3cureroy/Downloads/jekyll-theme-chirpy'
    Server address: http://0.0.0.0:4000/
  Server running... press ctrl-c to stop.
~~~

> Si el entorno se está virtualizando, para que sea accesible desde otra máquina, como pueda ser la máquina anfitrión, se debe añadir al comando el flag ```-H 0.0.0.0```

Entonces, si se accede mediante el navegador a la dirección IP de la máquina que tenga el servicio jekyll corriendo, y al puerto 4000, se mostrará la web correctamente.

> Recuerda que para agradecer al desarrollador del tema, en lugar de clonar el repositorio directamente, se puede hacer un *fork* a un repositorio propio y desde ahí, clonarlo. 


