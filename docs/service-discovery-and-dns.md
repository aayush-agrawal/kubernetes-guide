# Kubernetes Guide on Service Discovery and DNS

---

Kubernetes service discovery allows Pods and applications to communicate with each other within the cluster without explicitly specifying IP addresses. The Kubernetes DNS system automatically assigns DNS names to Services, making inter-Service communication seamless and dynamic.

In this guide, we will explore:
- What service discovery and DNS are in Kubernetes.
- How DNS works in Kubernetes.
- Setting up a Pod running MySQL.
- Setting up a dummy Pod with MySQL Client to connect to the MySQL database using DNS.
- Exploring the kube-system namespace and kube-dns service.
- How to verify the resolv.conf file.
- How to use nslookup to verify DNS resolution.

---

### 1. What is Service Discovery in Kubernetes?
In Kubernetes, Service Discovery is the mechanism that allows Pods and applications to discover and communicate with each other through Services. This eliminates the need to hard-code IP addresses or manually configure services to make applications discoverable.

Kubernetes uses DNS-based Service Discovery for name resolution. Each Service gets a DNS name, and Pods within the cluster can communicate with the Service using that DNS name. Kubernetes automatically sets up DNS records that map Services to their IP addresses.

---

### 2. What is DNS in Kubernetes?
DNS (Domain Name System) in Kubernetes allows Pods to discover Services by name. When a Service is created in Kubernetes, it is assigned an IP address (ClusterIP) and a DNS name. The DNS system resolves the DNS name to the ClusterIP, which forwards traffic to the backend Pods.

---

#### How DNS Works in Kubernetes:
Each Pod gets an internal IP address and can resolve Services by their DNS names.
Kubernetes runs a DNS server, which is responsible for translating the DNS names to the IP addresses of the Services.
The kube-dns service, which runs in the kube-system namespace, handles DNS resolution for the cluster.
The kube-dns service allows Pods to access Services in the following formats:
- **Service Name:** my-service
- **Service Name in a Namespace:** my-service.my-namespace
- **Fully Qualified Domain Name (FQDN):** my-service.my-namespace.svc.cluster.local

---

### 3. Setting up MySQL Pod and Service
##### MySQL Pod YAML
This Pod runs MySQL with a PersistentVolumeClaim for data persistence.

```yaml
# mysql-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  containers:
  - name: mysql
    image: mysql:5.7
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"  # Set MySQL root password
    ports:
    - containerPort: 3306  # MySQL default port
```

##### MySQL Service YAML
This Service exposes the MySQL Pod within the cluster, allowing other Pods to connect using the DNS name.
```yaml
# mysql-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
  - protocol: TCP
    port: 3306  # MySQL port
    targetPort: 3306
  clusterIP: None  # Headless service for DNS-based service discovery
```

##### Create the MySQL Pod and Service
```bash
kubectl apply -f mysql-pod.yaml
kubectl apply -f mysql-service.yaml
```

---

### 4. Setting up Dummy Pod with MySQL Client
To connect to the MySQL database running in the mysql Pod, we’ll create a dummy Pod that contains the MySQL client.

##### Dummy Pod YAML
```yaml
# dummy-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: dummy
spec:
  containers:
  - name: dummy-container
    image: mysql:5.7
    command: [ "sleep", "3600" ]  # Keep the container running
```

##### Create the Dummy Pod
```bash
kubectl apply -f dummy-pod.yaml
Install MySQL Client in Dummy Pod
```

After the Pod is running, connect to the Pod using kubectl exec and install the MySQL client.
```bash
kubectl exec -it dummy -- bash
# Inside the dummy Pod, install mysql-client
apt-get update && apt-get install -y mysql-client
```

---

### 5. Connecting to MySQL Database using DNS
Now that we have both the MySQL and dummy Pods running, you can connect to the MySQL database using DNS service discovery.

##### MySQL Connection Command
In the dummy Pod, use the mysql-client to connect to the MySQL service using the DNS name.
```bash
# From inside the dummy Pod
mysql -h mysql-service -u root -p
```
Here, mysql-service is the DNS name of the MySQL Service. Kubernetes resolves this DNS name to the corresponding Pod’s IP address.

---

### 6. Exploring the kube-system Namespace and kube-dns
The kube-system namespace contains the core components that run in the Kubernetes cluster, including the DNS service.

##### Check the kube-dns Service
To verify that DNS is running correctly, you can check for the kube-dns service in the kube-system namespace.

```bash
kubectl get svc -n kube-system

# Output should include something like:
NAME       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
kube-dns   ClusterIP   10.96.0.10    <none>        53/UDP,53/TCP   60d
```

##### Check the kube-dns Pod
Check that the kube-dns Pod is running and healthy:

```bash
kubectl get pods -n kube-system

# Output should show:
NAME                       READY   STATUS    RESTARTS   AGE
coredns-66bff467f8-vm28c   1/1     Running   0          60d
```

---

### 7. Verifying DNS Resolution with resolve.conf
To ensure DNS is properly configured in a Pod, you can check the /etc/resolv.conf file in any running Pod. This file is automatically configured by Kubernetes to point to the cluster's DNS service.

##### Verify DNS Configuration
```bash
kubectl exec -it dummy -- cat /etc/resolv.conf

# Output should contain a nameserver entry for the cluster DNS:
nameserver 10.96.0.10  # This is the kube-dns service ClusterIP
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
nameserver: Points to the kube-dns service.
search: Specifies the search domain for DNS names within the cluster (e.g., svc.cluster.local).
```

---

### 8. Verifying DNS with nslookup
The nslookup command is a network utility used to query DNS servers. In Kubernetes, you can use it to verify that the DNS names of Services are resolving correctly.

##### Install nslookup in the Dummy Pod
To use nslookup, you need to install it in your Pod. Since we are using a MySQL image, it doesn't come with nslookup pre-installed.

```bash
# Inside the dummy Pod, install nslookup
apt-get update && apt-get install -y dnsutils
```

##### Verify DNS Resolution with nslookup
Once dnsutils is installed, you can perform a DNS lookup for the MySQL Service.
```bash
# Run nslookup inside the dummy Pod
nslookup mysql-service

# Output should show the ClusterIP of the MySQL Service:
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   mysql-service.default.svc.cluster.local
Address: 10.96.50.161
```
This confirms that the DNS name mysql-service is being resolved correctly to the Service’s ClusterIP.

---

### 9. Summary of Key Commands
##### Create MySQL Pod and Service:

```bash
kubectl apply -f mysql-pod.yaml
kubectl apply -f mysql-service.yaml
```

##### Create Dummy Pod and Install MySQL Client:
```bash
kubectl apply -f dummy-pod.yaml
kubectl exec -it dummy -- bash
apt-get update && apt-get install -y mysql-client
```

##### Connect to MySQL using DNS:
```bash
mysql -h mysql-service -u root -p
```

##### Install nslookup in Dummy Pod:
```bash
apt-get install -y dnsutils
```

##### Verify DNS with nslookup:

```bash
nslookup mysql-service
```

##### Check kube-dns service:
```bash
kubectl get svc -n kube-system
```

##### Check DNS configuration:
```bash
kubectl exec -it dummy -- cat /etc/resolv.conf
```
