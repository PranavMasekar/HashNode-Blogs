---
title: "Sail through Kubernetes Deployments with Helm Charts"
datePublished: Sun Jun 04 2023 03:30:39 GMT+0000 (Coordinated Universal Time)
cuid: cligv88i705r8amnv2e460zyq
slug: helm-charts
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685686366852/07e125e7-6d9b-4299-9601-d97bed169856.png
tags: deployment, kubernetes, devops, helm, wemakedevs

---

Welcome to our tech blog, where we dive into the world of Helm charts and how they revolutionize Kubernetes deployments. As the complexity of containerized applications grows, managing their deployment and scaling becomes increasingly challenging. Thankfully, Helm charts offer a solution by providing a simple and standardized way to package, distribute, and manage applications on Kubernetes.

In this blog post, we will explore the power of Helm charts and uncover how they streamline the deployment process, making it easier than ever to manage and scale your applications in a Kubernetes environment. We will also see some helm commands and we will also create our helm chart.

Prerequisites - 

* Basic Kubernetes knowledge
    
* Minikube Installed
    
* Lot’s of passion to learn
    

**What is Helm?**

Helm is the package manager for Kubernetes, designed to simplify the deployment and management of applications on a Kubernetes cluster. For Flutter we have a pub, for Node we have NPM. With Helm, we can define and package our application as a Helm chart, which acts as a reusable template, which can be shared publicly to share your work. Helm enables you to easily install, upgrade, and uninstall applications on Kubernetes using simple Helm commands. 

One of the key advantages of Helm is its extensive library of pre-built charts available globally. This includes a famous monitoring stack containing Prometheus, Grafana and Alert Manager and many more. 

Helm also solves one of the core problems of Kubernetes and that is repetition. Traditionally we need to write separate deployment and services for each application. Well, this works when we have a Monolith application but in the case of Microservices, we need to create deployment and service files for each microservice instead of having the same format for each configuration file. Helm solves this problem by using templates. 

By using Helm we can create a custom template for our deployment and service files using Template interpolation and values.yaml file. We will see this in detail in a later part of the blog.

