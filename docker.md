# concept
* registry, daemon, client, images, container
* main registry is docker hub
* container are instances of images
* container has state, image doesn't
* images can have parent/child relationship
* daemon does all the heavy lifting
    * build image
    * running image
* docker engine = docker daemon + docker cli
* components
     * manager: run, stop, etc
     * builder: creates image
     * image registry: storage (normally cloud) of images


# solves
* dependency hell
    * missing
    * conflicting
* platform difference


# starting
* `docker run hello-world`
    * it'll pull `hello-world` from registry and run it

# dockerfile
* recipe for building a docker image
    * with a special name `Dockerfile`
* commands:
    * `FROM`: what's the base of this image
    * `RUN`: exe command at build time
    * `ENTRYPOINT`: exe command at container boot time
    * `EXPOSE`: expose port to external world
    * `ADD`: add file to image, a superset for `COPY`. allows remote file
    * `WORKDIR`: change working dir in container
    * `VOLUME`: mount from host (used together with -v flag when running image)
    * `USER`: specify user id for running commands
    * `CMD`: command for running your application
    * `COPY`: copy local file into container image
* to build, `docker build -t username/appname .`
    * `docker.io build -t wenliang/appname - < dockerfile`
    * has to be run as root on Linux

# docker commands
* `docker pull`
    * download image from registry
* `docker push`
    * push image into registry
* `docker run`
     * `docker run 5ba9dab47459 uname -a`
     * `docker run -i -t -p 8000:8000 ubuntu /bin/bash`
     * `-w` sets working directory inside the container
     * `-e` sets envar in the container. `-e MYSQL_ROOT_PASSWORD=aaabbb`
       can be specified multiple times
     * `-it` gives interactive shell
* `docker ps`
    * list all running container
    * `-a -q` to list all containers
* `docker stop`
* `docker start`
* `docker images`
    * list images
* `docker rm`
    * remove container
    * `docker rm $(docker ps -q -a)`
* `docker rmi` // remove image, can only be done is the container is removed
* `docker commit c3f279d17e0a  svendowideit/testimage:version3`
    * commits changes made to container into image
    * creates a new image
* `docker attach`
* `docker kill`
* `docker exec`
    * runs a command in an already running container
    * `docker exec -it /bin/bash`
* `docker cp <containerId>:/file/path/within/container /host/path/target`
    * copy file out of the container
    * also works the other direction
* `docker export`
    * tarball container's file system
* `docker diff`
    * show changed file in container's FS
* `docker tag ubuntu-git:latest ubuntu-git:1.9`
    * create a new tag


