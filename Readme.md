https://www.linkedin.com/learning/deploying-and-running-apache-kafka-on-kubernetes/introduction-to-this-course?u=95233017
Kate Stanley

## INSTALL and START
1) download kafka and unzip to folder tar -xvf kafka  
2) start zookeeper at 2181: ./bin/zookeeper-server-start.sh config/zookeeper.properties  
3) start kafka broker at 9092: ./bin/kafka-server-start.sh config/server.properties: KafkaServer id=0 started   

## PRODUCE/CONSUME to/from TOPIC
1) create topic:
./bin/kafka-topics.sh --create --topic mytopic --replication-factor 1 --partitions 1 --bootstrap-server localhost:9092
a) replication factor = number of brokers (allows for availability)
b) partitions = number of consumers (allows for scaling consumption)
2) verify topic is created: 
./bin/kafka-topics.sh --list --bootstrap-server localhost:9092
3) ./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic mytopic
4) ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mytopic --from-beginning

## START KAFKA CLUSTER WITH 3 BROKERS
1) cp config/server.properties config/server-1.properties
2) change id, port and log directory
broker.id=1
listener=PLAINTEXT://9093
log.dirs=/tmp/kafka-logs-1
3) Create topic to replicate to 3 brokers
./bin/kafka-topics.sh --create --topic mytopic --replication-factor 3 --partitions 1 --bootstrap-server localhost:9092
4) describe topic
./bin/kafka-topics.sh --describe --topic mytopic --bootstrap-server localhost:9092
### NOTE: Leader broker is broker 1- Only one of the brokers is the leader for the topic
### Message automatically rerouted to Leader broker 1 regardles to which broker you connect to
5) add log4j config file for producer and set env var to use it:
export KAFKA_OPTS="-Dlog4j/configuration=file:config/debug-tools-log4j.properties"
6) produce to topic via non leading broker 2:
./bin/kafka-console-producer.sh --topic mytopic --bootstrap-server localhost:9093
7) meta.properties - each broker has its own, it must have broker.id matching the id from server.properties

##  Kubernetes StatefulSets for brokers
1) unique identity
2) smooth updates
3) keep same underling storage

## INSTALL DOCKER DESKTOP / KUBERNETES
1) download from docker.com/products/docker-desktop or minikube
2) start docker: docker
3) docker run hello-world
4) enable kubernetes
5) kubectl get nodes

## DEPLOYING AND RUNNING SINGLE KAFKA BROKER ON KUBERNETES
1) start zookeeper (not running in kubernetes)
2) kubectl get nodes
3) create statefulset-kafka.yaml

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: kafka
spec:
  selector:
    matchLabels:
      app: kafka
  serviceName: "kafka"
  replicas: 1
  template:
    metadata:
      labels:
        app: kafka
    spec:
      containers:
      - name: kafka
        image: debezium/kafka
        ports:
        - containerPort: 9092
          name: kafka
        env:
        - name: BROKER_ID
          value: '0'
        - name: ZOOKEEPER_CONNECT
          value: 'host.docker.internal:2181'


4) kubectl apply -f statefulset-kafka.yaml
5) kubectl get pods -w
6) kubectl logs kafka-0
7) kubectl exec -it kafka-0 -- /bin/bash
8) ./bin/kafka-topics.sh --list --bootstrap-server kafka-0:9092
9) exit
10) Verify pod gets auto cretad when deleted:
kubectl get pods -w
pod "kafka-0" deleted


## GAINING ACCESS TO APACHE KAFKA

### Listeners and advertised listeners

gadg






https://www.linkedin.com/learning/docker-your-first-project/setting-up-your-development-environment-30191052?autoSkip=true&resume=false&u=95233017
Shelley Benhoff

## Goals:  
Project creates container image for Python Flusk Web Site  
1) set up dcoker development environment 
2) write a Dockerfile  
3) build and manage custom images  
4) run your Docker containers   

## Setup Docker Development Environment
1) Docker Desktop - download, start and login  
2) VS Code  
3) Docker Extension for VS Code - "Container Tools"  
4) git bash
5) git clone https://github.com/linkedInLearning/docker-your-first-project-4485003  



# IMAGES

## Create dockerfile

### Use an official Python runtime as the base image
FROM python:3.12-rc-bookworm  
### Set the working directory in the container to /app
WORKDIR /app  
### Copy the current directory contents into the container at /app
COPY . /app  
### Install the required packages
RUN pip install --no-cache-dir -r requirements.txt  
### Set the environment variable for Flask
ENV FLASK_APP=app.py  
### Run the command to start the Flask application
CMD ["flask", "run", "--host=0.0.0.0"]


## Build image from dockerfile
open new bash terminal
cd /c/SOURCE/java/BasicDocker/docker-your-first-project-4485003
docker build .

## finding base image
docker search python
docker search --filter is-official=true python
docker search --filter stars=100 python
docker search --no-trunc python  (full output)

## pull base image (alwasy pulls the latest version of image)
docker image pull python

## to see local image(s)
docker image ls 
docker image ls --no-trunc

## to tag your image (latest is default)
docker build -t big-star:v1 -t big-star .

## Push/pull image to/from docker repo with CLI
create private repo on docker hub e.g. glatisko/my_repo  
docker login 
tag your local image using new repo name: docker tag big-star:v2 glatisko/my_repo:big-star  
docker push glatisko/myrepo:big-star  
docker pull glatisko/my_repo:big-star  

## inspect image: 
docker image ls  
docker image inspect big-star:v2  

## Removing images
docker image ls  
remove tagged image: docker rmi -f big-star:v2
to remove all tags associated with the image: docker rmi -f image_id


# CONTAINERS

## Run a Container
1) Option 1 (reastarts stopped container): docker start big-star:v2
2) !!Option 2 (creates and starts new container): docker run big-star:v2

## Run container in Interactive mode
docker run -it -d -p 5000:5000 -v ${PWD}:/app/code big-star:v2  
localhost:5000

## stop/kill container
docker stop big-star:v2  
docker kill big-star:v2  

## show running containers
docker ps  

## show containers running and stopped
docker ps -a   

## inspect container
docker inspect big-star:v2  

## review container log file
docker logs --tail --details -f big-star:v2  

# VOLUMES AND MOUNTS
Volume - directory on host that is accessible to the container  


## create volume
docker volume create big_volume  

## list/inspect volumes
docker volume ls  
docker volume inspect big_volume  

## atach volume to container
docker run -it -d -p 5000:5000 -v big_volume:/app/data_in_container  


# CLEANUP 
You need to remove daily images/containers/volumes after you are done  

## to remove dangling images 
docker image prune  
docker container prune  
docker volume prune  

## remove images not referenced by existing containers
docker image prune -a -f  

## to prune all images/containers/networks/volumes
docker system prune --volumes -f  












