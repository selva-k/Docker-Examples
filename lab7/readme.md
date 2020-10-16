## 7.0 Docker Monitoring
**Prometheus configuration**
Start by creating a directory called mkdir prometheus. Make sure to cd prometheus before you start creating the files


### 7.1 Prometheus configuration
Create a prometheus.yml file and populate it with this configuration:

```
scrape_configs:
- job_name: cadvisor
  scrape_interval: 5s
  static_configs:
  - targets:
    - cadvisor:8080
```

### 7.2 Docker Compose configuration
In the same folder where you created the prometheus.yml file, create a docker-compose.yml file and populate it with this Docker Compose configuration:

```
version: '3.2'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
    - 9090:9090
    command:
    - --config.file=/etc/prometheus/prometheus.yml
    volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    depends_on:
    - cadvisor
  cadvisor:
    image: gcr.io/google-containers/cadvisor:latest
    container_name: cadvisor
    ports:
    - 8080:8080
    volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
    depends_on:
    - redis
  redis:
    image: redis:latest
    container_name: redis
    ports:
    - 6379:6379
```

### 7.3 Run Services
This configuration instructs Docker Compose to run three services, each of which corresponds to a Docker container:

1) The prometheus service uses the local prometheus.yml configuration file (imported into the container by the volumes parameter).
2) The cadvisor service exposes port 8080 (the default port for cAdvisor metrics) and relies on a variety of local volumes (/, /var/run, etc.).
3) The redis service is a standard Redis server. cAdvisor will gather container metrics from this container automatically, i.e. without any further configuration.
To run the installation:
```
docker-compose up
```
If Docker Compose successfully starts up all three containers, you should see output like this:

```
prometheus  | level=info ts=2018-07-12T22:02:40.5195272Z caller=main.go:500 msg="Server is ready to receive web requests."
```

You can verify that all three containers are running using the ps command:

```
docker-compose ps
```


Your output will look something like this:
```
   Name                 Command               State           Ports
----------------------------------------------------------------------------
cadvisor     /usr/bin/cadvisor -logtostderr   Up      8080/tcp
prometheus   /bin/prometheus --config.f ...   Up      0.0.0.0:9090->9090/tcp
redis        docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp
```


### 7.4 Exploring the cAdvisor web UI

You can access the cAdvisor web UI at http://localhost:8080. You can explore stats and graphs for specific Docker containers in our installation at http://localhost:8080/docker/<container>. Metrics for the Redis container, for example, can be accessed at http://localhost:8080/docker/redis, Prometheus at http://localhost:8080/docker/prometheus, and so on.


### 7.5 Exploring metrics in the expression browser

cAdvisor's web UI is a useful interface for exploring the kinds of things that cAdvisor monitors, but it doesn't provide an interface for exploring container metrics. For that we'll need the Prometheus expression browser, which is available at http://localhost:9090/graph.
