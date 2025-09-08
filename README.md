# DOCKER COMPOSE FILE STRUCTURE
Docker Compose tool looks for the docker compose file named `compose.yaml` (preferred) or `compose.yml` in the current working directory. If it does not exist, it looks for at least the file named 
`docker-compose.yaml` or `docker-compose.yml` in the current working directory. This compose file contains the definition of all services that make up the application, along with their dependencies and 
configurations. It is imperative to understand [YAML's syntax](https://github.com/srimarrivada/YAML-Introduction) before creating a Docker Compose file, as proper indentation must be followed for defining the Compose's hierarchical structure.

The structure of the Docker Compose file includes elements such as:
* `version` – It specifies the version of the Docker Compose file format. It is typically defined at the top of the file.
  This element has been deprecated in the latest version of Docker Compose.

* `name `– It defines the project name to use for the entire application stack. When this is not set, Docker compose uses a default project name.

* `services` – This section defines each containerized service required for the application. In Docker Compose, every component of the application operates as a separate service, with each service running a single container.    
  Each service is defined as a separate block with a name and its configuration options such as:
  * `container_name`: It is used to specify a user-friendly name for the container on the host machine.
  * `image`: It specifies the Docker image to use for the container and this image is pulled from the Docker Hub or any other registry.
  * `build`: It defines to build a custom image locally from a `Dockerfile` in the specified directory, instead of pulling an image from the Docker Hub. This is helpful to add custom code in the application.  
    It can have additional configuration options such as:
    * `context` which defines the directory path containing a Dockerfile and all other files needed for the build.
    * `dockerfile` which is optional to specify an alternate name for `Dockerfile`, relative to the build context.
    * `target` which is used to define a named stage to build within a multi-stage `Dockerfile`. This allows to build a specific version of image, such as a "development" version with debugging tools or a lean "production" version, from a single `Dockerfile`.
    * `args` which defines build-time variables that can be used by the `ARG` instruction inside the `Dockerfile`. These variables are only available during the build and do not persist in the final container.
	* `no_cache` which forces a rebuild of the image without using any cached layers from previous builds. This is useful for debugging, ensuring clean builds, or pulling the latest dependencies that may have been updated since the last build.
	* `platforms` which specifes the target operating system and architecture to build the image for, which is useful for multi-platform environments.  
    Refer to [Docker Compose Build Specification](https://docs.docker.com/reference/compose-file/build/) for complete details on build configuration.  
  * `environment`: It allows to pass environment variables directly to the service container at runtime. This is useful to pass configurations or sensitive information such as database credentials or API keys, to the service. It can be defined in two ways either as a simple list of KEY=VALUE pairs or as a key-value mapping. It is useful for small configurations.
  * `env_file`: It is used to specify one or more files (typically named as `.env`) that contain environment variables to be passed to the containers. Environment variables declared in the `environment` section override these values. It is helpful to keep sensitive data and large sets of variables separate from the Docker Compose file.
  * `ports`: It maps or publishes the ports from the host machine to the container’s internal ports, making the service accessible from outside the container. By default, containers are isolated from the host's network, so ports acts as a firewall rule to forward network traffic to the correct service. This is specified as list of strings in either HOST_PORT:CONTAINER_PORT format or CONTAINER_PORT format. When only container port is specified, it pubishes to a random available port on the host, avoiding conflicts.
  * `expose`: It defines the port or range of ports that are exposed from the container for internal communication between services running on the same Docker network without actually publishing or mapping the port to the host.
  * `volumes`: It attaches persistent storage to a service so that data is accessible even when a container is stopped or removed. It can be defined in three ways:
     * using NAMED_VOLUME:CONTAINER_PATH format along with additional access mode (`rw`, `ro`, `z`, `Z`) such as NAMED_VOLUME:CONTAINER_PATH:ACCESS_MODE format or
     * using bind mounts by mapping file paths from the host machine to the container in HOST_PATH:CONTAINER_PATH format or
     * using a more verbose long syntax for finer control by specifying addition configuration options such as `type` _(which defines volume type such as `volume`, `bind`, `tmpfs`, or `npipe`)_, `source` _(which defines host path)_, `target` _(which defines container path)_, `read_only` _(which sets the volume as read-only)_, etc. 
  * `working_dir`: It is used to set the working directory inside the container for the service. It is equivalent to the `--workdir` flag in a `docker run` command and overrides `WORKDIR` instruction specified for the container image in `Dockerfile`.
  * `restart`: It defines the restart policy, indicating how the service container should behave when it stops or fails. It can have values such as `no`, `always`, `on-failure` or `unless-stopped`.
  * `healthcheck`: It defines a command that Docker periodically runs inside a container to determine if the service is functioning correctly or not.  It works in the same way, and has the same default values, as the `HEALTHCHECK` `Dockerfile` instruction.  
    It can have additional configuration options such as:
    * `test` which specifies a command that Docker Compose runs insider the container to check its health.
    * `interval` which defines the time between health checks throughout the container lifecycle (default is `30` seconds).
    * `timeout` which defines the maximum time that Docker will wait for the `test `command to complete before it is considered a failure (default is `30` seconds).
    * `retries` which defines the number of consecutive failures needed to mark the container as `unhealthy` (default is `3`).
    * `start_period` which provides a grace period after the container starts during which health check failures are ignored. This is useful for services that need extra time to initialize without being marked as unhealthy prematurely. 
    * `start_interval` which defines the time between health checks specifically during the `start_period` to accelerate startup readiness checks.
    * `disable` which defines if health check to be disabled. It accepts `true` or `false` values.
    
	This configuration overrides the `HEALTHCHECK` instruction specified for the container image in `Dockerfile`.
  * `depends_on`: It is used to define dependencies between services to control the startup order of services. It ensures that one service starts only after its dependent services are up and running.  
    It can have additional options such as:
    * `restart` which defines to restart this service after it updates the dependency service.
    * `condition` which defines to look for the condition of dependency service before starting this service and it can have values such as `service_started`, `service_healthy` or `service_completed_successfully`.
    * `required` which is set to `true` by default to ensure the condition is satisfied and when it is set to `false`, Docker compose only warns when the dependency service is not started or available.
  * `command`: It allows to specify a custom command or arguments to run when the service's container starts. It overrides the default `CMD` instruction declared for the container image in `Dockerfile`. It can be easily overridden by passing arguments to the `docker run` command.
  * `entrypoint`: It defines the main executable that always runs when the container starts. This overrides the `ENTRYPOINT` instruction in the image `Dockerfile` and cannot be changed at runtime with arguments passed to the `docker run` command.
  * `hostname`: It is used to set a custom host name to use for the service container, which is primarily used for the container to identify itself internally on the network. By default, the container's hostname is its randomly generated ID. It must be unique within the same network.
  * `labels`: It is used to add metadata as key-value pairs or as a list to the service. It does not affect the runtime behavior of Docker, but they are widely used for filtering, organization, and automated workflows.
  * `extends`: It allows to extend and override configurations by reusing common configurations between different Compose files or projects, thereby reducing duplication.  
    It can have additional options such as:
    * `service` which specifies the name of the service to extend from the base file.
    * `file` which specifies the location of the base Compose file. If not specified, Compose looks for the service in the current file. 
  * `logging`: It allows to configure logging options for the service.  
    It can have additional options such as:
    * `driver` which specifies the logging driver that Docker should use for the container. Some common logging drivers include `json-file` (Docker's default driver that stores logs locally on the host machine in JSON format), `local` (offers automatic log rotation by default and uses an optimized format to save disk space), `syslog` (forwards container logs to the host's syslog server),`fluentd` (commonly used for centralized logging as it streams logs to a Fluentd daemon that can be routed to various destinations like Elasticsearch or AWS S3.),`awslogs` (for applications running on Amazon Web Services),`none` (to completely disable the logging container), etc.
    * `options` which is used to set the configuration settings specific to the selected driver. Options often include log rotation limits, network addresses for centralized logging, and other driver-specific parameters. 
  * `user`: It is used to specify the user (UID) and group (GID) that a container should run as to control permissions and access within the container. This helps to prevent containers from running with root privileges. It can be set using user by name or by its numeric UID, often combined with a group ID (GID) in UID:GID format.
  * `deploy`: It specifies configurations specific to deployment for managining containers across different environments. This is helpful to deploy services as a stack in a cluster platform such as Docker Swarm.  
     It can have additional configuration options such as:
    * `mode` which defines the replication model to run a service and it can have values including `global`, `replicated` (default), `replicated-job` and `global-job`.
    * `replicas` which defines the number of identical containers to run when the service is `replicated`.
    * `resources` which is used to configure physical resource constraints (`limits` and `reservations` for CPU and memory) for container to run on platform.
	* `placement` which specifies nodes or hosts where the service can be deployed by defining rules based on node labels or roles. It can have additional options such as `constraints` (hard requirements that a node must meet for a container to be scheduled on it) and `preferences` (strategies to spread tasks evenly).
    * `restart_policy` which is used to control how a service's containers are restarted after exiting.
      It can have addition options such as
      * `condition` which have values `none`(container is not automatically started), `on-failure` (container is restarted if it exits due to an error), `any` (default, container is restarted regardless always)
      * `delay` which defines the wait time between restart attemps		
      * `max_attempts` which defines maximum number of failed restarted attempts
      * `window` which defines the amount of time to wait after a restart to determine whether it was successful
    * `update_config` which is used to configure how the service should be updated based on rolling update strategy.
      It can have additional options such as:
      * `parallelism` which defines the number of containers to update at a time
      * `order` which defines the order of operations during updates. It can have values such as `stop-first` (default where old task is stopped before starting new one), `start-first` (where new task is started first, and the running tasks briefly overlap)
      * `failure_action` which defines the action up on update failure. It can have values such as `continue`, `rollback`, or `pause` (default). 
      * `delay` which defines time to wait between group of containers update
    * `rollback_config` which is used to configure how the service should be rolled back when update fails.
      * `parallelism` which defines the number of containers to rollback at a time
      * `order` which defines the order of operations during rollbacks. It can have values such as `stop-first` (default where old task is stopped before starting new one), `start-first` (where new task is started first, and the running tasks briefly overlap)
      * `failure_action` which defines the action up on rollback failure. It can have values such as `continue` or `pause` (default). 
      * `delay` which defines time to wait between rollbacking a group of containers
	* `labels` which adds metadata to the service stack itself. This is separate from the container-level labels and can be used for things like automatic routing.

  Refer to [Docker Compose Deploy Specification](https://docs.docker.com/reference/compose-file/deploy/) for complete details on deployment configuration.

* `networks` – This section allows to define custom networks that control how services can communicate with each other. These networs are attached to various services so that secure communication is established between containers over isolated networks. Services defined in Docker Compose by default uses an implicit `default` network to connect with each other and the `default` network can also be customized with this section.  
Each network is defined as a separate block with a name and configuration options such as:
  * `name`: It allows to set a custom name for the network.
  * `driver`:  It is used to specify the network driver to use such as `bridge` (default for local networks) or `overlay` (for multi-host communication in Docker Swarm).
  * `driver_opts`: It allows to define driver-specific options when creating a network. These options are passed directly to the chosen network driver, such as `bridge` or `overlay`, allowing for granular control over network behavior.
  * `attachable`: It is used to enable the standalone containers to attach to an `overlay` network along with the service containers if this is set to `true`. If a standalone container attaches to the network, it can communicate with services and other standalone containers that are also attached to the network.
  * `ipam`: It is used for custom IP Address Management (IPAM) configurations, such as the network-level IP settings including subnets and IP ranges to get more control over IP address space assigned to services. It uses additional options such as `driver`, `config`, `options`.
    It can have additional options such as
    * `driver` which specifies a custom IPAM driver
    * `config` which specifies a list that defines one or more IP address pools for the network. Each item in the list can specify detailed configurations such as `subnet` _(specifies the network segment in CIDR format (e.g., 172.20.0.0/16) from which IP addresses will be allocated)_, `ip_range` _(defines a sub-range within the specified subnet from which container IPs should be allocated)_, `gateway` _(sets the IPv4 or IPv6 gateway for the network)_ and `aux_addresses` _(specifies auxiliary IPv4 or IPv6 addresses for specific hosts on the network, given as a key-value mapping)_
    * `options` which defines a key-value map for passing driver-specific options to the IPAM driver
  * `external`: It allows to connect to an external network created outside of the Docker compose.
  * `internal`: It creates an isolated network from external networks when this option is set to `true`, making it a "back-end" network for services like databases that should not be publicly exposed. By default, Docker compose provides external connectivity to networks.
  * `labels`: It is used to add metadata to custom networks. This does not affect the network's runtime behavior directly, but useful for organization, filtering, and enabling automation with third-party tools. 
  
* `volumes` - This section defines and manages named volumes to persist data created by services and to share data between containers. This persisted data exists even if containers are stopped or removed.  
  Each volume is defined as a separate block with a name and its configuration options such as below:
 	* `name`: It allows to set a custom name for the volume.
    * `driver`: It specifies a storage driver to be used by the volume. The default driver is `local`
    * `driver_opts`: It allows to customize the volume driver with additional options such as:
      * `type` which specifies mount type such as `none` or `nfs`.
      * `o` which is used to pass mount options like address, read/write, and lock settings.
      * `device` which is used to specify the path on the NFS server.
    * `external`: It allows to use external volume that was created outside of Docker Compose when this is set to `true`.
    * `labels`: It is used to add metadata to volumes for filtering and organization, which is useful for third-party tools. 
    
* `configs` - This section allows to define configurations for the services using external files. These configurations can include environment variables, file paths, and other settings for storing non-sensitive configuration data outside a service's image or container.  
  Each config is defined as a separate block with a name and its configuration options such as below:
  * `name`: It allows to set a custom name for the config object in the container engine.
  * `file`: It is used to create a config with the contents of the given file at the specified path.
  * `environment`: It allows to create config content with the environment variable value.
  * `content`: It is used to create the content with the in-lined value.
  * `external`: It allows to use external config that was created outside of Docker Compose.

* `secrets` – This section allows to manage the sensitive data like passwords, API keys, or certificates provided to the services in a secure way without storing them in plain text in the configuration file or Docker image. Secrets are encrypted and stored on the Docker host, and they can be accessed by services within Docker Compose setup.  
  Each secret is defined as a separate block with its configuration options such as below:
  * `file`: It is used to create a secret with the contents of file at the specified path.
  * `environment`: It allows to create secret with the environment variable value.
  * `external`: It allows to use external secret that was created outside of Docker Compose.
<br/>

For a complete list of configuration settings in a Docker Compose, please visit the official [Docker Compose File Reference](https://docs.docker.com/reference/compose-file/).
<br/>
<br/>

Below is an example `docker-compose.yml` file that uses the most configuration settings to define a multi-container application with a web server, a backend service, a database, and configurations for a production-like environment.
```
# Specify the Compose file format version (deprecated in latest Compose version)
# version: "3.8"

# ---------------------------------------------------------------------------------------
# Top-level sections: Define reusable resources for the entire application stack, 
# such as persistent `volumes`, isolated `networks`, `configs` for configuration files, 
# and `secrets` for sensitive data.
# ---------------------------------------------------------------------------------------

# Define named volumes for persistent storage
volumes:
  # Volume for the database data. Docker manages this storage
  db-data:
    # Use the 'local' driver with bind mount options for development
    driver: local
    # Optionally specify driver options for advanced volume configuration
    driver_opts:
      type: "none"
      o: "bind"
      device: "${PWD}/db-data"
  # Volume for the web app's static files
  web-static-data:

# Define custom networks for service isolation
networks:
  # Public network for external access and communication
  frontend:
    driver: bridge
    labels:
      com.example.network.purpose: "Public-facing network"
  # Private, internal network for backend services only
  backend:
    driver: overlay
    attachable: true # Allows manual container attachment for debugging
    internal: true
    # Network with custom IP Address Management (IPAM)
    ipam:
      driver: default
      config:
        - subnet: "172.20.0.0/24"
          gateway: "172.20.0.1"
# Define configuration files that are mounted into containers
configs:
  nginx_config:
    # Create a config from a local file
    file: ./config/nginx.conf
  license_file:
    # Create a config from the content of an environment variable
    environment: LICENSE_CONTENT
  custom_message:
    # Create a config with inlined, plain text content
    content: "Welcome to our application!"

# Define secrets for sensitive information (password, API key, etc.)
secrets:
  # A secret that reads from the local db_password.txt file
  db_password:
    # Use an external file for the secret's content
    file: ./secrets/db_password.txt
  # A secret for an API key that reads from api_key.txt
  app_api_key:
    # Use an external secret managed by the Docker CLI
    external: true
    name: external_api_key_secret

# ---------------------------------------------------------------------------------------
# Service definitions: Configure individual service (containerized component) behavior 
# such as  `build` (for custom images), `ports` (for external access), `environment` (for variables), 
# `depends_on` (for service order), `healthcheck` (to verify readiness), and `logging` (for log management).
# ---------------------------------------------------------------------------------------

services:
  # Service for the web application (e.g., Nginx serving static files)
  web:
    image: nginx:1.21
    # Override the container name for easy identification
    container_name: web-server-1
    # Expose port 80 to the host, binding it to 8080
    ports:
      - "8080:80"
    # Connect to the public network
    networks:
      - frontend
    # Mount the custom Nginx config and static files
    volumes:
      - type: config
        source: nginx_config
        target: /etc/nginx/nginx.conf
        read_only: true
      - web-static-data:/usr/share/nginx/html
    # Specify the command to run on startup
    command: ["nginx", "-g", "daemon off;"]
    # Add labels for filtering and documentation
    labels:
      - "com.example.service.type=web"
      - "com.example.service.maintainer=Dev Team"
    # Load environment variables from an external file
    env_file:
      - ./web/.env
    # Define deployment configurations exclusively for the Docker Swarm cluster, managing aspects like 
    #`replicas`, `update_config` for rolling updates, `placement` constraints, and resource limits.
    deploy:
      mode: replicated
      replicas: 3
      resources: # Define resource limits for the container
        limits:
          cpus: "0.50"
          memory: 512M
      update_config:
        parallelism: 1
        delay: 10s
      placement:
        constraints:
          - node.role == worker
  # Service for the backend API (e.g., a Python application)
  api:
    # Build a custom image from a Dockerfile in the current directory
    build:
      context: ./api
      dockerfile: Dockerfile.api
      args:
        # Pass a build-time variable
        API_VERSION: "1.0"
        NODE_VERSION: 18-alpine
    # Set the working directory inside the container
    working_dir: /app
    # Set the user to run the application as
    user: "1001:1001"
    # Define an entrypoint script that runs before the main command
    entrypoint: ./docker-entrypoint.sh
    # Define the command and its arguments
    command: ["gunicorn", "app:app", "--bind=0.0.0.0:5000"]
    # Pass environment variables and secrets
    environment:
      - APP_ENV=production
      - API_URL=http://api:5000
      - API_KEY_FILE=/run/secrets/app_api_key
    secrets:
      - app_api_key
    # Mount the local source code for development (bind mount)
    volumes:
      - type: bind
        source: ./src
        target: /app/src
    # Use restart policies for service resilience
    restart: unless-stopped
    # Define health checks to determine readiness
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:5000/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 20s
    # Configure logging to send logs to a central server
    logging:
      driver: syslog
      options:
        syslog-address: "udp://127.0.0.1:514"
    # Specify dependencies and conditions to control startup order
    depends_on:
      db:
        condition: service_healthy
    # Connect this service to the private backend network
    networks:
      - backend
    # Expose a port without mapping it to the host
    expose:
      - "5000"

  # Service for the database (e.g., PostgreSQL)
  db:
    image: postgres:13
    hostname: db-primary
    # Set environment variables for database credentials from a secret file
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_USER: myuser
      POSTGRES_DB: mydatabase
    # Grant access to Docker secrets
    secrets:
      - db_password
    # Mount the named volume for data persistence
    volumes:
      - db-data:/var/lib/postgresql/data
    # Use the private backend network for communication
    networks:
      - backend
    # Define a restart policy for automatic recovery
    restart: on-failure
    # Use the healthcheck from the postgres image to verify readiness
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 15s
    # Define deployment options for a Swarm cluster
    deploy:
      mode: global
      placement:
        constraints:
          - node.labels.role == database-node
      resources:
        limits:
          cpus: "0.50"
          memory: 1G
        reservations:
          cpus: "0.25"
          memory: 512M

```
