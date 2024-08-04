---
title: "Day 2 : Dockerize a TODO Project"
datePublished: Sun Aug 04 2024 08:07:43 GMT+0000 (Coordinated Universal Time)
cuid: clzfa5ac6000n0ak12g9k9nim
slug: day-2-dockerize-a-todo-project
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1722742600278/58134afd-1ab8-44a8-bb90-eea4b6acc610.png
tags: day-240-hashtag40daysofkubernetes-challenge

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722742714225/5f4de54c-7858-4433-a1e8-9519f05daff8.png align="center")

The workflow for deploying a Dockerfile across different environments such as Development (Dev), Pre-production (Pre-prod), and Production (Prod) involves several key steps. Initially, developers provide the source code along with dependency files, which are used to create a Dockerfile. This Dockerfile is then utilized to build a Docker image that encapsulates the application and its dependencies. Once the image is built, it is pushed to a container registry like Docker Hub or a private registry. From the registry, the image is pulled and deployed in the development environment for testing. After successful testing and validation in Dev, the same image is deployed to the pre-production environment, where further testing is conducted to ensure it meets production standards. Finally, once all tests are completed successfully, the image is deployed to the production environment, ensuring a consistent and reliable deployment process across all stages

## Get started

### **Step -1 : Clone GitHub repository**

```plaintext
dheeraj@docker-node:~/docker-project$ git clone https://github.com/docker/getting-started-app.git
Cloning into 'getting-started-app'...
remote: Enumerating objects: 79, done.
remote: Counting objects: 100% (28/28), done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 79 (delta 17), reused 17 (delta 13), pack-reused 51
Receiving objects: 100% (79/79), 1.76 MiB | 3.38 MiB/s, done.
Resolving deltas: 100% (18/18), done.
dheeraj@docker-node:~/docker-project$ date
Sun Aug  4 07:32:25 AM UTC 2024
dheeraj@docker-node:~/docker-project$ cd getting-started-app/
dheeraj@docker-node:~/docker-project/getting-started-app$ ls -lrt
total 160
drwxrwxr-x 5 dheeraj dheeraj   4096 Aug  4 07:32 src
drwxrwxr-x 4 dheeraj dheeraj   4096 Aug  4 07:32 spec
-rw-rw-r-- 1 dheeraj dheeraj    269 Aug  4 07:32 README.md
-rw-rw-r-- 1 dheeraj dheeraj    648 Aug  4 07:32 package.json
-rw-rw-r-- 1 dheeraj dheeraj 147266 Aug  4 07:32 yarn.lock
dheeraj@docker-node:~/docker-project/getting-started-app$
```

### **Step -2 :** Create Empty Docker-file

```plaintext
dheeraj@docker-node:~/docker-project/getting-started-app$ touch Dockerfile
dheeraj@docker-node:~/docker-project/getting-started-app$ date
Sun Aug  4 07:34:17 AM UTC 2024
dheeraj@docker-node:~/docker-project/getting-started-app$ ls -lrt
total 160
drwxrwxr-x 5 dheeraj dheeraj   4096 Aug  4 07:32 src
drwxrwxr-x 4 dheeraj dheeraj   4096 Aug  4 07:32 spec
-rw-rw-r-- 1 dheeraj dheeraj    269 Aug  4 07:32 README.md
-rw-rw-r-- 1 dheeraj dheeraj    648 Aug  4 07:32 package.json
-rw-rw-r-- 1 dheeraj dheeraj 147266 Aug  4 07:32 yarn.lock
-rw-rw-r-- 1 dheeraj dheeraj      0 Aug  4 07:34 Dockerfile
dheeraj@docker-node:~/docker-project/getting-started-app$
```

### **Step -3 :** Open the file using a text editor

```plaintext
dheeraj@docker-node:~/docker-project/getting-started-app$ vi Dockerfile
dheeraj@docker-node:~/docker-project/getting-started-app$ cat Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000

dheeraj@docker-node:~/docker-project/getting-started-app$ date
Sun Aug  4 07:35:58 AM UTC 2024
dheeraj@docker-node:~/docker-project/getting-started-app$
```

