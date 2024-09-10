# Docker Containerisation

**Contents**:

- [Docker Containerisation](#docker-containerisation)
  - [Project Steps](#project-steps)
  - [Project Requirements](#project-requirements)
    - [For Java Web Applications](#for-java-web-applications)
    - [For Microservice Applications](#for-microservice-applications)
  - [Docker CLI Commands](#docker-cli-commands)
  - [Docker and Docker Compose Instructions](#docker-and-docker-compose-instructions)


## Project Steps

1. Set up the environment: Local, AWS EC2 instance, GCP's
Compute Engine instance, etc.

2. Install **Docker Engine**:
  
    https://docs.docker.com/engine/install/

3. Find the Docker base images for your application 
requirements

  * Browse for the service images in Docker Hub:
  https://hub.docker.com/

  * Check which services require a customised image

4. Fetch the source code (from GitHub):
  
  * Add SSH Keys to your GitHub account
  
  * Clone repo: `git clone`

5. Write custom `Dockerfile` if required

6. Write a `docker-compose.yml` file

7. Run the containerised environment: 
   
   * `docker compose build`
   
   * `docker compose up`

8. Perform final tests  

## Project Requirements

* Docker base images for services from **Docker Hub**
to add in the docker compose manifest file

* Docker custom images for services (if required),
separated in different directories

* The `docker-compose.yml` file

* The application source code

### For Java Web Applications

* `java-webapp/`:

  - `src/`: Java source code
  
  - `pom.xml`: Maven configuration file
  
  - `docker-compose.yml`
  
  - `app/`: 
  
    * `tomcat.Dockerfile` 
  
  - `db/`: 
    
    * `mysql.Dockerfile`
    
    * `init_db.sql`
  
  - `web/`:

    * `nginx.Dockerfile`
    
    * `nginx.conf`
  
  - `README.md`

* Project example: 
  
  [Containerising a Java Application using Docker Compose in AWS](https://github.com/scaceresg/devops-vprofile-deployments/tree/17e606f50d6ec0da6a119215989b59848ba9c8fd/docker-aws-ec2)

### For Microservice Applications

* `app/`:
  
  - `service-1/`:
  
    * `service-1.Dockerfile`
  
  - `service-2/`:
  
    * `service-2.Dockerfile`
  
  - `service-3/`:
  
    * `service-3.Dockerfile`
  
  - `api-gateway/`:
  
    * `gateway.conf`
  
  - `docker-compose.yml`
  
  - `README.md`

* Project example:
  
  [Deploying an E-Commerce Microservice App using Docker in AWS](https://github.com/scaceresg/devops-ecommerce-docker-aws.git)

## Docker CLI Commands

Check [**docker-cli.md**](https://github.com/scaceresg/scaffold-devops/blob/43ca63b0abc437340533949a4797e65a4827f533/docker/docker-cli.md)

## Docker and Docker Compose Instructions

Check [**dockefile-compose.md**](https://github.com/scaceresg/scaffold-devops/blob/43ca63b0abc437340533949a4797e65a4827f533/docker/dockerfile-compose.md)