# Running a Server in Minikube Docker
This guide provides an introduction to the Docker daemon, a step-by-step process to start Minikube, connect to Minikube’s Docker daemon, run a server within Minikube, and access that server from your browser.

---

### 1. What is Docker Daemon?
The Docker daemon (dockerd) is the core service responsible for running Docker containers. It listens for Docker API requests and manages Docker objects like images, containers, networks, and volumes. The Docker daemon communicates with other daemons to manage Docker services, and it's typically started as a background process on your machine.
- ##### Key Role of Docker Daemon:
    - Responsible for managing the entire lifecycle of containers.
    - Handles building, running, and distributing Docker containers.
    - Interacts with the Docker client (docker) to execute commands.
---

### 2. How to Start Minikube and Connect to the Minikube Docker Daemon
Minikube is a lightweight Kubernetes implementation that creates a local Kubernetes cluster. It comes with its own Docker environment, which allows you to use Docker without relying on Docker Desktop or another Docker setup.

##### Step 1: Install Minikube
Install Minikube on your system by following the official Minikube installation guide.
```bash
brew install minikube
```
##### Step 2: Start Minikube
To start Minikube and create a local Kubernetes cluster, run the following command:
```bash
minikube start
```
This command will start the Minikube Kubernetes cluster using the default driver (you can specify the Docker driver explicitly if needed):
```bash
minikube start --driver=docker
```

##### Step 3: Connect to Minikube's Docker Daemon
Minikube comes with its own Docker environment. You can connect to the Minikube Docker daemon by running the following command:

```bash
eval $(minikube docker-env)
```
This command sets environment variables in your terminal, allowing Docker commands to interact with Minikube’s Docker daemon. Now, any docker command you run in this terminal will be executed inside Minikube.

##### Step 4: Verify the Connection
After setting up the Docker environment, you can verify that you're connected to Minikube's Docker daemon by running:
```bash
docker ps
```
This will show the Docker containers running inside Minikube.

----

### 3. Running a Tomcat/NGINX Server Inside Minikube’s Docker
Once connected to Minikube's Docker environment, you can run a Tomcat or NGINX server inside a Docker container and access it from your browser.

#### Option 1: Running an NGINX Server
##### Step 1: Pull a Server Image
First, pull the NGINX image from Docker Hub:
```bash
docker pull nginx
```
Let's use a basic Python HTTP server image. You can pull the image from Docker Hub using the following command:
##### Step 2: Run the NGINX Server
Run an NGINX container, exposing port 80:
```bash
docker run -d -p 8080:80 --name my-nginx-server nginx
```
This starts the NGINX server and maps port 8080 on your local machine to port 80 inside the container.

#### Option 2: Running a Tomcat Server
##### Step 1: Pull the Tomcat Image
If you prefer Tomcat, pull the official Tomcat image:
```bash
docker pull tomcat
```

##### Step 2: Run the Tomcat Server
Run the Tomcat server, exposing port 8080:
```bash
docker run -d -p 8080:8080 --name my-tomcat-server tomcat
```
This starts the Tomcat server and maps port 8080 on your local machine to port 8080 inside the container.

##### Step 3: Verify the Server is Running
Check that the container is running by using:
```bash
docker ps
```
----

### 4. Accessing the Server from the Browser
To access the web server running inside Minikube’s Docker container, you need to expose the port to your local machine.

##### Step 1: Get Minikube's IP Address
Run the following command to get the IP address of your Minikube cluster:
```bash
minikube ip
```
Let's assume the IP address returned is 192.168.49.2.

##### Step 2: Access the Server
Open your browser and navigate to:
- For NGINX (running on port 8080):
```
http://192.168.49.2:8080
```

- For Tomcat (also on port 8080):
```
http://192.168.49.2:8080
```
This will load the default page served by your NGINX or Tomcat server.

---

### Step 5: Expose Kubernetes Services (Optional)
If you want to expose services running in Kubernetes, you can do so using the minikube service command. For example, if you have a service called my-service, you can run:
```bash
minikube service my-service
```
This command will open the service in your default web browser.

--- 

### 6. Summary of Commands
| Command | Description |
| ------- | ----------- |
| minikube start |	Start Minikube with the Docker driver |
| eval $(minikube docker-env) | Set up Docker to use Minikube's Docker daemon |
| docker pull nginx | Pull a Python image from Docker Hub |
| docker pull tomcat | Pull the Tomcat image from Docker Hub |
| docker run -d -p 8080:80 nginx | Run an NGINX server inside Minikube |
| docker run -d -p 8080:8080 tomcat | Run an Tomcat server inside Minikube |
| minikube ip | Get Minikube's IP address |
| http://<minikube-ip>:8080 | Access the server from a browser |
| minikube service <service-name> | Expose and access a Kubernetes service |

---

### Conclusion
This guide has introduced the Docker daemon, explained how to start and connect to Minikube's Docker environment, and demonstrated how to run either a Tomcat or NGINX server inside Minikube. You also learned how to access the server from your browser, making this setup ideal for local testing and development environments.