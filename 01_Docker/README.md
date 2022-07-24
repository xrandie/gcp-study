# Docker

## Commands

### Hello World
```bash
docker run hello-world
docker images

docker run hello-world
docker ps
docker ps -a
```

### Build Docker Images
```bash
# mkdir test && cd test
cd test
cat Dockerfile  # <<EOF
                # ...
                # EOF
cat app.js  # <<EOF
            # ...
            # EOF

docker build -t node-app:0.1 .
docker images
```

### Run Docker Images
```bash
docker run -p 4000:80 --name my-app node-app:0.1

# Open 2nd Cloud Shell
curl http://localhost:4000

# Close 1st Cloud Shell
# curl http://localhost:4000            # Optional
docker stop my-app && docker rm my-app
docker run -p 4000:80 --name my-app -d node-app:0.1
docker ps

docker logs [container_id]
```

```bash
# vi test/app.js
# ....
# const server = http.createServer((req, res) => {
#     res.statusCode = 200;
#       res.setHeader('Content-Type', 'text/plain');
#         res.end('Welcome to Cloud\n');
# });
# ....

docker build -t node-app:0.2 .
docker run -p 8080:80 --name my-app-2 -d node-app:0.2
docker ps

curl http://localhost:8080
curl http://localhost:4000
```

### Debugging
```bash
docker logs -f [container_id]
docker exec -it [container_id] bash

# in the Container
ls
exit

docker inspect [container_id]
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' [container_id]
```

### Deploy
```bash
REGISTRY=gcr.io
PROJECT_ID=$GOOGLE_CLOUD_PROJECT  # Your Project ID
IMAGE=node-app
TAG=0.2

IMAGE_NAME=$REGISTRY/$PROJECT_ID/$IMAGE:$TAG

docker tag node-app:0.2 $IMAGE_NAME
docker images
docker push $IMSAGE_NAME

# GCR URL
# echo https://gcr.io/$PROJECT_ID
```

```bash
docker stop $(docker ps -q)
docker rm $(docker ps -aq)

docker rmi $IMAGE_NAME
docker rmi node:6
docker rmi $(docker images -aq) # remove remaining images
docker images

docker pull $IMAGE_NAME
docker run -p 4000:80 -d $IMAGE_NAME
curl http://localhost:4000
```

### Connect to exposed port on the browser
![image](https://user-images.githubusercontent.com/21324361/180647920-84ffa0df-4ce1-4c66-8d97-b7142dcf8966.png)
