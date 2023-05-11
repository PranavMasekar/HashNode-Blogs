---
title: "Container Orchestration Made Easy: Deploying Dockerized Applications on Minikube"
datePublished: Thu May 11 2023 08:56:27 GMT+0000 (Coordinated Universal Time)
cuid: clhiwarys000409lggwuxae1w
slug: minikube
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683795313716/defdb4f4-6b7b-4bf2-b6de-d4cf5d4d27c2.png
tags: docker, kubernetes, devops, minikube, wemakedevs

---

Containerization has revolutionized the way developers build, package, and deploy applications. Docker has emerged as one of the most popular containerization platforms, allowing developers to easily create, distribute, and run applications in isolated environments.

Minikube is a lightweight Kubernetes implementation that allows developers to create a local Kubernetes cluster on their machine. It is a great tool for testing and developing Kubernetes applications locally without the need for a full-scale production environment. 

In this blog post, we will explore how to run Dockerized applications on a local Minikube cluster. We will deploy a sample docker application to our local cluster and expose it to access from a browser.

Prerequisites - 

1. Minikube installed and Configure - [LINK](https://minikube.sigs.k8s.io/docs/start/#:~:text=Download%20and%20run%20the%20installer%20for%20the%20latest%20release.&text=Add%20the%20minikube.exe%20binary,to%20run%20PowerShell%20as%20Administrator.&text=If%20you%20used%20a%20terminal,reopen%20it%20before%20running%20minikube.)
    
2. Dockerfile
    
3. A lot of excitement
    

Now let’s start coding 🧑‍💻

* We need to have a Dockerfile from which we are going to create a DockerImage. For this blog post, I am going to use one of my project's Dockerfile.
    
    [https://github.com/PranavMasekar/DevOps-Project](https://github.com/PranavMasekar/DevOps-Project).
    

> Note: To know how to create DockerFile please read my article on [Docker](https://sungod.hashnode.dev/golang-docker).

* Now Open your terminal and run 
    
    ```bash
    minikube start
    ```
    
    Which creates a local one-node Kubernetes cluster for our development.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683794384581/87cad262-a8b1-470a-91c3-316395e5fdae.png align="center")
    
* Now run 
    
    ```bash
    eval $(minikube docker-env)
    ```
    
    This command is used to configure the Docker CLI to use the Docker daemon running inside a Minikube cluster. By configuring your Docker CLI to use the Docker daemon running inside the Minikube cluster, you can build Docker images on your local machine and have them available inside the Minikube cluster.
    
* Now let’s build our Docker image from Dockerfile so for that run
    
    ```bash
    docker build -t <ImageName> .
    ```
    
    Ensure that you are in the same folder as Dockerfile.
    
* We need to create deployment and service files for Kubernetes to create objects in Minikube. We are going to create two files deployment.yaml and service.yaml.
    
* Create deployment.yaml file inside the same folder -
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: go-restro-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: go-restro
      template:
        metadata:
          labels:
            app: go-restro
        spec:
          containers:
          - name: go-restro
            image: pranav18vk/go-restro:v3.0.0
            ports:
                - containerPort: 8000
    ```
    
    **apiVersion** - It specifies what API version of Kubernetes we are using to create K8’s objects.
    
    **kind** - It specifies what type of object are we creating in our case it is Deployment
    
    **metadata** - It gives some description of our K8’s object.
    
    **spec** - Here we define the specification of our deployment
    
    **replicas** - It defines how many Kubernetes objects we want to create, in our case it’s just one.
    
    **selector** - selector is the core grouping primitive in Kubernetes.
    
    **template** - This describes our pod specifications
    
    **containers** - Here we define our container specification like name, image which needs to be executed in that container and port which will be used to forward the traffic.
    
* Create service.yaml file inside the same folder - 
    
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: go-restro-nodeport
    spec:
      type: NodePort
      selector:
        app: go-restro
      ports:
      - name: http
        port: 8000
        targetPort: 8000
    ```
    
    **kind**: Service: This specifies the type of resource being defined. In this case, it's a Service resource.
    
    **metadata**: This is a section that defines metadata about the Service resource, such as its name, namespace, and labels.
    
    **name: go-restro-nodeport:** This is the name of the Service resource.
    
    **spec**: This is the section that defines the Service's specifications, such as its type, selector, and ports.
    
    **type: NodePort:** This specifies the type of Service that is being created. In this case, it's a NodePort type, which exposes the Service on a static port on each Node in the Kubernetes cluster.
    
    **selector**: This defines the selector that the Service will use to route traffic to the correct set of Pods. In this case, it's using the label app: go-restro to select the Pods.  
    
    **ports**: This defines the ports that the Service will expose. In this case, it's exposing port 8000 on the Service.
    
    **name: http**: This specifies the name of the port being exposed. In this case, it's named "http".
    
    **port: 8000**: This specifies the port number that the Service will listen on.
    
    **targetPort: 8000:** This specifies the target port that the Service will route traffic to on the selected Pods. In this case, it's the same as the port number being exposed by the Service.
    
* Let’s apply our changes -
    
    ```bash
    kubectl apply -f deployment.yaml
    ```
    
    To verify the execution of the deployment file run 
    
    ```bash
    kubectl get pods
    
    kubectl get deployment <deployment-name>
    ```
    
    Now let’s expose our service file -
    
    ```bash
    kubectl apply -f service.yaml
    ```
    
    Verify the service creation using 
    
    ```bash
    kubectl get services <service-name>
    ```
    
* To get the accessible URL run the following command -
    
    ```bash
    minikube service <service-name>
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683794614116/18325b41-b4e0-4b4e-8d59-7b485e90b53e.png align="center")
    
* Test the application by accessing the URL in Postman.
    
* After completing the tutorial, it's a good idea to clean up any resources that were created during the process. To do so, you can use the
    
    ```bash
    minikube delete
    ```
    
    command to delete the local Minikube cluster and all of its resources. This will ensure that you don't have any lingering resources that could affect future tests or deployments.
    

**Conclusion** : 

In this tutorial, we learned how to run a Dockerized application on a local Minikube cluster. We created a Docker image of our application and deployed it to the cluster using a Kubernetes Service. We also saw how to expose our application to the outside world and test it to ensure that it was working correctly.

Remember to clean up any resources that were created during the tutorial to ensure that your local machine remains clean and free of any leftover resources. 

**Happy Kubernetes development!**