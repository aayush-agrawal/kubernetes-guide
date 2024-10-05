# Kubernetes Pod: A Comprehensive Guide

---

### What is a Pod?
A Pod is the smallest and simplest Kubernetes object, representing a single instance of a running process in your cluster. It can contain one or more containers that share the same network and storage resources. Pods are typically used to host a single application or group of tightly coupled applications.

### Key Characteristics of a Pod:
- **Multiple Containers:** A Pod can host multiple containers that share resources like storage volumes, IP addresses, and ports.
- **Shared Network:** All containers in a Pod share the same network namespace, meaning they can communicate with each other over localhost.
- **Ephemeral:** Pods are designed to be temporary and disposable.

---

### Steps to Create, Run, Describe, and Execute Commands in Pods
#### 1. Creating a Pod
You can create a Pod either using imperative commands or by defining a YAML manifest file.

##### Using an Imperative Command:
```bash
kubectl run my-pod --image=nginx
```
This will create a Pod named my-pod running the nginx container.

##### Using a YAML Manifest File:
1. Create a file called my-pod.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
```

2. Apply the manifest:
```
kubectl apply -f my-pod.yaml
```

#### 2. Running the Pod
After creating the Pod, check if it's running:
```
kubectl get pods
```

Output:
```
NAME      READY   STATUS    RESTARTS   AGE
my-pod    1/1     Running   0          2m
```

#### 3. Describing a Pod
To view detailed information about a Pod:
```
kubectl describe pod my-pod
```

#### 4. Executing Commands Inside a Pod
You can use the kubectl exec command to execute commands inside a running Pod:
```
kubectl exec -it my-pod -- /bin/bash
wget http://localhost:80
```

The command will interact with the web server running inside the container.

#### 6. Exposing the Pod to Access It from the Browser
##### 1. To access the nginx server from your local machine, expose the Pod:
```
kubectl expose pod my-pod --type=NodePort --port=80
```

Find the allocated NodePort:
```
kubectl get services
```

Example output:
```
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
my-pod   NodePort    10.107.120.231   <none>        8080:32000/TCP   5m
```

nginx server is accessible at http://<minikube-ip>:32000

##### 2.Finding the Minikube IP
Get the Minikube IP:
```
minikube ip
```
If your service type is NodePort, sometimes it may require minikube tunnel for external access.
```bash
minikube tunnel
```
This may open up access to the service from your local machine.

You can also minikube provided built-in command to access services
```bash
minikube service my-pod
```

##### 3. Access the nginx server by combining the IP and NodePort
Access the nginx server by combining the IP and NodePort:
```
http://<minikube-ip>:32000
```

#### 7. Deleting a Pod
To delete a Pod:
```
kubectl delete pod my-pod
```

---

### Summary of Key Commands

| Action | Command                                             |
| ------ |-----------------------------------------------------|
| Create a Pod (imperative) | kubectl run my-pod --image=nginx                    |
| Create a Pod (YAML) | kubectl apply -f my-pod.yaml                        |
| Get list of Pods | kubectl get pods                                    |
| Describe a Pod | kubectl describe pod my-pod                         |
| Execute command in Pod | kubectl exec -it my-pod -- /bin/bash                |
| Curl or Wget within a Pod | curl http://localhost:80 or wget http://localhost:80 |
| Expose Pod to outside | kubectl expose pod my-pod --type=NodePort --port=80 |
| Delete a Pod | kubectl delete pod my-pod                           |
