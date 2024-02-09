# Containers

## The TLDR

- A container is the runtime instance of an image.
- The difference between VM and a container is that containers are smaller, faster, and more portable.
- Run `docker run` command to start a container.
- Containers run until the main app exits.
- You can stop a running container with `docker stop` and restart it with `docker start`.
- To get rid of a container forever, you have to explicitly delete it with `docker rm`.

## The deep dive

### Containers vs VMs

- Containers and VMs both need a host (laptop, a bare metal server in your data center, an instance in the public cloud) to run.
- VM needs to run on a hypervisor. Hypervisor claims all **physical resources** such as CPU, RAM, storage, and network cards. It then carves these hardware resources into virtual constructs that look smell and feel exactly like the real thing. It then packages them into a software construct called a virtual machine.
- In the case of containers is different. We need to install a container engine such as Docker. The container engine then carves-up the **OS resources** (process tree, filesystem, network stack, etc.) and packages them into virtual operating systems called *containers*. Each container works just like a real OS. Inside of each container we run an application.
- At a high level, hypervisors perform **hardware virtualization**, they made physical hardware resources into virtual versions called VMs. For containers, it perform **OS virtualization**, which made OS resources into virtual versions called containers.

### The VM Tax

- VM model carves low-level hardware resources into VM with individual OS. Each VM needs its own OS to claim, intialize, and manage all of those virtual resources.
- Each OS consumes CPU, RAM, and storage. Some even need their own licenses, as well as people to maintain and update them.
- All of the things that has been mentioned so far is called, OS tax, or VM tax. Every OS is steeling resources you'd rather assign to applications.
- Containerse have a single OS tax, as they share host's OS, resulting in more efficient resource utilization.
- Containers start faster than VMs because they only need to start the application, with the kernel already running.
- Container modle is much more leaner, allowing more applications on fewer resources and reducing licensing and admin costs.

## The Commands

### Checking that Docker is running

- Check if Docker is running: `docker version`. Should have the Client and Server section.

### Starting a simple container

- `docker run`. Ex: `docker run -it ubuntu:latest /bin/bash`
- `-it` makes the container interactive and attach it to your terminal.

### Container processes

- If we run the command like the one above, that makes the Bash shell be the one-and-only process running inside the container. You can see this by running `ps -elf`.
- The first process in the list, with `PID 1`, is the Bash shell we told the container to run.
- If we type exit while logged in to the container, this will terminate the Bash process and the whole container will exit. This is because a container cannot exist without its designated main process. It's the same for Linux and Window containers - **killing the main process in the container will kill the container**.
- `Ctrl-PQ` to exit container without terminating its main process.
- Re-attach your terminal to the container with the `docker exec` command. Ex: `docker exec -it  e37f24dc7e0a bash`
- If you look at `ps -elf` now, you will see two Bash processes. This is because the `docker exec` command created a new Bash process and attached to that. This means typing `exit` in this shell will not terminate the container, because the original Bash process will continue running.

### Container lifecycle

- Create new container. Ex: `docker run --name percy -it ubuntu:latest /bin/bash`
- `docker run` with `-d` means starting a container and don't attach it to your terminal. Run in background. `-it` and `-d` can't be used at the same time.
- `docker run` with `-p <host-port:container-port>` maps port 80 on the Docker host to port 8080 inside the container. This means that traffic hitting the Docker host to port 80 will be directed to port 8080 inside of the container.
- `Ctrl-PQ` to exit the container without killing it.
- `docker stop` to stop the container and put in on vacation. Change container status as `Exited`.
- `docker ps` to list all running containers.
- `docker ps -a` show all containers, including those that are stopped.
- `docker start` to bring it back from vacation.
- `docker exec` to re-attach your terminal to the container.
- If you create a file or modify contents inside a container and stop it, those modified or created content won't get destroyed. This proves the persistent nature of containers. But you have to understand these two things:
	1. The data created is stored on the Docker hosts local filesystem. If the Docker host fails, the data will be lost.
	2. Containers are designed to be **immutable objects** and it's not a good practice to write data to them.
- For the point above, Docker provides *volumes*. These exist outside of containers but can be mounted into them.
- The data inside *volume` will persist even after the container has gone, since it's outside of the container.
- It's best to stop a container before deleteing it.

### Stopping containers gracefully

- `docker stop` sends a **SIGTERM** signal to the main application process inside the container (PID 1). This is a request to terminate and gives the process a chance to clean things up and gracefully shut itself down. If it's still running after 10 seconds, it will be issued a **SIGKILL** which terminates it with force.
- `docker rm <container> -f` doesn't bother asking nicely with a **SIGTERM**, it goes straight to the **SIGKILL**.

### Self-healing containers with restart policies

- It's often a good idea to run containers with a restart policy. This allows the local Docker engine to auto restart failed containers.
- Restart policies are applied per-container.
- The existing restart policies: `always`, `unless-stopped`, `on-failure`.
- `always`: Always restarts a failed container, unless it's explicitly stopped.

### Tidying up

- Get rid of every running container on a Docker host. This procedure will forcibly destroy all containers without giving them a chance to clean up. This should never be performed on production systems or systems running important containers.
	``` shell
	docker rmi $(docker ps -aq) -f
	```
