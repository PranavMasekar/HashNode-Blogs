---
title: "Simplifying CI/CD Pipelines with Argo CD and Kubernetes"
datePublished: Sun May 28 2023 03:30:42 GMT+0000 (Coordinated Universal Time)
cuid: cli6v5cd4026urunv6wro2p89
slug: argocd
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685198897464/9d2578da-a5b6-4ac2-a574-761a4fcc8e59.png
tags: kubernetes, cicd, minikube, argocd, wemakedevs

---

**Introduction :**

Argo CD has emerged as a game-changing tool for streamlining CI/CD workflows in Kubernetes deployments. With its declarative and GitOps-based approach, Argo CD simplifies the automation of application and configuration deployments. In this blog, we explore the power of ArgoCD and we will also create an entire CD pipeline with ArgoCD for our Minikube cluster.

**What is ArgoCD ??**

Argo CD is a powerful and widely adopted tool for continuous deployment (CD) in Kubernetes environments. It operates on the principles of declarative and GitOps-based continuous delivery. Argo CD functions as a control plane that connects to your Kubernetes cluster and continuously monitors the desired state of your applications. Argo CD connects with your public GitHub repository to get the latest desired state.

**Working on ArgoCD ??**

Most of the CI/CD tools like Jenkins, and GitHub actions work on push-based models. The Jenkins pipeline gets triggered when there is a push or commits in the GitHub repository. Then Jenkins tells Kubernetes to change the state of the application. But ArgoCD works on a Pull based model. The ArgoCD operator is installed inside the Kubernetes cluster and this operator continuously watches the GitHub repository. If there is a change or new commit in the repository then ArgoCD updates the state of the application automatically. 

But how do we define the state of the application ??

This is done using Helm charts. We store our Helm charts in the GitHub repository and ArgoCD watches these charts and if there are any changes in the Helm chart then it will update the state.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685196807066/f6c35340-bfda-495e-a2c1-a8da87e82c7b.webp align="center")

Now we have a basic understanding of how ArgoCD works, so let’s build an application that uses ArgoCD. 

Prerequisites - 

* Kubernetes knowledge
    
* Minikube 
    
* Helm chart
    
* GitHub Repository
    

For this blog, we are going to use one of my Helm charts i.e. [https://github.com/PranavMasekar/Helm-Charts.git](https://github.com/PranavMasekar/Helm-Charts.git). The chart is in the application folder. This helm chart just created deployment and NodePort service which runs a simple web application. Feel free to use your Helm chart.

🧑‍💻 Let’s start with the installation of ArgoCD - 

1. Start your minikube cluster using the command
    
    ```bash
     minikube start
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685196904810/de6395fe-67e4-44e1-92f7-be4bbca55e04.png align="center")
    
2. ArgoCD is installed in its namespace according to its [documentation](https://argo-cd.readthedocs.io/en/stable/) so we have to create a namespace called argocd and install ArgoCD there.\\
    
    ```bash
    kubectl create namespace argocd
    kubectl apply -n argocd -f   https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```
    
3. Access ArgoCD Server - 
    
    If we run
    
    ```bash
    kubectl get svc -n argocd
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685197464807/eeb86a0d-f50e-46ee-89ed-6319033db520.png align="center")
    
    We can see the argocd-server service running but its type of ClusterIP, we need to change the service type to LoadBalancer to access the service. So run 
    
    ```bash
    kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
    ```
    
    The output will be service/argocd-server patched
    
    Now to access the service we need to port forwarding because Minikube does not provide an external IP address for LoadBalancer service types. So for that run 
    
    ```bash
    kubectl port-forward svc/argocd-server -n argocd 9000:443
    ```
    
    Now visit [localhost:9000](http://localhost:9000), it will show a privacy error just click on Advanced and proceed to [localhost](http://localhost).
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685197527678/9e255298-74f5-4fe4-bc08-bd592d30ede0.png align="center")
    
4. Login to ArgoCD Server - 
    
    ArgoCD creates a random password for each new installation and stores it in the secret in Kubernetes in the name argocd-initial-admin-secret.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685197237625/8ad17827-660d-4753-984e-cda9fd506665.png align="center")
    
    To get the value from a Kubernetes secret run
    
    ```bash
    kubectl get secret -n argocd argocd-initial-admin-secret -o json
    ```
    
    Now copy the text in the password field but it's in base64 encoded format we need to decode the password to use it, So for that run
    
    ```bash
    echo <"YOUR_PASSWORD>" | base64 --decode
    ```
    
    Copy the output of the command go to the [localhost:9000](http://localhost:9000) put the username as admin and the password you copied and click on Sign In.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685197301773/b59e2307-f9b6-4301-9ba5-65dcc3d7eff7.png align="center")
    
    Now ArgoCD dashboard will show up -
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685197323891/d3c19a72-146b-4d5d-b07b-e2e19c4a3d50.png align="center")
    
5. Create an application in the ArgoCD dashboard
    
    1. Click on the New App icon
        
    2.  Give the application a name
        
    3.  Set project name as default
        
    4.  Set Sync policy to automatic
        
    5.  In the source, section provide the GitHub link of the repository and also select the branch and folder in which your helm chart is stored.
        
    6.  In the destination, section selects the default URL for cluster URL and type default in the namespace section.
        
    7.  In the Helm section provide values files path if required, in my case I am providing values.yaml file as values to helm chart.
        
    8.  Click on Create App.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685197752013/d4a526f8-ce22-429f-ba71-8b6c32d49a59.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685197762563/a30e91fc-c836-4f5a-a648-ffdb5a867830.png align="center")
        
        After the application is created it will be shown in the dashboard. As we selected the automatic option for the sync it will automatically fetch the desired state and will create deployments and services for us.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685197780168/00e74b2a-2ba1-45ca-80f7-f605365baf55.png align="center")
        
6. Access the application View
    
    Click on the application card in the dashboard to see their health status and other related things.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685197823937/55d96715-2a77-46ab-ba99-c94a247d9c48.png align="center")
    
7. Testing Auto sync functionality
    
    Now if we do any change in the repository of the helm chart like suppose we change the service type from NodePort to LoadBalancer. 
    
    It should get automatically updated in the Kubernetes cluster using ArgoCD.
    

**Congratulations** **!!!** 🎉 🎉 We successfully implemented a CD pipeline for Kubernetes using ArgoCD. Throughout this journey, we have seen how Argo CD simplifies the process of deploying and managing applications in Kubernetes clusters. Its integration with Git repositories and ability to compare and reconcile the desired state with the current state in the cluster