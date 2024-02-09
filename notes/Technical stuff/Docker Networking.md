# Docker Networking

1. **Default Network:**
   - When you run a container without specifying a network, it's connected to the default bridge network.

2. **Bridge Network:**
   - The default network type in Docker.
   - Containers on the same bridge network can communicate with each other.
   - Each container connected to a bridge network gets an IP address from the bridge's subnet.

3. **Host Network:**
   - Containers share the network namespace with the host machine.
   - Containers directly use the host's network stack.

4. **None Network:**
   - Containers run in isolated mode with no network access.
   - Useful for scenarios where you want to run a container without any network connectivity.

5. **Overlay Network (Swarm Mode):**
   - Used in Docker Swarm mode for multi-host communication.
   - Facilitates communication between containers running on different machines in a swarm.

6. **Macvlan Network:**
   - Allows you to assign a MAC address to a container.
   - Useful for scenarios where containers need to have their own unique IP addresses on the network.

7. **Custom Bridge Networks:**
   - You can create custom bridge networks with specific configurations.
   - Provides better isolation and control over container communication.

8. **Docker Compose Networking:**
   - Define networks in a `docker-compose.yml` file using the `networks` section.
   - Specify the network for each service using the `networks` property.

9. **Container DNS:**
   - Containers on the same network can typically discover each other using container names as hostnames.

10. **Network Inspection:**
   - Use `docker network ls` to list available networks.
   - Use `docker network inspect <network_name>` to inspect details of a specific network.

11. **Creating Networks:**
    - Use `docker network create` to create a custom network.

12. **Connecting Containers:**
    - Containers can be connected to a specific network using the `--network` option when running `docker run` or in Docker Compose.

13. **Removing Networks:**
    - Use `docker network rm <network_name>` to remove a network.

14. **Docker Compose Example:**
    ```yaml
    version: '3'
    services:
      web:
        image: nginx
        networks:
          - mynetwork

    networks:
      mynetwork:
    ```
    - In this example, the `web` service is connected to a custom network named `mynetwork`.

15. **Debugging and Troubleshooting:**
    - Use `docker logs <container_id>` to check container logs for network-related issues.
    - Inspect networks and containers using `docker network inspect` and `docker inspect` commands.

This overview covers the basics of Docker networking. As you gain more experience, you can explore advanced networking features and use cases based on your application requirements.

## Container DNS

When containers are connected to the same Docker network, Docker automatically provides a built-in DNS resolution mechanism that allows containers to discover and communicate with each other using their container names as hostnames. This feature simplifies the process of connecting and communicating between containers on the same network.

Here's a breakdown of how it works:

1. **Automatic DNS Resolution:**
   - Docker provides an internal DNS server for each user-defined bridge network.
   - This DNS server automatically maintains a record of container names and their corresponding IP addresses within the network.

2. **Container Names as Hostnames:**
   - Containers on the same network can use each other's container names as hostnames to communicate.
   - Docker resolves the container names to their respective IP addresses within the network using the internal DNS server.

3. **Example Usage:**
   - Suppose you have two containers, `web_server` and `database_server`, connected to the same network named `my_network`.
   - If `web_server` wants to communicate with `database_server`, it can use the hostname `database_server`.
   - Docker's internal DNS server resolves `database_server` to the actual IP address of the `database_server` container within the `my_network`.

4. **Docker Compose Example:**
  
   ```yaml
   version: '3'
   services:
     web_server:
       image: nginx
       networks:
         - my_network

     database_server:
       image: mysql
       networks:
         - my_network

   networks:
     my_network:
   ```
   - In this example, both `web_server` and `database_server` are connected to the `my_network` network. They can use each other's container names for communication.

5. **Benefits:**
   - Simplifies the configuration and communication between containers by using human-readable container names.
   - Allows for dynamic scaling and service discovery as new containers can be added to the network without manual configuration changes.

This automatic DNS resolution mechanism is a convenient feature in Docker networking, making it easier to build and manage multi-container applications where containers need to communicate with each other within the same network.

## The commands

Docker networking has its own docker network sub-command. The main commands include:
- `docker network ls`: Lists all networks on the local Docker host.
- `docker network create`: Creates new Docker networks. By default, it creates them with the nat driver on Windows and the bridge driver on Linux. You can specify the driver (type of network) with the -d flag. docker network create -d overlay overnet will create a new overlay network called overnet with the native Docker overlay driver.
- `docker network inspect`: Provides detailed configuration information about a Docker network. Same as docker inspect.
- `docker network prune`: Deletes all unused networks on a Docker host.
- `docker network rm`: Deletes specific networks on a Docker host.
