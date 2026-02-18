# Clustertal Platform

A production-grade migration of the WordPress stack from Docker Compose to Kubernetes.

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
    * Helm 3+
    * AWS CLI (configured with ECR permissions for image pulling)

2.  **Setup:**
    ```bash
    # Start the cluster
    minikube start --driver=docker

    # Add required Helm repositories
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

#### Install Monitoring Stack:
```bash
helm upgrade --install my-prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace

### **2. Database (StatefulSet):**
#### Deploy MariaDB:
`helm install my-db ./db`

### **3. Application (Deployment):**
#### Deploy WordPress:
`helm install my-wordpress ./wordpress`

## **Monitoring**
For a visual interface, you can access the Grafana Dashboard to monitor container uptime.

#### Accessing Grafana
Since this runs on Minikube, forward the port to your local machine:
`kubectl port-forward -n monitoring svc/my-prometheus-grafana 3000:80 --address 0.0.0.0 &`

URL: http://localhost:3000

Credentials: User: admin | Password: (Retrieve via Secrets)
password can be retrieved using the command:
``

#### Uptime Visualization
To visualize the specific uptime of the WordPress pod, use the following PromQL query in the "Explore" tab:
`time() - kube_pod_start_time{namespace="default", pod=~"my-wordpress.*"}`

#### Dashboard Preview:
INSERT SCREENSHOT HERE
