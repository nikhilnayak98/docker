# Useful Docker Stuff

### Installation:
1. `sudo apt update && sudo apt upgrade` -- update system
2. `sudo apt install docker.io` -- install docker
3. `systemctl status docker` -- check status
4. `sudo systemctl enable docker` -- if not enabled then enable so it starts on boot
5. `sudo systemctl start docker` -- if not started then start service
6. `sudo docker run hello-world` -- test to see if the default docker container works
7. `sudo usermod -aG docker $USER` -- add current user to docker group (so you don't need Sudo)
8. reboot machine and run `docker run hello-world` 

<br>

### Running Containers:
- `docker images` -- check installed docker images 
- `docker search ubuntu` -- search for the "ubuntu" docker image
- `docker pull ubuntu` -- download the "ubuntu" docker image 
- `docker run ubuntu` -- create a container for "ubuntu" docker image (if a container has no jobs/automatic services then the container will exit immeditately) 
- `docker run nginx` -- creates a container for "nginx" docker image (this will run because this docker image automatically runs the nginx service - has a job). If the container has not been installed locally then it will be automatically downloaded.
- `docker ps` -- lists docker containers currently running on the system 
- `docker ps -a` -- lists docker containers that have or are currently running on the system (all)
- `docker run -it ubuntu /bin/bash` -- creates a "ubuntu" docker container and opens a bash terminal for you to interact with it (press `<Ctrl-D>` to exit)
- can get the same with `docker run -it ubuntu` (`-it` = interactive terminal)


> by default, containers don't save changes (i.e. app installs) as they are not stateful; when they are stopped they are deleted. If you want to keep changes then they need to be part of the docker image (the blueprint)

<br>

### Making containers persistent:
- `docker run -it -d ubuntu` -- creates an interactive terminal for the "ubuntu" docker image and runs it in deamon mode (`-d`), which runs it in the background
- `docker ps` -> `docker attach <container_id>` -- attaches to the backgrounded docker image so you can interact with it (you don't have to type the whole container id, you can just type enough to distinguish it from the other containers)
- whilst in a container press `<Ctrl-P> + <Ctrl-Q>` to background the container and keep it running

<br>

### Accessing containerized apps:
> A docker entrypoint command (`/docker-entrypoint.sh`) is a command which runs automatically when a container starts. I will keep running and not return you back to your shell when you start a container (i.e. nginx)
- `docker run -it -d -p 8080:80 nginx` -- runs the "nginx" container in daemon mode, creates an interactive terminal, and maps local port 8080 to port 80 on the container. You can then connect to local port 8080 to access port 80 on the container, or the IP address of the machine docker is running on and port 8080)
- `docker run -it -d --restart unless-stopped -d 8080:80 nginx` -- same as before, but this time you can attach to the container, exit, and then the container will keep running. You have to explicitly stop the container to kill it. This is useful if you have a container which needs to be avaliable as it solves a purpose (i.e. serving a website).

<br>

### Creating docker image:
1. try to find a container which is as close to the container you want to create: 
	- `docker run -it ubuntu` -> `apt update && apt upgrade`
2. install packages you want:
	- `apt install apache2 nano`
3. check status of installed packages/services and get them running:
	- `/etc/init.d/apache2 status` -> `/etc/init.d/apache2 start`
4. disconnect from container and keep the changes made:
	- `<Ctrl-P> + <Ctrl-Q>` 
5. get the container's id and turn container into a re-usable image:
	- `docker ps`
	- `docker commit --change='ENTRYPOINT ["apachectl", "-DFOREGROUND"]	<container_id> lltv/apache-txt:1.0` -- creates a new container image called "lltv/apache-txt:1.0" with an entrypoint that starts apache
6. check image was created with `docker images`
7. stop the current container with `docker stop <container_id>`
8. run the new container image with `docker run -d -it -p 8080:80 lltv/apache-txt:1.0`

> to remove a docker image run `docker rm <container_id>` and `docker 

<br>

### Creating docker image from a docker file:
1. create a docker file with `nano Dockerfile`:
```docker
FROM ubuntu
MAINTAINER dev <dev@email.com>

# skip prompts
ARG DEBIAN_FRONTEND=noninteractive

# update packages
RUN apt update; apt dist-upgrade -y

# install packages
RUN apt install -y apache2 nano

# set entrypoint 
ENTRYPOINT apache2ctl -D FOREGROUND
```
2. build the docker image form the file:
	- `docker build -t lltv/apache-test:1.1 .` -- build "lltv/apache-test:1.1" from the docker file in the current working directory (`.`)
	- you need to make sure the docker file automatically builds the image without asking for any prompts (i.e. `ARG DEBIAN_FRONTEND=noninteractive` and `-y` apt option)

<br>

### Removing docker images:
1. remove docker containers associated with image:
	- `docker ps -a` 
	- `docker rm <container_id>`
2. remove docker image:
	- `docker images`
	- `docker rmi <image_id>`
