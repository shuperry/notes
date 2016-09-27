# Concept and Philosophy
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
* containers are completely stateless and immutable
    * data should be persisted and backedup with volumes
* 4 core things
    * standard packaging format
    * clearly defined interface
    * caching mechanism for re-use steps
    * central registry
## image
* images are just hierarchy of files


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
    * `RUN`: exe command at **build time**
    * `ENTRYPOINT`: exe command at container boot time
    * `EXPOSE`: expose port to external world
    * `ADD`: add file to image, a superset for `COPY`. allows remote file
    * `WORKDIR`: change working dir in container
    * `VOLUME`: mount from host (used together with `-v` flag when running image)
    * `USER`: specify user id for running commands
    * `CMD`: command for running your application
    * `COPY`: copy local file into container image
* to build, `docker build -t username/appname .`
    * `docker.io build -t wenliang/appname - < dockerfile`
    * has to be run as root on Linux
* every command in the file creates a new filesystem layer
* use `.dockerignore` to ignore files
* `USER` can be specified multiple times in a single file for different user

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
    * `-i` interactive
    * `-t` tty
    * even if the container exists, it's status is kept until `rm`ed
    * under the hood, a new writealbe file sysetm is created
* `docker run --link dnmonster:dnmonster identidock`
    * run `identidock` and link to `dnmonster` with the name `dnmonster`
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
    * `docker rm $(docker ps -a -q -f status=exited)`
* `docker rmi` // remove image, can only be done is the container is removed
* `docker commit c3f279d17e0a  svendowideit/testimage:version3`
    * commits changes made to container into image
    * creates a new image
    * commit can be done even if container has already exited
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
* `docker save from_image to_image`
    * save a image to a tarball
* `docker diff`
    * show changed file in container's FS
* `docker tag ubuntu-git:latest ubuntu-git:1.9`
    * create a new tag
* `docker import`
    * create image from a tarball
* `docker logs`
    * retrieve logs from a running/stopped container
* `docker inspect`
    * show internal stuff for container/image
* `docker port`
    * show exposed port for a container


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
## tag
* tags can be applied to an image. Unfortunately most people uses the numbered
  tags like `1.0, 2.0`, etc. And this implies version numbers. But tags are
  **NOT** version numbers. That's why `latest` doesn't follow the **latest**
  pushed tag


