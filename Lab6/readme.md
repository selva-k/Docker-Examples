## 6.0 Docker Volume
**The goal of this exercise is to create a Docker Volumes in the Host file system**


### 6.1 Create a nginx app

```
// pull the nginx image
docker pull nginx
// run the container
docker run -it --name=webApp -d -p 80:80 nginx
```

After the application starts, navigate to `http://localhost:80` in your web browser or run:
```
$ curl http://localhost:80
"Welcome to nginx!" default home page should show up 
```

### 6.2 Edit welcome page

Let’s use the docker exec command to edit the welcome page and load it

```
// list the running containers
docker ps
// exec command
docker exec -it webApp bash
// cd to welcome page and edit it
cd /usr/share/nginx/html
echo "I changed this file while running the conatiner" > index.html```
exit
```
Navigate to `http://localhost:80` in your web browser. "I changed this file while running the conatiner" should show up now.

Let’s stop the container and start it again. 
```
$ docker stop webApp
$ docker start webApp
```

We can still see the changes that we made. 

### 6.3 Create/Inspect Volume

```
// create a volume
docker volume create new_vol
// list volumes
docker volume ls
// inspect volumes
docker volume inspect new_vol
```


### 6.4 nginx container with volume mount
Let us start a nginx container with the welcome page mounted to volume new_vol that we created above and exposing the port 80.

```
$ docker run -d  --name devtest -v myvol2:/app  nginx:latest
```

Use docker inspect devtest to verify that the volume was created and mounted correctly. Look for the Mounts section

```
"Mounts": [
    {
        "Type": "volume",
        "Name": "myvol2",
        "Source": "/var/lib/docker/volumes/myvol2/_data",
        "Destination": "/app",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```

Stop the container and remove the volume. Note volume removal is a separate step

```
$ docker container stop devtest

$ docker container rm devtest

$ docker volume rm myvol2

```
