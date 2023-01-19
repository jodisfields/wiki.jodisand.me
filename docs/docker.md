# Docker Cheatsheet

### List all docker containers (running and stopped):

```sh
docker ps --all
```

### Start a container from an image, with a custom name:

```sh
docker run --name container_name image
```

### Start or stop an existing container:

```sh
docker start|stop container_name
```

### Pull an image from a docker registry:

```sh
docker pull image
```

### Display the list of already downloaded images:

```sh
docker images
```

### Open a shell inside a running container:

```sh
docker exec -it container_name sh
```

### Remove a stopped container:

```sh
docker rm container_name
```

### Fetch and follow the logs of a container:

```sh
docker logs -f container_name
```

### Stop and Remove ALL containers

```sh
docker stop $(docker ps -aq); docker rm $(docker ps -aq)
```

### Stop ALL containers

```sh
docker stop $(docker ps -a -q)
```

### Remove ALL containers

```sh
docker rm -f $(docker ps -a -q)
```

### Cleanup dangling images, containers, volumes, and networks

```sh
docker system prune
```

---

## Docker-Compose

### Elasticsearch v7.17.4 (Arkime)

```yml
version: '3.3'
services:
    elasticsearch:
        container_name: elasticsearch-v7.17.4
        environment:
            - discovery.type=single-node
        ports:
            - '9200:9200'
            - '9300:9300'
        image: 'docker.elastic.co/elasticsearch/elasticsearch:7.17.4'
        networks:
            - elastic
networks:
    elastic:
        driver: bridge
```
