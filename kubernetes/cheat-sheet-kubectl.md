# kubectl Cheat Sheet

[go to Collections](../README.md)

## kubectl

kubectl is the primary tool to interact with the Kubernetes clusters from the command line.

```
kubectl [command] [TYPE] [NAME] [flags]
```
### comand
see [https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

### flags
see [https://kubernetes.io/docs/reference/kubectl/kubectl/](https://kubernetes.io/docs/reference/kubectl/kubectl/)

## Install bash-completion

Source: [https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#enable-shell-autocompletion](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#enable-shell-autocompletion)

```text
echo 'source <(kubectl completion bash)' >>~/.bashrc
```

## Settings

| command | Description                                         |
| :---    | :---                                                |
| ```export KUBE_EDITOR="nano"``` | Set default editor for kubectl |
| ```kubectl config set-context --current --namespace=business``` | Set default namespace |

## Sections

[Role, RoleBinding, ClusterRole and ClusterRoleBinding](kubectl-rbac.md)

[Configuration](kubectl-configuration.md)

[Secrets, Security Context](kubectl-secrets.md)

[Workload](kubectl-workload.md)

[Troubleshooting](kubectl-troubleshooting.md)

## Links, books
1. [Certified Kubernetes Administrator (CKA) Study Guide](https://www.amazon.com/-/de/dp/1098107225)
2. [Certified Kubernetes Application Developer (CKAD) Study Guide](https://www.amazon.com/-/de/dp/1492083739)