# Monitoring Nginx Ingress Controller with Prometheus & Grafana

Before getting started, ensure that you have the following tools installed:

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


