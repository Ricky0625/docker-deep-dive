# Images

## The TLDR

- *Image*, *Docker image*, *container image*, and *OCI iamge* all mean the same thing.
- A container image is **read-only** package that contains everything you need to run an application. It includes application code, application dependencies, a minimal set of OS constructs, and metadata.
- A single image can be used to start one or more containers.
- For those who are familiar with VMware, you can think of images as similar to VM templates.
- A VM template is like a stopped VM - a container image is like a stopped container.
- If you are a developer, you can think of them as similar to *classes*. You can create one or more objects from a class - you can create one or more containers from an image.
- You get container images by pulling them from a registry. The most common registry is Docker Hub.
- The *pull* operation downloads an image to your local Docker host where Docker can use it to start one or more containers.
- Images are made up of multiple layers that are stacked on top of each other and represented as a single object.

## The deep dive

- Images are like stopped containers.
- You can stop a container and create a new image from it.
- Images are considered *build-time* constructs, whereas containers are *run-time* constructs.

## The commands

### Images and containers

- Use `docker run` and `docker service create` commands to start one or more containers from a single image.
- Once you've started a container from an image, the two constructs become dependent on each other, and you cannot delete the image until the last container using it has been stopped and destroyed.

### Images are usually small

- The purpose of a container is to run a single application or service, meaning it only needs the code and dependencies of the app it's running, it doesn't need anything else. This means images are small and stripped of all non-essential parts.
- Images don't include a kernel, because containers share the kernel of the host they're running on.
- Images contain just enough OS.

### Pulling images

- Use `docker images` to check if your Docker host has any images in its local repository.
- Use `docker pull` to pull images from the registry,

### Image naming

When pulling an image, you have to specify the name of the image you're pulling. To fully understand the image naming, we need a bit of background on how images are stored.

### Image registries

- Images were stored in a centralized places called *registries*, or sometime called as *OCI registries*.
- The job of a registry is to securely store container images and make them easy to access from different environments.
- The most common one is Docker Hub.
- Image registries contain one or more image repositories. In turn, image repositories contain one or more images.

### Image naming and tagging

- To address images from official repositories, just do name and tag separated by a colon.

```shell
docker pull <repository>:<tag>

docker pull alpine:latest
docker pull redis:latest

docker pull mongo:4.2.24
// this will pull the image tagged as '4.2.24' from the official 'mongo' repository

docker pull busybox:glibc

docker pull alpine
// by default, it will pull the image tagged as 'latest'
```

- If a repositofy doesn't have an image tagged as what you specified, the command will fail.
- `latest` doesn't have any magical powers. Just because an image is tagged as `latest` does not gurantee it is the most recent image in the repository.
- A single image can have as many tags as you want

### Filtering the output of `docker images`

- Docker provides the `--filter` flag to filter the list of images returned by `docker images`.

The following example will only return dangling images:

```shell
docker image --filter dangling=true
```

- `dangling iamge` is one that is no longer tagged and appears in the listings as `<none>:<none>`.
- You can delete all dangling images on a system with the `docker image prune` command. If you add the `-a` flag, Docker will remove all unused images (those not in use by any containers).
- Docker currently supports the following filters:
	- `dangling`: Accepts `true` or `false`, and returns only dangling images (true), or non-dangling images (false).
	- `before`: Requires an image name or ID as argument, and returns all images created before it.
	- `since`: Same as above, but returns images created after the specified image
	- `label`: Filter images based on the presence of a lable or label nad value.
	- For other filtering, you can use `reference`.

```shell
# images that taggeed as "latest"
docker images --filter=reference="*:latest"
```

### Images and layers

- A Docker image is a collection of loosely-connected read-only layers where each layer comprises one or more files.
- When we are pulling an image, we can actually see the layers that make up an image. Each line in the output below that ends with "Pull complete" represents a layer in the image that was pulled.
	
	```shell
	$ docker pull ubuntu:latest
	latest: Pulling from library/ubuntu
	952132ac251a: Pull complete
	82659f8f1b76: Pull complete
	c19118ca682d: Pull complete
	8296858250fe: Pull complete
	24e0251a0e2c: Pull complete
	Digest: sha256:f4691c96e6bbaa99d...28ae95a60369c506dd6e6f6ab
	Status: Downloaded newer image for ubuntu:latest
	docker.io/ubuntu:latest
	```

- Another way to inspect the layer is with the `docker inspect` command.
- Some Dockerfile instructions (`ENV`, `EXPOSE`, `CMD`, `ENTRYPOINT`) add metadata to the image and do not create a layer.
- All Docker images start with a base layer, and as changes are made and new content is added, new layers are added on top.

### Sharing image layers

- Multiple images can, and do, share layers (efficiencies in space and performance).
- If you do `docker pull` with the `-a` flag, and if you are lucky enough, you might see this `Already exists` comment. These lines tell us that Docker is smart enough to recognize when it's being asked to pull an image layer that it already has a local copy of it.

### Deleting Images

- You can delete an image that you no longer need on your Docker host using `docker rmi`. `rmi` is short for remove image.
- Deleting an image will remove the image and all of its layers from your Docker host. This means it will no longer show up in `docker images` commands and all directories on the Docker host containing the layer data will be deleted. However, if an image layer is shared by another image, it won't be deleted until all images that reference it have been deleted.
- You can reference the image you want to delete using its ID. `docker rmi 44dd6f223004`
- You can list muliple images on the same command by separating them with whitespaces. `docker rmi 44dd6f223004 a4d3716dbb72`
- You need to stop and delete any containers before deleting the image.
- A handy shortcut for deleting all images on a Docker host:

	```shell
	docker rmi $(docker images -q) -f
	```

	- `-q` with `docker images` returns a list containing just the image IDs of all local images.
	- `-f` means force

## Chapter summary

- Container images contain everything needed to run an application as a container.
- Container includes just enough OS, source code files, dependencies, and metadata.
- Images are used to start containers and are similar to VM templates or OOP classes.
- Images are made up of one or more read-only layers, that when stacked together, make up the overall image.

