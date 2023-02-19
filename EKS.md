# Amazon Elastic Kubernetes Service

Greetings everyone !!! In this article, we are going to deploy a sample application on Amazon EKS i.e. Elastic Kubernetes Service and access that application through the internet.

The prerequisite for this article is going to be 

* AWS account 
    
* EKSCTL installed on the local system
    
* A sample Docker image
    

Let’s look at the big picture of this article. We are going to provision our Kubernetes cluster using a command line tool called EKSCTL. Then we are going to use kubectl commands to deploy our application on a cluster and access it on the internet.

To install EKSCTL on the local system follow this [LINK](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html).

If you don’t know how to create a Docker image then follow this [LINK](https://sungod.hashnode.dev/golang-docker).

Now let’s start with our implementation - 

Before provisioning our cluster let’s prepare our deployments and service files.

So create yaml file named deployment.yaml

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

Let’s take a while to analyze the deployment file

**apiVersion** - It specifies what API version of Kubernetes we are using to create K8’s objects.

**kind** - It specifies what type of object are we creating in our case it is Deployment

**metadata** - It gives some description to our K8’s object.

**spec** - Here we define the specification of our deployment

**replicas** - It defines how many Kubernetes objects we want to create, in our case it’s just one.

**selector** - selector is the core grouping primitive in Kubernetes.

**template** - This describes our pod specifications

**containers** - Here we define our container specification like name, image which needs to be executed in that container and port which will be used to forward the traffic.

Now let’s create service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: go-restro-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: go-restro
  ports:
  - name: http
    port: 80
    targetPort: 8000
```

Let’s take a while to analyze the service file

Starting part of the service file is almost the same as the Deployment file 

In the specification, we are using LoadBalancer as a type of service.

> **NOTE: Aws will charge for using LoadBalancer Service.**

In the specification, we define ports that will be used to forward the traffic. In our case traffic coming from port 80 will be forwarded to any of the worker nodes on port 8000.

So we are finished with our config files, now it’s time to provision our EKS cluster and deploy our application.

So for that create a variable.yaml file

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: restro
  region: ap-south-1
nodeGroups:
  - name: MyNodeGroup
    instanceType: t2.micro
    desiredCapacity: 2
    ssh:
      allow: true
# eksctl create cluster -f variables.yaml
```

**apiVersion**: This specifies the version of the Kubernetes API that this configuration uses. In this case, it's [eksctl.io/v1alpha5](http://eksctl.io/v1alpha5), which is the API version used by eksctl for managing EKS clusters.

**kind**: This specifies the type of Kubernetes resource being defined. In this case, it's ClusterConfig, which represents a configuration for an EKS cluster.

**nodeGroups**: This is a list of node groups that will be created as part of the cluster. A node group is a group of EC2 instances that run the Kubernetes worker nodes.

**name**: This is the name of the node group.

**instanceType**: This specifies the EC2 instance type that will be used for the worker nodes in this node group. In this case, it's t2.micro, which is a small, general-purpose instance type.

**desiredCapacity**: This specifies the initial number of instances to launch in the node group. In this case, it's set to 2.

**ssh**: This is a section for SSH settings for the node group. allow specifies whether or not SSH access will be allowed to the instances in the node group. In this case, it's set to true, which means that SSH access will be allowed.

So preparations are done, now it’s time to create our cluster. So for that use this command in the same directory in which variables.yaml file is stored.

```bash
eksctl create cluster -f variables.yaml
```

This will take up to 15 - 20 Min to create a cluster in Amazon EKS.

You don’t need to set up kubeconfig in this case because this is done by EKSCTL.

After the successful execution of the command, we need to create our deployment so for that use

```bash
kubectl apply -f deployment.yaml
```

Check the status of pods running in the cluster by

```bash
kubectl get pods
```

When all pods are running then let’s create our service - 

```bash
kubectl apply -f service.yaml
```

Check the status of the load balancer service

```bash
kubectl get svc
```

In response to the above command you will be getting an external URL of service, This link will be used to access our application from the internet.

To access the URL from anywhere on the internet you need to make sure the security group created by EKSCTL has a rule for such a use case. So for that login to the AWS console and go to Security Groups. EKSCTL creates two security groups, one for internal cluster networking and one for external networking. Go to the security group which is responsible for external networking and add a rule which allows traffic from all TCP and from anywhere in the world.

**Congratulations** 🎉 🎉 You have successfully deployed your first application on Amazon EKS using EKSCTL.