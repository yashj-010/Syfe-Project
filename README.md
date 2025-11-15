# WordPress on Kubernetes with Monitoring

<img width="2835" height="1566" alt="image" src="https://github.com/user-attachments/assets/8b84c295-819c-4be0-b473-15d336e009a9" />

This project demonstrates a complete WordPress deployment on Kubernetes, including MySQL database, Nginx reverse proxy, and full monitoring stack using Prometheus and Grafana.

## Overview

I built this as part of learning Kubernetes deployments from scratch. The goal was to understand the entire workflow - from containerization to monitoring - without skipping any fundamentals.

**What's included:**
- Custom Docker images for WordPress, MySQL, and Nginx
- Kubernetes manifests and Helm charts
- Persistent storage configuration
- Monitoring with Prometheus and Grafana
- Custom dashboards and alerts

---

## Architecture

The application consists of three main components:

1. **MySQL Database** - Stores WordPress data with persistent storage
2. **WordPress** - PHP application running on Apache
3. **Nginx** - Reverse proxy handling incoming traffic

All components run as separate pods in Kubernetes, connected through services.

---

## Getting Started

### What You'll Need

- Docker (for building images)
- Minikube or any Kubernetes cluster
- kubectl CLI tool
- Helm 3

### Quick Start

**1. Clone and navigate to the project:**
```bash
git clone <your-repo-url>
cd wordpress-k8s-deployment
```

**2. Start Minikube:**
```bash
minikube start --cpus=4 --memory=4096
```

**3. Build the Docker images:**
```bash
# MySQL image
cd Dockerfiles/mysql
docker build -t mysql-custom:1.0 .

# Nginx image  
cd ../nginx
docker build -t nginx-custom:1.0 .

# WordPress image
cd ../wordpress
docker build -t wordpress-custom:1.0 .
```

**4. Deploy using Helm:**
```bash
cd ../../
helm install wordpress-app ./wp-helm-chart
```

**5. Check if everything is running:**
```bash
kubectl get pods
kubectl get svc
kubectl get pvc
```

You should see pods in Running state within a few minutes.

**6. Access WordPress:**
```bash
minikube service nginx-service
```

This will open WordPress in your browser.

---

## Development Journey

### Phase 1: Docker Compose Prototype

Before jumping into Kubernetes, I tested everything locally with Docker Compose. This helped me understand how the services communicate and what environment variables are needed.

Key things I learned:
- WordPress needs `WORDPRESS_DB_HOST`, `WORDPRESS_DB_USER`, etc.
- MySQL initialization can take time
- Nginx configuration for proxying PHP applications

### Phase 2: Manual Kubernetes Manifests

I wrote all the YAML files by hand first - deployments, services, secrets, PVCs. This was tedious but important for understanding what Helm actually does behind the scenes.

Challenges I faced:
- Getting PersistentVolumes to work correctly
- Understanding the difference between ClusterIP and NodePort services
- Managing secrets properly (initially hardcoded them by mistake)

### Phase 3: Helm Chart Creation

After understanding the manual approach, I converted everything into a Helm chart. I studied the Bitnami WordPress chart to understand best practices, then built my own from scratch.

The Helm chart makes it much easier to:
- Deploy/undeploy with single commands
- Customize values without editing YAML files
- Manage releases and rollbacks

### Phase 4: Adding Monitoring

The final step was setting up Prometheus and Grafana to monitor the cluster and application metrics.

What I monitored:
- Pod CPU and memory usage
- Nginx request rates and errors
- Database performance
- Storage utilization

---

## Persistent Storage

Both MySQL and WordPress use PersistentVolumeClaims to retain data across pod restarts.

**Storage setup:**
- MySQL: 5Gi volume for database files
- WordPress: 10Gi volume for wp-content (uploads, themes, plugins)

I initially experimented with NFS for ReadWriteMany access but simplified to RWO (ReadWriteOnce) for the final version since we're not scaling horizontally in this demo.

---

## Monitoring Setup

### Installing Prometheus and Grafana

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack
```

Wait for all pods to be ready:
```bash
kubectl get pods -l "release=monitoring" -w
```

### Accessing Grafana

```bash
kubectl port-forward svc/monitoring-grafana 3000:80
```

Open http://localhost:3000

**Login credentials:**
- Username: `admin`
- Password: Run this command to get it:
```bash
kubectl get secret monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

