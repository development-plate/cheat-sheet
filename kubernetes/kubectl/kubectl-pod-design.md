# kubectl Cheat Sheet > Pod Design

[go to kubectl Cheat Sheet](cheat-sheet-kubectl.md)

## Pod Design

### Labels and Annotation

Labels are an essential tool for querying, filtering, and sorting Kubernetes objects. Annotations only represent descriptive metadata for Kubernetes objects but have no ability to be used for queries.

___Labels___

Kubernetes lets you assign key-value pairs to objects so that you can use them later within a search query. Those key-value pairs are called labels. Kubernetes limits the length of a label to a maximum of 63 characters and a range of allowed alphanumeric and separator characters.

command line:

```text
kubectl run labeled-pod --image=nginx \
--restart=Never --labels=tier=backend,env=dev
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: labeled-pod
  labels:
    env: dev
    tier: backend
spec:
  containers:
  - image: nginx
    name: nginx
```

Queries:

```text
kubectl get pods --show-labels

kubectl get pods -l env=prod --show-labels

kubectl get pods -l 'team in (shiny, legacy)' --show-labels

kubectl get pods -l 'team in (shiny, legacy)',app=v1.2.4 --show-labels
```

___Annotation___

Annotations are declared similarly to labels, but they serve a different purpose. They represent key-value pairs for providing descriptive metadata. The most important differentiator is that annotations cannot be used for querying or selecting objects. Typical examples of annotations may include SCM commit hash IDs, release information, or contact details for teams operating the object.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotated-pod
  annotations:
    commit: 4711
    author: 'John Doe'
    branch: 'app/bugfix'
spec:
  containers:
  - image: nginx
    name: nginx
```

Inspecting Annotations

```text
kubectl describe pod annotated-pod | grep -C 2 Annotations:
...
Annotations:  author: John Doe
              branch: app/bugfix
              commit: 4711
...

$ kubectl get pod annotated-pod -o yaml | grep -C 3 annotations:
metadata:
  annotations:
    author: John Doe
    branch: app/bugfix
    commit: 4711
...
```

Modifying Annotations for a Live Object

```text
$ kubectl annotate pod annotated-pod oncall='0049 40 47114711'
pod/annotated-pod annotated

$ kubectl annotate pod annotated-pod oncall='0049 40 08150815' --overwrite
pod/annotated-pod annotated

$ kubectl annotate pod annotated-pod oncall-
pod/annotated-pod annotated
```

### Rolling Out a New Revision

```text
kubectl -n poddesign rollout history deployment my-deploy

deployment.apps/my-deploy
REVISION  CHANGE-CAUSE
1         <none>
```

```text
kubectl set image deployment my-deploy nginx=nginx:1.19.2
```

```text
 kubectl -n poddesign rollout history deployment my-deploy
deployment.apps/my-deploy

REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

```text
kubectl rollout history deployments my-deploy --revision=2
```

### Rolling Back to a Previous Revision

```text
kubectl -n poddesign rollout undo deployment my-deploy --to-revision=1
```

```text
kubectl -n poddesign rollout history deployment my-deploy

deployment.apps/my-deploy
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```

Kubernetes recognizes that revisions 1 and 3 are exactly the same. For that reason, the rollout history deduplicates revision 1 effectively; revision 1 became revision 3.

### Manually Scaling a Deployment

```text
kubectl -n poddesign scale deployment my-deploy --replicas=3
```

### Autoscaling a Deployment

#### Horizontal Pod Autoscaler

```text
kubectl -n poddesign autoscale deployment my-deploy --cpu-percent=70 --min=2 --max=8
```

```text
kubectl -n poddesign describe hpa my-deploy
```

