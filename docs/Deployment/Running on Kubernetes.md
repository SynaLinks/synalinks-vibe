# Running Synalinks on Kubernetes

This guide outlines how to deploy Synalinks applications in a Kubernetes environment for scalable, production-grade deployments.

## Prerequisites

- A working Kubernetes cluster (local or cloud-based)
- `kubectl` CLI tool installed and configured
- Docker image of your Synalinks application
- Basic understanding of Kubernetes concepts

## Architecture Overview

A typical Synalinks deployment on Kubernetes consists of:

1. **API Server Pods**: Running the FastAPI application
2. **Model Server Pods**: For serving large language models
3. **Redis Deployment**: For caching and rate limiting
4. **PostgreSQL StatefulSet**: For persistent storage (optional)
5. **Ingress Controller**: For routing external traffic

## Step 1: Create Namespace

First, create a dedicated namespace for your Synalinks deployment:

```bash
kubectl create namespace synalinks
```

## Step 2: ConfigMaps and Secrets

Create ConfigMaps for configuration and Secrets for sensitive information:

```yaml
# config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: synalinks-config
  namespace: synalinks
data:
  MODEL_PATH: "models/scientific_qa.json"
  LOG_LEVEL: "INFO"
  MAX_CONCURRENCY: "4"
---
apiVersion: v1
kind: Secret
metadata:
  name: synalinks-secrets
  namespace: synalinks
type: Opaque
data:
  # base64 encoded values
  API_KEY: YW4tZXhhbXBsZS1hcGkta2V5Cg==
  DATABASE_URL: cG9zdGdyZXNxbDovL3VzZXI6cGFzc3dvcmRAcG9zdGdyZXMvc3luYWxpbmtzCg==
```

Apply the configuration:

```bash
kubectl apply -f config.yaml
```

## Step 3: Deploy Database (Optional)

If your application requires a database:

```yaml
# postgres.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: synalinks
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:14
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          value: synalinks
        - name: POSTGRES_USER
          value: user
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: synalinks-secrets
              key: POSTGRES_PASSWORD
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: synalinks
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  clusterIP: None
```

Apply the database configuration:

```bash
kubectl apply -f postgres.yaml
```

## Step 4: Deploy Redis for Caching

```yaml
# redis.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: synalinks
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7
        ports:
        - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: synalinks
spec:
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
```

Apply the Redis configuration:

```bash
kubectl apply -f redis.yaml
```

## Step 5: Deploy Synalinks API

Create a deployment for your Synalinks API:

```yaml
# api.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: synalinks-api
  namespace: synalinks
spec:
  replicas: 3
  selector:
    matchLabels:
      app: synalinks-api
  template:
    metadata:
      labels:
        app: synalinks-api
    spec:
      containers:
      - name: api
        image: your-registry/synalinks-api:latest
        ports:
        - containerPort: 8000
        env:
        - name: MODEL_PATH
          valueFrom:
            configMapKeyRef:
              name: synalinks-config
              key: MODEL_PATH
        - name: REDIS_URL
          value: redis://redis:6379
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: synalinks-secrets
              key: API_KEY
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: synalinks-secrets
              key: DATABASE_URL
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 15
          periodSeconds: 20
---
apiVersion: v1
kind: Service
metadata:
  name: synalinks-api
  namespace: synalinks
spec:
  selector:
    app: synalinks-api
  ports:
  - port: 80
    targetPort: 8000
  type: ClusterIP
```

Apply the API configuration:

```bash
kubectl apply -f api.yaml
```

## Step 6: Create Ingress for External Access

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: synalinks-ingress
  namespace: synalinks
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - api.synalinks.example.com
    secretName: synalinks-tls
  rules:
  - host: api.synalinks.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: synalinks-api
            port:
              number: 80
```

Apply the ingress configuration:

```bash
kubectl apply -f ingress.yaml
```

## Step 7: Horizontal Pod Autoscaling

Set up autoscaling for your API deployment:

```yaml
# autoscaling.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: synalinks-api-hpa
  namespace: synalinks
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: synalinks-api
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

Apply the autoscaling configuration:

```bash
kubectl apply -f autoscaling.yaml
```

