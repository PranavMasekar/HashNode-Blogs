# CI pipeline with CircleCI

In this article, we are going to build a complete CI/CD pipeline that builds a docker image from our project hosted on GitHub and publish that image on the docker hub. We are going to achieve this by using a tool called CircleCI.

CircleCI is a continuous integration platform that helps us to create our CI/CD pipeline much faster and more efficiently.

DockerHub is a platform where we host our docker image which can be publicly available just like we host our project on GitHub.

So the prerequisite to this article is to have a DockerHub account and a CircleCI account.

First, we need a project from which we can build a docker image.

As the article is focused on CI/CD not on actual what a project is so I am not going into details about the project but to give an overview we are going to use a Golang server that I have created.

[LINK](https://github.com/PranavMasekar/Golang-CRUD/tree/circle-ci)

So for this article, you can clone the repository to get started directly.

Shift to the circle-ci branch as the master branch already contains the CI pipeline.

The repository already contains the Dockerfile which will create docker images from the projects.

If you don’t know how to create a Dockerfile for the Go project please read my other article about Golang + Docker.

So in the root folder of the project create a folder named .circleci and inside that folder create a file called config.yml.

Every CircleCI project needs a .circle folder inside which we write the configuration of the pipeline.

```yaml
orbs:
  docker: circleci/docker@1.5.0
version: 2.1
executors:
  docker-publisher: 
    environment:
      IMAGE_NAME: pranav18vk/go-movies-crud
    docker: # Each job requires specifying an executor
    # (either docker, macos, or machine), see
      - image: circleci/node:latest
        auth:
            username: $DOCKERHUB_USERNAME
            password: $DOCKERHUB_PASSWORD

jobs:
    publishLatestToHub: 
      executor: docker-publisher
 
      steps: 
        - checkout
        - setup_remote_docker
        - run: 
            name: Publish Docker Image to Docker Hub
            command: |
              echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
              docker build -t $IMAGE_NAME .
              docker push $IMAGE_NAME:latest
workflows:
  version: 2
  build-master:
    jobs:
       - publishLatestToHub
```

Now let’s take a look at our config file -

**Orbs**: CircleCI orbs are like packages in our project which we can use in our CI pipeline. When we create a Ci pipeline from scratch we need to set up the software like in our case Docker. So doing this manually might become quite messy for larger projects, so to solve this CircleCI offers orbs that do these things for us. So here we have to use docker orb which is managed by CircleCI itself.

**Executor: The e**xecutor defines the underlying environment which is needed for the CI pipeline, In our case, we need to login credentials of DockerHub to push the docker image. So we name the executor as docker-publisher and we can name anything you want. Now in the environment, we defined our image name it should be in this format only.

&lt;DOCKERHUB-USERNAME&gt;/&lt;IMAGE-NAME&gt;

After that, we defined the Auth credentials of the docker hub, for security purposes we can’t publicly show our username and password so we need to configure them in the environment variables of our project. We will be doing that in the CircleCI UI interface later.

Now we define what jobs need to be executed by the executor, we only have one job i.e. to create and upload a docker image.

First, we gave a name to the job i.e. publishLatestToHub

And we define which executor is allowed to perform this job so we define docker-publisher

Then we define steps -

1. First, we pull the latest code from a git repository
    
2. Second, we set up the docker
    

run tag is used to run shell commands inside the system.

1. First, we log in to DockerHub by credentials
    
2. Second, we build the docker image
    
3. We push the docker image to DockerHub
    

Now we have created a Job that uploads a docker image but we need to set up a workflow that will be run

So just under the workflows tag mention the name of the job that we want to run i.e. publishLatestToHub

So this concludes our config file.

Now head towards the CircleCI interface and log in to your account.

Select the Projects tab, you will see a list of your GitHub projects.

> NOTE: To see the GitHub project make sure your account is connected to GitHub account, you can do that while creating an account or you can follow this [link](https://circleci.com/docs/github-integration/).

Go to the project named Golang-CRUD and click setup projects.

Now you will be prompted to select a branch from the git repository, select the repository which contains the .circleci file and then click setup projects.

![]( align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671903230231/f8c7c3dc-e2c7-46bf-81bf-74e26c545e66.png align="center")

Now we want to set the DockerHub credentials in our project so go to

Settings -&gt; Environment Variables

Add the credentials here and make sure the names of variables are the same as we defined in the config.yml file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671905122359/981f123e-5ba9-424d-b4ce-af5f2d4cb463.png align="center")

Now the setup is completed, to run the CI pipeline either you can make a push to the branch so the CI pipeline will be triggered automatically or you can do that manually as well.

Now in the dashboard section click on the triggered pipeline to run the CI pipeline.

Wait for CI to execute and after successfully building your docker image will be updated on Docker Hub.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671903318261/928bb6c7-c724-4e37-bde0-131ca0bef430.png align="center")

![]( align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671903326782/55e6b1bb-efd5-4300-b3e1-c9c7d9f74ff7.png align="center")

Congratulations!!! You have successfully created your first CI pipeline with CircleCI.