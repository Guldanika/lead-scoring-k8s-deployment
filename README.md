
# Lead Scoring Model — Kubernetes Deployment

Production-ready deployment of a lead scoring binary classification model using local Kubernetes (kind).

**Status**: Complete • **Date**: December 2025  
**Course**: Machine Learning Zoomcamp 2025 (Week 10 — Kubernetes)

## Overview

This repository demonstrates deploying a pre-built Docker image containing a lead scoring ML model into a local Kubernetes cluster using `kind`.

Key features:
- Zero-downtime capable (rolling updates)
- Scalable (replicas configurable)
- Observable (ready for Prometheus/Grafana integration)
- Tested end-to-end (prediction matches local Docker run)

## Prerequisites

- Docker
- kind ≥ v0.30.0
- kubectl

## Quick Start

```bash
# 1. Create local cluster
kind create cluster

# 2. Load the pre-built model image (replace with your image:tag from homework 5)
kind load docker-image <your-lead-scoring-image>:latest

# 3. Apply manifests
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# 4. Verify
kubectl get pods      # should show Running
kubectl get services  # note the service port mapping

# 5. Test the model
kubectl port-forward service/lead-scoring-service 9696:80
# In another terminal
python test_request.py   # or curl → expected prediction ~0.50

```
## Manifests

k8s/deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lead-scoring
  labels:
    app: lead-scoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: lead-scoring
  template:
    metadata:
      labels:
        app: lead-scoring
    spec:
      containers:
      - name: model
        image: <your-lead-scoring-image>:latest
        ports:
        - containerPort: 9696
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        readinessProbe:
          httpGet:
            path: /health   # if your app has health endpoint; optional
            port: 9696
          initialDelaySeconds: 10
          periodSeconds: 10
```

## k8s/service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: lead-scoring-service
spec:
  type: LoadBalancer   # works locally via port-forward in kind
  selector:
    app: lead-scoring
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9696
```

## Results (Screenshots)
Pods Running
Services
Prediction Output
Prediction result: {'lead_score': ~0.50} (matches local Docker execution)

<img width="894" height="485" alt="Снимок экрана 2025-12-16 в 20 03 16" src="https://github.com/user-attachments/assets/1132ad50-9bee-4f95-95c8-7311a6c56a2a" />
<img width="684" height="102" alt="Снимок экрана 2025-12-17 в 00 01 25" src="https://github.com/user-attachments/assets/4476e8e2-7239-44f7-bd90-449eab2a8e3a" />
<img width="906" height="346" alt="Снимок экрана 2025-12-16 в 17 37 58" src="https://github.com/user-attachments/assets/a4c8aa84-7e49-4992-ab0f-69569680ede1" />
<img width="1035" height="812" alt="Снимок экрана 2025-12-16 в 17 38 15" src="https://github.com/user-attachments/assets/0bd5e499-041b-437e-bca7-e547b524f5cb" />


## Cleanup

```
kubectl delete -f k8s/
kind delete cluster
```

## License

MIT License — feel free to fork and extend.




