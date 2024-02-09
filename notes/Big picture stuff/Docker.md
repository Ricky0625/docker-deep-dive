# 2: Docker

## Docker - The TLDR

- A software that runs on Linux and Windows
- Creates, manages, and can even orchestrate containers.
- Docker, Inc. is the comopany that created the technology.

## The Docker technology

- Three things to be aware of when referring to Docker as a technology:
	1. The runtime
	2. The daemon (a.k.a engine)
	3. The orchestrator

![Three layers](https://static.packt-cdn.com/products/9781835081709/graphics/Images/figure2-2.png)

### The runtime

- Operates at the lowest level and is responsible for starting and stopping containers.
- Docker implements a tiered runtime achitecture with high-level and low level runtimes that work together.
- `runc`, the low level runtime. Its job is to interface with the underlying OS and start and stop containers. Every container was created and started by an instance of `runc`.
- `containerd`, the high level runtime. It manage a container's lifecycle (pulling images, and managing `runc` instances.

### Docker daemon

- Performs higher-level tasks such as exposing the Docker API, managing images, managing volumes, managing networks, and more.
- Abstracts the lower levels.
