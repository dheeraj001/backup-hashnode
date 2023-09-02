---
title: "Deploying a Flask API and MongoDB on Kubernetes"
datePublished: Sat Sep 02 2023 19:47:09 GMT+0000 (Coordinated Universal Time)
cuid: clm2fqowq000409jvci69g2mz
slug: deploying-a-flask-api-and-mongodb-on-kubernetes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1693681016216/7a869f16-bc43-4a2a-8849-65a8b1c49794.png

---

Kubernetes is a powerful container orchestration platform that allows you to deploy and manage complex applications at scale. In this tutorial, we'll walk through the process of setting up a Kubernetes cluster on AWS EC2 and deploying a Flask API and a MongoDB instance on the cluster.

## Prerequisites

Before we get started, make sure you have the following tools and services installed and configured:

* Two AWS EC2 instances with Ubuntu 20.04 LTS installed of instance type t2.medium
    
* Basic knowledge of Kubernetes concepts and YAML syntax
    

Since Kubernetes is a cluster-required service like API ,etcd and other service its minimum required is 2 CPU and 4 GB of RAM.

### Step 1.1: Installing Kubernetes dependencies

The first step in deploying our Flask API and MongoDB on Kubernetes is to set up a Kubernetes cluster. There are 2 ways we do it

1. minikube
    
2. kubeadm
    

In this demo, we are using the kubeadm as it is major used in Production deployment \\

### Step 1.1: Installing Kubernetes dependencies

Since Kubernetes is a cluster server it has a minimum of 2 nodes one is the master server which manages the cluster environment and the other is the worker node where the app or work is schedule.

First, we need to install some dependencies on our EC2 instance (both master and worker):

```bash
sudo apt update -y
sudo apt install docker.io -y

sudo systemctl start docker
sudo systemctl enable docker

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update -y
sudo apt install kubeadm=1.20.0-00 kubectl=1.20.0-00 kubelet=1.20.0-00 -y
```

### Step 1.2: Initializing the cluster

```bash
sudo su
kubeadm init
```

### Step 1.2: Initializing the cluster

Next, we need to initialize the Kubernetes cluster on the master node:

```bash
sudo su
kubeadm init
```

This command will create a new Kubernetes cluster with the default configuration and networking settings.

### Step 1.3: Configuring kubectl

After initializing the cluster, we need to configure `kubectl` to connect to it:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

This will copy the Kubernetes configuration file to your local machine and configure `kubectl` to use it.

### Step 1.4: Installing the Weave Net CNI plugin

Then we will deploy the Weave Net CNI plugin as a DaemonSet in your Kubernetes cluster. This will allow the Weave Net plugin to be run on every node in the cluster.

```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

### Step 1.5: Generating token to join nodes

And then will generate a new bootstrap token for joining new nodes to our Kubernetes cluster

```bash
kubeadm token create --print-join-command
```

### Step 1.6: Joining the worker node

Finally, we need to join the worker node to the cluster. On the worker node, run the command that we goy in output from command in step 1.5:

```bash
sudo su
kubeadm reset pre-flight checks
Paste the Join command on worker node and append `--v=5` at end
```

Here, I have used `kubeadm reset pre-flight checks` because it performs a series of pre-flight checks to ensure that the node is in a clean state and can be used as a worker node.

Congratulations! You now have a functioning Kubernetes cluster with one master and one worker node.

## Step 2: Deploying the Flask API

Now that we have our Kubernetes cluster set up, we can deploy the Flask API on it. I'll use a Docker image that I've already created and pushed to Docker Hub.

### Step 2.1: Creating the deployment

To create a deployment for the Flask API, we need to define a YAML file that describes the deployment:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: taskmaster
  labels:
    app: taskmaster
spec:
  replicas: 1
  selector:
    matchLabels:
      app: taskmaster
  template:
    metadata:
      labels:
        app: taskmaster
    spec:
      containers:
        - name: taskmaster
          image: trainwithshubham/taskmaster-python:latest
          ports:
            - containerPort: 5000
          imagePullPolicy: Always
```

Save this file as `taskmaster.yml` and deploy it to the Kubernetes cluster with the following command:

```bash
kubectl apply -f taskmaster.yml
```

This will create a new deployment for the Flask API with one replica.

