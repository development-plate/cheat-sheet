# kubectl Cheat Sheet

[go to Collections](../README.md)

## kubectl

kubectl is the primary tool to interact with the Kubernetes clusters from the command line.

```
kubectl [command] [TYPE] [NAME] [flags]
```

## Settings

| command | Description                                         |
| :---    | :---                                                |
| ```export KUBE_EDITOR="nano"``` | Set default editor for kubectl |
| ```kubectl config set-context --current --namespace=business``` | Set default namespace |

## Sections

[Role, RoleBinding, ClusterRole and ClusterRoleBinding](kubectl-rbac.md)

[Configuration](kubectl-configuration.md)

[Workload](kubectl-workload.md)

[Troubleshooting](kubectl-troubleshooting.md)

## Links, books
1. [Certified Kubernetes Administrator (CKA) Study Guide](https://www.amazon.com/-/de/dp/1098107225)