1. **FROM node:18-alpine**: Start with a basic image of Node.js on which our image is based,
    
2. **WORKDIR /app**: Set up a folder inside the container called ‘/app’ where our project will live.
    
3. **COPY . .** : Take all the files ***from our project*** on the computer and ***put them into the ‘/app’ folder*** inside the container.
    
4. **RUN yarn install --production**: Install only the necessary parts of our project to make it run .
    
5. **CMD \[“node”, “src/index.js”\]**: Executing our container using Node.js when it turns on.
    
6. **EXPOSE 3000**: port 3000.
    

### **Step -4 :**Docker Build

```plaintext
dheeraj@docker-node:~/docker-project/getting-started-app$ docker build -t day02-todo .
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  6.539MB
Step 1/6 : FROM node:18-alpine
18-alpine: Pulling from library/node
c6a83fedfae6: Pull complete
1475bb19bdb7: Pull complete
bc6e437c6fa9: Pull complete
62c7e5ec2b01: Pull complete
Digest: sha256:17514b20acef0e79691285e7a59f3ae561f7a1702a9adc72a515aef23f326729
Status: Downloaded newer image for node:18-alpine
 ---> e64c650c0cf3
Step 2/6 : WORKDIR /app
 ---> Running in 77145757eae0
Removing intermediate container 77145757eae0
 ---> f6ebe8b94061
Step 3/6 : COPY . .
 ---> 37ed030453ae
Step 4/6 : RUN yarn install --production
 ---> Running in 707bca03d472
yarn install v1.22.19
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 35.20s.
Removing intermediate container 707bca03d472
 ---> 6fa070459062
Step 5/6 : CMD ["node", "src/index.js"]
 ---> Running in 0ad531770e08
Removing intermediate container 0ad531770e08
 ---> 46adf24e76c3
Step 6/6 : EXPOSE 3000
 ---> Running in 81855410ae98
Removing intermediate container 81855410ae98
 ---> 75c79c41abe5
Successfully built 75c79c41abe5
Successfully tagged day02-todo:latest
dheeraj@docker-node:~/docker-project/getting-started-app$
```

### Verify Docker Image

```plaintext
dheeraj@docker-node:~/docker-project/getting-started-app$ docker images
REPOSITORY               TAG         IMAGE ID       CREATED          SIZE
day02-todo               latest      75c79c41abe5   53 seconds ago   219MB
dheerajlearn/node-goal   latest      9dda9b7bc7b8   6 days ago       1.12GB
goal                     latest      9dda9b7bc7b8   6 days ago       1.12GB
node                     18-alpine   e64c650c0cf3   3 weeks ago      128MB
python                   latest      89cae5f42a4d   3 weeks ago      1.02GB
node                     latest      16d00b0a86e6   7 weeks ago      1.11GB
dheeraj@docker-node:~/docker-project/getting-started-app$ date
Sun Aug  4 07:40:17 AM UTC 2024
dheeraj@docker-node:~/docker-project/getting-started-app$
```

Here is top day02-todo is our image you may different output in your terminal.

### **Step -5 :**Create a public repository on [Docker hub](http://hub.docker.com)

