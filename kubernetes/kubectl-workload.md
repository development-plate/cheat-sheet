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
        resources:
          limits:
            cpu: "500m"
            memory: "100Mi"
          requests:
            cpu: "250m"
            memory: "100Mi"        
```

### Listing Deployments and Their Pods

```
kubectl get deployments
```

```
kubectl get pods
```

### Rolling out a New Revision

The set image command is a handy shortcut for assigning a new image to a Deployment.
```
kubectl set image deployment nginx nginx=nginx:1.21.1 -n app --record
```

After the change, the rollout history should contain two revisions: one revision for the initial creation of the Deployment and another for the change to the image
```
kubectl rollout history deployments nginx -n app
```

### Rendering Deployment Details

```
kubectl describe deployment app-cache
```

### Scaling Workloads
__Manually Scaling a Deployment__


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

### Creating a ConfigMap

imperative command
```
kubectl create configmap env-configmap --from-literal=PROFILE=development --from-literal=DB_USERNAME=test -n data
```

Create the ConfigMap by using the YAML manifest defined in the file configmap.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-configmap
  namespace: data
data:
  PROFILE=development
  DB_USERNAME=test
```

```
kubectl apply -f configmap.yaml
```

### Consuming the ConfigMap

```
apiVersion: v1
kind: Pod
metadata:
  name: consumer
  namespace: data
spec:
  containers:
  - image: nginx
    name: nginx
    envFrom:
    - configMapRef:
        name: env-configmap
  restartPolicy: Never
```

```
kubectl apply -f pod.yaml
```

Verify that the environment variables
```
kubectl exec consumer -n data -- env
```

### Creating a Secret

```
apiVersion: v1
kind: Secret
metadata:
  name: api-basic-auth
  namespace: secure
type: kubernetes.io/basic-auth
stringData:
  username: bot
  password: pa88w0rD
```
__hint: the Secret automatically base64-encoded the values__

```
kubectl get secret api-basic-auth -o yaml -n secure
```

### Consuming the Secret

```
apiVersion: v1
kind: Pod
metadata:
  name: server-app
  namespace: secure
spec:
  containers:
  - image: nginx
    name: nginx
    envFrom:
    - secretRef:
        name: api-basic-auth
  restartPolicy: Never
```

Verify
```
kubectl exec server-app -n secure -- env
```

### Create a Secret (SSH private key)

```
cp ~/.ssh/id_rsa ssh-privatekey
```

```
kubectl create secret generic secret-ssh-auth --from-file=ssh-privatekey --type=kubernetes.io/ssh-auth -n magic
```

Ensure that the type of the Secret is correct:
```
kubectl get secret -n magic
```

```
kubectl get secret secret-ssh-auth -n magic -o json | jq '.data."ssh-privatekey"' -r
```

### Consuming the Secret

```
apiVersion: v1
kind: Pod
metadata:
  name: backend
  namespace: magic
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - name: ssh-volume
      mountPath: /var/app
      readOnly: true
  restartPolicy: Never
  volumes:
  - name: ssh-volume
    secret:
      secretName: secret-ssh-auth
```

Verify
```
kubectl exec backend -n magic -- ls /var/app/ssh-privatekey
```

### Identifying the node running the Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: alpine
  namespace: restricted
spec:
  containers:
  - name: alpine
    image: alpine:3.15.0
    args:
	- sh
	- -c
	- 'while sleep 3600; do :; done'
    resources:
      requests:
        memory: "256Mi"
        cpu: "1"
      limits:
        memory: "1024Mi"
        cpu: "2.5"
```

```
kubectl get pod alpine -n restricted -o yaml | grep nodeName:
```
