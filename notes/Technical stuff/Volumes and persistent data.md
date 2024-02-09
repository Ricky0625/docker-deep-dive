# Volumes and persistent data

There are two main types of data: persistent and non-persistent.

Persistent data is data that you need to keep, non-persistent is data that you don’t need to keep. By default, all containers get a layer of writable non-persistent storage that lives and dies with the container — we call this local storage and it’s ideal for non-persistent data. However, if your containers create data that you need to keep, you should store the data in a Docker volume.

Docker volumes are first-class objects in the Docker API and managed independently of containers with their own docker volume sub-command. This means that deleting a container will not delete the volumes it was using.

Third party volume plugins can provide Docker access to specialised external storage systems. They’re installed from Docker Hub with the docker plugin install command and are referenced at volume creation time with the -d command flag.

Volumes are the recommended way to work with persistent data in a Docker environment.

### 1. **Volume Basics:**

- **Definition:**
  - A volume is a specially designated directory within one or more containers.
  - It exists outside of the container filesystem, making data persistent even if the container is stopped or removed.

- **Purpose:**
  - Volumes are used to store and share data between containers or between the host and containers.
  - Common use cases include database storage, configuration files, and shared application data.

### 2. **Creating Volumes:**

- **Docker CLI:**
  - You can create a named volume using the `docker volume create` command:
    ```bash
    docker volume create myvolume
    ```

- **Docker Compose:**
  - In a `docker-compose.yml` file:
    ```yaml
    version: '3'
    services:
      web:
        image: nginx
        volumes:
          - myvolume:/app/data

    volumes:
      myvolume:
    ```

### 3. **Mounting Volumes:**

- **Docker CLI:**
  - When running a container, you can mount a volume using the `-v` option:
    ```bash
    docker run -v myvolume:/app/data myimage
    ```

- **Docker Compose:**
  - In a `docker-compose.yml` file, use the `volumes` section:
    ```yaml
    version: '3'
    services:
      web:
        image: nginx
        volumes:
          - myvolume:/app/data

    volumes:
      myvolume:
    ```

### 4. **Types of Volumes:**

- **Named Volumes:**
  - Created and managed by Docker, identified by a name.
  - Persist even if no containers are using them.

- **Anonymous Volumes:**
  - Created and managed by Docker, not identified by a name.
  - Useful for temporary or disposable data.

- **Host Bind Mounts:**
  - Mounts a specific directory from the host machine into the container.
  - Useful for development or when you need direct access to host files.

### 5. **Inspecting and Managing Volumes:**

- **List Volumes:**
  ```bash
  docker volume ls
  ```

- **Inspect a Volume:**
  ```bash
  docker volume inspect myvolume
  ```

- **Remove a Volume:**
  ```bash
  docker volume rm myvolume
  ```

### 6. **Default Volume Mounts:**

- Certain directories inside a container are automatically created as volumes, such as `/var/lib/mysql` in a MySQL container.

### 7. **Container-to-Container Volumes:**

- Multiple containers can share the same volume, allowing them to communicate and share data.

### 8. **Volume Drivers:**

- Docker supports various volume drivers for different use cases, such as NFS for network-attached storage.

### 9. **Data Migration and Backup:**

- Volumes make it easier to migrate and back up data, as the data resides outside the container.

### 10. **Permissions and Ownership:**

- Be cautious about permissions and ownership of data within volumes to ensure proper access by containers.

Volumes in Docker are a powerful feature that enables data persistence and sharing among containers, making them essential for many production applications. Understanding how to create, manage, and use volumes will enhance your ability to work effectively with Docker containers.

## More commands

- `docker volume create` is the command to create new volumes. By default, volumes are created with the local driver but you can use the `-d` flag to specify a different driver. 
- `docker volume ls` will list all volumes on the local Docker host. 
- `docker volume inspect` shows detailed volume information. Use this command to see many interesting volume properties, including where a volume exists in the Docker host’s filesystem.
- `docker volume prune` will delete **all** volumes that are not in use by a container or service replica. **Use with caution!**
- `docker volume rm` deletes specific volumes that are not in use.
