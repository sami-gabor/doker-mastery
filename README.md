# Doker Mastery
Docker is a self contained space for an app to run.
* app structure
* scalability
Images -> building blocks of containers
Doker Compose -> design for local dev and test
Orkestrators - Swarm, Kubernetes ??
MobyDock / @GordonTheTurtle
Nginx - basic web server

## Commnds
`--help`

#### Image vs. Container
Image = the app I want to run
Container = instance of that image running as a process(can be multiple)
Image registry - hub.docker.com (the equivalent of GitHub)

* `docker version` - verifies cli talk to engine
* `docker info` - most config values of engine
* `docker` - list of commands
* `docker container ls` ~ `docker ps` - list running containers
* `docker container ls -a` - list all containers
* `docker container start <container>` - start an existing stopped container
* `docker container stop <container>` - stop container
* `docker container rm <container>` - delete a container
* `docker container rm -f <container>` - delete a running container
* `docker container run` - start a new container
** `docker container run --publish 80:80 nginx` --> `--publish` (sets the port?!)
** `docker container run --publish 80:80 --detach nginx` --> `--detach` - stay active in the background after `Ctrl+C`
* `docker top <container>` - list processes container
* `ps aux` - show me all running processes
* `ps aux | grep <container>` - show the running process
* `docker image ls` - show all local images

##### What's going on in containers
* `docker container top <container>` - process list in one container
* `docker container inspect <container>` - details of one container config(json)
* `docker container stats [container]` - live performance stats on all containers[or a specific one]

##### Getting a shell inside containers
* No need for SSH(use docker cli)
`docker container run -it` - start a new container interactively(get a shell inside the container). Similar to SSH into a server
* `-t` - pseudo-tty (simulates a real terminal, like what SSH does)
* `-i` - interactive (keep session open to receive terminal input)
`docker container run -it --name proxy nginx bash`
* the default command for nginx container is to run the nginx program itself, but `bash` changed it to be bash, which gave the shell
* on `exit` the container stops - container only runs as long as the command (it run on start up) runs
`docker container exec -it` - runs additional commands in existing container(run a second process)

###### Ubuntu image
* It's default CMD is bash, so we don't specify it
`apt-get update` - like on a standard ubuntu server

###### Alpine Linux
A small security-focused distribution of linux(very small - 5Mb)
`docker container run -it alpine sh` - so small, it doesn't have bash installed
`apk` - alpine package manager

`docker container port <container>`
`docker container inspect --format "{{ .NetworkSettings.IPAddress }}" <container>` - get container IP

###### Docker Networks: CLI Management of Virtual Networks
`docker network ls` - show networks
`docker network inspect <network>` - inspect a network(json obj)
`docker network create <network> [--driver <driver>]` - [use built in and 3rd party drivers to] create a virtual network where containers can be attached
* network driver - built-in or 3rd party extensions that give you virtual network features(default is bridge)
* bridge driver - simple driver that simply creates a virtual network localy with it's own subnet somwhere around the 172.17... and above(increases.. 18, 19...). Doesn't have some of the advanced features like overlayed networks... or like other 3rd party drivers like weave...
`docker network ls` - show networks
`docker network connect <network> <container>` - attach a network to a container(dynamically creates a NIC in a container on an existing virtual network)
`docker network disconnect` - detach ...
`bridge` - default network that bridges through the NAT firewall to the physical network that the host is connected to
`172.17.0.0` - tipically the default network for any docker host(but it can be changed) and the gateway for this network that will eventually route out to the physical network
`host` - a special network that skips the virtual networking of docker and attaches the container directly to the host interface
* PRO - prevents the security boundries of containerization 
* CON - can improve the performance in certain situations
`none` - like an interface that's not attached to anything

###### Docker Networks: DNS
Static IP's and using IP's for talking to containers is anti-pattern!
Docker daemon has a built-in DNS server that containers use by default.
Docker uses the container names as the equivalent of a host name for containers talking to eachother.
Docker defaults the hostname to the container's name (but you can set aliases).
A network that's not the default bridge, gets a special new feature which is automatic DNS resolution for all the containers on that virtual network from all the other containers on that virtual network using their container names. If I crete a new container on the same network, they are able to find eachother regardless of the IP address, using their container names.

