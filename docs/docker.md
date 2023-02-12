# Docker CLI

#### List all docker containers, running and stopped:

```sh
docker ps --all
```

#### Start a container from an image, with a custom name:

```sh
docker run --name ubuntu ubuntu:latest
```

#### Start an existing container:

```sh
docker start ubuntu
```

#### Stop an existing container:

```sh
docker stop ubuntu
```

#### Remove a stopped container:

```sh
docker rm ubuntu
```

#### Pull an image from dockerhub:

```sh
docker pull ubuntu
```

#### Display the list of already downloaded images:

```sh
docker images
```

#### Fetch and follow the logs of a container:

```sh
docker logs -f ubuntu
```

#### Cleanup dangling images, containers, volumes, and networks

```sh
docker system prune
```

#### Open a shell inside a running container:

```sh
docker exec -it ubuntu /bin/bash
docker exec -it ubuntu uname -a
docker exec -it ubuntu pwd
```

#### Stop and Remove ALL containers

```sh
docker stop $(docker ps -aq); docker rm $(docker ps -aq)
```

#### Stop ALL containers

```sh
docker stop $(docker ps -a -q)
```

#### Remove ALL containers

```sh
docker rm -f $(docker ps -a -q)
```


