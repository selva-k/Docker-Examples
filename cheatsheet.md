# Docker Command Cheat Sheet

## Basic Management Commands
- `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`: Run a new container
- `docker start CONTAINER`: Start a stopped container
- `docker stop CONTAINER`: Stop a running container
- `docker restart CONTAINER`: Restart a container
- `docker rm CONTAINER`: Remove a stopped container
- `docker rmi IMAGE`: Remove an image
- `docker ps`: List running containers
- `docker ps -a`: List all containers, including stopped ones
- `docker images`: List all images

## Container Operations
- `docker exec -it CONTAINER COMMAND`: Execute a command in a running container
- `docker logs CONTAINER`: Fetch the logs of a container
- `docker attach CONTAINER`: Attach local standard input, output, and error streams to a running container
- `docker cp SOURCE DEST`: Copy files/folders between a container and the local filesystem

## Image Operations
- `docker pull IMAGE`: Pull an image from a registry
- `docker push IMAGE`: Push an image to a registry
- `docker build -t TAG .`: Build an image from a Dockerfile in the current directory
- `docker history IMAGE`: Show the history of an image
- `docker save IMAGE > IMAGE.tar`: Save an image to a tar archive
- `docker load < IMAGE.tar`: Load an image from a tar archive

## Network Operations
- `docker network ls`: List networks
- `docker network create [OPTIONS] NETWORK`: Create a network
- `docker network rm NETWORK`: Remove a network
- `docker network connect NETWORK CONTAINER`: Connect a container to a network
- `docker network disconnect NETWORK CONTAINER`: Disconnect a container from a network

## Volume Operations
- `docker volume ls`: List volumes
- `docker volume create VOLUME`: Create a volume
- `docker volume rm VOLUME`: Remove a volume
- `docker volume inspect VOLUME`: Display detailed information on one or more volumes

## Docker Compose Commands
- `docker-compose up`: Create and start containers defined in `docker-compose.yml`
- `docker-compose down`: Stop and remove containers, networks, images, and volumes
- `docker-compose build`: Build or rebuild services
- `docker-compose logs`: View output from containers
- `docker-compose pause`: Pause running services
- `docker-compose unpause`: Unpause services
- `docker-compose exec SERVICE COMMAND`: Execute a command in a running service

## Docker System Commands
- `docker info`: Display system-wide information
- `docker version`: Show the Docker version information
- `docker system prune`: Remove all unused containers, networks, images (both dangling and unreferenced), and optionally, volumes

## Miscellaneous Commands
- `docker login [SERVER]`: Log in to a Docker registry server
- `docker logout [SERVER]`: Log out from a Docker registry server
- `docker search TERM`: Search the Docker Hub for images

**Note:** Replace `CONTAINER` with the container ID or name, `IMAGE` with the image name or ID, `NETWORK` with the network name or ID, `VOLUME` with the volume name, and `TAG` with the tag name for the image. Also, `[OPTIONS]` can be replaced with specific options for each command, and `[COMMAND] [ARG...]` with the command you want to execute inside the container.
