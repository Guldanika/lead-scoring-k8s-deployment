
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

<img width="532" height="55" alt="proofs" src="https://github.com/user-attachments/assets/d334d371-22ef-42c5-985d-de230ce20474" />
<img width="684" height="102" alt="pods" src="https://github.com/user-attachments/assets/959b7e26-ef35-4cea-a74c-4d53e53d55e7" />
<img width="489" height="619" alt="servis" src="https://github.com/user-attachments/assets/8418a9d3-dec4-4bdb-86bc-3e366807a2b6" />
<img width="928" height="114" alt="running" src="https://github.com/user-attachments/assets/98746a87-d634-4792-8800-77e0a95c86cb" />
<img width="1035" height="812" alt="Снимок экрана 2025-12-16 в 17 38 15" src="https://github.com/user-attachments/assets/0a0768ef-d9b2-44a8-9582-b987718532a7" />


## Cleanup

```
kubectl delete -f k8s/
kind delete cluster
```

## License

MIT License — feel free to fork and extend.


Built as part of Machine Learning Zoomcamp 2025 by Guldanika Osmonova.




