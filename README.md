# Monitoring Nginx Ingress Controller with Prometheus & Grafana

Please note that following tools are deployed in Azure Kubernetes Service (AKS) and the Prometheus and Grafana are deployed as LoadBalancer services.

- Nginx Ingress Controller
- Prometheus
- Grafana

## Installation

### Nginx Ingress Controller

You can follow the installation script I used for NGINX Ingress Controller Plus from [here](https://github.com/ericausente/nginx-plus-ic-deploy-in-one-go).
But I highly recommend to always refer to the official documentation [here](https://docs.nginx.com/nginx-ingress-controller/logging-and-monitoring/prometheus/).

### Prometheus Integration

1. Run the Ingress Controller with the `-enable-prometheus-metrics` command-line argument. This will expose NGINX or NGINX Plus metrics in the Prometheus format via the path `/metrics` on port 9113. You can customize the port using the `-prometheus-metrics-listen-port` command-line argument.

2. Make Prometheus aware of the Ingress Controller targets by adding the following annotations to the template of the Ingress Controller pod. This assumes that your Prometheus is configured to discover targets by analyzing the annotations of pods:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "10"
...
spec:
  progressDeadlineSeconds: 600
  replicas: 1
...
  template:
    metadata:
      annotations:
        prometheus.io/port: "9113"
        prometheus.io/scheme: http
        prometheus.io/scrape: "true"
      creationTimestamp: null
      labels:
        app: nginx-ingress
    spec:
      automountServiceAccountToken: true
      containers:
      - args:
        - -enable-prometheus-metrics
        - -prometheus-metrics-listen-port=9113
        - -enable-latency-metrics
...
        env:
        name: nginx-plus-ingress
...
        ports:
...
        - containerPort: 9113
          name: prometheus
          protocol: TCP
```


After deploying the NGINX Ingress Controller with Prometheus metrics enabled, you can test if it's properly configured using the following steps:

Check the NGINX Ingress Controller pod status:

```
kubectl get pods -n nginx-ingress -o wide
```

Ensure that the pod is running and note its IP address.

Create a network multitool pod for testing:

```
kubectl run nm2 -it --rm --image=wbitt/network-multitool:minimal sh
```

Use curl to access the Prometheus metrics endpoint on the NGINX Ingress Controller pod's IP address:
```
curl http://<NGINX-Ingress-Controller-IP>:9113/metrics
```

Replace <NGINX-Ingress-Controller-IP> with the actual IP address of the NGINX Ingress Controller pod obtained in step 1.

Example:
```
# curl http://10.244.4.93:9113/metrics

# HELP nginx_ingress_controller_ingress_resources_total Number of handled ingress resources
# TYPE nginx_ingress_controller_ingress_resources_total gauge
nginx_ingress_controller_ingress_resources_total{class="nginx",type="master"} 0
nginx_ingress_controller_ingress_resources_total{class="nginx",type="minion"} 0
nginx_ingress_controller_ingress_resources_total{class="nginx",type="regular"} 0
# HELP nginx_ingress_controller_nginx_last_reload_milliseconds Duration in milliseconds of the last NGINX reload
# TYPE nginx_ingress_controller_nginx_last_reload_milliseconds gauge
nginx_ingress_controller_nginx_last_reload_milliseconds{class="nginx"} 155
# HELP nginx_ingress_controller_nginx_last_reload_status Status of the last NGINX reload
# TYPE nginx_ingress_controller_nginx_last_reload_status gauge
nginx_ingress_controller_nginx_last_reload_status{class="nginx"} 1
# HELP nginx_ingress_controller_nginx_reload_errors_total Number of unsuccessful NGINX reloads
# TYPE nginx_ingress_controller_nginx_reload_errors_total counter
```

If the NGINX Ingress Controller is properly configured, you should receive Prometheus metrics data in the response.

Make sure to replace <NGINX-Ingress-Controller-IP> with the correct IP address obtained from your NGINX Ingress Controller pod. This test confirms that the Prometheus metrics are accessible and that the NGINX Ingress Controller is correctly configured.


# Prometheus and Grafana Resource Creation 

### Create the monitoring namespace
```
kubectl create ns monitoring
```

### Apply Grafana configuration
```
kubectl apply -f grafana.yaml
```

### Apply Prometheus configuration
```
kubectl apply -f prometheus.yaml
```

### Verify the external IP address of Prometheus and Grafana services
Note: It may take a few moments for the external IP to be assigned.
```
kubectl get svc -n monitoring
```

Verify the setup and access Prometheus and Grafana for monitoring.
