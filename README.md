# Clustertal Platform

This project orchestrates a full WordPress + MySQL deployment using Helm Charts for reproducibility and the Kube-Prometheus-Stack for observability, mimicking a real-world DevOps environment on Minikube.

## **Features**

* **Orchestration:** Managed via **Helm Charts** for modular and repeatable deployments.
* **Storage:** **StatefulSet** implementation for MariaDB to ensure data persistence via Persistent Volume Claims (PVC).
* **Networking:** **NGINX Ingress Controller** routing external traffic to internal services.
* **Observability:** Centralized monitoring stack using **Prometheus & Grafana** to track container uptime and cluster health.

## **Architecture**
To ensure separation of concerns, the project is structured into three distinct Helm components:

* **Database (`/db`):** A StatefulSet chart managing the MariaDB instance and its storage.
* **Application (`/wordpress`):** A Deployment chart managing the WordPress pods and Ingress rules.
* **Monitoring:** An external `kube-prometheus-stack` chart integrating Grafana and Prometheus.

## **Installation**

1.  **Prerequisites:**
    * Minikube (running with Docker driver)
    * kubectl
    * Helm 3+
    * AWS CLI (configured with ECR permissions for image pulling)

2.  **Setup:**
    # Start the cluster
    ```bash
    minikube start --driver=docker
    ```

    # Add required Helm repositories
    ```bash
    helm repo add ingress-nginx [https://kubernetes.github.io/ingress-nginx](https://kubernetes.github.io/ingress-nginx)
    helm repo add prometheus-community [https://prometheus-community.github.io/helm-charts](https://prometheus-community.github.io/helm-charts)
    helm repo update
    ```

## **Deployment**
You can deploy the stack components directly from your terminal using Helm.

### **1. Infrastructure (Ingress & Monitoring):**
#### Install NGINX Ingress Controller:
```bash
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

#### Install Monitoring Stack:
```bash
helm upgrade --install my-prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
```

### **2. Database (StatefulSet):**
#### Deploy MariaDB:
```bash
helm install my-db ./db
```

### **3. Application (Deployment):**
#### Deploy WordPress:
```bash
helm install my-wordpress ./wordpress
```


## **Usage & Verification**

Once deployed, you can verify the status of your stack and access the WordPress application.

### **1. Verify Status**
Ensure all pods (Database and WordPress) are in the `Running` state:
```bash
kubectl get pods
```

Expected Output:
```bash
NAME                            READY   STATUS    RESTARTS   AGE
my-db-db-1                      1/1     Running   0          2m
my-wordpress-5d...              1/1     Running   0          1m
```

### **2. Accessing WordPress**
The application is exposed via the NGINX Ingress Controller at the hostname wordpress.local.

There are 2 options:

A:
To access the URL from your browser, you must map the Minikube IP to the hostname (on windows might need to run as Admin):
```bash
# 1. Get Minikube IP
minikube ip

# 2. Add to your hosts file (requires sudo)
# Example: 192.168.49.2 wordpress.local
echo "$(minikube ip) wordpress.local" | sudo tee -a /etc/hosts
```

Now navigate to: http://wordpress.local


B:
If you are running this on a remote server (like EC2) and cannot edit your local hosts file, forward the port directly:
```bash
kubectl port-forward svc/my-wordpress 8080:80 --address 0.0.0.0 &
```

URL: http://localhost:8080


## **Monitoring**
For a visual interface, you can access the Grafana Dashboard to monitor container uptime.

#### Accessing Grafana
Since this runs on Minikube, forward the port to your local machine:
```bash
kubectl port-forward -n monitoring svc/my-prometheus-grafana 3000:80 --address 0.0.0.0 &
```

URL: http://localhost:3000

Credentials: User: admin | Password: (Retrieve via Secrets)
password can be retrieved using the command:
```bash
kubectl get secret --namespace monitoring my-prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

#### Uptime Visualization
To visualize the specific uptime of the WordPress pod, use the following PromQL query in the "Explore" tab:
```bash
time() - kube_pod_start_time{namespace="default", pod=~"my-wordpress.*"}
```

* the 'duration (d hh:mm:ss)' option is recommended for more readable uptime:
<img width="325" height="393" alt="image" src="https://github.com/user-attachments/assets/e88ca3e3-cccc-4361-9f79-3ec2bb6c7726" />


#### Dashboard Preview:
In this example, the pod is running for more than 21 hours, but the dashboard only exists less than 3 hours:
<img width="462" height="297" alt="image" src="https://github.com/user-attachments/assets/8df6e70b-8e00-4c6f-aa29-25830444edc3" />


## **Cleanup**
To remove the application and database but keep the infrastructure:
```bash
helm uninstall my-wordpress my-db
```

To remove everything (including monitoring and ingress):
```bash
helm uninstall ingress-nginx -n ingress-nginx
helm uninstall my-prometheus -n monitoring
minikube stop
```