### Importing Dashboards

Once logged in:

1. Click **+** icon → **Import Dashboard**
2. Enter dashboard ID **8588** for Kubernetes pod metrics
3. Click Load, select Prometheus as data source
4. Click Import

Repeat for dashboard ID **12708** for Nginx metrics.

### Custom Monitoring Configuration

I created custom ServiceMonitor and PrometheusRule resources to scrape metrics from our application:

```bash
kubectl apply -f monitoring/servicemonitor.yaml
kubectl apply -f monitoring/prometheus-rules.yaml
```

These files configure:
- Metric scraping endpoints
- Alert rules for high CPU/memory usage
- Nginx-specific metrics collection

---

## Project Structure

```
.
├── helm-chart/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── _helpers.tpl
│       ├── configmap.yaml
│       ├── deployment-mysql.yaml
│       ├── deployment-wordpress.yaml
│       ├── pvc-mysql.yaml
│       ├── pvc-wordpress.yaml
│       ├── pvc.yaml
│       ├── secret.yaml
│       ├── service-mysql.yaml
│       ├── service-wordpress.yaml
├── mysql-build/
│   ├── Dockerfile
│   └── init-wordpress.sql
├── nginx-build/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── wordpress-proxy.conf
├── openresty-build/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── wordpress-proxy.conf
├── prometheus/
│   └── prometheus-rules.yaml
│   └── prometheus-values.yaml
├── wordpress-build/
│   └── Dockerfile
├── README.md


```

---

## Key Metrics Tracked

| Component | Metrics | Purpose |
|-----------|---------|---------|
| WordPress Pods | CPU, Memory, Restarts | Performance monitoring |
| MySQL Pod | CPU, Memory, Disk I/O | Database health |
| Nginx | Requests/sec, Error rate, Active connections | Traffic monitoring |
| Cluster | Node resources, Pod status | Overall health |

---

## Alerts Configuration

I set up basic alerts for:

- **High Pod CPU Usage** - Triggers when CPU > 80% for 5 minutes
- **Pod Restarts** - Alert when any pod restarts more than 3 times in 10 minutes
- **High Nginx Error Rate** - Triggers when 5xx errors exceed 5% of total requests

---

## Troubleshooting

**Pods stuck in Pending state:**
- Check PVC status: `kubectl get pvc`
- Verify sufficient cluster resources: `kubectl describe node`

**WordPress can't connect to MySQL:**
- Verify secret is created: `kubectl get secret mysql-secret -o yaml`
- Check MySQL pod logs: `kubectl logs <mysql-pod-name>`

**Can't access WordPress:**
- Check service type: `kubectl get svc`
- For Minikube, use: `minikube service nginx-service --url`

**Grafana shows no data:**
- Verify Prometheus is scraping targets: Port-forward Prometheus and check `/targets`
- Ensure ServiceMonitor is in the correct namespace

---

## What I Learned

This project taught me a lot about Kubernetes and cloud-native application deployment:

1. **Container networking** - How pods communicate through services
2. **Storage management** - PVs, PVCs, and storage classes
3. **Configuration management** - Using ConfigMaps and Secrets properly
4. **Helm templating** - The power of parameterized deployments
5. **Observability** - Why monitoring is crucial for production systems

The biggest challenge was debugging issues when things didn't work. Kubernetes error messages can be cryptic, and I learned to rely heavily on `kubectl describe` and `kubectl logs`.

---

## Future Improvements

Things I'd add if I continue this project:

- Horizontal Pod Autoscaling for WordPress pods
- Ingress controller instead of NodePort
- CI/CD pipeline for automated deployments
- Backup solution for MySQL data
- TLS certificates for HTTPS
- Redis cache for WordPress

---

## Useful Commands

**View all resources:**
```bash
kubectl get all
```

**Check logs:**
```bash
kubectl logs -f <pod-name>
```

**Update Helm release:**
```bash
helm upgrade wordpress-app ./wp-helm-chart
```

**Delete everything:**
```bash
helm uninstall wordpress-app
helm uninstall monitoring
```

**Debug pod issues:**
```bash
kubectl describe pod <pod-name>
kubectl exec -it <pod-name> -- /bin/bash
```

## Acknowledgments

- Bitnami WordPress Helm chart for reference
- Kubernetes & Helm documentation
- Prometheus & Grafana communities

**Author**: Yash Jain 

