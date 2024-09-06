# Dockerfile Instructions and Docker Compose Specifications

## Dockerfile Instructions

| Instruction  | Description                                            |
|--------------|--------------------------------------------------------|
| `FROM`       | Defines base image                                     |
| `LABEL`     | Adds metadata to an Image                              |
| `RUN`        | Executes commands in a new layer and commits the results|
| `ADD/COPY`   | Adds files and folders into image ( The difference is that `COPY` takes the files and dump it whereas `ADD` uses a link/archive you provide to download from it and put it into the image)|
| `CMD`        | Runs binaries/commands on `docker run`                 |
| `ENTRYPOINT` | Allows to configure a container that will run as an executable (higher priority than `CMD`)|
| `VOLUME`     | Creates a mount point and marks it as holding externally mounted volumes|
| `EXPOSE`     | Container listens on the specified network ports at runtime|
| `ENV`        | Sets environment variables                             |
| `USER`       | Sets username (or UID)                                 |
| `WORKDIR`    | Sets the working directory                             |
| `ARG`        | Defines a variable that users can pass at build-time   |
| `ONBUILD`    | Adds to the image a *trigger* instruction to be executed at a later time|
| `HEALTHCHECK`| Check a container's health on startup                  |

## Compose Specifications

* **name**: Top-level property that defines the project name to be used 
if you don't set one explicitly

  - Example: `name: myapp`

* **services**: Defines computing resources within an application, backed 
by a set of containers

  - Example: 
    ```
    services:
        web:
        ...
        
        db:
        ...
    ```
* **build**: Tells Compose how to (re)build an application from source 
and lets you define the build process within a Compose file in a portable 
way

  Examples:

  - Using a subdirectory with a `Dockerfile` inside it:
    ```
    services:
      frontend:
        image: repository/webapp
        build: ./webapp
    ```

  - Using a **custom** `Dockerfile` located in a specific location:
    ```
    services:
      backend:
      image: repository/database
      build:
        context: path-to-dockerfile/
        dockerfile: backend.Dockerfile
    ```

  - From a GitHub repository:
    ```
    services:
      webapp:
        build: https://github.com/mycompany/example.git#branch_or_tag:subdirectory
    ```

* **image**: Specifies the image to start the container from. It must follow the
following image format:

    `[<registry>/][<project>/]<image>[:<tag>|@<digest>]`
  
  Examples:

  - `<image>:<tag>`:
    ```
    services:
      frontend:
        image: redis:5
    ```
  
  - `<image>@<digest>`:
    ```
    services:
      frontend:
        image: redis@sha256:0ed5d5928d4737458944eb604cc8509e245c3e19d02ad83935398bc4b991aac7
    ```

  - `<project>/<image>`:
    ```
    services:
      frontend:
        image: library/redis
    ```

* **container_name**: Specifies a custom container name: 
`container_name: my-web-container`

* **ports**: Defines the port mappings between the host machine and the 
containers. Syntax: `"[HOST:]CONTAINER[/PROTOCOL]"`

  Examples:
    ```
    ports:
      - "3000-3005"
      - "8000:8000"
      - "9090-9091:8080-8081"
      - "127.0.0.1:8001:8001"
      - "6060:6060/udp"
    ```

* **expose**: Defines the (incoming) port or a range of ports that Compose 
exposes from the container. Syntax: `<portnum>/[<proto>]` or 
`<startport-endport>/[<proto>]` for a port range

* **depends_on**: Controls the order of service startup and shutdown. It is 
useful if services are closely coupled, and the startup sequence impacts 
the application's functionality

  Example: Compose creates first `db` and `redis` before `web`:
  ```
    services:
      web:
        build: .
        depends_on:
          - db
          - redis
      redis:
        image: redis
      db:
        image: postgres
  ```

* **environment**: Defines environment variables set in the container

  Example:
  ```
  environment:
    - RACK_ENV=development
    - SHOW=true
    - USER_INPUT
  ```

* **restart**: Defines the policy that the platform applies on container 
termination:

  - `no`: Default. It does not restart the container under any 
  circumstances

  - `always`: The policy always restarts the container until its removal
  
  - `on-failure[:max-retries]`: The policy restarts the container if the 
  exit code indicates an error. Optionally, limit the number of restart 
  retries the Docker daemon attempts

  - `unless-stopped`: The policy restarts the container irrespective of 
  the exit code but stops restarting when the service is stopped or removed.

* **secrets**: grants access to sensitive data defined by the secrets 
top-level element on a per-service basis. Services can be granted access 
to multiple secrets.

  Example: Grant the frontend service access to the `server-certificate`
  secret set to the contents of the file `./server.cert`
    ```
    services:
      frontend:
        image: example/webapp
        secrets:
          - server-certificate
    secrets:
      server-certificate:
        file: ./server.cert
    ```

* **volumes**: Define mount host paths or named volumes that are 
accessible by service containers. Syntax: 
`VOLUME:CONTAINER_PATH:[ACCESS_MODE]`

  - `VOLUME`: Can be either a host path on the platform hosting 
  containers (bind mount) or a volume name
    
    * **volume** (managed by Docker): More suitable for preserving 
    data. 
    
      Example:
        ```
        services:
          web:
            build: .
            ports:
                - "8000:5000"
            volumes:
                - logvolume01:/var/log
            
        volumes:
          logvolume01: {}
        ```

    * **bind mount**: Used to inject data from host machine to the 
    container

      Example:
        ```
        volumes:
        - .:/code
        ```

  - `CONTAINER_PATH`: The path in the container where the volume 
  is mounted.

  - `ACCESS_MODE`: A comma-separated `,` list of options: 
  
    * `rw`: Default. Read and write access
    * `ro`: Read-only access

* **commands**: Overrides the default command declared by the 
container image, for example by Dockerfile's `CMD`.

  - Example: `command: bundle exec thin -p 3000`

* **entrypoint**: Declares the default entrypoint for the service 
container. This overrides the `ENTRYPOINT` instruction from the 
service's Dockerfile.

  - Example: `entrypoint: /code/entrypoint.sh`