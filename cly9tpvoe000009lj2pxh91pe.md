---
title: "Day 1 : Docker Fundamentals"
datePublished: Sat Jul 06 2024 07:49:17 GMT+0000 (Coordinated Universal Time)
cuid: cly9tpvoe000009lj2pxh91pe
slug: day-1-docker-fundamentals
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1720248236539/a95bb314-c28a-478c-8aee-0cabf3c8a44e.png
tags: 40daysofkubernetes

---

### **What is Docker ?**

Docker is a tool that simplifies creating, deploying, and running applications by packaging them into containers, which include all necessary components like code, libraries, and settings. This ensures that applications run consistently across different environments without any configuration issues. By using Docker, developers can easily move applications between development, testing, and production, ensuring reliability and efficiency.

## Understanding Containers V/S Virtual Machines

Docker and virtual machines (VMs) are both used to run applications in isolated environments, but they do so in fundamentally different ways

## Docker (Containers):

* **Lightweight**: Containers share the host operating system's kernel, making them more lightweight and faster to start up.
    
* **Resource Efficiency**: Uses less system resources (CPU, memory) because containers share the OS kernel.
    
* **Isolation**: Provides process-level isolation but shares the OS kernel, leading to less overhead.
    
* **Portability**: Containers can be easily moved across different environments and platforms.
    
* **Deployment**: Ideal for microservices and applications that need to be deployed quickly and scaled easily.
    
* **Startup Time**: Very fast, typically in seconds
    

## **Virtual Machines (VMs):**

* **Heavyweight**: Each VM runs its own guest operating system on top of a hypervisor, making them heavier and slower to start.
    
* **Resource Consumption**: Uses more system resources because each VM includes a full OS instance.
    
* **Isolation**: Provides full isolation as each VM operates independently with its own OS, offering strong security boundaries.
    
* **Portability**: VMs are less portable due to the larger size and dependency on specific hypervisor configurations.
    
* **Deployment**: Suitable for running multiple applications that require different OS versions or configurations.
    
* **Startup Time**: Slower, typically in minutes.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720248805560/08e1dfd3-1c63-48bf-9976-befbc9afd591.png align="center")

# Challenges with the non-containerized applications

***" It works on my machine "***

Software can sometimes work fine when developers create it and test it, but still have problems when deploy to production. This happens because the environments where the software is created and tested aren't exactly the same as where it is used, causing unexpected issues. Containers help solve this by including everything the software needs to work properly, making sure it runs the same in all environments.

# Docker Work Flow

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720249057461/79f69715-fad2-4c16-8072-423bb164bad9.png align="center")

When a new application or feature is added, the process typically begins by updating or creating a Dockerfile. This Dockerfile contain set the set of instruction like where is code , which path need to run . Once the Dockerfile is ready, we build an image from it and push this image to a registry, such as Docker Hub, which acts like a version control system for images.Image contain all the code and dependency but its static not in running and its read only so we edit any image. Depending on the environment, we then pull the appropriate image and run it,its create container what is actual running instance of image.We can have many instance(container) of image. Now we start with dev test envioement once all good same we can deploy in prod.This integration ensures consistency and reliability across different environments, from development to production

# Docker Architecture

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720250001414/010da50a-90db-4682-b2ad-0d619c9781c8.png align="center")

Here will try to understand how docker run an image on the operating system.so whenever run docker build command on you laptop/system where is docker install.it interact with docker daemon.Docker daemon read the instruction form docker file and create an image .That image store in local register that comes when you install docker.Now push the image from local registry to cloud registry like docker-hub,nexus or j-frog using docker push.Now once image available in cloud you can pull the image to any environment using docker pull and run it with docker run.

# Reference

**Docker Tutorial For Beginners :** [Docker Fundamental's](https://www.youtube.com/watch?v=ul96dslvVwY)

**CKA 2024 Repo:** [Github Repository](https://github.com/piyushsachdeva/CKA-2024/tree/main)

Thank you for exploring with me! Let's keep learning, growing, and making a positive impact in the tech world together.