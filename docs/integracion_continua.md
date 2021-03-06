# Integración continua y herramientas de construcción

## Entidad "Acontecimiento"

Para esta entidad, primero se ha configurado la integración continua con Travis a través del archivo `.travis.yml`:

```
language: ruby

rvm:
 - 2.6.5
 - 2.7
             
before_install:
 - cd acontecimiento
 - gem install rake
 - rake build --trace=stdout
             
script:
 - rake test --trace=stdout
```

En este archivo, se puede observar que para esta entidad se va a usar el lenguaje de programación Ruby, en concreto, se
van a usar las versiones 2.6.5 (la última versión estable) y la versión 2.7 (que es ahora mismo la versión de 
desarrollo) para ejecutar los tests.

Luego, antes de la ejecución de los tests, se va a proceder a la instalación de las herramientas necesarias para la 
ejecución, pues bien, para empezar, se va a instalar la gema `rake`, una herramienta necesaria para ejecutar la tarea 
encargada de la construcción del entorno, con el comando `rake build --trace=stdout`; aquí, en este comando, se ha 
especificado la opción `--trace=stdout` para imprimir el resultado de la ejecución de los tests en la salida estándar;
pero antes, hay que dirigirse al directorio `acontecimiento`, en el cual se ubica el código asociado a esta entidad. 
Finalmente, se dirige al archivo `Rakefile` y lanza la tarea encargada de ejecutar los tests, con el comando
`rake test --trace=stdout`.

Lo siguiente que se ha hecho es configurar la herramienta de construcción, en el archivo `Rakefile`, en la cual se ha
agregado una tarea, llamada `test`, que se encarga de ejecutar los tests del microservicio con Rspec. También, se ha
creado otra tarea `build`, la cual se encarga de instalar la gema `bundle`, que va a permitir instalar el resto de gemas 
especificadas en el archivo `Gemfile`, por último, se procede a instalar el resto de paquetes con el comando 
`bundle install`; aquí, ambos comandos han sido especificados como comandos de shell (sh).

> buildtool: acontecimiento/Rakefile

```
task :test do
  sh "rspec tests/tests.rb"
end

task :build do
  sh "gem install bundle; bundle install"
end
```

## Entidad "Días no laborables"

Aquí, primero se ha configurado la integración continua, usando CircleCI, en el archivo `.circleci/config.yml`:

```
version: 2
jobs:
  test-3.6: &test
    docker:
      - image: circleci/python:3.6

    working_directory: ~/repo/diasnolaborables

    steps:
      - checkout:
          path: ~/repo

      - run:
          name: Instalar entorno de ejecución
          command: |
            python3 -m venv venv
            . venv/bin/activate
            pip3 install invoke
            invoke build

      - run:
          name: Ejecutar los tests
          command: |
            . venv/bin/activate 
            invoke test
  
  test-3.7:
    <<: *test
    docker:
      - image: circleci/python:3.7

  test-3.8-desarrollo:
    <<: *test
    docker:
      - image: circleci/python:3.8.0b3

workflows:
  version: 2
  build-and-test:
    jobs:
      - test-3.6
      - test-3.7
      - test-3.8-desarrollo
```

En este archivo, se puede observar que se han creado 3 trabajos que ejecutan los mismos comandos, tanto para construir
el entorno como para ejecutar los tests, diferenciándose únicamente en la versión de Python que se usa; en este caso, 
se ha optado por usar las últimas versiones estables del lenguaje de programación Python (3.6 y 3.7), añadiéndose 
también una versión de desarrollo `beta` de Python 3.8. 

En cada trabajo, lo que se hace primero es obtener una imagen de Docker específica, para ejecutar los tests en CircleCI
con la versión correspondiente de Python; luego, establecemos el directorio `diasnolaborables` como directorio de trabajo,
ya que es ahí donde se ubica el código de la entidad; después, hacemos un `checkout` al directorio que contiene el código 
del repositorio; posteriormente, se ejecutan los comandos asociados a la instalación del entorno de ejecución de los 
tests, entre los cuales se incluye la creación del entorno virtual de Python, la instalación de la herramienta de 
construcción `invoke` y, la ejecución, con esta herramienta, de las tareas asociadas a eliminar los archivos de caché 
de Python, con extensión `*.pyc` (clean) y, a instalar los paquetes necesarios para la ejecución de estos tests (build); 
finalmente, se ejecutan los tests con la tarea test de `invoke`, activando antes, para ello, el entorno virtual. 

Por último, se puede contemplar, que para el primer trabajo se ha creado un `anchor`, que va a ser extendido en los
otros 2 trabajos para así replicar los pasos a ejecutar, cambiando únicamente la versión de Python para ejecutar los tests.

Una vez se tiene la integración continua, lo único que quedaría es configurar la herramienta de construcción, para 
ello, se ha utilizado la herramienta `invoke` y se ha creado el siguiente archivo `tasks.py`:

> buildtool: diasnolaborables/tasks.py

```
from invoke import task

@task
def clean(c):
    c.run("rm -rf **/*.pyc")

@task
def build(c):
    c.run("pip3 install -r requirements.txt")

@task
def test(c):
    c.run("pytest tests/*")
```

En él, se han creado 3 tareas usando la herramienta `invoke`, una primera, llamada `clean`, para eliminar los archivos 
de caché de Python, otra, llamada `build`, para instalar los paquetes necesarios, especificados en el archivo de
requerimientos de Python y, una última, llamada `test`, que ejecuta los tests del microservicio con `pytest`.