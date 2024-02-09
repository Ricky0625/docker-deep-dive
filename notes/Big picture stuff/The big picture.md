# 4: The big picture

## The Ops Perspective

When we install Docker, we get two major components:

1. The Docker client
2. The Docker engine (Docker daemon) - Implements the runtime, API and everything else required to run containers.

### Images

- Think of a Docker image as an object that contains an OS filesystem, an application, and all application dependencies.
- It's like a virtual machine template, which is essentially a stopped virtual machine.
- In the Docker world, an image is effectively a stopped container.

Getting images onto your Docker host is called *pulling*.

Example:

```shell
# pull the ubuntu:latest image
docker pull ubuntu:latest
```

- It's enough for now to know that an image contains enough of an operating system, as well as all the code and dependencies to run whatever application it's designed for.
- If we are pulling an `ubuntu` image, it's basically a stripped down version of the Ubuntu Linux filesystem and a few of the common Ubuntu utilities.
- If we pull an application container, such as `nginx:latest`, we'll get an image with a minimal OS as well as the code to run the app (NGINX).
- Each image gets its own unique ID. We can reference an image using `ID`s or names.

### Containers

Now that we have an image pulled locally, we can use the `docker run` command to launch a container from it.

```shell
docker run -it ubuntu:latest /bin/bash
```

- `docker run` tells Docker to start a new container
- `-it` flags tell Docker to make the container interactive and to attach the current shell to the container's terminal.
- `ubuntu:latest` tells Docker that we want the container to be based on this
- `/bin/bash` tells Docker which process we want to run inside of the container - a Bash shell.

Press `Ctrl-PQ` to exit the container without terminating it. This will land your shell back at the terminal of your Docker host.

To see all the running containers on your systems, use:

```shell
docker ps
```

### Attaching to running containers

- Attach your shell to the terminal of a **running container** with the `docker exec` command.

```shell
docker exec -it <container \-name/container-id> <command/app>
```

- `-it` flags to attach our shell to the container's shell

Once you run this, you will be logged into the container again.

Stop the container and kill it using the `docker stop` and `docker rm` commands.

```shell
docker stop <container-name/container-id>
docker rm <container-name/container-id>

# to verify that the container was successfully deleted
docker ps -a
```

Verify that the container was successfully deleted by running the `docker ps` command with the `-a` flag. Adding `-a` tells Docker to list all containers, even those in the stopped state.

## The Dev Perspective

- Containers are all about the apps.
- Dockerfile is a plain text document that tells Docker how to build the app and dependencies into a Docker image.

Example of a Dockerfile:

```Dockerfile
FROM alpine
LABEL maintainer="nigelpoulton@hotmail.com"
RUN apk add --update nodejs nodejs-npm
COPY . /src
WORKDIR /src
RUN npm install
EXPOSE 8080
ENTRYPOINT ["node", "./app.js"]
```

Each line represents an instruction that Docker uses to build the app into an image.

Use `docker build` command to create a new image using the instructions in the Dockerfile.

```shell
docker build -t test:latest .
```

- `-t` is to name the image and optionally a tag. (format: "name:tag")
- `.` means you will use the Dockerfile in the local directory

Once the build is complete, check to make sure that the new `test:latest` iamge exists on your host using `docker images`. After that you can run the container as you wish and test the app.

Example:

```shell
docker run -d \
--name web1 \
--publish 8080:8080 \
test:latest
```

- `-d`: Run container in background and print container ID
- `--name`: Assign a name to taht container
- `--publish`: Publish a container's port(s) to the host

At this point, we've copied some application code from a remote Git repo, built it into a Docker image and ran it as a container. This process is called "containerizing an app".

## Chapter Summary

- **Ops**: Download a Docker image, launched a container from it, logged into the container, executed a command inside of it, then stopped and deleted the container.
- **Dev**: Containerized a simple application by pulling some source code from GitHub and building it into an image using instructions in a Dockerfile. You then ran the containerized app.
