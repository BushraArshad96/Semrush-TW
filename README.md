# Deployment of an Application on Kubernetes

Kubernetes is an open-source container orchestration engine for automating deployment, scaling, and management of containerized applications. Containers provide a redundant way to run your applications. Users need to have the containers that run the applications all the time, without any downtime. 

## Getting Started
To create an application image using Docker and deploy it on a Kubernetes cluster, you need to create a Python web application, containerize it using Docker, and then deploy it to Kubernetes.

## Prerequisites
Ensure that you have the following:

* An appropriate [Docker version](https://www.docker.com/products/docker-desktop/) for building and running containers.
* [Kubernetes CLI (kubectl)](https://kubernetes.io/docs/tasks/tools/) for interacting with your Kubernetes cluster.
* A proper [access to a Kubernetes cluster](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/).
* An appropriate [Python version](https://www.python.org/downloads/), such as 3.5 or newer to avoid syntax errors related to type annotations.

To create an application image using Docker and deploy it on a Kubernetes cluster, follow these steps:

1. You need to create the necessary directory structure for your project, using:

```bash
mkdir -p quickstart_docker/application quickstart_docker/docker/application
```

Your directory structure should look like this:

```bash
quickstart_docker/   # Root directory of the project
├── application/     # Directory containing the application code
└── docker/          # Directory containing Docker-related files
    └── application/ # Subdirectory for the application's Dockerfile
```

2. You need to create a simple Python web application. In the `quickstart_docker/application` directory, create a file named `application.py`:

```py

import http.server
import socketserver

PORT = 8000

Handler = http.server.SimpleHTTPRequestHandler
httpd = socketserver.TCPServer(("", PORT), Handler)

print("serving at port", PORT)
httpd.serve_forever()
```
> **_NOTE:_** This creates a basic web server that serves HTTP requests on port **8000**.

3. Once, the application is created, you need to containerize the application such that it can run in any environment. In the `quickstart_docker/docker/application` directory, create a file named `Dockerfile`, containing the following content:

```bash

# Use a base image from the Docker registry
FROM python:3.5

# Set the working directory inside the container to /app
WORKDIR /app

# Copying the application code into the container at /app
COPY ./application /app

# Exposing port 8000 
EXPOSE 8000

# Running the application when the container starts
CMD ["python", "/app/application.py"]
```

> **_NOTE:_**  This `Dockerfile` uses a **Python 3.5** base image, then copies the application code into the container, and configures the container to run the Python web server on port **8000**.

4. Now, you need to build the Docker image. Access a terminal and navigate to the root of the project directory i.e., `quickstart_docker`, then run the following command:

```bash
docker build -f docker/application/Dockerfile -t exampleapp .
-f docker/application/Dockerfile: Specifies the Dockerfile to use.
-t exampleapp: Tags the image with the name exampleapp.
.: Specifies the current directory as the build context.
```

You can verify that the image was built successfully, using:

```bash
docker images
```
Your output would look like this:

```bash
go
Copy code
REPOSITORY          TAG       IMAGE ID       CREATED          SIZE
exampleapp          latest    83ioe0edc28a   2 seconds ago    154MB
python              3.5       05stv8636w3f   6 weeks ago      154MB
```

5. _**(Optional)**_ If you want to deploy this image in a Kubernetes cluster, you can now push the image to a Container Registry. You need to push it to a container registry like Docker Hub, AWS ECR, or Google Container Registry.

For **Docker Hub**, you can use:

```bash
docker tag exampleapp yourdockerhubusername/exampleapp
docker push yourdockerhubusername/exampleapp
Replace yourdockerhubusername with your Docker Hub username.
```

6. Now, your image is ready, you can deploy it to your Kubernetes cluster. To deploy this to your Kubernetes cluster, save the YAML code to a file named `deployment.yaml`.

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: exampleapp-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: exampleapp
  template:
    metadata:
      labels:
        app: exampleapp
    spec:
      containers:
      - name: exampleapp
        image: exampleapp:latest
        ports:
        - containerPort: 8000
```

and then, apply deployment using:

```bash
kubectl apply -f deployment.yaml
```
You can now verify the deployment, using:

```bash
kubectl get deployments
```
This will create a deployment with the two replicas of your application running in the Kubernetes cluster.

> **_RESULT:_**  You have now successfully containerized a Python application and deployed it to a Kubernetes cluster. 