### Step 2.2: Scaling the deployment

To scale the deployment to three replicas, run the following command:

```bash
kubectl scale --replicas=3 deployment/taskmaster
```

This will increase the number of replicas of the deployment to three.

### Step 2.3: Creating the service

Next, we need to create a Kubernetes Service to expose the Flask API to the outside world:

```bash
apiVersion: v1
kind: Service
metadata:
  name: taskmaster-svc
spec:
  selector:
    app: taskmaster
  ports:
    - port: 80
      targetPort: 5000
      nodePort: 30007
  type: NodePort
```

Save this file as `taskmaster-svc.yml` and deploy it to the Kubernetes cluster with the following command:

```bash
kubectl apply -f taskmaster-svc.yml
```

This will create a new Kubernetes Service with a NodePort that maps port 80 to port 5000 on the Flask API container.

### Step 2.4: Accessing the API

Now that we have deployed the Flask API and created a Kubernetes Service for it, we can access it from outside the cluster using the IP address of the worker node and the NodePort we specified in the `taskmaster-svc.yml` file.

You can access the API at [`http://<ip-of-worker-node>:30007/`](http://52.23.180.54:30007/).

## Step 3: Deploying MongoDB

Now that we have the Flask API up and running, we need to deploy a MongoDB instance to store the data for the API. I'll use PersistentVolumeClaim to ensure that the data is persisted even if the MongoDB container is restarted.

### Step 3.1: Creating the PersistentVolume

First, we need to create a PersistentVolume to store the data for MongoDB:

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv
spec:
  capacity:
    storage: 256Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp/db
```

Save this file as `mongo-pv.yml` and deploy it to the Kubernetes cluster with the following command:

```bash
kubectl apply -f mongo-pv.yml
```

This will create a new PersistentVolume with 256MB of storage.

### Step 3.2: Creating the PersistentVolumeClaim

Next, we need to create a PersistentVolumeClaim to request storage from the PersistentVolume we just created:

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 256Mi
```

Save this file as `mongo-pvc.yml` and deploy it to the Kubernetes cluster with the following command:

```bash
kubectl apply -f mongo-pvc.yml
```

This will create a new PersistentVolumeClaim that requests 256MB of storage.

### Step 3.3: Creating the Mongo Deployment:

Finally, we can create the Deployment for MongoDB:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
  labels:
      app: mongo
spec:
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: storage
              mountPath: /data/db
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: mongo-pvc
```

Save this file as `mongo.yml` and deploy it to the Kubernetes cluster with the following command:

```bash
kubectl apply -f mongo.yml
```

This will create a new Deployment with one replica that runs the official MongoDB container and mounts the persistent volume claim we created earlier.

### Step 3.4: Creating the MongoDB Service

Finally, we need to create a Service to expose the MongoDB instance to the Flask API:

```bash
apiVersion: v1
kind: Service
metadata:
  labels:
    app: mongo
  name: mongo
spec:
  ports:
    - port: 27017
      targetPort: 27017
  selector:
    app: mongo
```

Save this file as `mongo-svc.yml` and deploy it to the Kubernetes cluster with the following command:

```bash
kubectl apply -f mongo-svc.yml
```

This will create a new Service that maps port 27017 to the MongoDB container.

## Step 4: **Test the Flask API**

With the MongoDB database and Flask API deployed, we can now test the API to ensure that everything is working as expected.

We will log in to Postman on Google to test out API.

Click on Collection and request for the GET and Post API.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693683196801/eabcef55-4432-4d13-93ac-5e08b165926e.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693683218272/35d90fe1-4170-4036-b665-a5dca669de6d.png align="center")

### Error: "ServerSelectionTimeoutError: mongo:27017"

`pymongo.errors.ServerSelectionTimeoutError`

Once the Mongo db is done and you check the flask container on the worker node you will find the above One of the solutions that I found is to restart the Coredns service.  
Here is a command.  
  
`kubectl delete pods -n kube-system -l k8s-app=kube-dns`

## Conclusion

While deploying the above microservice you will find many issues most of the time I tried to paste the error in Google and find the solution

Use the below link to understand more.  

[Video related to the above microserver deployment](https://www.youtube.com/watch?v=LPaWASGjwbs)