So as we now have a basic understanding of what Helm is, Let’s install Helm CLI in our local system. I am using a Linux system so I am going to show commands for Linux but for different operating systems, you can visit [this](https://helm.sh/docs/intro/install/).

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get_helm.sh

./get_helm.sh
```

You can use 

```bash
helm version
```

Command to ensure the installation of the helm.

As I said before we have prebuilt Helm charts of different applications available for us to reuse, But they are not hosted on the official website of Helm. These charts are hosted on [https://artifacthub.io/](https://artifacthub.io/). This is one of the major problems with Helm charts as anybody can host their helm chart on artifacthub.io, it becomes crucial to pick the right Helm chart which is hosted by a verified organization. So let’s move on to the actual hands-on part.

---

**🧑‍💻 Helm Commands and Demo -** 

Let’s use our first helm chart in the local minikube cluster.

So open up the link [https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack). This is the official helm chart for the complete monitoring stack on Kubernetes. By using this helm chart we can add monitoring to the cluster within seconds. So let’s run the following commands -

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

```bash
helm repo update
```

```bash
helm install monitoring prometheus-community/kube-prometheus-stack
```

First commands add the Helm chart in the repository of the Helm folder. The second commands help us to update if there is any newer version available. The third command installed a helm chart on our minikube cluster. So before running the third command ensure that Minikube is running. You can check the status of your Minikube cluster using the command -

```bash
kubectl get all 
```

This will list down all the deployments, services and other Kubernetes components created by the helm chart.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685684032411/9a9d2f89-61ef-485c-9a48-5e623177df80.png align="center")

And with one single command, we have installed a complete monitoring stack in our Minikube cluster. But we can’t access any of the services because these are of type ClusterIP which is not accessible. So for that, we need to somehow edit the service type of monitoring-grafana service from ClusterIP to NodePort. So for that, we need to dig up some [documentation](https://github.com/grafana/helm-charts/tree/main/charts/grafana) to find out which parameter in the monitoring helm chart changes the service type.

So if we observe the documentation of Grafana we can see that there is a variable name service.type that needs to be set to Nodeport instead of ClusterIP. So how do we change the configuration of the existing helm chart ??

We don't need to change the helm chart but we need to provide our custom values to the helm chart. Helm allows us to pass custom values through values.yaml file. We can find all lists of variables that can be changed in the official repository. So create myvalues.yaml file and add this

```yaml
grafana:
  adminPassword: admin
  service:
    type: NodePort
    nodePort: 30001
```

It is just a simple yaml file that defines the service type to NodePort and its port number. We also define a password for our Grafana dashboard. So let’s apply this file to our helm chart - 

```bash
helm upgrade monitoring --values=myvalues.yaml .
```

The syntax is like helm upgrade &lt;NAME&gt; --values=&lt;FileName&gt; Path

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685684160405/4f607f00-1b7e-484c-9ccd-df7128c937c9.png align="center")

Now we can access the Grafana UI with the following link -

&lt;minikube-ip&gt;/&lt;NodePort&gt;

i.e. in my case its [http://192.168.49.2:30001/](http://192.168.49.2:30001/login)

Enter username as admin and password as admin and click skip to access the dashboard. And with just a short amount of time, we have created monitoring of our cluster using Helm Chart.

To remove the helm chart from the cluster use

```bash
helm uninstall monitoring
```

---

**🧑‍💻 Creating custom Helm Chart -** 

So to create our helm chart run - 

```bash
helm create application 
```

Where an application is the name of a helm chart which can be anything. This will create a folder of the provided name. Inside that folder, helm has created subfolders like charts, templates and some yaml files. Everything inside the templates folder is the example files so go ahead and empty the entire folder. Also, delete all content from values.yaml file as we are going to create our own. So as of now, your folder structure should be like this: charts and template folders empty and values.yaml empty.

So let’s create templates for our deployment and service files.

Create deployment.yaml file inside the templates folder.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.deployment.metadataName }}
spec:
  replicas: {{ .Values.deployment.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.deployment.appName }}
  template:
    metadata:
      labels:
        app: {{ .Values.deployment.appName }}
    spec:
      containers:
      - name: {{ .Values.deployment.containerName }}
        image: {{ .Values.deployment.image }}
        ports:
            - containerPort: {{ .Values.deployment.containerPort }}
```

It’s a standard deployment file but all the hardcoded values are replaced with {{ .Values }} syntax. This is what makes it reusable. All the values which are defined using this syntax will be provided inside values.yaml file. I will explain that later when we make values.yaml file but let’s also create service.yaml file. 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.service.metadataName }}
spec:
  type: {{ .Values.service.serviceType }}
  selector:
    app: {{ .Values.service.appName }}
  ports:
  - name: http
    port: {{ .Values.service.port }}
    targetPort: {{ .Values.service.targetPort }}
```

Now to define values we modify values.yaml file.

```yaml
environment: dev

deployment: 
  metadataName: simple-web-app-deployment
  replicas: 1
  appName: simple-web-app
  containerName: simple-web-app
  image: yeasy/simple-web:latest
  containerPort: 80

service: 
  metadataName: simple-web-app-service
  appName: simple-web-app
  serviceType: NodePort
  port: 80
  targetPort: 80
```

We have defined all these values in this file and when we execute this command these will get substituted in our yaml file. The important thing is to notice the structure of the values and how to use it inside template files.

* We can access values in values.yaml file using {{ .Values }} syntax
    
* We need to provide a proper path for the values like if we want to use replicas values inside the deployment file, we have to use {{ .Values.deployment.replicas }}. We can’t use it directly {{ .Values.replicas }}.
    
* We can have as many values as possible in values.yaml file.
    
* All this templating is featured from Golang, to know more about this templating visit [this](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/).
    

As we have created our helm chart, let's install it in our Minikube cluster. So for that run - 

helm install &lt;Name&gt; &lt;Path-Helm-Chart-Folder&gt;

With this, we have successfully created our own Helm Chart using the Helm tool for Kubernetes. 

**Congratulations 🎉 🎉** You have successfully implemented Helm in your Kubernetes cluster.