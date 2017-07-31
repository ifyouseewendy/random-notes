# Into Docker

In daily work, we use [Buildkite](https://buildkite.com/home#features) to build environments before depolyment. I read Docker docs before, but I never have chance to put it into practice. This time, I'm gonna learn by using it in a side project.

The big picture is Docker builds up everything you need to deploy \(code, environments, .etc\) into an image, which can be run everywhere. The basic idea is use Dockerfile to create image, and run the image into a container.

### Basic Usage

Reference

* [Docker Tutorial by Code Academy](https://www.youtube.com/watch?v=pGYAg7TMmp0)

Basic commands

```sh
$ docker run <image>      # runs image to a container
$ docker ps [-a]          # which include stopped containers)
$ docker start <name|id>  # container
$ docker stop <name|id>   # container
$ docker rm <name|id>     # container
```

On your local machine, eg. OSX, you need a virtual machine \(boot2docker\) to load Docker.

```
$ boot2docker up           # boot docker VM
$ $(boot2docker shellinit) # make docker point to docker VM
```

After working on code, write a `Dockerfile` to set up your environment in docker

```
FROM nginx

RUN mkdir /etc/nginx/logs & touch /etc/nginx/logs/static.log

ADD ./nginx.conf /etc/nginx/conf.d/default.conf
ADD /src /www
```

Ok, build up

```
$ docker build -t learncodeacademy/static-nginx .
$ docker push learncodeacademy/static-nginx
```

On your server, eg. Digital Ocean droplet with Docker provisioned

```
$ docker login
$ docker run -d -p 80:80 —-name static learncodeacademy/static-nginx # {server_port}:{container_port}
```

### Commands

Reference

* [Docker Official Samples](https://docs.docker.com/samples/)
* [Docker Doc Get Started](https://docs.docker.com/get-started/)
* [Docker for Beginners](https://github.com/docker/labs/tree/master/beginner/)
* [Host path of volume](https://forums.docker.com/t/host-path-of-volume/12277)

**Image**

```
$ docker images -f dangling=true -q # List all the dangling images
$ docker rmi $(docker images -f dangling=true -q)

$ docker inspect alpine
$ docker tag <image> username/repository:tag  # Tag <image> for upload to registry
$ docker push username/repository:tag            # Upload tagged image to registry
$ docker run username/repository:tag                   # Run image from a registry
```

**Dockerfile**

* _RUN_ is used to build up the Image you're creating. For each RUN command, Docker will run the command then create a new layer of the image. This way you can roll back your image to previous states easily.
* _CMD_ defines the commands that will run on the Image at start-up. Unlike a RUN, this does not create a new layer for the Image, but simply runs the command. There can only be one CMD per a Dockerfile/Image. If you need to run multiple commands, the best way to do that is to have the CMD run a script.

**Container**

```
$ docker build -t ruby-script .
$ docker ps -a
$ docker rm $(docker ps -a -q)
$ docker run -it --rm ruby-script # -it means interactive tty

$ docker run --name static-site -e AUTHOR="Your Name" -d -P dockersamples/static-site
$ docker run --name static-site-2 -e AUTHOR="Your Name" -d -p 8888:80 dockersamples/static-site
# -P will publish all the exposed container ports to random ports on the Docker host
# -p will publish a container’s port to the host's
# -e is how you pass environment variables to the container
# -d will create a container with the process detached from our terminal

$ docker exec -it <mycontainer> bash
```

**Docker Compose**

```
$ docker-compose config # Validate and view the Compose file
$ docker-compose build
$ docker-compose up
$ docker-compose run #{app}
```

**Docker Machine**

```
$ docker-machine ls
$ docker-machine create --driver virtualbox default
$ eval "$(docker-machine env default)” # Set the default machine as the default one
$ docker-machine ip default
```

**Docker Deploy**

```
$ docker swarm init
$ docker stack deploy --compose-file docker-stack.yml #{service}
$ docker stack services #{service}
```

### Into wild

I'm making a side project called star reminder, so I try to put my docker knowledge into practice by dockerizing the project.

* Create a `Dockerfile` to instruct building of images and containers.
* Create a `docker-compose.yml` to orchestrate two services, script and redis.

Currently, the workflow of deployment would be: 

1. Local: Push changes to Github
2. Local: Commit changes to image and push to Dockerhub
3. Server: Pull changes from Github
4. Server: Pull changes from Dockerhub and `docker-compose build`
5. Server: `docker-compose up &`

I didn't choose cloud providers with Docker support \(Digital Ocean, AWS\) to deploy, as I want to know how to make it work on a typical or traditional server, which in my case is a Linode server. My understanding would be, to add the server as a docker machine which I could manipulate in my local just as the virtual machines. But, it seems there is not a valid linode `docker-machine` driver. \([taoh/docker-machine-linode](https://github.com/taoh/docker-machine-linode) I could not install this on my server by go errors\). What I'm going to investigate is how to add a remote server as a docker machine to your swarm. Hopefully this should be the right workflow:

1. Local: Commit changes to image and push to Dockerhub
2. Local: Use `docker-machine` or `docker stack deploy` to provision changes to remote docker machine.



