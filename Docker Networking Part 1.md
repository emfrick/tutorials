# Docker Networking
The purpose of this write-up is to create a web application using Docker containers.  We will be creating an application with 3 containers:
1) A MongoDB container
2) A NodeJS container
3) Javascript Client container

The MongoDB container should not be exposed to the outside world, and so we must create a Docker network.

The first part of this series will be running a NodeJS application inside a Docker container.

### NodeJS Application

```javascript
const express = require('express');
const app     = express();

app.get('/', (req, res) => {
    res.json({
        message: "Hello World"
    });
})


app.listen(3000, () => {
    console.log("Listening on port 3000");
})
```

### Dockerfile
```
FROM node:boron

# App Directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# Install dependencies
COPY package.json /usr/src/app
RUN npm install

# Bundle app source
COPY . /usr/src/app

# Bind to port 3000
EXPOSE 3000

# Start npm
CMD ["npm", "start"]
```

### Directory Structure (app location)

 - List item
 - .dockerignore
 - Dockerfile
 - package.json
 - server.js


### Docker Commands

#### Create a Docker image
`$ docker build -t <name:tag> .`

#### View networks
`$ docker network ls`

#### Create a new network
`$ docker network create --driver=bridge <network_name>`

#### Running a container on a specific network
`$ docker run --name <container_name> --network <network_name> -d <image_name>`

#### Connecting an existing container to a specific network
`$ docker network connect <network_name> <container_name>`