---
title: "Deployed a Reddit Copy on Kubernetes with Ingress"
datePublished: Tue Sep 05 2023 06:23:15 GMT+0000 (Coordinated Universal Time)
cuid: clm5xcfb600180alc6lwi66a9
slug: deployed-a-reddit-copy-on-kubernetes-with-ingress
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1693570161253/9cd7e5cf-a33f-4a84-b669-c83014eb5669.jpeg

---

Overview:

It's a simple Reddit clone application where the developer pushes the code on GitHub and as DevOps engineers we need to deploy the application. Here we first build the application using docker on the CI server and we will push the image to docker-hub for the versioning. Later we will deploy the application on the deployment server using kubernetes.

Below is a diagrammatic view:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693554903726/2da4adbb-df45-4d9b-acc5-8a7da6715547.png align="center")

## Prerequisites

Before we get started, make sure you have the following tools and services installed and configured:

* Two AWS EC2 instances with Ubuntu 20.04 LTS installed of instance type t2.medium
    
* Basic knowledge of Kubernetes concepts and YAML syntax
    

Since Kubernetes is a cluster-required service like API ,etcd and other service its minimum required is 2 CPU and 4 GB of RAM.

### Step 1 Installing Kubernetes dependencies

The first step in deploying our Flask API and MongoDB on Kubernetes is to set up a Kubernetes cluster. There are 2 ways we do it

1. minikube
    
2. kubeadm
    

In this demo, we are using the minikube.

### **Step 2: Clone the Code from Github**

On the CI-Server,clone the codebase of your Reddit clone web application.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693890456146/ac0e642b-31b4-4b6e-866b-ea467a208d9e.png align="center")

### **Step 3: Prerequisites**

**On Deployment Server**:

* Update the server: `sudo apt-get update`
    
* Install Docker: `sudo apt-get install` [`docker.io`](http://docker.io) `-y`
    
* Permit Docker: `sudo usermod -aG docker $USER && newgrp docker`
    
    ### **Step 4: Write a Dockerfile for the Project and build the image**
    

Go to the clone folder checkout the docker file and build the image.

```bash
FROM node:19-alpine3.15

WORKDIR /reddit-clone

COPY . /reddit-clone

RUN npm install

EXPOSE 3000

CMD [“npm”,”run”,”dev”]
```

docker build . -t devopslove/reddit-clone

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693890810988/3faf25a6-0df1-4205-b96b-79008efcba3f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693890825165/f4da448e-419a-4c66-b343-307af3ed4e68.png align="center")

After building the image, push it to Docker Hub by running.Firstly we need to login dokcer hub and then pushing the image to dokcerhub.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693891012317/27b627df-445b-4583-8293-fe0ca947ba14.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693891025585/0d740ea5-019f-49b2-92d1-7eeab52715fd.png align="center")

### **Step 5:On Deployment Serve**

Install docker and minikube.

```bash
sudo apt-get update

sudo apt-get install docker.io -y

sudo usermod -aG docker $USER && newgrp docker  

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

  sudo install minikube-linux-amd64 /usr/local/bin/minikube

  sudo snap install kubectl — classic

  minikube start — driver=docker
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693891374099/08fd8818-4eae-433a-b57d-703f425fe85e.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693891386614/7b2ef9f9-c7f8-4b3d-84b1-d4fe657ce490.png align="center")

### **Step 6: Create a Kubernetes Folder**

Create a folder named "K8s" on the Deployment Server.

### **Step 7: Create a Deployment YAML File**

Inside the "K8s" folder, create a file named "Deployment.yml" with following contect and apply deployment to minikube

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-clone-deployment
  labels:
    app: reddit-clone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: reddit-clone
  template:
    metadata:
      labels:
        app: reddit-clone
    spec:
      containers:
        - name: reddit-clone
          image: devopslove/reddit-clone
          ports:
            - containerPort: 3000
```

```bash
kubectl apply -f Deployment.yml
kubectl get deployments
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693891731184/9fa9238f-74d1-4318-8db2-51cff3ef8531.png align="center")

### **Step 8: Create a Service YAML File**

Inside the "K8s" folder, create a file named "Service.yml" and add apply the service

```bash
apiVersion: v1
kind: Service
metadata:
  name: reddit-clone-service
  labels:
    app: reddit-clone
spec:
  type: NodePort
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 31000
  selector:
    app: reddit-clone
```

```bash
kubectl get services
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693893606133/fae7c372-b0f0-4af3-8c39-3de6b28145be.png align="center")

### **Step 9: Expose the Application to the Internet**

To make your application accessible from the internet, you'll need to configure port forwarding.

```bash
kubectl port-forward svc/reddit-clone-service 3000:3000
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693893808620/b8ab0eb1-b5a2-4a59-a0ce-9546ae043dc8.png align="center")

### **Step 10: Access the Application**

```bash
minikube service reddit-clone-service — url
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693893909520/e824f771-f4b7-4d1e-b30e-4bf1259782fe.png align="center")

## Ingress:

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693894339098/53457d21-4db8-4075-99a9-84acc8b09c67.png align="center")

Let's write ingress.yml and put the following code in it:

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-reddit-app
spec:
  rules:
  - host: "domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
  - host: "*.domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
```

Minikube doesn't enable ingress by default; we have to enable it first using `minikube addons enable ingress` command.

* If you want to check the current setting for addons in minikube use `minikube addons list` command.
    

Now you can able to create ingress for your service. `kubectl apply -f ingress.yml` use this command to apply ingress settings.

* Verify that the ingress resource is running correctly by using `kubectl get ingress ingress-reddit-app` command.
    

# Test Ingress

Now It's time to test your ingress so use the ***curl -L domain/test*** command in the terminal.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693894681944/7a801904-6f45-4965-8d8f-a9aaaa7f608f.png align="center")

You can also see the deployed application on Ec2\_ip:3000

***Congratualtion💥💥 Your* Application *project is completed and your Reddit account created***