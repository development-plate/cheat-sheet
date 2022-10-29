# kubectl Cheat Sheet > Workload

[go to kubectl Cheat Sheet](cheat-sheet-kubectl.md)

### Workload
A workload is an application running on Kubernetes. 

```kubectl create deployment app-cache --image=memcached:1.6.8 --replicas=4```

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

```kubectl get deployments```

```kubectl get pods```

### Rendering Deployment Details

```kubectl describe deployment app-cache```

