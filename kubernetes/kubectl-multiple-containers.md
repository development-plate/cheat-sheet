# kubectl Cheat Sheet > Multi-Container Pods

[go to kubectl Cheat Sheet](cheat-sheet-kubectl.md)

## Multi-Container Pods

### Init Containers

Init containers provide initialization logic concerns to be run before the main application even starts.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: business-app
spec:
  initContainers:
  - name: configurer
    image: busybox:1.32.0
    command: ['sh', '-c', 'echo Hello World', \
              'mkdir -p /usr/shared/app && \
              echo -e "{\"dbConfig\": \
              {\"host\":\"localhost\",\"port\":5432,\"dbName\":\"customers\"}}" \
              > /usr/shared/app/config.json']
    volumeMounts:
    - name: configdir
      mountPath: "/usr/shared/app"
  containers:
  - image: bmuschko/nodejs-read-config:1.0.0
    name: web
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: configdir
      mountPath: "/usr/shared/app"
  volumes:
  - name: configdir
    emptyDir: {}
```

Verify:

```text
kubectl logs business-app -c configurer

Hello World
```

### Sidecar Container

