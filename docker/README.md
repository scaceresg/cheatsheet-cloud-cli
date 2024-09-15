# Docker Containerisation

**Contents**:

- [Docker Containerisation](#docker-containerisation)
  - [Project Steps](#project-steps)
  - [Project Requirements](#project-requirements)
    - [For Java Web Applications](#for-java-web-applications)
    - [For Microservice Applications](#for-microservice-applications)
  - [Docker CLI Commands](#docker-cli-commands)
  - [Docker and Docker Compose Instructions](#docker-and-docker-compose-instructions)
  - [Basic Templates](#basic-templates)
  - [Sample `Dockerfile` \& `compose.yml` Files for Applications](#sample-dockerfile--composeyml-files-for-applications)


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

* **Docker base images**:
  
  - Base images for services from **Docker Hub**
  
  - Prepare to add them to the docker compose manifest 
  file

* **Custom Dockerfiles**:
  
  - Custom images for services (if required)
  
  - Should be separated in different directories

* **Docker Compose file**: `docker-compose.yml`

* **Application source code**

### For Java Web Applications

```
java-webapp
├─ README.md
├─ docker-compose.yaml
├─ src
│  └─ source code ...
├─ pom.xml
├─ app
│  └─ tomcat.Dockerfile
├─ web
|  ├─ nginx.conf
│  └─ nginx.Dockerfile
└─ db
   ├─ init_db.sql
   └─ mysql.Dockerfile
```

* Project example: 
  
  [**Containerising a Java Application using Docker Compose in AWS**](https://github.com/scaceresg/devops-vprofile-deployments/tree/17e606f50d6ec0da6a119215989b59848ba9c8fd/docker-aws-ec2)

### For Microservice Applications

```
app
├─ README.md
├─ docker-compose.yaml
├─ service-1
│  ├─ service1.Dockerfile
│  └─ source code ...
├─ service-2
│  ├─ service2.Dockerfile
│  └─ source code ...
├─ apigateway
│  ├─ apigateway
│  │  └─ default.conf
│  └─ default.conf
└─ service-3
   ├─ service3.Dockerfile
   └─ source code ...
```

* Project example:
  
  [**Deploying an E-Commerce Microservice App using Docker in AWS**](https://github.com/scaceresg/devops-ecommerce-docker-aws.git)

## Docker CLI Commands

Check [**docker-cli.md**](docker-cli.md)

## Docker and Docker Compose Instructions

Check [**dockefile-compose.md**](dockerfile-compose.md)

## Basic Templates

Check [**Basic Templates**](./basic-templates/)

## Sample `Dockerfile` & `compose.yml` Files for Applications

Check 
[**Awesome Compose**](https://github.com/docker/awesome-compose/tree/master) 
repository.