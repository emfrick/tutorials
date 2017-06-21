# Docker Networking
The purpose of this write-up is to create a web application using Docker containers.  We will be creating an application with 3 containers:
1) A MongoDB container
2) A Node.js container
3) JavaScript Client container

The MongoDB container should not be exposed to the outside world, and so we must create a Docker network.

The first part of this series will be running a Node.js application inside a Docker container.

### Node.js Application
Let's begin with a simple Node application that responds with some JSON.

Initialize the project with ` $ npm init`.  Change the "main" to "server.js" when prompted.  All other values can remain the defaults.

Install [Express](https://expressjs.com/) with `$ npm install express --save`.
Install nodemon to help with ease of development `$ npm install nodemon --save-dev`.

Now create the `server.js` file.
```javascript
const express = require('express');
const app     = express();

app.get('/', (req, res) => {
    res.json({
        message: "Hello World"
    });
});


app.listen(3000, () => {
    console.log("Listening on port 3000");
});
```

Now open up the package.json file and add the following line to the "scripts" section:
`"dev": "nodemon"`

Now test that the application is working by running `npm run dev`, then open a browser and navigate to `http://localhost:3000`.

You should see the JSON response
```json
{"message":"Hello World"}
```

### Dockerfile
Now it's time to put the application into a Docker container.
Add a file called "Dockerfile" with the following contents:
```
FROM node:boron

# App Directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# Install dependencies
COPY package.json /usr/src/app
RUN npm install

# Mount the host source directory to the container
VOLUME /home/docker/Projects/checkbook-api /usr/src/app

# Start npm
CMD ["npm", "run", "dev"]
```

###.dockerignore
Create the .dockerignore file to exclude the node_modules directory and the npm-debug log file from the Docker image.
```bash
# .dockerignore
node_modules
npm-debug.log
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

#### View new image
```
$ docker images
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
emfrick/checkbook-api      latest              a4da94d87164        10 seconds ago      666MB
```

#### Run the container
`$ docker run --name checkbook-api --volume /home/docker/Projects/checkbook-api:/usr/src/app --publish 3000:3000 --detach emfrick/checkbook-api`

#### View running containers
```
$ docker ps
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS              PORTS                    NAMES
3556ad267ba6        emfrick/checkbook-api   "npm run dev"       2 minutes ago       Up 2 minutes        0.0.0.0:3000->3000/tcp   checkbook-api
```

Now the container is running, but the ports are exposed to the host.  If we wanted to run 3 containers with this same image, we would need to expose more ports on the host (e.g. 3000, 3001, 3002).

A better approach is to create a network of containers.

#### View networks
```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
aa21d007293f        bridge              bridge              local
49387594a1d0        host                host                local
c003274b8356        none                null                local
```
You can see the bridge network is automatically created.  This is the default network that each container binds to on startup.

#### Create a new network
Now lets create a new network for all of our application containers.
`$ docker network create --driver=bridge <network_name>`

And see the new network:
```
$docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
c59e6bebd0d9        app-network         bridge              local
aa21d007293f        bridge              bridge              local
49387594a1d0        host                host                local
c003274b8356        none                null                local
```

#### Connecting an existing container to a specific network
Now connect the running application container to the internal network
`$ docker network connect <network_name> <container_name>`

#### Inspect the new network
Verify the running container has been attached to the new network:
`$ docker network inspect app-network`

Output:
```json
[
    {
        "Name": "app-network",
        "Id": "c59e6bebd0d9876cead0fc8be2fec95fd56cbec92b32cadfab6db39c74662735",
        "Created": "2017-06-21T11:11:12.525953655-04:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "Containers": {
            "3556ad267ba63f6f6ea8ae3cae982f29342e3d1e68fa9f0e4fbc4996eda4ed47": {
                "Name": "checkbook-api",
                "EndpointID": "45ba6a41af848a7a8263d617e4c7203e087d258f8edc1bb1fa923645411e199a",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

#### Test
Open a browser and navigate to `http://172.18.0.2:3000` and verify that you still receive the same JSON output.
```json
{"message":"Hello World"}
```

You can still access the API from the localhost, which is not what we want.  So let's remove our running container and start it up so it can only be accessed from our custom network.

#### Stop and remove the running container
```bash
$ docker stop <container_name>
$ docker rm <container_name>
```

#### Running a container on a specific network
Now lets restart that container without publishing its ports to the host:
`$ docker run --name <container_name> --volume /home/docker/Projects/checkbook-api:/usr/src/app --network <network_name> --detach <image_name>`

#### Test
Now open a browser and navigate to `http://localhost:3000` and verify that the application is no longer available at that address.  However, it is now available on our custom network at `http://172.18.0.2:3000`.

Now we can start up several application containers, and simply change the container name, and they will each listen on their own internal IP address on port 3000.
```bash
$ docker run --name checkbook-api2 --volume /home/docker/Projects/checkbook-api:/usr/src/app --network app-network --detach emfrick/checkbook-api
$ docker run --name checkbook-api3 --volume /home/docker/Projects/checkbook-api:/usr/src/app --network app-network --detach emfrick/checkbook-api
```
Inspect the network
`$ docker network inspect app-network`

Output:
```json
[
    {
        "Name": "app-network",
        "Id": "c59e6bebd0d9876cead0fc8be2fec95fd56cbec92b32cadfab6db39c74662735",
        "Created": "2017-06-21T11:11:12.525953655-04:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "Containers": {
            "46d3554e61dde565cda613e01651e23f595b8621093eb906f2ec08f6564d3b5b": {
                "Name": "checkbook-api3",
                "EndpointID": "f959ea2321b9e3d7de2507e3aac49bcd6697c120e9978053b38295a35ce2605e",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "957ef010448df1116689f82139a05b58b6a317b66bf2c44659510ff3c2b5195d": {
                "Name": "checkbook-api",
                "EndpointID": "cbf086b8ea5d1d056cc66267209b7660665f90412edc5046cab6eecca035e52a",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "e8f469f7be33bf223a1dac6d945bce085cb900586c717748870da52444352429": {
                "Name": "checkbook-api2",
                "EndpointID": "8184471f67e8f9a987dc750b59e9b2675809de43ae9ab5ebdc5bddd3ae23b40c",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

### Conclusion
We now have 3 application containers running our code (mounted from /home/docker/Projects/checkbook-api) in an isolated network.

What does this buy us?
Our application is no longer exposed to the outside world and cannot be accessed directly.  This allows us to spin up new containers as often as needed.  In order to access those containers, we will be creating a load balancer and exposing that to the world in the next tutorial.