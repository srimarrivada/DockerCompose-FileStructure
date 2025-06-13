# DOCKER COMPOSE FILE STRUCTURE
Docker Compose tool looks for the docker compose file named `compose.yaml` (preferred) or `compose.yml` in the current working directory. If it does not exist, it looks for at least the file named 
`docker-compose.yaml` or `docker-compose.yml` in the current working directory. This compose file contains the definition of all services that make up the application, along with their dependencies and 
configurations. It is important to understand YAML scripting language to write Docker compose file since indentations are important for YAML file.

The structure of the docker compose file includes elements such as:
* `version` – It defines the version of the Compose file format specifying which Docker Compose features and syntax to use. It is typically defined at the top of the file.
  This element has been deprecated in the latest version of Docker Compose.

* `name `– It defines the project name to be used. When this is not defined, Docker compose uses a default project name.

* `services` – This section lists out each containerized service required for the application. In Docker Compose, every component of the application operates as a separate service, with each service running
  a single container such as a database or web server, or cache.  
  Each service is defined as a separate block with a name and its configuration options such as:
  * `container_name`: It is used to specify a customer container name.
  * `image`: It defines the Docker image to use by the service and this image is the one that is either built locally or pulled from the Docker Hub or any other registry.
  * `build`: It defines to build the image locally from `Dockerfile` in the specified directory. This is helpful to add custom code in the application.
    It can have additional configuration options such as:
    * `context` which defines a path to a directory containing a Dockerfile
    * `dockerfile` which is used to set the name of Docker file used
    * `target` which is used to define the stage to build. 
    
    Refer to [Docker Compose Build Specification](https://docs.docker.com/reference/compose-file/build/) for complete details on build configuration.  
  * `environment`: It allows to pass configurations or sensitive information such as database credentials or API keys, to the service.
  * `env_file`: It is used to specify one or more files that contain environment variables to be passed to the containers. Environment variables declared in the `environment` section override these values.
  * `ports`: It defines to map a container’s internal ports to those on the host machine, enabling access to services from outside the container.
  * `expose`: It defines the port or range of ports that are exposed from the container.
  * `volumes`: It attaches persistent storage to a service so that data is accessible even when a container is restarted.
  * `working_dir`: It is used to set the working directory. It overrides `WORKDIR` instruction specified for the container image in `Dockerfile`.
  * `restart`: It is used to define how the service should behave in case of failures. It can values including:
    *  `no`
    *  `always`
    *  `on-failure`
    *  `unless-stopped`
  * `restart-policy`: It is used to specify the restart policy for individual containers. It overrides the global restart policy set in the `services` section. It can have additional options such as:
    * `condition`
    * `delay`
    * `max_attempts
  * `healthcheck`: It is used to define health check configurations for the service and determine if a service is healthy or not. It can have additional options such as:
    *  `test`
    *  `interval`
    *  `timeout`
    *  `retries`
    
    It overrides `HEALTHCHECK` instruction specified for the container image in `Dockerfile`.
  * `depends_on`: It is used to define dependencies between services by controlling the startup order of services. It ensures that one service starts only after its dependent services are up and running.
    It can have additional options such as:
    * `restart` which defines to restart this service after it updates the dependency service.
    * `condition` which defines to look for the condition of dependency service before starting this service and it can have values including:
       *  `service_started`
       *  `service_healthy`
       *  `service_completed_successfully`
    * `required` which is set to `true` by default to ensure the condition is satisfied and when it   to `false`, Docker compose only warns when the dependency service is not started or available.
  * `command`: It allows to specify a custom command or arguments when starting the container. It overrides the default `CMD` instruction declared for the container image in `Dockerfile`.
  * `entrypoint`: It declares the default entry point for the service container and overrides `ENTRYPOINT` instruction in the image `Dockerfile`.
  * `hostname`: It is used to set a custom host name to use for the service container.
  * `labels`: It is used to add metadata to services and containers.
  * `extends`: It allows to extend and override configurations by reusing common configurations across multiple services or environments.
  * `logging`: It allows to configure logging options for the service. It can have additional options such as:
    *  `driver`
    *  `path`
    *  `options`.
  * `user`: It is used to specify the user and group that a container should run as to control permissions and access within the container.
  * `deploy`: It defines to specify deployment options to manage containers across different environments. This is helpful to deploy containers with multiple replicas in a cluster platform such as
    Docker Swarm.  It can have additional configuration options such as:
    * `mode` which defines the replication model to run a service and it can have values including `global`, `replicated` (default), `replicated-job` and `global-job`
    * `replicas` which defines the number of containers that should be running when the service is `replicated`
    * `resources` which is used to configure physical resource constraints for container to run on platform
    * `restart_policy` which is used to configure how to restart containers when they exit. It can have addition options such as
      * `condition` which have values `none`(container is not automatically started), `on-failure` (container is restarted if it exits due to an error), `any` (default, container is restarted regardless always)
      * `delay` which defines the wait time between restart attemps		
      * `max_attempts` which defines maximum number of failed restarted attempts
      * `window` which defines the amount of time to wait after a restart to determine whether it was successful
    * `update_config` which is used to configure how the service should be updated. It can have additional options such as:
      * `parallelism` which defines the number of containers to update at a time
      * `order` which defines the order of operations during updates. It can have values such as `stop-first` (default where old task is stopped before starting new one), `start-first` (where new task is started first, and the running tasks briefly overlap)
      * `failure_action` which defines the action up on update failure. It can have values such as `continue`, `rollback`, or `pause` (default). 
      * `delay` which defines time to wait between group of containers update
    * `rollback_config` which is used to configure how the service should be rolled back when update fails.
      * `parallelism` which defines the number of containers to rollback at a time
      * `order` which defines the order of operations during rollbacks. It can have values such as `stop-first` (default where old task is stopped before starting new one), `start-first` (where new task is started first, and the running tasks briefly overlap)
      * `failure_action` which defines the action up on rollback failure. It can have values such as `continue` or `pause` (default). 
      * `delay` which defines time to wait between rollbacking a group of containers
       
    Refer to [Docker Compose Deploy Specification](https://docs.docker.com/reference/compose-file/deploy/) for complete details on deployment configuration.

* `networks` – This section allows to define custom networks that are attached to various services so that secure communication is established between containers over isolated networks. Services defined in
  Docker Compose by default uses an implicit `default` network to connect with each other and the `default` network can also be customized with this section. Each network is defined as a separate block with
  a name and configuration options such as:
  * `name`: It allows to set a custom name for the network.
  * `driver`:  It is used to define the network driver type, such as:
    *  `bridge` which is default for local networks
    *  `overlay` which is used for multi-host networks in Docker Swarm
  * `driver_opts`: It allows to define additional settings on the network driver.
  * `attachable`: It is used to enable the standalone containers to attach to this network along with the service containers if this is set to `true`. If a standalone container attaches to the network, it
    can communicate with services and other standalone containers that are also attached to the network.
  * `ipam`: IP address management (IPAM) configures the network-level IP settings including subnets and IP ranges to get more control over IP address space assigned to services. It uses additional options
    such as `driver`, `config`, `options`.
  * `external`: It allows to connect to an external network created outside of the Docker compose.
  * `internal`: It specifies to create an isolated network when this option is set to `true`. By default, Docker compose provides external connectivity to networks.
  * `labels`: It is used to add metadata to networks.

*	`volumes` - This section defines various volumes to persist data created by services and to share data between containers. This persisted data exists even if containers are stopped or removed. Each volume is
  defined as a separate block with a name and its configuration options such as below:
 	* `name`: It allows to set a custom name for the volume
    * `driver`: It indicates the volume driver to be used by the volume. The default driver is `local`
    * `driver_opts`: It allows to customize the volume driver with additional options such as:
      * `type` which has values `none`, `nfs`
      * `o`
      * `device`
    * `external`: It allows to use external volume that was created outside of Docker Compose.
    * `labels`: It is used to add metadata to volumes.
    
* `configs` - This section allows to define configurations for the services using external files. These configurations can include environment variables, file paths, and other settings. Each config is
  defined as a separate block with a name and its configuration options such as below:
  * `name`: It allows to set a custom name for the config object in the container engine.
  * `file`: It is used to create a config with the contents of file at the specified path.
  * `environment`: It allows to create config content with the environment variable value.
  * `content`: It is used to create the content with the in-lined value.
  * `external`: It allows to use external config that was created outside of Docker Compose.

* `secrets` – This section allows to manage the sensitive data provided to the services securely. Secrets are encrypted and stored on the Docker host, and they can be accessed by services within Docker Compose
  setup. Each secret is defined as a separate block with its configuration options such as below:
  * `file`: It is used to create a secret with the contents of file at the specified path.
  * `environment`: It allows to create secret with the environment variable value.
  * `external`: It allows to use external secret that was created outside of Docker Compose.
	
