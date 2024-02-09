# Multi-container apps with Compose

## The TLDR

- Modern cloud-native apps are made of multiple smaller services that interact to form a useful app. This is the *microservices* pattern.
- A microservices app might have the following seven independent services that work together:
	- Web front-end
	- Ordering
	- Catalog
	- Back-end datastore
	- Logging
	- Authentication
	- Authorization
- Deploying and managing small microservices like these can be hard. This is where *Compose* comes in to play.
- *Compose* lets you describe everything in a declarative config file without typing all the long docker commands.
- You can manage your app's lifecycle with a simple set of commands.

## Deploying apps with Compose - The Deep Dive

### Compose background

- It was called *Fig* originally, and was a built tool by a company called Orchard.
- Behind the scenes, Fig would read the YAML file and call the appropriate Docker commands to deplay and manage it.
- The tool was so good that Docker, Inc. acquired Orchard and re-branded Fig as Docker Compose.
- Later down the line, `docker compose` was added to the `docker CLI` with its sub-command.

### Compose files

- Compose uses YAML files to define microservices applications.
- The default name for Compose YAML file is `compose.yaml` or `compose.yml`. You can use the `-f` flag to specify custom filenames.

Example:

```yaml
services:
	web-fe:
		build: .
		command: python app.py
		ports:
			- target: 8080
			  published: 5001
		networks:
			- counter-net
		volumes:
			- type: volume
			  source: counter-vol
			  target: /app
	redis:
		image: "redis:alpine"
		networks:
			counter-net:

networks:
	counter-net:

volumes:
	counter-vol:
```

The file has 3 top-level keys:

- `services`: This is where we define application microservices. This example defines two: a web frontend called `web-fe` and an in-memory cache called `redis`. Both of these microservices will be deployed to its own container.
- `networks`: Tells Docker to create new networks. By default, modern versions of Compose create `overlay` networks. More on this in the Docker network chapter.
- `volumes`: Tell Docker to create new volumes.

Other top-level keys exist, such as `secrets` and `configs`.

- It's important to know that Compose will deploy each of these as its own container and will use the name of the keys in the container names. In the example above, it defined two keys: `web-fe` and `redis`. This means Compose will deploy two containers, one will have `web-fe` in its name and the other will have `redis`.

Within the definition of the `web-fe` service, the following instructions were given:

- `build`: Tells Docker to build a new image using the `Dockefile` in the current directory (`.`).
- `command`: Tells Docker to run a Python app like so: `python app.py` in every container for this service. The `app.py` must exist in the image, and the image must have Python installed. The Dockerfile should take cares of both of these requirements.
- `ports`: Tells Docker to map port `8080` inside the container (`target`) to port `5001` on the host (`published`). This means traffic hitting the Docker host on port `5001` will be redirected to port `8080` on the container. The app inside the container listens on port `8080`.
- `networks`: Tells Docker which network to attach the service's containers to. The network should already exist or be defined in the **`networks` top-level key**.
- `volumes`: Tells Docker to mount the `counter-vol` volume (`source:`) to `/app` (`target:`) inside the container. The `counter-vol` volume needs to already exist or be defined in the **`volumes` top-level key** in the file.

Summary:

Compose will instruct Docker to deploy a single standalone container for the `web-fe` microservice. It will be based on an image built from a Dockerfile in the same directory as the Compose file. This image will be started as a container and run app.py as its main app. It will attach to the `counter-net` network, expose itself on port `5001` on the host, and mount a volume to `/app`.

> the `command: python app.py` is not needed as it's already defined in the Dockerfile. You can also use Compose to override instructions set in Dockerfile.

For `redis` service is simpler:

- `image`: This tells Docker to start a standalone container called `redis` based on the `redis:alpine` image. This image will be pulled from DockerHub.
- `networks`: This container will also be attached to the `counter-net` network.

As both of the services will be deployed onto the same `counter-net` network, they'll be able to resolve each other by name.

### Deploying apps with Compose

To use Compose to bring the app up, run the following commands:

```shell
docker compose up &
```

`docker compose up` is the most common way to bring up a Compose app. It builds or pulls all required images, creates all required networks and volumes, and starts all required containers.

> Notice that we didn't need to specify the name nor location of the Compose file as it's called `compose.yaml` and in the local directory. If your compose file is another name and at different location, use the `-f` flag. Example: `docker compose -f prod-equus-basse.yml up &`.

A few more options:

- `--detach`: bring the app up in the background
- `&`: gives us the terminal window back but forces Compose to output all messagse to the terminal window

With the app is build and running, we can use normal `docker` commands to view the images, containers, networks, and volumes that Compose created.

The images built by Compose will follow the following naming convention:

```text
<project-name>-<name-specified-in-the-compose>

Project name is the name of the directory with the Compose file in it. All resources created by Compose will follow this naming convention.

For example: multi-container-web-fe
```

As for the naming convention of containers, it will follow such naming convention:

```text
<project-name>-<name-specified-in-the-compose>-<numeric-suffix>

The numeric suffix indicates the instance number.

For example: multi-container-web-fe-1
```

To check the networks and volumes:

```shell
docker network ls
docker volume ls
```

### Managing apps with Compose

- Bring app up: `docker compose up`.
- Bring app down: `docker compose down`.

> When we bring the app down, it automatically delete the containers (microservices) and the network, but not **volumes**. This is because volumes are intended to be long-term persistent data stores and are entirely decoupled from application lifecycles.

- Bring app down along with all the associated volumes: `docker compose down --volumes`.
- Images that were built or pulled as part of the `docker compose up` operation will also remain on the system.
- Bring app down along with all the images built or pulled when starting the app: `docker compose down --rmi`.
- Bring app up but in background: `docker compose up --detach`
- List the containers created by a Docker Compose project. Show the status of containers defined in your YAML file. `docker compose ps`.
- List the processes running inside of each service (container): `docker compose top`.
- Stop the app without deleting its resources: `docker compose stop`.
- Delete a stopped Compose app with `docker compose rm`. This will delete the containers and networks but not the volumes or images. Nor will it delete the application source code in your project's build context directory.
- With the app in the stopped state, restart it with the `docker compose restart`.
- **Stop and delete** the app with this single command. This will also delete any volumes and images used to start the app. `docker compose down --volumes --rmi all`.