# layers
* versioned file system for docker container
* like git commit
* Unionfs makes sure of the layered file system
    * [LJ](http://www.linuxjournal.com/article/7714)
    * later version of docker switched to aufs
* aufs
    * advanced multi-layered unification filesystem
* Merges several directories into a single view
* directories are called **branches**
* view is call an **union**
* E.g when we create a new image with `docker commit`, we can see that a new
  layer is created with `docker inspect`
    * similarly with user generated images
* all layers in images are read only. when a container is in action, there is
  writable layer created
## UnionFS Practice
* `mount -t unionfs -o dirs=/Foo:/Bar none /mnt/ufs`
    * merge directory `Foo` and `Bar` into one view
    * `Foo` takes priority over `Bar`
* `mount -t unionfs -o dirs=/tmp/cdpatch,/mnt/cdrom  none /mnt/patched-cdrom`
    * how live CD like Knoppix works
## aufs practice
* `mount -t aufs -o dirs=/path/to/files1:/path/to/files2 none /path/to/files`
* `none    /path/to/files     aufs    dirs=/path/to/files1:/path/to/files2 0 0`


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


# remote API
* ...


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
* oldest
* Marathon UI
* lower level facilities so that swarm or kubernetes can run on top
## kubernetes
* A declarative language for launching containers
* a.k.a kube, k8s
* pod
    * a group of containers (typically 1-5)
    * cannot span machine
    * share single IP
    * shared volume
    * offer service(s) from a single IP
    * ephemeral rather than durable
* network
    * within a pod, containers talks to each other through ports
    * flat network space
    * pods talk to each other without NAT
    * flannel
* Label
    * key value pair attached to pods
    * for identifying group of objects
* Service
    * cache service, etc
    * provides single, stable name and address for a set of pods
* Replication controller
    * manages lifecycle of pods
## docker 1.12
* stack
    * a collection of services
    * defined in YAML file
## rancher
* nice UI
* forcus on hybrid infrastructure
* rancher-compose file
    * configure number of instances for a container
    * load-balancer
    * health check
    * binary provided across platforms
## OpenStack
* [magnum](https://wiki.openstack.org/wiki/Magnum)
## nomad
* ...
## Fleet
* Builds on top of systemd
    * systemd unit file (???)
* Each machine run an agent and an engine
* Etcd for service discovery and HA
* Supports scheduling hints and constraints


# docker-compose
* **Compose is great for development, testing, and staging environments, **
* Using YAML to define a cluster
    * services
    * networks
    * volumes
    * also can use JSON file
* default to `docker-compose.yml`
* `docker-compose up -d`
    * builds and start the cluster
    * `-d` for detach mode. releases the shell
* `docker-compose stop`
* `docker-compose rm -v`
    * cleanup
    * `-v` to stop making volume becoming orphan
* `docker-compose -p`
    * specify a project name, default to current directory
    * allows multiple setup of the same compose
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
* `command`
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
    * or use `docker $(docker-machine config machine_name bla` to run a command
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
* `docker-machine ip name`
    * print out ip of the machine
* `docker-machine create --drive=generic --generic-ip-address=x.x.x.x.`
    * create a generic machine who's ip is `x.x.x.x`
    * e.g. an embedded low cost machine
## drivers
* AWS
* digital ocean
* virtualbox
* vmware fusion
## Swarm
* `docker-machine create --swarm --swarm-master --swarm-discovery`
    * create swarm
    * create swarm master
    * specify swarm discovery node
* `docker-machine create --swarm --swarm-discovery`
    * this creates a swarm node
    * no `--swarm-master`


# Networking
* in terms of networking, there are 4 types of hosts
    * closed container
    * bridge container
    * joined container
    * open container
* **joined container**
    * all container startd in joined mode share the same network device. E.g.
      loopback
* links
    * when ICC is enabled, linking is easy. When it's disabled (via the daemon),
      link is achieved with extra firewall rules
    * links also add DNS override in the docker environment
    * links creates environment variable inside the linking container
    * links are uni-diretional
    * links are **NOT** tansitive
* network
    * introduced in 1.9
    * container can communicate with each other with an overlay network cross machine
## CLI
* `docker network create foobar`
    * create a network
* `docker run --net foorbar nginx`
    * launch into the network
* `docker network ls`
* `docker network create -d overlay myStack1`
    * create overlay network
* `docker run --net none alpine`
    * run a closed container - only loopback
* `docker run --net bridge alpine`
    * run a bridged container
* `docker run --name target --expose 1234 alpine nc -l 0.0.0.0:3306`
    * gives name to a container
* `docker run --link target:alias`
    * establish link to container `target`
## internal
* docker normally creates a bridge interface called `docker0`.
* all local containers share the same bridge and thus can communicated with each
  other
* `docker -d --icc=false`
    * run the docker daemon with inter container communication turned off
* `docker -d -b bridge_name`
    * use existing bridge instead of creating a new one
* `docker run --net container:brady bla`
    * start a container in joined mode and share network device with existing
      container **brady**
* `docker run --net host ip addr`
    * creates open container
## Overlay network
* manages multi-host networking
* Use VxLAN


# Data
* volume plugin

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
* `docker swarm join \
    --token SWMTKN-1-4zwb31crnhpkp5a4ri1swnvn9kvwehu46ckh95c94jc4py4hzd-ad2xvq9habvodcnbvk8pnoubh \
    192.168.99.100:2377`
    * run from a worker node to join the swarm
    * depend on the token, the node will join as a manager or worker
* `docker run swarm create`
    * has to be run on a docker machine (???)
* `docker-machine create --driver virtualbox --swarm --swarm-discovery
  token://TOKEN --swarm-master machine0-manager`
    * run from the host
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
## Default Token Based Discovery
* `docker run swarm create`
    * run the `swarm` container
    * generates a token
    * this token is share among machines via the hub
* `docker-machine create --swarm --swarm-master --swarm-discovery token`
    * create a manager with that token
* `docker-machine create --swarm --swarm-discovery token`
    * create a worker with that token
* `curl https://discovery-stage.hub.docker.com/v1/clusters/<token>`
    * returns a list of machine in that cluster
* This approach has a single point of failure: the hub. So it's not ideal
## Constraint
* Within a cluster, help the manager to decide which machine can run which
  container.
* `docker-machine create --engine-label dc=a`
    * create label
* `docker run -d -e constraint:dc=b`
    * run container with constraint
## Schedule Strategy
* Spread
    * least loaded
* binpack
    * most loaded
* random
    * well, random


# Remote API
* REST API that replaces the remote command line interface
* Sample API end points, all under `/containers`
    * `/reated`
    * `/{id}/start`
    * `/{id}/stop`
    * `/{id}/export`
    * etc


# Docker Internal
* relies on host kernel for
    * resource isolation (CPU, memory, I/O, network)
    * separate namespace
* [youtube]https://www.youtube.com/watch?v=sK5i-N34im8
    * demo that builts a container with shell
## cgroup
* control group
* kernel feature to limit, police and account the resource usage for a set of
  processes
    * [arch wiki](https://wiki.archlinux.org/index.php/cgroups)
## namespace
* kernel feature to isolate and virtualize resources
* pid, network, mount, hostname, userid, ipc, filesystem, etc
* `unshare`
## UnionFS
### copy on write
* very important for docker's success
### aufs
### btrfs
## LXC
* Linux container
* In user space
* Provides a virtual environment that has its own CPU, memory, etc
* Sits on top of cgroup and namespace
* Docker used to be based on LXC and later moved onto libcontainer


# Security
* libcontainer supports SElinux and AppArmor
## Things that are NOT namespaced
* user are *NOT* namespaced. So anyone in the container, if broken out, is the
  same user on the host.
* kernel and kernel modules are not namespaced
    * including the keyring
* time
* devices, disk, sound-card
* SELinux is not namespaced
* from docker 1.10, user name space can be remaped
    * `dockerd --userns-remap=default`


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


# Docker Plugins


# Platforms
* On linux, docker runs natively on the host
* On Mac and Windows, docker runs inside a VM
    * ??? changed in 1.12?
* zsh integration
    * turn on `docker` plugin in `.zshrc`


# Competition
## cgroup + namespace
* rtk
* runC
## Other Linux
* openvz
## BSD/Solaris
* jails
* zones


# Version history
## 1.12
* Swarm-mode networking
* Routing Mesh  ???
* Ingress and Internal Load-Balancing
* Service Discovery
* Encrypted Network Control-Plane and Data-Plane  ???
* Multi-host networking without external KV-Store 
* MACVLAN Driver ???


# Storage
## devicemapper
* this is the docker storage driver.
* the kernel framework has the same name and can be called "Device Mapper"
    * works on block level instead of file level
* this triggered the plugin architecture of docker storage
* default to `loop-lvm`
* production enviroment is suggested to use `direct-lvm`
## overlayfs
* modern union file system
* in mainline kernel since 3.18
    * there is `overlay2` after 4.0
    * `lsmod | grep overlay`
* docker support from 1.12
* `dockerd --storage-driver=overlay`
* faster than `aufs` and `devicemapper`
    * less mature


# CI/CD
* `git pull` when something is pushed
* `docker build .` to build an image
* `docker push` to push the image to a public/private registry
* notify server about the new image
* either test or deploy it
## Docker exceution
* Docker socket mount
    * `--docker-volumes /var/run/docker.sock:/var/run/docker.sock`
    * jenkins access hosts docker socket interface, and launches **sibling**
      containers
* Docker in Docker
    * discourage
    * [here is why](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/)
## Testing
* git hook for automation
* tag
## Staging and production
## Backup
* Rung the Jenkins container with a volume
## Jenkins
* Jenkins can have **build slaves**
* There is also docker plugin for jenkins



# Logging
* By default docker logs STDOUT and STDERR
* No log rotation
* `docker logs -f container`
    * Attach to the log stream of the container
* `docker run --log-driver`
    * json-file
    * syslog
    * journald
    * gelf
    * fluentd
## ELK
* ElasticSearch
    * near real-time, distributed text search
* Logstash
    * pre-process and forward raw logs
    * to ElasticSearch
* Kibana
    * Javascript based GUI for ElasticSearch
## Logspout
* docker-specific log collector
* Go + Alpine
* Can work with Logstash with an adaptor
* Needs to use the Dockr socket API


# TODO
* overlay network
* rancher
* try codeship
* deploy zoomapp
* linux bridge network
* libvirt
* libcontainer
* setup jenkins CI


# Exteral Tools
* [harbor](https://github.com/vmware/harbor)
    * enterprise class docker registry
    * added security, identity
* [shipyard](https://shipyard-project.com)
    * container management system with GUI
* [cadvisor](https://github.com/google/cadvisor)
    * container monitor tool
* [dubbo](https://github.com/alibaba/dubbo)
    * distributed RPC framework by alibaba
* [consul]()
    * KV service used for service discovery
    * built-in health check
    * normally run with `consul -server -bootstrap` flags
* [flocker]()
    * data volume plugin
* [glusterfs](https://www.gluster.org)
    * distributed file system
* [joyent](http://www.joyent.com)
    * secure enough to run on bare metal
* [prometheus](https://prometheus.io)
    * open source monitoring tool
* [packer](https://www.packer.io)
    * tool for creating machine and images
* [skydns](https://github.com/skynetservices/skydns)
    * service discovery with DNS for etcd
* [systemd](https://wiki.archlinux.org/index.php/systemd)
    * service manager
    * compatible with sysV and LSB
* [s6](https://github.com/just-containers/s6-overlay)
    * process supervisor
    * [official](http://skarnet.org/software/s6/overview.html)
* [CoreOS Enterprise Registry]()
* [Docker Trusted Registry]()
* [docker-backup]()
* [skynet dns]()
    * docker dns service based on etcd
* [so question](http://stackoverflow.com/questions/18285212/how-to-scale-docker-containers-in-production)
    * an answer with 24 updates with loads of links
* [rackHD]()
    * hardware provisioning open source software


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
* [prakhar](https://prakhar.me/docker-curriculum/)
    * extensive tutorial from starting point to AWS to elastic search
* [中文教程](http://blog.csdn.net/zhikun518/article/details/51114026)
* [not so short](https://www.jayway.com/2015/03/21/a-not-very-short-introduction-to-docker/)
    * pretty lengthy tutorial of docker
    * companion PPT is lengthy too
    * a bit dated but still good info
* [docker orchestration workshop](http://jpetazzo.github.io/orchestration-workshop/?utm_source=twitter&utm_medium=social&utm_term=Jerome+Petazzo%2Corchestration&utm_content=&utm_campaign=Social+Media+Department#1)
    * slides
