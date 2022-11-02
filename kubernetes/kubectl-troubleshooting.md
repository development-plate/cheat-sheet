# kubectl Cheat Sheet > Troubleshooting

[go to kubectl Cheat Sheet](cheat-sheet-kubectl.md)

## Get a Shell to a Running Container

Source: [kubernetes.io/docs](https://kubernetes.io/docs/tasks/debug/debug-application/get-shell-running-container/)

Create a Pod
```
kubectl apply -f https://k8s.io/examples/application/shell-demo.yaml
```

Verify that the container is running:
```
kubectl get pod shell-demo
```

Get a shell to the running container:
```
kubectl exec --stdin --tty shell-demo -- /bin/bash
```

Note: The double dash (--) separates the arguments you want to pass to the command from the kubectl arguments.