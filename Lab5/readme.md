## 5.0 Docker Compose
**The goal of this exercise is to create a Docker Compose with Python/Flask application with Nginx proxy and a Mongo database**

Project structure:
```
.
├── docker-compose.yaml
├── flask
│   ├── Dockerfile
│   ├── requirements.txt
│   └── server.py
└── nginx
    └── nginx.conf

```

### 5.1 Create a Python Flask app


Start by creating a directory called ```mkdir flask``` where we'll create the following files:

Make sure to ```cd flask``` before you start creating the files, because you don't want to start adding a whole bunch of other random files to your image.

### 5.2 server.py

Create the **server.py** with the following content:

```python
#!/usr/bin/env python
import os

from flask import Flask
from pymongo import MongoClient

app = Flask(__name__)

client = MongoClient("mongo:27017")

@app.route('/')
def todo():
    try:
        client.admin.command('ismaster')
    except:
        return "Server not available"
    return "Hello fom the MongoDB client!\n"


if __name__ == "__main__":
    app.run(host='0.0.0.0', port=os.environ.get("FLASK_SERVER_PORT", 9090), debug=True)

```

### 5.3 requirements.txt

In order to install the Python modules required for our app, we need to create a file called **requirements.txt** and add the following line to that file:

```
pymongo
flask
```


### 5.4 Write a Dockerfile

```
FROM python:3.7

WORKDIR /src
COPY . .
RUN pip install -r requirements.txt

CMD ["./server.py"]
```

### 5.5 nginx

Start by creating a directory called ```mkdir nginx``` at the root level  where we'll create the following file

### 5.6 nginx.config
```
server {
  listen 80;
  location / {
    proxy_pass http://$FLASK_SERVER_ADDR;
  }
}
```

At the root level create the following file

### 5.7 docker-compose.yaml
```
version: "3.7"
services:
  web:
    image: nginx
    volumes:
      - ./nginx/nginx.conf:/tmp/nginx.conf
    environment: 
      - FLASK_SERVER_ADDR=backend:9091  
    command: /bin/bash -c "envsubst < /tmp/nginx.conf > /etc/nginx/conf.d/default.conf && nginx -g 'daemon off;'" 
    ports:
      - 80:80
    depends_on:
      - backend
  backend:
    build: flask
    environment: 
      - FLASK_SERVER_PORT=9091
    volumes:
      - ./flask:/src
    depends_on:
      -  mongo  
  mongo:
    image: mongo
```

The compose file defines an application with three services `web`, `backend` and `db`.
When deploying the application, docker-compose maps port 80 of the web service container to port 80 of the host as specified in the file.
Make sure port 80 on the host is not being used by another container, otherwise the port should be changed.

### 5.8 Deploy with docker-compose

```
$ docker-compose up -d
Creating network "nginx-flask-mongo_default" with the default driver
Pulling mongo (mongo:)...
latest: Pulling from library/mongo
423ae2b273f4: Pull complete
...
...
Status: Downloaded newer image for nginx:latest
Creating nginx-flask-mongo_mongo_1 ... done
Creating nginx-flask-mongo_backend_1 ... done
Creating nginx-flask-mongo_web_1     ... done

```

### 5.9 Expected result

Listing containers must show three containers running and the port mapping as below:
```
$ docker ps
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                  NAMES
a0f4ebe686ff        nginx                       "/bin/bash -c 'envsu…"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp     nginx-flask-mongo_web_1
dba87a080821        nginx-flask-mongo_backend   "./server.py"            About a minute ago   Up About a minute                          nginx-flask-mongo_backend_1
d7eea5481c77        mongo                       "docker-entrypoint.s…"   About a minute ago   Up About a minute   27017/tcp              nginx-flask-mongo_mongo_1
```

After the application starts, navigate to `http://localhost:80` in your web browser or run:
```
$ curl localhost:80
Hello fom the MongoDB client!
```

Stop and remove the containers
```
$ docker-compose down
```


