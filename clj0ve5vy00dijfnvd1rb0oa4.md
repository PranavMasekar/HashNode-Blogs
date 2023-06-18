---
title: "Exploring the Power of Multi-Stage Builds in Docker"
datePublished: Sun Jun 18 2023 03:30:39 GMT+0000 (Coordinated Universal Time)
cuid: clj0ve5vy00dijfnvd1rb0oa4
slug: multi-stage-builds-in-docker
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1686981805403/27943c73-2909-4058-ad5a-e6adb421c86a.png
tags: optimization, docker, golang, devops, wemakedevs

---

### Introduction:- 

In the fast-paced world of software development, Docker has emerged as a game-changer, revolutionizing the way applications are built, shipped, and deployed. Among its powerful features, Docker multi-stage builds have gained immense popularity for their ability to streamline the development workflow and optimize the resulting container images. With multi-stage builds, developers can significantly reduce image size, enhance security, and improve build times. In this blog, we delve into the world of Docker multi-stage builds, uncovering their benefits, exploring best practices, and providing practical insights to empower developers in harnessing the full potential of this technology. 

### How does Docker Multi-Stage Builds work? 

Multi-stage Docker builds allow developers to create more efficient and optimized container images by using multiple build stages in a single Dockerfile. Each stage represents a distinct phase of the build process.

During a multi-stage build, Docker creates temporary containers for each stage, enabling the execution of different commands and ensuring that important artifacts like binary files are stored in the final docker image.

Typically, the first stage is responsible for compiling source code or installing dependencies. Once this stage is complete, the desired artifacts are copied to the next stage. By separating the build process into distinct stages, unnecessary build tools or intermediate artifacts can be discarded, resulting in a final image that contains only the essentials. This leads to faster build times, smaller image sizes, and improved security by reducing the attack surface of the container.

### 🧑‍💻 Let’s dive into the coding part -

If you are not familiar with docker and docker files then please read my blog on docker which explains every syntax and [working of docker](https://sungod.hashnode.dev/golang-docker). 

So for this demo, I am going to use one of my existing golang projects i.e. [Restaurant Management System.](https://github.com/PranavMasekar/DevOps-Project/tree/DevOps) 

So let’s create our dockerfile - 

```dockerfile
# Stage 1: Build the Go binary
FROM golang:1.16-alpine AS builder

RUN mkdir /app
ADD . /app
WORKDIR /app

RUN go mod download
RUN go build -o go-restaurant .

# Stage 2: Create the final image
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/go-restaurant .

EXPOSE 8000
CMD ["./go-restaurant"]
```

Stage 1 - 

* Base Image: `golang:1.16-alpine` - It sets up the Go development environment on an Alpine Linux distribution.
    
* Builder Setup: `FROM golang:1.16-alpine AS builder` - This defines the first stage with the alias "builder" for building the Go binary.
    
* Adding application files: `ADD . /app` - It copies all the files from the current directory to the "app" directory in the container.
    
* Setting the working directory: `WORKDIR /app` - It sets the working directory for subsequent commands.
    
* Downloading Go Modules: `RUN go mod download` \- It downloads the required Go modules specified in the go.mod file.
    
* Building the Go binary: `RUN go build -o go-restaurant .` - It compiles the Go code and creates an executable binary named "go-restaurant" in the current directory.
    

Stage 2 -

* Base Image: `alpine:latest` \- It sets up the final image based on the Alpine Linux distribution.
    
* Installing certificates: `RUN apk --no-cache add ca-certificates` - It installs the CA certificates required for secure connections.
    
* Setting the working directory: `WORKDIR /root/` - It sets the working directory to the root directory in the final image.
    
* Copying the Go binary: `COPY --from=builder /app/go-restaurant .` \- It copies the Go binary ("go-restaurant") from the "builder" stage to the current directory in the final image.
    
* Exposing a port: `EXPOSE 8000` - It documents that the container listens on port 8000.
    
* Defining the command: `CMD ["./go-restaurant"]` - It specifies the command to run when the container starts. In this case, it executes the Go binary "go-restaurant".
    

Now we run the docker build command - 

```bash
docker build -t pranav18vk/go-restro:v0.0.1 .
```

Results - 

By using traditional docker builds the docker images for the same code-base are around 200MB. But using docker multi-stage builds reduces the size to around 12MB. Approximately **94%** of the image size is reduced due to this technique. 

**Congratulations!!!** on completing this blog on Docker multi-stage builds! You've successfully explored a powerful technique for optimizing container images and learned how to leverage multi-stage builds to streamline your development workflow.