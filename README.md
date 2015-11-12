
Docker Intro
===============

View, Share, Fork!

Slides: <http://kim0.github.io/docker-intro/>

Source <https://github.com/kim0/docker-intro>

![Twitter](https://g.twimg.com/dev/documentation/image/Twitter_logo_blue_48.png) @ak_kim0

Contents:
----------
1. __Intro__
1. __Command line interface__
1. __Dockerfiles__
1. __docker-compose__

Lies, lies everywhere
======================

Lies upon lies?
-----------------------

* CPU Hardware
* ISA / Virtualization
* Linux Kernel ABI
* Cloud

Not really new!
---------------

![FreeBSD Jails](http://negativo17.org/site/wp-content/uploads/2013/09/freebsd.png)     ![Solaris Zones](http://www.operating-system.org/betriebssystem/gfx/logo/solaris_101-logo.jpg)

Problem
=======

What problem does docker solve
------------------------------

* Containers for "real" shipping
* Software people "ship" things too!

![](http://www.pacecargo.com/wp-content/uploads/2014/05/CONTAINERS.4jpg.jpg)

What Docker is
================

Key concepts
----------------
* Defined environment for launching processes
* Clean separation of environments
* Sandbox with defined resources
* One simple interface for launching applications

What Docker is not!
--------------------
* Not a virtualisation
* No seperate kernel
* No hypervisor

Installation
===============
Docker is linux based, but there exist convienent solutions to work on Windows and Mac OS X also.

Detailed Instructions: <https://docs.docker.com/installation/>

The simple way for installation in Ubuntu and Debian:

    curl https://get.docker.com/ | sh
      
The docker hello world
=========================

    docker run busybox echo 'Hello World'

What just happened?

* Download the image `busybox`
* Create a new container
* Execute the `echo` command within the new container
          
Images and Containers
==========================

Docker Image
---------------

* A template for containers
* Can be pulled and pushed towards a registry
* Image names have the form `[registry/][user/]name[:tag]`
* The default for the tag is `latest`

Docker Container
---------------

* An instance of an image
* Maintains changes within the filesystems
* Can be started, stopped, restarted, ..
      
      
Image handling
==============================

search, pull & push
----------------------

searching in the registry:

    docker search <term>

download or update an image from the registry:

    docker pull <image>

upload an image to the registry:

    docker push <image>

Image handling
==============================

list, tag & delete
----------------------

listing of downloaded images:

    docker images

give an image a new name (alias):

    docker tag <oldname> <newname>

delete an image locally:

    docker rmi <image>

Docker run
===============

Start a new container

    docker run <imagename>

My favorite run options:
    
    Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
     --name             Give the container a symbolic name
     -v, --volume=[]    Bind mount a volume
     -p, --publish=[]   Publish a container's port(s) to the host
     -e, --env=[]       Set environment variables
     --link=[]          Add link to another container
     --restart="no"     Restart policy (no, on-failure[:max-retry], always)
     --rm=false         Automatically remove the container when it exits
     -d, --detach=false Run container in background and print container ID
     -i, --interactive=false   Keep STDIN open even if not attached
     -t, --tty=false    Allocate a pseudo-TTY

See your containers:
========================

List containers
-----------------
The running containers:

    docker ps

All containers:

    docker ps -a

Show Metadata of a container
-----------------------------

    docker inspect <container>

Output a special field of the metadata:

    docker inspect --format='{{.Image}}' <container>


Container Livecycle
=========================

Stop running containers:

    docker stop <container..>

Start stopped containers:

    docker start <container..>

Kill running containers:

    docker kill <container..>

Remove containers:

    docker rm <container..>


Tricks: Container Id
==============================

give your containers a name

    docker run --name my_webserver nginx
    docker rm -f my_webserver

save the container id in shell variables

    c=$(docker run -d nginx)
    docker rm -f $c

start containers in foreground and with `--rm`, when playing arround:

    docker run --rm nginx


Interaction and debugging
==========================

exec
-----

Run a command in an existing container, e.g start a shell

    docker exec <container> <command>
    docker exec -it <container> bash

logs
--------
See the logs (stdout) of the container.

    docker logs -f <container>

copy
-------
copy files from and to docker container, e.g.

    docker cp my_webserver:/etc/nginx/nginx.conf ~/

Modify a container
===================

Inspect changes on a container's filesystem

    docker diff <container>

Create a new image from a container's changes:

    docker commit <container> <imagename>

Note:
-------

- Docker uses layered filesystems, so images and containers only need to store the diff to the their base image


Tricks: Cleanup Script
=======================================
You have to cleanup your local images and old containers regulary.

    docker rm $(docker ps -q -a -f status=exited)
    docker rmi $(docker images -q -f dangling=true)


Especially on test and build systems this should be part of a cron job.

    
    exited=$(docker ps -q -a -f status=exited | wc -l)
    
    if [ "$exited" != "0" ]; then
            docker rm $(docker ps -q -a -f status=exited)
    fi
    
    tagref=$(docker images -q -f dangling=true | wc -l)

    if [ "$tagref" != "0" ]; then
            docker rmi $(docker images -q -f dangling=true)
    fi

docker build
=====================

The normal way to create images is through `Dockerfile` build descriptions.

1. create a `Dockerfile`, e.g.

        FROM nginx
        COPY index.html /usr/share/nginx/html/
    
2. build the image an give it a name

        docker build -t my-nginx .
      
Note:
---------------------
- The build has the current directory as context
- All paths are relative to the Dockerfile
- Each command creates a new (temporary container)
- Every creation steps is cached, so repeated builds are fast


Dockerfile
=====================

FROM
--------

The `FROM` instruction sets the Base Image:

    FROM <image>:<tag>

Example: 

    FROM nginx:15:04

MAINTAINER
-------------

The `MAINTAINER` instruction sets the Author:

    MAINTAINER <name>

Example: 

    MAINTAINER Ahmed Kamal <akamal@cloud9ers.com>

Dockerfile
=====================

COPY
--------
`COPY` can be used to copy files or directories to the image.

    COPY <src>... <dest>

- Source can contain wildcards
- If dest does not exist it will be created

Example: 

    COPY service.config /etc/service/
    COPY service.config /etc/service/myconfig.cfg
    COPY *.config /etc/service/
    COPY cfg/ /etc/service/


Dockerfile
=====================

CMD
--------

With `CMD` you can specify the default command to execute on container startup.
It has two forms.

The exec form (preferred):

    CMD ["executable","param1","param2"]

The shell form:

    CMD executable param1 param2

__Attention__: the shell form starts the command in a sub process, so it will not get
the process id 1 and will not receive all signals e.g. from command line or `docker stop`.

    CMD ["nginx", "-g", "daemon off;"]

Dockerfile
=====================

ENTRYPOINT
-----------

The command in `ENTRYPOINT` will be executed on startup and allows you to configure a container that will run as an executable.

- The arguments in `CMD` are passed to the entrypoint by default
- If supplied, the `docker run` arguments overwrite those of the `CMD` and are passed as entrypoints arguments.

The exec form (preferred):

    ENTRYPOINT ["executable", "param1", "param2"]

Example: 

    ENTRYPOINT ["top", "-b"]
    CMD ["-c"]

Dockerfile
=====================

RUN
--------

The `RUN` command allows to execute commands arbitary commands in the container, which modify the
file system and produce a new layered container.

    RUN <command>

It is common to tie related commands together into one RUN command, using shell magic.

Example: 

    RUN apt-get update && \
        apt-get install -y ca-certificates nginx=${NGINX_VERSION} && \
        rm -rf /var/lib/apt/lists/*

Dockerfile
==========================

Practice
----------
Create a `greeting` image which can echo a configurable hello world greeting message in ascii art (e.g. using the ubuntu package figlet):

    docker run --rm greeting
    >  _   _       _ _         __  __                  _
    > | | | | __ _| | | ___   |  \/  | __ _ _ ____   _(_)_ __
    > | |_| |/ _` | | |/ _ \  | |\/| |/ _` | '__\ \ / / | '_ \ 
    > |  _  | (_| | | | (_) | | |  | | (_| | |   \ V /| | | | |
    > |_| |_|\__,_|_|_|\___/  |_|  |_|\__,_|_|    \_/ |_|_| |_|

    docker run --rm -e message=Hi greeting Arthur
    >  _   _ _      _         _   _
    > | | | (_)    / \   _ __| |_| |__  _   _ _ __ 
    > | |_| | |   / _ \ | '__| __| '_ \| | | | '__|
    > |  _  | |  / ___ \| |  | |_| | | | |_| | |
    > |_| |_|_| /_/   \_\_|   \__|_| |_|\__,_|_|


Dockerfile
=====================

ENV
------------------------
`ENV` sets environment variables which are present during container build and remain existent in the image.

    ENV <key> <value>
    ENV <key>=<value> ...

On container startup they can be overwritten with the `-e` or `--env` option:

    docker run -e key=value my_image

Example:

    docker run -e message='The answer is' -e answer=42 \
        ubuntu \
        bash -c 'echo $message $answer'
    The answer is 42

Dockerfile
=====================

ADD
--------
`ADD` can do the same as `COPY` with the following additions:

- If src is an URL, the file is downloaded
- If src is a local tar archive, it will be extracted to dest

Example: 

    ADD https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.4.tar.gz /es/
    RUN cd /es && tar xvfz elasticsearch-1.4.4.tar.gz

    ADD configs.tar.gz /etc/service/

Dockerfile 
=====================

VOLUME
------------------------
Declare folders for volume mounts.

    VOLUME ["/data"]

Benefit:
-------
- The user of you image has explicit documentation the available mounts
- The docker deamon and cloud tools can persist and backup them
- You can use the volumes from other containers by

        docker run --volumes-from container_with_volumes

Dockerfile 
=====================

EXPOSE
------------------------
With `EXPOSE` an image can declare the ports which should be exported.

    EXPOSE <port> [<port>...]

Benefit:
-------
- This information is needed for communication between linked containers
- The exposed Ports can be uses by the `docker run -P`:

        -P, --publish-all=false  Publish all exposed ports to random ports


Example nginx
=================
    FROM debian:jessie

    MAINTAINER NGINX Docker Maintainers "docker-maint@nginx.com"

    RUN apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys 573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
    RUN echo "deb http://nginx.org/packages/mainline/debian/ jessie nginx" >> /etc/apt/sources.list

    ENV NGINX_VERSION 1.9.3-1~jessie

    RUN apt-get update && \
        apt-get install -y ca-certificates nginx=${NGINX_VERSION} && \
        rm -rf /var/lib/apt/lists/*

    # forward request and error logs to docker log collector
    RUN ln -sf /dev/stdout /var/log/nginx/access.log
    RUN ln -sf /dev/stderr /var/log/nginx/error.log

    VOLUME ["/var/cache/nginx"]

    EXPOSE 80 443

    CMD ["nginx", "-g", "daemon off;"]


Setup multiple containers
==========================

The power of docker comes in, when you compose your apps out of multiple containers.

- Linking Containers
- Docker Compose


Linking Containers
===================
Docker has the concept of links between contianer:

- Get the container name into /etc/hosts
- Set sehll variables for the container
- Access to not exposed ports of the container

Example:
----------
    docker run --name nginx -d nginx
    docker run --rm --link nginx ubuntu \
        bash -c 'echo -e "GET / HTTP/1.1\nHost: nginx\n" | netcat nginx 80'

Inter Container Communication
----------------------------------
- Start the docker daemon with `--icc=false --iptables=true` to disable inter container communication by default.

docker-compose
================
A docker tool for starting multiple container compositions

Configuration for `docker-compose.yml`:

    web:
      build: .
      ports:
        - "5000:5000"
      volumes:
        - .:/code
      links:
        - redis
    redis:
      image: redis


docker-compose:
==================
    Usage:
      docker-compose [options] [COMMAND] [ARGS...]
      docker-compose -h|--help
    
    Commands:
      build              Build or rebuild services
      help               Get help on a command
      kill               Kill containers
      logs               View output from containers
      port               Print the public port for a port binding
      ps                 List containers
      pull               Pulls service images
      restart            Restart services
      rm                 Remove stopped containers
      run                Run a one-off command
      scale              Set number of containers for a service
      start              Start services
      stop               Stop services
      up                 Create and start containers
      migrate-to-labels  Recreate containers to add labels


Advanced Topics?
================

Docker-machine

Docker Swarm

Multihost network overlays

Hosting private registeries

Service registeration and discovery

Ambassador pattern

Scalable cloud hosting

Further reading
===============

https://www.docker.com

man docker

https://docs.docker.com/reference/builder/

https://hub.docker.com/

![Open Source](http://www.techweekeurope.co.uk/wp-content/uploads/2011/02/Open-Source-Software-.jpg)