`docker container run -d --name nginx_1 --network app_1 nginx:alpine`
`docker container run -d --name nginx_2 --network app_2 nginx:alpine`
`docker container exec -it nginx_1 ping nginx_2` - ping doesn't work in nginx by default anymore(must create alpine container)

The default `bridge` network has one disadvantage: doesn't have the DNS server built-in by default, so you can't use `--link. 
With `--link` option you can specify manual links between containers in that default bridge network, but it's just earier to create new networks for my apps so I don't have to do this every time (docker compose will make container communication much easier).

* Don't use IP addresses for inter-communication.
* Use DNS(built-in for custom networks) for communications between containers on the same host and across host
* Recomanded to always use custom networks. It's easier then using --link all the time
* docker compose will make it easier(especially the networking)

###### DNS Round Robin
The concept of having two different hosts with DNS aliases that respond to the same DNS name(helps being up 24/7) => multiple IP addresses in DNS records behind the name used on the internet'.
Cursom network => we can asign an alias so that multiple containers can respond to the same DNS name.
`elasticsearch` - data store for search(json)

`docker network create dude` - custom network
`docker container run --net dude -d --net-alias search elasticsearch:2` - custom containers attached to dude network
`docker container run --network dude --rm alpine nslookup search` - run nslookup on dns search entry(get all dude containers with DNS 'search'). Immediately clean up and exit with --rm
`docker container run --net dude centos curl -s search:9200` - Round Robin elasticsearch server named 'search'

## 4. Container images
* App binaries and dependencies
* Metadata about the image data and how to run the image
* not a complete OS. The host provides the kernel, kernel modules(drivers)
* unlike a VM it's not booting up a full OS, it's really just starting an app
* can be very small(one file - eg. go) or very big(Ubuntu - multiple GB)

###### The Mighty Hub: Using Docker Hub Registry Images
`docker image pull <image>`
`docker image history <image>`
Copy On Write - when a container modifies a file from an image, it actualy copy that file into container and changes it
* Images are made up of file system changes and metadata
* Each layer is uniquely identified and only stored once on a host
* This saves storage space on host and transfer time on push/pull
* A container is just a single read/write layer on top of image

###### Image tagging and Pushing to Docker Hub
`docker image tag --help`
`docker image push <image>` - uploads changed layers to a image registry (Hub)

###### Building images: The Dockerfile Basics
Dockerfile is a recipe for creating an image

###### Building images: The Dockerfile Basics
`docker image build .` - build from current folder(Dockerfile inside)

###### Building images: Extending official images
In default configuration nginx is acting as a web server and it's serving static files right of the container disk


## 5. Container Lifetime & Persistent Data: Volumes, Volumes, Volumes

#### Container Lifetime & Persistent Data
`docker volumes ls`
`docker volumes prune` - remove unused volumes
`docker container run -d --name test_mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True mysql` --> "-e" is required
`docker container run -d --name test_mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mysql`
`docker volume create` - if you want to vrete it ahead of time(not often)

#### Persistent Data: Bind Mounting
* bind mount - maps a host file/directory to a container file/directory
* two locations point to the same file
* can't be in Dockerfile
`docker container run -d --name nginx_test -p 8080:80 -v E:/Learning/doker-mastery:/usr/share/nginx/html nginx`

#### Real World Scenario: updating a db(posgres)
`/var/lib/postgresql/data` - volume path from DockerHub
`docker container run -d --name psql1 -v psql:/var/lib/postgresql/data postgres:9.6.1`
`docker container logs -f psql1` - "-f" keeps watching as it runs
`docker container stop <container1>`
`docker container run -d --name psql2 -v psql:/var/lib/postgresql/data postgres:9.6.2`
`docker volume ls` - check the correct name volume(local psql)
`docker container logs <container2>` -> database system is ready to accept connections!


#### Bind Mounts
* cool for local dev / locat testing
* use a static site generator
1. take host data
2. mount it into a container
3. change it on host
4. whatch it be reflected inside the container and watch the logs
`docker run -p 8081:4000 -v ${pwd}:/site bretfisher/jekyll-serve` - serve a static website with jakyll