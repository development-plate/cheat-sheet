# kubectl Cheat Sheet > Configuration

[go to kubectl Cheat Sheet](cheat-sheet-kubectl.md)

## Configuration

Imperativ command
```
kubectl create configmap db-config --from-literal=database_url=jdbc:postgresql://localhost/test --from-literal=user=test -o yaml --dry-run=client
```

***-o yaml --dry-run=client*** generate a yaml file like this.

configmap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: db-config
data:
  database_url: jdbc:postgresql://localhost/test
  user: test
```

config.properties
```properties
database_url=jdbc:postgresql://localhost/test
user=test
```

***--from-env-file*** use the content of file
```
kubectl create configmap db-config -o yaml --dry-run=client --from-env-file=config.properties
```

***--from-file***

Single file
```
kubectl create configmap db-config -o yaml --dry-run=client --from-file=config.txt
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: db-config
data:
  config.txt: |
    database_url=jdbc:postgresql://localhost/test
    user=test
```

Directory containing files

```text
kubectl create configmap db-config -o yaml --dry-run=client --from-file=config-folder
```

### Consuming a ConfigMap as Environment Variables

db-config.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod
spec:
  containers:
  - image: nginx:1.19.0
    name: app
    envFrom:
    - configMapRef:
        name: db-config
```

To verify

```text
kubectl exec configured-pod -- env
```

Result

```text
...
database_url=jdbc:postgresql://localhost/test
user=test
...
```

Reassigning environment variable keys for ConfigMap entries

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod
spec:
  containers:
  - image: nginx:1.19.0
    name: app
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: db-config
          key: database_url
    - name: USERNAME
      valueFrom:
        configMapKeyRef:
          name: db-config
          key: user
```

Result

```text
...
HOSTNAME=configured-pod
DATABASE_URL=jdbc:postgresql://localhost/test
...
```

### Mounting a ConfigMap as Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod
spec:
  containers:
  - image: nginx:1.19.0
    name: app
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: db-config
```

To verify

```text
kubectl exec -it configured-pod -- /bin/sh
```
