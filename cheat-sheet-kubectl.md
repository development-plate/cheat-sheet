# kubectl Cheat Sheet

[go to Collections](README.md)

## Foreword

## Settings

| command | Description                                         |
| :---    | :---                                                |
| ```export KUBE_EDITOR="nano"``` | Set default editor for kubectl |

### Creating Roles

command

```kubectl create role read-only --verb=list,get,watch --resource=pods,deployments,services```

role.yaml

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: read-only
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - services
  verbs:
  - list
  - get
  - watch
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - list
  - get
  - watch
```

### Listing Roles

command

```kubectl get roles```

### Rendering Role Details

command

```kubectl describe role read-only```

### Creating RoleBindings
Roles and RoleBindings apply to a particular namespace.

command

```kubectl create rolebinding read-only-binding --role=read-only --user=johndoe```

RoleBinding.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-only-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: read-only
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: johndoe
```

### Listing RoleBindings

command

```kubectl get rolebindings```

### Rendering RoleBinding Details

command

```kubectl describe rolebinding read-only-binding```

### check a user’s permissions

command

```kubectl auth can-i --list --as johndoe```