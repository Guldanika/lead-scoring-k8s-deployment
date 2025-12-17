
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



