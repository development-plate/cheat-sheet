# kubectl Cheat Sheet > Secrets

[go to kubectl Cheat Sheet](cheat-sheet-kubectl.md)

## Secrets

You can create a Secret imperatively with this command: 

```text
kubectl create secret
```

Similar to the command for creating a ConfigMap, you will have to provide an additional subcommand and a configuration option.

| Option| Description |
|---|---|
| generic | Creates a secret from a file, directory, or literal value.|
|docker-registry|Creates a secret for use with a Docker registry.|
|tls|Creates a TLS secret.|

Literal values

```text
kubectl create secret generic db-creds --from-literal=pwd=s3cre!
```

File containing environment variables

```text
kubectl create secret generic db-creds --from-env-file=secret.env
```

SSH key file

```text
kubectl create secret generic ssh-key --from-file=id_rsa=~/.ssh/id_rsa
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  creationTimestamp: null
  name: db-creds
data:
  pwd: czNjcmUh
```

### Consuming a Secret as Environment Variables

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
    - secretRef:
        name: db-creds
```

Verify

```text
kubectl exec configured-pod -- env
```

### Mounting a Secret as Volume

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
    - name: secret-volume
      mountPath: /var/app
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: ssh-key
```

Verify

```text
kubectl exec -it configured-pod -- /bin/sh

and inside
ls -1 /var/app
cat /var/app/id_rsa
```

### Security Contexts

Source: [[docs.kubernetes.io](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: non-root
spec:
  containers:
  - image: nginx:1.18.0
    name: secured-container
    securityContext:
      runAsNonRoot: true
```

```text
kubectl get pods

NAME       READY   STATUS                       RESTARTS   AGE
non-root   0/1     CreateContainerConfigError   0          7s
```

```text
kubectl describe pod/non-root

...
Events:
Type     Reason     Age              From               Message
----     ------     ----             ----               -------
Normal   Scheduled  <unknown>        default-scheduler  Successfully assigned \
                                                        default/non-root to minikube
Normal   Pulling    18s              kubelet, minikube  Pulling image "nginx:1.18.0"
Normal   Pulled     14s              kubelet, minikube  Successfully pulled image \
                                                        "nginx:1.18.0"
Warning  Failed     0s (x3 over 14s) kubelet, minikube  Error: container has \
                                                        runAsNonRoot and image \
                                                        will run as root
```

You will see that Kubernetes does its job; however, the image is not compatible. Therefore,  the  container  fails  during  the  startup  process  with  the  status  CreateContainerConfigError

You can use the bitnami/nginx container and it runs sucessful.

### Setting a security context on the Pod level

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fs-secured
spec:
  securityContext:
    fsGroup: 3500
  containers:
  - image: nginx:1.18.0
    name: secured-container
    volumeMounts:
    - name: data-volume
      mountPath: /data/app
  volumes:
  - name: data-volume
    emptyDir: {}
```

Verify

```text
kubectl create -f pod-file-system-group.yaml
pod/fs-secured created

kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
fs-secured   1/1     Running   0          24s

kubectl exec -it fs-secured -- /bin/sh

Inside container
id -G
cd /data/app
touch logs.txt
ls -l
-rw-r--r-- 1 root 3500 0 Jul  9 01:41 logs.txt
```

### Resource Boundaries

```text
kubectl create namespace team-awesome
```

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: awesome-quota
  namespace: team-awesome
spec:
  hard:
    pods: 2
    requests.cpu: "1"
    requests.memory: 1024m
    limits.cpu: "4"
    limits.memory: 4096m
```

Verify:

```text
kubectl describe resourcequota awesome-quota -n team-awesome

Name:            awesome-quota
Namespace:       team-awesome
Resource         Used  Hard
--------         ----  ----
limits.cpu       0     4
limits.memory    0     4096m
pods             0     2
requests.cpu     0     1
requests.memory  0     1024m
```

Verify:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx:1.18.0
    name: nginx
```

```
kubectl apply -f nginx-pod.yaml

Error from server (Forbidden): error when creating "nginx-pod.yaml": pods "nginx" is forbidden: failed quota: awesome-quota: must specify limits.cpu,limits.memory,requests.cpu,requests.memory
```

Solution:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx:1.18.0
    name: nginx
    resources:
      requests:
        cpu: "0.5"
        memory: "512m"
      limits:
        cpu: "1"
        memory: "1024m"
```