## Monitoring and Logging

### Prometheus and Grafana

Add Prometheus metrics to your Synalinks application:

```python
from fastapi import FastAPI
from prometheus_fastapi_instrumentator import Instrumentator

app = FastAPI()
Instrumentator().instrument(app).expose(app)
```

Install Prometheus and Grafana in your cluster:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

### Logging with Loki

For log aggregation:

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm install loki grafana/loki-stack \
  --namespace monitoring \
  --set grafana.enabled=false
```

Configure your application to use structured logging:

```python
import logging
import json

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_record = {
            "timestamp": self.formatTime(record),
            "level": record.levelname,
            "message": record.getMessage(),
            "module": record.module,
            "function": record.funcName,
        }
        if hasattr(record, "props"):
            log_record.update(record.props)
        return json.dumps(log_record)

# Set up logger
logger = logging.getLogger("synalinks")
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)
```

## Performance Tuning

### Uvicorn Workers

Optimize the number of workers for your FastAPI application:

```yaml
command: ["gunicorn", "app:app", "--workers", "4", "--worker-class", "uvicorn.workers.UvicornWorker", "--bind", "0.0.0.0:8000"]
```

### Resource Allocation

Tune resource requests and limits based on observed usage:

```yaml
resources:
  requests:
    memory: "1Gi"
    cpu: "500m"
  limits:
    memory: "2Gi"
    cpu: "1000m"
```

### Redis Caching

Implement Redis caching for expensive operations:

```python
import redis
import json
from functools import wraps

redis_client = redis.Redis(host='redis', port=6379, db=0)

def redis_cache(expiration=3600):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # Create a cache key
            key = f"cache:{func.__name__}:{hash(frozenset(args))}"
            
            # Try to get from cache
            cached = redis_client.get(key)
            if cached:
                return json.loads(cached)
            
            # Execute the function
            result = await func(*args, **kwargs)
            
            # Store in cache
            redis_client.setex(
                key, 
                expiration, 
                json.dumps(result)
            )
            
            return result
        return wrapper
    return decorator
```

## Security Considerations

1. **Network Policies**: Restrict pod communication

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
  namespace: synalinks
spec:
  podSelector:
    matchLabels:
      app: synalinks-api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: redis
    ports:
    - protocol: TCP
      port: 6379
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
```

2. **Pod Security Context**: Run containers as non-root

```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
```

3. **Service Mesh**: Consider Istio for advanced security

## Maintenance and Updates

### Rolling Updates

Kubernetes automatically performs rolling updates. Configure update strategy:

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
```

### Backup Strategy

For database backups:

1. Use Velero for cluster-wide backups
2. Configure database backups:

```bash
kubectl create cronjob postgres-backup --image=postgres:14 --schedule="0 1 * * *" \
  --namespace=synalinks -- \
  /bin/sh -c 'pg_dump -h postgres -U user -d synalinks > /backup/synalinks-$(date +%Y%m%d).sql'
```

## Troubleshooting

Common issues and solutions:

1. **Pod Crash Loops**: Check logs with `kubectl logs <pod-name> -n synalinks`
2. **Memory Issues**: Increase memory limits or optimize application code
3. **Network Problems**: Verify network policies and service connectivity

## Example: Complete Deployment Script

Create a shell script to deploy everything:

```bash
#!/bin/bash
# deploy-synalinks.sh

# Create namespace
kubectl create namespace synalinks

# Apply configurations
kubectl apply -f config.yaml
kubectl apply -f postgres.yaml
kubectl apply -f redis.yaml
kubectl apply -f api.yaml
kubectl apply -f ingress.yaml
kubectl apply -f autoscaling.yaml

# Wait for deployment
kubectl rollout status deployment/synalinks-api -n synalinks

echo "Synalinks deployment complete!"
echo "API available at: https://api.synalinks.example.com"
```

Make it executable and run:

```bash
chmod +x deploy-synalinks.sh
./deploy-synalinks.sh
```

## Next Steps

- Implement CI/CD pipeline integration
- Set up automated testing in the Kubernetes environment
- Configure monitoring alerts
- Develop a disaster recovery plan 