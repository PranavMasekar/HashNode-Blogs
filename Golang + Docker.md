# Golang + Docker

In this article, we are going to make a simple golang server of one HTML static page and we will also create a docker image out of this server. We will also learn some basics of docker and docker commands in this article.

So the prerequisite for this article is to have docker and golang installed on your local system.

For the context of the project, we are going to create a simple golang application with one route having a static page. Then we are going to create a docker image of the project and we will run the docker image on the local host. 

So let’s start by building our golang server -

We will have only one .go file i.e. main.go

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
	if r.URL.Path != "/hello" {
		http.Error(w, "404 not found", http.StatusNotFound)
		return
	}
	if r.Method != "GET" {
		http.Error(w, "Method not supported", http.StatusNotFound)
		return
	}
	fmt.Fprintf(w, "hello!")
}
func main() {
	http.HandleFunc("/hello", helloHandler)
	fmt.Println("Starting the server ................")
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		log.Fatal(err)
	}
}
```

We have a one /hello route that will just print hello into the web browser.

Now build the project and run it, after getting a log of Starting the server .....................

visit the [localhost:8080](http://localhost:8080) 

> NOTE: If you have Jenkins installed then its default port is 8080 so change the server port accordingly.

The first step of the article is done, now we are going to create a Dockerfile for the project.

So first create Dockerfile in the project directory.

```nginx
FROM golang:1.16-alpine
WORKDIR /app
COPY go.mod ./
RUN go mod download
COPY *.go ./
RUN go build -o golang-server .
EXPOSE 8080
CMD [ "/app/golang-server" ]
```

So let’s explore the Dockerfile

*   **FROM**:- Every custom docker image is based on some base image like ubuntu, etc. So we have used golang:1.16-alpine image which contains all the configurations of Golang.
    
*   **WORKDIR**:- This defines the current working directory in the Docker image.
    
*   **COPY**:- This command is used to copy the contents of the original file to the docker container.
    
    So we use this to first transfer our go.mod file which contains all the dependencies
    
*   **RUN**:- This is used to run any command in the docker container from the docker file.
    
*   **EXPOSE**:- Used this to expose the docker port 8080. 
    
    When the application is running in a docker container it doesn’t have any idea about the outside world that’s why we can’t access the local port of the docker container in our system.
    
    So for this, we use port forwarding and for that, we need to first expose the port of the container.
    
*   **CMD**:-  We use this to make a dockerfile out of the binary executable file created previously.
    

> NOTE: The name of the binary executable should be matched in the command as well.

Now after we create our docker file it’s time to create a docker image out of it.

For that open a terminal in your project directory and run - 

docker build -t &lt;ImageName&gt;:&lt;tag&gt; &lt;Directory&gt;

```bash
docker build -t go-demo-image:v1.0.0 .
```

Here -t defines a tag to image and . represents the current directory.

After some time, a docker image will be created and you can see it by using the docker images command.

Now to run a docker image we use the docker run command

```bash
docker run -p 8000:8080 go-demo-image:v1.0.0
```

\-p tag refers to port forwarding which means we are kind of connecting the docker container’s port 8080 to our local port 8000.

After executing this command we can see running containers by using the command docker ps.

Now visit [localhost:8000](http://localhost:8000) to see the webpage we built previously.

Congratulations!!! You have successfully created your first go docker image.