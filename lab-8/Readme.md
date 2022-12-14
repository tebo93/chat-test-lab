# Eighth Lab - Deploy multiple NodeJs app, ReactJs app and RabbitMQ on Kubernetes (minikube). Multi node cluster, using VirtualBox VMs. Stress test and horizontal scaling

In this lab, we are going to deploy both apps on a kubernetes cluster in the local machine using minikune. Each app will be deploy with 3 pod each, and they will be able to talk to each other. Also, our cluster will have 3 nodes. It also will have an autoscaling configuration for the backend.

It will have another cluster running that will generate traffic for backend.

Backend will have 10 replicas (pods), and the frontend will also have 10 replicas (pods), all of them devided evenly on the 3 VMs nodes.


## Solution diagrams

![Deployment Diagram](./lab-images/diagram-lab-1.png)

# Eighth lab instructions

For this lab you may need to have enough resources to run the cluster, we are going to use 12 CPUs and 30000 Mb of disk,(by default the nodes gets 2 cpus and 2000 Mb of disk).

## Create two clusters (app cluster and testing cluster)

Create app cluster:
On terminal:

```
minikube start --driver=virtualbox ---nodes 3 --cpus 3 --memory 10000
```

Create testing cluster:
On terminal:

```
minikube start -p testing-profile --driver=virtualbox --nodes 3 --cpus 2 --memory 20000
```


## Building docker images

Because we are using multiple clusters, the previous configuration will not work. So that there are two options, option one add a registry from which the cluster can pull the images or, two, we can build the image directly into the cluster registry.

### Building backend docker container

On terminal:

```
cd lab-8/backend
minikube -p minikube image build -t chat/backend .

```

### Building frontend docker container

On terminal:
```
cd lab-8/frontend
minikube -p minikube image build -t chat/fronend .
```

### Building testing docker container

On terminal:
```
cd lab-8/frontend
minikube -p testing-profile image build -t chat/test-backend .
```

## Install minikube addons

These are the addos to be installed:
* ingress
* dashboard
* metrics-server

To install them, run on terminal:

```
minikube -p minikube addons enable dashboard
minikube -p minikube addons enable metrics-server
minikube -p minikube addons enable ingress

minikube -p testing-profile addons enable dashboard
minikube -p testing-profile addons enable metrics-server
```

Note: the ingress addons can take some time to be installed.

## Deploy yaml file

### Deploy solution - for default cluster

On terminal:

``` 
kubectl context use-context minikube
cd lab-8/deployment

kubectl create -f deployment_config.yml
kubectl create -f deployment_broker.yml
kubectl create -f deployment_frontend.yml
kubectl create -f deployment_backend.yml
```

### Deploy solution - for testing cluster

``` 
kubectl context use-context testing-profile
cd lab-8/deployment

kubectl create -f deployment_config.yml
kubectl create -f deployment_test_backend.yml
```

## Get minikube cluster ip

On terminal:

```
minikube ip
```

## Add domains into hosts file

On ubuntu, open /etc/hosts file as sudo by

```
cd ; cd .. ; cd .. ; cd etc ; sudo open hosts
```

in the hosts file, add map of ip to domain.
```
Cluster-IP  domain
```

for the lab it will be:

```
ClusterIP   api.chat-test.com
ClusterIP   chat-test.com
```

For my local environment it looks like:

```
192.168.59.186  api.chat-test.com
192.168.59.186  chat-test.com
```

## Open dashboard 

on terminal:

```
minikube -p minikube dashboard
```


on another terminal:

```
minikube -p testing-profile dashboard
```

## Increase test-backend replicas to 80

To increase the replica, you can go to the testing-profile dashboard, go to deplyoments, and click on the 3 dots button and click the option scale, set the scale to 80.

Or on terminal:

```
kubectl config use-context testing-profile
kubectl scale deployment test-backend --replicas=50
```


After a couple of seconds, the Horizontal Auto Scale must start creating new pods of chat-backend-master.


Then you can reduce the number of replicas for the test-backend to 5, and after a couple of minutes, the Horizontal Auto Scale should start removing some pods.