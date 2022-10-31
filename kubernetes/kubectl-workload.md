# kubectl Cheat Sheet > Workload

[go to kubectl Cheat Sheet](cheat-sheet-kubectl.md)

## Workload
A workload is an application running on Kubernetes. 

```
kubectl create deployment app-cache --image=memcached:1.6.8 --replicas=4
```

deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-cache
  labels:
    app: app-cache
spec:
  replicas: 4
  selector:
    matchLabels:
      app: app-memcached
  template:
    metadata:
      labels:
        app: app--memcached
    spec:
      containers:
      - name: memcached
        image: memcached:1.6.8
```

### Listing Deployments and Their Pods

```
kubectl get deployments
```

```
kubectl get pods
```

### Rendering Deployment Details

```
kubectl describe deployment app-cache
```

### Scaling Workloads
**Manually Scaling a Deployment**


Imperative Command
```
kubectl scale deployment app-cache --replicas=6
```

or

Command

```
kubectl edit deployment app-cache
```
and change value at spec.replicas to 6.

**Manually Scaling a StatefulSet**

```
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: default
  labels:
    app: redis
spec:
  ports:
  - port: 6379
    protocol: TCP
  selector:
    app: redis
  type: ClusterIP
  clusterIP: None
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  replicas: 1
  serviceName: "redis"
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:6.2.5
        command: ["redis-server", "--appendonly", "yes"]
        ports:
        - containerPort: 6379
          name: web
        volumeMounts:
        - name: redis-vol
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: redis-vol
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

```
kubectl create -f redis.yaml
```

```
kubectl get statefulset redis
```

```
kubectl scale statefulset redis --replicas=3
```

```
kubectl get pods
```

### Autoscaling a Deployment

```
kubectl autoscale deployment app-cache --cpu-percent=80 --min=3 --max=5
```

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-cache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-cache
  minReplicas: 3
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 500Mi
```

```
kubectl get hpa
```