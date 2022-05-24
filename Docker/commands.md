# Commands

## Build
```
docker build -t <name of the registrey>/<name of the image>:<version> <path/of/the/Dockerfile>
```

## Run
```
docker run -it <name of the registrey>/<name of the image>:<version>
```

## Check the running containers
```
docker ps
```
### to see all the containers
```
docker ps -a
```

## Check the images in the local
```
docker images
```

## Pulling the image
```
docker pull <name of the registrey>/<name of the image>:<version>
```

## Deleting Containers
```
docker rm <container id>
```

## Deleting images
```
docker rmi <image-id>
```

## Entering into the container
```
docker exec -it <container name> /bin/bash
