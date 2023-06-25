---
title: "Kubernetes Architecture"
datePublished: Sun Jun 25 2023 15:13:23 GMT+0000 (Coordinated Universal Time)
cuid: cljbkkuue002409i898dt5rdn
slug: kubernetes-architecture
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687706239617/2ad323ca-724f-47d3-bc84-de0e896dd6ec.png
tags: kubernetes, architecture, devops, wemakedevs

---

# Introduction:- 

**Kubernetes** is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. It was originally developed by Google and is now maintained by the Cloud Native Computing Foundation (CNCF). 

At its core, the Kubernetes architecture follows a master-worker model, where the cluster consists of a set of nodes and a control plane. The control plane, also known as the master, manages the overall cluster operations and maintains the desired state of the system. The nodes, also called workers or minions, are the machines where the actual containers are deployed and run.

# Architecture:- 

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687705760191/6f97832c-24e0-4beb-904f-977966b9fcc4.png align="center")

## Control Plane - 

The control plane is a crucial component of the Kubernetes architecture. It consists of a set of components that manage and control the overall operation of the cluster. The control plane's primary responsibility is to maintain the desired state of the system by monitoring the cluster, making decisions, and taking actions to ensure that the applications and resources are running as intended. The control plane works with other Kubernetes components such as the API server, ETCD, Scheduler, and Controller Manager to achieve this desired state.

* ### API Server 
    
    The API server is the central control point for the entire Kubernetes cluster. It exposes the Kubernetes API, which allows users, administrators, and other components to interact with the cluster. It handles incoming API requests, validates them, and processes them accordingly. 
    
    All the incoming requests via Kubectl first interact with the API server.
    
* ### ETCD 
    
    ETCD is a distributed key-value store that acts as the cluster's database. It is used by the control plane components to store and retrieve the cluster's configuration data, including information about nodes, pods, services, and other resources. This stores the desired state of the Kubernetes cluster.
    
* ### Controller Manager
    
    The Controller Manager runs various controllers that monitor and regulate the state of the cluster. These controllers are responsible for maintaining the desired state of the cluster. The controllers continuously compare the current state of the cluster with the desired state and take actions to reconcile any differences.
    
* ### Scheduler
    
    The Scheduler is responsible for assigning pods to nodes in the cluster. The scheduler aims to optimize resource utilization and ensure the high availability and performance of the cluster.
    

## Worker Nodes - 

* ### Kubelet 
    
    The Kubelet is an agent that runs on each worker node and communicates with the control plane. It receives instructions from the control plane and ensures that the containers specified in the pod specifications are running and in the desired state. The Kubelet interacts with the container runtime to start, stop, and monitor containers.
    
* ### Container Runtime
    
    The container runtime is the software responsible for running containers. Kubernetes supports various container runtimes, including Docker, containerd, cri-o, and others.
    
* ### Kube Proxy
    
    The kube-proxy is a network proxy that runs on each worker node. It facilitates network connectivity and load balancing between different pods and services in the cluster. It manages the network routing rules and ensures that communication between pods, as well as external access to services, is properly handled. This provides a unique IP address to the worker node.