# Into Docker

Reference

* [Docker Tutorial by Code Academy](https://www.youtube.com/watch?v=pGYAg7TMmp0)

In daily work, we use [Buildkite](https://buildkite.com/home#features) to build environments before depolyment. I read Docker docs before, but I never have chance to put it into practice. This time, I'm gonna learn by using it in a side project.

### My understanding

The big picture is Docker builds up everything you need to deploy \(code, environments, .etc\) into an image, which can be run everywhere. The basic idea is use Dockerfile to create image, and run the image into a container.

### Usage

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



