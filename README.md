# dockerStudy
This project will take the basic information about docker until build an app using it
01 - Docker Commands Overview

docker version
- information about the installed docker

docker run $IMAGE_NAME
- will run create a container for the requested image
- if the image is an image from the docker store (https://store.docker.com/) it will download the image 1st
- images examples = ubuntu, hello-world, java, node, dockersamples/static-site



docker ps
- will display all the RUNNING containers, including ID, IMAGE NAME, STATUS, CONTAINER NAME…

docker ps -a
- will display all the containers RUNNING or NOT including ID, IMAGE NAME, STATUS, CONTAINER NAME…



docker start $CONTAINER_ID
- will start a container

docker stop $CONTAINER_ID
- will stop a container NOT delete it



docker rm $CONTAINER_ID
- will delete the container

docker container stats
- will display the containers and the process information such as CPU usage, memory…

docker inspect $CONTAINER_NAME
- will open the container configuration file

docker container prune
- will delete all NOT RUNNING containers



docker images
- will list down all images downloaded and used by the containers, including IMAGE ID, SIZE…

docker rmi $IMAGE_ID
- will delete the desired image



docker run -d -p 8080:3000 --name test -v "/Users/Documents/workspace:/var/www" -w "/var/www" node npm start
- this command will:
- create a container based on the NODE image (docker run node)
- the “-d” means to not logging in the container
- the opposite of this is the “-ti” parameter
- the “-p” is to setup the port 3000 to 8080 (this port 3000 is in the code of the node application)
- the “--name” allow us to setup a container name, in this case = test
- the “-v” helps to setup a volume in order to persist information from the image in the local machine in the described path
- the “-w” sets up a working point, it means, from where the application entry point should start, in this case is the same persistence directory



docker network ls
- it will list down all the networks
- per default docker creates a network to put all the containers inside. The network is managed by DOCKER HOST. The ip is something such as 172.17.0.1
- to check this information just run the docker inspect $CONTAINER_ID



docker network create --driver bridge $NETWORK_NAME
- it will create your own network to share information in between containers with the given name
- now just create a container using this network. For example:
- docker run -it --name $CONTAINER_NAME --network @NETWORK_NAME
- this new network will be still managed by the DOCKER HOST
- you can use the docker inspect $CONTAINER_ID to check the network configuration
- inside the container you have to install the ping command, because it just have the basic commands
- apt-get update && apt-get install iputils-ping
- then you can use ping to check the communication in between the containers
- ping $CONTAINER_NAME

02 - Docker File Example

The docker file is used to indicate you want to use docker, build an image, container… for the project you are using.

Usually you just name it as:
- dockerfile OR $APP_NAME.dockerfile

The basic setup of a docker file must have at least 8 information:
- FROM (means based in which image you are creating your docker file)
- COPY (means FROM where (in your local machine) TO where (in your image/container) you will copy your project)
- WORKDIR (means the directory to start your app)
- RUN (means which command should be called to start your app)
- ENTRYPOINT (is the file to start your app)
- EXPOSE (means the port will be used by your app)


Example:

FROM node:latest
MAINTAINER Breno Ribeiro
ENV NODE_ENV=production
ENV PORT=3000
COPY . /var/www
WORKDIR /var/www
RUN npm install
ENTRYPOINT npm start
EXPOSE $PORT



docker build -f Dockerfile -t $USER_NAME/IMAGE_NAME.
- will create a image based on the “Dockerfile” mentioned in the project described.
- the “-t user/image” is the name of the image you want to create
- Assuming you are already inside the project folder, as we can see in the “.” in the end



docker login (username / password from docker store)
- docker push $IMAGE_NAME
- it will upload your image to the docker store and place it in your content

03 - Docker Compose YML

The docker-compose.yml file it is a kind of script that help us to setup several containers and images easily. All the configuration will be in 1 file.

After create your docker-compose.yml you can run the following commands

docker-compose build
- this command will build all the images described in the docker-compose.yml file
- P.S.: You need to tun this command from the folder your docker-compose.yml is located



docker-compose up
- this command will up all the services and create everything described in the docker-compose.yml file. It means services, containers, network…



docker-compose ps
- will list all the the services running after you have started them in the up command



docker-compose down
- will stop all the services, containers and remove them



docker-compose.yml file BASIC template. The basic template has at least 3 information:

- version:
- services:
- service1:
- service configuration
- P.S.: Important to note the name of the service should be the same used in the NGINX configuration file or any other load balance tool
- service2:
- service configuration
- networks:
- network configuration


Example:

version: '3'

services:

nginx:
build:
dockerfile: ./docker/nginx.dockerfile
context: .
image: bribeiro/nginx
container_name: nginx
ports:
- "80:80"
- "1234:443"
networks:
- test-network
depends_on:
- "node1"
- "node2"
- "node3"

mongodb:
image: mongo
container_name: mongodb
networks:
- test-network

node1:
build:
dockerfile: ./docker/alura-books.dockerfile
context: .
image: douglasq/alura-books
container_name: alura-books-1
ports:
- "3000"
networks:
- test-network
depends_on:
- mongodb

node2:
build:
dockerfile: ./docker/alura-books.dockerfile
context: .
image: douglasq/alura-books
container_name: alura-books-2
ports:
- "3000"
networks:
- test-network
depends_on:
- mongodb

node3:
build:
dockerfile: ./docker/alura-books.dockerfile
context: .
image: douglasq/alura-books
container_name: alura-books-3
ports:
- "3000"
networks:
- test-network
depends_on:
- mongodb

networks:
test-network:
driver: bridge

04 - Docker Cloud

P.S.: A nice thing about docker cloud is that you can link it to Amazon or Azura or Digital Ocean and you can configure and deploy your app just using the docker cloud interface without touch the 3rd tools.


Repositories
- my projects


Services
- my apps


Containers
- my docker containers


Node and Node Cluster
- place where um install the Docker Host to create the containers in the cloud, it means, it is a virtual machine that contains your app
- you can create more that 1 node, it means, more than 1 virtual machine do deploy your app
- P.S.: Cluster is a collection of several nodes


Stacks
- It is a group of service to create everything your app needs, it is just something such as a docker-compose.yml.
- In this case you need to create a file similar to docker-compose.yml, but with less information. For instance, you don’t have to specify build, dependencies, network…
- But you have specific parameters such as LINKS when you can say with which container it has to communicate with.
- Something such like this:
SERVICE (service name, nginx, mongodb…)
- IMAGE (image name from the docker hub, mongo, ubuntu…)
- PORTS (ports will expose the apps, 80, 443…)
- LINKS (which other service it needs to communicate to)

Example:
nginx:
image: bribeiro/nginx
ports:
- "80:80"
- "1234:443"
links:
- "node1"
- "node2"
- "node3"

mongodb:
image: mongo

node1:
image: douglasq/alura-books
ports:
- "3000"
links:
- "mongodb"

node2:
image: douglasq/alura-books
ports:
- "3000"
links:
- "mongodb"

node3:
image: douglasq/alura-books
ports:
- "3000"
links:
- "mongodb"