# advanced tool
* `docker-machine` makes it easy to build host locally or in the cloud
* `docker-compose` is a tool for running multi-container applications with docker
    * [realpython](https://realpython.com/blog/python/dockerizing-flask-with-compose-and-machine-from-localhost-to-the-cloud/)
* `docker-swarm` for clustering


# registry & repository
* repository is a collection of images
* registery is a host that holds the collection
* `docker login`
* `docker search`
* `docker pull`
* `docker push`


# layers
* versioned file system for docker container
* like git commit


# docker volume
* by default, data in containers are not persisted
* volume can:
    1. keep data around
    2. share data between container and host
    3. share data between containers
* `docker run -v /local/path:/remote/mount_point/ image`
    * this will share stuff between container and host
    * also persists stuff
* `docker create -v /data --name data_container ubuntu`
    * creates a volume container
* `docker run -t -i --volumes-from data_container ubuntu /bin/bash`
    * Use it
    * `/data` will appear in the container
* `docker run -d --name bmweb -v ~/htdocs:/usr/local/apache2/htdocs...`
    * serve local html directory through apache
    * `-v ~/htdocs:mountpoint:ro` will mount the volume read only
* When specifying a volume map with the key, docker creates a so-called managed
  volume for the user
    * `docker run -v /some/mount/point`
    * `docker inspect` will show where the data are actually stored

## sharing
* when host directory is mounted on multiple containers, it can be shared
* also true for volume only container if it's mounted multiple times


# Orchestration
* The concept of a machine or a host does not exist anymore.
* We have a pool of resources (memory/cpu/network) which is the sum of all the
  resources of the machines on our cluster.
* Problems
    * Service Discovery
    * High Availability
    * Resource Management
    * Ports Management

## mesos
## kubernetes
* A declarative language for launching containers
* a.k.a kube, k8s
* pod
    * cannot span machine
    * single IP

## docker 1.12
* stack
    * a collection of services
    * defined in YAML file


# docker-compose
* **Compose is great for development, testing, and staging environments, **
* Using YAML to define a cluster
    * services
    * networks
    * volumes
* default to `docker-compose.yml`
* `docker-compose up -d`
    * builds and start the cluster
    * `-d` for detach mode. releases the shell
* `docker-compose stop`
* `docker-compose rm -v`
    * cleanup
    * `-v` to stop making volume becoming orphan
* not a replacment for mesos and kubernetes
    * single host only
## YAML file
* `build`
    * images are built from local directory
* `links`
    * link service
    * provide name resolve
    * also implies `depends_on`
* `depends_on`
    * dependency between services
    * defines service start order
* `environments`:
    * defines envars that'll be used by the container
* `ports`
    * expose and map port
    * `- 8080:80` exports port 80 and maps to 8080 on host
* `volumes_from`
    * mount volume from another container
* `volumes`
    * mount volume
    * can be docker managed volume or host volume
* `commands`
    * set default command
    * `"true"` when nothing to be done
## Networking
* by default, `docker-compose` creates a single network
* serice name can be used to resolve the container's IP
## command
* `docker-compose build`
    * build all images
    * `up` command will not automatically rebuild stuff
* `docker-compose scale SERVICE=3`
    * scale out
## Sample
    [yaml]
    version: '2'
    services:
     mysql:
      image: mysql
      container_name: mysql
      ports:
       - "3306"
      environment:
       - MYSQL_ROOT_PASSWORD=root
       - MYSQL_DATABASE=ghost
       - MYSQL_USER=ghost
       - MYSQL_PASSWORD=password
     ghost:
      build: ./ghost
      container_name: ghost
      depends_on:
        - mysql
      ports:
        - "80:2368"

# host
## Linux
* can be installed on ubuntu
    * 64 bit a bliss
    * 32 bit not so much so
* stuff are stored in `/var/lib/docker`

## mac
* download and install    **Docker Toolbox**
    * it provides a short cut to a special shell environment that allows docker to run.
    * running `docker` on a normal shell would fail
* default machine IP: 192.168.99.100


# docker-machine
* Manager for virtual machines, aka hosts
* On Mac, a default machine (virtualbox) is created
* Machines are bind to shell. Each shell has a concept of **active** machine.
    * images are actually bound to a machine, which is weird
    * `docker-machine active` checks which machine is active
    * `eval $(docker-machine env machine_name)` switches to **machine_name**
* docker images are pulled per machine
* `docker-machine create -d virtualbox host1`
    * create new machine with name `host1` and `virtualbox` driver
* `docker-machine ls`
    * active machine has `*` next to it
* `docker-machine inspect host1`
* `docker-machine ssh host1`
    * connects to the machine with a shell
    * also possible to run a command
* `docker-machine scp`
* `docker-machine stop foo`
* `docker-machine rm foo`
* `docker-machine start name`

## drivers
* AWS
* digital ocean
* virtualbox
* vmware fusion


# networking
* `docker network create foobar`
    * create a network
* `docker run --net foorbar nginx`
    * launch into the network
* `docker network ls`


# swarm
* a swarm is a cluster of docker hosts
* **program against your datacenter like it's a single pool of resources**
    * from Mesos
* a cluster normally has a master and multiple nodes
    * manager and worker nodes
    * each run a swarm agent
    * been a managers or a workers simply means that manager or agent software
      are running on the host
* **A Docker client could be configured to work with any of these machines
  individually. They’re all running the Docker daemon with an exposed TCP socket
  (like any other Docker machine). But when you configure your clients to use
  the Swarm endpoint on the master, you can start working with the cluster like
  one big machine.**
    * From **Docker in Action**
* each node can have a lable indicating what purpose it has
    * E.g. `java`, `database`
* in `docker compose` file, constrain can be specified what node can the
  container go into
* the master automatically distributes container into the right node(host)
* a service discovery layer is required
    * for the manager to know the nodes
    * E.g. `Consul`, `etcd`, `zookeeper`
* **with Linux containers for isolation and Docker for container tooling, the
  remaining major concerns are efficiency of resource usage, the performance
  characteristics of each machine’s hardware, and network locality. Select- ing
  a machine based on these concerns is called scheduling.**
    * From **Docker in Action**

## Service
* auto scaled docker container
* `docker service create --replica 3 --name frontend --network mynet --publish
  80:80/tcp frontend_image:latest`
    * 3 containers
    * internally load balanced and expose one name port pair
* failure is automatically detected and recovered
* `docker service scale frontend=6`
* service include tasks, which maps to containers
* `docker service inspect service_name`
* `docker service ps`
    * list all running services including where it's run
* `docker service rm helloworld`
    * remove service
    * stop all running containers

## working with compose
* have to use compose file format version 2, which includes **constrains**

## CLI
* `docker swarm init --advertise-addr <ip>`
    * start a new swarm
    * has to be run on the manager node
    * output will show how others can join the cluster
* `docker run swarm create`
    * has to be run on a docker machine (???)
* `docker-machine create --driver virtualbox --swarm --swarm-discovery
  token://TOKEN --swarm-master machine0-manager`
    * `--swarm` this machine should run the agent software
    * `--swarm-master` this is a master node
* `docker-machine create --driver virtualbox --swarm token://TOKEN machine1`
    * create a worker node
* `docker node ls`
    * list all nodes in the cluster
    * can only be run on a manager node

## Internal
* internally all nodes communicate through gossip protocol
* gRPC is used


# Hosting Services
## alauda 灵雀云
* has public registry
## daocloud
* has internal registry
* has an **accelerator** that's basically a local registry
* e.g.
    * `FROM daocloud.io/ubuntu:14.04.1`
* bind coding.net account
* start a build
    * when it finished, it'll be automatically pushed to daocloud.io registry
    * isn't obvious that the build is finished
* start a deployment
    * select a image
    * select a service for binding
    * at the end, there will be an address for access
* XXX: free account doesn't allow TCP protocol
* has Sass containers like mongodb, etc
## tenxcloud 时速云
* can have private build
* same old
    * link with coding.net
    * start a build
    * have to add tenxcloud generated key to coding.net for pulling from git
* similiarly has accelerator
* tenxcloud allow any port and any protocol to be exposed
* orchestration yaml syntax is private


# Platforms
* On linux, docker runs natively on the host
* On Mac and Windows, docker runs inside a VM
    * ??? changed in 1.12?
* zsh integration
    * turn on `docker` plugin in `.zshrc`


# Version history
## 1.12
* Swarm-mode networking 
* Routing Mesh  ???
* Ingress and Internal Load-Balancing 
* Service Discovery 
* Encrypted Network Control-Plane and Data-Plane  ???
* Multi-host networking without external KV-Store 
* MACVLAN Driver ???


# Links
* [docker cli reference](https://docs.docker.com/engine/reference/commandline/cli/)
* [openwrt image](http://wiki.openwrt.org/doc/howto/docker_openwrt_image)
* [nice cheat sheet](https://github.com/wsargent/docker-cheat-sheet)
* docker is now on the same level as openstack instances:
    * [news](https://wiki.openstack.org/wiki/Docker)
* baidu uses docker for its pass
    * [baidu](http://blog.docker.com/2013/12/baidu-using-docker-for-its-paas/)
* online slides
    * [slides](http://goldmann.fedorapeople.org/tmp/docker-preso/#/)
* Docker getting started
    * [started](http://tiewei.github.io/cloud/Docker-Getting-Start/)
    * CN translation
* [flux7-tutorial](http://blog.flux7.com/blogs/docker/docker-tutorial-series-part-1-an-introduction)
* [getting-started](https://opensource.com/business/14/7/guide-docker)
* [quick-ref](http://www.charlesmerriam.com/docker/quick_docker.html)
* [fullstackpython](http://www.fullstackpython.com/docker.html)
    * A nice list of links with description
* [docker orchestration](http://dockone.io/article/861)
    * docker orchestration, the full story
    * original is [here](https://railsadventures.wordpress.com/2015/11/15/docker-orchestration-the-full-story/)
* [yeosd docker](https://ilikewhenit.works/blog/1)
* [docker composer](https://www.linux.com/learn/introduction-docker-compose-tool-multi-container-applications)
    * building a Ghost blog site with docker cluster
* [compose file reference](https://docs.docker.com/compose/compose-file/)
    * from official site
* [load balancer tutorial](http://anandmanisankar.com/posts/docker-container-nginx-node-redis-example/)
* [how to scale](http://stackoverflow.com/questions/18285212/how-to-scale-docker-containers-in-production)
    * SO questions. Answer shows historical progression of this topic
* [native docker](http://thenewstack.io/native-docker-comes-windows-mac/)
* [local devel](http://www.masterzendframework.com/docker-development-environment/?mkt_tok=eyJpIjoiTlRBek9UVTRNREF5TjJaaSIsInQiOiJsN2pNZWNHT1kzbjB2SlNXbDhMdEVFamNQSitIdWZPQWl0SWJydVJTQ0ZzQTY4XC90cGFwNGF4QnRRTGF4RjlIRHVTSGdGaDc0b016am5oaUxtUmtNUmdRNThocHZXMVlYbCtyQlBQaDg1eEU9In0%3D)
    * local development environment setup
    * 3 main docker containers, php, nginx, mysql with volumes
    * volume container which shares code from the project directory to
      `/var/www/html`. developer writes code here
    * nginx: mounts from the volume container
    * php: mounts from the volume container
    * mysql: uses managed volume for data storage