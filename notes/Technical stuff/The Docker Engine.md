# 5: The Docker Engine

## The TLDR

- The core software that runs and manage containers. We often refer it as *Docker*.
- Think of Docker engine as being like VMware, VirtualBox.
- Docker Engine is like a car engine - both are modular and created by connecting many small specialized parts.
- The major components that make up the Docker engine are:
	- Docker daemon
	- build systems: `containerd`, `runc`
	- various plugins such as networking and volumes

## The Deep Dive

### `runc`

- A small, lightweight CLI wrapper for libcontainer (read up more if you are interested).
- Has a single purpose in life - create containers.

### `containerd`

- Its sole purpose in life is to manage container lifecycle operations such as `start`, `stop`, `pause`, `rm` ...
- It was intended to be smal, lightweight, and designed for a single task in life - container lifecycle operations. But over time, it has branched our and taken on more functionality like image pulls, volumes and networks.

## The commands

### Starting a new container (example)

The most common way of starting containers is using the Docker CLI. The following `docker run` command will start a simple new container based on the `alpine:latest` image.

```shell
docker run --name ctrl -it alpine:latest sh
```

Behind the scences, when you type commands like this into the Docker CLI, the Docker client converts them into the appropriate API payload and POSTs them to the API endpoint exposed by the Docker daemon.

Once the daemon receives the command to create a new container, it makes a call to `containerd`.

> The daemon communicates with `containerd` via a CRUD-style API.

`containerd` itself cannot create containers. It uses `runc` to do that. It converts the required Docker image into an OCI bundle and tells `runc` to use this to create a new container.

`runc` interfaces with the OS kernel to pull together everything to create a container. The container process is started as a child process of `runc`, and as soon as it starts, `runc` wil exit.

![Summary](https://miro.medium.com/v2/resize:fit:786/format:webp/0*9W7b21pk0aRxVPBj.jpg)