1. **Visit Docker Hub**: Go to [hub.docker.com.](http://hub.docker.com)
    
2. [**Start New R**](http://hub.docker.com)**epository**: Click on [](https://hub.docker.com/)‘Create R[epository’ in](http://hub.docker.com) the ‘Repositories’ section.
    
3. **Name Your Repository**: Enter a [n](https://hub.docker.com/)ame for y[our new reposi](http://hub.docker.com)tory.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722757427279/f61d339b-58ea-4ff6-b93a-608767da1107.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722757551294/786725fe-bf41-47f0-af59-9b4ea4f30e5a.png align="center")

Here my repo name is dheerajlearn/todo-repo.Once you do the you need to rename or tag the image as repo name then only you able to push.  

```plaintext
dheeraj@docker-node:~/docker-project/getting-started-app$ docker tag day02-todo:latest dheerajlearn/todo-repo
dheeraj@docker-node:~/docker-project/getting-started-app$ docker images
REPOSITORY               TAG         IMAGE ID       CREATED         SIZE
dheerajlearn/todo-repo   latest      75c79c41abe5   9 minutes ago   219MB
day02-todo               latest      75c79c41abe5   9 minutes ago   219MB
dheerajlearn/node-goal   latest      9dda9b7bc7b8   6 days ago      1.12GB
goal                     latest      9dda9b7bc7b8   6 days ago      1.12GB
node                     18-alpine   e64c650c0cf3   3 weeks ago     128MB
python                   latest      89cae5f42a4d   3 weeks ago     1.02GB
node                     latest      16d00b0a86e6   7 weeks ago     1.11GB
dheeraj@docker-node:~/docker-project/getting-started-app$ date
Sun Aug  4 07:48:50 AM UTC 2024
dheeraj@docker-node:~/docker-project/getting-started-app$
```

### **Step -6 :** Push Docker Image to the Repository

```plaintext
dheeraj@docker-node:~/docker-project/getting-started-app$ docker push dheerajlearn/todo-repo
Using default tag: latest
The push refers to repository [docker.io/dheerajlearn/todo-repo]
a14cfa5af116: Pushed
053d2b99e6be: Pushed
c6d382521ee5: Pushed
28576ee1ff32: Mounted from library/node
daa732f7b271: Mounted from library/node
7cc8f366b357: Mounted from library/node
78561cef0761: Mounted from library/node
latest: digest: sha256:b4faf6f32bb0a34829be94abb2775cef473401045babe88892f8e79e9b9d6d3a size: 1787
dheeraj@docker-node:~/docker-project/getting-started-app$ date
Sun Aug  4 07:51:13 AM UTC 2024
dheeraj@docker-node:~/docker-project/getting-started-app$
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722757921093/db13004e-d3ff-4f92-8021-23190ef40032.png align="center")

Refresh the docker hug and you will fine the image uploaded as show above with latest tags.

### **Step -7 :** Pull the image into another environment ( Dev , Test , Production )

```plaintext
dheeraj@docker-node:~/docker-project/getting-started-app$ docker pull dheerajlearn/todo-repo
Using default tag: latest
latest: Pulling from dheerajlearn/todo-repo
Digest: sha256:b4faf6f32bb0a34829be94abb2775cef473401045babe88892f8e79e9b9d6d3a
Status: Image is up to date for dheerajlearn/todo-repo:latest
docker.io/dheerajlearn/todo-repo:latest
dheeraj@docker-node:~/docker-project/getting-started-app$ date
Sun Aug  4 07:53:59 AM UTC 2024
dheeraj@docker-node:~/docker-project/getting-started-app$
```

Since i am using the same box where i pushed the image so latest image is already there

### **Step -8 :Run docker container**

```plaintext
dheeraj@docker-node:~/docker-project/getting-started-app$ docker run -dp 3000:3000 dheerajlearn/todo-repo
b7b92e313b68c82ce1b4a836d68421c545791ed8c3dddde484caaa812d0adb85
dheeraj@docker-node:~/docker-project/getting-started-app$ date
Sun Aug  4 07:55:42 AM UTC 2024
dheeraj@docker-node:~/docker-project/getting-started-app$
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722758237762/036cd108-f3c3-4528-9aed-85b9013750f9.png align="center")

Since is am running the docker in virtaul box so i have i given my ip address with port number.  
Reference

**Docker Tutorial For Beginners :**[Day 2 Docker Build](https://www.youtube.com/watch?v=nfRsPiRGx74)

**CKA 2024 Repo:**[Github Repository](https://github.com/piyushsachdeva/CKA-2024/tree/main)