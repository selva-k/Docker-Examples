## 5.0 Docker Compose
**The goal of this exercise is to create a Docker Compose with Python/Flask application with Redis database**


### 5.1 Create a Python Flask app

Start by creating a directory called ```mkdir composetest``` where we'll create the following files:
Project structure:

```
├── app.py
├── requirements.txt
├── Dockerfile
├── docker-compose.yaml
```

Make sure to ```cd composetest``` before you start creating the files.

### 5.2 app.py

Create the **app.py** with the following content:

```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

#### 5.3 requirements.txt

In order to install the Python modules required for our app, we need to create a file called **requirements.txt** and add the following line to that file:

```
flask
redis
```


### 5.4 Write a Dockerfile

```
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

### 5.7 docker-compose.yaml
```
version: "3.8"
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

The compose file defines an application with three services `web` and `redis` backend.

## 5.8 Deploy with docker-compose

```
$ docker-compose up -d
Creating network "composetest_default" with the default driver
Creating composetest_web_1 ...
Creating composetest_redis_1 ...
Creating composetest_web_1
Creating composetest_redis_1 ... done
Attaching to composetest_web_1, composetest_redis_1
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
redis_1  | 1:C 17 Aug 22:11:10.480 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis_1  | 1:C 17 Aug 22:11:10.480 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=1, just started
redis_1  | 1:C 17 Aug 22:11:10.480 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
web_1    |  * Restarting with stat
redis_1  | 1:M 17 Aug 22:11:10.483 * Running mode=standalone, port=6379.
redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
web_1    |  * Debugger is active!
redis_1  | 1:M 17 Aug 22:11:10.483 # Server initialized
redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
web_1    |  * Debugger PIN: 330-787-903
redis_1  | 1:M 17 Aug 22:11:10.483 * Ready to accept connections

```

## 5.9 Expected result

Listing containers must show two containers running and the port mapping

After the application starts, navigate to `http://localhost:5000` in your web browser or run:
```
$ curl http://localhost:5000
Hello World! I have been seen 1 times.
```

Refresh the page. The number should increment.

## 5.10 Edit the Compose file

Stop the application, either by running 

```
$ docker-compose down
```
from within your project directory in the second terminal or by hitting CTRL+C in the original terminal where you started the app.
Edit docker-compose.yml in your project directory to add a bind mount for the web service:

```
version: "3.8"
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_ENV: development
  redis:
    image: "redis:alpine"
```

The new volumes key mounts the project directory (current directory) on the host to /code inside the container, allowing you to modify the code on the fly, without having to rebuild the image. The environment key sets the FLASK_ENV environment variable, which tells flask run to run in development mode and reload the code on change. This mode should only be used in development.

## 5.11 Re-build and run the app with Compose

```
$ docker-compose up
Creating network "composetest_default" with the default driver
Creating composetest_web_1 ...
Creating composetest_redis_1 ...
Creating composetest_web_1
Creating composetest_redis_1 ... done
Attaching to composetest_web_1, composetest_redis_1
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
...
```
Check the Hello World message in a web browser again, and refresh to see the count increment.

## 5.12 Update the application

Because the application code is now mounted into the container using a volume, you can make changes to its code and see the changes instantly, without having to rebuild the image.

Change the greeting in app.py and save it. For example, change the Hello World! message to Hello from Docker!:

```
return 'Hello from Docker! I have been seen {} times.\n'.format(count)
```

Refresh the app in your browser. The greeting should be updated, and the counter should still be incrementing.