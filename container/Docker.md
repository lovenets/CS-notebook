# What Is Docker

Docker is a platform for developers and sysadmins to **develop, deploy, and run** applications with containers. The use of Linux containers to deploy applications is called *containerization*.

The goal of Docker is "Build once, run everywhere."

# Basic Concepts

## Image

An **image** is an executable package that includes everything needed to run an application--the code, a runtime, libraries, environment variables, and configuration files.

## Container

A **container** is a runtime instance of an image--what the image becomes in memory when executed (that is, an image with state, or a user process).

## Containers VS VM

A **container** runs *natively* on Linux and shares the kernel of the host machine with other containers. It runs a discrete process, taking no more memory than any other executable, making it lightweight.

By contrast, a **virtual machine** (VM) runs a full-blown “guest” operating system with *virtual* access to host resources through a hypervisor. In general, VMs provide an environment with more resources than most applications need.

![](https://docs.docker.com/images/Container%402x.png)

![](https://docs.docker.com/images/VM%402x.png)

# Common Docker Commands

*On Linux, all commands should be executed with `sudo` unless you do some [configurations]([https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04#step-2-%E2%80%94-executing-the-docker-command-without-sudo-(optional)]).*

```bash
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run

## Build Docker image
docker build

## Docker images
docker image ls

## Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq
docker container stop
docker container rm
```

# Dockerfile

`Dockerfile` defines what goes on in the environment inside your container. Access to resources like networking interfaces and disk drives is virtualized inside this environment, which is isolated from the rest of your system, so you need to map ports to the outside world, and be specific about what files you want to “copy in” to that environment. 

`Dockerfile`is composed of a bunch of *instructions*:

```dockerfile
# Comment
# The instruction is not case-sensitive. However, convention is for them to be UPPERCASE to distinguish them from arguments more easily.
INSTRUCTION arguments
```

Docker runs instructions in a `Dockerfile` in order. A `Dockerfile` **must start with a `FROM` instruction**. The `FROM` instruction specifies the [*Base Image*](https://docs.docker.com/engine/reference/glossary/#base-image) from which you are building. `FROM` may only be preceded by one or more `ARG` instructions, which declare arguments that are used in `FROM` lines in the `Dockerfile`.

## Common Instructions

### RUN

The `RUN` instruction will execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the `Dockerfile`.

```dockerfile
# shell form, the command is run in a shell, which by default is /bin/sh -c on Linux or cmd /S /C on Windows
RUN <command>
# exec form
RUN ["executable", "param1", "param2"]
```

### CMD

There can only be one `CMD` instruction in a `Dockerfile`. If you list more than one `CMD` then only the last `CMD`will take effect.

**The main purpose of a CMD is to provide defaults for an executing container.** These defaults can include an executable, or they can omit the executable, in which case you must specify an `ENTRYPOINT` instruction as well.

```dockerfile
CMD ["executable","param1","param2"] # (exec form, this is the preferred form)
CMD ["param1","param2"] #(as default parameters to ENTRYPOINT)
CMD command param1 param2 # (shell form)
```

### EXPOSE

The `EXPOSE` instruction informs Docker that the container listens on the specified network ports at runtime. You can specify whether the port listens on TCP or UDP, and the default is TCP if the protocol is not specified.

However, `EXPOSE` instruction does not actually publish the port. To actually publish the port when running the container, use the `-p` flag on `docker run` to publish and map one or more ports, or the `-P` flag to publish all exposed ports and map them to high-order ports.

```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

### ENV

The `ENV` instruction sets the environment variable `<key>` to the value `<value>`. This value will be in the environment for all subsequent instructions in the build stage and can be [replaced inline](https://docs.docker.com/engine/reference/builder/#environment-replacement) in many as well.

```dockerfile
ENV <key> <value>
ENV <key>=<value> ...
```

### ADD

The `ADD` instruction copies new files, directories or remote file URLs from `<src>` and adds them to the filesystem of the image at the path `<dest>`.

Multiple `<src>` resources may be specified but if they are files or directories, their paths are interpreted as relative to the source of the context of the build.

Each `<src>` may contain wildcards and matching will be done using Go’s [filepath.Match](http://golang.org/pkg/path/filepath#Match) rules.

```dockerfile
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"] # (this form is required for paths containing whitespace)
```

### COPY

The `COPY` instruction copies new files or directories from `<src>` and adds them to the filesystem of the container at the path `<dest>`.

Multiple `<src>` resources may be specified but the paths of files and directories will be interpreted as relative to the source of the context of the build.

Each `<src>` may contain wildcards and matching will be done using Go’s [filepath.Match](http://golang.org/pkg/path/filepath#Match) rules.

```dockerfile
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"] # (this form is required for paths containing whitespace)
```

**COPY vs ADD**

They both let you copy files from a specific location into a Docker image.

`COPY` only lets you copy in a local file or directory from your host (the machine building the Docker image) into the Docker image itself.

`ADD` lets you do that too, but it also supports 2 other sources. First, you can use a URL instead of a local file / directory. Secondly, you can extract a tar file from the source directly into the destination.

*If you’re copying in local files to your Docker image, always use `COPY` because it’s more explicit.*

### ENTRYPOINT

An `ENTRYPOINT` allows you to configure a container that will run as an executable.

```dockerfile
ENTRYPOINT ["executable", "param1", "param2"] # (exec form, preferred)
ENTRYPOINT command param1 param2 # (shell form)
```

### WORKDIR

The `WORKDIR` instruction sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the `Dockerfile`. If the `WORKDIR` doesn’t exist, it will be created even if it’s not used in any subsequent `Dockerfile` instruction.

```dockerfile
WORKDIR /path/to/workdir
```

# Example: Containerizing Go Web App

## 1. Write A Simple Go Web App

Create a Go file named `app.go`:

```go
package main

import (
    "net/http"
    "log"
)

func main() {
    http.HandleFunc("/", func(resp http.ResponseWriter, req *http.Request) {
        resp.Write([]byte("hello, docker\n"))
    })
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        log.Fatal(err)
    }
}
```

## 2. Write A Dockerfile

Create a `Dockerfile`in the same directory.

```dockerfile
FROM golang:1.12.0-alpine3.9
# We create an /app directory within our
# image that will hold our application source
# files
RUN mkdir /app
# We copy everything in the root directory
# into our /app directory
COPY . /app
# We specify that we now wish to execute 
# any further commands inside our /app
# directory
WORKDIR /app
# we run go build to compile the binary
# executable of our Go program
RUN go build -o app
# Our start command which kicks off
# our newly created binary executable
CMD ["./app"]
```

## 3. Build Docker Image

Run `docker build -t hello-docker-go .`command. Note that last argument in our example is a dot which means we want to execute this command in current directory.

We can now verify that our image exists on our machine by typing `docker image`:

```bash
$ docker image ls
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
hello-docker-go                                  latest              3f9244a1a240        2 minutes ago       355MB
```

## 4. Run Docker Container

Run following command:

```bash
$ docker run -p 8080:8080 -it hello-docker-go
```

- *-p 8080:8080* - Map port 8080 on our local machine to port 8080 which our Go web app will listen to within container.
- *-it* - This flag specifies that we want to run this image in interactive mode with a `tty` for this container process.

If we want to have it run permanently in the background, replace *-it* with *-d* to run this container in detached mode.

We can browse running containers by executing `docker ps`.

Using volumes is a good way to persist data on host file system. The following example mounts the volume `myvol2` into `/app/` in the container.

```bash
$ docker run -p 8080:8080 -d --mount source=myvol2,target=/app hello-docker-go
```

# Service

In a distributed application, different pieces of the app are called “services”. Services are really just “containers in production.” A service only runs one image, but it codifies the way that image runs—what ports it should use, how many replicas of the container should run so the service has the capacity it needs, and so on.

## Configure Service

Scaling a service changes the number of container instances running that piece of software, assigning more computing resources to the service in the process and so on. It’s very easy to define, run, and scale services with the Docker platform -- just write a `docker-compose.yml` file.

Here is an example:

```yaml
version: "3"
services:
  web:
    # replace username/repo:tag with 
    # your name and image details to 
    # pull the image we want to run as a service 
    image: username/repo:tag
    deploy:
      # 5 instances
      replicas: 5
      resources:
        limits:
          # limit each instance to use 
          # at most 10% of a single core 
          # of CPU time and 50MB of RAM
          cpus: "0.1"
          memory: 50M
      restart_policy:
        # restart immediately if one fails
        condition: on-failure
    ports:
      # map port 4000 on the host
      # to web's port 80
      - "4000:80"
    networks:
      # instruct web's containers 
      # to share port 80
      # via a load-balanced network called webnet
      - webnet
networks:
  # define the webnet network with default settings which is 
  # load-balanced overlay network
  webnet:
```

## Run App as Service

```bash
$ docker swarm init
$ docker stack deploy -c docker-compose.yml appname
```
Then we can get the service ID:
```bash
$ docker service ls
```

Look for output for the `web` service, prepended with your app name. The service ID is listed as well, along with the number of replicas, image name, and exposed ports.

A single container running in a service is called a *task*. Tasks are given unique IDs that numerically increment, up to the number of replicas you defined in docker-compose.yml. List the tasks for your service:

```bash
$ docker service ps servicename
```
## Scale the App

We can scale the app easily by modifying `docker-compose.yml` and re-running the `docker stack deploy` command. 

Docker performs an in-place update, no need to tear the stack down first or kill any containers.

## Take Down the App And the Swarm


Take the app down with docker stack rm:
```bash
$ docker stack rm appname
```
Take down the swarm.
```bash
$ docker swarm leave --force
```

# Swarm

A swarm is a group of machines that are running Docker and joined into a cluster. The machines in a swarm can be physical or virtual. After joining a swarm, they are referred to as **nodes**.

Swarm managers are the only machines in a swarm that can execute your commands, or authorize other machines to join the swarm as **workers**.